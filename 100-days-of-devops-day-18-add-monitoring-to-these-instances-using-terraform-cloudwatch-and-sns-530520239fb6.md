Unknown markup type 10 { type: [33m10[39m, start: [33m75[39m, end: [33m104[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m106[39m, end: [33m126[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m128[39m, end: [33m145[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m147[39m, end: [33m173[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m744[39m, end: [33m755[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m757[39m, end: [33m764[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m766[39m, end: [33m769[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m771[39m, end: [33m778[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m780[39m, end: [33m801[39m }

# 100 Days of DevOps — Day 18-Add monitoring to these instances using Terraform(CloudWatch and SNS)

Welcome to Day 18 of 100 Days of DevOps, Let continue our journey, so far we have discussed fundamentals of terraform, build VPC and EC2 instance using terraform, today let’s add monitoring piece to it
[**100 Days of DevOps — Day 15- Introduction to Terraform**
*Welcome to Day 15 of 100 Days of DevOps, Let continue our journey and focus on Automation especially on Infrastructure…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-15-introduction-to-terraform-7a168dec8d38)
[**100 Days of DevOps — Day 16- Building VPC using Terraform**
*Welcome to Day 16 of 100 Days of DevOps, Let continue our journey, yesterday I discussed terraform, today let’s build…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-16-building-vpc-using-terraform-7c507ce07413)
[**100 Days of DevOps — Day 17- Creating EC2 Instance using Terraform**
*Welcome to Day 17 of 100 Days of DevOps, Let continue our journey, so far I havr discussed terraform fundamentals and…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-17-creating-ec2-instance-using-terraform-c876a09d9d66)

* *Adding monitoring piece can be achieved via two ways, we can add separate cloudwatch and SNS module and then call it in our EC2 module*

* *We can call the Cloudwatch and SNS terraform code directly in EC2 terraform module, I prefer this approach as we want all our EC2 instance comes up with monitoring enabled*

* *First, let start with SNS topic, which is required to send out a notification via Email, SMS when an event occurs.*

<iframe src="https://medium.com/media/6b70325ad04d7d3a5e0bf1557e6b5116" frameborder=0></iframe>

    ** Here I am trying to create an SNS topic resource
    * give your SNS topic, some name
    * After that I am using a default policy *
[**AWS: sns_topic - Terraform by HashiCorp**
*Provides an SNS topic resource.*www.terraform.io](https://www.terraform.io/docs/providers/aws/r/sns_topic.html)

*NOTE: As with SNS, someone needs to confirm the email subscription that why I am using local-exec provisioners with terraform.*

*Now let’s take a look at terraform code for CloudWatch.*

*This code is divided into two parts*

* ***Setup CPU Usage Alarm using the Terraform***

<iframe src="https://medium.com/media/050ec8f8cba093a9511c18f59c37e8e3" frameborder=0></iframe>

    ** Setup an alarm name
    * This field is self explanatory,supported operators GreaterThanOrEqualToThreshold, GreaterThanThreshold, LessThanThreshold, LessThanOrEqualToThreshold.
    * evaluation_period: The number of periods over which data is compared to the specified threshold(I setup 2 just for demo purpose but its completly depend upon your requirement)
    * metric_name: Please check the link below list of services that publish cloudwatch metrics
    * namespace: The namespace for the alarm's associated metric(Check the second column of the link below)
    * period: period in second(I am using 120 sec or 2min but again it completly depend upon your requirements)
    * statistic: The statistic to apply to the alarm's associated metric, supported value: SampleCount, Average, Sum, Minimum, Maximum
    * threshold: The value against which the specified statistic is compared.(I set it up as 80% i.e when CPU utilization goes above 80%)
    * alarm_actions: The list of actions to execute when this alarm transitions into an ALARM state from any other state. Please note, each action is specified as an Amazon Resource Name (ARN)
    * dimensions: The dimensions for the alarm's associated metric. Again check the below mentioned link for supported dimensions*

## *AWS Services That Publish CloudWatch Metrics*
[**AWS Services That Publish CloudWatch Metrics - Amazon CloudWatch**
*The following AWS services publish metrics to CloudWatch. For information about the metrics and dimensions, see the…*docs.aws.amazon.com](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/aws-services-cloudwatch-metrics.html)

* *So what this is doing this code is going to send an email using SNS notification when CPU Utilization is more than 80%.*

* *Now look at the other part is to perform system and instance failure and send an email using SNS notification*

<iframe src="https://medium.com/media/b043630447867627eef48f47e14419fa" frameborder=0></iframe>

    ** Most of this code is almost similar, only difference is metric _name here is StatusCheckFailed*

Final EC2 code with CloudWatch Monitoring and SNS topic enabled look like this

<iframe src="https://medium.com/media/a1bf193e7adb1da569d1f65d015e1cf6" frameborder=0></iframe>

GitHub Link
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/tree/master/two-tier-environment)

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
