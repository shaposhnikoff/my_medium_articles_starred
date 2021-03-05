
# Updated: Highly Available Grafana running on AWS Fargate and RDS Aurora



***Updated on 12.01.2019***

*Since it has been over a year since I last visited this project, I wanted to add a few updates.*

*Upgraded to Terraform 0.12*

*Added GitHub Actions to publish and deploy the fargate container*

*Grafana version 6.5.1*

*Removed the bastion host (no longer need to run the mysql commands)*

*Removed nginx (was handling https redirects that are now in the lb listener)*

*Added IAM module to setup the Grafana role in the monitored accounts*

Grafana is a popular tool used to visualize time series data, but as with most open source tools, it is not highly available right out of the box (errr… container). Though with a few tweaks it is not hard to create a load balanced Grafana cluster utilizing the high availability and scalable features of Amazon Aurora. All without even having to run an EC2 instance.

*Update: Grafana 6.2 and up changed the way sessions are handled and it makes scaling up and running multiple containers even easier. By default now the sessions are handled in the default database.*

Our team recently migrated [Sportsline.com](https://www.sportsline.com) from a datacenter VM infrastructure into AWS using Fargate, Elasticache, and Aurora. When it came to monitoring this new infrastructure, we wanted to maintain the no server approach that we were using for the web application. Having zero EC2 instances in an account that you manage makes life much easier, and now the operation folks can focus more on making the development team more efficient and not fool with patch management, ssh key rotation, ebs volume management, etc. I believe that containers as a service is the future, and there will be two camps. The ones that run large kubernetes clusters and those that allow the big public cloud providers run their containers for them.

We were already running an instance of Grafana on ECS for some of our monitoring of 247Sports.com, but we have to utilize a volume, mounted to the ECS host, for the purpose of maintaining the state of the SQLite database that Grafana uses by default. Another issue with using the local database is that you can only run a single instance of Grafana behind a load balancer. Most of the time this is sufficient, but when there is an issue going on and everyone on the team is trying to view the graphs, it can cause the container to get overwhelmed.

In this post, we will cover this [**Terraform Module](https://github.com/ulikabbq/grafana-fargate)**, setting up a fully operational Grafana that is load balanced and backed by Aurora (*default disclaimer: this module will not run on a free tier and it will cost you a small amount*). If you don’t really care how it is put together and you just want to run it, follow the [**README](https://github.com/ulikabbq/grafana-fargate/blob/master/README.md)**.

Here is a basic example of the module configuration:

<iframe src="https://medium.com/media/bf500ab47c3954bc924925a5ca4314af" frameborder=0></iframe>

## The container setup

Grafana makes just about everything configurable with very straightforward environment variables. Running Grafana from the Grafana published image makes it really convenient to pass those variables in at run-time of the container.

We previously passed 20+ environment variables to the task definition, and we were storing the secrets in the aws parameter store, then retrieving those at the run-time of the container. The drawback is that we had to install python and the aws cli into the container, and it made things a bit heavier than we would like.

In comes [Chamber ](https://aws.amazon.com/blogs/mt/the-right-way-to-store-secrets-using-parameter-store/)to save us from bulky task definitions, and bulky containers. Chamber extracts parameters from the SSM Parameter Store dynamically as environment variables in the container. This method allows the configuration to be entirely managed by SSM, meaning that configuration changes just require relaunching the container to take effect, without changes to the task definition or Dockerfile.

We started using Chamber for some other projects and now we look to implement it anytime that we need to get environment variables into a container. You can read more about Chamber here— [https://github.com/segmentio/chamber](https://github.com/segmentio/chamber)

<iframe src="https://medium.com/media/2f1ec800540a5bdecea4294a289cdfa8" frameborder=0></iframe>

To utilize the module, there are two Parameter Store Values that need to be created before the module is applied. We use KMS to encrypt and store the default admin password for Grafana, and the RDS password. Those values are manually entered using the cli.

    example:
    aws ssm put-parameter — name “/grafana/GF_SECURITY_ADMIN_USER” — type “SecureString” — value “admin_password”
     
    aws ssm put-parameter — name “/grafana/GF_DATABASE_PASSWORD” — type “SecureString” — value “database_password”

Chamber will retrieve all parameters from ssm that are prefixed with /grafana so to add your own Grafana configs, simply create SSM Parameters prefixed with /grafana, and they will be brought into Grafana at the next restart of the container.

## Fargate Setup

The Fargate setup is really straight forward, but isn’t that the point of Fargate.

The service definition for Fargate is where it differs a bit from regular ECS. In the Terraform we define the launch type as Fargate, and you apply the security group to the service network configuration.

<iframe src="https://medium.com/media/ff7895828110d25dc361b09b9fa7d81f" frameborder=0></iframe>

The Fargate task is also configured to send all Grafana logs to Cloudwatch Logs. This is really the only way to debug any issues with the Fargate at this point.

*Update: [https://aws.amazon.com/blogs/aws/announcing-firelens-a-new-way-to-manage-container-logs/](https://aws.amazon.com/blogs/aws/announcing-firelens-a-new-way-to-manage-container-logs/)*

## Grafana

Now we are able to add a new Cloudwatch Data Source. Use the arn of the Grafana role that was created in the account to allow access to the Cloudwatch metrics.

![](https://cdn-images-1.medium.com/max/2000/1*7WBbeBw0TciPwgtczgN2Yg.png)

Now your Cloudwatch data source has all it needs to monitor everything Cloudwatch related in your aws account.

Here is a really basic dashboard I was able to setup to monitor the performance of the Grafana setup.

![](https://cdn-images-1.medium.com/max/2498/1*xn633Bu8JqHQm1GF7WukqQ.png)

## IAM Setup

To monitor multiple accounts, create a Grafana role in the account to be monitored. Here is the Terraform for the Grafana role to deploy in other managed accounts. The grafana_account_id variable is the aws account id that Grafana is deployed in. Make sure you add the monitored accounts id to the grafana module aws_account_ids.

<iframe src="https://medium.com/media/3567f6d823560d4be82c20605e755f8f" frameborder=0></iframe>

Overall this setup is working really nice for us. I do have some issues from time to time with how Grafana works with Cloudwatch, but I am generally able to overcome those issues one way or another. After you deploy any project, there are inevitably changes that you want to make.
