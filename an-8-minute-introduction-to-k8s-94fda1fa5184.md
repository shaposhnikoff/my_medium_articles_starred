Unknown markup type 10 { type: [33m10[39m, start: [33m353[39m, end: [33m363[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m448[39m, end: [33m458[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m471[39m, end: [33m478[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m681[39m, end: [33m685[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m687[39m, end: [33m691[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m696[39m, end: [33m700[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m758[39m, end: [33m762[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m764[39m, end: [33m768[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m117[39m, end: [33m129[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m345[39m, end: [33m353[39m }

# An 8-minute introduction to K8S

Core concepts, features and building blocks

![](https://cdn-images-1.medium.com/max/10178/1*oRUsjY0iizMyIwyx8GpSeQ.jpeg)
> This is a summary post I wrote for myself after completing “[Kubernetes: Up and Running](https://www.amazon.com/Kubernetes-Running-Dive-Future-Infrastructure/dp/1491935677)”. I decided it would be a good way of forcing myself to review what I’ve read, and summerize it for others that look for the TL;DR, *as would I if I had it.*

Built on the shoulders of Google’s [Borg](https://kubernetes.io/blog/2015/04/borg-predecessor-to-kubernetes/), k8s (“Kubernetes”) is a container [orchestration system](https://www.mongodb.com/containers-and-orchestration-explained); a very powerful one. K8s and its entire ecosystem (tools, modules, add-ons etc) are written in Golang, making it essentially a collection of API-oriented, very fast binaries, that are well documented and easy to contribute to or build applications upon.

It has a few core concepts any dev / ops / interested reader should be familiar with to get the grasp of the system, its different abilities and understand why almost everyone is using it.

Before moving along, I’d like to mention k8s top friends (or rivals): ECS, Nomad and Mesos. ECS being AWS’s own solution to orchestration, which had recently introduced EKS — a managed k8s system on AWS. Both offer [FARGATE](https://aws.amazon.com/fargate/), that allows the user to forget about running physical resources.

While k8s is without a doubt the big winner in adoption numbers, which is a function of it being an open source system, that’s also available in its managed form on each of the three major cloud providers; it is nevertheless more complex and tangled than the others. K8s can handle almost any kind of containerized workload and has a lot a tricks down its sleeves, but it doesn’t mean that **everyone** should run it. Companies can be just as happy with other solutions, e.g internet product companies that are solely deployed on AWS have far better chances of enjoying their production life with ECS rather than k8s, yes, rather than EKS too.

Having that said, k8s has its magic — it can be deployed anywhere, it has an active community with hundreds of core developers, and thousands of other open source contributors in the wide ecosystem around it. It’s fast, innovative, modular and API oriented, making it a super friendly system to build add-ons or services upon.
> # So without further ado, let’s do it; k8s intro in 11 steps:

## 1. Pods

**Pods**, are the single smallest interact-able unit in k8s. A pod can be comprised of multiple containers that will form a unit deployed on a single node together. A pod receives one IP, which is shared among its containers.
In a micro-service world, a pod would be a single instance of a micro-service doing some background work, or serving incoming requests.

## 2. Nodes

**Nodes** are machines. They are the “bare metal” (can also be VMs) on which k8s deploys its pods. Nodes provide the available cluster resources for k8s to keep data, run jobs, maintain workloads and creates network routes.

## 3. Labels & Annotations

Labels are k8s and its end users way to filter similar resources in the system, they are also the glue where one resource needs to “access” or relate to another resource. For example a Service that wants to open ports for a Deployment. Whether for monitoring, logging, debugging, testing, any k8s resource should be labeled for further inspection. E.g: app=worker, a label given to all worker pods in a system, that can be later selected using the --selector field using kubectl tool, or k8s api.

Annotations are very similar to labels, but are usually used to keep metadata for different objects in the form of freestyle strings, e.g “*Reason for change: Upgrading the applications version for security patches*”.

## 4. Service Discovery

Being an orchestrator, controlling many resources of different workloads, k8s manages networking for pods, jobs, and any physical resource that requires communication. In order to manage that, k8s uses [ETCD](https://kubernetes.io/docs/concepts/overview/components/#etcd).
ETCD is k8s “internal” database, the masters use it to know where everything is located. K8s also has actual “service discovery” for your services — it is using a custom DNS server that all Pods are using and you can resolve names of other services to get their IP addresses and ports. It works inside of a K8s cluster “out of the box” and there is nothing required to set it up.

## 5. ReplicaSets

While pods are a physical running task, it is usually not enough to use a single instance of them. For redundancy, and load handling, pods must be replicated for various reasons i.e “scaling”. In order to implement the layer that’s in-charge of scaling and replication, k8s uses **ReplicaSets**. This layer represents the desired state of the system in terms of number of replicas, and holds a current status for the system at any given moment.

This is also the place to configure auto-scaling, where additional replications are created when the system is loaded, as well as scale-in when those resources are no longer required to support the running workload.

## 6. DaemonSets

Sometimes, certain applications require no more than a single instance on every node. A very good example is a log collector such as [FileBeat](https://www.elastic.co/products/beats/filebeat). In order for the agent to collect logs from nodes, it needs to be on all of them, but only one instance of it. In order to create such a workload deployed, k8s has **DaemonSets** that allow exactly that.

## 7. StatefulSets

Although most of the micro-services world is involving immutable stateless applications, some of them are not. Stateful workloads demand to be reliably backed by some kind of disk volume. While the application container itself can be immutable and be replaced with newer versions or healthier instances of themselves, they would need the persistency of their data even with other replications. For that, **StatefulSets** allow deployment of applications that require the use of the same node throughout their lifetime. It also retains its “name”; both the `hostname` inside of containers and the name in service discovery across the cluster. A StatefulSet of 3 ZooKeepers can be named zk-1, zk-2 and zk-3 It can also be scaled to include additional members like zk-4, zk-5 etc… StatefulSets also manage PersistentVolumeClaim(s) (disks connected to pods).

## 8. Jobs

K8s core team has thought about the vast majority of applications that would use an orchestration system. While the majority require constant uptime to simultaneously server requests (e.g a web server), we sometimes need a batch of jobs to be spawned up and cleaned up once finished. A mini-serverless environment if you will. In order to achieve that in k8s, we can use the **Job** resource. Jobs are exactly what they sound, a workload that spins up containers to complete a specific work, and be destroyed on a *successful* completion. A good example can be a set of workers, reading jobs from a queue of data to be processed and stored. Once the queue is empty, the workers are no longer required, until the next batch is ready to be processed.

## 9. ConfigMaps & Secrets

If you aren’t already familiar with the The [Twelve-Factor App manifest](https://12factor.net/), you should. One of the key concepts of modern applications, is being environment-less and [configurable](https://12factor.net/config) from injected environment variables. An application should be completely agnostic to its location. In order to achieve this important concept in k8s, we’re given **ConfigMaps**. These are essentially a lists of key-value environment variables which are passed to running workloads in order to determine different runtime behaviours. On the same scope, we have **Secrets **which** **are similar to normal configuration entries, except being encrypted to prevent leaks of sensitive information like keys, passwords, certificates etc.
> The best option I personally know to use secrets on any system is Hashicorp’s Vault. Be sure to read the [a post I wrote about it](https://medium.com/prodopsio/security-for-dummies-protecting-application-secrets-made-easy-5ef3f8b748f7) last year, about the reasons why you would want Vault as part of your production, and another great and a [more technical one](https://medium.com/prodopsio/taking-your-hashicorp-vault-to-the-next-level-8549e7988b24) written by a colleague of mine.

## 10. Deployments

It’s all nice and dandy when you have your pods running, even with a ReplicaSet on top, scaling things when load is demanding. But we’ve all gathered here for a quick replacement of our applications with newer versions. We want to build, test and ship in small chunks, to enjoy [short feedback loops](https://www.ibm.com/developerworks/community/blogs/beingagile/entry/short_feedback_loops_everywhere?lang=en). K8s lets us continuously deploy new software using **Deployments**; a set of metadata describing new requirement from a certain running workload. A good example for that is a new version, a bug fix, or even a rollback (which is another internal option of k8s).

Deploying software in k8s has 2 main *strategies* to use:

1. Replacement — like it sounds, would replace your entire workload with the new requirement and naturally enforces downtime. It’s good for quick replacement of non-production resources.

1. RollingUpdate — k8s way of slowly replacing containers with new ones by listening to two specific configurations: a. MaxAvailable — a setting of what percentage (or exact number) of the workload should be available when deploying a new version, 100% meaning “I have 2 containers, keep 2 alive and serving requests throughout the deployment”. b. MaxSurge — sets the percentage (or number) of workload to deploy on top of the current live one, 100% meaning “I have X containers, deploy another X containers, and then start rolling out old ones”.

## 11. Storage

K8s adds a layer of abstraction on top of storage. Workloads can request specific storage for different tasks, and even manage persistency that outlasts a certain pod’s lifetime. In order to keep it short, I’d like to refer you to [a recent post I published about k8s storage](https://medium.com/prodopsio/k8s-will-not-solve-your-storage-problems-5bda2e6180b5) and specifically why it won’t solve completely data persistency requirements like DB deployments.

## Conceptual Understanding

K8S was (and still is) designed and developed in the light of a few guiding directions, each feature, concept and idea are built into the system considering the nature of the community. Moreover, end-users are guided to use the system in a certain way although never forced; best practices are known, but being an open source and free system that is not owned by anyone, you can do whatever you want **however** you want with it.

API oriented — every part in the system is built in a way that it’s interact-able via a well documented and operational API. The core developers make sure that you, as an end user, can make changes, queries and updates so that you are never shut behind a masking curtain or unwanted filters.

Wrapper tools welcoming — as a derivative for the previous point, k8s is welcoming tools and wrappers to be built around and on top of its API. It introduces itself as a raw platform built in a very customize-able way for others to use and further develop tools for different use cases. Some have become very famous and widely used like [Spinnaker](https://www.spinnaker.io/), [Istio](https://istio.io/) and many others.

Declarative state — users are encouraged to use the system with declarative descriptions rather than imperative ones. This means, that the system’s state and components better be described as code managed in some sort of version control like git, rather than an outcome of manual changes that led to a certain point. This way, k8s is more [DR](https://en.wikipedia.org/wiki/Disaster_recovery) resistant, easy to share among teams and pass responsibilities.

## That’s it

Trying to keep the focus on k8s introduction and main concepts, this has been the list of things to know, when being introduced to this great system.
Of course k8s has other very important areas like physical system building blocks like *kubelet*, *kube-proxy*, *api-server* and the ultimate control tool — *kubectl*. I’ll be discussing these and some other cool features on my next posts. Be sure to follow and stay tuned for more.

*My name is Omer, and I am an engineer at [ProdOps](http://prodops.io) — a global consultancy that delivers software in a Reliable, Secure and Simple way by adopting the Devops culture. Let me know your thoughts in the comments below, or connect with me directly on Twitter [**@**omergsr](https://twitter.com/omergsr). Clap if you liked it, it helps me focus my future writings.*
