
# Building a Microservices Platform with Confluent Cloud, MongoDB Atlas, Istio, and Google…

Leading SaaS providers have sufficiently matured the integration capabilities of their product offerings to a point where it is now reasonable for enterprises to architect multi-vendor, single- and multi-cloud Production platforms, without re-engineering existing cloud-native applications. In previous posts, we have integrated other SaaS products, including as MongoDB Atlas fully-managed MongoDB-as-a-service, ElephantSQL fully-manage PostgreSQL-as-a-service, and CloudAMQP RabbitMQ-as-a-service, into cloud-native applications on Azure, AWS, GCP, and PCF.

In this post, we will build and deploy an existing, Spring Framework, microservice-based, cloud-native API to Google Kubernetes Engine (GKE), replete with [Istio](https://cloud.google.com/istio/docs/istio-on-gke/overview) 1.0, on Google Cloud Platform (GCP). The API will rely on Confluent Cloud to provide a fully-managed, Kafka-based messaging-as-a-service (MaaS). Similarly, the API will rely on MongoDB Atlas to provide a fully-managed, MongoDB-based Database-as-a-service (DBaaS).

## Background

In a previous two-part post, [Using Eventual Consistency and Spring for Kafka to Manage a Distributed Data Model: Part 1](http://programmaticponderings.com/2018/06/17/using-eventual-consistency-%e2%80%a8and-spring-for-kafka-to-manage-a-distributed-data-model-part-1/) and [Part 2](http://programmaticponderings.com/2018/06/18/using-eventual-consistency-%e2%80%a8and-spring-for-kafka-to-manage-a-distributed-data-model-part-2/), we examined the role of [Apache Kafka](https://kafka.apache.org/uses) in an event-driven, [eventually consistent](https://en.wikipedia.org/wiki/Eventual_consistency), distributed system architecture. The system, an online storefront RESTful API simulation, was composed of multiple, Java Spring Boot microservices, each with their own MongoDB database. The microservices used a [publish/subscribe model](https://kafka.apache.org/documentation/#producerapi) to communicate with each other using Kafka-based messaging. The Spring services were built using the [Spring for Apache Kafka](https://spring.io/projects/spring-kafka) and [Spring Data MongoDB](https://projects.spring.io/spring-data-mongodb/) projects.

Given the use case of placing an order through the Storefront API, we examined the interactions of three microservices, the Accounts, Fulfillment, and Orders service. We examined how the three services used Kafka to communicate state changes to each other, in a fully-decoupled manner.

The Storefront API’s microservices were managed behind an API Gateway, [Netflix’s Zuul](https://github.com/Netflix/zuul/wiki). Service discovery and load balancing were handled by [Netflix’s Eureka](https://github.com/Netflix/eureka/wiki/Eureka-at-a-glance). Both Zuul and Eureka are part of the [Spring Cloud Netflix](https://cloud.spring.io/spring-cloud-netflix/single/spring-cloud-netflix.html) project. In that post, the entire containerized system was deployed to Docker Swarm.

![](https://cdn-images-1.medium.com/max/2000/0*i6yaPIcXmNU5tJWy)

Developing the services, not operationalizing the platform, was the primary objective of the previous post.

## Featured Technologies

The following technologies are featured prominently in this post.

### Confluent Cloud

![](https://cdn-images-1.medium.com/max/2000/0*XgclxHDuV4Pbmi7Y.png)

In May 2018, Google [announced a partnership with Confluence](https://cloud.google.com/blog/products/gcp/google-cloud-platform-and-confluent-partner-to-deliver-a-managed-apache-kafka-service) to provide Confluent Cloud on GCP, a managed Apache Kafka solution for the Google Cloud Platform. Confluent, founded by the creators of [Kafka](https://kafka.apache.org), Jay Kreps, Neha Narkhede, and Jun Rao, is known for their commercial, Kafka-based streaming platform for the Enterprise.

Confluent Cloud is a fully-managed, cloud-based streaming service based on Apache Kafka. Confluent Cloud delivers a low-latency, resilient, scalable streaming service, deployable in minutes. Confluent deploys, upgrades, and maintains your Kafka clusters. Confluent Cloud is currently available on both AWS and GCP.

Confluent Cloud offers two plans, Professional and Enterprise. The Professional plan is optimized for projects under development, and for smaller organizations and applications. Professional plan [rates](https://www.confluent.io/confluent-cloud-faqs/#how-much-does-confluent-cloud-cost) for Confluent Cloud start at $0.55/hour. The Enterprise plan adds full enterprise capabilities such as service-level agreements (SLAs) with a 99.95% uptime and virtual private cloud (VPC) peering. The limitations and supported features of both plans are detailed, [here](https://docs.confluent.io/current/cloud/limits.html).

### MongoDB Atlas

![](https://cdn-images-1.medium.com/max/2000/0*9Cb36XakntCJm41d.png)

Similar to Confluent Cloud, [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) is a fully-managed MongoDB-as-a-Service, available on AWS, Azure, and GCP. Atlas, a mature SaaS product, offers high-availability, uptime SLAs, elastic scalability, cross-region replication, enterprise-grade security, LDAP integration, BI Connector, and much more.

MongoDB Atlas currently offers four [pricing plans](https://www.mongodb.com/cloud/atlas/pricing), Free, Basic, Pro, and Enterprise. Plans range from the smallest, M0-sized MongoDB cluster, with shared RAM and 512 MB storage, up to the massive M400 MongoDB cluster, with 488 GB of RAM and 3 TB of storage.

MongoDB Atlas has been featured in several past posts, including [Deploying and Configuring Istio on Google Kubernetes Engine (GKE)](https://wp.me/p1RD28-5GH) and [Developing Applications for the Cloud with Azure App Services and MongoDB Atlas](https://wp.me/p1RD28-5ij).

### Kubernetes Engine

![](https://cdn-images-1.medium.com/max/2000/0*0sl84UlyebuG4zMp.png)

According to [Google](https://cloud.google.com/kubernetes-engine/docs/concepts/kubernetes-engine-overview), Google Kubernetes Engine (GKE) provides a fully-managed, production-ready Kubernetes environment for deploying, managing, and scaling your containerized applications using Google infrastructure. GKE consists of multiple [Google Compute Engine](https://cloud.google.com/compute) instances, grouped together to form a [cluster](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-architecture).

A forerunner to other [managed Kubernetes platforms](https://blog.codeship.com/a-roundup-of-managed-kubernetes-platforms/), like EKS (AWS), AKS (Azure), PKS (Pivotal), and IBM Cloud Kubernetes Service, GKE launched publicly in 2015. GKE was built on Google’s experience of running hyper-scale services like Gmail and YouTube in containers for over 12 years.

GKE’s [pricing](https://cloud.google.com/pricing/) is based on a pay-as-you-go, per-second-billing plan, with no up-front or termination fees, similar to Confluent Cloud and MongoDB Atlas. Cluster sizes range from 1–1,000 nodes. Node machine types may be optimized for standard workloads, CPU, memory, GPU, or high-availability. Compute power ranges from 1–96 vCPUs and memory from 1–624 GB of RAM.

## Demonstration

In this post, we will deploy the three Storefront API microservices to a GKE cluster on GCP. Confluent Cloud on GCP will replace the previous Docker-based Kafka implementation. Similarly, MongoDB Atlas will replace the previous Docker-based MongoDB implementation.

![[Click to Enlarge](https://programmaticponderings.files.wordpress.com/2018/12/ConfluentCloud-v3a.png)](https://cdn-images-1.medium.com/max/2000/0*clIBIepdNxKYG332.png)*[Click to Enlarge](https://programmaticponderings.files.wordpress.com/2018/12/ConfluentCloud-v3a.png)*

Kubernetes and Istio 1.0 will replace Netflix’s Zuul and Eureka for API management, load-balancing, routing, and service discovery. Google Stackdriver will provide logging and monitoring. Docker Images for the services will be stored in Google Container Registry. Although not fully operationalized, the Storefront API will be closer to a Production-like platform, than previously demonstrated on Docker Swarm.

![[Click to Enlarge](https://programmaticponderings.files.wordpress.com/2018/12/ConfluentCloudRouting.png)](https://cdn-images-1.medium.com/max/2742/0*NJHr4qZ8_Qf4snwD.png)*[Click to Enlarge](https://programmaticponderings.files.wordpress.com/2018/12/ConfluentCloudRouting.png)*

For brevity, we will not enable standard API security features like HTTPS, OAuth for authentication, and request quotas and throttling, all of which are essential in Production. Nor, will we integrate a full lifecycle API management tool, like Google [Apigee](https://apigee.com/api-management/#/homepage).

## Source Code

The source code for this demonstration is contained in four separate GitHub repositories, [storefront-kafka-docker](https://github.com/garystafford/storefront-kafka-docker), [storefront-demo-accounts](https://github.com/garystafford/storefront-demo-accounts), [storefront-demo-orders](https://github.com/garystafford/storefront-demo-orders), and, [storefront-demo-fulfillment](https://github.com/garystafford/storefront-demo-fulfillment). However, since the Docker Images for the three storefront services are available on [Docker Hub](https://hub.docker.com/u/garystafford), it is only necessary to clone the [storefront-kafka-docker](https://github.com/garystafford/storefront-kafka-docker) project. This project contains all the code to deploy and configure the GKE cluster and Kubernetes resources ([*gist](https://gist.github.com/garystafford/b6f2d8b33be7ec9e049e00b616923218)*).

<iframe src="https://medium.com/media/625af852d1324abb0d417fec198542ec" frameborder=0></iframe>

Source code samples in this post are displayed as GitHub [Gists](https://help.github.com/articles/about-gists/), which may not display correctly on all mobile and social media browsers.

## Setup Process

The setup of the Storefront API platform is divided into a few logical steps:

1. Create the MongoDB Atlas cluster;

1. Create the Confluent Cloud Kafka cluster;

1. Create Kafka topics;

1. Modify the Kubernetes resources;

1. Modify the microservices to support Confluent Cloud configuration;

1. Create the GKE cluster with Istio on GCP;

1. Apply the Kubernetes resources to the GKE cluster;

1. Test the Storefront API, Kafka, and MongoDB are functioning properly;

## MongoDB Atlas Cluster

This post assumes you already have a MongoDB Atlas account and an existing project created. MongoDB Atlas accounts are free to set up if you do not already have one. Account creation does require the use of a Credit Card.

For minimal latency, we will be creating the MongoDB Atlas, Confluent Cloud Kafka, and GKE clusters, all on the Google Cloud Platform’s us-central1 Region. Available GCP Regions and Zones for MongoDB Atlas, Confluent Cloud, and GKE, vary, based on multiple factors.

![](https://cdn-images-1.medium.com/max/2000/0*oFQ7kSa7ykSnmP7g)

For this demo, I suggest creating a free, M0-sized MongoDB cluster. The M0-sized 3-data node cluster, with shared RAM and 512 MB of storage, and currently running MongoDB 4.0.4, is fine for individual development. The us-central1 Region is the only available US Region for the free-tier M0-cluster on GCP. An M0-sized Atlas cluster may take between 7–10 minutes to provision.

![](https://cdn-images-1.medium.com/max/2000/0*kYmElCYqQm5dgFTJ)

MongoDB Atlas’ Web-based management console provides convenient links to cluster details, metrics, alerts, and documentation.

![](https://cdn-images-1.medium.com/max/2000/0*gy6gDzSutMba8X14)

Once the cluster is ready, you can review details about the cluster and each individual cluster node.

![](https://cdn-images-1.medium.com/max/2000/0*Y9w4AD6f6rdy2AC_)

In addition to the account owner, create a demo_user account. This account will be used to authenticate and connect with the MongoDB databases from the storefront services. For this demo, we will use the same, single user account for all three services. In Production, you would most likely have individual users for each service.

![](https://cdn-images-1.medium.com/max/2000/0*f8ZwhJpmi6PLJnLJ)

Again, for security purposes, Atlas requires you to whitelist the IP address or CIDR block from which the storefront services will connect to the cluster. For now, open the access to your specific IP address using [whatsmyip.com](https://whatsmyip.com/), or much less-securely, to all IP addresses (0.0.0.0/0). Once the GKE cluster and external static IP addresses are created, make sure to come back and update this value; do not leave this wide open to the Internet.

![](https://cdn-images-1.medium.com/max/2000/0*mJsl8R2fcbpbmNxL)

The Java Spring Boot storefront services use a Spring Profile, gke. According to [Spring](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html), Spring Profiles provide a way to segregate parts of your application configuration and make it available only in certain environments. The gke Spring Profile’s configuration values may be set in a number of ways. For this demo, the majority of the values will be set using Kubernetes Deployment, ConfigMap and Secret resources, shown later.

The first two Spring configuration values will need are the MongoDB Atlas cluster’s connection string and the demo_user account password. Note these both for later use.

![](https://cdn-images-1.medium.com/max/2000/0*ruGa23ISVjjfw-9e)

## Confluent Cloud Kafka Cluster

Similar to MongoDB Atlas, this post assumes you already have a Confluent Cloud account and an existing project. It is free to set up a Professional account and a new project if you do not already have one. Atlas account creation does require the use of a Credit Card.

The Confluent Cloud web-based management console is shown below. Experienced users of other SaaS platforms may find the Confluent Cloud web-based console a bit sparse on features. In my opinion, the console lacks some necessary features, like cluster observability, individual Kafka topic management, detailed billing history (always says $0?), and persistent history of cluster activities, which survives cluster deletion. It seems like Confluent prefers users to download and configure their [Confluent Control Center](https://www.confluent.io/confluent-control-center/) to get the functionality you might normally expect from a web-based Saas management tool.

![](https://cdn-images-1.medium.com/max/2000/0*TnFZM6xmXrTlBtM_)

As explained earlier, for minimal latency, I suggest creating the MongoDB Atlas cluster, Confluent Cloud Kafka cluster, and the GKE cluster, all on the Google Cloud Platform’s us-central1 Region. For this demo, choose the smallest cluster size available on GCP, in the us-central1 Region, with 1 MB/s R/W throughput and 500 MB of storage. As shown below, the cost will be approximately $0.55/hour. Don’t forget to delete this cluster when you are done with the demonstration, or you will continue to be charged.

![](https://cdn-images-1.medium.com/max/2000/0*jo8HNTxEvM46eea7)

Cluster creation of the minimally-sized Confluent Cloud cluster is pretty quick.

![](https://cdn-images-1.medium.com/max/2000/0*PtIeGNQvYhU1VZgc)

Once the cluster is ready, Confluent provides instructions on how to interact with the cluster via the [Confluent Cloud CLI](https://docs.confluent.io/current/cloud/cli/index.html). Install the Confluent Cloud CLI, locally, for use later.

![](https://cdn-images-1.medium.com/max/2000/0*fPvb-Z5NV2-6_c_o)

As explained earlier, the Java Spring Boot storefront services use a Spring Profile, gke. Like MongoDB Atlas, the Confluent Cloud Kafka cluster configuration values will be set using Kubernetes ConfigMap and Secret resources, shown later. There are several Confluent Cloud Java configuration values shown in the Client Config Java tab; we will need these for later use.

![](https://cdn-images-1.medium.com/max/2000/0*_VpxTMFTv3dHTEiK)

### SASL and JAAS

Some users may not be familiar with the terms, SASL and JAAS. According to [Wikipedia](https://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer), Simple Authentication and Security Layer (SASL) is a framework for authentication and data security in Internet protocols. According to [Confluent](https://docs.confluent.io/current/kafka/authentication_sasl/index.html), Kafka brokers support client authentication via SASL. SASL authentication can be enabled concurrently with SSL encryption (SSL client authentication will be disabled).

There are [numerous SASL mechanisms](https://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer#SASL_mechanisms). The PLAIN SASL mechanism (SASL/PLAIN), used by Confluent, is a simple username/password authentication mechanism that is typically used with TLS for encryption to implement secure authentication. Kafka supports a default implementation for SASL/PLAIN which can be extended for production use. The SASL/PLAIN mechanism should only be used with SSL as a transport layer to ensure that clear passwords are not transmitted on the wire without encryption.

According to [Wikipedia](https://en.wikipedia.org/wiki/Java_Authentication_and_Authorization_Service), Java Authentication and Authorization Service (JAAS) is the Java implementation of the standard [Pluggable Authentication Module](https://en.wikipedia.org/wiki/Pluggable_Authentication_Module) (PAM) information security framework. According to [Confluent](https://docs.confluent.io/current/kafka/authentication_sasl/index.html), Kafka uses the JAAS for SASL configuration. You must provide JAAS configurations for all SASL authentication mechanisms.

### Cluster Authentication

Similar to MongoDB Atlas, we need to authenticate with the Confluent Cloud cluster from the storefront services. The authentication to Confluent Cloud is done with an API Key. Create a new API Key, and note the Key and Secret; these two additional pieces of configuration will be needed later.

![](https://cdn-images-1.medium.com/max/2000/0*N2p53MWfxD0aBt52)

Confluent Cloud API Keys can be created and deleted as necessary. For security in Production, API Keys should be created for each service and regularly rotated.

![](https://cdn-images-1.medium.com/max/2000/0*4uIe3ZstI5ZRVKRs)

## Kafka Topics

With the cluster created, create the storefront service’s three Kafka topics manually, using the Confluent Cloud’s ccloud CLI tool. First, configure the Confluent Cloud CLI using the ccloud init command, using your new cluster’s Bootstrap Servers address, API Key, and API Secret. The instructions are shown above Clusters Client Config tab of the Confluent Cloud web-based management interface.

![](https://cdn-images-1.medium.com/max/2000/0*2ucqeIsPgNIpQWk_)

Create the storefront service’s three Kafka topics using the ccloud topic create command. Use the list command to confirm they are created.

    # manually create kafka topics
    ccloud topic create accounts.customer.change
    ccloud topic create fulfillment.order.change
    ccloud topic create orders.order.fulfill
      
    # list kafka topics
    ccloud topic list
      
    **accounts.customer.change**
    **fulfillment.order.change**
    **orders.order.fulfill**

Another useful ccloud command, topic describe, displays topic replication details. The new topics will have a replication factor of 3 and a partition count of 12.

![](https://cdn-images-1.medium.com/max/2000/0*cYXKkmXOtIq1dX3R)

Adding the --verbose flag to the command, ccloud --verbose topic describe, displays low-level topic and cluster configuration details, as well as a log of all topic-related activities.

![](https://cdn-images-1.medium.com/max/2000/0*SoeMIKa3Ringos6S)

## Kubernetes Resources

The deployment of the three storefront microservices to the dev Namespace will minimally require the following Kubernetes configuration resources.

* (1) Kubernetes Namespace;

* (3) Kubernetes Deployments;

* (3) Kubernetes Services;

* (1) Kubernetes ConfigMap;

* (2) Kubernetes Secrets;

* (1) Istio 1.0 Gateway;

* (1) Istio 1.0 VirtualService;

* (2) Istio 1.0 ServiceEntry;

The Istio networking.istio.io v1alpha3 API [introduced](https://istio.io/blog/2018/v1alpha3-routing/#configuration-resources-in-v1alpha3) the last three configuration resources in the list, to control traffic routing into, within, and out of the mesh. There are a total of four new io networking.istio.io v1alpha3 API routing resources: [Gateway](https://istio.io/docs/reference/config/istio.networking.v1alpha3/#Gateway), [VirtualService](https://istio.io/docs/reference/config/istio.networking.v1alpha3/#VirtualService), [DestinationRule](https://istio.io/blog/2018/v1alpha3-routing/#destinationrule), and [ServiceEntry](https://istio.io/docs/reference/config/istio.networking.v1alpha3/#ServiceEntry).

Creating and managing such a large number of resources is a common complaint regarding the complexity of Kubernetes. Imagine the resource sprawl when you have dozens of microservices replicated across several namespaces. Fortunately, all resource files for this post are included in the [storefront-kafka-docker](https://github.com/garystafford/storefront-kafka-docker) project’s [gke directory](https://github.com/garystafford/storefront-kafka-docker/tree/master/gke).

To follow along with the demo, you will need to make minor modifications to a few of these resources, including the Istio Gateway, Istio VirtualService, two Istio ServiceEntry resources, and two Kubernetes Secret resources.

### Istio Gateway & VirtualService

Both the Istio Gateway and VirtualService configuration resources are contained in a single file, [istio-gateway.yaml](https://gist.github.com/garystafford/8f01fe83e1d2a139dd23ab0db6d539ea). For the demo, I am using a personal domain, storefront-demo.com, along with the sub-domain, api.dev, to host the Storefront API. The domain’s primary A record (‘@’) and sub-domain A record are both associated with the external IP address on the frontend of the load balancer. In the file, this host is configured for the Gateway and VirtualService resources. You can choose to replace the host with your own domain, or simply remove the host block altogether on lines 13–14 and 21–22. Removing the host blocks, you would then use the external IP address on the frontend of the load balancer (*explained later in the post*) to access the Storefront API ([*gist](https://gist.github.com/garystafford/8f01fe83e1d2a139dd23ab0db6d539ea)*).

<iframe src="https://medium.com/media/fc52af4451c0cd6961635dccad2c4737" frameborder=0></iframe>

### Istio ServiceEntry

There are two Istio ServiceEntry configuration resources. Both ServiceEntry resources control egress traffic from the Storefront API services, both of their ServiceEntry [Location](https://istio.io/docs/reference/config/istio.networking.v1alpha3/#ServiceEntry-Location) items are set to MESH_INTERNAL. The first ServiceEntry, [mongodb-atlas-external-mesh.yaml](https://github.com/garystafford/storefront-kafka-docker/blob/master/gke/resources/other/mongodb-atlas-external-mesh.yaml), defines MongoDB Atlas cluster egress traffic from the Storefront API ([*gist](https://gist.github.com/garystafford/f869157752a8a5b4225e2f1833c91677)*).

<iframe src="https://medium.com/media/fe9f4f954da9eb89e607e321d5564be2" frameborder=0></iframe>

The other ServiceEntry, [confluent-cloud-external-mesh.yaml](https://github.com/garystafford/storefront-kafka-docker/blob/master/gke/resources/other/confluent-cloud-external-mesh.yaml), defines Confluent Cloud Kafka cluster egress traffic from the Storefront services ([gist](https://gist.github.com/garystafford/7e9fd1a8283e2aff1c5c721bf6d9b90e)).

<iframe src="https://medium.com/media/5b4348e03679e92c9b19528c48e2068b" frameborder=0></iframe>

Both need to have their host items replaced with the appropriate Atlas and Confluent URLs.

### Inspecting Istio Resources

The easiest way to view Istio resources is from the command line using the istioctl and kubectl CLI tools.

    istioctl get gateway
    istioctl get virtualservices
    istioctl get serviceentry
      
    kubectl describe gateway
    kubectl describe virtualservices
    kubectl describe serviceentry

### Multiple Namespaces

In this demo, we are only deploying to a single Kubernetes Namespace, dev. However, Istio will also support routing traffic to multiple namespaces. For example, a typical non-prod Kubernetes cluster might support dev, test, and uat, each associated with a different sub-domain. One way to support multiple Namespaces with Istio 1.0 is to add each host to the Istio Gateway (lines 14–16, below), then create a separate Istio VirtualService for each Namespace. All the VirtualServices are associated with the single Gateway. In the VirtualService, each service’s host address is the fully qualified domain name (FQDN) of the service. Part of the FQDN is the Namespace, which we change for each for each VirtualService ([gist](https://gist.github.com/garystafford/c4b142071241fb28d1a478b37e7737bc)).

<iframe src="https://medium.com/media/a3d7c31ee2f80c38337b3ba1724d4421" frameborder=0></iframe>

### MongoDB Atlas Secret

There is one Kubernetes Secret for the sensitive MongoDB configuration and one Secret for the sensitive Confluent Cloud configuration. The [Kubernetes Secret](https://kubernetes.io/docs/concepts/configuration/secret/) object type is intended to hold sensitive information, such as passwords, OAuth tokens, and SSH keys.

The [mongodb-atlas-secret.yaml](https://github.com/garystafford/storefront-kafka-docker/blob/master/gke/resources/config/mongodb-atlas-secret.yaml) file contains the MongoDB Atlas cluster connection string, with the demo_user username and password, one for each of the storefront service’s databases ([*gist](https://gist.github.com/garystafford/a69257f33252a2afeb12d8016d1efa3d)*).

<iframe src="https://medium.com/media/8d41e76cca01047faa8fd3c08a72684a" frameborder=0></iframe>

Kubernetes Secrets are [Base64](https://en.wikipedia.org/wiki/Base64) encoded. The easiest way to encode the secret values is using the Linux base64 program. The base64 program encodes and decodes Base64 data, as specified in RFC 4648. Pass each MongoDB URI string to the base64 program using echo -n.

    MONGODB_URI=mongodb+srv://demo_user:your_password@your_cluster_address/accounts?retryWrites=true
    echo -n $MONGODB_URI | base64

    **bW9uZ29kYitzcnY6Ly9kZW1vX3VzZXI6eW91cl9wYXNzd29yZEB5b3VyX2NsdXN0ZXJfYWRkcmVzcy9hY2NvdW50cz9yZXRyeVdyaXRlcz10cnVl**

Repeat this process for the three MongoDB connection strings.

![](https://cdn-images-1.medium.com/max/2000/0*SAIsrvDaIcx7VcQJ)

### Confluent Cloud Secret

The [confluent-cloud-kafka-secret.yaml](https://github.com/garystafford/storefront-kafka-docker/blob/master/gke/resources/config/confluent-cloud-kafka-secret.yaml) file contains two data fields in the Secret’s data map, bootstrap.servers and sasl.jaas.config. These configuration items were both listed in the Client Config Java tab of the Confluent Cloud web-based management console, as shown previously. The sasl.jaas.config data field requires the Confluent Cloud cluster API Key and Secret you created earlier. Again, use the base64 encoding process for these two data fields ([*gist](https://gist.github.com/garystafford/e748df1cce841187ba6686f6fc075bb3)*).

<iframe src="https://medium.com/media/f98d965af4ad0fbcabee4740e25fd8f5" frameborder=0></iframe>

### Confluent Cloud ConfigMap

The remaining five Confluent Cloud Kafka cluster configuration values are not sensitive, and therefore, may be placed in a [Kubernetes ConfigMap](https://cloud.google.com/kubernetes-engine/docs/concepts/configmap), [confluent-cloud-kafka-configmap.yaml](https://github.com/garystafford/storefront-kafka-docker/blob/master/gke/resources/config/confluent-cloud-kafka-configmap.yaml) ([*gist](https://gist.github.com/garystafford/c2377d40d9416d5237c1dd2e6392ab1c)*).

<iframe src="https://medium.com/media/a381867dca93d14464bb9e6d09486a5b" frameborder=0></iframe>

### Accounts Deployment Resource

To see how the services consume the ConfigMap and Secret values, review the Accounts Deployment resource, shown below. Note the environment variables section, on lines 44–90, are a mix of hard-coded values and values referenced from the ConfigMap and two Secrets, shown above ([*gist](https://gist.github.com/garystafford/f811d6264c6a3bae2230e6eba472f2cf)*).

<iframe src="https://medium.com/media/b059b74ed3eee123fe5bf008845ef706" frameborder=0></iframe>

## Modify Microservices for Confluent Cloud

As explained earlier, Confluent Cloud’s Kafka cluster requires some very specific configuration, based largely on the security features of Confluent Cloud. Connecting to Confluent Cloud requires some minor modifications to the existing storefront service source code. The changes are identical for all three services. To understand the service’s code, I suggest reviewing the previous post, [Using Eventual Consistency and Spring for Kafka to Manage a Distributed Data Model: Part 1](http://programmaticponderings.com/2018/06/17/using-eventual-consistency-%e2%80%a8and-spring-for-kafka-to-manage-a-distributed-data-model-part-1/). Note the following changes are already made to the source code in the gke git branch, and not necessary for this demo.

The previous Kafka SenderConfig and ReceiverConfig Java classes have been converted to Java interfaces. There are four new SenderConfigConfluent, SenderConfigNonConfluent, ReceiverConfigConfluent, and ReceiverConfigNonConfluent classes, which implement one of the new interfaces. The new classes contain the [Spring Boot Profile](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html) class-level annotation. One set of Sender and Receiver classes are assigned the @Profile("gke") annotation, and the others, the @Profile("!gke") annotation. When the services start, one of the two class implementations are is loaded, depending on the [Active Spring Profile](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-adding-active-profiles), gke or not gke. To understand the changes better, examine the Account service’s [SenderConfigConfluent.java](https://github.com/garystafford/storefront-demo-accounts/blob/gke/src/main/java/com/storefront/config/SenderConfigConfluent.java) file ([gist](https://gist.github.com/garystafford/64cd57d96af2659dcaa3ae1846c791da)).

Line 20: Designates this class as belonging to the gke Spring Profile.

Line 23: The class now implements an interface.

Lines 25–44: Reference the Confluent Cloud Kafka cluster configuration. The values for these variables will come from the Kubernetes ConfigMap and Secret, described previously, when the services are deployed to GKE.

Lines 55–59: Additional properties that have been added to the Kafka Sender configuration properties, specifically for Confluent Cloud.

<iframe src="https://medium.com/media/7f605d7b9a4900703de743fb53276609" frameborder=0></iframe>

Once code changes were completed and tested, the Docker Image for each service was rebuilt and uploaded to [Docker Hub](https://hub.docker.com/u/garystafford) for public access. When recreating the images, the version of the Java Docker base image was upgraded from the previous post to Alpine OpenJDK 12 (openjdk:12-jdk-alpine).

## Google Kubernetes Engine (GKE) with Istio

Having created the MongoDB Atlas and Confluent Cloud clusters, built the Kubernetes and Istio resources, modified the service’s source code, and pushed the new Docker Images to Docker Hub, the GKE cluster may now be built.

For the sake of brevity, we will manually create the cluster and deploy the resources, using the Google Cloud SDK [gcloud](https://cloud.google.com/sdk/gcloud/) and Kubernetes [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) CLI tools, as opposed to automating with CI/CD tools, like Jenkins or Spinnaker. For this demonstration, I suggest a minimally-sized two-node GKE cluster using n1-standard-2 machine-type instances. The latest available [release](https://cloud.google.com/kubernetes-engine/release-notes) of Kubernetes on GKE at the time of this post was 1.11.5-gke.5 and Istio 1.03 ([Istio on GKE](https://cloud.google.com/istio/docs/istio-on-gke/overview) still considered beta). Note Kubernetes and Istio are evolving rapidly, thus the configuration flags often change with newer versions. Check the GKE Clusters tab for the latest clusters create command format ([*gist](https://gist.github.com/garystafford/dc1761eb9f6ec5d6bc464f714084a4f4)*).

<iframe src="https://medium.com/media/adaf016106d4709df938ff9a038cf622" frameborder=0></iframe>

Executing these commands successfully will build the cluster and the dev Namespace, into which all the resources will be deployed. The two-node cluster creation process takes about three minutes on average.

![](https://cdn-images-1.medium.com/max/2000/0*LbwG6t9jvfa6dPNp)

We can also observe the new GKE cluster from the GKE Clusters Details tab.

![](https://cdn-images-1.medium.com/max/2000/0*aZ5vyDD3rjHAdUpy)

Creating the GKE cluster also creates several other GCP resources, including a TCP load balancer and three external IP addresses. Shown below in the VPC network External IP addresses tab, there is one IP address associated with each of the two GKE cluster’s VM instances, and one IP address associated with the frontend of the load balancer.

![](https://cdn-images-1.medium.com/max/2000/0*U407AlkuD-7iu_mE)

While the TCP load balancer’s frontend is associated with the external IP address, the load balancer’s backend is a target pool, containing the two GKE cluster node machine instances.

![](https://cdn-images-1.medium.com/max/2000/0*Zt3V6UeqvZwPT0XL)

A forwarding rule associates the load balancer’s frontend IP address with the backend target pool. External requests to the frontend IP address will be routed to the GKE cluster. From there, requests will be routed by Kubernetes and Istio to the individual storefront service Pods, and through the Istio sidecar (Envoy) proxies. There is an Istio sidecar proxy deployed to each Storefront service Pod.

![](https://cdn-images-1.medium.com/max/2000/0*VQt7EI3XhyxuoWnD)

Below, we see the details of the load balancer’s target pool, containing the two GKE cluster’s VMs.

![](https://cdn-images-1.medium.com/max/2000/0*zwy53kVIsHXM7ZJy)

As shown at the start of the post, a simplified view of the GCP/GKE network routing looks as follows. For brevity, firewall rules and routes are not illustrated in the diagram.

![](https://cdn-images-1.medium.com/max/2000/0*FyGpRFtmV-qGpEyZ)

## Apply Kubernetes Resources

Again, using [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/), deploy the three services and associated Kubernetes and Istio resources. Note the Istio Gateway and VirtualService(s) are not deployed to the devNamespace since their role is to control ingress and route traffic to the dev Namespace and the services within it ([gist](https://gist.github.com/garystafford/742713ba6e06332d3f3acbf31d99ec95)).

<iframe src="https://medium.com/media/80be856329324a3418e7e15913ff8795" frameborder=0></iframe>

Once these commands complete successfully, on the Workloads tab, we should observe two Pods of each of the three storefront service [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) deployed to the dev Namespace, all six Pods with a Status of ‘OK’. A Deployment controller provides declarative updates for Pods and ReplicaSets.

![](https://cdn-images-1.medium.com/max/2000/0*PeuvryRBpnI_WTLE)

On the Services tab, we should observe the three storefront service’s [Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/). A Service in Kubernetes is a REST object.

![](https://cdn-images-1.medium.com/max/2000/0*xz-kbfNBPNBAkBWN)

On the Configuration Tab, we should observe the Kubernetes ConfigMap and two Secrets also deployed to the dev Environment.

![](https://cdn-images-1.medium.com/max/2000/0*4BC6V13L1ZGoryKp)

Below, we see the confluent-cloud-kafka ConfigMap resource with its data map of Confluent Cloud configuration.

![](https://cdn-images-1.medium.com/max/2000/0*UHUkx2XZkK2MnA83)

Below, we see the confluent-cloud-kafka Secret with its data map of sensitive Confluent Cloud configuration.

![](https://cdn-images-1.medium.com/max/2000/0*Zziu1t9z7ydZ6QeW)

## Test the Storefront API

If you recall from [part two](http://programmaticponderings.com/2018/06/18/using-eventual-consistency-%e2%80%a8and-spring-for-kafka-to-manage-a-distributed-data-model-part-2/) of the previous post, there are a set of seven Storefront API [endpoints](https://smartbear.com/learn/performance-monitoring/api-endpoints/) that can be called to create sample data and test the API. The HTTP GET Requests hit each service, generate test data, populate the three MongoDB databases, and produce and consume Kafka messages across all three topics. Making these requests is the easiest way to confirm the Storefront API is working properly.

1. Sample Customer: [accounts/customers/sample](http://api.dev.storefront-demo.com/accounts/customers/sample)

1. Sample Orders: [orders/customers/sample/orders](http://api.dev.storefront-demo.com/orders/customers/sample/orders)

1. Sample Fulfillment Requests: [orders/customers/sample/fulfill](http://api.dev.storefront-demo.com/orders/customers/sample/fulfill)

1. Sample Processed Order Events: [fulfillment/fulfillment/sample/process](http://api.dev.storefront-demo.com/fulfillment/fulfillment/sample/process)

1. Sample Shipped Order Events: [fulfillment/fulfillment/sample/ship](http://api.dev.storefront-demo.com/fulfillment/fulfillment/sample/ship)

1. Sample In-Transit Order Events: [fulfillment/fulfillment/sample/in-transit](http://api.dev.storefront-demo.com/fulfillment/fulfillment/sample/ship)

1. Sample Received Order Events: [fulfillment/fulfillment/sample/receive](http://api.dev.storefront-demo.com/fulfillment/fulfillment/sample/ship)

Thee are a wide variety of tools to interact with the Storefront API. The project includes a simple Python script, [sample_data.py](https://github.com/garystafford/storefront-kafka-docker/blob/master/gke/sample_data.py), which will make HTTP GET requests to each of the above endpoints, after confirming their health, and return a success message.

![](https://cdn-images-1.medium.com/max/2000/0*K-y8us7NQo1YDkc_)

### Postman

Postman, my personal favorite, is also an excellent tool to explore the Storefront API resources. I have the above set of the HTTP GET requests saved in a [Postman Collection](https://www.getpostman.com/collection). Using Postman, below, we see the response from an HTTP GET request to the /accounts/customers endpoint.

![](https://cdn-images-1.medium.com/max/2000/0*155_4OP_4-seh_4I)

Postman also allows us to create integration tests and run Collections of Requests in batches using Postman’s [Collection Runner](https://learning.getpostman.com/docs/postman/collection_runs/starting_a_collection_run/). To test the Storefront API, below, I used Collection Runner to run a single series of integration tests, intended to confirm the API’s functionality, by checking for expected HTTP response codes and expected values in the response payloads. Postman also shows the response times from the Storefront API. Since this platform was not built to meet Production SLAs, measuring response times is less critical in the Development environment.

![](https://cdn-images-1.medium.com/max/2000/0*UOVXcwL1u3Xg3ZDe)

### Google Stackdriver

If you recall, the GKE cluster had the Stackdriver Kubernetes option enabled, which gives us, amongst other observability features, access to all cluster, node, pod, and container logs. To confirm data is flowing to the MongoDB databases and Kafka topics, we can check the logs from any of the containers. Below we see the logs from the two Accounts Pod containers. Observe the AfterSaveListener handler firing on an onAfterSaveevent, which sends a CustomerChangeEvent payload to the accounts.customer.change Kafka topic, without error. These entries confirm that both Atlas and Confluent Cloud are reachable by the GKE-based workloads, and appear to be functioning properly.

![](https://cdn-images-1.medium.com/max/2000/0*SROqsNRuqAmMRykT)

### MongoDB Atlas Collection View

Review the MongoDB Atlas Clusters Collections tab. In this Development environment, the MongoDB databases and collections are created the first time a service tries to connects to them. In Production, the databases would be created and secured in advance of deploying resources. Once the sample data requests are completed successfully, you should now observe the three Storefront API databases, each with collections of documents.

![](https://cdn-images-1.medium.com/max/2000/0*QnET6eUgq_If90Ys)

### MongoDB Compass

In addition to the Atlas web-based management console, [MongoDB Compass](https://docs.mongodb.com/compass/master/#) is an excellent desktop tool to explore and manage MongoDB databases. Compass is available for Mac, Linux, and Windows. One of the many great features of Compass is the ability to visualize collection schemas and interactively filter documents. Below we see the fulfillment.requests collection schema.

![](https://cdn-images-1.medium.com/max/5200/0*yWIJI3dE5y3HAT-8.png)

### Confluent Control Center

Confluent Control Center is a downloadable, web browser-based tool for managing and monitoring Apache Kafka, including your Confluent Cloud clusters. [Confluent Control Center](https://www.confluent.io/confluent-control-center/) provides rich functionality for building and monitoring production data pipelines and streaming applications. Confluent offers a free 30-day trial of Confluent Control Center. Since the Control Center is provided at an additional fee, and I found difficult to configure for Confluent Cloud clusters based on Confluent’s documentation, I chose not to cover it in detail, for this post.

![](https://cdn-images-1.medium.com/max/2000/0*i3M_llktUFDh_A-N)

![](https://cdn-images-1.medium.com/max/2000/0*MogoKfiABGJNICpJ)

## Tear Down

Delete your Confluent Cloud and MongoDB clusters using their web-based management consoles. To delete the GKE cluster and all deployed Kubernetes resources, use the cluster delete command. Also, double-check that the external IP addresses and load balancer, associated with the cluster, were also deleted as part of the cluster deletion ([*gist](https://gist.github.com/garystafford/50c6644590e8befb82a5bdff146c7fc6)*).

<iframe src="https://medium.com/media/a157a7fadc3ccd1a76d65530cd1846bd" frameborder=0></iframe>

## Conclusion

In this post, we have seen how easy it is to integrate Cloud-based DBaaS and MaaS products with the managed Kubernetes services from GCP, AWS, and Azure. As this post demonstrated, leading SaaS providers have sufficiently matured the integration capabilities of their product offerings to a point where it is now reasonable for enterprises to architect multi-vendor, single- and multi-cloud Production platforms, without re-engineering existing cloud-native applications.

In future posts, we will revisit this Storefront API example, further demonstrating how to enable HTTPS ([Securing Your Istio Ingress Gateway with HTTPS](http://programmaticponderings.com/2019/01/03/securing-your-istio-gateway-with-https/)) and end-user authentication ([Istio End-User Authentication for Kubernetes using JSON Web Tokens (JWT) and Auth0](http://programmaticponderings.com/2019/01/06/securing-kubernetes-withistio-end-user-authentication-using-json-web-tokens-jwt/))

*Originally published on [programmaticponderings.com](https://programmaticponderings.com/2018/12/28/building-a-microservices-platform-with-confluent-cloud-mongodb-atlas-istio-and-google-kubernetes-engine/), December 2018.*

*All opinions expressed in this post are my own and not necessarily the views of my current or past employers, their clients.*
