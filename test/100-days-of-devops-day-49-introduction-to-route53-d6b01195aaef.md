
# 100 Days of DevOps — Day 49-Introduction to Route53

Welcome to Day 49 of 100 Days of DevOps, Focus for today is Route53
> *What is AWS Route53?*

*Amazon Route 53 is a highly available and scalable Domain Name System (DNS) web service. You can use Route 53 to perform three main functions in any combination:*

* *Domain Registration*

* *DNS Routing*

* *Health Checking*
> *Key DNS Terms*

* ***A Record***

*A record is used to translate human-friendly domain names such as “www.example.com" into IP-addresses such as 192.168.0.1 (machine friendly numbers).*

* ***CNAME Record***

*A Canonical Name record (abbreviated as CNAME record) is a type of resource record in the Domain Name System (DNS) which maps one domain name (an alias) to another (the Canonical Name.)*

* ***NameServer Record***

*NS-records identify the DNS servers responsible (authoritative) for a zone.*

*Amazon Route 53 automatically creates a name server (NS) record that has the same name as your hosted zone. It lists the **four name servers** that are the authoritative name servers for your hosted zone. Do not add, change, or delete name servers in this record.*

    *ns-2048.awsdns-64.com
    ns-2049.awsdns-65.net
    ns-2050.awsdns-66.org
    ns-2051.awsdns-67.co.uk*

* ***SOA Record***

*A Start of Authority **record** (abbreviated as **SOA record**) is a type of resource **record** in the **Domain Name** System (**DNS**) containing administrative information about the zone, especially regarding zone transfers*

<iframe src="https://medium.com/media/7e59ef6d9658c711bcac390a3e399c3b" frameborder=0></iframe>
> ***AWS Specific DNS Terms***

* ***Alias Record***

*Amazon Route 53 alias records provide a Route 53–specific extension to DNS functionality. Alias records let you route traffic to selected AWS resources, such as CloudFront distributions and Amazon S3 buckets. They also let you route traffic from one record in a hosted zone to another record.*

*Unlike a CNAME record, you can create an alias record at the top node of a DNS namespace, also known as the **zone apex.** For example, if you register the DNS name example.com, the zone apex is example.com. You can’t create a **CNAME record** for example.com, but you can create an alias record for example.com that routes traffic to [www.example.com.](http://www.example.com.)*

* ***AWS Route53 Health Check***

*Amazon Route 53 health checks monitor the health and performance of your web applications, web servers, and other resources. Each health check that you create can monitor one of the following:*

* *The health of a specified resource, such as a web server*

* *The status of other health checks*

* *The status of an Amazon CloudWatch alarm*
> Step1: Create a hosted zone

    *Go to [https://console.aws.amazon.com/route53](https://console.aws.amazon.com/route53) --> Hosted zones --> Create Hosted Zone*

![](https://cdn-images-1.medium.com/max/5736/1*hZBeNOKB37OWw3owGJ-6GA.png)

![](https://cdn-images-1.medium.com/max/2000/1*3FnmwKfcjnBBsIwd6ihaFw.png)

    ** Domain Name: You must need to purchase this domain either from your Domain Registrar or you can purchase from Amazon
    * Comment: Add some comment
    * Type: Public Hosted Zone(if purchased by a domain registrar) OR you can set Private Hosted Zone for Amazon VPC*

* *The moment you create a hosted zone four NS and SOA record will be created for you.*

![](https://cdn-images-1.medium.com/max/3372/1*KsQs5HzRbRv5ENZ1jjHAbw.png)

*NOTE: Please don’t change or alter these record.*
> Step2: Create A record

![](https://cdn-images-1.medium.com/max/3392/1*vkv5paEkvzCTBihuKLmX_Q.png)

![](https://cdn-images-1.medium.com/max/2000/1*18gkRtxkCo_5NqH1SkdsbQ.png)

    ** Name : www
    * Type: A-IPv4 address
    * Alias: No
    * TTL: Select +1m(60second)
    * Value: Public IP of your EC2 instance
    * Routing Policy: Simple*

* *You will see the record like this*

![](https://cdn-images-1.medium.com/max/3380/1*uiGUkkWhZ6IvendMYHsWaQ.png)

*Terraform Code*

<iframe src="https://medium.com/media/d137eaf593586410b7b88c62ce2dd5f7" frameborder=0></iframe>

*Looking forward from you guys to join this journey and spend a minimum an hour every day for the next 100 days on DevOps work and post your progress using any of the below medium.*

* *Twitter: [@100daysofdevops](http://twitter.com/100daysofdevops) OR @[lakhera2015](https://twitter.com/lakhera2015)*

* *Facebook: [https://www.facebook.com/groups/795382630808645/](https://www.facebook.com/groups/795382630808645/)*

* *Medium: [https://medium.com/@devopslearning](https://medium.com/@devopslearning)*

* *Slack: [https://devops-myworld.slack.com/messages/CF41EFG49/](https://devops-myworld.slack.com/messages/CF41EFG49/)*

* *GitHub Link:[https://github.com/100daysofdevops](https://github.com/100daysofdevops)*

***Reference***
[**100 Days of DevOps — Day 0**
*D-day is just one day away and finally, this is a continuation of the post(I posted a month earlier)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-0-4f2c9750542d)
[**100 Days of DevOps**
*Motivation*medium.com](https://medium.com/@devopslearning/100-days-of-devops-81faf13bf772)
