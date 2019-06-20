
# 100 Days of DevOps — Day 11- Restricting S3 Bucket Access to Specific IP Addresses

Welcome to Day 11 of 100 Days of DevOps, Let continue our journey with IAM and let discuss one of the common topic/requirement I often encounter in our daily jobs where I need to restrict S3 access from specific IP address.

***Problem:** Restrict S3 bucket access(Get/Put Operation from specific IP)*

***Solution:** This can be done using the S3 bucket policies*

*S3 Bucket policies come under Resource Policies that control who has access to the specific resource.*

***Step1:***

    *Go to S3 console [https://s3.console.aws.amazon.com/s3/home?region=us-west-2](https://s3.console.aws.amazon.com/s3/home?region=us-west-2) → Specific Bucket → Permissions → Bucket Policy → Policy *

![](https://cdn-images-1.medium.com/max/5440/1*gSgEkTUFB7ecg5_buG6f7w.png)

*Step2: Fill all the details*

![](https://cdn-images-1.medium.com/max/4184/1*xdKk03HTLoNqnWZslx4Kww.png)

    ** Effect: Allow
    * Principal: *
    * AWS Service: Amazon S3
    * Action: Select GetObject and PutObject
    * Amazon Resource Name(ARN): <arn of your S3 bucket>/* **<--Don't forget to Put /* at the end**
    Add Conditions
    * Condition: IpAddress
    * Key: aws:SourceIp
    * Value: 192.168.0.2/24 (Specify your IP Address)*
[**S3 Bucket action doesn't apply to any resources**
*Thanks for contributing an answer to Stack Overflow! Please be sure to answer the question. Provide details and share…*stackoverflow.com](https://stackoverflow.com/questions/44228422/s3-bucket-action-doesnt-apply-to-any-resources)

*Final Policy will look like this*

<iframe src="https://medium.com/media/8f6b25fb33b5df79c96e6b6fba056338" frameborder=0></iframe>

* ***Step3:** Copy paste this policy to the Bucket Policy Editor and save it*

* ***Step4:** Test it*

## *AWS CLI*

    ** Create a json file bucketpolicy.json
    * aws s3api put-bucket-policy --bucket my-test-bucket --policy file://bucketpolicy.json*

## ***Terraform***

<iframe src="https://medium.com/media/0dbda24604a9810542574be5a27621ec" frameborder=0></iframe>

*GitHub link*
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/blob/master/s3bucketpolicy.tf)

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
