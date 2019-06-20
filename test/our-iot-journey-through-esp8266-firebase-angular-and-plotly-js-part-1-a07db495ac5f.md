Unknown markup type 10 { type: [33m10[39m, start: [33m158[39m, end: [33m162[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m73[39m, end: [33m82[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m87[39m, end: [33m100[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m72[39m, end: [33m80[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m84[39m, end: [33m97[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m27[39m, end: [33m31[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m135[39m, end: [33m143[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m43[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m99[39m, end: [33m114[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m129[39m, end: [33m157[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m43[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m235[39m, end: [33m259[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m323[39m, end: [33m336[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m349[39m, end: [33m362[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m62[39m, end: [33m71[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m111[39m, end: [33m114[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m140[39m, end: [33m151[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m164[39m, end: [33m174[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m47[39m, end: [33m67[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m43[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m61[39m, end: [33m74[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m44[39m, end: [33m51[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m56[39m, end: [33m64[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m68[39m, end: [33m74[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m201[39m, end: [33m221[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m406[39m, end: [33m412[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m536[39m, end: [33m563[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m588[39m, end: [33m608[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m15[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m113[39m, end: [33m128[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m114[39m, end: [33m126[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m141[39m, end: [33m152[39m }

# Post 1 of 3. Our IoT journey through ESP8266, Firebase and Plotly.js

What we will achieve… at the end of this 3-post serie

## TL;DR

This post stands in a serie of 3 as detailed below. These 3 posts reflect the Project Architecture we chose to achieve a simple luminosity logging/live plotting project. We hope they will **help developers or students discovering ESP8266 chip and Firebase platform**. Moreover, we believe that the solution developped here can be an interesting alternative to the comprehensive **Google, AWS and Azure IoT solutions** [[link](https://cloud.google.com/solutions/iot/), [link](https://docs.aws.amazon.com/fr_fr/iot/latest/developerguide/what-is-aws-iot.html) and [link](https://docs.microsoft.com/fr-fr/azure/iot-hub/iot-hub-device-management-overview)] when we have to **manage in a simple way only a few connected devices**. (For instance, we won’t set up a MQTT broker neither deal with a registry.)

![Project Architecture — Numbers 1, 2, 3 follow post numerotation](https://cdn-images-1.medium.com/max/2000/1*J0IGNrEAq3n8zxCZsLO5dQ.png)*Project Architecture — Numbers 1, 2, 3 follow post numerotation*

**Post 1** (this post): We investigate on how an **ESP8266** can regularly make acquisition of an analog data and push its 10-bit equivalent value to a **Firebase Realtime Database**. We finish with considerations on **ESP8266 power saving.**

**Post 2** [[link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-2-14b0609d3f5e)]: We write a **Firebase Cloud Function** in Typescript language to **timestamp **the data just pushed to Firebase Realtime Database.

**Post 3** [[link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-3-644048e90ca4)]: With a** **web app using **plotly.js library**, we lively plot in a browser the analog data versus time. The web app is hosted with **Firebase Hosting**.

***Post 1. An ESP8266 pushes luminosity measures to a Firebase Realtime Database.***

## General introduction

It’s been a while since we heard about the ESP8266 component for the first time, its low cost and its wifi feature. Willing to master it and having at last a little free time, we built a small **IoT (Internet of Things) project whose aim is to lively plot an analog data to a web app**. Which analog data? Hmmmh, let’s say luminosity in North of France!

## A) Hardware considerations

### A) 1) ESP8266: a short introduction

ESP8266 [[link](https://en.wikipedia.org/wiki/ESP8266)] is a **low cost and low power Wifi SoC** integrating a 32-bit Tensilica microcontroller running at 80 MHz. It is designed by ESPRESSIF chinese company and manufactured by companies such as ESPRESSIF, AI-THINKER or DOIT. Due to its embedded Wifi capability, it is widely used in IoT projects and products (in **Sonoff or Shelly switches **for instance : [[link](https://sonoff.itead.cc/en/products/sonoff/sonoff-basic), [link](https://shelly.cloud/)]). Until now, ESP8266 is declined in many versions but we use here the widespread one called ESP-12E [[link to specs as a pdf](https://www.kloppenborg.net/images/blog/esp8266/esp8266-esp12e-specs.pdf)].

*Note:* What do we mean by low cost ? We’ll use a **NodeMCU DEVKIT v1.0 **board integrating an ESP8266 and many items for development purpose (see a few lines below). It costs between $3 and $4 from classical chinese web sites. A device dedicated to production costs even less.

*Note:* An ESP8266 successor is ESP32 [[link](https://en.wikipedia.org/wiki/ESP32)]. Compared to ESP8266, everything is enhanced: speed, memory, peripherics, presence of BLE, memory encryption capability, etc. But it costs about twice.

Besides Wifi, ESP-12E is equipped with **all classical peripherals** : GPIOs, UART, a single ADC (Analog To Digital Converter), PWMs, I2C, SPI, and so on. It also comes with a 4 MiB SPI Flash memory which we’ll discuss later.

It is important to note that, though ESP-12E is 3.3 V powered, the ADC input (pin #2 on the pinout just below) voltage range is from 0 V to 1 V. The ADC is a 10-bit one so its quantum is about 1 mV (exactly: 1 V /(2¹⁰-1)).

![ESP-12E pinout [[link](http://ha2eqd.com/arduino/021_1st_ket_ESP_modulom/1st_ESP.html)]](https://cdn-images-1.medium.com/max/2000/1*WsCz-gCf0W2sr-mgGfTImg.jpeg)*ESP-12E pinout [[link](http://ha2eqd.com/arduino/021_1st_ket_ESP_modulom/1st_ESP.html)]*

Of course, you can’t use ESP-12E out of the box, a lot of things are missing (at least a supply). So read next paragraph…

### A) 2) NodeMCU DEVKIT v1.0 to immediately start development

Really soon (2014) after ESP8266 was released (2013) some people developped (in C and LUA languages) a **firmware called NodeMCU** [[link](https://en.wikipedia.org/wiki/NodeMCU)]. It allows to write scripts for ESP8266 in LUA language. A few months later, an **open-hardware platform **called NodeMCU DEVKIT v0.9** **was added to this project. Now outdated, we used in our project its successor **NodeMCU DEVKIT v1.0 **[[link to schematic as a pdf in github](https://github.com/nodemcu/nodemcu-devkit-v1.0/blob/master/NODEMCU_DEVKIT_V1.0.PDF)] on which we can find everything to immediately start IoT development:

1. An ESP-12E.

1. A USB to Serial Converter to communicate with a computer (for console and program flashing purposes); most of the time it’s a CP2102 chip but sometimes it’s a CH340. Installing a driver is sometimes needed.

1. A MicroUSB-B connector to connect our NodeMCU DEVKIT v1.0 to our computer during development/debug. You will need a cable.

1. A 3.3 V regulator to make 3.3 V from 5 V USB (typical reference: AMS1117).

1. A “flash” button (we didn’t have to use it) and a useful reset button.

1. An **active low** built-in blue LED, connected to D0 (GPIO16) (see pinout below, after NodeMCU DEVKIT v1.0 photo)

1. A voltage divider (not referenced on image below) whose role is explained just after.

1. …

![NodeMCU DEVKIT v1.0 [[link](https://github.com/nodemcu/nodemcu-devkit-v1.0/tree/master/Documents)]](https://cdn-images-1.medium.com/max/2048/1*pqruSzABNlOd60QRpDie9Q.jpeg)*NodeMCU DEVKIT v1.0 [[link](https://github.com/nodemcu/nodemcu-devkit-v1.0/tree/master/Documents)]*

![NodeMCU DEVKIT v1.0 pinout [[link](https://e.banana-pi.fr/esp8266/389-nodemcu-10-wifi-esp8266-esp-12e.html)]](https://cdn-images-1.medium.com/max/2000/1*9SeLUvl8eKo66k7ltcRfVg.jpeg)*NodeMCU DEVKIT v1.0 pinout [[link](https://e.banana-pi.fr/esp8266/389-nodemcu-10-wifi-esp8266-esp-12e.html)]*

On the above pinout, A0 is the (upper left) pin where you connect the analog input you want to measure. A0 is not directly connected to pin #2 of ESP-12E, there is a 100k/(100k+220k) = (1/3.2) **voltage divider** between them [[link to schematic as a pdf in github](https://github.com/nodemcu/nodemcu-devkit-v1.0/blob/master/NODEMCU_DEVKIT_V1.0.PDF), see part called “ADC”]. So, **on A0 of NodeMCU v1.0, the input voltage range can be from 0 V to 3.2 V**.

To conclude this hardware presentation, let’s say it is sometimes confusing to deal with the numerous NodeMCU DEVKIT versions and manufacturers. This post [[link](https://frightanic.com/iot/comparison-of-esp8266-nodemcu-development-boards/)] explains things well and also presents **ESP8266 alternative developpement boards** like **Adafruit Feather HUZZAH with ESP8266** (supplied by a 3.7 V LiPo battery or 5 V USB + power management included) or **WeMos mini**. It also gives links to at last buy our first NodeMCU DEVKIT v1.0!

## B) Software choice: Arduino language and environment

LUA is without doubt an interesting language but what if we could program ESP8266 with **a language and an IDE **we all already know, eliminating long learning hours ? Let’s say something like Arduino. This is possible thanks to “**Arduino core for ESP8266”**! And it has a fantastic network library!

*Note:* There are numerous other development options than NodeMCU and Arduino. See “SDKs” and “See also” parts in [[link](https://en.wikipedia.org/wiki/ESP8266)].

### B) 1) Installation

The official “Arduino core for ESP8266**” **github repo is here : [[link](https://github.com/esp8266/Arduino#installing-with-boards-manager)]. We follow the part called **Installing with Boards Manager**. At the end, when everything is installed and the NodeMCU DEVKIT v1.0 is selected as our development board, we get something like this:

![Arduino IDE after NodeMCU DEVKIT v1.0 is selected](https://cdn-images-1.medium.com/max/2000/1*9FyQ0JNIyGMDz9zDvmdB3A.png)*Arduino IDE after NodeMCU DEVKIT v1.0 is selected*

*Note:* There is no Arduino firmware flash to care about. And if there is already a firmware installed on our NodeMCU DEVKIT v1.0 when we get it, it will be deleted the first time we upload an Arduino program.

### B) 2) Time for “Hello, World!” program

In this **“Hello, World!” program** (“sketch” in Arduino style), we will make the NodeMCU DEVKIT v1.0 built-in blue **LED blink and print a message** **to Arduino IDE Serial Monitor** each time this LED is about to change its state. This program is inspired from:[[link](https://github.com/esp8266/Arduino/blob/master/libraries/esp8266/examples/Blink/Blink.ino)] (or we can reach it by: **Arduino_IDE>File>Examples>ESP8266>Blink**).

*Note:* Each time there is an action to perform, it is preceeded by a lowercase letter like **a. b. c. **etc.

**a.** With **Arduino_IDE>File>New**, we create a new program and copy/paste the following code. We save it.

<iframe src="https://medium.com/media/66d09c3741f423d982f052ab43578b19" frameborder=0></iframe>

**b.** If not done yet, we connect via USB our NodeMCU DEVKIT v1.0 to our computer.

**c.** With **Arduino_IDE>Tools>Port**, we choose the (emulated via USB) serial port we think our computer uses to communicate with NodeMCU DEVKIT v1.0, for instance COM6 on Windows.

d. Select **Arduino_IDE>Sketch>Upload** (or hit the famous Arduino right arrow icon). If the LED is not blinking after several seconds, we should change serial port and retry.

If the LED remains in the same state, we try changing upload speed with **Arduino_IDE>Tools>Upload Speed** and retry. Normally it’s 115200 Bd but it depends on the firmware our NodeMCU DEVKIT v1.0 is shipped with.

At this point, the LED should blink!

### B) 3) Memory

Let’s say a word about **ESP-12E memory**. On Arduino IDE status bar, we have this:

![Arduino IDE status bar](https://cdn-images-1.medium.com/max/2000/1*qWKeXXQ4xhSGpELuvmPKmg.jpeg)*Arduino IDE status bar*

* “NodeMCU 1.0 (ESP-12E Module)” is the board we work with.

* “115200” is the program’s upload speed, here 115200 Bd.

* “4M” is the size of SPI flash memory in ESP-12E, i.e. 4 MiB.

* “(3M SPIFFS)”: From the 4 MiB mentionned before, 3 are here reserved (this can be changed) to be handled by **SPI Flash File System** (SPIFFS). For instance, if we plan to use ESP8266 as a web server, our web site would be stored somewhere in those 3 MiB. So we have 1 MiB left to store our Arduino program and libraries.

* “COM6” is the serial port our computer uses to communicate with NodeMCU DEVKIT v1.0.

After uploading we have also these sentences in Arduino IDE:

    Sketch uses 224281 bytes (21%) of program storage space. Maximum is 1044464 bytes.
    Global variables use 31768 bytes (38%) of dynamic memory, leaving 50152 bytes for local variables. Maximum is 81920 bytes.

* The first and second sentences effectively show that we have 1 MiB for our program and libraries. We learn that we used 21% of this.

* The third and fourth sentences show that we used 38% of the 80 KiB available RAM.

*Note: *Some memory can be EEPROM for sketches to store data.

### B) 4) End of “Hello, World!” test

Now, let’s see the messages ESP8266 is sending us.

**e. **We select **Arduino_IDE>Tools > Serial Monitor**. When the Serial Monitor appears, we set its serial speed at 115200 Bd:

![Serial Monitor: Speed setting](https://cdn-images-1.medium.com/max/2000/1*L5sAVsRew5tiubrA_JJypg.png)*Serial Monitor: Speed setting*

This Serial Monitor speed is of course the same that the one we set for ESP8266 in this line of our program:

    Serial.begin(115200);

**f. **And this is eventually the scroll we get in Serial Monitor:

![Serial Monitor as program is running](https://cdn-images-1.medium.com/max/2000/1*0adTXweN_JYp_2gZUbRG5g.png)*Serial Monitor as program is running*

## C) Measuring an analog data like luminosity

Let’s go a step further by measuring an analog data with our NodeMCU DEVKIT v1.0.

The analog data wire should be connected to pin A0 of NodeMCU DEVKIT v1.0. Thanks to the voltage divider we mentionned in A)2), the analog data should be between 0 V and 3.2 V:

* 0 V produces the minimal integer 10-bit value, *i.e.* 0.

* 3.2 V produces the maximum integer 10-bit value *i.e.* 1023.

For instance, 2.5 V produces: 2.5 * (1023/3.2) = 799.

As we said before, we want to measure luminosity (or, more accurately, the voltage that is an image of it).

This document from Adafruit [[link to pdf](https://cdn-learn.adafruit.com/downloads/pdf/photocells.pdf)] explains how a **photo cell** (or Light Dependent Resistor, **LDR**) can be used to measure luminosity (*i.e.* how you can relate its ohm or voltage values to luminosity) and how you can connect it to a microcontroller analog input. It even gives Arduino program examples.

So this is the assembly we will adopt (no need of a voltage follower between LDR and NodeMCU DEVKIT v1.0, we checked that point):

![Luminosity measurement with a LDR and NodeMCU DEVKIT v1.0](https://cdn-images-1.medium.com/max/2000/1*bG-67j9rY2NYMmeKRGZRFA.jpeg)*Luminosity measurement with a LDR and NodeMCU DEVKIT v1.0*

* Total darkness should produce a 10-bit equivalent value close to 0.

* Direct strong sunlight should produce a 10-bit equivalent value close to 1023.

With a small breadbord, we can get this compact installation (in fact we’ve been given the whole hardware in a [@RedisLabs](http://twitter.com/RedisLabs)** **meetup [[link](https://www.meetup.com/fr-FR/Redis-Lille/events/247121494/)] hosted by [**@**ZenikaLille](https://twitter.com/ZenikaLille), we thank both of them here!):

![NodeMCU DEVKIT v1.0 with LDR and a 10k resistor](https://cdn-images-1.medium.com/max/2000/1*iHX6NLG9_v7LXJzbIvXD5w.jpeg)*NodeMCU DEVKIT v1.0 with LDR and a 10k resistor*

*Note: *A LDR is not the best luminosity sensor in town! But for sure, it’s the cheapest!

**g. **The program can be as simple as that:

<iframe src="https://medium.com/media/e479936c3800f3a9c22db543f52eef53" frameborder=0></iframe>

**h.** And eventually, after we upload this program to NodeMCU DEVKIT v1.0, we get the following sreenshot of Arduino IDE Serial Monitor (we produced different luminosity environments - covering LDR, adding light, … - to get this values):

![Serial Monitor: Different values of luminosity image](https://cdn-images-1.medium.com/max/2000/1*eNDIe3anhUhdUtaCoYYdVA.png)*Serial Monitor: Different values of luminosity image*

## D) Connecting NodeMCU DEVKIT v1.0 to our LAN and Internet

### D) 1) Procedure

Serious things begin now! To connect our NodeMCU DEVKIT v1.0 to our LAN, this latter should of course have a Wifi access point. Also, our LAN gateway should run a DHCP server to give an IPv4 address to our component when it asks for one. And if we want our component to go to Internet though we have only one public IP address, our gateway should implement NAT. Nothing to worry about, almost all domestic Fiber/ADSL boxes do that naturally.

To get connected to our LAN, we just have to copy the **void setup()** function of one of the “Arduino core for ESP8266” examples, for instance: [[link](https://github.com/esp8266/Arduino/blob/master/libraries/ESP8266WiFi/examples/WiFiClient/WiFiClient.ino)] (or you can reach it by: **Arduino_IDE>File>Examples>ESP8266Wifi>WifiClient**).

The short program looks like this, don’t forget to replace the values of WIFI_SSID and WIFI_PASSWORD by yours:

<iframe src="https://medium.com/media/c2f333962fdc9c7a4e172c8276d613c5" frameborder=0></iframe>

And this is the kind of screenshot we get on Serial Monitor:

![NodeMCU DEVKIT v1.0 got connected and obtained an IP address by DHCP](https://cdn-images-1.medium.com/max/2000/1*-Mt3uxPxnp8J2s-kJfPtiQ.png)*NodeMCU DEVKIT v1.0 got connected and obtained an IP address by DHCP*

Now, we encourage you to continue this example and implement totally a TCP client or even a web server, finding help in “Arduino core for ESP8266” examples and Internet. By lack of time, we won’t do it here (but again, do it!). We prefer focussing on a client that connects to a Firebase RealTime Database…

### D) 2) Security considerations

Hard coding Wifi SSID and password in the sketch might be a problem:

* if we want to communicate the sketch to others, we have to mask these informations,

* if the wifi configuration changes, we have to reprogram the device.

To avoid these drawbacks, there is a solution called **WifiManager** [[link](https://github.com/tzapu/WiFiManager)], we didn’t personally test it but it has many stars on github.

Anyway, whatever the manner **Wifi SSID and password** are specified with, at the end they **lie in plain text in the flash memory**. If a person with bad intention has **physical access to the chip**, it is really easy for it to get all credentials.

For instance, we dumped our 4MiB flash memory to a file with the use of **esptool **[[link](https://github.com/espressif/esptool)], an official Espressif tool:

    esptool.py -p COM6 read_flash 0 0x400000 flash_contents.bin

Then, searching in an editor the Wifi SSID, we find close to it the Wifi password:

![Searching for Wifi password once flash memory is dumped](https://cdn-images-1.medium.com/max/2000/1*VQ5XcrTjfo9MML-vAESqOQ.png)*Searching for Wifi password once flash memory is dumped*

Later, we will also have to store credentials that give access to a Firebase database. The problem will be the same as here… they will be easily discoverable.

**How can we protect credentials from hackers having physical access to the chip ?**

* ESP8266 doesn’t support native flash encryption. A possibility is to make it work with a encrypting device like **ATECC508A** or **ATECC608A **[[link](https://www.microchip.com/wwwproducts/en/ATECC608A)] from Microchip/Atmel. **Mongoose OS** (a step further from what we talk in this post) uses this solution to encrypt sensitive information when dealing with ESP8266 [[link](https://mongoose-os.com/docs/mongoose-os/userguide/security.md#atecc608a-crypto-chip)].

* But maybe it would be simpler to upgrade to a **ESP32 chip** (we talked about this ESP8266 successor at the beginning of this post), a device with native encryption capability.

## E) Connecting and pushing to a Firebase Realtime Database

### E) 1) What is Firebase Realtime Database?

![Firebase Reatime Database logo](https://cdn-images-1.medium.com/max/2056/1*ILPd9z7VdKnDWJklf3lpPg.png)*Firebase Reatime Database logo*

**Firebase** [[link](https://firebase.google.com/)] is a huge Google platform to help build sophisticated, realtime apps. Each Firebase project has one online NoSQL databases (to chose between **Firebase Realtime Database**, a simple yet powerful database,** **or **Cloud Firestore**, a more sophisticated, document oriented database), handles authentication, cloud messaging, cloud storage, cloud functions, etc.

*Note:* Firebase is deeply linked to **Google Cloud**. To know the relationship, read this post from [Doug Stevenson](undefined) : [[link](https://medium.com/@CodingDoug/whats-the-relationship-between-firebase-and-google-cloud-57e268a7ff6f)].

What interests us here is the first online NoSQL database called Firebase Realtime Database because we will push into it each new (10-bit equivalent) analog data NodeMCU DEVKIT v1.0 has just measured.

For the moment, this is a basic usage of this database (it is capable of much more!) but the good point is that new data won’t be stored in NodeMCU DEVKIT v1.0. Thus, we can have a really **big measurement history **without being concerned by our component memory limitation.

Another good point is that we will use later the “real time” functionality of this database to :

* add a timestamp to each new pushed value. This is discussed in a second post dealing with Firebase Cloud Functions [[link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-2-14b0609d3f5e)].

* plot to a web app data as it arrives. This is discussed in a third post dealing with Firebase SDK [[link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-3-644048e90ca4)].

### E) 2) Let’s store data!

After NodeMCU DEVKIT v1.0 obtained an Internet connection, it will connect **over a secure connection** and **with credentials** to our Firebase Realtime Database.

Then, for each captured value of luminosity, it will **push it to the Firebase Realtime Database** with the following process: In the database, each incoming **value **is associated with a unique **key**, called a **push ID**. Then, this key/value pair is stored in a **node** of the database.

Each key is unique, has 20 characters and keys are chronological. If you’re curious, this post [[link](https://firebase.googleblog.com/2015/02/the-2120-ways-to-ensure-unique_68.html)] describes how push IDs are generated.

For instance, after NodeMCU DEVKIT v1.0 made 3 pushes in the node named measures in the database, the latter looks like this, in JSON:

    {
      "measures" : {
        "-LH2fJPeouqr_JPE6XXZ" : 852,
        "-LH2gSuEplF4sAHWaGaE" : 801,
        "-LH2hbPTIfpIj0Ta6OsK" : 728
      },
      "another_node" : {
        ...
      }
    }

Of course, push is not the only avalaible operation in Firebase Realtime Database. You can update a node, listen to events on it (*e.g.* child added), delete it... There is a **Firebase SDK** to perform all these actions for several languages (JavaScript, C++ for instance) and operating systems (Android, iOS).

Unfortunately, in Firebase documentation, there is no mention of something like “Firebase SDK for Arduino core for ESP8266**”**. But, but… a library called **FirebaseArduino **exists and its github repo is here [[link](https://github.com/firebase/firebase-arduino)]!

Nevertheless, it’s written:
> “ The Arduino library is [under heavy development](https://github.com/googlesamples/firebase-arduino/issues), experimental, unversioned and its API is not stable.”

Ok! That’s why experimental projects are for! Let’s go ahead.

### E) 3) Setting up a Firebase project and its Realtime Database

*Note:* We won’t do it here but it is good practice to have a Firebase project for development and another one for production.

Below are detailed the steps we went through:

**a.** Create a Firebase project by going to **Firebase console**: [https://console.firebase.google.com](https://console.firebase.google.com)

**b.** Click the **Add project** blue icon and then give a name to it, for us, this will be esp8266-rocks:

![Firebase console — Creating a Firebase project](https://cdn-images-1.medium.com/max/2000/1*5_sokPgnuYyrJ_TDZI4cTg.jpeg)*Firebase console — Creating a Firebase project*

**c. **Click on **Database** on the aside menu:

![Firebase console — Selecting Database](https://cdn-images-1.medium.com/max/2000/1*cfStXjP9jVPehE46nJN3zw.jpeg)*Firebase console — Selecting Database*

**d.** Choose **Realtime Database**, not **Cloud Firestore**. Then **Create database**:

![Firebase Console — Creating the Realtime Database for our project](https://cdn-images-1.medium.com/max/2000/1*XmWZAJ54zjt0skiv6ly0jg.jpeg)*Firebase Console — Creating the Realtime Database for our project*

**e. **Set **read/write rules** to true for the moment. It’s not really important as we will come back later to this:

![Firebase Console — Setting temporary read/write rules](https://cdn-images-1.medium.com/max/2000/1*R3gxtzejCAlx1WM80nG_kA.jpeg)*Firebase Console — Setting temporary read/write rules*

**f. **We end up with a graphical view of our (still empty) database:

![Firebase Console — Graphical view of Firebase Realtime Database after creation](https://cdn-images-1.medium.com/max/2000/1*GA79EHXhkOxXWNmzKvzBqA.jpeg)*Firebase Console — Graphical view of Firebase Realtime Database after creation*

*Note:* We can discover our database by adding key/value pairs clicking “+” in the above screen. We can also nest data, have objects, arrays, etc. If we export this database we’ll see that it is in fact a Javascript object described in JSON.

*Note:* With **Firebasebase “Spark” pricing plan** (the free one), we can store up to 1 GB in a Firebase Realtime Database. To know the currently used storage , we hit **Project Overview** in Firebase Console:

![Firebase Console — Project Overview](https://cdn-images-1.medium.com/max/2000/1*iJHD5-c9eccbYKSC_B-tXQ.png)*Firebase Console — Project Overview*

**g. **Let’s fix the read/write rules now by clicking on **Rules** tab.

![Firebase Console — Setting Firebase Realtime Database read/write rules (We hit Publish button when done.)](https://cdn-images-1.medium.com/max/2000/1*N0uuGs86lVceXkJSTXLNBA.png)*Firebase Console — Setting Firebase Realtime Database read/write rules (We hit Publish button when done.)*

    {
      "rules": {
        "measures": {
            ".read": "false",
            ".write": "false"
        }
      }
    }

*What does this mean?*

Those rules mean that a third party with no admin privileges, *e.g. *a script on a website, an Android app… trying to write to/read from measures node in the database will be denied. On the contrary, with admin privileges, all write to/read from any database node are permitted!

*So, who has admin privileges?*

* The owner of the Firebase Project (us!) manipulating the database from the Firebase Console.

* Some Cloud Function like the one we’ll write in Post 2 [[link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-2-14b0609d3f5e)].

* An entity accessing the database with a **“secret” type authentication**. This is the case of an ESP8266 using **FirebaseArduino library** (described below).

**h. **At least, it’s time to get authentication credentials for our NodeMCU DEVKIT v1.0. We adapt here the (outdated) “configuration” in **FirebaseDemo** example from **FirebaseArduino** github repository [[link](https://github.com/firebase/firebase-arduino/tree/master/examples/FirebaseDemo_ESP8266)].

* What we will call later in Arduino program FIREBASE_HOST corresponds to the Project Id appended by .firebaseio.com , for example esp8266-rocks.firebaseio.com. It’s the URL to the database.

* What we will call later in Arduino program FIREBASE_AUTH corresponds to the **“secret”** we obtain by the 5-step sequence below:

![Firebase Console — Towards obtaining a “secret”](https://cdn-images-1.medium.com/max/2000/1*OwlSWHtk-qbkiwt7O_MrXg.jpeg)*Firebase Console — Towards obtaining a “secret”*

![Firebase Console — The “secret” is 40-char long string. It can be revealed and copied to clipboard by clicking Show button](https://cdn-images-1.medium.com/max/2368/1*vpWwKf7Er3NKEsJZpVg0tA.jpeg)*Firebase Console — The “secret” is 40-char long string. It can be revealed and copied to clipboard by clicking Show button*

As indicated, **getting authenticated by a Project ID and a secret is deprecated**. The reason is that when a device gets authenticated that way, it has admin privleges on the database and **this is not a good practice**. Moreover, using the “secret” authentication technique deprives us from the subtility of the Firebase Security Rules feature (see [[link](https://firebase.google.com/docs/database/security/)]).

But at the time of writing, it seems that there is no other such ready-made solution for authenticating an ESP8266 Arduino to Firebase. On the **FirebaseArduino API Reference **[[link](https://firebase-arduino.readthedocs.io/en/latest/)], the only function to connect to a Firebase project is Firebase.begin(..., ...) and it requires two parameters like the ones we’ve seen before:FIREBASE_HOST (a URL) and FIREBASE_AUTH (a secret). It is said however that this latter can also be a **token** (and so we wouldn’t use anymore this deprecated solution) but its management is not so easy, given our project.
[**Class Documentation - firebase-arduino 1.0 documentation**
*This implementation is designed to follow Arduino best practices and favor simplicity over all else. For more…*firebase-arduino.readthedocs.io](http://firebase-arduino.readthedocs.io/en/latest/)

Discussion about updating the authentication process in FirebaseArduino (*i.e.* no more use of a secret) is active on github [[link](https://github.com/firebase/firebase-arduino/issues/224)]. Informations on this topic are welcomed.

*Note: *We can have several secrets for one database (thanks to “Add secret” button on the view above), for instance one per ESP8266 device connected. In that case, the database should then be managed to have a supplementary nesting level, just like in the view below. But, one more time, with “secret” authentication, device #1 has access to device #2 data and *vice versa*.

    {
      "measures" : {
        "device01": {
          "-LH2fJPeouqr_JPE6XXZ" : 852,
          "-LH2gSuEplF4sAHWaGaE" : 801,
          "-LH2hbPTIfpIj0Ta6OsK" : 728
        },
        "device02": {
          "-LTOa64c9TKkr9s3_3Xn" : 16,
          "-LTObDyJrL_dsHw6mZ4k" : 13,
          "-LTOcLvAP49gBd6U9OhE" : 18,
          "-LTOdTsrDe_SEybwoXTI" : 14
        }
      }
    }

### E) 4) Setting up Arduino IDE to use FirebaseArduino library

We adapt here the (outdated) “software setup” documentation in **FirebaseDemo** example from **FirebaseArduino** github repository [[link](https://github.com/firebase/firebase-arduino/tree/master/examples/FirebaseDemo_ESP8266)].

**a.** First we download FirebaseArduino library:

[https://github.com/googlesamples/firebase-arduino/archive/master.zip](https://github.com/googlesamples/firebase-arduino/archive/master.zip)

**b. **Then we install it in Arduino IDE: **Arduino_IDE>Sketch>Include Library>Add .ZIP Library…** and we choose the zip we just downloaded.

But Firebase Arduino has a dependency called **ArduinoJson library** [[link](https://github.com/bblanchon/ArduinoJson)]. We also need to install it in Arduino IDE.

**c.** First we download ArduinoJson library:

[http://downloads.arduino.cc/libraries/github.com/bblanchon/ArduinoJson-5.13.2.zip](http://downloads.arduino.cc/libraries/github.com/bblanchon/ArduinoJson-5.13.2.zip)

(Another possible link for this library is [[link](https://github.com/bblanchon/ArduinoJson/releases)].)

**d. **Then we install it in Arduino IDE: **Arduino_IDE>Sketch>Include Library>Add .ZIP Library…** and we choose the zip we just downloaded.

### E) 5) NodeMCU DEVKIT v1.0 program to push values to Firebase Realtime Database

In FirebaseArduino github page, there is several example and one of them is called FirebaseDemo [[link](https://github.com/firebase/firebase-arduino/tree/master/examples/FirebaseDemo_ESP8266)]. It illustrates lot of functions of this FirebaseArduino library. We encourage you to explore theses functions (and see lively their effects in Firebase console) as we will only focus on the one we need in our project.

Indeed, among these functions we will just use the one called pushInt(). For example, to push the value of the int (integer) variable named sensorValue to the node "measures" of the Firebase Realtime Database, it is a simple as that:

    Firebase.pushInt("measures", sensorValue);
    if (Firebase.failed()) {
      // Do something if push went wrong
    }

We can thus complete our luminosity measurement program:

<iframe src="https://medium.com/media/74cb82ce9e9e5a7bbc11dece16f57ce1" frameborder=0></iframe>

WAAO! Look what we get in Firebase Realtime Database Console:

![Firebase Console — Each pushed value of luminosity is associated with a unique PushID](https://cdn-images-1.medium.com/max/2000/1*6aFAQyXJo3nQ-0BUjf45Rw.gif)*Firebase Console — Each pushed value of luminosity is associated with a unique PushID*

So, it seems that luminosity in North of France isn’t so bad, no ?

***Troubleshooting***

If things don’t work, you will get the message "pushing failed:..." in Arduino IDE Serial Monitor.

* In Arduino program, check your credentials FIREBASE_HOST and FIREBASE_AUTH.

* Try setting temporarily your database rules ."read" and .”write" to "true".

* UPDATE (November 27, 2018 - three months after this post was released) : Today we wanted to make our NodeMCU DEVKIT v1.0 work again. But, although everything was set up like before,we only got tons of "pushing failed:..."! Searching on the web, we managed to solve this issue. It was due to a **Firebase certificate** **fingerprint **that had changed. We can find this latter by typing our project URL appended by /.json in a browser and hit the lock icon to access certificate details. On the other side, in our FirebaseArduino library folder firebase-arduino-master/src, there is a file called FirebaseHttpClient.h. Inside it, there is a line where the fingerprint should be declared (hexadecimal digits by pairs). It has to match the one seen in the certificate details. That was no longer the case!

![Firebase fingerprint and its match in FirebaseHttpClient.h](https://cdn-images-1.medium.com/max/2000/1*sHZzyDUxNfBIiJpBppQ4iA.jpeg)*Firebase fingerprint and its match in FirebaseHttpClient.h*

## F) Let’s save power with ESP8266 deep sleep mode

When we need to push a new measure every 3 seconds (to be exact : measure *then* push *then *delay 3 seconds), it’s not worth putting ESP8266 Wifi off after the push is done. ESP8266 would lose seconds to reconnect and we would miss some measures. But we have to be aware that Wifi draws a lot of power. For instance, ESP-12E consumes **120 mA** while transmitting in Wifi N. In such a situation we can’t run in an autonomous way for days and days with a **USB (solar) power bank**.

Things are different when we need a new measure every 5 minutes (for instance). In this case it is worth putting ESP-12E in **deep sleep mode** (consumption: **10 µA**) after the push to database is done:

    // Measure analog value:
    // ...
    // Push value to database:
    // ...
    // Now let's go in deep sleep for 5 minutes:
    ESP.deepSleep(300e6, WAKE_RF_DEFAULT); // 1st parameter is in µs!

ESP.deepSleep() instruction puts ESP-12E in deep sleep mode during a time indicated in µs (first parameter). The WAKE_RF_DEFAULT mode (second parameter) ensures that Wifi will be restarted when the chip wakes from deep sleep.

Once the deep sleep duration is achieved, **D0 (GPIO16) **pin of NodeMCU DEVKIT v1.0 will issue a high to low transition. So **D0 pin has to be tied to RST pin in order to provoke a chip reset and make the program start again**:

![Enabling the chip to reset after deep sleep thanks to the D0 to RST connection](https://cdn-images-1.medium.com/max/2000/1*R4R0PZSDmoCT1hV4C5Cslw.jpeg)*Enabling the chip to reset after deep sleep thanks to the D0 to RST connection*

The overall average current will drop to **mA** and supplying the NodeMCU DEVKIT v1.0 by a power bank is now feasible. If we have time, we will make measures and update this post.

*Important note: *It’s a shame but D0 (GPIO16) controls as well the built-in LED of NodeMCU DEVKIT v1.0. Due to D0 to RST connection, **we must forget the idea of managing the built-in LED** as we did until now, **otherwise we will provoke a chip reset** when we set the built-in LED on!

*Important note:* While uploading the program from Arduino IDE to NodeMCU DEVKIT v1.0, the D0 to RST connection has to be removed otherwise upload won’t work.

One last thing: The code has to be refactored because we regularly provoke a reset. Thus, all code takes place in void setup() function. The void loop() function is an empty one.

The code is below. Try it now:

<iframe src="https://medium.com/media/e2976806e8ebeace4bcb00034d41414b" frameborder=0></iframe>

*Note:* When concerned with power saving, **Feather HUZZAH development board** (cf. part A) 2)) could be a good choice because it has a 3.7 V supply input. Sadly, we didn’t have time to investigate this solution.

## Conclusion

We’ve travelled a lot to discover different technologies (**ESP8266**, **Arduino IDE**, **Firebase Realtime Database**) and link them together to build the fundations of a low cost ans low power IoT project! And, even if we’re not fully satisfied with the ESP8266 “secret” authentication, it is a secure project… as long as a hacker doesn’t have physical access to it.

There is more to come:

* In a second post we’ll see how to add a timestamp to each measured value of luminosity thanks to a **Firebase Cloud Function**.

* And in a third one we will lively plot luminosity values as they arrive, thanks to **a web app **using **plotly.js**.

So stay tuned!

**Links to the posts: **Post 1, [Post 2](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-2-14b0609d3f5e), [Post 3](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-3-644048e90ca4), [GitHub](https://github.com/olivierlourme/esp8266webapp) and [Live Demo](https://esp8266-rocks.firebaseapp.com/).

**Check also this post:** [GCP-Cloud IoT Core with ESP32 and Mongoose OS](https://medium.com/@o.lourme/gcp-cloudiotcore-esp32-mongooseos-1st-5c88d8134ac7?source=---------4------------------).
