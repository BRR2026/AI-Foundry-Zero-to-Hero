# Module 38 — Training Guide

## Building Repeatable AI Offerings for Customers

**Azure AI Foundry: Zero to Hero** | ARC 8: Advanced Patterns & Enterprise | April 2026

---

## Module Overview

| Field               | Details |
|---------------------|---------|
| **Module Number**   | 38 of 45 |
| **Title**           | Building Repeatable AI Offerings for Customers |
| **Arc**             | 8 — Advanced Patterns & Enterprise |
| **Duration**        | ~4 hours (lecture + lab) |
| **Level**           | Advanced |
| **Prerequisites**   | Modules 1–37; experience with Azure AI Foundry agent deployment |

---

## Learning Objectives

By the end of this module, learners will be able to:

1. **Package AI solutions as repeatable products** — Transform one-off projects into configurable, distributable AI offerings using Azure AI Foundry.
2. **Design white-label and multi-tenant architectures** — Implement tenant isolation strategies that allow a single AI solution to serve multiple customer brands.
3. **Build solution templates and accelerators** — Create ARM/Bicep/Terraform templates and AI Foundry project templates that spin up complete AI solutions in minutes.
4. **Define pricing and licensing strategies** — Structure consumption-based, seat-based, and hybrid pricing models for AI products.
5. **Create a customer FAQ bot template** — Build an end-to-end solution template that provisions a knowledge-grounded FAQ agent with configurable branding (Mini Hack).

---

## Outline & Timing

| # | Topic | Duration | Type |
|---|-------|----------|------|
| 1 | From Projects to Products — The Productization Mindset | 30 min | Lecture |
| 2 | Packaging AI Solutions in Azure AI Foundry | 40 min | Lecture + Demo |
| 3 | White-Label Patterns — One Solution, Many Brands | 35 min | Lecture |
| 4 | Multi-Tenant Architecture for AI | 40 min | Lecture + Demo |
| 5 | Solution Templates & Accelerators | 35 min | Lecture + Demo |
| 6 | Pricing & Licensing for AI Offerings | 30 min | Lecture |
| 7 | Mini Hack: Customer FAQ Bot Template | 60 min | Hands-On Lab |
| 8 | Review & Assessment | 20 min | Quiz |

---

## Section 1: From Projects to Products — The Productization Mindset

### Key Concepts

- **One-off vs. repeatable**: The cost of re-building the same solution for each customer vs. parameterizing once and deploying many times.
- **Product thinking for AI**: Treating an AI solution as a product with versioning, SLA, documentation, and a clear value proposition.
- **The productization spectrum**: Custom consulting → configurable template → self-service SaaS.

### Discussion Points

- What makes an AI solution "repeatable"?
- Where does Azure AI Foundry fit in the ISV/partner ecosystem?
- How do Microsoft partners monetize AI Foundry–based solutions today?

### Key Takeaway

> Productizing AI is not just about code reuse — it is about building a **business system** around your technical capability, including onboarding, billing, support, and lifecycle management.

---

## Section 2: Packaging AI Solutions in Azure AI Foundry

### Core Patterns

| Pattern | Description | Best For |
|---------|-------------|----------|
| **Project Template** | Pre-configured Foundry project with agents, knowledge, evaluation | Partners deploying to customer tenants |
| **Bicep/Terraform Module** | IaC that provisions all Azure resources + Foundry assets | Enterprise IT teams |
| **Container Image** | Dockerized agent with `agent.yaml` | ISVs distributing via ACR/Marketplace |
| **ARM Template Spec** | Versioned ARM template stored in Azure | Managed service providers |

### Packaging Checklist

1. Externalize all configuration (endpoints, model names, knowledge sources)
2. Parameterize tenant-specific values (branding, domain, data sources)
3. Include evaluation datasets and baseline metrics
4. Provide `README.md` + onboarding guide
5. Add deployment validation tests

### Azure AI Foundry Project Template Structure

```
my-faq-offering/
├── agent.yaml                  # Agent definition
├── infra/
│   ├── main.bicep              # Infrastructure as Code
│   └── parameters.json         # Default parameters
├── src/
│   ├── prompts/
│   │   └── system-prompt.md    # Configurable system prompt
│   └── functions/              # Custom tool functions
├── data/
│   ├── sample-knowledge.jsonl  # Sample knowledge base
│   └── eval-dataset.jsonl      # Evaluation dataset
├── tests/
│   └── smoke-test.py           # Post-deployment validation
├── docs/
│   └── onboarding.md           # Customer onboarding guide
└── deploy.sh                   # One-click deployment script
```

