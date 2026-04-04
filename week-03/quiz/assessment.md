# Week 3 Assessment: AI Foundry Architecture & Components

## Zero to Hero: Azure AI Foundry Training Course

**Arc 1: Foundations | Week 3 of 45**

**Total Points: 100 | Passing Score: 70 | Time Limit: 45 minutes**

---

## Section 1: Multiple Choice (5 points each, 40 points total)

### Question 1
What is the correct resource hierarchy in Azure AI Foundry, from top to bottom?

A) Subscription → Hub → Resource Group → Project  
B) Resource Group → Subscription → Hub → Project  
C) Subscription → Resource Group → AI Foundry Account (Hub) → Project  
D) Subscription → Resource Group → Project → Hub  

---

### Question 2
Which of the following Azure resources is **NOT** automatically provisioned when you create an AI Foundry Hub?

A) Storage Account  
B) Key Vault  
C) Azure OpenAI Service  
D) Application Insights  

---

### Question 3
A company needs guaranteed throughput for their production GPT-4o chatbot serving 10,000 users daily. Which deployment type should they choose?

A) Global Standard  
B) Standard  
C) Provisioned Throughput (PTU)  
D) Serverless API (MaaS)  

---

### Question 4
Which RBAC role should you assign to a developer who needs to deploy models and create connections but should NOT manage access permissions?

A) Owner  
B) Contributor  
C) Azure AI Developer  
D) Reader  

---

### Question 5
When using a **Global Standard** deployment for Azure OpenAI, which statement is TRUE?

A) Data is guaranteed to stay within your selected region  
B) Data may be processed in any Azure region globally  
C) It provides the highest throughput guarantees  
D) It requires a monthly commitment  

---

### Question 6
Which networking option provides the HIGHEST security posture for AI Foundry?

A) Public Access with IP restrictions  
B) Private Endpoints  
C) Managed VNet (fully private)  
D) Service Endpoints  

---

### Question 7
Hub-level connections in AI Foundry are:

A) Only accessible by the first project created in the Hub  
B) Inherited by all projects within the Hub  
C) Automatically deleted when a project is removed  
D) Only available when using private endpoints  

---

### Question 8
Which Python SDK package would you use to perform chat completions against a model deployed in AI Foundry?

A) `azure-ai-ml`  
B) `azure-ai-projects`  
C) `azure-ai-inference`  
D) `azure-cognitiveservices`  

---

## Section 2: True or False (3 points each, 15 points total)

### Question 9
**True or False:** The AI Foundry Hub and Project resources themselves incur a direct charge on your Azure bill.

---

### Question 10
**True or False:** A Project can be moved from one Hub to another after creation.

---

### Question 11
**True or False:** Managed Identity is the recommended authentication method for AI Foundry resources accessing other Azure services.

---

### Question 12
**True or False:** Serverless API (MaaS) deployments require you to manage the underlying virtual machines.

---

### Question 13
**True or False:** Application Insights data ingestion beyond the free tier (5 GB/month) can become a significant cost factor.

---

## Section 3: Short Answer (5 points each, 15 points total)

### Question 14
List **three** scenarios where you would use **multiple Hubs** instead of multiple Projects under a single Hub.

---

### Question 15
Explain the difference between a **System-assigned Managed Identity** and a **User-assigned Managed Identity** in the context of AI Foundry. When would you use each?

---

### Question 16
A startup is building a prototype chatbot. They have a budget of $50/month and expect low usage (~100K tokens/month). Recommend a deployment type and explain your reasoning.

---

## Section 4: Matching (2 points each, 10 points total)

### Question 17
Match each Azure resource to its primary role in the AI Foundry architecture.

| Resource | | Role |
|----------|---|------|
| 1. Storage Account | | A. Stores API keys and connection strings securely |
| 2. Key Vault | | B. Hosts custom container images and environments |
| 3. Application Insights | | C. Stores data, model artifacts, logs, and flow definitions |
| 4. Container Registry | | D. Provides monitoring, telemetry, and diagnostics |
| 5. Azure OpenAI Service | | E. Hosts and serves large language models |

---

## Section 5: Diagram-Based Questions (10 points each, 20 points total)

### Question 18
Study the following architecture diagram and answer the questions below:

