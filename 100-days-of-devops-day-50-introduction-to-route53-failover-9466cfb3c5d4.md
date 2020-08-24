
# 100 Days of DevOps — Day 50-Introduction to Route53 Failover

Welcome to Day 50 of 100 Days of DevOps, Focus for today is Route53 Failover

*On Day 49, I talked about Route53, Let extend that concept further and talk about Failover Routing Policy.*
[**100 Days of DevOps — Day 49-Introduction to Route53**
*Welcome to Day 49 of 100 Days of DevOps, Focus for today is Route53*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-49-introduction-to-route53-d6b01195aaef)

***Scenario:** This is the common use case everyone faced, what will happen if my Primary site goes down, I need to have some sort of failover solution(eg: in this case S3) so that I will not lose my customer.*
> Step1: Add Route53 Health Check failover

* *This time click on Health checks → Create health check*

![](https://cdn-images-1.medium.com/max/5688/1*jS3agtXd1IRlQVWcyS669g.png)

![](https://cdn-images-1.medium.com/max/5436/1*KtIfdziSWuSPpvsUqll-Cw.png)

    ** Name: myhealthcheck
    * What to monitor: Endpoint
    * Specify endpoint by: IP address
    * Protocol: HTTP
    * IP Address: IP Address of your EC2 instance
    * Port: 80
    * Click Next
    * Create Alarm: No*

![](https://cdn-images-1.medium.com/max/4940/1*jjqYPNCteij19kq5unBNLg.png)

*NOTE: Please wait for a few mins till you see the status change from unknown to healthy*
> Step2: Configure DNS Failover to an Amazon S3 static website

* *On Day 49, I set up the A record and use the default Routing as Simple, its time to change the Routing Policy*

![](https://cdn-images-1.medium.com/max/2000/1*DQiX3O09HgQcd48AMjof6g.png)

    ** Change the Routing Policy as Failover
    * Failover Record Type: Primary
    * Set ID: Primary
    * Associate with Health Check: Yes
    From the drop down, choose the health check we created in Step1*
> Step3: Create an S3 bucket with the same name as your domain name

    ** Go to S3  Console [https://s3.console.aws.amazon.com/s3](https://s3.console.aws.amazon.com/s3) --> Create bucket*

![](https://cdn-images-1.medium.com/max/4416/1*xzuOBvdDfz4fj46GYlD2pQ.png)

* *Once the bucket is created, go the bucket properties*

![](https://cdn-images-1.medium.com/max/5048/1*0Mst-Do7y1CvsJNNSs2SjA.png)

    ** Choose Use this bucket to host a website
    * Index document: Create simple index.html file
    * Error document: error.html*

* *Sample Example of index.html file*

<iframe src="https://medium.com/media/3a8f7c0879feb89e42109286f31e6cf0" frameborder=0></iframe>

* *Copy the endpoint created in this step*

* *Go back to Route53 → Create Record Set*

![](https://cdn-images-1.medium.com/max/2000/1*5fvWsmdoscZ2vXOBlsn2OQ.png)

    ** Name: www
    * Type: A IPv4 Address
    * Alias: Yes(This time choose Alias to Yes)
    * Alias Target: Choose the S3 bucket
    * Routing Policy: Failover
    * Failover Record Type: Secondary
    * Set ID: Secondary
    * Evaluate Target Health: No
    * Associate with Health Check: No*

* *After a few mins, you will see these two records in your hosted zone*

![](https://cdn-images-1.medium.com/max/3352/1*sy4_wCZbpX3P7NGgGxTm2Q.png)
> Step4: Initiate a failover

* *Go to EC2 console and stop the instance*

* *Go back to your Route53 Management console and click on health check(it will take at least 2 min to get it reflected here and status change to unhealthy)*

* *After few min, browse your url again and you will see your site is back and served via S3.*

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
