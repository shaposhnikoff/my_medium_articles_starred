
# How to Visualise Your Istio Service Mesh on Kubernetes

Use Prometheus and Grafana to visualise the metrics of your microservices

![Photo by [Brooke Cagle](https://unsplash.com/@brookecagle?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/collections/353844/work?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/10256/1*pPM9DzL3h8tUn1_U4gR9LA.jpeg)*Photo by [Brooke Cagle](https://unsplash.com/@brookecagle?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/collections/353844/work?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

If you’re running [Istio](https://istio.io/) to manage your microservices within Kubernetes, collecting and visualising your metrics is one of the key features it provides. It gives you a lot of control and power over your mesh and allows you to understand your microservices better.

It not only provides your operations team with useful insights to troubleshoot issues but also provides your Security team with valuable data. That helps them do things like hooking your mesh with intrusion-detection software to secure your application further.

This story is a follow up to “[How to Use Istio to Inject Faults to Troubleshoot Microservices in Kubernetes](https://medium.com/better-programming/how-to-use-istio-to-inject-faults-to-troubleshoot-microservices-in-kubernetes-108250a85abc).” Today, let’s discuss collecting and visualising the Istio service mesh using [Prometheus](https://prometheus.io/) and [Grafana](https://grafana.com/).

## How Does Istio Collect Metrics?

If you’re following the series, you should be aware that Istio uses Envoy proxies as sidecars to your microservice containers.

Since all traffic flows through these proxies, they send telemetry data to Prometheus, which can be stored and visualised using tools such as Grafana.

You can also export your metrics to tools such as the [ELK stack](https://www.elastic.co/what-is/elk-stack) or a security scanner to gain further insights from the data and use that for future analysis and machine learning.

## Prerequisites

We’ll use the Bookinfo application in this task as well and generate some traffic. We’ll then query metrics using Prometheus and visualise telemetry data using Grafana.

Ensure you’re running a Kubernetes cluster and have Istio installed on top of it. Follow the “[Getting Started with Istio on Kubernetes](https://medium.com/better-programming/getting-started-with-istio-on-kubernetes-e582800121ea)” guide, and ensure you’ve completed all of the tasks from the guide and have deployed the Bookinfo application.

## Collecting Telemetry Data

The ratings-v2 service can connect with a back-end MongoDB database server. Let us use that so we can simulate a realistic front end–back end situation.

Install the ratings-v2 service:

    $ kubectl apply -f samples/bookinfo/platform/kube/bookinfo-ratings-v2.yaml
    serviceaccount/bookinfo-ratings-v2 created
    deployment.apps/ratings-v2 created

Now install the mongodb service:

    $ kubectl apply -f samples/bookinfo/platform/kube/bookinfo-db.yaml
    service/mongodb created
    deployment.apps/mongodb-v1 created

Now, let’s define the destination rules so we can route traffic according to selected labels. That is the same destination rule we’ve applied in previous articles.

    $ kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
    destinationrule.networking.istio.io/productpage created
    destinationrule.networking.istio.io/reviews created
    destinationrule.networking.istio.io/ratings created
    destinationrule.networking.istio.io/details created

The next step is to expose the microservices using virtual services. Let’s first have a look at the YAML file

<iframe src="https://medium.com/media/f4478eba81ae211a31e3cb6c3153fe41" frameborder=0></iframe>

As you see in the YAML, all reviews traffic should flow to reviews-v3 and all ratings traffic to ratings-v2.

Apply the virtual service by running the following:

    $ kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-db.yaml
    virtualservice.networking.istio.io/reviews created
    virtualservice.networking.istio.io/ratings created

## Send Traffic

The next step is to send some traffic through the service so we can check if Istio is collecting the metrics in the next section.

Visit the product page, and refresh your browser multiple times to generate some traffic.

![Product page](https://cdn-images-1.medium.com/max/3840/1*PA_6m18V0UvjxUq5gLDdLw.png)*Product page*

## Query Metrics Using Prometheus

Expose Prometheus through a port-forwarding proxy.

    $ kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=prometheus -o jsonpath='{.items[0].metadata.name}') 9090:9090 &
    [1] 910
    $ Forwarding from 127.0.0.1:9090 -> 9090

Now, open the link http://127.0.0.1:9090 through your browser.

![Prometheus dashboard](https://cdn-images-1.medium.com/max/3840/1*Dn4BiedqxoKu49U51q3fEQ.png)*Prometheus dashboard*

If you understand the Prometheus Query Language, we can search on a particular metric by just typing the name of the parameter.

Let’s start by typing istio_tcp_connections_opened_total.

![istio_tcp_connections_opened_total](https://cdn-images-1.medium.com/max/3840/1*GJbZH4gOqcnOynRoKPoGQw.png)*istio_tcp_connections_opened_total*

What about istio_tcp_connections_closed_total?

![istio_tcp_connections_closed_total](https://cdn-images-1.medium.com/max/3840/1*_mWRZSy8qezVokCYEJsqsg.png)*istio_tcp_connections_closed_total*

Let’s have a look at the total requests using istio_requests_total, but this time, we’ll have a look at the graphic view. Click on the Graph tab, type the metric on the search box, and click on Execute

![istio_requests_total](https://cdn-images-1.medium.com/max/3796/1*xzTC99TmOXKfeR1qiLCAsQ.png)*istio_requests_total*

Let’s now try some advanced queries. How about all requests to the reviews-v3 microservice?

    istio_requests_total{destination_service="reviews.default.svc.cluster.local", destination_version="v3"}

![istio_requests_total for reviews-v3](https://cdn-images-1.medium.com/max/3802/1*FsVdwEpRpnN_bgrKckmGRw.png)*istio_requests_total for reviews-v3*

And the rate of requests to the productpage microservice?

    rate(istio_requests_total{destination_service=~"productpage.*", response_code="200"}[5m])

![Rate of istio_requests_total for the product page](https://cdn-images-1.medium.com/max/3802/1*LEOVEMxvBVl86DKpDKynyw.png)*Rate of istio_requests_total for the product page*

Well, as you see, we’re successfully able to collect and query telemetry metrics using Prometheus.

## Visualise Metrics Using Grafana

For a technical person, querying metrics through Prometheus might seem enough. Still, if you want someone to visualise the data intuitively, especially the operations team and the people who want to analyse the data, we can use a data-visualising tool such as Grafana.

Expose Grafana through a port-forwarding proxy.

    kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}') 3000:3000 &

Open the URL from the browser.

![Grafana dashboards](https://cdn-images-1.medium.com/max/3840/1*rZdKYCkzy1T4554Xb4B4pQ.png)*Grafana dashboards*

Select the Istio folder, and as you see, you already get a lot of preconfigured Grafana dashboards out of the box when your install Istio. You can use the dashboard to monitor every Istio component and your microservices.

You can also customise and create new dashboards according to your requirements.

If you open the Grafana service dashboard, you can visualise metrics related to all of your services. You can also filter specific services to see their metrics.

Now generate some traffic by refreshing the product page multiple times, and you should see the Grafana dashboard start showing metrics.

![Istio Service Dashboard](https://cdn-images-1.medium.com/max/3840/1*XREazJF1Gtf3B-XDZyNL_g.png)*Istio Service Dashboard*

If you want to understand the workload, there’s an in-built workload dashboard in Grafana — open the dashboard to see the workload graph.

![Istio Workload Dashboard](https://cdn-images-1.medium.com/max/3840/1*6W9vDdzrL2zFGUhyBRR57g.png)*Istio Workload Dashboard*

Explore the other dashboards to see multiple visualisations and metrics you can collect and monitor using Istio.

Congratulations! You’ve completed the activity of visualising your microservices’ metrics using Prometheus and Grafana.

## Conclusion

Thanks for reading through! I hope you enjoyed the article. In the next part, we will discuss “[How to Harden Your Microservices on Kubernetes Using Istio](https://medium.com/better-programming/how-to-harden-your-microservices-on-kubernetes-using-istio-29c23dd90670)” so see you in the next story!
