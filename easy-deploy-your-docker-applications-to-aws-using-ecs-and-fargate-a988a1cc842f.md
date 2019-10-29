Unknown markup type 10 { type: [33m10[39m, start: [33m71[39m, end: [33m81[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m85[39m, end: [33m92[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m123[39m, end: [33m135[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m6[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m67[39m, end: [33m88[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m7[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m6[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m5[39m, end: [33m8[39m }

# Easy deploy your Docker applications to AWS using ECS and Fargate



In this post, I will try to demonstrate how you can deploy your Docker application into AWS using ECS and Fargate.

As an example, I will deploy [this app](http://openjobs.me/) to ECS. The source can be found [here](https://github.com/opensanca/opensanca_jobs/).

I will use [Terraform](https://www.terraform.io/) to spin the infrastructure so I can easily track everything that I create as a code. If you want to learn the basics of Terraform, please read my [post about it](https://thecode.pub/creating-your-cloud-servers-with-terraform-bfa01a499bad).

## ECS

What is ECS?

The Elastic Container Service (ECS) is an AWS Service that handles the Docker containers orchestration in your EC2 cluster. It is an alternative for Kubernetes, Docker Swarm, and others.

### ECS Terminology

To start understanding what ECS is, we need to understand its terms and definitions that differs from the Docker world.

* **Cluster: **It is a group of EC2 instances hosting containers.

* **Task definition: **It is the specification of how ECS should run your app. Here you define which image to use, port mapping, memory, environments variables, etc.

* **Service: **Services launches and maintains tasks running inside the cluster. A Service will auto-recover any stopped tasks keeping the number of tasks running as you specified.

## Fargate

Fargate is a technology that allows running containers in ECS without needing to manage the EC2 servers for cluster. You only deploy your Docker applications and set the scaling rules for it. Fargate is an execution method from ECS.

***Show me the code***

The full example is on [Github](https://github.com/duduribeiro/terraform_ecs_fargate_example).

## The project structure

Our Terraform project is composed of the following structure:

├── modules
│ └── code_pipeline 
│ └── ecs 
│ └── networking 
│ └── rds
├── pipeline.tf
├── production.tf
├── production_key.pub
├── terraform.tfvars
└── variables.tf

* **Modules** are where we will store the code that handles the creation of a group of resources. It can be reused by all environments (Production, Staging, QA, etc.) without needing to duplicate a lot of code.

* **production.tf **is the file that defines the environment itself. It calls the modules passing variables to it.

* **pipeline.tf **Since the pipeline can be a global resource without needing to isolate per environment. This file will handle the creation of this pipeline using the `code_pipeline` module.

## First part, the networking

The branch with this part can be found [here](https://github.com/duduribeiro/terraform_ecs_fargate_example/tree/01_networking).

The first thing that we need to create is the VPC with 2 subnets (1 public and 1 private) in each Availability Zone. Each Availability Zone is a geographically isolated region. Keeping our resources in more than one zone is the first thing to achieve high availability. If one physical zone fails for some reason, your application can answer from the others.

![Our networking](https://cdn-images-1.medium.com/max/3584/1*Cvu1YNJdfezuVfU8kAPgNA.png)*Our networking*

Keeping the cluster on the private subnet protects your infrastructure from external access. The private subnet is allowed only to be accessed from resources inside the public network (In our case, will be the Load Balancer only).

This is the code to create this structure (it is practically the same from my introduction post of Terraform):

<iframe src="https://medium.com/media/3000fa3a6cfa0ccdb9678d3e4660424d" frameborder=0></iframe>

The above code creates the VPC, 4 subnets (2 public and 2 private) in each Availability zone. It also creates a NAT to allow the private network access the internet.

**The Database**

The branch with this part can be found [here](https://github.com/duduribeiro/terraform_ecs_fargate_example/tree/02_database).

We will create a RDS database. It will be located on the private subnet. Allowing only the public subnet to access it.

<iframe src="https://medium.com/media/59d7deafef35b800131d6d8061ad1e82" frameborder=0></iframe>

With this code, we create the RDS resource with values received from the variables. We also create the security group that should be used by resources that want to connect to the database (in our case, the ECS cluster).

Ok. Now we have the database. Let’s finally create our ECS to deploy our app \o/.

## Take Three: The ECS

The branch with this part can be found [here](https://github.com/duduribeiro/terraform_ecs_fargate_example/tree/03_ecs).

We are approaching the final steps. Now, it is the part that we define the ECS resources needed for our app.

### The ECR repository

The first thing is to create the repository to store our built images.

<iframe src="https://medium.com/media/356acc348a4487a6434c6e60919f7c62" frameborder=0></iframe>

### The ECS cluster

Next, we need our ECS cluster. Even using Fargate (that doesn’t need any EC2), we need to define a cluster for the application.

<iframe src="https://medium.com/media/007965a6291dc6a49086c14471cdd92f" frameborder=0></iframe>

### The tasks definitions

Now, we will define 2 task definitions.

* Web: Contains the definition of the web app itself.

* Db Migrate: This task will only run the command to migrate our database and will die. Since it is a single run task, we don’t need a service for it.

<iframe src="https://medium.com/media/cbba359b380baf0de6398ebbf29d66c0" frameborder=0></iframe>

The tasks definitions are configured in a JSON file and rendered as a template in Terraform. 
This is the task definition of the web app:

<iframe src="https://medium.com/media/cd914e0e8af12d8d338da2acd81e85d2" frameborder=0></iframe>

In the file above, we are defining the task to ECS. We pass the created ECR image repository as variable to it. We also configure other variables so ECS can start our Rails app. 
The definition of the DB migration task is almost the same. We only change the command that will be executed.

### The load balancers

Before creating the Services, we need to create the load balancers. They will be on the public subnet and will forward the requests to the ECS service.

<iframe src="https://medium.com/media/3033baa23cf2903207443201d7762619" frameborder=0></iframe>

In the file above we define that our target group will use HTTP on port 80. We also create a security group to allow access into the port 80 from the internet. After, we create the Application Load Balancer and the listener. To use Fargate, you should use an Application Load Balancer instead an Elastic Load Balancer.

### Finally, the ECS service

Now we will create the service. To use Fargate, we need to specify the lauch_type as Fargate.

<iframe src="https://medium.com/media/9a7598bcb9d94c58ca13ca1be88a7d69" frameborder=0></iframe>

### Auto-scaling

Fargate allows us to auto-scale our app easily. We only need to create the metrics in CloudWatch and trigger to scale it up or down.

<iframe src="https://medium.com/media/0163c1beacb8ade34e9f09209dd80916" frameborder=0></iframe>

We create 2 auto scaling policies. One to scale up and other to scale down the desired count of running tasks from our ECS service.

After, we create a CloudWatch metric based on the CPU. If the CPU usage is greater than 85% from 2 periods, we trigger the alarm_action that calls the scale-up policy. If it returns to the Ok state, it will trigger the scale-down policy.

## The Pipeline to deploy our app

Our infrastructure to run our Docker app is ready. But it is still boring to deploy it to ECS. We need to manually push our image to the repository and update the task definition with the new image and update the new task definition. We can run it through Terraform, but it could be better if we have a way to push our code to Github in the master branch and it deploys automatically for us.

*Entering, [CodePipeline](https://aws.amazon.com/codepipeline) and [CodeBuild](https://aws.amazon.com/codebuild/).*

CodePipeline is a Continuous Integration and Continuous Delivery service hosted by AWS.

CodeBuild is a managed build service that can execute tests and generate packages for us (in our case, a Docker image).

With it, we can create pipelines to delivery our code to ECS. The flow will be:

* You push the code to master’s branch

* CodePipeline gets the code in the Source stage and calls the Build stage (CodeBuild).

* Build stage process our Dockerfile building and pushing the Image to ECR and triggers the Deploy stage

* Deploy stage updates our ECS with the new image

Let’s define our Pipeline with Terraform:

<iframe src="https://medium.com/media/68275343cb24ec50baf32ce6cc1002e3" frameborder=0></iframe>

In the above code, we create a CodeBuild project, using the following buildspec (build specifications file):

<iframe src="https://medium.com/media/40583c74d4308338360de35bb99a1921" frameborder=0></iframe>

We defined some phases in the above file.

* pre_build: Upgrade aws-cli, set some environment variables: REPOSITORY_URL with the ECR repository and IMAGE_TAG with the CodeBuild source version. The ECR repository is passed as a variable by Terraform.

* build: Build the Dockerfile from the repository tagging it as LATEST in the repository URL.

* post_build: Push the image to the repository. Creates a file named imagedefinitions.json with the following content: 
‘[{“name”:”web”,”imageUri”:REPOSITORY_URL”}]’
This file is used by CodePipeline to upgrade your ECS cluster in the Deployment stage.

* artifacts: Get the file created in the last phase and uses as the artifact.

After, we create a CodePipeline resource with 3 stages:

* Source: Gets the repository from Github (change it by your repository information) and pass it to the next stage.

* Build: Calls the CodeBuild project that we created in the step before.

* Production: Gets the artifact from Build stage (imagedefinitions.json) and deploy to ECS.

Let’s see they working together?

## Running all together

The code with the full example is [here](https://github.com/duduribeiro/terraform_ecs_fargate_example).

Clone it. Also, since we use Github as the CodePipeline source provider, you need to generate a token to access the repositories. [Read here]([https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)) to generate yours.

After generating your token, export it as an environment variable.

    $ export GITHUB_TOKEN=YOUR_TOKEN

Now, we need to import the modules and the provider library.

    $ terraform init

![](https://cdn-images-1.medium.com/max/2208/1*6HQQkkurojqHSIw3ywQTSw.png)

Now, let the magic begin!

    $ terraform apply

it will display that Terraform will create some resources, and if you want to continue

![](https://cdn-images-1.medium.com/max/2000/1*N_Ce_3nRV-hk4QFgd-J3sw.png)

Type yes.

Terraform will start create our infraestructure.

![](https://cdn-images-1.medium.com/max/2000/1*lZ7NXzq0NMQmObgMoyoJIg.gif)

Seriously, get a coffee until it finishes.

![](https://cdn-images-1.medium.com/max/2210/1*hl4G1GBfzMSBuBJ2axn0LQ.png)

![](https://cdn-images-1.medium.com/max/2284/1*-w-oLVLxuyebsaUStamQHg.png)

AWESOME!. Our infrastructure is ready!!. If you enter in your CodePipeline at AWS Dashboard, you can see that it also triggered the first build:

![](https://cdn-images-1.medium.com/max/2000/1*XfmkQU8ae8v9Pbp6iaJQIA.png)

Wait until all the Stages are green.

![](https://cdn-images-1.medium.com/max/2000/1*ZdNh3pr5qIdU-vMwlQfi4Q.png)

Get your Load Balancer DNS and check the deployed application:

    $ terraform output alb_dns_name

![](https://cdn-images-1.medium.com/max/2252/1*WW07XGYH37It1xq5tUnGrQ.png)

![It is working \o/](https://cdn-images-1.medium.com/max/5752/1*wJmKvO2Pd36jtJMZePd-3Q.png)*It is working \o/*

Finally, the app is running. Almost magic!

![](https://cdn-images-1.medium.com/max/2000/1*mPUc2fU1VPbW6gjbw1DjeQ.gif)

If you have any questions about it, contact me. This was just an introduction post about ECS with Fargate using Terraform.

![](https://cdn-images-1.medium.com/max/2560/1*dsHpznpcd482MHT1fvyc3Q.png)

Cheers 🍻
