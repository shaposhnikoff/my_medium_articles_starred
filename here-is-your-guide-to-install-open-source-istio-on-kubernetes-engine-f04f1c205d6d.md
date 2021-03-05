
# Here is your guide to install open source Istio on Kubernetes Engine

Qwiklabs is pretty “qwik” in adapting and spreading the knowledge when it comes to new technologies. Anthos is one of the latest application management platform and now you can get a hand on experience on Anthos using Qwiklabs. Anthos: Service Mesh is an interesting quest launched by Qwiklabs.

[Here](https://medium.com/@qwiklabs/a-mesh-made-in-heaven-17fde3bd796d?utm_source=Medium&utm_medium=blog&utm_campaign=anthos) is an insight into this quest. In this blog we will take a look at the second lab of the quest [**Installing Open Source Istio on Kubernetes Engine](https://google.qwiklabs.com/catalog_lab/2193?utm_source=Medium&utm_medium=blog&utm_campaign=anthos)**.

![](https://cdn-images-1.medium.com/max/2000/0*xDUgpqI4LSmfGCX_)

[Here](https://medium.com/@qwiklabs/a-mesh-made-in-heaven-17fde3bd796d?utm_source=Medium&utm_medium=blog&utm_campaign=anthos) is an insight into this quest. In this blog we will take a look at the second lab of the quest [Installing Open Source Istio on Kubernetes Engine](https://google.qwiklabs.com/catalog_lab/2193?utm_source=Medium&utm_medium=blog&utm_campaign=anthos).

## What is Istio?

In order to go hands on in this lab we first need to understand what is Isitio. In this lab you will get a brief introduction about what is isitio and how it is used.

![](https://cdn-images-1.medium.com/max/3200/0*nWFFwvvrrzhEuMAO)

## Open Source Istio

In this lab you will learn how to install Open Source Istio on a GKE Cluster. You will find some interesting links which you can go through in order to get a detailed information about Istio and GCP.

## Objectives

In this lab you will be covering the following objectives.

![](https://cdn-images-1.medium.com/max/2000/0*1FoWTEpHtnJmhYzr)

## Setup and requirements

The setup and requirements of this lab are similar to other labs as you will be using Google Cloud Shell for running the commands. Make sure that you are using incognito mode while performing the labs and follow all the instructions carefully for the setup of this lab.

## Install and configure a cluster with open source Istio using Helm

In this section you will find other subsections as well. Have a look at the diagram below for more clarity.

![](https://cdn-images-1.medium.com/max/2000/0*sTnvHOgpkkk780XJ)

## Create a GKE cluster

In this section you will be first setting environment variables and then create a cluster in a single zone, with 4 nodes, that can scale up to 8 nodes. The nodes are in the default VPC network. This will take a few minutes. When you have entered the commands you will be greeted with warnings which are totally safe to ignore. :)

![](https://cdn-images-1.medium.com/max/3200/0*YUvHN05XoPweAjOJ)

## Configure cluster access for kubectl

Once your clusters are created you will configure the cluster access for the kubectl for which you will be running two simple commands in the cloud shell as given in the labs. This will take a few seconds. Once you get the output as given in the lab instructions you can proceed further with the next step.

## Install open source Istio on your GKE cluster using Helm

Here are the steps which you will be following in order to install the open source Istio on your GKE Cluster

![Verify cluster and Istio installation](https://cdn-images-1.medium.com/max/2000/0*2SrNNOCvLGm-W6U_)*Verify cluster and Istio installation*

In this section you will be ensuring the following things:

* The Cluster is up and running

* Ensure that the required Kubernetes services are deployed

* Ensure the corresponding Kubernetes pods are deployed and all containers are up and running

* Verify that istioctl is working

## Deploy Bookinfo, an Istio-enabled multi-service application

In this task, you will set up the Bookinfo sample microservices application and explore the app.

Bookinfo is a sample application which is managed by Istio. You will get an overview of this application in order to understand the same in the lab instructions.

In order to deploy the Bookinfo app you will just need to run the commands which are given in the lab instructions. Once that is done you will have to enable external access using an Istio Ingress Gateway. In order to do that you will have to run two simple commands.

After this you will verify the deployments of the app. In order to do this you will go through seven steps as given in the lab instructions. After that you will be using the application.

## Use the Bookinfo application

In order to use the application, you will need to enter this address into your browser [**http://[$GATEWAY_URL]/productpage**.](http://[$GATEWAY_URL]/productpage.) Replace **[$GATEWAY_URL]** with the working external IP address. Please refer to the images below:

![](https://cdn-images-1.medium.com/max/3200/0*igirazKJtaC0oH7G)

![](https://cdn-images-1.medium.com/max/3200/0*QIsKEybuw51lS6nc)

You will see three different versions of reviews when you refresh every time. Switching among the three is normal Kubernetes routing/balancing behavior. Here are three options which you will see.

![](https://cdn-images-1.medium.com/max/3200/0*bvNHDkIDsXFLuIjr)

![](https://cdn-images-1.medium.com/max/3200/0*XARxYXPfP-t5QYkd)

![](https://cdn-images-1.medium.com/max/3200/0*BW7gkSvVQGJv_4gb)

## Explore Stackdriver Kubernetes Engine Monitoring

This is the final section of this lab in which you will explore the stackdriver kubernetes engine monitoring according to the instructions given in the lab. Once that is done you have completed this lab successfully!!

We hope you learned something useful from this lab. Enter code **1q-istio-111** [**here](https://www.qwiklabs.com/quests/100?utm_source=Medium&utm_medium=blog&utm_campaign=istio) **for 3 free credits on your learning journey (valid through December 7th).

If you need help with the first lab of this quest which is[ **Installing the Istio on GKE Add-On with Kubernetes Engine](https://google.qwiklabs.com/catalog_lab/2194?utm_source=Medium&utm_medium=blog&utm_campaign=anthos) **head** [here](https://www.youtube.com/watch?v=uxNdMz-wDJw&list=PLYx9Jx5nax9vfx1dyAp2oLdyaJmXs4cQB&utm_source=Medium&utm_medium=blog&utm_campaign=anthos)** to do it with our experts in our on-going web series QwikQuest with Anthos. [**Managing Traffic Routing with Istio and Envoy](https://google.qwiklabs.com/catalog_lab/2196?utm_source=Medium&utm_medium=blog&utm_campaign=anthos) **has been made easier for you with this episode [**here](https://www.youtube.com/watch?v=hmu_2MM2yaI&utm_source=Medium&utm_medium=blog&utm_campaign=anthos).**

Stay tuned with our** [Qwiklabs Youtube Channel** ](https://www.youtube.com/channel/UCgadTofKslPYREQE8TjY7AA?utm_source=Medium&utm_medium=blog&utm_campaign=anthos)in order to go through the lab [**Managing Policies and Security with Istio and Citadel](https://google.qwiklabs.com/catalog_lab/2195?utm_source=Medium&utm_medium=blog&utm_campaign=anthos)**. We also have an **exclusive offer** for our users who will be joining us for the livestream this Monday, **9th December 11:00 am IST!**
