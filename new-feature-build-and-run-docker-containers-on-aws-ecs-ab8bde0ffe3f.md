
# New feature: Build and run Docker containers on AWS ECS

We’re happy to announce that Buddy now supports deployments to the AWS ECS! Amazon Elastic Container Service is a scalable container orchestration service that lets you run Docker containers on the Amazon infrastructure.

With Buddy, you can easily create a delivery pipeline that will dockerize your application, push it to the ECR registry, and launch it on the ECS:

![](https://cdn-images-1.medium.com/max/2000/1*euDhBjfWfXQ_rShlLshfog.png)

The action registers a new revision of the task definition from a JSON file and updates the ECS service using this revision. It waits until the deployment has finished and fails in case there is an error during the deployment:

![](https://cdn-images-1.medium.com/max/2000/1*IsnVbrvKKorQg4Dtsx3_Sw.png)
> If you’re new to Docker and would to use it in your workflow, [this guide](https://buddy.works/guides/docker) will get you started.

## Other AWS integrations

If you are an AWS developer, you can use Buddy to automate your tasks across a series of services: from updating assets on S3, to EB & CodeDeploy uploads, running Lambda scripts and CLI commands, to purging CloudFront cache. Integration is very simple and requires authenticating in your AWS account with [proper policies](https://buddy.works/knowledge/deployments/list-of-aws-policies).

![](https://cdn-images-1.medium.com/max/2000/1*dZUHC_d2UadOkv1Tf1dXUA.png)
> If you didn’t find your integration on the list or have questions about reproducing your workflow in Buddy, drop a line in the comments or directly at [support@buddy.works](mailto:support@buddy.works).
