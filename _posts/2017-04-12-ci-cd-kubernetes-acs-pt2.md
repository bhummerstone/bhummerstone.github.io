---
layout: post
title: "CI/CD with Kubernetes on ACS - Part 2 - Creating ACS and ACR"
date: 2017-04-12
---

Part 1 - [Introduction]({% post_url 2017-04-12-ci-cd-kubernetes-acs-pt1 %})  
Part 2 - Creating ACS & ACR (this post)  
Part 3 - Configuring VSTS  
Part 4 - Kubernetes-ifying Application  
Part 5 - Build Definition  
Part 6 - Release Definition  
Part 7 - Wrap-up  

The first step is to get up and running with Azure and the Azure CLI. You can get a free trial for Azure [here](https://azure.microsoft.com/en-gb/free/), which will give you more than enough credit to get up and running. If you already have an Azure subscription, make sure that you have the necessary permissions to create [Service Principals](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) as this is required by Kubernetes for configuring the Azure resources.

You can get the Azure CLI v2 from [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).

Once you are set up with an Azure account and the CLI, open up a terminal/command prompt and run:

```bash
    az login
```

This will prompt you with a code, and you will need to open a web browser to http://aka.ms/devicelogin, enter the code, and login to your Azure account. Once this is complete, you will then be authenticated from the CLI.

Alternatively, depending on your account security settings, you can login using:

```bash
    az login -u <username> -p <password>
```

Next step is to create a new Resource Group in Azure to contain all of the resources we are going to be creating. The code for this is:

```bash
    az group create --name k8scicd --location westeurope
```

Note: I'm using West Europe for everything as part of this series, as it contains all of the services that are required. Double check the Azure services by region page to help choose a suitable region for you.

Now we have our Resource Group, it's time to create our ACS cluster:

```bash
	az acs create  --name k8scicd --resource-group k8scicd  --orchestrator-type kubernetes --dns-prefix bhk8s --verbose
```

This command creates the ACS cluster using k8s as the orchestrator and configures the DNS prefix for connecting to the cluster later.

Once the cluster has been created, you'll need to install kubectl, which is the CLI for k8s. You can do that by running the following:

```bash
	sudo az acs kubernetes install-cli
```

This installs kubectl to /usr/local/bin/kubectl by default, but you can change it with the --install-location parameter if required.

To check everythign is working, run:

```bash
	kubectl get nodes
```

Assuming everything has gone to plan, you should get a list back of the nodes in your ACS cluster.

I'll be honest: when I was writing this up, this command failed for me with the warning "Private key file is encrypted". After a bit of troubleshooting, I worked out that it was because I had an old kubernetes config file lurking around that was trying to connect me to a different cluster.

To resolve this, I removed the existing config and then copied the new config down from my k8s cluster:

```bash
	rm ~/.kube/config
	scp azureuser@bhk8s.westeurope.cloudapp.azure.com:.kube/config $HOME/.kube/config
```

You now have a k8s cluster up and running!

Before we move on, let's set up our Azure Container Registry. To do this, we create a storage account, and than an ACR:

```bash
	az storage account create --name bhacrstg --location westeurope --resource-group k8scicd --sku Standard_LRS
	az acr create --name bhacr --location westeurope --resource-group k8scicd --storage-account-name bhacrstg
```
	
Another command, another error: this time it was "The subscription is not registered to use namespace 'Microsoft.ContainerRegistry'". As ACR has only recently become Generally Available, your subscription might not have been updated to register with the Resource Provider that supports it. Easily solved though:

```bash
	az provider register --namespace Microsoft.ContainerRegistry
	az provider show --namespace Microsoft.ContainerRegistry | grep registrationState
```

registrationState should change to "registered" when complete, which might take a couple of minutes.

Now that you're up and running with ACS & ACR, check out [Part 3]() to prepare VSTS. 