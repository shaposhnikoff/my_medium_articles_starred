Unknown markup type 10 { type: [33m10[39m, start: [33m46[39m, end: [33m78[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m28[39m, end: [33m39[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m295[39m, end: [33m327[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m215[39m, end: [33m231[39m }

# PUSH TRIGGERED LOCAL BUILD AUTOMATION — JENKINS CI, GITHUB WEBHOOKS AND PROXY TUNNELING, … OH MY!

Posted on January 15, 2019 by Ben Nowak in Code, Featured

![](https://cdn-images-1.medium.com/max/2000/1*GT1OCMy0z_Dqj1frGt_01Q.jpeg)

In a recent lab session, I was challenged to get an automated build process set up using Jenkins on my local machine and a remote Git repo hosted on GitHub. While polling is a quick solution, it doesn’t scale well, and isn’t really the “pro” way to do it. It took a bit of research, a little trial and error, and one cool proxy service, but eventually my local build server was responding to pushes like a good CI soldier. Here’s how I did it.

## TLDR;

*Using the ngrok.io proxy service, you can create an internet facing tunnel between your local machine and a ngrok.io subdomain that points to a specific port on your local development machine. Using the created “subdomain” as the GitHub WebHook Payload URL, you can configure your local Jenkins instance to respond to GitHub repo changes and trigger builds or other automatic processes.*

## PRELIMINARIES

You must have a running instance of Jenkins on your local machine. Preferably you have it running as a service so that you don’t have to keep up with turning it on and off. You can find plenty of tutorials for running Jenkins locally. My preferred method on the mac is to go the homebrew route and have it start at boot time as a service. Be sure to give Jenkins a designated port number that is not in use. It will default to 8080, but I find my nginx server likes that port, and so I’ve used an alternate port. You do what’s best for your setup.

Obviously you need a GitHub account. If you’re reading this and don’t have one … what are you waiting for. Go get one. SERIOUSLY!!! I’ll wait.

Finally you’ll need a proxy tunneling service to handle “exposing” your machine’s CI Server (Jenkins) to the web so that GitHub and talk to your local Jenkins, and vice-versa. There are a couple of providers out there. The one I will be using today is called ngrok.io. For small testing setups and single repo projects a free account should do the trick for proving to yourself that this will work. However, should you prefer to not have to reset your WebHook Payload URL (more on that in a moment) every time you get ready to be productive, you may opt for a paid membership. The lowest, at $5 per month (less if you pay for a full year) is pretty reasonable for what you get.

## THE SETUP

Ok. Jenkins running locally. Check! GitHub account. Check! Proxy tunneling service (presumably ngrok.io … if not, you’re on your own on that part.) Check! So let’s get started.

## OVERVIEW

The basic steps are as follows:

1. Install Jenkins GitHub plugin

1. Get ngrok up and running

1. Configure Jenkins project

1. Setup repo Webhook on GitHub

1. Try it out

## STEP 1 — INSTALL JENKINS GITHUB PLUGIN

The first thing you will need to do is to install the Jenkins [GitHub plugin](https://plugins.jenkins.io/github). The Jenkins [official wiki](https://wiki.jenkins.io/display/JENKINS/Github+Plugin) has additional info as well. The easiest method to install the plugin is to simply navigate from your Jenkins Dashboard to Manage Jenkins -> Manage Plugins, then click on the Available tab and search for GitHub in the filter text box.

![](https://cdn-images-1.medium.com/max/2000/0*zfHjp6-vdA12kD24)

Simply check the box and click install.

## STEP 2 — GET NGROK UP AND RUNNING

![](https://cdn-images-1.medium.com/max/2000/0*7nug-WB0i1QqfVxQ)

Again. I must reiterate. I’m sure there are other options out there, but this is the one I settled on. If you’re the paranoid type (which I usually am) you can opt to do a hundred things to obfuscate your network exposure and do things like reverse proxies to servers, and all kinds of digital gymnastics ad nauseam. If you want simple, and you don’t have the time or infrastructure to spin up your own internal Git server (like a roll-your-own GitHub clone), then just use a proxy tunnel service like [ngrok.io](https://ngrok.io/)

There’s a few tiers of subscription, in addition to free, accounts. While you technically can do it using the free account, you would then be limited to a single tunnel, with changing sub-domain names. If you wan’t a static subdomain name (like my-nifty-tunneled-dev-laptop.ngrok.io ) then you’re gonna have to hand over some cheddar. For the price of a fancy espresso drink every month, you can have a subdomain all your own.

Once you’ve signed up and verified your account, you must install the ngrok client. There’s a host of binary files for your OS of choice, but again my preferred method is to use homebrew.

    brew cask install ngrok

The next step is to set your auth token which is displayed on your account page under the “Auth” sidebar.

    ngrok authtoken 4lksd09aeg_f9jFIEs9el752l1LErjIrao9s3ljgs09578

Once you’ve done all that it’s as simple as running a single command in your terminal of choice.

    ngrok http 8080 -subdomain=your-custom-name

This will cause any HTTP requests made to the [https://your-custom-name.grok.io](https://your-custom-name.grok.io/) URL to be tunneled securely to your local machine at the port you specify. There’s some other options and configurations to explore with ngrok, but I was particularly impressed at how easy configuration was.

## THE GRITTY DETAILS

Ok. So far so good. But now come the gritty details required to get everyone talking and working smoothly. So pay attention … to the details!

## STEP 3 — CONFIGURE JENKINS PROJECT

You will need to configure your Jenkins project to receive the Webhook HTTP Post messages from GitHub. This requires a few particulars in the project configuration as well as Jenkins itself (via Manage Jenkins).

You can do all of this with Jenkinsfile pipelines (if you’re so inclined), however for the sake of brevity and illustration we will use a Jenkins Freestyle Project. From your Jenkins dashboard, select New Item. Then choose Freestyle project and press OK.

![](https://cdn-images-1.medium.com/max/2000/0*rAVZ-8TSHiJN2pOo)

You will then be presented with the Jenkins project configuration screen.

![](https://cdn-images-1.medium.com/max/2000/0*0AGHrIjXCfMKAn6h)

Scroll down to Source Code Management (SCM), and select the Git radio button.

![](https://cdn-images-1.medium.com/max/2000/0*MpzjlKgOiggLqaYW)

Here you can choose to specify a particular branch, multiple branches, credentials (for private repos) and other fancy options.

Scroll down to build triggers, and select the GitHub hook trigger for GITScm polling checkbox. This will activate Jenkins to “listen” for incomming GitHub Webhooks.

![](https://cdn-images-1.medium.com/max/2000/0*mQUfwtSTp8ReBFk5)

There are plenty of other actions and configuration setting you can tweak, but these are the main Jenkins project configurations that need to be in place.

Next we will make a change to the Jenkins Global Security configuration. Since this isn’t (generally speaking) going to be a high volume, high exposure production environment, we can get away with making this one change for now. If someone thinks of a way to get around this, please post a comment and let me know.

![](https://cdn-images-1.medium.com/max/2000/0*h5sDqo14Hbczb2dZ)

Check the box that says “Enable proxy compatibility”. There appears to be a stripping away of header tokens through the tunneling that causes the proxy to trigger a 403 error. This was the only way I found to get the POST to successfully hit 200.

Finally, as far as Jenkins is concerned, you only need to get one more piece of info. A Crumb. It’s kind of like a reverse cookie, at least that’s what I’m going with. It’s a token that authorizes the Webhook to make the POST request. Navigate to your Jenkins local URL and append the path with /crumbIssuer/api/json?tree=crumb

![](https://cdn-images-1.medium.com/max/2000/0*7hAhj_9qpHoAT2Je)

Make sure you copy down the text returned and save it for later. You will need it later to input into the GitHub Webhook form.

![](https://cdn-images-1.medium.com/max/2000/0*XD5Rh5SmJyUBa8Aa)

## STEP 4 — SETUP REPO WEBHOOK ON GITHUB

Now for the final step. Navigate to the GitHub repo. Select settings (for the repo, not your account settings!). Select Webhooks on the sidebar. Click on the “Add webhook” button on the right side.

![](https://cdn-images-1.medium.com/max/2000/0*Ii8PWgjgJmB3EPhW)

You will be presented with the following page. Be sure that you enter the Payload URL correctly, and paste the “crumb” (which hopefully you saved somewhere) from before, into the “secret” field. Set content type to application/json

![](https://cdn-images-1.medium.com/max/2000/0*2zlMDKzkYYaGk7wM)

Before we continue, make sure you have a ngrok session up and running on your local machine. Head to your terminal and type :

    ngrok http 8080 -subdomain=whateveryoucalledyours

You should be presented with a terminal window that looks something like this.

![](https://cdn-images-1.medium.com/max/2000/0*JaMg4RiDEL7NImle)

Go back to the GitHub Webhooks page and click the green “Add webhook” button at the bottom of the page and cross your fingers … and toes. Just kidding. GitHub will push out a “test” POST and you should end up with a page that looks happy with a green check mark. This means that Jenkins responded with a 200 code and everything should be peachy.

![](https://cdn-images-1.medium.com/max/2000/0*D83YOnqLHOTHosGu)

## STEP 5 — TRY IT OUT!

At this point all the pieces are in place. Barring any errors you should be ready to test this whole thing out. Make some change to your local repo, do a commit, then a push, and watch the magic happen. Jenkins should give you some warm and fuzzies in the form of the similar outputs on the Console Output and GitHub Hook Log pages of the project.

![](https://cdn-images-1.medium.com/max/2000/0*R2C-9IuLB_MUqYg0)

![](https://cdn-images-1.medium.com/max/2000/0*i9aeA0Wq5Iby5E0b)

## THE WRAP UP

I hope that this walk through tutorial has inspired you to tinker and toy around with this set of cool tools and find great ways to enhance the functionality of your local development environment. There’s plenty of pitfalls to trip you up along the way, and surely some other considerations that have escaped my observations, but overall this should get you on the path, even if it doesn’t get you all the way there. Leave me a message or two about anything you come across, up against, or wade through.

And next time someone tells you it can’t be done, … figure it out!
[**Push Triggered Local Build Automation - Jenkins CI, GitHub WebHooks and Proxy Tunneling, ... oh My!**
*Jenktocat at Your Service! In a recent lab session, I was challenged to get an automated build process set up using…*Originally published at bencodes.blog](https://bencodes.blog/2019/01/15/jenkins-ci-github-local-build-automation/)
