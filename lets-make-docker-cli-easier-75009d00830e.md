Unknown markup type 10 { type: [33m10[39m, start: [33m18[39m, end: [33m31[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m102[39m, end: [33m112[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m116[39m, end: [33m119[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m140[39m, end: [33m155[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m205[39m, end: [33m214[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m321[39m, end: [33m332[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m382[39m, end: [33m387[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m27[39m, end: [33m34[39m }

# Let’s Make Docker CLI Easier

The Docker CLI is not very user friendly, you have to be explicit about lots of options which actually make sense as defaults in everyday usage and debugging when we are working in a multi-container environment. The aim of this post is to define some functions, aliases, and JSON templates to make the CLI easier for us.

![](https://cdn-images-1.medium.com/max/4326/1*-Oder74MJCJwuv4bd43TUQ.png)

[**Docker CLI](https://docs.docker.com/engine/reference/commandline/docker/) (client)** is a wrapper on top of **Docker API** which enables us to send **Docker API calls** down to **Docker Daemon**.

Let’s first look at** most frequent used Docker CLI commands** are their frequently used options (make sure to pay attention to comments):

<iframe src="https://medium.com/media/71936407f3631d25c6072be04ae16b14" frameborder=0></iframe>

Also, below shortcuts can make your life much easier. You can source these scripts in your profile or rc file. But, to get everything pre-setup, you can use [**goyalmunish/devenv](https://hub.docker.com/r/goyalmunish/devenv)** Docker image as your development environment (yes, you can easily connect it to Docker daemon running on your host machine).

<iframe src="https://medium.com/media/fce310d33258c045c109f70126e83ec9" frameborder=0></iframe>

It uses below helper file ([**goyalmunish/devenv](https://hub.docker.com/r/goyalmunish/devenv)** image takes care of all setup for you):

<iframe src="https://medium.com/media/d11c63e1556e61433131867d80384819" frameborder=0></iframe>

Notice that while [docker attach](https://docs.docker.com/engine/reference/commandline/attach/) attaches input, output, and error of docker’s process running through ENTRYPOINT or CMD to your screen, the [docker exec -it](https://docs.docker.com/engine/reference/commandline/exec/) lets you run a one-off command in docker such as /bin/bash (this way you run a shell itself so that you can run other commands). Note that the command started using docker exec only runs while the container’s primary process (PID 1) is running, and it is not restarted if the container is restarted.

**Here are some related interesting stories that you might find helpful:**

* [Let’s make Kubernetes CLI (kubectl) easier](https://medium.com/@goyalmunish/lets-make-kubernetes-cli-easier-5ba7f9c0509a)

* [Why and How to set Probes in Kubernetes? Design a robust K8s cluster](https://medium.com/@goyalmunish/why-and-how-to-set-probes-in-kubernetes-d7da39e94e64)

* [Host vs. Container Environment Variables in docker exec command](https://medium.com/@goyalmunish/passing-host-vs-container-environment-variables-to-docker-exec-5c1b18e6de8e)

* [Docker simplified!](https://medium.com/@goyalmunish/docker-simplified-ad1f8a7350bf)

* [Kubernetes simplified!](https://medium.com/@goyalmunish/kubernetes-simplified-300fef5fb0e6)
