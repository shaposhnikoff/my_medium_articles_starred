Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m279[39m, end: [33m292[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m167[39m, end: [33m179[39m }

# Best practices for deploying to Kubernetes using Helm

Best practices for deploying to Kubernetes using Helm

*This is part of a series on pragmatic, safe and repeatable ways of deploying to Kubernetes. These posts are built as part of building [Deliverybot](http://deliverybot.github.io), the missing link between deploying to Kubernetes from GitHub.*

In the last post we discussed deploying using bare Kubernetes manifests with [Kustomize to deploy application](https://medium.com/@colinjfw/kustomize-deploying-applications-using-the-new-kubernetes-templating-system-43f26706069)s. This post discusses one of the most common deployment tools for Kubernetes. Helm is the “package manager for Kubernetes” and therefore has become one of the most common tools for managing application deployments.

Helm, however, is a package format. Not a workflow for deploying code into Kubernetes. In this post I want to discuss how to use Helm in different types of workflows the most effectively to provide easy and safe deployments for your infrastructure on Kubernetes.

Helm is structured around charts. These, fundamentally, are sets of Yaml files which are evaluated using the go templating language. Charts allow us to build modules for Kubernetes. We can declare a set of values in Helm that a user supplies when deploying a chart to release an application that abstracts away some of the complexities of releasing certain applications to Kubernetes.

## A basic Helm chart

    ~/charts/app
    ├── Chart.yaml
    ├── README.md
    ├── templates
    │   ├── NOTES.txt
    │   ├── _helpers.tpl
    │   ├── deployment.yaml
    │   ├── ingress.yaml
    │   ├── secret.yaml
    │   └── service.yaml
    └── values.yaml

The basics of a Helm chart consists of a chart metadata (“Chart.yaml” and “values.yaml”) as well as the templates that make up your main chart. The main chart configuration consists of naming and versioning for your chart. The values file is where we declare values that are read internally by the templates of your chart that can be declared externally by users of your chart. This is Helm as an abstraction layer, simplifying a set of values that can be expanded into a multitude of Kubernetes manifests under the hood.

### A Deployment, a service and an ingress

This is the core of most Helm charts that focus on releasing networked applications. These three resources are the key to getting your application deployed and to be able to talk to it from outside the cluster. Very briefly:

* Deployments release your code.

* Services route traffic internally to your code.

* Ingress resources route external traffic into the cluster to your code.

This is why these three resources are the backbone of releasing networked applications. Let’s look at a very small example of a service resource in a Helm chart and how templating allows us to redefine options:

    apiVersion: v1
    kind: Service
    metadata:
      name: {{ include "app.fullname" . }}
      labels:
    {{ include "app.labels" . | indent 4 }}
    spec:
      type: {{ .Values.service.type }}

This basic service resource example shows some go templating tools used to create a helper to designate a full name for a resource. The corresponding values that create this service look like:

    service:
      type: ClusterIP
      port: 80

This is a simple example of the benefit of abstracting out your underlying resources using Helm.

## An opinionated release workflow

Helm is meant to de-duplicate work by the community to install software into your cluster. There are two primary workflows that you’ll end up adopting when choosing Helm to release code:

1. Releasing third party code into the cluster. This code is usually semantically versioned and released at a slower cadence, maybe monthly or yearly depending on the update cycle.

1. Continuous delivery into your Kubernetes cluster. This code is often released daily or even more frequently than this.

The first case is what Helm was originally designed for. It requires that charts implement semantic versioning and makes upgrading charts relatively simple. The most effective way to work with Helm for third party packages is to use the [Terraform provider](https://www.terraform.io/docs/providers/helm/index.html). This allows for a stable and consistent release process into your cluster. This often ties in with the rest of your infrastructure as code workflow.

The main goal of this post is to look at using Helm in continuous delivery. Doing this effectively may seem simple at the start but there are some things to keep in mind to ensure your deployment is effective.

Deploying with Helm using the CLI is relatively simple. Specify the following set of commands to release a new chart in the default namespace with values from the production value file:

    helm upgrade --install release-name \
      --namespace default \
      --values ./production.yml

What actually happens when I run this command? Under the hood Helm has a server side implementation called Tiller. Helm opens a connection to your Kubernetes cluster and writes your Helm chart to this connection along with the values that you passed in as arguments. The server side component renders the templates with these values and applies them into the Kubernetes cluster.

These are the basics of running a deployment with Helm. The opinionated parts and your workflows involve figuring out what values to pass as arguments and when to run the Helm upgrade command. Let’s discuss some of the right ways to do this within your organization.

### Avoid a chart per service

One of the benefits of using Helm charts is that you can take the common configuration that may be consistent across your organization or projects and consolidate this into a single chart. It may be simpler to put a single chart in every repository that you want to deploy from but invest in having a core set of charts that you use for services across you organization.

This can speed up things like having standardized health check paths and keep configuration common across resources. Additionally, if you want to adopt newer api versions of Kubernetes resources as they mature, you’ll be able to do this from a central location.

Helm charts can become incredibly verbose and are actually quite hard to author well. You need to take care to ensure that undefined variables will error out, that labels are consistent and that you won’t have any Yaml indentation errors. It’s best to put the authoring of Helm charts in the hands of those who are experienced in working with Kubernetes and allow your developers to use them as modules.

### Separating production and staging configuration values

The configuration values that you push into your charts should be separate between environments. Build simple extendable charts that can have overrides per environment.

For example, a good workflow would have different value files per environment with specific differences in configuration:

    ~/myapp
    └── config
        ├── production.yml
        └── staging.yml

Each value file is used depending on the environment for the deployment. This is in contrast to baking configuration into Helm charts using if statements or other logic. Helm is there to abstract away the Kubernetes resources and value files are the way to pass in your environment and application specific information.

Ensure that you use “require” statements and linting tools to ensure that you don’t have an undefined value when deploying. The following variable if undefined could actually cause a relatively serious configuration issue:

    host: {{ .Values.hostPrefix }}.example.com

### Secret management

Kubernetes secrets are one of the simplest ways of managing secrets for an application. Helm doesn’t attempt to manage secrets in any way and you may get caught in trying to configure complex workflows. A simple initial recommendation for secret management is to:

1. Use your CI/CD provider to store secrets in their dashboard.

1. Pass the secret values into your chart values on deployment.

1. Use a [checksum to roll out pods](https://github.com/helm/helm/blob/master/docs/charts_tips_and_tricks.md#automatically-roll-deployments-when-configmaps-or-secrets-change) when these secret values change.

We want releasing a new secret into the cluster to be auditable and visible in our deployment pipeline. We also want developers to have access to this. As always, think of the best solution for your situation.

[Vault](https://www.vaultproject.io/) is also a fantastic tool for secret management that integrates deeply with Kubernetes to provide improved security. It may take longer to setup but it offers advanced features that your organization can take advantage of down the road.

### Editing values

Ensure that when using the Helm CLI that only Helm will change values in your Kubernetes manifests. Don’t, for example, modify a deployment’s replicas outside of Helm and then also change the values using the Helm. If you’ve worked with Kubernetes before, you’ve likely used the kubectl apply tool. This is one of the simplest ways of applying infrastructure into the cluster. Helm doesn’t use [the same techniques](https://github.com/helm/helm/issues/2070#issuecomment-284839395) as Kubectl. This can mean that if you edit infrastructure outside of Helm it can cause issues when running Helm commands the next time around.

### Helm chart repositories

Helm chart repositories are just http servers that serve chart resources. You can see the officially curated chart resources at [github.com/helm/charts](https://github.com/helm/charts). Chart repositories are primarily designed for semantically versioned charts. As a result, chart repositories don’t handle concurrent updates very well. Avoid pushing to a chart repository in CI or patterns like making a new Helm chart for every commit.

Helm charts should be semantically versioned modules. They represent the underlying infrastructure to deploy an application. Think of them as a library that you are using to abstract away Kubernetes complexity. Store charts in a repository accessible across your organization and allow developers to easily use these charts to deploy using best practices in your organization.

A simple way of distributing charts across your organization can be using S3 or even GitHub releases as a storage for your Charts. A simple pattern is having a common myorg/charts repository with all charts that your team has curated and built that can be installed into Kubernetes.

### Secure your Tiller

If you have strict access controls for your Kubernetes cluster you may be tripped up by the issue that Tiller (the server side component of Helm) runs as a super user in your Kubernetes cluster. There are some workarounds for this but it takes some more complex configuration to work them out. For most teams, the default configuration will look like when you connect to Helm you have administrator access to the cluster.

One common approach to fix this problem is to create a different Tiller instance in different namespaces. If you have a very large Kubernetes cluster, this may be a way of segmenting specific teams to only have access to a single Tiller. Helm version 3 has more updates in this area to actually work client side only to apply resources.

## Summary

One of the hardest challenges with new Kubernetes deployments is wading through all of the best practices and opinions to get your deployment pipeline setup correctly the first time. Helm deployments have a lot of gotcha’s that you’ll need to navigate to setup correctly but once there provide flexibility and power in being able to wire up your deployments effectively.

*Stay tuned for the next in the series of pragmatic and safe deployments to Kubernetes. We’ll be discussing automation tools with GitHub to wrap all of these best practices in code.*
