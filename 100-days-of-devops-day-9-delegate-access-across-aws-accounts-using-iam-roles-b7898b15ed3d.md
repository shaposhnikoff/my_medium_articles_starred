
# 100 Days of DevOps — Day 9-Delegate Access Across AWS Accounts Using IAM Roles

Welcome to Day 9 of 100 Days of DevOps, On Day 8 I explained about the really critical topic STS https://medium.com/@devopslearning/100-days-of-devops-day-8-introduction-to-aws-security-token-service-sts-b0f164e5d6a3 on Day 9 let’s continue this journey in talking about Delegate Access Across AWS Accounts Using IAM Roles(Basically doing things via AWS Console)

***Problem:** How to share resources in different AWS accounts i.e User in Account B(Developer) should have Read-Only Access to S3 Bucket in Account A(Production).*

***Solution:** By setting up cross-account access using IAM roles.*

***Advantage***

* *We don't need to set up individual IAM user in each account*

* *The user doesn’t need to sign out of one account and sign into another account to access resources.*

***Pre-requisites***

* *You need two AWS accounts(Account A(PROD)) and Account B(Developer))*

* *An AWS S3 bucket created in Production Account A.*

***Step1:** Create an IAM Role in Account A(This is to establish the trust between the two accounts)*

* *Go to IAM console [https://console.aws.amazon.com/iam/home?region=us-west-2#/home](https://console.aws.amazon.com/iam/home?region=us-west-2#/home)*

* *Click on Roles, Create role*

* *This time, select Another AWS account and enter Account ID of Account B*

![](https://cdn-images-1.medium.com/max/4204/1*HYEfqzsBB9O1MF-aPwv2eg.png)

* *To get the account id(Click on the IAM user on the top of the console and click on My Account)*

![](https://cdn-images-1.medium.com/max/2000/1*a7jxLv3INMV43Kr_qt8COg.png)

* *In next screen click on Create Policy and paste the below mentioned(Change the bucket name with the name of the bucket you want to share with Development Account) OR Choose S3ReadOnlyPolicy*

<iframe src="https://medium.com/media/7a9512641bff61e7a30ff910a24ce5ed" frameborder=0></iframe>

* *Click Next and give your Role name*

![](https://cdn-images-1.medium.com/max/4112/1*KfrhBO6WjxC_BzDyyxb75A.png)

* *Note down the Role ARN, we need it later*

***Step2: **Grant Access to the role(This will allow users in Account B permissions to allow switching to the role)*

* *Go to the Developer Account B*

* *Go to the IAM console [https://console.aws.amazon.com/iam/home?region=us-west-2#/home](https://console.aws.amazon.com/iam/home?region=us-west-2#/home)*

* *Select the user and Add Inline policy*

![](https://cdn-images-1.medium.com/max/5036/1*C7wAYob94ZzX901J8L1y2Q.png)

* *Attach the below-mentioned policy with the IAM role we created in Step1*

<iframe src="https://medium.com/media/ba5ff054c238935d349b45e49f2472ef" frameborder=0></iframe>

* *Give this policy some name*

![](https://cdn-images-1.medium.com/max/4372/1*oAiyClVYd1H-xJTk49XtSA.png)

***Step3:** Test access by Switching the role*

* *Again go back to the Account Tab but this time click on Switch Role*

![](https://cdn-images-1.medium.com/max/2000/1*OS_GTCiSvpV1FZF5P7Jorg.png)

* *Fill all the details*

![](https://cdn-images-1.medium.com/max/5096/1*YegJz7sl-LZWsSL6xh9QbA.png)

    ** Account: This is Prod/Account A ID
    * Role: Role we created in Step1: **S3ReadOnlyAccesstoDevAccount
    * **Display Name: Any display name
    * Switch Role*

* *You will see something like this*

![](https://cdn-images-1.medium.com/max/5732/1*x652CQGeSS-shbLCYzXoGQ.png)

*NOTE: You cannot switch to a role when you are signed in as the AWS account root user.*

* *Now go to S3 console and try to access S3 bucket which is present in Account A.*

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
