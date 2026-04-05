# Module 39 — Training Guide
## AI Foundry in CSP & Managed Services
### ARC 8: Advanced Patterns & Enterprise | April 2026

---

## 📋 Module Overview

| Field | Details |
|---|---|
| **Module** | 39 of 45 |
| **Title** | AI Foundry in CSP & Managed Services |
| **Arc** | 8 — Advanced Patterns & Enterprise |
| **Duration** | 4 hours (lecture + hands-on) |
| **Level** | Advanced |
| **Prerequisites** | Modules 1–10 (Foundry fundamentals), Module 36 (Enterprise governance), basic Azure networking |

## 🎯 Learning Objectives

By the end of this module, participants will be able to:

1. **Explain** the Azure CSP program model and how it applies to AI Foundry resource provisioning
2. **Design** multi-tenant AI architectures that balance isolation, cost, and operational efficiency
3. **Implement** Azure Lighthouse delegations for cross-tenant AI Foundry management
4. **Configure** governance policies that enforce security and compliance across managed customer subscriptions
5. **Define** tiered SLA frameworks for managed AI services with measurable KPIs
6. **Build** cost allocation and chargeback solutions using tags, metrics, and token metering
7. **Architect** a complete CSP-managed AI Foundry offering from onboarding to ongoing operations

## 🗓️ Agenda

### Session 1: CSP Fundamentals & AI Foundry Integration (60 min)

| Time | Topic | Format |
|------|-------|--------|
| 0:00 – 0:15 | CSP program overview: Direct vs. Indirect, Partner Center, AOBO | Lecture |
| 0:15 – 0:30 | Provisioning AI Foundry resources in customer subscriptions | Demo |
| 0:30 – 0:45 | Automated customer onboarding with Bicep/Terraform | Code walkthrough |
| 0:45 – 1:00 | Q&A and discussion: CSP billing for AI workloads | Discussion |

### Session 2: Multi-Tenant Management & Governance (60 min)

| Time | Topic | Format |
|------|-------|--------|
| 0:00 – 0:15 | Deployment topology options: Dedicated vs. Shared vs. Hybrid | Lecture |
| 0:15 – 0:30 | Azure Lighthouse deep dive: Delegations and eligible authorizations | Demo |
| 0:30 – 0:45 | Azure Policy for AI Foundry: Tag enforcement, network rules, regional constraints | Hands-on |
| 0:45 – 1:00 | Cross-tenant monitoring with Azure Resource Graph | Lab exercise |

### Session 3: SLA Management & Security (60 min)

| Time | Topic | Format |
|------|-------|--------|
| 0:00 – 0:20 | Designing tiered SLAs: Availability, throughput, response times | Lecture |
| 0:20 – 0:35 | SLA monitoring dashboards with KQL and Azure Workbooks | Demo |
| 0:35 – 0:50 | Security baseline: Network isolation, CMK, Content Safety | Hands-on |
| 0:50 – 1:00 | Compliance mapping: SOC 2, HIPAA, GDPR, EU AI Act | Discussion |

### Session 4: Mini Hack — Multi-Tenant CSP Architecture (60 min)

| Time | Topic | Format |
|------|-------|--------|
| 0:00 – 0:10 | Scenario briefing and team formation | Briefing |
| 0:10 – 0:50 | Architecture design challenge (see Mini Hack below) | Team exercise |
| 0:50 – 1:00 | Team presentations and peer review | Presentations |

---

## 📖 Core Concepts

### 1. Cloud Solution Provider (CSP) Model

The Azure CSP program enables partners to own the complete customer relationship for Azure services:

- **Tier 1 (Direct):** Partner transacts directly with Microsoft, sets pricing, provides L1/L2 support
- **Tier 2 (Indirect):** Partner works through an indirect provider for billing; manages customer relationship
- **Admin on Behalf Of (AOBO):** Default CSP mechanism granting Owner access on customer subscriptions
- **Partner Center:** Portal and API for managing CSP customers, subscriptions, and billing

**Key Point:** AI Foundry resources (hubs, projects, model deployments, agents) are standard Azure resources that can be provisioned and managed through the CSP model.

### 2. Multi-Tenant Deployment Topologies

| Topology | Description | Use Case |
|----------|-------------|----------|
| **Dedicated Hub per Customer** | Each customer gets their own AI Foundry hub in their own subscription/tenant | Regulated industries, large enterprise, strict data isolation |
| **Shared Hub, Separate Projects** | Single hub with per-customer projects; RBAC enforces boundaries | SMB customers, standardized AI solutions, cost-sensitive |
| **Hybrid** | Dedicated for premium customers, shared for standard | Mixed customer portfolio |

### 3. Azure Lighthouse for Delegated Management

Azure Lighthouse provides a superior alternative to AOBO for managing customer AI Foundry resources:

- **Granular RBAC:** Assign specific roles at specific scopes (not just Owner on the subscription)
- **Eligible Authorizations:** Just-in-time elevation for privileged operations
- **Customer Transparency:** Customers see exactly what access is delegated and can revoke it
- **Audit Trail:** All partner actions logged in customer's activity log

**Recommended Roles for AI Foundry Management:**
- `Contributor` — Create/manage AI Foundry hubs and projects
- `Cognitive Services Contributor` — Manage AI Services and model deployments
- `Reader` — Monitoring and health checks
- `Monitoring Contributor` — Configure alerts and diagnostics

### 4. SLA Tier Framework

Design your SLA tiers around these dimensions:

