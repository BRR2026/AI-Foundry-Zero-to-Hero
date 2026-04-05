# Module 22: Multi-Model Orchestration & Routing — Training Guide

## Azure AI Foundry: Zero to Hero Training Course

**ARC 5: ORCHESTRATION & INTEGRATION** | Module 22 of 45 | April 2026

---

## Module Overview

Modern AI applications rarely depend on a single model. In this module, you will learn how to orchestrate multiple models within Azure AI Foundry — routing requests to the right model based on task complexity, cost constraints, and quality requirements.

We explore the practical realities of working with models like GPT-4o, Phi-4, and Meta-Llama-3.1-8B side by side. You will build router logic that directs simple classification tasks to lightweight models (saving cost) while escalating complex reasoning tasks to frontier models (preserving quality). By the end of this module, you will have built a working smart router that dynamically selects models based on configurable policies.

This is **not** Azure Machine Learning — all work is done within the Azure AI Foundry platform, using the unified model catalog and the `azure-ai-inference` SDK.

---

## Learning Objectives

By the end of this module, you will be able to:

1. **Explain** why multi-model architectures outperform single-model deployments for production workloads in terms of cost, latency, and resilience.
2. **Identify** the three core router patterns — cost-based, capability-based, and fallback/cascade — and select the appropriate pattern for a given scenario.
3. **Deploy** multiple models from the Azure AI Foundry model catalog and invoke them through a unified Python interface using the `azure-ai-inference` SDK.
4. **Implement** a smart router in Python that dynamically routes prompts to the optimal model based on task classification, cost thresholds, and quality gates.
5. **Analyze** performance-vs-cost trade-offs using a structured comparison framework covering latency, throughput, token economics, and output quality.
6. **Build** a fallback chain that gracefully degrades from a frontier model to progressively cheaper alternatives when rate limits, errors, or budget constraints are encountered.

---

## Prerequisites

- Completion of Modules 1–21 (ARCs 1–4)
- Active Azure subscription with an Azure AI Foundry resource provisioned
- At least 2 models deployed in your AI Foundry project (e.g., GPT-4o and Phi-4)
- Python 3.10+ with the `azure-ai-inference` SDK installed
- Familiarity with the Azure AI Foundry model catalog (covered in Module 21)
- Basic understanding of REST APIs and async Python (`asyncio`)

---

## Module Duration

| Section | Topic | Duration |
|---------|-------|----------|
| 1 | Introduction — The Multi-Model Shift | 10 min |
| 2 | When to Use Multiple Models | 15 min |
| 3 | Router Patterns Deep Dive | 20 min |
| 4 | Orchestrating GPT-4o + Phi + Llama | 20 min |
| 5 | Performance vs Cost Trade-offs | 15 min |
| 6 | **Mini Hack:** Build a Smart Router | 60 min |
| 7 | Quiz & Review | 15 min |
| | **Total** | **~2.5 hours** |

---

## Agenda

### 1. Introduction (10 min)

- Why single-model architectures hit a ceiling
- The economics of routing: not every prompt needs GPT-4o
- Overview of the module and what we will build

### 2. When to Use Multiple Models (15 min)

- Single-model vs multi-model decision framework
- Use cases that demand model diversity
- Organizational readiness checklist

### 3. Router Patterns (20 min)

- **Cost-Based Routing** — Route to the cheapest model that meets a quality threshold
- **Capability-Based Routing** — Route based on task type (reasoning, classification, extraction, summarization)
- **Fallback/Cascade Routing** — Try the cheapest model first, escalate if quality or confidence is low

### 4. Orchestrating GPT-4o + Phi + Llama (20 min)

- Deploying models from the AI Foundry model catalog
- Unified inference with `azure-ai-inference`
- Building the orchestration pipeline step by step

### 5. Performance vs Cost Trade-offs (15 min)

- Benchmarking methodology
- Comparison tables: latency, throughput, cost per 1K tokens
- When routing complexity pays for itself

### 6. Mini Hack: Build a Smart Router (60 min)

- Hands-on lab: implement a complete smart router
- Test with real prompts across multiple task types
- Add monitoring and logging

### 7. Quiz & Review (15 min)

- 10-question multiple-choice quiz
- Open Q&A and key takeaways

---

## Detailed Content

---

### Section 1: Why Multi-Model Matters

#### The Single-Model Ceiling

For the first wave of generative AI applications, teams deployed a single large language model — typically GPT-4 or GPT-4o — and routed every request to it. This approach is simple but creates three compounding problems:

