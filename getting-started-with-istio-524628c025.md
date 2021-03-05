
# Getting Started With Istio

Getting Started With Istio

### Understanding what a service mesh is and how it can be used effectively in a microservices architecture.

The last few years have brought about immense changes in the software architecture landscape. A major shift that we have all witnessed is the breakdown of large monolithic and coarse-grained applications into fine-grained deployment units called *microservices*, communicating predominantly by way of synchronous REST and gRPC interfaces, as well as asynchronous events and message passing. The benefits of this architecture are numerous, but the drawbacks are equally evident. Aspects of software development that used to be straightforward in the ‘old world’, such as debugging, profiling and performance management, are now an order of magnitude more complex. Also, a microservices architecture brings its own unique challenges. Services are more fluid and elastic, and tracking of their instances, their versions and dependencies is a Herculean challenge that balloons in complexity as the service landscape evolves. To top this off, services will fail in isolation, further exacerbated by unreliable networks. Given a large enough system, parts of it may be suffering a minor outage at any given point in time, potentially impacting a subset of users, quite often without the operator’s awareness. With so many ‘moving parts’, how does one stay on top of these challenges and ensure the system is running smoothly without impacting the customer and driving the developers out of their wits?

With the increased adoption of microservices, the industry has been steadily coming up with patterns and best-practices that have made the entire experience more palatable. *Resiliency Patterns*, *Service Discovery*, *Container Orchestration*, *Canary Releases*, *Observability Patterns*, *BFF*, *API Gateway*… These are some of the concepts that practitioners will employ to build more robust and sustainable distributed systems. But these concepts are just that — abstract notions and patterns — they require *someone* to implement them *somewhere* in the system. More often than not, that ‘someone’ is *you* and ‘somewhere’ is *everywhere*.

## Before we begin

This article intends to offer a ‘gentle’ introduction to Istio. By ‘gentle’, we don’t mean quick, dumbed-down or crammed. We shall start with the fundamental concepts, then take a tour showing how they come together with incremental examples. Expect a long read. Put the kettle on. Try to follow the examples in order, as some will depend on the previous ones. By the end of it, you should understand what Istio is, where it can be used, and have the confidence to drive it yourself.

The material presented in this article would be classed as intermediate to advanced on the Kubernetes knowledge spectrum. A strong background in Kubernetes and Docker is essential: you should have a thorough hands-on understanding of core Kubernetes concepts, ability to deploy and manage containerized workloads, and comfortably use kubectl to navigate and alter the state of your Kubernetes cluster.

## Introducing Istio

