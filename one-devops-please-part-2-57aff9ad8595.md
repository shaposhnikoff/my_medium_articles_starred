
# One DevOps Please — Part 2

“One DevOps Please” — Part 1 recap:

* DevOps is part of a learning journey for people

* We can categorize how people learn in 4 Stages

* The hardest part of the learning process can be a willingness to learn

* Companies don’t transform just because we teach a handful of people in isolation (BottomUp)

## Organizational Transformation — The path to “DevOps”

DevOps is an overloaded term these days. A DevOps journey might touch on agile, lean, learning, metrics, business value, pipelines, CI/CD, shifting left, 12 factor apps, Cloud, serverless and kubernetes. It can be exhausting! Why is DevOps so vague and buzzwordy? Because DevOps covers the **people**, process and technology aspects of modern software development. Let’s expand on our original definition from Part 1:
> “DevOps is a culture shift or a movement that encourages great communication and collaboration to foster **building better-quality software** more **quickly** with more **reliability**.” — Mike Kavis

But what does transformation look like for an entire company? While teaching somebody how to master automated pipelines could take a few weeks or months, it could take years to see changes that span the entire company. This is because companies are made up of people — each with their own contribution to it’s culture.

Let’s go on the same learning journey we did for people but through the lens of the entire company this time around. We’re going to leverage tried and trusted models from[ https://web.devopstopologies.com/](https://web.devopstopologies.com/) to take you on this journey because they’re such a good representation of what we see in the real world.

### Stage One — Everything is okay

We start off much the same way we did with the people side of things. The company is operating in Silos like “Dev” and “Ops” and thinking:
> “Everything is fine the way it is”

![Anti-pattern — Dev and Ops Silos](https://cdn-images-1.medium.com/max/2000/0*0vBMPZZa2Su93GSr)*Anti-pattern — Dev and Ops Silos*

### Stage Two — Automate all the things
> “Let’s do something and call it DevOps.”

In the next stage the organization recognizes they need to change so they “try to do DevOps”. This is where we typically see a few anti-patterns form as people try to find the sweet spot for their new team structures and ways of working — this is also more common than you might think. We often spend a lot of time with clients at this stage and it’s by far where the hardest work gets done.

So let’s explore the swinging pendulum of DevOps Anti-Patterns and appreciate how easy it is to get things wrong — even with the best of intentions.

***Anti-pattern 1 — The DevOps Silo: ***This happens when you create a “DevOps” or “Platform” team that’s separate from your development teams or cross-functional teams. You’ll write some great automation scripts but transformation doesn’t happen here.

![Anti-pattern 1 — DevOps Silo](https://cdn-images-1.medium.com/max/2000/0*CQ4GgVgmRX53CEle)*Anti-pattern 1 — DevOps Silo*

***Anti-pattern 2 — NoOps: ***While this doesn’t seem like a terrible idea at face value, downplaying the importance of operations within software development can lead to poor choices that wake you up at 3am.

![Anti-pattern 2 — Dev/No Ops](https://cdn-images-1.medium.com/max/2000/0*2yTARKX2CDq8iWiK)*Anti-pattern 2 — Dev/No Ops*

***Anti-pattern 3 — AllOps: ***Isolating developers is not going to lead to better software delivery, and most definitely won’t lead to transformation in your organization (unless you really need to spend 80% of your time building that wondering k8s cluster).

![Anti-pattern — DevOps Engineers](https://cdn-images-1.medium.com/max/2000/0*eZvMAhupHNx6hp9Q)*Anti-pattern — DevOps Engineers*

So at this point you’ve tried a few different changes and had varying success. Hopefully you’ve seen some improvements along the way and you’re ready to take your organization to the next stage.

### Stage Three — Transformation

Why don’t we try the “*You build it, you run it*” model. Now we’re talking!

*Operations*: Responsible for providing and operating the platform

*Developers*: Responsible for operating their applications, which are built on top of the enterprise guard rails.

This is where we usually start to see“shifting left” in terms of QA and Security — and that’s a win for everybody.

![Platform as a Service (PaaS) Operating Model](https://cdn-images-1.medium.com/max/2000/0*d6wxLPtF1b5E2bWI)*Platform as a Service (PaaS) Operating Model*

Here are some key indicators that your organization is maturing in it’s DevOps transformation journey:

* Improved collaboration across departments

* [Value stream mapping](https://en.wikipedia.org/wiki/Value-stream_mapping)** **exercises are performed to drive process improvement

* A learning organization takes shape

* [Blameless post mortems](https://landing.google.com/sre/sre-book/chapters/postmortem-culture/)

* [Gamedays](https://wa.aws.amazon.com/wat.concept.gameday.en.html)

* [Lean principles](https://en.wikipedia.org/wiki/Lean_software_development) are embraced — Eliminate waste, Amplify learning, Decide as late as possible, Deliver as fast as possible, Empower the team, Build integrity in, Optimize the whole

### Stage Four — The pursuit of Continuous Improvement

Hopefully by now you’ve realized there’s no perfect Venn diagram to determine the success of your organization. There are however some key indicators you can look at for.

Building on the success from true DevOps transformation in Stage Three you should now be capable of deploying multiple times a day with increased certainty and minimal risk. This means more automated tests and better quality code. Removing bottlenecks should second nature and everything should start shifting left.

What we should see now are these new ways of thinking spreading throughout the entire organization, no longer a “DevOps” thing for the I.T department — continuous improvement is happening in all parts of the business. Instead of teams building up towers of influence to protect their corner of the organization people are working together toward a common goal.

## The recipe for successful transformation

So far we’ve focused on “how we learn” and touched on “DevOps” but we haven’t really gone into practical advice for how to transform a company. Offering up a personalized plan in a blog post would be foolish but I can definitly set you off on the right path. If you’re a leader trying to kick-start your own company transformation there are some lessons others have learned that will save you years of banging your head against the wall.

* **Top-Down is half the recipe** — Your company needs to hear the right things from the leadership group, and people/teams/departments also need to be held accountable if they’re not supporting the target-state laid out in the vision by leadership group (actions always speak louder than words).

* You guessed it **Bottom-Up** is the second half — Teams on the ground need to be bought-in on the vision and willing to execute. We’re talking about completely changing people’s working lives and the way they think. Start with teams most willing to change, prove what works in small but significant teams and products, and use them as a lighthouse for the rest of the teams.

* Find **change-agents/champions** — These are people within your organization that are not only willing to change, they’re ready to shout it from the rooftops and inspire those around them.

* Don’t just hire contractors — **find genuine transformation partners** and learn absolutely everything you can from them.

*Homework for transformation leaders:* Take a deep-dive into how [Microsoft, GE and Amazon went on their own transformation journeys](https://www.forbes.com/sites/stevedenning/2019/03/17/how-mapping-the-agile-transformation-journey-points-the-way-to-continuous-innovation/#7d8b44e44b24) (and explore what went well, and what they could have improved).

**Transformation Partners** needs it’s own section…

I can come in and build you a shiny new k8s cluster, maybe I could take 4 of the brightest developers I know and churn out our own applications faster than most of your existing teams, or embed in your teams and give you a report showing how we increased development by 320%. But if everything goes back to how it was after I leave — you’ve made 0% progress towards transformation (also please stop building k8s clusters because it’s cool — focus on if each of your teams can succinctly explain who their customer is, how they obtain feedback in the shortest loop possible and what business value they’re delivering in each two-week period).

So what does a transformation partner look like (and am I about to sell you my services?). The only two things I need to tell you about the company I work for is “we charge reassuringly high rates” and we “fall in love with our customer”.

* If you can hire the right people to inspire, lead and transform your organization **please do that**. I assure you I’ll still find a way to put food on the table.

* If you want a power-boost, a consultancy that’s focused on transformation filled with people who’ve done it before should definitely be on your list — Pair them up with your change agents and the willing development teams in your organization, and watch your growth sky rocket (but **be prepared to do to work**).
> Hiring a consultancy for transformation and asking them to do all the work is like hiring a personal trainer, making them do all of your situps, then wondering why they look great but you still haven’t ‘transformed” — Andrew Khoury

Not keen on a consultancy and feel like you’re company only needs a small nudge in the right direction? Sounds like you want to head-hunt a few key people (when you do that make sure they’re empowered to make the changes you want them to make) and have some key mentors by your side to bounce ideas off and keep you honest.

## Summary

Many organizations try to replicate the success of others “doing DevOps” by hiring DevOps consultants or building DevOps teams but they often miss the mark in doing so — the journey is more important than the destination. Let’s recap what we learnt on this journey together:

**How people learn: **We explored how people learn and why it’s important to take the time to bring them along the journey. Trust and empathy are key here.

**Learning from your trusted partners: **We touched on how valuable it is to have a trusted partner in your corner, with a focus on drawing from their experience and maximizing how much you learn from them. This allows for new ways of thinking to take hold in your organization which ignites the transformation process.

**Organizational transformation: **We broke down transformation into four key stages:

1. **Everything is fine** the way it is

1. **Automation** — but not much direction (this is the hardest stage and where we learn what doesn’t work)

1. **Actual transformation** (this is where the hard work pays off — We see Improved collaboration, Value stream mapping, a learning organization, Blameless post mortems, Gamedays, Lean principles)

1. The pursuit of **Continuous Improvement** (this is where we refine what’s already working)

**Approach transformation with a plan** — That means Top-Down & Bottom-Up, identifying and empower your change-agents, holding people accountable for transformation and positive behaviors. Don’t forget to find those willing development teams who are ready to transform into cross-functional product teams with with a healthy obsession for their customers & continuous learning and improvement.

![Happy Transforming!](https://cdn-images-1.medium.com/max/2000/1*dklQFriHqa2IfjWlUnFaXg.jpeg)*Happy Transforming!*

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
