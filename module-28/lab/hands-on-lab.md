# 🧪 Hands-On Lab: Build a Monitoring Dashboard for Your Deployed Model

> **Module 28** — Monitoring, Observability & Drift Detection  
> **Duration:** 80 minutes · **Level:** Intermediate–Advanced  
> **Azure AI Foundry: Zero to Hero** · ARC 6: Deployment & MLOps

---

## 🎯 Lab Objectives

In this lab, you will build a complete monitoring and observability setup for a deployed AI model in Azure AI Foundry. By the end, you will have:

- ✅ A model endpoint instrumented with OpenTelemetry
- ✅ A Log Analytics workspace receiving AI telemetry
- ✅ KQL queries for latency, token usage, and error rate analysis
- ✅ An Azure Monitor dashboard with 4+ real-time tiles
- ✅ Drift detection evaluation running against production samples
- ✅ Alert rules that fire when latency or errors exceed thresholds

---

## 📋 Prerequisites

| Requirement | Details |
|---|---|
| Azure subscription | With Contributor role |
| Azure AI Foundry project | With a deployed chat completion model |
| Azure CLI | v2.60+ installed and authenticated |
| Python | 3.10+ with pip |
| Resource Group | With Log Analytics workspace (or create one) |

### Required Python Packages

```bash
pip install azure-ai-projects azure-ai-evaluation azure-identity \
            azure-monitor-opentelemetry opentelemetry-api opentelemetry-sdk
```

---

## 🏗️ Lab Architecture

```
┌─────────────────────────┐
│  Python Test Application │
│  (OpenTelemetry SDK)     │
└────────────┬────────────┘
             │ traces + metrics
             ▼
┌─────────────────────────┐       ┌─────────────────┐
│    Application Insights  │──────▶│  Azure Monitor   │
│    (Connection String)   │       │  Dashboard       │
└────────────┬────────────┘       └─────────────────┘
             │
             ▼
┌─────────────────────────┐       ┌─────────────────┐
│    Log Analytics         │──────▶│  Alert Rules     │
│    Workspace             │       │  (Action Group)  │
└─────────────────────────┘       └─────────────────┘
```

---

## Exercise 1: Set Up the Monitoring Infrastructure (15 min)

### Step 1.1: Create a Log Analytics Workspace

> Skip this step if you already have a Log Analytics workspace.

```bash
# Set variables
RG="module28-monitoring-lab"
LOCATION="eastus2"
LAW_NAME="law-ai-monitoring"

# Create resource group (if needed)
az group create --name $RG --location $LOCATION

# Create Log Analytics workspace
az monitor log-analytics workspace create \
  --resource-group $RG \
  --workspace-name $LAW_NAME \
  --location $LOCATION \
  --retention-time 30
```

### Step 1.2: Create an Application Insights Resource

```bash
# Create Application Insights linked to Log Analytics
az monitor app-insights component create \
  --app "appi-ai-monitoring" \
  --location $LOCATION \
  --resource-group $RG \
  --workspace "/subscriptions/{sub-id}/resourceGroups/$RG/providers/Microsoft.OperationalInsights/workspaces/$LAW_NAME" \
  --kind web

# Get the connection string (save this!)
az monitor app-insights component show \
  --app "appi-ai-monitoring" \
  --resource-group $RG \
  --query connectionString -o tsv
```

> 📝 **Save the connection string** — you'll need it in the next exercise.

### Step 1.3: Enable Diagnostic Settings on Your AI Foundry Project

```bash
# Replace with your AI Foundry project resource ID
AI_PROJECT_ID="/subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.MachineLearningServices/workspaces/{project-name}"

az monitor diagnostic-settings create \
  --name "ai-monitoring-diagnostics" \
  --resource "$AI_PROJECT_ID" \
  --workspace "/subscriptions/{sub-id}/resourceGroups/$RG/providers/Microsoft.OperationalInsights/workspaces/$LAW_NAME" \
  --logs '[
    {"category": "AmlComputeClusterEvent", "enabled": true},
    {"category": "AmlRunStatusChangedEvent", "enabled": true},
    {"category": "AmlModelsEvent", "enabled": true}
  ]' \
  --metrics '[{"category": "AllMetrics", "enabled": true}]'
```

### ✅ Checkpoint 1

- [ ] Log Analytics workspace created and accessible
- [ ] Application Insights resource linked to the workspace
- [ ] Diagnostic settings enabled on your AI Foundry project

---

## Exercise 2: Instrument Your Application (20 min)

