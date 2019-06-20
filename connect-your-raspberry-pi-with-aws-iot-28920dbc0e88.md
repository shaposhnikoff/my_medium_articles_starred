
# Connect your Raspberry Pi with AWS IoT



## Introduction

IoT or the Internet of Thing is a hot topic in these days even the word/concept has been in there in the industry so long time.

In this article I will work through the below list of sub topics, so that end of the session you would be able to connect you first IoT device to the AWS IoT with having secure connection over TLSv1.2 by using the Python.

* What would you need in terms of hardware and software/services.

* How to install NOOBS OS in the Raspberry Pi.

* AWS IoT Configurations.

* Setting up Raspberry Pi to connect with AWS IoT.

## What would you need in terms of hardware and software services

### Hardware:

As I’m going to use the Pi 3 model B v1.2, it has the build in WiFi capability so we don’t need any additional internet connection method. Also I’ll be connecting my TV to the Pi HDMI output rather than installing any SSH agent in the Pi. If you are preferred to install agent in the Pi you won’t need an output device.

![1. Raspberry Pi device.](https://cdn-images-1.medium.com/max/4714/1*4WZfBOHQ4S1CdkTJxhqN8w.png)*1. Raspberry Pi device.*

* Raspberry Pi device (I’ll be using Pi 3 model B v1.2 but any Pi will work).

* Micro SD card.

* HDMI cable.

* HDMI output compatible monitor or a TV.

* Keyboard and mouse.

* 5v output charger.

### Software and services:

* [NOOBS OS.](https://www.raspberrypi.org/downloads/noobs/)

* OpenSSL version 1.0.1+ (TLS version 1.2)

* AWS IoT Service.

## How to install NOOBS OS in the Raspberry Pi.

You can download the installation in [***here](https://www.raspberrypi.org/downloads/noobs/)***. When I’m preparing this article I used noobs offline version 2.7.0 and this OS contains the Python version 3.5.3 as well.

After downloading the OS zip file just extract it to separate directory and copy the whole file list which you would see as in the below screenshot to the SD card root.

![2. NOOBS OS file list.](https://cdn-images-1.medium.com/max/2000/1*fJkiJfca5-Ool1eKfNQ7aw.png)*2. NOOBS OS file list.*

Now insert the SD card to the Pi and just boot the device by connecting the charger to your device. You will be prompted to install the OS form the SD card itself. You can follow the wizard as it prompt.

I just explained the steps very briefly if you need more help please look in to this video. This video explains the things in more detail manner.

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/juHoJYX86Dg" frameborder="0" allowfullscreen></iframe></center>

Now we do have a Pi with noobs installed. The Python 3.5.x is also installed in our device as it came with the OS.

## AWS IoT Configurations

In the AWS IoT you need to do the several steps to connect our device to their IoT service such as creating a thing, assigning certificates and polities. I have already published an article on those steps. [***Please do read and follow the section of “AWS IoT Configurations” in this article.](https://medium.com/@sajithswa/connecting-aws-iot-via-the-wso2-ebs-mqtt-inbound-endpoint-publishing-events-to-the-wso2-das-and-do-e90616412113)***
> Note: As I explained in the above given article, in here I’m also using the same public key, private key, certificate and the same topic which is the “demo-topic”.

## Setting up Raspberry Pi to connect with AWS IoT.

To connect with the AWS IoT service we are going to use the AWS IoT Python SDK which is build top of the the [Paho MQTT Python client library](https://eclipse.org/paho/clients/python/). As we are to use MQTT connection over TLSv1.2 we should use the port as 8883.

Make sure that you are having the below minimum requirement in your Raspberry Pi to use the SDK,

* Python 2.7+ or Python 3.3+ (With noobs-2.7.0 we are having Python 3.5.3).

* OpenSSL version 1.0.1+ (With noobs-2.7.0 we are having OpenSSL version 1.1.x).
Verifying OpenSSL version,
Open the Python shell and type the below 2 commands;

![4. Checking the SSL version.](https://cdn-images-1.medium.com/max/2000/1*rnJLMYLDeLLA94Bar0R3bQ.png)*4. Checking the SSL version.*

After the verification we are going to install the AWS Python SDK in our Pi. To do so you can download the Python SDK form here: [https://s3.amazonaws.com/aws-iot-device-sdk-python/aws-iot-device-sdk-python-latest.zip](https://s3.amazonaws.com/aws-iot-device-sdk-python/aws-iot-device-sdk-python-latest.zip)

Unzip the package and install the SDK by running the setup.py.

![5. Installing AWS Python SDK.](https://cdn-images-1.medium.com/max/2000/1*2OrNXOham_t2znfzS1RGAQ.png)*5. Installing AWS Python SDK.*

    sudo python setup.py install

### Implementing the publisher.py

Now we are ready to implement our publisher program. Just follow the below steps to do so;

* Create a new directory named as **demo **as a working directory in the **/home/pi** directory

* Copy the **AWSIoTPythonSDK** directory which you can find in the downloaded SDK to the **demo** directory.

* Create a new directory called **demo-cert** in the **demo** directory.
- Copy the AWS Root CA certificate to this directory. (I renamed it as **aws-root-cert.pem**)
- Copy the certificate file which we associated with the thing, to this directory. (I renamed it **as iot-cert-.pem.crt**)
- Copy the private key file which we associated with the thing, to this directory. (I renamed it as **private-key.pem.key**)

* Open the Python 3.5.3 shell by clicking the home -> Programming -> Python 3 (IDEL)

* Create a new file, copy the below source code block to it and save the file by naming it as **publisher.py** in our working directory which is **demo** directory.

    from AWSIoTPythonSDK.MQTTLib import AWSIoTMQTTClient
    import logging
    import time
    import argparse
    import json
    
    host = "YOUR-THING-END-POINT"
    certPath = "/home/pi/demo/demo-cert/"
    clientId = "sajith-pi-demo-publisher"
    topic = "demo-topic"
    
    # Init AWSIoTMQTTClient
    myAWSIoTMQTTClient = None
    myAWSIoTMQTTClient = AWSIoTMQTTClient(clientId)
    myAWSIoTMQTTClient.configureEndpoint(host, 8883)
    myAWSIoTMQTTClient.configureCredentials("{}aws-root-cert.pem".format(certPath), "{}private-key.pem.key".format(certPath), "{}iot-cert.pem.crt".format(certPath))
    
    # AWSIoTMQTTClient connection configuration
    myAWSIoTMQTTClient.configureAutoReconnectBackoffTime(1, 32, 20)
    myAWSIoTMQTTClient.configureOfflinePublishQueueing(-1)  # Infinite offline Publish queueing
    myAWSIoTMQTTClient.configureDrainingFrequency(2)  # Draining: 2 Hz
    myAWSIoTMQTTClient.configureConnectDisconnectTimeout(10)  # 10 sec
    myAWSIoTMQTTClient.configureMQTTOperationTimeout(5)  # 5 sec
    myAWSIoTMQTTClient.connect()
    
    # Publish to the same topic in a loop forever
    loopCount = 0
    while True:
        message = {}
        message['message'] = "demo-topic-sample-message"
        message['sequence'] = loopCount
        messageJson = json.dumps(message)
        myAWSIoTMQTTClient.publish(topic, messageJson, 1)
        print('Published topic %s: %s\n' % (topic, messageJson))
        loopCount += 1
        time.sleep(10)
    myAWSIoTMQTTClient.disconnect()

Now the directory should be like the below screenshot.

![6. Working directory.](https://cdn-images-1.medium.com/max/2000/1*63c5p21Wdc0mfdbHxpDyqQ.png)*6. Working directory.*

### Executing the publisher and testing

Navigate to the publisher.py file and click the **Run -> Run Module** menu option. Python shell output will be opened when you hit the Run Module button. In this shell output we can see our JSON formatted message is been getting published to the AWS IoT topic.

![7. Running the publisher and output.](https://cdn-images-1.medium.com/max/2732/1*evc9TGz8s6T1CPRZZ2qvUw.png)*7. Running the publisher and output.*

We can use AWS IoT Test service as a subscriber to this topic. Navigate to the AWS IoT Test service, click on the **Subscribe to a topic** and named the topic as **demo-topic**. When you hit the “**Subscribe to topic”** button we can see what ever the messages are been published by our Raspberry Pi sample publisher program to the **demo-topic** are getting subscribed by this test service.

![8. Subscribing via the AWS IoT Test service.](https://cdn-images-1.medium.com/max/2034/1*_Rbk7NGAfpU2QX2WEyclmw.png)*8. Subscribing via the AWS IoT Test service.*

## Conclusion

In this article we have discussed how to connect Raspberry Pi with AWS IoT over the TLSv1.2. Using sample publisher program which wrote using the Python, we have published the JSON formatted MQTT message to the AWS IoT and tested the subscription using AWS IoT Test service.
