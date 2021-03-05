
# Exporting Kubernetes Logs to Elasticsearch Using Fluent Bit

In a previous tutorial, we discussed how to create a cluster-level logging pipeline using Fluentd log aggregator. As you learned, Fluentd is a powerful log aggregator that supports log collection from multiple sources and sending them to multiple outputs.

In this article, we’ll continue the overview of available logging solutions for Kubernetes focusing on the [Fluent Bit](http://fluentbit.io/). This is another component of the Fluentd project ecosystem made and sponsored by [Treasure Data](https://www.treasuredata.com/). As we’ll show in this article, Fluent Bit is an excellent alternative to Fluentd if your environment has a limited CPU and RAM capacity. This is because Fluent Bit is a very lightweight and performant log shipper and forwarder.

However, these benefits are associated with the trade-off of fewer input and output plugins supported. Therefore, you should still consider using Fluentd as a full log aggregator solution while using Fluent Bit as a log forwarder. This approach is common in other systems like ELK (Elasticsearch-Logstash-Kibana) stack, for example. In the Elasticsearch ecosystem, Logstash is used as a general-purpose log aggregator, while various components of the Beats family (e.g., Metricbeat or Filebeat) are used as lightweight log forwarders.

## Fluentd and Fluent Bit

As we have mentioned, both [Fluentd](http://fluentd.org/) and [Fluent Bit](http://fluentbit.io/) focus on collecting, processing, and delivering logs. However, there are some major differences between the two projects that make them suitable for different tasks.

* **Fluentd combines log collection and processing with *log aggregation***. Fluentd was designed to aggregate logs from multiple inputs, process them, and route to different outputs. Its engine has very performant queue processing threads that enable fast consumption and routing of big batches of logs. In addition, Fluentd has a rich ecosystem of input and output plugins (over 650), which makes it an excellent solution for *log aggregation*.

* **Fluent Bit is great for log collection, processing, and forwarding — but not for log aggregation**. The shipper was designed for running in the highly distributed compute environments where limited capacity and reduced overhead (memory and CPU) are a huge concern. That is why it is very lightweight (~450 KB) and performant (see the image below). The trade-off is that Fluent Bit has support for just 35 input and output plugins.

![](https://cdn-images-1.medium.com/max/2048/0*10eEoUZhsA5lbZ6T.png)

Source: [Fluent Bit documentation](https://fluentbit.io/documentation/0.8/about/fluentd_and_fluentbit.html)

This does not mean, however, that we cannot use Fluent Bit to directly ship logs to output destinations. Fluent Bit has great support for many common [inputs](https://fluentbit.io/documentation/0.12/input/) such as syslog, TCP, systemd, disk, CPU and can also send logs to a number of popular [outputs](https://fluentbit.io/documentation/0.12/output/) such as Elasticsearch, Kafka REST Proxy, and InfluxDB directly.

However, the best choice you can make is to use Fluentd as a Log Aggregator and Fluent Bit as a Log Forwarder. For example, a typical logging pipeline design for Fluentd and Fluent Bit in Kubernetes could be as follows. We could deploy Fluent Bit on each node using a DaemonSet. Each node-level Fluent Bit agent would collect logs and forward them to a single Fluentd instance deployed per cluster and working as a log aggregator — processing logs and routing them to a variety of output destinations.

In this tutorial, we’ll discuss the simplest possible setup — Fluent Bit working as a direct logging pipeline sending logs to Elasticsearch. This is enough to demonstrate how Fluent Bit works.

But before we begin, let’s describe a basic Fluent Bit workflow to give you a basic intuition of the Fluent Bit architecture.

![](https://cdn-images-1.medium.com/max/2048/0*YancwK7Ijgit9IAW.png)

Source: [Fluent Bit Documentation](https://fluentbit.io/documentation/0.8/getting_started/)

The first step of the workflow is taking logs from some input source (e.g., stdout, file, web server). By default, the ingested log data will reside in the Fluent Bit memory until it is routed to some output destination.

Before the logs are routed, they can be also optionally processed using parsers and filters. For example, you can use built-in parsers to convert unstructured data retrieved from the input source into a structured log message. Additionally, you can use various filter plugins to alter the data ingested by the input plugins.

The routing process begins after parsing and filtering are completed. The routing engine matches tags of logs with specific output destinations. The process of sending logs to the output destinations (e.g., Elasticsearch) is in its turn handled by various output plugins. This is very similar to how Logstash or Filebeat work.

Now that you have a basic understanding of the Fluent Bit architecture, we’ll walk you through a process of deploying and configuring Fluent Bit to ship Kubernetes logs to Elasticsearch. We are going to send logs generated by various applications running on Kubernetes to Elasticsearch cluster deployed on Kubernetes.

## Tutorial

To complete the tutorial, you’ll need the following prerequisites:

* A running Kubernetes cluster. See [Supergiant documentation](https://supergiant.readme.io/docs) for more information about deploying a Kubernetes cluster with Supergiant. As an alternative, you can install a single-node Kubernetes cluster on a local system using [Minikube](https://kubernetes.io/docs/getting-started-guides/minikube/).

* A **kubectl** command line tool installed and configured to communicate with the cluster. See how to install **kubectl** [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

* A running Elasticsearch deployed in your Kubernetes cluster. For the simplest way to deploy Elasticsearch in Kubernetes, you can consult [this article](https://chekkan.com/setting-up-elasticsearch-cluster-on-kubernetes-part-1/).

## Step 1: Create RBAC for the Fluent Bit

First, let’s isolate our future Fluent Bit deployment from the rest of the cluster by creating a new namespace.

    kubectl create namespace fluentbit-test

    namespace "fluentbit-test" created

To collect logs from Kubernetes applications and cluster components, we need to provide identity to Fluent Bit and grant it some permissions. For the former we will create a new Service Account in the fluentbit-test namespace where the Fluent Bit will be deployed:

    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: fluent-bit
      namespace: fluentbit-test

Fluent Bit needs permissions to get, list, and watch namespaces and Pods in your Kubernetes cluster. These can be granted using the ClusterRole manifest as in the example below:

    apiVersion: rbac.authorization.k8s.io/v1beta1
    kind: ClusterRole
    metadata:
      name: fluent-bit-read
    rules:
    - apiGroups: [""]
      resources:
      - namespaces
      - pods
      verbs: ["get", "list", "watch"]

Finally, we need to bind the Fluent Bit ServiceAccount to the ClusterRole using the ClusterRoleBinding resource.

    apiVersion: rbac.authorization.k8s.io/v1beta1
    kind: ClusterRoleBinding
    metadata:
      name: fluent-bit-read
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: fluent-bit-read
    subjects:
    - kind: ServiceAccount
      name: fluent-bit
      namespace: fluentbit-test

**Note**: please don’t forget to specify the correct namespace for the ClusterRoleBinding .

Let’s save these manifests in the file separating them by the —-- delimiter and create all resources in bulk:

    kubectl create -f rbac.yml

    serviceaccount "fluent-bit" created
    clusterrole.rbac.authorization.k8s.io "fluent-bit-read" created
    clusterrolebinding.rbac.authorization.k8s.io "fluent-bit-read" created

## Step 2: Create a ConfigMap

We need to configure Fluent Bit before deploying it as a DaemonSet. Fluent Bit has a unique configuration syntax different from Fluentd. For more information about configuring Fluent Bit, please, consult the [official documentation.](https://docs.fluentbit.io/manual/configuration)

The Fluent Bit configuration file consists of sections and key-value entries inside those sections. Each section is defined by a name or a title placed inside brackets.

    [SERVICE]
        HTTP_Listen   0.0.0.0
        HTTP_Port     2020

For example, above we defined a SERVICE section holding two entries: HTTP_Listen and HTTP_Port .

Each entry is defined by a line of text that contains a Key and a Value. Referring to the example above, the **[SERVICE]** section contains two entries: one is the key HTTP_Listen with the value 0.0.0.0 , and the other is the key HTTP_Port with the value 2020 . As you see, entries have the indentation level of four spaces (this is the ideal indentation for Fluent Bit configuration).

Fluent Bit supports four types of sections:

* [Service](https://docs.fluentbit.io/manual/configuration/file#config_section)

* [Input](https://docs.fluentbit.io/manual/configuration/file#config_input)

* [Filter](https://docs.fluentbit.io/manual/configuration/file#config_filter)

* [Output](https://docs.fluentbit.io/manual/configuration/file#config_output)

We’ll explain these types in a moment using the example ConfigMap for our Fluent Bit DaemonSet. Take a look at the example configuration:

    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: fluent-bit-config
      namespace: fluentbit-test
      labels:
        k8s-app: fluent-bit
    data:
      # Configuration files: server, input, filters and output
      # ======================================================
      fluent-bit.conf: |
        [SERVICE]
            Flush         2
            Log_Level     info
            Daemon        off
            Parsers_File  parsers.conf
            HTTP_Server   On
            HTTP_Listen   0.0.0.0
            HTTP_Port     2020
     
        [@INCLUDE](http://twitter.com/INCLUDE) input-kubernetes.conf
        [@INCLUDE](http://twitter.com/INCLUDE) filter-kubernetes.conf
        [@INCLUDE](http://twitter.com/INCLUDE) output-elasticsearch.conf
     
      input-kubernetes.conf: |
        [INPUT]
            Name              tail
            Tag               kube.*
            Path              /var/log/containers/*.log
            Parser            docker
            DB                /var/log/flb_kube.db
            Mem_Buf_Limit     5MB
            Skip_Long_Lines   On
            Refresh_Interval  10
     
      filter-kubernetes.conf: |
        [FILTER]
            Name                kubernetes 
            Match               kube.*
            Kube_URL            [https://kubernetes.default.svc.cluster.local:443](https://kubernetes.default.svc.cluster.local:443)
            Merge_Log           Off
            K8S-Logging.Parser  On
     
      output-elasticsearch.conf: |
        [OUTPUT]
            Name            es
            Match           *
            Host            ${FLUENT_ELASTICSEARCH_HOST}
            Port            ${FLUENT_ELASTICSEARCH_PORT}
            HTTP_User       ${FLUENT_ELASTICSEARCH_USER}
            HTTP_Passwd     ${FLUENT_ELASTICSEARCH_PASSWORD}
            Logstash_Format On
            Retry_Limit     False
     
      parsers.conf: |
        [PARSER]
            Name   apache
            Format regex
            Regex  ^(?<host>[^ ]*) [^ ]* (?<user>[^ ]*) \[(?<time>[^\]]*)\] "(?<method>\S+)(?: +(?<path>[^\"]*?)(?: +\S*)?)?" (?<code>[^ ]*) (?<size>[^ ]*)(?: "(?<referer>[^\"]*)" "(?<agent>[^\"]*)")?$
            Time_Key time
            Time_Format %d/%b/%Y:%H:%M:%S %z
     
        [PARSER]
            Name   apache2
            Format regex
            Regex  ^(?<host>[^ ]*) [^ ]* (?<user>[^ ]*) \[(?<time>[^\]]*)\] "(?<method>\S+)(?: +(?<path>[^ ]*) +\S*)?" (?<code>[^ ]*) (?<size>[^ ]*)(?: "(?<referer>[^\"]*)" "(?<agent>[^\"]*)")?$
            Time_Key time
            Time_Format %d/%b/%Y:%H:%M:%S %z
     
        [PARSER]
            Name   apache_error
            Format regex
            Regex  ^\[[^ ]* (?<time>[^\]]*)\] \[(?<level>[^\]]*)\](?: \[pid (?<pid>[^\]]*)\])?( \[client (?<client>[^\]]*)\])? (?<message>.*)$
     
        [PARSER]
            Name   nginx
            Format regex
            Regex ^(?<remote>[^ ]*) (?<host>[^ ]*) (?<user>[^ ]*) \[(?<time>[^\]]*)\] "(?<method>\S+)(?: +(?<path>[^\"]*?)(?: +\S*)?)?" (?<code>[^ ]*) (?<size>[^ ]*)(?: "(?<referer>[^\"]*)" "(?<agent>[^\"]*)")?$
            Time_Key time
            Time_Format %d/%b/%Y:%H:%M:%S %z
     
        [PARSER]
            Name   json
            Format json
            Time_Key time
            Time_Format %d/%b/%Y:%H:%M:%S %z
     
        [PARSER]
            Name        docker
            Format      json
            Time_Key    time
            Time_Format %Y-%m-%dT%H:%M:%S.%L
            Time_Keep   On
            # Command      |  Decoder | Field | Optional Action
            # =============|==================|=================
            Decode_Field_As   escaped    log
     
        [PARSER]
            Name        syslog
            Format      regex
            Regex       ^\<(?<pri>[0-9]+)\>(?<time>[^ ]* {1,2}[^ ]* [^ ]*) (?<host>[^ ]*) (?<ident>[a-zA-Z0-9_\/\.\-]*)(?:\[(?<pid>[0-9]+)\])?(?:[^\:]*\:)? *(?<message>.*)$
            Time_Key    time
            Time_Format %b %d %H:%M:%S

The Service section above defines the global properties of the Fluent Bit service. The configuration for this section is stored in the main fluent-bit.conf file. We use INCLUDE directive to include the configuration of inputs, filters, and outputs in this main file.

There are several parameters of the Service section worth of your attention:

* **Flush** — specifies how often (in seconds) the Fluent Bit engine flushes log records to the output plugin.

* **Daemon** — is a Boolean value that allows running Fluent Bit instance as a background process (Daemon).

* **Log_Level** — sets the logging verbosity level. The allowed values are error, info, debug, and trace.

* **HTTP_Server** — tells Fluent Bit to use a built-in HTTP Server.

* **HTTP_Listen** — sets a listening interface for HTTP Server if it’s enabled (default is 0.0.0.0 )

* **HTTP_Port** — sets a TCP Port for the HTTP Server

After configuring global settings, we need to include some inputs — sources where Fluent Bit watches for logs. These are defined in the INPUT section.

In the configuration above, we use the tail input plugin that allows monitoring one or several text files. The plugin has a functionality similar to the tail -f shell command. It reads every matched file in the path pattern and for every new line found generates a new record.

Fluent Bit has built-in support for other input sources such as:

* [cpu](https://docs.fluentbit.io/manual/input/cpu) — measures total CPU usage of the system.

* [disk](https://docs.fluentbit.io/manual/input/disk) — measures Disk I/Os.

* [exec](https://docs.fluentbit.io/manual/input/exec) — executes external programs and collects event logs.

* [forward](https://docs.fluentbit.io/manual/input/forward) — Fluentd forward protocol. This plugin can be used to forward logs collected by Fluent Bit to Fluentd for aggregation.

* [head](https://docs.fluentbit.io/manual/input/head) — reads first part of files.

* [proc](https://docs.fluentbit.io/manual/input/proc) — checks health of processes.

* [syslog](https://docs.fluentbit.io/manual/input/syslog) — reads syslog messages from a Unix socket.

For a full list of supported plugins, consult the official documentation [here](https://docs.fluentbit.io/manual/input).

Each input section has general and plugin-specific configuration options. Let’s discuss those specified in the Input configuration above:

* **Tag** — you can associate all records coming from the input with a specific tag. For example, we associated all records with the kube.* pattern.

* **Path** — tail plugin requires a path or path pattern to a log file/s. Our Fluent Bit instance will be watching for all container log files that are stored under /var/log/containers/*.log in Kubernetes.

* **Parser** — specifies the name of a parser to interpret the entry as a structured message (e.g., Docker).

* **DB** — the database file to keep track of the Fluent Bit position in the monitored files.

* **Mem_Buf_Limit** — defines a memory limit the tail plugin can use before the records are flushed to the output. If the limit is reached, the tail plugin will pause collecting the records until they are flushed.

* **Refresh_Interval** — the interval for refreshing a list of watched files. The default value is 60 seconds.

In the Filter section, we define the filter plugin(s) to process the collected logs. Since we are working with Kubernetes apps, we are using the [Kubernetes filter plugin](https://docs.fluentbit.io/manual/filter/kubernetes).

Kubernetes filter performs the following operations:

* Analyzes the data and extracts the metadata such as Pod name, namespace, container name, and container ID (this is quite similar to what Fluentd does).

* Queries Kubernetes API server to get extra metadata for the given Pod including the Pod ID, labels, and annotations. This metadata is then appended to each record (log message).

This data is cached locally in memory and is appended to each log record. The following parameters represent a minimum configuration for this filter used in the ConfigMap above:

* **Name** — the name of the filter plugin.

* **Kube_URL** — API Server end-point. E.g [https://kubernetes.default.svc.cluster.local/](https://kubernetes.default.svc.cluster.local/)

* **Match** — a tag to match filtering against.

The next crucial part of the Fluent Bit configuration is specifying the output plugin. The output plugin defines the destination to which Fluent Bit should flush the logs it collects from the input. Each output plugin has its specific configuration options. The Elasticsearch output we use has the following options:

* **Host** — Elasticsearch host.

* **Port** — Elasticsearch port.

* **HTTP_User** — Elasticsearch username if your cluster has authentication.

* **HTTP_Passwd** — A password for user defined in HTTP_User

* **Logstash_Format** — Enable Logstash format compatibility. This option takes a boolean value: True/False, On/Off

Along with inputs, filters, and outputs, we define seven parsers for common applications logs and input types. As we’ve mentioned, parsers allow transforming unstructured log data into structured form making them easier to process and filter.

Fluent Bit parsers can process log entries based on two types of formats: JSON Maps and Regular Expressions. All parsers must be defined in the parsers.conf file. By default, Fluent Bit ships with the pre-configured parsers for:

* Apache

* Nginx

* Docker

* Syslog rfc5424

* Syslog rfc3164

Let’s look at one of the parsers defined above to understand available configuration options:

    [PARSER]
    Name   nginx
    Format regex
    Regex ^(?<remote> ) (?<host> ) (?<user> ) [(?<time>])] "(?<method>\S+)(?: +(?<path>"?)(?: +\S)?)?" (?<code> ) (?<size> )(?: "(?<referer>")" "(?<agent>")")?$
    Time_Key time
    Time_Format %d/%b/%Y:%H:%M:%S %z

This configuration defines a Nginx parser. The most important configuration entries are the following:

* **Format** — the format of the parser. We use the regex format for Nginx parser.

* **Regex** — the Ruby Regular Expression used to parse and compose the structured message. We can use built-in Fluent Bit regex variables like <remote>, <host>, <time>, <method>

* **Time_Key** — If a log entry includes a field with a timestamp, you can use this option to specify the name of this field.

* **Time_Format** — Select the format of the time field so it can be properly recognized and analyzed. Fluent Bit uses strptime(3) to parse time so you can refer to [strptime documentation](https://linux.die.net/man/3/strptime) for available modifiers.

Great! Now that you understand key configuration options, let’s create a ConfigMap .

    kubectl create -f fluent-config.yml

    configmap "fluent-bit-config" created

## Step 3: Deploy Fluent Bit on Minikube

Now, we are ready to create a Fluent Bit DeamonSet using this ConfigMap. Take a look at the DaemonSet manifest we use:

    apiVersion: extensions/v1beta1
    kind: DaemonSet
    metadata:
      name: fluent-bit
      namespace: fluentbit-test
      labels:
        k8s-app: fluent-bit-logging
        version: v1
        kubernetes.io/cluster-service: "true"
    spec:
      template:
        metadata:
          labels:
            k8s-app: fluent-bit-logging
            version: v1
            kubernetes.io/cluster-service: "true"
          annotations:
            prometheus.io/scrape: "true"
            prometheus.io/port: "2020"
            prometheus.io/path: /api/v1/metrics/prometheus
        spec:
          containers:
          - name: fluent-bit
            image: fluent/fluent-bit:0.14.2
            imagePullPolicy: Always
            ports:
            - containerPort: 2020
            env:
            - name: FLUENT_ELASTICSEARCH_HOST
              value: "elasticsearch.default.svc.cluster.local"
            - name: FLUENT_ELASTICSEARCH_PORT 
              value: "9200"
            volumeMounts:
            - name: varlog
              mountPath: /var/log
            - name: varlibdockercontainers
              mountPath: /var/lib/docker/containers
              readOnly: true
            - name: fluent-bit-config
              mountPath: /fluent-bit/etc/
            - name: mnt
              mountPath: /mnt
              readOnly: true
          terminationGracePeriodSeconds: 10
          volumes:
          - name: varlog
            hostPath:
              path: /var/log
          - name: varlibdockercontainers
            hostPath:
              path: /var/lib/docker/containers
          - name: fluent-bit-config
            configMap:
              name: fluent-bit-config
          - name: mnt
            hostPath:
              path: /mnt
          serviceAccountName: fluent-bit
          tolerations:
          - key: node-role.kubernetes.io/master
            operator: Exists
            effect: NoSchedule

Before deploying this DaemonSet, please, don’t forget to specify the environmental variables values for your Elasticsearch host, port, and any credentials if needed. In the example above, we use the DNS record of the Elasticsearch service as the value for Elasticsearch host and Elasticsearch default port 9200. These values will be mapped to placeholders we used in the ConfigMap.

Let’s save this manifest in the fluentbit-deploy.yml and create the DaemonSet with the following command:

    kubectl create -f fluentbit-deploy.yml

    daemonset.extensions "fluent-bit" created

Let’s now check the Fluent Bit logs to verify that everything has worked out correctly. First, find the Fluentbit Pod in the “fluentbit-test” namespace.

    kubectl get pods --namespace=fluentbit-test

    NAME               READY     STATUS    RESTARTS   AGE

    fluent-bit-5fwcg   1/1       Running   1          7h

Then, run kubectl logs with the name of the Fluent Bit Pod:

    kubectl logs fluent-bit-5fwcg

    [2019/01/08 11:38:35] [ info] [engine] started (pid=1)

    [2019/01/08 11:38:35] [debug] [in_tail] inotify watch fd=20

    [2019/01/08 11:38:35] [debug] [in_tail] scanning path /var/log/containers/*.log

    [2019/01/08 11:38:35] [debug] [in_tail] add to scan queue /var/log/containers/elasticsearch-5d8b4974b6-dfcdm_default_elasticsearch-556a125605d123f9498dcbe67db2cf87053b4a0a116d576480815db373a7d8b3.log, offset=0

    [2019/01/08 11:38:35] [debug] [in_tail] add to scan queue /var/log/containers/elasticsearch-5d8b4974b6-dfcdm_default_elasticsearch-b26ed072ed625ed5e237e12fc549f494302cc68649141b5bc6c84cc84f667824.log, offset=76798

    [2019/01/08 11:38:35] [debug] [in_tail] add to scan queue /var/log/containers/etcd-minikube_kube-system_etcd-0dbef5482177a77a560ef042005aed9ab1ea0e0e8d5a137a87dc31d26236321a.log, offset=6869

These logs indicate that the Fluent Bit was successfully started and tail plugin began adding specified paths to its queue.

Let’s check if the logs were actually shipped to Elasticsearch as we expect. Assuming that you have Elasticsearch exposed as a Service, run

    minikube service elasticsearch --url

    http://192.168.99.100:30775

to retrieve the IP and port assigned to Elasticsearch.

Then, you can use the IP and port to cURL various Elasticsearch endpoints. For example, to check if the Elasticsearch is running:

    curl [http://192.168.99.100:30775](http://192.168.99.100:30775)
    {
      "name" : "_4EdnBi",
      "cluster_name" : "docker-cluster",
      "cluster_uuid" : "vMv_vCnuSOKLRrBK2522kw",
      "version" : {
        "number" : "6.2.1",
        "build_hash" : "7299dc3",
        "build_date" : "2018-02-07T19:34:26.990113Z",
        "build_snapshot" : false,
        "lucene_version" : "7.2.1",
        "minimum_wire_compatibility_version" : "5.6.0",
        "minimum_index_compatibility_version" : "5.0.0"
      },
      "tagline" : "You Know, for Search"
    }

Now, let’s check if the Fluent Bit has sent any logs to Elasticsearch. First, find the available indices:

    curl http://192.168.99.100:30775/_cat/indices

    green  open .monitoring-es-6-2019.01.08 oPYvNzxcRDiA26g-frWhlA 1 0    2244 66     2mb     2mb

    yellow open logstash-2019.01.08         MsvQMg8NQ1miOoOIRf8T9g 5 1 4652743  0 758.7mb 758.7mb

As you see, Fluent Bit has added the .monitoring-es-6–2019.01.08 index that contains Elasticsearch logs from the Elasticsearch instance we deployed. Let’s get some documents from this index:

    curl -X GET "[http://192.168.99.100:30775/.monitoring-es-6-2019.01.08/_search?pretty](http://192.168.99.100:30775/.monitoring-es-6-2019.01.08/_search?pretty)" -H 'Content-Type: application/json' -d'
    {             
        "query": {
            "match_all": {}
        }
    }
    '
    {
      "took" : 192,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : 634,
        "max_score" : 1.0,
        "hits" : [
          {
            "_index" : ".monitoring-es-6-2019.01.08",
            "_type" : "doc",
            "_id" : "JSJELWgBa3L_JOpgKD7W",
            "_score" : 1.0,
            "_source" : {
              "cluster_uuid" : "vMv_vCnuSOKLRrBK2522kw",
              "timestamp" : "2019-01-08T11:41:08.141Z",
              "interval_ms" : 10000,
              "type" : "node_stats",
              "source_node" : {
                "uuid" : "_4EdnBi5TACioRNsfX916w",
                "host" : "172.17.0.5",
                "transport_address" : "172.17.0.5:9300",
                "ip" : "172.17.0.5",
                "name" : "_4EdnBi",
                "timestamp" : "2019-01-08T11:41:08.082Z"
              },
              "node_stats" : {
                "node_id" : "_4EdnBi5TACioRNsfX916w",
                "node_master" : true,
                "mlockall" : false,
                "indices" : {
                  "docs" : {
                    "count" : 26288
                  },
                  "store" : {
                    "size_in_bytes" : 8014764
                  },
                  "indexing" : {
                    "index_total" : 26301,
                    "index_time_in_millis" : 96763,
                    "throttle_time_in_millis" : 0
                  },
                  "search" : {
                    "query_total" : 0,
                    "query_time_in_millis" : 0
                  },
                  "query_cache" : {
                    "memory_size_in_bytes" : 0,
                    "hit_count" : 0,
                    "miss_count" : 0,
                    "evictions" : 0
                  },
                  "fielddata" : {
                    "memory_size_in_bytes" : 0,
                    "evictions" : 0
                  },
                  "segments" : {
                    "count" : 27,
                    "memory_in_bytes" : 262381,
                    "terms_memory_in_bytes" : 218206,
                    "stored_fields_memory_in_bytes" : 11488,
                    "term_vectors_memory_in_bytes" : 0,
                    "norms_memory_in_bytes" : 22848,
                    "points_memory_in_bytes" : 2371,
                    "doc_values_memory_in_bytes" : 7468,
                    "index_writer_memory_in_bytes" : 0,
                    "version_map_memory_in_bytes" : 0,
                    "fixed_bit_set_memory_in_bytes" : 0
                  },
                  "request_cache" : {
                    "memory_size_in_bytes" : 0,
                    "evictions" : 0,
                    "hit_count" : 0,
                    "miss_count" : 0
                  }
                },
                "os" : {
                  "cpu" : {
                    "load_average" : {
                      "1m" : 8.9,
                      "5m" : 3.73,
                      "15m" : 1.44
                    }
                  },
                  "cgroup" : {
                    "cpuacct" : {
                      "control_group" : "/",
                      "usage_nanos" : 63699289375
                    },
                    "cpu" : {
                      "control_group" : "/",
                      "cfs_period_micros" : 100000,
                      "cfs_quota_micros" : -1,
                      "stat" : {
                        "number_of_elapsed_periods" : 0,
                        "number_of_times_throttled" : 0,
                        "time_throttled_nanos" : 0
                      }
                    },
                    "memory" : {
                      "control_group" : "/",
                      "limit_in_bytes" : "9223372036854771712",
                      "usage_in_bytes" : "498839552"
                    }
                  }
                },
                "process" : {
                  "open_file_descriptors" : 216,
                  "max_file_descriptors" : 1048576,
                  "cpu" : {
                    "percent" : 33
                  }
                },
                "jvm" : {
                  "mem" : {
                    "heap_used_in_bytes" : 358350200,
                    "heap_used_percent" : 33,
                    "heap_max_in_bytes" : 1056309248
                  },
                  "gc" : {
                    "collectors" : {
                      "young" : {
                        "collection_count" : 35,
                        "collection_time_in_millis" : 23077
                      },
                      "old" : {
                        "collection_count" : 2,
                        "collection_time_in_millis" : 129
                      }
                    }
                  }
                },
                "thread_pool" : {
                  "bulk" : {
                    "threads" : 2,
                    "queue" : 0,
                    "rejected" : 0
                  },
                  "generic" : {
                    "threads" : 4,
                    "queue" : 0,
                    "rejected" : 0
                  },
                  "get" : {
                    "threads" : 0,
                    "queue" : 0,
                    "rejected" : 0
                  },
                  "index" : {
                    "threads" : 0,
                    "queue" : 0,
                    "rejected" : 0
                  },
                  "management" : {
                    "threads" : 3,
                    "queue" : 0,
                    "rejected" : 0
                  },
                  "search" : {
                    "threads" : 0,
                    "queue" : 0,
                    "rejected" : 0
                  },
                  "watcher" : {
                    "threads" : 0,
                    "queue" : 0,
                    "rejected" : 0
                  }
                },
                "fs" : {
                  "total" : {
                    "total_in_bytes" : 17293533184,
                    "free_in_bytes" : 14133542912,
                    "available_in_bytes" : 13120671744
                  }
                }
              }
            }
          }
         ]
       }
    }

Awesome! As you see, Fluent Bit has been successful in tailing Elasticsearch log files and sending generated logs to the Elasticsearch index.

## Conclusion

That’s it! You’ve learned how to deploy Fluent Bit to a Kubernetes cluster and how to ship Kubernetes applications and components logs to Elasticsearch. Fluent Bit is a lightweight and performant log shipper that has a functionality similar to Fluentd. However, because it supports less input and output plugins than Fluentd and is less powerful in accumulating logs from multiple locations, it’s not as good for log aggregation than Fluentd.

In upcoming tutorials, we’ll discuss how to combine both Fluentd and Fluent Bit to create a centralized logging pipeline for your Kubernetes cluster. Stay tuned to the Supergiant blog to learn more!

*Originally published at [https://supergiant.io](https://supergiant.io/blog/exporting-kubernetes-logs-to-elasticsearch-using-fluent-bit/).*
