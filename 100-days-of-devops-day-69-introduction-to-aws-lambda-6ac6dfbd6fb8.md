
# 100 Days of DevOps — Day 69-Introduction to AWS Lambda

Welcome to Day 69 of 100 Days of DevOps, Focus for today is Introduction to AWS Lambda
> *What is AWS Lambda?*

*With AWS Lambda, you can run code without provisioning or managing servers. You pay only for the compute time that you consume — there’s no charge when your code isn’t running. You can run code for virtually any type of application or backend service — all with zero administration. Just upload your code and Lambda takes care of everything required to run and scale your code with high availability. You can set up your code to automatically trigger from other AWS services or call it directly from any web or mobile app.*

* *To start with Lambda*

*Go to [https://us-west-2.console.aws.amazon.com/lambda](https://us-west-2.console.aws.amazon.com/lambda) → Create a function*

![](https://cdn-images-1.medium.com/max/4836/1*0XfnTQ0OoUOBSnvDx4IKDw.png)

![](https://cdn-images-1.medium.com/max/5488/1*wNXRcGrb2ZDHgJSR2yqNjg.png)

* *Create function you have three options*

    ** **Author from scratch:** Which is self explanatory, i.e you are writing your own function
    * **Use a blueprint:** Build a lambda application from sample code and configuration preset for common use cases(Provided by AWS)
    * **Browse serverless app repository: **Deploy a sample lambda application from the AWS Serverless Application Repository(Published by other developers and AWS Patners)*

* *Function name: HelloWorld*

* *Runtime: Choose Python3.7 from the dropdown*

* *Permission: For the time being choose the default permission*

* *Click Create Function*

![](https://cdn-images-1.medium.com/max/5680/1*d3f8QAZEuD2FIkR9TZTneA.png)

![](https://cdn-images-1.medium.com/max/5668/1*2JStoX9rTVZXYKkIJgd9Lw.png)

![](https://cdn-images-1.medium.com/max/5532/1*yfHjrZvjIKTUjtDdkYd96w.png)
> *Invoking Lambda Function*

* *When building applications on AWS Lambda the core components are Lambda functions and event sources. An event source is the AWS service or custom application that publishes events, and a Lambda function is the custom code that processes the events*

    ** Amazon S3 Pushes Events
    * AWS Lambda Pulls Events from a Kinesis Stream
    * HTTP API requests through API Gateway
    * CloudWatch Schedule Events *

![](https://cdn-images-1.medium.com/max/2000/1*YCxOtCyCbsLorLq3l8st-w.png)

![](https://cdn-images-1.medium.com/max/2000/1*-rOiNiox31Hvhl05cLSMoQ.png)

* *From the list select CloudWatch Events*

![](https://cdn-images-1.medium.com/max/5140/1*nK8h1KzY2_XVQIarQXq-MQ.png)

* *As you can see under CloudWatch Events it says configuration required*

![](https://cdn-images-1.medium.com/max/5228/1*DeBtzgwWRjRrtejeYyfzgg.png)

* *Rule: Create a new rule*

* *Rule name: Everyday*

* *Rule description: Give your Rule some description*

* *Rule type: Choose Schedule expression and under its rate(1 day)(i.e its going to trigger it every day)*
[**Schedule Expressions Using Rate or Cron - AWS Lambda**
*AWS Lambda supports standard rate and cron expressions for frequencies of up to once per minute. CloudWatch Events rate…*docs.aws.amazon.com](https://docs.aws.amazon.com/lambda/latest/dg/tutorial-scheduled-events-schedule-expressions.html)

* *Click on Add and Save*

* *Now go back to your Lambda Code(HelloWorld)*

    *import json*

    *def lambda_handler(event, context):
        # TODO implement
        print(event) <--------
        return {
            'statusCode': 200,
            'body': json.dumps('Hello from Lambda!')
        }*

* *Add this entry, which simply means we are trying to print the event*

* *Again save it*

* *Let’s try to set a simple test event, Click on Test*

![](https://cdn-images-1.medium.com/max/5332/1*HbHMrj7ZdKUggRpSC-hG7g.png)

* *Under Event template, search for Amazon CloudWatch*

![](https://cdn-images-1.medium.com/max/2568/1*-kRNX6a4gUxH-0lpYl3S1Q.png)

* *Event Name: Give your event some name and test it*

![](https://cdn-images-1.medium.com/max/5244/1*-sqAzFQcFV4oX6i6Wb1TuQ.png)

* *Go back and this time Click on Monitoring*

![](https://cdn-images-1.medium.com/max/5184/1*dhqPmjoqGdO8Uub1IbkwiA.png)

* *Click on View logs in CloudWatch*

![](https://cdn-images-1.medium.com/max/5236/1*Y40EBULRvrwkxInMBlzhoA.png)

* *Click on the log stream and you will see the same logs you see in Lambda console*

![](https://cdn-images-1.medium.com/max/5092/1*-0kjFQo2hcMjaDWFjZREvg.png)
> *Lambda Programming Model*

* *Lambda supports a bunch of programming languages*

![](https://cdn-images-1.medium.com/max/2000/1*m3oILf06dgbtGEZRcZKOOQ.png)

* *You write code for your Lambda function in one of the languages AWS Lambda supports. Regardless of the language you choose, there is a common pattern to writing code for a Lambda function that includes the following core concepts.*

    ** **Handler:** Handler is the function AWS Lambda calls to start execution of your Lambda function, it act as an entry point.*

![](https://cdn-images-1.medium.com/max/4988/1*6nDVPbI-6zJCmm1kUGcKrQ.png)

* *As you can Handle start with lambda_function which is a Python Script Name and then lambda_handler which is a function and act as an entry point for event and context*

* ***Events:** We already saw in the previous example where we passed the CloudWatch Event to our code*

* ***Context** — AWS Lambda also passes a context object to the handler function, as the second parameter. Via this context object, your code can interact with AWS Lambda. For example, your code can find the execution time remaining before AWS Lambda terminates your Lambda function.*

* ***Logging** — Your Lambda function can contain logging statements. AWS Lambda writes these logs to CloudWatch Logs.*

* ***Exceptions** — Your Lambda function needs to communicate the result of the function execution to AWS Lambda. Depending on the language you author your Lambda function code, there are different ways to end a request successfully or to notify AWS Lambda an error occurred during the execution.*

* *One more thing, I want to highlight is the timeout*

![](https://cdn-images-1.medium.com/max/2512/1*RVDSzp0vApuH1v_vbtZwFA.png)

* *You can now set the timeout value for a function to any value up to 15 minutes. When the specified timeout is reached, AWS Lambda terminates execution of your Lambda function. As a best practice, you should set the timeout value based on your expected execution time to prevent your function from running longer than intended.*

*Looking forward from you guys to join this journey and spend a minimum an hour every day for the next 100 days on DevOps work and post your progress using any of the below medium.*

* *Twitter: [@100daysofdevops](http://twitter.com/100daysofdevops) OR @[lakhera2015](https://twitter.com/lakhera2015)*

* *Facebook: [https://www.facebook.com/groups/795382630808645/](https://www.facebook.com/groups/795382630808645/)*

* *Medium: [https://medium.com/@devopslearning](https://medium.com/@devopslearning)*

* *Slack: [https://devops-myworld.slack.com/messages/CF41EFG49/](https://devops-myworld.slack.com/messages/CF41EFG49/)*

* *GitHub Link:[https://github.com/100daysofdevops](https://github.com/100daysofdevops)*

***Reference***
[**100 Days of DevOps — Day 0**
*D-day is just one day away and finally, this is a continuation of the post(I posted a month earlier)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-0-4f2c9750542d)
[**100 Days of DevOps**
*Motivation*medium.com](https://medium.com/@devopslearning/100-days-of-devops-81faf13bf772)
