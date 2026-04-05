# Module 44 — Hands-On Lab

## Complete Your Governance & Security Layer

| Field            | Details                                                          |
|------------------|------------------------------------------------------------------|
| **Module**       | 44 — Add Governance, Monitoring & Security                       |
| **Arc**          | ARC 9 — CAPSTONE & MASTERY                                      |
| **Duration**     | 60–90 minutes                                                    |
| **Difficulty**   | Advanced                                                         |
| **Goal**         | Implement RBAC, Managed Identity, monitoring, content safety, and security hardening for your AI Foundry project |

---

## Prerequisites

Before starting this lab, ensure you have:

- [ ] An active Azure subscription with **Owner** or **User Access Administrator** role on the resource group
- [ ] An **Azure AI Foundry** project with at least one model deployment (from prior modules)
- [ ] An **Azure Key Vault** resource in the same resource group
- [ ] An **Azure Storage Account** in the same resource group
- [ ] A **Log Analytics Workspace** in the same resource group
- [ ] Azure CLI 2.60+ installed locally (`az --version`)
- [ ] Python 3.10+ with the following packages installed:

```bash
pip install azure-identity azure-ai-projects azure-ai-contentsafety azure-monitor-query
```

- [ ] Access to the Azure AI Foundry portal at [https://ai.azure.com](https://ai.azure.com)

---

## Lab Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Governance & Security Layer                    │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐   │
│  │   Entra ID   │  │  Azure       │  │  Azure Monitor       │   │
│  │   Security   │  │  Policy      │  │  Dashboards & Alerts │   │
│  │   Groups     │  │  Assignments │  │  Log Analytics        │   │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬───────────┘   │
│         │                 │                      │               │
│         ▼                 ▼                      ▼               │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              Azure AI Foundry Project                    │    │
│  │  ┌─────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐  │    │
│  │  │AI Svcs  │  │Key Vault │  │Storage   │  │AI Search│  │    │
│  │  │+ OpenAI │  │(Secrets) │  │(Data)    │  │(Index)  │  │    │
│  │  └────┬────┘  └────┬─────┘  └────┬─────┘  └────┬────┘  │    │
│  │       │             │             │              │       │    │
│  │       └─────────────┴──────┬──────┴──────────────┘       │    │
│  │                            │                             │    │
│  │                   Managed Identity                       │    │
│  │                (User-Assigned UAMI)                      │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
│  ┌──────────────────┐  ┌────────────────────────────────────┐   │
│  │  Content Safety   │  │  Microsoft Defender for Cloud      │   │
│  │  Filters &        │  │  Security recommendations         │   │
│  │  Blocklists       │  │  & threat detection               │   │
│  └──────────────────┘  └────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Exercise 1 — Configure RBAC

### Step 1.1 — Create Security Groups

Create three Entra ID security groups for different team roles:

```bash
# Create security groups (requires Entra ID Group Administrator role)
az ad group create --display-name "ai-developers" \
  --mail-nickname "ai-developers" \
  --description "AI Foundry developers — deploy models and run experiments"

az ad group create --display-name "ai-operators" \
  --mail-nickname "ai-operators" \
  --description "AI Foundry operators — monitor, manage endpoints, incident response"

az ad group create --display-name "ai-auditors" \
  --mail-nickname "ai-auditors" \
  --description "AI Foundry auditors — read-only access for compliance review"
```

### Step 1.2 — Assign Roles at Resource Group Scope

```bash
# Set your variables
RG="rg-ai-foundry-capstone"
SUB_ID=$(az account show --query id -o tsv)
SCOPE="/subscriptions/$SUB_ID/resourceGroups/$RG"

# Assign Azure AI Developer to the developers group
DEV_GROUP_ID=$(az ad group show --group "ai-developers" --query id -o tsv)
az role assignment create \
  --assignee "$DEV_GROUP_ID" \
  --role "Azure AI Developer" \
  --scope "$SCOPE"

# Assign Cognitive Services OpenAI User to developers
az role assignment create \
  --assignee "$DEV_GROUP_ID" \
  --role "Cognitive Services OpenAI User" \
  --scope "$SCOPE"

# Assign Monitoring Contributor to operators
OPS_GROUP_ID=$(az ad group show --group "ai-operators" --query id -o tsv)
az role assignment create \
  --assignee "$OPS_GROUP_ID" \
  --role "Monitoring Contributor" \
  --scope "$SCOPE"

# Assign Reader to auditors
AUDIT_GROUP_ID=$(az ad group show --group "ai-auditors" --query id -o tsv)
az role assignment create \
  --assignee "$AUDIT_GROUP_ID" \
  --role "Reader" \
  --scope "$SCOPE"

# Assign Log Analytics Reader to auditors
az role assignment create \
  --assignee "$AUDIT_GROUP_ID" \
  --role "Log Analytics Reader" \
  --scope "$SCOPE"
```

### Step 1.3 — Verify Role Assignments

```bash
# List all role assignments on the resource group
az role assignment list \
  --resource-group "$RG" \
  --output table \
  --query "[].{Principal:principalName, Role:roleDefinitionName, Scope:scope}"
```

**✅ Checkpoint**: You should see at least 5 role assignments (developers: 2, operators: 1, auditors: 2).

---

## Exercise 2 — Enable Managed Identity

### Step 2.1 — Create a User-Assigned Managed Identity

```bash
# Create the user-assigned managed identity
az identity create \
  --name "uami-ai-foundry-capstone" \
  --resource-group "$RG" \
  --location "eastus2"

# Store the principal ID and client ID
UAMI_PRINCIPAL=$(az identity show \
  --name "uami-ai-foundry-capstone" \
  --resource-group "$RG" \
  --query principalId -o tsv)

UAMI_CLIENT_ID=$(az identity show \
  --name "uami-ai-foundry-capstone" \
  --resource-group "$RG" \
  --query clientId -o tsv)

echo "Principal ID: $UAMI_PRINCIPAL"
echo "Client ID: $UAMI_CLIENT_ID"
```

### Step 2.2 — Grant the Managed Identity Access to Resources

```bash
# Grant access to AI Services
AI_RESOURCE_ID=$(az cognitiveservices account show \
  --name "your-ai-services" \
  --resource-group "$RG" \
  --query id -o tsv)

az role assignment create \
  --assignee "$UAMI_PRINCIPAL" \
  --role "Cognitive Services User" \
  --scope "$AI_RESOURCE_ID"

# Grant access to Key Vault
KV_RESOURCE_ID=$(az keyvault show \
  --name "your-keyvault" \
  --resource-group "$RG" \
  --query id -o tsv)

az role assignment create \
  --assignee "$UAMI_PRINCIPAL" \
  --role "Key Vault Secrets User" \
  --scope "$KV_RESOURCE_ID"

# Grant access to Storage
STORAGE_RESOURCE_ID=$(az storage account show \
  --name "yourstorageaccount" \
  --resource-group "$RG" \
  --query id -o tsv)

az role assignment create \
  --assignee "$UAMI_PRINCIPAL" \
  --role "Storage Blob Data Contributor" \
  --scope "$STORAGE_RESOURCE_ID"
```

### Step 2.3 — Test Managed Identity in Code

Create `test_managed_identity.py`:

```python
from azure.identity import DefaultAzureCredential, ManagedIdentityCredential
from azure.keyvault.secrets import SecretClient
from azure.storage.blob import BlobServiceClient
import os

# Use DefaultAzureCredential — automatically picks up managed identity in Azure
credential = DefaultAzureCredential()

# Test 1: Access Key Vault
kv_url = "https://your-keyvault.vault.azure.net"
kv_client = SecretClient(vault_url=kv_url, credential=credential)
print("✅ Key Vault access: Connected successfully")

# Test 2: Access Storage
storage_url = "https://yourstorageaccount.blob.core.windows.net"
blob_client = BlobServiceClient(account_url=storage_url, credential=credential)
containers = list(blob_client.list_containers(max_results=5))
print(f"✅ Storage access: Found {len(containers)} containers")

print("\n🎉 Managed Identity working — no API keys needed!")
```

Run the test:

```bash
python test_managed_identity.py
```

**✅ Checkpoint**: Both tests should pass with no API keys in the code.

---

## Exercise 3 — Build a Monitoring Dashboard

### Step 3.1 — Enable Diagnostic Settings

```bash
# Get the Log Analytics workspace ID
WORKSPACE_ID=$(az monitor log-analytics workspace show \
  --workspace-name "law-ai-foundry-capstone" \
  --resource-group "$RG" \
  --query id -o tsv)

# Enable diagnostics on AI Services
az monitor diagnostic-settings create \
  --name "diag-ai-services" \
  --resource "$AI_RESOURCE_ID" \
  --workspace "$WORKSPACE_ID" \
  --logs '[{"categoryGroup": "allLogs", "enabled": true}]' \
  --metrics '[{"category": "AllMetrics", "enabled": true}]'

# Enable diagnostics on Key Vault
az monitor diagnostic-settings create \
  --name "diag-keyvault" \
  --resource "$KV_RESOURCE_ID" \
  --workspace "$WORKSPACE_ID" \
  --logs '[{"categoryGroup": "allLogs", "enabled": true}]' \
  --metrics '[{"category": "AllMetrics", "enabled": true}]'
```

### Step 3.2 — Create KQL Queries for the Dashboard

Save these queries and add them to an Azure Monitor Workbook in the portal:

**Query 1 — API Latency Percentiles:**

```kql
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
| where Category == "RequestResponse"
| where TimeGenerated > ago(24h)
| summarize
    P50 = percentile(DurationMs, 50),
    P95 = percentile(DurationMs, 95),
    P99 = percentile(DurationMs, 99),
    RequestCount = count()
  by bin(TimeGenerated, 1h)
| order by TimeGenerated asc
```

**Query 2 — Error Rate by HTTP Status:**

```kql
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
| where TimeGenerated > ago(24h)
| summarize
    Total = count(),
    Success = countif(httpStatusCode_d >= 200 and httpStatusCode_d < 300),
    ClientErrors = countif(httpStatusCode_d >= 400 and httpStatusCode_d < 500),
    ServerErrors = countif(httpStatusCode_d >= 500),
    Throttled = countif(httpStatusCode_d == 429)
  by bin(TimeGenerated, 15m)
| extend ErrorRate = round(100.0 * (ClientErrors + ServerErrors) / Total, 2)
```

**Query 3 — Token Consumption:**

```kql
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
| where Category == "RequestResponse"
| where TimeGenerated > ago(7d)
| summarize
    TotalPromptTokens = sum(toint(properties_s has "promptTokens")),
    TotalRequests = count()
  by bin(TimeGenerated, 1d)
```

**Query 4 — Content Safety Filter Activity:**

```kql
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
| where TimeGenerated > ago(24h)
| where properties_s has "contentFilter"
| summarize BlockedCount = count() by bin(TimeGenerated, 1h)
```

### Step 3.3 — Create the Workbook

1. Navigate to **Azure Monitor** → **Workbooks** → **+ New**
2. Add a **Text** step with the title "AI Foundry Operations Dashboard"
3. Add a **Query** step for each of the four KQL queries above
4. Set visualization types: Line chart (latency), Bar chart (errors), Area chart (tokens), Table (content safety)
5. Add a **Parameters** step with a time range picker (default: Last 24 hours)
6. Save the workbook as "AI Foundry Capstone — Operations Dashboard"

**✅ Checkpoint**: Your workbook should display four panels with query results (data may take 15–30 minutes to appear after enabling diagnostics).

---

## Exercise 4 — Configure Alerts

### Step 4.1 — Create an Action Group

```bash
# Create the action group
az monitor action-group create \
  --name "ag-ai-ops-capstone" \
  --resource-group "$RG" \
  --short-name "AIOps" \
  --action email ai-team "your-email@example.com"
```

### Step 4.2 — Create Metric Alerts

```bash
# Alert 1: High API Latency (> 5 seconds average over 5 minutes)
az monitor metrics alert create \
  --name "alert-high-latency-capstone" \
  --resource-group "$RG" \
  --scopes "$AI_RESOURCE_ID" \
  --condition "avg Latency > 5000" \
  --window-size "5m" \
  --evaluation-frequency "1m" \
  --severity 2 \
  --action "ag-ai-ops-capstone" \
  --description "Average API latency exceeds 5 seconds"

# Alert 2: High Error Rate (> 10 server errors in 10 minutes)
az monitor metrics alert create \
  --name "alert-error-spike-capstone" \
  --resource-group "$RG" \
  --scopes "$AI_RESOURCE_ID" \
  --condition "total ServerErrors > 10" \
  --window-size "10m" \
  --evaluation-frequency "5m" \
  --severity 1 \
  --action "ag-ai-ops-capstone" \
  --description "Server error count exceeds 10 in 10 minutes"

# Alert 3: Throttling (> 20 throttled requests in 5 minutes)
az monitor metrics alert create \
  --name "alert-throttling-capstone" \
  --resource-group "$RG" \
  --scopes "$AI_RESOURCE_ID" \
  --condition "total ClientErrors > 20" \
  --window-size "5m" \
  --evaluation-frequency "1m" \
  --severity 2 \
  --action "ag-ai-ops-capstone" \
  --description "Client errors (including 429 throttling) exceed 20 in 5 minutes"
```

### Step 4.3 — Verify Alerts

```bash
# List all alerts in the resource group
az monitor metrics alert list \
  --resource-group "$RG" \
  --output table \
  --query "[].{Name:name, Severity:severity, Enabled:enabled}"
```

**✅ Checkpoint**: Three alerts should be listed, all with `Enabled: true`.

---

## Exercise 5 — Apply Content Safety Filters

### Step 5.1 — Analyze Text with Content Safety

Create `test_content_safety.py`:

```python
from azure.ai.contentsafety import ContentSafetyClient
from azure.ai.contentsafety.models import AnalyzeTextOptions, TextCategory
from azure.identity import DefaultAzureCredential

credential = DefaultAzureCredential()
client = ContentSafetyClient(
    endpoint="https://your-content-safety.cognitiveservices.azure.com",
    credential=credential,
)

# Test with a safe message
safe_text = "How do I configure monitoring for my AI application?"
request = AnalyzeTextOptions(
    text=safe_text,
    categories=[
        TextCategory.HATE,
        TextCategory.VIOLENCE,
        TextCategory.SELF_HARM,
        TextCategory.SEXUAL,
    ],
)

response = client.analyze_text(request)
print("=== Safe Text Analysis ===")
for result in response.categories_analysis:
    print(f"  {result.category}: severity {result.severity}")

# Verify all severities are 0 (safe)
all_safe = all(r.severity == 0 for r in response.categories_analysis)
print(f"\n{'✅' if all_safe else '❌'} All categories safe: {all_safe}")
```

Run the test:

```bash
python test_content_safety.py
```

### Step 5.2 — Configure Content Filters on Model Deployment

In the Azure AI Foundry portal:

1. Navigate to your project → **Deployments**
2. Select your model deployment (e.g., `gpt-4o`)
3. Click **Edit** → **Content filters**
4. Configure severity thresholds:
   - **Hate**: Medium and above → Block
   - **Violence**: Medium and above → Block
   - **Sexual**: Low and above → Block
   - **Self-harm**: Low and above → Block
5. Enable **Prompt Shield** for jailbreak detection
6. Enable **Protected material** detection for text
7. Save the configuration

### Step 5.3 — Create a Custom Blocklist (Optional)

```bash
# Create a blocklist
az rest --method PUT \
  --url "https://your-content-safety.cognitiveservices.azure.com/contentsafety/text/blocklists/capstone-blocklist?api-version=2024-09-01" \
  --headers "Content-Type=application/json" \
  --body '{"description": "Capstone project custom blocklist"}'

# Add terms to the blocklist
az rest --method POST \
  --url "https://your-content-safety.cognitiveservices.azure.com/contentsafety/text/blocklists/capstone-blocklist:addOrUpdateBlocklistItems?api-version=2024-09-01" \
  --headers "Content-Type=application/json" \
  --body '{"blocklistItems": [{"description": "Test blocked term", "text": "blockedterm123"}]}'
```

**✅ Checkpoint**: Content safety analysis should return severity 0 for safe text, and your model deployment should have content filters configured.

---

## Exercise 6 — Run the Security Hardening Checklist

Complete the following checklist and document your compliance status. Mark each item as ✅ (compliant), ⚠️ (partial), or ❌ (non-compliant):

### Identity & Access

| # | Control | Status | Notes |
|---|---------|--------|-------|
| 1 | Managed Identity enabled on all AI resources | | |
| 2 | No API keys in source code or config files | | |
| 3 | RBAC assigned at resource group scope (not subscription) | | |
| 4 | Security groups used for role assignments | | |
| 5 | MFA enforced via Conditional Access | | |

### Network

| # | Control | Status | Notes |
|---|---------|--------|-------|
| 6 | Private endpoints configured for AI Services | | |
| 7 | Public network access disabled | | |
| 8 | TLS 1.2+ enforced | | |
| 9 | NSG rules restrict to required traffic only | | |

### Data Protection

| # | Control | Status | Notes |
|---|---------|--------|-------|
| 10 | Encryption at rest enabled (default or CMK) | | |
| 11 | Key Vault has soft delete and purge protection | | |
| 12 | Storage account requires secure transfer | | |

### Monitoring

| # | Control | Status | Notes |
|---|---------|--------|-------|
| 13 | Diagnostic settings enabled on all resources | | |
| 14 | Log retention ≥ 90 days | | |
| 15 | At least 3 metric alerts configured | | |
| 16 | Action group with valid notification targets | | |

### Content Safety

| # | Control | Status | Notes |
|---|---------|--------|-------|
| 17 | Content filters configured on all deployments | | |
| 18 | Prompt Shield enabled | | |
| 19 | Protected material detection enabled | | |

### Verify Key Vault Configuration

```bash
# Check soft delete and purge protection
az keyvault show \
  --name "your-keyvault" \
  --query "{SoftDelete:properties.enableSoftDelete, PurgeProtection:properties.enablePurgeProtection}" \
  --output table
```

### Verify Storage Security

```bash
# Check HTTPS-only and minimum TLS version
az storage account show \
  --name "yourstorageaccount" \
  --resource-group "$RG" \
  --query "{HttpsOnly:enableHttpsTrafficOnly, MinTLS:minimumTlsVersion}" \
  --output table
```

**✅ Checkpoint**: At least 17 of 19 items should be ✅ (compliant) for a passing score.

---

## Clean Up (Optional)

If this is a training environment, clean up the test resources:

```bash
# Remove test role assignments (keep production ones)
# Remove test content safety blocklist
az rest --method DELETE \
  --url "https://your-content-safety.cognitiveservices.azure.com/contentsafety/text/blocklists/capstone-blocklist?api-version=2024-09-01"

# Delete test Python files
rm test_managed_identity.py test_content_safety.py
```

> **Note**: Do NOT delete the security groups, role assignments, diagnostic settings, or alerts if you are preparing for the Module 45 capstone review. These are required for the final evaluation.

---

## Summary

In this lab you:

1. ✅ Created three Entra ID security groups and assigned appropriate RBAC roles
2. ✅ Configured a user-assigned managed identity with access to Key Vault, Storage, and AI Services
3. ✅ Built an Azure Monitor workbook with four KQL-powered panels
4. ✅ Configured three metric alerts with an action group
5. ✅ Tested content safety analysis and configured model deployment filters
6. ✅ Completed a 19-point security hardening checklist

Your governance and security layer is ready for the capstone review in Module 45!
