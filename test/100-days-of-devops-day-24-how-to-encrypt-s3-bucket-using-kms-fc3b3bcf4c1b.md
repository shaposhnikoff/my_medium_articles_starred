
# 100 Days of DevOps — Day 24- How to encrypt S3 Bucket using KMS

Welcome to Day 24 of 100 Days of DevOps, Let continue our journey with terraform and on Day22 I give you a brief introduction about KMS, day 23 we learned how to encrypt EBS volume using KMS. Today I am going to demonstrate how to encrypt S3 bucket using KMS.
[**100 Days of DevOps — Day 23- How to encrypt EBS Volume using KMS**
*Welcome to Day 23 of 100 Days of DevOps, Let continue our journey with terraform and on Day22 I give you a brief…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-23-how-to-encrypt-ebs-volume-using-kms-3706f7990f3)

*There are two ways to use AWS KMS with AWS S3*

* *Server Side Encryption(I am only going to discuss this)*

* *Client Side Encryption*

***Server Side Encryption***

*We can protect data at rest in Amazon S3 using three different modes of server-side encryption: SSE-S3, SSE-C, or SSE-KMS*

* *SSE-S3 requires that Amazon S3 manage the data and master encryption keys.*

* *SSE-C requires that you manage the encryption key.*

* *SSE-KMS requires that AWS manage the data key but you manage the master key in AWS KMS.*
> ***Encrypt S3 bucket using SSE-KMS Key***

    *Go to S3 console --> [https://s3.console.aws.amazon.com/s3](https://s3.console.aws.amazon.com/s3) --> Particular bucket*

*Now when uploading any file to your bucket choose the KMS key*

![](https://cdn-images-1.medium.com/max/3460/1*ojPkePchgOOIEPfeyTXbeg.png)

***This is what happens behind the scene***

* *Amazon S3 requests a plaintext data key and a copy of the key encrypted under the specified CMK.*

* *AWS KMS creates a data key, encrypts it by using the master key, and sends both the plaintext data key and the encrypted data key to Amazon S3.*

* *Amazon S3 encrypts the data using the data key and removes the plaintext key from memory as soon as possible after use.*

* *Amazon S3 stores the encrypted data key as metadata with the encrypted data.*

***Now during decrypt operation***

* *Amazon S3 sends the encrypted data key to AWS KMS.*

* *AWS KMS decrypts the key by using the appropriate master key and sends the plaintext key back to Amazon S3.*

* *Amazon S3 decrypts the ciphertext and removes the plaintext data key from memory as soon as possible.*

***Terraform Code***

<iframe src="https://medium.com/media/2d6c25d286ea26296be12cb8d4230dd0" frameborder=0></iframe>

*GitHub Link*
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/tree/master/s3_kms)

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
