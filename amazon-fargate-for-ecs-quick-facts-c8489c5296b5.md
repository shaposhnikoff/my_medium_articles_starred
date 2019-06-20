
# Amazon Fargate for ECS — Quick Facts

Amazon Fargate allows to run Containers without having to manage clusters and servers. It is available for Amazon ECS (ECS based Cluster) now and will be available for Amazon EKS (Kubernetes based Cluster) in early 2018.

### Fargate Architecture courtesy of [AWS](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/launch_types.html)

![](https://cdn-images-1.medium.com/max/2000/1*NX3lhIy7wlmAp_qOp6W2kw.png)

### **Key Facts**

* Runs within Amazon VPC

* No management of compute infrastructure

* Based on Amazon Linux 2017.09

* The network mode should be **awsvpc** for Fargate

* Task size must be specified for Fargate launch. The range of values you specify for memory restricts the cpu values an d vice versa. For example, if you use 256 (.25 vCPU) for cpu, then valid values for memory are 512MB, 1 GB and 2 GB

* Fargate balances tasks across Availability Zones

* **Cost**: It is billed to nearest second with a min charge of 1 min. As of 12/4/2017, the cost per vCPU is $0.00001406 per second ($0.0506 per hour) and per GB memory is $0.00000353 per second ($0.0127 per hour). For a sample container of 0.5 cpu, 1 GB memory, it will be $333 / annum and about $1000 for a HA configuration across 3 Azs in a Region.

### **Key Limitations for Fargate**

* Fargate only supports ECR/Docker Hub, no private registries yet

* cannot use [placementConstraints](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-placement-constraints.html)

* cannot use [links](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html) which allow containers to communicate each other without using port mappings

* [LogConfiguration](http://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_LogConfiguration.html) can only use awslogs (for example you cannot use splunk/fluent or syslog )

* Conatiners cannot run in privileged mode which gives elevated permissions to host

* Docker Security Options cannot be specified which are typically used for [SELinux](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwi8yNv2yfDXAhVIzoMKHal6A0cQFggnMAA&url=https%3A%2F%2Fselinuxproject.org%2F&usg=AOvVaw3rGtHGB4VL_TVnpURNWKD8) / [AppArmor](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwi_2Z3_yfDXAhUq04MKHXpfDZgQFggnMAA&url=https%3A%2F%2Fwiki.ubuntu.com%2FAppArmor&usg=AOvVaw1WzylrmSdwSADN7420atY5)

* [linuxParamaters](http://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_LinuxParameters.html) option (such as Kernel Capabilities) cannot be specified

* In [Data Volumes](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_data_volumes.html), **sourcePath** and **host** parameters not supported

* Custom schedulers like [blox](https://blox.github.io/) are not compatible

* Cluster [Reservation](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/cloudwatch-metrics.html#cluster_reservation) and [Utilization](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/cloudwatch-metrics.html#cluster_utilization) metrics are not available

* Amazon ECS Events are restricted to Tasks, not container instance based events

### Service Limits for Fargate Launch type

Be aware of these service limits and request for increase as you approach the limits

* Number of tasks per region per account — 20

* No of Public IPs — 20

* Max size of docker image — 4 GB

* Max size of shared volume used by multiple containers — 4 GB

* Max Container Storage size — 10 GB
