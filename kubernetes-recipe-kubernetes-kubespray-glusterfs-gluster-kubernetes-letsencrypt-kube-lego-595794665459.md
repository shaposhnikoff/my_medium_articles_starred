Unknown markup type 10 { type: [33m10[39m, start: [33m207[39m, end: [33m208[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m96[39m, end: [33m156[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m166[39m, end: [33m172[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m193[39m, end: [33m252[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m269[39m, end: [33m355[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m583[39m, end: [33m623[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m802[39m, end: [33m823[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m15[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m26[39m, end: [33m30[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m82[39m }

# Kubernetes recipe: Kubernetes (kubespray) + GlusterFS (gluster-kubernetes) + Letsencrypt…

This guide will help you to get 3-node (master + 2 nodes) Kubernetes cluster on ScaleWay. We use Kubespray for Kubernetes installation and install GlusterFS Native Storage Service with dynamic provisioning using gluster-kubernetes. gluster-kubernetes demands that there must be at least three nodes with additional volume attached. In this tutorial we assume that each node has additional /dev/vdb volume.

## Contents

1. Create servers on ScaleWay

1. Install ansible on management host (local computer)

1. Install Kubernetes cluster

1. Install GlusterFS

1. Deploy Ingress controller

1. Deploy Letsencrypt certificate provider

1. Deploy example application

1. Deploy Heapster

## 1. Create servers on ScaleWay

Create three servers on [ScaleWay](https://www.scaleway.com/) ([Ubuntu Xenial, 16.04 latest](https://cloud.scaleway.com/#/zones/par1/images/89457135-d446-41ba-a8df-d53e5bb54710)) with 4Gb Memory (minimum) and additional [/dev/vdb](https://cloud.scaleway.com/#/zones/par1/volumes/cbb724a1-3424-49af-9c40-d277d5cb604a) volume.

node: master01
PublicIP: X.X.X.X
PrivateIP: A.A.A.A
node: node01
PublicIP: Y.Y.Y.Y
PrivateIP: B.B.B.B
node: node02
PublicIP: Z.Z.Z.Z
PrivateIP: C.C.C.C

Test connection to each node

    ssh root@X.X.X.X
    exit

## 2. Install ansible on management host (local computer)

    pip install virtualenv
    virtualenv venv
    source venv/bin/activate

    pip install ansible netaddr

## 3. Install Kubernetes cluster

Clone Kubespray on management host (local computer)

    git clone [https://github.com/kubernetes-incubator/kubespray.git](https://github.com/kubernetes-incubator/kubespray.git)

    cd [kubespray](https://github.com/kubernetes-incubator/kubespray.git)

    git checkout -b tags/v2.3.0

Create ansible inventory file (replace Public IP and Private IP for all nodes)

    cat <<EOF > inventory/inventory.cfg
    # Change ansible_ssh_host to your Public IP, ip to Private IP
    master01 ansible_ssh_host=X.X.X.X ip=A.A.A.A
    node01 ansible_ssh_host=Y.Y.Y.Y ip=B.B.B.B
    node02 ansible_ssh_host=Z.Z.Z.Z ip=C.C.C.C

    [kube-master]
    master01

    [etcd]
    master01

    [kube-node]
    master01
    node01
    node02

    [k8s-cluster:children]
    kube-node
    kube-master
    EOF

Change kubernetes definitions

    vi ./inventory/group_vars/k8s-cluster.yml

    # Enable access to kubernetes dashboard with basic auth
    kube_basic_auth: true

    # Use flannel for networking
    -kube_network_plugin: calico
    +kube_network_plugin: flannel

Run kubernetes installation

    ansible-playbook -u root -i inventory/inventory.cfg cluster.yml -b -v

Check installation

    ssh root@X.X.X.X

    kubectl get --all-namespaces all

    exit

## 4. Install GlusterFS on all nodes

On management host (local computer) run commands for all nodes using ansible ad-hoc commands:

    ansible kube-node -a "wipefs -a /dev/vdb" -u root -i inventory/inventory.cfg

    ansible kube-node -a "modprobe dm_thin_pool" -u root -i inventory/inventory.cfg

On management host (local computer) run commands to install mount.glusterfs utility (3.12 version) on each node:

    ansible kube-node -a "add-apt-repository ppa:gluster/glusterfs-3.12" -u root -i inventory/inventory.cfg

    ansible kube-node -a "apt-get update" -u root -i inventory/inventory.cfg

    ansible kube-node -a "apt-get install -y glusterfs-client" -u root -i inventory/inventory.cfg

**All next commands must be running on master node.**

Login to master node and install heketi-cli

    ssh root@X.X.X.X

    # Install heketi-cli
    HEKETI_BIN="heketi-cli"      # heketi or heketi-cli
    HEKETI_VERSION="5.0.0"       # latest heketi version => https://github.com/heketi/heketi/releases
    HEKETI_OS="linux"            # linux or darwin
    
    curl -SL https://github.com/heketi/heketi/releases/download/v${HEKETI_VERSION}/heketi-v${HEKETI_VERSION}.${HEKETI_OS}.amd64.tar.gz -o /tmp/heketi-v${HEKETI_VERSION}.${HEKETI_OS}.amd64.tar.gz && \
    tar xzvf /tmp/heketi-v${HEKETI_VERSION}.${HEKETI_OS}.amd64.tar.gz -C /tmp && \
    rm -vf /tmp/heketi-v${HEKETI_VERSION}.${HEKETI_OS}.amd64.tar.gz && \
    cp /tmp/heketi/${HEKETI_BIN} /usr/local/bin/${HEKETI_BIN}_${HEKETI_VERSION} && \
    rm -vrf /tmp/heketi && \
    cd /usr/local/bin && \
    ln -vsnf ${HEKETI_BIN}_${HEKETI_VERSION} ${HEKETI_BIN} && cd
    
    unset HEKETI_BIN HEKETI_VERSION HEKETI_OS

Install gluster-kubernetes

    $ git clone [https://github.com/gluster/gluster-kubernetes.git](https://github.com/gluster/gluster-kubernetes.git)
    $ cd gluster-kubernetes
    # git checkout -b 1879ab87b8ddddea1815f1f2759242c5fcc321af
    $ cd deploy

    cat <<EOF > topology.json
    {
      "clusters": [
        {
          "nodes": [
            {
              "node": {
                "hostnames": {
                  "manage": [
                    "master01"
                  ],
                  "storage": [
                    "A.A.A.A"
                  ]
                },
                "zone": 1
              },
              "devices": [
                "/dev/vdb"
              ]
            },
            {
              "node": {
                "hostnames": {
                  "manage": [
                    "node01"
                  ],
                  "storage": [
                    "B.B.B.B"
                  ]
                },
                "zone": 1
              },
              "devices": [
                "/dev/vdb"
              ]
            },
            {
              "node": {
                "hostnames": {
                  "manage": [
                    "node02"
                  ],
                  "storage": [
                    "C.C.C.C"
                  ]
                },
                "zone": 1
              },
              "devices": [
                "/dev/vdb"
              ]
            }
          ]
        }
      ]
    }
    EOF

    # You can use namespace with -n
    ./gk-deploy -gvy

    ...
    For dynamic provisioning, create a StorageClass similar to this:

    ---
    apiVersion: storage.k8s.io/v1beta1
    kind: StorageClass
    metadata:
      name: glusterfs-storage
    provisioner: kubernetes.io/glusterfs
    parameters:
      resturl: "[http://10.233.66.4:8080](http://10.233.66.4:8080)"

Install StorageClass using output from gk-deploy

    cat <<EOF | kubectl create -f -
    apiVersion: storage.k8s.io/v1beta1
    kind: StorageClass
    metadata:
      name: glusterfs-storage
    provisioner: kubernetes.io/glusterfs
    parameters:
      resturl: "[http://10.233.66.4:8080](http://10.233.66.4:8080)"
    EOF

## 5. Deploy Ingress controller

Run mandatory commands (from [https://github.com/kubernetes/ingress-nginx/blob/master/deploy/README.md#mandatory-commands](https://github.com/kubernetes/ingress-nginx/blob/master/deploy/README.md#mandatory-commands))

    curl [https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/namespace.yaml](https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/namespace.yaml) \
        | kubectl apply -f -

    curl [https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/default-backend.yaml](https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/default-backend.yaml) \
        | kubectl apply -f -

    curl [https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/configmap.yaml](https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/configmap.yaml) \
        | kubectl apply -f -

    curl [https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/tcp-services-configmap.yaml](https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/tcp-services-configmap.yaml) \
        | kubectl apply -f -

    curl [https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/udp-services-configmap.yaml](https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/udp-services-configmap.yaml) \
        | kubectl apply -f -

Install with RBAC roles (from [https://github.com/kubernetes/ingress-nginx/blob/master/deploy/README.md#install-with-rbac-roles](https://github.com/kubernetes/ingress-nginx/blob/master/deploy/README.md#install-with-rbac-roles))

    curl [https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/rbac.yaml](https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/rbac.yaml) \
        | kubectl apply -f -

    curl [https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/with-rbac.yaml](https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/with-rbac.yaml) \
        | kubectl apply -f -

Deploy Service (change your Private IP)

    cat <<EOF | kubectl create -f -
    apiVersion: v1
    kind: Service
    metadata:
      name: ingress-nginx
      namespace: ingress-nginx
    spec:
      type:
      ports:
      - name: http
        port: 80
        targetPort: 80
        protocol: TCP
      - name: https
        port: 443
        targetPort: 443
        protocol: TCP
      selector:
        app: ingress-nginx
      externalIPs:
      - X.X.X.X
    EOF

Check that nginx-ingress-controller and default-http-backend is running

    kubectl get pods -n ingress-nginx

    NAME READY STATUS RESTARTS AGE
    default-http-backend-66b447d9cf-4b8d9 1/1 Running 0 2m
    nginx-ingress-controller-c5b844d96-vr6bk 1/1 Running 0 1m

Check that nginx-ingress-controller was binded to 80, 443 port

    netstat -tupln

    tcp        0      0 A.A.A.A:80           0.0.0.0:*               LISTEN      20771/hyperkube
    tcp        0      0 A.A.A.A:443          0.0.0.0:*               LISTEN      20771/hyperkube

Check default-http-backend

    curl X.X.X.X
    default backend - 404

## 6. Deploy Letsencrypt certificate provider

Deploy [kube-lego](https://github.com/jetstack/kube-lego) (do not forget to change lego.email)

    cat <<EOF | kubectl create -f -
    apiVersion: v1
    kind: Namespace
    metadata:
        name: kube-lego
    ---
    kind: ClusterRole
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
      name: ingress-secret-admin
    rules:
    - apiGroups: [""]
      resources: ["secrets"]
      verbs:
      - get
      - watch
      - list
      - create
      - update
      - patch
    - apiGroups: [""]
      resources: ["services"]
      verbs:
      - get
      - create
    - apiGroups: ["extensions"]
      resources: ["ingresses"]
      verbs:
      - get
      - watch
      - list
      - create
      - update
      - patch
    ---
    kind: ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
      name: kube-lego
    roleRef:
      kind: ClusterRole
      name: ingress-secret-admin
      apiGroup: rbac.authorization.k8s.io
    subjects:
    - kind: ServiceAccount
      name: default
      namespace: kube-lego
    ---
    apiVersion: v1
    metadata:
      name: kube-lego
      namespace: kube-lego
    data:
      # modify this to specify your address
      **lego.email: "your@email.com"**
      # configure letsencrypt's production api
      lego.url: "[https://acme-v01.api.letsencrypt.org/directory](https://acme-v01.api.letsencrypt.org/directory)"
    kind: ConfigMap
    ---
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: kube-lego
      namespace: kube-lego
    spec:
      replicas: 1
      template:
        metadata:
          labels:
            app: kube-lego
        spec:
          containers:
          - name: kube-lego
            image: jetstack/kube-lego:0.1.5
            imagePullPolicy: Always
            ports:
            - containerPort: 8080
            env:
            - name: LEGO_EMAIL
              valueFrom:
                configMapKeyRef:
                  name: kube-lego
                  key: lego.email
            - name: LEGO_URL
              valueFrom:
                configMapKeyRef:
                  name: kube-lego
                  key: lego.url
            - name: LEGO_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            - name: LEGO_POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
            readinessProbe:
              httpGet:
                path: /healthz
                port: 8080
              initialDelaySeconds: 5
              timeoutSeconds: 1
    EOF

Check kube-lego is running

    kubectl get pods -n kube-lego

    NAME                         READY     STATUS    RESTARTS   AGE
    kube-lego-674567867b-5wmfh   1/1       Running   0          25s

## 7. Deploy example application

Add an A record in your DNS and wait for your changes to propagate

    Type: A
    Name: echo.example.com (your domain name)
    Value: X.X.X.X (Public IP of Ingress controller)

Deploy exampleapp (replace app.example.com with your domain name)

    cat <<EOF | kubectl create -f -
    apiVersion: v1
    kind: Namespace
    metadata:
      name: exampleapp
    ---
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
     name: exampleapp-pvc
     namespace: exampleapp
     annotations:
       volume.beta.kubernetes.io/storage-class: glusterfs-storage
    spec:
     accessModes:
      - ReadWriteOnce
     resources:
       requests:
         storage: 1Gi
    ---
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: exampleapp
      namespace: exampleapp
    spec:
      replicas: 1
      template:
        metadata:
          labels:
            app: exampleapp
        spec:
          containers:
          - image: gcr.io/google_containers/nginx-slim:0.27
            imagePullPolicy: Always
            name: exampleapp
            ports:
            - containerPort: 80
            volumeMounts:
              - name: exampleapp-vol
                mountPath: /usr/share/nginx/html
          volumes:
            - name: exampleapp-vol
              persistentVolumeClaim:
                claimName: exampleapp-pvc
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: exampleapp-svc
      namespace: exampleapp
    spec:
      ports:
      - port: 80
        targetPort: 80
        protocol: TCP
      selector:
        app: exampleapp
    ---
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: exampleapp-ingress
      namespace: exampleapp
      annotations:
        kubernetes.io/tls-acme: "true"
        kubernetes.io/ingress.class: "nginx"
    spec:
      tls:
      - hosts:
        # CHANGE ME
    **    - app.example.com**
        secretName: exampleapp-tls
      rules:
      # CHANGE ME
    **  - host: app.example.com**
        http:
          paths:
          - path: /
            backend:
              serviceName: exampleapp-svc
              servicePort: 80
    EOF

Check for letsencrypt certificate

    kubectl get pods -n kube-lego

    NAME                         READY     STATUS    RESTARTS   AGE
    kube-lego-674567867b-5wmfh   1/1       Running   0          25

    kubectl logs -f kube-lego-674567867b-5wmfh -n kube-lego

Check ingress resources

    kubectl get --all-namespaces ing

    NAMESPACE    NAME                 HOSTS                   ADDRESS   PORTS     AGE
    exampleapp   exampleapp-ingress   app.example.com             80, 443   27m

Place index.html in example application

    kubectl get -n exampleapp pods

    kubectl exec -ti -n exampleapp exampleapp-857d64d866-4ncm9 /bin/sh

    cd /usr/share/nginx/html
    echo 'Hello World from GlusterFS!!!' > index.html
    ls
    exit

    curl app.example.com
    Hello World from GlusterFS!!!

Test deployment

    curl https://app.example.com

Test GlusterFS volume

    kubectl get -o wide pods
    NAME                      READY     STATUS    RESTARTS   AGE       IP             NODE
    glusterfs-8t2h5           1/1       Running   0          6h        10.4.163.131   node01
    glusterfs-qrtbg           1/1       Running   0          6h        10.4.24.73     node02
    glusterfs-sr5qv           1/1       Running   0          6h        10.4.101.143   master01

    exec -ti glusterfs-sr5qv sh
    cat /var/lib/heketi/mounts/vg_RANDOM_GUID/brick_RANDOM_GUID/brick/index.html
    Hello World from GlusterFS!!!

## 8. Deploy Heapster

We have installed Kubernetes dashboard with Kubespray, it is available at [https://kube:PASSWORD@X.X.X.X:6443/ui,](https://kube:PASSWORD@X.X.X.X:6443/ui,) where PASSWORD is located in ./credentials/kube_user file of your Kubespray directory on management host. Kubernetes recommends Heapster as a cluster aggregator to monitor usage of nodes and pods. Also we need Heapster for running the [Horizontal Pod Autoscaler](http://kubernetes.io/docs/user-guide/horizontal-pod-autoscaling/).

Deploy Heapster

    curl [https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/rbac/heapster-rbac.yaml](https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/rbac/heapster-rbac.yaml) \
        | kubectl apply -f -
    curl [https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/standalone/heapster-controller.yaml](https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/standalone/heapster-controller.yaml) \
        | kubectl apply -f -

Restart Kubernetes dashboard

    kubectl get --all-namespaces pods |grep dashboard

    kubectl get pod kubernetes-dashboard-7fd45476f8-fgtr8 -n kube-system -o yaml | kubectl replace --force -f -

Open [https://kube:PASSWORD@X.X.X.X:6443/ui](https://kube:PASSWORD@X.X.X.X:6443/ui)

That’s all! Happy kuberneting!
