
# Moving a static website to AWS S3 + CloudFront with HTTPS

This Christmas I managed to pick up the worst cold I’ve had in years. Instead of heading out and crushing the Festive 500km challenge I decided to stay home and brush up on some tech while performing some website housekeeping for willmorgan.co.uk.

In the 2 years I’ve had the site online, I’ve picked up some better practices and learned a few tricks. I had a few specific goals in mind:

* **Enable HTTPS
**Google will start penalising websites that don’t support HTTPS in 2017.

* **Ensure the website can be developed and deployed from anywhere
**I started developing on my desktop, but when doing some updates using a new machine, it was painful getting a development environment set up.

* **Simplify and automate build tooling
**My current build process is error prone and slightly temperamental

* **Reduce the financial and time costs of hosting
**Considering this is a single static website, paying $5/pcm for hosting on DigitalOcean is too much money, and it requires too much maintenance. Also, after messing around with terrible shared hosting providers back in the early 2000s, and once again this year, I’m never doing that again.

## Stack review

### The website:

* Comprises of entirely static HTML, CSS, JS, images

* Uses Gulp to build the website — minification, concatenation, image compression, etc.etc

Gulp’s method of piping output is elegant and gives you a lot of control — whether or not you need it. These days I’m using npm scripts to perform most tasks, which is appealing as this approach requires no extra libraries, no adapters around vanilla command line tools, and it also makes some useful assumptions about your environment when running scripts in this way, such as adding your locally installed Node binaries to the script environment which cuts down on needing to install libraries globally.

I’ve also been using Webpack in production for other projects. Unlike Gulp, it covers both building and asset loading. It’s also mostly configuration based, and offers a variety of plugins to customise its functionality. It still requires a bit of adapteritis and wrapping, and a little more configuration than a beginner might be comfortable with, but it does a good job.

### The old environment:

* Sits on a DigitalOcean Ubuntu + Apache box for hosting

* DNS hosting is provided by AWS Route 53

* Requires a .htaccess rule and some environment variables to ensure that the built code is served up

* Uses Vagrant to bring up a VM for development

The DigitalOcean box is overkill for hosting a static website, and it’s probably time to phase it out. The trend of devops and cloud solutions really hits home here — the cumbersome tasks of keeping on top of patches for the latest CVEs, handling software updates and backups is a drag.

As the whole site is deployed at the moment, I rely on some environment variables to switch between source and distribution directories when serving the site. I’d like to minimise the amount of environment-specific code — switching public directories within the Apache config file is not ideal when it’s unreliable — even if that’s because of my own lack of knowledge!

