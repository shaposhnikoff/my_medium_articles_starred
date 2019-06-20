
# How to Program an ESP32 in Arduino while using ESP-IDF functions

I’ve just started using an ESP32 chip and it’s awesome. There are two paths to program it, though.

![An ESP32-based Chip Computer](https://cdn-images-1.medium.com/max/2000/1*I0oTIKabfKcRK2Q5H3IxVg.jpeg)*An ESP32-based Chip Computer*

### [Arduino](https://www.arduino.cc/)

Arduino is simple and well supported with lots of user-donated reasonable-quality libraries. The Arduino IDE is just awful but you can use Visual Studio Code just fine and it’s really good. Online help is excellent. Functionality is limited but excellent if you don’t need exotic ESP32 control.

### [ESP-IDF](https://esp-idf.readthedocs.io/en/latest/index.html)

ESP-IDF comes straight from Espressif and lets you do great stuff with the chip but it’s not so simple. It has fewer user-donated top-level methods, it’s C and not C++ (although compatible), and it uses typical old-arcane naming.

## How to Program

Espressif now supplies a very complete Arduino library so you can use Arduino exclusively for standard applications.

Here’s an example of two code fragments. Arduino and ESP-IDF to initialize and enable the serial port:

### Arduino

    Serial.begin(115200);

### ESP-IDF

The below code is from an ESP-IDF example.

    /* Configure parameters of an UART driver,
     * communication pins and install the driver */
     uart_config_t uart_config = {
     .baud_rate = 115200,
     .data_bits = UART_DATA_8_BITS,
     .parity = UART_PARITY_DISABLE,
     .stop_bits = UART_STOP_BITS_1,
     .flow_ctrl = UART_HW_FLOWCTRL_DISABLE
     };
     uart_param_config(EX_UART_NUM, &uart_config);
     //Set UART pins (using UART0 default pins ie no changes.)
     uart_set_pin(EX_UART_NUM, UART_PIN_NO_CHANGE, UART_PIN_NO_CHANGE, UART_PIN_NO_CHANGE, UART_PIN_NO_CHANGE);
     //Install UART driver, and get the queue.
     uart_driver_install(EX_UART_NUM, BUF_SIZE * 2, BUF_SIZE * 2, 20, &uart0_queue, 0);

## Harder Tasks

So, using Arduino makes sense unless you need something completely different.

![Working sniffer with lots of Arduino Code](https://cdn-images-1.medium.com/max/2000/1*rf0I5DT8zV6CKSF8nncMaQ.gif)*Working sniffer with lots of Arduino Code*

Here’s how you set up the wifi port as a sniffer in Arduino:

    {}

You can’t.

Here’s my sample code to set up the wifi port as a sniffer in Arduino, while using ESP-IDF

    #include "C:\msys32\home\Mark\esp\esp-idf\components\esp32\include\esp_wifi.h"

    // set up Wifi for promiscuous mode, random channel
    void WifiInitialize()
    {
    	wifi_init_config_t config = WIFI_INIT_CONFIG_DEFAULT();
    	esp_wifi_init(&config);
    	// Promiscuous works only with station mode	esp_wifi_set_mode(WIFI_MODE_STA);
    	esp_wifi_set_promiscuous(0);
    	// Set up promiscuous callback	esp_wifi_set_promiscuous_rx_cb(promisc_cb);
    	esp_wifi_set_promiscuous(1);
    	esp_wifi_set_channel(1, WIFI_SECOND_CHAN_NONE );
    }

    // the callback function
    void promisc_cb(void *buf, wifi_promiscuous_pkt_type_t ttype)
    {
    wifi_promiscuous_pkt_t* snifpak = (wifi_promiscuous_pkt_t*)buf;
    wifi_pkt_rx_ctrl_t& snifctrl = snifpak->rx_ctrl;  // the metadata
    ... more code
    }

## The Answer

***Just include the correct header**. That’s it.*
The libraries are already available, you just need to know that. The pre-built ESP-IDF libraries are embedded into the Arduino library build (in Tools/SDK/lib) and linked. The headers come with the Espressif-IDF install.

In the above code I point to my location for the right header. You would normally just set an include folder. The include path I use is: *C:/msys32/home/Mark/esp/esp-idf/components/esp32/include. *The path is from following Espressif’s installation instructions for the IDF component.

Once the code is written, use Arduino to compile it, which it will happily do — and then run it. There are some limitations to this approach (the Espressif IDF is more current than the Arduino, for example) but for most libraries it works admirably.

## Running Linux on Windows

By the way, if you are running Windows 10 I would not use the msys32 and mingw applications suggested by Espressif. I would absolutely use the Windows 10 [run Linux in Windows feature](https://docs.microsoft.com/en-us/windows/wsl/install-win10) and install Ubuntu. Just my 2cents.

**Note**: *in the GIF above showing MAC addresses I’ve specifically edited my code to not show the right MAC addresses. Instead it’s a random hash based off the MAC address.*
