# Module 26: Deploying Model Endpoints

> **Azure AI Foundry: Zero to Hero** · Arc 6: Deployment & MLOps · April 2026

---

## Module Overview

| Field | Details |
|---|---|
| **Title** | Deploying Model Endpoints |
| **Arc** | 6 — Deployment & MLOps |
| **Duration** | 3.5 hours (lecture + lab) |
| **Level** | Intermediate to Advanced |
| **Prerequisites** | Module 25 (CI/CD for AI), Azure AI Foundry project, Python 3.10+ |

## Learning Objectives

By the end of this module, learners will be able to:

1. **Describe** the three deployment types in Azure AI Foundry (managed compute, serverless API, provisioned throughput) and when to use each.
2. **Configure** endpoint settings including SKU, capacity, scaling, version policy, and content filters.
3. **Implement** blue/green deployments with traffic splitting for safe model rollouts.
4. **Manage** API keys with zero-downtime rotation and explain the advantages of Microsoft Entra ID authentication.
5. **Execute** a complete deployment lifecycle: create → validate → split traffic → monitor → cutover/rollback.

---

## Agenda

| Time | Topic | Activity |
|------|-------|----------|
| 0:00–0:15 | Welcome & Context | Where we are in Arc 6 |
| 0:15–0:50 | Deployment Types Deep Dive | Lecture + demo |
| 0:50–1:15 | Endpoint Configuration & Scaling | Lecture + code walkthrough |
| 1:15–1:30 | ☕ Break | — |
| 1:30–2:05 | Blue/Green Deployments & Traffic Splitting | Lecture + live demo |
| 2:05–2:30 | API Key Management & Authentication | Lecture + security best practices |
| 2:30–2:40 | 🎬 Video: How to create AI model deployment | Watch & discuss |
| 2:40–3:20 | 🧪 Mini Hack: Blue/Green Traffic Splitting | Hands-on lab |
| 3:20–3:30 | Wrap-Up & Preview of Module 27 | Q&A |

---

## Section 1: Deployment Types (35 min)

### Key Concepts

Azure AI Foundry provides three deployment paradigms:

#### 1.1 Managed Compute
- **What:** Dedicated VM-based infrastructure (GPU or CPU SKUs).
- **When:** Fine-tuned models, custom containers, models not in the catalog.
- **Billing:** Per-VM-hour (you pay for allocated instances).
- **Scaling:** Manual instance count or autoscaling (min/max instances, target utilization).

#### 1.2 Serverless API (Pay-Per-Token)
- **What:** Fully managed, no-infrastructure deployment for Model Catalog models.
- **When:** Rapid prototyping, variable/unpredictable load, no custom model requirements.
- **Billing:** Per-token (input and output tokens billed separately).
- **Scaling:** Automatic and transparent; you only configure rate limits (tokens per minute).

#### 1.3 Provisioned Throughput (PTU)
- **What:** Reserved capacity measured in Provisioned Throughput Units.
- **When:** Production SLA workloads requiring guaranteed, predictable latency.
- **Billing:** Per-PTU-hour (reserved commitment, often with volume discounts).
- **Scaling:** Fixed PTU allocation; use the Capacity Calculator for sizing.

### Discussion Questions

1. You have a fine-tuned Llama model. Which deployment type would you choose?
2. Your team is prototyping a chatbot. What deployment type minimizes upfront cost?
3. A financial services company needs guaranteed P99 latency < 2s. What do you recommend?

---

## Section 2: Endpoint Configuration & Scaling (25 min)

### Configuration Parameters

| Parameter | Description | Example Values |
|-----------|-------------|----------------|
| SKU Name | Deployment tier | `GlobalStandard`, `Standard`, `ProvisionedManaged` |
| SKU Capacity | Rate limit (TPM) or PTUs | `50` (50K TPM), `100` (100 PTUs) |
| Version Policy | Model upgrade behavior | `NoAutoUpgrade`, `AutoUpgrade` |
| Content Filter | Safety policy | `DefaultV2`, custom policies |
| Network | Access controls | Public, Private Endpoint, VNet |

### Scaling Deep Dive

**Serverless API:**
- Adjust `sku_capacity` to change rate limits.
- No cold starts for catalog models.

**Managed Compute:**
- Configure `AutoScaleSettings` with `min_instances`, `max_instances`, `target_utilization`.
- Scale-to-zero is possible but introduces cold start latency.
- Choose instance types based on model size (e.g., `Standard_NC24ads_A100_v4` for large LLMs).

**Provisioned Throughput:**
- Use the Azure AI Foundry Portal **Capacity Calculator** to determine required PTUs.
- Approximate sizing: ~10 PTUs ≈ 6 req/min for GPT-4o at typical prompt length.

### Trainer Demo

