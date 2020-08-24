
# 100 Days of DevOps — Day 37- Automate the Process of AMI Creation Using System Manager Maintenance…

Welcome to Day 37 of 100 Days of DevOps, Focus for today is Automate the Process of AMI Creation Using System Manager Maintenance Windows

*On Day 36 I discussed System Manager and its other components, let extend that concept further and see how we can automate the process of AMI creation using Maintenance Window*
[**100 Days of DevOps — Day 36-Introduction to AWS System Manager**
*Welcome to Day 36 of 100 Days of DevOps, Focus for today is AWS System Manager*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-36-introduction-to-aws-system-manager-21ffb5d634d0)

*What is AWS Systems Manager Maintenance Windows?*
> *AWS Systems Manager Maintenance Windows*

*AWS Systems Manager Maintenance Windows let you define a schedule for when to perform potentially disruptive actions on your instances such as patching an operating system, updating drivers, or installing software or patches. Each Maintenance Window has a schedule, a maximum duration, a set of registered targets (the instances that are acted upon), and a set of registered tasks. You can add tags to your Maintenance Windows when you create or update them.*
> ***Configuring access to Maintenance Window***

*This can be done with the help of IAM role so that System Manager can act on our behalf in creating and performing maintenance window*

*Go to IAM Role [https://console.aws.amazon.com/iam/](https://console.aws.amazon.com/iam/) → Create role → EC2 → Choose AmazonSSMMaintenanceWindowRole*

![](https://cdn-images-1.medium.com/max/3292/1*rUgKeecLSfKNdvrXBIUV3w.png)

* *Give your role name and create it*

* *Now click on the role you have just created and click on Trust relationship*

![](https://cdn-images-1.medium.com/max/5136/1*ctF1VW3SfsIU9CUACTqfsg.png)

<iframe src="https://medium.com/media/bff0f1881018b803c269bbe844591178" frameborder=0></iframe>

    ** Add this entry("Service": "ssm.amazonaws.com")
    * Please don't forget to add comma(,) after "Service": "ec2.amazonaws.com",*

* *Add an inline policy to the user, also make sure that particular user also have AWSSSMFullAccess Policy attach to it*

<iframe src="https://medium.com/media/b4efda6f85f832fe56c2f80cf2f60d4d" frameborder=0></iframe>
> ***Next step is to create the Maintenance Window***

*Go to [https://us-west-2.console.aws.amazon.com/systems-manager](https://us-west-2.console.aws.amazon.com/systems-manager) → Action → Maintenance Windows*

![](https://cdn-images-1.medium.com/max/2000/1*n-HcUpotmymv5_W4RY09Rw.png)
> ***Once the maintenance window create, choose Target → Register target***

![](https://cdn-images-1.medium.com/max/4896/1*SkJfFYbKuTjWu9y4TfBJMg.png)

![](https://cdn-images-1.medium.com/max/2332/1*i7fb9blLkC8L7TPFycdaNg.png)
> ***Click on the Tasks Tab and Choose AWS-Createimage as automation Document***

![](https://cdn-images-1.medium.com/max/4896/1*VQ7eHiak9br0KCbZ51Bm1w.png)

![](https://cdn-images-1.medium.com/max/2000/1*2xodJ9r6pc8FUUR-D9I3ww.png)

* *Keep everything default, except*

    ** Give the instance id from where you want to create the image
    * NoReboot: set it to true else it will reboot the instance,during image creation
    * AutomationAssumeRole: Paste the arn of role we create in earlier step*

![](https://cdn-images-1.medium.com/max/2000/1*AD_RBkEiCb3Qa43QDgiO-w.png)

* *Once the schedule hit, you will see something like this*

![](https://cdn-images-1.medium.com/max/4840/1*BqwNNik4kioYRSyapUkQBQ.png)

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
