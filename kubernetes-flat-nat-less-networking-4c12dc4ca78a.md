
# Kubernetes — flat, NAT-less Networking

So, how exactly does the does the Kubernetes achieves the flat and NAT-less network for an inter-pod communication? Well, not surprisingly it doesn’t. As it requires the Container Network Interface (CNI) plugin to set up the network.

The Kubernetes does not mandates to use specific networking technology to communicate with each other but it does mandates that there must be a way out that the communication must take place regardless of wherever the pods reside — on the same worker nodes or on different worker nodes. However, the network provided to pods for communication must be such that the IP address of a pod in question must be the same address when it is being seen by other pods.

![](https://cdn-images-1.medium.com/max/2348/1*QE2DIiperskcBSPXa6Q_4g.jpeg)

Henceforth, the absence of NAT between pods would enable containerized applications running inside the pods to self-register themselves to other pods — something similar you can think of it as a Service Discovery.

Now the requirement of NAT-less communication can be extended to pod-to-node and node-to-pod communication as well, think of a node as Worker Node.

Having said that, there are various methods and technologies available to achieve this but all of them do have their own pros and cons because of which we will first try to exemplify how the inter-pod networking works in general.

**How Networking Works**

Considering that you have had a brush with Linux network namespaces, whenever a container pod is instantiated a virtual Ethernet interface pair

(a *veth* pair) is also created, one of which is in the host’s namespace and the other one is in the container’s network namespace (commonly seen as *eth0*). Both of these interfaces act as two ends of the same pipe — similar to two devices connected by an Ethernet cable.

![](https://cdn-images-1.medium.com/max/2330/1*v79u5ijJ4Is8AtCuW0O9Pw.jpeg)

The container is configured to use the host’s network namespace Bridge and the *eth0 *interface of it is assigned an IP from the bridge’s address range. Since all the containerized pods are connected to bridge, the applications are reachable by any network interface that’s connected to this bridge

Yes, you got it right! This enables the communication of pods within the worker nodes, namely between Pod A and Pod B via the same bridge and without any wild guesses, we must have a routing mechanism to enable the inter-node (worker node) pods communication.

The inter-node communication can be achieved by connecting the two worker nodes bridges which can be done with an overlay or underlay networks or by a regular layer-3 routing — although the layer-3 routing could be tricky here. Let’s see how we can do this next.

**Inter-node Communications**

Going by the basic network rules, we already know the pod IP addresses have to be unique across the whole cluster and to achieve that the bridge in the two worker nodes is using the non-overlapping address ranges.

So with this setup, when a Pod A talks to Pod C, the packet is sent by the container in Pod A through the *veth *pair, then through the bridge to the node’s physical adapter, then over the wire to the other worker node’s physical adapter, and then to the other node’s bridge and finally the packet reaches the veth pair of the destination containerized application or pod.

Having said that some form of routing needs to be in place, the above example of inter-worker node communication is just the ideal one. In a production scenario, layer-3 routing is not an optimal solution as the number of nodes can be as huge as tens of thousands. As a best practice to follow it’s easier to use an SDN switch, which will make the nodes appear as if they are connected to the same switch irrespective of the underlying network topology.

With all this handful of concepts in your mind, can you recognize, when a pod communicates with services out on the internet, the source IP of the packets the pod sends does need to be changed, because the pod’s IP is private, but source IP of outbound packets is changed to the host worker node’s IP address.

**Conclusion**

Finally, with the conclusion note, I would like to say that this is an oversimplified version of Kubernetes networking and the real scenarios would need another infrastructure services such as Istio to cater the needs of Kubernetes networking.

I will continue to write on the CNI plugins as a next chapter. In the meantime you can reach me @ankur.all.in@gmail.com.

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Join our community Slack and read our weekly Faun topics ⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
