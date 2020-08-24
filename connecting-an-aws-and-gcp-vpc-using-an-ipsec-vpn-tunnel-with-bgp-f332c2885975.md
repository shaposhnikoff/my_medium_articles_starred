Unknown markup type 10 { type: [33m10[39m, start: [33m153[39m, end: [33m172[39m }

# Connecting an AWS and GCP VPC using an IPSec VPN Tunnel with BGP

The Peek booking platform processes thousands of tours, activities, and rentals each day. Behind the scenes, the DevOps team here has always been early adopters of containerizing our applications using Docker, and deploying these container using Kubernetes, and Helm. In addition to containerization, we have strived to move toward applying Infrastructure as Code (IaC) and leveraging tools such as Terraform across our fleet of services in both AWS and GCP.

As part of Peek’s journey this year, our focus turned to creating a multi-cloud environment using AWS and GCP. The goal was to have our private services and databases exist in either cloud provider VPC, and still be able to easily communicate with each other. To achieve a production ready infrastructure, we needed a solution that was also redundant. Our solution came down to setting up a high availability IPSec VPN Tunnel.

This guide will illustrate how to assemble the VPN tunnels between both cloud providers. As a quick overview, on both sides we’ll have a VPC with a public subnet and a private subnet. The public subnet we will use to create a bastion to access a private host in the private subnet. The private hosts in AWS and GCP will be able to communicate with each other after the VPN tunnels has been established. In order for the VPN to be production ready, we will need to setup a High Availability (HA) Cloud VPN on the GCP side and two Site-to-Site VPN Connections on the AWS side. This will form four total VPN Tunnels. Each VPN tunnel will also use dynamic BGP routing to determine the CIDR ranges from each VPC.

Here is a diagram of what the resulting network configuration will look like:

![](https://cdn-images-1.medium.com/max/3220/1*rglKDhn96Ubt4CQii4ppCA.png)

## Prerequisites

For this tutorial you will need admin access to an AWS account as well as admin access to a GCP project. A VPC network in each cloud provider with CIDR ranges that do not overlap. Each network should have a private subnet, and a NAT in order to enable egress on private instances. The following configuration values will be used:

**AWS**

* A VPC with a public and private subnet in us-east-1, making sure the CIDR range is not conflicting on the GCP side.

* VPC with a CIDR range of 10.0.0.0/16

* A public subnet with range of 10.0.0.0/24

* A private subnet with range of 10.0.0.1/24

* An instance in the private subnet to test the VPN Tunnel with ICMP and port 22 opened.

**GCP**

* A VPC with a public and private subnet in us-east4, making sure the CIDR range is not conflicting with the AWS side.

* VPC with a CIDR range of 10.10.0.0/16

* A public subnet with range of 10.10.0.0/24

* A private subnet with range of 10.10.0.1/24

* An instance in the private subnet to test the VPN Tunnel with ICMP and port 22 opened.

## Overview

GCP has two types of VPN Gateways you can create, classic and High Availability(HA). We will be creating the HA version of the VPN gateway which will allow us to achieve 99.99% SLA. In addition to this gateway, we must create 4 tunnels between GCP and AWS. On the AWS side, we will need to create two Customer Gateways and two Site-To-Site VPN connections.

## Steps

We will be bouncing between GCP and AWS, so sections for each provider are carefully demarcated “GCP Side” or “AWS Side”

**GCP Side**

1. Go to your GCP project, in the left nav bar, *go down to Hybrid Connectivity, select “Cloud Routers”. Click “Create Router”*

![](https://cdn-images-1.medium.com/max/2000/1*gkSrS5EKTN0Ty7CBbbB93g.png)

![](https://cdn-images-1.medium.com/max/2000/1*vMtsK_uuKr29me5KQzWhew.png)

2. In “Create a Cloud Router”, fill in the following. *Click “Create”*

* Name: gcp-sandbox-to-aws-sandbox-cloud-router

* Description: (blank or anything you want)

* Network: gcp-sandbox

* Region: us-east4 (Northern Virginia)

* Google ASN: 64520 (You can use any private ASN (64512–65534). Make sure it doesn’t conflict with the ASN in the AWS side later)
**Note: 4200000000–4294967294 is not allowed in AWS so don’t use this range.*

* Select “Advertise all subnets visible to the Cloud Router (Default)”

![](https://cdn-images-1.medium.com/max/2000/1*oiadYeVo-2XgLuRAPpoL1g.png)

3. Now in *Hybrid Connectivity, select VPN.*

![](https://cdn-images-1.medium.com/max/2000/1*Ey7UlkmlOj9jtsLWz-FYrQ.png)

4. *Select “Create VPN Connection” or “VPN Setup Wizard”, select “High-availability (HA) VPN”. Click Continue.*

![](https://cdn-images-1.medium.com/max/2000/1*SF8593pWDGaUOaBBxd7QvQ.png)

5. Create a VPN Gateway. Fill in the following. *Click “Create & Continue”.*

* Name: gcp-sandbox-vpn-gateway

* VPC Network: gcp-sandbox (VPC you created in GCP)

* Region: us-east4

![](https://cdn-images-1.medium.com/max/2496/1*gJZErJ8AE8odnNqXTq2Ptg.png)

6. The following screen will appear. Copy the Interface 0 and 1 IP address. We will need this on the AWS end. Leave it on this screen for now while we switch to AWS.

![](https://cdn-images-1.medium.com/max/2524/1*jKKqwNOFT-WmiMMZ0N9ySg.png)

**AWS Side**

7. In your AWS account, go to *VPC -> Virtual Private Network (VPN)*, select *“Customer Gateway”. Click “Create Customer Gateway”*.

![](https://cdn-images-1.medium.com/max/2000/1*cDaAzWNOuhHdV0eiMPh7Pg.png)

![](https://cdn-images-1.medium.com/max/2000/1*TfXCjIaOq7m9oOfPBzcmXg.png)

8. Fill in the following. *Click “Create Customer Gateway”*

* Name: aws-sandbox-to-gcp-sandbox-cg1

* Routing: Dynamic

* BGP ASN: 65420 (The ASN you used to create the GCP Cloud Router)

* IP Address: 35.242.61.73 (IP from Interface Tunnel 0 in GCP VPN Gateway)

* Certificate ARN: blank

* Device: blank

![](https://cdn-images-1.medium.com/max/5228/1*wN6ZhLzjPELHe8utyMjSEg.png)

9. Repeat step 8 and create a second Customer Gateway using the Tunnel 1 IP from the GCP VPN Gateway. Fill in the following.

* Name: aws-sandbox-to-gcp-sandbox-cg2

* Routing: Dynamic

* BGP ASN: 65420 (The ASN you used to create the GCP Cloud Router)

* IP Address: 35.220.60.77 (IP from Interface Tunnel 1 in GCP VPN Gateway)

* Certificate ARN: blank

* Device: blank

![](https://cdn-images-1.medium.com/max/5272/1*hCiROqEUEMfgQSm3lESRrw.png)

10. Now in *“Virtual Private Network (VPN)”, select “Virtual Private Gateway”. Click “Create Virtual Private Gateway”.*

![](https://cdn-images-1.medium.com/max/2000/1*cz6FeUA2FblDzv4Prt4vXQ.png)

![](https://cdn-images-1.medium.com/max/2000/1*zg9YyIxlFufNGpTNxdNdfw.png)

11. Fill in the following. *Click “Create Virtual Private Gateway”*

* Name tag: aws-sandbox-to-gcp-sandbox-vpg

* ASN: Custom ASN: 64512 (Any ASN you want. This will be used on the GCP side)

![](https://cdn-images-1.medium.com/max/5316/1*iJbEE2kSdq8cMMAyAtHtRQ.png)

12. After the creation, look in the list of Virtual Private Gateways, select the gateway you just created (aws-sandbox-to-gcp-sandbox-vpg). *Right click on the gateway and “Attach to VPC”.*

![](https://cdn-images-1.medium.com/max/2000/1*Jas8uUxONP7YQ5Dq54COcA.png)

13. In the following screen, select the VPC you want to connect with GCP (AWS Sandbox). *Click “Yes, Attach”.* The state will show “attached”

**Note: As of 7/22/2020, the [documentation from GCP](https://cloud.google.com/network-connectivity/docs/vpn/how-to/creating-ha-vpn) states to “Create two AWS Virtual Private Gateways”. This is not correct, since you can only attach one Virtual Private Gateway to the VPC. If you attempt to attach a second Virtual Private Gateway, it will throw an error*

![](https://cdn-images-1.medium.com/max/5164/1*8jXC834MlM2xzNagZNtBGA.png)

![](https://cdn-images-1.medium.com/max/5152/1*G0RufMvZn4cceG2AIytcIQ.png)

14. On the left nav bar, *go to Virtual Private Network (VPN) again, and select “Site-to-Site VPN Connections”. Click on “Create VPN Connection”*

![](https://cdn-images-1.medium.com/max/2000/1*kh-Lgcm6MdVzvs3d4XeL2A.png)

![](https://cdn-images-1.medium.com/max/2000/1*BDKYwOGqoYc7vtBz0c7Qlg.png)

15. In “Create VPN Connection”, fill in the following. **Do not click “Create VPN Connection” yet. **We need to change the Tunnel Options which is detailed in the next step.

* Name tag: aws-sandbox-to-gcp-sandbox-vpn1

* Target Gateway Type: Virtual Private Gateway

* Virtual Private Gateway: vgw-00a6862ef16d8404e (name tag: aws-sandbox-to-gcp-sandbox-vpg). *Select the Virtual Private Gateway you just created.*

* Customer Gateway: Existing

* Customer Gateway: cgw-05ec02a5c083d0ef6 (name tag: aws-sandbox-to-gcp-sandbox-cg1). *Select the first Customer Gateway you created that matches Tunnel 0 in GCP.*

* Routing Options: Dynamic

![](https://cdn-images-1.medium.com/max/4276/1*TU2n-sZ9vd94f3wHW9-MKQ.png)

16. Tunnel Options. Normally we do not need to alter the tunnel options, however there is a known compatibility issue with the size of the AWS transform set, which is larger than GCP will accept. This causes the GCP Cloud VPN Tunnel rekey process to fail. You can read more about this [here](https://cloud.google.com/network-connectivity/docs/vpn/how-to/creating-ha-vpn).

We reduced the transform sets to use the following options.

![](https://cdn-images-1.medium.com/max/2948/1*FsweTWCv-9bl5_jKnN6G6Q.png)

Repeat the same Advanced Options for Tunnel 2. *Click “Create VPN Connection”*

17. *Select the new Site-to-Site VPN connection (aws-sandbox-to-gcp-sandbox-vpn1). Click “Download Configuration”*

![](https://cdn-images-1.medium.com/max/2000/1*-wTi2fBGLP_qMLvpUCeEPw.png)

*Select the following.*

* Vendor: Generic

* Platform: Generic

* Software: Vendor Agnostic

![](https://cdn-images-1.medium.com/max/2000/1*DOOpWPjxt5X7xXu6mBRS3g.png)

*Click “Download”* to save the Configuration for use later when we go back to GCP (file name should be <vpn id.txt>).

18. Create a second Site-to-Site VPN Connection by repeating steps 14 to 17. The second Site-to-Site VPN Connection should have the following values.

* Name tag: aws-sandbox-to-gcp-sandbox-vpn2

* Target Gateway Type: Virtual Private Gateway

* Virtual Private Gateway: vgw-00a6862ef16d8404e (name tag: aws-sandbox-to-gcp-sandbox-vpg). *Select the Virtual Private Gateway you just created.*

* Customer Gateway: Existing

* Customer Gateway: cgw-02852ebb8f7e7b000 (name tag: aws-sandbox-to-gcp-sandbox-cg2). *Select the second Customer Gateway you created that matches Tunnel 1 in GCP.*

* Routing Options: Dynamic

**GCP Side**

19. Add VPN Tunnels — Moving back to the GCP Side and continuing from Step 6. For Peer VPN gateway, *make sure “On-prem or Non Google Cloud” is selected. “Peer VPN gateway name” Click choose in the drop down and select “Create new peer VPN Gateway”*

![](https://cdn-images-1.medium.com/max/2000/1*YDXMbe-IOLCmN-XRPxQe9g.png)

20. The “Add a peer VPN gateway” screen will pop up. Fill in the following.

* Name: gcp-sandbox-to-aws-sandbox-peer-gateway

* Interfaces: Select “four interfaces”

Open the first configuration file you downloaded from AWS.

* Interface 0 IP address: 23.22.97.192 *(Look for “IPSec Tunnel #1” -> “#3: Tunnel Interface Configuration” -> “Outside IP Addresses:” -> Virtual Private Gateway.*)

![](https://cdn-images-1.medium.com/max/2000/1*e7xf4o0MoTgMceRr1o5xDQ.png)

* Interface 1 IP Address: 54.84.104.211 *(Look for “IPSec Tunnel #2” -> “#3: Tunnel Interface Configuration” -> “Outside IP Addresses:” -> Virtual Private Gateway.)*

Open the second configuration file you downloaded from AWS

* Interface 2 IP Address: 3.209.145.120 *(Look for “IPSec Tunnel #1” -> “#3: Tunnel Interface Configuration” -> “Outside IP Addresses:” -> Virtual Private Gateway.)*

* Interface 3IP Address: 52.86.52.34 *(Look for “IPSec Tunnel #2” -> “#3: Tunnel Interface Configuration” -> “Outside IP Addresses:” -> Virtual Private Gateway.)*

Everything should look like the following:

![](https://cdn-images-1.medium.com/max/2000/1*fTLASZDsRX-3bGNu0fW0Rw.png)

*Click “Create”*

21. Your screen should now show additional options and 4 VPN Tunnels that are not configured.

![](https://cdn-images-1.medium.com/max/2488/1*PcctEdkeU8cGrLULa6XV9A.png)

22. *Under Cloud Router, select the Cloud Router* *we created in step 2.*

![](https://cdn-images-1.medium.com/max/2000/1*gjc74gR0v_vnX0ykK0dQoA.png)

23. *Click on the first “VPN tunnel* (not yet configured). Fill in the following. *Click “Done”*

* Associated Cloud VPN gateway interface: “0 : 35.242.61.73”

* Associated peer VPN gateway interface: “0 : 23.22.97.192”

* Name: gcp-sandbox-to-aws-sandbox-tunnel1

* Description: gcp-sandbox-to-aws-sandbox-tunnel1

* IKE version: IKEv2

* IKE pre-shared key: Lw3mshg60bimW61itK7rWI6UnIo16aM7 *(Look in the **first** configuration file you downloaded under “IPSec Tunnel #1” -> “#1: Internet Key Exchange Configuration” -> “- Pre-Shared Key”*

![](https://cdn-images-1.medium.com/max/2000/1*ii3afq7eYdjXcaGzO_6M0Q.png)

It should look like the following:

![](https://cdn-images-1.medium.com/max/2000/1*AF6VeBomtacW1q19C0C_-Q.png)

24.* Click on the second “VPN tunnel *(not yet configured). Fill in the following. *Click “Done”*

* Associated Cloud VPN gateway interface: “0 : 35.242.61.73”

* Associated peer VPN gateway interface: “1 : 54.84.104.211”

* Name: gcp-sandbox-to-aws-sandbox-tunnel2

* Description: gcp-sandbox-to-aws-sandbox-tunnel2

* IKE version: IKEv2

* IKE pre-shared key: Z8O3eDTtExyIaHsdp6UYFMhjJn4mSCt5 *(Look in the **first** configuration file you downloaded under “IPSec Tunnel #2” -> “#1: Internet Key Exchange Configuration” -> “- Pre-Shared Key”*

25. *Click on the third “VPN tunnel* (not yet configured). Fill in the following. *Click “Done”*

* Associated Cloud VPN gateway interface: “1 : 35.220.60.77”

* Associated peer VPN gateway interface: “2 : 3.209.145.120”

* Name: gcp-sandbox-to-aws-sandbox-tunnel3

* Description: gcp-sandbox-to-aws-sandbox-tunnel3

* IKE version: IKEv2

* IKE pre-shared key: _wcatTFJKrTwcZkOtwWcTKTEWcoRu0ka *(Look in the **second** configuration file you downloaded under “IPSec Tunnel #1” -> “#1: Internet Key Exchange Configuration” -> “- Pre-Shared Key”*

26. *Click on the fourth “VPN tunnel* (not yet configured). Fill in the following. *Click “Done”*

* Associated Cloud VPN gateway interface: “1 : 35.220.60.77”

* Associated peer VPN gateway interface: “3 : 52.86.52.34”

* Name: gcp-sandbox-to-aws-sandbox-tunnel4

* Description: gcp-sandbox-to-aws-sandbox-tunnel4

* IKE version: IKEv2

* IKE pre-shared key: j7QtMjzwJxGTptV66TvzhRGe8VqI__e1 *(Look in the **second** configuration file you downloaded under “IPSec Tunnel #1” -> “#1: Internet Key Exchange Configuration” -> “- Pre-Shared Key”*

27. The completed VPN Tunnels should appear as follows:

![](https://cdn-images-1.medium.com/max/2484/1*BxdoelRjVFsdk9XjRPIrzA.png)

*Click “Create & Continue”*

28. Configure BGP Sessions — You should see the following screen:

![](https://cdn-images-1.medium.com/max/2568/1*rKpMIcj8TFFOqqE25RbosQ.png)

*Click “Configure” on VPN tunnel1*

29. Create BGP session — Fill out the following using the **first** configuration file you downloaded from AWS. *Click “Save & Continue”*

* Name: gcp-sandbox-to-aws-sandbox-bgp1

* Peer ASN: 64512 (The ASN from the Virtual Private Gateway from AWS)

* Advertised route priority (MED): blank

* Cloud Router BGP IP: 169.254.224.134 *(Look for “IPSec Tunnel #1” -> “#3: Tunnel Interface Configuration” -> “Inside IP Addresses:” -> Customer Gateway)*

* BGP peer IP: 169.254.224.133 *(Look for “IPSec Tunnel #1” -> “#3: Tunnel Interface Configuration” -> “Inside IP Addresses:” -> Virtual Private Gateway)*

* *Select “Use Cloud Router’s advertisements (Default)”*

![](https://cdn-images-1.medium.com/max/2000/1*dJz0ZB-BKhG9QEEll-GkHA.png)

30. *Click on the second BGP Session configuration. *Create BGP session — Fill out the following using the **first** configuration file you downloaded from AWS. *Click “Save & Continue”*

* Name: gcp-sandbox-to-aws-sandbox-bgp2

* Peer ASN: 64512 (The ASN from the Virtual Private Gateway from AWS)

* Advertised route priority (MED): blank

* Cloud Router BGP IP: 169.254.95.90 *(Look for “IPSec Tunnel #2” -> “#3: Tunnel Interface Configuration” -> “Inside IP Addresses:” -> Customer Gateway)*

* BGP peer IP: 169.254.95.89 *(Look for “IPSec Tunnel #2” -> “#3: Tunnel Interface Configuration” -> “Inside IP Addresses:” -> Virtual Private Gateway)*

* *Select “Use Cloud Router’s advertisements (Default)”*

31. *Click on the third BGP Session configuration.* Create BGP session — Fill out the following using the **second **configuration file you downloaded from AWS. *Click “Save & Continue”*

* Name: gcp-sandbox-to-aws-sandbox-bgp3

* Peer ASN: 64512 (The ASN from the Virtual Private Gateway from AWS)

* Advertised route priority (MED): blank

* Cloud Router BGP IP: 169.254.240.26* (Look for “IPSec Tunnel #1” -> “#3: Tunnel Interface Configuration” -> “Inside IP Addresses:” -> Customer Gateway)*

* BGP peer IP: 169.254.240.25* (Look for “IPSec Tunnel #1” -> “#3: Tunnel Interface Configuration” -> “Inside IP Addresses:” -> Virtual Private Gateway)*

* *Select “Use Cloud Router’s advertisements (Default)”*

32. *Click on the third BGP Session configuration.* Create BGP session — Fill out the following using the **second **configuration file you downloaded from AWS. *Click “Save & Continue”*

* Name: gcp-sandbox-to-aws-sandbox-bgp4

* Peer ASN: 64512 (The ASN from the Virtual Private Gateway from AWS)

* Advertised route priority (MED): blank

* Cloud Router BGP IP: 169.254.112.106 *(Look for “IPSec Tunnel #2” -> “#3: Tunnel Interface Configuration” -> “Inside IP Addresses:” -> Customer Gateway)*

* BGP peer IP: 169.254.112.105 *(Look for “IPSec Tunnel #2” -> “#3: Tunnel Interface Configuration” -> “Inside IP Addresses:” -> Virtual Private Gateway)*

* *Select “Use Cloud Router’s advertisements (Default)”*

33. Your BGP Configuration should look like the following:

![](https://cdn-images-1.medium.com/max/2680/1*gTYZQGomSqBAuQxCjZep2Q.png)

*Click “Save BGP Configuration”*

34. You should now see a summary of the VPN Setup:

![](https://cdn-images-1.medium.com/max/2000/1*lSQ8lObsCsyvNwb_K0x5nA.png)

*Click “OK”*

35. In your list of Cloud VPN Tunnels, refresh you browser after a few minutes. You should see that the VPN tunnel and BGP Sessions are showing “Established” for all four tunnels.

![](https://cdn-images-1.medium.com/max/5720/1*-fAdxEB2e_k0lruI7HJzog.png)

**AWS Side**

36. If you now go back to AWS, list the Site-to-Site VPN connections*. Click on the tunnels you created. In the Tunnel Details tab should be the two created tunnels, both showing ‘UP’.*

![](https://cdn-images-1.medium.com/max/3792/1*65i37Ws4wJ4zsszVum5IYA.png)

37. Now that the connection are established, we need to make sure the routes from GCP are added to the route tables. *In VPC, click on “Route Tables” in the left nav bar. Select the route table associated with your VPC. Click on the “Route Propagation” tab. You should see the virtual private gateway you created with “Propagate” set to “No”. Click on “Edit route propagation”*

![](https://cdn-images-1.medium.com/max/5368/1*flriEVMyQ0iCfG53gBkDTw.png)

38. *Check the box under “Propagate”. Click “Save”*

![](https://cdn-images-1.medium.com/max/4200/1*j7qQBLsexj0DyBXssn9N-A.png)

39. Repeat step 37 & 38 for the other subnets that are part of the VPC.

40. When you *click on the “Routes” tab*, you should now see the 10.10.0.0/24 and 10.10.1.0/24 GCP CIDR ranges add to the table automatically.

![](https://cdn-images-1.medium.com/max/4996/1*-3AiSYcdv6lfZAFN-8yyfQ.png)

41. Verify it all works — For this tutorial my test bastions use the following ips:

* AWS — 10.0.1.220

* GCP — 10.10.1.2

SSH into the AWS host (10.0.1.220) in the private subnet. Ping the GCP host (10.10.1.2) which exist in the GCP private subnet. You can also run a netcat nc -zv 10.10.1.2 22 to confirm that it is also allowing connections.

    ubuntu@ip-10-0-1-220:~$ ping 10.10.1.2
    PING 10.10.1.2 (10.10.1.2) 56(84) bytes of data.
    64 bytes from 10.10.1.2: icmp_seq=1 ttl=63 time=3.97 ms
    64 bytes from 10.10.1.2: icmp_seq=2 ttl=63 time=3.03 ms
    64 bytes from 10.10.1.2: icmp_seq=3 ttl=63 time=3.05 ms
    64 bytes from 10.10.1.2: icmp_seq=4 ttl=63 time=3.49 ms
    ^C
    --- 10.10.1.2 ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss, time 3003ms
    rtt min/avg/max/mdev = 3.036/3.388/3.976/0.391 ms
    ubuntu@ip-10-0-1-220:~$ nc -zv 10.10.1.2 22
    Connection to 10.10.1.2 22 port [tcp/ssh] succeeded

Let us do the opposite. We’ll ping and netcat the AWS host (10.0.1.220) from the GCP host (10.10.1.2).

    ubuntu@app-service:~$ nc -zv 10.0.1.220 22
    10.0.1.220: inverse host lookup failed: Unknown host
    (UNKNOWN) [10.0.1.220] 22 (ssh) open
    ubuntu@app-service:~$ ping 10.0.1.220
    PING 10.0.1.220 (10.0.1.220) 56(84) bytes of data.
    64 bytes from 10.0.1.220: icmp_seq=1 ttl=63 time=3.72 ms
    64 bytes from 10.0.1.220: icmp_seq=2 ttl=63 time=3.09 ms
    64 bytes from 10.0.1.220: icmp_seq=3 ttl=63 time=3.04 ms
    64 bytes from 10.0.1.220: icmp_seq=4 ttl=63 time=3.17 ms

## Summary

Obviously setting up cross-cloud networking with high-availability is an involved process with many components, and requires a considerable effort to stay on top of all the gotchas. Hardening a setup of this nature is made possible by leveraging automation using tools like gcloud, aws-cli, or Terraform.

Now that we have a solid, highly available VPN connection across providers, we can connect services like CloudSQL to AWS.
