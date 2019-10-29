Unknown markup type 10 { type: [33m10[39m, start: [33m183[39m, end: [33m206[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m95[39m, end: [33m129[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m101[39m, end: [33m133[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m37[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m38[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m52[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m50[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m40[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m41[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m47[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m45[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m29[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m28[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m40[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m40[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m44[39m }

# A Deep Dive into Kubernetes Metrics — Part 4: The Kubernetes API Server



This is Part 4 of a [multi-part series](https://blog.freshtracks.io/a-deep-dive-into-kubernetes-metrics-b190cc97f0f6) about all the metrics you can gather from your Kubernetes cluster.

In [Part 3](https://blog.freshtracks.io/a-deep-dive-into-kubernetes-metrics-part-3-container-resource-metrics-361c5ee46e66), I dug deeply into all the container resource metrics that are exposed by the kubelet. In this article, I will cover the metrics that are exposed by the Kubernetes API server.

The Kubernetes API server is the interface to all the capabilities that Kubernetes provides. You use the API server to control all operations that Kubernetes can perform. Monitoring this critical component is critical to ensure a smooth running cluster.

The API server metrics are grouped into a few major categories:

* Request Rates and Latencies

* Performance of controller work queues

* Etcd helper cache work queues and cache performance

* General process status (File Descriptors/Memory/CPU Seconds)

* Golang status (GC/Memory/Threads)

Here is the Prometheus configuration we use for getting metrics from the Kubernetes API server, even in environments where the masters are hosted for you:

    - job_name: kubernetes-apiservers
          scrape_interval: 10s
          scrape_timeout: 10s
          metrics_path: /metrics
          scheme: https
          kubernetes_sd_configs:
          - api_server: null
            role: endpoints
            namespaces:
              names: []
          bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
          tls_config:
            ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
            insecure_skip_verify: true
          relabel_configs:
          - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
            separator: ;
            regex: default;kubernetes;https
            replacement: $1
            action: keep

Now that we are collecting more than 170 metrics, let’s take a look at what the API Server is telling us.

## Request Rates and Latencies

As in prior articles in this series, I will be using a particular method for choosing the important metrics to start watching. In [Part 2](https://blog.freshtracks.io/a-deep-dive-into-kubernetes-metrics-part-2-c869581e9f29), I mentioned that the RED method, Rate, Errors, and Duration, could be applied to “Services”. The API server is a service, so we will look at these metrics.

The API server understands the Kubernetes nouns like nodes, pods, and namespaces. If we want to get a feel for how often these resources are being requested we can look at the metric apiserver_request_count. This is the **Rate** metric:

    sum(rate(**apiserver_request_count**[5m])) by (resource, subresource, verb)

This will give you a five minute rate of all the Kubernetes resources by “verb”. The verbs in this case are HTTP verbs; WATCH, PUT, POST, PATCH, LIST, GET, DELETE, and CONNECT.

The **Errors** for the API server can be tracked as HTTP 5xx errors. Use this query to get the ratio of errors to the request rate:

    rate(**apiserver_request_count**{code=~"^(?:5..)$"}[5m]) / rate(**apiserver_request_count**[5m])

For **Duration**, we will look at the p90th latency for all the resources and verbs. Use the metricapiserver_request_latencies_bucket:

    histogram_quantile(0.9, sum(rate(**apiserver_request_latencies_bucket**[5m])) 
    by (le, resource, subresource, verb) ) / 1e+06

## Performance of Controller Work Queues

All work submitted to a Kubernetes cluster is handled by a controller. The work is queued and the controller works the queue as part of its control loop. Many metrics are collected about the performance of the work queue.

For any given controller, there are nine series that are [reported](https://github.com/kubernetes/kubernetes/blob/3dbbd0bdf44cb07fdde85aa392adf99ea7e95939/pkg/util/workqueue/prometheus/prometheus.go#L34) from the API server. Let’s use the APIServiceRegistrationController controller metrics as an example:

* APIServiceRegistrationController_**adds** — A counter for the number of adds to the system. Use rate() over this value.

* APIServiceRegistrationController_**depth** — The depth of the work queue. Generally this should always be near zero.

* APIServiceRegistrationController_**queue_latency** — Contains the quantiles of the summary metric for time spent in the queue.

* APIServiceRegistrationController_**queue_latency_count** — Contains the count of the number of items ever in the work queue (since the last restart).

* APIServiceRegistrationController_**queue_latency_sum** — Contains the sum of all time spent in this work queue.

* APIServiceRegistrationController_**retries** — A counter for the number of retries.

* APIServiceRegistrationController_**duration** — Contains the quantiles of the summary metric for processing time.

* APIServiceRegistrationController_**duration_count** — Contains the count of the number of things in the work queue.

* APIServiceRegistrationController_**duration_sum** — Contains the sum of all processing time duration.

## Etcd Interactions

The API Server keeps a write-through cache of objects from etcd. The [metrics](https://github.com/kubernetes/kubernetes/blob/2f09876c44bd0f8bde4c14c03be193bf53419ad8/staging/src/k8s.io/apiserver/pkg/storage/etcd/metrics/metrics.go#L28) look like this:

* etcd_helper_cache_entry_count — The number of elements in the cache.

* etcd_helper_cache_hit_count — The cache hit count.

* etcd_helper_cache_miss_count — The cache miss count.

* etcd_request_cache_add_latencies_summary — The amount time in microseconds to add entries to the cache.

* etcd_request_cache_add_latencies_summary_count — A counter for the number of cache adds.

* etcd_request_cache_add_latencies_summary_sum — The total number of time spent putting items in the cache.

* etcd_request_cache_get_latencies_summary — The amount of time in microseconds to get entries from the cache.

* etcd_request_cache_get_latencies_summary_count — A counter for the number of cache gets.

* etcd_request_cache_get_latencies_summary_sum — The total amount of time spent getting items from the cache.

## Standard Prometheus Golang Client Library Metrics

The golang client library for Prometheus collects many metrics about the running process and the golang runtime. I’m not going to enumerate all these metrics in this article. You can view the process metric definitions [here](https://github.com/prometheus/client_golang/blob/f1323f902ca878b3a87e1a76458be272fe1efe0d/prometheus/process_collector.go#L56) and the golang metric definitions [here](https://github.com/prometheus/client_golang/blob/bcbbc08eb2ddff3af83bbf11e7ec13b4fd730b6e/prometheus/go_collector.go#L28).

## Wrapping up

As is the case with all things Kubernetes, the API server is very well instrumented using the Prometheus metrics format.

![](https://cdn-images-1.medium.com/max/2496/1*TuGW7Lvvjmvks1Zap88NJg.png)

FreshTracks simplifies Kubernetes visibility. Hosted Prometheus and Grafana with machine learning enriched data for the best Day-2 Kubernetes metrics experience.
