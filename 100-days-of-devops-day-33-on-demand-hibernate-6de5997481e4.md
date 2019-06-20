
# 100 Days of DevOps — Day 33- On Demand Hibernate

Welcome to Day 33 of 100 Days of DevOps, Focus for today is On Demand Hibernate. This is the new feature released by AWS during reinvent 2018
> Why do we need On Demand Hibernate

* *EC2 Instance can be up in a matter of a second, but booting OS and Application sometimes take several minutes*

* *Warming up cache can take several minutes*

* *Hibernate stores the in-memory state of the instance(along with Private and Elastic IP)*

* *Pick up exactly where you left off*

* *Supported instance type(M3, M4, M5, C3, C4, C5, R3, R4, and R5)*

* *Supported OS(Amazon Linux 1)*

* *Amazon Linux 2(work in progress), No support for Centos family :(. (Coming soon: Amazon Linux 2, Ubuntu, Windows Server 2008 R2, Windows Server 2012, Windows Server 2012 R2, Windows Server 2016, along with the SQL Server variants of the Windows AMIs)*

* *Support for on-demand and reserved instance*
> ***How it works***

*When we instruct instance to hibernate it will write it’s in-memory data to the mounted EBS Volume and then shut down itself*

* *AMI as well as both EBS Volume need to be encrypted*

* *Encryption ensures proper protection for sensitive data when it is copied from memory to the EBS volume*

* *To test this feature, I need to do some reverse engineering*

***Step1:** Create the snapshot of root volume and then use copy option, which enables encryption option.*

![](https://cdn-images-1.medium.com/max/2912/1*QySKkrI37okpqDNhW2LfCA.png)

* *Create an AMI out of this volume(which is encrypted by default)*

![](https://cdn-images-1.medium.com/max/5600/1*56J7t6VBZCieOh9nmj00xg.png)

* *Choose the supported instance type and make sure hibernation is enabled*

![](https://cdn-images-1.medium.com/max/2176/1*FmK5hwCrSahCiz7ulpoE3w.png)

* *Also, make sure /root has sufficient space to hold in-memory data*

* *I can use uptime command to see that the instance has not been rebooted, but has continued from where it left off*

![](https://cdn-images-1.medium.com/max/2000/1*R9-qVlyty__Y_A34MNg-ig.png)

*Reference*
[**Amazon EC2 Now Lets you Pause and Resume Your Workloads**
*You can now hibernate your Amazon EC2 instances backed by Amazon EBS and resume them at a later time. Applications can…*aws.amazon.com](https://aws.amazon.com/about-aws/whats-new/2018/11/amazon-ec2-now-lets-you-pause-and-resume-your-workloads/)
[**New - Hibernate Your EC2 Instances | Amazon Web Services**
*As you know, you can easily build highly scalable AWS applications that launch fresh EC2 instances on an as-needed…*aws.amazon.com](https://aws.amazon.com/blogs/aws/new-hibernate-your-ec2-instances/)

*Looking forward from you guys to join this journey and spend a minimum an hour every day for the next 100 days on DevOps work and post your progress using any of the below medium.*

* *Twitter: [@100daysofdevops](http://twitter.com/100daysofdevops) OR @[lakhera2015](https://twitter.com/lakhera2015)*

* *Facebook: [https://www.facebook.com/groups/795382630808645/](https://www.facebook.com/groups/795382630808645/)*

* *Medium: [https://medium.com/@devopslearning](https://medium.com/@devopslearning)*

* *Slack: [https://devops-myworld.slack.com/messages/CF41EFG49/](https://devops-myworld.slack.com/messages/CF41EFG49/)*

* *GitHub Link:[https://github.com/100daysofdevops](https://github.com/100daysofdevops)*
