---
layout: post
title: "Automating Azure Bastion using PowerShell Functions"
date: 2019-11-29
---

Inspired by a conversation with a colleague about "Just In Time" access to workloads, I decided to investigate how to automate provisioning and de-provisioning Azure Bastion on weekdays only to reduce the overall running costs.

Here's a quick overview of my (self-imposed) requirements for the solution:

1. Deploy new at 08:30 each weekday, remove at 18:00
2. Minimise running costs for automation
3. No passwords to manage
4. No hard-coding of values in code
5. Keep same public IP. In theory, Bastion can create a new Public IP for you each time, but it helps to keep this static for e.g. whitelisting purposes

## Step 1 - Creating the Static Resources
In order for an Azure Bastion host to be created, it requires a few pre-requisites:

- Resource group in which to live
- Virtual Network in a supported region (e.g. West Europe)
- Subnet of a suitable size (at least /27) in that virtual network with the name AzureBastionSubnet
- Public IP in the same region as the virtual network with the Standard SKU
 

Therefore, the first step is to create these resources. I chose to use the Azure CLI from the Cloud Shell to do this, but use whatever method works for you:

```bash
az group create --name bh-bastion --location westeurope
az network vnet create --name bh-bastion-vnet --resource-group bh-bastion --location westeurope --address-prefixes '10.0.0.0/16' --subnet-name AzureBastionSubnet --subnet-prefixes '10.0.255.0/24'
az network public-ip create --name bh-bastion-pip --resource-group bh-bastion --location westeurope --allocation-method Static --sku Standard
```

Make a note of the names and Resource Group for these, as we'll need them later. Note that so far the only running cost for this is the Public IP Address, which comes in at £2.72/month... although you probably have a load of these already in use that you can reclaim as they won't be needed any more :)


## Step 2 - Creating the Function App
Now that we have our "landing zone" for the Bastion host, we need a way of deploying and destroying it on a regular basis. There are numerous different ways of doing this, but one that meets all of the requirements is doing this using PowerShell in an Azure Function App. PowerShell support went GA recently, and Azure Functions have some excellent built-in features such as Timer triggers and Managed Identities that align with our goals. We can also run this on a Consumption plan, so we only pay for the time the commands are running: perfect for requirement #2.

We can also assign some variables as Application Settings in the Function to tick requirement #4.

Creating a new Function App is also pretty straightforward:

```bash
az storage account create --name bhstgfuncbastion --location westeurope --resource-group bh-bastion --sku Standard_LRS
az functionapp create --name bhfuncbastion --resource-group bh-bastion --consumption-plan-location westeurope --name bhfuncbastion --storage-account bhstgfuncbastion --os-type Windows --runtime powershell --disable-app-insights true
az functionapp config appsettings set --name bhfuncbastion --resource-group bh-bastion --settings "BASTION_VNET_NAME=bh-bastion-vnet" "BASTION_VNET_RG=bh-bastion" "BASTION_PIP_NAME=bh-bastion-pip" "BASTION_PIP_RG=bh-bastion" "BASTION_NAME=bh-bastion" "BASTION_RG=bh-bastion"
```

## Step 3 - Setting up Identity and Access Control
For our Function App to be able to perform actions within Azure, it needs to have permission to deploy and remove resources from the Bastion resource group. Functions has an option to assign a Managed Identity, which is an identity for the Function App itself that exists in Azure Active Directory and can be combined with Role Based Access Control to grant permissions as required; this also ticks off requirement #3.

In our case, let's assign the identity and then give it Contributor access over the Bastion Resource Group. In an ideal world, I would use the following command:

```bash
az functionapp identity assign --name bhfuncbastion --resource-group bh-bastion --role Contributor --scope $(az group show --name bh-bastion --query 'id' -o tsv)
```

However, this doesn't currently work; see [this GitHub Issue](https://github.com/Azure/azure-cli/issues/11435). 

As a workaround, you can enable the Managed Identity through the Portal:

1. Browse to your Function App in the Azure Portal
2. Click on Platform features
3. Click on Identity, which is under Networking
4. Change the Status to On
5. Hit Save, and click Yes when prompted

Copy the object ID shown on the screen, and we can assign the Contributor role as follows:

```bash
az role assignment create --assignee <object_id_from_function_app> --role Contributor --scope $(az group show --name bh-bastion --query 'id' -o tsv)
```

## Step 4 - Creating the Create/Remove Functions
In terms of creating the Functions themselves, the easiest way I've found to do this is either using the built-in editor in the Azure Portal, or by using Visual Studio Code. If you end up doing any extensive work with Functions I highly recommend the latter option, but for now let's work in the Portal.

For the Create Function:

1. Browse to your Function App in the Azure Portal
2. On the left hand side, under the main Function drop down, click on Functions, then on the + New function button at the top
3. From the templates, choose Timer trigger
4. Give the Function a suitable name e.g. Create_Azure_Bastion
5. For the Schedule, enter "0 30 8 * * 1-5". This is cron syntax for "08:30 on days 1-5" i.e. Monday to Friday

You should now be in the in-Portal editor. Replace the code with the PowerShell located [here](https://raw.githubusercontent.com/bhummerstone/azure-function-bastion/master/powershell/Create_Azure_Bastion.ps1), and hit Save.

For the Remove Function:

1. Browse to your Function App in the Azure Portal
2. On the left hand side, under the main Function drop down, click on Functions, then on the + New function button at the top
3. From the templates, choose Timer trigger
4. Give the Function a suitable name e.g. Remove_Azure_Bastion
5. For the Schedule, enter "0 0 18 * * 1-5". This is cron syntax for "18:00 on days 1-5" i.e. Monday to Friday

As before, replace the code with the PowerShell located [here](https://raw.githubusercontent.com/bhummerstone/azure-function-bastion/master/powershell/Remove_Azure_Bastion.ps1) and hit Save.

## Step 5 - Increase Function App Timeout

One final (optional) item: in my testing, I occasionally saw the Removal take longer than the default 5 minute timeout supported by Functions (never the creation, oddly enough!). To workaround this, we can change the default timeout for our Functions to 10 minutes instead.

This is achieved by editing the host.json file for your Function App. This can be done locally using Visual Studio Code and the Function Core Tools, but you can also do this in the Portal:

1. Browse to your Function App in the Azure Portal
2. Click on Platform features
3. Click on App Service Editor, which is under Development Tools. This should open a new tab, allowing us to edit the files in the Function App
4. Click on host.json, which should then open in the right hand pane
5. Add the following code as a new line: "functionTimeout": "00:10:00"
6. The file automatically saves after you make a change, so you can now close the tab

## Conclusion
There we have it: one Function App, two Functions, and all of our requirements nicely met.

From a potential saving perspective, I did some rough calculations using the Azure Pricing Calculator:

- Running Bastion for 730 hours (roughly 24/7 for 1 month) in West Europe comes to **£51.69**
- With the Create/Remove method, we run for 9.5 hours/day, weekdays only, and if we say 21 weekdays per month, this is 199.5 hours, which totals **£14.16**

That's over 70% saving! 

Also, total running cost of our Functions? Free. Even if they run for the maximum of 10 minutes each time. Not bad for 5 lines of PowerShell.