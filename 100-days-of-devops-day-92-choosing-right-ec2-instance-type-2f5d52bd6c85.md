
# 100 Days of DevOps — Day 92-Choosing Right EC2 Instance Type

Welcome to Day 92 of 100 Days of DevOps, Focus for today is Choosing Right EC2 Instance Type

*What is Amazon Elastic Compute Cloud(EC2)?*

*It’s a Virtual Servers in the cloud*
> ***Advantage***

* *Scale up or down as quickly as needed*

* *Pay as you go, model*
> ***Instance Type:** AWS Provide different instance based on workload as some instance need more memory, some needs more CPU and some needs more I/O.*

![](https://cdn-images-1.medium.com/max/5600/1*4fojO7DUrCy4cZoDm0Lhvg.png)

***M5d.xlarge(API Name of EC2 Instance)***

    *M: Instance Family(Multipurpose/General Purpose good balance for CPU/Memory)*

    *5: Instance Generation(5th generation)*

    *d: Additional Capabilities which are optinally avilable(local high-speed NVMe disk)*

    *xlarge: Instance Size(T-shirt size)*
> ***Amazon Machine Image(AMI):** Think it as an OS Image for your Instance(Linux/Window).*

* ***Amazon Maintained:** Broad set of Linux and Windows Images. Kept up-to-date by Amazon in each region*

* ***MarketPlace Maintained:** Managed and maintained by AWS Market Place Partners*

* ***Custom AMI:** AMIs you have created from Amazon EC2 instances. Can keep private, share with other accounts or publish to the community.*
> ***Choice of processors and architectures***

    ** Intel Xeon Scalable(Skylake) processor*

    ** AMD EPYC processor*

    ** AWS Graviton Processor based on 64-bit Arm arch*

## ***General Purpose Instance Workload***

* *Web/App servers*

* *Enterprise apps*

* *Gaming servers*

* *Caching fleets*

* *Analytics applications*

* *Dev/test environments*
> ***M5: General purpose instances***

![](https://cdn-images-1.medium.com/max/5700/1*aeEuoGAwTg8ZW7oRAmknaA.png)

* *Balance of compute, memory and networking resources*

* *Powered by 2.5GHz Intel Xeon Scalable Processors(Skylake)*

* *As shown in the figure above, m5.24x large has 96vCPU and 384GiB of memory*

* *4:1 GiB to VCPU*
> ***M5a: Comes with AMD processor for 10% lower cost***

![](https://cdn-images-1.medium.com/max/5700/1*aM4rl6n9tpS2aiBOSQhlcQ.png)
> ***M5d: Comes with high-performance local NVMe SSD storage***

![](https://cdn-images-1.medium.com/max/5664/1*iSe1EGCDnqlY0QY4yzpmXA.png)

## ***Memory Optimized Instances Workloads***

* *In-memory caches*

* *High-Performance databases*

* *Big data analytics*

![](https://cdn-images-1.medium.com/max/5668/1*L5CLZnZXemTXiYUxIgOCmQ.png)
> ***R5: Memory Optimized Instances***

* *Memory-optimized instances with 8:1 GiB to VCPU*

*Similar to M5, R5 comes with R5d(local NVMe SSD storage) and R5a(AMD processor)*
> ***X1 and X1e: Large Scale Memory Optimized Instances***

![](https://cdn-images-1.medium.com/max/5712/1*lCS3_vnr4e5QgFQ66HXQUA.png)

* *For large in-memory workloads*

* *X1(16:1 GiB to vCPU ratio) and X1e(32:1 GiB to vCPU ratio)*

* *Suitable for in-memory database eg: SAP HANA, big data processing engines(Apache Spark) and DB Workloads(Oracle)*

## *I/O optimized instances workloads*

* *High-Performance Database*

* *Real-Time Analytics*

* *Transactional Workloads*

* *No SQL database*
> ***I3: I/O Optimized Instances***

* *Offers very high Random I/O(up to 3.3 million IOPS) and disk throughput(up to 16GB/s)*

![](https://cdn-images-1.medium.com/max/5700/1*zdYrbO_CfWGQtWLDmwVOvw.png)

## *Compute-optimized instances workloads*

* *Batch processing*

* *Distributed analytics*

* *High-perf computing(HPC)*

* *Ad serving*

* *Multiplayer Gaming*

* *Video encoding*
> ***C5: Compute Optimized Instances***

* *Custom 3.0GHz Intel Xeon Scalable Processor(Skylake)*

* *72vCPU and 144GiB of memory (2:1 Memory:vCPU ratio)*

* *25Gbps network bandwidth*

![](https://cdn-images-1.medium.com/max/5752/1*WLj6xYKZzXzL0VTT73uTkw.png)
> ***C5d with local NVMe-based SSD storage***

![](https://cdn-images-1.medium.com/max/5668/1*92XkBk6k9ZnRaRTNNixWcg.png)

**References**

### ***AWS Calculator***
[**Amazon Web Services Simple Monthly Calculator**
*The AWS Simple Monthly Calculator helps customers and prospects estimate their monthly AWS bill more efficiently. Using…*calculator.s3.amazonaws.com](https://calculator.s3.amazonaws.com/index.html)
> ***AWS Instance Price Comparision***
[**Amazon EC2 Instance Comparison**
*Edit description*www.ec2instances.info](https://www.ec2instances.info/?region=us-west-2)
> Just to summarize, in choosing the right instance type. In case if you are not sure about workload start with General workload.

![](https://cdn-images-1.medium.com/max/2220/1*74N4DPXPmq2Ojkkun6DKLw.jpeg)

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
[**100 Days of DevOps — Day 91-How to check if the file exists (Bash/Python)**
*Welcome to Day 91 of 100 Days of DevOps, Focus for today is How to check if the file exists (Bash/Python)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-91-how-to-check-if-the-file-exists-bash-python-ddc8087a3cbf)
