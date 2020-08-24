Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m32[39m }

# How to deploy a Docker Container (Python) on Amazon ECS using Amazon ECR

Developing applications, models, and software from scratch is another thing, but what good does it make if you can’t deploy it for the end-users to access? Exactly!

In this article, I’ll guide you on how to deploy a docker container on ECS using ECR. (Don’t worry you’ll be more familiarize with these short terms in few minutes)

![This is my simple drawing of what I’m about to do.](https://cdn-images-1.medium.com/max/2798/1*zWF66mV-bUbvBKvvZxVIiw.png)*This is my simple drawing of what I’m about to do.*

### Overview of Amazon ECS and Amazon ECR

[Amazon ECS](https://aws.amazon.com/ecs/) is a highly scalable, fast container management service that makes it easy to run and manage **Docker containers on a cluster of Amazon EC2 instances **and eliminates the need to operate your own cluster management or worry about scaling management infrastructure.

In order to reliably store Docker images on AWS, **ECR provides a managed Docker registry service that is secure, scalable, and reliable**. ECR is a **private Docker repository** with resource-based permissions using [IAM](https://aws.amazon.com/iam/) so that users or EC2 instances can access repositories and images through the Docker CLI to push, pull, and manage images.

If you want to read more about ECS and ECR, [Click here.](https://aws.amazon.com/blogs/compute/authenticating-amazon-ecr-repositories-for-docker-cli-with-credential-helper/)

This article is consists of 4 major parts.

1. **Creating a Repository on ECR (To push our docker image to ECR Cloud)**

1. **Dockerize the Python Application *(Basic containerization but carefully read and understand each step of it)***

1. **Push it on ECR (Elastic Cloud Registry) Container**

1. **Deploy that container on an ECS (Elastic Container Service) node**
> Before moving on with my article, I assume that you have a basic knowledge about AWS and hope you guys have settled inn for a account so you can create a new tab and follow me on with each step.

### 1. Create a Repository on ECR (Elastic Container Registry)

**What is Amazon ECR?**

Amazon Elastic Container Registry (ECR) is a fully-managed [Docker](https://aws.amazon.com/docker/) container registry that makes it easy for developers to store, manage, and deploy Docker container images. Amazon ECR is integrated with [Amazon Elastic Container Service (ECS)](https://aws.amazon.com/ecs/), simplifying your development to production workflow. Amazon ECR eliminates the need to operate your own container repositories or worry about scaling the underlying infrastructure. Amazon ECR hosts your images in a highly available and scalable architecture, allowing you to reliably deploy containers for your applications. Integration with AWS Identity and Access Management (IAM) provides resource-level control of each repository. With Amazon ECR, there are no upfront fees or commitments. You pay only for the amount of data you store in your repositories and data transferred to the Internet.

**How do I create a Repository in ECR?**

1. **Install the AWS CLI** — You can use the AWS command line tools to issue commands at your system’s command line to perform Amazon ECS and AWS tasks. This can be faster and more convenient than using the console. The command line tools are also useful for building scripts that perform AWS tasks.

To use the AWS CLI with Amazon ECR, install the latest AWS CLI version (Amazon ECR functionality is available in the AWS CLI starting with version 1.9.15). You can check your AWS CLI version with the **aws — version** command.

![](https://cdn-images-1.medium.com/max/2000/1*6bAcw4NWKHjYys_js4oX-g.png)

For information about installing the AWS CLI or upgrading it to the latest version, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the *AWS Command Line Interface User Guide*.

**2. Open Powershell/Terminal**

Type the following command to create a new repository called python-app-aws.

    aws ecr create-repository --repository-name python-app-aws

Go to AWS Console and search for ECR in services.

![](https://cdn-images-1.medium.com/max/2828/1*sD2uEErMaj403rY4xFdA1w.png)

And click on Repositories so you can see your repository has been created.

![](https://cdn-images-1.medium.com/max/3838/1*ArLNPJfHMbzjJizg0HOgDg.png)

### 2. Dockerize the Python Application

Go to the following link to clone/download this python app that I’m going to dockerize. (Please ignore the DP :) )
[**DilanKalpa/PythonDockerAWS**
*You can’t perform that action at this time. You signed in with another tab or window. You signed out in another tab or…*github.com](https://github.com/DilanKalpa/PythonDockerAWS)

As per my repository, You are going to need 3 files.

1. *app.py*

1. *Dockerfile*

1. *requirements.txt*

Once you have finished setting up the above files, Let’s try to build this folder in docker as the very first step for containerization.

Read more about Docker and how it’s been used in [this awesome tutorial](https://towardsdatascience.com/how-docker-can-help-you-become-a-more-effective-data-scientist-7fc048ef91d5).

### 3. Push it on ECR (Elastic Cloud Registry) Container

**** Ensure you have installed the latest version of the AWS CLI and Docker. For more information, see the [**ECR documentation](http://docs.aws.amazon.com/AmazonECR/latest/userguide/ECR_GetStarted.html)**.**

* Open Powershell in Windows/ Terminal in Mac to do the **ABTP**!

* **A- Authenticate**

* **B- Build**

* **T- Tag**

* **P- Push**

1. Retrieve the login command to use to authenticate your Docker client to your registry. Use AWS Tools for PowerShell:

    Invoke-Expression -Command (Get-ECRLoginCommand -Region ap-southeast-2).Command

2. **Build **your Docker image using the following command. For information on building a Docker file from scratch see the instructions [here](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/docker-basics.html). You can skip this step if your image is already built:

docker build -t python-app-aws .

3. After the build completes, **tag **your image so you can push the image to this repository: *Make sure to replace the name ID from your unique URL for the repository you have created*

    docker tag python-app-aws:latest **ID**.dkr.ecr.ap-southeast-2.amazonaws.com/python-app-aws:latest

4. Run the following command to **push **this image to your newly created AWS repository:

    docker push **ID**.dkr.ecr.ap-southeast-2.amazonaws.com/python-app-aws:latest

The word after colon (:) is the tag part so you can identify each version of pushes. For example; I’ve pushed two images called **:latest** and **:v2**

![Image tag has two attributes since I’ve pushed the image two times now.](https://cdn-images-1.medium.com/max/3214/1*Sy-R1Wlbn7V3xn3locPQYg.png)*Image tag has two attributes since I’ve pushed the image two times now.*

### 4. Deploy that container on an ECS (Elastic Container Service) node

**I. Create a Task Definition.**

Before you can run Docker containers on Amazon ECS, you must create a task definition. You can define multiple containers and data volumes in a task definition. For more information about the parameters available in a task definition, see [Task Definition Parameters](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html).

**To create a new task definition**

1. Open the Amazon ECS console at [https://console.aws.amazon.com/ecs/](https://console.aws.amazon.com/ecs/).

1. In the navigation pane, choose **Task Definitions**, **Create new Task Definition**.

1. On the **Select compatibilities** page, select the launch type that your task should use and choose the **Next step**.

1. Select [**Fargate ](https://aws.amazon.com/fargate/)**launch type.

* *The [Fargate ](https://aws.amazon.com/fargate/)launch type is not compatible with Windows containers.*

5. Follow [these steps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-task-definition.html) under the Fargate tab.

* **Important**: In the **containerDefinitions **section of your task definition, specify the ECR image **aws_account_id.dkr.ecr.region.amazonaws.com/repository:tag as the image property. (Click on below link for further information)**
[**Allow Amazon ECS Tasks to Pull Images from an Amazon ECR Image Repository**
*How do I allow Amazon Elastic Container Service (Amazon ECS) tasks to pull images from an Amazon Elastic Container…*aws.amazon.com](https://aws.amazon.com/premiumsupport/knowledge-center/ecs-tasks-pull-images-ecr-repository/?fbclid=IwAR2KmQe5Ce5i5VhqT6y1oC0IWUppgX7XMhqtwuEJSntUTLlRXfhzHEyZMx4)

After you created the Task Definition, it should look like this on the dashboard. (I’ll highlight the important fields in Yellow color in case you missed those essential efforts in the documentation links I’ve posted above; You can double-check and guarantee that you are on the correct path as I am.)

![Make sure to add the IAM role ecsTaskExecutionRole as Task Execution Role](https://cdn-images-1.medium.com/max/2732/1*6knro2jFF2Jx681Rr-dWvg.png)*Make sure to add the IAM role ecsTaskExecutionRole as Task Execution Role*

![Ensure that the container is attached to your task def. (Under image name; ECR URL should be displayed)](https://cdn-images-1.medium.com/max/3248/1*8VTjHJAy0aQ73JONhhTBNw.png)*Ensure that the container is attached to your task def. (Under image name; ECR URL should be displayed)*

**II. Create a Service to run the above task definition.**

![](https://cdn-images-1.medium.com/max/2000/1*YoWSL7i-TuQ4vR8IJibENg.png)

All services require some basic configuration parameters that define the service, such as the task definition to use [Which we want exactly], which cluster the service should run on, how many tasks should be placed for the service, and so on. This is called the service definition. For more information about the parameters defined in a service definition, see [Service Definition Parameters](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service_definition_parameters.html).

This procedure covers creating a service with the basic service definition parameters that are required. After you have configured these parameters, you can create your service or move on to the procedures for optional service definition configuration,** such as configuring your service to use a load balancer.**

[Click on this link and follow the steps to create a service for Fargate Launch type](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/basic-service-params.html). (Create 3 replica task)

After you created the Service, it should look like this on the dashboard. (I’ll highlight the important fields in Yellow color in case you missed those essential efforts in the documentation links I’ve posted above; You can double-check and guarantee that you are on the correct path as I am.)

![Target Group name is Highlighted](https://cdn-images-1.medium.com/max/2286/1*2-BwkIOQoq5k-z8VZg68og.png)*Target Group name is Highlighted*

![Note and understand the Service Type, Launch Type, Service Role.](https://cdn-images-1.medium.com/max/3340/1*tmnHuJdGWeOePwZMv1LAhw.png)*Note and understand the Service Type, Launch Type, Service Role.*

Select Target Group which leads you to here.

![Click on Load Balancers to checkout our load balancer and select the Load Balancer](https://cdn-images-1.medium.com/max/3836/1*1pQAywnl88JFb92Q1Wjpiw.png)*Click on Load Balancers to checkout our load balancer and select the Load Balancer*

![](https://cdn-images-1.medium.com/max/3826/1*cQJ5PCtD-rNyxuJmhXZRUA.png)

Copy the DNS name and paste it on a new browser window and Voila!

![Python app Hosted on ECS](https://cdn-images-1.medium.com/max/3278/1*ix8h7XVRyUOhbx744ZlFrQ.png)*Python app Hosted on ECS*

* Try to refresh the window several times and see how the Hostname getting changed and you can see three different hostnames which is according to the number of Replica tasks we have established in our service.

![](https://cdn-images-1.medium.com/max/2000/1*fKHlhQlaJXLEXruWqMp4Fw.png)

![](https://cdn-images-1.medium.com/max/2000/1*_GSH8qgqFXEKwZxSBvM26A.png)

![](https://cdn-images-1.medium.com/max/2000/1*t6oKYzA27rf7LPrIoSkcmA.png)

Thank you very much for your time on reading this post, This can be seriously difficult to understand in one read because trust me I had to spend days to gather all these information (We are data scientists, not DevOps engineers) before I successfully deployed this simple app on ECS so just be patient and progress by step by step. If you run into any errors, Let me know.

### Keen on getting to know me and my work? Click [here](https://www.djayasekara.com/) for more!

**Also, Check out my latest venture [ToyBubba](https://www.toybubba.com.au/).**

🔥 With using ToyBubba — Kids Toy Subscription service, Your baby will receive a box full of educational toys each month by our toy designers curated just for your baby according to your preferences!! 🔥
[**ToyBubba — Kids Toy Subscription Box in Australia / Toy Bubba**
*By using Toy Bubba — Kids Toy Subscription Service, Get this unique chance to surprise your kid each month by giving…*www.toybubba.com.au](https://www.toybubba.com.au/)

Cheers!

<iframe src="https://medium.com/media/723583d8d128e18cae4d9f7c0af1413b" frameborder=0></iframe>
