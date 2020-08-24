
# 6 reasons for using the AWS DevOps tools

6 reasons for using the AWS DevOps tools

AWS provides a set of services for supporting the DevOps tool chain. These services are [CodeCommit](https://aws.amazon.com/codecommit/), [CodePipeline](https://aws.amazon.com/codepipeline/), [CodeBuild](https://aws.amazon.com/codebuild/) and [CodeDeploy](https://aws.amazon.com/codedeploy/). With these tools it’s easy to run the complete DevOps Pipeline on AWS. Why should you use these tools instead of the myriad other services provided by other providers? Here are the 6 main reasons I always reiterate when discussing this topic with colleagues and clients.

## Fully Managed

As developers, we want to focus on building things. We don’t want to waste our time on maintaining the platform and tools which are there to support us.

With the Code*-Tools offered by AWS this is exactly the case. All of them are fully managed. One could also argue that they are *Serverless* to use another buzzword. This means that no servers need to be set up and maintained by the users of these services. Therefore, as users of the tools the developers are not responsible for patching operating systems, updating software, establish a disaster recovery strategy, etc. Instead, the tools are available to support us in doing our work more efficiently.

## Scaleable & Highly Available

The managed services provide another benefit besides freeing the users up from doing maintenance work: They are scalable and highly available by default. What does this mean?

When we set up the tools, like a CI server, we would get some server to run it on. Most of the time this server would be idle because there’s no new code to integrate. At other times the server might be fully utilized for building our project. Maybe more and more projects need to be build by the server over time which would mean that the periods of maximum utilization are longer. This in turn would mean that the developers need to wait longer for their integration builds to finish and the results to come in for review.

Furthermore, if the server fails for some reason or actual maintenance work takes place the service becomes unavailable. This means that the developers can not work as efficiently as before anymore.

Therefore, the tools would need to be set up in a scalable and highly available fashion. Of course, the costs for this kind of setup are very high. And it will be hard to argue why these investments are necessary and actually provide value. Actually, setting these things up and maintaining it does not provide any real value in the first place.

On the other hand, the tools offered by AWS are scalable and highly available by default. Nothing needs to be considered in addition to get these features.

## Pay as you go

Like many other cloud services the Code*-Tools offered by AWS — CodeCommit, CodePipeline, CodeBuild, CodeDeploy — are charged by the amount of usage. This means that the more you use them the higher the costs are. Of course, this also means that if you don’t use them very frequently the costs are close to zero.

For example, the [pricing model of CodeBuild](https://aws.amazon.com/codebuild/pricing/) is based on the minutes a build takes. The service offers three tiers of capacity — basically computing power — which are priced at $0.005, $0.01 and $0.02 per build minute.

Let’s say you would set up a t3.small server and run it continuously it would cost you $17,52 per month on average. With this you could get 3.500 build minutes using the smallest tier. If one build takes 10 minutes on average this means 350 builds per month. With a 5 day work week it would result in around 17 builds per day.

Of course you can optimize the costs for running a build server. For example, the server can be shut down on weekends or at night. Additionally, it might also be sufficient to use a smaller instance size or utilize spot instances for running the builds. But all of these things need to be set up and maintained. Of course that means that this time can not be used for building new features for your software and therefore creating actual value.

As with many AWS services the Code*-Tools provide a . This means that you get a certain amount of service for free. In case of CodeBuild this means 100 build minutes per month for the smallest compute tier. This means that it’s actually pretty cheap to get started and smaller projects with less frequent changes might actually cause no costs.

## Flexibility

The feature set of the Code*-Tools provided by AWS is actually quite limited when compared to the services of other providers. On the other hand they can be extended very easily with the capabilities of the AWS platform itself. All of the tools are either integrated with [CloudWatch Events](https://aws.amazon.com/cloudwatch/), [SNS](https://aws.amazon.com/sns/) or can directly call arbitrary code running in [Lambda](https://aws.amazon.com/lambda/). Oftentimes, more than one of these options exist.

This way it is possible to add almost anything to the tools that is needed for better customizing the tools for the development workflow or integrating it with other tools and services. As an example, I’ve already presented a solution for [updating the GitHub Commit Status](https://scratchpad.blog/2019/03/16/integrating-aws-codepipeline-with-the-github-commit-status-api.html) when a pipeline is run.

## Security

Of course, the Code*-Tools are well-integrated into the AWS platform itself. We have seen this with the flexible extension points which make it possible to do all kinds of things based on events triggered by the services.

Another important aspect is the integration with the [*Identity and Access Management* (IAM)](https://aws.amazon.com/iam/) service. In order for the services to do things — like putting artifacts into S3 or ECR or deploying applications to a fleet of servers — explicit permission must be granted via so-called *Service Roles*. Other external services often require to configure access to the AWS platform, e.g. for deploying your application, by providing secrets like Access Keys.

The same is true for actual users doing things with the tool chain, like changing the configuration or doing approvals for production deployments. This fine-grained access control makes it possible to make things pretty secure, especially in large teams or enterprises with tight compliance controls. Other services oftentimes allow only more coarse grained access control.

Of course, the downside is that it takes some effort to establish it.

## Easy Setup

The easiest way for setting things up is through the AWS Console. It provides a UI for configuring each of the services. This way you can get started very quickly.

Furthermore, the services can also be set up and configured programmatically, either through the CLI, SDKs or [CloudFormation](https://aws.amazon.com/cloudformation/). Of course it takes some time to get used to either the API or CloudFormation. On the other hand it helps to set things up quickly and repeatably for future projects. This especially includes setting things up securely from the start by configuring the appropriate access controls.

Over the time I already have collected a set of different [CloudFormation templates](https://github.com/jenseickmeyer/cloudformation-templates/tree/master/ci-cd) to cover all kinds of use cases and act as a foundation for setting up the DevOps Pipeline for future projects.

## Summary

I’ve presented the 6 main reasons why everyone who is developing applications which are deployed to AWS should take a look at the DevOps tools provided by the platform. This is especially true for anyone who has not yet done any (serious) automation regarding building, testing and deploying their applications. In my opinion it’s easy to take the first steps for achieving this automation. Because of the free tier it probably does not even increase the AWS bill for the first evaluations of the services.

If you need any help or struggle with setting things up I’m happy to help you out. Just reach out to me via one of the [social media](https://scratchpad.blog/about/) channels.

*Originally published at [scratchpad.blog](https://scratchpad.blog/devops/6-reasons-for-using-the-aws-devops-tools/) on May 25, 2019.*
