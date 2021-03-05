
# Istio, Part II

You could read that as part 2, but in truth it feels more like part 11!

Back in June I [wrote a post](https://medium.com/@richard.marshall_58248/starting-with-istio-610421f5a815) describing why we’d finally started to look at bringning Istio into our kubernetes platform. The long-and-short of it is that it didn’t happen. Other things took over and learning how to integrate Istio into a running platform took some serious headspace.

So here I am, 4 months later and I’m pleased to say that we’re now much closer to it happening and for a different reason.

In June, we tried Istio using the version 0.7 release, only to see it replaced soon after by 0.8. We’re now at 1.0.2 and things have come on rapidly but istio seems to be creeping closer to maturity which is great news. It does mean though that a lot of what I learned about Istio in June had to be forgotten and it was better to start from a clean sheet. That applies to both installing it and running it.

Installing Istio 1.0 is pretty straight forward. The maintainers have adopted Helm templates, and while we’re not using Helm + Tiller internally, we can use helm to generate “traditional” k8s yaml files and install from there which works well for us. The istio site documents this process and it’s pretty painless.

We’ve opted to install and use Istio across a number of namespaces, which the installer partially supports. This is what we’ve done, why and how.

![](https://cdn-images-1.medium.com/max/2000/1*6wulujCw5iUHR6b3uullaA.png)

We run applications in a number of isolated (via network-policies) namespaces and want to keep web traffic away from the Istio control channel. To address this, we’ve deployed both a kubernetes nginx ingress controller and an Istio ingress gateway into a stand-alone namespace with the remainder of the Istio components in the default “istio-system” namespace. The helm templates support not defining an ingress gateway easy (and therefore the build of the istio-system namespace as we want it) but they don’t offer the ability to only define an ingress gateway and link it to the istio control plane.

To make this work for us, based on the [how-to from istio](https://istio.io/docs/setup/kubernetes/helm-install/#option-1-install-with-helm-via-helm-template) we’ve done the following:

    ## Generate a "complete" definition, including the ingress gateway

    helm template install/kubernetes/helm/istio --name istio --namespace istio-system \
    # We don't want to use egress gateways, yet. 
      --set gateways.istio-egressgateway.enabled=false \
    # We want an ingress gateway, but not exposed publically.
      --set gateways.istio-ingressgateway.type=ClusterIP \
    # We need to deinfe cluster IP boundaries so pods can egress
      --set global.proxy.includeIPRanges="10.0.0.0/8" \  
      > istio-full.yaml

    ## Generate a "control" definition, without the ingress gateway

    helm template install/kubernetes/helm/istio --name istio --namespace istio-system \
      --set gateways.istio-egressgateway.enabled=false \
      --set gateways.istio-ingressgateway.enabled=false \
      > istio-system.yaml

    ## Diff the two files to end up with only the ingress gateway definitions, then update the definitions so the gateway can find the control plane, etc.

    diff --normal istio-full.yaml istio-system.yaml | grep "<" | \
    sed -e "s/< //" \
     -e "s/namespace: istio-system/namespace: ingress/" \
     -e "s/istio-pilot:8080/istio-pilot.istio-system.svc.cluster.local:8080/" \
     -e "s/zipkin:9411/zipkin.istio-system.svc.cluster.local:9411/"\
     -e "s/istio-statsd-prom-bridge:9125/istio-statsd-prom-bridge.istio-system.svc.cluster.local:9125/"\
     -e "/nodePort/d"\
     > istio-ingress.yaml

    # Now apply the files, as per the how-to docs:
    kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml
    kubectl apply -f istio-system.yaml -n istio-system
    kubectl apply -f istio-ingress.yaml -n ingress

You should now, or at least soon, have your ingress gatwate way deployed into a seperate “DMZ” namespace with the remainder of Istio system deployed into the istio-system naemspace.

In addition to this, we deploy a standard nginx ingress controller into the ingress namespace to accept the incoming public connections, then use an ingress resource to forward al requests into the istio ingress gateway, where the service mesh takes over traffic.

Deploying the ingress controller and default backend are both straight forward. The more interesting parts are the service and ingress definitions:

    ---
    kind: Service
    apiVersion: v1
    metadata:
      name: nginx-ingress
      namespace: ingress
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    spec:
      externalTrafficPolicy: Local
      selector:
        name: nginx-ingress
      ports:
      - name: https
        port: 443
        protocol: TCP
        targetPort: 443
      - name: http
        port: 80
        protocol: TCP
        targetPort: 80
      type: LoadBalancer

    ---

    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      annotations:
      name: istio
      namespace: ingress
    spec:
      rules:
      - http:
          paths:
          - backend:
              serviceName: istio-ingressgateway
              servicePort: 80
            path: /
    ---

Why do we use an nginx controller instead of serving straight from the istio ingress gateway?

Right now Istio ingress is still pretty young and it doesn’t have the features we want. In fact we find that most kube ingress controller don’t offer the features we currently need.

At present we use the ngin ingress controllers to add/remove/dynamicaly updates headers, use the kube ingress rules to add IP whitelist restrictions, etc. While there is support for some of these things in Istio it’s more comlpicated and not as mature, so right now we’re going to adopt a two-tier approach.

The final step to building the mesh is to enable sidecar injection in a namespace. This means that each time a pod is deployed, kubernetes will transparently inject the istio proxy sidecar and bring the pod “on-mesh” Without this, istio can’t really help. Fortunatly this step is easy:

    kubectl label namespace [application-namespace] istio-injection=enabled

There may be pods where you **don’t** want the sidecar injected, for example the nginx ingress controller, becuase you want it to sit outside the mesh. Fortunately, this is simple, and just requires an annotation on the pod/deployment. e.g.

    apiVersion: v1
    kind: Pod
    metadata:
      annotations:
        sidecar.istio.io/inject: "false"

Now we have a working istio service mesh deployed, we want to start making use of it. [In my next pos](https://medium.com/ww-engineering/istio-iii-why-use-it-2ba2500cceda)t, I’ll cover what problem we’re trying to solve and how we’re going about it. I also cover whitelisting [here](https://medium.com/ww-engineering/istio-ip-whitelists-1633acbd4205).
