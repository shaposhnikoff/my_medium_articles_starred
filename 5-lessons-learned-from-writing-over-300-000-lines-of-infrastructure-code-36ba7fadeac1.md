Unknown markup type 10 { type: [33m10[39m, start: [33m128[39m, end: [33m142[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m47[39m, end: [33m61[39m }

# 5 Lessons Learned From Writing Over 300,000 Lines of Infrastructure Code

A concise masterclass on how to write infrastructure code

![](https://cdn-images-1.medium.com/max/5760/1*WI6wrbABRbBflJq6NjNPYQ.png)

This October, I gave a talk at HashiConf 2018 where I shared 5 key lessons we learned at Gruntwork while creating and maintaining a [library of over 300,000 lines of infrastructure code](https://gruntwork.io/infrastructure-as-code-library/) that’s used in production by hundreds of companies. In this blog post, I’ll share with you the video and slides from the talk, as well as a condensed, written version of the 5 key lessons.

## Video and slides

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/RTEgE2lcyk4" frameborder="0" allowfullscreen></iframe></center>

<iframe src="https://medium.com/media/dd38bcd5571d092c8abe937cc87caf56" frameborder=0></iframe>

## Intro: DevOps is in the stone ages

Although the industry is full of cutting-edge buzzwords—Kubernetes, microservices, service meshes, immutable infrastructure, big data, data lakes, etc—the reality is that when you’re knee deep in building infrastructure, it doesn’t *feel* cutting edge.

To me, DevOps feels more like this:

![#thisisdevops](https://cdn-images-1.medium.com/max/2000/1*_NSJskuen6o3TVc6OFqrfQ.jpeg)*#thisisdevops*

![#thisisdevops](https://cdn-images-1.medium.com/max/2000/1*TyLmeTfiZshZfynREVK-pg.jpeg)*#thisisdevops*

![#thisisdevops](https://cdn-images-1.medium.com/max/2000/1*N-7cNi9SEOSp9PERacS2jw.jpeg)*#thisisdevops*

![(your company’s CI/CD pipeline) #thisisdevops](https://cdn-images-1.medium.com/max/2000/1*OWAKCmni2c4SviXJ-3PtpA.jpeg)*(your company’s CI/CD pipeline) #thisisdevops*

Building production-grade infrastructure is hard. And stressful. And time consuming. Very time consuming.

Here’s roughly how long you should expect your next infrastructure project to take, based on empirical data we’ve gathered while working with hundreds of different companies:

![](https://cdn-images-1.medium.com/max/3526/1*3sBV0ISBkG00eJ1LD505pA.png)

## Lesson 1: The Production-Grade Infrastructure Checklist

DevOps projects *always* take way longer than you expect. *Always*. Why is that?

Well, the first reason is [Yak Shaving](https://seths.blog/2005/03/dont_shave_that/), as perfectly illustrated in this clip from *Malcolm in the Middle:*

<iframe src="https://medium.com/media/f2750faadbdfd9f6ba1d66f8703f4fe3" frameborder=0></iframe>

The second reason is that building *production-grade infrastructure* (as in, the type of infrastructure you’d bet your company on) involves a thousand little details. The vast majority of developers don’t know what those details are, so when you’re estimating a project, you usually forget about number of critical—and time consuming—details.

To avoid this issue, every time you go to work on a new piece of infrastructure, go through the following checklist:

![](https://cdn-images-1.medium.com/max/2588/1*-nYI19LZZKDQjdAx0qhlFw.png)

Not every single piece of infrastructure needs every single item on the list, but you should consciously and explicitly document which items you’ve implemented, which ones you’ve decided to skip, and why.

## Lesson 2: the toolset

As of 2018, here are the primary tools we use at Gruntwork to build and manage infrastructure:

![](https://cdn-images-1.medium.com/max/3720/1*YfYyRXznTsu1DuaHHeD5Tg.png)

1. [**Terraform](https://terraform.io)**: We use Terraform to provision all the basic infrastructure, including networking, load balancers, databases, users, permissions, and all of our servers.

1. [**Packer](https://packer.io)**: We use Packer to define and build the virtual machine images we run on top of our servers.

1. [**Docker](http://docker.com)**: Some of our servers form clusters where we run applications as Docker containers. The main Docker cluster tools we use are [Kubernetes](https://kubernetes.io/), [ECS](https://aws.amazon.com/ecs/), and [Fargate](https://aws.amazon.com/fargate/).

Now, all of these tools are useful, but that’s not the real lesson here. The real lesson is that tools, by themselves, are not enough. You also need to change your team’s behavior.

In particular, the best tools in the world will not help your team one bit if your team isn’t bought into using those tools or if your team doesn’t have enough time to learn use those tools. Therefore, the key takeaway is that using infrastructure as code is an *investment*: there’s an up-front cost to get going, but if you invest wisely, you’ll earn big dividends over the long-term.

## Lesson 3: large modules considered harmful

Infrastructure as code newbies often define *all* of their infrastructure for *all* of their environments (dev, stage, prod, etc) in a single file or single set of files that are deployed as a unit. This is a Bad Idea.

Here are just a few of the downsides:

1. **Slow**: If all your infrastructure is defined in one place, running any command will take a long time. We’ve seen companies where terraform plan takes 5–6 minutes to run!

1. **Insecure**: If all your infrastructure is managed together, then to change *anything* you need permissions to access *everything*. That means that almost every user has to be an admin, which is another Bad Idea.

1. **Risky**: If all your eggs are in one basket, then a mistake *anywhere* could break *everything*. You might be making a minor change to a frontend app in dev, but due to a typo or running the wrong command, you delete the production DB.

1. **Hard to understand**: The more code you have in one place, the harder it is for any one person to understand it all. But if it’s all bundled together, the parts you don’t understand *could* hurt you.

1. **Hard to test**: Testing infrastructure code is hard; testing a large amount of infrastructure code is nearly impossible. We’ll come back to this point later.

1. **Hard to review**: The output of commands such as terraform plan becomes useless, as no one bothers to look through thousands of lines of plan output. Moreover, code reviews become useless:

<iframe src="https://medium.com/media/b726e0c9e3b9efaba21eb3da63079cc9" frameborder=0></iframe>

In short, you should build your code out of small, standalone, reusable, composable modules. This is not a new or controversial insight. You’ve heard it a thousand times before, albeit in slightly different domains:
> *“Do one thing and do it well”* —[*Unix Philosophy](https://en.wikipedia.org/wiki/Unix_philosophy#Do_One_Thing_and_Do_It_Well)*
> *“The first rule of functions is that they should be small. The second rule of functions is that they should be smaller than that.”*—Clean Code

## Lesson 4: infrastructure code without automated tests is broken

If your infrastructure code does not have automated tests, it’s broken. You just don’t know it yet. That said, testing infrastructure code is hard. You don’t really have “localhost” (e.g., you can’t deploy an AWS VPC on your laptop) and you don’t really have “unit tests” (e.g., you can’t isolate your Terraform code from the “outside” world as all Terraform does is talk to the outside world).

Therefore, to properly test your infrastructure code, you typically have to deploy it to a real environment, run real infrastructure, validate that it does what it should, and then tear it all down (for this style of testing, see [Terratest](https://blog.gruntwork.io/open-sourcing-terratest-a-swiss-army-knife-for-testing-infrastructure-code-5d883336fcd5), an open source library that includes tools for testing Terraform, Packer, and Docker code, working with AWS, GCP, and Kubernetes APIs, executing shell commands locally and on remote servers over SSH, and much more). What this means is that, with infrastructure testing, you have to slightly redefine terms:

![](https://cdn-images-1.medium.com/max/2984/1*clyUhTfMxCuIyvK347YXKA.png)

* **Unit tests** deploy and test one or a small number of closely related modules from one type of infrastructure (e.g., test the modules for a single database).

* **Integration tests **deploy and test multiple modules from different types of infrastructure to validate they work together correctly (e.g., test the modules of a database with the modules from a web service).

* **End-to-end (e2e) tests** deploy and test your entire architecture.

Note that the diagram is a pyramid, where we have lots of unit tests, a smaller number of integration tests, and a very small number of e2e tests. Why? Because of how long each type of test takes:

![](https://cdn-images-1.medium.com/max/2990/1*7kkzneYMMs0NnrQU4BSJTw.png)

Cycle time with infrastructure tests is slow, especially as you go up the pyramid, so you’ll want to catch as many bugs as you can as low in the pyramid as you can. That means you should:

1. Build small, simple, standalone modules (remember Lesson 3?) and write lots of unit tests for them to build your confidence that they work properly.

1. Combine these small, simple, battle-tested building blocks to create more complicated infrastructure that you test with a smaller number of integration and e2e tests tests.

## Lesson 5: the release process

Let’s now put everything in this talk together. Here’s how you’ll be building and managing infrastructure from now on:

1. Go through the [Production-Grade Infrastructure Checklist](#f769) to make sure you’re building the right thing.

1. Define your infrastructure as code using tools such as Terraform, Packer, and Docker. Make sure your team has the time to master these tools (see [DevOps Resources](https://gruntwork.io/devops-resources/)).

1. Build your code out of small, standalone, composable modules (or use the off-the-shelf modules in the [Infrastructure as Code Library](https://gruntwork.io/infrastructure-as-code-library/)).

1. Write automated tests for your modules using [Terratest](https://github.com/gruntwork-io/terratest).

1. Submit a pull request to get your code reviewed.

1. Release a new version of your code.

1. Promote that new version of your code from environment to environment.

![](https://cdn-images-1.medium.com/max/3722/1*w_io85Rnw-eUy8fziLX7pg.png)

*Get your DevOps superpowers at [Gruntwork.io](https://gruntwork.io/?ref=blog-lessons-learned-300k).*
