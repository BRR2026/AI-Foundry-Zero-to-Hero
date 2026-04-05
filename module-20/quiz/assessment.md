# Module 20 Assessment: Agent Evaluation, Tracing & Debugging

## Azure AI Foundry: Zero to Hero | Arc 4 — AI Agents

**Instructions:** Choose the best answer for each question. Each question has exactly one correct answer. Score: 8/10 to pass.

---

### Question 1

**What is the primary telemetry backend for Azure AI Foundry agent tracing?**

- A) Azure Log Analytics Workspace
- B) Application Insights
- C) Azure Event Hubs
- D) Azure Data Explorer

<details>
<summary>Show Answer</summary>

**Correct: B) Application Insights**

Application Insights is the primary telemetry backend for Azure AI Foundry agents. When connected to your AI Foundry project, it automatically captures traces for every agent run, including LLM calls, tool invocations, and token usage via the OpenTelemetry integration.

</details>

---

### Question 2

**Which OpenTelemetry semantic attribute captures the total number of input tokens consumed by an LLM call?**

- A) `gen_ai.tokens.input`
- B) `gen_ai.request.input_tokens`
- C) `gen_ai.usage.input_tokens`
- D) `opentelemetry.llm.input_tokens`

<details>
<summary>Show Answer</summary>

**Correct: C) `gen_ai.usage.input_tokens`**

The OpenTelemetry Semantic Conventions for Generative AI define `gen_ai.usage.input_tokens` as the standard attribute for tracking input token consumption. This follows the `gen_ai.usage.*` namespace for token usage metrics.

</details>

---

### Question 3

**An agent repeatedly calls the same tool 5+ times without making progress, causing high latency. What is the most likely root cause?**

- A) The model has insufficient context window
- B) Application Insights is dropping trace data
- C) Ambiguous agent instructions or tool output not actionable
- D) The OpenTelemetry exporter is buffering too many spans

<details>
<summary>Show Answer</summary>

**Correct: C) Ambiguous agent instructions or tool output not actionable**

A tool call loop occurs when the agent receives unclear results from a tool and tries the same approach repeatedly. The fix is to clarify agent instructions about when to use specific tools and improve tool error messages so the agent can take alternative actions when a tool call doesn't produce useful results.

</details>

---

### Question 4

**Which Python method enables automatic telemetry collection on the AIProjectClient?**

- A) `client.enable_tracing()`
- B) `client.telemetry.enable()`
- C) `client.configure_opentelemetry()`
- D) `client.set_telemetry(enabled=True)`

<details>
<summary>Show Answer</summary>

**Correct: B) `client.telemetry.enable()`**

After configuring Azure Monitor with `configure_azure_monitor()`, calling `client.telemetry.enable()` on the AIProjectClient activates automatic trace collection for all agent runs, LLM calls, and tool invocations executed through the client.

</details>

---

### Question 5

**When building an evaluation dataset for agent quality assessment, which of the following should NOT be included?**

- A) Happy-path scenarios the agent handles daily
- B) Edge cases with ambiguous queries
- C) Only synthetic examples with perfect formatting
- D) Adversarial inputs like prompt injection attempts

<details>
<summary>Show Answer</summary>

**Correct: C) Only synthetic examples with perfect formatting**

Evaluation datasets should reflect real-world usage patterns, not just ideal synthetic examples. A comprehensive dataset includes happy-path scenarios, edge cases, multi-step workflows, adversarial inputs, and boundary conditions. Using only synthetic, perfectly formatted examples would miss the messy, unpredictable nature of real user interactions.

</details>

---

### Question 6

**Which KQL query would correctly identify agent runs with a failure rate over the last 24 hours?**

- A) `traces | where message contains "error" | count`
- B) `dependencies | where timestamp > ago(24h) | where type == "az.ai.agents" | where success == false`
- C) `requests | where resultCode >= 500 | where name == "agent"`
- D) `customEvents | where name == "AgentFailure" | where timestamp > ago(24h)`

<details>
<summary>Show Answer</summary>

**Correct: B) `dependencies | where timestamp > ago(24h) | where type == "az.ai.agents" | where success == false`**

