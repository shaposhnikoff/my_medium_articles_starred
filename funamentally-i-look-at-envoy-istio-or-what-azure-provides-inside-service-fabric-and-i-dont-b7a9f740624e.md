
# Funamentally, I look at Envoy + Istio, or what Azure provides inside Service Fabric, and I don’t…

Just to be clear, using an API Gateway per microservice does not provide the features of a service mesh today anyway, and I also don’t want API Gateway to be the focus of such features.

There are two reasons I don’t want to use an API Gateway in front of every microservice. The first is performance: it introduces latency without providing much inherent benefit. The reason there is little inherent benefit is the second reason: that *within* my application, everything understands AWS resources anyway, so the benefit of an HTTP API is reduced. It’d be great to have a way to communicate the expected input format for a Lambda, but for internal invocations that’s more documentation than anything else.

I’ve already accepted a large amount of vendor lock-in by virtue of building a serverless application dependent on the FaaS and SaaS services in the platform — and, as I’ve argued, this lock-in should not be seen as troublesome. But the lock-in means that inside the system, I don’t need to abstract over the details of the platform.

API Gateways, to me, are useful exactly as *gateways*: entry points for external clients to access the system, whether that’s an IoT device, a mobile app, a web client, or a 3rd party application. And for those clients, I want to present a single endpoint to access. If every microservice is fronted by its own API Gateway, that means putting another API Gateway in front of those, adding additional latency.
