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
- A Linux machine running in Azure with access to the cloud and the following components installed:
   - Docker ([installation instructions](https://docs.docker.com/install/linux/docker-ce/ubuntu/))
   - .NET Core SDK ([installation instructions](https://www.microsoft.com/net/download/linux-package-manager/ubuntu16-04/sdk-current))

---

Estimated time to complete this lab: **90-120** minutes.

## Exercise 1: Log on to your VM

1. Open the [Azure Portal](https://portal.azure.com), log on with your lab account, if necessary. You can see the currently logged on account in the top right of the portal:

   ![Loggend on account in Azure portal](./media/portalloggedonuser.png)

1. Start the Azure Cloud Shell (Bash) by clicking the console icon in the top bar of the portal:

   ![Start Azure Cloud Shell](./media/portalopenconsole.png)

In case you have not worked with the Azure Cloud Shell before, you will be asked a few questions. Click **Bash (Linux)** and **Create Storage**, accept all defaults. Your console should then look like this:

   ![Ready Azure Cloud Shell](./media/portalconsoleready.png)

1. Log on to the machine with the user and machine address provided by your instructor, by typing the following command into the cloud shell:

    ```bash
    ssh <user>@<machine adress> 
    ```
    For example:
    ```bash
    ssh Carsten@cadullcontlablin.westeurope.cloudapp.azure.com 
    ```
    You will be asked whether to accept this new host (enter **yes**) and for the password (enter the password provided by your instructor).

    After this login succeeds, we have a bash shell running right in the Linux VM, in which we will work with Docker.

1. To test, whether Docker is indeed installed, type:

    ```bash
    docker --version 
    ```
    This should display the docker version.

## Exercise 2: Run a web server in a container

For a start, we will be running a very powerful and lightweight web server called [nginx](https://www.nginx.com/). This web server is available as a pre-packaged container image at [Docker Hub](https://hub.docker.com/_/nginx/).

1. We will tell docker to pull the nginx image from Docker Hub and let it run in our docker engine with a single command. Type:

    ```bash
    docker container run --name mynginx -d nginx
    ```
    The `-d` flag in the command tells Docker that the container should run detached, in the background. We do not currently want to interact with it directly. We just want it to run silently with default settings, listening on its default port.

1.  The engine just reports a lengthy id, which does not help us much understanding what is going on. Let's see whether the container actually started. To see all running containers, type: 
    ```bash
    docker container ls
    ```
    This returns all running containers with their name and a few more fields. In case you do not find your container in the list, something has gone wrong. To troubleshoot, you would... 
    ```bash
    docker container ls --all
    ```
    ..., which lists all non-running containers as well, and then use...
    ```bash
    docker container logs <containername>
    ```
    ..., to find out what the container reported while starting (or trying to start). In our case, the container logs will show nothing yet.

1. The container we just started already serves as a webserver, listening on port 80, but we cannot see its webpage yet. So far, the container is only attached to an internal virtual network called 'bridge', that can only be accessed from other containers on that network and our VM, which is hosting the network and the container. So, let's run another instance of nginx, but now we will map the containers port 80 to an externally visible port on the VM:

    ```bash
    docker container run --name mynginxwithport -d -p 80:80 nginx
    ```
    The `-p 80:80` flag tells docker to map the VM's port 80 to the containers port 80, so that the container now virtually listens on the VM's own port 80. If that port is already reserved by another process on the VM, our call would fail.

1. On your own machine (not the Lab-VM), open the web browser of your choice and navigate to the address of your VM (the same you used to log on the machine in the beginnning) as `http://<machine address>`:

   ![The welcome page served from our container](./media/nginxwelcome.png)

1. **Optional**: In case you want to get a glimpse of how the internal networking works, check the webserver from within the VM: You can find out the container's IP address by inspecting the default docker network 'bridge':

    ```bash
    docker network inspect bridge
    ```
    With that IP address, you can then curl the web servers start page like this:
    ```bash
    curl http://<container ip>
    ```