Unknown markup type 10 { type: [33m10[39m, start: [33m67[39m, end: [33m85[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m90[39m, end: [33m113[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m58[39m, end: [33m61[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m48[39m, end: [33m65[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m70[39m, end: [33m87[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m88[39m, end: [33m106[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m156[39m, end: [33m181[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m42[39m, end: [33m70[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m30[39m, end: [33m47[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m31[39m, end: [33m54[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m37[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m91[39m, end: [33m97[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m99[39m, end: [33m110[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m127[39m, end: [33m135[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m6[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m8[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m150[39m, end: [33m163[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m165[39m, end: [33m180[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m186[39m, end: [33m202[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m15[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m16[39m }

# Deploy to EC2 with AWS CodeDeploy from Bitbucket Pipelines

Deploy to EC2 with AWS CodeDeploy from Bitbucket Pipelines

Before Continuous Delivery and containers and the idea of immutable environments, people updated environments manually with the latest code at release time. It is a very common place to be and I would bet that most software shops are still working this way.

A very easy step towards Continuous Delivery from this state is to automate the manual step in that process. That is, automate updating environments with the latest and greatest code.

This is very easily achieved by combining a couple very useful tools: AWS CodeDeploy and Bitbucket Pipelines.

## Prerequisites

There are some requirements that need to be satisfied before this deployment pipeline can be operational. We will walk through these steps together. Some basic AWS knowledge is assumed.

### IAM

A user needs to be created that Bitbucket can use to upload artifacts to S3 and inform CodeDeploy that a new revision is ready to be deployed.

First create an IAM group called *CodeDeployGroup*. This group needs AmazonS3FullAccess and AWSCodeDeployFullAccess permissions. Create a user and add it to this group. This user only needs programmatic access.

![](https://cdn-images-1.medium.com/max/2492/1*lLygUWel3nHU8MXSy3JtBA.png)

![](https://cdn-images-1.medium.com/max/2000/1*unt1RlP18gdo2rQwmP4Icg.png)

Make note of this user’s access key. It will be required later.

Now we will create a role that can be associated to EC2 instances and interact with CodeDeploy.

After going to Roles and clicking *Create role*, select the EC2 AWS service since EC2 is the service that will interact with CodeDeploy.

![](https://cdn-images-1.medium.com/max/4168/1*ZCPr3TcUEK-bSDAbI8G-aw.png)

Make sure the **Trusted entities** and **Policies** are ec2.amazonaws.com and AWSCodeDeployRole AmazonS3FullAccess, respectively.

![](https://cdn-images-1.medium.com/max/4244/1*OKf4-fXuEn9UkfAe32P0AQ.png)

After creating the role, edit the **Trust Relationship** to be as follows. Change the region to the region you are working out of.

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "Service": [
                "ec2.amazonaws.com",
                "codedeploy.us-west-2.amazonaws.com"
            ]
          },
          "Action": "sts:AssumeRole"
        }
      ]
    }

To recap, we created a new IAM user that belongs to a group that has full CodeDeploy and S3 permissions. We wrote down this user’s access key information for later use.

We then created an IAM role for EC2 and edited the trust relationship to also allow access to CodeDeploy.

### S3

A bucket needs created that will store the *revisions* of your application. A revision is just an archive of whatever is required to run your code plus the [AppSpec](https://docs.aws.amazon.com/codedeploy/latest/userguide/application-specification-files.html) file.

![](https://cdn-images-1.medium.com/max/2000/1*K5qdjLSyXl96D5enhcRWrw.png)

In a real-world example you would want to tinker with the lifecycle management of this bucket to help save costs, but just leave everything as default for now.

### EC2

EC2 instance(s) need to be launched that CodeDeploy can use as deployment targets.

Choose the Amazon Linux AMI and t2.micro instance type. Configuration details are all default except for IAM role, which will be the role created earlier.

![](https://cdn-images-1.medium.com/max/3560/1*ljCDrZlYfSH7YgEHM1-f-Q.png)

Leave the storage settings as the default. Set a tag that you will remember. CodeDeploy uses this tag to identify the instances to deploy to. I use the tag Name=Code Deploy Instance. The security group should allow SSH and any ports you need in order to access your application.

![](https://cdn-images-1.medium.com/max/4980/1*o-1aXqZqDL4KegCrofaaBw.png)

Finally, the CodeDeploy agent needs to be installed on the instance. In order to not repeat information, just follow the [official directions for your platform](https://docs.aws.amazon.com/codedeploy/latest/userguide/codedeploy-agent-operations-install.html). This agent is how CodeDeploy actually makes changes on the EC2 instance.

The key points with configuring EC2 is that an IAM role must be attached that allows EC2 to communicate with CodeDeploy, the CodeDeploy agent needs to be running on the instance(s), and the instance(s) needs to be tagged so that CodeDeploy can identify the instance(s) participating in deployments.

### CodeDeploy

If this is your first time in CodeDeploy, you will get to the following screen.

![](https://cdn-images-1.medium.com/max/3320/1*J67o1y5HJ146r6N7eUXi9w.png)

Select Custom deployment.

Give the Application and the Deployment Group a name. Select EC2/On-premises and In-place deployment.

![](https://cdn-images-1.medium.com/max/3824/1*hBEYPGowQhB-6DLzmB2cEQ.png)

We are deploying to existing EC2 instances outside of any Auto Scaling, so select the *Amazon EC2 instances* option and provide the tag that identifies the instance(s) you want to deploy to.

![](https://cdn-images-1.medium.com/max/3852/1*N0fUVMPosTs5EFB_P3nUjw.png)

Leave the Deployment Configuration set to CodeDeployDefault.OneAtATime

![](https://cdn-images-1.medium.com/max/3792/1*GxoYHZRIEhCbeIZlcP5Z7Q.png)

The service role should be set to the IAM role that was created earlier.

![](https://cdn-images-1.medium.com/max/3780/1*2OBJOOCLjK7QbO1MhCoFoQ.png)

This was a lot. In short, all that we did was tell CodeDeploy that we have a new application and what EC2 instances are participating in the deployments.

### Bitbucket

We now need to tell Bitbucket about all of this information. Set the following Environment Variables for the repository. These will get used by AWS SDK via a Python script that will show up later.

![](https://cdn-images-1.medium.com/max/3404/1*_8OSC4fDsg0HMZAacYPjzw.png)

The AWS_SECRET_ACCESS_KEY and AWS_ACCESS_KEY_ID are for the user created at the beginning of this article, not your personal AWS user account.

## Build The Pipeline

![Not this kind](https://cdn-images-1.medium.com/max/3840/1*ZiPgZaC6aVePEaz7CyFZ-Q.jpeg)*Not this kind*

All the groundwork is done and the only thing left is to build the pipeline to trigger CodeDeploy. This happens in three steps:

1. Deploy the revision* *to S3

1. Tell CodeDeploy that a new revision is ready

1. Wait for CodeDeploy to perform the deployment

Detailing the programmatic way of doing this is out of the scope of this article, so just download [this Python script](https://bitbucket.org/awslabs/aws-codedeploy-bitbucket-pipelines-python/src/73b7c31b0a72a038ea0a9b46e457392c45ce76da/codedeploy_deploy.py?at=master&fileviewer=file-view-default) and place it in the root of your project directory. This script will perform all of the above steps. Feel free to read through it to get an understanding of what is going on. You will see that this script is using the environment variables we set in Bitbucket.

### bitbucket-pipelines.yml

The first step is to write the bitbucket-pipelines.yml file. For my very simple [Sinatra](http://sinatrarb.com/) server there’s nothing to do but create the revision to send to S3.

<iframe src="https://medium.com/media/ab4f34dcdbcef185c15da8fb89a7e8cb" frameborder=0></iframe>

If you are unfamiliar with AWS SDKs, boto3 is just a Python SDK for AWS.

As you can see we have created a zip archive (this can be any major archive format) out of app.rb, appspec.yml, and the entire scripts/ directory. This archive needs to contain all of the code and libraries required to run your application because this is the archive that is ultimately deployed.

### Application Code

app.rb is the very simple Sinatra I created for this. Nothing much to note here.

<iframe src="https://medium.com/media/bb1cfdbda99a08d57d930f905bcaae17" frameborder=0></iframe>

### Lifecycle Scripts

scripts/ contains all of the scripts that manage the lifecycle of the application. At the most basic configuration there should be scripts to start the server, stop the server, and install dependencies required by the server. Remember to make these scripts executable.

I have the following scripts for managing my Sinatra server. In a real environment, you would want to use something like systemd or pm2 to manage the starting and stopping of your server.

<iframe src="https://medium.com/media/b8f7645dbc8cfa99252c88d97c079930" frameborder=0></iframe>

### Application Specification

appspec.yml is the CodeDeploy configuration. There are several lifecycle event hooks that can be configured here, but we will only worry about three. BeforeInstall, ApplicationStop, and ApplicationStart.

<iframe src="https://medium.com/media/84250d154b69c942cbc41d0d6bb9d3a6" frameborder=0></iframe>

BeforeInstall is ran before the next revision is moved into place. This is a good place to make sure that the everything required to run the application is in place.

ApplicationStop is ran as the first step before the revision is even downloaded. This should contain some way to gracefully shutdown the application.

ApplicationStart is ran near the end of the CodeDeploy process and is just in charge of starting the application.

[Read more about these lifecycle event hooks and the others available](https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file-structure-hooks.html).

At this point your repository should look similar to this.

![](https://cdn-images-1.medium.com/max/3508/1*fojcGy4N5etOfv6mRo0w5w.png)

### Run the Pipeline

At this point we are ready to push everything to Bitbucket and start the pipeline. Hopefully everything has been configured correctly and you will get a nice green deployment!

![](https://cdn-images-1.medium.com/max/6224/1*LoyQExZRwIVc6c0BXaZmLQ.png)

![Success!](https://cdn-images-1.medium.com/max/7008/1*4Bzz3gJCJH5AbOWcemKG6A.png)*Success!*

Good job! You have now successfully automated deployments using CodeDeploy and Bitbucket Pipelines! This was a pretty basic setup but you should be armed with the knowledge to take this as far as you want.

Stay tuned for a future article where I discuss common CodeDeploy errors and how to troubleshoot them.

Thanks for reading! If you liked what you read, then leave a 👏 or few and follow.
