Unknown markup type 10 { type: [33m10[39m, start: [33m45[39m, end: [33m50[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m42[39m, end: [33m65[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m40[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m73[39m, end: [33m83[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m98[39m, end: [33m114[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m183[39m, end: [33m193[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m177[39m, end: [33m185[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m216[39m, end: [33m226[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m12[39m, end: [33m21[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m55[39m, end: [33m68[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m73[39m, end: [33m81[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m107[39m, end: [33m120[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m38[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m37[39m, end: [33m47[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m196[39m, end: [33m206[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m61[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m123[39m, end: [33m127[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m281[39m, end: [33m290[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m32[39m, end: [33m45[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m64[39m, end: [33m69[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m27[39m, end: [33m38[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m59[39m, end: [33m79[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m132[39m, end: [33m142[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m29[39m, end: [33m50[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m53[39m, end: [33m72[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m281[39m, end: [33m289[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m340[39m, end: [33m348[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m53[39m, end: [33m63[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m5[39m, end: [33m6[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m18[39m, end: [33m19[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m24[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m101[39m, end: [33m104[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m9[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m5[39m, end: [33m6[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m66[39m, end: [33m78[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m37[39m, end: [33m43[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m30[39m, end: [33m35[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m27[39m, end: [33m32[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m53[39m, end: [33m67[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m120[39m, end: [33m130[39m }

# Espressif ESP32 Tutorial — Programming (ESP-IDF)

The native software development framework for the ESP-32 is called the Espressif IoT Development Framework (ESP-IDF). If you want maximum functionality and compatibility this is what you should use. It is a bit more “hard core” than some of the other programming options but it does help you better understand what is happening under the hood.
 
 The ESP-IDF functionality includes menu based configuration, compiling and firmware download to ESP32 boards.

![ESP-IDF Tool Chain](https://cdn-images-1.medium.com/max/2000/0*f8p0XQ5TiQMxKIWm.png)*ESP-IDF Tool Chain*

To develop applications for ESP32 you will need:

* A PC loaded with either Windows, Linux or the Mac operating system

* Toolchain to build the Application for ESP32

* ESP-IDF that essentially contains the API for ESP32 and scripts to operate the Toolchain

* A text editor to write programs (Projects) in C, e.g. Eclipse

* The ESP32 board itself and a USB cable to connect it to the PC

The quickest way to start development with the ESP32 is by installing a prebuilt toolchain. Head over to the [Espressif site](https://esp-idf.readthedocs.io/en/v3.1-beta1/get-started/index.html#introduction) and follow the provided instructions for your OS. I will outline the process for Mac.

### Install Prerequisites

The first thing you need to do is install pip.

    sudo pip install pyserial

Then install pyserial:

    sudo pip install pyserial

### Tool Chain Setup

The ESP32 toolchain for macOS is available for download from the [Espressif website](https://dl.espressif.com/dl/xtensa-esp32-elf-osx-1.22.0-80-g6c4433a-5.2.0.tar.gz).

Download this file, then extract it into the ~/esp directory:

    mkdir -p ~/esp 
    cd ~/esp 
    tar -xzf ~/Downloads/xtensa-esp32-elf-osx-1.22.0-80-g6c4433a-5.2.0.tar.gz

The tool chain will be extracted into the ~/esp/xtensa-esp32-elf/directory.

![Editing .profile in Nano](https://cdn-images-1.medium.com/max/2000/0*8KRRLtz1tVgDYLjK.png)*Editing .profile in Nano*

To use it, you will need to update your PATH environment variable in the ~/.profile file. To make xtensa-esp32-elf available for all terminal sessions, add the following line to your ~/.profile file:

    export PATH=$PATH:$HOME/esp/xtensa-esp32-elf/bin

You can edit the .profile file using nano (e.g. nano ~/.profile). While you are here add the IDF_PATH environment variable to make the ESP-IDF available as well. To include the IDF_PATH add the following line to the ~/.profile file:

    export IDF_PATH=~/esp/esp-idf

Quitting Terminal will make this change effective.

**Note:**

If you have /bin/bash set as login shell, and both the .bash_profile and .profile files exist, then update .bash_profile instead.

Run the following command to check if IDF_PATH is set:

    printenv IDF_PATH

The path previously entered into the ~/.profile file (or set manually) should be printed out.

Alternatively, you may create an alias for the above command. This way you can get access the tool chain only when you need it. 
 
 To do this, add this line (instead of the previous one) to your ~/.profile file:

    alias get_esp32="export PATH=$PATH:$HOME/esp/xtensa-esp32-elf/bin"

Then when you need the tool chain you can just type get_esp32 on the command line and the tool chain will be added to your PATH. You can do something similar for IDF_PATH.

### Get the ESP-IDF

Besides the toolchain (that contains programs to compile and build the application), you also need ESP32 specific API / libraries. They are provided by Espressif in [ESP-IDF repository](https://github.com/espressif/esp-idf). To get it, open terminal, navigate to the directory you want to put ESP-IDF, and clone it using git clonecommand:

    cd ~/esp 
    git clone --recursive [https://github.com/espressif/esp-idf.git](https://github.com/espressif/esp-idf.git)

ESP-IDF will be downloaded into ~/esp/esp-idf.

### Establish a Serial Connection

Connect the ESP32 board to the PC using the USB cable. Once again, the following are for the Mac, if you have another OS then follow the [instructions on the Espressif web site.](https://esp-idf.readthedocs.io/en/v3.1-beta1/get-started/establish-serial-connection.html)

To check the device name for the serial port of your ESP32 board, run this command:

    ls /dev/cu.*

On my MacBook Pro (early 2015) running High Sierra, the port on the Duinotech ESP32 Dev Board was not recognized. If this happens, you may need to install the drivers for the USB-serial converter for this board. It uses a CP2102 IC, and the drivers are found on the Silicon Labs CP2102 website:
 [https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers](https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers).
 
 After you download the correct driver version for your OS. You will need to install it. On the Mac this involves, unzipping the archive, mounting the DMG disk image and then running the Silicon Labs VCP Driver.pkg. You will need to give permission to run this driver in the Security and Privacy preference (this will pop up). You should now be able to see the port.
 
 On my Mac, the port name was: /dev/cu.SLAB_USBtoUART

### Hello World Project

We will use the [getstarted/hello_world](https://github.com/espressif/esp-idf/tree/v3.1-beta1/examples/get-started/hello_world) project from the [examples](https://github.com/espressif/esp-idf/tree/v3.1-beta1/examples) directory in the IDF.

First, open up Terminal and copy [get-started/hello_world](https://github.com/espressif/esp-idf/tree/v3.1-beta1/examples/get-started/hello_world) to the ~/esp directory:

    cd ~/esp 
    cp -r $IDF_PATH/examples/get-started/hello_world .

You can find a range of other example projects in the [examples](https://github.com/espressif/esp-idf/tree/v3.1-beta1/examples) directory in ESP-IDF. There are examples for:

* Ethernet

* Mesh

* Protocols

* System

* Bluetooth

* Getting Started

* Peripherals

* Storage

* WiFi

These example project directories can be copied in the same way as shown above.

**Important:**

The esp-idf build system does not support spaces in paths to esp-idf or to the projects.

In Terminal, change to the hello_world directory by typing cd ~/esp/hello_world. Then start the project configuration utility using menuconfig:

    cd ~/esp/hello_world 
    make menuconfig

All going well, the following menu will be displayed:

![ESP-IDF Menu Config Program](https://cdn-images-1.medium.com/max/2000/0*izFC9vucTyaIEZM8.png)*ESP-IDF Menu Config Program*

In the menu, navigate to:
 
 Serial flasher config > Default serial port 
 
 to configure the serial port that your ESP32 is connected to. 
 
 Enter the name that you had above (e.g. /dev/cu.SLAB_USBtoUART). Confirm selection by pressing enter, save the configuration by selecting < Save > and then exit the config application by selecting < Exit >.

Here are couple of tips on the navigation and use of menuconfig:

* Use up & down arrow keys to navigate the menu.

* Use Enter key to go into a submenu, Escape key to go out or to exit.

* Type ? to see a help screen. Enter key exits the help screen.

* Use Space key, or Y and N keys to enable (Yes) and disable (No) configuration items with checkboxes “[*]”

* Pressing ? while highlighting a configuration item displays help about that item.

* Type / to search the configuration items.

**Build and Flash**

Now you can build and flash the application. Run:

    make flash

This will compile the application and all the ESP-IDF components, generate the bootloader, partition table, and application binaries, and flash these binaries to your ESP32 board.

    esptool.py v2.0-beta2 
    Flashing binaries to serial port /dev/ttyUSB0 (app at offset 0x10000)... 
    esptool.py v2.0-beta2 
    Connecting........___ 
    Uploading stub... 
    Running stub... 
    Stub running... 
    Changing baud rate to 921600 
    Changed. 
    Attaching SPI flash... 
    Configuring flash size... 
    Auto-detected Flash size: 4MB 
    Flash params set to 0x0220 
    Compressed 11616 bytes to 6695... 
    Wrote 11616 bytes (6695 compressed) at 0x00001000 in 0.1 seconds (effective 920.5 kbit/s)... 
    Hash of data verified. 
    Compressed 408096 bytes to 171625... 
    Wrote 408096 bytes (171625 compressed) at 0x00010000 in 3.9 seconds (effective 847.3 kbit/s)... 
    Hash of data verified. 
    Compressed 3072 bytes to 82... 
    Wrote 3072 bytes (82 compressed) at 0x00008000 in 0.0 seconds (effective 8297.4 kbit/s)... 
    Hash of data verified. 

    Leaving... 
    Hard resetting...

If there are no issues, at the end of build process, you should see messages describing progress of loading process. Finally, the end module will be reset and the “hello_world” application will start.

**Monitor**

To check if the “hello_world” application is indeed running, type make monitor. This command will launch the [IDF Monitor](https://esp-idf.readthedocs.io/en/v3.1-beta1/get-started/idf-monitor.html) application:

    $ make monitor 
    MONITOR 
    --- idf_monitor on /dev/ttyUSB0 115200 --- 
    --- Quit: Ctrl+] | Menu: Ctrl+T | Help: Ctrl+T followed by Ctrl+H --- ets Jun 8 2016 00:22:57 rst:0x1 (POWERON_RESET),boot:0x13 (SPI_FAST_FLASH_BOOT) 
    ets Jun 8 2016 00:22:57 ...

    rst:0x1 (POWERON_RESET),boot:0x13 (SPI_FAST_FLASH_BOOT)
    ets Jun  8 2016 00:22:57
    ...

Several lines on, after start up and the diagnostic log, you should see “Hello world!” printed out by the application.

![“Hello World” Displayed in Terminal.](https://cdn-images-1.medium.com/max/2000/0*zDDe2xQU3mIn01ZX.png)*“Hello World” Displayed in Terminal.*

To exit the monitor use the shortcut Ctrl+].

### The Blink Project

For direct comparison with the Arduino IDE in the previous post, we will also give the Blink project a crack. The process is very similar to hello_world.

Copy [get-started/blink](https://github.com/espressif/esp-idf/tree/v3.1-beta1/examples/get-started/blink) to the ~/esp directory:

    cd ~/esp 
    cp -r $IDF_PATH/examples/get-started/blink .

In Terminal, change to the blink directory by typing cd ~/esp/blink. Then start the project configuration utility using menuconfig:

    cd ~/esp/blink 
    make menuconfig

As for the hello_world example above, set the default serial port to the correct value, then save and exit.

Before building and flashing the ESP32, we just need to check which LED is going to blink. If you open up blink.c in your favourite text editor, you will see the line:

    #define BLINK_GPIO CONFIG_BLINK_GPIO

So you can either run “make menuconfig” to select the GPIO to blink, or just change the number here. Since I already had the file open, I changed the number and saved the file.

It is interesting to compare the code required to blink a LED in native ESP-32 C versus Arduino C. The ESP-32 version is shown below.

    #define BLINK_GPIO 2 

    void blink_task(void *pvParameter) 
    { 
        /* Configure the IOMUX register for pad BLINK_GPIO (some pads    
           are muxed to GPIO on reset already, but some default to other 
           functions and need to be switched to GPIO. Consult the 
           Technical Reference for a list of pads and their default 
           functions.) 
        */ 
        gpio_pad_select_gpio(BLINK_GPIO); 
        /* Set the GPIO as a push/pull output */ 
        gpio_set_direction(BLINK_GPIO, GPIO_MODE_OUTPUT); 
        while(1) { 
            /* Blink off (output low) */ 
            gpio_set_level(BLINK_GPIO, 0); 
            vTaskDelay(1000 / portTICK_PERIOD_MS); 
            /* Blink on (output high) */ 
            gpio_set_level(BLINK_GPIO, 1); 
            vTaskDelay(1000 / portTICK_PERIOD_MS); 
        } 
    } 

    void app_main() 
    { 
        xTaskCreate(&blink_task, "blink_task", configMINIMAL_STACK_SIZE, NULL, 5, NULL); 
    }

While not being a lot harder than doing it with Arduino C++, it is certainly less readable. The standard Arduino library hides a lot of the boiler plate code. I’ve reproduced the standard Arduino blink code below for comparison.

    void setup() { 
        pinMode(LED_BUILTIN, OUTPUT); 
    } 

    void loop() { 
        digitalWrite(LED_BUILTIN, HIGH); 
        // turn the LED on (HIGH is the voltage level) 
        delay(1000); 
        // wait for a second 
        digitalWrite(LED_BUILTIN, LOW); 
        // turn the LED off by making the voltage LOW 
        delay(1000); 
        // wait for a second 
    }

Now you can build and flash the blink application. Run:

    make flash

After a lot of scrolling on Terminal the blink binary will be flashed to your ESP-32 and the Dev Board reset. assuming you have a LED connected to pin 2, it should now be blinking.

![External LED connected to GPIO 2 of the ESP-32.](https://cdn-images-1.medium.com/max/2000/0*dviK5Xmi6yhtD5qy.jpg)*External LED connected to GPIO 2 of the ESP-32.*

### Conclusion

So the obvious conclusion from all this is that if you want to do simple applications and you are already familiar with the Arduino IDE and language, then the Arduino IDE plus the ESP-32 extension is probably your best route.
 
 If you want maximum functionality and compatibility, and are not afraid of a command line interface (CLI), then I would use the ESP-IDF.
 
 Next up we look at trying to get the best of both worlds. Combining native compatibility with a GUI which includes features like code completion. This option still requires the ESP-IDF behind the scenes so installing this will get you ahead of the curve in the next tutorial.

*Originally published at [reefwingrobotics.blogspot.com](https://reefwingrobotics.blogspot.com/2018/07/espressif-esp32-tutorial-programming_22.html) on July 22, 2018.*
