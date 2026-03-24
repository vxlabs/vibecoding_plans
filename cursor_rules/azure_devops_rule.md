# No Direct Azure Resource Modifications

Never directly modify live Azure resources. This includes but is not limited to:

- Key Vault secrets (e.g. `az keyvault secret set`)
- Azure SQL databases, firewall rules, or SKU changes
- App Service / Function App configuration or application settings
- Storage accounts, Service Bus, or any other Azure resource

## What to do instead

Describe the change that needs to be made and provide the exact value/command so the user can apply it themselves:

```
# ❌ NEVER do this
az keyvault secret set --vault-name "yourvalue name" --name "my-secret" --value "new-value"

# ✅ ALWAYS do this instead
Tell the user:
"Update Key Vault secret `my-secret` in vault `kv-ldmkeys-prod` to:
  new-value"
```

This applies to ALL environments: dev, beta/staging, and production.