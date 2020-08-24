
# Simplifying your AWS network architecture

Photo by NASA on Unsplash

Anyone that has been involved with a cloud environment knows that over time, it tends to grow more complex.

A company typically starts with a few proof-of-concepts in a single AWS account. Fast-forward a couple of years, and the company has adopted a new organisational model with autonomous teams (in order to be able to profit from agility and flexibility provided by public cloud). Each of these teams have their own AWS account, resulting fairly easily in having dozens of accounts.

From a networking perspective, this can easily result in hundreds of VPCs with redundant VPN connectivity to on-premise and numerous peering connections between these VPCs. The example below illustrates this network architecture with just 5 VPCs (Shared services VPC + 4 team VPCs).

![](https://cdn-images-1.medium.com/max/2000/1*m87C0obi5RnsfjjKARutEA.png)

When looking at the routing tables (see image below), it’s easy to see that there is a lot of repetition of configuration. When a new VPC is created, the configuration of all VPCs needs to be updated, which induces a lot of work and overhead, especially when the environment is growing fast. It also means teams spent time not working on their core mission.

![](https://cdn-images-1.medium.com/max/2000/1*qE8RQt1C415rYsaeTmgNUg.png)

At this point various challenges become visible and potentially create friction.

1. In order to enable communication between APIs, services or applications of different teams a mesh of VPC peering connections must be set up. This can either be done up front (as a new VPC is created) or ad hoc (as communication between VPCs is needed). Either case requires teams to deviate from their core mission and creates planning dependencies between teams.

1. Shared or centralised services (i.e. monitoring) running in a shared service VPCs need to be accessible from all other VPCs. How to ensure configuration of all other VPCs is updated consistently?

1. Lack of a general overview of the deployed network architecture makes troubleshooting more difficult.

1. Cost of networking components (NAT Gateways, VPN connections) starts to become significant (roughly $2,100/VPC/year — 3 NAT Gw + redundant VPN connection).

1. As environment grows further, service limits come into play (nr of routes in route table, active VPC peering connections per VPC)

## How about Transit VPC…

Via the AWS Marketplace [Transit VPC solutions](https://aws.amazon.com/marketplace/solutions/infrastructure-software/transit-VPC) have been available for some time. These require running multiple virtual routers on EC2 instances. While these solutions clearly reduce complexity of the architecture, some of the above challenges aren’t addressed (cost and service limits). On top the solution has to be managed. As a company you still have to do the heavy lifting yourself.

## Enter AWS Transit Gateway

At re:Invent 2018, AWS released the [Transit Gateway service](https://aws.amazon.com/blogs/aws/new-use-an-aws-transit-gateway-to-simplify-your-network-architecture/) that provides an elegant and scalable solution that addresses the challenges of the original architecture.

Transit Gateway is a service that allows deploying a highly scalable hub-and-spoke network architecture in AWS and extending this to companies on-premise datacenter. In addition the service also supports defining isolated routing domains (i.e. in order to prevent communication between production and non-production resources).

Let’s look at an example architecture that allows communication between all VPCs and how this removes the challenges stated above. The coloured arrows illustrate some of the possible traffic flows.

![](https://cdn-images-1.medium.com/max/2000/1*gMaLIalOkHkqdiDSiz5olQ.png)

In this example, a central Transit Gateway is deployed to which all VPCs and VPN connections from on-premise are connected. All VPC attachments are associated with the same routing table and propagation of the CIDR block of each VPC is configured in the route table. An extra VPC (egress) is added through which all outbound traffic of the private subnets is routed (green traffic flow).

Routing tables in each VPC are much simpler and are no longer subject to change whenever a new VPC is created.

* For the private subnets: the Transit Gateway attachment is configured as default gateway.

* For the public subnets: the Internet Gateway remains default gateway and the Transit Gateway attachment is configured as gateway for the overall CIDR block of the public cloud environment and for the on-premise data center.

* The only exceptions are the private subnets of the egress VPCs: their default gateway should be the NAT Gateways of the egress VPC instead of the Transit Gateway attachment.

![](https://cdn-images-1.medium.com/max/2000/1*MID05-9_X_SRPKd19nK75A.png)
> Note: The above routing tables don’t include routes for S3 and DynamoDB VPC endpoints. VPC endpoints provide a private connection from your VPC to an AWS service without the traffic passing over public Internet via a NAT Gateway, Internet Gateway or other Gateway. The use of VPC endpoints holds a few benefits: (i) improved security, (ii) reduced data transfer cost and (iii) improved performance.

When a new VPC is created, the procedure is actually simple: (i) create the Transit Gateway attachment, (ii) associate the attachment the correct routing table and (iii) configure the route propagation in relevant routing tables. As of that moment, the new VPC is reachable from within the other VPCs.

Revisiting the above challenges:

1. As new VPCs are added, the configuration of existing VPCs does not have to be changed. Transit Gateway simplifies management and avoid the dependencies linked to the old architecture.

1. Similar to 1st challenge. As the VPC for the shared services is attached to the Transit Gateway and routing tables are associated and updated, it will automatically become reachable from all VPCs.

1. Conceptually a hub-and-spoke architecture is easier to comprehend. The configuration of the Transit Gateway and its routing tables provides good insight into inter-VPC connectivity.

1. The total cost of network components for the original architecture with 5 VPCs is $10,687/year. The cost for the new architecture is $4,765/year.
If the environment grows to 50 VPCs, the difference grows to $106,872/year vs. $24,475/year. The use of Transit Gateway greatly reduces the cost of the network components.

1. The service limits of Transit Gateway are far higher (i.e. nr of active peering connections (50) vs nr of Transit Gateway attachments (5000)).
> **Important note**: at this moment Transit Gateway still lacks support to reference security groups in other VPCs. If this is essential for your environment, it’s probably too soon to adopt Transit Gateway.

## 3..2..1.. Action!

How do you migrate from the old to the new networking?

As a disclaimer, changing the architecture of your cloud network in such a fundamental way should be carried out with great care.

In the environment where I performed this migration, I managed the entire environment that consisted of production, non-production and shared services VPCs. I didn’t have VPN to on-premise connectivity though. The next few paragraphs contain a high-level overview of the steps I took.

**Phase I**: Central components and testing.

1. First step was to set up the central components: Transit Gateway and egress VPC. At the Transit Gateway, 3 different routing tables were configured: prod, non-prod and shared. A static route was added in all 3 routing tables to forward any traffic not destined for any of the VPCs (destination 0.0.0.0/0) to the egress VPC.

1. I set up a few test VPCs to simulate and validate the configuration works as expected. I checked network flows between VPCs and between VPC and Internet.

![](https://cdn-images-1.medium.com/max/2000/1*L9kxTTjGCqSxtLMnmak7bQ.png)

**Phase II**: Non-production.

1. Next step was attaching the non-production VPCs to the Transit Gateway and associating them with the non-production routing table. VPC routing tables weren’t updated yet.

1. Then I checked security groups associated with public resources (ELB, EC2 instance, …) in the non-production VPCs for rules that allow traffic originating from the public IPs of NAT Gateways (in any of the other VPCs). Where such rules existed, I added the public IPs of the NAT Gateways in the egress VPC. This ensured that as outgoing traffic shifted to the NAT Gateways in the egress VPC, the traffic wasn’t being blocked.

1. Updating route tables in all non-production VPCs.
In the route tables for public subnets, I added routes to forward traffic for overall public cloud network range (10.0.0.0/16) to the Transit Gateway and removed the routes to other non-production VPCs. Outgoing traffic to other non-production VPCs shifted to Transit Gateway. Return traffic from VPCs of which the route tables were not yet updated, still passed via the VPC peering connection, but that’s not a problem and is only temporarily.
In the route tables for private subnets, I changed the default route to forward traffic to the Transit Gateway. This shifted outgoing traffic to use the egress VPC.

1. I validated if the NAT Gateways were still forwarding traffic (in the monitoring tab of the NAT Gateways in the VPC Console). As this as no longer the case, the NAT Gateways were removed.

1. In the VPC Peering connections I checked if the peering connections were no longer in use and removed them.

At this point in the migration, the network architecture is similar to the schema below.

![](https://cdn-images-1.medium.com/max/2000/1*nRZmLIveO0h9jCL1MFZb4Q.png)

**Phase III**: Production VPCs

I performed similar steps to phase 2, but this time for the production VPCs.

![](https://cdn-images-1.medium.com/max/2000/1*JV6Ey-lcjCRiy4vHX4vZQA.png)

**Phase IV**: Shared VPC

This is the final phase of the migration. I attached and rerouted traffic for the shared VPCs. Again this involved steps similar to phase 2, but this time the routes in all VPCs needed to be updated, after which remaining VPC peering connections could be removed.

![](https://cdn-images-1.medium.com/max/2000/1*DxKLR2kFsb5olUj9dA-dzQ.png)

After completing the migration, the resulting architecture offers several advantages:

* Simplified network architecture

* Reduced management overhead

* Centralized control of network flow policies

* Reduced cost of network components.

## Conclusion

This article briefly illustrates how a network architecture in a public cloud environment organically will become more complex as it grows, which leads to a number of challenges, namely:

* Increased management and overhead.

* Lack of overview.

* Increased cost.

AWS Transit Gateway offers an elegant solution to create a scalable hub-and-spoke network architecture that is far simpler to comprehend and manage. The network architecture described mitigates all of the challenges induced by the old architecture.

— J
