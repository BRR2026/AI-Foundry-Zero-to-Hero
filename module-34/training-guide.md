# Module 34 — Responsible AI Dashboards & Content Safety at Scale

## Azure AI Foundry: Zero to Hero (Module 34 of 45)

**Arc 7: Security, Governance & Compliance** | April 2026

---

## 📋 Module Overview

| Field | Details |
|---|---|
| **Title** | Responsible AI Dashboards & Content Safety at Scale |
| **Arc** | 7 — Security, Governance & Compliance |
| **Duration** | 90 minutes |
| **Level** | Advanced |
| **Prerequisites** | Module 33, Azure AI Foundry project, Python 3.10+ |
| **Estimated Cost** | ~$2–5 USD for lab exercises |

## 🎯 Learning Objectives

By the end of this module, learners will be able to:

1. **Configure** Responsible AI dashboards in Azure AI Foundry using SDK and portal approaches
2. **Implement** enterprise-grade content safety policies with custom blocklists, severity thresholds, and multi-layer filtering
3. **Monitor** deployed AI systems for bias, fairness disparities, and harmful outputs using continuous evaluation pipelines
4. **Execute** structured incident response playbooks for AI safety events with defined SLAs and escalation paths
5. **Build** end-to-end content safety pipelines that integrate with Azure Monitor for real-time alerting

---

## 📚 Module Content Outline

### Section 1: Why Responsible AI at Enterprise Scale Matters (15 min)

**Key Concepts:**

- The business case for Responsible AI: trust, regulation, liability
- Microsoft's six Responsible AI principles mapped to Azure AI Foundry capabilities
- Regulatory landscape: EU AI Act, NIST AI RMF, ISO/IEC 42001, Executive Order 14110
- Distinction between Azure AI Foundry RAI tools and Azure Machine Learning RAI Toolbox

**Discussion Points:**

- How does your organization currently assess AI risk?
- What regulatory frameworks apply to your industry?
- Who owns Responsible AI within your organizational structure?

### Section 2: Responsible AI Dashboard Configuration (20 min)

**Key Concepts:**

- RAI Dashboard architecture and data flow within Azure AI Foundry
- Dashboard components: Error Analysis, Fairness Assessment, Data Explorer, Explanation Cards
- Programmatic configuration via Azure AI Foundry SDK (`RAIDashboardConfig`)
- Dashboard-as-Code: version-controlled governance configurations
- Scheduling evaluations and configuring alert recipients

**Hands-On Demo:**

```python
from azure.ai.evaluation import RAIDashboardConfig, EvaluatorConfig
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

credential = DefaultAzureCredential()
project_client = AIProjectClient(
    credential=credential,
    project_name="enterprise-gpt4o",
    subscription_id="your-subscription-id",
    resource_group="rg-ai-production"
)

dashboard_config = RAIDashboardConfig(
    display_name="Enterprise RAI Dashboard - Production",
    evaluators=[
        EvaluatorConfig(
            name="content_safety",
            severity_threshold=2,
            categories=["hate", "sexual", "violence", "self_harm"]
        ),
        EvaluatorConfig(
            name="fairness",
            protected_attributes=["gender", "age_group", "ethnicity"],
            disparity_threshold=0.1
        ),
    ],
    schedule="0 */4 * * *",
    alert_recipients=["rai-team@contoso.com"]
)

dashboard = project_client.rai_dashboards.create(config=dashboard_config)
```

**Best Practices:**

- Store dashboard configurations alongside model deployment configs in source control
- Use separate dashboards for development, staging, and production environments
- Schedule evaluations at intervals appropriate to your traffic volume
- Include both automated and human-in-the-loop review workflows

### Section 3: Content Safety at Enterprise Scale (20 min)

**Key Concepts:**

- Azure AI Content Safety service architecture and capabilities
- Built-in safety categories: Hate & Fairness, Sexual, Violence, Self-Harm
- Advanced detection: Jailbreak attacks, Protected material, Prompt shields
- Eight severity levels (0–7) for granular control
- Multi-modal safety: text, image, and code content
- Content safety pipeline: input filter → custom blocklists → model → output filter

**Custom Blocklists — Deep Dive:**

| Feature | Details |
|---|---|
| Max blocklists per resource | 100 |
| Max items per blocklist | 10,000 |
| Match types | Exact match, substring |
| Supported content | Text (image blocklists in preview) |
| Latency impact | < 5ms per blocklist check |
| API rate limit | 1,000 requests/10 seconds |

**Implementation Pattern:**

```python
from azure.ai.contentsafety import ContentSafetyClient
from azure.ai.contentsafety.models import (
    TextBlocklist,
    AddOrUpdateTextBlocklistItemsOptions,
    TextBlocklistItem,
    AnalyzeTextOptions
)

# Create domain-specific blocklists
blocklists = {
    "financial-compliance": [
        "guaranteed returns", "risk-free investment",
        "insider trading tip", "buy this stock now"
    ],
    "pii-protection": [
        "social security number", "credit card number",
        "bank account details", "passport number"
    ],
    "brand-safety": [
        "competitor-product-x", "defamatory-claim-y"
    ]
}

for name, items in blocklists.items():
    client.create_or_update_text_blocklist(
        blocklist_name=name,
        options=TextBlocklist(blocklist_name=name)
    )
    client.add_or_update_blocklist_items(
        blocklist_name=name,
        options=AddOrUpdateTextBlocklistItemsOptions(
            blocklist_items=[
                TextBlocklistItem(text=item) for item in items
            ]
        )
    )
```

