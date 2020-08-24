Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m35[39m }

# Fast Docker build in Kubernetes

Speed up docker build with cache in Kubernetes environment

This article describes my recent approach to do fast docker build with the DOCKER_BUILDKIT enhancement introduced since 18.09, and within a Kubernetes environment.

Below is the diagram.

![](https://cdn-images-1.medium.com/max/2244/1*3u2pQS4rRsa4AoCykFeS0w.png)

Note: To perform the demo in this article, a running Kubernetes cluster is required.

## Background

Docker caching layer is very important for a fast docker build when you change a little bit in a large Dockerfile. There are many articles to describe this. [[1](https://testdriven.io/blog/faster-ci-builds-with-docker-cache/)], [[2](https://circleci.com/docs/2.0/docker-layer-caching/)]

In my earlier docker build tasks designed a year ago, my method is to directly use a host machine with docker daemon running. As shown in below diagram.

![](https://cdn-images-1.medium.com/max/2228/1*ndo8_q6Ou5TXoOYOG2D_Kg.png)

It is straight-forward and works, with some drawbacks…

* It’s not highly available. All docker build tasks depend on this host

* As time goes by, the storage space used by docker cache has grown to hundreds of Gigabytes

* And even worse, I didn’t find a way to manage the docker cache. So I couldn’t selectively purge some obsolete cached layers and keep the useful ones

* If the build cache is purged for whatever reason, all image builds start from scratch. And it happened…

* Before Buildkit enhancement, multi-stage-multi-target Dockerfile build unused stages unnecessarily.

Recently I’m trying to do 3 improvements to the tasks. [This article](https://ownyourbits.com/2019/05/13/building-docker-containers-in-2019/) helped a lot to refresh my design.

* Switching to Kubernetes cluster to run docker build in a Pod

* Make use of cache-from to resolve cache problem

* Make use of DOCKER_BUILDKIT enhancement

### Switching docker build to Kubernetes

While moving most infrastructures into Kubernetes, I found I have to keep this single docker host for this single purpose of “docker image building”.

There are 2 major differences to run docker build with a docker host and within a pod in Kubernetes.

1. A dedicate Docker host provides the same “env” for each “docker build’ task, while Kubernetes provides a different Pod every time.

1. Docker host has the permanent storage for cached layers, while a pod has not.

There are a few ways to do docker build in Kubernetes, introduced in [this article](https://medium.com/hootsuite-engineering/building-docker-images-inside-kubernetes-42c6af855f25). I choose the same “docker in docker” approach as the author. I choose this way for its ease of use and the flexibility to use a version of docker of my choice.

For demo purpose I created a sample pod as below. In this article I will build a docker image for Doxygen, hosted [*here](https://github.com/liejuntao001/docker_doxygen). *The built image is in dockerhub as “baibai/doxygen:latest”, [*link](https://hub.docker.com/r/baibai/doxygen/)*.

<iframe src="https://medium.com/media/ff9defcd986ca8596e9dd0e9cdfe6f95" frameborder=0></iframe>

In this pod, there are 2 containers.

First container is named “docker”, using “docker:19.03.3-git” image. Version 19.03.3 is the latest at the time of writing this article. It’s newer than the docker version running on the host machine for the Kubernetes cluster. “19.03.3-git” the git variant provides git command to do build directly from remote git. “docker build” command will be run from this container.

Second one is named “dind”, using “ docker:19.03.3-dind” image. This is the docker-in-docker daemon which will actually do the docker build. DOCKER_TLS_CERTDIR is defined as empty to disable TLS verification.

The first container connects to the docker daemon provided by second container, over tcp://localhost:2375.

So let’s try to do the build in the pod. Example as below.

    $ kubectl apply -f docker_build.yml
    $ kubectl get pod
    NAME                                READY   STATUS    RESTARTS   AGE
    docker-build                        2/2     Running   0          17m
    $ kubectl exec -it docker-build -c docker sh
    / #

    / # docker build [https://github.com/liejuntao001/docker_doxygen.git](https://github.com/liejuntao001/docker_doxygen.git)
    Sending build context to Docker daemon  4.608kB
    Step 1/11 : FROM ubuntu:bionic-20190912.1
     ---> 2ca708c1c9cc
    Step 2/11 : MAINTAINER Liejun Tao "[liejuntao001@gmail.com](mailto:liejuntao001@gmail.com)"
     ---> Using cache
     ---> 33c9c4860773
    Step 3/11 : RUN apt-get update  && DEBIAN_FRONTEND=noninteractive apt-get install -y     doxygen graphviz build-essential texlive-latex-base texlive-fonts-recommended texlive-fonts-extra texlive-latex-extra texlive-font-utils  && rm -rf /var/lib/apt/lists/*
     ---> Running in 0bbc1e3de415
    Get:1 [http://security.ubuntu.com/ubuntu](http://security.ubuntu.com/ubuntu) bionic-security InRelease [88.7 kB]
    ^C
    / #

One thing that’s written right in the document but I ignored at the beginning is to add .git at the end of the github URL. Otherwise I saw an error message “Error response from daemon: Dockerfile parse error line 7: unknown instruction: <!DOCTYPE”.

    docker build [https://github.com/liejuntao001/docker_doxygen**.git](https://github.com/liejuntao001/docker_doxygen.git)**

### Make use of cache-from

The “cache-from” parameter would help resolve the build caching problem. Instead of using the cached layers in local storage, several remote images could be used as references. These 2 article [[1](https://andrewlock.net/caching-docker-layers-on-serverless-build-hosts-with-multi-stage-builds---target,-and---cache-from/)] [[2](https://testdriven.io/blog/faster-ci-builds-with-docker-cache/)] helped me to get an understanding.

As we don’t have any cache when a new Pod is created for each “docker build” task, we have to get some caches from existing images and hope the build to reuse as much as possible.

The usage of “cache-from” is like below pattern. It need a docker Registry to pull images. After build, push the built images to Registry for next-time usage.

    # pull the images that will be referred as cache
    docker pull somethings
    # build
    docker build --tag somethings --cache-from somethings
    # push for next time build
    docker push somethings

Example below shows with the help of cache-from image, the “docker build” took only 1 second, if there is no change in Dockerfile.

    # Remember to delete the pod and create a fresh one

    $ kubectl exec -it docker-build -c docker sh
    / # docker system prune -a
    WARNING! This will remove:
      - all stopped containers
      - all networks not used by at least one container
      - all images without at least one container associated to them
      - all build cache

    / # docker pull baibai/doxygen:latest

    / # time docker build [https://github.com/liejuntao001/docker_doxygen.git](https://github.com/liejuntao001/docker_doxygen.git) --tag baibai/doxygen:latest --cache-from baibai/doxygen:latest
    Sending build context to Docker daemon  4.608kB
    Step 1/11 : FROM ubuntu:bionic-20190912.1
    bionic-20190912.1: Pulling from library/ubuntu
    5667fdb72017: Already exists
    d83811f270d5: Already exists
    ee671aafb583: Already exists
    7fc152dfb3a6: Already exists
    Digest: sha256:b88f8848e9a1a4e4558ba7cfc4acc5879e1d0e7ac06401409062ad2627e6fb58
    Status: Downloaded newer image for ubuntu:bionic-20190912.1
     ---> 2ca708c1c9cc
    Step 2/11 : MAINTAINER Liejun Tao "[liejuntao001@gmail.com](mailto:liejuntao001@gmail.com)"
     ---> Using cache
     ---> 8ffb5c95d189
    Step 3/11 : RUN apt-get update  && DEBIAN_FRONTEND=noninteractive apt-get install -y     doxygen graphviz build-essential texlive-latex-base texlive-fonts-recommended texlive-fonts-extra texlive-latex-extra texlive-font-utils  && rm -rf /var/lib/apt/lists/*
     ---> Using cache
     ---> 52e1fdc3597f
    Step 4/11 : ARG user=dev
     ---> Using cache
     ---> 7c76582d646b
    Step 5/11 : ARG group=dev
     ---> Using cache
     ---> 432c73b054bb
    Step 6/11 : ARG uid=10000
     ---> Using cache
     ---> bc065242f8e4
    Step 7/11 : ARG gid=10000
     ---> Using cache
     ---> 373e56e46bf3
    Step 8/11 : ARG HOME=/home/${user}
     ---> Using cache
     ---> 8b8180b3fd54
    Step 9/11 : RUN groupadd -g ${gid} ${group} &&     useradd -d $HOME -u ${uid} -g ${gid} -m ${user} -s /bin/bash
     ---> Using cache
     ---> 0bfeaee7298d
    Step 10/11 : USER ${user}
     ---> Using cache
     ---> e89e4d7e9946
    Step 11/11 : WORKDIR /home/${user}
     ---> Using cache
     ---> 067d3b46f6e3
    Successfully built 067d3b46f6e3
    Successfully tagged baibai/doxygen:latest
    real    0m 1.77s
    user    0m 0.19s
    sys     0m 0.08s

    / # docker push baibai/doxygen:latest

As it actually takes time to pull images from Registry, it’s better to use a private Registry sitting next to the build environment.

### Make use of DOCKER_BUILDKIT enhancement

There are some introduction articles about the enhancement introduced in 18.09. [[1](https://docs.docker.com/develop/develop-images/build_enhancements/)] [[2](https://blog.mobyproject.org/introducing-buildkit-17e056cc5317)]

I’m mostly interested in this point:

* Detect and skip executing unused build stages

Prior to BUILDKIT enhancement, in multi-stage-multi-target Dockerfile, a target defined in the latter part of Dockerfile might be built with some unnecessary stages.

In below example, target2 doesn’t depend on target1. But if I want to build target2, target1 will be built unnecessarily. If there is any change in target1, it invalidate the caching to cause a re-build of target2.

    stage common1
    stage common2
    stage common3
    stage pre-targer1
    stage target1
    stage pre-target2
    stage target2

First try to do build with Buildkit, oops, I hit an error

    ~ # DOCKER_BUILDKIT=1 docker build [https://github.com/liejuntao001/docker_doxygen.git](https://github.com/liejuntao001/docker_doxygen.git) --tag baibai/doxygen:latest --cache-from baibai/doxygen:latest
    [+] Building 0.0s (1/1) FINISHED                                                                                                                       
     => ERROR [internal] load git source [https://github.com/liejuntao001/docker_doxygen.git](https://github.com/liejuntao001/docker_doxygen.git)                                                           0.0s
    ------
     > [internal] load git source [https://github.com/liejuntao001/docker_doxygen.git](https://github.com/liejuntao001/docker_doxygen.git):
    ------
    failed to solve with frontend dockerfile.v0: failed to resolve dockerfile: failed to build LLB: failed to load cache key: failed to init repo at /var/lib/docker/overlay2/ll0wakjyfu6j27tq64wh6x9fs/diff: exec: "git": executable file not found in $PATH

Not a big deal, I could either clone the source code or add git to dind container.

    # Remember to delete the pod and create a fresh one

    ~ # DOCKER_BUILDKIT=1 docker build [https://github.com/liejuntao001/docker_doxygen.git](https://github.com/liejuntao001/docker_doxygen.git) --tag baibai/doxygen:latest --cache-from baibai/doxygen:latest
    [+] Building 373.4s (8/8) FINISHED                                                                                                                     
     => [internal] load git source [https://github.com/liejuntao001/docker_doxygen.git](https://github.com/liejuntao001/docker_doxygen.git)                                                                 0.4s
     => [internal] load metadata for docker.io/library/ubuntu:bionic-20190912.1                                                                       0.9s
     => importing cache manifest from baibai/doxygen:latest                                                                                           0.6s 
     => [1/4] FROM docker.io/library/ubuntu:bionic-20190912.1@sha256:b88f8848e9a1a4e4558ba7cfc4acc5879e1d0e7ac06401409062ad2627e6fb58                 2.2s
     => => resolve docker.io/library/ubuntu:bionic-20190912.1@sha256:b88f8848e9a1a4e4558ba7cfc4acc5879e1d0e7ac06401409062ad2627e6fb58                 0.0s
     => => sha256:1bbdea4846231d91cce6c7ff3907d26fca444fd6b7e3c282b90c7fe4251f9f86 1.15kB / 1.15kB                                                    0.0s
     => => sha256:2ca708c1c9ccc509b070f226d6e4712604e0c48b55d7d8f5adc9be4a4d36029a 3.41kB / 3.41kB                                                    0.0s
     => => sha256:5667fdb72017d1fb364744ca1abf7b6f3bbe9c98c3786f294a461c2866db69ab 26.68MB / 26.68MB                                                  0.3s
     => => sha256:d83811f270d56d34a208f721f3dbf1b9242d1900ad8981fc7071339681998a31 35.35kB / 35.35kB                                                  0.3s
     => => sha256:ee671aafb583e2321880e275c94d49a49185006730e871435cd851f42d2a775d 850B / 850B                                                        0.2s
     => => sha256:b88f8848e9a1a4e4558ba7cfc4acc5879e1d0e7ac06401409062ad2627e6fb58 1.42kB / 1.42kB                                                    0.0s
     => => sha256:7fc152dfb3a6b5c9a436b49ff6cd72ed7eb5f1fd349128b50ee04c3c5c2355fb 163B / 163B                                                        0.3s
     => => extracting sha256:5667fdb72017d1fb364744ca1abf7b6f3bbe9c98c3786f294a461c2866db69ab                                                         1.3s
     => => extracting sha256:d83811f270d56d34a208f721f3dbf1b9242d1900ad8981fc7071339681998a31                                                         0.0s
     => => extracting sha256:ee671aafb583e2321880e275c94d49a49185006730e871435cd851f42d2a775d                                                         0.0s
     => => extracting sha256:7fc152dfb3a6b5c9a436b49ff6cd72ed7eb5f1fd349128b50ee04c3c5c2355fb                                                         0.0s
     => [2/4] RUN apt-get update  && DEBIAN_FRONTEND=noninteractive apt-get install -y     doxygen graphviz build-essential texlive-latex-base tex  339.8s
     => [3/4] RUN groupadd -g 10000 dev &&     useradd -d /home/dev -u 10000 -g 10000 -m dev -s /bin/bash                                             0.8s 
     => [4/4] WORKDIR /home/dev                                                                                                                       0.0s 
     => exporting to image                                                                                                                           28.7s 
     => => exporting layers                                                                                                                          28.7s 
     => => writing image sha256:c6a86114d67a3a6e02181e9764ad0561f4286b075396db401f4e2035dd467133                                                      0.0s 
     => => naming to docker.io/baibai/doxygen:latest

    / # docker push baibai/doxygen:latest

So this build works, and it took 373 seconds.

### Build with DOCKER_BUILDKIT and cache-from

In this build, it seems although it tried to do “importing cache manifest from baibai/doxygen:latest 0.6s”, but it’s not effective for the apt command. Maybe this is because the existing image is built without Buildkit?

So next experiment, I tried to do similar as previous build with cache-from, e.g first pull the image, then do the build. The image from the Registry is already built with DOCKER_BUILDKIT=1

    # remember to delete the pod and create a fresh one

    # this version of baibai/doxygen:latest is built from previous command with DOCKER_BUILDKIT=1
    / # docker pull baibai/doxygen
    Using default tag: latest
    ......
    Status: Downloaded newer image for baibai/doxygen:latest
    docker.io/baibai/doxygen:latest

    / # DOCKER_BUILDKIT=1 docker build [https://github.com/liejuntao001/docker_doxygen.git](https://github.com/liejuntao001/docker_doxygen.git) --tag baibai/doxygen:latest --cache-from baibai/doxygen:latest
    [+] Building 7.7s (5/7)                                                                                                                                
     => CACHED [internal] load git source [https://github.com/liejuntao001/docker_doxygen.git](https://github.com/liejuntao001/docker_doxygen.git)                                                          0.0s
     => [internal] load metadata for docker.io/library/ubuntu:bionic-20190912.1                                                                       0.3s
     => importing cache manifest from baibai/doxygen:latest                                                                                           0.0s
     => CACHED [1/4] FROM docker.io/library/ubuntu:bionic-20190912.1@sha256:b88f8848e9a1a4e4558ba7cfc4acc5879e1d0e7ac06401409062ad2627e6fb58          0.0s
     => CANCELED [2/4] RUN apt-get update  && DEBIAN_FRONTEND=noninteractive apt-get install -y     doxygen graphviz build-essential texlive-latex-b  7.3s
    failed to solve with frontend dockerfile.v0: failed to build LLB: context canceled

Same thing, it still didn’t use cache for step 2/4 the apt step. So far we couldn’t use cache when build with DOCKER_BUILDKIT=1.

### A secret parameter

I’ve searched quite lot and finally found a hint [here](https://github.com/moby/moby/issues/38815), that the parameter “ — build-arg BUILDKIT_INLINE_CACHE=1” is mentioned. There is brief explanation in buildx document [here](https://github.com/docker/buildx#--cache-tonametypetypekeyvalue).
> --build-arg BUILDKIT_INLINE_CACHE=1 can be used to trigger inline cache exporter
> Export build cache to an external cache destination…inline writes the cache metadata into the image configuration

I believe there should be more clear instructions about this parameter…

A rebuild is required, as the cache metadata is embedded in the image.

    # remember to delete the pod and create a fresh one

    / # DOCKER_BUILDKIT=1 docker build [https://github.com/liejuntao001/docker_doxygen.git](https://github.com/liejuntao001/docker_doxygen.git) --build-arg BUILDKIT_INLINE_CACHE=1 --tag baibai/doxygen:latest --cache-fro
    m baibai/doxygen:latest
    ...

    / # docker push baibai/doxygen:latest

Now the image that we pushed to Registry is built with BUILDKIT_INLINE_CACHE=1.

Check what happens if we do the build again

    # remember to delete the pod and create a fresh one

    / # DOCKER_BUILDKIT=1 docker build [https://github.com/liejuntao001/docker_doxygen.git](https://github.com/liejuntao001/docker_doxygen.git) --build-arg BUILDKIT_INLINE_CACHE=1 --tag baibai/doxygen:latest --cache-from baibai/doxygen:latest
    [+] Building 44.8s (9/9) FINISHED
     => [internal] load git source [https://github.com/liejuntao001/docker_doxygen.git](https://github.com/liejuntao001/docker_doxygen.git)                                                                                                0.3s
     => [internal] load metadata for docker.io/library/ubuntu:bionic-20190912.1                                                                                                      1.0s
     => importing cache manifest from baibai/doxygen:latest                                                                                                                          0.5s
     => [1/4] FROM docker.io/library/ubuntu:bionic-20190912.1@sha256:b88f8848e9a1a4e4558ba7cfc4acc5879e1d0e7ac06401409062ad2627e6fb58                                                0.0s
     => CACHED [2/4] RUN apt-get update  && DEBIAN_FRONTEND=noninteractive apt-get install -y     doxygen graphviz build-essential texlive-latex-base texlive-fonts-recommended tex  0.0s
     => CACHED [3/4] RUN groupadd -g 10000 dev &&     useradd -d /home/dev -u 10000 -g 10000 -m dev -s /bin/bash                                                                     0.0s
     => CACHED [4/4] WORKDIR /home/dev                                                                                                                                              42.9s
     => => pulling sha256:5667fdb72017d1fb364744ca1abf7b6f3bbe9c98c3786f294a461c2866db69ab                                                                                           0.4s
     => => pulling sha256:ee671aafb583e2321880e275c94d49a49185006730e871435cd851f42d2a775d                                                                                           0.1s
     => => pulling sha256:d83811f270d56d34a208f721f3dbf1b9242d1900ad8981fc7071339681998a31                                                                                           0.2s
     => => pulling sha256:7fc152dfb3a6b5c9a436b49ff6cd72ed7eb5f1fd349128b50ee04c3c5c2355fb                                                                                           0.1s
     => => pulling sha256:58ca762fac80a43c7bf90802811bf77349840dab645137847f3ff31fe1261d2c                                                                                           7.8s
     => => pulling sha256:7676292e39d2da183a901f0ab8df6898943f6691255aa880c6b263bb3e696fa5                                                                                           0.1s
     => => pulling sha256:4f4fb700ef54461cfa02571ae0db9a0dc1e0cdb5577484a6d75e68dc38e8acc1                                                                                           0.1s
     => exporting to image                                                                                                                                                           0.0s
     => => exporting layers                                                                                                                                                          0.0s
     => => writing image sha256:8dda7ca6ead30d69d4d5a44dee46f4487982c528eb183daa406932f6f7461595                                                                                     0.0s
     => => naming to docker.io/baibai/doxygen:latest                                                                                                                                 0.0s
     => exporting cache                                                                                                                                                              0.0s
     => => preparing build cache for export

So it works!

In the 1st second of the build, “importing cache manifest from baibai/doxygen:latest” has confirmed match of all layer:

    => CACHED [2/4]
    => CACHED [3/4]
    => CACHED [4/4]

The build time of 44 seconds was actually spent to pull the image from the Registry.

Note in the build, I could ignore the pattern of “docker pull”. Unlike the 3 step pattern “docker pull; docker build; docker push”, when building with “DOCKER_BUILDKIT=1”, the build step will pull the caching images automatically.
> # TL;DR

Create a Pod for docker build task from [this template](https://gist.github.com/liejuntao001/19b0f08a034c9866af8311144d8e7689).

Use below method to build your docker images in Kubernetes.

     DOCKER_BUILDKIT=1 docker build --build-arg BUILDKIT_INLINE_CACHE=1 --tag docker_image_name:latest --cache-from docker_image_name:latest .
    docker push docker_image_name:latest

Thanks for reading.

*UPDATE 10/19/2019*

I found a problem when building real images. I wrote down the problem and solution in my article “Fix Docker in docker network issue in Kubernetes” ([link](https://medium.com/@liejuntao001/fix-docker-in-docker-network-issue-in-kubernetes-cc18c229d9e5)). The Pod template has been updated to reflect the fix.

Based on this article, I’m building a Jenkins pipeline for docker image build task. To be continued…
