
# How to create Alexa Skill using Python?



Lately, I’ve built a couple of applications for Alexa devices. I found out that a lot of people started to be interested in Alexa skill kit in terms of building or using the applications. If you will go to [Alexa Skill Shop](https://www.amazon.com/alexa-skills/b?ie=UTF8&node=13727921011) you can find a lot of apps starting on simple voice bot through smart home skills and ending with top music app players.

Let’s build an application that can tell a joke from [icndb](http://www.icndb.com/api/).

Alexa allows using HTTP webhooks or [lambda functions](https://aws.amazon.com/lambda/) as a communication between your system and end-user device. It is recommended to use lambda functions if the whole system is built on top of Amazon Services.

In this tutorial, I would like to show you how simple it is to create webhooks and pin them up in Alexa Skill console.

You can find source code on github [here](https://github.com/Piotr-Rogulski/alexa-skill-in-python).

## Requirements:

* [Amazon developer account](https://developer.amazon.com/)

* [Python](https://www.python.org/)

* [alexa-skill](https://www.github.com/stanwood/alexa-skill)

* [Falcon](https://falcon.readthedocs.io/en/stable/)

## Getting started

Install alexa-skill package to simplify Alexa usage:

    pip install alexa-skill

Each Alexa skill needs to specify [build-in intents](https://developer.amazon.com/docs/custom-skills/implement-the-built-in-intents.html).

<iframe src="https://medium.com/media/fd94ad01201ef5fa6c4f33b19996536a" frameborder=0></iframe>

Implement a key intent which will return random joke using BaseIntents class:

<iframe src="https://medium.com/media/0f275fc4db3494928d71226c7edc8eaa" frameborder=0></iframe>

Mapped methods should return Alexa response and True if the response was successfully handled, otherwise False.This is very helpful when analytics tools need to be connected. For chatbots, I recommend to use [Chatbase](https://chatbase.com/welcome).

All requests which are coming from a platform should be handled by one webhook. To do this I will use [falcon](https://falcon.readthedocs.io/en/stable/) web framework:

<iframe src="https://medium.com/media/960ff17c106f9b16f7952177f8f4342c" frameborder=0></iframe>

## Testing local webhook

Alexa requires an endpoint secured with https. To develop and test webhook served from a local machine I recommend to use [ngrok](https://ngrok.com/). It simply exposes local servers behind NATs and firewalls to the public internet over secure tunnels.

1. Download [ngrok](https://ngrok.com/download)

1. Setup local server

    $ pip install gunicorn 
    $ gunicorn webhook:app

3. Start *ngrok*

$ pip install gunicorn $ gunicorn webhook:app

    $ ./ngrok http 80

    http://127.0.0.1:4040 Forwarding                    http://0374b9cf.ngrok.io -> localhost:8000 Forwarding                    https://0374b9cf.ngrok.io -> localhost:8000  Connections                   ttl     opn     rt1     rt5     p50     p90                               0       0       0.00    0.00    0.00    0.00

4. Copy and use *forwarding url*

## Alexa console setup

1. Go to [alexa console](https://developer.amazon.com/alexa/console/ask) and *create skill*

1. Go to Intents tab and add new intent

![](https://cdn-images-1.medium.com/max/3944/0*nVvzz9aflR-IqnJp.png)

3. Add sample utterances, something like *Tell me a joke, Make me laugh etc…*

![](https://cdn-images-1.medium.com/max/2000/0*f2z1vcR-f1Wdx2wr.png)

4. Add *ngrok* webhook url

![](https://cdn-images-1.medium.com/max/3596/0*NE44x0q5MWoKvQJg.png)

5. Go to *test* tab and enable tests for a skill

6. From now on your webhook is ready to use

7. You can test it on *simulator* or *Alexa device*

## Summary

Alexa skill development is very simple. All you need to know is basic Python Programming language knowledge, one of the web frameworks like Django, Flask, Falcon or other… and that is all!

You don’t need to know Amazon Web services to create a voice application but you can become AWS Alexa skill developer. You can easily develop and test your Alexa skill using *ngrok* which will expose your local Python server to the web without deploying it to the remote server.

A good idea, user-friendly utterances and well defined Alexa responses are the keys to success.

*Originally published at [rogulski.it](https://rogulski.it/blog/alexa-skill-in-python/) on November 8, 2018.*
