# Module 39 — Assessment
## AI Foundry in CSP & Managed Services
### ARC 8: Advanced Patterns & Enterprise | April 2026

---

## 📝 Quiz Instructions

- **Total Questions:** 10 multiple-choice
- **Passing Score:** 80% (8 of 10 correct)
- **Time Limit:** 15 minutes
- **Topic Coverage:** CSP integration, multi-tenant management, cross-tenant governance, SLA management

> **Note:** All questions pertain to **Azure AI Foundry** — the unified platform for building and managing AI agents and applications.

---

### Question 1
**Which Azure mechanism is recommended over Admin on Behalf Of (AOBO) for CSP partners managing customer AI Foundry resources?**

- A) Azure Active Directory B2B Guest Access
- B) Azure Lighthouse
- C) Service Principal with certificate authentication
- D) Shared Access Signatures (SAS)

<details>
<summary>Show Answer</summary>

**✅ B) Azure Lighthouse**

Azure Lighthouse provides granular, auditable, delegated access to customer resources. Unlike AOBO which grants Owner access broadly, Lighthouse allows assigning specific RBAC roles at specific scopes, supports Just-in-Time elevation, and all actions are logged in the customer's activity log. Customers can see and revoke delegations at any time.

</details>

---

### Question 2
**A CSP partner needs to manage AI Foundry hubs across 50 customer tenants. Which Azure service provides a unified view of resources across all these tenants?**

- A) Azure Portal Dashboard
- B) Azure Resource Graph
- C) Azure Cost Management
- D) Azure Service Health

<details>
<summary>Show Answer</summary>

**✅ B) Azure Resource Graph**

Azure Resource Graph enables querying resources across multiple subscriptions and tenants (when combined with Azure Lighthouse delegations). It provides fast, KQL-based queries that can enumerate all AI Foundry hubs, projects, and deployments across managed customer subscriptions in a single query.

</details>

---

### Question 3
**Which deployment topology should a CSP partner recommend for a healthcare customer requiring HIPAA compliance with PHI data?**

- A) Fully shared hub with logical isolation via RBAC
- B) Shared hub with separate projects per customer
- C) Dedicated AI Foundry hub per customer with full network isolation
- D) Single shared project with data classification tags

<details>
<summary>Show Answer</summary>

**✅ C) Dedicated AI Foundry hub per customer with full network isolation**

HIPAA compliance with PHI data requires the highest level of isolation. A dedicated AI Foundry hub ensures separate network boundaries (private endpoints in the customer's VNet), dedicated managed identities, customer-managed encryption keys, and no shared infrastructure components. Shared models or projects cannot guarantee the level of data isolation required for PHI.

</details>

---

### Question 4
**A CSP partner wants to enforce that all AI Foundry workspaces in managed customer subscriptions include `csp:customer` and `csp:tier` tags. What is the most effective approach?**

- A) Document the requirement in the partner agreement
- B) Create an Azure Policy with a deny effect for resources missing required tags
- C) Use a naming convention that embeds customer and tier information
- D) Schedule a weekly Azure CLI script to audit and tag resources

<details>
<summary>Show Answer</summary>

**✅ B) Create an Azure Policy with a deny effect for resources missing required tags**

Azure Policy with a deny effect prevents non-compliant resources from being created in the first place, enforcing the requirement proactively rather than reactively. This is the only option that provides guaranteed enforcement. Documentation alone doesn't prevent non-compliance, naming conventions can be bypassed, and scripted auditing only detects issues after the fact.

</details>

---

### Question 5
**In a tiered SLA model for CSP-managed AI services, which tier characteristic best differentiates Gold from Silver?**

- A) Gold includes Azure support, Silver does not
- B) Gold offers 99.95% availability with dedicated PTU and a shared support engineer
- C) Gold uses Standard model SKUs while Silver uses Basic
- D) Gold includes Defender for Cloud, Silver relies on basic monitoring

<details>
<summary>Show Answer</summary>

**✅ B) Gold offers 99.95% availability with dedicated PTU and a shared support engineer**

The key differentiators between Gold and Silver tiers are: higher availability commitment (99.95% vs. 99.9%), dedicated Provisioned Throughput Units (PTU) instead of standard, faster incident response times (30 min vs. 1 hour for Sev-1), and access to a shared support engineer. These create tangible value differences that justify the tier pricing.

</details>

---

### Question 6
**A CSP partner uses the shared hub / separate projects topology. What is the primary RBAC mechanism to prevent Customer A from accessing Customer B's data?**

- A) Network Security Groups between customer VNets
- B) Scoping RBAC role assignments to the specific project resource
- C) Using separate Azure AD tenants for each customer
- D) Encrypting each customer's data with different keys

<details>
<summary>Show Answer</summary>

**✅ B) Scoping RBAC role assignments to the specific project resource**

