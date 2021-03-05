
# Back to Microservices with Istio (Part 1)



**Istio** is an Open Source project developed in partnership between teams from Google, IBM, and Lyft and it provides a solution to the complexities of microservice based application, to name a few:

* **Traffic management**: Timeouts, retries, load balancing,

* **Security: **End-user Authentication and Authorization,

* **Observability:** Tracing, monitoring, and logging.

All of these can be solved in the application layer, but your services don’t end up being so ‘Micro’ anymore, all the additional effort for implementing these is a strain in the company’s resources, resources that can be used to deliver business value. Let’s take an example:
> PM: How long will it take to add a feedback feature?
> Dev: Two sprints.
> PM: What…? That’s just a CRUD!
> Dev: Creating the CRUD is the easy part but then we need to authenticate and authorize users and services. And because the network is not reliable we need to implement retries, and circuit breakers in the clients, and to make sure that we do not take the whole system down we need Timeouts and Bulkheads, additionally to detect issues we need monitoring, tracing […]
> PM: Let’s just stick it in the Product Service then. Jeez!

You get the idea, all the ceremony, and effort that must go in, for us to add one service is enormous. In this article, we’ll showcase how Istio removes all the above-mentioned cross-cutting concerns from our services.

![Figure 1. The Ceremony of a Microservice](https://cdn-images-1.medium.com/max/2000/1*Z9G1wbqPf1MhpDwJUKz1ZQ.png)*Figure 1. The Ceremony of a Microservice*

**Note:** This article assumes that you have a working knowledge of Kubernetes. If it’s not the case I recommend you to read [my introduction to Kubernetes](https://medium.freecodecamp.org/learn-kubernetes-in-under-3-hours-a-detailed-guide-to-orchestrating-containers-114ff420e882), and then continue with this article.

## The Idea of Istio

In a world without Istio one service makes direct requests to another and in cases of failure, the service needs to handle it by retrying, timeouting, opening the circuit breaker etc.

![Figure 2. Network traffic in Kubernetes](https://cdn-images-1.medium.com/max/NaN/1*NZLDRzdLh4WgP8E9t08QBQ.png)*Figure 2. Network traffic in Kubernetes*

To resolve this, Istio provides an ingenious solution by being completely separated from the services and act only by intercepting all network communication. And doing so it can implement:

* **Fault Tolerance** — Using response status codes it understands when a request failed and retries.

* **Canary Rollouts** — Forward only the specified percentage of requests to a new version of the service.

* **Monitoring and Metrics** — The time it took for a service to respond.

* **Tracing and Observability** — It adds special headers in every request and traces them in the cluster.

* **Security** — Extracts the JWT Token and Authenticates and Authorizes users.

To name a few (for real just a few) and get you intrigued! Let’s get to the Technical details!

## Istio’s Architecture

Istio intercepts all network traffic and applies a set of rules by injecting an intelligent proxy as a sidecar in every pod. The proxies that enable all the features comprise **The Data Plane** and those are dynamically configurable by **The Control Plane.**

## The Data Plane

The injected proxies enable Istio to easily achieve our requirements. For an example let’s check out the retrying and Circuit breaking functionalities.

![Figure 3. How envoys implement Retries & CircuitBreaking](https://cdn-images-1.medium.com/max/2000/1*tHn9Gv_6UlfB0jQzlX2BSw.gif)*Figure 3. How envoys implement Retries & CircuitBreaking*

To summarize:

1. Envoy sends the request to the first instance of service B and it fails.

1. The Envoy Sidecar retries. (1)

1. Returns a failed request to the calling proxy.

1. Which opens the Circuit Breaker and calls the next Service on subsequent requests. (2)

This means that you don’t have to use another Retry library, you don’t have to develop your own implementation of Circuit Breaking and Service Discovery in Programming Language X, Y or Z. All of those and more are provided out of the box by Istio and **NO **code changes are required.

Great! Now you want to join the voyage with Istio, but you still have some doubts, some open questions. Is this a One-Size-Fits-All-Solution, which you’re suspicious about, as it always ends up being One-Size-Fits None solution!

You finally mutter the question: “Is this configurable?”

Welcome to the cruise my friend and let’s get introduced to the Control Plane.

## The Control Plane

Is composed of three components: The **Pilot**, the **Mixer,** and the **Citadel** that in combination configure Envoys to route traffic, enforce policies and collect telemetry data. Visually presented in the image below.

![Figure 4. Control Plane in relation to Data Plane](https://cdn-images-1.medium.com/max/NaN/1*8gH0GAnncEE6VUIbwnGUww.png)*Figure 4. Control Plane in relation to Data Plane*

The envoys (i.e. the data plane) are configured using [Kubernetes Custom Resource Definitions](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) defined by Istio and specialized for this purpose. Which means that for you it’s just another Kubernetes Resource with a familiar syntax. Which after being created will be picked up by **the control plane** that applies it to the envoys.

## Relation of Services to Istio

We described the relation of Istio to our Services, but not the other way around, what’s the relation of our Services to Istio?

Frankly, our services have as much knowledge of Istio’s presence, as fish do of water, they will ask themselves “What the hell is water?”.

![Drawing by [Victoria Dimitrakopoulos](https://fityourself.club/@victoriadimitrakopoulos)](https://cdn-images-1.medium.com/max/2800/1*PaPk2TSvDlGJdXPubvXQ1g.png)*Drawing by [Victoria Dimitrakopoulos](https://fityourself.club/@victoriadimitrakopoulos)*

This means that you can pick a working cluster and after deploying the components of Istio, the services within will continue to work and in the same manner, you can remove the components and everything will be just fine. Understandably, you would lose the capabilities provided by Istio.

We had enough of theory, let’s put this knowledge into practice!

## Istio in Practice

Istio requires a Kubernetes Cluster with at least 4 vCPU and 8 GB of RAM. To quickly set up a cluster and follow up with this article, I recommend using the Google Cloud Platform, which provides new users with a [$300 free trial](https://console.developers.google.com/billing/freetrial?hl=en).

After creating the cluster and configuring access with the Kubernetes command line tool we’re ready to install Istio using the Helm Package manager.

## Installing Helm

Install the Helm client on your computer as explained in [the official docs](https://docs.helm.sh/using_helm/#installing-the-helm-client). Which we will use to generate the Istio installation templates in the next section.

## Installing Istio

Download Istio’s resources from the [latest release](https://github.com/istio/istio/releases/tag/1.0.5), extract the contents into one directory that we will refer to as [istio-resources].

To easily identify the Istio resources create a namespace istio-system in your Kubernetes Cluster:

    **$ kubectl create namespace istio-system**

Complete the installation by navigating to [istio-resources] directory and executing the command below:

    **$ helm template install/kubernetes/helm/istio \
      --set global.mtls.enabled=false \
      --set tracing.enabled=true \
      --set kiali.enabled=true \
      --set grafana.enabled=true \
      --namespace istio-system > istio.yaml**

The above command prints out the core components of Istio into the file istio.yaml. We customized the template using the following parameters:

* **global.mtls.enabled** is set to false to keep the introduction focused.

* **tracing.enabled **enables tracing of requests using jaeger.

* **kiali.enabled **installs Kiali in our cluster for Visualizing Services and Traffic

* **grafana.enabled **installs Grafana to visualize the collected metrics.

Apply the generated resources by executing the command:

    **$ kubectl apply -f istio.yaml**

This marks the completion of the Istio installation in our cluster! Wait until all pods in the istio-system namespace are in Running or Completed state by executing the command below:

    **$ kubectl get pods -n istio-system**

Now we’re ready to continue with the next section, where we will get the sample application up and running.

## Sentiment Analysis Application Architecture

We will use the same microservice application used in the [Kubernetes Introduction article](https://medium.freecodecamp.org/learn-kubernetes-in-under-3-hours-a-detailed-guide-to-orchestrating-containers-114ff420e882), it’s complex enough to showcase Istio’s features in practice.

The App is composed of four microservice:

* **SA-Frontend service**: Serves the frontend Reactjs application.

* **SA-WebApp service**: Handles requests for Sentiment Analysis.

* **SA-Logic service**: Performs sentiment analysis.

* **SA-Feedback service**: Receives feedbacks from the users about the accuracy of the analysis.

![Figure 6 Sentiment Analysis microservices](https://cdn-images-1.medium.com/max/NaN/1*AcfdupyEsbdgTtKV6obx7A.png)*Figure 6 Sentiment Analysis microservices*

In figure 6, besides the services we see the Ingress Controller which in Kubernetes routes incoming requests to the appropriate services, Istio uses a similar concept called Ingress Gateway, which will be introduced in continuation of the article.

## Running the Application with Istio Proxies

To follow up with this article clone the repository [istio-mastery](https://github.com/rinormaloku/istio-mastery), containing the application and the manifests for Kubernetes and Istio.

### Sidecar Injection

The injection is done **Automatically** or **Manually**. To enable automatic sidecar injection, we need to label the namespace with istio-injection=enabled, by executing the command below:

    **$ kubectl label namespace default istio-injection=enabled
    **namespace/default labeled

From now every pod that gets deployed into the default namespace will get the sidecar injected. To verify this let’s deploy the sample application by navigating to the root folder of the [istio-mastery] repository and executing the following command:

    **$ kubectl apply -f resource-manifests/kube
    **persistentvolumeclaim/sqlite-pvc created
    deployment.extensions/sa-feedback created
    service/sa-feedback created
    deployment.extensions/sa-frontend created
    service/sa-frontend created
    deployment.extensions/sa-logic created
    service/sa-logic created
    deployment.extensions/sa-web-app created
    service/sa-web-app created

With the services deployed verify that the pods have two containers (the service and the sidecar) by executing the command **kubectl get pods **and ensuring that under the column ready, we see the value “**2/2**” indicating that both containers are running. As seen below:

    **$ kubectl get pods**
    NAME                           READY     STATUS    RESTARTS   AGE
    sa-feedback-55f5dc4d9c-c9wfv   2/2       Running   0          12m
    sa-frontend-558f8986-hhkj9     2/2       Running   0          12m
    sa-logic-568498cb4d-2sjwj      2/2       Running   0          12m
    sa-logic-568498cb4d-p4f8c      2/2       Running   0          12m
    sa-web-app-599cf47c7c-s7cvd    2/2       Running   0          12m

Visually presented in figure 7.

![Figure 7. Envoy proxy in one of the Pods](https://cdn-images-1.medium.com/max/2000/1*GeTGpCf5vAdPHzPznKYhdg.png)*Figure 7. Envoy proxy in one of the Pods*

With the application up and running now we need to allow incoming traffic to reach our application.

## Ingress Gateway

A best practice for allowing traffic into the cluster is through Istio’s **Ingress Gateway** which positions itself at the edge of the cluster and on incoming traffic enables Istio’s features like routing, load balancing, security, and monitoring.

During Istio’s installation, the Ingress Gateway component and a service that exposes it externally were installed into the cluster. To get the services External IP execute the command below:

    **$ kubectl get svc -n istio-system -l istio=ingressgateway**
    NAME                   TYPE           CLUSTER-IP     EXTERNAL-IP
    istio-ingressgateway   LoadBalancer   10.0.132.127   13.93.30.120

In the continuation of this article we will access the application on this IP (referred to as the EXTERNAL-IP), for convenience, save it in a variable by executing the command below:

    **$ EXTERNAL_IP=$(kubectl get svc -n istio-system \
      -l app=istio-ingressgateway \
      -o jsonpath='{.items[0].status.loadBalancer.ingress[0].ip}')**

If you reach this IP in your browser and you will get a Service Unavailable error, as **by default Istio blocks any incoming traffic** until we define a Gateway.

## The Gateway Resource

A Gateway is a Kubernetes Custom Resource Definition defined upon Istio’s installation in our cluster that enables us to specify the Ports, Protocol and Hosts for which we want to allow incoming traffic.

In our scenario, we want to allow HTTP traffic on port 80, for all hosts. Achieved with the following definition:

<iframe src="https://medium.com/media/537dc8f0431ee39082e535b6f864351c" frameborder=0></iframe>

All the configuration is self-explanatory besides the selector istio: ingressgateway. Using this selector, we can specify to which Ingress Gateway to apply the configuration, and in our case, it’s the default ingress gateway controller installed on Istio setup.

Apply the configuration by executing the command below:

    **$ kubectl apply -f resource-manifests/istio/http-gateway.yaml **gateway.networking.istio.io/http-gateway created

The gateway now allows access in port 80 but it has no concept where to route the requests. That is achieved using **Virtual Services**.

## The VirtualService resource

The VirtualService instructs the Ingress Gateway how to route the requests that were allowed into the cluster.

For our application requests coming through the **http-gateway **must be routed to the **sa-frontend, sa-web-app** and** sa-feedback **services (show in figure 8).

![Figure 8. Routes to be configured with VirtualServices](https://cdn-images-1.medium.com/max/NaN/1*anirEAIId7_cg_obeYW4Zw.png)*Figure 8. Routes to be configured with VirtualServices*

Let’s break down the requests that should be routed to SA-Frontend:

* **Exact path /**should be routed to SA-Frontend to get the Index.html

* **Prefix path /static/*** should be routed to SA-Frontend to get any static files needed by the frontend, like Cascading Style Sheets and JavaScript files.

* **Paths matching the regex '^.*\.(ico|png|jpg)$'** should be routed to SA-Frontend as it is an image, that the page needs to show.

This is achieved with the following configuration:

<iframe src="https://medium.com/media/ac524824a3255a99da284bafc06141cc" frameborder=0></iframe>

The important points here are:

1. This VirtualService applies to requests coming through the **http-gateway.**

1. Destination defines the service where the requests are routed to.

**Note:** The configuration above is in the file sa-virtualservice-external.yaml, it also contains the configuration to route to SA-WebApp and SA-Feedback but was shortened for brevity.

Apply the VirtualService by executing:

    **$ kubectl apply -f resource-manifests/istio/sa-virtualservice-external.yaml
    **virtualservice.networking.istio.io/sa-external-services created

**Note:** When we apply Istio resources the Kubernetes API Server creates an event received by Istio’s Control Plane which then applies the new configuration to the envoy proxies of every pod. And the Ingress Gateway controller is another Envoy which is configured by the Control Plane, visually presented in figure 9.

![Figure 9. Configuring **Istio-IngressGateway** to route requests](https://cdn-images-1.medium.com/max/NaN/1*0XHfOTn7svQ8nvChkKmE5Q.png)*Figure 9. Configuring **Istio-IngressGateway** to route requests*

The Sentiment Analysis app is now accessible on http://{EXTERNAL-IP}/. If you get a Not Found status, do not worry *sometimes it takes a bit for the configuration to go in effect and update the envoy caches*.

Before moving to the next section use the app to generate some traffic.

## Kiali — Observability

To access Kiali’s Admin UI execute the command below:

    $ kubectl port-forward \
        $(kubectl get pod -n istio-system -l app=kiali \
        -o jsonpath='{.items[0].metadata.name}') \
        -n istio-system 20001

And open [http://localhost:20001/](http://localhost:20001/) login using “admin” (without quotes) for user and password. There is a ton of useful features, for example checking the configurations of Istio Components, visualizing services according to the information collected by intercepting network requests and answering, “who is calling who?”, “which version of a service has failures?” etc. Take some time to checkout Kiali before moving on to the next goodie, visualizing metrics with Grafana!

![Figure 10. Kiali — Service Observability](https://cdn-images-1.medium.com/max/NaN/1*5s-LAV-rWNMWkWkkRfH8jg.png)*Figure 10. Kiali — Service Observability*

## Grafana — Metrics Visualization

The metrics collected by Istio are scraped into Prometheus and Visualized using Grafana. To access the Admin UI of Grafana execute the command below and open [http://localhost:3000.](http://localhost:3000.)

    $ kubectl -n istio-system port-forward \
        $(kubectl -n istio-system get pod -l app=grafana \
        -o jsonpath={.items[0].metadata.name}) 3000

On the top left click the menu **Home **and** **select** Istio Service Dashboard **and on the top left corner select the service starting with** sa-web-app, **you will be presented with the collected metrics, as seen on the image below:

![](https://cdn-images-1.medium.com/max/NaN/1*2dfPTHXZnFZ2attoax7GKQ.png)

Holly molly that’s an empty and totally non-exciting view, management would never approve of this. Let’s cause some load by executing the command below:

    $ while true; do \
        curl -i [http://$EXTERNAL_IP/sentiment](http://$EXTERNAL_IP/sentiment) \
        -H “Content-type: application/json” \
        -d ‘{“sentence”: “I love yogobella”}’; \
        sleep .8; done

Now we have prettier graphs 😊, and additionally, we have the amazing tools of Prometheus for monitoring and Grafana for visualizing the metrics that enable us to know the performance, health, improvement or degradation of our services throughout time!

Lastly, we will investigate Tracing requests throughout services.

## Jaeger — Tracing

We need tracing because the more services we have the harder it gets to pinpoint the cause of failure. Let’s take the simple case in the image below:

![Figure 12. A commonly random failed request](https://cdn-images-1.medium.com/max/NaN/1*8jBrTp5SF4ONfWUnJ2ux3Q.png)*Figure 12. A commonly random failed request*

The request goes in, failure goes out, *what was the cause?* *The first service?* *Or the second?* Exceptions are in both, Let’s get down to the logs of each. How many times do you find yourself doing this? Our job feels more like Software Detectives than Developers.

This is a prevalent problem in Microservices and it’s solved using Distributed Tracing Systems where the services pass a unique header to each other and then this information is forwarded to the Distributed Tracing System where the request trace is put together. An example is shown in figure 13.

![Figure 13. TraceId used to identify the span of a request](https://cdn-images-1.medium.com/max/NaN/1*9355tU9NGsRR04GwwVbl8w.png)*Figure 13. TraceId used to identify the span of a request*

Istio uses Jaeger Tracer that implements the OpenTracing API, a vendor-neutral framework. To get access the Jaegers UI execute the command below:

    $ kubectl port-forward -n istio-system \
        $(kubectl get pod -n istio-system -l app=jaeger \
        -o jsonpath='{.items[0].metadata.name}') 16686

Then open the UI in [http://localhost:16686](http://localhost:16686/), select the **sa-web-app **service, *if the service is not shown on the dropdown generate some activity on the page and hit refresh*. Afterward click the button **Find Traces, **which** **displays the most recent traces, select any and a detailed breakdown of all the traces will be shown**, **as seen in figure 14.

![Figure 14. Jaeger — a request trace](https://cdn-images-1.medium.com/max/NaN/1*mMKj2ld0zeTYVF1F4FmTBw.png)*Figure 14. Jaeger — a request trace*

The trace shows:

1. The request comes to the **istio-ingressgateway** (it’s the first contact with one of the services so for the request the Trace ID is generated) then the gateway forwards the request to the **sa-web-app** service.

1. In the **sa-web-app** service, the request is picked up by the Envoy sidecar and a span child is created (that’s why we see it in the traces) and forwarded to the **sa-web-app** container instance.

1. Here the method **sentimentAnalysis** handles the request. These traces are generated by the application, meaning that code changes were required).

1. From where a POST request is started to **sa-logic. **Trace ID needs to be propagated by **sa-web-app**.

5. …

**Note:** At the 4th point our application needs to pick up the headers generated by Istio and pass them down on the next requests, as shown in the image below.

![Figure 15. (A) Istio propagating headers, **(B) Services propagating headers**](https://cdn-images-1.medium.com/max/NaN/1*YDMLGTWmx3l9QmUXT7TwZQ.png)*Figure 15. (A) Istio propagating headers, **(B) Services propagating headers***

Istio does the main heavy lifting as it generates the headers on incoming requests, creates new spans on every sidecar, propagates them, but without our services propagating the headers as well, we will lose the full trace of the request.

The headers to propagate are:

    x-request-id
    x-b3-traceid
    x-b3-spanid
    x-b3-parentspanid
    x-b3-sampled
    x-b3-flags
    x-ot-span-context

Despite it being a simple task, there are already [many libraries](https://github.com/opentracing-contrib) that simplify the process, for example in the **sa-web-app** service, the **RestTemplate** client is instrumented to propagate the headers by simply adding the Jaeger and OpenTracing libraries in the [dependencies](https://github.com/rinormaloku/istio-mastery/blob/master/sa-webapp/pom.xml#L36-L47).

*Note: The Sentiment Analysis app showcases implementations for Flask, Spring and ASP.NET Core.*

Now after investigating what we get out of the box (or partially out of the box 😜) let’s get to the main topic here, fine-grained routing, managing network traffic, security and more!

## Traffic Management

Using the Envoy’s Istio provides a host of new capabilities to your cluster enabling:

* **Dynamic request routing**: Canary deployments, A/B testing,

* **Load balancing: **Simple** **and Consistent Hash balancing,

* **Failure Recovery**: Timeouts, retries, circuit breakers,

* **Fault Injection**: Delays, abort requests etc.

In the sequence of this article, we’ll showcase these capabilities in our application and get introduced to new concepts along the way. The first concept we will delve into is DestinationRules and using those we’ll enable A/B Testing.

## A/B Testing — Destination Rules in Practice

A/B Testing is used when we have two versions of an application (usually the versions differ visually) and we aren’t 100% sure which will increase user interaction, so we try both versions at the same time and collect metrics.

Execute the command below to deploy a second version of the frontend, needed for demonstrating A/B Testing:

    **$ kubectl apply -f resource-manifests/kube/ab-testing/sa-frontend-green-deployment.yaml
    **deployment.extensions/sa-frontend-green created

The deployment manifest for the green version differs in two points:

1. The image is based on a different tag: **istio-green** and

1. Pods are labeled with **version: green**.

And as both deployments have the label **app: sa-frontend** requests routed by the virtual service **sa-external-services **to the service **sa-frontend** will be forwarded to all of its instances and will be load balanced using the round robin algorithm, which causes the issue presented in figure 16.

![Figure 16. Requested files are not found](https://cdn-images-1.medium.com/max/NaN/1*JD0GTy4FbRP9EHIFK80ZLA.png)*Figure 16. Requested files are not found*

The files are not found because they are named differently in the different versions of the app. Let’s verify that:

    **$ curl --silent [http://$EXTERNAL_IP/](http://$EXTERNAL_IP/) | tr '"' '\n' | grep main
    **/static/css/main.c7071b22.css
    /static/js/main.059f8e9c.js
    **$ curl --silent [http://$EXTERNAL_IP/](http://$EXTERNAL_IP/) | tr '"' '\n' | grep main
    **/static/css/main.f87cd8c9.css
    /static/js/main.f7659dbb.js

This means that the index.html that requests one version of the static files might be load balanced to the pods delivering the other version, where understandably the other files do not exist.

This means that for our app to work we need to introduce the restriction that **“the version of the app that served the index.html, must serve subsequent requests”**.

We’ll achieve this using Consistent Hash Loadbalancing, which** is the process that forwards requests from the same client to the same backend instance, **using a predefined property, like for example an HTTP header**. **Made possible by** **DestionatioRules.

## DestinationRules

After a request gets routed by the **VirtualService** to the correct service, then using **DestinationRules** we can specify policies that apply to the traffic intended for the instances of this Service, as seen in figure 17.

![Figure 17. Traffic Management with Istio Resources](https://cdn-images-1.medium.com/max/NaN/1*jBqAaJnlk_XIKZ8unokfrg.png)*Figure 17. Traffic Management with Istio Resources*

**Note:** Figure 17, visualizes how Istio resources are affecting the network traffic, in an easily understandable way. But, to be precise the decision to which instance to forward the request to is made in the Ingress Gateway’s Envoy, configured by the CRDs.

Using Destination Rules we can configure load balancing to have consistent hashing and ensure that the same user is responded by the same instance of the service. Achieved with the following configuration:

<iframe src="https://medium.com/media/f000ed39c1abf754a053933703a69eed" frameborder=0></iframe>

1. Generate a consistent hash according to the contents of the “version” header.

Apply the configuration by executing the command below and give it a try!

    **$ kubectl apply -f resource-manifests/istio/ab-testing/destinationrule-sa-frontend.yaml
    **destinationrule.networking.istio.io/sa-frontend created

Execute the command below and verify that you get the same files when specifying the version header:

    **$ curl --silent -H "version: yogo" [http://$EXTERNAL_IP/](http://$EXTERNAL_IP/) | tr '"' '\n' | grep main**

**Note: **You can use this [chrome extension](https://chrome.google.com/webstore/detail/modheader/idgpnmonknjnojddfkpgkljpfnnfcklj) to add different values to the version header, to test in your browser.

DestinationRules have more LoadBalancing capabilities, for all the details check out the [official docs](https://preliminary.istio.io/docs/reference/config/istio.networking.v1alpha3.html#LoadBalancerSettings).

Before moving on to explore VirtualService in more detail, remove the green version of the app and the destination rule by executing the commands below:

    **$ kubectl delete -f resource-manifests/kube/ab-testing/sa-frontend-green-deployment.yaml
    **deployment.extensions “sa-frontend-green” deleted
    **$ kubectl delete -f resource-manifests/istio/ab-testing/destinationrule-sa-frontend.yaml
    **destinationrule.networking.istio.io “sa-frontend” deleted

## Shadowing — Virtual Services in Practice

Shadowing or Mirroring is used when we want to test a change in production but not affect end-users, so we mirror the requests into a second instance that has the change and evaluate it. *To phrase it simpler it’s when one of your colleagues picks the most critical issue and makes a Big ball of mud Pull Request, that nobody can really review.*

To test out this feature lets create a second instance of SA-Logic (*which is buggy*) by executing the command below:

    **$ kubectl apply -f resource-manifests/kube/shadowing/sa-logic-service-buggy.yaml
    **deployment.extensions/sa-logic-buggy created

Execute the following command and verify that all instances are labeled with the respective versions and additionally with **app=sa-logic**:

    **$ kubectl get pods -l app=sa-logic --show-labels**
    NAME                              READY   LABELS
    sa-logic-568498cb4d-2sjwj         2/2     app=sa-logic,**version=v1**
    sa-logic-568498cb4d-p4f8c         2/2     app=sa-logic,**version=v1**
    sa-logic-buggy-76dff55847-2fl66   2/2     app=sa-logic,**version=v2**
    sa-logic-buggy-76dff55847-kx8zz   2/2     app=sa-logic,**version=v2**

As the service **sa-logic** targets pods labeled with **app=sa-logic**, any incoming requests will be load balanced between all instances, as shown in figure 18.

![Figure 18. Round Robin load balancing](https://cdn-images-1.medium.com/max/NaN/1*K0L5SspnCi2uI1Wjhl8d8Q.png)*Figure 18. Round Robin load balancing*

But we want requests to be routed to the instances with version v1 and mirrored to the instances with version v2, as shown in figure 19.

![Figure 19. Routing to v1 and Mirroring to v2](https://cdn-images-1.medium.com/max/NaN/1*8uf4iKo1yI3FtbcGBfR2bg.png)*Figure 19. Routing to v1 and Mirroring to v2*

This is achieved using a VirtualService in combination with a DestinationRule, where the destination rule specifies the subsets and VirtualService routes to the specific subset.

## Specifying Subsets with Destination Rules

We define the subsets with the following configuration:

<iframe src="https://medium.com/media/a87d885b176332576008cabadd227a37" frameborder=0></iframe>

1. Host defines that this rule applies only when routing has occurred towards **sa-logic** service

1. Subset name used when routing to instances of a subset.

1. Label defines the key-value pairs that need to match for an instance to be part of the subset.

Apply the configuration executing the command below:

    **$ kubectl apply -f resource-manifests/istio/shadowing/sa-logic-subsets-destinationrule.yaml
    **destinationrule.networking.istio.io/sa-logic created

With the subsets defined we can move on and configure a VirtualService to apply to requests towards **sa-logic** where the requests are:

1. Routed to the subset named v1 and,

1. Mirrored to the subset named v2.

And this is achieved with the manifest below:

<iframe src="https://medium.com/media/8c6543ff79f6bd6533699926f0418f91" frameborder=0></iframe>

As everything is self-explanatory let’s just see it in action:

    **$ kubectl apply -f resource-manifests/istio/shadowing/sa-logic-subsets-shadowing-vs.yaml
    **virtualservice.networking.istio.io/sa-logic created

Add some load by executing the following command:

    **$ while true; do curl -v [http://$EXTERNAL_IP/sentiment](http://$EXTERNAL_IP/sentiment) \
        -H “Content-type: application/json” \
        -d ‘{“sentence”: “I love yogobella”}’; \
        sleep .8; done**

Check the results in Grafana, where we can see that the buggy version is failing about 60% of the requests, but none of the failures affected the end-users as they were responded by the currently active service.

![Figure 20. The success rate of sa-logic service versions](https://cdn-images-1.medium.com/max/NaN/1*hgANZ4AinrNZZsZ6jjbVCA.png)*Figure 20. The success rate of sa-logic service versions*

In this section, we saw for the first time a VirtualService that was applied to the envoys of our services, when the **sa-web-app** makes a request towards **sa-logic **this goes through the sidecar envoy, which via the VirtualService is configured to route to the subset v1 and mirror to the subset v2 of the **sa-logic** service.

I can see you thinking “Darn man Virtual Services are simple!”, in the next section, we’ll extend the sentence to “Simply Amazing!”.

## Canary Deployments

Canary Deployment is the process of rolling out a new version of an application to a small set of users, as a step to verify the absence of issues, and then with a higher assurance of quality release to the wider audience.

We will continue with the same buggy subset of **sa-logic** to demonstrate canary deployments.

Let’s start boldly and send 20% of the users to the buggy version (this represents the canary deployment) and 80% to the healthy service by applying the VirtualService below:

<iframe src="https://medium.com/media/f73e8b5910255812b163b7ee521e42ea" frameborder=0></iframe>

1. Weight specifies the percentage of requests to be forwarded to the destination or subset of the destination.

Update the previous **sa-logic** virtual service configuration using the following commands:

    **$ kubectl apply -f resource-manifests/istio/canary/sa-logic-subsets-canary-vs.yaml
    **virtualservice.networking.istio.io/sa-logic configured

We immediately see that some of our requests are failing:

    **$ while true; do \
       curl -i [http://$EXTERNAL_IP/sentiment](http://$EXTERNAL_IP/sentiment) \
       -H “Content-type: application/json” \
       -d ‘{“sentence”: “I love yogobella”}’ \
       --silent -w “Time: %{time_total}s \t Status: %{http_code}\n” \
       -o /dev/null; sleep .1; done
    **Time: 0.153075s Status: 200
    Time: 0.137581s Status: 200
    Time: 0.139345s Status: 200
    Time: 30.291806s Status: 500

VirtualServices enable Canary Deployments and with this method, we reduced potential damages to 20% of our user base. Beautiful! Now we can use Shadowing and Canary Deployments every time we are insecure about our code, in other words always. 😜

## Timeouts & Retries

It’s not always that the code is buggy. In the list of “[The 8 fallacies of distributed computing](https://en.wikipedia.org/wiki/Fallacies_of_distributed_computing#The_fallacies)” the first fallacy is that “The network is reliable”. The network is NOT reliable, and that’s why we need Timeouts and Retries.

For demonstration purposes, we will continue to use the buggy version of **sa-logic**, where the random failures simulate the unreliability of the network.

The buggy service has a one-third chance of taking too long to respond, one-third chance of ending in an Internal Server Error and the rest complete successfully.

To alleviate these issues and improve the user experience we can:

1. Timeout if the service takes longer than 8 seconds and

1. Retry on failed requests.

This is achieved with the following resource definition:

<iframe src="https://medium.com/media/7583bd312760664c64a37288029dcad8" frameborder=0></iframe>

1. The request has a timeout of 8 seconds,

1. It attempts 3 times,

1. An attempt is marked as failed if it takes longer than 3 seconds.

This is an optimization, as the user won’t be waiting for longer than 8 seconds and we retry three times in case of failures, increasing the chance of resulting in a successful response.

Apply the updated configuration with the command below:

    **$ kubectl apply -f resource-manifests/istio/retries/sa-logic-retries-timeouts-vs.yaml
    **virtualservice.networking.istio.io/sa-logic configured

And check out the Grafana graphs for the improvement in success rate (shown in figure 21).

![Figure 21. Improvement after using Timeouts & Retries](https://cdn-images-1.medium.com/max/NaN/1*_b94m4BSkeBnBQg_nUUsrA.png)*Figure 21. Improvement after using Timeouts & Retries*

Before moving into the next section delete **sa-logic-buggy** and the VirtualService by executing the command below:

    **$ kubectl delete deployment sa-logic-buggy
    **deployment.extensions “sa-logic-buggy” deleted
    **$ kubectl delete virtualservice sa-logic
    **virtualservice.networking.istio.io “sa-logic” deleted

## Circuit Breaker and Bulkhead patterns

Two important patterns in Microservice Architectures that enable self-healing of the services.

The **Circuit Breaker **is used to stop requests going to an instance of a service deemed as unhealthy and enable it to recover, and in the meantime client’s requests are forwarded to the healthy instances of that service (increasing success rate).

The **Bulkhead pattern** isolates failures from taking the whole system down, to take an example, Service B is in a corrupt state and another service (a client of Service B) makes requests to Service B this will result that the client will use up its own thread pool and won’t be able to serve other requests (even if those are not related to Service B).

I will skip implementations of these patterns because you can check out implementations in the [official docs](https://istio.io/docs/tasks/traffic-management/circuit-breaking/) and I’m way too excited to showcase Authentication and Authorization, which will be the subject of the next article.

## Part I — Summary

In this article, we deployed Istio in a Kubernetes cluster and using its Custom Resource Definitions like **Gateways**, **VirtualServices**, **DestinationRules **and it’s components it enabled the following features:

* Observability over our services by answering what services are running, how are they performing and how are they related, using **Kiali**.

* Metric collection and visualization, with **Prometheus **and **Grafana**.

* Request tracing with **Jaeger** (german for Hunter).

* Full and fine-grained control over the network traffic, enabling **Canary Deployments**, **A/B Testing, and Shadowing**.

* Easy implementation of **Retries, Timeouts, and CircuitBreakers**.

And all were possible without code changes or any additional dependencies, keeping your services small, easy to operate and maintain.

For your development team removing these cross-cutting concerns and centralizing them into Istio’s Control plane, means that new services are easy to be introduced, they aren’t resource-heavy as developers can focus in solving business problems. And up to now, no developer complained about “having to solve interesting business problems!”.

I would love to hear your thoughts in the comments below and feel free to reach out to me on [Twitter](https://twitter.com/rinormaloku) or on my page [rinormaloku.com](https://rinormaloku.com).

Without further ado let’s move to the second part [Back to Microservices with Istio (Part 2)](https://medium.com/google-cloud/back-to-microservices-with-istio-part-2-authentication-authorization-b079f77358ac), and let’s tackle Authentication and Authorization!
