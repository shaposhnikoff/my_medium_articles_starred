
# 100 Days of DevOps -Day 3(Introduction to CloudTrail)

Welcome to Day 3of 100 Days of DevOps, Let extend the journey of DevOps Monitoring and Alerting
[**100 Days of DevOps — Day 1(Introduction to CloudWatch Metrics)**
*Welcome to Day 1 of 100 Days of DevOps, I would like to start this journey with one of the most critical concepts in…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-1-introduction-to-cloudwatch-metrics-b04be36307a8)
[**100 Days of DevOps — Day 2 -Introduction to Simple Notification Service(SNS)**
*Welcome to Day 2 of 100 Days of DevOps, Let extend the journey of DevOps Monitoring and Alerting…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-2-introduction-to-simple-notification-service-sns-97137b2f1f1e)

*with CloudTrail.*

***Problem:** Logs all the API calls in an AWS account(including AWS Console, CLI and API/SDK calls)*

### Solution

*This can be achieved via in one of the three ways*

* *AWS Console*

* *AWS CLI*

* *Terraform*

*NOTE: This is not the complete list and there are more ways to achieve the same*
> *What Is AWS CloudTrail?*

*AWS CloudTrail is an AWS service that helps you enable governance, compliance, and operational and risk auditing of your AWS account. Actions taken by a user, role, or an AWS service are recorded as events in CloudTrail. Events include actions taken in the AWS Management Console, AWS Command Line Interface, and AWS SDKs and APIs.*

