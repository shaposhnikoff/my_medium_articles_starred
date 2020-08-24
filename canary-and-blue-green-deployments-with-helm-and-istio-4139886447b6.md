
# Canary and Blue/Green Deployments with Helm and Istio



In this article, we are going to **review ways in which we can deploy two versions of our application in production-like environments in Kubernetes and apply two different availability approaches.** The first is to distribute network traffic evenly between the two versions (canary) and the second is for 100% of the traffic to either one of them (blue/green) with zero downtime.

This article requires readers to have some basic knowledge of the following technologies:

* **Kubernetes**

* **Helm**

* **Istio**

* **AWS**

* **Linux**

In case some of the above does not sound familiar, I will cover some of the basics, so hold tight and let us dive in!

**What is blue/green deployment?**

Blue-green deployment is a technique that reduces downtime and risk by running two identical production environments called Blue and Green.

At any time, only one of the environments is live, with the live environment serving all production traffic. For this example, Blue is currently live and Green is idle.

As you prepare a new version of your software, deployment and the final stage of testing takes place in the environment that is *not* live: in this example, Green. Once you have deployed and fully tested the software in Green, you switch the router so all incoming requests now go to Green instead of Blue. Green is now live, and Blue is idle.

![](https://cdn-images-1.medium.com/max/4748/1*x2btJQHE9DEBTzLunWCVJQ.png)

**What is canary deployment then?**

Canary deployments are a pattern for rolling out releases to a subset of users or servers. The idea is to first deploy the change to a small subset of servers, test it, and then roll the change out to the rest of the servers. The canary deployment serves as an early warning indicator with less impact on downtime: if the canary deployment fails, the rest of the servers aren’t impacted.

![](https://cdn-images-1.medium.com/max/2568/1*ZdO7_GVA6DMcw5Rfph_Tzg.png)

**What is Helm all about?**

Helm is the first application package manager running on top of Kubernetes. It allows you to describe the application structure through convenient helm-charts and to manage it with simple commands.

*Why is Helm important?* Because it’s a huge shift in the way the server-side applications are defined, stored, and managed. Adoption of Helm might well be the key to the mass adoption of microservices, as using this package manager simplifies their management greatly.

*Why are microservices so important?* They have quite a few uses:

* When there are several microservices instead of a monolithic application, each microservice can be managed, updated and scaled individually

* Issues with one microservice do not affect the functionality of other components of the application

* The new application can be easily composed of existing loosely-coupled microservices

**What is Istio?**

Istio addresses the challenges developers and operators face as monolithic applications transition towards a distributed microservice architecture. To see how it helps to take a more detailed look at Istio’s service mesh.

The term *service mesh* is used to describe the network of microservices that make up such applications and the interactions between them. As a service mesh grows in size and complexity, it can become harder to understand and manage. Its requirements can include discovery, load balancing, failure recovery, metrics, and monitoring. A service mesh also often has more complex operational requirements, like A/B testing, canary rollouts, rate limiting, access control, and end-to-end authentication.

Istio provides behavioral insights and operational control over the service mesh as a whole, offering a complete solution to satisfy the diverse requirements of microservice applications.

Now let us dive into the solution:

If you want to follow along with this article, you will need to install all necessary resources, as per my GitHub repository: [*https://github.com/infinitelambda/istio-helm-deployment](https://github.com/infinitelambda/istio-helm-deployment)*

* EKS cluster

* kubectl

* istio

* istioctl

* helm3

As you can see from the repository, I have wrapped my application into a Helm chart so that it can be installed as a package. Thanks to Helm I am able to dynamically install different versions of the application on runtime.

I have created two versions of the application that was built in Docker images here: [*https://hub.docker.com/repository/docker/nikolambda/testing-app/tags?page=1](https://hub.docker.com/repository/docker/nikolambda/testing-app/tags?page=1)*

We need to create two namespaces (*prod* and *stage*) in which we will put the different versions of our application — let us create them:

    $ kubectl create namespace prod && kubectl create namespace stage

Then we need to tell Istio to monitor these namespaces by labeling them so it can keep track of the resources inside them:

    $ kubectl label namespace prod istio-injection=enabled && kubectl label namespace stage istio-injection=enabled

Now we can deploy the two versions of our application in each individual namespace by:

    $ helm install demoapp helm-chart/demoapp/ --wait --set deployment.tag=v1 --namespace prod
    $ helm install demoapp helm-chart/demoapp/ --wait --set deployment.tag=v2 --namespace stage

Once deployed, we have to install the two Istio files in the istio-config directory that will basically do the magic:

    apiVersion: networking.istio.io/v1alpha3
    kind: Gateway
    metadata:
      name: app-gateway
    spec:
      selector:
        istio: ingressgateway
      servers:
      - port:
          number: 80
          name: http
          protocol: HTTP
        hosts:
        - "*"
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: demoapp
    spec:
      hosts:
      - "*"
      gateways:
      - app-gateway
      http:
        - route:
          - destination:
              host: flaskapp.prod.svc.cluster.local 
            weight: 100
          - destination:
              host: flaskapp.stage.svc.cluster.local
            weight: 0

Both of these files and almost everything in Istio are built and run thanks to the Kubernetes **Custom Resources Definitions**.

**Custom Resources Definition** (CRD) is a powerful feature introduced in Kubernetes which enables users to add their own customized objects to the Kubernetes cluster and use it like any other native Kubernetes objects. So what is the **Gateway** project then? Let us look below at the official Istio explanation.

*“Gateway describes a load balancer operating at the edge of the mesh receiving incoming or outgoing HTTP/TCP connections. The specification describes a set of ports that should be exposed, the type of protocol to use, SNI configuration for the load balancer, etc.”*

**What is VirtualService?**

Let us look again into the official documentation:

*“A VirtualService defines a set of traffic routing rules to apply when a host is addressed. Each routing rule defines matching criteria for the traffic of a specific protocol. If the traffic is matched, then it is sent to a named destination service (or subset/version of it) defined in the registry.”*

So basically Gateway describes the listening policy of the Istio’s ingress controller and VirtualService will “glue” this Gateway service to the services of the actual pods running in the cluster.

Please notice that from lines 10 to 17 of VirtualService we actually tell Istio how to distribute the traffic to our services. *Weight* in this case refers to the traffic percentage on the said environment. It is very important that we have a formal means of redistributing traffic between the two environments (*prod* & *stage*) with different versions of the code.

The goal is to have the new version of the application fully tested in the staging environment and then migrating it to production, with 100% traffic (while keeping the old code for roll-back purposes). Production migration has to happen gradually. The staging environment needs to mandatorily be tested at 0% traffic to ensure it is ready for the gradual increase of traffic. Once these tests are successful, staging is ready for a 30% traffic redistribution, leaving production with 70% of the traffic.

The traffic increase on staging needs to happen gradually until it reaches 100% and tests have to be performed at every stage. Once the entire traffic has been migrated, we will have a staging environment at 100% traffic and a production environment at 0%. The next step will be to manually update the production environment with the latest version (that is on staging) and then switch the whole traffic (100%) back to production, once we are ready.

In your Kiali dashboard you should be able to see the following:

![](https://cdn-images-1.medium.com/max/3116/1*LU51Rlh_4dGvprPvpJ2Hbw.png)

The next step will be to get the DNS name of the load-balancer created by Istio:

    $ kubectl get svc -n istio-system | grep -i LoadBalancer | awk '{print $4}'

Then open your browser and copy-paste the DNS name of the ALB you are going to see the current version set up by the VirtualService weight distribution:

![](https://cdn-images-1.medium.com/max/4324/1*BEAsfANL27S0ITTFI3TqMA.png)

With Istio, we can do a lot of advanced configuration such as SSL mutual authentication between microservices and apply advanced routing policies with the help of DestinationRule: [*https://istio.io/docs/reference/config/networking/destination-rule/](https://istio.io/docs/reference/config/networking/destination-rule/)*

**Conclusion**

In this article, we discovered together how can we apply a hybrid solution between Canary and Blue/Green deployment with the help of Helm and Istio. There will be an additional part to show you how to automate this in a pipeline with GitLab. Stay tuned!

Don’t want to wait? Drop us a message [*contact@infinitelambda.com](mailto:contact@infinitelambda.com) *and find out more!

Here are some sources for further readings:

* [*https://www.strive2code.com/post/2018/11/30/kubernetes-in-azure](https://www.strive2code.com/post/2018/11/30/kubernetes-in-azure)*

* [*https://octopus.com/docs/deployment-patterns/canary-deployments](https://octopus.com/docs/deployment-patterns/canary-deployments)*

* [*https://istio.io/docs/](https://istio.io/docs/)*

* [*https://www.oreilly.com/library/view/implementing-devops-with/9781787120532/51b706d5-d053-4229-b242-04c17ec03d67.xhtml](https://www.oreilly.com/library/view/implementing-devops-with/9781787120532/51b706d5-d053-4229-b242-04c17ec03d67.xhtml)*

* [*https://saucelabs.com/blog/using-canary-release-pipelines-to-achieve-continuous-testing](https://saucelabs.com/blog/using-canary-release-pipelines-to-achieve-continuous-testing)*
