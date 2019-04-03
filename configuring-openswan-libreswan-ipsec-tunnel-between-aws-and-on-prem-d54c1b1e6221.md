
# Configuring OpenSwan/LibreSwan IPSec Tunnel Between AWS and ON-PREM

TUNNELING AWS-ONPREM

![IPSEC Tunnel](https://cdn-images-1.medium.com/max/2252/1*lxURW8cri2rRSZ6cQd3eEA.png)*IPSEC Tunnel*

**Introduction**Walk through the creating IPSEC tunnel between AWS and ON-PREM

**Site to Site Tunnel between AWS and ON-Prem :**

**Prerequisite:**

Get the connection information from the client :

    Sample connection details -

    Public IP Address : 200.114.100.106/ 182.**.**.***

    Local Subnet :- [172.0.0.0/](http://192.168.48.0/20)16

    Pre-Shared : “FrdfBhf22” #get the details from client network admin

    IKE Version :1, Main Mode

    Phase1 Proposal:

    Algo :- Aes256, Sha1

    Dh Group : 2

    Key Lifetime :- 86400 seconds

    Phase2 Proposal:

    Encyption : Aes256 , Sha1

    Dh Group :2

    Enable PFS :- Yes

    Auto KeepAlive :- Yes

    Auto negotiate :- Yes

    Key Lifetime :- 43200 seconds

**AWS Setup Configuration :**

1. Select the region from AWS Dashboard, choose the region which is near for your DC, if it’s located in India, choose Mumbai Region from AWS console to avoid latency.

1. Create a VPC from the AWS Dashboard

![](https://cdn-images-1.medium.com/max/2092/0*t-NJPydcIfbT4Z4M.)

3. In the above diagram, we have selected VPC with Public and Private subnet, in this case one subnet will have Internet gateway and other machine will be in private subnet. You can add other subnets as well once the VPC setup done.

4. Now go to your AWS EC2 dashboard console. Select EC2 AMI from the list- Centos or Ubuntu and launch a server with min configuration of 2 core and 4GB RAM.

5. Make sure you have selected the same VPC and Subnets that we have created for this setup. Proceed with launching the machine in public subnet.Once launched, let’s assign a static IP i.e EIP with the machine, so that it won’t change in further. Let’s assume AWS EIP — 35.201.104.15

6. Now time to configure Security group : As it’s very crucial machine make sure we have proper firewall/security rule in place to protect the machine from vulnerabilities.

Open the below ports in the security group :

1. SSH — 22 for internal office IP

1. UDP — 4500 to the DC router/firewall/Host IP

1. UDP — 500 to the DC router/firewall/Host IP

![](https://cdn-images-1.medium.com/max/2000/0*mzelmurdxpm-JPig.png)

### In our case :

    AWS tunnel IP : 35.201.104.15

    AWS Range : 10.100.0.0/16

    On-Prem IP : 200.114.100.106

    Subnet OnPrem : 172.100.0.0/16

7. Time for command line action:

Login into your EC2 machine and execute below commands from a root user :

    # Disable send redirects

    echo 0 > /proc/sys/net/ipv4/conf/all/send_redirects

    echo 0 > /proc/sys/net/ipv4/conf/default/send_redirects

    echo 0 > /proc/sys/net/ipv4/conf/eth0/send_redirects

    echo 0 > /proc/sys/net/ipv4/conf/lo/send_redirects

    # Disable accept redirects

    echo 0 > /proc/sys/net/ipv4/conf/all/accept_redirects

    echo 0 > /proc/sys/net/ipv4/conf/default/accept_redirects

    echo 0 > /proc/sys/net/ipv4/conf/eth0/accept_redirects

    echo 0 > /proc/sys/net/ipv4/conf/lo/accept_redirects

8. vim /etc/sysctl.conf and change:

    net.ipv4.ip_forward = 1

And then append below lines in the config file:

    net.ipv4.conf.all.accept_redirects = 0

    net.ipv4.conf.all.send_redirects = 0

9. Now install libreswan or openswan in the EC2 machine using yum :

    yum install libreswan

    yum install openswan

10. Once installation is done, let’s start configuring As now we have all the connection information, let’s get started with configuring the tunnel

    cd /etc/ipsec.d/

11. Create a connection file, better to create a separate file for each connection :

    vim awsdconnection.conf

12. And append the below config and modify accordingly the connection details :

    conn awsdconnection

    type=tunnel

    authby=secret

    ike=aes256-sha1;modp1024

    left=%defaultroute

    leftid=35.201.104.15 #your ec2 instance EIP

    leftsourceip=10.100.1.112 #you instance private ip

    leftnexthop=%defaultroute

    leftsubnet=10.100.0.0/16#Refer AWS Console VPC range

    right= #DC network details

    rightsubnet=172.100.0.0/16 Take the values from the above shared details

    esp=aes256-sha1

    ikelifetime=86400s

    keylife=”43200"

    pfs=yes

    auto=start

    rekey=yes

    keyingtries=%forever

13. Add or modify the config file — sysctl.conf :

vim /etc/sysctl.conf

    # /etc/ipsec.conf — Libreswan IPsec configuration file

    # This file: /etc/ipsec.conf

    # Enable when using this configuration file with openswan instead of libreswan

    #version 2

    # Manual: ipsec.conf.5

    # basic configuration

    config setup

    # Debug-logging controls: “none” for (almost) none, “all” for lots.

    klipsdebug=all

    plutodebug=all

    # For Red Hat Enterprise Linux and Fedora, leave protostack=netkey

    protostack=netkey

    nat_traversal=yes

    #virtual_private=

    virtual_private=%v4:10.100.0.0/16%v4:172.0.0.0/16 #whitelist your subnet range

    oe=off

    # Enable this if you see “failed to find any available worker”

    #nhelpers=0

    # It is best to add your IPsec connections as separate files in /etc/ipsec.d/

    include /etc/ipsec.d/*.conf

14. Now add the PSK key that’s been shared :

vim /etc/ipsec.secrets : take a backup and empty the content of the file

    Add in the below format :

    #include /etc/ipsec.d/*.secrets

    AWSleftIP OnpremrightIP : shared pre-shared key from the above config

    35.201.104.15 200.114.100.106: PSK ‘FrdfBhf22’

    Once the value is added, now let’s start the IPSEC service in the system.

    /etc/init.d/ipsec restart

15. Once the above service is up, we will now make the network ip

    ipsec auto — add awsnetmagic

After this command execute :

    ipsec auto — up awsnetmagic

Once the above steps completed, disable the source and destination check:

1. Go to ec2 console, right click on the instance and select Networking : change source/destination check disable.

![Networking source/destination](https://cdn-images-1.medium.com/max/2000/0*CMewKnBXGvWz91xH.png)*Networking source/destination*

2. Now go to VPC section, and Filter your VPC and click on route table add the entry of your destination subnet range and add the target as the instance where we have configured our solution.

![Route Table](https://cdn-images-1.medium.com/max/2000/0*uxrAQ1DVz3sQucdV.png)*Route Table*

Vice versa entry will be added in the DC firewall rules as well, to allow AWS network.

### Few important commands to check the status of the tunnel for troubleshooting :

    ipsec — status whack

    Route -n

    ipsec auto — status

    ipsec setup status

    ipsec version

### Host to Host Tunnel :

In this case, follow the same steps just you don’t have to disable source and destination check and also no need of changing route table.

In you **datacentre** Box, add the route table to allow traffic from the AWS machine IP :

Something like :

    iptables -A INPUT -s 35.201.104.15-j ACCEPT

Note: Mentioned IP’s are just example and subjected to change as per your network infrastructure.

Hope this is useful!! Please share your feedback :-)
