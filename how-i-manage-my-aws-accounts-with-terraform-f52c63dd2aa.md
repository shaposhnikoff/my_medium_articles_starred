Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m61[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m126[39m, end: [33m139[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m13[39m, end: [33m28[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m37[39m, end: [33m45[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m127[39m, end: [33m140[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m140[39m, end: [33m152[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m300[39m, end: [33m316[39m }

# How I manage my AWS accounts with Terraform

Using AWS accounts to isolate workloads has long been established as a good practice but it can often be unclear how to manage that process.

![Like AWS accounts these boxes are used to isolate their contents from the other boxes. Photo by [William Felker](https://unsplash.com/@gndclouds?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/boxes-on-conveyor?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/4896/1*nnDiRk92G4oxUdBhVjRxag.jpeg)*Like AWS accounts these boxes are used to isolate their contents from the other boxes. Photo by [William Felker](https://unsplash.com/@gndclouds?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/boxes-on-conveyor?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

AWS recently released [Control Tower](https://aws.amazon.com/controltower/) to make it easier for teams to manage their AWS accounts. But Control Tower has some limitations, for example it doesn’t yet support existing accounts. And for small teams, or individuals, a simpler approach may be all that’s needed. Hopefully if you come into either of those categories then this approach will help you.

As a full time nerd I often have ideas for projects I want to work on in my spare time. Most of these never get finished, my attention span is appalling, so I have quite a few unfinished projects on the go at any one time. For these projects I like to work on them in their own their own AWS account. It feels ‘tidy’ this way to have each project isolated from the others.

With that in mind, here is how I manage my AWS accounts. A reference implementation for this approach is available on GitHub.
[**robbytaylor/terraform-aws-accounts-reference**
*This repo contains a simple Terraform reference configuration for managing multiple AWS accounts as part of an AWS…*github.com](https://github.com/robbytaylor/terraform-aws-accounts-reference)

This is a reference implementation based on my own Terraform configuration.

The account structure here is much simpler than other implementations, such as AWS Control Tower. That’s because I’m not a big organisation and my needs are much simpler.

![AWS account structure](https://cdn-images-1.medium.com/max/2000/1*HtcPdB43kKHy2yeolp1pSw.png)*AWS account structure*

It would be overkill for me to have a single sign on solution and associated account, for example. It’s only me that has access to my AWS accounts. However, the reference implementation could easily be adapted to accommodate more complex account structures if needed.

## Getting Started

I initially used the Terraform configuration in the bootstrap directory to create the resources for storing Terraform state.

I use [aws-vault](https://github.com/99designs/aws-vault) to manage my AWS credentials and use that to run Terraform in the different project accounts. An example of a ~/.aws/config file is:

    [profile root]
    region=eu-west-1
    mfa_serial=arn:aws:iam::123456789012:mfa/rob

    [profile project1]
    source_profile=rob
    role_arn = arn:aws:iam::111222333444:role/AdminUser

## Starting a new project

When I’m ready to start a new project I add a new account name to the accounts variable in the account creation Terraform:

[https://github.com/robbytaylor/terraform-aws-accounts-reference/blob/master/accounts/-creation/terraform.tfvars#L1](https://github.com/robbytaylor/terraform-aws-accounts-reference/blob/master/accounts/-creation/terraform.tfvars#L1)

And then run terraform apply within that directory against the root account.

    > aws-vault exec root -- terraform apply

This creates a new AWS account and sets up the required permissions for me to access the account and for Terraform state from that account to be stored in the state bucket.

I then create a new directory in the accounts directory where I define the resources for that account, add a new profile to my ~/.aws/config file, and apply the Terraform to the new account:

    > aws-vault exec project1 -- terraform init

    > aws-vault exec project1 -- terraform apply

## Improving the process

### Removing repetition

Looking at the code you’ll probably notice that it isn’t very DRY. In particular the bucket name for the Terraform state is repeated in the terraform.tf file for every account.

This hasn’t been enough of an issue for me so far to be motivated to do anything about it — I only have a few accounts. But this is exactly the sort of problem that [Terragrunt](https://github.com/gruntwork-io/terragrunt) solves. In the future, as I have more half-arsed ideas, and the number of accounts I have increases, I’ll probably wrap my Terrform cofiguration with Terragrunt. It should be quite easy.

### Automation

Account creation is a somewhat manual process at the moment. I have to update the list of account names and then manually create the account directory with some template Terraform and and then apply the changes.

Fortunately, if the project ever grew enough to justify it, this could easily be addressed with some simple automation such as a bash script.

Rather than using a variable to store the list of accounts you could use any of Terraform’s data sources. For example, create an account for every [GitHub repository](https://www.terraform.io/docs/providers/github/d/repositories.html) which matches a search criteria.

Hopefully this reference code will be useful for your own projects. As I mentioned, it’s more suited for individuals or small teams with a few accounts. But what I love about it is that it’s incredibly flexible, you could easily extend it to work with more advanced account structures. And thanks to terraform import it can easily be applied to existing account structures.

If you do find this useful then I’d love to hear from you.

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
