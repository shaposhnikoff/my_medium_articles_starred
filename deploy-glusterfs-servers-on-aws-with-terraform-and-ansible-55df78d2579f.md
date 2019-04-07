
# Deploy GlusterFS servers on AWS with Terraform and Ansible

How to deploy a very basic GlusterFS cluster for testing purpose on AWS.

At [Alchimie](https://www.alchimie.com/) we use a [GlusterFS](https://www.gluster.org/) cluster for storing our video assets and Apache HTTP servers to deliver our contents.

Before to deploy or doing some maintenance on production you may want to test your procedure and then avoid a tragedy :).

For this reasons I’ve scripted the deployment of a basic cluster on AWS with Terraform and Ansible for testing :

* storage with GlusterFS

* XFS partitioning with LVM

## Build and start the platform with Terraform

I chose Terraform for building this platform :

![](https://cdn-images-1.medium.com/max/2000/1*s1Tqj0j7UAIlPGMR_-LVpw.jpeg)

First define you variables into a file **vars.tf** :

<iframe src="https://medium.com/media/c3eb7f38181cf8d3281876b5af52d11d" frameborder=0></iframe>

Then you can define the common resources (SSH key, OS AMI Id, etc…) into a file named **01-common.tf**:

<iframe src="https://medium.com/media/2c73b190a0a708d3cd210afc1a059011" frameborder=0></iframe>

Then you can create the VPC and the subnets :

<iframe src="https://medium.com/media/8969a556885582d0499f67be720550c8" frameborder=0></iframe>

Now you have the VPC and the subnets, you can run the last step : build ans provision the EC2 instances:

<iframe src="https://medium.com/media/5e77db351fd2666bc5b53d2e20f8fbf0" frameborder=0></iframe>

We use [cloud-init](https://cloud-init.io/) scripts to install some packages on instances, and the file provisionner of Terraform to copy files on hosts.

To complete your setup you have to run the Ansible playbook copied by Terraform on the client instance.

## Setup the Gluster cluster with Ansible

Use the script** setup.sh** to run the setup.

It will install ansible for you and install the roles required to run the playbooks:

<iframe src="https://medium.com/media/b06fd187b7bedd3b0ef0e682c451b2a3" frameborder=0></iframe>

The first playbook use the role [gluster.infra](https://github.com/gluster/gluster-ansible-infra) to create XFS partitions with LVM:

<iframe src="https://medium.com/media/df7bdd5f57d2c9a743a084265999f665" frameborder=0></iframe>

The second playbook install **glusterfs-server** (version 3.12) on the instances, and create a volume with the role [gluster.cluster](https://github.com/gluster/gluster-ansible-cluster):

<iframe src="https://medium.com/media/71450547b54a0edfdd1e4694ae6dc08f" frameborder=0></iframe>

The last playbook install **glusterfs-client** and mount the glusterfs volume on the client instance:

<iframe src="https://medium.com/media/7d98c13e3753d86b8268dd86ea6e8653" frameborder=0></iframe>

*mount_src* variable is defined into the command line. 
See file setup.sh.

Enjoy :)

![](https://cdn-images-1.medium.com/max/2000/1*EZMVWXQCeduc5M75KbrkkA.png)

[Alchimie ](http://fr.alchimie.com/)recherche des ingénieurs talentueux pour construire et développer ses services OTT.

Pour plus d’informations, consultez notre page sur [Welcome to the jungle](https://www.welcometothejungle.co/companies/alchimie).
