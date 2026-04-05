# Module 22: Multi-Model Orchestration & Routing — Assessment

## Azure AI Foundry: Zero to Hero Training Course

**ARC 5: ORCHESTRATION & INTEGRATION** | Module 22 of 45 | April 2026

---

## Instructions

- 10 multiple-choice questions
- Select the single best answer for each question
- Passing score: 80% (8 out of 10 correct)
- Answers are provided at the end of this document

---

## Questions

### Question 1

A development team is building a customer-support platform on Azure AI Foundry. Some requests are simple FAQ look-ups, while others require nuanced legal-compliance reasoning. What is the **primary** reason to adopt a multi-model strategy instead of routing every request to a single large model like GPT-4o?

- A) Azure AI Foundry does not allow a single deployment to handle more than one prompt template, so multiple models are mandatory.
- B) Routing simple requests to a smaller, cheaper model (e.g., Phi-4) reduces cost and latency while reserving the large model for complex tasks that genuinely need its capabilities.
- C) Smaller models always produce higher-quality answers than larger models, so they should be preferred for every request.
- D) Azure AI Foundry automatically deploys multiple models behind every endpoint, so there is no additional effort involved.

---

### Question 2

You are implementing a **cost-based routing** strategy in Azure AI Foundry. A request arrives that any of your deployed models can handle adequately. How should the router select the model?

- A) Always select the model with the highest token-per-second throughput, regardless of per-token price.
- B) Select the model that was deployed most recently, because newer deployments have lower prices.
- C) Select the model with the lowest per-token cost that still meets the minimum quality threshold for the request, keeping overall spend as low as possible.
- D) Randomly distribute requests across all models to ensure each deployment receives equal traffic.

---

### Question 3

A **capability-based routing** strategy examines an incoming request and selects the model best suited to the task type. Which of the following routing decisions best illustrates capability-based routing?

- A) Sending every request to GPT-4o because it has the largest parameter count.
- B) Routing a code-generation request to GPT-4o for its strong coding ability, while routing a simple text-summarization request to Meta-Llama-3.1-8B because summarization is within its demonstrated capability set.
- C) Routing requests alphabetically by the user's display name to different models.
- D) Sending all requests to Phi-4 first and only using GPT-4o if the user explicitly asks for it in the prompt text.

---

### Question 4

Your orchestration layer implements a **fallback/cascade pattern**. The primary model (Phi-4) returns a low-confidence response for a complex reasoning task. What should happen next in a correctly implemented cascade?

- A) Return the low-confidence response to the user immediately and log a warning, because retrying with another model would violate Azure AI Foundry's rate-limit policies.
- B) Discard all model deployments and raise an unrecoverable error to the client application.
- C) Escalate the request to a more capable model (e.g., GPT-4o), evaluate its response against the confidence threshold, and return the first response that meets the threshold — or the best available response if all models are exhausted.
- D) Cache the low-confidence response and return it for every subsequent identical query without re-evaluating.

---

### Question 5

An architect is comparing the cost profiles of GPT-4o and Phi-4 for a high-volume internal Q&A workload on Azure AI Foundry. Which statement **most accurately** describes the typical cost and performance trade-off between these two models?

- A) GPT-4o and Phi-4 have identical per-token pricing on Azure AI Foundry, so the choice should be based solely on latency.
- B) Phi-4 is a smaller model with significantly lower per-token cost and faster inference, but it may underperform GPT-4o on tasks requiring deep multi-step reasoning or broad world knowledge.
- C) Phi-4 is more expensive than GPT-4o because small-language-model deployments require dedicated compute that cannot be shared.
- D) GPT-4o is always cheaper at scale because Microsoft offers unlimited free tokens for GPT-4o deployments.

---

### Question 6

Which of the following **best describes** a well-architected multi-model orchestration layer in Azure AI Foundry?

- A) A single Azure Function that hard-codes model endpoint URLs and uses if/else statements with no monitoring, retry logic, or fallback handling.
- B) A centralized routing component that classifies incoming requests, selects the appropriate model deployment based on configurable routing policies (cost, capability, latency), includes fallback logic, and emits telemetry for observability.
- C) A front-end JavaScript widget that lets end users manually pick which model to call from a dropdown menu.
- D) An orchestration layer that deploys every available model in the Azure AI Foundry model catalog regardless of whether the application needs them.

---

### Question 7

A developer writes the following Python pseudocode for a `SmartModelRouter` class that uses the `azure-ai-inference` SDK:

