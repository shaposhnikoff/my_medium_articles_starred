
# Internal load balancing for kubernetes services on Google Cloud

As discussed in my recent post on kubernetes ingress there is really only one way for traffic from outside your cluster to reach services running in it. You can read that article for more detail but the tl;dr is that all outside traffic gets into the cluster by way of a nodeport, which is a port opened on every host/node. Nodes are ephemeral things and clusters are designed to scale up and down, and because of this you will always need some sort of load balancer between clients and the nodeports. If you’re running on a cloud platform like GKE then the usual way to get there is to use a type LoadBalancer service or an ingress, either of which will build out a load balancer to handle the external traffic.

This isn’t always, or even most often what you want. Your case may vary but at [Olark ](https://www.olark.com/)we deploy a lot more internal services than we do external ones. Up until recently the load balancers created by kubernetes on GKE were always externally visible, i.e. they were allocated a non-private IP that is reachable from outside the project. Maintaining firewall rules to sandbox lots of internal services is not a tradeoff we want to make, so for these use cases we created our services as type [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport), and then provisioned an internal TCP load balancer for them using [terraform](https://www.terraform.io/).

Creating an external load balancer is still the default behavior, but fortunately there is now a way to annotate a LoadBalancer service so that Google Compute Engine will build out an internal load balancer with a private IP. When this feature was first released a couple of months ago it didn’t work right, and so we continued to wire this stuff up manually. It’s since been fixed and I was recently able to test it and confirm that it works, so now seems like a good time to take a quick look at what this can do for you. Here’s a sample service:

    **apiVersion: v1
    kind: Service
    metadata:
      name: test
      annotations:
        cloud.google.com/load-balancer-type: "Internal"
      labels:
        app: test-app
    spec:
      type: LoadBalancer
      ports:
        - port: 80
          targetPort: 8080
          protocol: TCP
          name: http
      selector:
        app: test-app**

As you can see above the change is a simple one-liner annotation. Assuming we have some endpoints (pods) to handle this traffic, then when we create this service we will, after a minute or two, see an internal lb pointed at it:

    **$ kubectl get svc test
    NAME TYPE         CLUSTER-IP  EXTERNAL-IP   PORT(S)           AGE
    test LoadBalancer 10.3.250.65 10.130.15.192 80:31755/TCP      3m**

The ‘EXTERNAL_IP’ in that column header means “outside the cluster,” and as you can see the allocated address is in the private address space of the project. Pretty cool. There are a couple of limitations to internal load balancers, however, that you should be aware of. They are regional devices and can’t handle traffic to/from vms in other regions. They are layer 4 and can’t handle http/s traffic at layer 7, meaning no virtual hosts, path-based routing, tls termination, etc. They can only handle a maximum of five ports. After that you’ll need to make another lb. That last one isn’t much of a restriction since as far as I know kubernetes will drive GCE to create a new internal lb for each service port anyway. That’s something I’ll try to test and report back on later.

In any case, this is nice feature to have and will make it easier for us to spec and build out the networking stack for our kubernetes services from within our [helm ](https://github.com/kubernetes/helm)chart repositories, rather than having to maintain a separate terraform configuration that relies on the manual discovery of backend instance groups in order to properly configure an internal load balancer. As always when it comes to configurations less is more.
