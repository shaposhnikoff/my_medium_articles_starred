
# Building an AWS CloudWatch dashboard using Terraform — 15 minute guide

Terraform is a leading Infrastructure as Code (IaC) tool. This is a short guide on what it is, why you should use it & how to start — in 15 minutes.

IaC means that you define infrastructure in code (usually in a text file) & then predictably generate that infrastructure in the Cloud specified. When you are done with it you can predictably remove the infrastructure created. Terraform is cloud agnostic, so can be applied to any major cloud.

![](https://cdn-images-1.medium.com/max/3110/1*VM0k87ohWX6sGufaXe-ACA.png)

## Why Terraform?

Why use Terraform instead of Puppet, Chef, Ansible etc? [Well Steve Strutt, IBM CTO, makes a very good case for Terraform](https://www.ibm.com/cloud/blog/chef-ansible-puppet-terraform), in that Terraform is simply not the same as other leading IaC tools — being an orchestration tool rather than a configuration tool — with his recommendation being to use Terraform to build infrastructure & Puppet, Chef etc to configure, patch and so on. [Gruntwork have made similar observations](https://blog.gruntwork.io/why-we-use-terraform-and-not-chef-puppet-ansible-saltstack-or-cloudformation-7989dad2865c).

Terraform is immutable so it fits with the “cattle” not “pets” concept that is a solid (apols!) principle of Cloud. Fundamentally losing a piece of infrastructure should be a trivial event that can happen at any time so being able to add or remove infrastructure should be a trivial act — not one whereby the configuration of the infrastructure keeps changing (configuration drift) to the extent that you daren’t change stuff.

If you look about you can probably find a more succinct definition that this!

## Install Terraform

Download Terraform [here](https://www.terraform.io/downloads.html) and follow the guide [here](https://www.terraform.io/intro/getting-started/install.html) on how to install Terraform on your specific system.

You should be able to see the help options or Terraform version when installed correctly:

![](https://cdn-images-1.medium.com/max/2902/1*EuXwusPxNqOeJ2Ehieqyeg.png)

## Building infrastructure on AWS using Terraform

Firstly you’ll need access to an AWS account.

Terraform configuration is deployed using the [(HCL) HashiCorp Configuration Language](https://github.com/hashicorp/hcl).

There are plenty of beginner tutorials for building an EC2 instance in AWS & then destroying it (like [here](https://hackernoon.com/introduction-to-aws-with-terraform-7a8daf261dc0), [here](https://medium.com/slalom-technology/creating-your-first-terraform-infrastructure-on-aws-ad986f952951) & [here](https://learn.hashicorp.com/terraform/getting-started/build)) & more [examples in the Terraform Registry](https://registry.terraform.io/). So instead of rehashing those examples I thought I’d offer a CloudWatch dashboard example, after all CloudWatch is used explicitly by AWS as part of auto-scaling (which looks very much like pre-baked IaC to me).

### IAM User

Firstly you’ll need to create an AWS IAM user with programmatic access that is able to create the infrastructure you need. Most beginner tutorials suggest that you give full admin rights, but you can be much more granular than that — save the access key & secret key you get at the end for use later:

![](https://cdn-images-1.medium.com/max/3966/1*tnZ8OHWcNk2i-mEYj_WVQw.png)

### TF File

Create a Terraform file in HCL format with an extention of .tf. If you want a simple start use the first section below to gain access to AWS (provider); then use an [official resource example](https://www.terraform.io/docs/providers/aws/r/cloudwatch_dashboard.html) (resource). This will give you a basis to then experiment.

### Example TF

Below is an example for creating a simple CloudWatch Dashboard & Alarm both in AWS. Displayed in VS Code using the official HashiCorp Configuration Language support for Visual Studio Code plug-in.

* First section gives the provider programmatic access to the Cloud (use your own IAM user keys)

* Second section sets up a variable that is used 5 times in the third section

* Third section adds a Cloudwatch dashboard

* Fourth section add a CloudWatch alarm

![](https://cdn-images-1.medium.com/max/3326/1*GMWVvfMjqlYpgLKmnE_8Nw.png)

![](https://cdn-images-1.medium.com/max/3326/1*eD0-bRZaQ9gQoQeuapvhLQ.png)

### Terraform Init

Run terraform init in the same folder as the TF file to initialise your setup:

![](https://cdn-images-1.medium.com/max/2000/1*RGQgyJg4aeBVXK_Dm-XwJQ.png)

### Terraform Plan

Run terraform plan to build the execution plan:

![](https://cdn-images-1.medium.com/max/2000/1*uS8cORH8GhbKqjyq_kXiOQ.png)

### Terraform Apply

Run terraform apply to apply the plan:

![](https://cdn-images-1.medium.com/max/2000/1*z1KV17eLF1MkbnUDUlcSDQ.png)

All being well after the terraform apply has completed the following infrastructure will exist:

If you’ve followed the example code above then you should see a CloudWatch dashboard called dashboard-i-xxxxxxxx with three widgets, you may not see activity on your widgets because I created a variable pointing at a running EC2 instance in my test setup — so you need to do the same by copying an instance id into the default value for the variable (section 2 above):

![](https://cdn-images-1.medium.com/max/4442/1*Tz2csX1WmMC_wmQo3Dvdtw.png)

A CloudWatch alarm that will fire when CPU Utilization hits 80%:

![](https://cdn-images-1.medium.com/max/6678/1*KdN3wosQ3RfQHcwLUNPZ0Q.png)

### Terraform Destroy

Run terraform destroy to reverse the plan & remove the CloudWatch stuff:

![](https://cdn-images-1.medium.com/max/2000/1*Krscql13WSLz4LheKaCrpg.png)

### So that’s Terraform?

So much more to it, but you should now have a working IaC example. In a few short years you’ll be an expert!

## Other Thoughts

### Interpolation

You can make TF files much more generic using [interpolation ](https://www.terraform.io/docs/configuration-0-11/interpolation.html)to output the instance id(s) after creation of an EC2 & then input it into CloudWatch terraform apply command using the -var switch (beyond the scope etc & so on…). This would mean you could create a standard dashboard targeting every EC2 instance you launch.

But I’ll save all that for another time.

Ref: [Terraform a CloudWatch dashboard](https://www.terraform.io/docs/providers/aws/r/cloudwatch_dashboard.html)

### Packer

You can also user a tool like [Packer ](https://www.packer.io/)to automate your generation of machine images & also bring them into a coding pipeline enabling auto-scaling setup or similar to become trivial.

### **Baked in Infrastructure as Code: Auto-scaling**

Infrastructure in the Cloud, being often virtual from the perspective of a user, naturally lends itself to tools that can manipulate the setup of servers. So a first-class implementation of what I believe is Infrastructure as Code that seems to be baked into every Cloud* is auto-scaling.

[Auto-Scaling is where you define server metrics](https://medium.com/@amelvin/starting-aws-architecture-auto-scaling-c5b56d3e367f) & when these metrics are exceeded the code will be run to scale up (increase the capability of existing servers, usually more RAM/CPU) or scale out (increase the number of server instances).

[Auto-scaling in Microsoft Azure](https://docs.microsoft.com/en-us/azure/architecture/best-practices/auto-scaling) | [Auto-scaling in AWS](https://aws.amazon.com/autoscaling/) | [Auto-scaling in Google Cloud](https://cloud.google.com/compute/docs/autoscaler/)

### **IaC Version Control, CI/CD & a Gotcha**

As HCL files are text files it’s a simple process to use GIT to maintain version control of Terraform files & as a bonus one of the pre-built .gitignore variants is for Terraform. Be aware that one of the gotchas with Terraform is the flipside of its immutability & declarative nature.

If you APPLY a TF file & then change the TF file & then APPLY it again the declarative aspect means that Terraform will build infrastructure that matches the second declaration — which is good.

But if you don’t APPLY the latest version of a TF file, what happens when you come to DESTROY it? Terraform may fail (depending on the conflict) or be hit with a mismatch of what is in the plan & what exists in the cloud. So version control needs a process whereby a DESTROY should only happen against IaC that matches the previous APPLY — suggesting IAC should really move into a CI/CD pipeline (see [here](https://medium.com/pageup-tech/continuously-deploying-your-infrastructure-how-why-6216c9f8c6c8), [here](https://stackoverflow.com/questions/42723189/any-way-to-trigger-a-codedeploy-deployment-with-terraform-after-changes-in-confi/48818173#48818173) & [here ](https://jaxenter.com/tutorial-aws-terraform-147881.html)). But again that is beyond the scope of this article.

## Conclusion

Thanks for reading, I think this is enough for a beginner? Terraform is a deep subject with a low barrier to entry — so hopefully this is enough to get you started. Contact me in the usual ways!
