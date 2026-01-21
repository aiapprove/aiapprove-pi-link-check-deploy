# Deploy to Azure Button

Use this button to deploy the PI Link Check Agent infrastructure to your Azure subscription.

## Button Snippet

Copy this markdown to add a "Deploy to Azure" button to your documentation:

```markdown
[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Faiapprove%2Fpi-link-check-agent%2Fmain%2Fdeploy-to-azure%2Fazuredeploy.json)
```

## Rendered Button

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Faiapprove%2Fpi-link-check-agent%2Fmain%2Fdeploy-to-azure%2Fazuredeploy.json)

## URL Structure

The Deploy to Azure URL follows this format:

```
https://portal.azure.com/#create/Microsoft.Template/uri/<encoded-template-url>
```

Where `<encoded-template-url>` is the URL-encoded path to the ARM template JSON file.

## Alternative: Azure CLI Deployment

If you prefer to deploy via CLI:

```bash
# Login and set subscription
az login
az account set --subscription "Your Subscription Name"

# Create resource group
az group create --name rg-pilinkcheck-prod --location ukwest

# Deploy template
az deployment group create \
  --resource-group rg-pilinkcheck-prod \
  --template-file azuredeploy.json \
  --parameters @azuredeploy.parameters.json
```

## Alternative: PowerShell Deployment

```powershell
# Login and set subscription
Connect-AzAccount
Set-AzContext -Subscription "Your Subscription Name"

# Create resource group
New-AzResourceGroup -Name "rg-pilinkcheck-prod" -Location "ukwest"

# Deploy template
New-AzResourceGroupDeployment `
  -ResourceGroupName "rg-pilinkcheck-prod" `
  -TemplateFile "azuredeploy.json" `
  -TemplateParameterFile "azuredeploy.parameters.json"
```

## After Deployment

See [README.md](./README.md) for post-deployment steps including:
- Creating the Entra ID app registration
- Granting Sites.Selected permission
- Provisioning SharePoint lists
- Building and uploading the Teams manifest
