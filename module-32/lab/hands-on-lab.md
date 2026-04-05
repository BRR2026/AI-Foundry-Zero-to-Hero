# Hands-On Lab: Data Governance & Purview Integration

## Module 32 — Azure AI Foundry: Zero to Hero

> **ARC 7 · SECURITY, GOVERNANCE & COMPLIANCE** | Estimated Duration: 60 minutes

---

## 🎯 Lab Objectives

By the end of this lab, you will have:

1. ✅ Provisioned a Microsoft Purview account for AI data governance
2. ✅ Registered Azure AI Foundry data sources in Purview
3. ✅ Configured and executed an automated classification scan
4. ✅ Created a custom classifier for AI-specific data patterns
5. ✅ Applied sensitivity labels to classified data assets
6. ✅ Explored data lineage for an AI pipeline
7. ✅ Configured data access policies for AI teams

---

## 📋 Prerequisites

- [ ] Azure subscription with **Contributor** role
- [ ] Azure CLI installed (`az --version` ≥ 2.55)
- [ ] Python 3.9+ with `azure-identity` and `azure-purview-catalog` packages
- [ ] Completion of Module 31 (Network Security)
- [ ] Familiarity with Azure AI Foundry project concepts

### Required Azure Providers

```bash
# Register required resource providers
az provider register --namespace Microsoft.Purview
az provider register --namespace Microsoft.Storage
az provider register --namespace Microsoft.MachineLearningServices
```

---

## Lab Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    RESOURCE GROUP                        │
│                  rg-purview-lab-32                       │
│                                                         │
│  ┌──────────────┐  ┌───────────────┐  ┌──────────────┐ │
│  │   Purview     │  │   Storage     │  │  AI Foundry  │ │
│  │   Account     │──│   Account     │──│   Project    │ │
│  │              │  │ (AI data)     │  │  (optional)  │ │
│  └──────┬───────┘  └───────────────┘  └──────────────┘ │
│         │                                               │
│  ┌──────┴───────┐                                       │
│  │ Classification│                                      │
│  │ + Lineage     │                                      │
│  │ + Policies    │                                      │
│  └──────────────┘                                       │
└─────────────────────────────────────────────────────────┘
```

---

## Exercise 1: Provision Purview Account (10 min)

### Step 1.1: Create Resource Group

```bash
# Set variables
RESOURCE_GROUP="rg-purview-lab-32"
LOCATION="eastus"
PURVIEW_ACCOUNT="purview-ai-lab-${RANDOM}"
STORAGE_ACCOUNT="aitraindata${RANDOM}"

# Create resource group
az group create \
    --name $RESOURCE_GROUP \
    --location $LOCATION \
    --tags "module=32" "series=zero-to-hero" "arc=7"

echo "Resource group created: $RESOURCE_GROUP"
```

### Step 1.2: Create Microsoft Purview Account

```bash
# Create Purview account
az purview account create \
    --account-name $PURVIEW_ACCOUNT \
    --resource-group $RESOURCE_GROUP \
    --location $LOCATION \
    --managed-group-name "${PURVIEW_ACCOUNT}-managed-rg"

# Wait for provisioning (this takes 3-5 minutes)
echo "Waiting for Purview provisioning..."
az purview account wait \
    --account-name $PURVIEW_ACCOUNT \
    --resource-group $RESOURCE_GROUP \
    --created

# Verify deployment
az purview account show \
    --account-name $PURVIEW_ACCOUNT \
    --resource-group $RESOURCE_GROUP \
    --query "{name:name, status:provisioningState, endpoint:endpoints.catalog}" \
    --output table
```

### Step 1.3: Assign Governance Roles

```bash
# Get your user object ID
USER_OBJECT_ID=$(az ad signed-in-user show --query id -o tsv)

# Get Purview account resource ID
PURVIEW_ID=$(az purview account show \
    --account-name $PURVIEW_ACCOUNT \
    --resource-group $RESOURCE_GROUP \
    --query id -o tsv)

# Assign Purview Data Curator role
az role assignment create \
    --role "Purview Data Curator" \
    --assignee-object-id $USER_OBJECT_ID \
    --scope $PURVIEW_ID

# Assign Purview Data Source Administrator role
az role assignment create \
    --role "Purview Data Source Administrator" \
    --assignee-object-id $USER_OBJECT_ID \
    --scope $PURVIEW_ID

