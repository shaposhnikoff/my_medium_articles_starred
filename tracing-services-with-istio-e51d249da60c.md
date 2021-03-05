
# Tracing services with Istio

Super quick post , When istio injects the envoy container side car into your pod , each request that comes in and out is “appended” with a numbers of http headers that then they’re use for tracing .

This is one of the many benefits of the “side car injection” approach that istio has embrace , bit intrusive yea , but so far seems to work nicely.

Ok so quickly you can deploy jaeger and zipkin by enabling it on the chart:
[**istio/istio**
*istio - An open platform to connect, manage, and secure microservices.*github.com](https://github.com/istio/istio/blob/master/install/kubernetes/helm/istio/values.yaml#L415)

In case you don’t have it already enabled.

Then if you look a little bit on the charts you’ll find references such as:

![](https://cdn-images-1.medium.com/max/2000/1*IeIAfZClvqJHvDkXTulDrg.png)

that’s from Mixer , so without too much digging you can see how mixer is passing stats to zipkin , and remember mixer sees everything .

So if we port-forward to were jaeger is listening on:

    kubectl port-forward -n istio-system istio-tracing-754cdfd695-ngssw 16686:16686

And we hit http://localhost:16686 , we’ll find jaeger:

![](https://cdn-images-1.medium.com/max/2370/1*5KEKom5j8tyagFVdSIGWlw.png)

It’s really interesting for tracing and to get a general idea of services that might be taking too long to process etc , I’ve force an error and it looks like:

![](https://cdn-images-1.medium.com/max/3424/1*gECzUb6Hh5QjxK0-ueYT8g.png)

If the pod nginx would be calling extra services they should be displayed there too , **cause remember ALL ingress/egress traffic is captured by the envoy sidecar inside your pod.**
