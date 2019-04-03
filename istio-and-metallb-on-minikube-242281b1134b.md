
# Istio and MetalLB on Minikube



**Minikube** is a tool that makes it easy to run K8S locally, but that is only part of the puzzle. We want that our local dev environment looks like production env as much as possible. As **Istio** becomes more popular and widely used I’ll try to explain how to set it up with **MetalLB** — bare metal load-balancer for K8S.

## Istio

Service mesh is a configurable infrastructure layer for microservice oriented applications. It’s responsible for the reliable delivery of requests through the complex topology of services that comprise a modern, cloud native application. In practice, the service mesh is typically implemented as an array of lightweight network proxies that are deployed alongside application code. The mesh provides service discovery, load balancing, encryption, authentication and authorisation, support for the circuit breaker pattern, and other capabilities.

The service mesh is usually implemented by providing a proxy instance, called a sidecar, for each service instance. Sidecars handle inter‑service communications, monitoring, security‑related concerns — anything that can be abstracted away from the individual services. This way, developers can handle development, support, and maintenance for the application code in the services; operations can maintain the service mesh and run the app.

**Istio**, backed by Google, IBM, and Lyft, is currently the best‑known service mesh architecture. **Kubernetes**, which was originally designed by Google, is currently the only container orchestration framework supported by Istio.

### Installing Istio

Istio will be installed in it’s own directory (current version is 1.0.2)

    $ curl -L [https://git.io/getLatestIstio](https://git.io/getLatestIstio) | sh -
    $ cd ~/istio-1.0.2
    $ sudo mv bin/istioctl /usr/local/bin

After

    $ helm init

is completed, Istio can be installed in **istio-system** namespace via helm charts:

    $ cd ~/istio-1.0.2
    $ helm install install/kubernetes/helm/istio --name istio --namespace istio-system

Don’t forget to label the namespace for injection:

    $ kubectl label namespace default istio-injection=enabled

Afterwards, namespace pods can be checked with:

    $ kubectl get pods --namespace=istio-system

Now check **istio-ingressgateway **service,

    kubectl get service -n istio-system istio-ingressgateway

similar output should appear:

    NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP                                                                                                             
    istio-ingressgateway   LoadBalancer   10.104.226.154   <pending>

Notice that **EXTERNAL-IP** is in **<pending>** state, so that’s the part when MetalLB comes in.

## MetalLB

If a Kubernetes cluster is installed on *bare metal* or on virtual machines, it’s missing an external load balancer. It’s a cloud network load balancer that comes with platforms such as Azure or GCM, that provides an externally-accessible IP address that sends traffic to the correct port on your cluster nodes. MetalLB is a** load balancer** designed to run on and to work with Kubernetes and it will allow you to use the type LoadBalancer when you declare a service.

*A LoadBalancer service is the standard way to expose a service to the internet. It will give you a single IP address that will forward all traffic to your service.*

### Install MetalLB on Minikube

MetalLB runs in two parts: a cluster-wide **controller**, and a per-machine protocol **speaker**. Install MetalLB by applying the manifest:

    $ kubectl apply -f [https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/metallb.yaml](https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/metallb.yaml)

This manifest creates a bunch of resources. Most of them are related to access control, so that MetalLB can read and write the Kubernetes objects it needs to do its job.

The two pieces of interest are the “**controller**” deployment, and the “**speaker**” DaemonSet. By monitoring:

    $ kubectl get pods -n metallb-system

similar output should appear:

    NAME                          READY     STATUS    RESTARTS   AGE
    controller-7fbd769fcc-58fm8   1/1       Running   0          50m
    speaker-rctgd                 1/1       Running   0          50m

In real environment one controller pod and one speaker pod for each node should appear in a cluster, but using Minikube, only one controller and one speaker show up.

### Apply ConfigMap

MetalLB’s configuration is a standard Kubernetes ConfigMap, config under the **metallb-system** namespace. It contains two pieces of information: what IP addresses it’s allowed to hand out and which protocol to use for such task.
In this configuration MetalLB is instructed to hand out address from the 192.168.99.100/28 range, using **layer 2 mode** (protocol: **layer2**). 
In local development, Minikube IP address is used as the start of the range. To get Minikube IP address use:

    $ minikube ip

Result example:

    192.168.99.100

Apply this configuration:

    apiVersion: v1
    kind: ConfigMap
    metadata:
      namespace: metallb-system
      name: config
    data:
      config: |
        address-pools:
        - name: custom-ip-space
          protocol: layer2
          addresses:
          - 192.168.99.100/28

Now check **istio-ingressgateway **service again,

    kubectl get service -n istio-system istio-ingressgateway

similar output should appear:

    NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP                                                                                                             
    istio-ingressgateway   LoadBalancer   10.104.226.154   **192.168.99.96**

MetalLB is used here in order to provide an IP address for the Istio IngressGateway service of LoadBalancer type thus emulating a cloud deployment. This is used to avoid NodePort implementations with Istio as they are not to be used due to Istio design.

## MetalLB and Istio example: Using an Istio Gateway

In a Kubernetes environment, the Kubernetes Ingress Resource is used to specify services that should be exposed outside the cluster. In an Istio service mesh, a better approach (which also works in both Kubernetes and other environments) is to use a different configuration model, namely Istio Gateway. A Gateway allows Istio features such as monitoring and routing rules to be applied to traffic entering the cluster.

### Before you begin

* Setup **Istio** and **MetalLB** by following the instructions in sections above.

* Make sure your current directory is the **istio directory**.

Start the **httpbin sample**, which will be used as the destination service to be exposed externally.

    $ kubectl apply -f <(istioctl kube-inject -f samples/httpbin/httpbin.yaml)

Execute the following command to determine if your Kubernetes cluster is running in an environment that supports external load balancers:

    $ kubectl get svc istio-ingressgateway -n istio-system

Check EXTERNAL-IP column for address of gateway:

    NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP   
    istio-ingressgateway   LoadBalancer   10.104.226.154  192.168.99.96

If the EXTERNAL-IP value is set, your environment has an external load balancer that you can use for the ingress gateway, in this case MetalLB. If the **EXTERNAL-IP** value is **<none>** (or perpetually **<pending>**), your environment does not provide an external load balancer for the ingress gateway, in other words, **check MetalLB installation**.

### Configure ingress using an Istio Gateway

An ingress Gateway describes a load balancer operating at the edge of the mesh that receives incoming HTTP/TCP connections. It configures exposed ports, protocols, etc. but, unlike Kubernetes Ingress Resources, does not include any traffic routing configuration. Traffic routing for ingress traffic is configured using Istio routing rules instead, exactly in the same way as for internal service requests.

* Create an Istio Gateway:

    apiVersion: networking.istio.io/v1alpha3
    kind: Gateway
    metadata:
      name: httpbin-gateway
    spec:
      selector:
        istio: ingressgateway # use Istio default gateway implementation
      servers:
      - port:
          number: 80
          name: http
          protocol: HTTP
        hosts:
        - "*"

* Configure routes for traffic entering via the Gateway:

    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: httpbin
    spec:
      hosts:
      - "*"
      gateways:
      - httpbin-gateway
      http:
      - match:
        - uri:
            prefix: /
        route:
        - destination:
            port:
              number: 8000
            host: httpbin

The Gateway configuration resources allow external traffic to enter the Istio service mesh and make the traffic management and policy features of Istio available for edge services.

In the preceding steps, service is created inside the service mesh and exposed an HTTP endpoint of the service to external traffic.

Visit [*http://192.168.99.96:80/](http://192.168.99.100:80/headers)* and see **httpbin** magic.
