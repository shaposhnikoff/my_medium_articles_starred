
# How Prometheus and the blackbox exporter makes monitoring microservice endpoints easy and free of…



Prometheus is a tool that can monitor the microservices and application metrics using the pull mechanism. It supports the Prom Query language for the searching of metrics and for creating custom metrics. It is a quick and easy way to set up endpoints monitoring at no cost. The blackbox exporter enables blackbox probing of endpoints over HTTP, HTTPS, DNS, TCP and ICMP.

In this blog, I will be showing how you can monitor the microservices endpoints using a Prometheus server and blackbox exporter, both as [containers](https://www.docker.com/what-docker). The latest version of Prometheus and other exporters can be downloaded [here](https://prometheus.io/download/) and an overview of the Prometheus monitoring tool and its use cases can be seen [here](https://prometheus.io/docs/introduction/overview/).

I have created the following ***Dockerfile*** using CentOS for my base images:

***Prometheus Dockerfile:***

    FROM centos

    ADD prometheus-1.7.1.linux-amd64.tar.gz /

    #Download prometheus and configure

    RUN cd /prometheus-* && \
        mv prometheus /bin/ && \
        mv promtool /bin/ && \
        mkdir /usr/share/prometheus/  && \
        mv console_libraries /usr/share/prometheus/console_libraries/ && \
        mv consoles/ /usr/share/prometheus/consoles/ && \
        mkdir /etc/prometheus && \
        cd  && \
        rm -rf /prometheus-*

    COPY prometheus.yml /etc/prometheus/prometheus.yml
    COPY prometheus.rules /etc/prometheus/prometheus.rules
    COPY alert.rules /etc/prometheus/alert.rules

    RUN ln -s /usr/share/prometheus/console_libraries /usr/share/prometheus/consoles/ /etc/prometheus/

    WORKDIR /usr/share/prometheus

    EXPOSE 9090

    ENTRYPOINT ["/bin/prometheus"]
    CMD  ["--config=/etc/prometheus/prometheus.yml", \
          "--storage.tsdb.path=/prometheus-storage", \
          "--web.external-url=http://localhost/"]

***Prometheus Config file(prometheus.yml):***

    # my global config
    global:
      scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.

    # A scrape configuration containing exactly one endpoint to scrape:
    # Here it's Prometheus itself.
    scrape_configs:

    - job_name: 'prometheus'

      # Override the global default and scrape targets from this job every 5 seconds.
      scrape_interval: 5s

      static_configs:
          - targets: ['localhost:9090']

Now build the Prometheus image and run it, using the following commands:

    docker build -t prometheus .
    docker run -d -p 9090:9090 --name prometheus prometheus

***Check the logs:***

    docker logs prometheus

![Check logs](https://cdn-images-1.medium.com/max/2000/1*J-4x3B_D_rmNOXmQ13rK9Q.png)*Check logs*

Connect to Prometheus via [http://localhost:9090](http://localhost:9090), as Prometheus is monitoring itself in this example, so it will scrape the number of metrics to monitor the Prometheus server.

![](https://cdn-images-1.medium.com/max/2000/0*4kdzqyL6v4eSgYru.)

Now, download the blackbox exporter and unzip it. Create a Docker file, build an image and run it.

***Blackbox Dockerfile:***

    FROM centos

    COPY blackbox_exporter  /bin/blackbox_exporter
    COPY blackbox.yml       /etc/blackbox_exporter/config.yml

    EXPOSE      9115
    ENTRYPOINT  [ "/bin/blackbox_exporter" ]
    CMD         [ "--config.file=/etc/blackbox_exporter/config.yml" ]

***Blackbox.yml:***

    modules:
      http_2xx:
        prober: http
        http:
      http_post_2xx:
        prober: http
        http:
          method: POST

Now build the blackbox image and run it:

    docker build -t blackbox .
    docker run -d -p 9115:9115 --name blackbox blackbox

Add the following lines to prometheus.yml file:

    - job_name: blackbox
            metrics_path: /probe
            params:
              module: [http_2xx]
            static_configs:
              - targets:
                - [https://www.robustperception.io/](https://www.robustperception.io/)
                - [http://prometheus.io/blog](http://prometheus.io/blog)
                - [http://yourdomain/usage-api/health](http://yourdomain/usage-api/health)
                - [http://yourdomain/google-apm/health](http://yourdomain/google-apm/health)
                - [https://google.com](https://google.com)            
                - [https://www.telegraph.co.uk](https://www.telegraph.co.uk)
                
            relabel_configs:
              - source_labels: [__address__]
                target_label: __param_target
              - source_labels: [__param_target]
                target_label: instance
              - target_label: __address__
                replacement: localhost:9115 # Blackbox exporter.

Go to the Prometheus server [http://localhost:9090](http://localhost:9090) — status — targets, and you will see all the endpoint targets added in here. Also, blackbox scrape the following probe metrics to monitor the endpoints:

![](https://cdn-images-1.medium.com/max/2000/0*MSix9-uSSvMYXZX5.)

![](https://cdn-images-1.medium.com/max/3200/0*_ibYGGjZUKVX-YFa.)

The above endpoints monitoring can be visualized in a better way, by using the tool Grafana.

Pull and run a Grafana image from the Docker registry:

    docker run -d -p 3000:3000 — name grafana grafana/grafana

Connect to [http://0.0.0.0:3000/login](http://0.0.0.0:3000/login) and login using credentials as admin/admin, click on add datasource and select type as Prometheus and add the url link of [http://localhost:9090](http://localhost:9090) in this case:

![](https://cdn-images-1.medium.com/max/2748/0*hr3f8mtz4el5Emg8.)

Create a new dashboard — Add row — Singlestat — right-click to edit and in the metrics tab add the query, for example:

probe_success{instance=”yourdomain/servicename/health”}

![](https://cdn-images-1.medium.com/max/3200/0*rqhrr7d_N3nEpo2K.)

In the ‘Value Mapping’ tab set the value 1 to UP and 0 to DOWN. In the options tab, set the threshold 0,1 and choose the background color. If the endpoint is reachable, probe_status is 1 and display the message as UP and if it is DOWN then probe_status returns 0 and the endpoint is unreachable.

![](https://cdn-images-1.medium.com/max/3200/0*sIFWsQ9-X0Oovjx3.)

By following the above steps, any number of microservices endpoints can be monitored.
