# Introduction to Infastructure as Code (IaC) for Azure

## Overview

One of the most common pitfalls in bringing applications to production is when our production environment is configured slightly different than the environments we tested in. Thus, we always want to be able to safely test any change to our application in an isolated environment that is as structurally similar to production as possible. This similarity is key to not experiencing production releases that fail badly because some detail in production was different from what we tested against.

In the past, keeping up the similarity of environments often was a matter of human discipline: Admins carefully documented all configuration steps they applied to any environment, so they could apply them to the other environment in the same way. Yet just like all processes that rely on human discipline, this approach is susceptible to human error - forgetting just one little detail can break the whole thing.

![ProdConfigBad](./media/prodconfigbad.png)

A good approach to solving this problem is called [Infrastructure as Code (IaC)](https://docs.microsoft.com/en-us/azure/devops/learn/what-is-infrastructure-as-code). In that approach, changes to an environment are always performed through scripts and templates that are version controlled and require no human interaction other than passing in some parameters. This approach makes sure 

![IaC Pipeline](./media/iac-pipeline.png)

Implementing IaC is much easier for cloud environments than for traditional environments because **everything** (including all networking) in the cloud is *software-defined*, meaning that everything can be configured through an API.

For Azure, the main configuration API is called Azure Resource Manager (ARM), available at https://management.azure.com/. In theory, we could provision all our Azure resources by working with that REST API directly, yet that is neither conventient nor efficient in most cases.

Instead, the two most useful and popular techniques for implementing IaC with Azure, are

* [ARM templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates) and
* [Terraform](https://www.terraform.io/)

Both of which we will explore in this lab.

here simple. Start points for more complex in [Azure Quickstart Templates](https://azure.microsoft.com/en-us/resources/templates/) ([github version](https://github.com/Azure/azure-quickstart-templates))

Introduce Idempotency as the gold standard for repeatability and maintainability

### Objectives

In this hands-on lab, you will learn how to:

- Automate Azure by using the Azure Command Line Interface 2.0 ([Azure CLI](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest)) on Windows or Linux
- Generate a basic [ARM template](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates) from an existing Azure deployment
- Deploy ARM templates with parameters using [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest)
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

## Exercise 2: Create resources with the Azure CLI

We will start by creating something simple in Azure from the the command line, using the cross-platform [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest). The Azure CLI can be universally used to provision and configure Azure resources where we have a Linux or Windows shell available.

One possible approach to enabling IaC for Azure thus is simply creating scripts in PowerShell or bash that contain Azure CLI commands that create the desired resources one by one. [Azure PowerShell modules](https://docs.microsoft.com/en-us/powershell/azure/overview?view=azps-1.1.0) are available as well to create such script based provisioning of resources.

We will not explore a purely script based approach in this lab though, because shell scripts are always **imperative**. To be able to reach our goal of **idempotency** (see section *Overview* above), an imperative script will always need to actively make sure it will work with the current state of our environment. For example, it will need to check whether a resource exists, before trying to create it, to make sure the script will not fail when we apply it a second time. This checking of current state to enable idempotency requires a lot of conditionals and "overhead" code that obfuscates our original intents and is hard to maintain. Thus, we prefer **declarative** approaches over imperative ones. In declarative approaches we simply declare our desired state and some engine then checks the actual state and determines the actions needed to reach the desired state. This represents a much easier and concise approach to IaC, as well requiring less maintenance once the initial configuration has been established.

Both ARM Templates and Terraform are such declarative approaches and will be explored later on.

For now we will simply use the Azure CLI to create **something** as a start point. What **exactly** we are creating does not matter at this point. We just need something to create a template from, which we can use subsequently to create copies of our simple resources.

The simple "environment" we create will simply consist of two [storage accounts](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-overview).

1. In the cloud shell, type:
    ```sh
    az storage account create -n <account name 1> -g <resource group>  
    az storage account create -n <account name 2> -g <resource group>  
    ```

    ...where `<resource group>` is the name of the resource group you were provided with by your instructor and `<account name 1>` and `<account name 2>` are two names you can freely choose but that must be still available globally. The client will tell you whether they are free, choose another name otherwise.

