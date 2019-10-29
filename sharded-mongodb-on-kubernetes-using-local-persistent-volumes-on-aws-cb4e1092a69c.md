Unknown markup type 10 { type: [33m10[39m, start: [33m148[39m, end: [33m193[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m28[39m, end: [33m50[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m75[39m, end: [33m91[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m96[39m, end: [33m112[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m40[39m, end: [33m52[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m156[39m, end: [33m165[39m }

# Sharded MongoDB on Kubernetes using local persistent volumes on AWS

This post will show how you can create sharded mongodb clusters using local persistent volumes on AWS.

i3 series instances come with high random IO performance SSD storage, excellent network bandwidth and are highly recommended for NoSQL workloads like MongoDB. We use the same for its MongoDB clusters. Now this requires us to use local persistent volumes and Kubernetes provides improved support for local persistent volumes in 1.9 with delayed volume binding. Local persistent volumes is also documented here [https://github.com/kubernetes-incubator/external-storage/tree/master/local-volume](https://github.com/kubernetes-incubator/external-storage/tree/master/local-volume). Through all of this, what is not apparent is that Kubernetes makes it really easy to use local persistent volumes.

## Enabling local persistent volumes

We will use Kubespray v2.5.0 to launch and manage our Kubernetes cluster. To enable local persistent volumes, set the following property to true in inventory/<folder>/group_vars/k8s-cluster.yml

    persistent_volumes_enabled: true
    local_volume_provisioner_enabled: true

This enables the Kubernetes PersistentLocalVolumes feature gate along with VolumeScheduling and MountPropagation.

Also note the following two variables

    local_volume_provisioner_base_dir: /mnt/disks
    local_volume_provisioner_mount_dir: /mnt/disks

Now go ahead and launch the cluster. For this post, I launched a cluster with 3 config nodes i3/other and 6 i3.2xlarge instances for the shards.

Once you launch your cluster with Kubespray you should be able to see your nodes with

    kubectl get nodes

Next you should tag your nodes to distinguish what is config and what is for shards. There are different ways to do it. But basically it amounts to issuing something like

    kubectl label nodes <node name> component=mongo-config
    kubectl label nodes <node name> component=mongo-shard

We will later use these tags to specify nodeAffinity for our pods so that we bind them to the right set of instances.

## Provisioning local persistent volumes

Now we will see how to provision the local pvs. All we need to do is to mount the local disks to /mnt/disks. They will be auto-detected and then made available as local pv automatically.

There are a couple ways to do this. You could use a daemonset or you could use a simple script that does this for each of the nodes labeled mongo-shard.

<iframe src="https://medium.com/media/7a4415b00033a15271a327c53f1d07b6" frameborder=0></iframe>

The above script would discover each disk in the i3 instance and format them using xfs (which is what is [recommended for MongoDB](https://scalegrid.io/blog/xfs-vs-ext4-comparing-mongodb-performance-on-aws-ec2/)) and mount to /mnt/disks. It also adds the entry to fstab so that the mount is durable across node reboots.

Now the Kubernetes magic happens and these volumes are made available automatically! The following snippet shows the same (but after binding to pods)

    kubectl get pv

    NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM                                        STORAGECLASS    REASON    AGE

    local-pv-1756e029                          442Gi      RWO            Delete           Bound     default/data-mongo-shard0-1                  local-storage             9d

    local-pv-277d7925                          442Gi      RWO            Delete           Bound     default/data-mongo-shard1-0                  local-storage             9d

    local-pv-5e05e368                          442Gi      RWO            Delete           Bound     default/data-mongo-shard1-1                  local-storage             9d

    local-pv-6836da3e                          442Gi      RWO            Delete           Bound     default/data-mongo-shard0-2                  local-storage             9d

    local-pv-a66723f4                          442Gi      RWO            Delete           Bound     default/data-mongo-shard0-0                  local-storage             9d

    local-pv-d3c10f22                          442Gi      RWO            Delete           Bound     default/data-mongo-shard1-2                  local-storage             9d

## Create key file

We will now create the key file. We will do it in a repeatable manner.

<iframe src="https://medium.com/media/d32c1770887f4d84ff800cf526479569" frameborder=0></iframe>

This script basically creates a mongodb-keyfile if it does not exist as a secret in Kubernetes

We will later refer to this key file secret in all of our yaml for mongo-config, mongo-shard etc.

## Spin up mongo-config

We will next bring up our mongo-config pods

You can find the yaml for that here [https://github.com/sabarishs/sharded-mongo-kube-local-pv/blob/master/src/mongo-config.yaml](https://github.com/sabarishs/sharded-mongo-kube-local-pv/blob/master/src/mongo-config.yaml)

Notice how we use a statefulset and also bind our pods to the mongo-config labeled nodes

    affinity:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
          - key: node.ignite.harman.com/component
          operator: In
          values: ["mongo-config"]

Now we need to initialize the config replicaset

    kubectl exec mongo-config-0 -- mongo --eval "rs.initiate({_id: \"crs\", configsvr: true, members: [ {_id: 0, host: \"mongo-config-0.mongo-config-svc.default.svc.cluster.local:27017\"}, {_id: 1, host: \"mongo-config-1.mongo-config-svc.default.svc.cluster.local:27017\"}, {_id: 2, host: \"mongo-config-2.mongo-config-svc.default.svc.cluster.local:27017\"} ]});"

## Spin up mongo query router pods

Next we will bring up a few query routers. Mongo best practices suggest that you should run this on each node requiring mongo access. You can run this as a daemonset and bind it to all nodes that will be needing access to mongo.

You can find the yaml for that here [https://github.com/sabarishs/sharded-mongo-kube-local-pv/blob/master/src/mongo-qr.yaml](https://github.com/sabarishs/sharded-mongo-kube-local-pv/blob/master/src/mongo-qr.yaml)

Notice how we use nodeAffinity to bind this to all nodes of type ‘api’

    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: node.ignite.harman.com/component
          operator: In
          values:
          - api

So all API pods can now access the local query router using <node-host-name>:27017

You could run this as an additional container in your API pod but then that would mean multiple query routers would run in a single API node and will result in bloating the number of open mongo connections. By running as a daemonset we restrict it to one per candidate node.

## Spin up mongo-shard

Next we will bring up a mongo-shard. Because you want to do multiple shards, the standard practice is to have a template and then generate the actual yaml artifacts for each shard by replacing the shard index.

You can find the yaml here [https://github.com/sabarishs/sharded-mongo-kube-local-pv/blob/master/src/mongo-shard.yaml](https://github.com/sabarishs/sharded-mongo-kube-local-pv/blob/master/src/mongo-shard.yaml)

We will generate one of these per shard, then initiate the replicaset and finally add the shard to the query router in a loop.

    while [[ $shards -lt $shardcount ]]
    do

      sed "s/shard0/shard$shards/g" mongo-shard.yaml > mongo-shard$shards.yaml

      kubectl apply -f mongo-shard$shards.yaml

      kubectl exec mongo-shard$shards-0 -- mongo --eval "rs.initiate({_id: \"shard$shards\", members: [ {_id: 0, host: \"mongo-shard$shards-0.mongo-shard$shards-svc.default.svc.cluster.local:27017\"}, {_id: 1, host: \"mongo-shard$shards-1.mongo-shard$shards-svc.default.svc.cluster.local:27017\"}, {_id: 2, host: \"mongo-shard$shards-2.mongo-shard$shards-svc.default.svc.cluster.local:27017\"} ]});"

      kubectl exec $qrPod -- mongo --eval "sh.addShard(\"shard$shards/mongo-shard$shards-0.mongo-shard$shards-svc.default.svc.cluster.local:27017\");"

      shards=$(($shards+1))

    done

Wonderful! We are done. If we check the pv now we will see they are bound. Our shards are now up with local persistent volumes on Kubernetes. Good job!

All of these artifacts and a bootstrap script that ties these together can be found in my github [https://github.com/sabarishs/sharded-mongo-kube-local-pv](https://github.com/sabarishs/sharded-mongo-kube-local-pv)

I would recommend you look at the bootstrap script. It can be rerun any number of times and also helps scale the number of shards.

    #the following sets up 2 sharded cluster
    mongo-bootstrap.sh 2

    #the following adds another shard to make it a 3 shard cluster
    mongo-bootstrap.sh 3

## Acknowledgement

I wouldn’t have accomplished all of this in 3 days if not for these excellent blog posts from Paul Done and Sunny.

[Paul Done’s excellent blog](http://pauldone.blogspot.in/2017/06/deploying-mongodb-on-kubernetes-gke25.html)
[Sunny’s blog on medium](https://medium.com/google-cloud/sharded-mongodb-in-kubernetes-statefulsets-on-gke-ba08c7c0c0b0)
