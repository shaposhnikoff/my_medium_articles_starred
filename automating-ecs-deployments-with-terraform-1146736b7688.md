
# Automating ECS Deployments with Terraform



We use AWS’s Elastic Container Service for the majority of our application deployments at Mesh. Whether we are standing up a cluster for one of our consulting customers, or for one of our in-house products, we interact with the service on a near-daily basis.

As the AWS marketing page describes, ECS is:
> A highly scalable, high-performance container management service that supports Docker containers and allows you to easily run applications on a managed cluster of Amazon EC2 instances.

The operative word here being ***easy***. While we love the reliability, and peace of mind that ECS brings, standing up an ECS cluster and deploying a Dockerized application via the AWS dashboard is far from easy. 
As is often the case with AWS, the dashboard can leave developers scratching their heads and wondering why things need to be so difficult.

## Tooling to the rescue

We are always on the lookout for opportunities to build tooling that will allow us to automate difficult, repetitive, and/or error prone tasks. Tools have the potential to save developers significant amounts of time and increase efficiency. We get to pass those savings on to our customers as well.

Deploying an ECS Cluster is indeed a difficult, repetitive, and error prone process. As such, the process was a perfect candidate for automation via tooling. To do so, we leveraged one of our favorite DevOps frameworks, [Terraform](http://www.terraform.com).

## Configuration with Terraform

Terraform allows you to describe your infrastructure via configuration files. Once you have your files in place, the Terraform CLI allows you to spin up cloud resources from the command line. As a bonus, it also helps us to avoid the AWS dashboard ;).

We typically configure our ECS clusters in a consistent manner. This makes it very easy for us to express our default configuration with Terraform. Our typical configuration is as follows:

* **Cluster Instances — **Min of 2, Max of 5, Desired of 3

* **Load Balancing — **Via an Application Load Balancer

* **Security — **Ingress on 80 (HTTP), 443 (HTTPS) and 22 (SSH), All Egress.

