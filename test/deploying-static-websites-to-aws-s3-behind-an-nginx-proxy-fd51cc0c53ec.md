
# Deploying Static Websites to AWS S3 Behind an Nginx Proxy

Animation by Dana Pavlichko

By** [Dan DeLauro](https://medium.com/@dandelauro)**

We are constantly improving our approach to code. We build it. We break it. We love it. We hate it.

And sometimes we blow it all up and start from scratch. If you caught Allison Wagner’s swansong [article about our starter files](http://cognition.happycog.com/article/happy-cog-starter-files-2016-edition), you can recognize the value in years of iteration. But that doesn’t stop with just code. We’re constantly iterating on process, workflow, content strategy, etc. You name it, we’re always looking for ways to improve it. Nothing is ever set in stone. And the same goes for some of the less glamorous (depending on who you ask) tasks like… how do we put these things on the web for people to see?

We’ve been [automating deployments](http://cognition.happycog.com/article/automating-your-deployments) for a while now. And we’re pretty dedicated to continuous branch deployments, which exposes the latest and greatest from everyone to the entire project team. Full disclosure FTW. The only drawback is, the more code you write, the more you deploy. Over time things add up. Servers run out of space, concurrent build tasks compete for resources, and all sorts of odd things begin to happen.

During some of my recent downtime, I started to tinker with our static deployments. I wanted to see if we could get away from building and serving vanilla HTML in a live environment, allowing servers to only worry about “real” apps that require backend resources. We’re still continuous, and we’re still committed to feature/fix branch deployments. We’re just changing it up a bit.

Behind the scenes we’re using CircleCI to consume our GitHub webhooks and run our builds. And we’ve got a simple Nginx proxy that routes requests to a single S3 bucket on Amazon Web Services. Set it and forget it.

Let’s take a look at how this works.

### Hosting a static website on Amazon’s Simple Storage Service (S3)

Unless you’ve been coding under a rock, you’ve probably at least heard of [Amazon’s cloud storage solution](https://aws.amazon.com/s3/), S3. Aside from its security, availability, and simple interface for storing objects in the cloud, you can also use it for hosting static websites. It’s perfect for serving HTML, CSS, JavaScript, and static assets like images, videos, etc.

Setting up a static S3 host is pretty simple to do, but that’s not what I’d like to cover here. If you’re just getting started with S3, take a few moments and run through the [AWSdocumentation](http://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html).

I’ll be working with a bucket called happycog-static that has website hosting enabled. And within that bucket, I’ve also created a folder called cognition-s3. As you can see, [we’re ready to roll with S3](http://happycog-static.s3-website-us-east-1.amazonaws.com/cognition-s3/master/hello.html).

### Using Nginx as a proxy for Amazon S3

Not that any of what we’re doing is actual magic, but if it were, this is when I would start to reach into my black hat. Nginx is pretty great. It’s extremely powerful, and configuring servers is actually quite simple and enjoyable. You might be asking yourself… why do we need Nginx if we’re talking about static websites and S3? And you’d be sorta right. We’re only using it as a proxy for that ridiculously long amazonaws URL above. And here’s how:

We’ve got a domain name: cogclient.com. In our DNS, We’ve created an A record that tells any requests to *.cognition-s3.cogclient.com to forward to our Nginx proxy’s IP address. If you’re doing the math here, you can probably guess what’s about to happen. But first, let’s take a look at our S3 proxy’s Nginx config:

<iframe src="https://medium.com/media/42de334b76d08a6de73abc9a9f4cac85" frameborder=0></iframe>

All in, this is a pretty vanilla server block. We’re listening on port 80, we’ve defined a server_name, and we’ve set some log paths to debug our errors and track our access. In our location block, we’ve got some special things going on. Let’s break them down:

    resolver 8.8.8.8;
    set $bucket "happycog-static.s3-website-us-east-1.amazonaws.com";
    rewrite ^([^.]*[^/])$ $1/ permanent;

First things first, we’ll resolve all of our requests to Google’s public DNS. Since that’s the easiest way to ¯\_(ツ)_/¯ and still have domains work. Next, we’ll set a variable for our root S3 bucket URL and do a quick rewrite to add a trailing slash to all of our URLS. This is important since we’re using nested objects in our bucket and hosting our site inside of a sub-folder. Otherwise, URLs would default to the root bucket and nothing would work.

    # matches: branch-name.repository-name
    if ($host ~ ^([^.]*)\.([^.]*)\.cogclient\.com) {
        set $branch $1;
        set $repo $2;
        proxy_pass http://$bucket/${repo}/${branch}${uri};
    }
    # matches: repository-name
    if ($host ~ ^([^.]*)\.cogclient\.com$) {
        set $repo $1;
        proxy_pass http://$bucket/${repo}/master${uri};
    }

What we’re doing here in our $host conditionals is how we support multiple branch deployments with unique URLs. Since Nginx will process these in order, we’ll check for branches first before we default to our master branch. In our branch conditional, we’re telling Nginx to use the first and second regex matches as branch and repo variables before we pass the request off to our fully qualified S3 URL. In our master conditional, we’re only looking for a single match. This allows us to view the latest and greatest without having to access master.cognition-s3.cogclient.com. Mind you, this will also work if you choose. And you can ignore the second conditional. But I’m a fan of clean, precise URLs. And this is a matter of preference, really.

### A Few Examples

**Staging**: [http://staging.cognition-s3.cogclient.com/hello.html](http://staging.cognition-s3.cogclient.com/hello.html)
**UAT**: [http://uat.cognition-s3.cogclient.com/hello.html](http://uat.cognition-s3.cogclient.com/hello.html)
**Master**: [http://cognition-s3.cogclient.com/hello.html](http://cognition-s3.cogclient.com/hello.html)

So what we have now is a unique, publicly accessible URL to a build for any branch that we create on our cognition-s3 project.

    roxy_intercept_errors on;
    proxy_redirect off;
    proxy_set_header Host $bucket;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_hide_header x-amz-id-2;
    proxy_hide_header x-amz-request-id;

The settings here are what tell AWS to allow the forwarding to take place and help the proxy identify itself appropriately. And that’s about it for the Nginx piece of this puzzle.

### Continuous integration and delivery with CircleCI

Are you having fun yet? If not, this is my favorite part. With our static hosting configured and an Nginx proxy distributing requests based on URL structure, we tie it all together with a deployment tool. While there are a handful of services and products out there that can listen and respond to web hooks, I’ve become a huge fan of CircleCI. I could talk your ear off about why, but I’ll limit it to a few very high-level benefits. It integrates nicely with GitHub, which makes it really simple to configure a project. It supports a ton of build tools out of the box. You can run native tests on your builds, and there’s built-in support for deployment across all of the major cloud providers. Most of all, your first build server is free, and you can share it across all of your projects. If you’re interested in diving deeper, their [documentation](https://circleci.com/docs/) is top notch. What I will focus on is configuring AWS permissions for your project and creating a circle.yml file that defines the build and fires a deployment script.

Once you have authenticated with GitHub, and your CircleCI project is ready to build, click through to your settings and find the AWS Permissions section. For this to work, you’ll need to generate an IAM user in your AWS account with a role that allows full access to S3. There’s great [documentation](http://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html) for that too. The keys you add to your project are what allow CircleCI to manage your bucket contents directly on their build server through the AWS command line utility.

Now let’s look at our circle.yml file. This is how we tell CircleCI what we need for each of our builds. In this case, we’re keeping it extremely lightweight. But don’t be fooled, you can do much more. If you’re curious, have a look at what else is possible.

<iframe src="https://medium.com/media/e3babfd7038cc7d2bb200fd996da56f1" frameborder=0></iframe>

If we break this down, it’s pretty easy to follow. Our build requires node 5.1.0 with npm and bower dependencies. We’re going to cache our node_modules directory so we can save a little time on our build and avoid running npm install on each push. Since we’re going to trigger a deployment on every branch, we’re adding our grunt build task to test override. Once our build server is up and our static site is built, we’re going to tell CircleCI to execute s3-deploy.sh, which is what puts our static files into the cloud. To top it all off, we’ll notify the rest of the team in our Slack channel that there’s a new deployment.

<iframe src="https://medium.com/media/70fc5b95bda4423af41530b6cbc10051" frameborder=0></iframe>

### Why? Because, it’s in the cloud.

We’ve been using this approach at Happy Cog for a few months now on several of our projects — ranging from plainHTML pages and Jekyll sites to full Pattern Lab build deployments. It’s been working out very well so far. Not only is it a simple addition to our existing git-flow and feature / issue branching strategy, but we’re also able to maintain an archive of fully functional branch deployments across all of our projects. We’re not maintaining a server, creating or managing virtual hosts, or ever worrying about disk space. And the best part is, it’s pretty cheap.

![[Dan DeLauro](http://happycog.com/delauro)](https://cdn-images-1.medium.com/max/2000/1*PfAm90aTUGVlS0g5oLbYQA.jpeg)*[Dan DeLauro](http://happycog.com/delauro)*

![*This post was authored by [Dan DeLauro](http://happycog.com/delauro) and originally appeared on [Cognition](http://cognition.happycog.com/article/deploying-static-websites-to-aws-s3-behind-an-nginx-proxy) on June 10, 2016.*](https://cdn-images-1.medium.com/max/2000/1*dUxq5EbGOI8vsez3PjV6oQ.png)**This post was authored by [Dan DeLauro](http://happycog.com/delauro) and originally appeared on [Cognition](http://cognition.happycog.com/article/deploying-static-websites-to-aws-s3-behind-an-nginx-proxy) on June 10, 2016.**
