
# ReplicationController in Kubernetes

Image by Christine Schmidt from Pixabay

## **Overview**

ReplicationController *a.k.a **rc** *ensures that specified number of pods are running all the time in the cluster *.*If there are more number of pods they will be killed and vice-versa. Pods will be replaced automatically if they are maintained by a Replication Controller in case of any crash/failure, deletion or termination. *Labels* associate replication controllers with the pods.

Common Usage patterns of ReplicationController are Rolling Upgrades,Re-Scheduling ,Scale-up and down based on application load.

**Deploying a Sample *ReplicationController***

In this example , we will deploy a sample ***hellowhale*** application through ReplicationController. It’s a simple web app displaying the official docker Blue Whale image.

![Hellowhale Replication Controller Manifest](https://cdn-images-1.medium.com/max/2000/1*Ts-3CwOH9cQ-rbai_CBZIg.png)*Hellowhale Replication Controller Manifest*

Replication controller will typically contain 3 main items ***kind,metadata and spec*** . Kind specifies the kind of the object that is getting created, here in this case it’s ***ReplicationController**.* Metadata specifies the name of the object(i.e. hellowhale-rc) . Spec will specifies the number of replicas(i.e. 3 in our example) and the pod specification under template.

As stated above ReplicationController will be associated with the pods through labels and selectors ( i.e. hellowhale-app in our example).

***ReplicationController Deployment on Kubernetes Cluster***

Ensure we have minikube cluster or any GKE cluster up and running.

![**ReplicationController Deployment**](https://cdn-images-1.medium.com/max/2000/1*clh6vNDfhnA70nd2ts3mZg.png)***ReplicationController Deployment***

![**ReplicationController created 3 hellowhale pods**](https://cdn-images-1.medium.com/max/2000/1*w90vjSyqyUsRf6OUeo0wuw.png)***ReplicationController created 3 hellowhale pods***

![**ReplicationController -Desired and Current Status**](https://cdn-images-1.medium.com/max/2000/1*ISIGiZs87vzjx8ME1T9dGQ.png)***ReplicationController -Desired and Current Status***

![**Describe the ReplicationController Status**](https://cdn-images-1.medium.com/max/2092/1*CpKyRjACJZOXUWiGPCuHBg.png)***Describe the ReplicationController Status***

Describe command shows the current status of the ReplicationController and Pod Status -i.e. Name,Age, Current Status , Selector and Labels used.

Access the pods using NodePort Service.

![**Service Definition**](https://cdn-images-1.medium.com/max/2000/1*Ho2sL0xHrPM8QjlxTsaGmA.png)***Service Definition***

![](https://cdn-images-1.medium.com/max/2000/1*gFgLGtfXBR9KLzmI6YsTxg.png)

Access the homepage from browser using the nodeip which is Minikube IP Address( 192.168.99.100) and the NodePort(31111) as defined and exposed through the service.

![](https://cdn-images-1.medium.com/max/2000/1*rDDfPzszlJRVXf7qwhCeEg.png)

Let’s try to scale-up and down the pods using the ReplicationController. 3 pods are scaled to 5 instances now.

![Scale-up to 5 Replicas using rc.](https://cdn-images-1.medium.com/max/2000/1*a8fvg-iSNDi1zfrpfQye9g.png)*Scale-up to 5 Replicas using rc.*

![Scale-down of pods](https://cdn-images-1.medium.com/max/2000/1*664m8ONhOFeD2lZBoIUz8Q.png)*Scale-down of pods*

Delete & Clean the entire setup

![](https://cdn-images-1.medium.com/max/2000/1*9pA0JbwuCIte2-gp8sC17g.png)

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
