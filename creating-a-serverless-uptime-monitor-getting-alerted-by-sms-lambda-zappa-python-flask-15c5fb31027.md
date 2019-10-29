Unknown markup type 10 { type: [33m10[39m, start: [33m96[39m, end: [33m119[39m }

# Creating a Serverless Uptime Monitor & Getting Alerted by SMS — Lambda, Zappa & Python

Creating a Serverless Uptime Monitor & Getting Alerted by SMS — Lambda, Zappa & Python

My last article was about [Creating a Serverless Python API Using AWS Lambda & Chalice](https://hackernoon.com/creating-a-serverless-python-api-using-aws-lambda-chalice-d321dc43ce2). After posting it to Reddit, I get a suggestion about using Zappa and since this is a framework that I already used, I thought about adding a new lesson to [Practical AWS training](http://practicalaws.com).

![](https://cdn-images-1.medium.com/max/2000/1*v828PObeIDREEKHwQM8HUg.jpeg)

[Zappa](https://github.com/Miserlou/Zappa#) is a python serverless microframework for AWS.

It allows developers build and deploy serverless Python applications (including, but not limited to, WSGI web apps) on AWS Lambda + API Gateway.

This is an introduction (probably much more) for those who’d like to test and learn how to use this framework and host an API (or any other application) in AWS Lambda.

As part of my work in creating the content of [Practical AWS](http://practicalaws.com) training, this article is a prototype. The complete lesson will be included in the training with better examples and explanations.
> # If you are interested in [Practical AWS training](https://practicalaws.com), you can watch this demo:

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/Z6UilbN8WEg" frameborder="0" allowfullscreen></iframe></center>

Start by creating the virtual environment:

    virtualenv -p python3.6 zappa_example

make sure to use Python 3.6

Let’s start a Zappa project:

    cd zappa_example
    . bin/activate
    mkdir zappa_project
    touch zappa_project/__init__.py
    zappa init 
    pip install zappa flask

When initializing a new Zappa project, the framework will ask you some questions that will be used to configure the project:

    ███████╗ █████╗ ██████╗ ██████╗  █████╗
    ╚══███╔╝██╔══██╗██╔══██╗██╔══██╗██╔══██╗
      ███╔╝ ███████║██████╔╝██████╔╝███████║
     ███╔╝  ██╔══██║██╔═══╝ ██╔═══╝ ██╔══██║
    ███████╗██║  ██║██║     ██║     ██║  ██║
    ╚══════╝╚═╝  ╚═╝╚═╝     ╚═╝     ╚═╝  ╚═╝

    Welcome to Zappa!

    Zappa is a system for running server-less Python web applications on AWS Lambda and AWS API Gateway.
    This `init` command will help you create and configure your new Zappa deployment.
    Let's get started!

    Your Zappa configuration can support multiple production stages, like 'dev', 'staging', and 'production'.
    What do you want to call this environment (default 'dev'): dev

    AWS Lambda and API Gateway are only available in certain regions. Let's check to make sure you have a profile set up in one that will work.
    We found the following profiles: default, xxxx, xxxxx, xxxxxx, xxxxxx, and xxxxxx. Which would you like us to use? (default 'default'):

    Your Zappa deployments will need to be uploaded to a private S3 bucket.
    If you don't have a bucket yet, we'll create one for you too.
    What do you want call your bucket? (default 'zappa-c49kj1rye'):

    What's the modular path to your app's function?
    This will likely be something like 'your_module.app'.
    Where is your app's function?: zappa_project.app

    You can optionally deploy to all available regions in order to provide fast global service.
    If you are using Zappa for the first time, you probably don't want to do this!
    Would you like to deploy this application globally? (default 'n') [y/n/(p)rimary]:

    Okay, here's your zappa_settings.json:

    {
        "dev": {
            "app_function": "zappa_project.app.app",
            "aws_region": "eu-west-1",
            "profile_name": "default",
            "project_name": "zappa-example",
            "runtime": "python3.6",
            "s3_bucket": "zappa-c49kj1rye"
        }
    }

    Does this look okay? (default 'y') [y/n]:

    Done! Now you can deploy your Zappa application by executing:

    $ zappa deploy dev

    After that, you can update your application code with:

    $ zappa update dev

    To learn more, check out our project page on GitHub here: [https://github.com/Miserlou/Zappa](https://github.com/Miserlou/Zappa)
    and stop by our Slack channel here: [https://slack.zappa.io](https://slack.zappa.io)

    Enjoy!,
     ~ Team Zappa!

As you may read, I used the “dev” environment and kept everything to the default values apart the modular path to your app’s function that I configured to

    zappa_project.app.app

since my module (the application main folder) is called zappa_project, my function will be written in app.py file and my Flask application will be called app.

Now you can see your configuration file under zappa_example:

    {
        "dev": {
            "app_function": "zappa_project.app",
            "aws_region": "eu-west-1",
            "profile_name": "default",
            "project_name": "zappa-example",
            "runtime": "python3.6",
            "s3_bucket": "zappa-c49kj1rye"
        }
    }

Now we can deploy our Zappa application by executing:

    zappa deploy dev

![](https://cdn-images-1.medium.com/max/2000/1*yMgYH8YggSTnKBdK2M9Fcg.gif)

Later after that, we can update our application code with:

    zappa update dev

After deploying, your source code will be zipped and uploaded to Lambda.

    ls *.zip 
    zappa-example-dev-1507300010.zip

This is the Flask application to start with:

    from flask import Flask
    app = Flask(__name__)

    [@app](http://twitter.com/app).route('/')
    def hello():
        return 'Hello\n'
        
    if __name__ == "__main__":
        app.run()

After adding the hello world code, we should update our deployment:

    zappa update dev

You can actually have some problems, so you will need to examine your logs. You can execute:

    zappa tail

Now let’s add a method to handle a POST request with user data using Flask:

    [@app](http://twitter.com/app).route('/ping', methods=["POST"])
    def ping():
        resp_dict = {}
        
        if request.method == "POST":
            data = request.form
            domain = data.get("domain", "")
            
            try:
                result = requests.get(domain, timeout=timeout).elapsed.total_seconds()            
                resp_dict = {"result": result, "response": "200"}
                response = Response(json.dumps(resp_dict), 200)
                
            except Exception as e:
                resp_dict = {"result": "error", "response": "408"}
                response = Response(json.dumps(resp_dict), 408)
                
        return response

This method will simply return the response time of the website the user will send.

Our Flask app will look like this:

    from flask import Flask, Response, json, request
    import requests

    app = Flask(__name__)

    [@app](http://twitter.com/app).route('/')
    def hello():
        return 'Hello World !'

    [@app](http://twitter.com/app).route('/ping', methods=["POST"])
    def ping():
        resp_dict = {}
        
        if request.method == "POST":
            data = request.form
            domain = data.get("domain", "")
            
            try:
                result = requests.get(domain).elapsed.total_seconds()
                resp_dict = {"result": result, "response": "200"}
                response = Response(json.dumps(resp_dict), 200)
                
            except Exception as e:
                resp_dict = {"result": "error", "response": "408"}
                response = Response(json.dumps(resp_dict), 408)
                
        return response

    if __name__ == "__main__":
        app.run()

We can test it locally using CURL.

You can run the Flask app locally:

    export FLASK_APP=app.py && flask run

Now execute the CURL that will send a POST request with the site domain as data:

    curl --data "domain=[http://devogslinks.com](http://devogslinks.com)" [http://127.0.0.1:5000/ping](http://127.0.0.1:5000/ping)

It should return something like this:

    {"response": "200", "result": 0.21425}

You can delete this serverless app and remove its API Gateway using:

    zappa undeploy dev

## What’s Next ?

When the website doesn’t reply, you should normally be notified. Using AWS SNS (Amazon Simple Notification Service) to send an SMS to the ops engineer who’s responsible of this website, is a good solution !

To make this, we can simply use Boto3 and schedule the function to ping the website each x seconds, this could be done using AWS.

This part can be found in [Practical AWS training](http://practicalaws.com).

![](https://cdn-images-1.medium.com/max/2000/1*niGGB6Q2M8uvBLswKHXgKA.png)
> # You can find the full lesson in [Practical AWS](http://practicalaws.com).

## Join Us !

If you like what you’ve read, please **share with your followers**, and **give a long clap** with the little hands button below.

We are working hard in order to give you the best of what we master. We are building online courses for everyone and every level: Newbies, intermediate and skilled people.

Our goal is giving people the opportunity to enter the DevOps world trough quality courses and practical learning paths **inspired from the real IT world use cases !**
> # If you are interested in [Practical AWS training](https://practicalaws.com), you can [make a preorder](http://practicalaws.com) from now and get the training at almost its half-price.
> # Receive [My Free Guide](https://app.getresponse.com/site2/practical_aws?u=hWKcs&webforms_id=BSioY) That Will Help You to Learn AWS

## Want to Contribute for Fun & Profit ?

We are looking for skilled people to help us deliver the DevOps ecosystem with great and high quality contents.
> # We ❤ DevOps ! [Join us here](https://eon01.typeform.com/to/hlmmcG).

![](https://cdn-images-1.medium.com/max/2000/0*Nu61Xm8x-xMrUIXE)

**Join our community Slack and read our weekly Faun topics ⬇**
[**Join a Community of Aspiring Developers.Get must-read articles, learn new technologies for free…**
*Join thousands of developers and IT experts, get must-read articles, chat with like-minded people, get job offers and…*www.faun.dev](https://www.faun.dev/join/?utm_source=medium.com%2Ffaun&utm_medium=medium&utm_campaign=faunmedium)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
