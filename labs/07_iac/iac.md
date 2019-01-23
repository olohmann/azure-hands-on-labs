# Introduction to Infastructure as Code (IaC) for Azure

## Overview

We will have one development environment and one "production" environment, so that we will be able to safely test any change in an isolated environment that is as structurally similar to production as possible. This similarity is key to not experiencing production releases that fail badly because some detail in production was different from what we tested against.

In the past, keeping up the similarity of environments often was a matter of human discipline: Admins carefully documented all configuration steps they applied to any environment, so they could apply them to the other environment in the same way. Yet just like all processes that rely on human discipline, this approach is susceptible to human error - forgetting just one little detail can break the whole thing.

A good approach to solving this problem is called [Infrastructure as Code (IaC)](https://docs.microsoft.com/en-us/azure/devops/learn/what-is-infrastructure-as-code). In that approach, changes to an environment are always performed through scripts and templates that are version controlled and require no human interaction other than passing in some parameters. Implementing IaC is much easier for cloud environments than for traditional environments because **everything** (including all networking) in the cloud is *software-defined*, meaning that everything can be configured through an API. For Azure, that API is called Azure Resource Manager (ARM) and with [ARM templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates) we can easily create our two environments in exactly the same way.

here simple. Start points for more complex in [Azure Quickstart Templates](https://azure.microsoft.com/en-us/resources/templates/) ([github version](https://github.com/Azure/azure-quickstart-templates))

### Objectives

In this hands-on lab, you will learn how to:

- Generate a basic [ARM template](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates) from an existing Azure deployment
- Deploy ARM templates with parameters using [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest
)
- Use ARM [Template Functions](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-template-functions) to create dynamic, repeatable templates and reduce the number of needed parameters
- Initialize Terraform with the [Azure Provider](https://www.terraform.io/docs/providers/azurerm/) and create a simple Azure deployment
- Use Terraform [variables](https://www.terraform.io/docs/configuration/variables.html) to parameterize an Azure deployment
- Use Terraform [modules](https://www.terraform.io/docs/modules/index.html) to reuse configuration parts
- Use Terraform [remote state](https://www.terraform.io/docs/state/remote.html) to enable team collaboration

### Prerequisites

Typically these should be preconfigured for you (if in doubt, ask your instructor):
* An active Azure subscription or at least two resource groups (one for "dev", one for "prod") to which you have owner permissions.
* A modern web browser (Edge, Chrome or Firefox preferred, Internet Explorer 11 to some degree) with internet access.
* **Optional**: Visual Studio Code with the ARM extension and Terraform extension installed.
---

Estimated time to complete this lab: **120-180** minutes.

## Exercise 1: Log on to the Azure Cloud Shell

1. Open the [Azure Portal](https://portal.azure.com), log on with your lab account, if necessary. You can see the currently logged on account in the top right of the portal:

    ![Logged on account in Azure portal](./media/portalloggedonuser.png)

    If this is not the lab user that was provided to you, please start a new "In Private" or "Incognito" window and start the Azure portal again.

1. Start the Azure Cloud Shell (Bash) by clicking the console icon in the top bar of the portal:

   ![Start Azure Cloud Shell](./media/portalopenconsole.png)

    In case you have not worked with the Azure Cloud Shell before, you will be asked a few questions. Click **Bash (Linux)** and **Create Storage**, accept all defaults. Your console should then look like this:

   ![Ready Azure Cloud Shell](./media/portalconsoleready.png)

---
**Tip:** *You can open another instance of cloud shell by starting a new browser tab or window and navigating to [https://shell.azure.com](https://shell.azure.com). This way you have more space and you can easily switch between the portal and the cloud shell.*

---

## Exercise 2: Create something simple using the Azure CLI

We will start by creating something simple in Azure manually. What **exactly** we are creating does not matter at this point. We just need something to create a template from, which we can use subsequently to create copies of our simple resources.

The simple "environment" we create will simply consist of two [storage accounts](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-overview).

1. In the cloud shell, type:
    ```sh
    az storage account create -n <account name> -g <resource group>  
    ```