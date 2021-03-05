
# Benchmarking Istio & Linkerd CPU at Scale

Background

I recently [published an article](https://medium.com/@michael_87395/benchmarking-istio-linkerd-cpu-c36287e32781) comparing Istio & Linkerd’s CPU usage. While the content of the article was sound, there were some reasonable criticisms that I felt needed addressing.

The most compelling one, however, was that the original benchmark was run on a relatively small cluster and gave the misleading impression that some of the results would increase linearly with scale.

The conclusion there was:
> Istio’s Envoy proxy uses more than 50% more CPU than Linkerd’s, for this synthetic workload. Linkerd’s control plane uses a tiny fraction of Istio’s, especially when considering the “core” components.

The question is: does this result change with scale?

## The Setup

Last time, I used SuperGloo to deploy the control plane. Unfortuntately, at the time SuperGloo only supported the 1.0.x series of Istio. I re-ran those tests and found that 1.1.3 didn’t produce significantly different results, but I still wanted more control over the installation, so I decided to do it from scratch.

Here’s roughly how I did it:

### Create the Cluster

At [Shopify](https://www.shopify.ca), we’re 100% on Google Cloud, so I created a perf-istio cluster in our Istio GCP project. I created two separate node pools: One for kube-system and istio-system namespaces, and another for the IRS workers.

(P.S. For more on the Istio Resiliency Simulator, see the previous article.)
(P.P.S. We’re now actively looking at open sourcing it, so stay tuned for that.)

The default-pool was multi-zone, a total of 4 n1-standard-4 nodes.
The irs-node-pool was also multi-zone, autoscaled, but a total of ~64 n1-standard-8 nodes for the duration of the tests.

In the experiment, we used 600 IRS workers and gave them 250 mcores each. The proxy was set at 250 mcores as well, so we needed half a core per worker. Since we were using 8 core machines, and had some DaemonSets taking up a little over 1 core per machine, that meant we needed >50 nodes to run the test.

A note: This test was *expensive* to run. I’m grateful that I have the resources at my disposal to be able to run these types of tests, but it’s not lost on me that not everyone is able to do so.

### Installing Istio

Create the namespace:
kubectl create namespace istio-system

Install the CRDs:
helm template install/kubernetes/helm/istio-init --name=istio-init --namespace=istio-system | kubectl apply -f -

Wait a few moments for the CRDs to install (kubectl -n istio-system get pods) and then install Istio:
helm template install/kubernetes/helm/istio --name=istio --namespace=istio-system --values=/path/to/values.yaml | kubectl apply -f -

<iframe src="https://medium.com/media/1c1b5eb0ee60ff4d45374c4742aad39d" frameborder=0></iframe>

I’ve included a diff of my values.yaml versus the Istio default. It’s basically off-the-shelf, but with mTLS enabled, and Prometheus/Grafana & Mixer disabled.

### Wait, you disabled Mixer???

Yes, this is a contentious point. The roadmap for Istio includes the complete deprecation of the Mixer as a separate component in the control plane. Instead, portions of the mixer will either go away and/or be replaced with components in Envoy.

According to the Istio performance team, lots of CPU time is wasted in the proxies sending telemetry data back to the mixer for no appreciable benefit. In fact, we can instrument DataDog to scrape the Prometheus endpoints on the proxies themselves to get most of the telemetry data that Mixer was previously supplying.

The most glaring difference, however, is that Mixer uses Pilot data to convert endpoint-centric telemetry data into service-centric data. Put simply, the proxy can deliver us telemetry tagged on endpoint IP addresses, but we need that tagged by cluster/service names.

This is definitely not at feature parity and is more forward looking, from a performance standpoint. I trust the Istio team will get back to feature parity with the deprecation of Mixer, so I’m comfortable doing the experiment this way.

### Pinning IRS workers

To get the IRS workers to use a different node pool, I used this annotation in the Deployment spec template:

nodeSelector:
 cloud.google.com/gke-nodepool: irs-node-pool

For the control plane(s), I used this script:

<iframe src="https://medium.com/media/f63aaa6cccaf2d6fac0c804adb7bb600" frameborder=0></iframe>

### Installing Linkerd

I installed Linkerd 2.3.0:
curl -sL [https://run.linkerd.io/install](https://run.linkerd.io/install) | sh

Then I installed Linkerd in the cluster, with proxy auto-injection enabled:
linkerd install --proxy-auto-inject --proxy-cpu-request=300m --proxy-cpu-limit=300m --proxy-memory-request=256Mi --proxy-memory-limit=256Mi |kubectl apply -f -

We’re using 300m CPU request/limit to get [Guaranteed QoS](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/#create-a-pod-that-gets-assigned-a-qos-class-of-guaranteed) for IRS. A previous run of this experiment showed that 250m was not enough for Linkerd, and caused a lot of noise in the results at the right side of the graphs.

## Results

### Method

![IRS Request Rate for the experiment](https://cdn-images-1.medium.com/max/5760/1*_Q6FuVmX_xaXUN7nUe0_IQ.png)*IRS Request Rate for the experiment*

I ran the experiment in two identical clusters, starting at 10 replicas per deployment (1,000 mesh-wide RPS) and stepping up 10 replicas every 20 minutes or so.

### Total CPU Usage

![Total CPU Usage](https://cdn-images-1.medium.com/max/5760/1*9irKpB-ZEkSLMYbeZ1W_Vg.png)*Total CPU Usage*

At 10 replicas (30 endpoints, 1,000 mesh-wide RPS) **Istio used 2.6 cores** and **Linkerd in used 3.4 cores**, approximately 30% more.

At 200 replicas (600 endpoints, 20,000 mesh-wide RPS), **Istio used 51 cores** and **Linkerd used 68 cores**, approximately 33% more.

As expected, the control plane made up a small fraction of the total CPU usage, when the proxies’ CPU usage is considered. It’s worth noting that Istio’s peak CPU usage was actually 54 cores, which is ~6% higher than the final steady-state usage.

### Total Memory Usage

![Total Memory Usage](https://cdn-images-1.medium.com/max/5760/1*db0-34qD4td4RLdmcDYeRw.png)*Total Memory Usage*

On the memory side, Istio does use more: **3x more** at low endpoint counts and **~45% more** when there are many endpoints.

Of note: I did not use Sidecar resources in this test. I don’t know how much it would have helped with memory/CPU usage in this test, given that there were only three services, but it might have been significant.

### Control Plane CPU Usage

![Control Plane CPU Usage](https://cdn-images-1.medium.com/max/5760/1*oRRe6_Fv04LdwBJDMtK7iA.png)*Control Plane CPU Usage*

The graphs above are kind of silly when the y-axes are matched, but it shows pretty clearly that the Istio control plane uses way more CPU than Linkerd’s. However, as shown above, the proxies’ CPU dominates, so it’s not that big a deal, at scale.

Istio’s peak CPU usage for the control plane is almost 2 cores. [Horizontal Pod Autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) is enabled and Pilot is horizontally scalable so this works as intended.

![Linkerd Control Plane CPU Usage](https://cdn-images-1.medium.com/max/2860/1*zSkiBwTpEpGNwJVLqg2Cvw.png)*Linkerd Control Plane CPU Usage*

For the record, this is what Linkerd’s control plane CPU usage looks like. Note that the y-axis is in *mcores* in this graph…

### Proxy CPU Usage

![Average Proxy CPU Usage](https://cdn-images-1.medium.com/max/5760/1*7Dk9A4fKbxOgIawbX3Wu6A.png)*Average Proxy CPU Usage*

It should be obvious from the totals, but the Linkerd proxy uses more CPU on average. We can see that there’s a slight upward trend as the number of endpoints increases, but the CPU usage overall is dictated by traffic throughput, not mesh size or mesh-wide RPS.

Istio’s proxy uses ~45/78/45 mcores for the loadgen/client/server, whereas Linkerd’s uses ~52/105/62 mcores (**15%/35%/35% more**).

### Latency

![IRS Max Client-Side Latency](https://cdn-images-1.medium.com/max/5760/1*ukADu9QvudnqCMeBG6zRvQ.png)*IRS Max Client-Side Latency*

The IRS client holds onto the request for 100ms, as does the IRS server, so any value over 200ms is network and proxy latency.

I haven’t done an in-depth analysis on this section, but a quick eyeball suggests that the overall P95+ latency through Istio is higher than Linkerd.

Istio’s P95 latency starts suffering at the very end, once we hit 600 endpoints. Linkerd’s P95 latency starts suffering at around 400 endpoints and at 500 endpoints the P50 latency is affected.

### Reliability

![](https://cdn-images-1.medium.com/max/5760/1*xup_bWKVR6RrfLr-XpANGA.png)

Now, there’s a lot going on in this test, but it’s interesting to note that the IRS client saw 124,710 504's during the test, which is 0.033% of total requests. It also saw 61,660 failed requests, which might have been timeouts.

Linkerd only saw 18k failed requests, but also saw a handful of 500's.

### A Note About Throttling

It took a while to get to the point where I could feel comfortable publishing these results. I was noticing some serious CFS throttling happening that was affecting the ability of the IRS workers to maintain a constant tick rate for the requests (and therefore achieve a stable RPS). Throttling in the Istio proxy was also causing increased latency to the point of timing out requests altogether.

In fact, even with a timeout of 500ms in the worker, I was seeing max latency numbers of 900ms+ for failed requests, which means that the worker couldn’t even get enough CPU time to stop the timer.

I did some very fine grained monitoring of Docker throttling metrics, CFS throttling metrics and overall node CPU usage and am pretty confident that even though there is some throttling still happening, it’s not affecting these results in a measurable way.

## Conclusion

While my previous article came to the conclusion that the Istio control plane used 10x more CPU than Linkerd’s, and that Envoy used 50% more CPU than linkerd-proxy, that result doesn’t translate to how one would run a service mesh like this in production.

When run at a reasonable scale, we see that the control plane’s overall CPU usage is a small fraction of the total and that the proxies dominate. At the peak of this test, Linkerd was using ~68 cores, **approximately 33% more than Istio’s** ~51 cores.

Note: *This result depends on disabling Mixer, however, and getting telemetry data some other way. Since we use DataDog AutoDiscovery to scrape Prometheus metrics from the Envoy proxies, this works for us but is not at 100% feature-parity.*
