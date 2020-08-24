
# Terraform Poka-Yokes — Writing Effective, Scalable, Dynamic, and Error-Resistant Terraform



*Poka-yoke is a Japanese term that means ‘mistake-proofing’ or ‘inadvertent error [prevention](https://en.wiktionary.org/wiki/prevention)’. A poka-yoke is any mechanism in any process that helps an equipment operator avoid (yokeru) mistakes (poka). Its purpose is to eliminate product defects by preventing, correcting, or drawing attention to [human errors](https://en.wikipedia.org/wiki/Human_error) as they occur. — [Wikipedia](https://en.wikipedia.org/wiki/Poka-yoke)*

Providing software — of any kind — as a service requires building a lot of poka-yokes. In my role as a Senior DevOps Engineer/SRE, my job is to provide easy, automated ways for developers to develop and manage their applications. This means ensuring: 1) We provide things that scale — Capital One shouldn’t need to hire more of me on a 1–1 relationship to teams; and 2) Developers have a good experience while deploying their applications.

A while back I read a great article describing an Infrastructure as Code (IaC) called the Infrastructure-Application Pattern (I-A). The full six-piece series describing the general implementation with practical examples is available over on [Martin Atkins Blog](https://apparently.me.uk/terraform-environment-application-pattern/overview.html). I wanted to extend the ideas in this article and discuss other possible benefits and implementations of the I-A Pattern, as well as describe rules that allow you to build in poka-yokes.

As a quick overview, the I-A pattern represents a method of writing IaC and managing said infrastructure that abstracts out the shared infrastructure and allows developers control of the application-specific infrastructure. For my teams, that means abstracting out the management of the ECS cluster, ALB, Security Groups, R53 rules, databases, and S3 buckets into shared terraform that the platform/SRE team can manage. This gives the developers control of and responsibility for their individual application, it’s routing rules, ports, memory/cpu, environment variables, and the actual data stored in datastores like S3 and DBs. Here at Capital One, this extends even further for us, because in order to manage AWS at scale we have teams that manage VPCs, Subnets, and other pieces of massively shared infrastructure. Thus, our I-A pattern becomes a multi-layered pyramid of shared infrastructure that allows us to manage it with a shared responsibility model that scales.

In contrast to Martin Atkins’ original article, I’ve chosen not to use Consul as a storage for the data shared between infrastructure levels. Instead, I’ve made the choice to do remote data calls directly to the provider APIs. This theoretically allows for account infrastructure to be managed using CloudFormation, Terraform, AWS CDK, or some other DSL, allowing my team to simply write Terraform, and then finally allowing the developers to manage their applications using something else. In this scenario, we don’t have to rely on a DSL interfacing with Consul, but instead can just allow the DSL to grab and filter from AWS directly. In our case, the application teams also use Terraform to manage their apps’ deployments which makes data calls even more useful.

Much like Martin Atkins, I enjoy practical examples to complement broad, abstract concepts. Let’s walk through an example ECS application and cluster that is deployed and managed separately. As we walk through I’ll be sharing some general rules that I’ve found to allow for easier Terraform management.

## Rule 1: Use Terraform Modules to abstract specific pieces of infrastructure into logical groupings

The nice people over at [gruntworks.io](https://www.gruntwork.io/) (writers of *Terragrunt, Terratest, and Terraform: Up & Running!*) have almost perfected this. If you look into their modules on [Github](https://github.com/gruntwork-io) you’ll see two different levels of modules.

First is a low level un-opinionated module. This doesn’t have to be a single resource (though depending on the size or complexity of said resource it could) but rather a set of useful resources grouped together. An example of this could be as follows:

    resource "aws_ecs_service" "service" {
        name = "${var.service-name}"
        cluster = "${data.aws_ecs_cluster.ecs-cluster.id}"
        task_definition = "${aws_ecs_task_definition.task_definition.arn}"
        iam_role = "${var.ecs_service_role}"

        load_balancer {
            target_group_arn = "${aws_lb_target_group.target_group.arn}"
            container_name = "${var.service-name}"
            container_port = "${var.container-port}"

        }

        ordered_placement_strategy {
            type = "spread"
            field = "attribute:ecs.availability-zone"
        }
    }

    resource "aws_appautoscaling_target" "service_asg" {
        max_capacity = "${var.asg_max}"
        min_capacity = "${var.asg_min}"
        resource_id = "service/${var.cluster-name}/${aws_ecs_service.service.name}"
        role_arn = "${var.ecs_service_role}"
        scalable_dimension = "ecs:service:DesiredCount"
        service_namespace = "ecs"
    }

    resource "aws_appautoscaling_policy" "up" {
        name = "${aws_ecs_service.service.name}_scale_up"
        service_namespace = "ecs"
        resource_id = "service/${var.cluster-name}/${aws_ecs_service.service.name}"
        scalable_dimension = "ecs:service:DesiredCount"

        step_scaling_policy_configuration {
            adjustment_type = "ChangeInCapacity"
            cooldown = 60
            metric_aggregation_type = "Maximum"
            step_adjustment {
                metric_interval_lower_bound = 0
                scaling_adjustment = 1
            }
        }

        depends_on = ["aws_appautoscaling_target.service_asg"]
    }

    resource "aws_appautoscaling_policy" "down" {
        name = "${aws_ecs_service.service.name}_scale_down"
        service_namespace = "ecs"
        resource_id = "service/${var.cluster-name}/${aws_ecs_service.service.name}"
        scalable_dimension = "ecs:service:DesiredCount"
        step_scaling_policy_configuration {
            adjustment_type = "ChangeInCapacity"
            cooldown = 60
            metric_aggregation_type = "Maximum"
            step_adjustment {
                metric_interval_lower_bound = 0
                scaling_adjustment = -1
            }
        }
        depends_on = ["aws_appautoscaling_target.service_asg"]
    }

    /* metric used for auto scale */

    resource "aws_cloudwatch_metric_alarm" "service_mem_high" {
        alarm_name = "${aws_ecs_service.service.name}_memory_high"
        comparison_operator = "GreaterThanOrEqualToThreshold"
        evaluation_periods = "2"
        metric_name = "MemoryUtilization"
        namespace = "AWS/ECS"
        period = "60"
        statistic = "Maximum"
        threshold = "50"
        dimensions {
            ServiceName = "${aws_ecs_service.service.name}"
        }
        alarm_actions = ["${aws_appautoscaling_policy.up.arn}"]
        ok_actions = ["${aws_appautoscaling_policy.down.arn}"]
    }

This module might be called ecs-service. It creates an ECS service (using EC2 as the service type), an autoscaling target, and two metrics an auto-scale up and down policy. This is a moderately un-opinionated module that can be reused in a variety of places, especially in more opinionated modules that create logical groupings of services based on applications or infrastructure in your environment.

You might then call this in another more opinionated module specific to an application. It might include SNS topics for notifications, or SQS queues. These modules should be far more opinionated. They should form the logical groupings of base modules required to deploy a team’s applications in a standard method.

**This is the first poka-yoke.** *By requiring the use of these more opinionated modules to create applications you can go a long way to ensuring best practices.* Things like: requiring encryption, or preventing bad Security Groups, managing IAM roles or securing ACLs on S3 buckets. You can obviously allow developers or teams to add Terraform as well. However, building these modules properly should prevent a ton of misconfiguration issues with sane defaults.

## Rule 2: Use Terraform Data calls to provide information

I started my career in tech as a helpdesk monkey. I always ended up with a bunch of lists of machines or servers or phones or other pieces of equipment that were constantly out of date. I quickly learned the value of 1) consistent naming and 2) using APIs to look up data on the fly rather than relying on humans to gather accurate data.

