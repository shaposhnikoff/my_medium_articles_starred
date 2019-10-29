
# How to deploy and scale your app in minutes with containers and AWS ECS

When we are developing applications for our customers, we are also responsible to help them install the application in their environment and more importantly in production.

DevOps is a key activity for the success of a project and for the productivity of all the involved teams. As developers, we should also care about a few important points:

* How to install the application?

* How to test the application in a consistent manner in all the environments?

* How should we scale the application and in which conditions?

* How to provide high availability?

Containers already answer some of these questions:

* Containers enable consistency, we can run the same images in dev, test and prod

* We can spin up new containers in seconds/minutes versus days in the past to get the hardware and install all needed components

From day-1, developers should care about scalability and high availability: the application must be designed to scale horizontally. For instance, session must be stored outside of the application. Take a look at The Twelve-Factor app ([https://12factor.net](https://12factor.net)) for more tips.

Amazon Web Services is available in multiple regions in the world and provides a lot of services for almost anything (computing, storing, machine learning, etc.), fully managed services or not. For even higher availability, each region has several availability zones (isolated locations connected through low-latency links).

![](https://cdn-images-1.medium.com/max/3000/1*m5g2p254ljSZuctrKmMsog.png)

The orange circles represent existing regions with the number of availability zones. The green ones will come in a near future.

AWS will help you with your high availability challenge and expansion around the world thanks to its multiple regions available.

Also when you start a business or a new product, it’s very hard to estimate the load. With AWS you don’t have to worry about that. Just use on-demand instances when the load increase then adapt your infrastructure to your real need and optimize your cost with reserved instances.

AWS proposes several computing services but today we will explore Amazon EC2 Container Service (ECS).

In a few words, ECS is a container orchestration service provided on top of EC2 instances that is only compatible with Docker. ECS itself is free; you pay only for the underlying resources like EC2 instances, load balancer, etc. ECS can effortlessly use Amazon EC2 Container Registry (ECR) but you can also use self-hosted registry or Docker Hub.

ECR is a container registry in your virtual cloud. It allows you to spin a new private registry proposing high availability in seconds. ECR is not free; you pay for storage and outgoing traffic ([https://aws.amazon.com/ecr/pricing/](https://aws.amazon.com/ecr/pricing/)).

In the rest of this article, we will deploy a Spring Boot application offering a REST API using ECS, ECR and auto scaling following these steps:

1. Dockerize your app

1. Push the Docker image on ECR

1. Create an ECS cluster

1. Create a task definition

1. Create an Application Load Balancer and a Target Group

1. Create an ECS Service

1. Scale your cluster and your service

![](https://cdn-images-1.medium.com/max/3584/1*4U_hYT-VpyCh7xApe-EvjQ.png)

Then we will discuss how to retrieve the application logs in AWS CloudWatch. Finally, we will discuss briefly how we can use ECS for blue green deployment.

*Disclaimer: The following scenario may exceed the AWS Free Tier. Do not forget to remove the unused resources.*

## ECS basics

An ECS cluster is a logical grouping of EC2 instances, also known in this context as ECS instances or container instances. These instances are spread on several availability zones. Each EC2 instances running in an ECS cluster is running an ECS container agent. The latter communicates with ECS to provide instances information and manage the containers running on its instance.

![](https://cdn-images-1.medium.com/max/3126/1*VEb-Qz9X0b3aDjLrSh7Ziw.png)

ECS allows you to run:

* Task: A container running until it’s stopped or exits on its own. For instances, cron jobs and batch jobs.

* Service: A long lived task that runs all the time. If a service instance crashes, ECS will launch a new instance automatically. Service can be configured with load balancer and target group. It’s the perfect match for running web/REST applications.

Both are described by task definitions; a blueprint that describes the application to run and in which conditions (Docker image, container port, etc.) to ECS.

You will probably run several Dockers on the same container instance. To do so in good conditions, we encourage you to let AWS manages the hostPort for you. If you are running services with an Application Load Balancer (ALB), ECS agent will also take care to register the service, the container instance and the assigned ephemeral port to the ALB (acting as a server-side service discovery).

When the ECS scheduler launches new services/tasks, it balances them across the availability zones of the cluster. The same mechanism is used when stopping them.

When we are scaling a service in ECS, 2 variables should be used for a proper scaling:

* Cluster scaling: the number of ECS instances providing computing power to our cluster

* Service scaling: the number of containers running for a particular service/task definition in the cluster

## Run our Spring Boot application in Docker

Let’s imagine that our Spring Boot application (named talk-service) manages talks for a well-known conference. This talk service provides 2 REST handlers:

* /health which returns standard Spring Boot Actuator health JSON

* /talk which returns a message and the local hostname (InetAddress.getLocalHost().getHostName())

The hostname will allow us to show the scaling of our application on several containers.

Define the Dockerfile:

    FROM openjdk:8-jre-alpine

    ADD target/talk-service-0.0.1-SNAPSHOT.jar app.jar

    ENV JAVA_OPTS=""

    ENTRYPOINT exec java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar

    EXPOSE 8080

It’s very simple:

* We are using the small base image 8-jre-alpine

* We add the packaged über-jar

* By default, our Spring Boot application is listening on port 8080 so we expose 8080 outside the container.

* Nothing special for ECS here.

Feel free to check the following start guide for running a Spring Boot application in Docker: [https://spring.io/guides/gs/spring-boot-docker/](https://spring.io/guides/gs/spring-boot-docker/)

### Runtime vs build time

When you design your container, you should always ask yourself “How can I reduce the risk of failure?”. With ECS, we are in the context of high availability and automatic scaling. When ECS starts a container, it must work.

In order to reduce this risk, avoid using Docker image doing dynamic things at startup or keep them as simple and quick as possible to be almost certain that they won’t fail.

For instance, don’t fetch your source code, build and deploy the app when you start the Docker. Push a Docker image that embeds a precise version of your source code already built. And test this image before you push it.

## Push our Docker image on ECR

With the menu of EC2 Container Service > Repositories, you can create your own private registry in seconds.

Let’s create a registry for our talk-service app.

![](https://cdn-images-1.medium.com/max/3460/1*e1VhFOrjBYhTLGrMABGibQ.png)

Once clicked on Next step, you just need to follow the instructions given by AWS Console. Note that AWS CLI must be installed ([http://docs.aws.amazon.com/cli/latest/userguide/installing.html](http://docs.aws.amazon.com/cli/latest/userguide/installing.html)) and configured ([http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-quick-configuration](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-quick-configuration)) (credentials, region, etc.).

![](https://cdn-images-1.medium.com/max/2000/1*B9ugjkk6OnpgBPysn9fQTg.png)

## Create an ECS cluster

Let’s start getting our hands dirty and create an empty ECS cluster step by step with the help of AWS CLI and AWS Management Console.

We will try to put in place the following empty cluster:

![](https://cdn-images-1.medium.com/max/3584/1*ggFMix1LnQzghgGS6B5JvQ.png)

First, you should create a new security group named conferenceEcsSecurityGroup for our ECS cluster using the AWS CLI:

    aws ec2 create-security-group --group-name conferenceEcsSecurityGroup --description "Conference ECS security group"

Then, create a new ECS cluster that uses the previously created security group with the help of the AWS Management Console: EC2 Container Service > Clusters > Create Cluster

![](https://cdn-images-1.medium.com/max/3304/1*qW-v_gLA6EBk6qdLTH539A.png)

Remarks:

* t2.micro is enough for our Spring Boot application.

* Keep in mind that a container instance cannot span on several EC2 instances: for instance if you choose t2.micro but you need more memory for your container, the container will not span on several t2.micro instances.

* Always think that something might go wrong during the execution of the cluster, so you need fallback instance(s): 2 instances will be provisioned.

* ECS-Optimized AMI ([http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-optimized_AMI.html](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-optimized_AMI.html)) is selected by default. It’s optimized for ECS and proposes also several tools for integrating the rest of the AWS stack. Later in this article, we will use its CloudWatch Logging integration.

* A SSH key pair can be added in order to test and debug the underlying EC2 instances.

* We use the default VPC and attach the subnets corresponding to the 3 availability zones of our region (note that in Ireland we have the chance to have 3 availability zones so we added all of them)

* Use the ecsInstanceRole IAM role

Tip: You can use [http://selec2or.info](http://selec2or.info) for comparing the EC2 instances.

Once the cluster is created, you can view the running instances in Clusters > conference-cluster > ECS Instances

![](https://cdn-images-1.medium.com/max/3804/1*nDllkt3572K2Cc3G-afomw.png)

As you can see, 2 instances have been provisioned in 2 availability zones but nothing is running (Running tasks count column set to 0). We are not yet using the 3 availability zones because we don’t have enough instances. If one more instance is added (by us or by auto scaling), it will be added to the third availability zone.

## Create a task definition for our Spring Boot talk-service

Once our Docker image is available on our ECR registry, we can create a task definition using this image.

We propose to create the task definition with the help of AWS CLI and a JSON task definition:

    {
      "family": "talk-service",
      "containerDefinitions": [
        {
          "name": "web",
          "image": "340754841610.dkr.ecr.eu-west-1.amazonaws.com/talk-service:latest",
          "cpu": 128,
          "memoryReservation": 128,
          "portMappings": [
            {
              "containerPort": 8080,
              "protocol": "tcp"
            }
          ],
          "command": [],
          "essential": true
        }
      ]
    }

In a few words, you should:

* Give a name to your task definition

* Specify the target image. Note that here you can use public Docker Hub, ECR or any self-managed registry

* You can tune the CPU and memory reservation according to your application needs

* The container port on which your image is listening

Note that when you don’t specify the host port (hostPort) or set it to 0, the container will automatically receive a port in the allowed ephemeral port range. It allows running several containers on the same ECS instance without conflict.

Then we will register this task definition with AWS CLI:

    aws ecs register-task-definition --cli-input-json file://talk-service-task-definition.json

Once created, verify that the task definition has been created in ECS:

![](https://cdn-images-1.medium.com/max/3304/1*kke5QxNS-lrANcgAcPBrYg.png)

## Create an Application Load Balancer and a Target Group

Now we have an empty cluster with provisioned ECS instances and a task definition describing a Docker image to run.

Before creating the running service based on this task definition, we will create an Application Load Balancer and a Target Group in order to balance the load across possible various services instances.

Go to EC2 > Load Balancing > Load Balancers > Create Load Balancer, then select Application Load Balancer.

Give a name to your Load Balancer, specify the listening port (in our case 80) and select the subnets corresponding to the target availability zones.

![](https://cdn-images-1.medium.com/max/3784/1*XwrjUemriO0cu5X5td9EgQ.png)

![](https://cdn-images-1.medium.com/max/3572/1*HtzmFSWEEBLfIpErXbKQWg.png)

![](https://cdn-images-1.medium.com/max/3516/1*GgMi11LgbXHRb8mfpNXhvw.png)

Once you press on Next: Configure Security Settings, you will have a warning regarding the non-use of SSL. You can ignore this warning for this test (don’t do that in production J).

Then we configure a new Security Group for our Load Balancer. With 0.0.0.0/0, ::/0, we allow all the traffic coming from Internet (outside AWS).

![](https://cdn-images-1.medium.com/max/3528/1*218btePTSrkT-wS4K28bYA.png)

In step 4, we create a Target Group and specify how to ensure the instance is still up. Here we will use the /health implemented by Spring.

![](https://cdn-images-1.medium.com/max/3236/1*kIxYye9vyHdORISyuf01oA.png)

In step 5, AWS Console proposes you to register targets. Ignore this step as ECS will do it for you automatically.

Then Review and confirm the creation.

Once created, you must allow the traffic from the ALB (conferenceAlbSecurityGroup Security Group) to the ECS cluster (conferenceEcsSecurityGroup Security Group).

    aws ec2 authorize-security-group-ingress --group-name conferenceEcsSecurityGroup --protocol tcp --port 1-65535 --source-group conferenceAlbSecurityGroup

## Define an ECS service

Now, we will create an ECS service and attach this service to the target group/load balancer previously created.

First, retrieve the ARN identifier of the target group by accessing EC2 > Load Balancing > Target Groups then select talkServiceTargetGroup. Copy and store somewhere the ARN id of this target group.

![](https://cdn-images-1.medium.com/max/2000/1*ToBlILi2moGYvj_N8bpOPg.png)

Then, replace the placeholder YOUR_TARGET_GROUP_ARN_HERE by the previously retrieved ARN.

Note that our service is named talk-service-ecs and the container port is 8080.

    {
      "cluster": "conference-cluster",
      "serviceName": "talk-service-ecs",
      "taskDefinition": "talk-service",
      "loadBalancers": [
        {
          "targetGroupArn": "YOUR_TARGET_GROUP_ARN_HERE",
          "containerName": "web",
          "containerPort": 8080
        }
      ],
      "desiredCount": 2,
      "role": "ecsServiceRole"
    }

Now register your ECS service by executing the following AWS CLI command with the previously edited file.

    aws ecs create-service --cli-input-json file://talk-service-ecs.json

Check the service has been correctly registered by navigating to EC2 Container Service > conference-cluster > Services. You should see the newly created talk-service-ecs service.

![](https://cdn-images-1.medium.com/max/3688/1*sUst9YRf9mwafPPNeZ_Q_Q.png)

## Check that the application is up and running

We can now verify that everything is up and running by requesting the ALB. In order to do so, you should retrieve the DNS name by navigating to EC2 > Load Balancing > Load Balancers then select conferenceAlb. Copy the DNS name.

![](https://cdn-images-1.medium.com/max/2000/1*I_BtzV5eTkHHJTJfB4i_7A.png)

If you remember ALB is listening on port 80, so you can try /talk: [http://conferencealb-1172803140.eu-west-1.elb.amazonaws.com/talk](http://conferencealb-1172803140.eu-west-1.elb.amazonaws.com/talk)

If you refresh several times, you will see different responses. This is the expected result because it’s the generated hostnames of our running containers (2 in this example).

If you have selected an SSH key pair during the ECS cluster creation, you should be able to connect the ECS instances and run: docker ps to list the running containers.

## Application Load Balancer for micro-services

ALB is very interesting for those running micro-services as a single ALB instance is able to manage several target groups/micro-services. Also, when a service instance is added/removed, the target group and the ALB detect the change almost instantly.

For instance, in the following ALB configuration form we configured 2 target groups managed by ECS. Both are running on the same port (80) and take advantage of path-based routing to route the traffic to the right micro-service.

![](https://cdn-images-1.medium.com/max/3908/1*KU93RHLMIBmoIa3aKiUx7Q.png)

## Scale your cluster and your service

Now let’s imagine that our talk-service must scale because of its big success. We would take advantage of the elasticity provided by AWS and only allocate resources when needed.

As depicted in the introduction, ECS should scale based on 2 variables: the number of EC2 instances and the number of running services/tasks on these instances.

### Scale your cluster

Go to EC2 > Auto Scaling > Auto Scaling Groups. AWS has created for you an Auto Scaling group corresponding to your ECS cluster.

![](https://cdn-images-1.medium.com/max/4228/1*Z1HNsUPCoT3DamlL-l98_g.png)

By default, we have 2 running instances, 2 desired instances, 0 minimum instance and maximum 2 instances. We should first edit these limits by selecting the group and clicking on button Actions > Edit. We will set 2 minimum instances and 4 maximum instances.

Then use the Scaling Policies tab and add a new Policy in order to add or remove instance(s) in specific conditions. In the following example, we add 2 instances when the CPU utilization is higher or equal to 30% for at least 60 seconds. You can also design your own alarm with CloudWatch.

Remark: When you are scaling out (in ECS or any other service), you should always keep in mind that something might go wrong. What happens if we have a peak of users and the new EC2 instance crashed at startup? You can reduce this risk by adding 2 instances even if a single one is enough.

![](https://cdn-images-1.medium.com/max/4268/1*L31cWTWaExwCdtdngv_-dQ.png)

In order to take full advantage of the scaling policy and reduce your cost, you should also put in place a scale-in policy in order to remove unnecessary instances. Here we propose to remove them 1 by 1.

### Scale your service

Now, we should apply the same principles on the ECS services. So ask yourself, in which conditions you need to scale-in and scale-out my service (on top of my ECS cluster)? Then you can design this rule in CloudWatch and attach a scaling policy to this alarm.

To do so, go to EC2 Container Service > Clusters > conference-cluster > Services tab > talk-service-ecs then click on Update.

![](https://cdn-images-1.medium.com/max/3312/1*ljF1HDnI6NHsy0pnVfhLPg.png)

Move to Step 3: Auto Scaling

![](https://cdn-images-1.medium.com/max/2812/1*e457KycGzmwO0CsWgQ3v-Q.png)

You can add your scaling policies with the help of standards alarms or custom ones designed in CloudWatch.

![](https://cdn-images-1.medium.com/max/4888/1*uM3Jcz2rzpha7o43FCOqqA.png)

## Push logs in CloudWatch

Now your application is up and running and you want to aggregate the logs.

To do so, you can integrate CloudWatch, a managed service of AWS allowing you to monitor your cloud resources and your applications. If you are running ECS cluster using ECS optimized AMI, you can enable CloudWatch log in two simple steps.

First you create a log group in CloudWatch using the AWS CLI:

    aws logs create-log-group --log-group-name talk-service-cloudwatch --region eu-west-1

Nothing special in this command, you specify the name of your log group and the region.

Then, you adapt the task definition we defined previously by enabling the log and specifying the log group configuration:

    {
      "family": "talk-service",
      "containerDefinitions": [
        {
          "name": "web",
          "image": "340754841610.dkr.ecr.eu-west-1.amazonaws.com/talk-service:latest",
          "cpu": 128,
          "memoryReservation": 128,
          "portMappings": [
            {
              "containerPort": 8080,
              "protocol": "tcp"
            }
          ],
          "logConfiguration": {
            "logDriver": "awslogs",
            "options": {
              "awslogs-group": "talk-service-cloudwatch",
              "awslogs-region": "eu-west-1",
              "awslogs-stream-prefix": "aws-talk-service-cloudwatch"
            }
          },
          "command": [],
          "essential": true
        }
      ]
    }

Then you update your definition in AWS by running the following AWS CLI command:

    aws ecs register-task-definition --cli-input-json file://talk-service-task-definition**-with-cloudwatch**.json

**<remark>**

If you are fast enough, you can see the rolling upgrade in Clusters > conference-cluster > talk-service-ecs > Tasks tab. As you can see in the following screenshot, during a certain amount of time both versions of the service are executed in parallel. When the new instances are up and running normally, ECS will automatically remove the previous version.

![](https://cdn-images-1.medium.com/max/3300/1*2InGKIaL8dSvFnwqt4POZw.png)

**</remark>**

Once deployed, this configuration will fetch the logs from the container and push it on CloudWatch (for more details, please refer to [http://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_cloudwatch_logs.html](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_cloudwatch_logs.html)).

With this simple first configuration, you will be able to see your logs in near-realtime in CloudWatch AWS Console:

![](https://cdn-images-1.medium.com/max/3076/1*ui0LadCfP1MytOhtQY5Log.png)

You will see one log stream per running tasks/services.

If you click on a stream, you will see the logs of the Spring boot app:

![](https://cdn-images-1.medium.com/max/3420/1*Q49p5uxtMDfoubZvxrELiw.png)

## Blue green deployment at your reach

Now, we will discuss how you can take advantage of ECS for adopting blue green deployment.

<iframe src="https://medium.com/media/0435f156ad1e3ec9f29076aa4d85a897" frameborder=0></iframe>

When performing blue green deployment, you run two versions of your app in production:

- The blue version (version n) currently used by your clients

- The green version (version n+1), the new version of your application

Once you are satisfied with the green version, you can reroute the traffic to the instance(s) of the green version. If something goes wrong, you can quickly revert your changes and reroute the traffic back the blue version (also known as fast rollback).

Blue green deployment is a classical pattern for zero downtime deployment and to reduce the risk of each deployment.

Putting in place blue green deployment on premise is expensive and not easy because:

* You have to buy additional machines

* You must install them

* And you will use these machines only during these parallel runs…

In the cloud you can take advantage of on-demand provisioning and pay only for the amount of times you need for your blue green swap.

### Run them on AWS ECS

With ECS, you can run the green version of your application on the same cluster as the blue version. Each version will have its own target group. If needed (and enabled), the cluster will auto scale by adding ECS instances.

Once both of them are up and running, you can reconfigure the application load balancer in order to switch the traffic from version blue to green and vice versa.

Once switched to the new version, you can continue to run the previous one for safety reason during several days. Then you disable the old version of the service, after a few moments the cluster will scale-in the number of ECS instances.

## Conclusion

Using Containers, a good design, AWS and AWS ECS, we are able to improve consistency, scalability and high availability.

To illustrate these concepts and services we deployed an application on AWS ECS, we scaled it, we updated its definition on the fly, we pushed the logs in CloudWatch and we investigated how to use ECS for blue green deployment.

As general rules, always be prepared for the worst and try to simulate these scenarios. You adapt your autoscaling strategy? Create a distributed load test and try to reach the limits of your system.

Feel free to ask question to Arnaud Koster ([https://twitter.com/arnaudkoster](https://twitter.com/arnaudkoster))

Edit: Java projects available on GitHub:

* [https://github.com/kosterar/talk-service](https://github.com/kosterar/talk-service)

* [https://github.com/kosterar/spearker-service](https://github.com/kosterar/spearker-service)
