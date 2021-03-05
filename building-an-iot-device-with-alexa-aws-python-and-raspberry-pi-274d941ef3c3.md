
# Building an IoT device with Alexa, AWS, Python and Raspberry Pi

The final product

I recently used AWS and a Raspberry Pi to create my own smart lights. In this series, I will guide you through setting up a strand of LED lights to respond to Amazon Alexa. I will mainly be using Python and AWS. To give credit where credit is due, as well as, to provide some great resources, I primarily used [Jay Proulx’s tutorial](https://medium.com/@jay_proulx/headless-raspberry-pi-zero-w-setup-with-ssh-and-wi-fi-8ddd8c4d2742) on setting up the Pi and [David Ordnung’s tutorial](https://dordnung.de/raspberrypi-ledstrip/) to set up the wiring for the LEDs to the Pi. I highly recommend referring to these and I will only briefly add some notes, as they have very detailed documentation.

### Background

In college I acquired this painting of a sea port with translucent windows for lights to shine through. After using a series (no pun intended) of LED lights and cheap controllers that would break, I turned to my friends and said “I’m tired of this, I can build a better solution.” Here is a bit of my experience set up in a way that any level of software developer can follow. By the end of this tutorial you will be able to turn on, off and dim lights with Alexa. Perhaps you’ll even add your own functionality afterwards. Please note all, commands will be for a Linux system.

The article will be broken into the following components:

### Part I: Initial set up and the Alexa Skill Kit

### [Part II: Setting up AWS IAM, IoT and Lambda](https://medium.com/@altonelli/building-an-iot-device-with-alexa-aws-python-and-raspberry-pi-part-ii-8ad84f24a3ee)

### [Part III: Setting up the Raspberry Pi and Lights](https://medium.com/@altonelli/building-an-iot-device-with-alexa-aws-python-and-raspberry-pi-part-iii-c1d383aa88c5)

### [Part IV: Review of code](https://medium.com/@arthurltonelli/building-an-iot-device-with-alexa-aws-python-and-raspberry-pi-part-iv-8725923daad0)

## What you will need

* [Raspberry Pi Zero W with Rasperian installed and a 5 V power supply](https://www.amazon.com/CanaKit-Raspberry-Wireless-Starter-Official/dp/B06XJQV162/ref=sr_1_4?s=electronics&ie=UTF8&qid=1522387067&sr=1-4&keywords=Raspberry+Pi+Zero+W)

* [A strand of SMD5050 LED RGB Lights and 12V power supply](https://www.amazon.com/Sunnest-Waterproof-300leds-Flexible-Controller/dp/B07571JKPC/ref=sr_1_9?ie=UTF8&qid=1522387239&sr=8-9&keywords=led+lights+5050)

* Three N-Channel MOSFETS [(I used P30N06LE**)](https://www.amazon.com/N-Channel-Power-Mosfet-RFP30N06LE-220/dp/B071Z98SRG/ref=sr_1_1?ie=UTF8&qid=1522387353&sr=8-1&keywords=P30N06LE)**

* [A breadboard and jumper wires](https://www.amazon.com/eBoot-Prototype-Solderless-Breadboard-Adhesive/dp/B06XKLRYRM/ref=sr_1_48?ie=UTF8&qid=1522387492&sr=8-48&keywords=breadboard+and+jumper+cables) (PCB board and solder if you want to make it more permanent after breadboarding)

* An Amazon Echo Device

* An Amazon Developer account (preferably made under the same Amazon account tied to your Echo)

* A copy of the code, which can be cloned from [this repo](https://github.com/altonelli/my_painting.git) or follow the code below.

### Initial set up and cloning the repository

The following is the set up for both your main development computer and the Raspberry Pi.

    $ cd <your_dev_dir>
    $ git clone [https://github.com/altonelli/my_painting.git](https://github.com/altonelli/alexa_skills.git)

After cloning the repo, touch a .env file and edit it.

    $ cd my_painting
    $ touch .env
    $ vi .env

Edit your .env should look like this:

    #General AWS credentials for Pi
    export AWS_IOT_CERTIFICATE_FILENAME="~/<path_to_credentials>/XXXXX-certificate.pem.crt"
    export AWS_IOT_PRIVATE_KEY_FILENAME="~/<path_to_credentials>/XXXXX-private.pem.key"
    export AWS_IOT_PUBLIC_KEY_FILENAME="~/<path_to_credentials>/XXXXX-public.pem.key"
    export AWS_IOT_ROOT_CA_FILENAME="~/<path_to_credentials>/root-ca.pem"
    #IoT Information
    export AWS_IOT_MQTT_HOST="XXXXX.iot.us-east-1.amazonaws.com"
    export AWS_IOT_MQTT_PORT=8883
    export AWS_IOT_MQTT_PORT_UPDATE=8443
    export AWS_IOT_MQTT_CLIENT_ID="my_painting"
    export AWS_IOT_MY_THING_NAME="my_painting"
    #Alexa Skill Information
    export AWS_ALEXA_SKILLS_KIT_ID="amzn1.ask.skill.XXXXX"

You may also want to set up a virtual environment, particularly during set up on the Raspberry Pi.

## The Alexa Interaction Model

Let’s get started. Setting up the interface for an Alexa skill is surprisingly simple. It is essentially a JSON file, though I will go through some of the details of what it all means.

### Create the Skill

From the [Alexa Skill console](https://developer.amazon.com/alexa/console/ask), there will be a few pages to start creating the skill. Select “Create Skill”, name the skill and select a language, select “Custom” and finally “Create skill”. Since this the lights will be lighting my painting, I gave it the ever so creative name “MyPainting”, since the lights are for my painting.

![](https://cdn-images-1.medium.com/max/3828/1*xkQre2La1-FTgJndDO6aNA.png)

![](https://cdn-images-1.medium.com/max/5488/1*2JiTPo_g-O4kRCjzPDyWbg.png)

![](https://cdn-images-1.medium.com/max/5472/1*Hz314NMZsP12PZ2cS3udKA.png)

Once created, note the skill ID, which will be something like “amzn1.ask.skill.xxxxx_amazon_skill_id_xxxxx” and add it to our .env file. We can also get this from the url of the skill editor.

### The Interaction Model

Next, go to the “JSON Editor” section in the left toolbar and upload or paste the [JSON file](https://github.com/altonelli/my_painting/blob/master/alexa_skill_kit/interaction_model.json) from the *alexa_skill_kit* directory of the repo to create the interaction model. Then select “Save Model”. Pretty easy!

![Paste or upload the JSON](https://cdn-images-1.medium.com/max/5496/1*nAhhJq75M7i7zpRl67yc3w.png)*Paste or upload the JSON*

We could also use the console’s interface to create the model but since it ends up being JSON anyways I prefer this method. I will highlight some of the key components of the interaction model below.

A typical interaction to the Alexa skill will be something like, “Alexa, tell *my painting* (*invocation name*) to *dim the lights* (*intent*) to *fifty* (*slot*) percent.”

* **Invocation Name**: The name or phrase that will trigger Alexa to have this skill handle our interaction. For consistency I chose “my painting” and will say “Alexa, tell *my painting* to…turn on my lights*.”*

* **Intents**: What we want interpreted from the skill. We have two custom intents *PowerStateIntent* and *BrightnessIntent*, for interpreting the power state (“on” or “off”) and brightness, respectively. We have a few default Amazon intents as well.

* **Slots**: Values we want to extract from a prompt. In the **Samples** listed, slots will be surrounded by {braces}. For every slot we list, we will also have to list the **Type** that can fill it.

* **Types**: A list of values that can fill a **Slot**. There are many [default Amazon types](https://developer.amazon.com/docs/custom-skills/slot-type-reference.html) you can chose from, like *AMAZON.DATE* for extracting dates and (in our case) *AMAZON.NUMBER* for numerical values. We can also create custom types, as with *PowerState*, by specifying the list of values, like *LIST_OF_POWER_STATES*. Custom types require additional specification in the JSON file (below).

* **Custom Types**: In addition to adding the **Type** to the **Slot**, since we are using a custom type, we also have to list them in the JSON. The *name* will be the same as we labeled under the **Slot** (*LIST_OF_POWER_STATES*). The *values* are a list of possible values for the type to represent, in this case “ON” and “OFF”. We could specify any synonyms for values. Since we are only going to be using the skill in development, we will be leaving the id blank.

* **Samples**: A list of sample utterances that users may say while using the skill. I listed out several for the *BrightnessIntent* as a user may vary their vocabulary. At the same time, we have to keep them unique to the intent. **We use these samples to shape how the user will interact with the skill.**

Thats a brief overview of what we have in our interaction model. For more details and additional items that can be added to the schema check out the [Skill Management API](https://developer.amazon.com/docs/smapi/interaction-model-schema.html).

### AWS Lambda Endpoint

So far we have written no code to build the interface for Alexa but we need something to handle the inputs of the interaction model! Don’t worry we’ll be coming back to this and create a Lambda function handle that logic. You may notice the “Endpoint” section in the toolbar. Once we have our Lambda function we’ll be selecting AWS Lambda and inputting the ARN. We’ll do that in the next section!

Helpful? Enjoy? Additional questions? Feedback in the form of 👏👏 or comments below always welcome!

Or, feel free to say hi at [hello@arthurtonelli.me](http://hello@arthurtonelli.me)!
