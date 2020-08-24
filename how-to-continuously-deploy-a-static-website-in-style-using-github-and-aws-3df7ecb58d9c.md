Unknown markup type 10 { type: [33m10[39m, start: [33m106[39m, end: [33m123[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m164[39m, end: [33m190[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m177[39m, end: [33m193[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m40[39m, end: [33m53[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m179[39m, end: [33m192[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m175[39m, end: [33m181[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m315[39m, end: [33m319[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m51[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m119[39m, end: [33m134[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m210[39m, end: [33m221[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m13[39m, end: [33m27[39m }

# How to continuously deploy a static website in style using GitHub and AWS



In this post we are going to learn how to use AWS CodePipeline and CodeDeploy to automatically retrieve the source code for a static website from GitHub and deploy that website onto S3.

We will configure the deployment to happen on any new commit to our master branch.

We begin by creating a code pipeline with a source stage linked to our GitHub repository. When a new commit is pushed to our master branch, the pipeline automatically checks out the latest code. We can then trigger a build step in our pipeline. This step can install dependencies, run tests, and package our site for deployment. Our final step then deploys our static website to our S3 static website bucket.

Now that we know what we are going to build, let’s jump in and learn even more by building this out.

**AWS CodePipeline Prerequisites**

To stand up an AWS CodePipeline in our account that communicates with our GitHub repository, there are some prerequisites that you need to take care of.

1. You should have an AWS account already setup.

1. You should have CLI access configured for your account.

1. Your static website should already be hosted out of AWS S3. If not, [check out this link](https://medium.freecodecamp.org/how-to-host-a-website-on-s3-without-getting-lost-in-the-sea-e2b82aa6cd38).

## Configuring GitHub and AWS Communication

In order for AWS to be able to poll for changes to our master branch in GitHub, we need to be able to generate an access token for our GitHub repository. We can generate a personal access token by completing the following steps from within GitHub.

1. While logged into GitHub, **click your profile photo** in the top right, then click **Settings**.

1. On the left, click **Developer settings**.

1. On the left, click **Personal access tokens**.

1. Click **Generate new token** and enter **AWSCodePipeline** for the name.

1. For **permissions**, select **repo**.

1. Click **Generate token**.

1. Copy the token somewhere so we can use it later.

![](https://cdn-images-1.medium.com/max/12032/1*r4nblgja-EQOiJt12au7FQ.jpeg)

## Creating Our CodePipeline By Hand

The first thing we need to provision is CodePipeline. Our pipeline is going to consist of two stages, a **Source** stage connected to GitHub, and a **Build** stage that deploys our static website.

Let’s go ahead and create our CodePipeline via the AWS Console:

1. Navigate to **CodePipeline** in the AWS Console.

1. Click **Create Pipeline**.

1. Enter a **name** for your Pipeline.

1. Select **GitHub** as the **source provider**.

1. Click **Connect to GitHub**. This will open a separate window where you sign into your GitHub account. Once signed in, you must grant repo access to AWS CodePipeline. This is the communication link between your GitHub repo and CodePipeline.

1. **Select the repository** you want to use in this Pipeline.

1. Enter **master**, or your default branch, in the Branch input.

1. Click **Next**.

1. For the Build provider we are going to **choose AWS CodeBuild**.

1. Select **Create a new build project**.

1. **Enter a name** for your Build project.

1. For the **Environment image** we will use an image provided by AWS CodeBuild.

1. Select **Ubuntu** as the operating system.

1. Select **Node.js** as the Runtime and **nodejs:6.3.1** as the Version.

1. Leave Build specification as the **buildspec.yml option**.

1. In the CodeBuild service role section, we want to **create a new service role**.

1. **Enter a name** for the service role CodeBuild will use.

1. Leave the rest of the values at their **default** settings.

1. Click **Save build project** and then click **Next**.

1. For Deployment provider we want **No deployment**.

1. Click **Next**.

## Who likes clicking buttons? Not me.

That was a lot of button clicking right? Could you do that again without looking at all 21 steps? I know I couldn’t.

Good news! There is a far better way of creating and managing your code pipelines, or any AWS infrastructure for that matter. You may have heard of the term Infrastructure-as-Code, and it is pretty much exactly as it sounds. Represent your infrastructure as code so that you can create, maintain, and destroy it without ever opening a GUI.

There is nothing wrong with starting with the GUI if you’re new to AWS, or any new cloud provider. But we want to aim for automation as we scale.

There are many tools out there that make this very easy to do. AWS provides CloudFormation, which allows you to define your resources inside of JSON or YAML templates.

CloudFormation is great but there are other tools out there as well. One I have been using a lot recently is [Terraform](https://www.terraform.io/). It is cloud provider agnostic and supports a variety of providers via community developed modules.

For this blog post, I put together a quick Terraform template that provisions our AWS infrastructure.

<iframe src="https://medium.com/media/ab58903454fe28b4e9845fc72683e4a0" frameborder=0></iframe>

Let’s take a quick journey through what this template is doing.

At the top, we are defining the variables to be passed into the template. To provision the resources, we need to pass in the following:

* name of our pipeline

* our GitHub username

* our GitHub token from earlier

* the GitHub repository we want to link to our pipeline

Than we specify that we want to use AWS as our provider with the region passed in as a variable. As we will see in a minute, this loads a provider from Terraform that supports most AWS resources. You can checkout what AWS resources Terraform supports [here](https://www.terraform.io/docs/providers/aws/).

The next set of resources we are creating are for our AWS CodePipeline.

* We create an S3 bucket that will hold the artifacts/outputs from each stage in our pipeline.

* We need to create an IAM policy that allows CodePipeline to assume a role we create here in our template, codepipeline_role. That role has a policy attached to it, attach_codepipeline_policy. The policy grants access to AWS services that we need to call during an invocation of our pipeline.

* We configure the resources needed in order for CodeBuild to work as expected. We define an assume role policy that allows CodeBuild to assume a role and access services via the codebuild_policy.

* We create our actual CodeBuild project, build_project, that runs the build stage of our CodePipeline. Notice here we specify the source to be codepipeline and our buildspec to be buildspec.yml.

To provision our CodePipeline, we assign the artifact store to be the S3 bucket we provisioned earlier. The role is the codepipeline role that we defined earlier as well. The Source stage uses the GitHub provider and the token we generated over on GitHub. We want to send the output of this stage to a label called code via the output_artifacts property.

The last stage in our CodePipeline resource is the Build stage. Here the provider is CodeBuild and we have defined our input_artifacts to be the code output_artifacts from our Source stage. Then we specify the ProjectName for the CodeBuild project that will be responsible for executing the Build stage.

Everything we need to provision for our continuous deployment pipeline is in this template. If you are just looking to get up and running with AWS, then configuring this by hand might be faster than writing your first Terraform template. But in the long run, defining your infrastructure as code has massive benefits:

* Your infrastructure definition lives in source control. It can be iterated on as code would be.

* Your infrastructure is now repeatable. If you need to move it to another AWS region, you can run the template in that region.

* You can quickly make changes by changing the template and applying updates.

Now that we know what this template is doing and we have Terraform installed, we can run this template from the command line.

First we run terraform init from the directory where our template lives. This pulls in the dependencies Terraform needs to run the template.

![](https://cdn-images-1.medium.com/max/3064/1*S53nYUCEa1YJ2cjmchFwVA.png)

Once our Terraform template has been initialized we can use the “plan” command in order to see what exactly is going to be created.

![](https://cdn-images-1.medium.com/max/4512/1*4fu76COEoAplllSEIndAVA.png)

Notice everything in green? These are the resources that will be created. If there were resources in yellow, these would be resources that were going to be updated. Then if there were resources in red, they would be deleted.

You can then run the apply command to actually create everything in the template. There is a confirmation prompt, simply type in yes.

![](https://cdn-images-1.medium.com/max/4548/1*I4HplNYdKsO4PIzCyAPQfw.png)

Once the template has completed, we should see that all of our AWS resources have been created to support our continuous deployment pipeline.

<iframe src="https://medium.com/media/f08e7ca6099477063182dfa5c48a12f1" frameborder=0></iframe>

## Wait, what did we just do?

That is a lot of steps and quite a bit of infrastructure we stood up in AWS. So let’s walk through at a high level what we just created.

At the top of our pipeline we have the Source, in our case this is our GitHub repository. We configured our pipeline to periodically poll the master branch of our repository. If there are new commits in the master branch, then the pipeline activates to kick off a new build process. This is what is often referred to as a trigger for our build pipeline.

When a new build starts, our pipeline checks out the latest commits from the master branch. Once the changes are checked out, they are passed to the next step in our pipeline, the Build step. For this step, we are using another AWS service, CodeBuild. We have configured our CodeBuild project to use a Node.js image provided by Amazon. This image comes with Node.js installed already so the build machine that builds our repository has access to it.

But how does AWS CodeBuild know how to build our repository? That is where the buildspec.yml comes in. This is a special file that we will put at the root of our repository. In it we configure different phases of the build process like pre_install, build, and post_build. For our use case we are just going to configure the build process in the buildspec file. This will consist of copying the contents from our Source to our S3 website bucket, effectively deploying our static website.

Let’s jump over to our static website repository and configure our buildspec file.

## Setting up our buildspec file

We are going to begin by adding a buildspec.yml file to the root of our static website.

This file is going to be the template that AWS CodeBuild uses to build, and in our case, deploy our static website. It consists of pre-install, install, build, and post build stages. For our use case, we are going to leverage the build stage.

<iframe src="https://medium.com/media/00361547470e6079165f209e143e2ba0" frameborder=0></iframe>

What we are doing in the above build specification is pretty straightforward, it is just one line in fact. We are taking the contents of our static website and copying it to the S3 bucket that hosts our site via the AWS command line interface. For the other steps, we are echoing out which step ran in our build process.

Of course we could do even more here if we had a need to do so. For instance, if we needed to run a build process in our package.json, then the build and post_build steps would look like this:

<iframe src="https://medium.com/media/3f1ae8562cfe5dac678e02020d2a819e" frameborder=0></iframe>

Now we are running npm build inside our build step and saving the s3 sync command for our post build step. Our buildspec is giving us the ability to script not only deployments of our site, but how they are built and tested as well. We could also leverage the other stages like install to add any dependencies our build process needs.

For now let’s stick with our original buildspec file that is copying our static site to S3. Make sure that it is at the root of your repository, as this is where CodeBuild will look for it. Check it into your repository so we can trigger our CodePipeline.

## Triggering our CodePipeline

Earlier we linked the Source stage of our CodePipeline to the GitHub repository of our static website. It is configured to watch for changes in our master branch. So, any new changes pushed to that branch trigger a new CodePipeline run. As we just checked in our buildspec.yml file, we should now see an invocation of our CodePipeline running.

![](https://cdn-images-1.medium.com/max/2000/1*koSBSO6UUPW6pesMuH4ASQ.gif)

We see here that our Source stage has been invoked due to the new changes in the master branch of our repository. This completed and sent its artifacts to the Build stage. The build stage took those artifacts and ran the buildspec.yml file to deploy them to our S3 bucket.

If we were to click on the **details** link on our DeployToS3 stage, we can see the logs that our build process is outputting.

![](https://cdn-images-1.medium.com/max/2760/1*A1rVNAexO3oSjvgDozFQlA.png)

Once our DeployToS3 stage has succeeded we should then be able to reload our static website in the browser and see our changes.

<iframe src="https://medium.com/media/482ca7466d46a8024d65a5cae65817bc" frameborder=0></iframe>

## Bam! We have continuous deployment

Channeling my inner Emeril here, we now have continuous deployment for our static website. With any new commit to our master branch, a new CodePipeline run is triggered. This checks out the latest code from GitHub and passes it to CodeBuild. Our build project then executes what is in our buildspec file.

Currently, our buildspec file is just copying the contents of our static website to our S3 bucket. But, we could extend this to do more things. We could run npm tasks to build our site or run tests. If we are also using CloudFront in front of our static website we can issue an invalidation request when we deploy our new site.

There is so many things you can learn by diving in and actually using AWS. A static website might seem like a simple use case, but it is awesome for learning a wide variety of things.

## Hungry to learn more Amazon Web Services?

I have been using AWS for over six years now and I am always learning new services and new ways to use existing ones. It is a massive platform with a lot of documentation. But, there are times when that documentation can feel like a massive sea of information. To the point to where you get lost in it.

![](https://cdn-images-1.medium.com/max/4000/1*VqpqxIxmMWTvRzKraxQOKg.png)

Inspired by this problem, I recently released an ebook and video course that cuts through the sea of information. It focuses on hosting, securing, and deploying static websites on AWS. The goal is to learn services related to this problem as you are using them. [If you have been wanting to learn AWS, but you’re not sure where to start, then check out my course.](https://www.kylegalbraith.com/learn-aws)

### 👏 If you enjoyed this, don’t forget to offer claps to show your support!
