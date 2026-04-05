# Module 44 — Assessment

## Add Governance, Monitoring & Security — Quiz

| Field         | Details                                        |
|---------------|------------------------------------------------|
| **Module**    | 44 of 45                                       |
| **Arc**       | ARC 9 — CAPSTONE & MASTERY                    |
| **Questions** | 10 Multiple Choice                             |
| **Passing**   | 80% (8 of 10)                                  |
| **Time**      | 15 minutes                                     |

---

### Question 1

**What is the correct formula for an Azure RBAC role assignment?**

- A) User + Subscription + API Key = Access
- B) Security Principal + Role Definition + Scope = Role Assignment
- C) Resource Group + Policy + Identity = Permission
- D) Tenant + Application + Secret = Authorization

<details>
<summary>Answer</summary>

**B) Security Principal + Role Definition + Scope = Role Assignment**

Azure RBAC follows the model of assigning a role (what permissions) to a security principal (who) at a specific scope (where). This applies to users, groups, service principals, and managed identities.

</details>

---

### Question 2

**Which built-in role should you assign to a developer who needs to call Azure OpenAI completion APIs but should NOT be able to manage model deployments?**

- A) Cognitive Services Contributor
- B) Cognitive Services OpenAI Contributor
- C) Cognitive Services OpenAI User
- D) Owner

<details>
<summary>Answer</summary>

**C) Cognitive Services OpenAI User**

The Cognitive Services OpenAI User role grants access to Azure OpenAI completion and embedding APIs without the ability to create, modify, or delete model deployments. The Contributor roles include deployment management permissions, which violates least-privilege in this scenario.

</details>

---

### Question 3

**What is the primary advantage of using Managed Identity over API keys for service-to-service authentication?**

- A) Managed Identity provides faster API response times
- B) Managed Identity eliminates the need to store and rotate credentials
- C) Managed Identity allows access from any Azure subscription
- D) Managed Identity bypasses RBAC role assignments

<details>
<summary>Answer</summary>

**B) Managed Identity eliminates the need to store and rotate credentials**

Managed Identity provides automatically managed credentials in Microsoft Entra ID, removing the need to embed API keys, connection strings, or secrets in code or configuration files. Credentials are automatically rotated by the Azure platform, reducing the risk of credential exposure.

</details>

---

### Question 4

**When should you choose a user-assigned managed identity over a system-assigned managed identity?**

- A) When the resource will be deleted frequently
- B) When multiple resources need to share the same identity for accessing common services
- C) When you want the identity to be automatically deleted with the resource
- D) When you only need read-only access to a single resource

<details>
<summary>Answer</summary>

**B) When multiple resources need to share the same identity for accessing common services**

User-assigned managed identities are standalone Azure resources with an independent lifecycle. They can be shared across multiple Azure resources (e.g., App Service, Container Apps, VMs), making them ideal for multi-service architectures where several components need the same access permissions.

</details>

---

### Question 5

**Which Python class should you use for credential-free authentication that works both locally (with Azure CLI) and in Azure (with Managed Identity)?**

- A) `AzureKeyCredential`
- B) `ClientSecretCredential`
- C) `DefaultAzureCredential`
- D) `InteractiveBrowserCredential`

<details>
<summary>Answer</summary>

**C) `DefaultAzureCredential`**

`DefaultAzureCredential` from the `azure-identity` package implements a credential chain that automatically tries multiple authentication methods in order: environment variables, managed identity, Azure CLI, Visual Studio Code, and interactive browser. This enables the same code to work seamlessly in local development and production Azure environments.

</details>

---

### Question 6

**You are configuring Azure Monitor alerts for your AI Foundry deployment. Which alert condition best detects API throttling issues?**

- A) Average CPU utilization > 80%
- B) HTTP 5xx error count > 10 in 5 minutes
- C) HTTP 429 response count > 50 per minute
- D) Average response time > 1000ms

<details>
<summary>Answer</summary>

**C) HTTP 429 response count > 50 per minute**

HTTP 429 is the "Too Many Requests" status code returned when the API rate limit is exceeded. Monitoring for 429 responses directly detects throttling. CPU utilization is infrastructure-level, 5xx errors indicate server failures (not throttling), and response time measures latency (not rate limiting).

</details>

---

### Question 7

**What is the purpose of Azure AI Content Safety's Prompt Shield feature?**

- A) Encrypting prompts before sending them to the model
- B) Detecting and blocking prompt injection and jailbreak attacks
- C) Compressing prompts to reduce token consumption
- D) Caching prompts for faster response times

<details>
<summary>Answer</summary>

**B) Detecting and blocking prompt injection and jailbreak attacks**

Prompt Shield is a content safety feature that analyzes incoming prompts for patterns commonly used in prompt injection and jailbreak attempts. It helps prevent attackers from manipulating the AI model into bypassing its system instructions or generating harmful outputs.

</details>

---

### Question 8

**You need to ensure that all Azure Cognitive Services resources in your subscription have diagnostic settings enabled. Which Azure service should you use to enforce this requirement automatically?**

- A) Azure Advisor
- B) Azure Policy
- C) Microsoft Defender for Cloud
- D) Azure Cost Management

<details>
<summary>Answer</summary>

**B) Azure Policy**

Azure Policy allows you to create and assign policy definitions that enforce compliance requirements across your Azure resources at scale. A built-in policy can audit or enforce that diagnostic settings are configured on all Cognitive Services resources, ensuring logs and metrics are captured consistently.

</details>

---

### Question 9

**What is the primary benefit of using private endpoints for Azure AI Services?**

- A) Private endpoints reduce the cost of API calls
- B) Private endpoints improve model inference accuracy
- C) Private endpoints ensure traffic stays within the Azure virtual network, blocking public internet access
- D) Private endpoints automatically scale the service to handle more requests

<details>
<summary>Answer</summary>

**C) Private endpoints ensure traffic stays within the Azure virtual network, blocking public internet access**

Private endpoints create a network interface in your VNet that connects to the Azure service through Azure Private Link. Combined with disabling public network access, this ensures all traffic between your applications and AI services flows through the private Microsoft backbone network, never traversing the public internet.

</details>

---

### Question 10

**You are completing the security hardening checklist for your AI Foundry capstone project. Which of the following is an anti-pattern that should be avoided?**

- A) Using `DefaultAzureCredential` in application code for authentication
- B) Assigning RBAC roles to Entra ID security groups instead of individual users
- C) Storing API keys directly in environment variables on the production VM
- D) Enabling diagnostic settings on all AI resources with a 90-day retention period

<details>
<summary>Answer</summary>

**C) Storing API keys directly in environment variables on the production VM**

While environment variables are better than hardcoding secrets in source code, they are still an anti-pattern for production workloads. API keys stored in environment variables can be exposed through process listings, crash dumps, or container inspection. The recommended approach is to use Managed Identity (eliminating keys entirely) or Azure Key Vault references for any required secrets.

</details>

---

## Scoring Guide

| Score | Grade | Status |
|-------|-------|--------|
| 10/10 | A+    | ⭐ Excellent — ready for capstone |
| 9/10  | A     | ✅ Strong understanding |
| 8/10  | B+    | ✅ Passing — review missed topics |
| 7/10  | B     | ⚠️ Below passing — review and retake |
| ≤6/10 | C     | ❌ Not passing — revisit module content |
