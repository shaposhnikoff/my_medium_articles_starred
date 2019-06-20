
# 100 Days of DevOps — Day 14- How to automate the process of EBS Snapshot Creation

Welcome to Day 14 of 100 Days of DevOps, Let continue our journey and focus of this week is a daily DevOps task, to run the business and to save money. We all understand the importance of backups and snapshot is the handy way to create a point in time backups.

*Problem: How to take a snapshot of our EBS Volumes everyday*

*Solution: There are multiple ways to achieve this, I am just focussing on two*

* *CloudWatch Events*

* *EBS LifeCycle Manager*
> *What is EBS Snapshots?*

* *Snapshots are a point in time backups of EBS Volumes that are stored in S3.*

* *They are incremental in nature i.e first snapshot is the complete backup of your Volume but all the subsequent snapshot is just an incremental change.*

* *We can use snapshots to create an AMI*

* *Mostly used for a backup purpose to fully restore your EBS Volumes.*

* *The recommendation is to frequently take snapshots depending upon your Recovery Time Objective(RTO)*

* *For data consistency, it’s good to stop any write operation before performing snapshots.*

## *CloudWatch Events*

    *Go to AWS Console → Management Tools → CloudWatch → Events → Create rule*

[*https://us-west-2.console.aws.amazon.com/cloudwatch](https://us-west-2.console.aws.amazon.com/cloudwatch)*

![](https://cdn-images-1.medium.com/max/5128/1*XRpiS4HR3BwI6cWeG40ong.png)

    ** Under Event Source, Choose Schedule 
    * Choose the Schedule based on your requirement(for eg: I want this Process to be triggered 1.25 am everyday
    * In the Targets section, Choose EC2 CreateSnapshot API call and give your Volume ID
    * For CloudWatch Event to interact with EBS Volume it need an IAM role, Choose any existing one or create new one*

***NOTE: All scheduled events use UTC time zone, so might need to change based on your timezone***

* *On the next screen give your Snapshot some name and Description*

* *Now go to the EC2 console [https://us-west-2.console.aws.amazon.com/ec2/](https://us-west-2.console.aws.amazon.com/ec2/)*

* *Under ELASTIC BLOCK STORE, click on Snapshots*

![](https://cdn-images-1.medium.com/max/2000/1*DPWszxs-5QIcpicGxMWLIQ.png)

* *Based on the Schedule we choose, we will see something like this*

![](https://cdn-images-1.medium.com/max/4976/1*gFVYDfZDkEZqkUF5Vl2pWQ.png)

* *One of the major limitations, I see in this approach, is I don't see any option to delete snapshot after n number of days(eg: After 30 days)*

## ***Data Life Cycle Manager***

* *This is probably the easiest way, where using single service we can Create and Manage(deleting it after ’n’ number of days) EBS snapshots*

    *Go to AWS → Compute → EC2 → Elastic Block Store → Lifecycle Manager → Create Snapshot Lifecycle Policy*

![](https://cdn-images-1.medium.com/max/3580/1*I-EZ17bxRpT8rsL48-vFQQ.png)

![](https://cdn-images-1.medium.com/max/4024/1*VMz3P8uL5_8I0Tf-q1DFjw.png)

    *Description: Add a description to your policy to help you identify what it is intended for.
    Target volumes with tags: Add the Volume 
    Create snapshots every: 12 or 24 hour
    Snapshot creation start time: 5:00(Timing is in UTC)
    Retention rule: 30(Depending upon your requirement)
    Keep all the other values as default*

***NOTE: Snapshot creation start time** **hh**:**mm** **UTC** — The time of day when policy runs are scheduled to start. **The policy runs start within an hour after the scheduled time***

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
