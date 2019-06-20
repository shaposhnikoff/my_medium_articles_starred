
# Introduction to VPC Peering — Part 2

This is the continuation of Part 1 of the series https://medium.com/p/c385d2ff8138 and in this, part we will actually start creating VPC Peering using AWS Console

Go to your VPC Dashboard and look for Peering Connections → Create Peering Connection

![](https://cdn-images-1.medium.com/max/4340/1*hQBLaj1OzIXCN7MJSdY_Zw.png)

* Give some meaningful name to Peering connection name tag(eg: vpc-peering-test)

* Select Requester VPC

* As mentioned in the first part of the series, we can create VPC Peering between different account as well as between different region

* Select Acceptor VPC(As you can see Acceptor VPC has complete different CIDR region, as overlapping CIDR is not supported)

Even I am creating VPC Peering between the same account, I still need to accept peering connection

![](https://cdn-images-1.medium.com/max/4984/1*4aNbFEHpWhowZ7pgmZwzCg.png)

![](https://cdn-images-1.medium.com/max/5088/1*qz1jD7vFif0gaopocprWgw.png)

* The final step is to update the individual VPC route table with the peering connection

![](https://cdn-images-1.medium.com/max/4288/1*Lwug62z11sHcVaFWd5CMlw.png)

![](https://cdn-images-1.medium.com/max/4376/1*X7qzMT_qjEXvZUnce5t0EA.png)

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/kMLG4IrzyV4" frameborder="0" allowfullscreen></iframe></center>
