
# 100 Days of DevOps — Day 62-Useful Linux Command for Network Troubleshooting

Welcome to Day 62 of 100 Days of DevOps, Focus for today is useful Linux Command for Network Troubleshooting

*First, let start with the understanding of OSI model*
> *What is OSI model?*

*The Open Systems Interconnection model (OSI model) is a conceptual model that characterizes and standardizes the communication functions of a telecommunication or computing system without regard to its underlying internal structure and technology. Its goal is the interoperability of diverse communication systems with standard protocols. The model partitions a communication system into abstraction layers. The original version of the model defined seven layers.*

![](https://cdn-images-1.medium.com/max/2000/1*oC88ySJPU5e7kMt0rrDUvQ.png)

* *Network Layer*

* *This is the layer where the IP Address live*

* *The first command to check IP address in any Linux based system*

* *The first step in network troubleshooting is to check if your system is getting the IP address and it’s correct.*

    *# ifconfig*

    *eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 9001*

    *inet 172.31.31.68  netmask 255.255.240.0  broadcast 172.31.31.255*

    *inet6 fe80::72:e4ff:fee6:e38c  prefixlen 64  scopeid 0x20<link>*

    *ether 02:72:e4:e6:e3:8c  txqueuelen 1000  (Ethernet)*

    *RX packets 61848394  bytes 3763288410 (3.5 GiB)*

    *RX errors 0  dropped 0  overruns 0  frame 0*

    *TX packets 18082424  bytes 267137328226 (248.7 GiB)*

    *TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0*

*OR*

    *# ip addr list*

    *1: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc pfifo_fast state UP group default qlen 1000*

    *link/ether 02:72:e4:e6:e3:8c brd ff:ff:ff:ff:ff:ff*

    *inet 172.31.31.68/20 brd 172.31.31.255 scope global dynamic eth0*

    *valid_lft 2908sec preferred_lft 2908sec*

    *inet6 fe80::72:e4ff:fee6:e38c/64 scope link*

    *valid_lft forever preferred_lft forever*

***NOTE***

* *The ifconfig command is deprecated and the ip command is now favored to provide similar functionality in Centos7/RHEL7*

* ***If the command is needed, it can be accessed by installing the net-tools package.***

*# yum install net-tools*
[**1119297 - Add net-tools to @core group**
*Edit description*bugzilla.redhat.com](https://bugzilla.redhat.com/show_bug.cgi?id=1119297)

* *Next useful command is netstat*

* *netstat — Print network connections, routing tables, interface statistics, masquerade connections, and multicast memberships*
> *To find out my gateway IP for a computer or a network device that allows or controls access to another computer or network.*

    *# netstat -rn*

    *Kernel IP routing table*

    *Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface*

    *0.0.0.0         **172.31.16.1 **    0.0.0.0         UG        0 0          0 eth0*

    *169.254.0.0     0.0.0.0         255.255.0.0     U         0 0          0 eth0*

    *172.31.16.0     0.0.0.0         255.255.240.0   U         0 0          0 eth0*

* ***— route** **,** **-r***

*Display the kernel routing tables. See the description in **route**(8) for details. **netstat** **-r** and **route** **-e** produce the same output*

* ***— numeric** **,** **-n***

*Show numerical addresses instead of trying to determine symbolic host, port or user names.*

* *Here you can see the second column (Gateway **172.31.16.1) which **acts as an entrance to another network*

*OR*

    *# ip r|grep -i default*

    ***default** via 172.31.16.1 dev eth0*

*OR*

    *# route -n*

    *Kernel IP routing table*

    *Destination     Gateway         Genmask         Flags Metric Ref    Use Iface*

    *0.0.0.0         172.31.16.1     0.0.0.0         UG    0      0        0 eth0*

    *169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0*

    *172.31.16.0     0.0.0.0         255.255.240.0   U     0      0        0 eth0*
> *To check for listening port*

    *[root@ip-172-31-31-68 ~]# netstat -antulp*

    *Active Internet connections (servers and established)*

    *Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name*

    *tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      1/systemd*

    *tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      4434/nginx: master*

    *tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      3926/sshd
    udp        0      0 0.0.0.0:68              0.0.0.0:*                           3720/dhclient*

* ***-a,** **— all***

*Show both listening and non-listening (for TCP this means established connections) sockets. With the **— interfaces** option, show interfaces that are not up*

* ***— numeric** **,** **-n***

*Show numerical addresses instead of trying to determine symbolic host, port or user names.*

* *-t tcp*

* *-u udp*

* ***-p,** **— program***

*Show the PID and name of the program to which each socket belongs.*

* ***-l,** **— listening***

*Show only listening sockets*

    ** Here in the first column you see protocol tcp or udp
    * **Recv-Q: **The count of bytes not copied by the user program connected to this socket
    * **Send-Q: **The count of bytes not acknowledged by the remote host
    * Local Address: Address and port number of the local end of the socket
    * **Foreign** **Address: **Address and port number of the remote end of the socket
    * **PID/Program** **name: **Slash-separated  pair  of  the process id (PID) and process name of the process that owns the socket
    * **State:** The state of the socket*

<iframe src="https://medium.com/media/bf6eb74d7d3e2acb5eb9b87b0f8b9bce" frameborder=0></iframe>
> *Next important command is arp which is use to map IP to MAC address*

    *# arp -a*

    *instance-data.us-west-2.compute.internal (169.254.169.254) at 02:e5:df:97:e9:cc [ether] on eth0*

    *ip-172-31-16-1.us-west-2.compute.internal (172.31.16.1) at 02:e5:df:97:e9:cc [ether] on eth0*

    *ip-172-31-29-138.us-west-2.compute.internal (172.31.29.138) at 02:82:95:84:29:f6 [ether] on eth0*

*OR*

    *# ip neigh*

    *169.254.169.254 dev eth0 lladdr 02:e5:df:97:e9:cc REACHABLE*

    *172.31.16.1 dev eth0 lladdr 02:e5:df:97:e9:cc REACHABLE*

    *172.31.29.138 dev eth0 lladdr 02:82:95:84:29:f6 REACHABLE*

*NOTE: arp program is obsolete. For replacement check **ip** **neigh***
> *Application Layer*

* *The **application layer** provides services for an **application** program to ensure that effective communication with another **application** program on a network is possible*

* *Telnet client can be used to open and test connection to a listening port*

    *# telnet localhost 80*

    *Trying ::1...*

    *Connected to localhost.*

    *Escape character is '^]'.*

* *nc can be used for both tcp as well as for udp*

    *# nc -vz localhost 80*

    *Ncat: Version 7.50 ( https://nmap.org/ncat )*

    *Ncat: Connected to ::1:80.*

    *Ncat: 0 bytes sent, 0 bytes received in 0.01 seconds.*

* *OpenSSL : for encrypted communication*

    *openssl s_client -connect www.google.com:443*

    *CONNECTED(00000003)*

    *depth=2 OU = GlobalSign Root CA - R2, O = GlobalSign, CN = GlobalSign*

    *verify return:1*

![](https://cdn-images-1.medium.com/max/2000/1*4bTSrjjyA1_n4kbZ7qoQCw.jpeg)
[**HTTP response status codes**
*HTTP response status codes indicate whether a specific HTTP request has been successfully completed. Responses are…*developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
> *Resolving IP Address*

    *# host www.google.com*

    *www.google.com has address 216.58.217.36*

    *www.google.com has IPv6 address 2607:f8b0:400a:800::2004*

*OR*

    *# nslookup www.google.com*

    *Server:  172.31.0.2*

    *Address: 172.31.0.2#53*

    *Non-authoritative answer:*

    *Name: www.google.com*

    *Address: 216.58.217.36*

*OR*

    *# dig www.google.com*

    *; <<>> DiG 9.9.4-RedHat-9.9.4-73.el7_6 <<>> www.google.com*

    *;; global options: +cmd*

    *;; Got answer:*

    *;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 57761*

    *;; flags: qr rd ra; **QUERY: 1, ANSWER: 1,** AUTHORITY: 0, ADDITIONAL: 1*

    *;; OPT PSEUDOSECTION:*

    *; EDNS: version: 0, flags:; udp: 4096*

    *;; QUESTION SECTION:*

    *;www.google.com.   IN A*

    *;; ANSWER SECTION:*

    *www.google.com.  22 IN A 216.58.217.36*

    *;; Query time: 0 msec*

    *;; SERVER: 172.31.0.2#53(172.31.0.2)*

    *;; WHEN: Sun Apr 14 06:18:44 UTC 2019*

    *;; MSG SIZE  rcvd: 59*

*OR*

* *dig in short format*

    *# dig www.google.com +short*

    *172.217.0.36*

* *Checking other records (eg: MAIL or MX)*

    *# nslookup*

    *> server 8.8.8.8*

    *Default server: 8.8.8.8*

    *Address: 8.8.8.8#53*

    *> set type=mx
    > google.com*

    *Server:  8.8.8.8*

    *Address: 8.8.8.8#53*

    *Non-authoritative answer:*

    *google.com mail exchanger = 50 alt4.aspmx.l.google.com.*

    *google.com mail exchanger = 10 aspmx.l.google.com.*

    *google.com mail exchanger = 20 alt1.aspmx.l.google.com.*

    *google.com mail exchanger = 40 alt3.aspmx.l.google.com.*

    *google.com mail exchanger = 30 alt2.aspmx.l.google.com.*

*OR*

    *# dig mx google.com*

    *; <<>> DiG 9.9.4-RedHat-9.9.4-73.el7_6 <<>> mx google.com*

    *;; global options: +cmd*

    *;; Got answer:*

    *;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 62971*

    *;; flags: qr rd ra; QUERY: 1, ANSWER: 5, AUTHORITY: 0, ADDITIONAL: 1*

    *;; OPT PSEUDOSECTION:*

    *; EDNS: version: 0, flags:; udp: 4096*

    *;; QUESTION SECTION:*

    *;google.com.   IN MX*

    *;; ANSWER SECTION:*

    *google.com.  60 IN MX 20 alt1.aspmx.l.google.com.*

    *google.com.  60 IN MX 30 alt2.aspmx.l.google.com.*

    *google.com.  60 IN MX 40 alt3.aspmx.l.google.com.*

    *google.com.  60 IN MX 50 alt4.aspmx.l.google.com.*

    *google.com.  60 IN MX 10 aspmx.l.google.com.*

    *;; Query time: 2 msec*

    *;; SERVER: 172.31.0.2#53(172.31.0.2)*

    *;; WHEN: Sun Apr 14 06:22:05 UTC 2019*

    *;; MSG SIZE  rcvd: 147*

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
