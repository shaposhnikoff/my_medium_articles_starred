
# Kubernetes Istio Canary Deployment

We’re using Istio+Kiali to perform and visualise Canary deployments

![[https://unsplash.com/photos/V41PulGL1z0](https://unsplash.com/photos/V41PulGL1z0)](https://cdn-images-1.medium.com/max/10434/1*kGCZ5BBqGz6elOrqEYLUsg.jpeg)*[https://unsplash.com/photos/V41PulGL1z0](https://unsplash.com/photos/V41PulGL1z0)*

### Parts

1. [Canary Deployment with GitlabCI + GitOps/Manual Approach](https://medium.com/@wuestkamp/kubernetes-canary-deployment-1-gitlab-ci-518f9fdaa7ed?)

1. [Canary Deployment with Argo Rollouts](https://codeburst.io/kubernetes-canary-deployment-2-argo-rollouts-5e68e99b4fa3?source=friends_link&sk=58557d4fa81ff77382e59e1258c06d61)

1. (this article)

1. [Canary Deployment using Jenkins-X Istio Flagger](https://medium.com/@wuestkamp/jenkins-x-istio-flagger-canary-deployment-9d5e187c2334?source=friends_link&sk=fa0cf82c7051958b0a98e205375cba86)

## Canary Deployment

[Make sure to read part 1](https://medium.com/@wuestkamp/kubernetes-canary-deployment-1-gitlab-ci-518f9fdaa7ed?source=friends_link&sk=4f3b424099f4f973f634bda75e397254) where we explained shortly what Canary Deployments are. In there we also show how to implement those using vanilla Kubernetes resources.

## Istio

This article assumes you already know what Istio is all about. You can [read this](https://itnext.io/kubernetes-istio-simply-visually-explained-58a7d158b83f?source=friends_link&sk=378ed718d2d6cfd09e6d23c7616cba81) if you do not yet.

## The simple test App

![Every pod has two containers, the application and istio-proxy](https://cdn-images-1.medium.com/max/2660/1*jkq1WIPPU-Y2KQgTFSgatw.png)*Every pod has two containers, the application and istio-proxy*

We use a simple test app with frontend-nginx and backend-python pods. The nginx pods simply redirect every request to the backend pods and act as a proxy. For details check the yamls:

* [frontend.yaml](https://gitlab.com/wuestkamp/k8s-deployment-example-istio-canary-infrastructure/blob/master/i/k8s/frontend.yaml)

* [backend.yaml](https://gitlab.com/wuestkamp/k8s-deployment-example-istio-canary-infrastructure/blob/master/i/k8s/backend.yaml)

* [istio.yaml](https://gitlab.com/wuestkamp/k8s-deployment-example-istio-canary-infrastructure/blob/master/i/k8s/istio.yaml)

### Deploy the test App yourself

If you would like to follow along and try out the example app yourself check the [project's readme](https://gitlab.com/wuestkamp/k8s-deployment-example-istio-canary-infrastructure/blob/master/README.md).

## Initial Deployment

We see our application pods all having 2 containers, which shows the Istio sidecar is injected:

![](https://cdn-images-1.medium.com/max/2000/1*-cDLii_P_-ighhNpwrphHQ.png)

We also see the Istio Gateway Loadbalancer in istio-system namespace:

![](https://cdn-images-1.medium.com/max/2008/1*JMkayQnfWpkzzsynhJx9xA.png)

### Create Traffic

We use that IP to fire up some traffic which will hit the frontend pods, which in turn will redirect to the backend pods:

    while true; do curl -s --resolve 'frontend.istio-test:80:35.242.202.152' frontend.istio-test; sleep 0.1; done

We could also add frontend.istio-test to our hosts file.

### View Mesh using Kiali

We installed the test app and Istio together with Tracing, Grafana, Prometheus and Kiali (more in the [project’s readme](https://gitlab.com/wuestkamp/k8s-deployment-example-istio-canary-infrastructure/blob/master/README.md)). Hence we can use Kiali with:

    istioctl dashboard kiali # admin:admin

![Kiali visualises the actual traffic in the Mesh](https://cdn-images-1.medium.com/max/3904/1*AFkGZRWAruqrJw2Du7H4oQ.png)*Kiali visualises the actual traffic in the Mesh*

We see 100% of traffic is reaching the frontend service, then pods of frontend with label v1. These are simple nginx proxies and forward the requests to the backend service which points to backend pods with label v1.

Kiali works great together with Istio and provides out-of-the-box Mesh visualisation. Just beautiful.

## Canary Deployment

Our backend already has two k8s deployments, one for v1 and one for v2. Now we just need to tell Istio to redirect a certain percentage of requests to v2.

### Step 1: 10%

All we have to do is to adjust the VirtualService weights in [istio.yaml](https://gitlab.com/wuestkamp/k8s-deployment-example-istio-canary-infrastructure/blob/master/i/k8s/istio.yaml):

    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: backend
      namespace: default
    spec:
      gateways: []
      hosts:
      - "backend.default.svc.cluster.local"
      http:
      - match:
        - {}
        route:
        - destination:
            host: backend.default.svc.cluster.local
            subset: v1
            port:
              number: 80
    **      weight: 90
    **    - destination:
            host: backend.default.svc.cluster.local
            subset: v2
            port:
              number: 80
    **      weight: 10**

![](https://cdn-images-1.medium.com/max/3228/1*WgDVnPdMzAJUZEUffZp7cA.png)

We see that 10% of requests are redirected to v2.

### Step 2: 50%

Very simple to raise this to 50%:

    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: backend
      namespace: default
    spec:
    ...
        - destination:
            host: backend.default.svc.cluster.local
            subset: v1
            port:
              number: 80
    **      weight: 50
    **    - destination:
            host: backend.default.svc.cluster.local
            subset: v2
            port:
              number: 80
    **      weight: 50**

![](https://cdn-images-1.medium.com/max/3248/1*FQ1TN9Monf00MK-4x8XC6A.png)

### Step 3: 100%

Now the Canary deployment can be considered done and all traffic is redirected to v2:

![](https://cdn-images-1.medium.com/max/3304/1*P9E42EALvXneMrV--VJ0_w.png)

## Manual Canary Testing

Let’s say we currently expose the backend v2 to 10% of requests. What if we would like to test v2 also manually ourselves to confirm all is working?

We can add a specific matching rule based on HTTP headers:

    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: backend
      namespace: default
    spec:
      gateways: []
      hosts:
      - "backend.default.svc.cluster.local"
      http:
    **  - match:
        - headers:
            canary:
              exact: "canary-tester"
        route:
        - destination:
            host: backend.default.svc.cluster.local
            subset: v2
            port:
              number: 80
          weight: 100**
      - match:
        - {}
        route:
        - destination:
            host: backend.default.svc.cluster.local
            subset: v1
            port:
              number: 80
          weight: 90
        - destination:
            host: backend.default.svc.cluster.local
            subset: v2
            port:
              number: 80
          weight: 10

Using curl we can now force requesting v2 by sending the header:

![](https://cdn-images-1.medium.com/max/3096/1*ZMzaFSefvrVY71wN_H8INQ.png)

Requests without the header will still be handled by the 1/10 ratio:

![](https://cdn-images-1.medium.com/max/3208/1*MttnDwmxjd_A0On_UBLEQA.png)

## Canary for two dependent service versions

Now we look at having version v2 for frontend and backend. For both we specified that 10% of traffic should go to v2:

![Kiali Graph without service nodes](https://cdn-images-1.medium.com/max/2332/1*Qi9JTucvX7DKLe9RxdN_9A.png)*Kiali Graph without service nodes*

We see that frontend v1 and v2 both redirect traffic in 1/10 ratio to backend v1 and v2.

What if we would like for only frontend-v2 to only connect to backend-v2, because it’s not compatible with v1? To achieve this we’ll only specify the 1/10 Canary ratio for frontend, then control which traffic reaches backend-v2 using the sourceLabels matcher:

    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: backend
      namespace: default
    spec:
      gateways: []
      hosts:
      - "backend.default.svc.cluster.local"
      http:
    ...
      - match:
    **    - sourceLabels:
            app: frontend
            version: v2**
        route:
        - destination:
            host: backend.default.svc.cluster.local
    **        subset: v2
    **        port:
              number: 80
          weight: 100

This results in what we want:

![Kiali Graph without service nodes](https://cdn-images-1.medium.com/max/2372/1*LkX4380PnmnKd7XhTrdt6A.png)*Kiali Graph without service nodes*

## Difference to manual Canary

In [part 1](https://medium.com/@wuestkamp/kubernetes-canary-deployment-1-gitlab-ci-518f9fdaa7ed?) we performed a Canary deployment manually, also using two k8s deployments. There we adjusted the request ratio by manipulating the replica count. It works but has huge disadvantages.

Using Istio it’s possible to define the request ratio independently of the replica count. This means, for example, we can use existing HPAs (Horizontal Pod Autoscalers) and don’t have to adjust these depending on the current Canary state.

## Recap

Istio is working great and the combination with Kiali is very powerful. Next on my list of interest is the combination of Spinnaker with Istio for automation and Canary analysis.

## Become Kubernetes Certified

![[https://killer.sh](https://killer.sh)](https://cdn-images-1.medium.com/max/3534/1*7Kbj17_6VncUuoBqNsAzzg.png)*[https://killer.sh](https://killer.sh)*
