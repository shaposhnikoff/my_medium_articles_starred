
# Using Twilio to Send SMS Texts via Python, Flask, and Ngrok



Python is great for many things. One thing, in particular, is hacking together code to develop and deploy simple web apps for particular functions or utilities you may need in your business. In this tutorial, I’ll be going over several how to integrate [Twilio’s Programmable SMS systems using their Communications REST API](https://www.twilio.com/docs/api?filter-product=sms), [Python’s Flask micro-framework](http://flask.pocoo.org), and [ngrok](https://ngrok.com), a cloud service that exposes local servers behind NATs and firewalls to the public internet over secure tunnels.

## Outbound SMS — How Does It Work?

![](https://cdn-images-1.medium.com/max/3210/1*U7-ZBE0RJrCOnd_S0DS6hg.png)

Using Twilio’s REST API, you can send outgoing SMS messages from your Twilio phone number to mobile phones around the globe.

Your application makes a post request to Twilio. Once received, Twilio sends a text message to a number declared in the Python script, or whatever variable is declared. It’s that simple.

## Creating a Twilio Account & Purchasing a Number

Before you begin, you’ll need to open an account with Twilio and also purchase a number that your messages will be sent from. It’s a fairly simple process and you won’t need to spend anything for this tutorial as they give you free credits to purchase and test out their services. If you plan on integrating Twilio into your business, you will incur some costs in the future so it’d be a good idea to skim over their pricing model.

Head over to [Twilio.com](https://www.twilio.com)and sign up for a free account. Once registered, you can purchase a number using the credits they give new members. For the sake of brevity, [I’ll link you to their support page on How to Purchase a Number.](https://support.twilio.com/hc/en-us/articles/223135247-How-to-Search-for-and-Buy-a-Twilio-Phone-Number-from-Console)

![Make sure you purchase a number that is SMS capable.](https://cdn-images-1.medium.com/max/2000/1*d5J_f-4R0Y9Gt43lZTrUUA.png)*Make sure you purchase a number that is SMS capable.*

## Setting Up Your Environment and Installing Packages

In my last post, I went over how to [configure Ubuntu for Python development, Flask apps, and VirtualEnv](https://medium.com/@jovanshernandez/configuring-ubuntu-18-04-for-a-python-dev-environment-with-flask-gunicron-and-virtualenv-674866bc2199). You might want to check that out first but if you already have, let’s begin by installing twiliopackage. *(NOTE: You should be on Python3 for this tutorial.)*

    pip install twilio

With Twilio installed, we can now send outbound messages from the number we purchased to target numbers with a simple Python script. Create and open a new file called send_sms.pyand type or paste in this code sample. Make sure to replace the placeholder values with your own information *(those being your account_sid, auth_token, body, from, and tofields. You can find your Account SID and Auth Token in your [Twilio Console.](https://www.twilio.com/console))*

    # Download the helper library from [https://www.twilio.com/docs/python/install](https://www.twilio.com/docs/python/install)

    from twilio.rest import Client

    # Your Account Sid and Auth Token from twilio.com/console
    account_sid = ‘AC00000000000000000000000000000’
    auth_token = ‘your_auth_token’
    client = Client(account_sid, auth_token)

    message = client.messages.create(
     body=’This is a test message!’,
     from_=’[+][1][5555555555]',
     to=’[+][1][6666666666]'
     )
    print(message.sid)

![Replace the placeholder values with your own information.](https://cdn-images-1.medium.com/max/2560/1*p9dM36e1Z4Q3ZZT4iArK9g.png)*Replace the placeholder values with your own information.*

Once done, save and run the script with the command below:

    python send_sms.py

That’s it! You should have a message from your Twilio phone number hit your cell phone shortly. You’ll see a prefix to your message if you are on a Trial account.

![Ignore my goofy test messages :P](https://cdn-images-1.medium.com/max/2004/1*MDxpF34OJRMdg8NHWHlnyg.png)*Ignore my goofy test messages :P*

## Receiving and Replying to Incoming SMS Messages

In order to receive and reply to incoming SMS messages, we will need to install Flask, setup a virtualenv sandbox, and also configure ngork so Twilio can talk to our app. Let’s begin.

### Installing Pip and Virtualenv

If you are familiar at all with Python development or have seen my last post on setting up a development environment, this should seem familiar. Lucky for us, if you are on Python 3.4+, pip should automatically be installed. If not, Google is your friend.

Not to install virtualenv, just run the following command:

    pip install virtualenv

Once installed, navigate to your directory where you are keeping this project and create a virtual environment.

    cd ~/path_to_your_project
    virtualenv — no-site-packages .

    ###ACTIVATE THE VIRTUALENV SANDBOX
    source bin/activate

Once activated, you should see the name of your project sandbox prefixed in your terminal. [If you are confused or ran into an issue, see one of my previous posts here.](https://medium.com/@jovanshernandez/configuring-ubuntu-18-04-for-a-python-dev-environment-with-flask-gunicron-and-virtualenv-674866bc2199)

### Installing dependencies in your Virtualenv

Virtualenv is great for testing apps and installing packages and dependencies unique to each project. For this Twilio app, we are going to need two packages. Start by creating a new file called requirements.txtand adding the following lines:

    ###YOUR VERSIONS MAY VARY WHEN DEPENDING WHEN YOU GO THROUGH THIS TUTORIAL

    Flask>=0.12
    twilio~=6.16.0

Then install these packages by running the following:

    bin/pip install -r requirements.txt

If all is well, we can test everything by creating a test file and running it locally. Make sure your virtualenv is activated, navigate to your project folder and create a new file called run.py. Paste or write the following code:

    from flask import Flask
    app = Flask(__name__)

    [@app](http://twitter.com/app).route(“/sms”)
    def hello():
     return “This is your Twilio App Working!”

    if __name__ == “__main__”:
     app.run(debug=True)

![Test code for the Twilio app in Flask.](https://cdn-images-1.medium.com/max/2560/1*-xvjRQdrbWWyzfaI1dtsKA.png)*Test code for the Twilio app in Flask.*

Now let’s run the script by running python run.py. 
You should see: * Running on [http://127.0.0.1:5000/](http://127.0.0.1:5000/) 
Naviage to localhost:5000/sms. You should see the following text in your browser:

![What you should see if you did it correctly.](https://cdn-images-1.medium.com/max/2560/1*CqFczmNCN9n3MpCsumO1OQ.png)*What you should see if you did it correctly.*

## Allow Twilio to talk to your Flask App Using ngrok

When you’re working on your Flask application in your development environment, your app is only reachable by other programs on your computer. This is why your app is only available on localhostor 127.0.0.1. In order to resolve this, we can deploy our app on something like Heroku, AWS, or other cloud services, but for this specific tutorial, we are going to use ngrok.

[Per their own website](https://ngrok.com/docs), ngrok *“allows you to expose a web server running on your local machine to the internet. Just tell ngrok what port your web server is listening on.” *Once started, Ngrok provides a unique URL on the ngrok.io domain which forwards incoming requests to your local development environment. If you’ve never used ngrok, head over to their [download page](https://ngrok.com/download)and grab the binary for your OS.

Once the package has been downloaded and unzipped, we can run our test app we made previously: python run.py. Then in a new terminal app, we’ll initialize a session with ngrok with the following command:

    ./ngrok http 5000

You should see output similar to the terminal in the top left of the screenshot below:

![Ngrok gives you public access to locally hosted applications.](https://cdn-images-1.medium.com/max/2560/1*kCz1zOyP7XsspTMqTyNLJA.png)*Ngrok gives you public access to locally hosted applications.*

Copy your unique public address, put it in a browser, and trail it with a *“/sms/”*. For instance, the address above should be “*http://a0f8f155.ngrok.io/sms*”.

## Replying to Incoming SMS Messages Sent from Twilio

When someone sends an SMS to your Twilio phone number, Twilio makes an HTTP request to your server asking for instructions on what to do next. Once you receive the request, you can tell Twilio to reply with an SMS, kick off a phone call, store details about the message in your database, or trigger an entirely different action.

For this specific tutorial, we’ll have Twilio reply to the user’s response with an acknowledgment receipt. Use a text editor to open run.pyagain and edit the code to reflect the following:

    # /usr/bin/env python
    # Download the twilio-python library from twilio.com/docs/libraries/python
    from flask import Flask, request
    from twilio.twiml.messaging_response import MessagingResponse

    app = Flask(__name__)

    [@app](http://twitter.com/app).route("/sms", methods=['GET', 'POST'])
    def sms_ahoy_reply():
        """Respond to incoming messages with a receipt SMS."""
        # Start our response
        resp = MessagingResponse()

    # Add a message
        resp.message("Thank you for your response! We are confirming your message.")

    return str(resp)

    if __name__ == "__main__":
        app.run(debug=True)

Feel free to edit the code above, especially the line under # Add a message.I was playing around with my code while watching the Matrix so my messages will seem goofy in the following examples.

Save the file and run the python run.pycommand again. Make sure ngrok is still running and copy the custom public URL they give you. For Twilio to know what URL to grab, we need to configure our Twilio phone number to call your webhook URL whenever a new message comes in, which is what ngrok does for us.

Log into Twilio.com and go to the [Console’s Numbers page](https://www.twilio.com/console/phone-numbers/incoming), click on your SMS-enabled phone number and find the Messaging section. The default “CONFIGURE WITH” is what you’ll need: “[Webhooks](https://www.twilio.com/docs/glossary/what-is-a-webhook)/[TwiML](https://www.twilio.com/docs/glossary/what-is-twilio-markup-language-twiml)”.

In the “A MESSAGE COMES IN” section, select “Webhook” and paste in the URL you want to use.

![Paste your custom ngrok URL in the Twilio Console Number’s page.](https://cdn-images-1.medium.com/max/3732/1*RHtUjvH5J8i3CWs7y3wkXA.png)*Paste your custom ngrok URL in the Twilio Console Number’s page.*

Now let’s test your entire SMS messaging app by executing your two python scripts.

    ###RUN THIS TO SEND THE INITIAL TEXT
    python send_sms.py

    ###RUN THIS TO MAKE SURE TWILIO IS LISTENING FOR THE RESPONSE
    python run.py

![Everything running together. You can confirm your response script is publicly available by going to the URL.](https://cdn-images-1.medium.com/max/2560/1*iXaD_Pi0DV30KkW5lERE3A.png)*Everything running together. You can confirm your response script is publicly available by going to the URL.*

You should receive your initial text, and if you reply to it, you should get the response text that you coded in your second python script.

![Again, ignore my goofy Matrix references :P](https://cdn-images-1.medium.com/max/2004/1*yhWkbxADa1LCXoZLSK9tjA.png)*Again, ignore my goofy Matrix references :P*

That’s pretty much it! With the knowledge from this tutorial, you can build out a fully-fledged web app that you have more than likely interacted with already. Many professions from doctors and dentists use services like this to confirm appointments, or even ride-sharing apps like Uber to confirm a user is ready to be picked up.

There are many other Twilio tutorials out there that guide you through similar processes. Piece some of them together to build your own custom app!
