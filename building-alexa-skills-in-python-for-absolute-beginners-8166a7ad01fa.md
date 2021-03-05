
# Building Alexa skills in Python, for absolute beginners.

Voice is an exciting field right now, and if you’ve ever wanted to make an Alexa skill, but couldn’t figure out how, you’ve come to the right place. I wrote this because the existing data, guides, and tutorials available are either scattered, unfocused, or hard to understand for a beginner

While I hope to make this article really easy to understand, with thorough noob-level explanations, I should point out that the reader should have at least an introductory grasp of Python, or if coming from a different language, be aware of how functions, decorators, and the so work in Python.

Without further ado, let’s get started.

Step 1: You should have Python (3.x) installed on your computer. You also need an editor for writing code. I personally use [Atom](https://atom.io/). When you install Python, pip and virtualenv get installed with it. In case they aren’t, Google any errors you come across for solutions. It’s beyond the scope of this guide, so I won’t go too much into it.

Step 2: Think about what we’re going to make. In this case, we’re going to make a skill that would fetch the top posts from the [*r/showerthoughts](https://www.reddit.com/r/showerthoughts) *subreddit. A good practice to follow while developing a voice application is called ‘dialogue-driven development’. It’s a fancy way of saying that the first thing to do is to take a piece of paper and write down a script of how you expect the skill to work. To dive deeper into this, read my article on VUI (Voice User Interface) Design for dummies (when it comes out, I haven’t written it yet lmao).

![](https://cdn-images-1.medium.com/max/2000/1*bw8cD8n1f3cxsQpaSlBeNg.png)
> # Let’s talk a bit about the NLP (Natural Language Processing) part. NLP is what enables Alexa to understand *what you mean or want *when you say or ask something, i.e, your **intent**. And what you say or ask is called an **utterance**.

Consider the above example. When Alexa asks you whether you’d like to hear a shower thought, you say ‘You bet’. This ‘You bet’ is your **utterance**. An NLP engine in the background uses machine learning models to link your utterance to the correct **intent**. In other words, Alexa takes your utterance and tries to figure out your intent. A developer makes and defines a set of intents and their respective utterances for a skill.

![](https://cdn-images-1.medium.com/max/2000/1*ZFAmNuYAF1gokmKSuK7NZQ.png)

If this doesn’t make much sense right now, don’t worry. It’ll become clearer over the next few steps.

Step 3: Head over to the [Alexa Developer Console](https://developer.amazon.com/alexa/console/ask). You’ll have to login using an Amazon account. If you use one for shopping, you could use the same account. Then click on the blue *Create Skill* button, probably somewhere to the right of your screen. On the next window, enter your skill’s name. I’ll enter mine as Showertime Thoughts. Choose *Custom* for your skill model and *Self-Hosted* for hosting method. Once that’s done, you’ll come to a place that looks like this.

![](https://cdn-images-1.medium.com/max/3818/1*OYzdtbsOro1GCS7QihqThg.png)

You’ll see that your invocation name is already set. An **invocation name** is the name, upon hearing which Alexa launches your skill. By default, your skill’s name is the invocation name. In this case, the invocation name is set as ‘showertime thoughts’. In our script, we say ‘*Alexa, open Showertime Thoughts*’ to start our skill.

Step 4: On the left, there is the Intents tab. By default, Amazon provides four intents. These are the basic bare-bones for a project. Let’s add our own intents. We know from our script that one of the intents a user will have is to get a new shower thought. Click on the Add button next to Intents, and name the it GetShowerThoughtIntent. You can choose to name it any appropriate name.

![](https://cdn-images-1.medium.com/max/3796/1*sXR3dLc3V5kkuxA59wLmjw.png)

Once you create an intent, you’ll reach the sample utterances page of that intent. This is where you can input a bunch of sample utterances that you expect a user would say for that intent. In our script, we know that a user would say things like ‘yes’, ‘yep’, ‘of course’, ‘absolutely’, ‘you bet’ etc. A user could also say something like ‘get me a new shower thought’, ‘i want a shower thought’ etc. Put in as many sample utterances as you can think of, the more the better.

![](https://cdn-images-1.medium.com/max/3768/1*hac4-iEDVOGdvLuRGsZWHw.png)

Once you’re done, click on the Save Model button on top. Then click on the *AMAZON.StopIntent* intent. According to our script, the user would also say something to stop the skill. Fill in some sample utterances for the StopIntent intent, like ‘bye’, ‘that’s enough’, ‘no more’ etc. Once you’ve filled in enough sample utterances for both intents, click on the Build Model button on top.

Step 5.1: In the Start Menu, search for Command Prompt. Right-click and choose ‘Run as administrator’. Then run the following commands.
> cd/
> mkdir ShowTho
> cd ShowTho

This creates a folder called ‘ShowTho’, probably in your C: drive, depending on your computer’s settings. Then run the following to install the modules we need. Notice that the period after ‘target’ is intentional and part of the command.
> pip install ask-sdk — — target .
> pip install requests — — target .

Step 5.2: Open up Atom. Start by importing the modules we just installed.
> import logging
> from ask_sdk_core.skill_builder import SkillBuilder
from ask_sdk_core.utils import is_request_type, is_intent_name
from ask_sdk_core.handler_input import HandlerInput
> from ask_sdk_model.ui import SimpleCard
from ask_sdk_model import Response
> import requests
import random

Initialize. (Add the following code to it.)
> sb = SkillBuilder()
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

Then add the following code. Don’t worry, I’ll explain it, line-by-line.
> @sb.request_handler(can_handle_func=is_request_type("LaunchRequest"))
def launch_request_handler(handler_input):
 speech = "Hi, welcome to Showertime Thoughts."
 asktext = "Would you like to hear one?"
 handler_input.response_builder.speak(
 speech + " " + asktext).ask(asktext)
 return handler_input.response_builder.response

The request_handler() decorator.
> @sb.request_handler(can_handle_func=is_request_type(“LaunchRequest”))

Your code checks the input it receives from Alexa to figure out what kind of request or intent the user asked for. When Alexa opens your skill, it sends a LaunchRequest to your backend, i.e, your code. The *is_request_type(“LaunchRequest”) *part identifies that Alexa has sent a LaunchRequest.
> def launch_request_handler(handler_input):

Once your code identifies which request or intent is being requested by Alexa, it runs the function linked to that request or intent. Here, the launch_request_handler function is linked to the LaunchRequest decorator, and since we’re getting a LaunchRequest from Alexa, this function will run.
> handler_input.response_builder.speak(speech + “ “ + asktext).ask(asktext)

The following code uses the response_builder to set the output response delivered by Alexa. Whatever you pass into speak() will be spoken by Alexa. If she does not get an appropriate response from the user, she will once again re-prompt the user with the value inside the ask() function. Note that we’re using ask() because we expect a response from the user. ask() is optional and needs to be used only when we’re asking the user something.
> return handler_input.response_builder.response

Once the output values have been set, we use the above line to return our built response to Alexa.

Next, let’s write a decorator for our GetShowerThoughtIntent.
> @sb.request_handler(can_handle_func=is_intent_name(“GetShowerThoughtIntent”))

Notice that for LaunchRequest, we use *can_handle_func=**is_request_type**(“LaunchRequest”), *but for an intent we use can_handle_func=***is_intent_name***(“GetShowerThoughtIntent”).
> def whats_my_color_intent(handler_input):
> url = ‘[https://www.reddit.com/r/showerthoughts.json?limit=100'](https://www.reddit.com/r/showerthoughts.json?limit=100')
 response = requests.get(url,headers={‘User-agent’:’showertimethoughts’})
 r = random.randint(1, 100)
 showerthought = response.json()[‘data’][‘children’][r][‘data’][‘title’]
 asktext = “Wanna hear one more?”
 handler_input.response_builder.speak(showerthought).ask(asktext)
> return handler_input.response_builder.response

We’re making a variable called url, that points to the Reddit url of the Shower Thoughts subreddit. We use a library called requests that we imported earlier to take in the data from the Reddit url, which we obtain in the JSON format. We get the top 100 shower thoughts from this. We use random.randint(1,100) to generate a random number and store that value in r. We then use this to pick a random shower thought and set its value as the variable showerthought. We also modify the asktext. We then pass both to the response_builder.

Save this file with the filename as *main.py* in the ShowTho folder.

To see if everything works, open Command Prompt and type in
> python main.py

Hopefully, your console won’t show anything. That means there weren’t any errors. If you do run into errors, try looking through your code to see if there are any bugs. If you were copy-pasting code, pay attention to the spacing and indentation.

The next thing to do is to upload this code into a hosted back-end. The recommended way would be to use AWS Lambda. To do so, check out the official guide by Amazon [here](https://developer.amazon.com/en-US/docs/alexa/custom-skills/host-a-custom-skill-as-an-aws-lambda-function.html).

Thanks for reading, and good luck!
> Hitesh Nair ([hiteshnair.com](http://hiteshnair.com))
