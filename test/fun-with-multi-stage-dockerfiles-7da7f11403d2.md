Unknown markup type 10 { type: [33m10[39m, start: [33m51[39m, end: [33m64[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m149[39m, end: [33m155[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m210[39m, end: [33m216[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m227[39m, end: [33m233[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m360[39m, end: [33m366[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m53[39m, end: [33m57[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m163[39m, end: [33m200[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m293[39m, end: [33m321[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m343[39m, end: [33m355[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m376[39m, end: [33m392[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m213[39m, end: [33m219[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m264[39m, end: [33m276[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m281[39m, end: [33m290[39m }

# Fun with Multi-stage Dockerfiles

Since Docker 17.05, there’s been a feature called multi-stage Dockerfiles. It lets you split your Dockerfile into stages, each with its own environment to perform a set of steps. In a single-stage Dockerfile, the environment you build in is the same environment you deploy. Especially for applications written in a compiled language (even more so if they’re statically linked), there are a lot of build-time dependencies that are not run-time dependencies.

What does a multi-stage Dockerfile look like?

    FROM golang:alpine AS build
    ADD . /go/src/github.com/koki/myproject
    RUN go install github.com/koki/myproject

    FROM alpine
    RUN apk add --update \
      ca-certificates \
      netbase
    RUN mkdir -p /opt/koki
    WORKDIR /opt/koki
    COPY --from=build /go/bin/myproject /opt/koki/myproject
    CMD ["/opt/koki/myproject"]

The first stage is a normal Golang build using the golang:alpine image. The second stage creates the image we actually deploy. Starting from a plain alpine base, we install some certs and copy over the binary — then the image is complete.

![Only the final stage of the Dockerfile arrives at the destination. ([src](https://artgallery.msfc.nasa.gov/4459.html))](https://cdn-images-1.medium.com/max/2000/1*Sqr47-ktgrX_bERKrAhf5Q.jpeg)*Only the final stage of the Dockerfile arrives at the destination. ([src](https://artgallery.msfc.nasa.gov/4459.html))*

## Build leaner images

In the example above, we had a build stage with a full development environment and a deployment setup stage installing a minimal runtime environment. This increases efficiency in a couple ways.

First: Without build tools and other dependencies installed, the resulting image is smaller. It takes less time to download onto the machine where it will run.

Second: It’s not just the total image size that matters. If an image shares layers with images already downloaded onto a machine, those layers won’t have to be downloaded again. If all your images are based on alpine, the base alpine layers only need to be downloaded once — you essentially get those layers for free. It’s less likely that all your images use golang, so it’s more likely you’ll have to wait for those layers to download.

It’s also a matter of storage space. The image cache can use a lot of space. In general, it’s just a good idea to keep your images small and simple.

## Keep secrets safe

Sometimes there’s more to the separation of build-time and run-time environments than efficiency. It can be a matter of security.

For example, you may be building code from a private repository. In order to fetch the code at build time, the build steps in your Dockerfile need the secret credentials to pull code. You definitely don’t want secret credentials in your Docker image. If there’s a vulnerability in the container or the machine it’s running on, your credentials are exposed. If you intentionally release the image as part of a product, your credentials are exposed.

This sort of thing is a long-standing concern with building Docker images. One of the better resources is this (still) open issue from 2014: [*Forward ssh key agent into container #6396](https://github.com/moby/moby/issues/6396)*. There isn’t a baked-in solution, but a multi-stage Dockerfile addresses the problem pretty well.

    **FROM node:8.6-alpine AS build**

    *# Install some build deps + ssh tools for the setup below.*
    RUN apk add --update git openssh python build-base

    *# Set up ssh things to access private Bitbucket repos.*
    ARG SSH_KEY_FILE
    **ADD ${SSH_KEY_FILE} /root/.ssh/id_rsa**
    RUN chmod 0600 /root/.ssh/id_rsa
    RUN ssh-keyscan bitbucket.org >> /root/.ssh/known_hosts

    *# Build things, incl. installing deps from private Bitbucket repos.*
    ADD ./myapp /workspace
    WORKDIR /workspace
    RUN npm install

    *# Remove the secret just in case its directory gets copied below.*
    **RUN rm -vf /root/.ssh/id_rsa**

    *# Set up the final (deployable/runtime) image.*
    **FROM node:8.6-alpine**
    RUN apk add --update python
    **COPY --from=build /workspace /workspace**
    USER node
    WORKDIR /workspace
    ENTRYPOINT ["node" "app.js"]

This Dockerfile has two stages, each starting with a FROM statement. The important steps are highlighted. First, copy the secret credentials into the build image (ADD ${SSH_KEY_FILE} /root/.ssh/id_rsa). Then, after using the credentials to pull private code, delete them from the build image (RUN rm -vf /root/.ssh/id_rsa). The build argument SSH_KEY_FILE defined in the line ARG SSH_KEY_FILE comes from the command line:

    docker build **--build-arg SSH_KEY_FILE =~/.ssh/id_rsa** -f Dockerfile .

It’s also possible to pass the SSH key itself, instead of the path to the key file, as a build argument. However, you have to make sure the SSH key is formatted properly for the command line. One option is to use base64 to encode your SSH key before passing it to docker build and base64 -d to decode it inside the Dockerfile. However, you might need to be careful with the command being too long — especially if you’re passing multiple keys as arguments.

## Conclusion

Build environments and runtime environments are conceptually separate. Multi-stage Dockerfiles make that concept a reality. They help us build leaner images and protect our secrets. They’re an indispensable tool for practicing safe, efficient containerization.
