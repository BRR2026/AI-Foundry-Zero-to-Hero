# Module 20: Agent Evaluation, Tracing & Debugging

## Azure AI Foundry: Zero to Hero | Arc 4 — AI Agents

| Field          | Details                                              |
| -------------- | ---------------------------------------------------- |
| **Module**     | 20 of 45                                             |
| **Arc**        | 4 — AI Agents                                        |
| **Duration**   | 90 minutes                                           |
| **Level**      | Intermediate–Advanced                                |
| **Date**       | April 2026                                           |

---

## Learning Objectives

By the end of this module, learners will be able to:

1. Evaluate agent quality using task completion, accuracy, and tool usage metrics
2. Integrate Application Insights for end-to-end agent tracing
3. Debug agent failures systematically using distributed traces and KQL queries
4. Apply OpenTelemetry semantic conventions for generative AI observability
5. Build monitoring dashboards and configure proactive alerts for agent health
6. Set up end-to-end tracing for an agent with Application Insights (Mini Hack)

---

## Prerequisites

- Completed Modules 17–19 (building, tool integration, and orchestrating agents)
- Active Azure subscription with an Azure AI Foundry project
- At least one deployed agent in the project
- Application Insights resource (or ability to create one)
- Python 3.9+ with `azure-ai-projects` and `azure-monitor-opentelemetry` packages
- Azure CLI authenticated (`az login`)

---

## Module Outline

### Section 1: Why Agent Observability Matters (10 min)

#### The Observability Gap

Traditional software observability focuses on request/response pairs. AI agents introduce unique challenges:

- **Non-determinism** — the same input can produce different outputs
- **Multi-step execution** — a single request can spawn dozens of LLM calls and tool invocations
- **Hidden reasoning** — the agent's decision-making process is opaque without tracing
- **Tool dependencies** — failures in external tools cascade through agent behavior

#### What You Need to Observe

| Layer              | What to Monitor                                    |
| ------------------ | -------------------------------------------------- |
| **Agent Runs**     | Success/failure rates, duration, token usage        |
| **LLM Calls**      | Model, prompt/completion tokens, finish reason      |
| **Tool Calls**     | Tool name, input/output, duration, errors           |
| **Business Logic** | Task completion, customer satisfaction, cost/query  |

---

### Section 2: Evaluating Agent Quality (20 min)

#### Evaluation Dimensions

Agent evaluation must go beyond simple accuracy:

1. **Task Completion** — Did the agent achieve the user's goal?
2. **Accuracy / Groundedness** — Were outputs factually correct and grounded in source data?
3. **Tool Usage Correctness** — Did the agent call the right tools with correct parameters?
4. **Efficiency** — Were resources (tokens, time, tool calls) used wisely?
5. **Safety** — Did the agent stay within safety guardrails?
6. **Instruction Adherence** — Did the agent follow its configured persona and rules?

#### Using the Azure AI Evaluation SDK

```python
from azure.ai.evaluation import AgentEvaluator
from azure.identity import DefaultAzureCredential

evaluator = AgentEvaluator(
    azure_ai_project={
        "subscription_id": "<subscription-id>",
        "resource_group_name": "<resource-group>",
        "project_name": "<project-name>",
    },
    credential=DefaultAzureCredential()
)

# Define test cases with ground truth
eval_dataset = [
    {
        "query": "What is the current stock price of MSFT?",
        "expected_tool_calls": ["bing_search"],
        "ground_truth": "Should retrieve live price from search."
    },
    {
        "query": "Summarize Q3 earnings from the uploaded PDF.",
        "expected_tool_calls": ["file_search"],
        "ground_truth": "Revenue was $56.5B, up 13% YoY."
    }
]

results = evaluator.evaluate(
    data=eval_dataset,
    evaluators={
        "task_completion": "task_completion",
        "groundedness": "groundedness",
        "tool_call_accuracy": "tool_call_accuracy",
    }
)

print(f"Task Completion: {results.metrics['task_completion']:.2%}")
print(f"Groundedness:    {results.metrics['groundedness']:.2f}/5")
print(f"Tool Accuracy:   {results.metrics['tool_call_accuracy']:.2%}")
```

#### Building Evaluation Datasets

