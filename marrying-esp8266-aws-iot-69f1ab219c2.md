Unknown markup type 10 { type: [33m10[39m, start: [33m255[39m, end: [33m342[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m309[39m, end: [33m322[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m217[39m, end: [33m234[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m58[39m, end: [33m62[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m111[39m, end: [33m115[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m120[39m, end: [33m128[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m165[39m, end: [33m177[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m356[39m, end: [33m360[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m158[39m, end: [33m166[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m241[39m, end: [33m249[39m }

# Marrying ESP8266 & AWS IoT

Summary: A project exploring the possibilities & complexity of associating low-cost IoT devices with the Amazon Web Services cloud. This project particularly focussed on security, all the while considering the possibility of automatic device provisioning. Part 1 addresses connecting an ESP8266 device to AWS. Part 2 develops features on top of that platform.

## In**t**roduction

AWS have continued to grow their IoT service offering rapidly, concentrating heavily on layering higher-level services that require very little backend code to be written to record and act upon sensor data in the cloud. [Their latest re:Invent announcements](https://aws.amazon.com/new/reinvent/) show several new IoT products around device management and actioning of data, alongside complimentary releases like DynamoDB On-Demand and AWS Timestream. Having never used an IoT service before, I wanted to start from the basics to get an idea of the fundamental building blocks, so I only use the IoT Core product here. With the product subtitle “*connect devices to the cloud*” comes the capability to provision and manage “things” in a secure manner, and set up simple triggers to other AWS services such as writing to databases, queues, and streams. There should be just enough instructions here to replicate what I’ve done, but some googling to fill in the gaps or gain an understand of the technologies used should be expected.

## The Hardware

![ESP8266 Boards: LoLin NodeMCU v3 & WeMos D1 mini](https://cdn-images-1.medium.com/max/8064/1*Ptm87ophtmwTf6xPLENVRA.jpeg)*ESP8266 Boards: LoLin NodeMCU v3 & WeMos D1 mini*

Low cost is always a driver for my projects. I had a pile of ESP8266-based boards around the house, such as the NodeMCU v3 and D1 Mini pictured above. The latter can be found for ~£1.50 on AliExpress, and comes with plenty of GPIO pins including an ADC and most importantly WiFi. If you don’t know about this ESP8266 chip already, it’s a 32-bit microcontroller that many dev/IoT boards have been created around, and has fantastic support for/by developers in the Arduino and MicroPython community. Its recent successor chip is the ESP32, although the prices for that are currently significantly higher. It’s a great all-rounder for these kinds of hobbyist projects.

For data to send to AWS, I used any sensors I had to hand while I waited for an air quality sensor to be delivered, so I started with cheap, analog light and temperature sensors. Air quality, especially indoors, is something I’ve recently become interested in, having spent time in Bangkok when the government was handing out free masks and spraying water into the air in an attempt to reduce the dangerous level of airborne particulates (it’s much better now, not due to those water cannons).

## What’s in a Cloud

![Just one click will make it better](https://cdn-images-1.medium.com/max/2026/1*eGRGgXv4epWdqsei7_pFFA.png)*Just one click will make it better*

Setting up the AWS environment for these devices is pretty simple. Starting in the [AWS IoT Core Console](https://eu-west-1.console.aws.amazon.com/iot/home), firstly you create a new “thing” and optionally place it into some self-defined management groups, and then you are immediately offered the one-click certificate generation option above. An individual [X.509 certificate](https://docs.aws.amazon.com/iot/latest/developerguide/x509-certs.html) per device is the recommended way of interacting with AWS IoT services from devices, offering the ability to burn the private key into the device upon enrolment that is then never transferred across the internet alongside requests, a security win. Download the private key and certificate for each device, and also a root CA (we’ll revisit this later). Make sure to hit that activate button so the certificate can be used.

![Certificate provisioning](https://cdn-images-1.medium.com/max/2016/1*NfEl6M97HKDFmn-H_-vmTg.png)*Certificate provisioning*

Next up is to attach a policy to the certificate, authorising the authenticated device to perform IoT actions on IoT resources. Below is the policy I used, which can be shared between multiple certificates/devices and allows them to connect and publish messages to any IoT topic.

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "iot:Connect",
          "Resource": "arn:aws:iot:eu-west-1:ACCOUNT_ID:client/*"
        },
        {
          "Effect": "Allow",
          "Action": "iot:Publish",
          "Resource": "arn:aws:iot:eu-west-1:ACCOUNT_ID:topic/*"
        }
      ]
    }

The resource asterisk after the client and topic is acceptable while developing, but should be further narrowed if productionising this setup. For example, the following change ensures that the device connecting is using the name of the registered thing: “Resource”: “arn:aws:iot:eu-west-1:ACCOUNT_ID:client/${iot:Connection.Thing.ThingName}”. The topic resource should be constrained to only the topic(s) that the device is publishing to.

## Client Software

[MQTT](https://en.wikipedia.org/wiki/MQTT) is a lightweight pub/sub (publish-subscribe) protocol that’s ideal for low-power devices operating in low-bandwidth environments. The template script I used for this is available on GitHub [here](https://github.com/copercini/esp8266-aws_iot/blob/272a98cc91a42b5f0927733252aab3892b5fe82d/examples/mqtt_x509_DER/mqtt_x509_DER.ino) (n.b. link is a permalink to the commit I used, check master for improvements & fixes). I’m using the example [mqtt_x509_DER](https://github.com/copercini/esp8266-aws_iot/tree/272a98cc91a42b5f0927733252aab3892b5fe82d/examples/mqtt_x509_DER) which requires the certificate files to be uploaded to the device’s flash rather than storing them inline with the script. This allows the same script to be used by multiple devices, that each read the required files from their internal flash storage.

To use this you’ll need the latest [Arduino IDE](https://www.arduino.cc/en/Main/Software) installed locally, with the ESP8266 plugin installed; instructions for installing that are [here](https://github.com/esp8266/Arduino#installing-with-boards-manager).

The following commands will convert the downloaded device certificate files to the correct format for this script. Coming back to the root CA mentioned earlier, Amazon provides several possible roots of which I found AmazonRootCA1.pem to work, I advise using use that one. Download it from [here](https://docs.aws.amazon.com/iot/latest/developerguide/managing-device-certs.html).

    $ openssl x509 -in xxx-certificate.pem.crt -out cert.der -outform DER 
    $ openssl rsa -in xxx-private.pem.key -out private.der -outform DER
    $ openssl x509 -in AmazonRootCA1.pem -out ca.der -outform DER

These DER-format files can be copied into a folder called data that will sit alongside your Arduino code, which can be uploaded to each device using [this tool](https://github.com/esp8266/arduino-esp8266fs-plugin).

Now the certificates are saved onto the device, three values from the example script will need filling in. The ssid and password for your WiFi access point, and the AWS_endpoint which is the address of the MQTT broker for your AWS account in a specific region, and can be found in the HTTP section of the Interact tab of any of your registered [things](https://eu-west-1.console.aws.amazon.com/iot/home?region=eu-west-1#/thinghub). The -ats endpoints are fairly new, and are used with the Amazon root CA rather than the older Verisign one, details on that development are [here](https://aws.amazon.com/about-aws/whats-new/2018/08/aws-iot-core-adds-new-endpoints-serving-amazon-trust-services-signed-certificates-to-help-customers-avoid-symantec-distrust-issues/).

![Finding the AWS_endpoint address](https://cdn-images-1.medium.com/max/2000/1*bd2tDNO0Eu6AGjpvF7nxvg.png)*Finding the AWS_endpoint address*

At this point you should have everything required for the device to publish a test message to an IoT topic. The example script has a hard-coded topic name of [outTopic](https://github.com/copercini/esp8266-aws_iot/blob/272a98cc91a42b5f0927733252aab3892b5fe82d/examples/mqtt_x509_DER/mqtt_x509_DER.ino#L83). Handily the IoT Console provides an [MQTT client](https://eu-west-1.console.aws.amazon.com/iot/home#/test), so you can subscribe to outTopic and watch the test messages flow in. 🤞

A few notes when bringing this all together:

* To save time while uploading to these devices (both code and especially SPIFFS), set the upload baud rate as high as does not produce errors. That was 921600 for my devices, up from the default of 115200, saving a lot of time while iterating.

* If you get errors when trying to upload the certificates to the device flash, you may not have SPIFFS enabled from the Arduino > Tools menu. Any of the SPIFFS storage size values will work, the certificates take up significantly less than 1 MB. All my devices had 4 MB of flash. If you ever change the board type then this setting is lost; I wiped my certificates several times while re-uploading code while I hadn’t noticed this, resulting in connection errors.

![Enabling SPIFFs for the certificate upload](https://cdn-images-1.medium.com/max/2000/1*3tL4jux7zc90yDXy1fXXOw.png)*Enabling SPIFFs for the certificate upload*

* Once the script is uploaded you can view the output on the Serial Monitor. For debugging any potential issues with the certificates and policies, I used the command on [this page](https://docs.aws.amazon.com/iot/latest/developerguide/diagnosing-connectivity-issues.html). This checks the chain of trust between the client certificate and root CA, and checks that a TLS connection can be established. Potential issues could arise from the certificate not being activated in the AWS Console, or the device clock not having synced with the NTP server which prevents certificate validation.

## Progress so far

Hopefully your device is now connected to your WiFi network, has authenticated itself with AWS, has the permissions to connect and publish to an IoT topic, and you’re seeing messages in MQTT client like this.

![Successful message publishing from the device](https://cdn-images-1.medium.com/max/2000/1*VlbmBLYZO8Fz-zc7xbS7eA.png)*Successful message publishing from the device*

Now would be a great time if you haven’t already, to study the example code to see what’s going on. There are sections that aren’t being used yet, like the callback for when the device receives a message, you can ignore that. You’ll see how the certificate files are read from the filesystem, and how the device syncs its time to an NTP server before the TLS connection is attempted. The business logic is very simple right now, simply initiating & checking the pub/sub connection, and publishing messages.

You’ve successfully sent data from a sub-$2 device to the cloud! 🎉

In **Part 2** of this project I build on top of the hardware/software platform we’ve set up. That includes persisting the submitted data, and making the devices easier to develop with. Topics covered:

* Connect a sensor to the analog input pin.

* Send JSON-formatted data to the IoT topic.

* Use AWS IoT Rules to persist data to DynamoDB, in a way that makes querying it as a time series efficient.

* Add the ability to OTA-update devices on the local network.

* Allow devices running different sensors to share the same codebase.
