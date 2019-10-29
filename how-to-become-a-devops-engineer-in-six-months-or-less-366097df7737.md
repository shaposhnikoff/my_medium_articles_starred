
# How To Become a DevOps Engineer In Six Months or Less

“Empty highway road in through a colorful desert” by Johannes Plenio on Unsplash

*NOTE: This is Part 1 of a multi-part series.*

*Part 2 is [here](https://medium.com/@devfire/how-to-become-a-devops-engineer-in-six-months-or-less-part-2-configure-a2dfc11f6f7d).*

## Target Audience

Are you a developer looking to shift your career towards a more DevOps model?

Are you are a classically trained Ops person and you would like to get a feeling of what this whole DevOps thing is all about?

Or are you neither, and having spent some time working with technology you are now simply looking for a career change and have no idea where to start?

If so, read on, for we are going to see how to become a mid-level DevOps engineer in six months!

Finally, if you have been doing this DevOps thing for years now, you might still find this useful as a validation of where we are and where this is going.

## What’s This, Now?

First, what is DevOps?

You can google the definitions and wade through all that buzzword extravaganza but know that most are embarrassingly long word salads stuffed into giant run-on sentences. (See what I did here?)

So, I’ll save you the clicks and distill it down:
> DevOps is a way to deliver software with shared pain and responsibility.

That’s it.

OK, but what does **that** mean?

It means that traditionally, the developers (people who create software) had incentives that were vastly different from operations (people who run software.)

For example, as a developer, I want to create as **many **new features as fast as possible. After all, this is my job and that’s what customers demand!

However, if I’m an ops person, then I want as **few **new features as possible because every new feature is a change and change is risky.

As a result of this misalignment of incentives, DevOps was born.

DevOps attempts to fuse development and operations (DevOps, get it?) into one group. The idea is that one group will now share both the pain and the responsibility (and presumably, the rewards) of creating, deploying, and generating revenue from customer-facing software.

Now, purists will tell you know that there is no such thing as a “DevOps Engineer”. “DevOps is a culture, not a role,” they will tell you.

Yeah, yeah. They are technically correct (the worst kind of correct!) but as it so often happens, the term has morphed beyond its original meaning.

Now, being a DevOps Engineer is something like “Systems Engineer 2.0.”

In other words, somebody who understands the Software Development Lifecycle and brings software engineering tools and processes to solve classic operations challenges.
> DevOps ultimately means building digital pipelines that take code from a developer’s laptop all the way to revenue generating prod awesomeness!

That’s what it’s all about!

Also note that as a career choice, the whole DevOps space is highly compensated, with almost every company either “doing DevOps” or claiming to do so.

Regardless of where the companies are, the overall DevOps job opportunities are plentiful, offering fun, meaningful employment for years to come.

NOTE: Be wary of companies hiring for a “DevOps team” or a “DevOps department.” Strictly speaking, such things should not exist because ultimately, DevOps is all about the culture and a way of delivering sofware, **not **a new team or department to be staffed up.

## Disclaimer

Now, let’s put the glass of Kool-Aid aside for a moment and consider the following.

Have you heard the old adage, “there are no junior DevOps engineers?”

If not, please know it is a popular trope on Reddit and StackOverflow. But what does that mean?

Simply put, it means that it takes many years of experience, combined with a solid understanding of tools, to eventually become a truly effective Senior DevOps practitioner. And sadly, there is no shortcut for experience.

So, this is not an attempt to cheat the system — I don’t think that’s actually possible to pretend to be a Senior DevOps Engineer with a few months of experience. Solid understanding of the rapidly changing tools and methodologies takes years to master and there is no getting around that.

However! There is a roughly agreed upon (trendy, if you will) menu of tools and concepts that most companies use and that is what the article is all about!

Again, tools are different from skills, so while you are learning the tools, make sure you don’t neglect your skills (interviewing, networking, written communication, troubleshooting, etc.)

Most importantly, don’t lose track of what we are after — **building a fully automated digital pipeline that takes ideas and turns them into revenue generating pieces of code.**

That is the single, most important take-away from this entire article!

## Enough Talk, Where Do I Start?

Below is your roadmap.

Master the following and you can safely and honestly call yourself a DevOps Engineer! Or a Cloud Engineer if you detest the “DevOps” title.

The map below represents mine (and probably the majority of folks working in this space) idea of what a competent DevOps Engineer should know. That said, it is only an opinion and there will certainly be dissenting voices. That is OK! We are not after perfection here, we are after a solid foundation upon which to build.

NOTE: You are meant to traverse this breadth-first, layer by layer. Start (and continue!) with the foundation first. Learn the technologies in blue first (Linux|Python|AWS), then if time permits or job market demands, go after the purple stuff (Golang|Google Cloud).

![DevOps Foundational Knowledge](https://cdn-images-1.medium.com/max/2000/1*GNxucS4v93-XdnD5-vWB_w.png)*DevOps Foundational Knowledge*

Again, go after the first layer in every pillar. Then, time permitting, go after the second layer to add depth to your expertise.

Honestly, the foundational layer above is something you can never really stop learning. Linux is complex and takes years to master. Python requires on-going practice to stay current. AWS evolves so rapidly that things you know today are but a fraction of the overall portfolio a year from now.

But once you have the Foundation layer reasonably figured out, move onto the real-world set of skills. Notice there are 6 blue columns total, one per month.

![Six parts, one part per month of learning real-world skills](https://cdn-images-1.medium.com/max/2922/1*yjU_IVVZRQ1oXtnxAuGUhQ.png)*Six parts, one part per month of learning real-world skills*

NOTE: What’s notably missing from the pipeline above is Test. That’s intentional — writing unit, integration, and acceptance tests is not easy and traditionally falls on the shoulders of developers. The “Test” phase omission is intentional, since the goal of this roadmap is rapid intake of new skills and tools. Lack of testing expertise is judged by the author to be an insignificant barrier to a proper DevOps employment.

Also, please remember, we are not after learning a whole bunch of unrelated techno-babble here. We are after a solid understanding of tools that taken together, tell a single, coherent story.

That story is **end-to-end process automation — a digital pipeline that moves bits around in an assembly line-like fashion.**

Moreover, you don’t want to learn a bunch of tools and stop. Tools change rapidly, concepts much less so. Therefore, what you want to do is use the tools as learning proxies for the higher level concepts.

OK, let’s dig in a little deeper!

## Foundational Knowledge

Under the top line labeled “Foundation” you will see the skills that every DevOps Engineer must master.

Here, you will see three industry-dominant pillars: operating system, programming language, public cloud. These things are not going to be something that you can learn really quickly, check them off the list and move on. These are going to be skills that you must acquire and keep sharp on a continuous basis, to stay relevant and up to date with what is going on.

Let’s go through them one by one.

Linux: where everything runs. Now, can you be an awesome DevOps practitioner and stay entirely within the Microsoft ecosystem? Of course, you can! There’s no law that mandates Linux for everything.

However! Please know that while all the *DevOps-y* things can certainly be done with Windows, it is far more painful and the job opportunities are far fewer. For now, you can safely assume that one cannot become a true DevOps professional without knowing Linux. Therefore, Linux is what you must learn and keep learning.

Honestly, the best way to do it is to just install Linux (Fedora or Ubuntu) at home and use that as much as you can. You will break things, you will get stuck and then you will have to fix it all and in the process, you will learn Linux!

For reference, in North America, the Red Hat variants are more prevalent. Therefore, it makes sense to start with [Fedora ](https://spins.fedoraproject.org/kde/download/index.html)or [CentOS](https://www.centos.org/download/). If you are wondering whether you should get the KDE or Gnome edition, get KDE. That’s what Linus Torvalds uses. :)

Python: the dominant back-end language these days. Easy to get started with, widely used. Bonus: Python is very prevalent in the AI/Machine Learning space, so if you ever want to transition to yet another hot field, you’ll be all set!

Amazon Web Services: Once again, it is impossible to become a seasoned DevOps professional without a solid understanding of how a public cloud works. And if knowledge of a cloud is what you are after, Amazon Web Services is the dominant player in this space, offering the richest set of tools to work with.

Is it possible to start with Google Cloud or Azure instead? Absolutely! But we are after the biggest bang for the buck here, so AWS is the safest play to make, at least in 2018.

You get a free tier to play with when you sign up for an account at AWS, so that is a good place to start.

Now, when you log into the [AWS console](https://aws.amazon.com/), you are greeted with a simple, easy to understand menu of choices.

![“Discovering yet another AWS feature I never knew about” by [Tom Pumford](https://unsplash.com/@tompumford?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/8134/0*mbMm7PT2bLCcr_Fd.)*“Discovering yet another AWS feature I never knew about” by [Tom Pumford](https://unsplash.com/@tompumford?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

That was sarcasm. Good news is, you don’t need to know every single Amazon tech.

Start with the following: VPC, EC2, IAM, S3, CloudWatch, ELB (under the EC2 umbrella), and Security Groups. These things are plenty to get you started and every modern, cloud-enabled enterprise will be using these tools heavily.

AWS’ own training [website](https://www.aws.training/?src=training) is a good place to start.

I recommend you set aside 20–30 minutes daily to practice Python, Linux, and AWS.

NOTE: this will be in **addition** to the other stuff you will have to learn. Altogether, I estimate that spending an hour daily, five times a week is enough to give you a solid understanding of what is going on in the DevOps space within 6 months or less.

Likewise, there are 6 main pillars in total, each corresponding to a month of learning.

That’s it for the Foundational Layer!

In the subsequent articles, we will explore the next level of complexity: how to [Configure](https://medium.com/@devfire/how-to-become-a-devops-engineer-in-six-months-or-less-366097df7737), [Version](https://medium.com/@devfire/how-to-become-a-devops-engineer-in-six-months-or-less-part-3-version-76034885a7ab), [Package](https://medium.com/@devfire/how-to-become-a-devops-engineer-in-six-months-or-less-part-4-package-47677ca2f058), Deploy, Run, and Monitor software in a fully automated way!
