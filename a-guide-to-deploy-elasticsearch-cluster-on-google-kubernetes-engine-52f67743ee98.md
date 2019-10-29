Unknown markup type 10 { type: [33m10[39m, start: [33m84[39m, end: [33m96[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m28[39m, end: [33m39[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m61[39m, end: [33m81[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m12[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m91[39m, end: [33m108[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m23[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m105[39m, end: [33m110[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m160[39m, end: [33m203[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m103[39m, end: [33m115[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m31[39m, end: [33m53[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m79[39m, end: [33m112[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m99[39m, end: [33m117[39m }

# A Guide to Deploy Elasticsearch Cluster on Google Kubernetes Engine

This is a simple guide that helps you to deploy Elasticsearch cluster on Google Kubernetes Engine. This guide should be relevant on any Kubernetes cluster with a simple tweak on the persistent volume part.

The overall step-by-step is like the following:

1. Enable the persistent volume via [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/).

1. Enable the Elasticsearch node discovery via [Headless Service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services).

1. Deploy the Elasticsearch Cluster via [Stateful Sets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/).

## 1. Persistent Volume

The first step is to create a Storage Class on your cluster. Create new file called storage.yaml with the following content:

    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: ssd
    provisioner: kubernetes.io/gce-pd
    parameters:
      type: pd-ssd
      zone: asia-southeast1-a

Note that the Storage Class provisioner that we use is GCE ( kubernetes.io/gce-pd ) with the following parameters:

1. type: pd-ssd to enable the SSD as persistent disk. You may update the parameters type with type: pd-standard to enable the standard disk as persistent disk

1. zone: asia-southeast1-a for the compute zone. You may update this to your compute zone. You can also use zones parameter for multiple zones like the following: zones: asia-southeast1-a, asia-southeast1-b .

Then enable the storage class using the following command:

    kubectl apply -f storage.yaml

You should be able to see the Storage Class on the [dashboard](https://console.cloud.google.com/kubernetes/storage):

![Storage Class on the my Laniakea Cluster](https://cdn-images-1.medium.com/max/2000/1*W1ne7-zQtj4CY7IcaiUStA.png)*Storage Class on the my Laniakea Cluster*

If you deploy the Elasticsearch Cluster on other than Google Kubernetes Engine, please update the provisioner and the parameters. You can see the available provisioners and parameters in [here](https://kubernetes.io/docs/concepts/storage/storage-classes/#parameters). The rest of this guide will applicable to any Kubernetes Cluster.

## 2. Elasticsearch Node Discovery

The second step is to enable elasticsearch node discovery via headless service. Create new file called service.yaml with the following content:

    apiVersion: v1
    kind: Service
    metadata:
      name: es
      labels:
        service: elasticsearch
    spec:
      clusterIP: None
      ports:
      - port: 9200
        name: serving
      - port: 9300
        name: node-to-node
      selector:
        service: elasticsearch

Then enable the headless service using the following command:

    kubectl apply -f service.yaml

You should be able to see the service on the dashboard:

![ES headless service on the laniakea cluster](https://cdn-images-1.medium.com/max/2604/1*d4G6UfjqDOmz2dtKpUaqyw.png)*ES headless service on the laniakea cluster*

Now, every pod that have label service: elasticsearch should be accessible via $PODNAME.es.default.cluster.local inside a kubernetes cluster. This will helps our elasticsearch nodes to discover each other and form a cluster.

## 3. Elasticsearch Cluster

The last step is to deploy the elasticsearch cluster using the StatefulSet. Create new file called elasticsearch.yaml with the following content:

<iframe src="https://medium.com/media/42d22351e2a3abd9104049cbb29f81c7" frameborder=0></iframe>

to deploy the Elasticsearch cluster, run the following command:

    kubectl apply -f elasticsearch.yaml

It may takes time before the cluster is ready, it depends on the size of the cluster. You can see if the cluster is ready or not by accessing the [workloads dashboard](https://console.cloud.google.com/kubernetes/workload).

If your cluster is ready, you can check if cluster is created or not by accessing one of the elasticsearch node via port-forward:

    kubectl port-forward elasticsearch-0 9200:9200

this will forward all request to [http://localhost:9200](http://localhost:9200) to the elasticsearch-0 node. Then:

    curl [http://localhost:9200/_cluster/state?pretty](http://10.20.8.3:9200/_cluster/state?pretty)

It should show you that the cluster is formed by 5 elasticsearch nodes.

That’s it! now you can enjoy my favorite gif:

<iframe src="https://medium.com/media/ac73413c1aff270b1c90ea43c8e5b792" frameborder=0></iframe>
