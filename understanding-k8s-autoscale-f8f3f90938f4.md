Unknown markup type 10 { type: [33m10[39m, start: [33m219[39m, end: [33m230[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m346[39m }

# Understanding k8s AutoScale

An HPA introduction

## What’s autoscaling?

Auto-scaling is a way to automatically increase or decrease the number of computing resources that are being assigned to your application based on your needs at any given time. It emerged from cloud computing technology, which revolutionized the way computer resources are allocated, enabling the creation of a fully scalable server in the cloud. When an application needs more computing power, you can launch additional features on-demand and use them for as long as you want.

## What’s the HPA?

HPA or Horizontal Pod Autoscaler is the autoscaling feature explained before but for Kubernetes pods. HPA offers the following advantages: economy, automatic sizing can offer longer uptime and more availability in cases where production workloads are variable and unpredictable. Automatic sizing differs from having a fixed amount of pods in that it responds to actual usage patterns and therefore reduces the potential disadvantage of having few or many pods for the traffic load. For example, if traffic is usually less at midnight, a static scale solution can schedule some pods to sleep at night, on the other hand, it can better handle unexpected traffic spikes.

## Requirements for HPA

## 1. Metrics Server

![](https://cdn-images-1.medium.com/max/3200/0*EPf8NQd7aq74X8Mj)
> [https://github.com/kubernetes-sigs/metrics-server](https://github.com/kubernetes-sigs/metrics-server)

Metrics Server is a scalable, efficient source of container resource metrics for Kubernetes built-in autoscaling pipelines.

Metrics Server collects resource metrics from Kubelets and exposes them in Kubernetes API server through [Metrics API](https://github.com/kubernetes/metrics) for use by [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) and [Vertical Pod Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler). Metrics API can also be accessed by kubectl top, making it easier to debug autoscaling pipelines.

Metrics Server is not meant for non-autoscaling purposes. For example, don’t use it to forward metrics to monitoring solutions, or as a source of monitoring solution metrics.

**Metrics Server Installation:**

    kubectl apply -f [https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml](https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml)

**Installation for KOPS:**

If you are using the kops to manager your cluster, you need to enable a specific configuration to enable the metrics-sever:
> [https://github.com/kubernetes/kops/tree/master/addons/metrics-server](https://github.com/kubernetes/kops/tree/master/addons/metrics-server)

**Validate Metrics-Server installation:**

After install metrics server the kubectl top command will be available on the cluster to use, this command gets the current metrics of the pods and nodes, if the command not working, review the metrics server installation.
> *kubectl top node*
> *kubectl top pod*

![](https://cdn-images-1.medium.com/max/2000/1*svV7QpJRwHa9VAVstUPBfw.png)

## 2. Cluster Auto-Scaler

When the HPA controller increases the number of pod replicas, we need to have several nodes (resources) that support these new replicas, the cluster auto-scaler is responsible to increase the number of nodes to supports this demand or decrease the number of nodes when the pods are subtilized.

Cluster Autoscaler is a tool that automatically adjusts the size of the Kubernetes cluster when one of the following conditions is true:

* some pods failed to run in the cluster due to insufficient resources,

* there are nodes in the cluster that have been underutilized for an extended period and their pods can be placed on other existing nodes.
> [https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler)

![](https://cdn-images-1.medium.com/max/2000/0*UFLnLvGXFaSG_wBG)

### Cluster Auto-Scaler Installation:

Supported cloud providers:

* GCE [https://kubernetes.io/docs/concepts/cluster-administration/cluster-management/](https://kubernetes.io/docs/concepts/cluster-administration/cluster-management/)

* GKE [https://cloud.google.com/container-engine/docs/cluster-autoscaler](https://cloud.google.com/container-engine/docs/cluster-autoscaler)

* AWS [https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md)

* Azure [https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/azure/README.md](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/azure/README.md)

* Alibaba Cloud [https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/alicloud/README.md](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/alicloud/README.md)

* OpenStack Magnum [https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/magnum/README.md](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/magnum/README.md)

* DigitalOcean [https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/digitalocean/README.md](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/digitalocean/README.md)

**Validate Cluster Auto-scaler Installation:**

Create a deployment and increase the number of replicas to more than resources available, if it’s working the cluster auto-scaler will create new nodes, decrease the number of replicas, and after a time the nodes will be removed.

## 3. Configure the Resources Requests/Limits and Liveness/Readiness Probe
> Is very important to configure the resources requests and limits for your application, because the HPA is based on the CPU percentage usage. [https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
> Is very important to configure the livenesss and readiness probe, is important because the HPA is based on the running pods to set a new desired if the pods are up but the applications are not running the HPA can do problems. [https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

## 4. Understanding the complete flow

![](https://cdn-images-1.medium.com/max/2000/0*d-LMnvOq16QCo4-r)

1. Metrics server takes the aggregated metrics from the current pods and sends them to the kubernetes API when requested

1. The HPA controller checks every 15 seconds by default, and if the values fall within the rule determined in the HPA it increases or decreases the number of pods

1. In the case of scale-up, the kubernetes scheduler will allocate the pods in the nodes that have available resources

1. If you don’t have any resources available, the cluster auto-scaler will check that you don’t have any resources available and will increase the number of nodes needed to supply the pods that are currently scheduled.

1. If the rule is scaledown, the HPA will decrease the number of replicas

1. The cluster auto-scaler cluster verifying that it has little use of nodes reallocates the pods that can be relocated to other nodes and remove the nodes (scale-down).

## Horizontal Pod Auto-Scaler

![](https://cdn-images-1.medium.com/max/2678/0*H_TuFHUKQjDzsnjk)
> [https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)

HPA is used to automatically scale the number of pods on deployments, replicasets, statefulsets or a set of them, based on observed usage of CPU, Memory, or using custom-metrics. Automatic scaling of the horizontal pod does not apply to objects that cannot be scaled, for example, DaemonSets.

The Horizontal Pod Autoscaler is implemented as a control loop, with a period controlled by the — horizontal-pod-autoscaler-sync-period flag of the controller manager (with a default value of 15 seconds). During each period, the controller manager consults resource usage based on the metrics specified in each HorizontalPodAutoscaler definition. The controller manager obtains metrics from the Resource Metrics API (for resource metrics per pod) or the Custom Metrics API (for all other metrics).

The HPA does this operation below to calculate the number of desired replicas:
> desiredReplicas = ceil[currentReplicas * ( currentMetricValue / desiredMetricValue )]

![](https://cdn-images-1.medium.com/max/3200/0*5zYNBt7tkGcRV6_3)

### The HPA Manifest:

    **apiVersion**: autoscaling/v2beta2
    **kind**: HorizontalPodAutoscaler
    **metadata**:
      **name**: php-apache
    **spec**:
      **scaleTargetRef**:
        **apiVersion**: apps/v1
        **kind**: Deployment
        **name**: php-apache
      **minReplicas**: 1
      **maxReplicas**: 10
      **metrics**:
      - **type**: Resource
        **resource**:
          **name**: cpu
          **target**:
            **type**: Utilization
            **averageUtilization**: 50

In this example, we scale up the number of replicas on the deployment php-apache when the CPU average of the all running pods of this application is equal or higher than 50%, and decrease the number of the replicas when the CPU Average is less than 50%.

**When we use the HPA we need to remove the number of replicas of the deployment, pod, replicaset. Because the number of replicas is set by the HPA Controller.**

For scale using the kubectl:
> kubectl autoscale deployment php-apache — cpu-percent=50 — min=1 — max=10

For verifying the HPA:
> kubectl get hpa php-apache

For describing the HPA:
> kubectl describe hpa php-apache

## TroubleShooting:
> If the kubectl get hpa command show a status **unknow**, we need to verify the metrics-server, because the HPA controller cannot getting the metrics
> If the pods don’t scale-up, if the kubectl describe pods show the status **FailedScheduling nodes not available**, we need to verify the Cluster-Autoscaler.

## References:
[**Horizontal Pod Autoscaler Walkthrough**
*Horizontal Pod Autoscaler automatically scales the number of pods in a replication controller, deployment, replica set…*kubernetes.io](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
[**kubernetes-sigs/metrics-server**
*Metrics Server is a scalable, efficient source of container resource metrics for Kubernetes built-in autoscaling…*github.com](https://github.com/kubernetes-sigs/metrics-server)
[**kubernetes/autoscaler**
*Cluster Autoscaler is a tool that automatically adjusts the size of the Kubernetes cluster when one of the following…*github.com](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler)
