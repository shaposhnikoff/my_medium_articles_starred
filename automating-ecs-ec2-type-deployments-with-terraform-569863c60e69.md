
# Automating ECS-EC2-Type Deployments with Terraform



## **AWS Elastic Container Service (ECS)**

ECS is Amazon’s Elastic Container Service. That’s greek for how you get docker containers running in the cloud. It’s sort of like Kubernetes

Amazon Elastic Container Service (Amazon ECS) is a scalable, high-performance container orchestration service that supports Docker containers and allows you to easily run and scale containerized applications on AWS. ECS eliminates the need for you to install and operate your own container orchestration software, manage and scale a cluster of virtual machines, or schedule containers on those virtual machines

### On this page:-

* Creating ECR registry for storing the docker image

* Creating Dockerfile and building the image

* Creating terraform code for IAM role

* Creating tf file for ECS-EC2-instance

* Creating ECS Task Definition

* Creating ECS Service

* Creating Application Load Balancer

* Creating Route 53 hosted zone

* Creating cloudwatch log group

* Creating terraform code for IAM role

### ECR Registry:-

Creating Terraform code for ECR repository

<iframe src="https://medium.com/media/be553ead1070515ea6953e141d6bd4ec" frameborder=0></iframe>

Creating a Docker file and build the image with below command

-Note*- Make sure you have made the connection with awscli:- **aws configure**

    - docker build -t swagger .

    - docker tag swagger:latest xxxxxAWS-ACCOUNT-NOXX.dkr.ecr.eu-west-1.amazonaws.com/swagger:latest

    - docker push xxxxxAWS-ACCOUNT-NOXX.dkr.ecr.eu-west-1.amazonaws.com/swagger:latest

Dockerfile:-

<iframe src="https://medium.com/media/dce44610a91716530449bd5ae39832c0" frameborder=0></iframe>

### Creating an IAM role and assigning Policy**:-**

Roles are a really brilliant part of the aws stack. Inside of IAM or identity access and management, you can create roles. These are collections of privileges. I’m allowed to use this S3 bucket, but not others. I can use EC2, but not Athena. And so forth. There are some special policies already created just for ECS and you’ll need roles to use them.

These roles will be applied at the instance level, so your ecs host doesn’t have to pass credentials around

* ecs-instance-role

* ecs-service-role

* ecs-instance-profile

<iframe src="https://medium.com/media/bfc40d4476140a1849a3397733b77705" frameborder=0></iframe>

### Elastic container service (ECS-EC2-Type):-

Here we are going to create the ECS cluster with launch type as EC2-TYPE. This involves the following resource.

1. ECS-Cluster.tf

1. ECS-ec2-instance.tf

1. ECS-task-defination.tf

1. ECS-services.tf

1. ECS-ALB.tf

1. ECS-cloudwatch.tf

1. ECS-Route53.tf

### ECS-Cluster.tf:-

<iframe src="https://medium.com/media/5abbc1f0c30c1f5ca735d2f9bb998bd6" frameborder=0></iframe>

### ECS-ec2-instance.tf :-

<iframe src="https://medium.com/media/b3d66f7df764b990bf3437ddc3a7c943" frameborder=0></iframe>

user_data.tpl

<iframe src="https://medium.com/media/4fafab59bf9fdbc16e0d15105116f6f1" frameborder=0></iframe>

### ECS-task-defination.tf:-

<iframe src="https://medium.com/media/1ae05776011aa31e92ab6f0c1d7df82d" frameborder=0></iframe>

task_definition_json:-

<iframe src="https://medium.com/media/ccf643517174e0e44a2ac2ba913a2fa8" frameborder=0></iframe>

### ECS-services.tf :-

<iframe src="https://medium.com/media/29d4a66d45438b3a4b0b1f4503a16aee" frameborder=0></iframe>

### ECS-ALB.tf:-

<iframe src="https://medium.com/media/baccde9de25a001bdf2be51ef52bfac7" frameborder=0></iframe>

### ECS-cloudwatch.tf:-

<iframe src="https://medium.com/media/f860589c972f233311d5a043a879ca0b" frameborder=0></iframe>

### **ECS-Route53.tf:-**

<iframe src="https://medium.com/media/4a7aec6f0f5c02c8c22610461d522921" frameborder=0></iframe>

### Running the Terraform script

Go to the project folder and type “terraform plan” , this command will show you what you will be creating in the AWS.

Then you can validate the terraform code with “terraform validate”

Finally, deploy the resource with “terraform apply”

And thats it! After our resources are provisioned, we can visit our EC2 Dashboard, find our Load Balancer URL and visit the site running on our newly deployed ECS cluster.

![](https://cdn-images-1.medium.com/max/3084/1*77sRKx8hLDAi7IzRmv67vQ.png)
