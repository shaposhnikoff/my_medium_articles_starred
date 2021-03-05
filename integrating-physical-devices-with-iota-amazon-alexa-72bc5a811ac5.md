
# Integrating physical devices with IOTA — Amazon Alexa

The 20th part in a series of beginner tutorials on integrating physical devices with the IOTA protocol

![](https://cdn-images-1.medium.com/max/4394/1*0Mare4fWGFLmIUbbMsiAYA.png)

*Alexa, Please send fifty IOTA’s to Alice
Alexa, What is my IOTA balance?
Alexa, What is the current IOTA transaction throughput?*

These are some of the cool things you could teach Alexa to do after completing this tutorial…

## Introduction

This is the 20th part in a series of beginner tutorials where we explore integrating physical devices with the IOTA protocol. In this tutorial we will look at integrating the [Amazon Alexa](https://en.wikipedia.org/wiki/Amazon_Alexa) virtual assistant with the IOTA protocol. At the same time, we will also get an introduction to the concept of cloud computing as our Python code for this project will be hosted as an [AWS Lambda function](https://aws.amazon.com/lambda/).

## The Use Case

Over time our hotel owner have ended up with multiple IOTA addresses where he receives funds from various IOTA powered devices and services. Now and then he needs to check the balance for these addresses. Only problem is that opening his Trinity wallet or checking [thetangle.org](https://thetangle.org/) takes him away from other tasks he might be doing at the same time. What if he could use some type of voice activated assistant to help him out? He could then get the information he needs, while at the same time stay focused on the task at hand.

Let’s see if we can help him out..

## What is Amazon Alexa?

Amazon Alexa is a virtual voice assistant app. developed by Amazon. The Alexa app. comes embedded in many different Alexa enabled devices such as smart speakers, smart TV’s, smart HI-FI systems, smart watches etc. You may even have Alexa embedded in your car in case you haven’t noticed.

![](https://cdn-images-1.medium.com/max/2000/0*KGtb6b2Lwr3HTUU1.jpg)

*Note!
Notice that you don’t need an Alexa enabled device to talk to Alexa. You can simply install the Amazon Alexa app. on your IOS or Android smartphone.*

## What are Alexa Skills?

Although Alexa can do a lot of things out-of-the box.., some times we need to teach her new “skills” that she does not already know how to do. And this is exactly what we will do in this tutorial.

At their simplest, Alexa skills are like voice activated smartphone apps. They are created by third party developers (like us). Some Alexa Skills work entirely on their own, while others interact with something else, such as an online service like Spotify, a smart home product like a Roomba robotic vacuum cleaner, or as in our case, the IOTA tangle.

## What is AWS Lambda?

AWS Lambda is a an event-driven, serverless computing platform provided by Amazon as a part of Amazon Web Services (AWS). AWS Lambda allows us to host and run the Python code required by our new Alexa Skill. The Python code (with all its dependencies/libraries) is typically packaged as a .zip file and uploaded to AWS in the form of an AWS Lambda function. We can now trigger the Lambda function (Python code) based on the response from some other AWS service, such as Alexa.

You can create and host AWS Lambda functions in the [AWS Management Console](https://aws.amazon.com/console/) for free by signing up for an AWS free tier account [here](https://aws.amazon.com/account/)

*Note!
AWS Lambda functions can be written different programming languages, such as Node.js, Java and Go, in case you prefer to write your Lambda code in one of these languages. In my example, I’m decided to go with Python as you may already be familiar with the PyOTA library from my previous tutorials.*

## Step-by-step implementation

When developing a new Alexa Skill we typically start by creating a new *Alexa Skill* in the [*Alexa Developer Console](https://developer.amazon.com/alexa/console/ask)*. Next, we define how we will interact with Alexa so that she understands our intent during the conversation. Next, we take the Python code where we have all the functions needed for our new Alexa Skill and upload the code as a hosted *AWS Lambda function *in the Amazon Web Services cloud. Next, we “connect” our Alexa Skill to the Lambda function so that Alexa know what code and functions to run when we interact with her. And finally, we test our new *Alexa Skill*.

Let’s take them step-by-step..

### Step 1 — Create a new Alexa Skill

Let’s start by creating a new Alexa Skill in the Alexa Developer Console

To create a new Alexa skill you first need to sign up for a free Alexa developer account. You will find a link to the Alexa Developer Console [here](https://developer.amazon.com/alexa/console/ask).

After signing up and loging in you should now see the main page of the Alexa Developer Console.

Lets start by creating a new Alexa skill using the “Create Skill” button.

![](https://cdn-images-1.medium.com/max/2000/1*S3Wfj7qufuUPicBRO8IYGg.png)

Next, we need to give our new skill a name, a default language, a model and a hosting method.

Let’s call the new skill **iota_balance** with default language **English (US)**. As model, select **Custom**, and as hosting method, select **Provision your own** before pushing the “Create Skill” button

Next, we need to select a template for our new skill.

Let’s select the **Hello World Skill** template before pushing the “Continue with template” button.

### Step 2 — Specify invocation, intents and utterances

Next, we need to teach Alexa how we will invoke and interact with her new skill during a typical conversation. This is done using *Invocations*, *Intents* and *Utterances*.

**Invocation**
The *Invocation* let’s Alexa know what skill to perform when we call for her. In my example i decided to go with the phrase: **get iota balance
**So if I now want Alexa to perform this particular skill, i would simply say: ***Alexa, get iota balance***

![](https://cdn-images-1.medium.com/max/2000/1*Mm29c9yESctj0ws2TLIMUg.png)

**Intents
**The *Intents* section specifies various “functions” within Alexa’s new skill that i might want to invoke during our conversation. In my example I have two main intents:

1. Get the balance of my personal IOTA address

1. Get the balance of the Hotel IOTA address

In my example I named these intents: *PersonalBalance* and *HotelBalance*

![](https://cdn-images-1.medium.com/max/2000/1*fGceyvOfr68EKi6x7H1IPw.png)

*Note!
Notice that there are some Buildt-In Intents (CancelIntent, HelpIntent etc.) that came with the **Hello World Skill** template. These are typical intents you would invoke in case you want to exit the skill, get some more information about the skill etc. For now, just leave them within the list of Intents.*

**Utterances***
*For each Intent you also need to specify some typical phrases you would use to call a particular Intent. These phrases are called *Utterances* and are associated with there respective Intent. For instance, if I want to invoke the *PersonalBalance* Intent, I might say something like “Get my personal balance” or “What is my personal balance”.

![](https://cdn-images-1.medium.com/max/2000/1*49qX-O-BoL__2K4BGh97Ag.png)

*Note!
By using some clever machine learning algorithms, Alexa normally understands your intent even if you don’t say the sample utterances directly as spelled out. It is often enough to just include one or two key words in the sentence.*

### Step 3 — Creating and uploading the AWS Lambda function for our Alexa Skill

Next, we will create a new AWS lambda function in AWS and upload the Python code to be used by our new Alexa Skill.

Start by going to the [AWS Management Console](https://aws.amazon.com/console/) and find the Lambda service.

After selecting the Lambda service, select functions and push the “Create function button”

Select the “Author from Scratch” template and give your new Lambda function a name. In my example I used the name *iota_balance*. Finally, select *Python 3.8* as your programming language and press the “Create function” button.

When the new Lambda function opens, next thing we need to do is to specify that the function will be triggered by an Alexa Skill. Go to the *Designer* panel and push the “Add trigger” button. Then from the “Trigger configuration” list, select “Alexa Skills Kit”. When asked for your *Skill ID*, paste the Skill ID for the Alexa Skill created in Step 1.

*Note!
You will find the Skill ID for your Alexa Skill by selecting Endpoint in your Alexa Skill.*

![](https://cdn-images-1.medium.com/max/3114/1*fJNwb0xC5ZvLSLSqmtj5sw.png)

Now that we have created our new Lambda function we can start uploading the Python code that will be associated with the function.

As mentioned previously, when hosting Python code as a Lambda function it is important that we also include and upload any non-standard Python libraries required by our function. In our case, the PyOTA library is required so we have to make sure we also include the files from this library. The way we do this in practice is that we package all our required files in one .zip file before uploading the .zip file to the AWS Lambda function.

*Important!
I have prepared a .zip file you can use in your Lambda function to give you a head start while creating your first Alexa Skill. The .zip file includes the lambda_function.py file used by Alexa together with the PyOTA library required when interacting with the IOTA tangle. You can find and download the .zip file (get_iota_balance.zip) in this [Github repository](https://github.com/huggre/amazon_alexa).*

Now that we have our .zip file, go to your new Lambda function in the AWS Management Console, select the *Actions* drop-down menu in the *Function code* panel and then select “Upload a .zip file”. You may now select and upload your .zip file.

### Step 4 — Connecting your Alexa Skill with the Lambda function

Last thing we need to do before we can start testing our new Alexa Skill is to “connect” our Alexa Skill to the Lambda function so that Alexa knows what Lambda function to use when performing her new Skill.

Start by copying the ARN id you see en the upper right corner when inside the Lambda function

![](https://cdn-images-1.medium.com/max/2000/1*oRLQBZBDNdI7CObhraugpQ.png)

Next, go to your Alexa Skill, select *Endpoint* and paste the id in the *Default Region* field

![](https://cdn-images-1.medium.com/max/2000/1*aMPA0lovOkNTRX9UJPMN7Q.png)

That’s it.., hit the “Save Model” and “Build Model” buttons in your Alexa Skill and we can start testing our new Alexa Skill.

### Step 4 — Testing our Alexa Skill

To test our new Alexa Skill, select “Test” from the Alexa Skill menu and then select *Development* from the “Skill testing is enabled in:” drop-down menu.

You can now start typing text or talking to Alexa by pushing the microphone icon in the Alexa Simulator.

If everything is working correctly you should see (and hear) the dialog going something like this:

![](https://cdn-images-1.medium.com/max/2000/1*P6Kqn0Zf3lh2xGTKGe3pRQ.png)

## The Python Code

Let’s have a quick look at the Lambda function (Python code) for this project and how it works.

When looking at the code you will see that we have some standard functions required for interacting with Alexa. The most important being the *on_intent()* function. Here we define the code to be executed as we call for each individual Intent as described in Step 2.

So in my example, if you say something like “Get my personal balance” or “What is my personal balance”, the Alexa Skill knows that the Intent we want to execute is the *PersonalBalance *intent. Next, we simply call a new custom function (*get_personal_balance()) *where we have all the code for interacting with the IOTA tangle together with the returning responses from Alexa.

That’s it, and here is the code..

<iframe src="https://medium.com/media/9e817ac68e2f8517c02b8992d036ba1e" frameborder=0></iframe>

## Creating your version of the project

The simplest way to get you started with creating your own version of this project (such as changing the IOTA addresses used for checking balances) is to simply modify the *lambda_function.py* file inside the .zip file and upload the modified .zip file to your existing Lambda function.

## Contributions

If you would like to make any contributions to this tutorial you will find a Github repository [here](https://github.com/huggre/amazon_alexa)

## Donations

If you like this tutorial and want me to continue making others feel free to make a small donation to the IOTA address below.

![](https://cdn-images-1.medium.com/max/2000/0*JyR_ttsrPttUHye6.png)

NYZBHOVSMDWWABXSACAJTTWJOQRPVVAWLBSFQVSJSWWBJJLLSQKNZFC9XCRPQSVFQZPBJCJRANNPVMMEZQJRQSVVGZ

### Also, Read

* The Best [Crypto Trading Bot](https://medium.com/coinmonks/crypto-trading-bot-c2ffce8acb2a)

* [Crypto Copy Trading Platforms](https://medium.com/coinmonks/top-10-crypto-copy-trading-platforms-for-beginners-d0c37c7d698c)

* The Best [Crypto Tax Software](https://medium.com/coinmonks/best-crypto-tax-tool-for-my-money-72d4b430816b)

* [Best Crypto Trading Platforms](https://medium.com/coinmonks/the-best-crypto-trading-platforms-in-2020-the-definitive-guide-updated-c72f8b874555)

* Best [Crypto Lending Platforms](https://medium.com/coinmonks/top-5-crypto-lending-platforms-in-2020-that-you-need-to-know-a1b675cec3fa)

* [Best Blockchain Analysis Tools](https://bitquery.io/blog/best-blockchain-analysis-tools-and-software)

* [Crypto arbitrage](https://medium.com/coinmonks/crypto-arbitrage-guide-how-to-make-money-as-a-beginner-62bfe5c868f6) guide: How to make money as a beginner

* Best [Crypto Charting Tool](https://medium.com/coinmonks/what-are-the-best-charting-platforms-for-cryptocurrency-trading-85aade584d80)

* [Ledger vs Trezor](https://medium.com/coinmonks/ledger-vs-trezor-best-hardware-wallet-to-secure-cryptocurrency-22c7a3fd391e)

* What are the [best books to learn about Bitcoin](https://medium.com/coinmonks/what-are-the-best-books-to-learn-bitcoin-409aeb9aff4b)?

* [3Commas Review](https://medium.com/coinmonks/3commas-review-an-excellent-crypto-trading-bot-2020-1313a58bec92)

* [AAX Exchange Review](https://medium.com/coinmonks/aax-exchange-review-2021-67c5ea09330c) | Referral Code, Trading Fee, Pros and Cons

* [Deribit Review](https://medium.com/coinmonks/deribit-review-options-fees-apis-and-testnet-2ca16c4bbdb2) | Options, Fees, APIs and Testnet

* [FTX Crypto Exchange Review](https://medium.com/coinmonks/ftx-crypto-exchange-review-53664ac1198f)

* [NGRAVE ZERO review](https://medium.com/coinmonks/ngrave-zero-review-c465cf8307fc)

* [Bybit Exchange Review](https://medium.com/coinmonks/bybit-exchange-review-dbd570019b71)

* [3Commas vs Cryptohopper](https://medium.com/coinmonks/cryptohopper-vs-3commas-vs-shrimpy-a2c16095b8fe)

* The Best Bitcoin [Hardware wallet](https://medium.com/coinmonks/the-best-cryptocurrency-hardware-wallets-of-2020-e28b1c124069?source=friends_link&sk=324dd9ff8556ab578d71e7ad7658ad7c)

* Best [monero wallet](https://blog.coincodecap.com/best-monero-wallets)

* [ledger nano s vs x](https://blog.coincodecap.com/ledger-nano-s-vs-x)

* [Bitsgap vs 3Commas vs Quadency](https://blog.coincodecap.com/bitsgap-3commas-quadency)

* [Ledger Nano S vs Trezor one vs Trezor T vs Ledger Nano X](https://blog.coincodecap.com/ledger-nano-s-vs-trezor-one-ledger-nano-x-trezor-t)

* [BlockFi vs Celsius](https://medium.com/coinmonks/blockfi-vs-celsius-vs-hodlnaut-8a1cc8c26630) vs Hodlnaut

* [Bitsgap review](https://medium.com/coinmonks/bitsgap-review-a-crypto-trading-bot-that-makes-easy-money-a5d88a336df2) — A Crypto Trading Bot That Makes Easy Money

* [Quadency Review](https://medium.com/coinmonks/quadency-review-a-crypto-trading-automation-platform-3068eaa374e1)- A Crypto Trading Bot Made For Professionals

* [PrimeXBT Review](https://medium.com/coinmonks/primexbt-review-88e0815be858) | Leverage Trading, Fee and Covesting

* [Ellipal Titan Review](https://medium.com/coinmonks/ellipal-titan-review-85e9071dd029)

* [SecuX Stone Review](https://blog.coincodecap.com/secux-stone-hardware-wallet-review)

* [BlockFi Review](https://medium.com/coinmonks/blockfi-review-53096053c097) | Earn up to 8.6% interests on your Crypto
