
# Kubernetes Operators to realize the dream of Zero-Touch Ops

Kubernetes Operators has the power to realize the dream of Zero-touch Ops, bringing in AIOps to life…and this is how I believe it will.

## Operators

As we step into MicroServices architectures, and ways to deploy these on cloud with containers, and all the goodness of DevOps …the application functionality grows..the clusters and the number of resources in the cluster also grows…if the application is not “built-for-manage”, its going to be a nightmare to manage these applications, and we might end up spending more effort in managing these applications, than building them…ironically!!! while the world of automation technology has huge promise, and we are talking about zero-touch ops as nirvana for managing cloud applications!!!.
> According to me Operators is the most important architectural component in the k8s world, that has a huge promise to carry us towards our zero-touch (or low-touch) ops journey..

Before I jump in…let me quickly walk u thru my understanding of operators (and I am sure there are a lot of blogs, vblogs, youtube videos, which might do a better job.. :-).)

k8s is all about Controllers & Resources.
> [Resource](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#:~:text=A%20resource%20is%20an%20endpoint,a%20collection%20of%20Pod%20objects.&text=However%2C%20many%20core%20Kubernetes%20functions,resources%2C%20making%20Kubernetes%20more%20modular.): A *resource* is an endpoint in the [Kubernetes API](https://kubernetes.io/docs/reference/using-api/api-overview/) that stores a collection of [API objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/) of a certain kind; for example, the built-in *pods* resource contains a collection of Pod objects.
> [Controllers](https://kubernetes.io/docs/concepts/architecture/controller/): In Kubernetes, controllers are control loops that watch the state of your [cluster](https://kubernetes.io/docs/reference/glossary/?all=true#term-cluster), then make or request changes where needed.

![Resources & Controllers](https://cdn-images-1.medium.com/max/2000/1*rVlbUlxIPAzfOMbudajeNA.png)*Resources & Controllers*

Controllers have the logic of managing the resources, and that's how the K8s cluster runs.

In the initial versions of the k8s, it came with defined resources, and we were only restricted to use those resources that came along with the k8s.
> Controllers are very good in managing stateless applications, as its like a constant control loop to track and fix. since applications are stateless, there is no backup/recovery/restore of state. for-example if a instance of webserver crashes, controller can easily replace that with another instance of webserver and bring it back to desired state.
> But for stateful applications like databases, it’s not that straight forward, and it will require manual intervention to restore the state!!! so we need something more than standard controllers.

Since the introduction of the Custom Resources, we have the flexibility to declare and create our own k8s resources.

Now imagine if we can start defining our own resources and letting the k8s also manage them!!!! and even better imagine, if we can build our own controllers to have our own custom manage logic, and letting k8s run our resources!!!…and that is what is “Operators”!!!

With Operators, we should be able to write the logic for complete management of custom resources, and let k8s manage our resources!!!..and that's how we can move to low-touch ops!!!

so what all can we automate with operators…the answer is “everything that can be automated”…right from installation, patching, updates, upgrades, backup, recovery, capturing telemetry, and acting based on AI (artificial intelligence to the nirvana stage of zero-touch ops.

![Operators Maturity Model](https://cdn-images-1.medium.com/max/2000/1*ZLRvdqerOAloSVbFWWyEfw.png)*Operators Maturity Model*

There is a very well defined Operators maturity model, that clearly defines the [5 phases of maturity](https://docs.openshift.com/container-platform/4.1/applications/operators/olm-what-operators-are.html).

![How Operators work](https://cdn-images-1.medium.com/max/2000/1*75UDw8T8l54FsAsezazQ8A.png)*How Operators work*

There are 3 main components of Operators Framework

**Operators SDK**: provides the tools to build, test, and package the Operators. Provides 3 SDK out of the box

* **Helm SDK**: provides a declarative way of building Operators, with this mainly install and configure kind of Operators can be built

* **Ansible SDK**, **Go SDK**: Ansible and GO SDKs provide more advanced ways of building the Operators. where you can build Operators all the way to “Auto-Pilot” maturity.

Apart from Operators SDK — there are some tools in the market such as [KUDO](https://kudo.dev/), [kubebuilder](https://book.kubebuilder.io/), [Metacontroller](https://metacontroller.app/)

**Operator Lifecycle Manager (OLM)**: manages the complete lifecycle of the Operator — installing and managing the Operator. OLM monitors the CRD that is deployed and when something changes..then it ensures that the changes are applied across the cluster

**Operator Metering**: reports the usage of the operator to help the metering

![Operator Architecture](https://cdn-images-1.medium.com/max/2000/1*Q0_PgdZLpRFFImPQCjxINQ.png)*Operator Architecture*

## Creating & Deploying an Operator

Here is a quick walk-thru of building and deploying an Operator. Just for the completeness, I thought I will do a very quick walk-thru

1. [**Install Operator SDK](https://sdk.operatorframework.io/docs/installation/install-operator-sdk/)**

1. [**Build, Test and Deploy](https://sdk.operatorframework.io/docs/olm-integration/quickstart-bundle/)**

1. [**Evolve & Mature](https://sdk.operatorframework.io/docs/advanced-topics/operator-capabilities/operator-capabilities/)**

## AIOps for Zero-Touch Ops

Artificial Intelligence & applying machine learning for ITOps has become a reality and has already become a very common practice to bring down the operational cost. So what capabilities are required for AIOps???

![AIOps Capability Architecture](https://cdn-images-1.medium.com/max/2000/1*VJrj3HE4H4_QqK_mvliS-w.png)*AIOps Capability Architecture*

The picture above illustrates my understanding of AIOps capability architecture. (thanks [Naveen E P](undefined) for brainstorming and contribution in building this nice picture).

AIOps goes beyond standard event detection to advanced prediction with actionable insights. The term “actionable” is important — it’s the recommendation or execution of the best action to fix the current or issues that might occur based on prediction. This is what we really need for an “Auto-Pilot” Maturity, where it will replace or augment Site Reliability Engineers (SRE).

Now if you connect this generic picture of AIOps with what k8s Operators bring to the table, it is very clear that the operators have all that we need to be our AIOps engine.
> All the various types of capabilities can be built as a CRs, and can be a bunch of operators that will bring all the pieces of AIOps to life, these operators co-locate inside the K8s cluster and run as PODs/Sidecars. They can also integrate with ServiceMesh for additional metrics and telemetry, and act proactively and operate the cluster.

![AIOps with Operators — Illustrative Architecture](https://cdn-images-1.medium.com/max/2000/1*QHEpWXqvYNRMEJ0JUREeRQ.png)*AIOps with Operators — Illustrative Architecture*

The above picture provides a high-level view of the idea, and let's see how it maps to the 3 layers that we talked on the AIOps capability architecture

* **Visibility**: Visibility layer can be built on [Grafana](https://grafana.com/), providing single pane visibility of the cluster health

* **Prediction**: Prediction layer has all the modules (python modules to advanced spark clusters as specific operators), that build machine learning models from the data that is streaming from [Prometheus](https://prometheus.io/), ServiceMesh/istio.

* **Resolution**: Resolution can be simple k8s commands to [Ansible](https://www.ansible.com/) playbooks or even invoking RPA digital works — depending on standard operating procedures, to recover the failures or take proactive measures

The best part is all of this AIOps is happening native to Kubernetes (except maybe RPAs)

There you go, Operators is the key to unlock the “Zero-Touch Ops” Journey.

In the meantime, I have been playing around with operators and will soon come back with a hands-on session…

Have fun, take care..ttyl

## References

* [https://access.redhat.com/documentation/en-us/openshift_container_platform/4.1/html/applications/operators](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.1/html/applications/operators)

* [https://developers.redhat.com/blog/2020/04/15/crafting-kubernetes-operators/?utm_content=bufferec5c6&utm_medium=social&utm_source=facebook.com&utm_campaign=buffer](https://developers.redhat.com/blog/2020/04/15/crafting-kubernetes-operators/?utm_content=bufferec5c6&utm_medium=social&utm_source=facebook.com&utm_campaign=buffer)

* [https://sdk.operatorframework.io/](https://sdk.operatorframework.io/)

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Subscribe to [FAUN topics](https://www.faun.dev/join?utm_source=medium.com/faun&utm_medium=medium&utm_campaign=faunmediumprebanner) and get your weekly curated email of the must-read tech stories, news, and tutorials **🗞️

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and [Instagram](https://instagram.com/fauncommunity/) **📷 **and join our [Facebook](https://www.facebook.com/groups/364904580892967/) and [Linkedin](https://www.linkedin.com/company/faundev) Groups **💬

![](https://cdn-images-1.medium.com/max/3000/1*_cT0_laE4iPcqW1qrbstAg.gif)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
