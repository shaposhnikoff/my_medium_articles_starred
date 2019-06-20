
# The Right Way™ to do Serverless in Python

The Right Way™ to do Serverless in Python

As you’ve undoubtedly heard, serverless has grown in leaps and bounds since the release of AWS Lambda in November 2013. And while the idea of not having to manage a server, paying only for the compute you actually use and horizontal scaling out-of-the box may sound really appealing, where do you start? As a Python developer, maybe you’ve heard of the [Serverless Framework](https://serverless.com/) or [Zappa](https://www.zappa.io/); and either the prospect of working with an unfamiliar ecosystem like Node.js (in Serverless’ case) is daunting, or maybe you’re unclear on what these tools actually do and if they’re overkill when you’re just getting started. If that’s the case, you’ve come to the right place. In this blog post we’ll take a brief tour of serverless as it applies to Python as well as some handy tips and tricks to help you get your feet wet.

## Is my app/workload right for serverless?

This is probably the most important question you need to ask about serverless. While serverless has a lot to offer, it isn’t ideal for all apps/workloads. For example, the current maximum duration for an AWS Lambda function invocation is five minutes. What this means is that if your app/workload can’t be broken up into five minute chunks, then serverless might not be the best option. If your app uses websockets, for example, which maintains a persistent connection with the server, on AWS Lambda this connection would be closed and need to be reestablished every five minutes. It would also mean you’re paying for all that compute time for a workload that is really just keeping a socket open. But if your app is mostly stateless, like a REST or GraphQL API, then serverless may be a great fit, as HTTP requests rarely go beyond 30 seconds, let alone five minutes.

Where serverless really shines is workloads where you might have spikes and/or periods of low activity. If you think about it, that description likely applies to most apps/workloads you’ve worked on. When your app/workload spikes, then AWS Lambda is there to provide considerable horizontal scaling, by default handling 1,000 concurrent requests out-of-the-box (this limit can be increased). And when your app/workload is in a low activity period, your meter isn’t running at full tilt, which can save you a lot on operating expenses. Think about it; most apps/workloads serve a range of timezones, so why should you pay full price to run your app/workload when your customers are asleep?

If you’re still not sure whether or not your app/workload is a good fit, here’s a [handy calculator](https://servers.lol/) to compare AWS Lambda to EC2.

## Python 2 or 3?

The Python ecosystem has gone through a lot of changes the past decade — the most significant being the release of Python 3 and the transition of many codebases from Python 2.x to 3.x. In the serverless world, the advice here is consistent with any other Python project: use Python 3.6 if it’s a new project. While Python 2.7 has served many of us well, it will [cease to receive updates](https://pythonclock.org/) after 2020. So if you haven’t already started your transition to 3.6, you have two years to do so.

If you have an existing Python 2.7 project, you’re in luck, as AWS Lambda supports 2.7, but you should seriously consider starting to port your code to Python 3.6 soon. Here’s a super helpful [cheatsheet](http://python-future.org/compatible_idioms.html) on how to write Python2+3 compatible code. And like the cheat sheet, the advice here will be 2.7/3.6 compatible.

## Web API or Worker?

Before we go over the serverless tools available to Python, lets drill down a little more into our app/workload. If you have a web app that serves a front-end with several web assets (HTML, JavaScript, CSS, images, etc.), then you should not be serving these up via your AWS Lambda function. That’s *not to say that you can’t, just that you shouldn’t*. Remember that with AWS Lambda you pay for the time your function is running. It doesn’t make much sense to spend this time serving up web assets. In fact, since your front-end likely has many web assets, this could be a relatively expensive way of doing what is otherwise a simple task. For serving up web assets, you should look at a CDN (Content Delivery Network). AWS has a service specifically built for this purpose called CloudFront, here’s [a guide](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/DownloadDistS3AndCustomOrigins.html) on how to use it with S3.

But that really only covers your web app’s front-end. Or maybe your app/workload doesn’t have a front-end at all. We’re going to break these workloads into two categories: web APIs (REST, GraphQL, etc.), and workers. You’re likely already familiar with the web API variety, and AWS Lambda allows us to serve them using a service called API Gateway (more on that later). The other, what I’m going to call “workers,” is all the other tasks your app might need to do, such as: sending email, uploading files, push notifications, etc. Serverless can handle this workload, and even offers it’s own cron job like service to handle scheduled tasks.

The reason I’m defining these two categories is that while in serverless these workloads may look similar (or indistinguishable, as they’re both essentially functions), some tools are better at or designed for one over the other. In this guide, I’ll point out where this is the case. Hopefully you’re already thinking about what parts of your app will be served via a web API and what parts can be worker tasks run in the background so you can pick the right tool for the job.

## Serverless Framework

The big kahuna in the serverless space is undoubtedly the [Serverless Framework](https://serverless.com/), and for good reason. The team behind it has poured considerable time and effort into developer experience to make it one of the most intuitive and accessible serverless tools out there. It also offers a feature set that is second to none, supporting multiple clouds in addition to AWS Lambda and a growing plugin ecosystem. For a Python developer, this would seem like the obvious starting point and it certainly is a great one.

But serverless framework is not without some Python-specific caveats. For starters, Serverless is written in Node.js, which may not be your cup of tea. But like any good tool, if done right, you shouldn’t even notice what language it is implemented in, and the case could certainly be made here for Serverless. That being said, you’ll still need to [install Node and NPM](https://nodejs.org/en/download/), but after that you’re only a npm install -g serverless away from having the Serverless CLI be your primary interface. You won't need to know any JavaScript, although your configs will be YAML by default.

Let’s give it a try.

    npm install -g serverless

The CLI can be accessed using either serverless or the shorthand sls. I prefer sls, so lets create a project:

    mkdir ~/my-serverless-project
    cd ~/my-serverless-project
    sls create -n my-serverless-project -t aws-python3

Here I’ve created a directory called my-serverless-project and created a project using sls create. I'm also specifying a template with -t aws-python3. Serverless comes bundled with several templates that will set some sensible defaults for you in your serverless.yml that will be created when you run sls create. In this case, I'm specifying the AWS template for Python 3.6. There's also a aws-python2 if you're project is Python 2.7. There are other templates for other languages and clouds, but that's outside of the scope of this guide. The -n my-serverless-project specifies a service name, you can change this to whatever you want to name your project. Once you run these commands you should have a serverless.yml in your my-serverless-project directory. Let's take a look at the contents:

    cat serverless.yml

As you will see the serverless.yml is YAML and comes loaded with several helpful comments explaining each section of the config. At this point I recommend reading through these comments as it will be helpful later on. Once done, let's write a hello world equivalent of a lambda function:

    def handler(event, context):
        return {"message": "hi there"}

And save that in our my-serverless-project directory as hello.py. The name handler is just what lambda functions are commonly referred, you can name your functions whatever you like. Now that we have a function, let's make Serverless aware of it by adding it to our serverless.yml. First open up your serverless.yml and look for the functions section. Then replace that section with the following:

    functions:
      hello:
        handler: hello.handler

Now save your serverless.yml. To deploy this function, we'll need to make sure we have our [AWS credentials configured](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html). Assuming we're all good there, then just run the following:

    sls deploy

Deployment will take a little while. In short, here’s the steps that Serverless is doing:

1. Creating a CloudFormation template based on the serverless.yml

1. Compressing the CloudFormation template and your hello.py into a zip archive

1. Creating a S3 bucket and uploading the zip archive to it

1. Executing the CloudFormation template, which includes configuring an AWS Lambda function and pointing it to the S3 zip archive

You could do all of these steps manually, but why would you want to? When our deploy is complete, we can test it with the following command:

    sls invoke -f hello

Here -f hello is the name of the function we specified in our serverless.yml file. The sls invoke command invokes the function. A function invocation is a fancy word for calling the function. When we invoke this function, it should respond with:

    {"message": "hi there"}

If that’s what you see, congratulations, you’ve just created your first AWS Lambda function. But this isn’t terribly useful.? What if we wanted to do something a little more challenging, like making a HTTP request and returning the result. Let’s create a new file called httprequest.pyand add the following:

    import requests

    def handler(event, context):
        r = requests.get("https://news.ycombinator.com/news")
        return {"content": r.text}

Update the functions section of your serverless.yml:

    functions:
      hello:
        handler: hello.handler
      httprequest:
        handler: httprequest.handler

And re-deploy and invoke our new function:

    sls deploy
    sls invoke -f httprequest

You should now see an ImportError. This is because requests is not installed. With AWS Lambda, we need to bundle any libraries we want to use with our function. We can do this a number of ways, with pip we can run:

    # Don't run this just yet, keep reading
    pip install requests -t .

This will install the requests wheel (and it's dependencies) in our my-serverless-projectdirectory. But there's a catch. AWS Lambda runs on 64-bit Linux. And because requestscomes with dependencies that are compiled, if you're not running 64-bit Linux as well this copy of requests you just installed won't run on AWS Lambda. So what do you do if you're running Mac OS? Or Windows? Or FreeBSD? Or, well, you get the picture.

Serverless plugins (and Docker) to the rescue.

Thankfully Serverless comes with a plugin ecosystem to fill in the gaps, and for Python, packages with compiled code just happens to be one of those gaps. The relevant plugin here is aptly named [serverless-python-requirements](https://github.com/UnitedIncome/serverless-python-requirements), and to install it:

    sls plugin install -n serverless-python-requirements

And add the following lines to the end of your serverless.yml:

    plugins:
      - serverless-python-requirements

Now right away this plugin enables requirements.txt support. So instead of the pip line above, we now just need to add a requirements.txt file to our project directory and the next time we run sls deploy the requirements will be installed and bundled automatically. Let's do that now:

    echo "requests" >> requirements.txt

But we haven’t solved our compilation problem yet. To do that we’ll need to add a customsection to our serverless.yml. This section is where we can put our custom configuration options, but it's also where plugins look to for their own config options. Our new custom section should look like this:

    custom:
      pythonRequirements:
        dockerizePip: true

What this does is tell the serverless-python-requirements plugin to compile the Python packages in a Docker container before bundling them in the zip archive to ensure they're compiled for 64-bit Linux. You'll also need to [install Docker](https://docs.docker.com/install/) in order for this to work, but once done this plugin will handle the dependencies you define in your requirements.txt file automatically. Pretty cool, huh?

Now let’s re-deploy and re-invoke our function:

    sls deploy
    sls invoke -f httprequest

Even if you are running 64-bit Linux, this is way cleaner, don’t you think?

Now before we continue, you likely have a few questions. What are event, context and how is any of this useful to me?

AWS Lambda functions are event-driven, so when you invoke a function, you’re actually triggering an event within AWS Lambda. The first argument of your lambda function contains the event that triggered the function, which is represented within AWS Lambda as a JSON object, but what is passed to Python is a dict of that object. When you run sls invoke -f hello, what is passed to the lambda function is an empty dict. But if it was an API request, it would contain the entire HTTP request represented as a dict. In other words, the event dict acts as your lambda function's input parameters and what your lambda function returns is the output. With AWS Lambda, your output needs to be JSON serializable as well. Here's [some example events](https://docs.aws.amazon.com/lambda/latest/dg/eventsources.html)you might see with AWS Lambda.

The second argument is the AWS Lambda context, which is a Python object with useful meta data about the lambda function and the current invocation. For example, every invocation has a aws_request_id which is useful if you want to track down what happened in a specific invocation within your logs. Check [here](https://docs.aws.amazon.com/lambda/latest/dg/python-context-object.html) for more information about the context object. You probably won't need to worry about the context object right away, but you'll eventually find it useful when debugging.

So how is this useful? Well, if your app/workload can work with a JSON serializable input and produce a JSON serializable output, then it can plug right into a AWS Lambda. If we go back to our two workload categories, web API and worker, we’ve already pretty much implemented what you’ll need for a worker. Lets say we wanted to run our httprequest function every ten minutes, in order to do that all we would need to do is add the following to our serverless.yml:

    functions:
      httprequest:
        handler: httprequest.handler
        events:
          - schedule: rate(10 minutes)

And re-deploy:

    sls deploy

And we’re done. Now httprequest is being triggered automatically every ten minutes. If you want more fine-grained control, you can [specify a specific time](https://serverless.com/framework/docs/providers/aws/events/schedule/) your function should be triggered as well. You can also build more complex workflows using SNS, SQS or other AWS services.

What about the web API? Remember earlier we covered events and mentioned that an HTTP request can be represented as an event? In the case of web APIs, it is Amazon’s API Gateway service that will be triggering the events for us. In addition to this, API Gateway will provide a hostname that can receive HTTP requests, transform those HTTP requests into an event object, invoke our lambda function, collect the response and pass it on to the requester as an HTTP response. That might sound complex, but thankfully the Serverless framework abstracts much of this for us. Let’s add an HTTP endpoint to our serverless.yml:

    functions:
      webapi:
        handler: webapi.handler
        events:
          - http:
              path: /
              method: get

This looks a lot like our scheduled worker task earlier, doesn’t it? Like in the scheduled task, here we configure this handler to handle http events and specify a path and method. As you've likely guessed, the path is our HTTP request path and the method is the HTTP method this handler will handle. You'll also notice we've added a new handler, let's create that now in webapi.py:

    import json

    def handler(event, context):
        return {"statusCode": 200, "body": json.dumps({"message": "I'm an HTTP response"})}

Here our handler will accept an event from the API Gateway and will respond with a JSON serializable dict. Within the dict we have two keys, the first is statusCode which is the HTTP status code we want the API Gateway to respond with and the second is the HTTP body of the response. You'll also notice that we serialize the body into JSON as well. The reason we do this is that API Gateway expects the HTTP response body to be a string. So if we want our web API to respond with JSON, we need to serialize it before handing it back to API Gateway. This also means that if you wanted to support some other form of serialization, like XML, YAML or something else -- you could do that, too.

Now let’s deploy it:

    sls deploy

Once done the Serverless framework will provide us with an endpoint:

    endpoints:
     GET - [https://XXXXXXXXXX.execute-api.us-east-1.amazonaws.com/dev](https://XXXXXXXXXX.execute-api.us-east-1.amazonaws.com/dev)

Your endpoint will look a little different. What just happened here? Well, in short, Serverless created our new AWS Lambda function and then configured an AWS API Gateway to point to this lambda function. The endpoint you see above is the one provided by API Gateway.

Let’s take our endpoint for a spin:

    curl [https://XXXXXXXXXX.execute-api.us-east-1.amazonaws.com/dev](https://XXXXXXXXXX.execute-api.us-east-1.amazonaws.com/dev)

And you should see the following response:

    {"message": "I'm an HTTP response"}

Congratulations, you’ve just created your first serverless web API. Now you’ve probably noticed that the URL API Gateway provides is pretty ugly. It would be a lot nicer if it could be something more readable like https://api.mywebapi.com/. Well, there's a [plugin for that](https://github.com/amplify-education/serverless-domain-manager), too.

## Cleaning Up

In this post we’ve created three Lambda functions and an API Gateway. But these were really just examples to help you get your feet wet in serverless. You’ll probably want to clean up and to do that just run:

    sls remove

And the Serverless framework will take care of the rest.

## In Summation

In this post we’ve gone over some of the AWS Lambda basics and created a simple web API and worker using the Serverless framework. In Part 2 we’ll cover some more advanced topics, including an introduction to two serverless frameworks specifically built for Python: [Zappa](https://www.zappa.io/) and [Chalice](https://github.com/aws/chalice). We’ll also demonstrate how quickly you can build a web API using these tools, including examples using popular web frameworks such as [Django](https://www.djangoproject.com/) and [Flask](http://flask.pocoo.org/).

[Check out part 2 for more!](https://read.iopipe.com/the-right-way-to-do-serverless-in-python-part-2-63430131239)
