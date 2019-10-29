
# 100 Days of DevOps — Day 15- Introduction to Terraform

Welcome to Day 15 of 100 Days of DevOps, Let continue our journey and focus on Automation especially on Infrastructure as Code (IAAC) and the tool I am going to use for that Purpose is Terraform. In the last few weeks I got the feedback that I shared a bunch of terraform code without any explanation, so next 9 days, I am completely going to dedicate to terraform.
[**100 Days of DevOps — Day 1(Introduction to CloudWatch Metrics)**
*Welcome to Day 1 of 100 Days of DevOps, I would like to start this journey with one of the most critical concepts in…*medium.com](https://medium.com/devopslinks/100-days-of-devops-day-1-introduction-to-cloudwatch-metrics-b04be36307a8)
[**100 Days of DevOps — Day 2 -Introduction to Simple Notification Service(SNS)**
*Welcome to Day 2 of 100 Days of DevOps, Let extend the journey of DevOps Monitoring and Alerting…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-2-introduction-to-simple-notification-service-sns-97137b2f1f1e)
[**100 Days of DevOps -Day 3(Introduction to CloudTrail)**
*Welcome to Day 3of 100 Days of DevOps, Let extend the journey of DevOps Monitoring and Alerting*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-3-introduction-to-cloudtrail-5ce923f44584)
[**100 Days of DevOps — Day 7(AWS S3 Event)**
*Welcome to Day 7 of 100 Days of DevOps, Let extend the journey of DevOps Monitoring and Alerting with S3 events.*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-7-aws-s3-event-cf64c6699ca1)
[**100 Days of DevOps — Day 10- Restricting User to Launch only T2 Instance**
*Welcome to Day 10 of 100 Days of DevOps, Let continue our journey with IAM and let discuss one of the common…*medium.com](https://medium.com/devopslinks/100-days-of-devops-day-10-restricting-user-to-launch-only-t2-instance-509aaaec5aa2)
[**100 Days of DevOps — Day 11- Restricting S3 Bucket Access to Specific IP Addresses**
*Welcome to Day 11 of 100 Days of DevOps, Let continue our journey with IAM and let discuss one of the common…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-11-restricting-s3-bucket-access-to-specific-ip-addresses-a46c659b30e2)

In the next 9 days, this is what we are going to build

![](https://cdn-images-1.medium.com/max/2840/1*DE1zJDZ_ZOo2j6ieHCl8uw.jpeg)

**Day15:** Introduction to Terraform

**Day16:** Building VPC using Terraform

**Day17:** Creating EC2 Instance inside this VPC using Terraform

**Day18:** Add monitoring to these instances using Terraform(CloudWatch and SNS)

**Day19:** Add Application Load Balancer in Front of it using Terraform

**Day20:** Put the frontend web server in Auto-Scaling Group using Terraform

**Day21:** Add Backend MySQL Database using Terraform

**Day22:** Introduction to KMS using Terraform

So next 9 days are full of events please buckle your seat belt and ready for adventure. At the end of 9 days, you have the basic two-tier fully automated environment ready for your startup.
> **What is terraform?**

*Terraform is a tool for provisioning infrastructure(or managing Infrastructure as Code). It supports multiple providers(eg, AWS, Google Cloud, Azure, OpenStack..)*
[**Providers — Terraform by HashiCorp**
*Terraform is used to create, manage, and manipulate infrastructure resources. Examples of resources include physical…*www.terraform.io](https://www.terraform.io/docs/providers/)
> ***Installing Terraform***

*Installing terraform is pretty straightforward, download it from terraform download page and select the appropriate package based on your operating system*
[***Download Terraform*- Terraform by HashiCorp**
Download Terraformwww.terraform.io](https://www.terraform.io/downloads.html)

*I am using Centos7, so these are the steps I need to follow, to install terraform on Centos7*

    ***Step1: Download the Package***

    *# wget [https://releases.hashicorp.com/terraform/0.11.7/terraform_0.11.7_linux_amd64.zip](https://releases.hashicorp.com/terraform/0.11.7/terraform_0.11.7_linux_amd64.zip) — no-check-certificate*

    *— 2018–06–10 16:42:58 —  [https://releases.hashicorp.com/terraform/0.11.7/terraform_0.11.7_linux_amd64.zip](https://releases.hashicorp.com/terraform/0.11.7/terraform_0.11.7_linux_amd64.zip)*

    *Resolving releases.hashicorp.com (releases.hashicorp.com)… 151.101.189.183, 2a04:4e42:2d::439*

    *Connecting to releases.hashicorp.com (releases.hashicorp.com)|151.101.189.183|:443… connected.*

    *WARNING: cannot verify releases.hashicorp.com’s certificate, issued by ‘/C=BE/O=GlobalSign nv-sa/CN=GlobalSign CloudSSL CA — SHA256 — G3’:*

    *Issued certificate not yet valid.*

    *HTTP request sent, awaiting response… 200 OK*

    *Length: 16490308 (16M) [application/zip]*

    *Saving to: ‘terraform_0.11.7_linux_amd64.zip’*

    *100%[==================================================================================================================================================================>] 16,490,308 6.13MB/s in 2.6s*

    *2018–06–10 16:43:01 (6.13 MB/s) — ‘terraform_0.11.7_linux_amd64.zip’ saved [16490308/16490308]*

    ***Step2: Unzip it***

    *# unzip terraform_0.11.7_linux_amd64.zip*

    *Archive:  terraform_0.11.7_linux_amd64.zip*

    *inflating: terraform*

    ***Step3: Add the binary to PATH environment variable***

    *# echo $PATH*

    */usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin*

    *# cp terraform /usr/local/bin/*

*To verify that terraform is installed correctly*

    *# terraform version*

    *Terraform v0.11.7*

*As mentioned above terraform support many providers, for my use case I am using AWS.*

    ***Prerequisites***

    *1: Existing AWS Account(OR Setup a new account)
    2: IAM full access(OR at least have AmazonEC2FullAccess)
    3: AWS Credentials(AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY)*

*Once you have pre-requisites 1 and 2 done, the first step is to export Keys AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY.*

    *export AWS_ACCESS_KEY_ID=”your access key id here”
    export AWS_SECRET_ACCESS_KEY=”your secret access key id here.”*

*This is required as we need to make changes to AWS account*

***NOTE: **These two variables are bound to your current shell, in case of reboot, or if open a new shell window, these changes will be lost*

*With all pre-requisites in place, it’s time to write your first terraform code, but before that just a brief overview about terraform language*

* *Terraform code is written in the HashiCorp Configuration Language(HCL)*

* *All the code ends with the extension of .tf*

* *It’s a declarative language(We need to define what infrastructure we want and terraform will figure out how to create it)*

*In this first example I am going to build EC2 instance, but before creating EC2 instance go to AWS console and think what the different things we need to build EC2 instance are*

* *Amazon Machine Image(AMI)*

* *Instance Type*

* *Network Information(VPC/Subnet)*

* *Tags*

* *Security Group*

* *Key Pair*

*Let break each of these steps by step*

* ***Amazon Machine Image(AMI):** It’s an Operating System Image used to run EC2 instance. For this example I am using Red Hat Enterprise Linux 7.5 (HVM), SSD Volume Type — ami-28e07e50. We can create our own AMI using AWS console or Packer.*
[**Introduction — Packer by HashiCorp**
*Welcome to the world of Packer! This introduction guide will show you what Packer is, explain why it exists, the…*www.packer.io](https://www.packer.io/intro/index.html)

![](https://cdn-images-1.medium.com/max/4704/1*5RkYQcbfCvOQC-UXw5-D-A.png)

*For more info please refer to AWS official doc*
[**Amazon Machine Images (AMI) — Amazon Elastic Compute Cloud**
*An AMI provides a software configuration for your instance.*docs.aws.amazon.com](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)

* ***Instance Type:** Type of EC2 instance to run, as every instance type provide different capabilities(CPU, Memory, I/O). For this example, I am using t2.micro(1 Virtual CPU, 1GB Memory)*

*For more info please refer to*
[**Amazon EC2 Instance Types — Amazon Web Services (AWS)**
*AWS EC2 provides a wide selection of instance types that allow you to scale your cloud resources to meet the…*aws.amazon.com](https://aws.amazon.com/ec2/instance-types/)

* ***Network Information(VPC/Subnet Id):** Virtual Private Cloud(VPC) is an isolated area of AWS account that has it’s own virtual network and IP address space. For this example, I am using default VPC which is the part of new AWS account. In case if you want to set up your instance in custom VPC, you need to add two additional parameters(vpc_security_group_ids and subnet_id) to your terraform code*

*For more information about VPC, please refer*
[**What Is Amazon VPC? — Amazon Virtual Private Cloud**
*Use Amazon VPC to launch AWS resources into a virtual network that is a logically isolated section of the AWS Cloud.*docs.aws.amazon.com](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Introduction.html)

***Tags:** Tag we can think of a label which helps to categorize AWS resources.*

*For more information about tag, please refer*
[**Tagging Your Amazon EC2 Resources — Amazon Elastic Compute Cloud**
*Assign your own metadata tags to each Amazon EC2 resource to manage your instances, images, and other resources.*docs.aws.amazon.com](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html)

***Security Group: **Security group can think like a virtual firewall, and it controls the traffic of your instance*

*For more information about Security Group, please refer*
[**Amazon EC2 Security Groups for Linux Instances — Amazon Elastic Compute Cloud**
*Use security groups and security group rules as a firewall to control traffic for one or more EC2 instances.*docs.aws.amazon.com](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html)

***Key Pair:** Key Pair is used to access EC2 instance*

*For more info about Key Pair, please refer to*
[**Understanding and Getting Your Security Credentials — Amazon Web Services**
*Understand the different types of AWS security credentials (passwords, access keys, multi-factor authentication, key…*docs.aws.amazon.com](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#key-pairs)

*Let review all these parameters and see what we already have and what we need to create to spun our first EC2 instance*

* *Amazon Machine Image(AMI) → ami-28e07e50*

* *Instance Type → t2.micro*

* *Network Information(VPC/Subnet) → Default VPC*

* *Tags*

* *Security Group*

* *Key Pair*

*So we already have AMI, Instance Type and Network Information, we need to write terraform code for rest of the parameter to spun our first EC2 instance*

***Let start with Key Pair***

*Go to terraform documentation and search for aws key pair*
[**AWS: aws_key_pair — Terraform by HashiCorp**
*Provides a Key Pair resource. Currently this supports importing an existing key pair but not creating a new key pair.*www.terraform.io](https://www.terraform.io/docs/providers/aws/r/key_pair.html)

    *resource **"aws_key_pair" "example" **{
      **key_name **= **"example-key"
      public_key **= **"Copy your public key here"
    **}*

*Let try to dissect this code*

* *Under resource, we specify the type of resource to create, in this case we are using aws_key_pair as a resource*

* *an example is an identifier which we can use throughout the code to refer back to this resource*

* *key_name: Is the name of the Key*

* *public_key: Is the public portion of ssh generated Key*

*The same thing we need to do for Security Group, go back to the terraform documentation and search for the security group*
[**AWS: aws_security_group — Terraform by HashiCorp**
*Provides a security group resource.*www.terraform.io](https://www.terraform.io/docs/providers/aws/r/security_group.html)

    *resource **"aws_security_group" "examplesg" **{
      
      **ingress **{
        **from_port   **= 22
        **to_port     **= 22
        **protocol    **= **"tcp"
        cidr_blocks **= [**"0.0.0.0/0"**]
      }
    }*

* *Same thing let do with this code, first two parameter is similar to key pair*

* *ingress: refer to in-bound traffic to port 22, using protocol tcp*

* *cidr_block: list of cidr_block where you want to allow this traffic(This is just as an example please never use 0.0.0.0/0)*

*With Key_pair and Security Group in place it’s time to create first EC2 instance.*

*Final Block: Creating your first EC2 instance*
[**AWS: aws_instance — Terraform by HashiCorp**
*Get information on an Amazon EC2 Instance.*www.terraform.io](https://www.terraform.io/docs/providers/aws/d/instance.html)

    *resource **"aws_instance" "ec2_instance" **{
      **ami **= **"ami-28e07e50"
      instance_type **= **"t2.micro"
      vpc_security_group_ids **= [**"${aws_security_group.examplesg.id}"**]
      **key_name **= **"${aws_key_pair.example.id}"
      tags **{
        **Name **= **"first-ec2-instance"
      **}
    }*

* *In this case we just need to check the doc and fill out all the remaining values*

* *To refer back to the security group we need to use interpolation syntax by terraform*
[**Interpolation Syntax — Terraform by HashiCorp**
*Embedded within strings in Terraform, whether you’re using the Terraform syntax or JSON syntax, you can interpolate…*www.terraform.io](https://www.terraform.io/docs/configuration/interpolation.html)

*This look like*

    *"${var_to_interpolate}*

*Whenever you see a $ sign and curly braces inside the double quotes, that means terraform is going to interpolate that code specially. To get the id of the security group*

    ***"${aws_security_group.examplesg.id}"***

*Same thing applied to key pair*

    ***"${aws_key_pair.example.id}"***

*Our code is ready but we are missing one thing, provider before starting any code we need to tell terraform which provider we are using(aws in this case)*

    *provider **"aws" **{
      **region **= **"us-west-2"
    **}*

* *This tells terraform that you are going to use AWS as provider and you want to deploy your infrastructure in us-west-2 region*

* *AWS has datacenter all over the world, which are grouped in region and availability zones. Region is a separate geographic area(Oregon, Virginia, Sydney) and each region has multiple isolated datacenters(us-west-2a,us-west-2b..)*

*For more info about regions and availability zones, please refer to the below doc*
[**Regions and Availability Zones — Amazon Relational Database Service**
*Protect your application from the failure of a single location by using AWS Regions and Availability Zones in data…*docs.aws.amazon.com](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html)

*So our final code look like this*

    *provider **"aws" **{
      **region **= **"us-west-2"
    **}*

    ***resource "aws_key_pair" "example" **{
      **key_name   **= **"example-key"
      public_key **= **"XXXX"
    **}*

    ***resource "aws_security_group" "examplesg" **{
      **ingress **{
        **from_port   **= 22
        **to_port     **= 22
        **protocol    **= **"tcp"
        cidr_blocks **= [**"0.0.0.0/0"**]
      }
    }*

    ***resource "aws_instance" "ec2_instance" **{
      **ami             **= **"ami-28e07e50"
      instance_type   **= **"t2.micro"
      vpc_security_group_ids **= [**"${aws_security_group.examplesg.id}"**]
      **key_name        **= **"${aws_key_pair.example.id}"***

    ***  tags **{
        **Name **= **"first-ec2-instance"
      **}
    }*
[**lakhera2014/terraform**
*GitHub is where people build software. More than 28 million people use GitHub to discover, fork, and contribute to over…*github.com](https://github.com/lakhera2014/terraform/blob/master/ec2-instance.tf)

*Before running terraform command to spun our first EC2 instance, run terraform fmt command which will rewrite terraform configuration files to a canonical format and style*

    *$ terraform fmt*

    *main.tf*

* *The first command we are going to run to setup our instance is terraform init, what this will do is going to download code for a provider(aws) that we are going to use*

    *$ terraform init*

    ***Initializing provider plugins...***

    *- Checking for available provider plugins on [https://releases.hashicorp.com...](https://releases.hashicorp.com...)*

    *- Downloading plugin for provider "aws" (1.29.0)...*

    *The following providers do not have any version constraints in configuration,*

    *so the latest version was installed.*

    *To prevent automatic upgrades to new major versions that may contain breaking*

    *changes, it is recommended to add version = "..." constraints to the*

    *corresponding provider blocks in configuration, with the constraint strings*

    *suggested below.*

    ** provider.aws: version = "~> 1.29"*

    ***Terraform has been successfully initialized!***

    *You may now begin working with Terraform. Try running "terraform plan" to see*

    *any changes that are required for your infrastructure. All Terraform commands*

    *should now work.*

    *If you ever set or change modules or backend configuration for Terraform,*

    *rerun this command to reinitialize your working directory. If you forget, other*

    *commands will detect it and remind you to do so if necessary.*

* *Next command we are going to run is “terraform plan”, this will tell what terraform actually do before making any changes*

* *This is good way of making any sanity check before making actual changes to env*

* *Output of terraform plan command looks similar to Linux diff command*

*1: (+ sign): Resource going to be created*

*2: (- sign): Resources going to be deleted*

*3: (~ sign): Resource going to be modified*

    *$ terraform plan*

    ***Refreshing Terraform state in-memory prior to plan...***

    *The refreshed state will be used to calculate this plan, but will not be*

    *persisted to local or remote state storage.*

    *------------------------------------------------------------------------*

    *An execution plan has been generated and is shown below.*

    *Resource actions are indicated with the following symbols:*

    *+ create*

    *Terraform will perform the following actions:*

    *+ aws_instance.ec2_instance*

    *id:                                    <computed>*

    *ami:                                   "ami-28e07e50"*

    *associate_public_ip_address:           <computed>*

    *availability_zone:                     <computed>*

    *cpu_core_count:                        <computed>*

    *cpu_threads_per_core:                  <computed>*

    *ebs_block_device.#:                    <computed>*

    *ephemeral_block_device.#:              <computed>*

    *get_password_data:                     "false"*

    *instance_state:                        <computed>*

    *instance_type:                         "t2.micro"*

    *ipv6_address_count:                    <computed>*

    *ipv6_addresses.#:                      <computed>*

    *key_name:                              "${aws_key_pair.example.id}"*

    *network_interface.#:                   <computed>*

    *network_interface_id:                  <computed>*

    *password_data:                         <computed>*

    *placement_group:                       <computed>*

    *primary_network_interface_id:          <computed>*

    *private_dns:                           <computed>*

    *private_ip:                            <computed>*

    *public_dns:                            <computed>*

    *public_ip:                             <computed>*

    *root_block_device.#:                   <computed>*

    *security_groups.#:                     <computed>*

    *source_dest_check:                     "true"*

    *subnet_id:                             <computed>*

    *tags.%:                                "1"*

    *tags.Name:                             "first-ec2-instance"*

    *tenancy:                               <computed>*

    *volume_tags.%:                         <computed>*

    *vpc_security_group_ids.#:              <computed>*

    *+ aws_key_pair.example*

    *id:                                    <computed>*

    *fingerprint:                           <computed>*

    *key_name:                              "example-key"*

    *public_key:                            "XXX"*

    *+ aws_security_group.examplesg*

    *id:                                    <computed>*

    *arn:                                   <computed>*

    *description:                           "Managed by Terraform"*

    *egress.#:                              <computed>*

    *ingress.#:                             "1"*

    *ingress.2541437006.cidr_blocks.#:      "1"*

    *ingress.2541437006.cidr_blocks.0:      "0.0.0.0/0"*

    *ingress.2541437006.description:        ""*

    *ingress.2541437006.from_port:          "22"*

    *ingress.2541437006.ipv6_cidr_blocks.#: "0"*

    *ingress.2541437006.protocol:           "tcp"*

    *ingress.2541437006.security_groups.#:  "0"*

    *ingress.2541437006.self:               "false"*

    *ingress.2541437006.to_port:            "22"*

    *name:                                  <computed>*

    *owner_id:                              <computed>*

    *revoke_rules_on_delete:                "false"*

    *vpc_id:                                <computed>*

    ***Plan:** 3 to add, 0 to change, 0 to destroy.*

    *------------------------------------------------------------------------*

    *Note: You didn't specify an "-out" parameter to save this plan, so Terraform*

    *can't guarantee that exactly these actions will be performed if*

    *"terraform apply" is subsequently run.*

* *To apply these changes, run** terraform apply***

    *$ terraform apply*

    *An execution plan has been generated and is shown below.*

    *Resource actions are indicated with the following symbols:*

    *+ create*

    *Terraform will perform the following actions:*

    *+ aws_instance.ec2_instance*

    *id:                                    <computed>*

    *ami:                                   "ami-28e07e50"*

    *associate_public_ip_address:           <computed>*

    *availability_zone:                     <computed>*

    *cpu_core_count:                        <computed>*

    *cpu_threads_per_core:                  <computed>*

    *ebs_block_device.#:                    <computed>*

    *ephemeral_block_device.#:              <computed>*

    *get_password_data:                     "false"*

    *instance_state:                        <computed>*

    *instance_type:                         "t2.micro"*

    *ipv6_address_count:                    <computed>*

    *ipv6_addresses.#:                      <computed>*

    *key_name:                              "${aws_key_pair.example.id}"*

    *network_interface.#:                   <computed>*

    *network_interface_id:                  <computed>*

    *password_data:                         <computed>*

    *placement_group:                       <computed>*

    *primary_network_interface_id:          <computed>*

    *private_dns:                           <computed>*

    *private_ip:                            <computed>*

    *public_dns:                            <computed>*

    *public_ip:                             <computed>*

    *root_block_device.#:                   <computed>*

    *security_groups.#:                     <computed>*

    *source_dest_check:                     "true"*

    *subnet_id:                             <computed>*

    *tags.%:                                "1"*

    *tags.Name:                             "first-ec2-instance"*

    *tenancy:                               <computed>*

    *volume_tags.%:                         <computed>*

    *vpc_security_group_ids.#:              <computed>*

    *+ aws_key_pair.example*

    *id:                                    <computed>*

    *fingerprint:                           <computed>*

    *key_name:                              "example-key"*

    *public_key:                            "XXXX"*

    *+ aws_security_group.examplesg*

    *id:                                    <computed>*

    *arn:                                   <computed>*

    *description:                           "Managed by Terraform"*

    *egress.#:                              <computed>*

    *ingress.#:                             "1"*

    *ingress.2541437006.cidr_blocks.#:      "1"*

    *ingress.2541437006.cidr_blocks.0:      "0.0.0.0/0"*

    *ingress.2541437006.description:        ""*

    *ingress.2541437006.from_port:          "22"*

    *ingress.2541437006.ipv6_cidr_blocks.#: "0"*

    *ingress.2541437006.protocol:           "tcp"*

    *ingress.2541437006.security_groups.#:  "0"*

    *ingress.2541437006.self:               "false"*

    *ingress.2541437006.to_port:            "22"*

    *name:                                  <computed>*

    *owner_id:                              <computed>*

    *revoke_rules_on_delete:                "false"*

    *vpc_id:                                <computed>*

    ***Plan:** 3 to add, 0 to change, 0 to destroy.*

    ***Do you want to perform these actions?***

    *Terraform will perform the actions described above.*

    *Only 'yes' will be accepted to approve.*

    ***Enter a value: yes <-----(**Make sure to type yes if your satisfied with your changes, note there is no rollback from here)*

    ***aws_key_pair.example: Creating...***

    *fingerprint: "" => "<computed>"*

    *key_name:    "" => "example-key"*

    *public_key:  "" => "XXXX"*

    ***aws_security_group.examplesg: Creating...***

    *arn:                                   "" => "<computed>"*

    *description:                           "" => "Managed by Terraform"*

    *egress.#:                              "" => "<computed>"*

    *ingress.#:                             "" => "1"*

    *ingress.2541437006.cidr_blocks.#:      "" => "1"*

    *ingress.2541437006.cidr_blocks.0:      "" => "0.0.0.0/0"*

    *ingress.2541437006.description:        "" => ""*

    *ingress.2541437006.from_port:          "" => "22"*

    *ingress.2541437006.ipv6_cidr_blocks.#: "" => "0"*

    *ingress.2541437006.protocol:           "" => "tcp"*

    *ingress.2541437006.security_groups.#:  "" => "0"*

    *ingress.2541437006.self:               "" => "false"*

    *ingress.2541437006.to_port:            "" => "22"*

    *name:                                  "" => "<computed>"*

    *owner_id:                              "" => "<computed>"*

    *revoke_rules_on_delete:                "" => "false"*

    *vpc_id:                                "" => "<computed>"*

    ***aws_key_pair.example: Creation complete after 0s (ID: example-key)***

    ***aws_security_group.examplesg: Creation complete after 2s (ID: sg-f34d6a83)***

    ***aws_instance.ec2_instance: Creating...***

    *ami:                             "" => "ami-28e07e50"*

    *associate_public_ip_address:     "" => "<computed>"*

    *availability_zone:               "" => "<computed>"*

    *cpu_core_count:                  "" => "<computed>"*

    *cpu_threads_per_core:            "" => "<computed>"*

    *ebs_block_device.#:              "" => "<computed>"*

    *ephemeral_block_device.#:        "" => "<computed>"*

    *get_password_data:               "" => "false"*

    *instance_state:                  "" => "<computed>"*

    *instance_type:                   "" => "t2.micro"*

    *ipv6_address_count:              "" => "<computed>"*

    *ipv6_addresses.#:                "" => "<computed>"*

    *key_name:                        "" => "example-key"*

    *network_interface.#:             "" => "<computed>"*

    *network_interface_id:            "" => "<computed>"*

    *password_data:                   "" => "<computed>"*

    *placement_group:                 "" => "<computed>"*

    *primary_network_interface_id:    "" => "<computed>"*

    *private_dns:                     "" => "<computed>"*

    *private_ip:                      "" => "<computed>"*

    *public_dns:                      "" => "<computed>"*

    *public_ip:                       "" => "<computed>"*

    *root_block_device.#:             "" => "<computed>"*

    *security_groups.#:               "" => "<computed>"*

    *source_dest_check:               "" => "true"*

    *subnet_id:                       "" => "<computed>"*

    *tags.%:                          "" => "1"*

    *tags.Name:                       "" => "first-ec2-instance"*

    *tenancy:                         "" => "<computed>"*

    *volume_tags.%:                   "" => "<computed>"*

    *vpc_security_group_ids.#:        "" => "1"*

    *vpc_security_group_ids.76457633: "" => "sg-f34d6a83"*

    ***aws_instance.ec2_instance: Still creating... (10s elapsed)***

    ***aws_instance.ec2_instance: Still creating... (20s elapsed)***

    ***aws_instance.ec2_instance: Still creating... (30s elapsed)***

    ***aws_instance.ec2_instance: Still creating... (40s elapsed)***

    ***aws_instance.ec2_instance: Still creating... (50s elapsed)***

    ***aws_instance.ec2_instance: Still creating... (1m0s elapsed)***

    ***aws_instance.ec2_instance: Still creating... (1m10s elapsed)***

    ***aws_instance.ec2_instance: Creation complete after 1m15s (ID: i-0f0cd1c7d727ef8fb)***

    ***Apply complete! Resources: 3 added, 0 changed, 0 destroyed.***

* *What terraform is doing here is reading code and translating it to api calls to providers(aws in this case)*

*W00t you have deployed your first EC2 server using terraform*

*Go back to the EC2 console to verify your first deployed server*

![](https://cdn-images-1.medium.com/max/3000/1*pxyy4Lf6zo8bdIINhyobow.png)

*Let say after verification you realize that I need to give more meaningful tag to this server, so the rest of the code remain the same and you modified the tag parameter*

    ***tags **{
      **Name **= **"first-webserver"
    **}*

* *Run terraform plan command again, as you can see*

***tags.Name: “first-ec2-instance” => “first-webserver”***

    *$ terraform plan*

    ***Refreshing Terraform state in-memory prior to plan...***

    *The refreshed state will be used to calculate this plan, but will not be*

    *persisted to local or remote state storage.*

    ***aws_key_pair.example: Refreshing state... (ID: example-key)***

    ***aws_security_group.examplesg: Refreshing state... (ID: sg-f34d6a83)***

    ***aws_instance.ec2_instance: Refreshing state... (ID: i-0f0cd1c7d727ef8fb)***

    *------------------------------------------------------------------------*

    *An execution plan has been generated and is shown below.*

    *Resource actions are indicated with the following symbols:*

    *~ update in-place*

    *Terraform will perform the following actions:*

    *~ aws_instance.ec2_instance*

    ***tags.Name: "first-ec2-instance" => "first-webserver"***

    ***Plan:** 0 to add, 1 to change, 0 to destroy.*

    *------------------------------------------------------------------------*

    *Note: You didn't specify an "-out" parameter to save this plan, so Terraform*

    *can't guarantee that exactly these actions will be performed if*

    *"terraform apply" is subsequently run.*

* *Now if we can think about it, how does terraform knows that there only change in the tag parameter and nothing else*

* *Terraform keep track of all the resources it already created in **.tfstate files**, so its already aware of the resources that already exists*

    *$ ls -la*

    *total 40*

    *drwxr-xr-x     7 plakhera  wheel    224 Jul 28 11:25 .*

    *drwx------+ 1349 plakhera  wheel  43168 Jul 28 09:37 ..*

    *drwxr-xr-x     6 plakhera  wheel    192 Jul 28 11:24 .idea*

    *drwxr-xr-x     3 plakhera  wheel     96 Jul 28 10:45 .terraform*

    *-rw-r--r--     1 plakhera  wheel    983 Jul 28 11:24 main.tf*

    ***-rw-r--r--     1 plakhera  wheel   7228 Jul 28 11:23 terraform.tfstate***

    ***-rw-r--r--     1 plakhera  wheel   7134 Jul 28 11:23 terraform.tfstate.backup***

* *If you notice at the top it says “**Refreshing Terraform state in-memory prior to plan…”***

* *If I refresh my webbrowser after running terraform apply*

    *$ terraform apply*

    ***aws_key_pair.example: Refreshing state... (ID: example-key)***

    ***aws_security_group.examplesg: Refreshing state... (ID: sg-f34d6a83)***

    ***aws_instance.ec2_instance: Refreshing state... (ID: i-0f0cd1c7d727ef8fb)***

    *An execution plan has been generated and is shown below.*

    *Resource actions are indicated with the following symbols:*

    *~ update in-place*

    *Terraform will perform the following actions:*

    *~ aws_instance.ec2_instance*

    *tags.Name: "first-ec2-instance" => "first-webserver"*

    ***Plan:** 0 to add, 1 to change, 0 to destroy.*

    ***Do you want to perform these actions?***

    *Terraform will perform the actions described above.*

    *Only 'yes' will be accepted to approve.*

    ***Enter a value:** yes*

    ***aws_instance.ec2_instance: Modifying... (ID: i-0f0cd1c7d727ef8fb)***

    *tags.Name: "first-ec2-instance" => "first-webserver"*

    ***aws_instance.ec2_instance: Modifications complete after 3s (ID: i-0f0cd1c7d727ef8fb)***

    ***Apply complete! Resources: 0 added, 1 changed, 0 destroyed.***

![](https://cdn-images-1.medium.com/max/4428/1*l1b1THkVcDg8nISQJrBYSQ.png)

*In most of the cases we are working in team where we want to share this code with rest of team members and the best way to share code is by using GIT*

    *git add main.tf*

    *git commit -m "first terraform EC2 instance"*

    *vim .gitignore*

    *git add .gitignore*

    *git commit -m "Adding gitignore file for terraform repository"*

* *Via .gitignore we are telling terraform to ignore(.terraform folder(temporary directory for terraform)and all *.tfstates file(as this file may contain secrets))*

    *$ cat .gitignore*

    *.terraform*

    **.tfstate*

    **.tfstate.backup*

* *Create a shared git repository*

    *git remote add origin https://github.com/<user name>/terraform.git*

* *Push the code*

    *$ git push -u origin master*

* *To Perform cleanup whatever we have created so far, run terraform destroy*

    *$ terraform destroy*

    ***aws_security_group.examplesg: Refreshing state... (ID: sg-154d6a65)***

    ***aws_key_pair.example: Refreshing state... (ID: example-key)***

    ***aws_instance.ec2_instance: Refreshing state... (ID: i-06d3d45b6102ecc3f)***

    *An execution plan has been generated and is shown below.*

    *Resource actions are indicated with the following symbols:*

    *- destroy*

    *Terraform will perform the following actions:*

    *- aws_instance.ec2_instance*

    *- aws_key_pair.example*

    *- aws_security_group.examplesg*

    ***Plan:** 0 to add, 0 to change, 3 to destroy.*

    ***Do you really want to destroy?***

    *Terraform will destroy all your managed infrastructure, as shown above.*

    *There is no undo. Only 'yes' will be accepted to confirm.*

    ***Enter a value:** yes*

    ***aws_instance.ec2_instance: Destroying... (ID: i-06d3d45b6102ecc3f)***

    ***aws_instance.ec2_instance: Still destroying... (ID: i-06d3d45b6102ecc3f, 10s elapsed)***

    ***aws_instance.ec2_instance: Still destroying... (ID: i-06d3d45b6102ecc3f, 20s elapsed)***

    ***aws_instance.ec2_instance: Still destroying... (ID: i-06d3d45b6102ecc3f, 30s elapsed)***

    ***aws_instance.ec2_instance: Still destroying... (ID: i-06d3d45b6102ecc3f, 40s elapsed)***

    ***aws_instance.ec2_instance: Still destroying... (ID: i-06d3d45b6102ecc3f, 50s elapsed)***

    ***aws_instance.ec2_instance: Destruction complete after 52s***

    ***aws_key_pair.example: Destroying... (ID: example-key)***

    ***aws_security_group.examplesg: Destroying... (ID: sg-154d6a65)***

    ***aws_key_pair.example: Destruction complete after 0s***

    ***aws_security_group.examplesg: Destruction complete after 0s***

    ***Destroy complete! Resources: 3 destroyed.***

*Looking forward from you guys to join this journey and spend a minimum an hour every day for the next 100 days on DevOps work and post your progress using any of the below medium.*

* *Twitter: [@100daysofdevops](http://twitter.com/100daysofdevops) OR @[lakhera2015](https://twitter.com/lakhera2015)*

* *Facebook: [https://www.facebook.com/groups/795382630808645/](https://www.facebook.com/groups/795382630808645/)*

* *Medium: [https://medium.com/@devopslearning](https://medium.com/@devopslearning)*

* *Slack: [https://devops-myworld.slack.com/messages/CF41EFG49/](https://devops-myworld.slack.com/messages/CF41EFG49/)*

* *GitHub Link:[https://github.com/100daysofdevops](https://github.com/100daysofdevops)*

***Reference***
[**100 Days of DevOps — Day 0**
*D-day is just one day away and finally, this is a continuation of the post(I posted a month earlier)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-0-4f2c9750542d)
[**100 Days of DevOps**
*Motivation*medium.com](https://medium.com/@devopslearning/100-days-of-devops-81faf13bf772)
