
# Zero Downtime Deployments with AWS CodeDeploy

A CTO’s experience switching cloud providers from Azure to AWS and some hard-earned lessons from the trenches

![**AWS CodeDeploy** is a service that automates code deployments to any instance](https://cdn-images-1.medium.com/max/11522/1*JxzXw1jY8StM_5f9ofnzoQ.jpeg)***AWS CodeDeploy** is a service that automates code deployments to any instance*

We at [Gatherer](https://www.gatherer.at) have recently changed our cloud provider from Microsoft Azure to Amazon AWS, so it was up to me (the CTO) to setup a completely new infrastructure for staging and production.

This was my first time ever working with auto scaling groups, elastic load balancers and services for automated deployment. I learned a lot from the process, and want to share my experiences and point out some nasty obstacles I encountered.
> AWS CodeDeploy is a service that automates code deployments to any instance making it easier to avoid downtime during application deployment and handle the complexity of updating your applications.

## Setting up the stack

The first thing to do is setting up all the services your company requires, like virtual machines, message queues or relational databases. Now in my opinion here comes the very first little obstacle: Amazon has its own terminology for their services. For example virtual machines are called EC2 instances in the AWS world. Luckily I found a neat little website translating all of their services to terms I and probably everyone else understands. It’s called [Amazon Web Services in Plain English](https://www.expeditedssl.com/aws-in-plain-english) — go check it out!

### Creating AWS Services

There are several ways you can setup your infrastructure. For example there is the AWS Console which is basically the graphical user interface on the website or the API which can be accessed from everywhere including from inside your applications.

![](https://cdn-images-1.medium.com/max/2000/1*_Qvy3JXT5-lK7YaZfBKYwg.png)

### Terraform

I stumbled across an interesting tool called Terraform when inspecting repositories on Github. Terraform is a cloud-agnostic tool (works for multiple cloud providers like Azure or AWS) which allows you to orchestrate your infrastructure via code — also called IAT, Infrastructure As Code.

This approach allows you to write a script that boots up all the services you need and have it source controlled. With a little creativity you can create modules and setup multiple directories that host scripts for multiple environments. Boom! In case you are interested definitely check out [A comprehensive Guide to Terraform](https://blog.gruntwork.io/a-comprehensive-guide-to-terraform-b3d32832baca#.fbzsgpzfx).

## Auto Scaling Groups

Auto Scaling Groups are a wonderful service of AWS that allows us to automatically scale up or scale down our cluster of EC2 instances based on CPU or Memory consumption by using scaling policies. Now here comes a little catch. There are three variables that define the number of instances in the Auto Scaling Group: “Min”, “Max” and “Desired”.
> The first thing to keep in mind is that Desired can’t be lower than Min and greater than Max —it has to be in the range you specify.

When a EC2 instance becomes unhealthy (which usually happens when the elastic load balancer can’t ping it anymore) or an up-scaling policy gets triggered (for example high CPU usage) the Auto Scaling Group automatically launches a new instance and always takes care to not exceed the Max value you specified. However, it does not automatically change your Desired value. In case you also defined a down-scaling policy (which would make sense) and the CPU normalises the Auto Scaling Group would terminate instances until it reaches the Min or Desired Value.

## Creating Deployment Groups

When using CodeDeploy one has to create a “Deployment Group” and choose between “In-Place deployment” and “Blue/Green deployment”.

![](https://cdn-images-1.medium.com/max/2000/1*sKR9UscVj3BylhImVaXJ0w.png)

Blue/Green deployment basically copies your Auto Scaling Group and applies updates to the new instances. Once they are ready it destroys the old Auto Scaling Group and its instances. Unfortunately this option didn’t work out for us because we are using scaling-policies.

As of today CodeDeploy isn’t able to copy those policies when cloning the Auto Scaling Group. At least that was the error I got when setting up CodeDeploy with Blue/Green deployment. If I’m wrong here please let me know.

Using In-place deployment means that the instances directly get patched with the new application version — and while this is happening, we don’t want to receive requests we can’t handle at the moment. The goal here is to put the instance into standby for the time of the deployment and make use of the Elastic Load Balancer (ELB) by having it route the traffic to another active instance.

## Zero Downtime Deployments

Following the [official best-practices by Amazon](https://aws.amazon.com/blogs/devops/use-aws-codedeploy-to-deploy-to-amazon-ec2-instances-behind-an-elastic-load-balancer-2/) we have to put our instances to standby (deregister from ELB) and reactivate them (register with ELB) once the deployment is finished.

To accomplish this we have to use the BeforeInstall: and ApplicationStart: hooks in our appspec.yml file and call the respective shell scripts that can be downloaded on their Blog.

    hooks:
      BeforeInstall:
        - location: aws/deregister_from_elb.sh
          timeout: 400
      ApplicationStart:
        - location: aws/register_with_elb.sh
          timeout: 120

What exactly happens here? If we take a closer look into the common_functions.sh file that has been shipped with the archive we see that before entering standby the script is also decrementing the Min value of our Auto Scaling Group. This should be around line 120.

    msg "Decrementing ASG $asg_name's minimum size to $new_min"
    msg $($AWS_CLI autoscaling update-auto-scaling-group \
        --auto-scaling-group-name $asg_name \
        --min-size $new_min)

This makes absolute sense, as we don’t want the Auto Scaling Group to fire up another instance just because we put one to standby. A few lines below we can find the command that finally makes the API call. Taking a closer look we see an important argument:--should-decrement-desired-capacity

    msg "Putting instance $instance_id into Standby"
    $AWS_CLI autoscaling enter-standby \
        --instance-ids $instance_id \
        --auto-scaling-group-name $asg_name \
        --should-decrement-desired-capacity

So what’s happening here? When putting an instance to standby, we also decrement the Min and Desired values of the Auto Scaling Group. Imagine you configured Min:1 Max:2 and Desired:1 and therefore have one running instance that you want to deploy to. Running the script would result into Min:0 Max:2 and Desired:0 and means that the Auto Scaling Group won’t fire up another instance with your old code base as long as you are deploying.

### **Problems & Solutions**

I ran into a few other problems. For instance, the blog article doesn’t state that the [CodeDeploy policy used by the EC2 instance profile](http://docs.aws.amazon.com/codedeploy/latest/userguide/how-to-create-iam-instance-profile.html#getting-started-create-ec2-role-console) needs additional permissions like updating the Auto Scaling Group.

I had lots of time consuming feedback cycles with my EC2 machines until I figured out all necessary permissions for deploying. Here they are:

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": [
                    "s3:Get*",
                    "s3:List*",
                    "autoscaling:DescribeAutoScalingGroups",
                    "autoscaling:DescribeAutoScalingInstances",
                    "autoscaling:UpdateAutoScalingGroup",
                    "autoscaling:EnterStandby",
                    "autoscaling:ExitStandby"
                ],
                "Effect": "Allow",
                "Resource": "*"
            }
        ]
    }

The second problem was that after my application got deployed and my instance was activated and attached to the ELB again, I ended up with an Auto Scaling Group with Min:0 Max:2 and Desired:1. The newly returned instance was active for about one minute and then suddenly got terminated.

Reading in the [official documentation](http://docs.aws.amazon.com/autoscaling/latest/userguide/as-enter-exit-standby.html) I found out that it’s completely normal that the Desired value gets incremented when an instance is put back into InService, so this explains the change from 0 to 1. Makes sense. However, I had no idea why the instance got terminated so quickly.
> After ordering a pizza while repeating the process it finally crossed my mind. Min:0 in combination with my scale-down policy (CPU) was the culprit!

Checking the autoscaling_exit_standby() function I found out that the script is never incrementing Min again which ends up in an inconsistent state, especially considering my Terraform setup. A small patch between exiting standby and waiting for the instance to make it to InService did the trick:

    ...
    local min_cap=$(echo $min_desired | awk '{print $1}')
    local desired_cap=$(echo $min_desired | awk '{print $2}')

    if [ -z "$min_cap" -o -z "$desired_cap" ]; then
        msg "Unable to determine minimum and desired capacity for ASG $asg_name."
        return 1
    fi
        
    local new_min=$(($min_cap + 1))
    msg "Incrementing ASG $asg_name's minimum size to $new_min"
    msg $($AWS_CLI autoscaling update-auto-scaling-group \
        --auto-scaling-group-name $asg_name \
        --min-size $new_min)
    if [ $? != 0 ]; then
        msg "Failed to increase ASG $asg_name's minimum size to $new_min. Cannot put this instance out of Standby."
        return 1
    fi
    ...

Looking good!

![](https://cdn-images-1.medium.com/max/2000/1*1w8p3LeYFi3FJs3mT_wuig.png)

## Learning the Basics is Key

Knowing the basics in shell programming helps a lot when using external scripts. While working with AWS can be quite overwhelming at the beginning, once you understand the terminology and IAM system with all its roles and policies — it gets smoother and smoother.
> Check out the [AWS courses from A Cloud Guru](https://acloud.guru/courses) to level-up your cloud computing skills, or visit their community [forums](https://acloud.guru/forums/home) to connect with industry experts.
> The [A Cloud Guru course on AWS CodeDeploy](https://acloud.guru/learn/aws-codedeploy) will guide you through the basic code deployment use cases to advanced zero-downtime patching of your app.
