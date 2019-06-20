Unknown markup type 10 { type: [33m10[39m, start: [33m29[39m, end: [33m91[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m30[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m88[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m32[39m, end: [33m49[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m57[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m179[39m, end: [33m184[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m212[39m, end: [33m226[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m6[39m, end: [33m16[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m17[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m7[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m11[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m8[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m12[39m, end: [33m14[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m9[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m13[39m, end: [33m15[39m }

# AWS Lambda + Serverless Framework + Python — A Step By Step Tutorial — Part 1 “Hello World”

I am creating a series of blog posts to help you develop, deploy and run (mostly) Python applications on AWS Lambda using Serverless Framework.

This is a series of blog posts about using AWS Lambda with the Serverless Framework. You can check previous similar blog posts like:
[**AWS Lambda + Serverless Framework + Python — A Step By Step Tutorial — Part 1 “Hello World”**
*I am creating a series of blog posts to help you develop, deploy and run (mostly) Python applications on AWS Lambda…*hackernoon.com](https://hackernoon.com/aws-lambda-serverless-framework-python-part-1-a-step-by-step-hello-world-4182202aba4a)
[**AWS Lambda + Serverless Framework + Python — A Step By Step Tutorial — Part 2 “Using AWS KMS with…**
*Using serverless technologies is becoming more and more mainstream. Serverless may make your life easier in several…*medium.com](https://medium.com/@eon01/aws-lambda-serverless-framework-python-a-step-by-step-tutorial-part-2-using-aws-kms-with-9bdad3381024)
[**AWS Lambda + Serverless Framework + Python — A Step By Step Tutorial — Part 3 “Sending Emails from…**
*One of the good things about AWS Lambda is that it integrates easily with many AWS services like AWS SES (Simple Email…*medium.com](https://medium.com/@eon01/aws-lambda-serverless-framework-python-a-step-by-step-tutorial-part-3-sending-emails-from-ad4119abca3c)

You can also find my other articles about the same topic but using other frameworks like [Creating a Serverless Uptime Monitor & Getting Alerted by SMS — Lambda, Zappa & Python](https://hackernoon.com/creating-a-serverless-uptime-monitor-getting-alerted-by-sms-lambda-zappa-python-flask-15c5fb31027) or [Creating a Serverless Python API Using AWS Lambda & Chalice](https://hackernoon.com/creating-a-serverless-python-api-using-aws-lambda-chalice-d321dc43ce2)

Don’t forget to subscribe to [Shipped: An Independent Newsletter Focused On Serverless, Containers, FaaS & Other Interesting Stuff](http://joinshipped.com/) and take a look at [Practical AWS, a training concerned with the actual use of AWS rather than with theory & ideas](https://www.practicalaws.com/).

**Let’s start!**

![](https://cdn-images-1.medium.com/max/NaN/1*OezhU9lHTNCk6O6FCUL5fQ.png)

### Install & Configure Python Virtual Environment

I’m using Ubuntu, so the installation steps could be used for Ubuntu, Debian and Ubuntu/Debian based distributions. For other OSs, you can find more information [here](https://stackoverflow.com/a/6587528), and in the [official website of Python](https://www.python.org/downloads/).

    sudo apt-get install python3
    sudo apt-get install python3-pip
    sudo pip3 install virtualenv
    mkdir -p serverless-python-examples/hello-world/app
    cd serverless-python-examples/hello-world

Create a virtual environment and activate it:

    virtualenv -p python3 venv
    . venv/bin/activate

Our working directory is

    serverless-python-examples/hello-world/app

### Create an AWS User for Your Application

The second step is signing in to your AWS console and creating a new user. (You can also use a user you already created) :

![](https://cdn-images-1.medium.com/max/3798/1*ctnt0PlCsqADriMYP91mdg.png)

Now add the permissions you need for the created user. In my case, I am going to choose the Administration permission (arn:aws:iam::aws:policy/AdministratorAccess) that provides full access to AWS services and resources.

**Note:** When thinking about security, the administrator access is not a good choice, you can refine your user permissions by choosing only the AWS permissions you need.

![](https://cdn-images-1.medium.com/max/3810/1*9sV1ZjmuwFORQws0cuLxAA.png)

Please keep in mind that you must download and keep the generated credentials in a safe place:

![](https://cdn-images-1.medium.com/max/3812/1*c7GzdmSwS7aeUaHgLdewLw.png)

For the sake of simplicity, I added a new profile to the file

    ~/.aws/credentials

with the name

    serverless

![](https://cdn-images-1.medium.com/max/2000/1*mVwju6xdWlXSCPdfJ1Eleg.png)

By adding the credentials to the AWS credentials file, I can use any AWS command using the desired profile.

Example:

    aws s3 cp my_file s3://my_bucket --profile serverless

### Installing and Configuring Serverless Framework

You can install Serverless using a single command:

    sudo npm install -g serverless

Let’s configure Serverless:

    serverless config credentials --provider aws --key <ACCESS KEY ID> --secret <SECRET KEY>

If you stored the key/secret to the credential files, we can use the different Serverless commands by designtating the profile at each command:

    serverless deploy --aws-profile serverless

We can also export the profile as well as the region to environement variables:

    export AWS_PROFILE="serverless" && export AWS_REGION=eu-west-1

### Initializing our Project

In the app folder, we need to create a serverless.yml file. This file is needed to configure how our application will behave.

We need also to create our Python function (called handler.py):

In order to do this, let’s execute this command:

    serverless create --template aws-python3 --name hello-world

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

Note: You need to change the profile name to use your own one.

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
    ................
    Serverless: Stack update finished...
    Service Information
    service: hello-world
    stage: dev
    region: us-east-1
    stack: hello-world-dev
    api keys:
      None
    endpoints:
      None
    functions:
      hello: hello-world-dev-hello

The function is now deployed.

### Adding Events to our Function

While you *do* already have a function defined called hello that's linked to a function handler, you currently have no way of triggering that function. That's why we need to add an event to the function defined in serverless.yml. Update it as follows:

We defined a function called hello but we did not define a way to trigger the function, this is when adding an “event” is useful. Edit the serverless.yml file:

    provider:
      name: aws
      runtime: python3.6

    functions:
      hello:
        handler: handler.hello
    **    events:
          - http:
              path: hello
              method: get**

According to our configuration, the function will be called using the /hello url.

Note: You can choose your own url.

Let’s deploy:

    serverless deploy -v

    Serverless: Packaging service...
    Serverless: Excluding development dependencies...
    Serverless: Uploading CloudFormation file to S3...
    Serverless: Uploading artifacts...
    Serverless: Uploading service .zip file to S3 (390 B)...
    Serverless: Validating template...
    Serverless: Updating Stack...
    Serverless: Checking Stack update progress...
    CloudFormation - UPDATE_IN_PROGRESS - AWS::CloudFormation::Stack - hello-world-dev
    CloudFormation - CREATE_IN_PROGRESS - AWS::ApiGateway::RestApi - ApiGatewayRestApi
    CloudFormation - UPDATE_IN_PROGRESS - AWS::Lambda::Function - HelloLambdaFunction
    CloudFormation - CREATE_IN_PROGRESS - AWS::ApiGateway::RestApi - ApiGatewayRestApi
    CloudFormation - UPDATE_COMPLETE - AWS::Lambda::Function - HelloLambdaFunction
    CloudFormation - CREATE_COMPLETE - AWS::ApiGateway::RestApi - ApiGatewayRestApi
    CloudFormation - CREATE_IN_PROGRESS - AWS::ApiGateway::Resource - ApiGatewayResourceHello
    CloudFormation - CREATE_IN_PROGRESS - AWS::Lambda::Permission - HelloLambdaPermissionApiGateway
    CloudFormation - CREATE_IN_PROGRESS - AWS::ApiGateway::Resource - ApiGatewayResourceHello
    CloudFormation - CREATE_IN_PROGRESS - AWS::Lambda::Permission - HelloLambdaPermissionApiGateway
    CloudFormation - CREATE_COMPLETE - AWS::ApiGateway::Resource - ApiGatewayResourceHello
    CloudFormation - CREATE_IN_PROGRESS - AWS::ApiGateway::Method - ApiGatewayMethodHelloGet
    CloudFormation - CREATE_IN_PROGRESS - AWS::ApiGateway::Method - ApiGatewayMethodHelloGet
    CloudFormation - CREATE_COMPLETE - AWS::ApiGateway::Method - ApiGatewayMethodHelloGet
    CloudFormation - CREATE_IN_PROGRESS - AWS::ApiGateway::Deployment - ApiGatewayDeployment1534113893559
    CloudFormation - CREATE_IN_PROGRESS - AWS::ApiGateway::Deployment - ApiGatewayDeployment1534113893559
    CloudFormation - CREATE_COMPLETE - AWS::ApiGateway::Deployment - ApiGatewayDeployment1534113893559
    CloudFormation - CREATE_COMPLETE - AWS::Lambda::Permission - HelloLambdaPermissionApiGateway
    CloudFormation - UPDATE_COMPLETE_CLEANUP_IN_PROGRESS - AWS::CloudFormation::Stack - hello-world-dev
    CloudFormation - UPDATE_COMPLETE - AWS::CloudFormation::Stack - hello-world-dev
    Serverless: Stack update finished...
    Service Information
    service: hello-world
    stage: dev
    region: us-east-1
    stack: hello-world-dev
    api keys:
      None
    endpoints:
      GET - [https://x7o0xwsbkd.execute-api.us-east-1.amazonaws.com/dev/hello](https://x7o0xwsbkd.execute-api.us-east-1.amazonaws.com/dev/hello)
    functions:
      hello: hello-world-dev-hello

    Stack Outputs
    HelloLambdaFunctionQualifiedArn: arn:aws:lambda:us-east-1:998335703874:function:hello-world-dev-hello:1
    ServiceEndpoint: [https://x7o0xwsbkd.execute-api.us-east-1.amazonaws.com/dev](https://x7o0xwsbkd.execute-api.us-east-1.amazonaws.com/dev)
    ServerlessDeploymentBucketName: hello-world-dev-serverlessdeploymentbucket-qqgrblrjprhu

As you may see in the deployment logs that the AWS lambda function has a new URL:

    [https://x7o0xwsbkd.execute-api.us-east-1.amazonaws.com/dev/hello](https://x7o0xwsbkd.execute-api.us-east-1.amazonaws.com/dev/hello)

We can test our function using the browser, tools like Postman, the CLI or Serverless invoke command.

**Using CURL:**

    curl -X GET [https://x7o0xwsbkd.execute-api.us-east-1.amazonaws.com/dev/hello](https://x7o0xwsbkd.execute-api.us-east-1.amazonaws.com/dev/hello)

Output:

    {
       "message":"Go Serverless v1.0! Your function executed successfully!",
       "input":{
          "resource":"/hello",
          "path":"/hello",
          "httpMethod":"GET",
          "headers":{
             "Accept":"*/*",
             "CloudFront-Forwarded-Proto":"https",
             "CloudFront-Is-Desktop-Viewer":"true",
             "CloudFront-Is-Mobile-Viewer":"false",
             "CloudFront-Is-SmartTV-Viewer":"false",
             "CloudFront-Is-Tablet-Viewer":"false",
             "CloudFront-Viewer-Country":"FR",
             "Host":"x7o0xwsbkd.execute-api.us-east-1.amazonaws.com",
             "User-Agent":"curl/7.47.0",
             "Via":"1.1 xxx.cloudfront.net (CloudFront)",
             "X-Amz-Cf-Id":"",
             "X-Amzn-Trace-Id":"Root=1-5b70bbe0-f318b862a8dfdb1e38fc337e",
             "X-Forwarded-For":"",
             "X-Forwarded-Port":"443",
             "X-Forwarded-Proto":"https"
          },
          "queryStringParameters":null,
          "pathParameters":null,
          "stageVariables":null,
          "requestContext":{
             "resourceId":"bsk6aq",
             "resourcePath":"/hello",
             "httpMethod":"GET",
             "extendedRequestId":"LiJLDHe3oAMF7kQ=",
             "requestTime":"12/Aug/2018:22:59:44 +0000",
             "path":"/dev/hello",
             "accountId":"xxx",
             "protocol":"HTTP/1.1",
             "stage":"dev",
             "requestTimeEpoch":1534114784336,
             "requestId":"6727b590-9e83-11e8-a9cc-61641c6c77ac",
             "identity":{
                "cognitoIdentityPoolId":null,
                "accountId":null,
                "cognitoIdentityId":null,
                "caller":null,
                "sourceIp":"",
                "accessKey":null,
                "cognitoAuthenticationType":null,
                "cognitoAuthenticationProvider":null,
                "userArn":null,
                "userAgent":"curl/7.47.0",
                "user":null
             },
             "apiId":"x7o0xwsbkd"
          },
          "body":null,
          "isBase64Encoded":false
       }
    }

**Using serverless invoke:**

    sls invoke -f hello

Output:

    {
        "statusCode": 200,
        "body": "{\"message\": \"Go Serverless v1.0! Your function executed successfully!\", \"input\": {}}"
    }

### Tailing Logs

Sometimes, you need to see the execution or the deployment logs. This is the command to tail the logs:

    sls logs -f hello --tail

### Deleting the Function

Using sls remove command, we can delete the deployed service from AWS Lambda.

    serverless remove

We can add other options like:

* --stage or -s The name of the stage in service.

* --region or -r The name of the region in stage.

* --verbose or -v Shows all stack events during deployment.

Note: Removing a service will also remove the S3 bucket.

## Connect Deeper

We have seen how to deploy our first Lambda function using the Serverless Framework. Stay in touch as the upcoming posts will go more in depth.

I am creating a series of blog posts to help you develop, deploy and run (mostly) Python applications on AWS Lambda using Serverless Framwork. You can find the 2nd part here:
[**AWS Lambda + Serverless Framework + Python — A Step By Step Tutorial — Part 2 “Using AWS KMS with…**
*Using serverless technologies is becoming more and more mainstream. Serverless may make your life easier in several…*medium.com](https://medium.com/@eon01/aws-lambda-serverless-framework-python-a-step-by-step-tutorial-part-2-using-aws-kms-with-9bdad3381024)

You can also find my other articles about the same topic but using other frameworks like [Creating a Serverless Uptime Monitor & Getting Alerted by SMS — Lambda, Zappa & Python](https://hackernoon.com/creating-a-serverless-uptime-monitor-getting-alerted-by-sms-lambda-zappa-python-flask-15c5fb31027) or [Creating a Serverless Python API Using AWS Lambda & Chalice](https://hackernoon.com/creating-a-serverless-python-api-using-aws-lambda-chalice-d321dc43ce2)

Don’t forget to subscribe to [Shipped: An Independent Newsletter Focused On Serverless, Containers, FaaS & Other Interesting Stuff](http://joinshipped.com/).

![](https://cdn-images-1.medium.com/max/3750/1*lpdjzNxQBaz1uFMNDZkSHA.png)

You may be interested in learning more about Lambda and other AWS service, so please take a look at [Practical AWS, a training concerned with the actual use of AWS rather than with theory & ideas](https://www.practicalaws.com/).

![](https://cdn-images-1.medium.com/max/2202/1*HobgTbb2pjbEz_fZ2QYAMg.png)
