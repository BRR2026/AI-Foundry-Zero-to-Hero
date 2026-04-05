# Module 39 — Hands-On Lab
## AI Foundry in CSP & Managed Services
### ARC 8: Advanced Patterns & Enterprise | April 2026

---

## 🎯 Lab Overview

| Field | Details |
|---|---|
| **Duration** | 120 minutes |
| **Level** | Advanced |
| **Goal** | Design and partially implement a multi-tenant AI Foundry architecture for a CSP scenario |
| **Prerequisites** | Azure subscription with Contributor access, Azure CLI installed, basic familiarity with Bicep |

> **⚠️ Important:** This lab focuses on **Azure AI Foundry** — the unified platform for building and managing AI agents and applications. Do not confuse with Azure Machine Learning.

---

## 📋 Lab Exercises

### Exercise 1: CSP Subscription & Resource Provisioning (25 min)

#### Objective
Simulate CSP customer onboarding by creating an AI Foundry hub and project in a target subscription.

#### Steps

**Step 1.1 — Set up resource group for a simulated customer**

```bash
# Define variables
CUSTOMER_NAME="contoso"
LOCATION="eastus2"
RG_NAME="rg-${CUSTOMER_NAME}-ai"
CSP_TIER="gold"

# Create resource group with CSP management tags
az group create \
  --name $RG_NAME \
  --location $LOCATION \
  --tags \
    "csp:customer=$CUSTOMER_NAME" \
    "csp:tier=$CSP_TIER" \
    "csp:managed=true" \
    "csp:onboarding-date=$(date +%Y-%m-%d)"
```

**Step 1.2 — Create AI Services account**

```bash
# Create the AI Services account (backing resource for AI Foundry)
az cognitiveservices account create \
  --name "ais-${CUSTOMER_NAME}" \
  --resource-group $RG_NAME \
  --kind "AIServices" \
  --sku "S0" \
  --location $LOCATION \
  --custom-domain "ais-${CUSTOMER_NAME}" \
  --tags \
    "csp:customer=$CUSTOMER_NAME" \
    "csp:tier=$CSP_TIER" \
    "csp:managed=true"
```

**Step 1.3 — Create AI Foundry Hub**

```bash
# Create AI Foundry Hub
az ai foundry hub create \
  --name "hub-${CUSTOMER_NAME}" \
  --resource-group $RG_NAME \
  --location $LOCATION \
  --display-name "${CUSTOMER_NAME^} AI Hub"

# Verify hub creation
az ai foundry hub show \
  --name "hub-${CUSTOMER_NAME}" \
  --resource-group $RG_NAME \
  --output table
```

**Step 1.4 — Create AI Foundry Project**

```bash
# Create project within the hub
az ai foundry project create \
  --name "proj-${CUSTOMER_NAME}-cs-bot" \
  --hub-name "hub-${CUSTOMER_NAME}" \
  --resource-group $RG_NAME \
  --display-name "${CUSTOMER_NAME^} Customer Service Agent"

# List projects in the hub
az ai foundry project list \
  --hub-name "hub-${CUSTOMER_NAME}" \
  --resource-group $RG_NAME \
  --output table
```

**Step 1.5 — Apply CSP tags to all resources in the group**

```bash
# Verify all resources have required CSP tags
az resource list \
  --resource-group $RG_NAME \
  --query "[].{Name:name, Type:type, Tags:tags}" \
  --output table
```

#### ✅ Checkpoint
- [ ] Resource group created with CSP tags
- [ ] AI Services account provisioned
- [ ] AI Foundry Hub created and verified
- [ ] Project created within the hub
- [ ] All resources tagged with `csp:customer`, `csp:tier`, `csp:managed`

---

### Exercise 2: Azure Lighthouse Delegation (20 min)

#### Objective
Create an Azure Lighthouse delegation template that grants your CSP operations team appropriate access to customer AI Foundry resources.

#### Steps

**Step 2.1 — Create the Lighthouse delegation template**

