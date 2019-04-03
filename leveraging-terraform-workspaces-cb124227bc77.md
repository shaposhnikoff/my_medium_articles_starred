
# Leveraging Terraform Workspaces



From Hashicorp:

    Each Terraform configuration has an associated [backend](https://www.terraform.io/docs/backends/index.html) that defines how operations are executed and where persistent data such as [the Terraform state](https://www.terraform.io/docs/state/purpose.html) are stored.

    The persistent data stored in the backend belongs to a *workspace*. Initially the backend has only one workspace, called "default", and thus there is only one Terraform state associated with that configuration.

    Certain backends support *multiple* named workspaces, allowing multiple states to be associated with a single configuration. The configuration still has only one backend, but multiple distinct instances of that configuration to be deployed without configuring a new backend or changing authentication credentials.

Essentially, Terraform workspaces rename state files. In doing this though they provide not only a way to test code without changing anything, but also a very clean way to interpolate environment names into configurations when running Terraform from a CI server. In this example we are using terraform with Gitlab, and Gitlab CI/CD.

When I first started using Terraform, every module required a ${var.environment} value to match up to maps and append to resource names. Really I used it everywhere. Some examples:

Below, we have a lookup to check the environment specific remote state s3 bucket against a map. The idea here was to make a module more reusable as it is promoted through environments.

    data "terraform_remote_state" "vpc" {
      backend = "s3"

    config {
        bucket = "${lookup(var.remote_state_bucket, var.environment)}"
        key    = "${lookup(var.remote_state_vpc_key, var.environment)}"
        region = "${lookup(var.remote_state_region, var.environment)}"
      }
    }

Interpolating environment name in a resource name:

    name = “${var.environment}-instance_1”

Now if the downside is just having to pass in a variable, I wouldn’t be writing this. The downside is a lot of repeated code.

Something like:

    stage
      └ vpc
      └ services
          └ frontend-app
          └ backend-app
              └ vars.tf
              └ outputs.tf
              └ main.tf
      └ data-storage
          └ mysql
          └ redis
    prod
      └ vpc
      └ services
          └ frontend-app
          └ backend-app
      └ data-storage
          └ mysql
          └ redis
    mgmt
      └ vpc
      └ services
          └ bastion-host
          └ jenkins
    global
      └ iam
      └ route53

So each file is more or less repeated in each of the environment directories.

Now even without some kind of CI server using workspaces reduces the amount of repetition and the margin of error.

Our secret weapon is: ${terraform.workspace}. This interpolation sequence will insert the workspace name wherever it us used. Automatically specifying the environment via the workspace, or the git branch, allows us to use less code. The same type of maps can be used, replacing the environment variable with the interpolation sequence:

    data "terraform_remote_state" "vpc" {
      backend = "s3"

    config {
        bucket = "${lookup(var.remote_state_bucket, terraform.workspace)}"
        key    = "${lookup(var.remote_state_vpc_key, terraform.workspace)}"
        region = "${lookup(var.remote_state_region, terraform.workspace)}"
      }
    }

Why is this preferable? Instead of relying on the editor of the terraform configs to ensure that all the code is replicated across environments correctly we get to keep our hands off the code. To ensure that there is no accidental overlap, workspaces will rename state files. With only one copy of the code, we have to do the dance of changing workspaces and branches to work with different versions of the files. Indeed, there is nothing wrong with that, but we still have to remember to change workspaces every time we checkout a branch. The real magic comes when we incorporate this into CI.

In this Gitlab-CI/CD example, I’ve use the git branch as the workspace name, with the exception of master because I want my interpolations to be “prod” as opposed to “master.”

before_script:

    - case "$CI_COMMIT_REF_SLUG" in
    master) WORKSPACE_NAME="prod" ;;
    *) WORKSPACE_NAME="$CI_COMMIT_REF_SLUG" ;;
    esac
    - rm -rf .terraform
    - terraform --version
    - terraform init
    - terraform workspace select $WORKSPACE_NAME || terraform workspace new $WORKSPACE_NAME

This allows development branches and production branches, as well as their matching environments to remain separate with minimal human interference.

Thanks for reading, check out what Ahead is doing with Terraform at the Hashicorp User Group on 12/12/2018.