Finally, building a 1GB Vagrant VM for development of a static website is mad. One alternative I found is the [http-server](https://www.npmjs.com/package/http-server) npm package. It’s configurable enough to handle setting up SSL and CORS, and even handles GZIP on a basic level. Whereas Vagrant is brilliant at providing a padded environment, a more portable library that can be installed alongside a project’s devDependencies is surely preferable.

### Deployments:

* Theoretically work automatically with Git hooks, but my code on the server is temperamental…

* Require a commit of the built source code in to the dist/ folder.

At one point this was all working smoothly — a GitHub post-receive hook would go and call a deploy script on my server, which would go and pull the latest changes so they’d get displayed on the production server. After a few years and a few updates to Apache / Ubuntu, this is no longer the case.

Exposing deployment functionality on a production environment, even if it’s protected behind some authentication, is slightly insecure and breaks the pattern of “static everything”. Having one-way deployment here would be preferable.

Making sure that I ran a command before committing and pushing changes to the Git repository, in this case gulp build to minify scripts, kept catching me out. The built files don’t provide much value, either — when it comes to tracking history, when the legible source files are available, I’d rather read the source files, not the build artefacts.

![Not really that useful.](https://cdn-images-1.medium.com/max/2000/1*BGeXUD4dT8LCZ86Vew0fWg.png)*Not really that useful.*

These days I’m relying more on CI services to perform build tasks in a neutral, easily-reproducible way. Scripting functionality means you’ll never forget to run gulp build again, and it also means that you can deploy from any machine, as long as you’ve got git push access.

## The new stack

At long last, I’ve moved to AWS S3 and CloudFront to serve the site statically. I’ve got [Travis CI handling builds and uploads](https://docs.travis-ci.com/user/deployment/s3/).

There’s a slight learning curve to getting these services configured, but if you’ve never done it before, here’s a quick primer:

* S3 (Simple Storage Service) is a storage service that lets you upload stuff to the internet. You have control over things like file permissions, and get a suite of functionality to transform files in some way. You can host websites with S3 with a bit of magic, although you might require AWS’s DNS hosting service, Route 53, to make this process a little easier.

* CloudFront is a CDN that replicates your content across multiple regions. It handles load balancing, compression, caching, and everything you might want out of a CDN.

* You configure CloudFront to “sit in front” of S3, so it pulls your content from S3 and then distributes it to all of its available regions. As with a ll CDNs, the end result is your site loads quickly regardless of its geographic access point.

* Travis is a hosted continuous integration service for your builds. New builds are run on pushes to a Git repository: a throwaway environment is created, your tasks are run, and depending on how they exit, a build is deemed a success or a failure. It has a free tier for public repositories, supports a wide range of languages, and requires just 20 lines of configuration to handle uploading builds to S3.

### S3 upload and configuration

To set up your S3 bucket, sign in to the AWS console and follow these steps:

1. **S3 -> Create Bucket**
Choose a bucket name (your domain name, optionally with a stage, like dev-willmorgan.co.uk). Unless you have specific requirements for AWS region, you can use the cheapest region for storage, which is US Standard (us-east-1) at time of writing. The region is mostly irrelevant as CloudFront distributes your site locally anyway.

1. Select your bucket, go to **Properties -> Static Website Hosting, **then **Enable Website Hosting
**You can also access Properties by choosing it on the dropdown menu triggered by right clicking the bucket name.

1. You will need to set the **Index Document** to your website’s index file, normally index.html. You should also probably set the **Error Document**, too.

1. Still under **Static Website Hosting**, if you have any redirect rules, you can set those up under **Edit Redirection Rules** using the [somewhat verbose XML syntax](https://docs.aws.amazon.com/AmazonS3/latest/dev/HowDoIWebsiteConfiguration.html#configure-bucket-as-website-routing-rule-syntax).

1. Now go to **Properties -> Permissions** (it’s the panel above Static Website Hosting). Although you’ve just configured S3 to behave like a web server, file access is still restricted by permissions, which are private by default.
You just need to alter your bucket policy to make all uploaded files public by default. This is akin to performing a chmod on your server’s web root. 
Selecting **Edit bucket policy** and allow access to the s3:GetObject action. You can [customise your own policy using the generator](http://awspolicygen.s3.amazonaws.com/policygen.html), or follow the template below:

    {
     "Version": "2012-10-17",
     "Id": "PublicBucketPolicy",
     "Statement": [
      {
       "Sid": "Stmt1482880670019",
       "Effect": "Allow",
       "Principal": "*",
       "Action": "s3:GetObject",
       "Resource": "arn:aws:s3:::***YOUR_BUCKET_NAME***/*"
      }
     ]
    }

After customising these settings you’ll need to save each section. Luckily, AWS does a good job of validating your configuration on entry, so you won’t need to debug bad config later.

Finally, **upload your built website to S3**. We’ll handle automatic deployment later —[there’s an open pull request that makes auto cache busting a little easier](https://github.com/ampedandwired/html-webpack-plugin/pull/535).

Make a note of the **Endpoint** listed in the **Static Website Hosting** panel as you’ll need this later.

### SSL certificate acquisition

AWS provides free SSL certificates, which work with browsers that support SNI. You can import your own SSL certificate, of course, but for my purposes, a free one will do. When creating an SSL certificate, consider whether you want to support your naked domain as well as the www subdomain.

If you decide to request a certificate with Certificate Manager, make sure your technical / administrative contact email addresses are accessible so that you can verify the request.

All in all, this process is straightforward.

### CloudFront configuration

The S3 and CloudFront integration experience is *mostly* seamless.

To get this set up, head over to the CloudFront page within your AWS console and follow these steps:

Create a new **Web** distribution. Under **Origin Settings**, use the **Endpoint** you copied from your S3 bucket’s static hosting setup. Many other guides instruct this without explaining why — this is because if you have redirect rules configured with your S3 bucket and you specify the internal AWS S3 resource, the redirects will no longer work. Therefore, you must specify the website endpoint domain to ensure redirection functionality works.

Under **Default Cache Behavior Settings**, it’s worth selecting *Redirect HTTP to HTTPS* and narrowing down the **Allowed HTTP Methods** — for a static website, *GET and HEAD* will do.

Importantly, due to CloudFront’s robust caching, you’ll need to work on a mechanism to serve updated files — one way to handle this is to define a cache based on a version query string, but you can always use another filename if you want to update in the future.

![](https://cdn-images-1.medium.com/max/2000/1*aC3YA8sF_rEYg72awUHYGg.png)

Don’t forget to check **Compress Objects Automatically**.

Under **Distribution Settings**, you’ll need to:

1. Set your domain names under **Alternate Domain Names** (**yourdomain.com**, and optionally, **www.yourdomain.com**)

1. Configure SSL by selecting **Custom SSL Certificate**, then choosing the certificate generated or imported in to **Certificate Manager**.

1. Finally, specify the **Default Root Object**. This should match your S3 bucket’s **Index Document**, usually index.html. This is simply where all requests are redirected to when a client requests your website URL without a path.

Your S3 and CloudFront configuration is now complete. This is mostly set and forget configuration, and provides a nice separation of infrastructure as opposed to getting tied up with software-specific configuration files like .htaccess and httpd.conf.

At this point, the status of your CloudFront distribution will most likely be *In Progress*.

Keep a note of your **CloudFront Domain Name** as that’ll be needed next.

It’s worthwhile waiting until the status is *Deployed* before thinking about moving to production. In the meantime, Route 53 needs configuring.

### DNS configuration with Route 53

At this point, you’ll have a **CloudFront Domain Name** like somewebsiteid.cloudfront.net. Now it’s time to get this domain aliased to your primary domain with Route 53. If you’re sensible, I recommend testing with a trial subdomain first before switching out your old A/AAAA records and promoting your AWS version to production.

If you don’t yet have your DNS hosted with Route 53, I’ve got to say that the (modest) additional cost is worth it. It’s also a requirement, because this configuration requires the use of an ALIAS record, which is a special kind of DNS record that works similar to a CNAME by specifying an AWS resource.

Here are the steps, assuming you’ve got your DNS hosted with Route 53, and you’ve already created your basic Hosted Zones:

1. If you already have A and AAAA record sets (that’s IPv4 and IPv6), you’ll want to edit the Record Sets. Otherwise, if you’re starting fresh, you’ll need to create them.

1. For every domain you want to serve your CloudFront website (your naked domain, like, mydomain.com and optionally its www counterpart, www.mydomain.com), you will need to create or update both A your and AAAA records to be aliases to your CloudFront distribution, like so:

![](https://cdn-images-1.medium.com/max/2000/1*Shn5TU7-2d1D1p-mEOvZJw.png)

At the end of this, you should have a record configuration like below:

    **Record   Domain             Alias Target
    **A      | mydomain.com     | somedistributionid.cloudfront.net
    AAAA   | mydomain.com     | somedistributionid.cloudfront.net
    A      | www.mydomain.com | somedistributionid.cloudfront.net
    AAAA   | www.mydomain.com | somedistributionid.cloudfront.net

Bear in mind that although Route 53’s DNS changes are near-instant these days, it‘ll take time for CloudFront to sync your S3 bucket across its distribution regions. If you see these strange things happen immediately after you’ve made these changes, double check the configuration, but it might be a matter of waiting for the changes to propagate until CloudFront’s distribution status is *Deployed*:

* If you’ve just set up the SSL certificate, you might see certificate errors on the domain you’ve set it up for

* Redirections to your S3 bucket’s URL

Testing your site in a private browsing tab with cache disabled is the sanest way to test this — as well as perhaps having a friend on another network sanity check it for you…

![Thanks, Dan.](https://cdn-images-1.medium.com/max/2328/1*aQh2ugzp3GYbo5YLs6hmvQ.png)*Thanks, Dan.*

It turns out that it was a matter of waiting for CloudFront to fully deploy. A lesson learned for a more important client website in the future…

![Success!](https://cdn-images-1.medium.com/max/2000/1*gG02hUxB2HKVOWQIOEQUfw.png)*Success!*

It did happen, in the end!

## In summary

I’ve yet to see actual traffic costs for CloudFront and S3, but I’m getting a faster, truly static and truly scalable website for a fraction of the cost of managing my own machine.

I’ve now got a much lighter setup, which means I can run npm install, then run http-server inside my build directory. Far more portable.

Webpack, along with the HtmlWebpackPlugin, handles bundling and outputting a built website. Travis then handles the upload. All my machine needs to have is a text editor — the rest of the build configuration is out of the picture, which is a benefit for use cases where I’ll want to fiddle on a new machine and quickly push.

The source code for the project is open source — you can view it here:
[https://github.com/willmorgan/willmorgan.co.uk](https://github.com/willmorgan/willmorgan.co.uk)

My next post will cover using Webpack, Git and Travis to perform automatic builds and deployment. Hopefully this one was useful!

I’m also available for hire: jobs+medium [at] willmorgan.co.uk.
