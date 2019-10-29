Unknown markup type 10 { type: [33m10[39m, start: [33m53[39m, end: [33m63[39m }

# Configuring HA Kubernetes cluster on bare metal servers, monitoring, logs and usage examples. 3/3



Hi there and welcome back to the third part of the “Kubernetes on bare metal” tutorial, in this part I wanna pay attention to the cluster monitoring & collecting logs, also we’ll run some test application for utilizing previously configured cluster components. After all, we will implement some stress tests and will check the stability of this cluster schema.

A first and very popular tool that Kubernetes community can provide for getting some web UI and statistical data about your cluster, it’s [**Kubernetes Dashboard](https://github.com/kubernetes/dashboard)**. In fact, it still under developing, but even now it can provide some extra data for troubleshooting applications and managing the cluster resources also.

It’s a bit controversial theme, is it really need to have any WEB UI for the cluster management, or using a console **kubectl** tool is quite enough. Well, sometimes it may be useful and complement each other.

Let’s deploy our own **Kubernetes Dashboard** and see then. The standard deployment will run this dashboard on localhost address only, so you’ll need to use the **kubectl proxy** command to expose it, but it’ll be still available only on your kubectl control machine locally. This is not bad for security reasons, but I like to have access to it outside the cluster in my browser and I ready to take some risks (anyway it uses SSL with the strong token).

To use my way, you need to change a bit standard deployment file in the service section. We’ll use our load balancer for exposing this dashboard on a public address.

Login to the machine that has configured **kubectl** utility and create:

    **control#** vi kube-dashboard.yaml

    # Copyright 2017 The Kubernetes Authors. 
    # 
    # Licensed under the Apache License, Version 2.0 (the "License"); 
    # you may not use this file except in compliance with the License. 
    # You may obtain a copy of the License at 
    # 
    #     http://www.apache.org/licenses/LICENSE-2.0 
    # 
    # Unless required by applicable law or agreed to in writing, software 
    # distributed under the License is distributed on an "AS IS" BASIS, 
    # WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. 
    # See the License for the specific language governing permissions and 
    # limitations under the License. 
     
    # ------------------- Dashboard Secret ------------------- # 
     
    apiVersion: v1 
    kind: Secret 
    metadata: 
      labels: 
        k8s-app: kubernetes-dashboard 
      name: kubernetes-dashboard-certs 
      namespace: kube-system 
    type: Opaque 
     
    --- 
    # ------------------- Dashboard Service Account ------------------- # 
     
    apiVersion: v1 
    kind: ServiceAccount 
    metadata: 
      labels: 
        k8s-app: kubernetes-dashboard 
      name: kubernetes-dashboard 
      namespace: kube-system 
     
    --- 
    # ------------------- Dashboard Role & Role Binding ------------------- # 
     
    kind: Role 
    apiVersion: rbac.authorization.k8s.io/v1 
    metadata: 
      name: kubernetes-dashboard-minimal 
      namespace: kube-system 
    rules: 
      # Allow Dashboard to create 'kubernetes-dashboard-key-holder' secret. 
    - apiGroups: [""] 
      resources: ["secrets"] 
      verbs: ["create"] 
      # Allow Dashboard to create 'kubernetes-dashboard-settings' config map. 
    - apiGroups: [""] 
      resources: ["configmaps"] 
      verbs: ["create"] 
      # Allow Dashboard to get, update and delete Dashboard exclusive secrets. 
    - apiGroups: [""] 
      resources: ["secrets"] 
      resourceNames: ["kubernetes-dashboard-key-holder", "kubernetes-dashboard-certs"] 
      verbs: ["get", "update", "delete"] 
      # Allow Dashboard to get and update 'kubernetes-dashboard-settings' config map. 
    - apiGroups: [""] 
      resources: ["configmaps"] 
      resourceNames: ["kubernetes-dashboard-settings"] 
      verbs: ["get", "update"] 
      # Allow Dashboard to get metrics from heapster. 
    - apiGroups: [""] 
      resources: ["services"] 
      resourceNames: ["heapster"] 
      verbs: ["proxy"] 
    - apiGroups: [""] 
      resources: ["services/proxy"] 
      resourceNames: ["heapster", "http:heapster:", "https:heapster:"] 
      verbs: ["get"] 
     
    --- 
    apiVersion: rbac.authorization.k8s.io/v1 
    kind: RoleBinding 
    metadata: 
      name: kubernetes-dashboard-minimal 
      namespace: kube-system 
    roleRef: 
      apiGroup: rbac.authorization.k8s.io 
      kind: Role 
      name: kubernetes-dashboard-minimal 
    subjects: 
    - kind: ServiceAccount 
      name: kubernetes-dashboard 
      namespace: kube-system 
     
    --- 
    # ------------------- Dashboard Deployment ------------------- # 
     
    kind: Deployment 
    apiVersion: apps/v1 
    metadata: 
      labels: 
        k8s-app: kubernetes-dashboard 
      name: kubernetes-dashboard 
      namespace: kube-system 
    spec: 
      replicas: 1 
      revisionHistoryLimit: 10 
      selector: 
        matchLabels: 
          k8s-app: kubernetes-dashboard 
      template: 
        metadata: 
          labels: 
            k8s-app: kubernetes-dashboard 
        spec: 
          containers: 
          - name: kubernetes-dashboard 
            image: k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1 
            ports: 
            - containerPort: 8443 
              protocol: TCP 
            args: 
              - --auto-generate-certificates 
              # Uncomment the following line to manually specify Kubernetes API server Host 
              # If not specified, Dashboard will attempt to auto discover the API server and connect 
              # to it. Uncomment only if the default does not work. 
              # - --apiserver-host=http://my-address:port 
            volumeMounts: 
            - name: kubernetes-dashboard-certs 
              mountPath: /certs 
              # Create on-disk volume to store exec logs 
            - mountPath: /tmp 
              name: tmp-volume 
            livenessProbe: 
              httpGet: 
                scheme: HTTPS 
                path: / 
                port: 8443 
              initialDelaySeconds: 30 
              timeoutSeconds: 30 
          volumes: 
          - name: kubernetes-dashboard-certs 
            secret: 
              secretName: kubernetes-dashboard-certs 
          - name: tmp-volume 
            emptyDir: {} 
          serviceAccountName: kubernetes-dashboard 
          # Comment the following tolerations if Dashboard must not be deployed on master 
          tolerations: 
          - key: node-role.kubernetes.io/master 
            effect: NoSchedule 
     
    --- 
    # ------------------- Dashboard Service ------------------- # 
     
    kind: Service 
    apiVersion: v1 
    metadata: 
      labels: 
        k8s-app: kubernetes-dashboard 
      name: kubernetes-dashboard 
      namespace: kube-system 
    spec: 
      type: LoadBalancer 
      ports: 
        - port: 443 
          targetPort: 8443 
      selector: 
        k8s-app: kubernetes-dashboard

Then run it:

    **control#** kubectl create -f kube-dashboard.yaml

    **control# **kubectl get svc --namespace=kube-system

    kubernetes-dashboard   LoadBalancer   10.96.164.141 192.168.0.240 443:31262/TCP   8h

Great, as you can see our LB added the 192.168.0.240 IP for this service, now you can try open [https://192.168.0.240](https://192.168.0.240) to see the Kubernetes Dashboard.

![](https://cdn-images-1.medium.com/max/2108/1*WACfyAolr26vwZFggzL1EA.png)

We can use two ways to get access in to it, using theadmin.conf file from our master node, that we previously used for configuring the kubectl or by creating a specific Service Account for it, with the security token.

Let's create some admin user then:

    **control#** vi kube-dashboard-admin.yaml

    apiVersion: v1 
    kind: ServiceAccount 
    metadata: 
      name: admin-user 
      namespace: kube-system 
    --- 
    apiVersion: rbac.authorization.k8s.io/v1beta1 
    kind: ClusterRoleBinding 
    metadata: 
      name: admin-user 
    roleRef: 
      apiGroup: rbac.authorization.k8s.io 
      kind: ClusterRole 
      name: cluster-admin 
    subjects: 
    - kind: ServiceAccount 
      name: admin-user 
      namespace: kube-system

    **
    control# **kubectl create -f kube-dashboard-admin.yaml

    serviceaccount/admin-user created 
    clusterrolebinding.rbac.authorization.k8s.io/admin-user created

Then we need to get token we can use to log in:

    **control# **kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')

    
    Name:         admin-user-token-vfh66 
    Namespace:    kube-system 
    Labels:       <none> 
    Annotations:  kubernetes.io/service-account.name: admin-user 
                  kubernetes.io/service-account.uid: 3775471a-3620-11e9-9800-763fc8adcb06 
     
    Type:  kubernetes.io/service-account-token 
     
    Data 
    ==== 
    ca.crt:     1025 bytes 
    namespace:  11 bytes 
    token:      erJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwna3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJr
    dWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VmLXRva2VuLXZmaDY2Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZ
    XJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIzNzc1NDcxYS0zNjIwLTExZTktOTgwMC03Nj
    NmYzhhZGNiMDYiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.JICASwxAJHFX8mLoSikJU1tbij4Kq2pneqAt6QCcGUizFLeSqr2R5x339ZR8W4
    9cIsbZ7hbhFXCATQcVuWnWXe2dgXP5KE8ZdW9uvq96rm_JvsZz0RZO03UFRf8Exsss6GLeRJ5uNJNCAr8No5pmRMJo-_4BKW4OFDFxvSDSS_ZJaLMqJ0LNpwH1Z09SfD8TNW7VZqax4zuTSMX_yVS
    ts40nzh4-_IxDZ1i7imnNSYPQa_Oq9ieJ56Q-xuOiGu9C3Hs3NmhwV8MNAcniVEzoDyFmx4z9YYcFPCDIoerPfSJIMFIWXcNlUTPSMRA-KfjSb_KYAErVfNctwOVglgCISA

Now copy the token and paste it into the Token field on the login screen.

![](https://cdn-images-1.medium.com/max/2672/1*ycW0WLC4CSo-FOhPKmcU-A.png)

After you log in, you can explore your cluster a bit more, I like this tool.

Next thing we can add for making our cluster monitoring deeper, it’s** **install** [heapster](https://github.com/kubernetes-retired/heapster)**
> Heapster enables Container Cluster Monitoring and Performance Analysis for [Kubernetes](https://github.com/kubernetes/kubernetes) (versions v1.0.6 and higher), and platforms which include it.

This tool can provide us console based cluster utilization statistics, but also it’ll add more information about node and pods resources to the Kubernetes Dashboard too.

It a bit tricky to install it on the bare metal cluster, and I needed to make some investigation about why it wasn't working from “box” but I have found the decision.

So let’s go on and deploy this add-on:

    **control#** vi heapster.yaml

    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: heapster
      namespace: kube-system
    ---
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: heapster
      namespace: kube-system
    spec:
      replicas: 1
      template:
        metadata:
          labels:
            task: monitoring
            k8s-app: heapster
        spec:
          serviceAccountName: heapster
          containers:
          - name: heapster
            image: gcr.io/google_containers/heapster-amd64:v1.4.2
            imagePullPolicy: IfNotPresent
            command:
            - /heapster
            - --source=kubernetes.summary_api:''?useServiceAccount=true&kubeletHttps=true&kubeletPort=10250&insecure=true
    ---
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        task: monitoring
        # For use as a Cluster add-on ([https://github.com/kubernetes/kubernetes/tree/master/cluster/addons](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons))
        # If you are NOT using this as an addon, you should comment out this line.
        kubernetes.io/cluster-service: 'true'
        kubernetes.io/name: Heapster
      name: heapster
      namespace: kube-system
    spec:
      ports:
      - port: 80
        targetPort: 8082
      selector:
        k8s-app: heapster
    ---
    kind: ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
      name: heapster
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: system:heapster
    subjects:
    - kind: ServiceAccount
      name: heapster
      namespace: kube-system

This is mostly standard deployment file from the Heapster community, but it differs a bit, for making it work on our cluster the “**source=**” string in heapster deployment has been changed this way:

    --source=kubernetes.summary_api:''?useServiceAccount=true&kubeletHttps=true&kubeletPort=10250&insecure=true

In this [description](https://github.com/kubernetes-retired/heapster/blob/master/docs/source-configuration.md), you can find all these options. I have changed the kubelet port to 10250 and disable SSL certificate checking (there was some issue with it).

Also, we need to add permissions, to get nodes statistic, into the Heapster RBAC role, add this few stings to the end of the role:

    **control# **kubectl edit clusterrole system:heapster

    ......
    ...

    - apiGroups:
      - ""
      resources:
      - nodes/stats
      verbs:
      - get

Finally, your RBAC role must look like this:

    # Please edit the object below. Lines beginning with a '#' will be ignored,
    # and an empty file will abort the edit. If an error occurs while saving this file will be
    # reopened with the relevant failures.
    #
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      annotations:
        rbac.authorization.kubernetes.io/autoupdate: "true"
      creationTimestamp: "2019-02-22T18:58:32Z"
      labels:
        kubernetes.io/bootstrapping: rbac-defaults
      name: system:heapster
      resourceVersion: "6799431"
      selfLink: /apis/rbac.authorization.k8s.io/v1/clusterroles/system%3Aheapster
      uid: d99065b5-36d3-11e9-a7e6-763fc8adcb06
    rules:
    - apiGroups:
      - ""
      resources:
      - events
      - namespaces
      - nodes
      - pods
      verbs:
      - get
      - list
      - watch
    - apiGroups:
      - extensions
      resources:
      - deployments
      verbs:
      - get
      - list
      - watch
    - apiGroups:
      - ""
      resources:
      - nodes/stats
      verbs:
      - get

OK, now lets run a command to make sure our heapster deployment running successfully.

    **control# ** kubectl top node

    NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
    kube-master1   183m         9%     1161Mi          60%       
    kube-master2   235m         11%    1130Mi          59%       
    kube-worker1   189m         4%     1216Mi          41%       
    kube-worker2   218m         5%     1290Mi          44%       
    kube-worker3   181m         4%     1305Mi          44%

Good, if you got the same output it’s mean that all was done right, now lets login back to our dashboard page and check new graphs that now available.

![](https://cdn-images-1.medium.com/max/3730/1*b9mloXwgvaqaO7QEcs8KUA.jpeg)

![](https://cdn-images-1.medium.com/max/3396/1*L784pqTq4JIPDSDJTCq_ow.jpeg)

From now we can also see the real resource's utilization for the cluster nodes, pods and so on.

If this not enough, then you can improve your statistic a bit more and add the InfluxDB+Grafana. This will add the ability to draw our own Grafana dashboards then.

We’ll use this [InfluxDB+Grafana](https://github.com/kubernetes-retired/heapster/tree/master/deploy/kube-config/influxdb) version of installation from the Heapster Git page, but with modification as usual. As we already have previously configured heapster deployment, the only things we need to do, it adds Grafana and InfluxDB and changes the existing heapster deployment then, to make it write metrics to Influx too.

OK, let's move on and create InfluxDB and Grafana deployments:

    **control#** vi influxdb.yaml

    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: monitoring-influxdb
      namespace: kube-system
    spec:
      replicas: 1
      template:
        metadata:
          labels:
            task: monitoring
            k8s-app: influxdb
        spec:
          containers:
          - name: influxdb
            image: k8s.gcr.io/heapster-influxdb-amd64:v1.5.2
            volumeMounts:
            - mountPath: /data
              name: influxdb-storage
          volumes:
          - name: influxdb-storage
            emptyDir: {}
    ---
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        task: monitoring
        # For use as a Cluster add-on ([https://github.com/kubernetes/kubernetes/tree/master/cluster/addons](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons))
        # If you are NOT using this as an addon, you should comment out this line.
        kubernetes.io/cluster-service: 'true'
        kubernetes.io/name: monitoring-influxdb
      name: monitoring-influxdb
      namespace: kube-system
    spec:
      ports:
      - port: 8086
        targetPort: 8086
      selector:
        k8s-app: influxdb

Then Grafana, also don’t forget to change the service configuration for enabling MetaLB load balancer on it, to get external IP for Grafana service.

    **control# **vi grafana.yaml

    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: monitoring-grafana
      namespace: kube-system
    spec:
      replicas: 1
      template:
        metadata:
          labels:
            task: monitoring
            k8s-app: grafana
        spec:
          containers:
          - name: grafana
            image: k8s.gcr.io/heapster-grafana-amd64:v5.0.4
            ports:
            - containerPort: 3000
              protocol: TCP
            volumeMounts:
            - mountPath: /etc/ssl/certs
              name: ca-certificates
              readOnly: true
            - mountPath: /var
              name: grafana-storage
            env:
            - name: INFLUXDB_HOST
              value: monitoring-influxdb
            - name: GF_SERVER_HTTP_PORT
              value: "3000"
              # The following env variables are required to make Grafana accessible via
              # the kubernetes api-server proxy. On production clusters, we recommend
              # removing these env variables, setup auth for grafana, and expose the grafana
              # service using a LoadBalancer or a public IP.
            - name: GF_AUTH_BASIC_ENABLED
              value: "false"
            - name: GF_AUTH_ANONYMOUS_ENABLED
              value: "true"
            - name: GF_AUTH_ANONYMOUS_ORG_ROLE
              value: Admin
            - name: GF_SERVER_ROOT_URL
              # If you're only using the API Server proxy, set this value instead:
              # value: /api/v1/namespaces/kube-system/services/monitoring-grafana/proxy
              value: /
          volumes:
          - name: ca-certificates
            hostPath:
              path: /etc/ssl/certs
          - name: grafana-storage
            emptyDir: {}
    ---
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        # For use as a Cluster add-on ([https://github.com/kubernetes/kubernetes/tree/master/cluster/addons](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons))
        # If you are NOT using this as an addon, you should comment out this line.
        kubernetes.io/cluster-service: 'true'
        kubernetes.io/name: monitoring-grafana
      name: monitoring-grafana
      namespace: kube-system
    spec:
      # In a production setup, we recommend accessing Grafana through an external Loadbalancer
      # or through a public IP.
      # type: LoadBalancer
      # You could also use NodePort to expose the service at a randomly-generated port
      # type: NodePort
      type: LoadBalancer
      ports:
      - port: 80
        targetPort: 3000
      selector:
        k8s-app: grafana

And create them:

    **control# **kubectl create -f influxdb.yaml

    deployment.extensions/monitoring-influxdb created
    service/monitoring-influxdb created

    **control# **kubectl create -f grafana.yaml 

    deployment.extensions/monitoring-grafana created
    service/monitoring-grafana created

Now it’s time to change a heapster deployment and add InfluxDB connection into it, only one string we need to add for it:

    - --sink=influxdb:[http://monitoring-influxdb.kube-system.svc:8086](http://monitoring-influxdb.kube-system.svc:8086)

Edit heapster deployment:

    **control# **kubectl get deployments --namespace=kube-system

    NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
    coredns                2/2     2            2           49d
    heapster               1/1     1            1           2d12h
    kubernetes-dashboard   1/1     1            1           3d21h
    monitoring-grafana     1/1     1            1           115s
    monitoring-influxdb    1/1     1            1           2m18s

    **control# **kubectl edit deployment heapster --namespace=kube-system

    *... beginning bla bla bla*

    spec:
          containers:
          - command:
            - /heapster
            - --source=kubernetes.summary_api:''?useServiceAccount=true&kubeletHttps=true&kubeletPort=10250&insecure=true
            - --sink=influxdb:[http://monitoring-influxdb.kube-system.svc:8086](http://monitoring-influxdb.kube-system.svc:8086)
            image: gcr.io/google_containers/heapster-amd64:v1.4.2
            imagePullPolicy: IfNotPresent

    *.... end *

Now let’s find the Grafana service external IP and login inside it:

    **control# **kubectl get svc --namespace=kube-system
    NAME     TYPE       CLUSTER-IP      EXTERNAL-IP PORT(S)        AGE

    ..... some other services here

    monitoring-grafana     LoadBalancer   10.98.111.200    192.168.0.241   80:32148/TCP    18m

Open [http://192.168.0.241](http://192.168.0.241) in your browser, with admin/admin credentials for the first time:

![](https://cdn-images-1.medium.com/max/2650/1*qW0kPxoWQfkuwkkgBXnGkA.jpeg)

My Grafana was empty after I have logged in, but fortunately, we can get all needed dashboards from **grafana.com**, we need to import the number 3649 and 3646 dashboards. Choose the right data source when you’ll import them.

After this, you’ll be able to see Nodes and Pods resources utilization and you can create own custom dashboards of course.

![](https://cdn-images-1.medium.com/max/3812/1*leAm0_I8fijhxsa7zQ9qZA.jpeg)

![](https://cdn-images-1.medium.com/max/3812/1*KRNsdZn7Q-WL5dMJMzXQ0g.jpeg)

Well, that’s all about monitoring, for now, the next things we may need, it’s storing logs from our applications and cluster. There are a few ways how we can do it, all of them described in Kubernetes [documentation](https://kubernetes.io/docs/concepts/cluster-administration/logging/). Based on my own experience I prefer to have an external installation of Elasticsearch & Kibana services and only logging agent will be running on every Kubernetes worker node. This will save our cluster from any overloads, due to the big amount of logs and other problems, also it’ll be possible to get logs even if our cluster will fully degrade.

The most popular log collecting stack for Kubernetes evangelists, it’s Elasticsearch, Fluentd, and Kibana (EFK stack). In this example, we’ll run Elasticsearch with Kibana on the external node (you can use your existing ELK stack also) and will run a Fluentd inside our cluster as a daemonset on each and every node, as a log collecting agent.

I will pass out the section about creating the VM with Elasticsearch & Kibana installations, it’s a popular enough theme, so you can find a lot of documents about how to do it better, for example, you can use mine [article](https://medium.com/devopslinks/nginx-logs-monitoring-using-elk-stack-docker-4b64f46e989) as well. Just remove the **logstesh** part of config from **docker-compose.yml** file and also remove 127.0.0.1 from the Elasticsearch ports section.

After finishing you must have a working Elasticsearch, that listened on VM-IP:9200 port, if you need any additional protection you can configure the login:pass or secure keys, between Fluentd and Elasticsearch. But I often just protecting it with iptables rules.

Only things we left to do now, it's creating some fluentd daemonset in Kubernetes and specify the outside elasticsearch **node:port** address in configuration.

We’ll use official Kubernetes Add-On yaml configs from [there](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/fluentd-elasticsearch), with small modifications:

    **control# **vi fluentd-es-ds.yaml

    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: fluentd-es
      namespace: kube-system
      labels:
        k8s-app: fluentd-es
        kubernetes.io/cluster-service: "true"
        addonmanager.kubernetes.io/mode: Reconcile
    ---
    kind: ClusterRole
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: fluentd-es
      labels:
        k8s-app: fluentd-es
        kubernetes.io/cluster-service: "true"
        addonmanager.kubernetes.io/mode: Reconcile
    rules:
    - apiGroups:
      - ""
      resources:
      - "namespaces"
      - "pods"
      verbs:
      - "get"
      - "watch"
      - "list"
    ---
    kind: ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: fluentd-es
      labels:
        k8s-app: fluentd-es
        kubernetes.io/cluster-service: "true"
        addonmanager.kubernetes.io/mode: Reconcile
    subjects:
    - kind: ServiceAccount
      name: fluentd-es
      namespace: kube-system
      apiGroup: ""
    roleRef:
      kind: ClusterRole
      name: fluentd-es
      apiGroup: ""
    ---
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: fluentd-es-v2.4.0
      namespace: kube-system
      labels:
        k8s-app: fluentd-es
        version: v2.4.0
        kubernetes.io/cluster-service: "true"
        addonmanager.kubernetes.io/mode: Reconcile
    spec:
      selector:
        matchLabels:
          k8s-app: fluentd-es
          version: v2.4.0
      template:
        metadata:
          labels:
            k8s-app: fluentd-es
            kubernetes.io/cluster-service: "true"
            version: v2.4.0
          # This annotation ensures that fluentd does not get evicted if the node
          # supports critical pod annotation based priority scheme.
          # Note that this does not guarantee admission on the nodes (#40573).
          annotations:
            scheduler.alpha.kubernetes.io/critical-pod: ''
            seccomp.security.alpha.kubernetes.io/pod: 'docker/default'
        spec:
          priorityClassName: system-node-critical
          serviceAccountName: fluentd-es
          containers:
          - name: fluentd-es
            image: k8s.gcr.io/fluentd-elasticsearch:v2.4.0
            env:
            - name: FLUENTD_ARGS
              value: --no-supervisor -q
            resources:
              limits:
                memory: 500Mi
              requests:
                cpu: 100m
                memory: 200Mi
            volumeMounts:
            - name: varlog
              mountPath: /var/log
            - name: varlibdockercontainers
              mountPath: /var/lib/docker/containers
              readOnly: true
            - name: config-volume
              mountPath: /etc/fluent/config.d
          terminationGracePeriodSeconds: 30
          volumes:
          - name: varlog
            hostPath:
              path: /var/log
          - name: varlibdockercontainers
            hostPath:
              path: /var/lib/docker/containers
          - name: config-volume
            configMap:
              name: fluentd-es-config-v0.2.0

Then we need to provide some configuration, for our fluentd:

    **control#** vi fluentd-es-configmap.yaml

    kind: ConfigMap
    apiVersion: v1
    metadata:
      name: fluentd-es-config-v0.2.0
      namespace: kube-system
      labels:
        addonmanager.kubernetes.io/mode: Reconcile
    data:
      system.conf: |-
        <system>
          root_dir /tmp/fluentd-buffers/
        </system>

    containers.input.conf: |-
        <source>
          [@id](http://twitter.com/id) fluentd-containers.log
          [@type](http://twitter.com/type) tail
          path /var/log/containers/*.log
          pos_file /var/log/es-containers.log.pos
          tag raw.kubernetes.*
          read_from_head true
          <parse>
            [@type](http://twitter.com/type) multi_format
            <pattern>
              format json
              time_key time
              time_format %Y-%m-%dT%H:%M:%S.%NZ
            </pattern>
            <pattern>
              format /^(?<time>.+) (?<stream>stdout|stderr) [^ ]* (?<log>.*)$/
              time_format %Y-%m-%dT%H:%M:%S.%N%:z
            </pattern>
          </parse>
        </source>

    # Detect exceptions in the log output and forward them as one log entry.
        <match raw.kubernetes.**>
          [@id](http://twitter.com/id) raw.kubernetes
          [@type](http://twitter.com/type) detect_exceptions
          remove_tag_prefix raw
          message log
          stream stream
          multiline_flush_interval 5
          max_bytes 500000
          max_lines 1000
        </match>

    # Concatenate multi-line logs
        <filter **>
          [@id](http://twitter.com/id) filter_concat
          [@type](http://twitter.com/type) concat
          key message
          multiline_end_regexp /\n$/
          separator ""
        </filter>

    # Enriches records with Kubernetes metadata
        <filter kubernetes.**>
          [@id](http://twitter.com/id) filter_kubernetes_metadata
          [@type](http://twitter.com/type) kubernetes_metadata
        </filter>

    # Fixes json fields in Elasticsearch
        <filter kubernetes.**>
          [@id](http://twitter.com/id) filter_parser
          [@type](http://twitter.com/type) parser
          key_name log
          reserve_data true
          remove_key_name_field true
          <parse>
            [@type](http://twitter.com/type) multi_format
            <pattern>
              format json
            </pattern>
            <pattern>
              format none
            </pattern>
          </parse>
        </filter>

    output.conf: |-
        <match **>
          [@id](http://twitter.com/id) elasticsearch
          [@type](http://twitter.com/type) elasticsearch
          [@log_level](http://twitter.com/log_level) info
          type_name _doc
          include_tag_key true
          host 192.168.1.253
          port 9200
          logstash_format true
          <buffer>
            [@type](http://twitter.com/type) file
            path /var/log/fluentd-buffers/kubernetes.system.buffer
            flush_mode interval
            retry_type exponential_backoff
            flush_thread_count 2
            flush_interval 5s
            retry_forever
            retry_max_interval 30
            chunk_limit_size 2M
            queue_limit_length 8
            overflow_action block
          </buffer>
        </match>

This is a very easy configuration, but it full enough for a quick start, it’ll collect a system and application logs. If you’ll need something more complicated, you may read the official documentation about fluentd plugins and configs for Kubernetes.

Now let’s create a fluentd daemonset in our cluster:

    **control#** kubectl create -f fluentd-es-ds.yaml

    serviceaccount/fluentd-es created
    clusterrole.rbac.authorization.k8s.io/fluentd-es created
    clusterrolebinding.rbac.authorization.k8s.io/fluentd-es created
    daemonset.apps/fluentd-es-v2.4.0 created

    **control# **kubectl create -f fluentd-es-configmap.yaml

    configmap/fluentd-es-config-v0.2.0 created

Make sure that all fluentd pods and other resources were started OK, and open Kibana then, in Kibana you need to find and add new index from fluentd. If you’ll find some, that’s mean everything was done right, if nope then check previous steps and recreate daemonset or edit configmap:

![](https://cdn-images-1.medium.com/max/3826/1*wKOkSRCoEWVPAaBqaUnfAw.jpeg)

![](https://cdn-images-1.medium.com/max/3844/1*GlpYTuYj3DzSPsiqB3pi5Q.jpeg)

Great, now after we start getting some logs from our cluster, we can create any kind of dashboard we want. Well, of course, it’s a very usual configuration so you may need to change it for your specific purposes. The main goal was to show the way how it can be done.

After all previously done steps, we got a really good-looking, production-ready Kubernetes cluster. So it’s time to deploy some test applications in it and see how it’ll be.

We can take for this example my small Python/Flask Kubyk application, that already has a Docker container so we can take it from the public docker registry. Then we need to add an external database file for this app, we’ll use our configured GlusterFS storage for it.

First, let's create a new **pvc** (persistent volume claim)volume for this app, we’ll store the [SQLite](https://www.sqlite.org/) database with users credentials in there. We can use a previously created storage class from part 2 of this tutorial.

    **control# **mkdir kubyk && cd kubyk

    **control# **vi kubyk-pvc.yaml
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: kubyk
      annotations:
        volume.beta.kubernetes.io/storage-class: "slow"
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi

    **control# **kubectl create -f kubyk-pvc.yaml

After we have created a new PVC for our app we ready to deploy it.

    **control#** vi kubyk-deploy.yaml 

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: kubyk-deployment
    spec:
      selector:
        matchLabels:
          app: kubyk
      replicas: 1
      template:
        metadata:
          labels:
            app: kubyk
        spec:
          containers:
          - name: kubyk
            image: ratibor78/kubyk
            ports:
            - containerPort: 80
            volumeMounts:
            - name: kubyk-db
              mountPath: /kubyk/sqlite
          volumes:
          - name: kubyk-db
            persistentVolumeClaim:
              claimName: kubyk

    **control#** vi kubyk-service.yaml 

    apiVersion: v1
    kind: Service
    metadata:
      name: kubyk
    spec:
      type: LoadBalancer
      selector:
        app: kubyk
      ports:
      - port: 80
        name: http

Now let's create deployment and service:

    **control# **kubectl create -f kubyk-deploy.yaml 

    deployment.apps/kubyk-deployment created

    **control# **kubectl create -f kubyk-service.yaml 

    service/kubyk created

Check the new assigned IP for service and pod status also:

    **control# **kubectl get po

    NAME                                READY   STATUS    RESTARTS AGE
    glusterfs-2wxk7                     1/1     Running   1        2d1h
    glusterfs-5dtdj                     1/1     Running   1        41d
    glusterfs-zqxwt                     1/1     Running   0        2d1h
    heketi-b8c5f6554-f92rn              1/1     Running   0        8d
    kubyk-deployment-75d5447d46-jrttp   1/1     Running   0        11s

    **control# **kubectl get svc
    NAME  TYPE     CLUSTER-IP       EXTERNAL-IP    PORT(S)        AGE

    ... some text.. 
    kubyk LoadBalancer 10.109.105.224   192.168.0.242 80:32384/TCP 10s

OK, it seems that we have successfully started our new application, if we’ll open [http://192.168.0.242](http://192.168.0.242) IP in the browser, we must see the login page of this app. We can use admin/admin credentials for login in, but if we try to log in on this stage, we’ll get an error, as there is no database available yet.

There is an error log messages example from a pod in Kubernetes dashboard:

![](https://cdn-images-1.medium.com/max/3514/1*l7ih-wAX-toh2aNnq_e0jQ.jpeg)

To fix this situation, we need to copy the SQLite DB file, from my git repository into the previously created PVC volume. So the application can start using this database then.

    **control# **git clone [https://github.com/ratibor78/kubyk.git](https://github.com/ratibor78/kubyk.git) 

    **control# **kubectl cp ./kubyk/sqlite/database.db kubyk-deployment-75d5447d46-jrttp:/kubyk/sqlite

We use the pod from the application and **kubectl cp **command to copy this file into the volume.

Also, we need grant write access to this directory for nginx user, as in docker container, my application runs under nginx user by [supervisord](https://github.com/ratibor78/kubyk/blob/master/supervisord.conf).

    **control# **kubectl exec -ti kubyk-deployment-75d5447d46-jrttp -- chown -R nginx:nginx /kubyk/sqlite/

And let’s try to log in one more time:

![](https://cdn-images-1.medium.com/max/3782/1*Gqmq1mIb9fwwG6RBHjID8w.jpeg)

Great, now our app is working well, and we can scale this **kubyk** deployment up to 3 replicas for example, for putting one copy of the app on one worker node. As we have created pvc volume previously, all our pods with application replicas will use the same DB, and also service will balance traffic between replicas using round-robin schema.

    **control# **kubectl get deployments

    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    heketi             1/1     1            1           39d
    kubyk-deployment   1/1     1            1           4h5m

    **control# **kubectl scale deployments kubyk-deployment --replicas=3

    deployment.extensions/kubyk-deployment scaled

    **control# **kubectl get po

    NAME                                READY   STATUS    RESTARTS   AGE
    glusterfs-2wxk7                     1/1     Running   1        2d5h
    glusterfs-5dtdj                     1/1     Running   21       41d
    glusterfs-zqxwt                     1/1     Running   0        2d5h
    heketi-b8c5f6554-f92rn              1/1     Running   0        8d
    kubyk-deployment-75d5447d46-bdnqx   1/1     Running   0        26s
    kubyk-deployment-75d5447d46-jrttp   1/1     Running   0        4h7m
    kubyk-deployment-75d5447d46-wz9xz   1/1     Running   0        26s

![](https://cdn-images-1.medium.com/max/3818/1*Llw3vjLxpMs157KMFzdgwg.jpeg)

Now we got one replica of our app per one worker node, so if we’ll lose some of the nodes, the application will still continue working. Also, we got an easy type of load balancing as I said previously. Not bad as for the first time.

Let's create some new user in our application:

![](https://cdn-images-1.medium.com/max/3818/1*_LNuRsTIKMxaePz_RImBsQ.jpeg)

![](https://cdn-images-1.medium.com/max/3130/1*OVWiFTBaDVg576S9Jjx_OQ.jpeg)

Any new requests will be processing by the next pod in a list, you can check this by looking into the pod's logs. For example, the new user will be created by the application in one pod, then next pod will respond for the next request and so on. As this app uses the same persistent volume for storing DB even if all of the replicas will disappear, all data will be in safe.

In big and complicated applications we’ll need not only dedicated volume for the DB but also different volumes for placing static content and more else things in them.

Well, it seems that we almost finished with this part of the tutorial, there is a lot of things I can add because the Kubernetes theme is big and dynamically growing, but I’ll stop for now. The main goal for this articles cycle was showing the way of creating your own Kubernetes cluster and I hope this was useful enough for you.

**P.S**

Stability and stress tests, of course.

Cluster schema that we used in this example may continue working without 2 workers nodes, 1 master and 1 Etcd node. You can shut down them and see that our test app still working.

When I wrote this tutorial, I have to prepare the same cluster for the production, in fact almost the same scheme. At one day, when the cluster was created and some part of the application was deployed in it, there was a big power supply issue in the DC and totally all cluster servers were shut down, like in scary movies for sysadmins. Some of the servers were down for a long time and some got a file system errors after it. But when I started them all again, I was very surprised when Kubernetes cluster fully recovered. All GlusterFS volumes, all deployments have started. It shows a big potential for this technology for me.

Goodbye and hope to meet you again.

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
