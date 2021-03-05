
# Jenkins-X Istio Flagger Canary Deployment

We’re using the Jenkins X way of performing Canary Deployments into a Kubernetes cluster

![[https://unsplash.com/photos/4NZlogMPIp0](https://unsplash.com/photos/4NZlogMPIp0)](https://cdn-images-1.medium.com/max/10944/1*yfK29TbgrIhralvpZzKPDw.jpeg)*[https://unsplash.com/photos/4NZlogMPIp0](https://unsplash.com/photos/4NZlogMPIp0)*

### Parts

1. [Canary Deployment with GitlabCI + GitOps/Manual Approach](https://medium.com/@wuestkamp/kubernetes-canary-deployment-1-gitlab-ci-518f9fdaa7ed?)

1. [Canary Deployment with Argo Rollouts](https://codeburst.io/kubernetes-canary-deployment-2-argo-rollouts-5e68e99b4fa3?source=friends_link&sk=58557d4fa81ff77382e59e1258c06d61)

1. [Canary Deployment with Istio](https://itnext.io/kubernetes-istio-canary-deployment-5ecfd7920e1c?source=friends_link&sk=2be48393ac175a2199bf5d486cb91acf)

1. (this article)

## What will we do here?

We’ll create a Jenkins X k8s cluster and a Python test app step by step. You can follow along or just read, look at the images and results to get an idea of the JenkinsX+Flagger+Istio Canary Deployment workflow and decide if its something worth diving into.

### Canary Deployment

[Make sure to read part 1](https://medium.com/@wuestkamp/kubernetes-canary-deployment-1-gitlab-ci-518f9fdaa7ed?source=friends_link&sk=4f3b424099f4f973f634bda75e397254) where we explained shortly what Canary Deployments are. In there we also show how to implement those using vanilla Kubernetes resources.

### Jenkins X

This article assumes that you know already a bit about Jenkins X. Else [you can do so here](https://itnext.io/k8s-native-jenkins-x-and-tekton-pipelines-e2b5a61a1d22?source=friends_link&sk=92a653ec0f914366bd7b13404941d9b0).

### Istio

This article assumes you already know what Istio is about. You can [read this](https://itnext.io/kubernetes-istio-simply-visually-explained-58a7d158b83f?source=friends_link&sk=378ed718d2d6cfd09e6d23c7616cba81) if you do not yet. You should also read [Part 3](https://itnext.io/kubernetes-istio-canary-deployment-5ecfd7920e1c?source=friends_link&sk=2be48393ac175a2199bf5d486cb91acf) where we used plain Istio for Canary Deployments for better understanding.

### Flagger
> Flagger is a Kubernetes operator that automates the promotion of canary deployments using Istio, Linkerd, App Mesh, NGINX, Contour or Gloo routing for traffic shifting and Prometheus metrics for canary analysis. The canary analysis can be extended with webhooks for running acceptance tests, load tests or any other custom validation. ([source](https://github.com/weaveworks/flagger))

## Create the test infrastructure and app

We will use the Jenkins X command-line tool jx to create a Kubernetes cluster, Github repositories and a quickstart application. This is great for fast prototyping.

### Jenkins X k8s cluster

First we create a Jenkins X cluster, here we use Gcloud:

    jx create cluster gke -n jenkinsx --machine-type n1-standard-4

You should choose the type “Serverless Jenkins X Pipelines with Tekton” and connect everything with your Github account. This will automatically create two Github repositories for staging and production environments.

Next we install some addons:

    jx create addon istio
    jx create addon flagger
    jx create addon prometheus

If you run into issues with Istio not working (I had missing istio_requests_total metrics in Prometheus) then you can install Istio manually:

    istioctl manifest generate --set values.kiali.enabled=true --set values.tracing.enabled=true --set values.grafana.enabled=true --set values.prometheus.enabled=true > istio.yaml

    kubectl -f istio.yaml install

The Flagger installation via jx will automatically enable Istio sidecar injection in namespace jx-production. To confirm this:

    kubectl get ns -L istio-injection

Add the label istio-injection: enabled to any namespace where the Istio sidecar should be injected.

### Test App / Quickstart

Now wait for the jxing-nginx-ingress-controller in namespace kube-system to have an External IP assigned. Then create a simple python app:

    jx create quickstart --project-name python-test

Select python-http as type. You can select anything though because the Canary Deployment works for every type. It should be something providing simple HTTP endpoints for testing though.

Jenkins X will automatically create the Github repository for the app.

I changed the do_GET method in the generated app.py in order to return text instead of an image:

    def do_GET(self):
      self.send_response(200)
      self.send_header('Content-type','text/html')
      self.end_headers()
      # Send the html message
      output = 'v1'
      self.wfile.write(output.encode('utf-8'))
      return

### Environments

Jenkins X will create three environments by default (jx get env):

![](https://cdn-images-1.medium.com/max/4008/1*97adQKIGiuK2k3FBacy84Q.png)

Environments have their own Git repositories where everything is configured via GitOps. Environments are actually configured as Helm Charts and reference all applications+versions installed in them via a requirements.yaml.

## Initial Deployment

When creating a new quickstart application, or importing an existing one (jx import), it will be released as version 0.0.1 in the staging environment. This is because PROMOTE is set to Auto for staging env. We can see this as a Pull Request on the staging repository which Jenkins X automatically creates and merges. We can see all Pipeline activities going on with:

    jx get activities -w

After the pipelines are successful we can check the available applications:

    jx get applications

![Jenkins X works automatically with Semver versioning](https://cdn-images-1.medium.com/max/3432/1*car56FMJb-686CiX2Qk4xw.png)*Jenkins X works automatically with Semver versioning*

Currently, we see that version 0.0.1 is deployed to staging, nothing in production yet. We can confirm this with:

    kubectl -n jx-staging get all       # shows the app
    kubectl -n jx-production get all    # shows nothing

We should be able to open access the staging app via the URL provided:

![](https://cdn-images-1.medium.com/max/3324/1*btM5nlafhZDiLCixtdP9ag.png)

### Promote to production

(Before going to production I introduced another change which bumped the version to 0.0.2)

Promote to production, where “promote” can be considered “deploy”. Via jx get env we can see that production needs manual promotion, so we do:

    jx promote --version 0.0.2 --env production

This will automatically create a Pull Request in the Jenkins X production Github repository. And now jx get applications will show version 0.0.2 in staging and also production:

![](https://cdn-images-1.medium.com/max/5008/1*bk9l2FeXnCsPeUFylzMGtw.png)

We could also promote to production by manually changing the requirements.yaml file in the production environment’s repo.

## Enable Canary for the production environment

Right now deployments into production or staging aren’t done using Canary but via Kubernetes rolling-rollouts.

In our Python test application repo, Jenkins X automatically created helm charts. If we open file charts/APP_NAME/values.yaml we can see various Flagger Canary options:

    *# Canary deployments
    # If enabled, Istio and Flagger need to be installed in the cluster
    ***canary:
      enabled: **false
      **progressDeadlineSeconds: **60
      **canaryAnalysis:
        interval: **"1m"
        **threshold: **5
        **maxWeight: **60
        **stepWeight: **20
        *# WARNING: Canary deployments will fail and rollback if there is no traffic that will generate the below specified metrics.
        ***metrics:
          requestSuccessRate:
            threshold: **99
            **interval: **"1m"
          **requestDuration:
            threshold: **1000
            **interval: **"1m"
      *# The host is using Istio Gateway and is currently not auto-generated
      # Please overwrite the `canary.host` in `values.yaml` in each environment repository (e.g., staging, production)
      ***host: **acme.com

With the above settings, a successful Canary deployment would take ~3 minutes, 1 minute for each step: 20%, 40%, 60%.

Though by default the Canary is disabled via canary: enabled: false.

We can override these values in every Jenkins X environments, which I do in the production repo environment-jenkinsx-production. Overriding the values of the Python application works because the environment repositories are Helm charts themselves and reference all deployed applications via requirements.yaml. I added the following to env/values.yaml in repo environment-jenkinsx-production:

    **...**
    **jenkinsx-istio-canary-python-test:**
      **canary:**
        **enabled:** true
        **host:** jenkinsx-istio-canary-python-test.35.204.67.7.nip.io

* The domain xxx.IP_ADDRESS.nip.io will simply redirect to IP_ADDRESS. Great for simple domain name testing

* The host: will be the Istio hostname through which requests can be accepted. Find your Istio Gateway external IP address with: kubectl -n istio-system get svc. You need to ensure that the domain name specified under host: redirects to the Istio Gateway external IP.

Making the change in the production environment repo will automatically trigger a build in Jenkins X, check with: jx get activity

Wait until all activities were successful. We should now see Istio VirtualService and Gateway automatically created:

    kubectl -n jx-production get virtualservices.networking.istio.io,gateways.networking.istio.io
> Note that the first time it is promoted it will not do a Canary as it needs a previous version data to compare to, but it will work from the second promotion on. ([source](https://jenkins-x.io/docs/managing-jx/tutorials/progressive-delivery/))

We should now be able to access our application in production via Istio on the specified host jenkinsx-istio-canary-python-test.35.204.67.7.nip.io:

![](https://cdn-images-1.medium.com/max/2544/1*BMEbz-giGvuVOzAYW31i5Q.png)

## Open Flagger Grafana Dashboard

Flagger will install Grafana in the istio-system namespace, hence we can do:

    kubectl port-forward -n istio-system service/flagger-grafana 3000:80
    # admin:admin

![Flagger comes with Grafana and Dashboard](https://cdn-images-1.medium.com/max/4764/1*vY95I47XZqvudbvo0LIDyA.png)*Flagger comes with Grafana and Dashboard*

In Grafana head to the Istio Dashboard and enter:

* namespace: jx-production

* primary: jx-jenkinsx-istio-canary-python-test-primary

* canary: jx-jenkinsx-istio-canary-python-test-canary

The primary and canary are the names of the services. These are created automatically during deployment when Canary is enabled.

If you have missing istio_requests_total metrics make sure Istio is installed correctly, maybe install manually instead of via jx.

## Flagger Canary Workflow and Analysis

In the Python application’s Helm Chart we have multiple settings we can configure ([check here](https://docs.flagger.app/usage/progressive-delivery) for an explanation of most of these):

    **canary:
      enabled: **false # false, but we set to true in prod env git repo
      **progressDeadlineSeconds: **60
      **canaryAnalysis:
        interval: **"1m"
        **threshold: **5
        **maxWeight: **60
        **stepWeight: **20
        *# WARNING: Canary deployments will fail and rollback if there is no traffic that will generate the below specified metrics.
        ***metrics:
          requestSuccessRate:
            threshold: **99
            **interval: **"1m"
          **requestDuration:
            threshold: **1000
            **interval: **"1m"

We should adjust these depending on our application Canary needs. Flagger will perform automatic metric analysis and only progresses with the deployment if the requirements are met.

To see the current status of the Canary deployment, check the Flagger resources:

    kubectl -n jx-production get canaries.flagger.app

## Perform Canary Deployment

### Deploy a change

Now we change the applications app.py to return a different text on GET request. Create a Pull Request to master in the app repo and merge it. Jenkins X will deploy it automatically to staging.

Now promote it manually to production, copy the version number which is running on staging. I’m now at 0.0.6 as I made some changes in between.

    jx promote --version 0.0.6 --env production

Monitor Canary status:

    kubectl -n jx-production get events --field-selector involvedObject.kind=Canary --sort-by='{.lastTimestamp}'

### Canary starts with weight 20%

    2m39s       Normal    Synced   Canary   Advance jx-jenkinsx-istio-canary-python-test.jx-production canary **weight 20**

We curl the production Istio endpoint in a while loop, which results in 20% requests are reaching the new version:

![](https://cdn-images-1.medium.com/max/2872/1*BeqO_if-crZ1xzzs03nzxA.png)

### Canary fails (without enough traffic)

We need to ensure enough traffic is hitting the endpoint, else the Canary analysis by Flagger will fail and a rollback is initiated:

![](https://cdn-images-1.medium.com/max/5544/1*ruGDrwUU1_k_hHarACEPOg.png)

Canary will also fail if the defined requirements aren’t met.

### Canary weight 40%

If there is enough successful traffic hitting the app, Flagger will raise to 40%:

    108s        Normal    Synced   Canary   Advance jx-jenkinsx-istio-canary-python-test.jx-production canary **weight 40**

![](https://cdn-images-1.medium.com/max/2892/1*PkhWU2nrzWXcU2yqldT6ig.png)

### Canary weight 60%

If there is enough successful traffic hitting the app, Flagger will raise to 60%:

    60s         Normal    Synced   Canary   Advance jx-jenkinsx-istio-canary-python-test.jx-production canary **weight 60**

![](https://cdn-images-1.medium.com/max/2880/1*HR57bzs44hZjnQ-O0fohzw.png)

### Canary success

    25s         Normal    Synced   Canary   Routing all traffic to primary

All requests are hitting the new version:

![](https://cdn-images-1.medium.com/max/2848/1*GGrmfEtVxmPUnCIi9fyhWQ.png)

## Recap

Jenkins X can be relatively easily configured to run with Istio and Flagger for Canary Deployments. I had issues with installing Istio via jx create addon istio and had to do it manually.

I haven’t played with the Flagger configuration, but just using the default seems great for poking around. I think once one gets a bit more familiar with Jenkins X and its proposed workflow it will take a lot of manual or repetitive work away.

Also without Jenkins X, I think the combination of Flagger and Istio for Canary Deployments is a powerful one which I might explore on its own soon.

## More / Sources

[https://jenkins-x.io/docs/managing-jx/tutorials/progressive-delivery](https://jenkins-x.io/docs/managing-jx/tutorials/progressive-delivery/)

[https://docs.flagger.app/usage/progressive-delivery](https://docs.flagger.app/usage/progressive-delivery)

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/7eePqtxW7NM" frameborder="0" allowfullscreen></iframe></center>

## Become Kubernetes Certified

![[https://killer.sh](https://killer.sh)](https://cdn-images-1.medium.com/max/3534/1*7Kbj17_6VncUuoBqNsAzzg.png)*[https://killer.sh](https://killer.sh)*
