Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m18[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m6[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m7[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m14[39m, end: [33m15[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m16[39m, end: [33m24[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m25[39m, end: [33m26[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m27[39m, end: [33m35[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m39[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m47[39m, end: [33m51[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m53[39m }

# 100 Days of DevOps — Day 16- Building VPC using Terraform

Welcome to Day 16 of 100 Days of DevOps, Let continue our journey, yesterday I discussed terraform, today let’s build VPC using terraform

**Pre-requisites:** I am assuming you already understand VPC, in case if you need a refresher
[**Introduction to VPC — Part 2**
*In the second part of this series, we actually start building VPC using AWS console. This is the continuation of Part 1*medium.com](https://medium.com/@devopslearning/introduction-to-vpc-part-2-b0cb4ab41e8f)
[**100 Days of DevOps — Day 15- Introduction to Terraform**
*Welcome to Day 15 of 100 Days of DevOps, Let continue our journey and focus on Automation especially on Infrastructure…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-15-introduction-to-terraform-7a168dec8d38)
> What is VPC?

*Without going to all the nitty-gritty details of VPC, first, let’s try to understand VPC in the simplest term. Before the cloud era, we use to have datacenters where we deploy all of our infrastructures.*

![](https://cdn-images-1.medium.com/max/2000/0*riYnQrOA97UKOOf9)

*You can think of VPC as your datacentre in a cloud but rather than spending months or weeks to set up that datacenter it’s now just a matter of minutes(API calls). It’s the place where you define your network which closely resembles your own traditional data centers with the benefits of using the scalable infrastructure provided by AWS.*

* *Today we are going to build the first half of the equation i.e VPC*

![](https://cdn-images-1.medium.com/max/2376/1*Lc5A0tkYwCGxnYv8tBhWgg.jpeg)

* *Once we create the VPC using AWS Console, these things created for us by-default*

    ** Network Access Control List(NACL)*

    ** Security Group*

    ** Route Table*

* *We need to take care of*

    ** Internet Gateways
    * Subnets
    * Custom Route Table*

*But the bad news is as we are creating this via terraform we need to create all these things manually but this is just one time task, later on, if we need to build one more VPC we just need to call this module with some minor changes(eg: Changes in CIDR Range, Subnet) true Infrastructure as a Code(IAAC)*

*This is how my terraform VPC module structure look like*

    *$ tree*

    *├── main.tf*

    *├── vpc_networking*

    *│   ├── main.tf*

    *│   ├── outputs.tf*

    *│   └── variables.tf*

    *├── outputs.tf*

    *├── terraform.tfvars*

    *└── variables.tf*

* *So the first step is to create a data resource, what data resource did is to query/list all the AWS available Availablity zone in a given region and then allow terraform to use those resource.*

<iframe src="https://medium.com/media/cb1ef4f92794771ce81dfff72d0a8679" frameborder=0></iframe>

*For more info*
[**AWS: aws_availability_zones - Terraform by HashiCorp**
*Provides a list of Availability Zones which can be used by an AWS account.*www.terraform.io](https://www.terraform.io/docs/providers/aws/d/availability_zones.html)

* ***Now it’s time to create VPC***

<iframe src="https://medium.com/media/8832ef940093e39748992c6743946823" frameborder=0></iframe>

* [*enable_dns_support](https://www.terraform.io/docs/providers/aws/r/vpc.html#enable_dns_support) - (Optional) A boolean flag to enable/disable DNS support in the VPC. Defaults true. Amazon provided DNS server(**AmazonProvidedDNS)** can resolve Amazon provided private DNS hostnames, that we specify in a private hosted zones in Route53.*

* [*enable_dns_hostnames](https://www.terraform.io/docs/providers/aws/r/vpc.html#enable_dns_hostnames) - (Optional) A boolean flag to enable/disable DNS hostnames in the VPC. Defaults false. This will ensure that instances that are launched into our VPC receive a DNS hostname.*

*For more info*
[**AWS: aws_vpc - Terraform by HashiCorp**
*Provides an VPC resource.*www.terraform.io](https://www.terraform.io/docs/providers/aws/r/vpc.html)

* ***Next step is to create an internet gateway***

    ** Internet gateway is a horizontally scaled, redundant and highly avilable VPC component.
    * Internet gateway serves one more purpose, it performs NAT for instances that have been assigned public IPv4 addresses.*

<iframe src="https://medium.com/media/dda8b8db461abede07149dda531d5b8b" frameborder=0></iframe>
[**AWS: aws_internet_gateway - Terraform by HashiCorp**
*Provides a resource to create a VPC Internet Gateway.*www.terraform.io](https://www.terraform.io/docs/providers/aws/r/internet_gateway.html)

* ***Next step is to create Public Route Table***

* *Route Table: Contains a set of rules, called routes, that are used to determine where network traffic is directed.*

<iframe src="https://medium.com/media/2594d6de29547adbf065923ee497a586" frameborder=0></iframe>

*For more info*
[**AWS: aws_route_table - Terraform by HashiCorp**
*Provides a resource to create a VPC routing table.*www.terraform.io](https://www.terraform.io/docs/providers/aws/r/route_table.html)

* *Now it’s time to create Private Route Table. If the subnet is not associated with any route by default it will be associated with Private Route table*

<iframe src="https://medium.com/media/1371f9e9a10258c71ba1abb867f83b3b" frameborder=0></iframe>

* *Next step is to create Public Subnet*

<iframe src="https://medium.com/media/eb36542952dc5ffef597ad2499e95504" frameborder=0></iframe>

* *Private Subnet*

<iframe src="https://medium.com/media/b33caabb557d59f74e474de32b2857fd" frameborder=0></iframe>
[**AWS: aws_subnet - Terraform by HashiCorp**
*Provides an VPC subnet resource.*www.terraform.io](https://www.terraform.io/docs/providers/aws/r/subnet.html)

* *Next step is to create a route table association*

<iframe src="https://medium.com/media/c8a3f3c26d5b0c15c9f2b3aef38eed8b" frameborder=0></iframe>
[**AWS: aws_route_table_association - Terraform by HashiCorp**
*Provides a resource to create an association between a subnet and routing table.*www.terraform.io](https://www.terraform.io/docs/providers/aws/r/route_table_association.html)

* ***Security Group** acts as a virtual firewall and is used to control the traffic for its associated instances.*

<iframe src="https://medium.com/media/6a8da48c14c1cf295639cd72fbef879a" frameborder=0></iframe>
[**AWS: aws_security_group - Terraform by HashiCorp**
*Provides details about a specific Security Group*www.terraform.io](https://www.terraform.io/docs/providers/aws/d/security_group.html)

* This is how our variables file look like

<iframe src="https://medium.com/media/7547724ccc8dc57be1e89517627a3085" frameborder=0></iframe>

* *Now let’s test it*

* *Initialize a Terraform working directory*

<iframe src="https://medium.com/media/6ac95ce6a01bc5e7db732f2302e0a42e" frameborder=0></iframe>

* *Execute terraform plan*

* *Generate and show an execution plan*

<iframe src="https://medium.com/media/8db311a8b771cef40dd65c5456b1e732" frameborder=0></iframe>

* *Final step terraform apply’*

* *Builds or changes the infrastructure*

<iframe src="https://medium.com/media/0031532e4cefcf4cab577f6f1fdfa4f4" frameborder=0></iframe>

**GitHub Link**
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/tree/master/two-tier-environment/vpc_networking)

### **Terraform Module**

* You can think of Terraform Module like any other language module eg: Python, it’s the same terraform file but just that after creating a module out it we can re-use that code OR Instead copy-pasting the code the same code in different places we can turn into reusable modules.

* Now the code structure will look like this

![](https://cdn-images-1.medium.com/max/2000/1*DNLYOcYlkaTRXdQq8ZMkqw.png)

**NOTE:** Please ignore the terraform.tfstate.* files

The syntax for the module

    **module** "NAME" {
      source = "SOURCE"
    
      [CONFIG ...]
    }

<iframe src="https://medium.com/media/f9275b477048d3e8674c3a20ed88fe08" frameborder=0></iframe>

* variables.tf file look like this

<iframe src="https://medium.com/media/2af3d221449ad1dc4e1b1590e52b372a" frameborder=0></iframe>

* terraform.tfvars

<iframe src="https://medium.com/media/4fdf4c29432701d7343bab200b70efaa" frameborder=0></iframe>

GitHub Link
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/tree/master/two-tier-environment)

* Few people asked me this question, how to run the terraform code

    **# Step1**

    $ git clone https://github.com/100daysofdevops/100daysofdevops.git

    Cloning into '100daysofdevops'...

    remote: Enumerating objects: 83, done.

    remote: Counting objects: 100% (83/83), done.

    remote: Compressing objects: 100% (64/64), done.

    remote: Total 83 (delta 28), reused 41 (delta 12), pack-reused 0

    Unpacking objects: 100% (83/83), done.

    **#Step2**

    cd 100daysofdevops/two-tier-environment/

    **#Step3 **

    Run all the terraform command

    * terraform init
    * terraform plan
    * terraform apply

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
