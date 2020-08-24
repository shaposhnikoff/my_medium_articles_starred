Unknown markup type 10 { type: [33m10[39m, start: [33m153[39m, end: [33m165[39m }

# 12 Steps To Master Docker and Pass DCA Exam

A step by step guide for anyone who wants to enter DevOps world

![Photo by [Håkon Sataøen](https://unsplash.com/@haakon?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/12032/0*NJzCbq3q0-ihzcO9)*Photo by [Håkon Sataøen](https://unsplash.com/@haakon?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

Docker is one of the essential technology that every developer needs nowadays. It's very important to get familiar with this if you want to succeed as a software engineer or DevOps Engineer. Docker is a technology where you can package your application and essential software to run on any environment you want.

There is a number of ways you can use Docker. Firstly, all the applications are containerized nowadays to facilitate the CI/CD process build once and run everywhere. If your application is containerized once you can run the same container everywhere all you need is Docker runtime. Sometimes you have to replicate the same environment as your production on your local machine to debug the production issue. If you want to run multiple applications together, for example, frontend, backend, and database you can define all in the Docker compose file and run it together with one command.

There are so many other ways you can use it. Once you get familiar with Docker you can use it for your own purpose. In this post, I tried to put together a 10 step guide to get familiar and master the Docker and pass the DCA exam.

* ***Understand The Docker Terminology***

* ***Download and Practice Initial Commands***

* ***Get Familiar with Dockerfile and Create Images***

* ***Container Management***

* ***Create DockerHub Account and Push Images***

* ***Learn Enough Linux For The Docker***

* ***Learn Docker Volumes***

* ***Learn Docker Compose***

* ***Understand the CNM Model and Docker Networking***

* ***Orchestration***

* ***Read Books***

* ***Practice Questions For DCA Exam***

## ***Understand The Docker Terminology***

You want to learn Docker and wondering where to start. Ok. You need to know the Docker terminology first. You don’t have to go through all the jargon right now, you have to familiar with at least below things before getting started.

**Docker Daemon: **It’s a process to manage containers on a single host and accept the REST API requests and process accordingly.

**Docker Image: **Docker image is** **nothing but a big tarball of a layered filesystem. It consists of read-only layers that package the entire dependencies to run any application.

**Container: **A running instance of an image is called a container. Every time you run a container out of image it adds an extra writable layer on top of read-only layers of an image.

**DockerHub:** DockerHub is the service from the Docker which you can share images with the team or the public and it’s also a repository to store images. You can create your own images and push them to the repository.

**Dockerfile: **It’s a set of commands executed sequentially to create an image.

## ***Download and Practice Initial Commands***

Whatever the machine you have: windows, Linux or Mac there is a Docker install for you. [You can visit this pag](https://www.docker.com/products/docker-desktop)e and install Docker on your local machine. Once you install Docker you can see the Docker icon on the top-right if its mac or bottom-right if it’s windows. Here is an example for Mac.

![**Docker Desktop is running**](https://cdn-images-1.medium.com/max/2000/1*5LYjbJHdd6L7gPNGG00HEw.png)***Docker Desktop is running***

There are so many options here where you can log in into Docker Hub, restart the Docker engine, View Documentation, etc. Feel free to play with it and get familiar with the options.

Once you are done with the installation and get familiar with these following commands.

    // check the Docker version
    docker --version

    // since you just installed pull the images from the Docker hub
    docker pull busybox

    // list all the images on your local machine
    docker images

    // run the busybox container
    docker run --name busybox busybox ls

    // pull the nginx image
    docker pull nginx

    // run the nginx container
    docker run --name nginx -p 80:80 nginx 

    // list all running containers on your local machine
    docker ps

    // remove containers
    docker stop <container name/id>
    docker rm <container name/id>

    // remove images
    docker rmi <image id>

## ***Get Familiar with Dockerfile and Create Images***

Now you know how to pull Docker images from the Docker Hub, run those images as containers on your local machine, start and stop those containers and remove those as well. It’s time to write Dockerfile and create your own images.

Dockerfile is nothing but a template that creates images and run containers out of it. Dockerfile is a series of commands that can be executed one by one to create images and can be named as **Dockerfile. **Here is an example of Dockerfile which runs the NGINX server.

In this Dockerfile, we are starting Nginx as the base image, set the working directory, place the text in the default index.html.

<iframe src="https://medium.com/media/9220893a1e04ec8ea482ff569a3f1be1" frameborder=0></iframe>

Create the image from this Dockerfile and run it as a container on your machine with these following commands.

    // build the image (should be in the same location as Dockerfile)
    docker build -t simple-nginx .

    // run the container
    docker run -d --name nginx-server -p 80:80 simple-nginx

    // access it on the browser
    [http://localhost](http://localhost/):80

[Here is a full article about Dockerfile.](https://medium.com/bb-tutorials-and-thoughts/docker-a-beginners-guide-to-dockerfile-with-a-sample-project-6c1ac1f17490) Practice it as much as you can until you are familiar with all the directives of the Dockerfile.

## ***Container Management***

You create images and run containers and it’s time to learn to manage your containers. I would recommend you read the below article to understand container management. Once you are done with this, you should be familiar with listing containers, exec into running containers, stopping containers, removing containers, etc.
[**Docker — Container Management With Examples**
*Exploring how to run and manage containers with different options.*medium.com](https://medium.com/bb-tutorials-and-thoughts/docker-container-management-with-examples-c280906158a8)

## ***Create DockerHub Account and Push Images***

DockerHub is the service from the Docker which you can share images with the team or the public and it’s also a repository to store images. You can create your own images and push them to the repository. Now, it’s time to create a DockerHub account, log in and push some of your own images into the repository.

[Here is the link to create the DockerHub account](https://hub.docker.com/signup?next=%2F%3Fref%3Dlogin). Once you are done with the Docker account and you will have a Docker Id with which you can log in with docker login command.

    // login provide a docker id and password
    docker login

    // we have created image earlier (simple-nginx) push it
    docker push simple-nginx

Once the image is pushed you can actually log into your DockerHub account and see your Image. Here is an example of images I pushed into my DockerHub account.

![**DockerHub**](https://cdn-images-1.medium.com/max/3900/1*9hNb6oG6X7tsfrsQ9RA7GQ.png)***DockerHub***

## ***Learn Enough Linux For The Docker***

As most of the workloads run on the Linux operating system, getting familiar with this is very important. You don’t have to master Linux but learn enough for the Docker. How much you want to learn is subjective and depends on the type of application you are working on. You need to learn at least these topics. There are so many online resources available for the Linux select one go through all the following topics.

* ***Open Source Philosophy***

* ***Why there are so many Linux distributions***

* ***Fundamental Concepts***

* ***How to access Linux Distribution***

* ***Understanding File System***

* ***Command Line Basics***

* ***How to get help with the command line***

* ***How to use directories and list files***

* ***How to work with files***

* ***How to write a script***

* ***Permissions***

## ***Learn Docker Volumes***

At this stage, you are pretty familiar with container management. You should know that containers don’t persist data. For example, if you have a running container and persists some data and dies after some time you no longer access that data. Each container has a writable layer on top of the read-only image layers. If you persist any data in the container you write that data into this writable layer. As containers stop or removed this writable layer will be deleted and you no longer access the data. To solve this problem we have Volumes.

***Volumes ***are saved in the host filesystem ***(/var/lib/docker/volumes/) ***which is owned and maintained by docker. Any other non-docker process can’t access it. But, other docker processes/containers can still access the data even container is stopped since it is isolated from the container file system. [Here is the guide to understanding the Docker volumes with an example.](https://medium.com/bb-tutorials-and-thoughts/understanding-docker-volumes-with-an-example-d898cb5e40d7)

## ***Learn Docker Compose***

We can create a container and run it anywhere with Dockerfile. if we want to run multiple services in separate ports, we could still do it running it with two Dockefiles in multiple terminals.

But what if we want to define a configuration file that has all the details about containers, start the containers with one command and uses this config file anywhere we want. The answer is ***docker-compose. [***Here is an article about Docker compose](https://medium.com/bb-tutorials-and-thoughts/docker-compose-part-1-development-environment-for-multi-container-applications-6e3ba461c219).

## ***Understand the CNM Model and Docker Networking***

Docker networking is the most important part when it comes to deploying and running applications on the Docker. You need to understand what are the available Network drivers and their use cases, how to publish a port so that it can be accessible to external. Here are some of the links to help you understand.

* [Docker Network Drivers](https://docs.docker.com/network/)

* [The Container Network Model](https://github.com/docker/libnetwork/blob/master/docs/design.md)

## ***Read Books***

Once you go through all the above. It’s time to read some books and there are so many books out there. Each book gives you a different perspective. I would recommend you read [at least this one.](https://www.amazon.com/Docker-Deep-Dive-Nigel-Poulton-ebook/dp/B01LXWQUFF)

* [Docker Deep Dive by Nigel Poulton](https://www.amazon.com/Docker-Deep-Dive-Nigel-Poulton-ebook/dp/B01LXWQUFF)

## ***Practice Questions For DCA Exam***

It’s time to practice for the DCA exam. The below two links help you validate your knowledge and practice for the exam.

* [250 Practice questions for the DCA exam](https://medium.com/bb-tutorials-and-thoughts/250-practice-questions-for-the-dca-exam-84f3b9e8f5ce)

* [Github link for the above](https://github.com/bbachi/DCA-Practice-Questions)

## Conclusion

Master it as soon as possible.
