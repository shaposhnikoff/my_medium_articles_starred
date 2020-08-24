
# Kubernetes Monitoring 101 — Core pipeline & Services Pipeline

Understanding Kubernetes monitoring pipeline(s) is essential to help you diagnose run-time problems and to manage the scale of your pods, and cluster. Monitoring is one of these areas that are evolving very rapidly inside Kubernetes. It has a lot of pieces that are still in the influx and hence some confusion. My goal as I hope in this article is to clarify it a bit and to give you a good starting point.

Kubernetes has two monitoring pipelines: (1) **The core metrics pipeline**, which is an integral part of Kubernetes and always installed with all distributions, and (2) **The services monitoring (non-core) pipeline**, which is a separate pipeline, and Kubernetes has no or limited dependency on. Keep reading to learn why :)

## Core Monitoring Pipeline

Sometimes is referred to as the [resource metrics pipeline](https://pxlme.me/6GomlbWB). The core monitoring pipeline is installed with every distribution. It provides enough details to other components inside the Kubernetes cluster to run as expected, such as the scheduler to allocate pods and containers, HPA and VPA to take proper decisions scaling pods.

![The workflow of the core monitoring pipeline](https://cdn-images-1.medium.com/max/4448/1*8JxjlAhc7mEw-c26zQIpsw.png)*The workflow of the core monitoring pipeline*

The way it works is relatively simple:

1. CAdvisor collects metrics about containers and nodes that on which it is installed. Note: CAdvisor is installed by default on all cluster nodes

1. Kubelet exposes these metrics (default is one-minute resolution) through Kubelet APIs.

1. Metrics Server discovers all available nodes and calls Kubelet API to get containers and nodes resources usage.

1. Metrics Server exposes these metrics through Kubernetes aggregation API.

**A few helpful points**

* Kubelet cannot run without CAdvisor. If you try to uninstall it or stop it, the cluster’s behavior will become unpredictable.

* Even though Heapster “soon to be deprecated” is currently dependent on CAdvisor, but CAdvisor is not going away anytime soon.

## Services Monitoring Pipeline

Services pipeline in abstract terms is relatively simple. Confusion usually comes from the plethora of services, agents that you can mix and match to get your pipeline up and running. Also, you can blame Heapster for that :)

Services Monitoring Pipeline consists of three main components: (1) Collection agent, (2) Metrics Server, and (3) Dashboards. I’ll not talk about alerting because it has lots of interesting twists :) I plan to discuss alerting in a separate article.

![The typical workflow of the services monitoring pipeline](https://cdn-images-1.medium.com/max/4448/1*AbZg3OPWAuHuJNonhj7Euw.jpeg)*The typical workflow of the services monitoring pipeline*

Below is the typical workflow, including most common components

1. Monitoring agent collects node metrics. cAdvisor collects containers and pods metrics.

1. Monitoring Aggregation service collects data from its own agent and cAdvisor.

1. Data is stored in the monitoring system’s storage.

1. Monitoring aggregation service exposes metrics through APIs and dashboards.

**A Few Notes:**

* [Prometheus](https://pxlme.me/Cr_NFw-N) is the official monitoring server sponsored and incubated by CNCF. It [integrates directly with cAdvisor](https://pxlme.me/D4f1AwHn). You don’t need to install a 3rd party agent to retrieve additional metrics about your containers. However, if you need deeper insights about each node, you need to install an agent of your choice — see [Prometheus integrations and third-party exporters page.](https://pxlme.me/OX7__rQz)

* Almost all monitoring systems piggyback on Kubernetes scheduling and orchestration. For example, their agents are installed as DeomonSets and depend on Kubernetes scheduler to have an instance scheduled on each node.

* Most monitoring agents depend on Kubelet to collect container relevant metrics, which in turn depends on cAdvisor. Very few agents collect container relevant details independently.

* Most monitoring aggregation services depend on agents pushing metrics to them. Prometheus is an exception. It pulls metrics out of the installed agents.

## What should you consider in Kubernetes Services Pipeline?

Ideal Services pipeline depends on two main factors: (1) collection of relevant metrics, (2) Awareness of continuous changes inside kubernetes cluster.

A good pipeline should focus on collecting relevant metrics. There are plenty of agents that can collect OS and process-level metrics. But you will find very few out there that can collect details about containers running at a given node, such as the number of running containers, container state, docker engine metrics, etc. cAdvisor is the best agent IMO for this job so far.

Awareness of continuous changes means that the monitoring pipeline is aware of different pods, containers instances and can relate them to their parent entities, i.e. Deployment, Statefulsets, Namespace, etc. It also means that the metrics server is aware of system-wide metrics that should be visible to users, such as the number of pending pods, nodes status, etc.

## What about Metrics Visualization?

You can visualize metrics in many different ways. The most common open source tool that easily integrates with Prometheus is[ Grafana](https://prometheus.io/docs/visualization/grafana/). The challenges you will face though is building proper dashboards to monitor the right metrics. That said, you should have dashboards monitoring the following:

* **Cluster level capacity utilization**, this shows how much CPU memory being across the whole cluster and per node.

* **Kubernetes Orchestration Metrics**, which tracks the status of your pods and containers inside your cluster. This includes the distribution of pods among nodes.

* **Kubernetes Core Services**, which visualizes the status of critical services such as CoreDNS, Calico, and any other service important for networking, storage, and pods scheduling.

* Application Specific Metrics, which tracks the status of your apps. They should reflect your users’ experience and business critical metrics.

**Note**: You can get started with [this Grafana template dashboards](https://pxlme.me/9wR_GkOT) for Kubernetes.
> Grafana is not best suited for alerting. I see a lot of teams depend on it to create alerting rules. However, it is not as reliable and comprehensive as Prometheus alerting manager.

<iframe src="https://medium.com/media/ed14b3d4c661e490c58836817d79e587" frameborder=0></iframe>

## Changes To Watch For

### Heapster is Going Away

Heapster is currently causing some confusion given that it is used to show both core pipeline metrics and services metrics. In reality, you can remove Heapster and nothing bad will happen to the core Kubernetes scheduling and orchestration scenarios. It was the default monitoring pipeline and I guess it still is the default in a lot of distributions. But you don’t have to use it at all.

So, the Kubernetes community wanted to make the separation clearer between core and services monitoring pipelines. Hence, Heapster will be deprecated and replaced by the Metrics Server (MS) as the main source of aggregated core metrics. Think of the MS as a trimmed down version of Heapster. Major immediate changes are: (1) No historical data or queries, (2) eliminating a lot of container-specific metrics, pod focus metrics only. Metrics Server is meant to provide core metrics that are needed for core Kubernetes scenarios, such as autoscaling, scheduling, etc., likely to take place in 2019 releases.

### Metrics Server Will Get More Cool Features

Infrastore will store Metric Server historical data with a support of simple SQL-like queries. It will support initially metrics collected by the Metrics Server. My guess, because Kubernetes community love extensibility, they will make it extensible and allow custom metrics to be added to the Metrics Server and its store.

## TL;DR

* You need to differentiate between core metrics pipeline and the services pipeline.

* Heapster will be deprecated. you should pick the best pipeline that works for your needs.

* The community official tool is Prometheus. You can use a variety of other open source or commercial tools, but I recommend to get started with Prometheus before you decide with any other tools that may cost you a lot down the road.

* Use [Grafana for metrics visualization](https://pxlme.me/dRJpGP5G). But I wouldn’t recommend using it for alerting.
