
# Introduction to VPC — Part 1

Welcome to the world of Virtual Private Cloud(VPC). This is a three-part series:

* ***Part1: I am just going to talk about all VPC fundamentals [**https://medium.com/p/cbca3b189329](https://medium.com/p/cbca3b189329)*

* ***Part2: Let’s build VPC using AWS Console [**https://medium.com/p/b0cb4ab41e8f](https://medium.com/p/b0cb4ab41e8f/edit)*

* ***Part3: Create one more VPC but this time using Terraform [**https://medium.com/p/1a7fb9a336e9](https://medium.com/p/1a7fb9a336e9/edit)*

*Without going to all the nitty-gritty details of VPC, first, let’s try to understand VPC in the simplest term. Before the cloud era, we use to have datacenters where we deploy all of our infrastructures.*

![](https://cdn-images-1.medium.com/max/2000/0*riYnQrOA97UKOOf9)

*You can think of VPC as your datacentre in a cloud but rather than spending months or weeks to set up that datacenter it’s now just a matter of minutes(API calls). It’s the place where you define your network which closely resembles your own traditional data centers with the benefits of using the scalable infrastructure provided by AWS.*

*Let pay special emphasis to **P in VPC** which means Private*

* *As it’s private, it’s entirely under your control.*

* *You are going to choose your Network/CIDR range.*

* *You get the control what subnet can and what can’t it talk to.*
> ***Default VPC***

* *When we first set up an account, AWS provided/created it by default. This VPC has a default subnet in each Availability Zone.*

* *We don’t have much control over default VPC eg: default VPC includes an internet gateway and has a public subnet, let say my requirement is I want to deploy a database server, and I don’t wish to that database server to talk to the internet, in those cases we want to create custom VPC or non-default VPC.*

***AWS VPC Limits***

![](https://cdn-images-1.medium.com/max/3232/1*D43zNbdwJ4lycOTGZoGlvg.png)

***Reference:***
[**Amazon VPC Limits — Amazon Virtual Private Cloud**
*Request increases to the following default Amazon VPC component limits as needed.*docs.aws.amazon.com](https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html)

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/2kWpSjrln-I" frameborder="0" allowfullscreen></iframe></center>
