# Module 26 Assessment: Deploying Model Endpoints

> **Azure AI Foundry: Zero to Hero** · Arc 6: Deployment & MLOps  
> **Passing Score:** 80% (8 out of 10)

---

## Instructions

- Select the **one best answer** for each question.
- Each question is worth 1 point (10 points total).
- Review the answer key at the bottom after completing all questions.

---

### Question 1

Which deployment type in Azure AI Foundry uses a pay-per-token billing model with no infrastructure management required?

- A) Managed Compute
- B) Serverless API
- C) Provisioned Throughput (PTU)
- D) Container Instances

---

### Question 2

You need guaranteed, predictable latency for a production chatbot that must meet strict SLA requirements. Which deployment type is most appropriate?

- A) Serverless API with high rate limits
- B) Managed Compute with autoscaling
- C) Provisioned Throughput (PTU)
- D) Serverless API with content filters

---

### Question 3

When configuring a blue/green traffic split in Azure AI Foundry, what must always be true about the traffic percentages?

- A) Blue must always have more traffic than green
- B) Percentages must sum to exactly 100
- C) Each deployment must have at least 5% traffic
- D) Traffic can only be split between exactly two deployments

---

### Question 4

Your team has fine-tuned a custom Llama model and needs to deploy it in Azure AI Foundry. Which deployment type should you choose?

- A) Serverless API — it supports all models
- B) Provisioned Throughput — it provides dedicated resources
- C) Managed Compute — it supports custom containers and fine-tuned models
- D) Any deployment type — all support custom models equally

---

### Question 5

What is the recommended approach for handling model version upgrades in production Azure AI Foundry endpoints?

- A) Deploy the new version directly to 100% traffic for immediate availability
- B) Delete the old deployment and create a new one with the updated version
- C) Use blue/green deployment with gradual traffic shifting (e.g., 10% → 50% → 100%)
- D) Update the model version in-place on the existing deployment

---

### Question 6

Which authentication method is recommended for production Azure AI Foundry endpoints?

- A) API key with the key hardcoded in application configuration files
- B) API key stored in environment variables
- C) Microsoft Entra ID with managed identity
- D) Shared access signatures (SAS tokens)

---

### Question 7

Each Azure AI Foundry deployment provides two API keys (primary and secondary). What is the purpose of having two keys?

- A) One key is for read operations and the other for write operations
- B) One key is for the blue deployment and the other for the green deployment
- C) It enables zero-downtime key rotation — one key stays active while the other is regenerated
- D) The secondary key is a backup that only works when the primary key is disabled

---

### Question 8

You have a managed compute deployment with autoscaling configured. Which parameter determines the threshold at which new instances are added?

- A) `max_instances`
- B) `polling_interval`
- C) `target_utilization`
- D) `sku_capacity`

---

### Question 9

During a blue/green deployment, you notice the green (new) deployment has a 12% error rate compared to 0.5% for blue. What is the correct immediate action?

- A) Increase the green deployment's SKU capacity to handle more load
- B) Delete the green deployment and recreate it
- C) Set traffic to 100% blue and 0% green to instantly roll back
- D) Wait for the error rate to stabilize before taking action

---

### Question 10

Which version policy setting should you use to prevent unexpected model behavior changes in a production deployment?

- A) `AutoUpgrade` — ensures you always have the latest improvements
- B) `NoAutoUpgrade` — keeps the exact model version you specified
- C) `LatestStable` — automatically uses the most stable release
- D) `PinnedVersion` — locks the deployment to a specific checkpoint

---

## Answer Key

| Question | Correct Answer | Explanation |
|----------|---------------|-------------|
| 1 | **B** | Serverless API provides pay-per-token billing with no infrastructure management. Managed Compute uses per-VM-hour billing, and PTU uses per-PTU-hour billing. |
| 2 | **C** | Provisioned Throughput (PTU) provides reserved capacity with guaranteed, predictable latency — ideal for strict SLA requirements. Serverless API has variable latency due to shared infrastructure. |
| 3 | **B** | Traffic percentages across all deployments behind an endpoint must always sum to exactly 100. There's no minimum percentage requirement, and more than two deployments can share traffic. |
| 4 | **C** | Managed Compute is the only deployment type that fully supports custom containers and fine-tuned models. Serverless API is limited to Model Catalog models. |
| 5 | **C** | Blue/green deployment with gradual traffic shifting is the recommended approach. It provides validation at each phase and enables instant rollback if issues are detected. |
| 6 | **C** | Microsoft Entra ID with managed identity eliminates secret management entirely and provides fine-grained RBAC and audit trails. API keys should never be hardcoded. |
| 7 | **C** | Dual keys enable zero-downtime rotation. You regenerate the primary key while clients can temporarily use the secondary key, then update clients and regenerate secondary. |
| 8 | **C** | `target_utilization` (as a percentage) determines when autoscaling adds or removes instances. When utilization exceeds this threshold, new instances are provisioned. |
| 9 | **C** | Instant rollback (set blue to 100%, green to 0%) is the correct immediate action. Traffic changes propagate within seconds, immediately stopping errors for end users. Investigate the green deployment after rollback. |
| 10 | **B** | `NoAutoUpgrade` ensures the deployment uses exactly the model version you specified. This prevents unexpected behavior changes when Microsoft releases new model versions. |

---

## Scoring

| Score | Result |
|-------|--------|
| 10/10 | ⭐ Excellent — you've mastered model endpoint deployment! |
| 8–9/10 | ✅ Pass — solid understanding of deployment concepts |
| 6–7/10 | ⚠️ Review Needed — revisit traffic splitting and authentication sections |
| Below 6/10 | ❌ Retake — review the full module and complete the hands-on lab |

---

*Module 26 Assessment · Azure AI Foundry: Zero to Hero · Arc 6: Deployment & MLOps*
