# Module 32 Assessment: Data Governance & Purview Integration

## Azure AI Foundry: Zero to Hero

> **ARC 7 · SECURITY, GOVERNANCE & COMPLIANCE** | 10 Questions | Passing Score: 80% (8/10)

---

## Instructions

- Select the **one best answer** for each question.
- Each question is worth 10 points (100 total).
- You need **80 points (8 correct)** to pass.
- Review the Module 32 blog, training guide, and hands-on lab before attempting.

---

### Question 1: Microsoft Purview Core Function

**What is the primary role of Microsoft Purview in an Azure AI Foundry governance strategy?**

- A) Replacing Azure AI Foundry as the model training platform
- B) Providing a unified platform for data governance, risk management, and compliance across AI data assets
- C) Serving as the primary compute engine for AI model inference
- D) Managing Azure subscription billing and cost optimization

<details>
<summary>✅ Show Answer</summary>

**Correct Answer: B**

Microsoft Purview is a unified data governance, risk, and compliance platform. It does not replace AI Foundry for model training (A), serve as a compute engine (C), or manage billing (D). Purview provides data discovery, classification, lineage, and compliance features that form the governance backbone for AI workloads.

</details>

---

### Question 2: Data Classification Capabilities

**How many built-in sensitive information types does Microsoft Purview provide for automated data classification?**

- A) 50+
- B) 100+
- C) 300+
- D) 1,000+

<details>
<summary>✅ Show Answer</summary>

**Correct Answer: C**

Microsoft Purview includes over **300 built-in sensitive information types** that automatically detect PII, PHI, financial data, credentials, and other sensitive patterns during classification scans. These can be supplemented with custom classifiers for AI-specific data patterns.

</details>

---

### Question 3: Sensitivity Label Hierarchy

**In a properly designed sensitivity label hierarchy for AI data, which label should be applied to datasets containing Protected Health Information (PHI)?**

- A) Public
- B) General
- C) Confidential
- D) Highly Confidential

<details>
<summary>✅ Show Answer</summary>

**Correct Answer: D**

PHI (Protected Health Information) requires the highest level of protection — **Highly Confidential**. This label enforces encryption, strict access controls, and full audit logging. PHI is regulated under HIPAA, which mandates the most rigorous data protection standards.

- Public: Open datasets, documentation
- General: Internal, non-sensitive training data
- Confidential: Customer data, PII
- Highly Confidential: PHI, financial, heavily regulated data

</details>

---

### Question 4: Data Lineage Purpose

**What critical question does data lineage in AI pipelines answer?**

- A) How much compute resources did the model training consume?
- B) Where did the model's training data come from, and how was it transformed?
- C) What is the current market price of Azure AI Foundry services?
- D) Which team members have logged into the Azure portal today?

<details>
<summary>✅ Show Answer</summary>

**Correct Answer: B**

Data lineage traces data from its source through all transformations to the final model output, answering the question: **"Where did this model's training data come from, and how was it transformed?"** This is essential for model explainability, regulatory audits, impact analysis, and debugging data quality issues.

</details>

---

### Question 5: GDPR Article 17 Compliance

**Under GDPR Article 17 (Right to Erasure), what Purview capability helps organizations locate personal data that a data subject requests to be deleted from AI training sets?**

- A) Data lineage visualization
- B) Data discovery and classification scans
- C) Business glossary management
- D) Data access policy enforcement

<details>
<summary>✅ Show Answer</summary>

**Correct Answer: B**

GDPR Article 17 grants data subjects the right to have their personal data erased. To comply, organizations must first **discover and locate** where that personal data exists across their AI data estate. Purview's **data discovery and classification scans** identify PII across all registered data sources, enabling organizations to find and remove the data upon request.

</details>

---

### Question 6: Purview Access to Data Sources

**What level of access should Microsoft Purview's managed identity be granted to AI Foundry data stores for governance scanning?**

- A) Storage Blob Data Contributor (read + write)
- B) Storage Blob Data Owner (full control)
- C) Storage Blob Data Reader (read-only)
- D) Storage Account Contributor (management access)

<details>
<summary>✅ Show Answer</summary>

**Correct Answer: C**

Purview requires **read-only** access (Storage Blob Data Reader) to scan and classify data. Granting write access is a security risk because Purview only needs to read metadata and content for classification — it should never modify the underlying AI data. Write access should be limited to AI Foundry pipeline identities only.

