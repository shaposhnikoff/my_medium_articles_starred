
# How to Use Istio to Inject Faults to Troubleshoot Microservices in Kubernetes

Improve your microservices running on Kubernetes

![Photo by [Brooke Cagle](https://unsplash.com/@brookecagle?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/10670/1*MwXaQ7Mx8IEGILGwQishZg.jpeg)*Photo by [Brooke Cagle](https://unsplash.com/@brookecagle?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

Imagine you’ve deployed the reviews-v2 microservice into production, and you have an issue. Users are complaining they can’t see the reviews in your application intermittently.

You’ve run a thorough investigation in your development environment, but you’re not able to find what the problem is.

The only way to investigate the problem is by looking at the traffic flowing through your production services. Someone from your operations team notice timeouts in the chain, and you need to investigate further.

This story is a follow-up to “[Locality-Based Load Balancing in Kubernetes Using Istio](https://medium.com/better-programming/locality-based-load-balancing-in-kubernetes-using-istio-a4a9defa05d3).” Today, let’s discuss fault injection and troubleshooting.

## What’s Fault Injection?

*Fault injection,* in the context of Istio, is a mechanism by which we can purposefully inject some issues within our mesh to mimic how our application would behave in case it encounter such problems.

That can be a great tool to test your app for operational readiness and resilience.

There are two forms of faults you can inject within your application.

* **Delays: **You can add a time delay in a virtual service that allows you to mimic situations like processing delay or network overload

* **Aborts: **These are failures within your microservices, such as an HTTP 500 Internal Server Error or TCP connection failures. This enables you to test your application for potential error handling and see if a small fault causes your application to crash.

## Investigating the Problem

We understand there are some issues with the application, and users are unable to see reviews intermittently.

Since the app contains a long chain of microservices, the investigation requires isolating the problematic microservice and fixing the problem.

To isolate the problem, let’s add a synthetic fault in our chain. We need to purposefully inject a seven-second delay within our chain because the ops team have reported that requests time out within six seconds.

![Fault injection (delay)](https://cdn-images-1.medium.com/max/2074/1*ae7HqKjMII7tb37VB0U79Q.png)*Fault injection (delay)*

That enables us to recreate the issue in the environment and helps us isolate the problem.

But wait. You’re investigating in production. We don’t want to impact our end customers with this investigation. What can we do now? Well, let’s make use of the kind old Jason here.

### Prerequisites

Ensure you have a running Kubernetes cluster. Follow the “[How to Manage Traffic Using Istio on Kubernetes](https://medium.com/better-programming/how-to-manage-traffic-using-istio-on-kubernetes-cd4b96e00b57)” guide to set up user identity–based routing in our application.

## Injecting a Delay

To fix the issue tactically, we need to roll back the change we made. We have to revert the switch to v2 and make sure the live traffic is going to reviews-v1; however, for our business tester, jason, we still need to route the traffic to reviews-v2.

If you’ve been following the last story, we left at routing the requests to reviews-v2 for all requests originating from our business tester, jason. For the rest of the users, the traffic was going to reviews-v1. Therefore, let’s start from there.

Since we’re routing based on user identity, no one apart from one user would be impacted by this. Therefore, we’re safe from disturbing our customers, and we can investigate in the background.

Let’s have a look at the manifest we need to apply to inject the delay.

<iframe src="https://medium.com/media/6ebb668f4f396385c2a1d0108a4178a1" frameborder=0></iframe>

If you see the manifest for the user jason, it’s injecting a seven-second delay in the ratings microservice for 100% of the traffic. For the rest of the messages, there’s no delay.

Time to apply the change.

    $ kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-delay.yaml
    virtualservice.networking.istio.io/ratings configured

## Testing the Configuration

Now let’s access the site and see what happens.

![Public site](https://cdn-images-1.medium.com/max/3840/1*J7_P73PQ7cDnQaeCECqj2w.png)*Public site*

We can see the reviews but no ratings. We expected that, as we rolled back our change from v2 to v1. Log in as the user jason — no password is necessary.

![Logged in as ‘Jason’](https://cdn-images-1.medium.com/max/3840/1*3GQcVTyUdMDyRsrv9v5uDg.png)*Logged in as ‘Jason’*

Did you notice a delay in loading the page? Why are we unable to see the reviews? The timeout between reviews-v2 and ratings is a hardcoded value of 10 seconds. So even with a fault injection of seven seconds, you should see the reviews, but that’s certainly not the case.

If you open the developer tools and check the network tab, you’ll find that product page loads in around six secs. But you added a seven-second delay? What happened? Well, we have uncovered a bug here.

In this case, the timeout occurs between the productpage and the reviews microservice. The timeouts between them are hardcoded to 3s+3s(retry), which is six secs. So for a response taking more than six seconds from the reviews microservice, the productpage times out.

Congratulations! We’ve now figured out what the issue is. These kind of problems are prevalent with large applications using several microservices and also when multiple teams develop parts of the app and work in silos.

## Fixing the Problem

So how would we fix this issue? Well, there are two ways of doing that:

* Increase the productpage to reviews timeout

* Decrease the reviews to ratings timeout

Well, we don’t want to disturb the existing running productpage microservice. Therefore, let’s fix the app by deploying the reviews-v3 microservice. The v3 version fixes the problem by reducing the timeout to two-and-a-half seconds.

If you migrate all traffic to v3, like we did in the last story, and inject a two-second delay, you should find that issue no longer persists.

Apply the below manifest:

    $ kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-v3.yaml

Access the page, and you should see reviews-v3 running successfully.

![Reviews v3](https://cdn-images-1.medium.com/max/3840/1*tT2CBJK8ZntafqTtA92dag.png)*Reviews v3*

## Injecting Aborts

There are other ways to investigate and isolate the problem as well. For example, you can inject an HTTP fault within a virtual service to examine how your application behaves in those circumstances.

![Fault injection (abort)](https://cdn-images-1.medium.com/max/2182/1*hP3bKbWJeXrLLg1pqOj8bQ.png)*Fault injection (abort)*

For a demo, you can apply the below manifest to insert an HTTP 500 (Internal Server Error) fault within the ratings microservice.

<iframe src="https://medium.com/media/4a29e67e754296584875a9bbcbc6a22c" frameborder=0></iframe>

As you see in the YAML file, for the user jason, Istio injects an HTTP 500 fault for 100% of the traffic. The rest of the traffic is unaffected.

Apply the manifest:

    kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-abort.yaml

## Testing the Abort Injection

Access the page, and you’ll see you now get a “Rating service is currently unavailable” message.

![HTTP fault injection](https://cdn-images-1.medium.com/max/3840/1*8x-BWN90yeTd9B3P8ug1ug.png)*HTTP fault injection*

That shows that fault injection is working correctly, and the developer has coded the page well — as instead of a crash, we see a graceful error.

## Conclusion

Thanks for reading through! I hope you enjoyed the article. In the next part, I’ll discuss “[How to Visualise Your Istio Service Mesh on Kubernetes](https://medium.com/better-programming/how-to-visualise-your-istio-service-mesh-on-kubernetes-209c7b439a41)” with a hands-on demonstration, so see you in the next story!
