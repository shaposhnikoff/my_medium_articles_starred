
# Centralized logging in Kubernetes

Elasticsearch, Kibana and Logstash

### What is Centralized logging?

Logging is an essential part of any ecosystem. With logs, we can find answers to,

* What is happening now

* What went wrong

It is essential to monitor the above information to be sure that the system is running as expected and if something has gone wrong, to find **WHAT?** and how to **PREVENT** happening it again.

If you are going to deploy your application in an Environment like Kubernetes managing logging is a task you should put much thought into. **Why is that?**

* A pod is not like a bare-metal machine or a VM. Its lifecycle is managed by Kubernetes cluster. Pods can be restarted, rescheduled at any time. If that is the case how can we find what went wrong?

* There can be hundreds of servers to monitor. Isn’t nice to have a centralized location to monitor all servers?

* Logs mean events which describe the system state. How do gonna find anything efficiently from these thousands of events?

This where Centralized logging system coming to picture. Centralized logging answers all concerns raised above. Basically, Centralized logging system is a single place where your all logs are managed. Each server publishes their logs to a central location and you can use advanced searching techniques, alerting to manage them.

### What are we gonna USE?

There are several solutions in the market addressing centralized logging. For this tutorial, we are going to use,

* [Elasticsearch](https://www.elastic.co/products/elasticsearch) to analyze logs

* [Logstash](https://www.elastic.co/products/logstash) to publish logs

* [Kibana](https://www.elastic.co/products/kibana) provides dashboards

### How can we do it in Kubernetes?

For this setup, we are gonna use a couple of components in the Kubernetes ecosystem. Abstract diagram of the deployment is shown below.

![](https://cdn-images-1.medium.com/max/2000/1*ABimxRMxYVAGoI9lomv0mQ.png)

Let go through each component in detail.

* APIM and Kibana Ingress — Exposes the services to the outside world. Please refer [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) for more detail.

* APIM, Elasticsearch, and Kibana services(SVC) — Exposes the pods within the cluster. Please refer to [Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/) for more details.

* Pod — A single or a set of containers which share the same file and network space. This is the basic unit in Kubernetes. Please refer to [Kubernetes Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/) for more details.

* Init container — Runs before the Application server start. In this case, we have used this component to verify whether the Elasticsearch service is already up and to configure the parameters needed for Elasticsearch. Please refer [Kubernetes Init containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) for more details.

    **initContainers:
    **- **name: **init-wso2-elasticsearch-service
      **image: **busybox
      **command: **['sh', '-c', 'until nc -z wso2-elasticsearch-service 9200; do echo waiting for wso2-elasticsearch-service; sleep 2; done;']

* Sidecar Container — In this scenario, Logstash is running within the same pod along with the Main application, and tails the Main application logs and publishes to Elasticsearch. We are using [WSO2 API Manager ](https://wso2.com/api-management/)which has Named a Leader in The Forrester Wave™: API Management Solutions, Q4 2018 Report as the main application server.

![](https://cdn-images-1.medium.com/max/2000/1*UYzGwwKOp2hFhMqd-xV45A.png)

As shown in the above diagram, a volume(**shared-logs**) is mounted to both WSO2 APIM and Logstash containers. Both servers see the same log file and while the main server writes to the log file, the Logstash, the sidecar container tails and publishes to the Elastic search.

That’s the basic idea of what are we gonna do. Let’s move onto the deployment now.

### Pre-requisites

* [Kubernetes cluster](https://kubernetes.io/docs/setup/pick-right-solution/) with Kubectl configured

* Clone git repository [https://github.com/ThilinaManamgoda/centralized-logging-monitoring.git](https://github.com/ThilinaManamgoda/centralized-logging-monitoring.git)(From now on referred to <Centralized_logging_resources>)

* Logstash docker image with Gork filter and Multiline codec plugins installed.

The first requirement is obvious right 😆. Let me explain a bit more about the last requirement.

WSO2 APIM server is a java application which writes its logs to a file. Logstash is the bridge between Elasticsearch and the Main server. Logstash reads the logs file and converts each event to a structured format(JSON) which Elasticsearch understand and can handle. When it comes to Java logs, it is not as easy as it sounds. There are multiple log lines for error(stack trace) and a single line for simple info. We have to handle both scenarios. [Gork filter plugin](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html) is used to parse loglines and [Multiline codec plugin](https://www.elastic.co/guide/en/logstash/current/plugins-codecs-multiline.html) is used to identify a log event which has multiple log lines.

The default docker image for the Logstash, docker.elastic.co/logstash/logstash:6.5.3 doesn’t have these plugins installed. We have to build our own Lostash Docker image in this case.

    FROM docker.elastic.co/logstash/logstash:6.5.3
    RUN /usr/share/logstash/bin/logstash-plugin install logstash-codec-multiline logstash-filter-grok

You can find above Dockerfile in the<Centralized_logging_resources>/dockerfiles/logstash/Dockerfile. I have already built and pushed this docker image(maanadev/logstash:6.5.3-custom) to the Docker hub. You can use it for this tutorial.

**You can deploy this whole setup by executing the shell script **<Centralized_logging_resources>/deployment/deploy.sh

But I’m gonna explain a bit about certain parts of the deployment below.

The following YAML file(<Centralized_logging_resources>/deployment/centralized-logging-deployment.yaml) contains the Kubernetes deployment for Elasticsearch and Kibana.

    **apiVersion: **extensions/v1beta1
    **kind: **Deployment
    **metadata:
      name: **wso2-elastic-search
    **spec:
      replicas: **1
      **minReadySeconds: **30
      **template:
        metadata:
          labels:
            deployment: **wso2-elastic-search
        **spec:
          initContainers:
          **- **name: **init-sysctl
            **image: **busybox:1.27.2
            **command:
            **- sysctl
            - -w
            - vm.max_map_count=262144
            **securityContext:
              privileged: **true
          **containers:
          **- **name: **wso2-elastic-search
            **image: **docker.elastic.co/elasticsearch/elasticsearch:6.5.3
            **livenessProbe:
              tcpSocket:
                port: **9200
              **initialDelaySeconds: **30
              **periodSeconds: **5
            **readinessProbe:
              httpGet:
                path: **/_cluster/health
                **port: **http
              **initialDelaySeconds: **20
              **periodSeconds: **5
            **imagePullPolicy: **Always
            **resources:
              requests:
                cpu: **0.25
              **limits:
                cpu: **1
            **ports:
            **-
              **containerPort: **9200
              **protocol: **"TCP"
              **name: **http
            -
              **containerPort: **9300
              **protocol: **"TCP"
            **env:
            **- **name: **discovery.type
              **value: **"single-node"
            - **name: **ES_JAVA_OPTS
              **value: **-Xms256m -Xmx256m
            - **name: **network.host
              **valueFrom:
                fieldRef:
                 fieldPath: **status.podIP
            - **name: **PROCESSORS
              **valueFrom:
                resourceFieldRef:
                  resource: **limits.cpu
    ---
    **apiVersion: **v1
    **kind: **Service
    **metadata:
      name: **wso2-elasticsearch-service
    **spec:
      selector:
        deployment: **wso2-elastic-search
      **ports:
        **-
          **name: **http-1
          **protocol: **TCP
          **port: **9200
        -
          **name: **http-2
          **protocol: **TCP
          **port: **9300
    ---
    **apiVersion: **extensions/v1beta1
    **kind: **Deployment
    **metadata:
      name: **wso2-kibana
    **spec:
      replicas: **1
      **minReadySeconds: **30
      **template:
        metadata:
          labels:
            deployment: **wso2-kibana
        **spec:
          initContainers:
          **- **name: **init-wso2-elasticsearch-service
            **image: **busybox
            **command: **['sh', '-c', 'until nslookup wso2-elasticsearch-service; do echo waiting for wso2-elasticsearch-service; sleep 2; done;']
          **containers:
          **- **name: **wso2-kibana
            **image: **docker.elastic.co/kibana/kibana:6.5.3
            **livenessProbe:
              tcpSocket:
                port: **5601
              **initialDelaySeconds: **20
              **periodSeconds: **5
            **readinessProbe:
              httpGet:
                path: **/api/status
                **port: **http
              **initialDelaySeconds: **10
              **periodSeconds: **5
            **imagePullPolicy: **Always
            **ports:
            **-
              **containerPort: **5601
              **protocol: **"TCP"
              **name: **http
            **volumeMounts:
            **- **name: **kibana-yml
              **mountPath: **/usr/share/kibana/config/kibana.yml
              **subPath: **kibana.yml
            **env:
            **- **name: **NODE_IP
              **valueFrom:
                fieldRef:
                 fieldPath: **status.podIP
          **volumes:
          **- **name: **kibana-yml
            **configMap:
              name: **kibana-yml
    ---
    **apiVersion: **v1
    **kind: **Service
    **metadata:
      name: **wso2-kibana-service
    **spec:
      selector:
        deployment: **wso2-kibana
      **ports:
        **-
          **name: **http
          **protocol: **TCP
          **port: **5601
    ---
    **apiVersion: **extensions/v1beta1
    **kind: **Ingress
    **metadata:
      name: **wso2-kibana-ingress
      **annotations:
        kubernetes.io/ingress.class: **"nginx"
    **spec:
      rules:
      **- **host: **wso2-kibana
        **http:
          paths:
          **- **path: **/
            **backend:
              serviceName: **wso2-kibana-service
              **servicePort: **5601

Next the WSO2 APIM and Logstash deployment. Please refer to the YAML <Centralized_logging_resources>/deployment/wso2apim-deployment.yaml

    **apiVersion: **extensions/v1beta1
    **kind: **Deployment
    **metadata:
      name: **wso2apim
    **spec:
      replicas: **1
      **minReadySeconds: **30
      **strategy:
        rollingUpdate:
          maxSurge: **1
          **maxUnavailable: **0
        **type: **RollingUpdate
      **template:
        metadata:
          labels:
            deployment: **wso2apim
        **spec:
          initContainers:
          **- **name: **init-wso2-elasticsearch-service
            **image: **busybox
            **command: **['sh', '-c', 'until nc -z wso2-elasticsearch-service 9200; do echo waiting for wso2-elasticsearch-service; sleep 2; done;']
          **containers:
          **- **name: **wso2apim
            **image: **wso2/wso2am:2.6.0
            **livenessProbe:
              exec:
                command:
                **- /bin/bash
                - -c
                - nc -z localhost 9443
              **initialDelaySeconds: **150
              **periodSeconds: **10
            **readinessProbe:
              exec:
                command:
                  **- /bin/bash
                  - -c
                  - nc -z localhost 9443
              **initialDelaySeconds: **150
              **periodSeconds: **10
            **imagePullPolicy: **Always
            **ports:
            **-
              **containerPort: **8280
              **protocol: **"TCP"
            -
              **containerPort: **8243
              **protocol: **"TCP"
            -
              **containerPort: **9763
              **protocol: **"TCP"
            -
              **containerPort: **9443
              **protocol: **"TCP"
            -
              **containerPort: **5672
              **protocol: **"TCP"
            -
              **containerPort: **9711
              **protocol: **"TCP"
            -
              **containerPort: **9611
              **protocol: **"TCP"
            -
              **containerPort: **7711
              **protocol: **"TCP"
            -
              **containerPort: **7611
              **protocol: **"TCP"
            **volumeMounts:
            **- **name: **shared-logs
              **mountPath: **/home/wso2carbon/wso2am-2.6.0/repository/logs/
          - **name: **logstash
            **image: **maanadev/logstash:6.5.3-custom
            **volumeMounts:
            **- **name: **shared-logs
              **mountPath: **/usr/share/logstash/mylogs/
            - **name: **logstash-yml
              **mountPath: **/usr/share/logstash/config/logstash.yml
              **subPath: **logstash.yml
            - **name: **logstash-conf
              **mountPath: **/usr/share/logstash/pipeline/logstash.conf
              **subPath: **logstash.conf
            **env:
            **- **name: **NODE_ID
              **value: **"wso2-apim"
            - **name: **NODE_IP
              **valueFrom:
                fieldRef:
                 fieldPath: **status.podIP
          **volumes:
          **- **name: **shared-logs
            **emptyDir: **{}
          - **name: **logstash-yml
            **configMap:
              name: **logstash-yml
          - **name: **logstash-conf
            **configMap:
              name: **logstash-conf
    ---
    **apiVersion: **v1
    **kind: **Service
    **metadata:
      name: **wso2apim-service
    **spec:
      ***# label keys and values that must match in order to receive traffic for this service
      ***selector:
        deployment: **wso2apim
      **ports:
        ***# ports that this service should serve on
        *-
          **name: **pass-through-http
          **protocol: **TCP
          **port: **8280
        -
          **name: **pass-through-https
          **protocol: **TCP
          **port: **8243
        -
          **name: **servlet-http
          **protocol: **TCP
          **port: **9763
        -
          **name: **servlet-https
          **protocol: **TCP
          **port: **9443
    ---
    **apiVersion: **extensions/v1beta1
    **kind: **Ingress
    **metadata:
      name: **wso2apim-ingress
      **annotations:
        kubernetes.io/ingress.class: **"nginx"
        **nginx.ingress.kubernetes.io/ssl-passthrough: **"true"
        **nginx.ingress.kubernetes.io/affinity: **"cookie"
        **nginx.ingress.kubernetes.io/session-cookie-name: **"route"
        **nginx.ingress.kubernetes.io/session-cookie-hash: **"sha1"
    **spec:
      tls:
      **- **hosts:
        **- wso2apim
        - wso2apim-gateway
      **rules:
      **- **host: **wso2apim
        **http:
          paths:
          **- **path: **/
            **backend:
              serviceName: **wso2apim-service
              **servicePort: **9443
      - **host: **wso2apim-gateway
        **http:
          paths:
          **- **path: **/
            **backend:
              serviceName: **wso2apim-service
              **servicePort: **8243

If you pay attention to the Config maps mounted to the Logstash, you will be able to see that I have mounted the logstash.conf file.

    input {
        file {
            add_field => {
                instance_name => "${NODE_ID}-${NODE_IP}"
            }
            type => "wso2"
            path => [ '/usr/share/logstash/mylogs/wso2carbon.log' ]
            codec => multiline {
                  pattern => "^TID"
                  negate => true
                  what => "previous"
                }
        }
    }

    filter {
        if [type] == "wso2" {
            grok {
                match => [ "message", "TID:%{SPACE}\[%{INT:tenant_id}\]%{SPACE}\[\]%{SPACE}\[%{TIMESTAMP_ISO8601:timestamp}\]%{SPACE}%{LOGLEVEL:level}%{SPACE}{%{JAVACLASS:java_class}}%{SPACE}-%{SPACE}%{JAVALOGMESSAGE:log_message}%{SPACE}{%{JAVACLASS:java_class_duplicate}}%{GREEDYDATA:stacktrace}" ]
                match => [ "message", "TID:%{SPACE}\[%{INT:tenant_id}\]%{SPACE}\[\]%{SPACE}\[%{TIMESTAMP_ISO8601:timestamp}\]%{SPACE}%{LOGLEVEL:level}%{SPACE}{%{JAVACLASS:java_class}}%{SPACE}-%{SPACE}%{JAVALOGMESSAGE:log_message}%{SPACE}{%{JAVACLASS:java_class_duplicate}}" ]
            }
            date {
                match => [ "timestamp", "ISO8601" ]
              }
        }
    }

    output {
        elasticsearch {
         hosts => ['wso2-elasticsearch-service']
         user => "elastic"
         password => "changeme"
         index => "${NODE_ID}-${NODE_IP}-%{+YYYY.MM.dd}"
        }
    }

With this configuration, I’m telling Logstash,

* How to detect multiline log event

* How to parse a log event. Based on INFO or ERROR.

* How to name the index(${NODE_ID}-${NODE_IP}-%{+YYYY.MM.dd})

* Elastic search endpoint

Next let’s talk a bit about the shared-logs volume,

WSO2 APIM and the Logstash containers run in the same Pod. Where the log files are shared between the containers via the shared volume named **shared-logs **and type [emptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir).

    **volumes:
       **- **name: **shared-logs
         **emptyDir: **{}

### Browsing the logs

If everything went well, you can get the Kibana and APIM Ingress info by running the following command,

kubectl get ing -n wso2

![](https://cdn-images-1.medium.com/max/2320/1*JuDZPHnj6VxHcV9u77_X0g.png)

Get the IP for wso2-kibana hostname and add a new entry in the /etc/hosts file.

    sudo echo “<IP>  wso2-kibana” >> /etc/hosts

* Now goto Kibana Dashboard [http://wso2-kibana](http://wso2-kibana)

* Goto Management dashboard and then Kibana index patterns

![](https://cdn-images-1.medium.com/max/6712/1*BYvBuwxTNFfdt-RYydfKhw.png)

* You should be able to see the existing index pattern as wso2-apim-<IP>–<Date> . Add an index pattern to capture all with wso2-apim-* and click Next step.

![](https://cdn-images-1.medium.com/max/4804/1*l8VWqZhYwA2FVJzzIK-3LQ.png)

* Select timestamp as the time filter filed from the drop-down menu and click create index pattern

![](https://cdn-images-1.medium.com/max/4784/1*Nsn2cQIcp-qsaTNhMfMktQ.png)

* Now go to the Discover tab where you can see log events. If you cannot see any events try adjusting the time range

![](https://cdn-images-1.medium.com/max/6720/1*hlA581M46OIk8TIIR5Ch-Q.png)