1. **Cost escalation.** Frontier models like GPT-4o charge premium rates per token. When every customer query — from "What are your store hours?" to "Analyze this 50-page contract" — hits the same expensive endpoint, costs grow linearly with traffic.

2. **Latency variance.** Large models are slower. Simple queries that a smaller model could answer in 200ms instead wait 1–3 seconds for a frontier model to respond.

3. **Single point of failure.** If your one model deployment hits a rate limit, experiences an outage, or is temporarily unavailable during a region failover, your entire application goes down.

#### The Multi-Model Advantage

A multi-model architecture addresses all three problems:

| Problem | Multi-Model Solution |
|---------|---------------------|
| High cost | Route simple tasks to cheap models (Phi-4 at ~10x lower cost) |
| High latency | Small models respond faster for straightforward queries |
| Single point of failure | Fallback chains provide automatic redundancy |
| One-size-fits-all quality | Specialized models can outperform generalists on narrow tasks |

> **Key Insight:** The goal is not to replace your frontier model — it is to reserve it for the tasks where it genuinely adds value.

#### The Azure AI Foundry Advantage

Azure AI Foundry makes multi-model orchestration practical because:

- The **model catalog** provides one-click deployment for hundreds of models (OpenAI, Microsoft, Meta, Mistral, Cohere, and more).
- The **`azure-ai-inference` SDK** provides a **unified client interface** — the same `ChatCompletionsClient` works with GPT-4o, Phi-4, and Llama without changing your code structure.
- **Model-as-a-Service (MaaS)** deployments eliminate the need to manage GPU infrastructure for each model.

---

### Section 2: When to Use Multiple Models

#### Decision Matrix

Use this matrix to determine whether your application needs multi-model orchestration:

| Scenario | Single Model | Multi-Model |
|----------|:------------:|:-----------:|
| Prototype / MVP with low traffic | ✅ | ❌ |
| Production app with consistent, simple tasks | ✅ | ❌ |
| Production app with mixed task complexity | ❌ | ✅ |
| Cost-sensitive workload (>100K requests/day) | ❌ | ✅ |
| Requires high availability / redundancy | ❌ | ✅ |
| Specialized tasks (code gen + summarization + classification) | ❌ | ✅ |
| Regulatory requirement for specific model providers | ❌ | ✅ |

#### When to Stay Single-Model

- Your application handles fewer than 1,000 requests per day
- All tasks are roughly the same complexity
- You need perfectly consistent output style and formatting
- The engineering overhead of routing is not justified by savings

#### When to Adopt Multi-Model

- **Cost optimization:** You are spending more than $500/month on inference and at least 40% of requests are simple (classification, extraction, FAQ)
- **Specialized tasks:** Different parts of your pipeline need different strengths (e.g., code generation vs. summarization vs. entity extraction)
- **Redundancy:** Your SLA requires 99.9%+ uptime and you cannot tolerate a single-model outage
- **Latency requirements:** Some user-facing interactions need sub-500ms responses

---

### Section 3: Router Patterns Deep Dive

#### Pattern 1: Cost-Based Routing

Cost-based routing sends each request to the **cheapest model that meets a minimum quality threshold** for the given task.

**How it works:**

1. Classify the incoming prompt's complexity (simple, moderate, complex).
2. Look up the cheapest model that historically meets the quality bar for that complexity tier.
3. Route to that model.

```python
from enum import Enum

class Complexity(Enum):
    SIMPLE = "simple"
    MODERATE = "moderate"
    COMPLEX = "complex"

# Cost per 1K input tokens (approximate, April 2026)
MODEL_COSTS = {
    "Phi-4": 0.00007,
    "Meta-Llama-3.1-8B-Instruct": 0.00015,
    "gpt-4o": 0.0025,
}

# Minimum model tier required for each complexity level
COMPLEXITY_TO_MODEL = {
    Complexity.SIMPLE: "Phi-4",
    Complexity.MODERATE: "Meta-Llama-3.1-8B-Instruct",
    Complexity.COMPLEX: "gpt-4o",
}

def classify_complexity(prompt: str) -> Complexity:
    """Classify prompt complexity using simple heuristics."""
    word_count = len(prompt.split())
    # Simple heuristic — in production, use a lightweight classifier
    if word_count < 30 and "?" in prompt:
        return Complexity.SIMPLE
    elif word_count < 150:
        return Complexity.MODERATE
    else:
        return Complexity.COMPLEX

def select_model_by_cost(prompt: str) -> str:
    """Select the cheapest model that can handle this prompt."""
    complexity = classify_complexity(prompt)
    return COMPLEXITY_TO_MODEL[complexity]
```

