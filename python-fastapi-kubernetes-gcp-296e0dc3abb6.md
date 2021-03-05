
# Deploy a Python API With Docker and Kubernetes

Autoscale compute-intensive workloads to keep up with changing demand

![Image source: Author](https://cdn-images-1.medium.com/max/3300/1*fRIL0ju9I7iIN3gYOzn44A.png)*Image source: Author*

Kubernetes lets you deploy multi-container applications across a cluster — either your own machines or in the cloud. An API is the gateway to your application, the interface that users (and even other services) can use to interact with it. Building an API needn’t be hard work, however. With [FastAPI](https://fastapi.tiangolo.com/) you can have a working system in minutes, and it's simple to debug and test at each stage as you package it for deployment.

In this article we cover:

1. Building an [OpenAPI](https://www.openapis.org/)-compatible interface in Python using FastAPI

1. Packaging the API inside a [Docker](https://www.docker.com/) container

1. Deploying to a local [Kubernetes](https://kubernetes.io/) cluster

1. Load testing using [Locust](https://locust.io/)

1. Deploying to the cloud with [Google Cloud Kubernetes Engine](https://cloud.google.com/kubernetes-engine) (GKE)

## Quickstart

If you just want to get up and running quickly, clone the companion repository and follow the instructions to deploy to a local Kubernetes cluster or Google cloud:
[**4OH4/kubernetes-fastapi**
*Barebones Python FastAPI in Kubernetes Built on fastapi-aws-lambda-example (MIT License): To run (in isolation)…*github.com](https://github.com/4OH4/kubernetes-fastapi)

If you need a local cluster for development, follow the [setup instructions for minikube](https://minikube.sigs.k8s.io/docs/start/).

## Introduction

The API is the interface to your code, whether that is a slick new AI model that recommends new content or a long-running data processing service that crawls the web for answers. The API makes your code accessible so that you can share it with the world. Deploying an API in Python is easy if you use a framework that takes care of the boilerplate code.

In this article, we package our service up inside a Docker container. This creates a handy package that incorporates all of the required dependencies for running our code. It is also a convenient building block that we can use for horizontal scaling: If demand increases, deploy more copies of the container image. We use Kubernetes to manage the orchestration — autoscaling the deployment as demand shifts.

Containers aren’t the only option, however: Function-as-a-service (FaaS) is a popular route that is particularly suitable for serving an API from a cloud service. In fact, this article was inspired by another piece on deploying an API to Amazon Web Services (AWS) Lambda functions:
[**How to continuously deploy a fastAPI to AWS Lambda with AWS SAM**
*My last article about fastAPI was supposed to be an article about how to deploy a fastAPI on a budget, but instead…*iwpnd.pw](https://iwpnd.pw/articles/2020-01/deploy-fastapi-to-aws-lambda)

Function-as-a-service is part of the serverless paradigm — it lets you focus on writing the code that implements your requirements, and the service provider works out how to deploy and scale. It is a layer of abstraction above the use of containers. If your code is pretty standard (e.g., pure Python) and is expected to complete execution in seconds (or up to a few minutes) then it is worth considering.

The example I use for this article would work quite nicely within the FaaS model, but I chose not to go down that route for a couple of reasons. When building the system that was the basis for this article, we wanted to be able to move easily between running on in-house or cloud systems. It was also advantageous to not be tied to any particular cloud provider; I use the Google Kubernetes Engine below, but the API code could work equally well on Amazon Elastic Kubernetes Service instead.

## Build the API

We use the Python package FastAPI as the framework for building the API. Key features:

* It's asynchronous, so can deal with multiple concurrent requests;

* It's OpenAPI compatible, so it is easy to document and interact with the API — such as using the included UI (see screenshot below).

* It's based on [pydantic](https://pydantic-docs.helpmanual.io/), so you can specify the input and output data structures using standard Python data types — no additional syntax or markup language.

![Screenshot of the UI (SwaggerUI) that documents the API endpoints, and even lets you test drive each one.](https://cdn-images-1.medium.com/max/2052/1*ALL0tEgh6QrFyWb-DErnKg.png)*Screenshot of the UI (SwaggerUI) that documents the API endpoints, and even lets you test drive each one.*

The code base contains the following main files:

    kubernetes-fastapi/
    ├── Dockerfile
    ├── api.yaml
    ├── autoscale.yaml
    ├── locustfile.py
    ├── requirements.txt
    ├── Dockerfile
    └── service/
        ├── main.py
        └── api/
            ├── api_v1/
            │  ├── api.py
            │  └── endpoints/
            │      └── hello.py
            └── core/
                ├── config.py
                ├── logic/
                │   └── business_logic.py
                └── models/
                    └── input.py
                    └── output.py

To customise the API for your purposes:

Update the input and output data structures in models/. The input.py and output.py files contain classes that define the fields and data types of the API input and response. Below is output.py — each field has a variable name, type (int, float, str, etc.) and a description.

<iframe src="https://medium.com/media/f150d3913dbcbfff5c6cddc90a37c821" frameborder=0></iframe>

The processing behind each endpoint is defined in the endpoints/ folder — this is hello.py. It receives the MessageInput type and returns a MessageOutput:

<iframe src="https://medium.com/media/bd201a59b96006e75994c8001dff7122" frameborder=0></iframe>

As an example, we run a calculation to find the largest prime factor of a random number. The results and metadata are returned to the caller, along with a formatted string message.

The calculation requires typically 1-100ms to complete and is solved by iteration. While not a huge amount of time when communicating with a remote server, it is a compute-intensive task and so the process is CPU-bound (as opposed to IO-bound). FastAPI uses async operations to handle multiple requests in parallel — this usually scales well as operations are often IO-bound, meaning time is spent waiting for other operations to complete. While waiting, control is handed over to another thread so that it can do its work. With a CPU-bound operation, this cannot happen so throughput is a lot lower (see the section below on Load Testing).

Other files:

* [main.py](https://github.com/4OH4/kubernetes-fastapi/blob/main/service/main.py) initialises the API, and we also define a ping endpoint for testing purposes.

* Dockerfile defines the content of the container image and loads any dependencies (defined in requirements.txt).

* api.yml describes the Kubernetes deployment.

* autoscale.yml describes a Kubernetes HorizontalPodAutoscaler that tells Kubernetes how to scale the application deployment based on CPU usage.

* locustfile.py describes a configuration for Locust to conduct load testing (see below).

In the stages below, we upload our API to the public cloud and make the endpoints publicly accessible. The example API is relatively low risk from a cybersecurity perspective, as it does not accept user input and contains no sensitive information. That said, there is nothing to stop a third party from accessing it or flooding it with requests. While beyond the scope of this article, if you are building a custom service for use in production, you should consider [input validation](https://fastapi.tiangolo.com/tutorial/query-params-str-validations/) and [authentication](https://fastapi.tiangolo.com/tutorial/security/).

## Local Test

Prior to deploying your API, it's a good idea to do some debug testing using a local Python environment. First, install the dependencies using the requirements.txt file in the repository:

    $ pip install -r requirements.txt

Then start a local webserver using uvicorn:

    $ uvicorn service.main:app --host 0.0.0.0 --port 8080 --reload

The console should display a number of messages as the application starts up. Then navigate to [http://localhost:8080/docs](http://localhost:8080/docs) to explore the API using the Swagger UI interface. You can exercise both the /hello and /ping endpoints using the Try It Out button.

The /hello endpoint responds with a greeting and the result of calculating the largest prime factor of a random integer. You should see a response body similar to the following:

    {
      "message1": "Hello, world!",
      "message2": "The largest prime factor of 1462370954730 is 398311. Calculation took 0.006 seconds.",
      "n": 1462370954730,
      "largest_prime_factor": 398311,
      "elapsed_time": 0.0057561397552490234
    }

The --reload argument tells uvicorn to monitor the source code and reload the API if changes are made — so you can work on and update your code and immediately test the results in the browser. Press Ctrl-C to exit.

You can also call the API directly from the command line if you have cURL installed:

    $ curl -H ‘Content-Type: application/json’ -d {} localhost:8080/api/v1/hello

    {"message1":"Hello, world!","message2":"The largest prime factor of 1927651733935 is 991080583. Calculation took 0.011 seconds.","n":19276517 ....

## Run in a Local Docker Container

Next step is to package the API inside a Docker container, test locally, and then push to [Docker Hub](https://hub.docker.com/). In the examples here, I build and push using my Docker Hub ID, 4oh4. You will need to replace that with your own ID. Alternatively, you can use the original images (unmodified) by skipping this build stage. To build the Docker image:

    $ docker build -t 4oh4/kubernetes-fastapi:1.0.0 .

To run the container:

    $ docker run -p 8080:8080 --name kubernetes-fastapi 4oh4/kubernetes-fastapi:1.0.0

If you skipped the build command, the image will now be pulled from Docker Hub instead. As before, navigate to [http://localhost:8080/docs](http://localhost:8080/docs) to explore the API using the Swagger UI interface. Press Ctrl-C to exit.

If you modified the API and built your own container, push the image to Docker Hub (remembering to change 4oh4 to your ID):

    $ docker push 4oh4/kubernetes-fastapi:1.0.0

By default, a new image is created as private. Rather than dealing with authentication, navigate to [Docker Hub](https://hub.docker.com/), log in, find your new image, and make it publicly accessible.

## Load Test With Locust

We can use the Python package Locust to simulate a large number of concurrent requests to the API. The GitHub repository contains a test configuration in locustfile.py that tells locust how to communicate with the API. Install the package and run it from the command line:

    $ pip install locust
    $ locust

    [2021-01-24 16:21:29,188] INFO/locust.main: Starting web interface at [http://0.0.0.0:8089](http://0.0.0.0:8089) (accepting connections from all network interfaces)

Navigate to [http://localhost:8089](http://localhost:8089) to see the web interface, and initialise testing — customise the host name if not running locally:

![](https://cdn-images-1.medium.com/max/2000/1*HJg1AI1sHLJUHASGOziWHg.png)

Hit Start and after a few seconds, Locust will start hitting the API with traffic from a pool of simulated users. Select the Charts tab to visualise the performance metrics:

![Performance results during simulated load testing using Locust. The API was deployed on a Kubernetes cluster, with a variety of autoscaling and configuration settings, as the number of concurrent users was increased up to 250.](https://cdn-images-1.medium.com/max/2052/1*vxEqxvDHSY6sVbuUiIVq1w.png)*Performance results during simulated load testing using Locust. The API was deployed on a Kubernetes cluster, with a variety of autoscaling and configuration settings, as the number of concurrent users was increased up to 250.*

**Exercise:** Try disabling the compute-intensive calculation and see the effect on response time and throughput (comment out the function call in endpoints/hello.py and return fixed values instead).

## Deploy to a Kubernetes Cluster

Now that our API is working from inside a Docker container, we are going to deploy our application to a Kubernetes cluster. Kubernetes will manage the orchestration of multiple containers so that the provision of computational resources matches the demand. We will use an autoscaling policy based on CPU usage so that more containers will be deployed to service API requests if the loading increases.

Access to a Kubernetes cluster is required for this section. Most of us do not have computing clusters at home, but it is possible to set up a single-node cluster running on your local machine using minikube or similar. [Installation instructions](https://minikube.sigs.k8s.io/docs/start/) can be in the documentation. Alternatively, a managed Kubernetes cluster from a cloud provider will also work — see below for detailed instructions on deployment to Google Kubernetes Engine. Whichever route you choose, we will use the kubectl tool to set up our cluster.

The core of our deployment is described in api.yaml. This configuration file contains:

* a Kubernetes deployment — This describes the Docker image to use, resources required, and ports to expose.

* a Kubernetes service — This describes how to present the deployment to the outside world, in this case via a load balancer that splits traffic between (potentially) multiple containers.

Apply the configuration using kubectl, check that the pod is up and running, and then list the services to find the external IP address:

    $ kubectl apply -f api.yaml

    service/kf-api-svc configured
    deployment.apps/kf-api configured

    $ kubectl get pods
    NAME                      READY   STATUS    RESTARTS   AGE
    kf-api-65d656b8c9-ppn7j   1/1     Running   0          4m

    $ kubectl get svc kf-api-svc
    NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
    kf-api-svc   LoadBalancer   10.105.177.140   <pending>     8080:32578/TCP   5m

As I’m usingminikube, the external IP hasn’t been allocated — use the tunnel command to get it connected:

    minikube tunnel

From another terminal (as the previous command is still running in the other one), list the services again:

    $ kubectl get svc kf-api-svc
    NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
    kf-api-svc   LoadBalancer   10.105.177.140   10.105.177.140     8080:32578/TCP  6m

The external IP is now exposed (it's actually the same as the cluster IP). You can now navigate to that address, port 8080 in your web browser, and should see the familiar Swagger UI. In my case it was [http://10.105.177.140:8080/docs,](http://10.105.177.140:8080/docs,) although you will need to replace the IP address with whichever address is allocated by your system.

Currently, we have a single replica of our Docker container running on the cluster, which is all well and good, but the real power of clusters comes from running at scale. Of course, you won’t see the benefit of scaling if you are only running a single-node cluster on minikube. Apply the HorizontalPodAutoscaler policy and check the results of scaling:

    $ kubectl apply -f autoscale.yaml
    horizontalpodautoscaler.autoscaling/kf-api-hpa configured

    $ kubectl get hpa
    NAME        REFERENCE          TARGETS   MINPODS  MAXPODS  REPLICAS   AGE
    kf-api-hpa  Deployment/kf-api  42%/50%   1        10       1          10m

The key lines in this configuration file tell Kubernetes to scale the deployment with between 1 and 10 replicas, by trying to keep the CPU utilisation at 50%:

      minReplicas: 1 
      maxReplicas: 10 
      targetCPUUtilizationPercentage: 50

If you like you can repeat the load testing with Locust now (just replace localhost with the cluster external IP).

Once you have finished testing, take down the deployment, service, and autoscaler:

    kubectl delete deployment kf-api
    kubectl delete svc kf-api-svc
    kubectl delete hpa kf-api-hpa

## Deploy to Google Cloud

Finally, we are going to deploy to a cloud-based Kubernetes cluster. The deployment process is actually very similar for local and cloud deployments — we just need to configure kubectl to talk to the cluster. In this section, we are going to provision a three-node cluster on Google Cloud and deploy our API to it. For further information, refer to the [documentation](https://cloud.google.com/kubernetes-engine/docs/quickstart):

### Requirements

* a Google Cloud account, either with billing enabled or sufficient free credits remaining. If you don’t leave the cluster running for very long, this should incur charges of about $1–$2.

* an active project on GCP — Create one via the [console](https://console.cloud.google.com/).

* Google Cloud SDK installed — [Download here](https://cloud.google.com/sdk/docs/install).

First off, make sure the GCP Kubernetes Engine is enabled for your project, either at [this page](https://console.cloud.google.com/apis/library/container.googleapis.com) or by using the following command:

    gcloud services enable container.googleapis.com

Then install the gcloud kubectl component and configure gcloud with your project ID and default region:

    gcloud components install kubectl
    
    gcloud config set project my-project-id
    gcloud config set compute/zone europe-west2-a

Replace my-project-id with the ID of your project — not the name. You can find the ID in the Project Info box on the dashboard page. I’m near London, UK, so have chosen europe-west2-a as my preferred region. It matters little which region you choose, although specifying it here saves entering it later.

Next, create a three-node Kubernetes cluster on GCP, and configure kubectl to talk to it by getting the required credentials:

    $ gcloud container clusters create my-cluster-name --num-nodes=3

    Creating cluster my-cluser-name in europe-west2-a... Cluster is being health-checked (master is healthy)...done.
    ...

    $ gcloud container clusters get-credentials my-cluster-name

    Fetching cluster endpoint and auth data.
    kubeconfig entry generated for my-cluser-name.

Here my-cluster-name can be anything of your choosing. Finally, deploy the application and service, apply the autoscaling policy, and check the results:

    $ kubectl apply -f api.yaml
    service/kf-api-svc created
    deployment.apps/kf-api created

    $ kubectl apply -f autoscale.yaml
    horizontalpodautoscaler.autoscaling/kf-api-hpa created

    $ kubectl get svc kf-api-svc
    NAME         TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)          AGE
    kf-api-svc   LoadBalancer   10.103.246.129   34.89.100.187   8080:31382/TCP   2m1s

    $ kubectl get hpa
    NAME         REFERENCE           TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
    kf-api-hpa   Deployment/kf-api   2%/50%    1         10        1          2m20s

As everything looks good, so carry on and apply some loading using Locust, using the cluster external IP (port 8080) as the target. You should see the CPU utilisation increase and the number of replicas increase in response:

    $ kubectl get hpa
    NAME     REFERENCE           TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
    kf-api   Deployment/kf-api   94%/50%   1         10        8          5m54s

When you are finished, remember to take down your deployment and delete the cluster:

    $ kubectl delete deployment kf-api
    $ kubectl delete svc kf-api-svc
    $ kubectl delete hpa kf-api-hpa

    $ gcloud container clusters delete my-clutser-name

It is worth checking in the Google Cloud console that all resources have been stopped — otherwise you might get an unexpected bill. The safe option is to delete the project entirely, from Project Settings.

## Summary

In this article, we built a simple API in Python and tested it locally (both in native Python and via Docker) before deploying to a Kubernetes cluster (both locally and in the cloud). Hopefully, you have seen how tools like kubectl and gcloud blur the line between working on a local development machine and cluster/cloud resources, making it easy to move between the two. I look forward to reading your comments and questions in the Responses* *section.

## References

* [How to continuously deploy a fastAPI to AWS Lambda with AWS SAM](https://iwpnd.pw/articles/2020-01/deploy-fastapi-to-aws-lambda)

* [FastAPI](https://fastapi.tiangolo.com/)

* [Google Cloud Kubernetes Engine API](https://console.cloud.google.com/apis/library/container.googleapis.com)
