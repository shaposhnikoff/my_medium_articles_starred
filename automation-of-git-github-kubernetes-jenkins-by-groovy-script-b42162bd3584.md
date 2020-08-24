
# Automation of Git,Github,Kubernetes,Jenkins by Groovy Script

Automation of Git,Github,Kubernetes,Jenkins by Groovy Script

Integration of CI/CD Tool & Container Orchestration Tool

## Git:

Git is a free and open-source distributed version control system designed to handle everything from small to very large projects with speed and efficiency.

Git is easy to learn and has a tiny footprint with lightning-fast performance. It outclasses SCM tools like Subversion, CVS, Perforce, and Clear Case with features like cheap local branching, convenient staging areas, and multiple workflows.

## GitHub:

GitHub is a code hosting platform for collaboration and version control.

GitHub lets you (and others) work together on projects.

## Jenkins:

Jenkins is a self-contained, open-source automation server that can be used to automate all sorts of tasks related to building, testing, and delivering or deploying software.

Jenkins is a CI/CD tool.

Jenkins can be installed through native system packages, Docker, or even run standalone by any machine with a Java Runtime Environment (JRE) installed.

## **Groovy Script:**

Apache Groovy is an object-oriented and Java syntax compatible programming language built for the Java platform.

Groovy can be used as a scripting language for the Java platform. It is almost like a super version of Java which offers Java’s enterprise capabilities.

## Kubernetes:

Kubernetes (K8s) is an open-source system for automating deployment, scaling, and management of containerized applications.

It groups containers that make up an application into logical units for easy management and discovery. Kubernetes builds upon 15 years of experience of running production workloads at Google, combined with best-of-breed ideas and practices from the community.

## TASK: Build an Automation
> ***Problem Statement:***

Perform the task given below with the help of Jenkins coding file ( called as jenkinsfile approach ) and perform the with following phases:

*1. Create container image that’s has Jenkins installed using dockerfile Or You can use the Jenkins Server on RHEL 8/7
2. When we launch this image, it should automatically start Jenkins service in the container.
3. Create a job chain of Job1, Job2, Job3 and Job4 using build pipeline plugin in Jenkins
4. Job1: Pull the Github repo automatically when some developers push the repo to Github.
5. Job2 :
1. By looking at the code or program file, Jenkins should automatically start the respective language interpreter installed image container to deploy code on top of Kubernetes ( eg. If code is of PHP, then Jenkins should start the container that has PHP already installed )
2. Expose your pod so that testing team could perform the testing on the pod
3. Make the data to remain persistent ( If server collects some data like logs, other user information )
6. Job3: Test your app if it is working or not.
7. Job4: if the app is not working, then send an email to the developer with error messages and redeploy the application after code is being edited by the developer.*
> ***Solution:***

**STEP1: **In this step, I had created a container image of CI/CD Tool i.e. Jenkins by using **Dockerfile.**

