Unknown markup type 10 { type: [33m10[39m, start: [33m165[39m, end: [33m188[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m153[39m, end: [33m164[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m81[39m, end: [33m90[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m108[39m, end: [33m111[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m146[39m, end: [33m149[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m72[39m, end: [33m75[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m8[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m11[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m13[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m14[39m, end: [33m29[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m209[39m, end: [33m212[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m225[39m, end: [33m228[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m50[39m, end: [33m61[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m192[39m, end: [33m203[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m8[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m8[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m8[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m8[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m154[39m, end: [33m163[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m98[39m, end: [33m107[39m }

# Create a secure and H/A VPC on AWS with Terraform

The second article is ready. Now you can provision an autoscaling group into the network, which you will create after this article;
[**Install a secure, fault-tolerant and H/A Wordpress on AWS with Terraform — 2**
*On our first article, we have talked about Secure and H/A networking on AWS;*medium.com](https://medium.com/muhammet-arslan/install-a-secure-fault-tolerant-and-h-a-wordpress-on-aws-with-terraform-2-b29d9a720aee)
> Amazon Virtual Private Cloud (Amazon VPC) lets you provision a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define. You have complete control over your virtual networking environment, including selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways. You can use both IPv4 and IPv6 in your VPC for secure and easy access to resources and applications. — [https://aws.amazon.com/vpc/](https://aws.amazon.com/vpc/)

AWS VPC consists of the following components,

* Subnets

* Route Tables

* Internet Gateways

* Egress-Only Internet Gateways

* DHCP Options Sets

* DNS

* Elastic IP Addresses

* VPC Endpoints

* NAT

* VPC Peering

Of course, we don’t need some of the above components to setup basic networking like peering.

For highly-available networking, you need to have at least 2 availability zone to put your resources in and hopefully, all AWS regions have at least 2 AZs. I’ll use frankfurt(eu-central-1) for post.

**What we will have the end of the session?**

* 1 VPC with enough IPs

* 6 Subnets (3 public and 3 private)

* 4 Routetables (1 for public and 3 for private)

* 1 Internet Gateway

* 3 NATs

![](https://cdn-images-1.medium.com/max/2802/1*jtCf-9EjTuOKEfAGLN5MXA.png)

## **VPC**

A virtual private cloud (VPC) is a virtual network dedicated to your AWS account. It is logically isolated from other virtual networks in the AWS Cloud. You can launch your AWS resources, such as Amazon EC2 instances, into your VPC.

When you create a VPC, you must specify a range of IPv4 addresses for the VPC in the form of a Classless Inter-Domain Routing (CIDR) block; for example, 10.0.0.0/16. This is the primary CIDR block for your VPC. For more information about CIDR notation, see [RFC 4632](https://tools.ietf.org/html/rfc4632).

It was much more important to choose CIDR range for VPC before AWS presented the EDIT CIDR feature. But again, it’s not so easy to add multiple CIDR by the configuration aspect of resources. So it’ll be much more logical to choose CIDR before.

On the following table, you can see the Masked IPs details.

![](https://cdn-images-1.medium.com/max/2000/1*vNHbsbN1uZnfy-372fa_Rw.png)

When you create a VPC, you must specify an IPv4 CIDR block for the VPC. The allowed block size is between a /16 netmask (65,536 IP addresses) and /28 netmask (16 IP addresses). After you've created your VPC, you can associate secondary CIDR blocks with the VPC. For more information, see [Adding IPv4 CIDR Blocks to a VPC](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html#vpc-resize).

When you create a VPC, AWS recommends that you specify a CIDR block (of /16 or smaller) from the private IPv4 address ranges as specified in [RFC 1918](http://www.faqs.org/rfcs/rfc1918.html):

* 10.0.0.0 - 10.255.255.255 (10/8 prefix)

* 172.16.0.0 - 172.31.255.255 (172.16/12 prefix)

* 192.168.0.0 - 192.168.255.255 (192.168/16 prefix)

For this article, I’ll use **10.0.0.0/20** for VPC CIDR.

![](https://cdn-images-1.medium.com/max/2802/1*XHnCGPrp7wgytsZ4_gypHw.png)

A VPC spans all the Availability Zones in the region

## **Subnet**

The CIDR block of a subnet can be the same as the CIDR block for the VPC (for a single subnet in the VPC), or a subset of the CIDR block for the VPC (for multiple subnets). The allowed block size is between a /28 netmask and /16 netmask. If you create more than one subnet in a VPC, the CIDR blocks of the subnets cannot overlap.

For example, if you create a VPC with CIDR block, 10.0.0.0/20, it supports 4096 IP addresses. You can break this CIDR block into 6 subnets;

* 10.0.0.0/23 — public subnet 1a — 512 IP Address

* 10.0.2.0/23 — public subnet 1b — 512 IP Address

* 10.0.4.0/23 — public subnet 1c — 512 IP Address

* 10.0.10.0/23 — private subnet 1a — 512 IP Address

* 10.0.12.0/23 — private subnet 1b — 512 IP Address

* 10.0.14.0/23 — private subnet 1c — 512 IP Address

Now, you will have 512 IP Address on each block and 2 more slots to might be defined in the future.

BUT!

The first four IP addresses and the last IP address in each subnet CIDR block are not available for you to use, and cannot be assigned to an instance. For example, in a subnet with CIDR block 10.0.0.0/23, the following five IP addresses are reserved:

* 10.0.0.0: Network address.

* 10.0.0.1: Reserved by AWS for the VPC router.

* 10.0.0.2: Reserved by AWS. The IP address of the DNS server is always the base of the VPC network range plus two; however, AWS also reserves the base of each subnet range plus two. For VPCs with multiple CIDR blocks, the IP address of the DNS server is located in the primary CIDR. For more information, see [Amazon DNS Server](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_DHCP_Options.html#AmazonDNS).

* 10.0.0.3: Reserved by AWS for future use.

* 10.0.1.255: Network broadcast address. AWS does not support broadcast in a VPC, therefore AWS reserves this address.

![](https://cdn-images-1.medium.com/max/2802/1*wnd4ix_v1fgdoENnvbtofg.png)

## Routetable

A *route table* contains a set of rules, called *routes*, that are used to determine where network traffic is directed.

Each subnet in your VPC must be associated with a route table; the table controls the routing for the subnet. A subnet can only be associated with one route table at a time, but you can associate multiple subnets with the same route table.

***Why do we need 3 private routetable but 1 public routetable?***

Because of NAT and IGW difference. NAT can be placed in one subnet only. So for H/A networking, we need 3 NAT. On each private route table, we will route 0.0.0.0/0 to its own NAT ID.

But IGW is attached to the whole vpc but not subnet, so 1 public routetable is enough for routing 0.0.0.0/0 to IGW ID.

![](https://cdn-images-1.medium.com/max/2802/1*amVJq7LHxLUGz25yHhZ04A.png)

## Internet Gateway

An internet gateway is a horizontally scaled, redundant, and highly available VPC component that allows communication between instances in your VPC and the internet. It, therefore, imposes no availability risks or bandwidth constraints on your network traffic.

An internet gateway serves two purposes: to provide a target in your VPC route tables for internet-routable traffic and to perform network address translation (NAT) for instances that have been assigned public IPv4 addresses.

![](https://cdn-images-1.medium.com/max/2802/1*wbNd_qMqz16EwFBW-08WZw.png)

## NAT Gateway

You can use a network address translation (NAT) gateway to enable instances in a private subnet to connect to the internet or other AWS services, but prevent the internet from initiating a connection with those instances. For more information about NAT, see [NAT](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat.html).

**IMPORTANT!**

You are charged for creating and using a NAT gateway in your account. NAT gateway hourly usage and data processing rates apply. Amazon EC2 charges for data transfer also apply. For more information, see [Amazon VPC Pricing](http://aws.amazon.com/vpc/pricing/).

NAT gateways are not supported for IPv6 traffic — use an egress-only internet gateway instead.

* Nat GW should be located on a public subnet

* You must associate an elastic IP for NAT GW.

* If you have resources in multiple Availability Zones and they share one NAT gateway, in the event that the NAT gateway’s Availability Zone is down, resources in the other Availability Zones lose internet access. To create an Availability Zone-independent architecture, create a NAT gateway in each Availability Zone and configure your routing to ensure that resources use the NAT gateway in the same Availability Zone.

![](https://cdn-images-1.medium.com/max/2802/1*-wuxdODUBq43o5zxP9RSfg.png)

As you see on the above diagram, Private Subnets’ routetable has an entry to send 0.0.0.0/0 (whole) traffic to NAT and because NAT is on the public subnet, it will use public routetable to go internet with IGW.

Okay for a secure and H/A networking, we are ready to terraform. Terraform has a great module for AWS to set up VPC and sub-components.

    resource "aws_eip" "nat" {
      count = 3
      vpc = true
    }

    module "vpc" {
      source = "terraform-aws-modules/vpc/aws"

    name = "dev-vpc"
      cidr = "10.0.0.0/20"

    azs             = ["eu-central-1a", "eu-central-1b", "eu-central-1c"]
      public_subnets = ["10.0.0.0/23", "10.0.2.0/23", "10.0.4.0/23"]
      private_subnets  = ["10.0.10.0/23", "10.0.12.0/23", "10.0.14.0/23"]

    enable_nat_gateway = true
      single_nat_gateway  = false
      reuse_nat_ips       = true  
      external_nat_ip_ids = "${aws_eip.nat.*.id}"

    enable_dns_hostnames = true
      enable_dns_support   = true

    tags = {
        Terraform = "true"
        Environment = "dev"
      }
    }

You can see, first, we have created 3 Elastic IPs and pass them to aws-vpc module of Terraform.

Now it’s time to provision your resources on your secure and H/A network.
