
# CI/CD With Argo On Kubernetes

Continuous Integration and Continuous Delivery is something everyone strives to. It makes everything so much easier. Of course, there are a lot of CI/CD tools out there but with the current Kubernetes movement, all of those tools need to be revised. Jenkins, for example, is a very mature CI/CD tool that is widely used across the globe but lacks innovation and feels rather heavy. The same can be said about Spinnaker. Great for enterprise solutions that have the resources to go in-depth about making it work, but not ideal for spinning up a CI/CD tool up fast and in a clean way. There are other tools out there that have more support for easier workflows and canary deployments. One of those tools I came across is Argo.

![](https://cdn-images-1.medium.com/max/2000/1*0n66zZ2D8-vDkoCDqRbiJQ.png)

What Argo does differently is how they manage the actual CI/CD. It is specifically developed for Kubernetes and integrates with it through CRD’s (Custom Resource Definitions). It defines a new CRD which is the ‘**Workflow**’. In this workflow you define what needs to happen by laying out steps in a yaml format. Each step runs in **its own Docker container** on your own Kubernetes cluster.
[**argoproj/argo**
*ArgoProj: Get stuff done with Kubernetes. Contribute to argoproj/argo development by creating an account on GitHub.*github.com](https://github.com/argoproj/argo)

### Argo/Argo CD/Argo CI

The Argo Project has several repositories that they’re working on. [**Argo](https://github.com/argoproj/argo)** is the main project which focuses on Kubernetes workflows, which can be used in a very generic way. [**Argo CD](https://github.com/argoproj/argo-cd)** is the GitOps way of handling deployments, meaning that git repositories are the single source of truth and that your Kubernetes cluster mirrors everything from those repositories. [**Argo CI](https://github.com/argoproj/argo-ci)** is not maintained anymore from the looks of it, but it should have been a CI tool that triggers workflows based on git changes. In order to set up a CI/CD tool, you will need Argo and its workflows but also a CI tool that will trigger these. Because Argo CI is not actively developed upon, I wrote my own Argo CI, which triggers Argo workflows through Bitbucket webhooks. By using my own CI tool, I managed to build a fully functioning CI/CD tool with Argo.
[**BouweCeunen/argo-continuous-integration**
*Continuous integration with Argo which support bitbucket private webhooks. - BouweCeunen/argo-continuous-integration*github.com](https://github.com/BouweCeunen/argo-continuous-integration)

### Argo Workflow

Argo has its own CRD, which is the Workflow. It has metadata which consists of a **generateName**. This will be the prefix of the name of the pods in which your workflow steps will run. It is possible to define **volumes**, like you would specify those in an ordinary Kubernetes context. They can be used in the templates you define after. The **arguments** of your workflow can consist of your repository names, revisions etc. After you set your configuration right, you can begin to define **templates **which are your workflow steps. You can also define a template which will hold other templates which I did in my case. You define a *cicd* template which will be your entrypoint. This template contains multiple steps, which in turn are all other templates. Each template can have input arguments, which are used to pass data between your workflow steps. It is up to you how many steps you define. Remember that each step will run in its own Docker container, fully utilizing your Kubernetes cluster resources and no hassle of spinning up EC2 instances on AWS. Which for example can be the case with Jenkins.

The ‘*checkout*’ template will pull a repository and pass this on to the other templates in order to use it. It will also pass on the git commit which can be used as an image tag. The ‘*build-push-docker*’ will build and push Docker containers to your Docker Registry. To run your tests, you have a ‘*run-tests’* template and if all these steps succeed, it will be deployed to your Kubernetes cluster by the ‘*deploy-kubernetes*’ template.

    apiVersion: argoproj.io/v1alpha1
    kind: Workflow
    metadata:
      namespace: argo
      generateName: crypto-gathering-backend--
    spec:
      hostNetwork: true
      serviceAccountName: argo
      entrypoint: cicd
      volumes:
      - name: docker-config
        secret: 
          secretName: regcred
      arguments:
        parameters:
        - name: repo
          value: [git@bitbucket.org](mailto:git@bitbucket.org):bouwe_ceunen/crypto-gathering-backend.git
        - name: revision
          value: master
        - name: image-name-backend
          value: docker-registry.k8s.bouweceunen.com/crypto-gathering/crypto-gathering-backend
      templates:
      - name: cicd
        steps:
          - - name: checkout
              template: checkout
          - - name: build-push-docker
              template: build-push-docker
              arguments:
                artifacts:
                - name: git-repo
                  from: "{{steps.checkout.outputs.artifacts.source}}"
                parameters:
                - name: image-tag
                  value: "{{steps.checkout.outputs.parameters.tag}}"
          - - name: run-tests
              template: run-tests
              arguments:
                artifacts:
                - name: git-repo
                  from: "{{steps.checkout.outputs.artifacts.source}}"
                parameters:
                - name: image-tag
                  value: "{{steps.checkout.outputs.parameters.tag}}"
          - - name: deploy-kubernetes
              template: deploy-kubernetes
              arguments:
                artifacts:
                - name: git-repo
                  from: "{{steps.checkout.outputs.artifacts.source}}"
                parameters:
                - name: image-tag
                  value: "{{steps.checkout.outputs.parameters.tag}}"

### Artifact Library

Argo uses something called an ‘artifact library’ to store state in between executing steps. It’s possible that the second step can use something the first step has built. All steps run in their own Docker container on Kubernetes, so state is transferred through an artifact library. There are a few options to choose which library to use. One of those is AWS S3, but it is also possible to use for example Google Cloud Storage. Everything is well documented on their [Github](https://github.com/argoproj/argo/blob/master/ARTIFACT_REPO.md) page. After running a few CI/CD pipelines, I began to think about how this was affecting my S3 storage. If you think about it, you only need it for a short amount of time. After your CI/CD pipeline is finished, artifacts are not needed anymore. I wrote small [cronjobs](https://github.com/BouweCeunen/argo-continuous-integration/tree/master/kubernetes/cronjobs) which will delete my S3 bucket from time to time, and clean up workflows so they won’t show in the Argo UI. Following ConfigMap shows the configuration of the Argo workflow controller.

    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: workflow-controller-configmap
      namespace: argo
    data:
      config: |
        artifactRepository:
          s3:
            bucket: k8s-argo-artifacts-bucket.bouweceunen.com
            endpoint: s3.amazonaws.com
        executor:
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
            limits:
              cpu: 400m
              memory: 800Mi

### Git Checkout

First things first, checking out the git repository. This is needed to build your Docker containers and deploy using your [Ansible](https://github.com/ansible/ansible) scripts inside your repository. In order for Argo to get your repository from your private Bitbucket account for example, it needs to have credentials. These can be provided by defining the ‘*git*’ annotation from within the template. The ‘*sshPrivateKeySecret*’ annotation gets the id_rsa key, which you use to access private repositories. In this case, from a Kubernetes Secret called ‘bitbucket-creds’. In this secret your id_rsa is stored just as you would store any other value in Kubernetes Secrets. It then outputs the repository as an **artifact **and exposes git commit which will be used as an image tag later on.

    - name: checkout
        inputs:
          artifacts:
          - name: git-repo
            path: /src
            git:
              repo: "{{workflow.parameters.repo}}"
              revision: "{{workflow.parameters.revision}}"
              sshPrivateKeySecret:
                name: bitbucket-creds
                key: id_rsa
        metadata:
          labels:
            app: argo
        container:
          image: alpine/git
          resources: 
            requests:
              cpu: 100m
              memory: 100Mi
            limits: 
              cpu: 400m
              memory: 800Mi
          command: [sh, -c]
          args: ["cd /src && git rev-parse --short HEAD > /tmp/git-commit"]
        outputs:
          artifacts:
          - name: source
            path: /src
          parameters:
          - name: tag
            valueFrom:
              path: /tmp/git-commit

### Building Docker

The next step is building your Docker containers so that they can be used to run your tests. You can define ‘**sidecars**’ in your Argo workflow, which will run a Docker daemon so that you can build Docker containers in your Docker containers. This is also referred to as docker-in-docker or **dind**. In order to be able to push containers to your private Docker registry, credentials need to be set. This is done by mounting a file in your root folder. The .dockerconfigjson is made when you create a Docker Secret in Kubernetes by executing *‘kubectl create secret docker-registry credentials*’. This volume is mounted in the root of your workflow under the annotation ‘*spec*’. Building and pushing your Docker containers from within an Argo workflow becomes very easy by defining it in a template, like this one below.

    - name: build-push-docker
        inputs:
          artifacts:
          - name: git-repo
            path: /src
          parameters:
          - name: image-tag
        metadata:
          labels:
            app: argo
        container:
          image: docker:18.09
          resources: 
            requests:
              cpu: 100m
              memory: 100Mi
            limits: 
              cpu: 400m
              memory: 800Mi
          workingDir: /src
          command: [sh, -c]
          args: ["until docker ps; do sleep 1; done; cd /src \ 
            && docker build . -t {{workflow.parameters.image-name-backend}}:{{inputs.parameters.image-tag}} && docker push {{workflow.parameters.image-name-backend}}:{{inputs.parameters.image-tag}}"]
          env:
          - name: DOCKER_HOST
            value: 127.0.0.1
          volumeMounts:
          - name: docker-config
            mountPath: /root/.docker/config.json
            subPath: .dockerconfigjson
        sidecars:
        - name: docker-in-docker
          image: docker:18.09-dind
          resources: 
            requests:
              cpu: 100m
              memory: 100Mi
            limits: 
              cpu: 400m
              memory: 800Mi
          securityContext:
            privileged: true
          mirrorVolumeMounts: true

### Continuous Integration

The CI part is much of the same, it executes Ansible which will deploy the application onto the Kubernetes cluster on which tests can be run. This enables fully integrated testing on real deployments. In the Ansible configuration a ‘kubectl exec’ with ‘yarn test’ is done to execute the tests in the corresponding pod. If this succeeds, it will clean up after itself and the workflow continues. The reason I ran tests with Ansible and not in the workflow itself is due to the fact that the workflow should have as little information as possible about actual deployment details, such as namespaces, cluster names, etc. With Ansible you can automatically set the environment right and Argo does not need to know any details about where the deployment is done or which tests it should exactly execute.

    - name: run-tests
        inputs:
          artifacts:
          - name: git-repo
            path: /src
          parameters:
          - name: image-tag
        metadata:
          labels:
            app: argo
        container:
          image: bouwe/ansible-kubectl-credstash:0.0.1
          workingDir: /src
          command: [sh, -c]
          args: ["cd /src/ansible \
              && ansible-playbook run-backend-test-on-k8s.yml -i environments/backend-test/backend-k8s -e docker_image_tag={{inputs.parameters.image-tag}}"]

### Continuous Delivery

The CD part will deploy your application to Kubernetes. This is done by running a template that executes [Ansible](https://github.com/ansible/ansible). As you can see, this template has some inputs, such as the git repository which is being handed down from the *cicd* template above. It also has the image tag so Ansible knows which Docker container to deploy onto Kubernetes. I have built a custom container that includes Ansible, Kubectl and [credstash](https://github.com/fugue/credstash) to be able to deploy and template variables into the defined yamls using credstash.

    - name: deploy-kubernetes
        inputs:
          artifacts:
          - name: git-repo
            path: /src
          parameters:
          - name: image-tag
        metadata:
          labels:
            app: argo
        container:
          image: bouwe/ansible-kubectl-credstash:0.0.1
          resources: 
            requests:
              cpu: 100m
              memory: 100Mi
            limits: 
              cpu: 400m
              memory: 800Mi
          workingDir: /src
          command: [sh, -c]
          args: ["cd /src/ansible \
            && ansible-playbook deploy-backend-to-k8s.yml -i environments/backend/backend-k8s -e docker_image_tag={{inputs.parameters.image-tag}}"]

It is also important to change the ClusterRole of Argo so that it can deploy in all namespaces on your Kubernetes cluster. The ClusterRole should be defined as follows, so it can deploy all CRD’s such as Deployments, Services etc.

    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: argo-cluster-role
    rules:
    - apiGroups: ["*"]
      resources:
      - pods
      - pods/exec
      - secrets
      - ingresses
      - services
      - jobs
      - deployments
      - statefulsets
      - cronjobs
      - workflows
      - configmaps
      verbs: ["*"]

### UI Dashboard
[**argoproj/argo-ui**
*Web-based UI for Argo workflow engine. Contribute to argoproj/argo-ui development by creating an account on GitHub.*github.com](https://github.com/argoproj/argo-ui)

As with all emerging tools out there, everything has to start somewhere and this is also the case with the Argo dashboard. It is minimal but does what it needs to do. Argo shows all the workflows and their steps, it updates automatically and all progress and logs can be viewed from here. This makes it very easy to monitor how everything is going. Don’t expect too much from it now, I hope they will continue to improve it, there’s much potential here.

![](https://cdn-images-1.medium.com/max/7680/1*k926VrJrVOwkI6rYNn3JEg.png)

### TL;DR

Argo is very easy to understand, it integrates with Kubernetes and uses your Kubernetes cluster to do CI/CD. It is much more clean and lean than let’s say Spinnaker, Istio, etc. Easy to set up and does what it needs to do. Argo is the main project which defines its own CRD, which is the ‘Workflow’. Argo CI is not actively developed anymore, but I created my own [implementation](https://github.com/BouweCeunen/argo-continuous-integration). Argo CD is the GitOps way of managing deployments. Together with my own implementation of Argo CI and with Argo workflows, its possible to set up a CI/CD pipeline that runs in your Kubernetes cluster.
