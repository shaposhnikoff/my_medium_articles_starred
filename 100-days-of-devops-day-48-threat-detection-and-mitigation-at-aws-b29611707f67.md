
# 100 Days of DevOps — Day 48- Threat detection and mitigation at AWS

Welcome to Day 48 of 100 Days of DevOps, Focus for today is Threat detection and mitigation at AWS

*This 27th Wednesday, I got the chance to attend AWS Santa Clara Summit and there I attended two Security Related Session, so just sharing my experience with everyone*

    ** Threat detection and mitigation at AWS
    * Find all the threats: AWS threat detection and mitigation(Chalk Talk)*

*As I am mostly involved in DevOps field but Security is one field which always fascinates me. I got a chance to learn four new AWS resources(I know/heard about these tools but never got a chance to work and implement this but hopefully I will implement that in future)*

    ** Amazon GuardDuty
    * Amazon Macie
    * AWS Security Hub
    * Amazon Inspector*
> *Security Solutions offered by AWS*

![](https://cdn-images-1.medium.com/max/5572/1*Yb3-g4cwvo8mkF3IK-IfEQ.png)

* *This is one of the main highlights, Keep human away from the data*

![](https://cdn-images-1.medium.com/max/5584/1*AbJIL8LjZ7vVN-Z9gefBrQ.png)
> *Data Inputs used for Threat Detection Pipelines*

![](https://cdn-images-1.medium.com/max/5620/1*FTu0y2VWjjXkREdVX7rAXw.png)

* ***CloudTrail** tracks all user activity*

![](https://cdn-images-1.medium.com/max/4124/1*wBqR8W152s6Azjb07ZSW7w.png)

*NOTE: Please make sure no one has access to turn off cloudtrail and if someone try to do that you should have AWS Lambda running to turn it on back.*
[**100 Days of DevOps -Day 3(Introduction to CloudTrail)**
*Welcome to Day 3of 100 Days of DevOps, Let extend the journey of DevOps Monitoring and Alerting*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-3-introduction-to-cloudtrail-5ce923f44584)

* ***VPC Flow Logs **to see all the network activity happening in your account*

![](https://cdn-images-1.medium.com/max/5712/1*76ALFsIRBcIUDS4UhzFSBQ.png)

***NOTE:** Pay special attention to the second last column and look for REJECT, make sure you have some type of alarm setup to check the frequency in which they are happening.*
[**100 Days of DevOps — Day 28- Introduction to VPC Flow Logs**
*Welcome to Day 28 of 100 Days of DevOps, Focus for today is VPC Flow logs*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-28-introduction-to-vpc-flow-logs-d11a99cd18ca)

* ***CloudWatch Logs: **To send all the different type of logs and monitor it in almost real time*

![](https://cdn-images-1.medium.com/max/5584/1*vK8ktSPF8dJiA1SFaqsNJA.png)
[**100 Days of DevOps — Day 4(CloudWatch log agent Installation — Centos7)**
*Welcome to Day 4 of 100 Days of DevOps, On the first day we discussed CloudWatch…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-4-cloudwatch-log-agent-installation-centos7-d11054fffdf4)

* ***DNS Logs:** All the queries occurred in your DNS resolver inside your VPC*
> *Now we have collected the data, its time to analyze it*

![](https://cdn-images-1.medium.com/max/5068/1*art006BDRJqPzKa87P4xng.png)

*OR*

![](https://cdn-images-1.medium.com/max/4392/1*sg1FN_Ywkdba3u4JJv6LAw.png)
> What is Amazon GuardDuty?

![](https://cdn-images-1.medium.com/max/3656/1*Q6XPOASaCpFjK3qxKX8QVQ.png)
> *How GuardDuty Works?*

![](https://cdn-images-1.medium.com/max/2080/1*5pEdAuTIj3ctjYC82Srytw.png)
> *What can Amazon GuardDuty Detect?*

![](https://cdn-images-1.medium.com/max/5060/1*MotefhzSnBr8do8cwwAPQw.png)
> Amazon Macie

![](https://cdn-images-1.medium.com/max/5676/1*OPPEkld-M8xH46R4HSn3xw.png)

![](https://cdn-images-1.medium.com/max/5496/1*qC5mZ-ZL778TOymuM-ttUQ.png)

![](https://cdn-images-1.medium.com/max/5280/1*wN5NVeSKeyHBNWq0xwEVWA.png)

![](https://cdn-images-1.medium.com/max/5504/1*A31J9IN6vnHQYBsCkMnQyw.png)

![](https://cdn-images-1.medium.com/max/5272/1*LueY8SokPmz2R6Imp4Nmfg.png)

![](https://cdn-images-1.medium.com/max/3504/1*FFWDdY7Zu9eTmFNIWNoY8g.png)
> AWS Security Hub

![](https://cdn-images-1.medium.com/max/5260/1*Bs2f7TT4hWuRuvLNJsAxeg.png)

![](https://cdn-images-1.medium.com/max/5204/1*gBSdAzwkHAHstlIhgfqZ6A.png)
> Amazon Inspector

![](https://cdn-images-1.medium.com/max/5268/1*tDSZ_NiWQ6PdDkmHlx9ynQ.png)

![](https://cdn-images-1.medium.com/max/3436/1*mwKwT_D-AO6qXdkncpfoPA.png)
> Threat Detection

![](https://cdn-images-1.medium.com/max/4048/1*ZIkLFshDQ83DVTQft33Dcg.png)
> AWS Config Rules

![](https://cdn-images-1.medium.com/max/5416/1*wSe1YvhXFPkmIR1qz7Aq7A.png)
> Amazon CloudWatch Events

![](https://cdn-images-1.medium.com/max/5276/1*p_AnjLi1p_4x4P_sh6w0Tw.png)
> Threat Remediation

![](https://cdn-images-1.medium.com/max/4640/1*UOqAeHhAGM0EX-IvQpWeVA.png)

*AWS Share this workshop*
[**Scaling threat detection and response in AWS**
*This hands-on workshop is where you will learn about a number of AWS services involved with threat detection and…*scaling-threat-detection.awssecworkshops.com](https://scaling-threat-detection.awssecworkshops.com/)

*I will highly recommend everyone to go through it as this will give you knowledge about tools like*

* *Amazon GuardDuty*

* *Amazon Macie*

* *Amazon Inspector*

* *AWS Security Hub*

*Not only to detect the thread but how to remediate and response to it.*

***NOTE:** Implementing this above lab will cost you money*

*Use AWS Security Services and learn how to use them to identify and remediate threats in your environment.*

*Scenario: As a DevOps engineer your task is to securely monitor your AWS infrastructure and respond to any security event in your environment.*

*End Goal: How to use these services to investigate threats during and after the attack and setup notification and response pipeline and add additional protections to improve the security posture of your environment.*
> *Architecture*

* *To set up the environment, AWS provides CloudFormation Template*

[*https://s3-us-west-2.amazonaws.com/sa-security-specialist-workshops-us-west-2/threat-detect-workshop/staging/01-environment-setup.yml](https://s3-us-west-2.amazonaws.com/sa-security-specialist-workshops-us-west-2/threat-detect-workshop/staging/01-environment-setup.yml)*

* *CloudFormation is going to set up the following environment*

![](https://cdn-images-1.medium.com/max/3020/1*7L3iVy8Mo8oNzZzgz5J9wQ.png)
> Enable GuardDuty

![](https://cdn-images-1.medium.com/max/4628/1*9Q204ev9g2LY1peD39xTug.png)

![](https://cdn-images-1.medium.com/max/3220/1*BdCoCDHHWeiNSVno08u-oA.png)
> Enable Amazon Macie

![](https://cdn-images-1.medium.com/max/5408/1*YmHZmP7lOxAA2YxYDGjn7w.png)

![](https://cdn-images-1.medium.com/max/4296/1*zLbIbtAeffAV-6q3WdIG6w.png)
> Enable AWS Security Hub

![](https://cdn-images-1.medium.com/max/4116/1*NkekCxnpNhMZ1Lj0L-49yA.png)

![](https://cdn-images-1.medium.com/max/2844/1*mSW6m2z6otzTXPJHiJJq8A.png)

*Complete Architecture will look like this*

![](https://cdn-images-1.medium.com/max/2000/1*2ny-3qGrq_RU0C9yGHkupg.png)
> To Simulate Attack

* *CloudFormation template which will simulate the actual attack you will be investigating*

***S3 URL***

[*https://s3-us-west-2.amazonaws.com/sa-security-specialist-workshops-us-west-2/threat-detect-workshop/staging/02-attack-simulation.yml](https://s3-us-west-2.amazonaws.com/sa-security-specialist-workshops-us-west-2/threat-detect-workshop/staging/02-attack-simulation.yml)*
> *Detect and Respond*

*After 15–20min, you will see a message like these in your email notification*

![](https://cdn-images-1.medium.com/max/2524/1*nosjuYeWbXrMIO6AqFcCrQ.png)

![](https://cdn-images-1.medium.com/max/4136/1*9hw7JhZ4T6tC93YJjugpnQ.png)

![](https://cdn-images-1.medium.com/max/4364/1*VaJSQ_gkwZ3OWsmS0M4_jw.png)

![](https://cdn-images-1.medium.com/max/4476/1*lr_OrfOAO-F0qGvAeOACMA.png)

* *If you go back to your AWS Guardduty page, you will see Guardduty has the following findings*

![](https://cdn-images-1.medium.com/max/4956/1*jRsbY-E7WVmDpuD9XxD2Rg.png)
> Part 1 — Compromised AWS IAM credentials

![](https://cdn-images-1.medium.com/max/5056/1*eHILToh7jSfnrvxUk-2_Yg.png)
> Part 2 — Determine if ssh password authentication is enabled on the EC2 instance (AWS Security Hub)

![](https://cdn-images-1.medium.com/max/4740/1*3HWKR5i5rG0pj98y9zOOGg.png)

![](https://cdn-images-1.medium.com/max/5060/1*E87pbWwodU436Be-fIMTUA.png)
> Part 3 — Compromised S3 bucket

![](https://cdn-images-1.medium.com/max/4140/1*K4fhV4WAn1VA8-H6klcsWg.png)

*Reference*
[**aws-samples/aws-scaling-threat-detection-workshop**
*A hands-on workshop to learn how to do threat detection and response in AWS. …*github.com](https://github.com/aws-samples/aws-scaling-threat-detection-workshop/)

<iframe src="https://medium.com/media/eed33d4de23f3aa14c7eccce58feedbd" frameborder=0></iframe>

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