```
┌──────────────────────────────────────────────┐
│              Resource Group                   │
│                                               │
│  ┌──────── Hub: hub-acme ──────────────────┐ │
│  │                                          │ │
│  │  ┌── Project A ──┐  ┌── Project B ──┐  │ │
│  │  │ GPT-4o (Std)   │  │ Llama-3 (???)  │  │ │
│  │  │ AI Search conn │  │ Blob conn      │  │ │
│  │  └────────────────┘  └────────────────┘  │ │
│  │                                          │ │
│  │  Shared: Storage, Key Vault, App Insights│ │
│  └──────────────────────────────────────────┘ │
└──────────────────────────────────────────────┘
```

**a)** (5 points) Project B deploys Llama-3, an open-source model. What deployment type is most likely used? Explain why.

**b)** (5 points) If the AI Search connection is defined at the Hub level, can Project B also use it? Explain.

---

### Question 19
Consider this networking diagram:

```
┌──────────────────────────────────────┐
│            Customer VNet             │
│                                      │
│  ┌──────┐    ┌──────────────────┐   │
│  │ App  │───►│ Private Endpoint │   │
│  │ VM   │    │ (AI Foundry Hub) │   │
│  └──────┘    └────────┬─────────┘   │
│                       │              │
│              ┌────────▼─────────┐   │
│              │ Private Endpoint  │   │
│              │ (Storage Account) │   │
│              └──────────────────┘   │
└──────────────────────────────────────┘
```

**a)** (5 points) What is the benefit of using Private Endpoints for both the Hub and the Storage Account?

**b)** (5 points) What additional Azure configuration is required to make Private Endpoints work for DNS resolution? Name the specific Azure resource.

---

## Answer Key

---

### Section 1: Multiple Choice

**Q1: C** — Subscription → Resource Group → AI Foundry Account (Hub) → Project  
*Explanation:* The AI Foundry hierarchy follows Azure's standard resource model. The Subscription is the billing boundary, Resource Groups organize resources by lifecycle, the Hub provides shared infrastructure, and Projects are team-level workspaces.

**Q2: C** — Azure OpenAI Service  
*Explanation:* When you create a Hub, Azure automatically provisions a Storage Account, Key Vault, Application Insights, and optionally a Container Registry. Azure OpenAI is connected as a separate resource through a Connection — it is not auto-provisioned with the Hub.

**Q3: C** — Provisioned Throughput (PTU)  
*Explanation:* Provisioned Throughput provides guaranteed, reserved capacity with consistent latency. For a production chatbot with 10,000 daily users, predictable performance is critical. Global Standard and Standard are pay-per-token with variable latency, and MaaS is for partner models.

**Q4: C** — Azure AI Developer  
*Explanation:* The Azure AI Developer role grants permissions to deploy models, create connections, and run experiments, but does not include access management permissions. Owner can manage access, Contributor has broad resource permissions, and Reader is view-only.

**Q5: B** — Data may be processed in any Azure region globally  
*Explanation:* Global Standard deployments route requests to any available Azure region to maximize availability and throughput. This means your data may leave your selected region. For data residency, use Standard or Provisioned deployments.

**Q6: C** — Managed VNet (fully private)  
*Explanation:* Managed VNet provides the highest security by keeping all traffic on private networks with no public endpoint exposure. Private Endpoints are the next most secure, and Public Access with IP restrictions is the least secure of the three main options.

**Q7: B** — Inherited by all projects within the Hub  
*Explanation:* Hub-level connections are shared resources available to all child projects. This is useful for central services like Azure OpenAI or AI Search that multiple teams need to access. Project-level connections are isolated to that project.

**Q8: C** — `azure-ai-inference`  
*Explanation:* The `azure-ai-inference` package provides the `ChatCompletionsClient` for performing chat completions, embeddings, and other model inference operations. `azure-ai-ml` is for workspace management, `azure-ai-projects` is for project-level operations.

---

### Section 2: True or False

**Q9: False**  
*Explanation:* The Hub and Project resources themselves are free. You pay for the underlying Azure resources they use — Storage Account, Key Vault, compute, model inference tokens, etc.

**Q10: False**  
*Explanation:* A Project is permanently bound to its parent Hub. Once created, it cannot be moved to a different Hub. You would need to create a new Project in the target Hub and migrate your assets.

**Q11: True**  
*Explanation:* Managed Identity eliminates the need for secrets in code and is the Microsoft-recommended authentication method. It provides automatic credential management and rotation.

