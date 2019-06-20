
# A beginner’s guide to Amazon’s Elastic Container Service

Image source.

This article is a beginner’s high level look at Amazon ECS. We’ll cover core concepts, terms, simple architecture diagrams, and abstracted examples. So let’s get started!

## Docker

To appreciate Amazon ECS, you first have to understand Docker.

Docker is a client-server application that can be installed on Linux, Windows, and MacOS and that allows you to run Docker [containers](https://en.wikipedia.org/wiki/Operating-system-level_virtualization). Containers are lightweight environments containing everything needed to run a specific application or part of an application. Multiple different containers can be run on one machine, so long as it has the Docker software installed.

If you’re interested in **how*** *they work, and how Docker is different from a virtual machine, then this [intro to Docker](https://medium.com/pintail-labs/docker-series-what-is-docker-9eddca88f434) is a great place to start.

![Adapted from Docker’s ‘[get started](https://docs.docker.com/get-started/#container-diagram)’, see here for ‘[bins/libs](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)’](https://cdn-images-1.medium.com/max/2000/1*wTvG0T8O5FtZMk4Cx5OX6Q.png)*Adapted from Docker’s ‘[get started](https://docs.docker.com/get-started/#container-diagram)’, see here for ‘[bins/libs](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)’*

Using Docker containers allows teams to have a consistent development environment by abstracting away the software, operating system, and hardware configuration into a standard building block that can be run on any machine.

Each container has exactly what it needs — for example, certain versions of a language or library — and no more than it needs. Multiple containers can be used for different parts of your application if you want, and they can be set up to communicate with each other when needed.

By using specified Docker containers to run your production code, you can be sure that your development environment is exactly the same as your production environment.

As your application grows, managing the deployment, structure, scheduling, and scaling of these containers rapidly becomes very complicated. This is where a “container management service” comes in. It aims to allow simple configuration options and handles the heavy lifting while you go back to writing the app.

## An intro to Amazon ECS

Amazon [Elastic Container Service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html) (ECS) is, according to Amazon,
> # *…a highly scalable, fast, container management service that makes it easy to run, stop, and manage Docker containers on a cluster.*

It is comparable to [Kubernetes](https://kubernetes.io/), [Docker Swarm](https://docs.docker.com/swarm/overview/), and [Azure Container Service](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/microsoft.acs).

![](https://cdn-images-1.medium.com/max/2468/1*F_Av332fsAf6RJ97gM2h8w.png)

ECS runs your containers on a cluster of [Amazon EC2](https://aws.amazon.com/ec2/) (Elastic Compute Cloud) [virtual machine instances](https://www.youtube.com/watch?v=TsRBftzZsQo) pre-installed with Docker. It handles installing containers, scaling, monitoring, and managing these instances through both an API and the AWS Management Console. It allows you to simplify your view of EC2 instances to a pool of resources, such as CPU and memory. The specific instance a container runs on, and maintenance of all instances, is handled by the platform. You don’t have to think about it.

It’s worth noting that it is tied into the Amazon infrastructure, unlike some other providers that allow more flexibility. However, that means it comes with excellent integration with other AWS services.

## Terms and architecture

Let’s give some imaginary context for the definitions we are about to look at. Say you are building an application that runs on two Docker containers, perhaps one for the main application, and one for managing metrics. Both are needed for the application to run as intended. If you had large amounts of traffic, you might need to run several pairs of containers.

Here we come to two sets of new terms:

* a **Task Definition**, **Task**, and **Service**, and

* a* ***Cluster**,* ***ECS Container Instance**,* *and* ***ECS Container Agent**.

### Task Definition

This is the blueprint describing which Docker containers to run and represents your application. In our example, it would be two containers. would detail the images to use, the CPU and memory to allocate, environment variables, ports to expose, and how the containers interact.

### Task

An instance of a Task Definition, running the containers detailed within it. Multiple Tasks can be created by one Task Definition, as demand requires.

![One Task Definition creates several identical Tasks](https://cdn-images-1.medium.com/max/2440/1*LYshH6EUVzEJBf0lirXJrQ.png)*One Task Definition creates several identical Tasks*

### Service

Defines the minimum and maximum Tasks from one Task Definition run at any given time, autoscaling, and load balancing. In our example, if the CPU was maxed out from the single task we had running, we may want it to add an additional Task.

We may, however, want to limit the maximum number of Tasks it can run, since we know that running extra Tasks uses additional resources that cost money.

![Service definition defining alarms of when to scale capacity](https://cdn-images-1.medium.com/max/2644/1*Js04HCWO-lZu-ZxuSymD5Q.png)*Service definition defining alarms of when to scale capacity*

Now that we have our Service, its Tasks need to be run somewhere in order to be accessible. It needs to be put on a **Cluster,** and the container management service will handle it running across one or more **ECS Container Instance(s)**.

### ECS Container Instances and ECS Container [Agents](https://github.com/aws/amazon-ecs-agent)

![One ECS Container Instance running 8 Tasks from multiple different Services](https://cdn-images-1.medium.com/max/2464/1*AgRJVpubxlnFFCJzmeM8Aw.png)*One ECS Container Instance running 8 Tasks from multiple different Services*

This is an [EC2 instance](http://AWS EC2 for Beginners) that has Docker and an ECS Container Agent running on it. A Container Instance can run many Tasks, from the same or different Services.

The Agent takes care of the communication between ECS and the instance, providing the status of running containers and managing running new ones.

### Cluster

![An example ECS cluster, with one Service running four Tasks across two ECS Container Instances](https://cdn-images-1.medium.com/max/3204/1*LAyPZNXwFXuNNjf7sPO3BQ.png)*An example ECS cluster, with one Service running four Tasks across two ECS Container Instances*

As seen above, a Cluster is a group of ECS Container Instances. Amazon ECS handles the logic of scheduling, maintaining, and handling scaling requests to these instances. It also takes away the work of finding the optimal placement of each Task based on your CPU and memory needs.

A Cluster can run many Services. If you have multiple applications as part of your product, you may wish to put several of them on one Cluster. This makes more efficient use of the resources available and minimizes setup time.

![Multiple Services allocated across multiple ECS Container Instances running on one Cluster](https://cdn-images-1.medium.com/max/4200/1*7glxzsjBabpq1T1Ax1tsUg.png)*Multiple Services allocated across multiple ECS Container Instances running on one Cluster*

## Conclusion

We have seen how a Dockerized application can be represented by a **Task*** ***Definition** that has a one-to-one relationship with a **Service** which in turn uses it to create many different **Task** instances.

This **Service*** *is deployed to a **Cluster*** *of* ***ECS Container Instances*** *that provide the pool of resources needed to run and scale your application. Additional Services can be deployed to the same Cluster.

Amazon ECS, or any container management service, aims to make this as simple as possible, abstracting away many complexities of running infrastructure at scale.

![A Cluster running 3 Services, each running a different amount of Tasks, across two ECS Container Instances](https://cdn-images-1.medium.com/max/2000/1*daNyrp9nG1OfGTNTe0KN4w.png)*A Cluster running 3 Services, each running a different amount of Tasks, across two ECS Container Instances*

As your needs become more complex, the container management service ensures this remains manageable. Using its API or Management Console, you can put definitions in place to add new Container Instances as you need them. This makes sure that there are always a healthy number of Tasks running, and intelligently allocates resources across Services.

Thanks for reading!

### Resources

* [Gentle Introduction to How AWS ECS Works with Example Tutorial](https://medium.com/boltops/gentle-introduction-to-how-aws-ecs-works-with-example-tutorial-cea3d27ce63d)

* [Deploying Clustered Akka Applications on Amazon ECS](https://medium.com/@ukayani/deploying-clustered-akka-applications-on-amazon-ecs-fbcca762a44c)

* [Building Blocks of Amazon ECS](https://medium.com/containers-on-aws/building-blocks-of-amazon-ecs-db7fdfeeaa6f)

* [Introduction to Amazon EC2 Container Service (ECS) — Docker Management on AWS](https://www.youtube.com/watch?v=zBqjh61QcB4)

* [Amazon ECS: Core Concepts](https://www.youtube.com/watch?v=eq4wL2MiNqo)

* [AWS EC2 for Beginners](https://hackernoon.com/aws-ec2-for-beginners-56df2e820d7f)

* [A Better Dev/Test Experience: Docker and AWS](https://medium.com/aws-activate-startup-blog/a-better-dev-test-experience-docker-and-aws-291da5ab1238)

* [Cluster-Based Architectures Using Docker and Amazon EC2 Container Service](https://medium.com/aws-activate-startup-blog/cluster-based-architectures-using-docker-and-amazon-ec2-container-service-f74fa86254bf)