**Here is poka-yoke number two.** *Don’t force your developers to maintain endless lists of data or variables. *Just look it up. Humans aren’t built for repetitive tasks; computers are. Utilize these strengths and build in lookups and filters into your terraform. Use filters to look up acceptable AMIs. Here is an example to look up custom AMIs that have been tagged with AmazonLinux-x64-HVM-Enc-<date>.

    data “aws_ami” “latest” {
        most_recent = true
        owners = [
            “self”,
        ]

        filter {
            name = “state”
            values = [
                “available”,
            ]
        }
        filter {
            name = “tag:release”
            values = [
                “ga”,
            ]
        }
        name_regex = “AmazonLinux-x64-HVM-Enc-[0–9]{6}(-[0–9]+)?$”
    }

Don’t make developers maintain clusters or ALB references. Look them up. It’s incredibly easy to get the information you want for a lookup against a provider’s APIs.

    data “aws_ecs_cluster” “ecs-cluster” {
        cluster_name = “${var.cluster-name}”
    }

    data “aws_lb” “alb” {
        name = “${var.alb-name}”
    }

These are simple examples, but that’s because the magic for this happens in how you name your infrastructure and how you interpolate the names together.

## Rule 3: Be Smart About Where Interpolation and Concatenation Happens

If you follow rule 2, you’ll quickly realize that keeping maps of variables to maintain consistency across environments is an easy way to have mistakes happen. Someone fat fingers a copy paste, or name, etc. Managing different Terraform files for each environment leads to the possibility of even greater issues in that variables have to be changed in multiple places, copied multiple times, etc. Instead we can use interpolation to build your variables and infrastructure names on the fly. Just don’t do that inside the modules.

