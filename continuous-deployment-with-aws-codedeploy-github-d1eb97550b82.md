
# Continuous Deployment with AWS CodeDeploy & Github



This post will walk you through how to **AutoDeploy **your application from **Github** using **AWS CodeDeploy**.

Let’s start first by creating 2 **IAM roles** we will use in this tutorial:

* IAM role for **CodeDeploy** to talk to **EC2** instances.

* IAM role for **EC2** to access **S3**.

**1 — CodeDeployRole**

Go to [AWS IAM Console](https://console.aws.amazon.com/iam/home) then navigate to “**Roles**“, choose “**Create New Role**“, Select “**CodeDeploy**” and attach “**AWSCodeDeployRole**” policy:

![](https://cdn-images-1.medium.com/max/2000/0*x42w_F_DHdVDOu0p.)

**2 — EC2S3Role**

Create another **IAM** role, but this time choose **EC2** as the trusted entity. Then, attach “**AmazonS3ReadOnlyAccess**” policy:

![](https://cdn-images-1.medium.com/max/2000/0*305ZkJ-z_tfJou9A.)

Now that we’ve created an IAM roles, let’s launch an EC2 instance which will be used by CodeDeploy to deploy our application.

**3 — EC2 Instance**

Launch a new **EC2** instance with the IAM role we created in last section:

![](https://cdn-images-1.medium.com/max/2000/0*eA1OrS-8b0jaCnG-.)

Next to **User Data** type the following script to install the **AWS CodeDeploy Agent** at boot time:

<iframe src="https://medium.com/media/be587c7bea59019cbbfbe9ffbddf15ee" frameborder=0></iframe>

![](https://cdn-images-1.medium.com/max/2000/0*T72RioSZq7b_9Dw5.)

Note: make sure to allow **HTTP** traffic in the security group.

![](https://cdn-images-1.medium.com/max/2000/0*NWLLHGfkMb-C0hfp.)

Once created, connect to the instance using the **Public IP** via **SSH**, and verify whether the **CodeDeploy** agent is running:

![](https://cdn-images-1.medium.com/max/2000/0*TX2ppY0ESB0463uN.)

**4 — Application**

Add the *appspec.yml* file to the application to describe to AWS CodeDeploy how to manage the lifecycle of your application:

<iframe src="https://medium.com/media/878d52a68135f0c1e4c2fff655009e50" frameborder=0></iframe>

The **BeforeInstall**, will install **apache** server:

<iframe src="https://medium.com/media/661488493d8d9f85bb151b4940db667d" frameborder=0></iframe>

The **AfterInstall** will restart apache server

<iframe src="https://medium.com/media/0cd3bff3f8344d7e1f1ad269436f8ed8" frameborder=0></iframe>

**5 — Setup CodeDeploy**

Go to [AWS CodeDeploy](https://console.aws.amazon.com/codedeploy) and create a new application:

![](https://cdn-images-1.medium.com/max/2000/0*RN_LnSUGAnRLjxDJ.)

Select **In-Place deployement** (with downtime):

![](https://cdn-images-1.medium.com/max/2000/0*IMoHC99fwzsRyrTQ.)

Click on “**Skip**“, because we already setup our EC2 instance:

![](https://cdn-images-1.medium.com/max/2000/0*NN0WElUr0jSZt9Oe.)

The above will take you to the following page where you need to give a name to your application:

![](https://cdn-images-1.medium.com/max/2000/0*oiYO475nvGsdf5zH.)

Select the EC2 instance and assign a name to the deployment group:

![](https://cdn-images-1.medium.com/max/2000/0*TcfRKRID3HNF2yPT.)

Select the **CodeDeployRole** we created in the first part of the tutorial:

![](https://cdn-images-1.medium.com/max/2000/0*BOWLtVW0jdECmXfh.)

Then click on “**Deploy**“:

![](https://cdn-images-1.medium.com/max/2000/0*x0c6BCy-JMt644V5.)

Create a deployment, select **Github** as the data source:

![](https://cdn-images-1.medium.com/max/2000/0*qaprzBoPb35PlQoH.)

Just select “**Connect to GitHub**“. Doing that will pop up a new browser window, take you to Github login where you will have to enter your username and password

![](https://cdn-images-1.medium.com/max/2000/0*xOkHIZJx5LWpyFKO.)

After that come back to this page, and you should see something like below, just enter the remaining details and click “**Deploy**”

![](https://cdn-images-1.medium.com/max/2000/0*WIlp9QfcjVa0etxv.)

This will take you to a page as follows:

![](https://cdn-images-1.medium.com/max/2000/0*a7rgYlGx9LOyjRLc.)

If you point your browser to the** EC2 public IP**, you should see:

![](https://cdn-images-1.medium.com/max/2000/0*VkR0NaL8H6jgwhvU.)

Now, let’s automate the deployment using **Github Integrations**.

**6 — Continuous Deployment**

Go to [IAM Dashboard](https://console.aws.amazon.com/iam), and create a new policy which give access to register and create a new deployment from **Github**.

![](https://cdn-images-1.medium.com/max/2000/0*4IG5FO9T82ZKqNK8.)

Next, create a new user and attach the policy we created before:

![](https://cdn-images-1.medium.com/max/2000/0*cOIjY3R--8U_rSpx.)

Note: copy to clipboard the user **AWS ACCESS ID** & **AWS SECRET KEY**. Will come handy later.

**7 — Github Integration**

Generate a [new token](https://github.com/settings/tokens/new) to invoke **CodeDeploy** from **Github**:

![](https://cdn-images-1.medium.com/max/2000/0*hDL_l_ZHiBmwAfTU.)

Once the token is generated, copy the token and keep it. Then, add **AWS CodeDeploy integration**:

![](https://cdn-images-1.medium.com/max/2000/0*x3QtKacfrbSW4bHm.)

Fill the fields as below:

![](https://cdn-images-1.medium.com/max/2000/0*NZIDql9_cWW8WJpX.)

Finally, add **Github Auto Deployment**

![](https://cdn-images-1.medium.com/max/2000/0*B3woYhHgsZf3c7b_.)

Fill the form as below:

![](https://cdn-images-1.medium.com/max/2000/0*fIkjK1oGLRKMauzP.)

To test it out, let’s edit a file or commit a new file. You should see a new deployment on **AWS CodeDeploy**:

![](https://cdn-images-1.medium.com/max/2000/0*qNzAnyXo5difzqfq.)
