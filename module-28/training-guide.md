# Module 28: Monitoring, Observability & Drift Detection

> **Azure AI Foundry: Zero to Hero** · ARC 6: Deployment & MLOps · Module 28 of 45  
> **Date:** April 2026 · **Level:** Intermediate–Advanced · **Duration:** 4 hours (instructor-led)

---

## 🎯 Module Overview

This module teaches participants how to build production-grade observability for AI workloads running on Azure AI Foundry. Students will learn to instrument model endpoints with Azure Monitor, track critical metrics (latency, token usage, error rates), implement drift detection pipelines, and build real-time dashboards with intelligent alerting.

### Learning Objectives

By the end of this module, participants will be able to:

1. **Configure Azure Monitor** diagnostic settings for AI Foundry projects
2. **Instrument applications** with OpenTelemetry to capture AI-specific metrics
3. **Write KQL queries** to analyze latency percentiles, token usage trends, and error rates
4. **Implement drift detection** using Azure AI Foundry evaluation framework
5. **Create alert rules** with appropriate severity levels and action groups
6. **Build monitoring dashboards** in Azure Portal with AI-focused tiles
7. **Troubleshoot** common AI observability issues in production

### Prerequisites

| Requirement | Details |
|---|---|
| Azure subscription | With Contributor access |
| Azure AI Foundry project | With a deployed model endpoint |
| Module 25–27 completion | Deployment fundamentals |
| Python 3.10+ | With pip |
| Azure CLI | v2.60+ |
| Familiarity with KQL | Basic query syntax |

---

## 📋 Agenda

| Time | Topic | Type |
|---|---|---|
| 0:00–0:20 | Why Observability Matters for AI | Lecture |
| 0:20–0:50 | Azure Monitor Integration | Lecture + Demo |
| 0:50–1:20 | Key Metrics Deep Dive | Lecture + Demo |
| 1:20–1:30 | ☕ Break | — |
| 1:30–2:00 | Model Drift & Quality Degradation | Lecture |
| 2:00–2:30 | Alerting & Dashboard Design | Lecture + Demo |
| 2:30–2:40 | Application Insights for Agents | Lecture |
| 2:40–4:00 | 🧪 Mini Hack: Build a Monitoring Dashboard | Hands-on Lab |

---

## 📚 Section 1: Why Observability Matters for AI (20 min)

### Instructor Notes

Begin by contrasting traditional application monitoring with AI-specific observability. The key insight is that AI systems fail *silently* — quality degrades without HTTP errors. Use the analogy: *"Traditional monitoring tells you the restaurant is open. AI monitoring tells you the food still tastes good."*

### Key Concepts

#### The Silent Failure Problem

- Traditional apps: failures = errors, crashes, timeouts → easy to detect
- AI systems: failures = wrong answers, lower quality, irrelevant responses → hard to detect
- Without AI-specific monitoring, degradation goes unnoticed until user complaints

#### Three + One Pillars of AI Observability

| Pillar | Traditional | AI-Specific |
|---|---|---|
| **Metrics** | CPU, memory, HTTP status | Latency, tokens, cost-per-request |
| **Traces** | Request flow through services | Agent step tracing, tool invocations |
| **Logs** | Error logs, access logs | Prompt/completion pairs, guardrail events |
| **Evaluations** | *(none)* | Quality scores, drift indicators, safety checks |

### Discussion Questions

1. How would you know if your AI agent started giving slightly worse answers?
2. What's the cost of detecting quality degradation 1 week late vs. 1 hour late?
3. How does AI observability differ from traditional APM?

---

## 📚 Section 2: Azure Monitor Integration for AI Workloads (30 min)

### Instructor Notes

Walk through the architecture diagram showing how Azure AI Foundry telemetry flows to Azure Monitor. Demo enabling diagnostic settings in the Azure Portal. Show how to connect Application Insights.

### Key Components

| Component | Role |
|---|---|
| **Azure Monitor** | Unified observability platform |
| **Log Analytics** | KQL-based log storage and querying |
| **Application Insights** | Distributed tracing and APM |
| **Azure Workbooks** | Interactive dashboard reports |
| **Action Groups** | Alert notification routing |

