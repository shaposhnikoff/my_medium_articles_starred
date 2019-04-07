
# Build your CI/CD with AWS CodePipeline and Elastic Beanstalk



DevOps is fast becoming a standard right now and CI/CD (Continuous Integration and Continuous Delivery/Deployment) is the term often associated with development and release of a software in the DevOps process. In simple terms, CI (Continuous Integration) is a process that allows developers to integrate their code in the main shared repository, multiple times a day and providing them with instant feedback about possible broken functionality or tests. Instead of building and integrating software work products from different teams/people at the end of the development cycle, with Continuous Integration you can build it at regular intervals. This helps developers to meet the code quality standards, resolve bugs early and reduces integration cost. Continuous Delivery is an extension of Continuous Integration that helps you focus on automating the delivery process of software development which helps in deploying, staging, and production at any time. Continuous Deployment is a process which automatically builds/deploys the code on the servers. The process can be fully automated which will build, test, and deploy in multiple environments. It will automatically handle any build failures and revert back to the previous good state.

There are several tools in the market which can help you build a CI/CD pipeline capable enough to answer all your agile and continuous build, deploy, delivery requirements. [Jenkins](https://jenkins.io/), GoCD, Travis CI, GitLab, Bamboo are some of the names which come to my mind immediately when someone says CI/CD. But in this blog, I am going to explain how you can build a complete CI/CD with the help of AWS Developers tools. With AWS you have multiple tools such as [AWS CodePipeline](https://aws.amazon.com/codepipeline/), [AWS CodeBuild](https://aws.amazon.com/codebuild/), [AWS CodeDeploy](https://aws.amazon.com/codedeploy/), and [Amazon EC2](https://aws.amazon.com/ec2/) to build your continuous pipeline. I am going to explain this by setting up a CI/CD pipeline using AWS CodePipeline and [AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/) service. So, let’s have a look at how you can do this step-by-step:

**Step 1: Create a deployment environment**
In continuous deployment, you need a deployment environment which can be either an EC2 server or a Docker container or Elastic Beanstalk (which can handle the environment configuration and bootstrapping automatically). In this case, I am going to use Elastic Beanstalk.

a. Elastic Beanstalk will host web application without the need to launch, configure, or operate. You don’t need to worry about the infrastructure, Elastic Beanstalk will take care of everything. I am going to deploy a PHP application on Elastic Beanstalk. So, let’s create an environment for PHP application with a single instance –

![](https://cdn-images-1.medium.com/max/2048/0*Vwj0omhYCHDaSm85.png)

b. Select PHP as a platform and other environment parameters like single instance, vpc, security group and keys in the advanced …[read more](https://www.opcito.com/build-your-ci-cd-with-aws-codepipeline-and-elastic-beanstalk/)
