Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m55[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m169[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m97[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m76[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m215[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m206[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m166[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m207[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m96[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m76[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m271[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m97[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m233[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m97[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m59[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m155[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m221[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m215[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m81[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m75[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m149[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m67[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m48[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m29[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m161[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m285[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m178[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m356[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m121[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m91[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m38[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m36[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m220[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m279[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m139[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m166[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m115[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m96[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m210[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m185[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m32[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m134[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m46[39m }

# How to setup an Amazon ECS cluster with Terraform

via GIPHY

ECS is Amazon’s Elastic Container Service. That’s greek for how you get docker containers running in the cloud. It’s sort of like Kubernetes without all the bells and whistles.

It takes a bit of getting used to, but This terraform how to, should get you moving. You need an EC2 host to run your containers on, you need a task that defines your container image & resources, and lastly a service which tells ECS which cluster to run on and registers with ALB if you have one.

**Join 38,000 others and [follow Sean Hull on twitter @hullsean](http://www.twitter.com/@hullsean).**

For each of these sections, create files: roles.tf, instance.tf, task.tf, service.tf, alb.tf. What I would recommend is create the first file roles.tf, then do:

$ terraform init
 $ terraform plan
 $ terraform apply

Then move on to instance.tf and do the terraform apply. One by one, next task, then service then finally alb. This way if you encounter errors, you can troubleshoot minimally, rather than digging through five files for the culprit.

This howto also requires a vpc. Terraform has [a very good community vpc](https://github.com/terraform-aws-modules/terraform-aws-vpc) which will get you going in no time.

I recommend deploying in the public subnets for your first run, to avoid complexity of jump box, and private IPs for ecs instance etc.

Good luck!

May the terraform force be with you!

### First setup roles

Roles are a really brilliant part of the aws stack. Inside of IAM or identity access and management, you can create roles. These are collections of privileges. I’m allowed to use this S3 bucket, but not others. I can use EC2, but not Athena. And so forth. There are some special policies already created just for ECS and you’ll need roles to use them.

These roles will be applied at the instance level, so your ecs host doesn’t have to pass credentials around. Clean. Secure. Smart!

resource "aws_iam_role" "ecs-instance-role" {
 name = "ecs-instance-role"
 path = "/"
 assume_role_policy = "${data.aws_iam_policy_document.ecs-instance-policy.json}"
 }

data "aws_iam_policy_document" "ecs-instance-policy" {
 statement {
 actions = ["sts:AssumeRole"]

principals {
 type = "Service"
 identifiers = ["ec2.amazonaws.com"]
 }
 }
 }

resource "aws_iam_role_policy_attachment" "ecs-instance-role-attachment" {
 role = "${aws_iam_role.ecs-instance-role.name}"
 policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonEC2ContainerServiceforEC2Role"
 }

resource "aws_iam_instance_profile" "ecs-instance-profile" {
 name = "ecs-instance-profile"
 path = "/"
 role = "${aws_iam_role.ecs-instance-role.id}"
 provisioner "local-exec" {
 command = "sleep 60"
 }
 }

resource "aws_iam_role" "ecs-service-role" {
 name = "ecs-service-role"
 path = "/"
 assume_role_policy = "${data.aws_iam_policy_document.ecs-service-policy.json}"
 }

resource "aws_iam_role_policy_attachment" "ecs-service-role-attachment" {
 role = "${aws_iam_role.ecs-service-role.name}"
 policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonEC2ContainerServiceRole"
 }

data "aws_iam_policy_document" "ecs-service-policy" {
 statement {
 actions = ["sts:AssumeRole"]

principals {
 type = "Service"
 identifiers = ["ecs.amazonaws.com"]
 }
 }
 }

**Related: [30 questions to ask a serverless fanboy](http://www.iheavy.com/2017/03/13/30-questions-to-ask-a-serverless-fanboy/)**

### Setup your ecs host instance

Next you need EC2 instances on which to run your docker containers. Turns out AWS has already built AMIs just for this purpose. They call them [ECS Optimized Images](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-optimized_AMI.html). There is one unique AMI id for each region. So be sure you’re using the right one for your setup.

The other thing that your instance needs to do is echo the cluster name to /etc/ecs/ecs.config. You can see us doing that in the user_data script section.

Lastly we’re configuring our instance inside of an auto-scaling group. That’s so we can easily add more instances dynamically to scale up or down as necessary.

#
 # the ECS optimized AMI's change by region. You can lookup the AMI here:
 # https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-optimized_AMI.html
 #
 # us-east-1 ami-aff65ad2
 # us-east-2 ami-64300001
 # us-west-1 ami-69677709
 # us-west-2 ami-40ddb938
 #

#
 # need to add security group config
 # so that we can ssh into an ecs host from bastion box
 #

resource "aws_launch_configuration" "ecs-launch-configuration" {
 name = "ecs-launch-configuration"
 image_id = "ami-aff65ad2"
 instance_type = "t2.medium"
 iam_instance_profile = "${aws_iam_instance_profile.ecs-instance-profile.id}"

root_block_device {
 volume_type = "standard"
 volume_size = 100
 delete_on_termination = true
 }

lifecycle {
 create_before_destroy = true
 }

associate_public_ip_address = "false"
 key_name = "testone"

#
 # register the cluster name with ecs-agent which will in turn coord
 # with the AWS api about the cluster
 #
 user_data = <> /etc/ecs/ecs.config
 EOF
 }

#
 # need an ASG so we can easily add more ecs host nodes as necessary
 #
 resource "aws_autoscaling_group" "ecs-autoscaling-group" {
 name = "ecs-autoscaling-group"
 max_size = "2"
 min_size = "1"
 desired_capacity = "1"

# vpc_zone_identifier = ["subnet-41395d29"]
 vpc_zone_identifier = ["${module.new-vpc.private_subnets}"]
 launch_configuration = "${aws_launch_configuration.ecs-launch-configuration.name}"
 health_check_type = "ELB"

tag {
 key = "Name"
 value = "ECS-myecscluster"
 propagate_at_launch = true
 }
 }

resource "aws_ecs_cluster" "test-ecs-cluster" {
 name = "myecscluster"
 }

**Related: [Is there a serious skills shortage in the devops space?](https://www.iheavy.com/2018/02/19/devops-skills-shortage-cloud-aws-gcp-automation/)**

### Setup your task definition

The third thing you need is a task. This one will spinup a generic nginx container. It’s a nice way to demonstrate things. For your real world usage, you’ll replace the image line with a docker image that you’ve pushed to ECR. I’ll leave that as an exercise. Once you have the cluster working, you should get the hang of things.

Note the portmappings, memory and CPU. All things you might expect to see in a docker-compose.yml file. So these tasks should look somewhat familiar.

data "aws_ecs_task_definition" "test" {
 task_definition = "${aws_ecs_task_definition.test.family}"
 depends_on = ["aws_ecs_task_definition.test"]
 }

resource "aws_ecs_task_definition" "test" {
 family = "test-family"

container_definitions = <

**Related: [Is AWS too complex for small dev teams?](https://www.iheavy.com/2016/04/26/is-aws-too-complex-for-small-dev-teams-startups/)**

### Setup your service definition

The fourth thing you need to do is setup a service. The task above is a manifest, describing your containers needs. It is now registered, but nothing is running.

When you apply the service your container will startup. What I like to do is, ssh into the ecs host box. Get comfortable. Then issue $ watch "docker ps". This will repeatedly run "docker ps" every two seconds. Once you have that running, do your terraform apply for this service piece.

As you watch, you'll see ECS start your container, and it will suddenly appear in your watch terminal. It will first show "starting". Once it is started, it should say "healthy".

resource "aws_ecs_service" "test-ecs-service" {
 name = "test-vz-service"
 cluster = "${aws_ecs_cluster.test-ecs-cluster.id}"
 task_definition = "${aws_ecs_task_definition.test.family}:${max("${aws_ecs_task_definition.test.revision}", "${data.aws_ecs_task_definition.test.revision}")}"
 desired_count = 1
 iam_role = "${aws_iam_role.ecs-service-role.name}"

load_balancer {
 target_group_arn = "${aws_alb_target_group.test.id}"
 container_name = "nginx"
 container_port = "80"
 }

depends_on = [
 # "aws_iam_role_policy.ecs-service",
 "aws_alb_listener.front_end",
 ]
 }

**Related: [Does AWS have a dirty secret?](https://www.iheavy.com/2016/06/07/does-aws-have-a-dirty-little-secret/)**

### Setup your application load balancer

The above will all work by itself. However for a real-world use case, you'll want to have an ALB. This one has only a simple HTTP port 80 listener. These are much simpler than setting up 443 for SSL, so baby steps first.

Once you have the ALB going, new containers will register with the target group, to let the alb know about them. In "docker ps" you'll notice they are running on a lot of high numbered ports. These are the hostPorts which are dynamically assigned. The container ports are all 80.

#
 #
 resource "aws_alb_target_group" "test" {
 name = "my-alb-group"
 port = 80
 protocol = "HTTP"
 vpc_id = "${module.new-vpc.vpc_id}"
 }

resource "aws_alb" "main" {
 name = "my-alb-ecs"
 subnets = ["${module.new-vpc.public_subnets}"]
 security_groups = ["${module.new-vpc.default_security_group_id}"]
 }

resource "aws_alb_listener" "front_end" {
 load_balancer_arn = "${aws_alb.main.id}"
 port = "80"
 protocol = "HTTP"

default_action {
 target_group_arn = "${aws_alb_target_group.test.id}"
 type = "forward"
 }
 }

You will also want to add a domain name, so that as your infra changes, and if you rebuild your ALB, the name of your application doesn't vary. Route53 will adjust as terraform changes are applied. Pretty cool.

resource "aws_route53_record" "myapp" {
 zone_id = "${aws_route53_zone.primary.zone_id}"
 name = "myapp.mydomain.com"
 type = "CNAME"
 ttl = "60"
 records = ["${aws_alb.main.dns_name}"]

depends_on = ["aws_alb.main"]
 }

**Related: [How to deploy on EC2 with vagrant](https://www.iheavy.com/2014/01/16/how-to-deploy-on-amazon-ec2-with-vagrant/)**

### Get more. [Grab our exclusive monthly Scalable Startups](http://www.iheavy.com/signup-scalable-startups-newsletter/). We share tips and special content. Our latest [Why I don't work with recruiters](http://us1.campaign-archive1.com/?u=4d383486fdb4338aa674c9657&id=f8be0734f4&e=[UNIQID])

*Originally published at [Scalable Startups](http://www.iheavy.com/2018/05/11/how-to-setup-an-amazon-ecs-cluster-with-terraform/).*
