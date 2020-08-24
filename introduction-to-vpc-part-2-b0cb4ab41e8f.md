Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m8[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m11[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m13[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m14[39m, end: [33m29[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m36[39m, end: [33m39[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m74[39m, end: [33m77[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m41[39m, end: [33m52[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m8[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m8[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m8[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m8[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m10[39m }

# Introduction to VPC — Part 2

In the second part of this series, we actually start building VPC using AWS console. This is the continuation of Part 1

[*https://medium.com/p/cbca3b189329](https://medium.com/p/cbca3b189329/edit)*

***Steps Involved in creating your first VPC***

* *Choose an IP address/CIDR range*

* *Divide your network into sub-networks, and this is for **High Availability**(Different Availability Zone — AZ)*

* ***Routing**, So that our network is reachable from the internet(**Route Table**) and we can send data to the Internet(**Internet Gateway**)*

* *Setup network filters, eg, we want which port/protocol/network range to be allowed or denied(**Security Group/Network ACL**)*

***In AWS Console VPC can be found under***

    *Networking & Content Delivery → VPC → Your VPCs → Create VPC **OR** [https://console.aws.amazon.com/vpc/](https://console.aws.amazon.com/vpc/)*

![](https://cdn-images-1.medium.com/max/5760/1*IuSXdYuWdwQnQiNfDDw4eA.png)

* *Enter the name of your VPC under the Name tag field*

* *Provide the CIDR block as we discussed in Part 1 of this series, I am using 10.0.0.0/16(As this will give me (**65,536 IP addresses**))*

* ***Tenancy: **Default(**shared**) or **Dedicated** hardware(incur extra cost).*
> ***The first step is to choose the IP ranges***

*Some IP ranges recommended by AWS*

* *10.0.0.0 - 10.255.255.255 (10/8 prefix)*

* *172.16.0.0 - 172.31.255.255 (172.16/12 prefix)*

* *192.168.0.0 - 192.168.255.255 (192.168/16 prefix)*

***NOTE:** One of the most important things to remember while choosing a network range is to avoid range that overlaps with your on-premise network or other VPC’s. Later on, if you require to connect your VPC with on-premises network/datacenter via VPN or create VPC Peering between two VPC these overlapping CIDR range will create conflicts.*
> *Some important point to note, as soon as we create VPC, a number of things created for us*

* ***Route table: **Contains a set of rules, called routes, that are used to determine where network traffic is directed.*

![](https://cdn-images-1.medium.com/max/4880/1*A46GoCcUVl9ybwkdh7AFhA.png)

* ***Network Access Control List(NACL): **NACL adds an additional layer to our VPC and controls traffic going in and out of one or more subnets.*

* *The default network ACL is configured to allow all traffic to flow in and out of the subnets to which it is associated*

![](https://cdn-images-1.medium.com/max/4872/1*npLflsWr3dl3HYJsgz4v5A.png)

* ***Security Group: **Security Group acts as a virtual firewall and is used to control the traffic for its associated instances.*

* *VPC comes with a default security group. Any instance not associated with another security group during launch is associated with the default security group.*

![](https://cdn-images-1.medium.com/max/4864/1*-XjfyIgzHLwB51VI9O9RqA.png)
> ***The second step is to choose the subnetwork from the IP range we choose for VPC.***

* *One of the main reason for that is this is how you deploy your application to provide High Availability*

* *The allowed block size is between a /16 netmask (65,536 IP addresses) and /28 netmask (16 IP addresses).*

* *The smallest subnet mask you can choose is /28 as the first four IP, and the last IP is not available for you to use.*

*For example, in a subnet with CIDR block,10.0.0.0/24 the following five IP addresses are reserved:*

* *10.0.0.0: Network address.*

* *10.0.0.1: Reserved by AWS for the VPC router.*

* *10.0.0.2: Reserved by AWS for Amazon DNS Server*

* *10.0.0.3: Reserved by AWS for future use.*

* *10.0.0.255: Network broadcast address. AWS does not support broadcast in a VPC. Therefore they reserve this address.*

* *In the navigation pane, choose Subnets, and then choose to Create a subnet*

![](https://cdn-images-1.medium.com/max/5568/1*2xnSr5awQ0pnjyXd2teDww.png)

* *Fill all the details*

    ** Enter the name of the subnet under Name tag
    * From the drop down, select the VPC that we have just created
    * **Availibility Zone:** Availibility Zones in which to create the subnet, I will Start with first one us-west-2a(If you are setting up in Oregon)
    * **Ipv4 CIDR block:** IPv4 address range use for the subnet,here I am using 10.0.1.0/24 as this will give 256(251 actual) IP address but please choose according to your requirement.*

*Same steps we need to do for other availability zones.*

![](https://cdn-images-1.medium.com/max/5700/1*P8_QLZZq7_imhcohuivmRg.png)
> ***Next step is to create an internet gateway, again on the left pane click on Internet gateways and then choose to Create Internet gateway***

![](https://cdn-images-1.medium.com/max/4832/1*idKhrAlnMGVt-M2KRXD3Aw.png)

* *By default, the Internet gateway is in a detached state*

![](https://cdn-images-1.medium.com/max/3200/1*UJA8VDQYPgAm9NJ313zlBw.png)

* *Attach Internet gateway to a VPC*

![](https://cdn-images-1.medium.com/max/2148/1*qZU7VI710VNmK_wr-w5IEQ.png)

![](https://cdn-images-1.medium.com/max/3456/1*spySs7puzS6C3NBDUh6J2w.png)

***NOTE***

    ** Internet gateway is a horizontally scaled, redundant and highly avilable VPC component.
    * Internet gateway serves one more purpose, it performs NAT for instances that have been assigned public IPv4 addresses.
    * Internet gateway supports both IPv4 and Ipv6 traffic*

***How Internet Gateway Works***

    ** As we understand so far, main work of an Internet gateway is to enable internet communication
    * Another side of the story is instance must have a public ipv4 address or elastic IP that's is associated with a private ipv4 address of our instance.
    * Instance is only aware of Private Ipv4 address.
    * Internet gateway logically provides the one-to-one NAT on behalf of our instance
    * When traffic leaves our VPC subnet and goes to internet, the reply address field is set to public IPv4 address or Elastic IP address to our instance and not the Private IP address.
    * Same way traffic that is destined for the public IPv4 address or Elastic IP address of our instance has its destination address translated into the instance private IPv4 address before traffic delivered to the VPC.*

***P.S: **IPv6 addresses are globally unique and therefore public by default.*
> ***Next step is to create a custom route table***
> ***Routing Traffic***

* *Route Table contains a set of rules, called routes, that are used to determine where network traffic is directed.*

* *Default routing rule says traffic destined from my VPC stay within your VPC.*

* *Each subnet in our VPC must be associated with a route table, that controls the routing for the subnets.*

* *We can’t delete the main route table, but we can replace the main route table with a custom table that we have created(this table will become a default table with each subnet is associated with).*

* *Now if we want to send traffic to the internet, we need to create an Internet Gateway and attach to our VPC and then add a route with destination 0.0.0.0/0 and a target of the Intenet Gateway(igw-xxxxxxx)*

* *When we create a subnet, we automatically associate it main route table for the VPC. The main route table doesn’t contain a route to an internet gateway.*

* *On the left pane, click on Route tables → Create route table*

* *Name your route table and then select your VPC*

![](https://cdn-images-1.medium.com/max/4824/1*HqqWjNKJIHhpr_6KnKCOlw.png)

* *The first row in the table is the local route, which enables instances within the VPC to communicate.*

* *This route is present in every route table by default and **can’t remove it***

* *Next step is to enable Internet Access, to do that click on Edit routes*

![](https://cdn-images-1.medium.com/max/4876/1*cZ8lvdoxdtS1jf5wjW4Ssw.png)

* *Add this route(0.0.0.0/0 to Internet Gateway we created earlier)*

![](https://cdn-images-1.medium.com/max/4644/1*HPwZhhD6QEsYGuXjcnl1zg.png)

*What this will do, it adds a route that sends traffic destined outside the VPC to the internet gateway.*

* *Click on Subnet Associations and then Edit subnet associations and add 10.0.1.0/24, which means 10.0.1.0/24 will be our Public Subnet now and 10.0.2.0/24 will be our Private Subnet(Associated with default route table)*

***NOTE:***

* *If your subnet is associated with a route table that has a route to an internet gateway, it’s known as a** public subnet.***

![](https://cdn-images-1.medium.com/max/4876/1*0qVOjkAWJdsTDU5yZHXo7w.png)

![](https://cdn-images-1.medium.com/max/4808/1*IzcIZaMqJGidXQ8vZjfexQ.png)

* *One more step we need to perform before we launch our instance, under Subnet tab, Auto Assign Public IP is set to no(value is yes for default VPC), we want instance that launched in Public Subnet will get Public IP*

![](https://cdn-images-1.medium.com/max/4848/1*c3q8hms9rUNzYZAtD4kz_w.png)

* *Modify auto-assign IP settings*

![](https://cdn-images-1.medium.com/max/4888/1*Zotl0FiC6P_cmyCy2AjG0Q.png)

![ity group](https://cdn-images-1.medium.com/max/5224/1*-sszFUfTiXzexc2gIYbUSQ.png)*ity group*

***Key takeaway***

* *Once we create the VPC, these things created for us by-default*

    ** Network Access Control List(NACL)*

    ** Security Group*

    ** Route Table*

* *We need to take care of*

    ** Internet Gateways
    * Subnets
    * Custom Route Table*

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/AiGKum-w3Zg" frameborder="0" allowfullscreen></iframe></center>
