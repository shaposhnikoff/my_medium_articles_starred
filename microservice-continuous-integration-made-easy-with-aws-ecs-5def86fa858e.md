
# Microservice continuous integration made easy with AWS ECS

Hi, we’re Airtime. Airtime lets real friends share real moments in real time through group video, messaging and more. Be together with your friends, wherever you are.

When we started thinking about re-launching [Airtime](http://www.airtime.com/) a few months ago, we had a unique opportunity where we could choose to stick with our current monolithic setup, or move in a different direction and rewrite. This warranted a hard look at our current architecture to identify pain points/challenges for our developers. Here’s what we learned were our biggest challenges:

* The application environment was inconsistent between local developer machines, and staging and production instances. This was due to the fact that hosts were updated using fragile Chef scripts on local developer machines which ran commands on remote instances via SSH. This technique gave every developer SSH access, resulting in servers riddled with local changes.

* The application design was monolithic which made it hard to optimize performance since each server ran a copy of each component despite the different components having vastly different networking, CPU, and memory requirements. Additionally, the monolithic design meant that issues in just one component could affect the performance and availability of the entire backend.

* Configuration management was a serious problem. Most configuration values could only be changed by redeploying the application using the Chef scripts mentioned above, which made it hard to recover when replacing a host.

* Deployments were often bottlenecked on the few people whose machines were currently in the right state to run the magic Chef scripts required to deploy the application. Since deployments were run by hand we couldn’t effectively gate them based on test results, allowing developers to accidentally deploy application revisions which didn’t pass tests. This made every deployment a “hold your breath” moment where it was hard to know whether it would go well, or whether everything would break.

Phew! But we can rebuild- we have the technology. So what did we want to do differently this time? A few goals:

* **User experience above all**: Deployments should be zero downtime, application servers should be redundant so that we can sustain losses without impacting experience, traffic should be distributed to reduce spikes in response rate, and we should have checks in place to prevent bad deploys that will break the application experience. If something bad does get deployed we should be able to rollback to a working state very quickly.

* **Developer efficiency**: Tests and deployments should be seamless and automatic. It should be as easy as possible to run an environment and test locally with confidence that your local environment is the same as staging and production. Additionally, it should be impossible to accidentally deploy code that does not pass tests.

* **Cattle, not pets: Automation is king**. Servers should be redundant, and replaceable, and setting them up and replacing them should be automatic.

* **Set ourselves up for success**: Get ready to scale! We should scale intelligently to support different amounts of traffic, and requests should be balanced between servers.

When we looked at our goals versus our issues with our previous setup, we chose to start again from the ground up. This meant keeping some of the core technology the same (Node.js, MongoDB, Redis), but moving from a large, monolithic application to container-based microservices. In terms of infrastructure, that meant sticking with AWS, but switching to a[ Docker](https://www.docker.com/)-based architecture, hosted by Amazon’s[ Elastic Container Service (ECS)](https://aws.amazon.com/ecs/).

**Why containers and ECS?**

The main selling point on containers for us was that a container was an atomic piece that had everything it needed for a service packaged inside: we could deploy the same container locally, on staging, and then on production, with the expectation that it would behave the same everywhere. They also reduce our need to have heavily customized servers, since most of the requirements are at the container level. From the deployment side, this means that a single service can be deployed individually, without disrupting the entire application.

Since we were heavy AWS users already, ECS was the logical choice for us; Integration with ELBs for distributing traffic behind containers, autoscaling for clusters, and Cloudwatch are easy to setup out of the box. Like many startups, we work with a small development team, and using ECS lets us share some of the responsibility for scheduling tasks, monitoring resources usage, and checking application health with AWS.

![Started from the bottom…](https://cdn-images-1.medium.com/max/NaN/0*B2ok-IuZi87EkWMC.)*Started from the bottom…*

![Now we’re here.](https://cdn-images-1.medium.com/max/NaN/1*tU6jwCJlxU-PozaNwjND4A.png)*Now we’re here.*

Each individual service is packaged as a separate Docker container, which is built, tested, and pushed to AWS [ECR (Elastic Container Registry)](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_Console_Repositories.html) via [CircleCI](https://circleci.com/). Our deployment script on CircleCI then makes a call to ECS to update that service with our changes.

With ECS, services are registered to [clusters](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_clusters.html). For us each ECS service is equivalent to one microservice, or a single container. There is more than one right way to do this in ECS, but we’ve gone with two separate clusters: staging, and production. Starting at the cluster level, it looks something like this:

![Staging cluster, showing the first 10 services registered](https://cdn-images-1.medium.com/max/NaN/0*-K9AWg0oQ_rwuSh5.)*Staging cluster, showing the first 10 services registered*

![Cluster host, showing the service revisions registered there](https://cdn-images-1.medium.com/max/NaN/0*x13iUNbNjgUYUAQZ.)*Cluster host, showing the service revisions registered there*

Each service has a couple of things: the number of copies of the task we want to have running (“Running Tasks”), the number of copies of the task that are currently running (“Desired Count”), and the current revision of the service (“Task Definition Name: Revision”).

ECS services are defined by a “[Task Definition](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_defintions.html)”. This defines things like environment variables, the [container image](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) you wish to use, and the resources you want to allocate to the service (port, memory, CPU). You can see an abbreviated version of one of our Task Definitions here:

![Abbreviated Task Definition](https://cdn-images-1.medium.com/max/NaN/0*scYrUN7nsZUXIff7.)*Abbreviated Task Definition*

In our case, Task Definitions are created, iterated, and registered to the service through our deployment process.

Our test and deployment process starts with developers working locally through [Vagrant](https://www.vagrantup.com/) (more on that in a future post). They can pull containers from ECR, develop and test features locally, and test feature branches on CircleCI. Once their changes have been made, they open a pull request against develop.

![Merging a feature branch into develop](https://cdn-images-1.medium.com/max/NaN/0*vGzSlnBz51sd_KTq.)*Merging a feature branch into develop*

We’ve gone the continuous integration/continuous deployment route, so changes to a develop or master branch are deployed automatically once tests pass. A typical develop build looks like this:

![Login to our container registry](https://cdn-images-1.medium.com/max/NaN/0*wRopoCZ4GiGXtCtd.)*Login to our container registry*

![Build container from service directory on CircleCI](https://cdn-images-1.medium.com/max/NaN/0*kPzJBIPsT_va--Y3.)*Build container from service directory on CircleCI*

![Run service container and test against it](https://cdn-images-1.medium.com/max/NaN/0*BUweqF5DCH7zcHGa.)*Run service container and test against it*

![Tag our new container with develop, so developers can quickly pull locally](https://cdn-images-1.medium.com/max/NaN/0*q9taG1jFtr863BQw.)*Tag our new container with develop, so developers can quickly pull locally*

![Call master repo to kick off deploy](https://cdn-images-1.medium.com/max/NaN/0*tFLi9cCzQbt3pSRQ.)*Call master repo to kick off deploy*

Triggering a deployment kicks off our deploy script, which handles the actual ECS updates:

![Step #1, create the Container Definition](https://cdn-images-1.medium.com/max/NaN/0*hhSmX3PA8Etah9qq.)*Step #1, create the Container Definition*

![Step #2: create Task Definition: this includes the Container Definition from Step #1](https://cdn-images-1.medium.com/max/NaN/0*bf1gfsHJ22kUyFGi.)*Step #2: create Task Definition: this includes the Container Definition from Step #1*

![Step #3: register our new Task Definition to the proper cluster and service](https://cdn-images-1.medium.com/max/NaN/0*TG9eYZLISITtZCao.)*Step #3: register our new Task Definition to the proper cluster and service*

After our ECS updates, we go for the bonus round: asking Hubot to run integration tests, posting to Slack that a new deployment has gone out, and registering our deployment in NewRelic.

![](https://cdn-images-1.medium.com/max/NaN/0*qb9pN6FA7a54ejW6.)

The goal with ECS and our deployment choices was to keep the end user experience as consistent as possible. Most significantly, we utilize connection-draining with ECS to do zero downtime deployments: connections to old tasks are drained off slowly, and not switched over to the new revision until the new version passes ELB health checks.

![Old connections are drained off, and tasks revisions gracefully switch over.](https://cdn-images-1.medium.com/max/NaN/0*ltHEW0cE2DAzNNkA.)*Old connections are drained off, and tasks revisions gracefully switch over.*

We can also autoscale pretty seamlessly here to keep up with demand: we’ve added rules at the cluster level to add additional hosts when our cpu usage breaches 85% usage across the cluster:

![Autoscale the ECS clusters when we need more resources](https://cdn-images-1.medium.com/max/NaN/0*wecznIfMHcrV-ZYz.)*Autoscale the ECS clusters when we need more resources*

We scale at the service level with the native ECS task autoscaling (formerly we achieved this with a combination of CloudWatch, SNS, and Lambda). Service autoscaling lets us watch for a Cloudwatch metric like a high number of a messages in a queue, and scale a service accordingly.

![Service scaling on queue length](https://cdn-images-1.medium.com/max/NaN/1*7TshyHDLqpKDN8WzhS4N8Q.png)*Service scaling on queue length*

Remember our list of goals? Let’s see how we’re doing.

**User experience above all: **most important here is user experience, and keeping it as consistent as possible. Almost every service is registered to an ELB through ECS, which lets us balance traffic evenly between containers. As we scale, new copies of the containers are registered to the ELB, so we can evenly distribute traffic as we scale to handle more requests:

![Response time stays consistent as we scale.](https://cdn-images-1.medium.com/max/NaN/1*oq8B670BRe2J-xSyRf3kdw.png)*Response time stays consistent as we scale.*

Rolling deployments, connection-draining, and health checks let us release new features without disrupting current connections, and minimize deploying revisions that will break the application. We gate deploys in two places: first at the test level, where CircleCI will fail the build with a non-zero exit code, and again at the ECS level, where the ELB health checks the container before draining connections:

![A service fails healthchecks, and doesn’t deploy.](https://cdn-images-1.medium.com/max/NaN/1*UGxGpLh-YYAPRaOimgLI4w.png)*A service fails healthchecks, and doesn’t deploy.*

![A service fails tests, and fails to make the API call to ECS.](https://cdn-images-1.medium.com/max/NaN/1*x_zX96DxUZB9g0nDFHFoiw.png)*A service fails tests, and fails to make the API call to ECS.*

**Set ourselves up for scaling success**: the ECS integration with autoscaling has also been big for consistency: as CPU or request volume spikes, we can add more containers and/or hosts to handle the load:

![As the request volume increases, we can add additional containers and hosts.](https://cdn-images-1.medium.com/max/NaN/1*z4fAIB8W0PI5P3XK5ToJcw.png)*As the request volume increases, we can add additional containers and hosts.*

![Autoscaling in action.](https://cdn-images-1.medium.com/max/NaN/1*vE4aEQ-wxH5rWj05plKuFg.png)*Autoscaling in action.*

**Developer efficiency: **The new setup allows engineers to work on features, and get them deployed live without waiting 5–6 days for someone to deploy for them. Previously, a single, major deploy happened once a week, which could cause issues when many things were updated at once. Now, we deploy many, many times a day, which lets us make smaller, atomic changes that affect only a single service, and get features to users faster:

![Deploys on deploys on Deploys](https://cdn-images-1.medium.com/max/NaN/1*UJmce6stl0jnM6LBpyASkw.png)*Deploys on deploys on Deploys*

We are also developing locally with the exact same container versions that we use on staging or production, which leads to less deployment issues. From our Ansible-based Vagrant provisioner:

![Local environment pulls straight from ECR](https://cdn-images-1.medium.com/max/NaN/1*sk6xD6YnjU1wHUimSk6K-A.png)*Local environment pulls straight from ECR*

**Cattle, not pets (automation is king)**: We’ve tried to make our infrastructure as automated, and replaceable as possible: ECS cluster hosts can be terminated, and will be automatically replaced, and each service runs a minimum of two copies, spread across multiple availability zones. We provision new servers at boot from user-data, rather than maintaining a custom AMI- since most our customizations are at the container level, we only need minimal setup at launch, like starting our logging container.

Bonus: since we can allocate resources more efficiently through ECS by having multiple services/containers share a host, and we scale up and down to use infrastructure only as necessary, we’ve been able to reduce our AWS bill significantly.

Big challenges and improvements coming up as we grow, though: more custom monitoring with CloudWatch, faster new instance launch times, and faster container build times.

Update 07/07/2017: awesome follow up piece to this by [Nathan Peck](https://medium.com/@nathan_peck) on Airtime’s architecture is here: [https://techblog.airtime.com/microservice-software-architecture-at-airtime-d5f8f02c8943#.5ofps9o8h](https://techblog.airtime.com/microservice-software-architecture-at-airtime-d5f8f02c8943#.5ofps9o8h)

*❤ what we do? Come [join us](https://airtime.com/about#join-us).*

*Originally published at [techblog.airtime.com](https://techblog.airtime.com/microservice-continuous-integration-made-easy-with-aws-ecs-10d470e31af0) on May 26, 2016.*
