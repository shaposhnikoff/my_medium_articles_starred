Unknown markup type 10 { type: [33m10[39m, start: [33m150[39m, end: [33m167[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m79[39m, end: [33m94[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m75[39m, end: [33m84[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m343[39m, end: [33m357[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m63[39m, end: [33m121[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m40[39m, end: [33m91[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m131[39m, end: [33m147[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m177[39m, end: [33m214[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m32[39m, end: [33m46[39m }

# Terraform Remote Backend Demystified

Terraform Remote Backend Demystified

### A quick start guide to explain Terraform backends and how your deployments can benefit from a non-local state file storage.

***You are reading this because:**
 — you already know what [Terraform](https://www.terraform.io/) is, and your knowledge of it is wedged somewhere in the middle between beginner and intermediate.*

## About Terraform

The concept behind the *Infrastructure as Code* has narrowed the distance between developers, architects, systems and network administrators. 
**Today this sounds already like a boring, old mantra. **
With a terminal, a few bytes of code and **no ssh logins**, we can spin up, in minutes, a production ready Kubernetes cluster, capable of heavy workloads. 
And here it comes **Terraform**: an extremely powerful tool written in Go, that helps us writing, planning, creating highly **predictable** and **stateful** Infrastructures as Code.

## Stage 1: Terraform State

Terraform keeps the **state** of the managed infrastructure and its configuration. By default the state is stored locally, in a JSON formatted file named terraform.tfstate. From the first lift to the latest change of our infrastructure’s plan, every chunk of information allowing to map real world resources to the terraform configuration and to keep track of metadata, is saved in the state file. This brings benefits and opens to scenarios like versioning, debugging, performance monitoring, rollbacks, rolling updates, immutable deployments, traceability, self healing capabilities, and so on.

### *Let’s scratch the surface to see how it works*

Terraform uses the **state** to create **plans,** which are the representation of the resources we are deploying and the changes we are applying.
Prior to any operation, a **refresh** is performed to synchronise the **state** with the running resources, if any. Hence, the state file contains very sensitive information and plays a crucial role in the entire management process of our infrastructure. Apparently trivial topics then, like “how to store the state”, “where to store state”, “how to secure it”, should be addressed paying the right amount of attention.

In a wide range of scenarios, the default strategy to store the state file locally, is far from being a good idea.
> # *What if the hard drive gets corrupted?
> # What if it is breached?
> # What if I will be working in a team?*

It is enough running a small pipeline within a personal project, to make the local state a loser strategy.

What determines how the state is stored and loaded? How do we modify the default behaviour? The answer is “Backend”.

## Stage 2: Terraform Backends

A “backend” in Terraform is an abstraction that determines how the state is handled and the way certain operations are executed, enabling a number of important features. 
Let’s walk through the gifts we get out of the box, when a remote backend is configured.
 
***Backends can store the state remotely and protect it with locks to prevent corruption**.* This makes possible for a team to work with ease, or, for instance, to run Terraform within a pipeline.

***Better protection for sensitive data**. *When the state is retrieved from a backend, it get stored in memory and never persisted on a local drive. Despite the memory can be anyway exploited, this makes it certainly more complicated, for bad actors, to exfiltrate or corrupt the content held within the state. However, a misconfiguration of the remote storage can equally lead to breaches.

***Remote operations**:* For large infrastructures or when pushing specific changes, terraform apply can take quite a long time. Normally, for the entire duration of the deployment, it is not possible to disconnect the client without compromising its execution. Some of the backends available in terraform however, can be delegated to execute remote operations, allowing the client to safely go offline. Along with remote state storage and locking, remote operations are a big help for teams and more structured environments.

## Stage 3: Configuration

Backends are configured in Terraform files with the HLC syntax, inside the terraform section. The following, simplified, snippet shows how a remote backend can be enabled leveraging an AWS s3 bucket, where the *terraform.tfstate* will persist.

    terraform {  
        backend "s3" {
            bucket = "<your_bucket_name>"
            key    = "terraform.tfstate"    
            region = "<your_aws_region>"  
        }
    }

When configuring a backend rather than the default for the first time, Terraform will provide the option to migrate the current state to the new backend, in order to transfer the existing data and not losing any information.
It is recommended, though not mandatory, to manually back up the state before initialising any new backend by running terraform init. To do that, it is enough to copy the state file outside the scope of the project, in a different folder. The initialisation process should create a backup as well.

Time for coding and demonstrate how to set up a remote backend, with a real life example.

## Stage 4: The AWS s3 Backend

Setting up a Terraform backend it is relatively easy. Let’s see how to implement one with AWS s3.
To run the code of the example, be sure to have available AWS IAM credentials with enough permissions to create/ delete s3 buckets and put bucket policies.
> This example leverages the **aws cli**, assuming it is already installed and configured. To set it up, please refer to the online official documentation at [this link](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html).

Let’s create a bucket, for instance in the region *eu-west-1* *(EU — Ireland),* named *terraform-backend-store. *To do so, open a terminal and run the following command:

    aws s3api create-bucket --bucket terraform-backend-store \
        --region eu-west-1 \
        --create-bucket-configuration \
        LocationConstraint=eu-west-1

    # Output:
    {
        "Location": "[http://terraform-backend-store.s3.amazonaws.com/](http://terraform-backend-store.s3.amazonaws.com/)"
    }

Once the bucket is created, it needs to be properly configured.
For a bucket that holds the Terraform state, it’s a *good idea* to enable the server side encryption. To do so, and keeping it simple, let’s get back to the terminal and set the server side encryption to AES256 (*Although it’s out of scope for this story, I recommend to use the kms and implement a proper key rotation)*:

    aws s3api put-bucket-encryption \
        --bucket terraform-backend-store \
        --server-side-encryption-configuration={\"Rules\":[{\"ApplyServerSideEncryptionByDefault\":{\"SSEAlgorithm\":\"AES256\"}}]}

    # Output: expect none when the command is executed successfully

Next, it’s important to restrict the access to the bucket.
Create an unprivileged IAM user:

    aws iam create-user --user-name terraform-deployer

    # Output:
    **{**
        "User"**:** **{**
            "UserName"**:** "terraform-deployer"**,**
            "Path"**:** "/"**,**
            "CreateDate"**:** "2019-01-27T03:20:41.270Z"**,**
            "UserId"**:** "AIDAIOSFODNN7EXAMPLE"**,**
            "Arn"**:** "arn:aws:iam::123456789012:user/terraform-deployer"
        **}**
    **}**

Take note of the **Arn** from the command’s output (it looks like: “Arn”**:** “arn:aws:iam::123456789012:user/terraform-deployer”).

To properly interact with the s3 service and DynamoDB at a later stage, our IAM user must be granted with a sufficient set of permissions. It is recommended to have **severe restrictions** in place in production environments, though, for sake of simplicity, in this example we will generously assign *AmazonS3FullAccess *and* AmazonDynamoDBFullAccess*:

    aws iam attach-user-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess --user-name terraform-deployer

    # Output: expect none when the command is executed successfully

    aws iam attach-user-policy --policy-arn arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess --user-name terraform-deployer

    # Output: expect none when the command is executed successfully

Back to our bucket now. The freshly created IAM user, must be allowed to execute the necessary actions, and have the terraform backend running smoothly.
Let’s create a policy file as follows:

    cat <<-EOF >> policy.json
    {
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": {
                    "AWS": "arn:aws:iam::123456789012:user/terraform-deployer"
                },
                "Action": "s3:*",
                "Resource": "arn:aws:s3:::terraform-remote-store"
            }
        ]
    }
    EOF

It will grant to the principal with arn* *“arn:aws:iam::123456789012:user/terraform-deployer”, to execute all the available actions (“Action”: “s3:*") against the bucket with arn* *“arn:aws:s3:::terraform-remote-store”. Again, in production is desired to force way stricter policies. For reference, have a look to the [AWS Policy Generator](https://awspolicygen.s3.amazonaws.com/policygen.html).
Back to the terminal and run the command as shown below, to enforce the policy in our bucket:

    aws s3api put-bucket-policy --bucket terraform-remote-store --policy file://policy.json

    # Output: none

As last step, we will enable the bucket’s versioning:

    aws s3api put-bucket-versioning --bucket terraform-remote-store --versioning-configuration Status=Enabled

This will allow to save different versions of the infrastructure’s state and rollback easily to a previous stage without struggling.

The AWS s3 bucket is ready, time to integrate it with Terraform. Listed below, is the minimal configuration required to set up this remote backend:

    # terraform.tf

    terraform {  
        backend "s3" {
            bucket  = "terraform-remote-store"
            encrypt = true
            key     = "terraform.tfstate"    
            region  = "eu-west-1"  
        }
    }

    ...

    # the rest of your configuration and resources to deploy

Once in place, terraform must be initialised *(again)*.

    terraform init

The remote backend is ready for a ride, test it.

### What about locking?

Storing the state remotely brings a pitfall, especially when working in scenarios where several tasks, jobs and team members will have access to it. Under these circumstances, the risk of multiple concurrent attempts to make changes to the state, is high. Here comes to help the **lock**, a feature that prevents the state file to be opened while already in use.

Back to our example, the lock can be implemented by creating an **AWS DynamoDB Table**, that will be used by terraform to set and unset the locks.

We can create the table resource using terraform itself:

    # create-dynamodb-lock-table.tf

    resource "aws_dynamodb_table" "dynamodb-terraform-state-lock" {
      name           = "terraform-state-lock-dynamo"
      hash_key       = "LockID"
      read_capacity  = 20
      write_capacity = 20

    attribute {
        name = "LockID"
        type = "S"
      }

    tags {
        Name = "DynamoDB Terraform State Lock Table"
      }
    }

and deploy it:

    terraform plan -out "planfile" ; terraform apply -input=false -auto-approve "planfile"

Once the command execution is completed, the locking mechanism must be added to our backend configuration as follow:

    # terraform.tf

    terraform {  
        backend "s3" {
            bucket         = "terraform-remote-store"
            encrypt        = true
            key            = "terraform.tfstate"    
            region         = "eu-west-1"
            dynamodb_table = "terraform-state-lock-dynamo"
        }
    }

    ...

    # the rest of your configuration and resources to deploy

All done. Remember to run again terraform init and enjoy the remote backend’s features and joys.

~ Till the next time.
