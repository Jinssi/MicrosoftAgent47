# GitHub Copilot Instructions

This project follows **MCAPS (Microsoft Customer and Partner Solutions)** development standards.

## Authentication & Security

- **NEVER use API keys, connection strings with secrets, or hardcoded credentials**
- **Always use Managed Identity** for Azure service authentication
- Use `DefaultAzureCredential` from Azure Identity SDK which automatically handles Managed Identity
- For local development, use Azure CLI authentication (`az login`) - the developer has **Contributor** access to the subscription
- Store non-secret configuration in environment variables or Azure App Configuration
- Use Azure Key Vault with Managed Identity for any secrets that cannot be avoided

## Azure Services

- **Default to PaaS (Platform as a Service)** solutions over IaaS
- Prefer Azure-managed services: Azure Functions, App Service, Azure SQL, Cosmos DB, Azure Storage, etc.
- Avoid VMs unless absolutely necessary
- Use serverless options when appropriate (Azure Functions, Logic Apps)
- Leverage Azure's built-in security features and compliance

## Scripting & CLI

- **Default to PowerShell** for all scripting and automation
- Use Azure PowerShell modules (`Az.*`) for Azure operations
- Developer has **Contributor role** on the Azure subscription via CLI
- Use `Connect-AzAccount` with device code or managed identity for authentication
- Prefer declarative infrastructure (Bicep/ARM) over imperative scripts when possible

## Code Standards

```powershell
# Example: Correct way to authenticate to Azure services
# Uses DefaultAzureCredential which works with Managed Identity in Azure
# and falls back to Azure CLI credentials locally

$credential = Get-AzAccessToken -ResourceUrl "https://management.azure.com"
```

```csharp
// Example: Correct way to use Azure SDK with Managed Identity
using Azure.Identity;

var credential = new DefaultAzureCredential();
var client = new SecretClient(new Uri(keyVaultUrl), credential);
```

```python
# Example: Correct way to authenticate in Python
from azure.identity import DefaultAzureCredential

credential = DefaultAzureCredential()
```

## Forbidden Patterns

❌ Do NOT generate code with:
- API keys or secret keys in code or config files
- Connection strings containing passwords
- `new ClientSecretCredential()` or similar secret-based auth
- Hardcoded subscription IDs, tenant IDs in source code
- Bash scripts (use PowerShell instead)
- IaaS solutions when PaaS alternatives exist
- OpenAI API keys for Azure/Foundry models (use Entra ID tokens instead)

## Preferred Patterns

✅ Always use:
- `DefaultAzureCredential` for Azure authentication
- `get_bearer_token_provider()` with scope `https://ai.azure.com/.default` for Foundry Models
- Managed Identity for service-to-service auth
- **Cognitive Services User** role for AI/Foundry inference (not Owner/Contributor)
- Azure Key Vault references for App Service/Functions settings
- PowerShell for automation scripts
- PaaS services with managed infrastructure
- Bicep for infrastructure as code

## Environment Variables

Configuration should use environment variables or Azure App Configuration:
- `AZURE_CLIENT_ID` - Only for user-assigned managed identity (optional)
- `AZURE_TENANT_ID` - Usually auto-detected
- Service endpoints without credentials (e.g., `STORAGE_ACCOUNT_URL`, `KEY_VAULT_URL`)

## Local Development

For local development and testing:
1. Authenticate via Azure CLI: `az login`
2. `DefaultAzureCredential` will automatically use CLI credentials
3. Use the same code paths as production - no separate "local" authentication code

## Microsoft Foundry & AI Services

When working with Microsoft Foundry Models (LLMs, AI inference):

- **Use keyless authentication with Microsoft Entra ID** - never use API keys
- Requires **Cognitive Services User** role on the Foundry resource (Owner/Contributor are NOT sufficient)
- Use scope `https://ai.azure.com/.default` for token acquisition

### Python with OpenAI SDK

```python
from openai import OpenAI
from azure.identity import DefaultAzureCredential, get_bearer_token_provider

token_provider = get_bearer_token_provider(
    DefaultAzureCredential(), 
    "https://ai.azure.com/.default"
)

client = OpenAI(
    base_url="https://<resource>.openai.azure.com/openai/v1/",
    api_key=token_provider,
)

completion = client.chat.completions.create(
    model="DeepSeek-V3.1",  # Your deployment name
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello!"}
    ]
)
```

### Role Assignment (PowerShell)

```powershell
# Get resource ID
$resourceId = az resource show -g $resourceGroup -n $accountName `
    --resource-type "Microsoft.CognitiveServices/accounts" --query id -o tsv

# Get user/identity object ID
$objectId = az ad signed-in-user show --query id -o tsv

# Assign Cognitive Services User role
az role assignment create --assignee-object-id $objectId `
    --role "Cognitive Services User" --scope $resourceId
```

### Disable Key-Based Auth (Production)

```powershell
Set-AzCognitiveServicesAccount -ResourceGroupName "my-rg" `
    -Name "my-foundry-resource" -DisableLocalAuth $true
```

### Production Best Practices

- Use `ManagedIdentityCredential` directly instead of `DefaultAzureCredential` in production
- Or use `ChainedTokenCredential` with explicit credential allowlist
- Configure system-assigned or user-assigned managed identities on compute resources
- Role assignments can take up to 5 minutes to propagate
