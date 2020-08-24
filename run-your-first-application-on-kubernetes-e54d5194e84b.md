
# Run your first application on Kubernetes



## **Introduction**

Nowadays, most applications are built from smaller pieces called microservices. Those microservices are deployed by developers many times a day. There’s a need for a tool which would allow us to maintain, deploy, and manage the system without any downtime. What should we do to achieve those restrictive requirements? None of us wants to experience a breakdown of the production system. What tools should we use to ensure the high availability of our application? For this purpose, we can use Kubernetes, which is a containerization management tool. Kubernetes helps us manage and deploy newer versions of our application without any downtime.

Kubernetes (k8s) is a platform created by Google, which was released in 2014 for the wider community under open source license. It’s used for running and managing container services. The platform improves deployment, and it also provides a simple way of scaling the application. Kubernetes helps to eliminate many steps in the manual processes related to deploying and scaling distributed systems.

The entire code included in this article is also available on [GitHub](https://github.com/lunarwings/kubernetes-series).

## **Cluster Architecture**

What is a cluster? How does it work? You should think about the cluster as a whole system, instead of individual machines because it takes care of handling and distributing your programs into particular nodes. Kubernetes connects many servers into one cluster, so it’s a group of nodes connected together. Cluster consists of two elements **Master (Control Plane)** and **Worker Nodes**. Control Plane is the brain of Kubernetes cluster. It is responsible for managing a cluster and also for coordinating worker nodes, on which all services are running. The main tasks of the Control Plane are serving API requests, scheduling containers, managing services. Thanks to these components, Kubernetes can do its job.

![](https://cdn-images-1.medium.com/max/2000/0*14iDPE6FZdDWTFOu)

The **Control Plane** is made of these components:

* **etcd **is a database where Kubernetes store all its information of the nodes. It keeps a configuration data, the actual and desired state of the system, and its metadata inside.

* **kube-scheduler **is a component that decides where to run newly created Pods.

* **kube-controller-manager **is responsible for running resource controllers, such as DaemonSets, Deployments, ReplicaSets, etc.

* **kube-apiserver **is a frontend server of the control plane, which handles API requests.

* **cloud-controller-manager **runs controllers that interact with the cloud provider.

**Worker Node** is responsible for deploying and running application containers, which means that the worker node has all the necessary services to manage the communication between containers. Worker node communicates with the Control Plane and assigns node resources for containers, then containers are installed to particular nodes.

Every Worker node of the Kubernetes cluster runs these components:

* **kubelet **is responsible for managing the container runtime, to start scheduled tasks on a node and monitoring its status.

* **container runtime** starts and stops containers and handles their communication. Usually, Docker is used, but Kubernetes supports other container runtimes like rkt and CRI-O.

* **kube-proxy **is a network proxy that runs on the nodes and routes requests between Pods and the Internet.

## **Installation / Create Cluster**

We already know how the Kubernetes clusters work, so it’s time to move to the tools that we’ll use.

* [**Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) **is a command-line interface for running commands which will be processed on Kubernetes clusters. With **kubectl**, you can deploy applications, check and manage cluster resources, view logs and much more.

* [**Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)** creates and implements the environment of the Kubernetes physical cluster in your local environment.

* [**VirtualBox](https://www.virtualbox.org/)** is a cross-platform virtualization tool.

After installing these tools, we can proceed to create our local cluster using **minikube**.

Let’s first check the version of **minikube **by typing **minikube version** command.

    $ minikube version
    minikube version: v1.3.1

As we can see, we have installed version **v1.3.1 **of **minikube**.

Now we can create our cluster in a virtual machine - let’s do this by typing **minikube start **and** minikube status**.

    $ minikube start
    😄  minikube v1.3.1 on linux (amd64)
    🔥  Creating virtualbox VM (CPUs=2, Memory=2048MB, Disk=20000MB) ...
    🐳  Configuring environment for Kubernetes v1.15.2 on Docker 18.09.8
    🚜  Pulling images ...
    🚀  Launching Kubernetes ...
    ⌛  Verifying: apiserver proxy etcd scheduler controller dns
    🏄  Done! kubectl is now configured to use "minikube"

    $ minikube status
    host: Running
    kubelet: Running
    apiserver: Running
    kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100

Let’s verify if our cluster works and if **kubectl** can communicate with cluster - and for that, we’ll use **kubectl cluster-info** command.

    $ kubectl cluster-info
    Kubernetes master is running at [https://192.168.99.100:8443](https://192.168.99.100:8443)
    KubeDNS is running at https://192.168.99.100:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

As we can see above, we have communication with our cluster, and we also can see Kubernetes components installed along with the API server. You can also connect to **minikube** virtual machine to see what processes are running on the node. To do this type **minikube ssh** on the command line.

Now we can check the list of available nodes on the cluster - let’s type **kubectl get nodes** in the terminal.

    $ kubectl get nodes
    NAME       STATUS   ROLES    AGE     VERSION
    minikube   Ready    master   3m41s   v1.14.3

As we can see above, we have one virtual node on the cluster on which we’ll be installing container with our application. You can also use the **kubectl describe node minikube** command to check a more detailed description of the node.

Let’s label this node so we can improve nodes management later. Type **kubectl label nodes minikube type=backend **command.

    $ kubectl label nodes minikube type=backend
    node/minikube labeled

To verify that label is added correctly type **kubectl get nodes --show-labels** command. Later I’ll explain what we can use that label for.

    $ kubectl get nodes --show-labels
    NAME     STATUS ROLES  AGE   VERSION  LABELS
    minikube Ready  master 6m36s v1.14.3

    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=minikube,kubernetes.io/os=linux,node-role.kubernetes.io/master=,type=backend
> **INFO -** By default, you can only create one virtual node using the **minikube**. In future articles, we’ll be using more instances. For that purpose, we’ll be using one of the managed Kubernetes services provided by Google (Google Kubernetes Engine) or Amazon (Elastic Kubernetes Engine). Those services allow us to scale application to a more significant number of nodes. It’ll be much easier to run a cluster using managed Kubernetes services instead of self-hosted one. Furthermore, Kubernetes management services lets you decrease the number of administrative tasks related to setting up and running Kubernetes clusters (particularly the Control Plane).
> **TIP -** You’ll be using **kubectl** command very often. You can make your life easier and add a shortcut. If you have never used shell aliases before, add the following line **alias k=kubectl** to the **bash_profiles** - that file is located at home directory named **.bashrc**. After saving changes you have to reload bashrc by typing **source ~/.bashrc** in the command line. Now instead of typing out **kubectl** for every command, you can just use **k **like **k get nodes.**

## **Create a simple application**

Cluster is ready to deploy our apps, so now we start creating the first app. It’ll be a simple REST API, from which we’ll build a Docker image. But first, let’s start with the application itself. To write our application we’ll use the Go programming language.

If you use Ubuntu, you can install Go using snap packages or download it from the [official site](https://golang.org/). After complete installation, create a working directory, then inside of it add another directory named **main **and place the file **hello.go** into it. The directory structure should look like this:

    <working directory>/main/hello.go

In the **hello.go** file, add the following code, which will allow us to run a simple server that we’ll use to test our deployment.

<iframe src="https://medium.com/media/eb362e470cdbc34d33bc1628fa4ddcd2" frameborder=0></iframe>

Now we can run our application - in the main directory type the following command **go run hello.go** to run our simple server. When you visit the browser at [**http://localhost:8080/api/hello](http://localhost:8080/api/hello), **you should see “Hello World”. Now we know that our application works so we can go to the next step - packing the application into the Docker container.

## **Containerize application**

To run an application in a cluster, you have to pack our app into a container, then create Docker image from the container and finally send it to the Docker registry. After that, you need to define which image should be installed on Kubernetes node. Kubernetes uses a mechanism called Pod to manage containers.

**Pod **is one or more linked containers, which runs on the worker nodes. Each of these Pod objects has its own logical IP address, the process that corresponds to the running application and also a hostname. All containers in the Pod resource operate on the same logical machine, while containers in other Pod are isolated from each other, even if they operate on the same node. Pod also has a self-healing mechanism - it runs underhood a health check for the container. That kind of feature can replace Pods that have failed or stopped responding for any reason.

**Docker **is a tool designed to make creating, deploying and running application easier by using containers. Docker is a container runtime library - it manages the life cycle of containers. It’s a command-line tool for packaging and running containers, and have an API for container management. Docker is one of the components used by Kubernetes.

Let’s now create a Docker image for our application. In the main directory add a file named **Dockerfile**.

    <working directory>/main/Dockerfile

The Dockerfile describes exactly what is needed to create the container image.** **It allows Docker to read instructions which should be executed. Dockerfile can use other previously created images, and we’ll be using Go image as an extension.

In the Dockerfile, add the following:

<iframe src="https://medium.com/media/25aed9d6b7bf3b259bfcca643d87f596" frameborder=0></iframe>

This **Dockerfile** describes multi-stage build. The first stage uses an official Golang container image, which is the Alpine Linux operating system with the Go language environment installed inside. In the next step, it runs go build command to compile **hello.go** file we’ve created earlier. The output will be a binary file named hello. The second stage takes a scratch image, which is a completely empty container image and copies the **hello** file with the app into it.

Why do we need the second build stage? The first stage with the Alpine Linux is only needed to build the program itself. The second stage puts hello binary to the empty container because we only need compiled binary to run an application. The main reason is the size of the image - the second stage ensures that the image is tiny about 6-7 MiB. Furthermore, that’s the image that can be deployed in production.

Let’s assume that we skip the second stage and that you will end up with an oversized container. The image will be about 350 MiB in size, which is unnecessary and will never be executed. The conclusion is, the smaller the container image, the faster it can be uploaded and downloaded, and the faster it will take to start up the Pod. Minimal containers also have reduced security issues. If there are less programs inside your container, then there is less chance of potential vulnerabilities. So remember, always keep your containers the smallest as you can.

Now we can start building the image. For that, we’ll use **docker build** command. Usually, after creating an image, we would like to store that image in some registry. To achieve that we’ll need to send an image to the Docker image registry like **Docker Hub,** **ECR **from Amazon or **GCR **from Google. However, in this article, I’ll show you how to use Docker, which is already installed in the Minikube virtual machine environment. Thanks to that, we’ll be able to share that image in the context of Minikube. Let’s go to the next chapter, where we’ll prepare deployment of our application, and after that, we’ll come back to building the image itself.

## **Deploy our application**

How can we run the application on a node in the cluster? On which node shall we allocate our container? First of all, we need to figure out what **Deployment Controller** is. From the previous section, you know that Pod represents one or more linked containers on the node. Deployment Controller is used to scale the application and is responsible for managing and creating new Pods. Deployment Controller can run many replicas of containers with the processes inside. If we have more than one node, we can reschedule Pods to different nodes (each node should be in a separate zone, which means that each node is launched in a different geographical area). This is required to ensure high availability.

When Deployment Controller is created Kubernetes master sends the information about which node should create Pod or group of Pods. Having knowledge of what is Deployment and Pod, so let’s prepare a Deployment.
> **TIP -** Kubernetes Deployment Controller can be created using the **kubectl run** command or by configuring a **YAML **file (which is the recommended way).

In the working directory, create the **hello-app-deployment.yaml** file and add the following instructions.

<iframe src="https://medium.com/media/28ebec7f1d2670f503451693e753c036" frameborder=0></iframe>

Here’s a brief explanation of fields:

* **apiVersion** - it specifies the Kubernetes API version.

* **kind** - it specifies a type of Kubernetes object/resource. There are many types like Deployment, Pod, Service, Replicasets, Secret, Jobs and much more. To see more details you can use **kubectl explain <resource>** command.

* **metadata** - data that helps specify unique information about the object. Kubernetes resource metadata could be used to delete a resource (here we have a label **hello-app**). If you want to delete that deployment controller, you could simply type **kubectl delete deployment -l app=hello-app**, but remember deleting deployment controller, removes all the objects defined by its configuration.

* **spec** - contains the actual description of the Pods contents, such as the containers, volumes, ports, etc.

* **template** - describes all instructions executed in the Pod.

* **replicas -** tells Kubernetes how many replicas (Pods) to create and distribute across the worker nodes.

* **labels** - adds a label to a Deployment, it is used for identifying objects within k8s.

* **selector **-** **defines which Pods should be managed by the Deployment.

* **nodeSelector** - that selector is used to label a node, so if you have the node labeled, all Pods with that label will be spread across nodes with that specified label. For more advanced scheduling you should use **nodeAffinity** and **podAntiAffinity** rules.

Ok, let’s create Deployment Controller - we’ll use the **kubectl create** command.

    $ k create -f hello-app-deployment.yaml
    deployment.apps/hello-app created

When we run **kubectl get pods**, we can see that our Pod is just created (check container creation state).

    $ k get pods
    NAME                         READY   STATUS         RESTARTS   AGE
    hello-app-74bbd87f67-4rxln   0/1     ErrImagePull   0          6s

When we run **kubectl get pods** again, we’ll see that something went wrong, because the status of Pod has changed to state **ErrImagePull **and then into **ImagePullBackOff**.

    $ k get pods
    NAME                         READY   STATUS         RESTARTS   AGE
    hello-app-74bbd87f67-4rxln   0/1     ErrImagePull   0          7s

    $ k get pods
    NAME                         READY   STATUS             RESTARTS   AGE
    hello-app-74bbd87f67-4rxln   0/1     ImagePullBackOff   0          2m47s

We already know from the state that Pod couldn’t download the image. Let’s look closer to the problem by checking logs - type the following command **kubectl logs <pod name>**, and **kubectl describe pod <pod name>**.

    $ k logs -l app=hello-app
    Error from server (BadRequest): container "hello-app" in pod "hello-app-74bbd87f67-4rxln" is waiting to start: image can't be pulled

    $ k describe po hello-app-74bbd87f67-4rxln
    .......................
    Events:
    Type     Reason     Age                    From              Message

    ----     ------     ----                   ----               ------
    Normal   Scheduled  4m2s                   default-scheduler  Successfully assigned default/hello-app-74bbd87f67-4rxln to minikube

    Normal   Pulling    2m29s (x4 over 4m1s)   kubelet, minikube  Pulling image "hello-app:v.01"

    Warning  Failed     2m26s (x4 over 3m58s)  kubelet, minikube  Failed to pull image "hello-app:v.01": rpc error: code = Unknown desc = Error response from daemon: pull access denied for hello-app, repository does not exist or may require 'docker login'

    Warning  Failed     2m26s (x4 over 3m58s)  kubelet, minikube  Error: ErrImagePull

    Warning  Failed     2m1s (x6 over 3m57s)   kubelet, minikube  Error: ImagePullBackOff

    Normal   BackOff    107s (x7 over 3m57s)   kubelet, minikube  Back-off pulling image "hello-app:v.01"

Our Pods are not able to download the image. We must first make an image available (in the context of the virtual machine that Minikube uses).

To switch Docker to the context of Minikube, enter the following command on the command line:

    eval $(minikube docker-env)

Now we can go to build the image - run the command in the directory with Dockerfile.

    $ docker build -f Dockerfile -t hello-app:v.01 .
    Sending build context to Docker daemon  3.072kB

    ....................

    Successfully built 9a6392b79abd

    Successfully tagged hello-app:v.01

Let’s check the list of available images with **docker images** command.

    $ docker images
    REPOSITORY TAG   IMAGE ID            CREATED              SIZE
    hello-app  v.01  9a6392b79abd        About a minute ago   6.51MB
    ..............................

As we can see on the list, our application is available in the repository called **hello-app** and tagged as** v.01**.
> **Important note**: You have to run **eval $(minikube docker-env)** on the terminal you want to use because it only sets the environment variables for the current shell session. If you’re going to disable Minikube Docker host use:

    eval $(minikube docker-env -u)

After a while, the Pod should change state to **Running**. You can also run **kubectl delete pod <pod name>** to delete the Pod. If you delete Pod manually, the Pod starts a termination, and after a while deployment controller creates a new one.

    $ k get pods
    NAME                         READY   STATUS    RESTARTS   AGE
    hello-app-74bbd87f67-4rxln   1/1     Running   0          18m
> **TIP **- To see more detailed information about Deployment like image, replicas, ports, status etc. run **kubectl describe deployment hello-app-deployment** command.

Ok, since we already know that Pod works, let’s check if application responds. We can use port forwarding to check if our app works by running **kubectl port-forward <pod name>**.

    $ k port-forward hello-app-74bbd87f67–4rxln 8080:8080
    Forwarding from 127.0.0.1:8080 -> 8080
    Forwarding from [::1]:8080 -> 8080

Now we have access to our application at [**http://localhost:8080/api/hello](http://localhost:8080/api/hello)**
> **INFO -** **port-forward** command is mainly for debugging, also grants you temporary access to this application on your local machine. If you need to connect to the application under a specific port and you do not want to expose it using the **Ingress** object, then you should use the **port-forward** command. Normally you would use Load Balancer service to expose traffic to each of your services, but **Ingress **allows you to create a way to route requests to particular services through one entrypoint.

## **Summary**

In this article, we learned what the outline of the Kubernetes architecture looks like, what is Minikube, what are the essential components of Kubernetes, and the basic kubectl** **commands. We managed to create a local Kubernetes cluster using Minikube. We also know how to pack our application in the Docker container and install it on the cluster using Deployment Controller.

In subsequent articles, we will take a closer look at the Kubernetes components. We’ll talk about topics such as:

* How to trigger database migrations using Kubernetes Job and Helm hooks?

* Sharing and using images in the Docker image registry

* How Pod manages the resources of a node?

* How to limit the resources for each Pod?

* What are the liveness and readiness probes?

* What is Ingress and how we can share our application to the whole world?

* How to automate deployment using Helm?

* How to use Let’s Encrypt with Kubernetes

Thank you for reading this article. If you would like to learn new exciting things about Kubernetes stay tuned and follow my profile.
