
# Terraform for teams, using a remote state backend with workspaces

Part 1

![](https://cdn-images-1.medium.com/max/2000/1*wETGs7GFKLrF12SOYvsjiw.png)

### Preface

After not having been involved with setting up infrastructure using Hashicorp’s [Terraform](https://www.terraform.io/) for 18 months or so, I have been keen to dive in and see what has changed since I last used it.

Using Terraform within a team, with more than one deployment environment was problematic in the past, especially if you did not use Atlas, the Hashicorp paid service. Managing state across these environments was a pain. Some new changes that have been made since I last used Terraform aim to change all this, such as [workspaces](https://www.terraform.io/docs/state/workspaces.html) and [remote backends](https://www.terraform.io/intro/getting-started/remote.html) which look good and would negate the need for a project I had originally written to solve the problem, [terraform_exec](https://github.com/nadnerb/terraform_exec).

A basic knowledge of AWS will help with the examples that follow.

### Getting started

Before you get started, if you are keen to follow along, make sure you have the latest version of Terraform [installed](https://www.terraform.io/intro/getting-started/install.html) (at the time of writing v0.11.1). If you are using a mac you can use Homebrew.

I have created a simple [Terraform template project](https://github.com/nadnerb/terraform-base) which can been used for the examples. I usually have a base project that I build the underlying infrastructure such as AWS VPCs. In this example the base project simply creates an [internal Route53 zone](http://docs.aws.amazon.com/Route53/latest/DeveloperGuide/hosted-zones-private.html). This will be used in subsequent posts in this series.

### Setting up the remote backend

First up, we store all our [Terraform state](https://www.terraform.io/docs/state/) in AWS S3. To follow the example you will need to setup a private bucket with an AWS account you have access to. There are a few different options for this if you do not want to use S3, please see the Terraform documentation on remote state for more info.
> Workspaces, which I will talk about shortly, can use multiple AWS accounts for deploying infrastructure. Your production environment is likely a different account from a demo environment. However, the remote backend should always use the same AWS account.

Run the following command substituting the partial backend variables. More information about partial backend configuration can be found [here](https://www.terraform.io/docs/backends/config.html).

    terraform init -backend-config=”access_key=your-aws-key” -backend-config=”secret_key=your-aws-secret” -backend-config=”region=your-aws-region" -backend-config=”bucket=star-wars-infrastructure” -backend-config=”key=base”

This will update Terraforms configuration to always push our state to S3. In the example, the bucket used is called star-wars-infrastructure. You will need to use a different bucket name as they are globally unique.

If you want to use a different name for the project, change backend-config=”key=something-else”*.*

If after running the terraform init command everything completes successfully, it will also download the corresponding AWS Terraform plugin.

### Setting up our workspace

Setting up a workspace is very straight forward.

    terraform workspace new rebel-alliance

You should see the following output.
> Created and switched to workspace “rebel-alliance”!

To display all available workspaces.

    terraform workspace list

Example output.
> default
* rebel-alliance

The * indicates that we are currently on the workspace rebel-alliance.

### Setting up our infrastructure

I have been using AWS profiles to manage account secrets, there are a few options available. AWS profiles are fairly simple to use.

Generally, each workspace will have specific configurations. In this simple example we only have two, zone_name and the vpc-id. I am still using a file for consistency as most projects are more complex. You can, however, pass these to the terraform command being run on the command line using -var zone_name='rebel-alliance'.

Running a terraform plan, as the name implies, will show what Terraform intends to change when applying a real infrastructure update.

    terraform plan -var aws_profile=rebel-alliance -var-file=workspace-variables/rebel-alliance.tfvars

In the previous example I have workspace variables in the workspace-variables directory.

Here is an example of the contents of our rebel-alliance.tfvars.

    zone_name = “rebel-alliance”
    vpc_id = “vpc-123456”

After running the terraform plan command, you should get output including:
> Plan: 1 to add, 0 to change, 0 to destroy.

If this is the case you can then apply the changes.

    terraform apply -var aws_profile=rebel-alliance -var-file=workspace-variables/rebel-alliance.tfvars

You should see something like the following:
> Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

The state of the workspace should be available on S3. Go to the bucket you created, you should see an *env:* directory, within it you should see a folder with your workspace name, in our case, rebel-alliance. Within that folder there should be a state file with the name of the project, in the example we used base.

### Switching workspaces

Generally teams have multiple deployment environments, this is where using workspaces really shines. To switch to another environment, simply repeat the steps from the *Setting up our workspace. *Switching workspaces is as simple as:

    terraform workspace select the-empire

Now, when using Terraform commands such as apply and destroy that result in a change in infrastructure, the state is all stored in S3. Multiple workspaces will look something like this:

![Our workspaces on S3](https://cdn-images-1.medium.com/max/2876/1*2lPgsUbS4qGMYADAMJ_eBA.png)*Our workspaces on S3*

Individual infrastructure projects can looks something like this within each of the workspaces.

![Infrastructure projects in a Terraform workspace in S3](https://cdn-images-1.medium.com/max/2000/1*0FqavXqqHfdcCY4ZajFjeg.png)*Infrastructure projects in a Terraform workspace in S3*

The workspaces above include our base project that we have created in the rebel-alliance workspace. The clamav-service and ecs-services are projects I will outline setting up in a follow up post.

### Summing up

Terraform workspaces and remote backends look pretty handy so far. In the simple example above you can start managing multiple environments within a team. There are additional features, such as locking of the Terraform state so multiple people can’t modify state at the same time, which is covered in the Terraform documentation [here](https://www.terraform.io/docs/state/locking.html).

Managing project variables is still a bit funky, committing them to the project in a private repo seems the simplest solution at the moment. [This](https://github.com/hashicorp/terraform/issues/15966) feature request talks about possible solutions for the future.

I aim to follow up this article with another on deploying Docker services to ECS using Terraform.

![](https://cdn-images-1.medium.com/max/3840/1*ON4ot1RtHUXeSSa1ZrM-8A.png)
