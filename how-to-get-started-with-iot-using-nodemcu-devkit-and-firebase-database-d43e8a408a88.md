
# How to get started with IoT using NodeMCU Devkit and Firebase database

Photo by Tim Käbel on Unsplash
> # ***“The Internet will disappear. There will be so many IP addresses, so many devices, sensors, things that you are wearing, things that you are interacting with, that you won’t even sense it. It will be part of your presence all the time. Imagine you walk into a room, and the room is dynamic. And with your permission and all of that, you are interacting with the things going on in the room.”***

Nowadays many devices that we use day to day are connected to the internet like Television, smart speakers, refrigerators, etc. These devices extend their primary functions which allows them to interact with other devices on the internet and to be controlled remotely.

You can build your own IoT devices using some sensors and microcontrollers. There are many development boards that will help you get started with IoT like Arduino, NodeMCU, Raspberry Pi, etc. You can automate your home by building from these devices.

In this post, we will be using NodeMCU devkit and Firebase for turning on and off LED remotely. NodeMCU devkit and Firebase are the best combinations to get started with building some IoT projects. NodeMCU is cheap and has built-in wifi for internet connectivity, and the Firebase free plan is more than enough.

## Setting up Development Environment

1. We will be using Arduino IDE for writing code and we will flash the code to the device. Download the latest version of the IDE [here](https://www.arduino.cc/en/main/software).

2. Since we are using NodeMCU which is not officially supported by Arduino IDE, we have to add the JSON file of the device. In Arduino IDE add this URL in
> Open File > Preferences > Additional Board Manager URLs
> [http://arduino.esp8266.com/stable/package_esp8266com_index.json](http://arduino.esp8266.com/stable/package_esp8266com_index.json)

3. Select your Board from
> Tools > Board > NodeMCU 1.o

4. To use firebase database in NodeMCU you need to download the firebase-arduino library which abstracts the REST API of the firebase. [Download firebase-arduino here](https://github.com/FirebaseExtended/firebase-arduino.git).

5. Include the downloaded zip file on Arduino IDE.
> Sketch > Include library > Add .zip > Select zip file

6. You also need to install the ArduinoJson library which can be downloaded from Arduino IDE itself.

Note: The library version should not be 6.x.x — use the latest 5.x.x
> Sketch > Include library > Manage Libraries > Search for ArduinoJson by Benoit Blanchon

## Setting up Firebase Database

7. Create a new firebase project from the [console ](https://console.firebase.google.com/)and head towards the database section. Select the firebase real-time database.

8. Copy the database secret for authentication from Settings Panel > Service accounts.

![Database secret](https://cdn-images-1.medium.com/max/2184/1*PQUqPFsKYw4XFWX3cFV-LA.png)*Database secret*

9. Add a led node to the firebase database. This value will decide whether to turn on or off the LED.

![](https://cdn-images-1.medium.com/max/2000/1*cpN78XVvwFe6O0AcoCBHiA.png)

## Configuring Arduino IDE and firebase database to work together

Now that all the setup procedures are done let’s start coding.

You need to create a macro for your database URL and firebase secret which you had copied in Step 8.
> #define FIREBASE_HOST “yourfirebasedatabase.firebaseio.com”
> #define FIREBASE_AUTH “*****”

For simplicity, we will write a simple code for turning on and off LED remotely

<iframe src="https://medium.com/media/bf28983afe56ae587216d41861ddc19c" frameborder=0></iframe>

10. The positive of the LED should be connected to the D1 pin and negative pin to the ground pin of NodeMCU.

![](https://cdn-images-1.medium.com/max/2000/1*lTdcYh-7O5uECpw8MczYow.png)

11. Upload your code from Arduino IDE.
> Sketch > Upload

12. Now try changing the database value to true and false. The led should now start turn on and off. Additionally, you can extend this project by creating a web app that will toggle the LED instead of manually changing the value in the database.

So now that you understand the basics of how to go about connecting NodeMCU to the internet and controlling it remotely, start hacking some new projects with it.