```python
class SmartModelRouter:
    def __init__(self, clients: dict[str, ChatCompletionsClient]):
        self.clients = clients  # e.g., {"phi-4": client1, "gpt-4o": client2}

    def route(self, request: str, complexity: str) -> str:
        if complexity == "low":
            model_key = "phi-4"
        else:
            model_key = "gpt-4o"
        response = self.clients[model_key].complete(
            messages=[UserMessage(content=request)]
        )
        return response.choices[0].message.content
```

What is the **most critical improvement** this router needs before it is production-ready?

- A) Replace the dictionary of clients with a single client, because the `azure-ai-inference` SDK only supports one endpoint per process.
- B) Add fallback handling so that if the selected model fails or returns a low-quality response, the router can retry with an alternative model, and add logging/telemetry for observability.
- C) Remove the `complexity` parameter entirely, because Azure AI Foundry automatically determines request complexity on the server side.
- D) Change the SDK import from `azure-ai-inference` to `openai`, because Azure AI Foundry does not support its own inference SDK.

---

### Question 8

A product team notices that routing **all** traffic to Phi-4 saves 70% on inference costs compared to GPT-4o, but customer-satisfaction scores for complex advisory queries have dropped by 25%. Which approach best balances performance and cost?

- A) Continue using only Phi-4 and retrain it on the advisory dataset to close the quality gap entirely, since fine-tuning always eliminates the need for larger models.
- B) Implement a tiered routing strategy: classify requests by complexity, route simple queries to Phi-4 for cost savings, and route complex advisory queries to GPT-4o to maintain quality — then monitor both cost and satisfaction metrics continuously.
- C) Switch all traffic back to GPT-4o and accept the higher cost, because mixing models in a single application is an unsupported anti-pattern.
- D) Deploy a third model (Meta-Llama-3.1-8B) and send all traffic there as a compromise between the two extremes, without any routing logic.

---

### Question 9

A solutions architect is browsing the **Azure AI Foundry model catalog** to select models for a multi-model orchestration pipeline. Which of the following capabilities does the model catalog provide to support this workflow?

- A) The catalog only lists Microsoft first-party models; open-source and third-party models must be imported manually via CLI before they can be used in orchestration.
- B) The catalog provides a searchable collection of models from multiple providers (Microsoft, Meta, Mistral, and others), with filters for task type, license, and deployment option — enabling architects to compare and deploy complementary models for different routing tiers.
- C) The catalog automatically builds the routing logic for you and deploys a pre-configured orchestration endpoint with no developer input required.
- D) The catalog is a read-only documentation site; models must be deployed through a separate Azure portal blade that has no connection to the catalog.

---

### Question 10

Which of the following is a **best practice** when designing a model-routing strategy for a production Azure AI Foundry application?

- A) Hard-code the routing rules directly into the application's front-end code so that changes can be deployed with a simple browser refresh.
- B) Use a single global confidence threshold for all models and all task types, because varying thresholds adds unnecessary complexity.
- C) Externalize routing configuration (model priorities, thresholds, fallback chains) so it can be updated without redeploying application code, and instrument the router with metrics (latency, cost, quality scores) to enable data-driven tuning over time.
- D) Avoid logging model-selection decisions to reduce storage costs, since routing metadata has no diagnostic value.

---

## Answer Key

### Question 1: B

**Explanation:** The core value proposition of a multi-model strategy is cost and latency optimization. Simple requests — such as FAQ look-ups or short extractive summaries — can be handled accurately by smaller, cheaper models like Phi-4, while complex tasks that demand deep reasoning or broad knowledge are escalated to a more capable (and more expensive) model like GPT-4o. This approach reduces overall inference spend and improves average response latency without sacrificing quality where it matters. Option A is incorrect because Azure AI Foundry does not impose such a limitation. Option C is factually wrong — smaller models do not universally outperform larger ones. Option D is incorrect because multi-model orchestration requires deliberate architectural design; it does not happen automatically.

### Question 2: C

**Explanation:** Cost-based routing selects the least expensive model that can still produce an acceptable-quality response for a given request. If multiple models are equally capable of handling a request, the router should prefer the one with the lowest per-token cost to minimize spend. Option A ignores cost entirely. Option B conflates deployment recency with pricing, which has no basis. Option D (round-robin) does not optimize for cost — it simply spreads load without considering price differences.

### Question 3: B

