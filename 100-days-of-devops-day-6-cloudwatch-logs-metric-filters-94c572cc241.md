
# 100 Days of DevOps — Day 6-CloudWatch Logs(Metric Filters)

Welcome to Day 6 of 100 Days of DevOps, So far

*Day 1(CloudWatch) [https://medium.com/devopslinks/100-days-of-devops-day-1-introduction-to-cloudwatch-metrics-b04be36307a8](https://medium.com/devopslinks/100-days-of-devops-day-1-introduction-to-cloudwatch-metrics-b04be36307a8)*

*Day 2(SNS) [https://medium.com/@devopslearning/100-days-of-devops-day-2-introduction-to-simple-notification-service-sns-97137b2f1f1e](https://medium.com/@devopslearning/100-days-of-devops-day-2-introduction-to-simple-notification-service-sns-97137b2f1f1e)*

*Day 3(CloudTrail) [https://medium.com/@devopslearning/100-days-of-devops-day-3-introduction-to-cloudtrail-5ce923f44584](https://medium.com/@devopslearning/100-days-of-devops-day-3-introduction-to-cloudtrail-5ce923f44584)*

*Day 4(CloudWatch Agent) [https://medium.com/@devopslearning/100-days-of-devops-day-4-cloudwatch-log-agent-installation-centos7-d11054fffdf4](https://medium.com/@devopslearning/100-days-of-devops-day-4-cloudwatch-log-agent-installation-centos7-d11054fffdf4)*

*Day 5(CloudWatch with Slack) [https://medium.com/@devopslearning/100-days-of-devops-day-5-cloudwatch-to-slack-notification-d2d84a192bf2](https://medium.com/@devopslearning/100-days-of-devops-day-5-cloudwatch-to-slack-notification-d2d84a192bf2)*

***Problem:** I want to deploy a simple monitoring system when any** **unauthorized trying to access my servers I will notify via SNS.*

***Solution:** This can be achieved using CloudWatch Metric Filter in combination with SNS.*

![](https://cdn-images-1.medium.com/max/2000/1*0OzqYNGqzR0wtHXdovwg0g.jpeg)

***Step1***

* *Install CloudWatch Agent(Make sure you are pushing /var/log/messages and /var/log/secure logs from your instance to CloudWatch log group)*
[**100 Days of DevOps — Day 4(CloudWatch log agent Installation — Centos7)**
*Welcome to Day 4 of 100 Days of DevOps, On the first day we discussed CloudWatch…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-4-cloudwatch-log-agent-installation-centos7-d11054fffdf4)

* *At the same time, go to CloudWatch Logs and search for Invalid user string*

![](https://cdn-images-1.medium.com/max/5068/1*W5R7viCyHdJG94DTwm9InQ.png)

***Step2***

* *Go to*

    *Management & Governance --> CloudWatch --> Logs --> messages --> 0 fileters --> Add Metric Filter*

![](https://cdn-images-1.medium.com/max/4744/1*QLf-o6MG72tdaT6VMmyg3g.png)

![](https://cdn-images-1.medium.com/max/2528/1*cjAgVUSNhp6Vkh8lTQwPtQ.png)

    ** Filter Pattern : Type Invalid user
    * Select Log Data to Test: Select the right instance*

* *Keep everything default and give your metric some name(Metric Name: InvalidUserlogin)*

![](https://cdn-images-1.medium.com/max/2788/1*qh3VTaI4BNC4U6f-0-AZCQ.png)

* *In the next screen, click on Create Alarm*

![](https://cdn-images-1.medium.com/max/2968/1*mPY8Qg-Ru-M2Koo4hSiDuQ.png)

    ** Give your alarm Name and Description
    * Set the threshold, for demo I am setting up as 1
    * Select the SNS topic *

![](https://cdn-images-1.medium.com/max/4316/1*VGdPnzVwxnqaa4kr8yMshQ.png)

* *Your simple notification system against un-authorized user is up and running.*

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
