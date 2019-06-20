
# Setting up Terraform Remote State

basic state file with no resources created yet!

**Introduction:**

When working with Terraform as a team, it is always ideal to set up a remote state as multiple people want to update the same state file and it is not really easy to work with multiple copies of divergent state files. One of the best features of setting up Remote state, along with the fact that a single state file is accessed and used by all the team members is that the state file comes with a locking mechanism. The remote state setup can be achieved by setting up backends specific to the cloud. For example, s3 is the backend used for AWS whereas storage accounts are used in Azure

**Creating the resources needed to setup Remote State:**

The first and foremost thing that most first timers miss (including myself when I started working on a remote state initially) is that, the resources which will be used as the backend, for instance, the s3 bucket and the dynamoDB or the Azure storage container, have to be created initially either with terraform (recommended) or manually in order to set up a remote backend in AWS or Azure

**Requirements for setting up a remote state in Azure:**

Azure uses storage accounts and stores the state as a blob with a key within the container of the storage account. This blob container also supports state locking and consistency checking with the help of native blob storage capabilities.

The following three resources have to be created initially in order to set up a remote state for Terraform in Azure

1. Resource group which contains the storage account. It is ideal to be different than the resource group in which all the needful resources are created

1. Storage account which will contain the blob storage container within which the remote state file will be located

1. Blob storage container which stores the remote file along with providing the locking facility so that only one person can access the state file (run terraform plan or apply) at the same time thus ensuring the state file is not corrupted.

    resource "azurerm_resource_group" "resource_group" {
      name = "backend_rg"
      location = "us-west-2"
    }

    resource "azurerm_storage_account" "az_backend" {
      name                     = "azbackend"
      resource_group_name      = "backend_rg"
      location                 = "us-west-2"
      account_tier             = "standard"
      account_replication_type = "LRS"
      tags {
        env = "prod"
      }
      lifecycle {
        prevent_destroy = true
      }
    }

    resource "azurerm_storage_container" "az_state_lock" {
      name                  = "azstatelock"
      resource_group_name   = "backend_rg"
      storage_account_name  = "${azurerm_storage_account.az_backend.name}"
      container_access_type = "private"
      lifecycle {
        prevent_destroy = true
      }
    }

After all these three resources are created then the backend has to be initialized using these resources. They can either be created in the same tf file or can be created by modularizing them if needed. For starters, creating them using the same tf file should be good. All these resources should ideally be created in a separate working directory and not be included along with the resources that would be using the remote state

Example backend config in Azure using the following resources in a separate working directory would be as follows

    terraform {
      backend "azurerm" {
        storage_account_name  = "azbackend"
        container_name        = "azstatelock"
        key                   = "azure_terraform.tfstate"
        access_key            = "<access_key>"
      }
    }

The access key can be obtained by printing it as an output (output.tf) when creating the storage resource or manually exploring it using the Azure portal

    output "properties"{
      value = "${azurerm_storage_account.az_backend.primary_access_key}"
    }

The terraform backend wouldn’t be able to use variable interpolations as this is the first thing that gets loaded when we use terraform init and it doesn’t have any information about variables.tf file.

    terraform init

After running the above command the terminal prompts if the existing state file can be exported to the new remote that is set up. If the existing state has to be preserved, the answer is yes. With this step, the remote state would be set up and any resources created would be stored in the remote tf file. The directory layout would typically be as follows

    -remote_state  #all the resources in this have to be created first
       --.terraform
       --setup.tf #contains resources to create backend
       --output.tf
       --terraform.tf
    -azure
        --.terraform
        --main.tf      # resources of interest that were/will be created
        --azbackend.tf # contains backend config

**Requirements for setting up a remote state in AWS:**

AWS uses s3 to store the terraform.tf file and uses dynamoDB to store the state lock information. So these two resources have to be created first in order to set up a backend for the resources of interest that are/will be created with terraform. It is ideal to create these resources in a separate working directory from the directory where the remote state is going to be set up using these resources.

    resource "aws_s3_bucket" "tf-state-storage" {
        bucket = "terraform-state-storage"
        versioning {
          enabled = true
        }
        lifecycle {
          prevent_destroy = true
        }
    }

    # create a dynamodb table for locking the state file
    resource "aws_dynamodb_table" "dynamodb-terraform-state-lock" {
      name = "terraform-state-lock"
      hash_key = "LockID"
      read_capacity = 20
      write_capacity = 20

    attribute {
        name = "LockID"
        type = "S"
      }
    }

After these resources are created, they can be used to configure the s3 backend which can be used to store the remote state. The following configuration helps to achieve that.

    terraform {
      backend "s3" {
        encrypt=true
        bucket = "terraform-state-storage"
        dynamodb_table = "terraform-state-lock"
        key    = "state-lock-storage.keypath"
        region = "us-west-2"
        access_key = "<aws_access_key>"
        secret_key = "<aws_secret_key>"
      }
    }

The terraform working repository has to be initialized again to reflect the new backend changes

**Closing Remarks:**

Using a remote state with terraform is always a better choice. A backend allows you to encrypt your files adding an extra layer of security for any secrets stored in your state. Versioning the state is also advantageous so that one can compare the differences between previous and current runs

Also, it’s a lot harder to accidentally delete your state file when it is stored remotely than on the local machine. Once the resources needed to create the remote state are created, it is just a block of code to include the backend both in AWS and Azure.

I hope you have found my post helpful. Any feedback about it is appreciated
