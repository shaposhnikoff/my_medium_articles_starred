
# Health Checks and Graceful Degradation in Distributed Systems

Thanks, as always, to Fred Hebert and Sargun Dhillon for reading a draft of this post and offering some invaluable suggestions.

In her [Velocity keynote](https://www.safaribooksonline.com/videos/velocity-conference/9781492026051/9781492026051-video320766), [Tamar Bercovici](https://twitter.com/tamarbercovici?lang=en) of Box highlighted the importance of health checks while automating database failovers. In particular, she emphasized how monitoring end to end query times is a better way of determining the health of a database than simplistic pings.
> … flips traffic to the other side thus remediating the outage. We had to build in some safeguards to prevent flip-flopping and other weird edge cases because you don’t want your automation to run away from you, but this is really straightforward. The trick to making this work successfully, though, is to know **when** to flip the database in the first place, which means you need to be able to correctly assess database health. Now a lot of the metrics we’re used to looking at like CPU load, lock timeouts, error rates — they are all secondary signals. None of those actually tell you whether a database is capable of serving client traffic or not. So if you use those to make your flip decisions, you could get both false positives and false negatives. Instead, our health checker actually executes simple queries against our database hosts and uses the success and failure of **those** to more accurately assess database health.

This led to a discussion with one of my friends who suggested that health checks must be as simple as possible and that live traffic is a better yardstick for understanding the health of a process.

As often as not, discussions around the implementation of a health check pivot around the two options at either extremity of the spectrum — simple pings/signals or comprehensive end-to-end tests. In this post, I aim to underscore the problem behind using the aforementioned form of health-checks for certain types of load balancing decisions as well as need for a more fine-grained approach for measuring the health* *of a process.

### The Two Types of Health Checks

Health checks, even in many modern systems, tend to typically fall under two categories — host level health-checks and service-level health checks.

For instance, Kubernetes implements health checks using *readiness *and *liveness *probes. A *readiness *probe is used to determine if a Pod can serve traffic. Failure of a readiness probe would result in the Pod being removed from the Endpoints that make up a [Service](https://kubernetes.io/docs/concepts/services-networking/service/), resulting in the Pod not being routed any traffic until the *readiness *probe succeeds. A *liveness *probe, on the other hand, is used to indicate if a service is *responsive *or if it’s hung or deadlocked. The failure of a *liveness *probe results in the [kubelet](https://kubernetes.io/docs/admin/kubelet/) restarting the individual container. [Consul](https://www.consul.io/docs/agent/checks.html), similarly, allows for multiple forms of checks, which can be script-based checks or HTTP-based checks that hit a specified URL or TTL based checks or even alias checks.

The most common way of implementing a *service* level health check is by defining a health check endpoint. For example, in gRPC, the health check becomes an RPC call in its own right. gRPC also allows for per *service *health checks as well as an overall *gRPC* *server* health check.

In the past, host level health checks were used as a signal to drive alerts. An example is alerting on CPU load average (rightfully considered to be an antipattern these days). Even when not directly used for alerting, health checks still form the basis upon which several other automated infrastructural decisions are made, such as load balancing and (on occassion) circuit breaking. Service mesh data planes like Envoy, for example, place primacy on *health check* information over service discovery data when it comes to determining whether to route traffic to an instance or not.

### Health is a Spectrum, not a Binary Taxonomy

<iframe src="https://medium.com/media/ae84f4dd6e61c536996439f4dd505806" frameborder=0></iframe>

A ping can only convey whether a service is *up *or down, whereas end-to-end tests are a proxy for whether the system can perform a certain unit of *work*, where the work* *could be something like *execute a database query *or *perform a certain computation. *Irrespective of what form the health check might take, the *result* of the health check is treated as a strictly binary outcome — either the health check “passes” or it “fails”.

In modern, dynamic and oftentimes “auto-scaled” infrastructures, a single process being merely “up” doesn’t matter if said process is unable to complete a given unit of work, rendering simplistic checks like pings almost useless.

While it’s easy to tell when a service is completely *down*, it’s much harder to determine the degree of *healthiness* of a service that’s alive. It’s eminently possible for a process to be “up” (i.e., passing health checks) and be routed traffic only for it to be unable to complete a given unit of work within, say, the p99 latency of the service.

Inability to complete work is often a result of the process getting overloaded. In highly concurrent services, “overload” neatly maps to the number of concurrent requests that can only be serviced by a single process with excessive queueing of the sort that can lead to an increase in latency for the RPC call (though more commonly, the downstream service will simply timeout the request and retry after a configured timeout). This is especially true if the health check endpoint is configured to blindly return an HTTP 200 status code, whereas the real work the service is doing involves network I/O or computation.

![](https://cdn-images-1.medium.com/max/5464/1*hMB-g6OQl7HjWB0AOs6ncA.png)

The “health” of a process is a spectrum. What we’re really interested in is the *quality-of-service — *such as how long it takes for a process to return the result of a given unit of work and the accuracy of the result.

It’s very possible for a process to fluctuate between different degrees of *healthiness* during the course of its lifetime, from being completely *healthy* (as in, being able to function at the expected level of concurrency) to verging on unhealthy (when the queues begin filling up) to the point where it flips entirely into the unhealthy zone (at which point requests experience a degraded quality of service). Only the most trivial of services can afford to be built under the assumption that there wouldn’t exist some degree of partial failure at all times, where partial failure implies some features being up and others being down, not just “some requests are failing and some are succeeding”. If the service architecture cannot gracefully handle partial failure, then the onus automatically falls on the *client* to deal with the error management complexity.

Adaptive, self-healing infrastructures should be built in-keeping with the reality that such fluctuations are entirely *normal. *It’s also important to remember that this distinction only matters as far as load balancing is concerned — it makes little sense, for instance, for the orchestrator to restart a process just because the process is on the cusp of being overloaded.

Put differently, it is entirely reasonable for the orchestration layer to treat the health of a process as a binary state and to only restart a process when it has crashed* *or is hung. However, it’s extremely important that the *load balancing *layer (whether it’s an out-of-process proxy like Envoy or a client side in-process library) act on more fine-grained information about the health of a process to make circuit-breaking and load shedding decisions accordingly. It’s impossible for a service to degrade gracefully if it’s not possible to determine the health of the service at any given time accurately.

In my experience, unbounded concurrency has often been the prime factor that’s led to service degradation or sustained under-performance. Load balancing (and by extension, load shedding) often boils down to managing concurrency effectively and applying backpressure before the system can get overloaded.

### The Need for Feedback Loops when applying Backpressure

[Matt Ranney](https://twitter.com/mranney) has a [phenonemal blog post](http://engineering.voxer.com/2013/09/16/backpressure-in-nodejs/) about unbounded concurrency and the need for backpressure in Node.js. The entire post is well worth a read, but the biggest takeaway (at least for me) was the need for feedback loops between a process and its downstream (usually a load balancer, but sometimes this could also be another service).
> The trick is, when resources are exhausted, something, somewhere, has to give. As demand increases, you can’t magically get more performance forever. To limit incoming work, a good first step is some kind of site-wide rate limiting, by IP address, user, session, or hopefully something meaningful to the application. Many load balancers can do rate limiting in a way that’s more sophisticated than Node.js’s incoming server limits, but they usually don’t notice a problem until your process is already deep in trouble.

Rate limiting and circuit breaking based on [static thresholds and limits can prove to be error-prone and brittle](https://www.infoq.com/articles/envoy-service-mesh-cascading-failure) from both correctness and scalability standpoints. Some load balancers (notably HAProxy) do provide a lot of statistics about the internal queue lengths on a *per server *and *per backend *basis*. *Furthermore*, *HAProxy also allows for an agent-check (an auxiliary check independent of a regular health check) which makes it possible for a process to provide more accurate and dynamic feedback to the proxy about its health. To [cite the docs](https://cbonte.github.io/haproxy-dconv/1.7/configuration.html#5.2-agent-check):
> An agent health check is performed by making a TCP connection to the port set by the agent-portparameter and reading an ASCII string. The string is made of a series of words delimited by spaces, tabs or commas in any order, optionally terminated by /rand/or /n, each consisting of :
> — An ASCII representation of a positive integer percentage, e.g. "75%".
 Values in this format will set the weight proportional to the initial
 weight of a server as configured when HAProxy starts. Note that a zero
 weight is reported on the stats page as DRAINsince it has the same
 effect on the server (it’s removed from the LB farm).
 — The string maxconn: followed by an integer (no space between). Values in
 this format will set the maxconn of a server. The maximum number of
 connections advertised needs to be multipled by the number of load balancers and different backends that use this health check to get the total number of connections the server might receive. Example: maxconn:30
 — The word ready. This will turn the server’s administrative state to the
 READY mode, thus cancelling any DRAIN or MAINT state
 — The word drain. This will turn the server’s administrative state to the
 DRAIN mode, thus it will not accept any new connections other than those
 that are accepted via persistence.
 — The word maint. This will turn the server’s administrative state to the
 MAINT mode, thus it will not accept any new connections at all, and health
 checks will be stopped.
 — The words down, failed, or stopped, optionally followed by a
 description string after a sharp (‘#’). All of these mark the server’s
 operating state as DOWN, but since the word itself is reported on the stats
 page, the difference allows an administrator to know if the situation was
 expected or not : the service may intentionally be stopped, may appear up
 but fail some validity tests, or may be seen as down (eg: missing process,
 or port not responding).
 — The word up sets back the server’s operating state as UP if health checks
 also report that the service is accessible.
> Parameters which are not advertised by the agent are not changed. For
example, an agent might be designed to monitor CPU usage and only report a
relative weight and never interact with the operating status. Similarly, an
agent could be designed as an end-user interface with 3 radio buttons
allowing an administrator to change only the administrative state.
> **However, it is important to consider that only the agent may revert its own actions, so if a server is set to DRAIN mode or to DOWN state using the agent, the agent must implement the other equivalent actions to bring the service into operations again.**
> Failure to connect to the agent is not considered an error as connectivity
is tested by the regular health check which is enabled by the “check”
parameter. Warning though, it is not a good idea to stop an agent after it
reports “down”, since only an agent reporting “up” will be able to turn the
server up again.

This pattern of having a service dynamically communicate its health to its downstream is extremely crucial for building self-adaptive infrastructures. A case in point would be an architecture I worked with at a previous job.

I previously worked at [imgix](https://www.imgix.com), a real-time image processing startup. With a simple URL API, images are fetched and transformed in real-time then served anywhere in the world via CDN. Our stack was fairly complex ([as previously described](https://stackshare.io/imgix/how-imgix-built-a-stack-to-serve-100000-images-per-second)), but in a nutshell, our infrastructure comprised of a load balancing and distribution layer which worked in tandem with the origin fetching layer, the origin caching layer, the image processing layer and the content delivery layer.

![](https://cdn-images-1.medium.com/max/5464/1*n-5ZgEH5KyqBxyBWDwWkzA.png)

At the heart of our load balancing layer was a service called Spillway, which served as both a reverse proxy as well as a request broker. Spillway was a purely internal service; at the edge we ran nginx and HAProxy, so Spillway wasn’t quite built to terminate TLS or perform any of the other myriad functionalities that’s typically within the purview of an edge proxy.

Spillway comprised of two components — a frontend (called Spillway FE) and a broker. While originally both components lived in the same binary, somewhere down the road we’d decided to split them into separate binaries which were deployed together on the same host. This was primarily owing to the fact that the two components had varying performance profiles, the frontend being almost entirely CPU bound. The frontend’s responsibility was to perform some pre-processing on every request, including a pre-flight to our origin caching layer to ensure the image was cached inside our datacenter before the image transformation request could be farmed out to a worker.

At any given time, we had a fixed pool of (a dozen or so, if memory serves me right) workers that would be connected to a single Spillway broker. These workers were responsible for performing the actual image transformation (cropping, resizing, PDF processing, GIF rendering and so forth). The workers processed everything from several hundred page PDF files to GIFs with hundreds of frames to plain image files. Another idiosyncrasy of the worker was that while all of the networking was entirely asynchronous, the actual transformation on the GPU itself was not. Considering we were a real-time service, it was impossible to predict what our traffic pattern at any given moment might look like. This required our infrastructure to be capable of self-adapting to different shapes of incoming traffic without requiring any manual operator intervention.

Given the disparate and protean traffic patterns we often saw, it became a desideratum for the workers to be able to refuse to accept the incoming requests (even when they were perfectly “healthy”) if accepting the connection meant the worker risked getting overloaded. Every request to the worker carried some metadata about the nature of the request, which enabled the worker to determine whether or not it was in a position to service that request. Each worker maintained its own set of statistics about the requests that it was currently operating on. The worker used these statistics in conjunction with the request metadata and other heuristics such as its socket buffer size to determine whether or not it was well-poised to accept the incoming request. When a worker determined that it could not accept a request, it crafted a response not unlike HAProxy’s agent-check which informed its downstream (Spillway) of its health.

Spillway tracked the health of all the workers in the pool. Spillway would first try to dispatch a request three times in succession to different workers (preferring the workers which were likely to have the original image in their local filesystem and which weren’t overloaded), and if all the three workers happened to refuse to accept the request, the request would be queued in the in-memory broker. The broker maintained three forms of queues — a LIFO queue, a FIFO queue and a priority queue. If all three queues happened to be full, the broker would simply reject the request, allowing the client (HAProxy) to retry after a backoff period. Once a request was queued in any one of the three queues, any free worker would be able to pop the request off the queue and process it. There are further intricacies around *how* priorities were assigned to requests and how decisions around *which* of the three queues (LIFO, FIFO, priority-based) any particular request must be placed in were made, but these are out of the scope of this post.

This form of dynamic feedback loop was non-negotiable for the healthy operation of our service. The broker queue size (of all the three queues) was something we monitored very closely and one of our key Prometheus alerts was when the queue size exceeded a certain threshold (which happened pretty infrequently).

![[Image from my presentation on the Prometheus monitoring system at Google NYC in November 2016](https://speakerdeck.com/copyconstructor/prometheus-at-google-nyc-tech-talks-nov-2016?slide=39)](https://cdn-images-1.medium.com/max/2036/1*W5-DBGy1uDcvkW2YuW5FcQ.png)*[Image from my presentation on the Prometheus monitoring system at Google NYC in November 2016](https://speakerdeck.com/copyconstructor/prometheus-at-google-nyc-tech-talks-nov-2016?slide=39)*

![[Alert taken from my presentation on the Prometheus monitoring system at OSCON in May 2017](https://speakerdeck.com/copyconstructor/prometheus-a-whirlwind-tour)](https://cdn-images-1.medium.com/max/2150/1*w2UFXd8snu9vRtCp02fOtw.png)*[Alert taken from my presentation on the Prometheus monitoring system at OSCON in May 2017](https://speakerdeck.com/copyconstructor/prometheus-a-whirlwind-tour)*

Uber had an interesting post from earlier this year which shed light on their approach to implementing a quality-of-service based load shedding layer.
> Analyzing outages that occurred over a six-month period, we found that 28 percent could have been mitigated or avoided through [graceful degradation](https://en.wikipedia.org/wiki/Fault_tolerance).
> The three most frequent types of failures we observed were due to:
> — Inbound request pattern changes, including overload and bad actors
 — Resource exhaustion such as CPU, memory, io_loop, or networking resources
 — Dependency failures, including infrastructure, data store, and downstream services
> We implemented an overload detector inspired by the [CoDel](https://en.wikipedia.org/wiki/CoDel) algorithm. A lightweight request buffer (implemented by goroutine and [channels](https://www.sohamkamani.com/blog/2017/08/24/golang-channels-explained/)) is added for each enabled endpoint to monitor the latency between when requests are received from the caller and processing begins in the handler. Each queue monitors the minimum latency within a sliding time window, triggering an overload condition if latency goes over a configured threshold.

However, it’s important to remember that if the backpressure isn’t propagated all the way back the call chain, there will be some degree of queueing at some component of the distributed system. Google published an infamous article back in 2013 called [**The Tail at Scale](https://www.dropbox.com/s/vd3divhvrkjqdv7/longtail.pdf?dl=0)**, which touched upon several causes of latency variability in systems with large fan-outs (queueing being an important one), as well as several neat techniques (often involving redundant requests) to mitigate this variability.

![](https://cdn-images-1.medium.com/max/4096/1*sNVlACQ4XPOcSWEapluykQ.jpeg)

Managing concurrency in a process in real-time forms the basis of distributed load shedding where each component in the system makes decisions based on local knowledge. While this helps with [scalability by obviating the need for centralized coordination](https://twitter.com/copyconstruct/status/1022671271631314944), it doesn’t entirely obviate the need for centralized rate limiting altogether.

![Myriad forms of rate limiting and load shedding techniques](https://cdn-images-1.medium.com/max/5464/1*dXw_5d2hFGp2Uteyc0LG3Q.png)*Myriad forms of rate limiting and load shedding techniques*

For those interested in learning more about formal performance modelling with queueing theory, I’d recommend watching the following talks:

1. [**Applied Performance Theory](https://www.infoq.com/presentations/little-usl-scalability-performance)**, [Kavya Joshi](https://twitter.com/kavya719) from QCon London2018

1. [**Queueing Theory in Practice: Performance Modeling for the Working Engineer](https://www.youtube.com/watch?v=yf6wSsOFqdI)**, [Eben Freeman](https://twitter.com/_emfree_) from LISA 2017

1. [**Stop Rate Limiting — Capacity Planning Done Right](https://www.youtube.com/watch?v=m64SWl9bfvk)**, [Jon Moore](https://twitter.com/jon_moore) from Strangeloop 2017

1. [**Predictive Load Balancing: Unfair but Faster and More Robust](https://www.youtube.com/watch?v=6NdxUY1La2I)**, [Steve Gury](https://twitter.com/stevegury) from Strangeloop 2017

1. The chapters on [**Handling Overload](https://landing.google.com/sre/book/chapters/handling-overload.html)** and [**Addressing Cascading Failures ](https://landing.google.com/sre/book/chapters/addressing-cascading-failures.html)**from the [**SRE Book](https://landing.google.com/sre/book.html)**

### Conclusion

Control loops and backpressure are already a solved problem in protocols like TCP/IP (where [congestion control algorithms](https://en.wikipedia.org/wiki/TCP_congestion_control) depend on load inference), IP [ECN](https://en.wikipedia.org/wiki/Explicit_Congestion_Notification) (which is an explicit mechanism to determine load, or near load), and Ethernet, with the effects of things like [PAUSE frames](https://en.wikipedia.org/wiki/Ethernet_flow_control).

Coarse-grained health checks might be sufficient for orchestration systems, but prove to be inadequate to ensure quality-of-service and prevent cascading failures in distributed systems. Load balancers need application level visibility in order to successfully and accurately propagate backpressure to clients. It’s impossible for a service to degrade gracefully if it’s not possible to determine its health at any given time accurately. Without timely and sufficient backpressure, services can quickly descend into the quicksands of failure.
