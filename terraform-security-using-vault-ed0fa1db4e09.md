
# Terraform Security Using Vault

“padlock on door hinge” by Markus Spiske on Unsplash

Terraform is a tool to write, plan and create infrastructure as a code. It helps in safely and predictably create and update infrastructure. We already looked in depth on different aspects of terraform as mentioned below:
1. [Infrastructure as a Code With Terraform](https://medium.com/@mitesh_shamra/infrastructure-as-a-code-with-terraform-e7021bf28d7d)
2. [State management with Terraform](https://medium.com/@mitesh_shamra/state-management-with-terraform-9f13497e54cf)
3. [Manage AWS VPC With Terraform](https://medium.com/@mitesh_shamra/manage-aws-vpc-with-terraform-d477d0b5c9c5)
4. [Module in Terraform](https://medium.com/@mitesh_shamra/module-in-terraform-920257136228)
5. [Terraform Provisioner](https://medium.com/@mitesh_shamra/terraform-provisioner-fa0911d65ce9)

To start using terraform with AWS, we need to provide AWS credentials to terraform AWS provider. Using long-lived AWS credentials for running terraform can be dangerous. If these tokens get leaked or captured by some bad entity, they can cause a lot of damage to our AWS account. We need a solution where we can dynamically generate AWS credentials with limited permissions and expire them once we are done with infrastructure creation.

We looked into Hashicorp Vault [here](https://medium.com/@mitesh_shamra/setup-hashicorp-vault-using-ansible-fa8073a70a56) which has the ability to generate dynamic credentials on demand for a variety of different backends. Dynamic credentials provide us with the capability to generate credentials when needed and not share or store credentials. Short-lived credentials prevent exposure if compromised. As each consumer gets unique credentials, we can easily identify and fix the breach. Vault helps provide automatic credentials rotations and expiry with a better audit trail.

Vault supports many [different backends](https://www.vaultproject.io/docs/secrets/index.html), few of them mentioned below:
1. AWS
2. Google Cloud
3. Azure 
4. Databases (MySQL, PostgreSQL, MSSQL, MongoDB etc)
6. PKI (Certificates)
7. Consul

We are going to use Terraform with Vault for generating dynamic access and secret keys. Terraform has Vault provider for making calls to vault backend. Vault authentication happens using tokens. Each token is assigned to a policy which decides its action and path. We need to generate Vault token so that terraform can talk to vault for enabling AWS backend, defining roles for IAM and generating credentials. We are going to write terraform code using vault provider to access AWS vault secret backend engine for generating dynamic credentials using vault tokens.

It has two parts : one where we are going to enable AWS vault secret engine and provide admin level AWS credentials which vault can use to dynamically generate new credentials. We need to define vault role that maps to a set of permissions in AWS as well as an AWS credential type. In another part, we fetch AWS credentials by providing a role for which credentials need to be generated. These two parts need to separate from each other. In first part we are providing admin level credentials, while the second part has no credentials. Consumers only need to know vault token and role to create their dynamic credentials.

In admin terraform workspace, we follow three steps:
1. Enable the AWS secrets backend
2. Configure credentials that Vault uses to communicate with AWS to generate the other IAM credentials for consumers.
3. Configure a vault role that maps to set of permissions and credential type with which anyone can ask vault to generate credentials with proper vault token.

    provider "vault" {
     address = "${var.vault_addr}"
     token = "${var.vault_token}"
    }

    resource "vault_aws_secret_backend" "aws" {
      access_key = "${var.access_key}"
      secret_key = "${var.secret_key}"
      region = "us-east-2"

      default_lease_ttl_seconds = "120"
      max_lease_ttl_seconds     = "240"
    }

    resource "vault_aws_secret_backend_role" "ec2-admin" {
      backend = "${vault_aws_secret_backend.aws.path}"
      name    = "ec2-admin-role"

    policy = <<EOF
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "iam:*", "ec2:*"
          ],
          "Resource": "*"
        }
      ]
    }
    EOF
    }

During this process, internally vault will connect to AWS using admin credentials. These credentials are used to generate new temporary credentials for a predefined set of roles.

In consumer terraform workspace, we are going to generate credentials against above-created role. During this request, the vault will create an IAM user and attach the specified policy document to the IAM user and return to our terraform code. These credentials are then used in AWS provider for creating AWS resources.

    provider "vault" {
     address = "${var.vault_addr}"
     token = "${var.vault_token}"
    }

    data "vault_aws_access_credentials" "creds" {
      backend = "aws"
      role    = "ec2-admin-role"
    }

    provider "aws" {
      access_key = "${data.vault_aws_access_credentials.creds.access_key}"
      secret_key = "${data.vault_aws_access_credentials.creds.secret_key}"
      region  = "${var.region}"
    }

    #resources
    resource "aws_vpc" "vpc" {
      cidr_block = "${var.cidr_vpc}"
      enable_dns_support   = true
      enable_dns_hostnames = true
      tags {
        "Environment" = "${var.environment_tag}"
      }
    }

    resource "aws_internet_gateway" "igw" {
      vpc_id = "${aws_vpc.vpc.id}"
      tags {
        "Environment" = "${var.environment_tag}"
      }
    }

    resource "aws_subnet" "subnet_public" {
      vpc_id = "${aws_vpc.vpc.id}"
      cidr_block = "${var.cidr_subnet}"
      map_public_ip_on_launch = "true"
      availability_zone = "${var.availability_zone}"
      tags {
        "Environment" = "${var.environment_tag}"
      }
    }

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

    resource "aws_route_table_association" "rta_subnet_public" {
      subnet_id      = "${aws_subnet.subnet_public.id}"
      route_table_id = "${aws_route_table.rtb_public.id}"
    }

    resource "aws_security_group" "sg_22" {
      name = "sg_22"
      vpc_id = "${aws_vpc.vpc.id}"

    # SSH access from the VPC
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

    resource "aws_key_pair" "ec2key" {
      key_name = "terraformPublicKey"
      public_key = "${file(var.public_key_path)}"
    }

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

This helps us to achieve ability to generate short lived AWS credentials for each terraform run that are automatically revoked after the run.

The complete code can be found in this git repository: [https://github.com/MiteshSharma/TerraformWithVault](https://github.com/MiteshSharma/TerraformWithVault)

***PS: If you liked the article, please support it with claps. Cheers***