echo "Governance roles assigned successfully"
```

> **✅ Checkpoint 1:** Verify Purview account shows `provisioningState: Succeeded` and you have Data Curator + Data Source Admin roles.

---

## Exercise 2: Prepare AI Training Data (10 min)

### Step 2.1: Create Storage Account

```bash
# Create storage account for AI training data
az storage account create \
    --name $STORAGE_ACCOUNT \
    --resource-group $RESOURCE_GROUP \
    --location $LOCATION \
    --sku Standard_LRS \
    --kind StorageV2 \
    --min-tls-version TLS1_2

# Create containers for different AI data categories
az storage container create \
    --name training-data \
    --account-name $STORAGE_ACCOUNT \
    --auth-mode login

az storage container create \
    --name model-artifacts \
    --account-name $STORAGE_ACCOUNT \
    --auth-mode login

az storage container create \
    --name evaluation-data \
    --account-name $STORAGE_ACCOUNT \
    --auth-mode login

echo "Storage account and containers created"
```

### Step 2.2: Upload Sample Data with Sensitive Information

```bash
# Create sample customer dataset (contains PII for classification testing)
cat > sample-customers.csv << 'EOF'
id,name,email,ssn,phone,address,diagnosis,insurance_id
1,John Smith,john.smith@example.com,123-45-6789,555-0101,123 Main St New York NY 10001,Type 2 Diabetes,INS-2024-001
2,Jane Doe,jane.doe@example.com,987-65-4321,555-0102,456 Oak Ave Boston MA 02101,Hypertension,INS-2024-002
3,Robert Johnson,rjohnson@example.com,456-78-9012,555-0103,789 Pine Rd Chicago IL 60601,Asthma,INS-2024-003
4,Emily Williams,ewilliams@example.com,321-54-9876,555-0104,321 Elm St Austin TX 73301,Migraine,INS-2024-004
5,Michael Brown,mbrown@example.com,654-32-1098,555-0105,654 Maple Dr Seattle WA 98101,Arthritis,INS-2024-005
EOF

# Create sample financial transaction data
cat > sample-transactions.csv << 'EOF'
transaction_id,customer_id,credit_card,amount,merchant,date,category
TXN-001,1,4532-1234-5678-9012,125.50,Amazon,2026-03-15,retail
TXN-002,2,5412-9876-5432-1098,89.99,Walmart,2026-03-16,grocery
TXN-003,3,4716-5678-1234-9876,250.00,BestBuy,2026-03-17,electronics
TXN-004,1,4532-1234-5678-9012,45.00,Starbucks,2026-03-18,food
TXN-005,4,6011-1234-5678-9012,1500.00,Delta Airlines,2026-03-19,travel
EOF

# Create sample model metadata
cat > model-metadata.json << 'EOF'
{
    "model_name": "customer-churn-predictor",
    "version": "2.1.0",
    "training_data": "training-data/customers/",
    "training_date": "2026-04-01",
    "framework": "azure-ai-foundry",
    "metrics": {
        "accuracy": 0.94,
        "f1_score": 0.91,
        "auc_roc": 0.96
    },
    "data_governance": {
        "classification_scan": "completed",
        "sensitivity_labels": ["Confidential - AI Training"],
        "lineage_tracked": true
    }
}
EOF

# Upload to Blob Storage
az storage blob upload-batch \
    --destination training-data \
    --source . \
    --pattern "sample-*.csv" \
    --account-name $STORAGE_ACCOUNT \
    --auth-mode login

az storage blob upload \
    --container-name model-artifacts \
    --file model-metadata.json \
    --name models/churn/metadata.json \
    --account-name $STORAGE_ACCOUNT \
    --auth-mode login

echo "Sample data uploaded successfully"
```

> **✅ Checkpoint 2:** Verify all three containers exist and contain the uploaded files.

---

## Exercise 3: Register Data Sources & Configure Scans (15 min)

### Step 3.1: Grant Purview Access to Storage

```bash
# Get Purview managed identity
PURVIEW_PRINCIPAL_ID=$(az purview account show \
    --account-name $PURVIEW_ACCOUNT \
    --resource-group $RESOURCE_GROUP \
    --query "identity.principalId" -o tsv)

