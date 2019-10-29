Unknown markup type 10 { type: [33m10[39m, start: [33m36[39m, end: [33m39[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m43[39m, end: [33m51[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m79[39m, end: [33m80[39m }

# Terraform by examples. Part 1

First of all I should say that I’m not familiar with Terraform and it’s just my notes about this tool. If you’ve never heard / used this tool it’s better to visit https://www.terraform.io/intro/getting-started/install.html

I decided to learn terraform “by examples”, so I tried to implement simple service (I chose zabbix) using only this tool. Deployment will consist of four parts, each in separate network with it’s own security rules.

![Network diagram](https://cdn-images-1.medium.com/max/2000/1*VD8oHWXZ5nJJCgsL5IG5Rw.png)*Network diagram*

## **Setup environment.**

Download terraform from [https://www.terraform.io/downloads.html](https://www.terraform.io/downloads.html) or use your package manager.

    $ brew install terraform terraform-inventory

Create aws user using AMI interface and download your credentials AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY.

## Init && download provider plugin.

Create directory where we’ll store our code base infrastructure definitions and create file **main.tf**
> When invoking any command that loads the Terraform configuration, Terraform loads all configuration files within the directory specified in alphabetical order.
> The files loaded must end in either .tf or .tf.json to specify the format that is in use. Otherwise, the files are ignored.

    provider "aws" {
        region = "${var.aws_region}"
        shared_credentials_file = "/Users/dhelios/.aws/credentials"
        profile = "TFtests"
    }

file **vars.tf** will be used for variables definitions

    # region / regions where we deploy our infrastructure
    variable aws_region { default = "us-west-2" }
    variable default_AZ { default = "us-west-2a" }
    variable default_keyname { default = "TF-tests-key.rsa" }

After that call **terraform init **to download provider specific plugins.

## Network definitions.

In file networks.tf I describe all network patterns and add additional network related variables in vars.tf file.

**networks.tf:**

    # Create separate virtual private cloud for project.
    resource "aws_vpc" "default" {
        cidr_block = "${var.cidr_block_main}"
        enable_dns_hostnames = true

    tags {
            Name = "${var.vpc_name}"
        }
    }

    # BEGIN: Network Section

    # network to place database instances of our service
    resource "aws_subnet" "subnet_for_db" {
        vpc_id  = "${aws_vpc.default.id}"
        cidr_block = "${var.cidr_block_for_db}"
        availability_zone = "${var.default_AZ}"
        map_public_ip_on_launch = true

    tags {
            Name = "Subnet_For_Databases"
        }
    }

    # Network to place zabbix server and zabbix web backends
    resource "aws_subnet" "subnet_for_app" {
        vpc_id = "${aws_vpc.default.id}"
        cidr_block = "${var.cidr_block_for_app}"
        availability_zone = "${var.default_AZ}"
        map_public_ip_on_launch = true

    tags {
            Name = "Subnet_For_Applications"
        }
    }

    # Network for ELB
    resource "aws_subnet" "subnet_for_lb" {
        vpc_id = "${aws_vpc.default.id}"
        cidr_block = "${var.cidr_block_for_lb}"
        availability_zone = "${var.default_AZ}"
        map_public_ip_on_launch = true

    tags {
            Name = "Subnet_For_LoadBalancers"
        }
    }

    # Network for bastion host
    resource "aws_subnet" "subnet_for_bhost" {
        vpc_id = "${aws_vpc.default.id}"
        cidr_block = "${var.cidr_block_for_bhost}"
        availability_zone = "${var.default_AZ}"
        map_public_ip_on_launch = true

    tags {
            Name = "Subnet_For_BastionHost"
        }
    }

    # Enable internet access
    resource "aws_internet_gateway" "gw" {
        vpc_id = "${aws_vpc.default.id}"

    tags {
            Name = "${var.vpc_name}_ig"
        }
    }

    # specify default router
    resource "aws_route_table" "r" {
      vpc_id = "${aws_vpc.default.id}"

    route {
        cidr_block = "0.0.0.0/0"
        gateway_id = "${aws_internet_gateway.gw.id}"
      }

    tags {
        Name = "aws_route_table"
      }
    }

    ##
    ## Add routes to networks
    ## (first three addresses in every 
    ## network are reserved for aws services + one for subnet)

    resource "aws_route_table_association" "rt_app" {
      subnet_id      = "${aws_subnet.subnet_for_app.id}"
      route_table_id = "${aws_route_table.r.id}"
    }

    resource "aws_route_table_association" "rt_db" {
      subnet_id      = "${aws_subnet.subnet_for_db.id}"
      route_table_id = "${aws_route_table.r.id}"
    }

    resource "aws_route_table_association" "rt_lb" {
      subnet_id      = "${aws_subnet.subnet_for_lb.id}"
      route_table_id = "${aws_route_table.r.id}"
    }

    resource "aws_route_table_association" "rt_bhost" {
        subnet_id = "${aws_subnet.subnet_for_bhost.id}"
        route_table_id = "${aws_route_table.r.id}"
    }
    # END: Network Section

All variables used in **networks.tf** must be defined in **vars.tf**

    variable vpc_name { default = "PocVpc" }
    variable cidr_block_main { default = "192.168.188.0/24" }
    # ---
    # Network for databases
    # HostMin:   192.168.188.5  (first four is reserved by amazon)
    # HostMax:   192.168.188.14
    variable cidr_block_for_db { default = "192.168.188.0/28" }
    # ---
    # Network for app services
    # HostMin:   192.168.188.21 (first four is reserved by amazon)
    # HostMax:   192.168.188.30
    variable cidr_block_for_app { default = "192.168.188.16/28" }
    # ---
    # Network for LB
    # HostMin:   192.168.188.37 (first four is reserved by amazon)
    # HostMax:   192.168.188.46
    variable cidr_block_for_lb { default = "192.168.188.32/28" }
    # ---
    # Network for Bastion Host
    # HostMin:   192.168.188.53 (first four is reserved by amazon)
    # HostMax:   192.168.188.62
    variable cidr_block_for_bhost { default = "192.168.188.48/28" }
    # ---
    # Constanstant CIDR blocks
    variable cidr_block_internet { default = "0.0.0.0/0" }

## Security groups

All security policies will be described in **security.tf. **Each** **security group has a self explained descriptions.

    # BEGIN: Sec Groups
    resource "aws_security_group" "sg_allow_internet" {
        name = "sg_allow_internet"
        description = "allow all outcomming traffic"
        vpc_id = "${aws_vpc.default.id}"

    # allow internet access, but block all incoming traffic
        egress {
            from_port = 0
            to_port = 0
            protocol = "-1"
            cidr_blocks = ["${var.cidr_block_internet}"]
        }
    }

    resource "aws_security_group" "sg_bastion" {
        name = "sg_bastion"
        description = "allow traffic on bastion host"
        vpc_id = "${aws_vpc.default.id}"

    # allow sshd connection
        ingress {
            from_port = 22
            to_port = 22
            protocol = "tcp"
            cidr_blocks = ["${var.cidr_block_internet}"]
        }
    }

    resource "aws_security_group" "sg_admin" {
        name = "sg_allow_bhost_traffic"
        description = "allow all traffic from bastion host"
        vpc_id = "${aws_vpc.default.id}"

    ingress {
            from_port = 0
            to_port = 0
            protocol = "-1"
            cidr_blocks = ["${var.cidr_block_for_bhost}"]
        }
    }

    resource "aws_security_group" "sg_lb2app" {
        name = "sg_allow_lb_to_app"
        description = "allow traffic from load balancer and to load balancer"
        vpc_id = "${aws_vpc.default.id}"

    ingress {
            from_port = "${var.web_http_port}"
            to_port = "${var.web_http_port}"
            protocol = "tcp"
            cidr_blocks = ["${var.cidr_block_for_lb}"]
        }

    ingress {
            from_port = "${var.web_https_port}"
            to_port = "${var.web_https_port}"
            protocol = "tcp"
            cidr_blocks = ["${var.cidr_block_for_lb}"]
        }
    }

    resource "aws_security_group" "elb" {
        name = "sg_elb"
        description = "elb security group"
        vpc_id = "${aws_vpc.default.id}"

    ingress {
            from_port = "${var.elb_listen_http_port}"
            to_port = "${var.elb_listen_http_port}"
            protocol = "tcp"
            cidr_blocks = ["${var.cidr_block_internet}"]
        }

    egress {
            from_port = 0
            to_port = 0
            protocol = "-1"
            cidr_blocks = ["${var.cidr_block_internet}"]
        }
    }

    resource "aws_security_group" "sg_app2db" {
        name = "sg_allow_app_to_db"
        description = "allow traffic from app hosts to databases"
        vpc_id = "${aws_vpc.default.id}"

    ingress {
            from_port = "${var.db_port}"
            to_port = "${var.db_port}"
            protocol = "tcp"
            cidr_blocks = ["${var.cidr_block_for_app}"]
        }
    }

    resource "aws_security_group" "sg_web2app" {
        name = "sg_allow_web_to_app"
        description = "allow traffic from app hosts to databases"
        vpc_id = "${aws_vpc.default.id}"

    ingress {
            from_port = "${var.app_port}"
            to_port = "${var.app_port}"
            protocol = "tcp"
            cidr_blocks = ["${var.cidr_block_for_app}"]
        }
    }

    # END: Sec Groups

## Discover AMIs

Virtual Machine images in aws have unique identifies depends on region. To simplify deployments and do not track id changes let’s describe automatic image selection.

    data "aws_ami" "ubuntu" {
      most_recent = true
      filter {
        name   = "name"
        values = ["ubuntu/images/hvm-ssd/ubuntu-xenial-16.04-amd64-server-*"]
      }
      filter {
        name   = "virtualization-type"
        values = ["hvm"]
      }
      owners = ["099720109477"] # Canonical
    }

    data "aws_ami" "centos" {
      most_recent = true
      filter {
        name = "name"
        values = ["CentOS Linux 7 x86_64*"]
      }
      filter {
        name = "virtualization-type"
        values = ["hvm"]
      }
      filter {
        name = "product-code"
        values = ["aw0evgkw8e5c1q413zgy5pjce"]
      }
      owners = ["679593333241"] # aws
    }

    data "aws_ami" "aws_linux" {
      most_recent = true
      filter {
        name = "name"
        values = ["amzn-ami-hvm-*"]
      }
      filter {
        name = "virtualization-type"
        values = ["hvm"]
      }
      owners = ["137112412989"] # aws
    }

## Deploy instances

Database and application (zabbix-server) instances will be EC2 t2.micro virtual machines and web-server I’ll implement throught autoscalling group.
> An *Auto Scaling group* contains a collection of EC2 instances that share similar characteristics and are treated as a logical grouping for the purposes of instance scaling and management
> An Auto Scaling group starts by launching enough EC2 instances to meet its desired capacity. The Auto Scaling group maintains this number of instances by performing periodic health checks on the instances in the group

    # BEGIN: Instances
    resource "aws_instance" "bhost" {
        ami = "${data.aws_ami.centos.id}"
        instance_type = "${var.bhost_inst_type}"
        subnet_id = "${aws_subnet.subnet_for_bhost.id}"
        key_name = "${var.default_keyname}"
        vpc_security_group_ids = [
            "${aws_security_group.sg_allow_internet.id}", 
            "${aws_security_group.sg_bastion.id}"
        ]

    monitoring = false

    count = "${var.bhost_inst_count}"

    tags {
            Name = "Bastion Host"
        }
    }

    resource "aws_instance" "app" {
        ami = "${data.aws_ami.centos.id}"
        instance_type = "${var.app_inst_type}"
        subnet_id = "${aws_subnet.subnet_for_app.id}"
        key_name = "${var.default_keyname}"
        vpc_security_group_ids = [
            "${aws_security_group.sg_allow_internet.id}", 
            "${aws_security_group.sg_bastion.id}",
            "${aws_security_group.sg_web2app.id}" 
        ]

    monitoring = false

    count = "${var.app_inst_count}"

    user_data = <<-EOF
                    #!/bin/bash
                    yum install -y [http://repo.zabbix.com/zabbix/3.4/rhel/7/x86_64/zabbix-release-3.4-1.el7.centos.noarch.rpm](http://repo.zabbix.com/zabbix/3.4/rhel/7/x86_64/zabbix-release-3.4-1.el7.centos.noarch.rpm)
                    yum install -y zabbix-server-mysql.x86_64 policycoreutils.x86_64 policycoreutils-devel
                    setenforce 0
                    cat > /etc/zabbix/zabbix_server.conf<<'_END'
                    LogFile=/var/log/zabbix/zabbix_server.log
                    LogFileSize=0
                    PidFile=/var/run/zabbix/zabbix_server.pid
                    SocketDir=/var/run/zabbix
                    DBName=zabbix
                    DBUser=zabbix
                    DBHost=${aws_instance.db.private_ip}
                    DBPassword=${var.db_password}
                    SNMPTrapperFile=/var/log/snmptrap/snmptrap.log
                    Timeout=4
                    AlertScriptsPath=/usr/lib/zabbix/alertscripts
                    ExternalScripts=/usr/lib/zabbix/externalscripts
                    LogSlowQueries=3000
                    _END
                    cat > /tmp/zabbix-policy.te <<'_END'
                    module zabbix-policy 1.0;
                    require {
                        type mysqld_port_t;
                        type zabbix_var_run_t;
                        type zabbix_t;
                        class sock_file create;
                        class tcp_socket name_connect;
                        class process setrlimit;
                        class unix_stream_socket connectto;
                    }
                    #============= zabbix_t ==============
                    allow zabbix_t mysqld_port_t:tcp_socket name_connect;
                    allow zabbix_t self:process setrlimit;
                    
                    allow zabbix_t self:unix_stream_socket connectto;
                    allow zabbix_t zabbix_var_run_t:sock_file create;
                    _END
                    checkmodule -M -m -o /tmp/zabbix-policy.mod /tmp/zabbix-policy.te
                    semodule_package -o /tmp/zabbix-policy.pp -m /tmp/zabbix-policy.mod
                    semodule -i /tmp/zabbix-policy.pp
                    systemctl start zabbix-server.service
                    systemctl enable zabbix-server.service
                    EOF

    tags {
            Name = "application"
        }

    }

Terraform does not provide loop statement like this

    for (i := 0; i < 10; i++) {
      resource "aws_instance" "web" {
        instance_type = "t2.medium"
        tags {
            Name = "web" + i
        }
      }
    }

But provides similar functionality through **count** attribute.

    resource "aws_instance" "web" {
        instance_type = "t2.medium"
        count = "10"
        tags {
            Name = "web + ${count.index}"
        }
      }

Another usefull thing is a **user_data** attribute. You can specify script payload or cloud daemon instruction. Data can be inline template (example above) or file template (need to install dependencies template module).

## Autoscalling groups

autoscalling group declaration is similar to **aws_instance**. We described desired configuration in **aws_launch_configuration **resource and added additional scalling options in **aws_autoscaling_group** resource. After that we defined load balancer which would be track the state of our service.

    data "template_file" "app_payload" {
        template = "${file("user_data_app.tpl")}"
        vars = {
            db_private_addr = "${aws_instance.db.private_ip}"
            db_password = "${var.db_password}"
            db_port = "${var.db_port}"
            zbx_private_addr = "${aws_instance.app.private_ip}"
        }
    }

    resource "aws_launch_configuration" "web_lc" {
        image_id = "${data.aws_ami.centos.id}"
        instance_type = "${var.web_inst_type}"
        security_groups  = [
                "${aws_security_group.sg_admin.id}",
                "${aws_security_group.sg_allow_internet.id}",
                "${aws_security_group.sg_lb2app.id}"
        ]

    key_name = "${var.default_keyname}"

    user_data = "${data.template_file.app_payload.rendered}"
        enable_monitoring = false

    lifecycle {
            create_before_destroy = true
        }
    }

    resource "aws_autoscaling_group" "web_asg" {
        launch_configuration = "${aws_launch_configuration.web_lc.id}"
        min_size = "${var.web_inst_min_count}"
        max_size = "${var.web_inst_max_count}"

    vpc_zone_identifier = ["${aws_subnet.subnet_for_app.id}"]
        tag {
            key = "Name"
            value = "web-asg"
            propagate_at_launch = true
        }

    load_balancers = ["${aws_elb.frontend_lb.name}"]
        health_check_type = "ELB"

    }

    resource "aws_elb" "frontend_lb" {
        name = "frontendlb"
        security_groups = [ 
            "${aws_security_group.elb.id}"]
        subnets = ["${aws_subnet.subnet_for_lb.id}"]

    health_check {
            healthy_threshold = 2
            unhealthy_threshold = 2
            timeout = 3
            interval = 30
            target = "HTTP:${var.web_http_port}/zabbix/"
        }

    listener {
            lb_port = "${var.elb_listen_http_port}"
            lb_protocol = "http"
            instance_port = "${var.web_http_port}"
            instance_protocol = "http"
        }

    }

    # END: Instances
> **NOTE: **Inline templates must escape their interpolations (as seen by the double *$* above). Unescaped interpolations will be processed *before* the template.

As you see I didn’t use inline template like described before. Instead of that I created shell script template and rendered it every time when I launched new instance.

**Updated vars.tf file:**

    # instancies
    variable bhost_inst_type  { default = "t2.nano" }
    variable bhost_inst_count { default = 1 }

    variable app_inst_type { default = "t2.nano" }
    variable app_inst_count { default = 1 }
    variable app_port { default = 10051 }

    variable web_inst_type { default = "t2.nano" }
    variable web_inst_min_count { default = 1 }
    variable web_inst_max_count { default = 2 }
    variable web_http_port { default = 80 }
    variable web_https_port { default = 443 }

    variable db_inst_type {default = "t2.nano" }
    variable db_inst_count { default = 1 }
    variable mysql_nofile { default = 10000 }
    variable db_password { default = "mypasswd1" }
    variable db_port { default = 3306 }

    variable elb_listen_http_port { default = 80 }

## Plan && Apply

Run terraform plan and see what’s resource it will create, also it shows static parameters which we specify in resources.

![](https://cdn-images-1.medium.com/max/2084/1*KAigtt1fR7uMPpI8DNTdqA.png)

Try to login and check if our payload in **user_data **executed successfully and setup our environment as needed.

![](https://cdn-images-1.medium.com/max/5112/1*r2IkkruurCS8Pg_zwXw5Nw.png)

![](https://cdn-images-1.medium.com/max/5108/1*4LAlnH8AETkhPf4Ej5elkg.png)

Complete Ex: [https://github.com/d-helios/examples/tree/master/terraform/TF-zabbix.simple](https://github.com/d-helios/examples/tree/master/terraform/TF-zabbix.simple)

In [next article](https://medium.com/@dhelios/terraform-by-examples-part-2-f50f35e5a8d4) I’ll show you how to do the same things, but with less efforts. We will consider using s3 as a backend to storing your tfstate files and look at the modules available at [https://registry.terraform.io/](https://registry.terraform.io/).
