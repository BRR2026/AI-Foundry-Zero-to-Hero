# Module 27 Assessment: CI/CD Pipelines for AI

> **Azure AI Foundry: Zero to Hero** · Arc 6: Deployment & MLOps  
> **Module 27 of 45** · 10 Multiple Choice Questions · **Passing Score: 80%**

---

## Instructions

- Select the **one best answer** for each question.
- Each question is worth 10 points (100 points total).
- You need **80 points (8/10 correct)** to pass.
- Time limit: **20 minutes**

---

### Question 1

**What is the recommended authentication method for GitHub Actions workflows that deploy to Azure AI Foundry?**

A) Store a service principal client secret in GitHub Secrets  
B) Use a personal access token (PAT) with Azure CLI  
C) Workload Identity Federation (OIDC) with federated credentials  
D) Embed Azure credentials directly in the workflow YAML file  

<details>
<summary>✅ Answer</summary>

**C) Workload Identity Federation (OIDC) with federated credentials**

Workload Identity Federation uses short-lived OIDC tokens generated per workflow run. No secrets are stored in GitHub, eliminating credential rotation and leakage risks. The `azure/login@v2` action supports this natively with `client-id`, `tenant-id`, and `subscription-id` parameters.

</details>

---

### Question 2

**In a multi-stage CI/CD pipeline for Azure AI Foundry, what is the correct stage ordering?**

A) Deploy Production → Evaluate → Deploy Staging → Validate  
B) Validate → Deploy Staging → Evaluate → Deploy Production  
C) Evaluate → Validate → Deploy Staging → Deploy Production  
D) Deploy Staging → Deploy Production → Validate → Evaluate  

<details>
<summary>✅ Answer</summary>

**B) Validate → Deploy Staging → Evaluate → Deploy Production**

The correct flow is: first validate the infrastructure code (lint, build, what-if), then deploy to staging, run evaluations against staging to verify quality, and only then proceed to production (typically with a manual approval gate).

</details>

---

### Question 3

**Which Bicep resource type and `kind` value represents an Azure AI Foundry Hub?**

A) `Microsoft.CognitiveServices/accounts` with kind `AIServices`  
B) `Microsoft.MachineLearningServices/workspaces` with kind `Hub`  
C) `Microsoft.MachineLearningServices/workspaces` with kind `Workspace`  
D) `Microsoft.AIFoundry/hubs` with kind `Standard`  

<details>
<summary>✅ Answer</summary>

**B) `Microsoft.MachineLearningServices/workspaces` with kind `Hub`**

Azure AI Foundry Hubs and Projects are both represented as `Microsoft.MachineLearningServices/workspaces` resources. The `kind` property distinguishes them: `Hub` for hubs and `Project` for projects. An AI Foundry Project references its parent Hub via the `hubResourceId` property.

</details>

---

### Question 4

**What Azure CLI command should you run in a PR pipeline to preview infrastructure changes without actually deploying them?**

A) `az deployment group create --mode Complete`  
B) `az deployment group validate`  
C) `az deployment group what-if`  
D) `az bicep build`  

<details>
<summary>✅ Answer</summary>

**C) `az deployment group what-if`**

The `what-if` command shows what changes would be made to your Azure resources without actually executing them. This is the equivalent of `terraform plan` and provides a clear view of creates, updates, and deletes. `validate` only checks syntax, while `bicep build` just compiles Bicep to ARM JSON.

</details>

---

### Question 5

**In the evaluation quality gate pattern, what should the CI/CD pipeline do when a groundedness score falls below the defined threshold?**

A) Log a warning and continue to production deployment  
B) Exit with a non-zero code to fail the pipeline and block production  
C) Send an email notification but proceed with deployment  
D) Automatically retrain the model and retry  

<details>
<summary>✅ Answer</summary>

**B) Exit with a non-zero code to fail the pipeline and block production**

The evaluation gate is a hard quality checkpoint. If evaluation scores fall below thresholds, the script should call `sys.exit(1)` to return a non-zero exit code. This causes the CI/CD job to fail, which prevents downstream jobs (like production deployment) from executing.

</details>

---

### Question 6

**What GitHub Actions permission is required for OIDC-based authentication to Azure?**

