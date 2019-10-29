Unknown markup type 10 { type: [33m10[39m, start: [33m82[39m, end: [33m99[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m130[39m, end: [33m140[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m165[39m, end: [33m179[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m184[39m, end: [33m199[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m296[39m, end: [33m324[39m }

# How to: Terraform Locking State in S3

tl;dr Terraform, as of v0.9, offers locking remote state management. To get it up and running in AWS create a terraform s3 backend, an s3 bucket and a dynamDB table.

### Intro

When your are building infrastructure with terraform config, a state file, called terraform.tfstat, gets generated locally in the .terraform directory. This state file contains information about the infrastructure and configuration that terraform is managing. When working on a team, it is better to store this state file remotely so that more folks can access it to make changes to the infrastructure.

[Terragrunt](https://github.com/gruntwork-io/terragrunt) first provided a great way to [lock remote terraform state files](https://blog.gruntwork.io/add-automatic-remote-state-locking-and-configuration-to-terraform-with-terragrunt-656a57565a4d) in AWS with an S3 bucket and a dyanmoDB table. Then in v0.9, Terraform added this functionality as well, so in this blog I will be showing how to set up remote state management with locking using Terraform not Terragrunt.

### Whats in the state file?

The state file contains information about what real resources exist for each object defined in the terraform config files. For example, if you have a DNS zone resource created in your terraform config, then the state file contains info about the actual resource that was created on AWS.

Here is an example of creating a DNS zone with Terraform along with its state file:

    # example.tf

    # create a DNS zone called example.com
    resource "aws_route53_zone" "example_dns_zone" {
      name = "example.com"
    }

    # terraform.tfstate

    # in the state file, the DNS zone ID along with its name is stored
    "aws_route53_zone.example_dns_zone": {
        "type": "aws_route53_zone",
        "primary": {
           "id": "Z2D3OUXZHH4NUA",
           "attributes": {       
              "name": "example.com"
            }
         }
    },

### Store State Remotely in S3

If you are working on a team, then its best to store the terraform state file remotely so that many people can access it. In order to setup terraform to store state remotely you need two things: an s3 bucket to store the state file in and an terraform s3 backend resource.

You can create an s3 bucket in a terraform config like so:

    # example.tf

    provider "aws" {
      region = "us-west-2"
    }

    # terraform state file setup
    # create an S3 bucket to store the state file in
    resource "aws_s3_bucket" "terraform-state-storage-s3" {
        bucket = "terraform-remote-state-storage-s3"
     
        versioning {
          enabled = true
        }
     
        lifecycle {
          prevent_destroy = true
        }
     
        tags {
          Name = "S3 Remote Terraform State Store"
        }      
    }

Then create the s3 backend resource like so:

    # terraform.tf

    terraform {
     backend “s3” {
     encrypt = true
     bucket = "terraform-remote-state-storage-s3"
     region = us-west-2
     key = path/to/state/file
     }
    }

### What is locking and why do we need it?

If the state file is stored remotely so that many people can access it, then you risk multiple people attempting to make changes to the same file at the exact same time. So we need to provide a mechanism that will “lock” the state if its currently in-use by another user. We can accomplish this by creating a dynamoDB table for terraform to use.

Create the dynamoDB table like this:

    # example.tf

    # create a dynamodb table for locking the state file
    resource "aws_dynamodb_table" "dynamodb-terraform-state-lock" {
      name = "terraform-state-lock-dynamo"
      hash_key = "LockID"
      read_capacity = 20
      write_capacity = 20
     
      attribute {
        name = "LockID"
        type = "S"
      }
     
      tags {
        Name = "DynamoDB Terraform State Lock Table"
      }
    }

You will need to modify the Terraform S3 backend resource and add in the dynamoDB table:

    # terraform.tf

    terraform {
     backend “s3” {
     encrypt = true
     bucket = "terraform-remote-state-storage-s3"
     dynamodb_table = "terraform-state-lock-dynamo"
     region = us-west-2
     key = path/to/state/file
     }
    }

### Putting pieces together:

Once you’ve created the S3 bucket and dynamoDB table, along with the backend S3 resource referencing those, then you can run your terraform configs like normal with terraform plan and terraform apply commands and the state file will show up in the s3 bucket. After those commands, if you inspect .terraform/terraform.tfstate, you will see that it contains the location of the state file now instead of the actual state file.

    Jessica$ cat .terraform/terraform.tfstate
    {
        "version": 3,
        "backend": {
            "type": "s3",
            "config": {
                "bucket": "terraform-remote-state-storage-s3",
                "dynamodb_table": "terraform-state-lock-dynamo",
                "encrypt": true,
                "key": "example/terraform.tfstate",
                "region": "us-west-2"
            }
        }
    }

### Helpful Resources:

Terraform [state docs](https://www.terraform.io/docs/state/index.html), [backend docs](https://www.terraform.io/docs/backends/index.html), [backends: s3 docs](https://www.terraform.io/docs/backends/types/s3.html)

[Terragrunt blog about Terraform](https://blog.gruntwork.io/why-we-use-terraform-and-not-chef-puppet-ansible-saltstack-or-cloudformation-7989dad2865c)