A high-quality evaluation dataset should include:

- **Happy-path scenarios** — common tasks the agent handles daily
- **Edge cases** — ambiguous queries, missing data, conflicting instructions
- **Multi-step workflows** — tasks requiring sequential tool calls
- **Adversarial inputs** — prompt injection attempts, off-topic requests
- **Boundary conditions** — very long inputs, empty inputs, special characters

> **Tip:** Export real user conversations from Application Insights traces to build evaluation datasets that reflect actual usage patterns.

---

### Section 3: Application Insights Integration (20 min)

#### Connecting App Insights to Your Project

1. **Create an Application Insights resource** (if not already available):

```bash
az monitor app-insights component create \
  --app my-agent-insights \
  --location eastus2 \
  --resource-group my-rg \
  --kind web \
  --application-type web
```

2. **Link it to your Azure AI Foundry project** through the portal settings.

3. **Enable tracing in your agent code:**

```python
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from azure.monitor.opentelemetry import configure_azure_monitor

# Enable telemetry export to Application Insights
configure_azure_monitor(
    connection_string="InstrumentationKey=<key>;..."
)

# Create client — traces are auto-collected
client = AIProjectClient(
    credential=DefaultAzureCredential(),
    endpoint="https://<project>.services.ai.azure.com",
)
client.telemetry.enable()
```

4. **Verify traces** in Application Insights → Transaction search.

#### What Gets Traced Automatically

| Span Type       | Captured Information                                   |
| --------------- | ------------------------------------------------------ |
| Agent Run       | agent.id, thread.id, run.id, status, duration          |
| LLM Call        | model, prompt_tokens, completion_tokens, finish_reason |
| Tool Call       | tool.name, tool.input, tool.output, duration           |
| Token Usage     | prompt_tokens, completion_tokens, total_tokens         |
| Exceptions      | exception.type, exception.message, stack trace         |

---

### Section 4: OpenTelemetry Semantic Conventions (10 min)

#### GenAI Semantic Attributes

Azure AI Foundry follows the OpenTelemetry Semantic Conventions for Generative AI:

```
# Model attributes
gen_ai.system           = "az.ai.agents"
gen_ai.request.model    = "gpt-4o"
gen_ai.response.model   = "gpt-4o-2024-08-06"

# Token usage
gen_ai.usage.input_tokens   = 1250
gen_ai.usage.output_tokens  = 340

# Operation attributes
gen_ai.operation.name       = "chat"
gen_ai.response.finish_reasons = ["stop"]

# Agent-specific
gen_ai.agent.id      = "asst_abc123"
gen_ai.thread.id     = "thread_xyz789"
gen_ai.thread.run.id = "run_def456"
```

#### Custom Span Enrichment

Add business context to traces:

```python
from opentelemetry import trace

tracer = trace.get_tracer("my-agent-app")

with tracer.start_as_current_span("process_customer_request") as span:
    span.set_attribute("customer.id", customer_id)
    span.set_attribute("request.type", "refund_inquiry")
    span.set_attribute("request.priority", "high")

    response = agent_client.run(agent_id=agent_id, thread_id=thread_id)

    span.set_attribute("agent.task_completed", True)
    span.set_attribute("agent.total_tokens", response.usage.total_tokens)
```

---

### Section 5: Debugging Agent Failures (15 min)

#### Common Failure Patterns

| Pattern                | Symptoms                            | Root Cause                          | Fix                                     |
| ---------------------- | ----------------------------------- | ----------------------------------- | --------------------------------------- |
| Tool Call Loop         | Same tool called 5+ times           | Ambiguous instructions              | Clarify instructions, improve errors    |
| Wrong Tool Selected    | File Search instead of Bing Search  | Tool descriptions too similar       | Make descriptions distinct              |
| Hallucinated Output    | Fabricated data in response         | Tool output empty, tool call skipped| Add verification guardrails             |
| Token Exhaustion       | Truncated or incomplete response    | Thread history too long             | Implement summarization                 |
| Timeout                | Run times out                       | External tool hangs                 | Add timeouts, circuit breakers          |

#### Debugging Workflow

1. **Identify the failed run** — query App Insights for failed runs:

