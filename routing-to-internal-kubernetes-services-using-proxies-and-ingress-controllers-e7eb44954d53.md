Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m49[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m33[39m, end: [33m49[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m35[39m, end: [33m48[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m28[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m68[39m, end: [33m74[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m28[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m51[39m, end: [33m66[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m139[39m, end: [33m145[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m167[39m, end: [33m171[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m33[39m, end: [33m51[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m71[39m, end: [33m77[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m45[39m, end: [33m54[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m126[39m, end: [33m144[39m }

# Routing to internal Kubernetes services using proxies and Ingress controllers

I dream about Ingress controllers a lot these days. What started off as a simple excursion into the whys and wherefores of internal path routing in Kubernetes quickly mushroomed into a slightly complex implementation that we are planning on using in a Production and Staging environment fairly soon.

To put it briefly, the problems we were trying to solve were:

* How do we route requests based on **domain names** and **paths** to specific **services/pods**?

* How do we route traffic **internally** from our existing **AWS** infrastructure to our **GCP** based Kubernetes cluster, over a **VPN connection**.

* We wanted to use **Internal **load balancers for the aforementioned routing. Both **AWS** and **GCP** provide internal load balancers, however it’s not currently possible to route requests to a GCP internal load balancer from outside the GCP network, even if you are connected over the VPN. So we needed to come up with an alternative approach.

Before I answer those questions, let’s take a quick look at some of the core concepts.

## So what are Ingresses and what is an Ingress controller, really?

An **Ingress **rule simply defines how a service should be accessed based on a few criteria, such as **hostname** and **path**. An **Ingress controller **forwards traffic based on these criteria to **Services**, which in turn forward onto the **Pods** in the backend, where all the magic happens.

This simple diagram will make it easier to understand:

![**Figure 1**: How Ingress controllers route hostnames / paths to backend **Services**](https://cdn-images-1.medium.com/max/2000/0*mHi57khsfKiQpezp.)***Figure 1**: How Ingress controllers route hostnames / paths to backend **Services***

In this example, any requests that hit the **Ingress controller** with a **Hostname **of **myapp.example.com **are forwarded onto the **MyApp **service, while requests with a **Hostname** of **foo.bar.com** and a **path **of “**/content**” get sent to the **Foo** service instead. Simply put, an Ingress controller is a routing mechanism. It provides **SSL Termination, URL based routing **as well as **Virtual Host **based routing.

To solve the problem of **inter-cloud communication**, we setup a VPN connection between AWS and GCP. However, we quickly discovered that Google doesn’t allow traffic ***ingressing*** from a VPN to hit an **internal **load balancer (**ILB**). We wanted to use internal load balancers because, understandably so, several of our services are internal only and shouldn’t be externally accessible over the Internet.

![**Figure 2**: Connections to a **ILB** inside GCP over a VPN connection are **not** allowed](https://cdn-images-1.medium.com/max/2000/0*-9HspF4GLhg61cE_.)***Figure 2**: Connections to a **ILB** inside GCP over a VPN connection are **not** allowed*

We decided to use two things to solve this problem: the **Nginx Ingress Controller **plus our own **Nginx Proxy** on top.

The **Nginx Ingress Controller **was setup as a **NodePort **service on port **31001** for **HTTP** and **32001** for **HTTPS** traffic. A **NodePort** service makes itself available on it’s specified port on every **Node **in the Kubernetes cluster. This allowed us to access it from our own **Nginx Proxy**.

Our **Nginx Proxy **is a simple GCP VM instance running **Nginx **on top of a **Linux OS** running within the same VPC that the Kubernetes cluster is using.

Here’s what this setup looks like:

![**Figure 3: **We replaced the ILB with our own **Nginx Proxy** and added an **Nginx** based **Ingress controller** to the cluster nodes on ports **31001** (**HTTP**) / **32001** (**HTTPS**)](https://cdn-images-1.medium.com/max/2000/0*c6TP5JDaSV_KNSGx.)***Figure 3: **We replaced the ILB with our own **Nginx Proxy** and added an **Nginx** based **Ingress controller** to the cluster nodes on ports **31001** (**HTTP**) / **32001** (**HTTPS**)*

## An actual Ingress Controller + Proxy setup

Let’s work through an actual example so you can see how this works in practice.

I’m going to use a Google Cloud hosted cluster (GKE) to demonstrate how to get this working in practice.

The GitHub repo for the Nginx Controller is here:

[https://github.com/kubernetes/ingress/tree/master/examples/deployment/nginx](https://github.com/kubernetes/ingress/tree/master/examples/deployment/nginx)

**UPDATE:** It’s been a year since I wrote this and the above URL has changed. You can find a newer version of the deployment at the following URL. Note however that I have not had the time to re-test everything here with this newer version of the Deployment:
[https://github.com/kubernetes/ingress-nginx/blob/master/docs/examples/static-ip/nginx-ingress-controller.yaml](https://github.com/kubernetes/ingress-nginx/blob/master/docs/examples/static-ip/nginx-ingress-controller.yaml)

### Install the default backend service

Let’s install the **default-backend** service first. Note that if you already have a default backend running in your namespace, you might want to try this in a different namespace altogether.

    $ kubectl create -f default-backend.yaml
    deployment “default-http-backend” created

### Install the Nginx Controller itself

$ kubectl create -f nginx-ingress-controller.yaml

We should confirm that the **deployment**, **pods** and **services** have been created:

    $ kc get deploy,po,svc
    NAME                              DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    deploy/default-http-backend       1         1         1            1           10m
    deploy/nginx-ingress-controller   2         2         2            2           10m

    NAME                                           READY     STATUS    RESTARTS   AGE
    po/default-http-backend-726995137-9r2pp        1/1       Running   0          10m
    po/nginx-ingress-controller-1575676611-fxkg8   1/1       Running   0          10m

    NAME                       CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
    svc/default-http-backend   10.4.250.245   <none>        80/TCP    10m

### Create a Service for the Nginx Controller

We are going to run the Nginx Controller as a **NodePort **service on port **31001** for HTTP and **32001** for HTTPS. Here is our **Service** definition:

    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-lb
    spec:
      type: NodePort
      ports:
      - name: http
        port: 80
        nodePort: 31001
        protocol: TCP
        targetPort: 80
      - name: https
        port: 443
        nodePort: 32001
        protocol: TCP
        targetPort: 443
      selector:
        k8s-app: nginx-ingress-controller

Let’s save this in a file called ing-service.yaml and apply it:

    $ kubectl create -f ing-service.yaml
    service “nginx-lb” created

Describe the service to make sure it’s been created properly:

    $ kubectl describe svc nginx-lb
    Name:             nginx-lb
    Namespace:        ingress-test
    Labels:           <none>
    Annotations:      <none>
    Selector:         k8s-app=nginx-ingress-controller
    Type:             NodePort
    IP:               10.4.243.159
    Port:             http 80/TCP
    NodePort:         http 31001/TCP
    Endpoints:        10.4.3.15:80,10.4.5.28:80
    Port:             https 443/TCP
    NodePort:         https 32001/TCP
    Endpoints:        10.4.3.15:443,10.4.5.28:443
    Session Affinity: None
    Events:           <none>

### Deploy a sample application

Now that we have successfully deployed the **Nginx Controller** and it’s Service, let’s deploy a sample application to test it. We are going to use the **echoheaders** application that you can find here:

[https://github.com/kubernetes/contrib/blob/master/ingress/echoheaders/echo-app.yaml](https://github.com/kubernetes/contrib/blob/master/ingress/echoheaders/echo-app.yaml)

    $ kc create -f echo-app.yaml
    service “echoheaders” created
    replicationcontroller “echoheaders” created

Let’s make sure the **deployment**, **pod** and **service** have been created and are in a **Running** state:

    $ kc get all
    NAME                                           READY     STATUS    RESTARTS   AGE
    ...
    po/echoheaders-r30sc                           1/1       Running   0          2m
    ...

    NAME             DESIRED   CURRENT   READY     AGE
    rc/echoheaders   1         1         1         2m

    NAME                       CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
    ...
    svc/echoheaders            10.4.240.210   <nodes>       80:30234/TCP                 2m

### Create the Ingress resource

The last thing we have to do, to tie all this together and to make the actual “**routing**” work is to add an Ingress resource. An Ingress resource is what describes the “**hostnames**”, “**paths**”, and their “**target**” services within the cluster.

Note that the “**nginx**” annotation used below is really important. This is what the **Nginx **controller is looking for. When it sees an Ingress resource with that annotation, it picks it up, and creates the appropriate routing rules. Without this annotation, the resource will be ignored by the Nginx controller.

    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: echomap
      annotations: {
        'kubernetes.io/ingress.class': nginx
      }
    spec:
      rules:
      - host: foo.bar.com
        http:
          paths:
          - path: /foo
            backend:
              serviceName: echoheaders
              servicePort: 80

Let’s save this into a file called **echo-ing.yaml** and add it to our cluster:

$ kc create -f echo-ing.yaml

So far, we’ve done the following:

1. Installed the **Nginx Controller** and a **Default backend**

1. Installed a sample “**echoheaders**” app

1. Created an **Ingress** resource

### Testing directly from the controller pod

We can exec directly into the pod to test that the controller is working properly, for example:

    $ kc exec -it nginx-ingress-controller-1575676611-fxkg8 /bin/bash
    root@nginx-ingress-controller-1575676611-fxkg8:/# curl localhost
    default backend — 404

The above output shows that we are hitting the default backend since we haven’t specified a **Host** header or a **path** when making the curl request. If we do specify both, we are able to make a successful request to the backend, which is our **echoheaders** app in this case:

    root@nginx-ingress-controller-1575676611-fxkg8:/# curl localhost/foo -H 'Host: foo.bar.com'
    CLIENT VALUES:
    client_address=10.4.5.28
    command=GET
    real path=/foo
    ...

    SERVER VALUES:
    server_version=nginx: 1.10.0 - lua: 10001

    HEADERS RECEIVED:
    accept=*/*
    connection=close
    host=foo.bar.com
    user-agent=curl/7.47.0
    ...

This confirms that our Ingress controller is running and forwarding requests to the backend service properly.

## Let’s build the Nginx Proxy!

The final step is to build an **Nginx Proxy** on top of this that will receive traffic over the VPN and forward it onto the controllers in the backend. Note that we could have directly opened up our cluster nodes on ports **31001/32001** over the VPN but that’s not a good idea for two reasons: the IPs of the nodes in the cluster can sometimes change, either due to upgrades or resizing of the cluster, and also because clients want to connect on ports 80 and 443. This is exactly why building an additional layer on top in the form of an Nginx Proxy is a good idea.

I will show a simple example of how to create the proxy below using **gcloud** commands. Let’s create a **test-proxy** instance first:

    $ gcloud compute instances create test-proxy \
    --image-project debian-cloud --image-family debian-9 \
    --network <network> --subnet <subnet> \
    --tags "<tags_needed>"

Use **gcloud** to SSH onto the instance and install Nginx:

$ sudo apt-get install nginx

We need to tweak the Nginx config a little. In the **/etc/nginx.conf** file, comment out the include for the virtual host config, and move the **conf.d** include outside the ‘**http**’ section, like below:

    ##
        # Virtual Host Configs
        ##

    #   include /etc/nginx/sites-enabled/*;
    }

    include /etc/nginx/conf.d/*.conf;

After you’ve done that, create a **ingress-proxy.conf** config file in the **conf.d** directory. Add the IP addresses of your backends along with the ports as shown in the example below.

    stream {
      upstream backend_nodes {
        server 10.7.0.2:31001;
        server 10.7.0.3:31001;
        server 10.7.0.4:31001;
      }

    upstream backend_nodes_ssl {
        server 10.7.0.2:32001;
        server 10.7.0.3:32001;
        server 10.7.0.4:32001;
      }

    server {
        listen 80;
        proxy_pass backend_nodes;
      }

    server {
        listen 443;
        proxy_pass backend_nodes_ssl;
      }
    }

## Time to test our proxy!

Now restart nginx and make a curl request to **localhost**. You will hit the **default-http-backend** in this case because it will not match any of the rules you specified in the Ingress resource we created above.

    $ curl localhost
    default backend — 404

Let’s test this with a real request against our **echoheaders** app. In case you are curious, let me show you the path your request will take:

1. It will hit the local Nginx instance (our **proxy**)

1. The proxy will forward the request to one of the **cluster nodes** in the backend where the **Nginx Controller** is running.

1. The Nginx Controller is keeping track of where the **backends** for our **services** are. This request specifies a **Host** header and a **path** that matches the **echoheaders** service in the Ingress resource.

1. The controller will forward the request to the **echoheaders** service.

1. Services in Kubernetes are used to group pods together and in this case our request will finally get forwarded onto the **pod** for **echoheaders**!

1. After all that, you should see the following response from your backend:

    $ curl localhost/foo -H 'Host: foo.bar.com'
    CLIENT VALUES:
    client_address=10.4.5.28
    command=GET
    real path=/foo
    ...

    SERVER VALUES:
    server_version=nginx: 1.10.0 - lua: 10001

    HEADERS RECEIVED:
    accept=*/*
    connection=close
    host=foo.bar.com
    user-agent=curl/7.52.1
    ...

Now that we have our Ingress Proxy running we can use it in place of an internal load balancer when sending traffic over the VPN from AWS or any other VPN connected location, like a remote office or data center.

Note that the IP address of the nodes in our cluster may change so it’s best to write a cronjob that periodically updates the **ingress-proxy.conf** file on the Nginx Proxy for us with an up to date list of the Kubernetes node-pool IPs.

So there you have it, folks! A complete solution to rolling your own **Nginx** based controller and proxy on **GKE**! In the next post I’m going to show you how to add **SSL** to this setup.
