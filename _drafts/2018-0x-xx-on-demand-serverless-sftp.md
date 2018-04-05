---
layout: post
title: "On-Demand, Serverless SFTP"
date: 2018-0x-xx
---

SSH File Transfer Protocol or SFTP has been around for over 20 years, and still remains a great way of securely transferring files, specifically allowing the use of SSH keys to ensure that only the right users can perform the necessary operations.

One such scenario is the ability to receive files from known third-parties. This would generally involve having a server running 24/7, and possibly left open to the internet all the time as well. Even in the cloud, having a VM (or two for HA) switched on full time for such a simple task can quickly rack up costs.

So I thought about creating an on-deman SFTP service. This could easily be done in a VM by just starting and stopping it, but VMs are quite heavyweight for just receiving files. Since "serverless" is the flavour of the month, let's see if we can create an on-demand SFTP service: no VMs required!

Containers are a good choice as a stand-in for VMs: their start times are generally much faster, and they can contain just the services we require; in our case, Open SSH.

I'm going to be using Microsoft Azure for this, as [Azure Container Instances](https://azure.microsoft.com/en-us/services/container-instances/) provides an incredibly easy way to run a container image: just give it an image from Docker Hub and you're pretty much set!

However, since ACI abstracts away from the underlying infrastructure running these containers, we need to use some kind of remote file system for persistent storage; enter [Azure Files](https://azure.microsoft.com/en-gb/services/storage/files/).

ACI provides native support for mounting Azure Files shares within running containers, so this is perfect for my requirements.

Finally, we need something to orchestrate everything, so I'll be using [Azure Logic Apps](https://azure.microsoft.com/en-us/services/logic-apps/) for this purpose, especially as there is a preview connector for creating ACI container groups.

## Getting started
I'll be doing all of this in the Azure CLI, which you can download from [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) or access via the Azure Cloud Shell at (https://shell.azure.com). Also, ACI isn't available in all of the Azure regions yet, so I'll be using West Europe.

Firstly, let's create a new Resource Group to hold the resources we're going to be creating as part of this.

```
az group create --name bh-rg-sftp-we-01 --location westeurope
```

Next, we need to create the storage account. We can then get the details of this account and store them as environment variables so that they can be referenced from future commands without us having to type the info in each time.

```
az storage account create --name bhstgsftpwe01 --location westeurope --resource-group bh-rg-sftp-we-01 -sku Standard_LRS --kind Storage

```

Using atmoz/sftp
Get public key from third party
Write username/key into config file to be mounted into container
Generate host key, and mount into container
Create new Azure File share for user and mount into container into their home directory

Extending further
- Check if container group is already running
- Trigger from web page/function?
- Allow self-service sign up?