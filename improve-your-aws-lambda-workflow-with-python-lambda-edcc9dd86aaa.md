
# Improve your AWS Lambda Workflow with python-lambda

Create and deploy AWS Lambda functions with ease in Python

![](https://cdn-images-1.medium.com/max/3000/1*HEojJ3WAcSlqJXsWS-lUTw.jpeg)

In our series about building [AWS APIs](https://hackersandslackers.com/tag/aws-api/), we’ve covered a lot of ground around learning the AWS ecosystem. Now that we’re all feeling a bit more comfortable, it may be time to let everybody in on the world’s worst-kept secret: Almost nobody builds architecture by interacting with the AWS UI directly. There are plenty of examples of how this is done, with the main example being **HashiCorp:** an entire business model based around the premise that AWS has a shitty UI, to the point where it’s easier to write code to make things that will host your code. What a world.

In the case of creating Python Lambda functions, the “official” (AKA manual) workflow of deploying your code to AWS is something horrible like this:

* You start a project locally and begin development.

* You opt to use **virtualenv, **because you’re well aware that you’re going to need the source for any packages you use available.

* When you’re ready to ‘deploy’ to AWS, you *copy all your dependencies from */site-packages *and move them into your root directory*, temporarily creating an abomination of a project structure.

* With your project fully bloated and confused, you cherry-pick the files needed to zip into an archive.

* Finally, you upload your code via zip either to Lambda directory or to S3, only to run your code, realize it's broken, and need to start all over.

## There Must Be a Better Way

Indeed there is, and surprisingly enough the solution is 100% Python (sorry HashiCorp, we’ll talk another time). This “better way” is my personal method of leveraging the following:

## Obligatory “Installing the CLI” Recap

First off, make sure you’re using a compatible version of Python on your system, as AWS is still stuck on version 3.6. Look, we can’t all be Google Cloud (and by the way, *Python 2.7 *doesn’t count as compatible — let it die before your career does).

    $ pip3 install awscli --upgrade --user

If you’re working off an EC2 instance, it has come to my attention pip3 does not come preinstalled. Remember to run:

* $ apt update

* $ apt upgrade

* $ apt install python3-pip

You may be prompted to run apt install awscli as well.

Awesome, now that we have the CLI installed on the *real* version of Python, we need to store your credentials. Your Access Key ID and Secret Access Key can be found in your IAM policy manager.

    $ aws configure 
    AWS Access Key ID [None]: YOURKEY76458454535 
    AWS Secret Access Key [None]: SECRETKEY*^R(*$76458397045609365493 $ Default region name [None]: 
    Default output format [None]:

On both Linux and OSX, this should generate files found under cd ~/.aws which will be referenced by default whenever you use an AWS service moving forward.

## Set Up Your Environment

As mentioned, we’ll use pipenv for easy environment management. We'll create an environment using Lambda's preferred Python version:

    $ pip3 install pipenv 
    $ pipenv shell --python 3.6 

    Creating a virtualenv for this project… 
    Pipfile: /home/example/Pipfile 
    Using /usr/bin/python3 (3.6.6) to create virtualenv… 
    ⠇Already using interpreter /usr/bin/python3 
    Using base prefix '/usr' 
    New python executable in /root/.local/share/virtualenvs/example-RnlD17kd/bin/python3 
    Also creating executable in /root/.local/share/virtualenvs/example-RnlD17kd/bin/python 
    Installing setuptools, pip, wheel...done.

Something you should be aware of at the time of writing: Pip’s latest version, 18.1, is actually a *breaking change* for Pipenv. Thus, the first thing we should do is force usage of pip 18.0 (is there even a fix for this yet?). This is solved by typing pip3 install pip==18.0 with the Pipenv shell activated. Now let's get to the easy part.

## python-lambda: The Savior of AWS

So far we’ve made our lives easier in two ways: we’re keeping our AWS credentials safe and far away from ourselves, and we have what is by far the superior Python package management solution. But this is all foreplay leading up to python-lambda:

    $ pip3 install python-lambda

This library alone is about to do you the following favors:

* Initiate your Lambda project structure for you.

* Isolate Lambda configuration to a* config.yaml* file, covering everything from the name of your entry point, handler function, and even program-specific variables.

* Allow you to run tests locally, where a *test.json* file simulates a request being made to your function locally.

* Build a production-ready zip file with all dependencies *completely separated *from your beautiful file structure.

* The ability to deploy *directly* to S3 or Lambda with said zip file from command-line.

Check out the commands for yourself:

    Commands: 
    build       Bundles package for deployment. 
    cleanup     Delete old versions of your functions 
    deploy      Register and deploy your code to lambda. 
    deploy-s3   Deploy your lambda via S3. 
    init        Create a new function for Lambda. 
    invoke      Run a local test of your function. 
    upload      Upload your lambda to S3.

## Initiate your project

Running lambda init will generate the following file structure:

    . 
    ├── Pipfile 
    ├── config.yaml 
    ├── event.json 
    └── service.py

## Checking out the entry point: service.py

python-lambda starts you off with a basic handler as an example of a working project. Feel free to rename service.py and its handler function to whatever you please, as we can configure that in a bit.

    # -*- coding: utf-8 -*- 

    def handler(event, context): 
    # Your code goes here! 
    e = event.get('e') 
    pi = event.get('pi') 
    return e + pi

## Easy configuration via configure.yaml

The base config generated by lambda init looks like this:

    region: us-east-1

    function_name: my_lambda_function
    handler: service.handler
    description: My first lambda function
    runtime: python3.6
    # role: lambda_basic_execution

    # S3 upload requires appropriate role with s3:PutObject permission
    # (ex. basic_s3_upload), a destination bucket, and the key prefix
    # bucket_name: 'example-bucket'
    # s3_key_prefix: 'path/to/file/'

    # if access key and secret are left blank, boto will use the credentials
    # defined in the [default] section of ~/.aws/credentials.
    aws_access_key_id:
    aws_secret_access_key:

    # dist_directory: dist
    # timeout: 15
    # memory_size: 512
    # concurrency: 500
    #

    # Experimental Environment variables
    environment_variables:
        env_1: foo
        env_2: baz

    # If `tags` is uncommented then tags will be set at creation or update
    # time.  During an update all other tags will be removed except the tags
    # listed here.
    #tags:
    #    tag_1: foo
    #    tag_2: bar

Look familiar? These are all the properties you would normally have to set up via the UI. As an added bonus, you can store values (such as S3 bucket names for boto3) in this file as well. That’s dope.

## Setting up event.json

The default event.json is about as simplistic as you can get, and naturally not very helpful at first (it isn't meant to be). These are the contents:

    { "pi": 3.14, "e": 2.718 }

We can replace this a real test JSON which we can grab from Lambda itself. Here’s an example of a Cloudwatch event we can use instead:

    {
      "id": "cdc73f9d-aea9-11e3-9d5a-835b769c0d9c",
      "detail-type": "Scheduled Event",
      "source": "aws.events",
      "account": "{{account-id}}",
      "time": "1970-01-01T00:00:00Z",
      "region": "us-east-1",
      "resources": [
        "arn:aws:events:us-east-1:123456789012:rule/ExampleRule"
      ],
      "pi": 3.14,
      "e": 2.718
      "detail": {}
    }

Remember that event.json is what is being passed to our handler as the event parameter. Thus, now we can run our Lambda function *locally* to see if it works:

    $ lambda invoke 
    5.8580000000000005

Pretty cool if you ask me.

After you express your coding genius, remember to output pip freeze > requirements.txt. **python-lambda** will use this as a reference for which packages need to be included. This is neat because we can use Pipenv and the benefits of the workflow it provides while still easily outputting what we need to deploy.

Because we already specified which Lambda we’re going to deploy to in config.yaml, we can deploy to that Lambda immediately. lambda deploy will use the zip upload method, whereas lambda deploy-s3 will store your source on S3.

If you’d like to deploy the function yourself, run with lambda build which will zip your source code *plus dependencies* neatly into a /*dist* directory. Suddenly we never have to compromise our project structure, and now we can easily source control our Lambdas by *.gitignoring *our build folders while hanging on to our Pipfiles.

Here’s to hoping you never need to deploy Lambdas using any other method ever again. Cheers.

*Originally published at [hackersandslackers.com](https://hackersandslackers.com/awsapi/improve-your-aws-lambda-workflow-with-python-lambda/) on November 8, 2018.*
