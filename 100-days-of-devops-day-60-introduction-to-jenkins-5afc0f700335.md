
# 100 Days of DevOps — Day 60-Introduction to Jenkins

Welcome to Day 60 of 100 Days of DevOps, Focus for today is Jenkins

![](https://cdn-images-1.medium.com/max/2000/1*kfCBDTE6WJP_WVO12x-crw.png)

*Jenkins is an open source automation server written in Java.*

* *As Jenkins is written in Java you must need to install java to run Jenkins*

    *#yum -y install java-1.8.0-**openjdk**.x86_64*
[**Jenkins**
*Jenkins - an open source automation server which enables developers around the world to reliably build, test, and…*jenkins.io](https://jenkins.io/)

* *To use this repository, run the following command:*

    *sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
    sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key*
[**RedHat Repository for Jenkins**
*You will need to explicitly install a Java runtime environment, because Oracle's Java RPMs are incorrect and fail to…*pkg.jenkins.io](https://pkg.jenkins.io/redhat/)
> Jenkins Installation

    *# yum -y install jenkins*

* *Start Jenkins Service*

    *# systemctl start jenkins*

* *Make sure it's available/enable after reboot*

    *systemctl enable jenkins*

* *Check the status of Jenkins Service*

    *# systemctl status jenkins*

    ***●** jenkins.service - LSB: Jenkins Automation Server*

    *Loaded: loaded (/etc/rc.d/init.d/jenkins; bad; vendor preset: disabled)*

    *Active: **active (running)** since Thu 2019-04-11 23:15:26 UTC; 10s ago*

    *Docs: man:systemd-sysv-generator(8)*

    *CGroup: /system.slice/jenkins.service*

    *└─2310 /etc/alternatives/java -Dcom.sun.akuma.Daemon=daemonized -Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins -jar /usr/lib/jenkins/jenkins.war --logfile=/var/log/jenkins/jenkins....*

    *Apr 11 23:15:25 ip-172-31-31-68.us-west-2.compute.internal systemd[1]: Starting LSB: Jenkins Automation Server...*

    *Apr 11 23:15:25 ip-172-31-31-68.us-west-2.compute.internal runuser[2296]: pam_unix(runuser:session): session opened for user jenkins by (uid=0)*

    *Apr 11 23:15:26 ip-172-31-31-68.us-west-2.compute.internal jenkins[2291]: Starting Jenkins [  OK  ]*

    *Apr 11 23:15:26 ip-172-31-31-68.us-west-2.compute.internal systemd[1]: Started LSB: Jenkins Automation Server.*

* *Now try to browse your Jenkins Server on http://<ip address>:8080*

![](https://cdn-images-1.medium.com/max/4016/1*c94wZ9r5a8I4lwZYJvTf4Q.png)

* *Cat the content of the file and paste it in the browser*

    *# cat /var/lib/jenkins/secrets/initialAdminPassword*

* *Install suggested plugins*

* *Create user and you are all set to go*

* *In some cases where you are running Jenkins in secure environment(where you have a port restriction, you will not be able to get to the instance), in these cases, we can run Jenkins behind Nginx which act as a reverse proxy*

    *# yum install nginx*

* *Go to the nginx configuration file /etc/nginx/nginx.conf*

<iframe src="https://medium.com/media/97d86296ed9b645eb0829107596ea03f" frameborder=0></iframe>

* *Here we are telling nginx(proxy_pass http://127.0.0.1), any connection request you are getting for root(/) on Port 80 forward it to 8080 port*

    *# Start Nginx 
    [root@ip-172-31-31-68 ~]# systemctl start nginx*

    *#Enable Nginx*

    *[root@ip-172-31-31-68 ~]# systemctl enable nginx*

    *Created symlink from /etc/systemd/system/multi-user.target.wants/nginx.service to /usr/lib/systemd/system/nginx.service.*

    *# Check Nginx Status*

    *[root@ip-172-31-31-68 ~]# systemctl status nginx*

    ***●** nginx.service - The nginx HTTP and reverse proxy server*

    *Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)*

    *Active: **active (running)** since Thu 2019-04-11 23:37:44 UTC; 10s ago*

    *Main PID: 3320 (nginx)*

    *CGroup: /system.slice/nginx.service*

    *├─3320 nginx: master process /usr/sbin/nginx*

    *└─3321 nginx: worker process*

    *Apr 11 23:37:44 ip-172-31-31-68.us-west-2.compute.internal systemd[1]: Starting The nginx HTTP and reverse proxy server...*

    *Apr 11 23:37:44 ip-172-31-31-68.us-west-2.compute.internal nginx[3314]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok*

    *Apr 11 23:37:44 ip-172-31-31-68.us-west-2.compute.internal nginx[3314]: nginx: configuration file /etc/nginx/nginx.conf test is successful*

    *Apr 11 23:37:44 ip-172-31-31-68.us-west-2.compute.internal systemd[1]: Failed to read PID from file /run/nginx.pid: Invalid argument*

    *Apr 11 23:37:44 ip-172-31-31-68.us-west-2.compute.internal systemd[1]: Started The nginx HTTP and reverse proxy server.*

* *Now there is a small issue, if you click on Manage Jenkins, you will see this reverse proxy error*

![](https://cdn-images-1.medium.com/max/5760/1*aeL3E92AHPiOUJb6AbZK9Q.png)

* *Go directly to this URL and copy paste all the content below proxy_pass*
[**Running Jenkins behind Nginx - Jenkins - Jenkins Wiki**
*If you are experiencing the following error when attempting to run long CLI commands in Jenkins > 2.80, and Jenkins is…*wiki.jenkins.io](https://wiki.jenkins.io/display/JENKINS/Running+Jenkins+behind+Nginx)

<iframe src="https://medium.com/media/c8534bed63542973f21a6bfa7ec85264" frameborder=0></iframe>
[**Jenkins behind an NGinX reverse proxy - Jenkins - Jenkins Wiki**
*However, you may want to access Jenkins from a folder on your main web server. This allows you to use the same TLS/SSL…*wiki.jenkins.io](https://wiki.jenkins.io/display/JENKINS/Jenkins+behind+an+NGinX+reverse+proxy)

* *Restart Nginx service*

    *# systemctl restart nginx*

* *As Jenkins is up and running now, let’s check a few more things*

    *# cat /etc/passwd|grep -i jenkins*

    ***jenkins**:x:997:994:**Jenkins** Automation Server:/var/lib/**jenkins**:/bin/false*

* *As you can see, default shell assign to Jenkins user is /bin/false which is too restrictive for us*

    *# su - jenkins*

    *Last failed login: Thu Apr 11 14:49:32 UTC 2019 from XXXXX on ssh:notty*

    *There was 1 failed login attempt since the last successful login.*

* */**bin**/**false** is just a binary that immediately exits, returning **false**, when it’s called, so when someone who has **false** as shell logs in, they’re immediately logged out when false exits.*

* *To change the shell from /bin/false to /bin/bash*

    *# usermod -s /bin/bash jenkins*

* *Next step is to generate the ssh key which I can use to login to slave nodes or to the master server itself without supplying the password*

    ***# su - jenkins***

    *Last login: Fri Apr 12 03:45:33 UTC 2019 on pts/0*

    ***-bash-4.2$ ssh-keygen***

    *Generating public/private rsa key pair.*

    *Enter file in which to save the key (/var/lib/jenkins/.ssh/id_rsa):*

    *Created directory '/var/lib/jenkins/.ssh'.*

    *Enter passphrase (empty for no passphrase):*

    *Enter same passphrase again:*

    *Your identification has been saved in /var/lib/jenkins/.ssh/id_rsa.*

    *Your public key has been saved in /var/lib/jenkins/.ssh/id_rsa.pub.*

    *The key fingerprint is:*

    *SHA256:LhzQwpGzgDttHiNAIV27P2LKlo7CkfdWjd5mOLku2rU jenkins@ip-172-31-31-68.us-west-2.compute.internal*

    *The key's randomart image is:*

    *+---[RSA 2048]----+*

    *|o=..o.           |*

    *|+ o.o+           |*

    *|.o .=o.          |*

    *|+ = .+           |*

    *| =.o. . S        |*

    *| o.. o = .       |*

    *|. o.+ B.=        |*

    *|.+oo.=.B.+       |*

    *|oo+.o.oE=        |*

    *+----[SHA256]-----+*

* *Then go to*

    *$ cd /var/lib/jenkins/.ssh/
    $ cat id_rsa.pub >> authorized_keys (make sure permission for this file must be set to 644)
    $ ssh localhost*

    *Last login: Fri Apr 12 03:58:54 2019 from localhost*

* *One more thing we can do is to allow Jenkins user to run any sudo command without entering the password*

    *#As root user run visudo command and add the below entry
    visudo
    **jenkins ALL=(ALL)       NOPASSWD: ALL***

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
