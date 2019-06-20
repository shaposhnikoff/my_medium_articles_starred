Unknown markup type 10 { type: [33m10[39m, start: [33m48[39m, end: [33m70[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m69[39m, end: [33m77[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m157[39m, end: [33m165[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m149[39m, end: [33m158[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m163[39m, end: [33m178[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m168[39m, end: [33m183[39m }

# Highly Available Grafana running on AWS Fargate and RDS Aurora



Grafana is a popular tool used to visualize time series data, but as with most open source tools, it is not highly available right out of the box (errr… container). Though with a few tweaks it is not hard to create a load balanced Grafana cluster utilizing the high availability and scalable features of Amazon Aurora. All without even having to run an EC2 instance.

Our team recently migrated [Sportsline.com](https://www.sportsline.com) from a datacenter VM infrastructure into AWS using Fargate, Elasticache, and Aurora. When it came to monitoring this new infrastructure, we wanted to maintain the no server approach that we were using for the web application. Having zero EC2 instances in an account that you manage makes life much easier, and now the operation folks can focus more on making the development team more efficient and not fool with patch management, ssh key rotation, ebs volume management, etc. I believe that containers as a service is the future, and there will be two camps. The ones that run large kubernetes clusters and those that allow the big public cloud providers run their containers for them.

We were already running an instance of Grafana on ECS for some of our monitoring of 247Sports.com, but we have to utilize a volume, mounted to the ECS host, for the purpose of maintaining the state of the SQLite database that Grafana uses by default. Another issue with using the local database is that you can only run a single instance of Grafana behind a load balancer. Most of the time this is sufficient, but when there is an issue going on and everyone on the team is trying to view the graphs, it can cause the container to get overwhelmed.

In this post, we will cover this [**Terraform Module](https://github.com/ulikabbq/grafana-fargate)**, setting up a fully operational Grafana that is load balanced and backed by Aurora (*default disclaimer: this module will not run on a free tier and it will cost you a small amount*). If you don’t really care how it is put together and you just want to run it, follow the [**README](https://github.com/ulikabbq/grafana-fargate/blob/master/README.md)**.

Here is a basic example of the module configuration:

<iframe src="https://medium.com/media/ed3a952c873782d2bc1f7fa91fd6ae1b" frameborder=0></iframe>

## The container setup

Grafana makes just about everything configurable with very straightforward environment variables. Running Grafana from the Grafana published image makes it really convenient to pass those variables in at run-time of the container.

We previously passed 20+ environment variables to the task definition, and we were storing the secrets in the aws parameter store, then retrieving those at the run-time of the container. The drawback is that we had to install python and the aws cli into the container, and it made things a bit heavier than we would like.

In comes [Chamber ](https://aws.amazon.com/blogs/mt/the-right-way-to-store-secrets-using-parameter-store/)to save us from bulky task definitions, and bulky containers. Chamber extracts parameters from the SSM Parameter Store dynamically as environment variables in the container. This method allows the configuration to be entirely managed by SSM, meaning that configuration changes just require relaunching the container to take effect, without changes to the task definition or Dockerfile.

We started using Chamber for some other projects and now we look to implement it anytime that we need to get environment variables into a container. You can read more about Chamber here— [https://github.com/segmentio/chamber](https://github.com/segmentio/chamber)

<iframe src="https://medium.com/media/85a1004076aa4d8c39099d9c698b22b7" frameborder=0></iframe>

To utilize the module, there are two Parameter Store Values that need to be created before the module is applied. We use KMS to encrypt and store the default admin password for Grafana, and the RDS password. Those values are manually entered using the cli.

    example:
    aws ssm put-parameter — name “/grafana/GF_SECURITY_ADMIN_USER” — type “SecureString” — value “admin_password”
     
    aws ssm put-parameter — name “/grafana/GF_DATABASE_PASSWORD” — type “SecureString” — value “database_password”

Chamber will retrieve all parameters from ssm that are prefixed with /grafana so to add your own Grafana configs, simply create SSM Parameters prefixed with /grafana, and they will be brought into Grafana at the next restart of the container.

There is also a nginx container that is deployed as a sidecar. The primary use of the nginx container is to redirect users to https. All requests are then passed to Grafana over port 3000.

<iframe src="https://medium.com/media/3df4d6870597a22a0750fb75ec9efa30" frameborder=0></iframe>

Both of these container images are hosted through the module, but they can be overwritten in the module configuration with your own images using the image_url and nginx_image_url variables.

## Fargate Setup

The Fargate setup is really straight forward, but isn’t that the point of Fargate.

The service definition for Fargate is where it differs a bit from regular ECS. In the Terraform we define the launch type as Fargate, and you apply the security group to the service network configuration.

<iframe src="https://medium.com/media/b49d3ff8f4de95a1e0b49d399e298739" frameborder=0></iframe>

The Fargate task is also configured to send all nginx and Grafana logs to Cloudwatch Logs. This is really the only way to debug any issues with the Fargate at this point.

## Aurora Setup

While we have this solution in Terraform and it is 98% automated, there is a manual step that you have to complete in order for the database to be setup. This also means that we need a temp ec2 instance to access the database and run the migration steps. The module has a variable for bastion count and once the database has been setup, this can be set to 0 to delete the instance.

Once logged into the bastion, connect to Aurora with the root user and the password that was created with the aws ssm put-parameter command. Once connected to Aurora, run the following commands with the database cluster endpoint and appropriate database password.

<iframe src="https://medium.com/media/0444301e98847bc729139abc4053a97f" frameborder=0></iframe>

## Grafana

Now we are able to add a new Cloudwatch Data Source. Use the arn of the Grafana role that was created in the account to allow access to the Cloudwatch metrics.

![](https://cdn-images-1.medium.com/max/2000/1*7WBbeBw0TciPwgtczgN2Yg.png)

Now your Cloudwatch data source has all it needs to monitor everything Cloudwatch related in your aws account.

Here is a really basic dashboard I was able to setup to monitor the performance of the Grafana setup.

![](https://cdn-images-1.medium.com/max/2498/1*xn633Bu8JqHQm1GF7WukqQ.png)

## IAM Setup

To monitor multiple accounts, create a Grafana role in the account to be monitored. Here is the Terraform for the Grafana role to deploy in other managed accounts. The aws_account_ids variable is the aws account id that Grafana is deployed in.

<iframe src="https://medium.com/media/db3866de15bd813a8f1a2352e1dd7ddf" frameborder=0></iframe>

Overall this setup is working really nice for us. I do have some issues from time to time with how Grafana works with Cloudwatch, but I am generally able to overcome those issues one way or another. After you deploy any project, there are inevitably changes that you want to make. Here are the next steps for this setup.

**Future updates:**

* move it to [Aurora Serverless ](https://aws.amazon.com/rds/aurora/serverless/)when that becomes GA

* store our Grafana data sources and dashboards as code

* create a Lambda function that will setup the database to remove the bastion dependency

In a future post we will cover how we use serverless http checks and write those results to Cloudwatch to monitor critical endpoints that our application relies upon.
