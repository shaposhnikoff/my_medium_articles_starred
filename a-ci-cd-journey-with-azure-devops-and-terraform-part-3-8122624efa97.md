
# A CI/CD journey with Azure DevOps and Terraform — Part 3

Introduction

We did discuss about the CI/CD concepts in [part 1](/@jamesdld23/a-cicd-journey-with-azure-devops-and-terraform-part-1-358f785b13f3), we built a complete CI/CD pipeline in [part 2](https://medium.com/faun/a-ci-cd-journey-with-azure-devops-and-terraform-part-2-524144511294) using [Terraform](https://www.terraform.io/) and [Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/pipelines/get-started/overview?view=azure-devops).

That did represent a lot of code that could definitively be used through a template. Let’s simplify that doing a Terraform [YAML template](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops)!

I need a Terraform YAML template able to perform the following action upon what I ask it to do :

* Select the Terraform version to use

* Terraform init

* Terraform plan

* Zip the output plan if the Terraform plan tracked any infrastructure changes

* Terraform apply

* Terraform destroy

## Workflow of the ADO Terraform YAML template

The following diagram illustrates the Terraform YAML template’s workflow, you could have a look on this template on my [Azure DevOps repository here](https://dev.azure.com/jamesdld23/Template/_git/template_pipeline?path=%2FREADME.md&version=GBmaster) :

![Terraform YAML template pipeline — Workflow](https://cdn-images-1.medium.com/max/2000/1*6ox6eNhxETb1uOkKDm9aYA.png)*Terraform YAML template pipeline — Workflow*

## Use the template

The following code samples can be merged in one yaml file that will be your a CI/CD pipeline.

1. Call the [Repository resource](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema#repository-resource) to get the Terraform template YAML

<iframe src="https://medium.com/media/7421a58b992eb3b7a2e17921fbfbaee6" frameborder=0></iframe>

2. Launch a Terraform init, plan and publish and artefact whithin a build pipeline job

<iframe src="https://medium.com/media/ebc145b2893f21d388a32e1eb4c8d4a1" frameborder=0></iframe>

3. Launch a Terraform apply in a deployment pipeline job

<iframe src="https://medium.com/media/89c92e3fa5d1bd8fc4c59d97bc948529" frameborder=0></iframe>

4. Launch a Terraform destroy in a deployment pipeline job

<iframe src="https://medium.com/media/c4667606266844d1aaf7261c58ef675c" frameborder=0></iframe>

And there we are ! Back to our 3 stages →

1. Build (Terraform init & plan)

1. Deploy (Terraform apply)

1. Deliver (Terraform destroy)

![](https://cdn-images-1.medium.com/max/2290/1*JUE5_I6A9G-5RRHnKDgH9A.png)

## Stories inventory

1. [Part 1](https://medium.com/faun/a-cicd-journey-with-azure-devops-and-terraform-part-1-358f785b13f3) : The CI/CD concept and tooling choices.

1. [Part 2](https://medium.com/@jamesdld23/a-ci-cd-journey-with-azure-devops-and-terraform-part-2-524144511294) : Build your CI/CD pipeline for an Kubernetes cluster with Azure Kubernetes Service through Azure DevOps and Terraform 0.12.

1. [Part 3](https://medium.com/faun/a-ci-cd-journey-with-azure-devops-and-terraform-part-3-8122624efa97) (current) : Azure DevOps YAML template to manage our Terraform action.

See you in the Cloud

Jamesdld

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
