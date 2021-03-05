
# Simplifying Microservices with Istio in Google Kubernetes Engine — Part I



Istio simplifies service to service communication, traffic ramping, fault tolerance, performance monitoring, tracing and much, much more. How can we leverage it to abstract out the infrastructure and crosscutting functions from our microservices?
> What I write about Istio is a subset of the [awesome documentation](https://istio.io/docs/) that is on the Istio site. Please read the official docs to understand more.
> **Note**: you can skip the Background section if you are familiar with microservices.

In Part I of this series, we’ll cover the following areas:

* **Background**: monolithic apps and introduction to Microservices

* The **Spring Cloud Netflix Stack** and its advantages

* Introducing **Istio**

* **Service-service communication example** with Istio

## **Background**:
> In the past, we had big, monolithic apps that “did it all”. This was a way to get our product to market and worked well initially as we just had to get our first application online, and we could always come back to improve it. It was easier to deploy one big app than to build and deploy multiple smaller pieces.
> However, such application development led to ‘Big Bang’ efforts (where we would deploy the whole application at once after months of work) and incremental changes were staggered out over long periods of time due to the complex nature of the build/test/deploy/release cycles. It wasn’t exactly much bang for the buck though, if you were a Product Owner, especially if a serious bug was found after deploying a new version of our application. This could even lead to a rollback of the application. Deploying such large applications to the Cloud and scaling them wasn’t easy either as the entire app would need to be scaled vs smaller components.
> **Enter Microservices:**
> [Microservices](https://martinfowler.com/articles/microservices.html) are suites of independently deployable services running within their own processes. They often communicate using HTTP resources and each service is typically responsible for a single area within the application. In the popular e-Commerce catalog example, you can have an ItemsService, a ReviewService and a RatingsService each focusing on one area.
> This approach helped distributed teams contribute to various services without having to build/test/deploy the whole application for every service change and also not have to step all over each other’s code. Deploying services to the Cloud is also easier as individual services could be auto-scaled based on need.
> Polyglot services (services written in different languages) were also made possible with this approach as we could have a Java/C++ service do more processing-intensive work and a Rails/Node.js service could be used to support front end applications and so on.

## Spring Cloud Netflix:

With the popularity of Microservices, there was a proliferation of frameworks that simplified service creation and management. My personal favorite in 2015 was the Netflix OSS stack ([Spring Cloud Netflix](https://cloud.spring.io/spring-cloud-netflix/)) that introduced me to a simple way of creating Microservices **in Java** from within my [Spring Tool Suite](https://spring.io/tools/sts/all) IDE.

I could get the following features with the Netflix suite (Figure 1 below):

* Service Registry with **Eureka** — for registration and discovery of services

* Client side load balancing with **Ribbon** — clients can choose what server to send their requests to.

* Declarative REST client **Feign** to talk to other services. Internally, it uses Ribbon.

* API Gateway with **Zuul** — single entry point to manage all API calls, routing rules etc. to our microservices

* Circuit Breakers with **Hystrix** — handling fault tolerance along with the ability to turn off a communication channel for a short period of time (breaking the circuit) and return a user friendly response if the destination service is down

* Dashboards with Hystrix and **Turbine** — visualizing traffic and circuit breakers

![Figure 1: Spring Cloud Netflix implementation for Microservices](https://cdn-images-1.medium.com/max/2600/1*0CPG9sbX_Eer9DWyNbOEKw.jpeg)*Figure 1: Spring Cloud Netflix implementation for Microservices*

However, this method of building and deploying applications comes with its own challenges.

## Challenges:
> ***Deployment**: How do we deploy our services to the Cloud in a consistent manner and ensure that they are always available and have them auto-scale to our needs?*
> ***Cross-cutting concerns**: How do we get all the features that we had seen in the Spring Cloud Netflix implementation above *plus more* **with little to no code changes to each microservice**? **Also,** **how do we handle services written in different languages?***

## Solutions:
> ***Deployment**: [Kubernetes](https://kubernetes.io/) has paved the way to efficiently deploy and orchestrate Docker containers in Google Kubernetes Engine (GKE). Kubernetes abstracts infrastructure and allows us to interact with it through APIs. Please see the links at the end of this article for more details.*
> ***Cross-cutting concerns: **We can use [**Istio](https://istio.io/docs/concepts/what-is-istio/overview.html)**. The official explanation on the Istio website says *“**Istio provides an easy way to create a network of deployed services with load balancing, service-to-service authentication, monitoring, and more, without requiring any changes in service code. You add Istio support to services by deploying a special sidecar proxy throughout your environment that intercepts all network communication between microservices, configured and managed using Istio’s control plane functionality**.”

## Introducing Istio:

In other words, with Istio, we can create our microservices and deploy them along with a “**lightweight sidecar proxy**” (Figure 2 below) that will handle many of our cross cutting needs such as:

* Service-Service communication

* Tracing

* Circuit Breaking (Hystrix-like functionality) and retries

* Performance monitoring and dashboards (similar to the Hystrix and Turbine dashboards)

* Traffic routing (example: send x% traffic to v2 of our app), canary deployments

* An added bonus (especially if you are working with **sensitive data** such as PHI in Healthcare) — egress (calls made to external services outside the Istio Service mesh) need to be explicitly configured and can prevent services from making ad-hoc calls outside of the service mesh.

![Figure 2: Using the Envoy proxy to communicate between polyglot services](https://cdn-images-1.medium.com/max/2000/1*Ywx4FCF4923ZAZjX8a_kOQ.jpeg)*Figure 2: Using the Envoy proxy to communicate between polyglot services*

In Figure 2 above, we have eliminated many of the components that were present in Figure 1 and added a new component ([Envoy Proxy](https://www.envoyproxy.io/)). Each service (A) that needs to talk to another service (B) communicates with its proxy which is pre-configured with rules to route to other destination proxies. Proxies talk to proxies. Since ALL the communication happens through the proxies, it is easy to monitor the traffic, gather metrics, apply circuit breaker rules etc. as needed.
> Declaratively configuring rules and policies for cross cutting features, without having to make code changes to our services allows us to focus on what matters most: building high quality services that deliver business value.

At a high level, Istio has the following components:

* **Envoy**: a high performance, low footprint proxy that enables communication between the services and helps with load balancing, service discovery and more.

* **Mixer**: is responsible for the access control policies across all the services in the ecosystem (the Service Mesh) and gathers telemetry information sent by Envoy and other services

* **Pilot**: helps with service discovery, traffic ramping and fault tolerance (circuit breaking, retries etc.)

* **Istio-Auth**: is used for service-service authentication as well as end user authentication using mutual TLS. The example in this article does not use Istio-Auth

## Service-service communication with Istio:

### Let’s see it in practice!

We’ll take a simple example where we show 3 microservices communicating via the Envoy proxy. They have been created in **Node.js **but, as mentioned before, they could be in any language. [**Github](https://github.com/nmallya/istiodemo)**

![Figure 3: A logical view of the 3 microservices used to fetch Pet Details](https://cdn-images-1.medium.com/max/2000/1*jcZYzlqHvueD5lA-1BLcCA.jpeg)*Figure 3: A logical view of the 3 microservices used to fetch Pet Details*

1. **PetService**: returns a pet’s information and medical history by calling the PetDetailsService and the PetMedicalHistoryService. It will be running on port 9080.

1. **PetDetailsService**: returns pet information such as name, age, breed, owner etc. It will be running on port 9081.

1. **PetMedicalHistoryService**: returns a pet’s medical history (vaccinations). It will be running on port 9082.

### Steps:

Create a Kubernetes cluster in GKE (**nithinistiocluster** in my case). Ensure that the default service account has the following permission:

roles/container.admin (Kubernetes Engine Admin)

Install istio as described in [https://istio.io/docs/setup/kubernetes/quick-start.html](https://istio.io/docs/setup/kubernetes/quick-start.html)

1. Now, we are ready to deploy our application (the 3 services above) to GKE and inject the sidecar proxy into the deployments

1. In the [**github repo](https://github.com/nmallya/istiodemo)**, you will see 4 folders (the istio folder that was created when I installed the various components and the 3 folders for my microservices)

![](https://cdn-images-1.medium.com/max/2000/1*MAf0A-CzGAijfUD5qbldQA.png)

5. For each microservice, I have a corresponding Kubernetes *Deployment* and *Service* created in the **kube** folder in the petinfo.yaml file. The *Services* are called **petservice, petdetailsservice and petmedicalhistoryservice**. Since the PetService is accessible publicly, it has a Kubernetes *Ingress* that points to the **petservice ***Service*.

6. You can go to each service folder, update your project and cluster names in the **deploy.sh file **and run it. It builds the service, creates a Docker image, uploads it to the Google Container Registry and then runs **istioctl **to inject the Envoy proxy. For example, for the PetService, it would look like:

    #!/usr/bin/env bash

    export PROJECT=nithinistioproject 
    export CONTAINER_VERSION=feb4v2
    export IMAGE=gcr.io/$PROJECT/petservice:$CONTAINER_VERSION
    export BUILD_HOME=.

    gcloud config set project $PROJECT
    gcloud container clusters get-credentials nithinistiocluster --zone us-central1-a --project $PROJECT

    echo $IMAGE
    docker build -t petservice -f "${PWD}/Dockerfile" $BUILD_HOME
    echo 'Successfully built ' $IMAGE

    docker tag petservice $IMAGE
    echo 'Successfully tagged ' $IMAGE

    #push to google container registry
    gcloud docker -- push $IMAGE
    echo 'Successfully pushed to Google Container Registry ' $IMAGE

    # inject envoy proxy
    **kubectl apply -f <(istioctl kube-inject -f "${PWD}/kube/petinfo.yaml")**

In the above code, the highlighted line shows how we can use the Istio command line tool (**istioctl**) to inject the proxy into our various Kubernetes deployments.

The **petinfo.yaml file** under the petservice folder contains configuration for a *Service*, a *Deployment* and an *Ingress*. It looks like:

    apiVersion: v1
    kind: Service
    metadata:
      name: petservice
      labels:
        app: petservice
    spec:
      ports:
      - port: 9080
        name: http
      selector:
        app: petservice
    ---
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: petservice-v1
    spec:
      replicas: 1
      template:
        metadata:
          labels:
            app: petservice
            version: v1
        spec:
          containers:
          - name: petservice
            image: gcr.io/nithinistioproject/petservice:feb4v2
            imagePullPolicy: IfNotPresent
            ports:
            - containerPort: 9080
    ---

    ###########################################################################
    # Ingress resource (gateway)
    ##########################################################################
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: gateway
      annotations:
        kubernetes.io/ingress.class: "istio"
    spec:
      rules:
      - http:
          paths:
          - path: /pet/.*
            backend:
              serviceName: petservice
              servicePort: 9080
    ---

Once you run deploy.sh *for each of the 3 services*, you can check whether the Deployment, Service and Ingress have been created by executing:

    mallyn01$ **kubectl get deployment**

    NAME                          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE

    petdetailsservice-v1          1         1         1            1           1h

    petmedicalhistoryservice-v1   1         1         1            1           58m

    petservice-v1                 1         1         1            1           54m

    mallyn01$ **kubectl get service**

    NAME                       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE

    kubernetes                 ClusterIP   10.51.240.1    <none>        443/TCP    2d

    petdetailsservice          ClusterIP   10.51.255.10   <none>        9081/TCP   1h

    petmedicalhistoryservice   ClusterIP   10.51.244.19   <none>        9082/TCP   59m

    petservice                 ClusterIP   10.51.242.18   <none>        9080/TCP   1h

    
    petservice mallyn01$ **kubectl get ing**

    NAME      HOSTS     ADDRESS        PORTS     AGE

    gateway   *         **108.59.82.93**   80        1h

    mallyn01$ **kubectl get pods**

    NAME                                           READY     STATUS    RESTARTS   AGE

    petdetailsservice-v1-5bb8c65577-jmn6r          **2/2**       Running   0          12h

    petmedicalhistoryservice-v1-5757f98898-tq5j8   **2/2**       Running   0          12h

    petservice-v1-587696b469-qttqk                 **2/2**       Running   0          12h

When you look at the **pods** in the above console output, you will notice that **2/2** containers are running even though you deployed only 1 service per container. The other container is the sidecar proxy that has been injected by the istioctl command.

7. Once all the above are running, you can use the Ingress’s IP Address and invoke a sample endpoint to fetch a Pet’s details.

    mallyn01$ **curl [http://108.59.82.93/pet/123](http://108.59.82.93/pet/123)**

    {
      "petDetails": {
        "petName": "Maximus",
        "petAge": 5,
        "petOwner": "Nithin Mallya",
        "petBreed": "Dog"
      },
      "petMedicalHistory": {
        "vaccinationList": [
          "Bordetella, Leptospirosis, Rabies, Lyme Disease"
        ]
      }
    }

**Note**: Since the PetService invokes the PetDetailsService and the PetMedicalHistoryService, the actual invocation would look like:

    fetch('[http://**petdetailsservice:9081**/pet/123/details'](http://petdetailsservice:9081/pet/123/details'))
              .then(res => res.text())
              .then(body => console.log(body));
            ;

    fetch('[http://**petmedicalhistoryservice:9082**/pet/123/medicalhistory'](http://petmedicalhistoryservice:9082/pet/123/medicalhistory'))
              .then(res => res.text())
              .then(body => console.log(body));
            ;

**Conclusion**: We covered a LOT of material (and this is just Part I !!)

In subsequent parts, I’ll include details on how to use other Istio features such as ramping traffic to newer versions of the app, using the Performance monitoring dashboards etc.

Special thanks to [Ray Tsang](undefined) for his [presentation](https://www.youtube.com/watch?v=AGztKw580yQ&t=231s) on Istio

**Resources**:

1. The Istio home page [https://istio.io/](https://istio.io/)

1. DevOxx Istio presentation by [Ray Tsang](undefined): [https://www.youtube.com/watch?v=AGztKw580yQ&t=231s](https://www.youtube.com/watch?v=AGztKw580yQ&t=231s)

1. Github link to this example: [https://github.com/nmallya/istiodemo](https://github.com/nmallya/istiodemo)

1. All things Kubernetes: [https://kubernetes.io/](https://kubernetes.io/)

1. Microservices: [https://martinfowler.com/articles/microservices.html](https://martinfowler.com/articles/microservices.html)

1. Spring Cloud Netflix: [https://github.com/spring-cloud/spring-cloud-netflix](https://github.com/spring-cloud/spring-cloud-netflix)