### Step 2.1: Create the Monitoring Wrapper

Create a file named `monitored_client.py`:

```python
"""
Monitored AI Client — OpenTelemetry instrumentation for Azure AI Foundry
"""

import time
from azure.monitor.opentelemetry import configure_azure_monitor
from opentelemetry import trace, metrics
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

# ── Configure Azure Monitor ──
# Replace with your Application Insights connection string
CONNECTION_STRING = "InstrumentationKey=<your-key>;IngestionEndpoint=https://..."

configure_azure_monitor(connection_string=CONNECTION_STRING)

# ── Set up tracer and meter ──
tracer = trace.get_tracer("ai-monitoring-lab")
meter = metrics.get_meter("ai-monitoring-lab")

# ── Define custom metrics ──
request_counter = meter.create_counter(
    "ai.requests.total",
    description="Total model inference requests"
)
error_counter = meter.create_counter(
    "ai.errors.total",
    description="Total model inference errors"
)
token_counter = meter.create_counter(
    "ai.tokens.consumed",
    description="Total tokens consumed"
)
latency_histogram = meter.create_histogram(
    "ai.request.latency_ms",
    unit="ms",
    description="Model request latency in milliseconds"
)


class MonitoredModelClient:
    """Wrapper around AIProjectClient that captures AI-specific telemetry."""

    def __init__(self, project_connection_string: str):
        self.client = AIProjectClient.from_connection_string(
            credential=DefaultAzureCredential(),
            conn_str=project_connection_string
        )

    def complete(self, messages: list[dict], model: str = "gpt-4o", **kwargs):
        """Send a chat completion request with full telemetry capture."""
        with tracer.start_as_current_span("model.completion") as span:
            span.set_attribute("ai.model", model)
            span.set_attribute("ai.message_count", len(messages))

            start_time = time.perf_counter()
            request_counter.add(1, {"model": model})

            try:
                response = self.client.inference.get_chat_completions(
                    model=model,
                    messages=messages,
                    **kwargs
                )

                # Record latency
                latency_ms = (time.perf_counter() - start_time) * 1000
                latency_histogram.record(latency_ms, {"model": model})

                # Record token usage
                usage = response.usage
                token_counter.add(
                    usage.prompt_tokens,
                    {"model": model, "type": "prompt"}
                )
                token_counter.add(
                    usage.completion_tokens,
                    {"model": model, "type": "completion"}
                )

                # Enrich the span
                span.set_attribute("ai.latency_ms", round(latency_ms, 2))
                span.set_attribute("ai.tokens.prompt", usage.prompt_tokens)
                span.set_attribute("ai.tokens.completion", usage.completion_tokens)
                span.set_attribute("ai.tokens.total",
                                   usage.prompt_tokens + usage.completion_tokens)

                return response

            except Exception as e:
                latency_ms = (time.perf_counter() - start_time) * 1000
                error_counter.add(1, {
                    "model": model,
                    "error_type": type(e).__name__
                })
                span.set_status(trace.StatusCode.ERROR, str(e))
                span.record_exception(e)
                raise
```

### Step 2.2: Generate Baseline Traffic

Create a file named `generate_traffic.py`:

```python
"""
Generate test traffic to populate monitoring dashboards.
"""

import time
import random
from monitored_client import MonitoredModelClient

# Replace with your project connection string
PROJECT_CONN = "<your-ai-foundry-project-connection-string>"

client = MonitoredModelClient(PROJECT_CONN)

# Test prompts with varying complexity
PROMPTS = [
    "What is Azure AI Foundry?",
    "Explain the difference between GPT-4 and GPT-4o.",
    "Write a Python function to calculate the Fibonacci sequence using memoization. Include type hints and docstring.",
    "Summarize the key principles of the Azure Well-Architected Framework.",
    "What are the best practices for securing API keys in a cloud-native application?",
    "Explain vector databases and their role in RAG architectures.",
    "Write a KQL query to find the top 10 slowest requests in the last 24 hours.",
    "Compare Azure AI Foundry with other AI platforms. What are the key differentiators?",
    "Explain the concept of model drift in production AI systems.",
    "Write a Bicep template for deploying an Azure AI Foundry project.",
]

print("🚀 Generating baseline traffic...")
print(f"   Sending {len(PROMPTS)} requests\n")

results = []
for i, prompt in enumerate(PROMPTS, 1):
    print(f"  [{i}/{len(PROMPTS)}] Sending: {prompt[:60]}...")
    try:
        response = client.complete(
            messages=[
                {"role": "system", "content": "You are a helpful Azure expert."},
                {"role": "user", "content": prompt}
            ],
            model="gpt-4o",
            max_tokens=500
        )
        tokens = response.usage.prompt_tokens + response.usage.completion_tokens
        print(f"           ✓ {tokens} tokens")
        results.append({"prompt": prompt, "tokens": tokens, "status": "success"})
    except Exception as e:
        print(f"           ✗ Error: {e}")
        results.append({"prompt": prompt, "tokens": 0, "status": f"error: {e}"})

    # Small delay between requests
    time.sleep(random.uniform(1, 3))

# Print summary
success = sum(1 for r in results if r["status"] == "success")
total_tokens = sum(r["tokens"] for r in results)
print(f"\n📊 Summary:")
print(f"   Successful: {success}/{len(results)}")
print(f"   Total tokens: {total_tokens:,}")
print(f"   Avg tokens/request: {total_tokens // max(success, 1):,}")
print(f"\n✅ Baseline traffic generated! Wait 2-5 minutes for data to appear in Log Analytics.")
```

