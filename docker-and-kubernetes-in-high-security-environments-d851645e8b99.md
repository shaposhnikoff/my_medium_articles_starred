
# Docker and Kubernetes in high security environments

A case-study at the Swedish Police Authority

![](https://cdn-images-1.medium.com/max/4744/1*c2HXMmSKX2u0N_WXqYfBgw.png)

*This is brief summary of **parts** of my master’s thesis and the conclusions to draw from it. This medium-story focuses on **containerized application isolation**. The thesis also covers **segmentation of cluster networks **in Kubernetes which is not discussed in this story.*

*You can read my full thesis here; it’s available through open access:
[Container Orchestration in Security Demanding Environments at the Swedish Police Authority](http://kth.diva-portal.org/smash/record.jsf?pid=diva2:1231856).*

Container orchestration and [cloud-native](https://www.cncf.io/about/charter/) computing has gained lots of traction the recent years. The adoption has increased to such level that even enterprises in finance, banking and the public sector are interested. Compared to other businesses they differ by having extensive requirements on information security and IT security.

One important aspect is how containers could be used in production environments while maintaining system separation between applications. As such enterprises uses private clouds powered by bare-metal virtualization, the separation loss upon migrating to a container orchestrated environment is not negligible. It is in this scope that my thesis is written –with the Swedish Police Authority as the target client.

The specific research question that the thesis explores is the following:
> How can Docker and Kubernetes support the separation of applications for the Swedish Police Authority compared with virtual machines powered by the bare-metal hypervisor ESXi?

That question has a lot to unwrap. To break this down, let’s start by looking in to the common denominator — the applications.

## Web apps are messy

Vulnerabilities in web apps are like a zoo of attack vectors where the most prominent risks represented in the OWASP Top 10 ([2013](https://www.owasp.org/index.php/Top_10_2013), [2017](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_2017_Project)). Such resources help educate developers to mitigate common risks. However, even though the developers have coded everything to perfection the application can still be vulnerable through, for example, package dependencies.

[David Gilbertson](https://hackernoon.com/im-harvesting-credit-card-numbers-and-passwords-from-your-site-here-s-how-9a8cb347c5b5) wrote a great story on how code injection could be achieved by distribution of a malicious package through for example NPM for Node.js based applications. While dependency scanners can be used to spot vulnerabilities, it only *reduces* the risk. It does not remove it.

Even if you built an apps which had no 3rd-party dependencies it would still be unrealistic to assume that an app would never ever be vulnerable.

**Due to these risks we cannot assume that web apps are secure.**

Instead we must use a [**defense in depth](https://searchsecurity.techtarget.com/definition/defense-in-depth)** strategy. So let’s look at the next layer — containers and virtual machines!

## Containers vs virtual machines – a tale of isolation

A container is an isolated user-space environment which is often realized through the use of kernel features. [Docker](https://docs.docker.com/engine/security/security/) for example uses Linux namespaces, control groups and capabilities to achieve this. In this regard Docker containers isolate **very differently** compared to virtual machines powered by bare-metal hypervisors.

In such virtual machines the separation can be implemented as far down as in the actual hardware through for example [Intel VT](https://www.intel.com/content/www/us/en/virtualization/virtualization-technology/intel-virtualization-technology.html). Docker containers on the other hand rely on the Linux kernel for the separation. This difference is a very important when considering **layer-below attacks**.

If an attacker is able to execute code in a virtual machines or container, then the attacker could attack the underlying layer to perform an **escape attack**.

![The separation is implemented in different layers depending on if containers or bare-metal virtual machines are used](https://cdn-images-1.medium.com/max/4744/1*42mQjHPLTh3EwMBinoJjzw.png)*The separation is implemented in different layers depending on if containers or bare-metal virtual machines are used*

Such attacks were proven possible on the bare-metal hypervisor [VMware ESXi](https://www.vmware.com/products/esxi-and-esx.html) during the Pwn2Own 2017 hacking contest as well as GeekPwn2018. This resulted in a couple of CVEs (e.g. [CVE-2017–4902](https://nvd.nist.gov/vuln/detail/CVE-2017-4902), [CVE-2018–6981](https://www.vmware.com/security/advisories/VMSA-2018-0027.html)) which could be used in layer-below attacks to achieve a **virtual machine escape.** Even though bare-metal virtual machines use hardware-level separation technologies it does not make them infallible.

If we on the other hand consider the attack vectors of the bare-metal hypervisor compared with the Linux kernel it’s obvious that the latter has a larger attack surface due to its size and range of functionality. A larger attack surface implies more potential attack vectors for clouds relying on container isolation. This is reflected by the increased attention to **container escape attacks **which have been proven possible through some kernel exploits (e.g. [CVE-2016–5195](https://nvd.nist.gov/vuln/detail/CVE-2016-5195), [CVE-2017–1000405](https://nvd.nist.gov/vuln/detail/CVE-2017-1000405)).

To increase isolation **inside** the container, security modules like [SELinux](https://opensource.com/business/13/11/selinux-policy-guide) or [AppArmor](https://en.wikipedia.org/wiki/AppArmor) could be used. Unfortunately, such kernel-based security mechanisms will not prevent kernel-based escape attacks. It would only restrict an attacker’s actions if an escape *cannot* be achieved. If we want to tackle container escapes we need some isolation mechanism either **outside** the container or even the kernel. A sandbox for instance!

[gVisor](https://github.com/google/gvisor) is a container runtime sandbox open-sourced by Google which adds an additional kernel between the container and the OS kernel. This type of sandbox could mitigate container escape attacks *which are kernel-based*. For an attacker however, kernel exploits are just one thing of what’s available in the toolbox.

To see how other attacks could achieve similar effects we must look at the bigger picture of how containers are used in the cloud-native era.

## The Impact of Container Orchestration on Isolation

In order to manage containers running on multi-node environments, we have adopted container orchestration where Kubernetes has a leading role. As it turns out, bugs in the orchestrator can also impact the container isolation.

[Tim Allclair](https://youtu.be/GLwmJh-j3rs?t=14s) gave us a great presentation at KubeCon 2018 where he highlighted several attack surfaces. In his talk, he [mentions](https://youtu.be/GLwmJh-j3rs?t=12m15s) one instance ([CVE-2017–1002101](https://nvd.nist.gov/vuln/detail/CVE-2017-1002101)) on how orchestration bugs can impact isolation, in this case through volume mounting of space outside the pod. This type of vulnerability is problematic since it also may bypass a sandbox which wraps the container.

By introducing Kubernetes, we add an attack surface. The systems hosted by the Kubernetes Master being one of them. One such system is the Kubernetes API server in which a privilege escalation vulnerability ([CVE-2018–1002105](https://access.redhat.com/security/cve/cve-2018-1002105)) was found recently*. *Since the attack surface of the Kubernetes master is outside the scope of my thesis this particular vulnerability is not taken into account.

So why are escape attacks a big deal? Containers introduces the ability to run multiple co-located applications on the same OS. This introduces a risk with regard to application separation. If a business-critical application and another vulnerable application is being run on the same host, then an attacker could through an attack on the vulnerable app gain access to the critical app.

Depending on the type of data which the enterprise handles, a data breach could harm both individuals and the nation besides the enterprise itself. Remember we’re talking about the public-sector, finance, banks etc. so a breach could impact people’s lives to a serious degree.

So can container orchestration be used at all in such environments? Before we go any further in this mindset we need to do some risk assessment.

## What is an acceptable risk?

Before introducing security measures it’s important to reflect on the actual information which the business is trying to protect. Depending on the data which an enterprise handles and the services it provides, it may or may not be a need to provide any further mitigation against container escape attacks.

To put this into perspective — in order for a container escape to occur on a *correctly configured* host which is hardened with container sandboxing, an attacker must:

1. Execute code in the container through, for instance, code injection or by using an existing vulnerability in the application code

1. Utilize another vulnerability that is either a zero-day or unpatched in order to escape the container *despite a container sandbox being in plac*e.

As you may understand, the enterprise which does not find the scenario above acceptable must be handling data or provide services which are quite* *demanding on **confidentiality**, **integrity **and/or **availability**.

As the scope of this thesis is written with such clients in mind, the loss of system isolation through container escapes is not acceptable as the consequences are too severe. So what steps can we take to further improve isolation? To take a step up in the isolation-ladder we need to consider sandboxes that wraps the OS kernel as well — virtual machines!

While virtualization technologies which utilize hosted hypervisors would provide an improvement, we want to limit the attack surface *even further*. So let’s explore bare-metal hypervisor and see where they’ll gets us.

## Bare-metal hypervisors

In a [report](https://www.foi.se/en/foi/news-and-pressroom/news/2017-12-04-virtualized-it-systems-require-risk-assessment.html) published by the Swedish Defence Research Agency the risks of virtualization were evaluated with regard to the Swedish Armed Forces. The report concluded that it could be useful for the Armed Forces despite its tough demands on security and the risks that virtualization brings.

In this regard we can assume that virtualization is used (to some degree) in the defense industry as it has **acceptable risks. **Since agencies and enterprises in the defense industry are among the most demanding clients in IT security, we can also assume that an acceptable risk for them is an acceptable risk for the clients in the scope of this thesis. This is also despite the possibility of virtual machine escapes which we discussed earlier.

If we decide to use this type of sandboxing with containers we need to take some things into account to make it more cloud-native.

## Virtual machine sandboxed nodes

The idea is to let the nodes of the Kubernetes cluster be virtual machines which utilizes bare-metal virtualization. As virtual machines will act as sandboxes for the containers which are running in the pods, we can view each node as a sandboxed environment.

An important note between this type of sandboxing and the previously mentioned container sandboxing is that this type allows multiple containers to be placed in the same sandbox. This flexibility provides allows a reduction in overhead compared to if each container would have its own sandbox. Since each sandbox brings its own OS, we want to reduce the number of sandboxes while maintaining separation.

![The bare-metal VM nodes acts as sandboxes for the containers. Containers running in different VMs are separated with an acceptable risk. However, containers running in the same VM are not.](https://cdn-images-1.medium.com/max/3904/1*KoxKLuq4gDCrVYS5XrhuPA.png)*The bare-metal VM nodes acts as sandboxes for the containers. Containers running in different VMs are separated with an acceptable risk. However, containers running in the same VM are not.*

However, since Kubernetes may relocate pods for various reasons, which** could break the sandboxing,** we need to add restrictions to the pod co-location mechanism. This can be done in many ways.

As of time of writing Kubernetes (v1.13) supports 3 main methods to control the scheduling and/or execution of pods.

* [NodeSelector](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector) & [NodeAffinity](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#node-affinity-beta-feature)

* [Taints & Tolerations](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)

* [PodAffinity & PodAntiAffinity](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#inter-pod-affinity-and-anti-affinity-beta-feature)

Which method(s) to use depends on the applications of the enterprise. However, an important note is that the methods differ in ability to reject pods after they have entered the execution phase. Currently only taints are able to do this through the NoExecute action. If this is not handled and some label changes, it could result in an unwanted co-location.

## Representing co-location demand

The thesis proposes the idea of using a classification system to provide a mapping on how the sensitivity reflects the co-location. The idea is to use a 1:1 relationship container and pod and let the co-location of pods be based on the containerized application classification.

For the sake of simplicity and re-usability the following 3-step classification system was used.

* **Class O**: The app is not sensitive and has no demands on separation. It can be scheduled on any node that does not belong to the other classes.

* **Class PG**: The app, together with a set of other apps, form a *private group* which in turn requires a dedicated node. The app can thus only be scheduled on nodes of class PG which share the private group identifier.

* **Class P**: The app requires a *private* and dedicated node and can only be scheduled on *empty* nodes of class P.

To satisfy the co-location demands on a set of classified applications, taints and tolerations were used to allocate each node to a class and PodAffinity to apply additional restrictions for pods containing P or PG class applications.

This simplified example shows how taints and tolerations could be used to implement a co-location control.

![Pod 2 & 3 contain apps from the same private group while the app in Pod 4 is more sensitive and requires a dedicated node](https://cdn-images-1.medium.com/max/2016/1*m8h3UDo1D3oXGnSe5FISIA.gif)*Pod 2 & 3 contain apps from the same private group while the app in Pod 4 is more sensitive and requires a dedicated node*

For class P and PG however, additional Affinity-rules are needed to ensure the separation demand is satisfied as the cluster and its hosted applications grow. Let’s look at how we can implement the affinity rules for the different classes:

<iframe src="https://medium.com/media/33c51d7a01ba90aac513155b0907e414" frameborder=0></iframe>

The Affinity rules for class P apps need dedicated nodes. In this case the pod will not be scheduled where a pod exists without the non-existing-key. This will work as long as no pod has this key.

For class PG apps the affinity rules will co-locate with pods which has the group identifier class-pg-group-1 and nodes which have pods without the identifier.

This will allow us to separate containers based on the separation demand of the containerized application through the class-based system.

So what does this provide us with?

## Conclusion

In this story we have now seen a method to enforce a bare-metal hypervisor-based sandboxing to create nodes in Kubernetes clusters and introduced a classification system to represent the separation demands of the containerized applications. This provides a benefit in comparison with the other solutions we have discussed with regard to system isolation.

An important takeaway is that the isolation strategy *restricts the propagation* of a container escape attack. In other words– **The container escape itself is not mitigated but its consequences are. **Obviously this brings more complexity which needs to be taken into consideration in a comparison.

In order to utilize this in clouds in a scaleable manner, there are additional requirements for automation that must be satisfied. For example, automating the creation of virtual machines, attaching them to the Kubernetes cluster. Most importantly one must also implement and verify that the application classifications are respected **at all times**.

This pretty much wraps up the part of my thesis which covers **containerized application isolation**.

However, to prevent an attacker which has achieved a container escape on one node from attacking services on other nodes we need to consider **network propagated attacks**. In an effort to consider those risks, my thesis continues with **c*luster network segmentation ***and presents some cloud architectures, one of which features a **hardware firewall**.

For this you may read my thesis, [Container Orchestration in Security Demanding Environments at the Swedish Police Authority](http://kth.diva-portal.org/smash/record.jsf?pid=diva2:1231856), as this medium-story is far too long as it is.

**Happy coding!**
