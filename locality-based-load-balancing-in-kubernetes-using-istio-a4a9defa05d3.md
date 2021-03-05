
# Locality-Based Load Balancing in Kubernetes Using Istio

Route requests within your service mesh using geographic location to improve performance and save money

![Photo by [Krzysztof Hepner](https://unsplash.com/@nsx_2000?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral).](https://cdn-images-1.medium.com/max/12000/0*-cDFcKrvzEu2E4u1)*Photo by [Krzysztof Hepner](https://unsplash.com/@nsx_2000?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral).*

[Istio](https://istio.io/) is one of the most feature-rich and robust service meshes for Kubernetes on the market. It is an open-source tool developed by Google, Lyft, and IBM and is quickly gaining popularity.

One of its significant features is traffic management. Istio provides a Layer 7 Proxy that helps you route traffic on multiple factors, such as HTTP headers, source IP, URL path, and hostname.

Organisations that have a global presence often run a high-scale global service mesh that involves microservices spread across the globe. Microservices need to interact with each other to provide complete functionality to customers. To ensure that customers get the best performance possible, it makes sense to route traffic to the nearest microservice rather than load balancing in a round-robin fashion, which Kubernetes provides by default.

Cost is another factor, as most cloud providers charge egress fees if traffic moves between zones and regions. Traffic within the same zone is treated as internal and therefore not charged.

Routing traffic to services within the same zone will save you plenty of money in egress charges if you architect your mesh using Istio.

Istio provides locality-based routing, which helps you route traffic to pods closest to the originating pod. That ensures that your customers experience low latency and you benefit from saved egress charges. A win-win for everyone!

This article is a follow-up to [Traffic Mirroring in Kubernetes Using Istio](https://medium.com/better-programming/traffic-mirroring-in-kubernetes-using-istio-dad0976b4e1). Today, let’s discuss locality-based routing in Istio.

## Prerequisites

Ensure that you have a regional or multi-regional Kubernetes cluster. For this demonstration, I have used a regional Google Kubernetes Engine with one node per zone in the us-central region. You also need to have some familiarity with Istio. Check out [How to Manage Microservices on Kubernetes With Istio](https://medium.com/better-programming/how-to-manage-microservices-on-kubernetes-with-istio-c25e97a60a59) for a brief introduction.

## Install Istio

Install Istio within your Kubernetes cluster by following the [Getting Started With Istio on Kubernetes](https://medium.com/better-programming/getting-started-with-istio-on-kubernetes-e582800121ea) guide. You don’t need to install the Book Info application for this demo.

Istio uses node labels to understand the source traffic’s region and zone. If you are using a managed Kubernetes cluster on the cloud, your cloud provider takes care of labelling the nodes.

If you are running an on-premise or custom setup, then label your nodes appropriately with regions and zones. See [well-known labels for region and zone](https://kubernetes.io/docs/reference/kubernetes-api/labels-annotations-taints/#failure-domainbetakubernetesioregion) for more details.

As we are running a managed Kubernetes service on the cloud, let’s start by listing the nodes and labels. We need this to understand which node belongs to what region.

<iframe src="https://medium.com/media/859b44c40e0067f042a942373dc87db2" frameborder=0></iframe>

We see that:

* Node gke-cluster-1-default-pool-527dc04e-kzgp belongs to us-central1-f.

* Node gke-cluster-1-default-pool-5cfcdb08–282p belongs to us-central1-a.

* Node gke-cluster-1-default-pool-97820a63-mfmq belongs to us-central1-c.

Create two namespaces, frontend and backend:

    $ kubectl create ns frontend
    namespace/frontend created
    $ kubectl create ns backend
    namespace/backend created

Label the namespaces to enable Istio’s automatic sidecar injection:

    $ kubectl label namespace frontend istio-injection=enabled
    namespace/frontend labeled
    $ kubectl label namespace backend istio-injection=enabled
    namespace/backend labeled

## Deploy the Back-End Nginx Service

Deploy two versions (v1 and v2) of [NGINX](https://www.nginx.com/) on Kubernetes. Add a node selector on nginx-v1 for us-central1-a and nginx-v2 for us-central1-f.

Deploy nginx-v1:

<iframe src="https://medium.com/media/038b6511171384f41eaff149ca5c4d82" frameborder=0></iframe>

Deploy nginx-v2:

<iframe src="https://medium.com/media/c44c25ac29f8ff0496016a4f67e23fb9" frameborder=0></iframe>

Create an NGINX service to expose the pods:

<iframe src="https://medium.com/media/9d401e5c0ee9960caa3f436ca59bd54d" frameborder=0></iframe>

List the pods to check if they are running in the correct zones:

<iframe src="https://medium.com/media/e0326a6bc5eea058e9b8ad04e6a0a1fd" frameborder=0></iframe>

## Deploy the Front-End sleep Microservice

Create a sleep deployment with a replica on each node. We use this to generate traffic to the back-end NGINX pods:

<iframe src="https://medium.com/media/5f2214217569b127d2bb6416f06c452e" frameborder=0></iframe>

We also need to create a service for the sleep pods. That is necessary for Istio’s service discovery to allow locality-based load balancing:

<iframe src="https://medium.com/media/e697a1fa2d5a2b2d2b5a8bb10cc831a0" frameborder=0></iframe>

List the pods to see if they are running across all nodes:

<iframe src="https://medium.com/media/8eef22efa95054ddfdb3dc56c42529e9" frameborder=0></iframe>

And they are! Let’s export some variables:

    $ export SLEEP_ZONE_1=sleep-bb596f69d-85s6d
    $ export SLEEP_ZONE_2=sleep-bb596f69d-c7gxr

Now let’s execute a sleep pod on us-central1-a and call the NGINX service:

<iframe src="https://medium.com/media/a480b213693bab7b904c6dc21893b5d6" frameborder=0></iframe>

They are now load-balanced entirely in a round-robin fashion.

## Locality-Prioritised Load Balancing

The default behaviour of locality-based load balancing is called locality-prioritised load balancing.

Locality-prioritised load balancing works using the following algorithm:

![](https://cdn-images-1.medium.com/max/2000/1*KIBHRTHyNmIqkAzclpRtIw.png)

To enable locality-prioritised load balancing, we need to have a virtual service and a destination rule with an outlier policy. The outlier policy checks whether the pods are healthy and makes routing decisions:

<iframe src="https://medium.com/media/67ebcf5df862ccb4805f617bc8930948" frameborder=0></iframe>

Run a test from the sleep pod on us-central1-a to the NGINX service:

<iframe src="https://medium.com/media/4cea5aeef9b05c0afe288a2d8a9a2efb" frameborder=0></iframe>

As we see, all requests are going to nginx-v1.

Run a test from the sleep pod on us-central1-f to the NGINX service:

<iframe src="https://medium.com/media/c5897f482cb5d38aa589ddb1f2258f2e" frameborder=0></iframe>

And we see that all requests are going to nginx-v2. This shows that locality-prioritised load balancing is working correctly!

Let’s try something different. Open a duplicate terminal and run the command below:

    $ for i in {1..100}; do   kubectl exec -it $SLEEP_ZONE_2 -c sleep -n frontend -- sh -c 'curl  [http://nginx.backend:8000'](http://nginx.backend:8000'); done

While the command above is running, delete the nginx-v2 deployment in the other terminal:

    $ kubectl delete deployment nginx-v2 -n backend
    deployment.apps "nginx-v2" deleted

Switch back to the first terminal and you should see the traffic going to nginx-v1 after we deleted the nginx-v2 deployment:

<iframe src="https://medium.com/media/2535b0a4615b40334446c7332458e00b" frameborder=0></iframe>

## Locality-Weighted Load Balancing

Most use cases work well with locality-prioritised load balancing. However, there might be certain use cases where you want to split traffic into multiple zones. You may not want to overload one zone if all requests are coming from a single zone. A typical use case can be a significant user base located in a particular city. You can use locality-weighted load balancing for such use cases.

![Locality-weighted load balancing](https://cdn-images-1.medium.com/max/2000/1*e2mrQcSo_zym379j8PBO0g.png)*Locality-weighted load balancing*

Recreate the nginx-v2 deployment that we deleted in the previous section by applying the relevant YAML.

Then apply the YAML below to:

* Route 80% of the traffic from us-central1-a to us-central1-a and 20% to us-central1-f.

* Route 20% of the traffic from us-central1-f to us-central1-a and 80% to us-central1-f.

<iframe src="https://medium.com/media/6fabde7717390d23bc47b0a448750aec" frameborder=0></iframe>

Run a test from the sleep pod on us-central1-a to the NGINX service:

<iframe src="https://medium.com/media/0cac0644b37e14dc08a41e07c1478a74" frameborder=0></iframe>

And we see that 20% of the traffic is going to nginx-v2 and 80% to nginx-v1.

Now let’s do the reverse. Run a test from the sleep pod on us-central1-f to the NGINX service:

<iframe src="https://medium.com/media/cd734fe85f95fe7ecbeee3e7b7fe24f7" frameborder=0></iframe>

Two out of ten times, we get the response from nginx-v1.

What would happen if we deleted the nginx-v2 deployment while running a test? Let’s find out.

Open a duplicate terminal window and run the following:

    $ for i in {1..100}; do   kubectl exec -it $SLEEP_ZONE_2 -c sleep -n frontend -- sh -c 'curl  [http://nginx.backend:8000'](http://nginx.backend:8000'); done

While the command above is running, delete the nginx-v2 deployment in the other window:

    $ kubectl delete deployment nginx-v2 -n backend
    deployment.apps "nginx-v2" deleted

Switch back to the terminal and you will see that all traffic is now flowing to nginx-v1 after we have deleted the nginx-v2 deployment:

<iframe src="https://medium.com/media/3eabb401ecabfeeb192d3f4a584247d1" frameborder=0></iframe>

## Conclusion

Thanks for reading! I hope you enjoyed the article.

Istio also provides a failover setup if you are running a multi-regional Kubernetes cluster. However, this is not in the scope of this article.

Istio has revolutionised the way we look at microservices, and it provides you with the type of great control that organisations had with traditional infrastructure.

The next story is “[How to Use Istio to Inject Faults to Troubleshoot Microservices in Kubernetes](https://medium.com/better-programming/how-to-use-istio-to-inject-faults-to-troubleshoot-microservices-in-kubernetes-108250a85abc),” so see you there!
