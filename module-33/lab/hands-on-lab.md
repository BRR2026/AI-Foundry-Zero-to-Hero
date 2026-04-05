# Module 33: Hands-On Lab — Model Governance & Audit Trails

## Azure AI Foundry: Zero to Hero

**Arc 7: Security, Governance & Compliance** | Duration: 60 minutes | Level: Advanced

---

## 🎯 Lab Objectives

In this lab, you will:

1. Register model versions in the Azure AI Foundry Model Registry with governance metadata
2. Configure diagnostic logging and query audit trails with KQL
3. Implement an automated approval gate that enforces risk-tier policies
4. Create and attach model cards to registered models
5. Build a complete governance pipeline from registration to deployment

---

## 📋 Prerequisites

| Requirement | Details |
|-------------|---------|
| **Azure Subscription** | Active subscription with Contributor access |
| **Azure AI Foundry Project** | Existing project with deployed resources |
| **Python** | 3.10 or later |
| **Azure CLI** | 2.60 or later |
| **Packages** | `azure-ai-ml`, `azure-identity`, `azure-monitor-query` |
| **Log Analytics Workspace** | Connected to your AI Foundry project |

---

## 🔧 Lab Setup

### Step 0: Environment Preparation

```bash
# Install required Python packages
pip install azure-ai-ml azure-identity azure-monitor-query

# Login to Azure
az login

# Set your subscription
az account set --subscription "your-subscription-id"

# Set environment variables
export AZURE_SUBSCRIPTION_ID="your-subscription-id"
export AZURE_RESOURCE_GROUP="rg-ai-governance"
export AZURE_AI_PROJECT="foundry-project-prod"
export LOG_ANALYTICS_WORKSPACE_ID="your-law-id"
```

### Verify Connectivity

```python
from azure.ai.ml import MLClient
from azure.identity import DefaultAzureCredential
import os

client = MLClient(
    credential=DefaultAzureCredential(),
    subscription_id=os.environ["AZURE_SUBSCRIPTION_ID"],
    resource_group_name=os.environ["AZURE_RESOURCE_GROUP"],
    workspace_name=os.environ["AZURE_AI_PROJECT"]
)

# Verify connection
workspace = client.workspaces.get(os.environ["AZURE_AI_PROJECT"])
print(f"✅ Connected to: {workspace.name}")
print(f"   Location: {workspace.location}")
print(f"   Resource Group: {os.environ['AZURE_RESOURCE_GROUP']}")
```

---

## Exercise 1: Governance Configuration (10 minutes)

### 1.1 Create a Governance Policy Configuration

Create a file named `governance_config.json` that defines your organization's model governance rules:

```json
{
  "schema_version": "1.0",
  "organization": "Contoso AI",
  "required_tags": [
    "risk_tier",
    "data_classification",
    "owner",
    "approval_status"
  ],
  "risk_tiers": {
    "high": {
      "min_reviewers": 2,
      "require_model_card": true,
      "require_fairness_eval": true,
      "require_security_review": true,
      "max_promotion_time_hours": 72,
      "allowed_approvers": ["MLOps Lead", "Compliance Officer"]
    },
    "medium": {
      "min_reviewers": 1,
      "require_model_card": true,
      "require_fairness_eval": false,
      "require_security_review": false,
      "max_promotion_time_hours": 48,
      "allowed_approvers": ["MLOps Lead"]
    },
    "low": {
      "min_reviewers": 1,
      "require_model_card": false,
      "require_fairness_eval": false,
      "require_security_review": false,
      "max_promotion_time_hours": 24,
      "allowed_approvers": ["MLOps Lead", "Senior Data Scientist"]
    }
  },
  "data_classifications": [
    "public",
    "internal",
    "confidential",
    "highly_confidential"
  ],
  "audit_retention_days": 365,
  "model_card_sections": [
    "model_details",
    "intended_use",
    "performance",
    "limitations",
    "ethical_considerations",
    "governance"
  ]
}
```

### 1.2 Load and Validate the Configuration

