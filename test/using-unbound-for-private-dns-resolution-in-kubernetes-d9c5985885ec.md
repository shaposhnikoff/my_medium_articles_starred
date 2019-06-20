Unknown markup type 10 { type: [33m10[39m, start: [33m101[39m, end: [33m114[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m36[39m, end: [33m49[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m204[39m, end: [33m216[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m379[39m, end: [33m397[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m613[39m, end: [33m620[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m656[39m, end: [33m688[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m56[39m, end: [33m73[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m286[39m, end: [33m319[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m270[39m, end: [33m287[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m9[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m151[39m, end: [33m156[39m }

# Using unbound for private DNS resolution in kubernetes



Workloads running in kubernetes pods commonly need access to services outside the cluster. In heterogeneous architectures where some services run in kubernetes and others are implemented on cloud VMs this often means resolving private DNS names that point to either specific hosts or to internal load balancers that provide ingress to groups of hosts.

In kubernetes the standard DNS resolver is [kube-dns](https://github.com/kubernetes/dns), which is a pod in the kube-system namespace that runs a [dnsmasq ](http://www.thekelleys.org.uk/dnsmasq/docs/dnsmasq-man.html)container as well as a container with some custom golang glue that interfaces between the dns server and the rest of the cluster control plane. The kube-dns service cluster IP is injected into pods via /etc/resolv.conf as we can see here:

    $ kubectl get svc kube-dns -n kube-system
    NAME     CLUSTER-IP  EXTERNAL-IP PORT(S)       AGE
    **kube-dns** **10.3.240.10** <none>      53/UDP,53/TCP 153d
    $ kubectl exec some-pod — cat /etc/resolv.conf
    **nameserver 10.3.240.10**
    search default.svc.cluster.local svc.cluster.local cluster.local options ndots:5

By default kube-dns provides resolution for kubernetes services inside the cluster using names in thecluster.local domain, and it forwards to a configured upstream DNS server for all other names. To get kube-dns to forward to a specific upstream for a private DNS zone we can edit its configmap in the kube-system namespace:

    apiVersion: v1
    data:
     **stubDomains: |
     {“myzone.net”: [“10.3.253.199”]}**
    kind: ConfigMap
    metadata:
     creationTimestamp: 2017–05–05T19:46:59Z
     labels:
     addonmanager.kubernetes.io/mode: EnsureExists
     name: kube-dns
     namespace: kube-system
     resourceVersion: “14031511”
     selfLink: /api/v1/namespaces/kube-system/configmaps/kube-dns
     uid: 9a32e0d4–31cb-11e7-a0b1–42010a800246

I’ve removed some annotations to shorten this example. There are two things of interest to note about the kube-dns configmap. The first is the *stubDomains* property, which is where we can set a zone and upstream resolver address as shown. Note that this address currently has to be an IP, although there is an [outstanding issue](https://github.com/kubernetes/dns/issues/82) aimed at making it possible to use resolvable service names. The second is the label *addonmanager.kubernetes.io/mode: EnsureExists*, which tells the addon-manager to make sure this configmap is always present. If you try to delete it and recreate it, for example, the second step will fail because the add-on manager will already have replaced it.

Once the configmap is updated using kubectl patch or some other method kube-dns will forward requests for names from the specified domain to the upstream resolver. Of course you’ll need a private DNS zone and resolver to point it at. In the past I have used both Amazon’s route53 and Google Cloud DNS to provide this service. The specifics of setting that up are beyond the scope of this post, but both are very easy to use and when you’re done you will have hosted name servers that respond to queries for your private zone. You could just point kube-dns at these servers and be done with it, however there are some advantages to running a local resolver in the cluster so we’ll look at how to do that next.
> # To get kube-dns to forward to a specific upstream for a private DNS zone we can edit its configmap

There are a number of tools you can use to provide private zone DNS lookups in your cluster. The one I’ll use for this example is [unbound](https://www.unbound.net/), simply because I have experience with it. Unbound is a lightweight caching, DNSSEC compliant name resolver written in C. The main advantage to running a local caching resolver in the cluster, rather than forwarding to external name servers, is simply the caching part. Internal service names tend to be pretty stable, and depending on how you set your DNS record TTLs the vast majority of name lookups can be handled within the cluster network.

In order to make it easy to install and configure unbound I’ve created [kunbound](https://github.com/Markbnj/kunbound) (unbound in kubernetes), which includes the dockerfile and necessary supporting files to build a slim unbound container on alpine, as well as a helm chart to install the DNS service in your cluster. It also includes a makefile, or you can build the image and interact with the helm chart directly if you prefer. For more information see the readme. If you aren’t familiar with helm you can [check it out here](https://helm.sh/). Helm is an emerging standard way to package and manage kubernetes resources, and we use it as a fundamental part of our automated deployment toolchain.

Installing this chart using the provided makefile is quite simple assuming you have helm installed both locally and in your cluster, and your current kubectl context set to point to that cluster. Running make releasefrom the root directory will run the chart through helm in dry-run mode and display the resulting yaml output. To actually install the chart into your cluster run make apply release. Note that these commands will pull the latest image from my docker repository. To build and push the image to your own repo see the [readme](https://github.com/Markbnj/kunbound/blob/master/README.md) for more makefile options. You can verify that the chart is installed with helm ls and that the pods are running with kubectl get pods | grep kunbound. Unfortunately the pods don’t do anything very useful yet, since the default unbound configuration installed with the chart doesn’t include any forward zones or resolved addresses.

Forward zones and upstream resolvers are defined in the /etc/unbound.conf file. The kunbound container image includes a default configuration that passes all queries to Google’s public DNS servers for testing purposes, however that file is replaced by the helm upgrade command when the kunbound/templates/configmap.yaml file is processed. If you’re not familiar with how to mount a configmap into a container as a file [see the kubernetes documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#add-configmap-data-to-a-volume). In order to get your forward zones and resolvers populated in that file you just need to present them as yaml values to the helm upgrade command. Here is an example of the correct structure:

    forwardZones:
    - name: "fake.net"  
      forwardHosts:  
      - "fake1.host.net"  
      - "fake2.host.net"
    - name: "stillfake.net"  
      forwardIps:  
      - "10.10.10.10"  
      - "10.11.10.10"

Note that you can have as many forward zones as needed, and you can specify the addresses of the upstream resolvers using either DNS names or IPs. Once you have your definitions save them to a file. We’ll look at how to pass it to the build below.

Before that there is one other thing that needs to be configured in order to make the chart work, and that is access control. The default configuration in the container image binds to all interfaces and allows queries from all hosts for testing purposes. However when the chart is installed into your cluster the default is to bind to localhost and only allow queries from 127.0.0.1/32. In order for unbound to serve as an upstream for kube-dns it needs to allow queries from pods in the cluster. This is done by setting the *clusterIpv4Cidr *property in the values to the CIDR range of the cluster’s pod network. You unfortunately can’t get this property from kubectl so you’ll have to resort to whatever command is appropriate in your environment. On Google Cloud Platform, for example:

    $ gcloud container clusters describe clustername
    clusterIpv4Cidr: 10.0.0.0/14

To upgrade the chart so that it has both the forwarding zones and the cluster CIDR for access control run the make command and pass the values on the command line as shown:

    make apply release \
    VALUES=yourzones.yaml \
    CLUSTER_IP4_CIDR=10.0.0.0/14

If you ran the earlier install command then running this one will upgrade the release, rather than installing it. Either way the forward zones and CIDR range will get rendered into the configmap and the pods will either be created or restarted with that data mounted in /etc/unbound.conf.

There’s one last thing to do. Recall the kube-dns configmap mentioned at the top of this post? We need to update it to point to our new DNS resolver service. First, extract the current configmap so that we can edit it and apply the changes:

    $ kubectl get configmap kube-dns -n kube-system -oyaml > config.yaml

Next get the cluster IP of the service created when you installed the chart:

    $ kubectl get svc | grep kunbound | awk '{print $2}'

Now edit config.yaml from the first command to set the stubDomains. For example if your zone is myzone.net and the cluster IP from the command above was 10.3.200.190 then:

    data:
    ** **stubDomains: |
     {“myzone.net”: [“10.3.200.190”]}

Save the file and apply the changed version in the cluster:

    kubectl apply -f config.yaml

That’s pretty much it. There is more information in the [repo readme](https://github.com/Markbnj/kunbound/blob/master/README.md) about running the makefile in different ways. I’ve also provided “raw” yaml in the yaml/ directory if you don’t want to use helm, although in that case you’ll have to edit the files manually to insert the correct values. There is also a lot more to unbound, which you can discover by digging into the [documentation](http://www.unbound.net/documentation/index.html). Happy helming!
