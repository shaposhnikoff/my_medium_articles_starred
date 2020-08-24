
# AWS Code Deploy Setup (EC2, IAM, Github)

CICD Setup Part 1— AWS Code Deploy Setup (EC2, IAM, Github)

In this guide we will build a code deploy setup that takes our commits from github repo for our source codes and deploys it automatically on our servers. CICD practices in the most simplest form. This guide will focus more on CICD practices.

This guide assumes that you have an intention of understanding how GitHub can be integrated into code deploy as a source.

This guide also assumes that code pipeline will be configured to consume the code deploy setup

I have used the most minimal of VM specifications for the purpose of this guide — configure your VM according to your application need.

This guide focuses on providing Continuous integration where developers focus more on their code and committing to GitHub which becomes a source for our pipeline that triggers build process

**GITHUB REPO: [**https://github.com/Faithful1/AWS_CodeDeploy_Example](https://github.com/Faithful1/AWS_CodeDeploy_Example.git)

## 1. Create IAM role for code deploy and ec2
> **Reasons** — i am role for ec2 instances Allows EC2 instances to call AWS services on your behalf and also we need to create one for code deploy

Select the IAM Service in the drop down list of aws services in your console

go to roles -> create roles -> (choose the service that will use this role)EC2 -> Next

In the search box search for ec2role as in the image below.

![](https://cdn-images-1.medium.com/max/2000/1*upo8Qa6gksQUKVqo57IAuA.png)

click tags and fill in the details as you wish — in this case i will chose to leave the tags blank. Click Next.

Give it a Role name and leave description as it is then click create role

![](https://cdn-images-1.medium.com/max/2000/1*4jC_4tFweGu9FBodXLvkFQ.png)

click roles in the left pane and you should find the newly created role in the existing list

Navigate to the newly created role and click on it -> select Add inline policy->click json tab->Replace the current json syntax with the syntax provided as seen in the image(i have provided this syntax in our sample repo)
> So we have to set our policy with the same region as our ec2 instance we will be creating hence this syntax. So all we need to do is create our instances in the same region.

![](https://cdn-images-1.medium.com/max/2000/1*icufwEdfD_ZyZGVJQYMpHA.png)

![](https://cdn-images-1.medium.com/max/2000/1*2CiixZYFZdLgG3IWkVQ3Zw.png)
> **WE NEED TO REPEAT THE SAME PROCESS AGAIN BUT THIS TIME FOR CODE DEPLOY TO INTERACT WITH OTHER AWS SERVICES**

Select the IAM Service in the drop down list of aws services in your console

go to roles -> create roles -> (choose the service that will use this role)EC2 -> Next

In the search box search for codedeploy as in the image below.

Create IAM for CodeDeploy

![](https://cdn-images-1.medium.com/max/2000/1*7f2_448jHYwnBx7Ajw7i2A.png)

Click next -> give the role a name and click create role.

Next we will edit the policy by selecting our newly created role and clicking edit policy.

Replace the existing policy with that as seen below. (Script is provided in the github repo)

![](https://cdn-images-1.medium.com/max/2000/1*iTwQJNJqYSloldfHAQ5lmA.png)

NOW we are done with granting our services the IAM rights to interact with each other. Now lets create our instances.

Please note that i will create only one instance here for the purpose of this guide. This can likely be copied through multiple instances for different stages. like staging and production servers.

lets call this our** staging server**.

## **2. Creating Our Staging Server**

**Assumption**:
> We will hardly have a high volume of traffic — i can arguably say this due to the complexity of our application as stated in the requirement. This idea is also mostly influenced by the idea that its an internal tool which even if it spans up to a multiple range of users, we can easily setup a contingency in the form of **Auto scaling(We will talk about this later)**
> Hence for this demonstration i have chosen Amazon Linux AMI t2 micro with 2gb of memory storage(helps to effectively manage cost more like a pay as you use bases as compared to buying up larger spaces and not getting to ever scale up to them — this is a different case for most complex applications with multiple users with huge potential of scaling up to thousands of users). The idea is every application has its own solution based on the requirements.

Go to EC2 in the Console/service drop-down

select Amazon Linux 2 AMI (HVM), SSD Volume Type

![](https://cdn-images-1.medium.com/max/2000/1*dCbo0-Fz5IwBX8iOzPCicg.png)

Select t2 micro free tier

![](https://cdn-images-1.medium.com/max/2000/1*DJC7DPeEfPintQCWJpR-Mw.png)

Click configure instance details

Select IAM Role in the i am role drop down and choose your created EC2 Role

![](https://cdn-images-1.medium.com/max/2062/1*buCcB37KiAkF7AapcbmtVA.png)

Click Next until we get to Configure security group.

Open up SSH, HTTP and HTTPS — For SSH **make sure** to put in your specific address because we only want our ip to be able to ssh into our server. We can see a warning there that tells us its not save to make access privileges open to all ips

HTTP TCP 80 0.0.0.0/0

HTTP TCP 80 ::/0

SSH TCP 22 (YOUR IP ADDRESS)

HTTPS TCP 443 0.0.0.0/0

HTTPS TCP 443 ::/0

![](https://cdn-images-1.medium.com/max/2396/1*T0xdzYW0j4Cyz6LvntpoEA.png)

Click Review and Launch. You should already see your new instance in the list of running instances.

To login to your Instance using SSH please follow this AWS guide
[**Connecting to Your Linux Instance Using SSH - Amazon Elastic Compute Cloud**
*Connect to your Linux instances using an SSH client.*docs.aws.amazon.com](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)

## 3. Setting Up Code Code Deploy

For code deploy to work we need a code deploy agent running in our existing server and it must validate with your AWS access secret and region.

Follow the guide above and get into your server.

once in your server make sure to update by running

***sudo su***

***yum -y update***

Now lets install aws-cli

***yum install -y aws-cli***

go to your home and ec2 user
> **cd /home/ec2-user**

Here you will setup your AWS access, secret, and region by running the following commands
> **aws configure**
> ***aws s3 cp s3://aws-codedeploy-us-east-1/latest/install . — region us-east-1 (if in east AWS)***

Install the Agent by following the AWS documentation below
[**Install or reinstall the AWS CodeDeploy agent for Amazon Linux or RHEL - AWS CodeDeploy**
*Sign in to the instance, and run the following commands, one at a time.*docs.aws.amazon.com](https://docs.aws.amazon.com/codedeploy/latest/userguide/codedeploy-agent-operations-install-linux.html)

## 4. Run Code Deploy from the console

So one thing we should know is that Code deploy listens for two things.

1. An agent in your server which we have just installed if you followed all the steps correctly

1. A configuration on a file set in your root folder called appspec.yml

1. create an appspec.yml file in your root folder and copy the code below

1. See the link below to understand every step of the appspec file [https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file.html](https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file.html)

![File can be found in the git hub repository](https://cdn-images-1.medium.com/max/2000/1*SDMdo12BUjpf4cCXkVAfhQ.png)*File can be found in the git hub repository*

Go to your console and select code deploy in the services drop down

Click create an application

![](https://cdn-images-1.medium.com/max/2000/1*GqVrGCtJjxwQ13jJJv6ISA.png)

![](https://cdn-images-1.medium.com/max/2000/1*dVtsX1dKHOpEZXmNHJuHqw.png)

Lets call our application pulse pulse and select EC2/On-premises
> An on-premises instance is any physical device that is not an Amazon EC2 instance that can run the AWS CodeDeploy agent and connect to public AWS service endpoints. which is what we have just done with our instance earlier. Read more about it here.
[**Working with On-Premises Instances for AWS CodeDeploy - AWS CodeDeploy**
*Learn how to configure an on-premises instance for use in AWS CodeDeploy deployments.*docs.aws.amazon.com](https://docs.aws.amazon.com/codedeploy/latest/userguide/instances-on-premises.html)
> Deploying an AWS CodeDeploy application revision to an on-premises instance involves two major steps:
> **Step 1** — Configure each on-premises instance, register it with AWS CodeDeploy, and then tag it.
> **Step 2** — Deploy application revisions to the on-premises instance.
> AWS Lambda is the service for AWS that runs Serverless functions. AWS Lambda lets you run code without provisioning or managing servers (this is another discussion for another day — should be super exciting to explore)

Click Create Application

Now we will be prompted to create a deployment group .

Enter the name of the deployment group, select the service role we created in step 1,

The major difference between **in-place** deployment and **Blue/green **deployment is that in-place goes offline for a while. which is referred to as downtime. whereas blue green is set up in such a way that it replaces the instances in the deployment group with new instances and deploys the latest application revision to them. After instances in the replacement environment are registered with a load balancer, instances from the original environment are deregistered and can be terminated.

This is the choice for every deployment as this is critical for business. You don’t want your app going down for any reason at all.

Blue green works in two ways — either we provision an auto scaling group or we manually provision instances. and this will definitely require load balancers

For this setup **i will use in-place** for simplicity of this guide

In the Deployment settings select **OneAtATime.**

Click Create Deployment Group

## **5. Create Deployment**

Select your deployment group and click Create Deployment

![](https://cdn-images-1.medium.com/max/2212/1*ASgLa_SNxpcCbJBK9w27oQ.png)

Specify your revision type either S3 or GitHub — Select GitHub since ours is stored in GitHub. Enter your github credentials and connect to github

![](https://cdn-images-1.medium.com/max/2000/1*qOi_PCK4A3mrvI4lDa-_ng.png)

Click Create deployment

Click View Events to see a step by step build of the deployment

![](https://cdn-images-1.medium.com/max/2000/1*A_x-jycINtupLiGgd0JeRg.png)

Now go to your Ec2 instance for your staging server and copy the IPv4 Public IP and paste on your browser and see your app or if you have a domain attached to your instance, simply run the instance.

You can go to your local/dev server and modify anything on the .html file to see how the changes are picked when you provide the commit id to code deploy.

**NOTE**: If you get errors it means you havent done something right. So how do you trouble shoot:

You can ssh into your Server and navigate to this path **/var/log/aws/codedeploy-agent/codedeploy-agent.log to see a log of the deployment**

**Recommendations:**

It is always safer to have your database in another server completely different from your application providing an endpoint to be accessible by your client server. paves way for easy scalability, management and Security

Auto-scaling is a good practice and is cost effective especially for highly scalable applications

Always restrict your IP for SSH rule to allow only a specific IP to be able to tunnel into your instance

SSL certificates are highly advisable preferable for encryption

Restrict access to admin login pages in your applications to avoid external access

## Upcoming Articles:

Next on my agenda is to walk you through how to **Create a Four-Stage Pipeline.**

Basically what we will do is to create a more complex pipeline that uses a GitHub repository for our **source**, a **Jenkins** build server to build the project, and an **AWS CodeDeploy** application to deploy the built code to a staging server. After the pipeline is created, we will edit it to add a stage with a test action to test the code, also using **Jenkins**.

Who is excited about amazing complex setups?
