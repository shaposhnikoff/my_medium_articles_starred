
# How to Authorize Non-Kubernetes Clients With Istio on Your K8s Cluster

Use JSON web tokens to authorize clients to interact with your Kubernetes microservices using Istio

![Photo by [Mujeres De México](https://unsplash.com/@mujeres_de_mexico?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/12032/1*bsKcUBHiLMCAQPmPILgHCA.jpeg)*Photo by [Mujeres De México](https://unsplash.com/@mujeres_de_mexico?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

[Istio](https://istio.io) is one of the most desired Kubernetes aware-service mesh technologies that grants you immense power if you host microservices on Kubernetes.

In my last article, “[Enable Access Control Between Your Kubernetes Workloads Using Istio](https://medium.com/better-programming/enable-access-control-between-your-kubernetes-workloads-using-istio-cf72a9f9bd5e),” we discussed how to use Istio to manage access between Kubernetes microservices.

That works well for internal communication. However, most use cases require you authorise non-Kubernetes clients to connect with your Kubernetes workloads — for example, if you expose APIs for third parties to integrate with.

Istio furnishes this capability through its Layer 7 Envoy proxies and utilises [JSON Web Tokens](https://jwt.io) (JWT) for authorisation. In this article, we’ll explore how we can leverage Istio to facilitate this with a hands-on demonstration.

## What Are JSON Web Tokens?

JSON Web Tokens (JWT) are tokens based on [RFC 7519](https://tools.ietf.org/html/rfc7519) that represent claims between two parties. You can employ them to hold identity information and other metadata.

A web token is produced by digitally signing a JSON string with a JSON Web Key (JWK) by a trusted identity provider. The signing process constructs a MAC, which becomes the JWT signature.

The server needs to confirm whether the JWK has signed the JWT during the authorisation process.

![JSON web token](https://cdn-images-1.medium.com/max/2000/1*JDIcHct1UYbhXX_ZrtIYcg.png)*JSON web token*

Below is an example of a JWT:

    **eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9**.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkdhdXJhdiBBZ2Fyd2FsIiwiaWF0IjoxNTE2MjM5MDIyLCJleHAiOjE1ODk0MDc5Mjh9.***KJzt_O-Xwtd1DF_Ie0yi5lVpEiH4spoyZBr3rATTHqw***

The bold part is the header that contains the payload type and key algorithm.

    {
      "alg": "HS256",
      "typ": "JWT"
    }

The non-formatted string is the payload. This payload includes claims, the issued time (iat), and the expiry time (exp).

    {
      "sub": "1234567890",
      "name": "Gaurav Agarwal",
      "iat": 1516239022,
      "exp": 1589407928
    }

The part in italic is the signature generated after signing the JWT with a JWK. If someone tampers with the payload, the JWT is deemed invalid, as a different MAC would be generated in the verification process.

A requestor logs into an identity provider with their credentials, the identity provider website issues a JWT token, and the user employs the JWT token for further interaction with the microservices.

JWT is usually sent as a Bearer token in the HTTP request Authorization header.

![](https://cdn-images-1.medium.com/max/2000/1*9xQRvE49mAzSBQlIlJn1vA.png)

## Prerequisites

Ensure you’re running a Kubernetes cluster and understand how Istio works. A great starting point for an introduction to Istio is “[How to Manage Microservices on Kubernetes With Istio](https://medium.com/better-programming/how-to-manage-microservices-on-kubernetes-with-istio-c25e97a60a59).”

Install Istio on the Kubernetes cluster by following “[Getting Started With Istio on Kubernetes](https://medium.com/better-programming/getting-started-with-istio-on-kubernetes-e582800121ea)” guide. You don’t need to deploy the Book Info application for the demonstration.

## Install the Sample Applications

Create a namespace, foo, and label the namespace so that Istio can inject sidecars automatically. Deploy the httpbin and sleep microservices, as below:

    $ kubectl create ns foo
    $ kubectl label namespace foo istio-injection=enabled
    $ kubectl apply -f samples/httpbin/httpbin.yaml -n foo
    $ kubectl apply -f samples/sleep/sleep.yaml -n foo

Now let’s test if we can call the httpbin microservice from the sleep microservice.

    $ kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl [http://httpbin.foo:8000/ip](http://httpbin.foo:8000/ip) -s -o /dev/null -w "%{http_code}\n"

    200

## Apply Request Authentication on the httpbin Microservice

Create an authentication policy to accept a JWT issued by testing@secure.istio.io.

<iframe src="https://medium.com/media/839084c12501cde6a66dac7365b00c50" frameborder=0></iframe>

The YAML selects the httpbin microservice and applies a JWT rule to examine if the issuer is testing@secure.istio.io. Additionally, it also has a jwksUri that links to the JWK to validate the JWT.

For the demonstration, the JWK is publicly available. However, you should secure the JWK using a credential-management system and protect it as a password. If your JWK is compromised, then anyone can access your microservices by generating new JWTs.

It’s an excellent exercise to frequently rotate JWKs and sync them with the identity provider.

## Testing the Authentication Policy

Now let’s trigger a request with an invalid token to verify if Istio denies it.

    $ kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl "[http://httpbin.foo:8000/headers](http://httpbin.foo:8000/headers)" -s -o /dev/null -H "Authorization: Bearer invalidToken" -w "%{http_code}\n"

    401

And we get 401 Unauthorised. Let’s try without a JWT token.

    $ kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl "[http://httpbin.foo:8000/headers](http://httpbin.foo:8000/headers)" -s -o /dev/null -w "%{http_code}\n"

    200

What happened? Well, we contemplated that as we haven’t applied an authorisation policy yet, Istio permits all requests without a JWT token for compatibility with legacy systems.

The authentication policy warrants that if your request contains a JWT, then it should be valid. If it doesn’t hold a JWT, the request is still allowed, and the authorisation policy should enforce additional rules.

## Create an Authorisation Policy

Now let’s create an authorisation policy that necessitates a valid JWT.

<iframe src="https://medium.com/media/519fbe4f72ffcefb2f2ea42b57dc98f1" frameborder=0></iframe>

The above YAML authorises all requests to the httpbin microservice that has a request principal testing@secure.istio.io/testing@secure.istio.io.

There are two segments of the request principal — issuer and subject. A valid JWT must include an issuer and subject claim equal to testing@secure.istio.io.

Let’s obtain a JWT token with the above details.

    $ TOKEN=$(curl https://raw.githubusercontent.com/istio/istio/release-1.6/security/tools/jwt/samples/demo.jwt -s) && echo $TOKEN | cut -d '.' -f2 - | base64 --decode -

    {"exp":4685989700,"foo":"bar","iat":1532389700,"iss":"[testing@secure.istio.io](mailto:testing@secure.istio.io)","sub":"[testing@secure.istio.io](mailto:testing@secure.istio.io)"}

## Testing the Authorisation Policy

Now transmit a request with a valid JWT token.

    $ kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl "[http://httpbin.foo:8000/headers](http://httpbin.foo:8000/headers)" -s -o /dev/null -H "Authorization: Bearer $TOKEN" -w "%{http_code}\n"

    200

What about a request lacking a JWT token?

    $ kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl "[http://httpbin.foo:8000/headers](http://httpbin.foo:8000/headers)" -s -o /dev/null -w "%{http_code}\n"

    403

And the request is declined. JWT authorisation is working at this point.

## Validating Custom Claims

We can also validate custom claims apart from the subject and the issuer. Let’s implement a rule that a JWT should include a group claim with a value group1.

<iframe src="https://medium.com/media/636db38aa0074d684d9484d9bf6e1c4f" frameborder=0></iframe>

The above YAML includes a when directive that permits requests only when the groups claim contains a value group1.

## Testing Custom Claims

Now let’s test the configuration. Create a JWT containing a claim called groups with values group1 and group2.

    $ TOKEN_GROUP=$(curl https://raw.githubusercontent.com/istio/istio/release-1.6/security/tools/jwt/samples/groups-scope.jwt -s) && echo $TOKEN_GROUP | cut -d '.' -f2 - | base64 --decode -

    {"exp":3537391104,"groups":["group1","group2"],"iat":1537391104,"iss":"[testing@secure.istio.io](mailto:testing@secure.istio.io)","scope":["scope1","scope2"],"sub":"[testing@secure.istio.io](mailto:testing@secure.istio.io)"}

Call the httpbin microservice with the above JWT.

    $ kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl "[http://httpbin.foo:8000/headers](http://httpbin.foo:8000/headers)" -s -o /dev/null -H "Authorization: Bearer $TOKEN_GROUP" -w "%{http_code}\n"

    200

And you get a successful response.

What about a JWT that doesn’t contain the groups claim?

    $ kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl "[http://httpbin.foo:8000/headers](http://httpbin.foo:8000/headers)" -s -o /dev/null -H "Authorization: Bearer $TOKEN" -w "%{http_code}\n"

    403

And this is rejected. Well done! You’ve successfully implemented custom-claims authorisation.

## Conclusion

Thanks for reading! I hope you enjoyed the article.

In the next article “[Istio Service Mesh on Multi-Cluster Kubernetes Environment](https://medium.com/better-programming/istio-service-mesh-on-multi-cluster-kubernetes-environment-518094cdcdc4)”, I will discuss managing an Istio Service Mesh on Multi-Cluster Kubernetes Environment, so see you there!
