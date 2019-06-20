
# 100 Days of DevOps — Day 8-Introduction to AWS Security Token Service(STS)

Welcome to Day 8 of 100 Days of DevOps, Let shift gears from monitoring to IAM and start with Security Token Service(STS)

*Problem: Rather than hardcode the value of access and secret access keys inside my instance to access any aws service, I want to use some automated ways so that keys should be rotated automatically and there is no way credentials would be compromised.*

*Solution: AWS Security Token Service(STS)*

*AWS Security Token Service(STS) that enables you to request temporary, limited privilege credentials for IAM Users or Federated Users).*

![](https://cdn-images-1.medium.com/max/2520/1*RNmKzxQesaeBs_oWan7kWg.jpeg)

*Benefits*

* *No need to embed token in the code*

* *Limited Lifetime(15min — 1 and 1/2 day)*

*Use Cases*

* *Identity Federation(Enterprise Identity Federation[Active Directory/ADFS]/ Web Identity Federation (Google, Facebook))*

* *Cross-account access(For Organization with multiple AWS accounts)*

* *Applications on Amazon EC2 Instances*

*Let see this in action*
> ***Step1***

* *Create an IAM user*

    *Go to AWS Console → Security, Identity, & Compliance → IAM → Users → Add user*

![](https://cdn-images-1.medium.com/max/3264/1*t3krDgT3iy2rY4ar8R0R7Q.png)

    ** User name: Please give some meaningful name
    * Access type: Only give this user Programmatic access*

* *In the next step **don’t add this user to any group or attach any existing policy***

* *Keep everything default, Review and Create user*
> ***Step2***

* *Create Roles*

* *Choose Another AWS account*

![](https://cdn-images-1.medium.com/max/3436/1*OU4hTp08sthIjY9wVN7NGg.png)

* *Attach a Policy(AmazonS3ReadOnlyAccess)*

![](https://cdn-images-1.medium.com/max/3256/1*0c5CtMRHFSf3yWrlKQx5lA.png)

* *Review and create role*

![](https://cdn-images-1.medium.com/max/3576/1*-hMCtYgYSO1b3LmY_gUS3w.png)
> ***Step3:***

* *Update/Modify Trust Relationships*

*Go to the Role we have just created and Click on Second Tab Trust relationships*

![](https://cdn-images-1.medium.com/max/4508/1*Aoe7BfUVCrBfI0oKhkbxqg.png)

    *{
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "AWS": "arn:aws:iam::XXXXXX:user/myteststsuser"
          },
          "Action": "sts:AssumeRole",
          "Condition": {}
        }
      ]
    }*

* *The current trust relation only allow root account to assume this role*

* *Modify it with the arn of the user(myteststsuser) we have just created*
> *Step4*

*Add inline policy to the user we have created*

![](https://cdn-images-1.medium.com/max/5064/1*n8iRp-iDXE4nsXgNAi0L4w.png)

![](https://cdn-images-1.medium.com/max/3864/1*I9wZhYKOmya-1dLPT3jttg.png)

    *Service: STS
    Action: AssumeRole
    Resource: ARN of the role we created earlier*

* *This is making our user assume the role*
> *Step5:*

*Testing*

    *$ aws configure --profile ststestprofile*

    *AWS Access Key ID [None]: XXXXXXXX*

    *AWS Secret Access Key [None]: XXXXXX*

    *Default region name [None]: us-west-2*

    *Default output format [None]: json*

*Also, export this profile for the time being*

    *$ export AWS_PROFILE=ststestprofile*

*As we set the user to assume Role, let generate the temporary credentails and security token by running the below mentioned command*

    *$ aws sts assume-role --role-arn arn:aws:iam::XXXXXX:role/sts-s3-read-only --role-session-name "mytestsession"*

    *{*

    *"AssumedRoleUser": {*

    *"AssumedRoleId": "XXXXXXX:mytestsession",*

    *"Arn": "arn:aws:sts::XXXXXXX:assumed-role/sts-s3-read-only/mytestsession"*

    *},*

    *"Credentials": {*

    *"SecretAccessKey": "XXXXXXX",*

    *"SessionToken": "XXXXXXX",*

    ***"Expiration": "2018-12-18T06:47:21Z",***

    *"AccessKeyId": "XXXXXXXXX"*

    *}*

    *}*

*and then export it*

    *export AWS_ACCESS_KEY_ID="XXXXXXX"
    export AWS_SECRET_ACCESS_KEY="XXXXXXX"
    export AWS_SECURITY_TOKEN="XXXXXXX"*

*Try to access S3 bucket*

    *$ aws s3 ls*

    *2018-12-13 20:53:05 mytestXXXXXX*

OR

    $ aws s3 cp bucketest s3://mytest*XXXXXX*

    upload failed: ./bucketest to s3://mytest*XXXXXX*/bucketest An error occurred (AccessDenied) when calling the PutObject operation: Access Denied

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