# Get storage account resource ID
STORAGE_ID=$(az storage account show \
    --name $STORAGE_ACCOUNT \
    --resource-group $RESOURCE_GROUP \
    --query id -o tsv)

# Grant Storage Blob Data Reader role to Purview
az role assignment create \
    --role "Storage Blob Data Reader" \
    --assignee-object-id $PURVIEW_PRINCIPAL_ID \
    --assignee-principal-type ServicePrincipal \
    --scope $STORAGE_ID

echo "Purview access to storage configured"
```

### Step 3.2: Register Data Source via Python SDK

```python
# save as: register_data_source.py
from azure.purview.scanning import PurviewScanningClient
from azure.identity import DefaultAzureCredential
import os

credential = DefaultAzureCredential()
purview_account = os.environ.get("PURVIEW_ACCOUNT", "purview-ai-lab")

scanning_client = PurviewScanningClient(
    endpoint=f"https://{purview_account}.purview.azure.com",
    credential=credential
)

# Register Azure Blob Storage as data source
storage_account = os.environ.get("STORAGE_ACCOUNT", "aitrainingdata")
data_source_body = {
    "kind": "AzureStorage",
    "properties": {
        "endpoint": f"https://{storage_account}.blob.core.windows.net",
        "collection": {
            "referenceName": "ai-training-assets",
            "type": "CollectionReference"
        }
    }
}

result = scanning_client.data_sources.create_or_update(
    data_source_name="ai-training-storage",
    body=data_source_body
)
print(f"Data source registered: {result['name']}")

# Configure classification scan
scan_body = {
    "kind": "AzureStorageCredentialScan",
    "properties": {
        "scanRulesetName": "AzureStorage",
        "credential": {
            "referenceName": "managed-identity",
            "credentialType": "ManagedIdentity"
        },
        "collection": {
            "referenceName": "ai-training-assets",
            "type": "CollectionReference"
        }
    }
}

scan_result = scanning_client.scans.create_or_update(
    data_source_name="ai-training-storage",
    scan_name="weekly-classification-scan",
    body=scan_body
)
print(f"Scan configured: {scan_result['name']}")

# Trigger scan run
run_result = scanning_client.scan_result.run_scan(
    data_source_name="ai-training-storage",
    scan_name="weekly-classification-scan",
    run_id="manual-run-001"
)
print(f"Scan triggered: {run_result}")
```

### Step 3.3: Run the Registration Script

```bash
# Install required packages
pip install azure-purview-scanning azure-identity

# Set environment variables
export PURVIEW_ACCOUNT=$PURVIEW_ACCOUNT
export STORAGE_ACCOUNT=$STORAGE_ACCOUNT

# Run the script
python register_data_source.py
```

### Step 3.4: Alternative — Register via Purview Studio Portal

If you prefer the portal approach:

1. Navigate to **https://purview.microsoft.com**
2. Open your Purview account
3. Go to **Data Map → Sources**
4. Click **Register** → Select **Azure Blob Storage**
5. Enter the storage account name and select subscription
6. Click **Register**
7. On the registered source, click **New Scan**
8. Accept defaults and click **Run**

> **✅ Checkpoint 3:** Verify the data source appears in Purview Data Map and a scan is running or completed.

---

## Exercise 4: Review Classifications & Create Custom Classifier (10 min)

### Step 4.1: Query Discovered Classifications

```python
# save as: query_classifications.py
from azure.purview.catalog import PurviewCatalogClient
from azure.identity import DefaultAzureCredential
import os, json

credential = DefaultAzureCredential()
purview_account = os.environ.get("PURVIEW_ACCOUNT", "purview-ai-lab")

catalog_client = PurviewCatalogClient(
    endpoint=f"https://{purview_account}.purview.azure.com",
    credential=credential
)

# Search for all classified assets
search_result = catalog_client.discovery.query(
    search_request={
        "keywords": "*",
        "filter": {
            "and": [
                {"objectType": "Files"},
                {"classification": "MICROSOFT.PERSONAL.US.SOCIAL_SECURITY_NUMBER"}
            ]
        },
        "limit": 25
    }
)

print("=" * 60)
print("DISCOVERED CLASSIFICATIONS")
print("=" * 60)

