
# Back to Microservices with Istio (Part 2) — Authentication & Authorization



*This is the second part of the article “[Back to Microservices with Istio](https://medium.com/@rinormaloku37/back-to-microservices-with-istio-part-1-827c872daa53)” (a prerequisite to follow along with the second part is completing the first one.)*

In the first article, we set up a Kubernetes cluster in which we deployed **Istio **and the sample microservice application “Sentiment Analysis”, to showcase Istio’s features.

Using Istio we kept our services small and rendered the following layers obsolete: Retries, Timeouts, Circuit Breakers, Tracing, Monitoring (shown in figure 1) and additionally, we enabled advanced testing and deployment techniques like A/B Testing, Shadowing and Canary Deployments.

![Figure 1. The Ceremony of a Microservice](https://cdn-images-1.medium.com/max/2000/1*rKvesxZlk_rlLAd5620Npw.png)*Figure 1. The Ceremony of a Microservice*

In this article, we will tackle the final layers of Authentication & Authorization and with Istio that’s a Joyride!

## Authentication & Authorization in Istio

I would have never believed that Authentication and Authorization would excite me! What on the technological spectrum could Istio possibly do to make these topics entertaining, and more importantly why should you be excited as well?

**The answer is simple:** Istio offloads these responsibilities from our services to the Envoy proxies, and by the time when requests reach our services they are already authenticated and authorized, and we just write the code that provides business value.

Sounds good? Let's just jump into it!

## Authentication with Auth0

As an Identity and Access Management server, we are going to use Auth0, which has a trial option, is intuitive to use, and I just love it! That said the same principles can be used for any [OpenID Connect implementation](https://openid.net/developers/certified/) like KeyCloak, IdentityServer and many more.

To get started, navigate to [Auth0 Portal](https://manage.auth0.com) login with your preferred account, create a tenant navigate under Applications > Default App and pick up the Domain, as seen in the image below:

![Figure 2. Default App in Auth0 management portal](https://cdn-images-1.medium.com/max/NaN/1*ftKR9RCcLA32Us8HA6-47w.png)*Figure 2. Default App in Auth0 management portal*

Update the file resource-manifests/istio/security/auth-policy.yaml to use your domain:

<iframe src="https://medium.com/media/98bbc68c6058e8c9f6a5ffec2c797d84" frameborder=0></iframe>

With this resource, the pilot configures the envoys to authenticate requests before forwarding them to the services: **sa-web-app** and **sa-feedback**. At the same time, this is not applied to the envoys of the service **sa-frontend** enabling us to get the frontend unauthenticated. To apply the Policy, execute the command:

    **$ kubectl apply -f resource-manifests/istio/security/auth-policy.yaml
    **policy.authentication.istio.io “auth-policy” created

Go back to the page and make a request, you will see that it will end in 401 Unauthorized, now let’s forward users from the frontend to authenticate with Auth0.

## Authenticating Requests with Auth0

To authenticate requests of an End User we need to create an API in Auth0 that represents the authenticated services namely: reviews, details, and ratings. To create an API, navigate to **Auth0 Portal** > **APIs** > **Create API**, as seen in the image below.

![Figure 3. Creating a new API in Auth0](https://cdn-images-1.medium.com/max/NaN/1*5HTL6Ji4yVn1mmzFrsoquA.png)*Figure 3. Creating a new API in Auth0*

The important information here is the Identifier later used in the script as:

* **Audience:** {YOUR_AUDIENCE}

And the rest of the needed details are under **Applications** in the Auth0 Portal and then select the **Test Application** created automatically with the same name as the API.

Here note down:

* **Domain:** {YOUR_DOMAIN}

* **Client Id: **{YOUR_CLIENT_ID}

Scroll down in the Test Application to the **Allowed Callback URLs **text field, where we specify the URL where the call should be forwarded after the Authentication is completed, in our case it is:

[http://{EXTERNAL_IP}/callback](http://{EXTERNAL_IP}/callback)

Add for the **Allowed Logout URLs **add the following URL:

[http://{EXTERNAL_IP}/logout](http://{EXTERNAL_IP}/logout)

Let’s move over to the frontend.

## Updating the Frontend

Checkout to the branch** auth0** of the [istio-mastery] repository. In this branch the frontend contains code changes to forward users to Auth0 for authentication and uses the JWT Token in requests to the other services as shown below:

<iframe src="https://medium.com/media/8d320aaf4ad7a2e7326e0f7c4e56040a" frameborder=0></iframe>

To update the frontend to use your tenant’s details navigate to the file sa-frontend/src/services/Auth.js and replace the following values, with the ones we noted down earlier:

<iframe src="https://medium.com/media/cd1a7a4f6ce84fe4665bd6dde09aaa10" frameborder=0></iframe>

The application is ready, specify your docker user id in the command below and then build and deploy the changes:

    **$ docker build -f sa-frontend/Dockerfile \
     -t $DOCKER_USER_ID/sentiment-analysis-frontend:istio-auth0 \
     sa-frontend**

    **$ docker push $DOCKER_USER_ID/sentiment-analysis-frontend:istio-auth0**

    **$ kubectl set image deployment/sa-frontend \
     sa-frontend=$DOCKER_USER_ID/sentiment-analysis-frontend:istio-auth0**

Give the app a try! You will be forwarded to Auth0 where you have to log in (or register) and are forwarded back to the page and can make Authenticated Requests. Meanwhile, if you try the earlier curl commands you will get a 401 Status Code indicating that the request was Unauthorized.

Let’s go one step further and authorize requests.

## Authorization with Auth0

Authentication enables us to know who a user is, but we need the authorization to know what they can access. Istio provides the tools for this as well!

As an example, we’ll create two groups of users (shown in figure 24):

* **Users**: with access to only the SA-WebApp and SA-Frontend service.

* **Moderators**: who can access all three services.

![Figure 4. Authorization Concept](https://cdn-images-1.medium.com/max/NaN/1*ywhs7Ccrmomw-_saf_Ny5Q.png)*Figure 4. Authorization Concept*

To create the user groups, we will use the Auth0 Authorization extension, and then using Istio we will provide them with different levels of access.

## Install and Configure Auth0 Authorization

In the Auth0 portal navigate to Extensions and install the ‘Auth0 Authorization’ extension. After the installation, navigate to the Authorization Extension and configure by clicking your tenant in the top right corner and selecting the menu option “Configuration”. Enable Groups and click the button **Publish rule**.

![Figure 5. Activating Groups in Token Contents](https://cdn-images-1.medium.com/max/NaN/1*Ws7lIabKUyps3tiYri8NIg.png)*Figure 5. Activating Groups in Token Contents*

## Create Groups

In the Authorization Extension navigate to **Groups** and create the Moderators group. Meanwhile, we will treat all Authenticated Users as Regular Users and as such, there is no need to create an additional group.

Select the Moderators group and click Add Members, add your main account. Leave some users without any Group to verify that for them access is forbidden. (You can register new Users manually in Auth0 Portal > Users > Create User)

## Add Group Claim to the Access Token

Users are added to the groups, but this information will not be reflected in the access token. To stay OpenID Connect conformant and at the same time to return the groups we need to [add custom namespaced claims](https://auth0.com/docs/tokens/access-token#add-custom-claims) to the token. This can be done using Auth0 rules.

To create a rule in the Auth0 Portal navigate to Rules, click “Create Rule” and pick an **empty rule** from the templates.

![Figure 6. Creating a new Rule](https://cdn-images-1.medium.com/max/NaN/1*cYQlF2UIETjY85hV9Tixzw.png)*Figure 6. Creating a new Rule*

Paste the code below and save the new rule named as “Add Group Claim”.

<iframe src="https://medium.com/media/e00870e70c8ad4f9a711dd95873926cb" frameborder=0></iframe>

**Note:** This code picks the first group of the user as defined in the Authorization Extension and adds it to the access token as a custom namespaced claim.

Go back to the **Rules page** and verify that you got two roles in this order:

* auth0-authorization-extension

* Add Group Claim

The order matters, because the group field is retrieved asynchronously by the **auth0-authorization-extension** rule and then it is added as a namespaced claim by the second rule, resulting in the following access token:

    {
     "https://sa.io/group": "Moderators",
     "iss": "https://sentiment-analysis.eu.auth0.com/",
     "sub": "google-oauth2|196405271625531691872"
     // [shortened for brevity]
    }

Now we must configure the Envoy proxies to verify user access by extracting the group from the claim https://sa.io/group in the returned access token. That is the topic of the next section, let’s move over there.

## Configuring Authorization in Istio

For authorization to kick in we need to enable RBAC for Istio. To do so apply to the Mesh the following configuration:

<iframe src="https://medium.com/media/2ec6a4d8ef6046429eed07636a71ae2b" frameborder=0></iframe>

1. Enables RBAC only for the services and or namespaces specified in the Inclusion field.

1. Include the list of services specified.

Apply the configuration by executing the command below:

    **$ kubectl apply -f resource-manifests/istio/security/enable-rbac.yaml
    **rbacconfig.rbac.istio.io/default created

Now all services require Role-Based Access Control, in other words, access to all services is denied and will result in the response “RBAC: access denied”. Enabling access to authorized users will be the topic of the next sections.

## Configuring Regular User access

All users should be able to access the **SA-Frontend** and **SA-WebApp **services, this is achieved with these Istio’s resources:

* **ServiceRole:** specifies the permissions that a user has and

* **ServiceRoleBinding:** specifies to whom a ServiceRole applies.

For regular users we will allow access to the specified services:

<iframe src="https://medium.com/media/01acb634dc776b4de29df10033adee66" frameborder=0></iframe>

And using the **regular-user-binding** we will apply the ServiceRole to all visitors of our page:

<iframe src="https://medium.com/media/c782756e173b41710972d1818b0b4bb7" frameborder=0></iframe>

Oh! All users this means that unauthenticated users can SA WebApp? No, the policy will still check the validity of the JWT token. 😉

Apply the configurations:

    **$ kubectl apply -f resource-manifests/istio/security/user-role.yaml
    **servicerole.rbac.istio.io/regular-user created
    servicerolebinding.rbac.istio.io/regular-user-binding created

## Configuring Moderator User access

For our moderators we want to enable access to all the services:

<iframe src="https://medium.com/media/14754faca5e4f1d8dc019104fc71edff" frameborder=0></iframe>

But we want to bind it only to users whose Access Token has the claim https://sa.io/group equal to the value Moderators.

<iframe src="https://medium.com/media/4d8fb9ef312e572270fb3b17d0a5256c" frameborder=0></iframe>

To apply the configurations, execute:

    **$ kubectl apply -f resource-manifests/istio/security/mod-role.yaml
    **servicerole.rbac.istio.io/mod-user created
    servicerolebinding.rbac.istio.io/mod-user-binding created

Due to caching in the envoys it could take a couple of minutes for the Authorization rules to go in effect, but after that, you will be able to verify that Users and Moderators have different levels of access.

## Part 2 — Summary

Seriously have you ever seen any simpler, zero-effort scalable and secure authentication and authorization concept?

Using only three Istio resources (RbacConfig, ServiceRole, and ServiceRoleBinding) we have fine-grained control of authenticating and authorizing end-user access to our services.

Additionally, we offload these concerns from our services to the envoys where we:

* Reduce boilerplate code where security issues and bugs could creep in,

* Reduce asinine cases where one endpoint was exposed by forgetting to annotate it.

* You remove the ripple effect of updating all the services every time you add a new role or permission.

* Keep adding new services simple, secure and fast.

## Conclusion

**Istio** enables your team, once again to focus their resources on providing business value, without the ceremonial overhead of services, finally sending them back to being **micro**.

This article provided you with firm knowledge and hands-on practice to get started with Istio in real-world projects.

I take this opportunity to say thanks for joining me on this voyage, it for sure wasn’t easy and you are amazing for sticking with it. I would love to hear your thoughts in the comments below and feel free to reach out to me on [Twitter](https://twitter.com/rinormaloku) or on my page [rinormaloku.com](https://rinormaloku.com).
