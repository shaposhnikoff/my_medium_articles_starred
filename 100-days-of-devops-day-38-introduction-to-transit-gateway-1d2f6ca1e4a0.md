
# 100 Days of DevOps — Day 38-Introduction to Transit Gateway

Welcome to Day 38 of 100 Days of DevOps, Focus for today Transit Gateway.

*During ReInvent 2018, AWS released a bunch of Products*
[**2018 re:Invent Product Announcements**
*AWS Step Functions, a fully-managed workflow service, is now integrated with Amazon SageMaker and AWS Glue, making it…*aws.amazon.com](https://aws.amazon.com/new/reinvent/)

*But if you ask me one product which stands out among these is Transit Gateway.*
> *What is Transit Gateway?*

*AWS Transit Gateway is a service that enables customers to connect their Amazon Virtual Private Clouds (VPCs) and their on-premises networks to a single gateway*

![](https://cdn-images-1.medium.com/max/2456/1*9YyVxC2jlhphb49CS2w7Mg.png)
> *Features of Transit Gateway*

* *Connect thousands of Amazon Virtual Private Clouds (VPCs) and on-premises networks using a single gateway*

* *Hub and Spoke Network Topology*

* *Scales up to **5000 VPCs***

* *Spread traffic over many VPN connections (Scale horizontally eg: Now two VPN connection combined together give 2.5GBPS(1.25GBPS + 1.25GBPS)*

* *Max throughput AWS tested so far is 50GBPS*

* *Direct Connect is still not supported(In AWS 2019 Roadmap)*

* *Under the hood, to make this happen AWS is using a technology called Hyperplane([https://twitter.com/awsreinvent/status/935740155499040768?lang=en](https://twitter.com/awsreinvent/status/935740155499040768?lang=en))*

![](https://cdn-images-1.medium.com/max/3616/1*P6aIMkNMxuuhxiWW9JRh1g.png)

* *Transit gateway each route table support 10000 routes(in case of VPC default route table limit is still 100)*

* *Difference between Transit VPC vs Transit gateway*

* *Transit Gateway is available under VPC console*

![](https://cdn-images-1.medium.com/max/3628/1*1N14hBMIKCOhMmYpF9J5hg.png)
> ***Step1: Build TGW***

    *Go to [https://us-west-2.console.aws.amazon.com/vpc](https://us-west-2.console.aws.amazon.com/vpc) → Transit Gateways → Transit Gateways --> Create Transit Gateway*

![](https://cdn-images-1.medium.com/max/2000/1*TIceqS244v1rIJ_fVCQseg.png)

![](https://cdn-images-1.medium.com/max/5760/1*EmSsPGXxJ3NACAaZcfLqQQ.png)

    ** **Name tag and Description:** Give some meaningful name to your Transit Gateway and Description
    * **Amazon side ASN:** Autonomous System Number (ASN) of your Transit Gateway. You can use an  existing ASN assigned to your network. If you don't have one, you can  use a private ASN in the 64512-65534 or 4200000000-4294967294 range.
    * **DNS Support:** Enable Domain Name System resolution for VPCs attached to this Transit Gateway(If you have multiple VPC, this will enable hostname resolution between two VPC)
    ***VPN ECMP support:** Equal-cost multi-path routing for VPN Connections that are attached to this Transit Gateway.Equal Cost Multipath (ECMP) routing support between VPN connections. If connections advertise the same CIDRs, the traffic is distributed equally between them.
    * **Default route table association:** Automatically associate Transit Gateway attachments with this Transit Gateway's default route table.
    * **Default route table propagation:** Automatically propagate Transit Gateway attachments with this Transit Gateway's default route table
    * **Auto accept shared attachments:** Automatically accept cross account attachments that are attached to this Transit Gateway.In case if you are planning to spread your TGW across multiple account.*
> ***Step2: Attach your VPC***

    *Go to Transit Gateways --> Transit Gateway Attachments --> Create Transit Gateway Attachment*

![](https://cdn-images-1.medium.com/max/5760/1*0UwBjmMq1xo4fa_lz8EWnw.png)

    ** Select your TGW created in Step1
    * Give your VPC attachment some name
    * Enable DNS support
    * Select your first VPC *

* *Perform the same step for VPC2*

***NOTE:** When you **attach** a VPC or create a VPN connection on a **transit gateway**, the **attachment** is associated with the default route table of the **transit gateway**.*
> ***Step3: Update Route Table***

* *If you click on the Transit Gateway Route Table, you will see we have the patch from Transit Gateway to our VPC*

![](https://cdn-images-1.medium.com/max/5168/1*_-DjV56-gqkfPPU23dx93w.png)

* *We need a return path(i.e from our VPC to TGW), VPC1 route table needs to be updated to point to TGW to route to the second VPC and vice-versa(i.e 10.0.0.0/16 to tgw on the second VPC)*

![](https://cdn-images-1.medium.com/max/3688/1*SSKGK4AaK-LqYGsBwvFzeg.png)
> *Some Key Terms*

* ***associations** — Each attachment is associated with exactly one route table. Each route table can be associated with zero to many attachments.*

* ***route propagation** — A VPC or VPN connection can dynamically propagate routes to a transit gateway route table.*

***Step4***

* *Try to create Instance on VPC1 and VPC2*

* *ssh to the instance on VPC1(using its Public IP)*

* *Also, copy your Public ssh key to an instance in VPC1*

* *Now try to ssh to an instance in VPC2, using its Private IP*

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
