Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m101[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m84[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m16[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m101[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m167[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m176[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m166[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m78[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m185[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m134[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m31[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m17[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m21[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m21[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m69[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m36[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m38[39m }

# Raspberry Pi 3 and Arduino communication via bluetooth HC-05

Well, I didn’t find any well documented article on google for connecting Arduino and raspberry pi over bluetooth HC-05. I found some, but outdated.

So, trying to present this article on how to make wireless communication between raspberry pi and Arduino via HC-05 bluetooth module.

You can also go through this [link](https://github.com/cloud-github/raspberry-pi-arduino-bluetooth-wireless-communication)
[**cloud-github/raspberry-pi-arduino-bluetooth-wireless-communication**
*Bluetooth communication between raspberry pi 3 and arduino using HC-05 bluetooth module …*github.com](https://github.com/cloud-github/raspberry-pi-arduino-bluetooth-wireless-communication)

## Components Needed

    * Raspberry pi 3 (it has inbuilt bluetooth)
    * Arduino UNO
    * HC-05 bluetooth module (for arduino side)

## Let’s start configuring arduino uno

## ARDUINO

Please follow this link to setup arduino with HC-05 bluetooth module

    [https://www.instructables.com/id/Controlling-Arduino-UNO-Via-Smart-Phone-Using-HC-0/](https://www.instructables.com/id/Controlling-Arduino-UNO-Via-Smart-Phone-Using-HC-0/)

Make sure you select PIN 13 for LED blink.

Upload this code to your Arduino

    int ledPin = 13;

    void setup() {
      Serial.begin( 9600 );    //  baud rate 9600 for the serial Bluetooth communication
    }

    void loop() {
      // listen for the data from raspberry pi
      if ( Serial.available() > 0 ) {
        // read a numbers from serial port
        int inputVal = Serial.parseInt();

        if (inputVal > 0) {
            Serial.print("Your input is: ");
            Serial.println(String(inputVal));
            // Here blink the LED
            blinkLED(inputVal);
        }
      }
    }

    void blinkLED(int inputVal) {
      for (int i=0; i< inputVal; i++) {
        digitalWrite(ledPin, HIGH);
        delay(500);
        digitalWrite(ledPin, LOW);
        delay(500);
      }
    }

That’s all for Arduino part.

Let’s move to raspberry pi,

## RASPBERRY PI :

You can setup bluetooth in raspberry pi via two methods.

## METHOD 1:

Installing blueman for configuring bluetooth via graphical user interface.

    sudo apt-get install pi-bluetooth
    sudo apt-get install bluetooth bluez blueman

Restart you pi (essential)

Now, you should be able to see menu -> preferences -> bluetooth manager

Open bluetooth manager

click bluetooth icon (actually you will see two icons, click blue one)

click setup new device

you will see “Welcome to bluetooth device setup assistant”

process Next

Search for HC-05

Proceed further.

if every thing goes fine, you will see such screen

![](https://cdn-images-1.medium.com/max/2732/0*SPB0XOC7f6TjG3d9.png)

![](https://cdn-images-1.medium.com/max/2732/0*eH3UMVSSDK18rpz1.png)

![](https://cdn-images-1.medium.com/max/2732/0*PRxO77aLsDf1G4Le.png)

Also go to Adapter menu -> Preferences , then set visibility setting — Always visible

### Python code:

    import serial
    import time

    port = serial.Serial("/dev/rfcomm0", baudrate=9600)
     
    # reading and writing data from and to arduino serially.                                      
    # rfcomm0 -> this could be different

    while True:
    	print "DIGITAL LOGIC -- > SENDING..."
    	port.write(str(3))
    	rcv = port.readline()
    	if rcv:
    	   print(rcv)
    	time.sleep( 3 )

## METHOD 2:

Run Terminal:

    sudo systemctl status bluetooth

    sudo bluetoothctl

    [bluetooth]# agent on

    [bluetooth]# scan on

    [bluetooth]# scan off

    [bluetooth]# pair 98:D3:32:11:06:B1  # change with your HC-05 address

    [bluetooth]# trust 98:D3:32:11:06:B1

    [bluetooth]# connect 98:D3:32:11:06:B1

if everything goes fine, you should see this screen

![](https://cdn-images-1.medium.com/max/2732/0*XIC6Pa9ttqI3Sx0w.png)

Finally, sending logic to Arduino via bluetooth from serial port /dev/rfcomm0

![](https://cdn-images-1.medium.com/max/2732/0*GZ7KV9aqkI915cDG.png)
