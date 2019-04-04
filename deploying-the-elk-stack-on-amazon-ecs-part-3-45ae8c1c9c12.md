
# Deploying the ELK stack on Amazon ECS, Part 3: ElasticSearch

Deploying the ELK stack on Amazon ECS, Part 3: ElasticSearch

Let’s recap where we are on our containerized ELK journey.

In [Part 1](https://medium.com/@devfire/deploying-the-elk-stack-on-amazon-ecs-dd97d671df06), we customized, built, and deployed an immutable ElasticSearch container to our private ECR registry.

In [Part 2](https://medium.com/@devfire/deploying-the-elk-stack-on-amazon-ecs-part-2-34c841e3b774), we customized and deployed an ECS cluster suitable to run our ELK container from Part 1.

We are now ready to run our ElasticSearch container in ECS!

NOTE: This part is the most technically complex of them all. Once you get through this successfully, it will be a significant accomplishment!

First of all, ECS itself is well documented [here](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html). But the two things we care about are *tasks definitions* and *services*.
> A task definition is a text file in JSON format that describes one or more containers that form your application.

NOTE: if you are familiar with docker-compose, Amazon *ecs-cli* is able to understand docker-compose.yml files and convert them to task definitions.

So, you can define a task (a blueprint of your containerized application) and submit it to ECS to run. However, if a container were to terminate unexpectedly, ECS will take no action. To ensure that failed containers are restarted, we need to create a service.
> Amazon ECS allows you to run and maintain a specified number (the “desired count”) of instances of a task definition simultaneously in an ECS cluster. This is called a service.

For our ElasticSearch container, we will create both — a task and a service.

First, let’s create a task definition.

![Task definition wizard in Amazon ECS](https://cdn-images-1.medium.com/max/2000/1*PXB5TVp_fzgYh1IPf176Zw.png)*Task definition wizard in Amazon ECS*

Name it elasticsearch and add a container down below. A few things of note:

* Image: paste the URL from the registry where we pushed our container in Part 1. For example: ***31415926**.dkr.ecr.us-east-1.amazonaws.com/$REPO_NAME:latest*

* Memory limits: the distinction between hard and soft memory limits is documented [here](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html). Personally, I prefer the soft limits to allow for optional container memory increase, if needed. If you specify the hard limit, the container will be killed if it exceeds its memory allocation.

* Port mappings: we need 9200 for external access and 9300 for intra-node communication.

NOTE: You can either statically map host ports to container ports OR map a dynamically assigned host port to a static container port. Static mappings will allow only 1 container per host (because multiple containers cannot use the same port). Dynamic mappings allow for multiple containers per host. In our case, we don’t want multiple database containers per host because that will put our High Availability at risk. Therefore, static mapping is fine.

**UPDATE Feb 2018**: ECS now offers a (relatively) new networking mode, called* [awsvpc](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-networking.html):*
> # When you use the awsvpc network mode in your task definitions, every task that is launched from that task definition gets its own elastic network interface, a primary private IP address, and an internal DNS hostname.

What this means is that you can, in theory, run more than one elasticsearch container, with static ports on all, because each container gets its own elastic network interface.

In fact, AWS Fargate — a new, unmanaged ECS option only supports *awsvpc.*

Still, I would recommend you do **not** do this. Reason is, same high availability criteria still apply — you do NOT want more than one elasticsearch container per host. If that host fails, you lose a dis-proportionally high number of containers.

* Environment variables: add **ES_JAVA_OPTS **key, with a **-Xms8g -Xmx8g** value. NOTE: For this to work, *jvm.options* file must have these settings commented out, otherwise the env variable will be ignored! This is done via the Dockerfile sed command (described in the previous section).

* Ulimits: we must raise the limit, ElasticSearch will [not start](https://www.elastic.co/guide/en/elasticsearch/reference/master/setting-system-settings.html) otherwise. Pick nofile, set both the Soft limit and Hard limit to 65536.

* Log driver: don’t leave the default json-file. Reason is, left unchecked, this will grow until the disk is full, causing an outage. Instead, either place an upper limit or better yet, switch to awslog like below. Just make sure to create the CloudWatch group *elasticsearch* ahead of time.

![awslog logging driver instead of the default json-file](https://cdn-images-1.medium.com/max/2000/1*cA7mQcZnERnzpNdpX-jjDg.png)*awslog logging driver instead of the default json-file*

* Ensure that a docker named volume is exposed to the container. This is needed for data storage, to persist the data even if a container were to die. Unfortunately, we must specify the host path, we can’t just leave it to the docker daemon. Reason is, AWS ECS will remove unnamed docker volumes after a certain period (ECS_ENGINE_TASK_CLEANUP_WAIT_DURATION) when no containers access it. We don’t want that, so we have to specify the host path, as seen below:

![esdata is a name we will use for container definition](https://cdn-images-1.medium.com/max/2000/1*9jvNnNxjIHXodTCtmMocHQ.png)*esdata is a name we will use for container definition*

* Finally, add the volume to the container definition, as a mount point:

![](https://cdn-images-1.medium.com/max/2000/1*OpkJlz9_ccL3Kg0etKNJfg.png)

NOTE: The documentation link says to edit the /etc/security/limits.conf file. We don’t want to do that because we would either have to bake a new AMI or modify the Launch Config. Instead, we’ll set it here.

Your final container definition should look like this.

![Final task definition. Note the REGION env var to configure the discovery endpoint.](https://cdn-images-1.medium.com/max/2286/1*mMINqJbTe8Lyi8Wqa7lbvg.png)*Final task definition. Note the REGION env var to configure the discovery endpoint.*

Click Create. We are done with the task definition.

Next, let’s create a new Application Load Balancer (ALB) to load balance the traffic across all 3 ElasticSearch nodes. We will then wire up the target group to the ElasticSearch service — that’s how the two will communicate and execute health checks.

NOTE: You can also create the ALB from the service wizard. Either way is fine.

Creation of an ALB is somewhat outside the scope of this guide, so I will simply call out the important aspects.

* Targets: pick the 3 nodes that make up your ECS cluster.

* Port: Stick with port 80. We will then NAT port 80 to the port 9200 on the ECS host. It will then be flipped again to 9200 (no change!) which is the container’s own internal port.

* Health check: We can’t use port 80, we have to specify 9200 override.

![Health checks for the target group](https://cdn-images-1.medium.com/max/2000/1*SNpBBZOrqCsZVC9WzOGgQg.png)*Health checks for the target group*

Note the path — it’s the ElasticSearch internal health check API.

* Security group: Assign it the Logging security group we created in Part 1. This will allow the ELB to talk to the ECS cluster. Additionally, ensure you have a separate security group that will allow external traffic to hit your ELB.

That’s it for the ELB creation!

Now, let’s create a service to ensure our ElasticSearch containers are always up and running.

Navigate to your cluster and click Create Service.

* Name it elasticsearch, type in elasticsearch for the task definition. If you want 3 containers to always be up and running, specify 3 for the desired count.

* Click the Load Balancing button below and select all the settings from the ALB configuration up above. Make sure you select the created listener (port 80) and the created target group.

* Service role: should have been created previously. AmazonEC2ContainerServiceRole is enough to get it started:

![A working ECS service role](https://cdn-images-1.medium.com/max/2016/1*txuA6bQX89JNxUL9PD230g.png)*A working ECS service role*

NOTE: If you had done everything correctly with the Application Load Balancer setup above, you should not have to create anything new in this section!

Create and save. Assuming everything went fine, your service details should look like this.

![Successful service definition](https://cdn-images-1.medium.com/max/2682/1*DuSAvOz6aZBkk1Ydi-0BLQ.png)*Successful service definition*

As a quick sanity check, ensure your targets are all healthy in the target group. Do not proceed if they are not!

![Healthy targets in the target group](https://cdn-images-1.medium.com/max/2562/1*4yRpDGgav1BdJOHzOJjBgA.png)*Healthy targets in the target group*

NOTE: You can optionally create a friendly Route 53 name and point it to the ELB. You can then create a routing rule in the ALB to route requests to the elasticsearch containers. In fact, I strongly recommend you do that — AWS ELB names are notoriously long and confusing.

![Routing rules to filter on Route 53](https://cdn-images-1.medium.com/max/2064/1*XNBUdZpHxB0sfop5S4jx1Q.png)*Routing rules to filter on Route 53*

If all went well, you can call the ElasticSearch health API to make sure all nodes are in the cluster

    [ec2-user@ansible ~]$ curl -s [http://elasticsearch-nonprod/_cat/health](http://elasticsearch-nonprod.pf-labs.net/_cat/health)
    1500567291 16:14:51 nonprod **green 3** 3 8 4 0 0 0 0 - 100.0%

Notice it says “green” and has 3 nodes.

NOTE: elasticsearch-nonprod is a DNS name I created locally. Yours may or may not differ.

And we are done!

That said, this is a complicated tutorial with lots of moving parts. If things didn’t work as expected, a few things you can try.

* Run the elasticsearch container locally to make sure you see no egregious errors. Just make sure you adjust the ulimits first or it will fail.

* SSH into your ECS nodes and run docker log on the container to see its log output.

* Make sure your security groups allow traffic from the Application Load Balancer to the ECS cluster (that’s the Logging security group from before).

* Make sure your security groups allow traffic to the Application Load Balancer from your desktop.

* Check the Events tab in the elasticsearch service. There should be a message like this: ***service [elasticsearch](https://console.aws.amazon.com/ecs/home?region=us-east-1#/clusters/platformnonprod/services/elasticsearch) has reached a steady state.***

However, if you managed to get through this successfully — congratulations! It took me hours of pouring through documentation, outdated guides and random AWS blogs posts to piece this together. Certainly no easy feat, so don’t feel bad if you get stuck. Post your questions, I’ll do my best to answer.

Next, we will deploy logstash to send data to our cluster.

[Part 4](https://medium.com/@devfire/deploying-the-elk-stack-on-amazon-ecs-part-4-84a1e9a32f53): logstash!