Our default Terraform configuration files for provisioning an ECS instance can be found in the following repository:
[**meshhq/terraform-meshhq-ecs-cluster**
*Contribute to terraform-meshhq-ecs-cluster development by creating an account on GitHub.*github.com](https://github.com/meshhq/terraform-ecs-cluster)

## **Single Command Deployments**

When we set out to build our ECS tooling, we had a simple goal in mind. We wanted to to be able to deploy a docker image on a newly provisioned ECS Cluster with a single command. We built this functionality into our in-house **Mesh CLI **via the following command:

<iframe src="https://medium.com/media/266693376628aeadc3b20502848a1a23" frameborder=0></iframe>

While this command looks simple to the consuming developer, there is a significant amount of infrastructure being provisioned underneath the hood. Let’s break it down.

## IAM Roles

Before deploying an ECS Cluster, we need to create two different IAM roles, and then attach specific policies to each of these roles. The roles are the following:

* [ECS Container Instance IAM Role](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/instance_IAM_role.html)

* [ECS Service Scheduler IAM Role](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/service_IAM_role.html)

The Mesh CLI generates the IAM resources via the following Terraform config files:

### [ecs-instance-role.tf](https://github.com/meshhq/terraform-ecs-cluster/blob/master/iam/ecs-instance-role.tf)

<iframe src="https://medium.com/media/562e00c61f929040d189ec4f35db6cae" frameborder=0></iframe>

### [ecs-service-role.tf](https://github.com/meshhq/terraform-ecs-cluster/blob/master/iam/ecs-service-role.tf)

<iframe src="https://medium.com/media/ad880156f5f59f2e93a912aa808f645f" frameborder=0></iframe>

### ecs-instance-profile.tf

<iframe src="https://medium.com/media/1b931b423471e28791e226be2a6b1700" frameborder=0></iframe>

## Virtual Private Cloud

Next we need to provision a number of[ Virtual Private Cloud (VPC)](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Introduction.html) resources. Our ECS cluster will eventually be deployed into the VPC.

This involves the following:

* [VPC](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/getting-started-ipv4.html) — The VPC into which our cluster will be deployed.

* [Subnets](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Subnets.html) — The subnets for our VPC. As ECS cluster requires two subnets.

* [Security Groups](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_SecurityGroups.html) — A virtual firewall for our EC2 instances.

* [Route Table](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Route_Tables.html) — Determines where VPC traffic should be routed.

* [Network ACL](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_ACLs.html) — A virtual firewall for our VPC.

* [Internet Gateway](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Internet_Gateway.html) — The gateway that exposes our Cluster to the internet.

The Mesh CLI generates the VPC resources via the following Terraform config files:

### vpc.tf

<iframe src="https://medium.com/media/a802fc8447a75fbf7369b569e61d2b52" frameborder=0></iframe>

### subnet.tf

<iframe src="https://medium.com/media/7d24ea005baef099180402d4d2ddcfa1" frameborder=0></iframe>

### security-group.tf

<iframe src="https://medium.com/media/55756ddaa35e2ee09b008e1f2eb6acd5" frameborder=0></iframe>

### route-table.tf

<iframe src="https://medium.com/media/45db4450491bdea6e298c4f50dc859a8" frameborder=0></iframe>

### network-acl.tf

<iframe src="https://medium.com/media/ab5fe3e385c59a21dd36c97e363634ca" frameborder=0></iframe>

### internet-gateway.tf

<iframe src="https://medium.com/media/f3f90d2407a36c045f9fab1c3fb32a96" frameborder=0></iframe>

## Elastic Compute Instances

Once we have our VPC setup, we can deploy some actual EC2 instances. We do this by configuring an Auto Scaling Group with an Elastic Load Balancer.

This involves the following:

* [Auto Scaling Group](http://docs.aws.amazon.com/autoscaling/latest/userguide/GettingStartedTutorial.html) — Responsible for launching the appropriate number of instances into our cluster.

* [Application Load Balancer](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) —Distributes incoming traffic across our instances.

* [Launch Configuration](http://docs.aws.amazon.com/autoscaling/latest/userguide/LaunchConfiguration.html) — Describes the type of instances that should be launched.
> **Note our Launch Configuration’s User Data. This bash script tells each instance that is launched by our Autoscaling group that it needs to be launched into our ECS Cluster, based on our cluster name.**

The Mesh CLI generates the EC2 resources via the following Terraform config files:

### **autoscaling-group.tf**

<iframe src="https://medium.com/media/b4a5cb7a1ce267ef14d9abb97d8a400c" frameborder=0></iframe>

### application-load-balancer.tf

<iframe src="https://medium.com/media/b69b09e00b1358d7b72e182927bc1812" frameborder=0></iframe>

### launch-configuration.tf

<iframe src="https://medium.com/media/e31fabb0d0dc1fb11bfb8088a5229f53" frameborder=0></iframe>

## Elastic Container Service

Lastly, it is time to deploy our actual ECS resources. This involves the following:

* [Cluster](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_clusters.html) — Our actual ECS Cluster. This is nothing more than a logical grouping of EC2 instances.

* [Task Definition](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html)- The mechanism by which our docker images are actually place on a running EC2 instance.

* [Service](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_services.html)s — Run and maintain a specified number of tasks on each instance.

The Mesh CLI generates the ECS resources via the following Terraform config files:

### cluster.tf

<iframe src="https://medium.com/media/856f473fb3e1359c3bb94c445f04a862" frameborder=0></iframe>

### service.tf

<iframe src="https://medium.com/media/8484ecf2a0d3c7e027cdc79fa78e5a48" frameborder=0></iframe>

### task-definition.tf

<iframe src="https://medium.com/media/dce635e85aa83fb515cff853bd6b37f1" frameborder=0></iframe>

## **Conclusion**

And thats it! After our resources are provisioned, we can visit our EC2 Dashboard, find our Load Balancer URL and visit the site running on our newly deployed ECS cluster.

Obviously this command does not take into account updates to resources after they are provisioned…but that will be a subject of another post.

In closing,** Terrafom** is a powerful tool that can be used to save development teams a ton of time, energy, and stress. We use it liberally when deploying AWS resources, and would encourage you to do the same!

## Who wrote this?

[Kevin](https://github.com/kcoleman731) [Coleman](https://twitter.com/KevFColeman) is a founder and partner at [Mesh Studio](https://meshstudio.io). [Mesh](https://meshstudio.io) is a software consultancy and product studio based in Seattle, WA. We design beautiful software for our clients, and launch our own in-house products. If you’d like to get in touch, [get in touch!](mailto:info@meshstudio.io) We’re [hiring](https://mesh.workable.com/) too!
