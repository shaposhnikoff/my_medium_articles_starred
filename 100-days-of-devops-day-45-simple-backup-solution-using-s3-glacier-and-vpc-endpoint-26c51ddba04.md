
# 100 Days of DevOps — Day 45-Simple Backup Solution using S3, Glacier and VPC Endpoint

Welcome to Day 45 of 100 Days of DevOps, Focus for today is Simple Backup Solution using S3, Glacier and VPC Endpoint

![](https://cdn-images-1.medium.com/max/2264/1*gqD38RVO5eB-87TQYSsMXg.jpeg)

*So this is what I am going to implement using Terraform*
> *Step1: Need to assign IAM Role to the instance, so that has permission to write to S3 bucket*

<iframe src="https://medium.com/media/e31bea9850e9d5ee4575c322d332f597" frameborder=0></iframe>
> *Step2: Create VPC endpoint for S3 bucket, so that data never leaves the AWS network*

<iframe src="https://medium.com/media/cd8265a0a397e9b050e12c03bff36857" frameborder=0></iframe>
> *Step3: Create an S3 bucket and assigned it a LifeCycle Policy so that data after 30 days move to Standard IA storage class and after 60 days to Glacier.*

<iframe src="https://medium.com/media/77a3649d5797c342f08d6337ceca8609" frameborder=0></iframe>
> *Step4: Login to host and install epel-release, this is required as we need pip to install aws cli*

    *# yum -y install epel-release*

* *Next step is to install pip*

    *# yum -y install python2-pip.noarch*

* *Then aws cli*

    *# pip install awscli*

* *Test your access to S3 bucket*

    *# aws s3 cp wtmp s3://terraform-20190327040316452900000001*

    *upload: ./wtmp to s3://terraform-20190327040316452900000001/wtmp*

*NOTE: As we already setup the IAM role, we don't need to hardcode the value of AWS_ACCESS_KEY and AWS_SECRET_ACCESS_KEY.*

* *Now I am going to write a simple script which is going to sync data from your local folder to s3 bucket every minute*

    *# cat /usr/bin/awss3sync.sh*

    *#!/bin/bash*

    *aws s3 sync /var/log/. s3://terraform-20190327040316452900000001*

* *Put that script in crontab so that it will execute every min*

    *[root@ip-172-31-31-68 bin]# crontab -l*

    **/1 * * * * /usr/bin/awss3sync.sh*

* *Dont forget to change the permission of the script*

    *# chmod +x /usr/bin/awss3sync.sh*

* *Your simple backup solution is ready, it's not a perfect solution but it’s easy to implement and will perform the given task.*

**GitHub Link**
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/tree/master/backup_solution)

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
