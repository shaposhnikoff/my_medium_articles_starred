
# Automating Multi-Environment Kubernetes Virtual Clusters with Cloud DNS and Istio

Kubernetes supports multiple virtual clusters within the same physical cluster. These virtual clusters are called Namespaces. Namespaces are a way to divide cluster resources between multiple users. Many enterprises use Namespaces to divide the same physical Kubernetes cluster into different virtual software development environments as part of their overall Software Development Lifecycle (SDLC). This practice is commonly used in ‘lower environments’ or ‘non-prod’ (not Production) environments. These environments commonly include Continous Integration and Delivery (CI/CD), Development, Integration, Testing/Quality Assurance (QA), User Acceptance Testing (UAT), Staging, Demo, and Hotfix. Namespaces provide a basic form of what is referred to as soft multi-tenancy.

Generally, the security boundaries and performance requirements between non-prod environments, within the same enterprise, are less restrictive than Production or Disaster Recovery (DR) environments. This allows for multi-tenant environments, while Production and DR are normally single-tenant environments. In order to approximate the performance characteristics of Production, the Performance Testing environment is also often isolated to a single-tenant. A typical enterprise would minimally have a non-prod, performance, production, and DR environment.

Using Namespaces to create virtual separation on the same physical Kubernetes cluster provides enterprises with more efficient use of virtual compute resources, reduces Cloud costs, eases the management burden, and often expedites and simplifies the release process.

## Demonstration

