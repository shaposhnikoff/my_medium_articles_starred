
# Securing Your Istio Ingress Gateway with HTTPS

In the last post, Building a Microservices Platform with Confluent Cloud, MongoDB Atlas, Istio, and Google Kubernetes Engine, we built and deployed a microservice-based, cloud-native API to Google Kubernetes Engine (GKE), with Istio 1.0, on Google Cloud Platform (GCP). For brevity, we neglected a few key API features, required in Production, including HTTPS, OAuth for authentication, request quotas, request throttling, and the integration of a full lifecycle API management tool, like Google Apigee.

In this brief post, we will revisit the previous post’s project. We will disable HTTP, and secure the GKE cluster with HTTPS, using simple TLS, as opposed to [mutual TLS authentication](https://en.wikipedia.org/wiki/Mutual_authentication) (mTLS). This post assumes you have created the GKE cluster and deployed the Storefront API and its associated resources, as explained in the previous post.

## What is HTTPS?

According to [Wikipedia](https://en.wikipedia.org/wiki/HTTPS), Hypertext Transfer Protocol Secure (HTTPS) is an extension of the [Hypertext Transfer Protocol](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) (HTTP) for securing communications over a computer network. In HTTPS, the [communication protocol](https://en.wikipedia.org/wiki/Communication_protocol) is encrypted using [Transport Layer Security](https://en.wikipedia.org/wiki/Transport_Layer_Security) (TLS), or, formerly, its predecessor, Secure Sockets Layer (SSL). The protocol is therefore also often referred to as HTTP over TLS, or HTTP over SSL.

Further, according to Wikipedia, the principal motivation for HTTPS is [authentication](https://en.wikipedia.org/wiki/Authentication) of the accessed website and protection of the privacy and integrity of the exchanged data while in transit. It protects against [man-in-the-middle attacks](https://en.wikipedia.org/wiki/Man-in-the-middle_attack). The bidirectional [encryption](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation) of communications between a client and server provides a reasonable assurance that one is communicating without interference by attackers with the website that one intended to communicate with, as opposed to an impostor.

## Public Key Infrastructure

According to Comodo, both the TLS and SSL protocols use what is known as an asymmetric [Public Key Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure) (PKI) system. An asymmetric system uses two keys to encrypt communications, a public key and a private key. Anything encrypted with the public key can only be decrypted by the private key and vice-versa.

Again, according to [Wikipedia](https://en.wikipedia.org/wiki/Public_key_infrastructure), a PKI is an arrangement that binds [public keys](https://en.wikipedia.org/wiki/Public_key) with respective identities of entities, like people and organizations. The binding is established through a process of registration and issuance of certificates at and by a [certificate authority](https://en.wikipedia.org/wiki/Certificate_authority) (CA).

## SSL/TLS Digital Certificate

Again, according to [Comodo](https://www.instantssl.com/ssl-certificate-products/https.html), when you request an HTTPS connection to a webpage, the website will initially send its SSL certificate to your browser. This certificate contains the public key needed to begin the secure session. Based on this initial exchange, your browser and the website then initiate the SSL handshake (actually, [TLS handshake](https://en.wikipedia.org/wiki/Transport_Layer_Security#TLS_handshake)). The handshake involves the generation of shared secrets to establish a uniquely secure connection between yourself and the website. When a trusted SSL digital certificate is used during an HTTPS connection, users will see the padlock icon in the browser’s address bar.

## Registered Domain

In order to secure an SSL Digital Certificate, required to enable HTTPS with the GKE cluster, we must first have a registered domain name. For the last post, and this post, I am using my own personal domain, storefront-demo.com. The domain’s primary A record (‘@’) and all sub-domain A records, such as api.dev, are all resolve to the external IP address on the front-end of the GCP load balancer.

For DNS hosting, I happen to be using [Azure DNS](https://azure.microsoft.com/en-us/services/dns/) to host the domain, storefront-demo.com. All DNS hosting services basically work the same way, whether you chose Azure, AWS, GCP, or another third party provider.

## Let’s Encrypt

If you have used [Let’s Encrypt](https://letsencrypt.org/) before, then you know how easy it is to get free SSL/TLS Certificates. Let’s Encrypt is the first free, automated, and open certificate authority (CA) brought to you by the non-profit Internet Security Research Group (ISRG).

According to Let’s Encrypt, to enable HTTPS on your website, you need to get a certificate from a Certificate Authority (CA); Let’s Encrypt is a CA. In order to get a certificate for your website’s domain from Let’s Encrypt, you have to demonstrate control over the domain. With Let’s Encrypt, you do this using software that uses the [ACME protocol](https://ietf-wg-acme.github.io/acme/), which typically runs on your web host. If you have generated certificates with Let’s Encrypt, you also know the domain validation by installing the [Certbot](https://certbot.eff.org/) ACME client can be a bit daunting, depending on your level of access and technical expertise.

## SSL For Free

This is where [SSL For Free](https://www.sslforfree.com/) comes in. SSL For Free acts as a proxy of sorts to Let’s Encrypt. SSL For Free generates certificates using their ACME server by using domain validation. Private Keys are generated in your browser and never transmitted.

![](https://cdn-images-1.medium.com/max/2000/0*PF6JqvoAXkYwLPe2)

SSL For Free offers three domain validation methods:

1. Automatic FTP Verification: Enter FTP information to automatically verify the domain;

1. Manual Verification: Upload verification files manually to your domain to verify ownership;

1. Manual Verification (DNS): Add TXT records to your DNS server;

Using the third domain validation method, manual verification using DNS, is extremely easy, if you have access to your domain’s DNS recordset.

![](https://cdn-images-1.medium.com/max/2000/0*EOg5HIiMqVSp9NjK)

SSL For Free provides [TXT records](https://en.wikipedia.org/wiki/TXT_record) for each domain you are adding to the certificate. Below, I am adding a single domain to the certificate.

![](https://cdn-images-1.medium.com/max/2000/0*w40nDryYqNY3vnRW)

Add the TXT records to your domain’s recordset. Shown below is an example of a single TXT record that has been to my recordset using the Azure DNS service.

SSL For Free then uses the TXT record to validate your domain is actually yours.

![](https://cdn-images-1.medium.com/max/2000/0*2lz_fdcFgNS7LG71)

With the TXT record in place and validation successful, you can download a ZIPped package containing the certificate, private key, and CA bundle. The CA bundle containing the end-entity root and intermediate certificates.

![](https://cdn-images-1.medium.com/max/2000/0*ByR5L6_PdB9FvCZG)

## Decoding PEM Encoded SSL Certificate

Using a tool like [SSL Shopper’s Certificate Decoder](https://www.sslshopper.com/certificate-decoder.html), we can decode our [Privacy-Enhanced Mail](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) (PEM) encoded SSL certificates and view all of the certificate’s information. Decoding the information contained in my certificate.crt, I see the following.

    Certificate Information: Common Name: api.dev.storefront-demo.com Subject Alternative Names: api.dev.storefront-demo.com Valid From: December 26, 2018 Valid To: March 26, 2019 Issuer: Let's Encrypt Authority X3, Let's Encrypt Serial Number: 03a5ec86bf79de65fb679ee7741ba07df1e4

Decoding the information contained in my ca_bundle.crt, I see the following.

    Certificate Information: Common Name: Let's Encrypt Authority X3 Organization: Let's Encrypt Country: US Valid From: March 17, 2016 Valid To: March 17, 2021 Issuer: DST Root CA X3, Digital Signature Trust Co. Serial Number: 0a0141420000015385736a0b85eca708

The Let’s Encrypt intermediate certificate is also [cross-signed](https://letsencrypt.org/certificates/) by another certificate authority, IdenTrust, whose root is already trusted in all major browsers. IdenTrust cross-signs the Let’s Encrypt intermediate certificate using their DST Root CA X3. Thus, the Issuer, shown above.

## Configure Istio Ingress Gateway

Unzip the sslforfree.zip package and place the individual files in a location you have access to from the command line.

    unzip -l ~/Downloads/sslforfree.zip Archive: /Users/garystafford/Downloads/sslforfree.zip Length Date Time Name --------- ---------- ----- ---- ** 1943 12-26-2018 18:35 certificate.crt 1707 12-26-2018 18:35 private.key 1646 12-26-2018 18:35 ca_bundle.crt **--------- ------- 5296 3 files

Following the process outlined in the Istio documentation, [Securing Gateways with HTTPS](https://istio.io/docs/tasks/traffic-management/secure-ingress/), run the following command. This will place the istio-ingressgateway-certs Secret in the istio-system namespace, on the GKE cluster.

    kubectl create -n istio-system secret tls istio-ingressgateway-certs \ --key path_to_files/sslforfree/private.key \ --cert path_to_files/sslforfree/certificate.crt

Modify the existing Istio Gateway from the previous project, [istio-gateway.yaml](https://github.com/garystafford/storefront-kafka-docker/blob/master/gke/resources/other/istio-gateway.yaml). Remove the HTTP port configuration item and replace with the HTTPS protocol item ([*gist](https://gist.github.com/garystafford/0413ed7dbdc2c065d9c709883546befb)*). Redeploy the Istio Gateway to the GKE cluster.

<iframe src="https://medium.com/media/ac766682d78cce508ad33f21b28b2087" frameborder=0></iframe>

By deploying the new istio-ingressgateway-certs Secret and redeploying the Gateway, the certificate and private key were deployed to the /etc/istio/ingressgateway-certs/ directory of the istio-proxy container, running on the istio-ingressgateway Pod. To confirm both the certificate and private key were deployed correctly, run the following command.

    kubectl exec -it -n istio-system \ $(kubectl -n istio-system get pods \ -l istio=ingressgateway \ -o jsonpath='{.items[0].metadata.name}') \ -- ls -l /etc/istio/ingressgateway-certs/ **lrwxrwxrwx 1 root root 14 Jan 2 17:53 tls.crt -> ..data/tls.crt** **lrwxrwxrwx 1 root root 14 Jan 2 17:53 tls.key -> ..data/tls.key**

That’s it. We should now have simple TLS enabled on the Istio Gateway, providing bidirectional encryption of communications between a client (Storefront API consumer) and server (Storefront API running on the GKE cluster). Users accessing the API will now have to use HTTPS.

## Confirm HTTPS is Working

After completing the deployment, as outlined in the previous post, test the Storefront API by using HTTP, first. Since we removed the HTTP port item configuration in the Istio Gateway, the HTTP request should fail with a connection refused error. Insecure traffic is no longer allowed by the Storefront API.

![](https://cdn-images-1.medium.com/max/2000/0*RHf9lnQ3id646vJ9)

Now try switching from HTTP to HTTPS. The page should be displayed and the black lock icon should appear in the browser’s address bar. Clicking on the lock icon, we will see the SSL certificate, used by the GKE cluster is valid.

![](https://cdn-images-1.medium.com/max/2000/0*62p0kw5ZHjZSgail)

By clicking on the valid certificate indicator, we may observe more details about the SSL certificate, used to secure the Storefront API. Observe the certificate is issued by Let’s Encrypt Authority X3. It is valid for 90 days from its time of issuance. Let’s Encrypt only issues certificates with a [90-day lifetime](https://letsencrypt.org/2015/11/09/why-90-days.html). Observe the public key uses SHA-256 with [RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) (Rivest–Shamir–Adleman) encryption.

![](https://cdn-images-1.medium.com/max/2000/0*vQA-GWjJVUHW7QA9)

In Chrome, we can also use the Developer Tools Security tab to inspect the certificate. The certificate is recognized as valid and trusted. Also important, note the connection to this Storefront API is encrypted and authenticated using TLS 1.2 (a strong protocol), ECDHE_RSA with X25519 (a strong key exchange), and AES_128_GCM (a strong cipher). According to How’s My SSL?, TLS 1.2 is the latest version of TLS. The TLS 1.2 protocol provides access to advanced cipher suites that support elliptical curve cryptography and AEAD block cipher modes. TLS 1.2 is an improvement on previous TLS 1.1, 1.0, and SSLv3 or earlier.

![](https://cdn-images-1.medium.com/max/2000/0*ed1mq4HIUBzJaw1D)

Lastly, the best way to really understand what is happening with HTTPS, the Storefront API, and Istio, is verbosely curl an API endpoint.

    curl -Iv [https://api.dev.storefront-demo.com/accounts/](https://api.dev.storefront-demo.com/accounts/)

Using the above curl command, we can see exactly how the client successfully verifies the server, negotiates a secure [HTTP/2](https://en.wikipedia.org/wiki/HTTP/2) connection (HTTP/2 over TLS 1.2), and makes a request ([*gist](https://gist.github.com/garystafford/75dcc1c5200fd7e1c82f2b4f3e357ed8)*).

* Line 3: DNS resolution of the URL to the external IP address of the GCP load-balancer

* Line 3: HTTPS traffic is routed to TCP port 443

* Lines 4–5: Application-Layer Protocol Negotiation (ALPN) starts to occur with the server

* Lines 7–9: Certificate to verify located

* Lines 10–20: TLS handshake is performed and is successful using TLS 1.2 protocol

* Line 20: CHACHA is the stream cipher and POLY1305 is the authenticator in the Transport Layer Security (TLS) 1.2 protocol [ChaCha20-Poly1305](https://tools.ietf.org/html/rfc7905) Cipher Suite

* Lines 22–27: SSL certificate details

* Line 28: Certificate verified

* Lines 29–38: Establishing HTTP/2 connection with the server

* Lines 33–36: Request headers

* Lines 39–46: Response headers containing the expected 204 HTTP return code

<iframe src="https://medium.com/media/59158c808e67e31a68705195ea48e286" frameborder=0></iframe>

## Mutual TLS

Istio also supports [mutual authentication](https://en.wikipedia.org/wiki/Mutual_authentication) using the TLS protocol, known as mutual TLS authentication (mTLS), between external clients and the gateway, as outlined in the Istio 1.0 [documentation](https://istio.io/docs/tasks/traffic-management/secure-ingress/#configure-a-mutual-tls-ingress-gateway). According to Wikipedia, mutual authentication or two-way authentication refers to two parties authenticating each other at the same time. Mutual authentication a default mode of authentication in some protocols ([IKE](https://en.wikipedia.org/wiki/Internet_Key_Exchange), SSH), but optional in TLS.

Again, according to Wikipedia, by default, TLS only proves the identity of the server to the client using [X.509 certificate](https://en.wikipedia.org/wiki/X.509_certificate)s. The authentication of the client to the server is left to the application layer. TLS also offers client-to-server authentication using client-side X.509 authentication. As it requires provisioning of the certificates to the clients and involves less user-friendly experience, it is rarely used in end-user applications. Mutual TLS is much more widespread in B2B applications, where a limited number of programmatic clients are connecting to specific web services. The operational burden is limited and security requirements are usually much higher as compared to consumer environments.

This form of mutual authentication would be beneficial if we had external applications or other services outside our GKE cluster, consuming our API. Using mTLS, we could further enhance the security of those types of interactions.

*Originally published at [programmaticponderings.com](https://programmaticponderings.com/2019/01/03/securing-your-istio-gateway-with-https/) on January 4, 2019.*

*All opinions expressed in this post are my own and not necessarily the views of my current or past employers, their clients.*
