
# AWS — Difference between Application load balancer (ALB) and Network load balancer (NLB)

ALB vs NLB in AWS — Application load balancer vs Network load balancer

![Application load balancer and Network load balancer](https://cdn-images-1.medium.com/max/2000/1*S1_KaLSMqVkE-WNsBwe9pA.jpeg)*Application load balancer and Network load balancer*

### TL;DR:

***ALB **— Layer 7, Flexible
**NLB **— Layer 4, Static IPs
**CLB **— Avoid, legacy*

*Both [Application Load Balancer](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) and [Network Load Balancer](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html) are designed from the ground up for the modern paradigm of dynamic port configurations as commonly seen in containerized deployments. Picking which load balancer is right for you will depend on the specific needs of your application, such as whether or not network traffic is HTTP, whether you need end to end SSL/TLS encryption, and whether or not you want host and path based traffic routing.*

*If you are deploying docker containers and using a load balancer to send network traffic to them [EC2 Container Service provides a tight integration with ALB and NLB](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-load-balancing.html) so you can keep your load balancers in sync as you start, update, and stop containers across your fleet.*

**Network Load Balancer **— This is the distribution of traffic based on network variables, such as IP address and destination ports. It is layer 4 (TCP) and below and is not designed to take into consideration anything at the application layer such as content type, cookie data, custom headers, user location, or the application behavior. It is *context-less*, caring only about the network-layer information contained within the packets it is directing this way and that.

This is a TCP Load Balancer only that does some NAT magic at the VPC level. It uses EIPs, so it has a static endpoint unlike ALB and CLBs (by default, contact support if this is a requirement for your CLB or ALB). Each Target can be on different ports.

**Application Load Balancer** — This is the distribution of requests based on multiple variables, from the network layer to the application layer. It is *context-aware *and can direct requests based on any single variable as easily as it can a combination of variables. Applications are load balanced based on their peculiar behavior and not solely on server (operating system or virtualization layer) information.

This is feature fulled Layer-7 load balancer, HTTP and HTTPS listeners only. Provides the ability to route HTTP and HTTPS traffic based upon rules, host based or path based. Like an NLB, each Target can be on different ports. Even supports HTTP/2. Configurable range of health check status codes (CLB only supports 200 OK for HTTP health checks).

The first difference is that the Application Load Balancer (as the name implies) works at the Application Layer (Layer 7 of the OSI model). The network load balancer works at layers 3 & 4 (network and transport layers). The network load balancer just forward requests whereas the application load balancer examines the contents of the HTTP request header to determine where to route the request. So, the application load balancer is performing content based routing.

The other difference between the two is important because network load balancing cannot assure availability of the *application. *This is because it bases its decisions solely on network and TCP-layer variables and has no awareness of the application at all. Generally a network load balancer will determine “availability” based on the ability of a server to respond to ICMP ping, or to correctly complete the three-way TCP handshake. An application load balancer goes much deeper, and is capable of determining availability based on not only a successful HTTP GET of a particular page but also the verification that the *content *is as was expected based on the input parameters.

This is also important to note when considering the deployment of multiple applications on the same host sharing IP addresses (virtual hosts in old school speak). A network load balancer will not differentiate between Application A and Application B when checking availability (indeed it cannot unless ports are different) but an application load balancer will differentiate between the two applications by examining the application layer data available to it. This difference means that a network load balancer may end up sending requests to an application that has crashed or is offline, but an application load balancer will never make that same mistake.

**References**
[**Elastic Load Balancing features - Amazon Web Services**
*You can select the appropriate load balancer based on your application needs. If you need flexible application…*aws.amazon.com](https://aws.amazon.com/elasticloadbalancing/features/)

*Happy Clouding!!!*
