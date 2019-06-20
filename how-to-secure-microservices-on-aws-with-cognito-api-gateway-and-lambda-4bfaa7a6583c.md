
# How to secure Microservices on AWS with Cognito, API Gateway, and Lambda

let me in! (Giphy)

Handling auth is painful. But most applications need to authenticate users and control what resources they can access. [Microservices](https://martinfowler.com/articles/microservices.html), though growing in popularity, can add complexity. You need to secure *both* the user’s actions and the interactions *between* services.

[AWS](https://aws.amazon.com/) offers some great building blocks for a microservices architecture. But like furniture from IKEA, you have to assemble the pieces yourself. Plus the instructions aren’t very good.

We’ll build a simple application and configure AWS to authenticate a user and secure a microservice.

## TL;DR (for the impatient)

**Working Demo: [**https://auth-api-demo.firebaseapp.com/](https://auth-api-demo.firebaseapp.com/) (user: demouser password: demoPASS123)

**GitHub Repo**: [https://github.com/csepulv/auth-api-demo](https://github.com/csepulv/auth-api-demo)

**Base Use Case/Assumption: **There are two groups of resources — **a)** those that need an *authenticated* user and **b)** those that do not.

We’ll use

* AWS [Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html), [API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html), and [Cognito](https://aws.amazon.com/cognito/dev-resources/)

* [Claudia.js](https://claudiajs.com/) (for building our API)

* [React](https://reactjs.org/) (for our web client)

For those who read till the end, there are some goodies.

Now, for the details.

## Conceptual Application Model

The demo application implements the following model.

![](https://cdn-images-1.medium.com/max/2000/1*vq_EQibUjpiCK1C_rF1azg.jpeg)

* A user signs into an application and gets an authentication token

* AWS uses this token to verify identity and to authorize user requests for protected resources

* the App Gateway creates a virtual *moat* between users and application resources

## AWS Services

If you are new to AWS, there is the official [AWS Getting Started](https://aws.amazon.com/getting-started/) portal. Also, Udemy has a free course, [AWS Essentials](https://www.udemy.com/aws-essentials/).

You will need access to an AWS account. You can signup for the AWS [free tier](https://aws.amazon.com/free/).

### AWS Lambda

While [EC2](https://aws.amazon.com/ec2/) is one of the most popular AWS options, I think [Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html) is better suited to microservices. EC2 instances are virtual machines. You are responsible for everything from the operating system to all the software it runs. Lambda is a [Function as a Service](https://martinfowler.com/articles/serverless.html) model. There is no server provisioning or deployment; you write your service logic.

For more info, refer to the [AWS Lambda docs](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html).

But there is a wrinkle with Lambdas. They can’t be directly reached by an application user. Lambdas need triggers that invoke the Lambda function. This can be a queued message, or in our case, an API gateway request.

### AWS API Gateway

An API gateway provides a moat around your application services. It can log user activity, authenticate requests and enforce usage policies (like rate limiting). (The [AWS API Gateway docs](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html) are a good reference.)

### AWS Cognito

[AWS Cognito](https://aws.amazon.com/cognito/dev-resources/) is a user management, authentication, and access control service. Unfortunately, all the features and configuration can be confusing at times. (As if security and authentication were ever easy. 😉 ) We will focus on the core elements of Cognito for securing our API.

## Application and Environment Setup

![App Elements](https://cdn-images-1.medium.com/max/2000/1*b46gRLzVry1kr8ikUOrXMg.jpeg)*App Elements*

The recipe for our demo application is:

1. In AWS Cognito, create a User Pool (with a client application) and a Federated Identity Pool.

1. In AWS API Gateway, create a usage plan and API key

1. Using Claudia JS, build and deploy a simple AWS Lambda-based API.

1. Update AWS IAM role to grant authenticated users access to protected API methods

1. Create a single page app (SPA) using create-react-app. It will use AWS Cognito and makes signed (and authenticated) API requests

The detailed AWS setup is in [aws-setup.md](https://github.com/csepulv/auth-api-demo/blob/master/docs/aws-setup.md), in the demo GitHub repo. We’ll highlight aspects of the setup and explain things work.

### AWS Cognito

**User Pool, Client Application, and Domain Name**

We’ll create a User Pool with the defaults. Details and screenshots:

* [User Pool](https://github.com/csepulv/auth-api-demo/blob/master/docs/aws-setup.md#user-pool)

* [Client Application](https://github.com/csepulv/auth-api-demo/blob/master/docs/aws-setup.md#app-client-settings)

* [Domain Name](https://github.com/csepulv/auth-api-demo/blob/master/docs/aws-setup.md#domain-name)

**Federated Identity Pool**

It may be a little confusing that we need both a User Pool and a Federated Identity Pool. [Ashan Fernando](undefined) has a pretty good explanation in this [post](https://codeburst.io/the-difference-between-aws-cognito-userpools-and-federated-identities-9b47571795d4). Put simply,

* *User Pools* provide access for a user to an application. This is like services such as [Auth0](https://auth0.io).

* A *Federated Identity Pool* provides access to AWS resources.

By combining the two pools, our application can authenticate a user and AWS will assign [temporary credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_request.html). These credentials allow the user to access AWS resources. The IAM role, configured in the Identity Pool, specifies the privileges for the temporary credentials.

The detailed Federated Identity Pool setup is [here](https://github.com/csepulv/auth-api-demo/blob/master/docs/aws-setup.md#federated-identity-pool).

### AWS API Gateway

I suggest creating a usage plan for our API. While not a requirement, it is a good practice, as AWS costs can “run away” if you aren’t careful. We will create a [**Usage Plan](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-api-usage-plans.html)**, named api-auth-demo and set a throttle and burst rate, and a daily quota for API calls. We will also create an API key, which the web client application will use. (Full setup details are [here](https://github.com/csepulv/auth-api-demo/blob/master/docs/aws-setup.md#api-gateway).)

![rate limits and quota](https://cdn-images-1.medium.com/max/5760/1*hyVIHxcwGElJbWp3cEYebw.png)*rate limits and quota*

We’ve finished the bulk of our AWS setup. We will now write our Lambda functions and then build our React web application.

## AWS Lambda and Claudia JS

We will write our Lambda functions using Node.js. [Claudia.js](https://claudiajs.com/) will deploy our Lambdas and configure the API Gateway. (As a note, the [Serverless](https://serverless.com/) framework provides similar functionality.)

We only need a simple API for our example. We’ll create two API methods (i.e. very simple microservices): one for authenticated users and one for guests.

We’ll use the [Claudia API Builder](https://claudiajs.com/claudia-api-builder.html), which lets multiple routes map to a single lambda. The routing mechanism is similar to routing in frameworks such as [Express.js](https://expressjs.com/en/guide/routing.html).

<iframe src="https://medium.com/media/4ae213586578ece9cadc97dd2c770ea1" frameborder=0></iframe>

We’ll use the Claudia.js [command line](https://github.com/claudiajs/claudia/blob/master/docs/create.md) to deploy the API to AWS.

    claudia create --region us-west-2  --api-module api --name auth-api-demo

NOTE: Any changes to api.js will need to be re-deployed. Useclaudia update...

**API Keys and Auth**

In api.js, {apiKeyRequired: true} indicates that API requests require an API key. {authorizationType: 'AWS_IAM'} configures the API Gateway to authorize using [AWS IAM](https://aws.amazon.com/iam/). The underlying authentication mechanism is not obvious. The [AWS docs](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-integrate-with-cognito.html) outline the approach, but a summary is:

* when a user signs in, Cognito will issue tokens for temporary credentials (obtained via [STS](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html)).

* for protected resources, the application needs to sign requests using these credentials

* AWS decodes and verifies the signature

* if the signature is valid, the API Gateway dispatches the request

There are other authorization methods available. The Claudia.js [docs](https://github.com/claudiajs/claudia-api-builder/blob/master/docs/authorization.md) outline how to specify other methods. (The corresponding AWS docs are [here](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-control-access-to-api.html).)

### AWS IAM Roles for Authenticated Users

We need to edit the privileges for the IAM roles for authenticated users. We need to allow invoking the API Gateway method we created.

We need the [ARN](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html) of the API Gateway. Go to the API Gateway console and find the API Gateway resource/method.

![ARN (shown highlighted)](https://cdn-images-1.medium.com/max/4216/1*EaRh0csoB_smLKrY42uIBA.jpeg)*ARN (shown highlighted)*

* Copy the ARN

* Go to the IAM console and find the *Authenticated role* created during the Cognito Federated Identity Pool setup

* add an *Inline Policy* as below

![enter ARN copied from the API Gateway resource (in highlighted area)](https://cdn-images-1.medium.com/max/5348/1*UZPI5CLzl3xdxg6A9IHwLg.jpeg)*enter ARN copied from the API Gateway resource (in highlighted area)*

* Specify the copied ARN for the API Gateway resource in the policy.

Authenticated users can now invoke our protected API methods.

## Service to Service Access Control

The Cognito setup will allow a user to invoke an API method. But this method invocation is a trigger for a Lambda function. The Lambda function executes within the context of a different IAM role. It is no longer a direct user request, but an AWS service to service interaction. IAM roles provide access control for this interaction.

Claudia.JS created the IAM role for the Lambda function. (You can also manually create this role and specify its identifier to Claudia.JS via the --role parameter. Details are [here](https://github.com/claudiajs/claudia/blob/master/docs/create.md).)

If our Lambda function needs access to other AWS resources, we will need to update the Lambda’s IAM role and provide these privileges. This might be an [RDS](https://aws.amazon.com/rds/) database, for example.

AWS has always used IAM to configure service to service access control. It is a well developed and [well-documented](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) model. It will probably be your primary mechanism for access control between microservices (within AWS). There might be cases where you need to augment or replace it, but I would start with IAM.

We can now build the web application for our users.

## React Web Application

I am going to build a [React](https://reactjs.org/) single page web application (SPA). A [Vue.js](https://vuejs.org/) or [Angular](https://angularjs.org/) application would work too. For the client application, there are two significant components: [AWS Amplify](https://aws-amplify.github.io/amplify-js/index.html) and the [aws4](https://www.npmjs.com/package/aws4) module.

AWS Amplify provides easy integration with AWS Cognito. aws4 is a popular library for signing AWS requests using [AWS Request Signatures Version 4](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html). AWS used signed requests for protected resources (i.e. authorized user requests).

Returning to the web client, we’ll use [create-react-app](https://github.com/facebook/create-react-app). I won't outline the steps, as they are well documented on the create-react-app home page, and there are numerous online tutorials. (I've even written a [few](https://medium.freecodecamp.org/how-to-build-animated-microinteractions-in-react-aab1cb9fe7c8). )

For authentication, we need to do some state management. The example application doesn’t use any framework, but in a real application I’d suggest [Mobx](https://mobx.js.org/) (or [Redux](https://redux.js.org/).)

In the demo application, auth-store.js manages the user authentication state. This consists of the user's authentication state and credentials. These are used to

* render different components and styles for authenticated vs. guest user

* sign requests for protected API methods

While AWS Amplify manages much of the AWS Cognito integration, there is some work for us to do.

**Determining Auth State from AWS Amplify**

AWS Amplify’s documentation is good in some areas and deficient in others. I suggest reading the [Authentication section](https://aws-amplify.github.io/amplify-js/media/authentication_guide) of the Amplify documentation. This describes theAuth component, which interacts with Cognito.

However, there are still some aspects that the documentation doesn’t clearly address. AWS Amplify doesn’t make it easy to know the authentication state. (A discussion of this complexity is [here](https://github.com/aws-amplify/amplify-js/issues/159#issuecomment-374028468).) Amplify configures itself asynchronously, without a callback. But there is an aws-amplify class that can help.

The [Hub](https://github.com/aws-amplify/amplify-js/blob/master/docs/media/hub_guide.md) class in the aws-amplify module behaves like an event emitter. We care about two events: configured and cognitoHostedUI.

![page load / configure sequence](https://cdn-images-1.medium.com/max/2000/1*4tvCgp1hgYXycSJleun6rw.jpeg)*page load / configure sequence*

After the AWS Amplify configures the Auth component, it emits the configured event. Our application can then inquire about the current user's authentication status. This is useful when our application is being loaded, for example.

![login / authenticated state change sequence](https://cdn-images-1.medium.com/max/2000/1*vWN0pVQVoa0qY-jg3HottA.jpeg)*login / authenticated state change sequence*

While using the application, we need to know if the authentication state changes. There is a sign-in event, but it isn't the event we want, as our demo application uses [OAuth and the Cognito Hosted UI](https://aws-amplify.github.io/amplify-js/media/authentication_guide#using-amazon-cognito-hosted-ui). The sign-in event is used in a custom sign-in/up screen or when using the built-in Amplify React UI. For OAuth, Amplify dispatches the cognitoHostedUI event after a completed OAuth sign-in flow.

**Signing Requests**

The current user will have credentials issued by AWS Cognito. These contain an *access id*, a *secret key*, and a *session key*. These are available by calling Auth.currentCredentials() in aws-amplify. For API methods authorized by IAM, you need to *sign* the request using [AWS V4 Request Signatures](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html). Thankfully, the [aws4](https://www.npmjs.com/package/aws4) module handles the complexities of generating these signatures.

In [api-client.js](https://github.com/csepulv/auth-api-demo/blob/master/web-ui/src/api-client.js),

<iframe src="https://medium.com/media/62522cfd5301fd9495641f2a9793f7aa" frameborder=0></iframe>

### Demo

We can finally run npm start and run the app! When we first arrive at the application, we are a guest (unauthenticated user). You can also go to [https://auth-api-demo.firebaseapp.com/](https://auth-api-demo.firebaseapp.com/) to try it out.

![](https://cdn-images-1.medium.com/max/2856/1*5jsOZ6Qsn6xFn9eqZixKDA.png)

We can access unprotected methods.

![auth is not required](https://cdn-images-1.medium.com/max/2856/1*fCm0PGScMXKBMtNzea7QOw.png)*auth is not required*

But if we try to access a protected resource, it will fail.

![not authenticated](https://cdn-images-1.medium.com/max/2856/1*UEcp7iF8kWvrE8dnjpZ0WQ.png)*not authenticated*

But if we sign in, we can access the protected resources.

Click **Sign In** and use demouser with password of demoPASS123.

![after sign in — buttons reflect an authenticated state](https://cdn-images-1.medium.com/max/2856/1*ot3j4-o8OaBiaagTy9hMfg.png)*after sign in — buttons reflect an authenticated state*

We can now click the Req. Auth button to access a protected API method.

![](https://cdn-images-1.medium.com/max/2856/1*kANusN4eyPy0PBufRlD51A.png)

Whew! We had to configure multiple services and digest a lot of information. But we now have an application that is a model for authenticating microservices on AWS.

![[Giphy](https://giphy.com/gifs/reese-witherspoon-y4OKEc5NuPDwY)](https://cdn-images-1.medium.com/max/2000/1*uvDysqC-6VLSBHIEmjrmmw.gif)*[Giphy](https://giphy.com/gifs/reese-witherspoon-y4OKEc5NuPDwY)*

## Now What?

This article’s approach is “all-in” on AWS. This was a deliberate choice, to show how the various AWS pieces fit together to solve a common need, namely auth. There are alternatives to methods in this article, and I outline a few [here](https://github.com/csepulv/auth-api-demo/blob/master/docs/auth-alternatives.md).

And for those who stayed with me to the end, I have some parting gifts.

* In the [demo repo](https://github.com/csepulv/auth-api-demo), there is a [script](https://github.com/csepulv/auth-api-demo/blob/master/scripts/create-resources.js) for automating the AWS setup. Its [README](https://github.com/csepulv/auth-api-demo/blob/master/scripts/README.md) has the details for running it.

* [resources-cheatsheet.md](https://github.com/csepulv/auth-api-demo/blob/master/docs/resource-cheatsheet.md) has the specific links for relevant AWS, Claudia.js, etc. documentation.

Thanks for reading! (A few 👏 are always appreciated. 😀)