```python
import json

def load_governance_config(config_path="governance_config.json"):
    """Load and validate governance configuration."""
    with open(config_path) as f:
        config = json.load(f)

    # Validate required fields
    assert "required_tags" in config, "Missing required_tags"
    assert "risk_tiers" in config, "Missing risk_tiers"
    assert all(
        tier in config["risk_tiers"]
        for tier in ["high", "medium", "low"]
    ), "Must define high, medium, and low risk tiers"

    print(f"✅ Governance config loaded: {config['organization']}")
    print(f"   Required tags: {config['required_tags']}")
    print(f"   Risk tiers: {list(config['risk_tiers'].keys())}")
    return config

governance = load_governance_config()
```

> **✅ Checkpoint:** You should see the governance configuration loaded with all required tags and risk tiers.

---

## Exercise 2: Model Registration with Governance Metadata (10 minutes)

### 2.1 Create a Sample Model Artifact

```python
import os
import json

# Create model artifacts directory
os.makedirs("./lab-models/fraud-detector-v1", exist_ok=True)

# Create a placeholder model file (simulating a real model)
model_info = {
    "model_type": "classification",
    "framework": "xgboost",
    "features": ["transaction_amount", "merchant_category", "time_delta", "location_risk"],
    "target": "is_fraud",
    "training_samples": 500000,
    "training_date": "2026-04-01"
}

with open("./lab-models/fraud-detector-v1/model_info.json", "w") as f:
    json.dump(model_info, f, indent=2)

print("✅ Model artifacts created")
```

### 2.2 Register Model Version 1 with Governance Tags

```python
from azure.ai.ml.entities import Model
from azure.ai.ml.constants import AssetTypes

model_v1 = Model(
    name="lab-fraud-detector",
    version="1",
    path="./lab-models/fraud-detector-v1",
    type=AssetTypes.CUSTOM_MODEL,
    description="Fraud detection model v1 — baseline XGBoost classifier",
    tags={
        "risk_tier": "high",
        "data_classification": "highly_confidential",
        "owner": "fraud-team@contoso.com",
        "approval_status": "pending_review",
        "framework": "xgboost",
        "training_dataset": "transactions-2025-q4",
        "accuracy": "0.921",
        "precision": "0.887",
        "recall": "0.934",
        "has_model_card": "false",
        "fairness_evaluated": "false"
    },
    properties={
        "training_compute": "Standard_DS4_v2",
        "training_duration_hours": "2.5",
        "feature_count": "4",
        "training_samples": "500000"
    }
)

registered_v1 = client.models.create_or_update(model_v1)
print(f"✅ Registered: {registered_v1.name} v{registered_v1.version}")
print(f"   ID: {registered_v1.id}")
```

### 2.3 Register Model Version 2 with Improved Metrics

```python
# Create v2 artifacts
os.makedirs("./lab-models/fraud-detector-v2", exist_ok=True)

model_info_v2 = {**model_info, "features": model_info["features"] + ["velocity_24h", "device_fingerprint"]}
with open("./lab-models/fraud-detector-v2/model_info.json", "w") as f:
    json.dump(model_info_v2, f, indent=2)

model_v2 = Model(
    name="lab-fraud-detector",
    version="2",
    path="./lab-models/fraud-detector-v2",
    type=AssetTypes.CUSTOM_MODEL,
    description="Fraud detection model v2 — additional features and retraining",
    tags={
        "risk_tier": "high",
        "data_classification": "highly_confidential",
        "owner": "fraud-team@contoso.com",
        "approval_status": "pending_review",
        "framework": "xgboost",
        "training_dataset": "transactions-2026-q1",
        "accuracy": "0.952",
        "precision": "0.941",
        "recall": "0.958",
        "has_model_card": "true",
        "fairness_evaluated": "true"
    }
)

registered_v2 = client.models.create_or_update(model_v2)
print(f"✅ Registered: {registered_v2.name} v{registered_v2.version}")
```

### 2.4 Compare Model Versions

