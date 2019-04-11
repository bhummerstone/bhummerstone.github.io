---
layout: post
title: "On-Demand, Serverless SFTP"
date: 2019-04-11
---

SSH File Transfer Protocol or SFTP has been around for over 20 years, and still remains a great way to transfer files securely, specifically allowing the use of SSH keys to ensure that only the right users can perform the necessary operations.

One such scenario is the ability to receive files from known third-parties. This would generally involve having a server running 24/7, and possibly left open to the internet all the time as well. Even in the cloud, having a VM (or two for HA) switched on full time for such a simple task can quickly rack up costs.

So I thought about creating an on-demand SFTP service. This could easily be done in a VM by just starting and stopping it, but VMs are quite heavyweight for just receiving files; let's see if we can create an on-demand SFTP service with no VMs required!

Containers are a good choice as a stand-in for VMs: their start times are generally much faster, and they can contain (pun intended) just the services we require; in our case, OpenSSH.

Microsoft Azure provides [Azure Container Instances (ACI)](https://azure.microsoft.com/en-us/services/container-instances/) for this purpose, which is an incredibly easy way to run a container image: just give it an image from Docker Hub and you're pretty much set.

However, since ACI abstracts away from the underlying infrastructure running these containers, we need to use some kind of remote file system for persistent storage; enter [Azure Files](https://azure.microsoft.com/en-gb/services/storage/files/).

ACI provides native support for mounting Azure Files shares within running containers, so this is perfect for my requirements.

Since I started writing this blog post, this usage of ACI has been made into a formal Azure Sample, so a reference template can be found [here](https://azure.microsoft.com/en-gb/resources/samples/sftp-creation-template/).

## Getting started

I'll be doing all of this in the Azure CLI, which you can download from [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) or access via the [Azure Cloud Shell](https://shell.azure.com). Also, ACI isn't available in all of the Azure regions yet, so I'll be using West Europe; make sure you check availability in your region of choice.

Firstly, let's create a new Resource Group to hold the resources we're going to be creating:

```bash
az group create --name bh-rg-sftp-we-01 --location westeurope
```

Next, we need to create the storage account. This will be used to host our file shares to be mounted into the containers at runtime:

```bash
az storage account create --name bhstgsftpwe01 --location westeurope --resource-group bh-rg-sftp-we-01 -sku Standard_LRS --kind Storage
```

Finally, let's create the file share that we'll mount into the ACI:

```bash
connection_string = $(az storage account show-connection-string -n <storage-account> -g <resource-group> --query 'connectionString' -o tsv)
az storage share create --name upload --quota 1024 --connection-string $connection_string
```

## Choosing our image

One of the best things about containers is the incredible community that has grown as part of this movement. Central to this is the work that Docker have done to maintain central image repositories such as Docker Hub to enable people to upload and share images that they have created.

With this in mind, I decided to use one of these pre-existing images for our SFTP server; specifically the sftp image created by atmoz, which is available [here](https://hub.docker.com/r/atmoz/sftp/).

There are a number of different ways to get the user/password details into this particular container, one of which is by using the SFTP_USERS environment variable. This needs to be in the format username:password:userID e.g. ftpuser:Password1234:1001. This environment variable will be composed of various parameters passed to a template at deployment time.

## Creating the ACI template

I decided to use an Azure Resource Manager (ARM) template to create the resources required for this project. This will allow us to define the infrastructure in code, and take advantage of all the usual version control and Continuous Integration/Continuous Deployment (CI/CD) pipelines. Note that you can use any other method of your choice for deploying the ACI e.g. Azure CLI, PowerShell, Terraform etc.

The template schema for creating ACIs is pretty straightforward, so let's focus on the sections responsible for mounting the Azure File Share. First, defining the mount point into the ACI:

```javascript
"containers": [
    {
        "name": "[variables('sftpContainerName')]",
        "properties": {
            ...
            ...
                "volumeMounts": [
                    {
                        "mountPath": "[concat('/home/', parameters('sftpUser'), '/upload')]",
                        "name": "sftpvolume",
                        "readOnly": false
                    }
                ]
            }
    }
]
```

Here, we are telling the ACI to mount a share (yet to be defined) called "sftpvolume" at the location /home/<username>/upload. The username is defined as a parameter of the template, and so is dynamically set at deployment time.

The next section defines the volume to be mounted:

```javascript
 "volumes": [
    {
        "name": "sftpvolume",
        "azureFile": {
            "readOnly": false,
            "shareName": "[parameters('existingFileShareName')]",
            "storageAccountName": "[parameters('existingStorageAccountName')]",
            "storageAccountKey": "[listKeys(variables('storageAccountId'),'2018-02-01').keys[0].value]"
        }
    }
]
```

This section of the template defines the volume we referenced previously. It is of type "azureFile" i.e. an Azure Files share, and we pass the share name and storage account names as parameters at deployment time. The storage account key is required to access this type of volume, so the listKeys() function is used to extract this on the fly (as long as the user deploying the template has permissions to do so).

Once we have this template defined, we can deploy it via the Azure CLI:

```bash
az group deployment create --resource-group bh-rg-sftp-we-01 --template-file aci-sftp.json --existingFileShareName xxx --existingStorageAccountName yyy ... etc.
```

Once deployed, connect via SFTP, copy some files, and they will appear in your Azure Files share. :)

Remember to delete the ACI once you have finished copying files to ensure you don't rack up unnecessary costs:

```bash
az aci delete --name bhsftpaci01 --resource-group bh-rg-sftp-we-01
```

## Taking it further

There are a few potential ways to extend this implementation, or to customise it to meet your requirements. For example:

* Use SSH instead of passwords: this can be achieved by mounting a second Azure Files share into /home/username/.ssh/keys
** example [here](https://github.com/bhummerstone/azure-templates/blob/master/compute/sftp/sftp-config-file.json)
* Create your own custom image with e.g. custom host keys defined
* Use the ACI Logic Apps connector to create the ACI as part of a wider workflow
* Run the container image in a Kubernetes cluster with the Azure Files share defined as a persistent volume

One key thing to remember is that the ACI should not be a long-lived object: if you need something to always be available, consider deploying a VM to fulfil that role. However, since the SFTP service only really needs to be online for the period of the transfer, it is well worth considering the value that this on-demand SFTP can bring.