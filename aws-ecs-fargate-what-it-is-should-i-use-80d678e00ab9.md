
# AWS ECS Fargate — What it is? Should I use?

AWS Fargate: What it is? Is it the right choice for my current project?

## Deploy your Docker containers and let AWS do the rest.

Last week, here on [Loopix](http://www.loopix.com.br/), me and my workmate Yurhi were responsible for deploying our application on AWS. After successfully deploying it on Elastic Beanstalk, I learned about ECS [Fargate](https://aws.amazon.com/fargate/). What brought my attention to ECS was how easy it is to manage your cluster, and Fargate makes it better by abstracting all infrastructure for you, scalling automatically and charging you only when the container is active. You don’t have to worry about CPU utilisation or Memory consumption (or paying for the unused CPU/Memory) after deploying. You have no access to EC2 instances or anything like that, you only set up the connection configuration, how much memory and CPU you container needs to run the process and launch it (which wasn’t a walk in the park, to be honest).

## What is ECS Fargate?

At the end of the year of 2017, AWS released a new service that made possible to run containers directly in the cloud: as a service, not as virtual infrastructure environment.

Fargate can be classified as a "CaaS" (Container as a Service), which opens a lot of possibilities. Fargate handles all underlying infrastructure and make you see your container as a single machine.

Fargate tries to make our life easier and takes care of our container execution. You inform your vCPU needed, the available memory for each process, setup the network settings and then let AWS scale.

![AWS Fargate deploy explanation](https://cdn-images-1.medium.com/max/3800/1*9_cw2S7vVHtn8eW4hrYBsw.png)*AWS Fargate deploy explanation*

After defining a SERVICE in ECS, it will take care to deploy and maintain it available, scaling as it needs. And price is surprisingly small. [Check here for more information on how you are charged with Fargate](https://aws.amazon.com/fargate/pricing/).

## Does it fit your needs?

If you already used ECS, you know that you have only one EC2 type available per Cluster, and you define it when creating the cluster. If you have a task that need more memory, for example, you would have to manually attach the EC2 instance to you cluster.

There are a few other solutions, but they all are full of layers of configuration. It is a lot of work, a lot of chances to misconfigure and it is incredibly hard to maintain.
> Should I use AWS Fargate on my projects?

If your project is suffering some of this issues, maybe Fargate can help you:

* Unused Memory/CPU, making you pay a lot of money for little use.

* On demand/scheduled tasks costs you EC2 dedicated instance, that is always running (and burning money)

For better comparison and choice, I recommend reading [this article](https://dev.to/diogoaurelio/container-orchestration-in-aws-comparing-ecs-fargate-and-eks-56d1), by Diogo Aurelio, as he compares ECS, ECS Fargate and EKS in a great manner.
> Ok, Fargate seems to be a great help for me. Where should I start?

For this step we need:

* A buildable Docker container.

* A Virtual Private Cloud (AWS VPC).

* Basic AWS setup: (Security groups, IAM roles, etc)

* (Optional) Elastic Load Balancer and Target Groups.

## Enough chitchat, let's deploy some containers.

This week I'll be posting 3 more articles on this topic. They will be linked here after they are published.

* How to setup your Task Definitions and Cluster

* Setting up your Load Balancers

* Setting up ECS Services (and why you should use them).

References (Other places where you can learn more about this):

* [https://medium.com/devopslinks/ecs-vs-eks-vs-fargate-the-good-the-bad-the-ugly-9f68bfc3bb73](https://medium.com/devopslinks/ecs-vs-eks-vs-fargate-the-good-the-bad-the-ugly-9f68bfc3bb73)

* [https://aws.amazon.com/fargate/](https://aws.amazon.com/pt/fargate/)

* [https://aws.amazon.com/fargate/pricing](https://aws.amazon.com/fargate/pricing)

* [https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_GetStarted.html](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_GetStarted.html)

* [https://dev.to/diogoaurelio/container-orchestration-in-aws-comparing-ecs-fargate-and-eks-56d1](https://dev.to/diogoaurelio/container-orchestration-in-aws-comparing-ecs-fargate-and-eks-56d1)

* [https://www.reddit.com/r/aws/comments/98pp40/ecs_fargate_or_ec2_which_one/](https://www.reddit.com/r/aws/comments/98pp40/ecs_fargate_or_ec2_which_one/)

* [https://medium.freecodecamp.org/amazon-fargate-goodbye-infrastructure-3b66c7e3e413](https://medium.freecodecamp.org/amazon-fargate-goodbye-infrastructure-3b66c7e3e413)