for asset in search_result.get("value", []):
    print(f"\nAsset: {asset['name']}")
    print(f"  Path: {asset.get('qualifiedName', 'N/A')}")
    classifications = asset.get('classification', [])
    if classifications:
        for c in classifications:
            print(f"  Classification: {c}")
    labels = asset.get('sensitivityLabel', 'Not labeled')
    print(f"  Sensitivity Label: {labels}")

# Get classification statistics
print("\n" + "=" * 60)
print("CLASSIFICATION SUMMARY")
print("=" * 60)

# Query for different classification types
classif_types = [
    "MICROSOFT.PERSONAL.US.SOCIAL_SECURITY_NUMBER",
    "MICROSOFT.PERSONAL.EMAIL",
    "MICROSOFT.PERSONAL.PHONE_NUMBER",
    "MICROSOFT.FINANCIAL.CREDIT_CARD_NUMBER"
]

for ctype in classif_types:
    result = catalog_client.discovery.query(
        search_request={
            "keywords": "*",
            "filter": {"classification": ctype},
            "limit": 1
        }
    )
    count = result.get("@search.count", 0)
    short_name = ctype.split(".")[-1]
    print(f"  {short_name}: {count} asset(s)")
```

### Step 4.2: Create a Custom Classifier

```python
# save as: create_custom_classifier.py
from azure.purview.administration import PurviewAccountClient
from azure.identity import DefaultAzureCredential
import os

credential = DefaultAzureCredential()
purview_account = os.environ.get("PURVIEW_ACCOUNT", "purview-ai-lab")

# Custom classification for AI training dataset naming patterns
custom_classification = {
    "name": "AI_TRAINING_DATASET",
    "description": "Identifies files used as AI model training datasets in Azure AI Foundry",
    "classificationRuleType": "Custom",
    "ruleStatus": "Enabled",
    "classificationRules": [
        {
            "kind": "RegexClassificationRule",
            "name": "ai-training-pattern",
            "description": "Matches training data file naming conventions",
            "pattern": r"(training[_-]data|train[_-]set|model[_-]training)",
            "minimumPercentageMatch": 50,
            "minimumCount": 1
        }
    ]
}

# Custom classification for model metadata
model_metadata_classification = {
    "name": "AI_MODEL_METADATA",
    "description": "Identifies AI model metadata files containing training configuration",
    "classificationRuleType": "Custom",
    "ruleStatus": "Enabled",
    "classificationRules": [
        {
            "kind": "RegexClassificationRule",
            "name": "model-metadata-pattern",
            "description": "Matches model metadata JSON patterns",
            "pattern": r"(model_name|training_date|framework|accuracy|f1_score)",
            "minimumPercentageMatch": 40,
            "minimumCount": 2
        }
    ]
}

print("Custom classifiers defined:")
print(f"  1. {custom_classification['name']}: {custom_classification['description']}")
print(f"  2. {model_metadata_classification['name']}: {model_metadata_classification['description']}")
print("\nApply these via Purview Studio → Data Map → Classification Rules → New")
```

> **✅ Checkpoint 4:** You should see SSN, email, phone, and credit card classifications discovered in your sample data.

---

## Exercise 5: Configure Sensitivity Labels (10 min)

### Step 5.1: Design Label Taxonomy

Create the following sensitivity labels in the Microsoft Purview Compliance Portal:

| Label | Scope | Protection | Auto-Label Triggers |
|-------|-------|-----------|-------------------|
| **Public – AI** | AI Foundry | None | Open datasets, documentation |
| **General – AI** | AI Foundry | Content marking | Internal training data |
| **Confidential – AI Training** | AI Foundry | Encryption + access restriction | PII detected (SSN, email) |
| **Highly Confidential – AI Healthcare** | AI Foundry | Encryption + strict access | PHI detected (diagnosis, medical records) |

### Step 5.2: Create Labels via Compliance Portal

1. Navigate to **https://compliance.microsoft.com**
2. Go to **Information Protection → Labels**
3. Click **+ Create a label**

**For "Confidential – AI Training" label:**

```
Name:        Confidential - AI Training
Description: Contains PII or sensitive data used for AI model training
Scope:       Files & emails, Azure Purview assets
Protection:
  - Encryption: Yes
  - Access: AI Team security group only
  - Permissions: View, Edit (no print, no forward)
