
# Istio step-by-step Part 01 — Introduction to Istio

Hi! Welcome to my Istio step-by-step tutorial series. As the first tutorial, I will do a small introduction about Istio.

![](https://cdn-images-1.medium.com/max/2936/1*N2L_eNdd-FazBpvAKopPjg@2x.jpeg)
> PLEASE NOTE,
> The new release has been updated with many new features. Refer my [09th tutorial](https://medium.com/faun/istio-step-by-step-part-09-whats-new-in-istio-1-4-8cdea2555ca3) for more information)

### **What is Istio?**

Nowadays *microservices *are* *a huge trend used by companies. Developers use microservices to architect for portability, while operators manage extremely large hybrid and multi-cloud deployments. Istio is one of the latest technologies that is used by developers to reduce the complexity of their deployments.

Istio is an open-source service mesh, that layers transparently onto existing distributed applications.

Istio’s diverse features set let you successfully and efficiently, run a distributed microservice architecture and provide a uniform way to secure, connect and monitor microservices.

So, **What is a service mesh? **A service mesh is a network of microservices that makeup applications and the interactions between them. Requirements for a service mesh are;

* Discovery

* Load balancing

* Failure recovery

* Metrics

* Monitoring

* A/B testing

* Canary releases

* Rate limiting

* Access control

* End-to-end authentication

### **Why Istio is so important?**

Istio makes it easy to create a network of deployed services with load balancing, service-to-service authentication, monitoring, and more. Istio supports services by deploying a special sidecar proxy throughout the environment that intercepts all network communications between micro-services, then configure and manage Istio using **control plane **functionality which includes,

* Automatic load balancing for HTTP, gRPC, WebSocket and TCP traffic.
> ** gRPC — a modern open-source high-performance RPC framework that can run in any environment

* Fine-grained control of traffic behaviour with rich routing rules, retries, failovers and fault injection

* A pluggable policy layer and configuration API supporting access controls, rate limits and quotas.

* Automatic metrics, logs and traces for all traffic within a cluster, including cluster ingress and egress.
> ** cluster ingress — a collection of rules that allow inbound connections to reach the cluster services.
> ** cluster egress — a collection of rules that allow outbound connections to reach the cluster services.

* Secure service-to-service communication in a cluster with strong identity-based authentication and authorization.

### **Let's check out the core features of Istio**

Mainly there are 05 core features such as;

* Traffic Management

* Security

* Observability

* Platform support

* Integration and customization

I will discuss the above core features more deeply in future tutorials.

Now we’ll take a look at the architecture of an Istio service-mesh.

![](https://cdn-images-1.medium.com/max/2000/1*Il6GOrXIkfzrkRKtOC-bSA.png)

An Istio service mesh is consist of two parts as,* **data plane* **and ***control plane***.

* Data plane — is composed of a set of intelligent proxies named Envoy which is deployed as a sidecar. These proxies mediate and control all the network communication between micro-services along with Mixer (a general-purpose and telemetry hub)

* Control plane — manages and configures the proxies to route traffic. Plus it configures Mixers to enforce policies and collect telemetry.

![](https://cdn-images-1.medium.com/max/2000/0*j6FjNk_05SQB_Zva)

Let me tell you about some most important components of the architecture.

First one is ***Envoy***. Envoy is a high-performance proxy developed in C++ to mediate all inbound and outbound traffic for all services in the service mesh. Some built-in features:

* Dynamic service discovery

* Load balancing

* TLS termination

* HTTP/2 and gRPC proxies

* Circuit breakers

* Health checks

* Staged rollouts with %-based traffic split

* Fault injection

* Rich metrics

Envoy is deployed as a sidecar to the relevant service in the same Kubernetes pod. This deployment allows Istio to extract a wealth of signals about traffic behaviour as attributes.
> ** attributes — an essential concept to Istio’s policy and telemetry functionality. An attribute is a small bit of data that describes a single property of a specific service request or the environment request. Eg: an attribute can specify the size of a specific request, the response code of an operation, the IP address where a request came from…etc. Each attribute has a name and a type. The type defines the kind of data that the attribute holds.
> The mixer is, in essence, an attribute processing machine. Envoy sidecar invokes Mixer for every request, giving Mixer a set of attributes that describe the request and the environment around the request. Based on its configuration and the specific set of attributes it was given, Mixer generates calls to a variety of infrastructure backends.

Next the ***Mixer***. The mixer is a platform-independent component which accesses control and usage policies across the service-mesh and collects telemetry data from the envoy proxy and other services. The proxy extracts request level attributes and send them to Mixer to evaluate.

This includes a flexible plugin model. The model enables Istio to interface with a variety of host environments and infrastructure backends. Thus, Istio abstracts the Envoy proxy and Istio-managed services from these details.

***Pilot ***provides services discovery for the Envoy sidecars, traffic management capabilities for intelligent routing (A/B testing, canary deployments ..etc) and resiliency (timeouts, retries, circuit breakers ..etc).

This converts high-level routing rules that control traffic behaviour into Envoy-specific configurations and propagates them to the sidecars at runtime. Pilot abstracts platform-specific service discovery mechanisms and synthesizes them into a standard format that any sidecar conforming with the Envoy data plane APIs can consume. This loose coupling allows Istio to run on multiple environments such as Kubernetes, Consul or Nomad while maintaining the same operator interface for traffic management.

***Citadel ***is also one of the main components in Istio service mesh. It provides strong service-to-service and end-user authentication with built-in identity and credential management. This can be used to upgrade unencrypted traffic in the service mesh. Using Citadel, operators can enforce policies based on service identity rather than on network controls.

Finally, ***Galley***. It validates user-authored Istio API configuration on behalf of the other Istio control plane components. Over time, Galley will take over responsibility as the top-level configuration ingestion, processing and distribution component on Istio. It will be responsible for insulting the rest of the Istio components form the details of the Istio components from the details of obtaining user configuration from the underlying platform (Kubernetes).

So, this is all for the introduction to Istio. I hope now you have an idea about what is Istio and why it is so important. Next tutorial will be Getting Started with Istio. Keep in touch to learn more. Don’t forget to comment on your ideas.

[Next tutorial](https://medium.com/@nethminiromina/istio-step-by-step-part-02-getting-started-with-istio-c24ed8137741) >>

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
