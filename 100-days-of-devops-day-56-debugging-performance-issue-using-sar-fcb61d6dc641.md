
# 100 Days of DevOps — Day 56-Debugging Performance Issue using SAR

Welcome to Day 56 of 100 Days of DevOps, Focus for today is Debugging Performance Issue using SAR
> What is SAR?

*SAR collect, report, or save system activity information. It’s a utility used to collect and report system activity. It collects data relating to most core system functions and writes those metrics to binary data files.*
> Installing SAR

    *# yum -y install sysstat*

* *To enable SAR on boot*

    *# systemctl enable sysstat (Centos7/RHEL7)*
> How SAR works

* *When we install a sysstat package it places a file in /etc/cron.d/sysstat*

    *# cat /etc/cron.d/sysstat*

    *# Run system activity accounting tool every 10 minutes*

    **/10 * * * * root /usr/lib64/sa/sa1 1 1*

    *# 0 * * * * root /usr/lib64/sa/sa1 600 6 &*

    *# Generate a daily summary of process accounting at 23:53*

    *53 23 * * * root /usr/lib64/sa/sa2 -A*

*This file setup two cron jobs*

* *1 job to record statistics every 10 minutes.*

* *2 job to write the binary sa\#\# file to a text sar\#\# file once a day (typically right before midnight).*
> *Additional config can be placed in a configuration file(/etc/sysconfig/sysstat)*

    *# cat /etc/sysconfig/sysstat*

    *# sysstat-10.1.5 configuration file.*

    *# How long to keep log files (in days).*

    *# If value is greater than 28, then log files are kept in*

    *# multiple directories, one for each month.*

    *HISTORY=28*

    *# Compress (using gzip or bzip2) sa and sar files older than (in days):*

    *COMPRESSAFTER=31*

    *# Parameters for the system activity data collector (see sadc manual page)*

    *# which are used for the generation of log files.*

    *SADC_OPTIONS="-S DISK"*

    *# Compression program to use.*

    *ZIP="bzip2"*
> How SAR is useful

* *Sar data is useful in pinpointing the system resource (networking, memory, IO, CPU, etc.) that is causing a performance issue.*

* *Sar data contains several useful sections for determining how the system was performing at a given time. By default, this is configured to run at ten minute intervals. Information on any of the descriptors used below (such as runq-sz, kbmemused, etc.) can be obtained by searching for these names after executing **man sar**.*
> Debugging High Load Average

* *Load — information regarding how many tasks are currently on the system.*

    *# sar -q 1 4*

    *00:00:01      runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15
    [...]
    04:20:01            1      2751      0.08      0.34      0.42
    04:30:01            0       955      2.18      1.37      1.01
    04:40:03           10      1645      8.18      6.58      3.83
    04:50:14           25      1704    170.37    113.04     55.24
    Average:            2      2605      6.67      4.64      2.51*

* *From the above information, we can see that the load had a massive spike around 4:50. Typically a system’s load should remain at 70% of the number of cores or lower. If the system’s load is consistently above this amount there may be performance degradation, and if the load ever rises above the number of cores there will be a significant slowdown.*
> Debugging High Memory Utilization

* *Memory — information regarding memory and swap usage.*

    *# sar -r 1 3
    00:00:01    kbmemfree kbmemused  %memused kbbuffers  kbcached kbswpfree kbswpused  %swpused  kbswpcad
    [...]
    04:30:01     62716272 201531148     76.27     15952   1556572   2048236        12      0.00         4
    04:40:03       191904 264055516     99.93      2692     28908         0   2048248    100.00      8496
    04:50:14       184100 264063320     99.93      1388     10600         0   2048248    100.00         0
    Average:      4415719 259831701     98.33   1357749  20307185   1906978    141270      6.90       297*

* *Linux likes to use memory at around 99%; however, if the system is actively swapping then the system is most likely experiencing memory pressure. Considering that we see all of the swap used above within 10 minutes the system containing this data was experiencing memory issues during this time.*
> Debugging High I/O Utilization

* *IO — information pertaining to the number of disk accesses.*

    *# sar -b 1 3
    00:00:01          tps      rtps      wtps   bread/s   bwrtn/s
    [...]
    04:30:01        67.95     51.84     16.12   4303.82   4664.14
    04:40:03       564.60    227.07    337.52  34338.84  87719.02
    04:50:14        51.05     40.25     10.80   1326.32    245.06
    Average:        31.12     11.00     20.12   1383.64   3346.65*

* *The number of disk reads and writes will vary based on the underlying hardware; however, we can take a look at what is considered ‘normal’ for this system by examining the data over a period of time, and then look for spikes. We can see a large spike at 4:40 where the number of reads and writes increases dramatically. Note that shortly after these go back down, indicating that this massive burst was resolved.*
> Debugging High CPU Utilization

* *CPU — information regarding where each of the system’s cycles are spent.*

    *# sar -u 1 10*

    *00:00:01          CPU     %user     %nice   %system   %iowait    %steal     %idle
    [...]
    04:40:03          all     10.45      0.00      1.67      0.89      0.00     86.99
    04:50:14          all      0.19      0.00     62.06      1.98      0.00     35.78*

* *Spending time in %user is expected behavior, as this is where all non-system tasks are accounted for. If cycles are actively being spent in %system then much of the execution time is being spent in lower-level code. If %iowait is high then it indicates processes are actively waiting due to disk accesses being a bottleneck on the system.*

* *In addition, sar data may be viewed graphically by downloading and using the ‘kSar’ tool. This tool is not provided from RedHat; however, it may be useful in pinpointing problematic times on the system. Documentation on this tool is at the following link*
[**ksar : a sar grapher**
*Download ksar : a sar grapher for free. ksar is a sar graphing tool that can graph for now linux,mac and solaris sar…*sourceforge.net](https://sourceforge.net/projects/ksar/)

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
