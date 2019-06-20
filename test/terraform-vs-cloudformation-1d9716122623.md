
# Terraform vs CloudFormation

© pexels.com

This question pops up once in a while — why use a third party tool if AWS has a service which does essentially the same thing?

Arguably there are fewer compelling reasons now than there were previously, as CloudFormation has improved tremendously over the last couple of years. However, if I had to choose again today, I think I would still stick with Terraform. Here are some of the reasons for this.

## Language

Terraform uses HCL (HashiCorp Configuration Language), developed to strike a balance between being human readable as well as machine-friendly.

CloudFormation, on the other hand, uses either JSON or YAML.

In general, YAML is significantly easier to read and author than JSON, but it still forces you to have multiple nested scopes and everything goes horribly wrong if you mix up indentation somewhere. In contrast, HCL normally only has one or two scopes (by scope I mean everything that goes inside the curly braces) and enforces some basic [Go-inspired formatting hygiene](https://www.terraform.io/docs/commands/fmt.html) that makes it easier on the eyes.

Terraform has a rich set of [string interpolations and built-in functions](https://www.terraform.io/docs/configuration/interpolation.html), including conditionals and ([to be natively supported soon](https://www.hashicorp.com/blog/hashicorp-terraform-0-12-preview-for-and-for-each)) loops, which allow modelling quite complex logic in DSL without having to resort to a fully fledged programming language (although this can be done transparently with [external data sources](https://www.terraform.io/docs/providers/external/data_source.html)). [Intrinsic functions](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html) of CloudFormation are noticeably limited in comparison.

Another thing in favour of Terraform is that it makes it easy to reuse code using modules and gives you a lot of leeway in structuring your projects the way it makes the most sense to you. CloudFormation supports nested stacks, but in general, you have to keep all your infrastructure code in a gigantic file (or several) in one project repository, whereas with Terraform you can slice and dice however you want. For regular humans, working with several 100-line files is usually much easier than with a single 5,000+ line file. Modules can be stored on GitHub or [public Terraform Module Registry](https://registry.terraform.io/) and be easily versioned and shared across multiple projects.

## State

Terraform keeps its state in a file. This file can be stored on disk, committed to source control (not recommended), or kept in S3 or some configuration management system. Whatever you choose, you must make sure that you don’t lose or corrupt your state file, or don’t accidentally use a state from production environment while deploying to test. It’s more difficult to shoot yourself in the foot after the introduction of workspaces (previously called ‘environments’), but in the early days, we did get a few battle scars dealing with Terraform state.

In contrast, CloudFormation runs on AWS infrastructure and manages the state for you, so you don’t really care how it does that. Unlike Terraform, CloudFormation also tries to rollback the changes if they could not be applied (well, unless it [gets stuck in UPDATE_ROLLBACK_FAILED state](https://aws.amazon.com/blogs/mt/recovering-aws-cloudformation-stacks-using-continueupdaterollback/)…). In addition, CloudFormation backend can receive signals from your resources, which enables an extremely useful option of configuring [rolling updates for an Auto Scaling group](https://aws.amazon.com/premiumsupport/knowledge-center/auto-scaling-group-rolling-updates/). CloudFormation also doesn’t mind if a machine that triggered a deployment suddenly went down (as can happen with fungible CI/CD servers), whereas Terraform would need to recover from a partial update.

So, on the face of it, this looks like a case where CloudFormation wins. If we dig a little bit deeper though we’ll see that Terraform’s state file is a very powerful concept and one of its main selling points. It allows us to import, adopt, or move around resources, which means **we can refactor our infrastructure** at will without having to tear down and recreate resources.

Another thing which is possible in Terraform is **managing configuration drift**. Every time we run ‘terraform plan’ it fetches the latest *actual *state of the infrastructure and compares it to the *desired *state, defined in the configuration files, and the *known *state, which mirrors the actual state when we last ran ‘terraform apply’ and is stored in the state file. If our infrastructure no longer matches the desired state (usually due to some out-of-band changes), Terraform would compute the diff and suggest to bring it back in line to what’s described in our configuration by reverting the manual changes. As long as we maintain idempotent deployments, running periodic Terraform plans is a convenient way to detect and revert the configuration drift, and is, in fact, something that is done on schedule in our [infrastructure provisioning pipelines](https://devblog.xero.com/ci-cd-with-jenkins-pipelines-part-2-managing-infrastructure-with-terraform-and-docker-bd4b81554aa7).

Meanwhile, CloudFormation would stay blissfully unaware of any changes made outside its state. Try this quick experiment.

First, deploy a new CloudFormation stack to create a new VPC and a security group with no ingress rules:

<iframe src="https://medium.com/media/0a434852ab0e39d5cabd4f066061e05b" frameborder=0></iframe>

Then, go to the web console and manually add a new ingress rule. For example, let’s open our SSH port to the world to do some real quick debugging, no big deal, right?

![What could possibly go wrong with a rule like this.](https://cdn-images-1.medium.com/max/2000/1*KgBzXXdOofqXtRGnhJvmRQ.png)*What could possibly go wrong with a rule like this.*

Next, try to update your CloudFormation stack. What you’d get is this error:

![](https://cdn-images-1.medium.com/max/2000/1*UQzEW8q5S-Jg78dKHQjcxg.png)

Running a Change Set would not detect this change either. As far as CloudFormation is concerned, our infrastructure hasn’t changed.

## Configuration

Parameters make it possible to re-use the same template in various environments. For example, you may want to use cheaper EC2 instances in your test environment, and something more powerful in production.

CloudFormation supports [up to 60 parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html) which must be provided at runtime. There are several ways to do this. The simplest way is to simply pass parameters one by one and default them to something sensible. Alternatively, you can supply parameters in bulk from a file (you need to make sure that you are passing the correct file), [import values from the output of another stack](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-importvalue.html), or read them from Parameter Store. You can also decide to use a [Mapping section](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/mappings-section-structure.html) and to look up parameters by some key (for instance, by environment).

Terraform is considerably more flexible than this. It has a concept of **data sources**, which provide a read-only representation of some resource, allowing you to access its properties without affecting its state. Data sources open a wide range of options to integrate related stacks easily and avoid hard-coding the required parameters. For example, you can access properties of externally managed resources (e.g. find the latest AMI), ingest outputs from a CloudFormation stack, reuse attributes exposed by other Terraform stacks (such as an endpoint for a database or a DNS record), and if that’s not enough, run some custom scripts that provide the required values. There is only **a single source of truth**, and this truth is the current state of the infrastructure, not its snapshot from a while ago, as frequently happens with configuration files. All changes to a resource propagate to all other resources which depend on it during the next Terraform run, without the possibility of a duplicate value being forgotten somewhere, and at the same time maintaining a clear separation of concerns. Another benefit is that you can use the same CI/CD pipeline for managing all your Terraform projects without having to worry about passing the correct set of arguments to each and every resource.

As Terraform is cloud-agnostic, you can compose your infrastructure from different cloud providers, along with resources from third-party services, such as PagerDuty or New Relic. In essence, Terraform gives you a convenient way to manage **all** your infrastructure as code, without having to learn separate tools and configuration syntax for each platform and service.

## Quality of life stuff (aka miscellanea)

Terraform validates the configuration files before trying to run the updates. It checks not only that all files use the correct syntax (CloudFormation has similar validation), but also that all parameters are accessible and the configuration as a whole is valid.

In Terraform, you can (and should) run a ‘plan’ step before applying any changes. This step tells you precisely what is going to change and why (with nice colours to boot!).

![This plan shows that Terraform would have to destroy and recreate two ECR repositories, and the reason for this is the change in their names.](https://cdn-images-1.medium.com/max/2136/1*e9Mpd33v3y-Xiap9iyHBHA.png)*This plan shows that Terraform would have to destroy and recreate two ECR repositories, and the reason for this is the change in their names.*

CloudFormation supports Change Sets, which are considerably [less human-friendly and hard to make sense of](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-changesets-view.html).

Terraform can automatically format code, helping to maintain a consistent code style and making pull requests easier to read. JSON (or now YAML) fatigue which plagues CloudFormation developers is hard to imagine with Terraform.

Neither Terraform nor CloudFormation cover 100% of available AWS resources, but generally Terraform is known to support newly available resources *months* earlier than they are made available in CloudFormation. All new AWS functionality must have REST API and SDK support for main languages when it is released. CloudFormation support, on the other hand, is optional. The open source software shines in this case, as Terraform developers quickly jump in to add the functionality that they want to use.

## Conclusion

So, does this all mean that Terraform is always superior to CloudFormation? Not really. There are some use cases where using CloudFormation makes more sense.

One of such cases is managing the S3 bucket used for storing Terraform state files. Creating it manually shouldn’t even be considered to be an option, as in this case one of the most important buckets in your infrastructure would be essentially unmanageable and unauditable. You can, of course, provision it in Terraform, but it seems to immediately turn into a chicken-and-egg proble m— where are you going to store the state file of the bucket used to store the state files? One option is to commit it to a source control system, which may work ok if you are not making frequent changes to this resource, but is generally not the best idea. CloudFormation can be a compelling option to master this bucket, as then you don’t have to worry about state file altogether. Alternatively, you could indeed create it in Terraform using local state at first, and then add the S3 backend pointing to the same bucket that was just created. After you reinitialise your project by running ‘terraform init’, Terraform would migrate the state file to S3.

Another use case is the rolling updates functionality for Auto Scaling groups. Unfortunately, it is only available as part of CloudFormation API, so there is no other option but to manage it using CloudFormation. As a workaround, you can wrap this CloudFormation stack resource in Terraform to have the best of both worlds. You can read more about setting up rolling updates in my previous post about [ECS cluster upgrades](https://devblog.xero.com/automating-ecs-cluster-upgrades-with-cloudformation-and-lambda-2b4f1bec575d).

<iframe src="https://medium.com/media/31da1c34f61f35e1f448fdfcd3467d37" frameborder=0></iframe>
