---
layout: post
title: "Tag Management with Azure Policy and Terraform"
date: 2020-06-18
---

When deploying resources via Terraform, especially as part of a CI/CD pipeline, one of the main pain points I've seen from a Governance perspective is how best to manage tagging. For example, say I want to define the following tags for filtering and cost management purposes:
- business_owner
- cost_centre
- application_name

When we're creating an individual resource, defining these tags is easy enough:

```terraform
resource "azurerm_resource_group" "tf_tags_rg" {
  name     = "tftags"
  location = "North Europe"

  tags = {
    business_owner = "benhu"
    cost_centre = "12345"
    application_name = "myapp"
  }
}
```

The problem is that we have to define the same tags for every resource we create. This is fine if there's a handful, but what if there are tens or hundreds?

I've seen people extract the tags to a separate variables file, which is definitely better, but you still then have to remember "tags = var.tags" in all your resources. What if there was a way to define tags on a Resource Group, and then have everything automatically inherit...

In Azure, there is something called Azure Policy, and this is generally the mechanism used for tag management if using native tooling:
- First Policy blocks creation of a Resource Group if Tags are not defined
- Second Policy auto-filters tags from the Resource Group down to the child resources

However, if you are deploying via Terraform, it won't be aware of these tags as they are applied by the Azure Policy engine and therefore won't be in the Terraform state. You could do some manual importing, but this is not sustainable in the long run.

Luckily, there's a pretty cool way you can do this using some custom lifecycle logic in your modules.

Let's look at our example from earlier. We'll create an Azure Policy and link it to our Subscription to enforce these tags, as well as link some built-in Policies to inherit these tags to resources. Note, you can create and apply this Policy using Terraform... but that's for another time.

![Screenshot of Azure Policy screen showing assigned Policies](/images/tf-tags-policies.png)

To manage the tag lifecycle, we then need to add some code to our Terraform modules. Here, I've got a module that creates an Azure Storage Account, but uses the lifecycle keyword in Terraform to ignore changes to specific Tags:

```terraform
# append random string to storage account for uniqueness
resource "random_string" "stg_suffix" {
  length = 8
  special = false
  upper = false
}

resource "azurerm_storage_account" "stg_acct" {
  name = "${var.name}${random_string.stg_suffix.result}"
  location = var.location
  resource_group_name = var.rg_name
  account_tier = var.type
  account_replication_type = var.replication

  tags = {
      business_owner = "placeholder"
      cost_centre = "placeholder"
      application_name = "placeholder"
  }

  lifecycle {
      ignore_changes = [
          tags["business_owner"],
          tags["cost_centre"],
          tags["application_name"]
      ]
  }
}
```

We can then call this code as per below. Note that we only define Tags on the Resource Group, and not on the Storage Account itself:

```terraform
resource "azurerm_resource_group" "tf_tags_rg" {
  name     = "tftags"
  location = "North Europe"

  tags = {
    business_owner = "benhu"
    cost_centre = "12345"
    application_name = "myapp"
  }
}

module "tf_tags_stg" {
  source = "./stg_tags"
  
  name = "bhtftag"
  location = azurerm_resource_group.tf_tags_rg.location
  rg_name = azurerm_resource_group.tf_tags_rg.name
  type = "Standard"
  replication = "LRS"
}
```

We see that the Resource Group gets created with the required Tags, the Storage Account is created by Terraform using the placeholder tags, and finally the Storage Account inherits the tags as per the defined Policies:

![Screenshot showing storage account being created with placeholder tags](/images/tf-tags-stg-create.png)

![Screenshot showing inherited tags](/images/tf-tags-stg-inherit.png)

If we want to then update the Storage Account from LRS to GRS, we can see that only the replication type is being changed, and not the Tags due to the lifecycle policy we defined in the module:

![Screenshot showing updated Storage Account in Terraform](/images/tf-tags-stg-update.png)