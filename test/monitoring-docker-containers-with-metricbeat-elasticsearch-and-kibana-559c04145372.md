Unknown markup type 10 { type: [33m10[39m, start: [33m35[39m, end: [33m38[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m82[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m123[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m70[39m, end: [33m77[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m54[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m209[39m, end: [33m223[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m339[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m1256[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m30[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m162[39m, end: [33m173[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m207[39m, end: [33m212[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m139[39m, end: [33m155[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m24[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m12[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m144[39m, end: [33m153[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m155[39m, end: [33m158[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m160[39m, end: [33m166[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m168[39m, end: [33m179[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m181[39m, end: [33m185[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m187[39m, end: [33m195[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m199[39m, end: [33m206[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m181[39m, end: [33m190[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m258[39m, end: [33m267[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m279[39m, end: [33m293[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m297[39m, end: [33m312[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m350[39m, end: [33m356[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m371[39m, end: [33m380[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m34[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m95[39m, end: [33m105[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m219[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m255[39m, end: [33m284[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m61[39m, end: [33m67[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m38[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m29[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m220[39m }

# Monitoring Docker Containers With Metricbeat, Elasticsearch, and Kibana

So you have moved all your applications to Docker and have begun enjoying all the fruits of lightweight and fast-to-deploy containers.

That’s great, but once you have multiple containers spread across multiple nodes, you’ll need to find a way to track their health, storage, CPU, memory usage, network load, etc.

To track these metrics, you need an efficient monitoring solution and some backend store to keep your container data for subsequent analysis and processing. Managing thousands of Docker containers in production made our team here at Qbox quickly realize that Docker container monitoring is a valuable addition to our cluster management process.

In the [previous article](https://qbox.io/blog/shipping-kubernetes-cluster-metrics-to-elasticsearch-with-metricbeat), we discussed how to use Metricbeat to ship metrics from Kubernetes. Now, it’s time to share our experience of using Metricbeat to monitor bare Docker containers and shipping container data to Elasticsearch and Kibana. This knowledge may be useful for developers and administrators who manage Docker containers without orchestration. Let’s get started!

## Prerequisites

Examples in this tutorial were tested in the following environment:

* Ubuntu 16.04 (Xenial Xerus)

* Metricbeat 6.3.2 download from the apt repository

* Docker 18.03.1-ce

We assume that you already have a working Docker environment on your system and a few containers running. If not, see the official [Docker installation guide](https://docs.docker.com/install/) and learn how to run Docker containers as daemons. You’ll need to have at least one container running in Docker to ship some useful data to Elasticsearch and Kibana.

## Install Metricbeat

Once your Docker containers are up and running and the Docker environment is properly configured, we can move on to installing Metricbeat. As you might already know, Metricbeat is a lightweight data shipper that can periodically collect metrics from the OS, applications, and services running on the server(s). It can be configured to work with a variety of metrics sources like OS, Redis, Zookeeper, and Apache and to send their metrics directly to Elasticsearch.

To install Metricbeat, you first need to add Elastic’s signing key used to verify the downloaded package:

    wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

Next, add the Elastic repository to your repository source list:

    echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list

Finally, update the repos on your system and install Metricbeat using apt-get:

    sudo apt-get update && sudo apt-get install metricbeat

## Metricbeat General Configuration

To run Metricbeat, you should configure input (metrics sources like Docker), output (a remote service or database to send these metrics to), and various modules if needed. This configuration is located in the metricbeat.yml inside the Metricbeat folder. Take a look at what edits we've made:

    #================ Modules configuration ============================ 
    metricbeat.config.modules: 
      # Glob pattern for configuration loading path:
      ${path.config}/modules.d/*.yml
      # Set to true to enable config reloading 
      reload.enabled: false 
      # Period on which files under path should be checked for changes   
      reload.period: 10s

    #==========Elasticsearch template setting ========================== 
    setup.template.settings: 
      index.number_of_shards: 1 
      index.codec: best_compression 
      #_source.enabled: false 
    #===================Dashboards ===================================== 
    # These settings control loading the sample dashboards to the Kibana #index. Loading the dashboards is disabled by default and can be 
    # enabled either by setting the options here, or by using the `-#setup` CLI flag or the `setup` command. 
    setup.dashboards.enabled: true 
    #=======================Kibana ===================================== 
    # Starting with Beats version 6.0.0, the dashboards are loaded via #the Kibana API. This requires a Kibana endpoint configuration.
    setup.kibana.host: "YOUR_KIBANA_HOST" 
    setup.kibana.protocol: "https" 
    setup.kibana.username: "YOUR_KIBANA_USERNAME" 
    setup.kibana.password: "YOUR_KIBANA_PASSWORD" #====================Outputs ===================================== 
    # Configure what output to use when sending the data collected by 
    # the beat. 
    #-------------------Elasticsearch output ---------------------------
    output.elasticsearch: 
      hosts: ["YOUR_ELASTICSEARCH_HOST"] 
      username: "YOUR_ELASTICSEARCH_USERNAME" 
      password: "YOUR_ELASTICSEARCH_PASSWORD" 

There are several configuration options to pay attention to:

* metricbeat.config.modules.path -- we load module configurations from external files to keep things isolated. All module configuration files are located under the /modules.d/ folder so to target them we used *.yml glob pattern. We have also enabled config reloading. Metricbeat will periodically monitor our configuration files, and, if any changes are detected, it will reload the entire configuration.

* setup.template.setting -- specifies the index template for Metricbeat. Our Metricbeat index will have 1 shard and will be compressed using best_compression type based on high compression ratio.

* setup.dashboards.enabled -- we will be loading Kibana example dashboards for Metricbeat. These dashboards include visualization and searches examples for our metrics.

* setup.kibana -- for the dashboards to work, we need to specify the Kibana endpoint. You'll need to enter the URL of your Kibana host and any credentials (username/password) if needed.

* output.elasticsearch -- specifies the output to which we send Metricbeat metrics. We are using Elasticsearch, so you'll need to provide Elasticsearch host, protocol, and credentials if needed.

## Metricbeat Docker Module

To fetch metrics from Docker containers, we are going to use [Metricbeat Docker module](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-module-docker.html). It comes with a number of default metricsets we, such as container, cpu, diskio, healthcheck, info, memory, and network.

First, we need to manually enable the Docker module because we load external configuration files into our general configuration. The default Docker module configuration sits in the modules.d directory. We can enable or disable any module configuration under modules.d by running modules enable or modules disable commands. For example, to enable the docker config in the modules.d directory, you can run:

    ./metricbeat modules enable docker

Now, as the module is enabled, let’s tweak its configuration. This is how it looks like in the docker.yml file:

    - module: docker 
      metricsets: 
      - "container" 
      - "cpu" 
      - "diskio" 
      - "healthcheck" 
      - "info" 
      - "image" 
      - "memory" 
      - "network" 
      hosts: ["unix:///var/run/docker.sock"]
      period: 10s 
      enabled: true

This is a minimal configuration suffice to get Metricbeat going. We have specified 8 metricsets including “image” metricset not included by default. Also, your Docker module needs access to the Docker daemon. By default, Docker listens on the Unix socket "unix:///var/run/docker.sock". We can use this socket to communicate with the daemon from within a container. Through this endpoint, Docker exposes the Docker API which can be used to get a stream of all events and statistics generated by Docker.

The next important configuration field we need to mention is period. This field defines how often Metricbeat accesses the Docker API. According to the official Metricbeat documentation, it is strongly recommended to run a Docker module with a period of at least 3 seconds or longer. That is because the request to Docker API takes up to 2 seconds itself, so specifying less than 3 seconds can cause request timeouts and no data returned to those requests.

Great! Now, everything is ready to run Metricbeat. One last advice is to have your Metricbeat instance as close to Docker as possible (preferably on the same host) to minimize network latency.

On Linux, you can run Metricbeat in the shell specifying the config file as a parameter.

    sudo ./metricbeat -e -c metricbeat.yml

Alternatively, you can start Metricbeat as a service to run at startup:

    sudo service metricbeat start

Almost immediately after being started, Metricbeat will begin sending Docker metrics to Elasticsearch index. You can verify this by curling your Elasticsearch host:

    curl -XGET 'localhost:9200/_cat/indices?v&pretty' 
    health status index uuid pri rep docs.count docs.deleted store.size pri.store.size 
    yellow open  metricbeat-6.3.2-2018.08.09 BikpOgqQR-pU_SuKrm5vw 1 1 2374 0 1.4mb 1.3mb

As you see, our index was created! Now, you can create an index pattern for this index in Kibana and access metrics directly from the Kibana dashboard.

Log in to your Kibana dashboard and follow the simple steps for creating a new Index Pattern under Management -> Index Patterns -> Create Index Pattern. If everything is ok, you’ll see the index mapping similar to this:

![](https://cdn-images-1.medium.com/max/5200/0*fx4uJfurOTBbtKFI.png)

You can see that Metricbeat created fields for various Docker module metricsets such as container, CPU, etc. You can also find batches of metrics shipped to Elasticsearch by clicking on Discovery tab in Kibana dashboard. You should see something like this:

![](https://cdn-images-1.medium.com/max/5200/0*Q2sBirO7CspoWoi2.png)

Awesome! As you remember, we loaded Kibana dashboards, so you can access example visualizations of the Metricbeat data in Kibana.

![](https://cdn-images-1.medium.com/max/5200/0*my1_YSxtX0Kx4PWz.png)

Pretty cool, isn’t it? In the dashboard, we have the statistics of running containers as well some data about their memory and CPU usage. You can experiment with Docker data by creating your custom visualizations. Check out our tutorial about [building visualizations for Metricbeat data in Kibana](https://qbox.io/blog/how-to-use-elasticsearch-to-visualize-data).

## Conclusion

That’s it! You have learned how to monitor Docker containers using Metricbeat, Elasticsearch, and Kibana. Metricbeat Docker module exposes key metrics and events provided by the Docker API. Making them accessible for subsequent processing and analysis in Kibana is really a great monitoring solution that requires minimal configuration. Installing Metricbeat and configuring Docker module should not take more than half an hour. Once the Docker container data is available in Elasticsearch, you can use powerful Kibana visualization tools like Visual Builder and Timelion to analyze your Docker containers.

Stay tuned to our upcoming tutorials to learn more about the Beats Autodiscover feature that allows automatically detecting container start/stop events and launch corresponding modules with the required configuration on the fly.

*Originally published at [qbox.io](https://qbox.io/blog/monitoring-docker-containers-with-metricbeat-elasticsearch-and-kibana).*
