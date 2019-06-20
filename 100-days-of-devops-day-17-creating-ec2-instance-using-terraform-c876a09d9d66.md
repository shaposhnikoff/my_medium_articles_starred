
# 100 Days of DevOps — Day 17- Creating EC2 Instance using Terraform

Welcome to Day 17 of 100 Days of DevOps, Let continue our journey, so far I have discussed terraform fundamentals and building VPC. Let continue our Journey and build our two EC2 instances in Public Subnet using terraform
[**100 Days of DevOps — Day 15- Introduction to Terraform**
*Welcome to Day 15 of 100 Days of DevOps, Let continue our journey and focus on Automation especially on Infrastructure…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-15-introduction-to-terraform-7a168dec8d38)
[**100 Days of DevOps — Day 16- Building VPC using Terraform**
*Welcome to Day 16 of 100 Days of DevOps, Let continue our journey, yesterday I discussed terraform, today let’s build…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-16-building-vpc-using-terraform-7c507ce07413)

*In order to deploy EC2 instance we need a bunch of resources*

* *AMI*

* *Key Pair*

* *EBS Volumes Creation*

* *User data*

*The first step in deploying EC2 instance is choosing correct AMI and in terraform, there are various ways to do that*

* *We can hardcode the value of AMI*

* *We can use data resource(similar to what we used for Availability Zone in VPC section) to query and filter AWS and get the latest AMI based on the region, as the AMI id is different in a different region.*

<iframe src="https://medium.com/media/d721dfe60b20cd663dbe56c771610deb" frameborder=0></iframe>

*NOTE: Use of data resource is not ideal and each and every used case, eg: In the case of Production we might want to use a specific version of CentOS.*

* *The above code will help us to get the latest Centos AMI, the code is self-explanatory but one important parameter we used is owners*

