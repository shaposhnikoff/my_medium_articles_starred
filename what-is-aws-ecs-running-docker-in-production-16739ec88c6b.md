
# What is AWS ECS? Running Docker in Production



Running Docker in production has quickly become the norm. Cloud hosting providers like AWS, Azure and Google Cloud realized that this is what Organizations are in need for. Services like EKS(Elastic Kubernetes Service) and ECS(Elastic Container Service) from AWS offer a completely managed environment for your Docker containers to run on. Through this article, we’ll take a closer look to one of them, Amazon ECS, which is Amazon Elastic Container Service. We are going to describe what AWS ECS is, its functions, and its importance in the current market.
> *“AWS ECS is a fully-managed, scalable and production-ready platform for running containers.”*

If you don’t know what any of this means, then the rest of the article is going to help you with that knowledge. “Fully-managed” implies you don’t have to pay any third-party software vendor to run your containerised application. “Scalable” means you don’t have to worry, ahead of time, about resource utilization. AWS will make resources(ECS Tasks), like CPU, memory and storage, available to you, on demand.

But why should you care about it? The reason behind it is two-fold.

First of all, is the flexibility and scalability of a microservice-based architecture which most applications are now adopting. As it turns out, Docker containers are really good for deploying microservices. So, it stands to reason that your application may get shipped as a Docker image. If that’s not the case, you may not want ECS, since this service is exclusive for those who intend to run Docker containers.

The second reason is cost-effectiveness. This is true especially if you use Amazon Fargate. It’s a pay-as-you-go policy that bills you by the minute and can result in significant price reductions. You can also launch your own ECS task on top of EC2 instances.

Moreover there are two ways you can deploy in a ECS Cluster. One is a traditional way of provisioning servers before hand and deploying containers on them, which is known as method using ECS Instances . There is another way to deploy your containers in ECS, using a serverless approach of Fargate where you dont have to define hard bound resources like Instances earlier.

## What is AWS ECS?

Of the many services that AWS offers like S3 for Storage, and VPC for networking, Athena for Log Query, ECS falls into the category of a compute service. This places it in the same category as Lambda functions or EC2 instances. Containers, just in case you don’t know, are like lightweight VMs running over a host OS that offers a secure environment for the users to run their application isolated from all the other applications running on the same infrastructure. They use Linux Namespaces for process isolation so that each container has its own environment.

So, what is AWS ECS?

Well, Amazon ECS runs and manages your containerized apps in the cloud. Typically, running containers in the cloud involves spinning up compute resources, installing Docker inside them, connecting it to your container image registry, securely, and then launching your containers on top of it or over a Mesh.

The application itself is made up of multiple containers, each with its own specific nuances and attributes. So operators use tools like docker-compose to launch multiple containers. These tools themselves are under constant improvement and one update or another might render the entire stack unusable.

Amazon ECS gives you a unified approach to launch and manage Docker containers at scale in a production-ready environment.ECS is designed to be a complete container solution from top to bottom. Docker images can be hosted in Amazon ECR (AWS’s own Container Repository like DockerHub) where you can host your private image repository, create a complete CI/CD workflow(using CodePipeline and CodeDeploy) and have fine-grained access control using IAM or ACL, etc.

## Is It like Kubernetes or Docker Swarm?

Why not use Elastic Kubernetes Service from Amazon or any other container orchestration services like DC/OS, OpenShift, Kubernetes, or Docker Swarm? These technologies are all free and open source and can be run on any cloud service. For example, if your Ops team is familiar with Kubernetes, they can setup Kubernetes and run applications on any cloud, not just on AWS.

Why bother with closed-source, externally-managed ECS, when others have better alternatives?

First and foremost, because of its complexity. Kubernetes is a complex body of software with many moving parts. In order to get the most out of it, you need either to get expensive support from your hosting provider, or your team will have to go through the steep learning curve themselves.

Secondly, the return on investment, especially for startups, is very small. Running your own container orchestration incurs on an additional charge, which can be more expensive than what your application itself might cost you alone!

If you are using ECS, there’s no additional cost for using ECS. You only pay for the resources your applications consume.

