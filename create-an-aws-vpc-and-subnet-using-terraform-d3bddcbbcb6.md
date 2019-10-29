
# Create an AWS VPC, Subnet, Security Group, and Network ACL using Terraform

For those learning AWS/AWS CLI Terraform is a tool for building infrastructure with various technologies including AWS.

Here is a simple document on how to use Terraform to build an AWS VPC along with a Subnet for the VPC.

I created the two following Terraform tf files vpc.tf and variables.tf.

vpc.tf is the actual configuration file and the variables are declared within the variables.tf file. If you have the AWS Command Line Interface Access Key and Secret Key exported in your .bashrc file you will not need to define them in the provider “aws” section. Terraform does not allow variable interpolation for the resource name. For example the following will produce an error during the terraform plan execution:

resource “aws_vpc” “${var.vpcName}” {

So far, this is the only shortcoming I have found with using Terraform for AWS deployment. In addition, the Medium blog site does not allow me to indent my code which ruins some of the readability.

## **vpc.tf:**

    # vpc.tf 
    # Create VPC/Subnet/Security Group/ACL

    provider "aws" {
            region     = "${var.region}"
    } # end provider

    # create the VPC
    resource "aws_vpc" "My_VPC" {
      cidr_block           = "${var.vpcCIDRblock}"
      instance_tenancy     = "${var.instanceTenancy}" 
      enable_dns_support   = "${var.dnsSupport}" 
      enable_dns_hostnames = "${var.dnsHostNames}"

    tags {
        Name = "My VPC"
      }
    } # end resource

    # create the Subnet
    resource "aws_subnet" "My_VPC_Subnet" {
      vpc_id                  = "${aws_vpc.My_VPC.id}"
      cidr_block              = "${var.subnetCIDRblock}"
      map_public_ip_on_launch = "${var.mapPublicIP}" 
      availability_zone       = "${var.availabilityZone}"

    tags = {
       Name = "My VPC Subnet"
      }
    } # end resource

    # Create the Security Group
    resource "aws_security_group" "My_VPC_Security_Group" {
      vpc_id       = "${aws_vpc.My_VPC.id}"
      name         = "My VPC Security Group"
      description  = "My VPC Security Group"

    ingress {
        cidr_blocks = "${var.ingressCIDRblock}"  
        from_port   = 22
        to_port     = 22
        protocol    = "tcp"
      }

    tags = {
            Name = "My VPC Security Group"
      }

    } # end resource

    # create VPC Network access control list
    resource "aws_network_acl" "My_VPC_Security_ACL" {
      vpc_id = "${aws_vpc.My_VPC.id}"
      subnet_ids = [ "${aws_subnet.My_VPC_Subnet.id}" ]

    # allow port 22
      ingress {
        protocol   = "tcp"
        rule_no    = 100
        action     = "allow"
        cidr_block = "${var.destinationCIDRblock}" 
        from_port  = 22
        to_port    = 22
      }

    # allow ingress ephemeral ports 
      ingress {
        protocol   = "tcp"
        rule_no    = 200
        action     = "allow"
        cidr_block = "${var.destinationCIDRblock}"
        from_port  = 1024
        to_port    = 65535
      }

    # allow egress ephemeral ports
      egress {
        protocol   = "tcp"
        rule_no    = 100
        action     = "allow"
        cidr_block = "${var.destinationCIDRblock}"
        from_port  = 1024
        to_port    = 65535
      }

    tags {
        Name = "My VPC ACL"
      }

    } # end resource

    # Create the Internet Gateway
    resource "aws_internet_gateway" "My_VPC_GW" {
      vpc_id = "${aws_vpc.My_VPC.id}"

    tags {
            Name = "My VPC Internet Gateway"
        }
    } # end resource

    # Create the Route Table
    resource "aws_route_table" "My_VPC_route_table" {
        vpc_id = "${aws_vpc.My_VPC.id}"

    tags {
            Name = "My VPC Route Table"
        }
    } # end resource

    # Create the Internet Access
    resource "aws_route" "My_VPC_internet_access" {
      route_table_id        = "${aws_route_table.My_VPC_route_table.id}"
      destination_cidr_block = "${var.destinationCIDRblock}"
      gateway_id             = "${aws_internet_gateway.My_VPC_GW.id}"
    } # end resource

    # Associate the Route Table with the Subnet
    resource "aws_route_table_association" "My_VPC_association" {
        subnet_id      = "${aws_subnet.My_VPC_Subnet.id}"
        route_table_id = "${aws_route_table.My_VPC_route_table.id}"
    } # end resource

    # end vpc.tf

## variables.tf:

    # variables.tf

    variable "region" {
     default = "us-east-1"
    }

    variable "availabilityZone" {
            default = "us-east-1a"
    }

    variable "instanceTenancy" {
     default = "default"
    }

    variable "dnsSupport" {
     default = true
    }

    variable "dnsHostNames" {
            default = true
    }

    variable "vpcCIDRblock" {
     default = "10.0.0.0/16"
    }

    variable "subnetCIDRblock" {
            default = "10.0.1.0/24"
    }

    variable "destinationCIDRblock" {
            default = "0.0.0.0/0"
    }

    variable "ingressCIDRblock" {
            type = "list"
            default = [ "0.0.0.0/0" ]
    }

    variable "mapPublicIP" {
            default = true
    }

    # end of variables.tf

Within the directory that the two files are located issue:

**terraform init**

The init argument will initialize the environment.

Then issue:

**terraform plan -out vpc.plan**

The plan argument will syntax check the files and prepare the deployment.

Deploy the VPC:

**terraform apply vpc.plan**

This will deploy the AWS VPC. To view data about the VPC/Subnet/Security Group from your local Linux box execute:

**terraform show**

To destroy the VPC execute:

**terraform destroy**

Deploying an AWS VPC can be pretty simple with terraform. To test the VPC create a new instance with the newly defined security group and subnet.
