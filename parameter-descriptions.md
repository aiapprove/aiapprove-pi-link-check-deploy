# ARM Template Parameter Descriptions

This document explains each parameter in the `azuredeploy.json` template.

---

## Required Parameters

### `sharePointSiteUrl`

**Type:** string
**Example:** `https://contoso.sharepoint.com/sites/PICheck`

The full URL of the SharePoint site where scan results will be stored. This site should already exist before deployment.

**Where to find it:**
1. Navigate to your SharePoint site
2. Copy the URL from the browser address bar
3. Remove any trailing paths (e.g., `/SitePages/Home.aspx`)

---

## Optional Parameters (with defaults)

### `environment`

**Type:** string
**Default:** `prod`
**Allowed values:** `dev`, `staging`, `prod`

Environment identifier used in resource naming and tagging. Choose based on your deployment stage.

| Value | Use Case |
|-------|----------|
| `dev` | Development/testing |
| `staging` | Pre-production validation |
| `prod` | Production deployment |

---

### `location`

**Type:** string
**Default:** `ukwest`
**Allowed values:** `uksouth`, `ukwest`, `northeurope`, `westeurope`, `eastus`, `westus2`

Azure region where resources will be deployed.

**Recommendations:**
- **UK customers:** Use `ukwest` (UK South has consumption plan quota limits)
- **EU customers:** Use `northeurope` or `westeurope`
- **US customers:** Use `eastus` or `westus2`

**Data residency:** Choose a region that meets your compliance requirements. All data stays within the selected region.

---

### `baseName`

**Type:** string
**Default:** `pilinkcheck`
**Min length:** 3
**Max length:** 20

Base name prefix for all resources. Resources will be named like:
- `{baseName}-func-{env}-{uniqueSuffix}` (Function App)
- `{baseName}st{env}{suffix}` (Storage Account)
- `{baseName}-kv-{env}-{suffix}` (Key Vault)

**Tips:**
- Use your company/project abbreviation (e.g., `contoso-pi`)
- Keep it short to avoid Azure naming limits
- Use only lowercase letters and hyphens

---

### `sharePointSiteId`

**Type:** string
**Default:** `""` (empty)

The SharePoint site ID from Microsoft Graph API. Used for API calls.

**Where to find it:**
```bash
# Using Microsoft Graph Explorer or CLI
GET https://graph.microsoft.com/v1.0/sites/{hostname}:/sites/{site-name}
# Response includes: "id": "contoso.sharepoint.com,guid1,guid2"
```

**Note:** Can be configured post-deployment in Function App settings.

---

### `entraAppClientId`

**Type:** string
**Default:** `""` (empty)

The Client ID (Application ID) of your Entra ID app registration.

**Note:** If you don't have an app registration yet, leave this blank and configure it after deployment. See post-deployment steps.

**Where to find it:**
1. Azure Portal → Entra ID → App registrations
2. Find or create your app
3. Copy the "Application (client) ID"

---

### `adminPrincipalIds`

**Type:** array of strings
**Default:** `[]` (empty array)

Object IDs of users who should have direct access to Key Vault secrets.

**Example:**
```json
["12345678-1234-1234-1234-123456789012", "87654321-4321-4321-4321-210987654321"]
```

**Where to find user Object IDs:**
1. Azure Portal → Entra ID → Users
2. Select user → Copy "Object ID"

**Note:** The Function App's managed identity automatically gets Key Vault access. This parameter is for additional admin users who need direct access.

---

## Outputs

After deployment, the template outputs these values:

| Output | Description | Use For |
|--------|-------------|---------|
| `functionAppUrl` | Function App URL | Backend URL for Teams manifest |
| `functionAppName` | Function App resource name | Azure CLI/Portal reference |
| `functionAppPrincipalId` | Managed identity ID | Grant Sites.Selected permission |
| `keyVaultName` | Key Vault name | Secret management |
| `keyVaultUri` | Key Vault URI | App configuration |
| `storageAccountName` | Storage account name | Diagnostics |
| `appInsightsName` | App Insights name | Monitoring |

---

## Example Parameter Files

### Minimal (UK customer)

```json
{
  "sharePointSiteUrl": {
    "value": "https://contoso.sharepoint.com/sites/PICheck"
  }
}
```

### Full (EU customer with admin access)

```json
{
  "environment": { "value": "prod" },
  "location": { "value": "northeurope" },
  "baseName": { "value": "contoso-pi" },
  "sharePointSiteUrl": {
    "value": "https://contoso.sharepoint.com/sites/PILinkCheck"
  },
  "sharePointSiteId": {
    "value": "contoso.sharepoint.com,abc123,def456"
  },
  "entraAppClientId": {
    "value": "12345678-1234-1234-1234-123456789012"
  },
  "adminPrincipalIds": {
    "value": ["user-object-id-1", "user-object-id-2"]
  }
}
```
