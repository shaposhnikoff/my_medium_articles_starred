
# Poor man’s load balancing with Docker


> I shall be telling this with a sigh
Somewhere ages and ages hence: 
Two roads diverged in a wood, and I —
 I took the one less traveled by,
And that has made all the difference 
**The Road Not Taken. Robert Frost**

We don’t know for sure where the expression “round robin” comes from, but we’re sure that it has nothing to do with a [well-fed red-breasted bird](http://www.dailymail.co.uk/news/article-1233134/Round-Robin-The-portly-bird-whos-fatter-Christmas-turkey.html) nor with any celebrity named Robin. The expression has had different meanings over time but the ‘ rotational’ meaning we use today seems to be related with an 18th century sailor’s term for a letter of complaint on which the names of those signing were written in a circle. The idea behind the circle was to disguise who had signed first. Mutiny was taken as a serious offense then and captains tended to recur to exemplary hanging.

![](https://cdn-images-1.medium.com/max/2560/1*gB4yQBZFdrNm2ybNQrjW3w.jpeg)

Today, we use the term”round robin” widely in IT to address less complicated problems.* *For example, in the context of load balancing among web servers as to not endanger the service (rather than our lives) and provide better response times.

### DNS Round Robin: A “**poor man’s**” load balancing solution.

There are different approaches and techniques for load balancing and DNS round robin is just one of them. With DNS round robin, we rely on the DNS server responses instead of a strictly dedicated appliance, server or container with more sophisticated algorithms and strategies…so we often refer DNS round robin as poor’s man load balancing. How does it work exactly? Simply by getting the DNS servers to respond to DNS requests not only with a single potential IP address, but with one out of a list of potential IP addresses corresponding to several servers or containers that host identical services.

### DNS Round Robin in Docker

So all we need is a DNS server then. Luckily for us, the Docker Engine implements an embedded DNS server for containers in user-defined networks since Docker 1.10. In particular, containers that are run with a network alias ( — net-alias) are resolved by this embedded DNS with the IP address of the container when the alias is used. Let´s see it:

    **$ docker -v**
    Docker version 1.11.1, build 5604cbe

    **$ docker network create frontend
    **4d74a7516ce33b405d1ce84bc5769f421a63b35e2861f42dd799ce10e2c5684c

    **$ docker run -d --net frontend --net-alias web nginx:alpine 
    **2103aca4af4183463ffa06b7dd060b7eb72f375ca13f8f42f0c7d31576d0c97d

    **$ docker inspect --format=’{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}’ lb_server_1**
    172.18.0.2

    $ **docker run --rm --net frontend lherrera/bind-tools host -t A web**
    web has address 172.18.0.2

Moreover, as of Docker 1.11, if you give multiple containers the same alias, Docker’s DNS server will return the IP addresses of all of them and facilitate load-balancing by alternating the IP at the top of the list the DNS server returns. In order words, different clients will get a list with the same IP addresses but in different order for the same name/alias:

    **$ docker run -d --net frontend --net-alias web nginx:alpine**
    f207666448458b2de42fb4c6fdba410613644bb1a4527e6a1134eb8be1dc2054

    **$ docker run -d --net frontend --net-alias web nginx:alpine**
    4f4aedafd294294c970a1c381b09d303de475b6b124bbb5ca0b5a38f1c70

    **$ docker run -d --net frontend --net-alias web nginx:alpine**
    342bf81acea8d33b238ba741dfced45f04f882f0bf7b102d6ef15e891c155409

    $ **docker run --rm -it --net frontend lherrera/bind-tools host -t A web**
    web has address 172.18.0.5
    web has address 172.18.0.3
    web has address 172.18.0.2
    web has address 172.18.0.4

    **$ docker run --rm -it --net frontend lherrera/bind-tools host -t A web**
    web has address 172.18.0.3
    web has address 172.18.0.4
    web has address 172.18.0.5
    web has address 172.18.0.2

Did you notice how Docker’s embebed DNS server actually shuffles IP address records? In principle, load will be evenly distributed between server containers , as clients will try to reach just the first IP of the list returned when they look up the server name/alias.

![](https://cdn-images-1.medium.com/max/2000/1*kMlt4lCK35y4LqpJX0oqgg.jpeg)

### The effect of RFC3484 on DNS Round Robin

So we are done, right? Let’s start load balancing..…Well, unfortunately not, there’s a mechanism called **default address selection** that interferes somehow with DNS round-robin (whether we’re using Docker or not) that you should be aware of . To make a [long story](https://daniel.haxx.se/blog/2012/01/03/getaddrinfo-with-round-robin-dns-and-happy-eyeballs/) short, it turns out that most OSes, applications and libraries today use an additional filter when getting the IP address, called destination address selection (defined in RFC3484). Chances are that the standard package or library you are using in your code too. So, when your client queries DNS and receives a list of IP adresses, a destination selection algorithm kicks in and returns the IP address or IP addresses which have the longest prefix match rather than the first address in the list provided by the DNS server. This somehow works against the randomization provided by DNS Round Robin and could give you just the opposite results you were looking for. But before we see an example, what do we mean with the longest matching prefix ? It´s an algorithm used by routers and servers that compares at bit level the client IP address with the IP addreses provided in the list and then returns one or more IP addresses with the largest number of leading bits matching the client IP address. Let’s take a look at an example to understand how this works.

Let’s say we have five servers with these IPs:

* server _1— 172.20.0.2–10101100.00010100.00000000.00000010

* server_2 — 172.20.0.4–10101100.00010100.00000000.00000100

* server_3 — 172.20.0.5–10101100.00010100.00000000.00000101

* server_4 — 172.20.0.6–10101100.00010100.00000000.00000110

* server_5 — 172.20.0.7–10101100.00010100.00000000.00000111

And two clients trying to reach them, with these IPs

* client_1 — 172.20.0.3–10101100.00010100.00000000.00000011

* client_2 — 172.20.0.8–10101100.00010100.00000000.00001000

Let’s say that all five servers share the same net-alias (server) and so all their IPs are returned by Docker embebed DNS server when one of the clients performs a DNS lookup. Now, here´s how the destination address selection algorithm will interfere with DNS round robin:

* Client_1. Matching leading bits: server_1(31), server_2(29), server_3(29), server_4(29),server_5(29). If the container was using DNS round robin, it should select server_1, server_2, server 3…However, as the destination address selection is applied, it will always choose server_1.

* Client_2, Matching leading bits: server_1(28), server_2(28), server_3(28), server_4(28),server_5(28). Here, as all servers have the same leading matching bits…the client will use them all successively…as if it was using DNS round robin in the first place.

Let’s see all this in practice:

    **$ git clone [https://github.com/lherrera/lb](https://github.com/lherrera/lb)**
    **$ cd lb
    $ docker-compose -v
    **docker-compose version 1.7.1, build 0a9ab35
    **$ docker-compose up -d**
    Creating lb_client_1
    Creating lb_server_1
    **$ docker-compose scale server=5
    **Creating and starting lb_server_2 ... done
    Creating and starting lb_server_3 ... done
    Creating and starting lb_server_4 ... done
    Creating and starting lb_server_5 ... done
    **$ docker-compose scale client=2**
    Creating and starting lb_client_2 ... done

    $ **docker inspect — format “{{.Name}} {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}” $(docker ps -aq) | sort -t ‘ ‘ -k2**
    /lb_server_1 172.20.0.2
    /lb_client_1 172.20.0.3
    /lb_server_2 172.20.0.4
    /lb_server_3 172.20.0.5
    /lb_server_4 172.20.0.6
    /lb_server_5 172.20.0.7
    /lb_client_2 172.20.0.8

    **$ docker-compose logs client | grep client_1 | grep PING | sort | uniq -c
    ** 19 client_1  | PING server (172.20.0.2)
    **$ docker-compose logs client | grep client_2 | grep PING | sort | uniq -c**
      4 client_2  | PING server (172.20.0.2)
      2 client_2  | PING server (172.20.0.4)
      5 client_2  | PING server (172.20.0.5)
      2 client_2  | PING server (172.20.0.6)
      5 client_2  | PING server (172.20.0.7)

To sum up, Docker’s embebed DNS server returns all the IP addreses for a given service name in random order and could be used for DNS round robin. But the order in which its presented to the client application depends on the resolver library/API being used at the client end (for example, most command line linux tools will use getaddrinfo() function that orders the entries based on RFC 3484 ). Make sure you override the address selection mechanism if you expect an even load distribution or use an alternative approach.

### Alternatives

Apart from the unexpected impact of the default address selection in DNS round robin, this approach has a number of potentially important drawbacks. For example, it does not take into account the current load or responsiveness of the nodes. Don’t take me wrong: DNS Round robin is extremely simple to implement and is an excellent mechanism to increment capacity in certain scenarios, provided that you take into account the default address selection bias. But if you require more advanced or sophisticated approaches you could resort to:

* [**HAproxy](https://hub.docker.com/_/haproxy/)**. It’s a **very** fast load balancer and proxy for TCP and HTTP-based applications.

* **Nginx** NGINX is used by over 40% of the world’s busiest websites and is an open-source reverse proxy server and load balancer for TCP and HTTP-based applications.

* [**Interlock](https://github.com/ehazlett/interlock).** Interlock is an event driven extension system for Docker. It uses the Docker Event stream to notify HAProxy or Nginx and provide in that way **dynamic** load balancing and reverse proxy functionality.

* [**Consul Template + Registrator**.](https://www.hashicorp.com/blog/introducing-consul-template.html) Consul template is a standalone application that can query service entries, keys, and key values in Consul and will reconfigure and reload Nginx containers on new changes. Registrator helps here by monitoring Docker Even stream and updates Consul Server accordingly.

* [**Gorb](https://github.com/kobolog/gorb). **An IPVS front-end with a REST API interface. You can use it to control local IPVS instance in the Kernel to dynamically register virtual services and backends. It also supports basic TCP and HTTP health checks.

* [**Citrix Netscaler** VPX as a container! ](https://msandbu.wordpress.com/2016/05/14/setting-up-the-netscaler-cpx-load-balancing-on-a-ubuntu-docker-host-with-nginx/)(Beta)

* [**Weave](https://www.youtube.com/watch?v=M9f2YVpR4aQ)**. Probably the best Docker’s network plugin out there. Here’s [a a](https://www.weave.works/guides/using-nginx-as-a-reverse-proxyload-balancer-with-weave-and-docker/)rticle explaining their approach.

* [Load balancing with Docker Universal Control Plane.](https://blog.docker.com/2016/03/configuring-load-balancing-service-discovery-docker-universal-control-plane/)

Be sure to check back for Part 2 of this article, where we’ll take a closer look at some of these other load balancing strategies!

Special thanks to [**Sebastiaan van Stij](https://github.com/thaJeztah)n, [Santhosh Manohar](https://github.com/sanimej) **and** [Bryan Boreham](https://github.com/bboreham) **for helping me understand how Rule 9 in RFC3484 was interfering with DNS Round Robin.
