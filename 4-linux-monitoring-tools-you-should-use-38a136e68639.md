Unknown markup type 10 { type: [33m10[39m, start: [33m60[39m, end: [33m67[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m71[39m, end: [33m77[39m }

# 4 Linux Monitoring Tools You Should Use

See what is really going on underneath the hood with these tools

![Photo by [Lukas Blazek](https://unsplash.com/@goumbik?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/graph?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText).](https://cdn-images-1.medium.com/max/12288/1*wZAZ1OQ0YrzGwiGEGm98qg.jpeg)*Photo by [Lukas Blazek](https://unsplash.com/@goumbik?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/graph?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText).*

If you don’t know what is happening within the underlying operating system of your servers, you’re flying blind. You can make educated guesses, but when it comes down to it, you need raw data — and you need it quickly.

In order to make informed decisions on changes to your system, you need to be able to pinpoint issues and examine clear data.

Have you ever noticed a machine behaving sluggishly, but CPU usage doesn’t seem very high? Have you discovered extremely high network utilization on a system but don’t know the offending process?

These tools can help. The best part is most of them can be used without a GUI and are installed easily on most servers.

Below are some of the best monitoring tools for Linux I use on a daily basis to help me diagnose issues quickly and accurately.

## 1. iotop

![Who is using I/O and how much in iotop.](https://cdn-images-1.medium.com/max/2232/1*eZ4yegDrlxZipvYJf0GlNA.png)*Who is using I/O and how much in iotop.*

If you’ve ever wondered how much of your precious I/O is being used up by certain processes, [iotop](https://linux.die.net/man/1/iotop) to the rescue! I’ve used this countless times to watch disk-hungry processes chew through IOPS.

You just can’t get the raw numbers from a traditional tool like top. When you use iotop, you get exactly what it sounds like: an I/O-focused look at processes and system usage.

You can use this for a multitude of purposes, but the most critical one is to see disk usage and flag potential bottlenecks. Coupling this tool with others like top or htop provides you with a bigger picture of what’s happening on your machine.

One thing I would not suggest using iotop for is benchmarking speed. Although you do get a clear picture of how much I/O each process is using, it is more geared toward live monitoring and not repeated testing. If you’re interested in benchmarking I/O, I would check out a tool called [fio](https://fio.readthedocs.io/en/latest/fio_doc.html).

## 2. htop

![All the crucial info in a single pane of glass.](https://cdn-images-1.medium.com/max/3388/1*mLIXkyNbmxd9Gp4wLUTu4w.png)*All the crucial info in a single pane of glass.*

This is one of my personal favorites. [This tool](https://hisham.hm/htop/) is way more visually appealing than regular top and has a great default color scheme. Right away, you have a clear understanding of what is going on with the system.

You can see how many cores the machine has and what their utilization is on a clear horizontal bar graph. You get easy memory usage stats in the same fashion, and you also have the classic top process list at the bottom.

The main reason I love htop so much is that it gives me the info I care about quickly. I want to see what my per-core CPU usage and memory utilization are like graphically — not a boring percentage.

If you think something is maxing out all the cores on your system and you open htop, then you’ll be seeing quite a lot of red. It’s fast *and* concise.

## 3. IPTraf

![Sending pings to Google while monitoring in iptraf-ng.](https://cdn-images-1.medium.com/max/3408/1*Ik1rkiwxOocaoyqzaW521A.png)*Sending pings to Google while monitoring in iptraf-ng.*

This is [an incredibly useful tool](http://iptraf.seul.org/) for diagnosing network issues. With this tool, you can monitor all of the network traffic transiting your system. You can set up filters for specific interfaces or traffic types (like a specific TCP port). This is very similar to [Wireshark,](https://www.wireshark.org/) except it is much more lightweight and can also be run without a GUI.

You can do neat things like get a statistical breakdown of the traffic by overall packet size:

![Statistical breakdown by packet size in iptraf-ng.](https://cdn-images-1.medium.com/max/2000/1*nxLez03HtE_tjF4vTNUaDw.png)*Statistical breakdown by packet size in iptraf-ng.*

You could do something similar with command-line tools like tcpdump or tshark, but this tool is menu-driven and much easier to navigate. If you’re into interactively filtering and viewing your traffic, then IPTraf is a game-changer.

## 4. Monit

![The result of running monit status with a few monitored processes.](https://cdn-images-1.medium.com/max/2128/1*mZaicnnznIou8-UcqFljvA.png)*The result of running monit status with a few monitored processes.*

This is one of the most versatile and robust monitoring tools you can leverage on Linux. [Monit](https://mmonit.com/) has been around for many years and can be configured in numerous ways to support different kinds of threshold and performance alerting.

Monit allows you to specify processes, ports, files, and more to monitor on a Linux system. You can set up dynamic alerting patterns with sophisticated back-off timers and messages.

One example might be that you want to monitor a specific process to ensure it is running. If the process crashes once, simply restart it. If it begins crash-looping multiple times, don’t restart the process and send an alert instead. Process monitoring like this can easily be achieved in a few lines of configuration with Monit.

Monit even has a nice lightweight web interface for the daemon to show you what’s going on at a glance:

![The Monit web interface. Source: mmonit.com.](https://cdn-images-1.medium.com/max/2000/1*H8HN2Rq5bIHrhoeGMtw8Vw.png)*The Monit web interface. Source: mmonit.com.*

Whether you manage a single server or a fleet, Monit is one of the easiest, most efficient, and cost-effective (free!) methods to keep an eye on the status of things.

## Conclusion

Thank you for taking the time to read. I hope you enjoyed learning about some of my favorite Linux monitoring tools and why they make such a difference when it comes to analyzing misbehaving systems.
