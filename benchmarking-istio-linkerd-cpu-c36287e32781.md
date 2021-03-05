
# Benchmarking Istio & Linkerd CPU

Background

Here at [Shopify](https://www.shopify.ca), we’re working on deploying [Istio](https://istio.io) as our service mesh. We’re doing quite well, but are hitting a wall: **Cost**.

Istio’s [published benchmarks](https://istio.io/docs/concepts/performance-and-scalability/#cpu-and-memory) state:
> As of Istio 1.1, a proxy consumes about 0.6 vCPU per 1000 requests per second.

For our first edge in the service mesh (2 proxies on either side of the connection) we’re looking at 1,200 cores for the proxy alone, per million requests per second. Google’s pricing calculator estimates around $40/month/core for n1-standard-64 nodes, which puts this single edge at over $50k/month/1MM RPS.

[Ivan Sim](undefined) did a [great writeup](https://medium.com/@ihcsim/linkerd-2-0-and-istio-performance-benchmark-df290101c2bb) of service mesh latency last year and promised a followup with memory and CPU data, but couldn’t generate them:
> Looks like the values-istio-test.yaml is going to raise the CPU requests by quite a bit. If I’ve done my math correctly, it’s around 24 CPUs for the control plane and 0.5 CPU for each proxy. That’s more than my current personal account quota. I will re-run the tests once my request to increase my CPU quotas is approved.

I needed to see for myself if Istio was comparable to another open-source service mesh: [Linkerd](https://linkerd.io).

## Installing the Service Meshes

First thing, I installed [SuperGloo](https://supergloo.solo.io/) in the cluster:

<iframe src="https://medium.com/media/fd62a4b4a1a3c3b45752616d7185bfdc" frameborder=0></iframe>

I used SuperGloo because it was super simple to get both services meshes bootstrapped quickly, with almost no effort on my part. We’re not using SuperGloo in production, but it was perfect for a task like this. It was literally two commands per mesh. I used two clusters for isolation— one for Istio, and one for Linkerd.

The experiment was run on Google Kubernetes Engine. I used Kubernetes 1.12.7-gke.7 and used a node pool with n1-standard-4 nodes with node autoscaling enabled (min 4, max 16).

I then installed both service meshes using the command line tool.
First Linkerd:

<iframe src="https://medium.com/media/6a69429048a8a57a676e61f7aca02830" frameborder=0></iframe>

Then Istio:

<iframe src="https://medium.com/media/44fdd954d74f8fe1c7f345b6dad51dd9" frameborder=0></iframe>

After a few minutes of CrashLooping, the control planes stabilized.

(*Note: SuperGloo currently only supports Istio 1.0.x. This experiment was re-tested with Istio 1.1.3 with no measurable difference.*)

## Set up Istio Auto Injection

To get Istio to install the Envoy sidecar, we use the sidecar injector, which is a MutatingAdmissionWebhook. It’s out of the scope of this article, but in a nutshell, a controller watches all new pod admissions and dynamically adds the sidecar and the initContainer which does the iptables magic.

At Shopify, we wrote our own admission controller to do sidecar injection, but for the purposes of this benchmark, I used the one that ships with Istio. The default one does injection when the label istio-injection: enabled is present on the namespace:

<iframe src="https://medium.com/media/be4da67ebe3fa4b58f0c4070d687f6d8" frameborder=0></iframe>

## Set up Linkerd Auto Injection

To set up Linkerd sidecar injection, we use annotations (which I added manually with kubectl edit):

    metadata:
      annotations:
        linkerd.io/inject: enabled

<iframe src="https://medium.com/media/8b7ead5f6bd615b48a2786fb1c06ca20" frameborder=0></iframe>

## The Istio Resiliency Simulator (IRS)

We developed the Istio Resiliency Simulator to try out some traffic scenarios that are unique to Shopify. Specifically, we wanted something that we could use to create an arbitrary topology to represent a specific portion of our service graph that was dynamically configurable to simulate specific workloads.

The flash sale is a problem that plagues Shopify’s infrastructure. Compounding that is the fact that Shopify actually [encourages merchants to have more flash sales](https://www.shopify.com/enterprise/flash-sale). For our larger customers, we sometimes get advance warning of a scheduled flash sale. For others, they come completely by surprise and at all hours of the day & night.

We wanted IRS to be able to run “workflows” that represented topologies and workloads that we’d seen cripple Shopify’s infrastructure in the past. One of the main reasons we’re pursuing a service mesh is to deploy reliability and resiliency features at the network level, and proving that it would have been effective at mitigating past service disruptions is a big part of that.

The core of IRS is a worker which acts as a node in a service mesh. The worker can be configured statically at startup, or dynamically via a REST API. We use the dynamic nature of the workers to create workflows as regression tests.

An example of a workflow might be:

* Start 10 servers, as service bar that returns 200/OK after 100ns

* Start 10 clients, sending 100 RPS each to bar

* Every 10 seconds, take down 1 server, monitoring 5xx levels at the client

At the end of the workflow, we can examine logs & metrics to determine a pass/fail for the test. In this way, we can both learn about the performance of our service mesh and also regression test our assumptions about resiliency.

(*Note: We’re thinking of open-sourcing IRS, but are not ready to do so right now.*)

## IRS for Service Mesh Benchmarking

For this purpose, we set up some IRS workers as follows:

* irs-client-loadgen: 3 replicas that send 100 RPS each to irs-client

* irs-client: 3 replicas that receive a request, waits 100ms and forwards the request to irs-server

* irs-server: 3 replicas that return 200/OK after 100ms

With this setup, we can measure a steady stream of traffic between 9 endpoints. The sidecars on irs-client-loadgen and irs-server receive a total of 100 RPS each and irs-client sees 200 RPS (inbound & outbound).

We monitor the resource usage via [DataDog](https://www.datadoghq.com), since we don’t maintain a Prometheus cluster.

## The Results

### Control Planes

First, we looked at the control plane CPU usage.

![Linkerd control plane: ~22 mcores](https://cdn-images-1.medium.com/max/3328/1*8v-xNiiK7fxQOdsnpH7CaQ.png)*Linkerd control plane: ~22 mcores*

![Istio control plane: ~750 mcores](https://cdn-images-1.medium.com/max/3328/1*RZUdBh6dEHTezm9u7UYOiA.png)*Istio control plane: ~750 mcores*

The Istio control plane uses **~35x more CPU** than Linkerd’s. Admittedly this is an out-of-the-box installation, and the bulk of the Istio CPU usage is from the istio-telemetry deployment, which can be turned off (at the cost of features). Removing the mixer still leaves over 100 mcores, which is still **4x more CPU **than Linkerd.

### Sidecar Proxies

Next, we looked at the sidecar proxy usage. This should scale linearly with your request rate, but there is some overhead for each sidecar which will affect the shape of the curve.

![Linkerd: ~100 mcore for irs-client, ~50 mcore for irs-client-loadgen](https://cdn-images-1.medium.com/max/3332/1*WLo9qbiJmCG2S46jPGuRtw.png)*Linkerd: ~100 mcore for irs-client, ~50 mcore for irs-client-loadgen*

These results made sense, since the client proxy receives 2x the traffic of the loadgen proxy: for every outbound request from the loadgen, the client gets one inbound and one outbound.

![Istio/Envoy: ~155 mcore for irs-client, ~75 mcore for irs-client-loadgen](https://cdn-images-1.medium.com/max/3332/1*0Q0EMOCB1DFbTW1GBkdHlQ.png)*Istio/Envoy: ~155 mcore for irs-client, ~75 mcore for irs-client-loadgen*

We see the same shape of results for the Istio sidecars.

Overall, though, the Istio/Envoy proxies use **~50% more CPU** than Linkerd.

We see the same pattern on the server side:

![Linkerd: ~50 mcores for irs-server](https://cdn-images-1.medium.com/max/3328/1*PhXTsL5UCgvrLcoXeTQ23g.png)*Linkerd: ~50 mcores for irs-server*

![Istio/Envoy: ~80 mcores for irs-server](https://cdn-images-1.medium.com/max/3324/1*deLq2Fg8JYxxc6izSgIxfQ.png)*Istio/Envoy: ~80 mcores for irs-server*

On the server side, the Istio/Envoy sidecar uses **~60% more CPU** than Linkerd.

## Conclusion

Istio’s Envoy proxy uses more than 50% more CPU than Linkerd’s, for this synthetic workload. Linkerd’s control plane uses a tiny fraction of Istio’s, especially when considering the “core” components.

We’re still trying to figure out how to mitigate some of this CPU overhead — if you have some insight or ideas, we’d love to hear from you.
