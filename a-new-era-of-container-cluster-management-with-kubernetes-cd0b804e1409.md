
# A New Era of Container Cluster Management with Kubernetes

A New Era of Container Cluster Management with Kubernetes

### **How Kubernetes was born at Google**

### Introduction

Almost everything at Google has been running on containers for more than a decade [1]. Google identified the importance of using Linux containers for efficiently utilizing underlying infrastructure more than virtual machines at very early stages. They contributed to the Linux kernel for implementing container technologies such as cgroups, namespaces and even implemented their own container stack ([lmctfy](http://www.linuxplumbersconf.org/2013/ocw/system/presentations/1239/original/lmctfy%20%281%29.pdf)). At the moment Google is spinning nearly two billion container instances a week across their global data centers [6]. According to a Google network engineer [7], these data centers are having a capacity of nearly 75,000 machines with over 1 petabit per second (which is around [125,000 gigabytes per second](https://www.google.lk/search?q=1%20petabit%20per%20second)) bandwidth in each.

### Babysitter & Global Work Queue: The Inception

Google has been using containers at such massive scale with the help of several container cluster management systems over the years. Initially, two separate systems called Babysitter and the Global Work Queue [4] were developed. **Babysitter was designed to run** **long running services** and **Global Work Queue was designed for executing** **batch jobs**. These are the main two types of tasks needed for implementing most of the software applications. Later on, Google replaced those two systems with Borg with the intention of introducing a more unified container cluster manager.

### Borg: Google’s First Unified Container Cluster Manager

![Figure 1: Borg Architecture, source: [2]](https://cdn-images-1.medium.com/max/3348/1*aXe-eoK8w9x9xe8xHG6YuA.png)*Figure 1: Borg Architecture, source: [2]*

Borg is Google first proprietary, unified container cluster manager. It includes a set of master nodes, a set of slave nodes, a CLI, and a web-based UI. All of the server components have been written in C++. The master provides an API to let the CLI, UI and other external systems to communicate with the cluster manager. In each slave node, an agent component called borglet is installed for managing containers, networking & routing. The scheduler takes container scheduling decisions based on the resource availability of the slave nodes and executes those via borglets. The state of the system is stored in Paxos, a distributed, highly available persistent storage.

Borg manages containers in a data center as a collection of cells. A cell is a collection of machines that managed as a unit with an average cell size of around 10,000 machines [2]. Each cell would belong to a single cluster. On high-level, Borg has been implemented with following cluster management features:

**Cluster Management Features of Borg [4]**

* Naming and service discovery (using Borg Name Service)

* Application-aware load balancing

* Horizontal and vertical autoscaling

* Rollout tools for deploying software/configuration updates

* Workflow tools for running multi-job analysis pipelines with interdependencies between the stages

* Monitoring tools to gather information about containers, aggregate, present on dashboards, and trigger alerts

### Omega: Google’s Second Generation Container Cluster Manager

![Figure 2: Omega Architecture, source: [3]](https://cdn-images-1.medium.com/max/3200/1*nXqqn0CynNusW2eGFpdZKw.png)*Figure 2: Omega Architecture, source: [3]*

Omega is Google’s second generation, proprietary container cluster manager. It was designed and built ground up, with the intention of improving the engineering and the architecture of the Borg system. It took almost all the container cluster management features of Borg and implemented those slightly differently for improving performance. Mainly it has improved two aspects of Borg; the way the cell state was persisted and the core scheduling architecture.

**External Cell State Persistence Model**

![Figure 3: Three different cell state persistent models analyzed by Google, source: [3]](https://cdn-images-1.medium.com/max/2000/1*jVE8aiwtqusL6pNQrfVf8A.png)*Figure 3: Three different cell state persistent models analyzed by Google, source: [3]*

Borg used a monolithic persistent model for storing the cell state inside the master. In Omega, it was moved out and managed as a separate centralized Paxos-based transactional store. This model allowed cluster manager features to be decomposed into separate components and interact with the persistent storage as peers [4]. This complies with the [microservices architecture](http://martinfowler.com/articles/microservices.html). Overall it provides a much efficient way of executing cluster manager component logic on the cell state, such as taking scheduling decisions as those can be executed in parallel with required level of concurrency and transaction management.

**Refined Scheduling Architecture**

![Figure 4: How schedulers’ busyness get reduced when cell state is shared, source: [3]](https://cdn-images-1.medium.com/max/2000/1*N3QT0Kj6yatmV91x6Fg94w.png)*Figure 4: How schedulers’ busyness get reduced when cell state is shared, source: [3]*

![Figure 5: How scheduler’s job wait time get reduced when cell state, source: [3]](https://cdn-images-1.medium.com/max/2000/1*H6yzJjz9Dg77Tgu1xXChTQ.png)*Figure 5: How scheduler’s job wait time get reduced when cell state, source: [3]*

Introduction of the external cell state persistent model allowed multiple schedulers run against the same cell state with optimistic concurrency controls. Google has analyzed above approaches in research paper [3] and found that shared cell state model allows optimizing scheduler busyness and job wait time significantly.

### Kubernetes: The Next Generation, General Purpose Container Cluster Manager

![Figure 6: Kubernetes high-level architecture, source: [http://kubernetes.io/](http://kubernetes.io/)](https://cdn-images-1.medium.com/max/2000/1*g7dba55qUPzoyh85H_iTcg.png)*Figure 6: Kubernetes high-level architecture, source: [http://kubernetes.io/](http://kubernetes.io/)*

In year 2014 Google started Kubernetes project with the intention of implementing a general purpose, open source container cluster management system by incorporating the experience they had with Borg and Omega systems. The name Kubernetes originates from Greek, meaning “helmsman” or “pilot”, and is the root of “governor” and “cybernetic” [8]. It took the architectural concepts from Omega such as master-slave separation, shared persistent storage, multiple schedulers, agent component in slave nodes, dynamic DNS management, etc and implemented those using latest technologies.

![Figure 7: Kuberentes architecture, source: [9]](https://cdn-images-1.medium.com/max/2000/1*HUq6gM5zKqE1OxaBH2Z_QQ.png)*Figure 7: Kuberentes architecture, source: [9]*

The entire system was written ground up, using Google’s latest system programming language Go. It reused existing open source technologies such as [**etcd](https://github.com/coreos/etcd)**; a distributed, reliable key/value store for centralized persistence storage, **f[lannel](https://github.com/coreos/flannel)** for implementing the overlay network, **cAdvisor** for monitoring resource usage in slave nodes, [**InfluxDB](https://influxdata.com/time-series-platform/influxdb/)** for persisting resource usage statistics, [**Grafana](http://grafana.org/)** for implementing monitoring dashboards, etc. The project is lead by Google, RedHat & CoreOS with the involvement of many other large organizations. At this moment, it has around 29,000 commits, 46 branches, 122 releases with around 800 contributors.

Kubernetes has implemented almost all the container cluster management features of Omega and added more advanced features with best-of-breed ideas and practices from the community:

**Key Features of Kubernetes [10]**

* Automatic binpacking (Mix critical and best-effort workloads optimal resource usage)

* Self-healing

* Horizontal manual/auto-scaling

* Service discovery & load balancing

* Automated rollouts and rollbacks

* Secret and configuration management

* Storage orchestration

* Batch execution

**Multi-Region/Cloud Deployments with Ubernetes**

![Figure 8: Ubernetes architecture, source: Kubernetes Github docs](https://cdn-images-1.medium.com/max/3012/1*6HnNrnpTcEFcZ40zBlcJKg.png)*Figure 8: Ubernetes architecture, source: Kubernetes Github docs*

Another important aspect of cloud computing is implementing multi-region/cloud deployments for high availability and distributed processing. Kubernetes has addressed this feature by implementing a separate control plane called [Ubernetes](https://github.com/kubernetes/kubernetes/blob/8813c955182e3c9daae68a8257365e02cd871c65/release-0.19.0/docs/proposals/federation.md) on top of Kubernetes as shown in the above figure. Ubernetes exposes a separate API for managing such deployments with advanced cluster management features such as location affinity, cross-cluster scheduling, cross-cluster service discovery and cross-cluster migration.

### Conclusion

Google has been using containers in production for more than a decade. They used several systems for this over the years; Babysitter, Global Work Queue, Borg, and Omega. These systems have been running on massive scale at Google data centers with efficient resource utilization techniques. However, all of them were proprietary and specifically designed for Google infrastructure. Google started Kubernetes project in the year 2014 with the intention of implementing a general purpose container cluster manager by involving other interested organizations. Within this short period of time Kubernetes has evolved rapidly and has become the best container cluster manager in its class.

### References

[1] John Wilkes, Google, Cluster Management at Google: [https://www.usenix.org/cluster-management-google](https://www.usenix.org/cluster-management-google)

[2] Google, Large-scale cluster management at Google with Borg: [http://static.googleusercontent.com/media/research.google.com/en//pubs/archive/43438.pdf](http://static.googleusercontent.com/media/research.google.com/en//pubs/archive/43438.pdf)

[3] Google, Omega: flexible, scalable schedulers for large compute clusters: [http://static.googleusercontent.com/media/research.google.com/en//pubs/archive/41684.pdf](http://static.googleusercontent.com/media/research.google.com/en//pubs/archive/41684.pdf)

[4] Google, Borg, Omega & Kubernetes: [http://static.googleusercontent.com/media/research.google.com/en//pubs/archive/44843.pdf](http://static.googleusercontent.com/media/research.google.com/en//pubs/archive/44843.pdf)

[5] The Linux Foundation, Who Writes Linux 2015, [http://www.linuxfoundation.org/publications/linux-foundation/who-writes-linux-2015](http://www.linuxfoundation.org/publications/linux-foundation/who-writes-linux-2015)

[6] Google, Cloud Platform Blog, An update on container support on Google Cloud Platform: [https://cloudplatform.googleblog.com/2014/06/an-update-on-container-support-on-google-cloud-platform.html](https://cloudplatform.googleblog.com/2014/06/an-update-on-container-support-on-google-cloud-platform.html)

[7] Google, Google Cloud Platform, Google Data Center 360° Tour: [https://www.youtube.com/watch?v=zDAYZU4A3w0](https://www.youtube.com/watch?v=zDAYZU4A3w0)

[8] Kubernetes, Kubernetes Docs, What is Kubernetes? [http://kubernetes.io/docs/whatisk8s/](http://kubernetes.io/docs/whatisk8s/)

[9] David K. Rensin, Kubernetes, Scheduling the Future at Cloud Scale: [http://www.oreilly.com/webops-perf/free/kubernetes.csp](http://www.oreilly.com/webops-perf/free/kubernetes.csp)

[10] Kubernetes, Kubernetes Official Website: [http://kubernetes.io/](http://kubernetes.io/)
