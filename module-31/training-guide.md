# Module 31 — Identity & Access Management

## Azure AI Foundry: Zero to Hero (45 Modules)

> **Arc 7: Security, Governance & Compliance** · 🔴 **Advanced Tier** · April 2026

---

## 🎉 Welcome to the Advanced Tier!

Congratulations on completing the Beginner and Intermediate tiers! Module 31 marks your entry into **Advanced** territory — enterprise-grade patterns that separate learning projects from production-ready AI systems. We start Arc 7 with the most critical security foundation: **Identity and Access Management (IAM)**.

---

## 📋 Module Overview

| Field | Detail |
|-------|--------|
| **Module** | 31 of 45 |
| **Title** | Identity & Access Management |
| **Arc** | 7 — Security, Governance & Compliance |
| **Tier** | 🔴 Advanced |
| **Duration** | 75 minutes (reading) + 30–40 minutes (Mini Hack) |
| **Prerequisites** | Modules 1–30 completed; Azure subscription with Owner or User Access Administrator role |
| **Video** | [AI Foundry Networking and Security: Deep Dive](https://www.youtube.com/watch?v=bL9n83uPV84) |

---

## 🎯 Learning Objectives

By the end of this module, you will be able to:

1. **Explain** the RBAC model for Azure AI Foundry and identify appropriate built-in roles for different personas
2. **Configure** system-assigned and user-assigned managed identities for service-to-service authentication
3. **Set up** service principals and Workload Identity Federation for CI/CD pipelines
4. **Apply** least-privilege access patterns across subscriptions, resource groups, and resources
5. **Create** custom RBAC role definitions tailored to specific AI Foundry requirements

---

## 📚 Training Content

### 1. Why IAM Matters for AI Foundry

Identity and Access Management controls **who** can access your AI resources, **what** they can do, and **how** services authenticate to each other. Over 80% of cloud security breaches involve misconfigured access — making IAM your most critical defense layer.

The three pillars of AI Foundry IAM:

- **Role-Based Access Control (RBAC):** Fine-grained permissions via built-in or custom roles
- **Managed Identities:** Credential-free service-to-service authentication
- **Service Principals:** Non-interactive identities for CI/CD and automation

---

### 2. RBAC Roles for Azure AI Foundry

#### Core Azure Roles

| Role | Permissions | When to Use |
|------|-------------|-------------|
| **Owner** | Full access + can assign roles | Subscription admins only (use sparingly) |
| **Contributor** | Create/manage resources, cannot assign roles | Developers building AI Foundry projects |
| **Reader** | View-only access | Auditors, stakeholders, monitoring |

#### AI Foundry-Specific Roles

| Role | Key Permissions | Best For |
|------|----------------|----------|
| **Azure AI Developer** | Create deployments, manage endpoints, experiments | Data scientists, ML engineers |
| **Azure AI Inference Deployment Operator** | Deploy models, manage inference endpoints | MLOps production teams |
| **Cognitive Services User** | Call AI APIs, list keys | Applications consuming endpoints |
| **Cognitive Services Contributor** | Create/manage Cognitive Services resources | Teams provisioning AI services |
| **Search Index Data Reader** | Query search indexes | RAG applications |
| **Storage Blob Data Contributor** | Read/write/delete blob data | AI Foundry accessing training data |

#### Assigning Roles via CLI

```bash
# Assign Azure AI Developer role at resource group scope
az role assignment create \
  --assignee "user@contoso.com" \
  --role "Azure AI Developer" \
  --scope "/subscriptions/{sub-id}/resourceGroups/{rg-name}"

# Assign Cognitive Services User to a managed identity
az role assignment create \
  --assignee-object-id "{principal-id}" \
  --assignee-principal-type "ServicePrincipal" \
  --role "Cognitive Services User" \
  --scope "/subscriptions/{sub-id}/resourceGroups/{rg-name}/providers/Microsoft.CognitiveServices/accounts/{account}"

# List all role assignments for a resource group
az role assignment list --resource-group "{rg-name}" --output table
```

#### Custom Roles

When built-in roles are too broad or narrow, define custom roles:

```json
{
  "Name": "AI Foundry Model Deployer",
  "Description": "Deploy and manage model endpoints only",
  "Actions": [
    "Microsoft.CognitiveServices/accounts/deployments/*",
    "Microsoft.CognitiveServices/accounts/models/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read"
  ],
  "NotActions": [
    "Microsoft.CognitiveServices/accounts/delete",
    "Microsoft.CognitiveServices/accounts/write"
  ],
  "DataActions": [
    "Microsoft.CognitiveServices/accounts/OpenAI/deployments/completions/action",
    "Microsoft.CognitiveServices/accounts/OpenAI/deployments/chat/completions/action"
  ],
  "AssignableScopes": ["/subscriptions/{subscription-id}"]
}
```

```bash
az role definition create --role-definition @custom-ai-foundry-role.json
```

> 💡 **Best Practice:** Always start with the narrowest built-in role. Only create custom roles when built-in roles genuinely cannot satisfy your requirements.

---

### 3. Managed Identity for Service-to-Service Auth

Managed identities eliminate credential storage. Azure handles token issuance and rotation automatically.

#### System-Assigned vs. User-Assigned

| Feature | System-Assigned | User-Assigned |
|---------|----------------|---------------|
| **Lifecycle** | Tied to resource | Independent resource |
| **Sharing** | Cannot share | Share across resources |
| **Best For** | Simple, single-service | Production, multi-service |
| **Management** | Automatic | Requires explicit management |

#### Authentication Flow

```
AI Foundry Project → Azure AD Token Service → Access Token (auto-rotated) → Target Service
```

No credentials stored or managed by your code.

#### CLI Commands

```bash
# Enable system-assigned managed identity
az cognitiveservices account identity assign \
  --name "my-ai-foundry-account" \
  --resource-group "rg-ai-foundry-prod"

# Create user-assigned managed identity
az identity create \
  --name "id-ai-foundry-prod" \
  --resource-group "rg-ai-foundry-prod" \
  --location "eastus2"

# Grant Storage Blob Data Contributor
PRINCIPAL_ID=$(az identity show \
  --name "id-ai-foundry-prod" \
  --resource-group "rg-ai-foundry-prod" \
  --query "principalId" -o tsv)

az role assignment create \
  --assignee-object-id "$PRINCIPAL_ID" \
  --assignee-principal-type "ServicePrincipal" \
  --role "Storage Blob Data Contributor" \
  --scope "/subscriptions/{sub-id}/resourceGroups/rg-ai-foundry-prod/providers/Microsoft.Storage/storageAccounts/staidata"
```

#### Python Example

```python
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient
from azure.storage.blob import BlobServiceClient

# Works with managed identity in Azure, CLI auth locally
credential = DefaultAzureCredential()

ai_client = AIProjectClient(
    credential=credential,
    subscription_id="your-subscription-id",
    resource_group_name="rg-ai-foundry-prod",
    project_name="my-ai-project",
)

blob_client = BlobServiceClient(
    account_url="https://staidata.blob.core.windows.net",
    credential=credential,
)

for container in blob_client.list_containers():
    print(f"Container: {container['name']}")
```

---

### 4. Service Principals & CI/CD Authentication

Service principals are required for code running outside Azure — CI/CD pipelines, on-premises automation, and external integrations.

#### Creating a Service Principal

```bash
az ad sp create-for-rbac \
  --name "sp-ai-foundry-cicd" \
  --role "Contributor" \
  --scopes "/subscriptions/{sub-id}/resourceGroups/rg-ai-foundry-prod" \
  --sdk-auth
```

> ⚠️ **Security:** Prefer **Workload Identity Federation** over client secrets. Federation eliminates long-lived secrets by exchanging OIDC tokens.

#### Workload Identity Federation (GitHub Actions)

```bash
# Create federated credential
az ad app federated-credential create \
  --id "{app-object-id}" \
  --parameters '{
    "name": "github-actions-deploy",
    "issuer": "https://token.actions.githubusercontent.com",
    "subject": "repo:contoso/ai-foundry-app:ref:refs/heads/main",
    "audiences": ["api://AzureADTokenExchange"]
  }'
```

**GitHub Actions workflow:**

```yaml
name: Deploy AI Foundry Model
on:
  push:
    branches: [main]

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Azure Login (Federated)
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Deploy Model Endpoint
        run: |
          az cognitiveservices account deployment create \
            --name my-ai-foundry-account \
            --resource-group rg-ai-foundry-prod \
            --deployment-name gpt-4o-prod \
            --model-name gpt-4o \
            --model-version "2024-11-20" \
            --model-format OpenAI \
            --sku-capacity 10 \
            --sku-name GlobalStandard
```

---

### 5. Least-Privilege Access Patterns

Every identity should have **only the minimum permissions required** for its function.

#### Access Pattern Matrix

| Persona | Roles | Scope | Auth Method |
|---------|-------|-------|-------------|
| Platform Admin | Owner (with PIM/JIT) | Subscription | Entra ID + MFA |
| AI Engineer | Azure AI Developer | Resource Group | Entra ID + MFA |
| Data Scientist | Cognitive Services User + Blob Data Reader | Resource | Entra ID + MFA |
| Application (Prod) | Cognitive Services User | Resource | Managed Identity |
| CI/CD Pipeline | Custom "Model Deployer" | Resource Group | Workload Identity Federation |
| Monitoring/Audit | Reader + Log Analytics Reader | Resource Group | Entra ID |

#### Security Hardening Checklist

- [ ] Enable Privileged Identity Management (PIM) for admin access
- [ ] Require MFA via Conditional Access for all human identities
- [ ] Replace API keys with Managed Identity or Workload Identity Federation
- [ ] Scope role assignments to the narrowest resource possible
- [ ] Audit role assignments quarterly with Access Reviews
- [ ] Set expiration on role assignments via PIM
- [ ] Enable diagnostic logging for identity operations
- [ ] Use separate identities for dev, staging, and production

---

## 🧪 Mini Hack: Configure RBAC & Managed Identity

**Estimated time: 30–40 minutes**

### Steps

1. **Create a Resource Group and AI Foundry Account** (or reuse from previous modules)
2. **Create a User-Assigned Managed Identity** named `id-ai-foundry-hack`
3. **Assign RBAC Roles**: `Cognitive Services User` on the AI account + `Storage Blob Data Reader` on storage
4. **Create a Custom Role**: "AI Foundry Model Reader" — list/read deployments only
5. **Validate with Python**: Use `DefaultAzureCredential` to authenticate and verify access
6. **Audit**: Run `az role assignment list` and document your findings

### Success Criteria

- ✅ Managed identity can call AI Foundry endpoints and read blobs
- ✅ Custom role restricts deployment creation
- ✅ No identity has Owner/Contributor at subscription scope
- ✅ All authentication uses managed identity (zero API keys)

---

## 📖 Additional Resources

- [Azure AI Foundry RBAC Documentation](https://learn.microsoft.com/azure/ai-services/openai/how-to/role-based-access-control)
- [Managed Identities Overview](https://learn.microsoft.com/entra/identity/managed-identities-azure-resources/overview)
- [Workload Identity Federation](https://learn.microsoft.com/entra/workload-id/workload-identity-federation)
- [Custom Azure Roles](https://learn.microsoft.com/azure/role-based-access-control/custom-roles)
- [Azure PIM](https://learn.microsoft.com/entra/id-governance/privileged-identity-management/pim-configure)

---

## 🔮 What's Next?

**Module 32** builds on IAM foundations to cover **network security** — private endpoints, VNETs, and securing AI Foundry traffic at the network layer.

---

> **Azure AI Foundry: Zero to Hero** · Module 31 of 45 · Arc 7: Security, Governance & Compliance  
> 🔴 Advanced Tier · April 2026
