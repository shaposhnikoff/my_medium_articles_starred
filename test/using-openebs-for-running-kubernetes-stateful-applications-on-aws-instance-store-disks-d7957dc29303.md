
# Using OpenEBS for running Kubernetes stateful applications on AWS instance store disks

Using OpenEBS with AWS instance stores

In this blog post, I cover the topic of “How to set up persistent storage” using AWS instance store disks for applications running on Kubernetes clusters. Instance store disks provide high performance, but they are not guaranteed to be present in the node all the time which of course means that when the node is rescheduled you can lose the data they are storing. There are two ways of obtaining storage on AWS for stateful applications.

1. Using AWS EBS disks

1. Using Local disks or instance store disks

## Running stateful apps using EBS volumes

![Stateful Applications using EBS volumes](https://cdn-images-1.medium.com/max/2000/0*s21MSo9P5bDNy64X)*Stateful Applications using EBS volumes*

## Problem with this approach

* When a node goes down, a new node comes up as part of Auto Scaling Groups(ASG). EBS disks that are associated with the old node have to be detached from the old node and attached to the new node.This is slow and not guaranteed to work seamlessly. Also, the new EBS volume will not have any data.

* If user application is capable of doing replication, then it will replicate data to this new disk which will take more time if it has large amount of data. This will have an impact on performance.

* EBS volumes are slow and users are not able to make use of faster disks such as SSDs and NVMe.

* Slow failover means effectively no High Availability.

* Poor I/O, unless you want to spend a lot on unused disk space.

## Why can’t we use instance disks as is for Kubernetes?

![Stateful Applications using instance stores](https://cdn-images-1.medium.com/max/2000/0*YF90x_1hjk9wJ5FP)*Stateful Applications using instance stores*

* When a node goes down, a new node comes up with it’s own disks and data is lost. This is because of AWS’ auto scaling group and other policies.Per the ASG policy,the entire component associated with an instance is deleted during the termination of a EC2 instance. So, user application which has the replication capability to manage replication by itself has to manage the data replication across nodes.

* What if your applications do not have this capability?

* If a local disk fails, then your data is lost.

## OpenEBS can help keep the data replicated across nodes

OpenEBS is an option for high availability of data combined with advantages of using physical disks.

## How is replication done with OpenEBS?

OpenEBS will have minimum 3 replicas to run OpenEBS cluster with high availability and if a node fails, OpenEBS will manage the data to be replicated to a new disk which will come up as part of ASG. In the meantime your workload is accessing the live data from one the replicas.

![Stateful Applications using OpenEBS and instance stores](https://cdn-images-1.medium.com/max/2000/0*WhjIjrek-7Fwo5kc)*Stateful Applications using OpenEBS and instance stores*

## How to quickly demonstrate OpenEBS on AWS?

1. Setup K8s nodes to auto mount the disks and to configure iSCSI initiators during reboot.

1. Similarly, configure AWS “User data” under launch configuration to configure iSCSI initiators and auto mount of disks during launch or reboot.

1. Install OpenEBS on Kubernetes Nodes. This should be easy and a couple of ways are discussed at the beginning of our docs, either using a Helm Chart or directly from Kubectl. More details are mentioned in [https://docs.openebs.io](https://docs.openebs.io).

1. Use OpenEBS Storage Classes to create Persistent Volumes for your stateful applications.

Below I provide step by step instructions that you should be able to cut and paste and customize. As you can see these include how to configure your AWS account for this simple POC as well.

## **Summary:**
> In summary, OpenEBS can be used to easily setup to run stateful applications on Kubernetes where AWS instance store disks are the underlying disks. This gives good manageability, improves resilience, and allows for relatively high performance for applications.

### Detailed explanation of OpenEBS cluster deployment on AWS and rebuilding of persistent volumes are given in below section.

## **Prerequisites**

* AWS account with full access to EC2, S3, and VPC

* Ubuntu 16.04

* KOPS tool installed

* AWS CLI installed for AWS account access

* SSH key generated to access EC2 instances

## On Local Ubuntu Machine

* Install with Ubuntu 16.04 LTS.

* Install kops utility package. I have followed[ Kubernetes documentation](https://kubernetes.io/docs/setup/custom-cloud/kops/). I have skipped step 2 and used my VPC id so that it can have full communication with its components.

* Install AWS CLI.This can be done [here](https://docs.aws.amazon.com/cli/latest/userguide/awscli-install-windows.html).

* AWS account with full access to EC2, S3, and VPC

* Generate a new ssh key. If you are not familiar, you can check it[ here](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/).

## Installing OpenEBS Cluster in AWS

**You should have the access to both AWS management console and local Ubuntu CLI for the installation of OpenEBS.**

## Perform following operations from AWS management console

1. Create VPC to create your Virtual Private Cloud. For that goto VPC service and create a VPC.

![](https://cdn-images-1.medium.com/max/2728/0*63YmEJtN2YLY6UyS)

2. Create Internet gateway and associate your VPC with the Internet Gateway.

![](https://cdn-images-1.medium.com/max/2732/0*SH6ODNJqiwgoVgjn)

This will attach internet connectivity to the vpc created by you. All nodes under this VPC will have the outside connectivity.

## **Perform the following procedure from your local ubuntu machine**

1. Download AWS CLI utility in your local machine.

1. Configure with your AWS account by executing the following command.

    aws configure

**Note:** You have to specify your AWS Access Key, Secret Access Key, Default region name, and Default output format for keeping the configuration details.

3. Create a S3 bucket to store your cluster configuration details as follows.

    aws s3 mb s3://<bucket_name>

4. Export the s3 bucket details using following command.

    export KOPS_STATE_STORE=s3://<bucket_name>

5. Create the cluster using following command.

    kops create cluster — name=<cluster_name>.k8s.local — vpc=<vpc_id> — zones=<zone_name>

This will create a cluster in the mentioned zone in your region provided as part of AWS configuration.

6. The above step will give a set of commands which can be used for customising your cluster configuration such as Cluster name change, Instance group for Nodes, and master etc. Following is an example output.

**Example:**

Cluster configuration has been created.

Suggestions:

* list clusters with: kops get cluster

* edit this cluster with: kops edit cluster ranjith.k8s.local

* edit your node instance group: kops edit ig — name=ranjith.k8s.local nodes

* edit your master instance group: kops edit ig — name=ranjith.k8s.local master-us-west-2a

Finally, configure your cluster with: kops update cluster name.k8s.local — yes

7. Change your instance image type and number of machines by executing corresponding commands. The exact command needed for your cluster will be shown at the end of the previous step. Following is an example.

**Example:**

Change your node configuration by executing as follows.

    kops edit ig — name=<cluster_name>.k8s.local nodes

Change your master instance type and number of machines by executing as follows.

    kops edit ig — name=<cluster_name>.k8s.local master-<zone_name>

**Note:** We used c3.xlarge as instance type for both Master and Nodes. Number of worker nodes used is 3 and master node as 1.

8. Once the customization is done, you can update the changes as follows.

    kops update cluster <cluster_name>.k8s.local — yes

9. The above step will deploy a 3 Node OpenEBS cluster in AWS. You can check the instance creation status by going to EC2 instance page and choosing the corresponding region.

10. From EC2 instance page, you will get each instance type Public IP.

**Example:**

![](https://cdn-images-1.medium.com/max/2274/0*l8vStlaBEgkyL8pX)

11. Go to **Launch Configuration** section in the EC2 page and take a *copy of Launch configuration* for nodes. Select the configuration for Node group and click on Actions pane.

**Example:**

![](https://cdn-images-1.medium.com/max/2266/0*wddk6Zhjnkxhx3EC)

12. Perform changes in the **Configure Details** section as follows.

a. Change the new configuration name if required.

b. Edit **Advanced Details** section and add the following entry at the end of **User data** section.

    #!/bin/bash

    set -x

    date

    apt-get install open-iscsi

    grep “@reboot root sleep 120;service open-iscsi restart” /etc/crontab || sudo sh -c ‘echo “@reboot root sleep 120;service open-iscsi restart” >> /etc/crontab’

    systemctl enable open-iscsi

    sh -c ‘echo “/dev/xvdd /mnt/openebs_xvdd auto defaults,nofail,comment=cloudconfig 0 2” >> /etc/fstab’

    reboot

    set -x

    umount /mnt/openebs/xvdd

    mount /dev/xvdd /mnt/openebs_xvdd

**Example:**

![](https://cdn-images-1.medium.com/max/2624/0*wLVeYNDu_v1bwViY)

Click **Skip** to review and proceed with Create launch configuration.

13. Go to **Auto Scale Group** section in the EC2 page. Select the configuration for Node group and click on Actions pane to edit Launch Configuration. Change the existing one with the new Launch Configuration and save the new configuration.

**Example:**

![](https://cdn-images-1.medium.com/max/2716/0*9aBUk7Md8igTZXGE)

14. SSH to each node using its public key as follows.

    ssh -i ~/.ssh/id_rsa admin@<public_ip>

15. SSH to all the Nodes where OpenEBS will be installed and perform the following commands to install iSCSI packages and auto mounting of local disk during reboot.

    sudo apt-get update

    sudo apt-get install open-iscsi

    sudo service open-iscsi restart

    sudo cat /etc/iscsi/initiatorname.iscsi

    sudo service open-iscsi status

    sudo sudo sh -c ‘echo “/dev/xvdd /mnt/openebs_xvdd auto defaults,nofail,comment=cloudconfig 0 2” >> /etc/fstab’

    grep “@reboot root sleep 120;service open-iscsi restart” /etc/crontab || sudo sh -c ‘echo “@reboot root sleep 120;service open-iscsi restart” >> /etc/crontab’

    sudo reboot

16. SSH to Master Node and perform the following commands to clone OpenEBS yaml file and deploy.

    wget [https://raw.githubusercontent.com/openebs/openebs/v0.6/k8s/openebs-operator.yaml](https://raw.githubusercontent.com/openebs/openebs/v0.6/k8s/openebs-operator.yaml)

    wget[ https://raw.githubusercontent.com/openebs/openebs/v0.6/k8s/openebs-storageclasses.yaml](https://raw.githubusercontent.com/openebs/openebs/v0.6/k8s/openebs-storageclasses.yaml)

17. Edit *openebs-operator.yaml* and add the following entry. This will create storage pool on one of the local disks attached to the hosts. Refer OpenEBS Storage Pools for more information.

     — -

    apiVersion: openebs.io/v1alpha1

    kind: StoragePool

    metadata:

    name: jivaawspool

    type: hostdir

    spec:

    path: “/mnt/openebs_xvdd”

     — -

18. Edit *openebs-storageclasses.yaml* by adding the following entry in your corresponding storage class.

    openebs.io/storage-pool: “jivaawspool”

**Example:**

     — -

    apiVersion: storage.k8s.io/v1

    kind: StorageClass

    metadata:

    name: openebs-percona

    provisioner: openebs.io/provisioner-iscsi

    parameters:

    openebs.io/storage-pool: “default”

    openebs.io/jiva-replica-count: “3”

    openebs.io/volume-monitor: “true”

    openebs.io/capacity: 5G

    openebs.io/storage-pool: “jivaawspool”

     — -

19. Apply openebs-operator.yaml by executing the following command.

    kubectl apply -f openebs-operator.yaml

20. Apply openebs-storageclasses.yaml by executing the following command.

    kubectl apply -f openebs-storageclasses.yaml

21. Deploy your application yaml which will be created on the local disk.

**Example:**

    kubectl apply -f percona-openebs-deployment.yaml

22. To check the status of application and Jiva Pods, use the following command.

    kubectl get pods -o wide

Output similar to the following is displayed.

    NAME READY STATUS RESTARTS AGE IP NODE

    percona-7f6bff67f6-cz47d 1/1 Running 0 1m 100.96.3.7 ip-172–20–40–26.us-west-2.compute.internal

    pvc-ef813ecc-9c8d-11e8-bdcc-0641dc4592b6-ctrl-84bcf764d6–269rj 2/2 Running 0 1m 100.96.1.4 ip-172–20–62–11.us-west-2.compute.internal

    pvc-ef813ecc-9c8d-11e8-bdcc-0641dc4592b6-rep-54b8f49ff8-bzjq4 1/1 Running 0 1m 100.96.1.5 ip-172–20–62–11.us-west-2.compute.internal

    pvc-ef813ecc-9c8d-11e8-bdcc-0641dc4592b6-rep-54b8f49ff8-lpz2k 1/1 Running 0 1m 100.96.2.8 ip-172–20–32–255.us-west-2.compute.internal

    pvc-ef813ecc-9c8d-11e8-bdcc-0641dc4592b6-rep-54b8f49ff8-rqnr7 1/1 Running 0 1m 100.96.3.6 ip-172–20–40–26.us-west-2.compute.internal

23. Get the status of PVC using the following command.

    kubectl get pvc

Output similar to the following is displayed.

    NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE

    demo-vol1-claim Bound pvc-ef813ecc-9c8d-11e8-bdcc-0641dc4592b6 5G RWO openebs-percona 3m

24. Get the status of PV using the following command.

    kubectl get pv

Output of the above command will be similar to the following.

    NAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM STORAGECLASS REASON AGE

    pvc-ef813ecc-9c8d-11e8-bdcc-0641dc4592b6 5G RWO Delete Bound default/demo-vol1-claim openebs-percona 3m
> Now you have deployed OpenEBS with AWS local disk in your k8s environment. You will get the advantages of both the low latency local disk use and fault tolerant architecture ensured by OpenEBS.

Hopefully this helps to understand benefits of using OpenEBS on top of AWS. Thank you for reading and please provide any feedback below or via twitter — @ranjithr005
