
# Azure Management using Hashicorp Terraform

When it comes to managing your Azure Resources, you have many options available. If your organization adheres to Infrastructure as Code principles, then odds are high you are considering, or actively using, HashiCorp Terraform to manage those Cloud Resources.

We have been using Terraform to manage the majority of our Azure footprint for many years now. In this time, we have iterated on our implementation a few times; learning along the way what seemed to work, what didn't, and what was cumbersome. Hopefully, by the time you are done with this article, you will have some idea how to structure your Terraform code to make managing your Azure Tenant easier.

This article is not meant as a primer for Terraform, and will not be teaching the reader how to use Terraform. This article assumes the reader has some understanding of the role Terraform plays in Infrastructure Management and Automation. This article does contain numerous links to documentation so that the reader is able to continue their research on their own.

## Code Organization

Terraform is a very open tool, and there are many ways to work with [Terraform Code](https://www.terraform.io/docs/configuration/index.html). There is nothing to stop you from creating one single code file that contains *all* of your infrastructure code. In fact, many people who are dipping their toes into Infrastructure as Code might organize their code in this manner.

As any developer knows, organizing your code is a key item that must be addressed if you want your project to be successful in the long run. No one likes working with dozens of code files that have no logical structure or order. This article details the code structure that we adhere to when it comes to organizing our Terraform code.

We organize our Terraform code into two separate Repositories (Repo’s); **terraform-modules** and **terraform-azure**. Each of these Repo’s contains Terraform code which serves a specific purpose; **terraform-modules** contains the Module Definitions that ***terraform-azure*** consumes, while **terraform-azure** contains the Terraform code that describes the Resources we have deployed to Azure. What follows is a detailed overview of these two Repositories, the role they play in our IaC deployment.

## terraform-modules

**terraform-modules** is our Module Library and contains all the [Resources](https://www.terraform.io/docs/configuration/resources.html) currently available from the [Azure Provider for Terraform](https://www.terraform.io/docs/providers/azurerm/index.html). The structure of this Repo closely mirrors the structure of the AzureRM Provider documentation. This allows Consumers of the Library to have a reference point which they can use to learn how to implement a given Module.

The code is organized into top-level folders, where the folder name is the name of the Resource Category. For each Category, there are one or more sub-folders for the Resources available within that Category. Within these folders is the Terraform Code itself.

![Azure Resource’s defined as Terraform Modules](https://cdn-images-1.medium.com/max/2000/1*kU8KLvHPRXwas_brRb3ObA.png)*Azure Resource’s defined as Terraform Modules*

As you can see, the code files come in one of three forms;

* main

* variables

* outputs

The **main** file contains the Resource Definition itself, with each property being tied to a given variable. We never hard-code any property values in the main files, the values always come from an associated variable.

![Azure App Configuration Module’s Main File](https://cdn-images-1.medium.com/max/2000/1*3_3qZ_T2xIIQS9i-v43yKw.png)*Azure App Configuration Module’s Main File*

The lifecycle map shown above instructs Terraform to ignore any changes to the “tags” associated with the Resource.

The **variables** file contains all the properties that be can be set on a given Resource. Typically, every property that can be set for a given Resource is represented in these files, though sometimes we omit certain properties that we never use.

![Azure App Configuration Module’s Variables File](https://cdn-images-1.medium.com/max/4668/1*xK15caMN4rWGvJWdTZrh8g.png)*Azure App Configuration Module’s Variables File*

This is the first place where our Azure Governance comes into play. We restrict and limit what people can do, and how people can do things, in all of our Azure Subscriptions. One way we do this is by enforcing standards across all of our Azure Subscriptions and Resources. For example, we only deploy Resources to East US, so for all the Resources which require a location property, we default that variable with the value “East US”, as can be seen in the above image. This is just one example, but there are others as well. Since our Module Library has default values that are specific to us, we have not open-sourced this library on Github.

The **outputs** file contains all the properties that are output from the Resource once it is created. Practically all Resources output at least one property, though some Resources will output multiple properties.

![Azure App Configuration Module’s Output File](https://cdn-images-1.medium.com/max/2220/1*OOXJQigCJv7ItX90bdM9qg.png)*Azure App Configuration Module’s Output File*

In many cases, code in these files is nothing more than a copy-paste of the code from the Documentation.

When it comes to Module Library development, we only build Modules for Azure Resources that we actually work with. As such, to date, we have about 150 of the 300 Resources available in the azurerm provider defined in our Module Library.

### Versioning

Code changes over time, and the azurerm provider is no exception. New properties find their way into the Provider all the time and our “default” way of doing things changes as well. Some Resource properties are immutable, and changing them would cause a Resource to be deleted and recreated using the new property values. We wouldn’t want to reconfigure existing Resources after they have been deployed as this could break things or lead to outages.

At the same time, as new properties and Resources are released, we want to keep our Module Library as up-to-date as possible, without inadvertently breaking things in Azure. When a Resource is deployed to Azure using Terraform, we need a way to ensure that the **definition** of that Resource does not change, even though the underlying Module code may change over time.

For these reasons, we version our Module Library. When we are developing our Module Library, we will cut a new branch off of master and do our development work. When we are complete, we; issue a Pull Request, review the work, and complete a merge to master. At this point, we create a new Tag against the master branch, effectively locking the state of the Repo at that point in time. All Tags follow the [semantic versioning](https://semver.org/) convention.

![Tags available in the Terraform-Modules Repository](https://cdn-images-1.medium.com/max/2152/1*YYg9nOfuXEPEzshexOiUcg.png)*Tags available in the Terraform-Modules Repository*

And yes, they appear out of order in Azure DevOps, which is something we just live with.

## terraform-azure

Code in the **terraform-azure** Repo typically consumes Modules defined in the **terraform-modules** Repo, though there are a few exceptions.

The **terraform-azure** Repo is organized in a way that matches our Azure configuration, with one top-level folder for each Subscription. Within each “Subscription” folder, there is a folder for each Resource Group we have deployed to Azure and within those folders are the Terraform files that describe the Resources deployed to that Resource Group. Terraform is always ran within one of these Subscription folders, never at the root of the Project.

An example of what this hierarchy looks like in practice is as follows;

![An Azure Subscription, with child Resource Groups, defined in Terraform](https://cdn-images-1.medium.com/max/2000/1*Aq6HRqFZS9Npu2doLRBT3A.png)*An Azure Subscription, with child Resource Groups, defined in Terraform*

Each Subscription folder contains 3 separate files; **main**, **providers**, and **variables**.

The Resource Group folders each contain 4 separate files inside of them; **main**, **output**, **rbac**, and **variables**.

When Terraform is ran inside of a folder, it will automatically parse and interpret every file in that folder that has a .tf extension. If a file contains a reference to another folder, Terraform will load all the .tf files in that folder as well. The way you get Terraform to reference code in another folder is by following the [Module Syntax](https://learn.hashicorp.com/terraform/modules/modules-overview), and source-ing the files in another folder.

    module ads-appservices-rg {
        **source = "./ads-appservices-rg"**
    }

    module ads-dev {
        source = "./ads-dev"
    }

It is this Module functionality that allows us to nest and organize our code in a way that is logical and easy to follow.

When we run Terraform, we always run it inside the context of one of the Subscription folders. We never run Terraform at the root of the project.

### Subscription Folder — main, providers, and variables

As previously mentioned, within each Subscription folder are three separate Terraform files; **main**, **providers**, and **variables**.

Let’s start with the **providers** file since this is arguably the most important of the three files. This file is where we declare and configure all the [Providers](https://www.terraform.io/docs/providers/index.html) that are required for a given Subscription. This file also contains the configuration required to store the [State File](https://www.terraform.io/docs/state/index.html) in a secure, remote, location. Since we are working with Azure, we store our State File in an Azure Blob Storage container.

The contents of this file are as follows, with sensitive information removed.

    terraform {
        backend azurerm {
            storage_account_name = "storage_account_name"
            container_name       = "terraform"
            key                  = "ads-dev.tfstate"

            subscription_id     = "guid_of_subscription"
            resource_group_name = "DevOpsSharedStorage"
        }
        required_version = "=0.12.28"
    }

    provider azurerm {
        subscription_id = "guid_of_subscription"
        version         = ">=2.16.0"
        features {}
    }

    provider azuread {
        subscription_id = "guid_of_subscription"
        version         = ">=0.10.0"
    }

    provider vault {
        address = "https://vault.your-domain.com"
        version = ">=2.11.0"
    }

There really isn’t a lot to this file. One thing to notice is that for each of the providers used, we always pin the provider version, especially for fast-moving providers which might introduce breaking changes. We have been burned in the past with providers pushing updates that caused issues with our ability to perform deployments.

The first block configures Terraform to use a [Remote Backend](https://www.terraform.io/docs/backends/index.html), which in this case is an [azurerm backend](https://www.terraform.io/docs/backends/types/azurerm.html). Remote Backends allow a team of engineers to all work on a common Terraform code base, without having all of those engineers store a copy of the State File on their local machines. Not only does this improve team performance, but it also ensures that sensitive data is not left lying around on multiple machines. Anyone working with Terraform in a team environment should be using some form of Remote Backend. Period.

You can see from the next three blocks that we are consuming three providers; [azurerm](https://www.terraform.io/docs/providers/azurerm/index.html), [azuread](https://www.terraform.io/docs/providers/azuread/index.html), and [vault](https://www.terraform.io/docs/providers/vault/index.html). [Vault](https://www.vaultproject.io/), if you aren’t aware, is a Secrets Storage Engine provided by Hashicorp. We store all of our Secrets in Vault, and source those secrets from Vault during the [**plan](https://www.terraform.io/docs/commands/plan.html)** and [**apply](https://www.terraform.io/docs/commands/apply.html)** phase. We will show an example of how we source secrets from Vault in a moment.

The next most important file is the **variables** file. As the name implies, this file is where we store all of the Subscription level variables. This file is where we would put all of the static [variables](https://www.terraform.io/docs/configuration/variables.html) and [**data sources](https://www.terraform.io/docs/configuration/data-sources.html)** that we would need to pass to a “Module.” In Terraform, a data source is used to fetch additional information that is external to the Terraform Code. We use Data Sources to fetch Secrets from Vault, and those secrets are then passed to all “Modules” where they are needed.

Here is an example of a variables file, with sensitive information removed.

    data vault_generic_secret azure_sql_info {
        path = "kv/Azure/azure_sql"
    }

    data vault_generic_secret windows_admin {
        path = "kv/Windows/devops-local-admin"
    }

    data vault_generic_secret azure_sql {
        path = "kv/Azure/Database/ads-dev-sql"
    }

    data vault_generic_secret sqlcredentials {
        path = "kv/Windows/service_account_name"
    }

You will notice that there are no actual **variables** defined in this file. For example, “location” is a common property that numerous resources require. Since we deploy all of our Resources to the East US Region, we could have defined a global variable here and passed it to every resource which needs it. Not only was this visually redundant, but it also added nothing to the codebase. In following clean-code practices, things that don't add value should be removed. We have opted to put static defaults in the **terraform-modules** code base instead, and we override them as needed.

Another thing that might end up in one of these variables files would be an Azure Resource which was NOT created in Terraform, but is still referenced by something that IS defined in Terraform. An example of this would be when you have an existing Resource Group that was NOT defined in Terraform, but you are trying to deploy new Resources to that Resource Group from Terraform. In this case, you would use a [data source to reference that existing Resource Group](https://www.terraform.io/docs/providers/azurerm/d/resource_group.html), and you would pass the output of the data source to the Terraform Resource you are trying to deploy. We have strayed away from this practice though and when we encounter this situation, we simply add the missing Resource to our Terraform Code and [Import](https://www.terraform.io/docs/import/index.html) the [Resource](https://www.terraform.io/docs/providers/azurerm/r/resource_group.html#import) into our State Files as needed.

The final file of interest is the **main **file. This file is just a bunch of Modules which point to the Resource Group Folders via the [source property](https://www.terraform.io/docs/configuration/modules.html#calling-a-child-module) while passing whatever variables are required to each respective Module. In essence, the main file pulls everything together in a way that instructs Terraform how to build its [Dependency Graph](https://www.terraform.io/docs/internals/graph.html) so that Resources are deployed in the correct order, as we will see in a minute.

An example of what this file might look like is as follows. Do note, I wrapped some of the variable values to another line to improve readability.

    module ads-dev {
        source = "./ads-dev"
    }

    module shared-resources-rg {
        source = "./shared-resources-rg"

        vnet_rg            = module.ads-dev.vnet_rg
        vnet_name          = module.ads-dev.vnet_name
        sql_admin_password =
            data.vault_generic_secret.azure_sql_info.data["password"]
    }

    module devops-rg {
        source = "./devops-rg"

        sql_backup_username =
            data.vault_generic_secret.sqlcredentials.data["userid"]
        sql_backup_password =
            data.vault_generic_secret.sqlcredentials.data["Password"]
    }

    module ads-appservices-rg {
        source = "./ads-appservices-rg"
    }

    module ambassador-mobile-app-rg {
        source = "./ambassador-mobile-app-rg"

        shared_app_service_plan_id = module.ads-appservices-rg.shared_app_service_plan_id
    }

    module tms-oms-dev-rg {
        source = "./tms-oms-dev-rg"
    }

    module standard-costing-rg {
        source = "./standard-costing-rg"

        sql_admin_password = data.vault_generic_secret.azure_sql_info.data["password"]
    }

There is a lot going on in this file, so I will take a little bit to describe what is happening here.

The first Module block references the “ads-dev” Resource Group, which is in a folder called “ads-dev”. Every one of our Subscriptions contains a Default Resource Group that is named the same thing as the Subscription. This Resource Group contains the Virtual Network, Peerings, and default Subnet for the Subscription. These Default Resource Groups rarely contain any other Azure Resources. In the case of the Terraform code, this Module does not accept any variables.

    module ads-dev {
        source = "./ads-dev"
    }

    module shared-resources-rg {
        source = "./shared-resources-rg"

        **vnet_rg**            = module.ads-dev.vnet_rg
        **vnet_name**          = module.ads-dev.vnet_name
        **sql_admin_password** =
            data.vault_generic_secret.azure_sql_info.data["password"]
    }

The next block references another Resource Group called “shared-resources-rg” which is also in its own folder. This Module accepts three variables, as can be seen in the above snippet. The first two variable values come from outputs of the “ads-dev” Module; **vnet_rg** and **vnet_name**. This variable binding helps Terraform setup its Dependency Graph; it knows that it must deploy “ads-dev” before it can deploy “shared-resources-rg” because “shared-resources-rg” requires information from the “ads-dev” Module.

The last variable “sql_admin_password” is provided by one of the data sources we showed earlier in the variables file, and its value comes from Vault.

    data vault_generic_secret azure_sql_info {
        path = "kv/Azure/azure_sql"
    }

When we run a plan or apply, Terraform will authenticate to Vault using our credentials, retrieve the secret stored in the path noted above, and then pass the value of the key named “password” to the Module.

This same basic pattern repeats for every Resource Group Folder we have defined in Terraform.

### Resource Group Folder — main, output, rbac, and variables

For reference here is a close-up view of one of the Resource Group Folders.

![An Azure Resource Group defined as a Terraform Module](https://cdn-images-1.medium.com/max/2000/1*x38cJdpq7DBQufFkYEmfUQ.png)*An Azure Resource Group defined as a Terraform Module*

The **main **file contains all the Azure Resources which are deployed to that Resource Group and minimally contains the Resource Group definition itself. Resources in this file are defined by referencing the Modules in the **terraform-modules** Repo.

An example of a **main** file that describes four Azure Resources is as follows;

    module resource_group {
        **source = "git::ssh://git@ssh.dev.azure.com/v3/organization/project_name/terraform-modules//base-resources/resource-groups?ref=v3.0.0"**

        name     = "shared-resources-rg"

        tags = {
            ApplicationID        = "NA"
            BusinessVertical     = "ADS"
            CostCenter           = "5728"
        }
    }

    module shared_resources_oms {
        source = "git::ssh://git@ssh.dev.azure.com/v3/organization/project_name/terraform-modules//oms/log-analytics-workspace?ref=v3.0.2"

        name                = "ads-dev-oms"
        resource_group_name = **module.resource_group.name**
        location            = **module.resource_group.location**
        sku                 = "Standard"
    }

    module subnet {
        source = "git::ssh://git@ssh.dev.azure.com/v3/organization/project_name/terraform-modules//network/subnet?ref=v3.0.3"

        name                 = "appservices-subnet"

        resource_group_name  = **var.vnet_rg**
        virtual_network_name = **var.vnet_name**
        address_prefixes     = [var.address_prefix]

        subnet_delegation = [{
            name            = "delegation"
            service_name    = "Microsoft.Web/serverFarms"
            service_actions =["Microsoft.Network/virtualNetworks/subnets/action"]
        }]
    }

    module vm_diagnostics_storage_account {
        source = "git::ssh://git@ssh.dev.azure.com/v3/organization/project_name/terraform-modules//storage/storage-account?ref=v3.0.1"

        name                = "adsdevvmdiagstg"
        resource_group_name = module.resource_group.name
    }

We can see that each Module starts with a [**source** property](https://www.terraform.io/docs/modules/sources.html) that references an [external git repository](https://www.terraform.io/docs/modules/sources.html#generic-git-repository). In our case, we are referencing a git Repo stored in Azure DevOps. We authenticate to this [Repo using an SSH key](https://docs.microsoft.com/en-us/azure/devops/repos/git/use-ssh-keys-to-authenticate?view=azure-devops&tabs=current-page) which allows Windows and Mac users to work with the Repo without any code changes. The company/project_name parts help Azure DevOps resolve requests to our Organization. The terraform-modules part of the URL is the name of the Repo we want to work with in the Azure DevOps Project. The double // part might look wrong but I assure you is actually quite correct, at least for Azure DevOps. I honestly do not remember how we stumbled onto this either, I tried to find a link to this but was unable.

The last part of the URL is the **?ref=v3.0.X** part. The ref querystring parameter allows us to reference a specific branch or tag in a git Repo.

The **variables** file contains all the local variables that are required for the Module.

    variable vnet_rg {}

    variable vnet_name {}

    variable address_prefix {
        default = "172.26.109.64/28"
    }

    variable sql_admin_password {}

Any variables defined in this file that do not have default values will need to get their values from somewhere, and we generally pass those variables in from the Subscription Folder’s **main** file, as seen earlier.

The **output** file contains any properties that we want to export from the Resource Group Module. These are generally used when a Resource Group folder contains a Resource that is shared across a given Subscription.

    output vm_diagnostics_storage_primary_blob_endpoint {
        value = **module.vm_diagnostics_storage_account.primary_blob_endpoint**
    }

For instance, all of our Virtual Machines dump their Diagnostics Logs into a common Blob Storage Account. The URL for this Storage Account is required when configuring a [Virtual Machines Diagnostics Settings](https://www.terraform.io/docs/providers/azurerm/r/windows_virtual_machine.html#storage_account_uri), so we output that property in the Resource Group Module. By outputting the property, we are able to pass that value as a variable to another Module which requires it.

The **rbac** file is where we store all of the permissions we have applied at the Resource Group level. We do not scope permissions at the Subscription level in our Azure tenant, rather, we only scope permissions at the Resource Group level, or lower.

An example of one of our **rbac** files is as follows;

    data azuread_group security_group {
        name = "Sg_AFI_Role_TransportationDev_Unv"
    }

    module reader_role_assignment {
        source = "git::ssh://git@ssh.dev.azure.com/v3/ashleyfurniture/Infra-Cloud/terraform-modules//authorization/default-roles-dev?ref=v3.0.0"

        scope        = **module.resource_group.id**
        principal_id = **data.azuread_group.security_group.id**
    }

In this example, we are using the azuread provider to get a reference to an existing Active Directory Security Group. We then pass the ID of this Security Group to one of our custom Modules — default-roles-dev. This Module sets a collection of Permissions to the scope provided via the variable. In this case, the scope is the ID of the Resource Group itself. Normally, we keep all of our data sources in the variables file, with this one exception. We keep all of the RBAC settings together in one file so that it is easier to understand what permissions are in place.

That, in a nutshell, is the gist of the **terraform-azure** Repo. There was a lot of information there, and I thank you for reading up to this point.

## Benefits, Issues, and Closing Thoughts

### Benefits

This setup has allowed us to scale-up and increase the number of active contributors very smoothly. We started out as a team of two people working with Terraform and now, three years later, we have more than 20 active contributors at our company. We have brought numerous development teams into the cycle and actively encourage them to define their own Resources in our Terraform code. Due to the sensitivity of the State File, and the lack of Permission at the Subscription level, we do not let developers deploy their own Resources to Azure.

This pattern has also allowed us to greatly reduce the amount of time it takes our Teams to deploy new Resources to Azure. Using these two codebases, we can reliably deploy new Virtual Machines to Azure simply by copy-pasting an existing Virtual Machine and changing a few variable values. Since most software Solutions tend to look like other Solutions, we can copy-paste-deploy an entire stack in a few minutes as well, all we need to do is copy an existing Resource Group Folder that “looks” like the Solution we are about to deploy.

We adopted Azure way before we implemented Infrastructure as Code. We currently have more than 8000 Resources deployed to our Azure Tenant, and over 2500 of those Resources are defined in Terraform. Even with this volume of Resources defined in Terraform, we can still run the **init\plan\apply** cycle in a few short minutes. We continuously import existing Resources into our terraform-azure codebase and expect by the end of the year that more than 70% of all Azure Resources will be defined in Terraform.

### Issues

It isn't all peaches and roses, and there are some issues with this approach.

The primary issue with this approach is that changes to the Provider are not automatically available to Consumers of the Provider. This is due to the fact that the Module Library sits between the Terraform Provider, and the Consumer (terraform-azure). This is, in reality, a pro and a con at the same time. While we do not benefit from day-one bug fixes and other updates, this pattern also shields us from day-one bugs that might be introduced in the Provider. The other issue is that a Consumer who wants to deploy a new Resource to Azure that is **not** defined in our Module Library — would have to develop that Module before they can do their deployment. This problem occurs much less frequently nowadays though since we have covered most of the Resources that most people would want to consume.

Another issue with this pattern is that a **terraform init** downloads numerous copies of the terraform-modules Repo. This is mostly due to the way that Terraform organizes Modules in the **.terraform** folder, as can be seen in the following image.

![git Repo Downloaded Multiple Times](https://cdn-images-1.medium.com/max/2000/1*QjIJUgGX6LgUcF0gZEJ1AQ.png)*git Repo Downloaded Multiple Times*

In this example, in the “ads-website-rg” Resource Group folder, there are two Application Insights Resources defined; **ads_website_application_insights** and **ads_website_services_application_insights**, both of which are actually pinned to the same Module Tag. When Terraform downloaded the code from the git Repo, it downloaded the whole Repo twice, even though it only needed one folder from the Repo. In a small code base, this wouldn't be a huge problem. In our codebase though, this adds up.

![Treesize of terraform-azure codebase on the local machine](https://cdn-images-1.medium.com/max/2000/1*VKTqZm33aOWtbpkypQmp4A.png)*Treesize of terraform-azure codebase on the local machine*

An additional issue comes up when developing in the **terraform-modules** Repo, and it is related to the previous issue. Module changes are tested by implementing those Modules in the terraform-azure Repo by simply passing the branch name in the ?ref= querystring parameter. This is perfectly fine — until you realize there is a bug in your Module Definition. When this happens, you have to find the Module Code in the .terraform folder and delete it in order to test your bug-fixes. This happens because Modules are only downloaded once when you run **terraform init**, and re-used locally from that point forward.

The final issue that comes up, mostly in a Team Environment, has to do with [State File Locking](https://www.terraform.io/docs/state/locking.html). Whenever a user performs a **plan** or an **apply**, the State File is locked in Azure Blob Storage, ensuring that only one user is modifying infrastructure at a given point in time. This is a good thing, but it can cause some issues.

The issue it can cause is if two people try to work on a given Subscription at the same time, one of the two people may encounter a State Lock error, and be unable to proceed until the Lock is removed. Since the Lock only lasts as long as the plan or apply, this usually isn't a huge problem.

![Locked State File Exception](https://cdn-images-1.medium.com/max/2558/1*Ndadhjy0vFotfXNzmS50Lg.png)*Locked State File Exception*

Remember that I mentioned that we always run Terraform from within the context of a Subscription Folder. We do this for a few reasons. One reason is to ensure that only one person is changing anything in a given Subscription. Another reason is that running a plan or apply from the root of the project would end up locking the State File for a much longer duration of time, thereby increasing the chances that other team members would be affected.

### Closing Thoughts

If you have read this far, I really do appreciate your time. There was a lot of ground to cover and I try to be as clear as I can in my writing. Hopefully, you found this article helpful and worth your time.

Terraform is an extremely powerful tool that makes working with Infrastructure easy and consistent. Once you commit to learning HCL, you will be able to write code in a consistent manner and target numerous Cloud & Infrastructure platforms alike. Whatever role you find yourself in, learning Terraform will definitely help you in your career.

Terraform allows you to organize your code however you see fit. As you grow in your adoption of Terraform and Infrastructure as Code, the complexity of your codebase will grow as well. Refactoring a sprawling codebase can be very challenging, and is best avoided for as long as possible. Hopefully, this article has given you some ideas about how you can organize your codebase so that it is easy to reason about, easy to work with, and maintainable for a long period of time.
