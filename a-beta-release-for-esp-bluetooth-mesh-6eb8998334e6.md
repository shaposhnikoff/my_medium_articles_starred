
# A Beta Release for ESP Bluetooth Mesh

Surprisingly perhaps, the Espressif ESP8266 and, more recently now the ESP32, both have official support for Wi-Fi mesh networking. In a mesh network, nodes can self-organise and dynamically talk to each other. Any node in the network is able to transmit data to any other node within range, which can then forward packets through the network to their final destination. If nodes are removed from the network, it should self-heal, and route around the damage.

The idea the Bluetooth too could act in a mesh arrived much more recently with the [Bluetooth 5](https://www.bluetooth.com/bluetooth-technology/bluetooth5) standard, and right now has fairly minimal [real world support](https://blog.hackster.io/an-nrf52840-based-bluetooth-5-dongle-94fff8e71da6). Which means that [yesterday’s announcement](https://esp32.com/viewtopic.php?f=13&t=7875) of support for BLE Mesh networking on the ESP32 is actually far more important than it might seem on the surface.

![The ESP32 chip. (📷: Alasdair Allan)](https://cdn-images-1.medium.com/max/8064/1*tmQLQOTz6MxOG6LkI9WGCg.jpeg)*The ESP32 chip. (📷: Alasdair Allan)*

Yesterday’s release is very much a beta release and, as such, Espressif has made a temporary [BLE Mesh branch](https://github.com/espressif/esp-idf/tree/feature/esp-ble-mesh-v0.5) of their official [IoT Development Framework](https://github.com/espressif/esp-idf) for the ESP32 for people to take a look at it. So be warned, this isn’t production ready code, at least not yet.

Right now the temporary branch [supports](https://esp32.com/viewtopic.php?f=13&t=7875) provisioning, along with relay, and segmentation of the mesh network, as well as proxies, and a range of generic client models such lighting and sensors. Built on top of Zephyr BLE Mesh stack the ESP SDK supports network provisioning and node control as well as node features like Proxy, Relay, Low Power and Friend.

At present, only the [ESP32-WROOM-32](https://www.espressif.com/sites/default/files/documentation/esp32-wroom-32_datasheet_en.pdf) and [ESP-WROVER KIT](https://www.espressif.com/en/products/hardware/esp-wrover-kit/overview) are officially supported for BLE Mesh implementation, but presumably support will broaden with later releases.

If you’re interested in [getting started](https://github.com/espressif/esp-idf/blob/feature/esp-ble-mesh-v0.5/components/bt/ble_mesh/mesh_docs/BLE-Mesh_Getting_Started_EN.md) with BLE mesh networking on the ESP32 you should grab the [beta branch of the Espressif IDF](https://github.com/espressif/esp-idf/tree/feature/esp-ble-mesh-v0.5), which supports mesh networking, and take a look at the [code](https://github.com/espressif/esp-idf/tree/feature/esp-ble-mesh-v0.5/components/bt/ble_mesh) and [examples](https://github.com/espressif/esp-idf/tree/feature/esp-ble-mesh-v0.5/examples/bluetooth/ble_mesh). But make sure to file any [issues](https://github.com/espressif/esp-idf/tree/feature/esp-ble-mesh-v0.5/examples/bluetooth/ble_mesh) you have with the beta, so that the team at Espressif can take a look at them.

![](https://cdn-images-1.medium.com/max/4000/1*tR-jNXEQIlTnHDyLChWiQQ.png)
