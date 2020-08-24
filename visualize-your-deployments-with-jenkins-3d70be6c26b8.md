Unknown markup type 10 { type: [33m10[39m, start: [33m136[39m, end: [33m156[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m85[39m, end: [33m87[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m203[39m, end: [33m214[39m }

# Visualize Your Deployments with Jenkins



Have you ever asked yourself or your colleague “Which version is currently deployed on the development environment?” or “Hey John, did you deploy that fix to production yesterday?” or “Bill, our client experienced a bug two days ago. Do you remember which version was deployed at that time?”.

If questions like this regularly pop up, and you use Jenkins for their CI/CD process, this plugin is definitely for you!

In the world of Agile development, we have to update our software applications very often. Each version should be deployed to numerous environments. Eventually, it becomes a mess when we talk about which version is deployed to which environment. It would be nice to have an overall deployment status in one place, right?

At Namecheap, we use Jenkins for CI\CD processes. So we decided to make sure we could always check every deployment status by writing a Jenkins plugin called [Deploy Dashboard](https://plugins.jenkins.io/deploy-dashboard/).

In this article, I will show you the capabilities of the plugin and how to use it.

## Visualizing with Deploy Dashboard

First of all, we wanted to know what code release versions have been deployed to what test and production environments (or devices). To cover this goal we made a custom view that is used as a dashboard.

![Example of the Deployment View](https://cdn-images-1.medium.com/max/3176/0*oiUhczwz8jK9RPaH)*Example of the Deployment View*

Moreover, it is possible to look at release history by clicking on a specific environment.

![History of the deployments to selected environment](https://cdn-images-1.medium.com/max/3200/0*4NHkkkX6gHt24OXw)*History of the deployments to selected environment*

## Getting Started: Add a New Release to Dashboard

Let’s assume that you already have a Jenkins job that builds and deploys your application. The only thing you have to do is to call the addDeployToDashboard method with the environment name and app version parameters.

    properties([parameters([
        string(name: 'version', description: 'App version to deploy'),
        choice(
            name: 'env',
            choices: ['dev', 'prod'],
            description: 'Environment where the app should be deployed'
        )
    ])])

    node {
        //...
        stage("Deploy") {
            // Deploy app version ${params.version} to ${params.env} env
            
            //add release information to the dashboard
            addDeployToDashboard(
                env: params.env,
                buildNumber: params.version
            )
        }
    }

## Create a Dashboard

On the Jenkins main page or folder, click the + tab to start the new view wizard (If you don’t see a +, it’s likely you do not have permission to create a new view).

![](https://cdn-images-1.medium.com/max/3200/0*3Qiz-5T5vToc986G)

On the “create new view” page, give your view a name and select the Deployment View type and click ok.

![](https://cdn-images-1.medium.com/max/3056/0*rU0hI8zW4fpQ50N3)

A regular expression can be used to specify the jobs to include in the view. (e.g.: “.*” will select all the jobs in the folder).

![Configuration of the Deployment View](https://cdn-images-1.medium.com/max/3046/0*OJfT0abyR8MB0GVU)*Configuration of the Deployment View*

## Add Deployment Buttons to Your Build

There are cases when you want to keep your CI pipeline separately from CD one. In this case, the Deploy Dashboard Plugin allows you to add additional buttons to the build sidebar. You should just call a buildAddUrl method with a title and URL address.

    node {
        stage("Build") {
            String builtVersion = "v2.7.5"
            // Build app with ${builtVersion} version

            //Add buttons to the left sidebar
            buildAddUrl(title: 'Deploy to DEV', url: "/job/app-deploy/parambuild/?env=dev&version=${builtVersion}")

            buildAddUrl(title: 'Deploy to PROD', url: "/job/app-deploy/parambuild/?env=prod&version=${builtVersion}")
        }
    }

![Example of the Deployment Buttons on the run details page](https://cdn-images-1.medium.com/max/2208/0*ZpI-19Oo2EbFe6Nb)*Example of the Deployment Buttons on the run details page*

This feature can be extremely useful for the QA team. They will be able to deploy any existing version to their environment just in a few clicks.

I hope it helps to improve your experience with Jenkins! You are welcome to contribute to the project in [GitHub](https://github.com/jenkinsci/deploy-dashboard-plugin).