In this post, we will re-examine the topic of virtual clusters, similar to the recent post, [Managing Applications Across Multiple Kubernetes Environments with Istio: Part 1](http://programmaticponderings.com/2018/04/13/managing-applications-across-multiple-kubernetes-environments-with-istio-part-1/) and [Part 2](http://programmaticponderings.com/2018/04/17/managing-applications-across-multiple-kubernetes-environments-with-istio-part-2/). We will focus specifically on automating the creation of the virtual clusters on GKE with Istio 1.0, managing the Google Cloud DNS records associated with the cluster’s environments, and enabling both HTTPS and token-based OAuth access to each environment. We will use the Storefront API for our demonstration, featured in the previous three posts, including [Building a Microservices Platform with Confluent Cloud, MongoDB Atlas, Istio, and Google Kubernetes Engine](http://programmaticponderings.com/2018/12/28/building-a-microservices-platform-with-confluent-cloud-mongodb-atlas-istio-and-google-kubernetes-engine/).

![[Click to Enlarge](https://programmaticponderings.files.wordpress.com/2019/01/gke-routing-5.png)](https://cdn-images-1.medium.com/max/2000/0*J4omBDl6x4Y4ot3U)*[Click to Enlarge](https://programmaticponderings.files.wordpress.com/2019/01/gke-routing-5.png)*

## Source Code

The source code for this post may be found on the gke branch of the [storefront-kafka-docker](https://github.com/garystafford/storefront-kafka-docker/tree/gke) GitHub repository.

    git clone --branch gke --single-branch --depth 1 --no-tags \
      [https://github.com/garystafford/storefront-kafka-docker.git](https://github.com/garystafford/storefront-kafka-docker.git)

Source code samples in this post are displayed as GitHub [Gists](https://help.github.com/articles/about-gists/), which may not display correctly on all mobile and social media browsers, such as LinkedIn.

This project contains all the code to deploy and configure the GKE cluster and Kubernetes resources.

![](https://cdn-images-1.medium.com/max/2000/0*hXonpwnLXF83kd8r)

To follow along, you will need to register your own domain, arrange for an Auth0, or alternative, authentication and authorization service, and obtain an SSL/TLS certificate.

## SSL/TLS Wildcard Certificate

In the recent post, [Securing Your Istio Ingress Gateway with HTTPS](http://programmaticponderings.com/2019/01/03/securing-your-istio-gateway-with-https/), we examined how to create and apply an SSL/TLS certificate to our GKE cluster, to secure communications. Although we are only creating a non-prod cluster, it is more and more common to use SSL/TLS everywhere, especially in the Cloud. For this post, I have registered a single wildcard certificate, *.api.storefront-demo.com. This certificate will cover the three second-level subdomains associated with the virtual clusters: dev.api.storefront-demo.com, test.api.storefront-demo.com, and uat.api.storefront-demo.com. Setting the environment name, such as dev.*, as the second-level subdomain of my storefront-demo domain, following the first level api.* subdomain, makes the use of a wildcard certificate much easier.

![](https://cdn-images-1.medium.com/max/2000/0*Oj5NhNfHq49Zi-bW)

As shown below, my wildcard certificate contains the Subject Name and [Subject Alternative Name](https://en.wikipedia.org/wiki/Subject_Alternative_Name) (SAN) of *.api.storefront-demo.com. For Production, api.storefront-demo.com, I prefer to use a separate certificate.

![](https://cdn-images-1.medium.com/max/2000/0*WdTQSC8jfj8bk-wM)

## Create GKE Cluster

With your certificate in hand, create the non-prod Kubernetes cluster. Below, the script creates a minimally-sized, three-node, multi-zone GKE cluster, running on GCP, with [Kubernetes Engine](https://cloud.google.com/kubernetes-engine/) cluster version 1.11.5-gke.5 and [Istio on GKE](https://cloud.google.com/istio/docs/istio-on-gke/overview) version 1.0.3-gke.0. I have enabled the master authorized networks option to secure my GKE cluster master endpoint. For the demo, you can add your own IP address CIDR on line 9 (i.e. 1.2.3.4/32), or remove lines 30–31 to remove the restriction ([*gist](https://gist.github.com/garystafford/d11b83b4bfee6ba57bfe0fe91277b8af)*).

* Lines 16–39: Create a 3-node, multi-zone GKE cluster with Istio;

* Line 48: Creates three non-prod Namespaces: dev, test, and uat;

* Lines 51–53: Enable Istio automatic sidecar injection within each Namespace;

<iframe src="https://medium.com/media/6be5ff26c330fb055ae62bfd7c5c0684" frameborder=0></iframe>

If successful, the results should look similar to the output, below.

![](https://cdn-images-1.medium.com/max/2000/0*YXcGhjMH24v9hsvy)

The cluster will contain a pool of three minimally-sized VMs, the Kubernetes nodes.

![](https://cdn-images-1.medium.com/max/2000/0*c5Ngg7PWRDN0HOpP)

## Deploying Resources

The Istio Gateway and three ServiceEntry resources are the primary resources responsible for routing the traffic from the ingress router to the Services, within the multiple Namespaces. Both of these resource types are new to Istio 1.0 ([*gist](https://gist.github.com/garystafford/0d03ebbb02480893d679bd95a43ce25f)*).

* Lines 9–16: Port config that only accepts HTTPS traffic on port 443 using TLS;

* Lines 18–20: The three subdomains being routed to the non-prod GKE cluster;

* Lines 28, 63, 98: The three subdomains being routed to the non-prod GKE cluster;

* Lines 39, 47, 65, 74, 82, 90, 109, 117, 125: Routing to FQDN of Storefront API Services within the three Namespaces;

<iframe src="https://medium.com/media/bb565e758790c5e0923f32c73e400425" frameborder=0></iframe>

Next, deploy the Istio and Kubernetes resources to the new GKE cluster. For the sake of brevity, we will deploy the same number of instances and the same version of each the three Storefront API services (Accounts, Orders, Fulfillment) to each of the three non-prod environments (dev, test, uat). In reality, you would have varying numbers of instances of each service, and each environment would contain progressive versions of each service, as part of the SDLC of each microservice([*gist](https://gist.github.com/garystafford/3666b4f47a22de34eca2db0a5ff8e8e6#file-part2_deploy_resources-sh)*).

* Lines 13–14: Deploy the SSL/TLS certificate and the private key;

* Line 17: Deploy the Istio Gateway and three ServiceEntry resources;

* Lines 20–22: Deploy the Istio Authentication Policy resources each Namespace;

* Lines 26–37: Deploy the same set of resources to the dev, test, and uat Namespaces;

<iframe src="https://medium.com/media/5ede459450a8d2f5db3cc0d8dd92dc1a" frameborder=0></iframe>

The deployed Storefront API Services should look as follows.

![](https://cdn-images-1.medium.com/max/2000/0*wXBBJa6rfTRDcdCD)

## Google Cloud DNS

Next, we need to enable DNS access to the GKE cluster using Google Cloud DNS. According to [Google](https://cloud.google.com/dns/), Cloud DNS is a scalable, reliable and managed authoritative [Domain Name System](https://en.wikipedia.org/wiki/Domain_Name_System) (DNS) service running on the same infrastructure as Google. It has low latency, high availability, and is a cost-effective way to make your applications and services available to your users.

Whenever a new GKE cluster is created, a new [Network Load Balancer](https://cloud.google.com/load-balancing/docs/network/) is also created. By default, the load balancer’s front-end is an external IP address.

![](https://cdn-images-1.medium.com/max/2000/0*zsLcIJ-Lqp8z4TAc)

Using a forwarding rule, traffic directed at the external IP address is redirected to the load balancer’s back-end. The load balancer’s back-end is comprised of three VM instances, which are the three Kubernete nodes in the GKE cluster.

![](https://cdn-images-1.medium.com/max/2000/0*HjMoOEClRbdPWQ6i)

If you are following along with this post’s demonstration, we will assume you have a domain registered and configured with Google Cloud DNS. I am using the storefront-demo.com domain, which I have used in the last three posts to demonstrate Istio and GKE.

Google Cloud DNS has a fully functional web console, part of the [Google Cloud Console](https://console.cloud.google.com/). However, using the Cloud DNS web console is impractical in a DevOps CI/CD workflow, where Kubernetes clusters, Namespaces, and Workloads are ephemeral. Therefore we will use the following script. Within the script, we reset the IP address associated with the A records for each non-prod subdomains associated with storefront-demo.com domain ([*gist](https://gist.github.com/garystafford/4ecefb60901b610248f39f2de1b9b93b)*).

* Lines 23–25: Find the previous load balancer’s front-end IP address;

* Lines 27–29: Find the new load balancer’s front-end IP address;

* Line 35: Start the Cloud DNS transaction;

* Lines 37–47: Add the DNS record changes to the transaction;

* Line 49: Execute the Cloud DNS transaction;

<iframe src="https://medium.com/media/a809347b01e4d68ffb0b1e3db7852745" frameborder=0></iframe>

The outcome of the script is shown below. Note how changes are executed as part of a transaction, by automatically creating a transaction.yaml file. The file contains the six DNS changes, three additions and three deletions. The command executes the transaction and then deletes the transaction.yaml file.

    > sh ./part3_set_cloud_dns.sh

    Old LB IP Address: 35.193.208.115
    New LB IP Address: 35.238.196.231

    Transaction started [transaction.yaml].

    **dev.api.storefront-demo.com.
    **Record removal appended to transaction at [transaction.yaml].
    Record addition appended to transaction at [transaction.yaml].

    **test.api.storefront-demo.com.
    **Record removal appended to transaction at [transaction.yaml].
    Record addition appended to transaction at [transaction.yaml].

    **uat.api.storefront-demo.com.
    **Record removal appended to transaction at [transaction.yaml].
    Record addition appended to transaction at [transaction.yaml].

    Executed transaction [transaction.yaml] for managed-zone [storefront-demo-com-zone].
    Created [https://www.googleapis.com/dns/v1/projects/gke-confluent-atlas/managedZones/storefront-demo-com-zone/changes/53].

    ID  START_TIME                STATUS
    55  2019-01-16T04:54:14.984Z  pending

Based on my own domain and cluster details, the transaction.yaml file looks as follows. Again, note the six DNS changes, three additions, followed by three deletions ([*gist](https://gist.github.com/garystafford/c0644c5af7685407e986021e2d40a8e4)*).

<iframe src="https://medium.com/media/1e50a9a7bffa3799accab9391ddb8ade" frameborder=0></iframe>

## Confirm DNS Changes

Use the dig command to confirm the DNS records are now correct and that [DNS propagation](https://www.namecheap.com/support/knowledgebase/article.aspx/9622/10/dns-propagation--explained#dnspropagation) has occurred. The IP address returned by dig should be the external IP address assigned to the front-end of the Google Cloud Load Balancer.

    > dig dev.api.storefront-demo.com +short
    **35.238.196.231**

Or, all the three records.

    echo \
      "dev.api.storefront-demo.com\n" \
      "test.api.storefront-demo.com\n" \
      "uat.api.storefront-demo.com" \
      > records.txt | dig -f records.txt +short

    **35.238.196.231**
    **35.238.196.231**
    **35.238.196.231**

Optionally, more verbosely by removing the +short option.

    > dig +nocmd dev.api.storefront-demo.com

    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 30763
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 512
    ;; QUESTION SECTION:
    ;dev.api.storefront-demo.com.   IN  A

    ;; ANSWER SECTION:
    **dev.api.storefront-demo.com. 299 IN A   35.238.196.231**

    ;; Query time: 27 msec
    ;; SERVER: 8.8.8.8#53(8.8.8.8)
    ;; WHEN: Wed Jan 16 18:00:49 EST 2019
    ;; MSG SIZE  rcvd: 72

The resulting records in the Google Cloud DNS management console should look as follows.

![](https://cdn-images-1.medium.com/max/2000/0*7mqN8HbQoVUm3svv)

## JWT-based Authentication

As discussed in the previous post, [Istio End-User Authentication for Kubernetes using JSON Web Tokens (JWT) and Auth0](http://programmaticponderings.com/2019/01/06/securing-kubernetes-withistio-end-user-authentication-using-json-web-tokens-jwt/), it is typical to limit restrict access to the Kubernetes cluster, Namespaces within the cluster, or Services running within Namespaces to end-users, whether they are humans or other applications. In that previous post, we saw an example of applying a machine-to-machine (M2M) Istio Authentication Policy to only the uat Namespace. This scenario is common when you want to control access to resources in non-production environments, such as UAT, to outside test teams, accessing the uat Namespace through an external application. To simulate this scenario, we will apply the following Istio Authentication Policy to the uat Namespace. ([*gist](https://gist.github.com/garystafford/9b053ccbd9eddefa6bf43515becd2eb6)*).

<iframe src="https://medium.com/media/92a432af6b32b8fa7b0c4f094d94728f" frameborder=0></iframe>

For the dev and test Namespaces, we will apply an additional, different Istio Authentication Policy. This policy will protect against the possibility of dev and test M2M API consumers interfering with uat M2M API consumers and vice-versa. Below is the dev and test version of the Policy ([*gist](https://gist.github.com/garystafford/9d264886882707f33646a7e2d1eab7e9)*).

<iframe src="https://medium.com/media/e5d294d10149af6b1119266fc349683d" frameborder=0></iframe>

## Testing Authentication

Using Postman, with the ‘Bearer Token’ type authentication method, as detailed in the previous post, a call a Storefront API resource in the uat Namespace should succeed. This also confirms DNS and HTTPS are working properly.

![](https://cdn-images-1.medium.com/max/2000/0*viE4qoSUI1osmd_-)

The dev and test Namespaces require different authentication. Trying to use no Authentication, or authenticating as a UAT API consumer, will result in a 401 Unauthorized HTTP status, along with the Origin authentication failed. error message.

![](https://cdn-images-1.medium.com/max/2000/0*5deWTwI5wKC9cgA5)

## Conclusion

In this brief post, we demonstrated how to create a GKE cluster with Istio 1.0.x, containing three virtual clusters, or Namespaces. Each Namespace represents an environment, which is part of an application’s SDLC. We enforced HTTP over TLS (HTTPS) using a wildcard SSL/TLS certificate. We also enforced end-user authentication using JWT-based OAuth 2.0 with Auth0. Lastly, we provided user-friendly DNS routing to each environment, using Google Cloud DNS. Short of a fully managed API Gateway, like [Apigee](https://apigee.com/api-management/), and automating the execution of the scripts with Jenkins or [Spinnaker](https://www.spinnaker.io/), this cluster is ready to provide a functional path to Production for developing our Storefront API.

*All opinions expressed in this post are my own and not necessarily the views of my current or past employers or their clients.*

*Originally published at [programmaticponderings.com](https://wp.me/p1RD28-6a6) on January 12, 2019.*