**Q12: False**  
*Explanation:* Serverless API (MaaS) is fully managed by Microsoft. You don't provision, manage, or scale any virtual machines. You simply call the API and pay per token consumed.

**Q13: True**  
*Explanation:* Application Insights charges per GB of data ingested beyond the free 5 GB/month. With heavy telemetry, this can reach $50-100+/month, making it a frequently overlooked cost factor.

---

### Section 3: Short Answer

**Q14:** Three scenarios for multiple Hubs (sample answers):

1. **Data residency / compliance requirements** — Different Hubs in different regions (e.g., US Hub for US data, EU Hub for GDPR-compliant EU data)
2. **Different networking policies** — One Hub with public access for development, another Hub with private endpoints for production
3. **Separate billing or organizational boundaries** — Different business units or departments that need completely isolated infrastructure and cost tracking

*Also acceptable:* Different security domains (HIPAA vs non-HIPAA), sovereign cloud requirements, disaster recovery across regions.

**Q15:**

- **System-assigned Managed Identity:** Created automatically with the Hub resource and tied to its lifecycle. If the Hub is deleted, the identity is deleted too. Use when the identity is exclusively for that one Hub's resources.
- **User-assigned Managed Identity:** Created independently as a standalone Azure resource. Can be shared across multiple resources. Use when multiple Hubs, projects, or other Azure resources need the same identity and access permissions.

**Q16:**

Recommended: **Global Standard** deployment.

Reasoning: Global Standard has the lowest per-token cost and zero minimum monthly commitment. At ~100K tokens/month with GPT-4o, the cost would be approximately $0.75/month for inference alone, well within the $50 budget. The remaining budget covers Storage (~$2), Key Vault (~$1), and other minimal infrastructure costs. Global Standard's variable latency is acceptable for a prototype. Provisioned Throughput ($2,000+/month) and even Standard deployments would be overkill for this usage level.

---

### Section 4: Matching

**Q17:**

| Resource | Match | Role |
|----------|-------|------|
| 1. Storage Account | **C** | Stores data, model artifacts, logs, and flow definitions |
| 2. Key Vault | **A** | Stores API keys and connection strings securely |
| 3. Application Insights | **D** | Provides monitoring, telemetry, and diagnostics |
| 4. Container Registry | **B** | Hosts custom container images and environments |
| 5. Azure OpenAI Service | **E** | Hosts and serves large language models |

---

### Section 5: Diagram-Based

**Q18a:**  
Llama-3 is an open-source model, so it would most likely use **Managed Compute** (Managed Online Endpoint). Serverless API (MaaS) deployments are primarily for select Microsoft and partner models through the model catalog. Open-source models like Llama-3 typically need to be deployed on dedicated compute (VMs with GPUs) as Managed Online Endpoints, giving you full control over the infrastructure. *Note: Some OSS models are now available as MaaS — full credit for either answer if properly justified.*

**Q18b:**  
Yes, Project B can also use the AI Search connection. Hub-level connections are **inherited by all child projects** within that Hub. Both Project A and Project B can access the AI Search index through the shared Hub-level connection. If the connection was defined at the Project A level, then Project B would NOT have access.

**Q19a:**  
Private Endpoints ensure that all network traffic between the App VM, the AI Foundry Hub, and the Storage Account travels over Microsoft's private backbone network — never traversing the public internet. This eliminates data exfiltration risk, prevents exposure to internet-based attacks, and ensures that sensitive data (model inputs/outputs, stored documents) remains within the virtual network boundary.

**Q19b:**  
**Azure Private DNS Zone** is required. When using Private Endpoints, the default public DNS names (e.g., `*.blob.core.windows.net`) need to resolve to the Private Endpoint's private IP address instead of the public IP. Azure Private DNS Zones (e.g., `privatelink.blob.core.windows.net`, `privatelink.api.azureml.ms`) handle this resolution automatically within the VNet. Without proper DNS configuration, clients would still try to connect to the public endpoint.

---

## Grading Rubric

| Section | Questions | Points Per Question | Total |
|---------|-----------|-------------------|-------|
| Multiple Choice | 8 | 5 | 40 |
| True/False | 5 | 3 | 15 |
| Short Answer | 3 | 5 | 15 |
| Matching | 5 | 2 | 10 |
| Diagram-Based | 2 (4 parts) | 5 each part | 20 |
| **Total** | **20** | | **100** |

---

*© 2025 Zero to Hero Training Team. Azure AI Foundry Training Course.*
