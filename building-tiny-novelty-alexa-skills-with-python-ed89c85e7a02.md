
# A quick introduction to building Alexa skills with Python

Get this: In less than a half an hour, you’ll be talking to your Amazon Echo device, triggering a skill that you created. Don’t believe me? Read on.

Today we’re going to build a simple Alexa skill using Python and a few neat libraries. This skill will check the date and report back whether or not it’s Christmas — spoiler alert: the answer is probably no.

By the end of the journey, you’ll have experience with Flask, Zappa, and AWS Lambda. Most importantly, you’ll have the tools and resources necessary to build more complex skills, allowing you to integrate Alexa with all sorts of cool things.

Let’s get to it.

### Step 1: Setting up the environment

*Disclaimer: This section may be the hardest depending on your familiarity with development tools on your OS. If you’re not comfortable working in a terminal, this section will require some additional effort from you.*

Before we start programming, we need to get our development environment set up. For Python development, that means using pip and virtualenv. If you don’t have either installed on your machine, give [this article](https://www.dabapps.com/blog/introduction-to-pip-and-virtualenv-python/) a read and then report back here.

Let’s open up our terminal and create a new directory to house our project and our virtual environment:

    $ mkdir ~/alexa-isitchristmas
    $ cd ~/alexa-isitchristmas
    $ virtualenv env

We then need to activate our virtual environment:

    $ source env/bin/activate

Finally, we need to install the libraries we’ll be using to build our skill:

    $ pip install flask flask-ask zappa awscli

Good stuff! We’re ready to start writing code.

### Step 2: Creating the barebones app

Go ahead and create a new file called app.py and open it in your favorite text editor.

Let’s start with the basic command-line version of our skill. This code is what we’re really interested in, anyway — Alexa is just the interface we’re accessing it through.

We’re going to rely on the datetime module to write this code:

<iframe src="https://medium.com/media/5cb5df1cd62b8c6a2979ad2dfcbde716" frameborder=0></iframe>

<iframe src="https://medium.com/media/a563691e6d6badb931d8f7f195e8bce8" frameborder=0></iframe>

Go ahead and save that file and then run it from your terminal. Unless you’ve been saving this article for December 25th, the output should look like this:

    $ python app.py 
    False

Looks good. Next we’ll get Flask and flask-ask involved so that the Alexa can talk to our skill.

### Step 3: Plugging in Flask and flask-ask

If you’ve never worked with Flask before, let me give you a short introduction. Flask is a micro web framework that allows developers to build web applications with minimal effort. This is important for us because we’ll be exposing a web endpoint for Alexa to communicate with. Flask, coupled with the super cool flask-ask library, will help us build that.

Let’s look back at app.py. We’re going to refactor it into a Flask application, making use of flask-ask to (magically) handle the Alexa-specific code.

<iframe src="https://medium.com/media/03857c91cde1c4e253b76f5a59362faf" frameborder=0></iframe>

Let’s go over the changes we’ve made here:

* Lines 3–5: Import our new libraries, standard Python stuff.

* Line 12: Create a new Flask application object.

* Line 13: Create a new Ask object that will wrap our Flask app, enabling it to receive and make sense of the data sent to it by Alexa.

* Lines 16–21: Create a new view function and mapping it to the launch intent. When Alexa talks to our skill, this function will get called, and depending on the date, it’ll tell us if it’s Christmas or not.

* Line 25: Run the Flask application.

If you’ve made it this far, *pat yourself on the back.* The end is in sight!

### Step 4: Deploying our skill

We’re finished writing our skill’s code, and now all we have to do is stumble through some AWS configuration. Fortunately, the next time you write a skill, most of this will already be done.

First, you’ll need an Amazon Web Services account. Go ahead and [sign up for the AWS free tier](https://portal.aws.amazon.com/billing/signup#/start) if you don’t have one already.

Once that’s done, we need to configure our AWS client in our terminal so that we can deploy to AWS Lambda automatically. We’ll do that by following [this quick guide](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-quick-configuration).

After we’ve configured our AWS client, we’ll use Zappa to deploy our code to AWS Lambda.

Go ahead and create an zappa_settings.json file with Zappa’s CLI:

    $ zappa init

If you don’t know what to fill out, that’s alright. Hitting enter will fallback on the default configuration for each step, which is sufficient. Once we’re done, we’ll see zappa_settings.json in our project directory.

    Okay, here's your zappa_settings.json:

    {
        "dev": {
            "app_function": "app.app", 
            "aws_region": "us-west-2", 
            "profile_name": "default", 
            "project_name": "alexa-isitchris", 
            "runtime": "python2.7", 
            "s3_bucket": "zappa-j3tct0jtf"
        }
    }

Next up, we’ll need to deploy our skill to AWS Lambda. We can do this by executing the following:

    $ zappa deploy dev

*(Note: dev can be swapped out with something else if you didn’t name your environment “dev”).*

Zappa will do its thing, and if all goes well, we’ll receive a URL in our terminal:

    Deploying API Gateway..
    Deployment complete!: [https://v7kw6t5376.execute-api.us-west-2.amazonaws.com/dev](https://v7kw6t5k76.execute-api.us-west-2.amazonaws.com/dev)

Go ahead and copy that URL down for later.

### Step 5: Adding a new skill to the Alexa Skills Kit

We’re almost done, I promise.

Go ahead and [log in to the Amazon Developer site](https://developer.amazon.com/edw/home.html#/skills) using your Amazon credentials. Once we’re logged in, this page will greet us:

![Get started with Alexa](https://cdn-images-1.medium.com/max/2242/1*IVTIEavDwACBmeGYaesK_A.png)*Get started with Alexa*

We’ll click “Get Started” underneath “Alexa Skills Kit” to create our new skill.

In the top right corner, you’ll find the “Add a New Skill” button — click that.

Go ahead and copy the configuration settings below:

![Note: We set our invocation name to “is it christmas” so all we need to do is ask “Alexa, is it Christmas?”](https://cdn-images-1.medium.com/max/2000/1*OYXYGgFD_3lgiXjKp6D67w.png)*Note: We set our invocation name to “is it christmas” so all we need to do is ask “Alexa, is it Christmas?”*

Click “Save” and then navigate to “Interaction Model” on the left side of the screen.

The interaction model describes how our users will interact with our skill. In a later tutorial, we’ll expand this skill to experiment with the power of the interaction model, but for now, we need to skip past it. Unfortunately, the Alexa Skills Kit expects data here, so we’ll need to provide it with some dummy intents and utterances to continue.

Go ahead and copy the following into your intent schema:

    {
      "intents": [
        {
          "intent": "NullIntent"
        }
      ]
    }

And the following into your Sample Utterances:

    NullIntent null

If you’re thinking this is bizarre, you’re not crazy. Normally, your skills will do more than just launch, but because that’s all we’re doing here, the intents we register don’t matter at all. Go ahead and click “Next” in the bottom right and keep going.

We’re now in the Configuration tab. Here, we’ll configure the endpoint for Alexa to communicate with. Remember that URL that Zappa gave us? Now’s the time to use it. Match your configuration to the image below:

![](https://cdn-images-1.medium.com/max/2000/1*EwN4p-CJLUT9hsY4O2uQLg.png)

Go ahead and hit “Next” in the bottom right corner.

Alexa will only communicate over SSL. Fortunately, AWS Lambda provides us with an SSL certificate, so we’ll select the following option:

![](https://cdn-images-1.medium.com/max/2000/1*yUqGscFDGZhNFIvvAfQldQ.png)

Click “Next” one more time, and… **we’re done!**

Go run to your Alexa device and ask: “Alexa, *is it Christmas?*”

I want to stress that this is as simple as it gets. The bulk of this tutorial is geared toward getting you acclimated to AWS’s developer tools and deploying with Zappa. There’s a lot more to the flask-ask library, and you should check out its [documentation](http://flask-ask.readthedocs.io/en/latest/) to learn more about what you can do with it.

In a future tutorial, we’ll be moving past novelty skills like *Is it Christmas? *and developing skills with sessions, audio streaming, and more.