---

## Section 3: White-Label Patterns — One Solution, Many Brands

### White-Label Architecture

A white-label AI offering allows customers to present the solution under their own brand without knowing the underlying provider.

**Key Design Decisions:**

| Decision | Options |
|----------|---------|
| **Branding** | Customer logo, colors, bot name, greeting messages |
| **Domain** | Custom domain or subdomain per customer |
| **Data isolation** | Separate knowledge indexes per tenant |
| **Model selection** | Shared model deployment vs. dedicated per tenant |
| **Prompt customization** | Customer-specific system prompts and guardrails |

### Implementation Approach

```yaml
# customer-config.yaml
tenant_id: "contoso"
display_name: "Contoso AI Assistant"
branding:
  logo_url: "https://contoso.com/logo.png"
  primary_color: "#0078D4"
  greeting: "Hello! I'm the Contoso virtual assistant."
knowledge:
  index_name: "contoso-knowledge-index"
  data_source: "contoso-blob-container"
model:
  deployment: "gpt-4o"
  temperature: 0.3
guardrails:
  blocked_topics: ["competitor-products", "pricing-negotiation"]
```

---

## Section 4: Multi-Tenant Architecture for AI

### Tenant Isolation Strategies

| Strategy | Isolation Level | Cost | Complexity |
|----------|----------------|------|------------|
| **Shared everything** | Low | Low | Low |
| **Shared infra, separate data** | Medium | Medium | Medium |
| **Separate Foundry projects** | High | High | Medium |
| **Separate subscriptions** | Maximum | Highest | High |

### Recommended Pattern: Shared Infra, Separate Data

- **Single Azure AI Foundry hub** with multiple projects (one per tenant)
- **Shared model deployments** with per-tenant rate limiting
- **Separate Azure AI Search indexes** per tenant for knowledge isolation
- **Tenant routing layer** that directs requests to the correct project/index
- **Centralized monitoring** with tenant-level dashboards

### Multi-Tenant Request Flow

```
Customer Request
       │
       ▼
┌─────────────────┐
│  API Gateway /   │
│  Tenant Router   │
└───────┬─────────┘
        │  (extract tenant ID)
        ▼
┌─────────────────┐
│  Config Lookup   │──→ Tenant config store
└───────┬─────────┘
        │
        ▼
┌─────────────────┐
│  AI Foundry      │
│  Agent Endpoint  │──→ Tenant-specific project
└───────┬─────────┘
        │
        ▼
┌─────────────────┐
│  Knowledge       │──→ Tenant-specific index
│  Retrieval       │
└───────┬─────────┘
        │
        ▼
    Response
```

---

## Section 5: Solution Templates & Accelerators

### What Is a Solution Accelerator?

A solution accelerator is a **production-ready, parameterized starting point** that reduces time-to-value from months to days. Unlike a demo or sample, an accelerator includes:

- Production-grade infrastructure code
- Security hardening and networking
- Monitoring, logging, and alerting
- Evaluation pipelines
- Documentation and onboarding automation

### Building Accelerators with Azure AI Foundry

1. **Start with a working solution** — Build the solution for one customer first.
2. **Extract parameters** — Identify what varies per deployment (model, data, branding, scale).
3. **Create IaC templates** — Wrap all resources in Bicep or Terraform modules.
4. **Add deployment automation** — Scripts or GitHub Actions for one-click provisioning.
5. **Include evaluation** — Embed eval datasets and pass/fail criteria.
6. **Document everything** — Onboarding guide, architecture diagram, troubleshooting guide.

### Template Parameterization Example

```bicep
@description('Customer name for resource naming')
param customerName string

@description('Azure region for deployment')
param location string = resourceGroup().location

@description('AI model to deploy')
@allowed(['gpt-4o', 'gpt-4o-mini', 'gpt-4.1'])
param modelName string = 'gpt-4o'

@description('Knowledge base blob container name')
param knowledgeContainer string = 'knowledge-base'

@description('Tokens-per-minute quota for the model')
param tpmQuota int = 30000

resource aiHub 'Microsoft.MachineLearningServices/workspaces@2024-10-01' = {
  name: '${customerName}-ai-hub'
  location: location
  kind: 'Hub'
  // ... configuration
}
```

---

## Section 6: Pricing & Licensing for AI Offerings

### Pricing Models for AI Products

