
# Deploying the ELK stack on Amazon ECS, Part 2: EC2 Container Service (ECS)

Deploying the ELK stack on Amazon ECS, Part 2: EC2 Container Service (ECS)

In the [previous post](https://medium.com/@devfire/deploying-the-elk-stack-on-amazon-ecs-dd97d671df06), we created our customized elasticsearch docker container, suitable for deployment in AWS EC2 (and ECS).

We will now setup a new ECS cluster, customize it to suit our ELK needs and deploy the containers.

Let’s get started!

First, go to your AWS console, navigate to the Amazon ECS section and create a new cluster:

![A new ECS cluster](https://cdn-images-1.medium.com/max/2376/1*v1H_aGq65kpBD0nm3jzl6Q.png)*A new ECS cluster*

This wizard will create a CloudFormation template that will eventually create and provision our ECS resources.

Let’s examine the configuration in more detail.

* Cluster name: can be whatever you want.

* Provisioning model: start with on-demand. You can always switch to spot if you know how spot pricing works. (Hint: major savings to be had with spot!)

* EC2 instance type: I picked t2.xlarge but this can be modified later.

* Number of instances: The goal here is to create an operationally ready, prod-like cluster. So, we are going with 3.

* ECS Ami: this is the latest ECS-optimized AMI as of this writing. Note: this changes quite frequently. When it does, you will have to adjust your Launch Config to make sure you use the latest image. More on that later but for now, you can get the notifications via an SNS [subscription](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS-AMI-SubscribeTopic.html).

* EBS Storage: This is **not **storage for your containers to store their data. For example, our elasticsearch containers will **not **be able to use this volume. This volume is strictly for docker images. Also note it is **not** exposed as a file system, so don’t bother looking for it with *df. *Amazon explains it in more detail here:
> The volume is configured as a Logical Volume Management (LVM) device and it is accessed directly by Docker via the devicemapper backend. Because the volume is not mounted, you cannot use standard storage information commands (such as **df -h**) to determine the available storage.

* Keypair: pick your own, this is a standard EC2 SSH keypair.

Since this is an ELK guide, not an ECS guide, I’m skipping over some of the important ECS details. Just make sure you pick the correct VPC, subnets, security groups and a role.

Click Create and watch the CloudFormation do its work. Note the Launch Config that was created as part of this deployment, we will need to customize it later.

Once completed, the good news is that you will have a working ECS cluster! Bad news is that it will be unsuitable for the ElasticSearch deployment.

![Sad!](https://cdn-images-1.medium.com/max/2000/0*zlZpAmVsDoW2SRN-.jpg)*Sad!*

That’s right, there are a few issues that need fixing.

First, there is no way to disable public IP assignments from the ECS wizard. Instead, we need to manually ensure our private cluster stays private, safely tucked inside our VPC. (Note: currently, no way to do it from *aws ecs* OR via *ecs-cli *either).

Second, ElasticSearch will fail due to a low *mmap* count; we need to increase that.

Third, we need to allocate more space to our ElasticSearch nodes than the standard 8GB granted by Amazon.

Finally, we need to customize the docker daemon to take advantage of the extra space.

Let’s fix these issues now.

Remember the launch config that was created via the CloudFormation? Navigate there and create a new copy.

* Name: it is a good idea to version these launch configs, so append _v2 or something similar to the end.

* User data: this is the important piece!

    Content-Type: multipart/mixed; boundary="==BOUNDARY=="
    MIME-Version: 1.0

    --==BOUNDARY==
    Content-Type: text/cloud-boothook; charset="us-ascii"

    # Set Docker daemon options
    cloud-init-per once docker_options echo 'OPTIONS="${OPTIONS} --storage-opt dm.basesize=250G"' >> /etc/sysconfig/docker

    --==BOUNDARY==
    Content-Type: text/x-shellscript; charset="us-ascii"

    #!/bin/bash
    # Set the ECS agent configuration options
    cat <<'EOF' >> /etc/ecs/ecs.config
    ECS_CLUSTER=platformnonprod
    ECS_ENGINE_TASK_CLEANUP_WAIT_DURATION=15m
    ECS_IMAGE_CLEANUP_INTERVAL=10m
    EOF
    sysctl -w vm.max_map_count=262144
    **mkdir -p /usr/share/elasticsearch/data/
    chown -R 1000.1000 /usr/share/elasticsearch/data/**

    --==BOUNDARY==--

First, we are using cloud-boothook to allow docker to use the additional storage. If we don’t, it will be set too late in the boot process and will not effect the docker daemon.

Second, we need to bump up the [*mmap *count](https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html):
> The default operating system limits on mmap counts is likely to be too low, which may result in out of memory exceptions.

Ignore the innocuous “may result.” ElasticSearch will not start unless this is increased.

Next, navigate to the Add Storage section and give your instances more storage. Here, I’m allocating 250GB. Note: this matches the *dm.basesize=250G *setting in the user data above.

![Storage settings](https://cdn-images-1.medium.com/max/2826/1*uVOcSVUrUaZjU5rF2uIOuQ.png)*Storage settings*

You maybe tempted to use [EFS ](https://aws.amazon.com/efs/)here. Don’t! It’s too slow for ElasticSearch low-latency reads and writes.

You may also consider using EBS to store ElasticSearch data. I don’t think there is anything inherently wrong with that. For the record, Elastic’s official documentation advises against [this](https://www.elastic.co/guide/en/elasticsearch/plugins/5.5/cloud-aws-best-practices.html):
> Works well if running a small cluster (1–2 nodes) and cannot tolerate the loss all storage backing a node easily or if running indices with no replicas. If EBS is used, then leverage provisioned IOPS to ensure performance.

Given the above, we are going to stick with the instance store.

NOTE: the lines in bold in user data create a new directory for elasticsearch and set the permissions to allow read/write. For this exercise, we will rely on ElasticSearch built-in replication mechanisms AND docker named volumes to persist data:
> With Instance Store one gets the performance benefit of having disk physically attached to the host running the instance and also the cost benefit of avoiding paying extra for EBS.

One final note on security groups. I like to use security group references when allowing in/out traffic. For this example, I created a Logging security group which allows traffic only from itself.

![Logging security group only allows traffic to/from itself](https://cdn-images-1.medium.com/max/2426/1*_kEHHFdCslGnr3JDqpSgag.png)*Logging security group only allows traffic to/from itself*

Then, when you assign the Logging security group to your Launch Config, all machines in the cluster will be able to communicate with each other, disallowing all outside traffic.

NOTE: if you would like to log data from outside the cluster, use an ELB to route traffic. We will cover this later in the guide.

This is it for the Launch Config modifications. Save the Launch Config and move over to the Autoscaling Group.

First, update your Autoscaling group with the new Launch Config. Then, make sure you assign the ElasticSearch ec2 discovery tags to every new instance. Otherwise, the nodes will not find each other.

![The ElasticSearch Key|Value is for autodiscovery](https://cdn-images-1.medium.com/max/2000/1*CbD_v_u0eZhLRNt9Si9Emg.png)*The ElasticSearch Key|Value is for autodiscovery*

Finally, take the desired count to ZERO to destroy the old machines and then back up to the 3 nodes.

And we are done! We now have an ECS cluster that is ready to run our ELK stack!

Check your work in Amazon ECS console.

NOTE: I am using 4 nodes but you can have as many as you desire.

![This cluster has 4 nodes registered](https://cdn-images-1.medium.com/max/2000/1*UcolDlfyDnkTzRM4XzIFnA.png)*This cluster has 4 nodes registered*

Next, we will work on deploying the ElasticSearch to our cluster.

Part 3 is [here](https://medium.com/@devfire/deploying-the-elk-stack-on-amazon-ecs-part-3-45ae8c1c9c12).
