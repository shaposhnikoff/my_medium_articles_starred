
# Tutorial: How to add SSL certificates to Elastic Beanstalk and CloudFront

AWS makes it easy to add certificates to certain services. Certificates are free and easy to set up.

![](https://cdn-images-1.medium.com/max/2000/1*8KwNzfola5AWNhvOer8Nfg.png)

### Why do you need SSL?

For anything even vaguely production and/or Internet-facing, I cannot stress enough how much you should be using SSL. This buys you a number of important benefits:

* First, it goes a long way to helping prevent the most trivial attacks on applications. The most obvious one is the non-SSL WordPress site with a username and password that can be read by anyone when you log-in (it’s transmitted as plain text across the Internet). But any http site with a login page is highly vulnerable to this kind of attack.

* Google has decided (wisely) that anyone with a serious website should really be using SSL at this point, and they have influenced their ranking algorithms accordingly. I expect they will get more aggressive on this stance in the future.

* It makes your sites look more professional and trustworthy to visitors, especially if you accept any personal information or payments in your application.

* The Chrome browser identifies http/https sites as simply “Not Secure” or “Secure” — depending upon your audience, this could be interpreted in different ways. Being “Secure” is clearly a better option here.

SSL is fairly mysterious to many people but it doesn’t need to be. AWS Certificate Manager handles most of the difficult work in implementing SSL (including renewals every 13 months). It’s also free. With the exception of OpenSSL and LetsEncrypt, most SSL certificates cost money, so if you’re managing many domains this is a useful benefit.

I put together this quick tutorial since I’ve met a number of people recently who have Elastic Beanstalk (EB) apps and CloudFront distributions running without a cert, and they simply didn’t realize it was so easy to set one up. If any of this isn’t clear, please let me know and I’d be happy to help.

### Creating the Certificate.

1. Configure a usable email address at the domain name.

Ensure the technical contact email in your domain is something you can access — AWS will send a confirmation email to this address. Alternatively setup email handlers at one of admin@*your domain or *webmaster@*your domain *if the first option isn’t possible. We’ll be coming back to this.

2. In the AWS Console, make sure you are in the correct region where your EB application is running.
> Important gotcha: CloudFront is a global service in AWS so you **must **create the certificate in us-east-1 (N. Virginia) or it will not be visible in CloudFront.

Go to Certificate Manager and click **Request a Certificate**. In the domain list, add the domain and any subdomains you want to include. If the www and non-www versions of your site redirect to each other, add both here. Click **Review and Request**. If you are only doing this for a CDN, then cdn.*yourdomain*.com is all you need here.

![](https://cdn-images-1.medium.com/max/2246/0*Ark4kHSdXRzNOtUf.)

On the next page, check your entries and click **Confirm and request**.

![](https://cdn-images-1.medium.com/max/2224/0*Iu4jzP5n0mbt7wrx.)

Finally, on the Validation page, ensure you have an active email address in on of the listed emails in the dropdowns, and click **Continue**.

![](https://cdn-images-1.medium.com/max/2232/0*3Fg9iqVm5T87dPC6.)

A confirmation email will arrive for each subdomain you listed in the certificate so click the link in every email.

![](https://cdn-images-1.medium.com/max/2000/1*U1I86nF-XtAiKsmvEiSCtQ.png)

Congratulations! You now have a certificate available in Certificate Manager for your domain. Now for hooking this up with Elastic Beanstalk or CloudFront…

![](https://cdn-images-1.medium.com/max/2254/1*eI0Ugz_2bdblQYBdUz0yVA.png)

### Elastic Beanstalk SSL setup

I’m assuming you have an existing workload running on EB and a custom domain. In the EB console, go to Configuration → Network Tier → Load Balancing and choose your SSL certificate from the dropdown.

*If this option isn’t available, then you’re not using a load balancer and this whole process isn’t going to work for you. For production-facing EB applications, I would strongly suggest using a load balancer.*

### CloudFront Distributions

CloudFront is the AWS CDN and it’s great for delivering assets cheaply and efficiently for your web applications.

When you create a distribution, however, you get an ugly sub-domain in the format https:// d1234589abcde.cloudfront.net. While there’s nothing wrong with this, wouldn’t it be great if it were served from https:// cdn.*yourdomain*.com?

1. Assuming you already have a distribution configured, your CloudFront console looks something like this:

![](https://cdn-images-1.medium.com/max/2000/1*39sgi2Tyw_cqtDRaOgJtxw.png)

Click the distribution you want to add the certificate to, and click **Edit**.

In Alternate Domain Names, enter the subdomain you set up for the certificate (this is important, or you will get a cipher mismatch error later on). Click the radio button for “Custom SSL Certificate” and in the dropdown select your new certificate. Add the bottom of the page click **Yes, Edit **(I have no idea why this isn’t called Save Changes).

![](https://cdn-images-1.medium.com/max/2000/1*iUwg6Zjdvw4BKh7niZJfsA.png)

In the main CloudFront screen, the Status column will indicate when this is ready (it can take 15–20 minutes). Wait for the status to change from In Progress to Deployed:

![](https://cdn-images-1.medium.com/max/2000/1*AQJz4DvbdoddyzNApN5fTQ.png)

4. In your DNS settings for your domain, you will need to add a CNAME record mapping the subdomain to the CloudFront distribution name. The steps here vary by registrar but it looks like this on Name.com:

![](https://cdn-images-1.medium.com/max/2000/1*DYy2zoJp2aYtJl41vqNqEw.png)

Once these changes are propagated (which can take anywhere from 10 minutes to 48 hours), you can test if it works by attempting to load an image through the CDN using the new URL.

## Quick Summary

You can add your own SSL certificate for *yourdomain.com* for certain AWS services using Certificate Manager — it’s free and manages the expiration and renewal process.

You can wildcard (*) all subdomains in a single certificate but personally I prefer limiting a certificate to a particular usage (e.g. one certificate for www or <blank> for the web application, cdn.*domain.com* for CloudFront, etc.)

You *cannot* use Certificate Manager to set up certificates on EC2 directly, as of July 2017. AWS requires that you place a load balancer in front of the EC2 fleet, which actually is easier to manage though it increases the cost marginally.
> It takes 30 minutes to complete this process in most cases, and most of that time is waiting for distributions on CloudFront or CNAME propagation.
