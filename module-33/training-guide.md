# Module 33: Model Governance & Audit Trails

## Azure AI Foundry: Zero to Hero — Training Guide

**Arc 7: Security, Governance & Compliance** | Module 33 of 45 | April 2026

---

## 📋 Module Overview

This module covers the essential practices for establishing enterprise-grade model governance in Azure AI Foundry. Learners will implement model versioning, configure audit logging, build approval workflows, and create model documentation that satisfies regulatory and compliance requirements.

| Detail              | Value                                               |
|---------------------|-----------------------------------------------------|
| **Duration**        | 90 minutes                                          |
| **Level**           | Advanced                                            |
| **Prerequisites**   | Module 31 (RBAC), Module 32 (Network Security)      |
| **Platform**        | Azure AI Foundry                                    |
| **Arc**             | 7 — Security, Governance & Compliance               |
| **Date**            | April 2026                                          |

---

## 🎯 Learning Objectives

By the end of this module, learners will be able to:

1. **Register and version models** in the Azure AI Foundry Model Registry with governance metadata
2. **Configure and query audit logs** for model deployments and access using Azure Monitor
3. **Design approval workflows** with risk-tier-based gates using GitHub Actions or Azure DevOps
4. **Create model cards** that meet regulatory requirements (EU AI Act, NIST AI RMF, ISO 42001)
5. **Implement automated governance checks** that enforce policy compliance before model promotion

---

## 📚 Module Outline

### Section 1: Why Model Governance Matters (15 min)

**Instructor Notes:**
- Begin with a real-world scenario: a financial services company deploys an unvalidated model to production, causing biased lending decisions — illustrate the risk of no governance
- Discuss the four pillars: Traceability, Compliance, Reproducibility, Accountability
- Overview the regulatory landscape driving governance requirements

**Key Topics:**
- The governance challenge in enterprise AI
- Regulatory requirements: EU AI Act, NIST AI RMF, ISO/IEC 42001, SOC 2
- How Azure AI Foundry addresses governance natively
- Cost of governance failures vs. investment in governance infrastructure

**Discussion Prompt:**
> "Think about your organization's current model deployment process. How many steps involve manual review? What happens if a model is deployed without documentation?"

---

### Section 2: Model Versioning & Registry (20 min)

**Instructor Notes:**
- Demo the Model Registry in the Azure AI Foundry portal
- Show how immutable versions work — once registered, artifacts cannot be modified
- Demonstrate tagging strategies for lifecycle stage tracking
- Walk through the SDK code for registering models with governance metadata

**Key Topics:**
- Azure AI Foundry Model Registry architecture
- Immutable versioning and artifact integrity
- Semantic tagging strategies (stage, risk tier, data classification)
- Lineage tracking: training data → model → deployment
- Cross-project model sharing with governance policies
- Archiving deprecated model versions

**Demo Sequence:**
1. Navigate to the Model Registry in the Foundry portal
2. Register a new model version using the SDK with governance tags
3. Show the version history and metadata comparison
4. Demonstrate how tags enable lifecycle stage transitions
5. Archive an older version and show it's no longer eligible for deployment

**Code Walkthrough:**
- `MLClient` initialization with `DefaultAzureCredential`
- `Model` entity creation with tags and properties
- `client.models.create_or_update()` for registration
- `client.models.list()` for version enumeration
- `client.models.archive()` for deprecation

---

### Section 3: Audit Logs for Deployments & Access (20 min)

**Instructor Notes:**
- Explain the difference between Azure Activity Log, Resource Logs, and Application-level logs
- Show how to enable Diagnostic Settings for the AI Foundry project
- Walk through KQL queries for common governance scenarios
- Emphasize log retention requirements for compliance

**Key Topics:**
- Azure Monitor integration with AI Foundry
- Event categories: Model Registry, Deployments, Access Control, Inference, Data Access
- Diagnostic Settings configuration for comprehensive logging
- KQL queries for governance reporting
- Log retention policies and archival strategies
- Alert rules for unauthorized or anomalous activities

**Demo Sequence:**
1. Open Azure Monitor → Activity Log filtered by AI Foundry resource
2. Show Diagnostic Settings configuration in the portal
3. Execute KQL queries in Log Analytics to find model deployment events
4. Create a simple alert rule for model deletion events
5. Show how to export audit data for compliance reporting

**KQL Examples:**
```kql
// All model lifecycle events in the last 30 days
AzureActivity
| where ResourceProvider == "Microsoft.MachineLearningServices"
| where OperationNameValue has_any ("models/write", "models/delete", "deployments/write")
| where TimeGenerated > ago(30d)
| project TimeGenerated, Caller, OperationNameValue, ActivityStatusValue
| order by TimeGenerated desc

// Track who deployed models to production endpoints
AzureActivity
| where OperationNameValue has "deployments/write"
| where ActivityStatusValue == "Success"
| project TimeGenerated, Caller, ResourceGroup, Properties_d
| order by TimeGenerated desc
```

---

### Section 4: Change Management & Approval Workflows (20 min)

