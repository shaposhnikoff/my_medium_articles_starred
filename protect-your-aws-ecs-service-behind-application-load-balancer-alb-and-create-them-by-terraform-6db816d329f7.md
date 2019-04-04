
# Protect your AWS ECS Service behind Application Load Balancer (ALB) and create them by Terraform

A small piece of post during the journey starting from zero and diving deep into Docker, AWS Elastic Container Service and Terraform DevOps orchestration.

![The entire journey to deploy docker infrastructure on AWS ECS from scratch.](https://cdn-images-1.medium.com/max/2532/1*DDQrSu1diJf8JwN45ll_rA.png)*The entire journey to deploy docker infrastructure on AWS ECS from scratch.*

This article is a write-up about how to deploy containers using Application Load Balancer. Just as when I was browsing posts, articles online for hours and hours and finally found that enlightening piece of information, I hope this article can do the same to you, and be one another resource out there.

Junior developers always have a hard time understanding the crazy tools all over the way in the industry. There’re still a lot for me to learn. I’ll make them clear for places where I don’t know what’s going on, or why it behaves that way. It’s largely possible that my understanding is wrong, so please feel free to give me feedback. Let’s discuss DevOps together!

To clarify, this post is not to walk through how to setup Node.js API, MongoDB and AWS ECS service or task definition (maybe for another post), this post focuses on how to use application load balancer (ALB) with Amazon’s docker container service ECS and setup security settings, also many Gotchas I came across when navigating through the AWS ecosystem.

## Tech Stack

Before we begin, let’s skim through the tech stack so you can see if this fits your case.

### Local

* Nginx as primary web server

* Node.js as API server. MongoDB using remote Atlas, so we don’t need to host our own MongoDB.

* Docker

### Orchestration

* Terraform to create and update AWS infra.

### AWS

* Elastic Container Service (ECS)

* Domain name purchased from Route 53

* SSL Certificate from AWS Certificate Manager

* Application Load Balancer, but only having one service to route to, so always use the / path only.

You can take a look at the resulting Github repo: [https://github.com/rivernews/bb-rest-api](https://github.com/rivernews/bb-rest-api)

We’re skipping pre-setups such as Docker (Docker Compose), Node.js and Nginx. We also skipped preparations on AWS such as IAM, domain name using Route 53, and all hard-coded stuffs in variables.tf and other Terraform files. If you’re looking at the repository for reference, don’t look at the folder terraform-experiments. All used terraform scripts are in terraform directory.

## Creating Application Load Balancer (ALB): The Frontier

![](https://cdn-images-1.medium.com/max/3880/1*dDlXR-czqLSnnIqEzRxCNw.png)

Instead of letting users access the instance’s public domain address or IP directly, we want to let load balancer serve as a single source of incoming request. Then, we can configure ALB to route internally to different web apps. However, in our example, we only have one service (i.e. one app), so our ALB will only forward to the root path /. Multiple services may require slightly different configuration of the listeners on ALB, which we’ll not cover here.

There are three components for ALB: security group, listener and target group.

### Security Group

![](https://cdn-images-1.medium.com/max/2000/1*bB4Cm-QC4AQKNaKBHjqIDw.png)

Security group is for limiting incoming (called `ingress`) and outgoing (called `egress`) traffic by port, protocol or certain IP source. It can be attached to various kinds of AWS resources, like load balancer, EC2 instance or autoscaling group (which is a dynamic portion of EC2 instances). You can also “limit access by another security group” — this lets you limit the traffic to internal AWS service, which we will apply later.

For our public-facing ALB, we don’t want to limit to specific IP, since we want the web app to be accessible globally. So we set the source as 0.0.0.0/0. If you want to restrict visitors to certain country, or even certain organization, this is the place to configure.

### Listeners on ALB

![](https://cdn-images-1.medium.com/max/2000/1*8TXQ3g7tDCkZ3JuIr2rVnQ.png)

Load balancer has “listeners” you can add. Listeners can accept incoming http or https request, and decide the following routing behavior, say forward to your web app or redirect to somewhere else.

I’m not sure why it’s specific to only http and https, after all they are both TCP and are only different by port (80 and 443). Perhaps it has something to do with the Layers — ALB listener is at Application Layer, whereas security group is at Network Layer.

Here, we set listeners for only HTTPS/443, since we want to block all http connections and only allow https. Since the server will be serving a API for a mobile app, this might be OK. In other case, e.g. a web app, you may want to accept HTTP/80 as well, and redirect to HTTPS so the web app is more available to the users, regardless of what protocol they type on their end.

### Target Group

![](https://cdn-images-1.medium.com/max/2648/1*P-iN5GJK1p9CGDgFSCG1dw.png)

ALB listener always forward to a target group. Listener and target group, they are pairs. ALB cannot forward directly to your web app — which is something a bit counter-intuitive if you’re new to AWS’s ELB like me.

Before request can be routed to your web app from ALB, you have to create a target group. You can then `register` your web app in the target group, and your web app is the “target”. Within one target group, you can register multiple web apps, with each using different ports. It’s worth mentioning at this point that, the port number is no longer associate with that port number of the request sent by internet users. This port number is used by the traffic emitted by ALB, which is an internal traffic.

ALB and target group together serve as a router that routes an incoming request based on its protocol (http or https) to your web app. Your web app is some server like Nginx, Django or Node.js, which again listen at a certain port. The port number you registered in target group has to correspond to the port number of your server app.

Here, our first layer web server is Nginx and listens on port 80, so we register a target of port 80 in the target group. Also, ALB’s health check happens here:

<iframe src="https://medium.com/media/acab2d232d4b853f4996906c9f286344" frameborder=0></iframe>

We’ll skip the task definition and service part.

## Protect Our Server and Set Behind ALB

![See the incoming request from left, which bypass load balancer and attempts to access EC2 instances directly.](https://cdn-images-1.medium.com/max/3532/1*XQuGsqk0zpPQC0nrmN_D1Q.png)*See the incoming request from left, which bypass load balancer and attempts to access EC2 instances directly.*

Even if we set our ALB as only listening to HTTP/443, this does not cover the case where an user accessing our EC2 instance’s public DNS/IP instead of the domain name associated with ALB. Access rules to EC2 instance is controlled by EC2 instance’s security group.

We have several options here:

* **1. Reuse ALB’s security group**: we can restrict port to only allow 443 ingress traffic, but this will allow request globally.

* **2. Create another security group and set ALB’s security group as the source**: this will let us accept traffic with the same ports as ALB, but limit traffic to only from ALB.

Option 2 allows our instances to be accessed only from ALB’s traffic, so is much secure and we’re using this option here. Note that now if you also limit port number to 80 for this security group, ECS Service will have trouble accessing the instance (so will the containers reside in it). This might be due to the fact that ALB and ECS Service use dynamic ports range from 32768 to 61000, in order to support routing to multiple services using one single ALB. I’m not super clear about this yet, I’m only sure that limiting to port 80 will give you 503 server temporarily unavailable.

<iframe src="https://medium.com/media/1d7f567ede3b30e12a77b767690ce21b" frameborder=0></iframe>

## Executing Terraform Script

Once we have all the necessary resources defined in the terraform script, we are ready to execute.

And that’s it! This is my first post on Medium, and I hope you enjoy it! We certainly skipped many steps in the preparation details. As I understand more about Docker, ECS and the ecosystem, I hope I can share more about them, and help more new comers like me to get started.

## Reference & Inspiration

* [Stackoverflow: How to allow elastic load balancer through port 80 in security groups?](https://serverfault.com/questions/321820/how-to-allow-elastic-load-balancer-through-port-80-in-security-groups)
The discussion here inspired me of specifying a security group (SG) instead of an IP in SG’s source field.

* [Reddit: AWS ECS with ELB and HTTPS](https://www.reddit.com/r/aws/comments/8efr6b/aws_ecs_with_elb_and_https/)
This discussion guided me through how I should configure load balancer and EC2 instance security group.

* [Terraform: AWS Provider](https://www.terraform.io/docs/providers/aws/index.html)
Also very useful — the Terraform official doc.
