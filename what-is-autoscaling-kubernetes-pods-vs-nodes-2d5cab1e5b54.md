
# what is Autoscaling? Kubernetes Pods vs. Nodes

Not only does Kubernetes have the capacity to deploy and manage containers, but it can also automatically scale the overall solution in numerous ways. This is a tremendous asset, especially in the modern cloud, where costs are based on the resources consumed.

The ability to scale up and down allows us to request resources on demand and then, critically, hand them back when we are finished. The concept of dynamic resource management extends beyond just individual containers, and allows us to automate, scale, and manage the state of applications and pods (collections of containers), complete clusters, and entire deployments.

There are a number of different ways to control scaling with Kubernetes. In this article, we’ll focus on two broad aspects of scaling: scaling pods and scaling clusters. It seems like a simple subject, but because we can scale along multiple axes, untangling the options is key to scaling a deployment successfully. Get More Additional Info At [**Kubernetes online Training](https://onlineitguru.com/kubernetes-training.html)**

## Scaling Pods

Pods are collections of containers managed as a group. They can contain one or more services deployed in containers. A pod might be restricted to a single application or an individual microservice. Scaling pods is maybe the best method we will scale a deployment. In general, we look to scale a pod when we need either more resources for a particular pod instance or we need to create further instances of a pod to spread a workload across a number of different container instances.

Pods will be scaled within the context of currently available resources. This means that if we have, for example, a node with 8GB of spare RAM not being used, then another pod can be scheduled on that node. When the pressure or loading on a cluster that is running a collection of pods becomes too much, we can decide to scale up the entire cluster; this use-case is described in the next section. Scaling pods is achieved using the Horizontal Pod Autoscaler (HPA) and Vertical Pod Autoscaler (VPA).

![](https://cdn-images-1.medium.com/max/2000/1*IrEFaqOakTVWgzwEFUfaNQ.png)

The HPA is what most users will be familiar with and associate as a core functionality of [**Kubernetes](https://onlineitguru.com/blogger/what-is-kubernetes)**. Its main purpose is to change the number of replicas of a pod, scaling to add or remove pod container collections as needed. Like all of the auto scaler workflows, the HPA achieves its goal by checking various metrics to see whether preset thresholds have been met and reacting accordingly.

Where the HPA takes care of scaling and distributing pods over the entire available Kubernetes cluster, the VPA is only concerned with increasing the resources available to a pod that resides on a particular node. The VPA gives you control over automatically allocating more (or less) CPU and memory to a pod. VPA can detect out-of-memory events and use this as a trigger to scale the pod. You can set both minimum and maximum limits for resource allocation, ensuring that you keep control of the ever-important cost budgets.

While it can be a helpful exercise to try to estimate resource usage by a pod, another way (over time) is to use the VPA Recommender feature of the VPA, which examines historical resource usage for pods and also takes into account out-of-memory events. Based on these data points, it makes recommendations on the requested values for pod configuration. It does not, however, recommend resource limits, so be careful how you set those to not monopolize resources.

## Scaling Clusters

Stepping beyond the realm of pods, we can take a look at how we manage scale in Kubernetes using Cluster Autoscaler (CA). Whereas the pod autoscaling is limited in how it operates by the available infrastructure resources provided by the cluster, the CA can interface directly with cloud providers (Azure, AWS and so on) and both request additional nodes to add to a cluster, or deallocate nodes from a cluster should the need arise.

![](https://cdn-images-1.medium.com/max/2000/1*4ugDsbdENFgALzpbEaRMTA.png)

Generally speaking, the CA operates by monitoring whether pods are in a pending state, which indicates a lack of available resources relative to computing demand. When pending, pods are literally waiting for cluster resources to do their work. Kubernetes can then request additional nodes and add pending pods to new nodes when available.

Likewise, CA can detect nodes that are no longer needed and scale down those resources.

CA also has the notion of expanders and cloud provider-specific logic to specify strategies for scaling nodes up or down, and PodDisruptionBudgets to configure how pods are added or removed to ensure that applications continue to run smoothly.

## Conclusion

With the ability to auto-scale both pods and clusters, Kubernetes is meeting the promise of the cloud: that there is built-in intelligence that can monitor the loading on a system and automatically scale up or down to meet the demand at a particular point in time.

When designing an architecture that uses container-based microservices with [**Kubernetes certification](https://onlineitguru.com/kubernetes-training.html)**, it is important to look at the overall picture and not just parts and available functionality in isolation. One of the challenges of running a Kubernetes deployment is being able to have all of the component parts of the ecosystem work together smoothly, especially when running hybrid systems, including multi-cloud and on-premises deployments.

Kublr simplifies and automates cluster deployment, management, and monitoring. Autoscaling can easily be defined from within the control plane which works across clouds and on-prem. Learn how Kublr will ease your Kubernetes deployment and download the free non-production licensed platform to require it for a check drive.

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and [Instagram](https://instagram.com/fauncommunity/) **📷 **and join our [Facebook](https://www.facebook.com/groups/364904580892967/) and [Linkedin](https://www.linkedin.com/company/faundev) Groups **💬**.**

**To join our community Slack team chat **🗣️ **read our weekly Faun topics **🗞️,** and connect with the community **📣** click here⬇**

![](https://cdn-images-1.medium.com/max/3000/1*6P3WpLjGv5v1ucm5dgkucg.png)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
