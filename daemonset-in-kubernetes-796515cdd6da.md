
# DaemonSet in Kubernetes

Image by Artie_Navarre from Pixabay

In this post, we will discuss DaemonSet, Use-Cases, How to create a DaemonSet, Upgrade and Delete it. Also, we will see how Daemonset will deploy the pods the moment the cluster is re-sized.

**What is DaemonSet?** The main reason to replicate the pod across all nodes is to have an agent or Daemon for a special purpose and to achieve the same, we will use DaemonSet in Kubernetes. Daemonset is a controller in Kubernetes which is different from ReplicaSet.It will assure a copy of the pod will run on all cluster nodes. The main difference between ReplicaSet and DaemonSet is that ReplicaSet can run multiple copies on a node and it can be decoupled from the node, whereas Daemonset will run only a single copy of the pod on a node.

**Use Cases: **Node monitoring, Log Collection & Cluster Storage

**Create a DaemonSet: **We have to mention the apiVersion, Kind as DaemonSet, metadata, and the spec with details of the container, label selectors, and Volumemounts. Below is the sample manifest which shows fluent bit pod deployed across all nodes in the cluster as DaemonSet

<iframe src="https://medium.com/media/de8bf768fc27f3620e3859456487bdab" frameborder=0></iframe>

Create a Kubernetes cluster and deploy the manifest, post-deployment we can see the DaemonSet across all the nodes in the cluster. In this example, I have used the GKE cluster -2 nodes. We can observe fluent-bit pods across 2 nodes in the cluster.

![](https://cdn-images-1.medium.com/max/2000/1*Ho_iWoLxRtYqal8u_TFSBQ.png)

**Re-Size the Cluster: **With resize of the cluster, we can observe that the pods are deployed into the new nodes and Daemonset Controller will monitor the add/removal of the nodes in the cluster and accordingly will deploy the pods in the new nodes.

We will increase the size of the cluster from 2 to 3 nodes

![](https://cdn-images-1.medium.com/max/2000/1*KtghIumeyxYeu4EY63WETQ.png)

With the re-size of the cluster, we could see that the Daemonset had deployed a fluent-bit pod on the new node.

![](https://cdn-images-1.medium.com/max/2000/1*ftl5sc8wwPt4NQnddNyLqg.png)

Also if we delete a pod explicitly, Daemonset controller will observe the current state and will deploy the necessary pods as per the desired state.

![](https://cdn-images-1.medium.com/max/2000/1*Lp6-xLMgi4lKEtO9UnlkEQ.png)

**Upgrade DaemonSet- **As explained in my previous article on [Rolling Upgrade](https://medium.com/faun/rolling-update-deployment-strategy-in-kubernetes-25e5b1566dd3), we can have a rolling upgrade of DaemonSet. This will create new pods by terminating the old one in a rolling fashion. In this post, we will see how we can upgrade the fluent-bit Daemon from 1.4.5 to 1.4.6 in a rolling fashion.

<iframe src="https://medium.com/media/8383628543afb4437d0cac37d735da88" frameborder=0></iframe>

![Old DaemonSet with 1.4.5](https://cdn-images-1.medium.com/max/2000/1*5rM3tuYU0vAv1ujrAZMzOQ.png)*Old DaemonSet with 1.4.5*

![Upgrade](https://cdn-images-1.medium.com/max/2114/1*svGzr5YGlL5cjuOG66OhnQ.png)*Upgrade*

![New Pod with 1.4.6](https://cdn-images-1.medium.com/max/2000/1*FtqewlZa8YYtr9MMgn-Viw.png)*New Pod with 1.4.6*

![All three pods created newly](https://cdn-images-1.medium.com/max/2000/1*rJsxg2-VPco-0ykx68gb4A.png)*All three pods created newly*

**Delete DaemonSet: **By deleting the daemonset, we will delete all the pods across all the nodes.

![](https://cdn-images-1.medium.com/max/2000/1*Ul_aFCq8701qicF8Ps0n6A.png)

**Source code: [https://github.com/vamsijakkula/daemonset](https://github.com/vamsijakkula/daemonset)**

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Subscribe to [FAUN topics](https://www.faun.dev/join?utm_source=medium.com/faun&utm_medium=medium&utm_campaign=faunmediumprebanner) and get your weekly curated email of the must-read tech stories, news, and tutorials **🗞️

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and [Instagram](https://instagram.com/fauncommunity/) **📷 **and join our [Facebook](https://www.facebook.com/groups/364904580892967/) and [Linkedin](https://www.linkedin.com/company/faundev) Groups **💬

![](https://cdn-images-1.medium.com/max/3000/1*_cT0_laE4iPcqW1qrbstAg.gif)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
