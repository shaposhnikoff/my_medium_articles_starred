Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m103[39m, end: [33m114[39m }

# Set Up HTTPS on Kubernetes

Add security to your containers

![Photo by [Philipp Katzenberger](https://unsplash.com/@fantasyflip?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/security?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/12000/1*oeYC1R1pqPOjb__v-0Sh9g.jpeg)*Photo by [Philipp Katzenberger](https://unsplash.com/@fantasyflip?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/security?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

## Introduction

In my [last article](https://medium.com/@puneeth1994/a-beginners-guide-to-docker-3167e635f857), I spoke about how you can build Docker images and run containers.

While it would make a good read for beginners, I’d like to focus on a problem that bugged me for weeks while setting up HTTPS on a Kubernetes cluster mainly because of how scattered the docs are.

[Cert manager](https://docs.cert-manager.io/) is a native add-on in Kubernetes that can be used for managing certificates. We will be using [Let’s Encrypt](https://letsencrypt.org/) as the certificate issuer.

For the purpose of this tutorial, we’ll be using a tool called [*Helm](https://helm.sh/)*. Helm will act as our package manager and helps us in installing the certificate manager on our cluster.

## Assumptions

In this article, I’ll be assuming that you have a working setup of a Kubernetes cluster and have linked at least one domain to it or have an idea about how the process of routing external traffic to services in your cluster works.

If not, I highly recommend going through the following link to understand how [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) helps us in [routing traffic](https://kubernetes.github.io/ingress-nginx/).

## What Is HTTPS and Why Do We Need It?

HTTPS is a way to communicate over a computer network securely. It is an extension of the standard HTTP protocol.

HTTP is a simple request-response protocol where a client requests data and a server responds to the client with the requested data and a status code indicating the type of response.

The problem with this approach is that the data sent over is clear text and can be intercepted by an unknown malicious party. This can be a problem if the data being exchanged between the two parties is sensitive, such as credit card information.

This is where HTTPS comes into the picture. There are two main reasons why HTTPS exists.

1. Verify that the web address corresponding to the server is correct (i.e., making sure that you don’t access a fake website portraying itself as the website that you’re trying to access).

1. Ensures that only you can read what the server is sending you and vice versa (i.e., data encryption).

HTTPS achieves the above using SSL.

SSL (Secure Socket Layer) is the standard security technology for establishing an encrypted link between the two systems. These can be browser to the server, server to server, or client to server.

Basically, SSL ensures that the data transfer between the two systems remains encrypted and private.

The image below shows how communicating through HTTP and HTTPS differs with encryption.

![](https://cdn-images-1.medium.com/max/2000/1*afrLfsAPgAe7qzRinHXH3A.png)

## Cert manager

This is primarily responsible for obtaining new certificates from the certificate authority and responding to new challenges by Let’s Encrypt.

Challenges act as a way to validate your ownership over a domain. This challenge can be as simple as putting a TXT record under a domain name or providing Let’s Encrypt with a way to retrieve a file with a specific token in it from your webserver.

Cert manager itself contains multiple components including certificate objects, issuers, orders, and so on. This is where Helm will help us.

Helm takes care of manually creating all the Kubernetes objects with a minimal configuration for our use case.

This would also work in our favor when we want to clean up deployments. We don’t have to delete every single object individually as it takes care of cleanup with a single command.

## Installing Helm and cert manager

Helm can be installed from a [very basic script](https://helm.sh/docs/intro/install/#from-script), this will install the latest version of Helm on your local machine.

Once Helm is installed, we will need to [install the cert manager](https://cert-manager.io/docs/installation/kubernetes/#installing-with-helm) before requesting new certificates.

Follow the steps as described in the link above under the “installing with Helm” section.

Note: We aren’t focussing on the steps for installation of Helm and cert manager as the docs tend to change a lot. It would be better to refer to the official links for an updated way of installing them.

## Requesting New Certificates for a Domain

Requests to Let’s Encrypt to obtain a new certificate involves two steps:

1. First, the agent proves to the CA (Let’s Encrypt) that the web server controls a domain.

1. The agent can then request, renew, and revoke certificates for that domain.

Once the cert manager has been installed, we need to create two objects:

1. ClusterIssuer — They represent certificate authorities and specify servers where the certificates need to be requested.

1. Certificate — This object contains a reference to the issuer and the DNS name/names for which the certificates need to be obtained.

A cluster issuer object can be created with a basic set of configs as shown below.

<iframe src="https://medium.com/media/4a9e1517e31e609d6ad4138a27dac27f" frameborder=0></iframe>

As seen in the above object, we specify the server from which the certificate needs to be requested.

A certificate object is as shown below:

<iframe src="https://medium.com/media/8951e13cbcb6b84cabbf588ba603dc8a" frameborder=0></iframe>

The certificate refers to our cluster issuer via its name.

Create YAML files for both the objects with the names certificate and issuer and execute the command below to create the objects.

    Kubectl apply -f certificate.yaml issuer.yaml

This will create the certificates and challenges and we can automate our certificate renewal using the renewBefore field. Any number of domains can be added to the certificate object for the issue of certificates.

## References

* You can read up on more of the individual [concepts of cert manager](https://cert-manager.io/docs/concepts/).

* Further, to understand how Let’s Encrypt issues certificates to domains: 
[How it works](https://letsencrypt.org/how-it-works/)

* Different challenges Let’s Encrypt uses: [Challenge types](https://letsencrypt.org/docs/challenge-types/).
