
# 100 Days of DevOps — Day 40-Introduction to AWS Config

Welcome to Day 40 of 100 Days of DevOps, Focus for today AWS Config
> *What Is AWS Config?*

*AWS Config provides a detailed view of the configuration of AWS resources in your AWS account. This includes how the resources are related to one another and how they were configured in the past so that you can see how the configurations and relationships change over time.*
> ***Features***

* *Track state of all resources(OS level too — Windows/Linux)*

* *Meet your compliance need(PCI-DSS, HIPAA)*

* *Validate against AWS Config Rule*
> ***Setting up AWS Config***

* *Go to AWS Config [https://us-west-2.console.aws.amazon.com/config](https://us-west-2.console.aws.amazon.com/config/home?region=us-west-2#/welcome) → Get started*

![](https://cdn-images-1.medium.com/max/5096/1*3xYoIrueZ1CZuyDr_MlQSQ.png)

    ** All resources: You can check on, Record all rsources supported in this region
    OR
    Global resources like IAM
    OR
    We can even check specific resources eg: EC2*

    ** Amazin S3 bucket: This bucket will recieve configuration history and configuration snapshot files
    * Amazon SNS topic(Optional): We can send config changes to S3 bucket
    * AWS Config role: It give AWS config read-only access(IAM Role)to AWS resource*

![](https://cdn-images-1.medium.com/max/3564/1*2RgtEYgOL3wPfK4pAX4pmA.png)

    ** Skip this for the time being*

* *Confirm and AWS Config setup for us.*

* *Check the status of AWS config, by click on the status icon on the top of the page*

![](https://cdn-images-1.medium.com/max/5760/1*TsPiEKfV8cKv5czXvXefYQ.png)

* *Now click on Resource and then Instance*

![](https://cdn-images-1.medium.com/max/3552/1*ZBCIWS01B2sk4aHSyiG8Wg.png)

![](https://cdn-images-1.medium.com/max/5168/1*BtZKUKsZn3VyJR0pa23Mpg.png)

* *Click on the Configuration timeline*

![](https://cdn-images-1.medium.com/max/5760/1*TuZKRYkTCXMjOSgCu-V8DA.png)

* *Scroll down and click on changes*

![](https://cdn-images-1.medium.com/max/4224/1*5sWVPV7huHhizeSgjjFtMw.png)

![](https://cdn-images-1.medium.com/max/3888/1*XeY2rZjJjIoSoowXmTiyBg.png)
> *Scenario: Last time we skipped the rule section, this time let add all the config rule, our task for today to make sure for an account is compliant*

* *CloudTrail must be enabled*

* *S3 bucket versioning must be enabled*

* *EC2 instance must be a part of VPC*

* *We are only using instance type as t2.micro*

![](https://cdn-images-1.medium.com/max/5740/1*BRo4-g4YCq3Kyy2lGh8GOA.png)
> *Search for CloudTrail and select cloudtrail-enabled*

![](https://cdn-images-1.medium.com/max/3236/1*1sxpl-th9fX7Xf2kxDQtGA.png)

* *You don't need to change any of the default value and click on save*

![](https://cdn-images-1.medium.com/max/3924/1*pAtlb2akRfAXR49FO4ymqA.png)
> *Same way search for S3 bucket versioning enabled*

![](https://cdn-images-1.medium.com/max/3672/1*pIfkGK7DCADqYTmBVclBbg.png)
> *Search for ec2-instances-in-vpc*

![](https://cdn-images-1.medium.com/max/3516/1*ad-_RvSlKa9l3fYNZhFaJQ.png)

* *This requires some changes as you need to specify your VPC id*

![](https://cdn-images-1.medium.com/max/3052/1*uhQ3Q2tetwWw-ojSbT3N6g.png)
> *Search for desired-instance-type*

![](https://cdn-images-1.medium.com/max/3532/1*Ak4OsdN4xsxQDuKIKspo5g.png)

* *Add the instanceType Value to t2.micro*

![](https://cdn-images-1.medium.com/max/3048/1*7WGCKmbkf4BJh0ipeu9A6A.png)

* *Finally, you will see something like this*

![](https://cdn-images-1.medium.com/max/5072/1*E6puaXOIjvIVYL90yPELuQ.png)

* *If you further drill down, as you can see this instance is using t2.medium while in config rule for the desired-instance-type we choose t2.micro*

![](https://cdn-images-1.medium.com/max/5112/1*FEcS5u-1IX3ahAKGaQcRiA.png)

* *One more example, as you can see in this case S3 bucket is non-compliant*

![](https://cdn-images-1.medium.com/max/5108/1*9ONDeM9ldz09gNgruPv6Ew.png)

* *If we can go to the S3 bucket and enabled versioning*

![](https://cdn-images-1.medium.com/max/2000/1*TiwAmy9TevJvMCBmYL8M0g.png)

* *As we remediated the issue, to see the immediate effect*

![](https://cdn-images-1.medium.com/max/5084/1*l8EadlyHaEtBkLJkcOcizg.png)

* *We are back in business*

![](https://cdn-images-1.medium.com/max/5072/1*sJS5Eg5Ku29ChtFP9QJ7oA.png)

***Terraform Code***

* *Now we need to automate the entire process and then is no better tool other then terraform to do a job for us*

<iframe src="https://medium.com/media/a0dea55065844733fe43edd0ad13ff87" frameborder=0></iframe>

*GitHub Link*
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/tree/master/aws_config)

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
