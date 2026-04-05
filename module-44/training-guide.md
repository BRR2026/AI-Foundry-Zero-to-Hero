# Module 44 — Add Governance, Monitoring & Security

## Training Guide

| Field             | Details                                                    |
|-------------------|------------------------------------------------------------|
| **Course**        | Azure AI Foundry: Zero to Hero (Module 44/45)              |
| **Arc**           | ARC 9 — CAPSTONE & MASTERY                                |
| **Title**         | Add Governance, Monitoring & Security                      |
| **Date**          | April 2026                                                 |
| **Duration**      | 3 hours (lecture + lab)                                    |
| **Prerequisite**  | Module 43 — Prior capstone module                          |

---

## Learning Objectives

By the end of this module, learners will be able to:

1. Implement Role-Based Access Control (RBAC) using built-in and custom roles for Azure AI Foundry resources.
2. Configure system-assigned and user-assigned Managed Identities for credential-free authentication.
3. Build Azure Monitor dashboards with KQL queries for API latency, error rates, and token consumption.
4. Create metric alerts and action groups for proactive incident detection.
5. Configure Azure AI Content Safety filters including custom blocklists and severity thresholds.
6. Apply Azure Policy to enforce compliance guardrails across AI resources.
7. Execute a comprehensive security hardening checklist for production readiness.
8. Set up private endpoints and network isolation for zero-trust architectures.

---

## Module Outline

### Section 1 — Introduction (10 min)
- Module roadmap and position within ARC 9 (CAPSTONE & MASTERY).
- Why governance, monitoring, and security are the final gate before production.
- Overview of the security layers: identity, network, data, application.

### Section 2 — RBAC for Azure AI Foundry (20 min)
- The RBAC assignment model: security principal + role definition + scope.
- Built-in roles for AI Foundry: Azure AI Developer, Cognitive Services User/Contributor, OpenAI User/Contributor.
- Assigning roles via Azure portal, CLI, and Bicep/Terraform.
- Best practice: use Entra ID security groups, not individual assignments.

### Section 3 — Managed Identity Patterns (20 min)
- System-assigned vs. user-assigned managed identities.
- Enabling managed identity on AI Services, App Service, and Container Apps.
- Using `DefaultAzureCredential` in Python for seamless local → cloud auth.
- Anti-pattern: hardcoded API keys and connection strings.
- Granting managed identity access to Key Vault, Storage, and AI Search.

### Section 4 — Monitoring Dashboards (20 min)
- Key metrics: API latency (P50/P95/P99), error rates, token consumption, cost.
- Building Azure Monitor workbooks with KQL queries.
- Diagnostic settings: routing logs and metrics to Log Analytics.
- Application Insights integration for end-to-end distributed tracing.

### Section 5 — Alerts & Action Groups (15 min)
- Designing an alert strategy: severity levels, evaluation windows, thresholds.
- Creating metric alerts via CLI and portal.
- Action groups: email, SMS, webhook (Teams/Slack), Azure Functions, Logic Apps.
- Alert processing rules for maintenance windows and suppression.

### Section 6 — Content Safety & Compliance (20 min)
- Azure AI Content Safety: input filters, output filters, Prompt Shield.
- Category filtering: hate, violence, sexual, self-harm with severity thresholds.
- Custom blocklists for domain-specific content filtering.
- Protected material detection and groundedness checks.
- Azure Policy for compliance enforcement: required SKUs, private endpoints, regions.
- Microsoft Purview for data classification and lineage.

### Section 7 — Network Security (15 min)
- Private endpoints for AI Services, Storage, Key Vault, AI Search.
- Disabling public network access.
- Private DNS zones and VNet linking.
- Managed virtual networks in Azure AI Foundry (preview).
- NSG rules and Azure Firewall integration.

### Section 8 — Security Hardening Checklist (15 min)
- Walk through the comprehensive checklist: identity, network, data, monitoring.
- Microsoft Defender for Cloud recommendations for AI resources.
- Privileged Identity Management (PIM) for just-in-time access.
- Incident response planning and runbook development.

### Section 9 — Mini Hack (45 min)
- Hands-on: implement the complete governance and security layer.
- See `lab/hands-on-lab.md` for detailed instructions.

---

## Featured Video

**AI Foundry Networking and Security: Deep Dive**
- YouTube: [https://www.youtube.com/watch?v=bL9n83uPV84](https://www.youtube.com/watch?v=bL9n83uPV84)
- Embed: `https://www.youtube.com/embed/bL9n83uPV84`
- Show during Section 7 or as pre-class preparation.

---

## Trainer Notes

- **Common misconception**: Learners may confuse Azure AI Foundry with Azure Machine Learning — clarify that AI Foundry is the unified platform for building AI applications, not a model-training-only platform.
- **Demo tip**: Show a live RBAC assignment in the Azure portal, then demonstrate how a user without the correct role gets a 403 error when calling the API. This makes the concept concrete.
- **Lab preparation**: Ensure the lab environment has a pre-provisioned resource group with AI Services, Key Vault, Storage, and Log Analytics workspace. Learners should have Contributor access to the resource group.
- **Content safety demo**: Use the Azure AI Content Safety Studio (portal) to show real-time text analysis with different severity levels. This is more impactful than CLI commands alone.
- **Time management**: The security hardening checklist can generate deep discussions. Consider running it as a group exercise where pairs take different sections.
- **Capstone connection**: Emphasize that Module 45 (final capstone review) will evaluate the governance layer built in this module. Everything here directly affects the capstone grade.

---

## Key Terminology

| Term | Definition |
|------|-----------|
| **RBAC** | Role-Based Access Control — authorization model using roles assigned at specific scopes |
| **Managed Identity** | Azure-managed credentials for service-to-service authentication without secrets |
| **Entra ID** | Microsoft's identity platform (formerly Azure Active Directory) |
| **Content Safety** | Azure service for detecting and filtering harmful content in AI applications |
| **Prompt Shield** | Protection against prompt injection and jailbreak attacks |
| **Private Endpoint** | Network interface that connects privately to an Azure service via Private Link |
| **Azure Policy** | Service for creating, assigning, and managing compliance policies at scale |
| **PIM** | Privileged Identity Management — just-in-time, time-bound elevated access |
| **KQL** | Kusto Query Language — query language for Azure Monitor Log Analytics |
| **Action Group** | Collection of notification preferences (email, SMS, webhook) triggered by alerts |

---

## Required Azure Resources

| Resource | SKU / Tier | Purpose |
|----------|-----------|---------|
| Azure AI Services | Standard S0 | Core AI services with RBAC and managed identity |
| Azure OpenAI | Standard | Model deployments with content filtering |
| Azure Key Vault | Standard | Secrets and certificate management |
| Azure Storage Account | Standard LRS | Data storage with private endpoint |
| Log Analytics Workspace | Pay-as-you-go | Centralized log collection and KQL queries |
| Azure AI Search | Basic | Search service with private endpoint |
| Azure AI Content Safety | Standard S0 | Content filtering and blocklist management |
| Virtual Network | — | Network isolation with private endpoint subnet |

---

## Assessment

- See `quiz/assessment.md` for the 10-question multiple-choice assessment.
- Passing score: 80% (8 of 10 correct).
- Time limit: 15 minutes.

---

## Next Module

**Module 45 — Capstone Review & Certification** — Present your complete Azure AI Foundry solution, including the governance and security layer from this module, for peer evaluation and certification.
