
# 100 Days of DevOps — Day 67-Introduction to Chrony

Welcome to Day 67 of 100 Days of DevOps, Focus for today is Introduction to Chrony

*What is Chrony?*

* *Chrony is a versatile implementation of the Network Time Protocol (NTP). It can synchronize the system clock with NTP servers, reference clocks (e.g. GPS receiver), and manual input using wristwatch and keyboard. It can also operate as an NTPv4 server and peer to provide a time service to other computers in the network*

*Before we go further and discuss Chronyd, let discuss how Time is organized in case of Linux*

* *Hardware Clock: Stored in the BIOS chip(hwclock or timedatectl)*

* *System Time: Stored in Memory (timedatectl or NTP or Chrony)*

* *The easiest way to check the time in case of Linux is by using the date command*

    *# date*

    *Thu Apr 18 17:03:05 UTC 2019*

* *If you want to check the hardware clock time*

    *# hwclock*

    *Thu 18 Apr 2019 05:03:43 PM UTC  -0.578505 seconds*

* *Generally, your hardware clock time and system time will drift apart from each other(eg: When you shutdown your system), to synchronize it back*

    *# hwclock --systohc*

* *w, — systohc set the hardware clock from the current system time*

* *To keep our System time update we use some kind of time server on the internet eg: NTP or Chronyd*

* *Now let’s explore the other way, let try to manually set the system time to the wrong value*

    *# date*

    *Thu Apr 18 17:10:51 UTC 2019*

    *# date --set="20190418 16:10"*

    *Thu Apr 18 16:10:00 UTC 2019*

* *Then use hwclock command to set the system time from the hardware clock*

    *# hwclock --hctosys*

* *s, — hctosys set the system time from the hardware clock*

* *To simplify the time date management system in case of Linux, all the system that uses systemd now have timedatectl command*

    *# timedatectl*

    *Local time: Thu 2019-04-18 17:16:29 UTC*

    *Universal time: Thu 2019-04-18 17:16:29 UTC*

    *RTC time: Thu 2019-04-18 17:16:29 **<--hardware time***

    *Time zone: UTC (UTC, +0000)*

    *NTP enabled: yes*

    *NTP synchronized: no*

    *RTC in local TZ: no*

    *DST active: n/a*

* *To list time-zones via timedatectl command*

    *# timedatectl list-timezones*

* *Now to set it to a particular timezone*

    *# timedatectl set-timezone America/Los_Angeles*
> *Chronyd*

*In the case of RHEL7/Centos7 ntpd is replaced by chronyd as the default protocol. Chronyd is considered a more accurate time sync mechanism*

    *# yum -y install chrony*

* *Starting chronyd daemon*

    *# systemctl start chronyd*

* *Checking the status of service*

    *# systemctl status chronyd*

    ***●** chronyd.service - NTP client/server*

    *Loaded: loaded (/usr/lib/systemd/system/chronyd.service; enabled; vendor preset: enabled)*

    *Active: **active (running)** since Thu 2019-03-28 03:46:19 UTC; 3 weeks 0 days ago*

    *Docs: man:chronyd(8)*

    *man:chrony.conf(5)*

    *Main PID: 3213 (chronyd)*

    *CGroup: /system.slice/chronyd.service*

    *└─3213 /usr/sbin/chronyd*

    *Apr 18 17:15:00 ip-172-31-31-68.us-west-2.compute.internal chronyd[3213]: Source 69.164.198.192 replaced with 4.53.160.75*

    *Apr 18 17:18:14 ip-172-31-31-68.us-west-2.compute.internal chronyd[3213]: Can't synchronise: no majority*

    *Apr 18 17:26:12 ip-172-31-31-68.us-west-2.compute.internal chronyd[3213]: Selected source 74.117.214.3*

* *To make sure it’s enabled at boot time*

    *# systemctl enable chronyd*

* *Default configuration file*

    */etc/chrony.conf*

* *To find the source where your server is synchronized*

    *# chronyc sources*

    *210 Number of sources = 4*

    *MS Name/IP address         Stratum Poll Reach LastRx Last sample*

    *===============================================================================*

    *^- 4.53.160.75                   2   7   377     4   +107us[ +107us] +/-   65ms*

    *^- li216-154.members.linode>     2   6   377    29  +2578us[+2578us] +/-   96ms*

    ***^* ada.selinc.com                1   6   377    46   -822us[ -431us] +/- 9160us***

    *^- au.kashra.pictures            2   6   377    33  +1123us[+1123us] +/-  110ms*

* *To find the stats*

    *# chronyc tracking*

    *Reference ID    : 4A75D603 (ada.selinc.com)*

    *Stratum         : 2*

    *Ref time (UTC)  : Thu Apr 18 17:36:57 2019*

    *System time     : 0.000189348 seconds fast of NTP time*

    *Last offset     : +0.000390079 seconds*

    *RMS offset      : 0.053420763 seconds*

    *Frequency       : 3.979 ppm fast*

    *Residual freq   : -0.262 ppm*

    *Skew            : 9.018 ppm*

    *Root delay      : 0.016061351 seconds*

    *Root dispersion : 0.001351765 seconds*

    *Update interval : 64.8 seconds*

    *Leap status     : Normal*
> *Use Chrony to setup NTP*

    *# timedatectl set-ntp yes*
> *Advantage of using Chrony*

* *Chrony is particularly useful for the system that is frequently rebooted eg: Desktop as it provides faster synchronization which happens in minutes rather than in hours to minimize the time and frequency error*

*Reference*
[**chrony - Manual for version 2.3**
*Although most of the program is portable between Unix-like systems, there are parts that have to be tailored to each…*chrony.tuxfamily.org](https://chrony.tuxfamily.org/manual.html)

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
