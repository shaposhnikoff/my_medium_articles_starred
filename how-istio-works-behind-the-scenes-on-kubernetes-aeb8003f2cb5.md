
# How Istio Works Behind the Scenes on Kubernetes

Insights into Istio architecture and how its various moving parts manage microservices in Kubernetes

![Photo by [Arif Wahid](https://unsplash.com/@arifrw?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/data?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/14720/1*9s5xMyOyZn4kswHD1kc6Fw.jpeg)*Photo by [Arif Wahid](https://unsplash.com/@arifrw?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/data?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

If you’re doing [microservices](https://en.wikipedia.org/wiki/Microservices) on [Kubernetes](https://kubernetes.io), then a [service mesh](https://en.wikipedia.org/wiki/Service_mesh) like [Istio](https://istio.io/) can work wonders for you. This article is a follow up on “[How to Manage Microservices on Kubernetes With Istio](https://medium.com/better-programming/how-to-manage-microservices-on-kubernetes-with-istio-c25e97a60a59).” Today, let’s discuss Istio architecture.

Istio helps you manage microservices through two major components:

* **Data Plane. **These are the sidecar [Envoy](https://envoyproxy.github.io/envoy/) proxies Istio injects into your microservices. These do the actual routing between your services and also gather telemetry data.

* **Control Plane. **This is the component that tells the data plane how to route traffic. It also stores and manages the configuration and helps administrators interact with the sidecar proxy and control the Istio service mesh. The control plane is the brain of Istio.

Likewise, two types of traffic flow through Istio: data plane traffic, which is your business-related traffic, and control plane traffic, which is made up of messages and interactions between Istio components, that controls mesh behaviour.

## Control Plane Components

In the current Istio versions (Istio v1.5 and above), the control plane is shipped as a single binary *Istiod *and comprises of three parts: *Pilot*, *Citadel*, and *Galley*.

![[Istio Architecture](https://istio.io/docs/ops/deployment/architecture/arch.svg)](https://cdn-images-1.medium.com/max/2000/1*6O2dHRl3Ev6z_sInZ6GhQA.png)*[Istio Architecture](https://istio.io/docs/ops/deployment/architecture/arch.svg)*

### Pilot

The Pilot is the central controller of the service mesh and is responsible for communicating with the Envoy sidecars using the [Envoy API](https://www.envoyproxy.io/docs/envoy/latest/api/api). They parse the high-level rules defined in the Istio manifests and convert that to Envoy configuration.

It fosters service discovery, traffic management, and intelligent routing, which allows you to do A/B Testing, blue-green deployments, canary rollouts, and much more. They also help provide resiliency in your mesh by configuring sidecars to provide timeout, retry, and circuit breaking capabilities.

They’re responsible for providing a loose coupling between Istio configuration and the underlying infrastructure Istio runs on. Therefore, you may run Istio on Kubernetes, Nomad or [Consul](https://www.consul.io/), but the way you interact with Istio is similar.

The Pilot is aware of what platform it’s running on and ensures the traffic management is the same, irrespective of the platform.

Pilot hosts the Traffic Management API, which helps in defining configuration and provides more granular control over traffic management within your service mesh.

### Citadel

Identity and access management between your services is the central feature of Istio. It helps you allow secure communication between your Kubernetes pods. What that means is that while your developer has designed components with insecure TCP, the Envoy proxy would ensure communication between pods is encrypted.

We can implement mutual TLS the hard way by configuring every Pod with a certificate. But then you end up managing hundreds of certificates if you have a sizeable microservice application. Istio abstracts this from you by providing an identity and access manager and certificate store called Citadel.

Citadel helps you manage what service can talk to which and how they would identify and authenticate with each other.

Citadel provides user authentication, credential management, certificate management, and traffic encryption. If any pod needs a validation credential citadel provides it.

### Galley

The galley is responsible for providing configuration validation, ingestion, processing, and distribution for your service mesh. It’s the interface for the underlying APIs with which the Istio control plane interacts.

For example, if you apply a new policy, the galley ingests it, validates it, processes it, and pushes it to the right components of the service mesh.

## Data Plane Components

The data plane component comprises of Envoy proxies. Istio fosters service discovery based on these proxies.

These are Layer 7 proxies that allow critical decisions based on the content of the messages it receives. The only components that interact with business traffic are the Envoy proxies.

They help provide the following:

* Dynamic service discovery.

* Load balancing.

* TLS termination.

* Staged rollouts with percentage-based metrics.

* Fault injection.

* Health checks.

* Circuit breaking.

* Gathering telemetry data.

As all traffic moves through these Envoy proxies, they help in gathering essential data regarding your business traffic. That allows you to gather insights which would be a key to plan your future.

Istio provides an out of the box [Prometheus](https://prometheus.io/) and [Grafana](https://grafana.com/) for monitoring and visualizing this data. It’s also possible to send that data to a log analytics tool as the [Elastic Logstash Kibana (ELK) stack](https://www.elastic.co/elastic-stack) to understand the trends and do machine learning on these metrics.

As Istio works using sidecars, there’s no need to redesign your microservices application running on Kubernetes.

It’s an excellent decoupler between your development, security and operations teams. Developers can develop their code the way they used to, without having to worry about the operational and security aspects.

The security and operations team can use Istio to inject sidecars in the application, which helps them to hook management and security policies on top of them.

The Envoy proxies help to provide most of the Istio features such as:

* **Traffic control**: They help in controlling how traffic moves through your service mesh, such as providing routing rules for HTTP, TCP, Websocket, and gRPC traffic.

* **Security and authentication** — They enforce identity and access management over the pods so that only the right pods can interact with another. They also implement mutual TLS and traffic encryption to prevent man-in-the-middle attacks. They provide rate limiting, which prevents runaway cost and denial of service attacks.

* **Network Resiliency** — They help provide network resiliency features such as retries, failover, circuit breaking, and fault injection.

It also provides a way to plug in custom policy enforcement and telemetry data gathering using a web assembly extensions model.

## How Components Interact With Each Other

Everything Istio does is via the Envoy proxy. Let’s look at the typical flow any packet through Istio would take.

![Image from [Istio](https://istio.io/docs/concepts/security/arch-sec.svg)](https://cdn-images-1.medium.com/max/2256/1*mXfP8dp7vpnf1d7QCeVcFQ.png)*Image from [Istio](https://istio.io/docs/concepts/security/arch-sec.svg)*

If you look at the above image, there is an Ingress, two microservices, and an Egress. Service A and Service B are microservices running on containers, and they share the same Pod with the Envoy Proxy.

Istio injects the Envoy proxy automatically into the Pod when they get deployed. Service A is the front end service, and the Service B is the back end service, as a typical implementation would have.

A traffic packet follows the following steps to move through your service mesh.

### Ingress

The traffic arrives at your service mesh through an Ingress resource. Ingress** **is just a bank of one or more Envoy proxies which are configured by Pilot as soon as they get deployed.

The Pilot uses the Kubernetes service endpoints to configure the Ingress proxy, and the Ingress proxy is aware of its back ends.

Ingress is therefore aware of where to send the packet if it receives — it does health checks and load balancing and makes intelligent decisions to send the message, based on defined metrics such as load, packets, quotas, and traffic balancing.

### Service A

Once Ingress routes to the first pod it hits the proxy of the Service A pod instead of the container.

Since the sidecar forms the part of the same pod as the microservice container, they share the same network namespace and are present in the same node.

Both containers have the same IP address and share the same IP Table rules. That makes the proxy take complete control over the pod and handle all the traffic that passes through it.

The Envoy proxy interacts with the Citadel to understand if any policies need enforcing. It also checks if it needs to encrypt traffic through the chain and establish TLS with the back end pod.

### Service B

Service A sends the encrypted packet to Service B, which follows the same steps as Service A.

It additionally identifies the sender of the packet and does a TLS handshake with the source proxy.

Once they establish trust, it accepts the packet and forwards it to the Service B container, and the chain moves on to the Egress layer.

### Egress

All traffic going out of the mesh, makes use of the Egress resource.

The Egress layer defines what traffic can go out of the mesh and uses Pilot and is similar in configuration to the Ingress layer.

Using an Egress resource, you can write policies to limit outbound traffic and only allow the required services to send packets out of the mesh.

### Telemetry data collection

In all the steps above, when the traffic moves through the proxies, it can collect and send the telemetry data to Prometheus.

The data can be visualised in Grafana, or sent to external tools like ELK for further analysis and machine learning of metrics.

## Conclusion

Thanks for reading! I hope you enjoyed the article.

In the next article “[Getting Started With Istio on Kubernetes](https://medium.com/better-programming/getting-started-with-istio-on-kubernetes-e582800121ea)”, I would discuss installing Istio, and we’ll do a hands-on demonstration with a sample application — so see you in the next part!
