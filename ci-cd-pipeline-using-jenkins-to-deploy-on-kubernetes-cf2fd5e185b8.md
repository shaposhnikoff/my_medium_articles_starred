Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m78[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m104[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m40[39m }

# CI/CD Pipeline using Jenkins to deploy on Kubernetes

Image by Robson Machado from Pixabay

In this tutorial we will see as how we can setup CI/CD Pipeline using Jenkins to deploy on Kubernetes

**Step-1 — **Install kubernetes cluster and in this tutorial , I have used LXC containers .

![Kubernetes Cluster Ready](https://cdn-images-1.medium.com/max/2000/1*Fno6orLTm_sq8w_CW-Ie0g.png)*Kubernetes Cluster Ready*

**Step-2 — Install Jenkins**

**a) Install Java**

    sudo apt update
    sudo apt install openjdk-8-jdk

**b) Add Jenkins Repository & append package repository address to the server’s source.list**

    wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -

    sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'

**c) Install Jenkins**

    sudo apt update
    sudo apt install jenkins

**d) check Jenkins status**

![](https://cdn-images-1.medium.com/max/2000/1*fT-KCn_QpBToyoeCUswXvw.png)

**e) Configure Jenkins with necessary credentials and login to Jenkins**

**f) Add Jenkins to the docker group “*sudo usermod -aG docker jenkins”***

**g) Install the following plugins for Jenkins**

Docker Pipeline 
Kubernetes 
Kubernetes Continuous Deploy

**h) The final step is to use ngrok to expose localhost Jenkins URL as public URL.**

Download ngrok and execute it as shown in the image, [http://a0ecbd0426f6.ngrok.io/](http://a0ecbd0426f6.ngrok.io/) is the new URL to login to Jenkins and accessible over the internet.

![](https://cdn-images-1.medium.com/max/2000/1*LXCvuboww1hbYHpG3tUlhQ.png)

**Step-3: Configure Jenkins**

**a) Select Demo1 as New Pipeline Project**

![](https://cdn-images-1.medium.com/max/2000/1*burAa-PDeVIz-vN5laxVeA.png)

**b) Mention the GitHub link**

![](https://cdn-images-1.medium.com/max/2000/1*q7s1fS-A30HZCyosu2CY0A.png)

**c) Mention the build trigger**

![](https://cdn-images-1.medium.com/max/2000/1*O0YitnIv8zDr4PHAnoR-9Q.png)

**d) In the Pipeline mention the git links and the branch it has to operate upon**

![](https://cdn-images-1.medium.com/max/2000/1*0G8-0QO54NH3Frv4NQPhLQ.png)

**e) Configure docker hub credentials**

![](https://cdn-images-1.medium.com/max/2000/1*HKmV8QNLR0-5wIrU6NNtHg.png)

**f) Update Jenkinsfile** -Its a declarative pipeline which is used as a code. It helps the pipeline code easier to read and write. This code is written in a Jenkinsfile.

In this implementation, the moment code is checked-in, Jenkins will have a notification through a webhook configured in Github. it follows the following stages below

i) Check out the source

ii)Build the new image

iii) Push it to the new repository

iv) Deploy the app on Kube

<iframe src="https://medium.com/media/14203bf2922b90d274f7a776cc6c12c4" frameborder=0></iframe>

**g) Have the configuration of mykubeconfig in Jenkins defined**

![](https://cdn-images-1.medium.com/max/2000/1*EJMopE140ClNiORGW6aWNA.png)

**h) Also have the connection between Jenkins and Kubernetes in place**

![](https://cdn-images-1.medium.com/max/2000/1*ltqa57TDabFD12TOqCUEig.png)

**i) Mention the Jenkins URL**

![](https://cdn-images-1.medium.com/max/2000/1*Vgl9EEAqzy821v5DTaRMAg.png)

**j)Update the pod template.**

![](https://cdn-images-1.medium.com/max/2000/1*jtxHDVc4HTEV-8l1pSfJJg.png)

**k) Configure the webhook in Github**

![](https://cdn-images-1.medium.com/max/2000/1*1ikCTfnGJ6uHCv1cY0bzJA.png)

**Step-4 — Develop application deployment manifest. In this tutorial, we are deploying a simple hellowhale app exposed as NodePort.**

<iframe src="https://medium.com/media/82a3ef3d6c81db7ef1d76a5db4599e6f" frameborder=0></iframe>

**Step-5: Check-in the application manifest in GitHub and we can see the deployment starting in Jenkins**

![](https://cdn-images-1.medium.com/max/2504/1*4lpG9yCT_SSk4Sgnbp7_LQ.png)

**Step-6 **Once the deployment is completed, we can see the following pods(3) are in place and exposed as NodePort(31113)

![](https://cdn-images-1.medium.com/max/2000/1*IKPzJuDUnFaP07xjoHR00A.png)

**Access the deployment on NodePort.**

![](https://cdn-images-1.medium.com/max/2000/1*EaNg5NbazKvvqVlbxpWnHA.png)

**Step-7: Update the deployment by changing the replica’s from 3 to 5 and we can see 5 pods are deployed on Kube.**

![](https://cdn-images-1.medium.com/max/2000/1*1WCUrTe7sBRYq9ZHTVOnbA.png)

**Source Code: [https://github.com/vamsijakkula/hellowhale](https://github.com/vamsijakkula/hellowhale)**

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Subscribe to [FAUN topics](https://www.faun.dev/join?utm_source=medium.com/faun&utm_medium=medium&utm_campaign=faunmediumprebanner) and get your weekly curated email of the must-read tech stories, news, and tutorials **🗞️

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and [Instagram](https://instagram.com/fauncommunity/) **📷 **and join our [Facebook](https://www.facebook.com/groups/364904580892967/) and [Linkedin](https://www.linkedin.com/company/faundev) Groups **💬

![](https://cdn-images-1.medium.com/max/3000/1*_cT0_laE4iPcqW1qrbstAg.gif)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
