# 📝 Module 31 Assessment — Identity & Access Management

## Azure AI Foundry: Zero to Hero

> **Arc 7: Security, Governance & Compliance** · 🔴 Advanced Tier · 10 Questions · Pass Mark: 80%

---

## Instructions

- Select the **single best answer** for each question
- Each question is worth **10 points** (100 total)
- You need **80 points (8/10)** to pass
- Answers are provided at the end — try without peeking first!

---

### Question 1

**Which Azure RBAC role allows a user to create and manage all types of resources but does NOT allow them to grant access to other users?**

- A) Owner
- B) Contributor
- C) User Access Administrator
- D) Reader

---

### Question 2

**An ML engineer needs to deploy models and manage inference endpoints in Azure AI Foundry, but should not be able to delete the AI Foundry account itself. Which role is most appropriate?**

- A) Owner
- B) Contributor
- C) Azure AI Inference Deployment Operator
- D) Reader

---

### Question 3

**What is the primary advantage of using a managed identity over a service principal with a client secret?**

- A) Managed identities support multi-factor authentication
- B) Managed identities can be assigned to more role types
- C) Azure automatically handles credential lifecycle — no secrets to store or rotate
- D) Managed identities have higher API rate limits

---

### Question 4

**A production application hosted on Azure App Service needs to call an Azure AI Foundry endpoint. Following least-privilege principles, how should authentication be configured?**

- A) Store an API key in the App Service application settings
- B) Use a system-assigned managed identity with the Cognitive Services User role scoped to the specific AI Foundry resource
- C) Create a service principal with Contributor role at the subscription level
- D) Use a shared access signature (SAS) token stored in Azure Key Vault

---

### Question 5

**What is the key difference between a system-assigned and a user-assigned managed identity?**

- A) System-assigned identities are faster for authentication
- B) User-assigned identities can only be used with virtual machines
- C) A system-assigned identity is tied to a single resource and is deleted with it; a user-assigned identity is an independent resource that can be shared across multiple resources
- D) System-assigned identities support custom role assignments; user-assigned identities do not

---

### Question 6

**A CI/CD pipeline in GitHub Actions needs to deploy models to Azure AI Foundry. Which authentication approach eliminates the need for long-lived secrets?**

- A) Store a client secret in GitHub Secrets and use `azure/login` with `creds`
- B) Use Workload Identity Federation with OIDC token exchange
- C) Use a personal access token (PAT) for Azure
- D) Embed the Azure CLI credentials in the workflow YAML file

---

### Question 7

**When defining a custom Azure RBAC role, what is the purpose of the `NotActions` property?**

- A) It specifies actions that are explicitly denied regardless of other role assignments
- B) It excludes specific actions from the set defined in `Actions`, narrowing the effective permissions
- C) It defines actions that trigger alerts in Azure Monitor
- D) It lists actions that require additional approval before execution

---

### Question 8

**Which Python class from the `azure-identity` library automatically selects the appropriate credential based on the environment — managed identity in Azure, Azure CLI locally?**

- A) `ManagedIdentityCredential`
- B) `ClientSecretCredential`
- C) `DefaultAzureCredential`
- D) `AzureCliCredential`

---

### Question 9

**According to least-privilege best practices, how should standing Owner access be managed for Azure AI Foundry subscriptions?**

- A) Assign Owner permanently to the team lead only
- B) Use Azure Privileged Identity Management (PIM) for just-in-time, time-bound Owner activation
- C) Create a custom role with all Owner permissions except role assignment
- D) Use a shared Owner account with a strong password rotated monthly

---

### Question 10

**A data scientist needs to query an Azure AI Search index used by a RAG application and read training data from Blob Storage, but should not be able to modify any resources. Which combination of roles and scope follows least-privilege principles?**

- A) Contributor at the resource group scope
- B) Search Index Data Reader on the Search resource + Storage Blob Data Reader on the Storage account
- C) Reader at the subscription scope
- D) Cognitive Services Contributor at the resource group scope + Storage Blob Data Contributor on the Storage account

---

## ✅ Answer Key

| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **B** | **Contributor** can create and manage resources but cannot assign roles. Owner (A) can assign roles. User Access Administrator (C) specifically manages access. Reader (D) is view-only. |
| 2 | **C** | **Azure AI Inference Deployment Operator** allows deploying models and managing endpoints without full resource management. Owner (A) and Contributor (B) are too broad. Reader (D) is too restrictive. |
| 3 | **C** | Managed identities have **Azure-managed credentials** — no secrets to store, rotate, or risk exposing. MFA (A) applies to human identities. Role support (B) and rate limits (D) are the same. |
| 4 | **B** | A **system-assigned managed identity** with **Cognitive Services User** scoped to the specific resource follows least privilege with zero secrets. API keys (A) are legacy. Subscription-level Contributor (C) is over-privileged. SAS tokens (D) are for storage, not AI services. |
| 5 | **C** | **System-assigned** identities are tied to and deleted with their resource; **user-assigned** identities are independent Azure resources that can be shared. Performance (A) is equivalent. User-assigned works with many resource types (B). Both support custom roles (D). |
| 6 | **B** | **Workload Identity Federation** exchanges OIDC tokens from GitHub Actions for Azure AD tokens — no long-lived secrets needed. Client secrets (A) require rotation. PATs (C) and embedded creds (D) are insecure. |
| 7 | **B** | `NotActions` **subtracts** actions from the `Actions` set, narrowing effective permissions. It is NOT an explicit deny (A) — deny assignments are a separate concept. It doesn't trigger alerts (C) or require approvals (D). |
| 8 | **C** | **`DefaultAzureCredential`** tries multiple credential sources in order: environment variables, managed identity, Azure CLI, etc. `ManagedIdentityCredential` (A) only works with managed identity. `ClientSecretCredential` (B) requires explicit secrets. `AzureCliCredential` (D) only uses CLI auth. |
| 9 | **B** | **Azure PIM** provides just-in-time, time-bound activation for privileged roles, with approval workflows and audit trails. Permanent assignment (A) violates least privilege. Custom pseudo-Owner roles (C) are complex and miss the JIT benefit. Shared accounts (D) violate identity best practices. |
| 10 | **B** | **Search Index Data Reader** + **Storage Blob Data Reader** at individual resource scopes provides exactly the needed read permissions. Contributor (A) and Contributor-level roles (D) are over-privileged. Subscription-scope Reader (C) is too broad in scope. |

---

## 📊 Scoring Guide

| Score | Rating | Recommendation |
|-------|--------|----------------|
| 10/10 | ⭐ Excellent | You've mastered IAM for AI Foundry! Ready for Module 32. |
| 8–9/10 | ✅ Pass | Solid understanding. Review any missed questions. |
| 6–7/10 | ⚠️ Needs Review | Re-read the training guide sections related to missed questions. |
| < 6/10 | ❌ Retry | Complete the hands-on lab and re-read the full module before retrying. |

---

> **Module 31 Assessment** · Azure AI Foundry: Zero to Hero · Arc 7: Security, Governance & Compliance  
> 🔴 Advanced Tier · April 2026
