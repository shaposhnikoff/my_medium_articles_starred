
# 100 Days of DevOps — Day 34- Terraform Pipeline using Jenkins

Welcome to Day 34 of 100 Days of DevOps, Focus for today is Terraform Pipeline using Jenkins.

### *What is the Pipeline?*

*A pipeline is typically referred to as a part of Continuous Integration and it’s mostly used to merge developer changes into code mainline.*

***Why do we need a pipeline?***

1. *Test changes before pushing to Production*

1. *Another pair of eyes in terms of approval*

1. *Logging to see who and when someone pushed the changes*

1. *Separate workspace/tfstate for Development and Production Environment*

### *Pipeline Architecture*

![](https://cdn-images-1.medium.com/max/2656/1*i8mcxAZfcSkZ_88CGAA6pw.jpeg)

## *How Jenkins Terraform Pipeline Works?*

* *User push their code changes to GitHub*

* *Code change trigger a Git Webhooks which triggers the terraform pipeline*
> *Setting up Jenkins*

* *Install Java*

    *# yum -y install java-1.8.0-openjdk.x86_64*

* *Download the latest version of Jenkins rpm*

    *# wget https://pkg.jenkins.io/redhat-stable/jenkins-2.164.1-1.1.noarch.rpm*
[**RedHat Repository for Jenkins**
*You will need to explicitly install a Java runtime environment, because Oracle's Java RPMs are incorrect and fail to…*pkg.jenkins.io](https://pkg.jenkins.io/redhat-stable/)

* *Install the Jenkins rpm*

    *# rpm -ivh jenkins-2.164.1-1.1.noarch.rpm*

    *warning: jenkins-2.164.1-1.1.noarch.rpm: Header V4 DSA/SHA1 Signature, key ID d50582e6: NOKEY*

    *Preparing...                          ################################# [100%]*

    *Updating / installing...*

    *1:jenkins-2.164.1-1.1              ################################# [100%]*

* *Start Jenkins*

    *# systemctl status jenkins*

    ***●** jenkins.service - LSB: Jenkins Automation Server*

    *Loaded: loaded (/etc/rc.d/init.d/jenkins; bad; vendor preset: disabled)*

    *Active: **active (running)** since Sat 2019-03-16 18:59:54 UTC; 5s ago*

    *Docs: man:systemd-sysv-generator(8)*

    *Process: 4241 ExecStart=/etc/rc.d/init.d/jenkins start (code=exited, status=0/SUCCESS)*

    *CGroup: /system.slice/jenkins.service*

    *└─4260 /etc/alternatives/java -Dcom.sun.akuma.Daemon=daemonized -Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins -jar /usr/lib/jenkins/jenkins.war --logfile=/var/log/jenkins/jenkins....*

    *Mar 16 18:59:53 ip-172-31-29-138.us-west-2.compute.internal systemd[1]: Starting LSB: Jenkins Automation Server...*

    *Mar 16 18:59:53 ip-172-31-29-138.us-west-2.compute.internal runuser[4246]: pam_unix(runuser:session): session opened for user jenkins by (uid=0)*

    *Mar 16 18:59:54 ip-172-31-29-138.us-west-2.compute.internal jenkins[4241]: Starting Jenkins [  OK  ]*

    *Mar 16 18:59:54 ip-172-31-29-138.us-west-2.compute.internal systemd[1]: Started LSB: Jenkins Automation Server.*

* *Browse your Jenkins URL on port 8080, you will see something like this*

![](https://cdn-images-1.medium.com/max/3996/1*AIVUfNa2zOvtmGbdfgukUg.png)

* *Copy and Paste the password(from below location)and then Select the installed Plugins*

    *# cat /var/lib/jenkins/secrets/initialAdminPassword*

* *Once Jenkins is up and running, Click on Manage Jenkins and then Manage Plugins*

![](https://cdn-images-1.medium.com/max/5064/1*5NWTnor7TxJNJ-R-GZyJLw.png)

* *Search for Terraform under Available tab, Select the Plugin and then Click on Install without restart*

![](https://cdn-images-1.medium.com/max/4420/1*552bl02uQgTZksgqDJD72w.png)

* *Go to Manage Jenkins but this Global Tool Configuration*

![](https://cdn-images-1.medium.com/max/4404/1*nZU09urzI9VLlSRyv6B3HQ.png)

    ** Name: This version must be consistent with the version of Jenkins we are going to refer in the Jenkins file
    * Version: Use the same version*

* *Again go back to the main Jenkins console, click on Credentials and then global*

![](https://cdn-images-1.medium.com/max/5724/1*ihQpaQPFjIanB1vvg1zwdA.png)

* *Add Credentials and then Secret text*

![](https://cdn-images-1.medium.com/max/5712/1*_6IorYTi-fj6W_H1PfqhRQ.png)

* *Here we want to add AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY in Secret Text so that when Jenkins job update doesn’t show AWS Key and Secret in plain text*

![](https://cdn-images-1.medium.com/max/4420/1*VZz3Ppklqg41P6x6KvcE8A.png)

![](https://cdn-images-1.medium.com/max/4376/1*zD-3ISJyE1345jfK398exg.png)

* *New things that I am introducing here is the Jenkinsfile*

<iframe src="https://medium.com/media/3e26b8252efc9bb256e08c0696a09056" frameborder=0></iframe>

    ** On the top of the pipeline, I am defining any agent that is available to run the pipeline
    * Next step I am defining tools that is required to run the configuration and in this case terraform with version number that we setup during the global configuration.*

    ** Next step we need some parameter that is required to run the pipeline, in this case workspace, Right now the only thing you need to know about workspace is that each workspace maintain seperate tfstate file, in this case I am putting development as default workspace
    *
[**State: Workspaces - Terraform by HashiCorp**
*Workspaces allow the use of multiple states with a single configuration directory.*www.terraform.io](https://www.terraform.io/docs/state/workspaces.html)

![](https://cdn-images-1.medium.com/max/4408/1*U1ccT6NVjDIpSdW5aR1Fbg.png)

    ** Next step is to setting up some environment variable 
    1: Where to find terraform
    2: Setting up TF_IN_AUTOMATION = "true" will make terraform don't spit too much information
    3: Access and Secret key to interact with AWS*

* *Various Stages of Terraform Pipeline*

![](https://cdn-images-1.medium.com/max/2516/1*c4O8hLyR7HZG0kpdqt2E7Q.png)
> *Terraform init*

*The first step is to check-in the code from GitHub Repo*
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/tree/master/jenkins-terraform-pipeline/ec2_pipeline)

    ** second step is setting input=false which means we if terraform need input and its not present fail the command
    * some extra command for informational purpose*
> *Terraform Format*

* *In this case, I am checking the file checked in to the git is properly formatted if not simply failed at that point*
> *Terraform Validate*

* *Simply check if the syntax of .tf file is proper or not*
> *Terraform Plan*

* *Try to create a new workspace and if its already present just select it(default workspace is development)*

* *Then I am running a simple plan command, providing access and secret key and storing the output in terraform.tfplan*

* *Next step is I am stashing the output generated by Previous command so that we can use it later*
> *Terraform Apply*

* *Here I am giving choice to the user whether he wants to apply this change or not, if he chooses Ready to Apply the config then terraform apply will run with the plan created earlier.*

![](https://cdn-images-1.medium.com/max/3196/1*uX06ULrGb0RmzE7MX0z1hg.png)

*OR*

![](https://cdn-images-1.medium.com/max/4356/1*KEoR2IpxiA6UwLlcASefBQ.png)

***Creating Jenkins Pipeline***

![](https://cdn-images-1.medium.com/max/4640/1*doXQ1maXWevLpglka92gLg.png)

***GitHub End***

![](https://cdn-images-1.medium.com/max/4544/1*gtEqN9dSfQDBCJimTfV5UA.png)

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
