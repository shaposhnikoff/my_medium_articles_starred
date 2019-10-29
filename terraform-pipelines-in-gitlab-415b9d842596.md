Unknown markup type 10 { type: [33m10[39m, start: [33m62[39m, end: [33m76[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m15[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m49[39m, end: [33m54[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m72[39m, end: [33m85[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m34[39m, end: [33m48[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m302[39m, end: [33m311[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m255[39m, end: [33m259[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m29[39m, end: [33m43[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m56[39m, end: [33m71[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m211[39m, end: [33m230[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m302[39m, end: [33m316[39m }

# Terraform Pipelines in GitLab

Image credit: https://bit.ly/2JaedFf

If you followed my previous posts and got as far as setting up a Terraform pipeline in Jenkins, a thought may have occurred to you at some point:
> Isn’t there a better CI system than this?

Of course there are many different CI systems, each with their pros and cons, and which one you choose will depend on your own environment, workflow and application needs. Jenkins seems to be an unavoidable demon in a lot of infrastructure stacks: It’s a big ugly Java monolith, difficult to declaratively configure or scale, and its pipeline system seems to use Apache Groovy just to make the task more complex than necessary. But there’s no arguing that Jenkins is also powerful, infinitely configurable and adaptable…

![](https://cdn-images-1.medium.com/max/2000/1*p4Yqglt2-fmLnGanM-vs2g.jpeg)

Thankfully if you’re lucky enough that your particular use case does not require the behemoth of Jenkins, you can use a more modern CI tool such as GitLab CI. First, a brief history.

![](https://cdn-images-1.medium.com/max/2000/1*8sqaOe8WFVnS6KOw-shavw@2x.png)

## What is GitLab?

[GitLab](https://gitlab.com/) is a hosted Git service, much like GitHub. You may have heard of them recently since a certain acquisition and the #movingtogitlab movement. If you don’t like hosted services, GitLab provide a standalone server that can be deployed on premises or in the cloud, and commendably most of the development of their server product is open source.

GitLab have been proponents of DevOps for a long time and bundle a ton of helpful tools into their product, including Kanban boards and a complete CI/CD system. You can even use GitLab for just the CI/CD features if your git repository lives elsewhere.

There are a few fundamental concepts for GitLab’s approach to CI, which make sense when you’re used to them, but may affect your decision to use it for your project:

1. Every repo has a single pipeline configuration, declared in a .gitlab-ci.yml file

1. Every commit to the repo will trigger a run of this pipeline

1. That means no promping a user for variables (if you’re used to parameterized Jenkins jobs this may come as a shock)

## Getting Started

This tutorial assumes you have created a free account at gitlab.com, but you can follow alone with a standalone server as well. Create a new project, and add your SSH key for convenient git usage (GitLab will prompt you to do this).

Next, add some Terraform config to your repo. You can use the example files from my [intro to Terraform post](https://medium.com/@timhberry/learn-terraform-by-deploying-a-google-kubernetes-engine-cluster-a29071d9a6c2) that build a basic GKE cluster. You will also need to set up remote state, which is detailed in [my previous pipelines post](https://medium.com/@timhberry/terraform-pipelines-in-jenkins-47267129ff06).

*This tutorial is really about GitLab pipelines in particular, so if you need any further details please go back and have a quick read through those posts. If you need a refresher on Git itself, [take a look here](http://rogerdudler.github.io/git-guide/).*

Finally, add a .gitignore file that excludes the creds directory that you created (if you followed along with the previous posts). You *do not* want to commit your service account credentials to git! Finally, your local repo should look like this:

    .gitignore
    creds/serviceaccount.json
    backend.tf
    gkecluster.tf
    provider.tf

You can now commit this code and push it to GitLab.

![](https://cdn-images-1.medium.com/max/2000/1*i7WKMNygQ4CRkvren_cZJA.png)

So far this is just a basic git repo. The magic happens when we add our gitlab-ci.yml file. Simply adding this file configures and enables Continuous Integration for our project. When we commit the file, and on any subsequent commit, GitLab will run the pipeline for us. Add this file to your repo for a very basic Terraform pipeline:

<iframe src="https://medium.com/media/b3134f8d578102ed3bcd6fc1333b5b36" frameborder=0></iframe>

Note: If you *don’t* name this file .gitlab-ci.yml it *won’t* be used to configure a pipeline. Let’s walk through some highlights in this file:

    image:
      name: hashicorp/terraform:light

GitLab runs pipelines in what it calls *Runners*. There are various ways to configure Runners, and one convenient option is to use ephemeral Docker containers. This is how Runners are configured for us when we use GitLab.com. This line of configuration specifies that our Runner container should use the terraform image provided by Hashicorp, as it contains the tools we need.

    before_script:
      - rm -rf .terraform
      - terraform --version
      - mkdir -p ./creds
      - echo $SERVICEACCOUNT | base64 -d > ./creds/serviceaccount.json
      - terraform init

This section specifies a brief script that should run before any of our stages. We do some quick cleanup to make sure there are no local conflicting caches, then initialise Terraform, and print out its version number. You’ll notice here that we’re also invoking an environment variable to write our service account credentials file on the fly — more on that in a moment.

    stages:
      - validate
      - plan
      - apply

Here we specify the stages of our pipeline. We want to validate our code, then plan and apply our changes. The following parts of the file just run the appropriate Terraform command to accomplish each of these tasks. One thing to note is this part of the plan stage:

    artifacts:
      paths:
        - planfile

This preserves the file that terraform plan creates for terraform apply to use in the next stage.

## Secret Variables

When the Runner executes Terraform for us, it needs a way to authenticate with Google’s APIs, otherwise it won’t have permission to make any changes to our infrastructure. From previous posts, you should have a serviceaccount.json file that contains the required credentials. However, we don’t want to add this to our repository, or bake it into a Docker image. Don’t forget that if someone obtains this file, they will have permission to edit your project, add or delete resources, and potentially run up a huge bill.

Thankfully, there is a secure way to provide these credentials to our Runner when the pipeline executes. GitLab allows you to store variables and retrieve them from the runtime environment of the Runner. In the GitLab UI, navigate to **Settings > CI/CD** and expand the **Variables **section. We will create a SERVICEACCOUNT variable to match our pipeline configuration. The value of this variable must be a string, so we will encode our service account file:

    cat creds/serviceaccount.json | base64 -w0

Copy and paste the output of this command into the value of the variable and click **Save Variable**. GitLab has a concept of protected variables to limit their use to specific git branches, but that’s beyond the scope of this post.

*You should consider rotating your service account keys, especially when entrusting them to a third party hosted service like this.*

## Run the Pipeline

As soon as you push your commit to GitLab, go back to the web UI and select **CI/CD > Pipelines** for your project, and you’ll see that a pipeline is already running! Click the status of the pipeline to see the stages inside it, and you’ll see that each stage we defined is represented by a job. You can click on these jobs to see the output as the commands are executed inside the Docker container.

![Ready to apply!](https://cdn-images-1.medium.com/max/2000/1*S1PE_iNPQ9bu0pif9bngMw.png)*Ready to apply!*

After a short while, the **Validate** and **Plan** jobs will complete, and the pipeline will pause because we specified that the **Apply** job should be manual. This will force GitLab to wait for you to manually start that job in the pipeline. This gives you a chance to explore the output of the **Plan** job and check that you’re happy with the changes Terraform is proposing.

![Looks good to me!](https://cdn-images-1.medium.com/max/2000/1*AFquNKZqUeUk8kR3c0O9Fw.png)*Looks good to me!*

Then you can click **Play** on the **Apply** job. As if by magic, the job will complete and you will have deployed a GKE cluster via a CI pipeline!

![The completed pipeline in GitLab](https://cdn-images-1.medium.com/max/2000/1*JEw8Pw_wDar_H-NF7YTcuw.png)*The completed pipeline in GitLab*

![Our GKE cluster, declared in code and applied via CI!](https://cdn-images-1.medium.com/max/2000/1*m_zuB_exdQuID4jUKO_3rg.png)*Our GKE cluster, declared in code and applied via CI!*

You now have the basics of a working Terraform CI pipeline in GitLab. Any subsequent changes and commits to your repo will trigger this pipeline, and Terraform will automatically be planned for you — all you need to do is click **Play** on the **Apply** job to accept the changes.

*At this point you may want to delete the GKE cluster you just created to avoid incurring charges.*

## What’s Next?

There are of course more advanced ways to implement this pipeline, depending on your workflow. A common approach I use is to manage my infrastructure code with the [Feature Branch workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow). In a nutshell:

* Changes to infrastructure code are made in new feature branches.

* The pipeline plans and stages changes to a *development* environment, because it knows that this is *not* the master branch.

* If I’m happy with the result of changes to the development environment I create a merge request (GitLab calls the Merge requests; GitHub calls them Pull Requests — they’re the same thing).

* Merges into Master get planned and staged to the *staging* environment, then await approval to make changes to the *production* environment, all as part of the same pipeline (ensuring environment parity).

* Feature branches are deleted once merged into Master.

* The master branch is protected, so you can’t push to it, thereby stopping anyone skipping the *development* environment.

If you think a full tutorial for setting up this advanced pipeline would be useful, please let me know. In the meantime, I would really recommend exploring what GitLab has to offer, and happy Terraforming!

Oh, and [follow me on Twitter](https://twitter.com/timhberry) I guess?
