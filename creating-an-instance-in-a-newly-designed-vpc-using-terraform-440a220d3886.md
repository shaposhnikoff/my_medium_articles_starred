Unknown markup type 10 { type: [33m10[39m, start: [33m113[39m, end: [33m127[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m156[39m, end: [33m169[39m }

# Creating an instance in a newly designed VPC using terraform

Photo by Samuel Zeller on Unsplash

**Understanding Virtual Private Cloud:**

A VPC is a logically isolated virtual network, where AWS nodes like EC2, load balancers and so on can be launched. A VPC provides the following functionalities:

* Logical Isolation of resources in your account from the rest of the Cloud

* Routing traffic to and from your private network

* Controlling access to the resources in your network

**Components and a brief description of AWS VPC:**

* VPC CIDR Block — Helps in providing private IP addresses to the nodes

* subnet — Subset of the VPC where the instances can be launched

* Internet gateway — Connects the instances to the internet

* Route Table — Place to specify the allowed routes for outbound network traffic

* Security group — Restricts the inbound traffic to the instance. It is stateful and hence the return traffic to the ports in the security group is automatically allowed

* Access Control Lists — Acts as a firewall to restrict access in and out of a subnet

Every AWS region comes with a default VPC and a maximum of 5 custom VPC’s can be created per region.

If networking planning isn’t really a crucial part of your infrastructure, that is being deployed in AWS it is ideal to choose the default VPC.

Any of the below factors would be a reason to set up a new VPC rather than using the default VPC

* If system architecture requires a few public-facing nodes and a few of them not publicly accessible. (The default VPC ensures that all the instances in it are publicly accessible.)

* If you require a specific size of CIDR block for your network design. (The default VPC comes with a CIDR block of172.31.0.0/16)

* If the size of subnets in the VPC should not be the same. (The default VPC comes with a default subnet per availability zone of the region with a CIDR Block172.31.0.0/20 , having an effective number of 4091 IP addresses per subnet.)

* If you want to tailor your network exactly how you want it

Once you are convinced that you need to create a new VPC for your infrastructure needs, the following resources are to be created in Terraform

### Brief introduction of the needful Terraform resources:

*aws_vpc:* Creates a VPC resource. A VPC spans All it needs for creating a VPC is the cidr_block. The CIDR Block helps in deciding the private address range for the nodes in the VPC. The VPC spans over multiple availability zones of a region.

*aws_subnet:* It is a subset of the aws_vpc which can have its own route table and security_groups attached. A fleet of similar nodes can be placed in the same subnet. The requirement parameters would be the cidr_block which should be a subset of the VPC cidr_block and the vpc_id of which it would be a part of. The subnet cannot be a part of more than one availability zone.

*aws_internet_gateway:* It is really crucial in providing access to the external network for all the nodes in the VPC.

*aws_default_route_table:* Provides a resource to manage a Default VPC Routing Table. It doesn’t create a new resource but adapts the existing default route table for every VPC that gets created. It deletes the existing entries in the default route table and configures it based on the input received from Terraform. It requires two parameters default_route_table_id and the route which includes the cidr_block and the gateway_id. Other options such as nat_gateway_id, network_interface_id can also be used instead of gateway_id based on where the traffic from the VPC has to be routed.
> The default route, mapping the VPC’s CIDR block to “local”, is created implicitly and cannot be specified. — [Terraform Documentation](https://www.terraform.io/docs/providers/aws/r/default_route_table.html)

*aws_security_group*: Security group are applied at the instance level. It is used to specify which type of traffic can be allowed to the instance and through which ports. While creation it requires the vpc_id in which the instance is created and the ingress outgress rules. Every VPC when created is attached to a default security group which allows all incoming and outgoing traffic.

*aws_eip: *Creates an Elastic IP which can be linked with the instance. There isn’t any required parameter for this resource.

*aws_eip_association:* Associates the elastic IP with the instance that is being created. It requires the instance_id of the instance being created and the association_id of the elastic IP that is created.

*aws_instance:* Creates the instance. Requires the AMI from which the instance has to be created and the instance_type.

**Modularizing Resources:**

It is a good practice to modularize resources in Terraform. A module can be created with the resources that can be grouped together.

In this case, the VPC, subnet, internet_gateway, and default_route_table can be grouped together.

The directory tree for modules can be as follows

    -- <environment> (optional)
       -- main.tf # place where the modules can be initialized/called
       -- variables.tf
       -- terraform.tfvars

    -- modules (folder)
      --vpc (folder)
        -- vpc.tf
        -- variables.tf
        -- output.tf

Each module will contain variables.tf which includes a declaration of variables that are needed to be passed into the resources of the module

    ## variables.tf of module
    variable "subnet" {
      default = "10.0.0.0/24"
    }

    variable "cidr_block" {
      default = "10.0.0.0/16"
    }

    variable "env" {}

The variables declared above will be used as parameters in the .tf file. The .tf file is specific to terraform 0.12 and wouldn’t work with older versions unless the variables which are declared with first-class expression syntax are changed to older string interpolation syntax

    ## vpc.tf
    resource "aws_vpc" "vpc" {
      cidr_block           = var.cidr_block
      tags = {
        Name = "${var.env}_vpc"
        Env  = var.env
      }
    }

    resource "aws_subnet" "subnet" {
      vpc_id                  = aws_vpc.vpc.id
      cidr_block              = var.subnet
      map_public_ip_on_launch = "true"

    tags = {
        Name = "${var.env}_subnet"
        Env  = var.env
      }
    }

    resource "aws_internet_gateway" "gw" {
      vpc_id = aws_vpc.vpc.id

    tags = {
        Name = "${var.env}_gw"
        Env  = var.env
      }
    }
    resource "aws_default_route_table" "route_table" {
      default_route_table_id = aws_vpc.vpc.default_route_table_id

    route {
        cidr_block = "0.0.0.0/0"
        gateway_id = aws_internet_gateway.gw.id
      }

    tags = {
        Name = "default route table"
        env  = var.env
      }
    }

Internet Gateway — The typical pitfall. The default VPC comes with the Internet gateway and when creating the instance in that VPC, the user doesn’t have to worry about creating an internet gateway or the route table entry to route traffic in the VPC to the internet gateway. I missed this part when I was creating a new VPC and had to troubleshoot for some time to figure out why I am not getting any responses back from the server in the VPC.

After the VPC creation is done, for creating the instance, we need the subnet ID of the VPC to be passed into the resource

    resource "aws_instance" "VM-in-new-vpc" {
      ami                    = "ami-221ea342" #id of desired AMI
      instance_type          = "m3.medium"
      subnet_id              = var.subnet_id
      vpc_security_group_ids = var.vpc_security_group_ids # list
        Env = "test"
      }
    }

If the AMI needs to be created from ebs_block_device than the resources ami and the instance can be grouped together into a module and can be re-used if required.

The following example illustrates the module initialization

    # main.tf inside <environment> folder
    module "example_vpc" {
      source     = "../modules/vpc"
      env        = var.env
      subnet     = "10.0.0.0/24"
      cidr_block = "10.0.0.0/16"

    }

The name of the module has to be unique per state file. This lets us re-use the same module if multiple VPCs have to be created, it has to be ensured that the module names are different.

The CIDR blocks have to be different if the VPC is created in the same region. If the VPC has to be created in a different region, the same module can be re-used but the provider information has to be included, which contains the region information.

    # main.tf inside <environment> folder
    module "example_vpc" {
      source     = "../modules/vpc"
      env        = var.env
      providers = {
        aws = aws.frankfurt
      }

    }

    # provider.tf
    provider "aws" {
      version    = "~> 2.22.0"
      region     = "eu-central-1"
      access_key = var.aws_access_key
      secret_key = var.aws_secret_key
      alias      = "frankfurt"
    }
