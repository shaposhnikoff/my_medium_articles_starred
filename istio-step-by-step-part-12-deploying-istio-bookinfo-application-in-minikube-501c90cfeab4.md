
# Istio step-by-step Part 12 — Deploying Istio BookInfo application in Minikube

Hello everyone, welcome back with Istio step-by-step tutorial series. Upon many requests, I thought to write about BookInfo application, to demonstrate the key features of Istio. You can find more information on Bookinfo page.

![Photo credits: [https://www.pexels.com/photo/black-and-white-browsing-business-coffee-265152/](https://www.pexels.com/photo/black-and-white-browsing-business-coffee-265152/)](https://cdn-images-1.medium.com/max/6512/1*VsCKCYBf7pSJDNQzhB_WaA.jpeg)*Photo credits: [https://www.pexels.com/photo/black-and-white-browsing-business-coffee-265152/](https://www.pexels.com/photo/black-and-white-browsing-business-coffee-265152/)*

Before starting you need to,

1. Start minikube

1. Install Istio with demo profile

1. Check for all the pods are up and running

![](https://cdn-images-1.medium.com/max/3412/1*k-bM-gAWO-dEkIQOnkkQfA.png)

## Introduction to Bookinfo Application

Bookinfo is an application, which holds information about a book such as details, reviews and ratings. This application is consist of 4 microservices as;

1. productpage

1. details

1. reviews

1. ratings

productpage microservice calls details and reviews microservices to populate the page. reviews microservice contains the reviews regarding the microservice, while ratings microservice carries information about ratings of the book.

Reviews microservice is in 3 versions.

* version 01 (v1) — No stars

* version 02 (v2) — Black stars

* version 03 (v3) — Red stars

This application is written in different programming languages.

1. productpage — python

1. details — Ruby

1. reviews — Java

1. ratings — node JS

This verifies that microservices work individually, so they do not have issues with the programming language they have been written.

### Summary

The following diagram shows the Bookinfo application **without** Istio.

![Photo credits: [https://istio.io/docs/examples/bookinfo/noistio.svg](https://istio.io/docs/examples/bookinfo/noistio.svg)](https://cdn-images-1.medium.com/max/2676/1*vkshHfol2pozqIlMJcN-HQ.png)*Photo credits: [https://istio.io/docs/examples/bookinfo/noistio.svg](https://istio.io/docs/examples/bookinfo/noistio.svg)*

### Deploy application

To deploy this application in Istio, we need to enable the sidecar injection. No need to rewrite the application. To enable injection, execute the command,

    kubectl label namespace default istio-injection=enabled

![](https://cdn-images-1.medium.com/max/3108/1*Tr0MilVcRB3M085SVbxJ7w.png)
> We deploy the application under the namespace *default*.

Now deploy the application.

    kubectl apply -f [samples/bookinfo/platform/kube/bookinfo.yaml](https://raw.githubusercontent.com/istio/istio/release-1.4/samples/bookinfo/platform/kube/bookinfo.yaml)

![](https://cdn-images-1.medium.com/max/3624/1*x6SRu3kiiUlsRtLv9nbpYg.png)

We have successfully deployed all the services. The following diagram will demonstrate how the application looks like after deploying it with Istio

![Photo credits: [https://istio.io/docs/examples/bookinfo/withistio.svg](https://istio.io/docs/examples/bookinfo/withistio.svg)](https://cdn-images-1.medium.com/max/3108/1*roRUQWThNb2nokVw9oQZZw.png)*Photo credits: [https://istio.io/docs/examples/bookinfo/withistio.svg](https://istio.io/docs/examples/bookinfo/withistio.svg)*

To verify you can check for the services and pods. It will take a while to run the pods.

    kubectl get services
    kubectl get pods

![](https://cdn-images-1.medium.com/max/2336/1*ko9F8XMzbxgbbGzv43Kqhw.png)

To verify the application is up and running, you can execute the command,

    kubectl exec -it $(kubectl get pod -l app=ratings -o jsonpath=’{.items[0].metadata.name}’) -c ratings — curl productpage:9080/productpage | grep -o “<title>.*</title>”

![](https://cdn-images-1.medium.com/max/6692/1*GDgrTGdsE7SpEu8Kx6-T4Q.png)

### Determine ingress IP and ingress port

This part is pretty much relatable to my third article ([deploying-an-application-with-istio-in-kubernetes](https://medium.com/faun/istio-step-by-step-part-03-deploying-an-application-with-istio-in-kubernetes-d2b1de64fb6b)) and I have done a separate article about [Istio ingress traffic routing](https://medium.com/faun/istio-step-by-step-part-04-traffic-routing-path-of-istio-service-mesh-part-a-ingress-routing-28e03cdaa048). So, I would like to recommend you to check for it for more details about how Istio ingress routing happens with gateway and virtualservice YAMLs.

As a short description, Istio services need gateway and virtualservice to be invoked by outside. You can apply the gateway to the application with the command and verify the gateway by the commands,

    kubectl apply -f [samples/bookinfo/networking/bookinfo-gateway.yaml](https://raw.githubusercontent.com/istio/istio/release-1.4/samples/bookinfo/networking/bookinfo-gateway.yaml)
    kubectl get gateway

![](https://cdn-images-1.medium.com/max/3776/1*nQBzBtWYhIIwxiQQSYaE3Q.png)

To determine the ingress port, first, you need to find whether the application has a load balancer.

    kubectl get svc istio-ingressgateway -n istio-system

![](https://cdn-images-1.medium.com/max/6344/1*9qozzO4kMvRl9BHtsL04Bw.png)

If the EXTERNAL-IP is set, that means your application has an external load balancer. But, if the IP is in <pending> or <none> status, that means, your application does not have an external load balancer. In that case, you can access the application with a node port.

**Ingress IP and host when load balancer exists**

    export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath=’{.status.loadBalancer.ingress[0].ip}’)

    export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath=’{.spec.ports[?(@.name==”http2")].port}’)

    export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath=’{.spec.ports[?(@.name==”https”)].port}’)

**Ingress IP and host when load balancer does not exist (when using Node Port)**

    export INGRESS_HOST=$(minikube ip)

    export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')

    export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
> Refer to [determining-the-ingress-ip-and-ports](https://istio.io/docs/tasks/traffic-management/ingress/ingress-control/#determining-the-ingress-ip-and-ports) for more information.

Now set the GATEWAY_URL.

    export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT

![](https://cdn-images-1.medium.com/max/6012/1*8g12a-S1ezHbmrlo1zngqw.png)

You can verify the application is running by executing

    curl -s [http://${GATEWAY_URL}/productpage](http://${GATEWAY_URL}/productpage) | grep -o “<title>.*</title>”

![](https://cdn-images-1.medium.com/max/3952/1*d0xbWCmxf7uA7I7chE1iug.png)

Visualize application

Get the GATEWAY_URL

    echo http://${GATEWAY_URL}/productpage

Open an incognito window in your browser and paste the URL. Try refreshing the tab a few times. You will see the **reviews stars** change.

![Reviews V1](https://cdn-images-1.medium.com/max/6720/1*2IiDOaab2FhngFpsoPxelg.png)*Reviews V1*

![Reviews V2](https://cdn-images-1.medium.com/max/6720/1*bFDKBh8pR9cS9ZCkhWoniA.png)*Reviews V2*

![Reviews V3](https://cdn-images-1.medium.com/max/6720/1*L-M-K0BGj6biBtQMiIilcQ.png)*Reviews V3*

For until this step we have completed a major part of the application. As we saw, when we refresh the page we get the reviews different version. So, we need to specify the real destination for our application. We need to define the available versions, called subsets in destination rules. Destination rules to configure what happens to traffic for that destination. These are applied after virtualservice.
> Read [https://istio.io/docs/concepts/traffic-management/#destination-rules](https://istio.io/docs/concepts/traffic-management/#destination-rules) for more information on destination rules.

So, for this application, we will apply the destinationrules.

* If you do not using MTLS use the command;

    kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml

* If you use MTLS use the command;

    kubectl apply -f [samples/bookinfo/networking/destination-rule-all-mtls.yaml](https://raw.githubusercontent.com/istio/istio/release-1.4/samples/bookinfo/networking/destination-rule-all-mtls.yaml)

![](https://cdn-images-1.medium.com/max/3916/1*RmVbPHPor2WvyfzikOtIjg.png)

This is all for BookInfo application. In this tutorial,

* we deployed an application with few microservices written in different programming languages.

* applied the gateway to determine the ingress port and host.

* visualised the application

Let’s get back with another tutorial on how to do the traffic routing between versions in Istio.

### References

1. [https://istio.io/docs/examples/bookinfo/](https://istio.io/docs/examples/bookinfo/)

1. [https://istio.io/docs/tasks/traffic-management/ingress/ingress-control/#determining-the-ingress-ip-and-ports](https://istio.io/docs/tasks/traffic-management/ingress/ingress-control/#determining-the-ingress-ip-and-ports)

1. [https://istio.io/docs/concepts/traffic-management/#destination-rules](https://istio.io/docs/concepts/traffic-management/#destination-rules)

<< [previous article](https://medium.com/faun/istio-step-by-step-part-11-customize-istio-installation-4d6af88b5fb7)

[next article](https://medium.com/@nethminiromina/istio-step-by-step-part-13-istio-traffic-management-istio-core-features-e513bfd66fb4) >>

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