```python
def compare_model_versions(client, model_name):
    """Compare governance metadata across model versions."""
    versions = list(client.models.list(name=model_name))

    print(f"\n{'='*70}")
    print(f"Model: {model_name} — Version Comparison")
    print(f"{'='*70}")

    for v in sorted(versions, key=lambda x: int(x.version)):
        print(f"\n  Version {v.version}:")
        print(f"    Description: {v.description}")
        print(f"    Risk Tier: {v.tags.get('risk_tier', 'N/A')}")
        print(f"    Approval: {v.tags.get('approval_status', 'N/A')}")
        print(f"    Accuracy: {v.tags.get('accuracy', 'N/A')}")
        print(f"    Model Card: {v.tags.get('has_model_card', 'N/A')}")
        print(f"    Fairness Eval: {v.tags.get('fairness_evaluated', 'N/A')}")
        print(f"    Created: {v.creation_context.created_at}")
        print(f"    Created By: {v.creation_context.created_by}")

compare_model_versions(client, "lab-fraud-detector")
```

> **✅ Checkpoint:** You should see two model versions registered with different accuracy scores and governance metadata.

---

## Exercise 3: Implementing the Approval Gate (15 minutes)

### 3.1 Build the Governance Validator

```python
from datetime import datetime

class GovernanceValidator:
    """Validates model governance compliance before promotion."""

    def __init__(self, client, governance_config):
        self.client = client
        self.config = governance_config
        self.validation_results = []

    def validate(self, model_name, version):
        """Run all governance checks on a model version."""
        model = self.client.models.get(name=model_name, version=version)
        self.validation_results = []
        passed = True

        # Check 1: Required tags
        for tag in self.config["required_tags"]:
            if tag not in model.tags:
                self._add_result("FAIL", f"Missing required tag: {tag}")
                passed = False
            else:
                self._add_result("PASS", f"Required tag present: {tag}={model.tags[tag]}")

        # Check 2: Valid risk tier
        risk_tier = model.tags.get("risk_tier", "unknown")
        if risk_tier not in self.config["risk_tiers"]:
            self._add_result("FAIL", f"Invalid risk tier: {risk_tier}")
            passed = False
        else:
            self._add_result("PASS", f"Valid risk tier: {risk_tier}")

        # Check 3: Risk-tier-specific requirements
        if risk_tier in self.config["risk_tiers"]:
            rules = self.config["risk_tiers"][risk_tier]

            # Model card check
            if rules.get("require_model_card"):
                if model.tags.get("has_model_card") == "true":
                    self._add_result("PASS", "Model card is present")
                else:
                    self._add_result("FAIL", f"Model card required for {risk_tier} risk tier")
                    passed = False

            # Fairness evaluation check
            if rules.get("require_fairness_eval"):
                if model.tags.get("fairness_evaluated") == "true":
                    self._add_result("PASS", "Fairness evaluation completed")
                else:
                    self._add_result("FAIL", f"Fairness evaluation required for {risk_tier} risk tier")
                    passed = False

        # Check 4: Valid data classification
        data_class = model.tags.get("data_classification", "unknown")
        if data_class not in self.config.get("data_classifications", []):
            self._add_result("WARN", f"Unknown data classification: {data_class}")
        else:
            self._add_result("PASS", f"Valid data classification: {data_class}")

        return passed

    def _add_result(self, status, message):
        self.validation_results.append({
            "status": status,
            "message": message,
            "timestamp": datetime.utcnow().isoformat()
        })

    def print_report(self, model_name, version):
        """Print a formatted validation report."""
        print(f"\n{'='*60}")
        print(f"GOVERNANCE VALIDATION REPORT")
        print(f"Model: {model_name} v{version}")
        print(f"Date: {datetime.utcnow().isoformat()}")
        print(f"{'='*60}")

        for result in self.validation_results:
            icon = {"PASS": "✅", "FAIL": "❌", "WARN": "⚠️"}[result["status"]]
            print(f"  {icon} [{result['status']}] {result['message']}")

        passed = all(r["status"] != "FAIL" for r in self.validation_results)
        print(f"\n{'='*60}")
        print(f"  RESULT: {'✅ APPROVED — all checks passed' if passed else '❌ BLOCKED — governance checks failed'}")
        print(f"{'='*60}")
        return passed
```

### 3.2 Run the Approval Gate Against Both Versions

