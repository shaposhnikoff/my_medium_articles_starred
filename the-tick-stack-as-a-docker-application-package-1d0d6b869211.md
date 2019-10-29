Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m12[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m172[39m, end: [33m186[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m82[39m, end: [33m86[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m110[39m, end: [33m120[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m80[39m, end: [33m90[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m49[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m61[39m, end: [33m74[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m24[39m, end: [33m38[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m18[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m38[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m12[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m12[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m64[39m, end: [33m66[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m37[39m, end: [33m49[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m96[39m, end: [33m114[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m16[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m15[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m12[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m21[39m, end: [33m39[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m96[39m, end: [33m110[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m89[39m, end: [33m101[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m163[39m, end: [33m175[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m36[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m140[39m, end: [33m147[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m25[39m, end: [33m32[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m54[39m, end: [33m72[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m47[39m, end: [33m60[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m107[39m, end: [33m121[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m61[39m, end: [33m69[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m56[39m, end: [33m62[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m25[39m, end: [33m32[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m94[39m, end: [33m102[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m67[39m, end: [33m76[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m111[39m, end: [33m115[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m254[39m, end: [33m258[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m173[39m, end: [33m185[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m141[39m, end: [33m155[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m117[39m, end: [33m128[39m }

# The TICK Stack as a Docker Application Package

TL;DR

![Photo by [Tirza van Dijk](https://unsplash.com/@tirzavandijk?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/search/photos/application?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/8094/1*suQarOhWeCNL3VQojarQcA.jpeg)*Photo by [Tirza van Dijk](https://unsplash.com/@tirzavandijk?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/search/photos/application?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

[Docker](https://www.docker.com/) Application Package is a utility presented at DockerCon SF 2018. It eases the packaging and the distribution of a Docker Compose application. The TICK stack is a good candidate to illustrate how this actually works.

### About the TICK stack

This application stack is mainly used to handle time-series data. That makes it a great choice for IoT projects, where devices send data (temperature, weather indicators, water level, etc.) on a regular basis.

Its name, TICK, comes from its components:

* Telegraf

* InfluxDB

* Chronograf

* Kapacitor

The schema below illustrates the overall architecture, and outlines the role of each component.

![](https://cdn-images-1.medium.com/max/2000/1*A7uRhGM_vOXFxT7kdqwNfw.png)

Basically, data is sent to Telegraph, then stored in an InfluxDB database. Chronograf allows to query the database through a nice web interface. Kapacitor can process, monitor, and raise alerts based on the data.

### Simple definition in a Compose file

The tick.yml file below defines the four components of the stack and the way they communicate with each other.

<iframe src="https://medium.com/media/99b6491ee6117740c9b02ed767c30771" frameborder=0></iframe>

TK Don’t think we have the Medium formatting issue anymore in gist? Note: if you copy/paste the content of this file, you’ll have to change the ‘—’ into a regular hyphen ‘-’ character. (Medium formatting issue here).

TK left both gist and grey lined code — please advise which is right. The Telegraf’s configuration is provided through a Docker Config object, created out of the following telegraf.conf file.

<iframe src="https://medium.com/media/482208dd516edf42e265b1a5b9588deb" frameborder=0></iframe>

    [agent]
      interval = "5s"
      round_interval = true
      metric_batch_size = 1000
      metric_buffer_limit = 10000
      collection_jitter = "0s"
      flush_interval = "5s"
      flush_jitter = "0s"
      precision = ""
      debug = false
      quiet = false
      logfile = ""
      hostname = "$HOSTNAME"
      omit_hostname = false

    [[outputs.influxdb]]
      urls = ["http://influxdb:8086"]
      database = "test"
      username = ""
      password = ""
      retention_policy = ""
      write_consistency = "any"
      timeout = "5s"

    [[inputs.http_listener]]
      service_address = ":8186"

    [cpu]
      # Whether to report per-cpu stats or not
      percpu = true
      # Whether to report total system cpu stats or not
      totalcpu = true

I won’t go too deep into the details of this file, but basically it:

* Defines an agent that gathers CPU metrics of the host on a regular basis.

* Defines an additional input method with allows Telegraf to receive data through an HTTP endpoint.

* Specifies that the data collected/received should be stored in the database named test*.*

## Docker Application Package

### **Installation (on MacOS)**

The following command allows you to install the latest version to date of the Docker Application Package (the docker-app Go binary)

    # Install release 0.6.0 (published on October 4th)
    VERSION="v0.6.0"
    wget --no-check-certificate https://github.com/docker/app/releases/download/$VERSION/docker-app-darwin.tar.gz
    tar xf docker-app-darwin.tar.gz
    cp docker-app-darwin /usr/local/bin/docker-app

We then check everything is fine.

    # Check version
    **$ docker-app version**
    Version:      v0.6.0
    Git commit:   9f9c6680
    Built:        Thu Oct  4 13:30:33 2018
    OS/Arch:      darwin/amd64
    Experimental: off
    Renderers:    none

### Available commands

Just for reference, the screenshot below lists all the commands provided by the docker-app CLI. We will illustrate some of them later in this article.

![](https://cdn-images-1.medium.com/max/2984/1*g1s9drrzJnLWlA9zMXlTMQ.png)

### **Creation of a Docker Application Package for the TICK stack**

Let’s start with a folder that only contains the Compose and configuration files defined above.

    $ tree .
    .
    ├── telegraf.conf
    └── tick.yml

We can now create the Docker Application, named “tick”, using the following command:

    $ docker-app init -c tick.yml tick

This creates the folder tick.dockerapp, and three additional files.

    $ tree .
    .
    ├── telegraf.conf
    ├── **tick.dockerapp
    **│ ├── **docker-compose.yml**
    │ ├── **metadata.yml**
    │ └── **settings.yml**
    └── tick.yml

* docker-compose.yml is the copy of the tick.yml file

* metadata.yml defines additional parameters as we can see below. I’ve just changed the namespace to match the Docker Hub account that will be used when publishing the application.

    $ cat tick.dockerapp/metadata.yml
    # Version of the application
    version: 0.1.0
    # Name of the application
    name: tick
    # A short description of the application
    description:
    # Namespace to use when pushing to a registry. This is typically your Hub username.
    namespace: lucj
    # List of application maintainers with name and email for each
    maintainers:
     — name: luc
       email:

* settings.yml defines the default parameters used for the application (more on this in a bit). By default this file is empty.

Note: when initializing the Docker App it’s possible to use the -s flag. This would result in the creation of a single file, with the content of the three files above, instead of a folder/files hierarchy.

**Define settings for dev and prod environments**

As we said above, the purpose of the settings.yml file is to provide some default value for the application. But, wait… default value for which parameters, as we did not specified any? This is actually one of the main features of the Docker Application Package — the possibility to use different settings for different environments.

Let’s consider a dev and a prod environment and suppose that those two only differ when it comes to the port exposed by the application to the outside world:

* Telegraf listens on port 8000 in dev and 9000 in prod.

* Chronograf listens on port 8001 in dev and 9001 in prod.

**Note**: of course, in a real world application, differences between dev and prod would not be limited to a port number. The current example is over-simplified, so it makes it easier to grasp the main concepts.

To do so, we will first create a settings file for each environment and then we will modify the docker-compose.yml file to add some placeholders.

The settings.yml file is used to define some default ports for both Telegraf and Chronograf services.

    // settings.yml
    ports:
      telegraf: 8186
      chronograf: 8888

We also define dev.yml to specify different values for the development environment,

    // dev.yml
    ports:
      telegraf: 8000
      chronograf: 8001

and prod.yml, for the production environment.

    // prod.yml
    ports:
      telegraf: 9000
      chronograf: 9001

Let’s now modify the docker-compose.yml file, and change the published ports of Telegraf and Chronograf, so they’re using the variables defined in the above files.

    $ cat tick.dockerapp/docker-compose.yml
    version: "3.6"
    services:
      telegraf:
        image: telegraf
        configs:
        — source: telegraf-conf
          target: /etc/telegraf/telegraf.conf
        ports:
        — **${ports.telegraf}**:8186
      influxdb:
        image: influxdb
      chronograf:
        image: chronograf
        ports:
        — **${ports.chronograf}**:8888
        command: ["chronograf", "--influxdb-url=http://influxdb:8086"]
      kapacitor:
        image: kapacitor
        environment:
        — KAPACITOR_INFLUXDB_0_URLS_0=http://influxdb:8086
    configs:
      telegraf-conf:
        file: ./telegraf.conf

As we can see in the changes above, the way to access the port number of Telegraf is to use the ports.telegraf notation. The same goes for the Chronograf port.

The render command allows to generate the Docker Compose file substituting the variables ${ports.XXX} with the content of the settings file specified. The default settings.yml is used if none are specified. As we can see below the Telegraf port is now 8186, and the Chronograf one is 8888.

    $ docker-app render
    version: "3.6"
    services:
      chronograf:
        command:
        — chronograf
        — --influxdb-url=http://influxdb:8086
        image: chronograf
        ports:
        — mode: ingress
          target: 8888
          published: **8888**
          protocol: tcp
      influxdb:
        image: influxdb
      kapacitor:
        environment:
          KAPACITOR_INFLUXDB_0_URLS_0: http://influxdb:8086
        image: kapacitor
      telegraf:
        configs:
        — source: telegraf-conf
          target: /etc/telegraf/telegraf.conf
        image: telegraf
        ports:
        — mode: ingress
          target: 8186
          published: **8186**
          protocol: tcp
    configs:
      telegraf-conf:
        file: telegraf.conf

If we specify a setting file in the render command, the values within that file are used. As we can see on the following example, if we use dev.yml during the rendering, Telegraf is published on port 8000 and Chronograf on port 8001.

    $ docker-app render -f dev.yml
    version: "3.6"
    services:
      chronograf:
        command:
        — chronograf
        —-influxdb-url=http://influxdb:8086
        image: chronograf
        ports:
        — mode: ingress
          target: 8888
          published: **8001**
          protocol: tcp
      influxdb:
        image: influxdb
      kapacitor:
        environment:
          KAPACITOR_INFLUXDB_0_URLS_0: http://influxdb:8086
        image: kapacitor
      telegraf:
        configs:
        — source: telegraf-conf
          target: /etc/telegraf/telegraf.conf
        image: telegraf
        ports:
        — mode: ingress
          target: 8186
          published: **8000**
          protocol: tcp
    configs:
      telegraf-conf:
        file: telegraf.conf

### Inspect the application

As we can see below, the inspect command provides all the information related to the application:

* Its metadata.

* The services involved (including the number of replicas defined, the ports exposed and the image used).

* The settings used.

* The attached files. (All the files used on top of the docker-compose.yml)

    $ docker-app inspect
    tick 0.1.0

    Maintained by: luc

    Services (4) Replicas Ports Image
     — — — — — — — — — — — — — — — -
    influxdb 1 influxdb
    chronograf 1 8888 chronograf
    kapacitor 1 kapacitor
    telegraf 1 8186 telegraf

    Settings (2) Value
     — — — — — — — — -
    ports.chronograf 8888
    ports.telegraf 8186

    Attachments (4) Size
     — — — — — — — — — — 
    dev.yml 43B
    prod.yml 43B
    telegraf.conf 668B

**Deploy the application on a Swarm**

As the stack uses a Docker Config based on the telegraf.conf file, we need to copy this file over into the tick.dockerapp folder. Docker Application Package has allowed you to embed configuration files since version 0.6). The folder structure is then as follows:

    $ tree .
    .
    ├── telegraf.conf
    ├── tick.dockerapp
    │   ├── dev.yml
    │   ├── docker-compose.yml
    │   ├── metadata.yml
    │   ├── prod.yml
    │   ├── settings.yml
    │   └── telegraf.conf
    └── tick.yml

Using the following command, we deploy the application on a Swarm through a Docker stack.

    $ docker-app deploy -f prod.yml
    Creating network tick_default
    Creating config tick_telegraf-conf
    Creating service tick_influxdb
    Creating service tick_chronograf
    Creating service tick_kapacitor
    Creating service tick_telegraf

Listing the services created, we can see the values from the prod.yml settings file have been taken into account (as the exposed ports are 9000 and 9001 for Telegraf and Chronograf respectively).

Note: there is currently a small issue in 0.6.0, as the deploy command doesn’t always pick up the attachments from the package, but uses the one in the current folder in some cases. This issue is addressed [here](https://github.com/docker/app/pull/406), and will be fixed in the 0.6.1 release.

### Deploy an application on a non-Swarm host

We could deploy the application on a non-Swarm Docker host, for development purposes, with the following command:

    $ docker-app render -f dev.yml | docker-compose -f — up

The command can be split into two parts:

* The rendering, using the dev.yml settings file, to generate the final Docker Compose file.

* This generated Compose file is piped to the second command, which reads the file from the standard input and runs the Compose application.

Note: this is just for the records, as there is no real point in deploying the app with Docker Compose here. Indeed, the config primitive would not be taken into account by Compose as it’s a Swarm-only thing.

## Tests

As we deployed the application using the settings file related to the production environment (prod.yml), we will send data to port 9000 (the port published by Telegraf), and check the result on port 9001 (the port published by Chronograf).

Let’s send out some dummy data. For this purpose, we use the image lucj/genx which is a simple Go application that can generate data following linear or cosine distributions (will be enhanced in the future).

    $ docker run lucj/genx
    Usage of /genx:
     -duration string
          duration of the generation (default “1d”)
     -first float
          first value for linear type
     -last float
          last value for linear type (default 1)
     -max float
           max value for cos type (default 25)
     -min float
           min value for cos type (default 10)
     -period string
           period for cos type (default “1d”)
     -step string
           step / sampling period (default “1h”)
     -type string
           type of curve (default “cos”)

In this example we simulate a temperature following a cosine curve. Let’s generate three days of data, with a one day period, min/max values of 10/25 and a sampling step of one hour. Obviously not a real world temperature model but enough for the tests.)

    $ docker run lucj/genx:0.1 -type cos -duration 3d -min 10 -max 25 -step 1h > /tmp/data

We then send those data to the Telegraf endpoint with a couple of shell commands:

    PORT=9000
    endpoint="http://localhost:$PORT/write"
    cat /tmp/data | while read line; do
      ts="$(echo $line | cut -d' ' -f1)000000000"
      value=$(echo $line | cut -d' ' -f2)
      curl -i -XPOST $endpoint --data-binary "temp value=${value} ${ts}"
    done

From the Chronograf UI, we can see the data was correctly received.

![](https://cdn-images-1.medium.com/max/5040/1*RcGt9hhefACcfbt9_TIP3w.png)

**Push the application to Docker Hub**

Once the application is working fine, it can be distributed through the Docker Hub via a simple push.

    $ docker-app push
    sha256:bb16877acb67...3462d4ac81a1cf440

We can then see it’s there, alongside the “regular” Docker images of my Docker Hub account.

![](https://cdn-images-1.medium.com/max/3804/1*4nuWmYyeWXeKKfVkXKS9xw.png)

The application is now ready to be used by anyone. If you want to go and modify this application, just use the fork command and specify (optional) a folder in which all the application’s assets will be downloaded. Below is an example of the usage of the fork command.

    $ docker-app fork lucj/tick.dockerapp:0.1.0 myuser/tick --path /tmp/mytickapp

**Generate a HELM chart**

We saw above how easy it is to deploy the application on a Swarm. It’s not much harder to have it ready to be deployed on a [Kubernetes](https://kubernetes.io/) cluster. This happens through the creation of a HELM chart (HELM being the Kubernetes Package Manager). The following command creates the chart:

    $ docker-app helm
    $ tree .
    .
    ├── telegraf.conf
    **├── tick.chart
    │   ├── Chart.yaml
    │   ├── templates
    │   │   └── stack.yaml
    │   └── values.yaml**
    ├── tick.dockerapp
    │   ├── dev.yml
    │   ├── docker-compose.yml
    │   ├── metadata.yml
    │   ├── prod.yml
    │   ├── settings.yml
    │   └── telegraf.conf
    └── tick.yml

As we can see, a folder and a couple of files were generated in the process:

* Charts.yaml contains the project metadata.

    description: ""
    keywords: []
    maintainers:
    - name: luc <>
    name: tick
    version: 0.1.0

* templates/stack.yaml contains the Kubernetes manifest of the application. A special Stack resource is used here.

    **kind: Stack
    **apiVersion: compose.docker.com/v1beta2
    metadata:
      name: tick
      generatename: ""
      namespace: ""
      selflink: ""
      uid: ""
      resourceversion: ""
      generation: 0
      creationtimestamp: "0001–01–01T00:00:00Z"
      deletiontimestamp: null
      deletiongraceperiodseconds: null
      labels: {}
      annotations: {}
      ownerreferences: []
      initializers: null
      finalizers: []
      clustername: 
    spec:
      services:
      — name: chronograf
        command:
        — chronograf
        — --influxdb-url=http://influxdb:8086
        image: chronograf
        ports:
        — mode: ingress
          target: {{.Values.ports.chronograf}}
          published: 8888
          protocol: tcp
      — name: influxdb
        image: influxdb
      — name: kapacitor
        environment:
          KAPACITOR_INFLUXDB_0_URLS_0: http://influxdb:8086
        image: kapacitor
      — name: telegraf
        configs:
        — source: telegraf-conf
          target: /etc/telegraf/telegraf.conf
        image: telegraf
        ports:
        — mode: ingress
          target: {{.Values.ports.telegraf}}
          published: 8186
          protocol: tcp
      configs:
        telegraf-conf:
          file: telegraf.conf

* values.yaml contains the default values that will be used in the placeholders above. As we did not specify any settings file when generating the HELM chart, the values from settings.yml are used.

    ports:
      chronograf: 8081
      telegraf: 8080

Once the helm chart is generated, the application can be deployed on Kube with the same deploy command we used above, but with the additional -o kubernetes flag indicating the orchestrator to use (the default is Swarm).

    $ docker-app deploy -o kubernetes
    Waiting for the stack to be stable and running…
    chronograf: Ready [pod status: 1/1 ready, 0/1 pending, 0/1 failed]
    influxdb: Ready [pod status: 1/1 ready, 0/1 pending, 0/1 failed]
    kapacitor: Ready [pod status: 1/1 ready, 0/1 pending, 0/1 failed]
    telegraf: Ready [pod status: 1/1 ready, 0/1 pending, 0/1 failed]

    Stack tick is stable and running

As we use the default settings, the published ports are 8080 for Telegraf and 8081 for Chronograf* *(values defined in values.yaml)

![](https://cdn-images-1.medium.com/max/2732/1*vlGvc2ADhewHovDbSKZP5g.png)

Note: the deployment on Kubernetes only works on Docker Desktop or Docker Enterprise Edition which run the server side component needed to deal with the **stack** resource.

## Summary

I hope this article provides some insights on the Docker Application Package. The project is still quite young — only a couple of months old at the time of writing — so breaking changes may occur before it reaches 1.0.0. It nevertheless looks really promising, and I’ll follow its evolution in future articles.

Thanks [Gareth Rushgrove](undefined) & Christ Crone for the review, and for pointing out additional resources.
