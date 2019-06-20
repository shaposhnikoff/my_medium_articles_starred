
# 100 Days of DevOps — Day 61-Jenkins Agent Node

Welcome to Day 61 of 100 Days of DevOps, Focus for today is Jenkins Agent Node

*Yesterday I showed you how to set up Jenkins Server*
[**100 Days of DevOps — Day 60-Introduction to Jenkins**
*Welcome to Day 60 of 100 Days of DevOps, Focus for today is Jenkins*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-60-introduction-to-jenkins-5afc0f700335)

*The above setup works great in a small environment but as your team grows you need to scale up your Jenkins and in those situations, you need Master/Agent setup. In Master/Agent setup, Master acts as a control node and Jenkins with the agent installed to take care of the execution of all the jobs.*
> *Requirement*

* *Jenkins Master*

* *Agent Node*

***NOTE:** I am performing all the steps on Centos7*
> Step1: Install the necessary package on the agent node

    *yum -y install java-1.8.0-openjdk git*

* *In this case, I am installing java and git*

* *You don’t Jenkins rpm package on the agent machine*
> Step2: Create user and copy the ssh key from Master to agent node

    *# useradd jenkins*

    *visudo*

    ***jenkins ALL=(ALL)       NOPASSWD: ALL***

    *# mkdir jenkins_build*

* *Then copy the key generated on Day60 from master to agent node*

    *$ ssh-copy-id jenkins@172.31.29.138*

    *# To test the connectivity*

    *$ ssh jenkins@172.31.29.138*

    *Last login: Fri Apr 12 04:31:49 2019*
> *Step3: Add the agent from Jenkins UI*

* *On the Jenkins UI, Click on Manage Jenkins and then Manage Nodes*

![](https://cdn-images-1.medium.com/max/3616/1*qrd22D6K8qidtiQILGgbxQ.png)

* *Click on the New Node*

![](https://cdn-images-1.medium.com/max/5724/1*pWrU_GBEJXO7ASTLXnos4w.png)

![](https://cdn-images-1.medium.com/max/4872/1*kSq_lAlAu9SpPtHER35ISg.png)

![](https://cdn-images-1.medium.com/max/4860/1*JfSRg9VLl22GgkCTVohUiw.png)

    ** Name: Give your agent some name
    * # of executors: The maximum number of concurrent builds that Jenkins may perform on this node.
    * Remote root directory:An agent needs to have a directory dedicated to Jenkins. Specify the path to this directory on the agent. It is best to use   an absolute path, such as /var/jenkins or c:\jenkins.   This should be a path local to the agent machine. There is no need for this path to be visible from the master(/var/lib/jenkins/jenkins_build) created in earlier step
    * Usage: Controls how Jenkins schedules builds on this node.         **       Use this node as much as possible - **This is the default setting.       
    In this mode, Jenkins uses this node freely. Whenever there is a build that can be done by using this node, Jenkins will use it.
    * Credentials: Select the credentials to be used for logging in to the remote host.
    * Host: Enter the hostname or IP address of your agent node in the Host field.*

![](https://cdn-images-1.medium.com/max/4340/1*7uZ_QkzegeNxTU7-UchMaA.png)

* *Check if everything works*

![](https://cdn-images-1.medium.com/max/4812/1*hZyPt6XQO-l2vD41zYxghg.png)

* *To use the slave node*

![](https://cdn-images-1.medium.com/max/2404/1*VbeCbbAtPNLfePLM7abjkA.png)

![](https://cdn-images-1.medium.com/max/3968/1*QHgdlpcSzLkT5Db9N2fQeQ.png)

    ** *

![](https://cdn-images-1.medium.com/max/3876/1*H9iyPZw85m3YHWVvDocjbw.png)

![](https://cdn-images-1.medium.com/max/3944/1*T3R9nbTzJ6z6vqParf11Fw.png)

    ** Restrict where this project can be run: Select the label we created in earlier step(jenkins_slave) this will make sure build run on agent host*

* *Run the job and you will see output like this*

![](https://cdn-images-1.medium.com/max/2996/1*qf7a9gAkrFvT0s2-Z5J0Eg.png)

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
