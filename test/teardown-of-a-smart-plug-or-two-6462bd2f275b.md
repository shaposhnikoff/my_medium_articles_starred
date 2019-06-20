
# Teardown of a Smart Plug (or Two)

Breaking open the Teckin SP23 and BlitzWolf BW-SHP4 smart plugs

Since I wrote about the [teardown of the LiFX Mini white bulb](https://blog.hackster.io/the-problem-with-throwing-away-a-smart-device-75c8b35ee3c7) a few weeks ago, [Limited Results](https://twitter.com/limitedresults) has gone back and [taken a look at a different bulb](https://limitedresults.com/2019/02/pwn-the-wiz-connected/), the [WIZ](https://www.wizconnected.com/en-GB/) connected bulb, and this time finding an [ESP-WROOM-02](https://www.espressif.com/en/products/hardware/esp-wroom-02/overview) module with an Espressif [ESP8266](https://www.espressif.com/en/products/hardware/esp8266ex/overview). It was equally hackable.

This left me thinking about how many other commercial Internet of Things smart devices might be hiding the same maker friendly, and entirely hackable, internals. So, ignoring smart bulbs, I decided to go and look at smart sockets.

![The [Teckin SP23](http://amzn.to/2UeqM8A) (left) and [Blitzwolf BW-SHP4](https://amzn.to/2SX3Kqw) (right). (📷: Alasdair Allan)](https://cdn-images-1.medium.com/max/7820/1*CqCjUhb-BRhr_3q8ASMjtA.jpeg)*The [Teckin SP23](http://amzn.to/2UeqM8A) (left) and [Blitzwolf BW-SHP4](https://amzn.to/2SX3Kqw) (right). (📷: Alasdair Allan)*

Heading to Amazon I picked up two, the [Teckin SP23](http://amzn.to/2UeqM8A) and [Blitzwolf BW-SHP4](https://amzn.to/2SX3Kqw) smart sockets. They’re both fairly cheap, and roughly the same size, and I had a sneaking suspicion they might both be fairly similar inside.

Going ahead and opening the Teckin smart socket with a [spudger](https://www.ifixit.com/Store/Tools/Metal-Spudger-Set/IF145-017-1), and a lot of brute force, it turned out to be built around an Espressif ESP8266 chip, just like the WIZ smart bulb, rather than the [ESP32](https://www.espressif.com/en/products/hardware/esp32/overview) used by the LiFX smart bulb.

![Cracking open the [Teckin SP23](http://amzn.to/2UeqM8A). (📷: Alasdair Allan)](https://cdn-images-1.medium.com/max/8064/1*vQZ-q7xiWFkXvIq9Xh6hKQ.jpeg)*Cracking open the [Teckin SP23](http://amzn.to/2UeqM8A). (📷: Alasdair Allan)*

The logic and power boards are entirely separate, with the logic board mounted vertically “through hole” in a slot in the main power board which houses the relay and mains switching hardware.

The main silicon on the logic board is the familiar [ESP8266EX](https://www.espressif.com/sites/default/files/documentation/0a-esp8266ex_datasheet_en.pdf) chip.

![The logic board. (📷: Alasdair Allan)](https://cdn-images-1.medium.com/max/5904/1*_Ba9qQFO9dogiGiYi9WTkA.jpeg)*The logic board. (📷: Alasdair Allan)*

However, there is another interesting piece of silicon on this side of the logic board, a small 8-pin chip labelled HJL-01. The clue to the function of this chip is when we flip the board over. There you can see that the neutral pin of the mains plug is connected through the logic board.

![The connector. (📷: Alasdair Allan)](https://cdn-images-1.medium.com/max/8064/1*HHgVCeWaH0kQCvNkSLLkOQ.jpeg)*The connector. (📷: Alasdair Allan)*

This makes a lot of sense, since the Teckin smart socket features the ability to monitor the energy usage through the plug. While I can’t find any information on the HJL-01, my current bet is that this is the silicon used to do the energy monitoring, and that the HJL-01 is a clone or copy of the HLW8012 chip used in a number of [other similar smart sockets](http://tinkerman.cat/hlw8012-ic-new-sonoff-pow/).

![The rear of the logic board. (📷: Alasdair Allan)](https://cdn-images-1.medium.com/max/6308/1*b5qKXHaAF6VHGE3POHzKIg.jpeg)*The rear of the logic board. (📷: Alasdair Allan)*

The only other piece of silicon on the logic board is on the rear of the board, and is a BoyaMicro [25D80](http://www.bmsemi.com/upload/file/20180129/15172094523103025.pdf), an 8Mbit SPI Flash memory chip.

At this point, for comparison purposes, I went ahead and opened up the Blitzwolf smart plug with the same [spudger](https://www.ifixit.com/Store/Tools/Metal-Spudger-Set/IF145-017-1), and this time a lot more brute force. If you want to be able to put the plug back together and use it again, I’d really recommend a hitting the external case of this plug with a rubber mallet around the sides—to attempt loosen the glue—before prying it open, because it’s really well secured.

![Cracking open the [Blitzwolf BW-SHP4](https://amzn.to/2SX3Kqw). (📷: Alasdair Allan)](https://cdn-images-1.medium.com/max/8064/1*ieWKsITuNb55J1Mj2j6Qiw.jpeg)*Cracking open the [Blitzwolf BW-SHP4](https://amzn.to/2SX3Kqw). (📷: Alasdair Allan)*

The insides of the second plug are superficially similar to the Teckin plug, although as you can see there are differences. The Blitzwolf smart plug is a two sided circuit board, with the through hole components on the top of the board and all the SMD mounted components on the rear, while the Teckin plug uses a single-sided board with through hole and SMD components on the top side.

![The [Blitzwolf BW-SHP4](https://amzn.to/2SX3Kqw) (left) and [Teckin SP23](http://amzn.to/2UeqM8A) (right). (📷: Alasdair Allan)](https://cdn-images-1.medium.com/max/8064/1*CY0jif8ZU6svMT_Ijl4GeQ.jpeg)*The [Blitzwolf BW-SHP4](https://amzn.to/2SX3Kqw) (left) and [Teckin SP23](http://amzn.to/2UeqM8A) (right). (📷: Alasdair Allan)*

This difference is a lot more evident when you sit the boards side by side. The Blitzwolf board look a lot cleaner from the top, but only because all the SMD components are hidden on the bottom of the board.

![](https://cdn-images-1.medium.com/max/8064/1*9tZpZl8QH5NwcpdEx3E_Gw.jpeg)

![The top (left image) and bottom (right image) of the [Blitzwolf BW-SHP4](https://amzn.to/2SX3Kqw) (left) and [Teckin SP23](http://amzn.to/2UeqM8A) (right) boards. (📷: Alasdair Allan)](https://cdn-images-1.medium.com/max/8064/1*_BAk-fsHrp42PqTgjXw9Dg.jpeg)*The top (left image) and bottom (right image) of the [Blitzwolf BW-SHP4](https://amzn.to/2SX3Kqw) (left) and [Teckin SP23](http://amzn.to/2UeqM8A) (right) boards. (📷: Alasdair Allan)*

What is similar is that the Blitzwolf board also separates the logic and power boards, with the logic board again being mounted vertically through hole. Interestingly here, though, the mains isn’t routed through the logic board, with the energy monitoring chip being located on the bottom of the power board.

![The energy monitoring chip, labelled BL 0937 (middle right). (📷: Alasdair Allan)](https://cdn-images-1.medium.com/max/4034/1*LjlhoOfP5ugnf3SEkIWQJg.jpeg)*The energy monitoring chip, labelled BL 0937 (middle right). (📷: Alasdair Allan)*

The chip, labelled BL0937 has been seen [on other smart sockets](https://github.com/xoseperez/espurna/issues/1502), is visible just below the logic board connector towards the top right of the rear of the board.

The logic board has the main silicon covered with an RF shield, prying that off with the supdger I found something rather unexpected.

![Removing the RF shield from the logic board. (📷: Alasdair Allan)](https://cdn-images-1.medium.com/max/6592/1*WLHMmImO3btj9a3vrXCg7g.jpeg)*Removing the RF shield from the logic board. (📷: Alasdair Allan)*

Instead of the now expected ESP8266, or ESP32, chip we find a rather less common—at least in the maker community—ESP8285 chip.

![Logic board with ESP8285 chip. (📷: Alasdair Allan)](https://cdn-images-1.medium.com/max/8064/1*onsoQet8lt1tLZvAf_CCBA.jpeg)*Logic board with ESP8285 chip. (📷: Alasdair Allan)*

Under the hood the Espressif [ESP8285](https://www.espressif.com/sites/default/files/documentation/0a-esp8285_datasheet_en.pdf) is just an ESP8266, but with 1MB of flash memory onboard, which explains the absence of an SPI flash chip similar to the one we saw in with the first plug.

Wiring the BlitzWolf plug to jumper wires and connecting it to USB to poke around with on serial connection is actually pretty trivial as the logic board itself is labelled on the silk screen.

![Wiring up the Blitzwolf’s ESP8285 module to UART serial. (📷: Alasdair Allan)](https://cdn-images-1.medium.com/max/8064/1*2DOYeS1kkxqdgJ9SAsi5lA.jpeg)*Wiring up the Blitzwolf’s ESP8285 module to UART serial. (📷: Alasdair Allan)*

Unfortunately we didn’t get so lucky with the Teckin smart plug, there were no convenient labels on the silkscreen to go by—although the ground and 3.3V pads were pretty obvious—so I had to get slightly more serious.

![Determining the Rx/Tx pins for the Teckin ESP8266 module. (📷: Alasdair Allan)](https://cdn-images-1.medium.com/max/8064/1*kA1i-wuymr8P7YKenZvjig.jpeg)*Determining the Rx/Tx pins for the Teckin ESP8266 module. (📷: Alasdair Allan)*

Soldering a a bunch of wires to the logic board connect, and breaking out my [logic analyser](https://amzn.to/2H25Ejc) the UART pins were still pretty easy to find. Which let me use the [esptool](https://github.com/espressif/esptool) suite of utilities to retrieve the contents of the memory, and just loading that into a hex editor let me retrieve the Wi-Fi credentials.

Just like [those smart bulbs](https://blog.hackster.io/the-problem-with-throwing-away-a-smart-device-75c8b35ee3c7).

However interestingly, we can go a lot further. Because, as you might well have gathered by now, it seems I was actually treading a well worn path.

Because both of these smart plugs are compatible with the alternative [Sonoff-Tasmota](https://github.com/arendst/Sonoff-Tasmota) open source ESP8266 firmware. This community firmware allows you to control the smart sockets over any of [MQTT](https://mqtt.org/), HTTP, serial, or even [KNX](https://www.knx.org/knx-en/for-professionals/What-is-KNX/A-brief-introduction/index.php) for integrations with other smart home systems.

While [reflashing the Blitzwolf BW-SHP4](https://github.com/arendst/Sonoff-Tasmota/wiki/BlitzWolf-BW-SHP4-(UK-Version)) involves open the plug up, just as I’ve done here, or perhaps more neatly [cutting a hole in the base](https://user-images.githubusercontent.com/9513181/50402552-62c05380-078f-11e9-9c18-a79e20af5078.jpg) to get to the logic board connectors, you can actually [reflash the Teckin SP23 over the air](https://github.com/arendst/Sonoff-Tasmota/wiki/Teckin-SP23).

While I’d trust the alternative open source firmware a lot more than whatever firmware was running on the plugs before hand, especially when it comes to [my privacy](https://medium.com/@aallan/has-the-death-of-privacy-been-greatly-exaggerated-f2c4f2423b5) and the possibility that these devices might be “reporting home,” opening up these plugs has confirmed what I pretty much already assumed.

There no way to make a computing device really secure. Therefore a modern approach to security is normally all about [defence in depth](http://a modern approach to security should be all about defence in depth, rather than any one individual security measure that would make a thing magically secure.), rather than a single measure that would make a thing secure. However the use of the ESP82xx means that these smart devices lack both [secure boot](https://docs.espressif.com/projects/esp-idf/en/latest/security/secure-boot.html) and [flash encryption](https://docs.espressif.com/projects/esp-idf/en/latest/security/flash-encryption.html), both are vital if you want to stop the sorts of attacks we’ve seen first [with the smart light bulbs](https://blog.hackster.io/the-problem-with-throwing-away-a-smart-device-75c8b35ee3c7), and now with these smart plugs.

They aren’t secure, and they can’t be made secure.

But, it’s all about relative risk. Reflashing these sorts of cheap smart plugs with community written [open source firmware](https://github.com/arendst/Sonoff-Tasmota) eliminates most of the known unknowns.

Just make sure you put them on their own network, rather than the one with your laptops and other personal devices, and don’t give in to the temptation to open ports on your router to allow you to control of them from the Internet.

At which point? They might be secure enough, and they’re certainly fun.

![](https://cdn-images-1.medium.com/max/4000/1*tR-jNXEQIlTnHDyLChWiQQ.png)
