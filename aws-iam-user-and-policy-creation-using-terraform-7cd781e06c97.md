
# AWS IAM User and Policy Creation using Terraform

What is IAM(Identity and Access Management)

*IAM allows us to manage users, groups and roles and their level of access to the AWS Console.*

*Let’s create IAM user using terraform*

    *resource "aws_iam_user" "example" {
      name = "prashant"
    }*
[**AWS: aws_iam_user - Terraform by HashiCorp**
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
[**Interpolation Syntax - Terraform by HashiCorp**
*Embedded within strings in Terraform, whether you're using the Terraform syntax or JSON syntax, you can interpolate…*www.terraform.io](https://www.terraform.io/docs/configuration/interpolation.html)
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

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Join our community Slack and read our weekly Faun topics ⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
