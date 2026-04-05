# Module 32: Data Governance & Purview Integration

## Azure AI Foundry: Zero to Hero Training Series

> **ARC 7 · SECURITY, GOVERNANCE & COMPLIANCE** | Module 32 of 45 | April 2026

---

## 🎯 Module Overview

This module covers Microsoft Purview integration with Azure AI Foundry for comprehensive data governance. You will learn to classify sensitive data, track lineage across AI pipelines, apply sensitivity labels, and achieve regulatory compliance with GDPR, HIPAA, and SOC 2.

### Learning Objectives

By the end of this module, participants will be able to:

1. **Explain** why data governance is critical for AI workloads in Azure AI Foundry
2. **Configure** Microsoft Purview to discover and catalog AI data sources
3. **Implement** automated data classification using built-in and custom classifiers
4. **Apply** sensitivity labels to protect AI training data and model artifacts
5. **Track** end-to-end data lineage across AI Foundry pipelines
6. **Design** compliance strategies for GDPR, HIPAA, and SOC 2 requirements
7. **Create** data access policies that enforce least-privilege access to AI data

### Prerequisites

- Completion of Modules 1–31 (especially Module 31: Network Security)
- Azure subscription with Contributor access
- Familiarity with Azure AI Foundry projects and data stores
- Basic understanding of data privacy regulations

### Duration

| Component          | Time       |
|--------------------|------------|
| Lecture / Reading  | 45 minutes |
| Hands-On Lab       | 60 minutes |
| Mini Hack          | 45 minutes |
| Quiz / Assessment  | 15 minutes |
| **Total**          | **~2.75 hours** |

---

## 📚 Module Content

### Section 1: Why AI Data Governance Matters

**Key Concepts:**

- AI systems consume vast volumes of data from diverse sources
- Data governance for AI extends beyond traditional data management
- Five pillars: Discovery, Classification, Lineage, Access Control, Compliance
- Azure AI Foundry's native integration with Microsoft Purview

**Discussion Points:**

- What happens when AI models train on unclassified, ungovernered data?
- Real-world examples of AI governance failures and their consequences
- The difference between data governance for traditional analytics vs. AI workloads

**Important:** Azure AI Foundry (not Azure ML) provides the modern platform for governed AI development with built-in Purview integration points.

---

### Section 2: Microsoft Purview Overview

**Core Capabilities for AI Governance:**

| Capability | AI Use Case | Purview Feature |
|------------|-------------|-----------------|
| Data Map | Discover AI training data sources | Automated scanning, metadata extraction |
| Data Catalog | Search and understand AI data assets | Business glossary, asset curation |
| Data Classification | Identify PII, PHI, financial data | 300+ built-in classifiers, custom rules |
| Sensitivity Labels | Mark and protect sensitive AI data | Auto-labeling, encryption, access policies |
| Data Lineage | Trace data through AI pipelines | Automated lineage capture, visual explorer |
| Data Estate Insights | Monitor governance health | Dashboards, sensitivity distribution reports |
| Access Policies | Enforce data access controls | Policy-based access, self-service discovery |

**Setup Steps:**

1. Create a Microsoft Purview account
2. Assign governance roles (Data Curator, Data Reader, Data Source Admin)
3. Register Azure AI Foundry data sources
4. Configure collection hierarchy for organizational structure

**Instructor Notes:**

