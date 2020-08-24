
# A Practical Guide to HashiCorp Consul — Part 1



*This is part 1 of 2 part series on A Practical Guide to HashiCorp Consul. This part is primarily focused on understanding the problems that Consul solves and how it solves them. The second part is more focused on a practical application of Consul in a real-life example and will be published next week. Let’s get started.*

How about setting up discoverable, configurable, and secure service mesh using a single tool?

What if we tell you this tool is platform-agnostic and cloud-ready?

And comes as a single binary download.

All this is true. The tool we are talking about is [HashiCorp Consul](https://www.consul.io/).

Consul provides [service discovery](https://en.wikipedia.org/wiki/Service_discovery), [health checks](https://microservices.io/patterns/observability/health-check-api.html), [load balancing](https://en.wikipedia.org/wiki/Load_balancing_(computing)), [service graph,](https://medium.com/@marcus.cavalcanti/using-service-graphs-to-reduce-mttr-in-a-http-based-architecture-624d2c9d54c1) [identity enforcement via TLS](https://www.nevatech.com/blog/post/What-you-need-to-know-about-securing-APIs-with-mutual-certificates), and [distributed service configuration management](https://www.pluralsight.com/guides/role-of-configuration-management-in-devops).

Let’s learn about Consul in details below and see how it solves these complex challenges and makes the life of a distributed system operator easy.

## Introduction

[Microservices](https://martinfowler.com/articles/microservices.html) and [other distributed systems](https://medium.com/microservices-learning/the-evolution-of-distributed-systems-fec4d35beffd) can enable faster, simpler software development. But there’s a trade-off resulting in greater operational complexity around inter-service communication, configuration management, and network segmentation.

![Monolithic Application (representational) — with different subsystems A, B, C and D](https://cdn-images-1.medium.com/max/4000/1*s1RzShXPrrlBK7ukbvZZMw.png)*Monolithic Application (representational) — with different subsystems A, B, C and D*

![Distributed Application (representational) — with different services A, B, C and D](https://cdn-images-1.medium.com/max/2000/1*pU3kq8BIvhIQG1OmhJan1w.png)*Distributed Application (representational) — with different services A, B, C and D*

[HashiCorp Consul is an open-source tool](https://github.com/hashicorp/consul) that solves these new complexities by providing service discovery, health checks, load balancing, a service graph, mutual TLS identity enforcement, and a configuration key-value store. These features make Consul an ideal control plane for a [service mesh](https://www.nginx.com/blog/what-is-a-service-mesh/).

![HashiCorp Consul supports Service Discovery, Service Configuration, and Service Segmentation](https://cdn-images-1.medium.com/max/2000/1*vU6KN4SVSvowi_-N3NcSQA.png)*HashiCorp Consul supports Service Discovery, Service Configuration, and Service Segmentation*

[HashiCorp announced Consul in April 2014](https://www.hashicorp.com/blog/consul-announcement) and it has since then got a good community acceptance.

This guide is aimed at discussing some of these crucial problems and exploring the various solutions provided by HashiCorp Consul to tackle these problems.

Let’s rundown through the topics that we are going to cover in this guide. The topics are written to be self-content. You can jump directly to a specific topic if you want to.

## Brief Background on Monolithic vs. Service-oriented Architectures (SOA)

Looking at traditional architectures of [application delivery](https://www.nginx.com/resources/glossary/application-delivery/), what we find is [a classic monolith](https://en.wikipedia.org/wiki/Monolithic_application). When we talk about monolith, we have [a single application deployment](http://www.codingthearchitecture.com/2014/11/19/what_is_a_monolith.html).

Even if it is a single application, typically it has multiple different sub-components.

One of the examples that HashiCorp’s CTO Armon Dadgar gave during his [introductory video for Consul](https://www.youtube.com/watch?v=mxeMdl0KvBI&t=) was about — delivering desktop banking application. It has a discrete set of sub-components — for example, authentication (say subsystem A), account management (subsystem B), fund transfer (subsystem C), and foreign exchange (subsystem D).

Now, although these are independent functions — system A authentication vs system C fund transfer — we deploy it as a single, monolith app.

Over the last few years, we have seen a [trend away from this kind of architecture](https://medium.com/swlh/moving-away-from-monolithic-architecture-8a19def7f7f9). There are several reasons for this shift.

Challenge with a monolith is: Suppose there is a bug in one of the subsystems, system A, related to authentication.

![Representational bug in Subsystem A in our monolithic application](https://cdn-images-1.medium.com/max/2000/1*408EI1HTDfUuO-tgRPBqYw.png)*Representational bug in Subsystem A in our monolithic application*

We can’t just fix it in system A and update it in production.

![Representational bug fix in Subsystem A in our monolithic application](https://cdn-images-1.medium.com/max/2000/1*mTZg6HMPstJTzLXGvYtACw.png)*Representational bug fix in Subsystem A in our monolithic application*

We have to update system A and do a redeploy of the whole application, which we need deployment of subsystems B, C, and D as well.

![Bugfix in one subsystem results in the redeployment of the whole monolithic application](https://cdn-images-1.medium.com/max/2000/1*SMgw13a_nPQohbAnn5y1LA.png)*Bugfix in one subsystem results in the redeployment of the whole monolithic application*

This whole redeployment is not ideal. Instead, we would like to do a deployment of individual services.

The same monolithic app delivered as a set of individual, discrete services.

![Dividing monolithic application into individual services](https://cdn-images-1.medium.com/max/2000/1*pU3kq8BIvhIQG1OmhJan1w.png)*Dividing monolithic application into individual services*

So, if there is a bug fix in one of our services:

![Representational bug in one of service, in this case Service A of our SOA application](https://cdn-images-1.medium.com/max/2000/1*NZyOUO8Xfd1YoA2yTQF6Sw.png)*Representational bug in one of service, in this case Service A of our SOA application*

and we fix that bug:

![Representational bug fix in Service A of our SOA application](https://cdn-images-1.medium.com/max/2000/1*tr3OWhWns8wN-y5cplphdg.png)*Representational bug fix in Service A of our SOA application*

We can do the redeployment of that service without coordinating the deployment with other services. What we are essentially talking about is one form of microservices.

![The bug fix will result in redeployment of only Service A within our whole application](https://cdn-images-1.medium.com/max/2000/1*uIC5YeUFzXRLYUySB5TaIA.png)*The bug fix will result in redeployment of only Service A within our whole application*

This gives a big boost to our [development agility](https://jaxenter.com/most-important-benefit-microservices-is-agility-129472.html). We don’t need to coordinate our development efforts across different development teams or even systems. We will have the [freedom of developing and deploying independently](https://www.nginx.com/blog/deploying-microservices/). One service on a weekly basis and other quarterly. This is going to be a big advantage for the development teams.

But, there is no such thing as a free lunch.

The development efficiency we have gained introduces its [own set of operational challenges](https://docs.aws.amazon.com/aws-technical-content/latest/microservices-on-aws/challenges-of-microservices.html). Let’s look at some of those.

## Service discovery in a monolith, its challenges in a distributed system, and Consul’s solution

### Monolithic applications

Assuming two services in a single application want to talk to one another. One way is to expose a method, make it public and allow other services to call it. In a monolithic application, it is a single app, and the services would expose public functions and it would simply mean function calls across services.

![Subsystems talk to each other via function call within our monolithic application](https://cdn-images-1.medium.com/max/4000/1*of_USTFz-8ymQisXT3Ujjw.png)*Subsystems talk to each other via function call within our monolithic application*

As this is a function call within a process, it has happened in-memory. Thus, it’s fast, and we need not worry about how our data was moved and if it was secure or not.

### Distributed Systems

In the distributed world, service A is no longer delivered as the same application as service B. So, how does service A finds service B if it wants to talk to B?

![Service A tries to find Service B to establish communication](https://cdn-images-1.medium.com/max/2000/1*2dDyMx3Oao_3TgSpjiU1wQ.png)*Service A tries to find Service B to establish communication*

Service A might not even be on the same machine as service B. So, there is a network in play. And it is not as fast and there is a latency that we can measure on the lines of milliseconds, as compared to nanoseconds of a simple function call.

### Challenges in Distributed Systems

As we already know by now, two services on a distributed system have to discover one-another to interact. One of the traditional ways of solving this is by using [load balancers](https://www.citrix.com/glossary/load-balancing.html).

![A load balancer sits between services to allow them to talk to each other](https://cdn-images-1.medium.com/max/4000/1*y5buEEXBDMNhvph1vrfmgQ.png)*A load balancer sits between services to allow them to talk to each other*

Load balancers would sit in front of each service with a static IP known to all other services.

![A load balancer between two services allows two way traffic](https://cdn-images-1.medium.com/max/4000/1*D52YvzuLj5e3Uoans7tX5Q.png)*A load balancer between two services allows two way traffic*

This gives an ability to add multiple instances of the same service behind the load balancer and it would direct the traffic accordingly. But this load balancer IP is static and hard-coded within all other services, so services can skip discovery.

![Load balancers allow communication between multiple instances of same service](https://cdn-images-1.medium.com/max/4000/1*p5ZJnWsJj9tvcBDMyW7eMA.png)*Load balancers allow communication between multiple instances of same service*

The challenge is now to maintain a set of load balancers for each individual services. And we can safely assume, there was originally a load balancer for the whole application as well. The cost and effort for maintaining these load balancers have increased.

With load balancers in front of the services, they are [a single point of failures](https://en.wikipedia.org/wiki/Single_point_of_failure). Even when we have multiple instances of service behind the load balancer if it is down our service is down. No matter how many instances of that service are running.

Load balancers also [increase](https://blog.buoyant.io/2016/03/16/beyond-round-robin-load-balancing-for-latency/) the [latency](https://en.wikipedia.org/wiki/Latency_(engineering)) of inter-service communication. If service A wishes to talk to service B, request from A will have to first talk to the load balancer of service B and then reach B. The response from B will also have to go through the same drill.

![Maintaining an entry of a service instances on an application-wide load balancer](https://cdn-images-1.medium.com/max/4000/1*6sNxPwoFl5WhQTLNJu8kCg.png)*Maintaining an entry of a service instances on an application-wide load balancer*

And by nature, [load balancers are manually managed](https://dzone.com/articles/load-balancers-and-high-volume-traffic-management-1) in most cases. If we add another instance of service, it will not be readily available. We will need to register that service into the load balancer to make it accessible to the world. This would mean manual effort and time.

### Consul’s Solutions

Consul’s solution to service discovery problem in distributed systems is [a central service registry](https://auth0.com/blog/an-introduction-to-microservices-part-3-the-service-registry/).

Consul maintains a central registry that contains the entry for all the upstream services. When a service instance starts, it is registered on the central registry. The registry is populated with all the upstream instances of the service.

![Consul’s Service Registry helps Service A find Service B and establish communication](https://cdn-images-1.medium.com/max/4000/1*FzvAjlmvA19cL3jx3vz_lw.png)*Consul’s Service Registry helps Service A find Service B and establish communication*

When a service A wants to talk to service B, it will discover and communicate with B by querying the registry about the upstream service instances of B. So, instead of talking to a load balancer, the service can directly talk to the desired destination service instance.

Consul also provides health-checks on these service instances. If one of the service instances or services itself is unhealthy or fails its health-check, the registry would then know about this scenario and would avoid returning the service’s address. The work that the load-balancer would do is handled by the registry in this case.

Also, if there are multiple instances of the same service, Consul would send the traffic randomly to different instances. Thus, leveling the load among different instances.

Consul has handled our challenges of failure detection and load distribution across multiple instances of services without the necessity of deploying a centralized load balancer.

The traditional problem of slow and manually managed load balancers is taken care of here. Consul programmatically manages registry, which gets updated when any new service registers itself and becomes available for receiving traffic.

This helps with scaling the services with ease.

## Configuration Management in a monolith, its challenges in a distributed environment, and Consul’s solution

### Monolithic Applications

When we look at the configuration for a monolithic application, they tend to be somewhere along the lines of [giant YAML, XML or JSON files](https://en.wikipedia.org/wiki/Configuration_file). That configuration is supposed to configure the entire application.

![Single configuration file shared across different parts of our monolithic application](https://cdn-images-1.medium.com/max/4000/1*JTBdjyxWxzZxXnM83gR1Nw.png)*Single configuration file shared across different parts of our monolithic application*

Given a single file, all of our subsystems in our monolithic application would now consume the configuration from the same file. Thus creating a consistent view of all our subsystems or services.

If we wish to change the state of the application using a configuration update, it would be easily available to all the subsystems. The new configuration is simultaneously consumed by all the components of our application.

### Distributed Systems

Unlike monolith, distributed services would not have a common view on configuration. The configuration is now distributed and there every individual service would need to be configured separately.

![A copy of application configuration is distributed across different services](https://cdn-images-1.medium.com/max/4000/1*a0-7aK49ga-QbGjQGhEjpw.png)*A copy of application configuration is distributed across different services*

### Challenges in Distributed Systems

* The configuration is to be spread across different services. Maintaining consistency between the configuration on different services after each update is a challenge.

* Moreover, the challenge grows when we expect the configuration to be updated dynamically.

### Consul’s Solutions

Consul’s solution for configuration management in a distributed environment is the central Key-Value store.

![Consul’s KV store allows seamless configuration mapping on each service](https://cdn-images-1.medium.com/max/4000/1*D-GLrfMja-nHYF15afDNhg.png)*Consul’s KV store allows seamless configuration mapping on each service*

Consul solves this challenge in a unique way. Instead of spreading the configuration across different distributed services as configuration pieces, it pushes the whole configuration to all the services and configures them dynamically on the distributed system.

Let’s take an example of state change in configuration. The changed state is pushed across all the services in real-time. The configuration is consistently present with all the services.

## Network segmentation in a monolith, its challenges in distributed systems, and Consul’s solutions

### Monolithic Applications

When we look at our classic monolithic architecture, the network is typically divided into three different zones.

The **first zone** in our network is publicly accessible. The traffic coming to our application via the internet and reaching our load balancers.

The **second zone** is the traffic from our load balancers to our application. Mostly an internal network zone without direct public access.

The **third zone** is the closed network zone, primarily designated for data. This is considered to be an isolated zone.

![Different network zones in typical application](https://cdn-images-1.medium.com/max/4000/1*KKlN_b99AwZ1kwQ-U49VUg.png)*Different network zones in typical application*

Only the load balancers zone can reach into the application zone and only the application zone can reach into the data zone. It is a straightforward zoning system, simple to implement and manage.

### Distributed Systems

The pattern changes drastically for distributed services.

![Complex pattern of network traffic and routes across different services](https://cdn-images-1.medium.com/max/4000/1*epcSJWV7XfsYl2JIwtqllA.png)*Complex pattern of network traffic and routes across different services*

There are multiple services within our application network zone itself. Each of these service talks to others within this network, making it a complicated traffic pattern.

### Challenges in Distributed Systems

* The primary challenge is that the [traffic is not in any sequential flow](https://dzone.com/articles/eastwest-is-the-new-northsouth). Unlike monolithic architecture, where the flow was defined from load balancers to the application and application to data.

* Depending on the access pattern we want to support, the traffic might come from different endpoints and reaching different services.

![The client essentially talks to each service within the application directly or indirectly](https://cdn-images-1.medium.com/max/4000/1*iOgYCmBe_c2fGU3I1zi-vg.png)*The client essentially talks to each service within the application directly or indirectly*

* Given multiple services and the ability to [support multiple endpoints](https://www.mulesoft.com/resources/api/microservices-and-apis) allows us to deploy multiple service consumers and providers.

* Due to the nature of the system, [security is our next challenge](https://www.owasp.org/images/2/20/Microservice_Security.pdf). Services should be able to identify that the traffic they are receiving is from a [verified and trusted entity in the network](https://medium.com/tech-tajawal/microservice-authentication-and-authorization-solutions-e0e5e74b248a).

![SOA demands control over trusted and untrusted sources of traffic](https://cdn-images-1.medium.com/max/4000/1*3RcqJaDEo4-5LaGAgm_pPw.png)*SOA demands control over trusted and untrusted sources of traffic*

* Controlling the flow of traffic and segmenting the network into groups or chunks will become a bigger issue. Also, making sure we have strict rules that guide us with [partitioning the network](https://en.wikipedia.org/wiki/Network_partition) based on who should be allowed to talk to whom and vice versa is also vital.

### Consul’s Solutions

Consul’s solution to the overall network segmentation challenge in distributed systems is by implementing service graphs and [mutual TLS](https://medium.com/sitewards/the-magic-of-tls-x509-and-mutual-authentication-explained-b2162dec4401).

![Service-level policy enforcement to define traffic pattern and segmentation using Consul](https://cdn-images-1.medium.com/max/4000/1*oViPnL_BJq6xA1cv2PFwEQ.png)*Service-level policy enforcement to define traffic pattern and segmentation using Consul*

Consul solves the problem of network segmentation by [centrally managing the definition around who can talk to whom](https://www.consul.io/docs/agent/acl-system.html). Consul has a dedicated feature for this called [Consul Connect](https://learn.hashicorp.com/consul/getting-started/connect).

Consul Connect enrolls these policies of inter-service communication that we desire and implements it as part of the service graph. So, a policy might say service A can talk to service B, but B cannot talk to C, for example.

The higher benefit of this is, it is not IP restricted. Rather it’s service level. This makes it scalable. The policy will be enforced on all instances of service and there will be no hardbound firewall rule specific to a service’s IP. Making us independent of the scale of our distribution network.

Consul Connect also handles service identity using popular [TLS protocol](https://en.wikipedia.org/wiki/Transport_Layer_Security). It distributes the TLS certificate associated with a service.

These certificates help other services securely identify each other. TLS also help with secure communication between the services. This makes for trusted network implementation.

Consul enforces TLS using an [agent-based proxy](https://www.consul.io/docs/connect/proxies.html) attached to each service instance. This proxy acts as [a sidecar](https://docs.microsoft.com/en-us/azure/architecture/patterns/sidecar). The use of proxy, in this case, prevents us from making any change into the code of original service.

This allows for the higher-level benefit of enforcing encryptions on data at rest and data in transit. Moreover, it will assist with fulfilling compliances required by laws around privacy and user identity.

## Basic Architecture of Consul

Consul is a distributed and [highly available system](https://en.wikipedia.org/wiki/High_availability).

Consul is shipped as a single binary download for all popular platforms. The executable can run as a client as well as the server.

Each node that provides services to Consul runs a Consul agent. Each of these agents talks to one or more Consul servers.

![Basic Architecture of Consul](https://cdn-images-1.medium.com/max/4000/1*CMGSDDRL4GE8B_mThBVwsg.png)*Basic Architecture of Consul*

The consul agent is responsible for health-checking the services on the node as the health-check of the node itself. It is not responsible for service discovery or maintaining key/value data.

Consul servers are where data is stored and replicated.

Consul can run with a single server, but it is recommended by HashiCorp to run a set of 3 to 5 servers to avoid failures. As all the data is stored at Consul server-side, with a single server, the failure could cause a data loss.

With a multi-servers cluster, they elect a leader among themselves. It is also recommended by HashiCorp to have a cluster of servers per datacenter.

During the discovery process, any service in search for other services can query the Consul servers or even Consul agents. The Consul agents forward the queries to Consul servers automatically.

![Consul Agent sits on a node and talks to other agents on the network synchronizing all service-level information](https://cdn-images-1.medium.com/max/4000/1*YX_1u7IYyRX3klLJ8VCdLg.png)*Consul Agent sits on a node and talks to other agents on the network synchronizing all service-level information*

If the query is the cross-data center, the queries are forwarded by the Consul server to the remote Consul servers. The results from remote Consul servers are returned to the original Consul server.

## Getting Started with Consul

This section is dedicated to closely looking at Consul as a tool, with some hands-on experience.

### Download and Install

As discussed above, Consul ships as a single binary downloaded from HashiCorps website or from Consul’s GitHub repo releases section.

Single binary can run as Consul Server or even as Consul Client Agent.

You can download Consul from here — [Download Consul page](https://www.consul.io/downloads.html).

![Various download options for Consul on different operating systems](https://cdn-images-1.medium.com/max/4000/1*V-Sru26vM1l_LoRW0RGB2g.png)*Various download options for Consul on different operating systems*

We will download Consul on the command line using the link from the download page

<iframe src="https://medium.com/media/975bbddfb33c8a1f4cf3aca933e8a396" frameborder=0></iframe>

Unzip the downloaded zip file.

<iframe src="https://medium.com/media/13b8f10037f4384b1c6d67ac6cfcf8be" frameborder=0></iframe>

Add it to PATH.

<iframe src="https://medium.com/media/4324665a04d55c7b78063cb2fe27757c" frameborder=0></iframe>

### Use Consul

Once you unzip the compressed file and put the binary under your PATH, you can run it like this.

<iframe src="https://medium.com/media/e921c6f79e9826d741dd50fd6b09c022" frameborder=0></iframe>

This will start the agent in development mode.

### Consul Members

While the above command is running, you can check for all the members in Consul’s network.

<iframe src="https://medium.com/media/fbbeb4252dc7ae2e17857ec709bf8891" frameborder=0></iframe>

Given we only have one node running, it is treated as a server by default. You can designate an agent as a server by supplying the server as a command line parameter or server as a configuration parameter to Consul’s config.

The output of the above command is based on the gossip protocol and is eventually consistent.

### Consul HTTP API

For a strongly consistent view of the Consul’s agent network, we can use HTTP API provided out of the box by Consul.

<iframe src="https://medium.com/media/5855c134e4de1ddab5d3156cf5eafb6f" frameborder=0></iframe>

### Consul DNS Interface

Consul also provides a DNS interface to query nodes. It serves DNS on 8600 port by default. That port is configurable.

<iframe src="https://medium.com/media/d144847721bb8032de59b51443a31855" frameborder=0></iframe>

Registering a service on Consul can be achieved either by writing a service definition or by sending a request over an appropriate HTTP API.

### Consul Service Definition

The service definition is one of the popular ways of registering a service. Let’s take a look at one such service definition example.

To host our service definitions we will add a configuration directory, conventionally names as consul.d [— ‘.d’ represents](http://blog.siphos.be/2013/05/the-linux-d-approach/) that there are set of configuration files under this directory, instead of single config under name consul.

<iframe src="https://medium.com/media/3af4de783c23dde6a800a3b747b9b2c0" frameborder=0></iframe>

Write the service definition for a fictitious Django web application running on port 80 on localhost.

<iframe src="https://medium.com/media/2eed9668b44a791812a2c5e107e63081" frameborder=0></iframe>

To make our consul agent aware of this service definition, we can supply the configuration directory to it.

<iframe src="https://medium.com/media/df2ae3be0bbbd7ce3246d073b7986923" frameborder=0></iframe>

The relevant information in the log here are the sync statements related to the “web” service. Consul agent as accepted our config and synced it across all nodes. In this case one node.

### Consul DNS Service Query

We can query the service with DNS, as we did with node. Like so:

<iframe src="https://medium.com/media/5930b98de68a9123794b8b821da71247" frameborder=0></iframe>

We can also query DNS for service records that give us more info into the service specifics like port and node.

<iframe src="https://medium.com/media/5a144090dd0fc27ba682d72d6c9d3519" frameborder=0></iframe>

You can also use the TAG that we supplied in the service definition to query a specific tag:

<iframe src="https://medium.com/media/ba0fdd02f45faa42afa436bd1c3c8593" frameborder=0></iframe>

### Consul Service Catalog Over HTTP API

Service could similarly be queried using HTTP API:

<iframe src="https://medium.com/media/7c45d05f7b58f8cc865f228e740471f7" frameborder=0></iframe>

We can filter the services based on health-checks on HTTP API:

<iframe src="https://medium.com/media/f0b9b2c10fd3255102fd4c88d0b571f0" frameborder=0></iframe>

### Update Consul Service Definition

If you wish to update the service definition on a running Consul agent, it is very simple.

There are three ways to achieve this. You can send a SIGHUP signal to the process, reload Consul which internally sends SIGHUP on the node or you can call HTTP API dedicated to service definition updates that will internally reload the agent configuration.

<iframe src="https://medium.com/media/b074170604a0d548aeea4bf97a38fb6f" frameborder=0></iframe>

Send SIGHUP to 21289

<iframe src="https://medium.com/media/2ae997cea09253b77a98405cd8f932b6" frameborder=0></iframe>

Or reload Consul

<iframe src="https://medium.com/media/30f4c5c38ccf89d7cc52e3b3ca0177fd" frameborder=0></iframe>

Configuration reload triggered

You should see this in your Consul log.

<iframe src="https://medium.com/media/e39c16e8863c9ce08602299714ea8ff7" frameborder=0></iframe>

### Consul Web UI

Consul provides a beautiful web user interface out-of-the-box. You can access it on [port 8500](https://www.consul.io/docs/agent/options.html).

In this case at [http://localhost:8500](http://localhost:8500). Let’s look at some of the screens.

The home page for the Consul UI services with all the relevant information related to a Consul agent and web service check.

![Exploring defined services on Consul Web UI](https://cdn-images-1.medium.com/max/4000/1*kK3VIHxTu5GJsPe96g8o3Q.png)*Exploring defined services on Consul Web UI*

Going into further details on a given service, we get a service dashboard with all the nodes and their health for that service.

![Exploring node-level information for each service on Consul Web UI](https://cdn-images-1.medium.com/max/4000/1*TyXNpLUX1VvgPM0w3NKyEQ.png)*Exploring node-level information for each service on Consul Web UI*

On each individual node, we can look at the health-checks, services, and sessions.

![Exploring node-specific health-check information, services information, and sessions information on Consul Web UI](https://cdn-images-1.medium.com/max/4000/1*F0qeDiDIcd1mOkUMssmT9A.png)*Exploring node-specific health-check information, services information, and sessions information on Consul Web UI*

Overall, Consul Web UI is really impressive and a great companion for the command line tools that Consul provides.

## How is Consul Different From Zookeeper, doozerd, and etcd?

Consul has first-class support for service discovery, health-check, key-value storage, multi-data centers.

[Zookeeper](https://zookeeper.apache.org/), [doozerd](https://github.com/ha/doozerd), and [etcd](https://github.com/etcd-io/etcd) are primarily based on the key-value store mechanism. To achieve something beyond such key-value, the store needs additional tools, libraries, and custom development around them.

All these tools, including Consul, uses server nodes that require a quorum of nodes to operate and are strongly consistent.

More or less, they all have similar semantics for key/value store management.

These semantics are attractive for building service discovery systems. Consul has out-of-the-box support for service discovery, which the other systems lack.

A service discovery system also requires a way to perform health-checks. As it is important to check for the service’s health before allowing others to discover it. Some systems use heartbeats with periodic updates and TTL. The work for these health checks grows with scale and requires fixed infra. The failure detection window is at least as long as TTL.

Unlike Zookeeper, Consul has client agents sitting on each node in the cluster, talking to each other in the gossip pool. This allows the clients to be thin, gives better health-checking ability, reduces client-side complexity, and solves debugging challenges.

Also, Consul provides native support for [HTTP](https://www.consul.io/api/index.html) or [DNS](https://www.consul.io/docs/agent/dns.html) interfaces to perform system-wide, node-wide, or service-wide operations. Other systems need those being developed around the exposed primitives.

Consul’s website gives a good commentary on comparisons between Consul and other tools.

## Open Source Tools Around HashiCorp Consul

HashiCorp and the community has built several tools around Consul

These Consul tools are created and managed by the dedicated engineers at HashiCorp:

[**Consul Template](https://github.com/hashicorp/consul-template)** (3.3k stars) — Generic template rendering and notifications with Consul. Template rendering, notifier, and supervisor for @hashicorp Consul and Vault data. It provides a convenient way to populate values from Consul into the file system using the consul-template daemon.

[**Envconsul](https://github.com/hashicorp/envconsul)** (1.2k stars) — Read and set environmental variables for processes from Consul. Envconsul provides a convenient way to launch a subprocess with environment variables populated from HashiCorp Consul and Vault.

[**Consul Replicate](https://github.com/hashicorp/consul-replicate) **(360 stars) — Consul cross-DC KV replication daemon. This project provides a convenient way to replicate values from one Consul datacenter to another using the consul-replicate daemon.

[**Consul Migrate](https://github.com/hashicorp/consul-migrate) **— Data migration tool to handle Consul upgrades to 0.5.1+.

The community around Consul has also built several tools to help with registering services and managing service configuration, I would like to mention some of the popular and well-maintained ones -

[**Confd ](https://github.com/kelseyhightower/confd)(5.9k stars) **— Manage local application configuration files using templates and data from etcd or consul.

[**Fabio ](https://github.com/fabiolb/fabio)(5.4k stars)** — Fabio is a fast, modern, zero-conf load balancing HTTP(S) and TCP router for deploying applications managed by the consul. Register your services in consul, provide a health check and Fabio will start routing traffic to them. No configuration required.

[**Registrator ](https://github.com/gliderlabs/registrator)(3.9k stars)** — Service registry bridge for Docker with pluggable adapters. Registrator automatically registers and deregisters services for any Docker container by inspecting containers as they come online.

[**Hashi-UI](https://github.com/jippi/hashi-ui) (871 stars) **— A modern user interface for HashiCorp Consul & Nomad.

[**Git2consul ](https://github.com/breser/git2consul)(594 stars)** — Mirrors the contents of a git repository into Consul KVs. git2consul takes one or many git repositories and mirrors them into Consul KVs. The goal is for organizations of any size to use git as the backing store, audit trail, and access control mechanism for configuration changes and Consul as the delivery mechanism.

[**Spring-cloud-consul](https://github.com/spring-cloud/spring-cloud-consul) (503 stars)** — This project provides Consul integrations for Spring Boot apps through autoconfiguration and binding to the Spring Environment and other Spring programming model idioms. With a few simple annotations, you can quickly enable and configure the common patterns inside your application and build large distributed systems with Consul based components.

[**Crypt ](https://github.com/xordataexchange/crypt)(453 stars)** — Store and retrieve encrypted configs from etcd or consul.

[**Mesos-Consul](https://github.com/mantl/mesos-consul) (344 stars)** — Mesos to Consul bridge for service discovery. Mesos-consul automatically registers/deregisters services run as Mesos tasks.

[**Consul-cli](https://github.com/mantl/consul-cli) (228 stars) **— Command line interface to Consul HTTP API.

## Conclusion

Distributed systems are not easy to build and setup. Maintaining them and keeping them running is altogether another piece of work. HashiCorp Consul makes the life of engineers facing such challenges easier.

As we went through different aspects of Consul, we learned how straightforward it would become for us to develop and deploy an application with distributed or microservices architecture.

Ease of use, excellent documentation, robust production-ready code and community backing, allows adapting and introducing HashiCorp Consul in our technology stack fairly easy.

We hope it was an informative ride on the journey of Consul. Our journey has not yet ended, this was just the first half. We will meet you again with the second part of this article that walks us through practical examples close to real-life applications.

Let’s us know what you would like to hear from us more or if you have any questions around the topic, we will be more than happy to answer those.

## Reference

* [HashiCorp Consul](https://www.consul.io/) and [its repo on GitHub](https://github.com/hashicorp/consul)

* [HashiCorp Consul Guides](https://www.consul.io/docs/guides/index.html) and [Code](https://github.com/hashicorp/consul-guides)

* [Microservices as explained by Martin Fowler et al.](https://martinfowler.com/articles/microservices.html)

* [HashiCorp Blog articles about Consul](https://www.hashicorp.com/blog/category/consul)

*******************************************************************

*This post was originally published on [**Velotio Blog](https://velotio.com/blog/2019/3/11/hashicorp-consul-guide-1).***

[**Velotio Technologies](https://velotio.com/)** is an outsourced software product development partner for technology startups and enterprises. We specialize in enterprise B2B and SaaS product development with a focus on artificial intelligence and machine learning, DevOps, and test engineering.

*Interested in learning more about us? We would love to connect with you on our [**Website](https://velotio.com)**, [**LinkedIn](https://www.linkedin.com/company/velotio-technologies/)** or [**Twitter](https://twitter.com/velotiotech)**.*

*******************************************************************
