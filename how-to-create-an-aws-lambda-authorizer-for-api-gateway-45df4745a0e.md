Unknown markup type 10 { type: [33m10[39m, start: [33m44[39m, end: [33m51[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m9[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m72[39m, end: [33m90[39m }

# How to create an AWS Lambda Authorizer for API Gateway

λ Authorizer and API Gateway flow

I’ve recently been working on managing authorization for authenticated users making calls to our API backend. This authorization process comes after the federated login UI consisting of Google Auth and AWS Cognito. Luckily, API Gateway is built for this and works perfectly with an AWS Lambda authorizer which handles how information is passed from Amazon API Gateway to other λ functions or backend services. Before we begin I’ll like to talk about some basic terminologies.

### What is an AWS API Gateway Lambda authorizer?

An AWS API Gateway Lambda authorizer(formerly know as ***custom authorizer***) is a Lambda function that you provide control access to your API methods. It uses bearer token authentication strategies such as OAuth, SAML or AWS Cognito. So when a client calls your API, API Gateway verifies whether a lambda authorizer is configured for the API method. If so, the Lambda function is called.

This call to API Gateway supplies the authorization token that is extracted from the request header or passes the incoming request parameters to the parameter-based authorizer function which checks to see if the token is valid and if this user is authorized to call this function. The λ authorizer generates a Policy and principal ID which is returned to API Gateway. The API Gateway then evaluates this policy and returns if allowed makes calls to the backend service(s) and returns a response to the user.

![serverless web application architecture](https://cdn-images-1.medium.com/max/2500/1*EpJzpxyCu8L7NAtNYVqlaA.png)*serverless web application architecture*

The diagram above shows a basic serverless web application architecture using API Gateway and the λ authorizer function to authorize requests to the backend services. Also note the aim of this tutorial is to provide basic steps on AWS lambda authorizers and connecting with other AWS services.

Let’s get to it then, this tutorial is made up of three main sections:

* Authenticate a user with AWS Cognito, other options include Google Auth, SAML etc.

* Create a basic λ function, link to an API on API Gateway.

* Create and wire an AWS Lambda authorizer function to API Gateway

* Lastly, wire up the entire application from the Cognito user pool UI, API Gateway, λ authorizer function, and the serverless backend service.

## **Step 0: Pre-requisite configs**

We will be using the Python runtime and the aws-cli even-though these concepts could easily be applied to other languages like Go, Node, etc.

All the code snippets for this tutorial can be found here on [GitHub](https://github.com/Ch3ck/api-gateway-authorizer-lambda)
[**Ch3ck/api-gateway-authorizer-lambda**
*Secure your APIs with AWS Lambda authorizer functions *github.com](https://github.com/Ch3ck/api-gateway-authorizer-lambda)

Setup the config.ini file with the following variables

    # AWS  λ configs
    [aws]
    AWS_DEFAULT_REGION:
    AWS_S3_BUCKET_NAME:

    [cognito]
    USERPOOL_ID:
    METHOD_ARN:
    APP_CLIENT_ID:
    ID_TOKEN:

## Step 1: Setup the Test Lambda Endpoint

This will be the lambda endpoint to be called after setting up API gateway and the Authorizer lambda. There are two ways to set this up: either from the terminal or from the AWS console.

![Create a new lambda function from the AWS Console](https://cdn-images-1.medium.com/max/6156/1*SlNKxXULHy_wC5mhGi-epw.png)*Create a new lambda function from the AWS Console*

Create a role for the current Lambda function to execute properly.

Next step will to add the source code the current endpoint. The **source code** can be found [**here](https://github.com/Ch3ck/api-gateway-authorizer-lambda/blob/master/step1/basic_lambda_endpoint.py)**

![basic Lambda endpoint code.](https://cdn-images-1.medium.com/max/6208/1*QVkPk8HuDAZD6i87b4gHhg.png)*basic Lambda endpoint code.*

Once the code is added click ‘save’ and add a testEvent as showed below

![configure test event with basic input variables](https://cdn-images-1.medium.com/max/3492/1*eO2WviFDLdrotJpyJQ5koQ.png)*configure test event with basic input variables*

Now you can test run the Lambda endpoint to make sure it works as expected.

![successful Lambda test run](https://cdn-images-1.medium.com/max/6200/1*QyuoQyPwSgv1HU1Rn_jEoA.png)*successful Lambda test run*

## Step 2: Create the Web API with API Gateway

Navigate to API Gateway on the AWS console and select the **new API** button to create your own API

![create basicAPI](https://cdn-images-1.medium.com/max/6628/1*vcYuWh14YvG1yy6asGB2hg.png)*create basicAPI*

Enter **basicAPI **as name and press **Create API. **On the next page, select **Actions** and create method** GET**

![creating new method GET](https://cdn-images-1.medium.com/max/6652/1*VeE6DfddWJWSHTEQpSf5eg.png)*creating new method GET*

On the next page, press “Actions” → “Create Method”. Select “GET”.

As soon as you click** Save **integrate the previously created lambda function basicLambda. This will trigger a box to inform you that API Gateway will receive permission to execute the** basicLambda** lambda function.

Now the API is configured and all is required next is to select** Actions** and click **Deploy API**

Next you’ll see the **invoke URL** for the newly created web service. Copy this URL and save for future use as shown below:

![Invoke URL for newly created service](https://cdn-images-1.medium.com/max/5316/1*ivtxrjbLd0w9wsXk91xRXA.png)*Invoke URL for newly created service*

## Step 3: Create an AWS Lambda Authorizer for this API

Now we need to configure an AWS Authorizer for this API. From the API Gateway console, select the authorizer menu and click Create Authorizer Lambda as shown:

![create new authorizer](https://cdn-images-1.medium.com/max/4600/1*O9EOsqZnHWM_rj87WA4x1w.png)*create new authorizer*

This consists of a lambda function registered with API gateway to secure access to the API. It expects a specific JSON input from API Gateway as shown:

    {
      "type": "TOKEN",
      "authorizationToken": "",
      "methodArn": ""
      
    }

Before we proceed with creating the new Authorizer we have to create another Lambda function from the AWS console. See the full source code [**here](https://github.com/Ch3ck/api-gateway-authorizer-lambda/blob/master/step3/authorizer-lambda.py)**.

Once the Lambda function is created and deployed on AWS together with a jwt-token verifier we can proceed with add the authorizer to our API.

![adding basicLambda Authorizer](https://cdn-images-1.medium.com/max/2920/1*FkQWYuG2kabiA0vz0P050Q.png)*adding basicLambda Authorizer*

Now the next step will be to test our **basicLambdaAuthorizer** Lambda to make sure it works well with our expected input.

First I need to create a test event to test our endpoint with the **methodArn **matching our API GET method.

Now I test the basicAPIAuthorizer Lambda and get the following output:

Also testing from the API Gateway menu fails without the **authorizationToken** as expected

![api access not authorized without authorizationToken](https://cdn-images-1.medium.com/max/2688/1*-aYgA2KHKgezG5RXc530DQ.png)*api access not authorized without authorizationToken*

Another option is to try the **invokeURL** on the browser

![api access Unauthorized](https://cdn-images-1.medium.com/max/2244/1*vHk9Ak7ZZdERfeddVffA7A.png)*api access Unauthorized*

Now our API is secure behind API Gateway and can only be accessed with the authorizationToken.

## Step 4: Wiring it all together

Now have successfully secured our API behind API Gateway and will require an Authorization header from the client to respond. However we need to enable CORS(Cross-Origin Resource sharing) for the API.

From the API Gateway console, select the basicAPI and click on** Resources**, select “**Enable CORS**” from the drop down and enter **Authorization** as **Access-Control-Allow-Header** and **Access-Control-Allow-Origin.**

### **Note:**

For **production environments** configure the **Access-Control-Allow-Origin** variable to match the URL you wish to deploy the endpoint to; I used the **‘*’** value just for testing purposes.

![enabling CORS](https://cdn-images-1.medium.com/max/6680/1*_exFTRgk0Mv87pfGZWYxJQ.png)*enabling CORS*

Click and replace existing CORS headers

![CORS enabled for API](https://cdn-images-1.medium.com/max/5908/1*d-56IxNVjRfs_DdkebjGIw.png)*CORS enabled for API*

Save and Redeploy API

Now you can reload and sign-in from your UI either with Google Auth or AWS Cognito and API gateway will use the user ID to authenticate and authorize API access for the current user.

**Conclusion**

Securing your endpoints is critical especially when you want to control costs, handle incorrect data and exposing a private beta. Using API Gateway to secure these endpoints via lambda authorizers helps manage what/who is allowed to make a certain request. Also, the Authorizer makes it easy to eliminate the responsibility of each Lambda REST service to perform these kinds of validation enabling cleaner code; since each lambda function focuses exclusively on the application/business logic. These notes are my methods for setting up secure APIs on the AWS infrastructure. It will help me and/or someone else in the future. I’ve not explored how this might work on GCP or Azure.

Hope you liked it and let me know if you have any questions or comments.

Clap for yourself!
