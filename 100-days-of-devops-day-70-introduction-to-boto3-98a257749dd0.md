
# 100 Days of DevOps — Day 70-Introduction to Boto3

Welcome to Day 70 of 100 Days of DevOps, Focus for today is Boto3

*What is Boto3?*

*Boto3 is the Amazon Web Services (AWS) SDK for Python. It enables Python developers to create, configure, and manage AWS services, such as EC2 and S3. Boto3 provides an easy to use, object-oriented API, as well as low-level access to AWS services.*

*Boto3 is built on the top of a library called Botocore, which is shared by the AWS CLI. Botocore provides the low level clients, session and credentials and configuration data. Boto3 built on the top of Botocore by providing its own session, resources, and collections.*
[**boto/boto3**
*AWS SDK for Python. Contribute to boto/boto3 development by creating an account on GitHub.*github.com](https://github.com/boto/boto3)
[**boto/botocore**
*The low-level, core functionality of boto 3. Contribute to boto/botocore development by creating an account on GitHub.*github.com](https://github.com/boto/botocore)
> *Installing boto3*

    *$ pip3 install boto3 --user*

* *--user makes pip install packages in your home directory instead, which doesn't require any special privileges.*

    *$ aws configure*

    *AWS Access Key ID [****************XXXX]:*

    *AWS Secret Access Key [****************XXXX]:*

    *Default region name [us-west-2]:*

    *Default output format [json]:*

* *Configure your aws credentials*

* *To test it*

    *$ aws sts get-caller-identity*

    *{*

    *"Account": "1234567889",*

    *"UserId": "XXXXXXXXX",*

    *"Arn": "arn:aws:iam::1234567889:user/plakhera"*

    *}*
> Task1: List out all S3 buckets using Boto3

    *>>> import boto3*

    *# Create high level resource using boto3
    >>> s3 = boto3.resource('s3') *

    *# Print all the bucket name
    >>> for bucket in s3.buckets.all(): *

    *...     print(bucket.name)*

    *...*

    *test-awsmacietrail-dataevent*

* *Resources represent an object-oriented interface to AWS services. They provide a higher-level abstraction than the raw low level calls made by service clients.*
> Task2: Create an EC2 instance using Boto3

    ***import **boto3*

    *# Create a low-level client with the service name
    ec2 **= **boto3.client**('ec2')***

    *# When we invoke client we always got response back
    response **= **ec2.run_instances**(
        **ImageId**='ami-01ed306a12b7d1c96'**,
        InstanceType**='t2.micro'**,
        KeyName**='vpcflowlogs'**,
        MinCount**=**1,
        MaxCount**=**1
    **)
    print(**response**)***

* *Clients provide a low level interface to AWS whose methods map close to 1:1 with services API. All service operations are supported by clients. Clients are generated from a JSON service definition file.*
[**EC2 - Boto 3 Docs 1.9.133 documentation**
*Edit description*boto3.amazonaws.com](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ec2.html#EC2.Client.run_instances)

*GitHub Link*
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/tree/master/boto3)

*Reference*
[**Boto 3 Documentation - Boto 3 Docs 1.9.133 documentation**
*Boto is the Amazon Web Services (AWS) SDK for Python. It enables Python developers to create, configure, and manage AWS…*boto3.amazonaws.com](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
[**AWS SDK for Python**
*Get started quickly using AWS with boto3, the AWS SDK for Python. Boto3 makes it easy to integrate your Python…*aws.amazon.com](https://aws.amazon.com/sdk-for-python/)

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
