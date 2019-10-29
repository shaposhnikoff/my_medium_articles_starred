Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m39[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m60[39m }

# How to build a serverless CI/CD pipeline on AWS



We’d all like to think the days of copy/paste deployments are long gone, unfortunately this is not the case and is still the preferred method of delivery for some developers. I could easily write a whole article as to why its bad, but I think its pretty self explanatory. With the trend towards serverless architectures, CI/CD Pipelines play a major part in your application delivery. They also feature in my [3 top tips for your next serverless project](https://medium.com/@gavinlewis/3-top-tips-for-your-next-serverless-project-2ea87bc833e7).

Continuous Integration and Delivery is something which has interested me for a long time and I first dipped my toes into the water with TeamCity a number of years ago. We still use TeamCity for the majority of our CI/CD pipelines today. TeamCity works great and I don’t have anything against it, but I’m always looking at how to better what we do. One of these things is being able to build our pipelines as code — which is one of the things TeamCity isn’t so good at.

It’s been some time since I’ve looked in detail at the integration and delivery tools AWS has available and while we use CodeDeploy for another project which runs on EC2, I’d never used them to deploy a serverless project. After getting reacquainted with the tools I could see there is now native integration for deploying CloudFormation and Lambda — presumably with AWS’ SAM — we use the [serverless framework](https://serverless.com) and although it generates CloudFormation templates, it doesn’t work out of the box with the tools in AWS.

## Getting Prepared

The AWS services I’ll be using are EC2, Docker, ECR, S3, IAM, CodeBuild, CodePipeline, CloudWatch, CloudTrail. You’ll need to understand at least a basic level what each of these services do to follow along.

I primarily write backend code in .NET, which this tutorial is based on. None of the pre-built CodeBuild images have both .NET and NodeJS runtimes on them (NodeJS is required for the serverless framework). If your Lambda functions are written in NodeJS, the process of setting up a deployment pipeline becomes a lot simpler as its the only runtime you need installed on your Docker image (you could potentially skip a lot of this tutorial). I should also mention, this was my first time getting acquainted with containers so I was excited to learn something new.

I also assume that your code lives in a repository of some kind such as git. For this tutorial we’ll just be uploading a file to S3 which contains a package of the code to be deployed — how you get it there is up to you. You can always go further with what I’ve built by connecting your pipeline to repositories such as github or CodeCommit.

### 1. Create an EC2 instance and install Docker

Start by spinning up a standard AWS Linux 2 EC2 instance — that should be self explanatory. Log in and install Docker with these commands:

    sudo yum update -y
    sudo amazon-linux-extras install docker
    sudo service docker start

We’ll also need to add the ec2-user to the docker group so you can execute Docker commands without using sudo:

    sudo usermod -a -G docker ec2-user

After executing the command, log out and log back into your EC2 instance so ec2-user can assume the new permissions. Then verify ec2-user can run Docker commands without sudo:

    docker info

![Output of the docker info command](https://cdn-images-1.medium.com/max/2000/1*p9mCWiMKlxJVWx8XzYCKqg.png)*Output of the docker info command*

### 2. Build Docker image and push to ECR

Assuming the above step has been successful, the next stage is to build the Docker image and push it to ECR. AWS provides the base images for [CodeBuild on github](https://github.com/aws/aws-codebuild-docker-images), which makes building our own image easier.

I’ve also published my image to github if you don’t want to go through the steps below to build your own: [https://github.com/effectivedigital/serverless-deployment-image](https://github.com/effectivedigital/serverless-deployment-image)

Start by cloning the images and navigating into the .NET Core 2.1 directory:

    git clone [https://github.com/aws/aws-codebuild-docker-images.git](https://github.com/aws/aws-codebuild-docker-images.git)
    cd aws-codebuild-docker-images
    cd ubuntu/dot-net/core-2.1/

Open up Dockerfile in your preferred text editor:

    nano Dockerfile

Add the commands to install NodeJS and the serverless framework at the end of the other commands already in Dockerfile. I was able to get the majority of these commands from the NodeJS Docker image from the same repository from AWS:

    # Install Node Dependencies
    ENV NODE_VERSION="10.14.1"

    # gpg keys listed at [https://github.com/nodejs/node#release-team](https://github.com/nodejs/node#release-team)
    RUN set -ex \
     && for key in \
     94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
     B9AE9905FFD7803F25714661B63B535A4C206CA9 \
     77984A986EBC2AA786BC0F66B01FBB92821C587A \
     56730D5401028683275BD23C23EFEFE93C4CFFFE \
     71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
     FD3A5288F042B6850C66B31F09FE44734EB7990E \
     8FCCA13FEF1D0C2E91008E09770F7A9A5AE15600 \
     C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
     DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
     4ED778F539E3634C779C87C6D7062848A1AB005C \
     A48C2BEE680E841632CD4E44F07496B3EB3C1762 \
     ; do \
     gpg - keyserver hkp://p80.pool.sks-keyservers.net:80 - recv-keys "$key" || \
     gpg - keyserver hkp://ipv4.pool.sks-keyservers.net - recv-keys "$key" || \
     gpg - keyserver hkp://pgp.mit.edu:80 - recv-keys "$key" ; \
     done
    RUN set -ex \
     && wget "[https://nodejs.org/download/release/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.gz](https://nodejs.org/download/release/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.gz)" -O node-v$NODE_VER$
     && wget "[https://nodejs.org/download/release/v$NODE_VERSION/SHASUMS256.txt.asc](https://nodejs.org/download/release/v$NODE_VERSION/SHASUMS256.txt.asc)" -O SHASUMS256.txt.asc \
     && gpg - batch - decrypt - output SHASUMS256.txt SHASUMS256.txt.asc \
     && grep " node-v$NODE_VERSION-linux-x64.tar.gz\$" SHASUMS256.txt | sha256sum -c - \
     && tar -xzf "node-v$NODE_VERSION-linux-x64.tar.gz" -C /usr/local - strip-components=1 \
     && rm "node-v$NODE_VERSION-linux-x64.tar.gz" SHASUMS256.txt.asc SHASUMS256.txt \
     && ln -s /usr/local/bin/node /usr/local/bin/nodejs \
     && rm -fr /var/lib/apt/lists/* /tmp/* /var/tmp/*

    RUN npm set unsafe-perm true

    CMD [ "node" ]

    # Install Serverless Framework
    RUN set -ex \
     && npm install -g serverless

Now the image can be built and tagged:

    docker build -t aws/codebuild/dot-net .

After building has completed, the image can be run to confirm all is working and serverless has installed correctly:

    docker run -it --entrypoint sh aws/codebuild/dot-net -c bash

    sls -v

![Running sls -v inside of the newly created container](https://cdn-images-1.medium.com/max/2000/1*3Pz6k_S853_zPlgq6iVrYQ.png)*Running sls -v inside of the newly created container*

Next we’ll create a repository in ECR using the AWS CLI. After the command has run the new repository will be visible in the AWS Console:

    aws ecr create-repository --repository-name codebuild-dotnet-node

![Response from the AWS CLI after creating the repository in ECR](https://cdn-images-1.medium.com/max/2106/1*1JLPeLm3SiZLt_ZtARorDQ.png)*Response from the AWS CLI after creating the repository in ECR*

![The newly created repository in ECR](https://cdn-images-1.medium.com/max/2414/1*hkS6gNG06xpRfJC-hx1rEw.png)*The newly created repository in ECR*

Now we’ll tag the *aws/codebuild/dot-net* image we created earlier with the repositoryUri value from the previous step:

    docker tag aws/codebuild/dot-net <ACCOUNTID>.dkr.ecr.ap-southeast-2.amazonaws.com/codebuild-dotnet-node

Run the get-login command to get the docker login authentication command string for your container registry:

    aws ecr get-login --no-include-email

![The login command to authenticate into ECR](https://cdn-images-1.medium.com/max/2106/1*eMugUnEU5DPwXanby8LTcA.png)*The login command to authenticate into ECR*

Run the docker login command that was returned by running the get-login command in the last step.

    docker login -u AWS -p eyJwYXlsb2FkIjoiNGZnd0dSaXM1L2svWWRLMmhJT1c0WWpOZEcxamJFeFJOK2VvT0Y5[...] [https://<ACCOUNTID>.dkr.ecr.ap-southeast-2.amazonaws.com](https://881539945095.dkr.ecr.ap-southeast-2.amazonaws.com)

If login is successful, we can now push our docker image to the repository created in ECR. It may take a few minutes depending on the size of the completed image:

    docker push <ACCOUNTID>.dkr.ecr.ap-southeast-2.amazonaws.com/codebuild-dotnet-node

![Docker on EC2 is creating our image](https://cdn-images-1.medium.com/max/2106/1*LPCnQNhaGVcm3AFo08QPcg.png)*Docker on EC2 is creating our image*

![The Docker image in ECR](https://cdn-images-1.medium.com/max/2414/1*LIXpfP-6u0bRMJwSzlZ8Vg.png)*The Docker image in ECR*

Once the image has been created, we’re going to allow anyone to access the image from ECR. Permission should be locked down in a production environment but for this example we’re going to allow it to be open. Navigate to the Permissions tab in the AWS Console, select *Edit policy JSON* and paste in this policy:

    {
      "Version": "2008-10-17",
      "Statement": [
        {
          "Sid": "EnableAccountAccess",
          "Effect": "Allow",
          "Principal": "*",
          "Action": [
            "ecr:BatchCheckLayerAvailability",
            "ecr:BatchGetImage",
            "ecr:DescribeImages",
            "ecr:DescribeRepositories",
            "ecr:GetAuthorizationToken",
            "ecr:GetDownloadUrlForLayer",
            "ecr:GetRepositoryPolicy",
            "ecr:ListImages"
          ]
        }
      ]
    }

### 3. Create your Pipeline

It’s time to build the pipeline. To make this easier and repeatedly deployable, and in the true form of serverless architectures, I’ve built the pipeline using the serverless framework. You could also achieve the same thing by building it in CloudFormation.

I won’t paste the entire source from my serverless.yml file, you can clone it from github instead: [https://github.com/effectivedigital/serverless-deployment-pipeline](https://github.com/effectivedigital/serverless-deployment-pipeline)

Take a look through the serverless template to understand exactly what it will be doing, but in short its going to setup:

* 3x S3 Buckets

* 1x Bucket Policy

* 3x IAM Roles

* 1x CodeBuild Project

* 1x CodePipeline Pipeline

* 1x CloudWatch Event

* 1x CloudTrail Trail

Once cloned, update the *DockerImageArn *to your image in ECR. If you’re going to be creating deployment packages with a filename other than *Deployment.zip*, update *DeploymentFilename *as well:

    **DockerImageArn:** <ACCOUNTID>.dkr.ecr.ap-southeast-2.amazonaws.com/codebuild-dotnet-node:latest
    **DeploymentFilename:** Deployment.zip

That’s it, the pipeline is now ready for deployment. Run the serverless deploy command and wait while everything is setup for you:

    sls deploy -v

![Our CloudFormation stack created by the serverless framework](https://cdn-images-1.medium.com/max/2496/1*I8L9Ygjzb4UxqG96cUEA6Q.png)*Our CloudFormation stack created by the serverless framework*

![The CodePipeline Pipeline created by the serverless framework](https://cdn-images-1.medium.com/max/2496/1*M6GBlFFWtms2N8_q-xe2qg.png)*The CodePipeline Pipeline created by the serverless framework*

![The CodeBuild Project created by the serverless framework](https://cdn-images-1.medium.com/max/2532/1*3ikvbMj8G--AksgzCRMEuw.png)*The CodeBuild Project created by the serverless framework*

### 4. Add buildSpec.yml to your application

When CodePipeline detects a change to the Deployment file in S3, it will tell CodeBuild to run and attempt to build and deploy your application. That said, CodeBuild also needs to know what commands need to be run to build and deploy your application, buildSpec.yml has the instructions which CodeBuild follows.

I’ve created a really simple hello world application which includes the example buildSpec.yml file you can include: [https://github.com/effectivedigital/serverless-deployment-app](https://github.com/effectivedigital/serverless-deployment-app)

Alternatively, you can create a buildSpec.yml file in your existing applications and populate it with the instructions below:

    version: 0.2

    phases:
      pre_build:
        commands:
          - chmod a+x *
      build:
        commands:
          - ./build.sh
      post_build:
        commands:
          - sls deploy -v -s $STAGE

### 5. Testing your Pipeline

Everything is now in place to run your Pipeline for the first time. Create a package called *Deployment.zip *which should include all of the files for your serverless application and the buildSpec.yml file.

After a few moments CloudTrail should log the PutObject event, trigger a CloudWatch Event Rule which will then trigger CodePipeline to start running.

![Deployment.zip has been uploaded to S3](https://cdn-images-1.medium.com/max/3052/1*PnkoJS88aX_F5c7eMDi78g.png)*Deployment.zip has been uploaded to S3*

![CodePipeline has begun and the build is in progress](https://cdn-images-1.medium.com/max/3052/1*i2a6_k2RvwsmhCJ97sl3zQ.png)*CodePipeline has begun and the build is in progress*

If we click into the details of the AWS CodeBuild step, we can look at the the progress of the build and deployment:

![CodeBuild will get the output from the Docker image performing the build and deployment](https://cdn-images-1.medium.com/max/3052/1*RqvRSHoOLUvCbADdHuBizg.png)*CodeBuild will get the output from the Docker image performing the build and deployment*

![Our deployment was successful!](https://cdn-images-1.medium.com/max/3052/1*pPdV9V2r_8EfGWU_O9MdLA.png)*Our deployment was successful!*

The new app which was deployed by our Pipeline is also visible in CloudFormation:

![](https://cdn-images-1.medium.com/max/3052/1*YxCtXXEo56UogezowvZVhw.png)

We can test the API endpoint created in our simple app (URL is in the CodeBuild output or API Gateway) and see that our app is working successfully:

![Using Postman to invoke our API](https://cdn-images-1.medium.com/max/2530/1*V_JQGBh1VXw907fEpH1RZg.png)*Using Postman to invoke our API*

## Summary

CodePipeline allows you to create a scalable, flexible and low cost CI/CD pipeline and helps solve some of the issues associated with traditional pipelines created on servers.

I would love to take this one step further and add unit testing into the mix once the deployment has completed, though it warrants an article of it’s own — something to stay tuned for in the future!

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Join our community Slack and read our weekly Faun topics ⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
