
# How to Harden Your Microservices on Kubernetes Using Istio

How Istio implements security for your microservices

![Image by [Kati](http://xilophotography) at [Xilo Photography](http://xilophotography.com)](https://cdn-images-1.medium.com/max/10944/1*g0zGM7_4j0jmEo4IEzhnhQ.jpeg)*Image by [Kati](http://xilophotography) at [Xilo Photography](http://xilophotography.com)*

Running [microservices](https://en.wikipedia.org/wiki/Microservices) in production provides many benefits. Some of them are independent scalability, agility, the reduced scope of change, frequent deployments, and reusability. However, they come with their own set of challenges.

With a monolith, the idea of security revolves around securing a single application. However, a typical enterprise-grade microservice application may contain hundreds of microservices interacting with each other.

[Kubernetes](https://kubernetes.io/) provides an excellent platform for hosting and orchestrating your microservices. However, all interactions between the microservices are insecure by default. They communicate on plaintext HTTP, and it might not be enough to satisfy your security requirements.

To apply the same principles on microservices that you would use on a typical enterprise monolith, you need to ensure the following:

* All communications between the microservices are encrypted to prevent a man-in-the-middle attack

* Access control is present between services so only the right microservices can interface with each other

* To capture, log, and audit telemetry data to understand traffic behaviour and detect an intrusion proactively

[Istio](https://istio.io/) provides these features out of the box, and the simplicity of installing and managing it allows developers, system administrators, and the security team to appropriately secure their microservices application without stepping on each other’s shoes.

With Istio, you can enforce strong identity and access management, transparent TLS and encryption, authentication and authorisation, and audit logging — all using a single control plane.

This story is a follow up to “[How to Visualise Your Istio Service Mesh on Kubernetes](https://medium.com/better-programming/how-to-visualise-your-istio-service-mesh-on-kubernetes-209c7b439a41).” Today, let’s discuss hardening your microservices running on Kubernetes using Istio.

This article assumes you have a basic understanding of Kubernetes and Istio. If not, I recommend you check out “[How to Manage Microservices on Kubernetes With Istio](https://medium.com/better-programming/how-to-manage-microservices-on-kubernetes-with-istio-c25e97a60a59)” for a quick introduction.

## How Istio Security Works

If you’re following the series, you should be aware that Istio injects sidecar proxies in your pods automatically and modifies the IP tables of your Kubernetes cluster to allow connections only via the proxy.

This setup enables TLS encryption by default. You don’t need to do any particular configuration for your microservices as the Envoy proxies communicate with each other via TLS.

While the default setup allows elementary security and prevents man-in-the-middle attacks, it’d make sense to harden your microservices further by enforcing policies. However, before delving into the features, let’s have a look at the high-level overview of how security works on Istio.

Istio contains the following components for enforcing security:

* A certificate authority for management of keys and certificates

* A configuration API server that distributes authentication policies, authorisation policies, and secure naming information to the Envoy proxies

* The sidecar proxies help in securing the mesh by providing policy enforcement points. That’s the component that’s involved in acting on the policy supplied to it.

* Envoy proxy extensions allow for telemetry data collection and auditing

![[Istio security](https://istio.io/docs/concepts/security/arch-sec.svg)](https://cdn-images-1.medium.com/max/2256/1*V8KAdUmBjtFhkXA0GChsHQ.png)*[Istio security](https://istio.io/docs/concepts/security/arch-sec.svg)*

## How Istio Manages Identity and Certificates

Istio needs to identify each of the microservices to apply policies on top of them. Istio establishes identity through X509 TLS certificates, which are used by every Envoy proxy.

The Envoy proxies contain Istio agents that talk to istiod to provide and rotate the certificate and private key for every microservice. That allows for additional security, as you don’t need to worry about rotating certificates.

If a particular key is compromised, Istio soon replaces it with a new one, therefore decreasing the attack surface by a significant amount.

Istio uses the following steps to enforce it.

1. Envoy sends a certificate and private-key request using the Envoy secret discovery service (SDS) API

1. The Istio agent responds to this request by generating a CSR and private key for the proxy and then sends a CSR to istiod with its credentials

1. The certificate authority (CA) hosted by istiod validates the request credentials and signs the CSR to generate the certificate

1. The istio agent then downloads the certificate and sends it to the Envoy proxy via the SDS API

The process repeats periodically to provide certificates and private-key rotation.

![[Istio certificate management](https://istio.io/docs/concepts/security/id-prov.svg)](https://cdn-images-1.medium.com/max/2000/1*6YOsS98-RbDUlbJMy0K0XA.png)*[Istio certificate management](https://istio.io/docs/concepts/security/id-prov.svg)*

## Istio Authentication

There are two types of authentication Istio provides:

* **Peer-to-peer authentication **— Istio applies this when two microservices interact with each other. Istio uses mutual TLS for peer-to-peer authentication.

* **Request authentication **— Istio allows end users and systems to interact with Istio microservices using request authentication. It uses [JSON Web Tokens](https://jwt.io/) (JWT) to enforce this. You can also hook Istio up with any custom auth providers that use [OAuth,](https://oauth.net/) such as [OpenID Connect](https://openid.net/connect/) and [Google Auth](https://www.google.com/landing/2step/).

Istio uses PeerAuthentication policies for determining the level of authentication and encryption requirements. By default, Istio authenticates clients in the PERMISSIVE mode. That means in addition to allowing TLS traffic from a peer, it also accepts plaintext traffic from a source.

Organisations recently introduced to Istio may require it as they’re in the process of migrating their workloads to Istio.

That removes a significant overhead from an operator who has issues upgrading the source to use Istio for various reasons. That provides organisations with a seamless onboarding experience.

Once you’ve onboarded your entire enterprise or if you’re starting from scratch, you should turn off PERMISSIVE mode and enable STRICT mutual TLS only mode. That not only prevents man-in-the-middle attacks but also ensures all traffic through your mesh is encrypted.

If required, you can turn off mutual TLS between your microservices by running Istio authentication in the DISABLE mode. That’s not recommended, and you shouldn’t use it unless you’re providing a custom security solution.

You can enforce PeerAuthentication policies at the global level, namespace level, or the workload level. Your specific use case determines how you want to apply it.

The below YAML applies STRICT PeerAuthentication policy at the foo namespace layer.

<iframe src="https://medium.com/media/064aeb6a73c1b77178f7e3d430a60e28" frameborder=0></iframe>

## Istio Authorisation

Let’s now get to the next level. Till now, we’ve ensured that communications between microservices in Istio are encrypted and that services understand who their client is.

However, what if you want to ensure that only the right microservices interface with each other. For example, if you have a front-end microservice, a business-tier microservice, and a back end, you wouldn’t want to allow communication from the front-end microservice directly to the back end. Instead, you’d want to route this communication via the business layer.

You can use AuthorizationPolicy resources to enable authorisation between your microservices and use the following to establish a proper traffic channel:

* The selector field specifies the policy target

* The action field specifies whether to allow or deny the request

* The rules section defines when to apply the action using the from, to, and when attributes

Let’s look at an example to understand further.

Below is a typical implementation of a microservice that allows requests from sources authenticated by Google in an OAuth scenario.

<iframe src="https://medium.com/media/b12a283454b80c529a6883e4d3483c69" frameborder=0></iframe>

The YAML file:

* Selects all targets labelled with app: httpbin and version: v1

* Allows requests from containers using the service account cluster.local/ns/default/sa/sleep and from all containers in the dev namespace to only the GET method of the target when the issuer claim of the supplied JWT has a value [https://accounts.google.com](https://accounts.google.com)

Let’s look at something more straightforward. How about allowing all requests only from the foo namespace to the httpbin:v1 microservice.

<iframe src="https://medium.com/media/8a6716ea5b4e0df47c81305e931c2a08" frameborder=0></iframe>

If you look at the YAML carefully, it implements a DENY action. Don’t get confused, as the source is notNamespaces. That means the policy denies all requests unless it originates from the foo namespace.

The deny policy takes precedence over the allow policy. It evaluates the deny policies first to ensure requests matching the deny policy get rejected irrespective of the allow policy, so keep that in mind when designing your policies.

Let’s look at a different scenario. What about allowing requests only to the GET and HEAD methods of the products application.

<iframe src="https://medium.com/media/21bdeaa6339e5d301318abcbf68ac526" frameborder=0></iframe>

As Istio Envoy proxies work on [layer 7](https://en.wikipedia.org/wiki/OSI_model#Layer_7:_Application_Layer), you can also match on things like hostnames, URL paths, and HTTP headers. The below example allows all requests to the paths /test/* and */info of the products microservice.

<iframe src="https://medium.com/media/4cdaa466cf1c16871691abb769330d68" frameborder=0></iframe>

Istio provides a lot of ways to enforce authorisation policies, and the above list isn’t exhaustive.

## Conclusion

Thanks for reading through. I hope you enjoyed the article.

In the next part “[Enable Mutual TLS Authentication Between Your Kubernetes Workloads Using Istio](https://medium.com/better-programming/enable-mutual-tls-authentication-between-your-kubernetes-workloads-using-istio-65338c8adf82)”, we’ll deep dive into Istio authentication with a hands-on demonstration, so see you in the next story!