![](https://cdn-images-1.medium.com/max/2000/0*vEUs70dtU5e7XX19)

![Jenkins Image](https://cdn-images-1.medium.com/max/3134/0*t8nTdcT9b36nfKth.png)*Jenkins Image*

![Jenkins Password](https://cdn-images-1.medium.com/max/2000/1*Mnr88dBqDaOH4BgQhV0xIQ.png)*Jenkins Password*

![Jenkins Admin User](https://cdn-images-1.medium.com/max/2000/1*AF9b3qBgwY0q85erWR4BJQ.png)*Jenkins Admin User*

![Jenkins Home Page](https://cdn-images-1.medium.com/max/3832/1*v2KcbzlLO4O4c0Ji3k58ig.png)*Jenkins Home Page*

**STEP2: **Write code in Git & push it to Github repo.

![Commit & Staging Area](https://cdn-images-1.medium.com/max/2034/1*kz5pJuouB4dnL6tn_zfEmA.png)*Commit & Staging Area*

**STEP3: **In this step, I will create a groovy script that will create 4 jobs with their own use cases and a build pipeline view for final view.

**JOB 1: **Loads the data from GitHub repo and copy it to my local system.

**JOB 2: **Launches a deployment of the image having PHP interpreter with that code which the developer has pushed to GitHub.

**JOB 3: **It checks that our server is working or not.

**JOB 4: **It checks that our server is working or not and if the server is not working it will send an email to the developer.

So for this at first we will create a seed job in which in build part we will select DSL Process option to run the script

![](https://cdn-images-1.medium.com/max/3786/1*Fut6i8jeBILBVF0iLf2bQw.png)

![](https://cdn-images-1.medium.com/max/3840/1*1Y9aeAstXdtVFl8u_MuNHQ.png)

![](https://cdn-images-1.medium.com/max/3824/1*nDebpt6dklpAgMMsjlMtwg.png)
> ***SCRIPT:***
> **job(‘task6_job1’){
description(‘It helps to pull the data from github when developer push their code to github by using some triggers.’)
scm{
github(‘ashwani7273/devopsal_task6’)
}
triggers {
scm(‘* * * * *’)
upstream(‘seed_job’,’SUCCESS’)
}
steps {
 shell(‘’’ if sudo cd /;ls | grep devopsal_task6
 then
 sudo cp -rvf * /devopsal_task6
 else
 sudo mkdir /devopsal_task6
 sudo cp -rvf * /devopsal_task6
 fi
 ‘’’)
 }
}
 
job(‘task6_job2’){
description(‘Run the environment to deploy code.’)**
> **triggers {
upstream(‘task6_job1’,’SUCCESS’)
}
steps{
 shell(‘’’ sudo cd /devopsal_task6
 if ls | grep “.php”
 then
 sudo kubectl delete all — all
 sudo kubectl create -f webDeployment.yml
 sudo kubectl create -f webService.yml
 sudo sleep 20
 status=$(sudo kubectl get pods -o ‘jsonpath={.items[0].metadata.name}’)
 sudo kubectl get all
 sudo kubectl cp /devopsal_task6/hello.php $status:/var/www/html/
 fi
 ‘’’)
}
}
 
job(‘task6_job3’){
description(‘App working status’)
triggers {
upstream(‘task6_job2’,’SUCCESS’)
}
steps{
 shell(‘’’ if sudo kubectl get deploy | grep web-deploy
 then
 echo “Web Server running”
 else
 echo “Server not running”
 fi
 ‘’’)**
> **}
}
job(‘task6_job4’){
description(‘OS working status & mail devilery’)
triggers {
upstream(‘task6_job3’,’SUCCESS’)
}
publishers {
extendedEmail {
recipientList(‘ashwani2399@gmail.com’)
defaultSubject(‘Webserver Issue’)
defaultContent(‘’’Hey Developer,
Your deployment is facing some issue so kindly troubleshoot it as quick as you can’’’)
contentType(‘text/plain’)
triggers {
beforeBuild()
stillUnstable {
subject(‘Subject’)
content(‘Body’)
sendTo {
developers()
requester()
culprits()
}
}
}
}
}
steps{
 shell(‘’’ if sudo kubectl get deploy | grep web-deploy
 then
 echo “Web Server running”
 exit 0
 else
 exit 1
 fi
 ‘’’)
}
}**
> **buildPipelineView(‘DevOps Assembly Lines Task6’) {
filterBuildQueue()
filterExecutors()
title(‘Task6’)
displayedBuilds(1)
selectedJob(‘seed_job’)
alwaysAllowManualTrigger()
showPipelineParameters()
refreshFrequency(60)
}**

Output Images:

![](https://cdn-images-1.medium.com/max/3794/1*OUEwHpoTVBIIwaMpefe0qQ.png)

![](https://cdn-images-1.medium.com/max/3840/1*5CXVnFtuhEzN4X2wErkjPw.png)

![](https://cdn-images-1.medium.com/max/3840/1*KMAHLWSuo9j36LSCyNY76w.png)

![](https://cdn-images-1.medium.com/max/3836/1*ZhYc0btdnSiTX_j_aXcMxA.png)

![](https://cdn-images-1.medium.com/max/3834/1*sO_Qrakrl5diEcAFFMvDgw.png)

![](https://cdn-images-1.medium.com/max/3832/1*IakhMfHAfdqCpFJ_GbzoHA.png)

![](https://cdn-images-1.medium.com/max/3834/1*I5ZDit1WyFyYj4sKtJkf9Q.png)

Thanks to all viewers.

Your comments & review are valuable for me.

LinkedIn Profile: [Click Here](https://www.linkedin.com/in/ashwani-s-6bb8ba110/)

GitHub Repo URL: [Click Here](https://github.com/ashwani7273/devopsal_task6.git)

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Subscribe to [FAUN topics](https://www.faun.dev/join?utm_source=medium.com/faun&utm_medium=medium&utm_campaign=faunmediumprebanner) and get your weekly curated email of the must-read tech stories, news, and tutorials **🗞️

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and [Instagram](https://instagram.com/fauncommunity/) **📷 **and join our [Facebook](https://www.facebook.com/groups/364904580892967/) and [Linkedin](https://www.linkedin.com/company/faundev) Groups **💬

![](https://cdn-images-1.medium.com/max/3000/1*_cT0_laE4iPcqW1qrbstAg.gif)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
