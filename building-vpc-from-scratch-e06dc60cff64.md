
# Building VPC from Scratch

Nowadays when we want to host any application on a cloud then the first thing we will come across is, how it has to be deployed inside a VPC. Even though cloud providers give a default VPC to start with, It is essential to understand the working and internals of VPC so that you can design your infrastructure in a more secure, manageable and highly available way. It is a must for people who handle production infrastructure, I have seen cases like, with one wrong update in the route table of a VPC took the entire production environment down.

VPC is an isolated environment with much granular control which can resemble a traditional data center. At the end of the blog, I have explained **how a user request packets reach app servers hosted inside a VPC** and how the response is being sent back to the users.

In this blog, I have tried to design it in a way such that, we will start by having only app servers and load balancer server placed inside a VPC and during each subsequent steps, we will encounter a new problem and a component will be introduced and we will see how the component address that problem, finally we will have fully functional VPC.

Let’s imagine we are starting with a simple hello world app, it returns the word “Hello world” for any request, hosted with two app servers behind a Load Balancer (Haproxy is a load balancer). All having public IP and booted inside a VPC.

*(NOTE: For ease of understanding, I have done the explanation step by step, Some of the intermediate steps in this blog are not possible like you can’t have an instance without a subnet inside a VPC.)*
> **Step 1 (VPC with all public IP’s)**

