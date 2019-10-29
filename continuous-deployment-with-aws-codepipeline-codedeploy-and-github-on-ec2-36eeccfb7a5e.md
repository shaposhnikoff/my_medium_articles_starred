Unknown markup type 10 { type: [33m10[39m, start: [33m115[39m, end: [33m126[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m273[39m, end: [33m276[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m56[39m, end: [33m67[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m67[39m, end: [33m75[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m98[39m, end: [33m103[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m118[39m, end: [33m129[39m }

# Continuous Deployment with AWS CodePipeline, CodeDeploy and Github on EC2

When it comes to cloud infrastructure — AWS is the standard and as much as I wish GCP was, in reality it is just not as widely adopted and as a result I present to you how I have setup CD on top AWS CodeDeploy with deployment on push to Github and compute on top of EC2.

*All of the steps below are mandatory to make it work so make sure you don’t skip any.*

## [Create an IAM Instance Profile](https://docs.aws.amazon.com/codedeploy/latest/userguide/getting-started-create-iam-instance-profile.html)

Create a file called `CodeDeploy-EC2-Trust.json` with the following contents:

<iframe src="https://medium.com/media/e5111b6ba376fcd49282c0c26b6306b5" frameborder=0></iframe>

Create a file called `CodeDeploy-EC2-Permissions.json` with the following contents:

<iframe src="https://medium.com/media/dcbbc0cb296bc34bc3940ba022e9de07" frameborder=0></iframe>

Run the following commands to create the IAM:

    aws iam create-role --role-name CodeDeploy-EC2-Instance-Profile --assume-role-policy-document file://CodeDeploy-EC2-Trust.json

    aws iam put-role-policy --role-name CodeDeploy-EC2-Instance-Profile --policy-name CodeDeploy-EC2-Permissions --policy-document file://CodeDeploy-EC2-Permissions.json

    aws iam create-instance-profile --instance-profile-name CodeDeploy-EC2-Instance-Profile

    aws iam add-role-to-instance-profile --instance-profile-name CodeDeploy-EC2-Instance-Profile --role-name CodeDeploy-EC2-Instance-Profile

## [Create an Amazon EC2 Instance for CodeDeploy](https://docs.aws.amazon.com/codedeploy/latest/userguide/instances-ec2-create.html)

For CodeDeploy to function we’ll need to install the CodeDeploy agent on each and every one of our targeted instances. Not surprisingly, we’ll have to install it ourselves (because why not make things complicated if you can?!)

Assuming you are using one of the Amazon AMIs — execute the following:

    yum -y update
    yum install -y ruby
    cd /home/ec2-user
    curl -O [https://aws-codedeploy-us-west-2.s3.amazonaws.com/latest/install](https://aws-codedeploy-us-west-2.s3.amazonaws.com/latest/install)
    chmod +x ./install
    ./install auto

Running `sudo service codedeploy-agent status` should tell you that the agent is running, if it doesn’t — try starting it manually: `sudo service codedeploy-agent start`
> At that you might already have some issues — in case you do — make sure to consult with the log file located at `/opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log`
> I found it useful to delete the contents of the folders under `/opt/codedeploy-agent/deployment-root` when things went wrong.

For additional troubleshooting please visit [this page](https://docs.aws.amazon.com/codedeploy/latest/userguide/troubleshooting-ec2-instances.html).

## Set an IAM role for the instance

Now that we have a dedicated IAM role and a properly configured instance — we’ll need to attach the IAM to the instance.
You can do that both while creating the instance or my attaching it later once it is already running.

To attach to an existing one — select the instance and follow this path:

![EC2 Instances screen](https://cdn-images-1.medium.com/max/2000/1*qelbu8BmFijrGGjEpPLpHQ.png)*EC2 Instances screen*

At the following screen, select the IAM we’ve created on the first step called `CodeDeploy-EC2-Instance-Profile`

## [The App](https://github.com/arliber/sample-node)

Now that we have our infrastructure we’ll need to add couple of things to our app to support CodeDeploy, namely an appspec.yml file:

<iframe src="https://medium.com/media/1497608b26c3ece085002d1915598e92" frameborder=0></iframe>

Basically it describes the steps in the build process. There are numerous ways of accomplishing what we want but as you can see I decided to ignore the input from pipeline in the previous step and use git to pull in the changes so the code from previuos steps is stored in raw folder, jus in case.

**before-install.sh**

    #!/bin/bash
    sudo rm -rf /home/ec2-user/raw

This script erases the previous version of the raw code, I found that there are some issue if you don’t do it.

**npm-install.sh**

    #!/bin/bash
    cd /home/ec2-user/sample-node
    git pull origin master
    npm install

This is straighforward and just pulls the code and runs npm install

Finally, **npm-start.sh**

    #!/bin/bash
    cd /home/ec2-user/sample-node
    sudo -H -u ec2-user bash -c 'npm start'

You might spot that I’m explicitly running the start command using ec2-user — this is because the runas command under appspec.yml doesnt work properly but I absolutely have to run it under ec-2user so I can later query PM2 for status.

this should be it for the configuration side of things, now on to the AWS console and setting up the CodePipline and CodeDeploy!

## Setting up CodeDeploy

CodeDeploy is what deploys the code to your actual instances. CodeDEploy will be triggered every time there are changes in our git repo by CodePipeline (next section).

In my case I deployed straight to EC2 instances so I had to identify them uniquely using tag groups — this is why it is crucial add meaningful tags to anything you do in AWS, specifically EC2 instances. That’s how it looks in my case:

![Picking the right instances using using tags](https://cdn-images-1.medium.com/max/2000/1*2fnc38dIV3ODxNaeU1_tzQ.png)*Picking the right instances using using tags*

Finally, make sure to select the IAM we’ve created previously though the command line, something like this:

![Service role ARN](https://cdn-images-1.medium.com/max/2000/1*G5gkuBujfnAuZBWGA_osGg.png)*Service role ARN*

## Setting up CodePipeline

Start by creating a new pipeline.
On step 2 we select the Github as the source provider which allows us to select both the repository and the branch we want to listen to and deploy:

![Selecting the code source](https://cdn-images-1.medium.com/max/2100/1*K1emVWImsyCq1keCntMTqA.png)*Selecting the code source*

We won’t get into building our project in this tutorial so just select “No Build” on step 3.

On Step 4 select the CodeDeploy application and group we have created in the previous step:

![](https://cdn-images-1.medium.com/max/2066/1*YrnTGl3rNrK2Fa6S7F4PQw.png)

Finally, on step 5 select the role we have created at the beginning of the tutorial using the command line, review the configuration and complete the wizard.

We are done! 👏🏼 🎉 🎊 👏🏼 🎉 🎊

Next time you push changes to Github — the pipeline will start, trigger CodeDeploy and update your servers.

Good luck
