
# Create AWS instace using Terraform

Create AWS instace using Terraform

Hi guys, this article brings you to learn how to spinning up an instance AWS using Terraform.

To do that, you have to follow these steps below,

1. Open AWS Account

1. Create IAM admin user

1. Create Terraform file to spin up t2.micro instance

1. Run Terraform apply

It is enough to create free tier account do this. As I mentioned in No 2, You should create IAM role and give ***Admin permission*** access to it when creating it.

![](https://cdn-images-1.medium.com/max/2000/1*gZ0ZsErbGFmlYPt3At9cGA.png)

![](https://cdn-images-1.medium.com/max/2156/1*JJCYpSQSbRKFtr_ZDhqYuw.png)

Also make sure to remember ***Access Key*** and ***Secret*** ***Key***.

Next we gonna create a terraform configuration file to start a EC2 Instance.

Create tf extension file add configuration.

    provider "aws" { 
      access_key = "ACCESS_KEY"
      secrect_key = "SECRECT_KEY"
      region = "us-region"
    }
    resource "aws_instance" "example" {
      ami = "ami-0d72.."
      instance_type = "t2.micro"
    }

Finally you just should run ,

    $ terraform apply