![Step 1 Image](https://cdn-images-1.medium.com/max/5448/1*LF8xJmQtsynboJHN3Aj08Q.jpeg)*Step 1 Image*

Before creating a VPC, decide a primary CIDR range for your VPC, which doesn’t overlap with any other existing VPC CIDR range, so that the new and existing VPC’s can be integrated later if required. The number of usable IPs of any VPC is decided from that. let’s say we have primary CIDR of 10.0.x.x/16, it can have around 65K(2¹⁶) usable IPs. the last part 16 denotes 16 bit out of 32 bit IP address is dedicated to the network part. so the starting two decimal in the IP address (10.0) remains the same for all IP addresses within the network and each machine inside the network will get unique IP by varying the last two decimal part, denoted (x.x) for clarity. (Read more about CIDR at [Wiki](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing))

Create a VPC with a primary CIDR of 10.0.0.0/16 and launch all the instances with assigned public IP. Even though the instances are having public IP, All instances will have a private IP chosen from the base CIDR range as the primary IP(eth0 interface). Public IP’s are virtually mapped to the private IP, you can see this by running “ip addr” command on any public instance, it will show the private IP of the machine. With this config alone, users will not be able to access our service since our instances are not connected to the internet.
> **Step 2 Internet gateway (IG)**

Now we wanted to allow users to access our service through the internet. Having only public IP is not enough to communicate with the internet, you need a gateway to connect internet, ISP does that for you when you connect from home. In a cloud you need to use the Internet gateway (cluster of internet routers managed by cloud providers) to provide internet connectivity for the instances.

![Step 2 Image](https://cdn-images-1.medium.com/max/5448/1*RjHZ84lZxfcFRhDRuO3miA.jpeg)*Step 2 Image*

In the next section we will see how to route packets to the Internet gateway and only the packets which need to reach the internet should be routed to Internet gateway not all packets.

*Internally public IP’s of instances are attached to the internet gateway and it rewrites the packets going out from public instances with the corresponding the public IP of the machine rather than its private IP, so it does the job of network address translation (NAT) for instances with assigned public IP addresses, a small introduction about NAT is done in the below step 5. In our case, all 3 instances are having public IP’s so internet gateway does the NAT for those instances.*
> **Step 3 (Internal route table)**

Each instance will have its own route table, route table controls where the outgoing packets should be routed, basically route table tells what is the **next hop of packets** to reach any particular destination. A route table rule is an entry with two parts (destination IP range and target), each IP packet will have destination IP where it needs to go, destination IP will be checked against all rules present in the route table, when there is a match then the packet is sent to the corresponding target of the matched rule, targets can be anything like an IG, NAT, local networks, etc. we can have many rules but when none of them matches then the packet is forwarded to the default route target ( default is specified using 0.0.0.0/0 as destination)

![Step 3 Image](https://cdn-images-1.medium.com/max/5448/1*AyH4eGQSnrO5Hi4Io3DaAw.jpeg)*Step 3 Image*

Inside a VPC, all instances will generally have two rules in its route table, the first rule is to provide reachability between all instances within its network, In our case of VPC with base range 10.0.0.0/16, then the rule will be
> # (Destination) 10.0.0.0/16* -> *(Target) *Local Network*

and the second rule is to provide internet connectivity using the default route which is an internet gateway.
> # (Destination) 0.0.0.0/0 -> (Target) Internet Gateway

Instances get internet connectivity by routing all packets to the internet gateway excluding the packets which match the chosen VPC CIDR range (10.0.0.0/16).
> **Step 4 (Private IP for app servers)**

Before proceeding further, I want to clarify one more thing, when the user request goes from Load balancer(LB) to any app servers, the response is being sent back to LB in the same connection and in our case, it is always a static “Hello World” message is being sent. The Internet gateway is not used in between LB and app servers.

![Step 4 Image](https://cdn-images-1.medium.com/max/5448/1*80Oj8KA4YH45LATZC7gJ5Q.jpeg)*Step 4 Image*

The Internet gateway is being used to provide internet connectivity to the load balancer (LB) so that the user requests can reach LB and the response can be sent back from LB to users through the internet, so we can remove the public IP from app servers to reduce the attack surface on your infra. Since the public IP of app servers are removed, we can also remove the default route to IG of the app servers route table. (will cover more in upcoming blog 2 about the components which don’t require a public IP)
> **Step 5 (NAT)**

Now comes a new requirement, Instead of sending static “Hello world” message response, you need to send the location of the user accessing the service using their IP address. To get the country location from IP, you are planning to hit some external service like iptocountry.com with IP say 52.1.2.3. 
When we try to hit the service with IP 52.1.2.3 from machine with private IP 10.0.5.5, Since default route is to mapped to internet gateway(IG), IG will get the request and it will forward to 52.1.2.3 server but the problem is source IP, it will be a private IP. when 52.1.2.3 server tries to establish a connection with the private IP, it can’t be done and packets will be eventually dropped, either timed out or some intermediate routers might drop packets on seeing the private IP’s.

NAT (Network address translation) machine solves the above problem, whenever a private machine (machine with private IP) **initiates a new connection **to outside, it needs to go to NAT and NAT will proxy the connection with its own IP. NAT does the job of rewriting the source IP of the packets with its public IP, and the external world will see the IP address of the NAT not the original IP address of the instance. It does the job of providing internet connectivity to instances in private subnets. Now to route the outgoing request to NAT, we need to change default entry in the app route tables to NAT from internet gateway (see the image ‘step 5’).

![Step 5](https://cdn-images-1.medium.com/max/4696/1*RKe45cHMcGXDZRTjDKJkrg.jpeg)*Step 5*

Reiterating the above point one more time, NAT is being used only when the request originates from an instance and tries to hit any external services, outside our network (i.e request to GitHub to download a package, etc), but when a request comes from a Load Balancer and the response is being sent in the same connection, the response will not go through the NAT.

It is recommended and easy to use managed service like NAT gateway, instead of manually hosting the NAT servers. Since it is a managed service, provided by cloud providers, it is highly available and scalable.
> **Step 6 Introducing Subnets**

Typically a region is divided into multiple availability zones (minimum of 3)** **which are completely independent, situated at distinct geographical locations so even if one AZ completely goes down, it should not affect other AZ’s, Each AZ internally maps to one or more connected data center(s). So while designing VPC, we need to divide the primary CIDR(10.0.x.x/16) into multiple subnets where each subnet maps to single AZ (a subnet can’t span multi-AZ). While creating a subnet, we need to give the CIDR range of the subnet and we need to say whether instances booted inside that subnet will have the public IP or not. When we launch instances we have to choose the subnet in which, it needs to be booted.

We can categorize the existing instances into two types, public and private instances.

* Private Instances (instances with private IP) can’t be reached directly from the internet

* Public Instances (instances with public IP) can be reached directly from the internet

To start with, we will create two subnets (one private and one public) in the same AZ and move all instances into the corresponding subnets, like private and public instances will be booted in private and public subnet respectively, It will also give us the advantage to instantly know whether any given IP belongs to application servers or other servers, since the prefix of the IP’s are different for each subnet.

![Step 6 Image](https://cdn-images-1.medium.com/max/5448/1*2MlQFPUaPM7TJHC9WDaapw.jpeg)*Step 6 Image*

In production, a minimum of at least two private and public subnets of different AZ is required for High availability (HA), Will cover move about the best practices in designing of the subnets to have High availability in the next upcoming blog.
> **Step 7 (Centralised route tables)**

If you see the current design(step 6 image), we can very well see a clear pattern, there is only two types of route tables are being used, private route tables used by app servers 1 and 2 in subnet 2, public route table used by LB, and NAT, so cloud providers have given centralised route tables.

Instead of using the machines route table, we add those rules in central route tables and we can name them like private and public route tables and attach it to the corresponding private and public subnets, so now we can control the routing on the subnet level.

![Step 7 Image](https://cdn-images-1.medium.com/max/5736/1*7azDIf1AxXPthz3kBsGUeQ.jpeg)*Step 7 Image*

Later when you decide to have one more type of private server like DB, we can create subnet 3 for DB, you attach that subnet in the same private route table.
The public route table has a route to the Internet gateway and a subnet is called a public subnet when it’s associated with a public route table.
> **Step 8 (Security groups)**

Security groups are virtual centralized IPtables, which can be attached to any instance and used for controlling both Inbound and outbound access for the instances. Inbound security group rules are closed model where by default all inbound access is restricted so we need to add rules to allow access for any instances.

Each rule of a security group contains three parts

* Protocol — like TCP, ICMP, UDP, etc

* Source — generally a IP or range of IPs or another security group can be given.

* Port — the port which needs to be accessible

Let’s say you want to allow users to access our haproxy from a set of restricted IP, then create a security group called haproxy-sec and allow HTTP protocol (HTTP is a TCP protocol), restricted IP address as the source, port 80(HTTP port) and attach the created the security group to the haproxy instance.

![Step 8 Image](https://cdn-images-1.medium.com/max/5896/1*7bDkzgO9Uo6nLBvI-d25fw.jpeg)*Step 8 Image*

Similarly, we need to allow traffic to app instances only from haproxy to do that, we can either specify a haproxy IP directly or its attached security group, **always whitelist traffic between instances using their security groups**, it is a more easy and manageable way so we will create another security group called app-sec and allow TCP as protocol, haproxy-sec as a source and 8080 as a port and attach it to both app instances. Here we are saying allow traffic only from instances which are tagged with haproxy-sec to the port 8080. The benefit of whitelisting using a security-grp name is, let’s say in future you are booting another haproxy instance and it is tagged with a haproxy-sec group then we don’t need to modify any existing rules.

You can also specify a range of IP’s as a source but try to stick with the name wherever possible. (In a security group there is we can also do restrict the outbound traffic).
> **Complete request and response flow**

We looked into all required components for a full-fledged working VPC, Now we will see how the end user request from browser reaches the app server hosted inside a VPC and how the response is being sent. During each hop, there are essentially two things are important
> # i) The next hop to reach a target machine. Source machine route table rules are evaluated to decide it.
> # ii) Whether the traffic is allowed to reach the target machine. Target machine Security group(s) rules are evaluated to decide it.

![Complete request and response flow](https://cdn-images-1.medium.com/max/2000/1*Ls3ZuTmoKroEqvTetWpMnw.jpeg)*Complete request and response flow*

### 1) A User request to Internet Gateway (IG)

When a user with IP address 54.0.0.1 tries to access the service “helloworld.com” which is mapped to the haproxy IP(52.2.3.4) by DNS, it is routed to our Internet Gateway.

### 2) IG to haproxy.

Internet gateway knows the IP (52.2.3.4) belongs to haproxy with private IP 10.0.2.5. It does the NAT and forwards the request to the haproxy.

### 3) Validation at haproxy security group

Before the request reaches haproxy, the request coming from the IP (54.0.0.1) is validated against haproxy security group rules. In *haproxy-sec-grp* we have the rule to allow traffic from only the IP 54.0.0.1. So request is allowed to reach the haproxy. To allow all traffic from anywhere we need to add 0.0.0.0/0 as source IP.

### 4) Public route table local

Haproxy load balances the incoming request among the two app servers, Let’s say it decides to send the request to app server 1 with IP (10.0.1.5). As I mentioned earlier, the route table dictates the next hop of any packet going out. The outgoing packets IP address is compared against the rules of the respective route table, in our case it is the* public route table.* since haproxy belongs to the public subnet and it has a public route table attached.

### 5) Haproxy to app server 1

The first rule of the *public route table* is if the target IP address if within the range 10.0.x.x (10.0.0.0/16) then packets need to be routed to the local network. Since the destination host(app server 1) presents within the local network, the request will be correctly routed to the app server 1.

### 6) Validation at app-sec-grp security group

Before the request reaches the app server 1, the request is validated against the rules of app-sec-grp, the only rule in *app-sec-grp* states that, allow TCP traffic to the port 80 from all instances which has the security group called *haproxy-sec-grp *attached*. *In our case the haproxy instance has the *haproxy-sec-grp *security group attached, so the request is allowed to reach the app server 1.

### 7) Private route table

After the request reaches app servers, it needs to hit the iptocountry.com ( 52.1.2.3 IP address of iptocountry.com) to get the country location for the user request. Similar to step 4, a private route table is referred to understand the next hop. since destination 52.1.2.3 is outside the local network, the first rule is not matched. The default rule will be matched and packets are routed to NAT.

### 8) App server 1 to NAT Gateway

The request is routed to NAT gateway. It is a managed service, we don’t need to manually configure any security group rules and

### 9) Public route table

Now NAT gateway needs to connect to remote service to get the response. NAT gateway is placed inside a public subnet, so *the public route table* is being used to decide the next hop and in our case, since the destination 52.1.2.3 is outside the local network, so the default route to Internet gateway is chosen.

### 10) NAT to Internet Gateway (IG)

The request is forwarded to the Internet gateway.

### 11) Internet Gateway to iptocountry.com

From internet gateway routers, the request will be routed to the iptocountry.com service. When the request reaches one of the servers in the iptocountry.com service, it will see NAT’s Public IP (53.1.2.3) as the source IP.

### 12) Responses flow

Similar to request, response packets are routed based on the route table rules. The response is sent in the below way from iptocountry.com to user iptocountry.com -> IG -> NAT -> APP -1 -> HAPROXY -> IG -> USER.

The current design is not highly available and scalable, imagine the AZ which we are using goes down, our service will completely go down. It is always recommended to have a minimum of two resources of any service in a different AZ. I will cover more about best practices in setting up an infrastructure that needs to be followed to be highly available and secure in upcoming blog 2. Also, I will share an AWS terraform template and steps to run it, which helps to set up the initial VPC and its components in 5–10 mins. The created VPC will have a lot of things in place, like subnets and security groups for components like load balancers, app servers, and DB, etc.
