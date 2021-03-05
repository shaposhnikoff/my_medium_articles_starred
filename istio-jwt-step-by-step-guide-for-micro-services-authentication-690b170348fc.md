
# Istio & JWT: Step by Step Guide for Micro-Services Authentication

Istio is the most popular service mesh out there. What’s a Service Mesh? A service mesh is an abstraction layer between your application and Kubernetes.

![](https://cdn-images-1.medium.com/max/4404/1*neo38T3rL9UMNBSRUeu4eQ.jpeg)

It has the capability to control your network and give you some rich functionalities that you don’t get with Kubernetes. Istio has been designed from scratch keeping Kubernetes in mind. So it integrates seamlessly with any Kubernetes application. And at some point of time if you decide not to use Istio, you can just uninstall it and your Kubernetes application will work as usual — without the Istio features obviously.

One of the features that Istio come with out of the box is the ability to validate the JWT tokens that comes inside a client request header(if the server implements JWT token Authentication that is). So before the user request hits your application code, Istio can

1. validate the JWT token inside the request header

1. forward request with valid JWT to application code

1. deny traffic with invalid JWT

If you are developing micro-services, then this feature can offload a lot of Authentication(AuthN) and Authorization(AuthZ) logic from your application code. Your application code doesn’t need to worry about the validity of the incoming request. Istio takes care of it for you.

If you are coming from a software development background, you know that when developing an application with any of the programming languages or frameworks, you have to build your own JWT authentication and authorization system. Some frameworks has JWT support out of the box. But in general, most of the tutorials and blogs only cover the very basics of what JWT has to offer. That too for only monolithic applications.

But if you are building micro-services, then setting up authentication and authorization for tens and hundreds of services can become very daunting. You may have to write the same Authentication and Authorization logic for every micro-service in your architecture. And support for these JWT authentication and authorization across languages and frameworks vary quite a lot. Istio can now do the task of validation for you.

Hopefully I have been able to sell you of the benefits of Istio JWT authentication feature. Let’s dive in a bit deeper.

## What we will discuss

* Take a look at Session based AuthN and AuthZ Steps

* JWT AuthN and AuthZ Steps

* Analysing JWT AuthN Steps

* JWT in the Context of a Micro-Service Architecture

* Where Does Istio Fit in this Whole Equation?

* Steps to Implement JWT in Istio

* Test out the Whole Shebang

## Prerequisites

* You have a good understanding of JWT Authentication & Authorization system.

* You have a good understanding how RSA public/private key pair based JWT works. Check out this blog post for more clarity.
[**JWT: The Complete Guide to JSON Web Tokens**
blog.angular-university.io](https://blog.angular-university.io/angular-jwt/#/JWT:%20The%20Complete%20Guide%20to%20JSON%20Web%20Tokens)

* You have an understanding how Istio uses Envoy Sidecar Proxies for all the magic that it does

* You have a basic understanding of the Istio Control plane and Data Plane

## Before JWT — Session based AuthN and AuthZ Steps

Before the rise of JWT, the more popular approach of authentication and authorization was session based. In session based method, the steps involved were

![](https://cdn-images-1.medium.com/max/8586/1*6kuetIBrZUE4pbhwRFPAtg.jpeg)

1. User provided their username and password

1. Server generates a **Session ID** and stores it on **server memory**

1. Server returns **Session ID **to the client

1. Client stores the **Session ID** on browser memory

1. When client makes any subsequent request, it sends the **Session ID**

1. Server matches the client **Session ID** with the IDs stored in it’s memory

1. If the server can validate the **Session ID**, it sends client’s requested data

### Problems with Session Based Approach

* **Overhead:** This works well with small services. But the problem with this approach is that the server has to store the Session ID in it’s memory. Managing that is an overhead that the server has to take care of. To make user experience consistent, you have to implement sticky sessions so that a client is served by the same server in the subsequent requests.

* **Scalability:** Then you can not horizontally scale your application just by adding more servers running. You have to take care of the memory that stores the Session ID and replicate it across the new servers.

## JWT AuthN and AuthZ Steps

With the help of Cryptography, JWT has introduced a system to complete this process without the need of storing anything on the server memory. In this process, you only store the JWT on the client side and server can validate the identity of the client just using a secret or public/private key. So the steps involved in JWT authentication system are as follows

![](https://cdn-images-1.medium.com/max/7752/1*73S5myhEoJbKAxOiRQtjjg.jpeg)

1. User provides their username and password

1. Server generates JWT token

1. Server sends the JWT token to the Client**(doesn’t store anything on server)**

1. Client stores the JWT Token in the browser memory

1. When client makes any subsequent request, it sends the **JWT Token**

1. Using the private/public key, server validates the token

1. If the server can validate the **JWT Token**, it sends client’s requested data

### Advantage of JWT compared to Session based Approach

* **Highly Scalable:** The advantage of this approach is that there is nothing stored in the server side. So scaling your application is simpler. In the context of Kubernetes, if your application needs more pods, Kubernetes can just bring up more pods without thinking about the stuff stored in memory.

* **Stateless & Thus Less Overhead:** Nothing is stored on the server side. So your server is stateless. And a client’s subsequent request can be handled by any of the servers behind a reverse proxy.

For more information on the comparison of JWT vs Session based AuthN and AuthZ, you can refer to this blog.
[**The Ins and Outs of Token Based Authentication**
scotch.io](https://scotch.io/tutorials/the-ins-and-outs-of-token-based-authentication/#/The%20Ins%20and%20Outs%20of%20Token%20Based%20Authentication)

## Analysing JWT AuthN Steps

One important thing to notice is that we can divide the server side tasks into two categories.

1. JWT Token Generation

1. JWT Token Validation

When you are developing a monolithic application, both of these tasks are done by the same application. But what happens when you have a group of micro-services? Where do you generate and where do you validate the token?

## JWT in the Context of a Micro-Service Architecture

We will take a look at three different approaches of implementing JWT in a micro-services architecture.

1. **Somewhat Scaleable but Secure:** A single micro-service can generate the token and only that service can validate the token

1. **Somewhat Scaleable but Not Secure**: One micro-service generates the token, but anyone with the private key can validate the token

1. **Scaleable and Secure:** One micro-service generates the token, but anyone with the public key can validate the token

### Generation and Validation on same service

If the authentication micro-service is responsible for both token generation and validation, then the systems become very chatty.

![](https://cdn-images-1.medium.com/max/6726/1*GnNQGefYnvqUYMRQ4LoMyQ.jpeg)

In the diagram, the client first authenticates itself by providing it’s username and password. Then when it makes a request to Service A, before serving the client’s request, it needs to validate the JWT token provided by the client. In this setup, since the authentication micro-service is responsible for token validation as well, Service A will make a request to the authentication service to validate the token.

Before Service A can send back the response to the client, it needs some data from Service B and Service C. And Service C in tern is dependent on Service D. So serving one client request is dependent on 4 different micro-services and each of these services before processing the incoming request, the need to first validate the incoming JWT token by calling the AuthN Service.

In a small group of micro-services it isn’t that big of a deal. But

* What happens when you have hundreds of micro-services? Every service makes a request to the authentication micro-service for validating the incoming JWT token. In these sort of complicated ecosystem of micro-services, sometime one client request gets served by the combined output of several micro-services. So replying to one single request, might require several JWT validation request.

* More importantly, in each of the micro-service, you have to implement the logic for calling the AuthN Service for validating the token and also do all the error handling that comes with it.

### Generates one service and Validates other services using Private key

Token validation task is the part that makes the system more chatty. You can decouple the token validation part from the token generation part by copying and pasting the private key on all the services that need to validate the JWT token.

![](https://cdn-images-1.medium.com/max/6966/1*YsmEoOVQhwFA9bdatqAwmA.jpeg)

But this approach is very insecure.

1. How do you share the private key among all the services securely?

1. If anyone of your service gets compromised and anyone can get the private key, he can generate his own token and access any other systems.

### Generates one service and Validates other services using Public key

If your Authentication micro-service uses public/private key pair based authentication system then any service that needs to validate the JWT Token can use the public key.

![](https://cdn-images-1.medium.com/max/6726/1*Q-0cMXq0R8UVRQrXZ0eagw.jpeg)

The private key stays secured in one place, inside the Authentication service and only that can generate JWT token. But you don’t need the private key to validate the token. Having the public key is good enough.

## Where Does Istio Fit in this Whole Equation?

Now that you have an understanding of the design considerations, we will discuss how Istio helps you implement the public/private key pair based JWT authentication which is the most scaleable and secure choice for a micro-services architecture.

So basically in public/private key pair based JWT authentication system, there will be one dedicated micro-service that will take care of the login task. It will take the username and password, generate the JWT token using the private key and send it back to the client.

And after that when the client tries to communicate to any other service using the JWT token, that service will use the public key and validate the token. In this scenario, every service that you have in the system will have to implement the application logic to

1. Get the public key from a public URL or from memory

1. Validate the token using the public key

1. Implement the case of a valid token

1. Implement the case of an invalid token

This may be really easy and straight forward to implement in one micro-service. But imagine if you have hundreds of micro-services with a variety of programming languages and framework. Then for every application has to implement and manage all these things. And different language and framework has their own implementation of these things which might not be consistent.

### **Istio to the rescue… :D**

Istio takes care of the task of validating the JWT tokens in the incoming user requests. So if you implement Istio JWT authentication feature, your application code doesn’t need to bother about the JWT token validation. Istio will do it for you. Not JWT token generation. Istio will not generate the tokens for you. You have to have an Authentication micro-service that generates the token.

### How does Istio do that?

In Istio JWT authentication is defined as a ***Request Authentication*** feature. A Custom Resource Definition(CRD) named ***RequestAuthentication*** is used to tell the control plane where the JWT public key can be found and who issued the JWT token.

After you define and configure the ***RequestAuthentication ***CRD with the ***jwksUri **or** jwks ***and the issuer information***(will explain the meaning of these in more detail in later section)***, ***Istiod — ***the master of the control plane tells all the Envoy proxy about this JWT authentication policy information.

Now when a request comes into the service mesh, before it can reach to your application, it has to go through Envoy proxy sidecar container that sits right beside the application container. This Envoy proxy can now validate the JWT token that the incoming request is carrying using the public key that is available in the ***jwks/jwksUri and the issuer ***information. If the token can be validated using the public key and also the issuer information matches with the ***iss ***that is mentioned in the JWT token payload, then ***Envoy*** forwards the request directly to the application container. And the application container can now handle the request without bothering about the validity of the request payload.

Awesome…! Right?

So using Istio you can offload a lot of your application level security concern.

## Steps to implement JWT in Istio

Let’s see how we can implement these things in a sample project. We will follow the Istio docs
[**Authentication Policy**
*This task covers the primary activities you might need to perform when enabling, configuring, and using Istio…*istio.io](https://istio.io/docs/tasks/security/authentication/authn-policy/#end-user-authentication)

which is fairly straight forward and easy to understand. The steps that you need to follow to see the effect of JWT authentication policy are as follows

### Prerequisites

1. You already have a Kubernetes cluster setup. I will be using GKE cluster.

1. You have installed latest stable release of Istio with ***Demo*** profile. In my demo I will be using GKE and Istio 1.5. Although GKE comes with an option to install Istio at cluster creation time by clicking a checkbox, I would encourage you to use ***istioctl ***to install the latest stable version of Istio. [Follow the Istio documentation](https://istio.io/docs/setup/getting-started/#download)

1. You have an understanding of what are the steps needed to Integrate Istio with an existing Kubernetes application. If you are new to Istio, please refer to this story of mine.
[**Istio and Google Kubernetes Engine: From 0 to 1**
*You have some experience with Kubernetes and deployed basic application on Kubernetes. Now you want to try out Istio…*medium.com](https://medium.com/intelligentmachines/istio-and-google-kubernetes-engine-from-0-to-1-8d84b4cea20)

It might help you to get up to speed.

### Setup Demo Application

In this step we will setup a demo application that we will be using to test the Istio JWT features.We will use the ***httpbin ***application form Istio docs.
[**Authentication Policy**
*This task covers the primary activities you might need to perform when enabling, configuring, and using Istio…*istio.io](https://istio.io/docs/tasks/security/authentication/authn-policy/#before-you-begin)

The Istio docs give show you the mTLS and JWT authentication on the same application setup. I will only be demonstrating the JWT features. For that I have consolidated all the commands needed to setup the demo. Just copy-paste the below commands and it should work like a charm.

**Install the demo application**

<iframe src="https://medium.com/media/39636c517656a90e474e821c87a1e2a3" frameborder=0></iframe>

**Install a new Gateway**

<iframe src="https://medium.com/media/74fcc6ba8fc9ec41beade01942a75b59" frameborder=0></iframe>

**Install the Virtual Service**

<iframe src="https://medium.com/media/a9f1dc2ca5609b9ec9e978151caab6f6" frameborder=0></iframe>

**Test the setup**

    export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

    curl $INGRESS_HOST/headers -s -o /dev/null -w "%{http_code}\n"

Our demo application is all set. Now we will setup JWT authentication for this demo application to make it secure.

You can setup JWT authentication in any language stack — Python, Nodejs, Java or Go.It doesn’t matter. What you need to be aware of is that you need to setup a RSA public/private key pair based JWT authentication system. If you have a JWT Authentication system that only uses a ***Secret Text ***in a variable or file to both generate and validate JWT token like most of the traditional monolithic applications do and JWT tutorial show you, it will not work with Istio. Istio only takes the Public Key or URL of the Public Key to validate the JWT token. That is how Istio works.

Now to give away the JWT Token validation responsibility to Istio, we have to follow these few steps

1. Generate a public/private key pair

1. Create a JWKS(JSON Web Key Set) representation of public key

1. Tell Istio where to find the JWKS using the ***RequestAuthentication*** CRD

1. Test Out the new feature

Let’s get down to business

### Step 1(a): Generate Public/Private Key Pair Using Python

This script generates RSA public/private key pair using python. And uses the Keys to Generate JWT Token. Just run this script following the instructions in the [GitHub](https://github.com/fai555/Istio-and-JWT) repo README.md and you will get a public key, private key and a valid JWT Token generated using that private key and can be validated using the public key.

<iframe src="https://medium.com/media/f2b4bb3b1a5a35f26063cd92b58b25b9" frameborder=0></iframe>

### Step 1(b): Generate Public/Private Key Pair using openssl

In this approach we will use an open-source tool named ***openssl ***to generate the token. You can also use ***ssh-keygen ***for this which generates effectively the same thing with different public key representation. But we will use ***openssl.***

    mkdir keys

    cd keys

    *openssl genrsa -out private-key.pem 2048*

    *openssl rsa -in private-key.pem -pubout -out public-key.pem*

After generating the keys, run the following script to generate and validate the token.

<iframe src="https://medium.com/media/c349e6fe52518f7f2215e196d21f4d23" frameborder=0></iframe>

### **Note:**

When generating the tokens, in the payload, there are some reserved keys like **iss, sub, aud **and several others that are not mandatory to define when you are working with JWT system in general. But when working with Istio, your AuthN system need to put in these these fields in the payload.

### Step 2: Create a JWKS(JSON Web Key Set) Representation of Public Key

Just creating the raw public key is not enough. Istio accepts the public key in a JWKS format. You can give Istio an array of public keys which Istio will use to validate the incoming the token.

If you have a public key, you need to convert it into JSON format not PEM format. An example public key is given below

    {"e":"AQAB","kty":"RSA","n":"3LlzeRY6gbIVwGO7AxO1bN3-CgWwIpWOT8m485AzkOdhxgCWc2F-3OqAigDyyDMqXtH1ovCaZnEIf3ZkJin7Y_zC48TNQwlKnuM29CrTjnYR1c_w30ZT4PNIisEwLKuEX5uRHuIrKYBxwwVf4eqoFmtpZbrmwDPCA1ZMFox0v40q1m_SecCB286alE42Ohb6j0ZuntjO5rg2ZyQt3EmxEDPE2Iuh737gYhXLuFhTiYH5S_kFokX1Yv0RdUyiGcmaxXgGaF3iglnsOHv9209uwlzrcDAouOD7PYbLjCoqpWydVLyxcJGqjF5i7CK36q_SVmpGHbIsdOlZQLWNA97AgQ"}

Then you need to encapsulate this key as an item in the following array. Like this.

    { "keys":[ 
    

    {"e":"AQAB","kty":"RSA","n":"3LlzeRY6gbIVwGO7AxO1bN3-CgWwIpWOT8m485AzkOdhxgCWc2F-3OqAigDyyDMqXtH1ovCaZnEIf3ZkJin7Y_zC48TNQwlKnuM29CrTjnYR1c_w30ZT4PNIisEwLKuEX5uRHuIrKYBxwwVf4eqoFmtpZbrmwDPCA1ZMFox0v40q1m_SecCB286alE42Ohb6j0ZuntjO5rg2ZyQt3EmxEDPE2Iuh737gYhXLuFhTiYH5S_kFokX1Yv0RdUyiGcmaxXgGaF3iglnsOHv9209uwlzrcDAouOD7PYbLjCoqpWydVLyxcJGqjF5i7CK36q_SVmpGHbIsdOlZQLWNA97AgQ"}

    
    ]}

This is a valid JWKS.

### **Step 3: Tell Istio where to Find the JWKS using the *RequestAuthentication* CRD**

To tell Istio to validate the JWT tokens in the incoming request, we have to define a CRD named ***RequestAuthentication. ***To enable Istio with the capability to validate the tokens we need to provide 2 crucial information to Istio in that ***RequestAuthentication ***CRD.

1. **Who issued the JWT token?** Basically when your JWT Authentication micro-service will generate the JWT token, it will have to specify the ***iss*** field in the payload. This ***iss*** field has to match with the issuer field in the ***RequestAuthentication ***CRD.

1. **Where is the public key is stored? **You can specify this value using either of the two ***jwtRules*** options

* **jwks(JSON Web Key Set)** or

* **jwksuri(JSON Web Key Set URI)**

### jwks(JSON Web Key Set)

If using ***jwks, ***what you effectively do is embed the jwks that we created in the previous step directly as a value like below

<iframe src="https://medium.com/media/7c302b9cc58a52506cb744f70a0c7b88" frameborder=0></iframe>

### jwksUri**(JSON Web Key Set URI)**

If using ***jwksUri, ***what you have to do is put the*** JWKS ***that we created in the previous step in a public URL. For this demo I am using the public GitHub repository I am using to store the source code of this blog and using the URL for the raw GitHub file.

[https://raw.githubusercontent.com/fai555/Istio-and-JWT/master/json-web-key-set.json](https://raw.githubusercontent.com/fai555/Istio-and-JWT/master/json-web-key-set.json)

And specify that URL in the ***jwksUri*** field like below.

<iframe src="https://medium.com/media/c78deb9175d4700725e3ec820d2b47c6" frameborder=0></iframe>

## Test out the Whole Shebang

If you have come this far then you should have

* the ***demo*** app running on a Kubernetes cluster

* a ***Gateway***

* a*** VirtualService***

* a*** RequestAuthentication*** CRD and

* a ***JWT ***Token Printed out in the console

Now you only need 2 more things. First put the JWT token inside a environment variable named **TOKEN.**

    export Token=<GENERATED JWT TOKEN>

And get the Ingress IP address

    export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

If you have all of these checked, you can start testing out the Istio JWT features. We will send request to our demo application using ***curl*** with invalid token, valid token and no token in the request header.

If you send a request with invalid token, see what happens.

    curl --header "Authorization: Bearer bla-bla-bla" $INGRESS_HOST/headers -s -o /dev/null -w "%{http_code}\n"

You will get a **401 Unauthorized Error. **The demo app that we set up has no Authentication system of any sort. So how did we get the **401 Unauthorized Error? **Istio is taking care of validating the incoming JWT token in the user request for you. **Nice!!!**

Then we will send a request with valid token

    curl --header "Authorization: Bearer $TOKEN" $INGRESS_HOST/headers -s -o /dev/null -w "%{http_code}\n"

You will get a successful request with HTTP response code 200 in the console. Let’s send a request with no header at all.

    curl $INGRESS_HOST/headers -s -o /dev/null -w "%{http_code}\n"

You will get a 200 in the console. By default Istio runs these Authentication policy check in permissive mode. Meaning you can send request if you provide a valid token or provide no token at all. It helps you in the gradual migration process when you are moving to an Istio based system. Not blocking your entire operation by being too strict.
> To reject requests without valid tokens, add an authorization policy with a rule specifying a DENY action for requests without request principals, shown as notRequestPrincipals: ["*"] in the following example. Request principals are available only when valid JWT tokens are provided. The rule therefore denies requests without valid tokens.

<iframe src="https://medium.com/media/f28c9e870620faa0db40e463cbc8a6af" frameborder=0></iframe>

Let’s test it out. Send same three requests with invalid, valid and no tokens

    curl --header "Authorization: Bearer bla-bla-bla" $INGRESS_HOST/headers -s -o /dev/null -w "%{http_code}\n"

Response: 401

Valid Token

    curl --header "Authorization: Bearer $TOKEN" $INGRESS_HOST/headers -s -o /dev/null -w "%{http_code}\n"

Response: 200

No Token:

    curl $INGRESS_HOST/headers -s -o /dev/null -w "%{http_code}\n"

Response: 403

Now it’s only allowing requests with valid tokens.

Effectively the scripts that we are using to generate the JWT tokens, that is mimicking the role of an Authentication micro-service. An authentication micro-service gives you a JWT token if you have a valid username and password. But the python scripts gives you valid JWT token if you just execute it. Any Authentication micro-service will have the same implementation logic as the python scripts provided.

So we have decoupled the JWT Token Generation and Validation steps. Your Authentication micro-service will do the JWT token Generation and Istio will handle the token validation.

Hope I was able to drive the point home. I know it’s too long. And at this point you may have already forgotten what the point was. :P Read the title again. It might make sense now.

This was an effort to document what I learned. Hope it helps you as well.

## Important links

When starting out, I wasn’t aware of a lot of the related concepts we well. So I had to google quite a lot to understand all these and put it all together to implement it in a real project. Some of those links are provided below.

### **JWT with Public Key Signature**
[**python-jwt**
*Module for generating and verifying JSON Web Tokens. Note: From version 2.0.1 the namespace has changed from jwt to…*pypi.org](https://pypi.org/project/python-jwt/)
[**JSON Web Tokens with Public Key Signatures**
*JSON Web Tokens offer a simple and powerful way to generate tokens for APIs. These tokens carry a payload that is…*blog.miguelgrinberg.com](https://blog.miguelgrinberg.com/post/json-web-tokens-with-public-key-signatures)
[**Welcome to JWCrypto's documentation! - JWCrypto 0.8.dev1 documentation**
*JWCrypto is an implementation of the Javascript Object Signing and Encryption (JOSE) Web Standards as they are being…*jwcrypto.readthedocs.io](https://jwcrypto.readthedocs.io/en/latest/)
[**(Python) How to Generate a JSON Web Key (JWK)**
*Demonstrates how to generate the following types of JSON Web Keys: RSA key pair EC key pair Octet sequence key…*www.example-code.com](https://www.example-code.com/python/jwk_generate_json_web_key.asp)
[**(Node.js) How to Generate a JSON Web Key (JWK)**
*Demonstrates how to generate the following types of JSON Web Keys: RSA key pair EC key pair Octet sequence key…*www.example-code.com](https://www.example-code.com/nodejs/jwk_generate_json_web_key.asp)

### JWT Details
[**JWT: The Complete Guide to JSON Web Tokens**
*This post is the first part of a two-parts step-by-step guide for implementing JWT-based Authentication in an Angular…*blog.angular-university.io](https://blog.angular-university.io/angular-jwt/)
[**The Ins and Outs of Token Based Authentication**
*Introduction Token based authentication is prominent everywhere on the web nowadays. With most every web company using…*scotch.io](https://scotch.io/tutorials/the-ins-and-outs-of-token-based-authentication)

### Googles JWKSURI for JWT token validation

The below link shows how Google stores their JWKS. The same way that we did on our demo.

[https://www.googleapis.com/oauth2/v3/certs](https://www.googleapis.com/oauth2/v3/certs)

### Troubleshooting
[**JWT - SSH-KEYGEN vs OPENSSL**
*We just tried to set up JWT using RSA on our TYK but found some problem When we use Openssl command to generate keypair…*community.tyk.io](https://community.tyk.io/t/jwt-ssh-keygen-vs-openssl/2877)
[**What are the differences between ssh generated keys(ssh-keygen) and OpenSSL keys (PEM)and what is…**
*Thanks for contributing an answer to Information Security Stack Exchange! Please be sure to answer the question…*security.stackexchange.com](https://security.stackexchange.com/questions/29876/what-are-the-differences-between-ssh-generated-keysssh-keygen-and-openssl-keys)
