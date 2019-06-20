
# 100 Days of DevOps — Day 36-Introduction to AWS System Manager

Welcome to Day 36 of 100 Days of DevOps, Focus for today is AWS System Manager
> *What Is AWS Systems Manager?*

*AWS Systems Manager is a collection of capabilities for configuring and managing your Amazon EC2 instances, on-premises servers and virtual machines, and other AWS resources at scale.*

![](https://cdn-images-1.medium.com/max/2476/1*vBasHglMDtq1i6QOl0pRsQ.png)

***Reference***
[**What Is AWS Systems Manager? - AWS Systems Manager**
*AWS Systems Manager is a collection of capabilities for configuring and managing your Amazon EC2 instances, on-premises…*docs.aws.amazon.com](https://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html)

### *Pre-requisites: There are two pre-requisites on setting up System Manager*
> *Setting up IAM Role for System Manager*

*To use system manager you need to set up two roles*

* *One role authorizes the user to use System Manager*

* *First one assign AmazonSSMFullAccess policy to the user*

![](https://cdn-images-1.medium.com/max/5324/1*PWFuPmByaOV_XdBHpDIT0w.png)

* *Other authorizes systems to be authorized by the system manager*

* *Create a new role and assign AmazonEc2Rolefor SSM*

![](https://cdn-images-1.medium.com/max/3144/1*HynikbegQAyGdQYleUtN0w.png)

![](https://cdn-images-1.medium.com/max/3144/1*mFGBkZzOc2J1FHdM2BTQ1A.png)

* *Attach the role, I have created earlier to an existing instance or during instance creation*

![](https://cdn-images-1.medium.com/max/5740/1*9jnnPX_rf0CRgNuywVp5eA.png)

*For more info about IAM*
[**100 Days of DevOps — Day 26-Introduction to IAM**
*Welcome to Day 26 of 100 Days of DevOps, Let continue our journey with terraform and see how we can terraform IAM.*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-26-introduction-to-iam-b69315623b01)
> *Installing SSM Agent*

<iframe src="https://medium.com/media/7e8c9ea2cb37f34674027c2c32adfba0" frameborder=0></iframe>

* *This is installed on the instance end and let instance to communicate with System Manager*

* *Once installed go to System Manager home page [https://us-west-2.console.aws.amazon.com/systems-manager](https://us-west-2.console.aws.amazon.com/systems-manager) → Shared Resources → Managed Instances*

![](https://cdn-images-1.medium.com/max/3824/1*e_i0q_SVEgo1jQlmy835sw.png)

* *Go to Actions → Run Command → AWS-RunShellScript → Commands → Type any Linux command eg: ls -l → Target Instance(Select the instance)*

![](https://cdn-images-1.medium.com/max/4800/1*9xxLysY-44sjWl5zpWlRbQ.png)

* *You can also check the output under view output tab*

![](https://cdn-images-1.medium.com/max/4720/1*gjaLzGsY4sDKCWLtK143sQ.png)

* *To execute the same command via aws cli*

<iframe src="https://medium.com/media/7305a6d46f501fc82c1c44bdf3b0d917" frameborder=0></iframe>
> ***What is AWS Systems Manager State Manager***

* *AWS Systems Manager State Manager is a secure and scalable configuration management service that automates the process of keeping your Amazon EC2 and hybrid infrastructure in a state that you define.*

* *One of the use case I found out of AWS System Manager State Manager is to run the command on a scheduled basis(eg: SnapShot Creation)*

![](https://cdn-images-1.medium.com/max/4804/1*t0lmRsE8JdN9jgkOLFpDDg.png)

![](https://cdn-images-1.medium.com/max/5708/1*10Aqgql2iCRe-uZu2sUhtg.png)

*But I believe there is a much better way to achieve this eg: EBS LifeCycle Manager*
[**100 Days of DevOps — Day 14- How to automate the process of EBS Snapshot Creation**
*Welcome to Day 14 of 100 Days of DevOps, Let continue our journey and focus of this week is a daily DevOps task, to run…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-14-how-to-automate-the-process-of-ebs-snapshot-creation-86418f2d7f09)

*One other option that has tried is the AMI creation on the scheduled basis but there is a bug already raised for this issue [https://forums.aws.amazon.com/thread.jspa?messageID=893995&#893995](https://forums.aws.amazon.com/thread.jspa?messageID=893995&#893995)*

![](https://cdn-images-1.medium.com/max/4748/1*yizpv3P5Uc2vkaGIPJi7lg.png)
> ***AWS Systems Manager Parameter Store***

*AWS System Manager Parameter store provides secure, hierarchical storage for configuration data management and secrets management. We can store data such as,*

* *passwords*

* *database strings*

* *license codes*

*Which we can then be programmatically accessed via the SSM API.
 
 Parameter store is offered at no additional charge*

*Go to the parameter store [https://us-west-2.console.aws.amazon.com/systems-manager](https://us-west-2.console.aws.amazon.com/systems-manager) → Create Parameter*

![](https://cdn-images-1.medium.com/max/2668/1*glSY7BXQz9LV6RPrBHRJPw.png)

* *Now to retrieve this value via command line*

    *$ aws ssm get-parameters --names "testpass"*

    *{*

    *"InvalidParameters": [],*

    *"Parameters": [*

    *{*

    *"Name": "testpass",*

    *"LastModifiedDate": 1552923749.085,*

    *"Value": "test123",*

    *"Version": 1,*

    *"Type": "String",*

    *"ARN": "arn:aws:ssm:us-west-2:XXXXXXX:parameter/testpass"*

    *}*

    *]*

    *}*
> ***How to store a secure string***

* *When we store a secure string in the EC2 parameter store, the data is encrypted by the KMS key associated with my account.*

![](https://cdn-images-1.medium.com/max/2300/1*vvMwruwViVVXOwsHm6oFNg.png)

* *If you try to verify via UI, you will see something like this*

![](https://cdn-images-1.medium.com/max/4844/1*eXyCv9oJaVrChUCvkBfO_w.png)

* *You can access it via command line*

    *$ aws ssm get-parameters --names "mysecurestring" --with-decryption*

    *{*

    *"InvalidParameters": [],*

    *"Parameters": [*

    *{*

    *"Name": "mysecurestring",*

    *"LastModifiedDate": 1552923877.289,*

    *"Value": "test123",*

    *"Version": 1,*

    *"Type": "SecureString",*

    *"ARN": "arn:aws:ssm:us-west-2:349934551430:parameter/mysecurestring"*

    *}*

    *]*

    *}*

* *To store the secret*

    *# To store the secret*

    ***# aws ssm put-parameter --name "secret-password" --value 'XXXXX' --type SecureString --key-id XXXXXX***

    *{*

    *"Version": 1*

    *}*
> ***AWS Systems Manager Inventory***

*AWS Systems Manager Inventory provides visibility into your Amazon EC2 and on-premises computing environment. You can use Inventory to collect metadata from your managed instances. You can store this metadata in a central Amazon Simple Storage Service (Amazon S3) bucket, and then use built-in tools to query the data and quickly determine which instances are running the software and configurations required by your software policy, and which instances need to be updated.*

* *Setting up inventory is pretty straightforward [https://us-west-2.console.aws.amazon.com/systems-manager/](https://us-west-2.console.aws.amazon.com/systems-manager/) → Inventory → Setup Inventory*

![](https://cdn-images-1.medium.com/max/3848/1*LAW6N7c4qk_Te8eqCnbsWw.png)

![](https://cdn-images-1.medium.com/max/3784/1*fJfULN6stvMGntylSwSIKQ.png)

    ** Give you inventory some name
    * Targets: Either Manually select the instance or better to use Tag so that all the future installed instance will be tracked automatically
    * Schedule: How frequently you want to collect Invnetory
    * Parameter: Different Parameter you want to collect*

* *After waiting for a few mins, you will see something like this*

![](https://cdn-images-1.medium.com/max/5084/1*ozBT2fZoASU2guBOBRcb6g.png)

* *If you go to managed instance tab, select your instance and then inventory tab*

![](https://cdn-images-1.medium.com/max/4864/1*czdCeVvbtT_brnbrxJtHHg.png)

* *AWS Inventory is nicely integrated with AWS Config service*

* *Go to config [https://us-west-2.console.aws.amazon.com/config](https://us-west-2.console.aws.amazon.com/config) and under Resource Type → SSM → ManagedInstanceInventory*

![](https://cdn-images-1.medium.com/max/5232/1*G-569CU1CGP298iwyzAGIQ.png)

* *Under configuration timeline, you will see something like this, all the changes happen to this instance*

![](https://cdn-images-1.medium.com/max/5760/1*KWqD0BM6yMsq9CcBKeabjw.png)

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
