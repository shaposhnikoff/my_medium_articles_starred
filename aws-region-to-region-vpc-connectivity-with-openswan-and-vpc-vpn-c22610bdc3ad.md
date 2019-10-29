
# AWS Region to Region VPC Connectivity with OpenSwan and VPC VPN

Connected AWS regions is not as simple as it should be. Hopefully AWS will someday provide a VPC Peering connection between regions but until then we need to connect regions with EC2 instances. A couple of techniques we have read up on were using 2 EC2 instances. One in the West region connecting to the East region. In another white paper we found Connecting Multiple VPCs with Astaro Security Gateway https://aws.amazon.com/articles/1909971399457482. This worked ok for a while, but seemed to be expensive and not as performant as it should be.

For a number of reason the Sophos instances from the AWS MarketPlace no longer kept up with what we needed.
- They were built on PV and not HVM. This limited the type of instances we could us.
- Support is limited to the forums unless you pay a additional fee for vendor
- HA inside of AWS is not a options on the Sophos platform
- Scripting of HA is limited to outside of the Sophos instances using health check for yet another instance.
- No ability to enable AWS enhanced networking

Enhanced network is support for SR-IOV, which is short lets a single physical Ethernet adapter show up as multiple adapters and the Hypervisor and thus your EC2 instance does not need as much CPU to push the same amount of network traffic. My understanding is it is like a TCP offload engine for virtualization. Less interrupts are use for networking and can be freed up for your app.
[http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enhanced-networking.html](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enhanced-networking.html)

If you are not passing very much traffic over the VPN connection you can start with a small instance. As soon as you need to do something like replication a database over VPN connection you will need to start increasing the size of your instance. [http://aws.amazon.com/ec2/instance-types/](http://aws.amazon.com/ec2/instance-types/)
I have yet to find documented with the instance types and Network Performance what High, Moderate and Low equates to in bits/sec. The rule of thumb I use is High is about 1Gb/sec and then 1/2 the network speed as the instance size decrease.
I have also been bit by relying on m3.medium instances too many times and getting poor network performance and ultimately having to re-size up to a larger instance like a c4.large

Now that we have the instance type squared away. I have decided to use Amazon Linux with enhanced networking enabled. The current Amazon Linux AMI comes with the enhanced network drives enabled by default. Next I am going to choose the IPsec server. For this I choose Openswan. Simply because I have much more experience with Openswan than any of the other services.
To get Openswan installed on Amazon Linux:
> yum update
yum install openswan

From here I am going to attach this EC2 instance running Openswan in the West to the AWS VPC VPN in the East. Generate you VPN config in the AWS Console on the East coast. For this instance I am using static routing and download the generic VPN config.
Because the AWS settings are static and AWS generates the PSK for you from here it is pretty straight forward. You just need to set the Openswan connection config to match AWS:

Here is the secret sauce in Openswan

    conn west-2-east

    type=tunnel
    authby=secret
    left=instance_ip
    leftid=elastic_ip
    leftnexthop=%defaultroute
    leftsubnet=west_coast_subnet
    right=vpg_ip
    rightnexthop=%defaultroute
    rightsubnet=east_coast_subnet
    phase2=esp
    phase2alg=aes128-sha1
    ike=aes128-sha1
    ikelifetime=28800s
    salifetime=3600s
    pfs=yes
    auto=start
    rekey=yes
    keyingtries=%forever
    dpddelay=10
    dpdtimeout=60
    dpdaction=restart_by_peer

Sources

Originally posted on [https://www.websiteops.com](https://www.websiteops.com)

Connecting Multiple VPCs with EC2 Instances (IPSec)
[https://aws.amazon.com/articles/5472675506466066](https://aws.amazon.com/articles/5472675506466066)

HA NAT:
[http://aws.amazon.com/articles/2781451301784570](http://aws.amazon.com/articles/2781451301784570)

Enhanced Networking
[http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enhanced-networking.html](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enhanced-networking.html)
