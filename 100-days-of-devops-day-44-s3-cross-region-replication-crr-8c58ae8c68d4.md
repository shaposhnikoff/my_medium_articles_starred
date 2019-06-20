
# 100 Days of DevOps — Day 44-S3 Cross Region Replication(CRR)

Welcome to Day 44 of 100 Days of DevOps, Focus for today is S3 Cross Region Replication
> *What is Cross-Region Replication*

*Cross-region replication (CRR) enables automatic, asynchronous copying of objects across buckets in different AWS Regions. Buckets configured for cross-region replication can be owned by the same AWS account or by different accounts.*
> *Features and Limitations*

* *It only replicates the object at the point of enabling replication, all the object before that can’t be replicated.*

* *Cross region replication by default only replicates un-encrypted objects or objects which encrypted using SSE-S3(Server-Side Encryption with Amazon S3-Managed Keys)*

* *SSE-C(Server-Side Encryption with Customer-Provided Keys) are not supported and SSE-KMS requires some extra configuration.*

* *By default ownership and ACL are replicated and maintained but we can always customize it.*

* *The storage class is maintained by default.*

* *Lifecycle events are not replicated*

*Create a source and destination bucket in two different regions under the same account*

* *Versioning must be enabled in both the bucket to configure Cross Region Replication*

* *Any object that resides in the bucket before versioning is enabled will not be replicated*
> *Step1: Create Source Bucket*

    *Go to AWS Console --> [https://console.aws.amazon.com/s3](https://console.aws.amazon.com/s3) --> Create bucket*

![](https://cdn-images-1.medium.com/max/4400/1*GWuxjrxPaQ8LOFDoomSIJA.png)

    ** Give your bucket some name
    * Choose Region as US East(N. Virginia)*

* *Once the bucket is created*
> *Step2: Enable versioning*

    *Click on the bucket --> Properties --> Versioning --> Enable versioning*

![](https://cdn-images-1.medium.com/max/4936/1*ePvlM0jzhNuhK4hJCCRcEQ.png)
> *Step3: Create a destination bucket*

![](https://cdn-images-1.medium.com/max/4392/1*KmcnQP7kbBwzQm_g5rDnHA.png)

    ** Everything will be same, except Bucket name will be my-destination-s3-bucket-to-test-crr
    * Region: US West(Oregon)
    * Enabled Versioning*
> *Step4: Enabled Cross Region Replication*

![](https://cdn-images-1.medium.com/max/4152/1*A-ftecwWctTp4rTrpAeyGQ.png)

    ** Go to your Source Bucket --> Management --> Replication --> Get started*

![](https://cdn-images-1.medium.com/max/2516/1*Dz2R8ZlfMb6pazyXallEBw.png)

    ** Select Entire bucket*

![](https://cdn-images-1.medium.com/max/2524/1*LLQSch1lNIzeOkQ2dd8HIg.png)

    ** Select the destination Bucket*

![](https://cdn-images-1.medium.com/max/2532/1*NSsuEb1xvSr7fICoXTEOiA.png)

    ** Select new IAM role
    * Give your Role name
    * Review your settings and save it*
> *Step5 : Test*

* *Go back to your Source S3 bucket(my-source-s3-bucket-to-test-crr) and try to upload some files*

![](https://cdn-images-1.medium.com/max/4820/1*PzQScnVsDLzwjiVyf4LaDw.png)

* *Wait for a few mins, you will see the same file replicated to the destination bucket*

![](https://cdn-images-1.medium.com/max/5544/1*9IwUWdsi1hBwDHDSqQ20kA.png)

* *Terraform code to automate the above setup*

<iframe src="https://medium.com/media/3c2a8e9bba708de479c94a39be910c49" frameborder=0></iframe>

* *GitHub Link*
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/tree/master/s3-crossregion-replication)

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
