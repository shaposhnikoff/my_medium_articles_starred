Unknown markup type 10 { type: [33m10[39m, start: [33m151[39m, end: [33m182[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m217[39m, end: [33m230[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m149[39m, end: [33m179[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m160[39m, end: [33m188[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m28[39m, end: [33m39[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m44[39m, end: [33m58[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m65[39m, end: [33m92[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m57[39m, end: [33m88[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m46[39m, end: [33m74[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m57[39m, end: [33m87[39m }

# How to Use Own Local Docker Images With Minikube

A step by step guide with an example project

![Photo by [Clem Onojeghuo](https://unsplash.com/@clemono2?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/10800/0*P41Qw179_MSno8T-)*Photo by [Clem Onojeghuo](https://unsplash.com/@clemono2?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

We often have this problem while working with minikube on your local machine where minikube not able to find local docker images. When we install minikube it runs on his own virtual machine and when we run the deployment and it always pulls the Docker images from public or private docker registry.

In this post, we will see how we can use local docker images instead of pulling from the docker registries, for example, docker hub, etc.

* ***Example Project***

* ***Problem We Are Facing***

* ***Solution***

* ***Implementation***

* ***Summary***

* ***Conclusion***

## Example Project

Here is an example project for this post. This is a simple nodejs express app serving on port 3000 and it has the Dockerfile as well to build the image.

    // clone the project
    git clone [https://github.com/bbachi/local-minikube-docker.git](https://github.com/bbachi/local-minikube-docker.git)

    // install and run the project
    npm install
    npm start

    // build the docker image
    docker build -t nodejs-api .

Here is the simple nodejs server running on 3000 which returns hello world on path **/** and returns name on the path **/hello?name=name**.

<iframe src="https://medium.com/media/a7d2dd6a0f985b28aa6f08d7ab0b1e0b" frameborder=0></iframe>

Here is the simple Dockerfile used for this project.

<iframe src="https://medium.com/media/48ef50e4f6e3336ded2cb4d6edb82e95" frameborder=0></iframe>

## Problem We Are Facing

When we install minikube on our machine it comes with its own Docker environment. If we build the Docker images on our machine and try to use that image for the Kubernetes Deployment like below. It always tries to pull the image from the Docker registry or Docker hub and produces an error while starting the pod.

Let’s understand the problem by building the image and running the manifest file on the example project above. Let’s build the image with this command docker build -t nodejs-server . and list images with this command docker images

![**Docker Image**](https://cdn-images-1.medium.com/max/3200/1*Yx9OMTF2Tkumdef_uDkFiA.png)***Docker Image***

Let’s run the** manifest** file here. This is just a Kubernetes deployment object with the image just built above. Create a deployment with this command kubectl create -f manifest.yml

<iframe src="https://medium.com/media/a344ed6cbf5679574066081b3eb34470" frameborder=0></iframe>

If we just check the deployment whether it is successful or not. It fails because of not being able to pull the image. The reason Kubernetes always try to pull the image from the registry instead of searching locally.

![**ImagePullBackOff error**](https://cdn-images-1.medium.com/max/3200/1*qMmr0dxHrH8X_ELSYC_HaQ.png)***ImagePullBackOff error***

If we change the **imagePullPolicy **to** never **and add it to the deployment object. see line number 23. Let’s delete the current deployment and create the new one with this updated file.

<iframe src="https://medium.com/media/d5867bb9a0794197e9fbfc388dda7ecd" frameborder=0></iframe>

You get the **ErrImageNeverPull **error now. That’s because Kubernetes can’t find the Image in its own docker environment. Here is the description with the command kubectl describe <pod name>.

![**ErrImageNeverPull**](https://cdn-images-1.medium.com/max/6672/1*7CCLMsLBqLiASStfjwNlLA.png)***ErrImageNeverPull***

If we just run this command kubectl ssh and docker images. You won’t see the docker image that we just built. ***The reason is the minikube docker daemon and actual docker daemon on your machine are different.***

![**minikube ssh and docker images**](https://cdn-images-1.medium.com/max/3200/1*DKsxGvIu31g2GbNqB9Pseg.png)***minikube ssh and docker images***

## ***Solution***

We know the problem now. Let's find out the solution by following these below steps.

* First, we need to set the environment variable with eval command eval $(minikube docker-env)

* Build the docker image with the Minikube’s Docker daemon docker build -t nodejs-server .

* Make sure you tag it with the same name as in the spec file which is **nodejs-server**

* Make sure that the pods ImagePullPolicy to Never

## ***Implementation***

Let’s implement the above steps one by one and see whether we can use the local images with the minikube.

![**implementation**](https://cdn-images-1.medium.com/max/3200/1*ACjDM8QbnmVISYbBVtOwDw.png)***implementation***

If you see the above image we are able to run the deployment with 5 replicas and all the 5 pods are in running state. If you want to check the deployment working properly or not you can follow these steps.

    // get one of the pod ip
    kubectl get po -o wide

    // run the busybox
    kubectl run busybox --image=busybox -it --restart=Never -- /bin/sh

    // shell
    # wget 172.17.0.5:3000

    // cat index.html
    # cat index.html

![**Deployment working**](https://cdn-images-1.medium.com/max/3200/1*3A8u8kTViZsrbkvkcBVfXg.png)***Deployment working***

## ***Summary***

* Minikube comes with its own docker daemon and not able to find images by default

* We need to set the environment variables with eval $(minikube docker-env).

* We need to build the docker image after we set the environment variables above and make sure to tag the image as same as in the deployment yaml file.

* We have to set ImagePullPolicy to Never in order to use local docker images with the deployment.

* We can unset the environment variables with this command eval $(minikube docker-env -u)

## ***Conclusion***

It’s always convenient to use Minikube and own docker images for the local testing. In this way, we don’t have to use Docker hub or some kind of registry to pull the images. We don’t even deploy in some kind of environment to test our workflow.