```python
validator = GovernanceValidator(client, governance)

# Test Version 1 (should FAIL — missing model card and fairness eval)
print("\n" + "🔍 " * 20)
print("Testing Version 1...")
v1_passed = validator.validate("lab-fraud-detector", "1")
validator.print_report("lab-fraud-detector", "1")

# Test Version 2 (should PASS — all governance requirements met)
print("\n" + "🔍 " * 20)
print("Testing Version 2...")
v2_passed = validator.validate("lab-fraud-detector", "2")
validator.print_report("lab-fraud-detector", "2")
```

### 3.3 Promote the Approved Model

```python
def promote_model(client, model_name, version, approver_email):
    """Promote a model version after approval gate passes."""
    model = client.models.get(name=model_name, version=version)

    model.tags["approval_status"] = "approved"
    model.tags["approved_by"] = approver_email
    model.tags["approved_at"] = datetime.utcnow().isoformat()
    model.tags["stage"] = "production_candidate"

    updated = client.models.create_or_update(model)
    print(f"✅ Model promoted: {updated.name} v{updated.version}")
    print(f"   Approved by: {approver_email}")
    print(f"   Stage: {updated.tags['stage']}")
    return updated

# Promote version 2 (which passed all checks)
if v2_passed:
    promoted = promote_model(
        client,
        "lab-fraud-detector",
        "2",
        "mlops-lead@contoso.com"
    )
else:
    print("❌ Cannot promote — governance checks failed")
```

> **✅ Checkpoint:** Version 1 should be blocked (missing model card/fairness eval), and Version 2 should be approved and promoted.

---

## Exercise 4: Creating Model Cards (10 minutes)

### 4.1 Generate a Comprehensive Model Card

```python
def generate_model_card(model_name, version, client, governance_config):
    """Generate a structured model card from model metadata."""
    model = client.models.get(name=model_name, version=version)

    card = {
        "schema_version": "1.0",
        "generated_at": datetime.utcnow().isoformat(),
        "model_details": {
            "name": model.name,
            "version": model.version,
            "description": model.description,
            "type": model.tags.get("framework", "unknown"),
            "owner": model.tags.get("owner", "unknown"),
            "created_at": str(model.creation_context.created_at),
            "created_by": model.creation_context.created_by
        },
        "intended_use": {
            "primary_uses": [
                "Real-time fraud detection for online transactions",
                "Batch scoring of historical transactions for audit"
            ],
            "primary_users": [
                "Fraud operations team",
                "Risk management analysts"
            ],
            "out_of_scope": [
                "Credit scoring or lending decisions",
                "Identity verification",
                "Insurance claim fraud (different domain)"
            ]
        },
        "performance_metrics": {
            "accuracy": float(model.tags.get("accuracy", 0)),
            "precision": float(model.tags.get("precision", 0)),
            "recall": float(model.tags.get("recall", 0)),
            "evaluation_dataset": model.tags.get("training_dataset", "unknown"),
            "disaggregated_performance": {
                "note": "Performance tested across transaction amount ranges and merchant categories",
                "high_value_transactions": {"accuracy": 0.962},
                "low_value_transactions": {"accuracy": 0.941},
                "online_merchants": {"accuracy": 0.955},
                "in_store_merchants": {"accuracy": 0.948}
            }
        },
        "limitations": [
            "Model has not been validated for cryptocurrency transactions",
            "Performance may degrade for transaction amounts exceeding $50,000",
            "Not validated for markets outside North America and Europe",
            "Requires at least 3 prior transactions for accurate velocity features"
        ],
        "ethical_considerations": {
            "fairness_assessment": "Completed — no significant demographic disparities detected",
            "bias_mitigations": [
                "Geographic bias reduction through balanced sampling",
                "Income-level-agnostic feature engineering"
            ],
            "human_oversight": "Required for all transaction blocks exceeding $10,000",
            "potential_harms": [
                "False positives may block legitimate transactions",
                "False negatives may allow fraudulent transactions"
            ]
        },
        "governance": {
            "risk_tier": model.tags.get("risk_tier", "unclassified"),
            "data_classification": model.tags.get("data_classification", "unknown"),
            "approval_status": model.tags.get("approval_status", "unknown"),
            "approved_by": model.tags.get("approved_by", "N/A"),
            "approved_at": model.tags.get("approved_at", "N/A"),
            "regulatory_mappings": [
                "EU AI Act — Article 6 (High-Risk AI Classification)",
                "EU AI Act — Article 13 (Transparency)",
                "PCI DSS — Requirement 6 (Secure Systems)",
                "ISO/IEC 42001 — AI Management System"
            ]
        }
    }
    return card

# Generate the model card
card = generate_model_card("lab-fraud-detector", "2", client, governance)
print(json.dumps(card, indent=2))
```

