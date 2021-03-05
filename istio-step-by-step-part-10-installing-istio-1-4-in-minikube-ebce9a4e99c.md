
# Istio step-by-step Part 10 — Installing Istio 1.4 in Minikube

Hi all, here I bring my 10th tutorial of Istio step-by-step. This tutorial is based on my second article (istio-step-by-step-part-02-getting-started-with-istio), which was written based on Istio 1.0.2 version. But new Istio 1.4, has many changes starting from installation. So, please refer to this article if you are using the latest version of Istio (1.4).

![Photo credits: [https://www.pexels.com/photo/access-close-up-computer-connection-306198/](https://www.pexels.com/photo/access-close-up-computer-connection-306198/)](https://cdn-images-1.medium.com/max/8064/1*dY2-W02gB7oMCqjNGqrkxA.jpeg)*Photo credits: [https://www.pexels.com/photo/access-close-up-computer-connection-306198/](https://www.pexels.com/photo/access-close-up-computer-connection-306198/)*

### Prerequisite

1. Install Kubectl, Minikube and Virtual box.

1. Start minikube minikube start

![](https://cdn-images-1.medium.com/max/3400/1*nOFVARUPML19KVXyK3LDTA.png)
> You can also refer to my first two articles for more information.
[**Istio step-by-step Part 01 — Introduction to Istio**
*Hi! Welcome to my Istio step-by-step tutorial series. As the first tutorial, I will do a small introduction about…*medium.com](https://medium.com/faun/istio-step-by-step-part-01-introduction-to-istio-b9fd0df30a9e)
[**Istio step-by-step Part 02 — Getting started with Istio**
*Hi! Welcome to my Istio step-by-step tutorial series. Through this tutorial, I will tell you how to install Istio in…*medium.com](https://medium.com/faun/istio-step-by-step-part-02-getting-started-with-istio-c24ed8137741)

Let’s start installing Istio.

### Step 01

Download the latest release.

    curl -L [https://istio.io/downloadIstio](https://istio.io/downloadIstio) | sh -

### Step 02

Move to the downloaded folder.

    cd <directory>

### Step 03

Add istioctl path to your OS.

    export PATH=$PWD/bin:$PATH

![](https://cdn-images-1.medium.com/max/6108/1*eq1wphPDAQpPrGLOx4mnmw.png)

### Step 04

Installing Istio. This is the final step. New Istio has configuration profiles for installing Istio. There are few build-in configuration profiles,

1. default — this will enable the default settings of Istio **control plane**.

1. demo — this profile is to show the key functions in Istio. Bookinfo application and the tasks associated with it can be executed with this profile. *(We are going to move on in this article with this application)*

1. minimal — this has the minimal configuration set for Istio traffic routing.

1. sds —(SDS-Secret Discovery Service) this profile is a bit similar to the default profile. But this profile comes with few additional authentication features which are enabled by default (Strict Mutual TLS). These features enable Istio’s SDS.

1. remote — this profile is used when working with remote clusters in a multicluster mesh which shares single control plane configurations.
> I will discuss about Istio Deployment modes in another tutorial.
> Check for my next tutorial on [Customize Istio installation](https://medium.com/@nethminiromina/istio-step-by-step-part-11-customize-istio-installation-4d6af88b5fb7).

You can refer [https://istio.io/docs/setup/additional-setup/config-profiles/](https://istio.io/docs/setup/additional-setup/config-profiles/) for information regarding Installation configurations. Now let’s come back to install Istio.

Install the demo profile

    istioctl manifest apply --set profile=demo

![](https://cdn-images-1.medium.com/max/2000/1*sxe1lVfbt5ANVNs8L2dzVg.png)

![](https://cdn-images-1.medium.com/max/2000/1*c0mindnU4ZHQ_TjtAYWx_w.png)

You can check for pods as services by executing the following commands.

    kubectl get svc -n istio-system

    kubectl get pods -n istio-system

![](https://cdn-images-1.medium.com/max/4008/1*InSG6bRwUyI2eA2BDQ8Y_w.png)

You can see all pods are up and running. So, this is all for this tutorial. If you need more information, as I mentioned before, please check my [tutorial 02](https://medium.com/faun/istio-step-by-step-part-02-getting-started-with-istio-c24ed8137741). Let’s get back with the next tutorial.

<< [Previous article](https://medium.com/faun/istio-step-by-step-part-09-whats-new-in-istio-1-4-8cdea2555ca3)

[Next article](https://medium.com/@nethminiromina/istio-step-by-step-part-11-customize-istio-installation-4d6af88b5fb7) >>

### References

1. [https://istio.io/docs/setup/getting-started/](https://istio.io/docs/setup/getting-started/)

1. [https://istio.io/docs/setup/additional-setup/config-profiles/](https://istio.io/docs/setup/additional-setup/config-profiles/)

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
