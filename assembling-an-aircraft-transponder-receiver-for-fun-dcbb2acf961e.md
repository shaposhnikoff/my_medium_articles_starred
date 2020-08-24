
# Assembling an Aircraft Transponder Receiver — for Fun

Assembling an Aircraft Transponder Receiver — for Fun

A couple of weekends ago, I hacked together an airplane transponder data receiver. “Hack” is a bit of a stretch here since it’s really just plugging a bunch of pipes together — but nonetheless I wanted to write an end-to-end post on how to build this for anyone who might be interested.

First, there’s a flurry of websites out there that allow you to track aircrafts around the world based on their position, altitude, speed, direction, and more. Examples of such sites include [Flightradar](https://www.flightradar24.com), [FlightView](https://www.flightview.com) and others. They’re pretty accurate in their tracking — which is quite useful if picking up a friend at the airport — and based on the near real-time aspect of the data, one may wonder if the FAA has a public API somewhere for developers to tap into.

Sadly, no.

## **Sourcing Live Aircraft Data**

Air Traffic Control (“ATC”) uses two radar types to track planes: primary radar is pretty basic and simply sweeps the skies for bounces; secondary radar listens to transponders and provides more contextual info to flight controllers.

Automatic Dependent Surveillance-Broadcast (“ADS-B”) is the system that provides this secondary radar info to ATC. ADS-B emits a signal roughly every second and uses the 1090 MHz frequency. Since the signal is not encrypted, websites displaying live aircraft can also listen to this signal to collect data.

The ADS-B signal however has a limited range so in order to get good geographical coverage of airplane transponder data you need a distributed network of antenna. That’s where the plane nerds (aka me!) come in.

As it turns out, most of the data you see on websites seems to be crowd-sourced by aviation fans. Those guys are kind of nuts about planes (e.g. [plane spotters](https://www.planespotters.net/)) and are more than happy to run base stations out of their garages. Furthermore, FlightRadar even [provides a free listening device](https://www.flightradar24.com/apply-for-receiver) to anyone who can help add geographical coverage. Conceptually, it’s still interesting to put one together which is the purpose of this blog post.

Last but not least, I’m not a lawyer but U.S. [law seems pretty clear that you can listen](https://www.law.cornell.edu/uscode/text/47/605) but not necessarily retransmit. I’m not entirely sure how websites are able to get around this to republish the data but for what it’s worth, I haven’t found any case law regarding this.

## **Building Your Receiver**

Alright so now that we know how this stuff kind of works, let’s put together an ADS-B receiver! First, there are 3 steps to building a receiver to capture data from planes flying near you:

1. Getting a USB radio antenna

1. Decoding the signal received on 1090 Mhz

1. Visualizing the data

**Getting The Hardware**

Without doing too much research, I first started by purchasing a [NooElec Dual-Band NESDR Nano 2 ADS-B receiver](https://www.amazon.com/NooElec-Dual-Band-Foreflight-FlightAware-Applications/dp/B01K5K3858). You can probably find cheaper ones elsewhere but this one works fine on my Macbook.

![](https://cdn-images-1.medium.com/max/2400/1*49wsHUBovK_ZYQDmEeFPcQ.png)

Once you’ve received your hardware, connect the 1090Mhz antenna to one of the wires, and connect the wire to your SDR dongle. Plug it in your laptop and run the command below:

    $ system_profiler SPUSBDataType

![](https://cdn-images-1.medium.com/max/4324/1*rC42_p9rqItdNf2sZcHRww.png)

Your device is now successfully connected.

**Testing The Hardware**

Before you start decoding transponder signal, it’s kind of neat to see if your SDR is actually receiving signal. SDR stands for [Software Defined Radio](https://en.wikipedia.org/wiki/Software-defined_radio), and you will want to download [CubicSDR](http://cubicsdr.com/) (and don’t forget to [donate](http://cubicsdr.com/?page_id=86) to this great open-source project) to visualize incoming signal.

Once you have installed and launched CubicSDR, go to File > SDR Devices. It should look like this:

![](https://cdn-images-1.medium.com/max/5760/1*KygqEH9TlcDFSgCFBd8SiA.png)

Click on your SDR device (in my case, Generic RLT2832U) and press start. You will then need to tune in. For fun, try to pick up a local FM station. To do so, locate the frequency adjuster on the right side:

![](https://cdn-images-1.medium.com/max/2672/1*LYySbHfj2v-d7CZcuvNKPw.png)

Press the space bar so that you can punch in your frequency. In San Francisco, I chose to tune in to my local NPR station (KQED)

![](https://cdn-images-1.medium.com/max/2000/1*V015S646W7tx3d0tjHnDrg.png)

Once the frequency is adjusted, this is what you should see:

![Each green/red band is a radio station’s signal. You can move around to check out different station or listen to your local city dispatch.](https://cdn-images-1.medium.com/max/6208/1*_pGatMTH_6g4gmBDD5jsZg.png)*Each green/red band is a radio station’s signal. You can move around to check out different station or listen to your local city dispatch.*

Once you’re tuned in, turn up the volume, and you should be able to hear your local station. Now let’s tune into 1090 MHz to capture ADS-B signal. Again, 1090MHz is where aircraft transponders broadcast their real-time status. Repeat the tuning procedure above but this time type in “1090 MHz” and press enter. You should now see this:

![](https://cdn-images-1.medium.com/max/6208/1*TltozQfQZHhSueB-U5OjQA.png)

If there are planes nearby, you will see dots on your main visualizer. If you zoom in, you should see this in the top visualizer:

![](https://cdn-images-1.medium.com/max/2664/1*R9PHCm4rmAITfyQn2TEn9w.png)

As far as I can tell, each red spot is a data payload from an aircraft’s transponder. I live roughly 10 miles from San Francisco International Airport (KSFO) so there’s a fair amount of air traffic around my place. For example, here’s a photo of an aircraft flying above my place as I captured this signal:

![Can you spot the plane?](https://cdn-images-1.medium.com/max/3392/1*3H4i7ylVBBPkKwM2MRqyTw.png)*Can you spot the plane?*

Anyways, if you see these red spots on your visualizer, congrats, you’re now receiving transponder data.

**Decoding The Data**

Alright so now that we’re getting data, we’ve got to decode it. I’ll skip the analog to digital theory, as well as how to [decode the binary](https://adsb-decode-guide.readthedocs.io/en/latest/) packets coming in. Let’s go straight to the easy part: there’s a neat library out there called [RTL-SDR](http://www.rtl-sdr.com/) that does all the heavy-lifting for you. Fire up your terminal and install it running your favorite package manager. I use brew, so I ran:

    $ brew install rtl-sdr

Once it’s installed, run it using

    $ rtl_tcp

You should now see this:

![](https://cdn-images-1.medium.com/max/4324/1*i11YhP7mxSQzgv_BswDGYA.png)

This will decode the incoming signals and make the payload available via tcp. Lastly to visualize the data, go download [Cocoa1090](http://www.blackcatsystems.com/software/cocoa1090.html). When you start Cocoa1090, the default settings should be good to connect to your running instance of rtl_tcp. In case you’re not seeing data, go to the preference menu and make sure that source Address and Port are as followed:

![](https://cdn-images-1.medium.com/max/2368/1*p35oUqJLzt0Qo1J5jHRkxw.png)

If you’re successfully getting data, here’s what it should look like:

![](https://cdn-images-1.medium.com/max/4428/1*eOELNs0ufsSVjZ5A5VTWFg.png)

It’s not exactly rush hour at KSFO and KOAK (Saturday night, 9PM Pacific Time) but there’s a few planes in the air. You can look up some of the aircraft by their tail number using [N Number Search](https://nnumber.org/search-results.php). For example, the Boeing 737 with tail N731SA is registered with Southwest Airline.

![](https://cdn-images-1.medium.com/max/4816/1*qkUibyqOtUci0Sp9aP7dmQ.png)

I quickly looked up this tail number on FlightRadar and it looks like flight SWA4267 is headed to Spokane. Godspeed SWA4267!

![](https://cdn-images-1.medium.com/max/4344/1*eoPMat8YL6O4H07diqSpug.png)

**So What’s Next?**

For starter, you could probably build a better antenna. There’s also half a dozen ways to visualize the data on an actual map. A friend of mine also suggested writing a script that runs on a computer at the FOB and detects when his Cessna is about to land in order to call him a Lyft home.

Lastly, maybe you can just spot your friends flying into your local airport and impress them. Or not.

![](https://cdn-images-1.medium.com/max/4656/1*tqD_HNjh3BC9PSiBdR9ozQ.png)

*Acknowledgement: Vicky for letting me spend $30 to buy an antenna and a Sunday afternoon buried in my laptop; my dogs [Jasmine](https://www.instagram.com/senorita.jasmine/) and [June](https://www.instagram.com/bernesejune/) for leaving me alone while I was trying to figure this stuff out.*
