Unknown markup type 10 { type: [33m10[39m, start: [33m325[39m, end: [33m332[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m93[39m, end: [33m101[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m154[39m, end: [33m158[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m166[39m, end: [33m170[39m }

# Kubernetes For Beginners

Kubernetes, for a more general audience

![Photo by [Joseph Barrientos](https://unsplash.com/@jbcreate_?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/11150/0*ICfb_U811Cv88Jlo)*Photo by [Joseph Barrientos](https://unsplash.com/@jbcreate_?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

[Kubernetes](https://kubernetes.io/) has been around for a while. It’s the leading-edge platform that’s changed the way we look at information technology today. The project was started by a bunch of Google developers as a way to orchestrate containers, which they open-sourced to the cloud-native computing foundation. Today it is one of the most popular systems and the de facto standard for running your containers.

There are various reasons for that. We’ll look into what Kubernetes is in the first place, why it is so popular, and what problem it solves, in plain and simple words. To follow this article, you do not need any prior experience with Kubernetes or containers. I will declutter these concepts one by one.

## What Is Kubernetes?

If we look at the [Wikipedia definition](https://en.wikipedia.org/wiki/Kubernetes), we see that “Kubernetes is an open-source container-orchestration system for automating application deployment, scaling, and management.”

There are two main words here: *container* and *orchestration*. We need to understand what each one is to understand Kubernetes.

## What Are Containers?

According to [Kubernetes](https://kubernetes.io/), “**Containers** are a technology for packaging the (compiled) code for an application along with the dependencies it needs at run time. Each **container** that you run is repeatable; the standardisation from having dependencies included means that you get the same behaviour wherever you run it.”

The definition may sound a bit too much for a beginner. Let me explain.

If we need to spin up a stack of applications in a server, such as a web application, database, messaging layer, etc., this will result in the following scenario.

![Application stack on a server](https://cdn-images-1.medium.com/max/2000/1*pZX1m5jE1r9Ue5jNmDyLHQ.png)*Application stack on a server*

There is a hardware infrastructure on which an operating system (OS) runs, and libraries and application dependencies are installed over the OS. Different applications then share the same libraries and dependencies to run.

## The Matrix of Hell

If you look into the design described in the previous section, there are bound to be multiple problems. If you’ve rightly guessed, a web server might need a different version of a library as compared to the database server, for example, and one version of dependency can be compatible with one application but incompatible with another. If we need to upgrade one of the dependencies, we need to ensure that we do not impact another application that might not support it. This scenario is known as the *Matrix of Hell* and is a nightmare for developers and admins alike.

## It Works on My Machine

Anyone who has worked in technology has encountered this phrase at least once in their career. “It works on my machine” is a typical conversation between a developer and a tester, where a developer says that the application works perfectly fine on their machine. Still, it does not work in the test environment, and things that might have worked perfectly fine in the test environment may not work the same in the production environment. Reason? The Matrix of Hell.

## Solutions

Before containers, organisations have been solving this issue by using virtual machines. A virtual machine is an emulation of a computer system. Virtual machines are based on computer architectures and provide the functionality of a physical computer using software called a *hypervisor*. Some of the popular hypervisors in the market are VMWare and Oracle Virtual Box. A typical VM-based stack looks like this:

![Application Stack on VMs](https://cdn-images-1.medium.com/max/2444/1*Z2fC5Tb6q3BpjoX3FyYc-g.png)*Application Stack on VMs*

So far, so good. We have resolved the dependency problem, and now we are out of the Matrix of Hell. This architecture was groundbreaking, lasted for over two decades, and is still in use today. However, this introduces another issue. Instead of running a single OS within a machine, we now have multiple guest OSs running within a physical device.

## Problems of the Virtual Machine Era

We were trying to solve the runtime, library, and dependency problems, but we introduced a heavyweight guest OS layer in between, which has its disadvantages. A virtual machine is heavy, slower to start, and presents an extra dependency to maintain a guest OS along with a host OS. Admins now need to take care of multiple servers rather than numerous dependencies.

Also, we need to allocate a minimum amount of resources to the guest OS, and organisations over-provision resources to VMs to meet the peak utilisation of the VM rather than the regular use. Resource sharing within a virtual machine is not optimal even if it is architected correctly. VMs waste a lot of resources because a significant percentage of allocated resources remain unutilised. Having a dedicated VM for an application leads to portability issues and operations teams soon find themselves trying to tend to the servers as pets. Applications running on servers are indispensable and if they fail for some reason, building them again from scratch is an issue.

## Introducing Containers

Containers balance the problem out by treating servers as servers. We no longer have a separate VM for the webserver, database, and messaging. Instead, we have different containers for them. Confused? See the diagram below.

![Application stack on containers](https://cdn-images-1.medium.com/max/2444/1*WzVlSOk77g2vlo0cB9jH6w.png)*Application stack on containers*

We have now got rid of the guest OS dependency, and containers now run as separate processes within the same OS. Containers make use of container runtimes. Some of the popular container runtimes are [Docker](https://www.docker.com/), [Rocket](https://www.rocker-project.org/), and [containerd](https://containerd.io/). The most popular of them, and more or less the de facto standard in container runtime technology, is Docker.

A container runtime provides an abstraction layer that allows applications to be self-contained and all applications and dependencies to be independent of each other.

Containers solve a lot of problems.

**Containers are portable **— Since containers have the runtime and application dependencies packaged together, they are portable. A container does not care where it is running and behaves the same in all environments.

**Containers are more efficient **— Since containers do not contain a guest OS, they boot up extremely fast. While it takes minutes to boot a VM, it takes seconds to spin up a new container.

**Containers are scalable **— Not that VMs are not scalable, but since containers boot fast and have a low footprint, spinning up new containers is way easier than spinning up new VMs. You can have multiple containers scaling up and down independently within an OS and sharing the resources, which makes it less wasteful than VMs.

**Containers are lightweight** — Since containers do not contain a heavy guest OS, containers have a very light footprint. You do not need to allocate set resources to a container, and it can use the underlying OS resources, similar to an application. They require less computing resources to run in comparison to VMs.

## Managing Containers

The idea of having multiple containers running within a server sounds tempting, but they come with their own set of problems. How you scale containers? How do you ensure that containers run and heal when they are unhealthy? What would happen if you suddenly see a spike and want to scale up your containers automatically? What if you see a downward surge in traffic and want to scale down your containers to the optimal level? Containers are suitable for a microservices architecture, and if you are using it, how would you make sure that the containers can talk to each other? What if you reach a resource limit and want to schedule your containers in another VM?

The answer to all these questions is a container orchestration platform like Kubernetes.

## Introducing Container Orchestration Using Kubernetes

The idea of using Kubernetes is simple. You have a cluster of servers that are managed by Kubernetes, and Kubernetes is responsible for orchestrating your containers within the servers. You treat servers as servers, and you run applications within self-contained units called containers.

Since containers can run the same in any server, it does not matter on what server your container is running, as long as the client can reach it. If you need to scale your cluster, you can add or remove nodes to the cluster without worrying about the application architecture, zoning, roles, etc. You handle all of these at the Kubernetes level.

Kubernetes uses a simple concept to manage containers. There are master nodes (control plane) which control and orchestrate the container workloads, and the worker nodes where the containers run. Kubernetes run containers in pods, which form the basic building block in Kubernetes.

Kubernetes essentially provides the following:

1. **Communicates with the underlying container runtime **— Kubernetes is not a container platform but a container orchestration platform. Its kubelet component, which runs as a service on every node, is responsible for talking to the underlying container platform to manage the containers. For example, when you create a pod using kubectl and your underlying container platform is Docker, kubelet issues a Docker run command to the Docker runtime in the chosen worker node.

1. **Stores state of the expected configuration **— When you apply a Kubernetes configuration using kubectl create/apply commands, Kubernetes stores that in its etcd datastore as an expected configuration.

1. **Maintains state based on the expected configuration** — Kubernetes keeps on trying to maintain the expected state of the cluster by looking at the configuration in the etcd datastore.

1. **Provides an abstract software-based network orchestration layer** — The pod network provided by Kubernetes ensures that containers can communicate with each other on some sort of an overlay or bridge network. This network is managed within the container runtime (using Docker bridge networks, for example) or through an internal or external network. When a pod talks to another pod, Kubernetes modifies routing tables to ensure that the connectivity is in place.

1. **Provides inbuilt service discovery **— Kubernetes provides service discovery of containers out of the box. You do not need an external application to manage it. Kubernetes exposes your pod on a DNS that maps the service name to any available Pod IP, therefore providing service discovery and load balancing between multiple pod replicas. It is because of this service discovery that pods are ephemeral. Services can also expose your pods to internal and external clients by creating listeners within your nodes and also by requesting cloud providers to provision a load balancer to point to your pods.

1. **Health checks the configuration **— Kubernetes ensures that the container workloads running within the cluster are of expected health, and if not, it destroys and recreates the containers.

1. **Requests cloud provider for objects **— If you are running Kubernetes within a cloud provider like GCP or Azure, it can use the Cloud APIs to dynamically provision resources like load balancers and storage. This way, you have a single control plane for managing everything you would need to run your applications within containers.

Following is a high-level architecture of a Kubernetes cluster:

![](https://cdn-images-1.medium.com/max/2864/1*cT56FWaY0s0V4Yh04OezEg.png)

You no longer need to worry about the state of your application but rely on Kubernetes to ensure that your application continues to run and is healthy. Developers only need to focus on the application code, and they need not worry about the underlying infrastructure, as containers will run the same in all environments. Kubernetes has made the life of administrators a lot easier: it provides a significant number of features, which makes managing Kubernetes clusters more effective and efficient.

## Further Reading

Thank you for reading through. I hope you enjoyed the article. If you are interested to learn further, check out “[Demystifying Kubernetes Objects](https://medium.com/better-programming/demystifying-kubernetes-objects-understanding-the-what-why-and-how-18b42c9ca9c2).”
