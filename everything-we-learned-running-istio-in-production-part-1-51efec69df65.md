
# Everything We Learned Running Istio In Production — Part 1

Delivering meals every week to millions of customers is more complicated than it seems on the surface. Behind the scenes, to operate at our scale, a large amount of infrastructure is required before a meal kit is ultimately delivered at a customer’s doorstep.

At HelloFresh we run hundreds of microservices that do everything from supply chain management and handling payments to saving customer preferences. Running microservices at scale is not without its own challenges and many companies are beginning to experience the pain of complexity. Like many other microservice adopters, we have found that as the number of services scale, it becomes increasingly difficult to understand the interactions between all these services. When a problem occurs in a microservice world, it can be really difficult to pinpoint where in the stack the problem may be. Early in 2019 we started looking at ways to tackle this problem and concluded that a service mesh, specifically Istio, was the best option.

![(HelloFresh Service Mesh)](https://cdn-images-1.medium.com/max/3200/0*Gz3KQ9cOxOECsQmI)*(HelloFresh Service Mesh)*

### **Enter Istio**

Istio is a service mesh that is built around an Envoy proxy to manage and control the flow of traffic, secure services and see what’s happening between them. In addition, Istio works well with other common infrastructure and monitoring components such as Jaeger, Grafana, Kiali and Prometheus. Since we had already [moved all of our workloads to Kubernetes / EKS](https://engineering.hellofresh.com/lessons-learned-from-running-eks-in-production-a6dddb8368b8) and already used the aforementioned tools, Istio was a natural fit for us. We spent several months deploying and configuring Istio on our staging and production workloads and by December 2019 we had rolled out our last sidecars alongside our production services. This series of blog posts will explain everything we learned during that time and will cover some of the lessons learned that may not necessarily be highlighted in the docs.

### **Focus on what you need**

Istio can do many things. Looking at the number of features it brings can be overwhelming, but at its core Istio really does four main things: connectivity, observability, security, and traffic control. Early on we decided that we would focus our efforts, providing observability and resilience. Our strategy was to first implement the building blocks of observability and resilience and once comfortable with that baseline move onto other features.

In Istio,this meant choosing the core set of features that enabled observability and reliability. For resilience we enabled Circuit Breaking, Outlier Detection, Retries (where necessary), and Timeouts. We enabled them in a way that developers could choose which features they want to implement but with a company standard interface (more on that below). In practical terms we added sidecar proxies (Envoy), Gateways, Virtual Services and Destination Rules for most services.

The recommendation here is that Istio may be too complex for most companies to roll out every feature all at once. So, focus on one or two aspects first, then add the rest later.

### **Centralized Istio Templates**

We deploy all services using Helm charts. Normally, to create Virtual Services and Destination Rules company wide, we would have to update every git repo with the appropriate Istio related Helm templates. However, we wrote a Helm plugin that loads centralized Helm templates which contain Istio Virtual Services, Destination Rules, Gateway resources. The plugin uses these templates in conjunction with a project specific values to generate a complete chart. Application teams can then configure the values file to meet their needs, while we maintain the templates. To enable Istio for everyone, all we had to do was create the templates and teams would just have to update their helm values. We created a simple interface to control all the aspects of Istio via helm values such as this:

![Sample Helm Values File](https://cdn-images-1.medium.com/max/2000/1*CHYkIc2Vx4EfDy1LNTodDw.png)*Sample Helm Values File*

### **To Sidecar or Not to Sidecar**

Eventually, after deliberating and deciding which features to roll out and how developers will interact with them, we set out to test. In Istio that meant adding a proxy sidecar to application pods. Istio works by injecting a sidecar proxy based on Envoy alongside your application which intercepts all inbound and outbound calls going to and from the pod. The sidecar proxy, can then be configured to control or secure traffic going through it but also collects telemetry metrics and traces making them available for Prometheus and Jaeger.

To start off we chose not to have a sidecar injected in every new pod so we disabled auto injection globally:

    global:
      proxy:
        autoinject: disabled

Likewise, not every Kubernetes namespace we have needed to have sidecars injected into them, so we limited the namespaces that could have a sidecar by adding the following label to istio-enabled namespaces:

    istio-injection: "enabled"

With these two pieces in place, we were able to roll out Istio on an application by application basis by adding the following annotation to pods:

    sidecar.istio.io/inject: "true"

Istio can drastically alter the intended behavior at the application level. Before you blindly deploy it everywhere you’ll want to make a conscious choice as to which applications to run it on. Also, not every application is compatible with Istio so be sure to [check first](https://github.com/istio/istio/issues/14743).

### **Rollout in Stages**

At HelloFresh we organize our teams into squads and tribes. Each tribe has their own Kubernetes namespace. As mentioned above, we enabled sidecar injection namespace by namespace then application by application. Before enabling applications for Istio we held workshops so that squads understood the changes happening to their application. Since we employ the model of “you build it, you own it”, this allows teams to understand traffic flows when troubleshooting. Not only that, it also raised the knowledge bar within the company. We also created Istio related [OKR’s](https://en.wikipedia.org/wiki/OKR) to track our progress and reach our Istio adoption goals.

![Progress of Istio adoption for a month](https://cdn-images-1.medium.com/max/2000/1*smlXYRM8TDkFqlyWF86Oig.png)*Progress of Istio adoption for a month*

### **Scale the Control Plane**

Every new sidecar adds more load to the Istio control plane. Pilot connects to every istio-proxy and every istio-proxy checks and reports with Mixer for each request (minus some caching) so these control plane components become very critical. Before every namespace we migrated to Istio we monitored and tuned the control plane. This provided us with a good checkpoint before proceeding to add more load. Regularly we had to adjust the number of replicas, CPU and memory requests/limits for each control plane component. Also, we kept an eye on the amount of memory consumed by each istio-proxy. Remember that although the Envoy proxy is relatively lightweight, having thousands running doesn’t come for free and can add up to a large memory footprint. We already are using Kubernetes cluster autoscaler so our clusters scale automatically, but it’s important to include these figures when calculating the cost and performance overhead of running a service mesh.

Likewise, all the logs, metrics and traces being generated, put additional load on our Graylog, Jaeger, and Prometheus infrastructure. Frequently, we had to add more capacity to our logging infrastructure, Jaeger, Cassandra databases, and also scale our Prometheus pods before more namespaces got onboarded to Istio.

### Conclusion

These are just some of the high-level steps that we took to ensure a smooth Istio rollout across our company. Stay tuned for Part 2 where we will dive a bit deeper into the implementation details.
