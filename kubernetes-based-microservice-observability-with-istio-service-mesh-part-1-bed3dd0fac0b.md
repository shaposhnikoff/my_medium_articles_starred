
# Kubernetes-based Microservice Observability with Istio Service Mesh: Part 1

In this two-part post, we will explore the set of observability tools which are part of the Istio Service Mesh. These tools include Jaeger, Kiali, Prometheus, and Grafana. To assist in our exploration, we will deploy a Go-based, microservices reference platform to Google Kubernetes Engine, on the Google Cloud Platform.

![](https://cdn-images-1.medium.com/max/3022/1*yp2_sLoiC_gWHICPzMKL8A.png)

## What is Observability?

Similar to blockchain, serverless, AI and ML, chatbots, cybersecurity, and service meshes, Observability is a hot buzz word in the IT industry right now. According to Wikipedia, observability is a measure of how well internal states of a system can be inferred from knowledge of its external outputs. Logs, metrics, and traces are often known as the three pillars of observability. These are the external outputs of the system, which we may observe.

The O’Reilly book, [Distributed Systems Observability](https://www.oreilly.com/library/view/distributed-systems-observability/9781492033431/), by Cindy Sridharan, does an excellent job of detailing ‘The Three Pillars of Observability’, in [Chapter 4](https://www.oreilly.com/library/view/distributed-systems-observability/9781492033431/ch04.html). I recommend reading this free online excerpt, before continuing. A second great resource for information on observability is [honeycomb.io](https://www.honeycomb.io/), a developer of observability tools for production systems, led by well-known industry thought-leader, [Charity Majors](https://twitter.com/mipsytipsy). The honeycomb.io site includes articles, blog posts, whitepapers, and podcasts on observability.

As modern distributed systems grow ever more complex, the ability to observe those systems demands equally modern tooling that was designed with this level of complexity in mind. Traditional logging and monitoring systems often struggle with today’s hybrid and multi-cloud, polyglot language-based, event-driven, container-based and serverless, infinitely-scalable, ephemeral-compute platforms.

Tools like [Istio Service Mesh](https://istio.io/) attempt to solve the observability challenge by offering native integrations with several best-of-breed, open-source telemetry tools. Istio’s integrations include [Jaeger](https://www.jaegertracing.io/) for distributed tracing, [Kiali](https://www.kiali.io/) for distributed system visualization, and [Prometheus](https://prometheus.io/) and [Grafana](https://grafana.com/) for metric collection, monitoring, and alerting. Combined with cloud platform-native monitoring and logging services, such as [Stackdriver](https://cloud.google.com/monitoring/) for [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/) (GKE) on Google Cloud Platform (GCP), and we have a complete observability platform for modern, distributed applications.

## A Reference Microservices Platform

To demonstrate the observability tools integrated with the latest version of Istio Service Mesh, we will deploy a reference microservices platform, written in Go, to GKE on GCP. I developed the reference platform to demonstrate concepts such as API management, Service Meshes, Observability, DevOps, and [Chaos Engineering](https://principlesofchaos.org/). The platform is comprised of (14) components, including (8) [Go-based](https://golang.org/) microservices, labeled generically as Service A — Service H, (1) Angular 7, [TypeScript-based](https://en.wikipedia.org/wiki/TypeScript) front-end, (4) MongoDB databases, and (1) RabbitMQ queue for event queue-based communications. The platform and all its source code is free and open source.

The reference platform is designed to generate HTTP-based service-to-service, TCP-based service-to-database (MongoDB), and TCP-based service-to-queue-to-service (RabbitMQ) IPC (inter-process communication). Service A calls Service B and Service C, Service B calls Service D and Service E, Service D produces a message on a RabbitMQ queue that Service F consumes and writes to MongoDB, and so on. These distributed communications can be observed using Istio’s observability tools when the system is deployed to a Kubernetes cluster running the Istio service mesh.

### Service Responses

On the reference platform, each upstream service responds to requests from downstream services by returning a small informational JSON payload (termed a *greeting* in the source code).

![](https://cdn-images-1.medium.com/max/3022/1*LmfoTz9lO47X8yplGMj2ug.png)

The responses are aggregated across the service call chain, resulting in an array of service responses being returned to the edge service and on to the Angular-based UI, running in the end user’s web browser. The response aggregation feature is simply used to confirm that the service-to-service communications, Istio components, and the telemetry tools are working properly.

![](https://cdn-images-1.medium.com/max/2038/1*9zR9c2xX7wgnLTsY2XY8JA.png)

Each Go microservice contains a /ping and /health endpoint. The /health endpoint can be used to configure Kubernetes Liveness and Readiness Probes. Additionally, the edge service, Service A, is configured for [Cross-Origin Resource Sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) (CORS) using the access-control-allow-origin: * response header. CORS allows the Angular UI, running in end user’s web browser, to call the Service A /ping endpoint, which resides in a different subdomain from UI. Shown below is the Go source code for [Service A](https://github.com/garystafford/golang-srv-demo/blob/master/service-a/main.go).

<iframe src="https://medium.com/media/37cf036d5a96e045420fa37fbaaf4578" frameborder=0></iframe>

For this demonstration, the MongoDB databases will be hosted, external to the services on GCP, on [MongoDB Atlas](https://www.mongodb.com/cloud/atlas), a MongoDB-as-a-Service, cloud-based platform. Similarly, the RabbitMQ queues will be hosted on [CloudAMQP](https://www.cloudamqp.com/), a RabbitMQ-as-a-Service, cloud-based platform. I have used both of these SaaS providers in several previous posts. Using external services will help us understand how Istio and its observability tools collect telemetry for communications between the Kubernetes cluster and external systems.

Shown below is the Go source code for [Service F](https://github.com/garystafford/golang-srv-demo/blob/master/service-f/main.go), This service consumers messages from the RabbitMQ queue, placed there by [Service D](https://github.com/garystafford/golang-srv-demo/blob/master/service-D/main.go), and writes the messages to MongoDB.

<iframe src="https://medium.com/media/a92761ac3241e03e51356265ad64560a" frameborder=0></iframe>

## Source Code

All source code for this post is available on GitHub in two projects. The Go-based microservices source code, all Kubernetes resources, and all deployment scripts are located in the [k8s-istio-observe-backend](https://github.com/garystafford/k8s-istio-observe-backend) project repository. The Angular UI [TypeScript-based](https://en.wikipedia.org/wiki/TypeScript) source code is located in the [k8s-istio-observe-frontend](https://github.com/garystafford/k8s-istio-observe-frontend) project repository. You should not need to clone the Angular UI project for this demonstration.

    git clone --branch master --single-branch --depth 1 --no-tags \
    https://github.com/garystafford/k8s-istio-observe-backend.git

Docker images referenced in the Kubernetes Deployment resource files, for the Go services and UI, are all available on [Docker Hub](https://hub.docker.com/u/garystafford/). The Go microservice Docker images were built using the official [Golang Alpine](https://hub.docker.com/_/golang) base image on DockerHub, containing Go version 1.12.0. Using the Alpine image to compile the Go source code ensures the containers will be as small as possible and contain a minimal attack surface.

## System Requirements

To follow along with the post, you will need the latest version of gcloud CLI (min. ver. 239.0.0), part of the Google Cloud SDK, [Helm](https://helm.sh/), and the just releases [Istio 1.1.3](https://github.com/istio/istio/releases/tag/1.1.3) (4/15/2019) installed and configured locally or on your build machine.

![](https://cdn-images-1.medium.com/max/2000/1*e3oUDTcQbS40dccSuxE0sg.png)

## Set-up and Installation

To deploy the microservices platform to GKE, we will proceed in the following order.

1. Create the MongoDB Atlas database cluster;

1. Create the CloudAMQP RabbitMQ cluster;

1. Modify the Kubernetes resources and scripts for your own environments;

1. Create the GKE cluster on GCP;

1. Deploy Istio 1.1.3 to the GKE cluster, using Helm;

1. Create DNS records for the platform’s exposed resources;

1. Deploy the Go-based microservices, Angular UI, and associated resources to GKE;

1. Test and troubleshoot the platform;

1. Observe the results in Part Two!

## MongoDB Atlas Cluster

[MongoDB Atlas](https://www.mongodb.com/cloud/atlas) is a fully-managed MongoDB-as-a-Service, available on AWS, Azure, and GCP. Atlas, a mature SaaS product, offers high-availability, guaranteed uptime SLAs, elastic scalability, cross-region replication, enterprise-grade security, LDAP integration, a BI Connector, and much more.

MongoDB Atlas currently offers four [pricing plans](https://www.mongodb.com/cloud/atlas/pricing), Free, Basic, Pro, and Enterprise. Plans range from the smallest, M0-sized MongoDB cluster, with shared RAM and 512 MB storage, up to the massive M400 MongoDB cluster, with 488 GB of RAM and 3 TB of storage.

For this post, I have created an M2-sized MongoDB cluster in GCP’s us-central1 (Iowa) region, with a single user database account for this demo. The account will be used to connect from four of the eight Go-based microservices, running on GKE.

![](https://cdn-images-1.medium.com/max/2000/0*a4sPUkiRyQsNLcGO)

Originally, I started with an M0-sized cluster, but the compute resources were insufficient to support the volume of calls from the Go-based microservices. I suggest at least an M2-sized cluster or larger.

## CloudAMQP RabbitMQ Cluster

[CloudAMQP](https://www.cloudamqp.com/) provides full-managed RabbitMQ clusters on all major cloud and application platforms. RabbitMQ will support a decoupled, eventually consistent, message-based architecture for a portion of our Go-based microservices. For this post, I have created a RabbitMQ cluster in GCP’s us-central1 (Iowa) region, the same as our GKE cluster and MongoDB Atlas cluster. I chose a minimally-configured free version of RabbitMQ. CloudAMQP also offers robust, multi-node RabbitMQ clusters for Production use.

## Modify Configurations

There are a few configuration settings you will need to change in the GitHub project’s Kubernetes resource files and Bash deployment scripts.

### Istio ServiceEntry for MongoDB Atlas

Modify the Istio ServiceEntry, [external-mesh-mongodb-atlas.yaml](https://github.com/garystafford/golang-srv-demo/blob/master/resources/other/external-mesh-mongodb-atlas.yaml) file, adding you MongoDB Atlas host address. This file allows egress traffic from four of the microservices on GKE to the external MongoDB Atlas cluster.

    apiVersion: networking.istio.io/v1alpha3
    kind: ServiceEntry
    metadata:
      name: mongodb-atlas-external-mesh
    spec:
      hosts:
    **  - {{ your_host_goes_here }}**
      ports:
      - name: mongo
        number: 27017
        protocol: MONGO
      location: MESH_EXTERNAL
      resolution: NONE

### Istio ServiceEntry for CloudAMQP RabbitMQ

Modify the Istio ServiceEntry, [external-mesh-cloudamqp.yaml](https://github.com/garystafford/golang-srv-demo/blob/master/resources/other/external-mesh-cloudamqp.yaml) file, adding you CloudAMQP host address. This file allows egress traffic from two of the microservices to the CloudAMQP cluster.

    apiVersion: networking.istio.io/v1alpha3
    kind: ServiceEntry
    metadata:
      name: cloudamqp-external-mesh
    spec:
      hosts:
    **  - {{ your_host_goes_here }}**
      ports:
      - name: rabbitmq
        number: 5672
        protocol: TCP
      location: MESH_EXTERNAL
      resolution: NONE

### Istio Gateway and VirtualService Resources

There are numerous strategies you may use to route traffic into the GKE cluster, via Istio. I am using a single domain for the post, example-api.com, and four subdomains. One set of subdomains is for the Angular UI, in the dev Namespace (ui.dev.example-api.com) and the test Namespace (ui.test.example-api.com). The other set of subdomains is for the edge API microservice, Service A, which the UI calls (api.dev.example-api.com and api.test.example-api.com). Traffic is routed to specific Kubernetes Service resources, based on the URL.

According to [Istio](https://istio.io/docs/reference/config/istio.networking.v1alpha3/#Gateway), the Gateway describes a load balancer operating at the edge of the mesh, receiving incoming or outgoing HTTP/TCP connections. Modify the Istio ingress Gateway, inserting your own domains or subdomains in the hosts section. These are the hosts on port 80 that will be allowed into the mesh.

    apiVersion: networking.istio.io/v1alpha3
    kind: Gateway
    metadata:
      name: demo-gateway
    spec:
      selector:
        istio: ingressgateway
      servers:
      - port:
          number: 80
          name: http
          protocol: HTTP
        hosts:
    **    - ui.dev.example-api.com**
    **    - ui.test.example-api.com**
    **    - api.dev.example-api.com**
    **    - api.test.example-api.com**

According to [Istio](https://istio.io/docs/reference/config/istio.networking.v1alpha3/#VirtualService), a VirtualService defines a set of traffic routing rules to apply when a host is addressed. A VirtualService is bound to a Gateway to control the forwarding of traffic arriving at a particular host and port. Modify the project’s four Istio VirtualServices, inserting your own domains or subdomains. Here is an example of one of the four VirtualServices, in the [istio-gateway.yaml](https://github.com/garystafford/golang-srv-demo/blob/master/resources/other/istio-gateway.yaml) file.

    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: angular-ui-dev
    spec:
      hosts:
    **  - ui.dev.example-api.com**
      gateways:
      - demo-gateway
      http:
      - match:
        - uri:
            prefix: /
        route:
        - destination:
            port:
              number: 80
            host: angular-ui.dev.svc.cluster.local

### Kubernetes Secret

The project contains a Kubernetes Secret, [go-srv-demo.yaml](https://github.com/garystafford/golang-srv-demo/blob/master/resources/secrets/go-srv-demo.yaml), with two values. One is for the MongoDB Atlas connection string and one is for the CloudAMQP connections string. Remember Kubernetes Secret values need to be base64 encoded.

    apiVersion: v1
    kind: Secret
    metadata:
      name: go-srv-config
    type: Opaque
    data:
    **  mongodb.conn: {{ your_base64_encoded_secret }}**
    **  rabbitmq.conn: {{ your_base64_encoded_secret }}**

On Linux and Mac, you can use the base64 program to encode the connection strings.

    > echo -n "mongodb+srv://username:password@atlas-cluster.gcp.mongodb.net/test?retryWrites=true" | base64
    **bW9uZ29kYitzcnY6Ly91c2VybmFtZTpwYXNzd29yZEBhdGxhcy1jbHVzdGVyLmdjcC5tb25nb2RiLm5ldC90ZXN0P3JldHJ5V3JpdGVzPXRydWU=**
      
    > echo -n "amqp://username:password@rmq.cloudamqp.com/cluster" | base64
    **YW1xcDovL3VzZXJuYW1lOnBhc3N3b3JkQHJtcS5jbG91ZGFtcXAuY29tL2NsdXN0ZXI=**

### Bash Scripts Variables

The bash script, [part3_create_gke_cluster.sh, ](https://github.com/garystafford/golang-srv-demo/blob/master/part3_create_gke_cluster.sh)contains a series of environment variables. At a minimum, you will need to change the PROJECT variable in all scripts to match your GCP project name.

    # Constants - CHANGE ME!
    **readonly PROJECT='{{ your_gcp_project_goes_here }}'**
    readonly CLUSTER='go-srv-demo-cluster'
    readonly REGION='us-central1'
    readonly MASTER_AUTH_NETS='72.231.208.0/24'
    readonly GKE_VERSION='1.12.6-gke.10'
    readonly MACHINE_TYPE='n1-standard-2'

The bash script, [part4_install_istio.sh](https://github.com/garystafford/golang-srv-demo/blob/master/part4_install_istio.sh), includes the ISTIO_HOME variable. The value should correspond to your local path to Istio 1.1.3. On my local Mac, this value is shown below.

    readonly ISTIO_HOME='/Applications/istio-1.1.3'

## Deploy GKE Cluster

Next, deploy the GKE cluster using the included bash script, [part3_create_gke_cluster.sh](https://github.com/garystafford/golang-srv-demo/blob/master/part3_create_gke_cluster.sh). This will create a Regional, multi-zone, 3-node GKE cluster, using the latest version of GKE at the time of this post, 1.12.6-gke.10. The cluster will be deployed to the same region as the MongoDB Atlas and CloudAMQP clusters, GCP’s us-central1 (Iowa) region. Planning where your Cloud resources will reside, for both SaaS providers and primary Cloud providers can be critical to minimizing latency for network I/O intensive applications.

![](https://cdn-images-1.medium.com/max/2000/0*hL_LMAFipcQbMnSZ)

## Deploy Istio using Helm

With the GKE cluster and associated infrastructure in place, deploy Istio. For this post, I have chosen to [install Istio using Helm](https://istio.io/docs/setup/kubernetes/helm-install/), as recommended my Istio. To deploy Istio using Helm, use the included bash script, [part4_install_istio.sh](https://github.com/garystafford/golang-srv-demo/blob/master/part4_install_istio.sh).

![](https://cdn-images-1.medium.com/max/2000/0*mMI3Hsu70sQeKwJO)

The script installs Istio, using the Helm Chart in the local Istio 1.1.3 install/kubernetes/helm/istio directory, which you installed as a requirement for this demonstration. The Istio install script overrides several default values in the [Istio Helm Chart](https://github.com/istio/istio/tree/master/install/kubernetes/helm/istio) using the --set, flag. The list of available configuration values is detailed in the Istio Chart’s [GitHub project](https://github.com/istio/istio/tree/master/install/kubernetes/helm/istio#configuration). The options enable Istio’s observability features, which we will explore in part two. Features include Kiali, Grafana, Prometheus, and Jaeger.

    helm install ${ISTIO_HOME}/install/kubernetes/helm/istio-init \
    --name istio-init \
    --namespace istio-system
    
    helm install ${ISTIO_HOME}/install/kubernetes/helm/istio \
    --name istio \
    --namespace istio-system \
    **--set prometheus.enabled=true \
    --set grafana.enabled=true \
    --set kiali.enabled=true \
    --set tracing.enabled=true
    **
    kubectl apply --namespace istio-system \
    -f ./resources/secrets/kiali.yaml

Below, we see the Istio-related Workloads running on the cluster, including the observability tools.

![](https://cdn-images-1.medium.com/max/2000/0*7GVQwUzCF6Kjwm6C)

Below, we see the corresponding Istio-related Service resources running on the cluster.

![](https://cdn-images-1.medium.com/max/2000/0*QvCy7tl9PaWNPWKZ)

## Modify DNS Records

Instead of using IP addresses to route traffic the GKE cluster and its applications, we will use DNS. As explained earlier, I have chosen a single domain for the post, example-api.com, and four subdomains. One set of subdomains is for the Angular UI, in the dev Namespace and the test Namespace. The other set of subdomains is for the edge microservice, Service A, which the API calls. Traffic is routed to specific Kubernetes Service resources, based on the URL.

Deploying the GKE cluster and Istio triggers the creation of a [Google Load Balancer](https://cloud.google.com/load-balancing/), four IP addresses, and all required firewall rules. One of the four IP addresses, the one shown below, associated with the Forwarding rule, will be associated with the front-end of the load balancer.

![](https://cdn-images-1.medium.com/max/2000/0*J02Qf5slXXmsYKNj)

Below, we see the new load balancer, with the front-end IP address and the backend VM pool of three GKE cluster’s worker nodes. Each node is assigned one of the IP addresses, as shown above.

![](https://cdn-images-1.medium.com/max/2000/0*WrIyICmCGYACT47Y)

As shown below, using [Google Cloud DNS](https://cloud.google.com/dns/), I have created the four subdomains and assigned the IP address of the load balancer’s front-end to all four subdomains. Ingress traffic to these addresses will be routed through the Istio ingress Gateway and the four Istio VirtualServices, to the appropriate Kubernetes Service resources. Use your choice of DNS management tools to create the four A Type [DNS records](https://en.wikipedia.org/wiki/List_of_DNS_record_types).

![](https://cdn-images-1.medium.com/max/2000/0*S5LKwH9-lR-I4dhR)

## Deploy the Reference Platform

Next, deploy the eight Go-based microservices, the Angular UI, and the associated Kubernetes and Istio resources to the GKE cluster. To deploy the platform, use the included bash deploy script, [part5a_deploy_resources.sh](https://github.com/garystafford/golang-srv-demo/blob/master/part5a_deploy_resources.sh). If anything fails and you want to remove the existing resources and re-deploy, without destroying the GKE cluster or Istio, you can use the [part5b_delete_resources.sh](https://github.com/garystafford/golang-srv-demo/blob/master/part5b_delete_resources.sh) delete script.

![](https://cdn-images-1.medium.com/max/2000/0*psLb8eGAcRrdZSLw)

The deploy script deploys all the resources two Kubernetes Namespaces, dev and test. This will allow us to see how we can differentiate between Namespaces when using the observability tools.

Below, we see the Istio-related resources, which we just deployed. They include the Istio Gateway, four Istio VirtualService, and two Istio ServiceEntry resources.

![](https://cdn-images-1.medium.com/max/2000/0*DMgBQbGiaPmp_iLo)

Below, we see the platform’s Workloads (Kubernetes Deployment resources), running on the cluster. Here we see two Pods for each Workload, a total of 18 Pods, running in the dev Namespace. Each Pod contains both the deployed microservice or UI component, as well as a copy of Istio’s [Envoy Proxy](https://istio.io/docs/concepts/what-is-istio/#envoy).

![](https://cdn-images-1.medium.com/max/2000/0*XSgGrQUgWtdN5JHv)

Below, we see the corresponding Kubernetes Service resources running in the dev Namespace.

![](https://cdn-images-1.medium.com/max/2000/0*ZfIuEGF6x8RHlQZG)

Below, a similar view of the Deployment resources running in the test Namespace. Again, we have two Pods for each deployment with each Pod contains both the deployed microservice or UI component, as well as a copy of Istio’s [Envoy Proxy](https://istio.io/docs/concepts/what-is-istio/#envoy).

![](https://cdn-images-1.medium.com/max/2000/0*MokIk3hxrOOPOfrh)

## Test the Platform

We do want to ensure the platform’s eight Go-based microservices and Angular UI are working properly, communicating with each other, and communicating with the external MongoDB Atlas and CloudAMQP RabbitMQ clusters. The easiest way to test the cluster is by viewing the Angular UI in a web browser.

![](https://cdn-images-1.medium.com/max/2038/1*9zR9c2xX7wgnLTsY2XY8JA.png)

The UI requires you to input the host domain of the Service A, the API’s edge service. Since you cannot use my subdomain, and the JavaScript code is running locally to your web browser, this option allows you to provide your own host domain. This is the same domain or domains you inserted into the two Istio VirtualService for the UI. This domain route your API calls to either the FQDN (fully qualified domain name) of the Service A Kubernetes Service running in the dev namespace, service-a.dev.svc.cluster.local, or the test Namespace, service-a.test.svc.cluster.local.

![](https://cdn-images-1.medium.com/max/2000/0*EVdnqdLcqTJTAr5X)

You can also use performance testing tools to load-test the platform. Many issues will not show up until the platform is under load. I recently starting using [hey](https://github.com/rakyll/hey), a modern load generator tool, as a replacement for Apache Bench (ab), Unlike ab, hey supports HTTP/2 endpoints, which is required to test the platform on GKE with Istio. Below, I am running hey directly from [Google Cloud Shell](https://cloud.google.com/shell/). The tool is simulating 25 concurrent users, generating a total of 1,000 HTTP/2-based GET requests to Service A.

![](https://cdn-images-1.medium.com/max/2038/1*P1AfmdstW5gTuZ2SUaT8-A.png)

## Troubleshooting

If for some reason the UI fails to display, or the call from the UI to the API fails, and assuming all Kubernetes and Istio resources are running on the GKE cluster (all green), the most common explanation is usually a misconfiguration of the following resources:

1. Your four Cloud DNS records are not correct. They are not pointing to the load balancer’s front-end IP address;

1. You did not configure the four Kubernetes VirtualService resources with the correct subdomains;

1. Your Go-based microservices cannot reach the external MongoDB Atlas and CloudAMQP RabbitMQ clusters. Likely, the Kubernetes Secret is constructed incorrectly, or the two ServiceEntry resources contain the wrong host information for those external clusters;

I suggest starting the troubleshooting by calling Service A, the API’s edge service, directly, using cURL or Postman. You should see a JSON response payload, similar to the following. This suggests the issue is with the UI, not the API.

![](https://cdn-images-1.medium.com/max/2000/0*0HsCHLDeANNdM61q)

Next, confirm that the four MongoDB databases were created for Service D, Service, F, Service, G, and Service H. Also, confirm that new documents are being written to the database’s collections.

![](https://cdn-images-1.medium.com/max/2000/0*5SdIYlelirDyuF2t)

Next, confirm the new RabbitMQ queue was created, using the CloudAMQP RabbitMQ Management Console. Service D produces messages, which Service F consumes from the queue.

![](https://cdn-images-1.medium.com/max/2000/0*VT7h5wf5Y5xvhMM4)

Lastly, review the Stackdriver logs to see if there are any obvious errors.

![](https://cdn-images-1.medium.com/max/2000/0*HUvnirZp4UJ34ZF2)

## Part Two

In [part two](https://medium.com/@GaryStafford/kubernetes-based-microservice-observability-with-istio-service-mesh-part-2-f25c4b474a65) of this post, we will explore each observability tool, and see how they can help us manage our GKE cluster and the reference platform running in the cluster.

![](https://cdn-images-1.medium.com/max/2000/0*ScdRTGqo4IBUizYv)

Since the cluster only takes minutes to fully create and deploy resources to, if you want to tear down the GKE cluster, run the [part6_tear_down.sh](https://github.com/garystafford/golang-srv-demo/blob/master/part6_tear_down.sh) script.

![](https://cdn-images-1.medium.com/max/2000/0*pO_KgMOoIJE19XAT)

*All opinions expressed in this post are my own and not necessarily the views of my current or past employers or their clients.*

*Originally published at [programmaticponderings.com](http://programmaticponderings.com/2019/03/10/kubernetes-based-microservice-observability-with-istio-service-mesh-part-1/) on March 11, 2019.*