### Demo: Enable Diagnostic Settings

1. Navigate to Azure AI Foundry project in Azure Portal
2. Go to **Monitoring** → **Diagnostic settings**
3. Click **+ Add diagnostic setting**
4. Select log categories: `AmlComputeClusterEvent`, `AmlRunStatusChangedEvent`, `AmlModelsEvent`
5. Select metric category: `AllMetrics`
6. Choose destination: Log Analytics workspace
7. Save

### Demo: OpenTelemetry Instrumentation

Walk through the `MonitoredModelClient` wrapper class:

```python
from azure.monitor.opentelemetry import configure_azure_monitor
from opentelemetry import trace, metrics

configure_azure_monitor(
    connection_string="InstrumentationKey=<key>;IngestionEndpoint=https://..."
)

tracer = trace.get_tracer("ai-foundry-app")
meter = metrics.get_meter("ai-foundry-app")
```

**Key packages:**
- `azure-monitor-opentelemetry`
- `opentelemetry-api`
- `opentelemetry-sdk`

---

## 📚 Section 3: Key Metrics Deep Dive (30 min)

### Instructor Notes

Walk through each metric category with concrete KQL queries. Show how to build queries incrementally in Log Analytics.

### Critical Metrics Table

| Category | Metric | Why It Matters | Alert When |
|---|---|---|---|
| **Latency** | P50/P95/P99 response time | User experience | P95 > 3s for 5 min |
| **Token Usage** | Prompt + completion tokens | Cost control | > 2x baseline average |
| **Error Rate** | 4xx/5xx, filter blocks | Reliability | > 1% over 10 min |
| **Throughput** | Requests per minute | Capacity planning | > 80% of rate limit |
| **Rate Limiting** | 429 response count | Quota saturation | Any sustained 429s |
| **Safety** | Content filter triggers | Guardrail health | > 5% trigger rate |

### KQL Query Workshop

Provide these queries for participants to run in Log Analytics:

**Query 1: Latency Percentiles**
```kql
AzureDiagnostics
| where Category == "AmlModelsEvent"
| where TimeGenerated > ago(24h)
| summarize
    P50=percentile(DurationMs, 50),
    P95=percentile(DurationMs, 95),
    P99=percentile(DurationMs, 99)
  by bin(TimeGenerated, 1h)
| render timechart
```

**Query 2: Error Rate by Model**
```kql
AzureDiagnostics
| where Category == "AmlModelsEvent"
| where TimeGenerated > ago(24h)
| summarize
    Total = count(),
    Errors = countif(ResultType != "Success"),
    ErrorRate = round(countif(ResultType != "Success") * 100.0 / count(), 2)
  by bin(TimeGenerated, 1h)
| render timechart
```

**Query 3: Token Usage Trends**
```kql
customMetrics
| where name == "ai.tokens.consumed"
| where timestamp > ago(7d)
| summarize TotalTokens=sum(value) by bin(timestamp, 1h)
| render barchart
```

---

## 📚 Section 4: Model Drift & Quality Degradation (30 min)

### Instructor Notes

This is the most conceptually challenging section. Use visual aids (the drift bar chart from the blog) to illustrate how quality scores change over time. Emphasize that drift is *inevitable* — the question is how fast you detect it.

### Types of Drift

| Drift Type | Description | Example |
|---|---|---|
| **Data Drift** | Input distribution changes | Users start asking about new topics |
| **Concept Drift** | Correct answers change | Regulations or product info updates |
| **Quality Drift** | Output quality degrades | Model responses become less accurate |
| **Behavioral Drift** | Model behavior changes | Provider updates the model version |

### Drift Detection Approach

1. **Establish baselines** — Run evaluations on a golden dataset before launch
2. **Sample production traffic** — Randomly select N requests per day
3. **Run automated evaluations** — Use AI-judge evaluators (groundedness, relevance, coherence, fluency)
4. **Compare against baselines** — Flag when scores drop > 10% from baseline
5. **Alert and investigate** — Trigger notification and root cause analysis

