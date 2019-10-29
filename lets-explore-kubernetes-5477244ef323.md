Unknown markup type 10 { type: [33m10[39m, start: [33m71[39m, end: [33m78[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m109[39m, end: [33m113[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m121[39m, end: [33m128[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m31[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m101[39m, end: [33m115[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m166[39m, end: [33m173[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m63[39m, end: [33m80[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m226[39m, end: [33m233[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m168[39m, end: [33m175[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m158[39m, end: [33m165[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m160[39m, end: [33m166[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m90[39m, end: [33m108[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m143[39m, end: [33m157[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m71[39m, end: [33m98[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m114[39m, end: [33m139[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m174[39m, end: [33m184[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m102[39m, end: [33m116[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m10[39m, end: [33m26[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m96[39m, end: [33m103[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m145[39m, end: [33m184[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m47[39m, end: [33m54[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m17[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m109[39m, end: [33m114[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m183[39m, end: [33m188[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m317[39m, end: [33m322[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m386[39m, end: [33m392[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m413[39m, end: [33m420[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m464[39m, end: [33m476[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m511[39m, end: [33m523[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m557[39m, end: [33m568[39m }

# Let’s Explore Kubernetes

Kubernetes + Gitlab + Continuous Integration & Deployment

I’m going to walk through my experience of setting up a new Kubernetes cluster (or k8s for short) with DigitalOcean, configuring my Gitlab project to use the k8s cluster, and configuring a CI/CD process for deployments. Keep on reading if you would like to see how simple it is to get a modern stack up and running.

![Photo by [Fatos Bytyqi](https://unsplash.com/@fatosi?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/10368/0*uv6xXJ_cYliFFAEY)*Photo by [Fatos Bytyqi](https://unsplash.com/@fatosi?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

## Building a Kubernetes Cluster

Kubernetes is a container orchestration platform gaining lots of popularity due to its simplicity. Kubernetes is great as you can define your deployment configuration, storage, and network using config files and the cluster will ensure your application is always running in that configuration.

Building a k8s cluster from source is a daunting task, however we can do this with a few clicks from the big cloud providers. I personally prefer the simplicity of DigitalOcean, and am lucky enough to be a part of the LTD release of their managed Kubernetes offering.

Let’s dive in to how to create a cluster on DigitalOcean.

Once you click on the create Kubernetes option, either via the side nav or the dropdown in the top nav, you are presented with this screen.

![DigitalOceans’ Kubernetes Cluster Creation](https://cdn-images-1.medium.com/max/4796/1*QnCh8Cm0d58UIhFBjFnpog.png)*DigitalOceans’ Kubernetes Cluster Creation*

At the time of this writing k8s version 1.13.1 was the latest release, so I’ve selected that version in a region closest to me.

The next step is to configure node pools, tags, and select a name. I’ve chosen to keep it simple with a single low-cost node for learning purposes. This can be changed in the future, so starting small will not limit your future capacity.

Tags are optional, and the name can be anything you would like. I find it useful to add the “k8s” tag to quickly identify droplets in the cluster.

![Configure nodes, tags, and a name](https://cdn-images-1.medium.com/max/4840/1*65S3eamCNjeysrXXSLVOvQ.png)*Configure nodes, tags, and a name*

Once you click “Create Cluster” the process will take about 4–5 minutes to complete. During this time we can get your machine setup to connect to your new k8s cluster.

The primary command line utility for interacting with a k8s cluster is kubectl. For MacOS users, you can use brew to install it by running the following command.

    ➜ brew install kubernetes-cli

Once brew has completed the installation, you will need to download the cluster config file from DigitalOcean to let the kubectl command know where your cluster is located. To do so, scroll all the way down on the DigitalOcean k8s cluster install page to the following section and click on the “Download Config File” button

![](https://cdn-images-1.medium.com/max/2000/1*7IygVOpv-yW6zs54grQZIw.png)

The file will be saved to your ~/Downloads directory. To make thing easier, copy or move the file to ~/.kube/config file. This file will be read automatically by the kubectl command.

    ➜ mkdir -p ~/.kube
    ➜ mv ~/Downloads/[k8s-cluster].yaml ~/.kube/config

Once the cluster is created, test your connectivity by running kubectl get nodes . This will show you the single node in the cluster.

    ➜ kubectl get nodes

    NAME                   STATUS    ROLES     AGE       VERSION
    tender-einstein-8m4m   Ready     <none>    21m       v1.13.1

In my case, the node (which is a DigitalOcean droplet) was named “tender-einstein-8m4m”, as we can see above. If you see similar output, your Kubernetes cluster was successfully created and you have connectivity to it via the kubectl command line utility.

## Connecting Gitlab to Kubernetes

Gitlab has a native integration with Kubernetes and we can configure any group or project to use it. You will need elevated (project creator and/or administrator) privileges on your Gitlab project to setup the k8s integration.

To begin, first select the Kubernetes tab under the Operations menu, then click on Add Kubernetes Cluster.

![Gitlab — Add Kubernetes Cluster](https://cdn-images-1.medium.com/max/4004/1*bKPrCf_V31kfMT50YllW8Q.png)*Gitlab — Add Kubernetes Cluster*

On the next screen click on the Add Existing Cluster tab. Here you will be prompted to input a few different items to allow Gitlab to connect to your k8s cluster. Gitlab has excellent documentation on [how to add a cluster](https://gitlab.com/help/user/project/clusters/index#adding-an-existing-kubernetes-cluster), which I recommend reading to get a thorough understanding of the integration. I will highlight the required steps here.

### Creating Accounts

First up, we will need to create a new system-level account for the Gitlab to connect with. This account is called a ServiceAccount. In order to do this we can use the kubectl command line utility. We will define the account using YAML syntax (which is used throughout k8s) as seen below:

    ➜ kubectl create -f - <<EOF
      apiVersion: v1
      kind: ServiceAccount
      metadata:
        name: gitlab
        namespace: default
    EOF

This YAML definition will create a ServiceAccount named “gitlab” in the “default” namespace.

Next step is to give the gitlab account cluster administrator privileges so it can freely create and destroy services on your behalf. Once again, we will use kubectl and a YAML definition.

    ➜ kubectl create -f - <<EOF
    kind: ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: gitlab-cluster-admin
    subjects:
    - kind: ServiceAccount
      name: gitlab
      namespace: default
    roleRef:
      kind: ClusterRole
      name: cluster-admin
      apiGroup: rbac.authorization.k8s.io
    EOF

### Connect the Cluster

Now, lets take a look at all the information that Gitlab requires to connect to the Kubernetes cluster.

![Gitlab — The form presented when adding a Kubernetes cluster](https://cdn-images-1.medium.com/max/3232/1*OSq2NV2wXcbyqLsmRBqepw.png)*Gitlab — The form presented when adding a Kubernetes cluster*

The first field is Kubernetes cluster name. This can be anything meaningful to you to help you identify the k8s cluster. It’s not really used that much so don’t spend too much time coming up with a name for it.

The next field API URL can be obtained by running the following command:

    ➜ kubectl cluster-info | grep 'Kubernetes master' | awk '/http/ {print $NF}'

    https://xxxxxx.k8s.ondigitalocean.com

Grab the URL returned by the command and paste it into the API URL field.

The CA Certificate and the Token can be obtained by extracting data from the “secret” created when the gitlab account was made. Kubernetes has the concept of a secret resource which is designed to store sensitive information. In addition to the setup process, you can create your own secrets to store your applications sensitive information such as database credentials, API keys, etc…

To list all secrets in the project run:

    ➜ kubectl get secrets                                                                                                                            

    NAME                  TYPE                                  DATA
    default-token-xfxg9   kubernetes.io/service-account-token   3 
    gitlab-token-vxhxq    kubernetes.io/service-account-token   3 

Here we see there are 2 secret objects in our cluster. We are interested in the one named gitlab-token-vxhxq. Find the secret that starts with gitlab-token-* and use it in the next command:

    ➜ kubectl get secret <SECRET_NAME> -o jsonpath="{['data']['ca\.crt']}" | base64 --decode

    -----BEGIN CERTIFICATE-----

    ...

    -----END CERTIFICATE-----

Copy and paste everything returned from the command, starting with the -----BEGIN CERTIFICATE----- and ending with-----END CERTIFICATE-----, into the CA Certificate field

The Token can be obtained in a similar fashion by executing the following:

    ➜ kubectl get secret <SECRET_NAME> -o jsonpath="{['data']['token']}" | base64 --decode

    WlhsS2FHSkhZMmxQYVVwVFZYc...

Once again, copy paste the returned value into the Token field within Gitlab.

As for Project Namespace, I have left that blank. Ensure to check the RBAC enabled cluster checkbox. When you are ready, go ahead and click the Add Kubernetes Cluster button.

![Gitlab — All required cluster details are filled in](https://cdn-images-1.medium.com/max/3272/1*6JTC4TNuLRp3Cprs1CFzVw.png)*Gitlab — All required cluster details are filled in*

## Setting up CI/CD

Setting up a Continuous Integration and Continuous Deployment pipeline is very simple in Gitlab. It’s baked into the Gitlab offering and easily configurable by just adding a .gitlab-ci file to the root of your project. A CI/CD pipeline is triggered when you push code to the Gitlab repo. The pipeline must run on a server, which is called a “Runner”. Runners can be virtual private servers, public servers, or anywhere you can install the Gitlab runner client. In our use case, we are going to install a runner on the k8s cluster so that jobs are executed in pods. This client also makes it scaleable so we can run multiple jobs in parallel.

To install the Gitlab runner client on the cluster, we will first need to install another tool named Helm. Helm is a package manager for Kubernetes and simplifies installation of software. I like to think of Helm being similar to Brew for Mac, they both have a repo of software that can be installed onto a system.

### Installing Helm

Installing Helm through Gitlab just requires you to click the Install button. Assuming everything was configured properly, it will take just a few seconds to install.

![Gitlab — Install Helm tiller on the k8s cluster](https://cdn-images-1.medium.com/max/3896/1*ybNwklkvLn4ShWVYfOnAcA.png)*Gitlab — Install Helm tiller on the k8s cluster*

After that has completed, lets take a peek at the cluster to see what Gitlab has installed. Using the kubectl get ns command we can see that Gitlab has created its own namespace, named gitlab-managed-apps.

    ➜ kubectl get ns 

    NAME                  STATUS    AGE
    default               Active    1d
    **gitlab-managed-apps   Active    23s**
    kube-public           Active    1d
    kube-system           Active    1d

If we run kubectl get pods we won’t see anything as the default namespace when not specified is default. To see pods in the Gitlab namespace run kubectl get pods -n gitlab-managed-apps.

    ➜ kubectl get pods -n gitlab-managed-apps

    NAME                             READY     STATUS    RESTARTS   AGE
    tiller-deploy-7dd47f89cc-27cmt   1/1       Running   0          5m

Here we see that the “Helm tiller” has been created successfully and is running.

### Installing the Gitlab Runner

As stated earlier, the Gitlab runner allows our CI/CD jobs to be run in the k8s cluster. Installing the runner with Gitlab is simple, just click the Install button.

![Gitlab — Install the runner on the k8s cluster](https://cdn-images-1.medium.com/max/3880/1*3PYiTgna0HqismGFb8Y7RA.png)*Gitlab — Install the runner on the k8s cluster*

This took about 1 minute for my cluster. After its completed, let’s take a look at the pods again and you should see a new pod for the Gitlab runner.

    ➜ kubectl get pods -n gitlab-managed-apps 

    NAME                                    READY     STATUS    RESTARTS
    **runner-gitlab-runner-5cffc648d7-xr9rq   1/1       Running   0**
    tiller-deploy-7dd47f89cc-27cmt          1/1       Running   0

You can verify that the runner is connected to your project by viewing the Settings ➜ CI/CD ➜ Runners section within Gitlab.

![Gitlab — Kubernetes runner is activated for the project](https://cdn-images-1.medium.com/max/2000/1*R2d7hMEzFgWKbgvZp5D1-w.png)*Gitlab — Kubernetes runner is activated for the project*

### Run a Pipeline

Great, so now we have a fully functional Gitlab project, connected to Kubernetes with runners ready to execute our CI/CD pipelines. Let’s setup an example Golang project to see how these pipelines can be triggered. For this project we will run a simple HTTP server that returns the classic “Hello World”.

First, write the go code:

    # main.go
    package main

    import (
     "fmt"
     "log"
     "net/http"
    )

    func main() {

    http.HandleFunc(
      "/hello",
      func(w http.ResponseWriter, r *http.Request) {
       fmt.Fprintf(w, "Hello World!")
      },
     )

    log.Fatal(http.ListenAndServe(":8080", nil))
    }

And then the Dockerfile to run it:

    # Dockerfile
    FROM golang:1.11

    WORKDIR /go/src/app
    COPY . .

    RUN go get -d -v ./...
    RUN go install -v ./...

    CMD ["app"]

Next we will create a .gitlab-ci.yml file to define our CI/CD pipeline. The file will be evaluated on every code push and if the branch, or tags, match any jobs, they will be executed automatically by one of the Gitlab runners that we configured earlier.

The first step in our pipeline will be to create a docker image of our application whenever we push to the master branch. We can do so with the following configuration:

    # Gitlab CI Definition (.gitlab-ci.yml)
    stages:
      - build
      - deploy

    services:
      - docker:dind

    variables:
      DOCKER_HOST: tcp://localhost:2375

    build_app:
      image: docker:latest
      stage: build
      only:
        - master
      script:
        - docker build -t ${CI_REGISTRY}/${CI_PROJECT_PATH}:${CI_COMMIT_REF_NAME} .
        - docker login -u gitlab-ci-token -p ${CI_BUILD_TOKEN} ${CI_REGISTRY}
        - docker push ${CI_REGISTRY}/${CI_PROJECT_PATH}:${CI_COMMIT_REF_NAME}

Let’s walk through each block in the file. The stages: block defines the order of stages in our pipeline. We only have 2 stages, build and then deploy.

The services: block includes the official Docker-in-Docker (or dind) image, that will be linked in all jobs. We need this as we will be our application docker container inside of the Gitlab CI docker containers.

Next we have the build_app: job. This name is made up by our project and can be anything you would like. The image indicates we are using the latest Docker image from Docker Hub. The stage tells Gitlab what stage this job is in. One neat thing to keep in mind is that jobs in the same stage will run in parallel. The only: tag indicates that we will only run this job on commits to the master branch. Finally the script: is the meat of the job, which will run the docker build command to create our image, then docker login to the Gitlab registry, and then docker push that image to our registry.

At this point we can commit and push the code and you should see a brand new image in the Gitlab registry.

After an image is built and saved on the registry, the next step is to deploy it. We need to define the deployment configuration that tells kubernetes how we want to run the application. The following yaml file is exactly that:

    # Deployment Configuration (deployment-template.yaml)
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: example-deployment
      labels:
        app: example
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: example
      template:
        metadata:
          labels:
            app: example
        spec:
          containers:
          - name: example
            image: **registry.gitlab.com/thisiskj/example:latest**
            ports:
            - containerPort: 8080

This file defines a deployment, with 1 single replica that will run the image from the project (registry.gitlab.com/thisiskj/example:latest).

To trigger a deployment, I have configured the .gitlab-ci.yml file to do so when the code repo is tagged. Here is the job definition to do so:

    deploy_app:
      image: thisiskj/kubectl-envsubst
      stage: deploy
      environment: production
      only:
        - tags
      script:
        - envsubst \$CI_COMMIT_TAG < deployment-template.yaml > deployment.yaml
        - kubectl apply -f deployment.yaml

This job will run the envsubst command to replace the $CI_COMMIT_TAG variable inside of the deployment-template.yaml, with the name of the git tag that triggered the build. The environment variable $CI_COMMIT_TAG is set by the Gitlab runner and we tell envsubst to essentially search and replace that variable within the file.

### Viewing the Application

At this point everything is wired up and our deployment will run on every new tag.

We can see the running pod:

    ➜  kubectl -n example-10311640 get pods
    NAME                                  READY   STATUS    RESTARTS
    example-deployment-756c8f6dc5-jk85w   1/1     Running   0

Now, thats great that the pod is running, however we cannot access the golang HTTP service externally. To allow external access, we can create a service of type LoadBalancer. Add the following spec to the deployment yaml to create a LoadBalancer on DigitalOcean.

    ---
    kind: Service
    apiVersion: v1
    metadata:
      name: example-loadbalancer-service
    spec:
      selector:
        app: example
      ports:
      - protocol: TCP
        port: 80
        targetPort: 8080
      type: LoadBalancer

On the next deployment, we can monitor the creation of the LoadBalancer. It might take a few minutes for the external IP to appear.

    ➜  kubectl -n example-10311640 get services
    NAME                           TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE

    example-loadbalancer-service   LoadBalancer   10.245.40.9   <pending>     80:30897/TCP   4s

    ...

    ➜  kubectl -n example-10311640 get services
    NAME                           TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)        AGE

    example-loadbalancer-service   LoadBalancer   10.245.40.9   157.230.64.204   80:30897/TCP   2m9s

We can also monitor the load balancer creation within the DigitialOcean console:

![DigitalOcean — Networking console](https://cdn-images-1.medium.com/max/4700/1*MtctDTZVA8EBNv7GF3pXKA.png)*DigitalOcean — Networking console*

Finally, we can view our application by navigating to the IP address in our browser:

![The service is running!](https://cdn-images-1.medium.com/max/2000/1*-oi9wlZQKzPhp5CC8j1Pkg.png)*The service is running!*

### Wrapping Up

I hope you found this walkthrough to be helpful in setting up your own cluster and modern DevOps workflow.

You can view all the code related to this project on [https://gitlab.com/thisiskj/example](https://gitlab.com/thisiskj/example)

If you have any other ways to integrate Gitlab CI/CD with your Kubernetes deployments, please feel free to share in the comments.
