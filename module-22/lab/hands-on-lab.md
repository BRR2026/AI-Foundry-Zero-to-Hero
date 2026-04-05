# Module 22 — Hands-on Lab: Build a Smart Model Router

## Azure AI Foundry: Zero to Hero Training Course

**ARC 5: ORCHESTRATION & INTEGRATION** | Mini Hack | April 2026

---

## Lab Overview

Build a smart model router that classifies incoming requests by complexity and routes them to the cheapest capable model deployed in Azure AI Foundry. You will deploy **GPT-4o**, **Phi-4**, and **Meta-Llama-3.1-8B-Instruct**, then build a Python-based routing system that optimizes for cost while maintaining quality.

| | |
|---|---|
| **Module** | 22 of 45 — Multi-Model Orchestration & Routing |
| **Arc** | ARC 5: Orchestration & Integration |
| **Mini Hack** | Build a smart router that picks the cheapest capable model |
| **Estimated Duration** | 60 minutes |
| **Difficulty** | Intermediate |

---

## Learning Objectives

By the end of this lab you will be able to:

1. Deploy multiple models from the Azure AI Foundry model catalog as serverless API endpoints
2. Implement a task-complexity classifier that categorizes prompts as **simple**, **moderate**, or **complex**
3. Build a cost-based routing engine that selects the cheapest model capable of handling each request
4. Implement fallback cascade logic that escalates to more powerful models when needed
5. Test the router with mixed workloads and measure cost savings versus a single-model baseline

---

## Prerequisites

