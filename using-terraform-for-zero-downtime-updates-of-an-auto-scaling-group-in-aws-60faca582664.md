
# Using Terraform for zero downtime updates of an Auto Scaling group in AWS

© pexels.com

A lot has been written about the benefits of immutable infrastructure. A brief version is that treating the infrastructure components as immutable helps us prevent configuration drift. After a component is deployed, it is never modified in place and can only be replaced.

Some infrastructure components don’t lend themselves too well to being managed this way — it’s quite hard and probably unnecessary to tear down and recreate the network layer or the database on each deployment, for example. On the other hand, this way of managing infrastructure works very well with servers. It’s important to prevent any downtime during such rollouts, and Terraform gives us a couple of ways of doing this.

## Blue / Green deployments

This approach was initially [suggested](https://groups.google.com/forum/#!msg/terraform-tool/7Gdhv1OAc80/iNQ93riiLwAJ) by Paul Hinze from Hashicorp.

The key element in this strategy is the interaction between the launch configuration and the Auto Scaling group using it. As a reminder, a [launch configuration](https://docs.aws.amazon.com/autoscaling/ec2/userguide/LaunchConfiguration.html) is a template that an Auto Scaling group uses to launch EC2 instances. It is immutable, so we can’t modify a launch configuration after it was created and we need to destroy it and create a new one instead.

In Terraform this is modelled using the **create_before_destroy** lifecycle setting. As we can’t create a new resource with the same name as the old one, we don’t hard-code the name and only specify the prefix. Terraform adds a random postfix to it, so the new configuration doesn’t clash with the old one before it is destroyed. In a simplified form this is how it can look:

<iframe src="https://medium.com/media/6bd0fedc5f32bbbbdc673ba419ae4494" frameborder=0></iframe>

Replacing the launch configuration of an Auto Scaling group by itself would not trigger any changes. New instances would be launched using the new configuration, but the existing instances [are not affected](https://docs.aws.amazon.com/autoscaling/ec2/userguide/change-launch-config.html). It is possible to force the Auto Scaling group to cycle the instances by adding some kind of post-deployment lambda function, but Terraform gives as a better option.

We can force the ASG resource to be inextricably tied to the launch configuration. To do this, we reference the launch configuration name in the name of the Auto Scaling group. Updating the name of an ASG [requires its replacement](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-as-group.html#cfn-autoscaling-autoscalinggroup-autoscalinggroupname), and the new Auto Scaling group would spin up its instances using the new launch configuration. To prevent Terraform from tearing down the old ASG before it creates the new one, we can use the same **create_before_destroy** parameter as we did with the launch configuration.

<iframe src="https://medium.com/media/ffc1fb8b6f9ef7ede1ec7d120cc189b5" frameborder=0></iframe>

This works beautifully, for the most part at least. Terraform creates a new Auto Scaling group and then, *when it’s ready* (we’ll get to this in a moment), swaps out the old one.

This approach is frequently called a “rolling” deployment, but this is not strictly correct. In a rolling deployment, the instances are replaced gradually, whereas here we see a complete replacement with an instant swap, which is a [classic form of Blue/Green](https://martinfowler.com/bliki/BlueGreenDeployment.html). This strategy also closely follows one of the patterns outlined in “Deep Dive into Blue/Green Deployments on AWS”, an excellent re:Invent talk by Andy Mui and Vlad Valsceanu.

![[AWS re:Invent 2015 | (DVO401) Deep Dive into Blue/Green Deployments on AWS](https://youtu.be/aX54mhZbN58?t=17m25s)](https://cdn-images-1.medium.com/max/2338/1*7vUIT3fqmcyRIRBWM28YmA.png)*[AWS re:Invent 2015 | (DVO401) Deep Dive into Blue/Green Deployments on AWS](https://youtu.be/aX54mhZbN58?t=17m25s)*
> One thing that is not specific to Terraform, but is still useful to keep in mind. When we briefly double the number of instances running during the Blue/Green deployment, we need to be sure that our account can support this. There are limits on the number of EC2 instances which we can run simultaneously, and even if we raise those limits by opening a ticket to AWS support, there may be cases where AWS is unable to supply the required number of instances of a given type in a region (yes, this does happen, AWS occasionally runs out of some instance types).

Unfortunately, as I hinted above, there are some issues with how Terraform handles this swap.

### Desired capacity

The desired capacity is an optional parameter, but if not set, it defaults to the minimum size value. So we need to provide this value to Terraform to actually populate our ASG with running servers, and the next question is how do we know how many instances we want? For static configurations, it’s ok to use a hard-coded value, but if the old Auto Scaling group had its capacity changed due to a scale in or scale out, we would lose this information. In extreme scenarios, this could lead to an outage.

As always, there are workarounds. Terraform is very extensible, and one solution would be to add an [external data source](https://www.terraform.io/docs/providers/external/data_source.html) that queries AWS API to find the current capacity and then passes it as a parameter to the new ASG.

### Health checks

This is a trickier one. Terraform is responsible for creating a new ASG, but it has no insight into whether the new instances actually work. This is a job for the Auto Scaling group itself, and it solves it using [health checks](https://docs.aws.amazon.com/autoscaling/ec2/userguide/healthcheck.html).

There are two main kinds of automatic health checks:

* [EC2 status checks](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/monitoring-system-instance-status-check.html) verify that an instance can boot, and that’s about it.

* [ELB health checks](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html) make sure that the target can respond to web requests within a specified timeout. If a server is not responding, it is marked as unhealthy and then terminated by the Auto Scaling group.

The ELB health checks can guarantee that the new launch configuration is valid and the app is running; however, they can only be used with web services. If you have a cluster of worker type instances, you’re pretty much out of luck. The Auto Scaling group has no way to verify that they are working as expected, which makes this strategy unsuitable for non-web workloads.

I’m sure there are dozens of ways to try and work around the health check issue, but this comes dangerously close to yak shaving, so let’s see if there’s another way.

## Rolling deployments

If we check how AWS CloudFormation approaches the same problem, we will find something interesting.

In CloudFormation, the **AutoScalingGroup** resource exposes **UpdatePolicy **— an attribute not available via the plain [Auto Scaling API](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_CreateAutoScalingGroup.html). This policy is triggered by updates to the Auto Scaling group (such as changing the Launch Configuration or the Launch Template) and provides granular control over how these updates are handled. One of the available policies is **AutoScalingRollingUpdate**, which determines whether AWS CloudFormation updates the instances in batches or all at once. Using this policy, we can control the batch size, the minimum number of instances we want to stay in service at any time, and a number of other useful parameters.

I’ve written about using this policy in my blog post about [zero downtime ECS cluster upgrades](https://devblog.xero.com/automating-ecs-cluster-upgrades-with-cloudformation-and-lambda-2b4f1bec575d), but here are the important bits.

### Desired capacity

Somewhere deep in the **UpdatePolicy **attribute is hidden a property called [**IgnoreUnmodifiedGroupSizeProperties](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-updatepolicy.html#cfn-attributes-updatepolicy-scheduledactions)**. When set to *true*, it tells CloudFormation to ignore the differences in group size between the current Auto Scaling group (actual configuration) and its template (desired configuration) during a stack update. In other words, CloudFormation will not try to reset the desired capacity to what’s defined in the template, unless the template itself has changed.

### Health checks

CloudFormation gives us access to [cfn-signal helper script](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-signal.html), which can be used to indicate whether Amazon EC2 instances have been successfully created. If we set **WaitOnResourceSignals **to *true*, all new instances *are considered unhealthy unless proven otherwise*. The instances are responsible for signalling to CloudFormation that they managed to complete user data scripts successfully within a specified timeout.

Sometimes instances fail due to transient errors (like network blips), and we can define what percentage of failed instances is acceptable before deciding to roll back by setting the **MinSuccessfulInstancesPercent **property.

Here’s how cfn-signal looks like in a user data script (notice the *set -euo pipefail* stanza at the very top, which would force the script to exit on the first error or unassigned variable):

<iframe src="https://medium.com/media/adf3e79d932dc80a308e4d7c5e67b24b" frameborder=0></iframe>

## Wait, isn’t this a post about Terraform?

It is indeed.

In most cases, I would take [Terraform over CloudFormation](https://medium.com/@endofcake/terraform-vs-cloudformation-1d9716122623), if only because of more pleasant and powerful syntax. And more pragmatically, it makes sense to choose one tool and stick with it, because it reduces the burden of maintaining the deployment pipelines. However, this seems to be a case where CloudFormation opens up possibilities not available with plain Terraform, so

![](https://cdn-images-1.medium.com/max/3176/1*ZAxb7rWoRk3uHdy7tXgEsw.jpeg)

We can define our Auto Scaling group as an **aws_cloudformation_stack **resource in Terraform, and it will look like this:

<iframe src="https://medium.com/media/7cef1a5a20d4f688ae0ec88fa21631a7" frameborder=0></iframe>
> CloudFormation is free, and by using it to manage the Auto Scaling groups in AWS, we are not increasing the vendor lock-in. So pragmatically, it is hard to find a reason not to leverage the functionality that is only available in CloudFormation. By embedding an **aws_cloudformation_stack **resource inside Terraform configuration, we get access to these capabilities, while still benefiting from the rich interpolation syntax and variable management in Terraform.

## Conclusion

When it comes to rolling out changes to Auto Scaling groups managed in Terraform, we have several options. Blue/Green deployments work well with web services, because ELB health checks provide a robust way to ensure that the service is operating correctly. Worker type workloads (such as ECS clusters) usually do not have a web endpoint which would allow us to use ELB health check. In such cases, we can fall back to using CloudFormation stack resource, with *cfn-signal* ensuring the health of the instances.
