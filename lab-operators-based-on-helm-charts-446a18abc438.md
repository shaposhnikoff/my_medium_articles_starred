
# Lab - Operators based on Helm charts

Overview

Helm is a popular package manager for Kubernetes. Helm makes some of the Kubernetes administrative tasks easier, such as installation and simple upgrades of some stateless applications but, when it comes to stateful applications, Helm cannot cover other more complex tasks, for example upgrading an application end-to-end.

![](https://cdn-images-1.medium.com/max/2320/1*BRDE-tKhpbYbo64utwP_og.png)

Imagine that you want to upgrade your MariaDB database, Helm could help with upgrading the database bits (using a new image) but probably you would also need to upgrade the database schema, and that’s where Helm is not such useful because you need to export, change and re-import the database and do it in a repeatable and controlled manner.

Helm Operator enables pairing a Helm Chart with the operational knowledge of installing, upgrading and managing the application on Kubernetes clusters. The Operator SDK can create an operator based on a Helm Chart and essentially allow enriching the Helm Chart capabilities by delivering the expertise of managing and running the application together with the application.

There is also an additional advantage of Helm based operators compared to traditional Helm installations when using Helm operators you do not require Tiller to be running in the cluster (Helm 3 could change that since probably it will be Tiller-less but in traditional Helm deployment is a requirement), so permissions of the controller’s service account can be more restrictive since the controller only manages installations of a single type of software, not all installations. You can use Kubernetes RBAC to restrict who can install that software using the operator.

## A “Hello World” kind example

For this lab we’ll be using [code-ready containers](https://github.com/code-ready/crc) project, which creates a local single-VM Openshift 4 cluster on your laptop. You also have to have the [operator-sdk installed in your system](https://github.com/operator-framework/operator-sdk/blob/master/doc/user/install-operator-sdk.md) which takes care of bootstrapping, building and packaging the operator.

There are other pre-requirements that must be met in your system, as you can see in the [Helm user guide for Operator SDK](https://github.com/operator-framework/operator-sdk/blob/master/doc/helm/user-guide.md):

* [git](https://git-scm.com/downloads)

* [docker](https://docs.docker.com/install/) version 17.03+.

* [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) version v1.11.3+ (or oc client).

* [dep](https://golang.github.io/dep/docs/installation.html) version v0.5.0+. (Optional if you aren’t installing from source)

* [go](https://golang.org/dl/) version v1.12+. (Optional if you aren’t installing from source)

* Access to a Kubernetes v1.11.3+ cluster.

## Check your environment readiness

Start your crc environment if it’s not already up.

    $ crc start

    INFO Checking if running as non-root

    INFO Checking if oc binary is cached

    INFO Checking if Virtualization is enabled

    INFO Checking if KVM is enabled

    INFO Checking if libvirt is installed

    INFO Checking if user is part of libvirt group

    INFO Checking if libvirt is enabled

    INFO Checking if libvirt daemon is running

    INFO Checking if crc-driver-libvirt is installed

    INFO Checking if libvirt ‘crc’ network is available

    INFO Checking if libvirt ‘crc’ network is active

    INFO Checking if NetworkManager is installed

    INFO Checking if NetworkManager service is running

    INFO Checking if /etc/NetworkManager/conf.d/crc-nm-dnsmasq.conf exists

    INFO Checking if /etc/NetworkManager/dnsmasq.d/crc.conf exists

    INFO Starting stopped VM …

    INFO Verifying validity of the cluster certificates …

    INFO Check internal and public dns query …

    INFO Starting OpenShift cluster … [waiting 3m]

    INFO

    INFO To access the cluster, first set up your environment by following ‘crc oc-env’ instructions

    INFO Then you can access it by running ‘oc login -u developer -p developer [https://api.crc.testing:6443'](https://api.crc.testing:6443')

    INFO To login as an admin, username is ‘kubeadmin’ and password is BMLkR-NjA28-v7exC-8bwAk

    INFO

    INFO These credentials can also be used to access the OpenShift web console at [https://console-openshift-console.apps-crc.testing](https://console-openshift-console.apps-crc.testing)

    CodeReady Containers instance is running

Check that you have access to your cluster using the oc command:

    $ alias oc=”/home/larizmen/.crc/bin/oc”

    $ oc login -u kubeadmin -p BMLkR-NjA28-v7exC-8bwAk [https://api.crc.testing:6443](https://api.crc.testing:6443)

    Login successful.

    You have access to the following projects and can switch between them with ‘oc project <projectname>’:

    * default

    kube-public

    kube-system

    openshift

    openshift-apiserver

    openshift-apiserver-operator

    openshift-authentication

    openshift-authentication-operator

    openshift-cloud-credential-operator

    openshift-cluster-machine-approver

    openshift-cluster-node-tuning-operator

    openshift-cluster-samples-operator

    openshift-cluster-storage-operator

    openshift-cluster-version

    openshift-config

    openshift-config-managed

    openshift-console

    openshift-console-operator

    openshift-controller-manager

    openshift-controller-manager-operator

    openshift-dns

    openshift-dns-operator

    openshift-etcd

    openshift-image-registry

    openshift-infra

    openshift-ingress

    openshift-ingress-operator

    openshift-kube-apiserver

    openshift-kube-apiserver-operator

    openshift-kube-controller-manager

    openshift-kube-controller-manager-operator

    openshift-kube-scheduler

    openshift-kube-scheduler-operator

    openshift-machine-api

    openshift-machine-config-operator

    openshift-marketplace

    openshift-monitoring

    openshift-multus

    openshift-network-operator

    openshift-node

    openshift-operator-lifecycle-manager

    openshift-operators

    openshift-sdn

    openshift-service-ca

    openshift-service-ca-operator

    openshift-service-catalog-apiserver-operator

    openshift-service-catalog-controller-manager-operator

    Using project “default”.

Check that you are able to use the Helm chart that you will be the base to create the operator. This step is not mandatory but it’s a good idea to run through it if the chart that you selected is not widely used, so you can be sure that everything it’s working from that point of view. In our example, we’ll use a well know mariadb chart but nevertheless we’ll continue with this test. If not already done, [Helm needs to be initialized](https://helm.sh/docs/using_helm/) in both the client and cluster, that will install Tiller on top of the cluster which, as commented before, it is not needed to run operators based in Helm charts, so at the end of this test we’ll remove it from our cluster.

    $ helm init — history-max 200

    $HELM_HOME has been configured at /home/larizmen/.helm.

    Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

    Please note: by default, Tiller is deployed with an insecure ‘allow unauthenticated users’ policy.

    To prevent this, run `helm init` with the — tiller-tls-verify flag.

    For more information on securing your installation see: [https://docs.helm.sh/using_helm/#securing-your-helm-installation](https://docs.helm.sh/using_helm/#securing-your-helm-installation)

    $ helm repo update

    Hang tight while we grab the latest from your chart repositories…

    …Skip local chart repository

    …Successfully got an update from the “stable” chart repository

    Update Complete.

Try to use a helm chart, for example, the mysql chart:

    $ helm install stable/mysql

    NAME: brazen-lamb

    LAST DEPLOYED: Tue Oct 1 11:52:40 2019

    NAMESPACE: default

    STATUS: DEPLOYED

    RESOURCES:

    ==> v1/ConfigMap

    NAME DATA AGE

    brazen-lamb-mysql-test 1 0s

    ==> v1/PersistentVolumeClaim

    NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE

    brazen-lamb-mysql Bound pv0019 10Gi RWO,ROX,RWX 0s

    ==> v1/Pod(related)

    NAME READY STATUS RESTARTS AGE

    brazen-lamb-mysql-84b4cccc85–78tfr 0/1 Init:0/1 0 0s

    ==> v1/Secret

    NAME TYPE DATA AGE

    brazen-lamb-mysql Opaque 2 0s

    ==> v1/Service

    NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE

    brazen-lamb-mysql ClusterIP 172.30.65.117 <none> 3306/TCP 0s

    ==> v1beta1/Deployment

    NAME READY UP-TO-DATE AVAILABLE AGE

    brazen-lamb-mysql 0/1 1 0 0s

    NOTES:

    MySQL can be accessed via port 3306 on the following DNS name from within your cluster:

    brazen-lamb-mysql.default.svc.cluster.local

    To get your root password run:

    MYSQL_ROOT_PASSWORD=$(kubectl get secret — namespace default brazen-lamb-mysql -o jsonpath=”{.data.mysql-root-password}” | base64 — decode; echo)

    To connect to your database:

    1. Run an Ubuntu pod that you can use as a client:

    kubectl run -i — tty ubuntu — image=ubuntu:16.04 — restart=Never — bash -il

    2. Install the mysql client:

    $ apt-get update && apt-get install mysql-client -y

    3. Connect using the mysql cli, then provide your password:

    $ mysql -h brazen-lamb-mysql -p

    To connect to your database directly from outside the K8s cluster:

    MYSQL_HOST=127.0.0.1

    MYSQL_PORT=3306

    # Execute the following command to route the connection:

    kubectl port-forward svc/brazen-lamb-mysql 3306

    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}

It could happen that you find this error:

    $ helm install stable/mysql

    Error: no available release name found

If so, you can solve it creating the service account and binding the role to it:

    $ kubectl create serviceaccount — namespace kube-system tiller

    $ kubectl create clusterrolebinding tiller-cluster-rule — clusterrole=cluster-admin — serviceaccount=kube-system:tiller

    $ kubectl patch deploy — namespace kube-system tiller-deploy -p ‘{“spec”:{“template”:{“spec”:{“serviceAccount”:”tiller”}}}}’

Once the helm chart finished, try to use the resource deployed with it:

    $ kubectl get secret — namespace default brazen-lamb-mysql -o jsonpath=”{.data.mysql-root-password}” | base64 — decode; echo

    EWTw4XWHal

    $ oc get pod

    NAME READY STATUS RESTARTS AGE

    brazen-lamb-mysql-84b4cccc85–78tfr 1/1 Running 0 8m19s

    $ oc rsh brazen-lamb-mysql-84b4cccc85–78tfr

    # mysql -h 127.0.0.1 -P3306 -u root -pEWTw4XWHal

    mysql: [Warning] Using a password on the command line interface can be insecure.

    Welcome to the MySQL monitor. Commands end with ; or \g.

    Your MySQL connection id is 68

    Server version: 5.7.14 MySQL Community Server (GPL)

    Copyright © 2000, 2016, Oracle and/or its affiliates. All rights reserved.

    Oracle is a registered trademark of Oracle Corporation and/or its

    affiliates. Other names may be trademarks of their respective

    owners.

    Type ‘help;’ or ‘\h’ for help. Type ‘\c’ to clear the current input statement.

    mysql>

    mysql> show databases;

    + — — — — — — — — — — +

    | Database |

    + — — — — — — — — — — +

    | information_schema |

    | mysql |

    | performance_schema |

    | sys |

    + — — — — — — — — — — +

    4 rows in set (0.00 sec)

    mysql>

Exit from the container and destroy the resources created by Helm in this test.

    $ helm ls

    NAME REVISION UPDATED STATUS CHART APP VERSION NAMESPACE

    brazen-lamb 1 Tue Oct 1 11:52:40 2019 DEPLOYED mysql-1.3.2 5.7.14 default

    $ helm delete — purge brazen-lamb

    release “brazen-lamb” deleted

As the final step, remove Tiller from the cluster

    $ oc get deploy -n kube-system

    NAME READY UP-TO-DATE AVAILABLE AGE

    tiller-deploy 1/1 1 1 169m

    $ helm reset — force

    Tiller (the Helm server-side component) has been uninstalled from your Kubernetes Cluster.

    $ oc get deploy -n kube-system

    No resources found in kube-system namespace.

## Build an Operator from an existing chart

First, let’s create the directory with everything required by the operator, be sure that you indicate that it will be an operator created from a Helm chart — type=helm and indicate the actual helm chart to be used with — helm-chart. As you can see in [Helm user guide for Operator SDK](https://github.com/operator-framework/operator-sdk/blob/master/doc/helm/user-guide.md) you could also change the default operator name with — kind and api version with — api-version but those become optional if — helm-chart is used. If left unset, the SDK will default — api-version to charts.helm.k8s.io/v1alpha1 and will deduce — kind from the specified chart.

    $ operator-sdk new mariadb-operator — type=helm — helm-chart stable/mariadb

    INFO[0000] Creating new Helm operator ‘mariadb-operator’.

    INFO[0001] Created helm-charts/mariadb

    INFO[0001] Generating RBAC rules

    WARN[0001] The RBAC rules generated in deploy/role.yaml are based on the chart’s default manifest. Some rules may be missing for resources that are only enabled with custom values, and some existing rules may be overly broad. Double check the rules generated in deploy/role.yaml to ensure they meet the operator’s permission requirements.

    INFO[0001] Created build/Dockerfile

    INFO[0001] Created watches.yaml

    INFO[0001] Created deploy/service_account.yaml

    INFO[0001] Created deploy/role.yaml

    INFO[0001] Created deploy/role_binding.yaml

    INFO[0001] Created deploy/operator.yaml

    INFO[0001] Created deploy/crds/charts_v1alpha1_mariadb_crd.yaml

    INFO[0001] Created deploy/crds/charts_v1alpha1_mariadb_cr.yaml

    INFO[0001] Run git init …

    Initialized empty Git repository in /home/larizmen/helm-operator/mariadb-operator/.git/

    INFO[0001] Run git init done

    INFO[0001] Project creation complete.

You can check how the previous command generated a directory structure with everything required to run: Dockerfile, Kubernetes objects, and the actual chart :

    $ tree mariadb-operator/

    mariadb-operator/

    ├── build

    │ └── Dockerfile

    ├── deploy

    │ ├── crds

    │ │ ├── charts_v1alpha1_mariadb_crd.yaml

    │ │ └── charts_v1alpha1_mariadb_cr.yaml

    │ ├── operator.yaml

    │ ├── role_binding.yaml

    │ ├── role.yaml

    │ └── service_account.yaml

    ├── helm-charts

    │ └── mariadb

    │ ├── charts

    │ ├── Chart.yaml

    │ ├── files

    │ │ └── docker-entrypoint-initdb.d

    │ │ └── README.md

    │ ├── OWNERS

    │ ├── README.md

    │ ├── templates

    │ │ ├── _helpers.tpl

    │ │ ├── initialization-configmap.yaml

    │ │ ├── master-configmap.yaml

    │ │ ├── master-pdb.yaml

    │ │ ├── master-statefulset.yaml

    │ │ ├── master-svc.yaml

    │ │ ├── NOTES.txt

    │ │ ├── rolebinding.yaml

    │ │ ├── role.yaml

    │ │ ├── secrets.yaml

    │ │ ├── serviceaccount.yaml

    │ │ ├── servicemonitor.yaml

    │ │ ├── slave-configmap.yaml

    │ │ ├── slave-pdb.yaml

    │ │ ├── slave-statefulset.yaml

    │ │ ├── slave-svc.yaml

    │ │ ├── test-runner.yaml

    │ │ ├── tests

    │ │ └── tests.yaml

    │ ├── values-production.yaml

    │ └── values.yaml

    └── watches.yaml

    10 directories, 32 files

Pay attention to the RBAC rules that will be created and double-check that they meet the operator requirements:

    $ cat mariadb-operator/deploy/role.yaml

    apiVersion: rbac.authorization.k8s.io/v1

    kind: Role

    metadata:

    creationTimestamp: null

    name: mariadb-operator

    rules:

    - apiGroups:

    - “”

    resources:

    - namespaces

    verbs:

    - get

    - apiGroups:

    - “”

    resources:

    - configmaps

    - secrets

    verbs:

    - ‘*’

    - apiGroups:

    - “”

    resources:

    - configmaps

    - pods

    - secrets

    - services

    verbs:

    - ‘*’

    - apiGroups:

    - apps

    resources:

    - statefulsets

    verbs:

    - ‘*’

    - apiGroups:

    - myexamples.com

    resources:

    - ‘*’

    verbs:

    - ‘*’

Next, you have to create and push the container image for your operator. Let’s check the Dockerfile created by the operator-sdk

    $ cat mariadb-operator/build/Dockerfile

    FROM quay.io/operator-framework/helm-operator:v0.8.1

    COPY watches.yaml ${HOME}/watches.yaml

    COPY helm-charts/ ${HOME}/helm-charts/

As you can see, it’s using the quay.io/operator-framework/helm-operator:v0.8.1 image as a base, and if you check [the manifest](https://quay.io/repository/operator-framework/helm-operator/manifest/sha256:e8cf4ee8c803c48fce4de8172bcac70b8d48420d66d45d3e5e12a72370048f4b) you can see how this image already includes the binary /usr/local/bin/helm-operator used to manage the helm chart which is maintained by the operator SDK team.

    $ cd mariadb-operator

Be sure that you have privileges to run docker build, in this case, we’ll be using sudo:

    $ sudo operator-sdk build mariadb-operator:v0.1

    INFO[0000] Building OCI image mariadb-operator:v0.1

    Sending build context to Docker daemon 290.8 kB

    Step 1/3 : FROM quay.io/operator-framework/helm-operator:v0.8.1

    Trying to pull repository quay.io/operator-framework/helm-operator …

    sha256:e8cf4ee8c803c48fce4de8172bcac70b8d48420d66d45d3e5e12a72370048f4b: Pulling from quay.io/operator-framework/helm-operator

    27402aa444ba: Pull complete

    50495ac9ea81: Pull complete

    a6bd365e4488: Pull complete

    b0af98884131: Pull complete

    c07440d205bd: Pull complete

    Digest: sha256:e8cf4ee8c803c48fce4de8172bcac70b8d48420d66d45d3e5e12a72370048f4b

    Status: Downloaded newer image for quay.io/operator-framework/helm-operator:v0.8.1

     — -> 197ba475d102

    Step 2/3 : COPY watches.yaml ${HOME}/watches.yaml

     — -> c611c7ad09b4

    Removing intermediate container c62ceca8e6b8

    Step 3/3 : COPY helm-charts/ ${HOME}/helm-charts/

     — -> 961127abb70c

    Removing intermediate container 13a76d6e2e2d

    Successfully built 961127abb70c

    INFO[0022] Operator build complete.

If you find an error like the next one, be sure that you have docker installed in your system and that the docker daemon is running:

    $ operator-sdk build mariadb-operator:v0.1

    INFO[0000] Building OCI image mariadb-operator:v0.1

    Error: failed to output build image mariadb-operator:v0.1: (failed to exec []string{“docker”, “build”, “-f”, “build/Dockerfile”, “-t”, “mariadb-operator:v0.1”, “.”}: exec: “docker”: executable file not found in $PATH)

Now the image is ready locally :

    $ sudo docker images

    REPOSITORY TAG IMAGE ID CREATED SIZE

    mariadb-operator v0.1 961127abb70c About a minute ago 153 MB

    quay.io/operator-framework/helm-operator v0.8.1 197ba475d102 4 months ago 153 MB

Next step is to push the image into the registry, in this case in Quay.io with my personal account:

    $ sudo docker tag mariadb-operator:v0.1 quay.io/luisarizmendi/mariadb-operator:v0.1

    $ sudo docker login quay.io

    Username: luisarizmendi

    Password:

    Login Succeeded

    $ sudo docker push quay.io/luisarizmendi/mariadb-operator:v0.1

    The push refers to a repository [quay.io/luisarizmendi/mariadb-operator]

    7c9993053dd3: Pushed

    06aba11a8818: Pushed

    f5f564a6c664: Pushed

    a9a9f6c83b12: Pushed

    d0bf58423527: Pushed

    97327e241a2a: Pushed

    5cb7ea77f481: Pushed

    v0.1: digest: sha256:6f7048b7d230cae54ca9a36ecd1679e2c1b72696ae86f67d9301e79f9a683721 size: 9227

## Install the Operator

There are two ways of installing the new operator, with or without the usage of the [Operator Lifecycle Manager (OLM)](https://docs.openshift.com/container-platform/4.1/applications/operators/olm-understanding-olm.html). As you can imagine, the preferred way to do it if using an OpenShift 4 cluster is using OLM, since it makes it easier to install, upgrade, and grant access to Operators running on the cluster, and that’s why this will be the method explained in the upcoming steps.

However, you could skip OLM usage by just creating directly all resources from the deploy directory (oc create -f deploy/) after changing the image name in the deploy/operator.yaml file.

Let’s review the OLM option. Before moving forward, OLM needs to create some files inside the directory to define the ClusterServiceVersion that represents the CRDs your Operator uses, the permissions it requires to function and other installation information:

    $ operator-sdk olm-catalog gen-csv — csv-version 0.0.1

    INFO[0000] Generating CSV manifest version 0.0.1

    INFO[0000] Fill in the following required fields in file deploy/olm-catalog/mariadb-operator/0.0.1/mariadb-operator.v0.0.1.clusterserviceversion.yaml:

    spec.keywords

    spec.maintainers

    spec.provider

    INFO[0000] Created deploy/olm-catalog/mariadb-perator/0.0.1/mariadb-operator.v0.0.1.clusterserviceversion.yaml

We need to modify some of those recently created files according to our needs, mainly including the container image that we created in previous steps and the namespace, let’s start with the image name:

    $ cat deploy/olm-catalog/mariadb-operator/0.0.1/mariadb-operator.v0.0.1.clusterserviceversion.yaml | grep “”image:

    **image:** REPLACE_IMAGE

    $ sed -i ‘s/REPLACE_IMAGE/quay.io\/luisarizmendi\/mariadb-operator:v0.1/’ deploy/olm-catalog/mariadb-operator/0.0.1/mariadb-operator.v0.0.1.clusterserviceversion.yaml

    $ cat deploy/olm-catalog/mariadb-operator/0.0.1/mariadb-operator.v0.0.1.clusterserviceversion.yaml | grep image:

    **image:** quay.io/luisarizmendi/mariadb-operator:v0.1

This operator example will be namespace-scoped but you can also create cluster-scoped operators with [some changes in the CDRs](https://github.com/operator-framework/operator-sdk/blob/master/doc/operator-scope.md).

As a next step, let’s configure the namespace used to deploy the operator, although you can use a custom namespace for such matter, we’ll be deploying our operator inside the openshift-operators since is easiest because it is already set up for us to grant access to this Operator for all users of the cluster:

    $ sed -i ‘s/namespace: placeholder/namespace: openshift-operators/’ deploy/olm-catalog/mariadb-operator/0.0.1/mariadb-operator.v0.0.1.clusterserviceversion.yaml

You have to customize the displayName and the Description of the CRD to be created in the same file.

    $ sed -i ‘s#mariadbs.charts.helm.k8s.io#mariadbs.charts.helm.k8s.io\n displayName: MariaDB\n description: MariaDB custom description#g’ deploy/olm-catalog/mariadb-operator/0.0.1/mariadb-operator.v0.0.1.clusterserviceversion.yaml

Moving forward, the next step is to deploy all needed CDRs in the cluster:

    $ oc project openshift-operators

    Now using project “openshift-operators” on server “https://api.crc.testing:6443".

    $ oc create -f deploy/crds/charts_v1alpha1_mariadb_crd.yaml

    customresourcedefinition.apiextensions.k8s.io/mariadbs.charts.helm.k8s.io created

    $ oc create -f deploy/service_account.yaml

    serviceaccount/mariadb-operator created

    $ oc create -f deploy/role_binding.yaml

    rolebinding.rbac.authorization.k8s.io/mariadb-operator created

    $ oc create -f deploy/role.yaml

    role.rbac.authorization.k8s.io/mariadb-operator created

    $ oc create -f deploy/olm-catalog/mariadb-operator/0.0.1/mariadb-operator.v0.0.1.clusterserviceversion.yaml

    clusterserviceversion.operators.coreos.com/mariadb-operator.v0.0.1 created

At this point, the operator should be ready to be used, check that the operator pod is running in your cluster:

    $ oc get pod -n openshift-operators

    NAME READY STATUS RESTARTS AGE

    mariadb-operator-76f7b69db8–86rg8 1/1 Running 0 3m34s

If you get a ImagePullBackOf error, then double-check that your registry is public and reachable by the cluster.

## Deploy an instance of the chart

Now it’s time to use the operator. Create the new CustomResource file that will be used to define your deployment. Remember that Helm uses a concept called [values](https://helm.sh/docs/using_helm/#customizing-the-chart-before-installing) to provide customizations to a Helm chart’s defaults, which are defined in the Helm chart’s values.yaml file, but in order to override these defaults you will need to set the desired values in this CustomResource file.

Check the object definition already created by operator-sdk, review the options and create your own definition file with your customizations (or feel free to use that file with the default values to run a quick test):

    $ more deploy/crds/charts_v1alpha1_mariadb_cr.yaml

    $ cat > ~/helmoperatorexample.yaml << EOF

    apiVersion: charts.helm.k8s.io/v1alpha1

    kind: Mariadb

    metadata:

    name: my-mariadb

    spec:

    rootUser:

    password: redhat

    EOF

    $ oc create -f ~/helmoperatorexample.yaml

    mariadb.charts.helm.k8s.io/my-mariadb created

At this moment, you can check how the custom resource has been created:

    $ oc get mariadbs

    NAME AGE

    my-mariadb 21s

Check that the operator has created new pods (among other objects):

    $ oc get pod

    NAME READY STATUS RESTARTS AGE

    mariadb-operator-76f7b69db8-z89xb 1/1 Running 1 15h

    my-mariadb-5jlezw5gmrrxivn1a28ux11e-master-0 1/1 Running 0 2m42s

    my-mariadb-5jlezw5gmrrxivn1a28ux11e-slave-0 1/1 Running 0 2m42s

## Cleanup

In order to remove your operator from the cluster remove you example deployment:

    $ oc delete -f ~/helmoperatorexample.yaml

    mariadb.charts.helm.k8s.io “my-mariadb” deleted

    $ oc delete -f deploy/crds/charts_v1alpha1_mariadb_crd.yaml

    customresourcedefinition.apiextensions.k8s.io/mariadbs.charts.helm.k8s.io deleted

    $ oc delete -f deploy/service_account.yaml

    serviceaccount/mariadb-operator deleted

    $ oc delete -f deploy/role_binding.yaml

    rolebinding.rbac.authorization.k8s.io/mariadb-operator deleted

    $ oc delete -f deploy/role.yaml

    role.rbac.authorization.k8s.io/mariadb-operator deleted

    $ oc delete -f deploy/olm-catalog/mariadb-operator/0.0.1/mariadb-operator.v0.0.1.clusterserviceversion.yaml

    clusterserviceversion.operators.coreos.com/mariadb-operator.v0.0.1 deleted
