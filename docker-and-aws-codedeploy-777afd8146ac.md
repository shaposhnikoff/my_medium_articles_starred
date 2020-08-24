
# Docker and AWS CodeDeploy

TL:DR They’re both great, but Docker gives you the core of AWS CodeDeploy for free any on any platform.

It was unsurprising to hear that the thing people missed most after leaving Amazon was working with Apollo. Working with software through abstractions like environments and packages reduces complexity and helps you get more done. Recently Amazon made Apollo available for the public and called it [AWS CodeDeploy](http://aws.amazon.com/codedeploy/).

AWS CodeDeploy is a managed deployment technology. It provides great features like rolling deployments, automatic rollback, and load balancer integration. It is technology agnostic and Amazon uses it to deploy everything. It’s not a build or dependency management system, they have something else for that. Now that CodeDeploy is available to the public, former Amazonians can relive their former productivity glory. It is wonderful for working in the AWS ecosystem.

The defining features of CodeDeploy are the push/pull mechanism for deployment and the technology agnostic containers. Everything else that it offers is built on this core.

When the internet introduced me to Docker in 2013, I immediately recognized it as a partial replacement Apollo (now CodeDeploy). Docker is an open source project that provides the same core features. It provides all the primitives that you would need to build the right system for your environment.

It doesn’t have all the bells and whistles like centralized management, rolling deployments, and load balancer integration. Out of the box it doesn’t even have centralized configuration management. But those things are easy to bolt on and environment dependent.

What it gives you is a pull deployment mechanism, and technology agnostic container abstractions. These tools are enough to break down real productivity limits faced by software developers and system admins. Distribution communities like Docker Hub and Quay.io make it simple to find and use software already packaged to run in containers. The ecosystem of projects around Docker already provide much of what CodeDeploy provides and in a more generalized way.

I’m sure that as Docker grows and develops, many of these related projects will become redundant. I think that is perfectly healthy. You need that kind of choice in open source. I really enjoy the powerful abstractions and new primitives that Docker provides. I hope that however they grow, they keep to that strategy. Today we are living in a world where an independent engineer or hacker can be as productive as they would be at some of the most technologically advanced companies in the world. I think that is incredibly exciting.

As a former Amazonian, I haven’t missed Apollo all that much. I could always go back and use AWS CodeDeploy, but I want to work outside of AWS. I think CodeDeploy is a fantastic product, but I use a multi-cloud strategy. This makes Docker and the ecosystem of projects surrounding it more appealing.

If you want to learn how to use Docker and build powerful systems like AWS CodeDeploy, checkout my new book. [Docker in Action](http://manning.com/nickoloff/) is available for early access through the [Manning Early Access Program](http://manning.com/nickoloff/). Use promo code, “mlnickoloff” at checkout for a 50% discount.
