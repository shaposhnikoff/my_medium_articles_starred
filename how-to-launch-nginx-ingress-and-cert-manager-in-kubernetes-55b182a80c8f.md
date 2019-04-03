
# How to launch nginx-ingress and cert-manager in Kubernetes



*by Ivan Khramov*

Some time ago I needed to launch nginx-ingress and cert-manager in my Kubernetes cluster for obtaining Let’s Encrypt certificates,but it turned out it’s not that easy. I consulted several guides which didn’t help me solve the problem (and identify it in the first place). After I spent some time figuring out how to launch it, I dediced to share this instruction with you.

Let’s assume that you have a ready Kubernetes cluster and kubectl for cluster administration.

Previously, the key instrument for issuing and managing TLS certificates was kube-lego. But ultimately its developers announced “warning: kube-lego is in maintenance mode only. There is no plan to support any new features”, so I shifted to cert-manager.

Cert-manager does it Kubernetes way.
K8s allows creating custom resources, and cert-manager uses this feature to create 2 new types:
- Issuers
- Certificates

To get a fully functioning cert-manager, we have to fulfill the following steps:

1. Install helm

1. Install nginx-ingress using helm

1. Add a DNS record to connect your domain name to your IP address used by Ingress

1. Install cert-manager using helm

1. Create an Issuer for Let’s Encrypt staging

1. Create a test Ingress to make sure that certificates are issued correctly

1. Create Production-Let’s Encrypt Issuer

1. Consider creating ClusterIssuers

## Install helm

Helm is a package manager for Kubernetes. It allows you to install packages of pre-configured Kubernetes resources and publish them as charts.

If you have already installed and setup kubectl to access your cluster, you can easily install helm following this [instruction](https://docs.helm.sh/using_helm/#installing-helm).

## Install nginx-ingress

This is an optional step where I stumpled upon the first problem. If you already have a pre-installed ingress controller, you are lucky and can skip this step :)

There are several ingress controllers out there, e.g. Traefik or the GCP controller.

I used nginx-ingress for my cluster.
Install it using helm:

    helm install --namespace kube-system --name nginx-ingress stable/nginx-ingress --set rbac.create=true

Now we have nginx-ingress.
The problem was that by default it creates a type of service with LoadBalancer I do not use, and I spent several hours to understand why it wasn’t working the way I needed.

There are several solutions to this problem:

1. You can edit nginx-ingress-controller service, change its type to externalIP, and specify the required IPs (assigned to one of the nodes) and the ports — 80 and 443.

1. Or you can do what I did — delete the service, restart nginx-ingress-controller deploy as a daemonset and add hostNetwork: true to it. As a result, you’ll have nginx-ingress running on each node, occupying ports 80 and 443 on each node to obtain certificates.

So we have a ready nginx-ingress. Now let’s launch nginx-ingress in Kubernetes and get Let’s Encrypt certificates using cert-manager.

## Add a DNS record to connect your domain name to your IP address used by Ingress

I just added an A record *.ikhramov.me that leads to the external IP of the nodes in my cluster.

Check that the DNS directs where it should -

    $ dig test.ikhramov.me

You will see that this domain is binded to the correct IP.

At this step the ingress controller should respond to HTTP and HTTPS requests with default backend - 404

Let’s use httpie as an example and see what we get:

    $ http test.ikhramov.me
    HTTP/1.1 404 Not Found
    Connection: keep-alive
    Content-Length: 21
    Content-Type: text/plain; charset=utf-8
    Date: Wed, 21 Mar 2018 10:38:16 GMT
    Server: nginx/1.13.9
    Strict-Transport-Security: max-age=15724800; includeSubDomains;

    default backend - 404

## Install Cert Manager

The component which is responsible for obtaining and renewing certificates is cert-manager. To install it with helm run:

    $ helm install stable/cert-manager

To work with cert-manager we need an *Issuer* so that we can obtain TLS certificates.

The entity *Certificate* keeps information used to verify certificates for the current *Issuer *(Certificate Authority).
cert-manager communicates with *Issuer* (i.e. with Let’s Encrypt) to obtain a certificate and creates a *Secret* in K8s, containting the generated certificates. These certificates will be used by Ingress.

## Create Staging Issuer

