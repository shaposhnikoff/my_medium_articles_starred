Unknown markup type 10 { type: [33m10[39m, start: [33m142[39m, end: [33m160[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m68[39m, end: [33m93[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m160[39m, end: [33m181[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m54[39m, end: [33m96[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m91[39m, end: [33m104[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m125[39m, end: [33m137[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m187[39m, end: [33m208[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m41[39m, end: [33m47[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m83[39m, end: [33m104[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m59[39m, end: [33m89[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m114[39m, end: [33m118[39m }

# Deploying multiple Traefik Ingresses with LetsEncrypt HTTPS certificates on Kubernetes

As detailed on my first article, I’ve set an architecture for Kubernetes to be as similar to “production” as possible even being run on small ARM boards.

Here I will detail the network where I use [Weaveworks Net](https://www.weave.works/oss/net/) as the overlay and focus on the LoadBalancer and Ingress controllers.

### Network Topology

![](https://cdn-images-1.medium.com/max/2000/1*laQBZYKVPlcAxrv3Vuo65w.png)

### IP Plan

    Network: 192.168.1.0/24

    Gateway: 192.168.1.1
    DNS: 192.168.1.1 (running dnsmasq on DD-WRT Router)
    Router DHCP range: 192.168.1.101 - 192.168.1.200

    Reserved: 192.168.1.2 - 192.168.1.15

    * 192.168.1.1 - Router
    * 192.168.1.3 - Managed Switch
    * 192.168.1.4 - RPi3 (media server)

    Kubernetes Nodes:
        - Master1: 192.168.1.50
        - Node1: 192.168.1.55
        - Node2: 192.168.1.56

    MetalLB CIDR: 192.168.1.16/28
        - 192.168.1.17 - 192.168.1.30

    Traefik Internal Ingress IP: 192.168.1.20
    Traefik External Ingress IP: 192.168.1.21

As detailed in the architecture above, I’ve deployed two Traefik instances to the cluster. One instance to serve the local requests in the internal wildcard domain managed in my router and another Traefik instance to serve the external requests coming from the internet thru a wildcard domain configured in my external DNS.

These instances have separate service IP addresses and each instance have it’s own ingress rules.

To allow external access, I’ve configured my external DNS server, managed by my domain registrar, to resolve all calls to the external domain *.cloud.domain.com using a wildcard entry. This “**A**” entry can be dynamically updated by the DynDNS config in the router and the CNAME points the wildcard to the A record.

Another option in case you use GoDaddy as registrar/DNS is generating their [API key](https://developer.godaddy.com/getstarted) and using [this project](https://github.com/carlosedp/docker-godaddy-ddns) to dynamically update the subdomain used here. I’ve created a deployment and configMap to run a pod updater.

![](https://cdn-images-1.medium.com/max/2064/1*p6O88Z9_T7AJ8AXT6zJjrA.png)

![](https://cdn-images-1.medium.com/max/2038/1*CY5te47E8b3EqnRGoAxXvw.png)

![Configuring Dynamic DNS in the router](https://cdn-images-1.medium.com/max/2000/1*X6DoxpJPZ8dL7qleBj5VhA.png)*Configuring Dynamic DNS in the router*

I’ve also set port-forwarding on my router forwarding the HTTP and HTTPS traffic to the IP address I requested on service manifest for the external Traefik instance. The IP pool is managed by MetalLB.

![](https://cdn-images-1.medium.com/max/2000/1*URs6Vm8FRhXSjU2O0fE-8A.png)

### MetalLB

MetalLB is a load-balancer implementation for bare metal Kubernetes clusters using standard routing protocols. It’s very simple to [deploy ](https://metallb.universe.tf/installation/)and can be configured to use [ARP mode](https://metallb.universe.tf/configuration/#layer-2-configuration) or [BGP mode](https://metallb.universe.tf/configuration/#bgp-configuration) (this requires a BGP capable router). I’ve [deployed](https://github.com/carlosedp/kubernetes-arm/blob/master/1-MetalLB/metallb-conf.yaml) with ARP mode.

What is great about MetalLB is that it provides the LB functionality exactly in the same way as a cloud LoadBalancer or big LB appliances would so you benefit having the same configuration on the lab or in a production or cloud environment.

### Internal Traefik ingress controller

The internal Traefik controller was deployed to the cluster using a serviceType: LoadBalancer in the [service manifest](https://github.com/carlosedp/kubernetes-arm/blob/master/2-Traefik/traefik-internal-service.yaml). The IP was requested manually from the pool to be able to configure on the DNS.

All applications are made accessible internally over the internal Traefik controller. To allow this, my internal router have a wildcard DNS entry resolving all *.internal.domain.com names to the allocated IP requested on the service manifest.
> The configuration added to DNSMasq options on router: address=/.internal.domain.com/192.168.1.20

This is the Deployment and ConfigMap used on internal Traefik ingress controller. It doesn’t require redirection to HTTPS (I’m using HTTP instead of HTTPS internally) neither certificate generation.

<iframe src="https://medium.com/media/d2ed7dc3b61f9340038086fa07d515fe" frameborder=0></iframe>

<iframe src="https://medium.com/media/37781bf485c1fe9dfc89afd0fb924de6" frameborder=0></iframe>

I also generate Prometheus statistics to be collected by my monitoring stack as detailed on [another post](https://itnext.io/creating-a-full-monitoring-solution-for-arm-kubernetes-cluster-53b3671186cb?source=user_profile---------4----------------).

To deploy the application ingresses, it’s just a matter of using a plain manifest with a PathPrefix rule pointing to the application service (K8s dashboard in this case):

<iframe src="https://medium.com/media/dfede89e735781624420619413ab7ebf" frameborder=0></iframe>

### External Traefik ingress controller

The external Traefik controller manages the access to applications from the internet.

This is a more delicate subject because I didn’t want all my applications available externally and also I required some level of protection to them like HTTPS and authentication for the ones that didn’t provide it.

Since this controller generates the certificates dynamically, they need to be in [cluster HA mode](https://docs.traefik.io/user-guide/cluster/) and share state and the certificates using a Consul or *etcd* KV store.

I deployed a Consul cluster with 3 nodes using Helm. Helm doesn’t provide ARM images but I’ve built them and have the deployment script/manifest in [https://github.com/carlosedp/kubernetes-arm/tree/master/7-Helm](https://github.com/carlosedp/kubernetes-arm/tree/master/7-Helm). After having Helm deployed and the client installed, it’s just a matter of:

    # Install Helm from [https://github.com/carlosedp/kubernetes-arm/tree/master/7-Helm](https://github.com/carlosedp/kubernetes-arm/tree/master/7-Helm) and execute the scripts just the first time if you don't have it already deployed

    # Get helm client tool
    ./get_helm.sh

    # Deploy Tiller, Helm's server side component
    ./helm_init.sh

    # Finally, deploy the Consul cluster
    helm install --name traefik stable/consul --set ImageTag=1.1.0

Another option would be using an Etcd cluster using CoreOS etcd-operator. The manifests are in the *etcd *dir and there is a batch file to deploy it.

The deployment is straightforward using the provided manifests from [here](https://github.com/carlosedp/kubernetes-arm/tree/master/2-Traefik):

    # First deploy etcd operator or Consul according to the instructions above.
     
    # Deploy external Traefik config
    kctl apply -f external-traefik-configmap.yaml

    # Load the configMap into KV store with a job
    kctl apply -f job-storeConfigMap-to-KV.yaml

    # Deploy external Traefik and it's service
    kctl apply -f external-traefik-service.yaml
    kctl apply -f external-traefik-statefulset.yaml

After deployment, the the KV store will have all keys for Traefik config and they will work as a cluster where only one replica will fetch the certificates and store them centralized. The values are loaded into KV store by a initContainer in de StatefulSet.

![](https://cdn-images-1.medium.com/max/2702/1*MMxwN035SSxBQBfd-9l4XA.png)

This is the config used for the external controller

<iframe src="https://medium.com/media/78fa157d9b984d7bbd23f9f01b14799b" frameborder=0></iframe>

The important parts in the configuration are the sections for HTTP to HTTPS redirection in [entryPoints] and the selector in [kubernetes]specifying that *only the ingresses with the label* traffic-type=external are picked by the external ingress.

Also, to provide HTTPS certificates, the [acme] section takes care of the requests to LetsEncrypt to dynamically generate and renew valid certificates. The method used is HTTP-01 that LetsEncrypt servers does a call to my local HTTP port (that’s why the port 80 is open and forwarded on my router) to validate that I own the domain.

In case configuration change is needed, update the ConfigMap, apply to the cluster and delete the Traefik pods so the StatefulSet recreates them. KV store will be updated by a initContainer in the pod itself.

![Kubernetes Dashboard with valid certificate](https://cdn-images-1.medium.com/max/2714/1*sVPFKUeu4grRTqU7SCyPSw.png)*Kubernetes Dashboard with valid certificate*

To create the external ingress, the format is pretty similar just adding the label traffic-type=externaland the forced HTTP to HTTPS redirection annotations like example below:

<iframe src="https://medium.com/media/21e8e3a6574969badc1e16c9bb0b7c61" frameborder=0></iframe>

Since the K8s dashboard doesn’t provide authentication, I’ve also set a simple auth using Traefik to protect it with a user/password. The authentication is stored on a secret in Kubernetes and is generated from a file created by a simple script that uses Openssl utility to create a [basic http auth](https://www.digitalocean.com/community/tutorials/how-to-set-up-password-authentication-with-nginx-on-ubuntu-14-04) file. The file can be edited to contain multiple users, one per line.

    #!/bin/bash
    if [[ $# -eq 0 ]] ; then
        echo "Run the script with the required auth user and namespace for the secret: ${0} [user] [namespace]"
        exit 0
    fi
    printf "${1}:`openssl passwd -apr1`\n" >> ingress_auth.tmp
    kubectl delete secret -n ${2} ingress-auth
    kubectl create secret generic ingress-auth --from-file=ingress_auth.tmp -n ${2}
    rm ingress_auth.tmp

The secret name and namespace must match the ones defined on the ingress manifest.

![](https://cdn-images-1.medium.com/max/2368/1*xFAyxMMvC9EiqNAhTlZ3yw.png)

## Considerations and quirks

**Lack of client IP on internal applications**

Due to a [limitation on MetalLB ARP mode](https://metallb.universe.tf/usage/#traffic-policies), it’s currently not possible to see the external client IP on the internal application logs. This limitation will be removed in the future and can be tracked in [this issue](https://github.com/google/metallb/issues/195). The logs will present the internal kube-proxy IP, usually something in the 10.40.0.0 range or similar.

After this gets solved, it will be just a matter of adding externalTrafficPolicy: “Local” to the Traefik services spec section.

### Conclusion

As can be seen, deploying a full stack of network elements to support your applications is not a complex task and can even provide HTTPS and load balancing across all nodes.

As usual, the files are available in [my repository](https://github.com/carlosedp/kubernetes-arm) and please, send me feedback on [Twitter ](https://twitter.com/carlosedp)or comments.

**References**

* [https://github.com/containous/traefik/issues/927](https://github.com/containous/traefik/issues/927)

* [https://github.com/containous/traefik/issues/725](https://github.com/containous/traefik/issues/725)

* [https://github.com/containous/traefik/issues/2712](https://github.com/containous/traefik/issues/2712)

* [https://docs.traefik.io/user-guide/cluster/](https://docs.traefik.io/user-guide/cluster/)

* [https://docs.traefik.io/user-guide/kv-config/#store-configuration-in-key-value-store](https://docs.traefik.io/user-guide/kv-config/#store-configuration-in-key-value-store)
