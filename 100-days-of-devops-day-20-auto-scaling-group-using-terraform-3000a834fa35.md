
# 100 Days of DevOps — Day 20— Auto-Scaling Group using Terraform

Welcome to Day 20 of 100 Days of DevOps, Let continue our terraform journey, yesterday we created an application load balancer, let expand that concept and put this load balancer as a part of the auto-scaling group.
[**100 Days of DevOps — Day 19 - Application Load Balancer using Terraform**
*Welcome to Day 19 of 100 Days of DevOps, Let continue our terraform journey and see how can we create Application Load…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-19-application-load-balancer-using-terraform-58794aeaf31f)
> *What is Auto Scaling?*

*What auto-scaling will do, it ensures that we have a correct number of EC2 instances to handle the load of your applications.*
> ***How Auto Scaling works***

* *It all started with the creation of the Auto Scaling group which is the collection of EC2 instances.*

* *We can specify a minimum number of instances and AWS EC2 Auto Scaling ensures that your group never goes below this size.*

* *The same way we can specify the maximum number of instances and AWS EC2 Auto Scaling ensures that your group never goes above this size.*

* *If we specify the desired capacity, AWS EC2 Auto Scaling ensures that your group has this many instances.*

* *Configuration templates(launch template or launch configuration): Specify Information such as AMI ID, instance type, key pair, security group*

* *If we specify scaling policies then AWS EC2 Auto Scaling can launch or terminate instances as demand on your application increased or decreased. For eg: We can configure a group to scale based on the occurrence of specified conditions(dynamic scaling) or on a schedule.*

![](https://cdn-images-1.medium.com/max/2000/1*x1uQTv92ZZZl3YrUSDzaMQ.png)

***Reference:** [https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html](https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html)*

*In the above example, Auto Scaling has*

* *A minimum size of one instance.*

* *Desired Capacity of two instances.*

* *The maximum size of four instances.*

* *Scaling policies we define adjust the minimum or a maximum number of instances based on the criteria we specify.*

***Step1:** The first step in creating the AutoScaling Group is to create a launch configuration, which specifies how to configure each EC2 instance in the autoscaling group.*

<iframe src="https://medium.com/media/101d69be47b3b7631a25cd7be41473cf" frameborder=0></iframe>

* *Most of the parameters look similar to EC2 configuration except lifecycle parameter which is required for using a launch configuration with an ASG*
[**100 Days of DevOps — Day 17- Creating EC2 Instance using Terraform**
*Welcome to Day 17 of 100 Days of DevOps, Let continue our journey, so far I havr discussed terraform fundamentals and…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-17-creating-ec2-instance-using-terraform-c876a09d9d66)

* *One of the available lifecycle settings are create_before_destroy, which, if set to true, tells Terraform to always create a replacement resource before destroying the original resource. For example, if you set create_before_destroy to true on an EC2 Instance, then whenever you make a change to that Instance, Terraform will first create a new EC2 Instance, wait for it to come up, and then remove the old EC2 Instance.*

* *The catch with the create_before_destroy the parameter is that if you set it to true on resource X, you also have to set it to true on every resource that X depends on (if you forget, you’ll get errors about cyclical dependencies). In the case of the launch configuration, that means you need to set create_before_destroy to true on the security group*
[**Resources - Configuration Language - Terraform by HashiCorp**
*Resources are the most important element in a Terraform configuration. Each resource corresponds to an infrastructure…*www.terraform.io](https://www.terraform.io/docs/configuration/resources.html#lifecycle-lifecycle-customizations)

*Step2:*

<iframe src="https://medium.com/media/6ae1f48d55ab47ce05904f1d99039533" frameborder=0></iframe>

* *Next step is to create an auto-scaling group using the aws_autoscaling_group resource.*

* *This autoscaling group will spin a minimum of 1 instance and a maximum of 2 instances OR completely based on your requirement.*

* *It’s going to use the launch configuration we created in the earlier step.*

* *We are using an aws_availibity_zone resource which will make sure instance will be deployed in different Availability Zone.*

* *A list of aws_alb_target_group ARNs, for use with Application Load Balancing, this is created During Day 19 when we created Application Load Balancer. If you are still not sure about the Value, in the outputs.tf file put this entry*

    *output **"target_group_arn" **{
      **value **= **"${aws_lb_target_group.my-alb-tg.arn}"
    **}*

*and then run terraform apply command, you will see something like this*

    ***Outputs:***

    ***target_group_arn = arn:aws:elasticloadbalancing:us-west-2:XXXXXX0:targetgroup/my-alb-tg/85a23209f9c37964***

GitHub Link for the complete ASG code
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/tree/master/two-tier-environment/auto-scaling)

*Looking forward from you guys to join this journey and spend a minimum an hour every day for the next 100 days on DevOps work and post your progress using any of the below medium.*

* *Twitter: [@100daysofdevops](http://twitter.com/100daysofdevops) OR @[lakhera2015](https://twitter.com/lakhera2015)*

* *Facebook: [https://www.facebook.com/groups/795382630808645/](https://www.facebook.com/groups/795382630808645/)*

* *Medium: [https://medium.com/@devopslearning](https://medium.com/@devopslearning)*

* *Slack: [https://devops-myworld.slack.com/messages/CF41EFG49/](https://devops-myworld.slack.com/messages/CF41EFG49/)*

* *GitHub Link:[https://github.com/100daysofdevops](https://github.com/100daysofdevops)*

***Reference***
[**100 Days of DevOps — Day 0**
*D-day is just one day away and finally, this is a continuation of the post(I posted a month earlier)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-0-4f2c9750542d)
[**100 Days of DevOps**
*Motivation*medium.com](https://medium.com/@devopslearning/100-days-of-devops-81faf13bf772)
