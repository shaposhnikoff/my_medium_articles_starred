
# Manage AWS VPC With Terraform

“person attaching RJ45 cable on white laptop” by rawpixel on Unsplash

If you are new to Terraform and its uses, please read [here](https://medium.com/@mitesh_shamra/infrastructure-as-a-code-with-terraform-e7021bf28d7d) for an introductory understanding.

From the customer’s perspective, when he hits a URL, he receives a response. Underneath that, the request first finds its destination using a mapping of IP address from DNS records, then the request lands on a server which owns this IP and the server does some processing to give a response, which will return to the origin of the request. As we are using Amazon Web Services (AWS), EC2 instance is an obvious choice for us. In production, running just an EC2 instance which can process requests is not enough. We need to have a virtual private cloud, which helps us logically isolate our network from other virtual networks in the AWS Cloud. [Amazon Virtual Private Cloud](https://aws.amazon.com/vpc/) (VPC) service lets us provision a private, isolated section of the AWS Cloud where we can launch our EC2 instance within a virtual network. VPC is not like data center networks, switches, routers but a software-defined network, optimized for packets transfer from ins and outs of and across AWS regions. AWS VPC provides features that help with security using security groups, network access control list, flow logs.

When we create a VPC, we must specify a [CIDR block](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing), for example, 10.0.0.0/16, which is a set of Internet Protocol standards that are used to create unique identifiers for networks and individual devices. After creating a VPC, we can add one or more subnets in each availability zone (one subnet can span over one availability zone only). Each subnet has its own CIDR block, which is a subset of the VPC CIDR block. We need an internet gateway which enables communication over the internet. The internet gateway can be seen as a router that will take packets from our EC2 instance inside the network and forward them to the public internet. If a subnet’s traffic is routed to an internet gateway using route table, the subnet is known as a public subnet. All instances inside a public subnet can access the internet using internet gateway. We will be using a public subnet to let our instance send traffic to the internet.

On an EC2 instance inside a VPC, all inbound traffic is blocked by default. We need to create a security group, which acts as a virtual firewall that controls the traffic from our EC2 instance. We create ingress rules and egress rules. The rules must have allowed ports, the port used for access from_port and the destination port to_port. We need to associate this security group to our EC2 instance to send and receive traffic to and from our instance. In this case, we are going add ingress rule to open port 22 for inbound traffic so we can SSH into our instance and egress rule to let all content go to the public internet. As we want to do SSH to our instance, we need to create a key pair, associate it with EC2 instance and provide the private key when we connect to the instance. Amazon EC2 stores the public key, and we store the private key which we later use to do SSH when needed.

We are going to create all these resources with Terraform. Let’s list out all resources to be created for running EC2 instance inside a virtual private network as that helps us in better understanding.

1. VPC

1. Subnet inside VPC

1. Internet gateway associated with VPC

1. Route Table inside VPC with a route that directs internet-bound traffic to the internet gateway

1. Route table association with our subnet to make it a public subnet

1. Security group inside VPC

1. Key pair used for SSH access

1. EC2 instance inside our public subnet with an associated security group and a generated key pair

As we have a clear picture of what resources we need to create for our production environment with EC2, we can try and identify some of our inputs as variables so that they are easily changed and maintained.

1. CIDR block for VPC

1. CIDR block for subnet which is a subset of CIDR block of VPC

1. Availability zone which is used to create our subnet

1. The public key path which is used in Key pair creation

1. AMI which is used to create EC2 instance

1. Instance type of EC2 instance

1. With AWS, we can start adding tags to track our resources. We are using environment tag.

As always let’s set provider for terraform as AWS with the access key, secret key, and region in which we want to create these resources.

    provider “aws” {
     access_key = “ACCESS_KEY_HERE”
     secret_key = “SECRET_KEY_HERE”
     region = “us-east-2”
    }

Once the provider is set, let’s add all variables which we are going to use while creating our resources.

    variable "cidr_vpc" {
      description = "CIDR block for the VPC"
      default = "10.1.0.0/16"
    }
    variable "cidr_subnet" {
      description = "CIDR block for the subnet"
      default = "10.1.0.0/24"
    }
    variable "availability_zone" {
      description = "availability zone to create subnet"
      default = "us-east-2a"
    }
    variable "public_key_path" {
      description = "Public key path"
      default = "~/.ssh/id_rsa.pub"
    }
    variable "instance_ami" {
      description = "AMI for aws EC2 instance"
      default = "ami-0cf31d971a3ca20d6"
    }
    variable "instance_type" {
      description = "type for aws EC2 instance"
      default = "t2.micro"
    }
    variable "environment_tag" {
      description = "Environment tag"
      default = "Production"
    }

As we have defined our variables, let us use them while creating our resources one by one. We are going to create VPC with defined CIDR block, enable DNS support and DNS hostnames so each instance can have a DNS name along with IP address.

    resource "aws_vpc" "vpc" {
      cidr_block = "${var.cidr_vpc}"
      enable_dns_support   = true
      enable_dns_hostnames = true
      tags {
        "Environment" = "${var.environment_tag}"
      }
    }

![VPC](https://cdn-images-1.medium.com/max/4788/1*ExakV_GA4R8a4Fjw_oPWGA.png)*VPC*

Internet gateway needs to be added inside VPC which can be used by subnet to access the internet from inside.

    resource "aws_internet_gateway" "igw" {
      vpc_id = "${aws_vpc.vpc.id}"
      tags {
        "Environment" = "${var.environment_tag}"
      }
    }

![Internet Gateway (igw)](https://cdn-images-1.medium.com/max/4828/1*WUdrl698g3yGexVVr9Nb0Q.png)*Internet Gateway (igw)*

The subnet is added inside VPC with its own CIDR block which is a subset of VPC CIDR block inside given availability zone.

    resource "aws_subnet" "subnet_public" {
      vpc_id = "${aws_vpc.vpc.id}"
      cidr_block = "${var.cidr_subnet}"
      map_public_ip_on_launch = "true"
      availability_zone = "${var.availability_zone}"
      tags {
        "Environment" = "${var.environment_tag}"
      }
    }

![Subnet](https://cdn-images-1.medium.com/max/4808/1*tjxRIuFJo654rii-hpKJxg.png)*Subnet*

Route table needs to be added which uses internet gateway to access the internet.

    resource "aws_route_table" "rtb_public" {
      vpc_id = "${aws_vpc.vpc.id}"

    route {
          cidr_block = "0.0.0.0/0"
          gateway_id = "${aws_internet_gateway.igw.id}"
      }

    tags {
        "Environment" = "${var.environment_tag}"
      }
    }

![Route Table](https://cdn-images-1.medium.com/max/4808/1*XasqGeC0-hwrK8U7GLhT6A.png)*Route Table*

Once route table is created, we need to associate it with the subnet to make our subnet public.

    resource "aws_route_table_association" "rta_subnet_public" {
      subnet_id      = "${aws_subnet.subnet_public.id}"
      route_table_id = "${aws_route_table.rtb_public.id}"
    }

Once we have our networking setup ready, we need to create an EC2 instance in which we can SSH using port 22. For this, we first need to create a security group which can be attached to our EC2 instance while creation.

    resource "aws_security_group" "sg_22" {
      name = "sg_22"
      vpc_id = "${aws_vpc.vpc.id}"

      ingress {
          from_port   = 22
          to_port     = 22
          protocol    = "tcp"
          cidr_blocks = ["0.0.0.0/0"]
      }

     egress {
        from_port   = 0
        to_port     = 0
        protocol    = "-1"
        cidr_blocks = ["0.0.0.0/0"]
      }

      tags {
        "Environment" = "${var.environment_tag}"
      }
    }

![Security Group](https://cdn-images-1.medium.com/max/4832/1*3DprzCgy33gCexgdMZBg2g.png)*Security Group*

Let’s create a key pair which we are going to use to SSH on our EC2

    resource "aws_key_pair" "ec2key" {
      key_name = "publicKey"
      public_key = "${file(var.public_key_path)}"
    }

Once everything is ready, let us start an EC2 instance within our public subnet with created key pair and security group.

    resource "aws_instance" "testInstance" {
      ami           = "${var.instance_ami}"
      instance_type = "${var.instance_type}"
      subnet_id = "${aws_subnet.subnet_public.id}"
      vpc_security_group_ids = ["${aws_security_group.sg_22.id}"]
      key_name = "${aws_key_pair.ec2key.key_name}"

     tags {
      "Environment" = "${var.environment_tag}"
     }
    }

![EC2 instance](https://cdn-images-1.medium.com/max/4848/1*k8UBXAybE8nAbsQ5vMOS0A.png)*EC2 instance*

With above terraform code, we have our EC2 instance ready. Now we can do SSH using “ec2-user” which is the default user created by AWS for EC2 instance access. Here is the command used for ssh:

    ssh -i [privateKey] ec2-user@[IpAddress]

Replace [privateKey] with the path of your private key and [IpAddress] with IP address of your EC2 instance to SSH inside your instance.

Complete code can be found in this git repository: [https://github.com/MiteshSharma/TerraformVpcInstance](https://github.com/MiteshSharma/TerraformVpcInstance)

***PS: If you liked the article, please support it with claps. Cheers***