Let’s Encrypt has two environments— staging and production. The staging environment issues certificates signed by ‘fake’ CAs, and has extended rate limits, so to ensure that everything works fine (and allow for some failures), I recommend using Let’s Encrypt *staging *api at first.

Create a file letsencrypt-staging.yaml with the following data:

    apiVersion: certmanager.k8s.io/v1alpha1
    kind: Issuer
    metadata:
      name: letsencrypt-staging
    spec:
      acme:
        # The ACME server URL
        server: https://acme-staging.api.letsencrypt.org/directory

        # Email address used for ACME registration
        email: "certificates@example.com"

        # Name of a secret used to store the ACME account private key
        privateKeySecretRef:
          name: letsencrypt-staging

        # Enable the HTTP-01 challenge provider
        http01: {}

Just change the email and add metadata — namespace, if necessary.

Now run kubectl create -f LetsEncrypt-staging.yaml, and here we are - we’ve created an *Issuer* in the default namespace.

## Create a test Ingress

Launch nginx in the default namespace and create a service for it.

Run kubectl create nginx --image nginx
and kubectl expose deploy nginx --port 80

You’ve just launched a pod with nginx in the default namespace with a service called nginx.

Create ingress.yaml

    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: test-ingress
      annotations:
        ingress.kubernetes.io/ssl-redirect: "true"
        kubernetes.io/tls-acme: "true"
        certmanager.k8s.io/issuer: letsencrypt-staging
        kubernetes.io/ingress.class: "nginx"
    spec:
      tls:
      - hosts:
        - test.ikhramov.me
        secretName: test-staging-letsencrypt
      rules:
      - host: test.ikhramov.me
        http:
          paths:
          - path: /
            backend:
              serviceName: nginx
              servicePort: 80

Run kubectl create -f ingress.yaml - we’ve just created the Ingress.
The certificate should be ready in about 30 seconds.

Now the whole traffic goes through https, and we’ve got certificates from Let’s Encrypt.

## Create Production Issuer

Creating a production *Issuer* is very simple.

Just create a new *Issuer* with another URL:

    apiVersion: certmanager.k8s.io/v1alpha1
    kind: Issuer
    metadata:
      name: letsencrypt-production
    spec:
      acme:
        # The ACME production api URL
        server: https://acme-v01.api.letsencrypt.org/directory

        # Email address used for ACME registration
        email: certificates@example.com

        # Name of a secret used to store the ACME account private key
        privateKeySecretRef:
          name: letsencrypt-production

        # Enable the HTTP-01 challenge provider
        http01: {}

That’s it, now when you create your Ingress, add the following annotation:

    certmanager.k8s.io/cluster-issuer: letsencrypt-production

That should work.

Now you can create an Ingress as in the previous step and test it.

## ClusterIssuer

You can also use ClusterIssuers.
The difference between just Issuers and ClusterIssuers is that the latter are created on a cluster lever, not namespace level.

If we have created an *Issuer* in the default namespace, corresponding Ingresses are also to be created in this namespace.

Create clusterissuer.yaml as follows:

    apiVersion: certmanager.k8s.io/v1alpha1
    kind: ClusterIssuer
    metadata:
      name: letsencrypt-prod
    spec:
      acme:
        email: main@ikhramov.me
        http01: {}
        privateKeySecretRef:
          key: ""
          name: letsencrypt-prod
        server: [https://acme-v01.api.letsencrypt.org/directory](https://acme-v01.api.letsencrypt.org/directory)

Now create it in K8s: kubectl create -f clusterissuer.yaml

Done. We have a ClusterIssuer called letsencrypt-prod

That enables us to create an Ingress in any namespace.
Just change 1 annotation -

    certmanager.k8s.io/cluster-issuer: letsencrypt-prod

Now you have a ready Ingress with certificates from Let’s Encrypt.

So, that’s it. I hope you find it useful and the steps are easy to follow. Please comment and/or ask questions if I missed something — I’d love to get your feedback. Also [follow us](https://twitter.com/@containerumcom) on Twitter and [join](https://t.me/joinchat/DHMHbEb8Q3EUpb3qWDzc5A) our Telegram chat to stay tuned!

Ivan Khramov, DevOps Enthusiast at [Containerum.com](http://containerum.com/)
