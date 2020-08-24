Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m15[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m8[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m245[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m245[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m80[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m119[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m24[39m, end: [33m33[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m47[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m72[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m183[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m82[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m62[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m152[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m32[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m102[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m78[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m34[39m, end: [33m53[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m52[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m88[39m, end: [33m105[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m35[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m17[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m111[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m54[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m48[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m89[39m, end: [33m101[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m8[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m124[39m, end: [33m144[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m37[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m126[39m, end: [33m138[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m72[39m }

# How to Create a Kubernetes Cluster Locally — Simple Tutorial

How to create a Kubernetes cluster locally and deploy simple front-end apps that communicate with Kubernetes

![](https://cdn-images-1.medium.com/max/2988/1*GO1z6qPosfNgy1QGV7zLEQ.jpeg)

*As a software engineer at Capital One, I get to explore cutting edge technologies every day in my work. I have worked with Docker and Docker Swarm and I always wanted to learn Kubernetes. However, I kept postponing it. Finally, I was able to dive into it and thought, “Why not create an application with Kubernetes and write about it while my mind is fresh!” That way others — and not just myself — can benefit from what I learned.*

## Introduction

Today we are going to create a Kubernetes cluster and deploy a simple React JS app which generates a random number by calling an Express JS app. We are going to orchestrate the whole process by using Kubernetes. We will first dockerize our front end/back end apps. Then using Kubernetes we will deploy the pods (React front end app/Express back end app) and access them via Kubernetes services.

![](https://cdn-images-1.medium.com/max/2000/0*rsMFDAVek8hXG_i5)

## What is a Kubernetes Cluster?

Like any cluster you provide a set of nodes to Kubernetes. You tell kubernetes how to deploy containers in the cluster. How much memory or processing units each container gets and how they interact with each other.

## Kubernetes Cluster Diagram

The below diagram depicts what we are going to achieve today, where we will create a Kubernetes cluster and services/deployments for our front and back ends.

![](https://cdn-images-1.medium.com/max/2500/1*Nqas3nUDLcWChCOZ6taFXQ.png)

## Prerequisites

1. [Docker for Desktop](https://www.docker.com/products/docker-desktop) (latest version)

1. [Kubernetes](https://kubernetes.io/docs/setup/learning-environment/minikube/) or [Docker Kubernetes](https://www.docker.com/products/kubernetes)

1. Node & NPM (Only if you want to run the applications stand alone)

1. YAML

***NOTE:** This tutorial requires basic/working knowledge on Docker, Node, & NPM.*

Let’s dive in without any further ado.

## Getting Started With Kubernetes

### Installing Kubernetes

*If you have **Docker Desktop**, go to **preferences**, go to the **Kubernetes tab,** and click **Enable Kubernetes**. It may take a while to spin up Kubernetes on to your machine, so go make a coffee while it does its magic. ☕*

![](https://cdn-images-1.medium.com/max/2780/1*hKknutHbe5NeFfgnVWHrpg.png)

To verify if Kubernetes is running, type the below two commands:

    kubectl version

    Outputs:

    Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.0", GitCommit:"e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529", GitTreeState:"clean", BuildDate:"2019-06-19T16:40:16Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"darwin/amd64"}

    Server Version: version.Info{Major:"1", Minor:"14", GitVersion:"v1.14.8", GitCommit:"211047e9a1922595eaa3a1127ed365e9299a6c23", GitTreeState:"clean", BuildDate:"2019-10-15T12:02:12Z", GoVersion:"go1.12.10", Compiler:"gc", Platform:"linux/amd64"}

    kubectl cluster-info

    Outputs:
    Kubernetes master is running at https://kubernetes.docker.internal:6443

    KubeDNS is running at [https://kubernetes.docker.internal:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy](https://kubernetes.docker.internal:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy)

## Bundling Application As Docker Images

### The full code for this can be found on [GitHub](https://github.com/chiku11/react-k8)

Please download the project. The project has two sub folders in it.

* Client -> React based app

* Server -> Express based app

Please follow the below steps to setup the project and start the Kubernetes cluster:

### Client

* *cd client*

* *npm install*

* *npm run build*

* ***docker build -t frontend:1.0 .***

* ***kubectl apply -f frontend.deploy.yml***

* ***kubectl apply -f frontend.service.yml***

### Backend

* *cd server*

* *npm install*

* ***docker build -t backend:1.0 .***

* ***kubectl apply -f backend.deploy.yml***

* ***kubectl apply -f backend.service.yml***

Go to the browser, type localhost and hit enter, you should see the application loaded.

## Bringing It Down Further

### Kubernetes Pods

Pods are the smallest deployable units of computing that can be created and managed in Kubernetes.

    kubectl run nginx-frontend --image=frontend:1.0

The above creates a pod which hosts the front end container. You can’t access it yet since a host port isn’t exposed for the container. We will uncover those later.

A pod can host multiple containers as well. Instead of creating via command line let’s create via a YAML file.

    # To create a pod with multiple containers
     kubectl apply -f app.pod.yml

    # To see container status
     kubectl get pod/mymulticontainerapp
     Outputs:
     NAME                  READY   STATUS    RESTARTS   AGE
     mymulticontainerapp   2/2     Running   0          9m

    # To see more details about the container
    kubectl describe pod/mymulticontainerapp

To access the container from the host machine you have to do a port forwarding, i.e attaching host port to container port. You can do that via the below command:

    kubectl port-forward pod/mymulticontainerapp 9999:80 3000:3000

    Outputs:
    Forwarding from 127.0.0.1:9999 -> 80
    Forwarding from [::1]:9999 -> 80
    Forwarding from 127.0.0.1:3000 -> 3000
    Forwarding from [::1]:3000 -> 3000

The above command exposes the host port 9999 to container port 80 and host port 3000 to container port 3000 which is what our front end/back end containers are listening on. Any request to host 9999 will be forwarded to container on port 80. Any request to the host 3000 will be forwarded to container on port 3000.

Go to the browser and open 127.0.0.1:9999 or localhost:9999, it should load the front end app.

To delete a pod you can do with below command:

    k delete pod/mymulticontainerapp

To inspect a pod you can use the below command:

    Front End:
    kubectl exec mymulticontainerapp -c myfrontendapp -it /bin/sh
    ls /usr/share/nginx/html
    exit

    Back End:
    kubectl exec mymulticontainerapp -c mybackendapp -it /bin/sh
    ls
    exit

The above commands open a shell inside a container to interact.

Before we jump on to the next topic **— **deployments — it’s crucial that you understand metadata in a YAML file. Metadata is data about the containers. You can add labels and key-values to the metadata. You can also use the labels or key-values as selectors to identify a pod which will be used by deployments/services later.

Let’s get all the pods labeled as mymulticontainerapp:

    kubectl get pods --selector=name=mymulticontainerapp

## Deployments

You describe a desired state in a deployment, and the Deployment [Controller](https://kubernetes.io/docs/concepts/architecture/controller/) changes the actual state to the desired state at a controlled rate. You can define deployments to create new ReplicaSets, or to remove existing deployments and adopt all their resources with new deployments.

It should look pretty similar to the previous YAML you saw. The salient point to note here is *kind *— it’s a deployment rather than pod.

**Selector:** It is used to identify existing pods by labels, such as their metadata. If there is a pod already running with such a label, it will be part of this deployment.

**Replicas: **It specifies how many pods within this container you wish to create. If you say two, it will have two pods running the back end.

Let’s say replicas is set to two and there are four pods already running with the label app: node-backend. It will terminate two of those pods to meet the requirement of two replicas. It will also scale up the pods by two if replicas is set to six.

**Spec:* ***It specifies the details about the container such as image name, container port, CPU/memory limits for the container inside the pod, etc.

**Template: **It specifies the tags to be used for the newly created pods via the spec.

*******

To create a deployment run the below command:

    kubectl apply -f backend.deploy.yml

    Output:
    deployment.apps/node-backend created

To see all the deployments:

    k get deployments

    Output:
    NAME           READY   UP-TO-DATE   AVAILABLE   AGE
    node-backend   2/2     2            2           46s

To access the containers created during deployment you can use port-forward.

    kubectl port-forward deployment/node-backend 3000:3000

    You can access by going to localhost:3000/random

## Kubernetes Services

An abstract way to expose an application running on a set of [pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) is as a network service.

Things to notice here — the kind is Service and the type is LoadBalancer. This creates a LoadBalancer on host port 3000 and proxies the request to container on 3000.

What is LoadBalancer balancing here? If you look closely we specified a selector. The selector searches for pods with label app: node-background and any request sent to the host on 3000 will be load balances among those pods. Here we have two replicas and the request will be load balanced to these two pods.

To create a service run the below command:

    kubectl apply -f backend.service.yml.

To see all the services run the below command:

    k get service

![](https://cdn-images-1.medium.com/max/2536/0*-A5_2m212NCTMtiY)

This tells you the service name and what type of service it is. Here if you see there is a service called backend and of type LoadBalancer, which we just created using the service yml.

You can do the same steps for frontend as well:

    kubectl apply -f backend.deploy.yml
    kubectl apply -f backend.service.yml

## Wasn’t That Easy?

We have looked at how to create a Kubernetes cluster and deploy a simple front end app that communicates with a backend app in Kubernetes. Hope this helps your understanding of the basics of Kubernetes. Kubernetes is a vast ocean and we just touched a tiny drop of it. But I hope this helps pave the path for your future Kubernetes learning!

**Useful Links:**

[https://kubernetes.io/docs/concepts/](https://kubernetes.io/docs/concepts/)

[https://docs.docker.com/](https://docs.docker.com/)

*DISCLOSURE STATEMENT: © 2020 Capital One. Opinions are those of the individual author. Unless noted otherwise in this post, Capital One is not affiliated with, nor endorsed by, any of the companies mentioned. All trademarks and other intellectual property used or displayed are property of their respective owners.*
