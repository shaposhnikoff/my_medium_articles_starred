
# Managing service mesh on Kubernetes with Istio

Photo by Jeremy Bishop on Unsplash

*by Nikita Mazur
[Containerum Platform](https://github.com/containerum/containerum)*

As the complexity of microservice applications grows, it becomes extremely difficult to track and manage interactions between services. To address this problem and make service-to-service communication simpler and more efficient, several service mesh applications exist, including Istio and linkerd. In this article we will have a look at Istio.

**But first, what is a service mesh?**

Basically, it is a dedicated infrastructure layer that ensures communication between services. Over the last year service mesh has become one of the key trends in cloud managent. It is implemented as an array of lightweight proxies that are deployed on top of applications. Service mesh software handles routing, load balancing, provides logging, telemetry, etc.

Istio was first announced in 2017, and on July 31 version 1.0 was released. It is based on Envoy proxy (L7) by Lyft, which works on network level (TCP/IP) and HTTP, supports gRPC, collects statistics and supports many Service Discovery and Load Balancing methods.

Istio is a production-ready solution that aims at solving common issues of applications with microservice architecture:

* Service discovery

* Load balancing

* High Availability

* Endpoint monitoring

* Dynamic routing

* Security

* … and more

One of the key benefits of Istio is that it can be launched ‘on top’ of an existing application — it deploys an Envoy proxy-server for each service as a sidecar-container inside the same Pod. It means you don’t have to make any change to the code of your applications.

In this tutorial we will install Istio, deploy a demo application and monitor its metrics in Grafana. Let’s install Istio first.

## **Install Istio**

Installation is pretty easy. Download the installation file for your OS:

    curl -L [https://git.io/getLatestIstio](https://git.io/getLatestIstio) | sh -
    cd istio-1.0.2
    export PATH=$PWD/bin:$PATH

Once in the Istio directory, run:

    kubectl apply -f install/kubernetes/istio-demo-auth.yaml

This will create the istio-system namespace and grant RBAC permissions. Besides, it will deploy plugins for metrics and logs, configure [mutual TLS authentication](https://istio.io/docs/concepts/security/mutual-tls.html) between Envoy sidecars, and install core Istio components:

* [Istio-Pilot](https://istio.io/docs/concepts/traffic-management/pilot.html) for service discovery and for configuring the Envoy sidecar proxies

* The [Mixer](https://istio.io/docs/concepts/policy-and-control/mixer.html) components Istio-Policy and Istio-Telemetry for usage policies and gathering telemetry data

* [Istio-Ingressgateway](https://istio.io/docs/tasks/traffic-management/ingress.html), which serves as an ingress point for external traffic

* [Istio-Citadel](https://istio.io/docs/concepts/security/mutual-tls.html%23key-management), which automates key and certificate management for Istio.

Now let’s check if all components are running:

    kubectl get service -n istio-system

![](https://cdn-images-1.medium.com/max/4292/1*7O6n6Yb16c_o0k5Hxfb5tQ.png)

And the same for pods:

    kubectl get pods -n istio-system

![](https://cdn-images-1.medium.com/max/2516/1*NUogxSY53GWU4aplTC1TZg.png)

Ok, now let’s deploy a sample application!

### **Deploy the BookInfo sample application**

To see how Istio works we will deploy [BookInfo](https://istio.io/docs/guides/bookinfo.html) application. This is a simple application made up of four services. The source code and all the other files used in this example are located at the local Istio installation’s [samples/bookinfo](https://github.com/istio/istio/tree/master/samples/bookinfo) directory.

To enable Istio to manage services, it is necessary to inject a sidecar container to a pod:

    kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/platform/kube/bookinfo.yaml)

Confirm that the application has been deployed correctly by running the following commands:

    kubectl get services

![](https://cdn-images-1.medium.com/max/2124/1*fDRi6Wu8E8V03qnFGtzD_g.png)

Check the pods:

    kubectl get pods

![](https://cdn-images-1.medium.com/max/2000/1*lXUv9NHACyLrsWoo5ajvnQ.png)

Finally, define the ingress gateway routing for the application to make it accessible from the outside:

    kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml

Check it:

    kubectl get svc -n istio-system

Now this can be important: we have created a Load Balancer type of Service. If your service provider doesn’t support Load Balancers, create a ClusterIP service instead:

    apiVersion: v1
    kind: Service
    metadata:
      labels:
        app: istio-ingressgateway
        chart: gateways-1.0.1
        heritage: Tiller
        istio: ingressgateway
        release: RELEASE-NAME
      name: istio-ingressgateway
      namespace: istio-system
    spec:
      ports:
      - name: http2
        port: 80
        protocol: TCP
        targetPort: 80
      - name: https
        port: 443 
        protocol: TCP
        targetPort: 443
      - name: tcp 
        port: 31400
        protocol: TCP
        targetPort: 31400
      - name: tcp-pilot-grpc-tls
        port: 15011
        protocol: TCP
        targetPort: 15011
      - name: tcp-citadel-grpc-tls
        port: 8060
        protocol: TCP
        targetPort: 8060
      - name: tcp-dns-tls
        port: 853
        protocol: TCP
        targetPort: 853
      - name: http2-prometheus
        port: 15030
        protocol: TCP
        targetPort: 15030
      - name: http2-grafana
        port: 15031
        protocol: TCP
        targetPort: 15031
      externalIPs:
        - 192.168.0.1 # your external IP here
      selector:
        app: istio-ingressgateway
        istio: ingressgateway
      sessionAffinity: None
      type: ClusterIP

Don’t forget to put your external IP instead of 192.168.0.1. Save as istio-svc.yaml and then run:

    kubectl create -f istio-svc.yaml

Let’s generate some load and send it to our sample app and see how Istio tracks it. For this purpose we’ll be using wrk utility. Let’s install it.

For CentOS:

    sudo yum groupinstall ‘Development Tools’
    sudo yum install -y openssl-devel git 
    git clone [https://github.com/wg/wrk.git](https://github.com/wg/wrk.git) wrk
    cd wrk
    make
    sudo cp wrk /usr/bin

For Ubuntu:

    sudo apt-get install build-essential libssl-dev git -y
    git clone https://github.com/wg/wrk.git wrk
    cd wrk
    sudo make
    # move the executable to somewhere in your PATH, ex:
    sudo cp wrk /usr/local/bin

Once installed, export your External IP address and launch wrk:

    export GATEWAY_URL=%YOUR_EXTERNAL_IP:80

    wrk -t1 -c1 -d60s [http://${GATEWAY_URL}/productpage](http://${GATEWAY_URL}/productpage)

It’s time to go to Grafana to see what’s going on.

First, find the Grafana pod:

    kubectl get po -n istio-system

![](https://cdn-images-1.medium.com/max/2324/1*JD2pD0jvvnBd8x6pCQykTg.png)

Copy the name of the pod and forward it to port 3000:

    kubectl port-forward %grafana-pod -n istio-system 3000

Open your browser: [http://127.0.0.1:3000/](http://127.0.0.1:3000/) and go to ‘Istio Mesh Dashboard’.

You should see something like this:

![](https://cdn-images-1.medium.com/max/3584/1*pQF9oJGNjdYmCjtPwAGdgQ.png)

This particular dashboard reflects the traffic that was generated as well as the global view of the services and workloads in the mesh. You can click on each particular service and see detailed stats, e.g.:

![](https://cdn-images-1.medium.com/max/5356/1*mYv9XB0Z-9tQINxPOpTkKA.png)

You can find more information about visualizing metrics in Grafana in the official [docs](https://istio.io/docs/tasks/telemetry/using-istio-dashboard/).

## Conclusion

We have just deployed Istio, a sample application and saw how to monitor it using Grafana. The article is a very basic introduction to Istio, and to learn more about it I’d suggest checking out the [docs](https://istio.io/docs/) which are really well written and easy to understand.

Do you use service mesh software in your clusters? Please, share! And don’t forget to [follow](https://twitter.com/@containerumcom) us on Twitter and [join](https://t.me/joinchat/DHMHbEb8Q3EUpb3qWDzc5A) our Telegram chat to stay tuned!

Containerum Platform is an open source project** **for managing applications in Kubernetes available on [GitHub](https://github.com/containerum/containerum). We are currently looking for community feedback, and invite everyone to test the platform! You can submit an issue, or just support the project by giving it a ⭐. Let’s make cloud management easier together!
