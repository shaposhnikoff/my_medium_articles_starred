
# AWS VPN High Availability

There are other approaches, that use two servers per VPC (OpenVPN or OpenSWAN/StrongSWAN etc) connecting to remote VPN servers. High availability (HA) is implemented using a script to monitor and swap routes in the route table. That might be cheaper to implement. You could also use Amazon’s Direct Connect etc. Inbound HA can be solved by using two or more servers. But Amazon does not provide an easy way to do outbound HA. This approach addresses outbound HA.

I used AWS Virtual Private Gateway (VPG)/VPN functionality. Amazon charges approximately $0.05 per hour per VPN connection. So this is relatively cheap. Amazon VPN’s come with two tunnels (you can use only one if you want). Amazon uses two tunnels to do HA on their side. But the other end of the tunnels have to terminate in the same customer machine (both tunnels are tied to same public IP). Refer to the following diagram. For simplicity, I also represented two VPN tunnels with a single line. I have one public and private network in each VPC. There could be more, depending on number of availability zones etc. I also chose hub-spoke model where multiple VPCs connect to a central VPC. This might not be ideal depending on how traffic flows. The arrows in the diagram are used to indicate who is initiating the VPN connection (not the direction in which traffic flows). Traffic always flows out through Amazon VPN’s.

![Simple setup with 3 VPC’s](https://cdn-images-1.medium.com/max/3916/1*InoUilY5nAKM5FFvHsH0fw.png)*Simple setup with 3 VPC’s*

I chose [VyOS](http://vyos.net/wiki/Main_Page) because it is free and easy to setup. I used version 1.1.3. If you have to use a older version, use 1.0.5 (which is available as EC2 AMI). Version 1.1.0 (which is the latest AMI version) has a nasty bug ([http://bugzilla.vyos.net/show_bug.cgi?id=358](http://bugzilla.vyos.net/show_bug.cgi?id=358)). See instructions at the bottom on how to upgrade VyOS to 1.1.3.

To start with, setup two VyOS instances in each VPC public subnet. VyOS should have a public IP address. Depending on your bandwidth requirements, pick the right instance type. Larger instances get more bandwidth. When I was testing this setup, I used t2.micro. But you might want something like m3.medium or better. VPG/VPN traffic is not throttled (according to Amazon). Replace the public IP addresses with Elastic IP (EIP) address. If for some reason, the VyOS instance goes away, it will be a hassle to replace it. Elastic IP addresses make it easier to replace VyOS instances. Disable source/dest checks for all VyOS instances. And you might also want to enable instance termination protection.

When creating VyOS instances, create a new security group and use it. Start with wide open permissions. Once the configuration is done, don’t forget to tighten the security group. Keep SSH (tcp/22) and NTP (udp/123) access. For BGP, IPSec etc, instead of opening individual ports, I gave all traffic access to the VPN tunnels public IP addresses. If you have Network ACL’s, don’t forget to tweak them.

Once the VyOS instances are up and elastic IP addresses assigned, create Virtual Private Gateway (VPG), Customer Gateway (CGW) and VPN connections. Attach the VPG to the VPC. I used dynamic routing for my purpose, with private BGP ASN 65000. CGW addresses should be the VyOS EIP, which are the other end of the tunnel. I used the above diagram as reference to figure out which VPN is connecting to which VyOS EIP.

Select each VPN connection and download the configuration (Vendor: Vyatta). Vyatta configuration is compatible with VyOS. The downloaded configuration cannot be used as-is. The downloaded file has duplicate information because it includes configuration for two tunnels.

Figure out common configuration for all VyOS instances. Use a tool like Cluster SSH to apply it on all instances:
> $ ssh -i <private-key> vyos@<elastic ip>
$ configure
> set vpn ipsec ike-group AWS lifetime ‘28800'
set vpn ipsec ike-group AWS proposal 1 dh-group ‘2'
set vpn ipsec ike-group AWS proposal 1 encryption ‘aes128'
set vpn ipsec ike-group AWS proposal 1 hash ‘sha1'
> set vpn ipsec ipsec-interfaces interface ‘eth0'
set vpn ipsec esp-group AWS compression ‘disable’
set vpn ipsec esp-group AWS lifetime ‘3600'
set vpn ipsec esp-group AWS mode ‘tunnel’
set vpn ipsec esp-group AWS pfs ‘enable’
set vpn ipsec esp-group AWS proposal 1 encryption ‘aes128'
set vpn ipsec esp-group AWS proposal 1 hash ‘sha1'
> set vpn ipsec ike-group AWS dead-peer-detection action ‘restart’
set vpn ipsec ike-group AWS dead-peer-detection interval ‘15'
set vpn ipsec ike-group AWS dead-peer-detection timeout ‘30'

When configuring the hub VyOS instances, you will need to repeat the following steps multiple times (depending on the number of spoke regions)

Next configure the interfaces on each VyOS instance. It might be useful to update the description. Like ‘Oregon to Virginia Tunnel 1'
> set interfaces vti vti0 address ‘169.A.B.C/30'
set interfaces vti vti0 description ‘Oregon to Virginia Tunnel 1'
set interfaces vti vti0 mtu ‘1436'
> set interfaces vti vti1 address ‘169.A.B.D/30'
set interfaces vti vti1 description ‘Oregon to Virginia VPC tunnel 2'
set interfaces vti vti1 mtu ‘1436'

In the **site-to-site** section, **local-address** will be set to the EIP. VyOS will not like that, because it does not know anything about the EIP. Change it to the local eth0 address. And apply the site-to-site configuration:
> set vpn ipsec site-to-site peer X.Y.Z.A authentication mode ‘pre-shared-secret’
set vpn ipsec site-to-site peer X.Y.Z.A authentication pre-shared-secret ‘XX1'
set vpn ipsec site-to-site peer X.Y.Z.A description ‘VPC tunnel 1'
set vpn ipsec site-to-site peer X.Y.Z.A ike-group ‘AWS’
set vpn ipsec site-to-site peer X.Y.Z.A local-address ‘10.A.B.C'
set vpn ipsec site-to-site peer X.Y.Z.A vti bind ‘vti0'
set vpn ipsec site-to-site peer X.Y.Z.A vti esp-group ‘AWS’
> set vpn ipsec site-to-site peer A.B.C.D authentication mode ‘pre-shared-secret’
set vpn ipsec site-to-site peer A.B.C.D authentication pre-shared-secret ‘XX2’
set vpn ipsec site-to-site peer A.B.C.D description ‘VPC tunnel 2'
set vpn ipsec site-to-site peer A.B.C.D ike-group ‘AWS’
set vpn ipsec site-to-site peer A.B.C.D local-address ‘10.X.Y.Z'
set vpn ipsec site-to-site peer A.B.C.D vti bind ‘vti1'
set vpn ipsec site-to-site peer A.B.C.D vti esp-group ‘AWS’

Next configure the BGP protocol:
> set protocols bgp 650xy neighbor 169.A.B.E remote-as ‘xyz1'
set protocols bgp 650xy neighbor 169.A.B.E soft-reconfiguration ‘inbound’
set protocols bgp 650xy neighbor 169.A.B.E timers holdtime ‘30'
set protocols bgp 650xy neighbor 169.A.B.E timers keepalive ‘30'
> set protocols bgp 650xy neighbor 169.A.B.F remote-as ‘xyz1'
set protocols bgp 650xy neighbor 169.A.B.F soft-reconfiguration ‘inbound’
set protocols bgp 650xy neighbor 169.A.B.F timers holdtime ‘30'
set protocols bgp 650xy neighbor 169.A.B.F timers keepalive ‘30'

In my setup, I also changed the ntp servers and the hostname:
> set system host-name **my-hostname
**delete system ntp
set system ntp server 0.a.b.ntp.org
set system ntp server 1.a.b.ntp.org
set system ntp server 2.a.b.ntp.org

Finally, configure the routes/networks BGP will advertise to the other end. For all the spoke VyOS instances, they just need to advertise their VPC subnet. For BGP to advertise the route, the route should be in the routing table. Amazon instances only get a route for their subnet and not the entire VPC. If you check the output of **show ip route**, you will see a route for the VyOS subnet. Add a static route for the entire VPC. The follow example assumes you have a 10.X.0.0/16 VPC:
> set protocols static route 10.X.0.0/16 next-hop 10.X.0.1 distance 10

Tell BGP about this subnet so that it starts advertising:
> set protocols bgp 650xy network 10.X.0.0/16

In my setup, I also used inbound and outbound policies on which BGP routes will be advertised/received. For the spoke VyOS instances, something similar to the following. Where 10.X.0.0/16 is the local spoke VPC subnet. 10.Y.0.0/16 is the hub VPC subnet.
> set policy prefix-list IN rule 10 action permit
set policy prefix-list IN rule 10 prefix 10.Y.0.0/16
set policy prefix-list OUT rule 10 action permit
set policy prefix-list OUT rule 10 prefix 10.X.0.0/16
set policy route-map IN rule 10 action permit
set policy route-map IN rule 10 match ip address prefix-list IN
set policy route-map IN rule 20 action deny
set policy route-map OUT rule 10 action permit
set policy route-map OUT rule 10 match ip address prefix-list OUT
set policy route-map OUT rule 20 action deny
> set protocols bgp 650xy neighbor 169.A.B.E route-map export OUT
set protocols bgp 650xy neighbor 169.A.B.E route-map import IN
> set protocols bgp 650xy neighbor 169.A.B.F route-map export OUT
set protocols bgp 650xy neighbor 169.A.B.F route-map import IN

That is it for the spoke VyOS instances. Commit the changes and backup the configuration. And keep a copy of the configuration somewhere safe (not on the VyOS instances).
> commit
save
save /home/vyos/backup.conf
exit

From the backed up configuration file, it is better to remove sections that are specific to the VyOS instance. This way, the configuration can be merged easily when instances need to be replaced later.
> interfaces ethernet eth0
service
system

You can refer to VyOS documentation Wiki, but some commands I found useful:
> show ip route
show ip bgp
show ip bgp summary
show ip bgp neighbor 169.A.B.E advertised-routes
show ip bgp neighbor 169.A.B.E received-routes
show vpn debug

Once the spoke VyOS instances are up and running, the VPC VPN in the hub regions should light up. You should see that tunnels are up and 1 BGP route is received

There are couple of ways to configure the Hub VyOS instances. You will have to apply all the spoke VPC configurations on the hub VyOS instances. The only differences are in the policy and route sections. You can redistribute the individual BGP routes to all spoke regions. But I chose to distribute one Class A subnet instead:
> set protocols static route 10.Y.0.0/16 next-hop 10.Y.0.1 distance 10
set protocols static route 10.0.0.0/8 next-hop 10.Y.0.1 distance 100
set protocols bgp 650xy network 10.0.0.0/8

As for policies, the OUT policy will be similar to above except it uses a Class A subnet
> *set policy prefix-list OUT rule 10 action permit
set policy prefix-list OUT rule 10 prefix 10.0.0.0/8
set policy route-map OUT rule 10 action permit
set policy route-map OUT rule 10 match ip address prefix-list OUT
set policy route-map OUT rule 20 action deny*

For IN policies, each spoke region will be advertising a different Class B subnet. So create IN policies for each spoke region:
> *set policy prefix-list IN1 rule 10 action permit
set policy prefix-list IN1 rule 10 prefix 10.X.0.0/16
set policy route-map IN1 rule 10 action permit
set policy route-map IN1 rule 10 match ip address prefix-list IN1
set policy route-map IN1 rule 20 action deny
….*

Identify the pair of bgp neighbors (for the IN policy) and apply the inbound and outbound policies:
> *set protocols bgp 650xy neighbor 169.A.B.E route-map export OUT
set protocols bgp 650xy neighbor 169.A.B.E route-map import IN1
….*

Commit, save, backup and exit

At this point, all VPN tunnels in all VPC’s should be green. And they should be receiving exactly 1 route. Modify all the VPC route tables and enable route propagation. At this point, all instances should be able to reach other instances irrespective of which VPC they are in.

If it is necessary to replace a VyOS instance:
> Kill the instance that is being replaced
Create another instance in the same public subnet with the same private IP
Choose the correct security group and SSH key
Disable the source/dest checks
Reassign the EIP from the old instance
SCP the backup configuration file to the new VyOS instance
SSH to the instance:
$ ssh -i <key> vyos@<eip>
$ configure
$ delete system ntp
$ commit
$ merge /home/vyos/backup.conf
$ commit
$ save
$ exit

To upgrade the VyOS instance. SSH to the instance and run the command:
> $ add system image [http://packages.vyos.net/iso/release/<version>/vyos-<version>-amd64.iso](http://packages.vyos.net/iso/release/1.1.3/vyos-1.1.3-amd64.iso)

There are 4 tunnels from each spoke VPC to the hub and vice-versa. If one VyOS box dies, traffic will start flowing through the other one. Start ping from an instance in spoke1 VPC to another instance in spoke2 VPC. While this is running, reboot VyOS1 instance in Hub region. You should see minimal disruption. Once the VyOS1 box comes up, reboot VyOS2 in hub VPC, traffic should fail over appropriately.
