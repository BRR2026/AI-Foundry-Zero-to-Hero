# 🧪 Hands-On Lab: Configure RBAC & Managed Identity for Azure AI Foundry

## Module 31 — Identity & Access Management

> **Arc 7: Security, Governance & Compliance** · 🔴 Advanced Tier · Estimated Time: **45–60 minutes**

---

## 🎯 Lab Objectives

By completing this lab, you will:

1. Create a user-assigned managed identity for an Azure AI Foundry project
2. Assign built-in RBAC roles with least-privilege scoping
3. Define and deploy a custom RBAC role for model operations
4. Authenticate using `DefaultAzureCredential` in Python (zero secrets)
5. Audit and verify role assignments across your resource group

---

## 📋 Prerequisites

| Requirement | Details |
|-------------|---------|
| **Azure Subscription** | With Owner or User Access Administrator role on a resource group |
| **Azure CLI** | v2.60+ installed and authenticated (`az login`) |
| **Python** | 3.9+ with `pip` |
| **VS Code** | Recommended (with Azure and Python extensions) |
| **Previous Modules** | Modules 1–30 completed (AI Foundry account from earlier labs helpful) |

---

## 🏗️ Lab Architecture

```
┌──────────────────────────────────────────────────────┐
│  Resource Group: rg-ai-foundry-iam-lab               │
│                                                      │
│  ┌──────────────────┐   ┌──────────────────────┐     │
│  │ AI Foundry       │   │ Storage Account      │     │
│  │ Account          │   │ (Blob)               │     │
│  │                  │   │                      │     │
│  │ Role: Cognitive  │   │ Role: Storage Blob   │     │
│  │ Services User    │   │ Data Reader          │     │
│  └────────┬─────────┘   └──────────┬───────────┘     │
│           │                        │                 │
│           └────────┬───────────────┘                 │
│                    │                                 │
│  ┌─────────────────▼──────────────────┐              │
│  │ User-Assigned Managed Identity     │              │
│  │ id-ai-foundry-iam-lab              │              │
│  └────────────────────────────────────┘              │
│                                                      │
│  ┌────────────────────────────────────┐               │
│  │ Custom Role:                       │               │
│  │ "AI Foundry Model Reader"          │               │
│  └────────────────────────────────────┘               │
└──────────────────────────────────────────────────────┘
```

---

## Exercise 1: Set Up the Environment (10 min)

### Step 1.1 — Define Variables

```bash
# Set your variables
RESOURCE_GROUP="rg-ai-foundry-iam-lab"
LOCATION="eastus2"
AI_ACCOUNT_NAME="ai-foundry-iam-lab-$(openssl rand -hex 4)"
STORAGE_ACCOUNT_NAME="staiiam$(openssl rand -hex 4)"
IDENTITY_NAME="id-ai-foundry-iam-lab"
SUBSCRIPTION_ID=$(az account show --query "id" -o tsv)

echo "Resource Group:   $RESOURCE_GROUP"
echo "AI Account:       $AI_ACCOUNT_NAME"
echo "Storage Account:  $STORAGE_ACCOUNT_NAME"
echo "Identity:         $IDENTITY_NAME"
echo "Subscription:     $SUBSCRIPTION_ID"
```

### Step 1.2 — Create Resource Group

```bash
az group create \
  --name "$RESOURCE_GROUP" \
  --location "$LOCATION" \
  --tags "purpose=iam-lab" "module=31"
```

### Step 1.3 — Create AI Foundry Account

```bash
az cognitiveservices account create \
  --name "$AI_ACCOUNT_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --kind "OpenAI" \
  --sku "S0" \
  --location "$LOCATION" \
  --yes
```

### Step 1.4 — Create Storage Account

```bash
az storage account create \
  --name "$STORAGE_ACCOUNT_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --location "$LOCATION" \
  --sku "Standard_LRS" \
  --kind "StorageV2" \
  --allow-blob-public-access false

# Create a sample container
az storage container create \
  --name "training-data" \
  --account-name "$STORAGE_ACCOUNT_NAME" \
  --auth-mode login
```

> ✅ **Checkpoint:** You should now have a resource group with an AI Foundry account and a storage account.

---

## Exercise 2: Create & Configure Managed Identity (10 min)

### Step 2.1 — Create User-Assigned Managed Identity

```bash
az identity create \
  --name "$IDENTITY_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --location "$LOCATION"
```

### Step 2.2 — Retrieve Identity Details

```bash
PRINCIPAL_ID=$(az identity show \
  --name "$IDENTITY_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --query "principalId" -o tsv)

IDENTITY_RESOURCE_ID=$(az identity show \
  --name "$IDENTITY_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --query "id" -o tsv)

echo "Principal ID:       $PRINCIPAL_ID"
echo "Identity Resource:  $IDENTITY_RESOURCE_ID"
```

### Step 2.3 — Assign Cognitive Services User Role

