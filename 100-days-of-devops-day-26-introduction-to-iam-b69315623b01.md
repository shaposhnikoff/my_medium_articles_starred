
# 100 Days of DevOps — Day 26-Introduction to IAM

Welcome to Day 26 of 100 Days of DevOps, Let continue our journey with terraform and see how we can terraform IAM.
> What is IAM?

*Identity and Access Management(IAM) is used to manage AWS*

* *Users*

* *Groups*

* *Roles*

* *Api Keys*

* *IAM Access Policies*

*and it provide access/access-permissions to AWS resources(such as EC2,S3..)*

![](https://cdn-images-1.medium.com/max/5708/1*FoOii_n9Q7osUVKjFefL9g.png)

*If we notice at the right hand side at the top of console it says **Global** i.e creating a user/groups/roles will apply to all regions*

*To create a new user,Just click on Users on the left navbar*

![](https://cdn-images-1.medium.com/max/5760/1*816nqQDqz0jLrCssLStbOQ.png)

*By default any new IAM account created with NO access to any AWS services(**non-explicit deny**)*

![](https://cdn-images-1.medium.com/max/5556/1*v4KsRYDWI7V97_EIwhER-g.png)

*Always follow the best practice and for daily work try to use a account with least privilege(**i.e non root user**)*

***IAM Policies: **A policy is a document that formally states one or more permissions.For eg: IAM provides some pre-built policy templates to assign to users and groups*

* ***Administrator access**: Full access to AWS resources*

* ***Power user access**: Admin access except it doesn’t allow user/group management*

* ***Read only access**: As name suggest user can only view AWS resources*

*Default policy is explicitly deny which will override any explicitly allow policy*

![](https://cdn-images-1.medium.com/max/5732/1*duCFoAWP8bSJzaQjkiyDHw.png)

*Let take a look at these policies*

***AdministratorAccess***

    *{
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "*",
          "Resource": "*"
        }
      ]
    }*

*We can create our own custom policy using **policy generator **or written from scratch*

![](https://cdn-images-1.medium.com/max/5736/1*SI5lEp82sdf-SBtGI5KqfA.png)

*So Custom Policy where everything denies for EC2 resources*

    *{
     “Version”: “2012–10–17”,
     “Statement”: [
     {
     “Sid”: “Stmt1491718191000”,
     **“Effect”: “Deny”,**
     “Action”: [
     “ec2:*”
     ],
     “Resource”: [
     “*”
     ]
     }
     ]
    }*

* *More than one policy can be attached to a user or group at the same time*

* *Policy cannot be directly attached to AWS resources(eg: EC2 instance)*

* *There is a really nice tool [**https://policysim.aws.amazon.com](https://policysim.aws.amazon.com/home/index.jsp?#) **which we can use to test and troubleshoot IAM and resource based policies*

*Below is the simulation I run where I created a test user who has only Amazon S3 read only access*

![](https://cdn-images-1.medium.com/max/5472/1*CuwNRGzsP4cFtVSvwk4hsQ.png)

*Now let me run the simulation,as you can see it’s a nice way to test your policies*

![](https://cdn-images-1.medium.com/max/5644/1*RVMBFNOl0vxaDHj-O8KAdA.png)

*Let’s create IAM user using terraform*

    *resource "aws_iam_user" "example" {
      name = "prashant"
    }*
[**AWS: aws_iam_user — Terraform by HashiCorp**
*Provides an IAM user.*www.terraform.io](https://www.terraform.io/docs/providers/aws/r/iam_user.html)

*Now if I want to create two IAM user*

*One way to achieve the same is copy paste the same piece of code but that defeats the whole purpose of DRY.*

*Terraform provides meta parameters called count to achieve the same i.e to do certain types of loops.The count is a meta parameter which defines how many copies of the resource to create.*

    *resource "aws_iam_user" "example" {
      count = 3
      name = "prashant"
    }*

*One problem with this code is that all three IAM users would have the same name, which would cause an error since usernames must be unique*

*To accomplish the same thing in Terraform, you can use count.index to get the index of each “iteration” in the “loop”:*

    *resource “aws_iam_user” “example” {
     count = 3
     name = “prashant.${count.index}”
    }*

*If we run the plan again we will see terraform want to create three IAM user, each with a different name(prashant.0, prashant.1, prashant.2)*

*Think of a real-world scenario, we are hardly going to see names like prashant.0–2*

*The solution for this issue, If we combine count.index with interpolation functions built into Terraform, you can customize each “iteration” of the “loop” even more. To achieve this we need two interpolation functions length and element*
[**Interpolation Syntax — Terraform by HashiCorp**
*Embedded within strings in Terraform, whether you’re using the Terraform syntax or JSON syntax, you can interpolate…*www.terraform.io](https://www.terraform.io/docs/configuration/interpolation.html)
> *element(list, index) — Returns a single element from a list at the given index. If the index is greater than the number of elements, this function will wrap using a standard mod algorithm. This function only works on flat lists.*

*OR*

*The element function returns the item located at INDEX in the given LIST*
> *length(list) — Returns the number of members in a given list or map, or the number of characters in a given string.*

*OR*

*The length function returns the number of items in LIST (it also works with strings and maps)*

    *#variables.tf
    variable "username" {
      type = "list"
      default = ["prashant","pankaj","ashish"]
    }*

    *#main.tf
    resource "aws_iam_user" "example" {
      count = "${length(var.username)}"
      name = "${element(var.username,count.index )}"
    }*

*Now when you run the plan command, you’ll see that Terraform wants to create three IAM users, each with a unique name*

*One thing to note as we have used count on a resource, it becomes the list of resources rather than just a single resource.*

*For example, if you wanted to provide the Amazon Resource Name (ARN) of one of the IAM users as an output variable, you would need to do the following:*

    *# outputs.tf
    output “user_arn” {
     value = “${aws_iam_user.example.0.arn}”
    }*

*If you want the ARNs of all the IAM users, you need to use the splat character, “*”, instead of the index:*

    *# outputs.tf
    output “user_arn” {
     value = “${aws_iam_user.example.*.arn}”
    }*

*As we are done with creating IAM user, now let attach some policy with these users(As by default new user have no permission whatsoever)*

***IAM Policies** are JSON documents used to describe permissions within AWS. This is used to grant access to your AWS users to particular AWS resources.*

*IAM Policy is a json document*

![](https://cdn-images-1.medium.com/max/3384/1*4513wAq5vgFCPhHgUbw8Fg.png)

*Terraform provides a handy data source called the aws_iam_policy_document that gives you a more concise way to define the IAM policy*

    *data "aws_iam_policy_document" "example" {
      statement {
        actions = [
          "ec2:Describe*"]
        resources = [
          "*"]
      }
    }*

* *IAM Policy consists of one or more statements*

* *Each of which specifies an effect (either “Allow” or “Deny”)*

* *One or more actions (e.g., “ec2:Describe*” allows all API calls to EC2 that start with the name “Describe”),*

* *One or more resources (e.g., “*” means “all resources”)*

*To create a new IAM managed policy from this document, we need to use aws_iam_policy resource*

    *resource “aws_iam_policy” “example” {
     name = “ec2-read-only”
     policy = “${data.aws_iam_policy_document.example.json}”
    }*

*This code uses the count parameter to “loop” over each of your IAM users and the element interpolation function to select each user’s ARN from the list returned by aws_iam_user.example.*.arn.*

    *resource “aws_iam_user_policy_attachment” “test-attach” {
     count = “${length(var.username)}”
     user = “${element(aws_iam_user.example.*.name,count.index )}”
     policy_arn = “${aws_iam_policy.example.arn}”
    }*

***IAM Roles** are used to granting the application access to AWS Services without using permanent credentials.*

*IAM Role is one of the safer ways to give permission to your EC2 instances.*

*We can attach roles to an EC2 instance, and that allows us to give permission to EC2 instance to use other AWS Services eg: S3 buckets*

***Motivation***

* *Give EC2 instance access to S3 bucket*
> ***Step1***

* *Create a file iam.tf*

* *Create an IAM role by copy-paste the content of a below-mentioned link*

* *assume_role_policy — (Required) The policy that grants an entity permission to assume the role.*
[**AWS: aws_iam_role — Terraform by HashiCorp**
*Provides an IAM role.*www.terraform.io](https://www.terraform.io/docs/providers/aws/r/iam_role.html)

    *resource "aws_iam_role" "test_role" {
      name = "test_role"*

    *  assume_role_policy = <<EOF
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Action": "sts:AssumeRole",
          "Principal": {
            "Service": "ec2.amazonaws.com"
          },
          "Effect": "Allow",
          "Sid": ""
        }
      ]
    }
    EOF*

    *  tags = {
          tag-key = "tag-value"
      }
    }*

* *This is going to create IAM role but we can’t link this role to AWS instance and for that, we need EC2 instance Profile*
> *Step2*

* *Create EC2 Instance Profile*

    *resource "aws_iam_instance_profile" "test_profile" {
      name = "test_profile"
      role = "${aws_iam_role.test_role.name}"
    }*
[**AWS: aws_iam_instance_profile — Terraform by HashiCorp**
*Provides an IAM instance profile.*www.terraform.io](https://www.terraform.io/docs/providers/aws/r/iam_instance_profile.html)

* *Now if we execute the above code, we have Role and Instance Profile but with no permission.*

* *Next step is to add IAM Policies which allows EC2 instance to execute specific commands for eg: access to S3 Bucket*
> *Step3*

* *Adding IAM Policies*

* *To give full access to S3 bucket*

    *resource "aws_iam_role_policy" "test_policy" {
      name = "test_policy"
      role = "${aws_iam_role.test_role.id}"*

    *  policy = <<EOF
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Action": [
            "s3:*"
          ],
          "Effect": "Allow",
          "Resource": "*"
        }
      ]
    }
    EOF
    }*
> *Step4*

* *Attach this role to EC2 instance*

    *resource "aws_instance" "role-test" {
      ami = "ami-0bbe6b35405ecebdb"
      instance_type = "t2.micro"
    **  iam_instance_profile = "${aws_iam_instance_profile.test_profile.name}"**
      key_name = "mytestpubkey"
    }*

*It’s time to execute code*

*1: This will initialize the terraform working directory OR it will download plugins for a provider(example: aws)*

    *terraform init*

*2: Let you see what terraform will do before making the actual changes*

    *terraform plan*

*3: To actually create the instance we need to run terraform apply*

    *terraform apply*

*Looking forward from you guys to join this journey and spend a minimum an hour every day for the next 100 days on DevOps work and post your progress using any of the below medium.*

* *Twitter: [@100daysofdevops](http://twitter.com/100daysofdevops) OR @[lakhera2015](https://twitter.com/lakhera2015)*

* *Facebook: [https://www.facebook.com/groups/795382630808645/](https://www.facebook.com/groups/795382630808645/)*

* *Medium: [https://medium.com/@devopslearning](https://medium.com/@devopslearning)*

* *Slack: [https://devops-myworld.slack.com/messages/CF41EFG49/](https://devops-myworld.slack.com/messages/CF41EFG49/)*

* *GitHub Link:[https://github.com/100daysofdevops](https://github.com/100daysofdevops)*

***Reference***
[**100 Days of DevOps — Day 0**
*D-day is just one day away and finally, this is a continuation of the post(I posted a month earlier)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-0-4f2c9750542d)
[**100 Days of DevOps**
*Motivation*medium.com](https://medium.com/@devopslearning/100-days-of-devops-81faf13bf772)