**Pros:** Maximizes cost savings. Easy to reason about.

**Cons:** Complexity classification can be inaccurate. Requires tuning thresholds.

---

#### Pattern 2: Capability-Based Routing

Capability-based routing matches the **task type** to a model that excels at that specific capability.

**How it works:**

1. Detect the task type from the prompt (reasoning, classification, extraction, summarization, code generation).
2. Route to the model with the highest benchmark score for that task type.

```python
from typing import Dict, List

# Capability scores (0–100) per model, based on internal benchmarks
CAPABILITY_MATRIX: Dict[str, Dict[str, int]] = {
    "gpt-4o": {
        "reasoning": 95,
        "classification": 90,
        "extraction": 88,
        "summarization": 92,
        "code_generation": 94,
    },
    "Phi-4": {
        "reasoning": 72,
        "classification": 85,
        "extraction": 80,
        "summarization": 78,
        "code_generation": 76,
    },
    "Meta-Llama-3.1-8B-Instruct": {
        "reasoning": 68,
        "classification": 82,
        "extraction": 78,
        "summarization": 75,
        "code_generation": 65,
    },
}

TASK_KEYWORDS: Dict[str, List[str]] = {
    "reasoning": ["explain", "why", "analyze", "compare", "evaluate", "think"],
    "classification": ["classify", "categorize", "label", "is this", "sentiment"],
    "extraction": ["extract", "find", "list all", "entities", "parse"],
    "summarization": ["summarize", "tldr", "brief", "overview", "key points"],
    "code_generation": ["write code", "function", "implement", "script", "program"],
}

def detect_task_type(prompt: str) -> str:
    """Detect the primary task type from prompt keywords."""
    prompt_lower = prompt.lower()
    scores = {}
    for task, keywords in TASK_KEYWORDS.items():
        scores[task] = sum(1 for kw in keywords if kw in prompt_lower)
    return max(scores, key=scores.get) if max(scores.values()) > 0 else "reasoning"

def select_model_by_capability(prompt: str, min_score: int = 75) -> str:
    """Select the best model for the detected task type."""
    task = detect_task_type(prompt)
    # Find the cheapest model that meets the minimum capability score
    candidates = [
        (model, scores[task])
        for model, scores in CAPABILITY_MATRIX.items()
        if scores[task] >= min_score
    ]
    if not candidates:
        return "gpt-4o"  # Fallback to most capable
    # Sort by cost (cheapest first) among qualified models
    cost_order = list(MODEL_COSTS.keys())
    candidates.sort(key=lambda x: cost_order.index(x[0]))
    return candidates[0][0]
```

**Pros:** Leverages each model's strengths. Higher overall quality.

**Cons:** Requires maintaining a capability matrix. Task detection adds latency.

---

#### Pattern 3: Fallback/Cascade Routing

Cascade routing tries the **cheapest model first** and escalates to more expensive models only when the initial response fails a quality check.

**How it works:**

1. Send the prompt to the cheapest model.
2. Evaluate the response (confidence score, format compliance, length check).
3. If the response passes quality gates, return it.
4. If it fails, escalate to the next model in the chain.

