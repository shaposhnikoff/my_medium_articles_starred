
# 100 Days of DevOps — Day 4(CloudWatch log agent Installation — Centos7)

Welcome to Day 4 of 100 Days of DevOps, On the first day we discussed CloudWatch https://medium.com/devopslinks/100-days-of-devops-day-1-introduction-to-cloudwatch-metrics-b04be36307a8 let extend that concept further by using CloudWatch logs and Collecting Custom Metrics via a cloudwatch agent

### *Problem Statement*

* *Collect the System logs(/var/log/messages and /var/log/secure) and push it to CloudWatch Logs*

* *Collect custom metrics(Memory, Disk Space, Swap Utilization)*

### *Solution*

*This can be achieved via in one of the two ways*

* *AWS Console*

* *AWS System Manager*
> *What are the Cloudwatch logs?*

*Amazon CloudWatch Logs is used to monitor, store, and access your log files from Amazon Elastic Compute Cloud (Amazon EC2) instances, AWS CloudTrail, Route 53, DNS Logs and other sources. You can then retrieve the associated log data from CloudWatch Logs.*

![](https://cdn-images-1.medium.com/max/2000/1*tliaClRKRUZPBrjvo0_0Zw.jpeg)
> *Scenario1: Perform the step manually*

* *Collect System as well as application logs*

* *System metrics(eg: memory, disk, swap utilization)*
> *Step1: Setup IAM Role(CloudWatch agent installed on EC2 instance, require permission to put data and that why we need to setup this role)*

* *Go to IAM console [https://console.aws.amazon.com/iam/home?region=us-west-2#/home](https://console.aws.amazon.com/iam/home?region=us-west-2#/home)*

* *Roles → Create role → AWS service → EC2*

* *Search for these two policies ( (This is required for the cloudwatch agent to interact with CloudWatch logs)and [AmazonSSMFullAccess](https://console.aws.amazon.com/iam/home?region=us-west-2#/policies/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2FAmazonSSMFullAccess)(To automate the process of cloudwatch agent installation)*

![](https://cdn-images-1.medium.com/max/4436/1*UC44-vdMYdFvJiZmT-g3UQ.png)

* *Create the role*
> *Step2: Attach this role to an EC2 instance*

![](https://cdn-images-1.medium.com/max/2972/1*nmYvcC-zTa5WPli_6-VnTQ.png)

![](https://cdn-images-1.medium.com/max/5708/1*ImmrgjgI8F0kjdeQ9icNLg.png)
> *Step3: Install the Cloudwatch agent*

    *# wget [https://s3.amazonaws.com/amazoncloudwatch-agent/linux/amd64/latest/AmazonCloudWatchAgent.zip](https://s3.amazonaws.com/amazoncloudwatch-agent/linux/amd64/latest/AmazonCloudWatchAgent.zip)*

    *# unzip AmazonCloudWatchAgent.zip*

    *# ./install.sh*

* *Finally the big step*

    ***# Invoke the installer(This will step us through, how to install the agent)***

    *# sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard*

    *=============================================================*

    *= Welcome to the AWS CloudWatch Agent Configuration Manager =*

    *=============================================================*

    ***On which OS are you planning to use the agent?***

    ***1. linux***

    *2. windows*

    *default choice: [1]:*

    *Trying to fetch the default region based on ec2 metadata...*

    ***Are you using EC2 or On-Premises hosts?***

    ***1. EC2***

    *2. On-Premises*

    *default choice: [1]:*

    ***Do you want to turn on StatsD daemon?***

    ***1. yes***

    *2. no*

    *default choice: [1]:*

    ***Which port do you want StatsD daemon to listen to?***

    ***default choice: [8125]***

    ***What is the collect interval for StatsD daemon?***

    ***1. 10s***

    *2. 30s*

    *3. 60s*

    *default choice: [1]:*

    ***What is the aggregation interval for metrics collected by StatsD daemon?***

    *1. Do not aggregate*

    *2. 10s*

    *3. 30s*

    ***4. 60s***

    *default choice: [4]:*

    ***Do you want to monitor metrics from CollectD?***

    *1. yes*

    *2. no*

    ***default choice: [1]:***

    ***Do you want to monitor any host metrics? e.g. CPU, memory, etc.***

    ***1. yes***

    *2. no*

    *default choice: [1]:*

    ***Do you want to monitor cpu metrics per core? Additional CloudWatch charges may apply.***

    *1. yes*

    ***2. no***

    *default choice: [1]:*

    *2*

    ***Do you want to add ec2 dimensions (ImageId, InstanceId, InstanceType, AutoScalingGroupName) into all of your metrics if the info is available?***

    ***1. yes***

    *2. no*

    *default choice: [1]:*

    ***Would you like to collect your metrics at high resolution (sub-minute resolution)? This enables sub-minute resolution for all metrics, but you can customize for specific metrics in the output json file.***

    *1. 1s*

    *2. 10s*

    *3. 30s*

    ***4. 60s***

    *default choice: [4]:*

    ***Which default metrics config do you want?***

    *1. Basic*

    *2. Standard*

    ***3. Advanced  <--Make sure you choose advance then only you will see(Memory,Swap,Disk...)***

    *4. None*

    *default choice: [1]:*

    *3*

    *Current config as follows:*

    *{*

    *"metrics": {*

    *"append_dimensions": {*

    *"AutoScalingGroupName": "${aws:AutoScalingGroupName}",*

    *"ImageId": "${aws:ImageId}",*

    *"InstanceId": "${aws:InstanceId}",*

    *"InstanceType": "${aws:InstanceType}"*

    *},*

    *"metrics_collected": {*

    *"collectd": {*

    *"metrics_aggregation_interval": 60*

    *},*

    *"cpu": {*

    *"measurement": [*

    *"cpu_usage_idle",*

    *"cpu_usage_iowait",*

    *"cpu_usage_user",*

    *"cpu_usage_system"*

    *],*

    *"metrics_collection_interval": 60,*

    *"totalcpu": false*

    *},*

    ***"disk": {***

    ***"measurement": [***

    ***"used_percent",***

    ***"inodes_free"***

    *],*

    *"metrics_collection_interval": 60,*

    *"resources": [*

    *"*"*

    *]*

    *},*

    ***"diskio": {***

    ***"measurement": [***

    ***"io_time",***

    ***"write_bytes",***

    ***"read_bytes",***

    ***"writes",***

    ***"reads"***

    *],*

    *"metrics_collection_interval": 60,*

    *"resources": [*

    *"*"*

    *]*

    *},*

    ***"mem": {***

    ***"measurement": [***

    ***"mem_used_percent"***

    *],*

    *"metrics_collection_interval": 60*

    *},*

    ***"netstat": {***

    ***"measurement": [***

    ***"tcp_established",***

    ***"tcp_time_wait"***

    *],*

    *"metrics_collection_interval": 60*

    *},*

    *"statsd": {*

    *"metrics_aggregation_interval": 60,*

    *"metrics_collection_interval": 10,*

    *"service_address": ":8125"*

    *},*

    *"swap": {*

    *"measurement": [*

    ***"swap_used_percent"***

    *],*

    *"metrics_collection_interval": 60*

    *}*

    *}*

    *}*

    *}*

    ***Are you satisfied with the above config? Note: it can be manually customized after the wizard completes to add additional items.***

    ***1. yes***

    *2. no*

    *default choice: [1]:*

    ***Do you have any existing CloudWatch Log Agent (http://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AgentReference.html) configuration file to import for migration?***

    ***1. yes***

    *2. no*

    *default choice: [2]:*

    ***Do you want to monitor any log files?***

    ***1. yes***

    *2. no*

    *default choice: [1]:*

    *Log file path:*

    ***/var/log/messages***

    ***Log group name:***

    *default choice: **[messages]***

    *Log stream name:*

    *default choice: [{instance_id}]*

    *Do you want to specify any additional log files to monitor?*

    *1. yes*

    *2. no*

    *default choice: [1]:*

    *Log file path:*

    ***/var/log/secure***

    *Log group name:*

    *default choice: **[secure]***

    *Log stream name:*

    *default choice: **[{instance_id}]***

    *Do you want to specify any additional log files to monitor?*

    *1. yes*

    *2. no*

    *default choice: [1]:*

    *2*

    *Saved config file to /opt/aws/amazon-cloudwatch-agent/bin/config.json successfully.*

    *Current config as follows:*

    *{*

    *"logs": {*

    *"logs_collected": {*

    *"files": {*

    *"collect_list": [*

    *{*

    *"file_path": "/var/log/messages",*

    *"log_group_name": "messages",*

    *"log_stream_name": "{instance_id}"*

    *},*

    *{*

    *"file_path": "/var/log/secure",*

    *"log_group_name": "secure",*

    *"log_stream_name": "{instance_id}"*

    *}*

    *]*

    *}*

    *}*

    *},*

    *"metrics": {*

    *"append_dimensions": {*

    *"AutoScalingGroupName": "${aws:AutoScalingGroupName}",*

    *"ImageId": "${aws:ImageId}",*

    *"InstanceId": "${aws:InstanceId}",*

    *"InstanceType": "${aws:InstanceType}"*

    *},*

    *"metrics_collected": {*

    *"collectd": {*

    *"metrics_aggregation_interval": 60*

    *},*

    *"cpu": {*

    *"measurement": [*

    *"cpu_usage_idle",*

    *"cpu_usage_iowait",*

    *"cpu_usage_user",*

    *"cpu_usage_system"*

    *],*

    *"metrics_collection_interval": 60,*

    *"totalcpu": false*

    *},*

    *"disk": {*

    *"measurement": [*

    *"used_percent",*

    *"inodes_free"*

    *],*

    *"metrics_collection_interval": 60,*

    *"resources": [*

    *"*"*

    *]*

    *},*

    *"diskio": {*

    *"measurement": [*

    *"io_time",*

    *"write_bytes",*

    *"read_bytes",*

    *"writes",*

    *"reads"*

    *],*

    *"metrics_collection_interval": 60,*

    *"resources": [*

    *"*"*

    *]*

    *},*

    *"mem": {*

    *"measurement": [*

    *"mem_used_percent"*

    *],*

    *"metrics_collection_interval": 60*

    *},*

    *"netstat": {*

    *"measurement": [*

    *"tcp_established",*

    *"tcp_time_wait"*

    *],*

    *"metrics_collection_interval": 60*

    *},*

    *"statsd": {*

    *"metrics_aggregation_interval": 60,*

    *"metrics_collection_interval": 10,*

    *"service_address": ":8125"*

    *},*

    *"swap": {*

    *"measurement": [*

    *"swap_used_percent"*

    *],*

    *"metrics_collection_interval": 60*

    *}*

    *}*

    *}*

    *}*

    *Please check the above content of the config.*

    *The config file is also located at /opt/aws/amazon-cloudwatch-agent/bin/config.json.*

    *Edit it manually if needed.*

    ***Do you want to store the config in the SSM parameter store?***

    ***1. yes***

    *2. no*

    *default choice: [1]:*

    *1*

    *What parameter store name do you want to use to store your config? (Use 'AmazonCloudWatch-' prefix if you use our managed AWS policy)*

    *default choice: [AmazonCloudWatch-linux]*

    ***CloudWatch-Advanceconfig-linux***

    *Trying to fetch the default region based on ec2 metadata...*

    *Which region do you want to store the config in the parameter store?*

    *default choice: [us-west-2]*

    ***Which AWS credential should be used to send json config to parameter store?***

    ***1. XXXXXXXX(From SDK) <-- As we are using Roles***

    *2. Other*

    *default choice: [1]:*

    *Successfully put config to parameter store CloudWatch-Advanceconfig-linux.*

    *Program exits now.*

* *The final step is the validation*

    *# sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s*

    */opt/aws/amazon-cloudwatch-agent/bin/config-downloader --output-dir /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.d --download-source file:/opt/aws/amazon-cloudwatch-agent/bin/config.json --mode ec2 --config /opt/aws/amazon-cloudwatch-agent/etc/common-config.toml --multi-config default*

    *Successfully fetched the config and saved in /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.d/file_config.json.tmp*

    *Start configuration validation...*

    */opt/aws/amazon-cloudwatch-agent/bin/config-translator --input /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json --input-dir /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.d --output /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.toml --mode ec2 --config /opt/aws/amazon-cloudwatch-agent/etc/common-config.toml --multi-config default*

    *Valid Json input schema.*

    *No csm configuration found.*

    *Configuration validation first phase succeeded*

    */opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent -schematest -config /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.toml*

    *Configuration validation second phase succeeded*

    *Configuration validation succeeded*

    *Created symlink from /etc/systemd/system/multi-user.target.wants/amazon-cloudwatch-agent.service to /etc/systemd/system/amazon-cloudwatch-agent.service.*

    *Redirecting to /bin/systemctl restart amazon-cloudwatch-agent.service*

* *For centos, you need to perform two additional steps*

    ***# yum -y install epel-release
    # yum -y install collectd***
> ***Scenario2: Using SSM Parameter store***

    *# wget [https://s3.amazonaws.com/amazoncloudwatch-agent/linux/amd64/latest/AmazonCloudWatchAgent.zip](https://s3.amazonaws.com/amazoncloudwatch-agent/linux/amd64/latest/AmazonCloudWatchAgent.zip)*

    *# unzip AmazonCloudWatchAgent.zip*

    *# ./install.sh*

* *But rather than perform the next wizard step manually, we can use system manager parameter store*

    *# sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c ssm:CloudWatch-Advanceconfig-linux -s*

    */opt/aws/amazon-cloudwatch-agent/bin/config-downloader --output-dir /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.d --download-source ssm:CentosCloudWatch-linux --mode ec2 --config /opt/aws/amazon-cloudwatch-agent/etc/common-config.toml --multi-config default
    Region: us-west-2
    credsConfig: map[]
    Successfully fetched the config and saved in /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.d/ssm_CentosCloudWatch-linux.tmp
    Start configuration validation...
    /opt/aws/amazon-cloudwatch-agent/bin/config-translator --input /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json --input-dir /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.d --output /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.toml --mode ec2 --config /opt/aws/amazon-cloudwatch-agent/etc/common-config.toml --multi-config default
    Valid Json input schema.
    No csm configuration found.
    Configuration validation first phase succeeded
    /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent -schematest -config /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.toml
    Configuration validation second phase succeeded
    Configuration validation succeeded
    Created symlink from /etc/systemd/system/multi-user.target.wants/amazon-cloudwatch-agent.service to /etc/systemd/system/amazon-cloudwatch-agent.service.
    Redirecting to /bin/systemctl restart amazon-cloudwatch-agent.service*

*Looking forward from you guys to join this journey and spend a minimum an hour every day for the next 100 days on DevOps work and post your progress using any of the below medium.*

*On CloudWatch Dashboard, you will see something like this*

![](https://cdn-images-1.medium.com/max/2000/1*ekHAV3wL7GqaQS1r9PlUmg.png)

![](https://cdn-images-1.medium.com/max/4348/1*ySqfNswtKEoD9vQ_3W9xEw.png)

![](https://cdn-images-1.medium.com/max/4960/1*ebaBuBX4vkC7tOLCcO_MDA.png)

![](https://cdn-images-1.medium.com/max/4480/1*RWLMNWVAu1WbiHgcnkwvqA.png)

*For CloudWatch LogGroup, you will see something like this*

![](https://cdn-images-1.medium.com/max/5160/1*JMrynHmmQDSjdsy20DV24w.png)

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
