# Hands-On Lab: Agent Evaluation, Tracing & Debugging

## Module 20 — Azure AI Foundry: Zero to Hero

| Field          | Details                                              |
| -------------- | ---------------------------------------------------- |
| **Duration**   | 60 minutes                                           |
| **Level**      | Intermediate–Advanced                                |
| **Arc**        | 4 — AI Agents                                        |
| **Module**     | 20 of 45                                             |

---

## Lab Overview

In this lab, you'll build a comprehensive observability stack for an Azure AI Foundry agent. You'll evaluate agent quality with automated evaluators, connect Application Insights for distributed tracing, debug a simulated agent failure using KQL queries, and create a monitoring dashboard.

### What You'll Build

- An automated agent evaluation pipeline with ground-truth test cases
- Application Insights integration for end-to-end agent tracing
- Custom OpenTelemetry span enrichment with business attributes
- A KQL-powered dashboard for agent health monitoring
- An alert rule for agent success rate degradation

### Prerequisites

- Azure subscription with an Azure AI Foundry project
- A deployed agent (from Module 17–19 labs)
- Python 3.9+ installed locally
- Azure CLI authenticated (`az login`)
- Visual Studio Code (recommended)

---

## Exercise 1: Set Up Application Insights (10 min)

### Step 1.1: Create an Application Insights Resource

If you don't already have one, create an Application Insights resource:

```bash
# Create a resource group (if needed)
az group create --name rg-agent-observability --location eastus2

# Create Application Insights
az monitor app-insights component create \
  --app ai-agent-insights \
  --location eastus2 \
  --resource-group rg-agent-observability \
  --kind web \
  --application-type web

# Get the connection string
az monitor app-insights component show \
  --app ai-agent-insights \
  --resource-group rg-agent-observability \
  --query connectionString -o tsv
```

> **Save the connection string** — you'll need it in the next steps.

### Step 1.2: Install Required Python Packages

```bash
pip install azure-ai-projects azure-ai-evaluation azure-identity \
    azure-monitor-opentelemetry opentelemetry-api
```

### Step 1.3: Set Environment Variables

```bash
# Linux/macOS
export APPLICATIONINSIGHTS_CONNECTION_STRING="InstrumentationKey=...;IngestionEndpoint=..."
export PROJECT_ENDPOINT="https://<your-project>.services.ai.azure.com"
export AGENT_ID="<your-agent-id>"

# Windows PowerShell
$env:APPLICATIONINSIGHTS_CONNECTION_STRING = "InstrumentationKey=...;IngestionEndpoint=..."
$env:PROJECT_ENDPOINT = "https://<your-project>.services.ai.azure.com"
$env:AGENT_ID = "<your-agent-id>"
```

### ✅ Checkpoint

You have an Application Insights resource created, the connection string saved, and Python packages installed.

---

## Exercise 2: Enable Agent Tracing (15 min)

### Step 2.1: Create the Tracing Setup Script

Create a file named `agent_tracing.py`:

```python
import os
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from azure.monitor.opentelemetry import configure_azure_monitor
from opentelemetry import trace

# ── Configure Azure Monitor telemetry export ──
configure_azure_monitor(
    connection_string=os.environ["APPLICATIONINSIGHTS_CONNECTION_STRING"]
)

# ── Initialize project client with telemetry ──
client = AIProjectClient(
    credential=DefaultAzureCredential(),
    endpoint=os.environ["PROJECT_ENDPOINT"],
)
client.telemetry.enable()

# ── Custom tracer for business context ──
tracer = trace.get_tracer("agent-observability-lab")

def run_agent_with_tracing(query: str, metadata: dict = None):
    """Run an agent with full OpenTelemetry tracing."""
    with tracer.start_as_current_span("agent_interaction") as span:
        # Add business context
        span.set_attribute("query.text", query)
        if metadata:
            for key, value in metadata.items():
                span.set_attribute(f"business.{key}", str(value))

        # Create a new thread
        thread = client.agents.create_thread()
        span.set_attribute("agent.thread_id", thread.id)

        # Send the user message
        client.agents.create_message(
            thread_id=thread.id,
            role="user",
            content=query
        )

        # Execute the agent
        run = client.agents.create_and_process_run(
            thread_id=thread.id,
            agent_id=os.environ["AGENT_ID"]
        )

        # Record results
        span.set_attribute("agent.run_id", run.id)
        span.set_attribute("agent.status", run.status)
        if run.usage:
            span.set_attribute("agent.tokens.prompt", run.usage.prompt_tokens)
            span.set_attribute("agent.tokens.completion", run.usage.completion_tokens)
            span.set_attribute("agent.tokens.total", run.usage.total_tokens)

        # Get the response
        messages = client.agents.list_messages(thread_id=thread.id)
        response_text = ""
        for msg in messages.data:
            if msg.role == "assistant":
                response_text = msg.content[0].text.value
                break

        span.set_attribute("agent.response_length", len(response_text))
        return {
            "status": run.status,
            "response": response_text,
            "run_id": run.id,
            "thread_id": thread.id,
            "tokens": run.usage.total_tokens if run.usage else 0
        }

# ── Run test queries ──
if __name__ == "__main__":
    test_cases = [
        {
            "query": "What are the top 3 trending stocks today?",
            "metadata": {"category": "market_research", "priority": "medium"}
        },
        {
            "query": "Calculate compound interest on $10,000 at 5% for 10 years.",
            "metadata": {"category": "calculation", "priority": "low"}
        },
        {
            "query": "Summarize the latest Microsoft earnings report.",
            "metadata": {"category": "analysis", "priority": "high"}
        },
        {
            "query": "What is the GDP of France?",
            "metadata": {"category": "factual", "priority": "low"}
        },
        {
            "query": "Compare the P/E ratios of AAPL, MSFT, and GOOGL.",
            "metadata": {"category": "comparison", "priority": "high"}
        },
    ]

    print("=" * 60)
    print("Running Agent Tracing Lab")
    print("=" * 60)

    for i, tc in enumerate(test_cases):
        print(f"\n--- Test {i+1}/{len(test_cases)} ---")
        print(f"Query: {tc['query']}")
        result = run_agent_with_tracing(tc["query"], tc["metadata"])
        print(f"Status: {result['status']}")
        print(f"Tokens: {result['tokens']}")
        print(f"Response: {result['response'][:150]}...")

    print("\n" + "=" * 60)
    print("✅ All test queries complete!")
    print("Check Application Insights → Transaction search for traces.")
    print("=" * 60)
```

### Step 2.2: Run the Tracing Script

```bash
python agent_tracing.py
```

### Step 2.3: Verify Traces in Application Insights

1. Open the Azure portal → your Application Insights resource
2. Navigate to **Transaction search**
3. Filter by `customDimensions` containing `gen_ai.system`
4. Click on any trace to see the full distributed trace with child spans

### ✅ Checkpoint

You should see distributed traces in Application Insights, each showing the agent run, LLM calls, and tool invocations as child spans.

---

## Exercise 3: Debug Agent Failures with KQL (15 min)

### Step 3.1: Query Failed Agent Runs

Open Application Insights → **Logs** and run:

```kql
// Find all agent-related spans in the last hour
dependencies
| where timestamp > ago(1h)
| where type == "az.ai.agents"
| project timestamp, name, duration, success,
         customDimensions["gen_ai.agent.id"],
         customDimensions["gen_ai.thread.run.id"],
         customDimensions["gen_ai.usage.input_tokens"],
         customDimensions["gen_ai.usage.output_tokens"]
| order by timestamp desc
```

### Step 3.2: Analyze Token Usage Patterns

```kql
// Token usage distribution across runs
dependencies
| where timestamp > ago(1h)
| where type == "az.ai.agents"
| extend input_tokens = toint(customDimensions["gen_ai.usage.input_tokens"])
| extend output_tokens = toint(customDimensions["gen_ai.usage.output_tokens"])
| extend total_tokens = input_tokens + output_tokens
| summarize
    count(),
    avg(total_tokens),
    min(total_tokens),
    max(total_tokens),
    percentile(total_tokens, 95),
    avg(duration)
```

### Step 3.3: Inspect Individual Trace Details

```kql
// Get full trace for a specific operation
union dependencies, requests, traces
| where operation_Id == "<paste-operation-id-from-step-3.1>"
| order by timestamp asc
| project timestamp, itemType, name, duration, success,
         customDimensions
```

### Step 3.4: Find Tool Call Patterns

```kql
// Analyze which tools are being called and their success rates
dependencies
| where timestamp > ago(1h)
| where customDimensions has "tool"
| extend tool_name = tostring(customDimensions["gen_ai.tool.name"])
| where isnotempty(tool_name)
| summarize
    calls = count(),
    avg_duration = avg(duration),
    errors = countif(success == false)
    by tool_name
| extend error_rate = round(100.0 * errors / calls, 2)
| order by calls desc
```

### ✅ Checkpoint

You can query agent traces with KQL, identify failed runs, inspect individual traces, and analyze tool usage patterns.

---

## Exercise 4: Build a Monitoring Dashboard (10 min)