**Explanation:** Capability-based routing matches the nature of the task to the strengths of each model. GPT-4o excels at code generation, complex reasoning, and multi-turn dialogue, while Meta-Llama-3.1-8B can handle straightforward summarization tasks effectively at lower cost. The router examines the task type (code generation vs. summarization) and dispatches accordingly. Option A ignores task-model fit. Option C is arbitrary and unrelated to model capabilities. Option D describes a naive default-with-override pattern, not true capability-based classification.

### Question 4: C

**Explanation:** The fallback/cascade pattern is designed to gracefully handle situations where the primary (often cheaper) model cannot meet the quality bar. When Phi-4 returns a low-confidence result, the orchestration layer should escalate to a more capable model such as GPT-4o, check its response against the confidence threshold, and return the best available result. This pattern maximizes cost efficiency on easy requests while preserving quality on hard ones. Option A prematurely returns a poor result. Option B is an overreaction that provides no value. Option D incorrectly caches and reuses a known-bad response.

### Question 5: B

**Explanation:** Phi-4, as a small language model (SLM), offers significantly lower per-token pricing and faster inference times compared to GPT-4o, which is a large frontier model. However, the trade-off is that Phi-4 may not match GPT-4o's performance on tasks that require extensive multi-step reasoning, nuanced instruction following, or broad factual knowledge. For a high-volume Q&A workload, many questions will be simple enough for Phi-4, but some will benefit from GPT-4o's deeper capabilities — making a routing strategy valuable. Option A is incorrect; pricing differs substantially. Option C inverts the cost relationship. Option D describes a non-existent pricing policy.

### Question 6: B

**Explanation:** A production-grade orchestration layer is a centralized component responsible for request classification, model selection based on configurable policies (cost, capability, latency), fallback/retry logic, and observability through telemetry and logging. This design enables the routing strategy to evolve independently of the application code and provides the metrics needed to tune the system over time. Option A lacks resilience and observability. Option C puts routing decisions on end users, which is not a scalable or reliable architecture. Option D wastes resources by deploying unnecessary models.

### Question 7: B

**Explanation:** The pseudocode shows a basic router that selects between Phi-4 and GPT-4o based on a complexity label, but it has no error handling, no fallback if the chosen model fails (network error, rate limit, low-quality response), and no logging or telemetry. In production, the router must catch exceptions from the `azure-ai-inference` SDK's `ChatCompletionsClient`, attempt an alternative model when the primary choice fails, and emit structured logs so operators can monitor routing decisions, latency, and error rates. Option A is incorrect — the SDK supports multiple client instances. Option C is incorrect — Azure AI Foundry does not auto-classify complexity. Option D is incorrect — the `azure-ai-inference` SDK is a fully supported SDK for Azure AI Foundry inference endpoints.

### Question 8: B

**Explanation:** A tiered routing strategy captures the best of both worlds: Phi-4 handles the high volume of simple queries at low cost, while GPT-4o is reserved for complex advisory queries where quality directly affects customer satisfaction. Continuous monitoring of both cost metrics and quality/satisfaction scores ensures the routing thresholds can be refined over time. Option A overstates the ability of fine-tuning to close all quality gaps. Option C incorrectly claims that multi-model routing is unsupported. Option D introduces a third model without routing logic, which does not solve the quality problem for complex queries.

### Question 9: B

**Explanation:** The Azure AI Foundry model catalog is a searchable, filterable collection of models from multiple providers — including Microsoft (GPT-4o, Phi-4), Meta (Llama family), Mistral, and others. Architects can filter by task type, license, and deployment option (serverless API, managed compute) to identify complementary models for different tiers of a multi-model pipeline, then deploy them directly from the catalog. Option A is incorrect — the catalog includes open-source and third-party models. Option C overstates the catalog's automation — routing logic is the developer's responsibility. Option D is incorrect — the catalog integrates deployment capabilities directly.

### Question 10: C

**Explanation:** Externalizing routing configuration — model priorities, confidence thresholds, fallback chains — allows operations teams to adjust the routing strategy without redeploying application code. This is critical in production where model performance, pricing, and availability can change. Instrumenting the router with latency, cost, and quality metrics enables data-driven tuning: teams can identify when a model's quality has degraded, when costs have shifted, or when a new model in the catalog could improve a routing tier. Option A exposes infrastructure details to the front end and couples routing to UI deployments. Option B ignores the reality that different models and task types need different thresholds. Option D discards valuable diagnostic data that is essential for troubleshooting and optimization.
