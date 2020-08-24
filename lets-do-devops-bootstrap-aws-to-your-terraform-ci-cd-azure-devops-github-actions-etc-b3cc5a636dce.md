
# Let’s Do DevOps: Bootstrap AWS to Your Terraform CI/CD (Azure DevOps, GitHub Actions, etc.)

This blog series focuses on presenting complex DevOps projects as simple and approachable via plain language and lots of pictures. You can do it!

Pairing Terraform with a CI/CD like Azure DevOps, Terraform Cloud, or GitHub Actions can be incredibly empowering. Your team can work on code simultaneously, check it into a central repo, and once code is approved it can be pushed out by your CI/CD and turned into resources in the cloud.

![Photo from [Skeeze@Pixabay](https://pixabay.com/photos/cowboy-boots-shelves-styles-shoe-3956786/)](https://cdn-images-1.medium.com/max/2000/1*T9QnCkGg0Rz38wyPrT9cyw.jpeg)*Photo from [Skeeze@Pixabay](https://pixabay.com/photos/cowboy-boots-shelves-styles-shoe-3956786/)*

When you start rolling this out, you run into an immediate catch22 — you need an S3 bucket for TF state, and a DynamoDB for state locking to run terraform, but you need to run terraform in order to build these resources.

The best method I’ve thought of to get around this problem I’m calling “pivoting”. The basic order is:

1. Run terraform from your local machine, and build the S3 bucket, DyanmoDB table, and any other bootstrap items you need.

1. Tell terraform to use the s3 bucket and DyanmoDB table, and push your local .tfstate to the remote storage.

1. Upload your terraform to the CI/CD, where it can access its state file and start building other cool things.

Let’s walk through the steps, and you’ll have an AWS account bootstrapped into your CI/CD before you can say “terraform can do that?”

## AWS IAM User — Authorization

Before we can do anything with terraform, we need to authenticate. The simplest way to do that is to build an IAM user in AWS.

First, jump into AWS and type IAM into the main page console, then click on the IAM dropdown.

![](https://cdn-images-1.medium.com/max/2000/0*r5eF6jrMRY6tutvm.png)

Click on “Users” in the left column, then click on “Add user” in the top left.

![](https://cdn-images-1.medium.com/max/2000/0*hN6nzO6482s-uLbH.png)

Name your user. It can be anything, but it’s helpful to have it named something you recognize as a service account used for this Terraform Cloud Service. Terraform can only do what the IAM user can do, and we’re going to make it a global administrator within this account, so I like to include “Admin” in the same so I remember not to share the access creds.

Also, check the “Programmatic access” checkbox to build an Access Key and Secret Access Key that we’ll use to link TF to AWS, then hit “Next: Permissions” in the far bottom right.

![](https://cdn-images-1.medium.com/max/3020/0*p0Ml4sN8J2t3wXqi.png)

This user now exists, but can’t do anything. For a more in-depth look at IAM (and it does go deep), refer to the same [earlier blog](https://medium.com/swlh/aws-iam-assuming-an-iam-role-from-an-ec2-instance-882081386c49) about assuming an IAM role. To grant it permissions, we can either create a custom policy with specific and limited permissions, or we can link to existing policies. For the sake of this demo, we’ll use “AdministratorAccess”, but remember this step as another opportunity for extra security in a real enterprise environment.

![](https://cdn-images-1.medium.com/max/2920/0*10T52mXiPYRCslrQ.png)

Hit Next Tags in the bottom right, then Next Review. If the Review page looks like the below, hit “Create user” in the bottom right, and we’re in business.

![](https://cdn-images-1.medium.com/max/2828/0*fZ9w4CxTJZA_709t.png)

You’re presented with an Access Key ID and a secret access key (behind the “show” link). You’ll need both of these, so don’t close this page.

![](https://cdn-images-1.medium.com/max/3532/0*hg2ef_zSQSTsWV8j.png)

Export that info to your terminal using this type of syntax:

<iframe src="https://medium.com/media/9f8220310a4065c2f70ba3c0486f57a0" frameborder=0></iframe>

Now your local terminal can run terraform will full administrative permissions in your account, which is great for our bootstrapping efforts.

## Terraform Bootstrap Code

Now that our terminal is ready to run Terraform code, we need some code to run. Remember, what we’re hoping to do is build an S3 bucket (for TF state storage) and a DynamoDB table (for TF state locking).

To make things super easy, I wrote a bootstrap Terraform script and stashed it here: [https://github.com/KyMidd/Terraform_CI-CD_Bootstrap](https://github.com/KyMidd/Terraform_CI-CD_Bootstrap)

First, we’ll establish our providers. Note that the backend is currently commented out — that’s by design — we need to build the backend before enabling it! But we’ll turn it on soon enough.

<iframe src="https://medium.com/media/33438d09243366a545b272cc1cdc2523" frameborder=0></iframe>

Then we call the bootstrap module. This isn’t all in the main.tf by design — you can build on your own modules in the future if you’d like. The items on the right are strings, feel free to change them. The one you definitely need to change is the “your_globally_unique_bucket_name”. That one has to be… you guessed it, globally unique.

<iframe src="https://medium.com/media/18ee83a259eb93dff3106b2e5ef334c0" frameborder=0></iframe>

We build an S3 bucket first:

<iframe src="https://medium.com/media/a0b656cca62dc8251660a5f2b4c7cfe2" frameborder=0></iframe>

Then we build the DyanmoDB table:

<iframe src="https://medium.com/media/1868d6354de27dea4cb72c50dbab7a34" frameborder=0></iframe>

Copy that code (cloning [the repo](https://github.com/KyMidd/Terraform_CI-CD_Bootstrap) is the easiest method), then get there in your terminal. Run “Terraform init” to initialize the code and make sure terraform is ready to go.

![](https://cdn-images-1.medium.com/max/2260/1*w-o6qDX6mimub2xIMKDs5A.png)

Then run a “terraform apply”. You should see that terraform will create 2 resources, and do nothing else. If it all looks good, type “yes” at the prompt and Terraform will build our resources.

![](https://cdn-images-1.medium.com/max/3368/1*k5DQl8hegfkYlolD1uBCrg.png)

Now that our bootstrap is ready, let’s push our local .tfstate file to it. The first step is to uncomment the remote state backend in our main.tf file. It’ll look like this, but with your bucket name:

<iframe src="https://medium.com/media/41cd09ca345e5aeaf89407bb91ec5961" frameborder=0></iframe>

Then do another “terraform init”. This time it’ll look a little different, as terraform realizes a remote backend has been added. Enter “yes” to copy your local .tfstate file to the remote backend.

![](https://cdn-images-1.medium.com/max/2424/1*Il5J9phKMgmZMIavfZnkLg.png)

## Now for the REALLY cool stuff

Great, now we have all the items done we need to build our CI/CD and integrate it with AWS.

For specifics on how to build them out, check out some of these blogs:

* [Azure DevOps](https://medium.com/swlh/connect-azure-devops-to-aws-b89120599103)

* [Terraform Cloud](https://medium.com/swlh/an-intro-to-terraform-cloud-github-and-aws-for-ci-cd-7cee09790005)

* [GitHub Actions](https://medium.com/@kymidd/lets-do-devops-github-actions-terraform-aws-77ef6078e4f2)

Good luck out there! Kyler
