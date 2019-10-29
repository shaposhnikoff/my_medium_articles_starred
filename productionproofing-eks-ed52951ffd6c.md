Unknown markup type 10 { type: [33m10[39m, start: [33m80[39m, end: [33m90[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m132[39m, end: [33m142[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m215[39m, end: [33m233[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m124[39m, end: [33m152[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m156[39m, end: [33m160[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m164[39m, end: [33m182[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m22[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m109[39m, end: [33m113[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m118[39m, end: [33m127[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m39[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m167[39m, end: [33m174[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m155[39m, end: [33m162[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m35[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m46[39m, end: [33m55[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m61[39m, end: [33m70[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m274[39m, end: [33m310[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m90[39m, end: [33m108[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m275[39m, end: [33m308[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m312[39m, end: [33m319[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m55[39m, end: [33m82[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m146[39m, end: [33m164[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m169[39m, end: [33m179[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m259[39m, end: [33m269[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m134[39m, end: [33m149[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m307[39m, end: [33m325[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m57[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m238[39m, end: [33m251[39m }

# Productionproofing EKS

Productionproofing EKS

We recently migrated SaleMove infrastructure from self-managed Kubernetes clusters running on AWS to using [Amazon Elastic Container Service for Kubernetes (EKS)](https://aws.amazon.com/eks/). There were many surprises along the way to getting our EKS setup ready for production. This post covers some of these gotchas (others may already be fixed or are not likely to be relevant for a larger crowd) and is meant to be used as a reference when thinking of running EKS in production.

## Background

As soon as EKS was [announced as GA](https://aws.amazon.com/about-aws/whats-new/2018/06/amazon-elastic-container-service-for-kubernetes-eks-now-ga/) in June this year, we started looking at how we could offload some of our Kubernetes management pain and effort to AWS. Our previous clusters were created with and managed by [kube-aws](https://github.com/kubernetes-incubator/kube-aws). We now manage everything with Ansible. It configures the AWS network, creates the EKS cluster, and manages worker nodes.

Our main production cluster currently consists of about 15 m5.xlarge worker nodes. If you’re operating at a significantly different scale, then bear in mind that some of these recommendations may not apply to you.

So without further ado, **here’s the list of things you might want to look at before creating an EKS cluster with the goal of serving production traffic.**

* [Networking](#a043)

* [Networking — Limited pod capacity per subnet & VPC](#80f0)

* [Networking — Limited pod capacity per worker node](#5f0a)

* [Networking — Kubernetes scheduler is unaware about actual IP availability](#146e)

* [Networking — Some pods cannot be accessed from peered networks by default](#e636)

* [Default worker AMI](#c304)

* [AMI — Based on Amazon Linux 2](#f74c)

* [AMI — No docker log rotation](#121d)

* [AMI — Docker freezes](#d0da)

* [AMI — Corrupted disk statistics](#e05c)

* [Authentication and authorization](#357c)

* [Auth — RBAC enabled](#b8fc)

* [Auth — AWS IAM authentication](#5903)

* [Auth — API Server endpoint is public](#3a45)

* [Limited availability](#1eba)

* [Alpha Kubernetes features are disabled](#d062)

* [CronJobs are problematic](#a99b)

* [CronJobs — Backoff limit does not work](#7b6a)

* [CronJobs don’t work well with the Kubernetes network plugin](#b718)

* [Single kube-dns pod by default](#1961)

## Networking

The [Cluster Networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/) changes have certainly been the greatest for us. We migrated from [Flannel](https://github.com/coreos/flannel#flannel) overlay network to [amazon-vpc-cni-k8s](https://github.com/aws/amazon-vpc-cni-k8s#amazon-vpc-cni-k8s). If you don’t have any previous experience with EKS, then it’s very likely that the networking plugin will also be new to you.

The main difference with standard overlay networks (Flannel, Weave, Calico) is that pods and services have actual IPs in the AWS VPC. This leads to a much simpler and more stable network setup — no need to wrap packets for the overlay network.

## Networking — Limited pod capacity per subnet & VPC

The first implication of EKS networking is that the there’s an effective limit to the number of pods you can run in your EKS cluster, depending on the VPC and subnets that you configure for it.

For a quick refresher on AWS network concepts, every VPC and subnet has a range of available IPv4 addresses, defined at creation in the form of a Classless Inter-Domain Routing (CIDR) block; for example, 10.0.0.0/16. VPCs span all of the Availability Zones (AZs) in a region, while subnets are specific to certain AZs. Subnets belong to VPCs and their CIDR blocks have to be a subset of the VPC’s CIDR block.

When you create an EKS cluster, you have to select the subnets that the worker nodes will live in. You can’t change this selection later without creating a new cluster. The sum of IPs available in the subnets’ CIDR blocks defines the effective limit to how many pods you can run in total in that EKS cluster. **So plan ahead when creating/selecting the subnets for your cluster. The subnets and their CIDR blocks set the limit to how large your cluster can grow.**

Note that [there’s a recent change](https://docs.aws.amazon.com/eks/latest/userguide/cni-custom-network.html) that lets you allocate pod IPs from subnets that you didn’t originally configure when creating your EKS cluster, but that requires more complicated configuration.

## Networking — Limited pod capacity per worker node

In addition to limits imposed by the VPC and subnets, there’s also a limit to how many pods can run on any node.

Pod IPs are secondary IPs of worker nodes. Every host can use a limited amount of secondary IPs from its subnet’s CIDR block. Secondary IPs are provided by [Elastic Network Interfaces (ENIs)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html). Each ENI has a certain amount of secondary IPs that it makes available for the host it is attached to. This amount varies by the instance type. Also, the number of secondary ENIs available to an instance varies by its type.

See the instance type specific ENI and IPs-per-ENI limits in [this AWS guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#AvailableIpPerENI). The effective pod count limits for any instance type can be found [here](https://github.com/awslabs/amazon-eks-ami/blob/45a12de008383d128b952e128e7a1da87e4c8142/files/eni-max-pods.txt).

For example t3.nano instances can only run 4 pods and m5.large instances have a limit of 29. So, in addition to thinking about CPU, memory, and storage characteristics, for EKS you also need to **consider IP capacity when selecting your instance types.** You will not be able to use t3.nano instances to run many lightweight applications, for example. If that is your use case, then EKS will probably not work well for you.

## Networking — Kubernetes scheduler is unaware about actual IP availability

Kubernetes doesn’t know if there are any IPs available on a node and therefore cannot take that information into account when deciding which node to schedule a pod onto.

To avoid pods being scheduled onto nodes that don’t have any IPs available, the [--max-pods kubelet flag](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet) is used to limit the amount of pods that can run on any node. The default EKS worker Amazon Machine Image (AMI, discussed below in more detail) [sets the value based on the worker’s instance type.](https://github.com/awslabs/amazon-eks-ami/blob/45a12de008383d128b952e128e7a1da87e4c8142/files/eni-max-pods.txt) **The maximum pod limit works well in most cases and means that the scheduler does not have to be aware about actual IP availability, but there’s a handful of scenarios where it can fail** and cause unexpected behavior.

For example, if attaching an ENI to a worker node fails, then the node may actually have less IPs available than specified with the --max-pods flag. We’ve [seen this happen](https://github.com/aws/amazon-vpc-cni-k8s/pull/194) when AWS [instance metadata service](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html) is unavailable or responds with stale information about attached ENIs. Usually these issues are temporary and an ENI does get attached at some point, even if it fails initially, but for the duration pods will be unable to start on the node, without scheduler being aware of that problem.

Similarly, if all unallocated IPs are in [*cooling mode](https://github.com/aws/amazon-vpc-cni-k8s/blob/74ecf61c60abb645aead82b3d7ad096788484455/docs/cni-proposal.md#pod-ip-address-cooling-period)*, then allocating an IP for a pod can fail for the cooling period (30 seconds). *Cooling mode* simply means that IPs cannot be assigned to new pods shortly after being unassigned from a pod.

Both of these scenarios are exacerbated by CronJobs in clusters with high IP capacity utilization. A topic which is covered in more detail below. But there are simpler failure scenarios as well. For example, if the amazon-vpc-cni-k8s *DaemonSet* pod isn’t running on the node for whatever reason, then pods will also fail to start.

Note that if IP allocation does fail for *any* reason then the pod will stay in *ContainerCreating* state until IP allocation succeeds. This means that if you currently monitor pods in the *Pending* state for scaling decisions, then your monitors will not help you in these scenarios. Similarly, [Cluster Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler#cluster-autoscaler) will [not be able to help you](https://github.com/kubernetes/autoscaler/issues/1366) either.

## Networking — Some pods cannot be accessed from peered networks by default

**If your nodes run in a private subnet and connect to the internet through an AWS NAT Gateway, then it is recommended to set AWS_VPC_K8S_CNI_EXTERNALSNAT to true** in [amazon-vpc-cni-k8s configuration](https://github.com/aws/amazon-vpc-cni-k8s/tree/74ecf61c60abb645aead82b3d7ad096788484455#cni-configuration-variables).

The value defaults to false and causes [issues](https://github.com/aws/amazon-vpc-cni-k8s/issues/53) when connecting to pods from outside of the VPC. Setting it to true also simplifies some of the routing rules and so is recommended if you meet the above criteria.

## Default worker AMI

EKS only manages the Kubernetes control plane for you. You are still responsible for managing the worker nodes. You don’t have to invent the worker setup from scratch, however, AWS documentation on [Getting Started with Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html) provides everything you need to get up and running.

Amazon provides a [recommended AMI for worker nodes](https://github.com/awslabs/amazon-eks-ami). It works pretty well out of the box, but there are some things that may take you by surprise.

## AMI — Based on Amazon Linux 2

The AMI is based on [Amazon Linux 2](https://aws.amazon.com/amazon-linux-2/). Chances are you don’t have previous experience with Amazon Linux 2. Good news is that you don’t have to. You won’t be too far off if you just treat it like CentOS or RHEL.

## AMI — No docker log rotation

One important thing that’s missing in current versions of the worker AMI is log rotation for Docker container logs. **If you don’t [configure Docker log rotation yourself](https://success.docker.com/article/how-to-setup-log-rotation-post-installation), then you’re risking pods being evicted due to disk pressure.**

It [is very likely](https://github.com/awslabs/amazon-eks-ami/pull/74) that you won’t have to configure it yourself with future versions of the AMI, but for now (v24) you do.

## AMI — Docker freezes

When using the AMI as-is, we experienced Docker frequently freezing up indefinitely and becoming unresponsive even to docker ps.

AWS support tells us that the underlying kernel issue with OverlayFS that we tracked this down to has been fixed in the latest version of the AMI (v24). Unfortunately we haven’t been able to verify this, because the issue with disk corruption described in the following section has stopped us from upgrading the AMI we use.

Instead, we run the following command to upgrade Docker to version 18.06. Note that Docker 18.06 [is the latest recommended Docker version](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.12.md#external-dependencies) for new Kubernetes versions, since 1.12.

    yum install -y docker-18.06.1ce-2.amzn2

## AMI — Corrupted disk statistics

Us and other EKS users have experienced [issues](https://github.com/awslabs/amazon-eks-ami/issues/51) where a kernel bug (probably) causes disk usage to be reported in large negative numbers. A 30GB disk may suddenly report having 65TB of free space, with -64TB of it being in use. The wildly inaccurate statistics then cause [kubelet Garbage Collection](https://kubernetes.io/docs/concepts/cluster-administration/kubelet-garbage-collection/) to never kick in, which allows the disk to fill up, rendering the node unusable.

We’ve worked around the issue by sticking to v22 of the AMI. Both older and newer versions of the AMI have exhibited this problem for us. [Others have had success](https://github.com/awslabs/amazon-eks-ami/issues/51#issuecomment-428753356) with **converting the root volume to ext4**.

## Authentication and authorization

Using a novel approach, [EKS provides authentication via AWS IAM](https://docs.aws.amazon.com/eks/latest/userguide/managing-auth.html). Authenticated IAM users are mapped to Kubernetes users and may also be assigned to groups. Granular access to the Kubernetes API is given to these users and groups with [Kubernetes Role Based Access Control (RBAC)](https://kubernetes.io/docs/reference/access-authn-authz/rbac/).

## Auth — RBAC enabled

Our previous Kubernetes clusters didn’t have RBAC enabled and so when migrating to EKS we had to reconfigure many of our applications to allow them to access the Kubernetes API.

As the Kubernetes community is moving towards having RBAC enabled, we would have had to do this work anyway at some point. But you may not realize that this will be necessary for migrating to EKS. So **if you’re not using RBAC today and you’re thinking of moving to EKS, then know that you need to include RBAC configuration in your effort estimations.**

## Auth — AWS IAM authentication

With IAM used for authentication Kubernetes users will have to use [aws-iam-authenticator](https://github.com/kubernetes-sigs/aws-iam-authenticator) for Kubernetes API access. Everyone who needs to access your EKS cluster with kubectl needs to have access to specific IAM Users or Roles with AWS CLI. If you already have infrastructure in place for that, great! If not, **you will have to either provision IAM Users to all of your Kubernetes users or set up federated access to IAM Roles.** Either way, this will likely require significant effort if you haven’t already done this.

There’s also a minor annoyance of Kubernetes access tokens not being cached by *aws-iam-authenticator*. This means that AWS IAM needs to contacted for every kubectl invocation, for example. This is also something that will likely not [be an issue in the future](https://github.com/kubernetes-sigs/aws-iam-authenticator/pull/140).

## Auth — API Server endpoint is public

Your EKS cluster’s API Server’s endpoint is publicly accessible over the internet. This might change in the future, but right now there’s nothing you can do to change that. **The only way to secure access to your EKS cluster is by securing access to IAM used for authentication.** This is wise in any case, but make sure that you handle your access credentials carefully.

## Limited availability

EKS is currently only available in us-west-2, us-east-1, and eu-west-1. Additionally, **it is not guaranteed to be available in all of the AZs in those regions.**

Before putting a lot of effort into configuring your AWS network for EKS, make sure that EKS is actually supported in the AZs that you’re planning to use it in. You can do this by trying to create an EKS cluster with subnets in the AZs you’re planning to use. If [you get an UnsupportedAvailabilityZoneException](https://docs.aws.amazon.com/eks/latest/userguide/troubleshooting.html#ICE), then know that it might take some time until you can use these exact AZs.

## Alpha Kubernetes features are disabled

EKS currently only supports Kubernetes 1.10. If you want to use any Kubernetes features that are still in alpha in that version, then unfortunately you are not able to do that with EKS. So, for example, if you were planning to [use *containerd* directly instead of using Docker](https://kubernetes.io/blog/2018/05/24/kubernetes-containerd-integration-goes-ga/), then you’re out of luck.

Probably the most serious implication of this is that [**Pod Priority and Preemption](https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/) is still in alpha in 1.10 and therefore isn’t available in EKS.** This means that even though [Cluster Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler#cluster-autoscaler) is supported, there is no way for you to guarantee which pods get priority during scaling (and regular) operations. This may be a problem for you, if some pods are critical and shouldn’t be restarted unnecessarily, while others are fine to redistribute at will. [PodDisruptionBudgets](https://kubernetes.io/docs/concepts/workloads/pods/disruptions) are supported however, and can provide some help in these scenarios.

## CronJobs are problematic

You need to be extra vary when you make use of a lot of CronJobs. We have many that run every minute and we keep running into issues that not a lot of other EKS users seem to report.

## CronJobs — Backoff limit does not work

EKS currently only supports Kubernetes 1.10.3 and [**there’s a bug in Kubernetes](https://github.com/kubernetes/kubernetes/issues/62382) that causes .spec.backoffLimit to be ignored for Jobs.** This means that if you don’t configure any other limits for Jobs, then unhealthy Jobs can run rampant creating hundreds of pods if you’ve set .spec.template.spec.restartPolicy to "Never".

One workaround for this issue is to a set a reasonable .spec.activeDeadlineSeconds to Jobs instead. This puts at least some sort of a limit on how many pods can be created for any Job.

## CronJobs don’t work well with the Kubernetes network plugin

CronJobs, especially if they’re configured to run once every minute, can create and delete many pods very often. This puts a lot of stress on the amazon-vpc-cni-k8s network plugin. Many IPs will constantly be in the *cooling mode*, which throws the Kubernetes scheduler off as described above. ENIs will frequently be attached and detached which creates a lot more opportunities for failure.

Unfortunately there is currently not a lot you can do about IPs in *cooling mode*. The safest bet is to ensure you’re comfortably overprovisioned in terms of IP capacity (--max-pods times the number of nodes). Passing a lower than default value to *kubelet* with --max-pods can also make it less likely for the scheduler schedule pods onto nodes where all available IPs are in *cooling mode*.

To avoid ENIs being attached and detached all the time (which we believe has caused some instability for us) **we recommend [setting the WARM_ENI_TARGET](https://github.com/aws/amazon-vpc-cni-k8s/tree/74ecf61c60abb645aead82b3d7ad096788484455#cni-configuration-variables) to something very high, like 20.** The maximum number of ENIs that can be attached to the largest instance types is 15. By setting this value to 20, you force amazon-vpc-cni-k8s to immediately attach all possible ENIs to all worker nodes.

## Single kube-dns pod by default

A single kube-dns pods means that there’s a single point of failure in your cluster. **If something happens to the node running the DNS pod, then all of your applications will be affected.**

A quick solution would be to scale out the kube-dns Deployment.

    kubectl -n kube-system scale --replicas 3 deploy/kube-dns

A more robust solution would be to install a [cluster-proportional-autoscaler](https://github.com/kubernetes-incubator/cluster-proportional-autoscaler) which automatically scales the number of kube-dns pods based on the size of the cluster. This is also the solution [recommended in Kubernetes documentation](https://kubernetes.io/docs/tasks/administer-cluster/dns-horizontal-autoscaling/). Just kubectl apply the following configuration.

<iframe src="https://medium.com/media/f6bb4791bef36b3fb96d3213286bba63" frameborder=0></iframe>

*Originally published at [techmovers.salemove.com](https://techmovers.salemove.com/infrastructure/2018/11/01/Productionproofing+EKS.html) on November 1, 2018.*
