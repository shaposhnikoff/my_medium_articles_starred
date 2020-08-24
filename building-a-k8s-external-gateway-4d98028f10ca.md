
# Building a k8s External Gateway

This design for an external k8s Gateway can provide a high level of security from outside access. It ensures that only the specific ip address and ports used by services are exposed to outside traffic and provides the option to add traffic rate limiting to services using additional annotations in the services.

![](https://cdn-images-1.medium.com/max/2578/0*4aaWCYYr-KpNVJ0b)

In addition to your k8s cluster, the solution consists of three components.

* Linux Gateway. A host running Free Range Routing (FRR) and configured to use NetFilter Tables.

* Metallb. Metalb is a k8s Load Balancer manager. It uses the Loadbalancer API to allocate ip addresses and direct traffic to PODs. It operates in two modes, this solution uses BGP mode to advertise service addresses.

* NFT Operator. The nft operator works alongside metallb configuring firewall rules and linux traffic control for optional rate limiting.

The Linux Gateway. Linux has all of the necessary networking capabilities to create the k8s gateway. In this solution we recommend the use of Ubuntu 18.04LTS however you can use other distro’s if you wish. One of the benefits of using ubuntu 18.04 is that the default output queuing discipline (qdisc) is fq_codel (fair queuing with controlled delay). This queuing mechanism is ideally suited for our purposes as its fair queue functionality allocates on a flow basis and the controlled delay functionality avoids starvation without requiring significant configuration. Practically this means that this queuing scheme should provide some protection against a small number of bad actors, assuming the count doesn’t get too high or the bandwidth overrun. It can be configured on distros either by default as every interface must have a default qdisc (often fifo) or via linux-tc. In addition the nft_operator also configures the ingress qdisc on the public network interface. This special qdisc is applied on input path and can be used for filtering and input rate shaping, it’s used for the latter by the nft_operator. There are lots of possible configuration options for managing traffic, we will cover some of them in part 3. There are a number of good routing implementations for linux, we chose FRR, a well maintained fork of quagga that uses common industry configuration syntax. The routing configuration is pretty simple, it uses BGP towards the k8s system and if deployed in an enterprise network could use BGP towards the existing network, however the solution does not require the use of BGP on the public network side.

### Creating & configuring the Linux Gateway

1. Host. Ubuntu 18.04 LTS server on host or VM with a minimum of 2 NIC. When running in a VM, recommend having sufficient physical NIC cards to allow the use of passthru avoiding more complex linux bridge configurations. Using Linux bridge is possible but not covered here.

1. Install FRR. FRR for Debian derivatives is available at [https://deb.frrouting.org](https://deb.frr.org). Follow the instructions exactly

1. Configure Linux.
a. Configure the two NICs, one in the same address range as the k8s cluster, the other an address on the public network. Note that only a single address is required, an additional subnet is used for the Service gateway. Ubuntu configures these interfaces with netplan. 
b. Enable linux networking features. 
 /etc/sysctl.conf — add net.ipv4.fib_multipath_hash_policy=1
 Also check that forwarding is enabled net.ipv4.ip_forward=1

1. Configure FRR.
a. /etc/frr/daemons — bgpd=yes & restart frr
b. Configure frr using vtysh

<iframe src="https://medium.com/media/05703737b51df9a52b12dc7759b43869" frameborder=0></iframe>

*Note in the BGP configuration that this includes a bgp peer to an upstream router. Where there is an upstream peer, it would be customary to summarize the range allocated to metallb seen in the aggregate-address entry. When a host route is advertised by metallb in this range, FRR will advertise the aggregate route only limiting the host route to the gateway. (Note. The metallb webpage provides another alternative advertising summaries however that solution will install a less specific route in the gateway defeating the benefit of host routes)*

Using BGP dynamic peers removes the need to manually add a peer entry for every node running the metallb speaker. As this is dynamic a password is suggested not for security, more for avoiding misconfiguration of bgp neighbors incorrectly populating the routing table.

While discussing the BGP configuration, it’s worth noting that the connection between the metallb speaker nodes and the gateway is eBGP, shown by the different AS numbers used. iBGP, where the same AS number is used, or eBGP could be used in the configuration, however the selection mechanism is different with eBGP allowing more extensive routing policy to be applied, it is not being used in this case.

Finally redistribute connected avoids the need to statically specify the local interface networks.

## Metallb

MetalLB is a k8s Load Balancer manager. It uses the service API to allocate IP addresses to services and configures IPtables to map connectivity into PODs. It has an address manager or IPAM for keeping track of addresses used by the service and runs goBGP in pods called speakers on all of the nodes that provide external access. In our example all nodes will be running the speaker, we will discuss alternatives and impacts later. Each service configured to use a load balancer is allocated a host route advertised by the speakers to bgp peers, in this case the Gateway. Therefore if no services are advertised there are no routes in the Gateway to the k8s system causing all packets to be dropped at the Gateway. This behaviour is very different to standard routing behaviour where the routers task is to forward all packets and then a firewall is put in place to limit what can be passed. Combining Metallb using BGP with a router is a good way to create secure access assuming you are careful on how the overall routing structure is configured.

### Installing & configuring Metallb.

1. [https://metalb.universe.ft/installation](https://metalb.universe.ft/installation) provides comprehensive instructions for installed metallb, just follow the instructions.

1. Configure Metalb. Metallb is configured using a configmap. A simple configuration can be used. Just apply the following in the metalb namespace with kubectl apply -f

<iframe src="https://medium.com/media/66a14ca49b4b1970a802de3b4853666e" frameborder=0></iframe>

*Note that this document covers how this system operates with kubeproxy in the default mode, iptables mode, not IPVS. If your k8s cluster has a default configuration, it will be using iptables mode.*

## NFT_Operator.

This k8s ansible operator increases the security of the linux gateway. When a service is created in k8s, the protocol and port number are defined, metallb dynamically adds a host route and the nft_operator watches for services to be created and applies filters based upon the service configuration. The operator uses net filter tables, not IPtables. In addition to performance benefits, one of the most important aspects of nftables used is the ability to make filter table changes atomic. Unlike iptables, nftables processes the complete configuration and then replaces it avoiding incomplete configuration errors that can be introduced in iptables. As previously mentioned, the nft_operator also configures linux traffic control (tc) adding an ingress qdisc to the public interface. Adding an annotation to the service definition, a traffic rate can be added for the service to control bandwidth for that service.

### Installing & configuring the nft_operator

1. [https://github.com/acnodal/arbf-nft-operator](https://github.com/acnodal/arbf-nft-operator) provides detailed instructions for the installation of the operator

1. Configure the nft_operator. The operator is configured using custom resource

<iframe src="https://medium.com/media/fba82de4ecf5037780e72954df445197" frameborder=0></iframe>

*Note enable_ssh and enable_bgp are applied on the firewalled interface allowing ssh access and bgp peers to be established from the public network. If you’re troubleshooting you can add a rule to /etc/nftables.d/local-chain.nft to enable your specific addresses to pass, this file is created but not managed by the nft_operator.*

## How it works.

* The metallb speaker runs on each node and a peer is established between each node running the speaker and the Gateway

<iframe src="https://medium.com/media/f0c12ec43249bad9f15598e0fbecbaa1" frameborder=0></iframe>

* A k8s user defines a service to access Pods and the in the spec the type is defined as LoadBalancer

<iframe src="https://medium.com/media/3b8d2752b17d110a01723fb537e7ca5b" frameborder=0></iframe>

* Kubeproxy ipdates iptables in the k8s nodes to enable access to POD(s)

<iframe src="https://medium.com/media/a0e8754f20dd9c69258f44359d7ee6c7" frameborder=0></iframe>

*Note: Just a snippet, take a look at iptables-save on a node.*

* Metallb allocated an ipaddress from the configured range and the speakers will advertise reachability to that address as a /32 host route resulting in the route being in the gateways routing table. It also updates iptables adding rules to direct public addresses to POD(s) building on the iptables managed by kubeproxy. Note ECMP load balancing to the nodes.

<iframe src="https://medium.com/media/99095b66200cbaf4aa7b4efc3bda50aa" frameborder=0></iframe>

* Nft_operator updates the nftables in the gateway using ansible adding specific firewall rules for the service defined and ingress traffic shaping if specified ensuring that only traffic specified in the k8s service definition gets to the application or ingress pods.

<iframe src="https://medium.com/media/c9ecd1bff7f7990cfce96fff7979a4b9" frameborder=0></iframe>

In Understanding Packet Forwarding we describe in detail how traffic is forwarded to nodes, the use of externalTrafficPolicy and ingress controllers.

.