Create a file named `lighthouse-delegation.json`:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-08-01/managementGroupDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "mspOfferName": {
      "type": "string",
      "defaultValue": "CSP AI Foundry Managed Services"
    },
    "mspTenantId": {
      "type": "string",
      "metadata": {
        "description": "The CSP partner's tenant ID"
      }
    },
    "aiOpsGroupId": {
      "type": "string",
      "metadata": {
        "description": "Object ID of the AI Operations security group"
      }
    },
    "monitoringGroupId": {
      "type": "string",
      "metadata": {
        "description": "Object ID of the Monitoring security group"
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.ManagedServices/registrationDefinitions",
      "apiVersion": "2022-10-01",
      "name": "[guid(parameters('mspOfferName'))]",
      "properties": {
        "registrationDefinitionName": "[parameters('mspOfferName')]",
        "description": "Delegated access for CSP partner to manage AI Foundry resources",
        "managedByTenantId": "[parameters('mspTenantId')]",
        "authorizations": [
          {
            "principalId": "[parameters('aiOpsGroupId')]",
            "roleDefinitionId": "b24988ac-6180-42a0-ab88-20f7382dd24c",
            "principalIdDisplayName": "CSP AI Operations Team"
          },
          {
            "principalId": "[parameters('aiOpsGroupId')]",
            "roleDefinitionId": "25fbc0a9-bd7c-42a3-aa1a-3b75d497ee68",
            "principalIdDisplayName": "CSP AI Operations Team - Cognitive Services"
          },
          {
            "principalId": "[parameters('monitoringGroupId')]",
            "roleDefinitionId": "acdd72a7-3385-48ef-bd42-f606fba81ae7",
            "principalIdDisplayName": "CSP Monitoring Team"
          },
          {
            "principalId": "[parameters('monitoringGroupId')]",
            "roleDefinitionId": "749f88d5-cbae-40b8-bcfc-e573ddc772fa",
            "principalIdDisplayName": "CSP Monitoring Team - Monitoring Contributor"
          }
        ]
      }
    }
  ]
}
```

**Step 2.2 — Validate the template**

```bash
# Validate the Lighthouse template
az deployment sub validate \
  --location $LOCATION \
  --template-file lighthouse-delegation.json \
  --parameters \
    mspTenantId="<your-csp-tenant-id>" \
    aiOpsGroupId="<your-ai-ops-group-id>" \
    monitoringGroupId="<your-monitoring-group-id>"
```

**Step 2.3 — Review delegation scope**

```bash
# List existing delegations (if any)
az managedservices definition list --output table

# List active assignments
az managedservices assignment list --output table
```

#### ✅ Checkpoint
- [ ] Lighthouse delegation template created with appropriate roles
- [ ] Template validation passes
- [ ] Understand the difference between Contributor, Cognitive Services Contributor, Reader, and Monitoring Contributor roles

---

### Exercise 3: Governance Policies (20 min)

#### Objective
Create and assign Azure Policy definitions that enforce CSP governance on AI Foundry resources.

#### Steps

**Step 3.1 — Create policy: Require CSP tags**

Create a file named `policy-require-csp-tags.json`:

```json
{
  "mode": "Indexed",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.MachineLearningServices/workspaces"
        },
        {
          "anyOf": [
            { "field": "tags['csp:customer']", "exists": false },
            { "field": "tags['csp:tier']", "exists": false },
            { "field": "tags['csp:managed']", "exists": false }
          ]
        }
      ]
    },
    "then": {
      "effect": "deny"
    }
  }
}
```

**Step 3.2 — Create policy: Deny public network access**

Create a file named `policy-deny-public-access.json`:

```json
{
  "mode": "Indexed",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.MachineLearningServices/workspaces"
        },
        {
          "field": "Microsoft.MachineLearningServices/workspaces/publicNetworkAccess",
          "notEquals": "Disabled"
        }
      ]
    },
    "then": {
      "effect": "deny"
    }
  }
}
```

**Step 3.3 — Create and assign the policy definitions**

```bash
# Create the tag enforcement policy
az policy definition create \
  --name "csp-require-ai-foundry-tags" \
  --display-name "Require CSP tags on AI Foundry resources" \
  --description "Denies creation of AI Foundry workspaces without required CSP management tags" \
  --rules policy-require-csp-tags.json \
  --mode Indexed

# Create the network access policy
az policy definition create \
  --name "csp-deny-public-ai-foundry" \
  --display-name "Deny public network access on AI Foundry" \
  --description "Ensures all AI Foundry hubs and projects have public network access disabled" \
  --rules policy-deny-public-access.json \
  --mode Indexed

# Assign to the customer resource group
az policy assignment create \
  --name "csp-ai-foundry-tags-${CUSTOMER_NAME}" \
  --display-name "CSP Tags - ${CUSTOMER_NAME}" \
  --policy "csp-require-ai-foundry-tags" \
  --scope "/subscriptions/$(az account show --query id -o tsv)/resourceGroups/${RG_NAME}"
```

**Step 3.4 — Verify policy compliance**

```bash
# Check compliance state (may take a few minutes to evaluate)
az policy state summarize \
  --resource-group $RG_NAME \
  --output table
