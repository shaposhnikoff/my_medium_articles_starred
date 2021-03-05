
# My first Alexa custom skill

Like millions of others, we got an Amazon Echo for Christmas. Well, it’s an Echo Dot, and it wasn’t exactly a surprise because I bought it for my husband, but I get to play with it too, and of course I wanted to write a custom skill. Amazon being Amazon, the documentation is spread out over about 50 different pages, and there are a few trip-hazards along the path, so I hope you find this little guide helpful.

## The idea

I wasn’t at home when I decided to start writing my skill — I was with a group of friends for a New Year house party weekend. Matthew had said “Alexa, play Vengaboys” one too many times, and I stropped off upstairs to write something more amusing. More amusing than [Vengaboys](https://www.youtube.com/watch?v=4SQ650jVKYU) isn’t a very high bar.

I thought it might be fun to be able to ask Alexa who was in the house and have her recite all 15 of us. Well, the kids might think it was mildly amusing to hear her listing out their names.

## Getting started

I had a bit of a browse around the Amazon documentation and pretty quickly discovered that there are two parts to implementing my Alexa custom skill:

* an Alexa skill that defines how a speech utterance is converted into Intents

* a Lambda function that does the work for each Intent.

## The Lambda function

In the AWS Console you’ll find a Lambda section, and when you go to create a new Lambda function you’re offered a selection of blueprints to start from. Filtering these down to the ones tagged “alexa” quickly gave me a good starting point.

![](https://cdn-images-1.medium.com/max/5752/1*KoU97d38xIZfc3q2GPHRaA.png)

A lot of the blueprints are in JavaScript, but I’m much more comfortable in Python, so my eye was drawn to the last one in the list — alexa-skills-kit-color-expert-python.

![](https://cdn-images-1.medium.com/max/3760/1*9HAi4Xb6h6TKlxMIDEIekg.png)

This Lambda function is automatically set up to be triggered by Alexa Skills Kit.

![](https://cdn-images-1.medium.com/max/3764/1*JLaZvE_nopgMb1VMP1cD-Q.png)

You get the option to edit your Lambda function in-line. The sample code is well-commented so it’s pretty easy to find your way around.

When you start interacting with an Alexa skill, you start a “session” in which you can interact conversationally. But my super-simple “Who’s in the house” custom skill doesn’t need the complexity of interactions. All I wanted the function to do was to read out the list of names of the 15 guests. For simplicity I modified the “get_welcome_response()” function so that it ends the session straight away.

    def get_welcome_response():

    session_attributes = {}
     card_title = “Welcome”
     speech_output = “Jon, Anne, Anna, Dave, Alice, Oscar, Gemma, Ed, Emily, Lucy, Matthew, Charlotte, Alfred, Liz and Phil are here in the house”
     should_end_session = True
     return build_response(session_attributes, build_speechlet_response(
     card_title, speech_output, None, should_end_session))```

### Lambda execution role

You have to set up an execution role for your Lambda function, which dictates what permissions are available. Taking a cue from [this tutorial](https://developer.amazon.com/blogs/post/TxDJWS16KUPVKO/new-alexa-skills-kit-template-build-a-trivia-skill-in-under-an-hour) it seems that you need a “lambda_basic_execution” role. I created a new one with the default policy and called it myAlexaRole.

![](https://cdn-images-1.medium.com/max/3120/1*LeKbVVhSvUM5JhG4odUavA.png)

### Testing the Lambda function

You can test your function within the console by sending it a test event. Scroll down far enough in the list of input test events and you’ll find the Alexa events. For my purposes, testing with Alexa Start Session was enough.

![](https://cdn-images-1.medium.com/max/4484/1*8rLY8eFlGv6DQ6Xtu9kG6Q.png)

## The Alexa skill kit

Somewhat oddly, you don’t write your Alexa skill within the AWS console, but within the [Developer portal](http://developer.amazon.com). This whole portal was entirely new to me and took me longer than it should have done to realise that no amount of searching within the AWS console was going to locate Alexa skills.

![](https://cdn-images-1.medium.com/max/4916/1*9HjznnVEynpjfL2BvnZbEg.png)

We want the Alexa Skills Kit.

### Skills language

I haven’t experimented extensively, but I believe the language you pick needs to match the language that your Amazon Echo uses. So for me, it needed to be UK English rather than US English. I’ve spoken to friends who tell me they’ve wasted hours figuring out that this was the reason their skills wouldn’t work.

### Invocation name

I want to be able to say “Alexa, who is in the house?” and have her reel off the list of names. So here’s what I set up as the invocation name.

![](https://cdn-images-1.medium.com/max/3276/1*l_Ibg4GAba2iE4Bad2PY2w.png)

### Intents

I’m not sure I ever get as far as triggering any intents, because I’m ending the session as soon as I’ve issued the welcome message, but I found this simple intent configuration allowed things to work. The intent name can’t contain spaces.

![](https://cdn-images-1.medium.com/max/3280/1*By9HttDghuNGjfN2kOugPg.png)

### Sample utterances

I’m not at all sure I have this quite right, as my sample utterances don’t actually work when I run my skill live, but the following was at least accepted. Note that the first word in the sample utterance has to match the intent name otherwise you’ll get an error.

![](https://cdn-images-1.medium.com/max/3436/1*EMt3M5S5INfcNoG_MiKTyw.png)

### Testing the skill

You can test the skill locally by typing in the utterance and seeing what comes back.

![](https://cdn-images-1.medium.com/max/3388/1*dJQQscvgzJCUIqaRZCtKqg.png)

## Running the skill on Amazon Echo

I have a Lambda function, and I have an Alexa Skill Kit that triggers it. Now all I have to do is try it out on an Echo.

### Disappointment

Remember that I’m at a house party for New Year? It seems there’s no convenient way to share your unpublished Skill with anyone else. We could have jumped through a bunch of hoops to reconfigure Jon’s Echo to use my account, and then reconfigure it back again later. But we had better things to do like playing [CodeNames](https://en.wikipedia.org/wiki/Codenames_(board_game)) and eating enough to fuel a small army. So, disappointingly the kids never got to ask Alexa who was in the house.

### Disappointment part 2

New Year! Hurray! And then we came home to our own Amazon Echo. I modified my Lambda function to reel off a very much shorted list of names. I checked that my skill was enabled for testing…

![](https://cdn-images-1.medium.com/max/4104/1*-qagUbok5g6v6eC7d_9wpg.png)

…and then spoke the magic words, “Alexa, who is in the house”.

Nope.

She had no clue what I was talking about.

I looked at my Alexa web page. There it was, under “Your Skills”.

![](https://cdn-images-1.medium.com/max/5136/1*gM5KMzDIS5KY0L1cDpp2Vw.png)

It’s there, it’s enabled…why doesn’t it work?

![](https://cdn-images-1.medium.com/max/5252/1*LzwpaEFB-p6aad2zaeGXNQ.png)

I wondered if Phil could see my skill, since we share an Amazon household. And he had the brainwave that perhaps Alexa was using the wrong profile.

Alexa has profiles? Who knew?

## Alexa has profiles

The second-to-last item under “Things to try” is “Switch between profiles”. And yes, Alexa was using Phil’s profile rather than mine, and couldn’t see my (in-development, unpublished) skill.

Amazon, if you’re reading this, perhaps you could make it a little bit clearer, when you’re looking at Your Skills in the web app, that that has absolutely not bearing on what’s happening if Alexa isn’t using your profile.

## Success!!

<iframe src="https://medium.com/media/dadb01f26f641baad72c095a91a96e9e" frameborder=0></iframe>

## What’s next?

I think my next step will be to add some intents so that you can tell Alexa when people arrive and leave. She’ll need to store that somewhere, so I’ll need to add some storage of state. And then I might build something a bit more useful!

*If you liked this article, please hit the Recommend button so that other people might find it more easily and write their own Alexa skills. Happy new year!*

**Next up — here’s [Part 2 of my Alexa journey](https://hackernoon.com/my-alexa-skill-with-storage-5adb1d097b88#.od5suf6rg), where I add database storage**

*Did you REALLY like this article? I’m considering writing a book about developing for Alexa. You can encourage me to do that by [signing up here](https://leanpub.com/adventureswithalexa). Thank you! ❤︎*

![](https://cdn-images-1.medium.com/max/2272/1*0hqOaABQ7XGPT-OYNgiUBg.png)

![](https://cdn-images-1.medium.com/max/2272/1*Vgw1jkA6hgnvwzTsfMlnpg.png)

![](https://cdn-images-1.medium.com/max/2272/1*gKBpq1ruUi0FVK2UM_I4tQ.png)
> [Hacker Noon](http://bit.ly/Hackernoon) is how hackers start their afternoons. We’re a part of the [@AMI](http://bit.ly/atAMIatAMI) family. We are now [accepting submissions](http://bit.ly/hackernoonsubmission) and happy to [discuss advertising & sponsorship](mailto:partners@amipublications.com) opportunities.
> If you enjoyed this story, we recommend reading our [latest tech stories](http://bit.ly/hackernoonlatestt) and [trending tech stories](https://hackernoon.com/trending). Until next time, don’t take the realities of the world for granted!

![](https://cdn-images-1.medium.com/max/30000/1*35tCjoPcvq6LbB3I6Wegqw.jpeg)
