
# How To Become a DevOps Engineer In Six Months or Less, Part 5: Deploy

What typical code deployments look like

## Quick Recap

Let’s quickly take stock on where we are in our DevOps journey.

In Part 1, we talked about the DevOps culture and the foundations required:
[**How To Become a DevOps Engineer In Six Months or Less**
*NOTE: This is Part 1 of a multi-part series.*medium.com](https://medium.com/@devfire/how-to-become-a-devops-engineer-in-six-months-or-less-366097df7737)

In Part 2, we discussed how to properly lay the foundation for future code deployments:
[**How To Become a DevOps Engineer In Six Months or Less, Part 2: Configure**
*NOTE: this is Part 2 of a multi-part series. For Part 1, please click here.*medium.com](https://medium.com/@devfire/how-to-become-a-devops-engineer-in-six-months-or-less-part-2-configure-a2dfc11f6f7d)

In Part 3, we talked about how to keep your deployed code organized:
[**How To Become a DevOps Engineer In Six Months or Less, Part 3: Version**
*NOTE: this is Part 3 of a multi-part series. For Part 1, please click here. Part 2 is here.*medium.com](https://medium.com/@devfire/how-to-become-a-devops-engineer-in-six-months-or-less-part-3-version-76034885a7ab)

In Part 4, we talked about how to package your organized code for easy deployment:
[**How To Become a DevOps Engineer In Six Months or Less, Part 4: Package**
*Quick Recap*medium.com](https://medium.com/@devfire/how-to-become-a-devops-engineer-in-six-months-or-less-part-4-package-47677ca2f058)

For reference, this is where we are on our map:

![Mental map of our journey](https://cdn-images-1.medium.com/max/2890/1*hJB9XRlPJWkJc1ouRGDOlw.png)*Mental map of our journey*

Therefore, if you spend about a month on each section, we are in month 4 at this point.

So, we know how to provision infrastructure that will run our software, we know how to version it appropriately, and we know how to package it for deployment.

Finally, we will discuss how to actually deploy your code!

## Code Deployment

Have you noticed how right above I didn’t actually say, “how to deploy your code easily”? That’s for a reason.

Unfortunately, proper code deployment from a dev environment to prod is still a painful process, fraught with errors and failures.

Why is that?

Reasons are numerous, of course, but in my opinion it mostly comes down to differences.

Specifically, differences between environments where code is created and where it actually runs.

In fact, I would argue minimizing those differences is the single biggest improvement you can make not just in your overall code deployment but also post deployment run-time.

So, how do we go about reducing and/or eliminating the differences between our prod and non-prod environments?

## It Worked On My Machine!

If your dev infrastructure looks like this

![Assembled by hand with tender love and care](https://cdn-images-1.medium.com/max/4096/1*nzcf0xP9UShiaXEXB0SQKg.jpeg)*Assembled by hand with tender love and care*

But your prod infrastructure looks like this

![Prod](https://cdn-images-1.medium.com/max/2000/1*0sdyinHewLDh9f5X2E4hDg.jpeg)*Prod*

then you have a problem.

If you are utilizing [infrastructure-as-code](https://medium.com/@devfire/how-to-become-a-devops-engineer-in-six-months-or-less-part-2-configure-a2dfc11f6f7d) instead of configuring things by hand, then you are 90% of the way there.

If you are not, don’t despair — you are not alone. Take an afternoon, identify what gaps you have (training, culture, people, processes, etc.) and methodically eliminate them one by one.

Bottom line is that you are not going to be successful in managing a modern tech stack if you are still configuring things by hand. It’s as simple as that.

So, first thing you need to do is make sure **everything **that touches prod is a versioned artifact, deployed by your deployment server.

Assuming that’s done, I will argue the best way to deploy code is not to deploy it at all.

## Modern Approaches to Code Deployment

That’s right — deploying code to prod machines is very 1990s.

![State of the art code deployment apparatus](https://cdn-images-1.medium.com/max/2000/0*dEje7JgKUoxD_pQq.jpg)*State of the art code deployment apparatus*

The biggest problem with deploying code to a set of fixed production machines is that by definition, your prod servers (where code runs) are different from your dev servers (where code is written).

So, it’s no wonder a ton of issues arise immediately post deployment that were never seen before — everything is different!

Therefore, you need to do all you can to make sure your deployment artifact is the **entire runtime**, not a piece of code.

In other words, deploy your code **once** to your dev environment, clone the entire machine your code runs on and then copy it everywhere it needs to go.

This is known as “immutable deployment” and is a very powerful pattern that will save you hours of post-deployment headaches.

Of course, if you run containers, same idea applies: you deploy the same container everywhere.

“But wait! My prod **is **different from dev!” you might say. Database usernames/passwords, connection strings, S3 bucket locations, etc. These are all different!

Yes, they are different.

The way to solve that problem is with the 12 factor app [config](https://12factor.net/config) principle. All your configuration needs to be externalized and passed as environment variables to your machine.

For example, if you are in AWS, use SSM as the external parameter store — it integrates nicely with CloudFormation. It is also super easy to set environment variables directly from the *aws ssm *cli commands. Of course, other cloud providers have similar mechanisms.

Moreover, resist the urge to “fix” your prod machines when things go wrong. The machines are **immutable** and that means whatever fixes you make must come from dev.

In fact, your goal should be NO access allowed to prod machines at all. No ssh, no scp, no prod access to anybody. Not you, not aspiring hackers.

But what if I need logs to troubleshoot the problem?

You guessed it — your logs should also be externalized, ideally shipped elsewhere either with an ElasticSearch/Logstash/Kibana (ELK) stack or commercial software like SumoLogic or Datadog.

Whatever you do, your prod machines are “cattle” — they are replaced at the slightest sign of being unhealthy. They are not “pets”, to be nursed back to health by spending hours on troubleshooting efforts.

NOTE: Yes, I know this analogy is overused and I hear from people who actually take care of cattle that this is not how this works, exactly, but the point stands. Don’t “fix” your prod machines, fix your dev and redeploy.

## Mechanics of Code Deployment

OK, so you know what to do but how to do it?

Unfortunately, this is where [Jenkins](https://jenkins.io/) comes in. If you don’t know, Jenkins is one of the most popular open-source deployment automation servers.

Now, I say “unfortunately” because Jenkins (and its predecessor, Hudson) have been around for almost a decade. And it shows.

It is complex to setup, even more complex to maintain. It comes with millions of plugins of dubious quality. These plugins tend to break at the most inopportune times, taking the whole thing down with them.

In fact, truly resilient, multi-master Jenkins setups are rare and are typically seen only by the largest organizations.

So, why am I recommending that you start with Jenkins?

Because despite all of its flaws, it is still extremely popular and is heavily used in our industry.

Knowing Jenkins, specifically how [*Jenkinsfile](https://jenkins.io/doc/book/pipeline/jenkinsfile/)* is structured is a huge benefit to your employment prospects and cannot be overlooked.

That said, when you learn Jenkins, make sure you follow the newer Pipeline [BlueOcean](https://jenkins.io/doc/book/blueocean/) path, not the older “Jenkins jobs” path.

This is critically important since you want your CI/CD pipeline to live right inside your code GitHub/GitLab repo. That way pipeline itself is a versioned piece of code!

In fact, this is so important that it bears repeating again.

**Everything is code.**

Your application, how it is deployed, how it is monitored, how it is configured, etc. — all pieces of code, stored in GitHub/GitLab/Whatever, properly versioned.

The goal here is to create a truly frictionless environment for the core developers (software engineers who write feature code.)

For example, I should be able to write my little microservice, add whatever tests I deem necessary, add a Jenkinsfile, add monitoring-as-code configuration, specify my parameters in some “env.yaml” file, store it all in one repo, have Jenkins auto-discover said repo, build it, test it, deploy it (as canary or blue/green) and send me an email when it’s done!

Whew.

That is the goal! In fact, this is the very **essence** of DevOps engineers’ core mission.

## Alternatives to Jenkins

Like I said earlier, Jenkins has been around since forever and there are now other, in my opinion better, even if less popular alternatives.

One is AWS’ own [CodeDeploy](https://aws.amazon.com/codedeploy/) service. It has limitations but the developers behind CodeDeploy have made significant improvements over the past year and if you are in AWS, I strongly urge you to give this a try.

Another is GitLab CI. If your organization runs GitLab, you should probably start there since it is neatly integrated with the rest of GitLab.

Finally, GitHub announced [Actions](https://developer.github.com/actions/), which is supposed to be their own automation.

Really, I don’t think tools matter all that much here. What matters is knowing that everything, including your code deployment pipelines, are versioned artifacts and nothing goes to prod unless it comes from dev first.

Regardless, if you are starting with Jenkins, try to set it up as a container. It is not super difficult and will be an awesome learning opportunity to figure out how to deploy a containerized Jenkins server, with elastic, containerized Jenkins worker nodes.

In fact, you can start simple, without any container orchestration, which is the subject of the next post. Stay tuned!