```

#### ✅ Checkpoint
- [ ] Tag enforcement policy created and assigned
- [ ] Public network access deny policy created
- [ ] Policies scoped to customer resource group
- [ ] Compliance evaluation triggered

---

### Exercise 4: SLA Monitoring Dashboard (20 min)

#### Objective
Create KQL queries for monitoring AI Foundry SLA metrics across managed customer environments.

#### Steps

**Step 4.1 — Availability monitoring query**

```kql
// SLA Availability — Hourly calculation
let sla_window = 30d;
AzureDiagnostics
| where TimeGenerated > ago(sla_window)
| where ResourceProvider == "MICROSOFT.MACHINELEARNINGSERVICES"
| where OperationName contains "inference"
| summarize
    TotalRequests   = count(),
    SuccessCount    = countif(ResultType == "Success"),
    FailedCount     = countif(ResultType != "Success"),
    AvgLatencyMs    = avg(DurationMs),
    P99LatencyMs    = percentile(DurationMs, 99)
    by Resource, bin(TimeGenerated, 1h)
| extend AvailabilityPct = round(toreal(SuccessCount) / TotalRequests * 100, 3)
| extend SLABreach = iff(AvailabilityPct < 99.9, "⚠️ BREACH", "✅ OK")
| order by TimeGenerated desc
```

**Step 4.2 — Token consumption by customer**

```kql
// Token consumption per customer per day
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.MACHINELEARNINGSERVICES"
| where Category == "RequestResponse"
| extend CustomerTag = tostring(tags_s["csp:customer"])
| extend TierTag = tostring(tags_s["csp:tier"])
| extend PromptTokens = toint(properties_s["promptTokens"])
| extend CompletionTokens = toint(properties_s["completionTokens"])
| summarize
    TotalPromptTokens = sum(PromptTokens),
    TotalCompletionTokens = sum(CompletionTokens),
    TotalRequests = count(),
    AvgLatencyMs = avg(DurationMs)
    by CustomerTag, TierTag, bin(TimeGenerated, 1d)
| extend EstimatedCost = (TotalPromptTokens * 0.000005) + (TotalCompletionTokens * 0.000015)
| order by CustomerTag, TimeGenerated desc
```

**Step 4.3 — Cross-tenant resource health**

```kql
// Resource health across all managed AI Foundry resources
resources
| where type == "microsoft.machinelearningservices/workspaces"
| where tags["csp:managed"] == "true"
| project
    CustomerName = tags["csp:customer"],
    Tier = tags["csp:tier"],
    HubName = name,
    ResourceGroup = resourceGroup,
    Location = location,
    State = properties.provisioningState,
    PublicAccess = properties.publicNetworkAccess
| order by CustomerName
```

**Step 4.4 — SLA breach alerting rule**

```bash
# Create an alert rule for SLA breaches
az monitor scheduled-query create \
  --name "csp-sla-breach-alert" \
  --resource-group $RG_NAME \
  --scopes "/subscriptions/$(az account show --query id -o tsv)/resourceGroups/${RG_NAME}" \
  --condition "count > 0" \
  --condition-query "AzureDiagnostics | where ResourceProvider == 'MICROSOFT.MACHINELEARNINGSERVICES' | where ResultType != 'Success' | summarize FailureCount = count() by bin(TimeGenerated, 5m) | where FailureCount > 10" \
  --description "Alert when AI Foundry error rate exceeds SLA threshold" \
  --severity 1 \
  --evaluation-frequency "5m" \
  --window-size "15m"
```

#### ✅ Checkpoint
- [ ] Availability monitoring query tested
- [ ] Token consumption query tested
- [ ] Resource health query tested
- [ ] SLA breach alert rule created

---

### Exercise 5: Cost Allocation & Chargeback (15 min)

#### Objective
Implement tag-based cost allocation for multi-tenant AI Foundry resources.

#### Steps

**Step 5.1 — Verify tag consistency across resources**

```bash
# List all resources with CSP tags
az resource list \
  --resource-group $RG_NAME \
  --query "[].{Name:name, Type:type, Customer:tags.\"csp:customer\", Tier:tags.\"csp:tier\"}" \
  --output table
```

**Step 5.2 — Query cost data by customer tag**

```bash
# Export cost data filtered by CSP customer tag
az costmanagement query \
  --type ActualCost \
  --scope "/subscriptions/$(az account show --query id -o tsv)" \
  --timeframe MonthToDate \
  --dataset-filter "{\"tags\":{\"name\":\"csp:customer\",\"operator\":\"In\",\"values\":[\"$CUSTOMER_NAME\"]}}" \
  --dataset-grouping name="ResourceType" type="Dimension" \
  --output table
