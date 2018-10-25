# Introduction to Containers on Azure

## Overview

Containers are revolutionizing the way how software is being deployed. This lab will provide the foundation for working with containers and enable participants to deploy single container applications to the Azure cloud. The lab will start with a general introduction to the Docker environment, then show how applications can be containerized using so-called Dockerfiles and published to Azure Container Registry (ACR). Finally, we will be deploying the dockerized application to Azure Container Instances (ACI) and Azure Web Sites.

### Why containers?

All too often, differences between our development environments ("runs on my machine!") and production environments lead to problems on releasing our app: version conflicts, subtle configuration issues, noisy neighbor applications taking all resources.

   ![Runs on my machine](./media/runsonmymachine.png)

Containers help solve this problem by virtualizing the interfaces the app has to the operating system (and thus to the rest of the world), like network calls, the file system, and so on.

   ![Containers virtualize the OS (not the hardware)](./media/virtualizedos.png)

This way, if an app runs in my container, it can run anywhere, because the world looks the same to it, regardless where the container is. It can always listen on its favorite network port, has all its libraries with it and all configuration files are tailored to exactly its needs.

   ![Runs on my machine](./media/runanywhere.png)

### Objectives

In this hands-on lab, you will learn how to:

- Use Docker to run containers
- Containerize applications with Dockerfiles and docker build
- Push containerized applications as Container Images to Azure Container Registries (ACR) 
- Deploy containerized applications to Azure Container Instances (ACI) and Azure Websites.

### Prerequisites

