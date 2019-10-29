Unknown markup type 10 { type: [33m10[39m, start: [33m313[39m, end: [33m326[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m90[39m, end: [33m94[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m132[39m, end: [33m136[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m81[39m, end: [33m85[39m }

# Configuring minimal RBAC permissions for Helm and Tiller



There are a lot of tutorials on the web demonstrating how to setup and configure Tiller using Role-based access control (RBAC); however, I struggled to find any taking in to account how the Helm client was being executed. Most people seem to be running Helm with their own credentials which usually tends to have cluster-admin permissions. Needless to say, this isn’t very good from a security perspective, especially so if it’s being run within CI/CD.

<iframe src="https://medium.com/media/2254494e93eeb6c8113526d764c0d5cf" frameborder=0></iframe>

***Disclaimer**: This article does get a little technical, I’ve made every attempt to make this as accessible as possible. If you are just starting out with Kubernetes or any of the topics here don’t quite make sense I’d recommend having a look at [https://kubernetes.io/docs/tutorials/](https://kubernetes.io/docs/tutorials/)*

Our team is currently in the process of migrating our existing [Kubernetes](https://kubernetes.io/) deployments to [Helm Charts](https://www.helm.sh/). We provision each development team their own namespace and in each namespace we deploy an instance of Tiller, this allows our developers to own their namespaces. The way we manage this is with [Helmsman](https://github.com/Praqma/helmsman). We have a central repository that defines which Helm Charts we want deployed, the version, the namespace etc. Helmsman parses this and then ensures the Helm Chart releases in the cluster are in line with what is defined.

## Some ‘brief’ information on Helmsman and Helm

![](https://cdn-images-1.medium.com/max/2756/1*CQPzT2paTatzI49VoZ1TiA.png)

Helmsman is a tool which allows you to automate the deployment/management of your Helm charts from version controlled code.

Helm is a package manager for Kubernetes. There are three parts to Helm: The Helm client (helm) the Helm server (Tiller) and the Charts themselves.

**The Helm Client** is a command-line client for end users. The client is responsible for the following domains:

* Local chart development

* Managing repositories

* Interacting with the Tiller server

* Sending charts to be installed

* Asking for information about releases

* Requesting upgrading or uninstalling of existing releases

**The Tiller Server** is an in-cluster server that interacts with the Helm client, and interfaces with the Kubernetes API server. The server is responsible for the following:

* Listening for incoming requests from the Helm client

* Combining a chart and configuration to build a release

* Installing charts into Kubernetes, and then tracking the subsequent release

* Upgrading and uninstalling charts by interacting with Kubernetes

In a nutshell, the client is responsible for managing charts, and the server is responsible for managing releases.

**A Helm Chart** is a collection of files that describe a related set of Kubernetes resources.

## **Requirements:**

* Three namespaces - one for the SRE team and two for dev teams.

* Each namespace except for SRE is to have its own tiller configured with minimum permissions and be restricted to only deploying resources in that namespace.

* Helm is being run inside of CI/CD for the “development” cluster and will also need the minimum possible permissions; however, it will need to be able to access Tiller in both the development namespaces and any additional namespaces that may be created at a later date.

## **RBAC Configuration**

**Namespaces**

Manifest to create three [namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/), “sre”, “dev1” and “dev2”.

    $ mkdir manifests

    $ cat > manifests/namespaces.yaml <<EOF
    kind: Namespace
    apiVersion: v1
    metadata:
      name: sre
    ---
    kind: Namespace
    apiVersion: v1
    metadata:
      name: dev1
    ---
    kind: Namespace
    apiVersion: v1
    metadata:
      name: dev2
    EOF

Create and verify namespaces

    $ kubectl create -f manifests/namespaces.yaml
    namespace/sre created
    namespace/dev1 created
    namespace/dev2 created

    $ kubectl get namespaces
    NAME          STATUS   AGE
    default       Active   12m
    dev1          Active   5s
    dev2          Active   5s
    kube-public   Active   12m
    kube-system   Active   12m
    sre           Active   5s

**Tiller Service Accounts**

Manifest to create two [Service Accounts](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#user-accounts-vs-service-accounts) named “tiller” in the “dev1” and “dev2” namepsaces.

    $ cat > manifests/tiller-sa.yaml <<EOF
    kind: ServiceAccount
    apiVersion: v1
    metadata:
      name: tiller
      namespace: dev1
    ---
    kind: ServiceAccount
    apiVersion: v1
    metadata:
      name: tiller
      namespace: dev2
    EOF

Create and verify Tiller Service Accounts

    $ kubectl create -f manifests/tiller-sa.yaml
    serviceaccount/tiller created
    serviceaccount/tiller created

    $ kubectl get sa --all-namespaces
    NAMESPACE     NAME                                 SECRETS   AGE
    ...
    dev1          tiller                               1         18s
    dev2          tiller                               1         18s
    ...

**Helm Service Account**

Manifest to create a Service Account named “helm” in the “sre” namespace we created previously.

    $ cat > manifests/helm-sa.yaml <<EOF
    kind: ServiceAccount
    apiVersion: v1
    metadata:
      name: helm
      namespace: sre
    EOF

Create and verify the Helm Service Account

    $ kubectl create -f manifests/helm-sa.yaml
    serviceaccount/helm created

    $ kubectl -n sre get sa
    NAME      SECRETS   AGE
    default   1         21m
    helm      1         11s

**Tiller Roles and Rolebindings**

In this manifest we create the [Roles](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole) that will restrict what Tiller can do. We are granting permissions on only the API groups and resources that Tiller needs to deploy and manage releases in its namespace. We then link those Roles to a [Rolebinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding) and to the Tiller Service Account we created previously

    $ cat > manifests/tiller-roles.yaml <<EOF
    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: tiller-manager
      namespace: dev1
    rules:
    - apiGroups: ["", "batch", "extensions", "apps"]
      resources: ["*"]
      verbs: ["*"]
    ---
    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: tiller-manager
      namespace: dev2
    rules:
    - apiGroups: ["", "batch", "extensions", "apps"]
      resources: ["*"]
      verbs: ["*"]
    EOF

    $ cat > manifests/tiller-rolebindings.yaml <<EOF
    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: tiller-binding
      namespace: dev1
    subjects:
    - kind: ServiceAccount
      name: tiller
      namespace: dev1
    roleRef:
      kind: Role
      name: tiller-manager
      apiGroup: rbac.authorization.k8s.io
    ---
    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: tiller-binding
      namespace: dev2
    subjects:
    - kind: ServiceAccount
      name: tiller
      namespace: dev2
    roleRef:
      kind: Role
      name: tiller-manager
      apiGroup: rbac.authorization.k8s.io
    EOF

Create and verify the Tiller Roles and RoleBindings

    $ kubectl create -f manifests/tiller-roles.yaml
    role.rbac.authorization.k8s.io/tiller-manager created
    role.rbac.authorization.k8s.io/tiller-manager created

    $ kubectl create -f manifests/tiller-rolebindings.yaml
    rolebinding.rbac.authorization.k8s.io/tiller-binding created
    rolebinding.rbac.authorization.k8s.io/tiller-binding created

    $ kubectl get roles.rbac.authorization.k8s.io,rolebindings.rbac.authorization.k8s.io -n dev1
    NAME                                                   AGE
    role.rbac.authorization.k8s.io/tiller-manager          2m38s

    NAME                                                   AGE
    rolebinding.rbac.authorization.k8s.io/tiller-binding   66s

    kubectl get roles.rbac.authorization.k8s.io,rolebindings.rbac.authorization.k8s.io -n dev2
    NAME                                                   AGE
    role.rbac.authorization.k8s.io/tiller-manager          2m41s

    NAME                                                   AGE
    rolebinding.rbac.authorization.k8s.io/tiller-binding   69s

**Helm ClusterRole and ClusterRolebindings**

Here we create the ClusterRole that restricts what the Helm Service Account can do, assigned are the minimum privileges required by helm to interact with Tiller. We then create a ClusterRoleBinding which binds the ClusterRole to the Helm Service Account.

    $ cat > manifests/helm-clusterrole.yaml <<EOF
    kind: ClusterRole
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: helm-clusterrole
    rules:
      - apiGroups: [""]
        resources: ["pods/portforward"]
        verbs: ["create"]
      - apiGroups: [""]
        resources: ["pods"]
        verbs: ["list", "get"]
    EOF

    $ cat > manifests/helm-clusterrolebinding.yaml <<EOF
    kind: ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: helm-clusterrolebinding
    roleRef:
      kind: ClusterRole
      apiGroup: rbac.authorization.k8s.io
      name: helm-clusterrole
    subjects:
      - kind: ServiceAccount
        name: helm
        namespace: sre
    EOF

Create and verify the Helm ClusterRole and ClusterRoleBinding

    $ kubectl create -f manifests/helm-clusterrole.yaml
    clusterrole.rbac.authorization.k8s.io/helm-clusterrole created
    $ kubectl create -f manifests/helm-clusterrolebinding.yaml
    clusterrolebinding.rbac.authorization.k8s.io/helm-clusterrolebinding created

    $ kubectl get clusterroles.rbac.authorization.k8s.io,clusterrolebindings.rbac.authorization.k8s.io
    NAME                                                      AGE
    ...
    clusterrole.rbac.authorization.k8s.io/helm-clusterrole    2m12s

    NAME                                                      AGE
    ...
    clusterrolebinding.rbac.authorization.k8s.io/helm-clusterrolebinding 2m7s

## **Deploy Tiller**

Now we have the RBAC configuration out of the way let’s deploy Tiller in to our two development namespaces

    $ helm init --service-account tiller --tiller-namespace dev1
    $ helm init --service-account tiller --tiller-namespace dev2

    $ kubectl get pods --all-namespaces
    NAMESPACE     NAME                                    READY  STATUS    
    dev1          tiller-deploy-565dcc78c-spthn           1/1    Running   
    dev2          tiller-deploy-84886f744-5vtvr           1/1    Running

## **Generate a Kubeconfig file from the Helm Service Account**

We’re going to generate a Kubeconfig file which will be used specifically by the helm client within CI/CD.

Credit to [Ami Mahloof](undefined) for this [script](https://gist.github.com/innovia/fbba8259042f71db98ea8d4ad19bd708#file-kubernetes_add_service_account_kubeconfig-sh) which I slightly modified for this section.

    # Find the secret associated with the Service Account
    $ SECRET=$(kubectl -n sre get sa helm -o jsonpath='{.secrets[].name}')

    # Retrieve the token from the secret and decode it
    $ TOKEN=$(kubectl get secrets -n sre $SECRET -o jsonpath='{.data.token}' | base64 -D)

    # Retrieve the CA from the secret, decode it and write it to disk
    $ kubectl get secrets -n sre $SECRET -o jsonpath='{.data.ca\.crt}' | base64 -D > ca.crt

    # Retrieve the current context
    $ CONTEXT=$(kubectl config current-context)

    # Retrieve the cluster name
    $ CLUSTER_NAME=$(kubectl config get-contexts $CONTEXT --no-headers=true | awk '{print $3}')

    # Retrieve the API endpoint
    $ SERVER=$(kubectl config view -o jsonpath="{.clusters[?(@.name == \"${CLUSTER_NAME}\")].cluster.server}")

    # Set up variables
    $ KUBECONFIG_FILE=config USER=helm CA=ca.crt

    # Set up config
    $ kubectl config set-cluster $CLUSTER_NAME \
    --kubeconfig=$KUBECONFIG_FILE \
    --server=$SERVER \
    --certificate-authority=$CA \
    --embed-certs=true

    # Set token credentials
    $ kubectl config set-credentials \
    $USER \
    --kubeconfig=$KUBECONFIG_FILE \
    --token=$TOKEN

    # Set context entry
    $ kubectl config set-context \
    $USER \
    --kubeconfig=$KUBECONFIG_FILE \
    --cluster=$CLUSTER_NAME \
    --user=$USER

    # Set the current-context
    $ kubectl config use-context $USER \
    --kubeconfig=$KUBECONFIG_FILE
    

## Deploy

Let’s try deploying some applications to our development namespaces using our new Helm Kubeconfig! We’ll install [Prometheus](https://prometheus.io/) in to the dev1 namespace and [Grafana](https://grafana.com/) in to the dev2 namespace.

    $ helm install \
    --name prometheus \
    stable/prometheus \
    --tiller-namespace dev1 \
    --kubeconfig config \
    --namespace dev1 \
    --set rbac.create=false

    NAME:   prometheus
    LAST DEPLOYED: Sun Oct 28 16:22:46 2018
    NAMESPACE: dev1
    STATUS: DEPLOYED

    $ helm install --name grafana \
    stable/grafana \
    --tiller-namespace dev2 \
    --kubeconfig config \
    --namespace dev2 \
    --set rbac.pspEnabled=false \
    --set rbac.create=false

    NAME:   grafana
    LAST DEPLOYED: Sun Oct 28 16:25:18 2018
    NAMESPACE: dev2
    STATUS: DEPLOYED

Looks like it worked! **🙌**

**TLDR**: So, what do we have at the end of this?

* Two instances of Tiller deployed in each of our development namespaces. These are locked down and are only allowed to manage resources in the namespace they reside in.

* A Helm Service Account in the SRE namespace configured with the minimum permissions required to interact with current Tiller deployments, as well as future Tiller deployments in to new namespaces.

* A Kubeconfig file created from the Helm Service Account which can be used within a CI/CD pipeline either with Helm or in conjunction with something like Helmsman which uses Helm under the hood.

All the Kubernetes manifests for setting up this example can be found on my [GitHub](https://github.com/devzx/helm-rbac)

Some useful resources:

* [https://kubernetes.io/docs/reference/access-authn-authz/rbac/](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

* [https://docs.helm.sh/using_helm/#tiller-and-role-based-access-control](https://docs.helm.sh/using_helm/#tiller-and-role-based-access-control)

Massive thanks to Alex Bastable for proofreading.
