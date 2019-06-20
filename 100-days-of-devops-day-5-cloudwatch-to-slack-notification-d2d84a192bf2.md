
# 100 Days of DevOps — Day 5-(CloudWatch to Slack Notification)

Welcome to Day 5 of 100 Days of DevOps, On Day 2 I discussed the SNS https://medium.com/@devopslearning/100-days-of-devops-day-2-introduction-to-simple-notification-service-sns-97137b2f1f1e and in the current DevOps world, no one denies the importance of Notification especially in cases when something went wrong with your infrastructure(Server Down/CPU Utilization high…).In this tutorial, I will show you how to integrate CloudWatch with Slack, so that you will be notified and take an effective measure.

![](https://cdn-images-1.medium.com/max/2040/1*QwPAPPp6ThOnAwUaDVcEBA.jpeg)

*This can be achieved by following a few steps*

* *Create an AWS Access key and Secret Key*

* *Create an IAM Role*

* *Deploy the lambda function*

* *Create an SNS topic*

* *Create a Cloudwatch Alarm*
> *Step1: Create an AWS Access key and Secret Key*

* *Go to IAM console [https://console.aws.amazon.com/iam/home?region=us-west-2#/home](https://console.aws.amazon.com/iam/home?region=us-west-2#/home)*

* *Click on Users → Particular user → Security Credentials*

* *Click on Create Access Key*

![](https://cdn-images-1.medium.com/max/4668/1*q0aNRZdFfkuwYkptldKa8Q.png)

* *Save this Access Key and Secret Key as we need them later while configuring Lambda function*
> *Step2: Create an IAM Role*

* *Go to IAM console [https://console.aws.amazon.com/iam/home?region=us-west-2#/home](https://console.aws.amazon.com/iam/home?region=us-west-2#/home)*

* *Roles → Create role → AWS service → Lambda*

![](https://cdn-images-1.medium.com/max/4056/1*qVUCJUMiZI7700PMJaMLMg.png)

* *Search for AWSLambdaBasicExecutionRole*

![](https://cdn-images-1.medium.com/max/4124/1*R37iF4MzAz9FlrtLMkoqQQ.png)

* *Give your Role a name*

![](https://cdn-images-1.medium.com/max/4728/1*S_b2we2mojjrKeSMZL4LRA.png)

* *Click on create Role*

* *Copy the role arn, we need it for future configuration*
> *Step3: Deploy the Lambda Function*

* *For the purpose of this demo, I am using Public GitHub Repo*
[**assertible/lambda-cloudwatch-slack**
*Send AWS CloudWatch notifications to a Slack channel using Lambda - assertible/lambda-cloudwatch-slack*github.com](https://github.com/assertible/lambda-cloudwatch-slack)

    *# Step 1
    $ git clone https://github.com/assertible/lambda-cloudwatch-slack.git*

    *Cloning into 'lambda-cloudwatch-slack'...*

    *remote: Enumerating objects: 244, done.*

    *remote: Total 244 (delta 0), reused 0 (delta 0), pack-reused 244*

    *Receiving objects: 100% (244/244), 668.48 KiB | 3.56 MiB/s, done.*

    *Resolving deltas: 100% (120/120), done.*

    *#Step 2
    $ cd lambda-cloudwatch-slack/*

    *#Step 3
    cp .env.example .env*

* *Now we need to perform some configuration at Slack End*

* *Go to Slack, Apps section and click on Add apps*

![](https://cdn-images-1.medium.com/max/2000/1*R-62pHPHiGHJn_pDjljw3w.png)

* *Search for incoming-webhook*

![](https://cdn-images-1.medium.com/max/2972/1*A6k6LKKlZNnlpWYW-OBhpA.png)

* *Enter the channel name where you want to send a notification, also note down Webhook URL*

![](https://cdn-images-1.medium.com/max/3964/1*5brSUIGepzEA9pfHHVZXlA.png)

* *Under .env file,enter the following info*

    *#ENCRYPTED_HOOK_URL= you can use ENCRYPTED_HOOK_URL if you want
    UNENCRYPTED_HOOK_URL=**Step3**
    AWS_FUNCTION_NAME=cloudwatch-to-slack
    AWS_REGION=us-west-2
    AWS_ROLE="**Step2**"
    
    # You can get AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY here: https://console.aws.amazon.com/iam/home#/users
    # Click on user -> Security credentials -> Access keys -> Create access key
    AWS_ACCESS_KEY_ID= **Step1**
    AWS_SECRET_ACCESS_KEY= **Step1***

* *After making these changes, execute the following command on the terminal*

    *$ npm install*

    *added 136 packages in 4.039s*

    *╭─────────────────────────────────────╮*

    *│                                     │*

    *│   Update available 5.6.0 → 5.8.0    │*

    *│     Run npm i -g npm to update      │*

    *│                                     │*

    *╰─────────────────────────────────────╯*

* *Finally, deploy it*

    *$ npm run deploy*

    *> lambda-cloudwatch-slack@0.3.0 deploy /Users/plakhera/Documents/lambda-cloudwatch-slack*

    *> ./scripts/deploy.sh*

    *Warning!!! You are building on a platform that is not 64-bit Linux (darwin.x64).*

    *If any of your Node dependencies include C-extensions, they may not work as expected in the Lambda environment.*

    *=> Moving files to temporary directory*

    *=> Running npm install --production*

    *=> Zipping deployment package*

    *=> Zipping repo. This might take up to 30 seconds*

    *=> Reading zip file to memory*

    *=> Reading event source file to memory*

    *=> Uploading zip file to AWS Lambda us-west-2 with parameters:*

    *{ FunctionName: 'cloudwatch-to-slack',*

    *Code: { ZipFile: <Buffer 50 4b 03 04 14 00 08 00 08 00 20 10 4a 4e 00 00 00 00 00 00 00 00 00 00 00 00 04 00 00 00 2e 65 6e 76 6d 90 5d 4f 83 30 18 85 ef f9 15 8d bb 5c 18 9b ... > },*

    *Handler: 'index.handler',*

    *Role: 'arn:aws:iam::XXXXXX:role/cloudwatch-to-lambda',*

    *Runtime: 'nodejs8.10',*

    *Description: 'Better Slack notifications for AWS CloudWatch',*

    *MemorySize: 128,*

    *Timeout: 60,*

    *Publish: false,*

    *VpcConfig: { SubnetIds: [], SecurityGroupIds: [] },*

    *Environment: { Variables: { UNENCRYPTED_HOOK_URL: 'https://hooks.slack.com/services/XXXXXXXX' } },*

    *KMSKeyArn: '',*

    *DeadLetterConfig: { TargetArn: null },*

    *TracingConfig: { Mode: null } }*

    *^C*

    *╭─────────────────────────────────────╮*

    *│                                     │*

    *│   Update available 5.6.0 → 6.7.0    │*

    *│     Run npm i -g npm to update      │*

    *│                                     │*

    *╰─────────────────────────────────────╯*

* *If everything looks good, you will see the new function on the lambda page*

![](https://cdn-images-1.medium.com/max/5192/1*aKQrPvo_qwEbaqdNUAq8Mw.png)
> *Step4: Create an SNS topic*

* *Go to [https://us-west-2.console.aws.amazon.com/sns/v2/home?region=us-west-2#/home](https://us-west-2.console.aws.amazon.com/sns/v2/home?region=us-west-2#/home)*

* *Click on create a topic and enter Topic name(eg: cloudwatch-to-lambda-sns-topic)*

![](https://cdn-images-1.medium.com/max/3256/1*7UEqQ5qzqW-MqpXCfU8LwQ.png)

* *Click on newly create a topic and then from Actions drop-down Subscribe to topic*

![](https://cdn-images-1.medium.com/max/2956/1*alMPTFPwOU94Ub8y3QV1YA.png)

* *Click on Create subscription, using AWS Lambda as Protocol*

![](https://cdn-images-1.medium.com/max/3256/1*z6UaSvQjN2wwS6a7SERmXg.png)
> *Step5: Create CloudWatch Alarm*

* *Go to CloudWatch home page [https://us-west-2.console.aws.amazon.com/cloudwatch/home?region=us-west-2](https://us-west-2.console.aws.amazon.com/cloudwatch/home?region=us-west-2)*

* *Alarms → Create Alarm → Metric → Select Metric → EC2 → Per Instance Metric*

* *Select CPU Utilization*

![](https://cdn-images-1.medium.com/max/5552/1*25P2oQVi0-c5RCrwF-moxA.png)

* *Fill all the necessary details*

![](https://cdn-images-1.medium.com/max/3932/1*kk60UOwqWsEpqherO-R1Rw.png)

* *In order to replicate the scenario, I am using stress tool, which is available as the part of RedHat epel repo*
[**EPEL - Fedora Project Wiki**
*The EPEL project strives to provide packages with both high quality and stability. However, EPEL is maintained by a…*fedoraproject.org](https://fedoraproject.org/wiki/EPEL)

    *# stress --cpu 10 --timeout 300*

    *stress: info: [15259] dispatching hogs: 10 cpu, 0 io, 0 vm, 0 hdd*

    *stress: info: [15259] successful run completed in 300s*

* *You will see the notification like this*

![](https://cdn-images-1.medium.com/max/3332/1*WhFX-XqWFinGZ1eT6sLb4w.png)

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
