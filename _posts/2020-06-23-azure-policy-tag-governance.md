---
layout: post
title: "Tag Management with Azure Policy"
date: 2020-06-23
---

Following on from my previous post about managing tags using Azure Policy and Terraform, there was some interest in how the Policies worked for both enforcing the tags, and making them inherit down from the Resource Group to the deployed resources.

This is actually deceptively simple as we can utilise a few built-in Policies to do the work for us!

The first Policy is called "Require a tag on resource groups", and has the Definition ID /providers/Microsoft.Authorization/policyDefinitions/96670d01-0a4d-4649-9c89-2d3abc0a5025.

The policyRule of this Definition is as follows:

```json
"policyRule": {
      "if": {
        "allOf": [
          {
            "field": "type",
            "equals": "Microsoft.Resources/subscriptions/resourceGroups"
          },
          {
            "field": "[concat('tags[', parameters('tagName'), ']')]",
            "exists": "false"
          }
        ]
      },
      "then": {
        "effect": "deny"
      }
    }
```

The logic here is:
- If what is being deployed is a Resource Group
- And a particular named tag doesn't exist
- Then deny the deployment

You can assign this same policy multiple times, with one instance for each tag to be enforced.

The Policy for the inheritance is called "Inherit a tag from the resource group", and has Definition ID /providers/Microsoft.Authorization/policyDefinitions/cd3aa116-8754-49c9-a813-ad46512ece54.

The policyRule of this Definition is as follows:

```json
"policyRule": {
      "if": {
        "allOf": [
          {
            "field": "[concat('tags[', parameters('tagName'), ']')]",
            "notEquals": "[resourceGroup().tags[parameters('tagName')]]"
          },
          {
            "value": "[resourceGroup().tags[parameters('tagName')]]",
            "notEquals": ""
          }
        ]
      },
      "then": {
        "effect": "modify",
        "details": {
          "roleDefinitionIds": [
            "/providers/microsoft.authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
          ],
          "operations": [
            {
              "operation": "addOrReplace",
              "field": "[concat('tags[', parameters('tagName'), ']')]",
              "value": "[resourceGroup().tags[parameters('tagName')]]"
            }
          ]
        }
      }
}
```

This uses a more interesting effect known as "Modify", which can alter resources as they are being deployed.

As such, the logic is:
- If the resource being deployed doesn't have a named tag matching one defined in the containing Resource Group
- And the Resource Group tag doesn't have an empty value
- Then modify the resource to include the Resource Group's tag value

As above, you can assign this Policy multiple times and specify the tag names to use in each case to match those enforced by the first Policy defined above.

To keep this all together, I'd recommend creating a Policy Initiative called "Tag Governance" or similar and use that to group the Definitions and Assignments together, as that will make administration much more consistent.

If you want to take this further, there's no reason why you can't create your own Policy Definitions to alter the behaviour. Next time, I'll show an example of how we can do that using Terraform.