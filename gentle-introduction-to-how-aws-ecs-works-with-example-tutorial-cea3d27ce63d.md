Unknown markup type 10 { type: [33m10[39m, start: [33m64[39m, end: [33m73[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m33[39m, end: [33m43[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m106[39m, end: [33m115[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m98[39m, end: [33m108[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m114[39m, end: [33m131[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m308[39m, end: [33m328[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m8[39m, end: [33m14[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m34[39m, end: [33m43[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m31[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m55[39m, end: [33m64[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m128[39m, end: [33m137[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m248[39m, end: [33m257[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m157[39m, end: [33m169[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m126[39m, end: [33m142[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m192[39m, end: [33m207[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m28[39m, end: [33m38[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m11[39m, end: [33m26[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m118[39m, end: [33m133[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m15[39m, end: [33m24[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m73[39m, end: [33m82[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m132[39m, end: [33m165[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m176[39m, end: [33m182[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m5[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m13[39m, end: [33m23[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m41[39m, end: [33m51[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m61[39m, end: [33m71[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m16[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m30[39m, end: [33m39[39m }

# Gentle Introduction to How AWS ECS Works with Example Tutorial

Gentle Introduction to How AWS ECS Works with Example Tutorial

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/P2gGjvM8liU" frameborder="0" allowfullscreen></iframe></center>

[ECS](https://aws.amazon.com/ecs/) is the AWS Docker container service that handles the orchestration and provisioning of Docker containers. This is a beginner level introduction to AWS ECS. I’ve seen some [nightmare](https://news.ycombinator.com/item?id=12058929) posts and some [glowing](https://segment.com/blog/rebuilding-our-infrastructure/) reviews about the ECS service so I knew it was going to interesting to get my hands dirty and see what ECS was all about.

## Summary of the ECS Terms

First we need to cover ECS terminology:

* Task Definition — This a blueprint that describes how a docker container should launch. If you are already familiar with AWS, it is like a LaunchConfig except instead it is for a docker container instead of a instance. It contains settings like exposed port, docker image, cpu shares, memory requirement, command to run and environmental variables.

* Task — This is a running container with the settings defined in the Task Definition. It can be thought of as an “instance” of a Task Definition.

* Service — Defines long running tasks of the same Task Definition. This can be 1 running container or multiple running containers all using the same Task Definition.

* Cluster — A logic group of EC2 instances. When an instance launches the ecs-agent software on the server registers the instance to an ECS Cluster. This is easily configurable by setting the ECS_CLUSTER variable in /etc/ecs/ecs.config described [here](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/launch_container_instance.html).

* Container Instance — This is just an EC2 instance that is part of an ECS Cluster and has docker and the [ecs-agent](https://github.com/aws/amazon-ecs-agent) running on it.

I remember when I first got introduced to the all the terms, I quickly got confused. AWS provides nice [detailed diagrams](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html) to help explain the terms. Here is a simplified diagram to help visualize and explain the terms.

![ECS Terms](https://cdn-images-1.medium.com/max/2000/1*k29gxIwwhDaP-Ge-G-yXCQ.png)*ECS Terms*

In this diagram you can see that there are 4 running Tasks or Docker containers. They are part of an ECS Service. The Service and Tasks span 2 Container Instances. The Container Instances are part of a logical group called an ECS Cluster.

I did not show a Task Definition in the diagram because a Task is simply an “instance” of Task Definition.

## Tutorial Example

In this tutorial example I will create a small Sinatra web service that prints the meaning of life: [42](https://en.wikipedia.org/wiki/42_(number)).

1. Create ECS Cluster with 1 Container Instance

1. Create a Task Definition

1. Create an ELB and Target Group to later associate with the ECS Service

1. Create a Service that runs the Task Definition

1. Confirm Everything is Working

1. Scale Up the Service to 4 Tasks.

1. Clean It All Up

The [ECS First Run Wizard](https://console.aws.amazon.com/ecs/home#/firstRun) provided in the [Getting Started with Amazon ECS documentation](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_GetStarted.html) performs the similar above with a CloudFormation template and ECS API calls. I’m doing it out step by step because I believe it better helped me understand the ECS components.

**1. Create ECS Cluster with 1 Container Instance**

Before creating a cluster, let’s create a security group called my-ecs-sg that we’ll use.

    aws ec2 create-security-group --group-name my-ecs-sg --description my-ecs-sg

Now create an ECS Cluster called my-cluster and the ec2 instance that belongs to the ECS Cluster. Use the my-ecs-sg security group that was created. You can get the id of the security group from the EC2 Console / Network & Security / Security Groups. It is important to select a Key pair so you can ssh into the instance later to verify things are working.

For the Networking VPC settings, I used the default VPC and all the Subnets associated with the account to keep this tutorial simple. For the IAM Role use ecsInstanceRole. If ecsInstanceRole does not yet exist, create it per [AWS docs](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/instance_IAM_role.html). All the my settings are provided in the screenshot. You will need to change the settings according to your own account and default VPC and Subnets.

![](https://cdn-images-1.medium.com/max/4192/1*E86wkAlczaasBXwSIWsWtg.png)

Wait a few minutes and the confirm that the Container Instance has successfully registered to the my-cluster ECS cluster. You can confirm it by clicking on the ECS Instances tab under Clusters / my-cluster.

![](https://cdn-images-1.medium.com/max/4540/1*5YqhlgNb5f-An2vc6UvFuA.png)

**2. Create a task definition that will be blueprint to start a Sinatra app**

Before creating the task definition, find a [sinatra docker image](https://hub.docker.com/r/tongueroo/sinatra/) to use and test that it’s working. I’m using the tongueroo/sinatra image.

    $ docker run -d -p 4567:4567 --name hi tongueroo/sinatra
    6df556e1df02e93b05aa46425fc539121f5e50afee630e1cd918b337c3b6c202
    $ docker ps
    CONTAINER ID        IMAGE                COMMAND             CREATED             STATUS              PORTS                    NAMES
    6df556e1df02        tongueroo/sinatra   "ruby hi.rb"        2 seconds ago       Up 1 seconds        0.0.0.0:4567->4567/tcp   hi
    $ curl localhost:4567 ; echo
    42
    $ docker stop hi ; docker rm hi
    $

Above, I’ve started a container with the sinatra image and curl localhost:4657. Port 4567 is the default port that sinatra listens on and it is exposed in the [Dockerfile](https://github.com/tongueroo/sinatra/blob/master/Dockerfile). It returns “42” as expected. Now that I’ve tested the sinatra image and verify that it works, let’s create the task definition. Create a task-definition.json and add:

    {
      "family": "sinatra-hi",
      "containerDefinitions": [
        {
          "name": "web",
          "image": "tongueroo/sinatra:latest",
          "cpu": 128,
          "memoryReservation": 128,
          "portMappings": [
            {
              "containerPort": 4567,
              "protocol": "tcp"
            }
          ],
          "command": [
            "ruby", "hi.rb"
          ],
          "essential": true
        }
      ]
    }

The task definition is also available on GitHub: [task-definition.json](https://github.com/tongueroo/sinatra/blob/master/task-definition.json). To register the task definition:

    $ aws ecs register-task-definition --cli-input-json file://task-definition.json

Confirm that the task definition successfully registered with the ECS Console:

![](https://cdn-images-1.medium.com/max/2000/1*t91w5HUUp_puCgh-vShJAw.png)

**3. Create an ELB and Target Group to later associate with the ECS Service**

Now let’s create an ELB and a target group with it. We are creating an ELB because we eventually want to load balance requests across multiple containers and also want to expose the sinatra app to the internet for testing. The easiest way to create an ELB is with the EC2 Console.

Go the EC2 Console / Load Balancing / Load Balancers, click “Create Load Balancer” and select Application Load Balancer.

Wizard Step 1 — Configure Load Balancer

* Name it my-elb and select internet-facing.

* Use the default Listener with a HTTP protocol and Port 80.

* Under Availability Zone, chose a VPC and choose the subnets you would like. I chose all 4 subnets in the default VPC just like step 1. It is very important to chose the same subnets that was chosen when you created the cluster in step 1. If the subnets are not the same the ELB health check can fail and the containers will keep getting destroyed and recreated in an infinite loop if the instance is launched in an AZ that the ELB is not configured to see.

Wizard Step 2 — Configure Security Settings

* There will be a warning about using a secure listener, but for the purpose of this exercise we can skip using SSL.

Wizard Step 3 — Configure Security Groups

* Create a new security group named my-elb-sg and open up port 80 and source 0.0.0.0/0 so anything from the outside world can access the ELB port 80.

Wizard Step 4 — Configure Routing

* Create a new target group name my-target-group with port 80.

Wizard Step 5 — Register Targets

* This step is a little odd for ECS. We do actually not register any targets here because ECS will automatically register the targets for us when new tasks are launched. So simply skip and click next.

Wizard Step 6 — Review

* Review and click create.

![](https://cdn-images-1.medium.com/max/2000/1*ZoyErMG47uqB0eMNI7C8gQ.png)

When we created the ELB with the wizard we opened it’s my-elb-sg group port 80 to the world. We also need to make sure that the my-ecs-sg security group associated with the instance we launched in step 1 allows traffic from the ELB. We created the my-ecs-sg group in step 1 at the very beginning of this tutorial. To allow all ELB traffic to hit the container instance run the following:

    $ aws ec2 authorize-security-group-ingress --group-name my-ecs-sg --protocol tcp --port 1-65535 --source-group my-elb-sg

Confirm the rules were added to the security groups via the EC2 Console:

![](https://cdn-images-1.medium.com/max/2158/1*ESlE1mLLVwys3ammHMQakQ.png)

With these security group rules, only port 80 on the ELB is exposed to the outside world and any traffic from the ELB going to a container instance with the my-ecs-group group is allowed. This a nice simple setup.

**4. Create a Service that runs the Task Definition**

The command to create the ECS service takes a few parameters so it is easier to use a json file as it’s input. Let’s create a ecs-service.json file with the following:

    {
        "cluster": "my-cluster",
        "serviceName": "my-service",
        "taskDefinition": "sinatra-hi",
        "loadBalancers": [
            {
                "targetGroupArn": "FILL-IN-YOUR-TARGET-GROUP",
                "containerName": "web",
                "containerPort": 4567
            }
        ],
        "desiredCount": 1,
        "role": "ecsServiceRole"
    }

You will have to find your targetGroupArn created in step 3 when we created the ELB. To find the targetGroupArn you can go to the EC2 Console / Load Balancing / Target Groups and click on the my-target-group.

Now create the ECS service: my-service.

    $ aws ecs create-service --cli-input-json file://ecs-service.json

You can confirm that the container is running on the ECS Console. Go to Clusters / my-cluster / my-service and view the Tasks tab.

![](https://cdn-images-1.medium.com/max/4672/1*SPD_iarZmPRcgiBWTg53OA.png)

**5. Confirm Everything is Working**

Confirm that the service is running properly. You want to be thorough about confirming that all is working by checking a few things.

Check that my-target-group is showing and maintaining healthy targets. Under Load Balancing / Target Groups, click on my-target-group and check the Targets tab. You should see a Target that is reporting healthy.

![](https://cdn-images-1.medium.com/max/4632/1*sgDdgJQ07cyYQjZdL-h2VQ.png)

If the target is not healthy, check these likely issues:

* Check that the my-ecs-sg security group is allowing all traffic from the my-elb-sg security group. This was done in Step 4 with the authorized-security-group-ingress command after you created the ELB.

* Check that the security groups for the ELB, in step 3, is set to the same security groups that you use when you created the ECS Cluster and Container Instance in step 1. Remember the ELB can only detect healthy instances in AZs that it is configure to use.

Let also ssh into the instance and see the running docker process is returning a good response. Under Clusters / ECS Instances, click on the Container Instance and grab the public dns record so you can ssh into the instance.

![](https://cdn-images-1.medium.com/max/4664/1*o3ohYBaJ1UQtkGUGpBaJ4Q.png)

![](https://cdn-images-1.medium.com/max/4000/1*tCmd5r33gvgajr_K5nIvTg.png)

    $ ssh ec2-user@ec2-52-3-252-86.compute-1.amazonaws.com
    $ docker ps
    CONTAINER ID        IMAGE                            COMMAND             CREATED             STATUS              PORTS                               NAMES
    9e9a55399589        tongueroo/sinatra:latest        "ruby hi.rb"        16 minutes ago      Up 16 minutes       8080/tcp, 0.0.0.0:32773->4567/tcp   ecs-sinatra-hi-1-web-d8efaad38dd7c3c63a00
    4fea55231363        amazon/amazon-ecs-agent:latest   "/agent"            41 minutes ago      Up 41 minutes                                           ecs-agent
    $ curl 0.0.0.0:32773 ; echo
    42
    $

Above, I’ve verified that the docker container running on the instance by curling the app and seeing a successful response with the “42” text.

Lastly, let’s also verify by hitting the external DNS address of the ELB. You can find the DNS address in the EC2 Console under Load Balancing / Load Balancers and clicking on my-elb.

![](https://cdn-images-1.medium.com/max/3504/1*izgeqwpFp9jzHxWWvEvNRg.png)

Verify the ELB publicly available dns endpoint with curl:

    $ curl my-elb-1693572386.us-east-1.elb.amazonaws.com ; echo
    42
    $

6. Scale Up the Service to 4 Tasks

This is the easiest part. To scale up and add more containers simply go to Clusters / my-cluster / my-service and click on “Update Service”. You can change “Number of tasks” from 1 to 4 there. After only a few moments you should see 4 running tasks. That’s it!

7. Clean It All Up

It is quickest to use the EC2 Console to delete the following resources:

* ELB: my-elb

* ECS Service: my-service Task Definition: sinatra-hi Cluster: my-cluster

* Security group: my-elb-sg and my-ecs-sg.

## Summary

In this post I covered the ECS terminology and went through a simple example to create a sinatra app behind a ELB.

Overall, I think that ECS is a pretty amazing service and it has taken the hassle of managing docker orchestration and provisioning responsibility away.
> # Thanks for reading this far. If you found this post useful, I’d really appreciate it if you recommend this post (by clicking the clap button) so others can find it too! Also, connect with me on [LinkedIn](https://www.linkedin.com/in/tongueroo/).

![](https://cdn-images-1.medium.com/max/2000/1*forZa8mfCGWYU5PAz-fRQA.gif)
> # P.S. Be sure to join the [BoltOps newsletter to receive free DevOps tips and updates.](https://www.boltops.com/subscribe)
