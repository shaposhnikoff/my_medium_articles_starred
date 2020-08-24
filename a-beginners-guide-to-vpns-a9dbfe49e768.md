
# A Beginners Guide to VPN’s

Whenever I find myself in a situation where there is free Wifi I never use it. Why? Because I have been warned by much more tech-savvy people than myself that these types of networks leave you vulnerable to malicious users that could gain access to your personal information (credit card numbers, SSN, login information, etc.) And they are right, how do I know that the WIFI at this particular coffee shop or airport has not been compromised? To take it a step further, when you use the internet how do you know that your Internet Service provider isn’t accessing your data in order to harvest information and profit from it? The real question then becomes, how can I safely protect information I send and receive while using the internet? After some quick googling, there was a term that kept popping up over and over again, “VPN” or “Virtual Private Network”. Before we get into what a VPN is and how it works let's take a step back and talk about Public and Private Networks.

## Public Network VS Private Network

![Image Source [https://buffered.com/faq/what-are-the-benefits-of-vpn/](https://buffered.com/faq/what-are-the-benefits-of-vpn/)](https://cdn-images-1.medium.com/max/2000/1*wG7BYi4TlF6oE1o-0z8G_Q.jpeg)*Image Source [https://buffered.com/faq/what-are-the-benefits-of-vpn/](https://buffered.com/faq/what-are-the-benefits-of-vpn/)*

A public network is a type of network where anyone can gain access and freely use it to connect to other networks or the internet. Think of this like the wifi you would use at Panera, an airport, anywhere that anyone can use the Internet. Public networks typically have little to no restrictions, thus there is a higher chance of security risks when accessing them. Since anyone can access a public network, malicious users may try to infiltrate the system of unsuspecting users that are also using the public network.

A private network is a network that has restrictions and access rules in order to allow access to certain individuals only. This type of network is configured so that devices outside the network cannot access it. Does your network at home require a password? Then it is considered a private network. Only users that have a username and password can access it.

There is no technical difference between private and public networks except for the security, addressing and authentication systems in place.

## What is a VPN?

![Image Source [https://www.iplocation.net/vpn](https://www.iplocation.net/vpn)](https://cdn-images-1.medium.com/max/2000/1*NYBPl7ei5TxMdyUH7yAZUg.png)*Image Source [https://www.iplocation.net/vpn](https://www.iplocation.net/vpn)*

This brings us to VPN’s. A Virtual Private Network is a Private Network that is built to be used over a public infrastructure. VPN’s allow users to securely access a network from different locations from a public network, typically through the Internet. Accessing a network from an unsecured public network puts your data at risk. But a VPN allows you to securely access a private network and share data remotely through public networks. VPN’s do this by routing your data through an encrypted tunnel (this is called tunneling), while simultaneously masking your identity and location (you are using the VPN’s address.

VPN’s are utilized by many businesses because they allow employees to securely access data stored on a company’s network, but VPN’s are very popular with individual users as well. VPN’s are the best way to secure your personal data (and your employer's data), hide your IP address, and subvert “The Man” (government content filters). This is because VPN’s allow individuals to spoof their physical location. The user’s IP address is replaced by the VPN provider, allowing them to bypass content filters.

## What Makes A VPN Secure?

A VPN is secure because of two things.

1. **Dedicated Connections: **a virtual path between two points for the user to send data through.

1. **Tunneling/Encryption Protocols: **The encapsulation of Data that is being sent across a public network. Some common encryption protocols include:

**IP Security (IPSec):** Used to encrypt data packets. There are two modes, Transport mode which only encrypts the message within the packet, and Tunneling Mode which encrypts the entire Data Packet.

**Layer 2 Tunneling Protocol (L2TP) /IPsec:** Generates the tunnel, while IPsec protocol handles the encryption, channel security, and data integrity checks to ensure all of the packets have arrived and the channel has not been compromised.

**Point-to-Point Tunneling Protocol (PPTP):** Not the most secure method. This method only tunnels and encapsulates the data packet. A secondary protocol has to be used to handle the encryption.

**Secure Shell (SSH):** Creates both the VPN tunnel and the encryption that protects it. The data itself isn’t encrypted but the channel it is moving through is.

With these two tools, the VPN can generate virtual p2p connections. Even if someone was able to access the transmitted data, the data would still be encrypted. Different VPN providers use different protocols to encrypt the data being sent through it.

## How can I access a VPN?

Accessing a VPN is essentially a two-step process:

1. Connect to the public internet through an Internet Service Provider.

1. Initiate a VPN connection using client-side software.

The Software established a secure connection and grants the user access to the private network. There are a number of different VPN providers that are a quick google away. But if you would like to get your feet wet with a free VPN option two of the more popular options are listed below:
[**OpenVPN | The World's Most Trusted Virtual Private Network**
*OpenVPN - The Open Source VPN. Your private path to access network resources and services securely for cloud, business…*openvpn.net](https://openvpn.net/)
[**VPN.net - Hamachi by LogMeIn**
*LogMeIn Hamachi is a hosted VPN service that lets you securely extend LAN-like networks to distributed teams, mobile…*www.vpn.net](https://www.vpn.net/)
