
# 100 Days of DevOps — Day 10- Restricting User to Launch only T2 Instance

Welcome to Day 10 of 100 Days of DevOps, Let continue our journey with IAM and let discuss one of the common topic/requirement I often encounter in Dev environment where company want to save money(in some conditions) and they want to restrict developers to only launch t2 instance series(Which is cheaper then another instance type and t2.micro comes under free tier)

***Problem:** I want to limit user in my Dev Account to only launch instance type which is t2.**

***Solution:** This is possible by using IAM managed policy*

***Step1:** Go to IAM console [https://console.aws.amazon.com/iam/home?region=us-west-2#/home](https://console.aws.amazon.com/iam/home?region=us-west-2#/home) → Policies → Create policy*

![](https://cdn-images-1.medium.com/max/4044/1*X0uwECVewZrOnf3CoUL3Pg.png)

***Step2:** Under Service, Search for ec2*

![](https://cdn-images-1.medium.com/max/3776/1*sHNnO9me2xTUzgk13yiscg.png)

***Step3:** Under Action, Select List and Read(User can’t do much damage with just list and read)*

![](https://cdn-images-1.medium.com/max/3760/1*vcLsn46t4qrdhSLFeQ_b9A.png)

*and under the Filter actions search for run and select **RunInstances**(I need to give a user permission to launch instances)*

![](https://cdn-images-1.medium.com/max/3768/1*-YAxCzY20vDUIa2jccusXg.png)

***Step4:** Resources(EC2 actually validates a bunch of resources), we can put a fine grain control but for the time being I am selecting any*

![](https://cdn-images-1.medium.com/max/3708/1*sgS0EIDE48f6BvgG14dlRw.png)

***Step5: **Request Conditions: This is where we are going to specify that the user can only launch t2 instance type*

* *Add condition*

![](https://cdn-images-1.medium.com/max/2000/1*aL-tuOi096Wb7y0jYPMIZw.png)

    ** Condition key: Select ec2:InstanceType
    * Qualifier: Default
    * Operator: StringLike(As I have star in the Value)
    * Value: t2.**

***Step6:** In the next screen, give your policy some name*

![](https://cdn-images-1.medium.com/max/3996/1*AVIuvQUu5_R_xKYP7AkKaw.png)

*Final policy look like this*

<iframe src="https://medium.com/media/d36c9183530eb3e571193092aa8baddd" frameborder=0></iframe>

*GitHub link*
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/blob/master/ec2-t2-restrict.json)

*Step7: Attach this policy to the user you want to test*

* *Go to the particular user*

![](https://cdn-images-1.medium.com/max/5028/1*wDkGxOeBmVZCAq14CXTFTQ.png)

* *Click on Add permissions*

![](https://cdn-images-1.medium.com/max/5300/1*aj8s9SZVlE7w9o-PPpgFfQ.png)

***Step8:** Logout and logged in as that particular user and try to launch instance type as t2.micro*

*NOTE: If you are getting this error*

![](https://cdn-images-1.medium.com/max/2212/1*3BFAOK-ZoPi6JzUB7mddBQ.png)

* *You might need to add one more policy to give your user permission to create keys and attach it to the user.*

![](https://cdn-images-1.medium.com/max/5036/1*uxaChJo5VMeAr9p7EgtpLw.png)

*OR Security Group error*

![](https://cdn-images-1.medium.com/max/5744/1*C4eH3w8rdjkLkpd1RbJK_A.png)

![](https://cdn-images-1.medium.com/max/4876/1*V5-SvnMBDMqmBrXIg1bzbQ.png)

* *After adding all these policies you will see something like this*

![](https://cdn-images-1.medium.com/max/5760/1*7q18-oDnhT0Pwl8aHcxEZg.png)

* *Now let me try to launch some other instance type*

![](https://cdn-images-1.medium.com/max/5736/1*v2USpTfGhlLbWmjTE_a80Q.png)

* *You will see a message something like this*

![](https://cdn-images-1.medium.com/max/5736/1*v2USpTfGhlLbWmjTE_a80Q.png)

* *If you want to decode this message, you will see an explicit deny in trying to launch a specific resource.*

    *$ **aws sts decode-authorization-message --encoded-message**  VoZsF9LnYO9v0_WhtW-_Ir4to1hUoQLUQ8HJDZ4PKpOnk-mu3_RnbB9L5JhMzB162pdVRq0QNXDQiBsi3f6-k2U64oiXOkbzviIOpyHXOiRCa5_BVkl04cvkP4G_NfWPu3I3eW3vcdyQ7XOebFWWR73ia-JsuL6XJR_fljhP1Llx-ymm0WVS84CkJLB99PJaAXpHzdd-zH3CNFfc8gSU8xs_47VQLvcL3GOKIMDAI7tPg5IQfakojCUuIuLaKk82EBnCvvZCgeJ8YgG75kWjkrzTSP3kvCmYHFmy1e0-slfkN3DeD5ZfiLU-m-qwbeFQ1LqSZBjeCUEK90mM0IUq8hVVA71gZpbQHNyhcDJDVj-ytaZIql56fttrYlkL0uiOjygfv33MUjhddXu4tzwP5faiP3ZRiaefhFgfh5DtwFmdelMPOC9aBj3aeVfEq4Dy3N6H0Vg_7hjQ6tM4IQTV8tYkoWMkCJhuZC0M2I36PQ1_z78WeicCvyQoxVRfHfpRDdQpkNxqi2rCFi7GbkSF0THI1WK2Ipk3qL1SSKhn_OcJWlt4ydRDVQXqX3B31upje8ZdrU589Kt2YMOlKplZwxrvdO8LZ8yX8NyC4UFd7vFPWbqwPY9RpDeNf6wfFMaR5irTTbdVrGpdIoV3ccwWNXg9fFM2Se9_7UIO*

    *{*

    *"DecodedMessage": "{\"allowed\":false,\"**explicitDeny\":false**,\"matchedStatements\":{\"items\":[]},\"**failures**\":{\"items\":[]},\"context\":{\"principal\":{\"id\":\"AIDAJMTDFB4RF7V2H7HJU\",\"name\":\"testiamuser\",\"arn\":\"arn:aws:iam::XXXXXX:user/testiamuser\"},\"action\":\"**ec2:RunInstances\**",\"resource\":\"arn:aws:ec2:us-west-2:XXXXXX:instance/*\",\"conditions\":{\"items\":[{\"key\":\"ec2:InstanceMarketType\",\"values\":{\"items\":[{\"value\":\"on-demand\"}]}},{\"key\":\"aws:Resource\",\"values\":{\"items\":[{\"value\":\"instance/*\"}]}},{\"key\":\"aws:Account\",\"values\":{\"items\":[{\"value\":\"XXXXXX\"}]}},{\"key\":\"ec2:AvailabilityZone\",\"values\":{\"items\":[{\"value\":\"us-west-2b\"}]}},{\"key\":\"ec2:ebsOptimized\",\"values\":{\"items\":[{\"value\":\"true\"}]}},{\"key\":\"ec2:IsLaunchTemplateResource\",\"values\":{\"items\":[{\"value\":\"false\"}]}},{\"key\":\"ec2:InstanceType\",\"values\":{\"items\":[{\"value\":\"m4.large\"}]}},{\"key\":\"ec2:RootDeviceType\",\"values\":{\"items\":[{\"value\":\"ebs\"}]}},{\"key\":\"aws:Region\",\"values\":{\"items\":[{\"value\":\"us-west-2\"}]}},{\"key\":\"aws:Service\",\"values\":{\"items\":[{\"value\":\"ec2\"}]}},{\"key\":\"ec2:InstanceID\",\"values\":{\"items\":[{\"value\":\"*\"}]}},{\"key\":\"aws:Type\",\"values\":{\"items\":[{\"value\":\"instance\"}]}},{\"key\":\"ec2:Tenancy\",\"values\":{\"items\":[{\"value\":\"default\"}]}},{\"key\":\"ec2:Region\",\"values\":{\"items\":[{\"value\":\"us-west-2\"}]}},{\"key\":\"aws:ARN\",\"values\":{\"items\":[{\"value\":\"arn:aws:ec2:us-west-2:XXXXXX:instance/*\"}]}}]}}}"*

    *}*

*The complete policy will look like this*

<iframe src="https://medium.com/media/1269508e781ac829dcaccc5723c533bc" frameborder=0></iframe>

*GitHub link:*
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/blob/master/ec2-t2-only-instance-full.json)

## ***Creating and Attaching Policy via AWS CLI***

    ** Create a file mytestpolicy and copy the policy we create aboveAWS*

    ** **$ aws iam create-policy --policy-name my-t2-restriction-policy --policy-document  file://mytestpolicy***

    *{*

    *"Policy": {*

    *"PolicyName": "my-t2-restriction-policy",*

    *"PermissionsBoundaryUsageCount": 0,*

    *"CreateDate": "2019-02-20T20:16:08Z",*

    *"AttachmentCount": 0,*

    *"IsAttachable": true,*

    *"PolicyId": "ANPAIOJ75XXVI6OW22S5Q",*

    *"DefaultVersionId": "v1",*

    *"Path": "/",*

    *"Arn": "arn:aws:iam::XXXXXX:policy/my-t2-restriction-policy",*

    *"UpdateDate": "2019-02-20T20:16:08Z"*

    *}*

    *}*

    *# To attach this policy to a particular user*

    *$ **aws iam attach-user-policy --policy-arn arn:aws:iam::XXXXX:policy/my-t2-restriction-policy --user-name plakhera***

## ***Creating and Attaching Policy via Terraform***

<iframe src="https://medium.com/media/e008edf3e8067c91def4bab3ae707c50" frameborder=0></iframe>

*GitHub link*
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/blob/master/ec2-t2-instance-restriction.tf)

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

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
