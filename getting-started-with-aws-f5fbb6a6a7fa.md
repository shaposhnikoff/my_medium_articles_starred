
# Getting Started with AWS

An Intro to AWS and Identity Access Management (IAM)

![Photo by [Samuel Zeller](https://unsplash.com/@samuelzeller?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/11886/0*B4GCFI7DQ_9bqaAJ)*Photo by [Samuel Zeller](https://unsplash.com/@samuelzeller?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

My post on VPC, S3 and DynamoDB did well but got feedback that a more introductory article would be useful for some people. If you do find this useful then please check out my other articles:

* [AWS S3 Overview](https://medium.com/@lewisdgavin/aws-s3-overview-6fe9ca2a1e0a)

* [AWS VPC for Beginners](https://medium.com/datadriveninvestor/aws-vpc-for-beginners-eee01e7083d1)

* [AWS DynamoDB Overview](https://medium.com/@lewisdgavin/aws-dynamodb-overview-184e53aedcd6)

This post and all those linked above are a brain dump of what I’ve learned on my journey in preparation for the AWS Developer Associate Exam.

## What is AWS?

Amazon Web Services is a cloud platform that allows fast and cheap provisioning of environments. This gives businesses the opportunity to build environments, scale them when their requirements change and test at a low cost, all with a few clicks.

The idea here is that instead of going to an external hardware infrastructure supplier, working out sizing and cost and having fixed contracts — you simply go to AWS and provision all this yourself, with no fixed contract, scaling up and down based on demand.

## Regions and Availability Zones

In order for this to work, there obviously needs to be some physical hardware infrastructure somewhere. Amazon splits theirs into **Regions**. [Amazon currently has 16 regions](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html) (as of writing this), this number is steadily increasing. These regions relate to geographic locations of data centres.

As an example, there is a region called *US East (Ohio)* and another called EU (London). That means that there are data centres in the specific geographic locations. When building an AWS instance, you want to make sure the Region selected is the one closest to those who will be accessing it frequently, to reduce latency. For me this was EU (London) as I am based in the UK.

Each region has a number of locations within it that are called **Availability Zones**. These can provide failover functionality so if one Availability Zone is down, the others should be isolated and therefore available. They are designed to be isolated but have low latency connections to other Availability zones in the same Region.

![](https://cdn-images-1.medium.com/max/2000/0*vBFrDTMS-IUmj4z1.png)

*Image taken from [AWS Docs](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/images/Con-AZ.png)*

## Edge Locations

As you can imagine, if you are accessing a service that is hosted in a particular Region, let’s say from the US, however are actually based in the UK like me. Then your latency will obviously increase as the data must be transferred across a larger physical distance.

To resolve this, AWS has the concept of **Edge Locations**. Edge Locations are just data centres that hold cached data that would have at some point in time, been transferred from the origin server. Now when someone wants to access that content, they have a much lower latency as they are accessing a cached copy that is local to them, rather than pulling it from another region.

Edge locations tend to be hosted within highly populated areas and naturally there are more edge locations that regions.

## Route 53 — DNS

Route 53 is the AWS Domain Name System (DNS) web service. It’s goal is to translate domain names to IP addresses so web content can be served to users.

![](https://cdn-images-1.medium.com/max/2400/0*LlG4ls8GpbCYhPhy.png)

*Image taken from [Loggly.com](https://www.loggly.com/wp-content/uploads/2014/09/route53howitworks.png)*

It gets its name from using the iconic Route 66 in America as a metaphor. The reason the number 53 was used as this is the port used for DNS Server requests.

## IAM

IAM stands for **Identity and Access Management.** IAM allows you to secure how people access resources and services within AWS. You can create Roles and Groups and assign them to users.

You can use IAM for other tasks such as enabling Multi Factor Authentication and enforcing a password policy for your users.

## Groups, Users, Roles and Policies

A **Group** in AWS is just as you would expect. It refers to a cluster of Users. This allows you to apply roles and policies to a group, and every user within that group will automatically have these roles and policies.

A **User** relates to an account that in most cases is for a single individual. As stated before, User’s can be added to groups to obtain their roles and policies. You can also add further roles and policies to an individual user if they have a special requirement that falls outside of their assigned Group.

A **Role** is a way to provide delegate access to AWS services or resources. There are a set of policies attached to a role similar to a standard User, however a role is intended to be used by whoever requires access to a particular service or resource within AWS, that wouldn’t usually have access to it.

For example, you want to grant a mobile application access to a service without embedding access keys directly into the app. Or you want to provide access to an AWS resource to someone in another AWS account. The idea is that you delegate access via Roles.

**Policies** specify a set of permissions. These permissions then allow users or Roles to perform certain activities. Policies are a great way to specify exactly what activities can and cannot be carried out. These policies can the be assigned to Users or Roles essentially defining what they can do.

## API Calls and Authentication

If you have a service that needs to continually connect to AWS, you don’t want to store access keys within the application in order to connect each time. This could allow people to extract them and therefore have access to your AWS instance.

To get around this, for web applications, AWS uses Web Identity Federation. Here a user authenticates with the origin application first, this application then gives the user a token that they can then use to call AWS using the AssumeRoleWithIdentity API. This will then grant the user temporary security credentials.

The same process flow applies when using Active Directory to authenticate against AWS. You log in to AD using Single Sign On (SSO) for example, the browser receives a SAML assertion from the AD server. The browser then posts this SAML assertion to the AWS SAML end point using the AssumeRoleWithSAML API to request temporary security credentials. The user can then access the AWS console.

I hope this intro helped and if you haven’t already, check out my other articles on this subject. They will be especially helpful if you are studying for the AWS Developer Associate Exam.

* [AWS S3 Overview](https://medium.com/@lewisdgavin/aws-s3-overview-6fe9ca2a1e0a)

* [AWS VPC for Beginners](https://medium.com/datadriveninvestor/aws-vpc-for-beginners-eee01e7083d1)

* [AWS DynamoDB Overview](https://medium.com/@lewisdgavin/aws-dynamodb-overview-184e53aedcd6)
