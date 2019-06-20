
# Using Terraform with Google Cloud Platform — Part 1/n



This blog series is aimed at those who are interested in developing scalable cloud infrastructure and automating repetitive tasks. I’ll walk you through the setup process to get Google Cloud Platform and Terraform working together and show you how to create a basic virtual machine using 3 files and less than 40 lines of code.

First, let’s talk about how infrastructure development use to be accomplished. Let’s use an example and show how it might have been solved. Imagine we’re a system admin for a medium-sized company that has several well-known clients. One of those clients needs us to create 30 servers, all which have different disk sizes, memory, processors, and operating systems. Oh, and to make things more difficult, they’re in different regions and spread across different cloud vendors — Google Cloud Platform, Amazon Web Services, and Microsoft Azure.

What a nightmare! This is going to take us hours to set up, and we might make some mistakes if we’re not careful. We also have no way to easily test if everything is working, so it’ll likely take us days to debug if we’re not incredibly smart. Thankfully, we have tools like Terraform that allow us to turn a little bit of code into something that can plan, deploy, modify, and destroy all of our systems. If we’re able to get it working, we’ll also need to make some changes to each system, such as modifying the disk size and memory, so that our client isn’t wasting money on unused resources.

Instead of modifying an existing system using SSH, which is a mutable process, your systems are rebuilt from a well-reviewed template, validated for correctness, and then deployed if they pass all the required checks. This is what’s called “immutable infrastructure”. Here’s a good explanation on [**what immutable infrastructure is](https://www.digitalocean.com/community/tutorials/what-is-immutable-infrastructure)**, along with some advantages to using this type of process.

Now let’s walk through several basic examples, define some important terms, and talk about the benefits of using Terraform.

Let’s get started with defining some terms and technology:

* **Terraform:** a tool used to turn infrastructure development into code.

* **Google Cloud SDK: **command line utility for managing Google Cloud Platform resources.

* **Google Cloud Platform:** cloud-based infrastructure environment.

* **Google Compute Engine: **resource that provides virtual systems to Google Cloud Platform customers.

### Why should I use Terraform?

So, why do we want to use Terraform? Because doing things manually is inefficient, sometimes boring, and could also lead to misconfigurations and costly mistakes. We also want to be able to spend more time focusing on more important things, such as the security of our services, our product’s features, and things of that nature.

I don’t want to go in depth about all the features of Terraform because that’s already well documented on their [**introduction page](https://www.terraform.io/intro/index.html)**.

Alright, let’s get into some basic examples on how to use Terraform with Google Cloud Platform.

### Downloading, installing and configuring Terraform

The first thing you’ll want to do is download Terraform. I’m using macOS Sierra 10.12.6 and my shell is ZSH.

You’ll want to run the following commands to download, install, and configure Terraform.

1. Download and unzip Terraform 0.10.8:

    **$ mkdir /usr/local/terraform && cd /usr/local/terraform**

    **$ wget [https://releases.hashicorp.com/terraform/0.10.8/terraform_0.10.8_darwin_amd64.zip](https://releases.hashicorp.com/terraform/0.10.8/terraform_0.10.8_darwin_amd64.zip?_ga=2.49839270.1833144170.1509008777-920873639.1508899386)**

    **$ unzip [terraform_0.10.8_darwin_amd64.zip](https://releases.hashicorp.com/terraform/0.10.8/terraform_0.10.8_darwin_amd64.zip?_ga=2.49839270.1833144170.1509008777-920873639.1508899386) -d /usr/local/terraform**

2. Set the PATH variable for Terraform:

    **$ vim ~/.zshrc **

    . . . 
    # Required by Terraform
    export PATH=$PATH:/usr/local/terraform
    . . .

    **$ source ~/.zshrc**

3. Make sure Terraform works:

    **$ terraform -v
    **Terraform v0.10.8

If your output is the same as mine, you can move onto the next step. If not, try reopening your terminal window and check that you specified the correct location of the Terraform binary file.

### Downloading and configuring Google Cloud SDK

Now that we have Terraform installed, we need to set up the command line utility to interact with our services on Google Cloud Platform. This will allow us to authenticate to our account on Google Cloud Platform, and subsequently use Terraform to manage our infrastructure.

Let’s get started, shall we?

1. Download and install Google Cloud SDK:

    **$ curl https://sdk.cloud.google.com | bash**

    . . .

    **$ source ~/.zshrc**

2. Initialize the gcloud environment:

    **$ gcloud init**

You’ll be able to connect your Google account with the gcloud environment by following the on-screen instructions in your browser. If you’re stuck, try checking out the [**official documentation](https://cloud.google.com/sdk/downloads)**.

### Configuring our Service Account on Google Cloud Platform

In the following few paragraphs I’ll explain how to create a project, set up a service account and set the correct permissions to manage our project’s resources.

1. Create a project and name it whatever you’d like.

1. Create a service account and specify the compute admin role.

1. Download the generated JSON file and save it to your project’s directory.

![Here’s an example from my service account dashboard.](https://cdn-images-1.medium.com/max/5182/1*jNSNok7RzSaFIRFBsI7aVQ.png)*Here’s an example from my service account dashboard.*

Be warned, the JSON file you just downloaded should be protected from non-authorized users. Think of this as a private key or password to manage your infrastructure’s resources. For development purposes we can add a **.gitignore **file to our project, adding **terraform-account.json **so that it’s not committed to our repository.

### Creating a Compute Engine Instance with Terraform

This is what you’ve been waiting for! We’re almost ready to use Terraform to create some VM instances. However, we have to do a few more things.

You’ll first need to create a few files to work with. The most important thing is that each file ends in **file.tf.** This allows Terraform to know what files to work with when initializing, planning, applying, and destroying.

Here are some examples:

    **$ vim provider.tf**

    # Specify the provider (GCP, AWS, Azure)
    provider “google” {
    credentials = “${file(“terraform-account.json”)}”
    project = “system-automation-184009”
    region = “us-central1”
    }

    **$ vim compute.tf**

    *# Create a new instance
    *resource "google_compute_instance" "ubuntu-xenial" {
       name = "ubuntu-xenial"
       machine_type = "f1-micro"
       zone = "us-west1-a"
       boot_disk {
          initialize_params {
          image = "ubuntu-1604-lts"
       }
    }

    network_interface {
       network = "default"
       access_config {}
    }

    service_account {
       scopes = ["userinfo-email", "compute-ro", "storage-ro"]
       }
    }

We’ll also need to follow the proceeding steps to make Terraform aware of the files we just created. Your screen should be similar to what’s presented below.

    **$ terraform init**

    Initializing provider plugins…
    - Checking for available provider plugins on [https://releases.hashicorp.com](https://releases.hashicorp.com)...
    - Downloading plugin for provider “google” (1.1.1)…

    The following providers do not have any version constraints in configuration,
    so the latest version was installed.

    To prevent automatic upgrades to new major versions that may contain breaking
    changes, it is recommended to add version = “…” constraints to the
    corresponding provider blocks in configuration, with the constraint strings
    suggested below.

    * provider.google: version = “~> 1.1”

    Terraform has been successfully initialized!

    You may now begin working with Terraform. Try running “terraform plan” to see
    any changes that are required for your infrastructure. All Terraform commands
    should now work.

    If you ever set or change modules or backend configuration for Terraform,
    rerun this command to reinitialize your working directory. If you forget, other
    commands will detect it and remind you to do so if necessary.

You’ll now want to run the following command to see what we’re about to create and configure. Once again, your output should be similar.

    **$ terraform plan**

    Refreshing Terraform state in-memory prior to plan…
    The refreshed state will be used to calculate this plan, but will not be
    persisted to local or remote state storage.

     — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — 

    An execution plan has been generated and is shown below.
    Resource actions are indicated with the following symbols:
     + create

    Terraform will perform the following actions:

    google_compute_instance.ubuntu-xenial
     id: <computed>
     boot_disk.#: “1”
     boot_disk.0.auto_delete: “true”
     boot_disk.0.device_name: <computed>

    . . . 

    Plan: 1 to add, 0 to change, 0 to destroy.

     — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — 

    Note: You didn’t specify an “-out” parameter to save this plan, so Terraform
    can’t guarantee that exactly these actions will be performed if
    “terraform apply” is subsequently run.

Now that everything looks good, and is staged, we can apply those settings and create our VM instance.

    **$ terraform apply**

    google_compute_instance.ubuntu-xenial: Creating…
     boot_disk.#: “” => “1”
     boot_disk.0.auto_delete: “” => “true”
     boot_disk.0.device_name: “” => “<computed>”
     
    . . .

    Error applying plan:

    1 error(s) occurred:

    * google_compute_instance.ubuntu-xenial: 1 error(s) occurred:

    * google_compute_instance.ubuntu-xenial: The user does not have access to service account '[298693621029-compute@developer.gserviceaccount.com](mailto:298693621029-compute@developer.gserviceaccount.com)'.  User: '[terraform@system-automation-184009.iam.gserviceaccount.com](mailto:terraform@system-automation-184009.iam.gserviceaccount.com)'.  Ask a project owner to grant you the iam.serviceAccountActor role on the service account

    Terraform does not automatically rollback in the face of errors.
    Instead, your Terraform state file has been partially updated with
    any resources that successfully completed. Please address the error
    above and apply again to incrementally change your infrastructure.

Oh no, we have an error! Actually, I did this on purpose to help you learn how to troubleshoot common mistakes when configuring service accounts.

Let’s focus on the following error output:

    * google_compute_instance.ubuntu-xenial: The user does not have access to service account '[298693621029-compute@developer.gserviceaccount.com](mailto:298693621029-compute@developer.gserviceaccount.com)'.  User: '[terraform@system-automation-184009.iam.gserviceaccount.com](mailto:terraform@system-automation-184009.iam.gserviceaccount.com)'.  **Ask a project owner to grant you the iam.serviceAccountActor role on the service account**

This is basically saying that we need to grant our **terraform@system-automation-184009.iam.gserviceaccount.com **service account the same permissions as **298693621029compute@developer.gserviceaccount.com.**

You should see a similar output as shown below:

    **$ gcloud projects get-iam-policy system-automation-184009**

    bindings:
    - members:
     — serviceAccount:[terraform@system-automation-184009.iam.gserviceaccount.com](mailto:terraform@system-automation-184009.iam.gserviceaccount.com)
     role: roles/compute.admin

    . . .

    - members:
     — serviceAccount:[298693621029-compute@developer.gserviceaccount.com](mailto:298693621029-compute@developer.gserviceaccount.com)
     role: roles/editor

    . . . 

    version: 1

Let’s make some changes and see what happens:

    **$ gcloud projects add-iam-policy-binding system-automation-184009 \
          --member serviceAccount:[terraform@system-automation-184009.iam.gserviceaccount.com](mailto:terraform@system-automation-184009.iam.gserviceaccount.com) -- role roles/editor**

    bindings:
    - members:
     — serviceAccount:[terraform@system-automation-184009.iam.gserviceaccount.com](mailto:terraform@system-automation-184009.iam.gserviceaccount.com)
     role: roles/compute.admin

    . . .

    - members:
     — serviceAccount:[298693621029-compute@developer.gserviceaccount.com](mailto:298693621029-compute@developer.gserviceaccount.com)
     role: roles/editor

    . . .

     — serviceAccount:[terraform@system-automation-184009.iam.gserviceaccount.com](mailto:terraform@system-automation-184009.iam.gserviceaccount.com)
     role: roles/editor

    . . .

    version: 1

Everything should be good now that we’ve [**made those modifications](https://cloud.google.com/iam/docs/granting-roles-to-service-accounts)**. Let’s get back to setting up our VM instance:

    **$ terraform apply**

    google_compute_instance.ubuntu-xenial: Creating…
     boot_disk.#: “” => “1”
     boot_disk.0.auto_delete: “” => “true”

    . . .

    google_compute_instance.ubuntu-xenial: Still creating... (10s elapsed)
    google_compute_instance.ubuntu-xenial: Creation complete after 17s (ID: ubuntu-xenial)

    Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

    **$ gcloud compute instances list**

    NAME                ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
    ubuntu-1604-server  us-central1-c  g1-small                   10.128.0.2   23.255.255.255  RUNNING

Huzzah, it works! Let’s destroy it.

    **$ terraform destroy**

    google_compute_instance.ubuntu-xenial: Refreshing state… (ID: ubuntu-xenial)

    An execution plan has been generated and is shown below.
    Resource actions are indicated with the following symbols:
     — destroy

    Terraform will perform the following actions:

    - google_compute_instance.ubuntu-xenial

    Plan: 0 to add, 0 to change, 1 to destroy.

    Do you really want to destroy?
     Terraform will destroy all your managed infrastructure, as shown above.
     There is no undo. Only ‘yes’ will be accepted to confirm.

    Enter a value: yes

    google_compute_instance.ubuntu-xenial: Destroying… (ID: ubuntu-xenial)
    google_compute_instance.ubuntu-xenial: Still destroying… (ID: ubuntu-xenial, 10s elapsed)
    google_compute_instance.ubuntu-xenial: Still destroying… (ID: ubuntu-xenial, 20s elapsed)
    google_compute_instance.ubuntu-xenial: Still destroying… (ID: ubuntu-xenial, 30s elapsed)
    google_compute_instance.ubuntu-xenial: Destruction complete after 38s

    Destroy complete! Resources: 1 destroyed.

Keep in mind this is only a simple example on how to set up a barebones VM instance in Google Cloud Platform. Please make sure you know what you’re doing when working in a production environment.

If you want to do some more reading, check out the official [**Google Cloud Provider documentation pages](https://www.terraform.io/docs/providers/google/)**. They’re very useful and contain all the syntax for setting up VM instances, VPC networks, firewall rules, and other resources.

The next part of this series will include steps to implement SSH key authentication and firewall rules and should be released by the second week of November 2017.
