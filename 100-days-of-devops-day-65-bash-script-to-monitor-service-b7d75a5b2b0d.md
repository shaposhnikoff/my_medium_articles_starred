
# 100 Days of DevOps — Day 65-Bash Script to Monitor Service

Welcome to Day 65 of 100 Days of DevOps, Focus for today is Bash Script to Monitor Service

***Step1:** How to check if the service is running?*

* *There are a number of ways we can check it*

* *First approach netstat*

    *# netstat -antulp|grep httpd*

    *tcp6       0      0 :::80                   :::*                    LISTEN      19122/**httpd***
[**100 Days of DevOps — Day 62-Useful Linux Command for Network Troubleshooting**
*Welcome to Day 62 of 100 Days of DevOps, Focus for today is useful Linux Command for Network Troubleshooting*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-62-useful-linux-command-for-network-troubleshooting-920430a2f75f)

* *The second approach is ps command*

    *# ps aux|grep -v grep |grep -i httpd*

    *root     19122  0.0  0.5 228280  5116 ?        Ss   Mar22   0:13 /usr/sbin/**httpd** -DFOREGROUND*

    *apache   19123  0.0  0.3 228416  3740 ?        S    Mar22   0:00 /usr/sbin/**httpd** -DFOREGROUND*

    *apache   19124  0.0  0.3 228416  3740 ?        S    Mar22   0:00 /usr/sbin/**httpd** -DFOREGROUND*

    *apache   19125  0.0  0.3 228416  3740 ?        S    Mar22   0:00 /usr/sbin/**httpd** -DFOREGROUND*

    *apache   19126  0.0  0.3 228416  3740 ?        S    Mar22   0:00 /usr/sbin/**httpd** -DFOREGROUND*

    *apache   19127  0.0  0.3 228416  3740 ?        S    Mar22   0:00 /usr/sbin/**httpd** -DFOREGROUND*

    *apache   19128  0.0  0.3 228416  3740 ?        S    Mar22   0:00 /usr/sbin/**httpd** -DFOREGROUND*

    *apache   19130  0.0  0.3 228416  3740 ?        S    Mar22   0:00 /usr/sbin/**httpd** -DFOREGROUND*

    *apache   19898  0.0  0.3 228416  3740 ?        S    Mar22   0:00 /usr/sbin/**httpd** -DFOREGROUND*

    *apache   19899  0.0  0.3 228416  3740 ?        S    Mar22   0:00 /usr/sbin/**httpd** -DFOREGROUND*

    *apache   19900  0.0  0.3 228416  3740 ?        S    Mar22   0:00 /usr/sbin/**httpd** -DFOREGROUND*

***Step2:** Once we checked if service is up and running, stored its status in some variable*

    *SERVICESTATUS=`echo $?`*

*Every command returns an exit status (sometimes referred to as a return status or exit code). A successful command returns a 0, while an unsuccessful one returns a non-zero value that usually can be interpreted as an error code.*

***Step3: **Use conditional statement(if-else) to determine if service is not running start it*

    *if [ "$SERVICESTATUS" != 0 ]*

    *then*

    *systemctl start $SERVICENAME*

    *echo "$SERVICENAME started"*

    *else*

    *echo "$SERVICENAME is already running"*

    *fi*

* *If the exit code is anything except zero starts the service(This is Centos7 system that why I am using systemctl) else just echo the message service is already running*

***Output***

* *If service is running*

    *# bash /tmp/httpd.sh*

    *tcp6       0      0 :::80                   :::*                    LISTEN      19122/httpd*

    *httpd is already running*

* *In case service is stopped*

    *# bash /tmp/httpd.sh*

    *httpd started*

<iframe src="https://medium.com/media/9614fed522a3df58d6e2444597cc6252" frameborder=0></iframe>

*Another way of writing the same script*

<iframe src="https://medium.com/media/993f21ac237e332d978d2fcf53929a28" frameborder=0></iframe>

    ** Check the number of httpd process*

    *# ps -ef | grep -v grep | grep httpd | wc -l*

    *6*

    ** If it greater then zero, then service is already running, else start the service*

* *Now this work perfectly fine, if I need to check one service, what would be the case if I need to verify a bunch of services*

<iframe src="https://medium.com/media/1c326de2394bddfbc9f943a2a1059507" frameborder=0></iframe>

    ** In this case we can define function(eg:servicecheck) and with the help of positional parameter we can pass service name to the function*

* *Another approach which I saw used by many people is to notify admins via email before starting the service*

    *echo "$servicename is not running" | mail -s "$servicename is down" test@gmail.com*

<iframe src="https://medium.com/media/2e0848dea2aa3a17dda26d658ca1dac6" frameborder=0></iframe>

* *I barely scratch the surface and these are one of the many ways. we can monitor service using shell script, please feel free to share your script or ideas on how to enhance this script*

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
