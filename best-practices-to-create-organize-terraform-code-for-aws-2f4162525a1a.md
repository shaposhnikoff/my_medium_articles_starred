
# Best practices to create & organize Terraform code for AWS



Terraform is a fairly new project (as most of DevOps tools actually) which was started in 2014. Terraform is powerful and one of the most used tool which allows managing infrastructure-as-code. It allows developers to do a lot of things and does not limit them from doing things in ways which will be difficult to support or integrate with. The collaborative infrastructure-as-code workflow is built on multiple IT best practices (like using version control and preventing manual changes), and one must adopt the foundations before we can fully adopt the suggested workflow. Achieving state-of-the-art provisioning practices is a journey, with several distinct stops along the way.

This describes the supported Terraform practices and how to adopt them.

**Structuring of Terraform configurations**

Main.tf is the main file that has provider and creates all the resources but increases the complexity of readability. Terraform code can be written in a single file but it would be better if we have several files split logically:

* main.tf — call modules, locals and data-sources to create all the resources

* variables.tf — contains information of variables used in main.tf

* outputs.tf — contains outputs from the resources generated in main.tf

By splitting the files in this format it helps in reusability and sharing of code as well as helps in improving whenever required.

**Modules as Building Blocks**

A module is a container for multiple resources that are used together. A module can call other modules, which lets you include the child module’s resources into the configuration in a concise way. Modules can also be called multiple times, either within the same configuration or in separate configurations, allowing resource configurations to be packaged and re-used.

Manage terraform resource with shared modules, this will save a lot of coding time. No need re-invent the wheel! It is because of its reusability nature and also helps in reducing duplication, enable isolation, and enhance testability. Module in can be accessed as by referring to the path in the *module *resource.

