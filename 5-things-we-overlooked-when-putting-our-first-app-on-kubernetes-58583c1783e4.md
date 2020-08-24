Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m36[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m19[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m14[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m16[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m16[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m12[39m, end: [33m27[39m }

# 5 Things We Overlooked When Putting Our First App on Kubernetes



From my experience, it seems that most people throw an application onto [Kubernetes](https://kubernetes.io/) (either with [Helm](https://helm.sh/) or manually) and then think they can call it a day. Through our use of Kubernetes at [GumGum](http://gumgum.com), we have encountered a number of “gotchas” that we wanted to list here to help you cover your bases before launching your app on Kubernetes.

## Step One: Configure Pod Requests and Limits

We will start with configuring a clean environment in which our pods can run. Kubernetes does a fantastic job at handling pod scheduling and failure states. One thing we learned, however, is that the [Kubernetes](https://kubernetes.io/) scheduler can sometimes have a hard time placing pods if it can’t gauge how many resources that pod needs to run successfully. This is where resource requests and limits come in. There is much debate on the best approach to setting app requests and limits; it really can feel like more of an art than a science. Here is how we think about them internally here at [GumGum](https://gumgum.com/):

**Pod Requests:** This is the primary value used by the scheduler to optimally place pods. From the [Kubernetes documentation](https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/):
> *The filtering step finds the set of Nodes where it’s feasible to schedule the Pod. For example, the PodFitsResources filter checks whether a candidate Node has enough available resource to meet a Pod’s specific resource requests.*

Internally we use app requests in this vein; we set them to be a good estimate of what the application *actually* needs to run a normal workload. This way, the scheduler will be able to place the nodes realistically. Initially, we wanted to set requests higher to ensure that each pod got plenty of resources, but when we did that, we noticed scheduling times greatly increased, with some pods failing to be scheduled entirely. This was similar to the behavior we observed when we had no resource requests specified at all. In this case, the scheduler would often “evict” pods and fail to re-schedule them due to the fact that the control plane had no idea how many resources the app would need — which is a key component of the scheduling algorithm.

**Pod Limits:** This is the somewhat more straight-forwards limit for the pod; it represents the maximum resources the cluster should allow the container to ever consume. [Again, from the official documentation](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)…
> *If you set a memory limit of 4GiB for that Container, the kubelet (and container runtime ) enforce the limit. The runtime prevents the container from using more than the configured resource limit. For example: when a process in the container tries to consume more than the allowed amount of memory, the system kernel terminates the process that attempted the allocation, with an out of memory (OOM) error.*

A container can always use more resources than its request, but never more than its limit. This is a tricky value to set correctly but very important. You ideally want to let your pod’s resource requirements change during the lifecycle of the process without interfering with other processes on the system — this is the goal of limits. Unfortunately, I cannot give you specifics on which values to set, but we followed this process to keep them tuned:

1. Using a load-testing tool, we simulated a base level of traffic and observed the resource usage of the pod (memory and CPU)

1. We set the pod requests arbitrarily low (while keeping pod resource limits at roughly 5x the request value) and observed behavior. When the requests were too low, the process would be unable to start, often throwing cryptic Go runtime errors.

One point I want to hone in on, is that higher resource limits lead to harder pod scheduling; since it needs a target node with sufficient resources available. Think of the case where you might have a lightweight webserver process running with a very high resource limit (4GB of memory, for example). This is a process you would likely need to scale horizontally, and each new pod would need to be scheduled on a node with at least 4GB of memory available. If that node does not exist, your cluster needs to introduce a new node to handle that pod, which can take time to spin up. It’s important to try to get the smallest “bounds” between resource requests and limits as possible to ensure rapid and smooth scaling.

## Step Two: Configure Liveness and Readiness Probes

Another subtle topic that is often debated in the Kubernetes community. It’s important to have a good grasp on Liveness and Readiness probes because they provide a mechanism to run fault-tolerant software and minimize downtime. They can, however, impose a harsh performance hit on your application if not configured correctly. Below is a rundown of both probes and how to reason about them:

**Liveness Probe:** “Indicates whether the Container is running. If the liveness probe fails, the kubelet kills the Container, and the Container is subjected to its restart policy. If a Container does not provide a liveness probe, the default state is Success.” — [Kubernetes Docs](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)

Liveness probes need to be cheap since they are run frequently and should inform Kubernetes if the application is running. Note that if you set this to run every second, that will impose 1 request/second of additional traffic, so take those additional resources needed to handle that request into account. Here at GumGum, our Liveness probes are set to respond to when the main components of the application are running, but data (from a remote database, or cache, for example) might not be fully available yet. An example of how we typically implement this is by setting up a specific “health” endpoint in our apps that simply returns a 200 response code. This is a good indication that your process has started and can handle requests (but not yet traffic).

**Readiness Probe:** “Indicates whether the Container is ready to service requests. If the readiness probe fails, the endpoints controller removes the Pod’s IP address from the endpoints of all Services that match the Pod.”

Readiness probes are much more expensive to run since they should hit the backend in a way that indicates the full application is running and ready to receive requests. There is a lot of debate in the community as to whether you should hit your database or not. Again, considering the overhead it does cause (these checks run frequently, but that can be adjusted), we have decided that for some of our apps, we will only “serve traffic” once we can return records from the database. By using well-thought-out readiness probes, we have been able to achieve a much higher level of availability as well as zero downtime deployments.

If you do decide a request to the database makes sense for your application’s readiness probe, make sure you keep that query as cheap as possible. For example…

SELECT small_item FROM table LIMIT 1

Here is an example of how we configure these two values in Kubernetes:

    livenessProbe:  
      httpGet:    
        path: /api/liveness     
        port: http  
    readinessProbe:   
      httpGet:     
        path: /api/readiness     
        port: http  periodSeconds: 2

Here are some additional configuration options you can add:

* initialDelaySeconds - Number of seconds after the container starts for the probes to begin running

* periodSeconds - Wait interval between runs of the probe

* timeoutSeconds - Number of seconds after which the pod is considered to be in a failing state. A traditional timeout.

* failureThreshold - How many times the pod must fail the probe before a restart signal is sent to the pod

* successThreshold - How many times the probe must succeed before going into a ready state (after a failure event when the pod starts or recovers)

## Step Three: Set up default pod network policies

Kubernetes uses a “flat” networking topography and by default, all pods can communicate directly with one another. In some cases, this is not desirable or even necessary. A potential security concern would be a single vulnerable application, if exploited, could provide the attacker with full access to sending traffic to all pods on the network. Like in many security realms, the policy of least access applies here as well, ideally network policies would be created to explicitly specify which pod-to-pod connections are allowed and which are not.

The following, for example, is a simple policy that would deny all ingress traffic for a specific namespace:

    --- 
    apiVersion: networking.k8s.io/v1 
    kind: NetworkPolicy 
    metadata:   
      name: default-deny-ingress 
    spec:   
      podSelector: {}   
      policyTypes:   
        - Ingress

Below is a visualization of this configuration:

![](https://cdn-images-1.medium.com/max/2072/1*-eiVw43azgzYzyN1th7cZg.gif)

More information [here](https://kubernetes.io/docs/concepts/services-networking/network-policies/).

## Step Four: Custom Behavior through Hooks and Init containers

One of our main goals for our Kubernetes system, was trying to provide zero-downtime deployments to developers out-of-the-box as much as possible. This is difficult due to the variety of ways applications shut themselves down and clean up utilized resources. One application we had particular difficulties with was [Nginx](http://nginx.org/). We noticed that when we initiated a rolling deployment of those pods, active connections were being dropped before being successfully terminated. After extensive research online, it turns out that Kubernetes was not waiting for [Nginx](http://nginx.org/) to drain its connections before terminating the pod. Using a pre-stop hook, we were able to inject this functionality and achieved zero downtime with this change.

    lifecycle:  
      preStop:
        exec:
          command: ["/usr/local/bin/nginx-killer.sh"]

And here is nginx-killer.sh:

    #!/bin/bash

    sleep 3
    PID=$(cat /run/nginx.pid)
    nginx -s quit

    while [ -d /proc/$PID ]; do
        echo "Waiting while shutting down nginx..."
        sleep 10
    done

Another extremely helpful paradigm is using init containers to handle app-specific startup tasks. Certain popular Kubernetes projects also make use of init-containers, such as [Istio](https://istio.io/), to inject [Envoy](https://www.envoyproxy.io/) handling code onto the pod. This is especially helpful in the case where you have a heavy database migration process that needs to run before your application can start. You can also specify a higher resource limit for this process without requiring that limit for the main application.

Another common pattern is to grant secrets access to an init-container that exposes those credentials to the main pod; preventing unauthorized secret access from the main app pod itself. As always, [from the documentation:](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)
> *Init containers can securely run utilities or custom code that would otherwise make an app container image less secure. By keeping unnecessary tools separate you can limit the attack surface of your app container image.*

## Step Five: Kernel Tuning

Finally, leaving a more advanced technique for last. Kubernetes is an exceptionally flexible platform that aims to allow you to run your workloads how you see fit. At GumGum, we have a number of highly performant applications that have extremely demanding resource needs. After doing extensive load-testing, we discovered that one of our applications was struggling to meet the expected traffic load using default Kubernetes settings. However, Kubernetes allows us to run a privileged container that can modify kernel parameters that will only apply to the specific running pod. Below is an example we used to modify the maximum open connections the pod would allow:

    initContainers:
       - name: sysctl
          image: alpine:3.10
          securityContext:
              privileged: true
           command: ['sh', '-c', "sysctl -w net.core.somaxconn=32768"]

This is a more advanced technique that often is not necessary. If your application is struggling to stay running under heavy load, you may want to try tuning some of these parameters. More information about this process and values you can tweak can be found, as always, [in the official documentation](https://kubernetes.io/docs/tasks/administer-cluster/sysctl-cluster/).

## In Summary

While Kubernetes might seem like a ready “out of the box” solution, there are some key steps you need to take to ensure smooth operation of your applications. It’s important to follow a load-testing “loop” throughout the process of converting your application to run on Kubernetes; run your app, load test it, observe metrics and scaling behavior, tune your configuration based on that data, repeat. Be realistic about the traffic you expect, and push it past that limit as a way to see which components might break first. Using this iterative approach, you may find success using only a subset of these recommendations, or may need deeper tuning. Always be asking yourself these questions:

1. What is the resource footprint of my application and how will it change?

1. What are the realistic scaling requirements of this service? How much average traffic will it be expected to handle? What about peak traffic?

1. How frequently will we expect the service to need to horizontally scale? How quickly will we need these new pods to be able to accept traffic?

1. Are our pods terminating gracefully? Do they need to be? Can we achieve zero-downtime deployments?

1. How can I minimize my security risk and limit the “blast radius” of any compromised pods? Do any of my services have permissions or access they don’t require?

Kubernetes provides an incredible platform that allows you to utilize best practices to deploy thousands of services across a cluster. That being said, not all software is created equal. Sometimes your application may require a bit more work and thankfully Kubernetes provides us with the knobs to tune to achieve our desired technical goals. Using a combination of resource requests and limits, liveness and readiness checks, init-containers, network policies, and custom kernel tuning, I believe you can achieve great baseline performance along with resiliency and rapid scalability.
