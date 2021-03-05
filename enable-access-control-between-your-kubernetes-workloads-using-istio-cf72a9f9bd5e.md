
# Enable Access Control Between Your Kubernetes Workloads Using Istio

A guide to Istio authorization between your microservices within Kubernetes

![Photo by [Ishan @seefromthesky](https://unsplash.com/@seefromthesky?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/roads?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/6168/1*pffSuk3T7wjlkp5izE-9nQ.jpeg)*Photo by [Ishan @seefromthesky](https://unsplash.com/@seefromthesky?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/roads?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

[Istio](https://istio.io) is one of the most popular service mesh technologies that you can run over Kubernetes to control your microservices effectively. Apart from allowing traffic management and visualization, Istio also provides a lot of fine-grained layer 7 security features for your Kubernetes workloads.

While you can use the [Kubernetes network policy](https://medium.com/better-programming/how-to-secure-kubernetes-using-network-policies-bbb940909364) to build a layer 3 firewall, it does not provide you with advanced security that is used by most modern firewalls and intrusion detection software.

Istio fills this gap by providing granular control for your workloads. It can allow or deny access based on parameters like message headers, HTTP method, and source service account.

Istio behaves exactly like modern firewalls and intrusion detection software, and it allows you to implement security the way you would achieve in a traditional infrastructure.

Istio is Kubernetes aware and, therefore, uses the inbuilt Kubernetes RBAC to ascertain identity. If the calling service is a Kubernetes workload, it can just use the Kubernetes service account to identify it and control access.

This story is a follow up to “[Enable Mutual TLS Authentication Between Your Kubernetes Workloads Using Istio](https://medium.com/better-programming/enable-mutual-tls-authentication-between-your-kubernetes-workloads-using-istio-65338c8adf82).” Today, let’s discuss how to enable access control between your Kubernetes microservices using Istio.

## Prerequisites

This story assumes that you have a basic understanding of how Kubernetes works and are aware of microservices and Istio. For a brief introduction to Istio, check out [How to Manage Microservices on Kubernetes With Istio](https://medium.com/better-programming/how-to-manage-microservices-on-kubernetes-with-istio-c25e97a60a59).

Ensure that you have a running Kubernetes Cluster. I have done the hands-on exercise in [Google’s Kubernetes Engine](https://cloud.google.com/kubernetes-engine), but it would work the same way on any other Kubernetes Cluster.

Install Istio in your Kubernetes Cluster and deploy the *Book Info* application by following the [Getting Started With Istio on Kubernetes](https://medium.com/better-programming/getting-started-with-istio-on-kubernetes-e582800121ea) guide.

## Configure the deny-all Policy

The starting point for any access control is to first implement a deny-all policy and then open connections as and when needed. Let’s create a default deny-all policy for the workloads by applying the following YAML.

<iframe src="https://medium.com/media/ea27eb2d27d0647f85f357b3fc0ec0f9" frameborder=0></iframe>

Open the product page of the Book Info application from the browser using [http://LOAD_BALANCER_IP/productpage](http://LOAD_BALANCER_IP/productpage.).

![RBAC Access Denied](https://cdn-images-1.medium.com/max/3840/1*8f3hSBxyVhmw-CuSW29YCQ.png)*RBAC Access Denied*

What happened? As expected, the deny-all policy is working as you are unable to view the page.

## Allow Clients to View the Product Page.

Now let’s create a policy for clients to view the product page via the browser by applying the below YAML

<iframe src="https://medium.com/media/1af6c2411044b02acc63764746271498" frameborder=0></iframe>

If you look at the YAML, it allows all traffic to the GET method of the product page.

Refresh the product page in the browser.

![Allow Product Page](https://cdn-images-1.medium.com/max/3840/1*i2cDuDXaW3KbNnoQOiunbQ.png)*Allow Product Page*

Well, you can now see the Book Info product page. However, you still see some errors. Why is that so? Well, although we have allowed clients to access the productpage microservice, the rest of the microservices are following the deny-all policy.

## Allow productpage Microservice to Access the Details and Reviews Microservice

The next logical step would be to allow access to the details microservice from the productpage microservice. Apply the below YAML file for that.

<iframe src="https://medium.com/media/b8c9fedec511183ecbcf6046cd491a47" frameborder=0></iframe>

If you observe the YAML, you see that the details microservice only allows traffic to the GET method from the source containing the service account cluster.local/ns/default/sa/bookinfo-productpage.

The productpage microservice uses this service account and should be able to connect.

Refresh the page once again, and you shall see that the details are now visible. However, we still need to work on reviews and ratings.

![Allow Details](https://cdn-images-1.medium.com/max/3840/1*OIubEy5Mx9h5VZualQjaOQ.png)*Allow Details*

Now, let’s allow the productpage microservice to call the reviews microservice.

<iframe src="https://medium.com/media/87b4c46f466a26365dbd152fab1d03a8" frameborder=0></iframe>

The above YAML is similar to the details-viewer policy and applies to the reviews microservice.

Refresh the product page again in the browser, and you should see that the reviews are visible as well. However, the ratings are still not available.

![Allow Ratings](https://cdn-images-1.medium.com/max/3840/1*jrIyzMOVGHMNYUIZCwcIXA.png)*Allow Ratings*

## Allow the Reviews Microservice to Access the Ratings Microservice

The next step in this chain is to allow traffic from the reviews microservice to the ratings microservice. Run the following to create a ratings-viewer policy.

<iframe src="https://medium.com/media/7b6df8527bb3a12ead1c59fef56d6900" frameborder=0></iframe>

As you see in the YAML, it applies to the ratings microservice and allows connection only from the workloads containing the bookinfo-reviews service account. The reviews microservice uses this service account.

Refresh the product page, and you should see that the ratings are visible as well.

![Allow Reviews](https://cdn-images-1.medium.com/max/3840/1*HqESMaSWjRftMTTKTPTQqA.png)*Allow Reviews*

You have successfully applied the policies on your microservices, and only the correct microservices can interact with each other on the right HTTP methods.

That enforces a proper traffic channel across your services and blocks unauthorized access to a microservice from another that it is not supposed to call.

## Conclusion

Thanks for reading! I hope you enjoyed the article.

In the next story “[How to Authorize Non-Kubernetes Clients With Istio on Your K8s Cluster](https://medium.com/better-programming/how-to-authorize-non-kubernetes-clients-with-istio-on-your-k8s-cluster-8a90fe95b137)”, I will discuss end-user authorization using [JSON Web Tokens](https://jwt.io/) with a hands-on demonstration, so, see you there!
