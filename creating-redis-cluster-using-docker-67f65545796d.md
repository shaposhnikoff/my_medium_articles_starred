
# Creating Redis Cluster using Docker



For starters**,** I strongly advise reading this [document](https://redis.io/topics/cluster-tutorial) to grasp a greater perspective about **Redis Cluster**.

First things first, you will need **Docker** in your environment. Follow the [instructions](https://docs.docker.com/engine/installation/) to install it. After the installation complete you can check with

    docker version

As of writing this document, I am using version 1.32.

Let**’**s start with pulling the [Redis](https://hub.docker.com/_/redis/) docker image.

    docker pull redis

If you want to have the same installation all the time don’t forget to set a tag on the docker image. As of writing this document, the latest distribution of Redis is tagged as 3.2 (redis:3.2).

Since we have a cluster on the local environment and want them to communicate with each other, we have to put them in the same network or link them. **Linking is not advised by Docker and also Redis Cluster document suggests to use networking mode. Hence, we will use networking mode to connect our containers.**

    docker network create red_cluster

You can view your newly created network with

    docker network ls

Now let’s create a Redis configuration file to enable cluster mode on for our containers. The following is a very basic configuration file, you can find additional properties on Redis official website.

    port 6379
    cluster-enabled yes
    cluster-config-file nodes.conf
    cluster-node-timeout 5000
    appendonly yes

The mentioned port above is the exposed port for of a Redis node. The value of cluster-node-timeout is in milliseconds. nodes.conf is created and managed automatically, we don’t need to do anything except adding this line.
Now, we are ready to create our cluster. Create redis containers with provided configuration.

    docker run -d -v \
    $PWD/cluster-config.conf:/usr/local/etc/redis/redis.conf \
    --name redis-1 --net red_cluster \
    redis redis-server /usr/local/etc/redis/redis.conf

Create as many node as you want, and when enough container is created get container IP addresses with following command

    docker inspect -f '{{ (index .NetworkSettings.Networks "red_cluster").IPAddress }}' redis-1

This command will return IP address of the container under red_cluster network (e.g. 172.19.0.2). All containers should have different IP addresses.

Now, we need to create a cluster from these containers. We’ll need a ruby script named as ‘**redis-trib.rb**’. This script is officially distributed by redis.io. You can use your local ruby installation or use a ruby docker container to run the script. If you are going to use the docker option, you will also need to add the ruby container into ‘**red_cluster**’ network, otherwise it will not be able to access Redis containers via their IP addresses.
**(Note that IP addresses may differ)**

    docker run -i --rm --net red_cluster ruby sh -c '\
     gem install redis \
     && wget [http://download.redis.io/redis-stable/src/redis-trib.rb](http://download.redis.io/redis-stable/src/redis-trib.rb) \
     && ruby redis-trib.rb create --replicas 1 172.19.0.2:6379 172.19.0.3:6379 172.19.0.4:6379 172.19.0.5:6379 172.19.0.6:6379 172.19.0.7:6379'

However creating a bash script to create containers and cluster is much easier.

    #!/usr/bin/env bash

    for ind in `seq 1 6`; do \
     docker run -d \
     -v $PWD/cluster-config.conf:/usr/local/etc/redis/redis.conf \
     --name "redis-$ind" \
     --net red_cluster \
     redis redis-server /usr/local/etc/redis/redis.conf; \
    done

    *echo *'yes' | *docker *run -i --rm --net red_cluster ruby sh -c '\
     gem install redis \
     && wget http://download.redis.io/redis-stable/src/redis-trib.rb \
     && ruby redis-trib.rb create --replicas 1 \
     '"**$**(**for **ind **in ***`*seq 1 6*`*; **do **\
      echo -n "**$**(docker inspect -f \
      '{{(index .NetworkSettings.Networks "red_cluster").IPAddress}}' \
      "redis-$ind")"':6379 '; \
      **done**)"

Let’s inspect two parameters,

    ' --replicas 1' 

Creates 1 replica for 1 master, and since we have 6 containers, Redis will create 3 master and 3 slave for us.

    docker inspect -f \
      '{{(index .NetworkSettings.Networks "red_cluster").IPAddress}}' \
      "redis-$ind"

And this docker command returns IP addresses of the containers within the “red_cluster” network.

After this the last command is executed, our cluster will be up and running. You can check master and slave nodes whether they are running using the following command

    docker exec redis-1 redis-cli cluster nodes

However, you cannot access this cluster from the outside network since we do not expose any ports to host. Therefore, you should run your application containers on this Docker network.

Let’s assume we have a simple Spring Boot application that uses Redis as cache. We need to set IP addresses of our master nodes in *application.properties* file.

    spring.cache.type=redis
    spring.redis.cluster.nodes=172.19.0.2:6379,172.19.0.3:6379,172.19.0.4:6379

Assume that we have created a single RESTful service, and prepared a *Dockerfile* for it. Create an image of this application.

    docker build -f Dockerfile -t testApp .

Now our image is built and we can create a container to run our application. Note that, it is important to set the network parameter to access Redis cluster.

    docker run -dp 8080:8080 --net=red_cluster --name testAppCon testApp

And that is it. The application will work with the Redis Cluster.
> ***References***
> [https://redis.io/topics/cluster-tutorial](https://redis.io/topics/cluster-tutorial)
> [https://redis.io/topics/cluster-spec](https://redis.io/topics/cluster-spec)
> [https://github.com/docker-library/redis](https://github.com/docker-library/redis)
