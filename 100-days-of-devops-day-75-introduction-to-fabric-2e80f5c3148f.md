
# 100 Days of DevOps — Day 75- Introduction to Fabric

Welcome to Day 75 of 100 Days of DevOps, Focus for today is Introduction to Fabric

*What is Fabric?*

*As per official documentation “Fabric is a Python library and command-line tool for streamlining the use of SSH for application deployment or systems administration tasks”*

*Installing Fabric*

*Installation is pretty straightforward in case of Linux System(**pip install fabric**)*

*NOTE: pip is not a part of standard package library,so in case of RedHat/Centos system you first need to install epel package*

    *rpm -ivh [http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-9.noarch.rpm](http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-9.noarch.rpm)
    yum -y install python-pip*

    *pip install fabric
    Collecting fabric
    Installing collected packages: fabric
    Successfully installed fabric-1.13.1*

*Once Fabric is installed it provide bunch of useful command and features.Let’s explore few command*

    *[root@fabric-server ~]# fab -h*

    *Usage: fab [options] <command>[:arg1,arg2=val2,host=foo,hosts=’h1;h2',…] …*

* *Now let say I want to check disk-space utilization of Client(192.168.0.16)*

    *[root@fabric-server ~]# fab -H 192.168.0.16 -I — df -TH*

    *Initial value for env.password:*

    *[192.168.0.16] Executing task ‘<remainder>’*

    *[192.168.0.16] run: df -TH*

    *[192.168.0.16] out: Filesystem Type Size Used Avail Use% Mounted on*

    *[192.168.0.16] out: /dev/mapper/cl-root xfs 20G 1.4G 19G 7% /*

    *[192.168.0.16] out: devtmpfs devtmpfs 510M 0 510M 0% /dev*

    *[192.168.0.16] out: tmpfs tmpfs 521M 0 521M 0% /dev/shm*

    *[192.168.0.16] out: tmpfs tmpfs 521M 7.0M 514M 2% /run*

    *[192.168.0.16] out: tmpfs tmpfs 521M 0 521M 0% /sys/fs/cgroup*

    *[192.168.0.16] out: /dev/sda1 xfs 102M 98M 3.7M 97% /boot*

    *[192.168.0.16] out: tmpfs tmpfs 105M 0 105M 0% /run/user/0*

    *[192.168.0.16] out:*

    *Done.*

    *Disconnecting from 192.168.0.16… done.*

*where*

    ***-I, — initial-password-prompt***

    *Force password prompt up-front*

    ***-H HOSTS, — hosts=HOSTS***

    *comma-separated list of hosts to operate on*

* *In case of multiple hosts, we can pass hostname/IP via comma separated value*

    *[root@fabric-server ~]# fab -H **192.168.0.15,192.168.0.16** -I — df -TH*

    *Initial value for env.password:*

    *[192.168.0.15] Executing task ‘<remainder>’*

    *[192.168.0.15] run: df -TH*

    *[192.168.0.15] out: Filesystem Type Size Used Avail Use% Mounted on*

    *[192.168.0.15] out: /dev/mapper/cl-root xfs 20G 1.4G 19G 7% /*

    *[192.168.0.15] out: devtmpfs devtmpfs 510M 0 510M 0% /dev*

    *[192.168.0.15] out: tmpfs tmpfs 521M 0 521M 0% /dev/shm*

    *[192.168.0.15] out: tmpfs tmpfs 521M 7.0M 514M 2% /run*

    *[192.168.0.15] out: tmpfs tmpfs 521M 0 521M 0% /sys/fs/cgroup*

    *[192.168.0.15] out: /dev/sda1 xfs 102M 98M 3.7M 97% /boot*

    *[192.168.0.15] out: tmpfs tmpfs 105M 0 105M 0% /run/user/0*

    *[192.168.0.15] out:*

    *[192.168.0.16] Executing task ‘<remainder>’*

    *[192.168.0.16] run: df -TH*

    *[192.168.0.16] out: Filesystem Type Size Used Avail Use% Mounted on*

    *[192.168.0.16] out: /dev/mapper/cl-root xfs 20G 1.4G 19G 7% /*

    *[192.168.0.16] out: devtmpfs devtmpfs 510M 0 510M 0% /dev*

    *[192.168.0.16] out: tmpfs tmpfs 521M 0 521M 0% /dev/shm*

    *[192.168.0.16] out: tmpfs tmpfs 521M 7.0M 514M 2% /run*

    *[192.168.0.16] out: tmpfs tmpfs 521M 0 521M 0% /sys/fs/cgroup*

    *[192.168.0.16] out: /dev/sda1 xfs 102M 98M 3.7M 97% /boot*

    *[192.168.0.16] out: tmpfs tmpfs 105M 0 105M 0% /run/user/0*

    *[192.168.0.16] out:*

    *Done.*

    *Disconnecting from 192.168.0.16… done.*

    *Disconnecting from 192.168.0.15… done.*

* *In both the above cases I need to pass password at the prompt,but in case of automation we need some way so that we don’t need to pass password each time*

*So the way to do it, is to set password variable at shell prompt*

    *export password=”test12"*

*Now we can run the same command without prompting for password,instead of passing -I we can pass(-p $password)*

    *[root@fabric-server ~]# fab -H 192.168.0.15,192.168.0.16 **-p $password** — df -TH*

    *[192.168.0.15] Executing task ‘<remainder>’*

    *[192.168.0.15] run: df -TH*

    *[192.168.0.15] out: Filesystem Type Size Used Avail Use% Mounted on*

    *[192.168.0.15] out: /dev/mapper/cl-root xfs 20G 1.4G 19G 7% /*

    *[192.168.0.15] out: devtmpfs devtmpfs 510M 0 510M 0% /dev*

    *[192.168.0.15] out: tmpfs tmpfs 521M 0 521M 0% /dev/shm*

    *[192.168.0.15] out: tmpfs tmpfs 521M 7.0M 514M 2% /run*

    *[192.168.0.15] out: tmpfs tmpfs 521M 0 521M 0% /sys/fs/cgroup*

    *[192.168.0.15] out: /dev/sda1 xfs 102M 98M 3.7M 97% /boot*

    *[192.168.0.15] out: tmpfs tmpfs 105M 0 105M 0% /run/user/0*

    *[192.168.0.15] out:*

    *[192.168.0.16] Executing task ‘<remainder>’*

    *[192.168.0.16] run: df -TH*

    *[192.168.0.16] out: Filesystem Type Size Used Avail Use% Mounted on*

    *[192.168.0.16] out: /dev/mapper/cl-root xfs 20G 1.4G 19G 7% /*

    *[192.168.0.16] out: devtmpfs devtmpfs 510M 0 510M 0% /dev*

    *[192.168.0.16] out: tmpfs tmpfs 521M 0 521M 0% /dev/shm*

    *[192.168.0.16] out: tmpfs tmpfs 521M 7.0M 514M 2% /run*

    *[192.168.0.16] out: tmpfs tmpfs 521M 0 521M 0% /sys/fs/cgroup*

    *[192.168.0.16] out: /dev/sda1 xfs 102M 98M 3.7M 97% /boot*

    *[192.168.0.16] out: tmpfs tmpfs 105M 0 105M 0% /run/user/0*

    *[192.168.0.16] out:*

    *Done.*

    *Disconnecting from 192.168.0.16… done.*

    *Disconnecting from 192.168.0.15… done.*

* *All these command executed linearly i.e one after the other, in case we need make these threads run in parallel use (-P)*

    *[root@fabric-server ~]# fab -H 192.168.0.15,192.168.0.16 -p $password -P — df -TH*

    *[192.168.0.15] Executing task ‘<remainder>’*

    *[192.168.0.16] Executing task ‘<remainder>’*

    *[192.168.0.16] run: df -TH*

    *[192.168.0.15] run: df -TH*

    *[192.168.0.16] out: Filesystem Type Size Used Avail Use% Mounted on*

    *[192.168.0.16] out: /dev/mapper/cl-root xfs 20G 1.4G 19G 7% /*

    *[192.168.0.16] out: devtmpfs devtmpfs 510M 0 510M 0% /dev*

    *[192.168.0.16] out: tmpfs tmpfs 521M 0 521M 0% /dev/shm*

    *[192.168.0.16] out: tmpfs tmpfs 521M 7.0M 514M 2% /run*

    *[192.168.0.16] out: tmpfs tmpfs 521M 0 521M 0% /sys/fs/cgroup*

    *[192.168.0.16] out: /dev/sda1 xfs 102M 98M 3.7M 97% /boot*

    *[192.168.0.16] out: tmpfs tmpfs 105M 0 105M 0% /run/user/0*

    *[192.168.0.16] out:*

    *[192.168.0.15] out: Filesystem Type Size Used Avail Use% Mounted on*

    *[192.168.0.15] out: /dev/mapper/cl-root xfs 20G 1.4G 19G 7% /*

    *[192.168.0.15] out: devtmpfs devtmpfs 510M 0 510M 0% /dev*

    *[192.168.0.15] out: tmpfs tmpfs 521M 13k 521M 1% /dev/shm*

    *[192.168.0.15] out: tmpfs tmpfs 521M 7.0M 514M 2% /run*

    *[192.168.0.15] out: tmpfs tmpfs 521M 0 521M 0% /sys/fs/cgroup*

    *[192.168.0.15] out: /dev/sda1 xfs 102M 98M 3.7M 97% /boot*

    *[192.168.0.15] out: tmpfs tmpfs 105M 0 105M 0% /run/user/0*

    *[192.168.0.15] out:*

    *Done.*

* *Now to control how many thread to fork when run in parallel(-z number of concurrent processes to use in parallel mode)*

    *[root@fabric-server ~]# fab -H 192.168.0.15,192.168.0.16 -p $password -z 1 — df -TH*

    *[192.168.0.15] Executing task ‘<remainder>’*

    *[192.168.0.15] run: df -TH*

    *[192.168.0.15] out: Filesystem Type Size Used Avail Use% Mounted on*

    *[192.168.0.15] out: /dev/mapper/cl-root xfs 20G 1.4G 19G 7% /*

    *[192.168.0.15] out: devtmpfs devtmpfs 510M 0 510M 0% /dev*

    *[192.168.0.15] out: tmpfs tmpfs 521M 0 521M 0% /dev/shm*

    *[192.168.0.15] out: tmpfs tmpfs 521M 7.0M 514M 2% /run*

    *[192.168.0.15] out: tmpfs tmpfs 521M 0 521M 0% /sys/fs/cgroup*

    *[192.168.0.15] out: /dev/sda1 xfs 102M 98M 3.7M 97% /boot*

    *[192.168.0.15] out: tmpfs tmpfs 105M 0 105M 0% /run/user/0*

    *[192.168.0.15] out:*

    *[192.168.0.16] Executing task ‘<remainder>’*

    *[192.168.0.16] run: df -TH*

    *[192.168.0.16] out: Filesystem Type Size Used Avail Use% Mounted on*

    *[192.168.0.16] out: /dev/mapper/cl-root xfs 20G 1.4G 19G 7% /*

    *[192.168.0.16] out: devtmpfs devtmpfs 510M 0 510M 0% /dev*

    *[192.168.0.16] out: tmpfs tmpfs 521M 0 521M 0% /dev/shm*

    *[192.168.0.16] out: tmpfs tmpfs 521M 7.0M 514M 2% /run*

    *[192.168.0.16] out: tmpfs tmpfs 521M 0 521M 0% /sys/fs/cgroup*

    *[192.168.0.16] out: /dev/sda1 xfs 102M 98M 3.7M 97% /boot*

    *[192.168.0.16] out: tmpfs tmpfs 105M 0 105M 0% /run/user/0*

    *[192.168.0.16] out:*

    *Done.*

    *Disconnecting from 192.168.0.16… done.*

    *Disconnecting from 192.168.0.15… done.*

*To save the output to a file*

    *[root@fabric-server ~]# fab -H 192.168.0.15,192.168.0.16 -p $password -z 1 — df -TH >> /tmp/faboutput*

* *Now a days most of the companies running there infrastructure in cloud,in case of AWS cloud we need to modify our command little bit*

    *fab -H ec-<ip add>.us-west-2.compute.amazonaws.com -u ec2-user -i test.pem — df -TH*

    *[ec-<ip add>.us-west-2.compute.amazonaws.com] Executing task ‘<remainder>’*

    *[ec-<ip add>.us-west-2.compute.amazonaws.com] run: df -TH*

    *[ec-<ip add>.us-west-2.compute.amazonaws.com] out: Filesystem Type Size Used Avail Use% Mounted on*

    *[ec-<ip add>.us-west-2.compute.amazonaws.com] out: /dev/xvda2 xfs 11G 1.9G 9.0G 17% /*

    *[ec-<ip add>.us-west-2.compute.amazonaws.com] out: devtmpfs devtmpfs 500M 0 500M 0% /dev*

    *[ec-<ip add>.us-west-2.compute.amazonaws.com] out: tmpfs tmpfs 520M 0 520M 0% /dev/shm*

    *[ec-<ip add>.us-west-2.compute.amazonaws.com] out: tmpfs tmpfs 520M 66M 455M 13% /run*

    *[ec-<ip add>.us-west-2.compute.amazonaws.com] out: tmpfs tmpfs 520M 0 520M 0% /sys/fs/cgroup*

    *[ec-<ip add>.us-west-2.compute.amazonaws.com] out: tmpfs tmpfs 104M 0 104M 0% /run/user/1000*

    *[ec-<ip add>.us-west-2.compute.amazonaws.com] out:*

    *Done.*

    *Disconnecting from ec-<ip add>.us-west-2.compute.amazonaws.com… done.*

*where*

    *i PATH path to SSH private key file. May be repeated*

    *-u USER, — user=USER username to use when connecting to remote hosts*

* *So far all these commands we are executing on the command line,in order to automate the whole process we need to create fabfile,which is a simple python script(here you can see we combine both memory as well as disk space utilization)*

    *#Fabfile to:*

    *# — Run the command on remote system(s)*

    *# — Check the memory and disk space*

    *#Import Fabric’s API Module*

    *from fabric.api import **

    *env.hosts = [‘192.168.0.15’,*

    *‘192.168.0.16’]*

    *# Set the username*

    *env.user = “root”*

    *# Set the password[NOT RECOMMENDED]”*

    *env.password =“test12”*

    *def memory_check():*

    *“””*

    *Check Memory usage on remote system*

    *“””*

    *run(“free -m”)*

    *def disk_check():*

    *“””*

    *Check Disk space usage on remote system*

    *“””*

    *run(“df -TH”)*

    *def tesenv():*

    *memory_check()*

    *disk_check*

*Now to verify it*

    *[root@fabric-server ~]# fab -l*

    *Available commands:*

    *disk_check*

    *memory_check*

    *testenv*

*To run it*

    *[root@fabric-server ~]# fab testenv*

    *[192.168.0.17] Executing task ‘testenv’*

    *[192.168.0.17] run: free -m*

    *[192.168.0.17] out: total used free shared buff/cache available*

    *[192.168.0.17] out: Mem: 992 79 701 6 211 762*

    *[192.168.0.17] out: Swap: 499 0 499*

    *[192.168.0.17] out:*

    *[192.168.0.17] run: df -TH*

    *[192.168.0.17] out: Filesystem Type Size Used Avail Use% Mounted on*

    *[192.168.0.17] out: /dev/mapper/cl-root xfs 20G 1.4G 19G 7% /*

    *[192.168.0.17] out: devtmpfs devtmpfs 510M 0 510M 0% /dev*

    *[192.168.0.17] out: tmpfs tmpfs 521M 0 521M 0% /dev/shm*

    *[192.168.0.17] out: tmpfs tmpfs 521M 7.0M 514M 2% /run*

    *[192.168.0.17] out: tmpfs tmpfs 521M 0 521M 0% /sys/fs/cgroup*

    *[192.168.0.17] out: /dev/sda1 xfs 102M 98M 3.7M 97% /boot*

    *[192.168.0.17] out: tmpfs tmpfs 105M 0 105M 0% /run/user/0*

    *[192.168.0.17] out:*

    *[192.168.0.18] Executing task ‘testenv’*

    *[192.168.0.18] run: free -m*

    *[192.168.0.18] out: total used free shared buff/cache available*

    *[192.168.0.18] out: Mem: 992 106 477 6 409 728*

    *[192.168.0.18] out: Swap: 499 0 499*

    *[192.168.0.18] out:*

    *[192.168.0.18] run: df -TH*

    *[192.168.0.18] out: Filesystem Type Size Used Avail Use% Mounted on*

    *[192.168.0.18] out: /dev/mapper/cl-root xfs 20G 1.4G 19G 7% /*

    *[192.168.0.18] out: devtmpfs devtmpfs 510M 0 510M 0% /dev*

    *[192.168.0.18] out: tmpfs tmpfs 521M 0 521M 0% /dev/shm*

    *[192.168.0.18] out: tmpfs tmpfs 521M 7.0M 514M 2% /run*

    *[192.168.0.18] out: tmpfs tmpfs 521M 0 521M 0% /sys/fs/cgroup*

    *[192.168.0.18] out: /dev/sda1 xfs 102M 98M 3.7M 97% /boot*

    *[192.168.0.18] out: tmpfs tmpfs 105M 0 105M 0% /run/user/0*

    *[192.168.0.18] out:*

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
[**100 Days of DevOps — Day 74- Introduction to GIT**
*Welcome to Day 74 of 100 Days of DevOps, Focus for today is Introduction to GIT*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-74-introduction-to-git-9374bafb08b6)