### 4.2 Save and Attach the Model Card

```python
# Save model card to model artifacts directory
card_path = "./lab-models/fraud-detector-v2/model_card.json"
with open(card_path, "w") as f:
    json.dump(card, f, indent=2)

print(f"✅ Model card saved to: {card_path}")
print(f"   Sections: {list(card.keys())}")
print(f"   Risk tier: {card['governance']['risk_tier']}")
print(f"   Regulatory mappings: {len(card['governance']['regulatory_mappings'])}")
```

> **✅ Checkpoint:** A complete model card should be generated with all required sections including governance metadata and regulatory mappings.

---

## Exercise 5: Querying the Audit Trail (10 minutes)

### 5.1 Configure Diagnostic Settings (if not already configured)

```bash
# Enable diagnostic logging for your AI Foundry project
az monitor diagnostic-settings create \
  --name "lab-governance-audit" \
  --resource "/subscriptions/$AZURE_SUBSCRIPTION_ID/resourceGroups/$AZURE_RESOURCE_GROUP/providers/Microsoft.MachineLearningServices/workspaces/$AZURE_AI_PROJECT" \
  --logs '[
    {"category": "AmlModelsEvent", "enabled": true, "retentionPolicy": {"enabled": true, "days": 365}},
    {"category": "AmlDeploymentEvent", "enabled": true, "retentionPolicy": {"enabled": true, "days": 365}}
  ]' \
  --workspace "/subscriptions/$AZURE_SUBSCRIPTION_ID/resourceGroups/$AZURE_RESOURCE_GROUP/providers/Microsoft.OperationalInsights/workspaces/$LOG_ANALYTICS_WORKSPACE_ID"
```

### 5.2 Query Audit Logs with Python

```python
from azure.monitor.query import LogsQueryClient
from azure.identity import DefaultAzureCredential
from datetime import timedelta

logs_client = LogsQueryClient(DefaultAzureCredential())

# Query model registration events
query = """
AzureActivity
| where ResourceProvider == "Microsoft.MachineLearningServices"
| where OperationNameValue has "models"
| where TimeGenerated > ago(1d)
| project
    TimeGenerated,
    Caller,
    OperationNameValue,
    ActivityStatusValue,
    ResourceGroup
| order by TimeGenerated desc
| take 20
"""

try:
    result = logs_client.query_workspace(
        workspace_id=os.environ["LOG_ANALYTICS_WORKSPACE_ID"],
        query=query,
        timespan=timedelta(days=1)
    )

    print(f"\n{'='*80}")
    print("AUDIT LOG — Model Registry Events (Last 24 Hours)")
    print(f"{'='*80}")

    if result.tables:
        for row in result.tables[0].rows:
            print(f"  {row[0]} | {row[1]} | {row[2]} | {row[3]}")
    else:
        print("  No events found (logs may take 5-15 minutes to appear)")

except Exception as e:
    print(f"⚠️ Log query note: {e}")
    print("   Audit logs may take a few minutes to propagate after model registration.")
```

### 5.3 Create a Local Audit Record

