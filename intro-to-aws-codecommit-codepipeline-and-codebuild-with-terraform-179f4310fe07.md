Unknown markup type 10 { type: [33m10[39m, start: [33m133[39m, end: [33m138[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m14[39m, end: [33m28[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m32[39m, end: [33m47[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m185[39m, end: [33m194[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m9[39m, end: [33m18[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m14[39m, end: [33m28[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m32[39m, end: [33m37[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m142[39m, end: [33m151[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m228[39m, end: [33m242[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m97[39m, end: [33m109[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m6[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m10[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m182[39m, end: [33m185[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m145[39m, end: [33m147[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m168[39m, end: [33m180[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m195[39m, end: [33m197[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m18[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m353[39m, end: [33m368[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m420[39m, end: [33m423[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m30[39m, end: [33m50[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m13[39m, end: [33m40[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m63[39m, end: [33m84[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m86[39m, end: [33m107[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m58[39m, end: [33m68[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m103[39m, end: [33m106[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m58[39m, end: [33m79[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m91[39m, end: [33m97[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m286[39m, end: [33m289[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m33[39m, end: [33m49[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m66[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m97[39m, end: [33m100[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m98[39m, end: [33m119[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m210[39m, end: [33m232[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m144[39m, end: [33m151[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m109[39m, end: [33m137[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m347[39m, end: [33m358[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m64[39m, end: [33m79[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m120[39m, end: [33m124[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m58[39m, end: [33m72[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m114[39m, end: [33m125[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m162[39m, end: [33m176[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m224[39m, end: [33m241[39m }

# Intro to AWS CodeCommit, CodePipeline, and CodeBuild with Terraform

Image courtesy of AWS
> This blog series focuses on presenting complex DevOps projects as simple and approachable via plain language and lots of pictures. You can do it!

AWS CodeCommit is one more CI/CD to enter the increasingly crowded competition for CI/CD products. AWS has provided an entire suite of products:

* [**CodeCommit](https://aws.amazon.com/codecommit/)**: A managed git repo. We’ll check our terraform code into a repo hosted in Codecommit. Enough said.

* [**CodeBuild](https://aws.amazon.com/codebuild/)**: A managed continuous integration service. It runs job definitions, dynamically spins up and down build servers, and can support your own tooling, i.e. terraform! We’ll write a deploy terraform build in CodeBuild.

* [**CodeDeploy](https://aws.amazon.com/codedeploy/)**: A managed deployment service that helps push code from a repo to AWS services where it can be executed. This is the only CodeX service from AWS we won’t use.

* [**CodePipeline](https://aws.amazon.com/codepipeline/)**: A managed deployment service that supports complex deployment processes including code testing, automated deployment all the way to production. We’ll write a pipeline to automate a PR merge → terraform deploy.

As you’ll soon see from this walkthrough, or by reading over the modules source code ([you can find it here](https://github.com/KyMidd/AwsCodePipelineDemo)), the amount of code required for this project is WAY more than any other project, specifically 1646 lines of terraform and yml config.

<iframe src="https://medium.com/media/22ae2a382f29aa6a2c951cf3d87363b0" frameborder=0></iframe>

## Create an IAM user for Bootstrapping

We’ll need an IAM user with administrative access for our initial cloud bootstrapping. However, unlike other CI/CDs I’ve played with, the AWS CodeBuild service can consume an IAM role when it’s fully operational, negating the need for hard-coded administrative permissions (which if that doesn’t scare you a little, it should!). Let’s do this.

Create an IAM user and check the Programmatic access box.

![](https://cdn-images-1.medium.com/max/3496/1*UIMOaMpOVAPG0TRw_RRR9Q.png)

Attach the IAM user to an existing policy. We’ll give this user unfettered AdministratorAccess. That’s not a great idea for prod — it’s worth thinking about what you want this user to do. In our case, this IAM user will only be used to deploy our bootstrap. When the CodeBuild is fully deployed, it’ll assume an IAM role directly, rather than needing a hard-coded user (read: way better security).

![](https://cdn-images-1.medium.com/max/3564/1*MogZ194r2irn2B5UkfYMvA.png)

Tags aren’t important for us now, so you can leave it blank.

![](https://cdn-images-1.medium.com/max/3524/1*J8WNn-J9RZyMa6UvMppkHQ.png)

Now we have an IAM user. Make sure to copy down the Secret access key — it won’t be shown again.

![](https://cdn-images-1.medium.com/max/3636/1*YIT7wK7OCoGHVXTDFdumvg.png)

In your local terminal, export those values to your local shell. We’ll only be using them for the bootstrap — to deploy our CodeBuild the first time. In the future, we’ll be able to use CodeBuild to update these resources with terraform.

![](https://cdn-images-1.medium.com/max/2952/1*pLmiI69OB1Ym3k32b16LQg.png)

Before we run terraform init or terraform apply to build some resources, let’s walk through each module. I’ll talk about each step in the sequence, but there are enough inter-dependencies we’ll need to create them all at once.

## The Bootstrap Module

We’re going to run our initial config from our local computer, but that’s not our end goal — we want our cloud CI/CD to pull its fair share of the weight here. The first module, called bootstrap, creates everything terraform needs to run against an ephemeral CI/CD environment. It also will build some IAM and other resources required for our AWS CodeBuild and CodePipeline services.

Find the bootstrap module in the main.tf and look for the strings in quotes. Those are names that we can customize. Notably, the s3 bucket name can’t use capital letters or many special characters and has to be globally unique (read: update it!). The other items only need to be locally unique, but feel free to customize them.

It’ll look like this:

![](https://cdn-images-1.medium.com/max/3028/1*Jyng18-2IqngI-Qe7SRNMQ.png)

Don’t run the terraform init or apply yet, we’ll customize all modules and run a big super build later.

## The CodeCommit Module

CodeCommit is by far the simplest service, and naturally, the simplest terraform module. This module builds a hosted git repo using the CodeCommit service. This is where we’ll check in our code.

If you’d like, update the repository_name to any string you’d like.

![](https://cdn-images-1.medium.com/max/2000/1*RZpLkUK77-gyQP1kY8AhKQ.png)

Look at this simple little module:

<iframe src="https://medium.com/media/f152b3f26c4cc5cfac4e2a99f25040f7" frameborder=0></iframe>

## The CodeBuild module

CodeBuild is the service that spins up virtual machine builders and instructs them to execute instructions. These instruction sets are called buildspec files and are written in YAML. Here’s the build spec we’ll be using for the Terraform Plan CodeBuild job.

The $TERRAFORM_VERSION variable you see if an environmental variable we’ll pass to the build from our CodeBuild environment configuration.

<iframe src="https://medium.com/media/7a02b91b6188e2b893629419f86ee200" frameborder=0></iframe>

Feel free to customize the name of the Terraform Plan and Terraform Apply jobs. I went with simple here.

![](https://cdn-images-1.medium.com/max/2620/1*GSQUsct3gCOyIhf2DJO2pg.png)

There is a lot going on in the CodeBuild module. Here’s the source for one of the jobs. Note the service_role which is the IAM role this CodeBuild job runs under. Figuring out these permissions took a long time and lots of iterating. Neither HashiCorp (Terraform’s builders) or AWS makes this particularly easy to figure out.

You can also see the environment information, like the type of hosts to spin up for the job, where logs are sent, as well as the artifact and source for running against, all of which is gleaned from a CodePipeline calling on this job and passing it information.

<iframe src="https://medium.com/media/2dbd2656335b2fb9970d2a806b9037db" frameborder=0></iframe>

## The CodePipeline Module

CodePipeline is a stitching-together DevOps tool. It allows you to generate workflows that grab sources from multiple places (including non-native AWS locations, like BitBucket or GitHub), send that info to builds, do manual approvals, and all sorts of other cool stuff I’m still discovering.

What we’re doing in this module is to build:

* An S3 bucket for the temporary artifact storage (read: code) pulled out of the git repo

* An IAM role that permits CodePipeline to assume it

* An IAM policy that permits CodePipeline to run — it has a lot of permissions

* The actual CodePipeline with every step, including downloading the source code from the CodeCommit repo (as well as watching the repo and triggering on changes), running a Terraform Plan CodeBuild stage based on the code, hanging for manual approval of changes, then once approved continuing onto a Terraform Apply CodeBuild stage

There’s clearly a ton here — here’s the config for the CodePipeline. There’s not a lot of steps of configuration, but the AWS documentation for these stages isn’t great, and I had to iterate a dozen or so times to figure out compatible categories, providers, and configurations for each step. I ended up basically reading the API docs and guessing how Terraform would abstract them. If you’re building this yourself, hopefully, this code snipped can save you some pain.

<iframe src="https://medium.com/media/fb3327c53fa59d060cbbdc250913fb44" frameborder=0></iframe>

Here’s the configuration snipped from the main.tf. Feel free to customize any of the quotes strings. You have to customize the S3 bucket used to store artifacts — it has to be globally unique (and lower-case, and no special characters).

![](https://cdn-images-1.medium.com/max/2852/1*uklE0yg9YD1HsJIJiSaZmw.png)

## It’s Showtime!

Run a terraform init to initialize the repo. Note that for now, we have the terraform S3 backend commented out — since the back-end doesn’t exist yet, we want to run all our changes locally. Also, the AWS provider has an “assume role” (used to assume an all-powerful administrative role when running from AWS) that we can’t use yet — that role also doesn’t exist yet, so it’s commented out.

<iframe src="https://medium.com/media/d91f02a04832c25093306498b34251a9" frameborder=0></iframe>

Let’s run terraform apply to verify our terraform config is all valid and your creds have been accepted. If you see a plan to build lots of items, you’re probably good to go. Answer yes and hit enter and terraform will start building.

![](https://cdn-images-1.medium.com/max/2000/1*LukwiOSYVxTb0rfcvdAV5w.png)

Terraform will build all the items it can. If it can’t, it’ll give you some error messages. Since this config works, the errors are probably around names that are invalid, particularly the S3 bucket names. If you left the names to what I provided, my bucket already exists, so the AWS API will report an error — just update the bucket name. It’ll complain also if the name doesn’t fit the required format — the errors are very clear, so read carefully and you’ll be able to fix the problem.

Hopefully, Terraform will report a happy state as seen below that both resources were successfully built.

![](https://cdn-images-1.medium.com/max/2000/1*AuxursDZ-Hl9fqLLGE2eww.png)

Now that our S3 bucket and TF locking DynamoDB are ready, let’s pivot our state to them. In our terraform provider config, uncomment (remove the /*) at the head of our backend "s3" block and the */ at the tail end. When you’re done it’ll look like the below snapshot.

Make sure the bucket name and DynamoDB table name you used are set here — they have to match the resources you just built for this to work properly!

![](https://cdn-images-1.medium.com/max/2000/1*hjL7_KZJtbkTcdbw5omtKw.png)

Run terraform init once more and terraform will note the new backend, and ask if you want to move your local state to there. Heck yes, we do — a remote state enables teams to work on the same infrastructure much more reliably than emailing around a state file, not to mention concurrently — the locking DynamoDB will prevent team members from executing terraform apply at the same time and tripping one another up. Type yes and hit enter.

![](https://cdn-images-1.medium.com/max/2336/1*axOxX6vr0rPYWuPeK0Y5wA.png)

Hopefully, the initialization goes well, and you’ll see a green success message.

![](https://cdn-images-1.medium.com/max/2324/1*yndr0Mjh6utmrQVHlha9NQ.png)

If you see the above success message, you have successfully pivoted your Terraform state and locking to the cloud. Awesome! Now let’s get CodePipeline going

## CodeCommit — It’s Simple… Right?

I didn’t expect a hosted git repo to consume any significant part of my time writing this. Git is open-source, and one of the more mature technologies of the modern web, so a hosted server by one of the biggest cloud players should be simple and intuitive.

However, that’s not the case. Pushing code from your local comp to a git repo requires an SSH key, and SSH keys can’t be added by the root user in an account. For example, the user you’re logged into your AWS account as right now.

Rather, you need to create an IAM user and put the public part of your SSH key under that user. Or cheat, as I did in this case, and put my public SSH key under the CodeBuild IAM root user we already created. In an enterprise environment, this is a bad idea — these creds have root access to this account. But in our lab, heck yeah.

First, open up the IAM panel, Users, and find the CodeBuild user we built by hand.

![](https://cdn-images-1.medium.com/max/2000/1*_E8wk3UpcIN3sHLT_CGjnw.png)

Open up the user and find the Security credentials tab.

![](https://cdn-images-1.medium.com/max/2172/1*DgFgmtxdV7KHkxWRusaUvg.png)

Look for the SSH keys for AWS CodeCommit section and click the Upload SSH public key button.

![](https://cdn-images-1.medium.com/max/2260/1*X49IMvzlXz_m0r_OzSEqwg.png)

AWS will show you a text field for you to enter the text of your public key.

![](https://cdn-images-1.medium.com/max/2168/1*KaUGvQDNAB7J6Vq0lsHQfA.png)

It’s likely you already have an RSA public key. You can test by running this command: cat ~/.ssh/id_rsa.pub. If you get a response, that’s the public key you should use.

If not, you need to generate an RSA key using the command ssh-keygen (on a mac or Linux). Then run the cat command again to get your public key.

![](https://cdn-images-1.medium.com/max/2000/1*t-uWtg32Iir2GZwwOfzP_w.png)

Copy the whole string into the SSH public key box and hit Upload SSH public key.

![](https://cdn-images-1.medium.com/max/2168/1*ImPLOYGiLcsp4JjfU9KJ-w.png)

If all went well, AWS will accept the public key and show the key ID here with a status of Active.

![](https://cdn-images-1.medium.com/max/3832/1*cPF9N_idIWcZ6EpfFN5Dqw.png)

And if this were any other git server, we’d be done. However, Amazon requires we also send a username with our SSH key. The username isn’t the name… of our user, like would make any semblance of sense, it’s the SSH key ID displayed above. You can either specify it every time you run a git command (ugh), or you can specify it in an .ssh

If you like vi, run this command:vi ~/.ssh/config to create (if not there) and edit the file to have the following host entry. You can also use any other text editor to create/update the file.

Host git-codecommit.*.amazonaws.com
 User (Your SSH public key ID)

![](https://cdn-images-1.medium.com/max/2000/1*VfsjH14Scgcm9MliCZaGQA.png)

Then save the file, and your git should now work. FINALLY!

## Push our code to CodeCommit

In order to see our CodePipeline in action, we need to push our code up. With the SSH key in place, let’s do that.

Head over to the CodeCommit web portal, and find your repo. Over on the right side, click on the SSH Clone URL button to copy it to your clipboard. Then get a CLI session going wherever you have your main.tf file stored.

![](https://cdn-images-1.medium.com/max/5084/1*hrGEZn7-lHAl76C2glApIg.png)

Initialize your git repo, then add all your files to the staging area and then into a commit. Run git remote add origin and then the SSH string you copied above to tell git where to attempt to push files. Then git push origin master to push everything up to our CodeCommit repo. If all goes well it’ll look like the below:

![](https://cdn-images-1.medium.com/max/3528/1*52Lq0z8_tAi19kWJGFUthw.png)

## Okay, Showtime Really This Time

Remember, our CodePipeline is watching our repo and will detect this change within a minute or two. Head over to the CodePipeline console and you can see that our pipeline is in progress.

![](https://cdn-images-1.medium.com/max/2688/1*ifK7LOLuCQaZsV4OfQ4nyw.png)

Click into the pipeline and you can watch the steps execute. If you see an error or want to see the CLI output from our YML buildspec, click on Details.

![](https://cdn-images-1.medium.com/max/2000/1*pSvOwiSoKofJjzmHtYmKqA.png)

Scroll all the way down to the bottom to see our status. If it looks like the below, and you see Terraform’s Infrastructure is up-to-date message then everything has gone well and you should buy a lottery ticket! If you see errors, read them closely. The first time I pushed this for take-pictures purposes, I forgot to uncomment the AWS provider assume_role block and Terraform reported the error pretty clearly.

![](https://cdn-images-1.medium.com/max/2308/1*SDObFCFA-gOY_KWVGGSMPQ.png)

Click back into the pipeline and if all went well, we’re on the Manual_Approval step. The logic here is terraform ran a plan and needs confirmation from a human before it continues on to building resources. Hit the “Review” button to take an action.

![](https://cdn-images-1.medium.com/max/2000/1*d7jKyWB8OsMMomggRqLDog.png)

The Review section allows us to leave a comment. Hit “Approve” if you’d like to see the Terraform_Apply step run. Otherwise, hit Reject to end this pipeline run.

![](https://cdn-images-1.medium.com/max/2176/1*OwrzczaEpAkhJhsPzJvkfg.png)

I hit Approve, and can see that the Terraform_Apply step succeeded. Hit details to read any CLI output from the run.

![](https://cdn-images-1.medium.com/max/2000/1*mI64QLFa6ctHvLNQQnKnpg.png)

Now that all is live, feel free to push terraform code changes to this repo. Add a VPC module, build some hosts, anything you’d like. Terraform is now fully capable of building any resource you’d like.

To destroy your lab to avoid any charges, comment out the remote_backend block in the terraform provider, and the assume_role block in the AWS provider, then run terraform init (say yes to copying state back to local), then terraform destroy.

## Summary

The AWS Development and CI/CD toolkit seems highly customizable, and each component can (must be) locked down via IAM policies. This both creates a highly secure environment and a lot of headaches for those responsible for building and maintaining it over time (boo!).

The amount of configuration and time required to build these features, tie them together in a meaningfully secure way, and even to push code to a repo greatly eclipses every other project I’ve worked with. The only one that comes close is Azure DevOps. There can be a great strength in the complexity available to you — you can build bespoke pipelines that do EXACTLY what you’re looking for.

But the complexity here is only somewhat that. A great deal of complexity is simply the byproduct of platform immaturity. Here’s hoping some time and magic AWS dust will further develop this platform into a juggernaut in the space, as they have with so many other cloud technologies.

The source code for everything is hosted here:
[**KyMidd/AwsCodePipelineDemo**
*You can't perform that action at this time. You signed in with another tab or window. You signed out in another tab or…*github.com](https://github.com/KyMidd/AwsCodePipelineDemo)

If you’re interested in reading about other CI/CDs I’ve built up, please visit my author page.

Thanks all. Good luck out there!
kyler.
