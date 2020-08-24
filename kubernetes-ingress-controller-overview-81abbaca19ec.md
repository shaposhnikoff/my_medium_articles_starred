
# Kubernetes Ingress Controller Overview

Comparing popular Ingress Controllers for Kubernetes & laying out important considerations for choosing the right one for you

![](https://cdn-images-1.medium.com/max/2380/1*J3j5pFqZhu3KFSni8WXslQ.png)

Even though Kubernetes was initially released in June 2014, you may be surprised to know that the [Kubernetes Ingress API remains in beta](https://kubernetes.io/docs/concepts/services-networking/ingress/) as of Kubernetes v1.18. Since its inception to beta status in early 2016 (Kubernetes v1.2), the Ingress API focused heavily on portability and stayed fairly lightweight throughout. Most recently at KubeCon North America 2019, Christopher Luciano from IBM and Bowei Du from Google presented on “[Evolving the Kubernetes Ingress APIs to GA and Beyond](https://kubernetes.io/docs/concepts/services-networking/ingress/)” detailing various improvements to the API (e.g. better path matching, new IngressClass resource, hostname wildcards). With the [Ingress API on track to graduate to GA in v1.19](https://kubernetes.io/blog/2020/04/02/improvements-to-the-ingress-api-in-kubernetes-1.18/), I put together a high-level comparison of existing, popular Ingress Controllers as well as some key considerations for choosing a solution.

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/cduG0FrjdJA" frameborder="0" allowfullscreen></iframe></center>

***Disclaimer: **This article is a culmination of personal experience, public information, and anecdotal blog posts. This is NOT a comprehensive list of all Ingress Controllers in the market. Also, due to the rapid pace of development, my information may become outdated. If you notice any inaccuracies, please leave a comment below, and I’ll update as soon as possible.*

## Ingress vs. Ingress Controller

Before diving into the various Ingress Controllers, let’s quickly review what a Kubernetes Ingress is and what an Ingress Controller does. In order to expose some functionality of applications, Kubernetes provides three service types:

1. **ClusterIP**: adds internal endpoints for in-cluster communication

1. **NodePort**: exposes a static port on each of the Nodes to route external calls to internal services

1. **LoadBalancer: **creates an external load balancer to route external requests to internal services

While an Ingress is not a Kubernetes Service, it can also be used to expose services to external requests. The advantage of an Ingress over a LoadBalancer or NodePort is that an Ingress can consolidate routing rules in a single resource to expose multiple services. Strictly speaking, an Ingress is an API object that defines the traffic routing rules (e.g. load balancing, SSL termination, path-based routing, protocol), whereas the Ingress Controller is the component responsible for fulfilling those requests.

![Image Credit: [Stack Overflow](https://stackoverflow.com/questions/56896490/what-exactly-kubernetes-services-are-and-how-they-are-different-from-deployments/56896982)](https://cdn-images-1.medium.com/max/2000/0*hDrnKe7ATiXoBxaY.png)*Image Credit: [Stack Overflow](https://stackoverflow.com/questions/56896490/what-exactly-kubernetes-services-are-and-how-they-are-different-from-deployments/56896982)*

## Ingress Controllers

Kubernetes as a project currently maintains [GLBC](https://github.com/kubernetes/ingress-gce/blob/master/README.md) (GCE L7 Load Balancer) and [ingress-nginx](https://github.com/kubernetes/ingress-nginx/blob/master/README.md) controllers. With the exception of GKE, which includes GLBC by default, ingress controllers must be installed separately prior to usage. To compare each of the popular options, I’ll first highlight cloud-provider specific Ingress Controllers and dive into other open-source options.

### Cloud-Specific Ingress Controllers

All three of the major cloud providers actively support and maintain Ingress Controllers compatible with their respective Load Balancer products:

* [AKS Application Gateway Ingress Controller](https://github.com/Azure/application-gateway-kubernetes-ingress)

* [AWS ALB Ingress Controller](https://github.com/kubernetes-sigs/aws-alb-ingress-controller)

* [GCP GLBC/GCE-Ingress Controller](https://github.com/kubernetes/ingress-gce/blob/master/README.md)

The key advantage of using a cloud provider-specific Ingress Controller is native integration with other cloud services. For example, GCE Ingress Controller supports [Cloud IAP for Google Kubernetes Engine](https://cloud.google.com/iap/docs/enabling-kubernetes-howto) to easily turn on Identity-Aware Proxy to protect internal Kubernetes applications (e.g. Vault, Prometheus, Grafana — see a [monitoring setup tutorial here](https://medium.com/google-cloud/practical-monitoring-with-prometheus-grafana-part-iv-d4f3f995cc78)). As for ALB Ingress Controller, it creates an Application Load Balancer by default (as opposed to the Network Load Balancer that it uses for other open-source Ingress Controllers) and integrates well with Route 53, Cognito, and AWS WAF. Similarly, if you are using Azure Pipelines to manage your DevOps process on Azure, AKS Application Gateway Ingress Controller fits well into the Azure CI/CD workflow.

On the other hand, if you are going for a hybrid or multi-cloud strategy, using an open-source option listed below will be easier than maintaining multiple solutions per cloud provider. Aside from AKS AGIC, cross-namespace ingress is not supported, which means that a new GCE Ingress or ALB Ingress must be created per namespace. Ingress resources (i.e. external L7 load balancer) plus static IP charges can rack up quickly in a large, multi-tenant cluster with lots of namespaces. Finally, these ingresses tend to take longer to create and update as they are creating a global (or multi-regional) load balancer with more stringent health check logic (especially in GKE).

Overall, AGIC on Azure, ALB on AWS, and GLBC/GCE on GKE provide excellent performance, native L7 routing, and integrations with other cloud products. Since GLBC comes out of the box on GKE, it makes for a great first option if you are simply looking for an HTTP/S routing solution.

### Open-Source Ingress Controllers

Apart from cloud provider-specific Ingress Controllers, [Kubernetes website](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) maintains a list of popular third-party solutions:

* [Ambassador](https://www.getambassador.io/): API Gateway based on [Envoy](https://www.envoyproxy.io/) with community/commercial support from [Datawire](https://www.datawire.io/)

* [Voyager](https://voyagermesh.com/): HAProxy based Ingress Controller from [AppsCode](https://appscode.com/)

* [Contour](https://projectcontour.io/): Envoy based Ingress Controller from Heptio (acquired by [VMWare](https://www.vmware.com/))

* [Gloo](https://docs.solo.io/gloo/latest/): Envoy based API Gateway with enterprise support from [solo.io](https://www.solo.io/)

* [Citrix](https://www.citrix.com/products/citrix-adc/cpx-express.html): Ingress Controller for MPX, VPX, and CPX ADC products

* [F5](https://clouddocs.f5.com/containers/latest/userguide/kubernetes/): Supports F5’s BIG-IP Container Ingress Services

* [HAProxy](https://haproxy-ingress.github.io/): Community-driven HAProxy Ingress Controller as well as enterprise offering from [HAProxy Tech](https://github.com/haproxytech/kubernetes-ingress)

* [Istio](https://istio.io/): Ingress Gateway for Istio-enabled clusters

* [Kong](https://github.com/Kong/kubernetes-ingress-controller): nginx-based API gateway with community/enterprise options from [KongHQ](https://konghq.com/solutions/kubernetes-ingress/)

* [NGINX](https://github.com/nginxinc/kubernetes-ingress): official Ingress for NGINX and NGINX Plus

* [Skipper](https://opensource.zalando.com/skipper/kubernetes/ingress-controller/): HTTP router and reverse proxy from [Zalando](https://jobs.zalando.com/en/tech/)

* [Traefik](https://github.com/containous/traefik): HTTP reverse proxy with commercial support from [Containous](https://containo.us/)

In terms of popularity, nginx and HAProxy kept its lead in 2019 with Envoy overtaking F5 for the third spot according to [CNCF Survey 2019](https://www.cncf.io/wp-content/uploads/2020/03/CNCF_Survey_Report.pdf). It’s unclear if the survey grouped various Ingresses by underlying technology (e.g. Ambassador, Contour, and Gloo under the Envoy bucket), but continued adoption of Istio may continue the trend of Envoy as the de facto Ingress Controller of choice.

![Image Credit: [CNCF Survey 2019](https://www.cncf.io/wp-content/uploads/2020/03/CNCF_Survey_Report.pdf)](https://cdn-images-1.medium.com/max/2776/1*dyFjUu2tLPjDcQrnDDKGTg.png)*Image Credit: [CNCF Survey 2019](https://www.cncf.io/wp-content/uploads/2020/03/CNCF_Survey_Report.pdf)*

In the following section, I’ll highlight a few Ingress Controllers from the official list in logical groups (nginx, HAProxy, Envoy, etc) with some thoughts based on personal experience or comments from other blog posts.

## NGINX-Based Ingress Controllers

### [***ingress-nginx](https://github.com/kubernetes/ingress-nginx)***

This is the only open-source Ingress Controller maintained by the Kubernetes team, built on top of NGINX reverse proxy. As such, it is one of the most popular options for a simple HTTP/S routing and SSL termination use case. Thanks to its popularity, there is extensive documentation and tutorials available for common ingress tasks and related tools (e.g. [cert-manager](https://github.com/jetstack/cert-manager) and [external-dns](https://github.com/kubernetes-sigs/external-dns)).

If you don’t need a complicated solution and want a straightforward reverse proxy, ingress-nginx is a safe and reliable option. On the other hand, if you are looking for high performance and additional features supported by NGINX (e.g. JWT validation, OpenTracing), consider using the Ingress Controller from NGINX instead. Finally, the default options for ingress-nginx may have performance issues at scale, so invest some time in configuring NGINX settings (see [Eric Liu’s article for an in-depth dive into ingress-nginx](https://itnext.io/kubernetes-ingress-controllers-how-to-choose-the-right-one-part-1-41d3554978d2)).

### [NGINX & NGINX Plus Ingress Controller](https://github.com/nginxinc/kubernetes-ingress)

This is the official Ingress Controller from [NGINX Inc](https://www.nginx.com/company/) (now owned by F5) supporting both the open-source and commercial (NGINX Plus) products. The list of differences between nginxinc/kubernetes-ingress and kubernetes/ingress-nginx is documented on [Github](https://github.com/nginxinc/kubernetes-ingress/blob/master/docs/nginx-ingress-controllers.md).

As you might expect, the free version is missing several key features (e.g. dynamic reconfiguration of endpoints) since it is shipped without Lua plugins. The paid version provides session persistence based on cookies, active health checks, JWT authentication (OpenID SSO), realtime monitoring, and high availability. If you have prior experience with NGINX, this will be an easy transition to use in Kubernetes.

### [Kong](https://github.com/Kong/kubernetes-ingress-controller)

Most people know and use Kong as an API Gateway to process and route API requests. In recent years, Kong implemented several features such as native gRPC support, request/response transformation, authentication, and active health checks on load balancers to also position itself as an ingress provider. Unlike ingress-nginx, Kong insists on [not implementing a cross-namespace](https://github.com/Kong/kubernetes-ingress-controller/issues/204) Ingress Controller, citing privilege escalation as a critical attack vector in those scenarios. I have not personally evaluated Kong since I read Bouwe Ceunen’s “[Why I switched Kong For Traefik](https://medium.com/axons/why-i-switched-kong-for-traefik-b997b9948878)” blog post when I was looking for an alternative solution to GCE ingress a year ago.

## HAProxy-Based Ingress Controllers

### [HAProxy Ingress](https://github.com/jcmoraisjr/haproxy-ingress)

Along with NGINX, HAProxy is a popular, battle-tested TCP/HTTP reverse proxy solution that existed before Kubernetes. As a result, if configuring the load balancing algorithm is your primary deciding factor, HAProxy Ingress is a great option with a proven record of high performance. As an Ingress Controller, HAProxy Ingress offers dynamic configuration update via API to address reliance on static configuration files with HAProxy.

### [Voyager](https://voyagermesh.com/)

Another HAProxy-based Ingress Controller with an enterprise support option, Voyager highlights both L4 and L7 load balancing for HTTP/TCP as well as seamless SSL integration with LetsEncrypt and AWS Certificate Manager on its website.

## Envoy-Based Ingress Controllers

### [Istio Ingress](https://istio.io/latest/docs/tasks/traffic-management/ingress/)

Istio makes heavy use of Envoy proxies to mediate all traffic within the service mesh. If you are already using Istio as the service mesh solution in your cluster, using the default Istio Ingress/Gateway makes the most sense. It provides the best integration with existing Istio fabric and services with traffic routing, observability, security, and deployment models. However, Istio is not lightweight and has a fairly large learning curve, so if Envoy proxy is the only functionality you are looking for, use the following options instead.

### [Ambassador](https://github.com/datawire/ambassador)

Technically, Ambassador is an API Gateway and L7 load balancer with Kubernetes Ingress support. It is, however, fully-featured with various protocol supports (gRPC, HTTP/2, TCP, WebSockets), security (automatic HTTPS, rate limiting, custom filters), high availability (sticky sessions, circuit breakers), and even Knativ serverless integration. Although it’s based on Envoy, it connects nicely with other service mesh solutions besides Istio (e.g. Consul, Linkerd). The [benchmark results](https://www.getambassador.io/resources/envoyproxy-performance-on-k8s/) posted on their blog compares favorably to NGINX and HAProxy, although it has not been updated for several months.

### [Contour](https://github.com/projectcontour/contour)

Contour was one of the first Ingress Controllers to make use of Custom Resource Definitions (CRDs) to extend the functionality of the Kubernetes Ingress API. The CRD (HTTPProxy — renamed from IngressRoute) primarily addresses the limitations of the native Kubernetes Ingress API in multi-tenant environments. Now that IngressRoute is officially defined in Kubernetes v1.18+, Contour’s original approach may merge well with Kubernetes’s overall direction.

### [Gloo](https://github.com/solo-io/gloo)

Gloo differentiates from other Envoy-based Ingress Controllers by offering what it calls “function-level routing”. This means that Gloo can act as an Ingress and API Gateway to route traffic to not only microservices, but also to serverless functions (e.g. AWS Lambda, Google Cloud Functions, OpenFaaS, Knative). It also has excellent support for legacy/hybrid apps where traffic must call an internal API (REST, SOAP, XML) or interact with a message queue (e.g. NATS, AMQP).

## Others

### [Skipper](https://github.com/zalando/skipper)

Skipper is a HTTP router and reverse proxy that grew out of [Project Mosaic](https://www.mosaic9.org/) in 2015. Its original goal was to build an alternative solution to NGINX and HAProxy that relied on static configuration files and implement modern features such as automated canary or blue-green deployments and shadowing traffic. As a “legacy” project, a lot of Skipper’s features are now supported by other Ingress Controllers named above. However, due to Skipper’s focus on HTTP routing, it offloads other load balancer functionality (e.g. SSL termination, automatic certificate rotation, WAF integration) and opted to integrate well with AWS ALB. This might make it an interesting option for AWS users looking to migrate to Kubernetes.

### [Traefik](https://github.com/containous/traefik)

Finally, we have Traefik, a fully-featured HTTP reverse proxy and load balancer written in Go. Traefik was originally written to solve traffic routing problem for microservices, updating and configuring routes automatically and dynamically. As a result, it supports a wide range of infrastructure besides Kubernetes (Docker, Docker Swarm, Marathon, Consul, etcd, Rancher, Amazon ECS). It supports HTTP/2, gRPC, and WebSockets as well as multiple load balancing algorithms and circuit breakers.

What originally drew me to Traefik was the seamless integration with Let’s Encrypt out of the box and nice web UI to visualize Traefik health and performance without exporting metrics to Prometheus or Datadog (although those integrations are also supported). Traefik v2 (released in Nov 2019) added TCP support with SNI routing, canary deployments, traffic mirroring, and IngressRoute CRDs.

*(For a quick start guide, check out [Traefik v2 on Kubernetes](https://medium.com/dev-genius/quickstart-with-traefik-v2-on-kubernetes-e6dff0d65216).)*

## So What Do I Use?

With so many options on the market, how do I choose which Ingress Controller is right for my use case? As a general rule, **ingress-nginx** is a safe and one of the most popular choices when you need a simple solution to get started. If you are using Istio as your service mesh, **Istio Ingress** is a natural fit; otherwise, consider an Envoy-based solution that works with Consul or Linkerd. Personally, I use a combination of **Traefik** and cloud provider-specific ingress solution for latency-critical or global/multi-regional deployments. I have not tried **Gloo, **but the function routing feature seems promising as containers and serverless start to integrate further.

Some other considerations before choosing a solution:

1. **Protocol Support**: Do you need TCP/UDP or gRPC integration?

1. **Enterprise Support**: Do you need a commercial/enterprise support for a mission-critical system?

1. **Advanced Features: **Are you looking for a lightweight solution or are canary deployments or circuit breakers must-haves for your use case?

1. **API Gateway Features: **Do you need some API Gateway functionalities (e.g. rate-limiting) or a pure Kubernetes Ingress?

If you need a more detailed side-by-side comparison, check out the comparison sheet on [Kubedex](https://kubedex.com/ingress/) or on a [blog post](https://medium.com/flant-com/comparing-ingress-controllers-for-kubernetes-9b397483b46b) by the engineers from Flant:
[**Ingress on kubedex.com**
*Reading Time: 4 minutesAs far as I know this is the complete list of Ingresses available for Kubernetes. Technically…*kubedex.com](https://kubedex.com/ingress/)
[**Comparing Ingress controllers for Kubernetes**
*11 Open Source solutions including NGINX, Traefik, Istio, HAProxy, Gloo, Ambassador, Skipper and others*medium.com](https://medium.com/flant-com/comparing-ingress-controllers-for-kubernetes-9b397483b46b)