![](https://cdn-images-1.medium.com/max/2000/0*t5Tx_3Ktm1w5_Jn7)

Following is a type of skeleton for creating modules in AWS using Terraform:

![](https://cdn-images-1.medium.com/max/2000/0*NrWZyhisZOV68Igt)

Modules* *folder contains terraform modules inherently related to the project. Modules are powerful because they make code more readable

**S3 as the backend for storing tfstate file**

This state file is extremely important; it maps various resource metadata to actual resource IDs so that Terraform knows what it is managing. This file must be saved and distributed to anyone who might run Terraform.

Terraform config can be used to provision many boxes on different infrastructure, each of which could have a different state. As it can also be run by multiple people this state should be in a centralized location (like S3) but *not* git and enable version control on this bucket. Storing in git could expose potentially sensitive data as there’s no encryption. As terraform state file is just a simple text file which would include all the metadata like AWS credentials.

    Example Configuration:

    terraform {
     backend “s3” {
     bucket = “mybucket”
     key = “path/to/my/key”
     region = “us-east-1”
     }
    }

**Locking of state file**

Terraform provides locking to prevent parallel runs against the same state. Locking helps make sure that only one team member runs the Terraform configuration. Locking helps us prevent conflicts, data loss and state file corruption due to multiple runs on same state file.

DynamoDB can be used as a locking mechanism to remote storage backend S3 to store state files. If multiple users are working simultaneously then the name of the DynamoDB table is to use for state locking and consistency. The table must have a primary key named LockID. If not present, locking will be disabled.

    Usage:

    resource “aws_dynamodb_table” “terraform_state_lock” {
     name = “terraform-lock”
     read_capacity = 5
     write_capacity = 5
     hash_key = “LockID”

    attribute {
     name = “LockID”
     type = “S”
     }
    }

    terraform {

    backend “s3” {

    bucket = “terraformbackend”

    key = “terraform”

    region = “us-east-2”

    dynamodb_table = “terraform-lock”

    }

    }

**Use variable files**

Variables on Terraform are a great way to pack your configuration with easily readable data. When you declare variables in the root module of your configuration, you can set their values using CLI options and environment variables. When you declare them in child modules, the calling module should pass values in the module block. They can be set in a number of ways:

* In variable.tf file

* Individually, with the -var command-line option.

* In variable definitions (.tfvars) files, either specified on the command line or automatically loaded.

* As environment variables.

**-var-file application in Terraform commands**

With var-file, you can easily manage the environment (dev/stage/uat/prod) variables and avoid running terraform with long list of key-value pairs. Can easily run terraform as below:

    terraform plan -var-file=terraform.tfvars
    terraform apply -var-file=terraform.tfvars
    terraform destroy -var-file=terraform.tfvars

**Encryption of credentials**

Static credentials can be provided by adding an access_key and secret_key in-line in the AWS provider block:

    Usage:

    provider “aws” {
     region = “us-west-2”
     access_key = “my-access-key”
     secret_key = “my-secret-key”
    }

This method should be avoided as hard-coding credentials into any Terraform configuration is not recommended, and risks secret leakage should this file ever be committed to a public version control system.

The AWS provider offers a flexible means of providing credentials for authentication. The following methods are supported:

* Environment variables

    Usage:

    $ export AWS_ACCESS_KEY_ID=”accesskey”
    $ export AWS_SECRET_ACCESS_KEY=”secretkey”
    $ export AWS_DEFAULT_REGION=”us-west-2"
    $ terraform plan

* Shared credentials file

    Usage:

    provider “aws” {
     region = “us-west-2”
     shared_credentials_file = “/Users/tf_user/.aws/creds”
     profile = “customprofile”
    }

* EC2 Role

    Usage:

    provider “aws” {
     assume_role {
     role_arn = “arn:aws:iam::ACCOUNT_ID:role/ROLE_NAME”
     session_name = “SESSION_NAME”
     external_id = “EXTERNAL_ID”
     }
    }

**Consistent structure and naming convention**

Like procedural code, Terraform code should be written for people to read, consistency will help whenever the changes happen. It is possible to move resources in Terraform state file but it may be harder to do if you have inconsistent structure and naming

**Extraction of metadata**

*Terraform_remote_state* is the only data source that retrieves state data from a Terraform backend. This allows you to use the root-level outputs of one or more Terraform configurations as input data for another configuration.

We have several layers to manage terraform resources, such as network, database, application layers. After you create the basic network resources, such as vpc, security group, subnets, nat gateway in vpc stack. Your database layer and applications layer should always refer the resource from vpc layer directly via *terraform_remote_state* data source.

    Usage:

    data “terraform_remote_state” “vpc” {
     backend = “s3”
     config = {
     bucket = var.s3_terraform_bucket
     key = “${var.environment}/vpc.tfstate”
     region = var.aws_region
     }
    }
     
    # Retrieves the vpc_id and subnet_ids directly from remote backend state files.
    resource “aws_xx_xxxx” “main” {
     # …
     subnet_ids = split(“,”, data.terraform_remote_state.vpc.data_subnets)
     vpc_id = data.terraform_remote_state.vpc.outputs.vpc_id
    }

**File Path instead of Inline Block**

The configuration for some Terraform resources can be defined either as inline blocks or as separate resources. While writing the Terraform code, you should always prefer using a separate resource. As it helps in becoming the code more flexible and configurable. For example, for *user-data.sh*, we should use the file interpolation function to read this file from disk. The *template_file* data source is the file interpolation function that renders a template from a template string, which is usually loaded from an external file.

    data “template_file” “init” {
     template = “${file(“${path.module}/init.tpl”)}”
    }

*path.module* helps to convert the path that is relative to the module folder and this can be one of the structure :

![](https://cdn-images-1.medium.com/max/2000/0*JD5_EdegQZ1ud3wo)

**Tagging Your Amazon Resources**

You can optionally assign your own metadata to each resource in the form of *tags*. Tags enable you to categorize your AWS resources in different ways, for example, by purpose, owner, or environment. This is useful when you have many resources of the same type — you can quickly identify a specific resource based on the tags you’ve assigned to it.

**Conclusion**

Terraform is a powerful tool to enable you and your teams to define and deploy infrastructure in a controllable and maintainable way. Performing these best exercises can help you to reduce downtime and allow engineers to focus on their primary job — providing business value.
