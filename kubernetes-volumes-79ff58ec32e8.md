Unknown markup type 10 { type: [33m10[39m, start: [33m25[39m, end: [33m33[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m37[39m, end: [33m45[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m27[39m, end: [33m30[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m35[39m, end: [33m55[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m57[39m, end: [33m66[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m71[39m, end: [33m88[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m43[39m, end: [33m52[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m56[39m, end: [33m62[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m27[39m, end: [33m33[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m35[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m28[39m, end: [33m45[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m51[39m, end: [33m59[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m320[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m338[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m120[39m, end: [33m135[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m104[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m169[39m, end: [33m173[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m181[39m, end: [33m196[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m279[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m151[39m, end: [33m166[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m329[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m16[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m68[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m22[39m, end: [33m28[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m308[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m108[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m63[39m, end: [33m74[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m194[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m45[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m47[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m53[39m, end: [33m66[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m76[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m125[39m, end: [33m128[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m52[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m1067[39m }

# Kubernetes Volumes



Docker also has a concept of [volumes](https://docs.docker.com/engine/admin/volumes/), though it is somewhat looser and less managed. In Docker, a volume is simply a directory on disk or in another Container. Lifetimes are not managed and until very recently there were only local-disk-backed volumes. Docker now provides volume drivers, but the functionality is very limited for now (e.g. as of Docker 1.7 only one volume driver is allowed per Container and there is no way to pass parameters to volumes).

A Kubernetes volume, on the other hand, has an explicit lifetime — the same as the Pod that encloses it. Consequently, a volume outlives any Containers that run within the Pod, and data is preserved across Container restarts. Of course, when a Pod ceases to exist, the volume will cease to exist, too. Perhaps more importantly than this, Kubernetes supports many types of volumes, and a Pod can use any number of them simultaneously.

Kubernetes volumes by example

A Kubernetes volume is essentially a directory accessible to all containers running in a pod. In contrast to the container-local filesystem, the data in volumes is preserved across container restarts. The medium backing a volume and its contents are determined by the volume type:

* node-local types such as emptyDir or hostPath

* file-sharing types such as nfs

* cloud provider-specific types like awsElasticBlockStore, azureDisk, or gcePersistentDisk

* distributed file system types, for example glusterfs or cephfs

* special-purpose types like secret, gitRepo

* A special type of volume is PersistentVolume.

### EmptyDir volume

emptyDir is a temporary directory that shares a pod’s lifetime. When the pod is destroyed, it will destroy the shared volume and all its contents.

Let’s create a pod with two containers that use an emptyDir volume to exchange data:

    apiVersion: v1
    kind: Pod
    metadata:
      name: sharevol
    spec:
      containers:
      - name: c1
        image: centos:7
        command:
          - "bin/bash"
          - "-c"
          - "sleep 10000"
        volumeMounts:
          - name: xchange
            mountPath: "/tmp/xchange"
      - name: c2
        image: centos:7
        command:
          - "bin/bash"
          - "-c"
          - "sleep 10000"
        volumeMounts:
          - name: xchange
            mountPath: "/tmp/data"
      volumes:
      - name: xchange
        emptyDir: {}

Lets create and see inside.

    $ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/volumes/pod.yaml
    
    $ kubectl describe pod sharevol
    Name:                   sharevol
    Namespace:              default
    ...
    Volumes:
      xchange:
        Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
        Medium:

### Hostpath volume

hostpath also node-local types k8s volume. This has the accessibility to from the host without accessing the cluster resources. But it must be provided absolute path to the directory . Then it is very important to care about the existence of the path before deployment and the permission available to the given directory. hostPath configuration in yaml file is shown bellow.

Ex :

    volumes:

      - name: phpldapadmin-certs

        hostPath:

            path: "/data/phpldapadmin/ssl/"

### Persistent Volume

A [persistent volume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) (PV) is a cluster-wide resource that you can use to store data in a way that it persists beyond the lifetime of a pod. The PV is not backed by locally-attached storage on a worker node but by networked storage system such as EBS or NFS or a distributed filesystem like Ceph.

The image at the beginning of this blog shows how pv works in a k8s cluster.

In order to use a PV you need to claim it first, using a persistent volume claim (PVC). The PVC requests a PV with your desired specification (size, speed, etc.) from Kubernetes and binds it then to a pod where you can mount it as a volume. So let’s create such a [PVC](https://github.com/openshift-evangelists/kbe/blob/master/specs/pv/pvc.yaml), asking Kubernetes for 1 GB of storage using the default [storage class](https://kubernetes.io/docs/concepts/storage/storage-classes/):

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: myclaim
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi

Lets apply it and check.

    $ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/pv/pvc.yaml
    
    $ kubectl get pvc
    NAME      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
    myclaim   Bound    pvc-27fed6b6-3047-11e9-84bb-12b5519f9b58   1Gi        RWO            gp2-encrypted   18m

To understand how the persistency plays out, let’s create a [deployment](https://github.com/openshift-evangelists/kbe/blob/master/specs/pv/deploy.yaml) that uses above PVC to mount it as a volume into /tmp/persistent:

    apiVersion: apps/v1beta1
    kind: Deployment
    metadata:
      name: pv-deploy
    spec:
      replicas: 1
      template:
        metadata:
          labels:
            app: mypv
        spec:
          containers:
          - name: shell
            image: centos:7
            command:
            - "bin/bash"
            - "-c"
            - "sleep 10000"
            volumeMounts:
            - name: mypd
              mountPath: "/tmp/persistent"
          volumes:
          - name: mypd
            persistentVolumeClaim:
              claimName: myclaim

This deployment can be found in the internet.

    kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/pv/deploy.yaml

Now we want to test if data in the volume actually persists. For this we find the pod managed by above deployment, exec into its main container and create a file called data in the /tmp/persistent directory (where we decided to mount the PV):

    $ kubectl get po
    NAME                         READY   STATUS    RESTARTS   AGE
    pv-deploy-69959dccb5-jhxx    1/1     Running   0          16m
    
    $ kubectl exec -it pv-deploy-69959dccb5-jhxxw -- bash
    bash-4.2$ touch /tmp/persistent/data
    bash-4.2$ ls /tmp/persistent/
    data  lost+found

It’s time to destroy the pod and let the deployment launch a new pod. The expectation is that the PV is available again in the new pod and the data in /tmp/persistent is still present. Let’s check that:

    $ kubectl delete po pv-deploy-69959dccb5-jhxxw
    pod pv-deploy-69959dccb5-jhxxw deleted
    
    $ kubectl get po
    NAME                         READY   STATUS    RESTARTS   AGE
    pv-deploy-69959dccb5-kwrrv   1/1     Running   0          16m
    
    $ kubectl exec -it pv-deploy-69959dccb5-kwrrv -- bash
    bash-4.2$ ls /tmp/persistent/
    data  lost+found

And indeed, the data file and its content is still where it is expected to be.

Note that the default behavior is that even when the deployment is deleted, the PVC (and the PV) continues to exist. This storage protection feature helps avoiding data loss. Once you’re sure you don’t need the data anymore, you can go ahead and delete the PVC and with it eventually destroy the PV:

    $ kubectl delete pvc myclaim
    persistentvolumeclaim "myclaim" deleted

The types of PV available in your Kubernetes cluster depend on the environment (on-prem or public cloud). Check out the [Stateful Kubernetes](https://stateful.kubernetes.sh/#storage) reference site if you want to learn more about this topic.

### Secret

Before using our credentials with secrets, some precautions must be taken First, secrets have a 1 MB size limitation. It works fine for defining several key-value pairs in a single secret. But, be aware that the total size should not exceed 1 MB. Next, secret acts like a volume for containers, so secrets should be created prior to dependent pods.

We can only generate secrets by using configuration…

You don’t want sensitive information such as a database password or an API key kept around in clear text. Secrets provide you with a mechanism to use such information in a safe and reliable way with the following properties:

* Secrets are namespaced objects, that is, exist in the context of a namespace

* You can access them via a volume or an environment variable from a container running in a pod

* The secret data on nodes is stored in [tmpfs](https://www.kernel.org/doc/Documentation/filesystems/tmpfs.txt) volumes

* A per-secret size limit of 1MB exists

* The API server stores secrets as plaintext in etcd

Let’s create a secret apikey that holds a (made-up) API key:

    $ echo -n "A19fh68B001j" > ./apikey.txt
    
    $ kubectl create secret generic apikey --from-file=./apikey.txt
    secret "apikey" created
    
    $ kubectl describe secrets/apikey
    Name:           apikey
    Namespace:      default
    Labels:         <none>
    Annotations:    <none>
    
    Type:   Opaque
    
    Data
    ====
    apikey.txt:     12 bytes

Now let’s use the secret in a [pod](https://github.com/openshift-evangelists/kbe/blob/master/specs/secrets/pod.yaml) via a [volume](http://kubernetesbyexample.com/volumes/):

    $ kubectl apply -f [https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/secrets/pod.yaml](https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/secrets/pod.yaml)

    OR

    apiVersion: v1
    kind: Pod
    metadata:
      name: consumesec
    spec:
      containers:
      - name: shell
        image: centos:7
        command:
          - "bin/bash"
          - "-c"
          - "sleep 10000"
        volumeMounts:
          - name: apikeyvol
            mountPath: "/tmp/apikey"
            readOnly: true
      volumes:
      - name: apikeyvol
        secret:
          secretName: apikey

If we now exec into the container we see the secret mounted at /tmp/apikey:

    $ kubectl exec -it consumesec -c shell -- bash
    [root@consumesec /]# mount | grep apikey
    tmpfs on /tmp/apikey type tmpfs (ro,relatime)
    [root@consumesec /]# cat /tmp/apikey/apikey.txt
    A19fh68B001j

Note that for service accounts Kubernetes automatically creates secrets containing credentials for accessing the API and modifies your pods to use this type of secret.

You can remove both the pod and the secret with:

    $ kubectl delete pod/consumesec secret/apikey

### How to protect PV from deletion

When the PV is protected. Delete the PV before deleting the PVC. Also, delete any pods/ deployments which are claiming any of the referenced PVCs. For further information do check out [Storage Object in Use Protection](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#storage-object-in-use-protection)

Cant delete pv because of this happens when persistent volume is protected. You should be able to cross verify this:

Command:

kubectl describe pvc PVC_NAME | grep Finalizers

Output:

Finalizers: [kubernetes.io/pvc-protection]

You can fix this by setting finalizers to null using kubectl patch:

    kubectl patch pvc PVC_NAME -p '{"metadata":{"finalizers": []}}' --type=merge

If PV still exists it may be because it has ReclaimPolicy set to Retain in which case it won’t be deleted even if PVC is gone. From the docs:

PersistentVolumes can have various reclaim policies, including “Retain”, “Recycle”, and “Delete”. For dynamically provisioned PersistentVolumes, the default reclaim policy is “Delete”. This means that a dynamically provisioned volume is automatically deleted when a user deletes the corresponding PersistentVolumeClaim. This automatic behavior might be inappropriate if the volume contains precious data. In that case, it is more appropriate to use the “Retain” policy. With the “Retain” policy, if a user deletes a PersistentVolumeClaim, the corresponding PersistentVolume is not be deleted. Instead, it is moved to the Released phase, where all of its data can be manually recovered

### Copying files and folders to another location from pod.

kubectl cp by default does recursive copies when given a directory, although it seems to be picky about trailing slashes. If foo is the directory you'd like to copy, simply run

    kubectl cp /path/to/foo <pod-id>:/path/in/container/

According to the help menu, the recursive option does not seem to exist.

    user@localhost ~ $ kubectl cp --help
    Copy files and directories to and from containers.
    
    Examples:
      # !!!Important Note!!!
      # Requires that the 'tar' binary is present in your container
      # image.  If 'tar' is not present, 'kubectl cp' will fail.
    
      # Copy /tmp/foo_dir local directory to /tmp/bar_dir in a remote pod in the default namespace
      kubectl cp /tmp/foo_dir <some-pod>:/tmp/bar_dir
    
      # Copy /tmp/foo local file to /tmp/bar in a remote pod in a specific container
      kubectl cp /tmp/foo <some-pod>:/tmp/bar -c <specific-container>
    
      # Copy /tmp/foo local file to /tmp/bar in a remote pod in namespace <some-namespace>
      kubectl cp /tmp/foo <some-namespace>/<some-pod>:/tmp/bar
    
      # Copy /tmp/foo from a remote pod to /tmp/bar locally
      kubectl cp <some-namespace>/<some-pod>:/tmp/foo /tmp/bar
    
    Options:
      -c, --container='': Container name. If omitted, the first container in the pod will be chosen
    
    Usage:
      kubectl cp <file-spec-src> <file-spec-dest> [options]
    
    Use "kubectl options" for a list of global command-line options (applies to all commands).

In order to copy files recursively, all files could be put in a directory and when this folder is copied to the pod, all files were copied:

### References:

Content of this blog taken from the bellow mentioned sites and some contents directly taken as it is. So ownership of the content goes those site owners.

    [http://kubernetesbyexample.com/volumes/](http://kubernetesbyexample.com/volumes/)
[**Volumes**
*Edit This Page On-disk files in a Container are ephemeral, which presents some problems for non-trivial applications…*kubernetes.io](https://kubernetes.io/docs/concepts/storage/volumes/)
