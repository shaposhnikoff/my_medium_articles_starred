
# AWS Lambda + Serverless Framework + Python — A Step By Step Tutorial — Part 2 “Using AWS KMS with…

Using serverless technologies is becoming more and more mainstream. Serverless may make your life easier in several contexts, however, you are always responsible for securing your code. As a developer, one of the things you need to know is how to store secrets safely. Just google “Github leaks”, and you will find how easy is finding logins, passwords and other sensitive information.

### Disclaimer

This content is part of / inspired by one of our online courses/training. We are offering up to 80% OFF on these materials, during the **Black Friday 2019**.

You can receive your discount [here](http://bf.eralabs.io).

![](https://cdn-images-1.medium.com/max/2000/1*B0qBLWIa0zP-zJcrenZN8w.png)

This is a series of blog posts about using AWS Lambda with the Serverless Framework. You can check previous similar blog posts like:
[**AWS Lambda + Serverless Framework + Python — A Step By Step Tutorial — Part 1 “Hello World”**
*I am creating a series of blog posts to help you develop, deploy and run (mostly) Python applications on AWS Lambda…*hackernoon.com](https://hackernoon.com/aws-lambda-serverless-framework-python-part-1-a-step-by-step-hello-world-4182202aba4a)
[**AWS Lambda + Serverless Framework + Python — A Step By Step Tutorial — Part 2 “Using AWS KMS with…**
*Using serverless technologies is becoming more and more mainstream. Serverless may make your life easier in several…*medium.com](https://medium.com/@eon01/aws-lambda-serverless-framework-python-a-step-by-step-tutorial-part-2-using-aws-kms-with-9bdad3381024)
[**AWS Lambda + Serverless Framework + Python — A Step By Step Tutorial — Part 3 “Sending Emails from…**
*One of the good things about AWS Lambda is that it integrates easily with many AWS services like AWS SES (Simple Email…*medium.com](https://medium.com/@eon01/aws-lambda-serverless-framework-python-a-step-by-step-tutorial-part-3-sending-emails-from-ad4119abca3c)

In this tutorial, you are going to learn how to use AWS services in order to deploy a serverless function without versioning your secret information.

You will learn how to use AWS KMS/SSM and invoke a secret from your Lambda code.

![](https://cdn-images-1.medium.com/max/2000/0*1Bf-_CTBDjQIRKjm)

Don’t forget to subscribe to [Shipped: An Independent Newsletter Focused On Serverless, Containers, FaaS & Other Interesting Stuff](http://joinshipped.com/) and take a look at [Practical AWS, a training concerned with the actual use of AWS rather than with theory & ideas](https://www.practicalaws.com/).
[**An Independent Newsletter Focused On Serverless, Containers, CaaS, FaaS & Other Interesting Stuff**
*Shipped is a weekly newsletter focused on technologies like serverless computing, containers, FaaS, CaaS and other…*joinshipped.com](http://joinshipped.com)

## Prerequisites

The first thing you need to do, especially if you are starting learning AWS Lambda and the Serverless Framework, is following the first tutorial.

If you don’t want to use your AWS default profile, you can use a different profile before starting this tutorial.

**Example**:

    export AWS_PROFILE=<profile_name>

You can also add the default AWS region.

    export AWS_PROFILE=<profile_name> && export AWS_REGION=<region_name>

Note that your profiles can be found in the file:

    $HOME/.aws/credentials

As said in the part 1 of this tutorial, if you don’t have a credential file, you can always configure one using:

    serverless config credentials --provider aws --key <key> --secret <secret>

**Example**:

    serverless config credentials --provider aws --key AKIAAAZE123RE6XVZEGFRBDA --secret azedv6P+yrf5yCXfGGqmHRgVjxt2gn2Azafdf1BXLXE4Z

If you already have a credential file, you can choose one of the profiles to use, each time you will execute a serverless command.

**Example**:

    serverless deploy --aws-profile <profile_name>

Let’s create a folder called using-kms and activate the virtual environment.

    virtualenv -p python3  using-kms
    cd using-kms && mkdir app

    . bin/activate
    cd app

### Initializing our Project

In the app folder, we need to create a serverless.yml file. This file is needed to configure how our application will behave.

We need also to create our Python function (called handler.py):

In order to do this, let’s execute this command:

    serverless create --template aws-python3 --name using-kms

This will create the handler file, the configuration file and the .gitignore file:

    .
    ├── .gitignore
    ├── handler.py
    └── serverless.yml

This is how our serverless.yml file looks like:

    provider:
      name: aws
      runtime: python3.6

    functions:
      hello:
        handler: handler.hello

This is how the handler.py file looks like:

    import json

    def hello(event, context):
        body = {
            "message": "Go Serverless v1.0! Your function executed successfully!",
            "input": event
        }

    response = {
            "statusCode": 200,
            "body": json.dumps(body)
        }

    return response

    # Use this code if you don't use the http event with the LAMBDA-PROXY
        # integration
        """
        return {
            "message": "Go Serverless v1.0! Your function executed successfully!",
            "event": event
        }
        """

### Deploying our Project

Once the template files are created, we have a working AWS Lambda function, we need to deploy it:

    export AWS_PROFILE="serverless"
    serverless deploy

**Note**: You need to change the profile name to use your own one.

The deployment output looks like this. You can see that our code is zipped and deployed to a S3 bucket before being deployed to Lambda.

    Serverless: Packaging service...
    Serverless: Excluding development dependencies...
    Serverless: Creating Stack...
    Serverless: Checking Stack create progress...
    .....
    Serverless: Stack create finished...
    Serverless: Uploading CloudFormation file to S3...
    Serverless: Uploading artifacts...
    Serverless: Uploading service .zip file to S3 (390 B)...
    Serverless: Validating template...
    Serverless: Updating Stack...
    Serverless: Checking Stack update progress...
    ...............
    Serverless: Stack update finished...
    Service Information
    service: using-kms
    stage: dev
    region: us-east-1
    stack: using-kms-dev
    api keys:
      None
    endpoints:
      None
    functions:
      hello: using-kms-dev-hello

The function is now deployed.

In the remaining parts, we will create secrets using AWS KMS, then call these secrets from our serverless code.

## Using AWS KMS/SSM

This is how AWS defines its service:
> # AWS Key Management Service (**AWS KMS**) is a service that combines secure, highly available hardware and software to provide a key management system scaled for the cloud. **AWS KMS** uses customer master keys (CMKs) to encrypt your **Amazon** S3 objects.

Let’s start by creating a key to use later:

    aws kms create-key

**Output**:

    {
        "KeyMetadata": {
            "KeyId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
            "KeyState": "Enabled",
            "AWSAccountId": "xxxxxxxxxxxx",
            "KeyUsage": "ENCRYPT_DECRYPT",
            "Origin": "AWS_KMS",
            "Arn": "arn:aws:kms:eu-west-1:xxxxxxxx:key/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
            "KeyManager": "CUSTOMER",
            "Enabled": true,
            "CreationDate": 1537517622.44,
            "Description": ""
        }
    }

This is an example of using AWS Secure Secrets Manager with the previously generated key:

    ssm put-parameter --name "my_password" --value "change_me" --type SecureString --key-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx

**Output**:

    {
        "Version": 1
    }
> # With AWS Systems Manager Parameter Store, you can create [Secure String parameters](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-paramstore-about.html#sysman-paramstore-securestring), which are parameters that have a plaintext parameter name and an encrypted parameter value. Parameter Store uses AWS KMS to encrypt and decrypt the parameter values of Secure String parameters

After creating the secret it will be encrypted using the generated key and we can view our secret using:

    aws ssm get-parameter --name "my_password"

**Output**:

    {
        "Parameter": {
            "Type": "SecureString",
            "ARN": "arn:aws:ssm:eu-west-1:998335703874:parameter/my_password",
            "Name": "my_password",
            "Value": "AQICAHie4+sV2HCKbGbbzSjCff1UOgpFydL+hrgLF9GuXFFl2AE+m1+xxxoCtfaRr1DacW+AAAAZzBlBgkqhkiGxxxBwagWDBWAgEAMFEGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMGY6rC+BXBcYSrYv+AgEQgCQjIrS3txpZOxxxcTpR7btbzFSMcyxEouMedexlXKKeBhvfQo=",
            "Version": 1,
            "LastModifiedDate": 1537518134.577
        }
    }

You can view the decrypted secret using:

    aws ssm get-parameter --name "my_password"  --with-decryption

**Output**:

    {
        "Parameter": {
            "Version": 1,
            "Type": "SecureString",
            "Name": "my_password",
            "LastModifiedDate": 1537518134.577,
            "Value": "change_me",
            "ARN": "arn:aws:ssm:eu-west-1:XXXXXXXXX:parameter/my_password"
        }
    }

Invoking Secrets from the Lambda Function

In order to use the secret that we created previously, we are going to edit the file handler.py in order to add this code:

    import json
    import boto3
    import os

    session = boto3.Session(
           region_name=os.environ['REGION_NAME'], 
           aws_access_key_id=os.environ['ACCESS_KEY'], 
           aws_secret_access_key=os.environ['SECRET_KEY']
          )

    ssm_client = session.client('ssm')

    def getSecret(event, context):

        my_password = ssm_client.get_parameter(Name='my_password', WithDecryption=True)

        return str(my_password)

We should, of course, have a valid serverless.yml file. This is mine for the above function:

    service: using-kms

    provider:
      name: aws
      runtime: python3.6
      stage: dev
            
      environment:
        REGION_NAME: 'eu-west-1'  
        ACCESS_KEY: 'XXXXXXXXXXXX'  
        SECRET_KEY: 'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'

    getSecret:
        handler: handler.getSecret
        description: This function will return a secret stored using KMS
        events:
          - http:
              path: get-secret
              method: get
              integration: lambda
              cors: true
              response:
                headers:
                  "Access-Control-Allow_Origin": "'*'"

(*Note that we are including the AWS key pair in our serverless.yml file which is not a good practice unless your files is not versioned and protected. We don’t prefer to complicate things for you right now but it could be a good practice to encrypt variables like ACCESS_KEY in the environment (build/test/prod/dev). This practice is more related to CI/CD and less to Serverless framework. Another way to configure your credential without using them in your yml file is using serverless config command:*serverless config credentials --provider aws --key <ACCESS KEY ID> --secret <SECRET KEY> )

We can now deploy and visit the generated URL:

    Serverless: Packaging service...
    Serverless: Excluding development dependencies...
    Serverless: Uploading CloudFormation file to S3...
    Serverless: Uploading artifacts...
    Serverless: Uploading service .zip file to S3 (393 B)...
    Serverless: Validating template...
    Serverless: Updating Stack...
    Serverless: Checking Stack update progress...
    .................................
    Serverless: Stack update finished...
    Service Information
    service: using-kms
    stage: dev
    region: us-east-1
    stack: using-kms-dev
    api keys:
      None
    endpoints:
      GET - [https://u33iac43v1.execute-api.us-east-1.amazonaws.com/dev/get-secret](https://u33iac43v1.execute-api.us-east-1.amazonaws.com/dev/get-secret)
    functions:
      getSecret: using-kms-dev-getSecret
    Serverless: Removing old service artifacts from S3...

As you can notice in the output above, this is the URL we can visit to get the secret:

    [https://u33iac43v1.execute-api.us-east-1.amazonaws.com/dev/get-secret](https://u33iac43v1.execute-api.us-east-1.amazonaws.com/dev/get-secret)

This is what our function return for now:

    "{'Parameter': {'Name': 'my_password', 'Type': 'SecureString', 'Value': 'change_me', 'Version': 1, 'LastModifiedDate': datetime.datetime(2018, 9, 21, 8, 22, 14, 577000, tzinfo=tzlocal()), 'ARN': 'arn:aws:ssm:eu-west-1:998335703874:parameter/my_password'}, 'ResponseMetadata': {'RequestId': '8b4aebde-a2b0-4d9b-a557-bf9e604ab547', 'HTTPStatusCode': 200, 'HTTPHeaders': {'x-amzn-requestid': '8b4aebde-a2b0-4d9b-a557-bf9e604ab547', 'content-type': 'application/x-amz-json-1.1', 'content-length': '191', 'date': 'Fri, 21 Sep 2018 09:32:13 GMT'}, 'RetryAttempts': 0}}"

In order to extract only the password value from this output, we can use

    return (my_password['Parameter']['Value'])

### Tailing Logs

Sometimes, you need to see the execution or the deployment logs. This is the command to tail the logs:

    sls logs -f getSecret --tail

### Deleting the Function

Using sls remove command, we can delete the deployed service from AWS Lambda.

    serverless remove

We can add other options like:

* --stage or -s The name of the stage in service.

* --region or -r The name of the region in stage.

* --verbose or -v Shows all stack events during deployment.

Note: Removing a service will also remove the S3 bucket.

## Connect Deeper

In [the first part of this tutorial,](https://hackernoon.com/aws-lambda-serverless-framework-python-part-1-a-step-by-step-hello-world-4182202aba4a) we have seen how to deploy our first Lambda function using the Serverless Framework.

This part (2) will help you to store and use secrets from your AWS Lambda function.

Stay in touch as the upcoming posts will go more in depth.

I am creating a series of blog posts to help you develop, deploy and run (mostly) Python applications on AWS Lambda using Serverless Framework.

You can find my other articles about the same topic but using other frameworks like [Creating a Serverless Uptime Monitor & Getting Alerted by SMS — Lambda, Zappa & Python](https://hackernoon.com/creating-a-serverless-uptime-monitor-getting-alerted-by-sms-lambda-zappa-python-flask-15c5fb31027) or [Creating a Serverless Python API Using AWS Lambda & Chalice](https://hackernoon.com/creating-a-serverless-python-api-using-aws-lambda-chalice-d321dc43ce2)

Don’t forget to subscribe to [Shipped: An Independent Newsletter Focused On Serverless, Containers, FaaS & Other Interesting Stuff](http://joinshipped.com/).

![](https://cdn-images-1.medium.com/max/3750/1*lpdjzNxQBaz1uFMNDZkSHA.png)

You may be interested in learning more about Lambda and other AWS service, so please take a look at [Practical AWS, a training concerned with the actual use of AWS rather than with theory & ideas](https://www.practicalaws.com/).

![](https://cdn-images-1.medium.com/max/2600/0*Kju8Dog2kjhAjAqG.png)
