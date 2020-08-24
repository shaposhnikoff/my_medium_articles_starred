Unknown markup type 10 { type: [33m10[39m, start: [33m150[39m, end: [33m161[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m37[39m, end: [33m40[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m246[39m, end: [33m260[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m262[39m, end: [33m275[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m277[39m, end: [33m290[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m140[39m, end: [33m151[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m199[39m, end: [33m214[39m }

# Why We Stopped Using So-Called Best Practices in Our CI/CD Process

Or reasons to avoid npm and deny the Google Play process

![Screenshot of our Kano application](https://cdn-images-1.medium.com/max/2610/1*5ctHE_pU_3W2dWOFypEIqw.png)*Screenshot of our Kano application*

## Preamble

At [Kalisio,](https://kalisio.com/) we develop open-source geospatial software — that’s to say, software that manages geolocated assets but in a more friendly and business-oriented way than GISs usually provide. We’ve built a strong ecosystem composed of various tools and applications, providing dozens of web services to deliver our solutions as SaaS:

* [Kaabah](https://kalisio.github.io/kaabah/), a solution to build and operate Docker Swarm infrastructures

* [Kargo](https://kalisio.github.io/kargo/), a Docker-based solution to deploy geospatial services

* [Krawler](https://kalisio.github.io/krawler/), a minimalist extract-transform-load (ETL) tool

* [Weacast](https://weacast.github.io/weacast-docs/), a platform to gather, expose, and make use of weather forecast data

* [KDK](https://kalisio.github.io/kdk/), a development kit to simplify building geospatial web applications

* [Kano](https://kalisio.github.io/kdk/api/kano/), a map and weather-forecast data explorer in 2D/3D

* [Akt’n’Map](https://aktnmap.com/), an application to manage real-time events on the field

![High-level view of our platform](https://cdn-images-1.medium.com/max/2000/0*q2v4CyX97Iw_ezB2.png)*High-level view of our platform*

Applications like [Kano](https://kalisio.github.io/kdk/api/kano/) or [Akt’n’Map](https://aktnmap.com/) have been developed as a set of loosely coupled modules using the [KDK](https://kalisio.github.io/kdk/) to prevent building a [monolithic piece of software](http://whatis.techtarget.com/definition/monolithic-architecture), ensure [s](https://en.wikipedia.org/wiki/Separation_of_concerns)eparation of concerns (SoC), and ease maintenance, at least from a source-code perspective.

Moreover, we manage dedicated infrastructures with different instances (e.g., configuration) of these solutions to be able to simultaneously run our own production version, beta version, and alpha version — in addition to customized versions for some of our customers. As a consequence, we need to rely on a predictable and mostly automated process to release our applications.

## CI/CD pipeline

The main purpose of the continuous integration and deployment (CI/CD) pipeline is to create/build application artifacts (Docker images for web applications and mobile-application bundles) and deploy them into production-like environments in order to use/test them.

Our CI/CD pipeline is illustrated in the following schema.

![Our CI/CD pipeline](https://cdn-images-1.medium.com/max/2108/0*mZinOP-LKi1XEVsW)*Our CI/CD pipeline*

The different operations performed by each stages are the following:

* **BUILD**: Retrieve source code to create the Docker image to run the web app

* **TEST**: Run back-end and front-end tests using the app image

* **DEPLOY**: Deploy the web app on the target infrastructure using the app image

* **ANDROID**: Build the Android APK and deploy it to Google Play

* **IOS**: Build the iOS IPA and deploy it to App Store Connect

So far, so good — it seems pretty straightforward, doesn’t it?

However, as explained hereafter, we had a hard time making it work smoothly under the hood.

## When Some Best Practices Break

### Package manager

[npm](https://nodejs.org/en/knowledge/getting-started/npm/what-is-npm/) is the de facto solution to manage modules with Node.js. It’s simple and works great for independent production-grade modules.

However, it has some drawbacks when it comes to managing packages during development, particularly on a large ecosystem.

First, the process to use unpublished (e.g., under-development) packages is quite different from the process to use released packages. While a simple npm installis sufficient in the second case to get your full dependency tree, you need to clone all repositories of your dependency tree that are currently under development and [link](https://docs.npmjs.com/cli/link) them in the first case.

Second, as the link process occurs at the global level of your Node.js installation, you can’t easily have multiple versions of the same module in your environment.

Last, when you have a large dependency tree and release deep-level modules, you have to perform an upgrade operation on all the dependent modules, which can be painful. Indeed, if you’d like to have predictable builds. you must rely on [lockfiles](https://docs.npmjs.com/files/package-locks). When dealing with a large code base, this means having to make changes across dozens of repositories.

**Note: **Due to some [changes](http://codetunnel.io/npm-5-changes-to-npm-link/) in the way npm manages linked modules, we actually prefer to use [Yarn](https://yarnpkg.com/) as a package manager, but this doesn’t restrict the scope of this analysis.

### App stores

App stores are really designed for native apps and not web-based apps, where you have a physical separation between the front end (e.g., client) and the back end (e.g., server).

As a consequence, the natural process of releasing an application is to promote the same app bundle (e.g., APK or IPA) from one publication state (e.g., private alpha release) to the next publication state (e.g., public beta release). This means you can’t change the default configuration of your package depending on the publication state.

As your client app should be able to at least run without being connected to your server (because the user doesn’t point a browser to the right server URL, in this case), it’s, for instance, natural to make the target server URL part of its configuration.

But if you have different endpoints for your different versions, then you have to rely on a fixed third-party server to relay requests (not simple) or a built-in GUI allowing users to change the target server URL (error-prone and not really user-friendly). More challenging is you can have only one version of an app installed at any point of time, since all of these share the same application ID. If some of your users are using the alpha version, they can no longer use the production one.

### **Sensitive data**

As an application relies on third-party services, its configuration must include sensitive data like API keys, SSH credentials, etc. to access the required services. Of course, this shouldn’t be pushed under source control if your repositories are public.

First, you can use [encrypted environment variables](https://docs.travis-ci.com/user/environment-variables/) set either in the build file or the repository settings. When you have a lot of environment variables, it starts becoming tricky and not flexible — as changing a value requires additional operations to encrypt data.

Moreover, this doesn’t work well when you have different configurations of the same application, like for alpha, beta, and production versions. You have to duplicate** **each variable for each available configuration using a naming convention (e.g., VARIABLE_ALPHA, VARIABLE_BETA, VARIABLE_PROD).

Last but not least, this doesn’t work if some of your secrets aren’t environment variables but rather files. In this case, you can create a secrets.tar containing all secured files and [encrypt](https://docs.travis-ci.com/user/encrypting-files/) it to secrets.tar.enc using [Travis CLI](https://kalisio.github.io/kdk/tools/cli.html#travis-cli). This file will be decrypted before the build or whenever you need something inside.

However, the same restrictions apply: You have to duplicate each file for each available configuration using a naming or path convention, and changing any file requires additional operations to encrypt data. Moreover, you still need to find a secured data store for the raw secret files.

### Front end and back end separation

This conceptual split has evolved into specialized developer roles for each, which is still the norm throughout the industry.

[Among many](https://www.thoughtworks.com/insights/blog/dividing-frontend-backend-antipattern), one of the most fallacious arguments is probably that changes in the back end won’t affect the front end — and vice versa. This is only right if you keep the API backward-compatible. The fact is we develop features, and features are neither back-end or front-end. Most of the time, features are both.

And in this case, effort increases with the division of the front end and back end, particularly if you have a lot of modules. In order to commit any changes, two synchronous commits are made rather than one. In order to release a new feature, the same process is applied twice. When dealing with a large code base, this means having to make changes and actions across dozens of repositories.

Team scalability is also an argument for the separation, but when working with a small team (like we do), it’s better to reduce miscommunications, facilitating smooth application development.

Last but not least, some parts of our modules are written using Isomorphic JavaScript, which run both on the client and the server, so the separation doesn’t make any sense on these parts.

### **SemVer**

A strict use of [SemVer](https://semver.org/lang/fr/) actually requires a public API, which is great for libraries or modules to be integrated in a third-party application but not really for the application itself.

In this case, users expect minor releases to be backward-compatible, with data produced by previous minor versions but with a couple of new features or enhancements in the workflow of existing features. Major releases might break this backward compatibility by removing existing features and defining a new data format.

## How We Solved It By Defining Our Own Best Practices

If you think a little bit about it, we already have a proven and reliable solution to manage multiple versions of our code: Git branches. Our idea was to stick as much as possible to this simple concept and to drive our process accordingly.

1Our CI/CD actually comes in three different flavors, as driven by the value of the current Git branch:

* **dev**: in order to deploy current development/alpha version, linked to the *master *branch of our code

* **test**: in order to deploy current staging/beta version, linked to the *test *branch of our code

* **prod**: in order to deploy current production version, linked to *version tags *on the *test *branch of our code

The output Docker image artifacts use the prerelease SemVer notation to identify which flavor has been used to produce it — e.g., 1.0.0-dev or 1.3.0-test.

2 We’ve built a simple [CLI](https://kalisio.github.io/kdk/tools/cli.html#kdk-cli), which is basically a multiplexer for usual Git/npm/Yarn commands used when developing KDK-based applications.

It allows us to easily clone, install, link, unlink, and switch branches on the application and all dependent modules using a single command.

The CLI relies on what we call a *workspace file*, defining the dependency tree between your application and the modules under development. A really simple sample workspace for our application template can be found [here](https://github.com/kalisio/kdk/blob/master/workspaces/kapp.js) — you can create a new working environment with a couple of commands:

    // Clone all required repositories with the target branch
    kdk your_workspace.js --clone your_branch
    // Install dependencies in all required modules
    kdk your_workspace.js --install
    // Link all required modules according to the dependency tree
    kdk your_workspace.js --link

The key point here is we use the same process whatever the flavor (dev, test, or prod).

**Note:** In production, what we actually clone from is a tag instead of a branch, but from Git’s point of view it’s the same: a specific commit.

3 We’ve declared a different app for each flavor on the different mobile stores. We build hydrid-mobile applications using [Cordova](https://cordova.apache.org/), and each application is only published with the state associated to the corresponding flavor:

* Private alpha version for dev

* Public beta version for test

* Public version for prod

![Our different [Akt’n’Map](https://aktnmap.com/) apps](https://cdn-images-1.medium.com/max/2000/1*1arPEO_OSt29a0hmZBvy9A.png)*Our different [Akt’n’Map](https://aktnmap.com/) apps*

4 We use a private GitHub repository called a *workspace *as a secured storage back end for the application configuration. Each workspace has subfolders for each flavor (dev, test, prod) plus a common* *folder containing configuration files shared by all flavors.

To avoid duplicating too many environment variables, we also merge .env files contained in the common and the flavor subfolders in order to build the final environment.

![Application workspace example](https://cdn-images-1.medium.com/max/2000/1*u6gT_LTvC5rcXzsDtwK5GQ.png)*Application workspace example*

5 We keep front end and back end code together in our modules in order to address a specific set of functional features from the user’s point of view (e.g., profile management, billing management, etc.).

As the final front-end bundle is actually generated when building the application with [webpack](https://webpack.js.org/), we avoid the painful task of managing a front-end build configuration in each module.

As we’re using [Quasar](https://quasar.dev/)/[Vue.js](https://vuejs.org/), we only publish raw component files (e.g., *.vue ) in modules, which are then processed by webpack and [dynamically loaded at run time](https://quasar.dev/quasar-cli/cli-documentation/lazy-loading) in the application.

## Conclusion

We don’t want to say here that our solution is more relevant than well-known approaches, but we pretend that best practices still need to be challenged, particularly when building large ecosystems and complex applications.

Some efforts have already been put into this direction, but there’s obviously no one-size-fits-all solution. For instance, Google built the [repo](https://source.android.com/setup/develop/repo) CLI to simplify working across multiple repositories when developing on Android (something similar to our own CLI).

With Git [submodules](https://medium.com/@porteneuve/mastering-git-submodules-34c65e940407), a repository can utilize code from others repositories but without automatic upgrades and support for branches (it’s just a pointer to a particular commit of the submodule’s repository).

Monorepo tools, like [Lerna](https://github.com/lerna/lerna), partially address the problem by simplifying the physical** **structure of your code base by merging repositories. However, you still have to manage the logical** **structure of your code base by upgrading and releasing the same set of packages. Similarly, [Bit](http://bit/) isolates building blocks (e.g., components or a library) from any repository to be used in any repository, but you still have to manage the logical structure.

We know we didn’t reach the perfect solution. However, the key lesson we’ve learned from our experience is the following:

Whatever tool you use, if you have to perform individual actions for each logical component to release your application, it’ll come with huge overhead.

In our case, we don’t need to execute specific actions to release, except the actions already performed every day by developers with Git: commit to, merge, and tag branches. Another benefit is completely segregated environments, from infrastructures to app bundles, which greatly simplifies the application life cycle and avoids polluting (pre)production with development-related issues. Of course, our approach also has drawbacks:

* Not using npm for releases is counter-intuitive, and linking modules leads to duplicated dependencies in production ;

* Managing multiple infrastructures, applications, etc. makes the configuration part more complex

* Using dynamic imports, chunks will be created for each component file, even if some aren’t used in the final application

* Without a build process for front-end files in modules, we only perform front-end testing at the application level

We look forward to hearing your feedback and solutions.
