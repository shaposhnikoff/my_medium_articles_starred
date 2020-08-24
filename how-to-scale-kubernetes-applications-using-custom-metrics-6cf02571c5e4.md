Unknown markup type 10 { type: [33m10[39m, start: [33m39[39m, end: [33m62[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m106[39m, end: [33m125[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m22[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m30[39m, end: [33m35[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m39[39m, end: [33m45[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m39[39m, end: [33m61[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m26[39m, end: [33m41[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m38[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m61[39m, end: [33m80[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m12[39m, end: [33m16[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m22[39m, end: [33m34[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m12[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m29[39m, end: [33m40[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m50[39m, end: [33m62[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m24[39m, end: [33m37[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m53[39m, end: [33m72[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m16[39m, end: [33m29[39m }

# How to Scale Kubernetes Applications Using Custom Metrics

Scale your containers using custom Stackdriver metrics that are important to your business

![Photo by [Carlos Muza](https://unsplash.com/@kmuza?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral).](https://cdn-images-1.medium.com/max/4852/0*EkgNtmZL0OipjIM1)*Photo by [Carlos Muza](https://unsplash.com/@kmuza?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral).*

If you look at the most popular features of Kubernetes, chances are that autoscaling will top the list. Kubernetes provides a resource called “HorizontalPodAutoscaler,” which is a built-in feature to scale your containers based on multiple factors, such as resource utilisation (CPU and memory), network utilisation, or traffic within the pods. If your cluster is nearing the resource limit and you need to add more nodes in the cluster, then managed Kubernetes services such as GKE provide Kubernetes Cluster Autoscaler out of the box.

Imagine you have launched an app and you naturally started small, as you did not anticipate much demand. However, a social media post gets your application to go viral and suddenly people realise how cool it is. Now everyone wants to use your app and there is a massive surge in traffic! You get worried because you don’t know whether your application can take on the additional load.

Thankfully, you realise you had architected your application to run on Kubernetes and deployed it on Google Kubernetes Engine. The app scales seamlessly according to demand because of the near-infinite scaling capacity provided by Google Cloud.

## How Horizontal Pod Autoscaler Works

A Horizontal Pod Autoscaler broadly makes use of three categories of metrics out of the box:

* Resource metrics — These are metrics such as CPU and memory utilisation.

* Pod metrics — These are metrics specific to the pod, such as network utilisation and traffic (like packets per second).

* Object metrics — These are metrics of particular types of objects. For example, if you make use of Ingress, you can make use of requests per second to scale your containers.

![](https://cdn-images-1.medium.com/max/2000/0*prTmB8dtLjgCB15L.png)

You can achieve all of this by using a HorizontalPodAutoscaler manifest like the one below:

<iframe src="https://medium.com/media/64bfff1893e5d43c9b995304a08def4d" frameborder=0></iframe>

If you apply the manifest above, it will autoscale Nginx deployment between one and ten replicas based on:

* Resource metric — Average CPU utilisation of less than 50%.

* Pod metric — Average pod traffic of 1K packets per second.

* Object metric — Traffic of 10K requests per second across all pods.

Object metrics have the option to use both Value and Average Value. To give an example, the object metric requests-per-second used in the Ingress part will describe the total traffic going to all the pods managed by the Ingress resource. If we had used Average Value, it would have divided the value with the number of pods and compared it with the target.

Keep in mind that with an increasing number of pods, the percentage of load distribution decreases. For example, if you are running a single pod and you spin another one, the new pod will take 50% of your load. But if you spin the third one, it will take 33%. A tenth one will just take 10% of your load. Therefore, always plan for the maximum number of pods while keeping in mind how much capacity you want and always over-provision your cluster by some amount to handle some burst capacity. Making use of the Kubernetes Cluster Autoscaler along with the Horizontal Pod Autoscaler is a great idea.

## Using External Metrics

Scaling based on Kubernetes object metrics is great. However, it doesn’t drive the customer experience. Metrics such as CPU utilisation, memory utilisation, and network throughput would make sense to an engineer. Still, for a customer using your application, the key performance indicators are metrics such as response time. Customers do not care how you run your application. All they care about is getting a seamless user experience — one glitch and you lose a potential customer.

Fortunately, Kubernetes provides a way to use external metrics to scale your containers. Let’s take an example of using custom metrics on Google Kubernetes Engine. Google uses Stackdriver as its default monitoring solution, and you can ship custom metrics from your application to it. You can also deploy a monitoring solution such as Prometheus, which can track custom metrics such as response time and publish that on Stackdriver. You can then use the Custom Metrics Stackdriver Adapter for Kubernetes to read Stackdriver metrics to autoscale your application containers.

![Custom Metrics](https://cdn-images-1.medium.com/max/2000/1*C5ZvajXL7mMwM-i1hTd_9g.png)*Custom Metrics*

So without further ado, let us look at how we can achieve that.

### Enable Stackdriver on Google Cloud

If you haven’t enabled Stackdriver yet on Google Cloud, go to your Google Cloud Console -> Monitoring. That should create a new workspace for you like the one below:

![](https://cdn-images-1.medium.com/max/3840/1*myZJc1X3lEn3cq-XDxIKWw.png)

### Deploy Custom Metrics Stackdriver Adapter

Google Kubernetes requires the [Custom Metrics Stackdriver Adapter](https://github.com/GoogleCloudPlatform/k8s-stackdriver/tree/master/custom-metrics-stackdriver-adapter) to allow Horizontal Pod Autoscalers to query the custom metrics from Stackdriver. Log into your GKE cluster and start by granting the current user the cluster-admin role so that they can deploy the adapter:

    $ kubectl create clusterrolebinding cluster-admin-binding \
        --clusterrole cluster-admin --user "$(gcloud config get-value account)"

Then run the command below to deploy the adapter:

    $ kubectl create -f [https://raw.githubusercontent.com/GoogleCloudPlatform/k8s-stackdriver/master/custom-metrics-stackdriver-adapter/deploy/production/adapter.yaml](https://raw.githubusercontent.com/GoogleCloudPlatform/k8s-stackdriver/master/custom-metrics-stackdriver-adapter/deploy/production/adapter.yaml)

### Export custom metrics from your application

You would then need to deploy an application to export metrics to Stackdriver. You can either use a monitoring tool such as Prometheus to do so (topic for another article) or a custom application that sends metrics to Stackdriver, which we are going to discuss now.

You can either pre-define a custom metric or use the custom metric auto-creation feature of Stackdriver to do so. You need to ensure you meet the following requirements:

* Metric kind should be GAUGE.

* The metric type can either be INT64 or DOUBLE.

* You should prefix the metric name with custom.googleapis.com/.

* The resource type must be "gke_container".

* And the resource labels should have a pod_id and an optional container_name = "".

* project_id, zone, and cluster_name are mandatory values, and they can be obtained from the [metadata server](https://cloud.google.com/compute/docs/storing-retrieving-metadata) using Google Cloud's compute metadata client.

* You can set namespace_id and instance_id to any value.

<iframe src="https://medium.com/media/1129a0c4c153cf337789753c4cacac2d" frameborder=0></iframe>

The manifest above defines a custom metric called responsetime and sends a configured response time every five seconds. Of course, the above is just for testing. Realistically, you need to have a monitoring solution in place that can capture and send those metrics to Stackdriver.

Google provides an example repository to demonstrate how you can ship metrics to Stackdriver, and we will explicitly make use of [this one](https://github.com/GoogleCloudPlatform/kubernetes-engine-samples/tree/master/custom-metrics-autoscaling/direct-to-sd). So let’s get started:

    $ git clone [https://github.com/GoogleCloudPlatform/kubernetes-engine-samples.git](https://github.com/GoogleCloudPlatform/kubernetes-engine-samples.git)
    $ cd kubernetes-engine-samples/custom-metrics-autoscaling/direct-to-sd/
    $ sed -i 's/foo/responsetime/g' custom-metrics-sd.yaml
    $ kubectl apply -f custom-metrics-sd.yaml
    $ kubectl get pod
    NAME                               READY   STATUS    RESTARTS   AGE
    custom-metric-sd-b6d766db9-96fd4   1/1     Running   0          7s
    $ kubectl logs custom-metric-sd-b6d766db9-96fd4
    2020/04/20 22:49:32 Failed to write time series data: googleapi: Error 500: One or more TimeSeries could not be written: An internal error occurred.: timeSeries[0], backendError
    2020/04/20 22:49:37 Failed to write time series data: googleapi: Error 500: One or more TimeSeries could not be written: An internal error occurred.: timeSeries[0], backendError
    2020/04/20 22:49:42 Finished writing time series with value: 0xc420015290
    2020/04/20 22:49:47 Finished writing time series with value: 0xc420015290
    2020/04/20 22:49:52 Finished writing time series with value: 0xc420015290
    2020/04/20 22:49:57 Finished writing time series with value: 0xc420015290
    2020/04/20 22:50:02 Finished writing time series with value: 0xc420015290
    2020/04/20 22:50:07 Finished writing time series with value: 0xc420015290
    2020/04/20 22:50:12 Finished writing time series with value: 0xc420015290

Right, so now we have the pod running and shipping custom metrics to Stackdriver. Let us check whether the parameters are reflected on Stackdriver or not.

Go to Google Cloud Console -> Monitoring -> Metrics Explorer.

On the METRIC tab, type gke_container and select the custom/responsetime metric.

![Custom Metrics on Stackdriver](https://cdn-images-1.medium.com/max/3840/1*5yiCbcanne8sU5pN-DaIlg.png)*Custom Metrics on Stackdriver*

As you see, the responsetime parameter is displayed and the current value is 40. We have successfully enabled custom metrics export from Google Kubernetes Engine to Stackdriver.

### Deploy the Horizontal Pod Autoscaler

As we have deployed the metrics exporter, let us implement a HorizontalPodAutoscaler that scales based on the metrics above:

<iframe src="https://medium.com/media/ac373b41bdd11ada1513c3b44b900f1c" frameborder=0></iframe>

    $ sed -i 's/foo/responsetime/g' custom-metrics-sd-hpa.yaml
    $ kubectl apply -f custom-metrics-sd-hpa.yaml
    $ kubectl get hpa
    NAME               REFERENCE                     TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
    custom-metric-sd   Deployment/custom-metric-sd   40/20     1         5         1          17s
    $ kubectl get pods
    NAME                               READY   STATUS    RESTARTS   AGE
    custom-metric-sd-b6d766db9-96fd4   1/1     Running   0          10m
    custom-metric-sd-b6d766db9-lbxnr   1/1     Running   0          97s
    custom-metric-sd-b6d766db9-n9tfv   1/1     Running   0          113s
    custom-metric-sd-b6d766db9-qdtgp   1/1     Running   0          97s
    custom-metric-sd-b6d766db9-xxhrt   1/1     Running   0          82s

As you see, the pods have quickly climbed to five replicas, as the parameters are crossing the expected average value of 20.

Let us modify the average response time to ten and try again:

    $ sed -i 's/40/10/g' custom-metrics-sd.yaml
    $ kubectl apply -f custom-metrics-sd.yaml
    $ kubectl get pods
    NAME                               READY   STATUS        RESTARTS   AGE
    custom-metric-sd-b6d766db9-lbxnr   0/1     Terminating   0          2m37s
    custom-metric-sd-b6d766db9-n9tfv   0/1     Terminating   0          2m53s
    custom-metric-sd-b6d766db9-qdtgp   0/1     Terminating   0          2m37s
    custom-metric-sd-b6d766db9-xxhrt   0/1     Terminating   0          2m22s
    custom-metric-sd-f6b4cc784-qbmqx   1/1     Running       0          2m20s
    $ kubectl get pod
    NAME                                READY   STATUS    RESTARTS   AGE
    custom-metric-sd-6ff96cfbf8-nrzb5   1/1     Running   0          4m5s

As you see, the HPA is terminating the pods to scale down the replica set.

If we now look back into the Stackdriver dashboard, we would see this:

![Custom Metrics](https://cdn-images-1.medium.com/max/3840/1*mNt7-PcwF0-1enGh6_xfHg.png)*Custom Metrics*

Congratulations! You have successfully configured a Horizontal Pod Autoscaler based on custom Stackdriver metrics on Google Kubernetes Engine.

## Conclusion

Thanks for reading! I hope you enjoyed the article. Remember, there are many ways of scaling your workloads based on your use cases. However, scaling your containers based on customer experience metrics would surely help your business go the extra mile and delight your customers!
