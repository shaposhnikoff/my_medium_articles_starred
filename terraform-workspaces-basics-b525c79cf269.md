Unknown markup type 10 { type: [33m10[39m, start: [33m41[39m, end: [33m55[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m91[39m, end: [33m108[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m31[39m, end: [33m45[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m46[39m, end: [33m59[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m22[39m, end: [33m39[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m40[39m, end: [33m53[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m137[39m, end: [33m149[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m70[39m, end: [33m92[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m61[39m, end: [33m69[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m71[39m, end: [33m76[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m223[39m, end: [33m234[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m85[39m, end: [33m92[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m157[39m, end: [33m171[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m205[39m, end: [33m212[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m237[39m, end: [33m254[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m272[39m, end: [33m282[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m64[39m, end: [33m71[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m76[39m, end: [33m86[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m117[39m, end: [33m124[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m215[39m, end: [33m222[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m227[39m, end: [33m237[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m325[39m, end: [33m332[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m403[39m, end: [33m410[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m475[39m, end: [33m482[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m487[39m, end: [33m497[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m49[39m, end: [33m60[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m92[39m, end: [33m100[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m20[39m, end: [33m29[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m7[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m70[39m, end: [33m85[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m106[39m, end: [33m125[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m7[39m, end: [33m15[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m54[39m, end: [33m61[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m92[39m, end: [33m101[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m28[39m, end: [33m41[39m }

# Terraform Workspaces Basics

Terraform workspaces are a great way to separate resources by stage. For instance, one might have Terraform defining infrastructure across multiple distinct stages (or AWS Accounts) each with their own stage-specific configuration. In this article we’re going to look at one way to use Terraform workspaces to handle multi-stage deployments.

Consider the following simple CodePipeline project:

![CodePipeline with build stage, sandbox deployment stage, and production deployment stage, each provided by CodeBuild projects.](https://cdn-images-1.medium.com/max/2000/1*CMYLxfc_T27K4ZAS775AdQ.png)*CodePipeline with build stage, sandbox deployment stage, and production deployment stage, each provided by CodeBuild projects.*

In a deployment pipeline flow like this, deploy-sandbox might deploy to one account, while deploy-production deploys to another. These two accounts might even be entirely different to the account in which the pipeline runs. In this case, we may want stage-specific configuration, which can easily be accomplished through the use of [Terraform workspaces](https://www.terraform.io/docs/state/workspaces.html).

A workspace is itself a sub-categorisation of Terraform’s state store. That is to say that Terraform separates state by workspace. In this way, we are able to create resources and their states for each stage, respectively.

For example, in the associated deploy-sandbox buildspec.yml file, we might have something like:

    phases:
      install:
        commands:
          - cd $CODEBUILD_SRC_DIR
          - cd terraform
          - TF_WORKSPACE=sandbox terraform apply -auto-approve

And conversely in the deploy-production buildspec.yml file:

    phases:
      install:
        commands:
          - cd $CODEBUILD_SRC_DIR
          - cd terraform
          - TF_WORKSPACE=production terraform apply -auto-approve

Provided these Terraform workspaces have been created on the remote backend, Terraform will apply conditional configuration based on the TF_WORKSPACE.

Access to the workspace name within Terraform is provided through the ${terraform.workspace} variable:

    resource "aws_s3_bucket" "bucket" {
      bucket = "${var.bucket-name}-${terraform.workspace"
      acl    = "private"
    }

This would create an S3 bucket named for the current stage ( -sandbox, -prod). The same logic can of course be applied to all resource names or tags as needed, though it becomes more valuable when combined with Terraform’s assume_role functionality.

Going back to the CodePipeline outlined above, let’s assume CodePipeline runs in the control AWS account, along with all other build-specific resources. The deploy-sandbox stage deploys resources into the sandbox AWS account,and likewisedeploy-production deploys into the production AWS account.

In order for Terraform to be able to provision resources in the sandbox and production accounts, the IAM Role in the control account attached to the CodePipeline stages would need to be able to assume a role in the sandbox and production accounts. This can be achieved by configuring an IAM role in those two accounts with a control account role as the Principal. By configuring CodePipeline to use the control account role, it is then able to assume the associated roles in sandbox and production.

Terraform enables this functionality through the assume_role configuration block within the provider configuration:

    provider "aws" {
      region = "${local.aws_region}"
      assume_role {
        role_arn = "${local.role_arns[terraform.workspace]}"
        session_name = "${local.project_name}-${terraform.workspace}"
      }
    }

With the associated role_arns local variable:

    locals {
      project_name = "example-project"
      role_arns = {
        "sandbox" = "arn:aws:iam::<sandbox_account_id>:role/buildrole
        "production" = "arn:aws:iam::<prod_account_id>:role/buildrole
    }

So the assume_role.role_arn interpolation would lookup the key in the local.role_arns map for the current terraform.workspace .

In the provider section above, we’re not specifying a profile . This is because Terraform will automatically use the CodePipeline service role.

There are many useful ways to utilize Terraform workspaces — we can use maps similar to the role_arns local variable above in order to specify workspace-specific configuration parameters for a number of different use-cases, such as mapping various code repositories to a specific stage:

    stage_to_branch_map = {
      "sandbox" = ["dev", "alpha", "beta", "qa"]
      "production = ["master"]
    }

Or setting a stage-specific desired_count for ECS or EC2 target groups:

    desired_count_map = {
      "sandbox" = "3"
      "production" = "6"
    }

While we could technically use per-line conditional statements:

    resource... {
      desired_count = "${terraform.workspace == "sandbox" ? "3" : "6"}"
    }

Using maps is far cleaner, and doesn’t limit us to a maximum of two options, especially if we want to do stage+branch-specific configuration:

    locals {
      branch_count = "${length(var.branches)}"
      stage_config_map = {
        "sandbox" = {
          "dev" = {
            "desired_count" = "3"
            "health_check_path" = "/dev-health"
            "port" = "8080"
          }
          "alpha" = {
            "desired_count" = "4"
            "health_check_path" = "/alpha-health"
            "port" = "8081"
          }
        }
        "production" = {
          "master" = {
            "desired_count" = "6"
            "health_check_path" = "/health-check"
            "port" = "80"
          }
        }
      }
    }
    variable "branches" {
      default = ["dev", "alpha", "beta"]
      type = "list
    }

Using the above data structure, we can build unique configuration for multiple branches:

    resource... {
      count = "${local.branch_count}"
      desired_count = "${local.stage_config_map[terraform.workspace][element(var.branches, count.index)]}"
    }

There are probably better ways to handle the above requirement; I just wanted to provide it as an obvious example of a use-case for this sort of mapping.

So, we’ve figured out use Terraform workspaces to manage stage-specific configuration, and how to configure IAM roles to enable Terraform to perform cross-account deployments — we also found some cool ways to use workspace mapping variables to manage stage-specific configuration. Leave any questions below!
