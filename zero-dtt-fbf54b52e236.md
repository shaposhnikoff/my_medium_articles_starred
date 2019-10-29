
# Is your deployment pipeline delivering on its zero downtime promise?

A story about an Angular application deployed in Kubernetes.

![Photo by [JJ Ying](https://unsplash.com/@jjying?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/7558/0*OZ9HvTz8VhwBXtdr)*Photo by [JJ Ying](https://unsplash.com/@jjying?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

It’s Tuesday morning and Ross was just starting off the day with a big cup of black coffee — such a stereotypical software engineer.

"Oh nice, seems like the team is making good progress. This Service Oriented Architecture is great for autonomous teams — it suits our backlog of low-coupled features very well," Ross thought as he was going through his GitHub notifications. Suddenly, Monica, the architect, approaches Ross.

Monica: "Ross, you know that application we are building for our flagship product?"

Ross: "Yes"

Monica: "As you know, it’s crucial for the company. We want to develop and deliver it in an iterative manner to get feedback as early as possible and show progress to the investors."

Ross: "Makes sense. What's the problem?"

Monica: "Well, we have got an automated pipeline to build, test, and deliver the app but I want to make sure the deployments do not affect the system — I don’t want the system to appear unavailable or misbehave when we deploy new versions."

Ross: "I see. What do you want me to focus on?"

Monica: "I’d like you to tackle the UI first. I won’t annoy you with the reasons but we cannot put the assets in something like a CDN so we decided to deploy them in our Kubernetes cluster. We need to make sure that new versions of the UI can be deployed with zero downtime."

Ross: "Got it. I am on it."

## What’s all the fuss around downtime during deployment?

Ross decides he wants to know more about the problem. Sounds like Monica is concerned about the availability of the system during deployments. But availability is not a new concern. Joey, a Software Reliability Engineer working in the same team, has brought the problem to their attention before, and put in place some [practices to define and observe it](https://medium.com/kudos-engineering/managing-reliability-with-slos-and-error-budgets-37346665abf6). Indeed, the UI repository has a document describing the SLO for the service and one of the objectives is about the availability of the system.

![SLI and SLOs for the service](https://cdn-images-1.medium.com/max/3604/1*EZQ4pokFmniZnRh7KgHMDA.png)*SLI and SLOs for the service*

Alright. That's nice, but it only tells us about the mean availability of the system, while what we really want is to focus on a particular point in the lifecycle of the service: *deployment*. In this situation we need a different SLI and SLO. Monica said there should be no downtime which directly translates to 100% of requests are successful during deployment.

Now we need to find a way to probe the system during deployment to verify that it meets the SLO — we need a Service Level Indicator. The UI service is a Single Page Application so a reasonable choice could be to request the main page with a certain throughput and wait for all the resources to be retrieved.

![](https://cdn-images-1.medium.com/max/3556/1*ZLJkoT1p-32_JDwdIwwEmw.png)

## Deployment Downtime Tests — DDT?

Ross starts looking into the service repository to see how the build and deployment of the application are structured. It includes:

* The source code for the Angular application.

* The pipeline script to build the application, package it in a Docker image, and deploy it in Kubernetes.

Ross decides he will make a change to the pipeline, decorating the deployment step with a pre and a post deployment step.

![Pipeline with Downtime Deployment Test](https://cdn-images-1.medium.com/max/2000/1*z6HzvqcBe7LbSn2kBQKJ4w.jpeg)*Pipeline with Downtime Deployment Test*

The pre-deployment step consists in starting a background task that will load the system with requests for the entire duration of the deployment. A request will ask for the Angular app and it will be considered successful if all the resources the page is composed of can be retrieved.

![](https://cdn-images-1.medium.com/max/3012/1*dLo0z_f-cmBiNz2EMaTSFA.png)

[Puppeteer](https://developers.google.com/web/tools/puppeteer) seems to be a good tool for the job since it allows to crawl SPAs. The script on the left could be used to generate traffic and express the two main requirements:

* Wait until all the resources are loaded (lines 17–19).

* Inspect the request completion status, reporting failures (lines 9–15).

Then the post-deployment step will stop the traffic generator and it will verify that no errors occurred. If so, the deployment is considered to be completed with zero downtime.

### Deploying a new version

Ross wants to rollout a new version of the system on his local Kubernetes cluster and see how the system behaves during deployment. He likes to use [minikube](https://kubernetes.io/docs/tutorials/hello-minikube/) since it's quite stable and well documented.

Before deploying, he needs to build the service. The build produces a docker image based on Nginx containing all the assets the application is composed of.

The deployment manifest is quite standard. It sets the number of replicas to 3 and uses rolling updates. From the kubernetes [documentation](https://kubernetes.io/docs/tasks/run-application/rolling-update-replication-controller/):
> To update a service without an outage, kubectl supports what is called [rolling update](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands/#rolling-update), which updates one pod at a time, rather than taking down the entire service at the same time.

Ross rolls out a new version of the system using the modified pipeline and he does not get a green flag from its deployment tests. Some requests aren't completed correctly, so it starts investigating. Kubernetes should guarantee no outages, shouldn't it?

![](https://cdn-images-1.medium.com/max/2000/1*lFeuvdn3G_Kd6IqV9dxg1w.jpeg)

It turns out that this is not Kubernetes' fault, that our user might experience downtime.

The problem is that the two versions of our system are not compatible with each other, and, during deployment, your requests might end up being served by one of the two versions. Since the pods have assets with different names they won't be able to serve requests for the previous version of the app.

Nice that we have these tests to spot the problem.

### How can we fix this?
> The root cause of the problem is that the interactions are not completely stateless.

Once the index file is served, subsequent requests are issued based on its version. As we have noticed, two different versions of the application coexist during deployment, so new pods need to be able to cope with this kind of traffic — they need to provide backward compatibility with at least one previous version to achieve zero downtime deployment.

Among the different options, Ross decides to fix the problem by creating a persistent volume in Kubernetes that each pod will mount so that every replica will have access to the same versions of the application.

![Pipeline that fixes the Deployment Downtime issue](https://cdn-images-1.medium.com/max/2000/1*dRQhff0OIgi0nVLstqMdkQ.jpeg)*Pipeline that fixes the Deployment Downtime issue*

When a new version is rolled out, first the new assets will be uploaded into the persistent volume and then the new version of the service will be deployed. Each version of the service contains only the index file that references the set of static assets in a particular version.

![](https://cdn-images-1.medium.com/max/2000/1*_mtH5lHO9Ws4bj0QrGbKSA.jpeg)

Ross implements his solution, runs the pipeline and finds out that he has solved the issue. He's now heading to Monica to present his findings.

## Conclusion

Tools like Kubernetes enable deployment of new system versions easily, frequently, and with no outages. However, we have shown how an Angular application could suffer downtime whilst the deployment is underway. The reason was that the application was relying on the file system to be in a particular state, i.e. storing the required assets. We had to modify the deployment pipeline to achieve zero downtime deployment. This highlights how stateful applications are not only the ones using a database, and that extra care is required during their deployment.

A tool, on its own, won’t guarantee zero downtime deployment and having some sort of tests around your assumptions, e.g. the downtime deployment test, helps to increase confidence that your applications is behaving as expected.
