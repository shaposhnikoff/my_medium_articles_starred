
# 100 Days of DevOps — Day 12- How to ensure that users can’t turn off CloudTrail

Welcome to Day 12 of 100 Days of DevOps, Let continue our journey and discuss other common problem such as how to restrict a user so that he cannot turn off the cloud trail.

***Problem:** Ensures users can’t turn off the Cloudtrail*

***Solutions:***

* *IAM Policy*

* *CloudWatch Events*

* *Lambda*

*We all understand the importance of CloudTrail, it records each action taken by a user, role, or an AWS service. Events include actions taken in the AWS Management Console, AWS Command Line Interface, and AWS SDKs and APIs.*

*For more info*
[**100 Days of DevOps -Day 3(Introduction to CloudTrail)**
*Welcome to Day 3of 100 Days of DevOps, Let extend the journey of DevOps Monitoring and Alerting*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-3-introduction-to-cloudtrail-5ce923f44584)
> ***Solution 1:** IAM Policy*

* *If we add this simple deny policy to the user he/she will not be able to disable CloudTrail*

<iframe src="https://medium.com/media/b6320089b3848aa321222b84ce0cb7c2" frameborder=0></iframe>

*For more info IAM Policy*
[**100 Days of DevOps — Day 10- Restricting User to Launch only T2 Instance**
*Welcome to Day 10 of 100 Days of DevOps, Let continue our journey with IAM and let discuss one of the common…*medium.com](https://medium.com/devopslinks/100-days-of-devops-day-10-restricting-user-to-launch-only-t2-instance-509aaaec5aa2)
> ***Solution2:** CloudWatch Events*

*Go to CloudWatch → Events → Rules*

![](https://cdn-images-1.medium.com/max/5756/1*Y6pLRUIF5zs585sxp5zB9w.png)

* *This is just the subset of what we are going to do in Solution 3*

* *What this will do, if now any user tries to disable CloudTrail, you will receive an email notification*

![](https://cdn-images-1.medium.com/max/4372/1*DeK1fL-rYNvqua5ATmxfkw.png)
> ***Solution3:** Using Lambda*

***Step1:** Create IAM Role so that Lambda can interact with CloudWatch Events*

    *Go to IAM Console [https://console.aws.amazon.com/iam/home?region=us-west-2#/home](https://console.aws.amazon.com/iam/home?region=us-west-2#/home) --> Roles --> Create role*

![](https://cdn-images-1.medium.com/max/3412/1*_C92ySkyjMw9l0wFubo_vQ.png)

* *In the next screen, select Create Policy*

![](https://cdn-images-1.medium.com/max/3324/1*mtYyb58SZfmu4xUJIyKqgA.png)

    ** Service: Search for CloudWatch Logs
    * Action: Select CreateLogGroup, CreateLogStream and PutLogEvents
    * Resource: Select Any*

* *Review and give your policy some name*

* *Final policy look like this*

<iframe src="https://medium.com/media/046283be397eb4b1f8421ba0f09bf419" frameborder=0></iframe>

* *Add this newly created policy to the role and add one more policy to it(AWSCloudTrailFullAccess), so that Lambda will respond to any CloudTrail event.*

![](https://cdn-images-1.medium.com/max/5220/1*lv5hs1KMRwIgm-5QB62nnQ.png)

***Step2:** Create Lambda function*

* *Go to Lambda [https://us-west-2.console.aws.amazon.com/lambda/home?region=us-west-2#/home](https://us-west-2.console.aws.amazon.com/lambda/home?region=us-west-2#/home)*

* *Select Create Function*

![](https://cdn-images-1.medium.com/max/5384/1*M3VKLoR9TVzH0kEkQH48fg.png)

    ** Select Author from scratch
    * Name: Give your Lambda function any name
    * Runtime: Select Python2.7 as runtime
    * Role: Choose the role we create in first step
    * Click on Create function*

* *Copy paste this code*

<iframe src="https://medium.com/media/8af544ec3d56b676eea841976e690fca" frameborder=0></iframe>

![](https://cdn-images-1.medium.com/max/5148/1*pCc0NGfpbMCO9RFWDeDvSA.png)

*P.S: I don’t own this code and I came across this code while doing a google search*

***Step3:** Create CloudWatch Events*

    *Go to CloudWatch --> [https://us-west-2.console.aws.amazon.com/cloudwatch/home?region=us-west-2#](https://us-west-2.console.aws.amazon.com/cloudwatch/home?region=us-west-2#) --> Rules --> Create rule*

![](https://cdn-images-1.medium.com/max/5076/1*-20L7B4gv_xH1UZNF74y3g.png)

    ***Event Source***

    ** Select Event Pattern
    * Service Name: Select CloudTrail
    * Event Type: AWS API Call via CloudTrail
    * Specific operation(s): StopLogging*

    ***Targets***

    ** Select Lambda function
    * Function: Select the Function we have just created*

    *Add target*

    ** This time chhose SNS topic
    * Choose the SNS topic we just create*

* *What this will do, if any user tries to disable CloudTrail*

    ** It will send SNS notification
    * It will revert the change and enable CloudTrail back*

*One thing I just want to highlight and people may notice that User/Developer shouldn’t have access to CloudTrail in a Production environment and I completely agree. I came across this issue while working in Dev environment where we give complete Admin access to Developer so that they will not be blocked but some user starts disabling CloudTrail for whatever reasons ;-).*

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
