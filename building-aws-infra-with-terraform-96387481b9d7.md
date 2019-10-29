Unknown markup type 10 { type: [33m10[39m, start: [33m61[39m, end: [33m66[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m66[39m, end: [33m71[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m32[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m23[39m, end: [33m41[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m29[39m, end: [33m40[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m29[39m, end: [33m40[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m28[39m, end: [33m41[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m9[39m, end: [33m17[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m37[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m58[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m88[39m, end: [33m99[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m63[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m28[39m, end: [33m54[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m31[39m, end: [33m50[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m96[39m, end: [33m107[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m58[39m, end: [33m67[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m65[39m, end: [33m74[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m35[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m166[39m, end: [33m174[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m79[39m, end: [33m86[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m23[39m, end: [33m29[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m75[39m, end: [33m78[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m83[39m, end: [33m92[39m }

# Building AWS Infra with Terraform

Building AWS Infra with Terraform 1

### Part 1: Infrastructure as Code using Terraform

This is a tutorial that teachers how to build an AWS infrastructure using [**Terraform](https://www.terraform.io/)**, starting with building a core network infrastructure as the ***infrastructure concern*** (or layer) and then building a web application as the ***web application concern***.

### The Concerns as Modules

We’ll implement this using a modular approach with [**Terraform](https://www.terraform.io/)** modules, with each *concern* as a module.

The *infrastructure* will have two sub-modules: *network* and *security*, while the *web application* will have two sub-modules: *application* and *database*.

![](https://cdn-images-1.medium.com/max/2000/1*n57M7JvfCJUqroC3A5TDXA.png)

### Infrastructure Concern

The core infrastructure will use the following AWS resources:

* [**Virtual Private Cloud](https://aws.amazon.com/vpc/)** ([**VPC](https://aws.amazon.com/vpc/)**)

* [**VPC Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)**

* [**VPC Subnets](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html)** (*private* and *public*)

* [**VPC Routing Tables](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)**

* [**VPC Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)**

At the end of this section, you should have a basic grasp on how to create a infrastructure that supports a secure private network and a public facing network.

### Web Application Concern

The web application layer will use the following AWS resources:

* [**Elastic Compute Cloud](https://aws.amazon.com/ec2/)** ([**EC2](https://aws.amazon.com/ec2/)**) instance for a web service

* [**Relational Database Service](https://aws.amazon.com/rds/)** ([**RDS](https://aws.amazon.com/rds/)**) for the DB used by the web service

* [**RDS Database Subnet Groups](https://docs.rightscale.com/cm/dashboard/clouds/aws/rds_subnet_groups.html)**

This application will serve to demonstrate the infrastructure, but also gain knowledge on how to create basic instance with database support. Will will also learn how to import the existing infrastructure output for use with your projects.

## The Tool Requirements

You’ll need essentially two tools:

* **AWS CLI**: [https://aws.amazon.com/cli/](https://aws.amazon.com/cli/)

* **Terraform**: [https://www.terraform.io/](https://www.terraform.io/)

***Note***: The instructions oriented toward using the [**GNU Bash Shell](https://www.gnu.org/software/bash/)**.

**Installing AWS CLI**

If you have a Python Environment, such as one installed with pyenv, you can simply run:

    pip install awscli

Further information on installation on AWS:

* **AWS CLI Installation Guide**: [https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)

I wrote a previous article on creating a Python environment using pyenv:
[**Installing Pythons with PyEnv**
*Using pyenv to manage Python 2 and Python 3 environments*medium.com](https://medium.com/@Joachim8675309/installing-pythons-with-pyenv-54cca2196cd3)

### Configuring AWS CLI

First follow the installation and configuration instructions from AWS docs:

* **AWS CLI Configuration Guide**: [https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)

**Note**: If not obvious, you need to have an AWS account with permissions to create resources, and copy of AWS access key id and AWS secret access key.

I recommend using AWS profiles, as it is easy to switch between professional work and learning home accounts.

    **aws **configure --profile learning

After, you will have a ~/.aws/credentials file that looks something similar to this, but with valid credentials.

    **[default] **
    aws_access_key_id = REDACTED
    aws_secret_access_key = REDACTED

    **[learning]**
    aws_access_key_id = REDACTED
    aws_secret_access_key = REDACTED

And you will want to have a ~/.aws/config file that looks something similar to this, with the desired region specified:

    **[default]**
    region = us-west-2
    output = json

    **[profile learning]**
    region = us-east-2
    output = json

**Note**: Pay attention to added word of profile in the config, which is different that the credentials file.

Test the profile:

    **export AWS_DEFAULT_PROFILE**=learning
    **export AWS_PROFILE**=learning

    **aws** sts get-caller-identity
    **aws **ec2 describe-instances

**Note**: Originally, AWS CLI used AWS_DEFAULT_PROFILE, but recent versions of AWS SDK only support AWS_PROFILE. Set both to be safe.

### About Terraform

[**Terraform](https://www.terraform.io/)**, for all intents and purposes, is a change configuration tool for configuring cloud resources and other resources that can be managed through a web interface ([**RESTful API](https://searchmicroservices.techtarget.com/definition/RESTful-API)**). When configuring cloud resources, the popular term is cloud provisioning.

[**Terraform](https://www.terraform.io/) **purposefully does not configure system resources, as there are popular CAPS ([**Chef](https://www.chef.io/chef)**, [**Ansible](https://www.ansible.com/)**, [**Puppet](https://puppet.com/products/puppet)**, or [**Salt Stack](https://www.saltstack.com/)**) tools for this already.

On macOS, if you have [**Homebrew](https://brew.sh/)** installed, you can install terraform using this:

    brew install terraform

On Windows 10, if you have [**Chocolatey](https://chocolatey.org/)** installed, you can install terraform using this in either command shell or PowerShell run with administrative privileges:

    choco install terraform

Otherwise, you can download and install [**Terraform](https://www.terraform.io/)** manually:

* [https://www.terraform.io/downloads.html](https://www.terraform.io/downloads.html)

## Getting Started

At this point, we should have both terraform and AWS CLI tools installed and available in the path, and we also have AWS environment configured with a profile called learning, and set this to the default.

We are going to create two separate projects areas to support [**SoC](https://en.wikipedia.org/wiki/Separation_of_concerns)** ([**Separation of Concerns](https://en.wikipedia.org/wiki/Separation_of_concerns)**). This allows us to be modular, as well as not blow up our infrastructure while making changes to the web application.

### Organizational Structure

To get started, let’s create the following organizational structure (output of tree -F):

    .
    └── tf-projects/
        ├── infra/
        │   ├── aws.tf
        │   ├── net/
        │   └── sec/
        └── webapp/
            ├── app/
            ├── aws.tf
            └── db/

We can create this in the [**GNU Bash Shell](https://www.gnu.org/software/bash/)** with the following commands:

    **mkdir** -p ~/tf-projects/{infra/{net,sec},webapp/{app,db}}
    **touch** ~/tf-projects/{infra,webapp}/aws.tf

### AWS Provider

We need to use [**AWS provider](https://www.terraform.io/docs/providers/aws/)** we can interact with AWS.

Let’s update our blank aws.tf files:

    **cat** <<-'**AWSPROFILE**' > ~/tf-projects/infra/aws.tf
    **variable** "region" {}

    **provider** "aws" {
      region     = "${var.region}"
    }
    **AWSPROFILE**

    **cp **~/tf-projects/infra/aws.tf ~/tf-projects/webapp/aws.tf

[**Terraform](https://www.terraform.io/)** can pick up our credentials, but not the region, so we need tell [**Terraform](https://www.terraform.io/)** about our default region:

    **export** **TF_VAR_region**=$(
      awk -F'= ' '/region/{print $2}' <(
        grep -A1 "\[.***$AWS_PROFILE**\]" ~/.aws/config)
    )

With that we have our region setup and valid credentials configured, we need to download the [**Terraform](https://www.terraform.io/)** AWS plugin:

    **pushd** ~/tf-projects/infra/ && **terraform** init && **popd**
    **pushd** ~/tf-projects/webapp/ && **terraform** init && **popd**

### AWS Key Pairs

When deploying systems we need to generate [**AWS Key Pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)**, which will be used to access the remote system using a private key.

You can use these steps to create a key pair:

    KEYPATH=".sekrets"
    KEYNAME="deploy-aws"

    **openssl** genrsa -out "**$KEYPATH**/aws.pem" 4096
    **openssl** rsa -in "**$KEYPATH**/aws.pem" -pubout > "**$KEYPATH**/aws.pub"
    **chmod** 400 "**$KEYPATH**/aws.pem"

    **aws** ec2 import-key-pair \
      --key-name **$KEYNAME** \
      --public-key-material \
            "$(**grep** -v PUBLIC **$KEYPATH**/aws.pub | 
               **tr** -d '\n')"

    **cp** **$KEYPATH**/aws.pem $HOME/.ssh/**$KEYNAME**.pem
    **cp** **$KEYPATH**/aws.pub $HOME/.ssh/**$KEYNAME**.pub

Verify you key pair is installed

    **KEYNAME**="deploy-aws"
    **aws** ec2 describe-key-pairs \
      --query 'KeyPairs[*].[KeyName]' \
      --output text | grep **$KEYNAME**

Afterward, when logging into a system created by [**Terraform](https://www.terraform.io/)**, you can use:

    **KEYNAME**="deploy-aws"
    **ssh** -i ~/.ssh/**$KEYNAME**.pem $AWS_HOST_IP

## Conclusion: Until Next Time

This is the first step to configure and setup a AWS [**Terraform](https://www.terraform.io/)** environment (aws and terraform tools), and follow up articles, I’ll walk through the first concern of AWS core infrastructure and then two articles on the web application concern, one the app and one for the database, both organized into modules that the code can be reused.
[**Building AWS Infra with Terraform 2**
*Creating Network Infrastructure and Security Groups*medium.com](https://medium.com/@Joachim8675309/building-aws-infra-with-terraform-2-ca60146666f8)

## References

This tutorial was inspired by web console version tutorial, and does the equivalent resources using [**infrastructure as code](https://en.wikipedia.org/wiki/Infrastructure_as_code)** with [**Terraform](https://www.terraform.io/).**

* [**Building Your First Amazon Virtual Private Cloud (VPC)](https://www.qwiklabs.com/focuses/3629?parent=catalog)** from QwikLabs
