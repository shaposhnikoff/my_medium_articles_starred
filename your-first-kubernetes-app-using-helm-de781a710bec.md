
# Your first Kubernetes app using Helm



I wrote an article on how to install Kubernetes on VirtualBox to play around with a little. This is a follow up on that article. You should be able to follow along if you are very knowledgeable without reading the article, but I highly recommend just going through it as it’s a short enough tutorial [https://medium.com/@tedjohanssondeveloper/kuberenets-in-minutes-c5892c04078b](https://medium.com/@tedjohanssondeveloper/kuberenets-in-minutes-c5892c04078b).

This tutorial will be about getting [Helm ](https://helm.sh/)running and even throwing up your first flask application on the cluster. I would like to reiterate that this is not for production applications, that’s a whole other story but more for you who want to start learning.

Helm helps you manage deployment of your applications on Kubernetes. It’s a templating tool for Kubernetes components which means that under it all are just Kubernetes [YAML](https://en.wikipedia.org/wiki/YAML)s with some tools to help you organize them.

Helm has a [bunch of community driven charts](https://github.com/helm/charts) which is what Helm calls their configurations, that makes it easy to bundle your app with third party tools like MongDB, Postgres, RabbitMQ and many more. You can also just run community driven charts without your own app if you want something like a Jenkins server.

## Setup kubectl on host

We have already installed kubectl on the master node so all we need to do is the same on our host.

### Install kubectl

As I’m using Ubuntu on Windows I will install using the Ubuntu guide over at Kubernetes but if you are using some other OS please check out the guide over there ([https://kubernetes.io/docs/tasks/tools/install-kubectl/](https://kubernetes.io/docs/tasks/tools/install-kubectl/)).

As always it’s good to keep you machine up to date

    **sudo apt-get update && sudo apt-get install -y apt-transport-https**

Add the apt key from google

    **curl -s [https://packages.cloud.google.com/apt/doc/apt-key.gpg](https://packages.cloud.google.com/apt/doc/apt-key.gpg) | sudo apt-key add -**

Ass kubernetes repo to the source list

    **echo “deb [https://apt.kubernetes.io/](https://apt.kubernetes.io/) kubernetes-xenial main” | sudo tee -a /etc/apt/sources.list.d/kubernetes.list**

Update the repos and install kubectl

    **sudo apt-get update && sudo apt-get install -y kubectl**

### Configure kubectl

In my last post on how to set up a Kubernetes cluster on VirtualBox we configured kubectl on the master node so we should at this point know where the config is located.

    mkdir -p $HOME/.kube

I’ll be using scp to fetch my file from the master node. As I named the VM master-node all I need to run is. Where <user> is your default user on the master node.

    scp <user>@master-node:$HOME/.kube/config $HOME/.kube/config

Make sure your current user has the correct permissions as well.

    sudo chown $(id -u):$(id -g) $HOME/.kube/config

Check if it worked

    kubectl get pods -A

## Installing helm

### Install helm on host machine

If you are using Ubuntu or linux 64 bit as me you can follow along here otherwise you can find other instructions on Helms website [https://helm.sh/docs/using_helm/#installing-helm](https://helm.sh/docs/using_helm/#installing-helm).

It’s fairly simple, just download a tar, untar and move it into the correct location.

You can find the latest version over at [https://github.com/helm/helm/releases](https://github.com/helm/helm/releases) I’ll be using Helm v2.14.0 which is the latest as of writing.

    wget [https://get.helm.sh/helm-v2.14.0-linux-amd64.tar.gz](https://get.helm.sh/helm-v2.14.3-linux-amd64.tar.gz)

Untar the file

    tar -zxvf helm-v2.14.3-linux-amd64.tar.gz

Move it to bin for global calls

    sudo mv linux-amd64/helm /usr/local/bin/helm

### Initialize helm tiller

The Helm tiller is the server part of your Helm setup. This should sit as a pod on the Kubernetes cluster. As the name implies the tiller is what controls the ship that is Kubernetes.

Now that we have helm installed on our host we can initialize it

First let’s create a [service account](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/) and a [cluster role binding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#default-roles-and-role-bindings) as we are using [rbac](https://kubernetes.io/docs/reference/access-authn-authz/rbac/).

    vim service-account-tiller.yaml

And enter the following

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

Then we create the service account and the cluster role binding

    kubectl create -f service-account-tiller.yaml

Currently there is a bug in Kubernetes v1.16.0 and helm v2.14.3 that breaks initializing the tiller ([https://github.com/helm/helm/pull/6462](https://github.com/helm/helm/pull/6462)).

    helm init — service-account tiller — override spec.selector.matchLabels.’name’=’tiller’,spec.selector.matchLabels.’app’=’helm’ — output yaml | sed ‘s@apiVersion: extensions/v1beta1@apiVersion: apps/v1@’ | kubectl apply -f -

If the bug is patch when you are doing this use the following command

    helm init — history-max 100 — service-account tiller

It might take awhile before the tiller is installed, you can check if the pod is starting up, just look for the tiller pod in.

    kubectl get pods -A

Let’s check if everything went ok.

    helm version

You’re looking for the server version here.

![](https://cdn-images-1.medium.com/max/2000/0*60S4XDsV1p_StiBn)

## Test deploy helm chart

Let’s update the repo first to make sure we have the latest and greatest of the charts.

    helm repo update

Deploy your first chart, I picked heartbeat since it doesn’t require any extra configurations.

    helm install --name heartbeat stable/heartbeat

Now you should see the helm deploy in the list

    helm ls

And you should see your pod being created in kubernetes

    kubectl get pods

Once the pods is open we can see what nonsense it’s spitting out. There should be some load statistics of the VM but nothing else really

    kubectl --namespace=default logs -l “app=heartbeat,release=heartbeat”

Now that we can see it worked we can just throw it out.

    helm delete heartbeat --purge

And it’s gone

    helm ls

## Your Hello World app

## Creating a docker image

Create account on dockerhub and create a public repo [https://hub.docker.com](https://hub.docker.com/).

My repo is called tedjohansson/hello-world:latest if you want to skip creating a docker image yourself.

I created my docker image on the master node as I am running on Windows and it’s a pain in the neck to get Docker running there.

If you want to use my flask app to test with and maybe change the value you can go ahead and clone it to the master node

    git clone [https://github.com/TedJohansson/hello-world-helm.git](https://github.com/TedJohansson/hello-world-helm.git)

I recommend to change the return value in main.py to something like “Hello Me, from Kubernetes” or something

Then you want to login to your dockerhub account

    sudo docker login

Then build the docker images where <remote-repo> is the name of your repo (mine is (tedjohansson/hello-world)

    sudo docker build --tag <remote-repo>:latest

Then push the image you just created to dockerhub

    sudo docker push <remote-repo>:latest

## Creating you first helm chart

Let’s just create a Helm chart using helms builtin command on the host machine.

    helm create hello-world

The default image Helm uses is [nginx ](https://www.nginx.com/)so we should see that when we run install

    helm install hello-world --name hello-world

Now to see that it has installed alright we can just port forward from the service that was created.

    kubectl port-forward svc/hello-world 8080:80

Now visit localhost on port 8080 in your browser.
> localhost:8080

You should see a welcome message to nginx. But we want to make our own app so delete that junk.

    helm delete hello-world --purge

That’s cool and all but we want to use the application we just created to show not some nginx standard message. This means we need to change some values in our deploy.

Open up the values.yaml file that was created in the hello-world directory. This is where a lot of the configuration should be done for you helm chart.

    vim hello-world/values.yaml

To pick your images you want to change where <remote-repo> is the repo created on dockerhub (mine is tedjohansson/hello-world). I like to have pullPolicy: Always so I can push my docker image and update it every time.

    image:
      repository: nginx
      tag: stable
      pullPolicy: IfNOtPresent

To

    image:
      repository: <remote-repo>
      tag: latest
      pullPolicy: Always

Let’s also change replicaCount to 3

    replicaCount: 3

Now that we have update our config a little we can deploy the same chart as before but this time it should be our app.

    helm install hello-world --name hello-world

Port forward

    kubectl port-forward svc/hello-world 8080:80

And there it is in the browser
> localhost:8080

Let’s just clean up before we leave it.

    helm delete hello-world --purge

No that’s only the beginning of what you can do with Helm and Kubernetes but now you have a starting point. Start building your own app and learn more. I would recommend trying to set up a microservice app architecture to see where Kubernetes really shine.

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
