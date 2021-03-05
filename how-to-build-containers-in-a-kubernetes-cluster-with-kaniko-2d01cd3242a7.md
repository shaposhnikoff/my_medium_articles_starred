
# How to Build Containers in a Kubernetes Cluster with Kaniko

Automate container builds within K8s without a Docker daemon

![Source: [Wikipedia Commons](https://commons.wikimedia.org/wiki/File:Kubernetes-Engine-Logo.svg)](https://cdn-images-1.medium.com/max/2000/1*l85zI6ot5txddTGeDq7BiA.png)*Source: [Wikipedia Commons](https://commons.wikimedia.org/wiki/File:Kubernetes-Engine-Logo.svg)*

Traditionally, organisations have built Docker images outside the Kubernetes cluster. However, with more and more companies adopting Kubernetes and the demand for virtual machines decreasing day by day, it makes sense to run your continuous integration builds within the Kubernetes cluster.

Building Docker images within a container is a security challenge because the container needs access to the worker node file system to connect with the Docker daemon.

You also need to run your container in privileged mode. That practice isn’t recommended as it opens up your nodes to numerous security threats. Most organisations rely on persistent external volumes for storing data, and in no case should a container have direct access to the node filesystem.

Running a container in privileged mode is a terrible idea as it provides the container root access to the host. That gives cybercriminals opportunities to compromise your system, potentially jeopardising an entire worker node instead of just the container.

Google solves this problem by providing a tool called [Kaniko](https://github.com/GoogleContainerTools/kaniko). Kaniko helps you build container images within a container without any access to the Docker daemon. That way, you can execute your build jobs within containers without granting any access to the host filesystem.

You just need to create a build manifest as a Kubernetes batch job and apply it to the cluster using any CI tool of your choice. The job takes responsibility for building your image and uploading it to the specified container registry.

## How Kaniko works

Kaniko:

* Reads the specified Dockerfile.

* Extracts the base image (specified in the FROM directive) into the container filesystem.

* Runs each command in the Dockerfile individually.

* Takes a snapshot of the userspace filesystem after every run.

* Appends the snapshot layer to the base layer on each run.

Because of this, Kaniko does not depend on a Docker daemon.

Let’s take a look to see how it works.

## Prerequisites

Ensure you have the following:

* A running Kubernetes cluster with permissions to create, list, update and delete jobs, services, pods, and secrets.

* A GitHub account for storing the Dockerfile and Kubernetes manifests.

* A Docker Hub account for hosting container images.

## Create the Container Registry Secret

Let’s start by setting up a container registry secret. It’s necessary to authenticate with the container registry to push the built image.

You will need the following:

* docker-server — The Docker registry server where you need to host your images. If you are using Docker Hub use [https://index.docker.io/v1/](https://index.docker.io/v1/).

* docker-username — The Docker registry username.

* docker-password — The Docker registry password.

* docker-email — The email configured on the Docker registry.

Run the following command, substituting the above values:

    $ kubectl create secret docker-registry regcred --docker-server=<docker-server> --docker-username=<username> --docker-password=<password> --docker-email=<email>

## GitHub repository

For the hands-on exercise fork [this repository](https://github.com/bharatmicrosystems/kubernetes-kaniko) into your GitHub account. Clone the forked GitHub repository and cd into it.

As we're building an [NGINX](https://www.nginx.com/) container, let’s start with the nginx:v3 tag. Run the following commands to replace the placeholders with values for your environment:

    $ export GHUSER="<YOUR_GITHUB_USER>"
    $ export GHREPO="<YOUR_GITHUB_REPO>"
    $ export DOCKERREPO="<YOUR_DOCKER_REPOSITORY>"

    #Substitute placeholders in build.yaml
    $ sed -i "s/GHUSER/${GHUSER}/g" build.yaml
    $ sed -i "s/GHREPO/${GHREPO}/g" build.yaml
    $ sed -i "s/<repo>/${DOCKERREPO}/g" build.yaml
    $ sed -i "s/<tag>/nginx:v3/g" build.yaml

    #Substitute placeholders in Dockerfile
    $ sed -i "s/v_x/3/g" Dockerfile

    #Substitute placeholders in the Kubernetes deployment file
    $ sed -i "s/<repo>/${DOCKERREPO}/g" workloads/nginx-deployment.yaml
    $ sed -i "s/<tag>/nginx:v3/g" workloads/nginx-deployment.yaml

Let’s take a look at the Dockerfile after the substitution:

<iframe src="https://medium.com/media/0e22837eb646dee7a1ffc5e85d2b19b1" frameborder=0></iframe>

The Dockerfile contains two steps. It declares the base image to nginx and writes This is version 3 to /usr/share/nginx/html/index.html. We should get that as a response when we hit the NGINX endpoint.

Let’s see what the build.yaml looks like after substitution:

<iframe src="https://medium.com/media/1371bef695e3dc1b993080d018af8eac" frameborder=0></iframe>

As you may notice, it’s a straightforward Kubernetes job with a lone container. The manifest creates a container using the gcr.io/kaniko-project/executor:latest image and runs it with the following arguments:

* docker-file — The path of the Docker file, relative to the context.

* context — The Docker context. In this case, we’ve indicated our GitHub repository

* destination — The Docker repository to push the built image.

Additionally, it also mounts a docker config JSON file on /kaniko/.docker to authenticate with the Docker repository. We defined this in the previous section.

What about the nginx-deployment.yaml file?

<iframe src="https://medium.com/media/2dc1d64bf9d24b2a0c3fb5ffafb1d313" frameborder=0></iframe>

This is a classic NGINX deployment, which deploys the nginx:v3 image that we build using Kaniko.

If you notice the annotations, it has fluxcd.io/automated: "true". This means that if we use [Flux CD](https://fluxcd.io) to deploy this manifest, Flux CD auto-updates it with the new version of NGINX image when it becomes available in the Docker repository.

Additionally, we have an NGINX service that exposes the NGINX pods to an external load balancer.

<iframe src="https://medium.com/media/5dba55be999fddb556e4610b943b241e" frameborder=0></iframe>

Push the changes to your remote GitHub repository.

## Build the Container Image Using Kaniko

Build the image by applying the build.yaml manifest:

    $ kubectl apply -f build.yaml
    job/kaniko created
    $ kubectl get pod
    NAME           READY   STATUS    RESTARTS   AGE
    kaniko-jztr6   1/1     Running   0          68s

Once the pod is running, print the logs:

<iframe src="https://medium.com/media/ba638b962c4c3b21107852014c8c6b5f" frameborder=0></iframe>

As you can see, Kaniko has built nginx:v3 and it should appear in your Docker registry.

## Install Flux CD

As mentioned in the previous section, we use Flux CD to deploy workloads to our Kubernetes cluster. [Flux CD](https://fluxcd.io/) is a continuous delivery tool that’s quickly gaining popularity. Weaveworks initially developed the project, before open-sourcing it to the Cloud Native Computing Foundation.

Flux CD synchronises the Kubernetes manifests stored in the Source Code Repository with the Kubernetes Cluster through periodically polling the repository.

Additionally, Flux CD also allows you to poll Container Registries and update the Kubernetes Manifests on your Git Repository with the latest images if you want to automate upgrades to your workloads.

Follow the instructions in my article [How to Continuously Deliver Kubernetes Applications With Flux CD](https://medium.com/better-programming/how-to-continuously-deliver-kubernetes-applications-with-flux-cd-502e4fb8ccfe), to install and configure Flux CD in your Kubernetes cluster. Please make sure you do not change the GHUSER and GHREPO variables when following that guide. The objective is to sync your GitHub repository with Flux CD.

Once you’ve set up Flux CD on your Kubernetes cluster, you can sync it with your GitHub repository by running the following:

    $ fluxctl sync --k8s-fwd-ns flux
    Synchronizing with ssh://[git@github.com](mailto:git@github.com)/bharatmicrosystems/kubernetes-kaniko-nginx
    Revision of master to apply is 3a33f0a
    Waiting for 3a33f0a to be applied ...
    Done.

Now let’s list all the resources within the web namespace to see whether Flux has deployed NGINX.

<iframe src="https://medium.com/media/a9ea76e1e5d4d66c3af172cf464cf8ac" frameborder=0></iframe>

Obtain the External IP from the nginx-service and trigger the endpoint:

    $ curl 35.193.173.129
    This is version 3

## Update the NGINX Version

Let’s try something different. As I described in the previous section, Flux CD automatically detects new versions of NGINX and updates the nginx-deployment.yaml manifest file with the latest version. What would happen if we build a new version of NGINX and push it to the container registry? Let’s find out.

### Update the manifests

We need to update the docker file and the build manifest to reflect the new version.

    # Update docker file with the new version
    $ sed -i 's/3/4/g' Dockerfile

    # Update the build.yaml with the new version
    $ sed -i 's/nginx:v3/nginx:v4/g' build.yaml

Let’s apply the build manifest again:

    $ kubectl apply -f build.yaml
    job/kaniko created
    $ kubectl get pod
    NAME           READY   STATUS    RESTARTS   AGE
    kaniko-qs9qz   1/1     Running   0          68s

Now list the logs once the pod is running:

<iframe src="https://medium.com/media/c1b17b38126000a7b91a8f62b4d46a67" frameborder=0></iframe>

Kaniko should build and push the image to the Docker repository:

![](https://cdn-images-1.medium.com/max/3782/1*msVo3_Cjhi0_vCSOKEYmlg.png)

## Flux CD Sync

Wait for five minutes for Flux to sync your GitHub repository with the cluster.

You should see that Flux CD has updated your GitHub Repository with the new nginx-deployment.yaml manifest:

![](https://cdn-images-1.medium.com/max/2498/1*0zj9MwaqU1Ma09_ba_p3bQ.png)

If you look at the nginx-deployment.yaml file, you’ll see that Kaniko has updated the image to nginx:v4:

<iframe src="https://medium.com/media/1203ae158630207773b655415485fc48" frameborder=0></iframe>

Let's list all resources from the web namespace:

<iframe src="https://medium.com/media/2830db9bb23fc44dd74ddf10d6c5feb3" frameborder=0></iframe>

We can see that the nginx-deployment pods are recent.

Curl the NGINX service again:

    $ curl 35.193.173.129
    This is version 4

Congratulations! You get version 4 now instead of version 3. That shows that Flux CD is automatically updating the configuration when it detects changes in the container registry.

## Conclusion

Thanks for reading! I hope you enjoyed the article.

Along with Flux CD, Kaniko can be a compelling CI/CD alternative to existing options and gives you an extra edge, especially if you are looking to harness the infinite scaling capacity of Kubernetes and the Cloud.
