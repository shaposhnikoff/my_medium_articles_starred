Unknown markup type 10 { type: [33m10[39m, start: [33m124[39m, end: [33m135[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m191[39m, end: [33m223[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m148[39m, end: [33m161[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m144[39m, end: [33m156[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m161[39m, end: [33m200[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m104[39m, end: [33m122[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m132[39m, end: [33m142[39m }

# Configuring RBAC For Your Kubernetes Service Accounts

In my previous article, I created a Service Account, and used its token and ca.crt file to access the Kuberentes API.
[**Accessing The Kubernetes API, sans The Proxy**
*For various reasons, I wanted to access the Kubernetes API from outside of the cluster, but I didn’t want to (couldn’t…*medium.com](https://medium.com/@lestrrat/accessing-the-kubernetes-api-sans-the-proxy-b24af1eb18a4)
[**Kubernetes: Up and Running; Dive into the Future of Infrastructure**
*Amazon配送商品ならKubernetes: Up and Running; Dive into the Future of Infrastructureが通常配送無料。更にAmazonならポイント還元本が多数。Kelsey…*www.amazon.co.jp](https://www.amazon.co.jp/gp/product/1491935677/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=d604-22&creative=1211&linkCode=as2&creativeASIN=1491935677&linkId=3fc338d8307638e3f2c19e6b90b3194f)

That code relied on using the legacy authorization system, actually. If you want better authorization system, you really want to use the Role Based Access Control, which was introduced in Kubernetes 1.6.

In this memo, I’m going to record what I did to enable RBAC and allow minimal resources to be accessed by the service account.

## Configuring The Environment

First things first: I use Google Container Engine (GKE).
[**Google Container Engine (GKE) for Kubernetes and containers | Google Cloud Platform**
*Google Container Engine is a powerful cluster manager and orchestration system for running your Docker containers. Set…*cloud.google.com](https://cloud.google.com/container-engine/)

As of this writing (Sep 2017), GKE’s clusters do not by default enable RBAC. You must explicitly ask for it by 1. Using the gcloud beta version of the container command, and 2. providing the --no-enable-legacy-authorization :

    $ gcloud beta container clusters create --no-enable-legacy-authorization ...

Also, you will need to give your GCP user account explicit permission to create Roles and RoleBindings (among other things), by giving yourself the cluster-admin role.

    $ ACCOUNT=$(gcloud info --format='value(config.account)')
    $ kubectl create clusterrolebinding owner-cluster-admin-binding \
        --clusterrole cluster-admin \
        --user $ACCOUNT

Without this, creating Roles/ClusterRoles/RoleBindings/ClusterRoleBindings may give you errors.

I think I should note that this information on having to give permission to your account was only found in the CoreOS troubleshooting guide. I may have overlooked something, but seriously, without this information, I was stuck. Thank you so much, CoreOS!
[**CoreOS**
*CoreOS provides Container Linux, Tectonic for Kubernetes and the Quay image registry; key components to secure…*coreos.com](https://coreos.com/operators/prometheus/docs/latest/troubleshooting.html)

## Roles And Bindings

Now, we’re ready to actually configure authorization for the service account.

In order to configure what resources a Service Account may access (and how), you need two extra things, which are Roles, and RoleBindings.

Roles define the authorization/capability that is to be applied to a Service Account, User, Group, etc. Here’s a role that allows you to access /api/v1/pods and /api/v1/namespace/default/pod/$pod-name.

    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
      namespace: default
      name: pod-reader
    rules:
    - apiGroups: [""] # "" indicates the core API group
      resources: ["pods"]
      verbs: ["get", "list"]

RoleBindings associates roles to Service Accounts, Users, Groups, etc. Here’s a role binding that binds my-service-account with the pod-reader above.

    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
      name: pod-reader-binding
      namespace: default
    subjects:
    - kind: ServiceAccount
      name: my-service-account
      namespace: default
    roleRef:
      kind: Role
      name: pod-reader
      apiGroup: rbac.authorization.k8s.io

Now your service account should be able to access *only* the pod related resources, and you should only be able to view them.

## ClusterRoles and ClusterRoleBindings

Further adding to the confusion, on top of Roles and RoleBindings, there are ClusterRoles and ClusterRoleBindings.

Roles can only refer to resources in a specific *namespace*. Namespaces are logical grouping of resources within a Kubernetes cluster. This means that if you want to, for example, access resources such as Nodes which are global to a cluster and are not namespaced, Roles may not be used to grant such authorization. If, for example, you need to allow Prometheus to access the nodes’ statistics, you need to use ClusterRoles.

ClusterRoleBindings and RoleBindings can almost be used interchangeably, but you cannot assign ClusterRole capabilities using RoleBindings. Confusing? You bet!

Here’s my attempt at explaining how it works. Let’s say you create a ClusterRole that allows access to all pods, by replacing Role to ClusterRole, and omitting the “namespace” parameter in the previous example.

    kind: ClusterRole
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
      name: pod-reader
    rules:
    - apiGroups: [""] # "" indicates the core API group
      resources: ["pods"]
      verbs: ["get", "list"]

If you use **RoleBinding** to bind the service account to this ClusterRole (using the previously example), the service account will only be able to view pods in the “default” namespace.

However, if you change this to a **ClusterRoleBinding**, the service account will be able to view *all* pods in the cluster.

    kind: ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
      name: pod-reader-binding
    subjects:
    - kind: ServiceAccount
      name: my-service-account
      namespace: default
    roleRef:
      kind: ClusterRole
      name: pod-reader
      apiGroup: rbac.authorization.k8s.io

That should be all that you need! Now you can create service accounts with minimum authorization to access only what’s needed from your Kubernetes cluster.

Have fun hacking!
