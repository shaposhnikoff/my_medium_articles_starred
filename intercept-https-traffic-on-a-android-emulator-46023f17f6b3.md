
# Intercept HTTPS Traffic On A Android Emulator

Sometimes we are in a situation where we want to see all the network traffic happening in a app. This can be quite troublesome and we therefor need a easy way of doing it.
In this article I will cover how to intercept HTTP/HTTPS traffic from a Android emulator by using a MITM (Man In The Middle) Proxy.

![](https://cdn-images-1.medium.com/max/2000/1*6SPvBeNj336kduC7XnYxmA.jpeg)

For getting started we need two things

* [Mitmproxy](https://mitmproxy.org/) (For creating our proxy server)

* [Android emulator](https://developer.android.com/studio/run/emulator) (For running the application on)

We start by installing Mitmproxy, they have nice [installation guides](https://mitmproxy.org/) on their website. When using Mac with HomeBrew you can do brew install mitmproxy.
After installing Mitmproxy, we need to setup a android emulator. The approach we are taking here requires us to use a Android version lower than 7. I use Android 6 (API 23).

![AVD example of setup with Android 6 (API 23)](https://cdn-images-1.medium.com/max/2044/1*0mUwTX4o0sKkopzG3GuTsw.png)*AVD example of setup with Android 6 (API 23)*

After installing all requirements, we will start the proxy and setup the emulator to use it. 
For starting the proxy we run mitmweb, this will start up a proxy running on port 8080. (mitmweb opens a website showing request, if you want to see it in the terminal only use mitmproxy instead)
For using the proxy with the emulator, open the emulator and go to the configuration window. Under the configuration window go to Settings→Proxy and change it to manual proxy config. 
Then enter the IP address of your computer (you can use ifconfig to find it) and the default port for the proxy is 8080. Remember that the image is showing an example with my local IP, yours will probably be different.

![Android emulator settings example for my computer](https://cdn-images-1.medium.com/max/3304/1*7_7o7EZ47CkKPHzdOFaTMw.png)*Android emulator settings example for my computer*

Now your proxy is all setup, the next step is to install the certificate. To install the certificate go to [mitm.it](http://mitm.it) on the android simulator and you should see the following screen.

![Android emulator showing mitm.it](https://cdn-images-1.medium.com/max/2000/1*6pmvdIptxA0AV-AYWApUhg.png)*Android emulator showing mitm.it*

On this screen choose android and give the certificate a name. I named mine mitm.it, but the name can be whatever you like. After installing the certificate, you are ready to use the proxy. 
Open up your application. You can now view your request in your browser so you can troubleshoot what went wrong.

![Example of data received through mitmproxy](https://cdn-images-1.medium.com/max/6720/1*_wq2goC1o4yr1uelXkWoUw.png)*Example of data received through mitmproxy*

Keep in mind that some apps might not allow you to do this, as this could be breaking the terms you agreed to.
