Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m24[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m35[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m74[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m55[39m, end: [33m80[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m26[39m, end: [33m40[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m37[39m, end: [33m41[39m }

# Automated Website deployment using Terraform

Clients connect to Google, Amazon, Flipkart more often because it is fast. Headquartered in any part of the world, how come at any place, the webpage shows up without any delay? The contents of the web page, even the images and all the graphics show up in less than a second.

## How it happens?

All of this is possible with the advent of **Cloud Technology**. In this article I will be showing how to create a Terraform code from scratch for a website deployment.

Using Terraform we will do multiple things in a proper sequence:

* Create an AWS Instance — **EC2**

* Install required dependencies, modules, softwares

* Create an **EBS** for persistent storage

* Attach, Format and Mount it in a folder in the instance

* Clone the code sent by developer on** GitHub **in the folder

* Create an **S3 Bucket** for storage of static data

* This will be sent to all the edge locations using **CloudFront**

* Finally loading the webpage on your favourite **browser** automatically

![**Terraform**](https://cdn-images-1.medium.com/max/2000/1*GSSjFIC3bH49YvKmG09dgQ.png)***Terraform***

**Terraform **uses **Infrastructure as Code** to provision and manage any cloud, infrastructure, or service. This means that we can write the code in 1 single unified language- HCL(HashiCorp Configuration Language) that will work with any kind of Cloud: Public or Private.

**NOTE: I am using Terraform on my local system running on Windows 10.**

For this particular setup you should have the following things installed on your machine:

1. **Git**

1. **AWS Command Line Interface**

1. **Terraform**

### Let’s start by building the code.

    **provider "aws" {
      profile = "daksh"
      region  = "ap-south-1"
    }**

**provider** is a keyword used to tell Terraform which cloud platform we are using. I have specified **aws, **now this will help terraform to download the plugins required for AWS cloud platform.

**profile** is given so that you need not login through the code. The cloud engineer just provides the profile name and the terraform code will pick up credentials from the local system as shown in the figure:

![**How to Configure a Profile**](https://cdn-images-1.medium.com/max/2000/1*VL7c339jAoeh9_tZ-dNT_Q.png)***How to Configure a Profile***

    **aws configure --profile**  ***profilename***

Using this command you can setup your profile.

    **resource "aws_security_group" "allow_traffic" {
      name        = "allow_traffic"
      description = "Allow TLS inbound traffic"
      vpc_id      = "vpc-59766a31"**

    **ingress {
        description = "http"
        from_port   = 80
        to_port     = 80
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
      }**

    **ingress {
        description = "ssh"
        from_port   = 22
        to_port     = 22
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
      }**

    **ingress {
        description = "ping"
        from_port   = -1
        to_port     = -1
        protocol    = "icmp"
        cidr_blocks = ["0.0.0.0/0"]
      }**

    **egress {
        from_port   = 0
        to_port     = 0
        protocol    = "-1"
        cidr_blocks = ["0.0.0.0/0"]
      }**

    **tags = {
        Name = "allow_traffic"
      }
    }**

**VPC ID **can be found in your AWS account details.

**Ingress **means the traffic that is coming in to our website. We need to specify this, keeping in mind what ports we want to keep open. I have kept open 3 ports: ssh, http, icmp.

* **SSH** for testing purpose so that remotely we can connect to the AWS EC2 instance.

* **HTTP **so that traffic can hit on the website.

* **ICMP **to check ping connectivity.

**Egress **has been set to all ports so that outbound traffic originating from within a network can go.

    **resource "aws_instance" "webserver" {
      ami                = "ami-0447a12f28fddb066"
      instance_type      = "t2.micro"
      security_groups    = [ "allow_traffic" ]
      key_name           = "key1"
     
      connection {
        type        = "ssh"
        user        = "ec2-user"
        private_key = file("C:/Users/Daksh/Downloads/key1.pem")
        host        = aws_instance.webserver.public_ip
      }**

    **provisioner "remote-exec" {
        inline = [
          "sudo yum install httpd php git -y",
          "sudo systemctl restart httpd",
          "sudo systemctl enable httpd"
        ]
      }
      tags = {
        Name = "webserver"
      }
    }**

Now as a next step we have to download the required tools, so that our webpage can be deployed, managed and can be viewed by the client.

So first we create an EC2 instance and specify all the required details.

Then we create a **connection **so that we can do SSH on the the instance, to install the tools.

Then using **Provisioner **we go to the remote system and using the **inline **method run multiple commands that are compatible with the Linux flavour I am using.

**(If you are using some other Operating System, then you must know the equivalent commands to do the same.)**

    **resource "aws_ebs_volume" "web_vol" {
     availability_zone = aws_instance.webserver.availability_zone
     size = 1
     tags = {
            Name = "web_vol"
     }
    }**

Then create another EBS storage, to make the data persistent. Make sure that you make the EBS in the same availability zone in which the instance is launched. that is the reason I have used a better approach to tackle it by using the internal keywords that can be found on the [Terraform docs.](https://www.terraform.io/docs/providers/aws/index.html)

    **resource "aws_volume_attachment" "web_vol" {**

    **depends_on = [
        aws_ebs_volume.web_vol,
      ]
     device_name  = "/dev/xvdf"
     volume_id    = aws_ebs_volume.web_vol.id
     instance_id  = aws_instance.webserver.id
     force_detach = true**

    **connection {
        type        = "ssh"
        user        = "ec2-user"
        private_key = file("C:/Users/Daksh/Downloads/key1.pem")
        host        = aws_instance.webserver.public_ip
      }**

    **provisioner "remote-exec" {
        inline = [
          "sudo mkfs.ext4 /dev/xvdf",
          "sudo mount /dev/xvdf /var/www/html",
          "sudo rm -rf /var/www/html/*",
          "sudo git clone [https://github.com/Dakshjain1/php-cloud.git](https://github.com/Dakshjain1/php-cloud.git) /var/www/html/"**

    **]
      }
    }**

Now to use a new block storage, first we need to **format** it, then **mount** it.

Also it is not mandatory to create a Partition in a Storage device, without creating a partition also we can format. So, here we are going to use a similar kind of approach.

Again using provisioner we use the inline method to run the commands:

* **sudo mkfs.ext4 /dev/xvdf** : create partition

* **sudo mount /dev/xvdf /var/www/html **: mount the partition

* **sudo rm -rf /var/www/html/*** : empty the folder because git clone works only in empty directory

* **sudo git clone [https://github.com/Dakshjain1/php-cloud.git](https://github.com/Dakshjain1/php-cloud.git) /var/www/html/ **: git clone to get the webpage files

    **resource "aws_s3_bucket" "s3bucket" {
      bucket = "123mywebbucket321"
      acl    = "public-read"
      region = "ap-south-1"**

    **tags = {
        Name = "123mywebbucket321"
      }
    }**

Next we create the **S3 bucket**. S3 bucket is used to store the static data like images, videos and other graphics. This is required so that anywhere from the world, if the website is opened, the static contents also get loaded without any delay or latency.

    **resource "aws_s3_bucket_object" "image-upload" {**

    **depends_on = [
        aws_s3_bucket.s3bucket,
      ]
        bucket  = aws_s3_bucket.s3bucket.bucket
        key     = "flower.jpg"
        source  = "C:/Users/Daksh/Desktop/CLOUD/task1/pic.jpg"
        acl     = "public-read"
    }**

    **output "bucketid" {
      value = aws_s3_bucket.s3bucket.bucket
    }**

Now I have to upload an object into the S3 bucket created.

First I have used a keyword **depends_on. **This is used because terraform code doesn’t work in a sequential manner. So to prevent a condition where the bucket is not yet created but object is trying to get uploaded we are doing this.

Like this the object will start to upload only after the bucket creation is done.

**key **is the name of the file that will show in the bucket and **source **is the path of the file I want to upload in the bucket. This file/object can come from anywhere Google, local system or somewhere else.

    **variable "oid" {
      type    = string
      default = "S3-"
    }**

    **locals {
      s3_origin_id = "${var.oid}${aws_s3_bucket.s3bucket.id}"
    }**

    **resource "aws_cloudfront_distribution" "s3_distribution" {
      depends_on = [
        aws_s3_bucket_object.image-upload,
      ]
      origin {
        domain_name = "${aws_s3_bucket.s3bucket.bucket_regional_domain_name}"
        origin_id   = "${local.s3_origin_id}"
      }**

    **  enabled = true**

    **  default_cache_behavior {
        allowed_methods  = ["DELETE", "GET", "HEAD", "OPTIONS", "PATCH", "POST", "PUT"]
        cached_methods   = ["GET", "HEAD"]
        target_origin_id = "${local.s3_origin_id}"**

    **forwarded_values {
          query_string = false
          cookies {
            forward = "none"
          }
        }**

    **viewer_protocol_policy = "allow-all"
        min_ttl            = 0
        default_ttl        = 3600
        max_ttl            = 86400
      }**

    **restrictions {
        geo_restriction {
          restriction_type = "none"
        }
      }**

    **viewer_certificate {
        cloudfront_default_certificate = true
      }**

    **connection {
        type        = "ssh"
        user        = "ec2-user"
        private_key = file("C:/Users/Daksh jain/Downloads/CLOUD/key1.pem")
        host        = aws_instance.webserver.public_ip
      }**

    **provisioner "remote-exec" {
        inline = [
          "sudo su <<END",
          "echo \"<img src='[http://${aws_cloudfront_distribution.s3_distribution.domain_name}/${aws_s3_bucket_object.image-upload.key}'](http://${aws_cloudfront_distribution.s3_distribution.domain_name}/${aws_s3_bucket_object.image-upload.key}') height='200' width='200'>\" >> /var/www/html/index.php",
          "END",
        ]
      }
    }**

The main aim of creating an S3 Bucket is that there is no latency. So this can be achieved in AWS using **Edge Locations. **These are small data centres that are created by AWS all over the world. To use this mechanism, we use **CloudFront **service of AWS.

I have created a variable where I have set a value to **“S3-”.**

This is because when we check the ID using the command **aws_s3_bucket.s3bucket.id **it gives only the ID but from GUI we know that the ID provided starts with **S3-**

![**Look at how Origin ID is provided**](https://cdn-images-1.medium.com/max/3840/1*3M4OfHl3h9SV6_h-Ovi8mg.png)***Look at how Origin ID is provided***

Then we create the rest of the code by providing the **domain name & origin ID.**

* Then we set the **default_cache_behavior **which is a required block of code.

* Then we set the **viewer_protocol_policy **specifying the default and maximum TTL.

* Then we can set any **restrictions **if required (whitelist & blacklist).

* Then we have to set the **viewer_certificate **as true.

Now finally one last thing we have to do. From this cloudfront the URL that is provided to the bucket object, we have to put in the code send to us by the developer so that the client can see it.

For this again I have made an SSH connection using **Connection **& **Provisioner.**
> **To write the code into an already existing file we have to be the root user, and right now we are ec2-user by default.**

SOLUTION TO THIS PROBLEM:

* **We can login as the root user.**

This is possible from GUI or CLI but from Terraform code this is not possible directly.

* **Switch user to root on the fly.**

This is a good solution and technically that is what I have done.

When we use this command: **sudo su — root , **a child shell is created. If this is written on the local system, this command works seamlessly.

But if this is written from a remote system, here Terraform, it fails to get the child shell.

![**Terraform hangs up but CloudFront gets deployed**](https://cdn-images-1.medium.com/max/3840/1*0S9auKUyXGwWxWiDq7m_sw.gif)***Terraform hangs up but CloudFront gets deployed***

* **So the final correct solution is use sudo to run a shell and use a [heredoc](https://en.wikipedia.org/wiki/Here_document) to feed it commands.**

A **Heredoc** is a file literal or **input stream literal**: it is a section of a source code file that is treated as if it were a separate file.

    **# sudo su <<END
    > date
    > cal
    > END**

    ***Its Output will be:
    - - - - - - - - - -***

    **Mon Jun 15 00:51:14 IST 2020
          June 2020
    Su Mo Tu We Th Fr Sa
        1  2  3  4  5  6
     7  8  9 10 11 12 13
    14 15 16 17 18 19 20
    21 22 23 24 25 26 27
    28 29 30**

Finally this is a Bonus part I have done using the **local-exec **of the provisioner.

    **resource "null_resource" "openwebsite"  {**

    **depends_on = [
        aws_cloudfront_distribution.s3_distribution,   aws_volume_attachment.web_vol
      ]**

    **provisioner "local-exec" {
         command = "start chrome [http://${aws_instance.webserver.public_ip}/](http://${aws_instance.webserver.public_ip}/)"
       }
    }**

I am running the chrome command on my local machine to launch the EC2 instance using the IP of the instance.

(PS. To launch chrome from Command Prompt on Windows, you have to set the Environment Variable PATH for Chrome Application.)

### OUTPUT:

![**Final Ouput**](https://cdn-images-1.medium.com/max/3840/1*9ZPeGk3zJqYd5WC3oHMT4w.gif)***Final Ouput***

You can find the code on my [**GitHub](https://github.com/Dakshjain1/Terraform.git).**

## That’s all folks!!

### For any queries, corrections or, suggestions you can always c**onnect with me on my [LinkedIn](https://www.linkedin.com/in/dakshjain09/).**

## Worked in collaboration with[ Ashish Kumar](https://www.linkedin.com/in/ashishkumar99/).
