Unknown markup type 10 { type: [33m10[39m, start: [33m170[39m, end: [33m184[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m121[39m, end: [33m135[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m119[39m, end: [33m161[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m164[39m, end: [33m208[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m12[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m17[39m, end: [33m32[39m }

# High Availability VPC With Terraform

VPC with Terraform

In this blog, we direct you how to create High availability Amazon VPC with multiple VPC subnets (private and public) in different AWS availability zones.

**Amazon VPC: **Amazon Virtual Private Cloud (Amazon VPC) enables us to launch AWS resources into an AWS virtual network that we defined. This virtual network closely resembles a traditional network that we operate in our own data center.

[**Terraform](https://www.terraform.io/): **It is an open source tool. Terraform is helping us to safely and predictably create, change, and improve the infrastructure of Amazon Cloud resources.

**Step-1: Pre-Requests To Create AWS VPC Using Terraform:**

**A.** We required AWS IAM API keys(access key and secret key) with creating and deleting permissions for AWS resources. 
**B.** and Terraform should be installed on the Ec2 Instance or Local Computer. If Terraform does not exist you can download and install it from [***here](https://www.terraform.io/downloads.html)**.
***C.** GitHub command line tools should be installed.

**Step-2: Amazon Resources Creating Using Terraform.**

1. Creating AWS VPC with 10.0.0.0/16 CIDR.

1. Creating Multiple AWS VPC Subnets which contains
**Amazon VPC Private Subnets: (**Instances in Private Subnet would not be reachable from internet)**
**app-subnets(Ec2 Application Instances)
db-subnets( Amazon RDS instance or EC2 Database Instances)
**Amazon VPC Public Subnets: (**Instances in Public subnet would be reachable from internet; which means traffic from internet can hit a machine in Public Subnet) **
**elb-subnets (Amazon Ec2 Load Balancers)
nat-subnets(Amazon VPC NAT Gateway)

1. Creating AWS VPC Internet Gate Way and attach it to AWS VPC.

1. Creating public and private AWS VPC Route Tables.

1. Creating AWS VPC NAT Gateway.

1. Associating AWS VPC Subnets with VPC route tables.

**Step -3: Creating the Amazon VPC:**

In this step, We build the Amazon VPC using the Terraform script which is provided on GitHub. 
To get the Terraform script, clone or download from the GitHub repository provided below. It contains the complete infrastructure code to build Amazon VPC.

Use the following command to clone or download the GitHub repository.

[git clone https://github.com/vineet67sharma/AWS-Terraform](https://github.com/vineet67sharma/AWS-Terraform)

The GitHub repository contains the following files.

**vpc-variable.tf:**
The vpc-variables.tf file contains the global variables required to build Amazon VPC. The Variable file contains aws_access_key, aws_secret_key, aws_region, availability_zone1, availability_zone2.

We can change the Variable values as per your requirement and values.

**vpc-main.tf**
The vpc-main.tf contains the complete code which required to build the highly available Amazon VPC. The Terraform code launch following resources in Amazon account Amazon VPC, Subnets, Internet Gateway, Route tables, Association route tables with route table and enabled route rules.

**aws.tf
**The aws.tf is the providers configuration file. it contains the AWS API keys to communicate with AWS API to provision the resources.

**terraform.tfvars
**The Terrafrom.tfvars is the default name for the variable input file in Terraform.
Terraform reads the input values from the file. We need to place or replace the AWS API Key in the file.

![replace the keys](https://cdn-images-1.medium.com/max/2000/1*LpXHV_JOEKVZ8nUy1AAEnw.png)*replace the keys*

**Step -4: Build Amazon VPC Infrastructure:**

1. To initialise the working directory containing Terraform configuration files. This is the first command that should be run after writing a new Terraform configuration.
$ terraform init

![](https://cdn-images-1.medium.com/max/3220/1*PLb-9IeHknJ8nuzS6mtaMQ.png)

2. The terraform plan command is used to create an execution plan. It display the resources which are its provisions. 
$ terraform plan

![](https://cdn-images-1.medium.com/max/2816/1*_3GN_BU1ZYoz4XMKo6_VoA.png)

3. The Terraform apply command is used to apply the changes required to reach the desired state of the configuration.
*#*terraform apply -var-file terraform.tfvars

![vpc](https://cdn-images-1.medium.com/max/4192/1*Bkhjgp-tx3KLvMDUjcAbCQ.png)*vpc*

![subnets](https://cdn-images-1.medium.com/max/4152/1*AidF0hL9EMOyxuYluAGBIA.png)*subnets*

**Step -5: Destroy The Amazon VPC Infrastructure:**

The terraform destroy command is used to destroy the Terraform-managed infrastructure. Run the following command to delete the Amazon VPC which is created above.
*# *terraform destroy -var-file terraform.tfvars

![terraform managed resources destroy](https://cdn-images-1.medium.com/max/2800/1*xLYWMMKZYKWbKzrunNN53Q.png)*terraform managed resources destroy*

The .tfstate and .tfstate.backup holds the last-known state of the infrastructure. We defined the state of an infrastructure in a group of files, bought it all up in a single command and can track future changes with state files.

By the way, check out our best AWS deal: [***https://www.avmconsulting.net/well-architected-review](https://www.avmconsulting.net/well-architected-review)***
