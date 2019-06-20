Unknown markup type 10 { type: [33m10[39m, start: [33m15[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m73[39m, end: [33m89[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m59[39m, end: [33m69[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m296[39m, end: [33m310[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m39[39m, end: [33m51[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m34[39m, end: [33m40[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m113[39m, end: [33m120[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m93[39m, end: [33m104[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m228[39m, end: [33m240[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m206[39m, end: [33m212[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m19[39m }

# Deploy Kubernetes Apps with Terraform

Deploy Kubernetes Apps with Terraform

### Google Kubernetes Engine

[**Infrastructure As Code](https://en.wikipedia.org/wiki/Infrastructure_as_Code)** during the *cloud age* is to use source code to document, version, and control your infrastructure. [**Terraform](https://www.terraform.io/)** is by far the most popular and intuitive tool for this process. There are numerous articles, blogs, how-tos, and source code repositories about using [**Terraform](https://www.terraform.io/)** to craft cloud resources on [**AWS](https://aws.amazon.com/)**, [**Azure](https://azure.microsoft.com/)**, and [**GCP](https://cloud.google.com/)**.

But [**Terraform](https://www.terraform.io/)** is a tool to create any resource that exposed through web api ([**RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer)**), and not only can you create your infrastructure with [**Terraform](https://www.terraform.io/)**, you can deploy applications on orchestration platforms like [**Kubernetes](https://kubernetes.io/)**.

This article illustrates spinning up a [**Kubernetes ](https://kubernetes.io/)**cluster on [**Google Cloud](https://cloud.google.com/)** using GKE, and then deploying the guestbook sample application ([**AngularJS](https://angularjs.org/)**, [**PHP](http://php.net/)**, [**Redis](https://redis.io/)**), which google has made available in their container registry.

## Installing the Tools

For this tutorial you need the following:

* **Google Cloud SDK** - [https://cloud.google.com/sdk/](https://cloud.google.com/sdk/)

* **Terraform** - [https://www.terraform.io/downloads.html](https://www.terraform.io/downloads.html)

* **Kubectl **- [https://kubernetes.io/docs/tasks/tools/install-kubectl/](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

After installing [**Google Cloud SDK](https://cloud.google.com/sdk/)**, you’ll want to [**initialize](https://cloud.google.com/sdk/docs/initializing)** and [**authorize](https://cloud.google.com/sdk/docs/authorizing)** [**Google Cloud SDK](https://cloud.google.com/sdk/)** for your account.

### Install Tools on Mac OS X

With [**HomeBrew](https://brew.sh/)** installed, you can do this to install the tools:

    cat <<-"BREWFILE" > Brewfile
    **cask** 'google-cloud-sdk'
    **brew** 'kubectl'
    **brew** 'terraform'
    BREWFILE

    brew bundle --verbose

## Organizing into Modules

We’ll start by setting up this directory structure, and files referenced will use this:

    .
    ├── gke
    │   ├── cluster.tf
    │   ├── gcp.tf
    │   └── variables.tf
    ├── k8s
    │   ├── k8s.tf
    │   ├── pods.tf
    │   ├── services.tf
    │   └── variables.tf
    └─── main.tf

We can create this structure and empty files:

    **mkdir** terraform-gke
    **cd** terraform-gke
    **mkdir** gke k8s
    **touch** main.tf
    **for** f **in** cluster gcp variables; **do** touch gke/$f.tf; **done**
    **for** f **in** k8s pods services variables; **do** touch k8s/$f.tf; **done**

Our top level, main.tf will reference two modules that we’ll create later. One module will create the GKE cluster, and the other module will use information from GKE to deploy software into the [**Kubernetes](https://kubernetes.io/)** cluster.

<iframe src="https://medium.com/media/3491be2edb8cb161f56e3a779ce0ca45" frameborder=0></iframe>

## Cluster Specification

This is the module that will create a [**Kubernetes](https://kubernetes.io/)** cluster on Google Cloud using GKE resource.

This first start by specifying all the variables this module will use in gke/variables.tf file:

<iframe src="https://medium.com/media/1744d10dd4d0bfe967b999398c4eafe8" frameborder=0></iframe>

We’ll need to specify a provider, which is Google Cloud in gke/gcp.tf file:

<iframe src="https://medium.com/media/140eca946f514c3251f3964b8b859e0c" frameborder=0></iframe>

With the variables specified and provider specified, we can now create our [**Kubernetes](https://kubernetes.io/)** infrastructure. In Google Cloud this is one resource, but this encapsulates many components (managed instance group and template, persistence store, GCE instances for worker nodes, GKE master). This in done in gke/cluster.tf file:

<iframe src="https://medium.com/media/88101c3b2a08ae5f50e1c811fc33963f" frameborder=0></iframe>

This creates a 3 worker node cluster. The output variables will be used later when we deploy applications. They are marked sensitive to avoid printing out to standard output.

## Guestbook Application Specification

[**Kubernetes](https://kubernetes.io/)** code repository has an example application called guestbook that uses Redis cluster to store information. This module is divided into four parts:

1. Variables used in this module

1. [**Kubernetes](https://kubernetes.io/)** provider to connect to Kubernetes API

1. Pods using Replication Controller

1. Services creates permanent end point and connecting them to internal IP addresses as pods are added or removed.

The variables we’ll use are defined in variables.tf file:

<iframe src="https://medium.com/media/8523e1c46c863196020ca68a3e248780" frameborder=0></iframe>

Our [**Kubernetes](https://kubernetes.io/)** provider is in the k8s.tf file:

<iframe src="https://medium.com/media/70d90d17edfb57b41a2d39008c789586" frameborder=0></iframe>

And now we create our minimum unit of deployment, the [**Kubernetes](https://kubernetes.io/)** pods using [**Kubernetes](https://kubernetes.io/)** Replication Controller in pods.tf file. These will be 1 redis master pod, 2 redis slave pods, and 1 frontend pod. The images for these components are available from Google’s Container Registry.

<iframe src="https://medium.com/media/8262c1db49c65a01e7a9caaed16695d5" frameborder=0></iframe>

This will create the pods that we can now use to deploy services into them. We’ll create the services.tf for the services we wish to deploy (redis master, redis slave, frontend). One note about the frontend, as it uses the type LoadBalancer, this will create a google load balancer outside of the cluster to send traffic one of three pods.

<iframe src="https://medium.com/media/cfb9b5709ce048fd18d28825fbe3e603" frameborder=0></iframe>

## Launch the Application

Before we start, we need to initialize some variables that the GCP provider requires, which is the target project and the desired region to create the cluster. We’ll use our default project configured with gcloud:

    export **TF_VAR_project**="$(gcloud config list \
      --format 'value(core.project)'
    )"
    export **TF_VAR_region**="us-east1"

Now we’ll need to specify the administrative account and a random password for the cluster:

    export **TF_VAR_user**="admin"
    export **TF_VAR_password**="m8XBWrg2zt8R8JoH"

With these setup, we can initialize our environment, which includes both downloading plugins required [**google cloud provider](https://www.terraform.io/docs/providers/google/index.html)** and [**kubernetes provider](https://www.terraform.io/docs/providers/kubernetes/index.html)**, as well as references to our modules.

    terraform init

Now we can see what we want to create and then create it:

    terraform plan
    terraform apply

After some time (10 to 20 minutes) we can test out our application. Run this to see the end points:

    kubectl get service

## Final Thoughts

This is an easy way to quickly test clusters, pods, services, and other components. Currently, [**Terraform](https://www.terraform.io/)** only supports deploying [**Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/)** and [**ReplicationControllers](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/)** as deployable units.

More popular now are [**ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)** and [**Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)** objects as deployable units, which [**Terraform](https://www.terraform.io/)** community doesn’t yet want to support as these are still in beta. If you need these other controllers, then you’ll have to use [**kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)** or [**helm](https://helm.sh/)** for deployments.
