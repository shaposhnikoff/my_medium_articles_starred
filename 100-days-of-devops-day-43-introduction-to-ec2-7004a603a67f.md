
# 100 Days of DevOps — Day 43- Introduction to EC2

Welcome to Day 43 of 100 Days of DevOps, Focus for today is Introduction to EC2.

*Elastic Compute Cloud(EC2) is the virtual server in the AWS cloud.*

* *EC2 virtual server also referred to as an instance and EC2 comes with different instance type [https://aws.amazon.com/ec2/instance-types](https://aws.amazon.com/ec2/instance-types/)*

* *It’s important to choose the correct instance type so that it will handle your application load properly.*

![](https://cdn-images-1.medium.com/max/5732/1*SC40Yg8Pcqx6FQw2pQScQw.png)

* *Linux/Windows are commonly run a flavor of Operating System*

*Some Key Terms*

* ***Amazon Machine Image(AMI)**: We can think of it as an operating system or preconfigured package(template) require to launch an EC2 instance*

*This is the first screen we usually see when we try to launch an instance*

![](https://cdn-images-1.medium.com/max/5728/1*uWSIBhAmlXDihb26WSgEJw.png)

*AMI has two virtualization type*

* ***Hardware Virtual Machine(HVM)**: Under this OS runs directly on top of Virtual Machine without any modification, it’s similar to running on the bare metal**. **It also takes advantage of hardware extension that provides fast access to the underlying hardware on the host system.*

*To find out If CPU Support **Intel VT or AMD-V Virtualization Support***

* *secure virtual machine(**svm**) for AMD*

* *virtual machine extension(**vmx**) for Intel CPU*

    ***# egrep -i — color “svm|vmx” /proc/cpuinfo***

![](https://cdn-images-1.medium.com/max/5748/1*-QQafKXKwFv35uOQqlX1Ww.png)

*Another way to check is via **lscpu** command*

    ***#** lscpu*

    *Architecture:          x86_64*

    *CPU op-mode(s):        32-bit, 64-bit*

    *Byte Order:            Little Endian*

    *CPU(s):                36*

    *On-line CPU(s) list:   0-35*

    *Thread(s) per core:    1*

    *Core(s) per socket:    18*

    *Socket(s):             2*

    *NUMA node(s):          2*

    *Vendor ID:             GenuineIntel*

    *CPU family:            6*

    *Model:                 63*

    *Model name:            Intel(R) Xeon(R) CPU E5-2699 v3 @ 2.30GHz*

    *Stepping:              2*

    *CPU MHz:               2297.377*

    *BogoMIPS:              4594.20*

    ***Virtualization:        VT-x***

    *L1d cache:             32K*

    *L1i cache:             32K*

    *L2 cache:              256K*

    *L3 cache:              46080K*

    *NUMA node0 CPU(s):     0-8,18-26*

    *NUMA node1 CPU(s):     9-17,27-35*

* ***Paravirtual Machine(PVM):** In case if your hardware doesn’t support virtualization(verified under /proc/cpuinfo and lscpu) we can use PV AMI. In the past PV guests had better performance than HVM but because of enhancement in HVM Virtualization and the availability of PV drivers in HVM AMI this is no longer true.*

*NOTE: Nowadays most of **Amazon AMI is HVM based***

![](https://cdn-images-1.medium.com/max/2880/1*Xd9Nv9ut2SUwSuHBUkoZ-A.png)

* ***Instance Type**: Instance types comprise varying combinations of CPU, memory, storage, and networking capacity and give us the flexibility to choose the appropriate mix of resources for your applications*

* ***Storage**: Generally comes with two options*

1. ***Elastic Block Storage(EBS)**: Provide Persistent Storage and they are network attached storage.*

* *They can only be attached to one EC2 instance at a time*

* *The Main benefit is they can be backed up into a snapshot, which we can use later to restore into a new EBS volume.*

*It’s is very important to choose an instance depending upon the I/O that application is going to perform whether it’s going to perform small or higher Input/Output(I/O) read/write.*

*That why in some case even with provisioned IOPS we will get the same performance and in those cases, we need to choose EBS optimized instance which prioritizes EBS traffic(Figure below shows some EBS Optimized instances)*

*NOTE: AWS measure IOPS in 256KB chunks(so 768KB is equivalent to 3 IOPS)*

![](https://cdn-images-1.medium.com/max/5676/1*2ELIIXQixuz_lB8o5snhOw.png)

*Types of EBS Volumes*

1. *General Purpose SSD*

1. *Provisioned IOPS*

1. *Magnetic*

![](https://cdn-images-1.medium.com/max/2000/1*06j6Lo0XA4gj0AV2eXWbbQ.png)

![](https://cdn-images-1.medium.com/max/5604/1*iovSg1AHGYfpJAMOI_G8dA.png)

*2.** Ephemeral Storage**: These are physically attached to the host computer that is running the instance. Data on the volume only exists for the duration of the life of the instance*

* *Stopped/Shutdown: Data erased*

* *Rebooted: Data remained*

***Buying Options in EC2 Cloud***

*On the right-hand side of the EC2 dashboard, you will see the different AWS Instances*

![](https://cdn-images-1.medium.com/max/2000/1*-tfdPmroqKYwq3RxDTHsow.png)

* ***On-Demand**: As the name suggests we can provision/terminate instance any time(on-demand).*

1. *The most important aspect is billed by the hour(when the instance is running)*

1. *Most expensive purchasing option but the most flexible one*

* ***Reserved**: Allow us to purchase an instance for a set time period(1 to 3 year)*

1. *Less expensive as compare to on-demand but once purchased we are completely responsible for the entire price regardless of how often we use*

1. *Comes with different pay options(**upfront,partial-upfront,no upfront**)*

* ***Spot**: We can even bid for an instance and we can only pay when the spot price is equal to or below our bid price*

1. *Generally used in the non-production scenario as the instance will be automatically terminate when the spot price is equal to or less than our bid*

1. *It allows Amazon to sell the unused instance for a short amount of time at a substantial discount prize*

***Elastic IP Address(EIP):** It’s a public IPv4 address designed for dynamic cloud mapping. We can attach our instance to EIP that was created only with a private IP address. The advantage of EIP that in the case of any instance failure we can detach it and attach it to the new instance.*

![](https://cdn-images-1.medium.com/max/4892/1*-djZSYCdNr4Xo8jllnBRtw.png)

***User-Data: **Using user-data we can add our own custom script during EC2 instance creation.*

![](https://cdn-images-1.medium.com/max/3828/1*2PJ1Mo069pgbR9P2dsMkJw.png)

*Now to view User-data and Instance Meta-Data use*

    *# curl [http://169.254.169.254/latest/meta-data/](http://169.254.169.254/latest/meta-data/)
    # curl [http://169.254.169.254/latest/user-data/](http://169.254.169.254/latest/user-data/)*

*For eg:*

    *# curl [http://169.254.169.254/latest/meta-data/](http://169.254.169.254/latest/meta-data/)*

    *ami-id*

    *ami-launch-index*

    *ami-manifest-path*

    *block-device-mapping/*

    *hostname*

    *instance-action*

    *instance-id*

    *instance-type*

    *local-hostname*

    *local-ipv4*

    *mac*

    *metrics/*

    *network/*

    *placement/*

    *profile*

    *public-hostname*

    *public-ipv4*

    *public-keys/*

    *reservation-id*

    *security-groups*

    *#To get specific information(eg:ami)
    # curl [http://169.254.169.254/latest/meta-data//ami-id](http://169.254.169.254/latest/meta-data//ami-id)*

    *ami-6f68cf0f*

***EC2 Key Pairs***

*EC2 key pair has two parts*

* ***Public Key**: AWS store the public key*

* ***Private Key**: As an administrator, we are responsible for Private Key(it’s available in the form of pem key and make sure permission must be set to **400**)*

*We use this keypair to login to an instance(Linux) via ssh*

    *ssh -i <keypair.pem> ec2-user@aws-hostname*

*At the time of instance creation, it will ask you*

* *Choose an existing key pair*

* *Create a new key pair*

![](https://cdn-images-1.medium.com/max/2784/1*6eH3gcblmQq6DlvPZKonxA.png)

***EBS Snapshots***

*Snapshots are point-in-time backups of EBS volumes that are stored in S3 and incremental in nature.*

* *It only stores the change since the most recent backup and thus helps us to reduce cost*

* *Using snapshot we can restore the EBS volumes and even in cases where the original snapshot is deleted data is still available in all the other snapshots*

* *While taking a snapshot it degrades the performance of EBS volumes, so it should be taken during the non-peak hour*

*To create a snapshot, just click on the left bar of EC2 console*

![](https://cdn-images-1.medium.com/max/5760/1*3Y1ycpUpiG5sEh5UBi2HYQ.png)

*Now as mentioned above we can use these snapshot to restore EBS volumes. As you can see we can in the pic below(just right-click on that snapshot)*

* *Create Volume*

* *Create Image*

* *Modify Permission*

![](https://cdn-images-1.medium.com/max/2880/1*hUSuoZJ_dgN3SfSvTkg9cw.png)

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
