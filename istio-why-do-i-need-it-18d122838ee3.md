
# Istio Why do I need it?

I’ve not had much time recently to play around with k8s lately and admittedly lost the plot re exactly what Istio would really add to the K8s party. Does it add more operational overhead just because? Does it simplify some things you normally need to do anyway? These were the questions bubbling around in my head.

( I suspect after publishing this a load of my teammates who are deeper into K8s than I am will be having conversations with me and my opinions on this ….. Those are my favourite conversations though debates with my teammates )

So what is Istio anyway?

From the [Istio site](http://istio.io) it says:

Istio gives you:

* Automatic load balancing for HTTP, gRPC, WebSocket, and TCP traffic.

* Fine-grained control of traffic behavior with rich routing rules, retries, failovers, and fault injection.

* A pluggable policy layer and configuration API supporting access controls, rate limits and quotas.

* Automatic metrics, logs, and traces for all traffic within a cluster, including cluster ingress and egress.

* Secure service-to-service authentication with strong identity assertions between services in a cluster.

You add Istio support to services by deploying a special sidecar proxy ( a helper container) throughout your environment ( There’s a reason it’s not just clusters which I was pretty impressed with, see later if you make it that far ) . Once the sidecar proxy is installed all network communication between the (micro)services is via this proxy. In addition all Network communication is then configured and managed using Istio’s control plane functionality.

Istio is a **service mesh**. The definition I use when thinking about service mesh’s is this one “A dedicated infrastructure layer for making service-to-service communication safe, performant and reliable”

However if like me you started at the [concept docs](https://istio.io/docs/concepts/what-is-istio/overview.html) which has this : “The term **service mesh** is often used to describe the network of microservices that make up such applications and the interactions between them. As a service mesh grows in size and complexity, it can become harder to understand and manage. Its requirements can include discovery, load balancing, failure recovery, metrics, and monitoring, and often more complex operational requirements such as A/B testing, canary releases, rate limiting, access control, and end-to-end authentication. Istio provides a complete solution to satisfy the diverse requirements of microservice applications by providing behavioral insights and operational control over the service mesh as a whole. “

You may have been confused like I was after reading that! I ended up getting diverted and explored the interwebs re what the general consensus was with regards what the heck a service mesh really is . It seems that the definition I use is by general consensus from my non representative sample a reasonable one to pick . But what was missing was the detail to separate it from an orchestration tool like k8s. Istio is used in conjunction with K8s and would not be a thing without k8s or some other container orchestration tooling. It does NOT do orchestration and actually seems to be designed to address the networking and operational complexity of managing large microservice based solutions. So an overlay that overlays something like k8s then! …. Now I really do need to get on with the post …

So I know what Istio is and what it gives you but what operational challenges is it actually addressing?

From the [why use Istio page](https://istio.io/docs/concepts/what-is-istio/overview.html) its states it provides a number of key capabilities uniformly across a network of services:

* Traffic management

* Observability

* Policy enforcement

* Service identity and security

For me to really understand the value of Istio it was hands on time so I used [this great codelab ](https://codelabs.developers.google.com/codelabs/cloud-hello-istio/#0). I figure who ever wrote the code lab was a neighbour’s fan. I’m not judging though!

The code lab introduced me to the four primary Istio control plane components:

* *Pilot*: Handles configuration and programming of the proxy sidecars.

* *Mixer*: Handles policy decisions for your traffic and gathers telemetry.

* *Ingress*: Handles incoming requests from outside your cluster.

* *CA*: the Certificate Authority.

Have a look at the [Istio architecture concepts](https://istio.io/docs/concepts/what-is-istio/overview.html#architecture) page to understand how these components hang together

The code lab gave me hands on with [route rules](https://istio.io/docs/concepts/traffic-management/rules-configuration.html#route-rules) — the traffic mgt part

I also tried out some of the various tasks from[ Istio.io](https://istio.io/docs/tasks/) as I needed to understand how it dealt with the other areas it was designed to deal with.

Tip: keep your cluster with the app up and running if you too decide to wander off piste when you’re done with the codelab. You’ll only end up using it again anyway .

So I had a rudimentary understanding of how it addressed those areas but if I use a managed k8s set up like GKE ( well you knew I’d pick that didn’t you?) how does Istio fit in if at all?

*Note: Yes there is more detail than I cover here but I focus on the aspects that I felt helped me decide why I would need to use Istio*

**Cluster end user/ developer access**

GKE uses a combination of [IAM](https://cloud.google.com/kubernetes-engine/docs/how-to/iam-integration) and [RBAC](https://cloud.google.com/kubernetes-engine/docs/how-to/role-based-access-control) , and yes there’s a lot to get your head around.

To grant more granulated permissions to users of your cluster than granted by Cloud IAM you use name spaces and RBAC for example to restrict access to specific pods or to exclude access to secrets.

[Istio RBAC](https://istio.io/docs/concepts/security/rbac.html) introduces two roles focused on services

* **ServiceRole** defines a role for access to services in the mesh.

* **ServiceRoleBinding** grants a role to subjects (e.g., a user, a group, a service).

They are K8s CustomResourceDefinition (CRD) objects. And IAM is still a thing you need to understand.

### Service identity

GKE can use service accounts to manage what GCP services the [applications running on GKE ](https://cloud.google.com/kubernetes-engine/docs/tutorials/authenticating-to-cloud-platform)can use . The keys for these service accounts are stored as secrets. The identity for processes that run in a pod are [k8s service accounts](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/) ( named just for laughs & giggles I feel!) together with RBAC . Istio uses[ istio-auth](https://istio.io/docs/concepts/security/mutual-tls.html) which provides strong service-to-service and end-user authentication using mutual TLS, with built-in identity and credential management. Istio-auth uses k8s service accounts

The docs do a really good job of explaining how it works so I’m just going to copy a teeny image of the architecture diagram here . Go [read the words ](https://istio.io/docs/concepts/security/mutual-tls.html)that go with it :-)

![teeny image of istio-auth architecture](https://cdn-images-1.medium.com/max/2000/0*gT-DsHMtK0VNInJq.)*teeny image of istio-auth architecture*

Itsio doesn’t provide anything to help with using GCP service accounts. It’s very early days but it’s on the roadmap looking at the plans for future development.

Istio-auth is nice and the planned enhancements are going to be worth waiting for. I get antsy about security complexity as that inevitably leads to errors in configuration so I am hoping for a more seamless alignment between service account types!

### Network controls

GKE ( for k8s version 1.7.6 +) uses [k8s network policies](https://cloud.google.com/kubernetes-engine/docs/how-to/network-policy) to manage which pods and services can communicate . This is relatively straightforward to configure. Istio also has network policies but they’re not the k8s policies you know and love so what’s the difference and why both? [This post](https://istio.io/blog/2017/0.1-using-network-policy.html) explains it well so I won’t repeat the reasoning here but this table summarises the differences and why you need both

![](https://cdn-images-1.medium.com/max/2000/0*OefOZPx4kbxfcWK2.)

Istio uses envoys as a sidecar proxy . Envoy operates at the application layer of the OSI model so at layer 7 .. I’m paraphrasing the blog explains it in detail for you.

You need both policy types it’s the defence in depth approach

### Multiple clusters

What Istio does have which imho is petty cool are [mixer adapters](https://istio.io/docs/concepts/policy-and-control/mixer.html#adapters) . In a nutshell they abstract from the underlying backend to deliver core functionality, such as logging, monitoring, quotas, ACL checking, and more. It exposes a single consistent API, independent of the backends in use. Just like GCS exposes a single API regardless of what storage class you use!

I think this image from the blog post on [mixer adapter model](https://istio.io/blog/2017/adapter-model.html) captures in one image the whole point of mixer adapters

![](https://cdn-images-1.medium.com/max/2000/0*Bdd76HtJB1suin0y.)

There’s an [early demo](https://istio.io/docs/guides/integrating-vms.html) of what I think is one of the most useful feature of istio’s potential which actually uses a VM to host a MySQL dbase for the rating dbase used in the codelab and incorporating that as part of the mesh that the GKE cluster is a member of. One mesh to rule them all ( I know I can’t help myself!)

### Traffic management

If you did the codelab you will have seen how easy it was to use istio to direct traffic using route rules . With K8s you can also do traffic management using canary deployments and direct a proportion of traffic to one version of your app in a similar manner as istio but Istio is way more flexible in this case by allowing you to set fine grained traffic percentages and controlling traffic using other criteria as in the code lab.

### Service discovery

Service registration is done at k8s. Istio abstracts the underlying service discovery mechanisms and conforms them into a standard format consumable by the envoy sidecars.

### Audit Logging & monitoring

Above and beyond the standard logging that GKE provides GKE can integrate with [StackDriver Logging](https://cloud.google.com/kubernetes-engine/docs/how-to/logging) to collect, process, and store in a persistent store *Container logs* , *System logs* and *Events* about activity in the cluster, such as the scheduling of Pods. You can also integrate with [StackDriver Monitoring](https://cloud.google.com/kubernetes-engine/docs/how-to/monitoring) to collect System metrics ( measurements of the cluster’s infrastructure, such as CPU or memory usage) and Custom metrics ( application-specific metrics) . Istio

Istio makes use of prometheus for logging & monitoring together with grafana as the dashboard. I love the [service graph configuration](https://istio.io/docs/tasks/telemetry/servicegraph.html) which gives you a graphical representation of your service mesh. You can also use Elasticsearch with kibana & fluentd

### So do I need Istio?

Istio’s traffic mgt is really nice and the mixer adapter model makes it easy to manage a mesh that covers multiple clusters and VM’s . I like that Istio gets you to think about services and not so much pods and nodes, Not saying you don’t have to worry about those but talking about services just feels right!

If you need to manage a distributed cluster then Istio should be on your shopping list. If you need more flexibility with traffic mgt than what k8s can provide again Istio would be worth looking at.

If you have the resources to play with something that is in a state of early development then understanding Istio early is worth it. The learning curve is low if you already use k8s in anger.

Remember it’s an overlay so you still need to do things at the k8s layer such as configuring k8s network policies to complement the istio network policies.

It’s early days so it doesn’t do all the things you’d expect/ hope it would yet . You can’t avoid flipping between the provider API’s and Istio to get a complete job done so it’s not the one pane of glass you’d hope for.

A dashboard would be nice to visualize the mesh configuration as YAML hacking will get tired fast! Yes you can set up dashboards to visualise metrics using your dashboard app of choice but I would like to see it integrated with StackDriver

So after learning a little about Istio overall I actually like what it promises .
