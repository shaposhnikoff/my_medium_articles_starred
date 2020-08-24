Unknown markup type 10 { type: [33m10[39m, start: [33m262[39m, end: [33m274[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m616[39m, end: [33m631[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m657[39m, end: [33m661[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m411[39m, end: [33m426[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m489[39m, end: [33m502[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m80[39m, end: [33m93[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m321[39m, end: [33m336[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m106[39m, end: [33m115[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m231[39m, end: [33m239[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m369[39m, end: [33m373[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m533[39m, end: [33m561[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m151[39m, end: [33m167[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m171[39m, end: [33m187[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m142[39m, end: [33m158[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m245[39m, end: [33m260[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m358[39m, end: [33m371[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m127[39m, end: [33m139[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m95[39m, end: [33m123[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m162[39m, end: [33m176[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m189[39m, end: [33m199[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m307[39m, end: [33m311[39m }

# Understanding kubernetes networking: ingress

In the first post of this series I described the network that enables pods to connect to each other across nodes in a kubernetes cluster. The second focused on how the service network provides load balancing for pods so that clients inside the cluster can communicate with them reliably. For this third and final installment I want to build on those concepts to show how clients outside the cluster can connect to pods using the same service network. For various reasons this will likely be the most involved of the three, and the concepts introduced in parts one and two are prerequisites to getting much value out of what follows.

First, having just returned from [kubecon 2017](http://events17.linuxfoundation.org/events/kubecon-and-cloudnativecon-north-america) in Austin I’m reminded of something I might have made clear earlier in the series. Kubernetes is a [rapidly maturing](https://venturebeat.com/2017/12/07/kubernetes-1-9-launches-to-guarantee-stability-for-key-features/) platform. Much of the architecture is plug-able, and this includes networking. What I have been describing here is the default implementation on [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine). I haven’t seen Amazon’s Elastic Kubernetes Service yet but I think it will be close to the default implementation there as well. To the extent that kubernetes has a “standard” way of handling networking I think these posts describe it in its fundamental aspects. You have to start somewhere, and getting these concepts well in hand will help when you start to think about alternatives like [unified service meshes](https://buoyant.io/2017/05/24/a-service-mesh-for-kubernetes-part-x-the-service-mesh-api/), etc. With that said, let’s talk ingress.

![](https://cdn-images-1.medium.com/max/3200/1*yjol6DXyAZVyFafOlMlCAQ.png)

## Routing is not load balancing

In the [last post](https://medium.com/@betz.mark/understanding-kubernetes-networking-services-f0cb48e4cc82) we created a deployment with a couple of pods, and a service that was assigned an IP, called the “cluster IP” to which requests intended for the pods were sent. I’ll continue building from that example here. Recall that the service’s cluster IP **10.3.241.152** is in an IP address range that is separate from the pod network, and from the network that the nodes themselves are on. I called this address space the “services network”, although it barely deserves the name, having no connected devices on it and consisting as it does entirely of routing rules. In the example we showed how this network is implemented by a kubernetes component called [kube-proxy](https://kubernetes.io/docs/reference/generated/kube-proxy/) collaborating with a linux kernel module called [netfilter ](http://www.netfilter.org/)to trap and reroute traffic sent to the cluster IP so that it is sent to a healthy pod instead.

![](https://cdn-images-1.medium.com/max/3200/1*Ow5A6_zjjwdKkf2Yi1KgYw.png)

Up to now we’ve been talking about “connections” and “requests” and even the more ambiguous “traffic,” but to understand why kubernetes ingress works the way it does we need to get more specific. Connections and requests operate at [OSI](https://en.wikipedia.org/wiki/OSI_model) layer 4 (tcp) or layer 7 (http, rpc, etc). Netfilter rules are routing rules, and they operate on IP packets at layer 3. All routers, including netfilter, make routing decisions based more or less solely on information contained in the packet; generally where it is from and where it is going. So to describe this behavior in layer 3 terms: each packet destined for the service at**10.3.241.152:80 **that arrives at a node’s **eth0** interface is processed by netfilter, matches the rules established for our service, and is forwarded to the IP of a healthy pod.

It seems pretty clear that any mechanism we use to allow external clients to call into our pods has to make use of this same routing infrastructure. That is, those external clients have to end up calling the cluster IP and port, because that is the “front end” for all the machinery we’ve talked about up to this point that makes it possible for us not to care where a pod is running at any given time. It’s not immediately obvious how to make this happen, however. The cluster IP of a service is only reachable from a node’s ethernet interface. Nothing outside the cluster knows what to do with addresses in that range. How can we forward traffic from a publicly visible IP endpoint to an IP that is only reachable once the packet is already on a node?

If you were trying to come up with a solution to this problem one of the things you might do is examine the netfilter rules using the [iptables ](http://ipset.netfilter.org/iptables.man.html)utility, and if you did you would discover something that seems surprising at first: the rules for the example service are not scoped to a particular origin network. That is, any packet from *anywhere *that arrives on the node’s ethernet interface with a destination of **10.3.241.152:80** is going to match and get routed to a pod. So can we just give clients that cluster IP, maybe assign it a friendly domain name, and then add a route to get those packets to one of the nodes?

![](https://cdn-images-1.medium.com/max/3200/1*skurXk737KHhzbXPbV-Xcg.png)

If you set things up that way it will work. Clients call the cluster IP, the packets follow a route down to a node, and get forwarded to a pod. At this point you might be tempted to put a bow on it, but this solution suffers from some serious problems. The first is simply that nodes are ephemeral, just like pods. They are not *as *ephemeral as pods, but they can be migrated to a new vm, clusters can scale up and down, etc. Routers operating on layer 3 packets don’t know healthy services from unhealthy. They expect the next hop in the route to be stable and available. If the node becomes unreachable the route will break and stay broken for a significant time in most cases. Even if the route were durable you’d have all your external traffic passing through a single node, which is probably not optimal.

However we bring client traffic in we have to do so in a way that doesn’t depend on the health of any single node in the cluster. There’s really no reliable way to do this using routing without some active management of the router, which is exactly kube-proxy’s role in managing netfilter. Extending kubernetes responsibilities out to management of an external router probably didn’t make much sense to the designers, especially given that we already have proven tools for distributing client traffic to a set of machines. They’re called load balancers, and not surprisingly that is the solution for kubernetes ingress that actually works durably. To see exactly how its time to climb up out of the layer 3 basement and talk about connections again.

To use a load balancer for distributing client traffic to the nodes in a cluster we need a public IP the clients can connect to, and we need addresses on the nodes themselves to which the load balancer can forward the requests. For the reasons discussed above we can’t easily create a stable static route between the gateway router and the nodes using the service network (cluster IP). The only other addresses available are on the network the nodes’ ethernet interfaces are connected to, **10.100.0.0/24** in the example. The gateway router already knows how to get packets to these interfaces, and connections sent from the load balancer to the router will get to the right place. But if a client wants to connect to our service on port 80 we can’t just send packets to that port on the nodes’ interfaces.

![](https://cdn-images-1.medium.com/max/3200/1*I03T4FnCjlPmHNXhao4arg.png)

The reason why this fails is readily apparent. There is no process listening on **10.100.0.3:80** (or if there is it’s the wrong one), and the netfilter rules that we were hoping would intercept our request and direct it to a pod don’t match that destination address. They only match the cluster IP on the service network at **10.3.241.152:80**. So those packets can’t be delivered when they arrive on that interface and the kernel responds with ECONNREFUSED. That leaves us with a conundrum: the network that netfilter is set up to forward packets for is not easily routable from the gateway to the nodes, and the network that is easily routable is not the one netfilter is forwarding for. The way to solve this problem is to create a bridge between these networks, and that is exactly what kubernetes does with something called a NodePort.

## NodePort Services

The example service that we created in the last post did not specify a type, and so took the default type **ClusterIP**. There are two other types of service that add additional capabilities, and the one that is important next is type **NodePort**. Here’s the example service as a NodePort service.

<iframe src="https://medium.com/media/c1f1cc1a97761270619965b6e8d292f0" frameborder=0></iframe>

A service of type NodePort is a ClusterIP service with an additional capability: it is reachable at the IP address of the node as well as at the assigned cluster IP on the services network. The way this is accomplished is pretty straightforward: when kubernetes creates a NodePort service kube-proxy allocates a port in the range 30000–32767 and opens this port on the **eth0** interface of every node (thus the name “NodePort”). Connections to this port are forwarded to the service’s cluster IP. If we create the service above and run **kubectl get svc service-test** we can see the NodePort that has been allocated for it.

    **$ kubectl get svc service-test
    NAME           CLUSTER-IP     EXTERNAL-IP   PORT(S)           AGE
    service-test   10.3.241.152   <none>        80:32213/TCP      1m**

In this case our service was allocated the NodePort 32213. This means that we can now connect to the service on either node in the example cluster, at **10.100.0.2:32213** or **10.100.0.3:32213 **and traffic will get forwarded to the service. With this piece in place we now have a complete pipeline for load balancing external client requests to all the nodes in the cluster.

![](https://cdn-images-1.medium.com/max/3200/1*Uo_wGCIlFopJZbf6THu0OQ.png)

In the diagram above the client connects to the load balancer via a public IP address, the load balancer selects a node and connects to it at **10.100.0.3:32213**, kube-proxy receives this connection and forwards it to the service at the cluster IP **10.3.241.152:80**, at which point the request matches the netfilter rules and gets redirected to the server pod on **10.0.2.2:8080**. It might all seem a little complicated, and it is in some ways, but it’s hard to think of a more straightforward solution that maintains all the cool capabilities put in place by the pod and service networks.

This mechanism is not without its issues. The use of NodePorts exposes your service to clients on a non-standard port. This is often not a problem as the load balancer can expose the usual port and mask the NodePort from end users. But in some scenarios such as internal load balancing on Google Cloud you’re going to be forced to propagate the NodePort upstream. NodePorts are also a limited resource, although 2768 is probably enough for even the largest clusters. For most applications you can let kubernetes choose the ports randomly, but if needed you can also set them explicitly. Lastly there are some restrictions around the preservation of source IPs in requests. You can refer to the [documentation article](https://kubernetes.io/docs/tutorials/services/source-ip/#source-ip-for-services-with-typeclusterip) on the topic for information on how to manage these issues.

NodePorts are the fundamental mechanism by which all external traffic gets into a kubernetes cluster. However by themselves they are not a complete solution. For the reasons outlined above you’re always going to need a load balancer of some kind in front of the cluster, whether your clients are internal or coming in over the public network. The platform’s designers recognized this and provided two different ways to specify load balancer configuration from within kubernetes itself, so let’s take a quick look at that next.

## LoadBalancer Services and Ingress Resources

These last two concepts are among the more complex functions that kubernetes performs, but I’m not going to spend a lot of time on them because they don’t really change any of what we just talked about. All external traffic ends up entering the cluster through a NodePort as described above. The designers could have stopped there and just let you worry about public IPs and load balancers, and indeed in certain situations such as running on bare metal or in your home lab that’s what you’re going to have to do. But in environments that support API-driven configuration of networking resources kubernetes makes it possible to define everything in one place.

The first and simplest approach to this is a third type of kubernetes service called a LoadBalancer service. A service of type **LoadBalancer** has all the capabilities of a NodePort service plus the ability to build out a complete ingress path, assuming you are running in an environment like GCP or AWS that supports API-driven configuration of networking resources.

<iframe src="https://medium.com/media/b4bf27e2f129dea73a49aabe52395bac" frameborder=0></iframe>

If we delete and recreate the example service on Google Kubernetes Engine we can soon see with **kubectl get svc service-test** that an external IP has been allocated.

    **$ kubectl get svc service-test
    NAME      CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
    openvpn   10.3.241.52     35.184.97.156   80:32213/TCP     5m**

I say “soon” although allocation of the external IP can take several minutes to happen, which is not surprising given the number of resources that have to be brought up. On GCP, for example, this requires the system to create an external IP, a forwarding rule, a target proxy, a backend service, and possibly an instance group. Once the IP has been allocated you can connect to your service through it, assign it a domain name and distribute it to clients. As long as the service isn’t destroyed and recreated (and there are rarely good reasons to do that) the IP won’t change.

Services of type LoadBalancer have some limitations. You cannot configure the lb to terminate https traffic. You can’t do virtual hosts or path-based routing, so you can’t use a single load balancer to proxy to multiple services in any practically useful way. These limitations led to the addition in version 1.2 of a separate kubernetes resource for configuring load balancers, called an [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/). LoadBalancer services are all about extending a single service to support external clients. By contrast an Ingress is a separate resource that configures a load balancer much more flexibly. The Ingress API supports TLS termination, virtual hosts, and path-based routing. It can easily set up a load balancer to handle multiple backend services.

The Ingress API is too large a topic to go into in much detail here, since as mentioned it has little to do with how ingress actually works at the network level. The implementation follows a basic kubernetes pattern: a resource type and a controller to manage that type. The resource in this case is an Ingress, which comprises a request for networking resources. Here’s what an Ingress resource might look like for our test service.

<iframe src="https://medium.com/media/57f2d681bcbf24dc80667f7b5d1f9af4" frameborder=0></iframe>

The ingress controller is responsible for satisfying this request by driving resources in the environment to the necessary state. When using an Ingress you create your services as type NodePort and let the ingress controller figure out how to get traffic to the nodes. There are ingress controller implementations for GCE load balancers, AWS elastic load balancers, and for popular proxies such as nginx and haproxy. Note that mixing Ingress resources with LoadBalancer services can cause subtle issues in some environments. These can all be easily worked around, but in general its probably best just to use Ingress for even simple services.

## HostPort and HostNetwork

The last two things I want to talk about really fall more into the category of interesting curiosities rather than useful tools. In fact I would suggest that they are anti-patterns for 99.99 per cent of use cases, and any implementation that makes use of either should get an automatic design review. I considered leaving them out entirely, but they are paths of ingress of a sort and so it’s worth a very brief look at what they do.

The first of these is HostPort. This is a property of a container (declared in a ContainerPort structure), and when set to a given integer port number causes that port to be opened on the node and forwarded directly to the container. There is no proxying, and the port is only opened on nodes the container is running on. In the early days of the platform before the addition of [DaemonSets ](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)and [StatefulSets ](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)this was a trick that could be used to make sure only one container of a type ran on any given node. For example I once used it to implement an elasticsearch cluster by setting HostPort to 9200 and specifying the same number of replicas as there were nodes. This would be considered a horrible hack now, and unless you are implementing a kubernetes system component you’re very unlikely to ever want HostPort set.

The second of these is even weirder in the context of kubernetes, and that is the HostNetwork property of a pod. When set to true this has the same effect as the **--network=host** argument to **docker run**. It causes all the containers in the pod to use the node’s network namespace, i.e. they all have access to **eth0 **and open ports directly on this interface. I don’t think it’s a stretch to suggest that you are never, ever going to need to do this. If you have a use case for this then you’re very likely already a kubernetes contributor and don’t require any assistance from me.

## Wrap-up

And that wraps up this three-part series on kubernetes networking. I’ve enjoyed learning about and working with this platform, and I hope some of that enthusiasm comes through in these articles. I think kubernetes heralds a revolution in terms of making it possible for us to reliably manage and interconnect fleets of containers rather than fleets of servers. It many ways it really is a data center in a bottle, and so it isn’t surprising that there is a fair bit of underlying complexity. I wrote these posts because I think that once you learn how each of the pieces works it all makes sense in a pretty elegant way. Hopefully I’ve contributed to making the path of entry easier for future new users of the platform.
