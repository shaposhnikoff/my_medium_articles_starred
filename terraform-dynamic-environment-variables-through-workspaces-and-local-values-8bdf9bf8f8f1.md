Unknown markup type 10 { type: [33m10[39m, start: [33m116[39m, end: [33m140[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m196[39m, end: [33m205[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m73[39m, end: [33m105[39m }

# Terraform Dynamic Environment Variables through Workspaces and Local Values

We’ve fallen in love with Terraform and have moved all infrastructure over to using it on AWS. This is has been a huge win for our team and allows us to confidently deploy infrastructure changes across multiple accounts.

Our team uses 3 separate AWS accounts for each product we release — develop, staging and production. In doing this we often need different namespaces for services — like S3 buckets, API Gateway custom domains, CloudWatch log retention policies, etc.

To handle this we use Terraform’s awesome feature called [workspaces](https://www.terraform.io/docs/state/workspaces.html) which are:
> Named workspaces allow conveniently switching between multiple instances of a *single* configuration within its *single*backend.

If you’re currently using Terraform, you’re already using the “default” workspace without realizing it. Try running terraform workspace list on an initialized Terraform project and it will return * default(the star indicates the workspace you currently are in).

![terraform workspace list](https://cdn-images-1.medium.com/max/2000/1*T4KizctP5UjPWPCNUi9sqQ.png)*terraform workspace list*

Now let’s go ahead and create a new workspace called “develop” by typing terrarform workspace new develop. This will create a new workspace and automatically switch to that workspace.

![](https://cdn-images-1.medium.com/max/2128/1*sMfrTpKPX1PuA0ftQdN7fA.png)

Now let’s use this newly created workspace to start setting variables. Below is a simple example that has a local mapping of S3 buckets and domain names. Then below that on lines 15 & 16, we use those mappings and Terraform’s **terraform.workspace **object to map those to a single variable that we will use in later.

<iframe src="https://medium.com/media/bd152165c09f551de66e664f4151eedd" frameborder=0></iframe>

We can now use those locals we set above in the rest of our templates for each environment.

<iframe src="https://medium.com/media/531fd66f5eb4b86c116e957c0767eb6a" frameborder=0></iframe>

So now when you’re in the “develop” workspace, the S3 bucket, for example, will be “my-develop-bucket” and in staging “my-staging-bucket”.

You can use this for any resource or namings that are specific to an environment. It’s been a big win for us in having a single Terraform script that runs in all environments to ensure each environment is consistent.
