Unknown markup type 10 { type: [33m10[39m, start: [33m440[39m, end: [33m460[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m178[39m, end: [33m185[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m416[39m, end: [33m443[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m276[39m, end: [33m296[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m76[39m, end: [33m85[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m367[39m, end: [33m378[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m728[39m, end: [33m742[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m757[39m, end: [33m767[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m793[39m, end: [33m809[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m869[39m, end: [33m880[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m120[39m, end: [33m129[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m23[39m, end: [33m33[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m270[39m, end: [33m285[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m116[39m, end: [33m145[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m314[39m, end: [33m320[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m229[39m, end: [33m250[39m }

# Centralize Your Docker Logging With FluentD

Centralize Your Docker Logging With FluentD

### Start Shipping Containers Logs to a Centralized Logging Server Using FluentD

![Photo by [Hubert Neufeld](https://unsplash.com/@htn_films?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/8480/0*tLxVNgr7k7ymqdgd)*Photo by [Hubert Neufeld](https://unsplash.com/@htn_films?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

In my previous post, [Centralize Your Docker Logging With Syslog](https://medium.com/@wshihadeh/docker-centralized-logging-with-syslog-97b9c147bd30)**, **I explained how to implement a centralized logging system for Docker services using Syslog. The described approach is only applicable to the cases where there are some limitations on the resources that can be used for building the infrastructure or limitation of running modern logging systems. On the other hand, these days there are several options to build a centralized logging system one of them is to use Fluentd, Elasticsearch, and Kibana also called EFK stack.
[**Centralize Your Docker Logging With Syslog**
*The best way to understand our systems and their successes or failures is through great logging*medium.com](https://medium.com/better-programming/docker-centralized-logging-with-syslog-97b9c147bd30)
> Fluentd is a cross-platform open-source data collection software project originally developed at Treasure Data. It is written primarily in the Ruby programming language.[Wikipedia](https://en.wikipedia.org/wiki/Fluentd)

Fluentd can be used to collect application logs and parse them to be ready for shipping to a remote destination (In our case this would be the Elasticsearch instance) or storing the logs locally in logs files.

In this post, I will demonstrate how to build a centralized logging system in Docker Swarm and parse the application logs using Fluentd.

### Logging Stack Applications:

**Demo Application**

The first application that we need to deploy is the demo application that simulates generating logs for services running in Swarm clusters. This application can be any application that is able to run as a docker swarm services and write logs to STDOUT. For simplicity reasons, I prepared a Docker image for a lightweight container that generates logs messages that describe the current system status. The Docker image for this container is wshihadeh/busylogbox. This application will produce logs that look like the below text.

<iframe src="https://medium.com/media/b8ab0d981da975a9f94c157af0359c76" frameborder=0></iframe>

The logic behind the application is shown below

<iframe src="https://medium.com/media/393b9c6665b7d5f474b34bdc68d5ed0f" frameborder=0></iframe>

**Elasticsearch**

The second application needed for the stack is the Elasticsearch. This application will serve as the database of the logs and the logs indices. In other words, we will configure FluentD to forward the logs to the Elasticsearch instance to be stored and indexed by the service. Elasticsearch allows viewing and looking up logs based on all the attributes that are available. I will be using the official Docker image elastic/elasticsearch:7.5.1 for deploying the service.

**Kibana**

This application provides us with an easy and intuitive web interface to connect to the Elasticsearch service, view logs, query the logs, visualize the logs using several types of graphs and many more features. For deploying Kibana, I will use the latest official Docker imageelastic/kibana:7.5.1.

**Fluentd**

We will be using Fluentd to collect the application logs, parse them and then forward them to the Elatsicsearch instance. To achieve this goal, I decided to build a Docker image that contains all the need libraries and configurations to run Fluentd and parse the logs. Below is a step by step description for witing the Flunetd configurations and building the Docker image.

**FlunetD Configurations**

The first section in FlunetD configuration is the source section, This section defines the log files that will be parsed by Flunetd. In the Docker world, containers are expected to write logs to the **STDOUT** of the containers and docker also provides a way to configure the log driver for each container. This means that we can configure docker to forward the logs directly to the Fluentd instances and then configure Flunetd with a **forward source section, **However, I don’t prefer this idea for several reasons:

* Logs will not be available from the command line on the Docker clusters.

* There is a risk to lose some of the logs in case the Flunetd service is down.

For the above reasons, I recommend keep using the Docker default log driver json-file and use a **tail source section. **By default, every Docker container will have a log file under the following path **/var/lib/docker/containers **The logs in this file will be the same logs that are sent to the STDOUT of the docker container and the same logs that is displayed using the docker logs command. Below are the configurations needed to instructing Flunetd to start tailing the logs files for all the deployed docker containers with a brief description of the most important keys used in the configurations

<iframe src="https://medium.com/media/9fa7b502fc6ee9e519dafa8bde00842c" frameborder=0></iframe>

* **@type**: This attribute defines the type of the input plugin, as mentioned above Flunetd is supporting several [input plugins](https://docs.fluentd.org/input).

* **@id**: A unique identifier that represents the input source.

* **path**: The path of the files to be read.

* **pos_file**: The path to the position file. a file that is used to store the last read position.

* **tag**: The used tag for tagging log message that comes from this input.

* **format**: The expected format of the input data.

Once the input source is configured, we can start processing the log messages and perform actions on these messages. The first action that I usually do is add Docker metadata to the log message such as the service namespace, service name, and the container name. This task can be achieved using the [docker_metadata](https://github.com/wshihadeh/fluent-plugin-filter-docker_metadata) plugin. Below is the configuration needed to attach these attributes to all log messages that are read from the defined input source (the tag is the key for selecting these logs).

<iframe src="https://medium.com/media/435e504876af6a07f6ece1db9ca691e5" frameborder=0></iframe>

The next step is to classify the logs based on their attributes to able to handle them separately. This is a very important step in the cases where the swarm cluster hosts applications with different log formats and there is a need to parse the logs from these applications differently. This is also applicable to the stack that we are trying to build since each of the applications described above will have a different log format and therefore we need to handle them separately. For the sake of simplicity, I decided to parse and forward the logs of the demo application only. Logs form the other applications will be simply ignored. Below are the configurations that I sued to classify the logs based on the Docker attribute container_name. As shown the busylogbox logs will be tagged with row.busyboxlog.* tag while logs from other applications will be tagged with ignore.dlog .

<iframe src="https://medium.com/media/87f59f3cfb5c84a5069850bf6f29ba89" frameborder=0></iframe>

Next, we need to handle the logs of the application. on the first hand, we will ignore all the log messages tagged with ignore.** tags.

<iframe src="https://medium.com/media/5253ceda603763fd4de44da8de412d4d" frameborder=0></iframe>

On the other hand, The busylogbox logs need to be parsed and more attributes need to be extracted from the log messages to be able to use them in Kibana and to build diagrams and visualizations based on the extracted attributes. To achieve this task I will be using the record_reformer plugin to modify the log messages and add attributes to the messages. In addition, Ruby Regex is needed to extract the values of each of these attributes.

<iframe src="https://medium.com/media/f239f1eeea55d8b4ca00cd70fc21d90d" frameborder=0></iframe>

One we are done with parsing the logs messages for the application, we can perform a final check on the log messages to remove all empty attributes and provide default values for the defined attributes. Below are the configurations that can be used to achieve this task.

<iframe src="https://medium.com/media/a86250899e62ab20378e437076f91b47" frameborder=0></iframe>

The last step is to configure Flunetd to forward the logs to Elasticsearch instance. This task can be achieved by using the Elasticsearch plugin and below are the configurations needed to implement the forward features.

<iframe src="https://medium.com/media/487e52fbabc67ee52296c83e6dacfb71" frameborder=0></iframe>

Configuring the buffer section is very important for keep retrying to send the logs to the Elasticsearch in the cases where the connection to the Elasticsearch is not available for a period of time. In these cases, Flunetd will write the log messages to the defined buffer and will keep retrying to send these logs to the Elasticsearch instance.

**Building the base Docker image**

Now we are done with writing the Fluentd configurations, the next step to deploy Fluentd service to docker swarm. Therefore we need first to build a custom docker image that includes all the dependencies (Flunetd plugins and configurations) needed for parsing the logs.

Since Installing these libraries and plugins can take some time, I decided to build a Fluentd base image that includes these libraries and then use the base image to build Fluentd docker images to collect logs.

<iframe src="https://medium.com/media/68fe463178a343e39a385aae2c8de273" frameborder=0></iframe>

The above Dockerfile is used to generate the Fluentd base image which is based on the official Fluentd Docker image fluent/fluentd:v1.2.2-onbuild. The Dockerfile for the base image includes instructions for installing the needed plugins for parsing the logs such as **fluent-plugin-filter-docker_metadata, **which is a plugin that attaches docker metadata to the logged messages such as the container name and the plugin “**fluent-plugin-elasticsearch” **is used to forward the logs to the Elasticsearch instance.

**Building Fluentd Docker image**

Using the base image for Flunetd makes it very easy, quick and simple to build Flunetd Docker images that include the needed configuration. Below is a sample of a Dockerfile for such an image that only takes care of including the Flunetd configurations and the entry-point into the docker image. The main benefit of this approach is that building docker images will be much faster in the cases where the change only affects the Flunetd configurations.

<iframe src="https://medium.com/media/dcd69886476e86312ff6f4a7ee13a4be" frameborder=0></iframe>

**Deploy the demo stack**

All the applications mentioned above can be deployed to Swarm clusters simply using the below Docker stack file. The below stack file includes the service definition for Fluentd, Busylogbox, Elasticsearch, and Kibana. One important note regarding Fluentd configs is that the service needs to be deployed using the global mode to make sure that we deploy a Fluentd container on each of the Docker swarm hosts to collect the logs from all the containers on these nodes.

<iframe src="https://medium.com/media/39410057035563930c888ce33f6f3377" frameborder=0></iframe>

Deploying the above stack can simple done by executing the below command on a Docker Swarm manager node.

    $ docker stack deploy -c docker-stack-fke.yml logging

**View logs on Kibana**

Once the services are deployed and Elasicsearch and Kibana are healthy and accessible, we can proceed by configuring Kibana to be able to view the logs. To do so we need to connect to Kibana web interface under the following URL [http://127.0.0.1:5601](http://127.0.0.1:5601.). And then add a new index object as shown in the image below

![](https://cdn-images-1.medium.com/max/4836/1*E56YKlCrdSJ-71Z7Enh3Jg.png)

Once the index is created, the logs will be viable in the search page and you can start filtering the logs based on the parsed attributes

![](https://cdn-images-1.medium.com/max/5364/1*VI4SoaJDDC_JY22tsgVFQQ.png)

![](https://cdn-images-1.medium.com/max/5216/1*1o0bqQq6CLsEUKpWYibE0Q.png)

As shown in the above images, the parsed attributes will be available in the Kibana web interface and the end-user can filter the logs, build visualizations, diagrams, and dashboards based on these attributes.

**Conclusion**

Syslog is not the only way to implement a centralized logging system 😆, In fact, there are several other options to implement it. Fluentd combined with Elasticsearch and Kibana is considered to be another method for Building a centralized logging system. In addition, Fluentd provides us with an easy way to analyze, parse and extract attributes from the log messages and as a result visualizing and querying the logs based on these attributes is much easier.
