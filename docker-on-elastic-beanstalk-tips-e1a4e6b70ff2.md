
# Deploy and monitor Docker containers using AWS Elastic Beanstalk in 3 steps

Deploy your application without provisioning the underlying infrastructure while maintaining high availability

![](https://cdn-images-1.medium.com/max/5120/1*HhWpR4px1zGdhQAj6tQZoQ.png)

AWS Elastic Beanstalk is one of the most used AWS services — allowing developers to deploy your application without provisioning the underlying infrastructure while maintaining the high availability of your application.

In this post, I’ll walk you through how to use Elastic Beanstalk to deploy Docker containers from scratch. We’ll then automate your deployment process with a Continuous Integration pipeline, and explore advanced topics including debugging and monitoring of your applications in EB.

![CI/CD pipeline with Elastic Beanstalk](https://cdn-images-1.medium.com/max/2598/1*SQWdJzC7SXfOsItNXPh_kg.png)*CI/CD pipeline with Elastic Beanstalk*

### **Step #1: Environment Setup**

To get started, let’s use the AWS command line interface (CLI) to create our new Elastic Beanstalk application with the following command:

<iframe src="https://medium.com/media/98b12c9d0f49e779c21d37d4a86a348f" frameborder=0></iframe>

* Next, create a new environment called *staging*:

<iframe src="https://medium.com/media/4bf5b8dcd41c9153501fc47c654cbab7" frameborder=0></iframe>

* In the AWS Console, you’ll now see our new environment created in the Elastic Beanstalk section:

![](https://cdn-images-1.medium.com/max/5748/1*JTY0D0Y2O8E-ko_sCxR17g.png)

* Direct your browser to the environment URL, and a sample Docker application will be displayed:

![](https://cdn-images-1.medium.com/max/4672/1*i4J5awXJDSvrMYkOPo49oQ.png)

* Now let’s deploy our application. I wrote a small web application in Go to return a list of Marvel Avengers —* I see you Thanos!*

<iframe src="https://medium.com/media/811096b3ad303fccca3c0db0720f8385" frameborder=0></iframe>

* Next, we’ll create a *Dockerfile* to build the Docker image.

* Since Go is a compiled language, we can use the Docker multi-stage feature to build a lightweight Docker image:

<iframe src="https://medium.com/media/90851f1cd9a8aaab91b0c3aa6e54869a" frameborder=0></iframe>

* Next, create a *Dockerrun.aws.json *which describes how the container will be deployed in Elastic Beanstalk:

<iframe src="https://medium.com/media/24c58a606eab93c93e05b4a980143d5a" frameborder=0></iframe>

* Now that the application is defined, create an application bundle by building a ZIP package:

<iframe src="https://medium.com/media/07cb4f9c17f1b16ad331f4135b7caba8" frameborder=0></iframe>

* Using the AWS CLI, create an S3 bucket to store the different versions of your application bundles:

<iframe src="https://medium.com/media/e96638b9082586ba381b9bbe9077aaf3" frameborder=0></iframe>

* Next, issue the following command to copy the application into the bucket:

<iframe src="https://medium.com/media/6f3953c6ea013d433cd9f8540d140f17" frameborder=0></iframe>

* Then create a new application version from the application bundle:

<iframe src="https://medium.com/media/29b26fd76222228d3e550a5db8ac1be5" frameborder=0></iframe>

![](https://cdn-images-1.medium.com/max/5664/1*Jp7BQ-FGsTDH4wkTS4Vi8Q.png)

* Finally, deploy the version to the staging environment:

<iframe src="https://medium.com/media/a11c5dc848fb45c498601d5925088a0e" frameborder=0></iframe>

* Give it a few seconds while it’s deploying the new version:

![](https://cdn-images-1.medium.com/max/5708/1*LHuKr-p4Waf2pCeX0ODDjA.png)

* Then, redirect your browser to the environment URL — a list of Avengers will be returned in a JSON format as follows:

![](https://cdn-images-1.medium.com/max/4076/1*zCOL6t5jG1Q6AJeAqwcXfQ.png)

* Now that our Docker application is deployed, let’s automate this process by setting up a CI/CD pipeline!

### **Step #2: CI/CD Pipeline**

For the CI/CD pipeline, I like using CircleCI — the same steps can be applied to whatever CI server you’re familiar with using.

<iframe src="https://medium.com/media/c85e3a8486ce0f25ae70469878a5ac44" frameborder=0></iframe>

Our CI/CD pipeline will:

1. install the AWS CLI and prepare the environment

1. run unit tests

1. build a Docker Image and push to DockerHub.

1. creating a new application bundle

1. deploy the bundle to Elastic Beanstalk

Let’s get started* *by* c*reating a *circle.yml* file with the following content:

<iframe src="https://medium.com/media/29535579bc9dd0cb7e66f4db09620a2f" frameborder=0></iframe>

* In order to grant Circle CI permissions to call AWS operations, we need to create a new IAM user with following IAM policy:

<iframe src="https://medium.com/media/40058ed2ab703b2bd091a6c24a006974" frameborder=0></iframe>

* Generate AWS access & secret keys

* Then, head back to Circle CI and click on the project settings and paste the credentials :

![](https://cdn-images-1.medium.com/max/5716/1*uEj-1T9L6XXDk0AGWnAlCQ.png)

* Going forward, a build will be triggered every time you push a change to your code repository

![](https://cdn-images-1.medium.com/max/5724/1*nXdaiKn2cN4Xptqjp3T56g.png)

* And a new version will be deployed automatically to Elastic Beanstalk:

![](https://cdn-images-1.medium.com/max/5684/1*Wec3mPlIu54bbuZ1YtheKg.png)

### **Step #3: Monitoring**

Monitoring your deployed applications is essential. Since AWS CloudWatch doesn’t expose metrics such as the memory utilization of your applications in Elastic Beanstalk, we’ll solve this issue by creating custom metrics.

In this section, we’ll install a data collector agent on the instance. The agent will collect metrics and push them to a time-series database.

To install the agent, we’ll use the .*ebextensions* folder to create the following three configuration files for collecting metrics:

* *01-install-telegraf.config*: install Telegraf on the instance

<iframe src="https://medium.com/media/987a3db2bb0f6fac8c4b5a720c5ac22d" frameborder=0></iframe>

* *02-config-file.config*: create a Telegraf configuration file to collect system usage & docker containers metrics.

<iframe src="https://medium.com/media/3507df4dd0442d0eba36085ca9f5d865" frameborder=0></iframe>

* *03-start-telegraf.config*: start Telegraf agent.

<iframe src="https://medium.com/media/69caf2baaaf1b4749ea996f1374c70b7" frameborder=0></iframe>

* Once the application version is deployed to Elastic Beanstalk, the metrics will be pushed to your time series database.

* In this example, I used InfluxDB as data storage nad created some dynamic Dashboards in Grafana to visualize metrics in real-time:

**Containers metrics:**

![](https://cdn-images-1.medium.com/max/5728/1*rX24KKhOiaLw274R_wenuA.png)

**Host metrics:**

![](https://cdn-images-1.medium.com/max/5704/1*-kafsySG_Wf0hEnoLXGc3A.png)

For in-depth explanation on how to configure Telegraf, InfluxDB & Grafana, be sure to read my previous [article](https://hackernoon.com/mysql-monitoring-with-telegraf-influxdb-grafana-4489e6df0220) that describes the process for creating an interactive, real-time & dynamic dashboard.

I’m hope the steps in this blog will save you some time and effort by automatically deploying your application using Docker containers and Elastic Beanstalk — without needing to provision the underlying infrastructure.

The full code can be found on my [GitHub](https://github.com/mlabouardy/cost-optimization). I’d be interested in your comments, feedback, or suggestions below — or connect with me directly on Twitter [**@**mlabouardy](https://twitter.com/mlabouardy).

![](https://cdn-images-1.medium.com/max/5120/1*R44tI1fqpXWrbeszlu9qjg.png)
