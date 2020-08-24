Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m108[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m103[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m450[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m107[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m82[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m19[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m78[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m75[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m115[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m292[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m59[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m249[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m39[39m }

# Monitoring Linux Processes using Prometheus and Grafana



Whether you are a Linux system administrator or a DevOps engineer, you spend a lot of time **tracking performance metrics on your servers**.

You may sometimes have **instances that are running very slow** without having any real clues of what the issues might be.

You may have **unresponsive instances** that might block you from running remote commands such as top or htop on them.

You may have a **simple bottleneck** on your server, but you cannot identify it in a simple and quick way.
> *What if we had a complete dashboard that would help us tracking general performance as well as individual processes?*
> *See it live here : [http://grafana.devconnected.com/d/nZMDMoiZk/grafana-top?orgId=1&refresh=5s](http://grafana.devconnected.com/d/nZMDMoiZk/grafana-top?orgId=1&refresh=5s)*

![](https://cdn-images-1.medium.com/max/2000/1*3I_uiXje7P2CK0zBz9ySEQ.png)

The goal of this tutorial is to create a **complete monitoring dashboard for Linux sysadmins**.

This dashboard will showcase different panels that are entirely customizable and scalable to multiple instances for distributed architectures.

## What You Will Learn

Before jumping right into this technical journey, let’s have a quick look at everything that you are going to learn by reading this article:

* Understanding current state-of-the-art ways to monitor process performance on Unix systems;

* Learn how to install the latest versions of **Prometheus v2.9.2**, **Pushgateway v0.8.0** and **Grafana v6.2**;

* Build a simple **bash script** that exports metrics to Pushgateway;

* Build a **complete Grafana dashboard** including the latest panels available such as the ‘Gauge’ and the ‘Bar Gauge’;

* Bonus : implementing **ad-hoc filters** to track individual processes or instances.

Now that we have an overview of everything that we are going to learn, and without further due, let’s have an introduction on what’s currently existing for Unix systems.

## Unix Process Monitoring Basics

When it comes to process monitoring for Unix systems, you have multiple options.

The most popular one is probably **‘top’.**

This command is widely used among sysadmins and is probably the first command run when a performance bottleneck is detected on a system (if you can access it of course!)

![](https://cdn-images-1.medium.com/max/2000/1*wuCYYK_fnFuwOSXIDuy_zA.png)

The top command is already pretty readable, but there is a command that makes everything even more readable than that: **htop**.

Htop provides the same set of functionalities (CPU, memory, uptime..) as top but in a colorful and pleasant way.

Htop also provides **gauges** that reflects current system usage.

![](https://cdn-images-1.medium.com/max/2020/1*l6YsSD0V4MhtfM94fjQJTA.png)
> *Knowing that those two commands exist, why would we want to build yet another way to monitor processes?*

The main reason would be **system availability**: in case of a system overload, you may have no physical or remote access to your instance.

By externalizing process monitoring, you can analyze what’s causing the outage without accessing the machine.

Another reason is that **processes get created and killed all the time**, often by the kernel itself.

In this case, running a top command would give you zero information as it would be too late for you to catch who’s causing performance issues on your system.

You would have to dig into kernel logs to see what has been killed.

With a monitoring dashboard, you can simply go back in time and see which process was causing the issue.

Now that you know why we want to build this dashboard, let’s have a look at the **architecture** put in place in order to build it.

![](https://cdn-images-1.medium.com/max/2000/1*QpR6Z1jSl-T599zRLkYv6w.png)

## Detailing Our Monitoring Architecture

Before having a look at the architecture that we are going to use, we want to use a solution that is:

* **Resource cheap** : i.e not consuming many resources on our host;

* **Simple to put in place** : a solution that doesn’t require a lot of time to instantiate;

* **Scalable** : if we were to monitor another host, we can do it quickly and efficiently.

Those are the points we will keep in mind throughout this tutorial.

The detailed architecture we are going to use today is this one:

![](https://cdn-images-1.medium.com/max/2790/1*YPKsl3IrBS4o-MctXeo1ag.png)

Our architecture makes use of four different components:

* A bash script used to send periodically metrics to the Pushgateway;

* **Pushgateway**: a metrics cache used by individual scripts as a target;

* **Prometheus:**, that instantiates a time series database used to store metrics. Prometheus will scrape Pushgateway as a target in order to retrieve and store metrics;

* **Grafana**: a dashboard monitoring tool that retrieves data from Prometheus via PromQL queries and plot them.

For those who are quite familiar with Prometheus, you already know that Prometheus scraps metrics exposed by HTTP instances and stores them.

In our case, the bash script has a very tiny lifespan and it doesn’t expose any HTTP instance for Prometheus.

This is why we have to use the Pushgateway ; designed for **short-lived jobs**, Pushgateway will cache metrics received from the script and expose them to Prometheus.

![](https://cdn-images-1.medium.com/max/2752/1*5gFsYfPwvXOmN9zOAOFiWg.png)

![](https://cdn-images-1.medium.com/max/2000/1*YmoBcdjaUa07cOLdbtgusA.png)

## Installing The Different Tools

Now that you have a better idea of what’s going on in our application, let’s install the different tools needed.

## a — Installing Pushgateway

In order to install **Pushgateway**, run a simple wget command to get the latest binaries available.

    wget [https://github.com/prometheus/pushgateway/releases/download/v0.8.0/pushgateway-0.8.0.linux-amd64.tar.gz](https://github.com/prometheus/pushgateway/releases/download/v0.8.0/pushgateway-0.8.0.linux-amd64.tar.gz)

Now that you have the archive, extract it, and run the executable available in the pushgateway folder.

    > tar xvzf pushgateway-0.8.0.linux-amd64.tar.gz 
    > cd pushgateway-0.8.0.linux-amd64/ 
    > ./pushgateway &

As a result, your **Pushgateway should start as background process**.

    me@schkn-ubuntu:~/softs/pushgateway/pushgateway-0.8.0.linux-amd64$ ./pushgateway & 
    [1] 22806 
    me@schkn-ubuntu:~/softs/pushgateway/pushgateway-0.8.0.linux-amd64$ INFO[0000] Starting pushgateway (version=0.8.0, branch=HEAD, revision=d90bf3239c5ca08d72ccc9e2e2ff3a62b99a122e) source="main.go:65"INFO[0000] Build context (go=go1.11.8, user=root@00855c3ed64f, date=20190413-11:29:19) source="main.go:66"INFO[0000] Listening on :9091. source="main.go:108"

Nice!

From there, **Pushgateway is listening to incoming metrics on port 9091**.

## b — Installing Prometheus

As described in the ‘Getting Started’ section of Prometheus’s website, head over to [https://prometheus.io/download/](https://prometheus.io/download/) and run a simple wget command in order to get the Prometheus archive for your OS.

    wget https://github.com/prometheus/prometheus/releases/download/v2.9.2/prometheus-2.9.2.linux -amd64.tar.gz

Now that you have the archive, extract it, and navigate into the main folder:

    > tar xvzf prometheus-2.9.2.linux-amd64.tar.gz 
    > cd prometheus-2.9.2.linux-amd64/

As stated before, **Prometheus scraps ‘targets’ periodically** to gather metrics from them. Targets (Pushgateway in our case) need to be configured via Prometheus’s configuration file.

    > vi prometheus.yml

In the ‘global’ section, modify the *‘scrape_interval’* property down to one second.

    global:
      scrape_interval:     1s # Set the scrape interval to every 1 second.

In the *‘scrape_configs’* section, add an entry to the targets property under the static_configs section.

    static_configs:
                - targets: ['localhost:9090', 'localhost:9091']

Exit vi, and finally run the prometheus executable in the folder.

Prometheus should start when launching the final prometheus command. To ensure that everything went correctly, you can head over to [http://localhost:9090/graph](http://localhost:9090/graph).

If you have access to Prometheus’s web console, it means that everything went just fine.

You can also verify that Pushgateway is correctly configured as a target in *‘Status’ > ‘Targets’* in the Web UI.

![](https://cdn-images-1.medium.com/max/2000/1*Bx13Dn8k-m3u0Qvd0F1GhA.png)

## c — Installing Grafana

Last not but least, we are going to install **Grafana v6.2**. Head over to [https://grafana.com/grafana/download/beta](https://grafana.com/grafana/download/beta).

As done before, run a simple wget command to get it.

    > wget https://dl.grafana.com/oss/release/grafana_6.2.0-beta1_amd64.deb> sudo dpkg -i grafana_6.2.0-beta1_amd64.deb

Now that you have extracted the deb file, grafana should run as a service on your instance.

You can verify it by running the following command:

    > sudo systemctl status grafana-server
    ● grafana-server.service - Grafana instance
       Loaded: loaded (/usr/lib/systemd/system/grafana-server.service; disabled; vendor preset: enabled)
       Active: active (running) since Thu 2019-05-09 10:44:49 UTC; 5 days ago
         Docs: http://docs.grafana.org

You can also check [http://localhost:3000](http://localhost:3000) that is the default address for Grafana Web UI.

Now that you have Grafana on your instance, we have to configure **Prometheus as a datasource**.

You can configure your datasource this way :

![](https://cdn-images-1.medium.com/max/2000/1*YGcj6edL-T_UUjvk9KwbRw.png)

**That’s it!**

Click on ‘Save and Test’ and make sure that your datasource is working properly.

![](https://cdn-images-1.medium.com/max/2000/1*t3PqnTLVM8765I88TtthUw.png)

## Building a bash script to retrieve metrics

Your next task is to build a simple bash script that retrieve metrics such as the CPU usage and the memory usage for individual processes.

Your script can be defined as a cron task that will run every second later on.

To perform this task, you have multiple candidates.

You could run top commands every second, parse it using sed and send the metrics to Pushgateway.

The hard part with top is that it runs on multiple iterations, providing a metrics average over time. This is not really what we are looking for.

Instead, we are going to use the **ps** command and more precisely the **ps aux** command.

![](https://cdn-images-1.medium.com/max/2000/1*FhdHTespbnn3ZQTk2yxLkA.png)

This command exposes individual **CPU and memory usages** as well as the exact command behind it.

This is exactly what we are looking for.

But before going any further, let’s have a look at what **Pushgateway is expecting as an input.**

Pushgateway, pretty much like Prometheus, works with **key value pairs**: the key describes the **metric monitored** and the value is self explanatory.

Here are some examples:

![](https://cdn-images-1.medium.com/max/2000/1*ny-AQp6fGdTQWPGHqbxRCw.png)

As you can tell, the first form simply describes the CPU usage, but the second one describes the CPU usage for the java process.

**Adding labels is a way of specifying what your metric describes more precisely.**

Now that we have this information, we can build our final script.

As a reminder, our script will perform a ps aux command, parse the result, transform it and send it to the Pushgateway via the syntax we described before.

Create a script file, give it some rights and navigate to it.

    > touch better-top 
    > chmod u+x better-top 
    > vi better-top

Here’s the script:

*If you want the same script for memory usage, simply change the ‘cpu_usage’ label to ‘memory_usage’ and the $3z to $4z*

    #!/bin/bash
    z=$(ps aux)
    while read -r z
    do
       var=$var$(awk '{print "cpu_usage{process=\""$11"\", pid=\""$2"\"}", $3z}');
    done <<< "$z"
    curl -X POST -H  "Content-Type: text/plain" --data "$var
    " http://localhost:9091/metrics/job/top/instance/machine

So what does this script do?

First, it performs the **ps aux** command we described before.

As you can tell, this script gathers all metrics for our processes but it only runs one iteration.

For now, we are simply going to execute it every one second using a sleep command.

Later on, you are free to create a service to execute it every second with a timer (at least with systemd).
> *Interested in systemd? [I made a complete tutorial about monitoring them with Chronograf](http://devconnected.com/monitoring-systemd-services-in-realtime-with-chronograf/)*

    > while sleep 1; do ./better-top; done;

Now that our metrics are sent to the Pushgateway, let’s see if we can explore them in **Prometheus Web Console.**

Head over to [http://localhost:9090](http://localhost:9090). In the ‘Expression’ field, simply type ‘ **cpu_usage**’. You should now see all metrics in your browser.

Congratulations! Your CPU metrics are now stored in Prometheus TSDB.

![](https://cdn-images-1.medium.com/max/2206/1*4_Q1cqHQvlhBzSyhzvU0PA.png)

![](https://cdn-images-1.medium.com/max/2000/1*ROMqvsQNPFxMmcN148UN6g.png)

## Building An Awesome Dashboard With Grafana

Now that our metrics are stored in Prometheus, we simply have to build a **Grafana dashboard** in order to visualize them.

For your comfort, I have annotated the final dashboard with numbers from 1 to 4.

They will match the different subsections of this chapter. If you’re only interested in a certain panel, head over directly to the corresponding subsection.

![](https://cdn-images-1.medium.com/max/3840/1*rI7JD34Iu-sLj95Z_WlSsQ.png)

## 1 — Building Rounded Gauges

Here’s a closer view of what rounded gauges in our panel.

![](https://cdn-images-1.medium.com/max/2000/1*HdQrSsL9ItWBaC7Pz9aSVQ.png)

For now, we are going to focus on the CPU usage of our processes as it can be easily mirrored for memory usage.

With those panels, we are going to track two metrics : **the current CPU usage of all our processes and the average CPU usage.**

In order to retrieve those metrics, we are going to perform PromQL queries on our Prometheus instance.

So.. what’s PromQL?

**PromQL is the query language designed for Prometheus**.

Similarly what you found find on InfluxDB instances with InfluxQL (or IFQL), PromQL queries can aggregate data using functions such as the sum, the average and the standard deviation.

The syntax is very easy to use as we are going to demonstrate it with our panels.

### a — Retrieving the current overall CPU usage

In order to retrieve the current overall CPU usage, we are going to use **PromQL sum function.**

At a given moment in time, our overall CPU usage is simply the sum of individual usages.

Here’s the cheat sheet:

![](https://cdn-images-1.medium.com/max/3706/1*aE6UqogBXx4ZlFoHKrAduA.png)

### b — Retrieving the average CPU usage

Not much work to do for average CPU usage, you are simply going to use the **avg function of PromQL**. You can find the cheat sheet below.

![](https://cdn-images-1.medium.com/max/3706/1*ArsvzssIhGvZbvAR1NnpNw.png)

## 2 — Building Horizontal Gauges

Horizontal gauges are one of the latest additions of Grafana v6.2.

Our goal with this panel is to expose the top 10 most consuming processes of our system.

To do so, we are going to use the** topk function** that retrieves the top k elements for a metric.

Similarly to what we did before, we are going to define thresholds in order to be informed when a process is consuming too many resources.

![](https://cdn-images-1.medium.com/max/3718/1*Me0WmLwQxA8Sf4Lct-O5FQ.png)

## 3 — Building Vertical Gauges

Vertical gauges are very similar to horizontal gauges, we only need to tweak the **orientation parameter** in the visualization panel of Grafana.

Also, we are going to monitor our memory usage with this panel so the query is slightly different.

![](https://cdn-images-1.medium.com/max/3720/1*JxQbRiJPlN3eIz27PfpVWg.png)

**Awesome**! We made great progress so far, one panel to go.

## 4 — Building Line Graphs

Line graphs have been in Grafana for a long time and this is the panel that we are going to use to have an historical view of how our processes have evolved over time.

This graph can be particularly handy when :

* You had some outage in the past and would like to investigate which processes were active at the time.

* A certain process died but you want to have a view of its behaviour right before it happened

When it comes to troubleshooting exploration, it would honestly need a whole article (especially with the recent Grafana Loki addition).

Okay, here’s the final **cheat sheet**!

![](https://cdn-images-1.medium.com/max/3236/1*-YYXi5y3ZKBLE7LqtXq0Yw.png)

From there, **we have all the panels that we need for our final dashboard.**

You can arrange them the way you want or simply take some inspiration from the one we built.

![](https://cdn-images-1.medium.com/max/2000/1*aVWCFXs19uQ30718RRu45g.png)

## Bonus : explore data using ad hoc filters

Real time data is interesting to see — but the real value comes when you are able **to explore your data.**

In this bonus section, we are not going to use the ‘Explore’ function (maybe in another article?), **we are going to use ad hoc filters.**

With Grafana, you can define **variables associated to a graph**. You have many different options for variables : you can for example define a variable for your data source that would allow to dynamically switch the datasource in a query.

In our case, we are going to use simple** ad hoc filters** to explore our data.

![](https://cdn-images-1.medium.com/max/2000/1*m_BDIAshz80NkVOlwQ0xaQ.png)

From there, simply click on **‘Variables’** in the left menu, then click on **‘New’.**

![](https://cdn-images-1.medium.com/max/2000/1*lMe76S8SDRBmqnuFTN_nRQ.png)

As stated, ad hoc filters are automatically applied to dashboards that target the Prometheus datasource. Back to our dashboard.

Take a look at the top left corner of the dashboard.

![](https://cdn-images-1.medium.com/max/2000/1*Xvf7niEUtWYBvoH0iZz-RQ.png)

Now let’s say that you want the performance of a certain process in your system : let’s take Prometheus itself for example.

Simply navigate into the filters and see the dashboard updating accordingly.

![](https://cdn-images-1.medium.com/max/3718/1*cqL7TEh8dU6k4k-UWKii8Q.png)

**Now you have a direct look at how Prometheus is behaving on your instance.**

You could even go back in time and see how the process behaved, independently from its pid!

![](https://cdn-images-1.medium.com/max/2000/1*gVMbXm4J0VZpUQQRonw1_Q.png)

## A quick word to conclude

From this tutorial, you now have a better understanding of what **Prometheus and Grafana** have to offer.

You know have a **complete monitoring dashboard** for one instance, but there is really a small step to make it scale and monitor an entire cluster of Unix instances.

**DevOps monitoring** is definitely an interesting subject — but it can turn into a nightmare if you do it wrong.

This is exactly why we write those articles and build those dashboards : to help you reach the maximum efficiency of what those tools have to offer.

We believe that great tech can be enhanced with useful showcases.

I made similar articles, so if you enjoyed this one, make sure to read the others :

Until then, have fun, as always.

<iframe src="https://medium.com/media/d10eb3b3cac810e9b7a2a2c42b2df9aa" frameborder=0></iframe>

*Originally published at [http://devconnected.com](http://devconnected.com/monitoring-linux-processes-using-prometheus-and-grafana/) on May 18, 2019.*