```kql
dependencies
| where timestamp > ago(24h)
| where type == "az.ai.agents"
| where success == false
| project timestamp, id, name, duration, resultCode
| order by timestamp desc
```

2. **Inspect the full trace** — view the distributed trace end-to-end:

```kql
union dependencies, requests, traces, exceptions
| where operation_Id == "<operation-id>"
| order by timestamp asc
| project timestamp, itemType, name, duration, success, customDimensions
```

3. **Analyze tool call sequences** — verify correct tool selection and parameters.

4. **Review token usage** — compare against expected ranges.

5. **Test the fix** — replay the same conversation to verify.

#### Programmatic Debugging

```python
# Retrieve run steps to inspect reasoning
run_steps = client.agents.list_run_steps(
    thread_id="thread_xyz789",
    run_id="run_def456"
)

for step in run_steps.data:
    print(f"Step: {step.type} | Status: {step.status}")
    if step.type == "tool_calls":
        for tool_call in step.step_details.tool_calls:
            print(f"  Tool: {tool_call.type}")
            print(f"  Input: {tool_call.function.arguments}")
```

---

### Section 6: Trace Analysis & Monitoring (10 min)

#### Key KQL Queries

**Agent Success Rate Over Time:**

```kql
dependencies
| where timestamp > ago(7d)
| where type == "az.ai.agents"
| summarize
    total = count(),
    succeeded = countif(success == true),
    failed = countif(success == false)
    by bin(timestamp, 1h)
| extend success_rate = round(100.0 * succeeded / total, 2)
| render timechart
```

**Token Usage Analysis:**

```kql
dependencies
| where timestamp > ago(7d)
| where type == "az.ai.agents"
| extend input_tokens = toint(customDimensions["gen_ai.usage.input_tokens"])
| extend output_tokens = toint(customDimensions["gen_ai.usage.output_tokens"])
| summarize
    avg_input = avg(input_tokens),
    avg_output = avg(output_tokens),
    p95_total = percentile(input_tokens + output_tokens, 95)
    by bin(timestamp, 1d)
```

#### Setting Up Alerts

Configure Azure Monitor alerts for:

- Success rate drops below 95% in any 1-hour window
- P95 latency exceeds 30 seconds
- Token usage spikes above 2× the daily average
- Error rate for a specific tool exceeds 10%

---

### Section 7: Mini Hack — End-to-End Agent Tracing (15 min)

Set up complete observability for an agent:

1. Configure Azure Monitor / Application Insights export
2. Enable telemetry on the AIProjectClient
3. Add custom span attributes for business context
4. Run a batch of test conversations through the agent
5. Build a KQL dashboard in Application Insights showing success rates, latency, and token usage
6. (Stretch) Set up an alert rule for success rate < 90%

> See the blog post and hands-on lab for the complete implementation walkthrough.

---

## Key Takeaways

1. **Observability is not optional** — AI agents are non-deterministic and require dedicated monitoring
2. **Evaluate holistically** — measure task completion, accuracy, tool usage, efficiency, and safety
3. **App Insights is your primary backend** — it captures traces automatically when connected to your project
4. **OpenTelemetry conventions** — use standard semantic attributes for consistent, queryable telemetry
5. **Debug systematically** — use distributed traces and KQL to find root causes, not guesswork
6. **Alert proactively** — set up monitoring dashboards and alerts before the first production incident

---

## Additional Resources

- [Azure AI Agent Service Documentation](https://learn.microsoft.com/azure/ai-services/agents/)
- [Azure AI Foundry Tracing Guide](https://learn.microsoft.com/azure/ai-studio/how-to/develop/trace-local-sdk)
- [OpenTelemetry GenAI Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/)
- [Azure Monitor OpenTelemetry for Python](https://learn.microsoft.com/azure/azure-monitor/app/opentelemetry-enable?tabs=python)
- [KQL Quick Reference](https://learn.microsoft.com/azure/data-explorer/kusto/query/kql-quick-reference)
- Video: [Debug Your AI Agents: Fix Failures Fast with Observability on Azure AI Foundry](https://www.youtube.com/watch?v=DpvR9ZgXK84)

---

## Next Module

**Module 21: Agent Security & Responsible AI** — Implementing guardrails, content filtering, and safety policies for production AI agents.
