
# Monitoring your Kubernetes Deployments with Prometheus

In the first part of the Kubernetes monitoring series, we discussed how Kubernetes monitoring architecture is divided into the core metrics pipeline for system components and monitoring pipeline based on the Custom metrics API. Full monitoring pipelines based on the Custom metrics API can process diverse types of metrics (both core and non-core), which makes them a good fit for monitoring both cluster components and user applications running in your cluster(s).

Plenty of solutions exist for monitoring your Kubernetes clusters. Some of the most popular are Heapster, Prometheus, and a number of proprietary Application Performance Management (APM) vendors like Sysdig, Datadog, or Dynatrace.

In this article, we discuss Prometheus because it is open source software with native support for Kubernetes. Monitoring Kubernetes clusters with Prometheus is a natural choice because many Kubernetes components ship Prometheus-format metrics by default and, therefore, they can be easily discovered by Prometheus. We’re going to overview the Prometheus architecture and walk you through configuring and deploying it to monitor an example application shipping the Prometheus-format metrics. Let’s get started!

## What Is Prometheus?

[Prometheus](https://github.com/prometheus) is an open source monitoring and alerting toolkit originally developed by SoundCloud in 2012. Since then, the platform has attracted a vibrant developer and user community. Prometheus is now closely integrated into cloud-native ecosystem and has native support for containers and Kubernetes.

When you deploy Prometheus in production, you get a number of useful features and benefits:

### **A multi-dimensional data model**

Prometheus stores all data as time series identified by metric name and key/value pairs. The data format looks like this:

    <metric name>{<label name>=<label value>, …}

For example, using this format we can represent a total number of HTTP POST request to the /messages endpoint like this:

    api_http_requests_total{method=”POST”, handler=”/messages”}

This approach resembles the way Kubernetes organizes data with labels. Prometheus data model facilitates flexible and accurate time series and is especially useful if your data is high-dimensional.

### **A Flexible Query Language**

Prometheus ships with PromQL, a functional query language that leverages high dimensionality of data. It allows users to select, query, and aggregate metrics collected by Prometheus preparing them for subsequent analysis and visualization. PromQL is powerful in dealing with time series due to its native support for complex data types such as instant vectors and range vectors, as well as simple scalar and string data types.

### **Efficient Pull Model for Metrics Collection**

Prometheus collects metrics via a pull model over HTTP. This approach makes shipping application metrics to Prometheus very simple. In particular, you don’t need to push metrics to Prometheus explicitly. All you need to do is to expose a web port in your application and design a REST API endpoint that will expose the Prometheus-format metrics. If your application does not have Prometheus-format metrics, there are several metrics exporters that will help you convert it to the native Prometheus format. Once the /metrics endpoint is created, Prometheus will use its powerful auto-discover plugins to collect, filter, and aggregate the metrics. Prometheus has good support for a number of metrics providers including Kubernetes, Open Stack, GCE, AWS EC2, Zookeeper Serverset, and more.

### **Developed Ecosystem**

Prometheus has a developed ecosystem of components and tools including various client libraries for instrumenting application code, special-purpose exporters to convert data into Prometheus format, AlertManagers, web UI, and more.

Efficient auto-discovery features and excellent support for containers and Kubernetes make Prometheus a perfect choice for monitoring Kubernetes applications and cluster components. In this tutorial, we will monitor a simple web application exporting Prometheus-format metrics. We used an [example application](https://github.com/prometheus/prometheus/blob/master/docs/getting_started.md) from the Go client library that exports fictional RPC latencies of some service. To deploy the application in the Kubernetes cluster, we containerized it using Docker and pushed to the [Docker Hub repository](https://hub.docker.com/r/supergiantkir/prometheus-test-app/).

To complete examples used below, you’ll need the following prerequisites:

* A running Kubernetes cluster. See [Supergiant documentation](https://supergiant.readme.io/docs) for more information about deploying a Kubernetes cluster with Supergiant. As an alternative, you can install a single-node Kubernetes cluster on a local system using [Minikube](https://kubernetes.io/docs/getting-started-guides/minikube/).

* A **kubectl** command line tool installed and configured to communicate with the cluster. See how to install **kubectl** [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

## Step 1: Enabling RBAC for Prometheus

We need to grant some permissions to Prometheus to access pods, endpoints, and services running in your cluster. We can do this via the ClusterRole resource that defines an RBAC policy. In the ClusterRole manifest, we list various permissions for Prometheus to manipulate (read/write) various cluster resources. Let’s look at the manifest below:

    apiVersion: rbac.authorization.k8s.io/v1beta1
    kind: ClusterRole
    metadata:
      name: prometheus
    rules:
    - apiGroups: [""]
      resources:
      - nodes
      - services
      - endpoints
      - pods
      verbs: ["get", "list", "watch"]
    - apiGroups: [""]
      resources:
      - configmaps
      verbs: ["get"]
    - nonResourceURLs: ["/metrics"]
      verbs: ["get"]

The above manifest grants Prometheus the following cluster-wide permissions:

* read and watch access to pods, nodes, services, and endpoints.

* read access to ConfigMaps

* read access to non-resource URLs such as /metrics URLs shipping the Prometheus-format metrics.

In addition to ClusterRole , we need to create a ServiceAccount for Prometheus to represent its identity in the cluster.

    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: prometheus

Finally, we need to bind the ServiceAccount and ClusterRole using the ClusterRoleBinding resource. The ClusterRoleBinding allows associating a list of users, groups, or service accounts with a specific role.

    apiVersion: rbac.authorization.k8s.io/v1beta1
    kind: ClusterRoleBinding
    metadata:
      name: prometheus
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: prometheus
    subjects:
    - kind: ServiceAccount
      name: prometheus
      namespace: default

Note that roleRef.name should match the name of the ClusterRole created in the first step and the subjects.name should match the name of the ServiceAccount created in the second step.

We are going to create these resources in bulk, so put the above manifests into one file (e.g., rbac.yml ), separating each manifest by --- delimeter. Then run:

    kubectl create -f rbac.yml

    clusterrolebinding.rbac.authorization.k8s.io “prometheus” created
    clusterrole.rbac.authorization.k8s.io “prometheus” created
    serviceaccount “prometheus” created

## Step 2: Deploy Prometheus

The next step is configuring Prometheus. The configuration will contain a list of scrape targets and Kubernetes auto-discovery settings that will allow Prometheus to automatically detect applications that ship metrics.

    global:
      scrape_interval: 15s # By default, scrape targets every 15seconds. # Attach these labels to any time series or alerts when #communicating with external systems (federation, remote storage, #Alertmanager).
      external_labels:
        monitor: 'codelab-monitor'
    # Scraping Prometheus itself
    scrape_configs:
    - job_name: 'prometheus'
      scrape_interval: 5s
      static_configs:
      - targets: ['localhost:9090']
    - job_name: 'kubernetes-service-endpoints'
      kubernetes_sd_configs:
      - role: endpoints
      relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_service_name]
        action: replace
        target_label: kubernetes_name

As you see, the configuration contains two main sections: global configuration and *scrape* configuration. The global section includes parameters that are valid in all configuration contexts. In this section, we define a 15-second scrape interval and external labels.

In its turn, the scrape config section defines jobs/targets for Prometheus to watch. Here, you can override global values such as a scrape interval. In each job section, you can also provide a target endpoint for Prometheus to listen to. As you understand, Kubernetes services and deployments are dynamic. Therefore, we can’t know their URL before running them. Fortunately, Prometheus auto-discover features can address this problem.

Prometheus ships with the Kubernetes auto-discover plugin named kubernetes_sd_configs that we use in the second job definition. We set kubernetes_sd_configs to watch for service endpoints shipping Prometheus-format metrics. We have also included some re-label rules for replacing lengthy Kubernetes names and labels with custom values to simplify monitoring. For this tutorial, we targeted only service endpoints, but you can configure kubernetes_sd_configs to watch nodes, pods, and any other resource in your Kubernetes cluster.

So far, we’ve mentioned just a few configuration parameters supported by Prometheus. You may be also interested in some others such as:

* scrape_timeout — how long it takes until a scrape request times out.

* basic_auth for setting “Authorization” header for each scrape request.

* service-specific auto-discover configurations for Consul, Amazon EC2, GCE, etc.

For a full list of available configuration options, see the [official Prometheus documentation](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#%3Ckubernetes_sd_config).

Let’s save the configuration above in the prometheus.yml file and create the ConfigMap with the following command:

    kubectl create configmap prometheus-config —-from-file prometheus.yml

    configmap “prometheus-config” created

Next, we will deploy Prometheus using the container image from the Docker Hub repository. Our deployment manifest looks like this:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: prometheus-deployment
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: prometheus
      template:
        metadata:
          labels:
            app: prometheus
        spec:
          containers:
          - name: prometheus-cont
            image: prom/prometheus
            volumeMounts:
            - name: config-volume
              mountPath: /etc/prometheus/prometheus.yml
              subPath: prometheus.yml
            ports:
            - containerPort: 9090
          volumes:
          - name: config-volume
            configMap:
              name: prometheus-config
          serviceAccountName: prometheus

To summarize what this manifest does:

* Launches two Prometheus replicas listening on the port 9090 .

* Mounts the ConfigMap created previously at the default Prometheus config path of /etc/prometheus/prometheus.yml

* Associates Prometheus Service Account with the deployment to grant needed permissions.

Let’s save the manifest in the prometheus-deployment.yml and create the deployment:

    kubectl create -f prometheus-deployment.yaml

    deployment.extensions “prometheus-deployment” created

To access the Prometheus web interface, we also need to expose the deployment as a service. We used the NodePort service type:

    kind: Service
    apiVersion: v1
    metadata:
      name: prometheus-service
    spec:
      selector:
        app: prometheus
      ports:
      - name: promui
        nodePort: 30900
        protocol: TCP
        port: 9090
        targetPort: 9090
      type: NodePort

Let’s create the service, saving the manifest in the prometheus-service.yaml and running the command below:

    kubectl create -f prometheus-service.yaml

    service “prometheus-service” created

Alternatively, you can expose the deployment from your terminal. By doing so, you don’t need to define the Service manifest:

    kubectl expose deployment prometheus-deployment --type=NodePort --name=prometheus-service

    service "prometheus-service" exposed

Once the deployment is exposed, you can access the Prometheus web interface. If you are using Minikube, you can find the Prometheus UI URL by running minikube service with the--url flag:

    minikube service prometheus-service --url

    http://192.168.99.100:30900

Take note of the URL to access the Prometheus UI a little bit later when our test metrics app is deployed.

## Step 3: Deploy an Example App Shipping RPC Latency Metrics

Prometheus is now deployed, so we are ready to feed it some metrics. Let’s deploy our example app serving metrics at the /metrics REST endpoint. Below is the deployment manifest we used:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: rpc-app-deployment
      labels:
        app: rpc-app
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: rpc-app
      template:
        metadata:
          labels:
            app: rpc-app
        spec:
          containers:
          - name: rpc-app-cont
            image: supergiantkir/prometheus-test-app
            ports:
            - name: web
              containerPort: 8081

This deployment manifest is quite self-explanatory. Once deployed, the app will be shipping random RPC latencies data to /metrics endpoint. Please, make sure that all the labels and label selectors match each other if you prefer to use your own names.

Go ahead and create the deployment:

    kubectl create -f rpc-app-deployment.yaml

    deployment.apps “rpc-app-deployment” created

As you remember, we configured Prometheus to watch service endpoints. That’s why, we need to expose our app’s deployment as a service.

    apiVersion: v1
    kind: Service
    metadata:
      name: rpc-app-service
      labels:
        app: rpc-app
    spec:
      ports:
      - name: web
        port: 8081
        targetPort: 8081
        protocol: TCP
      selector:
        app: rpc-app
      type: NodePort

For clarity, we set the value of the spec.ports[].targetPort to be the same as spec.ports[].port , although Kubernetes makes it automatically if no value is provided for targetPort .

As with the Prometheus service, you can either create the service from the manifest or expose it inline in your terminal. If you opt for the manifest, run:

    kubectl create -f rpc-app-service.yaml

    service “rpc-app-service” created

If you prefer the quick inline way, run:

    kubectl expose deployment rpc-app-deployment --type=NodePort --name=rpc-app-service

    service "rpc-app-service" exposed

Let’s verify that the service was successfully created:

    kubectl describe svc rpc-app-service

    Name:                     rpc-app-service

    Namespace:                default

    Labels:                   app=rpc-app

    Annotations:              <none>

    Selector:                 app=rpc-app

    Type:                     NodePort

    IP:                       10.110.41.174

    Port:                     web  8081/TCP

    TargetPort:               8081/TCP

    NodePort:                 web  32618/TCP

    Endpoints:                172.17.0.10:8081,172.17.0.9:8081

    Session Affinity:         None

    External Traffic Policy:  Cluster

    Events:                   <none>

As you see, the NodePort was assigned and the deployment’s endpoints were successfully added to the service. We can now access the app’s metrics endpoint on the specified IP and port. If you are using Minikube, you’ll first need to get the service’s IP with the following command:

    minikube service rpc-app-service --url

    http://192.168.99.100:30658

Now, let’s use cURL to GET some metrics from that endpoint:

    curl http://192.168.99.100:30658/metrics

    # TYPE promhttp_metric_handler_requests_total counter

    promhttp_metric_handler_requests_total{code="200"} 0
    promhttp_metric_handler_requests_total{code="500"} 0
    promhttp_metric_handler_requests_total{code="503"} 0
    # HELP rpc_durations_histogram_seconds RPC latency distributions.
    # TYPE rpc_durations_histogram_seconds histogram

    rpc_durations_histogram_seconds_bucket{le="-0.00099"} 0
    rpc_durations_histogram_seconds_bucket{le="-0.00089"} 0
    rpc_durations_histogram_seconds_bucket{le="-0.0007899999999999999"} 0
    rpc_durations_histogram_seconds_bucket{le="-0.0006899999999999999"} 2
    rpc_durations_histogram_seconds_bucket{le="-0.0005899999999999998"} 18
    rpc_durations_histogram_seconds_bucket{le="-0.0004899999999999998"} 59
    rpc_durations_histogram_seconds_bucket{le="-0.0003899999999999998"} 236
    rpc_durations_histogram_seconds_bucket{le="-0.0002899999999999998"} 669
    rpc_durations_histogram_seconds_bucket{le="-0.0001899999999999998"} 1514
    rpc_durations_histogram_seconds_bucket{le="-8.999999999999979e-05"} 2959
    rpc_durations_histogram_seconds_bucket{le="1.0000000000000216e-05"} 4727
    rpc_durations_histogram_seconds_bucket{le="0.00011000000000000022"} 6562
    rpc_durations_histogram_seconds_bucket{le="0.00021000000000000023"} 8059
    rpc_durations_histogram_seconds_bucket{le="0.0003100000000000002"} 8908
    rpc_durations_histogram_seconds_bucket{le="0.0004100000000000002"} 9350
    rpc_durations_histogram_seconds_bucket{le="0.0005100000000000003"} 9514
    rpc_durations_histogram_seconds_bucket{le="0.0006100000000000003"} 9570
    rpc_durations_histogram_seconds_bucket{le="0.0007100000000000003"} 9585
    rpc_durations_histogram_seconds_bucket{le="0.0008100000000000004"} 9588
    rpc_durations_histogram_seconds_bucket{le="0.0009100000000000004"} 9588
    rpc_durations_histogram_seconds_bucket{le="+Inf"} 9588
    rpc_durations_histogram_seconds_sum 0.11031870987310399
    rpc_durations_histogram_seconds_count 9588
    # HELP rpc_durations_seconds RPC latency distributions.
    # TYPE rpc_durations_seconds summary
    rpc_durations_seconds{service="exponential",quantile="0.5"} 6.43688069700151e-07
    rpc_durations_seconds{service="exponential",quantile="0.9"} 2.3703557539528334e-06
    rpc_durations_seconds{service="exponential",quantile="0.99"} 4.491775587389532e-06
    rpc_durations_seconds_sum{service="exponential"} 0.014317520025277117
    rpc_durations_seconds_count{service="exponential"} 14369
    rpc_durations_seconds{service="normal",quantile="0.5"} 5.97571029483546e-06
    rpc_durations_seconds{service="normal",quantile="0.9"} 0.0002795950678545625
    rpc_durations_seconds{service="normal",quantile="0.99"} 0.0004838671111576318
    rpc_durations_seconds_sum{service="normal"} 0.11031870987310399
    rpc_durations_seconds_count{service="normal"} 9588
    rpc_durations_seconds{service="uniform",quantile="0.5"} 8.961255876119688e-05
    rpc_durations_seconds{service="uniform",quantile="0.9"} 0.0001764412468147929
    rpc_durations_seconds{service="uniform",quantile="0.99"} 0.00019807911315607854
    rpc_durations_seconds_sum{service="uniform"} 0.715789691590982
    rpc_durations_seconds_count{service="uniform"} 7195

As you see, the request returned a number of Prometheus-formatted RPC latencies metrics. Each metric is formatted as <metric name>{<label name>=<label value>, …} and has a unique value.

Thanks to the Prometheus Kubernetes auto-discover feature, we can expect that Prometheus has automatically discovered the app and has begun pulling these metrics. Let’s access the Prometheus web interface to verify this. Use you Prometheus service IP and the NodePort obtained in Step 2 to access the Prometheus UI.

If you go to the /targets endpoint, you’ll see the list of the current Prometheus targets. There might be a lot of targets because we’ve configured Prometheus to watch all service endpoints. Among them, you’ll find a target labeled app=”rpc-app” . That’s our app. You can also find other labels and see the time of the last scrape.

![](https://cdn-images-1.medium.com/max/5624/1*gz4dOOPeIVjBF1N_fvwXww.png)

In addition, you can see the current Prometheus configuration under **Status -> Configuration** tab:

![](https://cdn-images-1.medium.com/max/5760/1*KyVApt74W08QNv1XE92o7g.png)

Finally, we can visualize RPC time series generated by our example app. To do this, go to the **Graph** tab where you can select the metrics to visualize.

![](https://cdn-images-1.medium.com/max/5760/1*qccUqRImSKiOmLflg87PxA.png)

In the image above, we visualized the rpc_durations_histogram_seconds_bucket metrics. You can play around with other RPC metrics and native Prometheus metrics as well. The web interface also supports Prometheus query language PromQL to select and aggregate metrics you need. PromQL has a rich functional semantics that allows working with time series, instance and range vectors, scalars, and strings. To learn more about PromQL, check out the [official documentation](https://prometheus.io/docs/prometheus/latest/querying/basics/).

## Conclusion

That’s it! We’ve learned how to configure Prometheus to monitor applications serving Prometheus-format metrics. Prometheus has a complex configuration language and settings, so we’ve just scratched the surface. Although Prometheus is a powerful tool, it might be challenging to configure and run it without a good knowledge of domain-specific language and configuration. To fill this gap, in [the next tutorial](https://medium.com/kubernetes-tutorials/simple-management-of-prometheus-monitoring-pipeline-with-the-prometheus-operator-b445da0e0d1a) we’ll look into configuring and managing your Prometheus instances with Prometheus Operator — a useful software management tool designed to simplify monitoring of your apps with Prometheus. Stayed tuned to our blog to find out more soon!

*Originally published at [supergiant.io](https://supergiant.io/blog/monitoring-your-kubernetes-deployments-with-prometheus/).*