</details>

---

### Question 7: HIPAA Compliance Requirement

**Which of the following is a mandatory HIPAA requirement when using AI models that process Protected Health Information (PHI) in Azure AI Foundry?**

- A) Publishing all PHI data to a public data catalog
- B) Having a Business Associate Agreement (BAA) with Microsoft
- C) Storing PHI data in an unencrypted format for faster model training
- D) Granting all team members unrestricted access to PHI data

<details>
<summary>✅ Show Answer</summary>

**Correct Answer: B**

HIPAA requires a **Business Associate Agreement (BAA)** with any cloud provider that processes PHI. Microsoft provides a BAA as part of Azure's HIPAA compliance. All other options violate HIPAA: PHI must never be public (A), must be encrypted (C), and access must follow the minimum necessary principle (D).

</details>

---

### Question 8: SOC 2 Trust Service Criteria

**Which SOC 2 Trust Service Criterion is most directly supported by Microsoft Purview's data lineage tracking capability?**

- A) Security
- B) Availability
- C) Processing Integrity
- D) Privacy

<details>
<summary>✅ Show Answer</summary>

**Correct Answer: C**

**Processing Integrity** requires that system processing is complete, valid, accurate, and authorized. Data lineage tracking directly supports this by providing evidence that data was processed correctly through documented, traceable transformations from source to output. While lineage can support other criteria, it most directly addresses processing integrity.

</details>

---

### Question 9: Governance Dashboard Metrics

**What is the recommended target for classification coverage of AI data assets in a well-governed Azure AI Foundry environment?**

- A) ≥ 50%
- B) ≥ 75%
- C) ≥ 95%
- D) Exactly 100%

<details>
<summary>✅ Show Answer</summary>

**Correct Answer: C**

The recommended target for **classification coverage** is **≥ 95%** of all AI data assets. While 100% is ideal, some dynamically generated or temporary assets may not be immediately classified. The key governance metrics are:

- Classification Coverage: ≥ 95%
- Label Coverage (for sensitive assets): 100%
- Lineage Completeness: ≥ 90%
- Policy Violations: 0 per month
- Scan Freshness: ≤ 7 days

</details>

---

### Question 10: Purview × AI Foundry Integration

**When integrating Microsoft Purview with Azure AI Foundry, which is the correct sequence of steps?**

- A) Apply sensitivity labels → Register data sources → Run classification scans → Configure access policies
- B) Register data sources → Run classification scans → Apply sensitivity labels → Configure access policies
- C) Configure access policies → Apply sensitivity labels → Register data sources → Run classification scans
- D) Run classification scans → Configure access policies → Register data sources → Apply sensitivity labels

<details>
<summary>✅ Show Answer</summary>

**Correct Answer: B**

The correct integration sequence is:

1. **Register data sources** — Add AI Foundry storage accounts, databases, and data lakes to Purview Data Map
2. **Run classification scans** — Discover and classify sensitive data in the registered sources
3. **Apply sensitivity labels** — Mark classified assets with appropriate protection labels
4. **Configure access policies** — Enforce data access controls based on classifications and labels

You must register and scan before you can label, and labels must exist before access policies can reference them.

</details>

---

## 📊 Scoring Guide

| Score | Result | Recommendation |
|-------|--------|----------------|
| 10/10 | ⭐ Excellent | Ready for Module 33: Threat Protection |
| 8–9/10 | ✅ Pass | Review missed topics, then proceed |
| 6–7/10 | ⚠️ Needs Review | Re-read relevant sections and retry |
| ≤ 5/10 | ❌ Not Ready | Complete the hands-on lab, review all materials |

---

## 📝 Answer Key (Quick Reference)

| Question | Answer | Topic |
|----------|--------|-------|
| 1 | B | Purview core function |
| 2 | C | Classification capabilities |
| 3 | D | Sensitivity label hierarchy |
| 4 | B | Data lineage purpose |
| 5 | B | GDPR Article 17 compliance |
| 6 | C | Purview access permissions |
| 7 | B | HIPAA BAA requirement |
| 8 | C | SOC 2 processing integrity |
| 9 | C | Classification coverage target |
| 10 | B | Integration sequence |

---

*Azure AI Foundry: Zero to Hero · Module 32 Assessment · ARC 7: Security, Governance & Compliance · April 2026*