| Requirement | Details |
|---|---|
| Azure subscription | Active subscription with sufficient quota |
| Azure AI Foundry project | A project created at [https://ai.azure.com](https://ai.azure.com) |
| Python | 3.10 or later |
| Python packages | `azure-ai-inference`, `azure-identity` |
| Prior module | Completion of Module 21 (Model Catalog & Deployment) |

Install the required packages before starting:

```bash
pip install azure-ai-inference azure-identity
```

---

## Part 1: Deploy Models in Azure AI Foundry (15 minutes)

You will deploy three models with different capability-cost profiles:

| Model | Strengths | Relative Cost |
|---|---|---|
| **GPT-4o** | Reasoning, code generation, complex analysis | $$$ |
| **Phi-4** | General tasks, summarisation, moderate reasoning | $$ |
| **Meta-Llama-3.1-8B-Instruct** | Simple Q&A, classification, extraction | $ |

### Step 1.1 — Deploy GPT-4o

1. Open your Azure AI Foundry project at [https://ai.azure.com](https://ai.azure.com).
2. In the left navigation pane, select **Model catalog**.
3. Search for **GPT-4o** and select the model card.
4. Click **Deploy** → **Serverless API**.
5. Configure the deployment:
   - **Deployment name:** `gpt-4o-router`
   - **Provisioning:** Accept default tokens-per-minute or set to your quota limit.
6. Click **Deploy** and wait for the status to show **Succeeded**.
7. On the deployment details page, copy the **Target URI** and **Key** — you will need these shortly.

### Step 1.2 — Deploy Phi-4

1. Return to **Model catalog** and search for **Phi-4**.
2. Select the model card and click **Deploy** → **Serverless API**.
3. Configure the deployment:
   - **Deployment name:** `phi-4-router`
4. Click **Deploy** and wait for **Succeeded**.
5. Copy the **Target URI** and **Key**.

### Step 1.3 — Deploy Meta-Llama-3.1-8B-Instruct

1. Return to **Model catalog** and search for **Meta-Llama-3.1-8B-Instruct**.
2. Select the model card and click **Deploy** → **Serverless API**.
3. Configure the deployment:
   - **Deployment name:** `llama-31-8b-router`
4. Accept the model licence terms when prompted.
5. Click **Deploy** and wait for **Succeeded**.
6. Copy the **Target URI** and **Key**.

### Step 1.4 — Record Endpoints

Create a file called **`router_config.py`** to store your deployment details:

```python
# router_config.py
# ── Model deployment configuration ──────────────────────────────
# Replace every placeholder with the values copied from AI Foundry.

MODEL_DEPLOYMENTS = {
    "gpt-4o": {
        "endpoint": "https://<your-project>.models.ai.azure.com",
        "deployment_name": "gpt-4o-router",
        "api_key": "<your-gpt4o-key>",          # or use DefaultAzureCredential
        "cost_per_1k_input_tokens": 0.005,
        "cost_per_1k_output_tokens": 0.015,
        "capability_level": 3,                   # 3 = highest
        "max_tokens": 4096,
    },
    "phi-4": {
        "endpoint": "https://<your-project>.models.ai.azure.com",
        "deployment_name": "phi-4-router",
        "api_key": "<your-phi4-key>",
        "cost_per_1k_input_tokens": 0.00014,
        "cost_per_1k_output_tokens": 0.00014,
        "capability_level": 2,
        "max_tokens": 4096,
    },
    "llama-3.1-8b": {
        "endpoint": "https://<your-project>.models.ai.azure.com",
        "deployment_name": "llama-31-8b-router",
        "api_key": "<your-llama-key>",
        "cost_per_1k_input_tokens": 0.00003,
        "cost_per_1k_output_tokens": 0.00003,
        "capability_level": 1,                   # 1 = lowest
        "max_tokens": 4096,
    },
}
```

> **Security note:** In production, store keys in Azure Key Vault or use `DefaultAzureCredential` with managed identity. Hard-coded keys are acceptable only for this lab exercise.

---

## Part 2: Build the Task Classifier (15 minutes)

The classifier inspects every incoming prompt and assigns a complexity level that the router uses to choose a model.

### Step 2.1 — Define Complexity Levels

| Level | Label | Typical tasks | Minimum capability |
|---|---|---|---|
| 1 | **simple** | Greeting, factoid lookup, single-sentence extraction | 1 (Llama 8B) |
| 2 | **moderate** | Summarisation, translation, short content generation | 2 (Phi-4) |
| 3 | **complex** | Multi-step reasoning, code generation, detailed analysis | 3 (GPT-4o) |

### Step 2.2 — Implement the Classifier

Create **`task_classifier.py`**:

```python
# task_classifier.py
"""Classify prompt complexity to drive model routing decisions."""

from __future__ import annotations

import re
from dataclasses import dataclass
from enum import IntEnum


class Complexity(IntEnum):
    SIMPLE = 1
    MODERATE = 2
    COMPLEX = 3


# ── keyword / pattern banks ────────────────────────────────────
COMPLEX_KEYWORDS: set[str] = {
    "analyze", "analyse", "compare and contrast", "write code",
    "implement", "debug", "refactor", "architecture",
    "step by step", "chain of thought", "reason through",
    "explain why", "trade-offs", "design pattern",
    "algorithm", "optimise", "optimize", "multi-step",
    "evaluate the implications", "critique",
}

MODERATE_KEYWORDS: set[str] = {
    "summarize", "summarise", "translate", "rewrite",
    "paraphrase", "list the", "describe", "outline",
    "generate a paragraph", "draft an email",
    "convert", "categorize", "categorise", "explain",
}

SIMPLE_PATTERNS: list[re.Pattern[str]] = [
    re.compile(r"^(hi|hello|hey|thanks|thank you)\b", re.IGNORECASE),
    re.compile(r"^what is the (capital|population|currency) of", re.IGNORECASE),
    re.compile(r"^(yes|no|true|false)[.!?]?$", re.IGNORECASE),
]


@dataclass
class ClassificationResult:
    """Stores the outcome of prompt classification."""
    complexity: Complexity
    reason: str
    estimated_input_tokens: int


class TaskClassifier:
    """Analyse a user prompt and return its complexity level."""

    # Rough approximation: 1 token ≈ 4 characters in English
    CHARS_PER_TOKEN = 4

    # Thresholds (in estimated tokens)
    SHORT_TOKEN_LIMIT = 50
    LONG_TOKEN_LIMIT = 300

    def classify(self, prompt: str) -> ClassificationResult:
        """Return a ClassificationResult for the given prompt."""
        est_tokens = max(1, len(prompt) // self.CHARS_PER_TOKEN)
        prompt_lower = prompt.lower()

        # ── Rule 1: simple patterns ────────────────────────────
        for pattern in SIMPLE_PATTERNS:
            if pattern.search(prompt):
                return ClassificationResult(
                    complexity=Complexity.SIMPLE,
                    reason="Matched simple pattern",
                    estimated_input_tokens=est_tokens,
                )

        # ── Rule 2: complex keyword scan ───────────────────────
        complex_hits = [kw for kw in COMPLEX_KEYWORDS if kw in prompt_lower]
        if complex_hits:
            return ClassificationResult(
                complexity=Complexity.COMPLEX,
                reason=f"Complex keywords detected: {', '.join(complex_hits[:3])}",
                estimated_input_tokens=est_tokens,
            )

        # ── Rule 3: moderate keyword scan ──────────────────────
        moderate_hits = [kw for kw in MODERATE_KEYWORDS if kw in prompt_lower]
        if moderate_hits:
            return ClassificationResult(
                complexity=Complexity.MODERATE,
                reason=f"Moderate keywords detected: {', '.join(moderate_hits[:3])}",
                estimated_input_tokens=est_tokens,
            )

        # ── Rule 4: length heuristic ──────────────────────────
        if est_tokens <= self.SHORT_TOKEN_LIMIT:
            return ClassificationResult(
                complexity=Complexity.SIMPLE,
                reason="Short prompt with no complexity markers",
                estimated_input_tokens=est_tokens,
            )

        if est_tokens >= self.LONG_TOKEN_LIMIT:
            return ClassificationResult(
                complexity=Complexity.COMPLEX,
                reason=f"Long prompt ({est_tokens} estimated tokens)",
                estimated_input_tokens=est_tokens,
            )

        # ── Default: moderate ──────────────────────────────────
        return ClassificationResult(
            complexity=Complexity.MODERATE,
            reason="Default — moderate length, no strong signals",
            estimated_input_tokens=est_tokens,
        )
```

### Step 2.3 — Test the Classifier

Create **`test_classifier.py`** and run it to verify the classifier works:

```python
# test_classifier.py
"""Quick smoke test for the TaskClassifier."""

from task_classifier import TaskClassifier, Complexity

classifier = TaskClassifier()

test_prompts = [
    # (prompt, expected_complexity)
    ("Hello!", Complexity.SIMPLE),
    ("What is the capital of France?", Complexity.SIMPLE),
    ("Summarize the key points of this article about climate change.",
     Complexity.MODERATE),
    ("Translate the following paragraph into Spanish.", Complexity.MODERATE),
    ("Write code for a binary search tree in Python with insert, "
     "delete, and search methods. Include unit tests.", Complexity.COMPLEX),
    ("Analyze the trade-offs between microservices and monolithic "
     "architecture for a high-traffic e-commerce platform.", Complexity.COMPLEX),
]

print(f"{'Prompt':<60} {'Expected':<10} {'Got':<10} {'Match'}")
print("=" * 100)

for prompt, expected in test_prompts:
    result = classifier.classify(prompt)
    match = "✓" if result.complexity == expected else "✗"
    print(f"{prompt[:57]:<60} {expected.name:<10} {result.complexity.name:<10} {match}")
    print(f"  └─ reason: {result.reason}")
```

Run:

```bash
python test_classifier.py
```

Expected output — every row should show **✓**.

---

## Part 3: Implement the Smart Router (20 minutes)

### Step 3.1 — Create the Model Registry

The router needs a registry that maps each capability level to the cheapest available model. This is already encoded in `router_config.py` via the `capability_level` and cost fields.

### Step 3.2 — Build the Router Engine

Create **`smart_router.py`**:

```python
# smart_router.py
"""Smart Model Router — cost-optimised multi-model orchestration."""

from __future__ import annotations

import time
from dataclasses import dataclass, field
from typing import Optional

from azure.ai.inference import ChatCompletionsClient
from azure.ai.inference.models import SystemMessage, UserMessage
from azure.core.credentials import AzureKeyCredential

from router_config import MODEL_DEPLOYMENTS
from task_classifier import TaskClassifier, ClassificationResult, Complexity


# ── Data classes ────────────────────────────────────────────────

@dataclass
class RoutingDecision:
    """Captures why a particular model was selected."""
    model_name: str
    complexity: Complexity
    reason: str
    estimated_cost: float


@dataclass
class RouterResponse:
    """The final output returned to the caller."""
    content: str
    model_used: str
    routing_decision: RoutingDecision
    latency_ms: float
    input_tokens: int
    output_tokens: int
    actual_cost: float
    attempts: int = 1


@dataclass
class RoutingStats:
    """Accumulates statistics across many requests."""
    total_requests: int = 0
    total_cost: float = 0.0
    baseline_cost: float = 0.0          # cost if every request went to GPT-4o
    model_usage: dict[str, int] = field(default_factory=dict)
    total_latency_ms: float = 0.0


# ── Router ──────────────────────────────────────────────────────

class SmartModelRouter:
    """Route prompts to the cheapest model that meets the complexity bar."""

    def __init__(self) -> None:
        self.classifier = TaskClassifier()
        self.stats = RoutingStats()

        # Build a sorted list: cheapest model first within each capability
        self._models_by_capability: dict[int, list[dict]] = {}
        for name, cfg in MODEL_DEPLOYMENTS.items():
            cap = cfg["capability_level"]
            entry = {**cfg, "name": name}
            self._models_by_capability.setdefault(cap, []).append(entry)

        # Sort each capability tier by input-token cost (ascending)
        for tier in self._models_by_capability.values():
            tier.sort(key=lambda m: m["cost_per_1k_input_tokens"])

        # Pre-build clients (one per deployment)
        self._clients: dict[str, ChatCompletionsClient] = {}
        for name, cfg in MODEL_DEPLOYMENTS.items():
            self._clients[name] = ChatCompletionsClient(
                endpoint=cfg["endpoint"],
                credential=AzureKeyCredential(cfg["api_key"]),
            )

    # ── public API ──────────────────────────────────────────────

    def route(
        self,
        prompt: str,
        system_message: str = "You are a helpful AI assistant.",
    ) -> RouterResponse:
        """Classify the prompt, pick the cheapest capable model, call it."""
        classification = self.classifier.classify(prompt)
        decision = self._cost_optimized_route(classification)

        response = self._call_model_with_cascade(
            prompt=prompt,
            system_message=system_message,
            decision=decision,
            classification=classification,
        )

        # Update stats
        self.stats.total_requests += 1
        self.stats.total_cost += response.actual_cost
        self.stats.total_latency_ms += response.latency_ms
        self.stats.model_usage[response.model_used] = (
            self.stats.model_usage.get(response.model_used, 0) + 1
        )

        # Baseline: what would GPT-4o have cost?
        gpt4o = MODEL_DEPLOYMENTS["gpt-4o"]
        baseline = (
            (response.input_tokens / 1000) * gpt4o["cost_per_1k_input_tokens"]
            + (response.output_tokens / 1000) * gpt4o["cost_per_1k_output_tokens"]
        )
        self.stats.baseline_cost += baseline

        return response

    def get_stats(self) -> dict:
        """Return a summary of routing statistics."""
        s = self.stats
        savings_pct = (
            ((s.baseline_cost - s.total_cost) / s.baseline_cost * 100)
            if s.baseline_cost > 0 else 0.0
        )
        return {
            "total_requests": s.total_requests,
            "total_cost_usd": round(s.total_cost, 6),
            "baseline_cost_usd": round(s.baseline_cost, 6),
            "savings_pct": round(savings_pct, 2),
            "avg_latency_ms": round(
                s.total_latency_ms / max(1, s.total_requests), 1
            ),
            "model_usage": dict(s.model_usage),
        }

    # ── internal helpers ────────────────────────────────────────

    def _cost_optimized_route(
        self,
        classification: ClassificationResult,
    ) -> RoutingDecision:
        """Pick the cheapest model whose capability >= required level."""
        required = int(classification.complexity)

        # Walk capabilities from required upward
        for cap_level in range(required, max(self._models_by_capability) + 1):
            models_at_level = self._models_by_capability.get(cap_level, [])
            if models_at_level:
                best = models_at_level[0]       # already sorted by cost
                est_cost = (
                    (classification.estimated_input_tokens / 1000)
                    * best["cost_per_1k_input_tokens"]
                )
                return RoutingDecision(
                    model_name=best["name"],
                    complexity=classification.complexity,
                    reason=(
                        f"Cheapest model at capability >= {required}: "
                        f"{best['name']}"
                    ),
                    estimated_cost=est_cost,
                )

        # Fallback — should never happen if config is correct
        fallback_name = list(MODEL_DEPLOYMENTS.keys())[0]
        return RoutingDecision(
            model_name=fallback_name,
            complexity=classification.complexity,
            reason="Fallback — no model matched capability requirement",
            estimated_cost=0.0,
        )

    def _call_model_with_cascade(
        self,
        prompt: str,
        system_message: str,
        decision: RoutingDecision,
        classification: ClassificationResult,
    ) -> RouterResponse:
        """
        Call the chosen model. If the call fails, cascade upward
        to the next more-capable (and more expensive) model.
        """
        models_to_try = self._cascade_order(decision.model_name)
        last_error: Optional[Exception] = None
        attempts = 0

        for model_name in models_to_try:
            attempts += 1
            cfg = MODEL_DEPLOYMENTS[model_name]
            client = self._clients[model_name]

            try:
                start = time.perf_counter()
                result = client.complete(
                    model=cfg["deployment_name"],
                    messages=[
                        SystemMessage(content=system_message),
                        UserMessage(content=prompt),
                    ],
                    max_tokens=cfg["max_tokens"],
                    temperature=0.7,
                )
                elapsed_ms = (time.perf_counter() - start) * 1000

                content = result.choices[0].message.content
                usage = result.usage

                # Quality gate: if the response is suspiciously short
                # for a complex prompt, escalate
                if (
                    classification.complexity == Complexity.COMPLEX
                    and len(content.split()) < 20
                    and model_name != "gpt-4o"
                ):
                    print(
                        f"  ⚠  Response from {model_name} too short "
                        f"({len(content.split())} words) — escalating."
                    )
                    continue

                actual_cost = (
                    (usage.prompt_tokens / 1000)
                    * cfg["cost_per_1k_input_tokens"]
                    + (usage.completion_tokens / 1000)
                    * cfg["cost_per_1k_output_tokens"]
                )

                return RouterResponse(
                    content=content,
                    model_used=model_name,
                    routing_decision=decision,
                    latency_ms=round(elapsed_ms, 1),
                    input_tokens=usage.prompt_tokens,
                    output_tokens=usage.completion_tokens,
                    actual_cost=actual_cost,
                    attempts=attempts,
                )

            except Exception as exc:
                last_error = exc
                print(
                    f"  ⚠  {model_name} failed ({exc}). "
                    f"Cascading to next model…"
                )

        # All models exhausted
        raise RuntimeError(
            f"All models failed. Last error: {last_error}"
        )

    def _cascade_order(self, start_model: str) -> list[str]:
        """
        Return models ordered for cascade: start with the chosen model,
        then move to increasingly capable (and expensive) ones.
        """
        ordered = sorted(
            MODEL_DEPLOYMENTS.items(),
            key=lambda kv: kv[1]["capability_level"],
        )
        model_names = [name for name, _ in ordered]

        start_idx = (
            model_names.index(start_model)
            if start_model in model_names
            else 0
        )
        return model_names[start_idx:] + model_names[:start_idx]
```

### Step 3.3 — Add Fallback Cascade

The cascade logic is built into `_call_model_with_cascade` above. Here is how it works:

```
Request arrives
  │
  ▼
TaskClassifier → complexity = SIMPLE (level 1)
  │
  ▼
_cost_optimized_route → picks Llama 8B (cheapest at level ≥ 1)
  │
  ▼
_call_model_with_cascade:
  ├─ Try Llama 8B ─── success? → return response
  │                     failure? ↓
  ├─ Try Phi-4 ─────── success? → return response
  │                     failure? ↓
  └─ Try GPT-4o ────── success? → return response
                        failure? → raise RuntimeError
```

### Step 3.4 — Add Response Validation

The quality gate is embedded in the cascade method. For **complex** prompts, the router verifies the response has at least 20 words. If a cheaper model returns a suspiciously short answer, the router automatically escalates. You can customise this threshold:

```python
# In _call_model_with_cascade, adjust the quality gate:
MINIMUM_WORDS = {
    Complexity.SIMPLE: 3,
    Complexity.MODERATE: 15,
    Complexity.COMPLEX: 20,
}

min_words = MINIMUM_WORDS.get(classification.complexity, 5)
if len(content.split()) < min_words and model_name != "gpt-4o":
    print(f"  ⚠  Response too short — escalating.")
    continue
```

---

## Part 4: Test & Measure (10 minutes)

### Step 4.1 — Create Test Workload

Create **`run_router.py`**:

```python
# run_router.py
"""Execute a mixed workload through the Smart Model Router."""

from smart_router import SmartModelRouter

# ── Test prompts (mixed complexity) ─────────────────────────────

TEST_PROMPTS = [
    # Simple
    "Hello, how are you?",
    "What is the capital of Japan?",
    "What year did World War II end?",

    # Moderate
    "Summarize the main benefits of cloud computing in three bullet points.",
    "Translate this to French: 'The weather is beautiful today and I plan "
    "to go for a walk in the park.'",
    "Describe the difference between SQL and NoSQL databases.",

    # Complex
    "Write code for a Python function that implements merge sort. Include "
    "type hints, docstrings, and three unit tests using pytest.",
    "Analyze the trade-offs between eventual consistency and strong "
    "consistency in distributed systems. Provide real-world examples of "
    "when each is appropriate.",
    "Compare and contrast the Transformer architecture with traditional "
    "RNNs for sequence-to-sequence tasks. Discuss attention mechanisms, "
    "training efficiency, and parallelisation. Provide step by step "
    "reasoning.",
]


def main() -> None:
    router = SmartModelRouter()

    print("=" * 70)
    print("  SMART MODEL ROUTER — Test Run")
    print("=" * 70)

    for i, prompt in enumerate(TEST_PROMPTS, 1):
        print(f"\n── Request {i}/{len(TEST_PROMPTS)} {'─' * 40}")
        print(f"  Prompt  : {prompt[:80]}{'…' if len(prompt) > 80 else ''}")

        response = router.route(prompt)

        print(f"  Model   : {response.model_used}")
        print(f"  Reason  : {response.routing_decision.reason}")
        print(f"  Tokens  : {response.input_tokens} in / "
              f"{response.output_tokens} out")
        print(f"  Cost    : ${response.actual_cost:.6f}")
        print(f"  Latency : {response.latency_ms:.0f} ms")
        print(f"  Attempts: {response.attempts}")
        print(f"  Response: {response.content[:120]}…")

    # ── Summary ─────────────────────────────────────────────────
    stats = router.get_stats()

    print("\n" + "=" * 70)
    print("  ROUTING SUMMARY")
    print("=" * 70)
    print(f"  Total requests      : {stats['total_requests']}")
    print(f"  Total cost (routed) : ${stats['total_cost_usd']:.6f}")
    print(f"  Baseline cost (4o)  : ${stats['baseline_cost_usd']:.6f}")
    print(f"  Cost savings        : {stats['savings_pct']:.1f}%")
    print(f"  Avg latency         : {stats['avg_latency_ms']:.0f} ms")
    print(f"  Model usage         :")
    for model, count in stats["model_usage"].items():
        print(f"    {model:<20} : {count} requests")
    print("=" * 70)


if __name__ == "__main__":
    main()
```

### Step 4.2 — Run Through Router

```bash
python run_router.py
```

Watch the console output. You should see:

- **Simple** prompts routed to **Llama 3.1 8B** (cheapest).
- **Moderate** prompts routed to **Phi-4**.
- **Complex** prompts routed to **GPT-4o**.
- If any model fails or returns a low-quality response, the cascade automatically escalates.

### Step 4.3 — Calculate Cost Savings

The summary printed at the end of `run_router.py` already compares routed cost with a GPT-4o-only baseline. With typical test workloads you should see **40–70 % savings** because the majority of simple and moderate requests never touch GPT-4o.

Example output:

```
══════════════════════════════════════════════════════════════════════
  ROUTING SUMMARY
══════════════════════════════════════════════════════════════════════
  Total requests      : 9
  Total cost (routed) : $0.001245
  Baseline cost (4o)  : $0.003870
  Cost savings        : 67.8%
  Avg latency         : 823 ms
  Model usage         :
    llama-3.1-8b         : 3 requests
    phi-4                : 3 requests
    gpt-4o               : 3 requests
══════════════════════════════════════════════════════════════════════
```

---

## Solution Code

Below is the complete, self-contained solution. Create four files in the same directory:

<details>
<summary><strong>router_config.py</strong> — deployment configuration</summary>

```python
# router_config.py
MODEL_DEPLOYMENTS = {
    "gpt-4o": {
        "endpoint": "https://<your-project>.models.ai.azure.com",
        "deployment_name": "gpt-4o-router",
        "api_key": "<your-gpt4o-key>",
        "cost_per_1k_input_tokens": 0.005,
        "cost_per_1k_output_tokens": 0.015,
        "capability_level": 3,
        "max_tokens": 4096,
    },
    "phi-4": {
        "endpoint": "https://<your-project>.models.ai.azure.com",
        "deployment_name": "phi-4-router",
        "api_key": "<your-phi4-key>",
        "cost_per_1k_input_tokens": 0.00014,
        "cost_per_1k_output_tokens": 0.00014,
        "capability_level": 2,
        "max_tokens": 4096,
    },
    "llama-3.1-8b": {
        "endpoint": "https://<your-project>.models.ai.azure.com",
        "deployment_name": "llama-31-8b-router",
        "api_key": "<your-llama-key>",
        "cost_per_1k_input_tokens": 0.00003,
        "cost_per_1k_output_tokens": 0.00003,
        "capability_level": 1,
        "max_tokens": 4096,
    },
}
```

</details>

<details>
<summary><strong>task_classifier.py</strong> — prompt complexity classifier</summary>

See [Part 2, Step 2.2](#step-22--implement-the-classifier) above.

</details>

<details>
<summary><strong>smart_router.py</strong> — routing engine with cascade</summary>

See [Part 3, Step 3.2](#step-32--build-the-router-engine) above.

</details>

<details>
<summary><strong>run_router.py</strong> — test harness</summary>

See [Part 4, Step 4.1](#step-41--create-test-workload) above.

</details>

---

## Challenge Extensions

Completed the lab early? Try one or more of these stretch goals:

### 1. Latency-Based Routing

Add a `max_latency_ms` parameter so time-sensitive requests are routed to the fastest-responding model regardless of cost:

```python
def route(self, prompt: str, max_latency_ms: float | None = None) -> RouterResponse:
    classification = self.classifier.classify(prompt)
    if max_latency_ms is not None:
        decision = self._latency_optimized_route(classification, max_latency_ms)
    else:
        decision = self._cost_optimized_route(classification)
    ...
```

### 2. Quality Scoring with a Judge Model

Use GPT-4o as a **judge** to score the output of a cheaper model on a 1–5 scale. If the score is below a threshold, re-route:

```python
def _judge_response(self, prompt: str, response: str) -> int:
    judge_prompt = (
        f"Rate the following response to the user's question on a scale "
        f"of 1 to 5 (5 = excellent). Return ONLY the number.\n\n"
        f"Question: {prompt}\n\nResponse: {response}"
    )
    result = self._clients["gpt-4o"].complete(
        model=MODEL_DEPLOYMENTS["gpt-4o"]["deployment_name"],
        messages=[UserMessage(content=judge_prompt)],
        max_tokens=5,
        temperature=0.0,
    )
    return int(result.choices[0].message.content.strip())
```

### 3. Redis Caching for Repeated Queries

Cache responses keyed by a hash of `(model, prompt)` to avoid paying for duplicate requests:

```python
import hashlib, json, redis

cache = redis.Redis(host="localhost", port=6379, db=0)

def _cache_key(self, model_name: str, prompt: str) -> str:
    raw = json.dumps({"model": model_name, "prompt": prompt}, sort_keys=True)
    return f"router:{hashlib.sha256(raw.encode()).hexdigest()}"
```

### 4. Routing Dashboard

Log every routing decision to a JSON-lines file and visualise with a simple Streamlit app:

```python
import json, datetime

def _log_decision(self, response: RouterResponse, prompt: str) -> None:
    entry = {
        "timestamp": datetime.datetime.utcnow().isoformat(),
        "prompt_preview": prompt[:100],
        "model_used": response.model_used,
        "complexity": response.routing_decision.complexity.name,
        "cost": response.actual_cost,
        "latency_ms": response.latency_ms,
    }
    with open("routing_log.jsonl", "a") as f:
        f.write(json.dumps(entry) + "\n")
```

---

## Cleanup

To avoid ongoing charges, delete the serverless deployments you created:

1. Open your project at [https://ai.azure.com](https://ai.azure.com).
2. Navigate to **Deployments** in the left pane.
3. For each deployment (`gpt-4o-router`, `phi-4-router`, `llama-31-8b-router`):
   - Click the deployment name.
   - Click **Delete** and confirm.
4. Verify that the deployment list is empty.

> **Tip:** If you provisioned dedicated compute (e.g., managed online endpoints), delete those resources as well from the Azure portal to stop billing.

---

## Key Takeaways

| Concept | What You Learned |
|---|---|
| **Model catalog** | Azure AI Foundry hosts hundreds of models from Microsoft, Meta, Mistral, and more — deployable as serverless APIs |
| **Task classification** | A simple keyword + length heuristic can effectively sort requests into complexity tiers |
| **Cost-optimised routing** | Routing simple tasks to small models can cut inference costs by 40–70 % |
| **Fallback cascade** | Automatically escalating on failure or low-quality output keeps reliability high |
| **Quality gates** | Validating response length (or using a judge model) prevents silent degradation |

---

*Module 22 of 45 — Azure AI Foundry: Zero to Hero*
*ARC 5: Orchestration & Integration*
*© 2026 — AI Foundry Training Course*
