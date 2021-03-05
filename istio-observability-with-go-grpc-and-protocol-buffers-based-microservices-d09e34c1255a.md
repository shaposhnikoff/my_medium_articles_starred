
# Istio Observability with Go, gRPC, and Protocol Buffers-based Microservices

In the last two posts, Kubernetes-based Microservice Observability with Istio Service Mesh and Azure Kubernetes Service (AKS) Observability with Istio Service Mesh, we explored the observability tools which are included with Istio Service Mesh. These tools currently include Prometheus and Grafana for metric collection, monitoring, and alerting, Jaeger for distributed tracing, and Kiali for Istio service-mesh-based microservice visualization and monitoring. Combined with cloud platform-native monitoring and logging services, such as Stackdriver on GCP, CloudWatch on AWS, Azure Monitor logs on Azure, and we have a complete observability solution for modern, distributed, Cloud-based applications.

In this post, we will examine the use of Istio’s observability tools to monitor Go-based microservices that use [Protocol Buffers](https://developers.google.com/protocol-buffers/) (aka *Protobuf*) over [gRPC](https://grpc.io) (gRPC Remote Procedure Calls) and [HTTP/2](https://en.wikipedia.org/wiki/HTTP/2) for client-server communications, as opposed to the more traditional, REST-based JSON (JavaScript Object Notation) over HTTP (Hypertext Transfer Protocol). We will see how Kubernetes, Istio, Envoy, and the observability tools work seamlessly with gRPC, just as they do with JSON over HTTP, on [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/) (GKE).

![](https://cdn-images-1.medium.com/max/2038/1*zVWt1z7biUy1Kw3V9HHNhA.png)

## Technologies

### gRPC

According to the [gRPC project](https://grpc.io), gRPC, a [CNCF](https://www.cncf.io/) incubating project, is a modern, high-performance, open-source and universal [remote procedure call](https://en.wikipedia.org/wiki/Remote_procedure_call) (RPC) framework that can run anywhere. It enables client and server applications to communicate transparently and makes it easier to build connected systems. Google, the original developer of gRPC, has used the underlying technologies and concepts in gRPC for years. The current implementation is used in several Google cloud products and Google externally facing APIs. It is also being used by Square, Netflix, CoreOS, Docker, CockroachDB, Cisco, Juniper Networks and many other organizations.

### Protocol Buffers

By default, gRPC uses Protocol Buffers. According to Google, [Protocol Buffers](https://developers.google.com/protocol-buffers/) (*aka Protobuf*) are a language- and platform-neutral, efficient, extensible, automated mechanism for serializing structured data for use in communications protocols, data storage, and more. Protocol Buffers are 3 to 10 times smaller and 20 to 100 times faster than XML. Once you have defined your messages, you run the protocol buffer compiler for your application’s language on your .proto file to generate data access classes.
> Protocol Buffers are 3 to 10 times smaller and 20 to 100 times faster than XML.

Protocol buffers currently support generated code in Java, Python, Objective-C, and C++, Dart, Go, Ruby, and C#. For this post, we have compiled for Go. You can read more about the binary wire format of Protobuf on Google’s [Developers Portal](https://developers.google.com/protocol-buffers/docs/encoding).

### Envoy Proxy

According to the [Istio project](https://istio.io/docs/concepts/what-is-istio/#envoy), Istio uses an extended version of the [Envoy](https://www.envoyproxy.io/) proxy. Envoy is deployed as a sidecar to a relevant service in the same Kubernetes pod. Envoy, created by Lyft, is a high-performance proxy developed in C++ to mediate all inbound and outbound traffic for all services in the service mesh. Istio leverages Envoy’s many built-in features, including dynamic service discovery, load balancing, TLS termination, HTTP/2 and gRPC proxies, circuit-breakers, health checks, staged rollouts, fault injection, and rich metrics.

According to the post by Harvey Tuch of Google, [Evolving a Protocol Buffer canonical API](https://blog.envoyproxy.io/evolving-a-protocol-buffer-canonical-api-e1b2c2ca0dec), Envoy proxy adopted Protocol Buffers, specifically [proto3](https://developers.google.com/protocol-buffers/docs/proto3), as the canonical specification of for version 2 of Lyft’s gRPC-first API.

## Reference Microservices Platform

In the last two posts, we explored Istio’s observability tools, using a RESTful microservices-based API platform written in Go and using JSON over HTTP for service to service communications. The API platform was comprised of eight [Go-based](https://golang.org/) microservices and one sample Angular 7, [TypeScript-based](https://en.wikipedia.org/wiki/TypeScript) front-end web client. The various services are dependent on MongoDB, and RabbitMQ for event queue-based communications. Below, the is JSON over HTTP-based platform architecture.

![](https://cdn-images-1.medium.com/max/3022/1*uSdUHacddypjAQQ3eueMmQ.png)

Below, the current Angular 7-based web client interface.

![](https://cdn-images-1.medium.com/max/2038/1*ptJ46gzI03ESsdJfMMS39A.png)

## Converting to gRPC and Protocol Buffers

For this post, I have modified the eight Go microservices to use [gRPC](https://grpc.io) and [Protocol Buffers](https://developers.google.com/protocol-buffers/), Google’s data interchange format. Specifically, the services use version 3 [release](https://github.com/protocolbuffers/protobuf/releases) (aka *proto3*) of Protocol Buffers. With gRPC, a gRPC client calls a gRPC server. Some of the platform’s services are gRPC servers, others are gRPC clients, while some act as both client and server, such as Service A, B, and E. The revised architecture is shown below.

![](https://cdn-images-1.medium.com/max/3550/1*-7udRZK9N4m_xx5sflBojQ.png)

## gRPC Gateway

Assuming for the sake of this demonstration, that most consumers of the API would still expect to communicate using a RESTful JSON over HTTP API, I have added a [gRPC Gateway](https://github.com/grpc-ecosystem/grpc-gateway) reverse proxy to the platform. The gRPC Gateway is a gRPC to JSON reverse proxy, a common architectural pattern, which proxies communications between the JSON over HTTP-based clients and the gRPC-based microservices. A diagram from the [grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway) GitHub project site effectively demonstrates how the reverse proxy works.

![](https://cdn-images-1.medium.com/max/2000/0*zQTQ1OoFDKbiraXB)

*Image courtesy: [https://github.com/grpc-ecosystem/grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway)*

In the revised platform architecture diagram above, note the addition of the reverse proxy, which replaces Service A at the edge of the API. The proxy sits between the Angular-based Web UI and Service A. Also, note the communication method between services is now Protobuf over gRPC instead of JSON over HTTP. The use of Envoy Proxy (via Istio) is unchanged, as is the MongoDB Atlas-based databases and CloudAMQP RabbitMQ-based queue, which are still external to the Kubernetes cluster.

### Alternatives to gRPC Gateway

As an alternative to the gRPC Gateway reverse proxy, we could convert the TypeScript-based Angular UI client to gRPC and Protocol Buffers, and continue to communicate directly with Service A as the edge service. However, this would limit other consumers of the API to rely on gRPC as opposed to JSON over HTTP, unless we also chose to expose two different endpoints, gRPC, and JSON over HTTP, another common pattern.

## Demonstration

In this post’s demonstration, we will repeat the same installation process, outlined in the previous post, [Kubernetes-based Microservice Observability with Istio Service Mesh](https://itnext.io/kubernetes-based-microservice-observability-with-istio-service-mesh-part-1-bed3dd0fac0b). We will deploy the revised gRPC-based platform to GKE on GCP. You could just as easily follow [Azure Kubernetes Service (AKS) Observability with Istio Service Mesh](https://itnext.io/azure-kubernetes-service-aks-observability-with-istio-service-mesh-4eb28da0f764), and deploy the platform to AKS.

### Source Code

All source code for this post is available on GitHub, contained in three projects. The Go-based microservices source code, all Kubernetes resources, and all deployment scripts are located in the [k8s-istio-observe-backend](https://github.com/garystafford/k8s-istio-observe-backend) project repository, in the new grpc branch.

    git clone \ 
      **--branch grpc** --single-branch \
      --depth 1 --no-tags \ 
      [https://github.com/garystafford/k8s-istio-observe-backend.git](https://github.com/garystafford/k8s-istio-observe-backend.git)

The Angular-based web client source code is located in the [k8s-istio-observe-frontend](https://github.com/garystafford/k8s-istio-observe-frontend) repository on the new grpc branch. The source protocol buffers .proto file and the generated code, using the protocol buffers compiler, is located in the new [pb-greeting](https://github.com/garystafford/pb-greeting) project repository. You do not need to clone either of these projects for this post’s demonstration.

All Docker images for the services, UI, and the reverse proxy are located on [Docker Hub](https://hub.docker.com/search?q=%22garystafford&type=image&sort=updated_at&order=desc).

## Code Changes

This post is not specifically about writing Go for gRPC and Protobuf. However, to better understand the observability requirements and capabilities of these technologies, compared to JSON over HTTP, it is helpful to review some of the source code.

### Microservices

First, compare the source code for [Service A](https://github.com/garystafford/k8s-istio-observe-backend/blob/grpc/services/service-a/main.go), shown below, to the [original code](https://github.com/garystafford/k8s-istio-observe-backend/blob/master/services/service-a/main.go) in the previous post. The service’s code is almost completely re-written. I relied on several references for writing the code, including, [Tracing gRPC with Istio](https://aspenmesh.io/2018/04/tracing-grpc-with-istio/), written by Neeraj Poddar of [Aspen Mesh](https://aspenmesh.io/) and [Distributed Tracing Infrastructure with Jaeger on Kubernetes](https://medium.com/@masroor.hasan/tracing-infrastructure-with-jaeger-on-kubernetes-6800132a677), by Masroor Hasan.

Specifically, note the following code changes to Service A:

1. Import of the [pb-greeting](https://github.com/garystafford/pb-greeting) protobuf package;

1. Local Greeting struct replaced with pb.Greeting struct;

1. All services are now hosted on port 50051;

1. The HTTP server and all API resource handler functions are removed;

1. Headers, used for distributed tracing with Jaeger, have moved from HTTP request object to metadata passed in the gRPC context object;

1. Service A is coded as a gRPC server, which is called by the gRPC Gateway reverse proxy (gRPC client) via the Greeting function;

1. The primary PingHandler function, which returns the service's Greeting, is replaced by the [pb-greeting](https://github.com/garystafford/pb-greeting) protobuf package's Greeting function;

1. Service A is coded as a gRPC client, calling both Service B and Service C using the CallGrpcService function;

1. CORS handling is offloaded to Istio;

1. Logging methods are unchanged;

Source code for revised gRPC-based [Service A](https://github.com/garystafford/k8s-istio-observe-backend/blob/grpc/services/service-a/main.go):

<iframe src="https://medium.com/media/ff8e897db96aa47a86f32d09cc957ecf" frameborder=0></iframe>

### Greeting Protocol Buffers

Shown below is the greeting source protocol buffers .proto file. The greeting response struct, originally defined in the services, remains largely unchanged. The UI client responses will look identical.

<iframe src="https://medium.com/media/d153d7e89ac01c6a20a67a910f65ede4" frameborder=0></iframe>

When compiled with protoc, the Go-based protocol compiler plugin, the original 27 lines of source code swells to almost 270 lines of generated data access classes that are easier to use programmatically.

<iframe src="https://medium.com/media/fd2f7eb7ce6dfbe225c996a13803cb4a" frameborder=0></iframe>

Below is a small snippet of that compiled code, for reference. The compiled code is included in the [pb-greeting](https://github.com/garystafford/pb-greeting) project on GitHub and imported into each microservice and the reverse proxy. We also compile a separate version for the reverse proxy to implement.

<iframe src="https://medium.com/media/f86683cdebf8d3dfd3df565ecbda0805" frameborder=0></iframe>

Using Swagger, we can view the greeting protocol buffers’ single RESTful API resource, exposed with an HTTP GET method. I use the Docker-based version of [Swagger UI](https://hub.docker.com/r/swaggerapi/swagger-ui/) for viewing protoc generated swagger definitions.

    docker run -p 8080:8080 -d --name swagger-ui \
      -e SWAGGER_JSON=/tmp/greeting.swagger.json \
      -v ${GOAPTH}/src/pb-greeting:/tmp swaggerapi/swagger-ui

The Angular UI makes an HTTP GET request to the /api/v1/greeting resource, which is transformed to gRPC and proxied to Service A, where it is handled by the Greeting function.

![](https://cdn-images-1.medium.com/max/2038/1*xEgXMNrjYUJD2G_9y0SwLQ.png)

### gRPC Gateway Reverse Proxy

As explained earlier, the [gRPC Gateway](https://github.com/grpc-ecosystem/grpc-gateway) reverse proxy service is completely new. Specifically, note the following code features in the gist below:

1. Import of the [pb-greeting](https://github.com/garystafford/pb-greeting) protobuf package;

1. The proxy is hosted on port 80;

1. Request headers, used for distributed tracing with Jaeger, are collected from the incoming HTTP request and passed to Service A in the gRPC context;

1. The proxy is coded as a gRPC client, which calls Service A;

1. Logging is largely unchanged;

The source code for the [Reverse Proxy](https://github.com/garystafford/k8s-istio-observe-backend/blob/grpc/services/service-rev-proxy/main.go):

<iframe src="https://medium.com/media/7750f45f0f154c9761d8b5dd672b076f" frameborder=0></iframe>

### Header Propagation

Below, in the Stackdriver logs, we see an example of a set of HTTP request headers in the JSON payload, which are propagated upstream to gRPC-based Go services from the gRPC Gateway’s reverse proxy. Header propagation ensures the request produces a complete distributed trace across the complete service call chain.

![](https://cdn-images-1.medium.com/max/2038/1*byW-K9PR0OcZDetVSduMYg.png)

## Istio VirtualService and CORS

According to feedback in the project’s [GitHub Issues](https://github.com/grpc/grpc-web/issues/435#issuecomment-454113721), the gRPC Gateway does not directly support Cross-Origin Resource Sharing (CORS) policy. In my own experience, the gRPC Gateway cannot handle OPTIONS HTTP method requests, which must be issued by the Angular 7 web UI. Therefore, I have offloaded CORS responsibility to Istio, using the VirtualService resource’s [CorsPolicy](https://istio.io/docs/reference/config/networking/v1alpha3/virtual-service/#CorsPolicy) configuration. This makes CORS much easier to manage than coding CORS configuration into service code.

<iframe src="https://medium.com/media/f5513ba5a8bc1fa0ca65ff7fcc24989f" frameborder=0></iframe>

## Set-up and Installation

To deploy the microservices platform to GKE, follow the detailed instructions in part one of the post, [Kubernetes-based Microservice Observability with Istio Service Mesh: Part 1](https://itnext.io/kubernetes-based-microservice-observability-with-istio-service-mesh-part-1-bed3dd0fac0b), or [Azure Kubernetes Service (AKS) Observability with Istio Service Mesh](https://itnext.io/azure-kubernetes-service-aks-observability-with-istio-service-mesh-4eb28da0f764) for AKS. You should also review the README document for this project, on GitHub.

1. Create the external MongoDB Atlas database and CloudAMQP RabbitMQ clusters;

1. Modify the Kubernetes resource files and bash scripts for your own environments;

1. Create the managed GKE or AKS cluster on GCP or Azure;

1. Configure and deploy Istio to the managed Kubernetes cluster, using Helm;

1. Create DNS records for the platform’s exposed resources;

1. Deploy the Go-based microservices, gRPC Gateway reverse proxy, Angular UI, and associated resources to Kubernetes cluster;

1. Test and troubleshoot the platform deployment;

1. Observe the results;

## The Three Pillars

As introduced in the first post, logs, metrics, and traces are often known as the three pillars of observability. These are the external outputs of the system, which we may observe. As modern distributed systems grow ever more complex, the ability to observe those systems demands equally modern tooling that was designed with this level of complexity in mind. Traditional logging and monitoring systems often struggle with today’s hybrid and multi-cloud, polyglot language-based, event-driven, container-based and serverless, infinitely-scalable, ephemeral-compute platforms.

Tools like [Istio Service Mesh](https://istio.io/) attempt to solve the observability challenge by offering native integrations with several best-of-breed, open-source telemetry tools. Istio’s integrations include [Jaeger](https://www.jaegertracing.io/) for distributed tracing, [Kiali](https://www.kiali.io/) for Istio service mesh-based microservice visualization and monitoring, and [Prometheus](https://prometheus.io/) and [Grafana](https://grafana.com/) for metric collection, monitoring, and alerting. Combined with cloud platform-native monitoring and logging services, such as [Stackdriver](https://cloud.google.com/monitoring/) for GKE, [CloudWatch](https://aws.amazon.com/cloudwatch/) for Amazon’s EKS, or [Azure Monitor](https://docs.microsoft.com/en-us/azure/azure-monitor/overview) logs for AKS, and we have a complete observability solution for modern, distributed, Cloud-based applications.

## Pillar 1: Logging

Moving from JSON over HTTP to gRPC does not require any changes to the logging configuration of the Go-based service code or Kubernetes resources.

### Stackdriver with Logrus

As detailed in part two of the last post, [Kubernetes-based Microservice Observability with Istio Service Mesh](https://itnext.io/kubernetes-based-microservice-observability-with-istio-service-mesh-part-1-bed3dd0fac0b), our logging strategy for the eight Go-based microservices and the reverse proxy continues to be the use of [Logrus](https://github.com/sirupsen/logrus), the popular structured logger for Go, and Banzai Cloud’s [logrus-runtime-formatter](https://github.com/sirupsen/logrus).

If you recall, the Banzai formatter automatically tags log messages with runtime/stack information, including function name and line number; extremely helpful when troubleshooting. We are also using Logrus’ JSON formatter. Below, in the Stackdriver console, note how each log entry below has the JSON payload contained within the message with the log level, function name, lines on which the log entry originated, and the message.

![](https://cdn-images-1.medium.com/max/2038/1*JipAO8ISQVr3NbS1iE06vQ.png)

Below, we see the details of a specific log entry’s JSON payload. In this case, we can see the request headers propagated from the downstream service.

![](https://cdn-images-1.medium.com/max/2038/1*byW-K9PR0OcZDetVSduMYg.png)

## Pillar 2: Metrics

Moving from JSON over HTTP to gRPC does not require any changes to the metrics configuration of the Go-based service code or Kubernetes resources.

### Prometheus

[Prometheus](https://prometheus.io/) is a completely open source and community-driven systems monitoring and alerting toolkit originally built at SoundCloud, circa 2012. Interestingly, Prometheus joined the [Cloud Native Computing Foundation](https://cncf.io/) (CNCF) in 2016 as the second hosted-project, after [Kubernetes](http://kubernetes.io/).

![](https://cdn-images-1.medium.com/max/2038/1*0iLxeXV_kVv6sXKlL6967w.png)

### Grafana

Grafana describes itself as the leading open source software for time series analytics. According to [Grafana Labs,](https://grafana.com/grafana) Grafana allows you to query, visualize, alert on, and understand your metrics no matter where they are stored. You can easily create, explore, and share visually-rich, data-driven dashboards. Grafana allows users to visually define alert rules for your most important metrics. Grafana will continuously evaluate rules and can send notifications.

According to [Istio](https://istio.io/docs/tasks/telemetry/using-istio-dashboard/#about-the-grafana-add-on), the Grafana add-on is a pre-configured instance of Grafana. The Grafana Docker base image has been modified to start with both a Prometheus data source and the Istio Dashboard installed. Below, we see two of the pre-configured dashboards, the Istio Mesh Dashboard and the Istio Performance Dashboard.

![](https://cdn-images-1.medium.com/max/2038/1*TS3f4dMj8oZQ3mC1z524vw.png)

![](https://cdn-images-1.medium.com/max/2038/1*g_w5CM4Ek9x4aBu2qucljQ.png)

## Pillar 3: Traces

Moving from JSON over HTTP to gRPC did require a complete re-write of the tracing logic in the service code. In fact, I spent the majority of my time ensuring the correct headers were propagated from the Istio Ingress Gateway to the gRPC Gateway reverse proxy, to Service A in the gRPC context, and upstream to all the dependent, gRPC-based services. I am sure there are a number of optimization in my current code, regarding the correct handling of traces and how this information is propagated across the service call stack.

### Jaeger

According to their website, [Jaeger](https://www.jaegertracing.io/docs/1.10/), inspired by [Dapper](https://research.google.com/pubs/pub36356.html) and [OpenZipkin](http://zipkin.io/), is a distributed tracing system released as open source by [Uber Technologies](http://uber.github.io/). It is used for monitoring and troubleshooting microservices-based distributed systems, including distributed context propagation, distributed transaction monitoring, root cause analysis, service dependency analysis, and performance and latency optimization. The Jaeger [website](https://www.jaegertracing.io/docs/1.10/architecture/) contains an excellent overview of Jaeger’s architecture and general tracing-related terminology.

Below we see the Jaeger UI Traces View. In it, we see a series of traces generated by [hey](https://github.com/rakyll/hey), a modern load generator and benchmarking tool, and a worthy replacement for Apache Bench ( ab). Unlike ab, hey supports HTTP/2. The use of hey was detailed in the previous post.

![](https://cdn-images-1.medium.com/max/2038/1*jqasEdoWGOWwexaMwqeXAQ.png)

A trace, as you might recall, is an execution path through the system and can be thought of as a [directed acyclic graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph) (DAG) of [spans](https://www.jaegertracing.io/docs/1.10/architecture#span). If you have worked with systems like Apache Spark, you are probably already familiar with DAGs.

![](https://cdn-images-1.medium.com/max/2038/1*8HuTbDyhgf6GdVlz3dHrsg.png)

Below we see the Jaeger UI Trace Detail View. The example trace contains 16 spans, which encompasses nine components — seven of the eight Go-based services, the reverse proxy, and the Istio Ingress Gateway. The trace and the spans each have timings. The root span in the trace is the Istio Ingress Gateway. In this demo, traces do not span the RabbitMQ message queues. This means you would not see a trace which includes the decoupled, message-based communications between Service D to Service F, via the RabbitMQ.

![](https://cdn-images-1.medium.com/max/2038/1*Hp5M-ey-QnUWgZRjq3lEWg.png)

Within the Jaeger UI Trace Detail View, you also have the ability to drill into a single span, which contains additional metadata. Metadata includes the URL being called, HTTP method, response status, and several other headers.

![](https://cdn-images-1.medium.com/max/2038/1*2c70_jiS0OwS2gQ2WtTcSg.png)

## Microservice Observability

Moving from JSON over HTTP to gRPC does not require any changes to the Kiali configuration of the Go-based service code or Kubernetes resources.

### Kiali

According to their [website](https://www.kiali.io/documentation/overview/), Kiali provides answers to the questions: What are the microservices in my Istio service mesh, and how are they connected? Kiali works with Istio, in OpenShift or Kubernetes, to visualize the service mesh topology, to provide visibility into features like circuit breakers, request rates and more. It offers insights about the mesh components at different levels, from abstract Applications to Services and Workloads.

The Graph View in the Kiali UI is a visual representation of the components running in the Istio service mesh. Below, filtering on the cluster’s dev Namespace, we should observe that Kiali has mapped all components in the platform, along with rich metadata, such as their version and communication protocols.

![](https://cdn-images-1.medium.com/max/2038/1*zVWt1z7biUy1Kw3V9HHNhA.png)

Using Kiali, we can confirm our service-to-service IPC protocol is now gRPC instead of the previous HTTP.

![](https://cdn-images-1.medium.com/max/2038/1*zZsX_4RjlzFePsC-d-86rA.png)

Although converting from JSON over HTTP to protocol buffers with gRPC required major code changes to the services, it did not impact the high-level observability we have of those services using the tools provided by Istio, including Prometheus, Grafana, Jaeger, and Kiali.

*All opinions expressed in this post are my own and not necessarily the views of my current or past employers or their clients.*

*Originally published at [http://programmaticponderings.com](https://programmaticponderings.com/2019/04/17/istio-observability-with-go-grpc-and-protocol-buffers-based-microservices/) on April 17, 2019.*
