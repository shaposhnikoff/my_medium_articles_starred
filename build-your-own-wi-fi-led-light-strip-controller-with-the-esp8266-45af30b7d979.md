
# Build Your Own Wi-Fi LED Light Strip Controller with the ESP8266

We can already easily control LED light strips wirelessly using store-bought Wi-Fi controllers, but some of those can cost an arm and a leg. Chris Ostmo (AKA appideas) has found a way of doing the same, but on a cheaper budget, and the fact that you can control two sets of LED strips is a bonus. The secret behind his low-cost Wi-Fi LED Light Strip Controller is the ESP8266 ESP-12E (NodeMCU) — a great microcontroller with wireless capabilities, and breadboard ready.

![The Wi-Fi LED Strip Controller features an ESP8266 ESP-12E microcontroller, 5V regulator, 12V power supply, and PCB boards. (📷: [Chris Ostmo](https://www.instructables.com/id/WiFi-LED-Light-Strip-Controller/))](https://cdn-images-1.medium.com/max/2048/1*-7iUsT-A3PC23Z7hnQBcFg.jpeg)*The Wi-Fi LED Strip Controller features an ESP8266 ESP-12E microcontroller, 5V regulator, 12V power supply, and PCB boards. (📷: [Chris Ostmo](https://www.instructables.com/id/WiFi-LED-Light-Strip-Controller/))*

Ostomos provides a complete BOM for his project, which beyond the microcontroller, features SuperNight 5M 5050 RGB LED strip lights, 1X 5V voltage regulator, 8X N-channel MOSFETs, 1X 12V (5A) power supply, 2X copper-clad PCBs, 10-meters of 5-strand cabling, and a host of odds-and-ends.

![Wiring the ESP8266 is pretty straightforward and can be done using a PCB, perf/strip board, or even a breadboard. (📷: [Chris Ostmo](https://www.instructables.com/id/WiFi-LED-Light-Strip-Controller/))](https://cdn-images-1.medium.com/max/2000/1*uC15HR-ren6qxLsFA_UXNQ.jpeg)*Wiring the ESP8266 is pretty straightforward and can be done using a PCB, perf/strip board, or even a breadboard. (📷: [Chris Ostmo](https://www.instructables.com/id/WiFi-LED-Light-Strip-Controller/))*

For his project, Ostmo wired the ESP8266 to a custom PCB board he created using a CNC router, however you can also use perf or strip boards if need be. Once wired, he dropped the platform into a 3D-printed housing (any non-conductive enclosure will do), and soldered the LED strips to the appropriate leads. He shares that the controller can be powered directly from the ESP8266 micro USB port over the 12V power supply if need be.

![Controlling the LED light strip is done through an easy to use iOS/Android app. (📷: [Chris Ostmo](https://www.instructables.com/id/WiFi-LED-Light-Strip-Controller/))](https://cdn-images-1.medium.com/max/2270/1*6a-TEjtuyKXSIT5FSnD6Ug.png)*Controlling the LED light strip is done through an easy to use iOS/Android app. (📷: [Chris Ostmo](https://www.instructables.com/id/WiFi-LED-Light-Strip-Controller/))*

Programming the Wi-Fi LED Light Strip Controller is done using the Arduino IDE via PC, and Ostmo conveniently uploaded the code in his project log. The controller needs to be tied into your local network to function and is managed using a simple mobile app (or web browser), which can be set up to run different light configurations.

This is an excellent project for novices and advanced makers alike and has an easy to understand walkthrough for every step. That being said, not everyone has access to a CNC machine or 3D printer, but Ostmo states that those aren’t critical to the build.

![](https://cdn-images-1.medium.com/max/4000/1*8_YOyZPmY2Lhloa6m_aPpQ.png)
