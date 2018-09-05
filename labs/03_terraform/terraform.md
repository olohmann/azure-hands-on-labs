# Introduction to Terraform

## Overview

Let us start with a quick definition from Wikipedia:

> Terraform is an infrastructure as code software by HashiCorp. It allows users to define a datacenter infrastructure in a high-level configuration language, from which it can create an execution plan to build the infrastructure [...]. [Wikipedia](https://en.wikipedia.org/wiki/Terraform_(software))

Building up the infrastructure with Terraform can happen in many environments. One of them is Azure. In this lab you are going to explore Terraform provider for Azure.

## Preparation

This lab assumes that you have a resource group assigned to you. If not, please create a resource group before you start with the exercise.

Make use of the [Terraform Azure Provider Documentation](https://www.terraform.io/docs/providers/azurerm/index.html) to solve the challenges.

## Challenge 1: Get familiar with the Terraform Loop

1. Create a Storage Account in your resource group.
1. Add a tag to you deployment and issue a new deployment.
1. Detect configuration drift by modifying the tag of your storage account in the Azure portal and re-running the Terraform deployment.
1. Think about a typical challenge with storage accounts and other multi-tenant resources: **getting a unique name**. How can you achieve that with Terraform? Hint: Look into [interpolations](https://www.terraform.io/docs/configuration/interpolation.html).
1. Finally, destroy the created resource.

## Challenge 2: Combine Multiple Resources

You will deploy a Virtual Machine in this challenge. As you might know, a VM consists of multiple elements. The great thing about Terraform is that you can build up things incrementally. So in each step, feel free to 

1. Start by setting up a virtual network with a subnet. You can choose any IP ranges.
1. Next, create a Public IP addres and Network Interface Card (NIC) resource. Make sure the NIC is registered in the previously created subnet.
1. Finally, create a Linux VM resource.

## Challenge 3: Cookie Cutter

1. Use Terraform's [count](https://www.terraform.io/docs/configuration/interpolation.html) feature to create more than one VM.
