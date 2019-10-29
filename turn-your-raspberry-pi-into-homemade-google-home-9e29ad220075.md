
# Turn your Raspberry Pi into homemade Google Home

Source

[Google Home](https://madeby.google.com/intl/en_us/home/) is a beautiful device with built-in [Google Assistant](https://assistant.google.com/) — A state of the art digital personal assistant by Google. — which you can place anywhere at your home and it will do some amazing things for you. It will save your reminders, shopping lists, notes and most importantly answers your questions and queries based on the context of the conversations.

In this article, you are going to learn to turn your Raspberry Pi into homemade Google Home device which is,

* Powered by Google Assistant.

* Voice activated. No need to press any button, just say “Ok Google” or “Hey Google” and ask your question.

* There will be a LED indicator which will stay on whenever the conversation between the user and the Google Assistant it in progress.

* It can initialize on boot so no need to login and run the script from terminal after reboots.

So, let’s get started.
[**The Chatbot Conference**
*The Chatbot Conference On September 12, Chatbot's Life, will host our first Chatbot Conference in San Francisco. The…*www.eventbrite.com](https://www.eventbrite.com/e/the-chatbot-conference-tickets-34868758395?aff=CBL)

## What things will you need?

* **Raspberry Pi **model 2 or 3.

* **MicroSD card** with Raspbian on it (Minimum 8GB recommended).

* **Power supply** to feed your raspberry pi. (Any USB mobile charger with minimum 5V, 2A output will work.)

* USB mic (As Raspberry Pi doesn’t have an inbuilt mic. I used MI-305).

* A **speaker**.

* A **LED**.

* A **couple of wires** to connect LED.

Once you have all these things, login to Raspbian desktop and go to the following steps one by one.

## Step -1: Setting up USB mic.

* Raspberry Pi doesn’t have inbuilt microphones. If you want to record audio, you need to attach a USB microphone.

* Plug your USB mic into any of the USB slots of your Raspberry Pi.

* Go to the terminal and type following command.

<iframe src="https://medium.com/media/2cdfb60595b299bdab576e26283524bc" frameborder=0></iframe>

* This command will list all the available audio record devices. You should get below output.

<iframe src="https://medium.com/media/81c3a8ba17320d6bc9b343329ffe12c1" frameborder=0></iframe>

As you can see your USB device is attached to card 1 and the device id is 0. Raspberry Pi recognizes card 0 as the internal sound card (which is bcm2835) and other external sound cards as external sound cards.

* Now, let’s change the audio configs. Type below command to edit the asound.conf file.

<iframe src="https://medium.com/media/c88e6b542f62b79972617558c382015b" frameborder=0></iframe>

* Add below lines in the file. Then press Ctrl+X and after that Y to save the file.

<iframe src="https://medium.com/media/cd9c5e202df9bc622c9810f037e0f75a" frameborder=0></iframe>

This will set your external mic (see pcm.mic) as the audio capture device (see in pcm!.default) and your inbuilt sound card (card 0) as the speaker device.

* Create a new file named .asoundrc in the home directory (/home/pi) by issuing following command and paste above configurations (which you added in /etc/asound.conf file.) to this file.

<iframe src="https://medium.com/media/33fa4a422ed29ca87eb213abdac4a59f" frameborder=0></iframe>

## Step -2: Setting up your speaker output.

* Connect your speaker to 3.5mm headphone jack of the Raspberry Pi.

* Run below command to open raspberry pi configuration screen.

<iframe src="https://medium.com/media/d942bd4a7e554dc3c4f12eeb5a319ecc" frameborder=0></iframe>

* Go to Advanced Options > Audio and select the desired output device.

## Step -3: Test the mic and speakers.

* To test your speaker run below command in the terminal. This will play a test sound. Press Ctrl+C when done. If you are not able to hear the test sound check your speaker connection.

<iframe src="https://medium.com/media/b34cf469686dee3935ee8e2c1cc9faed" frameborder=0></iframe>

* To test your mic run following command. This will record a short audio clip. If you get any error check step 1 again.

<iframe src="https://medium.com/media/42f6d0a62ce1bdbac5ae453b6a6d6163" frameborder=0></iframe>

* Play the recorded audio and confirm everything works correctly by issuing following command.

<iframe src="https://medium.com/media/33fb777c9ba6d4fca59f78905abb9688" frameborder=0></iframe>

### Okay. Our hardware is set.

## Step -4: Download required packages and configure Python environment:

* First, update your operating system.

<iframe src="https://medium.com/media/59a57b9ad5e77df5f161c1a35152742a" frameborder=0></iframe>

* Run below command one by one in the terminal.

<iframe src="https://medium.com/media/df4723ea85dc996dbf4abf92f999388a" frameborder=0></iframe>

This will create Python 3 environment (As the Google Assistant library runs on Python 3.x only) in your raspberry pi and install required dependencies.

* Activate the python environment.

<iframe src="https://medium.com/media/7e2d479042339de62235d8bb41e4ad30" frameborder=0></iframe>

* Now, install the Google Assistant SDK package, which contains all the code required to get the Google Assistant running on the Raspberry Pi. It should download the Google Assistant Library and the demo.

<iframe src="https://medium.com/media/1344f4748b839ea876f0fa749354b2a7" frameborder=0></iframe>

## Step -5: Enabling the Google Assistant cloud project.

* Open the [Google Cloud Console](https://console.cloud.google.com/) and create a new project. (You can name it whatever you want.) The account with which you sign in will be used to send queries to Google Assistant and get your personalized response.

![](https://cdn-images-1.medium.com/max/2496/1*ck-H-eGdWm7D6GMIm6pf2A.png)

* Head over to [API manager](https://console.cloud.google.com/apis/api/embeddedassistant.googleapis.com/overview) and enable the Google Assistant API.

* - Make sure that you enable Web & App Activity, Device Information and Voice & Audio Activity in [Activity Controls](https://myaccount.google.com/activitycontrols) for the account.

* - Go to “[Credentials](https://console.cloud.google.com/apis/credentials)" and set up OAuth Content Screen.

![](https://cdn-images-1.medium.com/max/2920/1*CwpL3vUj32QxXMQvsZcknw.png)

* Go to “Credentials” tab and Create new OAuth client ID.

![](https://cdn-images-1.medium.com/max/2120/1*NorVhwktAyzUVcKte2cTXA.png)

* Select application type as “Other” and give the name of the key.

![](https://cdn-images-1.medium.com/max/2788/1*Cx5UFLe7XY_0pLP1lvTIzA.png)

* Download the JSON file that stores the OAuth key information and keep it safe.

![](https://cdn-images-1.medium.com/max/4692/1*M9Lv1VGG7zO2SzY8O03riQ.png)

## Step -6: Authenticating your Raspberry Pi.

* Install authorization tool by running below command.

<iframe src="https://medium.com/media/c397ac6ea20fa10c2586bb6ff52aa56c" frameborder=0></iframe>

* Run the tool by running following command. Make sure you provide correct path for the JSON file you downloaded in step 5.

<iframe src="https://medium.com/media/4b495b04dfb4bce94e1d463c5a0ddcc6" frameborder=0></iframe>

* It should display as shown below. Copy the URL and paste it into a browser (this can be done on your developmen

<iframe src="https://medium.com/media/68e689afe6f8db9918135057b32561ae" frameborder=0></iframe>

If instead, it displays: *InvalidGrantError* then an invalid code was entered. Try again.

## Step -7: Setting up the LED indicator.

* Connect your LED between GPIO pin 25 and ground.

* The idea here is simple. We are going to set the GPIO pin 25 as the output pin. Google Assistant SDK provides a callback *EventType.ON_CONVERSATION_TURN_STARTED* when the conversion with the Google Assistant begins. At that point, we are going to set the GPIO 25 to glow the LED. Whenever the conversation terminates *EventType.ON_CONVERSATION_TURN_FINISHED* callback will be received. At that point, we will reset the GPIO 25 to turn off the LED.

![](https://cdn-images-1.medium.com/max/2000/1*8mUGLoc-gwrf42BnCUDFEA.png)

## Step -8: Initialise on boot complete:

* Whenever your Raspberry Pi completes boot process, we will run a python script that will authenticate and initialize the Google Assistant on boot.

* First add RPi.GPIO package to add GPIO support using following command.

<iframe src="https://medium.com/media/7aaf45566467e3e8b271b3ea25d1591c" frameborder=0></iframe>

* Go to the user directory. Create new python file main.py.

<iframe src="https://medium.com/media/2489994babd4913acb0ea290c296b3df" frameborder=0></iframe>

* Write following script and save the file.

<iframe src="https://medium.com/media/ba786a9b2f4e45d6a3d14596dc0706f0" frameborder=0></iframe>

* Now create one shell script that will initialize and run the Google Assistant.

<iframe src="https://medium.com/media/1ffa081231f513da230dcc45e763b058" frameborder=0></iframe>

* Paste below lines into the file and save the file.

<iframe src="https://medium.com/media/c452d1b59891d187f875ff3ebb46c701" frameborder=0></iframe>

* Grant the execute permission.

<iframe src="https://medium.com/media/5ce95d4acabc8a8ea81e56599e197a7c" frameborder=0></iframe>

You can run *google-assistant-init.sh* to initiate the Google Assistant any time.

### Let’s see how you can start the Google Assistant while booting.

* To enable Google Assistant on Boot there are two ways. Let’s see each of them.

**1. Autostart with Pixel Desktop on Boot:**

* This will start the Google Assistant as soon as Pixel desktop boots up. Make sure you have “Desktop” boot selected in Raspberry Pi configurations.

* Type below command.

<iframe src="https://medium.com/media/d3e02481d377208dfb095a4727d1013d" frameborder=0></iframe>

* Add the following after *@xscreensaver -no-splash*

<iframe src="https://medium.com/media/b97ee062c7ece375c1cacba2f3bdcd43" frameborder=0></iframe>

* Save and exit by pressing “Ctrl+X” and then “Y”.

**2. Autostart with CLI on Boot:**

* This will start the Google Assistant if you have set CLI boot. Make sure you have “CLI” boot selected in Raspberry Pi configurations.

* Type below command.

<iframe src="https://medium.com/media/e4ed4026c53597f384fe6561270f83ba" frameborder=0></iframe>

* Add below line at the end of the file.

<iframe src="https://medium.com/media/dd6751df9da480f08c68e0604fe1607a" frameborder=0></iframe>

* Save and exit by pressing “Ctrl+X” and then “Y”.
> That’s all!!! You “Homemade Google Home” is now ready. Reboot the device and ask your first question to your Google Assistant.

<iframe src="https://medium.com/media/2e228ed10e54ec045b30cbe3115e3dc1" frameborder=0></iframe>

## Conclusion:

You can do many daily stuff with your Google Home. If you want to perform your custom tasks like turning off the light, opening the door, you can do it with integrating [Google Actions](https://developers.google.com/actions/) in your Google Assistant. If you have any trouble with starting the Google Assistant, leave a comment below. I will try to resolve them.

![](https://cdn-images-1.medium.com/max/2000/1*jQojfXB3QbnutOQB4zBhFA.png)

*~If you liked the article, click the 💚 below so more people can see it! Also, you can follow me on [Medium](https://medium.com/@kevalpatel2106) or on [My Blog](https://medium.com/@kevalpatel2106), so you get updates regarding my future articles!!~*

![](https://cdn-images-1.medium.com/max/2000/1*JKyJlywuCXZV3NQktGjAiw.gif)
