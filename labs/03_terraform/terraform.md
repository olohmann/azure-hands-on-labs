# Introduction to Terraform

## Overview

Let us start with a quick definition from Wikipedia:

> Terraform is an infrastructure as code software by HashiCorp. It allows users to define a datacenter infrastructure in a high-level configuration language, from which it can create an execution plan to build the infrastructure [...]. [Wikipedia](https://en.wikipedia.org/wiki/Terraform_(software))

Building up the infrastructure with Terraform can happen in many environments. One of them is Azure. In this lab you are going to explore Terraform provider for Azure.

This lab is not providing you *copy&paste ready code*. Instead, you have to find the solution yourself and use the terraform documentation. There are also tons of hints to solve the challenges.

## Preparation

This lab assumes that you have a resource group assigned to you. If not, please create a resource group before you start with the exercise.

Make use of the [Terraform Azure Provider Documentation](https://www.terraform.io/docs/providers/azurerm/index.html) to solve the challenges.

## Challenge 1: Get familiar with the Terraform Loop

1. Create a Storage Account via Terraform in your resource group. Hints:
   - Documentation for [Azure Storage Account Terraform Resource](https://www.terraform.io/docs/providers/azurerm/r/storage_account.html)
   - [terraform plan command](https://www.terraform.io/docs/commands/plan.html)
   - [terraform apply command](https://www.terraform.io/docs/commands/apply.html)

1. Add a tag to your deployment and issue a new deployment.
1. Detect *configuration drift* by modifying the tag of your storage account in the Azure portal and re-running the Terraform deployment. Hint: look at the [terraform plan](https://www.terraform.io/docs/commands/plan.html) output to see the drift.
1. Update the resource in Azure with terraform to reverse the configuration drift.
1. Destroy the created resource with Terraform. Hint: [terraform destroy command](https://www.terraform.io/docs/commands/destroy.html)

## Challenge 2: Introduce Variables, create resources with dependencies and use Data Sources

1. Provide the name of the resource group that you want to deploy your storage into as a *variable* to the terraform deployment. Hints:
   - [Terraform Variables](https://www.terraform.io/docs/configuration/variables.html).
   - Terraform takes all input files within a directory. That is you can split your code in multiple files. Best practices for variables: create a file like 'variables.tf' and put your variable declaration into that file.

1. Create a *container* (comparable to a sub-folder) inside the storage account. Think about how you reference the storage account. Just by name (a simple string)? What would be the consequence? Is there a better way?

1. Most likely you hard-coded the location by setting the field as a string in the storage account properties. Coincidentally, this is perhaps the location of the resource group? If so - great. This is a best practice.
But how to make it even more transparent? Can't you just reference your already existing resource group? Similar to the reference you used between the storage account the storage container? Yes, you can! With [Data Sources](https://www.terraform.io/docs/configuration/data-sources.html). Hint: [Resource Group Data Source](https://www.terraform.io/docs/providers/azurerm/d/resource_group.html)

## Challenge 3: Use Terraform Utility Functions and generate Output

Think about a typical challenge with storage accounts and other multi-tenant resources: **getting a unique name**. Reason: they become a publicly listed hostname and hostnames have to be unique. How can you achieve that with Terraform?

1. Create the storage account with same approach as in the first challenge or just continue in your sources. Take the input variable for the storage account name as a prefix and concatenate a (pseudo-)unique suffix. Hints:
   - Look into [locals](https://www.terraform.io/docs/configuration/locals.html) to introduce local calculated values.
   - Maybe hashing something that is unique can help? Look into [interpolations](https://www.terraform.io/docs/configuration/interpolation.html).
   - Or instead of hashing maybe a random value generator can help? [Random Provider](https://www.terraform.io/docs/providers/random/index.html). Think about advantages disadvantages of random vs hash.
1. Generate a (sensitive) output that return the storage account's connection string. Hints:
   - [Terraform Output](https://www.terraform.io/docs/configuration/outputs.html)
1. Use the `terraform output` command to print the information as JSON. Interesting, right? Although we won't do anything with that JSON now, it gives you an idea how this output can be fed into other tools or systems.
1. Destroy everything and come back to a state *where no resource* is located in your Resource Group.

## Challenge 4: Combine Multiple Resources to build a VM

You will deploy a Virtual Machine in this challenge. As you might know, an Azure VM consists of multiple elements. The great thing about Terraform is that you can build up things incrementally. So in each step, feel free to deploy the intermediary state by running a *terraform apply*. When you look at the terraform documentation you can basically get a copy and paste ready solution. Feel free to copy over the resource config, but do it step by step to gain an understanding what is actually happening.

1. Create a fresh new folder for this task.
1. Start by setting up a virtual network with a subnet. You can choose any private IP range.
1. Next, create a Public IP address and Network Interface Card (NIC) resource. Make sure the NIC is registered in the previously created subnet.
1. Create a Linux VM resource linking it to the NIC. Make sure to use a small sized VM (1 vCPU, e.g. `Standard_D1_v2`).
1. Test to connect the the VM via the configured username/password or SSH key.
1. Finally, destroy everything. Re-creating all resources would now as simple as just go through a new plan/deploy cycle. That is, you should now be in a state *where no resource* is located in your Resource Group.

## Challenge 5: Doing deployments in Cookie Cutter Style

Imagine you need to deploy a world-wide used application on Azure. For example, you want to have a frontend available in US, Europe and Asia Pacific. And tomorrow you might need another instance in South Africa. In the best sense of the programming paradigm [Don't Repeat Yourself](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself), it is strongly discouraged to copy and paste three or four versions of your Azure resources. Instead, you should only take e.g. the locations as an input.

1. Create a fresh new folder for this task.
1. Copy the following snippet to your `main.tf` for a start:

   ```tf
    variable "locations" {
        type = list(string)
        default = ["westeurope", "westus"]
    }

    output "locations" {
        value = var.locations
    }
   ```

1. Now use Terraform's [count](https://www.terraform.io/docs/configuration/interpolation.html) to deploy a Web Application (and its required App Service Plan) in each of those listed regions. You can use the following config for the SKU to use free tier resources:

   ```hcl
    sku {
        tier = "Free"
        size = "F1"
    }
   ```

1. Now add a new region to the list. Either via tweaking the default value or via overriding the variable on the terraform command line. Hint: You get a list of Azure DC name via: `az account list-locations --query '[].name'`. A new instance should be created.

1. Destroy everything.
