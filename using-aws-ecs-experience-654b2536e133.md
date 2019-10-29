
# Using AWS ECS Experience

We were having this project that still have traditional way to code and deploy. The project was outsource and after we accept the code, we felt it’s kinda hard to do build, deploy and scaling things. So we decide to dockerize the project for making it easier later on for building / packaging and deployment, also adapt the CI/CD for the whole deployment things.

After we pretty much dockerize the project, we need a services to orchestrate the docker services. But at the meantime we don’t really have any experiences on the uprising kubernetes and i also found that the learning curve of kubernetes is high.

Originally we were using AWS for our project. We found AWS also has their docker orchestrator services called ECS (Elastic Container Service). After did some research, ECS was said is more simple and not that complex as kubernetes is. So we decide to give AWS a chance, and i will share my experience on AWS ECS after using it for a quite some time :)

## ECS Elements

![](https://cdn-images-1.medium.com/max/4772/1*PLBRRk1M7Ym4Y9K93G5Qng.jpeg)

## Service

Service will allow you to define specific number on running task definitions (similar like replicas). Also the minimum healthy percentage, maximum healthy percentage of the task definitions (will discussed a little bit more later on), load balancer.

## Container Definitions

### Static / Dynamic Ports

When you using a static fixed ports for a container definitions, it won’t allow you to run the task in the same instance because of the port conflicts. Nevertheless, there is a combination of using dynamic ports and ALB (Application Load Balancer) to allow multiple task running in the same instances. The distribution of ports is controlled by AWS itself that they will check which port is available right now to use. To define the dynamic ports you just input **0 **for $HOST_PORT.

## Task Definitions

For those who have already learn kubernetes we know **pods **is the smallest unit. It is a group of containers or maybe just a single container that running inside. In ECS however, **task definitions **is categorize as same as pods. It is like a blueprint of your application, you can set up the network mode, container definitions, attach volume to your container definitions.

AWS ECS has 3 kind of network mode, each work differs from others.

### Host

Service will directly us the host network itself (Host IP).

![](https://cdn-images-1.medium.com/max/4772/1*ILjIqzKX7DBKrmY8t7NAyQ.jpeg)

### AWS VPC

Service network will act independently, every task definition will get their own ENI (Elastic Network Interface) which means have their own private IP address. But each instances type(ECS worker nodes) have different limitation for giving ENI.

![](https://cdn-images-1.medium.com/max/4772/1*jPE2kViVWOwvOtCvg9vThA.jpeg)

### Bridge

Bridge network concept still the same as default bridge network of docker.

![](https://cdn-images-1.medium.com/max/4772/1*z6R6h5BQVKG-yHh0etwWTw.jpeg)

## ALB Target Group (Instance or IP)

When you define service, you can choose if you want to map it with load balancer to distribute the traffic or not. For now we just gonna mention Application Load Balancer as it’s the most common one.

When you create ALB, it need to have target group to allow ALB know where should it forward the traffic. You can have multiple target group for different ports on ALB.

Target Group have two types:

* Instance (host network, bridge network)

* IP (awsvpc network)

### Instance Target Group

![](https://cdn-images-1.medium.com/max/4772/1*RZwzBY-1g9-ZxZ1DLgdZ6Q.jpeg)

ALB will forward the traffic from the defined port to listed target group below

* 192.0.0.1:1234

* 192.0.0.1:3333

* 192.0.0.2:1234

* 192.0.0.2:3331

Using **instance **type is suitable for bridge network with dynamic port.

### IP Target Group

![](https://cdn-images-1.medium.com/max/4772/1*hZp_a7d5A1Nm7VzYufd_xg.jpeg)

ALB will forward the traffic from the defined port to listed target group below

* 192.2.2.1:3000

* 192.2.1.1:3000

* 192.2.7.3:3000

* 192.2.8.1:3000

Using **IP **type is more suitable for awsvpc network. For this configuration you dont need to use dynamic port to achieve zero down time when update your service. You can just have static port on host and will not have port conflict because of different primary IP address.

## Rolling Update

Beforehand, we need to have more understanding how the rolling update works. There is a **Minimum Healthy Percentage** and **Maximum Healthy Percentage **when you define a service. Let’s say we have **web-backend** service that have below settings:

* Task Definitions: web-backend

* Task Definitions Tag: 1.0

* Replicas: 1

* Minimum Healthy Percentage: 100%

* Maximum Healthy Percentage: 200%

Above definition explain it will only run one replicas at the time / start. The minimum healthy service to alive need to be at least **100% * 1 = 1 **replica available. The maximum healthy service can exist or alive at one time is **200% * 1 = 2** service at max (either from rolling update or auto scaling).

Let’s say you want to update the service to 2.0. ECS then will deploy v2.0. Hence, you have two replicas running right now. One of them v1.0, and another one v2.0. *Notice we can have two services running at the same time because of maximum healthy percentage that we set before is 2.* If we set to 100% which means you only can have 1 replicas running, then you must stop or delete the running replicas first before deploying the newer version. However, there is downtime and we do not want that.

In the beginning, even both the version are running. All the traffic still forwarded to the v1.0. Later on, after ECS really make sure the v2.0 running properly, ALB will open the connection to v2.0 and make it primary (Still have 2 service running here). After some time ALB will drain connection of v1.0, then kill the v1.0 make it only 1 replicas of v2.0 running.

There are still few concept missing here since i have not play it yet. Maybe will the missing part in the future. Feel free to correct me if i am wrong. :)

*Missing part*:

* *Blue/Green Deployment*

* *Fargate*
