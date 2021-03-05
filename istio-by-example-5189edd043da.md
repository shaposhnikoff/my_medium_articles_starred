
# Istio by Example

A basic introduction to service meshes with Istio.

![](https://cdn-images-1.medium.com/max/2000/1*8TBsg2mPp8SxsguQNYVhNg.jpeg)

## Introduction

What is a service mesh?
> For all the hype, the service mesh is architecturally pretty straightforward. It’s nothing more than a bunch of userspace proxies, stuck “next” to your services (we’ll talk about what “next” means in a bit), plus a set of management processes. The proxies are referred to as the service mesh’s data plane, and the management processes as its control plane. The data plane intercepts calls between services and “does stuff” with these calls; the control plane coordinates the behavior of the proxies, and provides an API for you, the operator, to manipulate and measure the mesh as a whole.

*— Bouyant — [The Service Mesh: What Every Software Engineer Needs to Know about the World’s Most Over-Hyped Technology](https://buoyant.io/service-mesh-manifesto/)*

What about Istio?
> Istio makes it easy to create a network of deployed services with load balancing, service-to-service authentication, monitoring, and more, with few or no code changes in service code. You add Istio support to services by deploying a special sidecar proxy throughout your environment that intercepts all network communication between microservices, then configure and manage Istio using its control plane functionality…

*— Istio — [What is Istio?](https://istio.io/latest/docs/concepts/what-is-istio/)*

Istio features are broadly categorized into [traffic management](https://istio.io/latest/docs/concepts/traffic-management/), [security](https://istio.io/latest/docs/concepts/security/), and [observability](https://istio.io/latest/docs/concepts/observability/). In this article, we will focus exclusively on the traffic management feature.
> Istio’s traffic routing rules let you easily control the flow of traffic and API calls between services. Istio simplifies configuration of service-level properties like circuit breakers, timeouts, and retries, and makes it easy to set up important tasks like A/B testing, canary rollouts, and staged rollouts with percentage-based traffic splits. It also provides out-of-box failure recovery features that help make your application more robust against failures of dependent services or the network.

*— Istio — [Traffic Management](https://istio.io/latest/docs/concepts/traffic-management/)*

The traffic management feature itself, however, is fairly complicated. In this article we will focus exclusively on its request routing aspect. In particular, we will use it to deploy a canary release.
> Canary release is a technique to reduce the risk of introducing a new software version in production by slowly rolling out the change to a small subset of users before rolling it out to the entire infrastructure and making it available to everybody

*— martinFowler.com — [CanaryRelease](https://martinfowler.com/bliki/CanaryRelease.html)*

## Prerequisites

In order to follow along, one will need to have installed:

* a Kubernetes cluster; see [*Platform Setup](https://istio.io/latest/docs/setup/platform-setup/)* for details

* [*kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)* CLI

* [Istio](https://istio.io/latest/docs/setup/install/)

## A Canary Release

Before we introduce the Istio resources, let us first examine the standard Kubernetes resources in this example:

* [*namespace-1](https://github.com/larkintuckerllc/hello-istio/blob/master/traffic-management/request-routing/00-namespace-1-namespace.yaml)* Namespace: The Namespace for the resources in this example. Includes the *istio-injection: enabled* label to [automatically inject the Istio sidecar proxy](https://istio.io/latest/docs/setup/additional-setup/sidecar-injection/#automatic-sidecar-injection)

* [*app-1](https://github.com/larkintuckerllc/hello-istio/blob/master/traffic-management/request-routing/app-1-00-service.yaml)* Service: A Service with fully qualified domain name (FQDN) name *app-1.namespace-1.svc.cluster.local *exposing port *80; *load balancing pods labeled with *app: app-1*

* [*app-1-v1](https://github.com/larkintuckerllc/hello-istio/blob/master/traffic-management/request-routing/app-1-v1/app-1-v1-01-configmap.yaml)* ConfigMap: A ConfigMap providing the *index.html* file for the production release

* [*app-1-v1](https://github.com/larkintuckerllc/hello-istio/blob/master/traffic-management/request-routing/app-1-v1/app-1-v1-02-deployment.yaml)* Deployment: A Deployment managing the pods, labeled with *app: app-1* and *version: v*1, for the production release. Defaults to one replica

* [*app-1-v1](https://github.com/larkintuckerllc/hello-istio/blob/master/traffic-management/request-routing/app-1-v1/app-1-v1-03-hpa.yaml)* HorizontalPodAutoscaler: A HorizontalPodAutoscaler managing the number of replicas for the production release

* [*app-1-v2](https://github.com/larkintuckerllc/hello-istio/blob/master/traffic-management/request-routing/app-1-v2/app-1-v2-01-configmap.yaml)* ConfigMap: Same as *app-1-v1 *ConfigMap* *but for the canary release

* [*app-1-v2* ](https://github.com/larkintuckerllc/hello-istio/blob/master/traffic-management/request-routing/app-1-v2/app-1-v2-02-deployment.yaml)Deployment: Same as *app-1-v1* Deployment but for the canary release; labeled with *app: app-1* and *version: v2*

* [*app-1-v2](https://github.com/larkintuckerllc/hello-istio/blob/master/traffic-management/request-routing/app-1-v2/app-1-v2-03-hpa.yaml)* HorizontalPodAutoscaler: Same as *app-1-v1 *HorizontalPodAutoscaler* *but for the canary release

Things to observe:

* Without Istio, a canary release would typically only consist of a deployment with a fixed number of replicas, i.e., without a HorizontalPodAutoscaler

There is a fundamental problem that Istio addresses in this example:
> Whether we use one deployment or two, canary management using deployment features of container orchestration platforms like Docker, Mesos/Marathon, or Kubernetes has a fundamental problem: the use of instance scaling to manage the traffic; traffic version distribution and replica deployment are not independent in these systems. All replica pods, regardless of version, are treated the same in the kube-proxy round-robin pool, so the only way to manage the amount of traffic that a particular version receives is by controlling the replica ratio. Maintaining canary traffic at small percentages requires many replicas (e.g., 1% would require a minimum of 100 replicas).

*— Istio — [Canary Deployments Using Istio](https://istio.io/latest/blog/2017/0.1-canary/)*

We will use two Istio resources in this example; the first being a Destination Rule:
> Along with virtual services, destination rules are a key part of Istio’s traffic routing functionality. You can think of virtual services as how you route your traffic to a given destination, and then you use destination rules to configure what happens to traffic for that destination. Destination rules are applied after virtual service routing rules are evaluated, so they apply to the traffic’s “real” destination.
> In particular, you use destination rules to specify named service subsets, such as grouping all a given service’s instances by version. You can then use these service subsets in the routing rules of virtual services to control the traffic to different instances of your services.

*— Istio — [Traffic Management (Destination Rules)](https://istio.io/latest/docs/concepts/traffic-management/#destination-rules)*

In this example, we have the [*app-1](https://github.com/larkintuckerllc/hello-istio/blob/master/traffic-management/request-routing/app-1-01-destinationrule.yaml)* DestinationRule.

<iframe src="https://medium.com/media/3623a5b849ea84291d1a2023df13c4bc" frameborder=0></iframe>

Things to observe:

* Here *host* refers to the name of a Kubernetes service; in this case *app-1 *which* *identifies pods labeled with *app: app-1*

* The subsets, *v1* and *v2*, differentiate pods labeled with *version: v1* and *version: v2* respectively

The other Istio resource is a VirtualService:
> Virtual services, along with destination rules, are the key building blocks of Istio’s traffic routing functionality. A virtual service lets you configure how requests are routed to a service within an Istio service mesh, building on the basic connectivity and discovery provided by Istio and your platform. Each virtual service consists of a set of routing rules that are evaluated in order, letting Istio match each given request to the virtual service to a specific real destination within the mesh. Your mesh can require multiple virtual services or none depending on your use case.

*— Istio — [Traffic Management (Virtual Services)](https://istio.io/latest/docs/concepts/traffic-management/#virtual-services)*

In this example, we have the [*app-1](https://github.com/larkintuckerllc/hello-istio/blob/master/traffic-management/request-routing/app-1-02-virtualservice.yaml)* VirtualService.

<iframe src="https://medium.com/media/11cc46f7c397c7a895ac7c0a9e1ccb01" frameborder=0></iframe>

Things to observe:

* Here the (first) host *app-1* implicitly refers to the FQDN of *app-1.namespace-1.svc.cluster.local. *With this in place, any http (i.e., HTTP/1.1, HTTP2, and gRPC) traffic destined to this name is routed by this VirtualService

* The host *app-1* in both *destination* blocks refer to the DestinationRule; 90% of the traffic is to be sent to its subset named *v1* and 10% to the one named *v2*

From the project’s root folder, we execute the following to create the aforementioned resources:

    **$ kubectl apply --recursive -f traffic-management/request-routing/**
    namespace/namespace-1 created
    service/app-1 created
    destinationrule.networking.istio.io/app-1 created
    virtualservice.networking.istio.io/app-1 created
    configmap/app-1-v1 created
    deployment.apps/app-1-v1 created
    horizontalpodautoscaler.autoscaling/app-1-v1 created
    configmap/app-1-v2 created
    deployment.apps/app-1-v2 created
    horizontalpodautoscaler.autoscaling/app-1-v2 created

One interesting observation is that because we included the *istio-injection: enabled *label on the namespace, the pods have a second container (the Istio sidecar proxy).

    **$ kubectl get pods -n namespace-1**
    NAME                        READY   STATUS    RESTARTS   AGE
    app-1-v1-69749dd8c8-sf9dq   2/2     Running   0          2m24s
    app-1-v2-58f99f44bb-8xjsg   2/2     Running   0          2m23s

Here we inspect the impact of the DestinationRule and VirtualService on the Service by creating a [*debug](https://github.com/larkintuckerllc/hello-istio/blob/master/debug-pod.yaml)* pod; executing:

    **$ kubectl apply -f debug-pod.yaml **
    pod/debug created

After [getting a shell in the container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/) of the *debug* pod and installing the apt packages *dnsutils* and *curl*, we can first confirm that DNS works as normal; here we lookup the *app-1.namespace-1.svc.cluster.local* name and get the IP address of the Kubernetes *app-1* Service (resolved by the Kubernetes DNS Service).

    **# dig app-1.namespace-1.svc.cluster.local**

    ; <<>> DiG 9.16.1-Ubuntu <<>> app-1.namespace-1.svc.cluster.local
    ;; global options: +cmd
    ;; Got answer:
    ;; WARNING: .local is reserved for Multicast DNS
    ;; You are currently testing what happens when an mDNS query is leaked to DNS
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16175
    ;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

    ;; QUESTION SECTION:
    ;app-1.namespace-1.svc.cluster.local. IN A

    ;; ANSWER SECTION:
    app-1.namespace-1.svc.cluster.local. 30 IN A 10.8.6.202

    ;; Query time: 3 msec
    ;; SERVER: 10.8.0.10#53(10.8.0.10)
    ;; WHEN: Sat Feb 20 17:03:47 EST 2021
    ;; MSG SIZE  rcvd: 69

We can browse the *app-1.namespace-1.svc.cluster.local *name and get a response from one of the two deployments; here from *app-1-v1*:

    **# curl app-1.namespace-1.svc.cluster.local**
    <html>
    <body>
    Hello World v1!
    </body>
    </html>

Things to observe:

* Because of how pods resolves DNS names, with search suffixes, we could have also simply used *app-1* as the name

While the traffic between the *debug* pod and the pods managed by the two deployments is being intercepted by Istio sidecar proxies on both ends and with the Istio resources in place, it is not immediately obvious that anything is different than interacting with the normal Kubernetes Service.

To observe the effect of the Istio sidecar proxies and Istio resources on the Service, we create a [*load](https://github.com/larkintuckerllc/hello-istio/blob/master/load-job.yaml)* job that browses the *app-1* name *200* times a second for five minutes.

    **$ kubectl apply -f load-job.yaml**
    job.batch/load created

After about a minute, we can observe the two HorizontalPodAutoscalers:

    **$ kubectl get hpa -n namespace-1**
    NAME       REFERENCE             TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
    app-1-v1   Deployment/app-1-v1   78%/80%   1         5         2          91m
    app-1-v2   Deployment/app-1-v2   21%/80%   1         5         1          91m

Things to observe:

* Here we can see the production release is getting substantially more traffic than the canary release; so much so, the production release HPA scaled up the deployment

## Wrap Up

This is just a taste of the power of service meshes and in particular Istio. Hopefully this will get you on the right foot for your future learning.
