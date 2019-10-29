Unknown markup type 10 { type: [33m10[39m, start: [33m106[39m, end: [33m116[39m }

# Highly Available and Scalable Elasticsearch on Kubernetes

Highly Available and Scalable Elasticsearch on Kubernetes

In the [previous](https://medium.com/@thakur.vaibhav23/scaling-mongodb-on-kubernetes-32e446c16b82) post we learned about Stateful Sets by scaling a MongoDB Replica Set. In this post we will be orchaestrating a HA Elasticsearch cluster ( with different Master, Data and Client nodes ) along with ES-HQ and Kibana

**Prerequisites:**

1. Basic knowledge of Elasticsearch, its Node types and their roles.

1. Running Kubernetes cluster with alteast 3 nodes (atleast 4C 4GB ).

1. Working knowledge of Kibana.

**Deployment Architecture**

![](https://cdn-images-1.medium.com/max/2022/1*TG61ojlzv2v-akGqwws2Ew.jpeg)

* Elasticsearch **Data Node Pods** are deployed as a **Stateful Set** with a headless service to provide **Stable Network Identities.**

* Elasticsearch **Master Node Pods** are deployed as a **Replica Set** with a headless service which will help in **Auto-discovery.**

* Elasticsearch **Client Node Pods** are deployed as a Replica Set with a internal service which will allow access to the Data Nodes for **R/W requests.**

* **Kibana and ElasticHQ Pods** are deployed as Replica Sets with Services accessible **outside the Kubernetes cluster** but still **internal to your Subnetwork** (not publicly exposed unless otherwise required ).

* **HPA(Horizonal Pod Auto-scaler)** deployed for** Client Nodes** to enable auto-scaling under high load.

Important things to keep in mind:

1. Setting **ES_JAVA_OPTS** env variable.

1. Setting **CLUSTER_NAME **env variable.

1. Setting **NUMBER_OF_MASTERS **(to avoid split-brain problem)** **env variable for master deployment. In case of 3 masters we have set it 2.

1. Setting correct **Pod-AntiAffinity** policies among similar pods in order to ensure HA if a worker node fails.

Let’s jump right at deploying these services to our GKE cluster.

<iframe src="https://medium.com/media/29f3366f3b6692a4938e12ad5b959b06" frameborder=0></iframe>

    root$ kubectl apply -f es-master.yml
    root$ kubectl -n elasticsearch get all
    NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE

    deploy/es-master   3         3         3            3           32s

    NAME                      DESIRED   CURRENT   READY     AGE

    rs/es-master-594b58b86c   3         3         3         31s

    NAME                            READY     STATUS    RESTARTS   AGE

    po/es-master-594b58b86c-9jkj2   1/1       Running   0          31s

    po/es-master-594b58b86c-bj7g7   1/1       Running   0          31s

    po/es-master-594b58b86c-lfpps   1/1       Running   0          31s

    NAME                          TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE

    svc/elasticsearch-discovery   ClusterIP   None         <none>        9300/TCP   31s

It is interesting to follow the logs of any of the master-node pods to witness the master election among them and then later on when new data and client nodes are added.

    root$ kubectl -n elasticsearch logs -f po/es-master-594b58b86c-9jkj2 | grep ClusterApplierService

    [2018-10-21T07:41:54,958][INFO ][o.e.c.s.ClusterApplierService] [es-master-594b58b86c-9jkj2] **detected_master {es-master-594b58b86c-bj7g7**}{1aFT97hQQ7yiaBc2CYShBA}{Q3QzlaG3QGazOwtUl7N75Q}{10.9.126.87}{10.9.126.87:9300}, **added {{es-master-594b58b86c-lfpps}{wZQmXr5fSfWisCpOHBhaMg}**{50jGPeKLSpO9RU_HhnVJCA}{10.9.124.81}{10.9.124.81:9300},**{es-master-594b58b86c-bj7g7}{1aFT97hQQ7yiaBc2CYShBA}**{Q3QzlaG3QGazOwtUl7N75Q}{10.9.126.87}{10.9.126.87:9300},}, reason: apply cluster state (from master [master {es-master-594b58b86c-bj7g7}{1aFT97hQQ7yiaBc2CYShBA}{Q3QzlaG3QGazOwtUl7N75Q}{10.9.126.87}{10.9.126.87:9300} committed version [3]])

It can be seen that es-master pod named **es-master-594b58b86c-bj7g7 **was elected as the master and other 2 pods added to it and each other.

The headless service named **elasticsearch-discovery** is set by default as an env variable in the docker image and is used for discovery among the nodes. This can ofcourse be overridden.

Similarly, we can deploy the **data and client** nodes. The configs can be found below:

**Data Nodes Deployment:**

<iframe src="https://medium.com/media/5f1038ace45c12d62f52b93ced918396" frameborder=0></iframe>

The headless service in case of data nodes provides **stable network identities **to the nodes and also help in **data transfer** among them.

It is important to **format the persistent volume** before attaching it to the pod. This can be done by specifiying the volume type when creating storage class. We can also set the flag to **allow volume expansion** on the fly. More can be read about that [here](https://kubernetes.io/blog/2018/07/12/resizing-persistent-volumes-using-kubernetes/).

    ...
    parameters:  
      type: pd-ssd  
      fsType: xfs
    allowVolumeExpansion: true
    ...

**Client Nodes Deployment:**

<iframe src="https://medium.com/media/22b9b47008144e09ba4629d062f54c1b" frameborder=0></iframe>

The service deployed here is to access the ES Cluster from outside the Kubernetes cluster but still internal to our subnet. The annotation “***cloud.google.com/load-balancer-type: Internal” ***ensures this.

However, if the application reading/writing to our ES cluster is deployed within the cluster then the ElasticSearch service can be accessed by [**http://elasticsearch.elasticsearch:9200](http://elasticsearch.elasticsearch:9200) .**

Once you create these 2 deployments, the newly created client and data nodes will be automatically added to the cluster. (Observe logs for the master pod)

    root$ kubectl apply -f es-data.yml
    root$ kubectl -n elasticsearch get pods -l role=data
    NAME        READY     STATUS    RESTARTS   AGE

    es-data-0   1/1       Running   0          48s

    es-data-1   1/1       Running   0          28s

    --------------------------------------------------------------------
    root$ kubectl apply -f es-client.yml 
    root$ kubectl -n elasticsearch get pods -l role=client

    NAME                         READY     STATUS    RESTARTS   AGE

    es-client-69b84b46d8-kr7j4   1/1       Running   0          47s

    es-client-69b84b46d8-v5pj2   1/1       Running   0          47s

    --------------------------------------------------------------------

    root$ kubectl -n elasticsearch get all
    NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE

    deploy/es-client   2         2         2            2           1m

    deploy/es-master   3         3         3            3           9m

    NAME                      DESIRED   CURRENT   READY     AGE

    rs/es-client-69b84b46d8   2         2         2         1m

    rs/es-master-594b58b86c   3         3         3         9m

    NAME                   DESIRED   CURRENT   AGE

    statefulsets/es-data   2         2         3m

    NAME                            READY     STATUS    RESTARTS   AGE

    po/es-client-69b84b46d8-kr7j4   1/1       Running   0          1m

    po/es-client-69b84b46d8-v5pj2   1/1       Running   0          1m

    po/es-data-0                    1/1       Running   0          3m

    po/es-data-1                    1/1       Running   0          3m

    po/es-master-594b58b86c-9jkj2   1/1       Running   0          9m

    po/es-master-594b58b86c-bj7g7   1/1       Running   0          9m

    po/es-master-594b58b86c-lfpps   1/1       Running   0          9m

    NAME                          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE

    svc/elasticsearch             LoadBalancer   10.9.121.160 10.9.120.8     9200:32310/TCP   1m

    svc/elasticsearch-data        ClusterIP   None           <none>        9300/TCP         3m

    svc/elasticsearch-discovery   ClusterIP   None           <none>        9300/TCP         9m

    --------------------------------------------------------------------

    #Check logs of es-master leader pod

    root$ kubectl -n elasticsearch logs po/es-master-594b58b86c-bj7g7 | grep ClusterApplierService

    [2018-10-21T07:41:53,731][INFO ][o.e.c.s.ClusterApplierService] [es-master-594b58b86c-bj7g7] **new_master {es-master-594b58b86c-bj7g7}**{1aFT97hQQ7yiaBc2CYShBA}{Q3QzlaG3QGazOwtUl7N75Q}{10.9.126.87}{10.9.126.87:9300}, added {{es-master-594b58b86c-lfpps}{wZQmXr5fSfWisCpOHBhaMg}{50jGPeKLSpO9RU_HhnVJCA}{10.9.124.81}{10.9.124.81:9300},}, reason: apply cluster state (from master [master {es-master-594b58b86c-bj7g7}{1aFT97hQQ7yiaBc2CYShBA}{Q3QzlaG3QGazOwtUl7N75Q}{10.9.126.87}{10.9.126.87:9300} committed version [1] source [zen-disco-elected-as-master ([1] nodes joined)[{es-master-594b58b86c-lfpps}{wZQmXr5fSfWisCpOHBhaMg}{50jGPeKLSpO9RU_HhnVJCA}{10.9.124.81}{10.9.124.81:9300}]]])

    [2018-10-21T07:41:55,162][INFO ][o.e.c.s.ClusterApplierService] [es-master-594b58b86c-bj7g7] **added {{es-master-594b58b86c-9jkj2}**{x9Prp1VbTq6_kALQVNwIWg}{7NHUSVpuS0mFDTXzAeKRcg}{10.9.125.81}{10.9.125.81:9300},}, reason: apply cluster state (from master [master {es-master-594b58b86c-bj7g7}{1aFT97hQQ7yiaBc2CYShBA}{Q3QzlaG3QGazOwtUl7N75Q}{10.9.126.87}{10.9.126.87:9300} committed version [3] source [zen-disco-node-join[{es-master-594b58b86c-9jkj2}{x9Prp1VbTq6_kALQVNwIWg}{7NHUSVpuS0mFDTXzAeKRcg}{10.9.125.81}{10.9.125.81:9300}]]])

    [2018-10-21T07:48:02,485][INFO ][o.e.c.s.ClusterApplierService] [es-master-594b58b86c-bj7g7] **added {{es-data-0}**{SAOhUiLiRkazskZ_TC6EBQ}{qirmfVJBTjSBQtHZnz-QZw}{10.9.126.88}{10.9.126.88:9300},}, reason: apply cluster state (from master [master {es-master-594b58b86c-bj7g7}{1aFT97hQQ7yiaBc2CYShBA}{Q3QzlaG3QGazOwtUl7N75Q}{10.9.126.87}{10.9.126.87:9300} committed version [4] source [zen-disco-node-join[{es-data-0}{SAOhUiLiRkazskZ_TC6EBQ}{qirmfVJBTjSBQtHZnz-QZw}{10.9.126.88}{10.9.126.88:9300}]]])

    [2018-10-21T07:48:21,984][INFO ][o.e.c.s.ClusterApplierService] [es-master-594b58b86c-bj7g7] **added {{es-data-1}**{fiv5Wh29TRWGPumm5ypJfA}{EXqKGSzIQquRyWRzxIOWhQ}{10.9.125.82}{10.9.125.82:9300},}, reason: apply cluster state (from master [master {es-master-594b58b86c-bj7g7}{1aFT97hQQ7yiaBc2CYShBA}{Q3QzlaG3QGazOwtUl7N75Q}{10.9.126.87}{10.9.126.87:9300} committed version [5] source [zen-disco-node-join[{es-data-1}{fiv5Wh29TRWGPumm5ypJfA}{EXqKGSzIQquRyWRzxIOWhQ}{10.9.125.82}{10.9.125.82:9300}]]])

    [2018-10-21T07:50:51,245][INFO ][o.e.c.s.ClusterApplierService] [es-master-594b58b86c-bj7g7] **added {{es-client-69b84b46d8-v5pj2}**{MMjA_tlTS7ux-UW44i0osg}{rOE4nB_jSmaIQVDZCjP8Rg}{10.9.125.83}{10.9.125.83:9300},}, reason: apply cluster state (from master [master {es-master-594b58b86c-bj7g7}{1aFT97hQQ7yiaBc2CYShBA}{Q3QzlaG3QGazOwtUl7N75Q}{10.9.126.87}{10.9.126.87:9300} committed version [6] source [zen-disco-node-join[{es-client-69b84b46d8-v5pj2}{MMjA_tlTS7ux-UW44i0osg}{rOE4nB_jSmaIQVDZCjP8Rg}{10.9.125.83}{10.9.125.83:9300}]]])

    [2018-10-21T07:50:58,964][INFO ][o.e.c.s.ClusterApplierService] [es-master-594b58b86c-bj7g7] **added {{es-client-69b84b46d8-kr7j4}**{gGC7F4diRWy2oM1TLTvNsg}{IgI6g3iZT5Sa0HsFVMpvvw}{10.9.124.82}{10.9.124.82:9300},}, reason: apply cluster state (from master [master {es-master-594b58b86c-bj7g7}{1aFT97hQQ7yiaBc2CYShBA}{Q3QzlaG3QGazOwtUl7N75Q}{10.9.126.87}{10.9.126.87:9300} committed version [7] source [zen-disco-node-join[{es-client-69b84b46d8-kr7j4}{gGC7F4diRWy2oM1TLTvNsg}{IgI6g3iZT5Sa0HsFVMpvvw}{10.9.124.82}{10.9.124.82:9300}]]])

**The logs of the leading master pod clearly depict when each node gets added to the cluster. It is extremely useful in case of debugging issues.**

Once all components are deployed we should verify the following:

1. Elasticsearch deployment from inside the kubernetes cluster using a ubuntu container.

    root$ kubectl run my-shell --rm -i --tty --image ubuntu -- bash
    root@my-shell-68974bb7f7-pj9x6:/# curl http://elasticsearch.elasticsearch:9200/_cluster/health?pretty

    {

    "cluster_name" : "my-es",

    "status" : "green",

    "timed_out" : false,

    "number_of_nodes" : 7,

    "number_of_data_nodes" : 2,

    "active_primary_shards" : 0,

    "active_shards" : 0,

    "relocating_shards" : 0,

    "initializing_shards" : 0,

    "unassigned_shards" : 0,

    "delayed_unassigned_shards" : 0,

    "number_of_pending_tasks" : 0,

    "number_of_in_flight_fetch" : 0,

    "task_max_waiting_in_queue_millis" : 0,

    "active_shards_percent_as_number" : 100.0

    }

2. Elasticsearch deployment from outside the cluster using the GCP Internal Loadbalancer IP (in this case 10.9.120.8)

    root$ curl [http://10.9.120.8:9200/_cluster/health?pretty](http://10.9.120.8:9200/_cluster/health?pretty)
    {

    "cluster_name" : "my-es",

    "status" : "green",

    "timed_out" : false,

    "number_of_nodes" : 7,

    "number_of_data_nodes" : 2,

    "active_primary_shards" : 0,

    "active_shards" : 0,

    "relocating_shards" : 0,

    "initializing_shards" : 0,

    "unassigned_shards" : 0,

    "delayed_unassigned_shards" : 0,

    "number_of_pending_tasks" : 0,

    "number_of_in_flight_fetch" : 0,

    "task_max_waiting_in_queue_millis" : 0,

    "active_shards_percent_as_number" : 100.0

    }

3. Anti-Affinity Rules for our ES-Pods

    root$ kubectl -n elasticsearch get pods -o wide 
    NAME                         READY     STATUS    RESTARTS   AGE       IP            NODE

    es-client-69b84b46d8-kr7j4   1/1       Running   0          10m       10.8.14.52   gke-cluster1-pool1-d2ef2b34-t6h9

    es-client-69b84b46d8-v5pj2   1/1       Running   0          10m       10.8.15.53   gke-cluster1-pool1-42b4fbc4-cncn

    es-data-0                    1/1       Running   0          12m       10.8.16.58   gke-cluster1-pool1-4cfd808c-kpx1

    es-data-1                    1/1       Running   0          12m       10.8.15.52   gke-cluster1-pool1-42b4fbc4-cncn

    es-master-594b58b86c-9jkj2   1/1       Running   0          18m       10.8.15.51   gke-cluster1-pool1-42b4fbc4-cncn

    es-master-594b58b86c-bj7g7   1/1       Running   0          18m       10.8.16.57   gke-cluster1-pool1-4cfd808c-kpx1

    es-master-594b58b86c-lfpps   1/1       Running   0          18m       10.8.14.51   gke-cluster1-pool1-d2ef2b34-t6h9

Note that no 2 similar pods are on same node. This ensures HA in case of node failures.

## **Scaling Considerations**

We can deploy** autoscalers **for our client nodes depending upon our CPU thresholds. A sample HPA for client node might look something like this:

    apiVersion: autoscaling/v1
    kind: HorizontalPodAutoscaler
    metadata:
      name: es-client
      namespace: elasticsearch
    spec:
      maxReplicas: 5
      minReplicas: 2
      scaleTargetRef:
        apiVersion: extensions/v1beta1
        kind: Deployment
        name: es-client
    targetCPUUtilizationPercentage: 80

Whenever the autoscaler will kick in, we can watch the new client-node pods being added to the cluster, by observing the logs of any of the master-node pods.

**In case of Data-Node** **Pods** all we have to do it increase the number of replicas using the K8 Dashboard or GKE console. The newly created data node will be automatically added to the cluster and start replicating data from other nodes.

**Master-Node Pods** do not require autoscaling as they only store cluster-state information but in case you want to add more data nodes make sure there are **no even number of master nodes** in the cluster also the environment variable **NUMBER_OF_MASTERS **is updated accordingly.

## Deploying Kibana and ES-HQ

[Kibana](https://www.elastic.co/products/kibana) and [ES-HQ](http://www.elastichq.org)

Kibana is a simple tool to visualize ES-data and ES-HQ helps in Administration and monitoring of Elasticsearch cluster. For our Kibana and ES-HQ deployment we keep the following things in mind

* We provide the name of the ES-Cluster as an environment variable to the docker image

* The service to access the Kibana/ES-HQ deployment is internal to our organisation only i.e. No public IP is created. We use a GCP Internal load balancer.

***Kibana Deployment***

<iframe src="https://medium.com/media/ee2343e63f70f3c6df125196f85b9e34" frameborder=0></iframe>

***ES-HQ Deployment***

<iframe src="https://medium.com/media/1904501ba333f483fd9d1441c9538efa" frameborder=0></iframe>

We can access both these services using the newly created Internal LoadBalancers.

    root$ kubectl -n elasticsearch get svc -l role=kibana

    NAME      TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE

    kibana    LoadBalancer   10.9.121.246   10.9.120.10   80:31400/TCP   1m

    root$ kubectl -n elasticsearch get svc -l role=hq

    NAME      TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE

    hq        LoadBalancer   10.9.121.150   10.9.120.9    80:31499/TCP   1m

Go to [http://<External-Ip-Kibana-Service>/app/kibana#/home?_g=()](http://10.9.120.10/app/kibana#/home?_g=())

![Kibana Dashboard](https://cdn-images-1.medium.com/max/6288/1*x_ohXVXlQDEAb0DTuOf3Og.png)*Kibana Dashboard*

Go to [http://](http://10.9.120.9/#!/clusters/my-es)[<External-Ip-ES-Hq-Service>](http://10.9.120.10/app/kibana#/home?_g=())[/#!/clusters/my-es](http://10.9.120.9/#!/clusters/my-es)

![ElasticHQ Dasboard for Cluster Monitoring and Management](https://cdn-images-1.medium.com/max/6276/1*cP7_RzAAI7cEeRJbKFQzDg.png)*ElasticHQ Dasboard for Cluster Monitoring and Management*

ES is one of the most widely used distributed search and analytics systems, and when used in conjunction with Kubernetes will eliminate key issues around scaling and HA. Also, deploying new ES clusters with Kubernetes takes no time. I hope this blog was useful for you and I really look forward to suggestions for improvements. Feel free to comment or reach out over [L**inkedIn](https://www.linkedin.com/in/vaibhavthakur/)**.

Don’t forget to check out my other posts:

1. [Continuous Delivery pipelines for Kubernetes using Spinnaker](/faun/continuous-delivery-pipeline-for-kubernetes-using-spinnaker-225fe9c9a6e6)

1. [Production Grade Kubernetes Monitoring using Prometheus](https://medium.com/faun/production-grade-kubernetes-monitoring-using-prometheus-78144b835b60)

1. [Scaling MongoDB on Kubernetes](/devopslinks/scaling-mongodb-on-kubernetes-32e446c16b82)

1. [AWS ECS and Gitlab-CI](https://www.linkedin.com/pulse/learners-guide-deploying-docker-ecs-using-gitlab-ci-vaibhav-thakur/)

![](https://cdn-images-1.medium.com/max/2000/0*kWGkiH8mfoHACNpz)

**Join our community Slack and read our weekly Faun topics ⬇**
[**Join a Community of Aspiring Developers.Get must-read articles, learn new technologies for free…**
*Join thousands of developers and IT experts, get must-read articles, chat with like-minded people, get job offers and…*www.faun.dev](https://www.faun.dev/join/?utm_source=medium.com%2Ffaun&utm_medium=medium&utm_campaign=faunmedium)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
