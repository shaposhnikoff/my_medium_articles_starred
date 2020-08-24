Unknown markup type 10 { type: [33m10[39m, start: [33m279[39m, end: [33m284[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m9[39m, end: [33m34[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m99[39m, end: [33m140[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m244[39m, end: [33m259[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m3[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m87[39m, end: [33m93[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m7[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m65[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m175[39m, end: [33m183[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m199[39m, end: [33m210[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m19[39m, end: [33m51[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m58[39m, end: [33m74[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m22[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m35[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m109[39m, end: [33m122[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m93[39m, end: [33m129[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m41[39m, end: [33m50[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m70[39m, end: [33m77[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m230[39m, end: [33m246[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m251[39m, end: [33m260[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m315[39m, end: [33m324[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m360[39m, end: [33m372[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m68[39m, end: [33m77[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m233[39m, end: [33m246[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m104[39m, end: [33m111[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m33[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m96[39m, end: [33m112[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m123[39m, end: [33m132[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m133[39m, end: [33m142[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m46[39m, end: [33m62[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m197[39m, end: [33m203[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m226[39m, end: [33m237[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m96[39m, end: [33m109[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m174[39m, end: [33m187[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m46[39m, end: [33m71[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m93[39m, end: [33m134[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m11[39m, end: [33m47[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m54[39m, end: [33m58[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m53[39m, end: [33m62[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m66[39m, end: [33m73[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m159[39m, end: [33m168[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m231[39m, end: [33m250[39m }

# Configuring your Linux host to resolve a local Kubernetes cluster’s service URLs

I do all my development in a Linux VM. I also run my test Kubernetes cluster there. Sometimes I need to resolve the DNS names of Kubernetes services. For example, I might need to curl https://kubernetes.default.svc.cluster.local, but this doesn’t work out of the box because I’m running curl in my VM, and it doesn’t know anything about the local cluster’s DNS server.

![Photo by [John Carlisle](https://unsplash.com/photos/l090uFWoPaI?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/10368/1*NaL3SxOBXbrwXrAciJyM3Q.jpeg)*Photo by [John Carlisle](https://unsplash.com/photos/l090uFWoPaI?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

A solution to this problem was critical for testing changes I make as I’m developing [Heptio Ark](https://github.com/heptio/ark). I frequently want to check the logs for my backups and restores after they’ve completed. In my development setup, these logs are stored in a[ Minio](https://minio.io/) pod running in the cluster, with a minio service sitting in front of it.

If I run ark backup logs my-backup, the Ark server returns a pre-signed URL for the log file under minio.heptio-ark-server.svc.cluster.local, and then the client tries to retrieve it. This fails, again because my VM doesn’t know anything about *.cluster.local.

Fortunately, I found a solution: [dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html).

dnsmasq makes it simple to specify the nameserver to use for a given domain. My VM runs Fedora, which uses NetworkManager, so configuring looks like this:

In /etc/NetworkManager/NetworkManager.conf, add or uncomment the following line in the [main] section:

    dns=dnsmasq

Create /etc/NetworkManager/dnsmasq.d/kube.conf with this line:

    server=/cluster.local/10.96.0.10

This tells dnsmasq that queries for anything in the cluster.local domain should be forwarded to the DNS server at 10.96.0.10. This happens to be the default IP address of the kube-dns service in the kube-system namespace. If your cluster’s DNS service has a different IP address, you’ll need to specify it instead.

Now, after you run systemctl restart NetworkManager, your /etc/resolv.conf should look something like this:

    # Generated by NetworkManager
    search localdomain
    nameserver 127.0.0.1

The important line is nameserver 127.0.0.1. This means DNS requests are sent to localhost, which is handled by dnsmasq.

I installed my local cluster using kubeadm, which installs a DNS service to run on the cluster to handle the cluster.local zone. (As of this writing, the default is[ kube-dns](https://github.com/kubernetes/dns)). After I made the NetworkManager changes and started my cluster, I checked to see if my new dnsmasq settings helped.

Unfortunately, this didn’t work exactly the way I expected. Attempts from my host to resolve kubernetes.default.svc.cluster.local stalled for a long time, but they eventually came back with the correct IP address. I saw the same behavior with DNS resolution for cluster DNS entries *inside* pods. Even worse, resolution of external DNS entries failed entirely (also after a long delay):

    $ kubectl run --rm --attach --restart Never --image busybox bbox -- nslookup google.com
    If you don't see a command prompt, try pressing enter.
    Server:    10.96.0.10
    Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

    nslookup: can't resolve 'google.com'
    pod default/bbox terminated (Error)

It turns out the DNS deployment sets the dnsPolicy for the DNS pod to Default. Somewhat confusingly, this means that the pod uses the same name resolution configuration as the kubelet. In other words, the pod uses the contents of /etc/resolv.conf, so 127.0.0.1 is its nameserver. (Note that this is not the default dnsPolicy for pods generally; the default is ClusterFirst. For more information, see [the documentation](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#pods-dns-policy).)

But wait! That means the DNS pod is going to try to talk to itself (127.0.0.1) to resolve DNS lookups — that’s not what we want! We get infinite recursion trying to do external DNS lookups, meaning they all fail. Attempts to resolve cluster.local names do end up working, but it takes a long time.

I started searching for a solution, and ultimately decided to try swapping the default DNS provider for kubeadm, kube-dns, with its likely eventual successor,[ CoreDNS](https://github.com/coredns/coredns). CoreDNS is highly configurable, and includes a feature for resolving Kubernetes services.

After I switched to CoreDNS, things still weren’t quite right — lookups still either took forever or didn’t work at all. Fortunately, CoreDNS is configured using a ConfigMap, so I looked at that first:

    $ kubectl -n kube-system describe configmap/coredns
    Name:         coredns
    Namespace:    kube-system
    Labels:       <none>
    Annotations:  <none>

    Data
    ====
    Corefile:
    ----
    .:53 {
        errors
        log
        health
        kubernetes cluster.local 10.96.0.0/12 {
           pods insecure
        }
        prometheus
        proxy . /etc/resolv.conf
        cache 30
    }

The line causing my issue is

    proxy . /etc/resolv.conf

This makes CoreDNS proxy all non-cluster.local DNS requests to whatever nameserver is listed in /etc/resolv.conf, which is 127.0.0.1 as shown above.

This is the same problem I initially faced with kube-dns. It leads to infinite recursion because CoreDNS *is* handling DNS requests on 127.0.0.1, so, it ends up proxying requests to itself. We need to find a way to proxy to a different resolver for external DNS requests.

To fix this, I edited the ConfigMap, changing /etc/resolv.conf to the IP address of my upstream resolver. Because I use Parallels Desktop as my VM hypervisor, I specify the first IP address in its Shared network’s DHCP range, 10.211.55.1. The updated ConfigMap now looks like this:

    Corefile:
    ----
    .:53 {
        errors
        log
        health
        kubernetes cluster.local 10.96.0.0/12 {
           pods insecure
        }
        prometheus
        proxy . 10.211.55.1:53
        cache 30
    }

I restarted the CoreDNS pod, and my DNS requests immediately started working! I can now resolve cluster.local names from my host. Inside pods, DNS requests for both internal cluster.local names and external names work as expected.

    $ kubectl run --rm -it --restart Never --image busybox bbox -- nslookup minio.heptio-ark-server.svc.cluster.local
    Server:    10.96.0.10
    Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

    Name:      minio.heptio-ark-server.svc.cluster.local
    Address 1: 10.100.88.74 minio.heptio-ark-server.svc.cluster.local

    $ kubectl run --rm --attach --restart Never --image busybox bbox -- nslookup google.com
    Server:    10.96.0.10
    Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local
    Name:      google.com
    Address 1: 2607:f8b0:4004:811::200e iad30s21-in-x0e.1e100.net
    Address 2: 172.217.13.78 iad23s60-in-f14.1e100.net

Circling back to my Ark needs, now when I run ark backup logs my-backup, my host can resolve minio.heptio-ark-server.svc.cluster.local and retrieve the logs successfully.

    $ ark backup logs b1| head -n6
    time="2018-02-26T16:10:50Z" level=info msg="Starting backup" backup=heptio-ark/b1 logSource="pkg/backup/backup.go:210"
    time="2018-02-26T16:10:50Z" level=info msg="Including namespaces: default" backup=heptio-ark/b1 logSource="pkg/backup/backup.go:213"
    time="2018-02-26T16:10:50Z" level=info msg="Excluding namespaces: *" backup=heptio-ark/b1 logSource="pkg/backup/backup.go:214"
    time="2018-02-26T16:10:50Z" level=info msg="Including resources: *" backup=heptio-ark/b1 logSource="pkg/backup/backup.go:217"
    time="2018-02-26T16:10:50Z" level=info msg="Excluding resources: *" backup=heptio-ark/b1 logSource="pkg/backup/backup.go:218"
    time="2018-02-26T16:10:50Z" level=info msg="Backing up group" backup=heptio-ark/b1 group=apiregistration.k8s.io/v1beta1 logSource="pkg/backup/group_backupper.go:135"

Talking to kubernetes.default.svc.cluster.local using curl also works:

    $ curl --silent --cacert /etc/kubernetes/pki/ca.crt https://kubernetes.default.svc.cluster.local/apis | jq -r .groups[].name
    apiregistration.k8s.io
    extensions
    apps
    events.k8s.io
    authentication.k8s.io
    authorization.k8s.io
    autoscaling
    batch
    certificates.k8s.io
    networking.k8s.io
    policy
    rbac.authorization.k8s.io
    storage.k8s.io
    admissionregistration.k8s.io
    apiextensions.k8s.io
    ark.heptio.com

One minor note on this setup: any other pods using a dnsPolicy of Default with standard pod networking won’t have functional DNS. They’ll send DNS requests to 127.0.0.1, but unless the pod has its own DNS server, the requests will fail. Pods with host networking, on the other hand, will work.

One final note on kube-dns: I learned after the fact that it does appear possible to achieve the same thing I did with CoreDNS. Just like CoreDNS, kube-dns uses a[ ConfigMap](https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/#example-upstream-nameserver) for its configuration. While I didn’t try it, setting the[ upstreamNameservers](https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/#example-upstream-nameserver) to list to my upstream resolver should work similarly to the change I made to the CoreDNS ConfigMap.

And that’s it! If you have any related DNS tricks, I’d love to hear them.

Learn more about what Heptio at [www.heptio.com](http://www.heptio.com).

p.s. If you’re on a Mac and running Minikube, check out Steve Sloka’s [post](https://stevesloka.com/2017/05/19/access-minikube-services-from-host/) on how to access Kubernetes services using DNS from your Mac.
