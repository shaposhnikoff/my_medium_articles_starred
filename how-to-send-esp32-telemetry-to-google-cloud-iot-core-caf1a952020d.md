
# How to send ESP32 telemetry to Google Cloud IoT Core

*** Updated 24/02/2019 to use latest MQTT Google example Esp32-lwmqtt rather than outdated https example ; with mqtt, initial connection is heavier and then each message publishing is smaller ; See section ‘Create & Configure Sketch’ to load the new sketch ***

I’m trying here to have a comprehensive sample demonstrating all the pieces to send telemetry from esp32 device to cloud. Hope this will help beginners ;)

I will demo how an ESP32 DevKit device communicates with Google Cloud IoT Core in both ways, meaning sending telemetry to Google Cloud and also reading messages from the Cloud (through the device configuration message).

The device will:

1. Send TMP36 temperature sensor telemetry to a Cloud Pub/Sub topic

1. Read Cloud IoT Core device messages (configuration text or command) and switch on/off the ESP32 internal led.

![](https://cdn-images-1.medium.com/max/2000/1*jujaAPD1KV6NcfcM8-NI9A.png)

### Disclaimer

We are using here Google Cloud IoT Core JWT library version 1.1.1 for Arduino IDE, implemented by IoT Googlers but not an officially supported Google product, it does not have a SLA/SLO, and should not be used in production.

## Steps

We will cover these steps:

* Create Google Cloud platform project

* Create Google Cloud console IoT Core structure (registry / device / pubsub)

* Prepare ESP32 with temperature sensor

* Prepare Arduino IDE sketch

* Run

* Check published messages

## Create Google cloud platform project

* Go to [https://console.cloud.google.com/](https://console.cloud.google.com/)

* [Create a project](https://cloud.google.com/resource-manager/docs/creating-managing-projects) and enable Billing

![](https://cdn-images-1.medium.com/max/2000/0*eZJ_qrrOYUNai-iX)

* Tip: Pin IoT Core, Pub/Sub

![](https://cdn-images-1.medium.com/max/2000/0*oRQjp4HklibTQwea)

## Create Google IoT Core Structure

We need to create:

1. A Pub/Sub topic

1. A Pub/sub subscription

1. An IoT Core Registry using the Pub/Sub topic

1. An IoT Core Device

Go!

* Open Google Cloud Shell

* Select your project and go to Google Cloud Shell

![](https://cdn-images-1.medium.com/max/2000/0*gf8Dzw8SpxhHlYjP)

* Create Google Cloud Pub/Sub
> ~$ gcloud pubsub topics create iot-sample-pubsub
> Created subscription [projects/black-burner-xxxxxx/subscriptions/iot-sample-sub].

* Create Pub/Sub subscription
> ~$ gcloud pubsub subscriptions create iot-sample-sub — topic=iot-sample-pubsub

* Create IoT Registry
> ~$ gcloud iot registries create iot-sample-registry — region=europe-west1 — event-notification-config=topic=iot-sample-pubsub
> Created registry [iot-sample-registry].

Note: this command will propose to enable IoT Core API if not already enabled on the project

* Generate your device certificates

Generate an Eliptic Curve (EC) ES256 private / public key pair:
> ~$ openssl req -x509 -newkey rsa:2048 -keyout iot-sample-device-rsa_private.pem -nodes -out rsa_cert.pem -subj “/CN=unused”
> ~$ openssl ecparam -genkey -name prime256v1 -noout -out iot-sample-device-ec_private.pem
> ~$ openssl ec -in iot-sample-device-ec_private.pem -pubout -out iot-sample-device-ec_public.pem

* Get and store the files in a secure location of your choice

* Create Device
> ~$ gcloud iot devices create iot-sample-device — region=europe-west1 — registry=iot-sample-registry — public-key=path=./iot-sample-device-ec_public.pem,type=es256

Go and see your Google Cloud IoT Core device!

* Got to IoT Core

![](https://cdn-images-1.medium.com/max/2000/0*VzVXm2XPbrmdq-gB)

* Click on the registry

![](https://cdn-images-1.medium.com/max/2000/0*fKglFB_LQNFXzyxM)

* Here is your device!

![](https://cdn-images-1.medium.com/max/2000/0*HZrv0OD2osVvEHB1)

If this is your first one, congratulations!!!

## Setup ESP32

We will use here a simple TMP36 Temperature Sensor for this demo. It is not production ready (with a -2° to +2° error but enough for a demo ;-) )

Simply wire as follows:

- 3.3V PIN to TMP36 left leg
- GND PIN to TMP36 right leg
- PIN 33 to TMP36 middle leg

(Note: on ESP 32 when wifi is used, low numbers pins cannot be used, do not try to use pin 2 for example)

![](https://cdn-images-1.medium.com/max/2000/0*WBXiVxHY4wLWnLmq)

## Setup Arduino IDE

Open Arduino and select the Sketch > Include Library > Library Manager menu item.

Install following libraries:

- “Google Cloud IoT Core JWT” (version 1.1.1)
- MQTT (version 2.4.3): Installed it with [zip file found here](https://www.arduinolibraries.info/libraries/mqtt).
- ESP32 WifiClientSecure

*Disclaimer: Please tell me if you discover that I forgot to mention a lib that was already installed on my IDE before.*

![](https://cdn-images-1.medium.com/max/2000/0*iKLRCGYgOqVCqRHc)

## Create & Configure Sketch

1. First clone Google google-cloud-iot-arduino project ([link](https://github.com/GoogleCloudPlatform/google-cloud-iot-arduino)) within your Arduino sketches directory
*- Note 1: on Mac OS, usually /Users/<your user>/Documents/Arduino
- Note 2: I used latest version from 21/02/2019
*This should create for you directory google-cloud-iot-arduino, containing examples & pio & src sub directories.

1. Download my esp32-lwmqtt rewriting from here → [link](https://drive.google.com/open?id=1BrWuw3k4rwDWn99lEhXT4vaPIhXLtC7p)
Unzip the directory and move it under google-cloud-iot-arduino/examples sub directory,
- *Note 1: do not replace the cloned Google Esp32-lwmqtt sub directory which is a lighter example sending wifi level as a telemetry, you might want to play with it later ;) )
- Note 2: Rename the subdirectory as you like but if you do so, rename also the .ino file with same name, this is required by Arduino IDE
- Note 3: this is almost the same content, the differences are a) methods *getTemperature & getTemperatureJSON replace getDefaultSensor method, b) messageReceivedUpdateLed replaces Google messageReceived method, c) LED_BUILTIN=2 instead of 13 because it was the case with my ESP32, you might need to check yours.

1. Open the sketch within Arduino IDE

1. Go to ciotc_config.h file

![](https://cdn-images-1.medium.com/max/2000/0*3xaVlSFvNT2a3R7b)

Fill the following variables for your context:

* ssid : your wifi SSID

* password : your wifi password

* project_id : Google cloud console project ID (see [here](https://cloud.google.com/resource-manager/docs/creating-managing-projects) if you need help locating it)

* location : your Google cloud console project region, for example: europe-west1

* registry_id : “iot-sample-registry”

* device_id : “iot-sample-device”

* for private_key_str:
Get the iot-sample-device-ec_private.pem file generated when creating Google Cloud IoT Core structure and run following command to get private key. The key will be found within priv part.
> $ openssl ec -in iot-sample-device-ec_private.pem -noout -text
read EC key
Private-Key: (256 bit)
priv:
**<priv key>
**pub:
<pub key>
ASN1 OID: prime256v1
NIST CURVE: P-256
> Example:The key is
00:d2:72:29:e5:44:10:e2:13:bc:e8:11:99:25:e4:
6f:c0:6d:bc:06:f7:fc:12:b1:1c:0f:5c:d0:9c:f8:
a0:45:81
> For this result:
$ openssl ec -in iot-sample-device-ec_private.pem -noout -text
read EC key
Private-Key: (256 bit)
priv:
**00:d2:72:29:e5:44:10:e2:13:bc:e8:11:99:25:e4:
6f:c0:6d:bc:06:f7:fc:12:b1:1c:0f:5c:d0:9c:f8:
a0:45:81
**pub:
02:ad:5e:11:15:92:46:a2:de:f4:89:60:71:92:15:
e2:46:7d:31:61:4e:95:e4:a4:4a:b4:6e:62:11:55:
5f:d3:07:3a:c8:1e:28:7f:86:b4:5f:d0:d4:0e:7d:
5b:05:58:8d:2e:0f:90:6e:69:cd:f4:f5:7b:8c:04:
34:6e:6c:d8:65
ASN1 OID: prime256v1
NIST CURVE: P-256

* root_cert: To get the root_cert

* 1) Open a text editor and create a new file

* 2) Within a terminal, run
openssl s_client -showcerts -connect mqtt.googleapis.com:8883

* 3) From the results, copy within the file editor the certificate (all lines between and including — -BEGIN CERTIFICATE — and — END CERTIFICATE — )

* 4) add “ char at the beginning of each line

* 5) add \n“ at the end of each line

* Example: you likely will have more lines, truncated it here :)

* “ — — -BEGIN CERTIFICATE — — -\n”
 “MIIEXDCCA0SgAwIBAgINAeOpMBz8cgY4P5pTHTANBgkqhkiG9w0BAQsFADBMMSAw\n”
 “HgYDVQQLExdHbG9iYWxTaWduIFJvb3QgQ0EgLSBSMjETMBEGA1UEChMKR2xvYmFs\n”
 “7a8IVk6wuy6pm+T7HT4LY8ibS5FEZlfAFLSW8NwsVz9SBK2Vqn1N0PIMn5xA6NZV\n”
 “c7o835DLAFshEWfC7TIe3g==\n”
 “ — — -END CERTIFICATE — — -\n”;

## Run

* Plug the ESP32 device on your computer

* Upload the sketch to device with Arduino IDE

* If everything is ok, you should see connection and events in serial monitor
> […]
Waiting on time sync...
checking wifi...
connecting...Refreshing JWT
connected!
[…]

* Note: not so warm, end of winter here ;)

* When the device sends temperature telemetry
> […]
Analog Temperature is: 688
Temperature is: 17.25
Analog Temperature is: 704
Temperature is: 18.82
[…]

Switch on/off built-in ESP32 led by sending a command or updating the device configuration with Google Cloud Console

* Open the device page

* Click on Send Command or Update Config

![](https://cdn-images-1.medium.com/max/2000/0*eP9cjHmYV4hz3yth)

* Enter text

* Containing ledon word to switch on

* Or another message without ledon to switch off

![](https://cdn-images-1.medium.com/max/2000/0*qKNx78m1Fot7zTxS)

* Check & acknowledge the 10 latest messages reaching the topic
> ~$ gcloud pubsub subscriptions pull iot-sample-sub --auto-ack --limit=10

## Useful links

[https://cloud.google.com/iot/docs/reference/](https://cloud.google.com/iot/docs/reference/)

[https://console.cloud.google.com/iot](https://console.cloud.google.com/iot)

[https://www.arduino.cc/en/Main/Software](https://www.arduino.cc/en/Main/Software)

[https://medium.com/@gguuss/accessing-cloud-iot-core-from-arduino-838c2138cf2b](https://medium.com/@gguuss/accessing-cloud-iot-core-from-arduino-838c2138cf2b)

[How-To guides ](https://cloud.google.com/iot/docs/quickstart)of Google Cloud IoT Core

[My videos](https://www.youtube.com/channel/UCw8Askz4Iw0gnyTHj1vi2Ow/videos) & [corresponding posts](https://iotlabexperience.blogspot.com/) on Arduino basics.
