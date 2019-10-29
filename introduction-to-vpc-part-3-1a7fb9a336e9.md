Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m70[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m18[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m20[39m }

# Introduction to VPC — Part 3

In the third part of this series, let create VPC one more but this time using terraform or more precisely let’s build VPC terraform module

*Part 1: [https://medium.com/p/cbca3b189329](https://medium.com/p/cbca3b189329/edit)*

*Part 2: [https://medium.com/p/b0cb4ab41e8f](https://medium.com/p/b0cb4ab41e8f/edit)*

*In the second part of the series we learned that*

* *Once we create the VPC, these things created for us by-default*

    ** Network Access Control List(NACL)*

    ** Security Group*

    ** Route Table*

* *We need to take care of*

    ** Internet Gateways
    * Subnets
    * Custom Route Table*

*But the bad news is as we are creating this via terraform we need to create all these things manually but this is just one time task, later on, if we need to build one more VPC we just need to call this module with some minor changes(eg: Changes in CIDR Range, Subnet) true Infrastructure as a Code(IAAC)*

*This is how my terraform VPC module structure look like*

    *$ tree*

    *├── main.tf*

    *├── vpc_networking*

    *│   ├── main.tf*

    *│   ├── outputs.tf*

    *│   └── variables.tf*

    *├── outputs.tf*

    *├── terraform.tfvars*

    *└── variables.tf*

* *So the first step is to create a data resource, what data resource did is to query/list all the AWS available Availablity zone in a given region and then allow terraform to use those resource.*

    *networking/main.tf*

    *# Declare the data source
    data "aws_availability_zones" "available" {}*

*For more info*
[**AWS: aws_availability_zones - Terraform by HashiCorp**
*Provides a list of Availability Zones which can be used by an AWS account.*www.terraform.io](https://www.terraform.io/docs/providers/aws/d/availability_zones.html)

* ***Now it’s time to create VPC***

    *# Creating VPC*

    *resource "aws_vpc" "main" {
      cidr_block = "${var.vpc_cidr}"
      enable_dns_hostnames = true
      enable_dns_support = true
    
      tags {
        Name = "my-test-vpc"
      }
    }*

* [*enable_dns_support](https://www.terraform.io/docs/providers/aws/r/vpc.html#enable_dns_support) - (Optional) A boolean flag to enable/disable DNS support in the VPC. Defaults true. Amazon provided DNS server(**AmazonProvidedDNS)** can resolve Amazon provided private DNS hostnames, that we specify in a private hosted zones in Route53.*

* [*enable_dns_hostnames](https://www.terraform.io/docs/providers/aws/r/vpc.html#enable_dns_hostnames) - (Optional) A boolean flag to enable/disable DNS hostnames in the VPC. Defaults false. This will ensure that instances that are launched into our VPC receive a DNS hostname.*

*For more info*
[**AWS: aws_vpc - Terraform by HashiCorp**
*Provides an VPC resource.*www.terraform.io](https://www.terraform.io/docs/providers/aws/r/vpc.html)

* ***Next step is to create an internet gateway***

    *# Creating Internet Gateway
    resource "aws_internet_gateway" "gw" {
      vpc_id = "${aws_vpc.main.id}"
    
      tags {
        Name = "my-test-igw"
      }
    }*

*For more info*
[**AWS: aws_internet_gateway - Terraform by HashiCorp**
*Provides a resource to create a VPC Internet Gateway.*www.terraform.io](https://www.terraform.io/docs/providers/aws/r/internet_gateway.html)

* ***Next step is to create Public Route Table***

    *# Public Route Table
    
    resource "aws_route_table" "public_route" {
      vpc_id = "${aws_vpc.main.id}"
    
      route {
        cidr_block = "0.0.0.0/0"
        gateway_id = "${aws_internet_gateway.gw.id}"
      }
    
      tags {
        Name = "my-test-public-route"
      }
    }*

*For more info*
[**AWS: aws_route_table - Terraform by HashiCorp**
*Provides a resource to create a VPC routing table.*www.terraform.io](https://www.terraform.io/docs/providers/aws/r/route_table.html)

* *Now it’s time to create Private Route Table. If the subnet is not associated with any route by default it will be associated with Private Route table*

    *# Private Route Table
    
    resource "aws_default_route_table" "private_route" {
      default_route_table_id = "${aws_vpc.main.default_route_table_id}"
    
      tags {
        Name = "my-test-private-route"
      }
    }*

* *Next step is to create Public Subnet*

    *#Public Subnet
    
    resource "aws_subnet" "public_subnet" {
      count = 2
      vpc_id     = "${aws_vpc.main.id}"
      cidr_block = "${var.public_cidrs[count.index]}"
      map_public_ip_on_launch = true
      availability_zone = "${data.aws_availability_zones.available.names[count.index]}"
    
      tags {
        Name = "my-test-public-subnet.${count.index + 1}"
      }
    }*

* *Private Subnet*

    *#Private Subnet
    
    resource "aws_subnet" "private_subnet" {
      count = 2
      vpc_id     = "${aws_vpc.main.id}"
      cidr_block = "${var.private_cidrs[count.index]}"
      map_public_ip_on_launch = true
      availability_zone = "${data.aws_availability_zones.available.names[count.index]}"
    
      tags {
        Name = "my-test-private-subnet.${count.index + 1}"
      }
    }*
[**AWS: aws_subnet - Terraform by HashiCorp**
*Provides an VPC subnet resource.*www.terraform.io](https://www.terraform.io/docs/providers/aws/r/subnet.html)

* *Next step is to create a route table association*

    *# Associate Public Subnet with Public Route Table
    
    resource "aws_route_table_association" "public_subnet_assoc" {
      count = "${aws_subnet.public_subnet.count}"
      route_table_id = "${aws_route_table.public_route.id}"
      subnet_id = "${aws_subnet.public_subnet.*.id[count.index]}"
      depends_on     = ["aws_route_table.public_route", "aws_subnet.public_subnet"]
    
    }*

    *# Associate Private Subnet with Private Route Table
    
    resource "aws_route_table_association" "private_subnet_assoc" {
      count = "${aws_subnet.private_subnet.count}"
      route_table_id = "${aws_default_route_table.private_route.id}"
      subnet_id = "${aws_subnet.private_subnet.*.id[count.index]}"
      depends_on     = ["aws_default_route_table.private_route", "aws_subnet.private_subnet"]
    }*
[**AWS: aws_route_table_association - Terraform by HashiCorp**
*Provides a resource to create an association between a subnet and routing table.*www.terraform.io](https://www.terraform.io/docs/providers/aws/r/route_table_association.html)

* *Security Group*

    *# Security Group
    
    resource "aws_security_group" "test_sg" {
      vpc_id = "${aws_vpc.main.id}"
      ingress {
        from_port = 22
        protocol = "tcp"
        to_port = 22
        cidr_blocks = ["0.0.0.0/0"]
      }
    
      egress {
        from_port = 0
        protocol = "-1"
        to_port = 0
        cidr_blocks = ["0.0.0.0/0"]
      }
    }*
[**AWS: aws_security_group - Terraform by HashiCorp**
*Provides details about a specific Security Group*www.terraform.io](https://www.terraform.io/docs/providers/aws/d/security_group.html)

* *Now let’s test it*

* *Initialize a Terraform working directory*

    *$ terraform init*

    ***Initializing modules...***

    *- module.vpc_networking*

    *Getting source "./networking"*

    ***Initializing provider plugins...***

    *- Checking for available provider plugins on https://releases.hashicorp.com...*

    *- Downloading plugin for provider "aws" (1.51.0)...*

    *The following providers do not have any version constraints in configuration,*

    *so the latest version was installed.*

    *To prevent automatic upgrades to new major versions that may contain breaking*

    *changes, it is recommended to add version = "..." constraints to the*

    *corresponding provider blocks in configuration, with the constraint strings*

    *suggested below.*

    ** provider.aws: version = "~> 1.51"*

    ***Terraform has been successfully initialized!***

    *You may now begin working with Terraform. Try running "terraform plan" to see*

    *any changes that are required for your infrastructure. All Terraform commands*

    *should now work.*

    *If you ever set or change modules or backend configuration for Terraform,*

    *rerun this command to reinitialize your working directory. If you forget, other*

    *commands will detect it and remind you to do so if necessary.*

* *Execute terraform plan*

* *Generate and show an execution plan*

    *$ terraform plan*

    ***Refreshing Terraform state in-memory prior to plan...***

    *The refreshed state will be used to calculate this plan, but will not be*

    *persisted to local or remote state storage.*

    ***data.aws_availability_zones.available: Refreshing state...***

    *------------------------------------------------------------------------*

    *An execution plan has been generated and is shown below.*

    *Resource actions are indicated with the following symbols:*

    *+ create*

    *Terraform will perform the following actions:*

    *+ module.vpc_networking.aws_default_route_table.private_route*

    *id:                                         <computed>*

    *default_route_table_id:                     "${aws_vpc.main.default_route_table_id}"*

    *owner_id:                                   <computed>*

    *route.#:                                    <computed>*

    *tags.%:                                     "1"*

    *tags.Name:                                  "my-test-private-route"*

    *vpc_id:                                     <computed>*

    *+ module.vpc_networking.aws_internet_gateway.gw*

    *id:                                         <computed>*

    *owner_id:                                   <computed>*

    *tags.%:                                     "1"*

    *tags.Name:                                  "my-test-igw"*

    *vpc_id:                                     "${aws_vpc.main.id}"*

    *+ module.vpc_networking.aws_route_table.public_route*

    *id:                                         <computed>*

    *owner_id:                                   <computed>*

    *propagating_vgws.#:                         <computed>*

    *route.#:                                    "1"*

    *route.~966145399.cidr_block:                "0.0.0.0/0"*

    *route.~966145399.egress_only_gateway_id:    ""*

    *route.~966145399.gateway_id:                "${aws_internet_gateway.gw.id}"*

    *route.~966145399.instance_id:               ""*

    *route.~966145399.ipv6_cidr_block:           ""*

    *route.~966145399.nat_gateway_id:            ""*

    *route.~966145399.network_interface_id:      ""*

    *route.~966145399.transit_gateway_id:        ""*

    *route.~966145399.vpc_peering_connection_id: ""*

    *tags.%:                                     "1"*

    *tags.Name:                                  "my-test-public-route"*

    *vpc_id:                                     "${aws_vpc.main.id}"*

    *+ module.vpc_networking.aws_route_table_association.private_subnet_assoc[0]*

    *id:                                         <computed>*

    *route_table_id:                             "${aws_default_route_table.private_route.id}"*

    *subnet_id:                                  "${aws_subnet.private_subnet.*.id[count.index]}"*

    *+ module.vpc_networking.aws_route_table_association.private_subnet_assoc[1]*

    *id:                                         <computed>*

    *route_table_id:                             "${aws_default_route_table.private_route.id}"*

    *subnet_id:                                  "${aws_subnet.private_subnet.*.id[count.index]}"*

    *+ module.vpc_networking.aws_route_table_association.public_subnet_assoc[0]*

    *id:                                         <computed>*

    *route_table_id:                             "${aws_route_table.public_route.id}"*

    *subnet_id:                                  "${aws_subnet.public_subnet.*.id[count.index]}"*

    *+ module.vpc_networking.aws_route_table_association.public_subnet_assoc[1]*

    *id:                                         <computed>*

    *route_table_id:                             "${aws_route_table.public_route.id}"*

    *subnet_id:                                  "${aws_subnet.public_subnet.*.id[count.index]}"*

    *+ module.vpc_networking.aws_security_group.test_sg*

    *id:                                         <computed>*

    *arn:                                        <computed>*

    *description:                                "Managed by Terraform"*

    *egress.#:                                   "1"*

    *egress.482069346.cidr_blocks.#:             "1"*

    *egress.482069346.cidr_blocks.0:             "0.0.0.0/0"*

    *egress.482069346.description:               ""*

    *egress.482069346.from_port:                 "0"*

    *egress.482069346.ipv6_cidr_blocks.#:        "0"*

    *egress.482069346.prefix_list_ids.#:         "0"*

    *egress.482069346.protocol:                  "-1"*

    *egress.482069346.security_groups.#:         "0"*

    *egress.482069346.self:                      "false"*

    *egress.482069346.to_port:                   "0"*

    *ingress.#:                                  "1"*

    *ingress.2541437006.cidr_blocks.#:           "1"*

    *ingress.2541437006.cidr_blocks.0:           "0.0.0.0/0"*

    *ingress.2541437006.description:             ""*

    *ingress.2541437006.from_port:               "22"*

    *ingress.2541437006.ipv6_cidr_blocks.#:      "0"*

    *ingress.2541437006.prefix_list_ids.#:       "0"*

    *ingress.2541437006.protocol:                "tcp"*

    *ingress.2541437006.security_groups.#:       "0"*

    *ingress.2541437006.self:                    "false"*

    *ingress.2541437006.to_port:                 "22"*

    *name:                                       <computed>*

    *owner_id:                                   <computed>*

    *revoke_rules_on_delete:                     "false"*

    *vpc_id:                                     "${aws_vpc.main.id}"*

    *+ module.vpc_networking.aws_subnet.private_subnet[0]*

    *id:                                         <computed>*

    *arn:                                        <computed>*

    *assign_ipv6_address_on_creation:            "false"*

    *availability_zone:                          "us-west-2a"*

    *availability_zone_id:                       <computed>*

    *cidr_block:                                 "172.16.3.0/24"*

    *ipv6_cidr_block:                            <computed>*

    *ipv6_cidr_block_association_id:             <computed>*

    *map_public_ip_on_launch:                    "true"*

    *owner_id:                                   <computed>*

    *tags.%:                                     "1"*

    *tags.Name:                                  "my-test-private-subnet.1"*

    *vpc_id:                                     "${aws_vpc.main.id}"*

    *+ module.vpc_networking.aws_subnet.private_subnet[1]*

    *id:                                         <computed>*

    *arn:                                        <computed>*

    *assign_ipv6_address_on_creation:            "false"*

    *availability_zone:                          "us-west-2b"*

    *availability_zone_id:                       <computed>*

    *cidr_block:                                 "172.16.4.0/24"*

    *ipv6_cidr_block:                            <computed>*

    *ipv6_cidr_block_association_id:             <computed>*

    *map_public_ip_on_launch:                    "true"*

    *owner_id:                                   <computed>*

    *tags.%:                                     "1"*

    *tags.Name:                                  "my-test-private-subnet.2"*

    *vpc_id:                                     "${aws_vpc.main.id}"*

    *+ module.vpc_networking.aws_subnet.public_subnet[0]*

    *id:                                         <computed>*

    *arn:                                        <computed>*

    *assign_ipv6_address_on_creation:            "false"*

    *availability_zone:                          "us-west-2a"*

    *availability_zone_id:                       <computed>*

    *cidr_block:                                 "172.16.1.0/24"*

    *ipv6_cidr_block:                            <computed>*

    *ipv6_cidr_block_association_id:             <computed>*

    *map_public_ip_on_launch:                    "true"*

    *owner_id:                                   <computed>*

    *tags.%:                                     "1"*

    *tags.Name:                                  "my-test-public-subnet.1"*

    *vpc_id:                                     "${aws_vpc.main.id}"*

    *+ module.vpc_networking.aws_subnet.public_subnet[1]*

    *id:                                         <computed>*

    *arn:                                        <computed>*

    *assign_ipv6_address_on_creation:            "false"*

    *availability_zone:                          "us-west-2b"*

    *availability_zone_id:                       <computed>*

    *cidr_block:                                 "172.16.2.0/24"*

    *ipv6_cidr_block:                            <computed>*

    *ipv6_cidr_block_association_id:             <computed>*

    *map_public_ip_on_launch:                    "true"*

    *owner_id:                                   <computed>*

    *tags.%:                                     "1"*

    *tags.Name:                                  "my-test-public-subnet.2"*

    *vpc_id:                                     "${aws_vpc.main.id}"*

    *+ module.vpc_networking.aws_vpc.main*

    *id:                                         <computed>*

    *arn:                                        <computed>*

    *assign_generated_ipv6_cidr_block:           "false"*

    *cidr_block:                                 "172.16.0.0/16"*

    *default_network_acl_id:                     <computed>*

    *default_route_table_id:                     <computed>*

    *default_security_group_id:                  <computed>*

    *dhcp_options_id:                            <computed>*

    *enable_classiclink:                         <computed>*

    *enable_classiclink_dns_support:             <computed>*

    *enable_dns_hostnames:                       "true"*

    *enable_dns_support:                         "true"*

    *instance_tenancy:                           "default"*

    *ipv6_association_id:                        <computed>*

    *ipv6_cidr_block:                            <computed>*

    *main_route_table_id:                        <computed>*

    *owner_id:                                   <computed>*

    *tags.%:                                     "1"*

    *tags.Name:                                  "my-test-vpc"*

    ***Plan:** 13 to add, 0 to change, 0 to destroy.*

    *------------------------------------------------------------------------*

    *Note: You didn't specify an "-out" parameter to save this plan, so Terraform*

    *can't guarantee that exactly these actions will be performed if*

    *"terraform apply" is subsequently run.*

* *Final step terraform apply’*

* *Builds or changes infrastructure*

    *$ terraform apply*

    ***data.aws_availability_zones.available: Refreshing state...***

    *An execution plan has been generated and is shown below.*

    *Resource actions are indicated with the following symbols:*

    *+ create*

    *Terraform will perform the following actions:*

    *+ module.vpc_networking.aws_default_route_table.private_route*

    *id:                                         <computed>*

    *default_route_table_id:                     "${aws_vpc.main.default_route_table_id}"*

    *owner_id:                                   <computed>*

    *route.#:                                    <computed>*

    *tags.%:                                     "1"*

    *tags.Name:                                  "my-test-private-route"*

    *vpc_id:                                     <computed>*

    *+ module.vpc_networking.aws_internet_gateway.gw*

    *id:                                         <computed>*

    *owner_id:                                   <computed>*

    *tags.%:                                     "1"*

    *tags.Name:                                  "my-test-igw"*

    *vpc_id:                                     "${aws_vpc.main.id}"*

    *+ module.vpc_networking.aws_route_table.public_route*

    *id:                                         <computed>*

    *owner_id:                                   <computed>*

    *propagating_vgws.#:                         <computed>*

    *route.#:                                    "1"*

    *route.~966145399.cidr_block:                "0.0.0.0/0"*

    *route.~966145399.egress_only_gateway_id:    ""*

    *route.~966145399.gateway_id:                "${aws_internet_gateway.gw.id}"*

    *route.~966145399.instance_id:               ""*

    *route.~966145399.ipv6_cidr_block:           ""*

    *route.~966145399.nat_gateway_id:            ""*

    *route.~966145399.network_interface_id:      ""*

    *route.~966145399.transit_gateway_id:        ""*

    *route.~966145399.vpc_peering_connection_id: ""*

    *tags.%:                                     "1"*

    *tags.Name:                                  "my-test-public-route"*

    *vpc_id:                                     "${aws_vpc.main.id}"*

    *+ module.vpc_networking.aws_route_table_association.private_subnet_assoc[0]*

    *id:                                         <computed>*

    *route_table_id:                             "${aws_default_route_table.private_route.id}"*

    *subnet_id:                                  "${aws_subnet.private_subnet.*.id[count.index]}"*

    *+ module.vpc_networking.aws_route_table_association.private_subnet_assoc[1]*

    *id:                                         <computed>*

    *route_table_id:                             "${aws_default_route_table.private_route.id}"*

    *subnet_id:                                  "${aws_subnet.private_subnet.*.id[count.index]}"*

    *+ module.vpc_networking.aws_route_table_association.public_subnet_assoc[0]*

    *id:                                         <computed>*

    *route_table_id:                             "${aws_route_table.public_route.id}"*

    *subnet_id:                                  "${aws_subnet.public_subnet.*.id[count.index]}"*

    *+ module.vpc_networking.aws_route_table_association.public_subnet_assoc[1]*

    *id:                                         <computed>*

    *route_table_id:                             "${aws_route_table.public_route.id}"*

    *subnet_id:                                  "${aws_subnet.public_subnet.*.id[count.index]}"*

    *+ module.vpc_networking.aws_security_group.test_sg*

    *id:                                         <computed>*

    *arn:                                        <computed>*

    *description:                                "Managed by Terraform"*

    *egress.#:                                   "1"*

    *egress.482069346.cidr_blocks.#:             "1"*

    *egress.482069346.cidr_blocks.0:             "0.0.0.0/0"*

    *egress.482069346.description:               ""*

    *egress.482069346.from_port:                 "0"*

    *egress.482069346.ipv6_cidr_blocks.#:        "0"*

    *egress.482069346.prefix_list_ids.#:         "0"*

    *egress.482069346.protocol:                  "-1"*

    *egress.482069346.security_groups.#:         "0"*

    *egress.482069346.self:                      "false"*

    *egress.482069346.to_port:                   "0"*

    *ingress.#:                                  "1"*

    *ingress.2541437006.cidr_blocks.#:           "1"*

    *ingress.2541437006.cidr_blocks.0:           "0.0.0.0/0"*

    *ingress.2541437006.description:             ""*

    *ingress.2541437006.from_port:               "22"*

    *ingress.2541437006.ipv6_cidr_blocks.#:      "0"*

    *ingress.2541437006.prefix_list_ids.#:       "0"*

    *ingress.2541437006.protocol:                "tcp"*

    *ingress.2541437006.security_groups.#:       "0"*

    *ingress.2541437006.self:                    "false"*

    *ingress.2541437006.to_port:                 "22"*

    *name:                                       <computed>*

    *owner_id:                                   <computed>*

    *revoke_rules_on_delete:                     "false"*

    *vpc_id:                                     "${aws_vpc.main.id}"*

    *+ module.vpc_networking.aws_subnet.private_subnet[0]*

    *id:                                         <computed>*

    *arn:                                        <computed>*

    *assign_ipv6_address_on_creation:            "false"*

    *availability_zone:                          "us-west-2a"*

    *availability_zone_id:                       <computed>*

    *cidr_block:                                 "172.16.3.0/24"*

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/CUjx-L074mw" frameborder="0" allowfullscreen></iframe></center>

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/IeJEn7y83a8" frameborder="0" allowfullscreen></iframe></center>

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/HRO8ZkjlWFI" frameborder="0" allowfullscreen></iframe></center>