**Instructor Notes:**
- Stress that governance without enforcement is just documentation
- Show the full approval workflow architecture diagram
- Walk through the GitHub Actions workflow file line by line
- Discuss risk-tier-based approval policies

**Key Topics:**
- Approval gate architecture: registration → validation → approval → deployment
- GitHub Environments with required reviewers
- Azure DevOps Environments and approval gates (alternative)
- Risk-tier-based policies (high/medium/low)
- Automated governance validation scripts
- Separation of duties: data scientists vs. MLOps vs. compliance

**Demo Sequence:**
1. Show the GitHub Actions workflow YAML for model promotion
2. Walk through the governance validation script
3. Demonstrate triggering the workflow and the approval gate UI
4. Show how different risk tiers require different approval levels
5. Demonstrate the audit trail created by the workflow

**Best Practices:**
- Use separate environments for each risk tier
- Require multiple reviewers for high-risk models
- Include compliance officers in the approval chain for regulated industries
- Automate as many validation checks as possible before human review
- Store governance configurations in version-controlled config files

---

### Section 5: Model Documentation & Model Cards (15 min)

**Instructor Notes:**
- Explain the origin of model cards (Google Research, 2018)
- Map model card sections to regulatory requirements
- Show how model cards are stored alongside model artifacts in the registry
- Emphasize that model cards are living documents that should be updated

**Key Topics:**
- Model card structure: intended use, performance, limitations, ethics
- Regulatory mapping: EU AI Act Article 13 (Transparency)
- Creating structured model cards in JSON format
- Attaching model cards to registered models
- Automated model card generation from evaluation results
- Model card review as part of the approval workflow

**Demo Sequence:**
1. Walk through a complete model card example
2. Generate a model card using the Python script
3. Register a model with the card included in artifacts
4. Show how model card content can be queried and reported

---

### Mini Hack: Implement Model Versioning with Approval Gates (30 min)

**Setup Requirements:**
- Azure subscription with AI Foundry project
- Python 3.10+ with `azure-ai-ml`, `azure-identity`, `azure-monitor-query`
- GitHub repository with Actions enabled (for approval workflow)
- Log Analytics workspace connected to the AI Foundry project

**Exercise Steps:**

| Step | Task | Duration |
|------|------|----------|
| 1 | Create governance configuration file | 5 min |
| 2 | Register a model with governance metadata | 5 min |
| 3 | Implement the approval gate validation script | 10 min |
| 4 | Query the audit trail from Azure Monitor | 5 min |
| 5 | Challenge: extend with notifications | 5 min |

**Success Criteria:**
- ✅ Model is registered with all required governance tags
- ✅ Approval gate correctly validates governance metadata
- ✅ High-risk models are blocked without explicit approval
- ✅ Audit trail shows all model lifecycle events
- ✅ Model card is attached and queryable

---

## 🎥 Featured Video

**Building Trustworthy AI: Ensuring Responsible AI With Content Safety**
- **Source:** Microsoft Azure
- **Video ID:** bkssatMl0NE
- **URL:** https://www.youtube.com/watch?v=bkssatMl0NE
- **Relevance:** Covers responsible AI practices and content safety frameworks that underpin model governance decisions

---

## 📝 Assessment

Learners should complete the Module 33 assessment quiz (`quiz/assessment.md`) covering:
- Model Registry concepts and versioning strategies
- Audit log configuration and KQL queries
- Approval workflow design and risk-tier policies
- Model card structure and regulatory requirements
- Governance metadata best practices

**Passing Score:** 80% (8/10 correct)

---

## 🔗 Resources

### Microsoft Documentation
- [Azure AI Foundry Model Registry](https://learn.microsoft.com/azure/ai-foundry/model-catalog)
- [Azure Monitor Activity Log](https://learn.microsoft.com/azure/azure-monitor/essentials/activity-log)
- [Azure Monitor Diagnostic Settings](https://learn.microsoft.com/azure/azure-monitor/essentials/diagnostic-settings)
- [Responsible AI Overview](https://learn.microsoft.com/azure/ai-foundry/responsible-use-of-ai-overview)

### Regulatory & Standards
- [EU AI Act Official Text](https://eur-lex.europa.eu/eli/reg/2024/1689)
- [NIST AI Risk Management Framework](https://www.nist.gov/artificial-intelligence/ai-risk-management-framework)
- [ISO/IEC 42001:2023](https://www.iso.org/standard/81230.html)

### Related Modules
- **Module 31:** Identity & RBAC in Azure AI Foundry
- **Module 32:** Network Security & Private Endpoints
- **Module 34:** Data Privacy & Protection *(Next)*

---

## 🗂️ Module Files

| File | Description |
|------|-------------|
| `blog.html` | Main content article (MS Dev Blog style) |
| `training-guide.md` | This instructor training guide |
| `slides/presentation.html` | Slide deck (12-15 slides, dark theme) |
| `lab/hands-on-lab.md` | Step-by-step hands-on lab |
| `quiz/assessment.md` | 10-question multiple choice assessment |

---

*Module 33 — Azure AI Foundry: Zero to Hero — Arc 7: Security, Governance & Compliance*
*© 2026 Azure AI Foundry Training*