Auto-labeling:
  - Sensitive info types:
    * U.S. Social Security Number (SSN) — confidence ≥ 85%
    * Email Address — confidence ≥ 90%
    * Credit Card Number — confidence ≥ 85%
  - Apply label automatically when content matches
```

### Step 5.3: Publish Label Policy

```
Policy Name:  AI-Data-Governance-Policy
Labels:       All four labels created above
Users/Groups: AI Team, Data Engineering Team
Settings:
  - Require justification to remove a label
  - Require users to apply a label to content
  - Default label for documents: General – AI
```

### Step 5.4: Verify Label Application

```python
# save as: verify_labels.py
from azure.purview.catalog import PurviewCatalogClient
from azure.identity import DefaultAzureCredential
import os

credential = DefaultAzureCredential()
purview_account = os.environ.get("PURVIEW_ACCOUNT", "purview-ai-lab")

catalog_client = PurviewCatalogClient(
    endpoint=f"https://{purview_account}.purview.azure.com",
    credential=credential
)

# Search for assets with sensitivity labels
labeled_assets = catalog_client.discovery.query(
    search_request={
        "keywords": "*",
        "filter": {
            "objectType": "Files"
        },
        "limit": 50
    }
)

print("SENSITIVITY LABEL REPORT")
print("=" * 50)

label_counts = {}
for asset in labeled_assets.get("value", []):
    label = asset.get("sensitivityLabel", "Unlabeled")
    label_counts[label] = label_counts.get(label, 0) + 1
    print(f"  {asset['name']}: {label}")

print("\nSUMMARY:")
for label, count in label_counts.items():
    print(f"  {label}: {count} asset(s)")
```

> **✅ Checkpoint 5:** Sensitivity labels should be published and auto-labeling policies active.

---

## Exercise 6: Explore Data Lineage (10 min)

### Step 6.1: Create Lineage for AI Pipeline

```python
# save as: create_lineage.py
from azure.purview.catalog import PurviewCatalogClient
from azure.identity import DefaultAzureCredential
import os

credential = DefaultAzureCredential()
purview_account = os.environ.get("PURVIEW_ACCOUNT", "purview-ai-lab")

catalog_client = PurviewCatalogClient(
    endpoint=f"https://{purview_account}.purview.azure.com",
    credential=credential
)

# Define AI pipeline lineage entity
pipeline_entity = {
    "entity": {
        "typeName": "Process",
        "attributes": {
            "name": "customer-churn-training-pipeline",
            "qualifiedName": "foundry://project/churn/pipeline/v2",
            "description": "End-to-end training pipeline for customer churn prediction model"
        },
        "relationshipAttributes": {
            "inputs": [
                {
                    "typeName": "azure_blob_path",
                    "uniqueAttributes": {
                        "qualifiedName": f"https://{os.environ.get('STORAGE_ACCOUNT')}.blob.core.windows.net/training-data/sample-customers.csv"
                    }
                },
                {
                    "typeName": "azure_blob_path",
                    "uniqueAttributes": {
                        "qualifiedName": f"https://{os.environ.get('STORAGE_ACCOUNT')}.blob.core.windows.net/training-data/sample-transactions.csv"
                    }
                }
            ],
            "outputs": [
                {
                    "typeName": "azure_blob_path",
                    "uniqueAttributes": {
                        "qualifiedName": f"https://{os.environ.get('STORAGE_ACCOUNT')}.blob.core.windows.net/model-artifacts/models/churn/metadata.json"
                    }
                }
            ]
        }
    }
}

# Create lineage
result = catalog_client.entity.create_or_update(body=pipeline_entity)
print(f"Lineage entity created: {result.get('guidAssignments', {})}")

print("\nLineage Summary:")
print("  Inputs:")
print("    → sample-customers.csv (PII: SSN, email, phone)")
print("    → sample-transactions.csv (Financial: credit card)")
print("  Process:")
print("    → customer-churn-training-pipeline v2")
print("  Outputs:")
print("    → churn model metadata v2.1")
```

### Step 6.2: Visualize Lineage in Purview Studio

1. Open **Purview Studio** → **Data Catalog**
2. Search for `customer-churn-training-pipeline`
3. Click the entity → Select **Lineage** tab
4. Explore the visual lineage graph showing:
   - Input data sources with their classifications
   - The training pipeline process
   - Output model artifacts

### Step 6.3: Perform Impact Analysis

```python
# save as: impact_analysis.py
from azure.purview.catalog import PurviewCatalogClient
from azure.identity import DefaultAzureCredential
import os

