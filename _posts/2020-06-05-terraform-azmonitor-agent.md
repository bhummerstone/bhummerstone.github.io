---
layout: post
title: "Deploying the Azure Monitor Agent for Linux using Terraform"
date: 2020-06-05
---

Hashicorp Terraform is a very popular tool for deploying and managing resources, both in a cloud environment or on-premises. The support in Azure for Terraform is excellent, but I had a bit of trouble getting the Azure Monitor agent installed as a VM Extension, so thought I would share my working code here.

This uses version 0.12 of the Terraform syntax, and was tested with version 2.13.0 of the Azure Provider. You'll need a Service Principal with the necessary rights in order to create these resources; once you have the SP, fill in the details in the provider section.

```terraform
provider "azurerm" {
  version = "=2.13.0"
  features {}

  client_id = "client_id"
  subscription_id = "subscription_id"
  tenant_id = "tenant_id"
  client_secret = "client_secret"
}

resource "random_string" "tfazmon_lga_suffix" {
  length = 6
  special = false
  upper = false
}

resource "azurerm_resource_group" "tfazmon_rg" {
  name     = "tfazmonitor"
  location = "North Europe"
}

resource "azurerm_virtual_network" "tfazmon_vnet" {
  name                = "tfazmonitor"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.tfazmon_rg.location
  resource_group_name = azurerm_resource_group.tfazmon_rg.name
}

resource "azurerm_subnet" "tfazmon_subnet" {
  name                 = "tfazmonitor"
  resource_group_name  = azurerm_resource_group.tfazmon_rg.name
  virtual_network_name = azurerm_virtual_network.tfazmon_vnet.name
  address_prefixes       = ["10.0.0.0/24"]
}

resource "azurerm_network_interface" "tfazmon_nic" {
  name                = "tfazmonitor"
  location            = azurerm_resource_group.tfazmon_rg.location
  resource_group_name = azurerm_resource_group.tfazmon_rg.name

  ip_configuration {
    name                          = "ipconfig1"
    subnet_id                     = azurerm_subnet.tfazmon_subnet.id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_log_analytics_workspace" "tfazmon_lga" {
  name                = "bhtfazmonitor${random_string.tfazmon_lga_suffix.result}"
  location            = azurerm_resource_group.tfazmon_rg.location
  resource_group_name = azurerm_resource_group.tfazmon_rg.name
  sku                 = "PerGB2018"
  retention_in_days   = 180
}

resource "azurerm_virtual_machine" "tfazmon_vm" {
  name                  = "tfazmonitor"
  location              = azurerm_resource_group.tfazmon_rg.location
  resource_group_name   = azurerm_resource_group.tfazmon_rg.name
  network_interface_ids = [azurerm_network_interface.tfazmon_nic.id]
  vm_size               = "Standard_F2s"

  storage_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }

  storage_os_disk {
    name          = "tfazmonitor"
    caching       = "ReadWrite"
    create_option = "FromImage"
    managed_disk_type = "Premium_LRS"
  }

  os_profile {
    computer_name  = "tfazmonitor"
    admin_username = "benhu"
    admin_password = "Password1234!"
  }

  os_profile_linux_config {
    disable_password_authentication = false
  }
}

resource "azurerm_virtual_machine_extension" "tfazmon_ext" {
  name                 = "OmsAgentForLinux"
  virtual_machine_id   = azurerm_virtual_machine.tfazmon_vm.id
  publisher            = "Microsoft.EnterpriseCloud.Monitoring"
  type                 = "OmsAgentForLinux"
  type_handler_version = "1.12"
  auto_upgrade_minor_version = true

  settings = <<SETTINGS
    {
        "workspaceId": "${azurerm_log_analytics_workspace.tfazmon_lga.workspace_id}"
    }
SETTINGS

    protected_settings = <<PROTECTEDSETTINGS
    {
        "workspaceKey": "${azurerm_log_analytics_workspace.tfazmon_lga.primary_shared_key}"
    }
PROTECTEDSETTINGS
}
```