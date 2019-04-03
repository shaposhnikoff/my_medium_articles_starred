
# Over-the-air updating an ESP32

Over-the-air updating an ESP32

*Update 02/09/2017: See our follow-up blog post, “[Secure over-the-air updates for ESP32](https://blog.classycode.com/secure-over-the-air-updates-for-esp32-ec25ae00db43#.rydbamai2)”.*

You might have heard of Espressif’s brand new chip, the [ESP32](https://espressif.com/en/products/hardware/esp32/overview). We were fascinated by the fact that this chip combines Wi-Fi, Bluetooth, two fast CPU cores and a large number of peripherals on a single integrated circuit and decided to order a couple of [SparkFun ESP32 Thing modules](https://www.sparkfun.com/products/13907).

It was a lucky coincidence that we received our illuminated company logo at around the same time as the SparkFun modules. Alex had the idea to use the ESP32 boards to let the illumination pulsate during Jenkins builds and reflect the build results in the brightness of the logo. As I was finalising a customer project and had some spare time, I installed the Espressif IDE and implemented a small ESP32 application to control the logo brightness via TCP commands. Alex wrote a small python script to forward the Jenkins state to the application. Within a couple of hours, the first [working prototype](https://twitter.com/toeggeler/status/802132849075884032) was ready.

(My initial idea was to use a power mosfet, driven by a PWM signal from the ESP32, to control the brightness of the LED’s. However, the logo illumination uses a special transformer which acts as a current source. The transformer provides a two-terminal input to connect a resistor to control the output current, so instead of the mosfet, I connected a digital potentiometer [AD8400](http://www.analog.com/en/products/digital-to-analog-converters/precision-dac/digital-potentiometers/ad8400.html) from Analog Devices to the input terminals. This chip can be controlled by a simple serial data input which I connected to a few GPIO pins of the ESP32.)

![Connections between the AD8400 and the SparkFun module](https://cdn-images-1.medium.com/max/8064/1*sQn9eMfYfSv-Scg1xeHR4w.jpeg)*Connections between the AD8400 and the SparkFun module*

While still under heavy construction, the Espressif tools and libraries are really cool to use, so I produced a series of updates for the logo control application. At a certain point, the process of loading the new application into the SparkFun module became annoying because it means that you have to disconnect the module from the logo, connect it to the computer, flash the firmware and re-connect the module to the logo. So, we decided to add an over-the-air update mechanism.

I was happily surprised to see that Espressif had already done some preparation work: The 2nd-stage boot loader is able to detect and boot from OTA partitions, and the app_update component allows to load custom images into a flash memory attached to the ESP32 chip. But then, I quickly realized that I was deep in uncharted waters. Writing to the flash worked (after manually erasing the flash pages, because the library implementation took so long that it caused the task watchdog to trigger), but the bootloader [refused](http://www.esp32.com/viewtopic.php?f=14&t=610) [to boot from the correct partition](http://www.esp32.com/viewtopic.php?f=14&t=615). Espressif had put their bootloader and library code on [GitHub](https://github.com/espressif/esp-idf), so I could easily fork it and implement a few work-arounds. (The day after, I noticed that I could just have waited — Espressif had already committed the fixes for these issues :-)

You can find a first version of the OTA code [in our repository](https://github.com/classycodeoss/esp32-ota). This is a very simple implementation without much error handling or security, and it’s not very fast yet, either. I got about 5 to 10 kBytes per second with the current implementation. Improving the performance is on the TO-DO list.

*Update 09.12.2016: In the most recent version, we keep the TCP connection open and transfer the update in binary format instead of hex strings. As a result, the transfer is now about 5 to 10 times faster. I’ve updated this blog post to reflect these improvements.*

The current implementation places the first OTA update into partition ota_0, the second into ota_1, the third in ota_0 again and so on:

![](https://cdn-images-1.medium.com/max/2000/1*s320_ezWr_0EUvkg5Y9Edg.png)

The OTA update code automatically switches to the next partition after a successful update.

Follow these steps to perform an OTA update:

* Use our fork of the esp-idf which contains a fix in the bootloader code. [https://github.com/classycodeoss/esp-idf/tree/master](https://github.com/classycodeoss/esp-idf/tree/master)
Don’t forget to specify the “recursive” flag to get the python tools:

    git clone --recursive [https://github.com/classycodeoss/esp-idf.git](https://github.com/classycodeoss/esp-idf.git)

* Erase the flash to make sure the ota_0 and ota_1 partitions are empty. We use the predefined partition scheme with two OTA partitions (Use the correct path to esptool.py and the correct usbserial device for your environment.):

    python esp-idf/components/esptool_py/esptool/esptool.py --chip esp32 --port /dev/tty.usbserial-DN027YY7 --baud 921600 erase_flash

* Put the correct SSID and password for your access point in the file “main/main.c”. The target needs to be reachable via WIFI for the update to work.

* Update the usbserial device (CONFIG_ESPTOOLPY_PORT) in the sdkconfig file (preferrably with “make menuconfig”).

* “make” and “make flash” the first version of the application. This will store the first version as the “factory version” in the factory partition.

* Reboot the device and make sure the application is working correctly. Check the log output from the ESP32 (e.g. with minicom) and write down the IP address of the target. Alternatively, your DHCP server should also provide that information.
On the sparkfun ESP32 Thing board, the application displays its state via the blue LED on the board. Repeated single, short flashes mean that the application has started. Two flashes mean that the application is connected to the access point. Four flashes indicate an ongoing OTA update. Three flashes indicate an ongoing OTA update with lost network connectivity.

* Modify the application code. For example, you can change the Factory-prepared in “#define SOFTWARE_VERSION “Factory-prepared”” to something else, for example, First-Update.

* Run “make” to compile the new version of the application, **but don’t run “make flash” **as this would replace the factory version.

* Go to the tools directory and execute the “update_firmware.py” script. The script takes two parameters, the IP address of the target and a path to the bin file of the application. On my machine, the command looks as follows:

    $ python update_firmware.py 192.168.33.52 ../build/esp32-ota-demo.bin

* You can watch the update process on both sides:

![](https://cdn-images-1.medium.com/max/2938/1*LMr0jwAQCsglA4uiOEgNiQ.png)

The update script will trigger a re-boot on the ESP32 after completing.

The following points are still on the TO-DO list:

* Switch back to Espressif’s version of the esp-idf on Github. *Currently, [there’s one remaining issue in the bootloader code](https://github.com/classycodeoss/esp-idf/commit/787b88d355de2861f04004713d1636c8c99d4e8c) that needs to be fixed before we can switch back.*

* Signature verification of the uploaded application. This is actually covered by the secure boot mechanism which won’t boot invalid partitions, but it would be nice to let the OTA code verify the signature after the upload and only activate the new version of the application if the signature is correct. May be this is as simple as calling the existing esp_secure_boot_verify_signature function.

* Better error handling in the upload script and also in the OTA code.

I guess Espressif will at some point provide an “official” OTA implementation, but until then, I hope you find this useful :-)
