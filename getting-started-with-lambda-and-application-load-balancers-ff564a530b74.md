
# Getting Started with Lambda and Application Load Balancers



I love Lambda for its versatility and flexibility. The list of [services which can invoke it](https://docs.aws.amazon.com/lambda/latest/dg/lambda-services.html) is growing all the time, making it able to take on the workloads you would traditionally require EC2, or more recently containers for. One of the most recent additions to this list is the Application Load Balancer.

When the announcement was made in November 2018, I was intrigued to find out more about the use cases of these two services working together. I’m going to admit, I struggled to see a solid real-world use case for it and found similar questions being asked as mine online. After some thinking and further research, the use cases started to become clearer. The options revolve around the ability to begin to replace components of a server-based application without the need to implement messy and complex solutions.

Before I go on, you need to understand how Application Load Balancers work vs traditional Load Balancers (ELB Classic). An ALB has two additional components:

* **Target Groups:** A target group is used to combine groups of servers together which perform like operations.

* **Listeners: **These receive connection requests for a particular port and protocol (eg. 80/HTTP or 443/HTTPS) and consist of a number of rules. Rules traditionally are path based and can route traffic to specific target groups.

In a real-world scenario, pretend we have an online ticket sales site which receives lots of traffic and needs to ensure customers who are purchasing tickets are not affected by those coming to the site to view available seats. We would create two target groups, group A and B. Using rules, traffic for /checkout can go to Group B and then everything else routed to Group A. The concept is relatively straightforward.

With that out of the way, here are some of my thoughts and scenarios of ways a Lambda function could be used behind an ALB:

* You have a traditional server-based application which also has a web API. This API is built into the overall monolithic application and you want to break it out to improve release processes and scalability.

* You have a traditional server-based application which renders some HTML pages. These pages are important but don’t necessarily need to be on servers or within the monolithic app and so you wish to break them out.

* You have a traditional server-based application with a number of static assets. You wish to store these on S3 to save on EBS costs but you are unable to easily re-architect the application.

Now you know the theory, let’s actually give this a try!

## Getting our Hands Dirty

I’m going to use the [Serverless Framework](https://serverless.com) and .NET to do this — it should be just as easy to understand for anyone who can read a CloudFormation or SAM template, with C# as the runtime. The steps are just as easily transposed into the Console! I’m also going to assume a public VPC has been created.

We’ll get started by creating a new service:

    sls create --template aws-csharp --path LambdaALB

I like to deploy components progressively when I’m working on a new PoC to see what works and what doesn’t, so the first components I’m going to create under the resources section of the serverless.yml file are:

* A Security Group which allows traffic on port 80

* An Application Load Balancer

* A Target Group (with no targets)

* A Listener

<iframe src="https://medium.com/media/b32ed1c24438613e5a82843dc5d58952" frameborder=0></iframe>

We now have a Load Balancer which doesn’t direct traffic anywhere. In the past, you would spin up some EC2 instances and assign them to the target group, but what I’m going to do next is create three Lambda functions for the scenarios mentioned earlier.

The definition for each function is as usual, however, instead of the traditional HTTP event, an ALB event will be used. A listener needs to be defined (referencing the one created earlier) along with a priority and path to respond to the requests on.

<iframe src="https://medium.com/media/fec167ed4a03dab541e617d63970d364" frameborder=0></iframe>

My [GitHub Repository](https://github.com/GavL89/LambdaALB/tree/master) has some slightly more detailed code in it, but here is an example of some very basic code to respond to a request from ALB. The key here is to reference the [ApplicationLoadBalancerEvents package](https://www.nuget.org/packages/Amazon.Lambda.ApplicationLoadBalancerEvents/) and return an ApplicationLoadBalancerResponse, otherwise, you’ll receive a 502 Gateway error from your ALB.

<iframe src="https://medium.com/media/5a49fb2737e1c5609aab078b40a28f7e" frameborder=0></iframe>

Now we have some basic code in place, let’s deploy what we’ve done so far and take a look at the result — success!

![](https://cdn-images-1.medium.com/max/5036/1*iA-Wu5Lc1PizVAt3TCeArQ.png)

I’m always intrigued about the event structure which invokes Lambda functions. This is especially useful if you’re using the path, method, query strings or any other information from the event to perform particular actions from within your Lambda function.

<iframe src="https://medium.com/media/bebf56beb1b611dc3d42c64c6701b9d5" frameborder=0></iframe>

OK — now we have should have three URLs, using my load balancer as an example (replace my URL with yours):

* **HTML Page:** *http://lambd-loadb-mzay8zovkhl7-1612334130.ap-southeast-2.elb.amazonaws.com/*

![](https://cdn-images-1.medium.com/max/5036/1*iA-Wu5Lc1PizVAt3TCeArQ.png)

* **API:** *http://lambd-loadb-mzay8zovkhl7-1612334130.ap-southeast-2.elb.amazonaws.com/api/*

![](https://cdn-images-1.medium.com/max/5036/1*ZgMnJsFmos2I152C1iuBBA.png)

* **S3 Assets:** *http://lambd-loadb-mzay8zovkhl7-1612334130.ap-southeast-2.elb.amazonaws.com/assets/200.jpg*

![](https://cdn-images-1.medium.com/max/5032/1*f7FN7EtGMQ6QPnr8K9nKIw.png)

I particularly like the last solution to offload static assets to S3. The best part about this solution is you do not have to change your application! The code, although rough and could do with some refinement, is relatively simple but clever. It takes the path from the request and retrieves the object from the S3 bucket with the same key. Just remember, this method exposes your entire bucket publically so don’t keep anything private in there!

<iframe src="https://medium.com/media/3356112b64df4970a4b842a3e0016dc6" frameborder=0></iframe>

## Conclusion

My main goal for this article was to experiment and find out more about why you would use Lambda behind an Application Load Balancer, as the use cases were not clear. Throughout the learning process, it became quite clear there are a number of use cases where this method can be useful to start using serverless technologies, without having to run multi-year projects to change large, complex architectures for a relatively simple change.

I’d be interested in hearing other use cases you may have come across, feel free to comment or message me on [Twitter](https://twitter.com/gavinlewis89) or [LinkedIn](https://www.linkedin.com/in/gavinlewis/)!

**Links: [**GitHub Repository](https://github.com/GavL89/LambdaALB/tree/master)
