
# Manage AWS VPC as Infrastructure as Code with Terraform



In this tutorial, I will show you how to setup a **VPC** as described in the network diagram below in less than **1 min **using **Terraform**:

![](https://cdn-images-1.medium.com/max/2000/0*JrIFRvnGAA7LVq4q.)

The **VPC** topology above is the best demonstration of what will be implemented:

* The **private subnet** is inaccessible to the internet (both in and out)

* The **public subnet** is accessible and all traffic (0.0.0.0/0) is routed directly to the **internet Gateway**

Before we dive in, all the code used in this demo is available at my [**Github](https://github.com/mlabouardy/terraform-aws-labs)**.

Note: I already did a [**tutorial ](http://www.blog.labouardy.com/manage-aws-infrastracture-as-code-with-terraform/)**on how to get started with **Terraform** so make sure to read it for more details.

**1 — Global variables**

This file contains environment specific configuration like region name, CIDR blocks, and AWS credentials …

<iframe src="https://medium.com/media/0f47e82b75013c9442007b248a3ef85f" frameborder=0></iframe>

**2 — Configure the AWS provider**

<iframe src="https://medium.com/media/403213016326509f75015107cf9c260d" frameborder=0></iframe>

**3 — Create a VPC**

<iframe src="https://medium.com/media/b9cbe335468384b2b88e116d3147293b" frameborder=0></iframe>

**4 — Create Subnets**

<iframe src="https://medium.com/media/424888cac9036f5d43d4c72eeb37ef5b" frameborder=0></iframe>

To make the **public subnet** addressable by the **Internet**, we need an **Internet Gateway**:

**5 — Internet Gateway**

<iframe src="https://medium.com/media/8364974ca1116ca96dbf6fa66bbf425d" frameborder=0></iframe>

To allow traffics from the **public subnet** to the internet throught the **NAT Gateway**, we need to create a new **Route Table**.

**6 — Route Table**

<iframe src="https://medium.com/media/3a5ea69dddaa559bbd21cb5d77f438a3" frameborder=0></iframe>

Next, we will create a security group for each subnet.

**7 — Security Groups**

**7 .1 — WebServer SG**

<iframe src="https://medium.com/media/9eb4b303ba30af886a2759d8f8bd47b0" frameborder=0></iframe>

This **Security Group **allows **HTTP/HTTPS** and **SSH** connections from **anywhere**.

**7.2 — Database SG**

<iframe src="https://medium.com/media/43ba6c4d0a0f7a5d28980f7350a48014" frameborder=0></iframe>

This **Security Group** enable **MySQL 3306** port, **ping** and **SSH** only from the **public subnet**.

Now we will deploy the **EC2** instances, but before that we need to create a **key pair** in order to connect later to the instances via **SSH**.

**8 — Key Pair**

<iframe src="https://medium.com/media/e42fd1ed88ffbe4ed7cc714146483ccc" frameborder=0></iframe>

**9 — EC2 Instances**

**9.1 — WebServer Instance**

<iframe src="https://medium.com/media/7bdfb31388184f8db7a50de91ef23517" frameborder=0></iframe>

This instance will play the role of a **webserver.** Therefore, we pass to the instance **userdata** a shell script **install.sh** which contains commands to install an **Apache Server**:

<iframe src="https://medium.com/media/9680e08b258a72c7e8ceda385e9eba9b" frameborder=0></iframe>

**9.2 — Database Instance**

<iframe src="https://medium.com/media/38157556af67cab6762d90dfd8cd36a2" frameborder=0></iframe>

Once you’ve defined all the required templates, make sure to set the **AWS** **credentials** variables as an **envrionment variables**:

|export AWS_ACCESS_KEY_ID=”YOUR ACCESS KEY ID”
|export AWS_SECRET_ACCESS_KEY=”YOUR SECRET ACCESS KEY”

Note: You can always use your **root user** which has access permission to everything, but for security perspective, its recommended to use only a limited permissions user account. So create a new one using **AWS IAM**.

To see how terraform plans to create the resources type “**terraform plan**“. To create the infrastructure type “**terraform apply**“:

![](https://cdn-images-1.medium.com/max/2000/0*8ZBaMFpWzm0Axa6h.)

That will bring up the **VPC**, and all the necessary resources. Now in your [**AWS Management Console](https://console.aws.amazon.com/)** you should see the resources created:

![](https://cdn-images-1.medium.com/max/2000/0*JmCBDx89kh40EK2M.)

If you click on the “**Subnets**” menu, you should see the **public** & **private subnets**:

![](https://cdn-images-1.medium.com/max/2000/1*tOLPMktXJwTtBze1NufvDQ.png)

The same goes for the **Route Tables**:

![](https://cdn-images-1.medium.com/max/2000/1*MVLuV_S91142DwquvMePzg.png)

And the** Internet Gateway**:

![](https://cdn-images-1.medium.com/max/2000/1*L6VHOfqv4tF9foyNqiScRw.png)

**Security Groups** also:

![](https://cdn-images-1.medium.com/max/2000/1*8QtqrBSP0iChVP3bQX__Yw.png)

**WebServer Security group:**

![](https://cdn-images-1.medium.com/max/2000/0*q-73WB-_QcvrL0vF.)

**Database Security Group:**

![](https://cdn-images-1.medium.com/max/2000/0*qQCmRMiENKY1A4gu.)

And finally the **EC2 Instances:**

![](https://cdn-images-1.medium.com/max/2000/1*sYFRuME7rjJV-W6X6iiZyA.png)

**WebServer Instance:**

![](https://cdn-images-1.medium.com/max/2000/1*cOlA_qc3lKKGQNg50IuJDw.png)

**Database Instance:**

![](https://cdn-images-1.medium.com/max/2000/1*vyxDBy_Y00EOQG8UMJmiGg.png)

Don’t forget to destroy the resources if they are not needed by typing “**terraform destroy**“:

![](https://cdn-images-1.medium.com/max/2000/1*lqLd8gbQ9buCrAF_0mXXvw.png)
