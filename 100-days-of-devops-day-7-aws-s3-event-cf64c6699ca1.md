
# 100 Days of DevOps — Day 7(AWS S3 Event)

Welcome to Day 7 of 100 Days of DevOps, Let extend the journey of DevOps Monitoring and Alerting with S3 events.

### *Problem Statement*

* *Whenever anyone deletes any object in S3 bucket you will get the notification*

### *Solution*

*This can be achieved via in one of the three ways*

* *AWS Console*

* *Terraform*

*NOTE: This is not the complete list and there are more ways to achieve the same*
> *What is S3 Event?*

*The Amazon S3 notification feature enables you to receive notifications(SNS/SQS/Lambda) when certain events(mentioned below)happen in your bucket.*

![](https://cdn-images-1.medium.com/max/2000/1*s66gGTDmVZgExB0iNSofrA.png)

![](https://cdn-images-1.medium.com/max/3160/1*cg5pjr3WhjaeRnDQymQcnQ.png)

*Just to re-iterate the same thing S3 events work at the object level, so if something happens to the object, in this case, maybe PUT, POST, COPY or DELETE then the event is generated and that event will be delivered to the target(SNS, SQS or LAMBDA)*
> *To Configure S3 events*

* *First, go to the SNS topic [https://us-west-2.console.aws.amazon.com/sns/v2/home?region=us-west-2#/home](https://us-west-2.console.aws.amazon.com/sns/v2/home?region=us-west-2#/home)*

* *Click on Topics → Actions → Edit topic policy*

![](https://cdn-images-1.medium.com/max/2164/1*jHlb247gLhtUsVRBHtRmwQ.png)

* *Paste this json policy(We still need permission on SNS topic to allow S3 event system to deliver events to it)*

<iframe src="https://medium.com/media/458e8d615065297c3dda766d0921a073" frameborder=0></iframe>

* *Go to S3 console [https://s3.console.aws.amazon.com/s3/home?region=us-east-1](https://s3.console.aws.amazon.com/s3/home?region=us-east-1)*

* *Your bucket → Properties → Events*

![](https://cdn-images-1.medium.com/max/2000/1*kHeN6A2Knl8es3-1S2uFuA.png)

![](https://cdn-images-1.medium.com/max/2172/1*v23I5bUHksztC4vnqH8F3w.png)

    ** Give your event a name
    * Event type I want to get a notification is (All object delete events) i.e I want to recieve notification when object is deleted from this topic.
    * Send to SNS Topic
    * SNS(Choose the SNS topic you created earlier)*

*For more info about SNS*
[**100 Days of DevOps — Day 2 -Introduction to Simple Notification Service(SNS)**
*Welcome to Day 2 of 100 Days of DevOps, Let extend the journey of DevOps Monitoring and Alerting…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-2-introduction-to-simple-notification-service-sns-97137b2f1f1e)

* *Now if you try to delete an object from the bucket*

![](https://cdn-images-1.medium.com/max/4556/1*8_kFlRQJXPCPxyVICQTIYg.png)

***NOTE:** Delivery of these events happens in almost real-time with no cost involved.*

*Terraform Code*

<iframe src="https://medium.com/media/8d9371cf6a91bafb19e63bbde4869938" frameborder=0></iframe>

*GitHub link*
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/blob/master/s3eventstosns.tf)

*For more info about supported events type*
[**Configuring Amazon S3 Event Notifications - Amazon Simple Storage Service**
*Set up and configure notifications so that key events on buckets cause a message to be sent to an Amazon SNS topic.*docs.aws.amazon.com](https://docs.aws.amazon.com/AmazonS3/latest/dev/NotificationHowTo.html#notification-how-to-event-types-and-destinations)

***NOTE: **This is not a completely automated solution, you need to go to a particular topic and Create Subscription. For complete info*
[**100 Days of DevOps — Day 2 -Introduction to Simple Notification Service(SNS)**
*Welcome to Day 2 of 100 Days of DevOps, Let extend the journey of DevOps Monitoring and Alerting…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-2-introduction-to-simple-notification-service-sns-97137b2f1f1e)

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
