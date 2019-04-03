
# A Kubernetes quick start for people who know just enough about Docker to get by

A Kubernetes quick start for people who know just enough about Docker to get by

### What if I told you this is quite literally the Kubernetes guide you have been looking for?

If you live on this side of our solar-system, you are guaranteed to have heard of Kubernetes before. You may not know what it is exactly. But that’s okay. Even if you haven’t heard of it before, stick around and find out. It’ll live up to your expectations.

I promise Kubernetes is not a crazy hamster at the helm of a catamaran. For all you know and care it might as well be, but no, it’s not, really. However, that would be awesome to see.

### TL;DR

If you want to jump to relevant topics covered in this guide feel free to do so by pressing any of the links below.

1. [Kubernetes is…?](#b5d6)

1. [Clusters…?](#37c7)

1. [Creating your own local cluster](#3664)

1. [Setting up a production cluster in the cloud](#1eac)

1. [Deploying to Kubernetes](#8787)

1. [Creating a Node.js and MongoDB deployment](#903e)

## Kubernetes is…?

Does anyone know what *“kubernetes”* even means? Apparently, if there are Greek readers, you’ll know. It means helmsman or pilot. How does this relate?
> # Kubernetes is a portable, extensible open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation.

Whoa… Okay, let’s break that down. Kubernetes, also called K8s, is a system for automating deployment, scaling and management of containerized applications.

It was developed by Google, and announced in mid-2014. Many of the key developers were previously working on Google’s in-house orchestration system called *“Borg”*. Following the release of Kubernetes v1.0 in the summer of 2015, Google open-sourced it and partnered with the Linux Foundation in a joint effort to advance the technology.

Let’s break down Kubernetes even further. Imagine it like this. You’ve created a Docker container to wrap your application. But you still need to manage the various DevOps tasks for deploying and managing your containers. Here’s where Kubernetes steps in. It will handle all the crucial deployment, scaling and management steps so you don’t have to.

It’s essentially a tool for managing your containers. A platform of sorts that holds your hand along the way. It’s up to you to create a network of intertwined containers, tell Kubernetes what to do, and how to serve your containers.

### Why do I need Kubernetes…?

Easy! Peace of mind, period.

You don’t have to worry about every instance you own. Nor, do you need to worry about whether the containers are running. If an instance fails, Kubernetes will re-create the containers of the failed instance on one that’s running. Fabulous!

## Clusters…?

In the sense of Kubernetes, a cluster would be a coupled network of containers connected in such a way they can freely communicate with each other.

Not to dive too deep into the inner working of Kubernetes, here’s what we need to know.

### Nodes

The machines on which a cluster is running can either be Masters or Nodes. The naming makes sense. The Master is the control panel of the whole cluster. All commands we will run will be run on the Master instance. It will then decide which Node, or worker machine, in the cluster will take the workload.

### Network

In order to enable the communication between the various containers in the cluster we need a network to provide IP addresses to them. Luckily, Kubernetes has a wide variety to choose from, and thankfully they work like magic. [Here’s](https://kubernetes.io/docs/concepts/cluster-administration/networking/#summary) a more detailed explanation. In all essence, such a network enables the pods in a cluster to talk to each other.

### Interaction

I’m sure you’re now wondering how the nodes communicate with each other. Well, every Node has a **Kubelet**, which is an agent for managing the Node and communicating with the Kubernetes Master. All of this is glued together with the **Kubernetes API** which the Master exposes. We then use the Kubernetes API to directly interact with the cluster using a CLI tool called **kubectl**. More about this further a bit down.

Apart from the API, what enables Kubernetes to work properly is a globally available configuration store called **etcd**. It’s a distributed key-value store that can be distributed across multiple nodes. Why is etcd so important? It stores configuration data for all of the nodes in the cluster so each and every part of it knows how to configure itself.

### Kubernetes objects

A Kubernetes object is a single unit that exists in the cluster. Objects include **deployments, replica sets, services, pods** and much more. We’ll focus on these main objects in the scope of this tutorial. When you create an object you’re telling the Kubernetes cluster about the **desired state** you want it to have.
> # *Kubernetes Objects* are persistent entities in the Kubernetes system. Kubernetes uses these entities to represent the state of your cluster.

Desired state means the cluster will work to keep it like you specified, even if a node in your cluster fails. Kubernetes will detect this and compensate by spinning up the objects on the remaining nodes in order to restore the desired state. With that understood, let’s define the objects we’ll be working with.

* [**Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/)** — a group of one or more containers (such as Docker containers), with shared storage/network, and a specification for how to run the containers. Even if the pod has several containers, they will all be reachable in the network through a single IP address.

* [**Service** ](https://kubernetes.io/docs/concepts/services-networking/service/)— an abstraction which defines a logical set of **Pods** and a policy by which to access them. Pods have a life cycle. They get created and die. We need a way to make them accessible on a regular basis, even if they are re-created. By giving Pods a certain label we use a Service to route traffic to all Pods with that particular label. Voila! Reliable access to Pods even if they are re-created.

* [**ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)** — give Pods a label and control their replication. Nowadays they are only used through Deployments.

* [**Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)** — describes the desired state and makes sure to change the actual state to the desired state if needed. A deployment manages Pods and ReplicaSets so you don’t have to. Just like magic!
> # The Deployment instructs Kubernetes how to create and update instances of your application. Once you’ve created a Deployment, the Kubernetes master schedules mentioned application instances onto individual Nodes in the cluster.

### Do you need Kubernetes clusters?

I can’t seem to find a single reason why not. Even if I had a single instance running on any of the many cloud providers out there, I’d go for a cluster setup. The deployments enable rolling updates, meaning zero downtime. I can have several instances of apps running at the same time, in parallel as well!

But, there’s always a *but*. It’s a bit fiddly to understand volumes and persistence. Containers are stateless after all. You need to create some sort of persistent storage, or use a 3rd party DBaaS. But, that’s all the craze now anyhow, with services such as [AWS RDS](https://aws.amazon.com/rds/) and [MongoDB Atlas](https://www.mongodb.com/cloud/atlas).

The peace of mind is what I like. Not having a headache at the end of the day is what floats my boat.

## Creating your own local cluster

We’ll use [Minikube](https://github.com/kubernetes/minikube) for development, a lightweight Kubernetes setup. It creates a tiny VM on your local machine, and deploys a Kubernetes cluster with a single node. There are a couple of steps we need to go through in order to install Minikube. Let’s jump in.

### 1. Install a **virtualization software**.

Either VirtualBox or KVM2 will do just fine. I’ve found using VirtualBox is the easiest way to go about this. [Here’s](https://www.virtualbox.org/wiki/Downloads) the downloads page where you can get started easily.

### 2. Install **kubectl**

Kubectl is the CLI tool for interacting with the Kubernetes cluster. [Jump over to the official page](https://kubernetes.io/docs/tasks/tools/install-kubectl/), pick your OS and run the provided commands. It’s really simple, here’s how you do it on Ubuntu. You can find the other samples on the link I added above.

    $ curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s [https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl](https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl)
    $ chmod +x ./kubectl
    $ sudo mv ./kubectl /usr/local/bin/kubectl

### 3. Install Docker

Docker will be tasked with creating and managing containers. Two more commands, if you’re on Ubuntu.

    $ sudo apt-get update
    $ sudo apt-get install -y docker.io

Otherwise, jump over [here for the Windows setup](https://docs.docker.com/docker-for-windows/install/), or [here if you have a Mac](https://docs.docker.com/docker-for-mac/install/).

### 4. Finally, install Minikube.

Only one more command for Ubuntu.

    $ curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

Jump over to the [official Minikube docs](https://github.com/kubernetes/minikube#installation) to find the commands for Mac and Windows.

### 5. Run Minikube

That’s it, you’ll have a Minikube CLI at your disposal. Go ahead and run minikube and you’ll get back the available commands. To start Minikube run, intuitively enough:

    $ minikube start

Your development cluster is up and running. To make sure it works, run a simple get nodes command.

    $ kubectl get nodes

The output should show you the basic cluster info, as you can see below.

    NAME       STATUS    ROLES     AGE       VERSION
    minikube   Ready     <none>    Xd        v1.9.0

With that running like a breeze, let’s see how to install Kubernetes on a server.

## Setting up a production cluster in the cloud

Feeling brave? I sure am. Let’s quickly go through the steps of installing Kubernetes on a real server in the cloud. For this endeavor you can choose any cloud provider you want.

I’ve used these very steps on DigitalOcean, and the installation takes less than 5 minutes. The only thing you need to keep in mind is to add all the VMs you provision to the same Virtual Private Network. They need to have private IPs through which they’ll communicate with each other.

From what we learned above, one instance will be the Master while all the others will be worker Nodes. Let’s jump into installing Kubernetes on the Master first.

### Installing Kubernetes on the Master

Every cloud provider will give you a set of steps to follow in order to connect to your VM. They involve an ssh command and the public IP address of the actual server.

Go ahead and connect to the server you wish to act as a Master. To get started with installing Kubernetes on a Debian based Linux machine, follow the steps below.

Here’s a sample ssh connection to a cloud server.

    ssh user@ip -p port

***Note**: A default ssh connection for a cloud provider like DigitalOcean or AWS would be something like this: ssh root@129.212.34.91 -p 22*

### 1. Install Docker

As with our Minikube installation, the server needs to have a containerization software installed. Hence, installing Docker.

    $ apt-get update
    $ apt-get install -y docker.io

### 2. Install Kubernetes

Once Docker is installed, go ahead and run the commands to install Kubernetes.

    $ apt-get update && apt-get install -y apt-transport-https
    $ curl -s [https://packages.cloud.google.com/apt/doc/apt-key.gpg](https://packages.cloud.google.com/apt/doc/apt-key.gpg) | apt-key add -

    $ cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
    deb http://apt.kubernetes.io/ kubernetes-xenial main
    EOF

    $ apt-get update
    $ apt-get install -y kubelet kubeadm kubectl

    $ export MASTER_IP=<master_ip>

You can see that apart from **kubectl** we’re also installing **kubelet** and **kubeadm**. You learned about kubelet previously in this walkthrough, however, not about kubeadm.

**Kubeadm **is tasked with bootstrapping the cluster, it creates all the necessary add-ons for the cluster to function properly, and it supports tokens for adding new Nodes to the cluster.

The last line in this set of commands will export the IP of the Master as an environment variable, so you can easily access it later on. Just replace <master_ip> with the public IP address of your VM.

### 3. Initialize Kubeadm

Initializing kubeadm is as easy as running one command.

    $ kubeadm init --apiserver-advertise-address $MASTER_IP

**Important**: You’ll see a bunch of output get returned to you in the terminal. There’s one thing you need to keep an eye out for. The command for adding Nodes to the cluster. It will be shown to you here. **Make sure to save it somewhere safe**.

**The command for adding Nodes to the Kubernetes cluster will look something like this.**

    kubeadm join --token <token> 138.197.186.42:6443 --discovery-token-ca-cert-hash sha256:<hashed_token>

### 4. Configure Kubectl

To manage the Kubernetes cluster, the client configuration and certificates are required. This configuration is created when **kubeadm** initializes the cluster. This command copies the configuration to the user’s home directory and sets the environment variable for use with the CLI.

    $ cp /etc/kubernetes/admin.conf $HOME/
    $ chown $(id -u):$(id -g) $HOME/admin.conf
    $ export KUBECONFIG=$HOME/admin.conf

It may happen that the environment variable is reset if you create a new ssh session with the server. Then you’ll see some funky stuff start happening. Just run the last command again and you’ll have the Kubernetes configuration back the way it should. Or, you can just run the kubectl command with the --kubeconfig flag. From the $HOME directory, where you placed the admin.conf file, run kubectl like this.

    kubectl --kubeconfig /path/to/admin.conf <command> <second_command>

    // example
    kubectl --kubeconfig ./admin.conf get nodes

### 5. Install the pod network

A crucial step in enabling communication between the Nodes is installing a pod network. I promise these are the last couple of commands we need to run on the Master.

    $ sysctl net.bridge.bridge-nf-call-iptables=1
    $ export kubever=$(kubectl version | base64 | tr -d '\n')
    $ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$kubever"

Awesome! Give yourself a pat on the back. Now to check that your cluster is operational, go ahead and run a command to check the nodes.

    $ kubectl get nodes

The output from this command will look something like this.

    NAME       STATUS    ROLES     AGE       VERSION
    master     Ready     master    Xd       v1.9.0

That’s it. Done with the Master, let’s connect to a Worker Node in the private network.

### Installing Kubernetes on the Node

Setting up a Node is even simpler than a Master. Many of the commands are exactly the same, first of all installing Docker and Kubernetes. Go ahead and ssh into the Worker Node and run these commands as the superuser.

### 1. Install Docker

Just as with the Master, we run the Docker install command.

    $ apt-get update
    $ apt-get install -y docker.io

### 2. Install Kubernetes

See the pattern? Let’s install Kubernetes as well.

    $ apt-get update && apt-get install -y apt-transport-https
    $ curl -s [https://packages.cloud.google.com/apt/doc/apt-key.gpg](https://packages.cloud.google.com/apt/doc/apt-key.gpg) | apt-key add -

    $ cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
    deb http://apt.kubernetes.io/ kubernetes-xenial main
    EOF

    $ apt-get update
    $ apt-get install -y kubelet kubeadm kubectl

    $ export WORKER_IP=<worker_ip>

Make sure to change <worker_ip> to the actual public IP address of your VM.

### 3. Join the cluster

Finally, we’re reached the fun part. Remember the **cluster join** command that got sent back to the output when we initialized **kubeadm**? We’ll need that now. Go ahead and run the whole command that got returned to you from the kubeadm init --apiserver-advertise-address $MASTER_IP command.

    $ kubeadm join --token <token> <master_ip>:6443 --discovery-token-ca-cert-hash sha256:<hashed_token>

***Note**: The default secure port for the Kubernetes API server is 6443.*

After you run the command above, you’ll see some output telling you the Node has successfully connected to the cluster.

### 4. Check the cluster status on the Master

Now, once you ssh into the Master once again, go ahead and check the status of the Nodes in the cluster.

    $ kubectl get nodes

You should see them with the Ready status. If you see NotReady instead, give it a few minutes.

    NAME       STATUS    ROLES     AGE       VERSION
    master     Ready     master    xmin      v1.9.0
    node       Ready     node      xmin      v1.9.0

***Note**: Remember, you can only interact with the Kubernetes cluster from the Master.*

### 5. Interact with the Master from your local development machine

It would be a bit inconvenient if you need to ssh into the Master every time you want to interact with the cluster. Kubernetes has an awesome feature though. By copying the config files created during the Kubernetes initialization on your Master to your local machine, where you have **kubectl** installed, you’ll be able to access your cluster remotely. How awesome is that!?

To copy the file from the Master to your development machine run the command below.

    $ scp <user>@<master ip>:/etc/kubernetes/admin.conf $HOME/
    $ kubectl --kubeconfig ./admin.conf get nodes

This will copy the config file to the home directory. From there you’ll run the **kubectl** commands with the --kubeconfig option.

Or, if you’re a tidy developer, unlike me, here’s a guide on how to handle multiple configurations.
[**Configure Access to Multiple Clusters**
*Edit This Page This page shows how to configure access to multiple clusters by using configuration files. After your…*kubernetes.io](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

### Using Google Cloud Kubernetes Engine

What about those of us who don’t like managing servers? Well, Google’s got us covered. With their Kubernetes Engine, you just have to go through a few simple steps, and you’re ready to interact with a live Kubernetes cluster from your local machine!
[**Quickstart | Kubernetes Engine Documentation | Google Cloud Platform**
*Google Cloud Shell is a shell environment for managing resources hosted on Google Cloud Platform (GCP). Cloud Shell…*cloud.google.com](https://cloud.google.com/kubernetes-engine/docs/quickstart)

Incredibly convenient. Take a look above if you don’t believe me.

***Note**: The various Kubernetes objects running just for the sake of keeping it operational all have their own dedicated ports. In case you enable a firewall, make sure to allow them access, otherwise the cluster won’t work right.*

![From [this awesome StackOverflow answer](https://stackoverflow.com/questions/39293441/needed-ports-for-kubernetes-cluster).](https://cdn-images-1.medium.com/max/2000/1*HMiaRF3fHJqS1jN31A8PDw.png)*From [this awesome StackOverflow answer](https://stackoverflow.com/questions/39293441/needed-ports-for-kubernetes-cluster).*

## Deploying to Kubernetes

Whoa… That was a fair share of configuring. Finally time to create a real Kubernetes deployment! Hang in there, this will get bumpy.

### Creating an Nginx deployment

Let’s keep things simple, but still use all the bells and whistles you usually would in a production environment. We’ll create a simple Nginx deployment, expose it through a service, and finally scale it out.

### 1. Create the deployment

When creating a deployment, Kubernetes will automatically create a pods and replica sets for you. Check it out.

Once connected to your Kubernetes instance running either locally or in the cloud, go ahead and run the command for creating a deployment.

    $ kubectl run nginx --image=nginx

The run command will create a deployment called **nginx **from the official nginx Docker image.

To make sure the deployment is running go ahead and use the get command to check its state.

    $ kubectl get deployment nginx

The output you’ll see will look something like this.

    NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx     1         1         1            1           1m

You have a single pod running in the deployment. But, it’s not visible outside of the private network. To expose it, you need to create a service. Guess what we’re doing next?

### 2. Exposing the deployment

What if I told you the command for exposing a deployment is actually called expose ? It really is! That’s just incredibly convenient. I enjoy when things just make sense. Anyhow, back to the service.

    $ kubectl expose deployment nginx --external-ip=$MASTER_IP --port=80 --target-port=80

The expose command will take a deployment parameter, where you specify the deployment you want to hook a service to. The --external-ip option will take the public IP address of your Master, or if you’re following along this tutorial running an instance of minikube, add the minikube-vm IP address. The first --port option specifies the port which will be exposed, while the
--target-port specifies which port the containers in the pods are running on.

To make sure your service is running, run this command.

    $ kubectl get service

The output will look like this, where the <master_ip> is the public IP address of your Master.

    NAME        TYPE       CLUSTER-IP      EXTERNAL-IP    PORT(S)    AGE
    kubernetes  ClusterIP  10.96.0.1       <none>         443/TCP    23m
    nginx       ClusterIP  10.111.213.194  <master_ip>    80/TCP     2m

### 3. Scaling out

Lovely! You have a running deployment and a service exposing a port of your choosing. What now? We’ll, you still need a way of scaling out. Thankfully, Kubernetes has a nifty little command for that.

    $ kubectl scale deployment nginx --replicas=6

Check the state of the deployment once again, and you’ll see the deployment has 6 pods.

    $ kubectl get deployment nginx

    NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx     6         6         6            6           1h

Feel free to check the individual pods with the get pods command.

    $ kubectl get pods

    NAME                   READY     STATUS    RESTARTS   AGE
    nginx-8586cf59-4qdgc   1/1       Running   0          1h
    nginx-8586cf59-fdq8r   1/1       Running   0          1h
    nginx-8586cf59-j5mdh   1/1       Running   0          1h
    nginx-8586cf59-ppqcd   1/1       Running   0          1h
    nginx-8586cf59-sn4hd   1/1       Running   0          1h
    nginx-8586cf59-w2z55   1/1       Running   0          1h

Looking great. The Nginx deployment has been scaled out successfully. But, this is just a simple Nginx container. What about a serious application with several containers with the need for persistent storage? That’s a proper challenge. Let’s get crackin’.

## Creating a Node.js and MongoDB deployment

Great job sticking with me to the end. It’s not until now we’ll bring out the big guns. Creating large scale Kubernetes clusters through the command line is not a sound option, neither for overview, nor mental health. That’s why I’ll focus on using configuration files for deploying Kubernetes objects in this last section of the walkthrough.

***Note**: For huge systems I’d encourage you to use [**Helm](https://helm.sh/)**. But that’s a bit out of the scope of this guide. Let’s leave that for another tutorial.*

Another crucial thing is to enable some sort of persistence to our cluster. As we know, Pods are stateless. If we delete one, the data is lost forever. To solve this, Kubernetes has enabled the concept of **volumes**. Not to dig too deep, what’s important to note is that a volume is a resource in the storage of a host machine which is at the disposal of all the objects in the cluster. The persistent volume is provisioned by an administrator.

We’ll touch upon using Kubernetes [Persistent Volumes and Persistent Volume Claims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/). Unlike the **persistent volumes** themselves, a **persistent volume claim** is requested by a user to use resources from a persistent volume.

Here’s an example. You have created a persistent volume with the size of 10 GB. But, you want the deployments in the cluster to only **claim** 3 GB each.

Hope that makes sense now. If it still doesn’t, let’s jump right into some examples for you to see.

### 1. Create a persistent volume

Like I mentioned above, we’ll use yml configuration files for creating the objects in this walkthrough. They are rather understandable to begin with, so don’t be scared.

Let’s start by creating a work directory where we’ll keep all the yml files. Create a directory called **cluster** and open it up in terminal. Let’s start slow and take a see what a persistent volume declaration looks like.

***Note**: I’ll add all the yml files as gists to keep the formatting nice and clean. Otherwise they won’t work properly.*

<iframe src="https://medium.com/media/86bfce3ff9c7fa156317611c78538b51" frameborder=0></iframe>

We’ll give it a capacity of 10 GB and a host path of /mnt/data/mongo with the Filesystem volume mode. This will create a **persistent volume** on the host machine.

Make sure to create a file named **mongo-persistent-volume.yml** in your **cluster** folder. While connected to your Kubernetes cluster, go ahead and run the create command with **kubectl**.

    $ kubectl create -f ./mongo-persistent-volume.yml

There we go, a new persistent volume is up and running. To list your persistent volumes run the command below.

    $ kubectl get persistentvolume

Now we need to add a **persistent volume claim**, for our deployments to use. Here’s what the yml file looks like.

<iframe src="https://medium.com/media/cf5b0b9ff66d5d00ef824ec25da2405b" frameborder=0></iframe>

Again run the create command to create the object.

    $ kubectl create -f ./mongo-persistent-volume-claim.yml

There we go! We have persistence. Ready to add some compute?

### 2. Create a MongoDB deployment

This creation process will be just as simple as what we did above. We’ll create two more files and run two more commands. That’s it!

<iframe src="https://medium.com/media/3d05923aa4d50e4f04e8063406e8e4cc" frameborder=0></iframe>

Add the **mongo-deployment.yml** file above to you **cluster** folder, and run yet another create command.

    $ kubectl create -f ./mongo-deployment.yml

Taking a closer look at the yml file above. We’re using the mongo image and binding the default port. Apart from that we’re also adding a **volume mount** pointing to the **persistent volume claim** we created in the previous step.

You can also see we added a tier: backend and app: mongo. This will give the deployment an alias in the cluster, through which it will be discoverable once we add services.

Speaking of services, let’s add one. Name it **mongo-service.yml**.

<iframe src="https://medium.com/media/5413d79658f3ef75cdaf53207861c720" frameborder=0></iframe>

Check the labels! This service will find all deployments with the app set to mongo and tier set to backend and expose them to the cluster network. The ports setting will make sure to let traffic through the default MongoDB port 27017. Finally, run the command to create the object.

    $ kubectl create -f ./mongo-service.yml

### 3. Create a Node.js deployment

The Node.js deployment will not differ significantly from the previous we created above. It will also have one deployment file and one service file.

<iframe src="https://medium.com/media/03a13b3bbd1e018995c828ee11d12ca6" frameborder=0></iframe>

Add the **node-deployment.yml** file and run:

    $ kubectl create -f ./node-deployment.yml

You can see we’re adding this deployment to the **backend tier** as well. However, we’ll add 9 replicas and set a rolling update strategy to ensure maximum up-time even during updates.

The image from which we will build the containers housed in the deployment pods is a [tiny boilerplate API](https://github.com/adnanrahic/boilerplate-api) I wrote a while back. It has basic authentication, which is perfect to test out our new cluster and persistent volume. I’ve also configured the database connection string to point to the MongoDB app label we specified in the **mongo-deployment.yml** file.

What’s left is to add a service and that’s it!

<iframe src="https://medium.com/media/ceda09586538cafde833d7c989fa859b" frameborder=0></iframe>

Check this out. The **node-service.yml** will be a **load balancer**! It’ll route all the traffic through port 80, in a round-robin fashion, to all the replicated pods in the deployment. How cool is that!?

***Note**: Replace the externalIPs: field with you Master server’s public IP address. (If you’re testing with Minikube, replace it with the external IP given to your Minikube)*

Would you believe me that’s all you need? I’m not even kidding. But, let’s be responsible developers and test it all, to make sure it works.

### Testing the Node.js & MongoDB cluster

First of all, check whether the resources are running like they should.

    $ kubectl get all

This command will return back to you a list of all deployments, replicasets, pods and services.

Verify it’s all running. Once you do, fire up a REST API testing client, such as [Insomnia](https://insomnia.rest/) or [Postman](https://www.getpostman.com/), and test a few endpoints.

![](https://cdn-images-1.medium.com/max/3838/1*LDVfBr3tIkibc_3JXcjFfA.png)

Hitting the IP address I set as external on the route /api, will return a simple string telling me the Node.js API is working like it should.

Let’s try registering a new user. Hit the /api/auth/register endpoint, and send along some JSON data in the format of an email, name, and password field.

![](https://cdn-images-1.medium.com/max/3838/1*ho7ABk2rATNG5MUZn5IK4A.png)

That worked! We got a token returned back. Let’s **grab the token** and add it to the x-access-token header in a GET request to the /api/auth/me endpoint. This should return back the info of John Doe we just registered and authenticated.

![](https://cdn-images-1.medium.com/max/3838/1*LiLVPGCdC0j32CvVjGzUbg.png)

Lovely! That worked just as we expected. Now, just to make sure, hit the /api/users endpoint to verify the user has really been added to the persistent volume. Feel free to restart deployments to assure yourself. Because we added the Retain setting in the persistentVolumeReclaimPolicy any deployment will re-use the existing data once it gets re-started. How convenient is that!

With all that over with, we’ve come to the end of an incredible journey. Give yourself a pat on the back. You now know the basics of Kuberentes.

## Wrapping up

Whoa, that was a lot to take in… 
Kubernetes eases the strain on using Docker containers in large scale production environments. I for one would never want to use anything but Kubernetes, because of the huge burden in takes off of my shoulders.
> # Kubernetes, is a system for automating deployment, scaling and management of containerized applications.

With Kubernetes you won’t have to put up with deployment headaches and angry clients shouting at you because their website is down. That’s why you need it! To have peace of mind.
> # If an instance fails, Kubernetes will re-create the resources of the failed instance on one that’s running.

You don’t need to worry about every instance you own, neither do you need to worry about whether the containers are running. I’ll let that speak for itself. Until next time. Happy Klustering!

If you want to take a look at all the code, and terminal commands, we wrote above, [here’s the repository](https://github.com/sourcerer-io/sourcerer-blog/tree/master/a-kubernetes-quick-start-for-people-who-know-just-enough-about-Docker-to-get-by). Or if you want to read my latest articles, head over here.
[**Latest stories written by Adnan Rahić - Medium**
*Read the latest stories written by Adnan Rahić on Medium. Software engineer @bookvar_co. Coding educator @ACADEMY387…*medium.com](https://medium.com/@adnanrahic/latest)

*Hope you guys and girls enjoyed reading this as much as I enjoyed writing it.* 
*Do you think this tutorial will be of help to someone? Do not hesitate to share. If you liked it, smash the **clap** below so other people will see this here on Medium. Don’t forget to show us some love by **following the Sourcerer blog!***