| Dimension | Description |
|-----------|-------------|
| **Availability** | Uptime percentage commitment (99.5% – 99.99%) |
| **Throughput** | Tokens per minute (TPM) guarantee |
| **Latency** | P99 response time commitment |
| **Incident Response** | Time to first response by severity |
| **Recovery** | RPO/RTO commitments |
| **Support** | Named engineer, review cadence, optimization services |

### 5. Cost Management & Chargeback

AI Foundry costs consist of:

- **Model Inference:** Billed per token (input/output) or provisioned throughput units (PTU)
- **Compute:** Agent hosting, evaluation pipelines
- **Storage:** Training data, conversation logs, model artifacts
- **Networking:** Private endpoints, VNet peering, data transfer

**Chargeback approaches:**
1. **Subscription-level:** Cleanest separation; one subscription per customer
2. **Tag-based:** Enforce `csp:customer` tags; use Cost Management filters
3. **Token metering:** Custom instrumentation tracking per-project/per-agent token consumption
4. **Hybrid:** Subscription for infrastructure costs + token metering for inference

### 6. Cross-Tenant Security Baseline

Every CSP-managed AI Foundry deployment should enforce:

1. ☑ Private endpoints on all AI Foundry hubs (no public network access)
2. ☑ Managed identities for all service-to-service authentication
3. ☑ Diagnostic settings forwarding to CSP's central Log Analytics workspace
4. ☑ Azure AI Content Safety enabled on all model deployments
5. ☑ Azure Policy initiatives enforcing CIS benchmarks
6. ☑ Microsoft Defender for Cloud enabled with Defender for AI
7. ☑ Customer-managed keys for Gold/Platinum tier customers
8. ☑ Geo-redundant deployments for Gold/Platinum tiers

---

## 🎥 Video Resource

**Microsoft Foundry — AI App & Agent Factory** (MS Mechanics)

- **URL:** https://www.youtube.com/watch?v=C6rxEGJay70
- **Focus Areas for CSP Context:**
  - How Foundry unifies the AI application lifecycle
  - Model catalog and deployment options (relevant for capacity planning)
  - Agent orchestration capabilities (what you'll package as a managed service)
  - Integration patterns (how customer workloads connect to Foundry)

---

## 🛠️ Mini Hack: Multi-Tenant CSP Architecture Design

### Scenario

You are a CSP partner launching "AI-as-a-Service." Your initial customer portfolio includes:

| Segment | Customer | Industry | Key Requirements |
|---------|----------|----------|-----------------|
| A | MedCare Health | Healthcare | HIPAA compliance, PHI isolation, dedicated capacity |
| B | RetailCo (5 stores) | Retail | AI customer service bots, cost-optimized, standard support |
| C | FinanceFirst | Financial Services | SOC 2, dedicated PTU, named support engineer |

### Challenge

Design a complete architecture covering:

1. **Resource Topology:** Deployment pattern per segment with justification
2. **Governance Model:** RBAC strategy, Azure Policy, Lighthouse delegations
3. **SLA Mapping:** Tier assignment per segment with specific commitments
4. **Cost Model:** Billing approach with margin calculation
5. **Security Controls:** Per-segment isolation and compliance controls
6. **Operations:** Monitoring, alerting, and incident response

### Deliverables

- [ ] Architecture diagram showing all three segments
- [ ] RBAC matrix mapping roles → resources → customers
- [ ] Bicep template for one segment's onboarding
- [ ] SLA dashboard KQL query set
- [ ] Executive summary (one page)

### Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Completeness | 25% | All six architecture areas addressed |
| Isolation correctness | 20% | Appropriate isolation levels per segment |
| Cost efficiency | 15% | Realistic cost model with margin preservation |
| Security posture | 20% | Controls match compliance requirements |
| Operational readiness | 10% | Runbooks and monitoring are actionable |
| Presentation quality | 10% | Clear communication to business stakeholders |

---

## ✅ Trainer Checklist

### Before the Session
- [ ] Verify Azure Lighthouse demo environment with two tenants
- [ ] Prepare Partner Center sandbox (or screenshots) for CSP provisioning demo
- [ ] Deploy sample AI Foundry hub with tagged resources for cost management demo
- [ ] Create Azure Workbook template with SLA monitoring queries
- [ ] Prepare architecture diagram templates for Mini Hack teams
- [ ] Print or share the compliance matrix handout

### During the Session
- [ ] Emphasize Azure AI Foundry (not Azure ML) throughout
- [ ] Demo Lighthouse delegation creation and cross-tenant resource view
- [ ] Show Azure Resource Graph queries running across multiple subscriptions
- [ ] Walk through the Bicep onboarding template step by step
- [ ] Facilitate Mini Hack by circulating among teams

### After the Session
- [ ] Collect Mini Hack deliverables for review
- [ ] Share reference architecture diagrams
- [ ] Distribute links to further reading and documentation
- [ ] Gather feedback on module difficulty and pacing

---

## 📚 Additional Resources

| Resource | URL |
|----------|-----|
| Azure AI Foundry Documentation | https://learn.microsoft.com/azure/ai-studio/ |
| CSP Program Overview | https://learn.microsoft.com/partner-center/csp-overview |
| Azure Lighthouse | https://learn.microsoft.com/azure/lighthouse/ |
| Azure Resource Graph | https://learn.microsoft.com/azure/governance/resource-graph/ |
| Azure Policy Built-in Definitions | https://learn.microsoft.com/azure/governance/policy/samples/built-in-policies |
| Azure Cost Management | https://learn.microsoft.com/azure/cost-management-billing/ |

---

*Module 39 of 45 · Azure AI Foundry: Zero to Hero · ARC 8: Advanced Patterns & Enterprise*
