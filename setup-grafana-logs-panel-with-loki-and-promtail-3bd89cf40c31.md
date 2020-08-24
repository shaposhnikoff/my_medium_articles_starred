
# Grafana Logs Panel with Loki and Promtail

We will look at the new Logs panel in Grafana, but we will first need to setup a data source that can read log files.

The requirements to follow and try the commands in this article exactly are to have Ubuntu 18.04, with Grafana installed. If you want to know more about installing Grafana, then I have an article here showing [How to Install and Start Grafana](https://medium.com/grafana-tutorials/install-and-start-grafana-7-fb93cedfe364).

In this article, we will:

1. Install Loki Binary and Start as a Service

1. Install Promtail Binary and Start as a Service

1. Configure Loki Data Source and Explore

1. Create some errors in the System Journal and see the data displayed in real time

![](https://cdn-images-1.medium.com/max/2732/1*7wh_lXu9c0-L6dyh3sDP5w.jpeg)

## Install Loki Binary and Start as a Service

To keep this as simple as possible, we will install the Loki binary as a service on our existing Grafana server. You can use another server if you prefer.

    cd /usr/local/bin

    sudo curl -fSL -o loki.gz “https://github.com/grafana/loki/releases/download/v1.4.1/loki-linux-amd64.gz"

    sudo gunzip loki.gz

And allow the execute permission on the Loki binary

    sudo chmod a+x loki

### Create the Loki Config

Now create the Loki config file.

    sudo nano config-loki.yml

And add this text

<iframe src="https://medium.com/media/a4a1ca36f41b1be339ca98e2ba5b9140" frameborder=0></iframe>

### Now Test Loki Process Works

You can now test Loki by running

    sudo loki -config.file /usr/local/bin/config-loki.yml

Open a browser and visit,

http://[Your-Server-Domain-or-IP]:3100/metrics

Now stop the Loki server by pressing **CTRL-C**. Note that it may take a minute for the process to stop.

### Configure Firewall

When your Loki server is running, it will be accessible remotely. If you only want localhost to be able to connect, then type

    iptables -A INPUT -p tcp -s localhost --dport 3100 -j ACCEPT
    iptables -A INPUT -p tcp --dport 3100 -j DROP
    iptables -L

### **Configure Loki as a Service**

Now we will configure Loki as a service so that we can keep it running in the background.

Create a file called *loki.service*

    sudo nano /etc/systemd/system/loki.service

Add the script and save

<iframe src="https://medium.com/media/c12813388c181b1806c785cc0132f4ba" frameborder=0></iframe>

Now start and check the service is running.

    sudo service loki start
    sudo service loki status

We can now leave the new Loki service running.

If you ever need to stop the new Loki service, then type

    sudo service loki stop
    sudo service loki status

## **Install Promtail Binary and Start as a Service**

Now we will create the Promtail service that will act as the collector for Loki.

    cd /usr/local/bin

    sudo curl -fSL -o promtail.gz "https://github.com/grafana/loki/releases/download/v1.4.1/promtail-linux-amd64.gz"

    sudo gunzip promtail.gz

And allow the execute permission on the Promtail binary

    sudo chmod a+x promtail

### **Create the Promtail Config**

Now we will create the Promtail config file.

    sudo nano config-promtail.yml

And add this script,

<iframe src="https://medium.com/media/028a11017ec21cf2112dd2e6170909ed" frameborder=0></iframe>

### **Test the Promtail Process Works**

You can now test Promtail by running

    sudo promtail -config.file /usr/local/bin/config-promtail.yml

Open a browser and visit,

http://[Your-Server-Domain-or-IP]:9080

After having a good look around to verify it works, stop the Promtail server by pressing **CTRL-C**.

### **Configure Firewall**

When your Promtail server is running, it will be accessible remotely. If you only want localhost to be able to connect, then type

    iptables -A INPUT -p tcp -s localhost — dport 9080 -j ACCEPT
    iptables -A INPUT -p tcp — dport 9080 -j DROP
    iptables -L

### **Configure Promtail as a Service**

Now we will configure Promtail as a service so that we can keep it running in the background.

Create a file called *promtail.service*

    sudo nano /etc/systemd/system/promtail.service

And add this script,

<iframe src="https://medium.com/media/93576cccecd52f1487376a6cc947e012" frameborder=0></iframe>

Now start and check the service is running.

    sudo service promtail start
    sudo service promtail status

We can now leave the new Promtail service running.

If you ever need to stop the new Promtail service, then type

    sudo service promtail stop
    sudo service promtail status

## Install Loki Data Source and Explore

Now we will install the Loki Data Source, point it to our new Loki Service, and Explore some of the log files on the server.

Create a new Data Source in the Grafana User Interface, and select
> Name : **Loki
**URL : [**http://127.0.0.1:3100](http://127.0.0.1:3100)**

**Note : **If you installed Loki on a different server than your local Grafana server, then the address will be different. eg, http://[your-loki-server-ip-address]:3100

Leave everything else default.

Then, visit the **Explore **menu option on the left.

Select **Loki **as the Data Source

And in the **Log labels** text area, try these examples one by one

    {job=”systemd-journal”}

    {job=”systemd-journal”} Info

    {job=”systemd-journal”} Error

    {job="systemd-journal"} Info|Error

    #Or For A Case Insensitive Search
    {job=”systemd-journal”} (?i)error

And experiment further with the other options shown in the **Log labels **drop down options.

You many not see any **errors **in particular when trying the above queries, so you can fake some by opening a SSH session on your server and create a few in the System Journal by using a command like below.

    echo 'abc123 this is a fake error def678' | systemd-cat

![](https://cdn-images-1.medium.com/max/2732/1*7wh_lXu9c0-L6dyh3sDP5w.jpeg)

## Taking This Further with Graph Annotations and Linking with the Log Panel

With another Data Source, that can be used for graphing, such as a Prometheus, InfluxDB, and more, we can create a new Dashboard with a Graph Panel, and a Log Panel, and then we can automate the display of Log Annotations by creating an Annotation Query. 
In the image below, if the word **error** appears in the Log Panel, it will show up with a red line next to it, and a red vertical line will also be drawn in the Graph Panel. You can press the red line in the Graph panel, and it will show the relevant log line in the Annotation panel, as well as in the Log panel.

![](https://cdn-images-1.medium.com/max/2732/1*uVE86YBWYC476o86shiyTQ.jpeg)

The image above comes from my Course on Grafana at [https://sbcode.net/grafana/](https://sbcode.net/grafana/) and demonstrates finding SNMP errors. 
The SNMP errors example in the image is significantly more work to set up than the example in this article. But if you feel adventurous and you want to study Grafana further, then my course will suit you since it has loads of examples also on Prometheus, MySQL, Zabbix, InfluxDB, Telegraf and SNMP, as well as more info on the Loki and Promtail Data Sources demonstrated in the article.
[**Grafana Tutorials**
*The tutorials in this documentation supplement my Grafana Course on Skillshare and Udemy.*sbcode.net](https://sbcode.net/grafana/)
