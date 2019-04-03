
# Running Istio on Kubernetes in production. Part I.



What is [Istio](https://istio.io)? Istio is a service mesh technology adding an abstraction layer to the network. It intercepts all or part of the traffic in a k8s cluster and executes a set of operations on it. Which operations are supported? For example, setting up smart routing or implementing a circuit breaker approach, setting up “canary deployment”. Moreover, Istio makes possible imposing a limit on external interactions and controlling all routes between the cluster and an external network. Furthermore, it supports setting up policy rules for controlling campaigns between different microservices. Finally, we can generate an entire network interactions map and make a unified collection of metrics completely transparent to applications.

A detailed description of Istio can be found in the [official documentation](https://istio.io/docs/concepts/). Here, I’d like to present the basic principles behind microservices interaction based on Istio and I will try to show that Istio is a really powerful tool for solving various problems. In this post, I’ll try to answer the questions that beginner users of Istio typically have. These are the things that will allow you to use Istio more efficiently.

But before installation, I want to present some central concepts and take a glance at Istio’s components and principle of interaction between those.

**Operating principle**

Istio is composed of two main components — control plane and data plane. Control plane contains the basic components ensuring that correct interaction between the other components. In the current version 1.0, the control plane has three main components: Pilot, Mixer, Citadel. Citadel won’t be discussed here as it’s needed to generate certificates for mutual TLS between services. Let’s have a look at the design and purpose of Pilot and Mixer.

![](https://cdn-images-1.medium.com/max/2596/1*Qz-ejoBJsCHU7Xnxunn50w.png)

Pilot is the main control component that distributes all information about what we keep inside the cluster — services, their endpoints and routing rules (for example, the rules of canary deployment or the circuit breaker rules).

Mixer is an optional control plane component that provides the ability to collect metrics, logs, and any information about network interactions. It also monitors compliance with policy rules and compliance with the rate limits.

The data plane component is implemented using sidecar proxy containers. By default, a powerful [proxy server envoy](https://www.envoyproxy.io) is used. To ensure Istio’s completely transparent for applications, there is an automatic injection system. The latest implementation supports kubernetes versions 1.9 and newer (mutational admission webhook). For kubernetes versions 1.7, 1.8, you can use the Initializer.

Sidecar containers connect to the Pilot via the GRPC protocol optimizing the pushdown model of changes inside the cluster. GRPC has been used in Envoy since version 1.6; in Istio it has been implemented since version 0.8 and is a pilot-agent — a wrapper on Go over envoy that configures startup parameters.

Pilot and Mixer are completely stateless components, all states are kept in the apps’ memory. Their configurations are specified in kubernetes Custom Resources stored in etcd. Istio-agent gets the Pilot address and opens the GRPC stream to it.

As I said, Istio implements all the functionality entirely transparent for the applications. Let’s see how. The algorithm works as follows:

1. We deploy a new version of the service.

1. Depending on the sidecar container injection type, an istio-init container and istio-agent container (envoy) are added during the configuration phase, or they can be manually inserted into the pod description of the kubernetes entity.

1. The istio-init container is a script that applies the iptables rules for a pod. There are two ways to configure traffic redirecting to an istio-agent container: using redirect iptables rules or [TPROXY](https://github.com/kristrev/tproxy-example/blob/master/tproxy_example.c). At the writing moment, the default is using redirect rules. In istio-init, it is possible to configure which traffic will be intercepted and sent to istio-agent. For example, in order to intercept all incoming and all outgoing traffic, you need to set the parameters -iand -bto *. You can specify specific ports to be intercepted. To avoid intercepting a certain subnet, you can specify it using the -xflag.

1. After init execution, the containers, including pilot-agent (envoy), are launched. It connects to the deployed Pilot by GRPC and gets information about all existing services and routing policies in the cluster. According to the received data, it configures the cluster and maps these directly to the application endpoints in the k8s cluster. There is an important moment: envoy dynamically configures listeners (IP, port pairs) that start listening. Therefore, when requests enter the pod and are redirected using iptables rules to sidecar, envoy is prepared to handle these connections and understands where to forward the proxy traffic. In this step, the information is sent to the Mixer, which we will be described below.

In the end, we get a whole network of envoy proxy servers, which can be configured from one point (Pilot). As a result, all inbound and outbound requests pass through an envoy. Moreover, only TCP traffic is intercepted. This means that the kubernetes service IP is resolved using kube-dns over UDP without modification. Then, after the resolver, an outbound request is intercepted and processed by envoy, which determines to which endpoint the request should be sent (or not, in case of access policies or triggering a circuit breaker algorithm).

Now that we are familiar with Pilot figured out, we will look at how Mixer works and why we need it. The official documentation on Mixer is available [here](https://istio.io/docs/concepts/policies-and-telemetry/overview/).

Mixer has two components: istio-telemetry, istio-policy (up to version 0.8, it used to be a single component istio-mixer). Both are a mixer. Istio telemetry receives GRPC from sidecar containers and reports information about service interactions and parameters. Istio-policy accepts check requests to verify compliance with Policy rules. These policy checks are cached on the client (in a sidecar) for a certain time. Report checks are sent in batch requests. We will look at how to configure it and which parameters need to be set a bit later.

Mixer is supposed to be a highly-available component providing uninterrupted assembly and processing of telemetry data. The system is a multi-level buffer. Initially, the data is buffered on the sidecar side of the containers, then on mixer side, and finally sent to so-called mixer backends. As a result, if any of system components fails, the buffer grows and, when the system has been restored, is flushed. Mixer backends are endpoints for sending telemetry data: statsd, newrelic and so on. Writing custom backends is easy, and later I’ll show how.

![](https://cdn-images-1.medium.com/max/2836/1*SrTQFpl1TU4rEfojcZzFxQ.png)

To sum up, the workflow of using istio-telemetry is as follows:

1. Service 1 sends a request to service 2.

1. On exiting Service 1, the request is redirected in its sidecar.

1. Sidecar envoy monitors the request for service 2 and prepares the necessary information.

1. Then it sends it to istio-telemetry using the Report request.

1. Istio-telemetry determines whether to send this Report to backends, where to send request and request content.

Now let’s see how to set up the Istio system with the two basic components Pilot and sidecar envoy. Let’s review the basic configuration (mesh) that Pilot reads:

    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: istio
      namespace: istio-system
      labels:
        app: istio
        service: istio
    data:
      mesh: |-

    # disable tracing mechanism for now
        enableTracing: false

    # do not specify mixer endpoints, so that sidecar containers do not send the information
        #mixerCheckServer: istio-policy.istio-system:15004
        #mixerReportServer: istio-telemetry.istio-system:15004

    # interval for envoy to check Pilot
        rdsRefreshDelay: 5s

    # default config for envoy sidecar
        defaultConfig:
          # like rdsRefreshDelay
          discoveryRefreshDelay: 5s

    # path to envoy executable
          configPath: "/etc/istio/proxy"
          binaryPath: "/usr/local/bin/envoy"

    # default name for sidecar container
          serviceCluster: istio-proxy

    # time for envoy to wait before it shuts down all existing connections
          drainDuration: 45s
          parentShutdownDuration: 1m0s

    # by default, REDIRECT rule for iptables is used. TPROXY can be used as well.
          #interceptionMode: REDIRECT

    # port for sidecar container admin panel
          proxyAdminPort: 15000

    # address for sending traces using zipkin protocol (not used as turned off in enableTracing option)
          zipkinAddress: tracing-collector.tracing:9411

    # statsd address for envoy containers metrics
          # statsdUdpAddress: aggregator:8126

    # turn off Mutual TLS
          controlPlaneAuthPolicy: NONE

    # istio-pilot listen port to report service discovery information to sidecars
          discoveryAddress: istio-pilot.istio-system:15007

Let’s put all the main control components (control plane) in the namespace istio-system in kubernetes.

The minimum configuration requires only Pilot deploy. Here we use the [following configuration](https://github.com/istio/istio/blob/release-1.0/install/kubernetes/helm/istio/charts/pilot/templates/deployment.yaml). And we will manually configure the injecting for the sidecar container.

Init container configuration:

    initContainers:
     - name: istio-init
       args:
       - -p
       - "15001"
       - -u
       - "1337"
       - -m
       - REDIRECT
       - -i
       - '*'
       - -b
       - '*'
       - -d
       - ""
       image: istio/proxy_init:1.0.0
       imagePullPolicy: IfNotPresent
       resources:
         limits:
           memory: 128Mi
       securityContext:
         capabilities:
           add:
           - NET_ADMIN

And sidecar configuration:

           - name: istio-proxy
             command:
             - "bash"
             - "-c"
             - |
               exec /usr/local/bin/pilot-agent proxy sidecar \
               --configPath \
               /etc/istio/proxy \
               --binaryPath \
               /usr/local/bin/envoy \
               --serviceCluster \
               service-name \
               --drainDuration \
               45s \
               --parentShutdownDuration \
               1m0s \
               --discoveryAddress \
               istio-pilot.istio-system:15007 \
               --discoveryRefreshDelay \
               1s \
               --connectTimeout \
               10s \
               --proxyAdminPort \
               "15000" \
               --controlPlaneAuthPolicy \
               NONE
             env:
             - name: POD_NAME
               valueFrom:
                 fieldRef:
                   fieldPath: metadata.name
             - name: POD_NAMESPACE
               valueFrom:
                 fieldRef:
                   fieldPath: metadata.namespace
             - name: INSTANCE_IP
               valueFrom:
                 fieldRef:
                   fieldPath: status.podIP
             - name: ISTIO_META_POD_NAME
               valueFrom:
                 fieldRef:
                   fieldPath: metadata.name
             - name: ISTIO_META_INTERCEPTION_MODE
               value: REDIRECT
             image: istio/proxyv2:1.0.0
             imagePullPolicy: IfNotPresent
             resources:
               requests:
                 cpu: 100m
                 memory: 128Mi
               limits:
                 memory: 2048Mi
             securityContext:
               privileged: false
               readOnlyRootFilesystem: true
               runAsUser: 1337
             volumeMounts:
             - mountPath: /etc/istio/proxy
               name: istio-envoy

For a successful deploy, you need to create ServiceAccount, ClusterRole, ClusterRoleBinding, CRD for Pilot; more information about these is available [here](https://github.com/istio/istio/tree/release-1.0/install/kubernetes/helm/istio/charts/pilot/templates). As a result, the service with injected sidecar and envoy will launch, retrieve all the discovery data from the pilot, and process the requests.

The crucial point is that all control plane components are stateless applications and can easily be scaled horizontally. All data are stored in etcd as custom descriptions of kubernetes resources.

Moreover, it is possible to run Istio (experimentally) outside the cluster and monitor and share service discovery between several kubernetes clusters. More information about this is available [here](https://istio.io/docs/setup/kubernetes/multicluster-install/). In a multicluster installation, consider the following limitations:

1. CIDR Pod and Service CIDR must be unique across all clusters and must not overlap.

1. All CIDR Pods must be accessible from any Pod CIDR between clusters.

1. All Kubernetes API servers must be accessible to each other.

This post will help you get started with Istio. However, there are many pitfalls to be considered in the upcoming posts. We’ll walk you through the features of external traffic routing, consider most common approaches to sidecar debugging and profiling. Furthermore, we’ll set up the tracing mechanism and scrutinize how it interacts with envoy.
