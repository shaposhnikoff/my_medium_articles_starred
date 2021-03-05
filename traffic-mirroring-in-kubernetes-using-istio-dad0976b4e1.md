
# Traffic Mirroring in Kubernetes Using Istio

How to run robust operational acceptance tests for your Kubernetes microservices

![Photo by [Ashim D’Silva](https://unsplash.com/@randomlies?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/mirror?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/6144/1*IOg4ZlvI8cwy93rnnLVXMw.jpeg)*Photo by [Ashim D’Silva](https://unsplash.com/@randomlies?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/mirror?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

Having multiple versions of your application running at the same time gives you a fair amount of flexibility. It enables you to switch over and switch back traffic between various versions as required. It allows you to do canary releases, A/B testing, and controlled rollouts to production.

With the advent of microservices and container orchestration platforms like [Kubernetes](https://kubernetes.io/) to support it, it has become the new norm. With [Istio](https://istio.io), you can create a robust deployment and release strategy through traffic mirroring.

This article is a follow up to “[Kubernetes Services over HTTPS With Istio’s Secure Gateways.](https://medium.com/better-programming/kubernetes-services-over-https-with-istios-secure-gateways-210b2ce91b71)” Today, let’s discuss *traffic mirroring.*

Traffic mirroring, also called shadowing, is a powerful concept that has gained ground in recent times. It’s a risk-free method of testing your releases in the production environment without impacting your end users.

Most traditional enterprises had a pre-production environment which used to be a replica of production. Ops team deployed releases to the pre-production environment where testers created synthetic traffic to mimic the live environment.

That allowed teams to understand how the code would behave in the production environment, and whether the code is functionally and non-functionally ready to be deployed to production.

They used pre-production environments for doing performance testing, volumetric testing, and operational acceptance testing for their applications.

While that all sounds good, it was not without issues. In a traditional infrastructure, you spent a lot of money and resources on creating static test environments. You paid engineers to maintain a replica of production.

Most synthetic traffic differed from live traffic because live traffic is current user interactions, while artificial traffic relies on historical data. It wasn’t an exception to miss a lot of scenarios there.

Traffic mirroring allows you to implement a similar setup for operational acceptance testing. It goes the extra mile by allowing you to do this testing using live traffic without impacting the end users.

## How Traffic Mirroring Works

Traffic mirroring works using the steps below:

1. You deploy a new version of the application and switch on traffic mirroring.

1. The old version responds to requests like before but also sends an asynchronous copy to the new version.

1. The new version processes the traffic but does not respond to the user.

1. The operations team monitor the new version and report any issues to the development team.

![Traffic mirroring](https://cdn-images-1.medium.com/max/2000/1*P_c0Vlpur7QjfM_N8kstEQ.png)*Traffic mirroring*

As the application processes live traffic, it helps the team uncover issues that they would typically not find in a pre-production environment. You can use monitoring tools, such as [Prometheus](https://prometheus.io/) and [Grafana](https://grafana.com/), for recording and monitoring your test results.

## Prerequisites

I will demonstrate how we can do traffic mirroring on a Kubernetes cluster using Istio. To gain some familiarity with Istio, check out “[How to Manage Microservices on Kubernetes With Istio](https://medium.com/better-programming/how-to-manage-microservices-on-kubernetes-with-istio-c25e97a60a59).”

You need to have a running Kubernetes cluster for following the hands-on guide. You may use a managed Kubernetes cluster such as Google Kubernetes Engine for this practice.

## Deploy the Nginx application

Install Istio within your Kubernetes cluster by following the “[Getting Started With Istio on Kubernetes](https://medium.com/better-programming/getting-started-with-istio-on-kubernetes-e582800121ea)” guide. You don’t need to install the Book Info application for this demo.

Deploy two versions (v1 and v2) of [Nginx](https://www.nginx.com/) on Kubernetes.

Deploy nginx:v1.

<iframe src="https://medium.com/media/e533447482cb59b90cca52e9ace853fc" frameborder=0></iframe>

Deploy nginx:v2.

<iframe src="https://medium.com/media/e4aad9a5856baf492415ccf94cc5168e" frameborder=0></iframe>

Expose the Nginx applications through a service.

<iframe src="https://medium.com/media/d74dad04bf7be5b6d3bd961a58c2bea4" frameborder=0></iframe>

Deploy the sleep microservice for sending traffic to the application.

<iframe src="https://medium.com/media/c80805966b441b9fbcc839fc1f2a0d8e" frameborder=0></iframe>

Get the pods to see if they are ready.

    $ kubectl get pod
    NAME                        READY   STATUS    RESTARTS   AGE
    nginx-v1-5ff9f64978-bwtqh   2/2     Running   0          3m36s
    nginx-v2-596ffd86d7-89n99   2/2     Running   0          19s
    sleep-674f75ff4d-cmt7p      2/2     Running   0          2m25s

Now let’s send some traffic to the Nginx service.

    $ export SLEEP_POD=$(kubectl get pod -l app=sleep -o jsonpath={.items..metadata.name})
    $ kubectl exec -it $SLEEP_POD -c sleep -- sh -c 'curl  [http://nginx:8000'](http://nginx:8000')
    This is version 1
    $ kubectl exec -it $SLEEP_POD -c sleep -- sh -c 'curl  [http://nginx:8000'](http://nginx:8000')
    This is version 2
    $ kubectl exec -it $SLEEP_POD -c sleep -- sh -c 'curl  [http://nginx:8000'](http://nginx:8000')
    This is version 2

Traffic is flowing equally between nginx:v1 and nginx:v2. Check the logs for both versions.

    $ export V1_POD=$(kubectl get pod -l app=nginx,version=v1 -o jsonpath={.items..metadata.name})
    $ kubectl logs $V1_POD -c nginx
    127.0.0.1 - - [13/May/2020:23:05:11 +0000] "GET / HTTP/1.1" 200 18 "-" "curl/7.35.0" "-"
    $ export V2_POD=$(kubectl get pod -l app=nginx,version=v2 -o jsonpath={.items..metadata.name})
    $ kubectl logs $V2_POD -c nginx
    127.0.0.1 - - [13/May/2020:23:05:12 +0000] "GET / HTTP/1.1" 200 18 "-" "curl/7.35.0" "-"
    127.0.0.1 - - [13/May/2020:23:05:13 +0000] "GET / HTTP/1.1" 200 18 "-" "curl/7.35.0" "-"

We see logs on both pods, which confirms our finding. Let’s delete the pods to clear the logs. Kubernetes replaces the deleted pods with fresh ones.

    $ kubectl delete pod $V1_POD
    pod "nginx-v1-5ff9f64978-bwtqh" deleted
    $ kubectl delete pod $V2_POD
    pod "nginx-v2-596ffd86d7-89n99" deleted
    $ kubectl get pod
    NAME                        READY   STATUS    RESTARTS   AGE
    nginx-v1-5ff9f64978-9vzhm   2/2     Running   0          46s
    nginx-v2-7cb4b5c868-86s5m   2/2     Running   0          34s
    sleep-674f75ff4d-cmt7p      2/2     Running   0          30m

## Create the Virtual Service and Destinations Rules

Now let’s create a destinations rule to define routes to both versions and create a virtual service to route all traffic to nginx:v1.

<iframe src="https://medium.com/media/a8bd571372fa7937ec2eeb69b143450c" frameborder=0></iframe>

Let’s generate some traffic and see where it goes.

    $ kubectl exec -it $SLEEP_POD -c sleep -- sh -c 'curl  [http://nginx:8000'](http://nginx:8000')
    This is version 1
    $ kubectl exec -it $SLEEP_POD -c sleep -- sh -c 'curl  [http://nginx:8000'](http://nginx:8000')
    This is version 1
    $ kubectl exec -it $SLEEP_POD -c sleep -- sh -c 'curl  [http://nginx:8000'](http://nginx:8000')
    This is version 1

We are getting all responses from nginx:v1. Query the logs for both versions.

    $ export V1_POD=$(kubectl get pod -l app=nginx,version=v1 -o jsonpath={.items..metadata.name})
    $ kubectl logs $V1_POD -c nginx
    127.0.0.1 - - [13/May/2020:23:08:11 +0000] "GET / HTTP/1.1" 200 18 "-" "curl/7.35.0" "-"
    $ export V2_POD=$(kubectl get pod -l app=nginx,version=v2 -o jsonpath={.items..metadata.name})
    $ kubectl logs $V2_POD -c nginx
    <none>

There are no logs recorded for nginx:v2. As expected, no traffic flows to v2.

## Mirror Traffic

Here comes the central part. Let’s mirror the traffic to nginx:v2 and see for ourselves what is going on by applying the below YAML.

<iframe src="https://medium.com/media/ee3e5d83210163bf9dc8234ab3779e5d" frameborder=0></iframe>

If you observe the YAML, the route still has nginx:v1 as the destination; however, it includes a mirror section that mirrors 100% of the traffic to nginx:v2. As a result, nginx:v1 responds actively to requests, and nginx:v2 receives traffic asynchronously. Bear in mind that the request is fire-and-forget: nginx:v2 does not respond to the requestor.

In this setup, we set the mirror_percent to 100, which means we mirror the entire traffic to nginx:v2. You can change it to a value more suitable for your use case. If you do not include the mirror_percent field, Istio mirrors all traffic to nginx:v2.

Let’s send the traffic thrice to see what we get.

    $ kubectl exec -it $SLEEP_POD -c sleep -- sh -c 'curl  [http://nginx:8000'](http://nginx:8000')
    This is version 1
    $ kubectl exec -it $SLEEP_POD -c sleep -- sh -c 'curl  [http://nginx:8000'](http://nginx:8000')
    This is version 1
    $ kubectl exec -it $SLEEP_POD -c sleep -- sh -c 'curl  [http://nginx:8000'](http://nginx:8000')
    This is version 1

We get “This is version 1” every time we send a request. Let’s check the logs for both services.

    $ kubectl logs $V1_POD -c nginx
    127.0.0.1 - - [13/May/2020:23:08:11 +0000] "GET / HTTP/1.1" 200 18 "-" "curl/7.35.0" "-"
    127.0.0.1 - - [13/May/2020:23:15:27 +0000] "GET / HTTP/1.1" 200 18 "-" "curl/7.35.0" "-"
    127.0.0.1 - - [13/May/2020:23:15:31 +0000] "GET / HTTP/1.1" 200 18 "-" "curl/7.35.0" "-"
    127.0.0.1 - - [13/May/2020:23:15:33 +0000] "GET / HTTP/1.1" 200 18 "-" "curl/7.35.0" "-"

    $ kubectl logs $V2_POD -c nginx
    127.0.0.1 - - [13/May/2020:23:15:27 +0000] "GET / HTTP/1.1" 200 18 "-" "curl/7.35.0" "10.4.2.9"
    127.0.0.1 - - [13/May/2020:23:15:31 +0000] "GET / HTTP/1.1" 200 18 "-" "curl/7.35.0" "10.4.2.9"
    127.0.0.1 - - [13/May/2020:23:15:33 +0000] "GET / HTTP/1.1" 200 18 "-" "curl/7.35.0" "10.4.2.9"

What happened? We got all responses from nginx:v1, but we see logs from v2 as well. If you look at the log timestamp, the logs at v2 match the logs at v1, and they occur at the same time. That shows that traffic mirroring is working fine.

## Conclusion

Thanks for reading! I hope you enjoyed the article

Traffic mirroring can help teams uncover errors that they usually can’t using traditional infrastructure. It is one of the most effective ways of doing operational acceptance testing of your releases. It saves you a lot of hassle and helps avoid customer incidents.

The next story is “[Locality-Based Load Balancing in Kubernetes Using Istio](https://medium.com/better-programming/locality-based-load-balancing-in-kubernetes-using-istio-a4a9defa05d3),” so see you there!
