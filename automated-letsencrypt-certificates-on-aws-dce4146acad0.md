Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m82[39m, end: [33m94[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m138[39m, end: [33m147[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m74[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m387[39m, end: [33m448[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m474[39m, end: [33m528[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m68[39m, end: [33m80[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m112[39m, end: [33m129[39m }

# Automated LetsEncrypt Certificates on AWS



This post focuses on automating deployment of wildcard Lets Encrypt SSL certificates on Amazon EC2 instances with DNS domain validation utilising Ansible. The post expects basic understanding of Docker, Ansible, AWS IAM and EC2 services. The content of this post describes how to utilise Ansible, Docker and Lets Encrypt technologies to provision valid wildcard LetsEncrypt SSL certificates for an EC2 web server. The automation ensures painless provisioning of a new web server in case of a disaster, as well as keeping the certificates up to date with minimal to no effort. The goal is to end up with an EC2 web server that is able to manage its own certificates using Lets Encrypt with a budget setup.

## Virtual Environment for Ansible

You might think a virtual environment is useless when running Ansible, however during my experience with Ansible versions, modules tend to behave quite differently from version to version. So you might have a different outcome with the same playbook but a different Ansible version. This project is not big enough so that this could be an issue, but it’s good practice to have a stable version of Ansible in a sandbox environment.

Run following commands to create a separate environment for Ansible:

    $ python2.7 -m virtualenv ~/.virtualenvs/ansible
    $ source ~/.virtualenvs/ansible/bin/activate
    $ pip install -r requirements.txt

The requirements.txt file to install.

    ansible==2.4.4.0
    boto==2.48.0
    boto3==1.7.48

## Creating AWS Resources

Here you can utilise manual instance, ELB and policy creation. However, since this is AWS automation blog, I have prepared CloudFormation and Terraform templates to provision the environment for you which are included below. Just to quickly go through all resources required:

### 1. EC2 instance

* Serving as either a web server or SSL control node. Make sure to adjust the size accordingly.

### 2. EC2 instance role

* Instance role must be able to add / change DNS records in your hosted zone.

### 3. Route53 hosted zone

* Hosted zone for the domain you want the SSL certificates to be registered to.

## EC2 Instance

If you are thinking about the simple configuration, that is one web server, adjust the size of the instance according to your estimated capacity requirements.

## EC2 Instance Role

The policy that is required for web server only configuration requires a policy that will ensure that CertBot installed on your EC2 instance is able to list hosted zones as well as change / add records in your hosted zone.

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "route53:GetChange",
                    "route53:ChangeResourceRecordSets",
                ],
                "Resource": [
                    "arn:aws:route53:::hostedzone/R4ND0MZ0N31D",
                    "arn:aws:route53:::change/*"
                ]
            },
            {
                "Effect": "Allow",
                "Action": "route53:ListHostedZones",
                "Resource": "*"
            }
        ]
    }

## Route53 Hosted Zone

This is also required as the automation will add TXT DNS records for domain validation in domains Route53 hosted zone. If you bought a domain from other provider do not worry, you can still either transfer domain to AWS or create a hosted zone in AWS and use the nameserver records from the hosted zone as nameservers in your domain provider hosted zone. This essentially means that requests won’t be resolved by your domain provider but by AWS.

## Automate Everything!

Although this might seem a little excessive, however it’s always good to configure your infrastructure utilising IaaC tools due to infrastructure being documented right in your version control repository. As many of you know it’s easy to loose track of your resources especially on a large platform like AWS. So IaaC tools do not just automate resources, but they come with many other benefits such as immediate documentation, version control or rollbacks.

Below is a Terraform script to build the entire infrastructure for you. I’m using ami-0b91bd72 which is the official Ubuntu18.04 image in eu-west-1 region. The script creates a security group that is opened to the world for on port 80, 443, and 22 for management. It also creates a role and attaches a required policy in order to modify hosted zone with TXT record validation.
> # *In case you already have created a hosted zone prior you can use terraform import HOSTEDZONEID to import it to Terraform state. However, I would not recommend this if you are just testing to see if my script works, since at the end you’ll probably run terraform destroy and if you forget you’ll nuke your entire hosted zone! Therefore, if you’re just testing, use a static variable with your hosted zone id.*

    # import providers
    provider "aws" {
      region                  = "eu-west-1"
      profile                 = "default"
    }

    # create default security group for the demo instance
    resource "aws_security_group" "demo_instance_default_security_group" {
      name   = "demo_instance_security_group"
      vpc_id = "vpc-12345678"
      ingress {
        protocol  = "tcp"
        from_port = "443"
        to_port   = "443"
        cidr_blocks = [
          "0.0.0.0/0",
        ]
      }
      ingress {
        protocol  = "tcp"
        from_port = "80"
        to_port   = "80"
        cidr_blocks = [
          "0.0.0.0/0",
        ]
      }
      ingress {
        protocol  = "tcp"
        from_port = "22"
        to_port   = "22"
        cidr_blocks = [
          "0.0.0.0/0",
        ]
      }
      egress {
        protocol  = "-1"
        from_port = "0"
        to_port   = "0"
        cidr_blocks = [
          "0.0.0.0/0",
        ]
      }
    }

    # create policy for demo instance
    # load ec2 assume role policy
    data "aws_iam_policy_document" "demo_instance_assume_role_policy" {
      statement {
        actions = [
          "sts:AssumeRole",
        ]
        principals {
          type = "Service"
          identifiers = [
            "ec2.amazonaws.com",
          ]
        }
      }
    }

    data "aws_iam_policy_document" "demo_instance_policy" {
      statement {
        actions = [
          "route53:ListHostedZones",
        ]
        resources = [
          "*",
        ]
      }
      statement {
        actions = [
          "route53:GetChange",
          "route53:ChangeResourceRecordSets",
          "route53:ListResourceRecordSets",
        ]
        resources = [
          "arn:aws:route53:::change/*",
          "arn:aws:route53:::hostedzone/${aws_route53_zone.domain_hosted_zone.zone_id}",
        ]
      }
    }

    # create default iam role for demo instance
    resource "aws_iam_role" "demo_instance_default_role" {
      name               = "demo_instance_default_role"
      assume_role_policy = "${data.aws_iam_policy_document.demo_instance_assume_role_policy.json}"
    }

    # create an instance profile for the demo instance
    resource "aws_iam_instance_profile" "demo_instance_attach_iam_role" {
      role = "${aws_iam_role.demo_instance_default_role.name}"
      name = "${aws_iam_role.demo_instance_default_role.name}"
    }

    # deploy policy
    resource "aws_iam_policy" "deploy_route53_policy" {
      policy = "${data.aws_iam_policy_document.demo_instance_policy.json}"
      name   = "policy_to_allow_route53_modifications"
    }
    # attach route53 policy to the role
    resource "aws_iam_role_policy_attachment" "demo_instance_attach_policy" {
      role       = "${aws_iam_role.demo_instance_default_role.name}"
      policy_arn = "${aws_iam_policy.deploy_route53_policy.arn}"
    }

    # create your web server instance
    resource "aws_instance" "demo_web_instance" {
      ami                  = "ami-0b91bd72"
      instance_type        = "t2.micro"
      subnet_id            = "subnet-1234567"
      key_name             = "general_key_pair"
      iam_instance_profile = "${aws_iam_instance_profile.demo_instance_attach_iam_role.id}"
      vpc_security_group_ids = [
        "${aws_security_group.demo_instance_default_security_group.id}",
      ]
    }

    # create elastic ip address
    resource "aws_eip" "demo_web_instance_eip" {
      instance                  = "${aws_instance.demo_web_instance.id}"
      vpc                       = true
      associate_with_private_ip = "${aws_instance.demo_web_instance.private_ip}"
    }

    # create hosted zone for your domain
    resource "aws_route53_zone" "domain_hosted_zone" {
      name = "mykraken.net"
    }

    # create dns record to point to your instance
    resource "aws_route53_record" "demo_dns_record" {
      zone_id = "${aws_route53_zone.domain_hosted_zone.zone_id}"
      name    = "demo.mykraken.net"
      type    = "CNAME"
      ttl     = "300"
      records = [
        "${aws_instance.demo_web_instance.public_dns}",
      ]
    }

    # create apex record for the instance
    resource "aws_route53_record" "demo_dns_record_2" {
      zone_id = "${aws_route53_zone.domain_hosted_zone.zone_id}"
      name    = "mykraken.net"
      type    = "A"
      ttl     = "300"
      records = [
        "${aws_instance.demo_web_instance.public_ip}",
      ]
    }

## Automated SSL Certificates on a Single Web Server

Pseudo Steps With Explanation:

### 1. Install Docker

* Docker is required to avoid a lot of package installations and dependencies.

### 2. Create Docker Containers

* There are two Docker containers created. One is for creating the initial SSL certificates and validating against domain via DNS. The other will serve to renew the certificates when expired.

### 3. Automate Certificate Renewal

* A Cron job is set to run every hour to check whether the certificates need renewing.

### 4. Create Symlinks

* Create symlinks from named Docker volumes to our certificate directory. This is essentially symlinking to synlinks.

### 5. Post Installation

* Add NGINX virtual hosts and other necessary configuration.

Notice the named volumes created for each container certs:/etc/letsencrypt. This is to preserve symlinks that are created when LetsEncrypt creates certs. I wasn’t able to use bind mounts since LetsEncrypt itself links certificates from “archive” directory to “live” directory, and bind mounts break these links. The directory where certificates reside on your host system will always be <docker_volume_dir><volume_tag>_data/live/<apex_domain_name>/ , therefore in this case /var/lib/docker/volumes/certs/_data/live/mykraken.net/

    ---
    - name: add docker apt key from keyserver
      apt_key:
        url: [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg)
        state: present

    - name: add docker repository
      apt_repository: 
        repo: "deb [arch=amd64] [https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) bionic edge"
        state: present
        update_cache: yes

    - name: install apt packages
      apt:
        name: "{{ item }}"
        state: present
      with_items:
        - docker-ce
        - ca-certificates
        - python3-pip

    - name: install pip packages
      pip:
        name: docker-py
        state: present
        executable: pip3

    - name: create a docker container to generate certificates
      docker_container:
        name: certbot-certs
        image: certbot/dns-route53
        command: 
          - "certonly -n --agree-tos"
          - "--email info@katapult.cloud"
          - "--dns-route53"
          - "-d mykraken.net"
          - "-d *.mykraken.net"
          - "--server [https://acme-v02.api.letsencrypt.org/directory](https://acme-v02.api.letsencrypt.org/directory)"
        state: present
        volumes:
          - certs:/etc/letsencrypt
      register: certbot_certs_container_status

    - name: start docker container for creating certificates
      docker_container:
        name: certbot-certs
        state: started
      when: certbot_certs_container_status.changed

    - name: create a docker container to renew certificates
      docker_container:
        name: certbot-renew
        image: certbot/dns-route53
        command: 
          - "renew"
        state: present
        volumes:
          - certs:/etc/letsencrypt

    - name: create a cron entry to start certbot renew
      cron:
        name: "renew certificates"
        minute: "00"
        job: "docker start certbot-renew"

    - name: create directory for certificates 
      file: 
        state: directory 
        path: "/etc/ssl/mykraken.net/"

    - name: create symlinks for certificates
      file:
        state: link
        src: "/var/lib/docker/volumes/certs/_data/live/mykraken.net/{{ item }}"
        dest: "/etc/ssl/mykraken.net/{{ item }}"
      with_items:
        - fullchain.pem
        - privkey.pem
        - cert.pem
        - chain.pem
    ...

And here they are!

    ubuntu@ip-10-0-1-81:~$ ls -l /etc/ssl/mykraken.net/
    total 8
    lrwxrwxrwx 1 root root 67 Jul 5 18:26 fullchain.pem -> /var/lib/docker/volumes/certs/_data/live/mykraken.net/fullchain.pem
    lrwxrwxrwx 1 root root 65 Jul 5 18:26 privkey.pem -> /var/lib/docker/volumes/certs/_data/live/mykraken.net/privkey.pem

    ubuntu@ip-10-0-1-81:~$ sudo cat /etc/ssl/mykraken.net/fullchain.pem 
    -----BEGIN CERTIFICATE-----
    MIIGEzCCBPugAwIBAgISA7E40m5KyRZ78tH7HkZvmltHMA0GCSqGSIb3DQEBCwUA
    MEoxCzAJBgNVBAYTAlVTMRYwFAYDVQQKEw1MZXQncyBFbmNyeXB0MSMwIQYDVQQD
    ...

After web server configuration SSL is fully working for apex domain mykraken.net as well as for wildcard domain demo.mykraken.net.

![](https://cdn-images-1.medium.com/max/3484/0*pAYc42qpm6tBmneE.png)
