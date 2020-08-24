Unknown markup type 10 { type: [33m10[39m, start: [33m78[39m, end: [33m110[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m93[39m, end: [33m107[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m132[39m, end: [33m145[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m70[39m, end: [33m76[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m48[39m, end: [33m54[39m }

# Using  a K3s Kubernetes Cluster For Your GitLab Project

Create a k3s cluster and integrate it into your GitLab project easily

![](https://cdn-images-1.medium.com/max/2000/1*4D9MtzlmIM2vNNI5ngvTLQ.png)

## TL;DR

k3s is a lightweight Kubernetes distribution (less than 40 MB), is very easy to install, and only requires 512 MB of RAM. It’s a perfect candidate for IoT devices and edge computing but also to run CI jobs. In this article, we will create a k3s cluster and show how to integrate it in a GitLab project.

## About k3s

[k3s](https://k3s.io/) is a lightweight Kubernetes distribution made by [Rancher Labs](https://rancher.com/).

It’s a Certified Kubernetes distribution with minimal system requirements:

* Linux 3.10+

* 512 MB of ram per server

* 75 MB of ram per node

* 200 MB of disk space

* x86_64, ARMv7, ARM64

This makes k3s a good fit for IoT-related things.

## Creation of a project in GitLab

Before installing k3s, we create a new project on Gitlab, which we call *api*.

![](https://cdn-images-1.medium.com/max/4728/1*8PqnZqF9AoH2t9llJFuA9Q.png)

Once created, we go into the *Operations > Kubernetes* menu.

![](https://cdn-images-1.medium.com/max/5416/1*H1ZJo1GXEvreOckV5-3Qcg.png)

From there we have two choices:

* We can create a Kubernetes cluster on GKE (Google Kubernetes Engine).

* We can import the configuration of an existing Kubernetes cluster (no matter where it was created).

**Note**: In the current release of GitLab, the creation of a new cluster is limited to GKE. [GitLab](undefined), any plan to allow this on other Kubernetes providers (AKS, EKS, DOKS …)? :)

![](https://cdn-images-1.medium.com/max/5416/1*3KzOebhb-Jp_D3d7cJqwCw.png)

We choose the *Add existing cluster* tab.

![](https://cdn-images-1.medium.com/max/5416/1*_abn2TFMXwA8zGrFaUtZzQ.png)

From there, we need to fill several fields to provide the configuration of the cluster we need to integrate. Let’s keep this tab open and create a brand new Kubernetes cluster first.

## Creation of a k3s Cluster

We will now initiate a Kubernetes based on k3s. Why k3s? Because I want to show how easy it is to set this up. :) To keep it simple, we will only set up a one-node cluster.

I have provisioned an Ubuntu 18.04 server named *node1*. Once we get a shell on that host, we only need to run the following command to install k3s, a certified Kubernetes cluster. Really!

    root@node1:~ $ curl -sfL [https://get.k3s.io](https://get.k3s.io) | sh -

The above command is similar to the one used for a quick Docker installation: curl [https://get.docker.com](https://get.docker.com) | sh

Once installed (it’s incredibly fast), the configuration file to connect to the cluster is available in /etc/rancher/k3s/k3s.yaml.

    **root@node1:~ $ cat /etc/rancher/k3s/k3s.yaml
    **apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: LS0tL...tCg==
        server: https://localhost:6443
      name: default
    contexts:
    - context:
        cluster: default
        user: default
      name: default
    current-context: default
    kind: Config
    preferences: {}
    users:
    - name: default
      user:
        password: 48f4b...4b4e7
        username: admin

The local kubectl is automatically configured to use this configuration.

    **$ kubectl get nodes**
    NAME    STATUS ROLES  AGE VERSION
    node1   Ready  master 3m  v1.14.5-k3s.1

Note: adding additional nodes is very easy as we can see at the end of the [Quick Start](https://k3s.io/). It’s basically only getting a token from */var/lib/rancher/k3s/server/node-token* on the master and joining some other nodes using the following command:

    $ curl -sfL [https://get.k3s.io](https://get.k3s.io) | K3S_URL=https://myserver:6443 K3S_TOKEN=XXX sh -

## Integration in Gitlab

Let’s now get all the information needed to integrate our brand new k3s cluster in our Gitlab’s project.

* Cluster name

Let’s call it *k3s*.

* URL of the API Server

In the configuration file, the API Server is specified as https://localhost:6443. To access it from outside, we need to provide the external IP address of *node1*.

* Cluster’s CA certificate

To provide the cluster CA certificate to Gitlab, we need to decode the one specified in the configuration (as it’s in base 64).

    $ kubectl config view --raw \
    -o=jsonpath='{.clusters[0].cluster.certificate-authority-data}' \
    | base64 --decode

* Service token

The process to get an identification token involves several steps. We first need to create a ServiceAccount and provide it with the cluster-admin role. This can be done with the following command:

    **$ cat <<EOF | kubectl apply -f -**
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: gitlab-admin
      namespace: kube-system
    ---
    apiVersion: rbac.authorization.k8s.io/v1beta1
    kind: ClusterRoleBinding
    metadata:
      name: gitlab-admin
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
    - kind: ServiceAccount
      name: gitlab-admin
      namespace: kube-system
    **EOF**

Once the service account is created, we retrieve the resource of type secret* *that is associated:

    $ SECRET=$(kubectl -n kube-system get secret | grep gitlab-admin | awk '{print $1}')

The next step is to extract the JWT token associated with the secret:

    $ TOKEN=$(kubectl -n kube-system get secret $SECRET -o jsonpath='{.data.token}' | base64 --decode)
    $ echo $TOKEN

We’re all set. Let’s now use all the information and fill the fields of GitLab’s *Add existing cluster* form:

![](https://cdn-images-1.medium.com/max/5416/1*lUSuJF20c1746GlgnG4eBA.png)

Once the cluster is integrated, we can install helm (Kubernetes package manager) directly from the web interface.

![](https://cdn-images-1.medium.com/max/5416/1*cEVsQkPLujdJb0LMCNfLRg.png)

![](https://cdn-images-1.medium.com/max/5416/1*cAIuFtzN9NK_yxpsoVFqAQ.png)

We can now check from the command line that the tiller daemon (helm’s server-side component) is running.

    **$ kubectl get deploy --all-namespaces | grep tiller**
    NAMESPACE           NAME          READY UP-TO-DATE AVAILABLE AGE
    gitlab-managed-apps tiller-deploy 1/1   1          1         67s

The cluster is now ready to use. On top of this, GitLab’s web interface allows one-click installation of additional components:

* Ingress Controller to expose services running within the cluster

* Cert-Manager to manager TLS certificates with Let’s Encrypt

* Prometheus to monitor applications running in the cluster

* Knative to deploy Serverless workload

* Etc.

![](https://cdn-images-1.medium.com/max/5416/1*9aIoVbZ_1uxgxdteW1LotQ.png)

## Summary

In this article, we saw how to create a k3s cluster and integrate it in a GitLab project. Of course, the same process can be used for any Kubernetes cluster.

We can now add resources to the project :

* source code

* Dockerfile to specify how to create a Docker image from the code

* Kubernetes resources such as Deployment, Service, …

* a *.gitlab-ci.yaml* file defining the CI pipeline and how the application should be deployed and tested against the associated Kubernetes cluster

You may find this [previous article](https://medium.com/better-programming/even-the-smallest-side-project-deserves-its-k8s-cluster-3fc6f8a65e13) useful. It provides additional information about how to setup the CI/CD pipeline.

Do you use Kubernetes integration for your GitLab projects?
