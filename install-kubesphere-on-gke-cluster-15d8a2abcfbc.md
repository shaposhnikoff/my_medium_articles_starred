Unknown markup type 10 { type: [33m10[39m, start: [33m116[39m, end: [33m129[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m157[39m, end: [33m194[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m37[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m37[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m30[39m, end: [33m57[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m16[39m, end: [33m30[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m348[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m33[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m109[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m56[39m, end: [33m59[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m53[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m107[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m137[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m586[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m91[39m, end: [33m103[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m7[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m27[39m, end: [33m37[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m50[39m, end: [33m54[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m87[39m, end: [33m95[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m98[39m, end: [33m110[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m107[39m, end: [33m122[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m151[39m, end: [33m156[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m55[39m, end: [33m71[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m51[39m }

# Install KubeSphere on GKE cluster



This guide walks you throungh the steps of KubeSphere minimal installation on [**Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/).**

## What is KubeSphere

[KubeSphere](https://github.com/kubesphere/kubesphere) is an enterprise-grade multi-tenant **container platform** that built on [Kubernetes](https://kubernetes.io/) , **it’s an open source project that supports installing on Linux and Kubernetes** . It provides an easy-to-use UI for users to manage Kubernetes resources with a few clicks, which reduces the learning curve and empowers the DevOps teams. It greatly reduces the complexity of the daily work of development, testing, operation, and maintenance, aiming to alleviate the pain points of Kubernetes’ storage, network, security and ease of use, etc.

[https://youtu.be/u5lQvhi_Xlc](https://youtu.be/u5lQvhi_Xlc)

## Prepare a GKE cluster

At first, a standard Kubernetes in GKE is a prerequisite of installing KubeSphere, we’ve created a GKE cluster with 1.14.8-gke.17in this demo, and chose the n1-standard-2 (2 vCPU, 7.5 GB memory)and 3 nodes in Machine configuration.

***Note:***
> n1-standard-2 (2 vCPU, 7.5 GB memory)is only the minimal requirements that is used for the minimal installation, it's recommended to choose higher machine configuration for production environment.
> n1-standard-2 (2 vCPU, 7.5 GB memory)is only used for minimal installation, KubeSphere 2.1 has decoupled several feature components, which supports installing these pluggable components in an easy way, you have to prepare enough machine configuration before you enable pluggable components, see [**Enabling pluggable components installation](https://kubesphere.io/docs/installation/pluggable-components/)**.
> Supported Kubernetes version: 1.13.0 ≤ K8s version < 1.16.

![](https://cdn-images-1.medium.com/max/5404/1*05W3wcZKCbT1W1mPgGBk5g.png)

![](https://cdn-images-1.medium.com/max/5384/1*gRgokAkiCTftBb4bfSUJuA.png)

## Create Tiller Service Account

KubeSphere requires [**Helm](https://v2.helm.sh/)** (>= v2.10.0, excluding v2.16.0) to trigger the installation. By default, [**Tiller](https://v2.helm.sh/)** is not ready on GKE, thus we need to install Tiller in advance.

When GKE cluster is ready, we can connect to Cloud Shell.

![](https://cdn-images-1.medium.com/max/5200/0*DMaBCj0ebEHalQnn.png)

Here, we create helm-rbac.yamlin GKE as following:

    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: tiller
      namespace: kube-system
    ---
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: tiller
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
      - kind: ServiceAccount
        name: tiller
        namespace: kube-system

Let’s create these resources using kubectl:

    $ kubectl apply -f helm-rbac.yaml

## Deploy Tiller

Initialize helm using the following command.

    $ helm init --service-account=tiller --tiller-image=gcr.io/kubernetes-helm/tiller:v2.14.1   --history-max 300

Check the Tiller status using kubectl, when it displays 1/1that means you are ready to continue.

    $ kubectl get deployment tiller-deploy -n kube-system

## Install KubeSphere

Install KubeSphere using kubectl, this command only triggers the minimal installation by default:

    $ kubectl apply -f [https://raw.githubusercontent.com/kubesphere/ks-installer/master/kubesphere-minimal.yaml](https://raw.githubusercontent.com/kubesphere/ks-installer/master/kubesphere-minimal.yaml)

Verify the real-time logs, when you see the following outputs, congratulation! You can access KubeSphere in your browser.

    $ kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
    

    #####################################################
    ###              Welcome to KubeSphere!           ###
    #####################################################
    Console: http://10.128.0.34:30880
    Account: admin
    Password: P@88w0rd
    NOTES：
      1. After logging into the console, please check the
         monitoring status of service components in
         the "Cluster Status". If the service is not
         ready, please wait patiently. You can start
         to use when all components are ready.
      2. Please modify the default password after login.
    #####################################################

## Access KubeSphere console

In this guide, we’ll show you how to access KubeSphere console by changing service type to LoadBalancer.

![](https://cdn-images-1.medium.com/max/4684/1*vSjSRlvy-Qx5mpL4CjSeNw.png)

Select Services & Ingress> ks-console, then click EDITand modify the service type from NodePortto LoadBalancer.

![](https://cdn-images-1.medium.com/max/4864/1*caKL0NGfnVqsOUumD8sjmw.png)

Now, you can access the KubeSphere Console using the Endpoints that were generated by GKE.

![](https://cdn-images-1.medium.com/max/5200/0*YhEUW4aG5fW6-g1o.png)
> *Note: In addition to changing the service type to LoadBalancer, you can also access KubeSphere console via NodeIP:NodePort, you may need to allow port 30880in firewall rules.*

Log in to KubeSphere console using the default account admin / P@88w0rd, you'll be able to see its dashboard as following screenshots.

![](https://cdn-images-1.medium.com/max/5704/1*gZF7o-411-AEvl_zjsv4Xg.png)

![](https://cdn-images-1.medium.com/max/5720/1*FSoRaazRgoRSwkyD_7jfKw.png)

![](https://cdn-images-1.medium.com/max/5360/1*yEG1LVqepGXrZhlFzAiD1Q.png)

## Enable Pluggable Components

The above installation is only used for minimal installation by default, execute the following command to enable more pluggable components installation, make sure your cluster has enough CPU and memory in advance, see [**Configuration Table](https://github.com/kubesphere/ks-installer/blob/master/README.md#configuration-table)**.

For enabling DevOps, OpenPitrix and etcd monitoring installation, you have to create CA and etcd certificates in advance, see [**ks-installer](https://github.com/kubesphere/ks-installer/blob/master/README.md#enable-pluggable-components)** for a complete guide.

    $ kubectl edit cm -n kubesphere-system ks-installer

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
