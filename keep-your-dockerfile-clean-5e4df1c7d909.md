Unknown markup type 10 { type: [33m10[39m, start: [33m15[39m, end: [33m23[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m18[39m, end: [33m24[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m37[39m, end: [33m43[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m171[39m, end: [33m177[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m179[39m, end: [33m196[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m101[39m, end: [33m118[39m }

# Keep your Dockerfile clean

What to do when your Dockerfile gets so big that it’s unmaintainable

![Photo by [David Pisnoy](https://unsplash.com/@davidpisnoy?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/4896/0*wNXVK6_31ka4sfGs)*Photo by [David Pisnoy](https://unsplash.com/@davidpisnoy?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

When a Dockerfile grows beyond a reasonable limit, several problems arise:

* It’s difficult to understand and maintain — we need to read hundreds of lines to understand all the dependencies

* A clear security issue can be overlooked in-between so many lines

* Git will raise more conflicts as everyone is changing the same file

* If we don’t clean up each dependency, it can result in a heavy image

The best solution is to split our Dockerfile into several Dockerfiles so our Dockerfiles are smaller and easier to understand and maintain.

Here are some tips to reduce the size of your Dockerfile.

## Refactor 1: Pull a Dependency From Its Official Image

Avoid creating an artifact copied from an official image.

### Example

Original:

    FROM golang:1.12

    **RUN apt-get update && \
        apt-get upgrade -y && \
        apt-get install -y git openssh-client zip
    WORKDIR $GOPATH/src/github.com/hashicorp/terraform
    RUN git clone [https://github.com/hashicorp/terraform.git](https://github.com/hashicorp/terraform.git) ./ && \
        git checkout v0.12.9 && \
        ./scripts/build.sh**

    WORKDIR /my-config
    COPY . /my-config/
    CMD ["terraform init"]

Refactor:

    **FROM hashicorp/terraform:0.12.9 AS terraform**

    FROM golang:1.12

    **COPY --from=terraform /go/bin/terraform /usr/bin/terraform**

    WORKDIR /my-config
    COPY . /my-config/
    CMD ["terraform init"]

## Refactor 2: Extract a Dependency Into Another Dockefile

When there’s no official image you can pull an artifact from, you should separate its build into another Dockefile.

Then copy the artifacts into the original Dockerfile.

### Example

Original:

    FROM golang:1.12

    **RUN apt-get update && \
        apt-get upgrade -y && \
        apt-get install -y git openssh-client
    WORKDIR /go/src/gitlab.com/sahilm/
    RUN git clone [https://github.com/sahilm/yamldiff.git](https://github.com/sahilm/yamldiff.git)
    RUN cd yamldiff && \
        go get -u github.com/golang/dep/cmd/dep && \
        dep ensure && \
        GOOS=linux go build -o /usr/local/yamldiff**

    WORKDIR /my-app
    COPY . /my-app/
    CMD ["./run.sh"]

Refactor:

Dockerfile for yamldiff:

    **FROM golang:1.12**

    **RUN apt-get update && \
        apt-get upgrade -y && \
        apt-get install -y git openssh-client
    WORKDIR /go/src/gitlab.com/sahilm/
    RUN git clone [https://github.com/sahilm/yamldiff.git](https://github.com/sahilm/yamldiff.git)
    RUN cd yamldiff && \
        go get -u github.com/golang/dep/cmd/dep && \
        dep ensure && \
        GOOS=linux go build -o /usr/local/yamldiff**

    **CMD ["bash"]**

Dockerfile for my app:

    **FROM Marvalero/yamldiff:latest AS yamldiff**

    FROM golang:1.12

    **COPY --from=yamldiff /usr/bin/yamldiff /usr/bin/yamldiff**

    WORKDIR /my-app
    COPY . /my-app/
    CMD ["./run.sh"]

## Refactor 3: Separate Your Image Into Stages

Docker has a multistage feature that comes in handy when your Dockerfile has different parts.

The most common use case is to have a build step and then copy the artifacts in the main image. Having different stages makes your Dockerfile more clear and secure.

### Example

Original:

    **FROM golang:1.12**

    RUN apt-get update && \
        apt-get upgrade -y && \
        apt-get install -y git openssh-client
    WORKDIR /go/src/gitlab.com/sahilm/
    RUN git clone [https://github.com/sahilm/yamldiff.git](https://github.com/sahilm/yamldiff.git)
    RUN cd yamldiff && \
        go get -u github.com/golang/dep/cmd/dep && \
        dep ensure && \
        GOOS=linux go build -o /usr/local/yamldiff

    CMD ["bash"]

Refactor:

    **FROM golang:1.12 as Builder**

    RUN apt-get update && \
        apt-get upgrade -y && \
        apt-get install -y git openssh-client
    WORKDIR /go/src/gitlab.com/sahilm/
    RUN git clone [https://github.com/sahilm/yamldiff.git](https://github.com/sahilm/yamldiff.git)
    RUN cd yamldiff && \
        go get -u github.com/golang/dep/cmd/dep && \
        dep ensure && \
        GOOS=linux go build -o /usr/local/yamldiff

    **FROM ubuntu:18.04**

    **COPY --from=Builder /usr/local/yamldiff /usr/local/yamldiff**

    CMD ["bash"]

## Refactor 4: Sort Multiline Arguments

Whenever possible, sort your multiline arguments. This helps to double-check that no package is duplicated.

![Photo by [Dan Meyers](https://unsplash.com/@dmey503?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/7984/0*fs69RC_NY6c9zoWx)*Photo by [Dan Meyers](https://unsplash.com/@dmey503?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

### Example

Original:

    FROM ubuntu:18.04

    **RUN apt-get -yqq install \
        ca-certificates \
        bash \
        jq \
        wget \
        curl \
        openssh-client \
        build-essential \
        libpng-dev \
        python \
        zip**

    CDM ["bash"]

Refactor:

    FROM ubuntu:18.04

    **RUN apt-get -yqq install \
        bash \
        build-essential \
        ca-certificates \
        curl \
        jq \
        libpng-dev \
        openssh-client \
        python \
        wget \
        zip**

    CDM ["bash"]

## Refactor 5: Tags

Keeping your tags clean is also critical when working with Docker images. I always find it very useful to have three types of tags:

* **Branch name**: identifies the latest version of my image for a specific branch

**Note:** Why not use latest? When using latest, I never know if it means the latest stable version or the latest build in the whole repository. Using the branch names (e.g., master, feature/new-class, etc.) that point to the latest version of a branch is way more intuitive.

* **Version**: needed to differentiate fixes from breaking changes. I recommend using Semantic Versioning (major.minor.patch).

* **Commit**: I always want to know which commit a tag is pointing to. Now, you can do this by creating version tags on your repository. But when this is not possible, just tag your images with its Commit SHA.

## Enjoy Your Pretty Dockerfiles

Thanks for reading. I hope it gets easier for you to maintain Dockerfiles.

If you want to know more about Docker and containers, you can check out my two previous articles:

* [Why You Need Containers](https://medium.com/hacking-talent/containers-part1-why-you-need-containers-6fc895477068)

* [Docker: All You Need To Know](https://medium.com/hacking-talent/docker-all-you-need-to-know-containers-part-2-31120eeb296f)
