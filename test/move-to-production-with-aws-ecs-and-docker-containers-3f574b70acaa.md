
# Move to Production with AWS ECS and Docker Containers

ESC is basically managed services that runs docker container on EC2 ( AWS) . Starting from simple web based or nodejs application ,you can run complex micro services on ECS .ECS is essentially a cluster scheduler that maps a set of work (batch jobs or long-running services) to a set of resources (EC2 instances)

![](https://cdn-images-1.medium.com/max/2732/1*jQjwTucibIX91zgagBhXcw.png)

## Building Blocks

**Instances **— ECS provides 2 types of Instances to run services
1. **EC2 **— You need to manage Instances 
2. **FARGATE **— Instances manage by AWS , you do not need to worry about instances.

**Configuration **AMI , Availability zone , Load balancer , Target group , region , route 53 — I assume audience know about it. Or you can read it on AWS reference guide

**Container **Containers are ready to ship build code that you can run anywhere.There are two types of containers I worked upon Docker and Vagrant. I will share my experience with Docker containers. The reason is it is widely use and has huge community support.

**ECR **is a amazon repository to store docker container , You can create your own repository.

Below are the definition taken from AWS Reference

**Cluster **— An Amazon ECS cluster is a logical grouping of tasks or services. If you are running tasks or services that use the EC2 launch type, a cluster is also a grouping of container instances

**Task Definitions **— A task definition is required to run Docker containers in Amazon ECS. Some of the parameters you can specify in a task definition include:
> The Docker images to use with the containers in your task
How much CPU and memory to use with each container
The launch type to use, which determines the infrastructure on which your tasks are hosted
Whether containers are linked together in a task
The Docker networking mode to use for the containers in your task
(Optional) The ports from the container to map to the host container instance
Whether the task should continue to run if the container finishes or fails
The command the container should run when it is started
(Optional) The environment variables that should be passed to the container when it starts
Any data volumes that should be used with the containers in the task
(Optional) The IAM role that your tasks should use for permissions

**Services **— Amazon ECS allows you to run and maintain a specified number (the “desired count”) of instances of a task definition simultaneously in an Amazon ECS cluster. This is called a service

Hands On — let us start hosting a service on ECS

**Step 1- Create a ECR**

This is first we have to create a repository where we can push or docker images.
*Open AWS console → ECS → Repository*
Create a repository give a proper name like wishrupt , click next
You repository has been created now . You will get all you commda ro push images to repository
*592180788752.dkr.ecr.us-west-2.amazonaws.com/wishrupt*

**Step 2- Push Docker Images to ECR**

Install AWS CLI on local machine and use below commands to push images on ECR .Preference will be any EC2 instances

Let us say I have 3 services that I need to deploy on ECS , Wishrupt a web application , sales a sales serviced , account a account related service. All are running on port 80

Build you image Docker build .

**Tag to ECR**

docker tag wishrupt:develop 592180788752.dkr.ecr.us-west-2.amazonaws.com/wishrupt:develop

docker tag wishrupt:develop 592180788752.dkr.ecr.us-west-2.amazonaws.com/account:develop

docker tag wishrupt:develop 592180788752.dkr.ecr.us-west-2.amazonaws.com/sales:develop

**Push to ECR**
> docker push 592180788752.dkr.ecr.us-west-2.amazonaws.com/sales:develop
> docker push 592180788752.dkr.ecr.us-west-2.amazonaws.com/account:develop
> docker push 592180788752.dkr.ecr.us-west-2.amazonaws.com/wishrupt:develop

**Step 3- Register Task Definition**
AWS Console → ECS → TaskDefinition →Fill the details

name : wishrupt-task 
Container name : wishrupt-webapp-container
image : 592180788752.dkr.ecr.us-west-2.amazonaws.com/wishrupt:develop
memory limits : 512
port mapping : 80
CPU units : 1
Env variables : if any
Log Configuration : if you wish to put logs on Cloud logs
Volumes : if you wish to mount volume
Repeat the same for account and sales service

**Step 4- Create a cluster**

AWS Console → Elastic containers services → Create Cluster

Fill required information

Provide required info like 
Instance type 
FARGET / EC2
Select EC2 Linux + Networking
Name 
Wishrupt-prod
Cluster Name : wishrup
EC2 Instance Type : choose t2.medium
Number of Instances : 2
Key pair : wisrupt.pem
VPC : wishruptVPC
Subnets : sb-wishrupt
Security Group : sg-wishrupt
Click “Create”

Please note you have to create/use your use your own VPC , keypair , subnet , security group

**Step 5 — Service**

Now we have created task definition and cluster let us move to last step that is Service.

AWS Console → Elastic containers services → Click on Created Cluster → Service ( create)

Launch Type : EC2
Task Definition : select from above wishrupt-task 
Cluster : wishrupt
Service name : wishrupt-web-service
Number of Tasks : 1
Load Balancer type : Application Load Balancer
Load balancer name : wishrupt-ALB
Target Group Name : wishrupt-tg
Click Finish

**Step 6 — Test**

Once the Service is created it will take some time to be in running state. Once it is running you can click in service , you will get the IP address of running instance. You can hit the IP and test your service

Using ALB — While creating you must have provide ALB for the same. Please use your ALB DNS to hit service
