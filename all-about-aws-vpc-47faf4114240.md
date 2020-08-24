Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m8[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m11[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m13[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m14[39m, end: [33m29[39m }

# All About AWS VPC

Skip to content

![](https://cdn-images-1.medium.com/max/2000/1*lscY2BjizCSB4pLZB8RfpQ.jpeg)

After making my hands dirty on AWS from last 6 years, I thought now I am ready to write on AWS VPC & I should share my learning with others. I generally don’t write something on hands on but this write up is little different as compared to all my previous articles.

Today we will talk about VPC & all the related/associated components to VPC.

**What is a VPC (Virtual Private Cloud) ?**

This is private virtual network with AWS resources associated with your AWS account & similar to your data center network but with capabilities to scale AWS infrastructure.

You can also say VPC as your own virtual logical data center in AWS.

Below are AWS VPC related components,

1. Subnets

1. Internet Gateway

1. Routing Table

1. NAT Gateways & NAT Instances

1. VPC Endpoint

Let us try to go through each one of them one after another,

**Subnets** are the range of IPs in a VPC. Subnets can not span across availability zone, while VPCs can.

* 10.0.0.0 – 10.255.255.255 (10/8 prefix)

* 172.16.0.0 – 172.31.255.255 (172.16/12 prefix)

* 192.168.0.0 – 192.168.255.255 (192.168/16 prefix)

Subnets can be public as well as private. When subnets are created we need to make sure they are without any overlapping IP addresses. You also need to be aware of CIDR block allocations. You can take numerous websites help for calculating number of IP address allocations in each CIDR block & number of IPs you are looking for.

[http://www.subnet-calculator.com/cidr.php](http://www.subnet-calculator.com/cidr.php)

As you can see in below screenshot I have created one VPC in my account with CIDR block **10.0.0.0/16, **which gives 65,536 IP addresses.

![VPC Creation](https://cdn-images-1.medium.com/max/2000/0*cv5QNLSf7dbTBSlS)*VPC Creation*

I have broken down this VPC into two subnets with each CIDR block allocation as shown in below screenshot & each subnet has maximum IP addresses available are 256. In AWS for our actual use AWS gives **251** IP addresses because Amazon keeps ***5** *IP addresses for their internal use.

![Subnet CIDR Block assignment](https://cdn-images-1.medium.com/max/2000/0*-HCEpaT108VwJfT4)*Subnet CIDR Block assignment*

By Default subnets are marked as private.When any VPC is created Main routing table ,Network ACL & Security group is created with default values. We can make it public by enabling auto assign public IP addresses to the resources, so that these can be accessed from outside. In below screenshot you can see I have made one of the subnet as public subnet.

![Subnet CIDR Block assignment](https://cdn-images-1.medium.com/max/2000/0*2ntvsiHqf4hNu606)*Subnet CIDR Block assignment*

![Making Subnet Public](https://cdn-images-1.medium.com/max/2000/0*1K-XntbNE8RBKRw7)*Making Subnet Public*

**Internet Gateway**

We have created subnet public but we haven’t given an access to external world & we can give that access by adding internet gateway.

Internet gateway can be created and attached to only one VPC at a time. In multiple availability zone you should create multiple internet gateways. This is to avoid any impact to your internet facing applications or resources because of any issues. You are disconnected from internet if your internet gateway stops working. Amazon promises to be highly available.

In our exercise I have created one gateway & attached it to our VPC as shown in below diagram.

![Gateway Creation & Association](https://cdn-images-1.medium.com/max/2000/0*pHent3zaWSqyD7f_)*Gateway Creation & Association*

**Routing Table**

When we create VPC with two subnets we get Main routing table which has entries related to both subnets, it also has entries related to how subnets can talk to each other. Now we have internet gateway but we need to give our subnet a way to internet gateway, this we can give by adding below entries into routing table as shown in diagram, but one thing to take care is we don’t want to expose both the subnets to external world, so we will not add entries to internet gateway in main routing table. We will create another routing table & associate public subnet with a routing table with internet gateway entries. As shown in below diagrams,

![Subnet Association with Routing Table](https://cdn-images-1.medium.com/max/2000/0*F3eHA6gZA_6dWWyd)*Subnet Association with Routing Table*

![Adding entries to Internet Gateway](https://cdn-images-1.medium.com/max/2000/0*trDsqbzM5vrz6sBV)*Adding entries to Internet Gateway*

Now we have successfully created a VPC with two subnets with one of them is public & other one is private. By default the two subnets can not talk to each other, after we make changes at the security group level & apply those security groups to the resources like EC2.

Let us extend this exercise further where we want private subnet resources like EC2 to have one way access to internet for downloading patches from internet & at the same time we want to make sure our entire subnet is not public.

**NAT(Network Address Translation) Gateway or NAT(Network Address Translation) Instances**

As the name suggests NAT Instance is nothing but EC2 instance which provide access to resource in private subnet to internet gateway & NAT gateway is the offering of AWS, which is again highly available.

**NAT Instance**

These are EC2 instance which acts as a gateway in between incoming & outgoing traffic.These need to be created under public subnet. The entries related to NAT instance must be added into routing table & these entries must be added in main routing table,so that private subnet have route to internet gateway. As you can see in below diagram I have created an EC2 instance from an available NAT AMI & following diagram I have disabled Source & Destination check to act this instance in the capacity of Gateway.

![NAT EC2 Instance from an AMI](https://cdn-images-1.medium.com/max/2000/0*8L8OpUfw1L8EmpeL)*NAT EC2 Instance from an AMI*

![Disable Source/Destination Check](https://cdn-images-1.medium.com/max/2000/0*jxjfgfu-UX7c1sw9)*Disable Source/Destination Check*

**NAT Gateway**

NAT gateway is AWS offering & it is much efficient with high availability in an availability zone. It is not associated with any security group like NAT instance. Enterprises prefer NAT gateway as there is no maintenance issues. All you need to do is add an entry in main routing table. We also need not to worry about security group configuration. NAT Gateway needs to be created in public subnet. As shown in below diagram.

![NAT Gateway](https://cdn-images-1.medium.com/max/2000/0*AHEcLuHbXCsdM5CD)*NAT Gateway*

**Network ACL(Access Control List)**

It is an additional level of security for VPC & it acts as firewall for controlling in or out traffic.It comes by default with VPC & by default it allows all the incoming as well as outgoing traffic. You can create custom NACLs & they are by default created with all the traffic denied. It takes precedence over security group. The number ordering takes precedence in execution of NACL rules, in below diagram, if I add DENY rule with 99 number, it will be executed first . Any number of subnets can be associated with one NACL but one Subnet is associated with only one NACL. As you can see in below diagram default VPC, I can modify it to limit some traffic & if I want to make that effective I need to add an entry with Rule# less than 100.

![NACL Default Rule](https://cdn-images-1.medium.com/max/2000/0*JIc2Uc2T0QZET2YZ)*NACL Default Rule*

NACL rules are different from security groups & Outbound Rules entries need to be managed separately than Inbound Rule. In case of security group if you are adding an inbound rule by default outbound rule is enabled.

**VPC Gateways**

Suppose you want to use some of the AWS services from your VPC through a private reliable network & you don’t want your traffic to be leaving the AWS network. VPC Gateway is the way to access it. Generally direct connect & NAT gateway are the alternatives. Directconnect is a again private link setup in between your network & AWS network.

There are two types of VPC gateways

**Interface Endpoints: **There are lot of services can be accessed from interface endpoints. It’s huge list & can be accessed from AWS documentation.

**Gateway Endpoints: **Support S3 & Dyanamo DB.

As you can see S3 endpoint creation is below & it adds those entries into routing table. You should add those entries in main routing table where all the subnets have access to it.

![S3 Gateway Enpoint](https://cdn-images-1.medium.com/max/2000/0*Vhxk7mYetCxM6UDd)*S3 Gateway Enpoint*

We should be able to execute all the s3 commands without going outside AWS network & privately.

So I hope you have enjoyed all the components related to VPC & this will certainly help you in future for understanding VPC and related components. It is one time activity mostly but understanding about it is very much important. I am keeping direct connect, transit gateway & bastion hosts for another article till then stay tuned & happy learning.

![](https://cdn-images-1.medium.com/max/2000/0*JEya-dYXW8GFY8mx)

## Published by Sachin Kapale

Sachin Kapale is having IT experience of 17+ yrs,with multiple MNCs & fortune 500 clients. He has certifications in Orale,JAVA,Websphere, UDB, HP area. He currently plays multiple roles in Architectural space. He works with senior management to establish strategic plans and objectives, document technical strategies to facilitate operational business needs , Assess systems requirements and design against best practices, technology advances, industry standards, and business needs. Provides direction on implementing feasible, cost-effective solutions to the overall system architecture and design to meet these needs Also Sachin works closely with Health Services Operations management and staff, Operations Systems’ technical engineers, and process engineers to provide guidance and ensure adequate design for meeting Health Services’ business objectives and strategy & Serve as the key technical point of contact in developing the technical solutions in response to Request for Proposals, Change Requests, and business needs. Sachin also review federal, state, and company policies to determine applicability to systems functionality, design, and operation.He also develop strategies to fulfill requirements and assess effectiveness of implementation. He also collaborate and coordinate with appropriate internal and external groups to ensure the confidentiality and security of all corporate information His expertise is in designing micro services based architecture on cloud based as well as containerized environments. Working as success serving as key point of contact for design of complex information technology solutions. He has technical papers presentations on Web 2.0. He can perform comfortably in a fast-paced, deadline oriented environment Perform quantitative and qualitative analyses of existing business processes based on in-depth industry knowledge of organizational and client objectives His current passion is Cloud based architecture, Artificial Intelligence ,machine learning in healthcare call center business and in back office area.[View all posts by Sachin Kapale](https://sachinkapale.blog/author/sachinkapale/)
