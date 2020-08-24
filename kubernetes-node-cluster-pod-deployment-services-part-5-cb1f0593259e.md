Unknown markup type 10 { type: [33m10[39m, start: [33m69[39m, end: [33m78[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m94[39m, end: [33m105[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m115[39m, end: [33m122[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m19[39m, end: [33m30[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m66[39m }

# Kubernetes: Node, Cluster, Pod, Deployment & Services Part 5

Kubernetes: Node, Cluster, Pod, Deployment & Services Part 5

When it comes to Kubernetes, mainly we have to talk about 4 components are Node, Cluster, Pod, Deployment & Service. There are some techniques and use cases to use them in depends on the situation. Through this article You will learn each one and how connects each other together.

## Node

The node can be a single machine, a single cloud server or any type of single virtual machine. it has its own CPU capacity, RAM capacity and other functionalities where common computer machine has.

Top Cloud service providers :

1. [AWS EC2](https://aws.amazon.com/ec2/)

1. [Google Cloud Platform](https://cloud.google.com/)

1. [Azure](https://azure.microsoft.com/en-us/)

1. [Digital Ocean](https://cloud.digitalocean.com)

## Cluster

Combining multiple nodes as together called a cluster. When the cluster is created, it supposed to distribute tasks on one of the nodes and that node provides computer functionalities to run that task.

![](https://cdn-images-1.medium.com/max/2160/1*FSdHsE88glKsBdJau7yR1Q.png)

With Kubernetes, it manages each node in the cluster as one unit and responsible to share the workload among nodes.

## Pods

— — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — -

In Kubernetes, the pod is the smallest deployable unit. It can be wrapped one or more containers inside the single pod. containers that are inside the pod share the same resources and network.

![](https://cdn-images-1.medium.com/max/2160/1*s_obmSDgj5PULX6BDRPdrw.png)

each pod has an IP address that can be used to talk between pods. that’s why we use the calico network plugin. Industry-level, Pods use for development purposes and very rarely use in production.

In my opinion, Try to avoid pods. Mainly pod itself cannot handle much load.

Let's see how to create a pod and apply it.

    $ kubectl run nginx --image=nginx --restart=Never --port 80   --dry-run -o yaml > nginx-pod.yaml

<iframe src="https://medium.com/media/408825536b598559b7d1f1354d1e72a9" frameborder=0></iframe>

Here we get nginx docker image and create nginx pod configuration by **--dry-run **parameter with **kubectl run **command. **dry-run **responsible not to run nginx container.

After running that **kubectl run** command, you can see **nginx-pod.yaml a **file like this.

You can change the name of the container if you want and finally, you can run the pods by running below command.

    $ kubectl apply -f nginx-pod.yaml

To check pod is up and running, run below command,

    $ kubectl get pods

Also, You can learn about Multiple containers inside a single Pod from [this article](https://www.mirantis.com/blog/multi-container-pods-and-container-communication-in-kubernetes/).

## Deployment

— — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — -

Deployment is the next level of Pod. Deployment manages pods’ Life cycle. Anything which comes to pod goes through the deployment component. Responsible for monitoring. One of the main differences in deployment is to control the number of replicas. You do not need to maintain it manually when you use deployment.

![](https://cdn-images-1.medium.com/max/2000/1*8D4NNejM0N68I7YekxxzPA.png)

Let’s see how to create a proper deployment for NGINX image.

    $ kubectl run nginx --image=nginx **--restart=Always** --replicas=3 --port 80   --dry-run -o yaml > nginx-deployment.yaml

    **or**

    $ kubectl run nginx --image=nginx --replicas=3 --port 80   --dry-run -o yaml > nginx-deployment.yaml

<iframe src="https://medium.com/media/e3a3a0932c3c9bed16bab9928166e7b2" frameborder=0></iframe>

To run this ***nginx-deployment.yaml***,

    $ kubectl apply -f nginx-deployment.yaml

To check running deployments,

    $ kubectl get deployments

Let’s get a real-world example;

## Here I will show you to deploy Nodejs service as a deployment.

First, you should build your docker image from the Nodejs project and push it into your Docker registry. If you use an AWS EKS Cluster, you can use ECR as your Docker registry where you can store your docker image. Also, you don't have to configure authentication between EKS and ECR, it is automatically done by EKS.

Think your NodeJs project name is **my-service-1,**

<script src=”[https://gist.github.com/SarasaGunawardhana/6108f522f102d5cf53192fe2d4640e0f.js](https://gist.github.com/SarasaGunawardhana/6108f522f102d5cf53192fe2d4640e0f.js)"></script>

From this file, you can see passing environment variables, also you should expose our container port.

run your deployment like this.

    $ kubectl apply -f nodejs-deployment.yaml

To check running deployments,

    $ kubectl get deployments

## Service

— — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — -

Kubernetes [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) are mortal. They are born and when they die, they are not resurrected. If you use a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) to run your app, it can create and destroy Pods dynamically.

Each Pod gets its own IP address, however, in a Deployment, the set of Pods running in one moment in time could be different from the set of Pods running that application a moment later.

This leads to a problem: if some set of Pods (call them “backends”) provides functionality to other Pods (call them “frontends”) inside your cluster, how do the frontends find out and keep track of which IP address to connect to, so that the frontend can use the backend part of the workload?

Enter *Services*.

If you want to communicate between services and want to communicate with outside the cluster, you have to define a service.

<script src=”[https://gist.github.com/SarasaGunawardhana/27ef85a354ca90f166c204e85ab459e0.js](https://gist.github.com/SarasaGunawardhana/27ef85a354ca90f166c204e85ab459e0.js)"></script>

we connect service and deployment from the **selector,**

you can learn more about the connection between Pods, deployment, and service from NEXT article.

If you like, Feel free to clap the button below a few times to show your support for the author! :D

Did you find this guide helpful? ***Make sure to subscribe to my newsletter so you don’t miss the next article with useful deployment tips!***

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
