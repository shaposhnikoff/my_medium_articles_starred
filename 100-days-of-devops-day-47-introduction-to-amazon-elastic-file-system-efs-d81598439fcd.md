
# 100 Days of DevOps — Day 47-Introduction to Amazon Elastic File System (EFS)

Welcome to Day 47 of 100 Days of DevOps, Focus for today is Amazon Elastic File System (EFS)
> *What is Amazon EFS?*

*Amazon EFS provides scalable file storage for use with Amazon EC2. You can create an EFS file system and configure your instances to mount the file system. You can use an EFS file system as a common data source for workloads and applications running on multiple instances.*
> *Step1: Create a security group to access your AWS EFS Filesystem*

* *To enable traffic between EC2 instance and EFS Filesystem we must need to allow port 2049.*

* *On the EC2 client side, we must need to allow outbound access to Port 2049. As we are allowing all the outbound traffic so this pre-requisite is already met.*

![](https://cdn-images-1.medium.com/max/4908/1*jT-Q31ZY0BeQrIIFKinjFQ.png)

* *On the mount target end, we must need to allow TCP Port 2049 inbound for NFS from all EC2 instances on which we want to mount the filesystem.*

    *Go to [https://us-west-2.console.aws.amazon.com/ec2](https://us-west-2.console.aws.amazon.com/ec2) → NETWORK & SECURITY → Security Groups --> Create Security Group*

![](https://cdn-images-1.medium.com/max/3680/1*ccEOrnQQLWb2a9GgpzCdbg.png)

    ** Type : Should be NFS
    * Source: Rather than opening it for the entire subnet, we are only opening it for EFS Client Security Group*
> *Step2: Create an Amazon EFS FileSystem*

* *EFS FileSystem can be mounted to multiple EC2 instances running in different availability zone with the same region. These instances use mount targets created in each Availability Zone to mount the filesystem using the standard Network File System v4.1(NFS v4.1).*

*NOTE: All the instances where we are trying to mount the Filesystem must be the part of the same VPC.*

[*https://us-west-2.console.aws.amazon.com/efs](https://us-west-2.console.aws.amazon.com/efs) → Create file system*

![](https://cdn-images-1.medium.com/max/4640/1*KBn2vMS9zAuqrKbOBvXMCA.png)

![](https://cdn-images-1.medium.com/max/5024/1*IGK1zd1nuCMCzI99-bp1DA.png)

    ** Add tag to your FileSystem
    * Keep all the other values default
    * Review and create the filesystem*

* *Once it's available you will see something like this*

![](https://cdn-images-1.medium.com/max/5200/1*n4yVrjwu8b4jRXyU21JHWA.png)

* *Now if click on the Amazon EC2 mount instructions(from local VPC), that will give you the detailed instruction which package to install and how to mount the filesystem*

![](https://cdn-images-1.medium.com/max/2636/1*BDsdD2IdytyP8Hwkx0S99w.png)

<iframe src="https://medium.com/media/0fdee3daa3c505642c71b6213cdf5e20" frameborder=0></iframe>

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
