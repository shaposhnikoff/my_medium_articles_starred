Unknown markup type 10 { type: [33m10[39m, start: [33m125[39m, end: [33m134[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m86[39m, end: [33m116[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m61[39m, end: [33m66[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m118[39m, end: [33m121[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m126[39m, end: [33m129[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m3[39m, end: [33m5[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m7[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m12[39m, end: [33m15[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m24[39m, end: [33m26[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m27[39m, end: [33m38[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m15[39m, end: [33m17[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m20[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m93[39m, end: [33m104[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m109[39m, end: [33m120[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m20[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m37[39m, end: [33m48[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m53[39m, end: [33m64[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m20[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m20[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m23[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m23[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m18[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m17[39m, end: [33m19[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m17[39m, end: [33m19[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m40[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m17[39m, end: [33m19[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m64[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m18[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m35[39m, end: [33m38[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m43[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m95[39m, end: [33m100[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m18[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m40[39m, end: [33m42[39m }

# Building AWS Infra with Terraform 2

Building AWS Infra with Terraform 2

### Part 2: Creating Network Infrastructure and Security Groups

A friend of mine is learning *cloud provisioning* with [**Terraform](https://www.terraform.io/)** and was asking me when I would continue with this series, and so here is the next part.

In previous introductory article, I described how to setup the tooling and separating out concerns so that it will be easier to maintain infrastructure code. This article will focus on the ***infrastructure concern***.

## Previous Article
[**Building AWS Infra with Terraform**
*Infrastructure as Code using Terraform*medium.com](https://medium.com/@Joachim8675309/building-aws-infra-with-terraform-96387481b9d7)

## Knowledge Prerequisites

Some of this will not make sense without basic understanding of TCP/IP protocol and routing. For the IP address ranges, e.g. X.X.X.X/X, you can use a CIDR calculator (numerous online) to see the address ranges or calculate yourself if you know the math.

For [**Terraform](https://www.terraform.io/)**, you need to know how to use and pass variables to modules, and how to use output values:

* **Input Variables**: [https://www.terraform.io/docs/configuration/variables.html](https://www.terraform.io/docs/configuration/variables.html)

* **Output Values**: [https://www.terraform.io/docs/configuration/outputs.html](https://www.terraform.io/docs/configuration/outputs.html)

You would also need to know the concept of providers, resources, and data sources:

* **Providers**: [https://www.terraform.io/docs/configuration/providers.html](https://www.terraform.io/docs/configuration/providers.html)

* **Resources**: [https://www.terraform.io/docs/configuration/resources.html](https://www.terraform.io/docs/configuration/resources.html)

* **Data Sources**: [https://www.terraform.io/docs/configuration/data-sources.html](https://www.terraform.io/docs/configuration/data-sources.html)

Additionally, some overall knowledge on creating AWS resources using the web console (https://console.aws.amazon.com) would be helpful. For documentation on the resources created:

* **VPC**: [https://aws.amazon.com/vpc/](https://aws.amazon.com/vpc/)

* **Subnets**: [https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html)

* **Internet Gateway**: [https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)

* **Route Table**: [https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)

* **Security Groups**: [https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)

## Infrastructure Concern

For the *infrastructure concern*, we’ll place everything under infra. In this module, we will organize two sub-modules: net and sec, which will contain code needed for networking (*internet* *gateway*, *route tables*, *subnets*) and security groups.

Security groups are good to keep in one place, especially when you need to audit your security. These change frequently, where the network infrastructure changes rarely.

In your directory structure, we’ll create the following:

    .
    └── infra/
        ├── aws.tf
        ├── main.tf
        ├── net/
        │   ├── main.tf
        │   └── output.tf
        ├── output.tf
        └── sec/
            ├── main.tf
            ├── output.tf
            └── variables.tf

### Create Structure

To create this, you can do the following under Bash:

    **cd** ~/tf-projects/infra
    **touch** {.,net,sec}/{main.tf,output.tf} sec/variables.tf

### Segue: Multi-Datacenter Structure

The net and sec directories will be all the networking infrastructure and security we will use. In an enterprise organization, you may have more have further subdivisions like, as examples:

* net/us-west-1

* net/us-east-1

* sec/us-west-1

* sec/us-east-1

This way can organize and associate particular network and security to particular data centers around the world.

## Creating Modules

First we’ll create the main module that will be responsible for creating our infrastructure concern:

    **cat** <<-'**INFRA_MODULE**' > ~/tf-projects/infra/main.tf

    **module** "network" {
      **source** = "./net"
    }

    **module** "security" {
      **source** = "./sec"
      **vpc_id** = "${module.network.vpc}"
    }

    **INFRA_MODULE**

This module will create the network infrastructure and security group. The security group module will need to know the VPC for creating the security groups, as these are tied to securing subnets within that VPC.

## Creating VPC

We can create our VPC with 10.0.0.0/16 network. We’ll add a few tags to describe the purpose of our VPC.

    **cat** <<-'**VPC**' > ~/tf-projects/infra/net/main.tf

    **resource** "aws_vpc" "my-main" {
      **cidr_block**           = "10.0.0.0/16"
      **enable_dns_hostnames** = false
      **enable_dns_support**   = true
      **instance_tenancy**     = "default"

    **tags** {
        **Site** = "my-web-site"
        **Name** = "my-vpc"
      }
    }

    **VPC**

## Creating Subnets

This will create the subnets within our VPC for our infrastructure.

First, let’s reference the *availability zones*, as this varies between regions, between 2–3. Regions will have 2–4 availability zones. Each AZ (availability zone) represents a unique data center within that region. Using this will allow us use an index instead of using a static string.

    **cat** <<-'**SUBNETS**' >> ~/tf-projects/infra/net/main.tf

    **data** "aws_availability_zones" "available" {}

    **SUBNETS**

Now we can add some subnets that will be used for systems that may have public IP addresses: 10.0.2.0/24 and 10.0.1.0/24. We want to keep *public* and *private* subnets separated, so if there is a breach, the attacker won’t be able to access systems on the *private* network easily.

    **cat** <<-'**SUBNETS**' >> ~/tf-projects/infra/net/main.tf

    **resource** "aws_subnet" "my-public1" {
      **vpc_id**                  = "${aws_vpc.my-main.id}"
      **cidr_block**              = "10.0.2.0/24"
      **availability_zone**       = "${data.aws_availability_zones.available.names[1]}"
      **map_public_ip_on_launch** = true

    **tags** {
        **Name** = "my-public2"
        **Site** = "my-web-site"
      }
    }

    **resource** "aws_subnet" "my-public2" {
      **vpc_id**                  = "${aws_vpc.my-main.id}"
      **cidr_block**              = "10.0.1.0/24"
      **availability_zone**       = "${data.aws_availability_zones.available.names[0]}"
      **map_public_ip_on_launch** = true

    **tags** {
        **Name** = "my-public1"
        **Site** = "my-web-site"
      }
    }

    **SUBNETS**

Now we can add some *private* subnets: 10.0.3.0/24 and 10.0.4.0/24.

    **cat** <<-'**SUBNETS**' >> ~/tf-projects/infra/net/main.tf

    **resource** "aws_subnet" "my-private1" {
      **vpc_id**                  = "${aws_vpc.my-main.id}"
      **cidr_block**              = "10.0.3.0/24"
      **availability_zone**       = "${data.aws_availability_zones.available.names[1]}"
      **map_public_ip_on_launch** = true

    **tags** {
        **Name** = "my-private1"
        **Site** = "my-web-site"
      }
    }

    **resource** "aws_subnet" "my-private2" {
      **vpc_id**                  = "${aws_vpc.my-main.id}"
      **cidr_block**              = "10.0.4.0/24"
      **availability_zone**       = "${data.aws_availability_zones.available.names[0]}"
      **map_public_ip_on_launch** = true

    **tags** {
        **Name** = "my-private2"
        **Site** = "my-web-site"
      }
    }

    **SUBNETS**

This finalizes creating *subnets*.

## Creating Internet Gateway

We need to create an Internet Gateway so that systems can get out to the Internet and respond to users that connect to our web server.

    **cat** <<-'**GATEWAY**' >> ~/tf-projects/infra/net/main.tf

    resource "aws_internet_gateway" "my-igw" {
      **vpc_id** = "${aws_vpc.my-main.id}"

      **tags** = {
        **Name** = "my-igw"
        **Site** = "my-web-site"
      }
    }

    **GATEWAY**

## Create Route Table

We need to tell our systems on the *public* subnets how to route traffic by first creating a route table.

    **cat** <<-'**ROUTETABLE**' >> ~/tf-projects/infra/net/main.tf

    **resource** "aws_route_table" "my-rt" {
      **vpc_id** = "${aws_vpc.my-main.id}"

      **route** {
        **cidr_block** = "0.0.0.0/0"
        **gateway_id** = "${aws_internet_gateway.my-igw.id}"
      }

      **tags** {
        **Site** = "my-web-site"
        **Name** = "my-rt"
      }
    }

    **ROUTETABLE**

This won’t do us any good unless we associate our *public* subnets to the route table.

    **cat** <<-'**ROUTETABLE**' >> ~/tf-projects/infra/net/main.tf

    **resource** "aws_route_table_association" "my-public1" {
      **subnet_id**      = "${aws_subnet.my-public1.id}"
      **route_table_id** = "${aws_route_table.my-rt.id}"
    }

    **resource** "aws_route_table_association" "my-public2" {
      **subnet_id**      = "${aws_subnet.my-public2.id}"
      **route_table_id** = "${aws_route_table.my-rt.id}"
    }

    **ROUTETABLE**

## Output Network Information

Now that we created our infrastructure, we need to share the information, so that other modules can use this information. We’ll want to share the VPC and subnets for other modules.

    **cat** <<-'**OUTPUT**' > ~/tf-projects/infra/net/output.tf

    **output** "vpc" {
      **value** = "${aws_vpc.my-main.id}"
    }

    **output** "sn_pub1" {
      **value** = "${aws_subnet.my-public1.id}"
    }

    **output** "sn_pub2" {
      **value** = "${aws_subnet.my-public2.id}"
    }

    **output** "sn_priv1" {
      **value** = "${aws_subnet.my-private1.id}"
    }

    **output** "sn_priv2" {
      **value** = "${aws_subnet.my-private2.id}"
    }

    **OUTPUT**

## Creating Security Groups

Now we can create our security groups so that parts of the infrastructure can communicate to each other, and so the web server can communicate to users.

### Input Variables

We will have one variable that we need, the VPC to where we apply these security groups.

    **cat** <<-'INPUT' > ~/tf-projects/infra/sec/variables.tf

    **variable** "vpc_id" {}

    **INPUT**

### Create Web Server SG

We want to allow the public Internet to access our web server (ingress):

    **cat** <<-'**WEBSG**' > ~/tf-projects/infra/sec/main.tf

    **resource** "aws_security_group" "my-webserver" {
      **name**        = "webserver"
      **description** = "Allow HTTP from Anywhere"
      **vpc_id**      = "${var.vpc_id}"

      **ingress** {
        **from_port**   = 80
        **to_port**     = 80
        **protocol**    = "tcp"
        **cidr_blocks** = ["0.0.0.0/0"]
      }

      **egress** {
        **from_port**   = 0
        **to_port**     = 0
        **protocol**    = "-1"
        **cidr_blocks** = ["0.0.0.0/0"]
      }

    **  tags** {
        **Name** = "my-webserver"
        **Site** = "my-web-site"
      }
    }

    **WEBSG**

### Create Database Server SG

We want to open up access to MySQL port 3306, but only for web servers we created earlier. We can do this by linking to security group id of the web server security group.

    **cat** <<-'**DBSG**' >> ~/tf-projects/infra/sec/main.tf

    **resource** "aws_security_group" "my-database" {
      **name**        = "database"
      **description** = "Allow MySQL/Aurora from WebService"
      **vpc_id**      = "${var.vpc_id}"

      **ingress** {
        **from_port**       = 3306
        **to_port**         = 3306
        **protocol**        = "tcp"
        **security_groups** = ["${aws_security_group.my-webserver.id}"]
        **self**            = false
      }

      **egress** {
        **from_port**   = 0
        **to_port**     = 0
        **protocol**    = "-1"
        **cidr_blocks** = ["0.0.0.0/0"]
      }

    **  tags** {
        **Name** = "my-database"
        **Site** = "my-web-site"
      }
    }

    **DBSG**

Now any web server that has the was configured with my-webserver security group, will now automatically have access to the database.

### Output Security Group

After creating the needed security groups for the webapp, we’ll need to share the output for other modules to use.

    **cat** <<-'**OUTPUT**' > ~/tf-projects/infra/sec/output.tf

    **output** "sg_web" {
      **value** = "${aws_security_group.my-webserver.id}"
    }

    **output** "sg_db" {
      **value** = "${aws_security_group.my-database.id}"
    }

    **OUTPUT**

## Output Network and Security Groups

We want to forward the output form net and sec submodules to anything that may want to use the infra module. We can reference the output returned by the modules we called.

    **cat** <<-'**OUTPUT**' > ~/tf-projects/infra/output.tf

    *# Net module output***
    output** "vpc" {
      value = "${module.network.vpc}"
    }

    **output** "sn_pub1" {
      value = "${module.network.sn_pub1}"
    }

    **output** "sn_pub2" {
      value = "${module.network.sn_pub2}"
    }

    **output** "sn_priv1" {
      value = "${module.network.sn_priv1}"
    }

    **output** "sn_priv2" {
      value = "${module.network.sn_priv2}"
    }

    *# Sec module output***
    output** "sg_web" {
      value = "${module.security.sg_web}"
    }

    **output** "sg_db" {
      value = "${module.security.sg_db}"
    }

    **OUTPUT**

### Segue: Keep ’Em Separated

For pure separation of concerns, we may not want to do this. We would reference the information separately and not depend on output of infra module. The reason why you might want to do this is to avoid accidents that can occur by taking out your network infrastructure.

## Testing the Project

You can create your infrastructure by doing the following:

    *# setup variables***
    export** **AWS_DEFAULT_PROFILE**="learning"
    **export** **AWS_PROFILE**=$AWS_DEFAULT_PROFILE
    **export** **TF_VAR_profile**=$AWS_DEFAULT_PROFILE
    **export** **TF_VAR_region**=$(
      awk -F'= ' '/region/{print $2}' <(
        grep -A1 "\[.***$AWS_PROFILE**\]" ~/.aws/config)
    )

    *# initialize modules and see changes*
    **cd** ~/tf-projects/infra
    **terraform** init
    **terraform** plan

    *# create infrastructure*
    **terraform** apply

    *# cleanup infrastructure*
    **terraform** destroy

## Conclusion

This completes the *infrastructure concern* for our web app. With this infrastructure, we have two usable networks:

* *public* subnet that can host instances with private and public Internet addresses

* *private* subnet where access is granted on per-instance basis by linking to another security group and can only have private Internet addresses.

### Next Article

In the next article I will show how to create the database infrastructure and the front end web application.
[**Building AWS Infra with Terraform 3**
*Creating Web Application and Database Infrastructure*medium.com](https://medium.com/@Joachim8675309/building-aws-infra-with-terraform-3-1d21f3c40bb0)
