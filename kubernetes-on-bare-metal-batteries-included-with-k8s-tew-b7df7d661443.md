
# Kubernetes on bare-metal “batteries included“ with k8s-tew

As a Kubernetes administrator, I usually deploy standalone kubernetes cluster to test some features, applications, or even kubernetes tooling. But in this case, it takes some time to deploy Kubernetes, CNI, storage solution, load balancer service implementation, ingress controller, monitoring, logging, etc. With Kubernetes-ready cloud distribution like GKE, or DigitalOcean, it is fair simple, but how to do the same with bare-metal ? This may, of course, include your linux VM on your machine …

**k8s-tew** is a single binary written in GO, with no external dependency, that will easily deploy a Kubernetes cluster over bare-metal (understand : no cloud provider required) even for a single node test-bench for your POCs.

![k8s-tew logo](https://cdn-images-1.medium.com/max/2000/1*eQtyht27Xec__QQtP8vxpw.png)*k8s-tew logo*

**k8s-tew** will deploy :

* HA or non-HA cluster setup that passes all CNCF conformance tests (Kubernetes [1.10](https://github.com/cncf/k8s-conformance/tree/master/v1.10/k8s-tew), [1.11](https://github.com/cncf/k8s-conformance/tree/master/v1.11/k8s-tew), [1.12](https://github.com/cncf/k8s-conformance/tree/master/v1.12/k8s-tew) & [1.13](https://github.com/cncf/k8s-conformance/tree/master/v1.13/k8s-tew))

* Container Management: [Containerd](https://containerd.io/)

* Networking: [Calico](https://www.projectcalico.org/)

* Ingress: [NGINX Ingress](https://kubernetes.github.io/ingress-nginx/) and [cert-manager](http://docs.cert-manager.io/en/latest/) for [Let’s Encrypt](https://letsencrypt.org/)

* Storage: [Ceph/RBD](https://ceph.com/)

* Metrics: [metering-metrics](https://github.com/kubernetes-incubator/metrics-server) and [Heapster](https://github.com/kubernetes/heapster)

* Monitoring: [Prometheus](https://prometheus.io/) and [Grafana](https://grafana.com/)

* Logging: [Fluent-Bit](https://fluentbit.io/), [Elasticsearch](https://www.elastic.co/), [Kibana](https://www.elastic.co/products/kibana) and [Cerebro](https://github.com/lmenezes/cerebro)

* Backups: [Ark](https://github.com/heptio/ark), [Restic](https://restic.net/) and [Minio](https://www.minio.io/)

* Controller Load Balancing: [gobetween](http://gobetween.io/)

* Package Manager: [Helm](https://helm.sh/)

* Dashboard: [Kubernetes Dashboard](https://github.com/kubernetes/dashboard)

* [WordPress](https://wordpress.com/) and [MySQL](https://www.mysql.com/) to test drive the installation

Even if **k8s-tew** is able to deploy HA clusters on multiple nodes, one interesting feature is to enable the deployment of a full feature cluster on single node, facilitating kubernetes self-training.

The quickstart section of the [documentation](https://darxkies.github.io/k8s-tew/) is all about single node setup, and it is pretty simple :

    *# Switch to user root*
    sudo su -

    *# Download Binary*
    wget https://github.com/darxkies/k8s-tew/releases/download/2.2.4/k8s-tew
    chmod a+x k8s-tew
    
    *# Everything is installed relative to the root directory*
    export K8S_TEW_BASE_DIRECTORY=/
    
    *# Create cluster data*
    ./k8s-tew initialize
    ./k8s-tew node-add -s 

    # Only on Ubuntu 18.04 :
    ./k8s-tew configure --resolv-conf=/run/systemd/resolve/resolv.conf

    *# Prepare cluster
    *./k8s-tew generate
    
    *# Activate and start service*
    systemctl daemon-reload
    systemctl enable k8s-tew
    systemctl start k8s-tew

You can then download your *kubeconfig* file in :

    /etc/k8s-tew/k8s/kubeconfig/admin.kubeconfig

… and after a few minutes (time taken to let Kubernetes converge to required state with all application set-up) you can enjoy your Kubernetes cluster :

    # List of PODs by namespace
    NAMESPACE     NAME
    backup        ark-7bbcb7ff9b-t6jq5
    backup        minio-85974b6ff-d8b58
    backup        restic-rzfgh
    default       elasticsearch-operator-sysctl-6tzfh
    kube-system   coredns-74c648b76d-hf724
    kube-system   coredns-74c648b76d-rczxk
    kube-system   etcd-lab1
    kube-system   gobetween-lab1
    kube-system   heapster-heapster-849c47d8c9-wxznk
    kube-system   kube-apiserver-lab1
    kube-system   kube-controller-manager-lab1
    kube-system   kube-proxy-lab1
    kube-system   kube-scheduler-lab1
    kube-system   kubernetes-dashboard-565db58d47-4v46w
    kube-system   tiller-deploy-c8bfd74c4-525mc
    logging       cerebro-elasticsearch-cluster-7785948b9d-85dd9
    logging       elasticsearch-operator-5c597554d4-v5dpr
    logging       es-client-elasticsearch-cluster-cb795d4b8-g6f9x
    logging       es-data-elasticsearch-cluster-default-0
    logging       es-master-elasticsearch-cluster-default-0
    logging       fluent-bit-2d4nk
    logging       kibana-elasticsearch-cluster-7fb7f88f55-5fk65
    monitoring    alertmanager-kube-prometheus-0
    monitoring    kube-prometheus-exporter-kube-state-6599774694-tkznb
    monitoring    kube-prometheus-exporter-node-kx75d
    monitoring    kube-prometheus-grafana-6cc9c9c96b-ntk2r
    monitoring    metrics-server-5df85b997d-qwp5s
    monitoring    prometheus-kube-prometheus-0
    monitoring    prometheus-operator-7d9844f964-gx7mh
    networking    calico-node-v6v6x
    networking    cert-manager-58898b5d55-246mp
    networking    metallb-controller-7444c5f7b5-5kthm
    networking    metallb-speaker-pfrbp
    networking    nginx-ingress-controller-v8nnd
    networking    nginx-ingress-default-backend-7b84c755cc-88ncb
    showcase      mysql-54d86bc8d-4vlsb
    showcase      wordpress-65779ff967-sz8dc
    storage       ceph-mds-lab1-688dc7dcd-549tx
    storage       ceph-mgr-6fc4d984cd-k9hs2
    storage       ceph-mon-lab1-6db48dc68d-vcpvn
    storage       ceph-osd-lab1-5d8c7dc6bc-9548r
    storage       ceph-rgw-6fcb67fd78-vlwg2
    storage       csi-cephfsplugin-attacher-0
    storage       csi-cephfsplugin-gdjvg
    storage       csi-cephfsplugin-provisioner-0
    storage       csi-rbdplugin-attacher-0
    storage       csi-rbdplugin-provisioner-0
    storage       csi-rbdplugin-vwr75

You can then access:

* Wordpress showcase : [http://[node-ip]:30100](http://[node-ip]:30100)

* Minio UI : [http://[node-ip]:30800](http://[node-ip]:30800)

* Grafana UI : [http://[node-ip]:30900](http://[node-ip]:30900)

* Kibana UI : [https://[node-ip]:30980](https://[node-ip]:30980)

* Cerebro UI : [http://[node-ip]:30990](http://[node-ip]:30990)

* Ceph UI : [https://[node-ip]:30700](https://[node-ip]:30700)

## **Subjective point of view :**

I think **k8s-tew** is a very good tool for playing with Kubernetes and managing “day two” and not just k8s raw setup. But I would not recommend using **k8s-tew** in production because this project has only been maintained by one person so far (Darx Kies / [https://github.com/darxkies](https://github.com/darxkies), many many thanks to him for providing this to the community) who started this project in June 2018. This project really deserves to be supported by many more people.

Discover **k8s-tew** by yourself on [https://darxkies.github.io/k8s-tew/](https://darxkies.github.io/k8s-tew/)

*Edit : For more setup options, see : [https://github.com/darxkies/k8s-tew/tree/master/setup](https://github.com/darxkies/k8s-tew/tree/master/setup)*
