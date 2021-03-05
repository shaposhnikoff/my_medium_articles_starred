
# How to Manage Microservices on Kubernetes With Istio

How to implement DevSecOps on microservices architecture with a service mesh

![Image by [PIRO4D](https://pixabay.com/users/PIRO4D-2707530/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=3357028) from [Pixabay](https://pixabay.com/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=3357028)](https://cdn-images-1.medium.com/max/3840/1*52Wm1a8SR7XTWM-XMCyKug.jpeg)*Image by [PIRO4D](https://pixabay.com/users/PIRO4D-2707530/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=3357028) from [Pixabay](https://pixabay.com/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=3357028)*

Containers and container orchestration platforms like [Kubernetes](https://kubernetes.io) have simplified the way we run [microservices](https://en.wikipedia.org/wiki/Microservices).

Container technology helped popularize the concept as it made it possible to run and scale different components of the applications in self-contained units with their independent runtimes.

While a microservices architecture enables faster delivery by reducing the scope of change to smaller parts, improving system resilience, simplifying testing, and scaling parts of the application independently, implementing it has its challenges.

Microservices are a management and security nightmare as instead of having one monolithic application, you now have multiple moving parts, each catering to specific functionality.

An extensive app can utilize hundreds of microservices interacting with each other and soon you may find things are overwhelming. The main questions your security and operations teams may ask you are:

* How do we ensure that communication between the microservices is secured? Instead of securing one monolithic application, we need to secure hundreds of smaller services.

* In case of an issue, how do we isolate the microservice to fix the problem?

* How do we test deployments with a small percentage of traffic before releasing it to the general public to gain trust?

* How do we consolidate the application logs as now they are spread in multiple places?

* How do we check the health of the services? With more components making the application, the task of application monitoring has become more complex.

While Kubernetes addresses some of the management issues, it is supposed to be a container orchestration platform, and it is very good at that.

It does not solve the problems of microservices architecture, and it has a different purpose. Service management isn’t Kubernetes’ expertise and, therefore, it does not answer all of it by default.

Communication between Kubernetes containers by default is insecure and there is no easy way to enforce TLS between pods as you would end up maintaining hundreds of TLS certificates all by yourself.

Pods communicating between each other do not apply identity and access management between them.

Though there are tools such as [Kubernetes Network Policy](https://medium.com/better-programming/how-to-secure-kubernetes-using-network-policies-bbb940909364) you can implement to run a firewall between pods, they are a [layer 3](https://en.wikipedia.org/wiki/OSI_model) solution rather than a [layer 7](https://en.wikipedia.org/wiki/OSI_model) solution, which is what most modern firewalls are.

That means that, while you can know the source of traffic, you cannot peek into the data packets to understand what they contain. That does not allow you to make vital metadata-driven decisions such as routing on a new version of a pod, based on an HTTP header.

There are Kubernetes Ingress objects which do provide a reverse proxy based on layer 7, but they don’t offer anything more.

Kubernetes does offer different ways of deploying your Pods and do some form of [A/B testing](https://en.wikipedia.org/wiki/A/B_testing) and canary deployments. It is more through implementing replicas of containers to achieve that.

For example, if I want to deploy a new version of a microservice and just pass 10% of traffic through it, I have to scale my containers to at least ten, nine being the old version and one is the new version.

Kubernetes cannot split the traffic intelligently and instead just loads balances between Pods in a round-robin fashion.

Every Kubernetes container within a Pod has separate logging, and you have to implement a custom solution over Kubernetes to capture and consolidate them.

Although the Kubernetes dashboard offers features like monitoring Pods and checking their health, it does not provide how the components interact with each other, how much traffic is flowing through each of the Pods, and what chains of containers make up the application.

As you cannot trace traffic flow through Kubernetes Pods, you cannot tell on what part of the chain the failure occurred for the request.

You can address all these problems by using a service mesh technology such as [Istio](https://istio.io/).

## Introducing Istio

So, what is Istio? Istio is a service mesh technology that helps in connecting, securing, controlling, and observing services.

When you run a microservices application, every individual microservice runs independently in containers. As a result, they have many interactions with each other.

A service mesh enables you to discover, enable, and control these interactions, often using a side-car proxy. Too many technical terms? Let me declutter it one-by-one for you.

Let us take a typical example of a Kubernetes application with a front-end and a back-end Pod. Kubernetes offers an inbuilt service discovery between Pods using Kubernetes services and [CoreDNS](https://coredns.io/).

Therefore, you can route from one Pod to another using the service name. However, you won’t have much control over how these interactions work, and what to do with the runtime traffic.

What Istio does is that it injects a side-car container within your Pod which acts as a proxy, and your main container communicates with the other container through the proxy.

Now, since all requests go via the proxy, you can control the traffic and gather the data to do what you want with them. You can also encrypt communication between Pods and enforce identity and access management using a single control plane.

![Image from [Istio](https://istio.io/docs/concepts/security/arch-sec.svg)](https://cdn-images-1.medium.com/max/2256/1*mXfP8dp7vpnf1d7QCeVcFQ.png)*Image from [Istio](https://istio.io/docs/concepts/security/arch-sec.svg)*

## The Core Functionalities of Istio

Istio has the following core functionalities.

### Traffic management

Because of the side-car proxy (also known as the envoy proxy) and Ingress and Egress gateways, Istio manages traffic and creates routing rules that allow you to control traffic and define interactions between services.

You can do cool things like implementing timeouts, retries, circuit breakers, and much more, all by changing configurations in a control plane. That allows you to do smart things like A/B testing, canary deployments, and staged rollout with traffic splitting based on percentages.

It helps you slowly roll out a release and move gradually from an existing version (Blue) to a new version (Green) all out of the box with simple controls.

If you need to run an operational test in production and see how your product behaves in live traffic, you can do live traffic mirroring to your test instances. As a result, you can gather live insights and identify potential production issues even before you go live!

You can also have an app running multiple language-specific versions of microservices and use geolocation or the user’s profile to route requests to the correct version, and much more!

### Security

Istio enables you to secure your microservices through an envoy proxy by establishing identity access management between Pods through mutual TLS.

They help you defend against man-in-the-middle attacks as they offer traffic encryption between Pods out of the box. That is, there can be a mutual authentication between a front end and a back end, and only the front end can connect to the back end.

The back end only trusts the front end and vice versa, so if one of the Pods gets compromised, no one can do anything with the rest of your application.

Istio also provides you with features to limit access through fine-grained policies and determines what is going on in your cluster using auditing tools that Kubernetes lacks at the moment.

### Observability

Because of the envoy side-car, Istio is aware of the traffic through the Pods and therefore gathers telemetry details from the services. That helps in gaining insights about service behaviour, and you can learn about future possibilities that you can use to optimize your applications.

It also consolidates application logs and can enable traffic tracing through multiple microservices. That assists you in identifying issues faster and allows you to isolate the faulty service and fix it.

## DevSecOps for Your Team

The best thing Istio offers is the fact that it does not burden developers with worrying about the security and management details of their implementation.

As Istio is Kubernetes-aware, they can still develop their applications like a standard Kubernetes deployment, and Istio automatically injects side-car containers into the Pods.

Once Istio inserts the side-car containers, the operations and security teams can enforce policies to the traffic and help secure and operate the application. A win-win situation for everyone!

Istio empowers the security and operations teams to effectively monitor microservices applications without killing the productivity of your development team.

It also enables teams within the organization to retain their niche and look at their areas of expertise.

## Conclusion

Thank you for reading! I hope you enjoyed the article. The next story describes “[How Istio works behind the scenes on Kubernetes](https://medium.com/better-programming/how-istio-works-behind-the-scenes-on-kubernetes-aeb8003f2cb5)”. So see you in the next part!