In a shared hub topology with separate projects, RBAC is the primary isolation mechanism. By assigning Customer A's users/groups roles scoped only to Project A, and Customer B's users/groups only to Project B, you prevent cross-customer data access at the authorization layer. While other options (network isolation, encryption) provide defense-in-depth, RBAC scoping at the project level is the primary control.

</details>

---

### Question 7
**Which approach provides the most accurate cost chargeback for AI Foundry inference usage across multiple CSP customers sharing infrastructure?**

- A) Splitting total subscription cost equally among customers
- B) Tag-based cost allocation using Azure Cost Management filters
- C) Custom token metering that tracks per-project/per-agent token consumption
- D) Estimating costs based on the number of deployed agents per customer

<details>
<summary>Show Answer</summary>

**✅ C) Custom token metering that tracks per-project/per-agent token consumption**

When customers share infrastructure (shared hub), subscription-level and tag-based allocation cannot accurately attribute inference costs to individual customers. Custom token metering — tracking prompt and completion tokens per project or per agent — provides the most accurate chargeback for the variable-cost component of AI Foundry. This can be combined with tag-based allocation for fixed infrastructure costs (a hybrid approach).

</details>

---

### Question 8
**A CSP partner needs to ensure that AI Foundry resources for a European customer stay within EU regions. Which Azure capability enforces this requirement?**

- A) Azure Front Door geographic routing
- B) Azure Policy with allowed locations restriction
- C) Azure Traffic Manager with geographic routing
- D) AI Foundry hub configuration setting for region locking

<details>
<summary>Show Answer</summary>

**✅ B) Azure Policy with allowed locations restriction**

Azure Policy with an allowed locations restriction (using the built-in policy definition) prevents any resources from being created outside specified Azure regions. This is enforced at the ARM layer, meaning no one — including the CSP admin — can provision AI Foundry resources in non-compliant regions. This is the correct approach for data residency requirements under GDPR and other regulations.

</details>

---

### Question 9
**Which of the following represents the correct compliance responsibility model for a CSP-managed AI Foundry deployment?**

- A) Microsoft is solely responsible for all compliance requirements
- B) The CSP partner is solely responsible since they manage the customer relationship
- C) Compliance is shared: Microsoft provides the platform certifications, the CSP partner documents controls and access reviews, and the customer owns their data classification
- D) The end customer bears full responsibility as the data owner

<details>
<summary>Show Answer</summary>

**✅ C) Compliance is shared: Microsoft provides the platform certifications, the CSP partner documents controls and access reviews, and the customer owns their data classification**

In the CSP model, compliance follows a shared responsibility model. Microsoft certifies the Azure platform (infrastructure, physical security, base services). The CSP partner is responsible for implementing and documenting operational controls, access management, incident response, and AI-specific governance. The customer retains responsibility for their data, data classification, and ensuring their use of AI complies with relevant regulations.

</details>

---

### Question 10
**A CSP partner discovers that a customer's AI Foundry agent is producing potentially harmful content. Under which security control should this be addressed?**

- A) Azure Policy content type restrictions
- B) Azure AI Content Safety filters configured on the model deployment
- C) Network Security Group rule to block outbound traffic
- D) Azure Key Vault access policy revocation

<details>
<summary>Show Answer</summary>

**✅ B) Azure AI Content Safety filters configured on the model deployment**

Azure AI Content Safety provides built-in content filtering for AI model deployments, detecting and blocking harmful content categories (hate, violence, self-harm, sexual content). CSP partners should ensure Content Safety is enabled on all customer model deployments as part of their security baseline. This is the appropriate control for content safety issues — network controls and key vault policies address different security domains.

</details>

---

## 📊 Scoring Guide

| Score | Rating | Recommendation |
|-------|--------|----------------|
| 10/10 | ⭐ Excellent | Ready for advanced CSP AI architecture work |
| 8-9/10 | ✅ Passing | Strong understanding; review missed areas |
| 6-7/10 | ⚠️ Needs Review | Revisit the module content, especially governance and SLA sections |
| Below 6 | ❌ Not Passing | Re-study the module and complete the hands-on lab before retaking |

---

## 🔑 Key Concepts Tested

| Question | Concept |
|----------|---------|
| Q1 | Azure Lighthouse vs. AOBO for delegated management |
| Q2 | Azure Resource Graph for cross-tenant visibility |
| Q3 | Deployment topology selection based on compliance |
| Q4 | Azure Policy for governance enforcement |
| Q5 | SLA tier differentiation for managed AI services |
| Q6 | RBAC scoping in shared infrastructure topology |
| Q7 | Cost chargeback accuracy with token metering |
| Q8 | Data residency enforcement with Azure Policy |
| Q9 | Shared responsibility model in CSP compliance |
| Q10 | Azure AI Content Safety for content governance |

---

*Module 39 of 45 · Azure AI Foundry: Zero to Hero · ARC 8: Advanced Patterns & Enterprise*
