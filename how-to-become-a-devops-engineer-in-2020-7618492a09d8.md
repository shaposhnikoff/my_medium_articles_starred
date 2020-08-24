Unknown markup type 10 { type: [33m10[39m, start: [33m342[39m, end: [33m346[39m }

# How to Become a DevOps Engineer in 2020

Your guide to getting started in a career in DevOps

![Source: [https://www.scnsoft.com/blog/how-to-become-a-devops-engineer](https://www.scnsoft.com/blog/how-to-become-a-devops-engineer)](https://cdn-images-1.medium.com/max/2000/0*PvR5JPF50QYNi-3L.png)*Source: [https://www.scnsoft.com/blog/how-to-become-a-devops-engineer](https://www.scnsoft.com/blog/how-to-become-a-devops-engineer)*

The 2010s saw the rise of the DevOps engineer. DevOps engineers have now planted their flag in the industry and are here to stay. Interested in becoming a DevOps engineer? Here are the five things you need to do to get that title:

1. **Have a developer mindset**. You’ll be managing something-as-code, so you need to look at challenges and problems through the lens of a developer. Learn Git and write code that's maintainable for years to come.

1. **Gain foundational information in system engineering**. Understand three-tier application architecture. Be able to explain basic system administration tasks (and how to automate them). Learn the basics of Linux.

1. **Be able to talk about experience in the cloud. **Nearly every company at this point is adopting the cloud in some form or fashion. This also gives you the ability to prove your experience with configuration-as-code.

1. **Know something about containers.** You don’t need experience in full-blown [Kubernetes](https://kubernetes.io/), but you do need to know what a container is. Focus on the use of containers during CI builds.

1. **Yes,** **soft skills are important.** DevOps is not just tooling and technology but also culture. To be a successful DevOps engineer, you’ll need to be able to make friends and influence people.

![Source: [https://www.techrepublic.com/](https://www.techrepublic.com/)](https://cdn-images-1.medium.com/max/2000/0*qLEYzpURuIfOi7SX.png)*Source: [https://www.techrepublic.com/](https://www.techrepublic.com/)*

## 1. Have a Developer Mindset

Do you need to know Java or .NET development in order to be a DevOps engineer? Absolutely not! When interviewing potential DevOps engineers, I’ve seen too many lack confidence when explaining development concepts such as Git, pull requests, and SDLC. Be able to answer basic questions regarding those topics.

Also, realize that when you write scripts or use a tool such as Ansible, Chef, or Terraform, you are in fact writing code. You need to write tests for your code! How else can you be confident that the code you write actually works?

### On maintainability

The difference between someone learning DevOps and someone who is practicing DevOps is one who can use whatever tool to write code that is maintainable. This isn’t just some one-off task you are scripting for. Know how you can write your code (and use variables) so that it can be re-used and/or refactored without needing to be completely redone.

![Source: [https://ops.fhwa.dot.gov/plan4ops/sys_engineering.htm](https://ops.fhwa.dot.gov/plan4ops/sys_engineering.htm)](https://cdn-images-1.medium.com/max/2000/0*wcClDL4gpamcWDdi.png)*Source: [https://ops.fhwa.dot.gov/plan4ops/sys_engineering.htm](https://ops.fhwa.dot.gov/plan4ops/sys_engineering.htm)*

## 2. System Engineering

This is mostly for developers interested in learning more about the operations side of things. You should gain experience in and understanding of how operating systems work with middleware. What are some of the key things to configure in both? How does network traffic flow from the browser to the application server? What is a three-tier application architecture?

When you understand what things need to be configured in a system, only then can you start to configure them as code. Knowing which things might change often or things that could be different between different applications allows you to know what to expose as variables in your code vs. what can be hardcoded.

![Source: [https://timesofcloud.com/cloud-tutorial/history-and-vision-of-cloud-computing/](https://timesofcloud.com/cloud-tutorial/history-and-vision-of-cloud-computing/)](https://cdn-images-1.medium.com/max/2048/0*eRmzokrTvkTCQqbl.png)*Source: [https://timesofcloud.com/cloud-tutorial/history-and-vision-of-cloud-computing/](https://timesofcloud.com/cloud-tutorial/history-and-vision-of-cloud-computing/)*

## Cloud Computing

I expect nearly everyone reading this article knows what I’m talking about when I say *the cloud*. Most enterprises have adopted Azure or AWS or even both as their cloud computing vendor. I would not hire a DevOps engineer if they didn’t have at least some experience with the cloud. If you aren’t using either in your company right now, create your own account and start playing with some services. You don’t need to understand every AWS service that’s available, but you do need to know enough about the basics to speak about them, what they do, and why they are important.

In addition, talk about your experience with using an infrastructure-as-code tool, such as [Terraform](https://www.terraform.io/docs/cloud/index.html) or [Cloudformation](https://aws.amazon.com/cloudformation/). Without configuring your cloud environment as code, you’re gonna have a bad time.

![Source: [https://www.docker.com/resources/what-container](https://www.docker.com/resources/what-container)](https://cdn-images-1.medium.com/max/2306/0*ZviAhlo6XdTqdI3k.png)*Source: [https://www.docker.com/resources/what-container](https://www.docker.com/resources/what-container)*

## Containers

The use of containers has become a bit controversial given the rise of serverless applications. However, for most enterprises, not every application will be able to move to be 100% serverless. There will be middleware or stateful aspects that need to run in a container. You don’t need to become a Kubernetes expert by any means to become a DevOps engineer. Again, for most enterprises, full-blown K8 is a challenge for a couple of years down the road. Just the ability to install Docker and run containers in a corporate environment is what many are struggling with today.

Because containers give your developers the ability to run their full application stack locally, being able to build containers for your applications and use them is vital to truly being a DevOps shop. In addition, speeding up CI builds by using containers for a build job is vital. Why manage and patch versions of Java on a VM for building .jar files when you can just use a container? In addition, know how to use these containers in your CI tool of choice ([Jenkins](https://jenkins.io/), [Gitlab](https://about.gitlab.com/), etc,).

![Source: [https://www.wikijob.co.uk/content/interview-advice/competencies/soft-skills](https://www.wikijob.co.uk/content/interview-advice/competencies/soft-skills)](https://cdn-images-1.medium.com/max/3808/0*Bjyxjy97BAsTKaCf.png)*Source: [https://www.wikijob.co.uk/content/interview-advice/competencies/soft-skills](https://www.wikijob.co.uk/content/interview-advice/competencies/soft-skills)*

## Soft Skills

This might be the most difficult skill set or aspiring engineers, and the most often overlooked. DevOps is very new to a lot of organizations and this requires a lot of training and “upskilling.” Being able to communicate effectively, work across business silos, and partner with different people in your organization is the only way to be successful.

I’ve learned the hard way that putting your foot in the ground and deciding you know the best way to do something will not influence change for most people. In fact, you’ll create more barriers to accomplishing what you are trying to do.

Listen and be able to understand where someone else is coming from. Also, don’t feel the need to solve every problem. The most effective DevOps engineer can influence and empower someone else to automate their own issue.

## Additional DevOps Training Resources

* Read the book [The Phoenix Project](https://www.goodreads.com/book/show/17255186-the-phoenix-project)

* Take this free [DevOps pre-requisite course](https://www.udemy.com/course/learn-devops/)

* Watch this video by Rackspace that explains the [meaning of DevOps in simple English](https://youtu.be/_I94-tJlovg).

* Read these interesting answers on Quora by experts on [becoming a good DevOps engineer](https://www.quora.com/How-do-I-be-a-good-DevOps-engineer)

* Watch and learn side-by-side with this [Docker for beginners full, free course](https://youtu.be/zJ6WbK9zFpI) by Mumshad (One of the top Udemy instructors)

* Join developer community forums like [dev.to](https://dev.to/), [Hashnode](https://hashnode.com/), [Dzone](https://dzone.com/), [DevOps subreddit](https://www.reddit.com/r/devops/), [Stack Overflow](https://stackoverflow.com/), [DevOps StackExchange](https://devops.stackexchange.com/), [Changelog](https://changelog.com/)
