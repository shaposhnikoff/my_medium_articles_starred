
# Configuring Certificate-Based Mutual Authentication with Kubernetes Ingress-Nginx

I was recently adding e2e test cases to Kubernetes Ingress-Nginx when I realized that there aren’t too many resources out there to show how to properly configure and test Mutual(Client Certificate) Authentication.

I’m writing this post to show you how easy it is to get it done!

![“two man doing shake hands” by [rawpixel](https://unsplash.com/@rawpixel?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/12000/0*meWedpQr04C_cMB0)*“two man doing shake hands” by [rawpixel](https://unsplash.com/@rawpixel?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

## What is Mutual Authentication?

Mutual authentication is also known as 2-way authentication. It is a process in which both the client and server verify each others identity via a Certificate Authority. [CodeProject.com](https://www.codeproject.com/Articles/326574/An-Introduction-to-Mutual-SSL-Authentication) has a great definition for Mutual Authentication:
> # Mutual SSL authentication or certificate based mutual authentication refers to two parties authenticating each other through verifying the provided digital certificate so that both parties are assured of the others’ identity.

![Mutual Authentication flow from [codeproject.com](https://www.codeproject.com/Articles/326574/An-Introduction-to-Mutual-SSL-Authentication)](https://cdn-images-1.medium.com/max/2000/1*x8eHQ4KJ4Eb9WuVGWzw7xw.png)*Mutual Authentication flow from [codeproject.com](https://www.codeproject.com/Articles/326574/An-Introduction-to-Mutual-SSL-Authentication)*

In this tutorial the client will be us via *curl*, and the server is of course *NGINX*.

## Setting Up the Ingress Controller

There are many ways of configuring Ingress-Nginx on your Kubernetes cluster. If you already have an Ingress-Nginx controller setup, then you can skip this step. However, note that this guide was written using **Minikube version [0.30](https://github.com/kubernetes/minikube/releases/tag/v0.30.0)*** *with **Ingress-Nginx version [0.19](https://github.com/kubernetes/ingress-nginx/releases/tag/nginx-0.19.0)**.

* If you just wanna try this out for yourself in *Minikube* see my previous blog, [**“Getting Started with Kubernetes Ingress-Nginx on Minikube”](https://medium.com/@awkwardferny/getting-started-with-kubernetes-ingress-nginx-on-minikube-d75e58f52b6c), **and comeback here once you’ve deployed the ingress controller.

* For other ways of deploying the ingress controller, you can checkout the Ingress-Nginx deployment [**documentation](https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/index.md)**.

## Setting Up Mutual Authentication

In order to setup Mutual Authentication you will need to perform a couple of steps.

### Creating the Certificates

For this example we will be creating self-signed certificates(just for test purposes, not to ever be done in production). As a simple introduction, here are a couple of terms it would be useful to know:

* **CommonName(CN)**: Identifies the hostname or owner associated with the certificate.

* **Certificate Authority(CA)**: A trusted 3rd party that issues Certificates. Usually you would obtain this from a trusted source, but for this example we will just create one. The CN is usually the name of the issuer.

* **Server Certificate**: A Certificate used to identify the server. The CN here is the hostname of the server. The Server Certificate is valid only if it is installed on a server where the hostname matches the CN.

* **Client Certificate**: A Certificate used to identify a client/user. The CN here is usually the name of the client/user.

    # Generate the CA Key and Certificate
    $ openssl req -x509 -sha256 -newkey rsa:4096 -keyout ca.key -out ca.crt -days 356 -nodes -subj '/CN=Fern Cert Authority'

    # Generate the Server Key, and Certificate and Sign with the CA Certificate
    $ openssl req -new -newkey rsa:4096 -keyout server.key -out server.csr -nodes -subj '/CN=meow.com'
    $ openssl x509 -req -sha256 -days 365 -in server.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out server.crt

    # Generate the Client Key, and Certificate and Sign with the CA Certificate
    $ openssl req -new -newkey rsa:4096 -keyout client.key -out client.csr -nodes -subj '/CN=Fern'
    $ openssl x509 -req -sha256 -days 365 -in client.csr -CA ca.crt -CAkey ca.key -set_serial 02 -out client.crt

### Creating the Kubernetes Secrets

We must store the certificates generated above in a Kubernetes Secret in order to use them in our Ingress-NGINX controller.

In this example, for simplicity, our Secret will contain both our Server Certificate and our CA Certificate. The Ingress Controller will understand which certs to use and where to use them. They can also be Split into Separate Secrets. See [here](https://github.com/kubernetes/ingress-nginx/tree/master/docs/examples/auth/client-certs) for detailed info.

    $ kubectl create secret generic my-certs --from-file=tls.crt=server.crt --from-file=tls.key=server.key --from-file=ca.crt=ca.crt

    $ kubectl get secret my-certs
    NAME       TYPE     DATA   AGE
    my-certs   Opaque   3      1m

### Deploying an Application with Mutual Authentication

1. Deploy our pods(containers), using deployments. A deployment, pretty much just manages the state of our pods, It’s out of the scope of this tutorial, but you can learn more [here](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

    $ echo "
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: meow
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: meow
      template:
        metadata:
          labels:
            app: meow
        spec:
          containers:
          - name: meow
            image: gcr.io/kubernetes-e2e-test-images/echoserver:2.1
            ports:
            - containerPort: 8080
    " | kubectl apply -f -

    # wait a min for the deployment to be created
    $ kubectl get deploy
    NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE
    meow       2         2         2            2

    # you should have 2 pods running
    $ kubectl get pods
    NAME                       READY     STATUS
    meow-5557bc7c54-cw2ck     1/1       Running
    meow-5557bc7c54-kfzm5     1/1       Running

**gcr.io/kubernetes-e2e-test-images/echoserver:2.1 **just responds with information about the request.

2. Expose our pods using Services. More info about exposure [here](https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/)

    $ echo "
    apiVersion: v1
    kind: Service
    metadata:
      name: meow-svc
    spec:
      ports:
      - port: 80
        targetPort: 8080
        protocol: TCP
        name: http
      selector:
        app: meow
    " | kubectl apply -f -

    # wait a min for the service to be created
    $ kubectl get svc
    NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)
    meow-svc       ClusterIP   10.107.78.24    <none>        80/TCP

3. Setup the Ingress Rules, and add **meow.com **to** /etc/hosts**

    $ echo "
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      annotations:
        nginx.ingress.kubernetes.io/auth-tls-verify-client: \"on\"
        nginx.ingress.kubernetes.io/auth-tls-secret: \"default/my-certs\"
      name: meow-ingress
      namespace: default
    spec:
      rules:
      - host: meow.com
        http:
          paths:
          - backend:
              serviceName: meow-svc
              servicePort: 80
            path: /
      tls:
      - hosts:
        - meow.com
        secretName: my-certs
    " | kubectl apply -f -

    $ kubectl get ing cat-nginx
    NAME           HOSTS     ADDRESS     PORTS     AGE
    meow-ingress   meow.com  10.0.2.15   80, 443   1m

    # Add meow.com to /etc/hosts
    $ sudo -- sh -c "echo $(minikube ip)  meow.com >> /etc/hosts"

This allows us to access the service *meow-svc* via *https://meow.com/*.

* TLS is enabled and it is using the *tls.key* and *tls.crt* provided in the *my-certs* secret.

* The *nginx.ingress.kubernetes.io/auth-tls-secret** ***annotation uses *ca.crt* from the *my-certs* secret.

### Testing it all out

Sending a request without the Client Certificate and Key, should give a 400 Error as follows:

    $ curl https://meow.com/ -k
    ...
    <center><h1>400 Bad Request</h1></center>
    <center>No required SSL certificate was sent</center>
    ....

Sending a request with the Client Certificate and Key, should redirect you to the *meow-svc:*

    $ curl https://meow.com/ --cert client.crt --key client.key -k
    ...
    ssl-client-issuer-dn=CN=Fern Cert Authority
    ssl-client-subject-dn=CN=Fern
    ssl-client-verify=SUCCESS
    user-agent=curl/7.54.0
    ...

If the above did not work for you, skip down to the troubleshooting section.

### Other possible settings

* *nginx.ingress.kubernetes.io/auth-tls-verify-client* enables verification of client certificates.

* *nginx.ingress.kubernetes.io/auth-tls-verify-depth* sets the validation depth between the provided client certificate and the certification authority chain.

* *nginx.ingress.kubernetes.io/auth-tls-error-page* sets the URL/Page that user should be redirected in case of a Certificate Authentication Error.

* *nginx.ingress.kubernetes.io/auth-tls-pass-certificate-to-upstream *indicates if the received certificates should be passed or not to the upstream server.

For more information about these setting can be seen in the [Ingress-NGINX documentation](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#client-certificate-authentication) as well as the [NGINX documentation](http://nginx.org/en/docs/http/ngx_http_ssl_module.html) for the directives set by Ingress-NGINX.

### Troubleshooting

First thing we can do is check if the NGINX configuration was generated properly with the CA:

    $ kubectl get pods -n kube-system | grep nginx-ingress-controller
    nginx-ingress-controller-5984b97644-qbwtv   1/1       Running

    $ kubectl exec -it -n kube-system nginx-ingress-controller-5984b97644-qbwtv cat /etc/nginx/nginx.conf | grep ssl_client_certificate -A 1

    ssl_client_certificate                  /etc/ingress-controller/ssl/default-my-certs.pem;
    ssl_verify_client                       on;

Another thing we can look at is logs, and see if theres anything going on with the CA:

    $ kubectl get pods -n kube-system | grep nginx-ingress-controller
    nginx-ingress-controller-5984b97644-qbwtv   1/1       Running

    $ kubectl logs -n kube-system nginx-ingress-controller-5984b97644-qbwtv
    ....
    I0831 21:45:34.090212       5 nginx.go:271] Starting NGINX process
    ....

If it’s still not working, I’d suggest creating an [issue](https://github.com/kubernetes/ingress-nginx/issues) or asking questions via [slack](http://kubernetes.slack.com/messages/ingress-nginx).

Thank you for reading and I hope this tutorial helped!!
