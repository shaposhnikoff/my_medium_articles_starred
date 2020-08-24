
# How To Become a DevOps Engineer In Six Months or Less, Part 4: Package

“Packages” by chuttersnap on Unsplash

## Quick Recap

In Part 1, we talked about the DevOps culture and the foundations required:
[**How To Become a DevOps Engineer In Six Months or Less**
*NOTE: This is Part 1 of a multi-part series.*medium.com](https://medium.com/@devfire/how-to-become-a-devops-engineer-in-six-months-or-less-366097df7737)

In Part 2, we discussed how to properly lay the foundation for future code deployments:
[**How To Become a DevOps Engineer In Six Months or Less, Part 2: Configure**
*NOTE: this is Part 2 of a multi-part series. For Part 1, please click here.*medium.com](https://medium.com/@devfire/how-to-become-a-devops-engineer-in-six-months-or-less-part-2-configure-a2dfc11f6f7d)

In Part 3, we talked about how to keep your deployed code organized:
[**How To Become a DevOps Engineer In Six Months or Less, Part 3: Version**
*NOTE: this is Part 3 of a multi-part series. For Part 1, please click here. Part 2 is here.*medium.com](https://medium.com/@devfire/how-to-become-a-devops-engineer-in-six-months-or-less-part-3-version-76034885a7ab)

Here, we’ll talk about how to package your code for easy deployment and subsequent execution.

For reference, we are here in our journey:

![Package](https://cdn-images-1.medium.com/max/2000/1*uTJj1toNrJRl9f6qxR73rQ.png)*Package*

NOTE: You can see how every part builds on the previous part and lays the foundation for the subsequent part. This is important and is done on purpose.

The reason is, whether you are talking to your current or future employers, you have to be able to articulate what DevOps is and why it’s important.

And you do so by telling a coherent story — a story of how best to quickly efficiently ship code from a developer’s laptop to a money-making prod deployment.

Therefore, we are not after learning a bunch of disconnected, trendy DevOps tools. We are after learning a set of skills, driven by business needs, backed by technical tools.

And remember, each phase is about a month worth of learning, for a total of six months.

OK, enough chatter, let’s get to it!

## Primer on Virtualization

Remember physical servers? The ones you had to wait weeks to be PO-approved, shipped, data center accepted, racked, networked, OS-installed and patched?

Yeah, those. They came first.

Essentially, imagine if the only way to have a place to live is to build a brand new house. Need a place to live? Wait for however long it takes! Kind of cool since everybody gets a house but also not really because it takes a long time to build a house. In this analogy, a physical server is like a house.

Then people got annoyed about how long everything took and some really smart people came up with the idea of *virtualization*: how to run a bunch of pretend “machines” on a single physical machine and have each fake machine pretend to be a real machine. Genius!

So, if you really wanted a house, you could build your own and wait six weeks. Or you could go and live in an apartment building and share the resources with other tenants. Maybe not as awesome but good enough! And most importantly, there is no wait!

This went on for a while and companies (i.e. VMWare) made an absolute killing on this.

Until other smart people decided that stuffing a bunch of virtual machines into a physical machine is not good enough: we need **more **compact packing of **more **processes into **fewer **resources.

At this point, a house is too expensive and an apartment is still too expensive. What if I just need a room to rent, temporarily? That’s amazing, I can move in and out at a moment’s notice!

Essentially, that’s Docker.

NOTE: As of December 2018,

## Birth of Docker

Docker is new but the *idea* behind Docker is very old. An operating system called FreeBSD had a concept of [jails ](https://en.wikipedia.org/wiki/FreeBSD_jail)that dates back to 2000! Truly everything new is old.

The idea then and now was to isolate individual processes within the same operating system, in what is known as “operating system level virtualization.”

NOTE: This is not the same thing as “full virtualization”, which is running virtual machines side by side on the same physical host.

What does this mean in practice?

In practice, this means that the rise of Docker’s popularity neatly mirrors the rise of microservices — a software engineering approach where software is broken into many individual components.

And these components need a home. Deploying them individually, as stand-alone Java applications or binary executables is a huge pain: how you manage a Java app is different from how you manage a C++ app and that’s different from you manage a Golang app.

Instead, Docker provides a single management interface that allows software engineers to package (!), deploy and run various applications in a consistent way.

That is a huge win!

OK, let’s talk more about the pros and cons of Docker.

## Benefits of Docker

### Process Isolation

Docker allows every service to have full **process isolation**. Service A lives in its own little container, with all of its dependencies. Service B lives in its container, with all its dependencies. And the two are not in conflict!

Moreover, if one container crashes, only that container is affected. The rest will (should!) continue running happily.

This benefits security as well. If a container is compromised, it is extremely difficult (but not impossible!) to get out of that container and hack the base OS.

Finally, if a container is misbehaving (consuming too much CPU or memory) it is possible to “contain” the blast radius to that container only, without impacting the rest of the system.

### Deployment

Think about how the various applications are built in practice.

If it’s a Python app, it will have a slew of various Python packages. Some will be installed as *pip* modules, others are *rpm* or *deb* packages, and others are simple *git clone *installs. Or, if done with *virtualenv*, it will be a zip file of all the dependencies in the *virtualenv* directory.

On the other hand, if it’s a Java app, it will have a gradle build, with all of its dependencies pulled and sprinkled into appropriate places.

You get the point. Various apps, build with different languages and different runtimes pose a challenge when it comes to deploying these apps to prod.

How can we possibly keep all of the dependencies satisfied?

Plus, the problem is worse if there are conflicts. What if service A depends on Python library v1 but service B depends on Python library v2? Now there is a conflict since both v1 and v2 cannot co-exist on the same machine.

Enter Docker.

Docker allows not only for full process isolation but also for full **dependency isolation.** It is entirely possible and common to have multiple containers running side by side, on the same OS, each with its own and conflicting libraries and packages.

### Runtime Management

Again, how we manage disparate applications differs between applications. Java code logs differently, is started differently and monitored differently from Python. And Python is different from Golang, etc.

With Docker, we gain a single, unified management interface that allows us to start, monitor, centralize logs, stop, and restart many different kinds of applications.

This is a huge win and greatly reduces operational overhead of running production systems.

NOTE: As of December 2018, you no longer have to make a choice between fast startup of Docker and security of VMs. [Project Firecracker](https://thenewstack.io/how-firecracker-is-going-to-set-modern-infrastructure-on-fire/), courtesy of Amazon, attempts to fuse the best of both worlds. Still, this is a very new tech and not to be deployed to prod just yet.

However, as awesome as Docker is, it has downsides.

## Enter Lambda

First, running Docker is still running servers. Servers are brittle and flaky. They must be managed, patched and otherwise cared for.

Second, Docker is not 100% secure. Not like a VM, at least. There is a reason why huge companies that run hosted containers do so inside VMs, not on top of bare metal. They want fast startup times of containers and security of VMs!

Third, nobody really runs Docker as is. Instead, it is almost always deployed as part of a complex container orchestration fabric, such as Kubernetes, ECS, *docker-swarm* or Nomad. These are fairly complex platforms that require dedicated personnel to operate (more on these solutions later).

However, if I’m a developer, I just want to write code and have somebody else run it for me. Docker, Kubernetes and all that jazz are not simple things to learn — do I really have to?!

Short answer is, it depends!

For people who just want somebody else to run their code, [AWS Lambda](https://aws.amazon.com/lambda/) (and other solutions like it) are the answer:
> # AWS Lambda lets you run code without provisioning or managing servers. You pay only for the compute time you consume — there is no charge when your code is not running.

If you have heard of the whole “serverless” movement, this is it! No more servers to run or containers to manage. Just write your code, package it up in to a zip file, upload to Amazon and let them deal with the headache!

Moreover, since Lambdas are short lived there is nothing to hack — Lambdas are pretty secure by design.

Great, isn’t it?

It is but (surprise!) there are caveats.

First, Lambdas can only run for a max of 15 minutes (as of November 2018). This implies that long running processes, like Kafka consumers or number crunching apps cannot run in Lambda.

Second, Lambdas are Functions-as-a-Service, which means your application must be fully decomposed into microservices and orchestrated with other complex PaaS services like [AWS Step Functions](https://aws.amazon.com/step-functions/). Not every enterprise is at this level of microservice architecture.

Third, troubleshooting Lambdas are difficult. They are cloud-native runtimes and all bug fixing takes place within Amazon ecosystem. This is oftentimes challenging and non-intuitive.

In short, there is no free lunch.

NOTE: There are now “serverless” cloud container solutions as well. [AWS Fargate ](https://aws.amazon.com/fargate/)is one such approach. The mechanics are largely similar. In fact, if you are just starting out, I highly recommend giving Fargate a try — it’s an incredibly easy way to get containers running “the right way.”
> # UPDATE 1/13/2019: AWS just announced a significant price [reduction ](https://aws.amazon.com/blogs/compute/aws-fargate-price-reduction-up-to-50/)for Fargate, which now makes it a very attractive choice for running “serverless” containers.

## Summary

Docker and Lambda are two of the most popular modern, cloud-native approaches to packaging, running and managing production applications.

They are often complimentary, each suited for slightly different use cases and applications.

Regardless, a modern DevOps engineer must be well versed in both. Therefore, learning Docker and Lambda are good short- and medium-term goals.

NOTE: Thus far in our series we have dealt with topics that Junior to Mid-Level DevOps Engineers are expected to know. In subsequent sections, we will start discussing techniques that are more suited for Mid-Level to Senior DevOps Engineers. As always, however, there are no shortcuts to experience!

## Next

To learn all about the modern deployment best practices, read on!
[**How To Become a DevOps Engineer In Six Months or Less, Part 5: Deploy**
*Month 4*medium.com](https://medium.com/@devfire/how-to-become-a-devops-engineer-in-six-months-or-less-part-5-deploy-83e790545c23)
