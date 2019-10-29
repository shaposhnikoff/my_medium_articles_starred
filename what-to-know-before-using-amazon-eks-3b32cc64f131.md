Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m83[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m34[39m, end: [33m42[39m }

# What to Know Before Using Amazon EKS



Amazon EKS has brought a managed Kubernetes cluster to Amazon. The other cloud providers already offer their own alternatives. Azure has offered AKS for the past year and Google Cloud Platform was the first to offer the service.

I’ve been working with EKS since it became generally available and my journey has led me down a few pretty odd paths. It solved a lot of problems for us, hard problems that would require both organizational and engineering solutions. Alas, as with all trades, there is a price. From here on in, I’ll be listing out some of the odd quirks we’ve found with EKS and what to expect.

## It isn’t as sophisticated as GKE

![](https://cdn-images-1.medium.com/max/2000/0*VqQPgsuAeKzwoKAr)

I think this is common knowledge, but it’s worth calling out. When you sign into the AWS console and take a look at the EKS cluster, you get some very basic information. Your cluster, some cloudwatch metrics and some cursory information about your cluster’s “state”.

Compare that against GKE. All of your services are listed, with their applications. Using just the UI you can:

* Find out which installations are running on which node

* Deploy new services from the GCP marketplace

* Perform node upgrades

GCP has a much more feature-rich offering that might create a kinder learning curve for your engineers if K8s is new to your organization.

### But be careful here…

The good thing about the bare-bones offering of EKS is that your engineers learn the K8s tooling very quickly. Of course, it is the only way to actually get anything done. The abstraction layers introduced by GCP will make their lives a little easier, but it might introduce some bad habits (like button clicking changes, rather than scripting them). Worth considering!

## EKS consumes a ton of IP addresses

![An actual picture of EKS looking at my available IPs](https://cdn-images-1.medium.com/max/2000/0*g6BJy3U5iCc1iZgr.png)*An actual picture of EKS looking at my available IPs*

Typically, nodes (EC2 instances in AWS) will receive an IP address from their subnet. Then, pods will receive an internal docker IP address. This serves to do two things:

* It increases the limit for the number of pods that can be ran on a node

* It doesn’t consume as many IP addresses

When Amazon was implementing their EKS CNI (the networking layer for Kubernetes), they wanted to ensure compatibility with other AWS networking services. The docker private IPs would not work with some of their solutions, like their VPC flow logs. As a result, they made a choice. Every pod gets an IP address from the subnet.

If you’re running a subnet with a limited number of IP addresses (around a few hundred available IPs, for example), you’re going to very quickly run into trouble.

Let me give you an example. Let’s say you have a cluster with five nodes. That’s five IP addresses, understandably. Then you’ve got the pods that Kubernetes needs to deploy in order to run. Let’s call that another 10 IP addresses. If you deploy a single daemonset (a service that runs one pod on every node, for example for log aggregation), you’ve got another five IPs taken up.

So with just an app and some log aggregation, you’ve consumed 17 IP addresses. Throw some load balancers into that and, of course, multiple availability zones.

### What can be done about this?

AWS VPN supports a secondary CIDR range (100.64.0.0/10). This is actually for [carrier-grade NAT](https://en.wikipedia.org/wiki/Carrier-grade_NAT) but if you don’t see that being a networking problem, it opens up some four million IP addresses for you. When you install EKS however, you’ll need to follow the [AWS documentation](https://eksworkshop.com/advanced-networking/secondary_cidr/) for using a secondary CIDR range. This is harder to do once you’re deployed and live so it’s something you should consider upfront.

## Direct Connect, VPC Peering or Transit VPC

If you’re using any of the above services, you may run into a problem with *SNAT. *SNAT allows nodes in a private subnet to communicate with the internet, by translating the network address into the internet gateway IP that they use.

In a typical setup, this is fine, but if your AWS account has any of the above services, your nodes may not be talking directly to an internet gateway. They may be traveling through a NAT gateway. In this case, SNAT is going to cause some bizarre networking issues for you. The symptoms we’ve seen are:

* No egress out to the internet

* Nodes in the same subnet unable to communicate with one another

* Nodes from another private subnet unable to communicate with the nodes in another private subnet

As part of the EKS CNI (remember that pesky networking implementation?), your Kubernetes cluster will translate the IP address of your pod to the IP address of your internet gateway. In the absence of an internet gateway, this falls apart.

### However, it can be solved with one command!

If you are running in this type of environment, the fix for this is trivial.

    kubectl set env daemonset -n kube-system aws-node AWS_VPC_K8S_CNI_EXTERNALSNAT=true

This will instruct the EKS CNI implementation to not perform SNAT, and rely on whatever egress solution you’re using to do it for you. When we ran into this problem, we ran this command and everything magically began to work again.

## You need to patch more than an AMI

When you’re updating your Kubernetes cluster, it is tempting to think of the classic components you need to update:

* The API version, using either Terraform or a button click in the AWS console

* The node AMI version, by updating your ASG run configuration.

BUT, you’d be missing out on some other quite important components…

* The EKS CNI pod version, known as aws-node

* The Core DNS pod version

* The version of Kube Proxy

Whenever you update your Kubernetes AMI version and API version, you *need *to update the CNI, Core DNS and Kube Proxy pods. Fortunately, Amazon has provided [a lovely table](https://docs.aws.amazon.com/eks/latest/userguide/update-cluster.html) to let you know which version is compatible with which. Make sure these upgrades are baked into your node upgrade strategy from day one, to avoid any headaches in the future.

## With all that said…

Every technology has its pitfalls and quirks. The problems that EKS solves are enormous. If you need a gentle ride into Kubernetes, EKS can help with that. If you need a highly available K8s API, EKS can help with that. If you need AMIs for the new version of Kubernetes, EKS has got you covered. These are some tricky problems that could slow down any adoption.

If you’re looking for a gentler learning curve and many of the hard problems fixed for you, EKS can help you out. As with all solutions, you simply need to be aware of the small print.

I regularly [tweet](https://twitter.com/chris_cooney) about K8s, DevOps, Security, “Agile” and much more.