```python
import asyncio
from azure.ai.inference.aio import ChatCompletionsClient
from azure.core.credentials import AzureKeyCredential

# Ordered from cheapest to most expensive
CASCADE_ORDER = [
    {
        "name": "Phi-4",
        "endpoint": "https://<your-phi4-endpoint>.inference.ai.azure.com",
        "key": "<your-phi4-key>",
    },
    {
        "name": "Meta-Llama-3.1-8B-Instruct",
        "endpoint": "https://<your-llama-endpoint>.inference.ai.azure.com",
        "key": "<your-llama-key>",
    },
    {
        "name": "gpt-4o",
        "endpoint": "https://<your-gpt4o-endpoint>.inference.ai.azure.com",
        "key": "<your-gpt4o-key>",
    },
]

def passes_quality_gate(response_text: str, prompt: str) -> bool:
    """Evaluate whether a model response meets minimum quality standards."""
    # Gate 1: Response must not be empty or trivially short
    if len(response_text.strip()) < 20:
        return False
    # Gate 2: Response should not contain refusal patterns
    refusal_patterns = ["i cannot", "i'm unable", "as an ai", "i don't have"]
    if any(p in response_text.lower() for p in refusal_patterns):
        return False
    # Gate 3: Response length should be proportional to prompt complexity
    if len(prompt.split()) > 100 and len(response_text.split()) < 50:
        return False
    return True

async def cascade_route(prompt: str) -> dict:
    """Try models in order from cheapest to most expensive."""
    for model_config in CASCADE_ORDER:
        try:
            client = ChatCompletionsClient(
                endpoint=model_config["endpoint"],
                credential=AzureKeyCredential(model_config["key"]),
            )
            response = await client.complete(
                messages=[{"role": "user", "content": prompt}],
                max_tokens=1024,
            )
            response_text = response.choices[0].message.content

            if passes_quality_gate(response_text, prompt):
                return {
                    "model": model_config["name"],
                    "response": response_text,
                    "escalated": model_config != CASCADE_ORDER[0],
                }
            # Quality gate failed — escalate to next model
            print(f"[Router] {model_config['name']} failed quality gate, escalating...")
        except Exception as e:
            print(f"[Router] {model_config['name']} error: {e}, escalating...")
            continue

    return {"model": "none", "response": "All models failed.", "escalated": True}
```

**Pros:** Guarantees lowest cost per request on average. Built-in redundancy.

**Cons:** Higher latency on escalated requests (multiple model calls). Quality gate design is critical.

---

### Section 4: Building the Orchestration Pipeline

This section walks through building a complete orchestration pipeline in Azure AI Foundry, from model deployment to a working router.

#### Step 1: Deploy Models in Azure AI Foundry

Before writing any code, deploy the models you need from the AI Foundry model catalog.

**Models for this lab:**

| Model | Deployment Type | Why |
|-------|----------------|-----|
| GPT-4o | Provisioned / Pay-as-you-go | Frontier reasoning and generation |
| Phi-4 | Serverless API (MaaS) | Fast, cheap, Microsoft-native SLM |
| Meta-Llama-3.1-8B-Instruct | Serverless API (MaaS) | Strong open-source alternative |

