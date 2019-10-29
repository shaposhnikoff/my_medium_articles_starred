Unknown markup type 10 { type: [33m10[39m, start: [33m248[39m, end: [33m279[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m311[39m, end: [33m312[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m328[39m, end: [33m336[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m33[39m, end: [33m41[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m88[39m, end: [33m96[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m188[39m, end: [33m196[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m272[39m, end: [33m280[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m87[39m, end: [33m99[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m5[39m, end: [33m17[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m62[39m, end: [33m83[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m120[39m, end: [33m124[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m41[39m, end: [33m58[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m141[39m, end: [33m156[39m }

# Using NodeSelector to Schedule Deployments with large volumes of Stateful Data on Kubernetes

Source: https://www.wocintechchat.com/

I recently migrated an application that reads from a very substantial data store to Kubernetes; the big challenge was that my standard Kubernetes nodes looked something like:

    8-16 GB, 50 GB SSD, 150 GB SSD (/mnt/kube-data)

However, its data-store is something like 4–6TB, and since the application’s primary function is to stream (almost entirely read activity — writes happen from a Kubernetes Job, which I won’t cover here, but the principle will be the same) much of this data, the latency for something like an NFS-backed volume was vaguely too high, so I mounted the block devices (containing replicas of the data) to 2 of the hosts, and planned to schedule the Deployment to target only those two nodes.

In this case, these are block devices dedicated to this application, mounted to the Kube nodes I want to target, and while there are more involved/less-brittle configurations for this kind of setup, I elected to use the mountpoint for the devices, /mnt/kube-data/imported-volume/ (containing a directory called app-data`), as a hostPath volume.

The benefit here is that, with a hostPath volume, the deployment can also fail if the device isn’t mounted, and likewise if the scheduling rule (which I’ll cover in a moment) doesn’t work, the Deployment won’t be run (because that path won’t exist):

    spec:
          containers:
          - name: streamer-v4
            image: coolregistryusa.biz
            ports:
            - containerPort: 8880
            volumeMounts:
            - mountPath: /data/assets
              name: test-volume
          volumes:
          - name: test-volume
            hostPath:
    **          path: /mnt/kube-data/imported-volume/app-data**
              type: Directory

So, this hostPath volume will depend on the block device being mounted in order for the app-data directory to be present, whereas other host volume related types may not detect this (i.e. emptyDir will create a missing directory, and [there are a couple of other types for hostPath itself that can, indeed, target a specific block device](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath) — which may not be ideal for a few reasons, but does exist if you can count on consistent naming, UUIDs, etc.-).

Now that our manifest knows how to make use of the data volume on the host, we can use nodeSelector to tell the API server to only schedule this resource on nodes that contain this large volume:

    spec:
          containers:
          - name: streamer-v4
            image: coolregistryusa.biz/jmarhee/streamer-v4
            ports:
            - containerPort: 8880
            volumeMounts:
            - mountPath: /data/assets
              name: test-volume
          volumes:
          - name: test-volume
            hostPath:
              path: /mnt/kube-data/imported-volume/app-data
              type: Directory
          imagePullSecrets:
          - name: regcred
    **      nodeSelector:
            big-streaming-storage: "true"**

What nodeSelector does here is check for hosts with the label big-streaming-storage applied, and if that has a value of true ; your label, and the accompanying value can be set to whatever you’d like. You can also similarly label and taint nodes to prevent or explicitly set affinity for scheduling rules:
[**Assigning Pods to Nodes**
*Edit This Page You can constrain a pod to only be able to run on particular nodes or to prefer to run on particular…*kubernetes.io](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector)
[**Running Spark Jobs with Kubernetes on DigitalOcean High CPU Droplets**
*With the release high CPU droplets on DigitalOcean, running data workflows, streaming and processing, etc. can be…*medium.com](https://medium.com/@jmarhee/running-spark-jobs-with-kubernetes-on-digitalocean-high-cpu-droplets-6edaa415c09)

So, in my case, I took the node IDs from kubectl get nodes :

    # kubectl get nodes
    NAME                   STATUS   ROLES    AGE   VERSION
    kube-controller-a      Ready    master   16h   v1.12.2

    kube-controller-b      Ready    master   16h   v1.12.2
    kube-node-scw-40f957   Ready    <none>   15h   v1.12.2
    kube-node-scw-79ec60   Ready    <none>   12h   v1.12.2
    kube-node-scw-f96397   Ready    <none>   16h   v1.12.2

I know that these two nodes have the big volume attached, so I’m going to apply my label to them:

    kubectl label nodes kube-node-scw-79ec60 **big-streaming-storage**=true
    kubectl label nodes kube-node-scw-f96397 **big-streaming-storage=true**

and then you can validate by applying the label to your kubectl command to list the nodes with that label attached:

    kubectl get nodes -l **big-streaming-storage=true**

Let’s go back to our manifest, which should look like this completed:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: streamer-v4-deployment
      labels:
        app: streamer-v4
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: streamer-v4
      template:
        metadata:
          labels:
            app: streamer-v4
        spec:
          containers:
          - name: streamer-v4
            image: coolregistryusa.biz/jmarhee/streamer-v4
            ports:
            - containerPort: 8880
            volumeMounts:
            - mountPath: /data/assets
              name: test-volume
          volumes:
          - name: test-volume
            hostPath:
              path: /mnt/kube-data/imported-volume/app-data
              type: Directory
          imagePullSecrets:
          - name: regcred
          nodeSelector:
            big-streaming-storage: "true"

and go ahead and apply:

    kubectl apply -f streaming-deployment.yaml

Once it applies successfully, you can validate that the pods were scheduled per your rules, using the labels on the application above ( i.e. app=streamer-v4 ):

    # kubectl describe pod -l app=streamer-v4 | grep Node

    Node:               **kube-node-scw-79ec60**/10.1.70.10
    Node-Selectors:  big-streaming-storage=true
    Node:               **kube-node-scw-f96397**/10.1.70.9

You’ll see that the Nodes containing the 2 replicas of the Pod you defined in your Deployment above ended up on the 2 nodes containing the host volume we specified, using that node label.
