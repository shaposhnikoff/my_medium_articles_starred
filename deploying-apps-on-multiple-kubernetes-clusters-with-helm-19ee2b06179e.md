Unknown markup type 10 { type: [33m10[39m, start: [33m155[39m, end: [33m169[39m }

# Deploying apps on multiple Kubernetes clusters with Helm

Dailymotion’s Kubernetes journey: applications deployment

![](https://cdn-images-1.medium.com/max/4080/1*IabPcL9_oHoOeHmptXmk0A.jpeg)

**At Dailymotion, we started using Kubernetes in our production environment 3 years ago. However, deploying applications on multiple clusters can be somewhat challenging, which is why we’ve been working to improve our tools and workflows over the past few years.**

## How we got here

*Here we will focus on how we deploy our applications on multiple Kubernetes clusters around the world.*

In order to deploy multiple Kubernetes objects in one shot, we use [**Helm](https://helm.sh/) **and all our charts are stored in a single git repository. To be able to deploy a complete application stack composed of multiple services we use what we call an **umbrella** chart. This is basically a chart that declares dependencies and allows us to bootstrap our API and its services in a single command line.

Additionally, we’ve created a small python script on top of Helm that makes some checks, builds charts, adds the secrets and deploys our applications. All these tasks are completed from a **central CI Platform** using a docker image.
> ***Note:** By the time you’ll read this blog post Helm 3’s first[ release candidate ](https://github.com/helm/helm/releases/tag/v3.0.0-alpha.1)will have already been announced. This major version is coming with a bunch of enhancements that will definitely solve some of the issues we encountered in the past.*

## Charts development workflow

For our applications, we use the [branching workflow](https://git-scm.com/book/en/v2/Git-Branching-Branching-Workflows) and we’ve decided to do the same with our charts development.

* the **dev** branch is used to build charts that are meant to be tested on development clusters.

* Then, when a pull request is merged on **master**, they are validated in a staging environment.

* Finally, we create a pull request to merge the changes into the **prod** branch in order to apply the changes in production.

Each environment has its own private repository that stores our charts and we use [**Chartmuseum](https://chartmuseum.com/)** that exposes a really useful API. That way we enforce clear **isolation between environments** and we ensure that the charts have been battle tested before using them in production.

![Chart repository per environment](https://cdn-images-1.medium.com/max/2504/1*eIBPN3HCb5AQk-QQar-S9g.png)*Chart repository per environment*

It is worth noting that when the developers push their dev branch, a version of their chart is automatically pushed to the dev Chartmuseum. Thus all the developers are using the same dev repository and they have to be careful to specify their own chart version in order to avoid using someone else’s chart changes.

Furthermore, our small python script validates the Kubernetes objects against the Kubernetes OpenAPI specs by using [**Kubeval](https://kubeval.instrumenta.dev/) **before pushing them to the Chartmusem

### Summary of the chart development workflow

![](https://cdn-images-1.medium.com/max/2000/1*gQ5wsmgYGmEv98WUaCVQ7Q.png)

1. Setup our pipeline tasks according to the [**gazr.io](https://gazr.io/)** specification for the quality tasks (lint, unit-test)

2. Push the docker image that contains the Python tooling used to deploy our applications

3. Set the environment according to the branch name

4. Check the Kubernetes yamls with Kubeval

5. Increment the chart version and its parents automatically (charts that depend on the one being changed)

6. Push the chart to the Chartmuseum that corresponds to its environment

## Managing clusters differences

### Cluster federation

At some point, we were using the [Kubernetes cluster federation](https://kubernetes.io/docs/concepts/cluster-administration/federation/) that allowed us to declare Kubernetes objects from a single API endpoint. But we faced issues namely that some of the Kubernetes objects couldn’t be created in the federation endpoint making it hard to maintain federated objects and other *per-cluster* objects.

To alieviate this issue, we decided to manage our clusters independently and which ended up making the process much easier (we were using the federation v1 though, things may have changed with the v2).

### Geo-distributed platform

Currently, our platform is spread across 6 regions, 3 on-premises and 3 in the cloud.

![Distributed deployment](https://cdn-images-1.medium.com/max/4068/1*FA8rxWOZZjdU6wFrKveImg.png)*Distributed deployment*

### Helm global values

4 global Helm values allow us to define the differences between our clusters. These are the minimum default values for all of our charts.

<iframe src="https://medium.com/media/a9bab9823f943b3037d686c0062c0326" frameborder=0></iframe>

This information helps us define a context for our applications and they’re used for many things such as monitoring, tracing, logging, making external calls, scaling etc...

* “*cloud*”: We have a hybrid Kubernetes platform. For instance, our API is deployed on both GCP zones and in our own datacenters.

* “*env*”: Some values may change for non-production environments. Essentially the resource definitions and the autoscaling configurations.

* “*region*”: This information serves to identify the location of the cluster and can be used to define the closest endpoints for external services.

* “*clusterName*”: If and when we need to/want to define a value per cluster.

Here is a concrete example:

<iframe src="https://medium.com/media/f8889c480a27a1ee1bb64fda674aedf8" frameborder=0></iframe>

Note that this logic is defined in a helper template in order to keep the Kubernetes YAML fairly readable.

### Declaring an application

Our deployment tooling is based on several YAML files, below is an example of us declaring a service and its scaling topology (number of replicas) per cluster.

<iframe src="https://medium.com/media/e158f82db356cf043667f117b64bc78e" frameborder=0></iframe>

This is a diagram of all the steps that define our deployment workflow. The last step will deploy the application simultaneously on several production clusters.

![Jenkins deployment steps](https://cdn-images-1.medium.com/max/2000/1*aVw14rBd5nyCOIeuX2nMRg.png)*Jenkins deployment steps*

## What about the secrets?

In the security area, we focused on tracking all the secrets that could be spread in different locations and stored them in a unique [**Vault](https://www.vaultproject.io/)** backend located in Paris.

Our deployment tooling is in charge of retrieving the secrets values from Vault and injects them into Helm when it is time for deployment.

In order to do so, we defined a mapping between the secrets stored in Vault and the ones needed by our apps as follows:

<iframe src="https://medium.com/media/67db012223f0d874f2e41a43600b8d29" frameborder=0></iframe>

* We defined common rules to follow when writing secrets into Vault.

* If the secret is **specific to a context/cluster,** a specific entry must be added. (Here the *cluster1* context has its own value for the secret *stack-app1-password*).

* Otherwise, the ***default*** value will be used.

* For each item on this list, a key/value will be inserted into the **Kubernetes secret**. That way the secret template in our charts is really simple

<iframe src="https://medium.com/media/319d26aaa3d4ede24a18a49903b443b2" frameborder=0></iframe>

## Caveats and limitations

### Working with multiple repositories

Currently, the charts and our application developments are decoupled. That means that the developer has to work on two git repositories, one for the application and the other one to define how it’s deployed on Kubernetes. Indeed, 2 git repositories mean two workflows and that can be pretty confusing for a newcomer.

### Managing umbrella charts can be messy

As mentioned earlier the umbrella charts are great for defining dependencies and quickly deploying several applications. However we make use of the option --reuse-valuesin order to avoid passing all the values each time we deploy an application that is part of the umbrella chart.

In our continuous deployment workflow, there are only 2 values that change regularly: the number of replicas and the image tag (version). For the other, more stable values, changing them requires a manual update and that can be hard to figure out. Furthermore, a single mistake on the deployment of an umbrella chart can lead to severe outages, we’ve seen it first-hand.

### Multiple config files to update

When adding a new application the developer has to change several files: the application declaration, the secrets list and add it to the dependencies if the app is part of an umbrella chart.

### Jenkins permissions are too extended on Vault

Currently, we have a single [AppRole](https://www.vaultproject.io/docs/auth/approle.html) that can read all the secrets from Vault.

### Rollback process is not automated

Rolling back requires to run the command on multiple clusters which can be error-prone. We make this operation manual because we want to make sure we apply the right revision id .

## Our GitOps future is being written

### Our target

The idea is to put back a chart under the repository of the application it deploys.
The workflow will be the same as for development. For example, whenever a branch is merged on master a deployment would be triggered automatically. The main difference between this approach and the current workflow is that** everything will be managed from git** (The application itself and the way we deploy it in Kubernetes)

These are some of the benefits:

* Much **simpler** to understand from a developer point of view. Learning how to apply changes in the local chart will be easier.

* Defining the service deployment definition in the **same location as the code** of this service.

* **The removal of the umbrella charts** management. A service will have its own Helm release. That makes application lifecycle management (rollback, upgrade) atomic, no other service would be impacted.

* **Benefits of git** features for the chart management: revert, audit log… If you want to revert a chart change you can do it using git. Deployment will be automatically triggered.

* We can consider an improvement of the development workflow with tools like **Skaffold** that allow developers to test their changes in a context similar to the production.

### 2 steps migration

Our developers have been using the workflow described above for 2 years so we need the migration to run as smoothly as possible. That’s why we’ve decided to add an intermediate step before reaching our target.

The first step is simple:

* We’ll keep a similar structure for configuring our application deployment but in a single object named “DailymotionRelease”

<iframe src="https://medium.com/media/b27b7d2d4cd07ff4e198340bcafa3238" frameborder=0></iframe>

* One Release per application (no more umbrella charts)

* The charts in the application git repository

We have begun spreading the word to all the developers and the migration process has already begun. This first step is still controlled using the CI platform. I will write another blog post shortly that will describe the second step: how we achieved our migration to a GitOps workflow with [**Flux](https://github.com/weaveworks/flux)**. We will describe our setup and the challenges faced (multiple repositories, secrets …).

We tried here to describe our progress in recent years regarding our application deployment workflow that led us to consider the GitOps approach. This journey is not finished yet and we will keep you posted soon but we are now convinced that keeping things simple and getting closer to the developers’ habits is the right choice.
[**How we built our hybrid Kubernetes platform**
*Dailymotion’s Kubernetes journey: a timeline of our geo-distributed infrastructure around the world mixing Cloud and…*medium.com](https://medium.com/dailymotion/how-we-built-our-hybrid-kubernetes-platform-d121ea9cb0bc)
[**Introducing Tartiflette: dailymotion's Open Source GraphQL Implementation for Python 3.6+**
*This introspection confirmed what we already knew: our desire to geo-distribute our platform and build API-driven web…*medium.com](https://medium.com/dailymotion/tartiflette-graphql-api-engine-python-open-source-a200c5bbc477)
