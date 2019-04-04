
# One-click Deployment to AWS with CodeDeploy and BitBucket



One-click deployment, it’s the developer dream. Write some code, then test code and finally deploy. More often than not the deployment can become the most tricky party. But there are great new tools out there and with some DevOps the one-click deploy can be a reality. In this article we’re going to cover using Amazon Web Services (AWS) CodeDeploy along with Bitbucket to one-click deploy. Let’s get to it.

Amazon Web Services (AWS) is by the far the most used cloud hosting services today. They offer a wide range of products for just about anything you need. In this article we’re going to focus on [CodeDeploy](https://aws.amazon.com/codedeploy/) which is used for automated deployment. We’re going to integrate that with Atlassian [BitBucket](https://bitbucket.org/) and in the end we’ll be able to one-click deploy any commit to our ec2 instances on AWS.

**Pre-requisites
**To get started you will need to get an account at AWS and Bitbucket. Bitbucket is free for private repositories and AWS has a free tier plan. Once you have your accounts created then you are ready to go.

**Create Sample Repository
**First lets create a sample application in Bitbucket. You can call it anything you want and I would suggest codedeploy-sample. Once it’s created we’ll need some files. Let’s use the Flask hello world example for a simple Python wsgi application. Create a file called wsgi.py and paste the following code.

    from app.app import app as application

Create a folder name app and inside the folder a file name app.py and paste the following code.

    from flask import Flask
    app = Flask(__name__)
    
    @app.route('/')
    def hello_world():
        return 'Hello, World!'

**Bitbucket CodeDeploy Integration
**Now, let’s add CodeDeploy integration to BitBucket. Click your picture on the top-right corner of your Bitbucket account and choose integrations. Search for **AWS CodeDeploy for Bitbucket** and click add. Then click Grant Access to proceed.

**AWS EC2
**Now we need to have an EC2 instance with CodeDeploy agent installed that we can deploy our code. Create a new EC2 instance using Ubuntu or Amazon Linux and be sure you have SSH access. We need to verify that we have CodeDeploy agent running on this instance. Login to your instance via SSH and run the following, sudo service codedeploy-agent status. If you get an error back then the agent is not installed. Follow these instructions for [Amazon Linux](http://docs.aws.amazon.com/codedeploy/latest/userguide/how-to-run-agent-install.html#how-to-run-agent-install-linux) or [Ubuntu](http://docs.aws.amazon.com/codedeploy/latest/userguide/how-to-run-agent-install.html#how-to-run-agent-install-ubuntu) and confirm the agent is running.

**AWS CodeDeploy Application
**It’s time to create the CodeDeploy application. CodeDeploy will push code to your EC2 instances via EC2 tags or Auto Scaling groups. We’re going to use EC2 tags in our case. Go to your EC2 instanace in AWS console and create a tag with key: *CodeDeploy* and value: *sample-app* for the instance we created.

Navigate to AWS CodeDeploy in the console create a new application. Here we need an Application name and we can use *sample-app*. We also need a Deployment group name which in real world application would be something like testing, staging or production. In our case let’s just use *dev* for now. In the Add instances section use the key and value we created earlier for our EC2 instance tag. Under the Service role choose the *codedeploy-service-role* we create in the next section and Create Application.

**AWS Roles and Policy
**First thing we’ll need to do is setup two IAM roles in AWS. The first is a CodeDeploy Service Role and it will allow CodeDeploy to perform actions on your behalf.

1. Navigate to IAM and choose *Roles* on the left side and *Create New Role*.

1. Choose codedeploy-service-role for the name and hit next.

1. For *Role Type* scroll down and find select *AWS CodeDeploy*.

1. Check the *AWSCodeDeployRole* policy and click next step followed by Create Role.

Now, we’ll need to create a Policy and Role for Bitbucket which will access S3 to place the files and ability to start CodeDeploy application. Let’s create the policy first.

1. Navigate to IAM and choose *Policies* then *Create Policy*.

1. Select Create Your Own Policy

1. Choose a name for your policy (e.g. *BitbucketCodedeploy*) and past the following in the policy document section and create policy.

    {
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Action": ["s3:ListAllMyBuckets", "s3:PutObject"],
            "Resource": "arn:aws:s3:::*"
        }, {
            "Effect": "Allow",
            "Action": ["codedeploy:*"],
            "Resource": "*"
        }]
    }

Now let’s create the Bitbucket Role.

1. In IAM create another role and name it *BitbucketCodeDeployAddon*