```

**Step 5.3 — Create a cost allocation report script**

Create a file named `cost-report.py`:

```python
"""CSP Customer Cost Allocation Report Generator"""
from azure.identity import DefaultAzureCredential
from azure.mgmt.costmanagement import CostManagementClient
import json

credential = DefaultAzureCredential()
client = CostManagementClient(credential)

# Define scope (subscription level)
scope = "/subscriptions/<your-subscription-id>"

# Query costs grouped by csp:customer tag
query = {
    "type": "ActualCost",
    "timeframe": "MonthToDate",
    "dataset": {
        "granularity": "Daily",
        "grouping": [
            {"type": "TagKey", "name": "csp:customer"},
            {"type": "Dimension", "name": "ServiceName"}
        ],
        "filter": {
            "tags": {
                "name": "csp:managed",
                "operator": "In",
                "values": ["true"]
            }
        }
    }
}

result = client.query.usage(scope=scope, parameters=query)

# Process and display results
print(f"{'Customer':<20} {'Service':<35} {'Cost':>12}")
print("-" * 70)
for row in result.rows:
    cost, customer, service, currency = row[0], row[1], row[2], row[3]
    if cost > 0:
        print(f"{customer:<20} {service:<35} {currency} {cost:>8.2f}")
```

#### ✅ Checkpoint
- [ ] All resources have consistent CSP tags
- [ ] Cost query by customer tag works
- [ ] Cost report script created

---

### Exercise 6: Mini Hack — Multi-Tenant Architecture Design (20 min)

#### Objective
Design a complete multi-tenant architecture for the CSP scenario described below.

#### Scenario

You are a CSP partner. Your customer portfolio:

| Customer | Industry | Requirements |
|----------|----------|-------------|
| **MedCare Health** | Healthcare | HIPAA, dedicated capacity, PHI isolation |
| **RetailCo** (5 stores) | Retail | AI chatbots, cost-optimized, standard SLA |
| **FinanceFirst** | Finance | SOC 2, dedicated PTU, named support engineer |

#### Tasks

1. **Draw an architecture diagram** showing:
   - Subscription/tenant structure per customer
   - AI Foundry Hub and Project placement
   - Network isolation boundaries
   - Monitoring and management plane

2. **Create an RBAC matrix:**

| Role | MedCare Scope | RetailCo Scope | FinanceFirst Scope |
|------|--------------|----------------|-------------------|
| CSP AI Ops | ? | ? | ? |
| CSP Monitoring | ? | ? | ? |
| Customer Admin | ? | ? | ? |
| Customer User | ? | ? | ? |

3. **Assign SLA tiers:**

| Customer | Tier | Justification |
|----------|------|--------------|
| MedCare | ? | ? |
| RetailCo | ? | ? |
| FinanceFirst | ? | ? |

4. **Define the billing model** for each customer segment.

#### Deliverable
A markdown document or whiteboard sketch containing all four elements above.

#### ✅ Checkpoint
- [ ] Architecture diagram complete
- [ ] RBAC matrix filled in
- [ ] SLA tiers assigned with justifications
- [ ] Billing model defined

---

## 🧹 Cleanup

```bash
# Remove lab resources
az group delete --name "rg-${CUSTOMER_NAME}-ai" --yes --no-wait

# Remove policy definitions
az policy definition delete --name "csp-require-ai-foundry-tags"
az policy definition delete --name "csp-deny-public-ai-foundry"

echo "Lab cleanup initiated."
```

---

## 📝 Lab Summary

In this lab you:

1. **Provisioned** AI Foundry resources with CSP management tags
2. **Created** an Azure Lighthouse delegation template for cross-tenant access
3. **Implemented** Azure Policy definitions for governance enforcement
4. **Built** SLA monitoring queries using KQL
5. **Configured** cost allocation using tag-based filtering
6. **Designed** a multi-tenant architecture for a CSP scenario

### Key Learnings

- AI Foundry resources are standard Azure resources manageable through CSP workflows
- Azure Lighthouse provides superior cross-tenant management compared to AOBO
- Tags are the foundation of cost allocation and governance in multi-tenant environments
- KQL queries enable SLA monitoring across managed customer subscriptions
- Architecture decisions (isolated vs. shared) depend on compliance, cost, and capacity needs

---

*Module 39 of 45 · Azure AI Foundry: Zero to Hero · ARC 8: Advanced Patterns & Enterprise*
