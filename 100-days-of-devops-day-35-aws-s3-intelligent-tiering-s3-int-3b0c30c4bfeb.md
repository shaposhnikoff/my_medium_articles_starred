
# 100 Days of DevOps — Day 35-AWS S3 Intelligent-Tiering (S3 INT)

Welcome to Day 35 of 100 Days of DevOps, Focus for today is AWS S3 Intelligent-Tiering (S3 INT)
> *What is AWS S3 Intelligent-Tiering (S3 INT)?*

*S3 Intelligent Tiering is the new storage class, AWS launched during AWS re:invent 2018.*

![](https://cdn-images-1.medium.com/max/5640/1*PZryXkcXoVj99unK-XNqUA.png)

* *Now we can choose Intelligent-Tiering while uploading any object to S3 bucket*

![](https://cdn-images-1.medium.com/max/2000/1*GCh4tJzogvL4Vh3bk6pJJQ.png)

* *Use Machine Learning under the hood(monitoring access pattern over our data) to move objects that are not been accessed from 30 days.*

![](https://cdn-images-1.medium.com/max/4888/1*V6TosZNp6ctJVEg2ov0VZQ.png)

* *There are no retrieval fees in moving the object back to S3 Standard tier in case we access the object in an infrequent tier.*

* *Comes with additional cost.*

### What is the use case or problem this is going to solve?

*Currently, if we want to move data from one storage class to another, we have two options*

* ***LifeCycle Policies:** We can transition from Standard to Standard-IA, One Zone-IA, or Glacier based on their creation date*

![](https://cdn-images-1.medium.com/max/2756/1*Pj80U08j0b_oXVdziNAk4A.png)

* *Now we have the new choice of intelligent tiering(besides choosing it **during upload**)*

![](https://cdn-images-1.medium.com/max/2768/1*XdPSeUw8S4esditNSC7jgQ.png)

* ***Storage Class Analytics:** Other options we have is by using Storage Class Analytics. This will help us to find out if the data stored in S3 Standard class is suited for S3 Infrequent access.*

![](https://cdn-images-1.medium.com/max/3072/1*dQi9OGz0eLbWvMB1PbqgTQ.png)

*Reference: [https://docs.aws.amazon.com/AmazonS3/latest/dev/analytics-storage-class.html](https://docs.aws.amazon.com/AmazonS3/latest/dev/analytics-storage-class.html)*

## Issues

* *Access pattern of data is irregular*

* *You don’t have time to use Storage Class Analytics*

## S3 Intelligent-Tiering

* *If you are facing the above two problems then S3 Intelligent Tiering is here for your rescue.*

* *It incorporates two access tier(Standard and Standard Infrequent Access)*

* *As mentioned above, it monitors access patterns and moves objects that have not been accessed for 30 consecutive days to the infrequent access tier.*

* *If the data is accessed later, it is automatically moved back to the frequent access tier with no additional cost.*

* *It can be specified while uploading objects OR during LifeCycle Policy.*

* *Comes with additional cost*

* *There are no additional fees while retrieving data back from S3 Infrequent to S3 Standard.*

* *Supports all S3 features cross-region replication, encryption, object tagging, and inventory.*

***Some points to consider before using S3 Intelligent Tiering Class***

* *An object smaller then 128KB will never be transitioned to S3 Infrequent access and will be billed usually.*

* *Not fit for an object which lives less than 30 days, all objects will be billed with a minimum of 30 days.*

***Final Word***

* *S3 Intelligent Tiering comes with an additional cost, if that is less than what you are paying right now, then this class is for you*

* *It looks promising, as AWS is using Machine Learning Model under the hood, to fed trillions of objects and millions of requests to predict future access patterns for each object. The results were then used to inform storage of your S3 objects in the most cost-effective way possible*

![](https://cdn-images-1.medium.com/max/3928/1*UArR_OadlrW1ZYNzHTr0yA.png)

***Reference***
[**Cloud Storage Pricing | S3 Pricing by Region | Amazon Simple Storage Service**
*Detailed information on Free, Storage, Requests and GovCloud pricing options for all classes of S3 cloud storage*aws.amazon.com](https://aws.amazon.com/s3/pricing/)
[**Object Storage Classes — Amazon S3**
*The S3 Intelligent-Tiering storage class is designed to optimize costs by automatically moving data to the most…*aws.amazon.com](https://aws.amazon.com/s3/storage-classes/#Unknown_or_changing_access)

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
