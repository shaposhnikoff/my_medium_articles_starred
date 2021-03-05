
# Istio Gateway

Istio Gateway

*From [Istio in Action](http://bit.ly/2GsekPn) by* *Christian Posta*

___________________________________________________________________

Take 37% off [*Istio in Action](http://bit.ly/2GsekPn)*. Just enter **fccposta** into the discount code box at checkout at [manning.com](http://bit.ly/2Gw62q3).
___________________________________________________________________

This article explores the Istio gateway.

### **Istio Gateway**

Istio has a concept of an ingress Gateway which plays the role of the network-ingress point and it’s responsible for guarding and controlling access to the cluster from traffic that originates outside of the cluster. Additionally, Istio’s Gateway also plays the role of load balancing and virtual-host routing.

![Figure 1 Istio Gateway plays the role of network ingress and uses Envoy Proxy to do the routing and load balancing](https://cdn-images-1.medium.com/max/2000/0*wg4G_iDLx2OruDWl.png)*Figure 1 Istio Gateway plays the role of network ingress and uses Envoy Proxy to do the routing and load balancing*

Figure 1 Istio Gateway plays the role of network ingress and uses Envoy Proxy to do the routing and load balancing

In Figure 1 we see that, by default, Istio uses an Envoy proxy as the ingress

proxy. Envoy is a capable service-to-service proxy, but it can also be used to load balance and route proxy traffic from outside the service mesh to services running inside of it. All of the key features of Envoy are also available in the ingress gateway.

Let’s take a closer look at how Istio uses Envoy to implement an ingress gateway. A listing of components that make up the control plane and any additional components that support the control plane can be seen below.

![Figure 2 Review of key components; some comprise the Istio control plane while others support it](https://cdn-images-1.medium.com/max/2000/0*p0JZL7RrHbruNZUm.png)*Figure 2 Review of key components; some comprise the Istio control plane while others support it*

If we do a listing of the Kubernetes pods in the istio-system namespace (where the control plane’s installed), we should see the istio-ingressgateway component:

Listing 1 List the running components installed into Kubernetes

    $  kubectl get pod -n istio-system  NAME                                        READY     STATUS    RESTARTS  grafana-7ff77c54b-hqvxt                     1/1       Running   0  istio-citadel-676785b9df-hsm9x              1/1       Running   0  istio-egressgateway-856c4f776c-552lk        1/1       Running   0  istio-ingressgateway-69cc4b4d7-z5kvn        1/1       Running   0  istio-pilot-6d8fc46b4c-p5l5v                2/2       Running   0  istio-policy-865c5fd87c-7jkq4               2/2       Running   0  istio-sidecar-injector-5597c8564-28zlm      1/1       Running   0  istio-statsd-prom-bridge-6dbb7dcc7f-xt8w6   1/1       Running   0  istio-telemetry-779d6689fc-4qj8v            2/2       Running   0  istio-tracing-748b4f77c4-nhgcj              1/1       Running   0  prometheus-586d95b8d9-nk7d5                 1/1       Running   0  servicegraph-7875b75b4f-2zzr4               1/1       Running   0

You can see in the “READY” column for the ingress-gateway-xxx component that it has 1/1 containers ready, and some of the other ones have 2/2. Some of the other components have a service proxy injected alongside them. This allows those components in the control plane to take advantage of the service-mesh capabilities. The istio-ingressgateway component has only a single Envoy proxy container deployed, and we see 1/1.

**NOTE: **In the above listing, right next to the istio-ingressgateway pod, you may notice the istio-egressgateway component. This component is responsible for routing traffic *out* of the cluster.

If you’d like to verify that the Istio service proxy is indeed running in the Istio ingress gateway, you can run something like this:

    $  INGRESS_POD=$(kubectl get pod -n istio-system \  
    | grep ingressgateway | cut -d ' ' -f 1)  
    $  kubectl -n istio-system exec $INGRESS_POD  ps aux

We should see a process listing as the output showing the Istio service proxy command line with both the discovery-agent and the envoy processes.

At this point, although we’ve got a running Envoy playing the role of the Istio Gateway, we’ve no configuration or rules about what traffic we should let into the cluster. We can verify that we don’t have anything by running the following commands:

    $  istioctl -n istio-system proxy-config listener $INGRESS_POD  
    Error: no listeners found  
    $  istioctl -n istio-system proxy-config route $INGRESS_POD  
    Error: config dump has no route dump

To configure Istio’s Gateway to allow traffic into the cluster and through the service mesh, we’ll start by exploring two concepts: Gateway and VirtualService. Both are fundamental, in general, to getting traffic to flow in Istio, but we’ll look at them only within the context of allowing traffic into the cluster.

### **Specifying Gateway resources**

To configure a Gateway in Istio, we use the Gateway resource and specify which ports we wish to open on the Gateway, and what virtual hosts to allow for those ports. The example Gateway we’ll explore is quite simple and exposes an HTTP port on port 80 that accepts traffic destined for virtual host apiserver.istioinaction.io:

    apiVersion: networking.istio.io/v1alpha3
    kind: Gateway
    metadata:
      name: coolstore-gateway
    spec:
      selector:
        istio: ingressgateway
      servers:
      - port:
          number: 80
          name: http
          protocol: HTTP
        hosts:
        - "apiserver.istioinaction.io"

This Gateway definition is intended for the istio-ingressgateway which was created when we set up Istio initially, but we could have used our own definition of Gateway. We can define to which gateway the configuration applies by using the labels in the selector section of the Gateway configuration. In this case, we’re selecting the gateway implementation with the label istio: ingressgateway which matches the default istio-ingressgateway. The Gateway for istio-ingressgateway is an instance of Istio is service proxy (Envoy), and its Kubernetes deployment configuration looks something like this:

    containers:
    - name: ingressgateway
      image: "gcr.io/istio-release/proxyv2:1.0.0"
      imagePullPolicy: IfNotPresent
      ports:
        - containerPort: 80
        - containerPort: 443
      args:
      - proxy
      - router
      - -v
      - "2"
      - --serviceCluster
      - custom-ingressgateway
      - --proxyAdminPort
      - "15000"
      - --discoveryAddress
      - istio-pilot.istio-system:8080

Our Gateway resource configures Envoy to listen on port 80 and expect HTTP traffic. Let’s create that resource and see what it does. From the root of the source

code you should see achapter-files/chapter4/coolstore-gw.yaml file and you can create the resource like this:

    $  kubectl create -f chapter-files/chapter4/coolstore-gw.yaml

Let’s see whether our settings took effect.

    $  istioctl proxy-config listener $INGRESS_POD  -n istio-system  
    ADDRESS     PORT     TYPE  
    0.0.0.0     80       HTTP

We’ve exposed our HTTP port correctly! If we take a look at the routes for virtual services, we see that the Gateway doesn’t have any at the moment:

    $  istioctl proxy-config route $INGRESS_POD -o json  -n istio-system
    [
        {
            "name": "http.80",
            "virtualHosts": [
                {
                    "name": "blackhole:80",
                    "domains": [
                        "*"
                    ],
                    "routes": [
                        {
                            "match": {
                                "prefix": "/"
                            },
                            "directResponse": {
                                "status": 404
                            },
                            "perFilterConfig": {
                                "mixer": {}
                            }
                        }
                    ]
                }
            ],
            "validateClusters": false
        }
    ]

We see our listener is bound to a “blackhole” default route that routes everything to HTTP 404. In the next section, we’ll take a look at setting up a virtual host for routing traffic from port 80 to a service within the service mesh.

Before we go to the next section there’s an important last point to be made here.

The pod running the gateway, whether it’s the defaultistio-ingressgateway or your own custom gateway, needs to be able to listen on a port or IP which is exposed outside the cluster. For example, on our local minikube that we’re using for these examples, the ingress gateway is listening on a NodePort A NodePort uses a real port on one of the Kubernetes clusters’ nodes. On minikube there’s only one node and it’s the VM that minikube runs on. If you’re deploying on a cloud service like GKE, you’ll want to make sure you use a LoadBalancer that gets an externally routable IP address. More information can be found at [https://istio.io/docs/tasks/traffic-management/ingress/](https://istio.io/docs/tasks/traffic-management/ingress/).

### **Gateway routing with Virtual Services**

This far, all we’ve done is configure the Istio Gateway to expose a specific port, expect a specific protocol on that port, and define specific hosts to serve from the port/protocol pair. When traffic comes into the gateway, we need a way to get it to a specific service within the service mesh, and to do that we’ll use the VirtualService resource. In Istio, a VirtualService resource defines how a client talks to a specific service through its fully qualified domain name, which versions of a service are available, and other routing properties (like retries and request timeouts). It’s sufficient to know that VirtualService allows us to route traffic from the ingress gateway to a specific service.

An example of a VirtualService that routes traffic for the virtual host apigateway.istioinaction.io to services deployed in our service mesh looks like this:

    **apiVersion**: networking.istio.io/v1alpha3
    **kind**: VirtualService
    **metadata**:
      **name**: apigateway-vs-from-gw
    **spec**:
      **hosts**:
      - "apiserver.istioinaction.io"
      **gateways**:
      - coolstore-gateway
      **http**:
      - **route**:
        - **destination**:
            **host**: apigateway
            **port**:
              **number**: 8080

With this VirtualService resource, we define what to do with traffic when it comes into the gateway. In this case, as you can see with spec.gateways field, these traffic rules apply only to traffic coming from the coolstore-gateway gateway definition which we created in the previous section. Additionally, we’re specifying a virtual host of apiserver.istioinaction.io for which traffic must be destined for these rules to match. An example of matching this rule is a client querying [http://apiserver.istioinaction.io](http://apiserver.istioinaction.io/) which resolves to an IP which the Istio gateway is listening on. Additionally, a client could explicitly set the Host header in the HTTP request to be apiserver.istioinaction.io which we’ll show through an example.

First, let’s create this VirtualService and explore how Istio exposes this on the gateway:

    $  kubectl create -f chapter-files/chapter4/coolstore-vs.yaml

After a few moments (it may take a few for the configuration to sync), we can re-run our commands to list the listeners and routes :

    $  istioctl proxy-config listener \
    istio-ingressgateway-5ff9b6d9cb-pnrq9  -n istio-system
    ADDRESS     PORT     TYPE
    0.0.0.0     80       HTTP
    $  istioctl proxy-config route \ istio-ingressgateway-5ff9b6d9cb-pnrq9 -o json  -n istio-system
    [
      {
        "name": "http.80",
        "virtualHosts": [
          {
          "name": "apigateway-vs-from-gw:80",
          "domains": [
              "apiserver.istioinaction.io"         
          ],
          "routes": [
            {
              "match": {             "prefix": "/"
              },
               "route": {        
                "cluster": "outbound|8080||apigateway.istioinaction.svc.cluster.local",
                "timeout": "0.000s"
              }
            }
          ]
          }
        ]
      }
    ]

The output for the route should look similar to the previous listing, although

it may contain other attributes and information. The critical part is we can see how defining a VirtualService created an Envoy route in our Istio Gateway which routes traffic matching domain apiserver.istioinaction.io to service apiserver in our service mesh.

This configuration assumes you’ve installed the apigateway and catalog services with Istio’s service proxy injected alongside the services, as is shown below:

    $  kubectl create -f <(istioctl kube-inject \  
    -f install/catalog-service/catalog-all.yaml)  
    $  kubectl create -f <(istioctl kube-inject \  
    -f install/apigateway-service/apigateway-all.yaml)

Once all the pods are ready, you should see something like this:

    $  kubectl get pod NAME                         READY     STATUS    RESTARTS   AGE apigateway-bd97b9bb9-q9g46   2/2       Running   18         19d catalog-786894888c-8lbk4     2/2       Running   8          6d

Verify that your Gateway and VirtualService are installed correctly:

    $  kubectl get gateway NAME                CREATED AT coolstore-gateway   2h  $  kubectl get virtualservice NAME                    CREATED AT apigateway-vs-from-gw   11m

Now, let’s try to call the gateway and verify the traffic is allowed into the cluster:

    $  HTTP_HOST=$(minikube ip) $  HTTP_PORT=$(kubectl -n istio-system get service istio-ingressgateway \ -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')  
    $  URL=$HTTP_HOST:$HTTP_PORT  
    $  curl $URL/api/products

We should see no response. Why is that? If we take a closer look at the call by printing the headers, we should see that the Host header we sent in *isn’t* a host which the gateway recognizes.

    $  curl -v $URL/api/products *   Trying 192.168.64.27...
    TCP_NODELAY set
    Connected to 192.168.64.27 (192.168.64.27) port 31380 (#0)
    > GET /api/products HTTP/1.1
    > Host: 192.168.64.27:31380
    > User-Agent: curl/7.61.0
    > Accept: */*
    > 
    < HTTP/1.1 404 Not Found    
    < date: Tue, 21 Aug 2018 16:08:28 GMT
    < server: envoy
    < content-length: 0
    < 
    Connection #0 to host 192.168.64.27 left intact

The Istio Gateway, nor any of the routing rules we declared in VirtualService knows anything about Host: 192.168.64.27:31380 but it knows about virtual host apiserver.istioinaction.io. Let’s override the Host header on our command line and then the call should work:

    $  curl $URL/api/products -H "Host: apiserver.istioinaction.io"

Now you should see a successful response.

### **Overall view of traffic flow**

In the previous subsections, we got hands on with the Gateway and VirtualService resources from Istio. The Gateway resource defines our ports, protocols, and virtual hosts that we wish to listen for at the edge of our service mesh cluster. The VirtualService resources define where traffic should go once it’s allowed in at the edge. In Figure 3 we see the full end-to-end flow:

![Figure 3 Flow of traffic from client outside of service mesh/cluster to services inside the service mesh through the ingress gateway](https://cdn-images-1.medium.com/max/2000/0*2poDW9YTWXah6Uno.png)*Figure 3 Flow of traffic from client outside of service mesh/cluster to services inside the service mesh through the ingress gateway*

### **Istio Gateway vs Kubernetes Ingress**

When running on Kubernetes, you may ask “why doesn’t Istio use the Kubernetes Ingress resource to specify ingress?” In some of Istio’s early releases there was support for using Kubernetes Ingress, but there are significant drawbacks with the Kubernetes Ingress specification.

The first issue is that the Kubernetes Ingress is a simple specification geared toward HTTP workloads. Each implementation of Kubernetes Ingress (like NGINX, Heptio Contour, etc) is geared toward HTTP traffic. In fact, Ingress specification only considers port 80 and port 443 as ingress points. This severely limits the types of traffic a cluster operator can allow into the service mesh. For example, if you’ve Kafka or NATS.io workloads, you may wish to expose direct TCP connections to these message brokers. Kubernetes Ingress doesn’t allow for that.

Second, the Kubernetes Ingress resource is severely underspecified. It lacks a common way to specify complex traffic routing rules, traffic splitting, or things like traffic shadowing. The lack of specification in this area causes each vendor to re-imagine how best to implement configurations for each type of Ingress implementation (HAProxy, Nginx, etc).

Lastly, because things are underspecified, the way most vendors choose to expose configuration’s through bespoke annotations on deployments. The annotations between vendors varied and aren’t portable, and if Istio continues this trend there will be many more annotations to account for all the power of Envoy as an edge gateway.

Ultimately, Istio decided on a clean slate for building ingress patterns and specifically separating out the layer 4 (transport) and layer 5 (session) properties from the layer 7 (application) routing concerns. Istio Gateway handles the L4 and L5 concerns, and VirtualService handles the L7 concerns.

That’s all for this article. If you want to learn more about the book, check it out on liveBook [here](http://bit.ly/2GsdUsh) and see this [slide deck](http://bit.ly/2Gw5W1F).

**About the author:**
**Christian Posta** is a Chief Architect of cloud applications at Red Hat, an author, a blogger, a speaker, and an open-source enthusiast and committer. He also puts his expertise to good use helping companies deploy their enterprise systems and microservices.

*Originally published at [freecontent.manning.com](https://freecontent.manning.com/istio-gateway/).*
