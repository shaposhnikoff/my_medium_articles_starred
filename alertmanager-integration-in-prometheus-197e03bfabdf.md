Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m65[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m45[39m }

# AlertManager Integration with Prometheus

One day I got a call from one of my friend and he said to me that he is facing difficulties while setting up AlertManager with Prometheus. Then, I observed that most of the people face such issues while establishing a connection between AlertManager and receiver such as E-mail, Slack etc.

From there, I got motivation for writing this blog so AlertManager setup with Prometheus will be a piece of cake for everyone.

If you are new to AlertManager I would suggest you go through with our [**Prometheus](https://medium.com/@abhishekbhardwaj510/prometheus-overview-and-setup-dc0ee20791fb) **blog.

## What Actually AlertManager is?

AlertManager is used to handle alerts for client applications(like **Prometheus**). It also takes care of alerts deduplicating, grouping and then routes them to different receivers such as E-mail, Slack, Pager Duty.

In this blog, we will only discuss on Slack and E-mail receivers.

AlertManager can be configured via command-line flags and configuration file. While command line flags configure system parameters for AlertManager, the configuration file defines inhibition rules, notification routing, and notification receivers.

## Architecture

Here is a basic architecture of Alertmanager with Prometheus.

![The architecture of Alertmanager with single Prometheus Server](https://cdn-images-1.medium.com/max/2800/0*Zp5xEhpasFynSMP1)*The architecture of Alertmanager with single Prometheus Server*

This is how Prometheus architecture works:-

* If you see in the above picture Prometheus is scraping the metrics from its client application(exporters).

* When the alert is generated then it pushes it to the AlertManager, later AlertManager validates the alerts groups on the basis of labels.

* and then forward it to the receivers like Email or Slack.

If you want to use a single AlertManager for multiple Prometheus server you can also do that. Then architecture will look like this:-

![The architecture of Alertmanager with multiple Prometheus Servers](https://cdn-images-1.medium.com/max/2000/0*vsKAKG-v4tbk1H6O)*The architecture of Alertmanager with multiple Prometheus Servers*

## **Installation**

Installation part of AlertManager is not a fancy thing, we just need simply to download the latest binary of AlertManager from [here](https://github.com/prometheus/alertmanager/releases).

    $ cd /opt/

    $ wget [https://github.com/prometheus/alertmanager/releases/download/v0.11.0/alertmanager-0.11.0.linux-amd64.tar.gz](https://github.com/prometheus/alertmanager/releases/download/v0.11.0/alertmanager-0.11.0.linux-amd64.tar.gz)

After downloading, let’s extract the files.

    $ tar -xvzf alertmanager-0.11.0.linux-amd64.tar.gz 

So we can start AlertManager from here as well but it is always a good practice to follow Linux directory structure.

    $ mv alertmanager-0.11.0.linux-amd64/alertmanager /usr/local/bin/

## **Configuration**

Once the tar file is extracted and binary file is placed at the right location then the configuration part will come. Although AlertManager extracted directory contains the configuration file as well, but it is not of our use. So we will create our own configuration. Let’s start by creating a directory for configuration.

    $ mkdir /etc/alertmanager/

Then the configuration file will take place.

    $ vim /etc/alertmanager/alertmanager.yml

The configuration file for **Slack **will look like this:-

<iframe src="https://medium.com/media/82e17bebb7797661dd65ea9b4a4f13e5" frameborder=0></iframe>

**Note:- You just have to replace the channel name and api_url of the Slack with your information.**

The configuration file for **E-mail **will look something like this:-

<iframe src="https://medium.com/media/97aee748db841d7f70f880ebbd48bb8e" frameborder=0></iframe>

In this configuration file, you need to update the sender and receiver mail details and the authorization password of the sender.

Once the configuration part is done we just have to create a storage directory where AlertManger will store its data.

    $ mkdir /var/lib/alertmanager

Then only last piece which will be remaining is my favorite part i.e creating service :-)

    $ vi /etc/systemd/system/alertmanager.service

The service file will look like this:-

    [Unit]
    Description=AlertManager Server Service
    Wants=network-online.target
    After=network-online.target
    
    [Service]
    User=root
    Group=root
    Type=Simple
    ExecStart=/usr/local/bin/alertmanager \
        --config.file /etc/alertmanager/alertmanager.yml \
        --storage.tsdb.path /var/lib/alertmanager
    
    [Install]
    WantedBy=multi-user.target

Then reload the daemon and start the alertmanager service.

    $ systemctl daemon-reload
    $ systemctl start alertmanager
    $ systemctl enable alertmanager

Now you are all set to fire up your **monitoring** and **alerting**. So just take a beer and relax until AlertManager notifies you for alerts. All the best!!!!
