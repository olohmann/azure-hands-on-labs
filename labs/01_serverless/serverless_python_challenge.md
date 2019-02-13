# Serverless with Azure Functions - Python Challenge

## Overview

The newest addition to Azure Functions is support for Python. In this lab you are challenged to run through the same exercise as in the [.NET guided version of the lab](./serverless.md).

That is, you will create an Azure Function that monitors a blob container in Azure Storage for new images, and then performs automated analysis of the images using the Microsoft Cognitive Services [Computer Vision API](https://www.microsoft.com/cognitive-services/en-us/computer-vision-api). Specifically, The Azure Function will analyze each image that is uploaded to the container for adult or racy content and create a copy of the image in another container. Images that contain adult or racy content will be copied to one container, and images that do not contain adult or racy content will be copied to another. In addition, the scores returned by the Computer Vision API will be stored in blob metadata.

The following illustration provides an overview:
![Overview](./media/overview.png)

## Setup your Python Environment in the Azure Shell

We need to setup some CLI tooling to get you started with Azure Functions for Python. In order to keep your local environment vanilla, we are going to setup the Azure Shell for you. If you have already a local Python environment, feel free to skip this step and setup your local environment instead. You can follow [this guide](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-function-python) to do so.

### Step 1: Install the Azure Functions CLI tool

1. Open the Azure Shell.
1. Issue the following series of commands to install the Azure Functions CLI Tool (`func`):

    ```sh
    mkdir "${HOME}/.npm-packages"
    echo 'prefix=${HOME}/.npm-packages' >> ${HOME}/.npmrc
    echo 'NPM_PACKAGES="${HOME}/.npm-packages"' >> ${HOME}/.bashrc
    echo 'export PATH="$NPM_PACKAGES/bin:$PATH"' >> ${HOME}/.bashrc
    echo 'unset MANPATH' >> ${HOME}/.bashrc
    echo 'export MANPATH="$NPM_PACKAGES/share/man:$(manpath)"' >> ${HOME}/.bashrc
    source ${HOME}/.bashrc
    npm install -g azure-functions-core-tools
    ```

1. Type `func`. This should print an output similar to this:

    ```sh

                  %%%%%%
                 %%%%%%
            @   %%%%%%    @
          @@   %%%%%%      @@
       @@@    %%%%%%%%%%%    @@@
     @@      %%%%%%%%%%        @@
       @@         %%%%       @@
         @@      %%%       @@
           @@    %%      @@
                %%
                %

    Azure Functions Core Tools (2.4.317 Commit hash: 63e1996b6c281079427e7c9b8a0a1c633988a195)
    Function Runtime Version: 2.0.12285.0
    Usage: func [context] [context] <action> [-/--options]

    <... many additional lines... >
    ```

### Step 2: Setup pyenv to get a dedicated local Python version

1. Again, open the Azure Shell or continue working in the current instance.

1. Issue the following series of commands to install [pyenv](https://github.com/pyenv/pyenv):

    ```sh
    git clone https://github.com/pyenv/pyenv.git ${HOME}/.pyenv
    echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ${HOME}/.bashrc
    echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ${HOME}/.bashrc
    echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ${HOME}/.bashrc
    source ${HOME}/.bashrc
    ```

1. Type `pyenv`. The output should look like this:

   ```sh
   pyenv 1.2.9-2-g6309aaf
   Usage: pyenv <command> [<args>]
   ```

1. Install the `virtualenv` plugin for `pyenv`:

    ```sh
    git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
    echo 'eval "$(pyenv virtualenv-init -)"' >> ${HOME}/.bashrc
    source ${HOME}/.bashrc
    ```

1. Now setup a local Python version 3.6.8 with venv support:

    ```sh
    pyenv install 3.6.8
    pyenv shell 3.6.8
    pyenv virtualenv venv368 # creates a new virtualenv from the current version
    pyenv activate venv368
    ```

    > Everytime you fire up a new Azure Shell instance, you need to activate the environment again with `pyenv activate venv368`.

2. Verify that you have the following output - especially the change in prompt (might change in future versions):

    ```sh
    pyenv-virtualenv: prompt changing will be removed from future release. configure `export PYENV_VIRTUALENV_DISABLE_PROMPT=1' to simulate the behavior.
    (venv368) labuser01@Azure:~$
    ```

3. Type `python --version` and you should see:

    ```sh
    Python 3.6.8
    ```

### Step 3: Setting up your Function Boilerplate

1. Run the following commands which should start the Azure Functions CLI wizard:

    ```sh
    mkdir ${HOME}/az-func-lab
    cd ${HOME}/az-func-lab
    func init
    ```

1. In the func wizard, select `python (preview)`. This will install python packages to the venv.

1. Run `func new` and select `Azure Blob Storage` trigger. Name it `ClassifyImage`. Let the installation finish.

1. Take a look at the generated output by issuing the command `ll`. Feel free to use `code .` to inspect all files more closely.

### Step 4: Creating a new Function App Instance in Azure 

1. Replace the name, location, app insights and resource group variables according to your setup. Make sure to use a unique name for the storage account and function app name.

    ```sh
    export STORAGE_ACCOUNT_NAME=labuser01pyfn
    export LOCATION=westeurope
    export RESOURCE_GROUP_NAME=RG001
    export FN_APP_NAME=labuser01pyfn
    export APP_INSIGHTS_NAME=labuser01pyfn
    ```

1. Create the Azure resources:

    ```sh
    az storage account create --name ${STORAGE_ACCOUNT_NAME} --location ${LOCATION} --resource-group ${RESOURCE_GROUP_NAME} --sku Standard_LRS
    az resource create \
        --resource-group ${RESOURCE_GROUP_NAME} \
        --resource-type "Microsoft.Insights/components" \
        --name ${APP_INSIGHTS_NAME} \
        --location ${LOCATION} \
        --properties '{"ApplicationId":"pyfn","Application_Type":"other", "Flow_Type":"Redfield", "Request_Source":"IbizaAIExtension"}'
    az resource show -g ${RESOURCE_GROUP_NAME} -n ${APP_INSIGHTS_NAME} --resource-type "Microsoft.Insights/components" --query properties.InstrumentationKey
    INSTRUMENTATION_KEY=$(az resource show -g ${RESOURCE_GROUP_NAME} -n ${APP_INSIGHTS_NAME} --resource-type "Microsoft.Insights/components" --query properties.InstrumentationKey -o tsv)
    az functionapp create --resource-group ${RESOURCE_GROUP_NAME} --os-type Linux --consumption-plan-location ${LOCATION} --app-insights ${APP_INSIGHTS_NAME} --app-insights-key ${INSTRUMENTATION_KEY} --runtime python --name ${FN_APP_NAME} --storage-account ${STORAGE_ACCOUNT_NAME}
    ```

1. Take a look into your assigned resource group. It should look like this:

    ![Python Function App](./media/py-fn-app-rg-view.png)

### Step 5: Publish your function

1. Make sure you carry over your function app name, e.g.:

    ```sh
    export FN_APP_NAME=labuser01pyfn
    ```

1. Issue this commands to publish your function

    ```sh
    cd ${HOME}/az-func-lab
    func azure functionapp publish ${FN_APP_NAME}
    ```

    > If you see an error, make sure that your pyenv is set correctly. When the Azure Shell is closed and reopened, a new bash session is started. This way you loose the pyenv configuration, too. Just issue `pyenv activate venv368` to fix that.

1. Expect output like this:

    ```sh
    Getting site publishing info...
    pip download -r /home/oliver/az-func-lab/requirements.txt --dest /tmp/azureworkerb7ae3m95

    [...]

    Preparing archive...
    Uploading 76.53 MB [##############################################################################]
    Upload completed successfully.
    Deployment completed successfully.
    Syncing triggers...
    Functions in labuser01pyfn:
    ClassifyImage - [blobTrigger]
    ```

## Build the Solution

Now it is on you to build up the source code to classify images! Take the [.NET version](./serverless.md) of the lab as guidance.

### Hint 1

...

### Hint 2

...