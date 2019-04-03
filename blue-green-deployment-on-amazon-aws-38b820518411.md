
# Blue Green Deployment on Amazon AWS.

Using Amazon AWS and Python.

## Introduction to components

In [Blue green deployment approach](https://martinfowler.com/bliki/BlueGreenDeployment.html) we create replica of the current production infrastructure and route all the traffic to the replica. The current infrastructure is termed as **Blue** while the replica is **Green**. Of course we make all code updates in the green fleet. This ensure high availability during deployments.

This article is about implementing blue green deployment using Amazon AWS and Python. We will use [Elastic Compute Cloud(EC2)](https://aws.amazon.com/ec2/?nc2=h_m1), [Autoscaling Groups](https://aws.amazon.com/autoscaling/), [Launch Configurations(LC)](http://docs.aws.amazon.com/autoscaling/latest/userguide/LaunchConfiguration.html), [Elastic Load Balancer(ELB)](https://aws.amazon.com/elasticloadbalancing/?nc2=h_m1), [Route 53](https://aws.amazon.com/route53/?nc2=h_m1), [Python Boto](http://boto.cloudhackers.com/en/latest/) and [Python Fabric](http://www.fabfile.org/).

![Fleet of application instances behind ELB, managed by ASG.](https://cdn-images-1.medium.com/max/4200/1*C5_IVHuyT5hRrmALI-lSEw.png)*Fleet of application instances behind ELB, managed by ASG.*

Our infrastructure contains a fleet of EC2 instances behind an ELB managed by an ASG. Route 53 acts as a gateway for users to interact with our service. All traffic to our domain reaches the ELB which distributes the load to the EC2 instances. ASG manages our cluster of instances. ASG can add an instance or remove it based on the configurable policies. ASG use [Launch Configurations (LC)](http://docs.aws.amazon.com/autoscaling/latest/userguide/LaunchConfiguration.html) and [Amazon Machine Images (AMI)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html). Launch configurations define which virtual image(AMI) to use while scaling out.

Python Boto and Fabric are being used to automate the process of deployments. Boto is a library to access Amazon AWS via python scripts. An alternative to Boto is [Amazon AWS CLI](https://aws.amazon.com/cli/). Fabric is another python library that comes handy to run deployment tasks on remote servers via SSH (concurrently!).

## Talk is cheap, show me the CODE!

There can be two approaches to implement blue green deployments.

1. We can replace the blue(exiting) EC2 instance with green(with latest code) EC2 instances. In this approach, we create the same amount of instance in the existing infrastructure, update the instances with new code and add them behind the load balancer and kill blue instances.

1. Or, we can make the switch on DNS level using Route 53. In this approach we create an exact replica of our infrastructure with updated code i.e. a new ASG, new ELB and new set of instances that will contain the latest code.

In the next section we give implementation details for both the approaches.

### Approach 1

![Add green instances code behind ELB and terminate blue instances.](https://cdn-images-1.medium.com/max/4322/1*wUCM4XlotIUbdn5iIPV0ZQ.png)*Add green instances code behind ELB and terminate blue instances.*

*Step 1.* We get the details of the production ASG. We define a function describe_asg which takes the name of the ASG and gets details using Boto.

<iframe src="https://medium.com/media/ff32e6a44b2b2d4d8b8757cbe7fa7cf6" frameborder=0></iframe>

*Step 2. *Using asg_details we can get the number of instances and also the LC name. We use LC name to describe details like AMI id, instance type etc. Now we spun off instances with exact same configuration. We will update these instances with latest code.

<iframe src="https://medium.com/media/b14a925a5b986a5faf3e194674560c5d" frameborder=0></iframe>

*Step 3. *We use Python Fabric to update code in new instances. With Fabric we define task to run on remote machines. We use env.rolesdef to define machine roles. Any task attached to a role will only be executed in a machine with same role. Here we are defining deployment role. We will add all new instances to this role. We can define tasks like update_code , validate_service etc.

<iframe src="https://medium.com/media/c5a339007c2d901fe1f86ff788d13ad6" frameborder=0></iframe>

*Step 4.* Now that we have the latest revision of code available in the deployment instances we need to bake an image. This image will be used by the ASG in case of a scale out. AWS provides a way to create image without the need for a server to reboot. We are using this option so that we don’t need to wait for service to be up and running before we attach it to the ASG in next steps.

<iframe src="https://medium.com/media/70d754f3cb04dbbcbd9d8d667dcdee34" frameborder=0></iframe>

*Step 5.* Now we have to attach the deployment instances to the ASG and ELB and terminate all the old instances. We have to attach the instances to ASG first and then to ELB as this is a precondition for ELB. While attaching the the ASG we may have to increase the max capacity of instances ASG can increase to scale out. When we attach the instances to ASG it increases the desired capacity. ASG throws an error if we try to add more instances than the max capacity. Also, we must check that instances in ELB pass health checks i.e. they are in service. If instances are not in service it indicates that some error has occurred and the deployment should **fail**.

<iframe src="https://medium.com/media/a5822273843293bb51784b6b1a675732" frameborder=0></iframe>

*Step 6.* If *Step 5. *succeeds we can now update the ASG to use the new image to spun off new instances. To update the ASG we will have to create a new LC which will be an updated copy of the old LC. We replace the image id with the newly formed AMI and attach new LC to the ASG.

<iframe src="https://medium.com/media/d6ae29444bdb79427647546eb2a0851b" frameborder=0></iframe>

*Step 7.* If all the above steps succeed then we will have a set of instances with old code and a set with new code. Now we need to terminate the old instances.

<iframe src="https://medium.com/media/e693a078b3662978ab404e200eea8fcf" frameborder=0></iframe>

So, we have replaced the infrastructure with new code successfully.

### Approach 2

![Create exact replica of the Blue ASG and switch traffic to it using Route 53.](https://cdn-images-1.medium.com/max/4410/1*RlCC1IC8QcmKN79y5sW9lA.png)*Create exact replica of the Blue ASG and switch traffic to it using Route 53.*

This approach is a simpler one as we depend on Amazon ASG to create the instances and attach them to the ELB. All we got to do is create an AMI with updated code and create an ASG.

To create and AMI with updated code we can follow the *Step 1, 2, 3 and 4 *of **Approach 1**. Note that in *Step 2 *we don’t need to create an exact number of instances we just need to create an AMI using only one deployment instance. Once that is done we may terminate the deployment instance using *Step 7 *of **Approach 1**.
> Creating ASG Replica

*Step 1. *First up we create a new LC using copy_lc_with_new_ami. We use the AMI with updated code in this LC.

*Step 2. *Now we create a new ELB that we will attach to the new ASG. This ELB has to be configured exactly as the older one. So we will copy the listeners, security_groups, subnets and health_checks .

<iframe src="https://medium.com/media/14f8897ae1f1e5e8aaea9c2328eba4ca" frameborder=0></iframe>

*Step 3. *We create a new ASG using the newly created ELB and Launch configuration. We use check_instances_in_service to validate success of this step.

<iframe src="https://medium.com/media/6a0d44c766e172cd61c9ed84077513b0" frameborder=0></iframe>

In order for ASG to scale in/out we need to [configure the scaling policies](http://docs.aws.amazon.com/autoscaling/latest/userguide/policy_creating.html#policy-updating-console). This can be done using AWS Cloudwatch Alarms and Simple Scaling Policies. So, we create a cloudwatch alarm which can monitor a metric and notify the scaling policy to perform the desired action.

<iframe src="https://medium.com/media/dbbd52fb2dc2421a0aea29c5452758de" frameborder=0></iframe>

*Step 4. *Now that ASG is set up and there are instances ready to serve traffic behind the new ELB last step is to switch the traffic using Route53. We will create an A record entry in Route53 to redirect all traffic to our desired domain to reach to green ELB’s DNS.

<iframe src="https://medium.com/media/b0ac9ff79bf67eb51b7966caba3940f8" frameborder=0></iframe>

*Step 5. *As traffic is now being served by the Green ASG we do not need the Blue one. We will set the minimum capacity of Blue ASG to zero. This will scale down the Blue ASG gracefully.

<iframe src="https://medium.com/media/e3010ff5c98d038d090845b96048f486" frameborder=0></iframe>

Thus we have successfully updated our infrastructure in Blue Green Approach. We can delete the Blue ASG(old) once sanity of the Green ASG has been ensured.

Follow us on [twitter](https://twitter.com/practodev) for regular updates. If you liked this article, please hit the ❤ button to recommend it. This will help other Medium users find it.