* [*owners](https://www.terraform.io/docs/providers/aws/d/ami.html#owners) - (Optional) Limit search to specific AMI owners. Valid items are the numeric account ID, amazon, or self.*

* [*most_recent](https://www.terraform.io/docs/providers/aws/d/ami.html#most_recent) - (Optional) If more than one result is returned, use the most recent AMI.This is to get the latest Centos AMI as per our use case.*

* *For more info*
[**AWS: aws_ami - Terraform by HashiCorp**
*Get information on a Amazon Machine Image (AMI).*www.terraform.io](https://www.terraform.io/docs/providers/aws/d/ami.html)

* *Other ways to find the AMI ID*

    *Go to [https://us-west-2.console.aws.amazon.com/ec2](https://us-west-2.console.aws.amazon.com/ec2) --> Instances --> Launch Instances --> Search for centos *

![](https://cdn-images-1.medium.com/max/5720/1*DAcqAiBGWYiDCEF4hJgz3A.png)

* *Same thing you can do using AWS CLI*

    *aws --region us-west-2 ec2 describe-images --owners aws-marketplace --filters Name=product-code,Values=aw0evgkw8e5c1q413zgy5pjce*

<iframe src="https://medium.com/media/41e4b97a9dbebff6549aa5e9b06ad8c7" frameborder=0></iframe>

* *If you look at the Description field*

    *"Description": "CentOS Linux 7 x86_64 HVM EBS 1708_11.01",*

*and then check the Terraform code in of the filter we use **“CentOS Linux 7 x86_64 HVM EBS *” **and that is one of the reasons of using that*

    *filter {
      name   = "name"
      values = [**"CentOS Linux 7 x86_64 HVM EBS *"]**
    }*
[**Cloud/AWS - CentOS Wiki**
*We welcome all contributions for guides and howtos, so get your favorite tools mentioned here by joining the CentOS…*wiki.centos.org](https://wiki.centos.org/Cloud/AWS)

* *As we are able to figure out the AMI part, the next step is to create and use the key pair*

* *Either we can hardcode the value of key pair or generate a new key via command line and then refer to this file*

    *$ ssh-keygen*

    *Generating public/private rsa key pair.*

    *Enter file in which to save the key (/Users/plakhera/.ssh/id_rsa): /tmp/id_rsa*

    *Enter passphrase (empty for no passphrase):*

    *Enter same passphrase again:*

    *Your identification has been saved in /tmp/id_rsa.*

    *Your public key has been saved in /tmp/id_rsa.pub.*

    *The key fingerprint is:*

    *XXXXXXXXXXXXX*

    *The key's randomart image is:*

    *+---[RSA 2048]----+*

    *|                 |*

    *|                 |*

    *|                 |*

    *|   . . .   .   . |*

    *|. = . +.S .o. o .|*

    *| *.= +. .oo.oo + |*

    *|o =+=  o o+o+ * .|*

    *|.. ++.. +..o =.+.|*

    *|  .o+o   Eo oo+o |*

    *+----[SHA256]-----+*

<iframe src="https://medium.com/media/8f035591ac42191488e27ae142f66c47" frameborder=0></iframe>

    ** and in **var.my_public_key set** the location as **/tmp/id_rsa.pub
    * **To refer to this file, we need to use the file function*

* *After AMI and Keys out of our way, let start building EC2 instance*

<iframe src="https://medium.com/media/321bb8eca1c4a9fdad264c304085b310" frameborder=0></iframe>

    *Most of these parameters I already discussed in the first section, but let's quickly review it and check the new one
    * count: The number of instance, we want to create
    * ami: This time we are pulling ami using data resource
    * instance_type: Important parameter in AWS Realm, the type of instance we want to create
    * key_name: Resource we create earlier and we are just referencing it here*

    *Below two ones are special, because both of these resource we created during the vpc section, so now what we need to do is to output it during VPC module and use there output as the input to this module. I will discuss more about it later*

    ** tags: Tags are always helpful to assign label to your resources.*
[**100 Days of DevOps — Day 15- Introduction to Terraform**
*Welcome to Day 15 of 100 Days of DevOps, Let continue our journey and focus on Automation especially on Infrastructure…*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-15-introduction-to-terraform-7a168dec8d38)

* *If you notice the above code, one thing which is interesting here is vpc_security_group_ids and subnet_id*

* *The interesting part, we already created these as a part of VPC code, so we just need to call in our EC2 terraform and the way to do it using outputs.tf*

<iframe src="https://medium.com/media/8e4289419ec291c5bf09bff74b6acf4b" frameborder=0></iframe>

* *After calling these values here, we just need to define as the part of main module and the syntax of doing that is*

    *module.<module name>.<output variable>*

    *subnet_id      = "${module.vpc_networking.public_subnets}"                         security_group = "${module.vpc_networking.security_group}"*

* *Final module code for EC2 instance look like this*

<iframe src="https://medium.com/media/4925677df0084e7a4faad2a79a200a6a" frameborder=0></iframe>

* *Let’s create two EBS volumes and attach it to two EC2 instances we created earlier*

<iframe src="https://medium.com/media/19b1bd0370b6943442b7feea7fa2f88b" frameborder=0></iframe>

    ** To create EBS Volumes, I am using ebs_volume resource and to attach it use aws_volume_attachment
    * We are creating two Volumes here
    * AS Volume is specific to Availibility Zone, I am using aws_availibility_zone data resource
    * Size of the Volume is 10GB
    * Type is gp2(other available options "standard", "gp2", "io1", "sc1" or "st1" (Default: "standard"))*
[**AWS: aws_ebs_volume - Terraform by HashiCorp**
*Provides an elastic block storage resource.*www.terraform.io](https://www.terraform.io/docs/providers/aws/r/ebs_volume.html)
[**AWS: aws_volume_attachment - Terraform by HashiCorp**
*Provides an AWS EBS Volume Attachment*www.terraform.io](https://www.terraform.io/docs/providers/aws/r/volume_attachment.html)

* *Next step is to define user data resource, what this will do, during the instance building process it’s going to attach EBS Volumes we created in earlier step.*

<iframe src="https://medium.com/media/e2ad3e8b2e72514c063165a5e3426b39" frameborder=0></iframe>

<iframe src="https://medium.com/media/fa115c42abdab5571478be08df1bcb48" frameborder=0></iframe>

    ** Special parameter in this is path.module which is going to refer to exisiting module path in our case ec2_instance*

* *The last step is to refer user_data resource back in your terraform code*

    *user_data = **"${**data**.template_file.user-init.rendered}"***

<iframe src="https://medium.com/media/14e3f7d394acd5cc47c507cd3ab3389b" frameborder=0></iframe>

* *After that, You need to follow the same steps we have done while creating modules for VPC*

    ***# Step1***

    *$ git clone https://github.com/100daysofdevops/100daysofdevops.git*

    *Cloning into '100daysofdevops'...*

    *remote: Enumerating objects: 83, done.*

    *remote: Counting objects: 100% (83/83), done.*

    *remote: Compressing objects: 100% (64/64), done.*

    *remote: Total 83 (delta 28), reused 41 (delta 12), pack-reused 0*

    *Unpacking objects: 100% (83/83), done.*

    ***#Step2***

    *cd 100daysofdevops/two-tier-environment/*

    ***#Step3***

    *Run all the terraform command*

    ** terraform init
    * terraform plan
    * terraform apply*

*GitHub Link*
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/tree/master/two-tier-environment)

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
