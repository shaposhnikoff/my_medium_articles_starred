
# 100 Days of DevOps — Day 39-Introduction to VPC EndPoint

Welcome to Day 39 of 100 Days of DevOps, Focus for today VPC Endpoint
> *What is VPC EndPoint?*

*A VPC endpoint enables you to privately connect your VPC to supported AWS services and VPC endpoint services powered by PrivateLink without requiring an internet gateway, NAT device, VPN connection, or AWS Direct Connect connection. Instances in your VPC do not require public IP addresses to communicate with resources in the service. Traffic between your VPC and the other service does not leave the Amazon network.*

*Endpoints are virtual devices. They are horizontally scaled, redundant, and highly available VPC components that allow communication between instances in your VPC and services without imposing availability risks or bandwidth constraints on your network traffic.*

*There are two types of VPC endpoints:*

* ***Interface endpoints**(using private links): An interface endpoint is an elastic network interface with a private IP address that serves as an entry point for traffic destined to a supported service*

    *# Supported Services*

    [*Amazon API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-private-apis.html)*

    [*AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-vpce-bucketnames.html)*

    [*Amazon CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch-and-interface-VPC.html)*

    [*Amazon CloudWatch Events](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/cloudwatch-events-and-interface-VPC.html)*

    [*Amazon CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/cloudwatch-logs-and-interface-VPC.html)*

    [*AWS CodeBuild](https://docs.aws.amazon.com/codebuild/latest/userguide/use-vpc-endpoints-with-codebuild.html)*

    [*AWS Config](https://docs.aws.amazon.com/config/latest/developerguide/config-VPC-endpoints.html)*

    *Amazon EC2 API*

    *Elastic Load Balancing API*

    [*Amazon Elastic Container Registry](https://docs.aws.amazon.com/AmazonECR/latest/userguide/vpc-endpoints.html)*

    [*Amazon Elastic Container Service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/vpc-endpoints.html)*

    [*AWS Key Management Service](https://docs.aws.amazon.com/kms/latest/developerguide/vpc-endpoint.html)*

    [*Amazon Kinesis Data Streams](https://docs.aws.amazon.com/streams/latest/dev/vpc.html)*

    [*Amazon SageMaker and Amazon SageMaker Runtime](https://docs.aws.amazon.com/sagemaker/latest/dg/interface-vpc-endpoint.html)*

    [*Amazon SageMaker Notebook Instance](https://docs.aws.amazon.com/sagemaker/latest/dg/notebook-interface-endpoint.html)*

    [*AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotation-network-rqmts.html)*

    [*AWS Security Token Service](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_sts_vpce.html)*

    *AWS Service Catalog*

    [*Amazon SNS](https://docs.aws.amazon.com/sns/latest/dg/sns-vpc.html)*

    [*Amazon SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-vpc-endpoints.html)*

    [*AWS Systems Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-setting-up-vpc.html)*

    [*Endpoint services](https://docs.aws.amazon.com/vpc/latest/userguide/endpoint-service.html) hosted by other AWS accounts*

    *Supported AWS Marketplace partner services*

* ***Gateway endpoints:** A gateway endpoint is a gateway that is a target for a specified route in your route table, used for traffic destined to a supported AWS service*

    *# Supported Services*

    ** Amazon S3
    * DynamoDB*

***Scenario1:** I want to push logs from EC2 private instance(running on Private IP)to CloudWatch Logs.*

* *To setup VPC Endpoint*

    *Go to [https://us-west-2.console.aws.amazon.com/vpc](https://us-west-2.console.aws.amazon.com/vpc) --> EndPoints*

![](https://cdn-images-1.medium.com/max/3752/1*MeVW4tuL4joWPO-hG-QQpQ.png)

![](https://cdn-images-1.medium.com/max/3628/1*u5aKQFyPKIpGDH_YOGRkhg.png)

![](https://cdn-images-1.medium.com/max/3448/1*i3IP-n7QRyeCaTJpjU1ljQ.png)

* *Once the endpoint is created you will see an elastic network interface with a private IP address which acts as an entry point for traffic destined to a supported service*

![](https://cdn-images-1.medium.com/max/5208/1*pQXKPI7D2XaTfbIKmN9MiQ.png)

*Terraform Code*

<iframe src="https://medium.com/media/2a2ccce8a95ce110f0bcafce44eda29c" frameborder=0></iframe>

***Scenario2:** I want to push logs from EC2 private instance(running on Private IP)to AWS S3.*

![](https://cdn-images-1.medium.com/max/4192/1*9XaeizghWcTKPIxVMxWhHQ.png)

![](https://cdn-images-1.medium.com/max/3904/1*ZbKwyF15qCFut53JNWeQLw.png)

* *In the case of gateway endpoint, you will see the entry in the route table, used for traffic destined to a supported AWS service*

***Terraform Code***

<iframe src="https://medium.com/media/e60d0f4588f8a48acc7dc59e999ec92b" frameborder=0></iframe>

*GitHub Link*
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/tree/master/vpc_endpoint)
> *Limitations*

* *Only Support IPv4*

* *Support only for the same region*

* *Interface endpoint cannot only be accessible via VPC Peering or VPN connection only via Direct Connect.*

*Looking forward from you guys to join this journey and spend a minimum an hour every day for the next 100 days on DevOps work and post your progress using any of the below medium.*

* *Twitter: [@100daysofdevops](http://twitter.com/100daysofdevops) OR @[lakhera2015](https://twitter.com/lakhera2015)*

* *Facebook: [https://www.facebook.com/groups/795382630808645/](https://www.facebook.com/groups/795382630808645/)*

* *Medium: [https://medium.com/@devopslearning](https://medium.com/@devopslearning)*

* *Slack: [https://devops-myworld.slack.com/messages/CF41EFG49/](https://devops-myworld.slack.com/messages/CF41EFG49/)*

* *GitHub Link:[https://github.com/100daysofdevops](https://github.com/100daysofdevops)*

***Reference***
[**100 Days of DevOps — Day 0**
*D-day is just one day away and finally, this is a continuation of the post(I posted a month earlier)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-0-4f2c9750542d)
[**100 Days of DevOps**
*Motivation*medium.com](https://medium.com/@devopslearning/100-days-of-devops-81faf13bf772)
