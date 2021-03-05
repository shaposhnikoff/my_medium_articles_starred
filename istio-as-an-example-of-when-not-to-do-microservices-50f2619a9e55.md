
# Istio as an Example of When Not to Do Microservices

I’ve been pretty invested in helping organizations with their cloud-native journeys for the last five years. Modernizing and improving a team (and eventually an organization’s) velocity to deliver software-based technology is heavily influenced by it’s people, process and eventual technology decisions. A microservices approach may be appropriate when the culmination of an application’s architecture has become a bottleneck (as a result of the various people/process/tech factors) for making changes and “going faster”, but it’s not the only approach.
> Microservices is not THE “utopian application architecture”.

I’ve written in the past how I didn’t think many teams [would be able to pull it off](https://blog.christianposta.com/microservices/youre-not-going-to-do-microservices/), [how there are “hard parts” to getting it working](https://blog.christianposta.com/microservices/the-hardest-part-about-microservices-data/), and even heads up about some technology that might [be beneficial to your efforts in the long run](https://blog.christianposta.com/microservices/the-hardest-part-of-microservices-calling-your-services/). FWIW I even [wrote a book](https://www.oreilly.com/library/view/microservices-for-java/9781492042228/) on the topic.

[Staying away from microservices](https://blog.christianposta.com/microservices/when-not-to-do-microservices/) is likely the best place to start, though many organizations are way passed that today.

## You’ve already gone down the path of microservices

If you do go down the path of microservices, be honest with yourself and the organization when it’s not working. Correcting course may be the right move for the success of your product.

## Honesty about when it’s not working

Despite best intentions, it may be the right choice to go back to a monolith once you’ve started with microservices, even if it was for the right reasons. “It’s okay” to go back to a monolith if your assumptions or the context around your decisions have changed.

![](https://cdn-images-1.medium.com/max/2000/0*AJHVxOYdeFzuBw2n.png)

In the [Istio community](https://istio.io/), which builds a [service mesh](https://istio.io/docs/concepts/what-is-istio/#what-is-a-service-mesh) for microservices communication, the implementation of the control plane will be [gradually changing from a microservices approach to a more monolithic approach](https://docs.google.com/document/d/1v8BxI07u-mby5f5rCruwF7odSXgb9G8-C9W5hQtSIAg/edit#). [Louis Ryan](https://twitter.com/louiscryan?lang=en), who’s the Principal Engineer and architect on Google’s API infrastructure, gave a talk back at the Istio meetup at KubeConNA 2019 detailing the motivations as well as [outlining the case in a design doc](https://docs.google.com/document/d/1v8BxI07u-mby5f5rCruwF7odSXgb9G8-C9W5hQtSIAg/edit#). Starting in Istio 1.5 (expected mid-February 2020), we should begin to see the effect of the istiod approach where functionality previously assigned to various microservices deployments will be coalesced into a single daemon.

Istio is used to help solve difficult application-networking challenges introduced by a microservices/cloud architecture, so why would Istio itself move away from a microservices architecture? The most straight-forward answer is:
> The complexity of the microservices approach proved to not deliver the value or goals it intended. On the contrary, it worked against those goals.

For the Istio project, it looks like a monolithic approach would better contribute to those goals. Let’s take a closer look.

## Istio implemented as microservices

Istio is a open-source [service mesh](https://istio.io/docs/concepts/what-is-istio/#what-is-a-service-mesh), which is architected similar to other service-mesh implementations with a control plane and a data plane. The data plane consists of the proxies that live with each application instance and is in the request path. The control plane lives outside of the request path and is used to administer and control the behavior of the data plane.

![](https://cdn-images-1.medium.com/max/2000/0*CUzt_5iaNjmVHemw.png)

Historically, Istio’s control plane was implemented as separately deployable services that did the following:

* Pilot - the core data-plane config (xDS) server

* Galley - configuration watching, validation, forwarding

* Injector - responsible for auto-injecting the data plane and setting up bootstrap

* Citadel - certificate signing, secret generation, integration with CAs, etc

* Telemetry - a “mixer” component responsible for aggregating and syndicating telemetry to various backends

* Policy - a request-path “mixer” component responsible for enforcing policy

These services would be driven by a set of operator-defined configuration and coordinate to eventually serve and direct the data plane.

## Microservices benefits

Microservices can enable an organization to go faster by reducing friction to make changes to the system. With a microservices architecture, each service would likely be operated independently (each with their own team) and have it’s own release cadence/lifecycle independent from others. This would give the developers and operators parallel tracks to move faster without the locking/synchronization/coordination of making changes which can serve to slow down deployments and feature changes.

Another reason why a service may be further broken down is it’s usage patterns and scaling properties. For a simple example, a service that has heavy reads and writes may benefit from separating out reads from writes because reads may be more memory intensive (maybe needing more cache space to make the reads super fast) while writes may be more storage or network intensive. You could optimize the read part of a service on machines/quotas that allow it to scale independently (more memory) then have the write part of the service on other machines that have SSD or optimized EBS/SAN, etc.

Some other reasons why you might split an app into services:

* security concerns

* domain grouping

* different language optimizations

* criticality of service

The #1 tradeoff when going to a microservices architecture is complexity. When you go from one thing (monolith) to a bunch of little things communicating with each other (to optimize for a particular concern), you significantly increase complexity both in the architecture as well as the infrastructure necessary to run things.

This may be a necessary tradeoff insofar you realize the benefits. If not, it’s best you evaluate your assumptions and correct course. That’s what’s happening with Istio right now.

## Correcting course

The first thing to understand is who is developing and operating your services architecture. In the Istio community, there are different components in the project as can be seen with the [different community working groups](https://github.com/istio/community/blob/master/WORKING-GROUPS.md). On the other hand, the persona downloading and operating an Istio installation is not as deconstructed. In fact, the observation so far is that a single group (or even single person) operates the Istio control plane. In some ways, the Istio control plane as a set of microservices would work well if run as a larger SaaS, but in its current adoption that doesn’t seem to be the case.

The second thing to understand is how is a release done? Can the services be released independently? The answer for Istio was “theoretically yes” in practice this doesn’t seem to be the case. When a new version of Istio is released, you need to upgrade/deploy all of the control-plane components.

Lastly, in the Istio case, you could ask “aren’t there other scaling variables and security concerns that are different for the various components?” And an honest answer would realize that there isn’t really. Taken directly from the Istio design doc for istiod:
> This is not the case in Istio today for the majority of components however — control plane costs are dominated by a single feature (serving XDS). Every other control plane feature has a marginal cost by comparison, therefore there is little value to separation.

For security, all of the control-plane services had the same level of privilege:
> This is not the case today as the powers exercised by the Mutating Webhook, Envoy Bootstrap & Pilot are similar to those of Citadel in many respects and therefore exploits of them have near equivalent damage

As the subtext in the Istiod design doc states *“Complexity is the root of all evil or: How I Learned to Stop Worrying and Love the Monolith”*.

istiod is an incarnation of a monolith which supports all of the functionality of the previous releases with significantly reduced complexity. Note that the services that made up the previous control planes are still implemented as sub-modules within the project (complete with boundary and contracts, etc) but the operational experience is improved. An operator now needs to worry about running and upgrading a single binary vs a collection of them.

![](https://cdn-images-1.medium.com/max/2000/0*Nycv7iVC7objIctH.png)

For Istio going to a monolithic control plane, a significant amount of complexity can be reduced which never fully paid off:

* Installation/upgrade becomes easier with only a single deployable service

* Configuration complexity is reduced, as configuration to orchestrate services is no longer needed

* Easier to debug issues (look in one place vs all the places)

* Increase efficiency/reduce overhead of transmissions, sharing of caches, etc

See the [Istiod design doc](https://docs.google.com/document/d/1v8BxI07u-mby5f5rCruwF7odSXgb9G8-C9W5hQtSIAg/edit#) for more detail.

Also as a side note: [you can check out a demo I did of this](https://www.youtube.com/watch?v=QD115XiBXwY)istiod approach which should appear in Istio 1.5. Just note, it’s demo’d on a super alpha build of Istio, so not all polished like it will be :)

## Conclusion

I’m happy to see the Istio community continue to improve its usability and operability characteristics. Going to a monolithic deployment of the Istio control plane makes a lot of sense for the project. Is that something that makes sense for your project? Is that something you would consider if it did? Are you measuring the value-to-complexity ratio for your microservices architecture (and associated infrastructure) in such a way that you’d be able to determine the time to change approaches?

Reach out to me [@christianposta on twitter](https://twitter.com/christianposta) if you have thoughts you’d like to share. A good follow up post would be something along the lines of a decision table or key indicators of when you should be tempted to change course once you’ve decided to go down the path of microservices et. al. Hit me up if you’d like to contribute to that.
