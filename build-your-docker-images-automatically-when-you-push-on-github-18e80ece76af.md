Unknown markup type 10 { type: [33m10[39m, start: [33m34[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m54[39m, end: [33m64[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m17[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m20[39m, end: [33m30[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m92[39m, end: [33m103[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m257[39m, end: [33m268[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m588[39m, end: [33m599[39m }

# Build Your Docker Images Automatically When You Push on GitHub

Achieving Docker Hub and GitHub integration

![](https://cdn-images-1.medium.com/max/2400/1*HxKyyQhQFKTaZgdmrjQZjA.jpeg)

[*Docker Hub](https://hub.docker.com) integrates with [GitHub](https://github.com) (and [Bitbucket](https://bitbucket.org)), allowing you to automatically build your container’s image when new code is pushed. The tag for your image can be extracted from your repository’s tags (or branches) and automated tests can be executed to ensure your image was built as expected before the image becomes available for download. In this piece I will show you how to quickly setup such a workflow.*

## Create Your GitHub Repo and Add a Dockerfile

The first step is, obviously, to setup your repository on GitHub:

![Creating a public GitHub repository](https://cdn-images-1.medium.com/max/2134/1*X8-itvK_vGTsy4MNyRKaXg.png)*Creating a public GitHub repository*

Next, clone your new repository locally, add a sample Dockerfile, and push the changes back to the repository:

![Adding a default Dockerfile into your repository](https://cdn-images-1.medium.com/max/3472/1*A7eNikcL3ZRn5PzvllNbvA.png)*Adding a default Dockerfile into your repository*

As you can see, the Dockerfile we created is as simple as it gets — just using the official hello-world image.

## Enabling Automated Builds on Docker Hub

To enable automated builds you need to connect your Docker Hub account with your account on GitHub. Login to Docker Hub and go to *Account Settings > Linked Accounts*. Scroll-down to find *GitHub* and click on the *Connect* link. You will be taken through the usual OAuth process where you login to GitHub (if not already logged-in) and authorise Docker to access your GitHub repositories. If everything goes well, you should receive a notification from GitHub:

![Successfully connecting Docker Hub with GitHub](https://cdn-images-1.medium.com/max/2000/1*9y9ZXZheuHuyFJxSpRy_Zw.png)*Successfully connecting Docker Hub with GitHub*

Now it’s time to create the Docker Hub repository and link it to your GitHub repository:

![Creating a Docker Hub repository with GitHub access](https://cdn-images-1.medium.com/max/2844/1*s9CE1TsgQGU57mk5vPkzeQ.png)*Creating a Docker Hub repository with GitHub access*

There are a few things going on here, so let us break them down:

* **Name your repo**
Not much to say here, just using a meaningful name. I opt for the same name as my GitHub repository.

* **Choose GitHub repo**
In the previous steps you connected your Docker Hub to GitHub, so here you should see a list of all your organisations and repositories in them. Choose the repository you created in the previous step.

* **Set up build rules**
Here’s the really interesting part. Since code can be pushed at anytime, you should let Docker Hub know when your image should be build and which tag should have that image assigned to it. You configure both using *Build rules*.

Just click on *Create* for now (do not click *Create & Build* as this will trigger a manual build).

## Build Rules

To let Docker Hub know how and when to automatically build your images you can specify Build rules. You may have multiple rules that apply in parallel, effectively having multiple tags assigned to your images with just one git-push:

![Build rules for tag-based and master branch automated builds](https://cdn-images-1.medium.com/max/2000/1*bM66MA1KMKDTCdL7QpCxkQ.png)*Build rules for tag-based and master branch automated builds*

On the above figure we set up two different rules — let’s see what they do.

### Tag rule

A “tag rule” allows Docker Hub to start building an image upon discovery of a new tag in your git repository. This should probably be your preferred way to build images for the official releases of your image.

As git tags can be arbitrary and contain anything a developer might choose, Docker Hub allows you to define a [Regular Expression](https://regex101.com), in order to identify which part of a git tag should become part of the tag for the Docker image. In the example above we chose /^[0-9.]+$/ as the regular expression with which the image tag is extracted. Practically speaking, this tells Docker Hub that our git tags are eligible for automated builds, should be in the form of numbers with dots, and that should be the only text of the tag. The image tag that’s extracted and assigned to it is denoted by the {sourceref} attribute entered under *Docker Tag*.

### Branch rule

A “branch rule” allows Docker Hub to start building an image upon activity on a specific branch. The typical use case here is to have such a rule to build a *latest* image based on the daily activity of your git repository. The branch to be monitored is entered under *Source* and the tag to be assigned to the Docker images built from it under *Docker Tag*.

## Build your first image

As we now have everything in place, we only need to tag the current state of our git repository and push the tag back to GitHub; Docker Hub will do its magic and build an image for us:

![Pushing a tag to trigger automatic build](https://cdn-images-1.medium.com/max/2748/1*o_IrCZbd8Z48gdTwGjN8yg.png)*Pushing a tag to trigger automatic build*

Indeed, a couple of minutes later Docker Hub has automatically built and tagged the 1.0.0 image:

![The first automated build of release 1.0.0](https://cdn-images-1.medium.com/max/2000/1*KjuasngMgYZ0AnIqW1MwtA.png)*The first automated build of release 1.0.0*

To also see our *latest* image being created automatically we need to push a new change in the *master* branch:

![Pushing a change to *master branch*](https://cdn-images-1.medium.com/max/3600/1*mcdkTt0GAFIs1jMWwkR6UQ.png)*Pushing a change to *master branch**

Again, after a few minutes:

![latest image built from master branch](https://cdn-images-1.medium.com/max/2000/1*UW46SPP1nnRKjZ5dRQSRyA.png)*latest image built from master branch*

Docker Hub provides you two additional screen on which you can monitor your builds:

![Builds screen](https://cdn-images-1.medium.com/max/2794/1*q73pTHUCpqb8jidZTHj9yA.png)*Builds screen*

![Timeline screen](https://cdn-images-1.medium.com/max/2000/1*YG19ubyJOa8ycyhDcxnWew.png)*Timeline screen*

**Other articles in this series
**This article is part of a series on *Docker Hub and automated builds*. Other articles in this series include:
[**How to Test Your Automated Builds on Docker Hub**
*How to test your automated Docker Hub builds*medium.com](https://medium.com/@NMichas/how-to-test-your-automated-builds-on-docker-hub-e40879f35d1e)
[**How to Recover From a Failed Automated Docker Hub Build**
*Does your Docker Hub automated build fail? Here is a quick guide on how to recover it.*medium.com](https://medium.com/@NMichas/how-to-recover-from-a-failed-automated-docker-hub-build-8b6c1cc3d7d4)
