
# Amazon Alexa Skill Recipe with Python 3.6

A “skill” is another word for “app” as it pertains to Alexa voice-commands and responses. This article provides a very basic, nearly elementary, procedure for setting up your first Alexa app (skill) using Python 3.6 in an AWS Lambda service.

## Ingredients:

* A developer account on [https://developer.amazon.com](https://developers.amazon.com) (“Amazon Developer Console”)

* An AWS account on [https://aws.amazon.com](https://aws.amazon.com) (“AWS Console”)

* Beginner knowledge of Python 3.x syntax

*Note: You can use your existing Amazon account for the first two services.*

## Instructions:

In your Amazon Developer Console, sign in. Choose **Alexa** from top menu, then **Alexa Skills Kit**.

![Choose “Alexa” from main menu](https://cdn-images-1.medium.com/max/2244/1*aTvVdA8Fk0s2rnbu68KQbA.png)*Choose “Alexa” from main menu*

![Choose “Get Started” under “Alexa Skills Kit”](https://cdn-images-1.medium.com/max/2000/1*R0rbXiTrKccoP896V7K2DA.png)*Choose “Get Started” under “Alexa Skills Kit”*

On the next screen, choose **Add a New Skill**.

![Choose “Add a New Skill”](https://cdn-images-1.medium.com/max/2000/1*93zEs9QGE480WWhoE2HaXg.png)*Choose “Add a New Skill”*

You will be presented with a screen where you will configure global settings for your app. We will make an app that gives us dinner suggestions. We will name it “Dinner Bot”.

“Dinner Bot” will be both the name and ***invocation*** for our app (meaning, we will launch it by the same words, “Dinner Bot”). Therefore we will enter “Dinner Bot” in two places:

![](https://cdn-images-1.medium.com/max/4632/1*FDdQFEVK2H3CfjA_rvufbw.png)

Click **Save**. Then click **Next**. *Note: if “Dinner Bot” changes to lowercase in the invocation input, don’t worry. It’s not the formal name; just the spoken way to launch your app.*

On the next screen you will configure the “Interaction Model”. Think about what things you might want to ask dinner bot. Amazon calls these ***utterances***:

*“What should I have for dinner?”*

*“Do you have a dinner idea?”*

*“What’s for dinner?”*

We will link all three of these utterances with an ***intent***. An intent is just a fancy label for a group of actions (functions) that will occur if the utterances are spoken to Dinner Bot. We need a name for this intent. We will call it DinnerBotIntent (not very original, but it will do). Fill out the Interaction Model as follows (JSON on top, DinnerBotIntent utterances on bottom):

![](https://cdn-images-1.medium.com/max/2400/1*hiHocIBsEe9ppAuwGJ7RMA.png)

Notice the removal of all punctuation from the utterances — this is important!

Click Save, and Next.

You’ll see this screen:

![](https://cdn-images-1.medium.com/max/2280/1*iBf0Vz9SFNjMgh1CaujycA.png)

Stop here! Don’t close this tab…

In another browser tab, sign in to [https://aws.amazon.com](https://aws.amazon.com)

***Important! You may need to switch your location (upper right corner) to “N. Virginia” to access Alexa triggers.***

Upon successful login, you will see the following screen where you should choose Lambda (if you don’t see the full menu, you can expand “All services”:

![](https://cdn-images-1.medium.com/max/2812/1*BaXh2Dg1qigHRkF_rf-AdA.png)

On the next screen, you should see, and choose, Create Lambda Function. Instead, if you have not created a Lambda previously, you may have to go through a “Get Started” screen first. Worse, if you’re a new user you might have to verify your account. But hopefully, you’ll eventually see and click this:

![](https://cdn-images-1.medium.com/max/2304/1*SL0j6y2VUcB95s3Z0MHDhg.png)

On the next (Select blueprint) screen, select “Blank Function”:

![](https://cdn-images-1.medium.com/max/3148/1*bdTlzESiue-rJbjHi2O0dA.png)

On the next screen select the trigger box, then “Alexa Skills Kit”, then **Next**.

![](https://cdn-images-1.medium.com/max/3288/1*oOEgQ4Ix4sdLJXGAzHUB9w.png)

After clicking Next you will see this screen. This is where you’ll give your (lambda) code a name and switch its environment to Python 3.6. Never mind that I spelled Lambda wrong in the example. What’s done is done:

![](https://cdn-images-1.medium.com/max/3208/1*3hkb8nW8gMoX191gHgysmA.png)

Let’s change the default code to something just slightly more interesting:

![](https://cdn-images-1.medium.com/max/3164/1*XG4Vqjkn5mjmMAmGnTtWMQ.png)

Now let’s configure our lambda with the right “role”. If you’ve created Lambdas before you may see the option “Choose an existing role”, and be able to choose lambda_basic_execution. If you do not see the option to “Choose an existing role”, you should choose “Create a custom role” from the drop-down, which will take you to a screen that prompts you to create a lambda_basic_execution role for which you will need to click “Allow” (see graphic below):

![](https://cdn-images-1.medium.com/max/2000/1*EQBwfl1UL_A6dUhruCq4Bg.png)

Then you’ll see *index.handler* and *existing*, along with your new role (or *lambda_handler* like the graphic below if you’ve already done this previously). Then choose **Next.**

![](https://cdn-images-1.medium.com/max/3136/1*Tq_QvOTtHqxiOUljmPp7bg.png)

On the verification screen, simply click **Create Function**:

![](https://cdn-images-1.medium.com/max/2728/1*W7IRsMIF5QxjJ1Qec5N6CQ.png)

Finally, on the confirmation (success) screen, copy that ARN number up in the right corner. You’ll need it:

![](https://cdn-images-1.medium.com/max/3308/1*t9KQWCJ2-mHN7SO7EnXf-A.png)

Now let’s go back to the tab in the developer portal where we left off and fill in the missing pieces to our Alexa Skill (I’m on the east coast, US):

![](https://cdn-images-1.medium.com/max/2396/1*Q4JrZryROxSk7UYv2KhBNw.png)

Click **Save**, and then **Next**.

On the next screen you finally get to test your Alexa Skill! Type in one of your utterances in the testing box and click **Ask Dinner Bot**. The right box should populate with a JSON response:

![](https://cdn-images-1.medium.com/max/2464/1*YaxRHr0JYOM-xZLbZvaiWA.png)

Great, right?

It could be better if you have an Amazon Echo, but if you don’t, no worries, just fire up the simulator.

Visit [https://echosim.io/](https://echosim.io/) and sign in with your Amazon account.

Ask your Echo hardware, or hold the mic button down on Echosim while you (clearly) ask…

***“Alexa, ask Dinner Bot, what’s for dinner?”***

Then let go of the Echosim mic button, and wait. Echosim should say “processing” and then return with your answer.

![](https://cdn-images-1.medium.com/max/2744/1*vMZwQAKA8MRQdsEBB6qotQ.png)

If Echosim doesn’t answer, don’t worry. It can be flaky. A better indication of your working app is the JSON in the previous screenshot, or better yet, a physical Amazon Echo (linked to your account). The Echo Dot is only $49.99 and plenty capable!

What to do next? If you want Dinner Bot to do something different based on another voice command, create another intent in the Developer portal for your Alexa skill. Then follow up in the Lambda code with an if statement based on the request JSON’s intent name. Here is what the request JSON looks like:

![](https://cdn-images-1.medium.com/max/2000/1*cnzCISESjqPa6-eWeQgc4A.png)

So, grabbing the incoming request through the event dict…

intentName=event["request"]["intent"]["name"]

You could test intentname for the value of your intent string, e.g. “DinnerBotIntent”, and craft the response accordingtly.

I hope this has been useful. Have fun!
