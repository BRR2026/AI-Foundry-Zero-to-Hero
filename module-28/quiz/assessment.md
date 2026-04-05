# Module 28: Assessment — Monitoring, Observability & Drift Detection

> **Azure AI Foundry: Zero to Hero** · ARC 6: Deployment & MLOps  
> **Module 28 of 45** · 10 Multiple-Choice Questions · **Passing Score: 80%**

---

## Instructions

- Select the **single best answer** for each question.
- Each question is worth **10 points** (100 points total).
- You need **80 points (8/10)** to pass.
- Review the explanations after submitting to reinforce your learning.

---

### Question 1

**What makes monitoring AI systems fundamentally different from monitoring traditional applications?**

A) AI systems use more compute resources than traditional applications  
B) AI systems can fail silently by producing subtly wrong answers without error codes  
C) AI systems always require GPU monitoring in addition to CPU monitoring  
D) AI systems generate larger log files that require specialized storage  

<details>
<summary>✅ Answer</summary>

**B) AI systems can fail silently by producing subtly wrong answers without error codes**

Traditional applications fail with clear signals (HTTP errors, crashes, timeouts). AI systems are stochastic and can produce plausible but incorrect or degraded responses without triggering any error conditions. This "silent failure" problem is the core reason AI-specific observability is required beyond standard APM.

</details>

---

### Question 2

**Which of the following is the fourth pillar of AI observability that does NOT exist in traditional application monitoring?**

A) Metrics  
B) Traces  
C) Evaluations  
D) Logs  

<details>
<summary>✅ Answer</summary>

**C) Evaluations**

The four pillars of AI observability are metrics, traces, logs, and evaluations. The first three are shared with traditional monitoring, but **evaluations** — which include quality scores, drift indicators, and safety checks — are unique to AI systems. Traditional applications don't need ongoing quality scoring of their outputs.

</details>

---

### Question 3

**You want to route telemetry from your Azure AI Foundry project to a Log Analytics workspace. Which Azure feature do you configure?**

A) Azure Event Grid subscriptions  
B) Azure Monitor diagnostic settings  
C) Azure Key Vault access policies  
D) Azure Service Bus topic filters  

<details>
<summary>✅ Answer</summary>

**B) Azure Monitor diagnostic settings**

Diagnostic settings on an Azure AI Foundry project resource allow you to route logs (categories like `AmlModelsEvent`) and metrics (`AllMetrics`) to destinations including Log Analytics workspaces, Azure Storage, and Event Hubs.

</details>

---

### Question 4

**Which KQL function would you use to calculate the 95th percentile latency of model requests over time?**

A) `avg(DurationMs)`  
B) `max(DurationMs)`  
C) `percentile(DurationMs, 95)`  
D) `stdev(DurationMs)`  

<details>
<summary>✅ Answer</summary>

**C) `percentile(DurationMs, 95)`**

The `percentile()` function in KQL calculates a specific percentile of a numeric column. Using `percentile(DurationMs, 95)` returns the P95 value, meaning 95% of requests completed within this duration. P95 is the standard metric for latency alerting because it captures tail latency without being skewed by extreme outliers like P99.

</details>

---

### Question 5

**Your AI model's token usage suddenly doubles but latency and error rates remain stable. What is the most likely cause?**

A) The model endpoint is being rate-limited by Azure  
B) User prompts have become longer or more complex, or system prompts changed  
C) The Log Analytics workspace has run out of storage  
D) The model is experiencing concept drift  

<details>
<summary>✅ Answer</summary>

**B) User prompts have become longer or more complex, or system prompts changed**

Token usage doubling while latency and error rates stay stable indicates that the inputs or outputs have grown in size. This is most commonly caused by changes to system prompts, longer user queries, or the model generating more verbose responses. Rate limiting would increase errors (429s), storage issues would affect logging not usage, and concept drift affects quality scores not token counts.

</details>

---

### Question 6

**Which type of model drift occurs when the "correct" answer changes over time — for example, when product information or regulations are updated?**

A) Data drift  
B) Concept drift  
C) Quality drift  
D) Behavioral drift  

<details>
<summary>✅ Answer</summary>

**B) Concept drift**

