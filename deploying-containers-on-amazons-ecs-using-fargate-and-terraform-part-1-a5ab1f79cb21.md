
# Deploying Containers on Amazon’s ECS using Fargate and Terraform: Part 1



Before we jump into the tutorial I wanted to do a brief overview. Fargate is a new launch type within ECS for deploying containers. It came out around the end of November in 2017 and has now expanded to us-east-1, us-east-2, us-west-2, and eu-west-1. In a nutshell, Fargate gives you the ability to run containers without having to manage servers. You no longer have to provision, configure, and scale clusters of virtual machines to run containers.

![](https://cdn-images-1.medium.com/max/2000/1*DV4NyKOc2w_9I4ntaigQhg.jpeg)

Terraform enables you to safely and predictably create, change, and improve infrastructure. It’s essentially “infrastructure as code.” In part 1 of this tutorial we’re going to take a Docker image and deploy it to ECS using Fargate. We’ll run three containers, put them behind a load balancer, and set up some auto scaling rules. In [part 2](https://medium.com/@bradford_hamilton/deploying-containers-on-amazons-ecs-using-fargate-and-terraform-part-2-2e6f6a3a957f), we’ll automate all of it with Terraform.

## Let’s Get Started

Log in to you AWS account (or create one) and go to the [ECS console](https://us-west-2.console.aws.amazon.com/ecs/home?region=us-west-2#/getStarted). Click on “Get Started” which should be right in the middle of the page. If you already have clusters within ECS then it will be in grey next to “Create Cluster.” Create cluster is similar but the Get Started path is geared a bit more towards people doing it for the first time and has slightly less options with the ability to configure.

**Step 1: Container and Task**

First thing we’re going to do is click on “Configure” for a custom container definition.

![](https://cdn-images-1.medium.com/max/3724/1*jMcOgi6dUj6peIYOlJX2fA.png)

Once inside we’ll begin defining our container. For this walk through I’m going to use an image I created called [crystal_blockchain](https://hub.docker.com/r/bradfordhamilton/crystal_blockchain/). If you’re interested in building the application itself, head over to [https://medium.com/@bradford_hamilton/write-your-own-blockchain-and-pow-algorithm-using-crystal-d53d5d9d0c52](https://medium.com/@bradford_hamilton/write-your-own-blockchain-and-pow-algorithm-using-crystal-d53d5d9d0c52).

It’s a small web app that listens on port 3000. We’ll be using a lot of defaults and lower memory configurations, etc that we know the app will run fine with. When you deploy your own application you’ll want to figure out the appropriate configurations for your app’s needs.

First we’ll name the container, give it the image, and map it to port 3000. For the image name, if it’s an image on Docker hub – you can simply refer to it with [namespace]/[image]:[tag]. However, if it’s a private image on Docker hub it seems you will need to host it on [ECR](https://us-west-2.console.aws.amazon.com/ecs/home?region=us-west-2#/repositories) instead (Amazon’s Elastic Container Registry) for it to work. Things are always changing fast and I could be wrong so feel free to look further into that if and when you need to point to a private image. Lastly port mappings allow containers to access ports on the host container instance to send or receive traffic. We’ll skip adding any soft/hard memory limits for this walk through.

![](https://cdn-images-1.medium.com/max/4448/1*_JoSIqhUJVtKAPvWtEANJA.png)

Next in the “Advanced container configuration” add 1024 CPU units in the environment section.

![](https://cdn-images-1.medium.com/max/4308/1*0qBnqF6kUHmiI1b36nwvQA.png)

Finally in storage and logging, change the awslogs-group value to /ecs/crystal-blockchain-task (which we’ll be naming soon).

![](https://cdn-images-1.medium.com/max/4352/1*pX4aYNeQ18BvHaiVXlZdHg.png)

Click “Update” then right below the container definition is the task definition which may be showing an error regarding Task CPU. Let’s go in and edit that.

![](https://cdn-images-1.medium.com/max/3636/1*4tYa7EDOZuRjUt4_fZglbw.png)

Rename it, change the Task memory to 2GB, and the Task CPU to 1vCPU. We also need a task execution role as this is what authorizes ECS to pull images and publish logs for your task. This takes the place of the EC2 Instance role when running Fargate tasks. If you don’t already have an “ecsTaskExecutionRole” of sorts then select the option to create one.

![](https://cdn-images-1.medium.com/max/4436/1*tYnfvwM4vQ8KYmszBuCrvA.png)

Save that and click next.

**Step 2: Service**

For our service we’ll want to select “Application Load Balancer” and we can leave the rest as is.

![](https://cdn-images-1.medium.com/max/3496/1*PSqGkVuJOAB77IHcDAlA6A.png)

**Step 3: Cluster**

Here lets rename the cluster and it will by default be set to create a new VPC and new subnets. Again, click next.

![](https://cdn-images-1.medium.com/max/3500/1*JxtNhfhATSE_L9rOGQJasQ.png)

**Step 4: Review**

If all is well click create! It will bring you to a “Launch Status” screen and you can see your resources being created.

![](https://cdn-images-1.medium.com/max/6284/1*GnZPCM-HFKa5aI0SThNE8A.png)

This can take a few minutes so be patient. Once all the checks are green, we’ll click on “View service.” From there click on “Update” in the upper right hand corner of the service screen.

**Step 1: Configure service**

Here the only thing we’re going to do is change the “Number of tasks” to 3 and move to the next step.

![](https://cdn-images-1.medium.com/max/3556/1*ohg0CssRS7rEta1_kO_Z6A.png)

**Step 2: Configure network**

Everything on this screen will be the defaults so we can just move on to step 3.

**Step 3: Set Auto Scaling**

We’re going to make a few changes here. Again this would very much depend on your application’s needs but we’ll set up some auto scaling to see how it works. Click the “Configure Service Auto Scaling” radio button and more content will display. We’ll set the minimum number of tasks to 3. This makes it so that no matter what, your application will always run at least 3 tasks and won’t scale below that. We can also set the desired number to 3 which is the number of tasks the service will start with before any scaling begins. For maximum number of tasks I asked for 10. If our application gets slammed and it starts to scale up, 10 will be the maximum number of tasks it will scale to.

![](https://cdn-images-1.medium.com/max/3616/1*UR5epplxdW9BJAxCw4h2rQ.png)

We’ll need an autoscale role so if you don’t already have something like an “escAutoscaleRole,” again here we’ll select to create one for us. This gives your service permission to describe your CloudWatch alarms and registered services, as well as permission to update your service’s desired count on your behalf.

Now click on “Add scaling policy” and select “Step scaling.” Here we define a policy based around a CloudWatch alarm. We’re going to say that if the CPU utilization rises above or equal to 85% for over 5 minutes one time, run this policy.

![](https://cdn-images-1.medium.com/max/3148/1*_NpEglGp45uBrfl23w25MA.png)

Click the save button inside the create new alarm box we just filled out. Now add the action we want to call when the alarm is triggered.

![](https://cdn-images-1.medium.com/max/3292/1*RpPPN_PXNUt1xS0lW4x0Jw.png)

Here we’re simply saying we want to add one task when this policy is called. We’ll set the cooldown period to 60 seconds between scaling actions. Let’s do the same thing for scaling down. Save and then click to create another policy.

![](https://cdn-images-1.medium.com/max/3312/1*KeplNR1TeUhDxYUC0IckCQ.png)

Very similar set up here except we want to say when the CPU utilization is below or equal to 10% for 5 minutes one time, scale down. Set the scaling action to remove 1 task when this alarm is triggered with the same 60 second cooldown between scaling actions.

![](https://cdn-images-1.medium.com/max/3320/1*FKe7x3WaQU2YV5sF0fvsXw.png)

Save and go to step 4.

**Step 4: Review**

We’re so close! Double check everything is how it should be and and click Update Service! It should go through and give you green check marks on your updates and then you can view your service again. Now if you click on the “Tasks” tab in your service you should see 3. They may be spinning up and in a pending state but give it a few minutes and you will see them all running.

### You did it!

So now what? Where do we view our app? Well since we put all the instances behind a load balancer, all we have to do is go to it in our browser. It will take care of alternating between your containers to keep traffic equally distributed. I highly recommend digging around your service, tasks, etc to get a good feel for everything.

To get the load balancer url go back into the details tab of your service and click on the target group name under Load Balancing. You’ll then be inside your EC2 management console in the Target Groups section. Scroll down into the description tab and click on the load balancer associated with this target group.

Now do the same thing within the load balancer console, scroll down into the description and you will see the “DNS name.” Copy that!!

Visit that in your browser and don’t forget to tag on port :3000. You should see some JSON returned successfully from the crystal blockchain app.

### Where to go from here

Take a break! Part 2 of this tutorial will be creating and automating all of this with Terraform. The application we deployed in this example is small and we had no need to set up a database or any other services that a production app would likely have but I think it’s a good start for getting familiar with AWS. I sometimes find AWS to be a bit intimidating and/or cumbersome to use, so I wanted to hopefully help someone get off the ground with a set up like this.

[Part 2 is here!](https://medium.com/@bradford_hamilton/deploying-containers-on-amazons-ecs-using-fargate-and-terraform-part-2-2e6f6a3a957f) Get ready to automate this entire deployment with Terraform :)