```bash
AI_ACCOUNT_ID=$(az cognitiveservices account show \
  --name "$AI_ACCOUNT_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --query "id" -o tsv)

az role assignment create \
  --assignee-object-id "$PRINCIPAL_ID" \
  --assignee-principal-type "ServicePrincipal" \
  --role "Cognitive Services User" \
  --scope "$AI_ACCOUNT_ID"
```

### Step 2.4 — Assign Storage Blob Data Reader Role

```bash
STORAGE_ACCOUNT_ID=$(az storage account show \
  --name "$STORAGE_ACCOUNT_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --query "id" -o tsv)

az role assignment create \
  --assignee-object-id "$PRINCIPAL_ID" \
  --assignee-principal-type "ServicePrincipal" \
  --role "Storage Blob Data Reader" \
  --scope "$STORAGE_ACCOUNT_ID"
```

> ✅ **Checkpoint:** The managed identity now has `Cognitive Services User` on the AI account and `Storage Blob Data Reader` on the storage account.

---

## Exercise 3: Create a Custom RBAC Role (10 min)

### Step 3.1 — Define the Custom Role JSON

Create a file named `ai-foundry-model-reader.json`:

```json
{
  "Name": "AI Foundry Model Reader",
  "IsCustom": true,
  "Description": "Can list and read model deployments but cannot create, update, or delete them.",
  "Actions": [
    "Microsoft.CognitiveServices/accounts/deployments/read",
    "Microsoft.CognitiveServices/accounts/models/read",
    "Microsoft.CognitiveServices/accounts/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read"
  ],
  "NotActions": [],
  "DataActions": [
    "Microsoft.CognitiveServices/accounts/OpenAI/models/read"
  ],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/YOUR_SUBSCRIPTION_ID"
  ]
}
```

Replace `YOUR_SUBSCRIPTION_ID` with your actual subscription ID:

```bash
sed -i "s/YOUR_SUBSCRIPTION_ID/$SUBSCRIPTION_ID/g" ai-foundry-model-reader.json
```

### Step 3.2 — Create the Custom Role

```bash
az role definition create --role-definition @ai-foundry-model-reader.json
```

### Step 3.3 — Assign the Custom Role to the Managed Identity

```bash
az role assignment create \
  --assignee-object-id "$PRINCIPAL_ID" \
  --assignee-principal-type "ServicePrincipal" \
  --role "AI Foundry Model Reader" \
  --scope "$AI_ACCOUNT_ID"
```

> ✅ **Checkpoint:** A custom "AI Foundry Model Reader" role has been created and assigned to the managed identity.

---

## Exercise 4: Validate with Python (10 min)

### Step 4.1 — Install Python Dependencies

```bash
pip install azure-identity azure-mgmt-cognitiveservices azure-storage-blob
```

### Step 4.2 — Create Validation Script

Create a file named `validate_iam.py`:

```python
"""
Module 31 Lab — Validate IAM Configuration
Verifies managed identity access to AI Foundry and Blob Storage.
"""

import os
from azure.identity import DefaultAzureCredential
from azure.mgmt.cognitiveservices import CognitiveServicesManagementClient
from azure.storage.blob import BlobServiceClient

SUBSCRIPTION_ID = os.environ.get("SUBSCRIPTION_ID", "your-subscription-id")
RESOURCE_GROUP = os.environ.get("RESOURCE_GROUP", "rg-ai-foundry-iam-lab")
AI_ACCOUNT_NAME = os.environ.get("AI_ACCOUNT_NAME", "your-ai-account-name")
STORAGE_ACCOUNT_NAME = os.environ.get("STORAGE_ACCOUNT_NAME", "your-storage-account")


def validate_ai_foundry_access(credential):
    """Verify read access to AI Foundry account."""
    print("\n🔍 Testing AI Foundry access...")
    client = CognitiveServicesManagementClient(credential, SUBSCRIPTION_ID)

    try:
        account = client.accounts.get(RESOURCE_GROUP, AI_ACCOUNT_NAME)
        print(f"  ✅ Account Name: {account.name}")
        print(f"  ✅ Kind: {account.kind}")
        print(f"  ✅ Location: {account.location}")
        print(f"  ✅ Provisioning State: {account.properties.provisioning_state}")
    except Exception as e:
        print(f"  ❌ Failed: {e}")
        return False

    # List deployments (if any)
    try:
        deployments = list(client.deployments.list(RESOURCE_GROUP, AI_ACCOUNT_NAME))
        print(f"  ✅ Deployments found: {len(deployments)}")
        for d in deployments:
            print(f"     - {d.name} ({d.properties.model.name})")
    except Exception as e:
        print(f"  ⚠️ Could not list deployments: {e}")

    return True


def validate_storage_access(credential):
    """Verify read access to Blob Storage."""
    print("\n🔍 Testing Blob Storage access...")
    blob_url = f"https://{STORAGE_ACCOUNT_NAME}.blob.core.windows.net"
    blob_client = BlobServiceClient(account_url=blob_url, credential=credential)

    try:
        containers = list(blob_client.list_containers())
        print(f"  ✅ Containers found: {len(containers)}")
        for c in containers:
            print(f"     - {c['name']}")
    except Exception as e:
        print(f"  ❌ Failed: {e}")
        return False

    return True


def validate_no_write_access(credential):
    """Verify that write access is denied (least privilege)."""
    print("\n🔍 Testing that write access is denied...")
    blob_url = f"https://{STORAGE_ACCOUNT_NAME}.blob.core.windows.net"
    blob_client = BlobServiceClient(account_url=blob_url, credential=credential)

    try:
        container_client = blob_client.get_container_client("training-data")
        container_client.upload_blob("test-file.txt", b"should fail", overwrite=True)
        print("  ❌ UNEXPECTED: Write succeeded — identity has too many permissions!")
        return False
    except Exception:
        print("  ✅ Write correctly denied — least privilege confirmed")
        return True


def main():
    print("=" * 60)
    print("Module 31 — IAM Validation Script")
    print("=" * 60)

    credential = DefaultAzureCredential()
    results = []

    results.append(("AI Foundry Access", validate_ai_foundry_access(credential)))
    results.append(("Storage Read Access", validate_storage_access(credential)))
    results.append(("Write Denied (Least Privilege)", validate_no_write_access(credential)))

    print("\n" + "=" * 60)
    print("RESULTS SUMMARY")
    print("=" * 60)
    all_passed = True
    for name, passed in results:
        status = "✅ PASS" if passed else "❌ FAIL"
        print(f"  {status} — {name}")
        if not passed:
            all_passed = False

    print("\n" + ("🎉 All checks passed!" if all_passed else "⚠️ Some checks failed. Review your configuration."))


if __name__ == "__main__":
    main()
```

