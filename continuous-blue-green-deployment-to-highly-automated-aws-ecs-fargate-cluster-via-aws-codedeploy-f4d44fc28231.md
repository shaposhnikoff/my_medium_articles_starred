
# Continuous Blue-Green Deployment to Highly Automated AWS ECS Fargate Cluster via AWS CodeDeploy…

CI/CD pipeline to AWS ECS Fargate via Codedeploy

After a long time, I got a chance to write a post about some nice things which I worked on for a while. Devops is come into prominence day by day and many development teams want to adapt it in their daily work. With the adapt of devops culture, new automation tools are taking over old legacy structures. By the way, teams who want to adapt devops and deploy applications frequently to their production systems are looking for new ways to make it easy, portable, avoiding waste of time, and with more structural way(infrastructure as code, IaC).

In this post, I will show how to create a pipeline which builds source code, packages into docker image, and deploy to AWS ECS Fargate via terraform in a blue-green strategy. If you need this kind of continuous deployment in your team, and if your environment is in Amazon web services, you can easily implement this kind of pipeline which automates your deployment in every commit.

Before starting to implement a pipeline, I will briefly explain some frequently used terms which consist of this post.

**IAC(Infrastructure as code):**

Infrastructure as code, also referred to as IaC, is a type of IT setup wherein developers or operations teams automatically manage and provision the technology stack for an application through software, rather than using a manual process to configure discrete hardware devices and operating systems. Infrastructure as code is sometimes referred to as programmable or software-defined infrastructure. I personally use “[**Terraform](https://www.terraform.io/)**” to create infrastructure of environments in cloud providers.

Ok, but **why Terraform**? Terraform provides adapters which integrates with cloud provider’s(aws,azure,google cloud) api for you. So, with little code block, you can launch an **aws ec2 instance**, or create **azure kubernetes cluster**. Meanwhile, in addition to this kind of adapters, terraform can provide different adapters for different tools as well, like creating automated dashboard in grafana, or create alert in pagerduty etc. Other advantage of Terraform, you can execute “plan” command to show changes to be made to infrastructure even before making the actual changes.

Sample Terraform IAC:

    provider "aws" {
      region **=** "us-west-2"
    }

    # Gets the most current Instance AMI from AWS
    data "aws_ami" "ubuntu" {
      most_recent **=** **true**
    
      filter {
        name   **=** "name"
        values **=** ["ubuntu/images/hvm-ssd/ubuntu-trusty-14.04-amd64-server-*"]
      }
    
      filter {
        name   **=** "virtualization-type"
        values **=** ["hvm"]
      }
    
      owners **=** ["099720109477"] # Canonical
    }

    # Creates AWS EC2 instance in a "t2.micro" type with ubuntu AMI
    resource "aws_instance" "web" {
      ami           **=** "${data.aws_ami.ubuntu.id}"
      instance_type **=** "t2.micro"
    
      tags **=** {
        Name **=** "HelloWorld"
      }
    }

**Gitlab CI/CD**

Gitlab was providing you code repository at the past. But, then, they started to invest in devops processes by implementing ci-cd pipelines, operating kubernetes clusters, managing environments, creating issues and tracking them etc.

By the way, Gitlab CI/CD saves your time by avoiding creation of ci server. It automatically launches docker containers in its runner to execute your tasks(i.e building code, executing terraform command).So, it saves your money, because you don’t need to create any AWS EC2 instances to host ci server. Gitlab CI/CD is much more appropriate to devops process.

Here is the sample CI/CD pipeline:

![[https://ohdear.app/img/blogs/gitlab-ci-for-laravel/ohdear_pipelines_all_green.png](https://ohdear.app/img/blogs/gitlab-ci-for-laravel/ohdear_pipelines_all_green.png)](https://cdn-images-1.medium.com/max/2870/0*4-9AdZb3__osIja9.png)*[https://ohdear.app/img/blogs/gitlab-ci-for-laravel/ohdear_pipelines_all_green.png](https://ohdear.app/img/blogs/gitlab-ci-for-laravel/ohdear_pipelines_all_green.png)*

**Blue/Green Deployment**

Blue-green deployments are a pattern whereby we reduce downtime during production deployments by having two production environments (“blue” and “green”). In our blue-green deployment scenario, we will have identical task group for new version of the applications(green) in addition to current version of the apps(blue). With the help of AWS Codedeploy, we will manage traffic switching to green or rollback to blue ones.

**AWS Architecture**

Let’s come into setup of AWS architecture for our deployment scenario. We will have 1 Application load load balancer, 1 green target group(for green fargate tasks), 1 blue target group(for blue fargate tasks), dns record in Route53, 2 target groups which are blue/green, 1 ecs fargate cluster. You can see high level topology as below:

![](https://cdn-images-1.medium.com/max/2044/1*M_2kivhtz2-LU-gn49jb6A.png)

Now, lets deep dive into implementation of continuous b/g deployment to fargate cluster.

**Prerequisites:**

1. You have to create highly available VPC, with private & public subnets in different azs. You can check and create vpc topology according to my previous post: [creation of vpc](https://medium.com/@ahmetatalay/building-highly-available-scalable-and-reliable-ecs-clusters-part-1-creating-vpc-1321640225cc)

1. Put the following tag into your VPC:
 **Key **= **BlueGreenDemo**

![](https://cdn-images-1.medium.com/max/5040/1*m64wPta7D4VaaVLqwRRqng.png)

3. Put the following tag into private subnets:
 **Type **= **Private**

![](https://cdn-images-1.medium.com/max/5028/1*1zoEVM4C-56pS6sIzRqBFQ.png)

4. Put the following tag into public subnets:
 **Type **= **Public**

![](https://cdn-images-1.medium.com/max/5064/1*3ZH8mJtFpSu4u19_L_cA4Q.png)

5. Buy domain from AWS Route53 and create Route 53 hosted zone with domain name(i.e. mycompany.com) or just only add nameservers of created hosted zone into your different hosting company

![](https://cdn-images-1.medium.com/max/4532/1*SVX90NWkov9e_2eOG_XCWQ.png)

6. Create IAM User with access key & secret keys. These will be used in Gitlab CI env vars as described below.

**Preparation**

Before preparing ci-cd pipeline which deploys continuously to AWS ECS, you have to do following steps:

1. Clone [sample project](https://github.com/onedaywillcome1/BlueGreenDemo) from my github repo and push into your **Gitlab **account.

1. Set aws iam user keys & domain_name into the environment variables under project settings

![](https://cdn-images-1.medium.com/max/5120/1*0_76EnQwHOaVYFvnvgPoQQ.png)

**Playing with pipeline**

After completing prerequisites & preparation steps, you are now ready to use gitlab ci-cd pipeline. Whenever you commit to your project, you will see that pipeline will automatically start, build your code, package into docker image and will push to aws ecr/ecs fargate.

Here is the gitlab-ci.yml file which provides a pipeline in Gitlab. It calls “execute.sh” script with different parameters in each stage:

    image: onedaywillcome/roro:v1.0
    
    variables:
      DOCKER_DRIVER: overlay
      DOCKER_HOST: tcp://docker:2375
    
    stages:
      - build
      - push-ecr
      - deploy
      - destroy
    
    build-code:
      stage: build
      script: ./gradlew clean build
      artifacts:
        paths:
          - build
    
    push-ecr:
      stage: push-ecr
      script: chmod 775 ./deploy/execute.sh && ./deploy/execute.sh dockerize
      services:
        - docker:dind
    
    deploy:
      stage: deploy
      script: chmod 775 ./deploy/execute.sh && ./deploy/execute.sh deploy
      services:
        - docker:dind
    
    destroy:
      stage: destroy
      when: manual
      script: chmod 775 ./deploy/execute.sh && ./deploy/execute.sh destroy

Here is the pipeline which happens to deploy “hello world” spring microservice into fargate.

![](https://cdn-images-1.medium.com/max/2200/1*zCLEbFTdk0Xg2pWnpmWBlw.gif)

After “deploy” stage in a pipeline, let’s switch to Codedeploy cockpit area. We will manage blue-green deployment stages from here.

You can do following things in codedeploy:
1. Reroute traffic(switch traffic to green tasks in ECS fargate)
2. Terminate original task set(kills the blue tasks)
3. Rollback stages( before switch or after switch traffic, you can do rollback)

![](https://cdn-images-1.medium.com/max/2200/1*e4TkzkJIfo4YYEmTglIVew.gif)

You can check deploy/terraform folder in a sample project and you will see some terraform codes. These codes do the following things:
1. Create alb in a public subnet
2. Create ECS Cluster
3. Create Fargate task definition, fargate service
4. Create codedeploy app, deployment group
5. Invoke codedeploy pipeline
6.Create task scaling( according to sample cpu thresholds, it will scale in or out in a specific boundaries)

![](https://cdn-images-1.medium.com/max/2236/1*Sp2IAeiLOazs-0ZKc7836A.png)

I will end up here. After you did steps explained above, you can play with your ci-cd pipeline. If you have any questions, you can ask me at below. If you like this post, you can clap:)