Even when you are using Amazon EKS as your Kubernetes provider, you have to pay $.20 per hour for EKS control plane alone in the USA. This doesn’t even include the EC2 instances that you will have to allocate as worker nodes. Keep in mind that there will be quite a few of these EC2 instances since the application is supposed to be scalable. Here is a [beginner’s guide to creating Amazon EC2 instances](https://www.clickittech.com/aws/create-amazon-ec2-instance/).

## Features and Benefits of Using AWS ECS

Amazon ECS comes packed with every feature you may already know and love about AWS and Docker. For example, developers running Docker on their personal devices are familiar with Docker Networking modes like Docker NAT and Bridge Networking. The same simple technology running on your laptop is what you see when you launch ECS containers across an EC2 cluster.

The same services that we use for logging, monitoring, and troubleshooting EC2 instances can be used to monitor running containers as well. Not only that, you can automate the action that needs to be taken when a certain event is seen in your monitoring system. For example, AWS CloudWatch alarms can be used for auto-scaling purposes. So when the load increases, more containers are spawned to pick up the slack, but once things are back to normal, then extra containers are killed. This reduces human intervention and optimizes your AWS bills quite a bit as well.

As modern desktops programs are stored on the disk as executable files, containers are stored as container images. A single application is made up of multiple containers, and each one of these has a corresponding image.

As your app evolves, new versions of these images are introduced, and various branches for development, testing, and production are created. To manage all of this, ECS comes with another service, ECR, which acts as a private repository where you can manage all the images and securely deploy them when needed.

## You Will Need Docker

Docker is one of the key technologies that underlies all the container orchestration systems like Kubernetes, DC/OS, and Amazon ECS. Docker is what enables containers to run on a single operating system. This could be a desktop or an EC2 instance. Docker runs and manages various containers on the OS and ensures that they are all secure and isolated from one another as well as the rest of the system.

Technologies like ECS take this model of running multiple containers on a single OS and scales it up so that containers can run across entire data centers. Given the importance of Docker and its concepts, keep in mind that they are an essential prerequisite before you adopt ECS. Applications are broken down into microservices and then each one of these microservices is packaged into a Docker container.

If Docker is not a core part of your development workflow, you should probably try to incorporate it. The free and open-source Community edition is available for most desktop operating systems including Windows, Mac OS, and most Linux distros.

You can start by choosing the base image upon which to build your application. If a microservice is written in Python, there are Python base images available to get started with. If you need MongoDB for data storage, there’s an image available for that as well. Start from these building blocks and gradually grow your application as new features are designed and added.

Containers are the fundamental unit of deployment for ECS. You will have a hard time migrating to ECS if your app is not already packaged as a Docker image. Conversely, if you have a “Dockerized” application, you do not have to over complicate the task with Docker Compose or Docker Swarm. Everything else, including how the containers will talk to one another, as well as networking, load balancing, and more, can be managed on ECS platform itself.

In terms of your Docker skillset, you only need to be aware of the basic networking, volumes, and Dockerfiles.

## Tools You Won’t Need

If you are already accustomed to using Docker, there are a plethora of services that can help you deploy Docker containers. Services like Docker Compose help you deploy applications that are made up of multiple containers. You can define storage volumes, networking parameters and expose ports using Docker Compose.

However, most of these tools are limited to a single VM or Docker Swarm exclusively. Services like Docker Swarm are incompatible with AWS ECS. Creating a Docker Swarm instance typically involves launching a cluster of EC2 instances, installing Docker on all of them, running Docker Swarm, and creating a Docker Swarm out of the EC2. Then you have to install Docker Compose, write docker-compose.yaml files for each application and then deploy them.

Furthermore, you will need to maintain and update all the underlying software like Docker, Docker Compose, Docker Swarm, and then ensure that the docker-compose files are compatible with the new versions.

AWS ECS, on the other hand, takes all of that away from you. You don’t need to allocate EC2 instances for Docker Swarm master nodes. You won’t have to worry about updating any of the container management software. Amazon does that for you. You can deploy multi-container applications using a single Task definition. Task definitions replace your docker-compose.yml files and can be supplied either using the Web Console or as a JSON payload.

ECS, when used with services like AWS Fargate, can take away even the EC2 instances away. You still have to pay for compute and memory, however, your containers don’t consume all of the allocated memory and compute allocated to them, resulting in cost savings.

## Pricing for AWS ECS

The pricing model for ECS depends on one important question: where are your containers running? They can run on either Amazon Fargate or on your own EC2 cluster

If you choose the traditional way of running containers on EC2 instances then you simply pay for the EC2 prices. Every EC2 pricing policy works. You can use Spot Instances for non-critical workload, On-demand Instances, or Reserved Instances, whichever makes economic sense for your applications.

If you are using AWS Fargate to run your containers on, then the pricing consists of two independent factors:

When you define your services, you will set the values for vCPU and memory for each different kind of container you will be launching. At the end of the month, your Amazon Fargate bill would include memory utilization charges plus the CPU utilization charges.

You are billed by the second, with a minimum of one minute of usage any time you run an ECS task on Fargate. Overall, you are billed from the instant you start your task to the moment that task terminates. The pricing differs from one region to another, and you can visit this page for more details. The CPU values start from 0.25 vCPU all the way up to 4 vCPUs and each CPU value has a minimum and maximum memory that can be associated with it.

For example, a single vCPU needs at least 2GB of memory and can’t have more than 8GB of memory.

The bill you incur depends upon the way your application scales. Suppose you are running a single container and suddenly the workload spikes up. Then the application will autoscale and spawn, say, *n* more containers. This would result in *n* times normal resource utilization. Consequently, in times of peak load, you will be charged more.

AWS Fargate can save you a lot of money, if you want to run containers for batch processes like data processing and analytics. For services, like web servers, which are supposed to be active all the time, your billing would not differ all that much from EC2 prices. However, you may still want to leverage ECS for running containers over EC2, because containers come with a whole different set of advantages.

## Amazon ECS Architecture

## Tasks and Task Definition

An application consists of many microservices, and each one of these services can be shipped as a Docker image (a container image). You define an ECS task to within which the Docker image is selected, the CPU and memory allocated per container are selected. IAM roles can be associated with the task definition for granular privilege control and also various other Docker specific parameters like Networking Mode and Volumes can be specified here.

You can have multiple containers inside a single task definition, but rarely should you ever run your entire application on it. For example, if you are running a web app, a task definition can have the front-end web server image. Similarly, you can have a different task associated with your backend database.

Later, you may realize that your app can perform better if the front-end has a caching mechanism. So you can update the task definition to include a Redis container to go along with your front-end container.

To summarize, you can have multiple closely-related containers in a task. A task is run according to its task definition. The task definition can be updated to update a part of your application. Notice, you don’t touch the backend software when you update the front-end task definition.

If you are familiar with Kubernetes, then tasks are similar to pods in a Kubernetes cluster.

## Service

Remember that we still have to ensure that our application is scalable. Services are what allow us to do that. It’s the next level of abstraction on top of tasks. You can run multiple instances created from the same task definition across your entire cluster (multiple EC2 instances, for example).

Services help you autoscale your application based on CloudWatch alarms; they can have load balancers to distribute the incoming traffic to individual containers and are the interface via which one part of your application talks to another. Going back to our previous example, a web server doesn’t directly talk to the database but instead talks to the database service, which in turn, talks to the underlying containers running your database server. This is the service in a microservice-based architecture. Kubernetes has a similar concept with the same name.

## Cluster, VPC and Networking

Lastly, you may want to logically separate one set of services from another. Say you have multiple applications. You can create a different ECS Cluster for each one of them. Inside each Cluster would reside the services that make up the application and inside those services the tasks run.

Moreover, from a security standpoint, it is better to run each ECS cluster on its VPC — Virtual Private Cloud. This will provide you with a range of private IP addresses and you can further split it into subnets if you so desire. Sensitive information can reside in a different subnet with only one gateway, this way if a service has any vulnerability and gets compromised, it may not reach the sensitive stuff.

The ECS console creates a VPC for you if you don’t have one.

## AWS Fargate

We have talked a little about Fargate before and how it is different in terms of pricing from the regular EC2 clusters and how management is simpler with it. Let’s take a closer look at it.

A given ECS cluster can pool compute resources from both EC2 and AWS Fargate and schedule containers across them as and when needed. However, when you are writing task definitions you need to specify whether the task would run on AWS Fargate or is it designed for EC2.

Besides the ease of management and a highly-scalable model that AWS Fargate offers, it also offers the right environment to practice running containers in production. You don’t get the option of tweaking the underlying VM or restarting your container from the Docker host. This is important if we are ever going to run containers on bare metal servers.

The ultimate goal for cloud providers is to run containers from multiple users on the same server, instead of virtualizing the hardware and then running containers on top of it. We as application developers should no longer desire to “restart our containers” from the VM. Worse still is having an implicit assumption that your container will run in an isolated VM instead of a multi-tenant environment.

AWS Fargate doesn’t let you get away with those assumptions. Instead, it encourages cloud-native logging and monitoring solutions, fine-grained access policies and allows you to build apps that are ultimately scalable without us having to spin up more VMs or EC2 instances. Some things are still region-specific, but it is certainly a step in the right direction.

## To Summarize

Amazon ECS, from a business perspective, is easy to use as a means of learning to manage and deploy apps. It lets you run Dockerized apps across multiple EC2 instances or on Amazon Fargate without paying for control nodes or setting up Kubernetes or any other distributed system on your own.

Yes, there is always a fear that this will lead to vendor lock-ins, but Docker containers are fairly portable to begin with, so if you wish to migrate away from AWS you won’t have to rewrite your code. You can also save a significant amount of money in terms of your AWS bills if you use AWS Fargate and/or set up auto-scaling to leverage the pay-as-you-go model of AWS.

Finally, running Docker containers in production is the way going forward in the future. Adopting technologies like ECS will also make your application and your team well-prepared for the multi-tenant cloud computing environment.
