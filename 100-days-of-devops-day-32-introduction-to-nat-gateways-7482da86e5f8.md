
# 100 Days of DevOps — Day 32-Introduction to NAT Gateways

Welcome to Day 32 of 100 Days of DevOps, Focus for today is NAT Gateway
> *What is NAT Gateway*

*NAT gateway enables instance in Private Subnet to connect to the internet or other AWS services but prevent the internet from initiating a connection with those instances.*
> ***How NAT works***

* *NAT device has an Elastic IP address and is connected to the Internet through an internet gateway.*

* *When we connect an instance in a private subnet through the NAT device, which routes traffic from the instance to the internet gateway and routes any response to the instance*

* *NAT maps multiple private IPv4 addresses to a single public IPv4 address.*

*NAT gateway doesn’t support IPv6 traffic for that you need to use Egress only gateway.*
> ***NOTE: **IPv6 traffic is separate from IPv4 traffic, route table must include separate routes for IPv6 traffic.*

***More info***
[**Comparison of NAT Instances and NAT Gateways — Amazon Virtual Private Cloud**
*Compare NAT gateways and NAT instances.*docs.aws.amazon.com](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-comparison.html)
> *To create a NAT gateway*

***Go to VPC Dashboard → NAT Gateways → Create NAT gateways***

![](https://cdn-images-1.medium.com/max/4640/1*21vH1k4TtqCMj50JntnkNw.png)

* *Make sure you select the Public Subnet in your custom VPC*

* *For NAT gateway to work, it needs Elastic IP*

***NOTE: NAT Gateway creation will take 10–15 min***

* *Once the NAT gateway is available, add it to your default Route table*

![](https://cdn-images-1.medium.com/max/4180/1*awVpUBQ5zqID8ePTW0M7oQ.png)

***The advantage of NAT Gateways***

* *NAT gateway is highly available but we need it per availability zone.*

* *Can scale up to 45Gbps*

* *Managed by AWS*

*Most of the code is the same as VPC Code*
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/tree/master/two-tier-environment/vpc_networking)

* *Some additions*

<iframe src="https://medium.com/media/de305a972bdd26a03abae7411753f9e4" frameborder=0></iframe>

*As well as we need to tell associate NAT gateway to Private Route Table*

<iframe src="https://medium.com/media/42fa1a146e7ef13d41c4c7fe406dcc59" frameborder=0></iframe>

*Complete Teraform Code*
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/tree/master/nat_gateway)[\](https://github.com/100daysofdevops/100daysofdevops/tree/master/nat_gateway)

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