**Concept drift** occurs when the relationship between inputs and the correct outputs changes. When regulations update, product information changes, or facts change, the model's previously correct responses become outdated. Data drift refers to changes in input distribution, quality drift to gradual output degradation, and behavioral drift to provider-side model changes.

</details>

---

### Question 7

**You are designing an alert strategy for your AI workload. An alert for "daily token usage exceeding 80% of budget" should be assigned which severity level?**

A) Sev 0 — Critical  
B) Sev 1 — Error (page on-call immediately)  
C) Sev 3 — Informational (notify team, next business day)  
D) Sev 2 — Warning (urgent, check within the hour)  

<details>
<summary>✅ Answer</summary>

**C) Sev 3 — Informational (notify team, next business day)**

Token budget alerts are a cost management signal, not an operational emergency. The system is still functioning correctly; you're approaching a spending threshold. This warrants a notification to the team for review but does not require immediate paging. Sev 1 is for service-impacting events like error spikes. Sev 2 is for warnings like high latency or drift detection.

</details>

---

### Question 8

**In Application Insights, what field correlates all spans within a single agent invocation (e.g., LLM calls, tool calls, retrieval operations)?**

A) `session_id`  
B) `operation_Id`  
C) `user_id`  
D) `deployment_name`  

<details>
<summary>✅ Answer</summary>

**B) `operation_Id`**

Application Insights uses `operation_Id` as the distributed trace correlation identifier. All spans (dependencies, requests, custom events) generated during a single agent invocation share the same `operation_Id`, enabling you to reconstruct the full execution flow in the Application Insights transaction view.

</details>

---

### Question 9

**You want to detect quality drift in your production AI agent. Which approach is recommended?**

A) Monitor HTTP error rates — if they increase, the model is drifting  
B) Ask users to manually rate every response and track average ratings  
C) Sample production traffic regularly and run AI-judge evaluators (groundedness, relevance, coherence) against baseline scores  
D) Compare the model's output token count to a historical average  

<details>
<summary>✅ Answer</summary>

**C) Sample production traffic regularly and run AI-judge evaluators (groundedness, relevance, coherence) against baseline scores**

Quality drift detection requires measuring the actual quality of model outputs, not proxy metrics like error rates or token counts. The recommended approach is to sample a subset of production requests, run automated AI-judge evaluators (using metrics like groundedness, relevance, and coherence), and compare the results to pre-established baselines. A drop exceeding the drift threshold (e.g., 10%) triggers an alert.

</details>

---

### Question 10

**Which of the following is a best practice for AI observability in production?**

A) Log 100% of all prompt-completion pairs with full content to maximize debugging ability  
B) Use static alert thresholds only — dynamic thresholds create too many false positives  
C) Sample successful requests (e.g., 10%) but log 100% of errors, and include model version in all telemetry  
D) Rely on user feedback as the primary drift detection mechanism  

<details>
<summary>✅ Answer</summary>

**C) Sample successful requests (e.g., 10%) but log 100% of errors, and include model version in all telemetry**

This balances cost efficiency with comprehensive error capture. Logging 100% of all prompts is expensive and raises PII concerns. Static-only thresholds miss anomalies that dynamic thresholds catch. User feedback is valuable but too slow and unreliable as a primary detection mechanism. Sampling successes while capturing all errors, combined with rich metadata (model version, deployment version, environment), provides the best observability foundation.

</details>

---

## 📊 Scoring

| Score | Result |
|---|---|
| 10/10 | ⭐ Exceptional — You're ready to build production AI monitoring! |
| 8–9/10 | ✅ Pass — Solid understanding of AI observability |
| 6–7/10 | 🔄 Review — Revisit drift detection and alerting sections |
| < 6/10 | 📚 Study — Re-read the module and redo the hands-on lab |

---

## 🔗 Review Materials

- [Module 28 Blog Post](../blog.html)
- [Training Guide](../training-guide.md)
- [Hands-On Lab](../lab/hands-on-lab.md)
- [Presentation Slides](../slides/presentation.html)
- [Featured Video: Debug Your AI Agents](https://www.youtube.com/watch?v=DpvR9ZgXK84)

---

> **Next Module:** [Module 29 →](../../module-29/quiz/assessment.md)  
> **Previous Module:** [← Module 27](../../module-27/quiz/assessment.md)
