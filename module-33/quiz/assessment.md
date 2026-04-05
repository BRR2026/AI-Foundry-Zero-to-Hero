# Module 33: Assessment Quiz — Model Governance & Audit Trails

## Azure AI Foundry: Zero to Hero

**Arc 7: Security, Governance & Compliance** | 10 Questions | Passing Score: 80%

---

### Question 1

**What is the primary benefit of immutable model versioning in the Azure AI Foundry Model Registry?**

- A) It allows models to be updated in-place to save storage costs
- B) It ensures that once a model version is registered, its artifacts and metadata cannot be altered, guaranteeing audit integrity
- C) It automatically deploys new model versions to production endpoints
- D) It restricts model access to only the user who registered the model

<details>
<summary>✅ Show Answer</summary>

**Answer: B**

Immutable versioning means that once a model version is registered in the Azure AI Foundry Model Registry, its artifacts and metadata are locked. This guarantees that audit trails reference a known, unchangeable snapshot of the model, which is critical for compliance and reproducibility. Models cannot be modified after registration — a new version must be created instead.

</details>

---

### Question 2

**Which Azure service provides the primary audit log for tracking model lifecycle events (registrations, deployments, access changes) in Azure AI Foundry?**

- A) Azure Key Vault Audit Logs
- B) Azure Active Directory Sign-In Logs
- C) Azure Monitor Activity Log
- D) Azure Blob Storage Access Logs

<details>
<summary>✅ Show Answer</summary>

**Answer: C**

Azure Monitor Activity Log captures control-plane operations for Azure AI Foundry resources, including model registrations, deployments, and RBAC changes. The Activity Log records who performed what operation, when, and from where — providing the core audit trail for governance compliance.

</details>

---

### Question 3

**In a risk-tier-based approval workflow, which requirement is typically expected for high-risk models but NOT for low-risk models?**

- A) Model must be registered in the Model Registry
- B) At least one reviewer must approve the model
- C) A fairness evaluation and model card must be completed before promotion
- D) The model must be trained using Azure compute

<details>
<summary>✅ Show Answer</summary>

**Answer: C**

High-risk models typically require stricter governance controls including fairness evaluations, comprehensive model cards, and multiple reviewer approvals (often including a compliance officer). Low-risk models may only need a single ML lead approval and may not require a model card or fairness evaluation, though all models must be registered in the registry.

</details>

---

### Question 4

**What is the purpose of model cards in the context of model governance?**

- A) To encrypt model artifacts during storage and transmission
- B) To provide standardized documentation covering intended use, performance, limitations, and ethical considerations
- C) To automate model deployment to production endpoints
- D) To manage network access controls for model inference endpoints

<details>
<summary>✅ Show Answer</summary>

**Answer: B**

Model cards are standardized documents that describe a model's intended use, performance characteristics, limitations, ethical considerations, and governance metadata. They serve as the primary transparency artifact for governance, helping stakeholders understand what a model can and cannot do. They are also a key compliance artifact for the EU AI Act Article 13 (Transparency).

</details>

---

### Question 5

**Which KQL query correctly filters Azure Activity Log for model deployment events in Azure AI Foundry?**

- A)
  ```
  AzureActivity
  | where ResourceProvider == "Microsoft.CognitiveServices"
  | where OperationNameValue has "models/write"
  ```
- B)
  ```
  AzureActivity
  | where ResourceProvider == "Microsoft.MachineLearningServices"
  | where OperationNameValue has "deployments/write"
  ```
- C)
  ```
  SecurityEvent
  | where ResourceProvider == "Microsoft.MachineLearningServices"
  | where EventID == 4624
  ```
- D)
  ```
  AzureMetrics
  | where ResourceProvider == "Microsoft.AzureAI"
  | where MetricName == "DeploymentCount"
  ```

<details>
<summary>✅ Show Answer</summary>

**Answer: B**

Azure AI Foundry uses the `Microsoft.MachineLearningServices` resource provider. Deployment events are logged in the Azure Activity Log table `AzureActivity` with operation names containing `deployments/write`. The correct query filters by both the resource provider and the deployment operation.

</details>

---

### Question 6

