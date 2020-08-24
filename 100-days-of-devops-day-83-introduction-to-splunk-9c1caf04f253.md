
# 100 Days of DevOps — Day 83-Introduction to Splunk

Welcome to Day 83 of 100 Days of DevOps, Focus for today is Introduction to Splunk

*As per wiki “Splunk (the product) captures, indexes and correlates (near)real-time data in a searchable repository from which it can generate graphs, reports, alerts, dashboards, and visualizations”*

***Built-In***

* *Front End: Python(CherryPy)*

* *Backend: C/C++*

***Downloading and Installing Splunk***

[*https://www.splunk.com/en_us/download.html](https://www.splunk.com/en_us/download.html)*

    *wget -O splunk-6.5.2–67571ef4b87d-linux-2.6-x86_64.rpm ‘[https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=x86_64&platform=linux&version=6.5.2&product=splunk&filename=splunk-6.5.2-67571ef4b87d-linux-2.6-x86_64.rpm&wget=true'](https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=x86_64&platform=linux&version=6.5.2&product=splunk&filename=splunk-6.5.2-67571ef4b87d-linux-2.6-x86_64.rpm&wget=true%27)*

    *# rpm -ivh splunk-6.5.2–67571ef4b87d-linux-2.6-x86_64.rpm*

    *warning: splunk-6.5.2–67571ef4b87d-linux-2.6-x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 653fb112: NOKEY*

    *Preparing… ################################# [100%]*

    *Updating / installing…*

    *1:splunk-6.5.2–67571ef4b87d ################################# [100%]*

    *complete*

***Starting Splunk***

    *# pwd*

    */opt/splunk/bin*

    ***# ./splunk start — accept-license***

    *Splunk> All batbelt. No tights.*

    *Checking prerequisites…*

    *Checking http port [8000]: open*

    *Checking mgmt port [8089]: open*

    *Checking appserver port [127.0.0.1:8065]: open*

    *Checking kvstore port [8191]: open*

    *Checking configuration… Done.*

    *Creating: /opt/splunk/var/run/splunk/appserver/i18n*

    *Creating: /opt/splunk/var/run/splunk/appserver/modules/static/css*

    *Creating: /opt/splunk/var/run/splunk/upload*

    *Creating: /opt/splunk/var/spool/splunk*

    *Creating: /opt/splunk/var/spool/dirmoncache*

    *Creating: /opt/splunk/var/lib/splunk/authDb*

    *Creating: /opt/splunk/var/lib/splunk/hashDb*

    *Checking critical directories… Done*

    *Checking indexes…*

    *Validated: _audit _internal _introspection _telemetry _thefishbucket history main summary*

    *Done*

    *New certs have been generated in ‘/opt/splunk/etc/auth’.*

    *Checking filesystem compatibility… Done*

    *Checking conf files for problems…*

    *Done*

    *Checking default conf files for edits…*

    *Validating installed files against hashes from ‘/opt/splunk/splunk-6.5.2–67571ef4b87d-linux-2.6-x86_64-manifest’*

    *All installed files intact.*

    *Done*

    *All preliminary checks passed.*

    *Starting splunk server daemon (splunkd)…*

    *Generating a 1024 bit RSA private key*

    *………++++++*

    *…………………………++++++*

    *writing new private key to ‘privKeySecure.pem’*

    *— — -*

    *Signature ok*

    *subject=/CN=ip-XXXX/O=SplunkUser*

    *Getting CA Private Key*

    *writing RSA key*

    *Done*

    *[ OK ]*

    *Waiting for web server at [http://127.0.0.1:8000](http://127.0.0.1:8000/) to be available… Done*

    *If you get stuck, we’re here to help.*

    *Look for answers here: [http://docs.splunk.com](http://docs.splunk.com/)*

    *The Splunk web interface is at [http://XXXX:8000](http://ip-172-31-8-8.us-west-2.compute.internal:8000/)*

***Enabling it at boot time***

    ***# ./splunk enable boot-start***

    *Init script installed at /etc/init.d/splunk.*

    *Init script is configured to run at boot.*

    ***# To disable it***

    ***#./splunk disable boot-start***

    *# To run Splunk as a specific user
    **#./splunk enable boot-start -user splunkuser***

*Once it’s up and running you will see screen like this*

![](https://cdn-images-1.medium.com/max/5760/1*gS6FBC2pBNx_6fl9s6sPUg.png)
> # Default username: admin
> # password: changeme

![](https://cdn-images-1.medium.com/max/5756/1*cW-ciGXSouCy9gD-SPK0tw.png)

***Adding Linux Logs to Splunk(local host)***

*Click on Add Data → Monitor → Files and Directories*

![](https://cdn-images-1.medium.com/max/4376/1*qG7fpjN8APxNKqNCq-qXmg.png)

*Set Source Type → Input Settings → Review → Done(For the first time go with default settings)(We have added /var/log/messages from the local box)*

*Finally, you will see data like this*

![](https://cdn-images-1.medium.com/max/5760/1*CpAgpECvstKmsILejv-Z0g.png)

***Setting up Splunk to receive logs from a remote machine***

***Forwarders**: These are the agents installed on the client side and used to send data to Splunk Server Indexer(Indexer: Where Splunk stores data in the form of indexes)*

*Splunk Provide two kind of forwarders*

* ***Universal Forwarder***

* ***Heavy Forwarder***

![](https://cdn-images-1.medium.com/max/2000/1*5Hm-pOqx5M3yVgmQQ1gSzw.png)

### Splunk Architecture

![](https://cdn-images-1.medium.com/max/2000/1*B4zkKWwsCe-YTjW4LLJSrw.png)

* ***Search Head:** Used for Visualization and connects to indexer to fetch the data. The user usually logs in to search head to search and visualize data.*

*There are two other components*

* ***Deployment Servers:** It manages all Splunk configuration/servers(indexer,forwarder..) from one machine. Let say I want to modify one file and push it to all servers that task I can do with the help of deployment servers.*

* ***Licensing Server: **Manage and monitor license usage*

*Under Setting Links → Forwarding and receiving*

![](https://cdn-images-1.medium.com/max/2584/1*L0JtQyR0BBfjo4ZEiM6Y8g.png)

*Under Receive data → Add New*

![](https://cdn-images-1.medium.com/max/5680/1*U4cOXnaVv3-q-hA_uZz-qA.png)

![](https://cdn-images-1.medium.com/max/2048/1*iwWFdJd5LnFOIGJpbEFu8w.png)

*Restart Splunk Server*

    ***# ./splunk stop***

    *Stopping splunkd…*

    *Shutting down. Please wait, as this may take a few minutes.*

    *.. [ OK ]*

    *Stopping splunk helpers…*

    *[ OK ]*

    *Done.*

    ***# ./splunk start***

    *Splunk> All batbelt. No tights.*

    *Checking prerequisites…*

    *Checking http port [8000]: open*

    *Checking mgmt port [8089]: open*

    *Checking appserver port [127.0.0.1:8065]: open*

    *Checking kvstore port [8191]: open*

    *Checking configuration… Done.*

    *Checking critical directories… Done*

    *Checking indexes…*

    *Validated: _audit _internal _introspection _telemetry _thefishbucket history main summary*

    *Done*

    *Checking filesystem compatibility… Done*

    *Checking conf files for problems…*

    *Done*

    *Checking default conf files for edits…*

    *Validating installed files against hashes from ‘/opt/splunk/splunk-6.5.2–67571ef4b87d-linux-2.6-x86_64-manifest’*

    *All installed files intact.*

    *Done*

    *All preliminary checks passed.*

    *Starting splunk server daemon (splunkd)…*

    *Done*

    *[ OK ]*

    *Waiting for web server at [http://127.0.0.1:8000](http://127.0.0.1:8000) to be available.. Done*

    *If you get stuck, we’re here to help.*

    *Look for answers here: [http://docs.splunk.com](http://docs.splunk.com)*

    *The Splunk web interface is at [http://XXXX:8000](http://ip-172-31-8-8.us-west-2.compute.internal:8000)*

    *#To check splunk status*

    ***# ./splunk status***

    *splunkd is running (PID: 5711).*

    *splunk helpers are running (PIDs: 5719 5727 5832 5889).*

*We can also **restart Splunk from WebUI***

***Settings → Server controls → Restart Splunk***

![](https://cdn-images-1.medium.com/max/2576/1*sXRfzEF9rEgppBU-zw7gaQ.png)

![](https://cdn-images-1.medium.com/max/2552/1*FpcTNmOeyRTSjhpa3CeyPg.png)

***Important files in Splunk***

    *# Log File Location*

    ***/opt/splunk/var/log***

    *# Config file location*

    ***/opt/splunk/etc***

*On the client side*

    *# wget -O splunkforwarder-6.5.2–67571ef4b87d-linux-2.6-x86_64.rpm ‘https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=x86_64&platform=linux&version=6.5.2&product=universalforwarder&filename=splunkforwarder-6.5.2-67571ef4b87d-linux-2.6-x86_64.rpm&wget=true'*

    ***# rpm -ivh splunkforwarder-6.5.2–67571ef4b87d-linux-2.6-x86_64.rpm***

    *warning: splunkforwarder-6.5.2–67571ef4b87d-linux-2.6-x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 653fb112: NOKEY*

    *Preparing… ########################################### [100%]*

    *1:splunkforwarder ########################################### [100%]*

*After Installing RPM, start the splunk forwarder*

    *# pwd*

    */opt/splunkforwarder/bin*

    ***# ./splunk start — accept-license***

    *This appears to be your first time running this version of Splunk.*

    *Splunk> All batbelt. No tights.*

    *Checking prerequisites…*

    *Checking mgmt port [8089]: open*

    *Creating: /opt/splunkforwarder/var/lib/splunk*

    *Creating: /opt/splunkforwarder/var/run/splunk*

    *Creating: /opt/splunkforwarder/var/run/splunk/appserver/i18n*

    *Creating: /opt/splunkforwarder/var/run/splunk/appserver/modules/static/css*

    *Creating: /opt/splunkforwarder/var/run/splunk/upload*

    *Creating: /opt/splunkforwarder/var/spool/splunk*

    *Creating: /opt/splunkforwarder/var/spool/dirmoncache*

    *Creating: /opt/splunkforwarder/var/lib/splunk/authDb*

    *Creating: /opt/splunkforwarder/var/lib/splunk/hashDb*

    *New certs have been generated in ‘/opt/splunkforwarder/etc/auth’.*

    *Checking conf files for problems…*

    *Done*

    *Checking default conf files for edits…*

    *Validating installed files against hashes from ‘/opt/splunkforwarder/splunkforwarder-6.5.2–67571ef4b87d-linux-2.6-x86_64-manifest’*

    *All installed files intact.*

    *Done*

    *All preliminary checks passed.*

    *Starting splunk server daemon (splunkd)…*

    *Done*

    *[ OK ]*

*Now to enable Splunk forwarder at boot time*

    ***# ./splunk enable boot-start***

    *Init script installed at /etc/init.d/splunk.*

    *Init script is configured to run at boot.*

*It’s always a good idea to change the default password(changeme)*

    ***# ./splunk edit user admin -password test1234 -role admin -auth admin:changeme***

    *User admin edited.*

*Now to add the forwarder*

    ***# ./splunk add forward-server 172.31.8.8:9997 -auth admin:Redhat12***

    *Added forwarding to: 172.31.8.8:9997*

*Now you can define which file you want to monitor(eg:/var/log/secure)*

    ***# ./splunk add monitor /var/log/secure***

    *Added monitor of ‘/var/log/secure’.*

![](https://cdn-images-1.medium.com/max/5756/1*uRYJg-IqJZNkvk0qLcupBQ.png)

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
[**100 Days of DevOps — Day 82- Python Object Oriented Programming(OOP)**
*Welcome to Day 82 of 100 Days of DevOps, Focus for today is Python Object Oriented Programming(OOP)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-82-python-object-oriented-programming-oop-44786b0184f6)
