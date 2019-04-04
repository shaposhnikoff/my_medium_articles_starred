
# Deploying the ELK stack on Amazon ECS, Part 4: Logstash

Deploying the ELK stack on Amazon ECS, Part 4

This is Part 4 of a multi-part series on how to deploy a containerized, production-ready ELK stack in Amazon’s EC2 container service.

Please refer to [Part 1](https://medium.com/@devfire/deploying-the-elk-stack-on-amazon-ecs-dd97d671df06), [Part 2](https://medium.com/@devfire/deploying-the-elk-stack-on-amazon-ecs-part-2-34c841e3b774), [Part 3](https://medium.com/@devfire/deploying-the-elk-stack-on-amazon-ecs-part-3-45ae8c1c9c12) for the previous tutorials.

At this point, you should have your ECS cluster deployed, ElasticSearch container configured and running, with the Application ELBs routing traffic across all nodes.

The next step is to deploy logstash to our cluster. Let’s get started!

NOTE: There have been a few minor changes for logstash v6.x, mainly due to xpack configuration. I noted those below.

First, create a new logstash directory. All our files will go here.

Next, create a Dockerfile to build our customized container.

NOTE: We are deploying an immutable logstash container, in accordance with the “immutable infrastructure” paradigm. This means that any config changes will necessitate deploying a new container.

Dockerfile for logstash v5.x (legacy):

    FROM docker.elastic.co/logstash/logstash:5.5.0
    RUN rm -f /usr/share/logstash/pipeline/logstash.conf
    ADD pipeline/ /usr/share/logstash/pipeline/
    ADD config/ /usr/share/logstash/config/

Dockerfile for logstash 6.x (latest version):

    FROM docker.elastic.co/logstash/logstash-oss:6.0.0
    RUN rm -f /usr/share/logstash/pipeline/logstash.conf
    RUN /usr/share/logstash/bin/logstash-plugin install logstash-input-sqs && \
     /usr/share/logstash/bin/logstash-plugin install logstash-filter-json
    ADD pipeline/ /usr/share/logstash/pipeline/
    ADD config/ /usr/share/logstash/config/

Let’s go through this line by line.

We first specify the base image for our customized logstash container. We use the OSS image with xpack disabled:

    FROM docker.elastic.co/logstash/logstash-oss:6.0.0

Then, we remove the default logstash.conf that came with the original container. That’s ok, we’ll add our own!

    RUN rm -f /usr/share/logstash/pipeline/logstash.conf

Also, I want to use the SQS plugin to get events from an SQS queue and process messages using the json plugin:

    RUN /usr/share/logstash/bin/logstash-plugin install **logstash-input-sqs** && \
     /usr/share/logstash/bin/logstash-plugin install **logstash-filter-json**

We will then add the pipeline configs and the logstash configs:

    ADD pipeline/ /usr/share/logstash/pipeline/
    ADD config/ /usr/share/logstash/config/

NOTE: Logstash 5.x changed how it handles its configuration. The Input Processing Output pipeline configs now go into the pipeline/ directory. The logstash configs go into config/ directory.

Next, let’s create a quick logstash.conf to handle a simple json input.

    input {
      tcp {
        port => 5000
        codec => json
        type => logstash
      }
    }

    output {
      elasticsearch {
          hosts => [ "elasticsearch-nonprod:80" ]
          index => "%{type}-%{+YYYY.MM.dd}"
      }
    }

Simply drop the above into a pipeline/logstash.conf file in your logstash/ directory.

NOTE: This pipeline handles [logspout ](https://github.com/gliderlabs/logspout)input only. Feel free to add your own. I will show you how to configure, build and deploy logspout later in the guide. For now, this pipeline accepts a JSON-formatted input over TCP, on a port 5000 and forwards it to your ElasticSearch Docker ECS cluster.

Next, let’s create a simple, working logstash v5.x config file:

    http.host: "0.0.0.0"
    path.config: /usr/share/logstash/pipeline
    xpack.monitoring.enabled: false

NOTE: For logstash v6.x you must remove the last line, like so:

    http.host: "0.0.0.0"
    path.config: /usr/share/logstash/pipeline

Only thing of note is that I’m disabling xpack monitoring. If you wish to enable it, switch it to true.

At this point, we are done with the logstash configuration!

Let’s create a new ECR repo to house our custom-built logstash containers.

Navigate to the Amazon ECS console and create a new logstash repository. Just like in [Part 1](https://medium.com/@devfire/deploying-the-elk-stack-on-amazon-ecs-dd97d671df06), we will use the same deployment script to build and push our container to the repository.

If all goes well, you should have an output similar to the below (for v5.x):

    Flag --email has been deprecated, will be removed in 17.06.
    Login Succeeded
    Sending build context to Docker daemon  103.9kB
    Step 1/4 : FROM docker.elastic.co/logstash/logstash:5.5.0
     ---> d54929b7aa07
    Step 2/4 : RUN rm -f /usr/share/logstash/pipeline/logstash.conf
     ---> Using cache
     ---> c710781ad283
    Step 3/4 : ADD pipeline/ /usr/share/logstash/pipeline/
     ---> Using cache
     ---> fbdfa931442b
    Step 4/4 : ADD config/ /usr/share/logstash/config/
     ---> Using cache
     ---> bd12516a181a
    Successfully built bd12516a181a
    Successfully tagged logstash:latest
    The push refers to a repository [31415926.dkr.ecr.us-east-1.amazonaws.com/logstash]
    8d39707b9d62: Layer already exists
    02b278d9fe0c: Layer already exists
    2fd791280d50: Layer already exists
    6eab574f5a05: Layer already exists
    228d3e0f8224: Layer already exists
    5432ae84157d: Layer already exists
    153c2d03faf4: Layer already exists
    04206b86a529: Layer already exists
    a3d3ccd6cfc6: Layer already exists
    ea273129a5de: Layer already exists
    aa2a9c663de0: Layer already exists
    8dd164900a2d: Layer already exists
    178e5c405069: Layer already exists
    99b28d9413e4: Layer already exists
    latest: digest: sha256:bc0f67c609c5d08afb9f1331b8d03b61ad0119cd1e2bc194deb1ced33e11c308 size: 3237

I used the same bakeandpush.sh script from Part 1:

<iframe src="https://medium.com/media/28634434c937a231d0e76aad88474857" frameborder=0></iframe>

Again, thank you [Jason Moody](undefined) for the *eval* snippet.

Run it with a logstash parameter. When you check your repo, you should have your logstash container, with the latest tag, ready for deployment.

Let’s do that now!

First, create a new task definition. If you need a refresher on what these are or how they work, either see the official Amazon [docs](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html) or [Part 2](https://medium.com/@devfire/deploying-the-elk-stack-on-amazon-ecs-part-2-34c841e3b774) of this tutorial.

Here’s a screenshot of a working logstash task definition:

![A working logstash task definition](https://cdn-images-1.medium.com/max/2334/1*Qd8T6MnGHI9w-xvxuouvfg.png)*A working logstash task definition*

A few things of note.

* Image: this is the URL that points to the ECR container.

* Soft limit: 2048MB but can be adjusted as needed.

* The port mappings are static 5000:5000. The reason is that our logstash accepts TCP port 5000 traffic only, not HTTP. As a result, we must use the Classic ELBs and those are not integrated with ECS. Consequently, dynamic port assignments will not work.

* LS_JAVA_OPTS: these are enough for smaller logstash containers. You might have to increase this for higher traffic deployments.

Save the task and we are done with this part!

Next, create a new service to manage our logstash Docker container.

Remember, because our logstash config accepts JSON over TCP:5000 input only, we cannot use Application Load Balancers (those are for HTTP traffic). We will use a Classic ELB instead.

First, create a new Classic ELB. Things of note:

* Use TCP:5000 as the listener

* Add the EC2 machines that form our cluster as target instances

* To communicate with the cluster, make sure you add the same Logging security group you created before. Additional security groups will have to be added to accept traffic from outside the ELB

* Don’t forget to add this new ELB to your Autoscaling Group that controls the ECS machines

* Health check: TCP:5000 — no surprises here

We are now ready to create our service. The only thing of note when creating a new logstash service is to specify a “One Task per Host” task placement strategy. This is because static 5000:5000 port mappings prevent more than one logstash container from running on the same host.

Now you are done! You should have a deployed, working logstash container that routes traffic to your ElasticSearch Docker cluster.

In Part 5, we will create a new Kibana container and wire it up to our ElasticSearch cluster.