A) `actions: write`  
B) `id-token: write`  
C) `deployments: write`  
D) `packages: write`  

<details>
<summary>✅ Answer</summary>

**B) `id-token: write`**

The `id-token: write` permission allows the GitHub Actions runner to request an OIDC token from GitHub's token endpoint. This token is then exchanged for an Azure access token via the federated credential configured on the Azure AD app registration. The `contents: read` permission is also typically needed to check out the repository.

</details>

---

### Question 7

**Which of the following is an advantage of using Bicep over Terraform for Azure AI Foundry deployments?**

A) Bicep supports multi-cloud deployments  
B) Bicep requires no external state file management  
C) Bicep has a larger community marketplace  
D) Bicep supports importing existing resources automatically  

<details>
<summary>✅ Answer</summary>

**B) Bicep requires no external state file management**

Bicep compiles to ARM templates and leverages Azure Resource Manager's built-in state tracking. There's no separate state file to manage, store, or lock — unlike Terraform which requires a remote backend (e.g., Azure Storage Account) for team collaboration. This simplifies operations for Azure-only deployments.

</details>

---

### Question 8

**In an Azure DevOps pipeline, which feature provides manual approval gates before production deployment?**

A) Variable groups  
B) Pipeline triggers  
C) Environments with approvals and checks  
D) Service connections  

<details>
<summary>✅ Answer</summary>

**C) Environments with approvals and checks**

Azure DevOps Environments support configurable approvals, business hours checks, and other gates that must pass before a deployment job targeting that environment can execute. This creates a human-in-the-loop checkpoint for production deployments.

</details>

---

### Question 9

**You want to ensure that your CI/CD pipeline deploys identical AI Foundry resources across dev, staging, and production. What is the best approach?**

A) Deploy manually in each environment using the Azure portal  
B) Use the same Bicep/Terraform template with environment-specific parameters  
C) Create separate IaC templates for each environment  
D) Use Azure Resource Mover to clone resources between environments  

<details>
<summary>✅ Answer</summary>

**B) Use the same Bicep/Terraform template with environment-specific parameters**

Using a single parameterized template ensures structural consistency across all environments. Only the values change (resource names, SKU sizes, capacity) — the architecture and configuration remain identical. This prevents configuration drift and makes it easy to promote changes through environments.

</details>

---

### Question 10

**Which Python package provides the `GroundednessEvaluator` and `RelevanceEvaluator` classes used for AI quality evaluation in CI/CD pipelines?**

A) `azure-ai-projects`  
B) `azure-ai-evaluation`  
C) `azure-ai-inference`  
D) `azure-cognitiveservices-vision`  

<details>
<summary>✅ Answer</summary>

**B) `azure-ai-evaluation`**

The `azure-ai-evaluation` package provides built-in evaluators including `GroundednessEvaluator`, `RelevanceEvaluator`, `CoherenceEvaluator`, `FluencyEvaluator`, and safety evaluators. It also provides the `evaluate()` function that runs evaluators against a dataset and returns aggregated metrics.

</details>

---

## 📊 Answer Key

| Question | Answer | Topic |
|---|---|---|
| 1 | C | Authentication — OIDC |
| 2 | B | Pipeline stage ordering |
| 3 | B | Bicep resource types |
| 4 | C | IaC change preview |
| 5 | B | Evaluation quality gates |
| 6 | B | GitHub Actions permissions |
| 7 | B | Bicep vs. Terraform |
| 8 | C | Azure DevOps approvals |
| 9 | B | Multi-environment IaC |
| 10 | B | AI Evaluation SDK |

---

## 🎯 Score Interpretation

| Score | Rating | Recommendation |
|---|---|---|
| 10/10 | ⭐ Excellent | Ready for the Mini Hack and Module 28 |
| 8–9/10 | ✅ Pass | Review the missed topics, then proceed |
| 6–7/10 | ⚠️ Needs Review | Re-read Sections 2–5 of the blog, redo the lab |
| ≤5/10 | ❌ Retry | Re-study the full module before retaking |

---

*Azure AI Foundry: Zero to Hero — Module 27 Assessment*  
*© 2026 AI Training Series*
