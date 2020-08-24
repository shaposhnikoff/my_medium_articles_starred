
# Kubernetes Best Practices — Security



There’s no doubt that Kubernetes adoption has increased a lot since its first release. Kubernetes is insecure by design and the cloud only makes it worse. Not everyone has the same security needs, and some developers and engineers might want more granular control on specific configurations. Kubernetes offers the ability to enforce security practices as you need, and the evolution of the available security options has improved a lot over the years.

Today’s post covers a few suggestions on what can you do to make your Kubernetes workloads more secure. I won’t go deep in many of the topics-some of them are pretty straightforward, while others need more theory. But I’ll make sure you at least get the idea of why and when you need to implement these practices. And I’ll provide links for further reading in case any of these are the right fit for you.

![](https://cdn-images-1.medium.com/max/2000/0*f1xU2irUKwttn-5N)

Enough boring words. Let’s get into the details.

### Disable public access

Avoid exposing any Kubernetes node to the internet. Aim to work only with private nodes if you can. If you decide to run Kubernetes in the cloud and use its managed service offering, you can disable public access to the API’s control pane. Don’t think about it, just disable it. An attacker that has access to the API can get sensitive information from the cluster. You can either use a bastion host, configure a VPN tunnel, or use a direct connection to access the nodes and other infrastructure resources. And in the cloud, look for disabling servers’ metadata from pods with network policies-more on this later.

If you need to expose a service to the internet, use a load balancer or an API gateway and enable only the ports you need. Always look to implement the least-privileged principle, and close everything by default.

![](https://cdn-images-1.medium.com/max/2000/0*uKE7hOmuLpqxDdIU)

## Implement role-based access control

Stop using the “default” namespace and plan according to your workload permission needs. Make sure that [role-based access control (RBAC)](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) is enabled in the cluster. RBAC is simply an authorization method on top of the Kubernetes API. When you enable RBAC, everything is denied by default. But you’ll be able to define more granular permissions to users that will have access to the API. You’d first start by creating roles and assigning users to those roles. A role will contain only allowed permissions, like the ability to list pods, and its scope applies to a single namespace. You can also create cluster roles where the permissions apply to all namespaces.

I suggest you read the [official docs](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) for RBAC in Kubernetes to learn more about its capabilities and how to implement it in your cluster.

## Encrypt secrets at rest

Kubernetes architecture uses ETCD as the database to store Kubernetes objects. All the information about the cluster, the workloads, and the cloud’s metadata is persisted in ETCD. If an attacker has control of etcd, they can do whatever they want-such as revealing secrets for database passwords or accessing sensitive information. Since Kubernetes 1.13, you can [enable encryption at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/). Backups will be encrypted, and attackers won’t be able to decrypt the secrets without the master key. A recommended practice is to use a key management service (KMS) provider like HashiCorp’s Vault or AWS KMS.

![](https://cdn-images-1.medium.com/max/2000/0*HBQuCcPVA9kt1m8j)

## Configure admission controllers

After a request to the Kubernetes API has been authorized, you can use an [admission controller](https://kubernetes.io/blog/2019/03/21/a-guide-to-kubernetes-admission-controllers/) as an extra layer of validation. An admission controller may change the request object or deny the request. As Kubernetes usage grows in your company, you need to enforce specific security policies in the cluster automatically. For example, enforce that containers always run as unprivileged users or that containers pull images only from authorized image repositories and enforce the usage of images that you’ve analyzed before. You can find other policies [on the official Kubernetes docs site](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#what-does-each-admission-controller-do).

![](https://cdn-images-1.medium.com/max/2000/0*Pg5eGKxj2PWp3Bpo)

## Implement networking policies

Similar to admission controllers, you can also configure access policies at the networking layer for pods. [Networking policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) are like firewall rules to pods. You can limit access to pods through label selectors, similar to how you might configure a service by defining label selectors for which pods to include in the service. When you set a network policy, you configure the labels and values a pod needs to have to communicate with a service. Another notable scenario is the one I mentioned before about the attacker accessing instance metadata in the cloud. You can define a network policy to deny egress traffic to pods, limiting the access to the instance metadata API.

## Configure secure context for containers

Even if you’ve implemented all of the previous practices I mentioned before, an attacker can still do some damage through a container. Because of Kubernetes and Docker architectures’ nature, someone could potentially have access to the underlying infrastructure. For that reason, make sure you run containers with the privileged flag turned off.

[There are other tools and technologies](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) you can use to increase security in the cluster by adding another layer of protection like [AppArmor](https://kubernetes.io/docs/tutorials/clusters/apparmor/), [Seccomp](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/), or [gVisor](https://gvisor.dev/). These types of technologies help by sand-boxing containers to run securely in regards to other tenants in the system. Although these are still emerging practices, it’s worth it to keep them in mind.

![secure kubernetes](https://cdn-images-1.medium.com/max/2000/0*mthieBGqn_1_5eXf)*secure kubernetes*

## Segregate sensitive workloads

Another option is to use Kubernetes features like [namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/), [taints, and toleration](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/) to segregate sensitive workloads. You can apply more restrictive policies and practices to those workloads where you can’t afford the luxury of a data breach or service downtime. For instance, you can tag a cluster of worker nodes (node pool) and restrict who can schedule pods to those nodes with RBAC roles.

## Keep base images small

Start with the leanest most viable base image and then build your packages on top so that you know what’s inside.

Smaller base images also reduces overhead. Your app may only be about 5 mb, but if you blindly take an off-the-shelf image, with Node.js for example, it includes an extra 600MB of libraries you don’t need.

Other advantages of smaller images:

* Faster builds

* Less storage

* Image pulls are faster

* Potentially less attack surface

![](https://cdn-images-1.medium.com/max/2000/0*n0R-Y_ZMmTwUZK1B.png)

## Don’t use sidecars for bootstrapping!

Although sidecars are great for handling requests both outside and inside the cluster, Sandeep doesn’t recommend using them for bootstrapping. In the past bootstrapping was the only option, but now Kubernetes has ‘init containers’.

In the case of a process running in one container that is dependant on a different microservice, you can use `init containers` to wait until both processes are running before starting your container. This prevents a lot of errors from occurring when processes and microservices are out of sync.

Basically the rule is: use sidecars for events that always occur and use init containers for one time occurrences.

## Don’t use :latest or no tag

This one is pretty obvious and most people doing this today already. If you don’t add a tag for your container, it will always try to pull the latest one from the repository and that may or may not contain the changes you think it has.

![](https://cdn-images-1.medium.com/max/2730/0*eMRMwXuYjCQ5Jdyw.jpeg)

## Scan container images

Avoid using container images from public repositories like DockerHub, or at least only use them if they’re from official vendors like Ubuntu or Microsoft. A better approach is to use the Dockerfile definition instead, build the image, and publish it in your own private image repository where you have more control. But even though you can build your own container images, make sure you include tools like [Clair](https://github.com/coreos/clair) or [MicroScanner](https://github.com/aquasecurity/microscanner) to scan containers for potential vulnerabilities.

## Enable audit logging

At some point in time, your systems may get infected. And when that happens (or if it happens) you better have logs to find out what the problem is and how the attacker was able to bypass all your security layers. In Kubernetes, you can create [audit policies](https://kubernetes.io/docs/tasks/debug-application-cluster/audit/) to decide at which level and what things you’d like to log each time the Kubernetes API is called. Once you have logs enabled, you can work on having a centralized place to persist these logs. Depending on the tool you use to persist logs, you can configure alerts, send notifications, or use webhooks to automate a patch. For instance, you might set an immediate action like terminating existing pods in the cluster that could have been affected.

If you’re running in the cloud, you can enable audit logging in the control plane. This is true at least for the three major cloud providers: AWS, Azure, and GCP.

Last but not least, make sure you’re always running the latest version of Kubernetes. You can see the [list of vulnerabilities that Kubernetes has had in the CVE site](https://www.cvedetails.com/vulnerability-list/vendor_id-15867/product_id-34016/Kubernetes-Kubernetes.html). For each vulnerability, the CVE site has a score that tells you how bad the vulnerability is. Always plan to upgrade your Kubernetes version to the latest available. If you’re using a managed version from cloud vendors, some of them deal with the upgrade for you. If not, [Google published a post](https://cloud.google.com/blog/products/gcp/kubernetes-best-practices-upgrading-your-clusters-with-zero-downtime) with a few recommendations on how to upgrade the cluster with no downtime. It doesn’t matter if you’re not running on Google-all the advice they give applies independently of where you’re running Kubernetes.

![](https://cdn-images-1.medium.com/max/2000/0*_y_J2Rt3GJvN1ben)

This strategy of course will only work if you doing http or web stuff and it won’t work for UDP or TCP based applications.

## Type: Nodeport can be “good enough”

This is more of a personal preference and not everyone recommends this. NodePort exposes your app to the outside world on a VM on a particular port. The problem with it is it may not be as highly available as a load balancer. For example, if the VM goes down so does your service.

## Use Static IPs they are free!

On Google Cloud this is easy to do by creating Global IPs for your ingress’. Similarly for your load balancers you can use Regional IPs. In this way, when your service goes down you don’t have to worry about your IPs changing.

## Map External Services to Internal Ones

This is something that most people don’t know you can do in Kubernetes. If you need a service that is external to the cluster, what you can do is use a service with the type ExternalName. Now you can just call the service by its name and the Kubernetes manager passes you on to it as if it’s part of the cluster. Kubernetes treats the service as is if it is on the same network, but it sits actually outside of it.

![](https://cdn-images-1.medium.com/max/2000/0*aVNradmhcpymUKch)

## What now?

That covers a good range of Kubernetes security best practices that everyone should consider. As you’ve noticed, I didn’t discuss many of the topics in too much detail, and by the time you’re reading this post, Kubernetes might have published another feature to increase security. For example, admission controllers is a feature that went live a few months ago. If you’d like to dive deeper into the current state of Kubernetes security options, I’d suggest you go and read the [Kubernetes official documentation](https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/) for more in-depth recommendations on securing a cluster. Also you can learn about kubernetes more by preparing for CKAD or CKAA examinations by Cloud Native for Kubernetes.

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
