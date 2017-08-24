New-AzureRMKeyVault -Name bhkeyvault -ResourceGroupName bhkeyvaultrg -Location ukwest -EnabledForDeployment -EnabledForTemplateDeployment

Set-AzureKeyVaultSecret -VaultName bhkeyvault -Name adminPassword
Enter SecretValue when prompted

(Get-AzureKeyVaultSecret -VaultName bhkeyvault -Name adminPassword).SecretValueText

Set-AzureRmKeyVaultAccessPolicy -VaultName bhkeyvault -ResourceGroup
Name bhkeyvault -EnabledForTemplateDeployment

Add reference in parameters file

      "sqlsvrAdminLoginPassword": {
        "reference": {
          "keyVault": {
            "id": "/subscriptions/{guid}/resourceGroups/{group-name}/providers/Microsoft.KeyVault/vaults/{vault-name}"
          },
          "secretName": "adminPassword"
        }
      },