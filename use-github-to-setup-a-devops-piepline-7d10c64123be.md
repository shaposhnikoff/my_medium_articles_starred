
# Use Github to setup a DevOps piepline

Connect CodeDeploy to Github. Push to Master branch with git and the full solution will be automatically deployed to AWS.

### 📚 🚀 **Amazon tutorials:**

* [What Is AWS CodeDeploy?](http://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html)

* [Automatically Deploy from GitHub Using AWS CodeDeploy](https://aws.amazon.com/blogs/devops/automatically-deploy-from-github-using-aws-codedeploy/)

* [Tutorial: Use AWS CodeDeploy to Deploy an Application from GitHub](http://docs.aws.amazon.com/codedeploy/latest/userguide/tutorials-github.html)

[CodeDeploy](https://aws.amazon.com/codedeploy/) is an Amazon Web Service that makes it easy to deploy code to EC2 instances. Github is going to notify CodeDeploy when there is a push to a repo and CodeDeploy will grab the latest changes.

### 🐙 🐈 Github

You must have a Github account! Create a new Github repository:

![](https://cdn-images-1.medium.com/max/2000/1*-MtQmROOzU-JxIgQGrt4YQ.png)

We are going to pull in a sample repository that Amazon is hosting on AWS S3. Head into the terminal and whatever directory you store code or projects. Run the following commands:

    $ mkdir deploy-to-gitub
    $ cd deploy-to-gitub
    $ curl -O [http://s3.amazonaws.com/aws-codedeploy-us-east-1/samples/latest/SampleApp_Linux.zip](http://s3.amazonaws.com/aws-codedeploy-us-east-1/samples/latest/SampleApp_Linux.zip)
    $ unzip SampleApp_Linux.zip
    $ rm SampleApp_Linux.zip
    $ ls

    LICENSE.txt appspec.yml index.html  scripts

As you can see, this brings in some starter files from the AWS project and drops them into our repo. Add them to git and push them to the git repository we created:

    $ git init
    $ git add -A
    $ git commit -m 'omg first commit'
    $ git remote add origin [https://github.com/connor11528/deploy-to-github.git](https://github.com/connor11528/deploy-to-github.git)
    $ git push origin master
    $ git status

The files from your local machine now show up in that repo on github.com! The sample code is [available here](https://github.com/connor11528/deploy-to-github) (give it a ⭐ if you like).

To see the application running locally, you can use Python [SimpleHTTPServer](http://www.linuxjournal.com/content/tech-tip-really-simple-http-server-python) or [http-server](https://github.com/indexzero/http-server) Node.js package.

### 💻 📡 AWS CodeDeploy

![From [the docs](http://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html)](https://cdn-images-1.medium.com/max/2000/1*ez8vS3j9wcWIFy4xHfpQEg.png)*From [the docs](http://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html)*

In AWS world we have EC2 (Elactic Cloud Compute) instances. Those are our servers and virtual machines in the cloud. We are going to use **In-place deployment. **In an in-place deployment, CodeDeploy reads the code from our **appspec.yml **file. The App Spec (appspec.yml) file is important because it defines the deployment actions we want AWS CodeDeploy to execute.

We then supply information about our **Github repo** to CodeDeploy. In AWS CodeDeploy a set of EC2 instances and Auto Scaling groups are called **deployment groups**. Each time we push to Github CodeDeploy will push the new code revisions to our specified deployment groups.

We want to automate our deployments so that whenever we git push to the master branch on Github our app automatically deploys to the internet and AWS.

**Create an IAM user with the appropriate permissions**

Set up an IAM user that has permissions to work with CodeDeploy.

If you have an existing IAM user that you use you can add the

![AWSCodeDeployFullAccess or AdministratorAccess is permission levels we need for the IAM user.](https://cdn-images-1.medium.com/max/2000/1*nwdySC6vbybwVjTCO0elCQ.png)*AWSCodeDeployFullAccess or AdministratorAccess is permission levels we need for the IAM user.*

**Set up integration with CodeDeploy**

Select CodeDeploy on your Github repo

![](https://cdn-images-1.medium.com/max/2000/1*hWk9pI6OL8WBlWWg3Q1G_Q.png)
