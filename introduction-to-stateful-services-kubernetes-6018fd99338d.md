Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m1253[39m }

# Introduction to Stateful Services — Kubernetes

Kubernetes Logo

Kubernetes is a container orchestration tool that is used to scale, deploy and manage containerized applications. These are the ways in Kubernetes to deploy pods: ReplicaSet, Deployments, StatefulSets, and DeamonSet.

***A question arises, why do we need StatefulSets?***

While writing this blog, I am making an assumption that you are familiar with the basics of Kubernetes. And, you know the meaning of terms like containers, pods, persistence volume, etc before.

## StatefulSets

ReplicaSet, Deployments, and DaemonSet work seamlessly with stateless applications but they are not suitable for stateful applications like MySQL, Kafka etc. This is where StatefulSets comes to the picture. Imagine a scenario where you want to deploy your containers in order, this is possible with StatefulSets. This blog is all about StatefulSets as we are talking about stateful applications.

StatefulSets in Kubernetes are used for Stateful applications. The pods in StatefulSets are named with unique Identities and stable hostname. It stores the state information and other resilient data in a persistent volume. This persistent volume is attached to Cloud-based storage. It is going to be responsible for maintaining the state of your application.

A persistent volume claim in the context of your StatefulSet, so that when that StatefulSet is created and each replica is created, Kubernetes will go ahead and create a different disk for each replica in the StatefulSet.

Now, when Kubernetes started, the only sort of way that you could do replication was using a ReplicaSet. With a replica set, every single replica was treated entirely identically. They have random hashes on the end of their application names. And if a scaling event happens, for example, a scaled-down, a container is chosen at random and deleted. These characteristics make ReplicaSet very hard to map to stateful applications. Many stateful applications expect their hostnames to be constant. So, those complexities of using ReplicaSet and stateful applications led to the eventual development of StatefulSets. A StatefulSets in Kubernetes is similar to a ReplicaSet, but it adds some guarantees that make it easier to manage stateful applications inside of Kubernetes.

    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: mysql
    spec:
      selector:
        matchLabels:
          app: mysql
      serviceName: mysql
      replicas: 3
      template:
        metadata:
          labels:
            app: mysql
        spec:
          initContainers:
          - name: init-mysql
            image: mysql:5.7
            command:
            - bash
            - "-c"
            - *|
              set -ex
              # Generate mysql server-id from pod ordinal index.
              [[ `hostname` =~ -([0-9]+)$ ]] || exit 1
              ordinal=${BASH_REMATCH[1]}
              echo [mysqld] > /mnt/conf.d/server-id.cnf
              # Add an offset to avoid reserved server-id=0 value.
              echo server-id=$((100 + $ordinal)) >> /mnt/conf.d/server-id.cnf
              # Copy appropriate conf.d files from config-map to emptyDir.
              if [[ $ordinal -eq 0 ]]; then
                cp /mnt/config-map/master.cnf /mnt/conf.d/
              else
                cp /mnt/config-map/slave.cnf /mnt/conf.d/
              fi*
            volumeMounts:
            - name: conf
              mountPath: /mnt/conf.d
            - name: config-map
              mountPath: /mnt/config-map
      volumeClaimTemplates:
      - metadata:
          name: data
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 10Gi

The working of StatefulSets is similar to Deployments, but in StatefulSets the deployment of containers happens in order. Rather than deploying all the containers in one go, it is deployed sequentially one by one. Once the first pod is deployed and gets ready, then only the second pod can start. In order to have the correct reference, these pods have a name with a unique ID which showcases there unique identity. So, for example, if there are 3 pods of MySQL, the names would be mysql-0 , mysql-1 and mysql-2. And, if any of these pod fail, a new pod with the same name will be deployed by StatefulSets. A headless service is required by StatefulSets to manage unique identities. The diagram below shows the architecture of StatefulSets:

![](https://cdn-images-1.medium.com/max/2000/0*1TVMvhJpYYurIsfZ)

## Scaling in StatefulSet

When Kubernetes decides to scale up or scale down a StatefulSet, it does it in a well-understood way. For example, when you initially create a StatefulSet, the first replica is created, and Kubernetes waits for it to become healthy and available before creating the second replica. This means that when the second replica is being created, you can depend on the fact that the zeroth index (1st replica) is available for you to connect to. It can point back to the original member of the StatefulSet. This makes it much easier to rendezvous to declare an initial leader and many other things that are necessary when you’re creating stateful applications.

Likewise, when you scale down, Kubernetes will delete starting at the highest index. So, if you scale down from three replicas to two replicas, it’s going to start by deleting the replica at index two.

## Conclusion

So, hopefully, that gives you an illustration of how stateful applications are deployed in Kubernetes and how you can use StatefulSets for such applications. The next step after this blog would be to go ahead and set up a StatefulSet.

— — — — —

*I’m consulting in Kubernetes, if you have any inquiries — please feel free to contact me. (https://danielckv.com || contact@danielckv.com)*