Typically these should be preconfigured for your (if in doubt, ask your instructor):
- An active Azure subscription or resource group to which you have contributor permissions.
- An Ubuntu Linux machine running in Azure with access to the cloud and Docker installed ([installation instructions](https://docs.docker.com/install/linux/docker-ce/ubuntu/)).

---

Estimated time to complete this lab: **120-180** minutes.

## Exercise 1: Log on to your VM

 ![Exercise 1 Goal](./media/exercise1.png)

1. Open the [Azure Portal](https://portal.azure.com), log on with your lab account, if necessary. You can see the currently logged on account in the top right of the portal:

    ![Loggend on account in Azure portal](./media/portalloggedonuser.png)

    If this is not the lab user that was provided to you, please start a new "In Private" or "Incognito" window and start the Azure portal again.

1. Start the Azure Cloud Shell (Bash) by clicking the console icon in the top bar of the portal:

   ![Start Azure Cloud Shell](./media/portalopenconsole.png)

    In case you have not worked with the Azure Cloud Shell before, you will be asked a few questions. Click **Bash (Linux)** and **Create Storage**, accept all defaults. Your console should then look like this:

   ![Ready Azure Cloud Shell](./media/portalconsoleready.png)

1. Log on to the machine with the user and machine address provided by your instructor, by typing the following command into the cloud shell:

    ```sh
    ssh <user>@<machine adress> 
    ```

    For example:

    ```sh
    ssh labuser29@labuser29-xxxx.westeurope.cloudapp.azure.com
    ```

    (Pasting into the cloud shell will likely require using the browser's context menu. Thus, if it does not work, try a right click into the console with your mouse)

    You will be asked whether to accept this new host (enter **yes**) and for the password (enter the password provided by your instructor).

    After this login succeeds, we have a bash shell running right in the Linux VM, in which we will work with Docker.

1. To test whether Docker is indeed installed type:

    ```sh
    docker --version 
    ```
    This should display the docker version.

## Exercise 2: Run a web server in a container

For a start, we will be running a very powerful and lightweight web server called [nginx](https://www.nginx.com/). This web server is available as a pre-packaged container image at [Docker Hub](https://hub.docker.com/_/nginx/).

 ![Exercise 2 Goal](./media/exercise2.png)

1. We will tell docker to pull the nginx image from Docker Hub and let it run in our docker engine with a single command. Type:

    ```sh
    docker container run --name mynginx -d nginx
    ```

    The `-d` flag in the command tells Docker that the container should run detached, in the background. We do not currently want to interact with it directly. We just want it to run silently with default settings, listening on its default port.

1.  The engine just reports a lengthy id, which does not help us much understanding what is going on. Let's see whether the container actually started. To see all running containers, type: 
    
    ```sh
    docker container ls
    ```
    This returns all running containers with their name and a few more fields. In case you do not find your container in the list, something has gone wrong. To troubleshoot, you would... 
    
    ```sh
    docker container ls --all
    ```

    ..., which lists all non-running containers as well, and then use...

    ```sh
    docker container logs <containername>
    ```

    ..., to find out what the container reported while starting (or trying to start). In our case, the container logs will show nothing yet.

1. The container we just started already serves as a webserver, listening on port 80, but we cannot see its webpage yet. So far, the container is only attached to an internal virtual network called 'bridge', that can only be accessed from other containers on that network and our VM, which is hosting the network and the container. So, let's run another instance of nginx, but now we will map the containers port 80 to an externally visible port on the VM:

    ```sh
    docker container run --name mynginxwithport -d -p 80:80 nginx
    ```

    The `-p 80:80` flag tells docker to map the VM's port 80 to the containers port 80, so that the container now virtually listens on the VM's own port 80. If that port is already reserved by another process on the VM, our call would fail.

1. On your own machine (not the Lab-VM), open the web browser of your choice and navigate to the address of your VM (the same you used to log on the machine in the beginnning) as `http://<machine address>`:

   ![The welcome page served from our container](./media/nginxwelcome.png)

1. **Optional**: In case you want to get a glimpse of how the internal networking works, check the webserver from within the VM: You can find out the container's IP address by inspecting the default docker network 'bridge':

    ```sh
    docker network inspect bridge
    ```

    With that IP address, you can then curl the web servers start page like this:

    ```sh
    curl http://<container ip>
    ```

1. Before moving on we need to clean up, because we will want to reuse port 80, on which the container from this exercise is still listening. (**CAUTION:** This will silently remove ALL running and non-running containers):

    ```sh
    docker container rm -f $(docker container ls -a -q)
    ```

    This command consists of two parts: `docker container ls` within the brackets lists all (`-a` flag) containers with only their IDs (`-q` flag). `docker container rm` then takes all these IDs and removes the containers, even if they are still running (`-f`).

## Exercise 3: Create and containerize a Web App

Now that we know how to run a pre-packaged app from a public container registry like Docker Hub, we would like to see the same for our own applications. We will create a new app from scratch (no coding required) and put that into a container image like the nginx image we used in the preceding exercise.

This exercise is available in two versions. Please choose:

* [Create and containerize a **.NET Core** Web App ](containers_on_azure_3_dotnet.md)
* [Create and containerize a **Java** Web App (with Maven and Tomcat)](containers_on_azure_3_java.md)

## <a name="exercise3b"></a> Exercise 3.b: **Optional**: Multi Stage Build

In the preceding exercise we created our app container by first building and packaging the application by issuing build commands that worked on the file system of the docker host - our 'development machine'. The resulting package then was copied into our container with the `COPY` command. We did this from a container to avoid version conflicts and other configuration issues that might arise in case we relied on the correctness of the dev machine itself.

While it is a good lab exercise to set up volume mounts and working interactively with the shell inside a running container (as we just did), this approach is much too complex and error prone for a real development workflow.

To solve this problem, the building and packaging steps themselves should be defined in a Dockerfile as well.

This exercise is available in two versions. Please choose:

* [Multi Stage Build for **.NET Core**](containers_on_azure_3_b_dotnet.md)
* [Multi Stage Build for **Java** (with Maven and Tomcat)](containers_on_azure_3_b_java.md)

 

## <a name="exercise4"></a> Exercise 4: Create Azure Container Registry (ACR) and push image

So far we only ran the container on our own docker host (our Linux VM). To be able to deploy to other container environments, we need a [registry](https://docs.docker.com/registry/). We will push our image to the registry, so that others can pull it from there. Azure offers a managed service for this, called Azure Container Registry (ACR), which we will use in this lab.

 ![Exercise 4 Goal](./media/exercise4.png)

1. Open the Cloud Shell (in case you are stilled logged in to the VM, just type `exit` and you should be back).

1. Create an ACR with Azure CLI:

    ```sh
    az acr create --name <registry name> --resource-group <resource group> --sku basic
    ```

    Where...
    *  `<registry name>` is a name that you can freely choose, but that **should** consist only of lowercase characters and must still be available as `<registry name>.azurecr.io`.
    *  `<resorce group>` is the name of the resource group that you have contributor permissions to (when in doubt, ask your instructor).

    Now we have our own private registry running in Azure available at `<registry name>.azurecr.io`. We say that it is a **private** registry (although it is running in the public cloud), because it is protected by authentication. By default, access to the registry is restricted to identities that were granted permissions through Azure Active Directory (AAD) and Role Based Access Control (RBAC). Yet for the purpose of this lab, we will use an easier option: A credentials based authentication mechanism called the [Admin account](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-authentication#admin-account). To enable it and get the credentials, use these commands:

   ```sh
    az acr update --n <registry name> -g <resource group> --admin-enabled true
    az acr credential show -n <registry name> -g <resource group>
    ```

    Note the password and username.

1. Log in to the Docker VM again with:

    ```sh
    ssh <user>@<machine adress> 
    ```
    For example:

    ```sh
    ssh labuser29@labuser29-xxxx.westeurope.cloudapp.azure.com
    ```

1. Now we can log in with docker into our registry (replace `<registry name>` with your registry's name):

    ```sh
    docker login <registry name>.azurecr.io
    ```

    You will be prompted for username and password - enter the credentials noted in the previous step.

1. If we are successfully logged in, we can now push our image 'myappimage' (from the previous exercise) to the registry. The registry an image is pushed to, is always encoded in its image tag. Thus, to push to our own registry, we first need to re-tag our image:

    ```sh
    docker image tag myappimage <registry name>.azurecr.io/myappimage:v1.0
    ```

    This command is tagging the image that currently has the name *myappimage* with a new tag containing the registry name. All image names are following this pattern:

    ```txt
    <registry>/<repository>:<tag>
    ```

    Thus, *myappimage* is actually the name of a repository of images, distinguished from each other by tags. In this case, the tag is marking the version of our app, so that we can exactly define which version to pull and run. You will recognize the hierarchy of registry/repository/tag in one of the next steps, after we pushed to the registry.

1. Now we can finally push:

    ```sh
    docker image push <registry name>.azurecr.io/myappimage:v1.0
    ```

1. To see our image in the registry, in the Azure portal, navigate to our newly created registry: Type your registry's name in the search bar at the top of the portal, click it. Click **Repositories** on the left, choose the **'myappimage'** repository and click the **'v1.0'** tag.). There you might want to expand the **Manifest** to see the structure of our app container image. 

Now we are ready to deploy to any Azure service from our registry.

## Exercise 5: Deploy to Azure Container Instances (ACI)

The easiest way to deploy a container to a PaaS service in Azure is to use ACI, which can be seen as "serverless" containers. The Azure platform takes care of everything that needs to be in place in the background, like the container host on which the container will run. We will never need to care about patching or securing the infrastructure and can focus on developing and securing our application itself.

 ![Exercise 5 Goal](./media/exercise5.png)

1. Open the Cloud Shell (in case you are stilled logged into the VM, just type `exit` and you should be back).

1. Look up the credentials for your registry again.

1. The actual deployment itself is done with a single command:

    ```sh
    az container create --resource-group <resource group> --name myapp --image <registry name>.azurecr.io/myappimage:v1.0 --cpu 1 --memory 1 --registry-login-server <registry name>.azurecr.io --registry-username <registry user> --registry-password <registry password> --dns-name-label <some unique name> --ports 80 
    ```

    For the Java deployment, use `--ports 8080` instead of `--port 80` as the Tomcat server is listening per default on port 8080. In addition, `<some unique name>` is the prefix for the public DNS name under which your container will be available.

1. Track the progress of your deployment with:

    ```sh
    az container show --resource-group <resource group> --name myapp --query instanceView.state
    ```
1. Once the state is 'Running', you can look up the full address of your container by running `az container show` again but without the `--query` flag. Look up the `fqdn` field in the output of the command and copy its value. 

1. On your own machine (not the Lab-VM), open the web browser of your choice and navigate to the address you just copied: `http://<fqdn>`. This should show you the same view of our app as in the previous exercise.

1. To troubleshoot, you can use:

    ```sh
    az container logs --resource-group <resource group> --name myapp
    ```
    This will give you the same type of logs as in the previous exercises, when we called `docker logs`, only that this time the container is running in the cloud.

ACI (specifically with the `az container` commands) can be treated as your giant Docker engine in the cloud that is just there, no setup required. This enables great use cases, a few of which are described in the [Azure documentation](https://azure.microsoft.com/en-us/services/container-instances/). For example, any batch process that simply runs in the background (some data but not serving any traffic interactively), would be a perfect fit for ACI. Our sample app though is just a simple web app - and for such single-container web apps there is an even better fit available for running in the cloud: Azure App Service. Which is the topic of the next exercise. 

## Exercise 6: Deploy to Azure App Service

So far in this lab, we looked at our containers in a generic way: Some process needs to run and optionally listen on a specific network port. Yet this might be anything, web, mail, a database engine - on this generic level, any added service (like adding authentication) needs to be connected to our application manually.

Azure App Service is specifically designed for web apps and thus adds many features like simple SSL setup or simple Azure Active Directory integration for authentication that can be turned on by setting simple switches. Azure App Service Web Apps support containers (Linux and Windows) as a deployment vehicle, thus for our simple web app, this is the best fit for letting it run as a PaaS in the cloud.

![Exercise 6 Goal](./media/exercise6.png)

1. First we need to create an App Service Plan (for more info see the [Azure documentation](https://docs.microsoft.com/en-us/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)). An App Service Plan defines the size (and price), available features and more settings for our Web App. In your Cloud shell, execute this command:

    ```sh
    az appservice plan create --name myappserviceplan --resource-group <resource group> --sku B1 --is-linux
    ```

    This creates an app service plan in the B1 size category with support for Linux containers.

1. The actual thing that is running a website in Azure App Service is the (aptly named) Web App. We create one with this command:

    ```sh
    az webapp create --resource-group <resource group> --plan myappserviceplan --name <app name> --deployment-container-image-name <registry name>.azurecr.io/myappimage:v1.0
    ```

    Where `<app name>` is a name that must be still available as the FQDN `<app name>.azurewebsites.net`. This creates a web app and alreay tells it where it can pull the container image to run as a container in this web app.

1. On your own machine (not the Lab-VM), open the web browser of your choice and navigate to: `http://<app name>.azurewebsites.net`. This will not work yet:

    ![App Service Unavailable](./media/apperviceunavailable.png)

1. In the Azure portal, navigate to your web app by entering "<app name>" in the search bar at the top and selecting **Container settings** on the left. It should be showing a problem with the container image:

    ![Web App Failing To Pull](./media/webappfailingtopull.png)

1. The pulling of the image does not work, because by default it is  trying to pull from Docker hub, where our image cannot be found. We need to tell the web app to pull from our registry:

    ```sh
    az webapp config container set --name <app name> --resource-group <resource group> --docker-custom-image-name <registry name>.azurecr.io/myappimage:v1.0 --docker-registry-server-url https://<registry name>.azurecr.io
    ```
    You might need to restart the web app a few times before it starts working:
    ```sh
    az webapp restart --name <app name> --resource-group <resource group>
    ```
    Only in case your ACR is in a different resource group or subscription than the web app, you might see an error like "Authentication failed" in the logs in the **Container settings** in the portal. This is because the app is not by default authenticated for any ACR. The easiest way to fix this would then be adding the credentials for the registry that we noted in the previous exercise. To do so, use this command:

    ```sh
    az webapp config container set --name <app name> --resource-group <resource group> --docker-registry-server-user <registry user> --docker-registry-server-password <registry password>
    ```

1. One of the features of Web Apps that can make life easier for us is that it automatically adds https (SSL) support, without us needing to install and configure certificates. Now to enforce SSL for our app, we do:

    ```sh
    az webapp update -g <resource group> -n <app name> --https-only true
    ```

    Now, when we browse to our web app again with `http://<app name>.azurewebsites.net`, whe should automatically be redirected to `https://<app name>.azurewebsites.net`.

---

## Extra Challenge: Use Environment Variables

So, Container should be able to run everywhere and everything looks the same all the time? Unfortunately, that is not entirely correct. Typically, there **are** differences between environments, like connection strings or such. In a container world though, we always try to keep these differences down to a level that simple key-value-pairs are enough to configure these settings. The most universal method to pass key-value-pairs to a process is to use environment variables, which is why environment variables are key to this problem in the container world.

In this challenge, the idea is to be able to use the same container image of our app, but let it display info about the environment it is running in in its "About" page.

1. Change the code in `HomeController.cs` to display the label. You will use code similar to this: 

    ```cs
    var label = Environment.GetEnvironmentVariable("ENVIRONMENT_LABEL");
    ```
    nd then use that label in the About-String that this controller returns.

    Or this for Java (JSP):

    ```HTML
    <html>
      <body>
        <h2>Hello World from environment <%= System.getenv("ENVIRONMENT_LABEL")    %>!</h2>
      </body>
    </html>
    ```

1. Rebuild the image, give it a new tag and push it to the registry.

1. Try to run the image on the Docker host with:

    ```sh
    docker container run -e ENVIRONMENT_LABEL=LinuxVM -d -p 80:80 myappimage
    ```
    (or with `-p 80:8080` for Java)

1. Set the new image name and tag and the environment variables in ACI and App Service.

---

## Summary

We now have everything in place to containerize and deploy applications to Azure. The start of everything is the Dockerfile we created, representing the gateway from "out there in the wild" into the new world of containers, where our app will always run in its own beautiful isolated space. We as well created a container registry, which is the foundation for any deployment of containers to a production platform.

Finally, the deployment methods to App Service (for web apps, utilizing the battle tested features of App Service like integrated Authentication or SSL support) or Azure Container Instances (more lightweigt and generic) represent production ready runtime options for **single**-container applications. **Multi**-container applications (e.g. any microservice application) require more, they need *orchestration*, a concept that nowadays we typically address by using an orchestration service like Kubernetes. This will be the topic of our next lab!