**Enterprise Patterns:**

- Layer 1: Input safety check (before model inference)
- Layer 2: Output safety check (after model inference)
- Layer 3: Groundedness verification (for RAG scenarios)
- Layer 4: Custom business rule validation

### Section 4: Monitoring for Bias, Fairness & Harmful Outputs (15 min)

**Key Fairness Metrics:**

| Metric | Description | Threshold |
|---|---|---|
| Demographic Parity | Equal positive outcome probability across groups | Ratio ≥ 0.8 |
| Equalized Odds | Equal TPR and FPR across groups | Difference ≤ 0.1 |
| Quality Disparity | Output quality gap across demographics | Gap ≤ 0.15 |
| Refusal Rate Disparity | Refusal rate differences by group | Ratio ≥ 0.9 |
| Stereotyping Score | Stereotypical association presence | Score ≤ 0.05 |

**Monitoring Architecture:**

- Azure AI Foundry evaluation pipelines → OpenTelemetry → Azure Monitor
- Custom KQL queries for fairness metric dashboards
- Azure Monitor alert rules with action groups for threshold breaches
- Integration with incident management tools (ServiceNow, PagerDuty, Jira)

**Key KQL Query for Fairness Alerts:**

```kql
customMetrics
| where name == "ai.foundry.fairness.quality_disparity"
| where timestamp > ago(1h)
| summarize
    avg_disparity = avg(value),
    max_disparity = max(value),
    breach_count = countif(value > 0.15)
    by bin(timestamp, 5m), cloud_RoleName
| where breach_count > 0
| order by timestamp desc
```

**Best Practices:**

- Never collect protected demographic data from end users directly
- Use controlled evaluation datasets with known demographic attributes
- Monitor aggregate output patterns for proxy indicators of bias
- Implement drift detection to catch fairness degradation early

### Section 5: Incident Response for AI Safety Events (15 min)

**Incident Classification Matrix:**

| Severity | SLA | Examples |
|---|---|---|
| Critical | 15 min | Self-harm instructions, PII at scale, CSAM |
| High | 1 hour | Jailbreak success, systematic bias, copyright |
| Medium | 4 hours | Elevated false negatives, edge cases |
| Low | 24 hours | Minor toxicity drift, cosmetic issues |

**Incident Response Playbook:**

1. **Phase 1 — Detection** (0–15 min): Alert fires, validate, assign severity, designate incident commander
2. **Phase 2 — Containment** (15–60 min): Emergency blocklist, reduce model temperature, restrict API access
3. **Phase 3 — Investigation** (1–24 hrs): Root cause analysis via tracing, reproduce in sandbox
4. **Phase 4 — Remediation** (1–7 days): Permanent fixes, regression test cases
5. **Phase 5 — Post-Incident Review**: Blameless retrospective, update playbooks, share learnings

**Emergency Response Code Pattern:**

```python
async def activate_emergency_block(client, attack_pattern, incident_id):
    """Immediately block a discovered attack pattern."""
    client.add_or_update_blocklist_items(
        blocklist_name="emergency-hotfix",
        options={
            "blocklistItems": [
                TextBlocklistItem(
                    text=attack_pattern,
                    description=f"Emergency - Incident {incident_id}"
                )
            ]
        }
    )
    await send_incident_notification(
        channel="#ai-safety-critical",
        severity="CRITICAL",
        incident_id=incident_id
    )
```

### Mini Hack: Enterprise Content Safety Policy (30 min)

See the hands-on lab (`lab/hands-on-lab.md`) for the complete step-by-step guide.

---

## 🎥 Featured Video

**Building Trustworthy AI: Ensuring Responsible AI With Content Safety**

- **Source:** Microsoft Azure
- **Video ID:** bkssatMl0NE
- **URL:** [https://www.youtube.com/watch?v=bkssatMl0NE](https://www.youtube.com/watch?v=bkssatMl0NE)

Watch before or after the module for additional context on Microsoft's approach to content safety at scale.

---

## 📖 Recommended Reading

1. [Azure AI Content Safety Documentation](https://learn.microsoft.com/azure/ai-services/content-safety/)
2. [Responsible AI Dashboard in Azure AI Foundry](https://learn.microsoft.com/azure/ai-studio/responsible-use-of-ai-overview)
3. [NIST AI Risk Management Framework](https://www.nist.gov/artificial-intelligence/risk-management-framework)
4. [EU AI Act Overview](https://digital-strategy.ec.europa.eu/en/policies/regulatory-framework-ai)
5. [Microsoft Responsible AI Standard v2](https://aka.ms/rai)
6. [Azure AI Evaluation SDK Reference](https://learn.microsoft.com/python/api/azure-ai-evaluation/)

---

## 🧭 Navigation

| Previous | Current | Next |
|---|---|---|
| [← Module 33](../module-33/blog.html) | **Module 34** — Responsible AI Dashboards & Content Safety at Scale | [Module 35 →](../module-35/blog.html) |

---

## 📝 Assessment

Complete the 10-question assessment in `quiz/assessment.md` to validate your understanding of this module.

---

*Azure AI Foundry: Zero to Hero — Module 34 of 45*
*Arc 7: Security, Governance & Compliance*
*© 2026 Azure AI Foundry Training*
