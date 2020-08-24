
# Creating and attaching an AWS IAM role, with a policy to an EC2 instance using Terraform scripts

What is Terraform?

[This](https://www.terraform.io/) is an infrastructure as a code, which is equivalent to the [AWS CloudFormation](https://aws.amazon.com/cloudformation/), that allows the user to create, update, and version any of the Amazon Web Services (AWS) infrastructure.

### Why Terraform?

Terraform utilizes the cloud provider APIs (Application programming interfaces**)** to provision infrastructure, hence there’re no authentication techniques after, what the customer is using with the cloud provider already. This could be considered as one of the best option, in terms of maintainability, security and ease-of-use.

The motivation behind this post is to, illustrate an example of:

1. creating an AWS IAM role using ***terraform.***

1. creating an IAM policy using ***terraform.***

1. attaching the policy to the role using ***terraform.***

1. creating the IAM instance profile using **terraform**.

1. Assigning the IAM role, to an EC2 instance on the fly using ***terraform***.

### 1. Creating an AWS IAM role using Terraform:

This is where, the [IAM role](https://www.terraform.io/docs/providers/aws/r/iam_role.html) creation will be done. The assume_role_policy parameter is a must to be given within the resource block, and there are other optional parameters as well such as name, path, description etc.

The terraform script:

<iframe src="https://medium.com/media/a91b005b0b7284dbf8a9c48b61ff88a3" frameborder=0></iframe>
> The resource block above, constructs a resource of the stated **TYPE** (i.e. the initial parameter “aws_iam_role”) and **NAME** (i.e. the second parameter “ec2_s3_access_role”). The integration of the type and name must be distinctive. Within the block (the { }) is the configuration for the resource.

A [resource](https://www.terraform.io/docs/configuration/resources.html) component in terraform, constructs a resource, of the given **TYPE** (first parameter) and **NAME** (second parameter) when defining a resource. As an example, if the script is:

    resource "aws_iam_instance_profile" "test_profile" {                             name  = "test_profile"                         
    roles = ["${aws_iam_role.ec2_s3_access_role.name}"]
    }

So in the above block, aws_iam_instance_profile is the **TYPE **and test_profile is the **NAME**. The combination of the type and name must be unique.

[assume_role_policy](https://www.terraform.io/docs/providers/aws/r/iam_role.html#assume_role_policy) parameter in the above resource block, allows an entity, permission to assume the role.

The assume role policy:

<iframe src="https://medium.com/media/5c0d2a19ca3cf94980091da2e34ebeec" frameborder=0></iframe>

### 2. Creating an AWS IAM policy using Terraform:

[This](https://www.terraform.io/docs/providers/aws/r/iam_policy.html) is where we need to define the required policy (i.e. permissions) according to the necessities. For example, allowing the IAM role to access all the S3 buckets within the region. Providing the policy is a required parameter, where as there are other parameters as well such as arn, path, id etc.

The terraform script:

<iframe src="https://medium.com/media/295b433e1b3c2f2f3c5783759d50878f" frameborder=0></iframe>

The policy parameter in the above block, requires an IAM policy in a **JSON **format. What the following policy does is that, it allows the IAM role to access all the S3 buckets and also to perform any kind of actions (i.e. list buckets, put objects, delete objects etc.) on those buckets.

The IAM policy:

<iframe src="https://medium.com/media/fdf405aa50c1be0b9601ea4831441ec0" frameborder=0></iframe>

### 3. Attaching the policy to the role using Terraform:

This is where, we’ll be attaching the policy which we wrote above, to the role we created in the first step.

The terraform script:

<iframe src="https://medium.com/media/a38c7fc24e49e0bada9b876aabbf5fdc" frameborder=0></iframe>

The [aws_iam_policy_attachment](https://www.terraform.io/docs/providers/aws/r/iam_policy_attachment.html) in the above resource block, is used to attach a Managed IAM Policy to user(s), role(s), and/or group(s). But in our case, it was a role. The value for the ***roles*** parameter has been accessed from the resource block which we created in step 1.

Value of the role = ${aws_iam_role.ec2_s3_access_role.name}

Explanation:

> aws_iam_role is the type of the resource block which we created in step 1.

> ec2_s3_access_role is the name of the variable which we defined.

> name is a property of that resource block.

The same thing applies to the value for ***policy_arn.***

### 4. Creating the IAM instance profile using **terraform**:

This is the resource, which must be used to tag the IAM role to the EC2 instance. As in, when we are creating the resource block for an [EC2 instance](https://www.terraform.io/docs/providers/aws/r/instance.html), in order for us to assign the role to that instance, it expects the [aws_iam_instance_profile](https://www.terraform.io/docs/providers/aws/r/iam_instance_profile.html) to be given as a parameter.

The terraform script:

<iframe src="https://medium.com/media/fe98859e0e2a003300b8eee5f551d1ff" frameborder=0></iframe>

The value for the ***roles*** parameter has been accessed from the resource block which we created in step 1.

### 5. Assigning the IAM role, to an EC2 instance on the fly using ***terraform***:

Here we will be creating a basic free tier [EC2 instance](https://www.terraform.io/docs/providers/aws/r/instance.html) and attaching the iam instance profile which we created above in the step 4.

The terraform script:

<iframe src="https://medium.com/media/ea4877efd2fa84a1b99b0b2fe6d223ad" frameborder=0></iframe>

The ***tags ***parameter is defined to identify or rather differentiate the EC2 instance from the others. It simply represents a mapping. The value of ***ami, ***is being retrieved from the predefined variables which are defined on a different terraform script as shown below:

<iframe src="https://medium.com/media/111d931cac6da814c8ed006da59738a3" frameborder=0></iframe>

The following commands should be executed from the terminal in the respective order within the directory where the scripts are being saved.

1. Initializing a new or an existing Terraform configuration

    terraform init

2. Generate and show an execution plan from the resources we’re trying to provision

    terraform plan

3. Validating the Terraform files

    terraform validate

4. Builds or changes the infrastructure

    terraform apply

The complete list of commands are available [here](https://www.terraform.io/docs/commands/index.html).

Complete source-code is available here for grab:

[https://github.com/Kulasangar/terraform-demo](https://github.com/Kulasangar/terraform-demo)