### Step 2.3: Run the Traffic Generator

```bash
python generate_traffic.py
```

> ⏳ Wait 2–5 minutes for telemetry to propagate to Log Analytics.

### ✅ Checkpoint 2

- [ ] `monitored_client.py` created with OpenTelemetry instrumentation
- [ ] `generate_traffic.py` created and executed successfully
- [ ] At least 10 requests completed without errors
- [ ] Console shows token counts for each request

---

## Exercise 3: Build KQL Queries (15 min)

Navigate to your **Log Analytics workspace** in the Azure Portal and run these queries.

### Query 1: Latency Percentiles

```kql
// Latency percentiles over the last 4 hours
customMetrics
| where name == "ai.request.latency_ms"
| where timestamp > ago(4h)
| summarize
    P50 = percentile(value, 50),
    P95 = percentile(value, 95),
    P99 = percentile(value, 99),
    Requests = count()
  by bin(timestamp, 5m)
| render timechart
```

### Query 2: Token Usage Breakdown

```kql
// Token consumption by type (prompt vs. completion)
customMetrics
| where name == "ai.tokens.consumed"
| where timestamp > ago(4h)
| extend TokenType = tostring(customDimensions.type)
| summarize TotalTokens = sum(value) by bin(timestamp, 5m), TokenType
| render barchart
```

### Query 3: Error Rate

```kql
// Error rate over time
let allRequests = customMetrics
    | where name == "ai.requests.total"
    | where timestamp > ago(4h)
    | summarize TotalRequests = sum(value) by bin(timestamp, 5m);
let errors = customMetrics
    | where name == "ai.errors.total"
    | where timestamp > ago(4h)
    | summarize TotalErrors = sum(value) by bin(timestamp, 5m);
allRequests
| join kind=leftouter errors on timestamp
| extend ErrorRate = iff(TotalRequests > 0,
    round(TotalErrors * 100.0 / TotalRequests, 2), 0.0)
| project timestamp, TotalRequests, TotalErrors, ErrorRate
| render timechart
```

### Query 4: Request Summary Table

```kql
// Summary dashboard data
customMetrics
| where name in ("ai.requests.total", "ai.errors.total", "ai.tokens.consumed", "ai.request.latency_ms")
| where timestamp > ago(1h)
| summarize
    Value = iff(name == "ai.request.latency_ms", round(avg(value), 1), sum(value))
  by name
| project Metric = name, Value
```

### ✅ Checkpoint 3

- [ ] All 4 queries execute without errors
- [ ] Latency query shows data points (timechart renders)
- [ ] Token query shows prompt vs. completion breakdown
- [ ] You've saved each query for use in the dashboard

---

## Exercise 4: Create the Azure Monitor Dashboard (15 min)

### Step 4.1: Create a New Dashboard

1. Go to **Azure Portal** → **Dashboard** → **+ New dashboard** → **Blank dashboard**
2. Name it: `AI Model Monitoring — Module 28`

### Step 4.2: Add Tiles

Add these tiles by clicking **+ Add tile** → **Logs** and pasting KQL queries:

| Tile # | Title | Query | Visualization |
|---|---|---|---|
| 1 | Latency Percentiles | Query 1 (latency) | Time chart |
| 2 | Token Usage by Type | Query 2 (tokens) | Bar chart |
| 3 | Error Rate | Query 3 (errors) | Time chart |
| 4 | Request Summary | Query 4 (summary) | Table |

