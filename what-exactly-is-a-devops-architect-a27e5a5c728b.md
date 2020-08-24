
# What Exactly Is a DevOps Architect?

The difference between automating and empowering an organization

![Photo by [Christina @ wocintechchat.com](https://unsplash.com/@wocintechchat?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/servers?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/12032/1*qN8jbgSpsi-uotXOMk7azA.jpeg)*Photo by [Christina @ wocintechchat.com](https://unsplash.com/@wocintechchat?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/servers?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

Many organizations across the world now have a few employees with the job title *DevOps engineer* in their IT department.

This has opened many exciting and new career paths for traditional system administrators and engineers to continue to grow their skills, build their careers, and increase their pay.

But what exactly is a DevOps architect and how is that different from a DevOps engineer? Here are the three key things that distinguish the two:

1. The architect understands the full software development lifecycle and knows how to integrate all aspects of it into a CI/CD pipeline across multiple tools, deployment environments, and technologies. This is true of both application code and infrastructure-as-code.

1. Knowledge of deployment patterns and site-reliability engineering skills is vital for a successful DevOps architect. Every organization has a different environment to account for so the architect must be able to come up with an automated deployment pattern that works, allows for rollbacks, and doesn’t introduce downtime with a new change.

1. Thought leader and visionary for the organization. DevOps is a journey, not a destination. The architect must be able to guide an organization through this transformational change. This means building strong relationships across the organization, gaining trust in their peers, and empowering developers, operations, security, and project management teams to automate their work.

## **The DevOps Architect Skillset**

![The DevOps architect requires a wide variety of skills](https://cdn-images-1.medium.com/max/2000/1*_ltCZnV4LTJecinxPOfIhw.png)*The DevOps architect requires a wide variety of skills*

DevOps encompasses the entire software development lifecycle from writing that first line of code to decommissioning a legacy application. A successful DevOps architect is able to speak the language of both application developers and operations teams.

### **Development experience**

**DevOps engineers are traditionally people with an operations background**

I can’t stress enough how important it is for those aspiring to grow into an architect role to gain some development experience. I made this transition from developing chef’s cookbooks with Ruby to building Ruby on Rails web-apps on the side.

**Version and dependency management**

You quickly learn that when writing infrastructure or configuration as code, you’ll need to have a system of versioning in place or else things will quickly become very brittle.

You might also start pulling in other packages/roles/cookbooks/etc. and experience “dependency hell.” These are common issues developers face and have to solve.

**Building and testing code**

The need to write automated tests** **and run those tests during a CI process is vital.

It does not matter if you are writing [Ansible roles](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html) or chef cookbooks or something else. Automated tests are the only way to ensure the confidence needed for automated deployments.

Let’s also not forget about the need to bring in our security experts into the fold, early on in a software project. We don’t want to get to the end of development, be ready to deploy, only to have to refactor code to fix a major security flaw in the design.

## **Deployment Patterns and SRE Skills**

![Source: [https://electric-cloud.com/blog/key-takeaways-continuous-discussions-c9d9-episode-49-advanced-deployment-patterns/](https://electric-cloud.com/blog/key-takeaways-continuous-discussions-c9d9-episode-49-advanced-deployment-patterns/)](https://cdn-images-1.medium.com/max/2000/0*4YiaFugm0cY4AsRH.png)*Source: [https://electric-cloud.com/blog/key-takeaways-continuous-discussions-c9d9-episode-49-advanced-deployment-patterns/](https://electric-cloud.com/blog/key-takeaways-continuous-discussions-c9d9-episode-49-advanced-deployment-patterns/)*

One of the biggest challenges facing organizations today is how to do continuous deployments. There are a number of different approaches to take, some of which are:

* Nuke and pave (bad idea for production deployment)

* Canary

* Feature flagging

* Incremental/rolling deployment

* Blue/green

Each of these has pros and cons. They each require a different lens to write either your infrastructure-as-code or application code.

For example, if I’m going to do feature flagging for introducing new features, I need to write my application code to support that.

However, if I’m going to introduce new features with a blue/green deploy, I need to focus a lot more on ensuring my infrastructure-as-code is solid and my networking/DNS/load balancing can swap between deployments.

A DevOps architect can help navigate application teams through those options and recommend the best approach based upon the team’s existing skills, technology, and DevOps maturity.

## **Thought Leader and Visionary**

![Source: [https://twitter.com/vladobotsvadze/status/1186534907377524736?lang=da](https://twitter.com/vladobotsvadze/status/1186534907377524736?lang=da)](https://cdn-images-1.medium.com/max/2000/0*-ElEAtHR80sKcF5u.jpg)*Source: [https://twitter.com/vladobotsvadze/status/1186534907377524736?lang=da](https://twitter.com/vladobotsvadze/status/1186534907377524736?lang=da)*

DevOps is not a single technology, tool, process, or person. It is a journey and for large organizations, a difficult and long one. It takes someone who can persistently and persuasively enable change within an organization.

This starts small, within a single team, by improving development processes or simply utilizing [Jira](https://www.atlassian.com/software/jira) for tracking development progress.

A DevOps architect has to look at an organization, find key people who can be those change agents, break down the silos, and bring them together to make something new happen.

The biggest challenge for DevOps architects is leading without formal authority. This means getting people to help you achieve something even though it may not be something they traditionally do or think they have to.

The only way to do this is to be able to be a salesperson. You have to convince individuals that doing something new will make life better for them and for the organization.

That means taking input from multiple perspectives, getting buy-in for a solution, and then holding yourself and others accountable for getting it done.

## Conclusion — Are You Ready to Be an Architect?

![Source: [https://intland.com/codebeamer/devops-it-operations/](https://intland.com/codebeamer/devops-it-operations/)](https://cdn-images-1.medium.com/max/2044/0*L94oVx2n5o9c35fX.png)*Source: [https://intland.com/codebeamer/devops-it-operations/](https://intland.com/codebeamer/devops-it-operations/)*

DevOps is such a large term for software development in this new age. It encompasses so many different skills since it encompasses every aspect of development.

The great news is that to move from a DevOps engineer to a DevOps architect *you don’t have to be an expert in everything!*

If you’re already modeling the way for your team, start to include people from other teams in what you are doing. Build a DevOps community within your organization.

Get some of that exposure to others that specialize in things you aren’t that familiar with, such as sitting with a developer for a day and see what they do, or vice-versa.

A DevOps architect role is a huge responsibility but is also greatly rewarding. The greatest thing about working in such a role is to look back at all the transformational changes you helped influence within an organization.
