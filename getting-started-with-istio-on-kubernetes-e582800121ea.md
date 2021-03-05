
# Getting Started With Istio on Kubernetes

Install and configure Istio within your Kubernetes cluster

![Photo by [Sven Read](https://unsplash.com/@starburst1977?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral).](https://cdn-images-1.medium.com/max/8064/0*vNF1YOd0GFz0x609)*Photo by [Sven Read](https://unsplash.com/@starburst1977?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral).*

[Istio](https://istio.io/) is by far the most popular service mesh that integrates with [Kubernetes](https://kubernetes.io/) very well. For a [microservices](https://en.wikipedia.org/wiki/Microservices) architecture, Istio is not only useful to have but a necessity. This story is a follow-up to [How Istio Works Behind the Scenes on Kubernetes](https://medium.com/better-programming/how-istio-works-behind-the-scenes-on-kubernetes-aeb8003f2cb5). Today, let’s discuss setting up Istio in your Kubernetes cluster.

## Prerequisites

You need to have a running Kubernetes cluster to install Istio. If you are running on a cloud, a managed service like [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine) would be a perfect fit, as it allows automatic sidecar injection.

I am going to discuss how to install Istio in the cloud using GKE. However, there are only minor differences if you are running it within your self-hosted or on-premise Kubernetes cluster.

## Installing Istio

Istio is a continuously evolving project. At the time of writing, the latest stable version is Istio 1.6, and we will install that.

Make sure that you log in as a cluster-admin within your Kubernetes cluster.

### Download Istio

Download Istio and set the path to the download binaries by running the following:

    $ curl -L [https://istio.io/downloadIstio](https://istio.io/downloadIstio) | ISTIO_VERSION=1.6.1 sh -
    $ cd istio-1.6.1
    $ export PATH=$PWD/bin:$PATH

The installation YAML files are present in the install/kubernetes directory.

Istio also provides some samples that we can use to understand and experiment on the features of Istio in the samples/ directory. We will use the sample application in the rest of the series.

### Install Istio

Install Istio by applying the Istio manifests using the appropriate configuration profile.

There are six configuration profiles to choose from:

1. Default** **— This is recommended for production deployments and configures the default settings of the IstioOperator API. That enforces most rules by default, and you can customise the configuration according to your requirements.

1. Demo** **— You can use it for playing around with Istio and learning, especially when you are using Minikube or a setup that has limited resources. For running the sample application, this is the most suitable profile, and we are going to use it for the demo.

1. Minimal** **— It contains a minimum amount of features just to support traffic management.

1. Remote** **— If you are running more than one Kubernetes cluster and want to use Istio to manage a multi-cluster environment, then this is the most suitable profile. It provides you with a shared control plane to manage all your clusters from one place.

1. Empty** **— Well, this profile deploys nothing, and you can use it if you want to customise Istio and start with a base profile.

1. Separate** **— This is deprecated and not recommended, and it is used only to support legacy features.

Install Istio with the Demo profile, as it gives us most features for evaluation and training:

    $ istioctl manifest apply --set profile=demo       
    - Applying manifest for component Base...
    ✔ Finished applying manifest for component Base.
    - Applying manifest for component Pilot...
    ✔ Finished applying manifest for component Pilot.
      Waiting for resources to become ready...
    - Applying manifest for component IngressGateways...
    - Applying manifest for component EgressGateways...
    - Applying manifest for component AddonComponents...
    ✔ Finished applying manifest for component EgressGateways.
    ✔ Finished applying manifest for component IngressGateways.
    ✔ Finished applying manifest for component AddonComponents.

Label the namespaces on which you want to enable Istio to inject sidecar containers automatically. Start with the default namespace:

    $ kubectl label namespace default istio-injection=enabled
    namespace/default labeled

## Testing the Configuration

Now that you have installed Istio and configured it to inject sidecar containers automatically onto your default namespace, install the sample Book Info application and see if Istio is working.

If you have a look at the YAML file of the Book Info application, you would see there are four microservices: details, ratings, reviews, and productpage.

The reviews microservice contains three versions of pods, each labelled v1, v2, and v3. The rest of the microservices have just one version (v1).

Apply the YAML file by running the kubectl command:

    $ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
    service/details created
    serviceaccount/bookinfo-details created
    deployment.apps/details-v1 created
    service/ratings created
    serviceaccount/bookinfo-ratings created
    deployment.apps/ratings-v1 created
    service/reviews created
    serviceaccount/bookinfo-reviews created
    deployment.apps/reviews-v1 created
    deployment.apps/reviews-v2 created
    deployment.apps/reviews-v3 created
    service/productpage created
    serviceaccount/bookinfo-productpage created
    deployment.apps/productpage-v1 created

List down the services and pods:

    $ kubectl get svc
    NAME          TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
    details       ClusterIP   10.8.12.70    <none>        9080/TCP   31s
    kubernetes    ClusterIP   10.8.0.1      <none>        443/TCP    19m
    productpage   ClusterIP   10.8.15.230   <none>        9080/TCP   28s
    ratings       ClusterIP   10.8.14.21    <none>        9080/TCP   30s
    reviews       ClusterIP   10.8.12.51    <none>        9080/TCP   30s

    $ kubectl get pod
    NAME                              READY   STATUS    RESTARTS   AGE
    details-v1-74f858558f-xv2rz       2/2     Running   0          56s
    productpage-v1-76589d9fdc-dk9vn   2/2     Running   0          53s
    ratings-v1-7855f5bcb9-2dcx8       2/2     Running   0          55s
    reviews-v1-64bc5454b9-wtc8l       1/2     Running   0          54s
    reviews-v2-76c64d4bdf-ddrfn       2/2     Running   0          54s
    reviews-v3-5545c7c78f-8rmqr       2/2     Running   0          53s

Did you notice something? Nowhere within the YAML did we see the pods containing two containers, but if you look at the READY column, you see 2/2.

Why is that? We just deployed one container within the pod, but we are seeing two running in the pod.

Don’t be surprised! Istio is spinning up the Envoy sidecar container within the pod. That shows Istio is injecting sidecars automatically.

Now try to access the product page of the sample application by doing a kubectl exec on the ratings microservice container and running a curl to the product page:

    $ kubectl exec -it $(kubectl get pod -l app=ratings -o jsonpath=’{.items[0].metadata.name}’) -c ratings -- curl productpage:9080/productpage

### Expose the Book Info application externally

For this, we would need to create a Book Info Ingress gateway to associate the app with the Istio Ingress gateway.

Istio deploys the Istio Ingress Gateway when you first install it and exposes the service on a LoadBalancer by default. If your cluster does not support Load Balancers, you can use the NodePort of the Service and use the NodeIP:NodePort combination to access your applications.

To determine the LoadBalancer and the NodePort, run the following:

    $ kubectl -n istio-system get service istio-ingressgateway
    NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
    istio-ingressgateway LoadBalancer 10.8.14.188 34.71.3.244 15020:31344/TCP,80:30347/TCP,443:31784/TCP,15029:30240/TCP,15030:31113/TCP,15031:30358/TCP,15032:30211/TCP,31400:32315/TCP,15443:32359/TCP 17m

As you see, Istio has exposed several ports on your load balancer. In this example, we need to run our application on port 80, and therefore we can either use the LoadBalancerIP:80 or NodeIP:NodePort combination of the NodePort mapped to port 80.

Make sure you have firewalls opened to access your application through the LoadBalancer or NodePort. If you are using GKE, for example, then run the following:

    $ gcloud compute firewall-rules create allow-gateway-http --allow tcp:80
    Creating firewall...⠹Created [[https://www.googleapis.com/compute/v1/projects/playground-s-11-5eaba8/global/firewalls/allow-gateway-http](https://www.googleapis.com/compute/v1/projects/playground-s-11-5eaba8/global/firewalls/allow-gateway-http)].
    Creating firewall...done.
    NAME                NETWORK  DIRECTION  PRIORITY  ALLOW   DENY  DISABLED
    allow-gateway-http  default  INGRESS    1000      tcp:80        False

Istio provides the bookinfo-gateway.yaml in the samples/bookinfo/networking/ directory for associating the Book Info application with the Ingress Gateway:

<iframe src="https://medium.com/media/52ee310da60f62018368c71853fbd2e2" frameborder=0></iframe>

If you look at the YAML file, you would see that it defines a Gateway on port 80 and a VirtualService running on the bookinfo-gateway. The VirtualService matches all hosts having /productpage, /static, /login, /logout, and /api/v1/products paths and routes them to the productpage service running on port 9080.

As Istio is Kubernetes-aware, you can simply use kubectl commands to create Istio objects:

    $ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
    gateway.networking.istio.io/bookinfo-gateway created
    virtualservice.networking.istio.io/bookinfo created

Now, try to access the application using the Load Balancer IP on a browser:

![Product page with red stars](https://cdn-images-1.medium.com/max/3840/1*yqoP8jC56bLsEMhmTiV15A.png)*Product page with red stars*

Refresh it, and you will see a different page come up:

![Product page with black stars](https://cdn-images-1.medium.com/max/3840/1*bj9vAi1SR8PRPTncJJHFmQ.png)*Product page with black stars*

Refresh it again, and you will see another page come up:

![Product page with no stars](https://cdn-images-1.medium.com/max/3840/1*t43DbQ0Ywa8h8MP9VovRaA.png)*Product page with no stars*

What is happening?

Well, when we deployed the application, we implemented three versions of the ratings microservice, which it displays in a round-robin fashion.

That is the default routing policy of Istio, and this shows that Istio is installed and working correctly.

## Access the Dashboard

Istio provides numerous optional dashboards with which you can hook your mesh. The demo installation comes with the Kiali dashboard that provides you with a topology of your mesh and how traffic flows through it.

To access the dashboard, you can use istioctl to open a proxy:

    $ istioctl dashboard kiali
    [http://localhost:20001/kiali](http://localhost:20001/kiali)

Then use the proxy URL to open the Kiali dashboard on your browser and use the default username admin and default password admin to log in.

![Kiali overview](https://cdn-images-1.medium.com/max/3840/1*tPDfgPZ_cl02S0Z4phb9tA.png)*Kiali overview*

Go to Graph and select the default namespace to view the mesh topology:

![Kiali graph](https://cdn-images-1.medium.com/max/3840/1*rt7xgTnvveL6VeaCfvZtrQ.png)*Kiali graph*

## Conclusion

Thanks for reading through! I hope you enjoyed the article. In the next article [How to Manage Traffic Using Istio on Kubernetes](https://medium.com/better-programming/how-to-manage-traffic-using-istio-on-kubernetes-cd4b96e00b57), I will discuss traffic management through Istio with a hands-on demonstration, so see you in the next part!
