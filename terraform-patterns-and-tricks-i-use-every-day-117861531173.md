
# Terraform patterns and tricks I use every day

Dear reader, in this small article I would like to share you some Terraform patterns and tricks I use the most so far after being Terraform user for a couple of month.

### 1. Deployment name

A lot of [resources](https://www.terraform.io/docs/configuration/resources.html) in Terraform take name argument. For some resource types the name has to be unique and has to be created (e. g. [aws_iam_role](https://www.terraform.io/docs/providers/aws/r/iam_role.html)), for some it's just one of the tags and may be omitted (e. g. [aws_instance](https://www.terraform.io/docs/providers/aws/r/instance.html)). Even despite the fact that Terraform may assign a random value of name for you when it's required, it's usually considered to be a good practice building that value based on the so called "current deployment's name" - a string variable that you pass to your top-level Terraform script. Doing this, you will be able to easily lookup your provider resources without Terraform, having just the value of deployment name, and it may simplify debugging and further support of your infrastructure. For those resource types, which have name attribute as **required** and **unique** across regions (like forementioned *aws_iam_role*), you should be using name_prefix attribute. The difference between name and name_prefix is that the latter uses a passed value only as a *prefix* for name, and Terraform appends some random string to it to guarantee the *uniqueness*. So these are the examples of using deployment_name:

    resource "aws_iam_role" "instance" {
      name_prefix = "${var.deployment_name}"
      description = "ECS cluster instance role for the ${var.deployment_name} deployment."
    
      assume_role_policy = <<EOF
    {
    "Version": "2008-10-17",
    "Statement": [
      {
        "Sid": "",
        "Effect": "Allow",
        "Principal": {
          "Service": "ec2.amazonaws.com"
        },
        "Action": "sts:AssumeRole"
      }
    ]
    }
    EOF
    }

    resource "aws_instance" "instance" {
      count = "${var.instance_count}"
      tags = {
        Name = "${var.deployment_name}-ecs-${count.index}"
      }
    }

### 2. Locals

Not all of the Terraform newcomers know about such a feature (and I used to be one of them) as [Local Value Configuration](https://www.terraform.io/docs/configuration/locals.html) or just locals. Without using locals, you always have to build a necessary value for the attribute right where you declare that attribute, and if it requires using some functions or ternary operator and resulting value is used in a couple of places, you have to copy that logic over and over, violating a [DRY](https://en.wikipedia.org/wiki/Don't_repeat_yourself) principle (with all the consequences). For example, without using locals:

    resource "aws_iam_role" "instance1" {
      name_prefix = "${var.name_prefix != "" ? var.name_prefix : var.deployment_name}"
      ...
    }
    
    resource "aws_iam_role" "instance2 " {
      name_prefix = "${var.name_prefix != "" ? var.name_prefix : var.deployment_mame}"
      ...
    }

With using locals:

    locals {
      name_prefix = "${var.name_prefix != "" ? var.name_prefix : var.deployment_mame}"
    }
    
    resource "aws_iam_role" "instance1" {
      name_prefix = "${local.name_prefix}"
      ...
    }
    
    resource "aws_iam_role" "instance2" {
      name_prefix = "${local.name_prefix}"
      ...
    }

As you can see, with locals we compute the value in one place, and use it referencing as local.<name> everywhere it's needed.

### 3. Modules

Terraform has a core feature called [modules](https://www.terraform.io/docs/modules/index.html). If you are developer or just know what OOP is, a solid mental model for *modules* would be:

* *module* is a *class* from OOP and encapsulates a piece of configuration

* whenever you do module { source = ... } you create an instance of that class (module)

* *module* takes [input variables](https://www.terraform.io/docs/configuration/variables.html)

* when *module* is instantiated, it’s executed: a *module* takes input parameters (forementioned variables), creates [side effects](https://en.wikipedia.org/wiki/Side_effect_(computer_science)) in a form of modifying your infrastructure, and spits an output (so that it can be used with the rest of configuration or in composition with other modules)

* *module* can return [output variables](https://www.terraform.io/intro/getting-started/outputs.html)

Java-like OOP pseudocode for declaring a module may look like this:

    // built-in TF interface
    interface Module {
      OutputVariable[] run()
    }
    
    // your own module directory creates this class
    class MyModule implements Module {
      MyModule(String source, variable[] variables)
    
      // that's where logic of your configuration is implemented
      OutputVariable[] run()
    }

and when using a module in TF

    module {
      source = "..."
      variable1 = "..."
      ...
    }

it can be mapped to

    (new MyModule(source, variables)).run()

### 4. Switch

Let’s consider the next scenario: you have a Terraform configuration, and based on a passed parameter, you would like to omit creating some of the resources, and create all other. Terraform doesn’t have if-else statement, but it has a [ternary operator](https://www.terraform.io/docs/configuration/interpolation.html#conditionals), so you can implement switch-like behavior with using resource's [count](https://www.terraform.io/docs/configuration/resources.html#using-variables-with-count) and ternary operator:

    resource "aws_instance" "instance1" {
      count = "${var.is_master == "" ? 0 : var.instance_count}"
      tags = {
        Name = "${var.deployment_name}-ecs-${count.index}"
      }
    }
    
    resource "aws_instance" "instance2" {
      count = "${var.instance_count}"
      tags = {
        Name = "${var.deployment_name}-ecs-${count.index}"
      }
    }

instance1 will be created only if is_master variable not empty, and it will have count = 0 otherwise and thus resource won't be created.

### Conclusion

Terraform is a really powerful tool, and [Infrastructure as Code](https://en.wikipedia.org/wiki/Infrastructure_as_Code) paradigm changed the way how deployment is done nowadays. [HCL](https://www.terraform.io/docs/configuration/syntax.html) has some limitations, but there are solutions to near any problem, however sometimes they may seem a little bit hacky and not that neat like a forementioned *switch* feature.