- Demo the Purview governance portal (https://purview.microsoft.com)
- Show how to register an Azure Blob Storage data source
- Emphasize the difference between Purview roles and Azure RBAC roles

---

### Section 3: Data Classification & Sensitivity Labels

#### 3.1 Built-in Classifiers

Purview provides 300+ pre-built sensitive information types organized by category:

- **PII**: SSN, passport numbers, email, phone numbers, national IDs
- **PHI**: Medical record numbers, diagnosis codes, drug names
- **Financial**: Credit card numbers, bank accounts, SWIFT codes
- **Credentials**: API keys, connection strings, certificates
- **Intellectual Property**: Source code patterns, algorithms

#### 3.2 Custom Classifiers for AI Data

For AI-specific use cases, create custom classifiers:

```python
custom_classification = {
    "name": "AI_Training_Dataset",
    "description": "Data used for AI model training in Foundry",
    "classificationRuleType": "Custom",
    "classificationRules": [
        {
            "kind": "RegexClassificationRule",
            "pattern": r"training[_-]data[_-]\d{8}",
            "minimumPercentageMatch": 60
        }
    ]
}
```

#### 3.3 Sensitivity Label Hierarchy

Design labels that reflect your AI data lifecycle:

```
Public         → Open datasets, documentation
General        → Internal training data, non-sensitive features
Confidential   → Customer data, PII-containing datasets
Highly Confidential → PHI, financial data, regulated datasets
```

#### 3.4 Auto-Labeling Policies

Configure automatic label application based on:
- Content inspection (sensitive info types detected)
- Data source location (regulated data stores)
- Custom conditions (file naming patterns, metadata)

**Instructor Notes:**

- Live demo: Run a classification scan on sample data with PII
- Show auto-labeling in action with a dataset containing SSNs
- Discuss the 85% confidence threshold for auto-labeling decisions

---

### Section 4: Data Lineage Tracking for AI Pipelines

#### 4.1 Why Lineage Matters for AI

- **Model Explainability**: Trace predictions back to training data sources
- **Impact Analysis**: Understand which models are affected by data changes
- **Regulatory Audits**: Prove data provenance for compliance requirements
- **Debugging**: Track down data quality issues across the pipeline

#### 4.2 Lineage Architecture

```
Raw Data → Data Prep → Feature Store → Model Training → Deployment
  ↕           ↕            ↕              ↕              ↕
        Microsoft Purview captures lineage at each stage
```

#### 4.3 Capturing Lineage in Azure AI Foundry

Purview captures lineage automatically from:
- Azure Data Factory pipelines
- Synapse Analytics data flows
- Azure AI Foundry pipeline runs
- Custom lineage via REST API or Atlas SDK

```python
lineage_payload = {
    "typeName": "azure_ai_foundry_pipeline",
    "attributes": {
        "name": "customer-churn-training-pipeline",
        "qualifiedName": "foundry://project/churn/pipeline/v2"
    },
    "inputs": [...],  # Source data assets
    "outputs": [...]  # Model artifacts
}
```

#### 4.4 Lineage Visualization

- **Visual Explorer**: Interactive graph showing data flow
- **Column-level lineage**: Track individual fields through transformations
- **Cross-system lineage**: Blob → Data Factory → AI Foundry → Endpoint

**Instructor Notes:**

- Demo the Purview lineage graph for a sample AI pipeline
- Show impact analysis: "What happens if this data source goes offline?"
- Discuss custom lineage registration for non-standard data sources

---

### Section 5: Compliance — GDPR, HIPAA & SOC 2

#### 5.1 GDPR Compliance for AI

| Article | AI Requirement | Purview Solution |
|---------|---------------|-----------------|
| Art. 13–14 | Transparency about AI decisions | Data catalog + lineage |
| Art. 15 | Right of access to personal data | Data map + subject rights |
| Art. 17 | Right to erasure from training data | Discovery + classification |
| Art. 22 | Human review of automated decisions | Lineage + audit logs |
| Art. 25 | Data protection by design | Sensitivity labels + policies |
| Art. 35 | Data Protection Impact Assessment | Data estate insights |

#### 5.2 HIPAA Compliance for Healthcare AI

Key requirements:
- BAA (Business Associate Agreement) with Microsoft
- PHI encryption at rest and in transit
- Access audit logging for all PHI interactions
- Minimum necessary access (least privilege)
- Data retention and destruction policies

#### 5.3 SOC 2 Trust Service Criteria

Five pillars mapped to AI governance:
1. **Security** — Access controls, encryption, network security
2. **Availability** — System uptime, disaster recovery
3. **Processing Integrity** — Accurate data processing with lineage
4. **Confidentiality** — Sensitivity labels, DLP policies
5. **Privacy** — Personal data handling, consent management

**Instructor Notes:**

- Use the compliance matrix to map requirements to Purview features
- Emphasize that compliance is a continuous process, not a one-time setup
- Discuss data retention policies for different regulatory frameworks

---

### Section 6: Purview × Azure AI Foundry Integration

#### 6.1 Integration Points

1. **Data Source Registration**: Register AI Foundry storage in Purview
2. **Automated Scanning**: Schedule scans of training data stores
3. **Classification Propagation**: Classifications flow to AI Foundry access controls
4. **Lineage Capture**: Pipeline runs automatically reported to Purview
5. **Policy Enforcement**: Purview policies control data access

#### 6.2 Governance Dashboard Metrics

| Metric | Description | Target |
|--------|-------------|--------|
| Classification Coverage | % of AI data assets classified | ≥ 95% |
| Label Coverage | % of sensitive assets with labels | 100% |
| Lineage Completeness | % of pipelines with lineage | ≥ 90% |
| Policy Violations | Access violations per month | 0 |
| Scan Freshness | Days since last scan | ≤ 7 |

---

### Section 7: Data Access Policies

#### Policy Types

1. **Self-Service Access** — Data consumers request access via catalog
2. **DevOps Policies** — Metadata read access for pipeline automation
3. **Data Owner Policies** — Stewards manage collection-level access
4. **Resource Set Policies** — Rules for groups of related data assets

#### Best Practices

- Always grant Purview managed identity **read-only** access to data stores
- Use collections to organize AI data assets by project, sensitivity, or team
- Implement approval workflows for sensitive data access requests
- Monitor and review access policies quarterly

---

## 🧪 Hands-On Activities

### Activity 1: Set Up Purview for AI Governance (20 min)
- Create Purview account
- Register AI data sources
- Configure collection hierarchy

### Activity 2: Classify AI Training Data (20 min)
- Run classification scan on sample data
- Review discovered classifications
- Create custom classifier for AI data patterns

### Activity 3: Track AI Pipeline Lineage (20 min)
- Register lineage for a sample pipeline
- Explore lineage graph
- Perform impact analysis

### Activity 4 (Mini Hack): End-to-End Data Classification (45 min)
- Provision complete governance environment
- Upload sample data with PII/PHI
- Configure auto-classification and sensitivity labels
- Verify compliance posture

---

## 📺 Video Resource

**AI Foundry Networking and Security: Deep Dive**

- URL: https://www.youtube.com/watch?v=bL9n83uPV84
- Duration: ~45 minutes
- Topics: Security architecture, governance integration, network controls

> Watch before or after the lecture for additional context on AI Foundry's security and governance architecture.

---

## ✅ Assessment

Complete the Module 32 quiz (10 multiple-choice questions) covering:
- Microsoft Purview core capabilities
- Data classification and sensitivity labels
- Data lineage tracking
- GDPR, HIPAA, and SOC 2 compliance requirements
- Purview × AI Foundry integration patterns

**Passing Score:** 80% (8 out of 10 correct)

---

## 📋 Key Terminology

| Term | Definition |
|------|-----------|
| **Microsoft Purview** | Unified data governance, risk, and compliance platform |
| **Data Map** | Automated discovery and cataloging of data assets |
| **Classification** | Process of identifying and tagging sensitive data |
| **Sensitivity Label** | Protective marking applied to data based on sensitivity |
| **Data Lineage** | Visualization of data flow from source to consumption |
| **PII** | Personally Identifiable Information |
| **PHI** | Protected Health Information |
| **GDPR** | General Data Protection Regulation (EU) |
| **HIPAA** | Health Insurance Portability and Accountability Act (US) |
| **SOC 2** | Service Organization Control 2 audit framework |
| **BAA** | Business Associate Agreement (HIPAA requirement) |
| **DLP** | Data Loss Prevention |
| **DPIA** | Data Protection Impact Assessment |

---

## 🔗 Additional Resources

- [Microsoft Purview Documentation](https://learn.microsoft.com/en-us/purview/)
- [Azure AI Foundry Documentation](https://learn.microsoft.com/en-us/azure/ai-studio/)
- [Data Classification Best Practices](https://learn.microsoft.com/en-us/purview/concept-best-practices-classification)
- [GDPR Compliance Center](https://learn.microsoft.com/en-us/compliance/regulatory/offering-gdpr)
- [HIPAA/HITECH Compliance](https://learn.microsoft.com/en-us/compliance/regulatory/offering-hipaa-hitech)
- [SOC 2 Audit Reports](https://learn.microsoft.com/en-us/compliance/regulatory/offering-soc-2)

---

## ➡️ Next Module

**Module 33: Threat Protection** — Learn about Azure AI Foundry threat detection, security monitoring, and incident response.

---

*Azure AI Foundry: Zero to Hero · ARC 7 · Module 32 · April 2026*
