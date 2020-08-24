Unknown markup type 10 { type: [33m10[39m, start: [33m17[39m, end: [33m29[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m62[39m, end: [33m68[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m92[39m, end: [33m100[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m97[39m, end: [33m120[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m132[39m, end: [33m147[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m53[39m, end: [33m68[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m59[39m, end: [33m72[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m48[39m, end: [33m62[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m7[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m25[39m, end: [33m36[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m127[39m, end: [33m137[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m70[39m, end: [33m91[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m264[39m, end: [33m286[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m101[39m, end: [33m113[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m195[39m, end: [33m203[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m130[39m, end: [33m144[39m }

# Blue/Green Deployments with Amazon ECS — Part 1 — Console

Amazon EC2 Container Service (ECS) is Amazon’s solution for running and orchestrating Docker containers. It provides an interface for defining and deploying Docker containers to run on clusters of EC2 instances.

The initial setup and configuration of an ECS cluster is not exactly trivial, but once configured it works well and makes running and scaling container-based applications relatively smooth. ECS also has support for blue-green deployments built in, but first we’ll cover some basics about getting set up with ECS and about what is Blue/Green Deployments.

## Why Blue/Green Deployments

With hardware becoming more and more cost-effective, Blue/Green deployments are becoming common. Blue/Green deployments allow customers to validate an entirely new version of an application without disturbing production traffic at all.

In Blue/Green deployments, two versions of the application coexist. A Blue version (production version) is the actual application receiving production traffic, whereas the Green Version (stage/new version) is the idle version that is available for testing.

New applications would always be created as a Green version. Developers can test this version of the application thoroughly. Once this new version is verified and is safe to promote to production, it can become the production version almost instantaneously.

Now, developers can choose to downsize the older version of the application or keep it around for some time. Or, required, it can be switched back with the production version.

![](https://cdn-images-1.medium.com/max/4410/1*7aGlAjiRhd0Y2m7_OLHU7w.png)

**Benefits Of Blue/Green Deployment**

* If issues are found with the new version (Green version), it can be used for troubleshooting, whereas production traffic is still uninterrupted as the original version(Blue version) is untouched.

* Almost zero downtime because switching between prod and stage applications is almost instantaneous. The routes are simply updated

## Containers make it simpler

Historically, blue/green deployments were not often used to deploy software on-premises because of the cost and complexity associated with provisioning and managing multiple environments. Instead, applications were upgraded in place.

Although this approach worked, it had several flaws, including the ability to roll back quickly from failures. Rollbacks typically involved re-deploying a previous version of the application, which could affect the length of an outage caused by a bad release. Fixing the issue took precedence over the need to debug, so there were fewer opportunities to learn from your mistakes.

## **Lets get Started**

### Step 1

To begin, we have a single service running two tasks. The two tasks are run on one EC2 Instances with dynamic port mapping.

Make a NodeJS Application and Run it on as a Docker Container

    const express = require('express')
    const app = express();

    // Constants
    const PORT = 8080;
    const HOST = '0.0.0.0';

    app.get('/healthcheck', (req, res) => res.status(200).json({status: 'ok'}));
    app.get('/version', (req, res) => res.status(200).json({status: 'Version 1'}));

    app.listen(PORT, HOST);
    console.log(`Running on [http://${HOST}:${PORT}`](http://${HOST}:${PORT}`));

We have a simple /healthcheck api end-point that just returns 200 OK when called and also a /version endpoint which tells which code version is it running.

### Step 2:

Make a Dockerfile and other necessary files and build a image. A sample repository can be found at the [link](https://github.com/sanudatta11/nodejs-docker-medium). Build the Docker Image and push to DockerHub or ECR or keep it as it is, we can do it later by directly using our pipeline.

    FROM node:10

    # Create app directory
    WORKDIR /usr/src/app

    # Install app dependencies
    # A wildcard is used to ensure both package.json AND package-lock.json are copi$
    # where available (npm@5+)
    COPY package*.json ./

    RUN npm install
    # If you are building your code for production
    # RUN npm ci --only=production

    # Bundle app source
    COPY . .

    EXPOSE 8080
    CMD [ "node", "server.js" ]

### Step 3:

Make a new Cluster in ECS and use proper subnet configurations in it. Make sure it has ample IP Space Available.

### Step 4:

Create a Task Definition with proper details and you may use the Bridge or awsvpc Network type for the TD.

Click on Add Container Button in the Container Definition section to add details about the Docker Image required for the Task. Add a Soft Memory limit if required. Put proper Docker Image Tag in the Container Tag Position. Also put the Port Mappings as needed for the deployment.

![](https://cdn-images-1.medium.com/max/3456/1*uly7YZGR3t8vwMGWeC8ICQ.png)

### **Step 5:**

Create a service with the appropriate details about the task. Create the Service with no Autoscaling as of now and allocate Load Balancer if required. Now we need to configure the ELB to use our API Endpoint for healthcheck. When creating the service there would be an option to add a Load Balancer in the second part. Make sure to add the port mappings correctly and also add the healthcheck endpoint as shown in the screenshot below.

![](https://cdn-images-1.medium.com/max/3492/1*iey2B8bwh5eCaqnadC8bhA.png)

**Few Important Pointers:-**

1. Target Group Port should be the port of your container listening

1. The Production Listener Port should be your Load Balancer port which is accepting the connections.

1. It is recommended to have a higher healthy threshold than unhealthy threshold as is 4 and 2 here. More on that in the next point.

1. Close the ECS configuration and open the application’s task definition. Under deployment, update Minimum Healthy Percent to **100** and Maximum Percent to **200**. So, when a new version of code is deployed, ELB will maintain old and new instances of the application for sometime and destroy the old instance after healthy threshold is achieved with newly deployed version.

### Step 6:

Now as ECS is setup what we need to do is configure the pipeline.We will walkthrough the steps to create the above CI/CD pipeline for our test app to ensure that whenever new code is checked in, all the tests are run, packaged and deployed without causing any downtime.

1. First navigate to Code Pipeline console and click on Create Pipelinebutton.

![](https://cdn-images-1.medium.com/max/2644/1*qzpcLBQxOs70Hzg8bwtxQg.png)

2. Point to the repository where the application code is available. My Code is available in a public github repository at [here](https://github.com/sanudatta11/nodejs-docker-medium).

![](https://cdn-images-1.medium.com/max/2644/1*O4SbViKWDgMt0fs173F9rg.png)

3. On the next screen, we need to select build provider as AWS CodeBuild and choose the region of our ECS Cluster and add a project name by selecting an already existing one or create a new project using the radio option.

**Note:** You can also add any Environment Variables you would like to the Build directly from here as shown below.

![](https://cdn-images-1.medium.com/max/3000/1*UDdatkesZZFSqrvS0-69qw.png)

4. As we will be creating a New Project please follow the below screenshot very carefully.

![](https://cdn-images-1.medium.com/max/3000/1*UDdatkesZZFSqrvS0-69qw.png)

![](https://cdn-images-1.medium.com/max/3164/1*-Uc80rtKnwNJlrh_HTwBpQ.png)

5. Leave the BuildSpec Category to default. The build_spec.yml has been already provided in the code repository. Use that with minor alterations and it should work just fine.

![](https://cdn-images-1.medium.com/max/2956/1*CmKh5aNfibSaQpLi7d2ffA.png)

Lets understand the build_spec.yml file a bit.

The current git commit is captured as IMAGE_TAG to be suffixed to docker image name.

* We use npm ci instead of npm install to install the dependencies as a pre-build step.

* The aws login command is to initialise the variables to later push the docker image to [ECR](https://aws.amazon.com/ecr/).

* After the pre-build phase, we run the test suite and if that succeeds, we build the docker image (this post assumes you have a Dockerfile in the root of the repo relevant to booting a Node.js app)

* In the post-build phase, we push the docker image to ECR and create a imagedefinitions.json file that will be used in the deploy step.

6. Now that we have added the build stage, the next stage or final stage is the deploy stage. As we are using Blue Green Deployment we would need to use the AWS ECS Blue Green Deployment Method using AWS Code Deploy. Click on Next and on the Deploy stage , choose Amazon ECS(Blue/Green) .

![](https://cdn-images-1.medium.com/max/2716/1*DATSO1rqcWGpMvGJ2UhNUg.png)

Now, if you have a Code Deploy Application already created go ahead and select the application. One simple way to create the Deployment group is while creating the service you can choose the update to be AWS ECS( Blue/Green), this will go ahead and create the exact Code Deploy Application and Group for you. Find the screenshot below for a example.

![](https://cdn-images-1.medium.com/max/3076/1*kqWSTSRiQ_KN_a7bthwnew.png)

Select the proper Code Deploy Application and group in the Pipeline and voila its set. You can write your own appspec.yml for the deployment configurations .

![](https://cdn-images-1.medium.com/max/2936/1*6HPavoi9NV6WmmVeiTKo4w.png)

7. One More way is to use general AWS ECS Deployment and using the imagdefinitions.json file for the update by making 200% of resources and slowly booting off the older ones.

Select Amazon ECS as Deployment Provider in the Deploy Stage as shown below.

![](https://cdn-images-1.medium.com/max/2712/1*Jf5W2yA_tScgQh8NBAmuUg.png)

Select the appropriate region and Cluster and service name for the ECS Cluster. Put the name of our image defnition file in the image definitions field. It can be of any name you have used , just make sure its json formatted properly.

Review your updates by clicking next and then Create the Pipeline. So that is all you need to create a Blue Green Deployment in AWS ECS with Container Instances.

## Final Talks

ELB will keep pinging each task instance of the current running version of the application using the /healthcheck url to confirm that all tasks are healthy. You can check the version by going to /version of the app and change the version text for each revision.

When there is a new commit checked into the source code repository, Code Pipeline will detect the change and execute the steps in build_spec.ymlWhen the new version is deployed, ECS will create a new task definition with latest image and for brief period of time, there will be 200% or 6 tasks of the application running. 3 with the old version of code and 3 with the latest version.

After ELB confirms that the three new task instances are healthy, it will kill off the 3 old instances and restore the task count to three or desired 100%.

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
