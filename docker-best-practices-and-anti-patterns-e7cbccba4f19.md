Unknown markup type 10 { type: [33m10[39m, start: [33m79[39m, end: [33m98[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m542[39m, end: [33m555[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m32[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m205[39m, end: [33m221[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m245[39m, end: [33m251[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m28[39m, end: [33m38[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m236[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m184[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m37[39m, end: [33m41[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m446[39m, end: [33m450[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m116[39m, end: [33m120[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m67[39m, end: [33m78[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m121[39m, end: [33m129[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m258[39m, end: [33m262[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m266[39m, end: [33m267[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m9[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m156[39m, end: [33m159[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m66[39m, end: [33m71[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m100[39m, end: [33m105[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m49[39m, end: [33m59[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m30[39m, end: [33m34[39m }

# Docker Best Practices and Anti-Patterns

How to build production-ready containers

![Photo by [Matthew Kwong](https://unsplash.com/@mattykwong1?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/premier?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/8544/1*vBPEQmji5XSKEMBq4I5GQA.jpeg)*Photo by [Matthew Kwong](https://unsplash.com/@mattykwong1?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/premier?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

Most of the time with Docker, we don’t pay attention to its inner workings. Just because you fire up a Docker container and your application is running doesn’t mean you’ve implemented a good solution. Sometimes, thanks to time constraints, we fall into the trap of copy-pasting Docker images without understanding the implementation details and the nuances of how it was built.

In this article, we’ll be looking at Docker best practices and anti-patterns. Anti-patterns are a common response to a recurring problem — when we implement ineffective and counterproductive solutions that can undermine our Docker stack.

Let’s jump in and take a look at some of the things that we may be doing wrong.

## We Need Those Tags

Tagging is a must, allowing you to convey useful information about your Docker images. Think of tags as aliases to your docker image ID. They can be compared to Git tags, which refer to a particular commit in your history, allowing you to version your Docker images at different points in time. Forgetting to tag is a small thing that has a few drawbacks.Namely, if no tags are specified the default image will be tagged with the latest.

    FROM your_image_name:latest

If you do this often enough your image may not actually be latest and may refer to an older version instead. Always use appropriate tags and adhere to a versioning schema, such as [semantic versioning](https://semver.org/). This ensures that users of a Docker image can guarantee compatibility, stay updated, and use the correct versions predictably.

There’s another thing you might want to avoid. The latest default tag is using FROM python3:latest for example in your Dockfile to pull the latest image from a Docker registry. At first glance this might seem like a good idea but there are some unintended side effects — each latest pull might derive an entirely different Docker image than the previous build. Trying to decipher what’s broken your Docker image may be hard since from your end you expect immutability. Hence there's a strong case to use a specific tag for an image (example: python3:1.0.1). This will ensure your Dockerfile remains immutable.

## Running Multiple Services in the Same Container

Yes, by all means, you can do this — probably successfully — but they are a couple of reasons why you might not want to go down this path. When we work with Docker services single responsibility is something we should strive for. It’s best-practice that every different service which composes your application should run in its own container —by all means, try to package each piece of independent functionality into separate independent container images.

It’s may be tempting to add multiple services to one Docker image — but the notion of thinking of container images as virtual machines should also be abandoned. Having multiple services in one container makes it hard to horizontally scale your application. A core concept of Docker containers is that they’re transient and designed for distribution, which is ideal for modern web applications where the transient nature simplifies scaling and concurrency. Adding multiple services increases the difficulty of managing this distribution.

Additionally, multiple services on a single container make managing security harder. Bloated image sizes could slow down CI/CD, if that’s a concern for you. The Docker [documentation does ](https://docs.docker.com/config/containers/multi-service_container/)a pretty good job of elaborating on this further.

## Use LABEL to Catalog your Images

This is by no means an antipattern but I thought it deserved a mention. One of the things I’ve noticed when working with various Docker images is that sometimes the creators of these images don’t have the LABEL maintainer tag. This tag sets the Author field of the image in events — when things go wrong or you need clarification it’s easier to know who to contact internally or external if the image is shared publicly.

This is by no means the only label you can use. You can define various labels, as per your need to catalog your images, record licensing information or labels that are useful for automation.

Multiline labels other than maintainer:

    # Set one or more individual labels
    LABEL com.example.version="0.0.1-beta"
    LABEL vendor1="RBTSB Incorporated"
    LABEL vendor2=TIPTAPCODE\ Incorporated
    LABEL com.example.release-date="202-04-02"
    LABEL com.example.version.production="0.0.1"

Single line labels prior to Docker 1.10 would create new docker layers so if you are using the latest Docker version, you don’t have to worry about the overhead layers.

    LABEL vendor=ACME\ Incorporated \
          com.example.is-beta= \
          com.example.is-production="" \
          com.example.version="0.0.1-beta" \
          com.example.release-date="2015-02-12"

It’s always a good practice to add as much metadata as possible to your immutable Docker Images for traceability, visibility, and maintainability.

## Avoid Building Environment Specific Images

When we build Docker images we should always keep immutability in mind. It’s good practice to not use different images tagged as, for example, dev, test, staging, and production as this voids the principle of a single source of truth. Another issue is that there is no guarantee that images are similar in nature when performing verification or debugging on different environments.

## Why Use a Non-Root Container?

By default, Docker containers run as root. A Docker container running as root has full control of the host system. This is not desirable due to security concerns. Docker containers images that run with non-root add an extra layer of security and are generally recommended for production environments. However, because they run as a non-root user, privileged tasks are typically off-limits. You need to do some context switching if leveraging the USER instruction to specify a non-root user, as illustrated in the example below, is required.

    FROM python:3.6-slim-buster
    LABEL maintainer="Timothy Mugayi <timothy.mugayi@gmail.com>"

    RUN apt-get update && apt-get install -y --no-install-recommends \
        wget && rm -rf /var/lib/apt/lists/*

    # Dumb init
    RUN wget -O /usr/local/bin/dumb-init [https://github.com/Yelp/dumb-init/releases/download/v1.2.0/dumb-init_1.2.0_amd64](https://github.com/Yelp/dumb-init/releases/download/v1.2.0/dumb-init_1.2.0_amd64)
    RUN chmod +x /usr/local/bin/dumb-init

    RUN pip install --upgrade pip

    WORKDIR /usr/src/app

    COPY requirements.txt .

    RUN pip install -r requirements.txt

    COPY helloworld.py .

    USER 1001

    ENTRYPOINT ["/usr/local/bin/dumb-init", "python3", "-u", "./helloworld.py"]

If you use a base image that was not generated using root and you have to switch back you can perform the following:

    FROM <namespace>/<image>:<tag_version>
    USER root

## Don’t Run Too Many Processes Inside a Single Container

The beauty of containers — and also the advantage of containers over virtual machines — is that it’s easy to make multiple containers interact with one another in order to compose a complete application. There’s no need to run a full application inside a single container. Instead, as much as possible, break your application down into discrete services, and distribute services across multiple containers. This maximizes flexibility and reliability.

Yes, it is possible to install and run a complete Linux operating system inside a container. But should you do it?

It’s probably not such a great idea. Docker images are built with the concept of layers, so the more you add to the image the more bloated it gets. A full-blown OS voids the ideal use-case for Docker. Ideally, install only the essential pieces that are necessary inside your containers.

To make the most of containers, you want each of them to be as lean as possible. This maximizes performance and minimizes security risks. For this reason, avoid running services that are not strictly necessary. For example, unless it is absolutely essential to have an SSH service running inside the container — which is probably not the case because there are other ways to log in to a container, such as with the Docker exec call — don’t include an SSH service.

## Don’t Run Unnecessary Services Inside a Container

When working with Docker there’s a tendency to bloat our images with tools such as sonar for code coverage.

Use the builder pattern [Docker Multistage builds](https://docs.docker.com/engine/userguide/eng-image/multistage-build/) — since Docker CE 17.05+ you now have the ability to have multiple FROMstages in a Dockerfile. The ephemeral builder stage container will be discarded so the final runtime container image will be lean. A practical use case is when you need to compile some binaries from source files then, on second data, copy those leaner binaries to your final image.

### **Derived benefits**

* Builds are faster i.e you CI/CD process can be improved with leaner images that take less time as well to transmit via the network.

* Need less storage.

* Cold starts (image pull) are faster.

* Potentially less attack surface.

### **Drawbacks**

* Fewer tools inside a container, but a small price to pay to keep your containers lean.

## Include Oberservablity Tools in Your Containers

You can run containers without monitoring solutions in place, but something you need to keep in mind is it becomes inherently hard for you to know what is going on inside your containers — especially as the number of containers increases.

Docker out of the box exposes very detailed metrics about CPU, memory, network, and I/O usage for each running container via the /stats endpoint of Docker’s remote API. App dynamics and Newrelic, to mention two, have ready-made agents that can be prepackaged with your Docker image to give you more visibility, including application-level, on how your applications and containers are performing.

## Love-Hate the Arbitrary Base Images
> # “Seriously, do you know who built the image and what was added to it?”

It’s all about traceability. Keeping security in mind is something that all software developers should always strive to do. Learn how to trace the origin of Docker images and understand their content. ]

There are are a couple of things you need to keep in mind:

* How the image was created.

* Verify that it isn’t changed after its creation.

* Verify the content of the image.

* Scan for security vulnerabilities.

There are some tools for conducting static analysis of containers. They’re pretty comprehensive and go well beyond the scope of this article, but I advise you to take some time out to learn how they work.

[Clair](https://github.com/coreos/clair) is an interesting tool that provides automatic container vulnerability and security scanning for your Docker applications. Scans are based on a common vulnerabilities and exposure (CVE) database. If you’re running Docker locally you can give it a spin by downloading Postgres and linking Clair to it. Below are the minimum configurations you will need to get it up and running:

    $ mkdir $PWD/clair_config

    $ curl -L [https://raw.githubusercontent.com/coreos/clair/master/config.yaml.sample](https://raw.githubusercontent.com/coreos/clair/master/config.yaml.sample) -o $PWD/clair_config/config.yaml

    $ docker run -d -e POSTGRES_PASSWORD="" -p 5432:5432 postgres:9.6

    $ docker run --net=host -d -p 6060-6061:6060-6061 -v PWD/clair_config:/config quay.io/coreos/clair:latest -config=/config/config.yaml

If you wish to change the port for Postgres, ensure you change the config.yaml file as well. If you already have another Postgres running on your system take care not to change the docker port forward to something other than 5432.

    clair:
      database:
        *# Database driver
        *type: pgsql
        options:
          *# PostgreSQL Connection string
          # [https://www.postgresql.org/docs/current/static/libpq-connect.html#LIBPQ-CONNSTRING](https://www.postgresql.org/docs/current/static/libpq-connect.html#LIBPQ-CONNSTRING)
          *source: host=localhost port=5432 user=postgres password=123456 sslmode=disable statement_timeout=60000

Once your images are fired up and executing, you can run Docker ps to ensure your containers are up and running

![](https://cdn-images-1.medium.com/max/5488/1*nY88oT5rjCrMEe5DELix6Q.png)

Now that you’ve got this far you need to be aware that Clair doesn’t have a web UI or a CLI — the only way to work with it is via its REST API or a third-party CLI tool. To get you on your way, see more details [here](https://www.nearform.com/blog/static-analysis-of-docker-image-vulnerabilities-with-clair/) — how to use Clair goes beyond the scope of this article.

Bayan [Collector](https://github.com/banyanops/collector) is a light-weight app that allows you to perform [static analysis](https://searchwindevelopment.techtarget.com/definition/static-analysis) by launching containers from a registry, run arbitrary scripts and gather useful information (e.g., packages installed), enforce policies, validate invariants in your images. For more details see the GitHub repository.

[Docker Bench for Security](https://github.com/docker/docker-bench-security) is a tool created by the Docker team that runs through a checklist of security best practices to adhere to on a Docker host and flags any issues it finds. For more details see the GitHub repository.

## Don’t Store Sensitive Data Inside a Container Image

This is a mistake you want to avoid. It could lead to private and sensitive information being leaked to the outside world if your Docker images are publicly shared, or if developers inadvertently push images to public Docker registries. You should never use COPY or ` with sensitive information in a Dockerfile.

To avoid this, store sensitive data on a secure filesystem to which the container can connect. In most scenarios, this filesystem would exist on the container host or be available via block storage such as [AWS Elastic Block Storage (EBS)](https://cloudacademy.com/course/aws-storage-fundamentals-2016/amazon-elastic-block-store-ebs-1/) or object storage services such as S3.

Additionally, you should avoid storing security credentials in Docker images. As developers, we’re sometimes tempted to take shortcuts and hard code passwords and secret keys. Get into the habit of using e-arguments to specify environment variables for your Docker containers at runtime.

Leverage on — env-file, which can also be used to read environment variables from a file. Custom scripts that pull credentials from third-party sources via CMD or `can also be used to pull in relevant credentials required by your docker container.

## Don’t Store Data or Logs Inside Containers

Containerization changes the nature of logging. Containers are ideal for stateless applications that are transient in nature and are meant to be ephemeral aka (short-lived). Any data that might be stored inside a running container is ephemeral in nature — as you’ve probably noticed, when the container shuts down your data is lost. So it makes sense to store data out of the Docker container. There are tools to help you to extract Docker logs and put them in more permanent data stores.

Something to keep in mind when approaching how to tackle logging with Docker: A Docker installation has at least three distinct levels of logging; i.e the Docker container, the Docker service, and the host operating system; your choice of logging method should be able to extract logs on all layers.

## Don’t Write to Your Container’s Filesystem

Every time you write something to a container’s filesystem, it activates [copy on write strategy](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/#container-and-layers). A new storage layer is created using a storage driver (devicemapper, overlayfs or others). In active usage, it can put a lot of load on storage drivers, especially in the case of Devicemapper or BTRFS.

Make sure your containers only write data to volumes. You can use tmpfs for small temporary files — tmpfs is a temporary filesystem that resides in memory and/or swap partition(s).

## Don’t Run PID 1

This is a common problem that most people probably don’t know about.

Docker does not run processes under a special [init process that properly reaps child processes](http://en.wikipedia.org/wiki/Zombie_process), so it's possible for the container to end up with zombie processes that may cause unintended issues to arise.

### Use tini or dumb-init

PID 1 is special in UNIX so omitting an init system often leads to incorrect handling of processes and signals. This can result in problems such as containers which can’t be gracefully stopped, or leaking containers which should have been destroyed.

Zombie processes are processes that have stopped running but their process table entry still exists because the parent process hasn’t retrieved it via the wait syscall. Technically, each process that terminates is a zombie for a very short period of time but they can live for longer.

[Tini](https://github.com/krallin/tini) or [dumb-init](https://github.com/Yelp/dumb-init) can be used if you have a process that spawns new processes but doesn’t have good signal handlers implemented to catch child signals and stop your child if your process should be stopped. Bash scripts, for example, do *not *handle and emit signals properly.

Here is an example of how to run dumb init where prepare.sh is either a shell script or command to execute your application`:

    RUN wget -O /usr/local/bin/dumb-init [https://github.com/Yelp/dumb-init/releases/download/v1.2.0/dumb-init_1.2.0_amd64](https://github.com/Yelp/dumb-init/releases/download/v1.2.0/dumb-init_1.2.0_amd64)

    RUN chmod +x /usr/local/bin/dumb-init

    ENTRYPOINT ["/usr/local/bin/dumb-init", "/usr/bin/prepare.sh"]

Alternatively, if you opt for tini, below is an example of how you can set it up if you’re are using Python anaconda conda:

    RUN conda install --yes -c conda-forge tini

    ENTRYPOINT** **["tini", "-g", "--", "/usr/bin/prepare.sh"]

Finally, here’s a more generic approach that does not depend on any programming language:

    FROM node:13.12.0-slim
    
    MAINTAINER Timothy Mugayi <timothy.mugayi@gmail.com>
    
    ENV TINI_VERSION='v0.13.0'
    
    # Add tini init, see https://github.com/krallin/tini
    ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
    
    RUN chmod +x /tini
    
    # Set tini as entrypoint
    ENTRYPOINT ["/tini", "--"]

## Apply DockerFile Linting into Your CI/CD

Apparently Docker has linters too — just like your programming languages.

Taking advantage of linters has a lot of benefits since enforcing best practices verbally is hard. Applying linting to your CI/CD may help your team avoid common mistakes and establish best practices when building out your production Docker images. A good place to start is leveraging on [hadolint](https://github.com/hadolint/hadolint), a Haskell Dockerfile Linter that parses your Dockerfiles into an [AST](https://en.wikipedia.org/wiki/Abstract_syntax_tree) and performs rules on top of the AST.

Let's run through an example. To perform linting using Docker you can perform the following:

    $ docker run --rm -i hadolint/hadolint < Dockerfile

Once the lint is complete below results will be displayed, which may vary depending on the nature of your Dockerfile.

![hadolint lint results](https://cdn-images-1.medium.com/max/5480/1*ajVYzflKLKgtlct8gNKOFA.png)*hadolint lint results*

The DL3008 code illustrated above “Pin versions in apt get install” based on hadolint code description states. Version pinning forces the build to retrieve a particular version regardless of what’s in the cache. This technique can also reduce failures due to unanticipated changes in required packages.

## Final Thoughts

We’ve covered a lot — from security to streamlining your Docker images.

There are more things you can do to make your Docker images better and more secure. Hopefully, this article has given you some insights into the things you should or shouldn't be doing and possible solutions that you can start applying to your internal or external docker containers and images.

Thanks for reading, if you have any other recommendations please leave a comment below. If you think the content will be useful for the next developer please share it.

Stay safe, happy Docker image building!
