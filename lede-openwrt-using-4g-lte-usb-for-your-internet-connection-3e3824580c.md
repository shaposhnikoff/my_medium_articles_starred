
# LEDE/OpenWRT — Using 4G LTE USB For Your Internet Connection

This guide will walk you through setting up your OpenWRT device to use a 4G LTE/3G USB dongle as it’s source of internet.

## SSH to your LEDE/OpenWRT device

If you are using Windows then start [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) and click **Session** on the left side, select **SSH** from the options, and then enter in the IP Address of your LEDE/OpenWRT box into the **Host Name** field.

Once you’ve done this just click on **Open** to start up the SSH connection.

![PuTTY](https://cdn-images-1.medium.com/max/2000/1*ArocFHqU2HFLSE-YiUjY1Q.png)*PuTTY*

If you are connecting via terminal, then just SSH to your LEDE/OpenWRT device using the following command, where 192.168.1.1 is your LEDE/OpenWRT device’s IP address.

    ssh root@192.168.1.1

## Prerequisits

You will need a LTE/3G USB dongle that provides/can provide an NDIS interface (QMI Mode), and enable it.

Once enabled, connect it to your OpenWRT device.

## Installation

We want to install all of the required packages needed to get this working, so run the following commands:

    opkg update

    opkg install usb-modeswitch kmod-mii kmod-usb-net kmod-usb-wdm kmod-usb-net-qmi-wwan uqmi

Once everything is installed, give your OpenWRT device a reboot.

## Configuration

First we want to see if everything is running correctly, so run the following commands:

    uqmi -d /dev/cdc-wdm0 --get-data-status

This should return a disconnected status.

Then run:

    uqmi -d /dev/cdc-wdm0 --get-signal-info

Which should output some info on your signal.

Once you have confirmed both of these work we need to run a command to initialise your internet connection. Replacing PROVIDER_APN with the APN from your provider.

    uqmi -d /dev/cdc-wdm0 --start-network PROVIDER_APN --autoconnect

Then run the two previous commands again to confirm you are connected.

## Network Config

Now we need to create a new interface for this connection.

Simply run the following command:

    vi /etc/config/network

Then add the following in:

    config interface 'wwan'
            option ifname 'wwan0'
            option proto 'dhcp'

After doing this, the final step is to add a firewall rule in. The easiest way to do this is through the LuCI interface.

* Head into Network > Firewall

* Scroll down to WAN and click edit

* Under the covered networks heading, tick the box next to wwan.

* Click save and apply to confirm your changes

**If you found this post helpful please let us know by clicking the ♥ below.**

*This blog was brought to you by [Cucumber Wi-Fi](http://www.cucumberwifi.io). Cucumber helps you run a more efficient Wi-Fi network. Check it out [here](http://www.cucumberwifi.io).*

*Cucumber Wi-Fi — control any (Wi-Fi) device from the cloud.*
