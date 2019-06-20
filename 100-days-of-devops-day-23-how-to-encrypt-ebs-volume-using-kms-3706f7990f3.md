
# 100 Days of DevOps — Day 23- How to encrypt EBS Volume using KMS

Welcome to Day 23 of 100 Days of DevOps, Let continue our journey with terraform and on Day22 I give you a brief introduction about KMS
[**100 Days of DevOps — Day 22-Introduction to Key Management System(KMS)**
*Welcome to Day 22 of 100 Days of DevOps, Let continue our journey with terraform and today we are going to create KMS…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-22-introduction-to-key-management-system-kms-4c73ff555169)

*KMS is integrated with a bunch of AWS Services, for the complete list check the link below*
[**How AWS Services use AWS KMS - AWS Key Management Service**
*Learn which AWS services use AWS Key Management Service (AWS KMS) and the technical details of each service…*docs.aws.amazon.com](https://docs.aws.amazon.com/kms/latest/developerguide/service-integration.html)

*How it's integrated with particular service is service specific, the responsibility of KMS is to generate Customer Master Key(CMK) and Data Keys and then handover Data Key to that Particular service at this point KMS responsibility ends*
> ***Encrypt EBS Volume using KMS Key***

    *AWS Console --> EC2 --> ELASTIC BLOCK STORE --> Volumes --> Create Volume*

![](https://cdn-images-1.medium.com/max/4120/1*0XCxKlHFnU5lPbH1n1EsEQ.png)

* *One important point to note here, KeyManager will tell you if this is managed by AWS(anything starts with aws is an aws managed key)or its customer Managed keys(key that is created by us)*

![](https://cdn-images-1.medium.com/max/4420/1*O1bhYQzF1BKeKSLmpf7SwA.png)

    ** Check the Encryption tab
    * Under Master Key(Select the KMS we just created from the drop down)*

*If you go back to the Volume, you will see something like this*

![](https://cdn-images-1.medium.com/max/5028/1*sYiwHfI8oNyk2fl26yLhSA.png)

*To summarize this how it works*

1. *When you create an encrypted EBS volume, Amazon EBS sends a [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html) request to AWS KMS, specifying the CMK that you chose for EBS volume encryption.*

1. *AWS KMS generates a new data key, encrypts it under the specified CMK, and then sends the encrypted data key to Amazon EBS to store with the volume metadata.*

1. *When you attach the encrypted volume to an EC2 instance, Amazon EC2 sends the encrypted data key to AWS KMS with a [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) request.*

1. *AWS KMS decrypts the encrypted data key and then sends the decrypted (plaintext) data key to Amazon EC2.*

1. *Amazon EC2 uses the plaintext data key in hypervisor memory to encrypt disk I/O to the EBS volume. The plaintext data key persists in memory as long as the EBS volume is attached to the EC2 instance.*

*NOTE:*

* *All the snapshots created from this volume are all encrypted.*

* *The plaintext data key persists in Hypervisor memory.*

### *Terraform Code*

<iframe src="https://medium.com/media/2ce43aa761c98ebfc53ae609c2a44194" frameborder=0></iframe>

* *Most of the EBS code, I already explained in the EBS section*
[**100 Days of DevOps — Day 17- Creating EC2 Instance using Terraform**
*Welcome to Day 17 of 100 Days of DevOps, Let continue our journey, so far I havr discussed terraform fundamentals and…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-17-creating-ec2-instance-using-terraform-c876a09d9d66)

    *# Some changes
    * encrypted: This is required to encrypt the disk
    * kms_key_id: The ARN for the KMS encryption key*

*NOTE: encrypted parameter is mandatory to set, else you will run into this error*

    **** aws_ebs_volume.my-test-kms-ebs: Error creating EC2 volume: InvalidParameterDependency: The parameter [KmsKeyId] requires the parameter Encrypted to be set.***

    ***status code: 400, request id: feeb7131-82ea-48d2-854e-84e4fbb7e5e1***

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