### Step 4.3 — Run the Validation

```bash
export SUBSCRIPTION_ID="$(az account show --query id -o tsv)"
export RESOURCE_GROUP="rg-ai-foundry-iam-lab"
export AI_ACCOUNT_NAME="<your-ai-account-name>"
export STORAGE_ACCOUNT_NAME="<your-storage-account-name>"

python validate_iam.py
```

> ✅ **Checkpoint:** The script should confirm read access to AI Foundry and storage, and confirm that write access is denied.

---

## Exercise 5: Audit Role Assignments (5 min)

### Step 5.1 — List All Role Assignments

```bash
az role assignment list \
  --resource-group "$RESOURCE_GROUP" \
  --output table
```

### Step 5.2 — Verify Least Privilege

```bash
# Check: No identity should have Owner or Contributor at subscription scope
az role assignment list \
  --scope "/subscriptions/$SUBSCRIPTION_ID" \
  --role "Owner" \
  --output table

az role assignment list \
  --scope "/subscriptions/$SUBSCRIPTION_ID" \
  --role "Contributor" \
  --output table
```

### Step 5.3 — Review the Custom Role Definition

```bash
az role definition list \
  --custom-role-only true \
  --output table

az role definition show \
  --name "AI Foundry Model Reader" \
  --output json
```

> ✅ **Checkpoint:** You can see all role assignments and verify that no identity is over-privileged.

---

## 🧹 Cleanup

When you're done, remove all lab resources:

```bash
# Delete the resource group (removes all resources inside)
az group delete --name "$RESOURCE_GROUP" --yes --no-wait

# Delete the custom role definition
az role definition delete --name "AI Foundry Model Reader"
```

---

## ✅ Success Criteria

| # | Criteria | Status |
|---|---------|--------|
| 1 | User-assigned managed identity created | ☐ |
| 2 | `Cognitive Services User` assigned at resource scope | ☐ |
| 3 | `Storage Blob Data Reader` assigned at resource scope | ☐ |
| 4 | Custom "AI Foundry Model Reader" role created and assigned | ☐ |
| 5 | Python validation confirms read access to AI Foundry | ☐ |
| 6 | Python validation confirms read access to blob storage | ☐ |
| 7 | Python validation confirms write access is denied | ☐ |
| 8 | No identity has Owner/Contributor at subscription scope | ☐ |
| 9 | Zero API keys or secrets in code | ☐ |
| 10 | All role assignments scoped to individual resources | ☐ |

---

## 🔗 Additional Resources

- [Azure RBAC Built-in Roles](https://learn.microsoft.com/azure/role-based-access-control/built-in-roles)
- [Managed Identities for Azure Resources](https://learn.microsoft.com/entra/identity/managed-identities-azure-resources/overview)
- [Custom Roles Tutorial](https://learn.microsoft.com/azure/role-based-access-control/tutorial-custom-role-cli)
- [DefaultAzureCredential](https://learn.microsoft.com/python/api/azure-identity/azure.identity.defaultazurecredential)

---

> **Module 31 Lab** · Azure AI Foundry: Zero to Hero · Arc 7: Security, Governance & Compliance · 🔴 Advanced Tier
