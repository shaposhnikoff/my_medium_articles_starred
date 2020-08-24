
# Create a VPS with Google Cloud’s Compute Engine

Compare the advantages of Google’s Compute Engine to other cloud providers.

![](https://cdn-images-1.medium.com/max/3000/1*m7VUiOQgAeb4AUKWkxIWyw.jpeg)

I’ve been a big fan of Google Cloud Platform from the beginning, and they aren’t even paying me to say that. A lot of critics place GCP far enough behind AWS and Azure that most people haven’t bothered to consider the “third option,” but this leaves much of the story untold.

We engineers love to compare hard numbers, and the number we compare the most in the cloud space tends to be the *number of services*. AWS has 10 million services to date, and will always win in this category. AWS *also* considers SNS, SQS, and Kineses to be three separate services, as opposed to, *I don’t know*, designing a single service which can handle those needs? Picking quality over quantity, *in my personal opinion*, is Google Cloud’s differentiator. Google Cloud will never offer hundreds of services because it focuses on doing the things it does well, whether that be in terms of performance, ease of use, or flexibility. It’s clear Google hired quality Product people where other cloud providers did not.

Spinning up a VPS is kind of like a “hello world” for gauging cloud providers. We can learn a lot about the dumb decisions (or lack thereof) embedded in a vendor’s philosophy by how they chose to design the most simple task imaginable. Let’s see what we’ve got here.

## Speed & Price Comparison

In the blue corner, weighing in at 1 CPU and 3.75 RAM, we have our challenger: **Google Compute Engine**. In the opposing corner, weighing in at 4gb RAM and 2 CPUs we have the defending champion:** Digital Ocean**. If we were going by numbers alone, it would seem like Digital Ocean has a clear advantage of winning this matchup.

We’re going to deploy the same site on both machines and compare their performance rankings. How will we do this? With Google’s [Lighthouse](https://developers.google.com/web/tools/lighthouse/), of course! Now, using a Google tool to benchmark a Google product is hardly fair, but let’s be honest: the load time of a site shouldn’t theoretically change to any degree noticeable by humans just by changing clouds, right? Website load times are far more dependent on execution than VPS host…

### Digital Ocean

Let’s start with Digital Ocean, where we’re hosting **hackersandslackers.com**. Not great, but not terrible:

![Lighthouse report for hackersandslackers on Digital Ocean.](https://cdn-images-1.medium.com/max/4394/1*XX1Lxcx-6ZLoEB3zBkLL1A.jpeg)*Lighthouse report for hackersandslackers on Digital Ocean.*

### Compute Engine

Here’s the same site being hosted on Google Cloud, on a machine with theoretically lower specs:

![Lighthouse report for hackersandslackers on Compute Engine.](https://cdn-images-1.medium.com/max/7384/1*VH4gh6KwGWbq6xNngD8mrQ.png)*Lighthouse report for hackersandslackers on Compute Engine.*

Jesus! 10 points is a lot for *no changes whatsoever*. All other metrics stayed the same across the board, which reinforces the fact that no other aspects are at play. Ten points also seems like a suspiciously clean bump. I’m not saying the results are rigged, but it’s worth asking whether or not Google silently favors their own customers. If Google *does* in fact favor sites running on their infrastructure, what are the SEO implications of boosted ratings? Most people would pay a premium to be 10% better in the eyes of Google’s algorithm, so these results are significant regardless.

### Pricing

Let’s do our best to see how Compute Engine pricing stacks up:

![](https://cdn-images-1.medium.com/max/3076/1*TednXRGiyMNaJOP9QPnhKQ.png)

Making these comparisons is difficult to do accurately. We’ve already seen how the quality of RAM or CPU can vary between cloud providers, so these numbers are not 1-to-1. To make matters worse, AWS considers VPS storage to be a completely separate product (wow).

From what we know, this price chart actually looks pretty strong for Compute Engine. GCP machines are substantially cheaper than their equivalent Azure machines. The EC2 instances would actually cost more than what we see listed after adding disk space, which means we’re essentially paying for an extra CPU.

## Does in Fact Compute

Google Cloud gives us a lot of options for customizing our Compute Engine. There are two ways of selecting a machine size: by selecting from the list of preconfigured VMs, or by tweaking the specs of our machine by using the **custom** option.

Here are the preconfigured choices we have:

![Select a preconfigured Compute Engine.](https://cdn-images-1.medium.com/max/3400/1*XahVQdOPbhqtQDcNGGJJfg.png)*Select a preconfigured Compute Engine.*

Picking from a cookie-cutter machine has a notable advantage: when we do so, we’re able to select which CPU we want. This allows us to explicitly specify that we’d like a Skylake processor:

![Select your Compute Engine’s CPU.](https://cdn-images-1.medium.com/max/3432/1*QY667_qBfNrEIcxg0PyuJw.png)*Select your Compute Engine’s CPU.*

When customizing a machine we have the very cool ability to select precisely the number of cores and RAM we want via some nifty sliders. The disadvantage to doing this is we lose the ability to reserve a CPU type. Thus, we almost certainly get a lower quality CPU than we would otherwise:

![](https://cdn-images-1.medium.com/max/3440/1*mN7Wx9ZGEfNT2jxwY1iaGA.png)

### OS and Boot Disk

There are no shortage of Linux distros to pick from on GCP (and Windows, if you’re into that):

![OS Selection for Compute Engine.](https://cdn-images-1.medium.com/max/2724/1*5JMNBWZ4oxIFtLOefs8J2w.png)*OS Selection for Compute Engine.*

The size and type of our instance’s boot disk is always up to us, which is a great feature. I highly suggest taking this chance to create an instance with more than 10GB… I learned the hard way that maxing out a server’s boot disk bricks your machine.

### Firewall Stuff

In case you’re following along, make sure you check **Allow HTTP** **traffic** and **Allow HTTPS traffic** before continuing (assuming you want public traffic). You could change this later, but save yourself the headache of troubleshooting this later.

![Simple preliminary firewall configuration.](https://cdn-images-1.medium.com/max/3148/1*5sZ4m3xl_A9UfzglhELH6A.png)*Simple preliminary firewall configuration.*

Once your instance is created, you should immediately able to SSH into your server via GCP’s browser client.

## Point DNS to Your Instance

Let’s see what’s involved in pointing a domain to Compute Engine. Edit the server you just created and go to the network interface section to add a static IP:

![Adding an external IP.](https://cdn-images-1.medium.com/max/3460/1*ebOoCK3vly5mHk3dPHc9Iw.png)*Adding an external IP.*

A massive advantage that GCP has over AWS is the ease with which we can configure important network stuff. We’re able to do things like assign IP addresses without jumping between multiple screens, configuring security groups, navigating subnets, creating CIDR blocks, or whatever. Even though all of that is possible, Google surfaces the things we want in the places we might expect them to be, like directly on our VPS edit screen.

### Cloud DNS

Google handles DNS via it’s “product” **Cloud DNS**, which handles all the things you’d expect to do with domain name records. To set up a domain we need to create a DNS zone for the domain we’ll be pointing to Google Cloud’s nameservers:

![Creating a Cloud DNS zone.](https://cdn-images-1.medium.com/max/3112/1*-UYAnMZDCTN_45jhq1bdhw.png)*Creating a Cloud DNS zone.*

## VPC & Firewall Settings

The last thing worth highlighting is the way Google handles VPC settings. Find the **Firewall Rules** page to get a taste of how easy it is to manage incoming & outgoing traffic rules:

![](https://cdn-images-1.medium.com/max/3720/1*74ZjGZE-lBmrG4nd6k8_Hw.png)

Selected *Create a Firewall Rule* and check out the process for setting this up:

* **Targets** — This will be where our traffic routes. We want to route to our instance, which is a *specified service account.*

* **Target service account** — Referring to the above, this is where we select the computer instance we want to hit.

* **Target service account** **scope** — Select “in this project”.

* **Source Filter **— Once again, select the *specified service account.*

* **Source service account** **scope **— Select “in this project”

* **Source service account** — This is where we say where the traffic is coming from. It’s coming from the *App engine*, as this is where we specified our DNS.

* For **IPs** and **ports, **well, do what you want. It’s your server.

The process of setting up a firewall rule is so straightforward that you probably missed an important fact: we did all of this painlessly on a single screen. Streamlining this process can be considered an achievement when we look at art the horrendously low bar set by other cloud providers when it comes to networking configuration. I want to rip into AWS again here, but insulting Amazon’s horrendous UX almost seems distasteful at this point.

## Bigger Picture

Having laid out these comparisons, there are actually more advantages of hosting with GCP than I expected. Compute Engine offers arguably the most competitive and transparent pricing of the three major cloud providers. We also have way more power to customize our machine as we see fit. Lastly (and perhaps most important), the GCP ecosystem offers powerful simplicity for users. We shouldn’t underestimate the impact that comes with user experiences — we’d likely have fewer [devastating hacks on enterprises](https://start.jcolemorrison.com/the-technical-side-of-the-capital-one-aws-security-breach/) if cloud providers were less convoluted.

GCP isn’t completely free of its own redundancies. As much as I love to hate on AWS, it seems almost inevitable at this point that any enterprise cloud service will maintain a certain level of complex processes. This flexibility is great for enterprises scaling quickly, but let’s be honest: if these platforms were easy to use, who would pay for the certifications?

Cheekiness aside, I’ve become a strong fan of GCP. Google seems to have hit a middle ground between being user-friendly and powerful, which fills a niché we’ll realize was desperately needed. For a fair review of the platform itself, I find myself agreeing mostly with this stranger from the internet: [https://www.deps.co/blog/google-cloud-platform-good-bad-ugly/](https://www.deps.co/blog/google-cloud-platform-good-bad-ugly/)

*Originally published at [hackersandslackers.com](https://hackersandslackers.com/setting-up-dns-with-google-cloud-platform/) on July 14, 2018.*
