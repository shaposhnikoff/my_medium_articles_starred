
# Migrating Databases using Ansible and Terraform

Ansible and Terraform — Not rocket science anymore!

![Photo by [“My Life Through A Lens”](https://unsplash.com/@bamagal?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/6104/0*z0SYPdCk0XdoS0PL)*Photo by [“My Life Through A Lens”](https://unsplash.com/@bamagal?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

It’s always fun to learn something by getting a hands-on and real-world use case experience. Recently, I worked on a task to set up a database-server in the DEV (development) environment in an automated way. In the beginning, I was thinking to prepare some scripts to do this, later I decided to try out using some cool DevOps tools available today — Ansible & Terraform.

Complete source code: [https://github.com/apkan/dev-db-setup](https://github.com/apkan/dev-db-setup)

### Introduction

This post is about an automated process of creating or replacing a Database server in a DEV environment using a set of production database backups.

You may encounter situations where you need to do testing using production data in a DEV environment. This task may not be very difficult if you have just one database setup. If you have more databases it will be troublesome. You need to perform the following actions:

* Take a backup in the production database server

* Copy the backup file to the DEV database server

* Clean the existing data in DEV databases.

* Import the production backup data into DEV databases.

* Update production related configuration in DB (eg. production web URLs, other application server IPs)

Just imagine if you have 5 different database setups, you need to perform all these tasks 5 times; of course, it can be automated using scripts. But it can be done in a more cleaner way — you can spin a completely new database server using these database backups.

Let’s say you need to create a development database with last month of production data. This is how the setup going to work;

* *configure ‘date’*

* *terraform plan*

* *terraform apply*

![Photo by [Nathan Dumlao](https://unsplash.com/@nate_dumlao?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/11548/0*czQdQrG4g8kl3yKT)*Photo by [Nathan Dumlao](https://unsplash.com/@nate_dumlao?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

That’s it! This will create a new database server along with production data from a particular date in the past in your development environment.

**Database instances in production:
**Let’s assume you have the following database instances in your production environment:

* Accounts Mongo-DB (3 node replica setup)

* Order-history Mongo-DB (3 node replica setup)

* Balance MySQL-DB (Master slave setup)

* Reporting MySQL-DB (Master slave setup)

Also, assume you have already configured daily database backups to GCP storage bucket. You need to use any of these backups in the development environment.

![Production data backups in GCP storage](https://cdn-images-1.medium.com/max/2000/1*P0iugeLIX9wG8LOAtLVezQ.png)*Production data backups in GCP storage*

## What you will see in this post:

I am not going to explain the basic details about Ansible & Terraform in this post. This post will explain the following interesting challenges:

* Execute Ansible playbook from Terraform

* prepare Ansible roles for multiple databases

* Make Ansible roles as idempotent baked in.

* Change production related info in DB.

* Download files from another project in GCP Storage.

Let’s see how this has been done.

### Required Tools

* Ansible

* Terraform

### Execution steps:

* git clone [https://github.com/apkan/dev-db-setup.git](https://github.com/apkan/dev-db-setup.git)

* define variables in terraform.tfvars

* define variables in ansible/dbServer.yml

* export GCP variables (Service account location & GCP storage keys)

    export GOOGLE_APPLICATION_CREDENTIALS=/etc/*****************.json
    export GS_ACCESS_KEY_ID=GO**********HM
    export GS_SECRET_ACCESS_KEY=sS************************aP

* terraform plan

* terraform deploy

![terraform deploy — expected result](https://cdn-images-1.medium.com/max/2104/1*uGHV86O8_sykMyJg8HCrig.png)*terraform deploy — expected result*

I used both products from HashiCorp — Vagrant and Terraform. One cool thing I find in both products is the way it works to create and destroy a setup. Using just one command you can spin-up a whole environment or destroy it. Especially in Terraform, it really helps when you are practicing with your own cloud account. You don’t need to worry much about your bill, you can just destroy the whole setup with one command once you are done.

## Terraform — behind the scenes

![](https://cdn-images-1.medium.com/max/2000/1*cCu8tUpy8afe1A_rJHSmDQ.png)

### Variables

Configure all the variables for Terraform in ‘**terraform.tfvars’** file:

    project                = "ap-evergreen-dev"
    region                 = "asia-southeast1"

    public_key_path        = "~/.ssh/id_rsa.pub"
    private_key_path       = "~/.ssh/id_rsa"
    ssh_user               = "pratheep"
    db_server_name         = "apeg-dev-db"
    db_server_machine_type = "n1-standard-1"
    db_server_zone         = "asia-southeast1-c"
    db_server_image        = "centos-7"
    db_server_disk_size    = "20"

Keeping all the configurable parameters in this file is a good practice for maintainability. Also, its recommended to include this file in ‘**.gitignore**’ so it won’t be committed in your repository. This is to avoid exposing secret information (passwords, security keys, etc) related to your environment.

### Module

Here you create only one database server, so you may think you don’t really need to worry much about creating a module. But, it’s a best practice in Terraform and it’s always good to follow the best practices!

    provider "google" {
      project     = "${var.project}"
      region      = "${var.region}"
    }

    module "db-server" {
      source                  = "modules/db-server"

      ssh_user                = "${var.ssh_user}"
      public_key_path         = "${var.public_key_path}"
      private_key_path        = "${var.private_key_path}"
      db_server_name          = "${var.db_server_name}"
      db_server_machine_type  = "${var.db_server_machine_type}"
      db_server_zone          = "${var.db_server_zone}"
      db_server_image         = "${var.db_server_image}"
      db_server_disk_size     = "${var.db_server_disk_size}"
    }

This helps to group your resources together and make it as an abstract so that you can configure once and reuse it in multiple places throughout your code (Eg. dev, staging, production).

Normally, if you want to create a database server, you need to create using Terraform scripts, and then to install and configure database services, you need to manually execute Ansible-playbooks on that newly created server. You can avoid that manual execution and let Terraform handle all the Ansible-playbook executions for you.

### Execute Ansible playbook from Terraform

I wanted to make this setup executable with just one command. So I spent some time on this to create the communication works between Terraform and Ansible. Please refer to ‘**main.tf**’ file in the DB-server module for complete details about how this has been done.

    [db_server]
    **${host_ip}** ansible_user=pratheep ansible_ssh_common_args='-o StrictHostKeyChecking=no'

I used above ‘**inventory.tpl**’ template file to generate below inventory file for Ansible setup. As you can see here, the *host_ip* value will be replaced by Terraform whenever you run ‘terraform apply’ command.

    [db_server]
    **35.247.165.139** ansible_user=pratheep ansible_ssh_common_args='-o StrictHostKeyChecking=no'

Terraform will execute the ansible-playbook as local_exec.

    provisioner "local-exec" {
        command = "ansible-playbook  -i ./ansible/inventory.yml --private-key ${var.private_key_path} ./ansible/dbServer.yml"
      }

Ansible inventory file will be generated dynamically and the ansible-playbook command will be executed via SSH using your configured SSH key pairs.

## Ansible — behind the scenes

![](https://cdn-images-1.medium.com/max/2000/1*Hog3LSuYHbJfeiq541wRXQ.png)

I created two Ansible roles for this setup — Mongo-DB & MySQL-DB. As explained in the case introduction, this setup requires two separate Mongo DB and MySQL instances.

### Prepare Ansible roles for multiple databases

In a development environment, you don’t normally use separate servers for each database instances. To run multiple instances of the same database, you need to use a different set of configs for each — port, datadir, pidfile, logdir, etc.

The goal is to create an Ansible role as a blueprint for Mongo-DB and MySQL-DB and create any number of databases from that.
> This way of reusing an Ansible role may help in many other cases as well.

<iframe src="https://medium.com/media/8fa97fbd88c3cf9958dbcd7497d03747" frameborder=0></iframe>

### Make Ansible roles as idempotent baked in.

This is to ensure that your environment will be in a consistent state even if you execute Ansible playbook multiple times. Ansible will work fine for package installations — it will simply skip the installations if the package already installed.

But this will be tricky for a DB setup. You only once need to configure the root password. You can make these set of tasks as idempotent using a status file — create a file once all these tasks completed for the first time and these tasks will be skipped if the file exists.

![Create DB users with idempotency](https://cdn-images-1.medium.com/max/2000/1*Le4AyU5fdyB6287_KQxzWA.png)*Create DB users with idempotency*

During consecutive executions:

![Tasks will be skipped during consecutive executions](https://cdn-images-1.medium.com/max/2000/1*p5VzcdVAGyDHZeEtfuICPg.png)*Tasks will be skipped during consecutive executions*

### Change production related info in DB

Let’s say you store some configuration parameters in DB which should be different in a DEV environment than PROD. Also, you need to set up a Redis instance in the same DEV DB server and it’s IP address needs to be updated in Mongo-DB.

You can configure an Ansible task to replace this information whenever you create a DB setup in the DEV environment.

This can be done using the template option in Ansible.

![ansible/roles/mongo-db/templates/configurations.json.j2](https://cdn-images-1.medium.com/max/2000/1*2f174Lt_AAl_DGxYTnkwHQ.png)*ansible/roles/mongo-db/templates/configurations.json.j2*

Just for simplicity, I included only two config options here, but for real setup, you may have more configurations in DB.

### Download files from another project in GCP Storage.

As a standard practice, you set up two separate projects for PROD and DEV environments.

![](https://cdn-images-1.medium.com/max/2000/1*85CK9Sd6lVjtM0xImwHJzA.png)

All the PROD data backups will be stored in a storage bucket in ‘ap-evergreen-prod’ project. You need to download those files from a VM instance in ‘ap-evergreen-dev’ project.

You can use either one of these two options:

1. [Cloud storage transfer service](https://cloud.google.com/storage-transfer/docs/create-manage-transfer-program).

1. [Enable interoperability API](https://cloud.google.com/storage/docs/interoperability) and create keys.

I used the second option for this setup. You need to create keys from ‘GCP Storage → Settings → interoperability’. These two keys need to be exported before you execute Ansible playbook.

    export GS_ACCESS_KEY_ID=GO**********HM
    export GS_SECRET_ACCESS_KEY=sS************************aP

Then, I used the [gc_storage ](https://docs.ansible.com/ansible/latest/modules/gc_storage_module.html)module from Ansible for this.

![Ansible task to download files from GCP storage.](https://cdn-images-1.medium.com/max/2000/1*RYcfCKN3OhDTAgANtF5_Cw.png)*Ansible task to download files from GCP storage.*

## Wrapping up

I hope, the implementation details of this setup can be helpful in other real-world use cases as well.

Few things to keep in mind:

* If your backup data file size is huge, it may take a long time to execute the DB import via Ansible task. Ansible may not be very helpful for long running tasks. In that case, you can configure the setup using Ansible and execute DB import manually.

* Some companies prefer to stop DEV environment servers in cloud during non-working time. In that case, Terraform will have issues in maintaining that infrastructure.

### References:

1. [https://www.redhat.com/cms/managed-files/pa-terraform-and-ansible-overview-f14774wg-201811-en.pdf](https://www.redhat.com/cms/managed-files/pa-terraform-and-ansible-overview-f14774wg-201811-en.pdf)

1. [https://blog.gruntwork.io/a-comprehensive-guide-to-terraform-b3d32832baca](https://blog.gruntwork.io/a-comprehensive-guide-to-terraform-b3d32832baca)