![](https://cdn-images-1.medium.com/max/2240/1*HMPrcWW80mljWUQKMpWgEQ.jpeg)

* *It’s enabled when the account is created*

* *When activity occurs in your AWS account, that activity is recorded in a CloudTrail event.*

* *Entries can be viewed in **Event History**(**for 90 days**)*

* *Event logs can be aggregated across accounts.*

***NOTE***

* *Historically CloudTrail was not enabled by default*

* *It won’t logs events like **SSH/RDP** only API call.*

![*CloudTrail Dashboard(Most Recent Events)*](https://cdn-images-1.medium.com/max/2000/1*lL1KjsMCnFWD58TuMtE-PA.png)**CloudTrail Dashboard(Most Recent Events)**

*Reference: [https://aws.amazon.com/blogs/aws/new-aws-api-activity-lookup-in-cloudtrail/](https://aws.amazon.com/blogs/aws/new-aws-api-activity-lookup-in-cloudtrail/)*

*For an ongoing record of activity and events in your AWS account, create a **trail**. It allows us to send logs to S3 bucket and can be single or multi-region.*
> *To create a trail*

    *Go to AWS Console --> Management & Governance --> CloudTrail --> Trails --> Create trail*

![](https://cdn-images-1.medium.com/max/4408/1*YN4N25JXG_aZkL7jMuNyUQ.png)

![](https://cdn-images-1.medium.com/max/4500/1*Mh39mSAqYvYcVIrvswETww.png)

    ** **Trail name:** Give your trail name
    * **Apply trail to all regions:** You have an option to choose all regions or specific region.
    * **Read/Write events:** You have the option to filter the events
    * **Data events:** Data events provide insights into the resource operations performed on or within a resource
    **S3:** You can record S3 object-level API activity (for example, GetObject and  PutObject) for individual buckets, or for all current and future buckets  in your AWS account
    **Lambda:**You can record Invoke API operations for individual functions, or for all current and future functions in your AWS account.
    * **Storage Locations:** Where to store the logs, we can create new bucket or use existing bucket
    * **Log file prefix:** We have the option to provide prefix, this will make it easier to browse log file
    * **Encrypt log file with SSE-KMS:** **Default SSE-S3 Server side encryption(AES-256)** or we can use KMS
    * **Enable log file validation:** To determine whether a log file was modified, deleted, or unchanged after CloudTrail delivered it, you can use CloudTrail log file integrity validation
    * **Send SNS notification for every log file delivery:**SNS notification of log file delivery allow us to take action immediately*

***NOTE**:*

* *There is always a delay between when the event occurs vs displayed on CloudTrail dashboard. On top of that, there is an additional delay when that log will be transferred to S3 bucket.*

* ***Delivered every 5(active) minutes with up to 15-minute delay***

* *All CloudEvents are JSON structure you will see something like this when you try to view any event*

![](https://cdn-images-1.medium.com/max/5520/1*Q0EZ-A_TgIZOtGMNCW4YVg.png)

![](https://cdn-images-1.medium.com/max/2704/1*F1H3syKyRNMLUR4rr4OANA.png)
> ***Validating CloudTrail Log File Integrity***

* *To determine whether a log file was modified, deleted, or unchanged after CloudTrail delivered it, you can use CloudTrail log file integrity validation.*

* *This feature is built using industry-standard algorithms: SHA-256 for hashing and SHA-256 with RSA for digital signing.*

* *This makes it computationally infeasible to modify, delete or forge CloudTrail log files without detection.*

* *You can use the AWS CLI to validate the files in the location where CloudTrail delivered them*

![*CloudTrail-Digest Bucket*](https://cdn-images-1.medium.com/max/5556/1*dmac8w21hbdt_QHbdqfmPw.png)**CloudTrail-Digest Bucket**

* *Validation of logs can only be performed at the command line*

    *$ aws cloudtrail validate-logs --trail-arn arn:aws:cloudtrail:us-east-1:XXXXXXX:trail/mytestcloudtrail --start-time  2018-12-27T00:09:00Z  --end-time 2018-12-27T00:10:00Z  --verbose*

    *Validating log files for trail arn:aws:cloudtrail:us-east-1:XXXXXXX:trail/mytestcloudtrail between 2018-12-27T00:09:00Z and 2018-12-27T00:10:00Z*

    *Results requested for 2018-12-27T00:09:00Z to 2018-12-27T00:10:00Z*

    *No digests found*

* *We can also configure a CloudTrail to send copies of logs to CloudWatch Logs(a central location which aggregates logs)*

* *Go back to the trail we have just configured and click on Configure*

![](https://cdn-images-1.medium.com/max/4280/1*MZA1rFi0Mr32zHjgHncjRQ.png)

![](https://cdn-images-1.medium.com/max/4224/1*bZc9JCNdWSmijUq9o3VRpA.png)

![](https://cdn-images-1.medium.com/max/5680/1*Uj3X_dX9S47HuzerK4FCyA.png)

    *In order to successfully deliver CloudTrail events to your CloudWatch  Logs log group, CloudTrail will assume the role you are creating or  specifying. Assuming the role grants CloudTrail permissions to two  CloudWatch Logs API calls:*

    *1. CreateLogStream: Create a CloudWatch Logs log stream in the CloudWatch Logs log group you specify
     2. PutLogEvents: Deliver CloudTrail events to the CloudWatch Logs log stream*

* *Under CloudWatch logs, you will now see the newly created Log Groups*

![](https://cdn-images-1.medium.com/max/3200/1*KHZRHwpVeE2xtFBcLozn4A.png)

### *AWS CLI*

* ***Create Trail(Single Region)***

    *$ aws cloudtrail create-trail --name my-test-cloudtrail --s3-bucket-name mytests3bucketforcloudtrail*

    *{*

    *"IncludeGlobalServiceEvents": true,*

    *"IsOrganizationTrail": false,*

    *"Name": "my-test-cloudtrail",*

    *"TrailARN": "arn:aws:cloudtrail:us-west-2:XXXXXXXXX:trail/my-test-cloudtrail",*

    *"LogFileValidationEnabled": false,*

    *"IsMultiRegionTrail": false,*

    *"S3BucketName": "mytests3bucketforcloudtrail"*

    *}*

* ***Create Trail(That applies to multi-region)***

    *$ aws cloudtrail create-trail --name my-test-cloudtrail-multiregion --s3-bucket-name mytests3bucketforcloudtrail --is-multi-region-trail*

    *{*

    *"IncludeGlobalServiceEvents": true,*

    *"IsOrganizationTrail": false,*

    *"Name": "my-test-cloudtrail-multiregion",*

    *"TrailARN": "arn:aws:cloudtrail:us-west-2:XXXXXXXXX:trail/my-test-cloudtrail-multiregion",*

    *"LogFileValidationEnabled": false,*

    *"IsMultiRegionTrail": true,*

    *"S3BucketName": "mytests3bucketforcloudtrail"*

    *}*

* ***To get the status/list all the trails***

    *$ aws cloudtrail describe-trails*

    *{*

    *"trailList": [*

    *{*

    *"IncludeGlobalServiceEvents": true,*

    *"IsOrganizationTrail": false,*

    *"Name": "my-test-cloudtrail",*

    *"TrailARN": "arn:aws:cloudtrail:us-west-2:XXXXXXXXXX:trail/my-test-cloudtrail",*

    *"LogFileValidationEnabled": false,*

    *"IsMultiRegionTrail": false,*

    *"HasCustomEventSelectors": false,*

    *"S3BucketName": "mytests3bucketforcloudtrail",*

    *"HomeRegion": "us-west-2"*

    *},*

    *{*

    *"IncludeGlobalServiceEvents": true,*

    *"IsOrganizationTrail": false,*

    *"Name": "my-test-cloudtrail-multiregion",*

    *"TrailARN": "arn:aws:cloudtrail:us-west-2:XXXXXXXXXX:trail/my-test-cloudtrail-multiregion",*

    *"LogFileValidationEnabled": false,*

    *"IsMultiRegionTrail": true,*

    *"HasCustomEventSelectors": false,*

    *"S3BucketName": "mytests3bucketforcloudtrail",*

    *"HomeRegion": "us-west-2"*

    *}*

* ***Start logging for the trail***

    *$ aws cloudtrail start-logging --name my-test-cloudtrail*

*NOTE: When we create the trail using CloudTrail console, logging is turned on automatically*

* ***To verify if logging is enabled***

    *$ aws cloudtrail get-trail-status --name my-test-cloudtrail*

    *{*

    *"LatestDeliveryTime": 1550014519.927,*

    *"LatestDeliveryAttemptTime": "2019-02-12T23:35:19Z",*

    *"LatestNotificationAttemptSucceeded": "",*

    *"LatestDeliveryAttemptSucceeded": "2019-02-12T23:35:19Z",*

    ***"IsLogging": true,***

    *"TimeLoggingStarted": "2019-02-12T23:34:58Z",*

    *"StartLoggingTime": 1550014498.331,*

    *"LatestNotificationAttemptTime": "",*

    *"TimeLoggingStopped": ""*

    *}*

* ***To enable log file validation***

    *$ aws cloudtrail create-trail --name my-test-cloudtrail-multiregion-logging --s3-bucket-name mytests3bucketforcloudtrail --is-multi-region-trail --enable-log-file-validation*

    *{*

    *"IncludeGlobalServiceEvents": true,*

    *"IsOrganizationTrail": false,*

    *"Name": "my-test-cloudtrail-multiregion-logging",*

    *"TrailARN": "arn:aws:cloudtrail:us-west-2:349934551430:trail/my-test-cloudtrail-multiregion-logging",*

    *"LogFileValidationEnabled": true,*

    *"IsMultiRegionTrail": true,*

    *"S3BucketName": "mytests3bucketforcloudtrail"*

    *}*

* ***To delete a particular trail***

    *$ aws cloudtrail delete-trail --name my-test-cloudtrail-multiregion-logging*

<iframe src="https://medium.com/media/18465a67bbb1c877adec431cf2ee0172" frameborder=0></iframe>

***GitHub link***
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/blob/master/cloudtrail_aws_cli)

***Terraform***

<iframe src="https://medium.com/media/5409deb7eaed098dd700557ae64f76f9" frameborder=0></iframe>

*GitHub Link*
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/blob/master/cloudtrail.tf)

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
