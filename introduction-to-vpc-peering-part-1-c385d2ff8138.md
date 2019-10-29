
# Introduction to VPC Peering— Part 1

Welcome to the world of Virtual Peering. Similar to the earlier series related to VPC https://medium.com/p/cbca3b189329 this is the three-part series

* Part 1: *I am just going to talk about all VPC Peering fundamentals [*https://medium.com/p/c385d2ff8138](https://medium.com/p/c385d2ff8138)

* *Part2: Let’s build VPC Peering using AWS Console [*https://medium.com/p/12ae81ab0b88/](https://medium.com/p/12ae81ab0b88/)

* *Part3: We will re-create the same VPC Peering using Terraform*

What is VPC Peering?

* Let say two VPC want to communicate with each other or share service between them, the best way to do that with the help of VPC Peering

* VPC Peering connection is a networking connection between two VPCs that allow us to route traffic between them using private IPv4 addresses.

* Instances in either VPC can communicate with each other as if they are part of the same network

* AWS uses the existing infrastructure of a VPC to create a VPC peering connection

* It’s neither a gateway nor a VPN connection and doesn’t rely on a separate piece of physical hardware

* There is no single point of failure or a bandwidth bottleneck i.e bandwidth between instances in peered VPC is no different than bandwidth between instances in the same VPC.

![](https://cdn-images-1.medium.com/max/2760/1*4CSwmya2dARocLABjiwq8Q.jpeg)

* VPC Peering doesn’t support transitive peering i.e VPC1 can talk to VPC 2, VPC 2 can talk to VPC3 but VPC1 can’t talk to VPC3. This is because of the security reason so if VPC1 want to communicate with VPC3 we need to establish one more peering connection between VPC1 and VPC3.

* Once VPC Peering is established instance in two VPC can communicate with each other using Private IP(no need to communicate via Internet Gateway)

* Inter-region VPC is supported

* VPC Peering is even supported between two different accounts

* Make sure there is no over-lapping IP between two VPC’s

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/Ee4YYA3fcA8" frameborder="0" allowfullscreen></iframe></center>
