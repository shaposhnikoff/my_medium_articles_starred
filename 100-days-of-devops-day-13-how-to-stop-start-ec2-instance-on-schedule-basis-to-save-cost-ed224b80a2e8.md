
# 100 Days of DevOps — Day 13- How to stop/start EC2 instance on schedule basis to save cost

Welcome to Day 13 of 100 Days of DevOps, Let continue our journey and focus of this week is a daily DevOps task, to run the business and to save money. This is one of the ask I came across in Dev env in terms of saving money where I need to shut down all the EC2 instance at 6 pm and bring it back next day at 9 am.

***Problem:** Shutdown all EC2 instance in AWS DEV account at 6pm and bring it back next day at 9am(Monday to Friday)*

***Solution:** Using Lambda function in combination of CloudWatch events.*

* *One of the major challenge in implementing this what would be the case if Developer is working late and he wants his instance to be run beyond 6 pm or if there is an urgent patch he needs to implement and need to work on the weekend?*

* *One common solution we come out is manually specifying the list of an instance in Python Code(Lambda Function) and in case of exception it goes through change management process where we need to remove developer instance manually(Agree not an ideal solution but so far works great)*

***Step1:** Create IAM Role so that Lambda can interact with CloudWatch Events*

    *Go to IAM Console [https://console.aws.amazon.com/iam/home?region=us-west-2#/home](https://console.aws.amazon.com/iam/home?region=us-west-2#/home) --> Roles --> Create role*

![](https://cdn-images-1.medium.com/max/3412/1*_C92ySkyjMw9l0wFubo_vQ.png)

* *In the next screen, select Create Policy, IAM Policy will look like this*

<iframe src="https://medium.com/media/8913c9b112c94c9d3248ccdb37712d74" frameborder=0></iframe>

* *Add this newly created policy to the role*

***Step2:** Create Lambda function*

* *Go to Lambda [https://us-west-2.console.aws.amazon.com/lambda/home?region=us-west-2#/home](https://us-west-2.console.aws.amazon.com/lambda/home?region=us-west-2#/home)*

* *Select Create Function*

![](https://cdn-images-1.medium.com/max/5384/1*M3VKLoR9TVzH0kEkQH48fg.png)

    ** Select Author from scratch
    * Name: Give your Lambda function any name
    * Runtime: Select Python2.7 as runtime
    * Role: Choose the role we create in first step
    * Click on Create function*

* *In this scenario, we need to create Function one to stop instance and others to start an instance*

![](https://cdn-images-1.medium.com/max/5328/1*5hVgwtJR-BplYjzxFbLogw.png)

* *To stop the instance, the code will look like this*

<iframe src="https://medium.com/media/8911fdf1221c0898108d4b8456b7d4af" frameborder=0></iframe>

    ** Change the Value of region
    * In the instance field specify instance id*

* *Keep all the settings as default, just change the timeout value to 10sec*

* *Now we need to perform the same steps for starting the instance*

<iframe src="https://medium.com/media/c2bbc9d2d3716c1f197657f4ef3fb392" frameborder=0></iframe>

***Step2: **Create the CloudWatch event to trigger this Lambda function*

* *Open the [Amazon CloudWatch console](https://console.aws.amazon.com/cloudwatch/).*

* *Choose Events, and then choose Create rule.*

* *Choose Schedule under Event Source.*

![](https://cdn-images-1.medium.com/max/5108/1*QZHLYbupENS6rxLUm3LnLw.png)

    ** Under Cron expression choose * 18 * * ? * (If you want to shutdown your instance at 6pm everyday)
    * Choose Add target, and then choose Lambda function that you created earlier to stop the instance*

* *Same steps need to be repeated for Starting the instance*

![](https://cdn-images-1.medium.com/max/5080/1*vBYvNlrM7JpKTo-d3rgGWg.png)

    ** Only difference is different time schedule
    * Under target different Lambda function(to start the instance)*

***NOTE: **One very important point to note is that all scheduled event is in **UTC timezone**, so you need to customize it based on your timezone.*

* *Once the event is triggered, go to your*

    *Lambda function --> Monitoring --> View logs in CloudWatch*

![](https://cdn-images-1.medium.com/max/5096/1*IowvatUak1QvLXCrFP5nTw.png)

* *For stopping the instance, you will see something like this*

![](https://cdn-images-1.medium.com/max/5044/1*48AYgeziMtKt-odkFxnl7w.png)

* *For starting the instance*

![](https://cdn-images-1.medium.com/max/5032/1*yJRZcHARg71DWimbVyATWw.png)

* *The simple automation system is ready in stopping/starting the instance and to save some company money.*

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
