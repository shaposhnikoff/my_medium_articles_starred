
# Enable Mutual TLS Authentication Between Your Kubernetes Workloads Using Istio

A guide to Istio authentication and mutual TLS between your microservices on Kubernetes

![Photo by [Lanju Fotografie](https://unsplash.com/@lanju_fotografie?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/future?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/8952/1*q_4OezvkEa-wVB20bvpjaA.jpeg)*Photo by [Lanju Fotografie](https://unsplash.com/@lanju_fotografie?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/future?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

In software engineering, access control generally consists of two parts, authentication and authorisation. While authentication refers to understanding whether the requestor is what it claims to be, authorisation refers to specifying what actions the requestor is allowed to perform.

This article is a follow up to “[How to Harden Your Microservices on Kubernetes Using Istio](https://medium.com/better-programming/how-to-harden-your-microservices-on-kubernetes-using-istio-29c23dd90670)”. Today, let’s discuss how to enable mutual TLS authentication between your Kubernetes microservices using Istio.

## Prerequisites

This article assumes that you have knowledge of Kubernetes and Microservices and are aware of [Istio](https://istio.io). For an introduction to Istio, I recommend you check out [How to Manage Microservices on Kubernetes With Istio](https://medium.com/better-programming/how-to-manage-microservices-on-kubernetes-with-istio-c25e97a60a59). For a quick introduction to Istio security, please read [How to Harden Your Microservices on Kubernetes Using Istio](https://medium.com/better-programming/how-to-harden-your-microservices-on-kubernetes-using-istio-29c23dd90670).

Ensure you have a running Kubernetes cluster for following the hands-on demonstration. I have done the below illustration on [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine), but it would work the same way on any other Kubernetes cluster.

## Mutual Authentication by Default

Istio, by default, enables TLS communication between the workloads which has side-cars injected. That allows for end-to-end encryption between microservices to prevent a man-in-the-middle attack.

While this is the default setting and may work for most cases, this runs in compatibility mode. This means that traffic between two services that have side-cars injected would send encrypted traffic, but workloads that do not have side-car inserted can interface with the backend microservice on plaintext HTTP.

It means teams that have just introduced Istio don’t have to struggle to make all source traffic TLS enabled.

Let's understand this behaviour with the following hands-on demonstration.

Install Istio within your Kubernetes cluster by following the [Getting Started With Istio on Kubernetes](https://medium.com/better-programming/getting-started-with-istio-on-kubernetes-e582800121ea) guide. You don’t need to install the Book Info application for this demo.

Create three namespaces, foo, bar and legacy, and enable automatic side-car injection on the foo and bar namespaces. Don’t label the legacy namespace as we want to simulate traffic from workloads not containing the side-car.

    $ kubectl create ns foo
    namespace/foo created
    $ kubectl create ns bar
    namespace/bar created
    $ kubectl create ns legacy
    namespace/legacy created
    $ kubectl label namespace foo istio-injection=enabled
    namespace/foo labeled
    $ kubectl label namespace bar istio-injection=enabled
    namespace/bar labeled

Let us now create two applications, httpbin and sleep, on all three namespaces.

    $ kubectl apply -f [samples/httpbin/httpbin.yaml](https://raw.githubusercontent.com/istio/istio/release-1.5/samples/httpbin/httpbin.yaml) -n foo
    $ kubectl apply -f [samples/sleep/sleep.yaml](https://raw.githubusercontent.com/istio/istio/release-1.5/samples/sleep/sleep.yaml) -n foo 
    $ kubectl apply -f [samples/httpbin/httpbin.yaml](https://raw.githubusercontent.com/istio/istio/release-1.5/samples/httpbin/httpbin.yaml) -n bar
    $ kubectl apply -f [samples/sleep/sleep.yaml](https://raw.githubusercontent.com/istio/istio/release-1.5/samples/sleep/sleep.yaml) -n bar
    $ kubectl apply -f [samples/httpbin/httpbin.yaml](https://raw.githubusercontent.com/istio/istio/release-1.5/samples/httpbin/httpbin.yaml) -n legacy 
    $ kubectl apply -f [samples/sleep/sleep.yaml](https://raw.githubusercontent.com/istio/istio/release-1.5/samples/sleep/sleep.yaml) -n legacy

Now list the pods from all three namespaces to see what we get.

Let’s start with foo:

    $ kubectl get pod -n foo
    NAME                       READY   STATUS    RESTARTS   AGE
    httpbin-654c6cbbb9-76dgp   2/2     Running   0          96s
    sleep-6bdb595bcb-vwh29     2/2     Running   0          86s

What about bar:

    $ kubectl get pod -n bar
    NAME                       READY   STATUS    RESTARTS   AGE
    httpbin-654c6cbbb9-rvxgv   2/2     Running   0          102s
    sleep-6bdb595bcb-rhf7g     2/2     Running   0          91s

And legacy:

    $ kubectl get pod -n legacy
    NAME                       READY   STATUS    RESTARTS   AGE
    httpbin-654c6cbbb9-xsmd9   1/1     Running   0          90s
    sleep-6bdb595bcb-t9bkx     1/1     Running   0          83s

Do you see the difference? Since we don’t have automatic side-car injection enabled on the legacy namespace, we just have one container within the pods. The foo and bar namespaces have two containers within the pods as we labeled them appropriately to enable Istio side-car injection.

Now let’s generate traffic from the sleep microservices to httpbin microservices from all three namespaces to all three namespaces, as shown:

![Mutual Auth by Default](https://cdn-images-1.medium.com/max/2000/1*q7rpRUNxIrhiBZGGDvDfWg.png)*Mutual Auth by Default*

<iframe src="https://medium.com/media/a5a4e14c71a28b8f7a82a28eeb62677b" frameborder=0></iframe>

If you look at the results, you see that you get an HTTP 200 response for all requests. Istio is intelligent enough to understand with whom it’s interacting and chooses to either send TLS traffic or plaintext traffic based on the target. Istio sends an X-Forwarded-Client-Cert* *header to the back end if it uses Mutual TLS Authentication, and it’s* *an* *excellent way to understand whether the traffic is encrypted or not.

Let’s create traffic from sleep.foo to httpbin.foo.

    $ kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl [http://httpbin.foo:8000/headers](http://httpbin.foo:8000/headers) -s | grep X-Forwarded-Client-Cert

    "X-Forwarded-Client-Cert": "By=spiffe://cluster.local/ns/foo/sa/httpbin;Hash=05b619f1ebd6d8e4f59158bb614e8cac558a4228ec7bda28a80c9ad1a2594bf1;Subject=\"\";URI=spiffe://cluster.local/ns/foo/sa/sleep"

Since both source and target have side-cars, we see the header. Istio encrypts the traffic as evident by the X-Forwarded-Client-Cert header.

Now let’s create traffic from sleep.foo to httpbin.legacy. In this case, the source contains a side-car, but the target does not.

    $ kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl [http://httpbin.legacy:8000/headers](http://httpbin.legacy:8000/headers) -s | grep X-Forwarded-Client-Cert

We see no such header. That means that traffic between the source and destination is plaintext.

## Enabling Strict Mutual TLS

Running Istio using the default settings provides some level of protection, but this is not all that Istio is capable of.

Istio delivers a lot of other features such as allowing only the required traffic to propagate and controlling access so that only related microservices interact with each other.

You may want to enable strict TLS to disable insecure traffic within your service mesh altogether, to ensure security is of prime importance.

You can enable Strict Mutual TLS at the global level, namespace level, or the workload level.

To enable Strict Mutual TLS on the Global Level, run the following:

<iframe src="https://medium.com/media/75e510ba86c70b1162e7ef37766c3d5d" frameborder=0></iframe>

As you see, this is applying a PeerAuthentication policy on the istio-system namespace with an mtls mode STRICT. That means that all namespaces within the Kubernetes cluster would only allow secure TLS traffic.

Now let’s run the same example and see what we get.

<iframe src="https://medium.com/media/37fa120496adb807cb1df092d1505892" frameborder=0></iframe>

As you see, Istio rejects all requests from the legacy namespace with an exit code 56. That means the TLS handshake is not successful, as the service mesh is expecting TLS traffic only.

Now, let’s enable Mutual TLS on a namespace level. But before we do that, we need to clean up the existing policy by running the following:

    $ kubectl delete peerauthentication -n istio-system default
    peerauthentication.security.istio.io "default" deleted

To enable strict Mutual TLS on a namespace level, run the following:

<iframe src="https://medium.com/media/7a5e1f7f9a78ef779221b297880cb4ab" frameborder=0></iframe>

That enforces strict TLS checking on the foo namespace only. Let us rerun the test:

<iframe src="https://medium.com/media/b4498faec8a6133f0f455e7965b360c0" frameborder=0></iframe>

As you can see, only the request from sleep.legacy to httpbin.foo is failing, as expected.

You can also enable strict TLS on a workload level in a similar way. But first, let’s clean up the namespace policy.

    $ kubectl delete peerauthentication -n foo default
    peerauthentication.security.istio.io "default" deleted

Let’s now create a PeerAuthentication policy and a DestinationRule to enforce strict TLS on the workload level:

<iframe src="https://medium.com/media/ebe45099858732350ef1c42a90989646" frameborder=0></iframe>

Let’s rerun the test and see what happens:

<iframe src="https://medium.com/media/cd83d021b3a01126235962293e4cf828" frameborder=0></iframe>

As expected, only traffic from sleep.legacy to httpbin.bar is failing.

## Conclusion

Thanks for reading! I hope you enjoyed the article.

In the next part “[Enable Access Control Between Your Kubernetes Workloads Using Istio](https://medium.com/better-programming/enable-access-control-between-your-kubernetes-workloads-using-istio-cf72a9f9bd5e)”, we will deep dive into Istio Authorisation with a hands-on demonstration, so see you there!
