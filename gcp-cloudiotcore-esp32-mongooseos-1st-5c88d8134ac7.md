
# How to check the weather using GCP-Cloud IoT Core with ESP32 and Mongoose OS

This is a step-by-step tutorial for newbies to Google Cloud Platform-Cloud IoT Core. The devices are ESP32 Wifi chips running Mongoose OS.

**Live demo is here:** [https://hello-cloud-iot-core.firebaseapp.com/](https://hello-cloud-iot-core.firebaseapp.com/)

**GitHub for last section* **(Logging, storing and visualizing weather data with Firebase) **is here:* [**https://github.com/olivierlourme/iot-store-display](https://github.com/olivierlourme/iot-store-display)

**This post is completed by a second one:** see [here](https://medium.com/@o.lourme/gcp-cloudiotcore-esp32-mongooseos-2nd-config-state-encrypt-7c5e937e5be9).

![“For the moment we’re just the two of us but soon we’ll be a thousand!”](https://cdn-images-1.medium.com/max/6528/1*GGNvAgxLJXeagpiyQnlHvQ.jpeg)*“For the moment we’re just the two of us but soon we’ll be a thousand!”*

## TL;DR

In this post, we describe the concepts and then the setup of an **IoT system** managed by **Google Cloud Platform-Cloud IoT Core**, the devices being **ESP32 **Wifi** **chips having **Mongoose OS** for OS.

## Introduction

### 1) History

In a previous 3-post series [[link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-1-a07db495ac5f), [link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-2-14b0609d3f5e), [link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-3-644048e90ca4)], we used an **ESP8266 Wifi chip **to regularly measure luminosity and feed a database with the obtained data. The data set was ultimately lively plotted to a web app (see live plot here: [[link](https://esp8266-rocks.firebaseapp.com/)]). We massively used **Firebase products **(Realtime Database, Cloud Functions, SDK and Hosting) to meet our goals.

This project works fine, it draws very little power and we enjoyed developing it — but:

* **This project was okay to handle just a few connected sensors**. Setting up a set of a hundred sensors would require a lot of (rigorous) manual intervention and monitoring them would be challenging as well. Indeed, there is no central place where we can manage our system.

* **Arduino IDE** and **Arduino core for ESP8266** were great for discovering ESP8266 but they are **quickly insufficient**: The IDE file management is really basic, there is only one program in the chip, and **there is no Operating System** **providing useful APIs for IoT**.

* **FirebaseArduino** **library**, allowing an ESP8266 to push data to a Firebase Realtime Database, **was experimental**. Some features like authentication should be improved. For now, the “secret” type authentication we used gives ESP8266 admin rights over the whole database!

* Eventually, **ESP8266 SPI flash memory was not designed to be encrypted**. In our first post [[link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-1-a07db495ac5f)], we showed how easy it was to recover a Wifi password when reading this memory.
> In a word, this past project couldn’t be used in an industrial context. It was more a prototype for a Proof of Concept. We learnt a lot with it but today **we’d like to develop a professional and fully secured solution capable of managing in a simple way a lot of connected sensors**.

This is why we decided to:

* **investigate Google Cloud Platform-Cloud IoT Core** [[link](https://cloud.google.com/iot-core/)]** **to manage our system : devices setup, provision, authentication and monitoring;

* **move from ESP8266 to ESP32**, which offers memory encryption;

* **run Mongoose OS** [[link](https://mongoose-os.com/)] in our ESP32s. This OS accepts programs written in Javascript(JS) and provides a lot of APIs to deal with time, MQTT protocol, sensors, provisioning, etc. It is easy to interface with the main IoT platforms, including Google Cloud Platform-Cloud IoT Core.

### **2) A word about ESP32 Wifi chip**

ESP32 Wifi chip is a successor of the famous ESP8266 we described here: [[link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-1-a07db495ac5f)]. Compared to it, every feature is enhanced (speed up to 240 MHz, two cores, 520 kiB RAM, number of GPIOs, variety of peripherals, etc.) and there are some new ones (Bluetooth: legacy/BLE, **4 MiB-flash memory encryption capability**, **cryptographic hardware acceleration**: AES, SHA-2, RSA, ECC, RNG). There are a lot of resources on the web concerning ESP32. The following one deals with the **ESP32 DEVKIT V1 development board **that we will use and gives its pinout: [[link](https://randomnerdtutorials.com/esp32-pinout-reference-gpios/)].

There is also this extensive resource concerning the wide variety of ESP32 chips and development kits : [http://esp32.net/](http://esp32.net/) . On their home page, searching for “ESP32 DevKit” or “GeekCreit” leads to a link to the [schematic](https://github.com/SmartArduino/ESP/blob/master/SchematicsforESP32.pdf) of our ESP32 DEVKIT V1. This development board embeds an official Espressif ESP32-WROOM-32 chip and costs about 6€ at Banggood.

## Basic IoT concepts explained through our use case

So, what will be our playground for test all these new tools?
> To illustrate IoT concepts through Cloud IoT Core, we chose to build **a weather station** **reporting humidity and temperature from different places**.

For simplicity we’ll handle only 2 places: inside our house (“indoor”) and outside our house (“outdoor”). It’s up to you to deal with many more places.

### 1) Project hardware : ESP32 & DHT22

At each of theses places, we’ll install a connected sensor (**“a device”**) constituted by a **DHT22** **humidity/temperature sensor **(description: [[link](https://learn.adafruit.com/dht)], datasheet: [[link](https://cdn-shop.adafruit.com/datasheets/Digital+humidity+and+temperature+sensor+AM2302.pdf)], 4€ at Banggood)** **connected to an **ESP32 DEVKIT V1** development board. DHT22 observes a kind of “1-Wire” protocol. Each ESP32 will house **Mongoose OS** for operating system. Its installation on an ESP32, a Hello, World! and a test with a DHT22 are given in the following section below.

Just below are given DHT22 specifications. Afterwards, we think the accuracy figures are a bit optimistic but that’s not our concern today.

![DHT22 sensor characteristics ([[link](https://learn.adafruit.com/dht/overview)])](https://cdn-images-1.medium.com/max/2000/1*WIxEy4NLdDr_2SVhcjmv0g.png)*DHT22 sensor characteristics ([[link](https://learn.adafruit.com/dht/overview)])*

We can already build the following assembly twice (one for indoor and one for outdoor). For now, power will come from the USB connector connected to our host machine. In production, power may come from a power bank.

![Assembly Diagram — ESP32 DEVKIT V1 and DHT22 sensor constitute a “device”. Pinout is here : [[link](https://randomnerdtutorials.com/esp32-pinout-reference-gpios/)]](https://cdn-images-1.medium.com/max/2094/1*tFxxVtWwVH9gFajtrCWQ0w.png)*Assembly Diagram — ESP32 DEVKIT V1 and DHT22 sensor constitute a “device”. Pinout is here : [[link](https://randomnerdtutorials.com/esp32-pinout-reference-gpios/)]*

That’s all for hardware! The rest of the project uses **serverless solutions **from Google. We describe them now…

### 2) Project architecture : Cloud IoT Core & Firebase

All this “Project architecture” section is theoretical, there is no step to perform. Its aim is to introduce vocabulary and notions related to IoT, more specifically when this domain involves Google Cloud solutions.

Here is the general architecture of our project:

![Project architecture](https://cdn-images-1.medium.com/max/3204/1*PttQxTGMbDHCGwzCsWlZvA.png)*Project architecture*

*Note: *There is no **gateway **between our devices and Cloud IoT Core because they “speak” MQTT.

*Note:* Devices can also communicate with Cloud IoT Core via its **HTTP bridge**. As it is less performant than the **MQTT bridge** (see a comparison : [[link](https://cloud.google.com/iot/docs/concepts/protocols)]), we will disallow this communication later during registry configuration. Limiting access just to what is necessary is a good practice.

Let’s explain this architecture in three sections:

* “From Devices to Cloud Pub/Sub” describes the classical Google IoT architecture.

* “From Cloud Pub/Sub to data storage and visualization”, describes the choices we made to exploit data.

* “Additional config and state topics” completes this architecture presentation.

**From Devices to Cloud Pub/Sub**

* Cloud IoT Core

**Cloud IoT Core** is the Google Cloud Platform service to which each of our **registered devices **will send temperature/humidity data. When such a data is sent, we say that **the device publishes a telemetry event** (sometimes also called a “telemetry message”).

*Note:* Pricing is detailed here : [[link](https://cloud.google.com/iot/pricing)]. For small projects with a few devices, there’s little chance you get charged.

* *MQTT*

This publication is done through a **MQTT connection**. MQTT is a publish/subscribe-based message protocol; most of the time it lies over TCP [[link](https://en.wikipedia.org/wiki/MQTT)] (or better: over TLS, itself being over TCP). The telemetry message has to be published by the device (a MQTT client) to the Cloud Iot Core “MQTT bridge” (a MQTT server) in a **MQTT topic** whose name imperatively respects this format:

    /devices/{device-id}/events

*Note:* Sub-folders in the topic name are possible. We won’t need this feature here but see [[link](https://cloud.google.com/iot/docs/how-tos/mqtt-bridge#publishing_telemetry_events)], as it can sometimes be useful.

{device-id} is unique to each device. In our case, Mongoose OS creates it from the last 3 bytes of the MAC address of the ESP32. For example it could be esp32_ABB3B4.

* *Quality of Service (QoS)*

The MQTT specification describes three **Quality of Service (QoS)** levels, when publishing to a topic ([[link](https://cloud.google.com/iot/docs/how-tos/mqtt-bridge#quality_of_service_qos)]):

> QoS 0, the message is delivered at most once;

> QoS 1, the message is delivered at least once;

> QoS 2, the message is delivered exactly once.

Cloud IoT Core does not support QoS 2. And QoS 1 is better than QoS 0. So **QoS 1 is the one we will adopt**. Mongoose OS can do that.

* *Security*

Concerning **security**, in our Mongoose OS/Cloud IoT Core context, MQTT communications are made over **TLS **([[link](https://cloud.google.com/iot/docs/how-tos/mqtt-bridge#mqtt_server)]), so (1) the device is assured to be connected to Cloud IoT Core MQTT server (CA’s certificates are stored in Mongoose OS ca.pem file), (2) the data exchange will be private and (3) data integrity will be checked. On the other way, **device authentication **([[link](https://cloud.google.com/iot/docs/how-tos/mqtt-bridge#device_authentication)]) with Cloud IoT Core is performed with a per-device public/private key authentication using **JSON Web Tokens (JWT)**. The device performs the signature part of the JWT with its private key and Cloud IoT Core validates it using the related public key. Mongoose OS tools handles this keys generation and distribution, we’ll see that soon in the section called “Device registration within the Cloud IoT Core project” lying a few paragraphs below. In this section, we’ll see also how to store securely the private key on the device by performing memory encryption (preventing as well reverse engineering).

*Note:* Beyond JWT device authentication, for additional security, it’s possible to impose TLS from Cloud IoT Core to devices (so each device has also a public key certificate, etc.). It is an option we won’t use but it’s described [here](https://mongoose-os.com/docs/mongoose-os/api/net/mqtt.md) for Mongoose OS side (see “mutual TLS”) and [here](https://cloud.google.com/iot/docs/how-tos/credentials/verifying-credentials) for Cloud Iot Core side. It’s good to know that AWS IoT imposes this mutual TLS, unconditionally ([[link](https://docs.aws.amazon.com/iot/latest/developerguide/iot-security-identity.html)]).

* *Registry*

Devices sharing the same purpose are regrouped within a **registry**.

* *Cloud Pub/Sub*

**Telemetry data from all devices belonging to the same registry is then *forwarded *to a Cloud Pub/Sub topic **(Cloud Pub/Sub is a GCP product [[link](https://cloud.google.com/pubsub/)], not specifically a Cloud IoT Core one). The name of the Cloud Pub/Sub topic follows this pattern:

    projects/**id-of-google-cloud-project**/topics/**name-of-telemetry-topic**

So, if we call our Google Cloud project hello-cloud-iot-core, if we choose weather-telemetry-topic for the name of our Pub/Sub telemetry topic and if finally our registry is called weather-devices-registry, we’ll get sooner or later that kind of view in **Google Cloud Console** :

![Project ID, registry ID and telemetry Pub/Sub topic name in Google Cloud Console](https://cdn-images-1.medium.com/max/2000/1*BeVR0exU6l-rXg8LqJvfeQ.jpeg)*Project ID, registry ID and telemetry Pub/Sub topic name in Google Cloud Console*

But no stress, everything will be explained step by step to reach that.

*Note:* As it is said here ([[link](https://cloud.google.com/iot/docs/how-tos/mqtt-bridge#publishing_telemetry_events)]), each message in the Cloud Pub/Sub topic contains a copy of the telemetry message published by the device but also some **message attributes**, the most important being probably deviceID, allowing us to match some received data with the device that published it.

*Note:* We talk a lot about **Pub(lish)**, but where is the **Sub(scribe)**? In fact, we’ll create quickly with Google Cloud Command Line Interface a Cloud Pub/Sub subscription (a “pull” one) in order to view the messages published to the telemetry topic. Later in this post, we’ll create a Firebase Cloud Function reacting to each publication and this will automatically create another subscription (a “push” one this time).

**From Cloud Pub/Sub to data storage and visualization**

We’re following the right part of the project architecture diagram given at the beginning of this post:

![Project Architecture — Weather data storage and visualization](https://cdn-images-1.medium.com/max/2000/1*aN_fNiSZXWfgoNrHPmNXeQ.png)*Project Architecture — Weather data storage and visualization*

A publication to the Cloud Pub/Sub topic will **trigger a Firebase Cloud Function** that will itself **fulfill a Firebase Realtime Database** with the new data. A web app hosted by** Firebase Hosting **will lively plot data from the Firebase Realtime Database, in the same way as we did in a previous post: [[link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-3-644048e90ca4)].

There are other options in the Google ecosystem to store/treat/visualize data. [Alvaro Viebrantz](undefined)’s really good post [[link](https://medium.com/google-cloud/build-a-weather-station-using-google-cloud-iot-core-and-mongooseos-7a78b69822c5)] that helped us uses **Big Query** ([[link](https://cloud.google.com/bigquery/)]) and **Data Studio** ([[link](https://datastudio.google.com)]).

**Additional “config” and “state” topics**

On the project architecture diagram given at the beginning of this post, we see besides telemetry two other data flows: **Config **([[link](https://cloud.google.com/iot/docs/how-tos/config/configuring-devices)]) and **State **([[link](https://cloud.google.com/iot/docs/how-tos/config/getting-state)]):

![Config and State data flows](https://cdn-images-1.medium.com/max/2000/1*kBpmaNkEksBh_SAju-XpxA.png)*Config and State data flows*

Indeed, the Cloud IoT Core service may publish **configuration update messages** to a special topic the device has subscribed to ([[link](https://cloud.google.com/iot/docs/how-tos/config/configuring-devices)]). It is useful when we need the device to go to a new state, *e.g.* by updating a parameter of its associated sensor, by changing a deep sleep period, moving a servomotor, etc.

For efficiency, there shouldn’t be more than one message of that type per second per device. Such a message is an arbitrary user-defined blob (we’ll use JSON), up to 64 kiB. At last, the name of this special MQTT topic is imperatively:

    /devices/{device-id}/config

On the other hand, a device may publish to a special topic — that Cloud IoT Core has automatically subscribed to — **messages concerning its state **([[link](https://cloud.google.com/iot/docs/how-tos/config/getting-state)])**, ***e.g.* quantity of RAM available, state of a button, etc. It is often used to see if the previous config message sent to the device had the desired effect.

For efficiency, this kind of publication shouldn’t be done more than once per second per device. Such a message is an arbitrary user-defined blob (we’ll use JSON), up to 64 kiB. At last, the topic to which the device publishes its state data has imperatively this name:

    /devices/{device-id}/state

*Note:* Sending** commands **to devices is also possible from Cloud IoT Core: see [[link](https://cloud.google.com/iot/docs/how-tos/commands)] but we won’t illustrate it.
> But for the moment, we will focus on telemetry. After this journey, in a “coming soon” post we will show how to handle *config* and *state* special topics.

UPDATE March 29, 2019: This post about *config* and *state* special topics is out: [[link](https://medium.com/@o.lourme/gcp-cloudiotcore-esp32-mongooseos-2nd-config-state-encrypt-7c5e937e5be9)].

## **Mongoose OS installation on devices**

### 1) A short description of Mongoose OS

**Mongoose OS** ([[link](https://mongoose-os.com/)], [[link](https://lwn.net/Articles/733297/)]) is a smart IoT-oriented OS, runnable on several chips, including ESP8266 and ESP32. Mongoose OS is in partnership with the major actors in IoT ([[link](https://mongoose-os.com/about.html)]). It comes with a development tool called **mos**, working either in a UI or with a Command Line Terminal (like cmd.exe in Windows). In either cases, we’ll write mos commands. There is also a device management app called mDash but we didn’t try it. **Numerous APIs dealing with most of the network and sensor protocols are provided.** Programs can be written in both C/C++ and JS.

At last, there is a 12 tutorial series on YouTube, really useful:

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/videoseries" frameborder="0" allowfullscreen></iframe></center>

*Note:* We used Mongoose OS Community Edition, which is free, licensed under Apache 2.0.

### 2) Mongoose OS installation on ESP32

This installation has to be performed on each device.

We head to the **developers section** of Mongoose OS web site ([[link](https://mongoose-os.com/docs/quickstart/setup.md)]) in order to perform the **first seven steps** of the list given in this resource:

![Mongoose OS setup steps](https://cdn-images-1.medium.com/max/2044/1*LZbBhMBqdngkZwdJrmtANg.png)*Mongoose OS setup steps*

**Step #1**, **Step #2** and **Step #3** are trivial. At Step #3 don’t forget to connect the device to the host machine via a USB cable.

For **Step #4 **“Create new app”, we choose to call the app app1. When mos clone https://github.com/mongoose-os-apps/demo-js app1 indicated on the web site is completed, mos tool automatically goes to the just created app1/ folder.

In app1/fs/ folder, there is a source file called init.js. It is a demo file capable to communicate with different IoT platforms (if they are configured of course). We will basically test it and soon simplify it for our purposes.

![**mos tool** launched in a UI ; ESP32 selected; Serial console (on the right) is by default at 115200 bds, ESP32’s default speed.](https://cdn-images-1.medium.com/max/2168/1*5GYs7d1Ws_km6oDnWPaa6Q.png)***mos tool** launched in a UI ; ESP32 selected; Serial console (on the right) is by default at 115200 bds, ESP32’s default speed.*

**Step #5** “Build app firmware” is launched with mos build command (add --arch esp32 to this command if you launch it from a Command Line Terminal, not from mos tool). It may take a while but normally we have to perform this build only once. After it, we have many more files. One called app1/build/fw.zip contains the binaries of the OS and init.js. It will be flashed to ESP32 in the next step.

**Step #6** “Flash firmware” is launched with mos flash command. It has normally to be done just once. Even if later we change some files (like init.js for instance), we will use a mos put command to upload a file from the host machine to the **local device’s file system**. Of course this command is only available after the flash process.

*Note:* Firmware flash step can be tricky with a brand new ESP32. With our ESP32 DEVKIT V1, we had messages in console (that’s a first good point!) reporting issues about failing to connect to ESP32 ROM. Retrying to flash by pressing the BOOT button (closed to USB connector) finally turned out to a successful flash. Though, be ready to wait for one minute or two.

Then, the device automatically reboots and executes init.js. We obtained every second the following information in mos console (or in any serial terminal @115200 bds) :

![Console after initial ESP32 flash firmware with Mongoose OS](https://cdn-images-1.medium.com/max/2000/1*0i3bn3fhEkAolrkfvBjQVg.png)*Console after initial ESP32 flash firmware with Mongoose OS*

In **Step #7** we connect ESP32 to our wifi network (we use mos tool):

    mos wifi WIFI_NETWORK_NAME WIFI_PASSWORD

The device will reboot by itself after getting an IP address and synchronizing time by contacting a SNTP server. We then ping our device to check its internet connexion.

*Note: *We get device information (IP address for instance) by hitting **CTRL+i **in mos tool, or by typing mos call Sys.GetInfo.

*Note: *We reset the device by hitting **CTRL+u** in mos tool, or by typing mos call Sys.Reboot.

*Note: *Steps #5, #6 and #7 could be the beginning of a “provisioning script”, useful if we have many devices to setup. It is optional to rerun Step #5 if all devices are the same, *e.g.* only ESP32.

### 3) ESP32 “Hello, World!” program with Mongoose OS

To get used to Mongoose OS JS programming style and mos tool, let’s write a small program whose aim is to make the blue built-in LED blink and print messages on console. **This led is connected to GPIO2 pin** of ESP32 DEVKIT V1 (see Assembly Diagram at the beginning of this post). On our host machine, let’s replace the content of app1/fs/init.js by this one:

<iframe src="https://medium.com/media/7588ba61b732872a9a5f00c76017b662" frameborder=0></iframe>

From mos tool or from a Command Line Terminal, we upload this file to Mongoose OS file system and finally we reboot the device:

    mos put fs/init.js
    mos call Sys.Reboot

The blue led should blink and we should see alternatively Tick and Tock printed on console.

### 4) DHT22 test with Mongoose OS

At the beginning of this post, there is an Assembly Diagram showing how to connect DHT22 sensor with ESP32 DEVKIT V1. We chose to connect **DHT22 data pin** **to GPIO0 of ESP32 DEVKIT V1**.

So, here is another short init.js program. This one prints periodically to serial console DHT22 measures (temperature and humidity - as an object in JSON, no MQTT publication yet):

<iframe src="https://medium.com/media/1a5ddf7eee10ee76e77a01ee189ae780" frameborder=0></iframe>

Then:

    mos put fs/init.js
    mos call Sys.Reboot

And this is the related console we get after uploading the init.js program and rebooting the device. Humidity is in % and temperature is in Celsius degrees:

![DHT22 measures as printed to console. A bit too accurate, isn’t it ?](https://cdn-images-1.medium.com/max/2000/1*2xr980fPqZCYC84iUq4qEw.png)*DHT22 measures as printed to console. A bit too accurate, isn’t it ?*

Numbers seem to have a long decimal part but this will be fixed later within a Cloud Function.

*Note:* For pedagogical purposes, we chose all over this post explicit long key names like temperature or humidity. This will have consequences on the volume of data stored later in a NoSQL database (Firebase Realtime Database) as those keys will be repeated for each measure. Shorter key names could be a good idea.

### 5) Let’s publish data to the MQTT telemetry topic

**This is our last program, the one ready to work with Cloud IoT Core!** On the previous program, we just add a publication to the telemetry topic we already talked about: /devices/{device-id}/events.

Note that messages are published in JSON as it will facilitate later their content retrieval with the Firebase Cloud Function reacting to messages publication.

<iframe src="https://medium.com/media/3e9fd42be8ae0ab9d396271443eec67a" frameborder=0></iframe>

We name this file init.js, upload it to Mongoose file system, then provoke a reset:

    mos put fs/init.js
    mos call Sys.Reboot

*Note: *These commands could be appended to the “provisioning script” we mentioned earlier.

When running, this last program prints data to console but it fails to publish data to the MQTT bridge of Cloud IoT Core (MQTT.pub() returns 0):

![Telemetry data can still not be published as we haven’t set up a Google Cloud project yet.](https://cdn-images-1.medium.com/max/2000/1*Q-FIqugYuMQGd5rjjeu56Q.png)*Telemetry data can still not be published as we haven’t set up a Google Cloud project yet.*

Indeed, we haven’t set up any Google Cloud project yet, neither *a fortiori *registered a single device to it. Let’s do it now!

## Cloud IoT Core project setup

### 1) Google Cloud SDK installation

Firstly we need to **install Google Cloud SDK** because we will have to type some **gcloud commands** in a Command Line Terminal. At the time of writing, it requires Python 2.7. It won’t work with Python 3.5. The Google Cloud SDK download page ([[link](https://cloud.google.com/sdk/downloads)]) offers versions of the SDK with Python bundled inside (if you’re sure you don’t have Python already installed and don’ t want to handle this Python point).

Then, Cloud IoT Core requires some **Beta versions of gcloud commands**. So in a Command Line Terminal, from any folder, we type:

    gcloud components install beta

These two previous steps have to be done just one time!

*Note:* Most of the following actions on Google IoT Core can be performed in three ways:

* with **Google Cloud Console** (on the web)

* with **some APIs** in different languages, and

* with **Command Line Interface** in a terminal, typing gcloud commands.

We will use the latter to configure things and we’ll check facts with Google Cloud Console (on the web).

### 2) Google Cloud project setup

We follow now this guide from Mongoose OS web site : [[link](https://mongoose-os.com/docs/quickstart/cloud/google.md)].

    **# Commands indicated in this grey frame have to be done just once to configure the Google Cloud project! They can be performed from any folder.**

    **# Get authenticated with Google Cloud**
    gcloud auth login

    **# Create cloud project. We chose *hello-cloud-iot-core* as PROJECT_ID**
    gcloud projects create *hello-cloud-iot-core*

    **# Give Cloud IoT Core permission to publish to Pub/Sub topics**
    gcloud projects add-iam-policy-binding *hello-cloud-iot-core* --member=serviceAccount:cloud-iot@system.gserviceaccount.com --role=roles/pubsub.publisher

    **# Set default project for gcloud**
    gcloud config set project *hello-cloud-iot-core*

    **# Create Pub/Sub topic for device telemetry
    **gcloud beta pubsub topics create *weather-telemetry-topic*

    **# Create a Pub/Sub subscription to the just created topic**
    gcloud beta pubsub subscriptions create --topic *weather-telemetry-topic* *weather-telemetry-subscription*

    **# Create devices registry (we call it *weather-devices-registry*)
    # Precise Pub/Sub topic name for event notifications**
    **# Disallow device connections to the HTTP bridge**
    gcloud beta iot registries create *weather-devices-registry* --region europe-west1 --no-enable-http-config --event-notification-config=topic=*weather-telemetry-topic
    ***# Say 'yes' to enable API (if prompted).
    # But the last command may not work all the same
    # if you don't enable billing.
    # So, follow the link to enable billing and retry last command.
    # It should end up to "**Created registry [weather-devices-registry]."

### 3) Device registration within the Cloud IoT Core project

Let’s now register the devices to the project! One at a time of course. **mos tool is really helpful for this task. **From mos tool launched in its UI or from Command Line Terminal, placed in our app1 folder, we type the following command (project id and registry name are involved, as you see):

    **# Register device with Cloud IoT Core (do it for each device!)**
    mos gcp-iot-setup --gcp-project *hello-cloud-iot-core* --gcp-region europe-west1 --gcp-registry *weather-devices-registry*

*Note: *This command could be the last one of the “provisioning script” we mentioned already twice.

This command is a mos command that will itself use gcloud commands. The device about to be registered must be connected via the serial port to our host computer because some information will be uploaded to it just like keys, MQTT bridge address, etc.

Indeed, we see on mos console that **two keys (one private, one public) are generated**. We can inspect them in app1 project folder. The private one is for ESP32 and the public one is for Google IoT Core. They are used during the authentication process involving the JSON Web Token we mentioned earlier.

![Pair of keys just generated (private and public)](https://cdn-images-1.medium.com/max/2000/1*iVJ-dbpnqmQJqwuGFC917A.png)*Pair of keys just generated (private and public)*

*Note concerning security: ***The private key shouldn’t be stored in plain text in ESP32 flash memory**. This is why we describe in the post following this one ([[link](https://medium.com/@o.lourme/gcp-cloudiotcore-esp32-mongooseos-2nd-config-state-encrypt-7c5e937e5be9)]) how to encrypt this memory. Also, **the private key file shouldn’t be stored in plain text on the host development computer**. At least, protect access to its content with a password.

When the device reboots, we see in the console that it successfully connects to the Google MQTT bridge and publishes telemetry messages (MQTT.pub() returns 1):

![Publications to MQTT bridge are successful. Nice job!](https://cdn-images-1.medium.com/max/2000/1*H9T-zF4B-HwYgO_ZdratEw.png)*Publications to MQTT bridge are successful. Nice job!*

### 4) Checking the project setup in Google Cloud Console

We head to [https://console.cloud.google.com/iot/](https://console.cloud.google.com/iot/) to check that everything was well configured:

![Project ID, registry ID and telemetry Pub/Sub topic name in Google Cloud Console](https://cdn-images-1.medium.com/max/2000/1*BeVR0exU6l-rXg8LqJvfeQ.jpeg)*Project ID, registry ID and telemetry Pub/Sub topic name in Google Cloud Console*

Clicking on the **Registry ID** weather-devices-registry reaches another screen. Clicking on “Devices” on this new screen lists provisioned devices and gives details like the last time they were seen (but this is not a live update, we have to refresh the page):

![Devices View in Google Cloud Console](https://cdn-images-1.medium.com/max/2326/1*sfa7ErglL4OqCsk2jIZqUA.png)*Devices View in Google Cloud Console*

Clicking on the **Telemetry Pub/Sub topic** name goes to **Pub/Sub console** to show the subscription we created before, *i.e.* the one related to the telemetry topic:

![Google Cloud Console (Pub/Sub) — names of topic and related subscription](https://cdn-images-1.medium.com/max/2000/1*EQ1r5SozuzjM5Jrbl4vdVg.png)*Google Cloud Console (Pub/Sub) — names of topic and related subscription*

### 5) Viewing at last some telemetry data

Now it would be nice to see the data that devices are publishing. For this, we have the subscription we already created. From any folder of the host computer, we type:

    gcloud beta pubsub subscriptions pull --auto-ack *weather-telemetry-subscription *--limit=***2***

This command ([[link](https://cloud.google.com/sdk/gcloud/reference/beta/pubsub/subscriptions/pull)]) pulls until ***2*** Pub/Sub messages from our weather-telemetry-subscription subscription. We can see data in JSON, messages ids and a list of attributes for each message. Among them the deviceId attribute is present. Unfortunately there are no timestamps, we’ll see how to get them later.

![Result of pulling telemetry messages from a subscription](https://cdn-images-1.medium.com/max/2000/1*78OYVPPUeKZWRRvlWdI4Mg.png)*Result of pulling telemetry messages from a subscription*
> If you have reached this milestone, congrats! We’re now ready to write a Firebase Cloud Function reacting to each publication to the Pub/Sub telemetry topic!

## Logging, storing and visualizing weather data with Firebase

### 1) Introduction

We’re now tackling this part of the project:

![The part we study now](https://cdn-images-1.medium.com/max/2000/1*UwR_qn8GX1b2CkMA62kT3w.png)*The part we study now*

On that diagram, we see that our project needs 3 Firebase products :

A **Firebase Cloud Function** (more exactly “a Cloud Function for Firebase”) must react to any publication to the telemetry topic in order to store the weather data of this publication to a **Firebase Realtime Database**. This storage allows weather data persistence and is used to feed a web app hosted by **Firebase Hosting**. This web app draws live plots of this weather data across time.

The good new is that it’s possible to configure all these products with one command.

### **2) Firebase configuration, GitHub repository**

We are still working on the same Google Cloud project called hello-cloud-iot-core. Firebase will just “enhance” this project with its products.

We made a GitHub repository for the Firebase aspects of our project:
[**olivierlourme/iot-store-display**
*Contribute to olivierlourme/iot-store-display development by creating an account on GitHub.*github.com](https://github.com/olivierlourme/iot-store-display)

Clone this repository in your favorite development folder and head to the newly created directory:

    **c:\_app>**git clone https://github.com/olivierlourme/iot-store-display
    **c:\_app>**cd iot-store-display

**Global Firebase configuration**

*Note:* We suppose you have **Firebase tools** installed (*i.e. *Node.js installed and npm install -g firebase-tools was run, see [[link](https://firebase.google.com/docs/functions/get-started)] for details).

Let’s perform the Firebase initializations:

    **c:\_app\iot-store-display>**firebase init 

First step is to choose Firebase products we want to use:

![Choosing Firebase products](https://cdn-images-1.medium.com/max/2000/1*FYoZkLMMrQBQDflK8Z46fQ.png)*Choosing Firebase products*

We are then prompted to associate the current directory (iot-store-display) with one of the listed Firebase projects. The problem is that our project hello-cloud-iot-core doesn’t appear in the list because before being a Firebase project it’s also a Google Cloud project! Read [Doug Stevenson](undefined)’s posts for relationships between Firebase and Google Cloud: [[link](https://medium.com/google-developers/whats-the-relationship-between-firebase-and-google-cloud-57e268a7ff6f)] and [[link](https://medium.com/google-developers/firebase-google-cloud-whats-different-with-cloud-functions-612d9e1e89cb)].

To overcome this, first we hit CTRL+C to stop this initialization process and then we go to** Firebase Console** at [https://console.firebase.google.com](https://console.firebase.google.com). We choose “Add a project”:

![Firebse Console — Add a project](https://cdn-images-1.medium.com/max/2000/1*nOcFfL_ZwGUKGe-PDnsZUQ.png)*Firebse Console — Add a project*

And we can see our project (with Google Cloud logo) and choose it:

![Fierbase Console — Even Google Cloud projects are listed.](https://cdn-images-1.medium.com/max/2000/1*QqI3k8bH5P348eAInetTOQ.png)*Fierbase Console — Even Google Cloud projects are listed.*

*Note:* You might then be asked to confirm Firebase billing plan if the Google Cloud project itself has a billing plan.

Great! We restart the Firebase initialization with firebase init command and this time our Google Cloud project hello-cloud-iot-core is listed. We choose it:

![Our **iot-store-display** directory is associated with **hello-cloud-iot-core **project.](https://cdn-images-1.medium.com/max/2000/1*qr4CKgpdnEG9UorzVv3yrw.png)*Our **iot-store-display** directory is associated with **hello-cloud-iot-core **project.*

*Note:* If you still don’t see your project you might be logged to Firebase without the correct Google account. In this case, type firebase logout followed by firebase login.

**Realtime Database configuration**

Then the wizard asks a single question about the Realtime Database and its rules: the name of the file where they will be saved. We maintain the default name. It’s more practical to have these rules in a file lying in the project directory than to go to Firebase Console as we did in past posts. We will detail these rules later.

![Name of file storing Realtime Database rules](https://cdn-images-1.medium.com/max/2000/1*YtlGoKNyjspvirVxSoXYNg.png)*Name of file storing Realtime Database rules*

**Cloud Functions configuration**

Here are the answers we made to the wizard concerning Functions Setup:

![Cloud Functions setup](https://cdn-images-1.medium.com/max/2000/1*ml7Pc1OSEgqi6dRsy5J_aA.png)*Cloud Functions setup*

Of course, we choose not to overwrite the functions/index.js file obtained from [GitHub](https://github.com/olivierlourme/iot-store-display).

**Firebase Hosting configuration**

And here are the answers we made to the wizard concerning Hosting Setup:

![Hosting setup](https://cdn-images-1.medium.com/max/2000/1*F_7qI1Norg4qjQfPhouCWw.png)*Hosting setup*

Of course we choose not to overwrite the public/index.html file obtained from [GitHub](https://github.com/olivierlourme/iot-store-display).

**Deploying (not now!)**

Later, if we want to deploy some updates we made to our 3 products, we can type globally:

    **c:\_app\iot-store-display>**firebase deploy

But if we want to deploy only, respectively:

* updated database rules,

* updated cloud functions,

* updated web app,

we type, respectively:

    firebase deploy --only database
    firebase deploy --only functions
    firebase serve --only hosting *(local deployment)* OR firebase deploy --only hosting *(remote deployment)*

### 3) Pub/Sub trigger Cloud Function

**Introduction**

In a past post, we explained that we could write **Firebase Cloud Functions** triggered on some events happening to some of the Google products.
[**Post 2 of 3. Our IoT journey through ESP8266, Firebase and Plotly.js**
*A Firebase Cloud Function appends a timestamp to each value pushed to a Firebase Realtime Database.*medium.com](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-2-14b0609d3f5e)

**Cloud Pub/Sub** is one of these products and so it is possible to trigger a function each time a message is published to a Pub/Sub topic : [[link](https://firebase.google.com/docs/functions/pubsub-events)].

So if the Firebase Cloud Function is triggered on each publication to the weather-telemetry-topic topic, watching its log will allow us to watch the telemetry topic’s activity.

The code of the Cloud Function has to store each new published data to the Firebase Realtime Database associated with our project.

**Cloud Function source code**

The beginning of the source code looks like this:

    exports.detectTelemetryEvents = functions.pubsub.topic('weather-telemetry-topic').onPublish(
        (message, context) => {...

The full Cloud Function source code lies in the file named index.js. This file is in the functions folder of our iot-store-display directory on [GitHub](https://github.com/olivierlourme/iot-store-display). It is fully commented, so run and study it, it’s short and not complicated.

**Cloud Function deployment**

It’s time to deploy the Cloud Function:

    **c:\_app\iot-store-display>**firebase deploy --only functions

**Cloud Function validation**

Once Cloud Function is deployed, we can watch the Cloud Function logs and among other things, we’ll see the results of the console.log(`Device=${deviceId}...) we wrote at the end of index.js.

Where to see those logs? We have two opportunities:

* in Firebase Console ([https://console.firebase.google.com](https://console.firebase.google.com)):

![Firebase Console — Firebase Cloud Functions logs](https://cdn-images-1.medium.com/max/3082/1*SH9086pyf8yyuEnyCDu0UQ.png)*Firebase Console — Firebase Cloud Functions logs*

* in Google Cloud Console ([https://console.cloud.google.com/functions/](https://console.cloud.google.com/functions/)):

![Google Cloud Console — Access to **Cloud Function logs** and to **Cloud Function deletion**](https://cdn-images-1.medium.com/max/2748/1*D3QmCuAsqE5jUkuIyPTB-w.png)*Google Cloud Console — Access to **Cloud Function logs** and to **Cloud Function deletion***

We prefer this latter solution, as logs are clearer:

![Google Cloud Console- Cloud Functions logs](https://cdn-images-1.medium.com/max/2956/1*kgpwdupeMVLKQ2msHpO4cQ.png)*Google Cloud Console- Cloud Functions logs*

Concerning storage, here is what lies in Firebase Realtime Database after each device has made 2 telemetry data publication. Data is of course sorted by device as we specified it in index.js:

![Firebase Console — Realtime Database after 2 telemetry data publications per device](https://cdn-images-1.medium.com/max/2022/1*gJX7raPOkRvRZzAHhQC14Q.png)*Firebase Console — Realtime Database after 2 telemetry data publications per device*

*Note: *Don’t forget to **delete your Cloud Function** on Google servers if you don’t use it, otherwise you might either reach the invocation quota or pay for service you don’t use, as indicated here: [[link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-2-14b0609d3f5e)]. Function deletion is to be performed on Google Cloud Console (see “Delete”, 3 screenshots above).

*Note:* The Cloud Function has admin rights over the database, whatever is the content of the database.rules.json file. At this step, the database.rules.json file can still be very restrictive. Don’t forget to deploy them, once edited.

    {
      "rules": {
        ".read": false,
        ".write": false
      }
    }

### 4) A web app using Firebase and plotlys.js to visualize weather data

**Introduction**

*Note:* We’re now building a “homemade” (and satisfying) data visualization solution. For enhanced UI (dashboard, etc.), maybe you should investigate Data Studio we already mentioned elsewhere in this post.

We focus on **building a small web** app, hosted by **Firebase Hosting**. This web app **lively plots** the data stored in the Firebase Realtime Database. We used [plotly](undefined) ([https://plot.ly/javascript/](https://plot.ly/javascript/)) for the plotting library. We are familiar with that work as we already undertook a similar one in a previous post:
[**Post 3 of 3. Our IoT journey through ESP8266, Firebase and Plotly.js**
*A web app hosted by Firebase Hosting susbscribes to the data stream coming from a Firebase Realtime Database and plot…*medium.com](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-3-644048e90ca4)

What’s different today is that we have to:

* draw several charts: temperature *vs* time and humidity *vs* time,

* inside each chart, we have one plot per device.

**Database rules & devices-ids node**

Concerning the database rules, what should be now the database.rules.json file? The device-telemetry node needs to be **read **by the web app. And if you looked attentively at the Realtime Database screenshot given a few screenshots before, there is another node called devices-ids.

![Firebase Console — Detail of devices-ids node in Realtime Database](https://cdn-images-1.medium.com/max/2000/1*Si6-O12MUJXpzN3qyZyBVg.png)*Firebase Console — Detail of devices-ids node in Realtime Database*

You need to create **manually in the Firebase Console** this devices-ids node and fill it appropriately for the web app to work properly. It is a simple mean to declare to the web app the devices we want plots for and also to give aliases to the devices. Its role and necessity are fully explained in comments of the public/script.js file given in [GitHub](https://github.com/olivierlourme/iot-store-display).

*Note:* An improvement could be a form (accessed through authentication) that, once fulfilled, calls a script to generate this devices-ids node.

This devices-ids node also needs to be read by the web app. So the database.rules.json file should eventually become:

    {
      "rules": {
        "devices-ids": {
          ".read": true,
          ".write": false
        },
        "devices-telemetry": {
          ".read": true,
          ".write": false
        }
      }
    }

These new rules, once edited and saved, must be deployed with:

    **c:\_app\iot-store-display>**firebase deploy --only database

**Web app source code**

The web app source code lies in the public directory of our hello-cloud-iot-core folder or in [GitHub](https://github.com/olivierlourme/iot-store-display). The content of the folder, especially script.js, is fully commented so you know where to study (and improve!) it.

*Note: *we have only two devices for this demo but the source code is okay for *x* devices as long as you declare them in the devices-ids node.

**Web app local deployment and validation**

For testing purposes, Firebase Hosting can launch a local live server:

    **c:\_app\iot-store-display>**firebase serve --only hosting

We head to http://localhost:5000 and we’re happy to get this:

![At last the charts we wanted!](https://cdn-images-1.medium.com/max/2000/1*02UfatBLkuqCD12-JNBgjw.gif)*At last the charts we wanted!*

**Web app remote deployment**

At last, Firebase offers us the hosting of our web app and an access to it via https:

    **c:\_app\iot-store-display>**firebase deploy --only hosting

We quickly get the public URL of our web app : [https://hello-cloud-iot-core.firebaseapp.com](https://hello-cloud-iot-core.firebaseapp.com)

![Web app deployment is complete!](https://cdn-images-1.medium.com/max/2000/1*EZjq0siYgb0K6sMaFHlk4Q.png)*Web app deployment is complete!*

*Note: *If you have your own domain, you can connect your Firebase web app to it. See [[link](https://firebase.google.com/docs/hosting/custom-domain)].

## Conclusion

In this post we discovered how to combine **ESP32**, **Mongoose OS** and **Cloud IoT Core, **obtaining a serious, secure and **professional IoT project**. Now that we know, it can go really fast to provision 10, 100… 1000 devices acquiring weather data all over an area, as long as they can get a Wifi connection. Now, devices are centrally managed, it is easy to provision and monitor them. But we can go further!

Indeed, in addition to this post, there is a second one ([[link](https://medium.com/@o.lourme/gcp-cloudiotcore-esp32-mongooseos-2nd-config-state-encrypt-7c5e937e5be9)]). Inside it:

* We’ll focus on ESP32 flash **memory encryption**, to achieve a fully secured system.

* We’ll see how to use the config special topic, allowing us to **trig an action on the device from the Google Cloud Console**.

* We’ll see how to use the state special topic, allowing **the device to communicate to Google Cloud Console informations about its current state**.

We hope you enjoyed this really long post and that you learnt something! Don’t hesitate to ping me if you have any questions or improvement suggestions…
