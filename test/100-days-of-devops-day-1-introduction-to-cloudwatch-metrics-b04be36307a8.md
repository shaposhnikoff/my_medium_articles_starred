
# 100 Days of DevOps — Day 1(Introduction to CloudWatch Metrics)

Welcome to Day 1 of 100 Days of DevOps, I would like to start this journey with one of the most critical concepts in DevOps Monitoring and Alerting.

### ***Problem Statement***

* *Create a CloudWatch alarm that sends an email using SNS notification when CPU Utilization is more than 70%.*

* *Creating a Status Check Alarm to check System and Instance failure and send an email using SNS notification*

### *Solution*

*This can be achieved via in one of the three ways*

* *AWS Console*

* *AWS CLI*

* *Terraform*

*NOTE: This is not the complete list and there are more ways to achieve the same*
> *What is CloudWatch?*

*AWS CloudWatch is a monitoring service to monitor AWS resources, as well as the applications that run on AWS.*

*As per official documentation*
> *Amazon CloudWatch monitors your Amazon Web Services (AWS) resources and the applications you run on AWS in real time. You can use CloudWatch to collect and track metrics, which are variables you can measure for your resources and applications.*

*Reference:*
[**What is Amazon CloudWatch? - Amazon CloudWatch**
*Monitor your AWS resources and applications using Amazon CloudWatch to collect and track metrics on performance.*docs.aws.amazon.com](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)

*EC2/Host Level Metrics that CloudWatch monitors by default consist of*

* *CPU*

* *Network*

* *Disk*

