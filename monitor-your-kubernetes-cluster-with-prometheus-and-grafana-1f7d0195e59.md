Unknown markup type 10 { type: [33m10[39m, start: [33m21[39m, end: [33m47[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m30[39m, end: [33m40[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m45[39m, end: [33m55[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m38[39m, end: [33m48[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m53[39m, end: [33m67[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m25[39m, end: [33m61[39m }

# Monitor Your Kubernetes Cluster With Prometheus and Grafana

Using Helm to set up Prometheus and Grafana on your Kubernetes cluster

![Photo by [Charles Deluvio](https://unsplash.com/@charlesdeluvio?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral).](https://cdn-images-1.medium.com/max/10944/0*z2Mt8-HFC6hPzojI)*Photo by [Charles Deluvio](https://unsplash.com/@charlesdeluvio?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral).*

[Prometheus](https://prometheus.io/) and [Grafana](https://grafana.com/) are some of the most popular monitoring solutions for Kubernetes, and they now come as default in most managed clusters such as GKE.

These tools have recently graduated from the Cloud Native Computing Foundation, which means that they are ready for production use and are first-class citizens among open-source monitoring tools.

With [Helm](https://helm.sh/), installing and managing Prometheus and Grafana on your Kubernetes cluster has become much more straightforward. It not only provides you with a hardened, well-tested setup but also provides a lot of preconfigured dashboards to get you started right away.

It also gives you the flexibility to adapt both tools according to your use case, which is a plus. While Prometheus offers the core monitoring capability of collecting and receiving logs, indexing them, and optimising storage, Grafana provides a means for visualising the metrics in graphical dashboards.

In this hands-on demonstration, let’s look at how we can set up an effective monitoring solution for your Kubernetes cluster in minutes.

## Prerequisites

You will need the following:

* A running Kubernetes cluster. We will use a managed GKE cluster in this demo, but it works well on any Kubernetes setup.

* Kubernetes Metrics server pre-installed and configured on your cluster if you are using a self-hosted installation.

## Install Helm

For those who don’t know about Helm, it is a package manager for Kubernetes that allows you to install and manage your Kubernetes applications. It is similar to what apt is to Ubuntu or yum is to Red Hat.

Installing Helm is a piece of cake, as you just need to download the latest release, unpack the contents, and move the Helm binary to your bin directory or set the appropriate path. Helm v3 does not require Tiller, so it is more lightweight and straightforward to manage:

<iframe src="https://medium.com/media/594e623a6a0e0b10142f834d7e2e6b9a" frameborder=0></iframe>

As it’s a fresh install, we need to define the public Kubernetes chart repository in the Helm configuration:

    $ helm repo add stable [https://kubernetes-charts.storage.googleapis.com](https://kubernetes-charts.storage.googleapis.com)

We can now install the Prometheus operator using the Helm chart.

## Install Prometheus Chart

With Helm, you do not need to worry about the internals of writing manifests and wiring things up. Helm provides you with battle-hardened, production-grade setups extensively tested across multiple scenarios and use cases.

We will be using the stable/prometheus-operator chart for this.

It is a great idea to install the Prometheus operator in a separate namespace, as it is easy to manage it this way.

Create a new namespace called monitoring:

    $ kubectl create ns monitoring
    namespace/monitoring created

Install the Prometheus Operator chart in the monitoring namespace of the cluster:

<iframe src="https://medium.com/media/cc4e1f5ecb4842744f765d6c874d3d2a" frameborder=0></iframe>

Let’s check if Helm has rolled out the operator correctly:

<iframe src="https://medium.com/media/fcb713bd9c358ee3dbcf8d1b4e9d0266" frameborder=0></iframe>

As we can see, the pods are all up and running.

## Accessing Prometheus

Let’s now access the Prometheus dashboard and see for ourselves what we get. We will be using the Kubernetes proxy to access it. The Kube proxy allows us to securely tunnel connections to Prometheus using TLS via the Kube API server.

    $ kubectl port-forward -n monitoring prometheus-prometheus-prometheus-oper-prometheus-0 9090
    Forwarding from 127.0.0.1:9090 -> 9090

Now access the Prometheus dashboard on [http://127.0.0.1:9090](http://127.0.0.1:9090):

![](https://cdn-images-1.medium.com/max/3840/1*yrBwQhRvFeaqzFMLmWtyLA.png)

And as we see, Prometheus is up and running. You can use the Prometheus Query Language to query Kubernetes metrics in the old-fashioned geeky way.

For example, let’s look at the time series for the API Server request count:

![](https://cdn-images-1.medium.com/max/3840/1*4OqxEWN_34gw6uvZlVMcnQ.png)

## Accessing the Grafana Dashboard

For the folks who are more into graphs, let’s try to access the Grafana Dashboard. First, we need to locate the Grafana pod and then port forward to access it via the Kube proxy:

    $ kubectl get pod -n monitoring|grep grafana
    prometheus-grafana-85b4dbb556-8v8dw    2/2     Running   0     7m45s
    $ kubectl port-forward -n monitoring prometheus-grafana-85b4dbb556-8v8dw 3000
    Forwarding from 127.0.0.1:3000 -> 3000

Access the Grafana dashboard on [http://127.0.0.1:3000](http://127.0.0.1:3000):

![](https://cdn-images-1.medium.com/max/3840/1*A2eGxbcKj2a063rmq1gEwA.png)

Now, we need the credentials to log into Grafana. For that, we would get the username and password from the secret that Helm installed for us:

    $ kubectl get secret -n monitoring grafana-credentials -o yaml

From the YAML, pick the values of the admin-user and admin-password fields, and base64 decode them to get the plaintext credentials to log into Grafana.

Once logged in, you will get the default dashboard like the one below. As you see, the Helm chart comes with several predefined dashboards that are great for a start. If you like, you can further customise them according to your use case.

![](https://cdn-images-1.medium.com/max/3840/1*j0hPewv_xa5dyiN6UgIGSw.png)

Let’s have a look at the Kubernetes/Compute Resources/Cluster dashboard:

![](https://cdn-images-1.medium.com/max/3840/1*s88KRCelHuV3qB_S84R42w.png)

As you can see, it is giving us a variety of metrics related to the general cluster’s health and utilisation.

Let’s look at the stats for a particular node:

![](https://cdn-images-1.medium.com/max/3840/1*7ubN4WuYVHKEh7-Y8Uc4Tg.png)

This gives us another variety of stats, such as current CPU utilisation, memory utilisation, and their time series for a particular node.

Congratulations! You are all set up and ready to go.

## Conclusion

You can discover a lot of other dashboards that give you valuable insights about the health of your cluster, resource usage patterns of particular application pods, network traffic flowing across the cluster, and much more.

Prometheus and Grafana are powerful tools for monitoring your Kubernetes cluster, and with Helm, they are so much easier to set up as well.

Thanks for reading! I hope you enjoyed the article.
