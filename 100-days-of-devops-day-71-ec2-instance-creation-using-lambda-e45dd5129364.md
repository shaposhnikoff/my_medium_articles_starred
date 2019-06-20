
# 100 Days of DevOps — Day 71-EC2 Instance creation using Lambda

Welcome to Day 71 of 100 Days of DevOps, Focus for today is EC2 Instance creation using Lambda

*On Day69 and Day70, I blogged about Lambda and Boto3, let extend this concept further by Creating EC2 instance using Lambda and Boto3*
[**100 Days of DevOps — Day 69-Introduction to AWS Lambda**
*Welcome to Day 69 of 100 Days of DevOps, Focus for today is Introduction to AWS Lambda*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-69-introduction-to-aws-lambda-6ac6dfbd6fb8)
[**100 Days of DevOps — Day 70-Introduction to Boto3**
*Welcome to Day 70 of 100 Days of DevOps, Focus for today is Boto3*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-70-introduction-to-boto3-98a257749dd0)

*Go to Lambda Console [https://console.aws.amazon.com/lambda](https://console.aws.amazon.com/lambda) → Create a function*

![](https://cdn-images-1.medium.com/max/5408/1*2hcoaFkmHoH4WtOCFF1O3g.png)

    ** Choose Author from scratch
    * Function name: Give your function some name(eg: EC2InstanceCreation)
    * Runtime: Choose Python3.7 as runtime
    * Permission: Choose or create an execution role and then from drop down--> "Create a new role with basic Lambda permissions"*

* *IAM Role that has been created only gives Permission for creating and putting CloudWatch logs*

    *{
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "logs:CreateLogGroup",
                "Resource": "arn:aws:logs:us-east-1:635034346210:*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "logs:CreateLogStream",
                    "logs:PutLogEvents"
                ],
                "Resource": [
                    "arn:aws:logs:us-east-1:635034346210:log-group:/aws/lambda/mytest:*"
                ]
            }
        ]
    }*

* *So as we are planning to create EC2 instance, this IAM role requires little modification*

    *{
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "logs:CreateLogGroup",
                "Resource": "arn:aws:logs:us-east-1:635034346210:*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "logs:CreateLogStream",
                    "logs:PutLogEvents"
                ],
                "Resource": [
                    "arn:aws:logs:us-east-1:635034346210:log-group:/aws/lambda/mytest:*"
                ]
            },
            {
                "Effect": "Allow",
                "Action": [
                    "ec2:RunInstances"
                ],
                "Resource": [
                    "*"
                ]
            }
        ]
    }*

* *Lambda code will look like this*

    *import os
    import boto3*

    *AMI = os.environ['AMI']
    INSTANCE_TYPE = os.environ['INSTANCE_TYPE']
    KEY_NAME = os.environ['KEY_NAME']
    SUBNET_ID = os.environ['SUBNET_ID']*

    *ec2 = boto3.resource('ec2')*

    *def lambda_handler(event, context):*

    *instance = ec2.create_instances(
            ImageId=AMI,
            InstanceType=INSTANCE_TYPE,
            KeyName=KEY_NAME,
            SubnetId=SUBNET_ID,
            MaxCount=1,
            MinCount=1
        )*

    *print("New instance created:", instance[0].id)*

* *To make it more modular, I setup environment variable*

![](https://cdn-images-1.medium.com/max/3956/1*_b-Wsr8bYej0pfwDp08f2w.png)

* *To test it, click on Configure test event and remove all the content so that it will look like { }*

![](https://cdn-images-1.medium.com/max/2664/1*ejyXuofuYAHxPzNxMlMkHg.png)

* *Check the execution logs and you will see something like this*

![](https://cdn-images-1.medium.com/max/5336/1*H3JQxasPgEZjBJkktNHF0A.png)

* *If you go to the EC2 console, you will see something like this*

![](https://cdn-images-1.medium.com/max/5052/1*g12lxboBvDstJHjyqbY4LQ.png)

* *Congrats, you have created your first EC2 instance using Lambda*

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
