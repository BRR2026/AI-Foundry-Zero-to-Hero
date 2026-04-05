# Module 38 — Assessment Quiz

## Building Repeatable AI Offerings for Customers

**Azure AI Foundry: Zero to Hero** | ARC 8: Advanced Patterns & Enterprise | April 2026

**Instructions:** Choose the single best answer for each question. Passing score: 80% (8 of 10).

---

### Question 1

**What is the primary advantage of productizing an AI solution rather than building custom implementations for each customer?**

A) Productized solutions always use fewer Azure resources
B) It eliminates the need for customer-specific data entirely
C) It enables repeatable deployments with parameterized configuration, reducing time-to-value and increasing margin
D) Productized solutions do not require any testing

**Correct Answer:** C

**Explanation:** Productization transforms one-off projects into repeatable, parameterized offerings that can be deployed for multiple customers quickly. This reduces delivery time, increases margins through reuse, and provides a consistent quality baseline. Customer-specific data is still used — it is just parameterized rather than hard-coded.

---

### Question 2

**Which packaging approach is MOST appropriate for an ISV distributing a self-contained AI agent through Azure Marketplace?**

A) ARM Template Spec stored in a customer's subscription
B) A shared Jupyter notebook with deployment instructions
C) A Dockerized container image with `agent.yaml` published to Azure Container Registry
D) A PDF document describing the architecture

**Correct Answer:** C

**Explanation:** Container images with an `agent.yaml` definition are the standard packaging format for distributing AI agents through Azure Marketplace and ACR. They are self-contained, versioned, and can be deployed programmatically. ARM Template Specs are stored per-subscription and are better suited for enterprise IT teams, not marketplace distribution.

---

### Question 3

**In a white-label AI solution, which of the following should be configurable per customer?**

A) The underlying Azure region only
B) Branding (logo, colors, greeting), knowledge source, system prompt, and guardrails
C) The Azure AI Foundry SDK version
D) The Python runtime version used by the agent

**Correct Answer:** B

**Explanation:** White-label solutions allow customers to present the AI offering under their own brand. Key configurable elements include visual branding (logo, colors, greeting messages), knowledge sources (so each customer's agent answers from its own data), system prompts (to customize the agent's persona), and guardrails (to control what topics the agent addresses).

---

### Question 4

**Which multi-tenant isolation strategy provides the best balance of cost efficiency and data separation for most AI offerings?**

A) Shared everything — all tenants use the same resources, same data
B) Shared infrastructure with separate data — one AI Foundry hub, separate search indexes per tenant
C) Fully separate Azure subscriptions per tenant
D) Running all tenants on a single developer workstation

**Correct Answer:** B

**Explanation:** The "shared infrastructure, separate data" pattern is the recommended default for multi-tenant AI offerings. It uses a single AI Foundry hub with per-tenant projects and separate Azure AI Search indexes to achieve data isolation while sharing compute resources. Fully separate subscriptions provide maximum isolation but at significantly higher cost and management complexity.

---

### Question 5

**What distinguishes a solution accelerator from a simple demo or sample application?**

A) Accelerators are always free; demos require payment
B) Accelerators include production-grade infrastructure, security hardening, monitoring, evaluation pipelines, and documentation — not just proof-of-concept code
C) Accelerators do not include any source code
D) Demos are written in Python; accelerators are written in C#

**Correct Answer:** B

**Explanation:** A solution accelerator goes far beyond a demo. It provides production-ready infrastructure code (Bicep/Terraform), security and networking configuration, monitoring and logging setup, evaluation pipelines with baseline metrics, and comprehensive documentation. It is designed to reduce time-to-production from months to days.

---

### Question 6

**When building a Bicep template for a multi-tenant FAQ bot, which parameter is MOST critical for ensuring tenant isolation?**

A) `location` — the Azure region
B) `tenantId` — used to namespace all resource names, indexes, and containers
C) `searchSku` — the AI Search pricing tier
D) `tpmQuota` — the tokens-per-minute rate limit

**Correct Answer:** B

**Explanation:** The `tenantId` parameter is the foundation of tenant isolation. It is used to namespace all resource names (storage accounts, search services, AI Foundry projects) and data artifacts (blob containers, search indexes). This ensures that each tenant's resources and data are logically separated, even when sharing the same Azure subscription.

---

### Question 7

**Which pricing model is MOST suitable for a customer-facing FAQ bot that handles variable volumes of conversations?**

A) Per-seat licensing with a fixed annual fee
B) Per-conversation pricing with tiered volume discounts
C) One-time perpetual license fee
D) Free with no monetization

**Correct Answer:** B

**Explanation:** Per-conversation pricing aligns costs with usage, which is ideal for FAQ bots that may handle wildly different volumes across customers. Tiered volume discounts incentivize higher usage while protecting margins. Per-seat licensing does not map well to bot interactions since usage varies regardless of user count.

---

### Question 8

**When calculating the price for an AI offering, which cost components should be included?**

A) Only the Azure AI model inference cost (tokens consumed)
B) Model inference, Azure AI Search queries, storage, networking, monitoring, and your team's support effort
C) Only the salary of the developer who built the solution
D) Only the Azure subscription fee

**Correct Answer:** B

**Explanation:** A comprehensive cost model for AI offerings must account for all variable and fixed costs: model inference (tokens), search queries, storage for knowledge bases, networking (private endpoints, VNet), monitoring (Application Insights), and the human support effort your team provides per customer. Omitting any of these leads to margin erosion.

---

### Question 9

**In the Mini Hack lab, what validates that the FAQ bot template correctly isolates tenants?**

A) Checking that both tenants share the same search index name
B) Verifying that the rendered `agent.yaml` contains the tenant-specific search index name, and that `diff` between deployments shows only customer-specific values
C) Confirming that both tenants use identical system prompts
D) Checking that the Bicep template has no parameters

**Correct Answer:** B

**Explanation:** Tenant isolation is validated by confirming that each rendered `agent.yaml` references a tenant-specific search index (e.g., `contoso-faq-index` vs `fabrikam-faq-index`). Running `diff` between the two deployment directories verifies that only customer-specific values differ (tenant ID, display name, model, index) while the template structure remains consistent.

---

### Question 10

**A partner wants to license their AI FAQ solution to customers while ensuring customer data stays private. Which approach is correct?**

A) Store all customer data in the partner's central Azure subscription for easier management
B) Co-mingle customer data in a shared search index with row-level security
C) Deploy per-tenant Azure AI Search indexes within the customer's own tenant or a tenant-isolated architecture, and clearly define data ownership in the licensing agreement
D) Require customers to email their FAQ documents for manual upload

**Correct Answer:** C

**Explanation:** Data privacy requires that customer data remains isolated — ideally within the customer's own tenant or in a strictly isolated multi-tenant architecture. The licensing agreement must clearly state that the customer owns their data (documents, queries, fine-tuned models). Co-mingling data in a shared index creates privacy and compliance risks, even with row-level security.

---

## Scoring

| Score | Result |
|-------|--------|
| 10/10 | ⭐ Excellent — Ready for Module 39 |
| 8–9/10 | ✅ Pass — Review missed topics |
| 6–7/10 | ⚠️ Partial — Re-study Sections 3–6 before proceeding |
| < 6/10 | ❌ Retry — Review all sections and repeat the lab |

---

*Module 38 of 45 — Azure AI Foundry: Zero to Hero*
*ARC 8: Advanced Patterns & Enterprise*
*© 2026 Azure AI Foundry Training Team*
