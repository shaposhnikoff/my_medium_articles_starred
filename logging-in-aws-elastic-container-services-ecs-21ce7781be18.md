
# Logging in AWS Elastic Container Services (ECS)

I really like AWS, but there are times when Amazon releases things before they are ready. Logging is such a basic requirement, I thought AWS would have this covered. From what I can tell, it was only recently introduced in ECS and it’s certainly half-baked. I decided to document my solution, and the alternatives I considered, and hope this helps others in the AWS/ECS Docker community.

(If you just want our solution, skip to the Fluentd section below.)

### Docker AWS Log Driver

[Docker has an AWS Log Driver that logs to CloudWatch](https://docs.docker.com/engine/reference/logging/awslogs/). When I originally read about this, I was very hopeful. I even pointed our story low because I thought it would be a slam dunk!

This new logger is great except that AWS ECS does not support this logger yet: [https://forums.aws.amazon.com/thread.jspa?threadID=219551](https://forums.aws.amazon.com/thread.jspa?threadID=219551). There goes the obvious solution.

### Amazon’s Clumsy Logging

My next consideration was this article because it had a lot of references from AWS: [https://blogs.aws.amazon.com/application-management/post/TxFRDMTMILAA8X/Send-ECS-Container-Logs-to-CloudWatch-Logs-for-Centralized-Monitoring](https://blogs.aws.amazon.com/application-management/post/TxFRDMTMILAA8X/Send-ECS-Container-Logs-to-CloudWatch-Logs-for-Centralized-Monitoring). I found this very clumsy/dirty and searched for something else. I did not want to pollute all of our Dockerfiles with the commands suggested in the article. The developer who writes the Dockerfile should not need to know about this confusing and complex logging requirement. They should just use the basic Docker log drivers. Further, I don’t like the idea of launching two Docker containers, and linking the containers, just for something simple and ubiquitous like logging.

### JSON Log Driver

Next up was using the JSON log driver and send the JSON logs to CloudWatch. Nope. The JSON files are kept in a subdirectory name after the Docker container instance. This is not configurable.

In the example below, you can see the container identifier is a parent directory to the log file.

    [pwd]
    /var/lib/docker/containers/**00187b71b559c4658ee3642626ccafc086195d9d52519b63070c42d6c608bcdd**

    [ls]
    00187b71b559c4658ee3642626ccafc086195d9d52519b63070c42d6c608bcdd-json.log

This location makes it impossible to import to CloudWatch because the *awslogs *service does not support wildcards in their paths for directory names. I really wanted to use something like this in */etc/awslogs/awslogs.conf*, but it doesn’t work:

    file = /var/lib/docker/container/*/*json.log

### Fluentd

I was trying to avoid the other Docker logging drivers to keep the system simple and to reduce unneeded moving parts and dependencies. Once I decided we needed an external logging service, I chose Fluentd over the others (syslog, journald, gelf) because it seemed the most flexible for our future needs. If I was going to go through this pain, I’d get future mileage out of it.

Fluentd seems very powerful — I’m a Fluentd novice — with a mind-numbing number of permutations between the sources and outputs. For now, we just needed our logs saved to S3 so our developers could see why their services were crashing. Although this is fully supported by AWS, the documentation is frustratingly incomplete.

Setting up the Fluentd service is quite easy. I started with a nice Docker image (fluentd/fluentd:latest) and added the S3 plugin in our own Dockerfile. Our simple Dockerfile is below:

    FROM fluent/fluentd:latest
    MAINTAINER [bill@centricient.com](mailto:bill@centricient.com)

    USER ubuntu

    WORKDIR /home/ubuntu

    ENV PATH /home/ubuntu/ruby/bin:$PATH
    RUN fluent-gem install fluent-plugin-s3

    # Temp area for fluentd
    RUN mkdir -p /home/ubuntu/log/fluent/s3
    RUN chmod 744 /home/ubuntu/log/fluent/s3

    CMD fluentd -vv -o /home/ubuntu/fluentd.log -c /fluentd/etc/$FLUENTD_CONF -p /fluentd/plugins $FLUENTD_OPT

Updating Docker on ECS Node

When setting up your ECS Task Definition, you are allowed to choose Fluentd, but this won’t work by default. The *ecs-optimized* AMIs provided by Amazon use Docker 1.7. Fluentd support was not introduced until Docker 1.8. Out of the box, ECS AMIs will not support Fluentd, even through the ECS UIs and CLI make it appear so. The first step is to update Docker on all of your images.

Using Ansible, we upgraded Docker and ecs-init, pushed our Fluentd configurations, and downloaded and started the Fluentd Docker image. With this in place, I thought we’d be ready to go, but I hit another AWS ECS snag. The Docker container would not start. I received the ECS error, “unable to place a task because no container instance met all of its requirements.”

![](https://cdn-images-1.medium.com/max/2896/1*toDNmODkZQxHsvQY7N9nAg.png)

I surmised that I needed to configure the ECS nodes somehow to support this Docker Logging Driver. I dug around and found that you need to add this line to */etc/ecs/ecs.config:*

    ECS_AVAILABLE_LOGGING_DRIVERS=[“json-file”,”syslog”,**”fluentd”**]

Some of the other stuff I understood, but this is honestly stupid. When you configure Fluentd in the ECS task definition, you specify the address of the Fluentd server. It can be anywhere, and certainly does not need to be on the ECS node. Therefore, it makes no sense that the ECS node must be configured to be a Fluentd log driver. None of my griping matters, because you just have to do this. It’s another task for Ansible.

The good news, is this is the last flaming hoop you have to jump through. You can see if your nodes are ready to go by running *describe-container-instances* from the AWS ECS CLI. You should see a line like the one in bold below.

    “attributes”: [
     {
     “name”: “com.amazonaws.ecs.capability.privileged-container”
     },
     {
     “name”: “com.amazonaws.ecs.capability.docker-remote-api.1.17”
     },
     {
     “name”: “com.amazonaws.ecs.capability.docker-remote-api.1.18”
     },
     {
     “name”: “com.amazonaws.ecs.capability.docker-remote-api.1.19”
     },
     {
     “name”: “com.amazonaws.ecs.capability.docker-remote-api.1.20”
     },
     {
     “name”: “com.amazonaws.ecs.capability.logging-driver.json-file”
     },
     {
     “name”: “com.amazonaws.ecs.capability.logging-driver.syslog”
     },
     {
    ** “name”: “com.amazonaws.ecs.capability.logging-driver.fluentd”**
     },
     {
     “name”: “com.amazonaws.ecs.capability.ecr-auth”
     }
     ],

### Conclusion

In the end, I totally blew the the points on my story, but logging is in place and working. I may have missed something obvious. If so, please let me know.

I appreciate that AWS is always playing catch-up with the Docker community and I hope this article helps AWS and their users. I suspect ECS will evolve in to a mature AWS offering, but until then there are a lot of sharp edges. You’ll cut yourself. Bring bandages.
