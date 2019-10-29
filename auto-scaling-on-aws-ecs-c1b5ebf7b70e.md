
# Auto Scaling on AWS ECS

As part of our growth in The Appraisal Lane, we have been digging into Automatic Scaling of infrastructure and services to be ready for whats to come.

In the path to auto scaling, we’ve investigated and tested different approaches on how to scale a cluster running in AWS ECS. I will try to explain how it is possible to automate both the Scaling Up and Down of such systems.

* This work asumes your ECS is mapped to a Auto Scaling Group (ASG).

First thing we need to know is that Auto Scaling does not only mean Scaling up (adding resources) in order to respond to peaks, but also includes scaling down, so that when you are not under peaks, you can save some resources.

Another thing to take into consideration, is that we need to scale up and down two different resources:

* Instances in the ASG

* Tasks within the Service

### Scaling Services

There is one assumption that makes scaling of services really simple:

* We always have space to fit in the highest memory/cpu container (we will see how to accomplish this when scaling instances)

With this in place we just need to accomplish the following:

* Define all services that we want to auto scale as “Scalable”. The minimum capacity will be respected even though you fire any scaling down policy.

![This is an example using Python and boto3 scaling client, this can be done in the ECS dashboard.](https://cdn-images-1.medium.com/max/2000/1*uLgoPEVNlSGtV5oPCpxYaQ.png)*This is an example using Python and boto3 scaling client, this can be done in the ECS dashboard.*

* Then we need to define the Scaling Policies for the Service, we will be using Step Scaling but not going to really use any other step other than +1 or -1 as this is enough for our requirements

![Example using Python and boto3 scaling client. This method allows defining both positive adjustments and negative adjustments. This call also be done in the ECS dashboard.](https://cdn-images-1.medium.com/max/2000/1*OktsXlRzZjgPHTAYjVfi9A.png)*Example using Python and boto3 scaling client. This method allows defining both positive adjustments and negative adjustments. This call also be done in the ECS dashboard.*

* Last you will need to define CloudWatch Alarms to monitor your CloudWatch Metrics that measure CPU/Memory for a specific service. You will need 2 different alarms, one for High Memory/Cpu usage, that will fire the ScalingUp policy, and another for Low Memory/Cpu usage to call the ScalingDown policy. This can both be done in the AWS CloudWatch Dashboard or using boto3.

### Scaling Instances

In the Service Scaling we assumed that we always have space to fit a new container in the cluster. This is because we are Scaling Instances up every time we determine we can not fit a new container.

With scaling instances, the Scaling Down needs special attention, as you will not want to bring down an instance that is running a task that is not running in any other instance in the cluster. We always want to be 100% available when also scaling down. To do this, AWS allows you to safely “DRAIN” an instance, moving each of the tasks on that instance to other one in the cluster, and stopping the tasks once the new one is already available.

**Scaling Instances Up**

As we stated before, scaling instances up is all about always being able to fit the biggest container at any moment. To do so, we will need to somehow measure if we can or can not fit our biggest container into the cluster instances.

We basically need something that checks each of our instances in the cluster, and check how many of our biggest containers we can fit. To do so, we can use Lambda.

The Lambda Function will run periodically every 1 minute, obtaining the total amount of containers we can fit on each of the instances, and then publish that value into a new CloudWatch Metric that we will call SchedulableContainers. This solution is taken from [here](http://garbe.io/blog/2017/04/12/a-better-solution-to-ecs-autoscaling/) and has been written by Philipp Garbe.

**Scaling Instances Down**

Following up with the Scaling Up, Philipp Garbe thought on how to Scale down an instance with this same Metric we created for the Scaling Up. But as we mentioned before, we are not going to just directly terminate instances once we want to scale down.

We will use the ASG LifeCycle Hook, a SNS Topic (that will be fired by the hook) and a Lambda Function to set the instance to “DRAIN” and then monitor it until it’s finished.

This solution is described in the AWS blog “[How to Automate Container Instance Draining in Amazon ECS](https://aws.amazon.com/blogs/compute/how-to-automate-container-instance-draining-in-amazon-ecs/)”

Basically, when the CloudWatch Alarm for Scaling Down is activated, we hit the LifeCycle hook to run hit the SNS with the corresponding params. The SNS fires the Lambda function, and on the first run, it will set the corresponding instance to “DRAINING” and then hit the SNS so that the Lambda is fired again. We will enter a loop, until the instance has no more tasks running or until the TTL of the hook has expired. If we determine that the instance has no more tasks running, we then execute the terminate.

![The lambda function for this can be found in the link above.](https://cdn-images-1.medium.com/max/2880/0*hevJo8ywXb-eNFdV.)*The lambda function for this can be found in the link above.*

### Conclusions

Auto Scaling the system is not only important to support all the peaks your system has, but also to reduce costs when the resources are not used as much.

![Example on how Auto Scaling can benefit an organization](https://cdn-images-1.medium.com/max/2000/1*HIwkdGpDmFCuyZ7xg_giGQ.png)*Example on how Auto Scaling can benefit an organization*

Auto Scaling also allows you to be prepared for unexpected traffic during special occasions or future growth.

Even though full Auto Scaling is not out of the box, it is not that difficult to achieve, and there are plenty of ways on how to do it. We’ve implemented a mix of solutions that best fit our needs.
