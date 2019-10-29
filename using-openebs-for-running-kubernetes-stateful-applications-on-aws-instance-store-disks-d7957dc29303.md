
# Using OpenEBS for running Kubernetes stateful applications on AWS instance store disks

Using OpenEBS with AWS instance stores

In this blog post, I cover the topic of “How to set up persistent storage” using AWS instance store disks for applications running on Kubernetes clusters. Instance store disks provide high performance, but they are not guaranteed to be present in the node all the time which of course means that when the node is rescheduled you can lose the data they are storing. There are two ways of obtaining storage on AWS for stateful applications.

1. Using AWS EBS disks

1. Using Local disks or instance store disks

## Running stateful apps using EBS volumes

![Stateful Applications using EBS volumes](https://cdn-images-1.medium.com/max/2000/0*s21MSo9P5bDNy64X)*Stateful Applications using EBS volumes*

## Problem with this approach

* When a node goes down, a new node comes up as part of Auto Scaling Groups(ASG). EBS disks that are associated with the old node have to be detached from the old node and attached to the new node. This is slow and not guaranteed to work seamlessly. Also, the new EBS volume will not have any data.

* If user application is capable of doing replication, then it will replicate data to this new disk which will take more time if it has a large amount of data. This will have an impact on performance.

* EBS volumes are slow and users are not able to make use of faster disks such as SSDs and NVMe.

* Slow failover means effectively no High Availability.

* Poor I/O, unless you want to spend a lot of unused disk space.

## Why can’t we use instance disks as-is for Kubernetes?

![Stateful Applications using instance stores](https://cdn-images-1.medium.com/max/2000/0*YF90x_1hjk9wJ5FP)*Stateful Applications using instance stores*

* When a node goes down, a new node comes up with its own disks and data is lost. This is because of AWS’ auto-scaling group and other policies. Per the ASG policy, the entire component associated with an…

Read the complete article in [MayaData’s Blog](https://blog.mayadata.io/openebs/using-openebs-for-running-kubernetes-stateful-applications-on-aws-instance-store-disks)