```python
# Create a local audit log for immediate verification
audit_log = []

def log_governance_event(action, model_name, version, user, details=""):
    """Record a governance event to the local audit log."""
    event = {
        "timestamp": datetime.utcnow().isoformat(),
        "action": action,
        "model": f"{model_name} v{version}",
        "user": user,
        "details": details
    }
    audit_log.append(event)
    return event

# Log our lab activities
log_governance_event("MODEL_REGISTERED", "lab-fraud-detector", "1", "lab-user@contoso.com", "Baseline model")
log_governance_event("MODEL_REGISTERED", "lab-fraud-detector", "2", "lab-user@contoso.com", "Improved model")
log_governance_event("GOVERNANCE_CHECK", "lab-fraud-detector", "1", "system", "FAILED — missing model card")
log_governance_event("GOVERNANCE_CHECK", "lab-fraud-detector", "2", "system", "PASSED — all checks")
log_governance_event("MODEL_APPROVED", "lab-fraud-detector", "2", "mlops-lead@contoso.com", "Promoted to production candidate")

# Print the audit trail
print(f"\n{'='*80}")
print("LOCAL GOVERNANCE AUDIT TRAIL")
print(f"{'='*80}")
for event in audit_log:
    print(f"  [{event['timestamp']}] {event['action']}")
    print(f"    Model: {event['model']} | User: {event['user']}")
    print(f"    Details: {event['details']}")
    print()

# Save audit trail
with open("governance_audit_trail.json", "w") as f:
    json.dump(audit_log, f, indent=2)

print(f"✅ Audit trail saved: {len(audit_log)} events")
```

> **✅ Checkpoint:** You should see a complete audit trail showing model registrations, governance checks (pass/fail), and approval events.

---

## Exercise 6: Challenge Extension (5 minutes, optional)

### 6.1 Build a Governance Dashboard Report

```python
def governance_dashboard(client, model_names, governance_config):
    """Generate a governance compliance dashboard."""
    print(f"\n{'='*70}")
    print("📊 MODEL GOVERNANCE COMPLIANCE DASHBOARD")
    print(f"{'='*70}")

    for name in model_names:
        versions = list(client.models.list(name=name))
        print(f"\n  📦 {name} ({len(versions)} versions)")

        for v in sorted(versions, key=lambda x: int(x.version)):
            risk = v.tags.get("risk_tier", "?")
            approval = v.tags.get("approval_status", "?")
            card = v.tags.get("has_model_card", "?")
            fairness = v.tags.get("fairness_evaluated", "?")

            risk_icon = {"high": "🔴", "medium": "🟡", "low": "🟢"}.get(risk, "⚪")
            approval_icon = {"approved": "✅", "pending_review": "⏳", "rejected": "❌"}.get(approval, "❓")

            print(f"    v{v.version}: {risk_icon} {risk} | {approval_icon} {approval} | Card: {card} | Fairness: {fairness}")

    print(f"\n{'='*70}")

governance_dashboard(client, ["lab-fraud-detector"], governance)
```

---

## 🧹 Lab Cleanup

```python
# Archive lab models (optional — keeps them but marks as deprecated)
try:
    client.models.archive(name="lab-fraud-detector", version="1")
    client.models.archive(name="lab-fraud-detector", version="2")
    print("✅ Lab models archived")
except Exception as e:
    print(f"Cleanup note: {e}")

# Clean up local files
import shutil
if os.path.exists("./lab-models"):
    shutil.rmtree("./lab-models")
if os.path.exists("governance_config.json"):
    os.remove("governance_config.json")
if os.path.exists("governance_audit_trail.json"):
    os.remove("governance_audit_trail.json")

print("✅ Local lab files cleaned up")
```

---

## ✅ Lab Completion Checklist

| # | Task | Status |
|---|------|--------|
| 1 | Created governance configuration with risk-tier policies | ☐ |
| 2 | Registered two model versions with governance tags | ☐ |
| 3 | Approval gate correctly blocked non-compliant model (v1) | ☐ |
| 4 | Approval gate correctly approved compliant model (v2) | ☐ |
| 5 | Generated comprehensive model card with all sections | ☐ |
| 6 | Queried audit trail (Azure Monitor or local log) | ☐ |
| 7 | Promoted approved model with approval metadata | ☐ |

---

## 🔗 Additional Resources

- [Azure AI Foundry Model Registry Documentation](https://learn.microsoft.com/azure/ai-foundry/model-catalog)
- [Azure Monitor Diagnostic Settings](https://learn.microsoft.com/azure/azure-monitor/essentials/diagnostic-settings)
- [KQL Query Language Reference](https://learn.microsoft.com/azure/data-explorer/kusto/query/)
- [Responsible AI Practices](https://learn.microsoft.com/azure/ai-foundry/responsible-use-of-ai-overview)

---

*Module 33 Lab — Azure AI Foundry: Zero to Hero — Arc 7: Security, Governance & Compliance*
*© 2026 Azure AI Foundry Training*
