
# AWS IoT Hands-On — A Practical Tutorial

If you love fiddling around with electronics, C/C++ programming and making small things, then IoT is for you and this tutorial might help you get a glimpse of AWS IoT and what it can do (hint: absolutely everything!)

![STM32 Discovery kit IoT node in front of the data sent to AWS IoT cloud](https://cdn-images-1.medium.com/max/2000/1*u1_bP7kJm9BuTHMvPsYd-A.png)*STM32 Discovery kit IoT node in front of the data sent to AWS IoT cloud*

## In short what this is about

In this article or tutorial I will explain how to connect an ST Microelectronics **STM32 Discovery kit IoT Node** with **AWS IoT** using **ARM Mbed OS** to display temperature, humidity and air pressure on an S3 website hosted through CloudFront.

## What is AWS IoT about?

In order to understand this article it is absolutely recommended to have at least an idea of what AWS, the Amazon cloud, does. Beside all these large scale services that AWS offers, it also offers a nice set of IoT services. These include, but are not limited to, the following.

* AWS IoT Core for managing IoT devices or device fleets

* AWS Greengrass *(Won’t cover it in this article, but it’s awesome!)*

* AWS IoT 1-Click — the Dash Button’s services

Many other services are listed in the service overview of your AWS console and they mainly all bring you to the AWS IoT user interface. The interface has nicely put all services together, so it is hard to estimate what is a separate service or not. This allows to manage all AWS IoT services in one place.

![The AWS IoT User Interface — A view of the Monitor dashboard](https://cdn-images-1.medium.com/max/2676/1*BaMHzbLKDSAVb38j56v_UQ.png)*The AWS IoT User Interface — A view of the Monitor dashboard*

In simple terms the AWS IoT service is an MQTT message gateway that can send and receive MQTT messages to and from devices or things as I will call them further on. Although this doesn’t sound overly exciting, the features AWS has put behind the gateway certainly are. You can manage things, monitor them, test them and also send and receive messages yourself using the AWS IoT interface’s test suite. The test suite is a fully-loaded web-based MQTT messaging client. Cloud integration is done through the “Act”-section of AWS IoT which does all the cloud magic. You can run SQL on MQTT messages, import the data into S3, Dynamo, Kinesis or other services and have Lambdas handle each message.

## Starting off with this Thing — STM32L4

In order to start using AWS IoT, the Cloud for the Internet Of Things, you need of course a “**Thing**”. A Thing in the case of IoT is a small microprocessor with communications either on the chip or on the board. The Thing I got myself is an STM32 Discovery kit IoT node that comes with an ARM Cortex-M4 core 80 Mhz STM32L475VGT6 MCU equipped with WiFi, NFC, BLE and Sub-Ghz plus tons of sensors directly on the board. I got one at around $40.

![STMicroelectronics STM32 Discovery kit IoT Node — connected to my Mac](https://cdn-images-1.medium.com/max/2000/1*oByXOFJ5i_rG787smnf8Pg.png)*STMicroelectronics STM32 Discovery kit IoT Node — connected to my Mac*

When connecting the STM32 to your computer through the STLink USB-Port (it has two), the Thing will pop up as a USB storage device and also a serial port on your system *(I used a Mac)*. You can certainly flash it with Amazon [FreeRTOS](https://docs.aws.amazon.com/freertos/latest/userguide/getting_started_st.html) operating system, but that seemed a little complicated to me so I sticked to the **ARM Mbed OS** that was installed on the STM32 already. Also because I found the Mbed OS easy to program using the Online Compiler from ARM — it’s a quite impressive web-based C/C++ compiler. Although often a little slow and I experienced a couple of minor issues with it which were mainly unexpected error messages. Before you go ahead I highly recommend upgrading the firmware of your board using the [STSW-LINK007](https://www.st.com/content/st_com/en/products/development-tools/software-development-tools/stm32-software-development-tools/stm32-programmers/stsw-link007.html) software from ST.

You start off by creating yourself a free account on [os.mbed.com](https://os.mbed.com) and launch the online compiler with an example program. There are several ones listed on the [DISCO-L475VG-IOT01A](https://os.mbed.com/platforms/ST-Discovery-L475E-IOT01A/) page.

![The Mbed OS Online Compiler](https://cdn-images-1.medium.com/max/2710/1*G1xHy7vfzVRrLgplWwRX8Q.png)*The Mbed OS Online Compiler*

There is no one working example for connecting the STM32 with Mbed OS to the AWS IoT system, but I found the article “[*AWS IoT from Mbed OS device](https://os.mbed.com/users/coisme/notebook/aws-iot-from-mbed-os-device/)*” from [Osamu Koizumi](https://github.com/coisme) on mbed.com. He describes on how to connect to AWS IoT on Mbed through Ethernet with a K64F from NXP. Much of his work was absolutely helpful in achieving what I did, however I had to write the code myself using examples from ST, AT&T and his code. In order to save you hours of your life *(it took me hours!)*, I published my code on GitHub:

[https://github.com/jankammerath/L475VG-IOT01A-Mbed-AWS-IoT](https://github.com/jankammerath/L475VG-IOT01A-Mbed-AWS-IoT)

You can import the code directly into the Mbed online compiler by using the following link when you are already logged in to mbed.com with your account: [**→ Import L475V-IOT1A-Mbed-AWS-IoT program to Mbed.com](https://os.mbed.com/compiler/#import:https://github.com/jankammerath/L475VG-IOT01A-Mbed-AWS-IoT)**

The code will then show up in your Mbed online compiler. All you need to do is add your AWS IoT endpoint, certificate and private key to the “[MQTT_server_setting.h](https://github.com/jankammerath/L475VG-IOT01A-Mbed-AWS-IoT/blob/master/core/MQTT_server_setting.h)” file in the “*core*”-directory. Your Wifi’s SSID and password need to be inserted into the [mbed_app.json](https://github.com/jankammerath/L475VG-IOT01A-Mbed-AWS-IoT/blob/master/mbed_app.json) file. We will come back to that point later in this article.

## Creating your first Thing in AWS IoT

In order to connect your STM32L4 device to AWS IoT, you need to create it as a Thing in the AWS console. You can easily get to the interface by picking “*IoT Core*” from the “*Services*”-menu at the top. In the left menu under “*Manage*” and then “*Things*” you will see a “*Create*” button on top right.

![The AWS console’s “Creating AWS IoT things” screen](https://cdn-images-1.medium.com/max/2314/1*GZ50ID4acKzm0r0OHyTUpw.png)*The AWS console’s “Creating AWS IoT things” screen*

You choose to “*Create a single thing*” for the beginning and also only need to add a name. If you wish you can also create a group, but for this first start it’ll be of no effect. It’s sufficient to just enter a name and hit “*Next*”.

![“Add your device to the thing registry” screen on AWS IoT](https://cdn-images-1.medium.com/max/2290/1*w5YBGERkuUB10vlYsgddLA.png)*“Add your device to the thing registry” screen on AWS IoT*

Right after creating the Thing, AWS IoT will ask for the type of certificate that you want to use. You can use the “*One-click certificate creation*” which is also recommended. This way AWS IoT will create the certificates for you.

![AWS IoT Thing certificate successfully created](https://cdn-images-1.medium.com/max/2280/1*9_5RdG23hKYEq0LIzC01rA.png)*AWS IoT Thing certificate successfully created*

**Download all the files** and then make sure to activate the certificate by hitting the “Activate” button as otherwise you will not be able to authenticate your messages from the device to AWS IoT. Afterwards you need to attach a policy. For the first start you can create your Thing without a policy attached. In order to create one, you chose “*Secure*” on the left hand menu in the main screen and then “*Policies*” where you’ll find a “*Create*” button.

![“Advanced mode” of the “Create a policy” screen](https://cdn-images-1.medium.com/max/2238/1*0De9bsNfdZYuUxX049GPaw.png)*“Advanced mode” of the “Create a policy” screen*

Make sure to pick advanced mode to get the policy statement editor to insert your policy document. Insert the following policy document into the editor.

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "iot:Connect",
          "Resource": "*"
        },
        {
          "Effect": "Allow",
          "Action": "iot:Publish",
          "Resource": "*"
        },
        {
          "Effect": "Allow",
          "Action": "iot:Subscribe",
          "Resource": "*"
        },
        {
          "Effect": "Allow",
          "Action": "iot:Receive",
          "Resource": "*"
        }
      ]
    }

Clicking on “*Certificates*” inside the “*Secure*” section, then choosing the certificate, you just created, allows you to attach a policy by hitting the “*Actions*” menu on the top right and then picking “*Attach policy*”. **You have now successfully created your Thing**.

### Adding the Thing certificate and key to the program

You can now insert the contents of your certificate and key file into the corresponding section of the “[MQTT_server_setting.h](https://github.com/jankammerath/L475VG-IOT01A-Mbed-AWS-IoT/blob/master/core/MQTT_server_setting.h)” configuration file of the program.

    const char MQTT_SERVER_HOST_NAME[] = "INSERT_THING_ENDPOINT_HERE.iot.eu-central-1.amazonaws.com";
    
    const char* SSL_CLIENT_CERT_PEM = "-----BEGIN CERTIFICATE-----\n"
    "INSERT YOUR THING CERTIFICATE HERE!\n"
    "-----END CERTIFICATE-----\n";
    
    const char* SSL_CLIENT_PRIVATE_KEY_PEM = "-----BEGIN RSA PRIVATE KEY-----\n"
    "INSERT YOUR THING PRIVATE KEY HERE!\n"
    "-----END RSA PRIVATE KEY-----\n";

Your Thing endpoint, to which the STM32L4 will send the MQTT messages to, can be found in the “*Settings*” section of the AWS IoT console. Simply copy and paste it into the configuration file.

![“Settings” screen with the AWS IoT endpoint domain name](https://cdn-images-1.medium.com/max/2388/1*0QNJGGdr3UnT9pv772AtDg.png)*“Settings” screen with the AWS IoT endpoint domain name*

You have now pretty much created all the required configuration on AWS IoT and your program code in the Mbed online compiler is ready to compile.

### Compile your IoT program for the STM32L4

Back in your Mbed online compiler with the program code open, the certificate and key inserted as well as the Wifi configuration, you can now compile the program code for your STM32L4 board.

![Mbed online compiler while compiling the program code](https://cdn-images-1.medium.com/max/2420/1*vh8KXiJMfmI5HCLM6rqHog.png)*Mbed online compiler while compiling the program code*

Once finished compiling, the Mbed online compiler will provide you with a compiled bin-file to copy onto your device. Do not just directly download it onto your device, but keep it somewhere on your disk for later use.

### Run the program on your STM32L4

Your STM32L4 will show up as a USB mass storage device on your system as well as a serial interface. I’m using a Mac on which I start the terminal and then use screen to connect to the USB serial interface.
> screen /dev/tty.usbmodem1421410 115200

The name of your interface might be different. Windows users will have to use their serial port (e.g. named “*COM4*”) using software like [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) to connect. Linux users can use screen as well and also will find their serial device for the STM32L4 under “*/dev*”. Once connected to the serial, you can copy the bin-file to the USB mass storage device that represents your STM32L4.

![STM32L4 USB console debug information](https://cdn-images-1.medium.com/max/2000/1*7xUPpxyBv_aWYCgF3b_wSQ.png)*STM32L4 USB console debug information*

The program outputs the message for each message that it publishes to AWS IoT. I put a 60 seconds timeout into program which means that an MQTT message is sent to AWS IoT every minute.

### Checking your MQTT messages in the AWS IoT monitor

When opening the AWS IoT console’s monitor you should see that connections are being made and that messages are received by AWS IoT.

![AWS IoT Monitor showing a chart with incoming MQTT messages](https://cdn-images-1.medium.com/max/3840/1*HD4PGUQAPAe9MmB9Rl0m5Q.png)*AWS IoT Monitor showing a chart with incoming MQTT messages*

Note that the program on the STM32L4 connects and authenticates to AWS IoT once when it starts and then sends a message every minute. This also means that you will only see one connect, but multiple messages which is the fundamental way MQTT is designed and works.

## Act — Doing things with the Thing’s things

As MQTT messages are now flowing into AWS IoT we need to do something with the Thing’s data. If we stop here, then AWS IoT will just discard all the messages coming in and not actually do anything with them. When looking around in the AWS IoT console, you might have seen a section called “*Act*”. Hit the “Create” button to create a new rule for our messages.

![AWS IoT Rule to forward message to Lambda](https://cdn-images-1.medium.com/max/2218/1*9wfkZb-peSlwE10WFAVd0Q.png)*AWS IoT Rule to forward message to Lambda*

The rule configuration can be quite complex as you’ll have to use SQL querying to define what data you want to receive from what MQTT messages. The [AWS IoT SQL Reference](https://docs.aws.amazon.com/iot/latest/developerguide/iot-sql-reference.html) is a helpful resource if you build more complex MQTT messages. In the case of our program the following will extract all message data.
> SELECT sensor.temperature as temperature, sensor.humidity as humidity, sensor.pressure AS pressure FROM ‘stm32/sensor’

The Lambda action executes a Lambda where the event-Object, passed onto the Lambda as an argument, simply contains the entire document we have defined here. You can also define S3, Kinesis and a lot of other AWS services to handle your MQTT messages from AWS IoT.

### Pushing the Thing’s data onto S3

If you just want to display your IoT data from the MQTT messages that AWS IoT received, you can push it onto S3. The AWS IoT Rule for that is quite simple.

![AWS IoT pushing MQTT message date to S3 with an AWS IoT Rule](https://cdn-images-1.medium.com/max/2196/1*z5wUwag_oX7A20HybxpGzQ.png)*AWS IoT pushing MQTT message date to S3 with an AWS IoT Rule*

The “*Store messages in an Amazon S3 bucket*” with a fixed value for “*Key*” will make AWS IoT store the file on S3 and overwrite it when the next message arrives. If you want to store a file for each message on S3, then change the “Key” to “*iotdata_${timestamp()}.json*” and AWS IoT will attach a timestamp to the file.

![S3 Bucket with MQTT message data from AWS IoT as JSON — and an HTML frontend](https://cdn-images-1.medium.com/max/2350/1*uvXweaXAC0Ip9W-QthGwfg.png)*S3 Bucket with MQTT message data from AWS IoT as JSON — and an HTML frontend*

### Serving the AWS IoT data on S3 through CloudFront

If you intend to present your IoT data in a web frontend, you can switch your S3 Bucket to public access using the following Bucket policy. You can then serve an HTML page that will just display the contents of the JSON file using JavaScript.

    {
        "Version": "2012-10-17",
        "Id": "IotS3DataBucketPolicy",
        "Statement": [
            {
                "Sid": "AllObjectAccess",
                "Effect": "Allow",
                "Principal": {
                    "AWS": "*"
                },
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::iotsensordata/*"
            }
        ]
    }

If you plan with millions of users viewing your IoT data or want to use a custom domain name, I suggest adding CloudFront in front of the S3 bucket to reduce the number of hits onto your bucket. As you’re dealing with data updated every minute, “*Custom Object Caching*” for CloudFront is highly recommended as otherwise your data will not update properly.

![](https://cdn-images-1.medium.com/max/2310/1*4xL6MIS5IR5pSAaNP5JnTw.png)

I won’t go too much into the details of the other AWS services like S3, CloudFront, ACM, Route 53 etc. that you need to bring up a nice IoT Frontend for your STM32L4 sensor data. With the data on S3 you are pretty much able to serve it online in whatever form you wish. If there is anything you’re missing, you can always use a Lambda, Kinesis or any other AWS service that fits your requirements.

![Custom IoT Frontend on S3 served through CloudFront](https://cdn-images-1.medium.com/max/2700/1*gGhij4BJvfpVoqZHIo7zPw.png)*Custom IoT Frontend on S3 served through CloudFront*

## Review of AWS IoT and what it can do

AWS IoT is not just an MQTT Gateway with a number of nice tools and services attached to it. It has got fleet management and numerous other awesome features *(Thing Shadows!)* that we have not explored in this quick Hands-On tutorial. Anyone who had dealt with IoT or embedded devices with sensors before knows the cumbersome work needed to distribute that data over the web for scale.

![STM32L4 with a USB battery pack in a water-proof case for outdoor use](https://cdn-images-1.medium.com/max/8320/1*E5bIj-RoJSG-BPYFrHm3cw.jpeg)*STM32L4 with a USB battery pack in a water-proof case for outdoor use*

I tried to figure out what cost I caused by using AWS IoT with S3 and CloudFront, but could not find anything noticable. Without a doubt is AWS IoT probably the lowest priced, highest quality MQTT Gateway and IoT solution you can find out there. AWS IoT can scale from a hobbyist or small educational installation up to a multi billion dollar off-shore windpark or a smart city with millions of users using the services.

AWS IoT unleashes it’s full power with the connection of other AWS services and there are almost no limits of what the AWS cloud can do. The greatest benefit, in my opinion, is that even a small organisation can use Enterprise-grade IoT solutions that scale into unbelievable heights at incredibly low cost.

**Thank you for taking the time to read my article!**

I hope you enjoy the AWS IoT and cloud services as much as I do.

Jan Kammerath
