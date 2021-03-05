
# Kubernetes Istio simply visually explained

So what is this Istio everyone is talking about?

![](https://cdn-images-1.medium.com/max/6400/1*Qsnaoptl1U3djpmUH41c8A.jpeg)

**Make sure you understand Kubernetes Services from Part 1:**

* Part 1: [Kubernetes Services simply visually explained](https://medium.com/swlh/kubernetes-services-simply-visually-explained-2d84e58d70e5?source=friends_link&sk=49a5e832662a689111a6087e1fe1232a)

* Part 2: [Kubernetes Ingress simply visually explained](https://codeburst.io/kubernetes-ingress-simply-visually-explained-d9cad44e4419?source=friends_link&sk=e8ca596700f5b58c7ab0d85d4dab6386)

* Part 3: (this article)

* Part 4: [Kubernetes Serverless simply visually explained](https://medium.com/@wuestkamp/kubernetes-serverless-simply-visually-explained-ccf7be05a689?sk=142c4725e110bcd9ad67f93bd2b37ede)

## TL;DR / what is Istio?

Istio is a Service Mesh which allows for more detailed, complex and observable communication between pods and services in the cluster.

It manages this by extending the Kubernetes API with CRDs. It injects proxy containers into all pods which then control the traffic in the cluster.

## Kubernetes Services

From here on you should already understand Kubernetes services, you can read [part 1 of this series](https://medium.com/swlh/kubernetes-services-simply-visually-explained-2d84e58d70e5). We will now shortly dive into how Kubernetes services are implemented. I think this helps to understand how Istio does things the same and different.

![Image 1: Kubernetes native service request](https://cdn-images-1.medium.com/max/3128/1*TEQvfHTdlONaTuQh7oldEg.png)*Image 1: Kubernetes native service request*

Image 1 shows a Kubernetes cluster with two nodes and 4 pods with one container each. There is service service-nginx which points to the nginx pods and service service-python which points to the python pods. The red line shows a request made from the nginx container in pod1-nginx to the service-python service, which redirects the request to pod2-python.

ClusterIP services do a simple random or round-robin distribution by default. Services in Kubernetes are not living on specific nodes but simply exist in the whole cluster. We see this in more detail in image 2:

![Image 2: Kubernetes native service request with kube-proxy](https://cdn-images-1.medium.com/max/3220/1*ZuC2G-FzsstMG_q-OE12Pg.png)*Image 2: Kubernetes native service request with kube-proxy*

Image 2 shows the same example as in image 1, just in more detail. Services in Kubernetes are implemented by the kube-proxy component which runs on every node. This component creates iptables rules which redirect requests to pods. Hence services are nothing else than iptables rules. (There are other proxy modes available which don’t use iptables, but the procedure is the same.)

In image 2 we see that the Kubernetes API programs each kube-proxy. This happens whenever a change in the service configuration or the service’s pods occurs. This way the Kubernetes API (and the whole master node or control plane) could go down but the services would still work.

## Kubernetes Istio

We now look at the same example with Istio configured:

![Image 3: Istio Control Plane programs istio-proxy](https://cdn-images-1.medium.com/max/3188/1*PdqXungMDEXQtZrflNqeAQ.png)*Image 3: Istio Control Plane programs istio-proxy*

Image 3 shows Istio installed, which comes with the Istio Control Plane. Also prevalent to see is that every pod has a second container called istio-proxy, which gets injected into pods during creation automatically.

The most prevalent proxy for Istio is [Envoy](https://www.envoyproxy.io/) with its amazing capabilities. Though it's possible to use other proxies ([like Nginx](https://www.nginx.com/blog/nginmesh-nginx-as-a-proxy-in-an-istio-service-mesh/)) which is why we’ll only refer to the proxy as istio-proxy from now on.

We can see that the kube-proxy components are no longer shown, this is done to keep the image clean. These still exist, but pods having the istio-proxy won’t use the kube-proxycomponents any longer.

The Istio Control Plane programs all istio-proxy sidecars whenever a change in configuration or the service’s pods occurs. Similar to how the Kubernetes API programs all kube-proxy components in image 2. The Istio Control Plane uses the existing Kubernetes services just for receiving all pods each service points to. Using the pod IP addresses, Istio implements its own routing.

After the Istio Control Plane programmed all istio-proxy sidecars, it could look like this:

![Image 4: Istio Control Plane programmed all istio-proxys](https://cdn-images-1.medium.com/max/3184/1*UjuYdYvsbqJGlowAV_do0Q.png)*Image 4: Istio Control Plane programmed all istio-proxys*

In image 4 we see how the Istio Control Plane applied the current configuration to all istio-proxy containers in the cluster. For simplicity, these also include the “ClusterIP” declaration. Though ClusterIP is a Kubernetes internal service type. Istio will convert Kubernetes service declarations into its own routing declarations. But it helps to imagine this as displayed in the image.

Let’s see how a request is made using Istio:

![Image 5: Request made with Istio](https://cdn-images-1.medium.com/max/3172/1*ClFLEg7N-U8YjdX9GJgwhg.png)*Image 5: Request made with Istio*

In image 5 all the istio-proxy containers have been programmed by the Istio Control Plane and contain all necessary routing information like seen in image 3/4. The nginx container from pod1-nginx makes a request to service service-python.

The request gets intercepted by the istio-proxy container of pod1-nginx and redirected to the istio-proxy container of one python pod, which then redirects it to the python container.

## What happened here?

Images 1–5 display the same example Kubernetes application with nginx and python pods. We have seen how a request happens using default Kubernetes services and then using Istio.

**The important thing**: whatever method is used, the result is the same and the application itself doesn’t need to be changed, only the infrastructure code.

## Why all this, why using Istio?

If nothing has changed when using Istio (the nginx pods can still connect to python pods just as before) why use Istio then in the first place?

**The amazing advantage is** that now all traffic is routed over a istio-proxy container in every pod. Whenever an istio-proxy receives and redirects a request it also submits information about it to the Istio Control Plane.

Hence the Istio Control Plane knows exactly from which pod the request came, which HTTP headers were present, how long a request from one istio-proxy to another took and much more. In a cluster with many services communicating with each other, this allows for more observability and better control over all traffic.

### Advanced Routing

Kubernetes internal services can only do round-robin or random distribution of service requests to pods. With Istio much more complex ways are possible. Like redirecting depending on request headers, if errors occurred or to the service least used.

### Deployments

It allows routing certain percentages of traffic to certain service versions and hence allows for Green/Blue and Canary deployments.

### Encryption

Cluster internal traffic between pods, from istio-proxy to istio-proxy, can be encrypted.

### Monitoring / Graph generation

Istio connects to monitoring tools like Prometheus. It also works great with Kiali to display all services and their traffic out of the box.

![[https://istio.io/docs/tasks/observability/kiali](https://istio.io/docs/tasks/observability/kiali/)](https://cdn-images-1.medium.com/max/4832/1*7HCqzSuYof7lujZqTXwWCQ.png)*[https://istio.io/docs/tasks/observability/kiali](https://istio.io/docs/tasks/observability/kiali/)*

### Tracing

Because the Istio Control Plane has loads of data about requests, these can be traced and inspected with for example Jaeger.

![[https://istio.io/docs/tasks/observability/distributed-tracing/jaeger](https://istio.io/docs/tasks/observability/distributed-tracing/jaeger/)](https://cdn-images-1.medium.com/max/6452/1*VVDu8J0MRpfmhlfxbDKdrg.png)*[https://istio.io/docs/tasks/observability/distributed-tracing/jaeger](https://istio.io/docs/tasks/observability/distributed-tracing/jaeger/)*

### Multi cluster mesh

Istio has an internal service registry which can use existing Kubernetes services. Though it’s also possible to add resources from outside the cluster or even connect different clusters into one mesh.

![[https://istio.io/docs/ops/deployment/deployment-models/#multiple-clusters](https://istio.io/docs/ops/deployment/deployment-models/#multiple-clusters)](https://cdn-images-1.medium.com/max/4952/1*G81ZdoAQgHf1ucP5CcIUyw.png)*[https://istio.io/docs/ops/deployment/deployment-models/#multiple-clusters](https://istio.io/docs/ops/deployment/deployment-models/#multiple-clusters)*

## Sidecar Injection

For Istio to work every pod which should be part of the mesh needs to have the istio-proxy sidecar injected. This can be done automatically for whole namespaces during pod creation (via [Admission Controller Hook](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)) or done manually.

## Does Istio replace Kubernetes services?

No. One question I asked myself when I began with Istio was if it would replace existing Kubernetes services. The answer is no. Istio uses existing Kubernetes services to get all their endpoints/pod IP addresses.

## Does Istio replace Kubernetes Ingress?

(Maybe read into Ingress in [part 2 of this series](https://codeburst.io/kubernetes-ingress-simply-visually-explained-d9cad44e4419?source=friends_link&sk=e8ca596700f5b58c7ab0d85d4dab6386))

Yes. Istio provides new resources, like Gateway and VirtualService, and even comes with the ingress converter istioctl convert-ingress. A good source is [https://istio.io/docs/concepts/traffic-management](https://istio.io/docs/concepts/traffic-management/)

Image 6 shows how an Istio Gateway can handle ingress traffic. The Gateway itself also is a istio-proxy component.

![Image 6: Istio Gateway](https://cdn-images-1.medium.com/max/3992/1*wjqpnadF3azeNjSOxXkLHQ.png)*Image 6: Istio Gateway*

## Control Plane Components

The Istio Control Plane consists of a few smaller components like Pilot, Mixer, Citadel and Galley. If you like to dive deeper I suggest heading to [https://istio.io/docs/ops/deployment/architecture](https://istio.io/docs/ops/deployment/architecture/)

## What happens if the Istio Control Plane is down?

Because all istio-proxy sidecars are already programmed, the Istio Control Plane could go down and traffic works as before. But config updates or new pods created wouldn’t be applied.

Though for advanced routing, like sending traffic to the least used pod, or policies ([https://istio.io/docs/tasks/policy-enforcement](https://istio.io/docs/tasks/policy-enforcement/)), there needs to be communication between all istio-proxys through the Istio Control Plane. Then it would be necessary for every istio-proxy to check back to the Istio Control Plane before allowing a request.

For these configurations to work properly I think the control plane must be available at all times. If you have links regarding this feel free to add in the comments.

## What could you do next?

I wrote an example article and Github repo about [Istio Canray Deployments](https://medium.com/@wuestkamp/kubernetes-istio-canary-deployment-5ecfd7920e1c?source=friends_link&sk=2be48393ac175a2199bf5d486cb91acf).

Istio provides a great example application with a few microservices. If you like to dive into Istio this is a good way to start: [https://istio.io/docs/setup/getting-started](https://istio.io/docs/setup/getting-started/)

[This video](https://www.youtube.com/watch?v=cB611FtjHcQ) is also great if you like to dive even deeper.

## Recap

This was a simple introduction and broad overview which I hope helps people to start. Istio definitely adds another level of complexity on top of Kubernetes. Though for modern microservice architectures it actually provides a much simpler way than having to implement tracking or observability into the application code itself.

## Become Kubernetes Certified

![[https://killer.sh](https://killer.sh)](https://cdn-images-1.medium.com/max/3534/1*7Kbj17_6VncUuoBqNsAzzg.png)*[https://killer.sh](https://killer.sh)*