1. In *Role Type* pick *Role for Cross-Account Access* and select *Provide access between your AWS account and a 3rd party AWS account*.

1. Here you will need the Account ID and External ID which we’ll need to get from Bitbucket.

1. Navigate to Bitbucket and the sample application we created earlier.

1. Go to Setting on the bottom left and choose *CodeDeploy Settings*.

1. On this page you will find the Account ID and External ID.

1. Enter the Account ID and External ID in AWS and hit next

1. Select the *BitbucketCodedeploy* policy we created earlier and create role.

Once the role is created, copy the Role ARN and go back to Bitbucket CodeDeploy settings and enter the information. You’ll also need to create an S3 bucket that Bitbucket will use to push your files. Navigate to AWS S3 and create a bucket then use that bucket in the Bibucket settings. For the CodeDeploy application use the sample-app that we created earlier.

**appspec.yml
**The way CodeDeploy understands what to do with your deployment is through the *appspec.yml* file in the root folder of your application. The appspect file has a files, permissions and hooks section. You can read more details about the file reference [here](http://docs.aws.amazon.com/codedeploy/latest/userguide/app-spec-ref.html). We are going to talk about a few of the hooks that are useful for deployment. In the root of your Bitbucket application create a file name appspec.yml and paste the following.

    version: 0.0
    os: linux
    files:
      - source: /
        destination: /home/ubuntu/sample-app
    permissions:
      - object: /home/ubuntu/sample-app
        owner: ubuntu
        group: ubuntu
    hooks:
      BeforeInstall:
        - location: before_install.sh
          runas: ubuntu
      AfterInstall:
        - location: after_install.sh
          runas: ubuntu

The files section says that we are going to copy the entire application to the directory /home/ubuntu/sample-app. We are using the permissions section to set the owner of that directory to Ubuntu.

The *BeforeInstall* hook is run before any of the files are copied over to the server. This is a good place to setup your server and installed any required libraries. Although you can run commands directly in the appspec file, I find it easier and more flexible to have script files. Create a file name before_install.sh and paste the following code.

    sudo apt-get update
    
    sudo apt-get install -y \
        apache2 \
        apache2-dev \
        libapache2-mod-wsgi-py3 \
        python3 \
        python3-dev \
        python3-pip
    
    if [ -d /home/ubuntu/sample-app ]; then
      sudo rm -R /home/ubuntu/sample-app
      mkdir /home/ubuntu/sample-app
    fi

We are installing apache2, mod-wsgi and python 3. We are also creating an empty sample-app folder when our new code will be copied. If you don’t do this then CodeDeploy will fail if there are already files in that folder with the same name. It will not overwrite the files. This also ensures that we have a clean copy when deploying new code.

Once your application files will are copied to the server, then the *AfterInstall* hook will run. This is a good place to run application level code. For python this would be installed your python libraries in your virtual environment from the application requirements.txt file. If you are using sqlalchemy then you can run alembic command to upgrade your database. Static files can be copied over to the apache directory as well.

There various other lifecycle hooks in CodeDeploy and you can read more details on the AWS [docs](http://docs.aws.amazon.com/codedeploy/latest/userguide/app-spec-ref-hooks.html). Once the appspect.yml file is created then we are ready to deploy. Simple go to any commit on Bitbucket and you will a Deploy to AWS button on the right.

![](https://cdn-images-1.medium.com/max/2000/1*uTj2c1iPX7lE389ZCLhIVQ.png)

Click on it and you will see the choices of deployment groups for the CodeDeploy application that was setup in the settings. In a typical situation we might have stage, testing or prod here. Choose your group and click Submit. You can follow the process in the AWS console for CodeDeploy under the deployment options. Once the deployment is done you can SSH into the server and confirm the files have been copied. You can then make changes and deploy again to confirm the new updates.

Using this setup it becomes very easy to deploy code from any branch to any server you have running. You can easily deploy code to testing or staging severs from various branches for testing. You can also deploy code to your production server as well.

If you servers are in a classic load balancer you will have to follow some extra instructions. AWS has [sample](http://docs.aws.amazon.com/codedeploy/latest/userguide/elastic-load-balancing-integ.html) script files that will assist in register and de-registring from a load balancer. This also include autoscaling groups as well.

Definitely there is some leg work involved in setting up CodeDeploy for your application. But once it’s setup it becomes a dream to write code and deploy. You will have confidence in your deployment and it will make your continuous integration workflow smooth.
