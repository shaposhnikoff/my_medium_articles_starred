
# DevOps Pipeline Fundamentals



## DevOps 101 - Pipeline Fundamentals

**DevOps **(Development & Operations) is basically where it is at in the IT world right now. In reality Blockchain, AI, ML for example has somewhat less demand from my experience.

So what is DevOps? DevOps is an enterprise software development phrase that is used to describe an optimal agile relationship between development and IT operations.

The main goal of DevOps is to change and improve the relationship of teams by advocating better communication and collaboration between these two or more business units.

DevOps is not only a technical shift in the way of running enterprise software deployments but also a cultural shift from the enterprises old ways of running software releases, testing, updates and integrations, etc.

Another common way to look at DevOps is that it is a software development lifecycle (SDLC) and its central goal is to provide a cultural change.

Developers and non-developers work in an environment where the two departments may not be working together but instead against each other . Organizations that take improvement seriously wont tolerate that and will invest in a DevOps strategy.

To sum up DevOps it is a movement which enhances collaboration between Development and Operations, hence, the name DevOps.
> ***DevOps is really a derivative of several best practices namely Agile methodology and the Software Development Life Cycle (SDLC)***

***CI/CD and Continuous Development- Tell me More…*** ***What is a pipeline? (aka — mainline)***

A pipeline in the world of software development is a term used to describe a workflow. In some vendor implementations a workflow specifically has stages or phases as they may also be referred to.
> *A phase is when you advance a stage moving towards a production deployment of software.*

Pipelines should ideally be designed for businesses that want to improve applications frequently and require a reliable automated delivery process

**Overview of how pipelines compare compared to the phases of the development lifecycle**

![](https://cdn-images-1.medium.com/max/2000/0*2woITKKwMTul_3rN)

We can see from the diagram above that Continuous Integration is focused on the Code to Integrate phases. Continuous Delivery takes it to the release phase which is still manually driven. Mature SDLC and Agile organization make it the Continuous Deployment where automation can be realized with efficiency.

**Continuous Integration (CI)**

Continuous Integration (CI) is the practice of merging all developer working copies to a shared pipeline several times a day.

In practice, CI involves a centralized server that continually pulls in all new source code changes as developers commit them and builds the software application from scratch, notifying the team of any failures in the process

A CI pipeline at its simplest form would be similar to this whether it was deployed on premises or in the cloud.

When discussing a CI pipeline in the cloud it’s hard not to discuss options in AWS, MS Azure or GCP . The cloud providers really do have some amazing services.

***Benefits of Continuous Integration (CD)***
> *The main benefits of Continuous Integration are really focused on cost efficiency, reduction of risk and the removal of manual processes*

**Continuous Delivery (CD)**

Continuous Delivery is the ability to facilitate changes of all types with developer copies which includes new features, configuration changes, bug fixes and experiments into production, or into the hands of users, safely and quickly in a sustainable way. Continuous Deployments can be thought of as an extension of continuous integration, aiming at minimizing lead time, the time elapsed between development writing one new line of code and this new code being used by live users, in production

Benefits of Continuous Delivery
> *The main benefits of Continuous Delivery are focused on the lower risk of release, faster time to market and the high quality of software that is produced at lower costs*

**Continuous Deployment**

Continuous Deployments can be thought of as an extension of continuous integration effectively minimizing lead time which is the time elapsed between development writing one new line of code and this new code being used by live users, in production

Continuous deployment is just difficult and takes significant investment of which few small and mid size organizations invest in.

There are some well-known companies that have documented use cases of their own organizations running a Continuous Deployment environment.

Some companies are Netflix, Amazon, Pinterest, Google

**Benefits of Continuous Deployment**
> *The main benefits of Continuous Deployments are lower risks, reduce lead time to market, quicker feedback and an improved Return on Investment (ROI) as well.*

With Google Cloud Platform I generally like to highlight the workflow and describe why the code is kept in a Github and how the pipeline works from the start to finish.

For example when reviewing Cloud Services you want to get an idea of what services are supported and how.

**List of DevOps related services in GCP and AWS**

![](https://cdn-images-1.medium.com/max/2000/0*6TKaqqby_-cCn16k)

**Example CI Pipeline in Google Cloud Platform (GCP)**

![](https://cdn-images-1.medium.com/max/2000/0*aQYOSG_kNF1ksxLn)

In Google Cloud Platform the services that are used are listed. Cloud Source Repositories is a “GitHub” that is hosted on GCP.

Cloud Build is the service that can be used to execute your builds.

Container Registry is for artefact management meaning that the code is kept safe and revision control is used. Kubernetes Engine is Google Cloud’s Docker container service.

**Let us *review* your knowledge** ***1. What is the term used to describe a workflow in DevOps? Select One***

***2. Your company is getting ready to move to the cloud all their development focused services. You are currently using Jenkins on-premises and would like to confirm you could move all working copies to the cloud.***

What is the term used for merging all developer working copies to a shared pipeline several times a day?

1. Continuous Integration

1. Continuous Delivery

1. Continuous Deployment

1. This is not possible in the cloud with on-premises services.

**If you guessed answer 1 for Question 1 and for 2 Great Job**

Thinking of taking a serious DevOps exam. Review my post on the Google Cloud DevOps Engineer exam!

Carry on my cloud friends and please do let me know any feedback or suggestions.

*Joe Holbrook, the Cloud Tech Guy*

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and [Instagram](https://instagram.com/fauncommunity/) **📷 **and join our [Facebook](https://www.facebook.com/groups/364904580892967/) and [Linkedin](https://www.linkedin.com/company/faundev) Groups **💬**.**

**To join our community Slack team chat **🗣️ **read our weekly Faun topics **🗞️,** and connect with the community **📣** click here⬇**

![](https://cdn-images-1.medium.com/max/3000/1*6P3WpLjGv5v1ucm5dgkucg.png)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
