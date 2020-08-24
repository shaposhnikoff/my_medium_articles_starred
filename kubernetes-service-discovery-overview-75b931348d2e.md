Unknown markup type 10 { type: [33m10[39m, start: [33m62[39m, end: [33m94[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m126[39m, end: [33m130[39m }

# Kubernetes Service Discovery — Overview



Modern applications rely on microservices to remain scalable and efficient. Kubernetes provides the perfect environment for microservices to exist and work together with the tools and features it offers. As each part of the application gets placed in a container, the system as a whole becomes more scalable.

The use of microservices and containers also lends itself to today’s CI/CD workflow. There is no need to bring the entire system down for an update because each microservice (container) can be updated separately. This, in turn, makes the lifecycle of containers shorter.

During the lifetime of an app and it’s microservices, some can fail and go down producing unexpected issues. This is when the service mesh can help with re-routing, security, etc.

## What is Kubernetes Service Discovery

![](https://cdn-images-1.medium.com/max/2000/0*upvUbQIXynCAxP0m)

## Dynamic IP Allocation

Before we can get to how services can be managed, and how efficient service discovery can be established, we must first look at the primary challenge of service discovery: IP allocation; more specifically, the way Kubernetes dynamically allocate IP addresses to pods and services.

Yes, it is possible to define IP addresses for individual pods and services but doing so actually limits the scalability of your Kubernetes environment. The environment defaults to allocating new IP addresses to pods and services with every restart of the cluster, pod or service.

To overcome this challenge, there are two methods you can use. The first is by looking into the environment variables of the services. Similar to how Docker allows for containers to communicate with each other, Kubernetes lets you scan environment variable s injected to containers.

If you have a service that runs on several ports, running the kubectl exec memcached-rm58b env command and then doing a quick grep for the service name will reveal the available IP address and ports assigned to that service. This, of course, isn't the most efficient way to manage service discovery.

## Kube-DNS to the Rescue

The second approach is often considered more efficient in the long run, and that approach is made possible thanks to a Kubernetes add-on Kube-DNS. As the name suggests, Kube-DNS is an add-on that acts as an internal DNS resolver.

Kube-DNS is more agnostic and relies only on namespaces. There is no need to configure the pods and services differently either. There is even no need to modify the configuration files of your cluster, pods, and services to allow for DNS-based service discovery.

Kube-DNS also supports advanced DNS queries and DNS policies. You can, for example, configure each pod to follow different DNS properties than the node it runs on. This means you have the ability to use private DNS zones to customize how pods communicate with each other.

It is also possible to take it a step further and configure DNS policies on a pod-per-pod basis. All you need to do is set the node DNS policy to *None*, and then manually configure each pod to suit your specific needs.

## Labels and Selectors

![](https://cdn-images-1.medium.com/max/2048/0*gwGq5kX1L1kYXZ_p.png)

As mentioned before, you can use parameters to further influence how pods and services can communicate with each other. Kubernetes service discovery supports the use of labels and selectors for advanced controls. Labels, in particular, are handy when you are managing a complex cluster; labels can be assigned to components and pods to allow for easy identification.

The way Kubernetes treat labels and selectors makes these parameters even easier to work with. They are essentially simple key-value parameters added to the metadata. Yes, they don’t actually affect the system or the rest of the environment. You can freely use labels and selectors across pods and services-even nodes-in a complex environment.

Next, we have the replication controllers. Again, the name of the tool says it all; it is a tool that allows Kubernetes-based systems to be highly available and scalable. You can use replication controllers to create and manage replica pods and maintain high availability. Just as easily, you can remove pods and their replicas in one sweep.

## Service Mesh and Highly Scalable Systems

![](https://cdn-images-1.medium.com/max/3368/0*d-tpJ3ShWAXzif0V)

To complete the set, we have advanced service discovery approaches that are tied to existing infrastructure and platforms. AWS Cloud Map is an interesting example. Application resources within your AWS environment can have unique names, and those resources will be automatically mapped by Cloud Map. Services automatically become discoverable once they are registered, and that registration process happens as soon as the pods or services are launched.

The latest approach is using service mesh to make managing complex arrays of microservices easy. A service mesh adds standardization to the way services and pods communicate. If you are trying to create a highly available system, using service mesh to maintain visibility of pods within the environment is the perfect solution.

If your environments are on AWS though, you can utilize its service mesh capability in the form of [AWS App Mesh.](https://aws.amazon.com/app-mesh/) It automatically handles everything from traffic routes, traffic balancing, calls, and circuit breaking using API calls. All microservices can have App Mesh enabled for easier management. Since this tool is a part of the Amazon ecosystem, it automatically works with additional tools such as Amazon EKS, IAM, and VPC.

Kubernetes service discovery is one of the elements that make this container platform capable and flexible. Advanced approaches such as service mesh certainly make Kubernetes service discovery more powerful through standardization. As long as services are running, the right API calls can be used to feed data to and from each pod without disruption.

Original at Dzone.com

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and [Instagram](https://instagram.com/fauncommunity/) **📷 **and join our [Facebook](https://www.facebook.com/groups/364904580892967/) and [Linkedin](https://www.linkedin.com/company/faundev) Groups **💬**.**

**To join our community Slack team chat **🗣️ **read our weekly Faun topics **🗞️,** and connect with the community **📣** click here⬇**

![](https://cdn-images-1.medium.com/max/3000/1*6P3WpLjGv5v1ucm5dgkucg.png)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