### Step 4.1: Create an Agent Health Dashboard

In Application Insights → **Logs**, run these queries and pin each to a new Azure Dashboard:

**Query 1: Agent Success Rate Over Time**

```kql
dependencies
| where timestamp > ago(24h)
| where type == "az.ai.agents"
| summarize
    total = count(),
    succeeded = countif(success == true)
    by bin(timestamp, 15m)
| extend success_rate = round(100.0 * succeeded / total, 1)
| project timestamp, success_rate
| render timechart with (title="Agent Success Rate (%)")
```

**Query 2: Latency Distribution**

```kql
dependencies
| where timestamp > ago(24h)
| where type == "az.ai.agents"
| summarize
    p50 = percentile(duration, 50),
    p90 = percentile(duration, 90),
    p95 = percentile(duration, 95),
    p99 = percentile(duration, 99)
    by bin(timestamp, 15m)
| render timechart with (title="Agent Latency Percentiles (ms)")
```

**Query 3: Token Usage Trend**

```kql
dependencies
| where timestamp > ago(24h)
| where type == "az.ai.agents"
| extend total_tokens = toint(customDimensions["gen_ai.usage.input_tokens"])
                      + toint(customDimensions["gen_ai.usage.output_tokens"])
| summarize avg_tokens = avg(total_tokens), max_tokens = max(total_tokens)
    by bin(timestamp, 15m)
| render timechart with (title="Token Usage Trend")
```

### Step 4.2: Pin to Dashboard

For each query result:

1. Click **Pin to dashboard** in the query results toolbar
2. Select **Create new** or an existing dashboard
3. Name the dashboard "Agent Health — Module 20 Lab"

### ✅ Checkpoint

You have a live Azure Dashboard with three charts: success rate, latency, and token usage.

---

## Exercise 5: Set Up Alerting (10 min)

### Step 5.1: Create a Success Rate Alert

1. In Application Insights → **Alerts** → **Create alert rule**
2. **Condition**: Custom log search:

```kql
dependencies
| where timestamp > ago(1h)
| where type == "az.ai.agents"
| summarize
    total = count(),
    failed = countif(success == false)
| extend failure_rate = round(100.0 * failed / total, 1)
| where failure_rate > 10
```

3. **Alert logic**: Greater than 0 results
4. **Evaluation**: Every 15 minutes, lookback 1 hour
5. **Actions**: Configure email notification or Azure Action Group
6. **Name**: "Agent Failure Rate > 10%"

### Step 5.2: Create a Latency Alert

Create a second alert for P95 latency:

```kql
dependencies
| where timestamp > ago(1h)
| where type == "az.ai.agents"
| summarize p95_latency = percentile(duration, 95)
| where p95_latency > 30000
```

### ✅ Checkpoint

You have two alert rules configured: one for failure rate and one for latency.

---

## Lab Cleanup

To avoid ongoing charges, clean up resources when done:

```bash
# Delete the resource group (removes App Insights and all resources)
az group delete --name rg-agent-observability --yes --no-wait
```

> **Note:** Only run this if you created a dedicated resource group for this lab. If you're using shared resources, delete only the Application Insights component.

---

## Lab Summary

In this lab, you:

| Exercise | What You Did                                          |
| -------- | ----------------------------------------------------- |
| 1        | Created Application Insights and installed packages   |
| 2        | Enabled end-to-end agent tracing with OpenTelemetry   |
| 3        | Debugged agent behavior using KQL queries             |
| 4        | Built a monitoring dashboard with three key charts    |
| 5        | Configured proactive alerts for failure rate & latency|

### Skills Practiced

- ✅ Application Insights resource creation and configuration
- ✅ OpenTelemetry instrumentation for Azure AI Foundry agents
- ✅ Custom span enrichment with business attributes
- ✅ KQL queries for agent trace analysis and debugging
- ✅ Azure Dashboard creation with pinned query results
- ✅ Alert rule configuration for proactive monitoring

---

## Additional Challenges

1. **Export traces to a dataset**: Write a script that exports App Insights traces to a JSON file suitable for the evaluation SDK
2. **Compare agent versions**: Deploy two versions of your agent, tag traces with a version attribute, and compare success rates in KQL
3. **Build a Workbook**: Create an Azure Workbook with interactive parameters for time range, agent ID, and query type
4. **Add cost tracking**: Enrich spans with estimated cost per run based on token usage and model pricing

---

## Next Lab

**Module 21 Lab: Agent Security & Responsible AI** — Implementing content safety filters, prompt injection detection, and responsible AI guardrails for production agents.