credential = DefaultAzureCredential()
purview_account = os.environ.get("PURVIEW_ACCOUNT", "purview-ai-lab")

catalog_client = PurviewCatalogClient(
    endpoint=f"https://{purview_account}.purview.azure.com",
    credential=credential
)

# Simulate impact analysis: "What happens if customer data source changes?"
print("IMPACT ANALYSIS: customer-data source change")
print("=" * 50)
print()
print("Affected Data Source: sample-customers.csv")
print("Classifications: SSN, Email, Phone, PHI")
print()
print("Downstream Impact:")
print("  → customer-churn-training-pipeline (DIRECT)")
print("    → churn-model v2.1 (INDIRECT)")
print("      → churn-prediction-endpoint (INDIRECT)")
print()
print("Recommended Actions:")
print("  1. Re-scan source for classification changes")
print("  2. Validate model still meets accuracy thresholds")
print("  3. Update lineage metadata with new data version")
print("  4. Notify model owners of data change")
```

> **✅ Checkpoint 6:** You should see lineage graph connecting your data sources through the pipeline to model artifacts.

---

## Exercise 7: Configure Data Access Policies (5 min)

### Step 7.1: Create Access Policy

```bash
# Ensure Purview can enforce policies on the storage account
PURVIEW_PRINCIPAL_ID=$(az purview account show \
    --account-name $PURVIEW_ACCOUNT \
    --resource-group $RESOURCE_GROUP \
    --query "identity.principalId" -o tsv)

STORAGE_ID=$(az storage account show \
    --name $STORAGE_ACCOUNT \
    --resource-group $RESOURCE_GROUP \
    --query id -o tsv)

# The Storage Blob Data Reader role was already assigned in Exercise 3
# Now configure policy enforcement in Purview Studio:
echo "Configure in Purview Studio → Data Policy → New Policy"
echo ""
echo "Policy Configuration:"
echo "  Name: AI-Training-Data-Read-Policy"
echo "  Data Source: ai-training-storage"
echo "  Effect: Allow"
echo "  Principals: AI-Team security group"
echo "  Actions: Read"
echo "  Scope: training-data container"
```

---

## 🧹 Cleanup

```bash
# Remove all lab resources when done
az group delete \
    --name $RESOURCE_GROUP \
    --yes \
    --no-wait

# Clean up local sample files
rm -f sample-customers.csv sample-transactions.csv model-metadata.json
rm -f register_data_source.py query_classifications.py
rm -f create_custom_classifier.py verify_labels.py
rm -f create_lineage.py impact_analysis.py

echo "Lab resources cleanup initiated"
```

---

## ✅ Lab Completion Checklist

| # | Task | Status |
|---|------|--------|
| 1 | Purview account provisioned and accessible | ☐ |
| 2 | Storage account with sample PII/PHI data created | ☐ |
| 3 | Data source registered in Purview Data Map | ☐ |
| 4 | Classification scan completed with PII/PHI detected | ☐ |
| 5 | Custom AI classifier created | ☐ |
| 6 | Sensitivity labels designed and published | ☐ |
| 7 | Data lineage created for AI pipeline | ☐ |
| 8 | Lineage visualized in Purview Studio | ☐ |
| 9 | Data access policy configured | ☐ |
| 10 | Lab resources cleaned up | ☐ |

---

## 🔗 Additional Resources

- [Microsoft Purview Documentation](https://learn.microsoft.com/en-us/purview/)
- [Purview Python SDK Reference](https://learn.microsoft.com/en-us/python/api/overview/azure/purview)
- [Data Classification Best Practices](https://learn.microsoft.com/en-us/purview/concept-best-practices-classification)
- [Sensitivity Labels Overview](https://learn.microsoft.com/en-us/purview/sensitivity-labels)
- [Data Lineage in Purview](https://learn.microsoft.com/en-us/purview/concept-data-lineage)

---

## ➡️ Next Steps

Continue to **Module 33: Threat Protection** to learn about Azure AI Foundry threat detection, security monitoring, and incident response patterns.

---

*Azure AI Foundry: Zero to Hero · Module 32 Lab · ARC 7: Security, Governance & Compliance · April 2026*