Agent runs are captured as dependency telemetry in Application Insights with the type `az.ai.agents`. Filtering by `success == false` identifies failed runs. This is the standard pattern for querying agent trace data via KQL.

</details>

---

### Question 7

**What is the recommended sampling rate for tracing high-volume production agents to balance observability with cost?**

- A) 100% — all traces should be captured
- B) 1% — only capture a tiny fraction
- C) 10–25% — enough data for analysis without excessive cost
- D) 50% — capture every other request

<details>
<summary>Show Answer</summary>

**Correct: C) 10–25% — enough data for analysis without excessive cost**

For high-volume production agents, sampling at 10–25% provides sufficient data for identifying patterns, debugging issues, and computing reliable metrics while keeping Application Insights ingestion costs manageable. The environment variable `OTEL_TRACES_SAMPLER_ARG=0.1` configures 10% sampling.

</details>

---

### Question 8

**Which evaluation dimension measures whether an agent called the correct tools with the right parameters?**

- A) Task Completion
- B) Groundedness
- C) Tool Usage Correctness (precision/recall)
- D) Instruction Adherence

<details>
<summary>Show Answer</summary>

**Correct: C) Tool Usage Correctness (precision/recall)**

Tool usage correctness evaluates whether the agent selected the appropriate tools for a given task and provided the correct input parameters. This is measured using precision (were the tools called relevant?) and recall (were all necessary tools called?). Task completion measures the end goal, groundedness measures factual accuracy, and instruction adherence measures persona/rule compliance.

</details>

---

### Question 9

**You need to add a customer ID and request type to agent traces for business analytics. Which approach is correct?**

- A) Set HTTP headers on the agent API request
- B) Use `tracer.start_as_current_span()` and call `span.set_attribute()` with custom attributes
- C) Add the metadata to the agent's system instructions
- D) Log the information separately to a custom database table

<details>
<summary>Show Answer</summary>

**Correct: B) Use `tracer.start_as_current_span()` and call `span.set_attribute()` with custom attributes**

OpenTelemetry's span API allows you to create custom spans and enrich them with business-specific attributes using `span.set_attribute()`. These attributes are automatically exported to Application Insights via the Azure Monitor exporter, allowing you to query and filter traces by customer ID, request type, or any other business dimension using KQL.

</details>

---

### Question 10

**An agent's response contains fabricated data that doesn't appear in any tool output. When debugging with traces, what should you check first?**

- A) Whether the model temperature is set too high
- B) Whether the agent actually called a tool or skipped the tool call and hallucinated the data
- C) Whether Application Insights has a data retention policy configured
- D) Whether the agent's thread history is too short

<details>
<summary>Show Answer</summary>

**Correct: B) Whether the agent actually called a tool or skipped the tool call and hallucinated the data**

When an agent returns fabricated data, the first debugging step is to inspect the trace to verify whether a tool call was actually made. The most common cause of hallucinated output is that the agent skipped the tool call entirely (or the tool returned empty/error results) and generated a plausible-sounding response from its training data instead. The trace will clearly show whether tool call spans exist and what their outputs were.

</details>

---

## Scoring

| Score   | Result                                           |
| ------- | ------------------------------------------------ |
| 10/10   | ⭐ Excellent — You've mastered agent observability |
| 8–9/10  | ✅ Pass — Strong understanding of evaluation and tracing |
| 6–7/10  | 🔄 Review — Revisit App Insights and debugging sections |
| < 6/10  | 📖 Restudy — Complete the module again before proceeding |

---

## Key Concepts to Review if Needed

- **Application Insights integration** — Blog Section: "App Insights Integration"
- **OpenTelemetry semantic conventions** — Blog Section: "OpenTelemetry Conventions"
- **Debugging failure patterns** — Blog Section: "Debugging Failures"
- **Evaluation dimensions and SDK** — Blog Section: "Agent Evaluation"
- **KQL queries for trace analysis** — Blog Section: "Trace Analysis"
- **Video reference**: [Debug Your AI Agents](https://www.youtube.com/watch?v=DpvR9ZgXK84)
