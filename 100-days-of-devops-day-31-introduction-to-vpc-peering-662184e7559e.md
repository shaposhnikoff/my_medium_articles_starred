
# 100 Days of DevOps — Day 31-Introduction to VPC Peering

Welcome to Day 31 of 100 Days of DevOps, Focus for today is VPC Peering
> *What is VPC Peering?*

* *Let say two VPC want to communicate with each other or share service between them, the best way to do that with the help of VPC Peering*

* *VPC Peering connection is a networking connection between two VPCs that allow us to route traffic between them using private IPv4 addresses.*

* *Instances in either VPC can communicate with each other as if they are part of the same network*

* *AWS uses the existing infrastructure of a VPC to create a VPC peering connection*

* *It’s neither a gateway nor a VPN connection and doesn’t rely on a separate piece of physical hardware*

* *There is no single point of failure or a bandwidth bottleneck i.e bandwidth between instances in peered VPC is no different than bandwidth between instances in the same VPC.*

![](https://cdn-images-1.medium.com/max/2760/1*4CSwmya2dARocLABjiwq8Q.jpeg)

* *VPC Peering doesn’t support transitive peering i.e VPC1 can talk to VPC 2, VPC 2 can talk to VPC3 but VPC1 can’t talk to VPC3. This is because of the security reason so if VPC1 want to communicate with VPC3 we need to establish one more peering connection between VPC1 and VPC3.*

* *Once VPC Peering is established instance in two VPC can communicate with each other using Private IP(no need to communicate via Internet Gateway)*

* *Inter-region VPC is supported*

* *VPC Peering is even supported between two different accounts*

* *Make sure there is no over-lapping IP between two VPC’s*

*Go to your VPC Dashboard and look for Peering Connections → Create Peering Connection*

![](https://cdn-images-1.medium.com/max/4340/1*hQBLaj1OzIXCN7MJSdY_Zw.png)

* *Give some meaningful name to Peering connection name tag(eg: vpc-peering-test)*

* *Select Requester VPC*

* *As mentioned in the first part of the series, we can create VPC Peering between different account as well as between different region*

* *Select Acceptor VPC(As you can see Acceptor VPC has complete different CIDR region, as overlapping CIDR is not supported)*

*Even I am creating VPC Peering between the same account, I still need to accept peering connection*

![](https://cdn-images-1.medium.com/max/4984/1*4aNbFEHpWhowZ7pgmZwzCg.png)

![](https://cdn-images-1.medium.com/max/5088/1*qz1jD7vFif0gaopocprWgw.png)

* *The final step is to update the individual VPC route table with the peering connection*

![](https://cdn-images-1.medium.com/max/4288/1*Lwug62z11sHcVaFWd5CMlw.png)

![](https://cdn-images-1.medium.com/max/4376/1*X7qzMT_qjEXvZUnce5t0EA.png)

***Terraform Code***

<iframe src="https://medium.com/media/1535906f024bc07eba415e2941a7e97f" frameborder=0></iframe>

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
