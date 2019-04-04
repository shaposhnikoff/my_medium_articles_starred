
# How To Become a DevOps Engineer In Six Months or Less, Part 3: Version

“Close-up of a backlit laptop keyboard” by Markus Petritz on Unsplash

## Quick Recap

Let’s quickly recap where we are.

In short, this series of posts tells a story.

And that story is learning how to take an idea and turn it into money, as quickly as possible — the essence of the modern DevOps movement.

Specifically, in [Part 1](https://medium.com/@devfire/how-to-become-a-devops-engineer-in-six-months-or-less-366097df7737) we talked about the DevOps culture and goals.
[**How To Become a DevOps Engineer In Six Months or Less**
*NOTE: This is Part 1 of a multi-part series.*medium.com](https://medium.com/@devfire/how-to-become-a-devops-engineer-in-six-months-or-less-366097df7737)

In [Part 2](https://medium.com/@devfire/how-to-become-a-devops-engineer-in-six-months-or-less-part-2-configure-a2dfc11f6f7d), we talked about how to lay the foundation for future code deployments with Terraform. Of course, Terraform is code also!
[**How To Become a DevOps Engineer In Six Months or Less, Part 2: Configure**
*NOTE: this is Part 2 of a multi-part series. For Part 1, please click here.*medium.com](https://medium.com/@devfire/how-to-become-a-devops-engineer-in-six-months-or-less-part-2-configure-a2dfc11f6f7d)

Consequently, in this post, we will discuss how to keep all these pieces of code from completely going haywire all over the place. Spoiler, it’s all about [*git](https://git-scm.com/)*!

Bonus: we will also talk about how to use this git business to build and promote your own personal brand.

For reference, we are here in our journey:

![DevOps Journey](https://cdn-images-1.medium.com/max/2922/1*N-4zkp9GM6apxn3GMWcT0A.png)*DevOps Journey*

## Why Bother?

When we talk of “Versioning” what do we mean?

Imagine you are working on some piece of software. You are making changes to it constantly, adding or removing features as needed. Often, the latest change will be a “breaking” change. In other words, whatever it was you did last, broke whatever you had working prior.

Now what?

Well. If you are really Old School, you probably had a tendency to name your first file like this:

    awesome_code.pl

Then you start making changes and you need to preserve what works, in case you have to go back to it.

So, you rename your file to this:

    awesome_code.12.25.2018.pl

And that works fine. Until one day you make more than one change per day, so you end up with this:

    awesome_code.GOOD.12.25.2018.pl

And so on.

Of course, in a professional environment, you have multiple teams collaborating on the same codebase, which breaks this model even further.

Needless to say, this crazy train derails fast.

## Source Code Control

Enter Source Code Control: a way to keep your files in a **centralized** location, where multiple teams can work together on a common codebase.

Now, this idea is not new. The earliest [mention ](https://en.wikipedia.org/wiki/Source_Code_Control_System)of such a thing that I’ve been able to find dates back to 1972! So, the idea that we should centralize our code in one place is definitely old.

What is relatively new, however, is the idea that **all production artifacts must be versioned**.

What does that mean?

It means that everything that touches your production environment must be stored in version control, subject to tracking, review, and history of changes.

Moreover, enforcing the “all prod artifacts must be versioned” law really forces you to approach problems with the “automation first” mindset.

For example, when you decide to just click your way through a complex problem in your Dev AWS environment, you can pause and think, “Is all this clicking a **versioned artifact**?”

Of course, the answer is, “no”. So, while it is OK to do rapid prototypes via UI to see if something works or not, these efforts must really be short-lived. Longer term, please make sure you do everything in Terraform or another infrastructure-as-code tool.

OK, so if everything is a versioned artifact, how do we store and manage these things, exactly?

The answer is git.

## Git

Until [git](https://git-scm.com/doc) came along, using source code control systems like SVN or others was clunky, not user friendly and was in general a pretty painful experience.

What git does differently is that embraces the notion of a **distributed** source code control.

In other words, you are not locking other people out of a centralized source code repository, while you are working on your changes. Instead, you are working on a complete **copy **of the codebase. And that copy then gets *merged *into the *master* repository.

Keep in mind, the above is a gross oversimplification of how git works. But it is enough for the purposes of this article, even though knowing the inner workings of git is both valuable and takes a while to master.

![[https://xkcd.com/1597/](https://xkcd.com/1597/)](https://cdn-images-1.medium.com/max/2000/0*hoGY4_63YI8B7Pbc.png)*[https://xkcd.com/1597/](https://xkcd.com/1597/)*

For now, just remember that git is **not** like SVN of old. It is a distributed source code control system, where multiple teams can work on a shared codebase safely and securely.

## Why Bother?

Specifically, I would strongly argue that you **cannot** become a professional DevOps (Cloud) Engineer without knowing how git works. It’s as simple as that.

OK, so how does one learn git, exactly?

I must say, Googling for a “git tutorial” has the dubious distinction of coming up with extremely comprehensive and extremely confusing tutorials.

However, there are a few that are really, really good.

One such series of tutorials I urge everyone to read, learn and practice with is [Atlassian’s Git Tutorials](https://www.atlassian.com/git/tutorials).

In fact, they are all pretty good but one section in particular is what is used by professional software engineers the world over: [Git Workflows](https://www.atlassian.com/git/tutorials/comparing-workflows).

Another really good tutorial is [Learn Git Branching](https://learngitbranching.js.org/).

Atlassian tutorials are just reading and learning (if that’s your thing) while Learn Git Branching is an interactive tutorial (if that’s your thing).

Regardless, you will not get far in this business if you don’t understand how git works!

I cannot stress this enough. Time and again, lack of understanding how git feature branching works or failure to explain Gitflow is what sinks 99% of aspiring DevOps engineers candidacies.

This is a key point. You can come to an interview and not know Terraform or whatever the latest trendy infrastructure-as-code tool is and that’s ok — one can learn it on the job.

But not knowing git and how it works is a signal that you lack the fundamentals of modern software engineering best practices, DevOps or not. That signals to hiring managers that your learning curve is going to be very steep. You do not want to signal that!

Conversely, your ability to confidently speak of git best practices tells the hiring managers that you come with a software engineering mindset first — exactly the kind of image you want to project.

To summarize: you don’t need to become world’s foremost git expert to land that awesome DevOps role but you do need to live and breathe git for a while to be able to confidently speak about what’s going on.

At a minimum, you should be well versed in how to

1. Fork a repo

1. Create branches

1. Merge changes from upstream and back

1. Create Pull Requests

## Now What?

Now, once you get through the introductory git tutorials, get yourself a [GitHub ](https://help.github.com/)account.

NOTE: GitLab is OK also but at the time of this writing, GitHub is the most prevalent open source git repository, so you want to be where everyone else is.

Once you have your GitHub account, start contributing your code to it! Whatever you learn that requires you to write code, make sure you commit it to GitHub regularly.

This not only instills good source code control discipline but helps you build your own personal brand.

NOTE: when you are learning how to use git+GitHub, pay special attention to [Pull Requests](https://help.github.com/articles/about-pull-requests/) (or PRs, if you want to be cool).

![Pull Request, by [Vidar Nordli-Mathisen](https://unsplash.com/@vidarnm?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/5760/0*E1Y3iKOJjkKiwcoa)*Pull Request, by [Vidar Nordli-Mathisen](https://unsplash.com/@vidarnm?utm_source=medium&utm_medium=referral)*

## Brand

Speaking of cool — a brand is a way to showcase to the wider world what you are capable of.

One way (currently, one of the better ways!) is to establish a solid GitHub presence as a proxy for your brand. Almost all employers these days will ask for one anyway.

Therefore, you should strive to have a neat, carefully curated GitHub account — something you can put on your resume and be proud of.

In later sections, we’ll talk about how to build a simple but cool-looking website on GitHub using the [Hugo ](https://gohugo.io/)framework. For now, just putting your code into GitHub is enough.

Later on, as you get more experienced, you might consider having two GitHub accounts. One for your personal stuff you use to store practice code you write and another account to store code you want to show others.

To summarize:

* Learn git

* Contribute everything you’ve learned to GitHub

* Leverage #1 and #2 as a showcase of all the things you have learned thus far

* Profit!

## Final Thoughts

Finally, please keep in mind the latest developments in this space, such as [GitOps](https://queue.acm.org/detail.cfm?ref=rss&id=3237207).

GitOps takes all the ideas we have been discussing thus far to new levels — where everything is done via git, pull requests, and deployment pipelines.

Note that GitOps and approaches like it speak to the **business** side of things. Specifically, that we are not after using complex things like git because they are cool.

Instead, we are using git to enable business agility, speed up innovation and deliver features faster — these are things that allow our business to make more money in the end!

That’s all for now!

And if you are ready to move on, [Part 4](https://medium.com/@devfire/how-to-become-a-devops-engineer-in-six-months-or-less-part-4-package-47677ca2f058) is here.
