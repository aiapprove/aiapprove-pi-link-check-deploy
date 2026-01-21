# Deploy to Azure - In-Tenant Deployment

This folder contains everything needed to deploy the PI Link Check Agent infrastructure to your own Azure subscription using the "Deploy to Azure" button.

---

## Overview

The **In-Tenant** deployment model runs all infrastructure within your Azure subscription:

- **Function App** - Hosts the PI link validation backend
- **Key Vault** - Securely stores secrets
- **Storage Account** - Function App runtime storage
- **Application Insights** - Monitoring and diagnostics
- **Log Analytics** - Centralized logging

**Data residency:** All data stays within your Azure subscription and chosen region.

---

## Prerequisites

Before deploying:

| Requirement | Notes |
|-------------|-------|
| Azure subscription | Contributor access to create resources |
| SharePoint Online site | Where scan results will be stored |
| Resource group | Create or use existing (deployment creates resources inside) |

---

## One-Click Deploy

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Faiapprove%2Fpi-link-check-agent%2Fmain%2Fdeploy-to-azure%2Fazuredeploy.json)

**Click the button above** to open the Azure Portal deployment wizard.

---

## Deployment Steps

### Step 1: Click Deploy Button

1. Click the "Deploy to Azure" button above
2. Sign in to Azure Portal if prompted
3. Select your subscription and resource group

### Step 2: Configure Parameters

Fill in the required parameters:

| Parameter | Required | Description |
|-----------|----------|-------------|
| `sharePointSiteUrl` | Yes | Your SharePoint site URL |
| `location` | Yes | Azure region (default: ukwest) |
| `environment` | No | dev/staging/prod (default: prod) |
| `baseName` | No | Resource name prefix (default: pilinkcheck) |

See [parameter-descriptions.md](./parameter-descriptions.md) for detailed parameter documentation.

### Step 3: Deploy

1. Click **Review + create**
2. Review the summary
3. Click **Create**
4. Wait for deployment to complete (typically 3-5 minutes)

### Step 4: Note Outputs

After deployment completes, go to **Outputs** tab and note:

- `functionAppUrl` - Your backend URL
- `functionAppPrincipalId` - Managed identity ID (needed for SharePoint permissions)
- `keyVaultName` - For storing secrets

---

## Post-Deployment Steps

The ARM template deploys infrastructure, but additional configuration is required:

### 1. Create Entra ID App Registration (5 min)

If you don't have an app registration:

```powershell
# Create app registration
az ad app create --display-name "PI Link Check Agent" \
  --sign-in-audience AzureADMyOrg \
  --web-redirect-uris "https://teams.microsoft.com/auth-callback"

# Note the appId from output
```

Then update the Function App setting:
```powershell
az functionapp config appsettings set \
  --name <functionAppName> \
  --resource-group <resourceGroup> \
  --settings "ENTRA_APP_CLIENT_ID=<appId>"
```

### 2. Grant Sites.Selected Permission (5 min)

Grant the Function App's managed identity access to your SharePoint site:

```powershell
# Connect to SharePoint
Connect-PnPOnline -Url $SiteUrl -Interactive

# Grant permission (use functionAppPrincipalId from outputs)
Grant-PnPAzureADAppSitePermission `
  -AppId <functionAppPrincipalId> `
  -DisplayName "PI Link Check Agent" `
  -Site $SiteUrl `
  -Permissions Write
```

### 3. Provision SharePoint Lists (2 min)

Run the list provisioning script:

```powershell
cd packages/connector/deploy
.\provision-lists.ps1 -SiteUrl "https://contoso.sharepoint.com/sites/PICheck"
```

This creates:
- `PILinkCheck_ScanRuns` - Scan metadata
- `PILinkCheck_FileResults` - Per-file results
- `PILinkCheck_UrlResults` - Per-URL validation results

### 4. Deploy Function App Code (5 min)

Deploy the backend code to the Function App:

```bash
cd packages/functions
npm install && npm run build
func azure functionapp publish <functionAppName>
```

### 5. Build and Upload Teams Manifest (5 min)

```powershell
# Build manifest with your backend URL
npm run connector:build-manifest -- -BackendUrl "https://<functionAppName>.azurewebsites.net"

# Upload to Teams
# 1. Open Teams → Apps → Manage your apps → Upload an app
# 2. Select packages/connector/teams/manifest.zip
# 3. Add to your team/channel
```

---

## Verification

After completing all steps:

```bash
# Health check
curl https://<functionAppName>.azurewebsites.net/api/linkcheck/health

# Expected response:
# {"status":"healthy","lists":{"scanRuns":true,"fileResults":true,"urlResults":true}}
```

---

## Files in This Folder

| File | Description |
|------|-------------|
| `azuredeploy.json` | ARM template for Azure Portal deployment |
| `azuredeploy.parameters.json` | Example parameters file |
| `deploy-button.md` | Deploy button snippet for docs |
| `parameter-descriptions.md` | Detailed parameter documentation |
| `README.md` | This file |

---

## Alternative: Azure CLI Deployment

```bash
# Create resource group
az group create --name rg-pilinkcheck-prod --location ukwest

# Deploy
az deployment group create \
  --resource-group rg-pilinkcheck-prod \
  --template-file azuredeploy.json \
  --parameters sharePointSiteUrl="https://contoso.sharepoint.com/sites/PICheck"
```

---

## Troubleshooting

### "QuotaExceeded" in UK South

UK South has limited consumption plan quota. Use `ukwest` or `northeurope` instead.

### Function App not starting

Check Application Insights for errors:
1. Azure Portal → Function App → Application Insights
2. Look for exceptions in Failures blade

### SharePoint permission errors

Verify Sites.Selected permission:
```powershell
Get-PnPAzureADAppSitePermission -Site $SiteUrl
```

---

## Cost Estimate

| Resource | SKU | Estimated Monthly Cost |
|----------|-----|----------------------|
| Function App | Consumption (Y1) | Pay-per-execution (~$0-20) |
| Key Vault | Standard | ~$0.03/secret/month |
| Storage | Standard LRS | ~$1-5 |
| App Insights | Pay-as-you-go | ~$2-10 |
| Log Analytics | Pay-as-you-go | ~$2-5 |

**Total:** Approximately $5-40/month depending on usage.

---

## Next Steps

- [Deployment Options](../docs/DEPLOYMENT_OPTIONS.md) - Compare in-tenant vs SaaS
- [Pilot Playbook](../docs/CLIENT_PILOT_PLAYBOOK.md) - Run a client pilot
- [Security Documentation](../packages/connector/docs/SECURITY.md) - Data access details
