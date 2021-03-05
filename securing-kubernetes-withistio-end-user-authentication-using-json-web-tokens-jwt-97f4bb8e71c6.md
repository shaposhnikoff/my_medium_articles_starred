
# Istio End-User Authentication for Kubernetes using JSON Web Tokens (JWT) and Auth0

In the recent post, Building a Microservices Platform with Confluent Cloud, MongoDB Atlas, Istio, and Google Kubernetes Engine, we built and deployed a microservice-based, cloud-native API to Google Kubernetes Engine, with Istio 1.0.x, on Google Cloud Platform. For brevity, we intentionally omitted a few key features required to operationalize and secure the API. These missing features included HTTPS, user authentication, request quotas, request throttling, and the integration of a full lifecycle API management tool, like Google Apigee.

In a follow-up post, [Securing Your Istio Ingress Gateway with HTTPS](https://medium.com/@GaryStafford/securing-your-istio-ingress-gateway-with-https-8c59972cb5d7), we disabled HTTP access to the API running on the GKE cluster. We then enabled bidirectional encryption of communications between a client and GKE cluster with HTTPS.

In this post, we will further enhance the security of the Storefront Demo API by enabling [Istio end-user authentication](https://istio.io/help/ops/security/end-user-auth/) using JSON Web Token-based credentials. Using JSON Web Tokens (JWT), pronounced ‘*jot*’, will allow Istio to authenticate end-users calling the Storefront Demo API. We will use [Auth0](https://auth0.com/), an Authentication-as-a-Service provider, to generate JWT tokens for registered Storefront Demo API consumers, and to validate JWT tokens from Istio, as part of an OAuth 2.0 token-based authorization flow.

![](https://cdn-images-1.medium.com/max/2000/0*4kPp_M9gsTk7l9dw)

## JSON Web Tokens

Token-based authentication, according to Auth0, works by ensuring that each request to a server is accompanied by a signed token which the server verifies for authenticity and only then responds to the request. JWT, according to [JWT.io](https://jwt.io/introduction/), is an open standard ([RFC 7519](https://tools.ietf.org/html/rfc7519)) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed. Other common token types include Simple Web Tokens (SWT) and Security Assertion Markup Language Tokens (SAML).

JWTs can be signed using a secret with the [Hash-based Message Authentication Code](https://en.wikipedia.org/wiki/HMAC) (HMAC) algorithm, or a public/private key pair using [Rivest–Shamir–Adleman](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) (RSA) or [Elliptic Curve Digital Signature Algorithm](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm) (ECDSA). Authorization is the most common scenario for using JWT. Within the token payload, you can easily specify user roles and permissions as well as resources that the user can access.

A registered API consumer makes an initial request to the Authorization server, in which they exchange some form of credentials for a token. The JWT is associated with a set of specific user roles and permissions. Each subsequent request will include the token, allowing the user to access authorized routes, services, and resources that are permitted with that token.

## Auth0

To use JWTs for end-user authentication with Istio, we need a way to authenticate credentials associated with specific users and exchange those credentials for a JWT. Further, we need a way to validate the JWTs from Istio. To meet these requirements, we will use Auth0. [Auth0](https://auth0.com/) provides a universal authentication and authorization platform for web, mobile, and legacy applications. According to [G2 Crowd](https://www.g2crowd.com/categories/customer-identity-and-access-management), competitors to Auth0 in the [Customer Identity and Access Management](https://en.wikipedia.org/wiki/Customer_Identity_Access_Management) (CIAM) Software category include Okta, Microsoft Azure Active Directory (AD) and AD B2C, Salesforce Platform: Identity, OneLogin, Idaptive, IBM Cloud Identity Service, and Bitium.

![](https://cdn-images-1.medium.com/max/2000/0*clvinu4H43_sW0oJ)

Auth0 currently offers four pricing plans: Free, Developer, Developer Pro, and Enterprise. Subscriptions to plans are on a monthly or discounted yearly basis. For this demo’s limited requirements, you need only use Auth0’s Free Plan.

![](https://cdn-images-1.medium.com/max/2000/0*QN_kOpfaHij65c6n)

The OAuth 2.0 protocol [defines four flows](https://auth0.com/docs/protocols/oauth2), or *grants types*, to get an Access Token, depending on the application architecture and the type of end-user. We will be simulating a third-party, external application that needs to consume the Storefront API, using the [Client Credentials](https://oauth.net/2/grant-types/client-credentials/) grant type. According to [Auth0](https://auth0.com/docs/api-auth/tutorials/client-credentials), The Client Credentials Grant, defined in The OAuth 2.0 Authorization Framework [RFC 6749, section 4.4,](https://tools.ietf.org/html/rfc6749#section-4.4) allows an application to request an Access Token using its Client Id and Client Secret. It is used for non-interactive applications, such as a CLI, a daemon, or a Service running on your backend, where the token is issued to the application itself, instead of an end user.

![](https://cdn-images-1.medium.com/max/2000/0*LhNVo8eFFFaSVwJ8.png)

With Auth0, we need to create two types of entities, an Auth0 [API](https://auth0.com/docs/apis) and an Auth0 [Application](https://auth0.com/docs/applications). First, we define an Auth0 API, which represents the Storefront API we are securing. Second, we define an Auth0 Application, a consumer of our API. The Application is associated with the API. This association allows the Application (*consumer of the API*) to authenticate with Auth0 and receive a JWT. Note there is no direct integration between Auth0 and Istio or the Storefront API. We are facilitating a decoupled, mutual trust relationship between Auth0, Istio, and the registered end-user application consuming the API.

Start by creating a new Auth0 API, the ‘Storefront Demo API’. For this demo, I used my domain’s URL as the Identifier. For use with Istio, choose RS256 (RSA Signature with SHA-256), an asymmetric algorithm that uses a public/private key pair, as opposed to the HS256 symmetric algorithm. With RS256, Auth0 will use the same private key to both create the signature and to validate it. Auth0 has published a good [post](https://auth0.com/blog/navigating-rs256-and-jwks/) on the use of RS256 vs. HS256 algorithms.

![](https://cdn-images-1.medium.com/max/2000/0*RUxJs0Os3EQ1ZQBK)

![](https://cdn-images-1.medium.com/max/2000/0*iGw3Hh4xNdtgdUf9)

Auth0 allows granular access control to your API through the use of Scopes. The permissions represented by the Access Token in OAuth 2.0 terms are known as scopes, According to [Auth0](https://auth0.com/docs/scopes/current). The scope parameter allows the application to express the desired scope of the access request. The scope parameter can also be used by the authorization server in the response to indicate which scopes were actually granted.

Although it is necessary to define and assign at least one scope to our Auth0 Application, we will not actually be using those scopes to control fine-grain authorization to resources within the Storefront API. In this demo, if an end-user is authenticated, they will be authorized to access all Storefront API resources.

![](https://cdn-images-1.medium.com/max/2000/0*f2O86eW8VVWLcNMM)

Next, define a new Auth0 Machine to Machine (M2M) Application, ‘Storefront Demo API Consumer 1’.

![](https://cdn-images-1.medium.com/max/2000/0*dQILS2zrQ0db62Dl)

Next, authorize the new M2M Application to request access to the new Storefront Demo API. Again, we are not using scopes, but at least one scope is required, or you will not be able to authenticate, later.

![](https://cdn-images-1.medium.com/max/2000/0*JZsu-5HGs6r2VUC-)

Each M2M Application has a unique Client ID and Client Secret, which are used to authenticate with the Auth0 server and retrieve a JWT.

![](https://cdn-images-1.medium.com/max/2000/0*3aAtpp1dvuLukYLb)

Multiple M2M Applications may be authorized to request access to APIs.

![](https://cdn-images-1.medium.com/max/2000/0*07j_q7jgB9IwFB4_)

In the Endpoints tab of the Advanced Application Settings, there are a series of OAuth URLs. To authorize our new M2M Application to consume the Storefront Demo API, we need the ‘OAuth Authorization URL’.

![](https://cdn-images-1.medium.com/max/2000/0*N3AnHviagp4qFYrg)

To test the Auth0 JWT-based authentication and authorization workflow, I prefer to use Postman. Conveniently, Auth0 provides a [Postman Collection](https://auth0.com/docs/api/info) with all the HTTP request you will need, already built. Use the [Client Credentials](https://auth0.com/docs/api-auth/tutorials/client-credentials) POST request. The grant_type header value will always be client_credentials. You will need to supply the Auth0 Application’s Client ID and Client Secret as the client_id and client_secret header values. The audience header value will be the API Identifier you used to create the Auth0 API earlier.

![](https://cdn-images-1.medium.com/max/2000/0*u3b-vdxjivKCVMHG)

If the HTTP request is successful, you should receive a JWT access_token in response, which will allow us to authenticate with the Storefront API, later. Note the scopes you defined with Auth0 are also part of the response, along with the token’s TTL.

## jwt.io Debugger

For now, test the JWT using the [jwt.io](https://jwt.io/) Debugger page. If everything is working correctly, the JWT should be successfully validated.

![](https://cdn-images-1.medium.com/max/2000/0*FovVNOG6MN-Rd3L0)

## Istio Authentication Policy

To enable Istio end-user authentication using JWT with Auth0, we add an [Istio Policy authentication resource](https://istio.io/docs/reference/config/istio.authentication.v1alpha1/#Policy) to the existing set of deployed resources. You have a few choices for end-user authentication, such as:

1. Applied globally, to all Services across all Namespaces via the Istio Ingress Gateway;

1. Applied locally, to all Services within a specific Namespace (i.e. uat);

1. Applied locally, to a single Service or Services within a specific Namespace (i.e prod.accounts);

In reality, since you would likely have more than one registered consumer of the API, with different roles, you would have more than one Authentication Policy applied the cluster.

For this demo, we will enable global end-user authentication to the Storefront API, using JWTs, at the [Istio Ingress Gateway](https://istio.io/docs/tasks/traffic-management/ingress/). To create an Istio Authentication Policy resource, we use the Istio Authentication API version authentication.istio.io/v1alpha1([*gist](https://gist.github.com/garystafford/2437110cfb64a16ca3cefa224fb8a3c5)*).

<iframe src="https://medium.com/media/46a3fc97e8b30320963ffa633a74bbcd" frameborder=0></iframe>

The single audiences YAML map value is the same Audience header value you used in your earlier Postman request, which was the API Identifier you used to create the Auth0 Storefront Demo API earlier. The issuer YAML scalar value is Auth0 M2M Application’s Domain value, found in the ‘Storefront Demo API Consumer 1’ Settings tab. The jwksUri YAML scalar value is the JSON Web Key Set URL value, found in the Endpoints tab of the Advanced Application Settings.

![](https://cdn-images-1.medium.com/max/2000/0*nUePpQRYlip3nMl-)

The JSON Web Key Set URL is a publicly accessible endpoint. This endpoint will be accessed by Istio to obtain the public key used to authenticate the JWT.

![](https://cdn-images-1.medium.com/max/2000/0*cBMyFS2aRIpqVQz6)

Assuming you have already have deployed the Storefront API to the GKE cluster, simply apply the new Istio Policy. We should now have end-user authentication enabled on the Istio Ingress Gateway using JSON Web Tokens.

    kubectl apply -f ./resources/other/ingressgateway-jwt-policy.yaml

## Finer-grain Authentication

If you need finer-grain authentication of resources, alternately, you can apply an Istio Authentication Policy across a Namespace and to a specific Service or Services. Below, we see an example of applying a Policy to only the uat Namespace. This scenario is common when you want to control access to resources in non-production environments, such as UAT, to outside test teams or a select set of external beta-testers. According to [Istio](https://istio.io/docs/tasks/security/authn-policy/#namespace-wide-policy), to apply Namespace-wide end-user authentication, across a single Namespace, it is necessary to name the Policy, default ([*gist](https://gist.github.com/garystafford/9b053ccbd9eddefa6bf43515becd2eb6)*).

<iframe src="https://medium.com/media/92a432af6b32b8fa7b0c4f094d94728f" frameborder=0></iframe>

Below, we see an even finer-grain Policy example, scoped to just the accounts Service within just the prod Namespace. This scenario is common when you have an API consumer whose role only requires access to a portion of the API. For example, a marketing application might only require access to the accounts Service, but not the orders or fulfillment Services ([*gist](https://gist.github.com/garystafford/40c17e18e677d95f48041dfa00545813)*).

<iframe src="https://medium.com/media/272e00d3ca1503f08024c748526cd3de" frameborder=0></iframe>

## Test Authentication

To test end-user authentication, first, call any valid Storefront Demo API endpoint, without supplying a JWT for authorization. You should receive a ‘401 Unauthorized’ HTTP response code, along with an Origin authentication failed. message in the response body. This means the Storefront Demo API is now inaccessible unless the API consumer supplies a JWT, which can be successfully validated by Istio.

![](https://cdn-images-1.medium.com/max/2000/0*pFMh-xPW9vMVKHPL)

Next, add authorization to the Postman request by selecting the ‘Bearer Token’ type authentication method. Copy and paste the JWT (access_token) you received earlier from the Client Credentials request. This will add an Authorization request header. In curl, the request header would look as follows ([*gist](https://gist.github.com/garystafford/38d0e46e2d6d3cfd88097669272cf5b8)*).

<iframe src="https://medium.com/media/8cf3205bf100df07b611e861496ac759" frameborder=0></iframe>

Make the request with Postman. If the Istio Policy is applied correctly, the request should now receive a successful response from the Storefront API. A successful response indicates that Istio successfully validated the JWT, located in the Authorization header, against the Auth0 Authorization Server. Istio then allows the user, the ‘Storefront Demo API Consumer 1’ application, access to all Storefront API resources.

![](https://cdn-images-1.medium.com/max/2000/0*ZP8Mz34M_wKkuben)

## Troubleshooting

Istio has several pages of online documentation on [troubleshooting](https://istio.io/help/ops/security/end-user-auth/) authentication issues. One of the first places to look for errors, if your end-user authentication is not working, but the JWT is valid, is the [Istio Pilot](https://istio.io/docs/concepts/traffic-management/#pilot-and-envoy) logs. The core component used for traffic management in Istio, Pilot, manages and configures all the Envoy proxy instances deployed in a particular Istio service mesh. Pilot distributes authentication policies, like our new end-user authentication policy, and secure naming information to the proxies.

Below, in Google Stackdriver Logging, we see typical log entries indicating the Pilot was unable to retrieve the JWT public key (recall we are using RS256 public/private key pair [asymmetric algorithm](https://en.wikipedia.org/wiki/Public-key_cryptography)). This particular error was due to a typo in the Istio Policy authentication resource YAML file.

![](https://cdn-images-1.medium.com/max/2000/0*deGsPeopiex0yhK9)

Below we see an Istio Mixer log entry containing details of a Postman request to the Accounts Storefront service /accounts/customers/summary endpoint. According to [Istio](https://istio.io/docs/concepts/policies-and-telemetry/), Mixer is the Istio component responsible for providing policy controls and telemetry collection. Note the apiClaims section of the textPayload of the log entry, corresponds to the Payload Segment of the JWT passed in this request. The log entry clearly shows that the JWT was decoded and validated by Istio, before forwarding the request to the Accounts Service.

![](https://cdn-images-1.medium.com/max/2000/0*EUK6SBiq4XcC9y2D)

## Conclusion

In this brief post, we added end-user authentication to our Storefront Demo API, running on GKE with Istio. Although still not Production-ready, we have secured the Storefront API with both HTTPS client-server encryption and JSON Web Token-based authorization. Next steps would be to add mutual TLS (mTLS) and a fully-managed API Gateway in front of the Storefront API GKE cluster, to provide [advanced API features](https://apigee.com/about/cp/api-management-features), like caching, quotas and rate limits.

*All opinions expressed in this post are my own and not necessarily the views of my current or past employers or their clients.*

*Originally published at [programmaticponderings.com](https://programmaticponderings.com/2019/01/06/securing-kubernetes-withistio-end-user-authentication-using-json-web-tokens-jwt/) on January 7, 2019.*