Walk through endpoint configuration in the Azure AI Foundry Portal:
1. Navigate to project → Deployments → + Deploy Model.
2. Select a model from the catalog.
3. Choose deployment type and configure SKU.
4. Show the monitoring dashboard after deployment succeeds.

---

## Section 3: Blue/Green Deployments & Traffic Splitting (35 min)

### Concept Overview

Blue/green deployment enables zero-downtime model upgrades:

```
Client Request → Endpoint (single URL)
                    ├── Blue Deployment (v1) ← 90% traffic
                    └── Green Deployment (v2) ← 10% traffic
```

### Traffic Splitting Phases

| Phase | Blue | Green | Duration | Validation |
|-------|------|-------|----------|------------|
| 1. Canary | 90% | 10% | 30 min | Error rates, latency |
| 2. Partial | 50% | 50% | 1 hour | A/B quality comparison |
| 3. Majority | 10% | 90% | 30 min | Final confirmation |
| 4. Cutover | 0% | 100% | — | Decommission blue |

### Key Rules

- Traffic percentages **must sum to 100**.
- Changes propagate within **seconds**.
- In-flight requests complete on their original deployment.
- Rollback is instant: set blue to 100%.

### Code Walkthrough

Demonstrate the full lifecycle using the Azure AI Foundry SDK:
1. Create blue deployment.
2. Create green deployment.
3. Configure 90/10 split.
4. Validate with test requests.
5. Shift to 50/50, then 100% green.
6. Perform rollback.

---

## Section 4: API Key Management & Authentication (25 min)

### Authentication Methods

| Method | Pros | Cons |
|--------|------|------|
| **API Key** | Simple, universal client support | Secret management required, rotation overhead |
| **Microsoft Entra ID** | No secrets, RBAC, audit trail | Requires identity infrastructure, token refresh |

### Key Rotation Strategy

1. Each deployment has a **primary** and **secondary** key.
2. Clients use the primary key.
3. Regenerate primary → update clients → regenerate secondary.
4. Store keys in **Azure Key Vault**, never in code.

### Security Best Practices

- ✅ Prefer Microsoft Entra ID (managed identity) for production.
- ✅ Store API keys in Azure Key Vault.
- ✅ Automate key rotation (e.g., Azure Functions on a 90-day schedule).
- ✅ Enable private endpoints for network isolation.
- ❌ Never commit keys to source control.
- ❌ Never share keys across environments (dev/staging/prod).

---

## Video Resource

📺 **How to create AI model deployment | Microsoft Foundry**
- YouTube: [https://www.youtube.com/watch?v=LLO0lOufbwc](https://www.youtube.com/watch?v=LLO0lOufbwc)
- Duration: ~15 minutes
- Use as supplementary viewing or in-class demo.

---

## Mini Hack: Deploy a Model with Blue/Green Traffic Splitting (40 min)

### Objective
Deploy two versions of a model behind a single Azure AI Foundry endpoint and implement phased traffic splitting.

### Success Criteria
- [ ] Blue deployment created and serving requests.
- [ ] Green deployment created with a different model version.
- [ ] 90/10 traffic split configured and validated.
- [ ] Traffic shifted to 50/50, then 100% green.
- [ ] Rollback performed successfully (100% blue).

### Resources
- Full lab instructions: `lab/hands-on-lab.md`
- Assessment: `quiz/assessment.md`

---

## Assessment

- **Quiz:** 10 multiple-choice questions covering deployment types, scaling, traffic splitting, and authentication. See `quiz/assessment.md`.
- **Passing Score:** 80% (8/10)
- **Lab Completion:** All success criteria met.

---

## Supplementary Resources

| Resource | Link |
|----------|------|
| Azure AI Foundry Deployment Docs | [learn.microsoft.com/azure/ai-foundry/deploy](https://learn.microsoft.com/azure/ai-foundry/) |
| Provisioned Throughput Overview | [learn.microsoft.com/azure/ai-services/openai/concepts/provisioned-throughput](https://learn.microsoft.com/azure/ai-services/openai/concepts/provisioned-throughput) |
| Blue/Green Deployment Pattern | [learn.microsoft.com/azure/architecture/patterns](https://learn.microsoft.com/azure/architecture/) |
| Azure Key Vault Best Practices | [learn.microsoft.com/azure/key-vault/general/best-practices](https://learn.microsoft.com/azure/key-vault/general/best-practices) |

---

## Key Takeaways

1. **Choose the right deployment type** for your workload: serverless for flexibility, managed compute for custom models, PTU for guaranteed performance.
2. **Always use blue/green deployments** for model version upgrades — never deploy directly to 100% traffic.
3. **Secure your endpoints** with Microsoft Entra ID in production; use API key rotation with Key Vault as a fallback.
4. **Monitor continuously** — deployment is not the end; it's the beginning of the operational lifecycle.

---

*Module 26 of 45 · Azure AI Foundry: Zero to Hero · Arc 6: Deployment & MLOps*
