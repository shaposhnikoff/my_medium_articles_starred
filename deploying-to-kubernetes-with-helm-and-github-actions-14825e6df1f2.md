Unknown markup type 10 { type: [33m10[39m, start: [33m340[39m, end: [33m357[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m14[39m, end: [33m18[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m189[39m, end: [33m193[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m387[39m, end: [33m391[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m316[39m, end: [33m323[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m292[39m, end: [33m299[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m25[39m, end: [33m32[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m11[39m, end: [33m17[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m97[39m, end: [33m104[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m6[39m, end: [33m16[39m }

# Deploying to Kubernetes with Helm and GitHub Actions

Deploying to Kubernetes with Helm and GitHub Actions

*This is part of a series on pragmatic, safe and repeatable ways of deploying to Kubernetes. These posts are built as part of building [Deliverybot](http://deliverybot.github.io/), the missing link between deploying to Kubernetes from GitHub.*
> Note, if you want to skip the intro and just look at an example repo head over to [github.com/deliverybot/example-helm](https://github.com/deliverybot/example-helm). Or fork the repository and follow along with the tutorial.

GitHub actions are a new CI/CD service from GitHub. Actions are simple workflows configured as Yaml files which run configurable steps of code based on GitHub events. Since they are baked into GitHub, they reduce significantly the overhead in getting a CI/CD pipeline setup.

This tutorial will go through the basics of GitHub actions as well as deploying to Kubernetes using a pre-built Helm action. For this guide we’re assuming some basic knowledge of Kubernetes and Helm but you’ll probably be able to follow along regardless. Let’s get started with some of the basic concepts of GitHub actions.

### The basics

The first concept to introduce is a workflow file. This is a pipeline similar to a Jenkinsfile. It describes a set of steps to execute given a GitHub event. Workflow files are composed of jobs, steps and actions. Jobs are a set of steps that run in parallel while actions are the actual logic that run in a step. Workflow files live in the .github/workflows folder. Let’s take a look at a basic workflow file below.

    # .github/workflows/cd.yml
    name: 'Deploy'     # Name of the action.
    on: ['deployment'] # Listen for deployment events.

    jobs:
      deployment:
        runs-on: 'ubuntu-latest'
        steps:
        - name: 'Checkout'  # Checkout the repository code.
          uses: 'actions/checkout@v1'

The above GitHub action listens for a deployment event from GitHub and checks out the repository code. We don’t actually do any deployments in this step, that’s next.

Note that the uses directive actually refers to a GitHub repository. GitHub will clone that repository and run the code inside that repository as part of our job. This is one of the best parts about actions. Sharing modules is simple, straightforwards and just requires making a git repository.

### Helm action

Now let’s look at building out the steps needed to deploy to Kubernetes. The first thing that we need is a [Helm](https://helm.sh) chart. Helm is the standard packaging format for Kubernetes and is becoming the defacto tool for managing deployments. Helm is effectively a CLI tool which comprises a templating system and a connector that applies manifest files into Kubernetes. It’s main aim is to abstract away the details of Kubernetes deployments into re-usable modules called charts. Our Helm action needs to grab one of these modules and execute the necessary commands to apply this into our Kubernetes cluster of choice.

The Helm action that we’ll use is hosted at [github.com/deliverybot/helm](https://github.com/deliverybot/helm). This action supports Helm version 3 which is going to be released very soon and brings a lot of improvements.

The Helm chart that we’re going to be using is a module that provides safe defaults for deploying HTTP based workloads to Kubernetes. This chart lives at [deliverybot/helm/charts/app](https://github.com/deliverybot/helm/tree/master/charts/app). It applies ingress, deployment and service resources with best practices baked in so you don’t have to think about them. Let’s see the basics for a workflow below:

    # .github/workflows/cd.yml
    name: 'Deploy'     # Name of the action.
    on: ['deployment'] # Listen for deployment events.

    jobs:
      deployment:
        runs-on: 'ubuntu-latest'
        steps:
        - name: 'Checkout'  # Checkout the repository code.
          uses: 'actions/checkout@v1'
    
    **    - name: 'Deploy'
          uses: 'deliverybot/helm@master'
          with:
            token: '${{ github.token }}'
            chart: 'app'
          env:
            KUBECONFIG_FILE: '${{ secrets.KUBECONFIG }}'**

The code from the [deliverybot/helm](https://github.com/deliverybot/helm) repository will be fetched and executed with the arguments that we provide when this workflow executes. We supply these arguments to the action using the with directive. This translates these values into environment variables that are available inside the action code to execute our deployment. Let’s look at an example with some more arguments in the with parameter.

    ...
        - name: 'Deploy'
          uses: 'deliverybot/helm@master'
          with:
            token: '${{ github.token }}'
            chart: 'app'
    **        secrets: '${{ toJSON(secrets) }}'
            chart: 'app'
            namespace: production
            release: production-myapp
            value-files: './config/production.yml'**
          env:
            KUBECONFIG_FILE: '${{ secrets.KUBECONFIG }}'

These arguments specify some new variables that will be familiar to Helm users. These values are briefly described below:

* Namespace — Kubernetes namespace for separating resources.

* Release — global name for the release used by Helm.

* Secrets — JSON encoded secrets available in value files.

* Chart — Helm chart to use.

* Token — GitHub token for access to GitHub api’s.

One of the most interesting ones to call out is the value-file argument. Since Helm charts often need to be customized for specific environments we can easily provide a value file which will be used as arguments for that specific deployment. An example of values that the “app” chart intakes is below:

    app:
      name: example-rails
      version: v1
    image:
      repository: myrepo
      tag: v1
    ingress:
      enabled: true
      hosts:
        - host: example.com  # Host the app at example.com
          paths: ["/"]
    secrets:
      - name: DATABASE_URL
        value: postgres://[123@test.com](mailto:123@test.com)

Simply provide different value files per environment to keep your configuration consistent and clean. You can read more in the [app](https://github.com/deliverybot/helm/tree/master/charts/app) chart repository for arguments and examples.

### Secret management

Secret management is tricky to get right with Kubernetes. The above Helm action doesn’t try and solve all problems related to secret management but it does have a feature which can get you up and running quickly. The chart allows template variables inside your value files. One of the objects available for use is a secrets object. You define this when running the Helm action by setting the following variable:

    with:
      **secrets: '${{ toJSON(secrets) }}'**

This is actually setting the secrets value to all secrets defined in our GitHub repository. Secrets are a new GitHub feature coming to all repositories for use with actions, if you have access to actions you should see it in your repository settings. Now, inside my value file, I can use the secrets key to create a Kubernetes application secret:

    # config/values.yml
    **secrets:
    - name: API_KEY
      value: '${{ secrets.API_KEY }}'**

The environment variable API_KEY will now be available inside my application after deployment. This is a simple, straightforward and relatively secure way of getting started with secret management on Kubernetes.

### Triggering deployments

![](https://cdn-images-1.medium.com/max/2000/1*o-qze3t7qY0lT7GF9CGNwg.gif)

This example presented so far relies on the GitHub deployments API. The deployments API is a flexible event driven API that works well with actions. A system initiates a deployment by calling the API. Your action then responds to this event and pulls out parameters like the commit and payload information to run the deployment.

We take advantage of a feature with the Helm action that it will pull values from the deployment event if this is what triggered the GitHub action. This allows us to parameterize our workflows outside of the action files. Here’s an example of triggering a deployment, it’s just using the curl command:

    curl -XPOST \
      [https://api.github.com/repos/:owner/:repo/deployments](https://api.github.com/repos/colinjfw/deliverybot-example/deployments) \
      -d '{
      "required_contexts": [],
      "ref": "master",
      "environment": "staging",
      "description": "Staging",
      "payload": {
        "value_files": ["./config/staging.yml"],
        "release": "staging-myapp",
        "namespace": "staging",
        "track": "stable",
        "values": {"replicaCount": 1,"version": "v1"}
      }
    }'

Let’s stop for a second. This is pretty cool! We now have a deployment method to Kubernetes that can be triggered using a simple curl command. This enables flexible automation around our deployments by making them API driven and available to scripting and more tools. This opens up deployments to be easily triggered by Slack, web applications or scripts. [Deliverybot](https://deliverybot.dev) provides a method to trigger deployments either automatically or manually when you are ready to deploy a commit to production.
> We now have a deployment method to Kubernetes that can be triggered using a simple curl command. This enables flexible automation around our deployments by making them API driven and available to scripting and more tools.

### Putting it all together

The following repository [github.com/deliverybot/example-helm](https://github.com/deliverybot/example-helm) contains a full example of using GitHub actions to deploy to Kubernetes. The full example contains the following files:

    ~/example-helm
    ├── .github
    │   ├── deploy.yml
    │   └── workflows
    │       └── cd.yml
    └── config
        ├── production.yml
        ├── review.yml
        └── staging.yml

Inside the config directory is all of our value files per environment while inside the top level .github folder contains our configuration and workflow files. You can get started using this repository by forking it and following these steps:

1. Install [deliverybot](https://github.com/apps/deliverybot) on the new repo.

1. Add a KUBECONFIG secret into the secrets tab. This contains our access to a Kubernetes cluster.

1. Push a commit to your new fork and watch the example workflows kick off.

To try creating a deployment using the deployments API you can also use the curl command above to kick off a deployment to the staging environment.

We’ve now followed through on a tutorial of using the GitHub deployments api, Helm and GitHub actions to deploy new code to Kubernetes. Comment below if you have feedback on this tutorial or ways in which you think it could be improved!
