
# Kubernetes components

When you deploy Kubernetes, you get a cluster.

A cluster may be a set of machines, called nodes, that run containerized applications managed by Kubernetes. A cluster has at least one worker node and one master node

The worker node(s) host the pods that are the components of the application. The master node(s) deals with the worker nodes and the pods in the cluster. Various master nodes are utilized to provide a cluster with failover and high accessibility.

This document outlines the different components you need to have a complete and working Kubernetes cluster.

if you want to gain more knowledge on Kubernetes go through this link [**Kubernetes course](https://onlineitguru.com/kubernetes-training.html)**

Here’s a diagram of a Kubernetes cluster with all the components tied together.

![](https://cdn-images-1.medium.com/max/2000/1*qqaBGr3hgS7mGAIDbxGbYg.png)

## **Master components**

Master components provide the cluster’s control plane. Master components make global decisions about the cluster (for example, scheduling), and that they detect and answer cluster events (for example, beginning a replacement pod when a deployment’s *replicas* field is unsatisfied).

Master components are often run on any machine within the cluster. However, for simplicity, found out scripts typically start all master components on an equivalent machine, and don’t run user containers on this machine. See Building High-Availability Clusters, for instance, multi-master-VM setup. Get more additional info At [**Kubernetes online training](https://onlineitguru.com/kubernetes-training.html)**

These master components comprise a master node:

* **Kube-API server. **Exposes the API.

* **Etcd.** Key-value stores all cluster data. (Can be run on the same server as a master node or on a dedicated cluster.)

* **Kube-scheduler.** Schedules new pods on worker nodes.

* **Kube-controller-manager.** Runs the controllers.

* **Cloud-controller-manager. **Talks to cloud providers.

## Node components

Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.

* **Kubelet.** The agent that ensures containers in a pod is running.

* **Kube-proxy.** Keeps network rules and perform forwarding.

* **Container runtime.** Runs containers.
[**What is Kubernetes | Kubernetes Orchestration | OnlineItguru**
*Kubernetes is a distributed, portable, extensible, open-source platform for container orchestration to manage apps…*onlineitguru.com](https://onlineitguru.com/blogger/what-is-kubernetes)

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and [Instagram](https://instagram.com/fauncommunity/) **📷 **and join our [Facebook](https://www.facebook.com/groups/364904580892967/) and [Linkedin](https://www.linkedin.com/company/faundev) Groups **💬**.**

**To join our community Slack team chat **🗣️ **read our weekly Faun topics **🗞️,** and connect with the community **📣** click here⬇**

![](https://cdn-images-1.medium.com/max/3000/1*6P3WpLjGv5v1ucm5dgkucg.png)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