Istio is a service mesh — an application-aware infrastructure layer for facilitating service-to-service communications. By ‘application-aware’, it is meant that the service mesh understands, to some degree, the nature of service communications and can intervene in a value-added manner. For example, a service mesh can implement resiliency patterns (retries, circuit breakers), alter the traffic flow (shape the traffic, affect routing behavior, facilitate canary releases), as well as add a whole host of comprehensive security controls. Being intrinsically aware of the traffic passing between services, Istio can also provide fine-grained instrumentation and telemetry insights, providing a degree of observability to an otherwise opaque distributed system.
> ***Note:** Service meshes refer to [Layer 7](https://en.wikipedia.org/wiki/OSI_model#Layer_7:_Application_Layer) protocols in the OSI reference model, but can also be configured to operate at layers 3 and 4.*

Istio is backed by Google, IBM, and Lyft, and is currently the most widely-adopted service mesh architecture. Kubernetes, which was originally designed by Google, also dovetails nicely into Istio. It would be fair to label Istio as a ‘Kubernetes-native service mesh’.

### How it works

Like most service mesh implementations, Istio complements existing application containers with a proxy container, called a **sidecar**. Sidecar proxies are specially configured Envoy instances that intercept network traffic entering and leaving service containers and reroute the traffic over a dedicated network, as illustrated below.

![](https://cdn-images-1.medium.com/max/2860/0*bWQio3rKD0XpE8Kl)

Sidecar proxies are lightweight components optimized for latency and throughput, housing minimal configuration and routing intelligence. Routing decisions are made on the basis of policies hosted by a separate **control plane** — the metaphorical ‘brain’ of the service mesh. The control plane comprises a dedicated set of components deployed into the Kubernetes cluster — much like any other containerized application — residing in a dedicated istio-system namespace. The components of the control plane are intentionally separated from the **data plane** — the elements that perform the actual routing of network traffic and interface directly with the application containers. This separation is illustrated in the diagram below.

![](https://cdn-images-1.medium.com/max/3652/0*0t8pXEEjVrGnubPt)
> ***Note:** The sidecar proxy pattern isn’t the only approach to service mesh implementations. An alternative approach, pioneered by the Netflix technology suite, employed client-side libraries (Ribbon, Hysterix, Eureka, Archaius) to fill the role of a data plane, taking routing cues from a centralised control plane. This approach had the benefit of being agnostic of the container orchestration engine, or indeed containers altogether — it could be deployed in an embedded form within legacy, container-less applications. The benefits of Netflix libraries were also its drawbacks — they required intrusive changes to applications and offered support for a limited number of programming languages, being predominantly JVM-focused. By contrast, more modern sidecar-based service mesh designs are language-agnostic — deployed transparently alongside existing applications, requiring comparatively little engagement on behalf of application developers.*
> *Unsurprisingly, sidecar-based service meshes have enjoyed considerable adoption from operations and DevOps teams — the highly desirable separation between business logic and infrastructure is carried forward in a contemporary service mesh design.*

## Core concepts

Istio expands upon the nomenclature of a ‘vanilla’ Kubernetes setup with several Istio-specific resource types. Being a Kubernetes-native service mesh, Istio designs have implemented these concepts using custom resource definitions (CRDs). CRDs are nothing more than declarative snippets of configuration specified in YAML and managed using kubectl, akin to the built-in types such as Pod, Service, Deployment, and so on.

### Virtual services

A **virtual service** specifies how requests inbound to a specific virtual host are routed to the underlying destinations.

Virtual services decouple service consumers from service providers to the degree that simply isn’t possible with conventional Kubernetes services, which are rudimentary load balancers by comparison. Typical use cases for virtual services include the partitioning of request traffic to different versions of the service, which are specified as service **subsets**. Clients send requests to a virtual service with no awareness of the underlying provider implementation, and then Envoy forwards traffic to different versions depending on the rules defined within the virtual service configuration. For example, *“X percent of calls go to a new version”* or *“Calls from these users go to version Y”*. The astute reader will recognize these use cases as ‘canary’ rollouts, where traffic to a new service release is gradually ramped up to mitigate the risks of a big-bang release.

In addition to splitting traffic to multiple underlying provider implementations, virtual services also allow you to do the opposite — combine multiple disparate services into a single virtual service. In other words, virtual services can act as a rudimentary aggregation layer, obviating the need for a dedicated reverse proxy such as NGINX.

A well-planned virtual service can also facilitate the [Strangler Pattern](https://martinfowler.com/bliki/StranglerFigApplication.html). Provided that the service contracts are unchanged, fine-grained service routing rules can target individual endpoints, shifting the request traffic to a microservices-style implementation once the original endpoint has become deprecated on the monolith. Combining matching rules with percentage-based traffic policies, one can transparently orchestrate a gradual migration purely on the provider-end — the service consumers remaining none the wiser.

The snippet below provides a simple example of a virtual service.

    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: reviews
    spec:
      hosts:
      - shoppingcart
      http:
      - match:
        - headers:
            end-user:
              exact: emil.koutanov
        route:
        - destination:
            host: shoppingcart
            subset: v3
      - route:
        - destination:
            host: shoppingcart
            subset: v3
          weight: 25
        - destination:
            host: shoppingcart
            subset: v2
          weight: 75

The hosts section binds the virtual service to a user-addressable destination(s). In other words, it acts as a trigger to invoke the virtual service. These destinations can be fixed IP addresses, DNS names or service names — the latter being either a short Kubernetes name or an FQDN (fully-qualified domain name). Wildcard prefixes (denoted by the * character) are also supported — triggering the virtual service on a subdomain. The specified hosts don't actually need to resolve to any existing service names; you can specify arbitrary hosts to match on.
> ***Note:** The flexibility of the hosts matching model is a double-edged sword. Because the host name can be arbitrary, Istio will not conduct any form of sanity checking. For example, if the host name in the above example was misspelled as ‘shopingcart’, Istio will happily apply the configuration. Later, when the clients attempt to invoke the correctly named shoppingcart host, the request will be routed to the service directly — our virtual service rule will not have had any effect.*

The http section describes the routing rules — match conditions and actions for routing HTTP/1.1, HTTP/2, and gRPC traffic sent to the destination(s) specified in the host's field. (You can also use tcp and tls sections to configure routing rules for TCP and unterminated TLS traffic.)

A routing rule consists of the destination where the traffic is to be forwarded, as well as zero or more match conditions. In the example above, the first rule only matches requests from the user emil.koutanov. Routing rules are evaluated in a strict top-to-bottom sequence: the first matching rule is actioned, falling through to the next rule if unmatched. So in our example, the user emil.koutanov will always be directed to v3 of the shoppingcart service. All other users will be served v3 in 25% of cases. Because rules with a match predicate might not get fired, it is considered a good practice to leave the last rule without a match predicate — effectively posing as a 'catch-all'.

Unlike the virtual names specified in the hosts section, the destination hosts must be resolvable to *real* addresses. When providing a short name for the destination host, Istio will happily add a domain suffix based on the namespace of the virtual service. If the destination lies in a different namespace, the host should specify a fully-qualified service name. Istio maintainers recommend using fully-qualified names in production, as it avoids potential ambiguity and misconfiguration.

So far we have seen destinations and subsets without paying much attention to where they are defined. A **destination rule** is an *optional* fine-grained policy that controls the traffic for a specific destination. Destination rules are applied *after* virtual service routing rules are evaluated, in other words, they apply to the traffic’s ‘real’ destination.

Destination rules are defined as a CRD of the DestinationRule kind. The following is an example of a destination rule.

    apiVersion: networking.istio.io/v1alpha3
    kind: DestinationRule
    metadata:
      name: shoppingcart-destinationrule
    spec:
      host: shoppingcart
      trafficPolicy:
        loadBalancer:
          simple: RANDOM
      subsets:
      - name: v2
        labels:
          version: v2
        trafficPolicy:
          loadBalancer:
            simple: ROUND_ROBIN
      - name: v3
        labels:
          version: v3

This example should begin to make more sense, now that it’s actually complete. The v2 and v3 subsets that we were using in the VirtualService definition are merely references to labeled versions of the shoppingcart service. Labels are a standard Kubernetes concept — free-form key-value pairs that can be used to annotate Kubernetes resources. The version label will presumably appear in the metadata section of the service's Deployment resource definition, and will be matched at runtime when the appropriate destination rule subset is invoked.

### Gateways

A **gateway** controls the flow of traffic into and out of the service mesh. Behind the scenes, a gateway is an Envoy proxy instance deployed in a standalone configuration (not attached to an application container) at the notional boundary of the data plane.

The overwhelming majority of use cases for gateways revolve around the management of inbound traffic. In this capacity, gateways act similarly to regular Kubernetes ingress resources. The primary difference between a gateway and a Kubernetes ingress is that the former is designed to work specifically with Istio, while the latter is a standard API designed to handle external traffic. Ingresses and ingress controllers are generally independent of a service mesh and can function without one. In theory, one can deploy an ingress controller and configure an ingress to pre-route traffic before it reaches an Istio gateway. This strategy may be useful for aggregating services, where some services might be wired using a service mesh, while others may be deployed conventionally.

The following snippet is an example of a gateway definition.

    apiVersion: networking.istio.io/v1alpha3
    kind: Gateway
    metadata:
      name: shoppingcart-gateway
    spec:
      selector:
        istio: ingressgateway
      servers:
      - port:
          number: 443
          name: https
          protocol: HTTPS
        hosts:
        - shoppingcart.example.com
        tls:
          mode: SIMPLE
          serverCertificate: /tmp/tls.crt
          privateKey: /tmp/tls.key

This simple example accepts HTTPS traffic destined to shoppingcart.example.com on port 443. The traffic cannot flow into the service mesh until we first bind the gateway to a virtual service. Expanding on our earlier virtual service example:

    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: reviews
    spec:
      hosts:
      - shoppingcart
      gateways:
      - shoppingcart-gateway
      http:
      - match:
        - headers:
            end-user:
              exact: emil.koutanov
        route:
        - destination:
            host: shoppingcart
            subset: v3
      - route:
        - destination:
            host: shoppingcart
            subset: v3
          weight: 25
        - destination:
            host: shoppingcart
            subset: v2
          weight: 75

All we added to the previous example was the gateways section, specifying our gateway by its name. Ingress traffic on the shoppingcart-gateway gateway will now be permitted to flow into the shoppingcart virtual service.

In addition to handling ingress traffic, a gateway can also act as a controlled exit point for traffic leaving the mesh. A gateway will let you limit which services can access external networks and monitor the traffic that is permitted to leave. There are several reasons why one might want to limit egress at the infrastructure level, independent of the underlying applications. Compliance is chief among them — for example, the PCI DSS standard requires that outbound traffic from the cardholder data environment (CDE) is restricted to authorized communications, demanding that default-deny rules are in place to block unspecified traffic. Another common reason is to mitigate attacks, whereby one or more components in your system may become compromised and could attempt to leak sensitive data.

### Service entries

Service entry is an internal definition of a service maintained by Istio in its dedicated service registry. Service entries are not something one comes across too often; you can deploy a complete distributed system in Istio without ever touching this concept. Still, it is classed as a core Istio concept, and one should at least be aware of it.

The major sets of use cases for service entries fall into the following broad categories:

* **Legacy application integration**: communications with services that are not deployed in Kubernetes or are not directly reachable from within the Istio data plane.

* **Multi-cluster compositing**: logical aggregation of services from several physical Kubernetes clusters.

* **Extending the mesh beyond Kubernetes**: adding workloads deployed on physical hardware and VMs to an existing service mesh.

* **Instrumenting external services**: for example, retries, percentage routing, tracing, etc. of calls to external services.
> ***Note:** You don’t need to configure a service entry just to access an external service, for example, maps.googleapis.com. Istio egress Envoy proxies are configured to pass-through requests to unknown services by default. However, unregistered destinations will not benefit from the fine-grained traffic policies that apply to Istio-enhanced services.*

For a practical example of a service entry scenario, consider an external service hosted by someprovider.com that requires mutual TLS (mTLS) for authentication. One option is to deploy X.509 certificates and signed PEM keys to every consumer deployed in our Kubernetes cluster. This presents a logistical challenge: there may be several such consumers and each will require potentially intrusive changes to the application code to support the use of mTLS. On top of that, we have to distribute and rotate key material. This challenge can be solved by configuring Istio to act as a forward proxy, transparently encapsulating traffic originating from our data plane into a TLS tunnel, acting as an mTLS terminator. See the resource definitions below.

    apiVersion: networking.istio.io/v1alpha3
    kind: ServiceEntry
    metadata:
      name: external-serviceentry
    spec:
      hosts:
      - someprovider.com
      ports:
      - number: 443
        name: https
        protocol: HTTPS
      location: MESH_EXTERNAL
      resolution: DNS
    ---
    apiVersion: networking.istio.io/v1alpha3
    kind: DestinationRule
    metadata:
      name: external-destinationrule
    spec:
      host: someprovider.com
      trafficPolicy:
        tls:
          mode: MUTUAL
          clientCertificate: /etc/certs/myclientcert.pem
          privateKey: /etc/certs/client_private_key.pem
          caCertificates: /etc/certs/rootcacerts.pem

Two resources have been defined: service entry and a corresponding destination rule. The former does comparatively little — it enrolls someprovider.com as a service entry, placing it within the scope of Istio. The latter does the heavy lifting — specifying the authentication mode, the CA certificate for verifying the provider, and the private key and certificate for authenticating the client.

### Sidecars

We have previously touched upon sidecars as a defining element of the Istio service mesh architecture. Indeed, the humble Envoy proxy sidecar is the proverbial workhorse in the Istio ecosystem, moving traffic in the data plane at the discretion of the control plane.

Sidecars are sort of in the same boat as service entries, in that you might get a fair way through your build without dealing with them directly. Sidecars are routinely injected into application pods with minimal involvement from the user and will inherit a default configuration that works out of the box. By default, Istio will configure all sidecar proxies in the mesh to reach every workload instance, as well as accept traffic on all the ports associated with the workload.

One may explicitly amend a sidecar configuration for the following reasons:

* Limit the set of ports and protocols that an Envoy proxy accepts.

* Limit the set of services that an Envoy proxy can reach.

Sidecar configurations may be applied to an entire namespace, or specific workloads by using a workloadSelector. This gives rise to scoping rules. When determining the sidecar configuration to be applied to a particular workload instance, preference will be given to the Sidecar resource with a workloadSelector over a configuration void of a workloadSelector. Furthermore, a namespace can have *at most one* sidecar configuration without a workloadSelector. If no sidecar configuration is matched in the local namespace, the global default configuration defined in the istio-system namespace (or the configured root Istio namespace) will take effect. To make things that bit more complicated, the behavior of the system is undefined if two or more sidecar configurations with a workloadSelector select the same workload instance.

The global default configuration is quite permissive. You can view the default by running:

    kubectl get sidecar default -n istio-system -o yaml

Resulting in:

    apiVersion: networking.istio.io/v1alpha3
    kind: Sidecar
    metadata:
      annotations:
        kubectl.kubernetes.io/last-applied-configuration: |
          *** omitted for brevity ***
      creationTimestamp: "2019-12-25T01:32:02Z"
      generation: 1
      labels:
        operator.istio.io/component: IngressGateway
        operator.istio.io/managed: Reconcile
        operator.istio.io/version: 1.4.0
        release: istio
      name: default
      namespace: istio-system
      resourceVersion: "4527"
      selfLink: /apis/networking.istio.io/v1alpha3/namespaces/istio-system/sidecars/default
      uid: 5cee1cb3-1d48-11ea-84b7-025000000001
    spec:
      egress:
      - hosts:
        - '*/*'

The following sample amendment to the global default configuration restricts egress traffic only to other workloads in the same namespace, and to services in the istio-system namespace.

    apiVersion: networking.istio.io/v1alpha3
    kind: Sidecar
    metadata:
      name: default
      namespace: istio-system
    spec:
      egress:
      - hosts:
        - "./*"
        - "istio-system/*"

## Getting Started

Now that we have a solid appreciation of the core Istio concepts, let’s put it to use. We are going to deploy the sample *Bookinfo* application included in the Istio distribution, which we will later spice up with a few Istio bells and whistles. Specifically, we are going to demonstrate distributed tracing with Zipkin and mandatory user authentication with JWT.

### Requirements

The examples are going to use Istio 1.4. This version of Istio has been tested with Kubernetes releases 1.13, 1.14, and 1.15. The examples have been verified with Kubernetes 1.14.8.

It is assumed that you have access to a Kubernetes cluster and a solid background in Kubernetes. The Kubernetes cluster that comes with Docker Desktop will do just fine.

### Installation

Start by downloading the Istio distribution:

    curl -L [https://istio.io/downloadIstio](https://istio.io/downloadIstio) | sh -

Move to the Istio package directory. At the time of writing, the latest stable Istio version was 1.4.2. You might have a newer version, so please change the path accordingly.

    cd istio-1.4.2

The installation directory contains the following:

* Istio resource definitions — required to install Istio to a Kubernetes cluster. Recall, Istio is just another application deployed into Kubernetes.

* Sample applications in the samples/ directory.

* The istioctl client binary in the bin/ directory.

The next step is optional but highly recommended. Add istioctl to your path:

    export PATH=$PWD/bin:$PATH

Istio offers several **configuration profiles**. These profiles provide pre-canned customizations of the Istio control plane and the sidecars for the Istio data plane. You can start with one of Istio’s built-in configuration profiles and then tailor the configuration for your specific needs. There are five built-in profiles; we will briefly touch upon three of the more common ones.

1. default: The recommended profile for production deployments. Features minimal add-ons and uses production-grade defaults.

1. demo: Used to showcase the breadth of Istio's functionality. Features the complete set of add-ons and configuration optimized for minimal resource usage. It also contains an elevated amount of tracing and access logging, so it is generally not recommended for performance-sensitive deployments.

1. minimal: A minimalistic deployment of Istio sufficient to utilize its traffic management capabilities.

Let’s go ahead and install the demo profile:

    istioctl manifest apply --set profile=demo \
        --set values.tracing.enabled=true \
        --set values.tracing.provider=zipkin

Running this operation for the first time may take a little while. Kubernetes will deploy a dozen or so services, pulling in a whole bunch of Docker images. In addition, we have enabled the Zipkin add-on that we expect to showcase at a later point. If you have a previous installation of Istio without this add-on, don’t worry — you can rerun istioctl manifest apply at any time, specifying an alternate profile or a different set of add-ons.

Having deployed Istio, verify that all components are present by querying the istio-system namespace. (This is the default Istio *root* namespace, and can be reconfigured to an alternate namespace if necessary.)

    kubectl get svc -n istio-system

Outputting:

    NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                                                                      AGE
    grafana                  ClusterIP      10.105.190.242   <none>        3000/TCP                                                                                                                     3d3h
    istio-citadel            ClusterIP      10.100.144.109   <none>        8060/TCP,15014/TCP                                                                                                           3d3h
    istio-egressgateway      ClusterIP      10.101.65.30     <none>        80/TCP,443/TCP,15443/TCP                                                                                                     3d3h
    istio-galley             ClusterIP      10.100.233.70    <none>        443/TCP,15014/TCP,9901/TCP,15019/TCP                                                                                         3d3h
    istio-ingressgateway     LoadBalancer   10.106.108.201   localhost     15020:30912/TCP,80:30218/TCP,443:30059/TCP,15029:31861/TCP,15030:32620/TCP,15031:30363/TCP,15032:30868/TCP,15443:30359/TCP   3d3h
    istio-pilot              ClusterIP      10.108.102.237   <none>        15010/TCP,15011/TCP,8080/TCP,15014/TCP                                                                                       3d3h
    istio-policy             ClusterIP      10.111.28.108    <none>        9091/TCP,15004/TCP,15014/TCP                                                                                                 3d3h
    istio-sidecar-injector   ClusterIP      10.103.185.22    <none>        443/TCP                                                                                                                      3d3h
    istio-telemetry          ClusterIP      10.111.73.89     <none>        9091/TCP,15004/TCP,15014/TCP,42422/TCP                                                                                       3d3h
    kiali                    ClusterIP      10.109.207.229   <none>        20001/TCP                                                                                                                    3d3h
    prometheus               ClusterIP      10.96.164.188    <none>        9090/TCP                                                                                                                     3d3h
    tracing                  ClusterIP      10.108.205.179   <none>        80/TCP                                                                                                                       3d3h
    zipkin                   ClusterIP      10.107.7.90      <none>        9411/TCP                                                                                                                     3d3h

Ensure the corresponding Istio pods are also deployed and have a status of Running. When deploying for the first time, it may take a while before the READY status of each pod transitions from 0/1 to 1/1:

    kubectl get pods -n istio-system

Outputting:

    NAME                                      READY   STATUS    RESTARTS   AGE
    grafana-5f798469fd-qmzqd                  1/1     Running   1          3d3h
    istio-citadel-6dc789bc4c-qcpq6            1/1     Running   2          3d3h
    istio-egressgateway-75cb89bd7f-96xbw      1/1     Running   1          3d3h
    istio-galley-5bcd89bd9c-6z9hm             1/1     Running   1          3d3h
    istio-ingressgateway-7d6b9b5ffc-n9vrp     1/1     Running   1          3d3h
    istio-pilot-678b45584b-h2pxv              1/1     Running   1          3d3h
    istio-policy-9f78db4cb-bjk54              1/1     Running   5          3d3h
    istio-sidecar-injector-7d65c79dd5-th5ts   1/1     Running   2          3d3h
    istio-telemetry-fc488f958-9np6n           1/1     Running   5          3d3h
    istio-tracing-6bc9795bd7-4gqrr            1/1     Running   1          3d2h
    kiali-7964898d8c-bsxvd                    1/1     Running   1          3d3h
    prometheus-586d4445c7-qcjxk               1/1     Running   1          3d3h

### Enabling sidecar injection

Istio will automatically inject sidecar containers into application pods launched in any namespace labeled with istio-injection=enabled.

As our Bookinfo example will be hosted in the default namespace, let's prepare it for sidecar injection:

    kubectl label namespace default istio-injection=enabled

Istio does not force automatic sidecar injection upon your application. You can use istioctl kube-inject to enrich any existing resource definitions, explicitly injecting Envoy containers in your application pods before deploying them:

    istioctl kube-inject -f <resource-definitions>.yaml | kubectl apply -f -

### About the Bookinfo example

The Bookinfo application is included in the Istio distribution in the samples/bookinfo/ directory. It's a polyglot application that displays information about a book, similar to a catalog entry of an online book store. It comprises four microservices communicating in a directed acyclic graph (DAG) arrangement:

productpage: Invokes the details and reviews services to populate the page.

details: Contains book information.

reviews: Contains book reviews. Depending on the implementation version, the reviews service will call the ratings service to obtain book ratings. There are three versions of reviews deployed:

* v1 doesn’t call the ratings service.

* v2 calls the ratings service, displaying each rating as 1 to 5 *black* stars.

* v3 calls the ratings service, displaying each rating as 1 to 5 *red* stars.

ratings: Contains book rating information that accompanies a book review.

The high-level architecture of Bookinfo is illustrated below.

### Deploying Bookinfo

Deploy the application using kubectl:

    kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
    kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml

Note: If you chose not to enable automatic sidecar injection, the equivalent could be achieved with istioctl:

    kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/platform/kube/bookinfo.yaml)

Run kubectl get svc and kubectl get po to verify that the application has been deployed:

    NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
    details       ClusterIP   10.104.247.109   <none>        9080/TCP   3d4h
    kubernetes    ClusterIP   10.96.0.1        <none>        443/TCP    3d5h
    productpage   ClusterIP   10.102.89.169    <none>        9080/TCP   3d4h
    ratings       ClusterIP   10.103.85.175    <none>        9080/TCP   3d4h
    reviews       ClusterIP   10.100.227.109   <none>        9080/TCP   3d4h

    NAME                             READY   STATUS    RESTARTS   AGE
    details-v1-c5b5f496d-zz5dh       2/2     Running   2          3d4h
    productpage-v1-c7765c886-zdqv4   2/2     Running   0          2m17s
    ratings-v1-f745cf57b-lln4d       2/2     Running   2          3d4h
    reviews-v1-75b979578c-t4ns7      2/2     Running   0          2m17s
    reviews-v2-597bf96c8f-jrs2c      2/2     Running   0          2m17s
    reviews-v3-54c6c64795-zxzk2      2/2     Running   0          2m17s

Once deployed in Istio, the architecture of Bookinfo will be amended slightly to reflect the presence of sidecar proxies:

Let’s take a moment to look around. The first step is to determine whether we have a virtual service:

    kubectl get vs

Outputting:

    NAME       GATEWAYS             HOSTS   AGE
    bookinfo   [bookinfo-gateway]   [*]     3d4h

Dump the virtual service configuration to YAML:

    kubectl get vs bookinfo -o yaml

Outputting:

    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      annotations:
        kubectl.kubernetes.io/last-applied-configuration: |
          *** omitted for brevity ***
      creationTimestamp: "2019-12-25T01:54:39Z"
      generation: 1
      name: bookinfo
      namespace: default
      resourceVersion: "6896"
      selfLink: /apis/networking.istio.io/v1alpha3/namespaces/default/virtualservices/bookinfo
      uid: 85c222bd-1d4b-11ea-84b7-025000000001
    spec:
      gateways:
      - bookinfo-gateway
      hosts:
      - '*'
      http:
      - match:
        - uri:
            exact: /productpage
        - uri:
            prefix: /static
        - uri:
            exact: /login
        - uri:
            exact: /logout
        - uri:
            prefix: /api/v1/products
        route:
        - destination:
            host: productpage
            port:
              number: 9080

What does that tell us? Well for starters, we have a virtual service named bookinfo that traps requests to any (*) host. The virtual service will accept requests that match the paths specified in spec.http.match, forwarding them to the productpage destination on port 9080.

We can also see that the bookinfo virtual service is bound to the bookinfo-gateway gateway. Let's drill into it:

    kubectl get gw bookinfo-gateway -o yaml

Outputting:

    apiVersion: networking.istio.io/v1alpha3
    kind: Gateway
    metadata:
      annotations:
        kubectl.kubernetes.io/last-applied-configuration: |
          *** omitted for brevity ***
      creationTimestamp: "2019-12-25T01:54:39Z"
      generation: 1
      name: bookinfo-gateway
      namespace: default
      resourceVersion: "6895"
      selfLink: /apis/networking.istio.io/v1alpha3/namespaces/default/gateways/bookinfo-gateway
      uid: 85bf7194-1d4b-11ea-84b7-025000000001
    spec:
      selector:
        istio: ingressgateway
      servers:
      - hosts:
        - '*'
        port:
          name: http
          number: 80
          protocol: HTTP

This is informing us that an Istio gateway is serving HTTP traffic on port 80 for all hosts. We should just call the Bookinfo service already, but first we need to know what address and port number the gateway is reachable on from outside the cluster. The answer to this depends on how the underlying Istio ingress gateway service is exposed. (Remember, Istio is made up of regular Kubernetes components — they need to be exposed to be reachable from outside the cluster.)

    kubectl get svc istio-ingressgateway -n istio-system

Outputting:

    NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                                                                      AGE
    istio-ingressgateway   LoadBalancer   10.106.108.201   localhost     15020:30912/TCP,80:30218/TCP,443:30059/TCP,15029:31861/TCP,15030:32620/TCP,15031:30363/TCP,15032:30868/TCP,15443:30359/TCP   3d20h

I happen to be running this example on the Kubernetes cluster that ships with Docker Desktop, so in my case, the EXTERNAL-IP is set to localhost. You might be running a Kubernetes cluster in AWS or GCP, in which case the LoadBalancer service type will be implemented using a cloud-native load balancer (for example, an ELB or an NLB). Alternatively, you might find yourself in an environment that doesn't support a LoadBalancer service — for example, MiniKube without tunnel — in which case the EXTERNAL-IP will be displayed as <none> or perpetually <pending>. In this case, you can access the ingress gateway service using its node port by checking the port mapping for port 80 in the PORT(S) column.

Let’s invoke our app. Browse to [localhost/productpage](http://localhost/productpage). (Replace the host and port as appropriate.)

![](https://cdn-images-1.medium.com/max/2000/0*rlEW7rFeFrTNNZon)

Refresh the screen several times. You’ll notice that a slightly different view is served each time. That’s because the reviews service is selecting all pods with the label app: reviews. We will deal with that later.

Congratulations! You have successfully deployed your first application using Istio.

### Distributed tracing with Zipkin

Before we make any drastic changes to our Bookinfo deployment, let’s first use Zipkin to gain some insights on how traffic flows throughout our microservices architecture.

Zipkin is not accessible directly. To use it, you must set up temporary port forwarding:

    istioctl dashboard zipkin

This will launch the Zipkin UI in a new browser tab. Click the ‘Find Traces’ button to see the recent traces:

![](https://cdn-images-1.medium.com/max/2150/0*L74G8YG261l21uB9)

Zipkin will render traces that traverse the chosen service. You can expand a trace, revealing the underlying spans:

![](https://cdn-images-1.medium.com/max/2184/0*__n5ZD5-FJ6LLJOT)

Zipkin also lets us visualise the high-level dependencies between services. Click on the ‘Dependencies’ link at the top of the screen:

![](https://cdn-images-1.medium.com/max/2200/0*h96woC2K1V26fSN-)

Not bad, huh? We got all this without lifting a finger.

### Tailored routing

The routing to the reviews service is a bit all over the place at the moment: customers are being served three versions at random. Let's put some structure around how we deliver a service to our consumers. Run the following, which will inject a pair of resource definitions into kubectl via standard input, *heredoc* style.

    cat <<EOF | kubectl apply -f -
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: reviews
    spec:
      hosts:
      - reviews
      http:
      - match:
        - headers:
            end-user:
              exact: bob
        route:
        - destination:
            host: reviews
            subset: v1
      - route:
        - destination:
            host: reviews
            subset: v3
          weight: 25
        - destination:
            host: reviews
            subset: v2
          weight: 75
    ---
    apiVersion: networking.istio.io/v1alpha3
    kind: DestinationRule
    metadata:
      name: reviews
    spec:
      host: reviews
      subsets:
      - name: v1
        labels:
          version: v1
      - name: v2
        labels:
          version: v2
      - name: v3
        labels:
          version: v3
    EOF

We are now serving a canary release of v3 to 25% of our general visitors, with the remainder still being on v2. In this example, we have a particularly fussy user — bob — who is presumably quite important to us. This user will always be served v1 once he signs in.

Try the Bookinfo site now. Refreshing the page will mostly serve v2, with the occasional v3 coming through. Sign in with another user — say, charlie — the behavior is still the same. (By the way, the password field is purely ornamental — any password will work.) Now sign out, and sign back in as a user bob. The screen will render the v1 view, no how many times you press that refresh button.
> ***Note:** In case you are wondering, Istio does not have any special comprehension of user identity. The end-user header is simply appended by the productpage service for all authenticated sessions.*

## Enabling origin authentication

Authentication is another common use case for service meshes. Much like an off-the-shelf API gateway, a service mesh can furnish transparent authentication and authorization controls on top of existing services. Our next example will use JSON web tokens to authenticate a user to a service. Istio calls this **origin authentication**, in contrast to **transport authentication** which verifies services.

To experiment with JWT authentication we need both a valid JWT and a JWKS (JSON Web Key Set) endpoint. The latter is a set of signed public keys that can be used to verify a JWT. A [sample JWKS endpoint](https://raw.githubusercontent.com/istio/istio/release-1.4/security/tools/jwt/samples/jwks.json) has been provided for us by the good folks at Istio. Next, we will create a Policy resource that mandates the use of JWT for the productpage service, but only for those resources that start with the /productpage or /api/v1 paths. Selective authentication is useful when the same application also serves static content that needs to be kept open to unauthenticated users.

    cat <<EOF | kubectl apply -f -
    apiVersion: "authentication.istio.io/v1alpha1"
    kind: "Policy"
    metadata:
      name: "productpage-policy"
      namespace: default
    spec:
      targets:
      - name: productpage
      origins:
      - jwt:
          issuer: "testing@secure.istio.io"
          jwksUri: "https://raw.githubusercontent.com/istio/istio/release-1.4/security/tools/jwt/samples/jwks.json"
          trigger_rules:
          - included_paths:
            - prefix: /productpage
            - prefix: /api/v1
      principalBinding: USE_ORIGIN
    EOF

Give Istio a few seconds to propagate the new policy. Then invoke the productpage service with curl, outputting just the status code:

    curl -s [http://localhost/productpage](http://localhost/productpage) -o /dev/null -w "%{http_code}\n"

Resulting in:

    401

Unauthorized, as expected. Let’s try again, this time passing in a valid JWT:

    JWT=$(curl [https://raw.githubusercontent.com/istio/istio/release-1.4/security/tools/jwt/samples/demo.jwt](https://raw.githubusercontent.com/istio/istio/release-1.4/security/tools/jwt/samples/demo.jwt) -s)
    curl -H "Authorization: Bearer $JWT" -s [http://localhost/productpage](http://localhost/productpage) -o /dev/null -w "%{http_code}\n"

Resulting in:

    200

We have successfully enabled bearer token authentication. This means we can now replace our crude authentication form with an external identity provider — such as AWS Cognito — by simply plugging in an alternate JWKS file.

### Visualisation with Grafana

One of the instant gratifications of adopting a service mesh is the sheer amount of telemetry one gets out of the box. We have already looked at distributed tracing with Zipkin; let’s now take a look at service and mesh-level metrics that Istio provides.

Before attempting this example, let’s delete the authentication policy that we used in the previous example:

    kubectl delete policy productpage-policy

The demo profile of Istio comes with Prometheus and Grafana installed. We can proxy access to Grafana via the istio dashboard command:

    istioctl dashboard grafana

This will open a browser tab to the Grafana homepage. There are a couple dozen pre-canned dashboards for all aspects of Istio’s behaviour. On the top-left, select ‘Dashboards’, followed by ‘Istio Service Dashboard’.

The initial impressions may be underwhelming: empty graphs and melancholic widgets showing ‘No Data’. That shouldn’t surprise us; after all, no-one is using our Bookinfo service just now. Let’s simulate some traffic:

    for i in `seq 1 1000`; do 
      curl -s -o /dev/null [http://localhost/productpage](http://localhost/productpage;)
    done

Go back to the dashboard and select productpage.default.svc.cluster.local from the 'Service' drop-down. If the drop-down is empty, refresh the screen and it should re-populate. The dashboard will light up like a Christmas tree. We are going to see our request throughput (queries per second), the error rate, the turnaround time (request duration), as well as more fine-grained metrics.

![](https://cdn-images-1.medium.com/max/2248/0*RUEfMpuux9EWWb9t)

Take your time to browse the other dashboards. Istio exposes its critical metrics for the underlying control plane components — Mixer, Citadel, Pilot, and Galley. Also, Istio provides a convenient summary ‘Istio Performance Dashboard’ that merges the key components into a single view.

Think of a service mesh as a pixie that works behind the scenes, unthanked and unacknowledged, to help with traffic management, instrumentation and observability, security policy enforcement, and other mundane but necessary tasks that you would leave you bored out of your wits trying to implement in your application code. Particularly when it comes to information security, implementing these concerns at the application layer, even with the help of client libraries, has never been easy — which also explains the flourishing of commercial API gateway vendors.

This article has only scratched the surface of Istio. We didn’t discuss resiliency patterns such as rate limiting, circuit breakers, back-off and retry; nor did we cover fault injection or chaos testing — an entire area of reliability engineering saved for another day. Istio’s advanced load-balancing was given a miss, along with certificate management and authorization. There is only so much one can fit into an article before it becomes overbearing. Nonetheless, you should by now have the necessary knowledge and tools to start using Istio in your application ecosystem.

There is undoubtedly a fair amount of practice necessary to become proficient in Istio, but it is a gentle ramp-up in comparison to the much steeper learning curve of Kubernetes. Also, when compared to the effort it would take to *correctly* implement each of its capabilities in isolation and to maintain these in the long run, the return makes the investment appear very viable indeed.

*Was this article useful to you? I’d love to hear your feedback, so don’t hold back! If you are interested in Kafka, Kubernetes, microservices, or event streaming, or just have any questions, [follow me on Twitter](https://twitter.com/i/user/562466177). I’m also a maintainer of [Kafdrop](https://github.com/obsidiandynamics/kafdrop) and the author of [Effective Kafka](https://www.apachekafkabook.com).*
