
# Creating your first Terraform infrastructure on AWS

Creating your first Terraform infrastructure on AWS

![Terraform AWS](https://cdn-images-1.medium.com/max/2000/0*Z4PaQ5J668yt_iir.png)*Terraform AWS*

Terraform is an agnostic cloud-provisioning tool created by Hashicorp. Terraform allows you to create, manage, and update your infrastructure in a safe and efficient manner. Terraform’s configuration is all done using a language called the [HashiCorp Configuration Language (HCL)](https://github.com/hashicorp/hcl). Apart from other tools out there, Terraform is not constrained to any specific platform and supports many leading cloud providers out there.

[Click here](https://www.terraform.io/docs/providers/) to see a list of Terraform’s up-to-date providers.

In this example, we will show you how to build a simple EC2 instance using Terraform on AWS.

## First Step — Setup AWS Account

Let's assume all our work will be done in the region us-east-1

1. Create a new user in the IAM Section on AWS [here](https://console.aws.amazon.com/iam/home?region=us-east-1#/users).

1. Select **Programmatic access **below and enter your user details.

![](https://cdn-images-1.medium.com/max/2000/0*mqw7TNGMLJrmB-8w)

3. Click next and select the admin group.

![](https://cdn-images-1.medium.com/max/2000/1*loP1FGwFsKrZC11c01TeAA.png)

4. Continue with the steps until you reach the Create User section and confirm the user has been created. Once the user is created you will get an **Access key ID **and **Secret access key. **Store these in a safe location as you will need these later. See below for an example.

![](https://cdn-images-1.medium.com/max/2406/1*gDT7i2zNX02jyqlOsmV3Yw.png)

## Install Terraform

Download Terraform [here](https://www.terraform.io/downloads.html) and follow the guide [here](https://www.terraform.io/intro/getting-started/install.html) on how to install Terraform on your specific system.

Once you have successfully installed Terraform, continue to the next section.

## Build and Destroy your first instance using Terraform

Terraform configuration is deployed using the [(HCL) HashiCorp Configuration Language](https://github.com/hashicorp/hcl).

For more information on HCL [click here](https://www.terraform.io/docs/configuration/syntax.html).

Now that we have created our AWS account and created an IAM user, let’s spin up our first EC2 instance using Terraform.

Let's create a file called instance.tf with the following code.

![instance.tf](https://cdn-images-1.medium.com/max/2000/1*R_XcNQ6PXCKRIhyEHuCRCA.png)*instance.tf*

Here we explicitly state that we are using the AWS provider plugin from Terraform. We provide the access key/secret from the user in IAM that we created. We also supply the region in which we want to make all changes. In this example, we are using “us-east-1".

We then use the resource identifier “aws_instance” to state that we are trying to bring up an EC2 instance followed by the name identifier “example”. This can be anything you desire.

We supply the AMI type on AWS. An AMI is an identifier specific to AWS for the image you wish to install on the instance (Ubuntu/Windows 32/64bit etc..). You can find the specific AMI you wish to deploy per region [here](https://cloud-images.ubuntu.com/locator/ec2/).

We also supply the “instance_type”. In this example, we will use a t2.micro instance type as it is supported by the [AWS Free Tier](https://aws.amazon.com/free/) package.

Now we will run the “terraform init” command where we created our instance.tf file to download and initialize the appropriate provider plugins. In this case, we are downloading the AWS provider plugin we specified in our instance.tf file.

![](https://cdn-images-1.medium.com/max/2000/1*rBNakmuIQO0uXC2U072swQ.png)

Once this is complete, let's run the “terraform plan” command. This will let us see what Terraform will do before we decide to apply it.

![](https://cdn-images-1.medium.com/max/2000/1*RXIibGVV1Ivu7A8CjO2BVw.png)

Now to create the instance, we run the “terraform apply” command.

![](https://cdn-images-1.medium.com/max/2000/1*kErioRG3lUaK3sQEpOFksw.png)

If we go to our EC2 section on the AWS console you can see that a t2.micro instance was created successfully!

**Congratulations You just created your first Terraform based infrastructure using AWS!**

![](https://cdn-images-1.medium.com/max/2000/1*Fx3x8Q6koNLq9HUCxZlU8Q.png)

Now if we wish to destroy the Terraform infrastructure we created, we can simply run “terraform destroy”. This will destroy all the resources we have created in our Terraform infrastructure.

![](https://cdn-images-1.medium.com/max/2000/1*1cIT2auT_eaqGyiK1kvVPA.png)

If you are creating or deleting multiple resources, Terraform is able to handle these in Parallel giving us a faster delivery/destruction of our infrastructure.

## Why use Terraform

Terraform makes it easy to describe and understand the infrastructure you wish to create. If you decide to leverage [Terraform’s Enterprise](https://www.hashicorp.com/products/terraform) solution, you can take advantage of its powerful features.

Here are a couple of features Terraform Enterprise adds to your workflow:

1. [GUI Workspace Management](https://www.terraform.io/docs/enterprise/workspaces/index.html)

1. [A private module registry](https://www.terraform.io/docs/enterprise/registry/index.html)

1. [Sentinel policies](https://www.terraform.io/docs/enterprise/sentinel/manage-policies.html)

1. [Team Management](https://www.terraform.io/docs/enterprise/users-teams-organizations/index.html)

1. [Audit Logging](https://www.terraform.io/docs/enterprise/private/logging.html#audit-logs)
