# Introduction to Terraform

## Overview

Let us start with a quick definition from Wikipedia:

> Terraform is an infrastructure as code software by HashiCorp. It allows users to define a datacenter infrastructure in a high-level configuration language, from which it can create an execution plan to build the infrastructure [...]. [Wikipedia](https://en.wikipedia.org/wiki/Terraform_(software))

Building up the infrastructure with Terraform can happen in many environments. One of them is Azure. In this lab you are going to explore Terraform provider for Azure.

## Preparation

This lab assumes that you have a resource group assigned to you. If not, please create a resource group before you start with the exercise.

Make use of the [Terraform Azure Provider Documentation](https://www.terraform.io/docs/providers/azurerm/index.html) to solve the challenges.

## Challenge 1: Get familiar with the Terraform Loop

1. Create a Storage Account in your resource group. Hints: [Terraform Azure Storage Account](https://www.terraform.io/docs/providers/azurerm/r/storage_account.html), [terraform plan](https://www.terraform.io/docs/commands/plan.html) command and [terraform apply](https://www.terraform.io/docs/commands/apply.html) command.
1. Add a tag to your deployment and issue a new deployment.
1. Detect configuration drift by modifying the tag of your storage account in the Azure portal and re-running the Terraform deployment. Hint: look at the [terraform plan](https://www.terraform.io/docs/commands/plan.html) output.
1. Update the resource in Azure with terraform to reverse the configuration drift.
1. Destroy the created resource with Terraform. Hint: [terraform destroy](https://www.terraform.io/docs/commands/destroy.html)

## Challenge 2: Use Terraform Utility Functions

1. Think about a typical challenge with storage accounts and other multi-tenant resources: **getting a unique name**. How can you achieve that with Terraform? Hint: Look into [interpolations](https://www.terraform.io/docs/configuration/interpolation.html).
1. Create the storage account with the unique name (same approach as in the first challenge).
1. Finally, destroy the created resource.

## Challenge 3: Combine Multiple Resources

You will deploy a Virtual Machine in this challenge. As you might know, a VM consists of multiple elements. The great thing about Terraform is that you can build up things incrementally. So in each step, feel free to deploy the intermediary state.

1. Start by setting up a virtual network with a subnet. You can choose any private IP range.
1. Next, create a Public IP address and Network Interface Card (NIC) resource. Make sure the NIC is registered in the previously created subnet.
1. Create a Linux VM resource linking to the NIC.
1. Test to connect the the VM via the configured username/password or SSH key. To avoid setting up any corporate proxy for the connection, you can use the [Azure Cloud Shell](https://shell.azure.com/) to open up an SSH connection to the newly created VM.
1. Finally, destroy everything. Re-creating all resources is now as simple as just go through a new plan/deploy cycle.

## Challenge 4: Cookie Cutter

1. Use Terraform's [count](https://www.terraform.io/docs/configuration/interpolation.html) feature to create more than one VM.
1. Validate it, e.g. by connecting via SSH.
1. Finally, destroy it.