| Model | How It Works | Best For |
|-------|-------------|----------|
| **Per-conversation** | Charge per completed conversation / session | Customer service bots |
| **Per-query** | Charge per API call or knowledge retrieval | Search / analytics tools |
| **Per-seat / per-user** | Fixed monthly fee per end user | Enterprise copilots |
| **Tiered consumption** | Volume-based pricing tiers | High-volume applications |
| **Flat subscription** | Fixed monthly/annual fee with usage limits | SMB solutions |
| **Outcome-based** | Charge based on measurable business outcome | Premium offerings |

### Cost Structure Awareness

When pricing your AI offering, account for:

- **Azure AI Foundry compute** — Model inference (tokens consumed)
- **Azure AI Search** — Index storage and query volume
- **Azure Storage** — Knowledge base document storage
- **Networking** — Private endpoints, VNet integration
- **Monitoring** — Application Insights, Log Analytics
- **Support** — Your team's support effort per customer

### Margin Calculation Example

```
Per-conversation pricing example:
  Average tokens per conversation:  2,500
  GPT-4o cost per 1K tokens:        ~$0.005 (blended in/out)
  Azure AI Search cost per query:   ~$0.001
  Infrastructure overhead:          ~$0.002
  ─────────────────────────────────
  Total cost per conversation:      ~$0.020
  Your price per conversation:      $0.10
  Gross margin:                     80%
```

### Licensing Considerations

- **Model licensing**: Ensure your Azure AI model usage complies with Microsoft's terms for redistribution.
- **Data licensing**: Customer data stays in the customer's tenant — do not co-mingle.
- **IP ownership**: Clearly define who owns fine-tuned models, evaluation datasets, and custom prompts.
- **Usage reporting**: Provide customers with transparent usage dashboards.

---

## Section 7: Mini Hack — Customer FAQ Bot Template

> **Objective**: Create a reusable, parameterized solution template that deploys a knowledge-grounded FAQ bot for any customer in under 10 minutes.

### Lab Overview

See `lab/hands-on-lab.md` for the full step-by-step guide.

**What you'll build:**
1. A parameterized `agent.yaml` with customer-configurable fields
2. A Bicep template that provisions AI Foundry + AI Search + Storage
3. A deployment script that accepts a customer config file and provisions everything
4. A smoke test that validates the deployed agent responds correctly

### Success Criteria

- [ ] Deploy the FAQ bot template for "Contoso" in < 10 minutes
- [ ] Deploy the same template for "Fabrikam" with different branding and knowledge
- [ ] Both deployments pass the included smoke tests
- [ ] Each tenant's agent only answers from its own knowledge base

---

## Assessment

See `quiz/assessment.md` for the 10-question multiple-choice assessment covering all sections.

---

## Supplementary Resources

### Official Documentation

- [Azure AI Foundry Documentation](https://learn.microsoft.com/azure/ai-studio/)
- [Bicep Templates for AI Workloads](https://learn.microsoft.com/azure/templates/microsoft.machinelearningservices/workspaces)
- [Azure AI Search Multi-Tenancy](https://learn.microsoft.com/azure/search/search-modeling-multitenant-saas-applications)
- [Azure Architecture Center — Multi-Tenant SaaS](https://learn.microsoft.com/azure/architecture/guide/multitenant/overview)

### Related Modules

| Module | Title | Relevance |
|--------|-------|-----------|
| 35 | Enterprise Governance for AI | Governance foundations for productized AI |
| 36 | Cost Optimization & FinOps | Cost management underpinning pricing strategy |
| 37 | Advanced Networking & Security | Network isolation for multi-tenant deployments |
| 39 | AI Marketplace & Partner Ecosystem | Publishing offerings to Azure Marketplace |

### Video Resource

📺 **Microsoft Foundry — AI app & agent factory** (MS Mechanics)
[Watch on YouTube](https://www.youtube.com/watch?v=C6rxEGJay70)

---

## Instructor Notes

- **Audience calibration**: This module targets partners, ISVs, and enterprise architects who want to build repeatable AI businesses. Adjust examples based on audience.
- **Live demo recommendation**: Show a live deployment of the FAQ bot template for two different "customers" to demonstrate the speed and isolation.
- **Common misconception**: Students often conflate multi-tenancy with multi-instance. Emphasize that multi-tenancy is about shared infrastructure with logical isolation.
- **Time management**: The pricing section often generates extensive discussion — keep it focused on framework, not specific numbers.

---

*Module 38 of 45 — Azure AI Foundry: Zero to Hero*
*ARC 8: Advanced Patterns & Enterprise*
*© 2026 Azure AI Foundry Training Team*
