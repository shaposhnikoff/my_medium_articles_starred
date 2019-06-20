
# CI CD Pipeline:  Learn How to Setup a CI CD Pipeline From Scratch

CI CD Pipeline - Edureka

CI CD Pipeline implementation or the Continuous Integration/Continuous Deployment software is the backbone of the modern DevOps environment. You can find the requirement of [*Continuous Integration](https://www.edureka.co/blog/continuous-integration/?utm_source=medium&utm_medium=content-link&utm_campaign=ci-cd-pipeline) & Continuous Deployment skills* in various job roles such as Data Engineer, Cloud Architect, Data Scientist, etc. CI/CD bridges the gap between development and operations teams by automating build, test, and deployment of applications. In this blog, we will know What is CI CD pipeline and how it works.

Before moving onto the CI CD pipeline’s working, let’s start by understanding DevOps.

## What is DevOps?

![](https://cdn-images-1.medium.com/max/2270/1*w0aSgmjyxkDOZWjzAr061g.png)

DevOps is a software development approach which involves continuous development, continuous testing, continuous integration, continuous deployment and continuous monitoring of the software throughout its development life cycle. This is exactly the process adopted by all the top companies to develop high-quality software and shorter development life cycles, resulting in greater customer satisfaction, something that every company wants.

## DevOps Stages

Your understanding of what is DevOps is incomplete without learning about its life cycle. Let us now look at the DevOps lifecycle and explore how they are related to the software development stages.

![](https://cdn-images-1.medium.com/max/2270/1*sln2Y2DyyEo227IRRC67cA.png)

## What is CI CD Pipeline?

CI stands for Continuous Integration and CD stands for Continuous Delivery and Continuous Deployment. You can think of it as a process which is similar to a software development lifecycle.
 Now let us see how does it work.

![](https://cdn-images-1.medium.com/max/7520/1*vMNZfDEYPz6My4M4qx56tQ.png)

The above pipeline is a logical demonstration of how software will move along the various phases or stages in this lifecycle, before it is delivered to the customer or before it is live on production.

Let’s take a scenario of CI CD Pipeline. Imagine you’re going to build a web application which is going to be deployed on live web servers. You will have a set of developers who are responsible for writing the code which will further go on and build the web application. Now, when this code is committed into a version control system(such as [git](https://www.edureka.co/blog/git-tutorial/?utm_source=medium&utm_medium=content-link&utm_campaign=ci-cd-pipeline), svn) by the team of developers. Next, it goes through the **build phase** which is the first phase of the pipeline, where developers put in their code and then again code goes to the version control system having a proper version tag.

![](https://cdn-images-1.medium.com/max/7520/1*oMqMc6sdKH99ZH_W7gHH2Q.png)

Suppose we have a [Java](https://www.edureka.co/blog/java-tutorial/?utm_source=medium&utm_medium=content-link&utm_campaign=ci-cd-pipeline) code and it needs to be compiled before execution. So, through the version control phase, it again goes to build phase where it gets compiled. You get all the features of that code from various branches of the repository, which merge them and finally use a compiler to compile it. This whole process is called the **build phase**.

## Testing Phase:

![](https://cdn-images-1.medium.com/max/7520/1*Z2_WhEPQbbYcrFS5aVyLpg.png)

Once the build phase is over, then you move on to the **testing phase**. In this phase, we have various kinds of testing, one of them is the *unit test* (where you test the chunk/unit of software or for its sanity test).

## Deploy Phase:

![](https://cdn-images-1.medium.com/max/7520/1*H9OypUG5606AjmX2xdbb4g.png)

When the test is completed, you move on to the **deploy phase**, where you deploy it into a staging or a test server. Here, you can view the code or you can view the app in a simulator.

## Auto Test Phase:

![](https://cdn-images-1.medium.com/max/7520/1*atGLIAIVFFxNloZIg_tm-Q.png)

Once the code is deployed successfully, you can run another set of a sanity test. If everything is accepted, then it can be deployed to production.

## Deploy to Production:

![](https://cdn-images-1.medium.com/max/7520/1*63Ii4W-N4qzKCcbrPcRupQ.png)

Meanwhile in every step, if there is some error, you can shoot a mail back to the development team so that they can fix them. Then they will push it into the version control system and goes back into the pipeline.

Once again if there is any error reported during testing, again the feedback goes to the dev team where they fix it and the process re-iterates if required.

## Measure+Validate:

![](https://cdn-images-1.medium.com/max/7512/1*kTZjSjA-nR6xLM8bH8ihWw.png)

So, this lifecycle continues until we get a code or a product which can be deployed in the production server where we measure and validate the code.

We have understood CI CD Pipeline and its working, now we will move on to understand what Jenkins is and how we can deploy the demonstrated code using Jenkins and automate the entire process.

## Jenkins — The Ultimate CI Tool and Its Importance in CI CD Pipeline

Our task is to automate the entire process, from the time the development team gives us the code and commits it to the time we get it into production.

Our task is to automate the pipeline in order to make the entire software development lifecycle on the dev-ops mode/ automated mode. For this, they would need automation tools.

![](https://cdn-images-1.medium.com/max/4096/1*iKuaNfxgZSTe_J2x3PYRUg.png)

Jenkins provides us with various interfaces and tools in order to automate the entire process.

So what happens, we have a git repository where the development team will commit the code. Then Jenkins takes over from there which is a front-end tool where you can define your entire job or the task. Our job is to ensure the continuous integration and delivery process for that particular tool or for the particular application.

From Git, Jenkins pulls the code and then moves it to the **commit phase**, where the code is committed from every branch. Then Jenkins moves it into the **build phase** where we compile the code. If it is Java code, we use tools like maven in Jenkins and then compile that code, which we can be deployed to run a series of tests. These test cases are overseen by Jenkins again.

Then it moves on to the staging server to deploy it using [Docker](https://www.edureka.co/blog/docker-tutorial?utm_source=medium&utm_medium=content-link&utm_campaign=ci-cd-pipeline). After a series of Unit Tests or sanity test, it moves to the production.

This is how the delivery phase is taken care by a tool called **Jenkins,** which automate everything. Now in order to deploy it, we will need an environment which will replicate the production environment, I.e., **Docker**.

## Docker

![](https://cdn-images-1.medium.com/max/4096/1*7k5Z3-gJaqrY7YFxQx509g.png)

Docker is just like a virtual environment in which we can create a server. It takes a few seconds to create an entire server and deploy the artifacts which we want to test. But here the question arises,

***Why do we use docker?***

As said earlier, you can run the entire cluster in a few seconds. We have storage registry for images where you build your image and store it forever. You can use it anytime in any environment which can replicate itself.

## Hands-On: Building CI CD Pipeline Using Docker and Jenkins

**Step 1:** Open your terminal in your VM. Start Jenkins and Docker using the commands “**systemctl start jenkins**“, “**systemctl enable jenkins**“, “**systemctl start docker**“.
> **Note:** Use **sudo **before the commands if it display “privileges error”.

![](https://cdn-images-1.medium.com/max/2000/1*_V973kwydbydjbjuByQeIQ.png)

**Step 2:** Open your Jenkins on your specified port. Click on **New Item **to create a Job.

![](https://cdn-images-1.medium.com/max/2000/1*knyeojAlHRoK4WblYbBHsA.png)

**Step 3:** Select **freestyle** project and provide the item name (here I have given Job1) and click OK.

![](https://cdn-images-1.medium.com/max/2000/1*j-TFvu_MOMVsdEbGkO_e0Q.png)

**Step 4:** Select **Source Code Management** and provide the **Git** repository. Click on **Apply** and **Save** button.

![](https://cdn-images-1.medium.com/max/2000/1*PNTIe1gAhb2kSaFR4CURIQ.png)

**Step 5:** Then click on **Build->Select Execute Shell**.

![](https://cdn-images-1.medium.com/max/2000/1*J2jpLriZkrtkebC-yTovPA.png)

**Step 6:** Provide the shell commands. Here it will build the archive file to get a war file. After that, it will get the code which is already pulled and then it uses maven to install the package. So, it simply installs the dependencies and compiles the application.

![](https://cdn-images-1.medium.com/max/2000/1*kmJhcAixHbGENYqdLy0gLw.png)

**Step 7:** Create the new **Job** by clicking on New Item.

![](https://cdn-images-1.medium.com/max/2000/1*knyeojAlHRoK4WblYbBHsA.png)

**Step 8:** Select **freestyle** project and provide the item name (here I have given Job2) and click on OK.

![](https://cdn-images-1.medium.com/max/2000/1*Va92M7--FpIfQQADOhgXbg.png)

**Step 9:** Select **Source Code Management** and provide the **Git** repository. Click on **Apply** and **Save** button.

![](https://cdn-images-1.medium.com/max/2000/1*PNTIe1gAhb2kSaFR4CURIQ.png)

**Step 10:** Then click on **Build->Select Execute Shell**.

![](https://cdn-images-1.medium.com/max/2000/1*J2jpLriZkrtkebC-yTovPA.png)

**Step 11:** Provide the shell commands. Here it will start the integration phase and **build** the Docker Container.

![](https://cdn-images-1.medium.com/max/2000/1*4Kd1illXWp6Ygohs3y0ZmQ.png)

**Step 12:** Create the new **Job** by clicking on New Item.

![](https://cdn-images-1.medium.com/max/2000/1*knyeojAlHRoK4WblYbBHsA.png)

**Step 13:** Select **freestyle** project and provide the item name (here I have given Job3) and click on OK.

![](https://cdn-images-1.medium.com/max/2000/1*S-JUP6FQ6ALNH7UQ4Aagpg.png)

**Step 14:** Select **Source Code Management** and provide the **Git** repository. Click on **Apply** and **Save** button.

![](https://cdn-images-1.medium.com/max/2000/1*PNTIe1gAhb2kSaFR4CURIQ.png)

**Step 15:** Then click on **Build->Select Execute Shell**.

![](https://cdn-images-1.medium.com/max/2000/1*J2jpLriZkrtkebC-yTovPA.png)

**Step 16:** Provide the shell commands. Here it will check for the Docker Container file and then deploy it on port number 8180. Click on Save button.

![](https://cdn-images-1.medium.com/max/2000/1*nYXJOQLi_M7gz_22rS9klg.png)

**Step 17:** Now click on **Job1 -> Configure**.

![](https://cdn-images-1.medium.com/max/2000/1*3zrWChBfkUTeVJ1a1hKhVA.png)

**Step 18:** Click on **Post-build Actions -> Build other projects**.

![](https://cdn-images-1.medium.com/max/2000/1*GeAhSQ_3KpwI6yvTZsbxKQ.png)

**Step 19:** Provide the project name to build after Job1 (here is Job2) and then click on **Save**.

![](https://cdn-images-1.medium.com/max/2000/1*xNUYniDTuZd2jd5WnQyv2w.png)

**Step 20: **Now click on **Job2 -> Configure**.

![](https://cdn-images-1.medium.com/max/2000/1*mepXFcMyb8MfWGiOra9sNw.png)

![](https://cdn-images-1.medium.com/max/2732/1*Ug5BWpA9tB2UrgvV6aWM3Q.png)

**Step 21:** Click on **Post-build Actions -> Build other projects**.

![](https://cdn-images-1.medium.com/max/2000/1*GeAhSQ_3KpwI6yvTZsbxKQ.png)

**Step 22:** Provide the project name to build after Job2 (here is Job3) and then click on **Save**.

![](https://cdn-images-1.medium.com/max/2000/1*VRBvBG3dWlIuOMYMZtlhzg.png)

**Step 23:** Now we will be creating a Pipeline view. Click on ‘+’ sign.

![](https://cdn-images-1.medium.com/max/2000/1*X8qEIdSmz-e9oeWo2E4R-g.png)

**Step 24:** Select **Build Pipeline View** and provide the view name (here I have provided CI CD Pipeline).

![](https://cdn-images-1.medium.com/max/2732/1*q3EAP12eVE4UpsJIoqFZtA.png)

**Step 25:** Select the **initial** **Job** (here I have provided Job1) and click on OK.

![](https://cdn-images-1.medium.com/max/2732/1*zMTq2YexhFFMn1dguuxKQw.png)

**Step 26:** Click on **Run **button to start the CI CD process.

![](https://cdn-images-1.medium.com/max/2732/1*f40jsHRh4ITtKOrGrvbuCw.png)

**Step 27:** After successful build open **localhost:8180/sample.text**. It will run the application.

![](https://cdn-images-1.medium.com/max/3584/1*0XbyVIHdqP1sEs-Fh8hycw.png)

So far, we have learned how to create CI CD Pipeline using Docker and Jenkins. The intention of DevOps is to create better-quality software more quickly and with more reliability while inviting greater communication and collaboration between teams. If you wish to check out more articles on the market’s most trending technologies like Artificial Intelligence, Python, Ethical Hacking, then you can refer to[ Edureka’s official site.](https://www.edureka.co/blog?utm_source=medium&utm_medium=content-link&utm_campaign=ci-cd-pipeline)

Do look out for other articles in this series which will explain the various other aspects of DevOps.
> 1. [DevOps Tutorial](https://medium.com/edureka/devops-tutorial-89363dac9d3f)
> 2. [Git Tutorial](https://medium.com/edureka/git-tutorial-da652b566ece)
> 3. [Jenkins Tutorial](https://medium.com/edureka/jenkins-tutorial-68110a2b4bb3)
> 4. [Docker Tutorial](https://medium.com/edureka/docker-tutorial-9a6a6140d917)
> 5. [Ansible Tutorial](https://medium.com/edureka/ansible-tutorial-9a6794a49b23)
> 6. [Puppet Tutorial](https://medium.com/edureka/puppet-tutorial-848861e45cc2)
> 7. [Chef Tutorial](https://medium.com/edureka/chef-tutorial-8205607f4564)
> 8. [Nagios Tutorial](https://medium.com/edureka/nagios-tutorial-e63e2a744cc8)
> 9. [How To Orchestrate DevOps Tools?](https://medium.com/edureka/devops-tools-56e7d68994af)
> 10. [Continuous Delivery](https://medium.com/edureka/continuous-delivery-5ca2358aedd8)
> 11. [Continuous Integration](https://medium.com/edureka/continuous-integration-615325cfeeac)
> 12. [Continuous Deployment](https://medium.com/edureka/continuous-deployment-b03df3e3c44c)
> 13. [Continuous Delivery vs Continuous Deployment](https://medium.com/edureka/continuous-delivery-vs-continuous-deployment-5375642865a)
> 14. [Docker Compose](https://medium.com/edureka/docker-compose-containerizing-mean-stack-application-e4516a3c8c89)
> 15. [Docker Swarm](https://medium.com/edureka/docker-swarm-cluster-of-docker-engines-for-high-availability-40d9662a8df1)
> 16. [Docker Networking](https://medium.com/edureka/docker-networking-1a7d65e89013)
> 17. [Ansible Roles](https://medium.com/edureka/ansible-roles-78d48578aca1)
> 18.[ Ansible Vault](https://medium.com/edureka/ansible-vault-secure-secrets-f5c322779c77)
> 19. [Ansible for AWS](https://medium.com/edureka/ansible-for-aws-provision-ec2-instance-9308b49daed9)
> 20. [Jenkins Pipeline](https://medium.com/edureka/jenkins-pipeline-tutorial-continuous-delivery-75a86936bc92)
> 21. [Top Git Commands](https://medium.com/edureka/git-commands-with-example-7c5a555d14c)
> 22. [Top Docker Commands](https://medium.com/edureka/docker-commands-29f7551498a8)
> 23. [Git vs GitHub](https://medium.com/edureka/git-vs-github-67c511d09d3e)
> 24. [DevOps Interview Questions](https://medium.com/edureka/devops-interview-questions-e91a4e6ecbf3)

*Originally published at [www.edureka.co](https://www.edureka.co/blog/ci-cd-pipeline/) on August 9, 2018.*
