
# 100 Days of DevOps — Day 63- Wireshark for HTTP/HTTPS Analysis

Welcome to Day 63 of 100 Days of DevOps, Focus for today is HTTP/HTTPS analysis using Wireshark.

*Yesterday on Day62, I wrote about some useful Linux command to debug Network issue*
[**100 Days of DevOps — Day 62-Useful Linux Command for Network Troubleshooting**
*Welcome to Day 62 of 100 Days of DevOps, Focus for today is useful Linux Command for Network Troubleshooting*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-62-useful-linux-command-for-network-troubleshooting-920430a2f75f)

*Let extend that concept further and see about very useful tool Wireshark to dig deeper.*

***How HTTP works***

*To Demonstrate that let’s use Sample Captures from Wireshark website(http.cap)*
[**SampleCaptures — The Wireshark Wiki**
*It’s also a very good idea to put links on the related protocol pages pointing to your file. Referring to an attachment…*wiki.wireshark.org](https://wiki.wireshark.org/SampleCaptures#HyperText_Transport_Protocol_.28HTTP.29)

*Before start analyzing any packet, please turn off “**Allow subdissector to reassemble TCP streams**”(Preference → Protocol → TCP)(This will prevent TCP packet to split into multiple PDU unit)*

![](https://cdn-images-1.medium.com/max/2920/1*6QqDbFqDnN_1jHS2B0K8eQ.png)

![*http.cap*](https://cdn-images-1.medium.com/max/5748/1*O75zSb5buGfmc4r-Xotm2g.png)**http.cap**

*As you can see I am using HTTP so that the encryption will not be hidden behind TLS.*

*As you can see at line number 13 standard DNS resolution is happening.*

![](https://cdn-images-1.medium.com/max/5760/1*sAgUaYLb-y_SFkQzU_cYjg.png)

*In line number 17 you see the response we are getting back with full DNS resolution*

![](https://cdn-images-1.medium.com/max/5760/1*Eo1wyTOsrFiA_9JBvOvsGg.png)

*Now if you look at Packet number 4 i.e is get request,HTTP primarily used two command*

*1:** GET:** To retrieve information*

*2: **POST:** To send information(For eg: when we submit some form we fill some data i.e is POST)*

![](https://cdn-images-1.medium.com/max/5760/1*oyPPjW_klKtW-lE0A1kdJw.png)

*Here I am trying to get download.html via HTTP protocol 1.1(The new version of protocol is now available i.e 2.0)*

*Then at line number 5 we see the acknowledgment as well as line number 6 server was able to found that page and send HTTP status code 200.*

*If you want more info about **HTTP status code***
[**HTTP Status Codes**
*HTTP status codes and how to use them in RESTful API or Web Services.*www.restapitutorial.com](http://www.restapitutorial.com/httpstatuscodes.html)

*You will see some more info like for packet 6, like Server type is Apache, content type is HTML, how long is the content length is,*

![](https://cdn-images-1.medium.com/max/5232/1*JF3_JDHl0d8SXij1l4qFRg.png)

*Then you will see bunch of continuation that is due to TCP window where you don’t get acknowledgement for each and every packet*

![](https://cdn-images-1.medium.com/max/5672/1*ACIMy-fIOZurcyUiyfDCqg.png)

*and at that top some usual TCP handshake*

![](https://cdn-images-1.medium.com/max/5540/1*_S1C4Qu86cLkl9gfYbHXmA.png)

***Now lets try to dissect HTTPS capture***
[**SampleCaptures — The Wireshark Wiki**
*It’s also a very good idea to put links on the related protocol pages pointing to your file. Referring to an attachment…*wiki.wireshark.org](https://wiki.wireshark.org/SampleCaptures#SSL_with_decryption_keys)

![*snakeoil2*](https://cdn-images-1.medium.com/max/5744/1*V5Fftdzs9tP48Gkx7Zdltw.png)**snakeoil2**

*as you can see*

* *3 way handshake is happening,*

* *hello from SSL client and then acknowledgement from Server*

* *Server Hello and then ACK*

* *Exchanging some key and cipher information*

* *Finally it actually start exchanging data.*

*Then if we click on any application data that data is unreadable to us it’s all gibberish but with wireshark we can decrypt that data only thing we need is the Private Key of the server.*

![](https://cdn-images-1.medium.com/max/5760/1*GX9egA81_ea0ypIALRFY1g.png)

*Once again go to Preference → Protocol → SSL*

*Add these value*
> # *IP address: 127.0.0.1*
> # *Port: 443*
> # *Protocol: http*
> # *Key File: [https://wiki.wireshark.org/SampleCaptures?action=AttachFile&do=get&target=snakeoil2_070531.tgz](https://wiki.wireshark.org/SampleCaptures?action=AttachFile&do=get&target=snakeoil2_070531.tgz)*

![](https://cdn-images-1.medium.com/max/2228/1*AHDMZZI-ePmdtNdHb7y6sg.png)

*as you can see data is now decrypted*

![](https://cdn-images-1.medium.com/max/5200/1*Iu6qVdRtNbAO3xGF9PvYmw.png)

*Looking forward from you guys to join this journey and spend a minimum an hour every day for the next 100 days on DevOps work and post your progress using any of the below medium.*

* *Twitter: [@100daysofdevops](http://twitter.com/100daysofdevops) OR @[lakhera2015](https://twitter.com/lakhera2015)*

* *Facebook: [https://www.facebook.com/groups/795382630808645/](https://www.facebook.com/groups/795382630808645/)*

* *Medium: [https://medium.com/@devopslearning](https://medium.com/@devopslearning)*

* *Slack: [https://devops-myworld.slack.com/messages/CF41EFG49/](https://devops-myworld.slack.com/messages/CF41EFG49/)*

* *GitHub Link:[https://github.com/100daysofdevops](https://github.com/100daysofdevops)*

***Reference***
[**100 Days of DevOps — Day 0**
*D-day is just one day away and finally, this is a continuation of the post(I posted a month earlier)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-0-4f2c9750542d)
[**100 Days of DevOps**
*Motivation*medium.com](https://medium.com/@devopslearning/100-days-of-devops-81faf13bf772)
