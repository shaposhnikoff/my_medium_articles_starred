Unknown markup type 10 { type: [33m10[39m, start: [33m165[39m, end: [33m188[39m }

# Monitoring Traefik With Grafana

Monitoring Traefik With Grafana

### Let us start monitoring the Incoming requests and responses on the reverse proxy with Grafana

![](https://cdn-images-1.medium.com/max/4980/1*tZ7Ta4hwOzUbvyg2o7UEFQ.png)

In my previous post, I demonstrated how to deploy Traefik to Docker Swarm cluster and how to use some of the most important features that Traefik supports out of the box (If you missed the post, it is linked below).
[**Traefik 2.1 as a Reverse Proxy**
*Dynamically create and expose routing rules for your services without restarting or redeploying the reverse proxy.*medium.com](https://medium.com/@wshihadeh/traefik-2-1-as-a-reverse-proxy-c9e274da0a32)

One of the features that I demonstrated is the Prometheus metrics for monitoring Traefik. These metrics can be used not only to monitor the health status of the Traefik services but also to monitor the routes and the incoming requests responses.

In this post, I will present how to deploy Prometheus and Grafana to Docker swarm clusters to Monitor Traefik reverse proxy and how to build a Grafana dashboard to monitor Traefik service and the hosted services.

Deploying Prometheus and Grafana

To be able to visualize Traefik metrics we need to scrape the metrics from the exposed endpoint and store the results in a database. In addition, we need to deploy Grafana to be able to build the dashboards for Traefik.

![](https://cdn-images-1.medium.com/max/2000/1*LTLZGuLuyzuzBdodOIZ_8A.png)

Before we jump into deploying the service, Iw ould like to briefly describe the applications or services needed for building the monitoring dashboards

→ [Grafana](https://grafana.com/): Grafana is open-source analytics and interactive visualization software that can be used to build visualizations or diagrams based on several data sources such as Prometheus or MySQL. In addition, Grafana provides solutions for triggering alters.

→ [Prometheus](https://prometheus.io/docs/introduction/overview/): is an open-source system monitoring and alerting toolkit. [Prometheus](https://prometheus.io/docs/introduction/overview/) can be used to collect metrics from several targets and store the collected metrics in a time series database. In addition, it provides tools to query the collected metrics and allows generating alerts based on the collected metrics.

→ Traefik: is an open-source router or reverse-proxy.

The below docker-compose file can be used to deploy all the services needed for demonstrating building the monitoring dashboard.

<iframe src="https://medium.com/media/f0e81575e8e50b21af9bc3f6629a90e2" frameborder=0></iframe>

For demo purposes and to keep the deployment as simple as possible, I used the default values for most of the configurations for both Grafana and Prometheus. For Grafana, I mainly configured the admin username and password (These credentials will be used to login to Grafana) and I used the official docker image.

On the other hand, I used a custom docker image for Prometheus to include all the custom configurations needed for defining Prometheus jobs or targets (The endpoints where Prometheus will be scraping metrics from).

<iframe src="https://medium.com/media/16ff3c31b7ab86cc819f249d21d55660" frameborder=0></iframe>

Deploying the above stack can be done using the following command s

    # add the belwo line to /etc/hosts
    127.0.0.1 prometheus.wshihadeh.cloud grafana.wshihadeh.cloud dashboard.wshihadeh.cloud

    $> git clone [git@github.com](mailto:git@github.com):wshihadeh/traefik_monitoring.git
    $> cd traefik_monitoring
    $> make deploy

**Building the monitoring dashboard**

Once the monitoring stack is deployed and all services are running, you should be able to access Grafana web interface on the domain name defined in the above steps grafana.wshihadeh.cloud.

![](https://cdn-images-1.medium.com/max/6720/1*NuG8eSWNmhU0z6lE8RxJLQ.png)

After a successful login to Grafana, we can start by adding prometheus as a datasource and then start creating a new dashboard form the left side menu as shown in the below image.

→ Add prometheus datasource can be done simply by clicking on the “Data sources” item from the configuration menu on the left and then add a Prometheus data source as shown in the below images.

![](https://cdn-images-1.medium.com/max/2004/1*VNT5fp0QQmYM7eYwMpKMKA.png)

![](https://cdn-images-1.medium.com/max/2588/1*axAlNdg5T_yGJ2Qnm0AVuQ.png)

→ Adding a dashboard can be done from the left side menu as shown in the below image.

![](https://cdn-images-1.medium.com/max/6720/1*oCRjpp8N0NqaNDHVo6iJHA.png)

**→ Dashboard Variables**

Before jump into adding panels and graphs to our Grafana dashboard, we need to define a couple of variables that will be used in the graphs and panels of the dashboard. Below is a brief description of these variables:

* Job: The variable will be used to select the Prometheus job name (The source of the metrics). In our example, we have only one job however Prometheus can be configured to scrape metrics for multiple jobs. The below query can be used to list all the supported jobs

    label_values(job)

![](https://cdn-images-1.medium.com/max/5216/1*YwtIEvyB41m8ZKr1ovxJKw.png)

* Service: This variable will hold the names of all the services managed by Traefik. The below query can be used to get the names of the services. Note that this variable depends on the ***job ***variable to get the services defined for the selected job.

    label_values(traefik_service_requests_total{job=~"[[job]]"},service)

![](https://cdn-images-1.medium.com/max/5172/1*mGlcJOJ5grSxI-9n3kesKA.png)

* Protocol: This variable will be used to list Traefik supported protocols. Below is the query needed for generating the variable data

    label_values(traefik_service_requests_total, protocol)

* Interval: This variable will be used to configure the metrics interval for some of the panels such as “Server-side errors in the last X interval”.

→ **Dashboard Panels**

![](https://cdn-images-1.medium.com/max/6344/1*_IkuFdG9pj4ZvoDWI-sSIQ.png)

* **Traefik UP time**: This panel will tell us for how long is the selected Traefik service is up and running.

    time() - process_start_time_seconds{job="$job"}

* **Average response time**: This panel will display the average response time for all the requests served by the selected Traefik job. Below is the Prometheus query needed for this panel.

    sum(
      traefik_entrypoint_request_duration_seconds_sum{job=~"$job"}
    )/
    sum(
      traefik_entrypoint_requests_total{job=~"$job"}
    )

**Server-side error count**: This panel displays the sum of the increase in the server-side for the given interval. In other words, how many server-side errors happened in the selected interval. Below is the Prometheus query needed for this panel.

    sum(
      increase(
        traefik_service_requests_total{
          code=~"5\\d\\d",job=~"$job",
          service=~"$service",
          protocol=~"$protocol"
        }[$interval]
      )
    )

**Client-side error count**: Present the count of client errors in the selected interval.

    sum(
      increase(
        traefik_service_requests_total{
          code=~"4\\d\\d",job=~"$job",
          service=~"$service",
          protocol=~"$protocol"
        }[$interval]
      )
    )

**None-error responses count**: Displays the count of all successful requests (2xx and 3xx requests).

    sum(
      increase(
        traefik_service_requests_total{
          code=~"(2|3)\\d\\d",job=~"$job",
          service=~"$service",
          protocol=~"$protocol"
        }[$interval]
      )
    )

**Requests count by response code**: Display the distribution of the incoming requests grouped by their response code.

    sum(
      label_replace(
        traefik_service_requests_total{
          protocol=~"$protocol",
          job=~"$job",
          service=~"$service"
        }, 
      "response_code",  "$1,xx", "code", "(\\d).+")
    )  by (response_code)

![](https://cdn-images-1.medium.com/max/2000/1*e07ymzVo-vOttlHYbXK4Tg.png)

**Average response time by service: **This panel displays the average response time for each of the services supported by Traefik.

    sum( traefik_service_request_duration_seconds_sum{job=~"$job",service=~"$service",protocol=~"$protocol"}
    ) by (service) /
    sum(
    traefik_service_request_duration_seconds_count{job=~"$job",service=~"$service",protocol=~"$protocol"}
    ) by (service)

![](https://cdn-images-1.medium.com/max/6652/1*Q9WEpJMtntLis3dtpzqwsw.png)

**A server-side error by service: **Shows** **the server-side errors grouped by the backend service. Below is the Prometheus query needed for generating this graph.

    sum(
      increase(
        traefik_service_requests_total{
          job="$job",
          service=~"$service",
          code=~"5\\d\\d",
          protocol=~"$protocol" 
        }[$interval]
      )
    ) by (service)

**A client-side error by service**: Shows** **the client-side errors grouped by the backend service. Below is the Prometheus query needed for generating this graph.

    sum(
      increase(
        traefik_service_requests_total{
          job="$job",
          service=~"$service",
          code=~"4\\d\\d",
          protocol=~"$protocol" 
        }[$interval]
      )
    ) by (service)

**Total requests by service**: Shows** **the total count of incoming requests grouped by the backend service. Below is the Prometheus query needed for generating this graph.

    sum(
      increase(
        traefik_service_requests_total{
          job="$job",
          service=~"$service",
        }[$interval]
      )
    ) by (service)

![](https://cdn-images-1.medium.com/max/6604/1*54xlUf_L4zpUV06eQez5zA.png)

**Service open connection**: This panel shows the count of the services open connections grouped by the request method (GET, PUT, POST). Below is the Prometheus query needed for generating this graph.

    sum(
     traefik_service_open_connections{
      job=~"$job", service=~"$service"}
    ) by (method)

**Alerts**

Grafana supports defining alters and notifications on the defined graphs. The notifications can be sent to several channels such as email, Slack or Hipchat. The below image shows how to define a slack channel from the Grafana interface.

![](https://cdn-images-1.medium.com/max/2928/1*A6yiTJXV8aJg50M9dH6anw.png)

Defining the alters can be done on the individual graphs on the alerts tap as shown below in the image.

![](https://cdn-images-1.medium.com/max/6660/1*eUoO6mJ0qYfIYH-mFJA_5Q.png)

Once the alert is triggered, it will deliver the notification to the defined channels (you can have multiple channels for each alert). The notification will look like the below image on Slack.

![](https://cdn-images-1.medium.com/max/2276/1*NDTx00RGs8UlmmE1_nxpHA.png)

**Conclusion**

Grafana is a great visualization tool that can be used to build several types of graphs or diagrams to visualize the data from several data sources such as MySQL or Prometheus. In addition, Grafana supports defining and managing alerts based on the defined graphs.

The full implementation of the [POC](https://github.com/wshihadeh/traefik_monitoring) presented in this post can be found on [GitHub](https://github.com/wshihadeh/traefik_monitoring). In addition, the [Traefik dashboard](https://grafana.com/grafana/dashboards/11741) presented in this post can be found on the [Grafana web site here](https://grafana.com/grafana/dashboards/11741).
