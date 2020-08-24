Unknown markup type 10 { type: [33m10[39m, start: [33m7[39m, end: [33m21[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m74[39m, end: [33m97[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m103[39m, end: [33m111[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m117[39m, end: [33m127[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m20[39m, end: [33m23[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m143[39m, end: [33m152[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m76[39m, end: [33m85[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m28[39m, end: [33m37[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m31[39m, end: [33m43[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m53[39m, end: [33m58[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m54[39m, end: [33m59[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m87[39m, end: [33m92[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m97[39m, end: [33m103[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m151[39m, end: [33m156[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m184[39m, end: [33m190[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m74[39m, end: [33m88[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m80[39m, end: [33m89[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m102[39m, end: [33m108[39m }

# Building AWS Infra with Terraform 3

Building AWS Infra with Terraform 3

### Creating Web Application and Database Infrastructure

I wanted to get this out quickly as someone besides my friend would want to see the final step to this small series. This is continuation of the series to learn how to provision AWS with [**Terraform](https://www.terraform.io/)**.

In the last article we covered the ***infrastructure concern***. In this article we will cover the ***web application concern*** that will include the following:

* Instance hosting web application ([**EC2](https://aws.amazon.com/ec2/)**)

* Database used by web application ([**MySQL](https://www.mysql.com/)** managed by [**RDS](https://aws.amazon.com/rds/)**)

* Web Application itself (simple [**LAMP](http://wikipedia)** Application)

## Previous Article
[**Building AWS Infra with Terraform 2**
*Creating Network Infrastructure and Security Groups*medium.com](https://medium.com/@Joachim8675309/building-aws-infra-with-terraform-2-ca60146666f8)

## Web Application Concern

We can start to put applications on top of the infrastructure foundation we just created.

In the ~/tf-projects/ directory, we’ll create the following structure:

    .
    └── webapp/  
        ├── app/
        │   ├── main.tf
        │   ├── user_data.sh
        │   └── variables.tf
        ├── aws.tf
        ├── db/
        │   ├── main.tf
        │   └── variables.tf
        ├── main.tf
        └── variables.tf

### Create Structure

Run this under [**Bash](https://www.gnu.org/software/bash/)** to create the structure and the files we will edit:

    **cd** ~/tf-projects/webapp
    **touch** {.,app,db}/{main.tf,variables.tf} app/user_data.sh

## Create WebApp Module

We will create the module for the web application concern with two sub-modules: one for the web application itself and the other for the database.

### WebApp Input Variables

First we will create some variables that will be used in this module. We’ll start with the variables for the [**AWS provider](https://www.terraform.io/docs/providers/aws/index.html)**:

    **cat** <<-'**WEBAPP_VARIABLES**' > ~/tf-projects/webapp/variables.tf

    **variable** "profile" {}
    **variable** "region" {}

    **WEBAPP_VARIABLES**

We’ll want to add variables that we’ll pull from the infra module (from the previous article) and reuse in the web application:

    **cat** <<-'**WEBAPP_VARIABLES**' >> ~/tf-projects/webapp/variables.tf

    *# security groups*
    **variable** "sg_web" {}
    **variable** "sg_db" {}

    *# subnets*
    **variable** "sn_web" {}
    **variable** "sn_db1" {}
    **variable** "sn_db2" {}

    **WEBAPP_VARIABLES**

Lastly, we’ll want to add configuration that is unique to our web application, the database user name and password to a newly created database.

    **cat** <<-'**WEBAPP_VARIABLES**' >> ~/tf-projects/webapp/variables.tf

    *# config artifact*
    **variable** "database_name" {}
    **variable** "database_user" {}

    *# secrets artifact*
    **variable** "database_password" {}

    *# instance key pair*
    **variable** "key_name" {}

    **WEBAPP_VARIABLES**

Variables that we configure for an application can be called *configuration artifacts*, and *configuration artifacts* that are sensitive are called *secrets artifacts*.

Ideally, we will want store *configuration artifacts* somewhere that can be referenced, and *secrets artifacts* should be stored in encrypted format.

In order to keep things simple for this tutorial, we’ll store these in as ~/tf-projects/db.tfvars to store secrets and configuration:

    **cat** <<-'**SECRETS**' >> ~/tf-projects/db.tfvars

    database_name     = "webdb"
    database_user     = "admin"
    database_password = "[@U1bO8s](http://twitter.com/U1bO8s)$^&GkUAz*l$$@BG87"

    **SECRETS**

Never check this file into a code repository because our secret would not be safe. We will want to put *.tfvars into .gitignore file.

### WebApp Main

    **cat** <<-'**WEBAPP_MODULE**' > ~/tf-projects/webapp/main.tf

    **module** "instances" {
      **source** = "./app"

      **sg_web** = "${var.sg_web}"
      **sn_web** = "${var.sn_web}"
      **key_name** = "${var.key_name}"
    }

    **module** "db" {
      **source** = "./db"

      **sg_db**  = "${var.sg_db}"
      **sn_db1** = "${var.sn_db1}"
      **sn_db2** = "${var.sn_db2}"

      **database_name**     = "${var.database_name}"
      **database_user**     = "${var.database_user}"
      **database_password** = "${var.database_password}"
    }

    **WEBAPP_MODULE**

## Create Web Application

For this sub-module app, we’ll create an EC2 instance to host the web application and install the web application itself.

**Note:** this application is not highly available, as it is only installed on a single public subnet that lives on a single AZ (availability zone). Should we want to make it more available, we would create at least two identical web servers installed on subnets in different AZs, and then park these behind an ELB (elastic load balancer) that could send traffic to one of these two web servers. For this exercise, we’re keeping it simple.

### App Input Variables

This sub-module takes two inputs, a public subnet and a security group.

    **cat** <<-'**APP_VARIABLES**' >> ~/tf-projects/webapp/app/variables.tf

    variable "sg_web" {}
    variable "sn_web" {}
    variable "key_name" {}

    **APP_VARIABLES**

### System Image Data Source

We the operating system we wish to use, we’re going to use [**Amazon Linux](https://aws.amazon.com/amazon-linux-ami/)**, which based from [**RHEL](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux)**. We have to find [**AMI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)** ([**Amazon Machine Image](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)**) for us-east-2.

The lazy way is to find the ID, but then this make the script only work for us-east-2, and also invites security vulnerabilities, as these machine images churn often to fix bugs and plug vulnerabilities.

For ameliorate this, we can look up the information using a data source:

    **cat** <<-'**APP_MODULE**' > ~/tf-projects/webapp/app/main.tf

    **data** "aws_ami" "amazon-linux-2" {
      **most_recent** = true

      **filter** {
        **name**   = "virtualization-type"
        **values** = ["hvm"]
      }

      **filter** {
        **name**   = "architecture"
        **values** = ["x86_64"]
      }

      **filter** {
        **name**   = "name"
        **values** = ["amzn2-ami-hvm-2.0*"]
      }

      **owners** = ["137112412989"] *# Amazon*
    }

    **APP_MODULE**

Now that we have the this, we can reference our target image with:

    data.aws_ami.amazon-linux-2.id

### User Data Startup Script

We need a script to provision our server with the web service. Amazon provided a small web application that we’ll download and install. We also want to install [**Apache HTTP server](http://httpd.apache.org/)**, [**PHP](https://www.php.net/)**, and [**MySQL](https://www.mysql.com/)** client that the application needs:

    **cat** <<-'**USER_DATA**' > ~/tf-projects/webapp/app/user_data.sh

    *#!/bin/bash -ex*
    **yum** -y update
    **yum** -y install httpd php mysql php-mysql

    **chkconfig** httpd on
    **service** httpd start

    **cd** /var/www/html

    S3_HOST=[s3-us-west-2.amazonaws.com](https://s3-us-west-2.amazonaws.com/us-west-2-aws-training/awsu-spl/spl-13/scripts/app.tgz)
    APP_PATH=[us-west-2-aws-training/awsu-spl/spl-13/scripts/app.tgz](https://s3-us-west-2.amazonaws.com/us-west-2-aws-training/awsu-spl/spl-13/scripts/app.tgz)
    **wget** [https://${S3_HOST}/${APP_PATH}](https://${S3_HOST}/${APP_PATH})

    **tar** xvfz app.tgz
    **chown** apache:root /var/www/html/rds.conf.php

    **USER_DATA**

### Instance Resource

Now the fun starts with our EC2 instance:

    **cat** <<-'**APP_MODULE**' >> ~/tf-projects/webapp/app/main.tf

    **resource** "aws_instance" "my-webserver" {
      **ami**           = "${data.aws_ami.amazon-linux-2.id}"
      **instance_type** = "t2.micro"
      **key_name**      = "${var.key_name}"
      **user_data**     = "${file("${path.module}/user_data.sh")}"
      **subnet_id**     = "${var.sn_web}"

      **associate_public_ip_address** = true

      **vpc_security_group_ids** = [
        "${var.sg_web}",
      ]

      **tags** {
        "Name" = "my-webserver"
        "Site" = "my-web-site"
      }
    }

    **APP_MODULE**

This code will reference the following external bits to build the EC2 instance:

* [**key pair](https://docs.aws.amazon.com/cli/latest/userguide/cli-services-ec2-keypairs.html)** name that we created earlier (see [first article](https://medium.com/@Joachim8675309/building-aws-infra-with-terraform-96387481b9d7))

* latest [**Amazon Linux](https://aws.amazon.com/amazon-linux-ami/)** AMI for us-east-2

* [**user data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)** provisioning script (user_data.sh)

* a public [**subnet](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html)** created with *infrastructure concern* (infra module)

* a [**security group](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)** created with *infrastructure concern* (infra module)

## Create Database Application

We can now create a MySQL using Amazon [**RDS](https://aws.amazon.com/rds/)** ([**Relational Database Service](https://aws.amazon.com/rds/)**). By using [**RDS](https://aws.amazon.com/rds/)**, we do not have to manage our own database, but instead allow Amazon to manage it for us.

### Input Variables

    **cat** <<-'**DB_VARIABLES**' > ~/tf-projects/webapp/db/variables.tf

    variable "sg_db" {}
    variable "sn_db1" {}
    variable "sn_db2" {}

    variable "database_name" {}
    variable "database_user" {}
    variable "database_password" {}

    **DB_VARIABLES**

### Database Subnet Group

    **cat** <<-'**DB_MODULE**' > ~/tf-projects/webapp/db/main.tf

    **resource** "aws_db_subnet_group" "my-dbsg" {
      **name**        = "my-dbsg"
      **description** = "my-dbsg"
      **subnet_ids**  = ["${var.sn_db1}", "${var.sn_db2}"]

      **tags** {
        "Name" = "my-dbsg"
        "Site" = "my-web-site"
      }
    }

    **DB_MODULE**

### Database Instance

We’ll create a small [**MySQL](https://www.mysql.com/)** 5.6.40 database that has no backup. This is a small throwaway database, so don’t use this code for a production database.

    **cat** <<-'**DB_MODULE**' >> ~/tf-projects/webapp/db/main.tf

    **resource** "aws_db_instance" "my-db" {
      **identifier**        = "my-db"
      **allocated_storage** = 20
      **storage_type**      = "gp2"
      **engine**            = "mysql"
      **engine_version**    = "5.6.40"
      **instance_class**    = "db.t2.micro"

      **name**     = "${var.database_name}"
      **username** = "${var.database_user}"
      **password** = "${var.database_password}"

      **parameter_group_name**   = "default.mysql5.6"
      **db_subnet_group_name**   = "${aws_db_subnet_group.my-dbsg.id}"
      **vpc_security_group_ids** = ["${var.sg_db}"]

      # set these for dev db
      **backup_retention_period** = 0

      # required for deleting
      **skip_final_snapshot**       = true
      **final_snapshot_identifier** = "Ignore"

      **tags** {
        "Name" = "my-db"
        "Site" = "my-web-site"
      }
    }

    **DB_MODULE**

## Creating Main Terraform Script

We need to create a main Terraform script that calls both of our modules together, the infra and webapp modules. This script will take output from the infra module, and pass it to the webapp module.

    **cat** <<-'**MAIN**' >> ~/tf-projects/main.tf

    *#### VARIABLES*
    **variable** "profile" {}
    **variable** "region" {}
    **variable** "database_name" {}
    **variable** "database_user" {}
    **variable** "database_password" {}
    **variable** "key_name" {
      **default** = "deploy-aws"
    }

    *#### CALL MDOULES*
    **module** "core_infra" {
      **source**  = "./infra"
      **profile** = "${var.profile}"
      **region**  = "${var.region}"
    }

    **module** "webapp" {
      **source**   = "./webapp"
      **profile ** = "${var.profile}"
      **region**   = "${var.region}"

      **key_name** = "${var.key_name}"

      *# pass web security group and public networks*
      **sg_web** = "${module.core_infra.sg_web}"
      **sn_web** = "${module.core_infra.sn_pub1}"

    *  # pass database security group and private networks*
      **sg_db**  = "${module.core_infra.sg_db}"
      **sn_db1** = "${module.core_infra.sn_priv1}"
      **sn_db2** = "${module.core_infra.sn_priv2}"

      *# database parameters*
      **database_name**     = "${var.database_name}"
      **database_user**     = "${var.database_user}"
      **database_password** = "${var.database_password}"
    }

    **MAIN**

### Execute the Script to Create the Infrastructure and Web App

To run this altogether, we’d do something like this:

    **cd** ~/tf-projects

    **export** **AWS_PROFILE**=learning
    **export** **TF_VAR_region**=$(
      awk -F'= ' '/region/{print $2}' <(
        grep -A1 "\[.***$AWS_PROFILE**\]" ~/.aws/config)
    )

    # show changes required (using db variables file)
    **terraform** plan -var-file="db.tfvars"

    # apply changes required (using db variables file)
    **terraform** apply -var-file="db.tfvars"

## Testing the Web Application

First we will need to fetch information. We can get computed values using terraform show command. We first need to get the public IP address so that we can log into web app database:

    **terraform** show | **grep** -o 'public_ip = .*$'

After navigating to the public IP using a web browser, we should see an interface like this:

![](https://cdn-images-1.medium.com/max/2968/1*TscTy9RbLdc9QvJYR1qjqg.png)

The web application has no configuration, so we’ll need to fill in the information manually. Let’s get the database endpoint:

    **terraform** show | **grep** -o 'endpoint = .*$'

This will give you an endpoint similar to this format:

    my-db.cknof0oc3nnn.us-east-2.rds.amazonaws.com:3306

Enter this information, plus the database name, username, and password saved in db.tfvars and hit the Submit button. After you should see see something like this:

![](https://cdn-images-1.medium.com/max/2928/1*rKez37a5rPk2GNeBK_n8Qg.png)

## Wrapping Up

There you have it: how to create VPC and network infrastructure with front end public subnets and backend private subnets. The webapp is not highly available, as it is installed on a single public subnet.

In order to remedy the low availability, we would create another web application on a different subnet, and then create a load balancer ELB to distribute the traffic between those systems. But that is for a future article…

Additionally, this script is dependent on the infrastructure, so it is not really a separated concern. In order to make the web app module truly independent, we’d need to use data sources to lookup the security groups and subnets we need. The downside to this, if the infrastructure was not created, this would then fail, possibly with a cryptic message. I’m considering a follow up article for these concept, as well as the one above…
