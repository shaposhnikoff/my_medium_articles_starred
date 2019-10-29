Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m31[39m }

# 100 Days of DevOps — Day 41-Real-Time Apache Log Analysis using Amazon Kinesis and Amazon…

Welcome to Day 41 of 100 Days of DevOps, Focus for today is Real-Time Apache Log Analytics using Amazon Kinesis and Amazon Elasticsearch Service

*Scenario: To analyze Apache logs using Kinesis Firehose, and ElasticSearch Service*

![](https://cdn-images-1.medium.com/max/2060/1*8sWVykRRIga2R8YnXdNeWA.jpeg)

*Step1: Setup ElasticSearch Cluster*

*Step2: Setup firehose delivery Pipeline, this will continuously insert logs to ElasticSearch Cluster*

*Step3: Send data to Firehose Delivery Stream*

*Step4: Visualize the data using Kibana*
> *What is Amazon Kinesis*

*Amazon Kinesis is the streaming service provided by AWS which makes it easy to collect, process, and analyze real-time, streaming data so you can get timely insights and react quickly to new information.*

![](https://cdn-images-1.medium.com/max/3716/1*u7dWQkbDPIBrZv_A6H07Cw.png)
> *What is AWS ElasticSearch Service*

*AWS ElasticSearch Service is a cost-effective managed service that makes it easy to deploy, manage, and scale open source Elasticsearch for log analytics, full-text search and more.*
> *Step1: Setup ElasticSearch Cluster*

    *Go to [https://us-west-2.console.aws.amazon.com/es/](https://us-west-2.console.aws.amazon.com/es/) --> Create a new domain*

![](https://cdn-images-1.medium.com/max/4976/1*gTdt7v0cfiTUSORR7V796w.png)

    ** Choose Development and testing or based on your requirement*

![](https://cdn-images-1.medium.com/max/3704/1*616AhmPopqlc-nfs1ox01w.png)

![](https://cdn-images-1.medium.com/max/3740/1*CnB3yy0rQO6Uh2Hdw0Hbdw.png)

    ** Give your Elasticsearch domain name(I need to change it to hundreddaysofdevops as numeral 100 is not accepted by AWS) 
    * Choose all the default options*

![](https://cdn-images-1.medium.com/max/3644/1*GWwVv-sw8XDdGvb2yOmmTQ.png)

    ** This is my test cluster and that why I am choosing these wide open options, these are definitely not recommended for PRD environment
    * Network configuration(Public access for your environment choose particular VPC)
    * Access policy: Your policy must be restricted*
> *Step2: Setup firehose delivery Pipeline, this will continuously insert logs to ElasticSearch Cluster*

    *Go to [https://us-west-2.console.aws.amazon.com/firehose](https://us-west-2.console.aws.amazon.com/firehose) --> Create Delivery Stream*

![](https://cdn-images-1.medium.com/max/4420/1*Csk0Xiesaph39LIR-vcNhw.png)

    ** Give your Delivery stream some name and keep all the options as default*

***To see all the Lambda blueprints for Kinesis Data Firehose, with examples in Python and Node.js***

1. *Sign in to the AWS Management Console and open the AWS Lambda console at [https://console.aws.amazon.com/lambda/](https://console.aws.amazon.com/lambda/).*

1. *Choose to **Create function**, and then choose **Blueprints**.*

1. *Search for the keyword “kinesis-firehose-apachelog-to-json-python" to find the [Kinesis Data Firehose Lambda Blueprints](https://console.aws.amazon.com/lambda/home?region=us-east-1#/create?f0=a3c%3D%3AZmlyZWhvc2U%3D&tab=blueprints)*

![](https://cdn-images-1.medium.com/max/5040/1*OEyz-9i1sCWNunbJVI6nuw.png)

![](https://cdn-images-1.medium.com/max/2884/1*2RnbkcPaVu3vdsGNCIhWyg.png)

![](https://cdn-images-1.medium.com/max/5108/1*75JFzdvK4087q2UOi9CpDA.png)

![](https://cdn-images-1.medium.com/max/4336/1*1p1fe47Z8BhXP2TkTyLR2Q.png)

![](https://cdn-images-1.medium.com/max/4340/1*BSSfYivLo8t2bEM_M6s3kw.png)

    ** Source record transformation: Enabled and Choose the lambda function
    * Select Destination: Choose Amazon ElasticSearch Service
    * Domain: Should be auto-populated with ElasticSearch Name
    * Index: Give your index some name(eg: mytestindex)
    * Type(eg: apache)
    * Backup mode: Create a new S3 bucket*

    *NOTE: Backup Mode is to prevent any data loss,firehose store the data in S3 bucket*

* *In the next screen, keep all the setting default, just click on Create or choose new IAM role and create one*

![](https://cdn-images-1.medium.com/max/2484/1*6ARUXLLrTDlrOBk3loqyfQ.png)

* *This role will be auto-populated*

![](https://cdn-images-1.medium.com/max/3032/1*kaVauu50mt5RbaOQvbixGQ.png)

* *Click Next and Create Delivery Stream*
> *Step3: Send data to Firehose Delivery Stream*

* *Installing Kinesis Agent*

    *# sudo yum install –y https://s3.amazonaws.com/streaming-data-agent/aws-kinesis-agent-latest.amzn1.noarch.rpm*

    *# cat /etc/aws-kinesis/agent.json*

<iframe src="https://medium.com/media/a1fd8b6badef2513783b02d16ac741f3" frameborder=0></iframe>

    ***service aws-kinesis-agent start***
> *Step4: Visualize the data using Kibana*

*To create the line chart:*

![](https://cdn-images-1.medium.com/max/2660/1*AzJjAcStJy7H07tx6pxnKQ.png)

*a. Click on Visualize in the navigation bar, and choose Line chart.*

*b. Choose From a new search. To configure your chart, you first need to tell Kibana what data to use for the y-axis:*

*c. In the metrics section, click the arrow next to Y-Axis to configure this.*

*d. Under Aggregation, choose Sum.*

*e. Under Field, choose STATUSCOUNT.*

*f. Now you need to configure the x-axis: f. In the buckets section, select X-Axis under Select buckets type.*

*g. Under Aggregation, choose Date Histogram.*

**Reference:**
[**Send Apache Web Logs to Amazon Elasticsearch Service with Kinesis Firehose | Amazon Web Services**
*We have many customers who own and operate Elasticsearch, Logstash, and Kibana (ELK) stacks to load and visualize…*aws.amazon.com](https://aws.amazon.com/blogs/database/send-apache-web-logs-to-amazon-elasticsearch-service-with-kinesis-firehose/)

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