### Key Evaluators in Azure AI Foundry

- `GroundednessEvaluator` — Is the response grounded in provided context?
- `RelevanceEvaluator` — Does the response address the user's question?
- `CoherenceEvaluator` — Is the response logically coherent?
- `FluencyEvaluator` — Is the response well-written and natural?

---

## 📚 Section 5: Alerting & Dashboard Design (30 min)

### Instructor Notes

Focus on *actionable* alerting. Discuss alert fatigue and severity levels. Demo creating a dashboard in Azure Portal.

### Alert Design Principles

1. **Be specific** — Alert on the metric that matters, not a proxy
2. **Include context** — Alert messages should tell the operator what to check first
3. **Use severity levels** — Sev 1 = page, Sev 2 = urgent, Sev 3 = next business day
4. **Dynamic thresholds** — Let Azure Monitor learn normal patterns vs. static thresholds
5. **Suppress during maintenance** — Use action rules to suppress during planned outages

### Dashboard Layout Recommendations

| Row | Left Panel | Right Panel |
|---|---|---|
| 1 | Latency percentiles (timechart) | Request volume (barchart) |
| 2 | Error rate (timechart) | Token usage trend (barchart) |
| 3 | Drift scores (timechart) | Top errors (table) |
| 4 | Cost trend (barchart) | Active alerts (list) |

---

## 📚 Section 6: Application Insights for AI Agents (10 min)

### Quick Coverage Points

- Application Insights captures distributed traces for agent invocations
- Each agent step (LLM call, tool use, retrieval) is a span in the trace
- Use `operation_Id` to correlate all steps in a single invocation
- PII considerations: sample or redact prompt/completion content
- Use Live Metrics for real-time monitoring during agent debugging

---

## 🧪 Mini Hack: Build a Monitoring Dashboard (80 min)

*See `lab/hands-on-lab.md` for detailed step-by-step instructions.*

### Hack Overview

Students will build a complete monitoring setup including:
- OpenTelemetry instrumentation for a deployed model
- Log Analytics queries for AI metrics
- An Azure Monitor dashboard with 4+ tiles
- Drift detection evaluation
- At least 2 alert rules

### Success Criteria

- [ ] Dashboard displays real-time latency, token, and error metrics
- [ ] KQL queries return accurate percentile data
- [ ] Drift evaluation produces scores for 3+ quality metrics
- [ ] Alert fires when simulated degradation exceeds thresholds

---

## 📖 Additional Resources

| Resource | Link |
|---|---|
| Azure Monitor documentation | [learn.microsoft.com/azure/azure-monitor](https://learn.microsoft.com/azure/azure-monitor) |
| Azure AI Foundry monitoring | [learn.microsoft.com/azure/ai-studio/how-to/monitor](https://learn.microsoft.com/azure/ai-studio/how-to/monitor) |
| OpenTelemetry for Python | [opentelemetry.io/docs/languages/python](https://opentelemetry.io/docs/languages/python) |
| KQL reference | [learn.microsoft.com/azure/data-explorer/kql-quick-reference](https://learn.microsoft.com/azure/data-explorer/kql-quick-reference) |
| Azure AI Evaluation SDK | [learn.microsoft.com/python/api/azure-ai-evaluation](https://learn.microsoft.com/python/api/azure-ai-evaluation) |
| Featured Video | [Debug Your AI Agents: Fix Failures Fast with Observability](https://www.youtube.com/watch?v=DpvR9ZgXK84) |

---

## ✅ Module Checklist

- [ ] Understand why AI observability differs from traditional monitoring
- [ ] Can configure diagnostic settings for Azure AI Foundry
- [ ] Can instrument applications with OpenTelemetry
- [ ] Can write KQL queries for AI metrics
- [ ] Understand drift types and detection approaches
- [ ] Can create alert rules with appropriate thresholds
- [ ] Can build Azure Monitor dashboards for AI workloads
- [ ] Completed Mini Hack: monitoring dashboard

---

> **Next Module:** [Module 29 →](../module-29/training-guide.md)  
> **Previous Module:** [← Module 27](../module-27/training-guide.md)
