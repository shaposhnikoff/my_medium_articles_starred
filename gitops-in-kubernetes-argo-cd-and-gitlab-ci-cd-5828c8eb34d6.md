Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m95[39m }

# GitOps in Kubernetes: How to do it with GitLab CI/CD and Argo CD

GitOps in Kubernetes: How to do it with GitLab CI and Argo CD

The world of Cloud Native in recent days is continuously speaking about GitOps. Indeed this model of Continuous Delivery is a kind of revolution in modern IT world. I’m not going to describe what GitOps is, because you can find so many resources in the Internet about it. An example [article](https://www.weave.works/technologies/gitops/) from the first google result exhaustively describes what it is. The following quotation appeals to me most easily “**Git as a single source of truth for declarative infrastructure and application**”, and I’m going to stick to it.

To find the definition of GitOps is relatively easy task, on the other hand to find the example of such workflow can be quite challenging. This is the main reason for me to write this story. I hope you can find here a lot of useful information which might help you create the GitOps in your current infrastructure setup.

## TLDR

* Introduction

* Argo CD setup

* GitLab Project setup

* GitLab CI Pipeline

* How it works?

* Summary

## Introduction

The below picture shows the GitOps workflow demonstrated in this story.

![GitOps Workflow](https://cdn-images-1.medium.com/max/2000/1*e4w0j0SUdsfx_U7hdHHpyw.png)*GitOps Workflow*

GitLab and Argo CD play the main role here, so I want to say a couple of words about them now.

**Argo CD** is a declarative, GitOps continuous delivery tool for Kubernetes. I like it, because its relatively easy to configure and use (requires basic Kubernetes knowledge). It comes with nice and easy to understand GUI. For me it was important, that Argo CD supports manifests in several ways (kustomize, helm, ksonnet or even jsonnet files). Applications can be configured in form of dedicated Custom Resource Definition added by Argo CD. It automates the deployment of desired application states in the specified target environments. More about the architecture, available features in official project [website](https://argoproj.github.io/argo-cd/).

**GitLab CI** as the name suggests is the Continuous Integration and Continuous Delivery tool from GitLab. For me, it’s very good choice for CI/CD tasks if compared to other competitors. I like it because creating pipelines is super easy (simple [**.gitlab-ci.yaml](https://docs.gitlab.com/ee/ci/yaml/)** file) if you compare it to other tools like Jenkins. For this demo, I’m going to use a free version that supports most of the necessary functions. So let’s get started!

## Argo CD Setup

I’m going to set up GitOps on [the Minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/) cluster hosted on my PC. The first step is to make sure, that I can access the cluster via kubectl. I usually list nodes for that purpose.

    ~> kubectl get nodes
    NAME   STATUS   ROLES    AGE   VERSION
    m01    Ready    master   42s   v1.17.3

I need also an ingress controller to access the Argo CD GUI. In Minikube, just enable extra add-ons.

    ~> minikube addons enable ingress

Next, I’m going to install the Argo CD using Helm (preferred version 3):

    ~> kubectl create namespace argocd
    ~> helm repo add argo [https://argoproj.github.io/argo-helm](https://argoproj.github.io/argo-helm)
    ~> helm install argocd -n argocd argo/argocd --values values.yaml

Here is the **values.yaml** file used by me in this example:

    appVersion: "1.4.2"
    version: 1.8.7

    server:
      ingress:
        enabled: true
        annotations: 
          kubernetes.io/ingress.class: "nginx"
          nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
          nginx.ingress.kubernetes.io/ssl-passthrough: "true"
          nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
        hosts:
          - argocd.minikube.local

After that, my Argo CD is installed. To access it, get the ingress address and add a necessary entry to your local /etc/hosts.

    ~> kubectl get ingress -n argocd
    NAME            HOSTS                   ADDRESS          PORTS   AGE
    argocd-server   argocd.minikube.local   192.168.99.105   80      15m

    ~> echo '192.168.99.105 argocd.minikube.local' | sudo tee -a /etc/hosts

Now, I can access the Argo CD GUI from my browser.

![](https://cdn-images-1.medium.com/max/2000/1*AbtlgpKVNcCgZXpkkSlpsg.png)

Default username is **admin**, password is the name of the pod. Example:

    ~> kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | cut -d'/' -f 2
    argocd-server-5cbcf6864-587hr

![Accessing Argo CD](https://cdn-images-1.medium.com/max/2000/1*bxyBj2No4sPBK8Ka_f-gMg.gif)*Accessing Argo CD*

That’s all for now. Next step is to create GitLab project with example code.

## GitLab Project Setup

For this demo, I have a small application written in Go. This is the web app that displays the text message and pod name next to it. Everything is in Git. Here is the repository:
[**Andrzej Kaczynski / gitops-webapp**
*GitOps example on sample web application*gitlab.com](https://gitlab.com/andrew.kaczynski/gitops-webapp)

GitLab comes not only with CI/CD, but also provides container registry for each project. I want to use it in my example.

It is also necessary to create an Access Token with API scope. This will be used later in the pipeline for git commits. You can create your own in** USER -> SETTINGS -> ACCESS TOKENS**

Next, add necessary Environment Variables to the project ( **Settings -> CI/CD -> Variables**):

* CI_PUSH_TOKEN — the token

* CI_USERNAME — the username of the token owner

## **ArgoCD Application Setup**

Its time to configure our applications in Kubernetes using GitOps. As I mentioned before, Argo CD comes with a set of CRDs which can be used to declarative configuration. Of course, this is the recommended way, because we want to keep our Infrastructure as a Code. Here are the manifests which configure web app for dev and prod environments:

    ---
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: web-app-dev
      namespace: argocd
    spec:
      project: default
      source: 
        repoURL: [https://gitlab.com/andrew.kaczynski/gitops-webapp.git](https://gitlab.com/andrew.kaczynski/web-app-gitops-example.git)
        targetRevision: HEAD
        path: deployment/dev
      destination:
        server: [https://kubernetes.default.svc](https://kubernetes.default.svc)
        namespace: dev
      syncPolicy:
        automated:
          prune: true
    ---
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: web-app-prod
      namespace: argocd
    spec:
      project: default
      source: 
        repoURL: [https://gitlab.com/andrew.kaczynski/gitops-webapp.git](https://gitlab.com/andrew.kaczynski/web-app-gitops-example.git)
        targetRevision: HEAD
        path: deployment/prod
      destination:
        server: [https://kubernetes.default.svc](https://kubernetes.default.svc)
        namespace: prod
      syncPolicy:
        automated:
          prune: true

The most important parts here are:

* name — the name of our Argo CD application

* namespace — must be the same as Argo CD instance

* project — project name where the application will be configured (this is the way to organize your applications in Argo CD)

* repoURL— URL of our source code repository

* targetRevision — the git branch you want to use

* path — the path where Kubernetes manifests are stored inside repository

* destination — Kubernetes destination-related things (in this case the cluster is the same where Argo CD is hosted)

Apply these manifests as any others using kubectl. Two objects type “application” will be created in your cluster.

    ~> kubectl get application -n argocd
    NAME           AGE
    web-app-dev    20m
    web-app-prod   17m

When you visit the Argo CD web dashboard, you can see those two applications.

![Application deployment in Argo CD](https://cdn-images-1.medium.com/max/2612/1*HrORJbzNdj03-J7NxjJ37A.gif)*Application deployment in Argo CD*

Click on one of them to see more details about the application and deployed Kubernetes objects. You can find the object inside the deployment/<env> directory in gitops-webapp repository. You can see that inside each folder, I created **kustomization.yaml**. Argo CD recognized it and apply using [**Kustomize](https://kustomize.io/) **without any additional settings!

I recommend you take a look at available options. The GUI is very intuitive, so you shouldn’t have any problems with understanding it.

## GitLab CI Pipeline

The next step is to create the pipeline, which will automatically build our application, push to the container registry and update Kubernetes manifests in desired environments.

The below example is far from being perfect (missing tests etc.), but it should demonstrate to you the entire GitOps workflow.

Basically, the idea is, that developers are working on their own branches. For each commit and push of their branch, the stage build will be triggered. When they merge their changes with the master branch, the full pipeline will be triggered. It will build the application, containerize it, push an image to the registry and automatically update Kubernetes manifests depending on the stage. In addition, deploying to Prod requires manual action from DevOps.

Pipeline definition in GitLab CI is stored in **.gitlab-ci.yml** file in the root directory of the project.

At the top of the file are the definition of stages and default environment variables, images or before script steps. Excerpt from my pipeline:

    stages:
    - build
    - publish
    - deploy-dev
    - deploy-prod

Next is the stage definition and required tasks. The build process here is very simple. It just executes one command inside docker image golang. Then it saves the artifact for the next stages. This step works for any new commit in any branch.

    build:
      stage: build
      image:
        name: golang:1.13.1
      script:
        - go build -o main main.go
      artifacts:
        paths:
          - main
      variables:
        CGO_ENABLED: 0

The next stage is to build an image and push it to the GitLab registry. Here I’m using Kaniko, because the image is built inside the container which has no access to the docker engine. The image is building with current commit hash (variable $CI_COMMIT_HASH) and pushing to the GitLab registry. This stage is triggered only on changes in the master branch.

    publish:
      stage: publish
      image:
        name: gcr.io/kaniko-project/executor:debug
        entrypoint: [""]
      script:
        - echo "{\"auths\":{\"$CI_REGISTRY\":{\"username\":\"$CI_REGISTRY_USER\",\"password\":\"$CI_REGISTRY_PASSWORD\"}}}" > /kaniko/.docker/config.json
        - /kaniko/executor --context $CI_PROJECT_DIR --dockerfile ./Dockerfile --destination $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
      dependencies:
        - build  
      only:
        - master

The next stage is to deploy the application to a dev environment. In GitOps it means, to update the Kubernetes manifests so Argo CD can pull updated version and apply the change. Here I’m using environment variables defined for the project with username and token which are needed to push the changes to the master branch. When GitLab CI receives commit with message [skip ci], the pipeline is not triggered (we don’t want to run it again when we publish changes related to our kustomize manifests)

    deploy-dev:
      stage: deploy-dev
      image: alpine:3.8
      before_script:
        - apk add --no-cache git curl bash
        - curl -s "[https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh](https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh)"  | bash
        - mv kustomize /usr/local/bin/
        - git remote set-url origin https://${CI_USERNAME}:${CI_PUSH_TOKEN}[@gitlab](http://twitter.com/gitlab).com/andrew.kaczynski/gitops-webapp.git
        - git config --global user.email "[gitlab@gitlab.com](mailto:gitlab@gitlab.com)"
        - git config --global user.name "GitLab CI/CD"
      script:
        - git checkout -B master
        - cd deployment/dev
        - kustomize edit set image $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
        - cat kustomization.yaml
        - git commit -am '[skip ci] DEV image update'
        - git push origin master
      only:
        - master

Finally, we have a deployment to prod environment. It is very similar to the previous one but requires manual action before it starts.

    deploy-prod:
      stage: deploy-prod
      image: alpine:3.8
      before_script:
        - apk add --no-cache git curl bash
        - curl -s "[https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh](https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh)"  | bash
        - mv kustomize /usr/local/bin/
        - git remote set-url origin https://${CI_USERNAME}:${CI_PUSH_TOKEN}[@gitlab](http://twitter.com/gitlab).com/andrew.kaczynski/gitops-webapp.git
        - git config --global user.email "[gitlab@gitlab.com](mailto:gitlab@gitlab.com)"
        - git config --global user.name "GitLab CI/CD"
      script:
        - git checkout -B master
        - git pull origin master
        - cd deployment/prod
        - kustomize edit set image $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
        - cat kustomization.yaml
        - git commit -am '[skip ci] PROD image update'
        - git push origin master
      only:
        - master
      when: manual

This is the full pipeline definition. I hope you see how easy is to create a pipeline in GitLab when compare to Jenkins :) That’s why I love it!

## How it works?

Now let’s see how it all works together. First, get the IPs of application ingresses and add necessary entries to your local /etc/hosts file.

    ~> kubectl get ingress --all-namespaces |grep gitops
    dev         gitops-webapp   webapp.dev.minikube.local    192.168.99.105   80      25m
    prod        gitops-webapp   webapp.prod.minikube.local   192.168.99.105   80      25m

    ~> echo "192.168.99.105 webapp.dev.minikube.local" | sudo tee -a /etc/hosts

    ~> echo "192.168.99.105 webapp.prod.minikube.local" | sudo tee -a /etc/hosts

Now you can use your browser to display the web application message. Let’s see the dev webapp.

![Dev web app](https://cdn-images-1.medium.com/max/2000/1*7N-W_cwUR4hDfBkrDQRkYA.png)*Dev web app*

Then its time to make a small change and push it to the master. Simply edit main.go file and change the name ANDREW to GITOPS in variable welcome.

    func main() {
       welcome := Welcome{"GITOPS", time.Now().Format(time.Stamp), os.Getenv("HOSTNAME")}

Commit and push the change to the master. Then go to GitLab project -> CI/CD -> Pipelines. You will see a new pipeline which has just started.

![GitLab CI/CD Pipeline](https://cdn-images-1.medium.com/max/2584/1*HKvr8FH804gh1QYON0oHSQ.png)*GitLab CI/CD Pipeline*

Wait a couple of minutes, you will see a new pipeline with status skipped when deploying to dev stage is done (the pipeline already updated the manifest for dev). Argo CD in auto-sync mode will update the Kubernetes state in less than a minute. You can see the progress in the Argo GUI. Check again the webapp to see an updated message.

![GitLab CI/CD Pipeline](https://cdn-images-1.medium.com/max/2524/1*WNd42lZYHvVSZxeQJX7nvQ.png)*GitLab CI/CD Pipeline*

![Updated webapp message](https://cdn-images-1.medium.com/max/2000/1*jzkDSpk4KmFM-arXxzJFIQ.png)*Updated webapp message*

Finally, manually trigger the prod build by approving the pipeline stage. After that, the image in prod will be updated as well.

![GitLab CI/CD Prod deployment](https://cdn-images-1.medium.com/max/2440/1*MiJJMQylGhi6kMWLVir7DA.gif)*GitLab CI/CD Prod deployment*

Here is the Argo CD updated dashboard when Sync is ready.

![](https://cdn-images-1.medium.com/max/3044/1*kfHC2zovxjI1QAgOwHvuhA.png)

And this is all, my application has been successfully deployed to dev and prod environment using GitOps!

## Summary

I hope you enjoyed the reading of my post. Maybe one day you will switch your Kubernetes environment to GitOps which is awesome. If you have any comments or questions, I’m always happy to discuss it with you :-)
