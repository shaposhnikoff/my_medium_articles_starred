
# Deploying the ELK stack on Amazon ECS, Part 1

Deploying the ELK stack on AWS ECS, Part 1: Introduction & First Steps

*NOTE: This is Part 1 of a multi-part post. It deals with ElasticSearch Docker container customization and creation.*

[*Part 2](https://medium.com/@devfire/deploying-the-elk-stack-on-amazon-ecs-part-2-34c841e3b774) handles ECS deployment.*

[*Part 3](https://medium.com/@devfire/deploying-the-elk-stack-on-amazon-ecs-part-3-45ae8c1c9c12) deals with running the ElasticSearch containers as proper ECS services.*

[*Part 4](https://medium.com/@devfire/deploying-the-elk-stack-on-amazon-ecs-part-4-84a1e9a32f53) handles Logstash.*

[*Part 5](https://medium.com/@devfire/deploying-the-elk-stack-on-amazon-ecs-part-5-kibana-74407bc7cb6c) is for Kibana.*

[*Part 6](https://medium.com/@devfire/deploying-the-elk-stack-on-amazon-ecs-part-6-logspout-b0b360f1d09e) will deal with awslogs customization and deployment.*

Like many people in tech, we have deployed the ELK (ElasticSearch Logstash Kibana) stack as our common logging and error handling platform.

To be fair, the journey has been a bit twisty. At my current employer, we have started with our own deployment, both on-prem and EC2. Then, faced with the ever-increasing operational burden of managing a distributed database, we switched to Amazon’s managed ElasticSearch service.

However, faced with the additional costs, slow support response, lack of reserved instances, crippled API and slow upgrade cycles, we are now ready to bring ElasticSearch back in-house.

That said, I did not want to simply pile on more EC2 instances. I wanted to see if it would be possible to run ElasticSearch, Logstash, and Kibana as Docker containers in ECS (Amazon’s EC2 Container Service).

This will be a multi part post. First, I’ll walk you through how to properly configure, build, and push an immutable ElasticSearch container to AWS ECR — Amazon’s private docker registry. We will then create the necessary load-balancers (Application ELBs) and deploy the containers in ECS. Finally, we will do the same for Logstash, followed by Kibana.

Albeit somewhat crude, this diagram represents what we are trying to build.

![High-level architecture overview](https://cdn-images-1.medium.com/max/2000/1*5Hl_0cXZzu8MfNxSFv-b-Q.png)*High-level architecture overview*

Up top, a single application load balancer to handle all our HTTP traffic.

The big box in blue is our ECS cluster, specially configured to run ElasticSearch (and other containers, obviously!).

All containers come from the EC2 Container Registry (except Kibana, we’ll parameterize the official image instead).

Unfortunately, we have to have a Classic ELB to handle our TCP (non-HTTP) in-bound traffic.
> # UPDATE 1/13/2019: The above is no longer true. AWS Network Load Balancer handles TCP traffic and has native integration with ECS. The Classic Load Balancer does not. However, the Network Load Balancer works very differently from the Classic one — it does not terminate connections but passes them onto the target hosts (what’s known as a “***load balancer on a stick”*** type configuration.) Therefore, you cannot assign security groups to the Network Load Balancer. Whether this is acceptable or not is up to you. I’m leaving the Classic load balancer as is for now.

In the diagram above, we have the following containers:

* Three ElasticSearch containers to ensure high availability. Odd number to minimize split brain scenarios.

* Two Kibana containers for UI

* Two logstash containers to handle traffic

With that, let’s begin!

We are going to start by creating our own ElasticSearch container, based on the official Elastic Docker image. The reasons why we need our own are twofold.

* First, we want to create a versioned elasticsearch.yml (main config file) and bake it into our image. That will make tracking changes easier.

* Second, we need to install a special EC2 plug-in to ensure that multiple ElasticSearch nodes can discover each other.

NOTE: Any changes to the elasticsearch.yml file will require rebuilding, re-deploying and replacing the running containers with a new version. This is known as an “immutable infrastructure” and is a popular paradigm these days, for a good reason! It greatly simplifies and eliminates configuration drift across environments.

OK, back to our task. Start by creating an elasticsearch directory (we will later use it as a git repo to commit our files) and use this Dockerfile to build our image:

    FROM docker.elastic.co/elasticsearch/elasticsearch:5.6.4
    ENV REGION us-east-1
    ADD elasticsearch.yml /usr/share/elasticsearch/config/
    USER root
    RUN chown elasticsearch:elasticsearch config/elasticsearch.yml
    USER elasticsearch
    WORKDIR /usr/share/elasticsearch
    RUN bin/elasticsearch-plugin install discovery-ec2 && bin/elasticsearch-plugin install repository-s3 && sed -e '/^-Xm/s/^/#/g' -i /usr/share/elasticsearch/config/jvm.options

Let’s go through this line by line.

First, we tell docker to use the official Elastic image as the base:

    FROM docker.elastic.co/elasticsearch/elasticsearch:5.6.4
> # UPDATE 1/13/2019: The above version is now old, the latest version of ElasticSearch is 6.x However, it should all still work the same way.

Set the default region to us-east-1. If you are in a different region, change this accordingly. If you are in MULTIPLE regions, pass this as an environment variable in the task definition (described later):

    ENV REGION us-east-1

Then, we will **ADD o**ur customized elasticsearch.yml into our new docker container:

    **ADD **elasticsearch.yml /usr/share/elasticsearch/config/

Then, we need to switch to user *root *and change the config file ownership, to make sure elasticsearch can read it:

    USER root
    RUN chown elasticsearch:elasticsearch config/elasticsearch.yml

Now, we switch back to elasticsearch and install a special ec2 plugin, to enable node discovery in AWS, install an s3 plugin to perform backups to S3 and comment out the heap settings so we can pass the values via an environment variable:

    USER elasticsearch
    WORKDIR /usr/share/elasticsearch
    RUN bin/elasticsearch-plugin install discovery-ec2 && bin/elasticsearch-plugin install repository-s3 && sed -e '/^-Xm/s/^/#/g' -i /usr/share/elasticsearch/config/jvm.options

And that’s it for the Dockerfile!

Let’s now look at our customized elasticsearch.yml:

    cluster.name: "elasticsearch"
    bootstrap.memory_lock: false
    network.host: 0.0.0.0
    network.publish_host: _ec2:privateIp_
    transport.publish_host: _ec2:privateIp_
    discovery.zen.hosts_provider: ec2
    discovery.ec2.tag.ElasticSearch: nonprod
    discovery.ec2.endpoint: ec2.${REGION}.amazonaws.com
    s3.client.default.endpoint: s3.${REGION}.amazonaws.com
    cloud.node.auto_attributes: true
    cluster.routing.allocation.awareness.attributes: aws_availability_zone
    xpack.security.enabled: false

I won’t go through every setting, they are well documented [here](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html). A few that we care about are explained below.

Cluster name:

    cluster.name: "nonprod"

you can obviously set it to whatever you want.

Discovery endpoints:

    discovery.ec2.endpoint: ec2.${REGION}.amazonaws.com
    s3.client.default.endpoint: s3.${REGION}.amazonaws.com

NOTE: These settings have been changed in recent versions of elasticsearch.

Force elasticsearch to listen on all interfaces:

    network.host: 0.0.0.0

Make sure the nodes can find each other via this tag:

    discovery.ec2.tag.ElasticSearch: nonprod

The other settings are all ec2 plug-in specific.

NOTE: ElasticSearch has a ton of important system configuration parameters. Please read the [documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html) for details. This guide only covers the essential configs to get you started.

We now have two artifacts in our elasticsearch directory:

    [igor@ansible elasticsearch]$ ls
    Dockerfile  elasticsearch.yml

We are now ready to publish our customized container to Amazon’s EC2 Container Registry, or ECR.

First, go to your AWS console, navigate to the EC2 Container Service section and create a new elasticsearch repository:

![A new private docker repo](https://cdn-images-1.medium.com/max/2000/1*TaCIw42IBjPrG19FpbTLIg.png)*A new private docker repo*

You will then get back your account specific ECR URL, that will serve as the repository from which ECS will deploy the containers.

I use the following script to create and push a new docker image into our private docker repo:

<iframe src="https://medium.com/media/28634434c937a231d0e76aad88474857" frameborder=0></iframe>

NOTE: A huge thank you to [Jason Moody](undefined) for the eval shortcut.

Also, make sure you replace the *31415926* π number with your account number.

When run, the script takes a single parameter — the repo name, elasticsearch in our case. Now run the script and watch the magic happen!

If all goes well, your output should be similar to this:

    Flag --email has been deprecated, will be removed in 17.06.
    Login Succeeded
    Sending build context to Docker daemon  3.072kB
    Step 1/7 : FROM docker.elastic.co/elasticsearch/elasticsearch:5.5.0
     ---> 2377bc62195f
    Step 2/7 : ADD elasticsearch.yml /usr/share/elasticsearch/config/
     ---> Using cache
     ---> d21a7ad3b76d
    Step 3/7 : USER root
     ---> Using cache
     ---> ef71b878add0
    Step 4/7 : RUN chown elasticsearch:elasticsearch config/elasticsearch.yml
     ---> Using cache
     ---> 42c45b37b10d
    Step 5/7 : USER elasticsearch
     ---> Using cache
     ---> 51ae4c2106f5
    Step 6/7 : WORKDIR /usr/share/elasticsearch
     ---> Using cache
     ---> f3e1ccd6d4c8
    Step 7/7 : RUN bin/elasticsearch-plugin install discovery-ec2
     ---> Using cache
     ---> 18e4c81c4afd
    Successfully built 18e4c81c4afd
    Successfully tagged elasticsearch:latest
    The push refers to a repository [**31415926**.dkr.ecr.us-east-1.amazonaws.com/elasticsearch]
    23e6d115444d: Layer already exists
    a0957bb18e23: Layer already exists
    02b8dbaf43bc: Layer already exists
    d7f35f110e8d: Layer already exists
    538c8274ff81: Layer already exists
    210f6c7d2a97: Layer already exists
    48d29a4044d0: Layer already exists
    19dfa2acc30a: Layer already exists
    6537126d9ffe: Layer already exists
    6309a0db5afd: Layer already exists
    513204734a1f: Layer already exists
    0f6703231140: Layer already exists
    99b28d9413e4: Layer already exists
    latest: digest: sha256:60f9bb7c8f50cb61f1951d98d27edf94ecf41342af311c74bed90fa83a5625b2 size: 3035

Check your ECR repo to make sure the image has indeed been published.

![Docker containers in the ECR](https://cdn-images-1.medium.com/max/2000/1*GCIp44gjTPtD2CnMJsER5g.png)*Docker containers in the ECR*

Note the *latest* tag. We will use it later to deploy our container to ECS.

Whew! We now have our freshly baked, customized, EC2-enabled elasticsearch container deployed to our private ECR, ready for a future immutable deployment. A significant milestone!

In the next post, we will explore how to create and customize our ECS cluster for a successful ELK deployment.

[Part 2: ECS creation and customization.](https://medium.com/@devfire/deploying-the-elk-stack-on-amazon-ecs-part-2-34c841e3b774)

[Part 3: ElasticSearch ECS Service creation and deployment.](https://medium.com/@devfire/deploying-the-elk-stack-on-amazon-ecs-part-3-45ae8c1c9c12)

[Part 4](https://medium.com/@devfire/deploying-the-elk-stack-on-amazon-ecs-part-4-84a1e9a32f53): Logstash Docker creation and deployment.
