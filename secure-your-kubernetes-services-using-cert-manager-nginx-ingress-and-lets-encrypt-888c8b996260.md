
# Secure your Kubernetes Services using Cert-Manager, NGINX Ingress and Let’s Encrypt



## **Introduction**

There is little denying that we live in the age of microservices and containers. Most organizations, for the right use-cases, are moving from a monolithic architecture to a microservice architecture. Microservices enable rapid development of complex applications by splitting a large monolithic application into loosely coupled services that coordinate with each other. And containers are very well suited to deploy and run these microservices. However, microservices or containers bring in a new set of challenges. These challenges are inherent in any distributed architecture — from interservice communication and transactions that span service boundaries to testing, deployment and monitoring. Enter [Kubernetes](https://kubernetes.io/). Kubernetes is an open source orchestration tool that handles the operational complexity of microservices deployed in containers. It helps in automating deployments, scaling targeted services and the general management of containers.

This is all great, but consider this: you have developed a microservices based application and deployed it on a Kubernetes cluster. You have also exposed your single page web application, backed by your microservices or REST APIs, on a public facing IP address. One of the next logical steps before going live is to enable HTTPS and tie your public IP to your registered domain. Thanks to [Let’s Encrypt](https://letsencrypt.org/), it is now easier than ever to acquire digital certificates (for free) and to enable HTTPS (SSL/TLS) for your services and websites. This article walks you through the configuration steps required to automatically enable TLS on your public Kubernetes services.

## **Prerequisites**

To follow the steps in this article, you will need the following:

* A kubernetes cluster version 1.8+ running somewhere, like on a cloud provider of your choice.
> Shameless promotion: you can sign-up for a free account on [Oracle Cloud](https://cloud.oracle.com/en_US/containers) to run a Kubernetes engine (OKE).

* The [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) CLI installed and configured to talk to your Kubernetes cluster. You can quickly check your current context by running this command:

    kubectl config current-context

* A cluster role binding to grant administrative rights to the user principal you use to run your kubectl commands. If you don’t have the proper rights, run this command (replace the user_id token):

    kubectl create clusterrolebinding cluster-admin-binding \
        --clusterrole cluster-admin \
        --user <user_id>

* One or more services running inside the Kubernetes cluster with the spec type set to “ClusterIP”. For reference, here’s a sample yaml file that was used to create a service named “my-webapp”:

    apiVersion: v1
    kind: Service
    metadata:
      name: my-webapp
      labels:
        app: my-webapp
        tier: frontend
    spec:
      type: ClusterIP
      ports:
      — port: 80
        targetPort: 8080
      selector:
        app: my-webapp
        tier: frontend

## **Step 1: Install Helm**

[Helm](https://helm.sh/) is an official kubernetes native package manager. You can use helm to install and upgrade kubernetes applications. Helm uses a concept called charts — a collection of files that describe the kubernetes resources required to install and run an application. If you are a mac/brew user, run the following command:

    brew install kubernetes-helm

To install Helm on a different OS or through other means, follow the instructions outlined in this Helm [documentation](https://docs.helm.sh/using_helm/#installing-helm).

## **Step 2: Install Tiller**

Tiller is the server side component of Helm that typically runs in your Kubernetes cluster. For the purposes of this article, we will install tiller on the system namespace with [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) enabled.

First, create a service account that will be used to run Tiller. Use the following command:

    kubectl create serviceaccount tiller --namespace kube-system

Next, create a yaml file named “tiller-clusterrolebinding.yaml” and add the following text to it:

    kind: ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
      name: tiller-clusterrolebinding
    subjects:
      — kind: ServiceAccount
        name: tiller
        namespace: kube-system
    roleRef:
      kind: ClusterRole
      name: cluster-admin
      apiGroup: ""

Now run the following command to create the cluster role binding and grant the “tiller” service account cluster wide admin rights:

    kubectl create -f tiller-clusterrolebinding.yaml

Finally, install Tiller by running the following command:

    helm init --service-account tiller

Optionally, verify that Tiller is installed and in service by running the following command and checking its output:

    kubectl get pods --namespace kube-system

## **Step 3: Install Cert-Manager**

[Cert-Manager](https://cert-manager.readthedocs.io/en/latest/index.html) is a kubernetes native certificate manager. One of the most significant features that Cert-Manager provides is its ability to automatically provision TLS certificates. Based on the annotations in a Kubernetes ingress resource, the cert-manager will talk to Let’s Encrypt and acquire a certificate on your service’s behalf.

Run the following command to install Cert-Manager:

    helm install \
      --name cert-manager \
      --namespace kube-system \
      --set ingressShim.defaultIssuerName=letsencrypt-staging \
      --set ingressShim.defaultIssuerKind=ClusterIssuer \
      stable/cert-manager \
      --version v0.3.0

Now let’s create a ClusterIssuer resource. A ClusterIssuer represents a certificate authority like Let’s Encrypt from which signed certificates can be obtained. At least one ClusterIssuer resource should be present in order for the certificate manager to begin issuing certificates.

Begin by creating a yaml file named “letsencrypt-staging.yaml” and add the following text to it:

    apiVersion: certmanager.k8s.io/v1alpha1
    kind: ClusterIssuer
    metadata:
      name: letsencrypt-staging
    spec:
      acme:
        # The ACME server URL
        server: [https://acme-staging-v02.api.letsencrypt.org/directory](https://acme-staging-v02.api.letsencrypt.org/directory)
        # Email address used for ACME registration
        email: <me@example.com>
        # Name of a secret used to store the ACME account private key
        privateKeySecretRef:
          name: letsencrypt-sec-staging
        # Enable HTTP01 validations
        http01: {}

Replace the value of the email field with the email address that you used to register your domain. Notice that we have configured an http challenge provider for this cluster issuer. You can read more about other challenges [here](http://letsencrypt.readthedocs.io/en/latest/challenges.html).

Now create the ClusterIssuer resource by running this command:

    kubectl create -f letsencrypt-staging.yaml
> Let’s Encrypt production service imposes strict limits. I suggest that you use their staging service to test things out. Once you are ready, create a new ClusterIssuer that points to production: [https://acme-v02.api.letsencrypt.org/directory](https://acme-v02.api.letsencrypt.org/directory)

## **Step 4: Install NGINX Ingress**

In the Kubernetes world, Ingress is an object that manages external access to services within a cluster. An Ingress resource provides load balancing and SSL termination. The [NGINX Ingress](https://kubernetes.github.io/ingress-nginx/) controller is based on the Ingress resource. We will configure this controller to act as an HTTPS load balancer and forward requests to specific services within the Kubernetes cluster.

Run the following command to install the NGINX Ingress controller into the system namespace with RBAC enabled:

    helm install stable/nginx-ingress \
      --name uck-nginx \
      --set rbac.create=true \
      --namespace kube-system

## **Step 5: Create Ingress**

Let’s create the Ingress resource and specify a rule to forward requests to the service “my-webapp”. We need to annotate the resource definition so that Cert-Manager can automate the process of acquiring the required TLS certificate from Let’s Encrypt.

Begin by creating a yaml file named “my-ingress.yaml” and add the following text to it:

    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: my-ingress
      annotations:
        kubernetes.io/ingress.class: nginx
        certmanager.k8s.io/cluster-issuer: letsencrypt-staging
        kubernetes.io/tls-acme: “true”
    spec:
      rules:
      — host: app.example.com
        http:
          paths:
          — path: /
            backend:
              serviceName: my-webapp
              servicePort: 80
      tls:
      — secretName: tls-staging-cert
        hosts:
        — app.example.com

And now, for the grand finale, create the Ingress resource by running the following command:

    kubectl create -f my-ingress.yaml

Sit back, relax and wait as Cert-Manager gets to work and acquires the certificate for your domain. Once it’s done, you can reach your service from the internet over HTTPS. With the sample Ingress above, that would be [https://app.example.com](https://app.example.com).

If you have multiple sub domains like console.example.com, api.example.com and so on, you can get a wildcard certificate to simplify things. For instance, in the yaml above, simply specify “*.example.com” as the single element in the “hosts” list under “tls”.

## **Conclusion**

Using Cert-Manager, NGINX Ingress and Let’s Encrypt streamlines the process of securely exposing your microservices that run on a Kubernetes cluster.

That’s all, folks.
> The views expressed in this article are my own and do not represent those of my employer.
