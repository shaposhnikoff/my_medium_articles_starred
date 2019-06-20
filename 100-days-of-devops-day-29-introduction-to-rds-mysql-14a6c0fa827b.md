
# 100 Days of DevOps — Day 29- Introduction to RDS — MySQL

Welcome to Day 29 of 100 Days of DevOps, Focus for today is RDS
> *What is AWS RDS?*

*Amazon Relational Database Service (Amazon RDS) is a web service that makes it easier to set up, operate, and scale a relational database in the cloud. It provides cost-efficient, resizable capacity for an industry-standard relational database and manages common database administration tasks.*

*To create a database, go to Database section(AWS Console) and click on RDS*

![](https://cdn-images-1.medium.com/max/2000/1*IponEkF1PLeFM9PzA9FEGQ.png)

*Click on Get Started Now*

![](https://cdn-images-1.medium.com/max/4340/1*19QxAwn-M2LKuw5yBQI95A.png)

*Then Select MySQL*

![](https://cdn-images-1.medium.com/max/3080/1*SM-5rw1FixQrwqpc4GLknw.png)

*On the next screen, choose Dev/Test -MySQL(or depend upon your requirement, as my use case is only for testing purpose)*

![](https://cdn-images-1.medium.com/max/3164/1*aSBZCapc461fLgRMVCSgTg.png)

*On the next screen provide all the info*

![](https://cdn-images-1.medium.com/max/3080/1*idk8K3QjnShqAnisdbb9yw.png)

![](https://cdn-images-1.medium.com/max/3040/1*u0bHxX0z7JRflKZ8HV6GpQ.png)

*As this is for testing Purpose*

* *DB instance class(db.t2.micro)*

* *Skip MultiAZ deployment for the time being*

* *Gave all the info like(DB instance identifier, Master username, Master password)*

*Fill all the details in the next screen*

![](https://cdn-images-1.medium.com/max/3080/1*FfdcPEBBtJolpvz0cUKozQ.png)

![](https://cdn-images-1.medium.com/max/3088/1*D3IHPk60-Lc0PDT2SAX-7A.png)

![](https://cdn-images-1.medium.com/max/3072/1*nZEXI7pDoJbvasaRHEIKUA.png)

![](https://cdn-images-1.medium.com/max/3064/1*tjF3y9hb0s_D5Aan8NZLow.png)

![](https://cdn-images-1.medium.com/max/3180/1*y7usgU_HPJAn5MwyB2infg.png)

*Mainly you need to fill*

* *Database name(Don’t confuse it DB instance identifier)*

* *Backup retention period(0 days, for the time being)*

*Then click on Launch DB instance*

*Wait for few mins 5–10min(or depend upon your instance type and size) and check Instance Status(It should be available)*

![](https://cdn-images-1.medium.com/max/4368/1*oxM1mT4QdiJ6pQmnh1Uv7w.png)

*Now lets try to create Read Replica out of this database*

![](https://cdn-images-1.medium.com/max/4416/1*8V7wW7ajNaEy8BYSoxVoQQ.png)

*Ohho no Create read replica option is not highlighted for me and the reason for that*

* *We don’t have a snapshot*

* *We don’t have an automated backups*

*Read replica is always created from a snapshot or the latest backup*

*Let’s take a snapshot of this database*

![](https://cdn-images-1.medium.com/max/3528/1*mKw4qo3DkVEFO2S-ojBOvQ.png)

*Once the snapshot creation is done, let’s try to convert this into multi-AZ. Go to Instance actions and click on Modify*

![](https://cdn-images-1.medium.com/max/4428/1*6D6259EawaEyuPZf0R6SLA.png)

![](https://cdn-images-1.medium.com/max/3364/1*Y3JwHW--XRJj8Ld5S9FFoQ.png)

![](https://cdn-images-1.medium.com/max/3236/1*zm6UBN_AKBo5MFOs626CNQ.png)

![](https://cdn-images-1.medium.com/max/3228/1*buAgrz5eEvNrrLdjwTy5gg.png)

![](https://cdn-images-1.medium.com/max/3384/1*nQfYOhlm7ciueUE2-VfSVw.png)

*These are the things you need to modify*

* *Multi-AZ set to Yes*

* *Under settings you need to enter the password again*

* *I am enabling backup and set it to 1 day*

* *On the final screen, you have the option*

*1: Apply during the next *scheduled maintenance* window*

*2: Apply immediately(This will cause a downtime)*

*To restore a database from the snapshot*

![](https://cdn-images-1.medium.com/max/5736/1*o2_zVMKDHPM65YnHOMUkyw.png)

*and then on the next screen, give DB Instance Identifier or any other setting you want to modify while restoring*

![](https://cdn-images-1.medium.com/max/3492/1*T5epSX4GY68j-EFdr7S3Bw.png)

*To Verify if Multi-AZ is enabled, Click on the particular DB*

![](https://cdn-images-1.medium.com/max/4284/1*bSICBnthWA6ScdmUdf8ilg.png)

*Now let’s try to create read-replica again, as you can see Create read replica tab is now enabled*

![](https://cdn-images-1.medium.com/max/4600/1*bB_LSNFTEnCnLehSMuBrCw.png)

*The Important thing to remember we can create read replica in any other region*

![](https://cdn-images-1.medium.com/max/3164/1*PJKjmhKiKt_Z8zox5HvjeA.png)

*Under the Settings tab, give it a unique name*

![](https://cdn-images-1.medium.com/max/2880/1*8V9oODhM8DqXf09Ij3kkQA.png)

**Terraform Code**

<iframe src="https://medium.com/media/a4803750e281ee3bc9f6ca3fa5720669" frameborder=0></iframe>

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
