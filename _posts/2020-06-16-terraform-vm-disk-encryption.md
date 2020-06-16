---
layout: post
title: "Deploying a Linux VM with CMK Disk Encryption using Terraform"
date: 2020-06-16
---

Continuing the recent Terraform theme, I've also been working on an example of how to deploy a VM in Azure using the new method of [Disk Encryption with Customer Managed Keys](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/disk-encryption#customer-managed-keys).

This is comprised of a few key bits of functionality:
- A Disk Encryption Set to contain the disks to be encrypted
- An Azure Key Vault to store the encryption keys, as well as access policies for the Disk Encryption Set and (optionally) the user deploying the code

This uses version 0.12 of the Terraform syntax, and was tested with version 2.13.0 of the Azure Provider. You'll need a Service Principal with the necessary rights in order to create these resources; once you have the SP, fill in the details in the provider section.

```terraform
provider "azurerm" {
  # whilst the `version` attribute is optional, we recommend pinning to a given version of the Provider
  version = "=2.13.0"
  features {}

  client_id = "client_id"
  subscription_id = "subscription_id"
  tenant_id = "tenant_id"
  client_secret = "client_secret"
}

data "azurerm_client_config" "current" {}

resource "azurerm_resource_group" "tfencrypt_rg" {
  name     = "tfencrypt"
  location = "North Europe"
}

resource "azurerm_virtual_network" "tfencrypt_vnet" {
  name                = "tfencrypt_vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.tfencrypt_rg.location
  resource_group_name = azurerm_resource_group.tfencrypt_rg.name
}

resource "azurerm_subnet" "tfencrypt_subnet" {
  name                 = "tfencrypt_subnet"
  resource_group_name  = azurerm_resource_group.tfencrypt_rg.name
  virtual_network_name = azurerm_virtual_network.tfencrypt_vnet.name
  address_prefixes       = ["10.0.0.0/24"]
}

resource "azurerm_network_interface" "tfencrypt_nic" {
  name                = "tfencrypt_nic"
  location            = azurerm_resource_group.tfencrypt_rg.location
  resource_group_name = azurerm_resource_group.tfencrypt_rg.name

  ip_configuration {
    name                          = "ipconfig1"
    subnet_id                     = azurerm_subnet.tfencrypt_subnet.id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_key_vault" "tfencrypt_kv" {
  name                        = "des-tfencrypt-keyvault"
  location                    = azurerm_resource_group.tfencrypt_rg.location
  resource_group_name         = azurerm_resource_group.tfencrypt_rg.name
  tenant_id                   = data.azurerm_client_config.current.tenant_id
  enabled_for_disk_encryption = true
  soft_delete_enabled         = true
  purge_protection_enabled    = true
  sku_name                    = "standard"
}


resource "azurerm_key_vault_access_policy" "tfencrypt_kvuserpol" {
  key_vault_id = azurerm_key_vault.tfencrypt_kv.id

  tenant_id = data.azurerm_client_config.current.tenant_id
  object_id = data.azurerm_client_config.current.object_id

  key_permissions = [
    "get",
    "create",
    "delete"
  ]
}

resource "azurerm_key_vault_key" "tfencrypt_kvkey" {
  name         = "des-tfencrypt-key"
  key_vault_id = azurerm_key_vault.tfencrypt_kv.id
  key_type     = "RSA"
  key_size     = 2048

  depends_on = [
    azurerm_key_vault_access_policy.tfencrypt_kvuserpol
  ]

  key_opts = [
    "decrypt",
    "encrypt",
    "sign",
    "unwrapKey",
    "verify",
    "wrapKey",
  ]
}

resource "azurerm_disk_encryption_set" "tfencrypt_des" {
  name                = "tfencrypt_des"
  resource_group_name = azurerm_resource_group.tfencrypt_rg.name
  location            = azurerm_resource_group.tfencrypt_rg.location
  key_vault_key_id    = azurerm_key_vault_key.tfencrypt_kvkey.id

  identity {
    type = "SystemAssigned"
  }
}

resource "azurerm_key_vault_access_policy" "tfencrypt_kvdiskpol" {
  key_vault_id = azurerm_key_vault.tfencrypt_kv.id

  tenant_id = azurerm_disk_encryption_set.tfencrypt_des.identity.0.tenant_id
  object_id = azurerm_disk_encryption_set.tfencrypt_des.identity.0.principal_id

  key_permissions = [
    "get",
    "decrypt",
    "encrypt",
    "sign",
    "unwrapKey",
    "verify",
    "wrapKey",
  ]
}

resource "azurerm_linux_virtual_machine" "tfencrypt_vm" {
  name                  = "tfencrypt"
  location              = azurerm_resource_group.tfencrypt_rg.location
  resource_group_name   = azurerm_resource_group.tfencrypt_rg.name
  network_interface_ids = [azurerm_network_interface.tfencrypt_nic.id]
  size               = "Standard_F2s"
  admin_username = "benhu"
  admin_password = "Password1234!"
  disable_password_authentication = false

  source_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }

  os_disk {
    name          = "tfencrypt_osdisk"
    caching       = "ReadWrite"
    storage_account_type = "Premium_LRS"
    disk_encryption_set_id = azurerm_disk_encryption_set.tfencrypt_des.id
  }
}
```