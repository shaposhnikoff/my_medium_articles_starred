
# Istio and Kubernetes in production. Part 2. Tracing



In the previous [post](https://medium.com/avitotech/running-istio-on-kubernetes-in-production-part-i-a8bbf7fec18e), we took a look at the building blocks of Service Mesh Istio, got familiar with the system, and went through the questions that new Istio users often ask. In this post, we will look at how to organize the collection of tracing information over the network.

The first thing that developers and system administrators think about when they hear the term *Service Mesh *is tracing. Indeed, to each microservice we add a special proxy server to handle all TCP traffic. You may think that now you can easily collect information about all networking events. Unfortunately, in reality, there are many nuances that have to be kept in mind. Let’s have a look at these.

### One of the misconceptions is that we can easily get network interaction data on network traffic

In fact, what is relatively easily is only a diagram of our system’s nodes connected by arrows and the data rate between services (in fact, only bytes per unit of time). However, in most situations, the services communicate over some application layer protocol, such as HTTP, gRPC, Redis, etc. And, of course, we want to see the tracing information of these protocols, we want to see the rate of application level requests and not the rate of data. Further, we want to know the protocol’s request latency. Finally, we want to see the full path made by the request from the moment the user enters to the moment the response was received. Unfortunately, this is not an easy task.

First, let’s examine how sending tracing spans look from an architectural perspective in Istio. As I mentioned in the previous blog post, Istio has a dedicated component for collecting telemetry called Mixer. However, in the current version 1.1.*, tracing spans are sent directly from proxy servers, namely, from envoy proxies. An envoy proxy supports sending tracing spans using the zipkin protocol out of the box. Other protocols require a plugin to be connected. Istio comes with a precompiled and preconfigured envoy proxy, supporting only the zipkin protocol. If we want to use, for example, the Jaeger protocol and send tracing spans via UDP, we need to build a custom istio-proxy image. Istio-proxy does support custom plugins, however, it is still in the alpha version. Therefore, if we want to avoid multiple custom settings, the range of solutions that can be used for receiving and storing tracing spans is limited. Of the most popular protocols, either Zipkin or Jaeger can be used, but in the latter case, everything has to be uploaded using a zipkin-compatible protocol (which is much less efficient). The zipkin protocol sends all the tracing information to the collectors via the HTTP protocol, which is far from cost-effective.

As I said, we want to trace application layer protocols. This means that the proxy servers implemented next to each service should understand exactly what networking event is happening at the moment. By default, Istio comes with plain TCP configured for all ports, which means that no traces are sent. To send traces, we need to, first, enable the respective option in the main mesh config and then — just as importantly — name all the ports of the kubernetes service entities in accordance with the protocol implemented in the service. For example, like this:

    apiVersion: v1

    kind: Service

    metadata:

      name: nginx

    spec:

      ports:

      - port: 80

        targetPort: 80

        name: http

      selector:

        app: nginx

We can also use composite names, such as http-magic (Istio will see http and recognize the port as http endpoint). The format is proto-extra.

To avoid patching multiple configurations to define the protocol, a dirty workaround exists: patch the Pilot component at the moment when it is [executing the protocol determination logic](https://github.com/istio/istio/blob/75153eb2ee73c57da167be1441f9fa9f6316605b/pilot/pkg/serviceregistry/kube/conversion.go#L62). Then, of course, you will have to revert to the standard logic and switch to the naming convention of all ports.

In order to understand whether the protocol is defined correctly, enter any of the sidecar containers from the envoy proxy and make a request to the admin port of the envoy interface from location /config_dump. In the resulting configuration, check the operation field of the respective service. In Istio, it functions as an identifier of the request’s destination. To customize this parameter in Istio (we will then see it in the tracing system), specify the serviceCluster flag when launching the sidecar container. For example, it can be calculated like this from the variables received from the downward kubernetes API:

    --serviceCluster ${POD_NAMESPACE}.$(echo ${POD_NAME} | sed -e 's/-[a-z0-9]*-[a-z0-9]*$//g')

A good example helping to understand how tracing works in envoy is available [here](https://github.com/envoyproxy/envoy/tree/master/examples/zipkin-tracing).

The endpoint for sending tracing spans must also be specified in the envoy proxy launch flags, for example: —-zipkinAddress tracing-collector.tracing:9411

### Another misconception is that one can easily, out of the box extract full traces of requests in the system

Unfortunately, this is not the case. The complexity of implementation depends on how your services interact. Why so?

The problem is that, to make Istio proxy understand the match between the service’s incoming and outgoing requests, intercepting all traffic is not enough. You need some kind of match identifier. HTTP envoy proxy uses special headers, by which envoy understands exactly which request to the service generates specific requests to other services. Some of these headers are:

* x-request-id,

* x-b3-traceid,

* x-b3-spanid,

* x-b3-parentspanid,

* x-b3-sampled,

* x-b3-flags,

* x-ot-span-context.

If you have a single point, for example, a base client, where this logic can be added, all you need to do is to wait until the library has updated in all the clients. But if you are dealing with a very heterogeneous system with no unification of the service-to-service network traffic, then you most likely have a problem. Without adding this logic, all tracing information will be single-level. You will get all the service-to-service interactions, but they will not be spliced in a single chain of network traffic.

### Conclusion

Istio provides a convenient tool for collecting tracing information in the network, but its implementation requires modifications to the system taking into account Istio’s implementation peculiarities. There are two main issues to be dealt with: defining the application-layer protocol (which should be supported by the envoy proxy) and setting up forwarding of information matching incoming and outgoing requests (using headers, in the case of the HTTP protocol). When these two issues are resolved, you get a powerful tool allowing to transparently collect information from the network, even in highly heterogeneous systems coded in multiple languages and frameworks.

In the next post on Service Mesh, we will examine one of Istio’s biggest challenges — high RAM usage by each of the sidecar proxy containers — and discuss how to deal with this.
