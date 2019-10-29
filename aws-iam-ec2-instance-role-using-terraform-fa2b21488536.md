
# AWS IAM EC2 Instance Role using Terraform

IAM Roles are used to granting the application access to AWS Services without using permanent credentials.

*IAM Role is one of the safer ways to give permission to your EC2 instances.*

*We can attach roles to an EC2 instance, and that allows us to give permission to EC2 instance to use other AWS Services eg: S3 buckets*

***Motivation***

* *Give EC2 instance access to S3 bucket*
> ***Step1***

* *Create a file iam.tf*

* *Create an IAM role by copy-paste the content of a below-mentioned link*

* *assume_role_policy — (Required) The policy that grants an entity permission to assume the role.*
[**AWS: aws_iam_role - Terraform by HashiCorp**
*Provides an IAM role.*www.terraform.io](https://www.terraform.io/docs/providers/aws/r/iam_role.html)

    *resource "aws_iam_role" "test_role" {
      name = "test_role"
    
      assume_role_policy = <<EOF
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Action": "sts:AssumeRole",
          "Principal": {
            "Service": "ec2.amazonaws.com"
          },
          "Effect": "Allow",
          "Sid": ""
        }
      ]
    }
    EOF
    
      tags = {
          tag-key = "tag-value"
      }
    }*

* *This is going to create IAM role but we can’t link this role to AWS instance and for that, we need EC2 instance Profile*
> *Step2*

* *Create EC2 Instance Profile*

    *resource "aws_iam_instance_profile" "test_profile" {
      name = "test_profile"
      role = "${aws_iam_role.test_role.name}"
    }*
[**AWS: aws_iam_instance_profile - Terraform by HashiCorp**
*Provides an IAM instance profile.*www.terraform.io](https://www.terraform.io/docs/providers/aws/r/iam_instance_profile.html)

* *Now if we execute the above code, we have Role and Instance Profile but with no permission.*

* *Next step is to add IAM Policies which allows EC2 instance to execute specific commands for eg: access to S3 Bucket*
> *Step3*

* *Adding IAM Policies*

* *To give full access to S3 bucket*

    *resource "aws_iam_role_policy" "test_policy" {
      name = "test_policy"
      role = "${aws_iam_role.test_role.id}"
    
      policy = <<EOF
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Action": [
            "s3:*"
          ],
          "Effect": "Allow",
          "Resource": "*"
        }
      ]
    }
    EOF
    }*
> *Step4*

* *Attach this role to EC2 instance*

    *resource "aws_instance" "role-test" {
      ami = "ami-0bbe6b35405ecebdb"
      instance_type = "t2.micro"
    **  iam_instance_profile = "${aws_iam_instance_profile.test_profile.name}"**
      key_name = "mytestpubkey"
    }*

*It’s time to execute code*

*1: This will initialize the terraform working directory OR it will download plugins for a provider(example: aws)*

    *terraform init*

*2: Let you see what terraform will do before making the actual changes*

    *terraform plan*

*3: To actually create the instance we need to run terraform apply*

    *terraform apply*
