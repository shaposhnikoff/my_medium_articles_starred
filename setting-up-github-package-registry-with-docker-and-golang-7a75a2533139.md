Unknown markup type 10 { type: [33m10[39m, start: [33m107[39m, end: [33m110[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m127[39m, end: [33m132[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m54[39m, end: [33m57[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m143[39m, end: [33m156[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m161[39m, end: [33m175[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m263[39m, end: [33m267[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m152[39m, end: [33m163[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m34[39m, end: [33m39[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m44[39m, end: [33m48[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m109[39m, end: [33m121[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m232[39m, end: [33m234[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m264[39m, end: [33m268[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m274[39m, end: [33m279[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m21[39m, end: [33m36[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m41[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m94[39m, end: [33m114[39m }

# Setting up GitHub Package Registry with Docker and Golang

Note: This was originally posted at martinheinz.dev

Generally, for any programming language, to run your application you need to create some kind of package ( npm for *JavaScript*, NuGet for *C#*, ...) and then store it somewhere. In case of *Docker*, people usually just throw their images into *Docker Hub*, but we now have new alternative here...

The alternative is *GitHub Package Registry* — it has been in beta for a while now and it seems that more and more people are getting access to it, so it feels like the time has come to explore its features, here specifically for *Docker* and *Go* projects.

This post is part of the series *“All You Need For Your Next Golang Project”*, if that sound interesting, go ahead and checkout previous part [here](https://gist.github.com/MartinHeinz/134729d0d26ca88b5afb23961759de6e).

*Note: This article is applicable to any project using docker images, not just Golang.*

![](https://cdn-images-1.medium.com/max/2406/1*-Wpfv7kBoBBCb94EWKm8AQ.png)

## Why Use GitHub Registry

First of all, why should you even consider switching from, let’s say, *Docker Hub* or any other registry, to *GitHub Package Registry*:

* If you are already using *GitHub* as your SCM, then it makes sense to use *GitHub Package Registry*, as it allows you to keep everything in one place instead of pushing your packages elsewhere.

* There is another shiny new (*beta*) feature of *GitHub* — *Actions*, which you can leverage in conjunction with *GitHub Package Registry* (more on this in another post…).

* Even though I consider *Docker* images superior to e.g. npm packages, you can also push those to the *GitHub Package Registry*, if you prefer *non-Docker* artifacts.

## Let’s Do It!

So, now, let’s see how to use it. Let’s start with building and tagging your image:

*Note: If you are working with Go, then you might want to checkout [my repository here](https://github.com/MartinHeinz/go-project-blueprint), where all the GitHub Package Registry fun is already tied into Makefile targets.*

<iframe src="https://medium.com/media/10f6472c14480c311a01f667b5e7d0cd" frameborder=0></iframe>

To be able to push images to *GitHub Package Registry*, you need to name it using format shown above — which really is just URL of registry with your *GitHub* username and repository name.

Next, how do we access it?

First, to be able to authenticate ourselves with *GitHub Package Registry*, we need to create personal access token. This access token must have read:packages and write:packages scope and additionally in case you have a private repository, you will have to include repo scope as well.

There is already very nice guide on *GitHub* help website on how to create personal token, so I’m not gonna copy and paste the steps here. You can read about it [here](https://help.github.com/en/articles/creating-a-personal-access-token-for-the-command-line)

Now, that we have personal token let’s login:

<iframe src="https://medium.com/media/5f68434fdd8974cbf4deacd0588b43f4" frameborder=0></iframe>

Finally, time to push our image:

<iframe src="https://medium.com/media/9b09e709f5f0f9502989f13680c49648" frameborder=0></iframe>

And obviously you can also pull the image:

<iframe src="https://medium.com/media/543a67e529fffd180dc74385c3d89d6d" frameborder=0></iframe>

## Use It in Your CI/CD Pipeline

Last thing we might want to to do with *GitHub Package Registry* is to integrate it with CI/CD tools. Let’s look at how it can be done with *Travis* ( *Full .travis.yml can be found in my repository [here](https://github.com/MartinHeinz/go-project-blueprint/blob/master/.travis.yml)*):

<iframe src="https://medium.com/media/36dba30c46024c36c28a116b45f040f5" frameborder=0></iframe>

As you can see above, you can run build and push the same way as on your machine, the only difference is the docker login. Here, we use environment variables specified in *Travis* UI. Username gets passed to the login command through -u parameter and password using echo into stdin, this is needed, so that we don't end-up with our personal *GitHub* token being printed in *Travis* logs.

So, how do we set those environment variables? These are the steps:

* Navigate to your *Travis* job settings for your repository e.g. [https://travis-ci.com/MartinHeinz/go-project-blueprint/settings](https://travis-ci.com/MartinHeinz/go-project-blueprint/settings)

* Scroll down to *Environment Variables* section

* Set variable name to DOCKER_USERNAME and DOCKER_PASSWORD respectively. In case of password (*GitHub* token) make sure, that *Display value in build log* is set to false.

* Click *Add* and trigger the build

If you are not using *Travis* and want to use *GitHub Webhook* to trigger build, then you can use [RegistryPackageEvent](https://developer.github.com/v3/activity/events/types/#registrypackageevent). This could be useful if you are using for example *Jenkins*, *OpenShift* or *Kubernetes* and want to trigger deployment every time your package is published or updated in *GitHub Package Registry*.

## Words of Caution

One thing you should keep in mind when using *GitHub Package Registry* is that you can’t delete packages that you push to registry. This is so that you don’t break projects that depend on your package. You can request package deletion from *GitHub Support*, but you should not count on them actually deleting anything.

## Conclusion

I hope after reading this, you will give the *GitHub Package Registry* a shot. If you don’t have *beta* access yet, you can sign up [here](https://github.com/features/package-registry/signup). In case you have any questions, don’t hesitate to reach out to me or you can also have a look at my [repository](https://github.com/MartinHeinz/go-project-blueprint), where you can find examples of *GitHub Package Registry* usage.

## Resources

* [https://github.com/features/package-registry](https://github.com/features/package-registry)

* [https://help.github.com/en/articles/about-github-package-registry](https://help.github.com/en/articles/about-github-package-registry)
