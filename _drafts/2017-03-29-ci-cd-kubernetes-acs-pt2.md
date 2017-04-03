---
layout: post
title: "CI/CD with Kubernetes on ACS - Part 2 - Creating ACS and ACR"
date: 2017-03-29
---

Part 1 - Introduction
Part 2 - Creating ACS & ACR (this post)
Part 3 - Configuring VSTS
Part 4 - Kubernetes-ifying Application
Part 5 - Build Definition
Part 6 - Release Definition
Part 7 - Wrap-up

The first step is to get up and running with Azure and the Azure CLI. You can get a free trial for Azure [here](), which will give you more than enough credit to get up and running. If you already have an Azure subscription, make sure that you have the necessary permissions to create [Service Principles]() as this is required by Kubernetes for configuring the Azure resources.

You can get the Azure CLI v2 from [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).

Once you are set up with an Azure account and the CLI, open up a terminal/command prompt and run:

{% highlight bash %}
    az login
{% endhighlight %}

This will prompt you with a code, and you will need to open a web browser to http://aka.ms/devicelogin, enter the code, and login to your Azure account. Once this is complete, you will then be authenticated from the CLI.

Alternatively, depending on your account security settings, you can login using:

{% highlight bash %}
    az login -u <username> -p <password>
{% endhighlight %}

Next step is to create a new Resource Group in Azure to contain all of the resources we are going to be creating. The code for this is:

{% highlight bash %}
    Az group create --name k8scicd --location westeurope
{% endhighlight %}

Note: I'm using West Europe for everything as part of this series, as it contains all of the services that 
Az acs create  --name k8scicd --resource-group k8scicd  --orchestrator-type kubernetes --dns-prefix bhk8s --verbose
Kubectl get nodes
	- Failed initially: "Private key file is encrypted"
	- Old kubernetes config file in ~/.kube/config
	- Rm ~/.kube/config
	- Scp azureuser@bhk8s.westeurope.cloudapp.azure.com:.kube/config $HOME/.kube/config
	- All working then!


Az storage account create --name bhacrstg --location westeurope --resource-group k8scicd --sku Standard_LRS
Az acr create --name bhacr --location westeurope --resource-group k8scicd --storage-account-name bhacrstg
	- "The subscription is not registered to use namespace 'Microsoft.ContainerRegistry'…
	- Az provider register --namespace Microsoft.ContainerRegistry
	- Az provider show --namespace Microsoft.ContainerRegistry | grep registrationState
		○ Changes to "registered" when complete (might take a couple of mins)
bhacr-microsoft.azurecr.io


Now that you're up and running with ACS & ACR, check out [Part 3]() to prepare VSTS. 