If you want consistent naming you might do something like:

    resource “aws_ecs_cluster” “cluster” {
        name = “${var.app-name}-cluster-${var.env}”
    }

This allows for easy lookups. Provide env based on some input parameters, keep app-name as a variable used for all kinds of naming, and you’ll quickly have resources named consistently and easily to find via console or CLI.

But, if you do exactly what I listed above, you’ll quickly discover that it doesn’t scale. Because if you need to change that interpolation then everything breaks. And then you get to spend forever updating every other piece of Terraform that depends on you.

Instead, consider this interpolation (ideally) when you call a module.

    resource “aws_ecs_cluster” “cluster” {
        name = “${var.cluster-name}”
    }

    module “ecs-cluster” {
       source = “<path>”
       cluster-name = “${var.app-name}-cluster-${var.env}”
    }

Now, while you might still have a few places to update if your concatenation changes, you aren’t going to break everything at the same time.

If you need to concatenate inside a module (to provide a better developer experience or require certain naming) make sure to then use default definitions in your variables liberally and intelligently. Set a useful default and you can also minimize the breaking changes you introduce.

## Rule 4: Implement State Locking for Ease of Deployments

**This is poka-yoke number 3.** *State-locking is a perfect example of something that prevents human mistakes and/or notifies if there is a possible issue.*

[Jessica G has a great article on using state locking for Terraform](https://medium.com/@jessgreb01/how-to-terraform-locking-state-in-s3-2dc9a5665cb6) and I won’t rehash it here other than to explain the importance of doing so. If you don’t use state locking may have some headaches. If you don’t have some other way (like limiting the number of jenkins jobs running at the same time) to prevent simultaneous updates state locking becomes even more important. State locking is arguably the only sane method to manage it. So please implement it. It takes 5 minutes to implement and will prevent many headaches.

The concept of poka-yokes should drive a lot of the decisions we make. DevOps is a mindset. It’s built on the principle of increased control and responsibility and poka-yokes allow developers to fail fast, early, and without breaking things they shouldn’t. We’ve been implementing the I-A pattern with poka-yokes in our IaC for a while for the six teams we support. Between them they run some 75+ microservices in a variety of clusters and with a variety of requirements and traffic.

They deploy a lot. Non-Prod environments might see some 100 deployments on a high day. And building in poka-yokes into the Terraform provides a better developer experience and allows for less downtime and miscommunication.

My team can provide scalable support for the infrastructure while allowing developers to retain control of their application. And, allowing developers to retain control of their application all the way to production is fundamentally what DevOps is about. There is definitely some added complexity but that complexity can be managed by well structured and written Terraform. More fundamentally, that complexity is inherent in any system that tries to grant developers control and a lot of it represents people problems, not technology problems. And technology can never solve people problems, only mitigate them.

*DISCLOSURE STATEMENT: © 2020 Capital One. Opinions are those of the individual author. Unless noted otherwise in this post, Capital One is not affiliated with, nor endorsed by, any of the companies mentioned. All trademarks and other intellectual property used or displayed are property of their respective owners.*