### Step 4.3: Arrange the Layout

Arrange tiles in a 2×2 grid:

```
┌──────────────────────┬──────────────────────┐
│  Latency Percentiles │  Token Usage by Type │
│  (time chart)        │  (bar chart)         │
├──────────────────────┼──────────────────────┤
│  Error Rate          │  Request Summary     │
│  (time chart)        │  (table)             │
└──────────────────────┴──────────────────────┘
```

### Step 4.4: Save the Dashboard

Click **Save** and optionally **Share** with your team.

### ✅ Checkpoint 4

- [ ] Dashboard created with 4 tiles
- [ ] Each tile displays data from your test traffic
- [ ] Dashboard is saved and accessible

---

## Exercise 5: Implement Drift Detection (10 min)

### Step 5.1: Create the Drift Evaluator

Create a file named `drift_check.py`:

```python
"""
Drift detection using Azure AI Foundry evaluators.
"""

import statistics
from azure.ai.projects import AIProjectClient
from azure.ai.evaluation import (
    GroundednessEvaluator,
    RelevanceEvaluator,
    CoherenceEvaluator,
)
from azure.identity import DefaultAzureCredential

# Replace with your project details
PROJECT_CONN = "<your-ai-foundry-project-connection-string>"

project = AIProjectClient.from_connection_string(
    credential=DefaultAzureCredential(),
    conn_str=PROJECT_CONN
)

# Baseline thresholds (from initial evaluation)
BASELINES = {
    "groundedness": 0.80,
    "relevance":    0.75,
    "coherence":    0.80,
}

DRIFT_THRESHOLD = 0.10  # 10% drop from baseline

# Sample production data (replace with real sampled data)
SAMPLES = [
    {
        "query": "What is Azure AI Foundry?",
        "response": "Azure AI Foundry is Microsoft's unified platform for building, deploying, and managing AI applications. It provides tools for model deployment, prompt engineering, evaluation, and monitoring.",
        "context": "Azure AI Foundry is a comprehensive platform that unifies AI development tools, model catalog, prompt flow, evaluations, and deployment capabilities."
    },
    {
        "query": "How do I deploy a model in Azure AI Foundry?",
        "response": "To deploy a model, navigate to the Model Catalog in your AI Foundry project, select a model, configure deployment settings including compute and scaling, then click Deploy.",
        "context": "Models can be deployed through the Azure AI Foundry portal model catalog. Select a model, choose deployment options, configure compute resources, and deploy to an endpoint."
    },
    {
        "query": "What is RAG?",
        "response": "RAG (Retrieval-Augmented Generation) is a pattern that combines information retrieval with language model generation. It retrieves relevant context from a knowledge base and uses it to ground the model's response.",
        "context": "RAG combines retrieval and generation. A retriever fetches relevant documents from a knowledge base, which are then provided as context to a language model to generate grounded responses."
    },
]

print("🔍 Running drift detection evaluation...\n")

evaluators = {
    "groundedness": GroundednessEvaluator(project),
    "relevance":    RelevanceEvaluator(project),
    "coherence":    CoherenceEvaluator(project),
}

results = {name: [] for name in evaluators}

for i, sample in enumerate(SAMPLES, 1):
    print(f"  Evaluating sample {i}/{len(SAMPLES)}: {sample['query'][:50]}...")
    for name, evaluator in evaluators.items():
        score = evaluator(
            query=sample["query"],
            response=sample["response"],
            context=sample.get("context", "")
        )
        results[name].append(score.score)

print("\n📊 Drift Detection Results:")
print("-" * 65)
print(f"{'Metric':<18} {'Current':<10} {'Baseline':<10} {'Drift %':<10} {'Status'}")
print("-" * 65)

any_drift = False
for metric, scores in results.items():
    avg_score = statistics.mean(scores)
    baseline = BASELINES[metric]
    drift_pct = ((baseline - avg_score) / baseline) * 100

    drifted = drift_pct > (DRIFT_THRESHOLD * 100)
    status = "🚨 DRIFT" if drifted else "✅ OK"
    if drifted:
        any_drift = True

    print(f"  {metric:<16} {avg_score:<10.3f} {baseline:<10.2f} {drift_pct:<10.1f} {status}")

print("-" * 65)

if any_drift:
    print("\n⚠️  Drift detected! Review model performance and consider retraining or prompt updates.")
else:
    print("\n✅ All metrics within acceptable thresholds.")
```

### Step 5.2: Run the Drift Check