> **Azure AI Foundry Portal Steps:**
> 1. Open your AI Foundry project at [https://ai.azure.com](https://ai.azure.com).
> 2. Navigate to **Model catalog** in the left sidebar.
> 3. Search for each model, click **Deploy**, and select **Serverless API** (for Phi-4 and Llama) or **Provisioned** (for GPT-4o).
> 4. Note the **endpoint URL** and **API key** for each deployment.

#### Step 2: Install Dependencies

```bash
pip install azure-ai-inference azure-identity python-dotenv
```

#### Step 3: Configure Environment

Create a `.env` file with your deployment details:

```env
# GPT-4o
GPT4O_ENDPOINT=https://<your-project>.openai.azure.com
GPT4O_KEY=<your-key>

# Phi-4
PHI4_ENDPOINT=https://Phi-4-<uid>.eastus2.models.ai.azure.com
PHI4_KEY=<your-key>

# Meta-Llama-3.1-8B-Instruct
LLAMA_ENDPOINT=https://Meta-Llama-3-1-8B-Instruct-<uid>.eastus2.models.ai.azure.com
LLAMA_KEY=<your-key>
```

#### Step 4: Create the Unified Client Factory

The `azure-ai-inference` SDK provides a consistent `ChatCompletionsClient` interface across all models. We wrap this in a factory for clean orchestration:

```python
"""model_clients.py — Unified client factory for Azure AI Foundry models."""

import os
from dotenv import load_dotenv
from azure.ai.inference import ChatCompletionsClient
from azure.core.credentials import AzureKeyCredential

load_dotenv()

MODEL_REGISTRY = {
    "gpt-4o": {
        "endpoint": os.getenv("GPT4O_ENDPOINT"),
        "key": os.getenv("GPT4O_KEY"),
        "cost_per_1k_input": 0.0025,
        "cost_per_1k_output": 0.01,
    },
    "Phi-4": {
        "endpoint": os.getenv("PHI4_ENDPOINT"),
        "key": os.getenv("PHI4_KEY"),
        "cost_per_1k_input": 0.00007,
        "cost_per_1k_output": 0.00014,
    },
    "Meta-Llama-3.1-8B-Instruct": {
        "endpoint": os.getenv("LLAMA_ENDPOINT"),
        "key": os.getenv("LLAMA_KEY"),
        "cost_per_1k_input": 0.00015,
        "cost_per_1k_output": 0.00015,
    },
}

def get_client(model_name: str) -> ChatCompletionsClient:
    """Return a ChatCompletionsClient for the specified model."""
    config = MODEL_REGISTRY[model_name]
    return ChatCompletionsClient(
        endpoint=config["endpoint"],
        credential=AzureKeyCredential(config["key"]),
    )

def get_all_model_names() -> list[str]:
    """Return all registered model names."""
    return list(MODEL_REGISTRY.keys())
```

#### Step 5: Implement the Smart Router

```python
"""smart_router.py — Multi-model router with cost-based and cascade strategies."""

import time
import logging
from azure.ai.inference.models import UserMessage, SystemMessage
from model_clients import get_client, MODEL_REGISTRY

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger("SmartRouter")


class SmartRouter:
    """Routes prompts to the optimal model based on configurable policies."""

    def __init__(self, strategy: str = "cascade"):
        """
        Args:
            strategy: One of 'cost', 'capability', or 'cascade'.
        """
        self.strategy = strategy
        self.request_log = []

    def route(self, prompt: str, system_prompt: str = "") -> dict:
        """Route a prompt to the best model and return the response."""
        if self.strategy == "cascade":
            return self._cascade_route(prompt, system_prompt)
        elif self.strategy == "cost":
            return self._cost_route(prompt, system_prompt)
        elif self.strategy == "capability":
            return self._capability_route(prompt, system_prompt)
        else:
            raise ValueError(f"Unknown strategy: {self.strategy}")

    def _invoke_model(self, model_name: str, prompt: str, system_prompt: str) -> dict:
        """Invoke a single model and return response with metadata."""
        client = get_client(model_name)
        messages = []
        if system_prompt:
            messages.append(SystemMessage(content=system_prompt))
        messages.append(UserMessage(content=prompt))

        start_time = time.time()
        response = client.complete(messages=messages, max_tokens=1024)
        elapsed = time.time() - start_time

        result = {
            "model": model_name,
            "response": response.choices[0].message.content,
            "latency_ms": round(elapsed * 1000),
            "input_tokens": response.usage.prompt_tokens,
            "output_tokens": response.usage.completion_tokens,
            "estimated_cost": self._estimate_cost(
                model_name,
                response.usage.prompt_tokens,
                response.usage.completion_tokens,
            ),
        }
        self.request_log.append(result)
        logger.info(
            f"[{model_name}] {result['latency_ms']}ms | "
            f"${result['estimated_cost']:.6f} | "
            f"{result['input_tokens']}+{result['output_tokens']} tokens"
        )
        return result

    def _estimate_cost(
        self, model_name: str, input_tokens: int, output_tokens: int
    ) -> float:
        """Estimate the cost of a model invocation."""
        config = MODEL_REGISTRY[model_name]
        input_cost = (input_tokens / 1000) * config["cost_per_1k_input"]
        output_cost = (output_tokens / 1000) * config["cost_per_1k_output"]
        return input_cost + output_cost

    def _cascade_route(self, prompt: str, system_prompt: str) -> dict:
        """Try models cheapest-first, escalating on failure."""
        model_order = ["Phi-4", "Meta-Llama-3.1-8B-Instruct", "gpt-4o"]
        for model_name in model_order:
            try:
                result = self._invoke_model(model_name, prompt, system_prompt)
                if self._passes_quality_gate(result["response"], prompt):
                    result["strategy"] = "cascade"
                    result["escalated"] = model_name != model_order[0]
                    return result
                logger.warning(f"[{model_name}] Failed quality gate, escalating...")
            except Exception as e:
                logger.error(f"[{model_name}] Error: {e}, escalating...")
        return {"model": "none", "response": "All models failed.", "strategy": "cascade"}

    def _cost_route(self, prompt: str, system_prompt: str) -> dict:
        """Route to the cheapest model for the prompt complexity."""
        complexity = self._classify_complexity(prompt)
        model_map = {
            "simple": "Phi-4",
            "moderate": "Meta-Llama-3.1-8B-Instruct",
            "complex": "gpt-4o",
        }
        model_name = model_map[complexity]
        result = self._invoke_model(model_name, prompt, system_prompt)
        result["strategy"] = "cost"
        result["detected_complexity"] = complexity
        return result

    def _capability_route(self, prompt: str, system_prompt: str) -> dict:
        """Route to the best model for the detected task type."""
        task = self._detect_task(prompt)
        task_to_model = {
            "reasoning": "gpt-4o",
            "classification": "Phi-4",
            "extraction": "Meta-Llama-3.1-8B-Instruct",
            "summarization": "Meta-Llama-3.1-8B-Instruct",
            "code_generation": "gpt-4o",
            "general": "Phi-4",
        }
        model_name = task_to_model.get(task, "gpt-4o")
        result = self._invoke_model(model_name, prompt, system_prompt)
        result["strategy"] = "capability"
        result["detected_task"] = task
        return result

    @staticmethod
    def _classify_complexity(prompt: str) -> str:
        word_count = len(prompt.split())
        if word_count < 30:
            return "simple"
        elif word_count < 150:
            return "moderate"
        return "complex"

    @staticmethod
    def _detect_task(prompt: str) -> str:
        prompt_lower = prompt.lower()
        task_signals = {
            "reasoning": ["explain", "why", "analyze", "compare", "evaluate"],
            "classification": ["classify", "categorize", "label", "sentiment"],
            "extraction": ["extract", "find all", "list", "entities", "parse"],
            "summarization": ["summarize", "tldr", "brief", "key points"],
            "code_generation": ["write code", "function", "implement", "script"],
        }
        for task, signals in task_signals.items():
            if any(s in prompt_lower for s in signals):
                return task
        return "general"

    @staticmethod
    def _passes_quality_gate(response_text: str, prompt: str) -> bool:
        if len(response_text.strip()) < 20:
            return False
        refusals = ["i cannot", "i'm unable", "i don't have access"]
        if any(r in response_text.lower() for r in refusals):
            return False
        return True

    def get_cost_summary(self) -> dict:
        """Return aggregate cost and latency statistics."""
        if not self.request_log:
            return {"total_requests": 0}
        total_cost = sum(r["estimated_cost"] for r in self.request_log)
        avg_latency = sum(r["latency_ms"] for r in self.request_log) / len(self.request_log)
        model_counts = {}
        for r in self.request_log:
            model_counts[r["model"]] = model_counts.get(r["model"], 0) + 1
        return {
            "total_requests": len(self.request_log),
            "total_cost": round(total_cost, 6),
            "avg_latency_ms": round(avg_latency),
            "model_distribution": model_counts,
        }
```

#### Step 6: Run the Router

```python
"""main.py — Demonstrate the smart router in action."""

from smart_router import SmartRouter


def main():
    router = SmartRouter(strategy="cascade")

    test_prompts = [
        # Simple — should be handled by Phi-4
        "What is the capital of France?",
        # Moderate — may need Llama or GPT-4o
        "Summarize the key differences between REST and GraphQL APIs, "
        "including when to use each approach.",
        # Complex — likely needs GPT-4o
        (
            "Analyze the following quarterly financial report and identify "
            "three potential risks to the company's growth strategy. Consider "
            "market conditions, competitive pressures, and internal factors. "
            "Provide your analysis in a structured format with evidence."
        ),
    ]

    print("=" * 60)
    print("Smart Router — Cascade Strategy Demo")
    print("=" * 60)

    for prompt in test_prompts:
        print(f"\n--- Prompt: {prompt[:60]}...")
        result = router.route(prompt)
        print(f"    Model:   {result['model']}")
        print(f"    Latency: {result['latency_ms']}ms")
        print(f"    Cost:    ${result['estimated_cost']:.6f}")
        print(f"    Response: {result['response'][:120]}...")

    print("\n" + "=" * 60)
    print("Cost Summary:")
    summary = router.get_cost_summary()
    for key, value in summary.items():
        print(f"  {key}: {value}")


if __name__ == "__main__":
    main()
```

---

### Section 5: Performance vs Cost Analysis

#### Model Comparison Table

| Metric | GPT-4o | Phi-4 | Meta-Llama-3.1-8B |
|--------|--------|-------|--------------------|
| **Parameters** | ~200B (estimated) | 14B | 8B |
| **Input cost / 1K tokens** | $0.0025 | $0.00007 | $0.00015 |
| **Output cost / 1K tokens** | $0.0100 | $0.00014 | $0.00015 |
| **Avg latency (100 tokens out)** | 800–1500ms | 150–400ms | 200–500ms |
| **MMLU score** | ~88% | ~78% | ~68% |
| **Reasoning quality** | Excellent | Good | Moderate |
| **Classification accuracy** | ~95% | ~90% | ~87% |
| **Max context window** | 128K | 16K | 128K |

#### Cost Savings Analysis

Consider an application processing **100,000 requests per day** with the following task distribution:

| Task Type | % of Traffic | Tokens (avg) | Best Cheap Model |
|-----------|:------------:|:------------:|:----------------:|
| Simple FAQ / Classification | 50% | 200 in / 100 out | Phi-4 |
| Moderate Summarization | 30% | 500 in / 300 out | Llama-3.1-8B |
| Complex Reasoning | 20% | 800 in / 500 out | GPT-4o |

**Daily cost comparison:**

| Approach | Daily Cost | Monthly Cost |
|----------|:----------:|:------------:|
| All GPT-4o | ~$425 | ~$12,750 |
| Smart routing (mixed) | ~$85 | ~$2,550 |
| **Savings** | **~$340/day** | **~$10,200/month** |

> **Rule of Thumb:** If more than 40% of your traffic can be handled by a smaller model, multi-model routing pays for itself within the first week — even accounting for the engineering time to build the router.

#### When Routing Complexity Is NOT Worth It

- Traffic is under 1,000 requests/day (savings < $10/day)
- All tasks genuinely require frontier-level reasoning
- Latency budget cannot tolerate the occasional cascade (double-call)
- Team lacks the engineering capacity to maintain routing logic

---

## Mini Hack: Build a Smart Router (60 min)

### Objective

Build a complete smart router that:
1. Accepts user prompts via a command-line interface
2. Classifies each prompt's complexity and task type
3. Routes to the optimal model using the cascade pattern
4. Logs every decision with model name, latency, cost, and quality gate result
5. Displays a cost summary at the end of the session

### Instructions

1. **Clone the starter code** from the module's lab folder.
2. **Configure** your `.env` file with at least two model endpoints.
3. **Implement** the `_passes_quality_gate()` method with at least three quality checks.
4. **Implement** the `_classify_complexity()` method — try improving beyond word count (consider question marks, technical terms, multi-part instructions).
5. **Run** the router with the provided test prompts and verify that simple prompts go to cheap models.
6. **Add** a new routing strategy of your choice (e.g., latency-based, random A/B testing).
7. **Stretch goal:** Add async support using `azure.ai.inference.aio` for parallel model calls.

### Success Criteria

- [ ] Router correctly routes simple queries to Phi-4
- [ ] Router escalates complex queries to GPT-4o
- [ ] Fallback chain handles model errors gracefully
- [ ] Cost summary shows at least 50% savings vs all-GPT-4o baseline
- [ ] All routing decisions are logged with timestamps

---

## Instructor Notes

### Teaching Tips

- **Start with the "why."** Students often ask "why not just use GPT-4o for everything?" Lead with the cost analysis — show them the $10K/month savings example before diving into patterns.
- **Live demo the cost difference.** Send the same simple prompt to both GPT-4o and Phi-4 side by side. Show the latency and cost difference in real time.
- **Common misconception:** Students may think "cheaper model = worse quality." Emphasize that for classification and extraction tasks, smaller models often perform comparably to frontier models.
- **Lab pacing:** The Mini Hack is ambitious for 60 minutes. Encourage students to focus on getting the cascade pattern working first, then add capability-based routing as a stretch goal.

### Common Student Questions

| Question | Suggested Answer |
|----------|-----------------|
| "Can I use Azure API Management for routing instead?" | Yes — APIM can handle model routing at the infrastructure level. This module teaches application-level routing for finer control, but APIM is a valid production pattern. |
| "How do I handle different prompt formats across models?" | The `azure-ai-inference` SDK normalizes the chat format. System/user/assistant roles work consistently across all models in the catalog. |
| "What about streaming responses with routing?" | Streaming works with the cascade pattern, but you need to buffer the first chunk to evaluate the quality gate before committing to stream the full response. |
| "Does Azure AI Foundry have built-in routing?" | As of April 2026, routing is application-level logic. Azure API Management and Azure AI Foundry's gateway features can assist, but the routing decision logic is yours to implement. |

### Demo Suggestions

1. **Side-by-side inference:** Open two terminal windows. Send the same prompt to Phi-4 and GPT-4o. Compare speed and quality.
2. **Cost calculator:** Run the test suite of 20 prompts through the router and display the cost summary. Compare with the "all GPT-4o" baseline.
3. **Failure injection:** Temporarily break the Phi-4 endpoint URL to demonstrate the cascade fallback in action.

---

## Key Terminology

| Term | Definition |
|------|-----------|
| **Model Router** | A software component that selects which AI model should handle a given request based on configurable policies (cost, capability, availability). |
| **Cascade Pattern** | A routing strategy that tries the cheapest model first and escalates to more expensive models only when the response fails a quality gate. Also called "fallback routing." |
| **Cost-Based Routing** | A strategy that selects the cheapest model capable of handling a request at a given complexity tier. |
| **Capability Matrix** | A lookup table mapping each model's proficiency scores across different task types (reasoning, classification, extraction, etc.). |
| **Model Orchestration** | The practice of coordinating multiple AI models within a single application pipeline, including routing, sequencing, and aggregating their outputs. |
| **Fallback Chain** | An ordered list of models to try sequentially. If the first model fails (error, rate limit, or quality gate failure), the next model in the chain is attempted. |
| **Token Economics** | The study of cost optimization in LLM-based applications, focusing on minimizing token usage and routing to cost-effective models without sacrificing output quality. |
| **Quality Gate** | A programmatic check applied to a model's response to determine whether it meets minimum standards before returning it to the user. |
| **Model-as-a-Service (MaaS)** | A deployment mode in Azure AI Foundry where models are hosted and managed by the platform, accessible via serverless API endpoints without GPU infrastructure management. |
| **Prompt Complexity Classification** | The process of analyzing an incoming prompt to estimate how difficult it is for a model to answer, used to inform routing decisions. |

---

## Featured Video

- **Playlist:** Azure AI Foundry — Zero to Hero
- **Link:** [https://www.youtube.com/playlist?list=PLC8Ke1cGWBoskdzX_Z4QBA_YP3QtiAIue](https://www.youtube.com/playlist?list=PLC8Ke1cGWBoskdzX_Z4QBA_YP3QtiAIue)

> 📺 Watch the Module 22 walkthrough video for a live demonstration of the smart router, including real-time cost comparisons and cascade fallback in action.

---

## Additional Resources

| Resource | Link |
|----------|------|
| Azure AI Foundry Documentation | [https://learn.microsoft.com/azure/ai-studio/](https://learn.microsoft.com/azure/ai-studio/) |
| Azure AI Foundry Model Catalog | [https://ai.azure.com/explore/models](https://ai.azure.com/explore/models) |
| `azure-ai-inference` SDK (PyPI) | [https://pypi.org/project/azure-ai-inference/](https://pypi.org/project/azure-ai-inference/) |
| `azure-ai-inference` SDK Documentation | [https://learn.microsoft.com/python/api/azure-ai-inference/](https://learn.microsoft.com/python/api/azure-ai-inference/) |
| Azure AI Foundry Pricing | [https://azure.microsoft.com/pricing/details/ai-studio/](https://azure.microsoft.com/pricing/details/ai-studio/) |
| Phi-4 Model Card | [https://ai.azure.com/explore/models/Phi-4](https://ai.azure.com/explore/models/Phi-4) |
| Meta-Llama-3.1-8B on Azure | [https://ai.azure.com/explore/models/Meta-Llama-3.1-8B-Instruct](https://ai.azure.com/explore/models/Meta-Llama-3.1-8B-Instruct) |
| Azure API Management for AI Gateway | [https://learn.microsoft.com/azure/api-management/](https://learn.microsoft.com/azure/api-management/) |

---

## Assessment

- Complete the **10-question multiple-choice quiz** (see `quiz/assessment.md`)
- **Passing score:** 80% (8 out of 10 correct)
- Questions cover:
  - Router pattern selection (cost, capability, cascade)
  - When to use multi-model vs single-model architectures
  - Token economics and cost estimation
  - Quality gate design principles
  - Azure AI Foundry model deployment types
  - Fallback chain behavior and error handling
  - The `azure-ai-inference` SDK's unified client interface
  - Trade-offs between routing strategies

---

## Next Module

**Module 23** — Continue in **ARC 5: Orchestration & Integration**

Building on the multi-model routing patterns from this module, Module 23 will introduce **Agent Orchestration with Semantic Kernel**, where you will learn to build agentic workflows that combine model routing with tool use, memory, and multi-step planning within Azure AI Foundry.

---

*Azure AI Foundry: Zero to Hero — Module 22 of 45*
*© 2026 — ARC 5: Orchestration & Integration*
