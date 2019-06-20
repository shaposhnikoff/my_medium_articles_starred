
# Production setup for Kubernetes with KOPS in AWS

Containers getting ready to be loaded onto a ship for transportation. (credits: gcaptain.com)

## Introduction

In the previous blog we setup a [simple K8 cluster using KOPS in AWS](https://medium.com/@naikaa/kubernetes-with-kops-7d5b3378ca7a), this is the next part to extend the setup to a Highly Available and scalable K8 cluster for production workloads.

For a production grade K8 cluster we will create the cluster in private subnet. We will create 3 Masters one in each AZ. We will create 3 instance groups one per AZ and configure autoscaling groups so the underlying K8 nodes can scale up or down in each AZ depending on the workload. The architecture of the K8 cluster looks as follows.

![VPC architecture of highly scalable, highly available private kubernetes cluster](https://cdn-images-1.medium.com/max/2000/1*lVdrfKCbn_ubkf0uQATI2A.jpeg)*VPC architecture of highly scalable, highly available private kubernetes cluster*

Installation steps:

1. Perform steps 1 through 7 as mentioned in this blog [kubernetes with KOPS in AWS](https://medium.com/@naikaa/kubernetes-with-kops-7d5b3378ca7a)

2. create a cluster.yaml, use this template file and update it as needed.

    apiVersion: kops/v1alpha2
    kind: Cluster
    metadata:
      creationTimestamp: 2018-03-17T09:37:40Z
      name: useast1.prod.kubernetes.yourcompany.com
    spec:
      additionalPolicies:
        node: |
          [
            {
              "Effect": "Allow",
              "Action": [
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:DescribeAutoScalingInstances",
                "autoscaling:DescribeTags",
                "autoscaling:DescribeLaunchConfigurations",
                "autoscaling:SetDesiredCapacity",
                "autoscaling:TerminateInstanceInAutoScalingGroup"
              ],
              "Resource": "*"
            }
          ]
      api:
        loadBalancer:
          type: Internal
      authorization:
        rbac: {}
      channel: stable
      cloudProvider: aws
      configBase: s3://clusters.kubernetes.yourcompany.com/useast1.prod.kubernetes.yourcompany.com
      dnsZone: useast1.prod.kubernetes.yourcompany.com
      etcdClusters:
      - etcdMembers:
        - instanceGroup: master-us-east-1a
          name: a
        - instanceGroup: master-us-east-1b
          name: b
        - instanceGroup: master-us-east-1c
          name: c
        name: main
      - etcdMembers:
        - instanceGroup: master-us-east-1a
          name: a
        - instanceGroup: master-us-east-1b
          name: b
        - instanceGroup: master-us-east-1c
          name: c
        name: events
      iam:
        allowContainerRegistry: true
        legacy: false
      kubeDNS:
        provider: CoreDNS
      kubelet:
        featureGates:
          ExpandPersistentVolumes: "true"
          PodPriority: "true"
      kubernetesApiAccess:
      - 0.0.0.0/0
      kubernetesVersion: 1.11.6
      masterInternalName: api.internal.useast1.prod.kubernetes.yourcompany.com
      masterPublicName: api.useast1.prod.kubernetes.yourcompany.com
      networkCIDR: 172.22.0.0/16
      networking:
        calico:
          logSeverityScreen: warning
      nonMasqueradeCIDR: 100.64.0.0/10
      sshAccess:
      - 0.0.0.0/0
      subnets:
      - cidr: 172.22.32.0/19
        name: us-east-1a
        type: Private
        zone: us-east-1a
      - cidr: 172.22.64.0/19
        name: us-east-1b
        type: Private
        zone: us-east-1b
      - cidr: 172.22.96.0/19
        name: us-east-1c
        type: Private
        zone: us-east-1c
      - cidr: 172.22.0.0/22
        name: utility-us-east-1a
        type: Utility
        zone: us-east-1a
      - cidr: 172.22.4.0/22
        name: utility-us-east-1b
        type: Utility
        zone: us-east-1b
      - cidr: 172.22.8.0/22
        name: utility-us-east-1c
        type: Utility
        zone: us-east-1c
      topology:
        dns:
          type: Public
        masters: private
        nodes: private

    ---

    apiVersion: kops/v1alpha2
    kind: InstanceGroup
    metadata:
      creationTimestamp: 2018-03-12T09:37:40Z
      labels:
        kops.k8s.io/cluster: useast1.prod.kubernetes.yourcompany.com
      name: master-us-east-1a
    spec:
      image: kope.io/k8s-1.11-debian-stretch-amd64-hvm-ebs-2018-08-17
      machineType: m5.large
      maxSize: 1
      minSize: 1
      nodeLabels:
        kops.k8s.io/instancegroup: master-us-east-1a
      role: Master
      subnets:
      - us-east-1a

    ---

    apiVersion: kops/v1alpha2
    kind: InstanceGroup
    metadata:
      creationTimestamp: 2018-03-12T09:37:41Z
      labels:
        kops.k8s.io/cluster: useast1.prod.kubernetes.yourcompany.com
      name: master-us-east-1b
    spec:
      image: kope.io/k8s-1.11-debian-stretch-amd64-hvm-ebs-2018-08-17
      machineType: m5.large
      maxSize: 1
      minSize: 1
      nodeLabels:
        kops.k8s.io/instancegroup: master-us-east-1b
      role: Master
      subnets:
      - us-east-1b

    ---

    apiVersion: kops/v1alpha2
    kind: InstanceGroup
    metadata:
      creationTimestamp: 2018-03-12T09:37:41Z
      labels:
        kops.k8s.io/cluster: useast1.prod.kubernetes.yourcompany.com
      name: master-us-east-1c
    spec:
      image: kope.io/k8s-1.11-debian-stretch-amd64-hvm-ebs-2018-08-17
      machineType: m5.large
      maxSize: 1
      minSize: 1
      nodeLabels:
        kops.k8s.io/instancegroup: master-us-east-1c
      role: Master
      subnets:
      - us-east-1c

    ---

    apiVersion: kops/v1alpha2
    kind: InstanceGroup
    metadata:
      creationTimestamp: 2018-08-08T13:35:12Z
      labels:
        kops.k8s.io/cluster: useast1.prod.kubernetes.yourcompany.com
      name: nodes-us-east-1a
    spec:
      cloudLabels:
        k8s.io/cluster-autoscaler/enabled: "true"
        kubernetes.io/cluster/useast1.prod.kubernetes.yourcompany.com: "true"
      image: kope.io/k8s-1.11-debian-stretch-amd64-hvm-ebs-2018-08-17
      machineType: m5.xlarge
      maxPrice: "0.22"
      maxSize: 5
      minSize: 1
      nodeLabels:
        kops.k8s.io/instancegroup: nodes-us-east-1a
      role: Node
      subnets:
      - us-east-1a

    ---

    apiVersion: kops/v1alpha2
    kind: InstanceGroup
    metadata:
      creationTimestamp: 2018-08-08T13:35:12Z
      labels:
        kops.k8s.io/cluster: useast1.prod.kubernetes.yourcompany.com
      name: nodes-us-east-1b
    spec:
      cloudLabels:
        k8s.io/cluster-autoscaler/enabled: "true"
        kubernetes.io/cluster/useast1.prod.kubernetes.yourcompany.com: "true"
      image: kope.io/k8s-1.11-debian-stretch-amd64-hvm-ebs-2018-08-17
      machineType: m5.xlarge
      maxPrice: "0.22"
      maxSize: 5
      minSize: 1
      nodeLabels:
        kops.k8s.io/instancegroup: nodes-us-east-1b
      role: Node
      subnets:
      - us-east-1b

    ---

    apiVersion: kops/v1alpha2
    kind: InstanceGroup
    metadata:
      creationTimestamp: 2018-08-08T13:35:13Z
      labels:
        kops.k8s.io/cluster: useast1.prod.kubernetes.yourcompany.com
      name: nodes-us-east-1c
    spec:
      cloudLabels:
        k8s.io/cluster-autoscaler/enabled: "true"
        kubernetes.io/cluster/useast1.prod.kubernetes.yourcompany.com: "true"
      image: kope.io/k8s-1.11-debian-stretch-amd64-hvm-ebs-2018-08-17
      machineType: m5.xlarge
      maxPrice: "0.22"
      maxSize: 5
      minSize: 1
      nodeLabels:
        kops.k8s.io/instancegroup: nodes-us-east-1c
      role: Node
      subnets:
      - us-east-1c

3. Create the cluster

    export CLUSTER_NAME=useast1.prod.kubernetes.yourcompany.com
    export KOPS_STATE_STORE=s3://clusters.ue2.yourcompany.com
    export SSH_PUBLIC_KEY=~/id_rsa.pub

    # Force replace the cluster.yaml to the S3 kops bucket
    kops replace -— name $(CLUSTER_NAME) \ 
    -— state $(KOPS_STATE_STORE) \ 
    -— filename ./cluster.yaml \ 
    -— force

    # Upload ssh key to the S3 kops bucket to ssh into master and nodes
    kops create secret sshpublickey admin \ 
    -— name $(CLUSTER_NAME) \ 
    -— state $(KOPS_STATE_STORE) \ 
    -— pubkey $(SSH_PUBLIC_KEY)

    # Create the k8 cluster config and preview
    kops update cluster \ 
    —- name $(CLUSTER_NAME) \ 
    -- state $(KOPS_STATE_STORE)

    # Create the k8 cluster, pass --yes to execute, wait for 10-15 mins 
    kops update cluster \ 
    —- name $(CLUSTER_NAME) \ 
    —- state $(KOPS_STATE_STORE) \ 
    -— yes

    # Rolling update the cluster, pass --yes to execute  
    kops rolling-update cluster $(CLUSTER_NAME) —- yes

4. Validate the cluster

    kops validate cluster $(CLUSTER_NAME)

5. Continue from step 9 and beyond in this blog [kubernetes with KOPS in AWS](https://medium.com/@naikaa/kubernetes-with-kops-7d5b3378ca7a) to install dashboard and deploy test application.