**When implementing approval gates with GitHub Actions, which feature provides the human review requirement before model deployment?**

- A) GitHub Actions matrix strategy
- B) GitHub Environments with required reviewers
- C) GitHub Branch protection rules with status checks
- D) GitHub Issues with approval labels

<details>
<summary>✅ Show Answer</summary>

**Answer: B**

GitHub Environments allow you to configure required reviewers who must manually approve a workflow run before jobs targeting that environment can execute. This is the mechanism used to implement approval gates in model promotion pipelines — the deployment job references an environment (e.g., `production`) that has required reviewers configured, pausing execution until approval is granted.

</details>

---

### Question 7

**What is the recommended minimum audit log retention period for compliance with most regulatory frameworks, including the EU AI Act?**

- A) 30 days
- B) 90 days
- C) 365 days
- D) 7 days

<details>
<summary>✅ Show Answer</summary>

**Answer: C**

Most regulatory frameworks require audit logs to be retained for a minimum of one year (365 days). The EU AI Act requires that logs for high-risk AI systems be retained for at least the duration of the system's operational period. Azure Monitor supports long-term retention with archival tiers for cost-effective compliance.

</details>

---

### Question 8

**Which of the following is a best practice for using tags to manage model lifecycle stages in the Azure AI Foundry Model Registry?**

- A) Encode the lifecycle stage in the model version number (e.g., version "3-production")
- B) Use separate model names for each stage (e.g., "sentiment-model-staging", "sentiment-model-production")
- C) Apply tags like "stage": "candidate", "stage": "production" to track lifecycle stages while keeping version numbers immutable
- D) Create a new model registration for each stage transition with a new version number

<details>
<summary>✅ Show Answer</summary>

**Answer: C**

Tags should be used to indicate model lifecycle stages rather than encoding stage names in version numbers or model names. This allows the same model version to progress through stages (candidate → staging → production) while maintaining clean, immutable version identifiers. Tags are the correct metadata mechanism for stage tracking in Azure AI Foundry.

</details>

---

### Question 9

**Which section of a model card is specifically required by the EU AI Act Article 13 for high-risk AI systems?**

- A) Training compute specifications and cost analysis
- B) Transparency information including intended use, limitations, and conditions under which the system performs as intended
- C) A list of all Python packages used in the training environment
- D) Marketing materials and competitive analysis

<details>
<summary>✅ Show Answer</summary>

**Answer: B**

EU AI Act Article 13 requires that high-risk AI systems be designed to ensure sufficient transparency for deployers to interpret the system's output and use it appropriately. This includes documentation of intended purpose, the level of accuracy/robustness, known limitations, and any circumstances under which the system may pose risks — all of which are captured in a model card's core sections.

</details>

---

### Question 10

**In a model governance pipeline, what should happen when a high-risk model is registered without the required governance metadata tags?**

- A) The model should be automatically deployed to a staging environment for testing
- B) The registration should succeed, but the model should be blocked from promotion by the approval gate until all required tags are present
- C) The model registration should be silently deleted from the registry
- D) An email should be sent to the model owner, but deployment should proceed normally

<details>
<summary>✅ Show Answer</summary>

**Answer: B**

The model should be allowed to register in the Model Registry (to preserve artifacts and prevent data loss), but the approval gate should block any promotion or deployment until all required governance metadata is present. This separation of concerns ensures that data scientists can iterate on models while governance controls prevent non-compliant models from reaching production. The governance validation script checks for required tags before any approval is granted.

</details>

---

## 📊 Scoring Guide

| Score | Rating | Recommendation |
|-------|--------|----------------|
| 10/10 | ⭐ Excellent | Ready for Module 34 — Data Privacy & Protection |
| 8-9/10 | ✅ Proficient | Review any missed topics, then proceed |
| 6-7/10 | 📖 Developing | Review the blog article and retry the lab exercises |
| Below 6 | 🔄 Needs Review | Re-study the module content and complete the lab before retaking |

---

*Module 33 Assessment — Azure AI Foundry: Zero to Hero — Arc 7: Security, Governance & Compliance*
*© 2026 Azure AI Foundry Training*
