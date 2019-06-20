
# AWS Lambda + Serverless Framework + Python — A Step By Step Tutorial — Part 3 “Sending Emails from…

One of the good things about AWS Lambda is that it integrates easily with many AWS services like AWS SES (Simple Email Service). This may help you to develop faster applications without even managing infrastructure (at least in the traditional way).. but beware of cloud lock!

This is a series of blog posts about using AWS Lambda with the Serverless Framework. You can check previous similar blog posts like
[**AWS Lambda + Serverless Framework + Python — A Step By Step Tutorial — Part 1 “Hello World”**
*I am creating a series of blog posts to help you develop, deploy and run (mostly) Python applications on AWS Lambda…*hackernoon.com](https://hackernoon.com/aws-lambda-serverless-framework-python-part-1-a-step-by-step-hello-world-4182202aba4a)

Or
[**AWS Lambda + Serverless Framework + Python — A Step By Step Tutorial — Part 2 “Using AWS KMS with…**
*Using serverless technologies is becoming more and more mainstream. Serverless may make your life easier in several…*medium.com](https://medium.com/@eon01/aws-lambda-serverless-framework-python-a-step-by-step-tutorial-part-2-using-aws-kms-with-9bdad3381024)

If you are interested in Serverless computing using AWS lambda, you may be also interested in learning other frameworks like Zappa and Chalice. I invite to give them a try:
[**Creating a Serverless Uptime Monitor & Getting Alerted by SMS — Lambda, Zappa & Python**
*My last article was about Creating a Serverless Python API Using AWS Lambda & Chalice. After posting it to Reddit, I…*hackernoon.com](https://hackernoon.com/creating-a-serverless-uptime-monitor-getting-alerted-by-sms-lambda-zappa-python-flask-15c5fb31027)
[**Creating a Serverless Python API Using AWS Lambda & Chalice**
*Chalice is a python serverless microframework for AWS, created by Amazon Web Services.*hackernoon.com](https://hackernoon.com/creating-a-serverless-python-api-using-aws-lambda-chalice-d321dc43ce2)

## Preuesities

The first thing you need to do, especially if you are starting learning AWS Lambda and the Serverless Framework, is following the first tutorial.

If you don’t want to use your AWS default profile, you can use a different profile before starting this tutorial.

Example:

    export AWS_PROFILE=<profile_name>

You can also add the default AWS region.

    export AWS_PROFILE=<profile_name> && export AWS_REGION=<region_name>

Note that your profiles can be found in the file:

    $HOME/.aws/credentials

As said in [part 1](https://hackernoon.com/aws-lambda-serverless-framework-python-part-1-a-step-by-step-hello-world-4182202aba4a) of this tutorial, if you don’t have a credential file, you can always configure one using:

    serverless config credentials --provider aws --key <key> --secret <secret>

Example:

    serverless config credentials --provider aws --key AKIAAAZE123RE6XVZEGFRBDA --secret azedv6P+yrf5yCXfGGqmHRgVjxt2gn2Azafdf1BXLXE4Z

If you already have a credential file, you can choose one of the profiles to use, each time you will execute a serverless command.

Example:

    serverless deploy --aws-profile <profile_name>

Let’s create a folder called using-ses and activate the virtual environment.

    virtualenv -p python3  using-ses
    cd using-ses && mkdir app

    . bin/activate
    cd app

![](https://cdn-images-1.medium.com/max/3000/0*d9StXsLXPoYied5L.png)

### Initializing our Project

In the app folder, we need to create a serverless.yml file. This file is needed to configure how our application will behave.

We need also to create our Python function (called handler.py):

In order to do this, let’s execute this command (inside the app folder):

    serverless create --template aws-python3 --name using-ses

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

    export AWS_PROFILE="<your_profile_name>"
    serverless deploy

Note: You need to change the profile name to use your own one.

You will a similar output on your terminal screen:

    Serverless: Generating boilerplate...
     _______                             __
    |   _   .-----.----.--.--.-----.----|  .-----.-----.-----.
    |   |___|  -__|   _|  |  |  -__|   _|  |  -__|__ --|__ --|
    |____   |_____|__|  \___/|_____|__| |__|_____|_____|_____|
    |   |   |             The Serverless Application Framework
    |       |                           serverless.com, v1.32.0
     -------'

    Serverless: Successfully generated boilerplate for template: "aws-python3"

The deployment output looks like this. You can see that our code is zipped and deployed to an S3 bucket before being deployed to Lambda.

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
    service: using-ses
    stage: dev
    region: us-east-1
    stack: using-ses-dev
    api keys:
      None
    endpoints:
      None
    functions:
      hello: using-ses-dev-hello

The function is now deployed.

In the remaining parts, we will create secrets using AWS SES, then send emails from our serverless code.

Serverless Configuration

Let’s start with some basic configurations.

In this tutorial, since we are using Python 3, we will use the library Boto3 in order to interact with AWS services, including SES.

We need to modify our serverless.yml file — just erase everything you find in the file to add this code:

    service: using-kms # NOTE: update this with your service name

    provider:
      name: aws
      runtime: python3.6
      stage: dev
      region: 'eu-west-1'
    
            
      environment:
        REGION_NAME: 'eu-west-1'  
        ACCESS_KEY: 'xxxxxxxxxxxxxxxxxxxxx'  
        SECRET_KEY: 'xxxxxxxxxxx/xxxx+xxxxxxxxxxxxxx+xxxxxxxxxxx'

In the above code, we set up the region name as well as our key pair as environment variables. This is a good practice since we can have multiple environments with multiple environment variables. The other good thing about this is that you will not hard code variables like your key pair.

You should, of course, change your preferred region as well as your key pair(access and secret key).

We need also to define our function in the same file:

    functions:
      sendEmail:
        handler: handler.sendEmail
        description: This function will send an email
        events:
          - http:
              path: send-email
              method: post
              integration: lambda
              cors: true
              response:
                headers:
                  "Access-Control-Allow_Origin": "'*'"

* The path will be used in the URL to call the function.

* The method is POST

* We can optionally define the headers of the response and the CORS

## AWS SES Integration

AWS describes its service as follow:
> # Amazon Simple Email Service (Amazon SES) is a cloud-based email sending service designed to help digital marketers and application developers send marketing, notification, and transactional emails. It is a reliable, cost-effective service for businesses of all sizes that use email to keep in contact with their customers.

We are going to use Boto3 in order to send emails from SES service.

This is how we usually use SES:

    client = boto3.client('ses' )    
            
        response = client.send_email(
            Destination={
                'ToAddresses': [destination]
                },
            Message={
                'Body': {
                    'Text': {
                        'Charset': 'UTF-8',
                        'Data': _message,
                    },
                },
                'Subject': {
                    'Charset': 'UTF-8',
                    'Data': subject,
                },
            },
            Source=source,

    )

We are going to read the message from the data send using a POST:

    data = event['body']

    name = data ['name']    
        source = data['source']    
        subject = data['subject']
        message = data['message']    
        destination = data['destination']

    _message = "Message from: " + name + "\nEmail: " + source + "\nMessage content: " + message

Our code will look like this:

    import json
    import boto3
    import os

    region_name = os.environ['REGION_NAME']
    aws_access_key_id = os.environ['SECRET_KEY']
    aws_secret_access_key = os.environ['SECRET_KEY']

    def sendEmail(event, context):
        data = event['body']

    name = data ['name']    
        source = data['source']    
        subject = data['subject']
        message = data['message']    
        destination = data['destination']

    _message = "Message from: " + name + "\nEmail: " + source + "\nMessage content: " + message    
        
        client = boto3.client('ses' )    
            
        response = client.send_email(
            Destination={
                'ToAddresses': [destination]
                },
            Message={
                'Body': {
                    'Text': {
                        'Charset': 'UTF-8',
                        'Data': _message,
                    },
                },
                'Subject': {
                    'Charset': 'UTF-8',
                    'Data': subject,
                },
            },
            Source=source,

    )
        return _message + str(region_name)

If we want to test it, we need to first deploy it:

    sls deploy

Then we can use a simple CURL command to send the POST data and send the email:

    curl -X POST \
    -d "name=aymen" \
    -d "[source=hello@aymenelamri.com](mailto:source=hello@aymenelamri.com)" \
    -d "subject=This is me" \
    -d "[destination=aymen@eralabs.io](mailto:destination=aymen@eralabs.io)" \
    -d "message=This is my message" \
    [https://w7iht695k9.execute-api.eu-west-1.amazonaws.com/dev/send-email](https://w7iht695k9.execute-api.eu-west-1.amazonaws.com/dev/send-email)

And voilà.

## Managing Permissions

In reality, the example above lacks something. If you followed this tutorial from the beginning, and execute every command, you will notice that you will have a similar error to the following one:

    An error occurred (AccessDenied) when calling the SendEmail operation: User `arn:aws:sts::xxxxxx:assumed-role/' is not authorized to perform `ses:SendEmail' on resource `arn:aws:ses:us-east-1:xxxxxxxxx:[identity/](mailto:identity/aymen@erlabs.io)xxxxxxxxxx'

It’s simply because you don’t have the right to do so.

These first thing you need to do is verifying your email using the AWS SES Console:

![](https://cdn-images-1.medium.com/max/3812/1*1TQzzjx36yCwzoVlbc8BeA.png)

The second thing, you should check is the approval of AWS to use SES and send emails to any email. If you are waiting for the approval, you can still send and receive (at the same time) emails using your verified email.

The third that we should add s permissions to the serverless.yml file. Our application should be granted access to SES:

    iamRoleStatements:
        - Effect: Allow
          Action:
            - ses:SendEmail
            - ses:SendRawEmail
          Resource: "*"

The serverless.yml file will look like this:

    service: using-kms # NOTE: update this with your service name

    provider:
      name: aws
      runtime: python3.6
      stage: dev
      region: 'eu-west-1'
      iamRoleStatements:
        - Effect: Allow
          Action:
            - ses:SendEmail
            - ses:SendRawEmail
          Resource: "*"        
      environment:
        REGION_NAME: 'eu-west-1'  
        ACCESS_KEY: 'xxxxxxxxxxxx'  
        SECRET_KEY: 'xxxxxxxxxxx/xxx+xxxxxxxxx+xxxxxxx'

    functions:
      sendEmail:
        handler: handler.sendEmail
        description: This function will send an email
        events:
          - http:
              path: send-email
              method: post
              integration: lambda
              cors: true
              response:
                headers:
                  "Access-Control-Allow_Origin": "'*'"

Now deploy and test:

    sls deploy

    curl -X POST -d "name=aymen" -d "[source=hello@aymenelamri.com](mailto:source=hello@aymenelamri.com)" -d "subject=This is me" -d "[destination=aymen@eralabs.io](mailto:destination=aymen@eralabs.io)" -d "message=this is my message" [https://w7iht695k9.execute-api.eu-west-1.amazonaws.com/dev/send-email](https://w7iht695k9.execute-api.eu-west-1.amazonaws.com/dev/send-email)

Don’t forget to change the sender and receiver emails with the good ones for you.

![](https://cdn-images-1.medium.com/max/2000/1*w59zIeH1-IaOc1qsvQy5GA.png)

I highly recommend that you setup DKIM, SPF & DMARC for your SES account. You can follow my guide:
[**Why your Marketing Email will Land in my Spam Folder ? The Non-marketing Guide: DKIM, SPF & DMARC**
*Investing time and money then land in your customer spam folder just like a spammer ? Not a good idea !*startupsventurecapital.com](https://startupsventurecapital.com/why-your-marketing-email-will-land-in-my-spam-folder-the-non-marketing-guide-dkim-spf-dmarc-43f91f9c6940)

### Tailing Logs

Sometimes, you need to see the execution or the deployment logs. This is the command to tail the logs:

    sls logs -f sendEmail --tail

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

[The part 2 ](https://medium.com/@eon01/aws-lambda-serverless-framework-python-a-step-by-step-tutorial-part-2-using-aws-kms-with-9bdad3381024)will help you if you want to store and use secrets from your AWS Lambda function.

In this part, we saw how we can integrate AWS Lambda with other services like SES and send emails. This can be done using Boto3 and Serverless Framework.

Stay in touch as the upcoming posts will go more in depth.

I am creating a series of blog posts to help you develop, deploy and run (mostly) Python applications on AWS Lambda using Serverless Framework.

You can find my other articles about the same topic but using other frameworks like [Creating a Serverless Uptime Monitor & Getting Alerted by SMS — Lambda, Zappa & Python](https://hackernoon.com/creating-a-serverless-uptime-monitor-getting-alerted-by-sms-lambda-zappa-python-flask-15c5fb31027) or [Creating a Serverless Python API Using AWS Lambda & Chalice](https://hackernoon.com/creating-a-serverless-python-api-using-aws-lambda-chalice-d321dc43ce2)

Don’t forget to subscribe to [Shipped: An Independent Newsletter Focused On Serverless, Containers, FaaS & Other Interesting Stuff](http://joinshipped.com/).

![](https://cdn-images-1.medium.com/max/3750/1*lpdjzNxQBaz1uFMNDZkSHA.png)

You may be interested in learning more about Lambda and other AWS service, so please take a look at [Practical AWS, a training concerned with the actual use of AWS rather than with theory & ideas](https://www.practicalaws.com/).

![](https://cdn-images-1.medium.com/max/2600/0*Kju8Dog2kjhAjAqG.png)
