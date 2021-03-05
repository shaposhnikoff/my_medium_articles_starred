
# Kubernetes Services over HTTPS With Istio’s Secure Gateways

Expose your microservices over TLS to the external world

![Photo by [Benjamin Child](https://unsplash.com/@bchild311?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](/collections/385548/work-and-collaboration?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/8754/1*mlMaK0mb-DJWTg7vpcO47w.jpeg)*Photo by [Benjamin Child](https://unsplash.com/@bchild311?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](/collections/385548/work-and-collaboration?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

[Istio](https://istio.io) provides you with many features that help you connect, secure, control and observe your microservices. It gives Kubernetes much control on top of what it’s generally capable of. Istio gateways are a powerful resource that allow you to define entry points into your service mesh from the external world.

In the previous article, we exposed the BookInfo application to the HTTP protocol. However, Istio provides secure gateways to host your microservices on HTTPS. Istio provides both simple and mutual TLS authentication, and it makes your life simpler, as you no longer need to manage certificates outside of the Kubernetes cluster.

This article is a follow up to “[How to Manage Traffic Using Istio on Kubernetes](https://medium.com/better-programming/how-to-manage-traffic-using-istio-on-kubernetes-cd4b96e00b57).” Today, let’s discuss how to expose microservices to the external world over HTTPS using Istio secure gateways.

## Prerequisites

You need a running Kubernetes cluster for the hands-on exercise.

Install Istio within your Kubernetes cluster by following the “[Getting Started With Istio on Kubernetes](https://medium.com/better-programming/getting-started-with-istio-on-kubernetes-e582800121ea)” guide.

Determine the INGRESS_HOST and SECURE_INGRESS_PORT by running the below code.

If you’re using a load balancer, run the following.

    $ export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    $ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')

If you aren’t using a load balancer, you can use a Node IP and Node port combination as well. Run the below to use that.

    export INGRESS_HOST=<worker-node-address>
    export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')

## Generate TLS Certificates

For the hands-on demo, we assume the example.com domain.

Let’s use OpenSSL to generate a self-signed root certificate for example.com.

    $ openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -subj '/O=example Inc./CN=example.com' -keyout example.com.key -out example.com.crt

Create a certificate and private key for bookinfo.example.com using the example.com root cert. Let’s first generate a private key and a CSR, and then use the example.com root cert to sign the CSR to create a public bookinfo.example.com certificate.

    $ openssl req -out bookinfo.example.com.csr -newkey rsa:2048 -nodes -keyout bookinfo.example.com.key -subj "/CN=bookinfo.example.com/O=bookinfo organization"
    $ openssl x509 -req -days 365 -CA example.com.crt -CAkey example.com.key -set_serial 0 -in bookinfo.example.com.csr -out bookinfo.example.com.crt

## Configure a Simple TLS for the BookInfo Application

Create a secret with the generated key and certificate on the istio-system namespace. Please note that the secret name shouldn’t begin with istio or prometheus, as they’re keywords reserved for system secrets.

    $ kubectl create -n istio-system secret tls bookinfo-credential --key=bookinfo.example.com.key --cert=bookinfo.example.com.crt
    secret/bookinfo-credential created

Create a BookInfo gateway on the default Istio ingress using the credential. The below YAML defines a gateway called bookinfogateway on the default Istio ingressgateway, listening on port 443 on a simple TLS protocol, and uses the bookinfo-credential for the host bookinfo.example.com.

<iframe src="https://medium.com/media/f24f1dee1adf4099fb04597c3bcb6fe7" frameborder=0></iframe>

Create a virtual service to connect to the product page.

<iframe src="https://medium.com/media/070e96d59096dbe96691bba6deb4bcdc" frameborder=0></iframe>

Apply the destination rules.

    $ kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
    destinationrule.networking.istio.io/productpage created
    destinationrule.networking.istio.io/reviews created
    destinationrule.networking.istio.io/ratings created
    destinationrule.networking.istio.io/details created

Make the following entry to your host file. It’s needed if you want to access the product page from the browser.

    <INGRESS_HOST> bookinfo.example.com

Access the BookInfo application from your browser.

Go to [https://bookinfo.example.com/productpage](https://bookinfo.example.com/productpage), and you should see the following.

![BookInfo product page](https://cdn-images-1.medium.com/max/3840/1*72AdXqFD9ZWcLogBJyOuZg.png)*BookInfo product page*

As the certificate is self-signed, we get an insecure warning. If you use a certificate signed by a public CA, the notice should disappear.

Let’s access the page through curl to see if we get a validation error if we use the example.com CA certificate for validation.

<iframe src="https://medium.com/media/70d75165e1c30e5484344e8c0aaaeb36" frameborder=0></iframe>

And we get an HTTP 200 response. That proves that simple TLS is working fine.

## Certificate Rotation

Rotating certificates for the ingress gateway is just a few commands away.

Generate a new set of certificates from the root cert.

    $ mkdir new_certificates
    $ openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -subj '/O=example Inc./CN=example.com' -keyout new_certificates/example.com.key -out new_certificates/example.com.crt
    $ openssl req -out new_certificates/bookinfo.example.com.csr -newkey rsa:2048 -nodes -keyout new_certificates/bookinfo.example.com.key -subj "/CN=bookinfo.example.com/O=bookinfo organization"
    $ openssl x509 -req -days 365 -CA new_certificates/example.com.crt -CAkey new_certificates/example.com.key -set_serial 0 -in new_certificates/bookinfo.example.com.csr -out new_certificates/bookinfo.example.com.crt

Now delete the old secret, and create a new one with the new credentials.

    $ kubectl -n istio-system delete secret bookinfo-credential
    secret "bookinfo-credential" deleted
    $ kubectl create -n istio-system secret tls bookinfo-credential \
    --key=new_certificates/bookinfo.example.com.key \
    --cert=new_certificates/bookinfo.example.com.crt
    secret/bookinfo-credential created

Let’s now access the BookInfo application using the new cert.

<iframe src="https://medium.com/media/caf21c05789029b23f4eed5e21fa08af" frameborder=0></iframe>

What would happen if we used the old certificates to connect with the service? Let’s find out.

<iframe src="https://medium.com/media/f4d5782069d1ada54935b475058a876a" frameborder=0></iframe>

And it fails! That shows that certificate rotation is working correctly.

## Configuring Mutual TLS

Mutual TLS might be necessary if you need to establish trust between client and server and vice versa. That might be necessary if you’re hosting an API and want only trusted clients to access it.

For the gateway to use Mutual TLS, we need to redefine the bookinfo-credential to set the tls.key, tls.cert, and the cacert property to hold the CA certificate. That is required, as now the server needs to trust the client, and it needs to validate the client certificate using the configured CA certificate.

    $ kubectl -n istio-system delete secret bookinfo-credential
    secret "bookinfo-credential" deleted
    $ kubectl create -n istio-system secret generic bookinfo-credential --from-file=tls.key=bookinfo.example.com.key \
    --from-file=tls.crt=bookinfo.example.com.crt --from-file=ca.crt=example.com.crt
    secret/bookinfo-credential created

Update the ingress gateway to use MUTUAL TLS authentication.

<iframe src="https://medium.com/media/b88a8628c888e2526fbfa60fd2985747" frameborder=0></iframe>

Now let’s access the application using the old method.

<iframe src="https://medium.com/media/7ba2c780982d9fa8f13f431be45130dc" frameborder=0></iframe>

And we get an error! That means that it needs a client certificate to authenticate with the BookInfo application. Let’s generate the client key and certificate using OpenSSL.

    $ openssl req -out client.example.com.csr -newkey rsa:2048 -nodes -keyout client.example.com.key -subj "/CN=client.example.com/O=client organization"
    $ openssl x509 -req -days 365 -CA example.com.crt -CAkey example.com.key -set_serial 1 -in client.example.com.csr -out client.example.com.crt

Let’s use the client certificate to authenticate with the BookInfo application.

<iframe src="https://medium.com/media/640d8ea9a745ca2a3bd3676bd8f09f72" frameborder=0></iframe>

And we get an HTTP 200 response.

Let’s try to access the BookInfo application from the browser.

![](https://cdn-images-1.medium.com/max/3836/1*nKE5sK-SOqTNRgwqlst3pw.png)

And now we see it doesn’t allow us to connect because we haven’t supplied a valid client cert when we open the page from the browser.

Congratulations! You’ve successfully configured a mutual TLS in Istio!

## Conclusion

Thanks for reading. I hope you enjoyed the article.

In the next part, I’ll discuss “[Traffic Mirroring in Kubernetes Using Istio](https://medium.com/better-programming/traffic-mirroring-in-kubernetes-using-istio-dad0976b4e1)” with a hands-on demonstration, so see you there!