```bash
python drift_check.py
```

### ✅ Checkpoint 5

- [ ] `drift_check.py` executes without errors
- [ ] Drift scores are computed for groundedness, relevance, and coherence
- [ ] Output shows current vs. baseline comparison

---

## Exercise 6: Create Alert Rules (10 min)

### Step 6.1: Create a Latency Alert

```bash
# Create action group first
az monitor action-group create \
  --resource-group $RG \
  --name "ai-ops-team" \
  --short-name "AIops" \
  --email-receiver name="Team Lead" email-address="your-email@company.com"

# Create latency alert
az monitor scheduled-query create \
  --name "ai-high-latency" \
  --resource-group $RG \
  --scopes "/subscriptions/{sub-id}/resourceGroups/$RG/providers/Microsoft.OperationalInsights/workspaces/$LAW_NAME" \
  --condition "count > 5" \
  --condition-query "customMetrics
    | where name == 'ai.request.latency_ms'
    | where value > 5000
    | where timestamp > ago(5m)
    | summarize HighLatencyCount = count()" \
  --severity 2 \
  --evaluation-frequency "5m" \
  --window-size "5m" \
  --action-groups "/subscriptions/{sub-id}/resourceGroups/$RG/providers/Microsoft.Insights/actionGroups/ai-ops-team" \
  --description "AI model requests exceeding 5 second latency threshold"
```

### Step 6.2: Create an Error Rate Alert

```bash
az monitor scheduled-query create \
  --name "ai-error-spike" \
  --resource-group $RG \
  --scopes "/subscriptions/{sub-id}/resourceGroups/$RG/providers/Microsoft.OperationalInsights/workspaces/$LAW_NAME" \
  --condition "count > 0" \
  --condition-query "let errors = customMetrics
    | where name == 'ai.errors.total'
    | where timestamp > ago(10m)
    | summarize ErrorCount = sum(value);
    let total = customMetrics
    | where name == 'ai.requests.total'
    | where timestamp > ago(10m)
    | summarize TotalCount = sum(value);
    errors | join total on \$left.ErrorCount == \$left.ErrorCount
    | extend ErrorRate = ErrorCount * 100.0 / TotalCount
    | where ErrorRate > 5" \
  --severity 1 \
  --evaluation-frequency "5m" \
  --window-size "10m" \
  --action-groups "/subscriptions/{sub-id}/resourceGroups/$RG/providers/Microsoft.Insights/actionGroups/ai-ops-team" \
  --description "AI model error rate exceeds 5% threshold"
```

### Step 6.3: Verify Alert Rules

```bash
az monitor scheduled-query list \
  --resource-group $RG \
  --query "[].{Name:name, Severity:severity, Enabled:enabled}" \
  -o table
```

### ✅ Checkpoint 6

- [ ] Action group created with email receiver
- [ ] Latency alert rule created (Sev 2)
- [ ] Error rate alert rule created (Sev 1)
- [ ] Both alerts show as Enabled in the list

---

## 🏁 Final Validation

Run through this checklist to confirm lab completion:

| # | Criterion | Status |
|---|---|---|
| 1 | Log Analytics workspace receives telemetry from AI Foundry | ☐ |
| 2 | Application Insights captures custom AI metrics | ☐ |
| 3 | Python wrapper captures latency, tokens, and errors | ☐ |
| 4 | KQL queries return data for latency, tokens, errors | ☐ |
| 5 | Azure Monitor dashboard has 4+ tiles with live data | ☐ |
| 6 | Drift detection evaluates 3+ quality metrics | ☐ |
| 7 | At least 2 alert rules are created and enabled | ☐ |

---

## 🧹 Cleanup (Optional)

To avoid ongoing charges, delete the lab resources:

```bash
az group delete --name $RG --yes --no-wait
```

---

## 🎓 Stretch Goals

If you finish early, try these:

1. **Add a cost tracking tile** — Calculate estimated cost per request based on token pricing
2. **Dynamic thresholds** — Switch from static to Azure Monitor dynamic threshold alerting
3. **Workbook** — Create an Azure Monitor Workbook with parameterized queries (time range, model name)
4. **Automated drift pipeline** — Schedule `drift_check.py` to run daily via Azure Functions
5. **Custom dashboard** — Add tiles for content filter trigger rates and rate limiting events

---

> **Next:** [Module 29 →](../../module-29/lab/hands-on-lab.md)  
> **Back:** [Module 27 ←](../../module-27/lab/hands-on-lab.md)
