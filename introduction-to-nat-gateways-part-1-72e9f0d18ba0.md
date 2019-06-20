
# Introduction to NAT Gateways — Part 1

NAT gateway enables instance in Private Subnet to connect to the internet or other AWS services but prevent the internet from initiating a connection with those instances.

***How NAT works***

* *NAT device has an Elastic IP address and is connected to the Internet through an internet gateway.*

* *When we connect an instance in a private subnet through the NAT device, which routes traffic from the instance to the internet gateway and routes any response to the instance*

* *NAT maps multiple private IPv4 addresses to a single public IPv4 address.*

*NAT gateway doesn’t support IPv6 traffic for that you need to use Egress only gateway.*
> ***NOTE: **IPv6 traffic is separate from IPv4 traffic, route table must include separate routes for IPv6 traffic.*

***More info***
[**Comparison of NAT Instances and NAT Gateways - Amazon Virtual Private Cloud**
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

<iframe src="https://medium.com/media/0e8f109874455e7f33dccf2bff932c55" frameborder=0></iframe>
