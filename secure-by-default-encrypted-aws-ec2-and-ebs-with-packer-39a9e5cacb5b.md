
# Secure by default — Encrypted AWS EC2 and EBS with Packer

I’ve worked on numerous AWS based projects, in a number of highly regulated industries, throughout the years. Whether they fall under PCI, HIPAA, or FISMA, one of the first things we as engineers have had to do is ensure that all data in our systems is encrypted at rest. What does that mean in the AWS world? Ultimately it starts with encrypting all EBS volumes, including the root device attached to EC2 instances.

The way to ensure that all instances start (launch) encrypted by default, is to create an AMI with an encrypted root device. Any EC2 instance, or service requiring an EC2 instance (like EMR), should be created using these AMIs.

My teams over the years have solved this a number of ways, by following the same basic [steps](https://aws.amazon.com/blogs/aws/new-encrypted-ebs-boot-volumes/).

![Image from AWS [blog](https://aws.amazon.com/blogs/security/how-to-create-a-custom-ami-with-encrypted-amazon-ebs-snapshots-and-share-it-with-other-accounts-and-regions/) on encrypting EBS volumes](https://cdn-images-1.medium.com/max/2000/0*FGNMsoFPdQaXAuJL.png)*Image from AWS [blog](https://aws.amazon.com/blogs/security/how-to-create-a-custom-ami-with-encrypted-amazon-ebs-snapshots-and-share-it-with-other-accounts-and-regions/) on encrypting EBS volumes*

In order to accomplish this we’ve used a number of approaches including Terraform and even just the AWS console. However, there were problems with with these solutions. The AWS console approach is manual and therefore not repeatable while Terraform does not officially support encrypted volume AMIs and requires a [workaround](https://github.com/terraform-aws-modules/terraform-aws-ec2-instance/issues/6).

The approach I’ve found to be the most scalable, repeatable, and maintainable is via [Packer](https://www.packer.io/intro/), by Hashicorp.

![](https://cdn-images-1.medium.com/max/2000/1*QXBA8jahvaqaGYMkxZjgkA.png)

## How it works

Packer works by ingesting a configuration file you provide and popping out a machine image with an operating system and various other softwares installed on it. This machine image can then be used as the blueprint to create virtual machines. Packer supports a multitude of image [formats](https://www.packer.io/intro/index.html), including AWS AMIs.

## Getting starter

Packer has a few [installation](https://www.packer.io/intro/getting-started/install.html) methods, the easiest of which is to download the [pre-compiled binary](https://www.packer.io/intro/getting-started/install.html#precompiled-binaries).

Hashicorp also supports a [docker container ](https://hub.docker.com/r/hashicorp/packer/)for running commands, which I find to be quite convenient for playing around with Packer for the first time. I recommend giving that a try!

## Defining the configuration

Packer uses JSON to define its configuration. Let’s take a look at a sample Packer configuration to create a simple Ubuntu AMI

<iframe src="https://medium.com/media/1d3e89e47e964a004816371951966e16" frameborder=0></iframe>

This barebones example is pretty straight forward. You just need to specify the machine type (in this case **amazon-ebs**) and various metadata like name, description, etc. The other interesting bit to pay attention to is **source_ami_filter. **This section of the template describes how Packer should find the AMI to use as our base image. In this case it will grab the most recent (**most_recent: true**) version of the **ubuntu-xenial-16.04-amd64-server **image. It is worth noting that the seemingly magical number, **099720109477**, is the AWS marketplace owner id for [Canonical](https://aws.amazon.com/marketplace/seller-profile?id=565feec9-3d43-413e-9760-c651546613f2), the group that supports [Ubuntu](https://www.canonical.com/).

Additionally, Packer will need the correct IAM permissions in order to create the AMI. Here is an example policy with the minimum IAM permissions required:

<iframe src="https://medium.com/media/91ea9095c349ddb17a4045716fe89c06" frameborder=0></iframe>

Further details on how you can grant Packer the appropriate permissions can be found [here](https://www.packer.io/docs/builders/amazon.html).

## Encrypting Root Device

We could use the example template to generate an AMI, but the root volume for any EC2 instance using the AMI would be unencrypted; so how do we extend it to encrypt the root device? We just need to add the following JSON key-value pair:

    "encrypt_boot": true

And here is what out example looks like now

<iframe src="https://medium.com/media/aeb5c7fd483cce7510e4658bfd635b88" frameborder=0></iframe>

That’s it! After adding our encrypted flag we can run the Packer [build](https://www.packer.io/docs/commands/build.html) command

    packer build encrypted_ami.json

If our AWS credentials are setup correctly, we should see something similar to the following in the terminal

![Sample packer terminal output](https://cdn-images-1.medium.com/max/3232/1*X1GW1pTQ4HeSJvWVO0DNmw.png)*Sample packer terminal output*

Once Packer has finished we can go to the AWS console and see the AMI that was created

![AWS Console view of Encrypted EBS AMI](https://cdn-images-1.medium.com/max/4316/1*0JKxA5foUQ9wBpmeSk-Obw.png)*AWS Console view of Encrypted EBS AMI*

If we launch an instance using this AMI we can also see that the root device is encrypted

![](https://cdn-images-1.medium.com/max/4492/1*caybRPVHXs7vw9-hWYjmQg.png)

## Additional Volume Encryption

Encrypting additional volumes is just as easy. Here is an example snippet for specifying an encrypted EBS volume as part of the AMI:

<iframe src="https://medium.com/media/1b2f1351cc50f3fb147c1ba1eba4d664" frameborder=0></iframe>

And here’s the full example:

<iframe src="https://medium.com/media/cd893e22f51ba65b750c9a09808b7822" frameborder=0></iframe>

That’s it!

## Summary

Encryption is an important part of keeping our systems secure in order to protect user data. Cloud providers like AWS give us the tools to build secure systems; while tools like Hashicorp’s Packer work to give us the ability to harness these tools more easily.

## Where to go next?

Packer is designed to create an easy workflow for creating machine images. As part of that workflow I encourage you to check out how to introduce testing into that process with tools like Chef’s [InSpec](https://www.chef.io/products/chef-inspec/). I’ve written a bit on [InSpec already](https://medium.com/@andrew.larse514/tdd-with-inspec-and-dockerfiles-71fdcf1d716d) as well!

*A complete working example of the code snippets from article can be found in the following github repository*
[**larse514/packer-example**
*Contribute to larse514/packer-example development by creating an account on GitHub.*github.com](https://github.com/larse514/packer-example)
