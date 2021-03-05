
# Securing Kubernetes Clusters with Istio and Auth0

Securing Kubernetes Clusters with Istio and Auth0

**TL;DR:** In this article, you will learn how to secure a Kubernetes cluster (and the applications that run on it) with Istio and Auth0. You will start by creating a brand-new cluster; then you will deploy an unsecured sample application; then, after testing the deployment, you will learn how to secure this application and its *pods* with Istio and Auth0. For reference, [you can find this application in this GitHub repository](https://github.com/rinormaloku/istio-auth0).
> [*“Learn how to secure a microservices application with Istio and Auth0.”](https://twitter.com/intent/tweet?text=%22Learn+how+to+secure+a+microservices+application+with+Istio+and+Auth0.%22%20via%20@auth0%20http://auth0.com/blog/securing-kubernetes-clusters-with-istio-and-auth0/)*
> [*TWEET THIS](https://twitter.com/intent/tweet?text=%22Learn+how+to+secure+a+microservices+application+with+Istio+and+Auth0.%22%20via%20@auth0%20http://auth0.com/blog/securing-kubernetes-clusters-with-istio-and-auth0/)*

## Preface

Security is the most important aspect *to get right* in every application. Failing to secure your apps and the identity of your users can be very expensive and can make customers and investors lose their faith in your ability to deliver high-quality services. That’s why, when developing an application, it’s of paramount importance to follow standards and best practices strictly. Luckily, there are big companies like [Auth0](https://auth0.com/), [Azure AD](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-whatis), [Facebook](https://facebook.com/), and [Google](https://google.com/) that can simplify this task by working as the identity providers of your apps. These companies, alongside with increased security, also bring to the table the benefit of enabling users to easily log into your apps without having to create yet another set of credentials.
> # Check out the [**Auth0 Blog](https://auth0.com/blog/?utm_source=medium&utm_medium=sc&utm_campaign=medium_test)** 🔐 and find everything you need to know about Identity Infrastructure, Access Management, SSO, JWT Authentication, and the latest in Security. 👉 [**AUTH0 BLOG](https://auth0.com/blog/?utm_source=medium&utm_medium=sc&utm_campaign=medium_test) **👈

Now, when it comes to microservices architecture implementing authentication and authorization can be even harder if you decide to implement it on every single microservice. The scenario can become even more problematic if you end up using different stacks to build these microservices. For each stack, you would have a different set of best practices and libraries to implement (probably even write), increasing the surface area of bugs to slip in and consuming company resources that could be invested in providing business value. You don’t want that, right?

To solve this problem, you will learn about Istio and about how to integrate it with Auth0. As you will see, by using one of [the authentication features provided by Istio](https://istio.io/docs/concepts/security/#authentication), you can easily avoid this problem and secure your applications just once.

## Prerequisites

Before starting to learn about Istio and how to use it, you will have to have admin access to a Kubernetes cluster. To get one, you can choose a service like [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine/), [Azure Kubernetes Service](https://azure.microsoft.com/en-us/services/kubernetes-service/), [Amazon Elastic Kubernetes Service (Amazon EKS)](https://aws.amazon.com/eks/), [OpenShift Kubernetes](https://www.openshift.com/learn/topics/kubernetes/) or [Kubernetes on DigitalOcean](https://www.digitalocean.com/products/kubernetes/). All of those providers have mature implementations, however, DigitalOcean is the easiest one to get started with, and it’s free for a good amount of time.

To avoid investing too much time on the prerequisites, if you don’t have a Kubernetes cluster available for you yet, you can [use this referral link to get a $100 USD, 60-day credit to spend on DigitalOcean resources](https://m.do.co/c/5c07f2e48a4d). With this amount, you will be able to run your new Kubernetes cluster for more than 500 hours (i.e., for more than 20 days).

Besides a cluster, you will also have to have kubectl installed and Helm (a package manager for Kubernetes). The next three subsections will show you how to get everything configured.

## Creating a Kubernetes Cluster on Digital Ocean

If you already have a Kubernetes cluster that you will use, you can skip this section. Otherwise, please, follow the instructions here to create your Kubernetes cluster on DigitalOcean. For starters, you will have to use [the referral link](https://m.do.co/c/5c07f2e48a4d) mentioned before. If you don’t use a referral link, you will end up paying from the very begin.

After using this link to create your account on DigitalOcean, you will get a confirmation email. Use the link sent to you to confirm your email address. Confirming your address will make DigitalOcean ask you for a credit card. Don’t worry about this. If you don’t spend more than $100 USD, they won’t charge you anything.

After inputting a valid credit card, you can use the next screen to create a *project*, or you can [use this link to skip this unnecessary step and to head to the Kubernetes dashboard](https://cloud.digitalocean.com/kubernetes/clusters). At the time of writing, DigitalOcean was offering Kubernetes on a *Limited Access* plan. If that’s still the case, don’t worry, this offer is robust enough for this tutorial.

![](https://cdn-images-1.medium.com/max/5200/0*0abxF4NWrHQ370gM.png)

From this screen, you can hit the *Create a Kubernetes cluster* button. Then, DigitalOcean will show you a new page with a form that you can fill like so:

* *Select a Kubernetes version*: The instructions on this article were tested with the 1.13.1-do.1 version. If you feel like testing other versions, feel free and go ahead. Just let us know how it went.

* *Choose a datacenter region*: Feel free to choose whatever region you prefer.

* *Add node pool(s)*: Make sure you have just one *node pool*, that you choose the $40/Month per node option, and that you have three nodes. Istio consumes a good amount of resources and, as such, you will need a robust cluster.

* *Add Tags*: Don’t worry about tagging anything.

* *Choose a name*: You can name your cluster whatever you want (e.g., “istio-tutorial”). Just make sure that the name is accepted by DigitalOcean (e.g., names can’t contain spaces).

![](https://cdn-images-1.medium.com/max/5200/0*ykRQ-CBHT-ggO9Pv.png)

After filling out this form, you can click on the *Create cluster* button. In a few minutes (roughly 4 mins), DigitalOcean will finish creating your cluster for you.

## Installing and Configuring Kubectl

As mentioned, besides a Kubernetes cluster, you will also need kubectl in your local machine to control this cluster. This tool, if you are not aware, is a Command-Line Interface (CLI) tool that enables you to manage Kubernetes clusters with ease from a terminal.

To install kubectl you can head to [the official documentation, and follow the instructions for your own operating system](https://kubernetes.io/docs/tasks/tools/install-kubectl/). After that, you will have to wait until your cluster is created by DigitalOcean, then you will have to download the cluster's config file. This file contains the credentials needed to act as the admin of the cluster, and you can find it on the cluster's dashboard. If you choose your new cluster on [your clusters' list](https://cloud.digitalocean.com/kubernetes/clusters), you will see its dashboard and, if you scroll to the bottom, a button called *Download Config File*. Click on this button to download the config file.

![](https://cdn-images-1.medium.com/max/5200/0*yO1BGVVU1h2njkg8.png)

When you finish downloading this file, open a terminal and move the file to the .kube directory in your home dir (you might have to create it):

    # make sure .kube exists mkdir ~/.kube # move the config file to it mv ~/Downloads/istio-tutorial-kubeconfig.yaml ~/.kube

If needed, adjust the last command with the correct path to the downloaded file.

The ~/.kube directory is a good place to keep your Kubernetes credentials. By default, kubectl will use a file named config, if it finds one in the .kube dir. To use a different file, you have three alternatives:

* First, you can specify another file by using the --kubeconfig flag in your kubectl commands, but this is too cumbersome.

* Second, you can define the KUBECONFIG environment variable to avoid having to type --kubeconfig all the time.

* Third, merge contexts in the same config file and then [you can switch contexts](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/#define-clusters-users-and-contexts).

The second option, setting the KUBECONFIG environment variable, is the easiest one, but feel free to choose another approach if you prefer. To set this environment, you can issue the following command:

    export KUBECONFIG=~/.kube/istio-tutorial-kubeconfig.yaml
> ***Note:** Your file path might be different. Make sure the command above contains the right path.*

Keep in mind that this command will set this environment only to this terminal session. If you open a new terminal, you will have to execute this command again. With kubectl properly installed and configured, the next thing you will do is to install Helm.

## Installing and Configuring Helm

Helm is a package manager for Kubernetes and will allow you to install and configure Istio in no time.

So, head to [the official Helm’s documentation](https://docs.helm.sh/using_helm/#installing-the-helm-client) and follow the instructions there to install this tool locally. There, you will be able to find specific instructions to install Helm on [Linux](https://docs.helm.sh/using_helm/#from-snap-linux), on [MacOS](https://docs.helm.sh/using_helm/#from-homebrew-macos), and on [Windows](https://docs.helm.sh/using_helm/#from-chocolatey-windows) too.

After installing Helm, you will need to initialize it in your Kubernetes cluster by executing the following command:

    helm init

Then, to check if this initialization worked properly, you can issue the following command:

    kubectl get pods --all-namespaces

This command will output a list of the *pods* running in your cluster, and one of these pods must be Tiller (this pod will have a name similar to tiller-deploy-6f4dbc6d67-wlqks).

![](https://cdn-images-1.medium.com/max/3592/0*ovS-VZ0e4vzi8NEy.png)
> ***Note:** if you are using Helm on anything other than a test environment (i.e., if you are running it on a production cluster), you must also check [the *Securing your Helm Installation* instructions](https://docs.helm.sh/using_helm/#securing-your-helm-installation).*

## Introduction to Istio

When building and managing microservice-based applications, a myriad of complexities arises. While using this architecture, you need to handle *service discovery*, *load balancing*, *application resilience*, and *hardware utilization*, to name just a few. When Google introduced Kubernetes, and the community saw that it provided solutions to these complexities, the technology was immediately well received and reached a wide adoption in all cloud computing service providers. However, Kubernetes lacked solutions to other problems that companies started facing recurrently while adopting microservices.

[Istio](https://istio.io/), [which is described by Google as an open source independent service mesh that provides the fundamentals you need to run a distributed microservice architecture successfully](https://cloud.google.com/istio/), enhances and adds additional features to Kubernetes to make maintaining microservice-based applications easy. For example, while dealing with microservices, Istio can help you, through a set of battle-tested solutions, deal with different aspects of your architecture like:

* **Traffic management**: Timeouts, retries, and canary releases.

* **Security:** End-user authentication and authorization,

* **Observability:** Tracing, monitoring, and logging.

Furthermore, [this is just a subset of Istio capabilities, as you can see here](https://istio.io/docs/). In this article, you will:

* be introduced to Istio;

* deploy Istio in a Kubernetes cluster;

* run a set of microservices that compose an application;

* and configure Auth0 in the service mesh to authenticate end users that want to use your app.

## Installing and Configuring Istio

Now that you have all the prerequisites configured and that you know what you implement here, the next thing to do is to install Istio in your Kubernetes cluster. Before that though, to easily identify Istio resources in your cluster, you will create and use a namespace called istio-system:

    kubectl create namespace istio-system

After creating this namespace, you will have to download Istio locally and use Helm to prepare a file with the Istio resources that you will need to install in your cluster:

    # download istio wget https://github.com/istio/istio/archive/1.0.5.zip # unzip it locally unzip 1.0.5.zip # move to Istio's directory cd istio-1.0.5 # use Helm to set it up on Kubernetes helm template install/kubernetes/helm/istio \ --namespace istio-system > istio.yaml
> ***Note:** At the time of writing, the latest Istio version to reach *General Availability* is 1.0.5 and that is the version used while the article was written. You can try newer versions if you will, but these are not guaranteed to work equally.*

Lastly, you can apply the generated resources from the istio.yaml file to your cluster by running the following command:

    kubectl apply -f istio.yaml

Although this command runs quite fast, it might take several seconds before Kubernetes finishes creating and starting all pods. To see the current status, you can issue the following command:

    kubectl get pods -n istio-system

The output of this command will be something similar to this:

    NAME READY STATUS RESTARTS AGE istio-citadel-649cd9445c-zgv7g 1/1 Running 0 67s istio-cleanup-secrets-xzsbl 0/1 Completed 0 97s istio-egressgateway-5d5657697b-jk8pr 1/1 Running 0 70s istio-galley-5ffd994c56-cwprl 1/1 Running 0 70s istio-ingressgateway-6b5b5998d5-6m6l2 1/1 Running 0 69s istio-pilot-786dc4c88d-wth25 2/2 Running 0 68s istio-policy-d4f59498f-tkvjq 2/2 Running 0 69s istio-security-post-install-9k2s4 0/1 Completed 2 96s istio-sidecar-injector-7779c4c94b-65nrs 1/1 Running 0 66s istio-telemetry-75469b9fc5-xclpp 2/2 Running 0 68s prometheus-76b7745b64-plvgb 1/1 Running 0 67s

When Kubernetes finishes creating and starts running your pods (that is, your pods contain the Running or the Completed status), you will be good to go! In the next section, you will get an application up and running so you can see in action how easy it is to secure a Kubernetes cluster with Istio and Auth0.

## Securing Kubernetes Clusters with Istio

After installing Istio in your cluster, it’s time to learn how to configure this service mesh to secure your microservices. However, to do that, you will need a couple of microservices running, right? Don’t worry, this won’t be time consuming, to speed up you will use a sample app provided by the Istio team.

This sample, [a book info application composed of four separate microservices, displays information about a book, similar to a single catalog entry of an online book store](https://istio.io/docs/examples/bookinfo/). This application is composed of the following microservices:

* productpage: The productpage microservice calls the details and reviews microservices to populate a page with book information.

* details: The details microservice contains details about the book.

* reviews: The reviews microservice contains reviews about the book. It also calls the ratings microservice.

* ratings: The ratings microservice contains book ranking information that accompanies a book review.

What is nice about this sample is that it uses four different stacks to compose the application:

* Python, which provides the product web pages that web browsers consume;

* Java, the language used to build the reviews microservice;

* Ruby, the language used to build the details microservice;

* and Node.js, the language used to build the ratings microservice.

The figure below illustrates how these microservices are organized and how they communicate:

![](https://cdn-images-1.medium.com/max/3828/0*_FMsjGl6I848H8-y.png)

## Running the Book Info Application

Now that you know what sample application you are going to use throughout the rest of the article, you will need to fetch its source code locally. To do so, use your terminal to clone [the GitHub repository that contains the book info application](https://github.com/auth0-blog/istio-auth0):

    # clone the sample app in a directory called istio-auth0 git clone https://github.com/auth0-blog/istio-auth0.git # move into it cd istio-auth0

After that, you will have to configure the namespace that you will use in your cluster (default in this case) to make Istio inject sidecar containers automatically:

    kubectl label namespace default istio-injection=enabled
> ***Note:** A sidecar, in this context, is a *container* that will be added to your pods. Istio will use these containers to intercept calls to your pod and to enhance them with its features.*

With this label in place, every pod that is deployed into the default namespace will get Istio's sidecar automatically injected. Now, you will have to execute the following command from the istio-auth0 directory to deploy the sample application:

    kubectl apply -f platform/kube/bookinfo.yaml

This command will finish executing quite fast and will generate an output similar to:

    service/details created deployment.extensions/details-v1 created service/ratings created deployment.extensions/ratings-v1 created service/reviews created deployment.extensions/reviews-v1 created deployment.extensions/reviews-v2 created deployment.extensions/reviews-v3 created service/productpage created deployment.extensions/productpage-v1 created

Although this command finishes quite fast, Kubernetes might need several seconds to run all the pods. To see the current status, you can execute the below command (now without the -n flag, which means that you want to see the status of pods running on the default namespace):

    kubectl get pods

Issuing this command will get you an output similar to:

    NAME READY STATUS RESTARTS AGE details-v1-64f56d694d-thq8t 2/2 Running 0 6m productpage-v1-b8cf99cd-mkwpt 2/2 Running 0 6m ratings-v1-8546767f5b-vxtzk 0/2 PodInitializing 0 6m reviews-v1-546d9f77ff-gq4cj 0/2 PodInitializing 0 6m reviews-v2-8fc76589f-sb9ln 0/2 PodInitializing 0 6m reviews-v3-7fdf5c754c-tml2s 0/2 PodInitializing 0 6m

You will know that you are good to go when all rows show 2/2 on the READY column and Running on the STATUS column. When you have this output, you will have an application up and running. However, to access it, you will need to allow incoming traffic.
> ***Note:** The READY column shows */2 because each pod (details-v1-*, productpage-v1-*, etc) contains two containers (as mentioned). One of them is a container added by the sample application, and the other is the Istio's container.*

## Enabling Application Access Through Istio Gateways

A best practice to control ingress traffic (incoming traffic) is to use [the *Istio Ingress Controller](https://istio.io/docs/tasks/traffic-management/ingress/)* and configure it using the [Istio Gateway resource](https://istio.io/docs/reference/config/istio.networking.v1alpha3/#Gateway). The controller was installed during Istio installation, and it positions itself at the edge of the cluster making sure Istio’s features (like monitoring, tracing, and configuring route rules) run in your cluster.

So, what’s left to do is to configure a Gateway resource. This resource will enable exposing ports and protocols in your cluster. For example, for your application, you will want to be able to access it using the HTTP protocol and through its default port (i.e., port 80). To achieve that, you will use a gateway defined like that (the repository you cloned already has this file, so don't worry about creating it):

    apiVersion: networking.istio.io/v1alpha3 kind: Gateway metadata: name: bookinfo-gateway spec: selector: istio: ingressgateway # use istio default controller servers: - port: number: 80 name: http protocol: HTTP hosts: - "*"

The selector istio: ingressgateway specifies to which *Ingress Gateway* the configuration will apply. In this case, the gateway will apply to a service that is labeled with istio: ingressgateway. If you run the following command on your terminal:

    kubectl get svc -n istio-system -l istio=ingressgateway

You will see that Istio created a service called istio-ingressgateway. And, as you ran the above command with -l istio=ingressgateway, you are confirming that this service is labeled with istio: ingressgateway.

So, if you run the following command from the sample application directory:

    # run from the istio-auth0 directory kubectl apply -f networking/bookinfo-gateway.yaml

You will install the gateway defined above in your cluster and, as this gateway uses the istio: ingressgateway selector, your istio-ingressgateway service (a load balancer) will be guarded by this gateway.

## Defining a Virtual Service for Your Application

A virtual service, in the Istio context, tells the Ingress Gateway on how to route the requests that arrive into your cluster. Without explicit rules telling how requests should be routed, your cluster will answer these requests with a 404 Not Found status.

In the following snippet, you can see a virtual service definition with explicit rules that tell to your cluster that /productpage, /login, /logout, /callback, and /api/v1/products are valid routes that must be redirected to the productpage microservice:

    apiVersion: networking.istio.io/v1alpha3 kind: VirtualService metadata: name: bookinfo spec: hosts: - "*" gateways: - bookinfo-gateway # 1 http: - match: - uri: exact: /productpage - uri: exact: /login - uri: exact: /logout - uri: exact: /callback - uri: prefix: /api/v1/products route: - destination: host: productpage # 2 port: number: 9080

Each one of these rules provides a specific functionality:

* /productpage: This is the main route that you will consume in your browser. This route will render the book info application.

* /login: This is the route that users will use to log in. After you integrate Auth0 in your app, this route will redirect users to the Auth0 login page so they can sign in or sign up.

* /logout: This is the route that users will use to log out from your app.

* /callback: This is the route that Auth0 will use to send your users back to your app after they sign in. Through this route, your application will get a code that it will be able to use to exchange for [access tokens](https://auth0.com/docs/tokens/overview-access-tokens).

* /api/v1/products: This is the route that the product page microservice will use to consume product details.

As the GitHub repository already contains this virtual service definition, you can issue the following command to apply it:

    # run from the istio-auth0 directory kubectl apply -f networking/bookinfo-virtualservice.yaml

After running this command, you will be able to use your application. To find the public IP address of your Kubernetes cluster, you can issue the following command:

    kubectl get svc \ -n istio-system istio-ingressgateway \ -o=jsonpath='{.status.loadBalancer.ingress[0].ip}'
> ***Note:** On the command above, you are using a Kubernetes feature called *JSONPath* to extract the exact property you want from your load balancer (its public IP address). [Learn more about the JSONPath feature here](https://kubernetes.io/docs/reference/kubectl/jsonpath/).*

The output of this command will be a public IP address (e.g., 174.138.124.123). Copy this IP, paste on a browser, append /productpage to it, and hit *enter*. This will make your product page application open in your browser.

![](https://cdn-images-1.medium.com/max/4504/0*xLS5dqXYJR6UVg14.png)

Keep in mind that this application is still unsecured and it does not support sign in and sign up yet. In the next sections, you will learn how to achieve this state.

## Authenticating Users with Istio and Auth0

Now that the sample application is up and running, you will learn how to use both Istio and Auth0 together to secure it. For starters, you will need to [sign up for a free Auth0 account](https://auth0.com/signup) (or you can use an existing one if you already have it).

After signing up, you will have to go to your Auth0 dashboard and create a new Auth0 Application. You can do that by going to [the *Applications* page](https://manage.auth0.com/#/applications) of your dashboard and by clicking on the *Create Application* button. When you click on this button, Auth0 will show you a dialog where you will have to input two things:

* *Application Name*: You can use anything here to identify your application (e.g., “Auth0 Istio Sample”).

* *Application Type*: As the product page is a classic web application (i.e., it is not a single-page app nor a native app), you will have to choose *Regular Web App* for this field.

Then, when you click on *Create*, Auth0 will redirect you to the *Quick Start* tab of your new application. From there, you can go to the *Settings* tab and change two fields on it:

* *Allowed Callback URLs*: Through this field, you will white label a URL that Auth0 will call after your users authenticate. Here, you can insert http://<YOUR-CLUSTER-PUBLIC-IP>/callback.

* *Allowed Logout URLs*: Through this field, you will white label a URL that Auth0 will call after your users log out. Here, you can insert http://<YOUR-CLUSTER-PUBLIC-IP>/logout.

Make sure that you replace <YOUR-CLUSTER-PUBLIC-IP> with the public IP address of your cluster while updating these fields, then hit the *Save Changes* button on the bottom of the *Settings* page.

![](https://cdn-images-1.medium.com/max/4700/0*cEaF13JM85VMTVlg.png)

Besides an Auth0 Application, you will need an Auth0 API. To create one, head to [the APIs section of your dashboard](https://manage.auth0.com/#/apis) and click on *Create API*. When you do so, Auth0 will show you a form where you will have to input three things:

* A *Name* for your API: For that, you can use something like “Auth0 Istio Sample” again.

* An *Identifier* for your API: For that, you can use a URI like https://bookinfo.io. This doesn't need to be a valid URL, nothing will call it as such.

* A *Signing Algorithm*: Make sure you use RS256 for this field. If you want to understand more about this, [check out this resource](https://community.auth0.com/t/jwt-signing-algorithms-rs256-vs-hs256/7720/2).

After filling out this form, click on the *Create* button. Then, head back to the *Application* section of your Auth0 dashboard and open the application you created before. Leave this page open as you will need to copy some values from it in no time.

After that, head back to your terminal, and use an editor to update the ./security/bookinfo-policy.yaml file to use your Auth0 domain. In this file, you will see two placeholders called {YOUR_DOMAIN}. To replace them, use the value presented in the *Domain* field of your Auth0 Application (e.g., istio-auth0.auth0.com).

    apiVersion: "authentication.istio.io/v1alpha1" kind: "Policy" metadata: name: bookinfo spec: targets: - name: reviews - name: ratings - name: details peers: - mtls: {} origins: - jwt: issuer: "https://{YOUR_DOMAIN}/" jwksUri: "https://{YOUR_DOMAIN}/.well-known/jwks.json" principalBinding: USE_ORIGIN

The booking-policy.yaml file (shown above), contains an Istio Policy definition that adds end-user authentication capabilities to some microservices. In this case, you are instructing Istio that you want this policy to apply to two services: reviews and details. Doing so will guarantee that services will require that requests:

1. are encrypted with mutual TLS and

1. contain valid access tokens issued by https://{YOUR_DOMAIN}/ and that these are checked against the keys defined on the https://{YOUR_DOMAIN}/.well-known/jwks.json URL.

Also, as you are not adding productpage to the targets property of this file, you are telling Istio to keep this service publicly available.

Now, to apply this policy, execute the following command:

    kubectl apply -f security/bookinfo-policy.yaml

Then, after a minute or so, if you refresh the /productpage URL in your browser, you will see that now it shows two "error fetching ..." messages. What this means is that your policy is up and running and that the productpage microservice was unable to fetch the product details and the product reviews because you are not signed in (i.e., you don't have access tokens).

![](https://cdn-images-1.medium.com/max/4592/0*hn3UgZlAlIvDuc04.png)

## Enabling Service to Service Authentication

Istio simplifies Service to Service authentication and secure communication using Mutual TLS. This is made simple with Destination Rules, which notify callers of a service to encrypt their traffic, achieved by the sample below:

    apiVersion: networking.istio.io/v1alpha3 kind: DestinationRule metadata: name: details spec: host: details trafficPolicy: tls: mode: ISTIO_MUTUAL subsets: - name: v1 labels: version: v1 - name: v2 labels: version: v2

DestinationRules for all the services are found in the repository, apply those by executing the command below:

    kubectl apply -f networking/destination-rule-mtls.yaml

Now your traffic is encrypted. Next step is end user authentication.

## Enabling Users to Authenticate Through Auth0

Right now, your productpage web application does not contain the endpoints necessary to allow users to sign in and sign out. The HTML page presented by this web application does have the links that will enable users to authenticate, but you will have to update the backend code to integrate with Auth0.

To do so, open the src/productpage/requirements.txt file, and add the following dependencies:

    authlib==0.10 six==1.11.0

Then, open the src/productpage/productpage.py file, and update it as follows (don't worry if you don't know Python, this will be quite easy):

    # ... other import statements ... from authlib.flask.client import OAuth from six.moves.urllib.parse import urlencode # ... # app = Flask(__name__) # ... # Bootstrap(app) AUTH0_CALLBACK_URL = "http://{YOUR-CLUSTER-PUBLIC-IP}/callback" AUTH0_CLIENT_ID = "{YOUR-APPLICATION-CLIENT-ID}" AUTH0_CLIENT_SECRET = "{YOUR-APPLICATION-CLIENT-SECRET}" AUTH0_DOMAIN = "{YOUR-AUTH0-DOMAIN}" AUTH0_BASE_URL = 'https://' + AUTH0_DOMAIN AUTH0_AUDIENCE = "{YOUR-AUDIENCE}" oauth = OAuth(app) auth0 = oauth.register( 'auth0', client_id=AUTH0_CLIENT_ID, client_secret=AUTH0_CLIENT_SECRET, api_base_url=AUTH0_BASE_URL, access_token_url=AUTH0_BASE_URL + '/oauth/token', authorize_url=AUTH0_BASE_URL + '/authorize', client_kwargs={ 'scope': 'openid profile', }, ) # ... leave the rest of the code untouched ...

In the new version of this file, you are using the imported OAuth library with your own Auth0 properties to handle the authentication process.
> ***Note:** You will have to replace the placeholders above with your own values. For the first placeholder, {YOUR-CLUSTER-PUBLIC-IP}, you can use the public IP address of your Kubernetes cluster. For the following three placeholders ({YOUR-APPLICATION-CLIENT-ID}, {YOUR-APPLICATION-CLIENT-SECRET}, {YOUR-AUTH0-DOMAIN}), you can use the properties of your Auth0 Application (*Client ID*, *Client Secret*, and *Domain*). For the last placeholder, {YOUR-AUDIENCE}, you will have to use the identifier that you gave to your Auth0 API.*

After adding the code above, you will (still on the same file) add the following endpoints:

    # ... leave the rest of the code untouched ... # find this route @app.route('/health') def health(): return 'Product page is healthy' # add the following endpoints underneath it @app.route('/login') def login(): return auth0.authorize_redirect(redirect_uri=AUTH0_CALLBACK_URL, audience=AUTH0_AUDIENCE) @app.route('/callback') def callback(): response = auth0.authorize_access_token() # 1 session['access_token'] = response['access_token'] # 2 userinfoResponse = auth0.get('userinfo') # 3 userinfo = userinfoResponse.json() session['user'] = userinfo['nickname'] # 4 return redirect('/productpage') @app.route('/logout') def logout(): session.clear() params = {'returnTo': url_for('front', _external=True), 'client_id': AUTH0_CLIENT_ID} return redirect(auth0.api_base_url + '/v2/logout?' + urlencode(params)) # ... other endpoints definition and code stay untouched ...

The first endpoint that you are adding, /login, will redirect users to the Auth0 login page. The second endpoint, /callback, will be accessed by users who are coming back from Auth0 after authenticating. The third endpoint, /logout, will redirect users to the log out endpoint at Auth0, then back to the productpage web app.

Notice that the /callback endpoint is responsible for:

1. Exchanging codes for access tokens

1. Save access tokens into your users’ sessions

1. Make an external request to Auth0 to get user information

1. Add your users’ nickname to their sessions
> ***Note:** The external request that gets users’ information from Auth0 will require you to configure an *Egress Rule* in your Istio configuration. You will learn about it in a bit.*

Lastly, still on the src/productpage/productpage.py file, you will have to update the getForwardHeaders function to include the access tokens you retrieve for your users. So, find this function definition and, after the headers = {} declaration, add the following code:

    def getForwardHeaders(request): headers = {} if 'access_token' in session: headers['Authorization'] = 'Bearer ' + session['access_token']

With that in place, the productpage web application will be able again to consume the other microservices. This is because you are adding access tokens to the Authorization headers that are forwarded to each microservice.

## Deploying the New Version of the Product Page

After refactoring the productpage application, you will have to make your Kubernetes cluster update its pods to run the new version of your app. To do so, the first thing you will have to do is to use the following docker commands to add a new image of the productpage to [Docker Hub](https://hub.docker.com/):

    # build the new version docker build -t {YOUR-DOCKER-HUB-USER}/productpage:istio-auth0 ./src/productpage # push the new image to Docker Hub docker push {YOUR-DOCKER-HUB-USER}/productpage:istio-auth0
> ***Note:** You will need an account at Docker Hub and you will need to replace {YOUR-DOCKER-HUB-USER} with your Docker Hub username. If you don't have a Docker Hub account, [follow the steps here](https://docs.docker.com/docker-hub/). If you don't have Docker installed in your machine, [follow the instructions here](https://docs.docker.com/install/#supported-platforms).*

Then, to make the productpage pod use this new image, you can issue the following command to open its definition:

    kubectl set image deployment/productpage-v1 \ productpage={YOUR-DOCKER-HUB-USER}/productpage:istio-auth0

This command is telling Kubernetes to update the productpage container of the productpage-v1 deployment to use the {YOUR-DOCKER-HUB-USER}/productpage:istio-auth0 image. Make sure you replace {YOUR-DOCKER-HUB-USER} with your Docker Hub username.

After a few seconds, your cluster will have replaced the productpage application to use the new image version available. As such, you will be able to refresh the productpage application page in your browser and hit to *Sign In* button. When you hit this button, you will be redirected to the Auth0 login page where you will be able to sign in. However, when you sign in, you will be redirected back to your application where you will see an error page saying requests.exceptions.ConnectionError. This is happening because, by default, Istio blocks unexpected outgoing requests (in this case, Istio is blocking your web application from communicating with Auth0 to get details about who logged in).

![](https://cdn-images-1.medium.com/max/4696/0*Gak1AfzYU5WbKvTn.png)

The next section will teach you how to fix that.

## Defining an Istio Egress

By default, the applications that Istio is controlling in your cluster are unable to communicate with URLs outside of the cluster. Therefore, your application won’t be able to fetch user information from Auth0. To circumvent this problem, you will use two new Istio resources: a *Service Entry* that will allow your application to issue requests to your Auth0 domain, and a *Virtual Service* that will allow this communication to use TLS (i.e., to be secured).

Both resources are defined in the networking/auth0-egress.yaml file of the repository you cloned. So, open this file, and replace all {YOUR_DOMAIN} placeholders with your own Auth0 domain. In case you forgot what that is, you can copy it from the *Domain* field of your Auth0 Application (e.g., istio-auth0.auth0.com).

After replacing these placeholders and saving this file, issue the following command from the terminal:

    kubectl apply -f networking/auth0-egress.yaml

Now, if you refresh the page that shows the requests.exceptions.ConnectionError error, you will see that your application is able to fetch your details from Auth0 and that the page is loaded properly again. Also, you will notice that the app now shows a nickname for you (e.g., rinor.maloku).

![](https://cdn-images-1.medium.com/max/5200/0*glvH1H8_m3gVLVrK.png)

That’s it! You just finished securing a microservices application with Istio and Auth0. How fun!?
> [*“I just learned how to secure a microservices application that is running on Kubernetes with Istio and Auth0.”](https://twitter.com/intent/tweet?text=%22I+just+learned+how+to+secure+a+microservices+application+that+is+running+on+Kubernetes+with+Istio+and+Auth0.%22%20via%20@auth0%20http://auth0.com/blog/securing-kubernetes-clusters-with-istio-and-auth0/)*
> [*TWEET THIS](https://twitter.com/intent/tweet?text=%22I+just+learned+how+to+secure+a+microservices+application+that+is+running+on+Kubernetes+with+Istio+and+Auth0.%22%20via%20@auth0%20http://auth0.com/blog/securing-kubernetes-clusters-with-istio-and-auth0/)*

## Conclusion

In this article, you learned how to use Istio and Auth0 together to secure a microservices application. You started by creating a Kubernetes cluster. Then, you learned how to configure Istio in your cluster. After that, you deploy an unsecured sample application. In the end, you learned how to secure this sample with Istio and Auth0.
> *I am Rinor Maloku and I want to thank you for joining me on this voyage. Since you read this far I know that you loved this article and would be interested in more. I write articles that go into this depth of detail for new technologies every 3 months. You can always expect an example application, hands-on practice, and a guide that provides you with the right tools and knowledge to tackle any real-world project.*
> *To stay in touch and not miss any of my articles subscribe to my [newsletter](https://tinyletter.com/rinormaloku), follow me on [Twitter](https://twitter.com/rinormaloku), and checkout my page [rinormaloku.com](https://rinormaloku.com/).*

*Originally published at [auth0.com](https://auth0.com/blog/securing-kubernetes-clusters-with-istio-and-auth0/).*
