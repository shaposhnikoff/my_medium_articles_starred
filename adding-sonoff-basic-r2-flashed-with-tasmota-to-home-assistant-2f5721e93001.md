
# Adding Sonoff Basic R2 flashed with Tasmota to Home Assistant

I used a couple Sonoff Basics to automate my Christmas lights this year. I setup a few automations based on how we wanted them to turn on and off. We are going to walk through manually adding a Sonoff Basic R2 flashed with the Tasmota software without using auto discovery. Then we will setup a basic automation. If you haven’t already flashed and configured Tasmota you might want to checkout my article about that first.
[**Sonoff Basic R2 + Tasmota**
*Flashing Sonoff Basic R2 with Tasmota and no soldering.*medium.com](https://medium.com/@jordanrounds/sonoff-basic-r2-tasmota-aa6f9d4e033f)

### Switch Template:

The template is pretty straight forward, and you can reference the docs for adding an [MQTT Switch](https://www.home-assistant.io/integrations/switch.mqtt/) and all the different configuration options that are available. You can copy the template below into *config/configuration.yaml*.

<iframe src="https://medium.com/media/d43350888749a1b02b24484ba865b745" frameborder=0></iframe>

This switch template is assuming you used the same topic formatting in my other article. Be sure your topics match what you configured in the Tasmota web ui.

### The Automation

This automation will turn on the lights in the morning around when I get up, then shut them off again around work. They then come on at night and shut off around bed time.

<iframe src="https://medium.com/media/805c87d15b471ed65a10d4ab3a66f648" frameborder=0></iframe>

If you were using this automation for outdoor lights you might want to use the [trigger](https://www.home-assistant.io/cookbook/automation_sun/) for sunrise and sunset.

    trigger:
      platform: sun
      event: sunrise

### Wrapping Up

All thats left to do at this point is to check the config, and if there are no errors then restart Home Assistant. In the left nav click on Configuration and then Server Control. You’ll see this screen.

![](https://cdn-images-1.medium.com/max/3940/1*WZ3Ec6r8-G1kUlYhsnsFxg.png)

Once you’re here click the Check Config button and give it a minute. You should see a Configuration valid message. If not you will see errors telling you the lines that have the issue.

![](https://cdn-images-1.medium.com/max/2416/1*Zyn5-jnXlSCb2Wq5gWr5VA.png)

After the configuration is validated, click the restart button down at the bottom to restart Home Assistant.

Now you’re set to go with some automated Christmas lights.