![](https://cdn-images-1.medium.com/max/4696/1*k2jBAHOMsKRkmjOIt22dhw.png)

* *Status Check*

![](https://cdn-images-1.medium.com/max/4452/1*1Ldt-iquTehzqQ3Rkji2UQ.png)

*There are two types of status check*

* ***System status check:** Monitor the AWS System on which your instance runs. It either requires AWS involvement to repair or you can fix it by yourself by just stop/start the instance(in case of EBS volumes).Examples of problems that can cause system status checks to fail*

    ** Loss of network connectivity
    * Loss of system power
    * Software issues on the physical host
    * Hardware issues on the physical host that impact network reachability*

* ***Instance status check:** Monitor the software and network configuration of an individual instance. It checks/detects problems that require your involvement to repair.*

    ** Incorrect networking or startup configuration
    * Exhausted memory
    * Corrupted filesystem
    * Incompatible kernel*

***NOTE***

* *Memory/RAM utilization is custom metrics.*

* *By default, EC2 monitoring is 5 minutes intervals but we can always enable detailed monitoring(1 minutes interval, but that will cost you some extra $$$)*

*Reference*
[**Amazon CloudWatch Pricing - Amazon Web Services (AWS)**
*EC2 Detailed Monitoring is priced at $2.10 per instance per month and goes down to $0.14 per instance at the lowest…*aws.amazon.com](https://aws.amazon.com/cloudwatch/pricing/)

***P.S:** CloudWatch can be used on premise too. We just need to install the SSM(System Manager) and CloudWatch agent.*

*Enough of the theoretical concept, let setup first CloudWatch alarm*

* ***Scenario1:** We want to create a CloudWatch alarm that sends an email using SNS notification when CPU Utilization is more than 70%*

![](https://cdn-images-1.medium.com/max/2000/1*GQ1ShLi5sQjSZswT0DpFSg.jpeg)

***Solution1:** Setup a CPU Usage Alarm using the AWS Management Console*

* *Open the CloudWatch console at [https://console.aws.amazon.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/).*

* *In the navigation pane, choose **Alarms**, **Create Alarm**.*

* *Go to Metric → Select metric → EC2 → Per-Instance-Metrics → CPU Utilization → Select metric*

![](https://cdn-images-1.medium.com/max/5696/1*KHdMvmF0A6zGW7Oo0Nkn1Q.png)

![](https://cdn-images-1.medium.com/max/4208/1*ToyTn3_8jiFicZalMxEI8w.png)

    *Define the Alarm as follows*

    ** Type the unique name for the alarm(eg: HighCPUUtilizationAlarm)
    * Description of the alarm
    * Under whenever,choose >= and type 70, for type 2. This specify that the alarm is triggered if the CPU usage is above 70% for two consecutive sampling period
    * Under Additional settings, for treat missing data as, choose bad(breaching threshold), as missing data points may indicate that the instance is down
    * Under Actions, for whenever this alarm, choose state is alarm. For Send notification to, select an exisiting SNS topic or create a new one 
    * To create a new SNS topic, choose new list, for send notification to, type a name of SNS topic(for eg: HighCPUUtilizationThreshold) and for Email list type a comma-seperated list of email addresses to be notified when the alarm changes to the ALARM state.
    * Each email address is sent to a topic subscription confirmation email. You must confirm the subscription before notifications can be sent.
    * Click on Create Alarm*

![](https://cdn-images-1.medium.com/max/3648/1*fzyaNZVUqk6rFAmCUmhkOQ.png)

***Solution2: Setup CPU Usage Alarm using the AWS CLI***

* *Create an alarm using the put-metric-alarm command*

    *aws cloudwatch put-metric-alarm --alarm-name cpu-mon --alarm-description "Alarm when CPU exceeds 70 percent" --metric-name CPUUtilization --namespace AWS/EC2 --statistic Average --period 300 --threshold 70 --comparison-operator GreaterThanThreshold  --dimensions "Name=InstanceId,Value=i-12345678" --evaluation-periods 2 --alarm-actions arn:aws:sns:us-east-1:111122223333:MyTopic --unit Percent*

* *Using the command line, we can test the Alarm by forcing an alarm state change using a set-alarm-state command*

* *Change the alarm-state from INSUFFICIENT_DATA to OK*

    *# aws cloudwatch set-alarm-state --alarm-name "cpu-monitoring" --state-reason "initializing" --state-value OK*

* *Change the alarm-state from OK to ALARM*

    *# aws cloudwatch set-alarm-state --alarm-name "cpu-monitoring" --state-reason "initializing" --state-value ALARM*

* *Check if you have received an email notification about the alarm*

![](https://cdn-images-1.medium.com/max/4644/1*i5C6lc2ZgXnf8xdZ7uw9Yg.png)

*GITHUB link:*
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/blob/master/cloudwatch_cpu_alarm_via_aws_cli)

*Solution3: **Setup CPU Usage Alarm using the Terraform***

    *#cloudwatch.tf*

    *resource "aws_cloudwatch_metric_alarm" "cpu-utilization" {
      alarm_name                = "high-cpu-utilization-alarm"
      comparison_operator       = "GreaterThanOrEqualToThreshold"
      evaluation_periods        = "2"
      metric_name               = "CPUUtilization"
      namespace                 = "AWS/EC2"
      period                    = "120"
      statistic                 = "Average"
      threshold                 = "80"
      alarm_description         = "This metric monitors ec2 cpu utilization"
      alarm_actions             = [ "${aws_sns_topic.alarm.arn}" ]*

    *dimensions {
        InstanceId = "${aws_instance.my_instance.id}"
      }
    }*

*GitHub link:*
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/blob/master/cloudwatch_cpu_alarm_via_terraform)
> ***Scenario2: Create a status check alarm to notify when an instance has failed a status check***
> ***Solution1: Creating a Status Check Alarm Using the AWS Console***

1. *Open the Amazon EC2 console at [https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/).*

1. *In the navigation pane, choose **Instances**.*

1. *Select the instance, choose the **Status Checks** tab, and choose to **Create Status Check Alarm**.*

![](https://cdn-images-1.medium.com/max/4944/1*nTyJCYo4gW88Y2D-G_MGdQ.png)

![](https://cdn-images-1.medium.com/max/3848/1*U2jtEvsc1iShNHQ4mLV76g.png)

    ** You can create new SNS notification or use the exisiting one(I am using the existing one create in earlier example of high CPU utilization)
    * In **Whenever**, select the status check that you want to be notified about(options Status Check Failed(Any), Status Check Failed(Instance) and Status Check Failed(System)
    * In **For at least**, set the number of periods you want to evaluate and in **consecutive periods**, select the evaluation period duration before triggering the alarm and sending an email.
    * In **Name of alarm**, replace the default name with another name for the alarm.
    * Choose **Create Alarm**.*
> ***Solution2: To create a status check alarm via AWS CLI***

* *Use the put-metric-alarm command to create the alarm*

    *aws cloudwatch put-metric-alarm --alarm-name StatusCheckFailed-Alarm-for-test-instance --metric-name StatusCheckFailed --namespace AWS/EC2 --statistic Maximum --dimensions Name=InstanceId,Value=i-1234567890abcdef0 --unit Count --period 300 --evaluation-periods 2 --threshold 1 --comparison-operator GreaterThanOrEqualToThreshold --alarm-actions arn:aws:sns:us-west-2:111122223333:my-sns-topic
    *
> *GitHub link:*
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/blob/master/cloudwatch_status_check_via_aws_cli)
> ***Solution3:To create a status check alarm via Terraform***

    *resource "aws_cloudwatch_metric_alarm" "instance-health-check" {
      alarm_name                = "instance-health-check"
      comparison_operator       = "GreaterThanOrEqualToThreshold"
      evaluation_periods        = "1"
      metric_name               = "StatusCheckFailed"
      namespace                 = "AWS/EC2"
      period                    = "120"
      statistic                 = "Average"
      threshold                 = "1"
      alarm_description         = "This metric monitors ec2 health status"
      alarm_actions             = [ "${aws_sns_topic.alarm.arn}" ]*

    *dimensions {
        InstanceId = "${aws_instance.my_instance.id}"
      }
    }*

*GitHub link:*
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/blob/master/cloudwatch_status_check_via_terraform)

*Looking forward from you guys to join this journey and spend a minimum an hour every day for the next 100 days on DevOps work and post your progress using any of the below medium.*

* *Twitter: [@100daysofdevops](http://twitter.com/100daysofdevops) OR @[lakhera2015](https://twitter.com/lakhera2015)*

* *Facebook: [https://www.facebook.com/groups/795382630808645/](https://www.facebook.com/groups/795382630808645/)*

* *Medium: [https://medium.com/@devopslearning](https://medium.com/@devopslearning)*

* *Slack: [https://devops-myworld.slack.com/messages/CF41EFG49/](https://devops-myworld.slack.com/messages/CF41EFG49/)*

* *GitHub Link:[https://github.com/100daysofdevops](https://github.com/100daysofdevops)*

**Reference**
[**100 Days of DevOps — Day 0**
*D-day is just one day away and finally, this is a continuation of the post(I posted a month earlier)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-0-4f2c9750542d)
[**100 Days of DevOps**
*Motivation*medium.com](https://medium.com/@devopslearning/100-days-of-devops-81faf13bf772)
