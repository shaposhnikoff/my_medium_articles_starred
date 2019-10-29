Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m447[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m12[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m10[39m }

# Hosting Static Websites on AWS

Hosting Static Websites on AWS

In many cases a website only consists of static content, e.g. HTML pages, CSS and JavaScript files, images, etc. Examples include blogs managed by tools like [Jekyll](https://www.jekyllrb.com/) or modern *Single Page Applications* (SPA) backed by frameworks like [Angular](https://angular.io/), [React](https://reactjs.org/), etc. Traditionally, when it comes to hosting these kinds of websites a webserver is used. Although setting this up is no real challenge and step-by-step tutorials are easy to find it has some disadvantages.

First, the software running on the server must be updated regularly at least when it comes to applying security patches. Of course this often simply is not done. Next, servers can fail which means that the website is not available at least temporarily. If the website is used for selling products or services the corresponding stream of revenue runs dry. Furthermore, your website might get some serious attention which might result in a high amount of traffic. Running only on one server might mean that the site “gets slow” and the experience for your users degrades. Again, this might have a negative impact on money earned with the website. Last but not least, you are paying for running the server even if no users are visiting your site.

Modern public cloud offerings, for example the ones provided by *Amazon Web Services* (AWS), allow hosting static websites by combining multiple managed services. In this case, the shortcomings outlined above are not relevant since the services are regularly updated, are highly available and scale automatically. This means (almost) no operations overhead. Another nice benefit of such a solution is its cost effectiveness. In general, the costs scale with the number of visitors and the amount of traffic instead of the time a server is up and running.

Unfortunately, setting up such a solution is not very easy either and requires some knowledge about the platform and its services. This makes it hard for beginners to benefit from the advantages of using the public cloud services.

Therefore, I’ve created a CloudFormation template which can be used by (almost) anyone to set up all the services in a couple of minutes. Additionally, common best practices for such a solution are already applied.

## Services

The solution basically combines the following four services provided by AWS: *Simple Storage Service (S3)*, *Route 53*, *CloudFront* and *Amazon Certificate Manager (ACM)*.

![](https://cdn-images-1.medium.com/max/2000/0*ymwarrW5h-evOWeu.png)

[Amazon S3](https://aws.amazon.com/s3) is used to store the actual contents of the website — HTML pages, images, CSS files, etc. Additionally, all the logfiles are stored in S3 as well.

The domain of the website is managed by [Route 53](https://aws.amazon.com/route53) and associates the CloudFront Distribution with a domain name.

[CloudFront](https://aws.amazon.com/cloudfront) is a *Content Delivery Network* (CDN) which provides the benefit of accelerating the delivery of the content of the website to your users. Additionally, it’s the only way for enabling encrypted data transfer via SSL in this scenario.

The SSL certificate itself is provided and managed by the [Amazon Certificate Manager (ACM)](https://aws.amazon.com/acm). The services ensures that it is regularly and automatically renewed.

## Prerequisite

What is needed to get started with hosting your static website on AWS? First of all, you need an AWS account. If you do not already have one you can [create one](https://portal.aws.amazon.com/billing/signup?nc2=h_ct&src=header_signup&redirect_url=https%3A%2F%2Faws.amazon.com%2Fregistration-confirmation#/start) in a few minutes. All you need is an email address and a credit card.

Next, within AWS offers its services in multiple regions all over the world. Because of some specific requirements imposed by CloudFront we need to set up everything in the region “North Virginia” (us-east-1).

For actual end to end automation of the setup a hosted zone for the domain of the website needs to be created in Route 53. You can either do this manually in the AWS Console or automatically by using a simple CloudFormation template I’ve created.

Last, since all communication with the website should be encrypted via SSL an application needs to be installed to automate the creation of the appropriate DNS records needed by ACM to verify the certificate request. The application is called [ACM DNS Record Manager](https://serverlessrepo.aws.amazon.com/#/applications/arn:aws:serverlessrepo:us-east-1:022876999554:applications~acm-dns-record-manager) and can be easily installed from the *Serverless Application Repository* at no additional costs. It creates a SNS topic to which the events must be send. The ARN of this topic must be configured when launching the CloudFormation stack.

## Setup

The actual configuration of the services is done by [CloudFormation](https://aws.amazon.com/cloudformation) based on a [template](https://github.com/jenseickmeyer/cloudformation-templates/blob/master/static-website/static-website.yaml) I’ve created. If you are familiar with the [AWS CLI](https://aws.amazon.com/cli) you can create a new CloudFormation stack using the following command:

    aws cloudformation create-stack --stack-name example-site --template-url https://s3.eu-central-1.amazonaws.com/com.carpinuslabs.cloudformation.templates/static-website.yaml --parameters ParameterKey=HostedZoneId,ParameterValue=Z2CV0JKAQA6QWN ParameterKey=DomainName,ParameterValue=www.example.com --notification-arns arn:aws:sns:us-east-1:123456789012:serverlessrepo-acm-dns-record-manager-CloudFormationEventsTopic-TC763GODBOWM --region us-east-1

The script has two mandatory parameter:

* HostedZoneId: This is the ID of the hosted zone in Route 53 to which the DNS record for associating the CloudFront distribution with your domain will be added. Additionally, the DNS records for validating the SSL certificate by ACM are added to this Hosted Zone as well.

* DomainName: The domain of the website.

Launching the stack only takes a couple of minutes. After that you can push the content of your website to the appropriate S3 bucket and access it through your browser.

## Costs

As said before, the costs for the solution described above are pretty low. Most of them are dependent on the actual usage of the website. This means that they scale with the number of users visiting your site.

Calculating the actual costs for your site is difficult. It depends on the usage characteristics of your site. For example, you pay for each request send to CloudFront. This means that the costs for serving lots of small files is more expensive than serving fewer but bigger ones. I think it’s safe to say that hosting a website on AWS costs approximately $1 per month. This would already include some basic traffic (~ 4 GB / month). Additionally, the major benefit is that you do not need to manage any servers or think about the availability and scalability of your site. This is all done automatically for you.

## Next Steps

Depending on the kind of content you are hosting using this solution there are other things you could do.

For example, one usage scenario is hosting some content which should not be available to the whole world, like *Single Page Applications* (SPA) intended for internal usage, internal documentation and reports, etc. In this case you probably want to restricting the access to your website by either limiting it to a certain IP range or by requiring the authentication via login credentials. In the latter case you can have a look at the solution for [extending CloudFront with a Basic Authorizer](https://scratchpad.blog/2018/09/20/cloudfront-basic-authorizer.html).

Furthermore, if the content is generated by some tool, like Jekyll is generated for this blog, you can automate the whole process for generating the content and publishing it. This way it is very easy, convenient and safe to publish new content. You can do it either by using a script that you can run on your laptop everytime you want to publish your site. If you are managing the contents of your site in a source code repository like git you can automate this even further by [setting up a so-called CI/CD pipeline](https://scratchpad.blog/2019/01/31/a-build-pipeline-for-jekyll-sites.html) which takes of pubishing whenever the source code of your site has been updated.

*Originally published at [scratchpad.blog](https://scratchpad.blog/serverless/hosting-static-websites-on-aws/) on January 25, 2019.*
