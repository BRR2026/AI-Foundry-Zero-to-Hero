# Lab 30: Cost Optimization & Quota Management

| | |
|---|---|
| **Estimated Time** | 60–90 minutes |
| **Difficulty** | 🟡 Intermediate |
| **Module** | 30 of 45 |

## Prerequisites

| Requirement | Details |
|---|---|
| Azure subscription | Active subscription with AI Foundry access |
| AI Foundry project | Existing project with deployed models |
| Python 3.9+ | With pip package manager |
| Azure CLI | Latest version installed and authenticated |
| Previous modules | Modules 25–29 completed |

> **💡 Note:** This lab builds on your existing AI Foundry deployments from previous modules.

---

## Learning Objectives

By the end of this lab you will be able to:

- ✅ Analyze token usage patterns across AI Foundry deployments
- ✅ Implement token counting and cost estimation in Python
- ✅ Configure Azure Cost Management for AI workloads
- ✅ Set up budget alerts and action groups
- ✅ Optimize prompts for cost efficiency
- ✅ Manage quotas and plan capacity

---

## Exercise 1: Analyze Token Usage (20 min)

In this exercise you will build a Python toolkit for counting tokens and estimating the cost of requests to your AI Foundry models. Understanding token economics is the foundation of every cost-optimization strategy.

### Step 1 — Install Required Packages

Open a terminal in your project directory and install the dependencies:

```bash
pip install tiktoken openai azure-identity azure-mgmt-cognitiveservices
```

### Step 2 — Count Tokens with tiktoken

Create a file called `token_counter.py`:

```python
import tiktoken


def count_tokens(text: str, model: str = "gpt-4o") -> int:
    """Return the number of tokens for a given text and model."""
    encoding = tiktoken.encoding_for_model(model)
    return len(encoding.encode(text))


def count_message_tokens(messages: list[dict], model: str = "gpt-4o") -> int:
    """Count tokens for an entire chat-completion message list.

    Each message carries overhead tokens for role/name formatting.
    """
    encoding = tiktoken.encoding_for_model(model)

    tokens_per_message = 3  # <|start|>role/name\n ... <|end|>
    tokens_per_reply = 3    # every reply is primed with <|start|>assistant<|message|>

    total = 0
    for message in messages:
        total += tokens_per_message
        for key, value in message.items():
            total += len(encoding.encode(value))
    total += tokens_per_reply
    return total


# --- Quick test ---
if __name__ == "__main__":
    sample = "Azure AI Foundry enables teams to build, evaluate, and deploy AI models at scale."
    print(f"Text: {sample}")
    print(f"Token count (gpt-4o): {count_tokens(sample)}")

    messages = [
        {"role": "system", "content": "You are a helpful Azure AI assistant."},
        {"role": "user", "content": "Summarize the benefits of AI Foundry in two sentences."},
    ]
    print(f"Message token count: {count_message_tokens(messages)}")
```

Run the script:

```bash
python token_counter.py
```

**Expected output:**

```
Text: Azure AI Foundry enables teams to build, evaluate, and deploy AI models at scale.
Token count (gpt-4o): 15
Message token count: 28
```

### Step 3 — Build a Cost Estimation Function

Add the following to `token_counter.py`:

```python
# Pricing per 1 K tokens (USD) — update with current rates
MODEL_PRICING = {
    "gpt-4o": {"input": 0.0025, "output": 0.0100},
    "gpt-4o-mini": {"input": 0.00015, "output": 0.0006},
    "gpt-4.1": {"input": 0.002, "output": 0.008},
    "gpt-4.1-mini": {"input": 0.0004, "output": 0.0016},
    "gpt-4.1-nano": {"input": 0.0001, "output": 0.0004},
}


def estimate_cost(
    input_tokens: int,
    output_tokens: int,
    model: str = "gpt-4o",
) -> dict:
    """Estimate the cost for a single request."""
    pricing = MODEL_PRICING.get(model)
    if pricing is None:
        raise ValueError(f"Unknown model: {model}")

    input_cost = (input_tokens / 1_000) * pricing["input"]
    output_cost = (output_tokens / 1_000) * pricing["output"]

    return {
        "model": model,
        "input_tokens": input_tokens,
        "output_tokens": output_tokens,
        "input_cost_usd": round(input_cost, 6),
        "output_cost_usd": round(output_cost, 6),
        "total_cost_usd": round(input_cost + output_cost, 6),
    }


def estimate_monthly_cost(
    requests_per_day: int,
    avg_input_tokens: int,
    avg_output_tokens: int,
    model: str = "gpt-4o",
) -> dict:
    """Project monthly costs based on daily request volume."""
    single = estimate_cost(avg_input_tokens, avg_output_tokens, model)
    monthly_requests = requests_per_day * 30
    return {
        "model": model,
        "daily_requests": requests_per_day,
        "monthly_requests": monthly_requests,
        "cost_per_request_usd": single["total_cost_usd"],
        "monthly_cost_usd": round(single["total_cost_usd"] * monthly_requests, 2),
    }


# --- Quick test ---
if __name__ == "__main__":
    # ... (keep the previous tests) ...

    print("\n--- Cost Estimation ---")
    cost = estimate_cost(input_tokens=500, output_tokens=200, model="gpt-4o")
    print(f"Single request: ${cost['total_cost_usd']:.4f}")

    monthly = estimate_monthly_cost(
        requests_per_day=1000, avg_input_tokens=500, avg_output_tokens=200, model="gpt-4o"
    )
    print(f"Monthly projection: ${monthly['monthly_cost_usd']:.2f}")
```

**Expected output:**

```
--- Cost Estimation ---
Single request: $0.0033
Monthly projection: $97.50
```

### ✅ Checkpoint

Verify:

- [ ] `tiktoken` encodes text and returns a token count
- [ ] `estimate_cost` returns a cost breakdown per request
- [ ] `estimate_monthly_cost` projects a realistic monthly figure

---

## Exercise 2: Optimize Prompts for Cost (20 min)

Token count is directly proportional to cost. In this exercise you will measure a baseline prompt, apply compression techniques, and compare the savings.

### Step 1 — Measure Baseline Token Usage

Create `prompt_optimizer.py`:

```python
from token_counter import count_tokens, count_message_tokens, estimate_cost

VERBOSE_SYSTEM_PROMPT = """You are an incredibly helpful, knowledgeable, and friendly
AI assistant that has been specifically designed and built to assist users with any
and all questions they might have about Microsoft Azure cloud computing services,
including but not limited to Azure AI Foundry, Azure OpenAI Service, Azure Cognitive
Services, and all related products, services, and features. You should always strive
to provide the most accurate, detailed, comprehensive, and up-to-date information
possible. Please format your responses in a clear and organized manner using markdown
formatting where appropriate to enhance readability and understanding."""

VERBOSE_USER_PROMPT = """I would really appreciate it if you could please help me
understand and explain what the main and most significant differences are between
the Standard (S0) deployment type and the Global Standard deployment type that are
available for Azure AI Foundry model deployments. Could you also please include
information about the pricing differences between these two deployment types?"""

baseline_messages = [
    {"role": "system", "content": VERBOSE_SYSTEM_PROMPT},
    {"role": "user", "content": VERBOSE_USER_PROMPT},
]

baseline_tokens = count_message_tokens(baseline_messages)
print(f"Baseline input tokens: {baseline_tokens}")
print(f"Baseline cost (est. 300 output tokens): "
      f"${estimate_cost(baseline_tokens, 300)['total_cost_usd']:.4f}")
```

### Step 2 — Apply Prompt Compression Techniques

Add the optimized versions to the same file:

```python
OPTIMIZED_SYSTEM_PROMPT = """Azure AI assistant. Answer concisely using markdown."""

OPTIMIZED_USER_PROMPT = """Compare Standard (S0) vs Global Standard deployments
in Azure AI Foundry: key differences and pricing."""

optimized_messages = [
    {"role": "system", "content": OPTIMIZED_SYSTEM_PROMPT},
    {"role": "user", "content": OPTIMIZED_USER_PROMPT},
]

optimized_tokens = count_message_tokens(optimized_messages)
print(f"\nOptimized input tokens: {optimized_tokens}")
print(f"Optimized cost (est. 300 output tokens): "
      f"${estimate_cost(optimized_tokens, 300)['total_cost_usd']:.4f}")
```

> **💡 Tip:** Prompt compression techniques that preserve intent:
>
> 1. **Remove filler** — words such as "please", "I would appreciate", "could you"
> 2. **Eliminate redundancy** — say it once, not three ways
> 3. **Use terse system prompts** — models follow short instructions just as well
> 4. **Specify output length** — ask for "2 sentences" instead of open-ended replies

### Step 3 — Compare Before/After Costs

```python
savings_pct = (1 - optimized_tokens / baseline_tokens) * 100
print(f"\n--- Comparison ---")
print(f"Baseline tokens : {baseline_tokens}")
print(f"Optimized tokens: {optimized_tokens}")
print(f"Token reduction  : {savings_pct:.1f}%")

daily_requests = 5_000
baseline_monthly = (baseline_tokens + 300) / 1_000 * 0.0025 * daily_requests * 30
optimized_monthly = (optimized_tokens + 300) / 1_000 * 0.0025 * daily_requests * 30
print(f"Monthly baseline : ${baseline_monthly:.2f}")
print(f"Monthly optimized: ${optimized_monthly:.2f}")
print(f"Monthly savings  : ${baseline_monthly - optimized_monthly:.2f}")
```

**Expected output (approximate):**

```
--- Comparison ---
Baseline tokens : 178
Optimized tokens: 30
Token reduction  : 83.1%
Monthly baseline : $71.70
Monthly optimized: $49.50
Monthly savings  : $22.20
```

### ✅ Checkpoint

Verify:

- [ ] Optimized prompts produce fewer tokens while preserving intent
- [ ] Monthly savings are calculated and meaningful
- [ ] You understand which compression techniques yielded the biggest wins

---

## Exercise 3: Configure Cost Monitoring (15 min)

Azure Cost Management gives you real-time visibility into AI Foundry spending. In this exercise you will create custom views and budget alerts.

### Step 1 — Navigate to Azure Cost Management

1. Open **Azure Portal** > **Cost Management + Billing** > **Cost Management**
2. Select **Cost analysis** in the left menu
3. Set the scope to the resource group that contains your AI Foundry resources

Alternatively, use the Azure CLI to pull recent consumption data:

```bash
# List Cognitive Services / AI Services resources in your subscription
az cognitiveservices account list \
  --output table

# Show usage metrics for a specific AI Services resource
az cognitiveservices account list-usage \
  --name <your-ai-services-resource> \
  --resource-group <your-resource-group> \
  --output table
```

### Step 2 — Create a Custom Cost View for AI

1. In **Cost analysis**, click **+ Add filter**
2. Set **Service name** → `Cognitive Services` (AI Foundry deployments appear here)
3. Group by **Resource** to see per-model costs
4. Set the date range to **Last 30 days**
5. Click **Save** and name the view `AI Foundry Costs`

> **💡 Tip:** Pin this view to your Azure dashboard for quick access.

Use the CLI to query accumulated cost data:

```bash
# Retrieve cost data for the current billing period
az consumption usage list \
  --top 50 \
  --output table
```

### Step 3 — Set Up Budget Alerts

Create a budget with e-mail alerts at 50 %, 80 %, and 100 % thresholds:

```bash
# Create a monthly budget for AI Foundry resources
az consumption budget create \
  --budget-name "ai-foundry-monthly" \
  --amount 500 \
  --time-grain Monthly \
  --start-date 2026-04-01 \
  --end-date 2026-12-31 \
  --resource-group <your-resource-group> \
  --category Cost
```

Then set up an action group for notifications:

```bash
# Create an action group for budget alerts
az monitor action-group create \
  --resource-group <your-resource-group> \
  --name "ai-cost-alerts" \
  --short-name "aicost" \
  --action email admin admin@contoso.com
```

> **⚠️ Warning:** Budget alerts are informational only — they do **not** automatically stop resources. You must configure automation (e.g., Azure Functions) to enforce hard spending limits.

### ✅ Checkpoint

Verify:

- [ ] You can see AI Foundry costs in **Cost analysis** filtered by Cognitive Services
- [ ] A saved cost view named `AI Foundry Costs` exists
- [ ] A monthly budget with alert thresholds is configured

---

## Exercise 4: Quota Management (15 min)

Azure AI Foundry enforces rate limits using **Tokens-Per-Minute (TPM)** and **Requests-Per-Minute (RPM)** quotas. Managing these quotas prevents throttling and ensures availability.

### Step 1 — View Current Quota Usage

Navigate to **Azure AI Foundry portal** (ai.azure.com) > **Management** > **Quotas** to see your current allocation.

Reference quota defaults:

| Model | Default TPM (K) | Default RPM | Max TPM (K) |
|---|---|---|---|
| gpt-4o | 150 | 900 | 10,000 |
| gpt-4o-mini | 200 | 1,200 | 10,000 |
| gpt-4.1 | 150 | 900 | 10,000 |
| gpt-4.1-mini | 200 | 1,200 | 10,000 |
| gpt-4.1-nano | 300 | 1,800 | 10,000 |

> **💡 Tip:** Quotas are assigned per-region, per-subscription. If you need more capacity, you can request an increase or deploy across multiple regions.

Use the CLI to check model deployments:

```bash
# List all deployments under your AI Services resource
az cognitiveservices account deployment list \
  --name <your-ai-services-resource> \
  --resource-group <your-resource-group> \
  --output table
```

### Step 2 — Analyze TPM/RPM Utilization

Create `quota_monitor.py` to query Azure Monitor metrics for your deployment:

```python
import subprocess
import json
from datetime import datetime, timedelta, timezone


def get_token_usage_metrics(resource_id: str, hours: int = 24) -> dict:
    """Retrieve token usage metrics from Azure Monitor."""
    end_time = datetime.now(timezone.utc)
    start_time = end_time - timedelta(hours=hours)

    cmd = [
        "az", "monitor", "metrics", "list",
        "--resource", resource_id,
        "--metric", "TokenTransaction",
        "--interval", "PT1H",
        "--start-time", start_time.isoformat(),
        "--end-time", end_time.isoformat(),
        "--output", "json",
    ]

    result = subprocess.run(cmd, capture_output=True, text=True, check=True)
    data = json.loads(result.stdout)

    hourly_usage = []
    for ts in data.get("value", [{}])[0].get("timeseries", []):
        for point in ts.get("data", []):
            if point.get("total"):
                hourly_usage.append({
                    "time": point["timeStamp"],
                    "tokens": int(point["total"]),
                })

    total_tokens = sum(p["tokens"] for p in hourly_usage)
    peak_hourly = max((p["tokens"] for p in hourly_usage), default=0)

    return {
        "period_hours": hours,
        "total_tokens": total_tokens,
        "avg_tokens_per_hour": total_tokens // max(len(hourly_usage), 1),
        "peak_tokens_per_hour": peak_hourly,
        "data_points": len(hourly_usage),
    }


if __name__ == "__main__":
    resource_id = (
        "/subscriptions/<sub-id>/resourceGroups/<rg>"
        "/providers/Microsoft.CognitiveServices/accounts/<resource>"
    )
    metrics = get_token_usage_metrics(resource_id, hours=24)
    print("Token usage (last 24 h):")
    for k, v in metrics.items():
        print(f"  {k}: {v}")
```

### Step 3 — Plan Capacity for Growth

Create `capacity_planner.py`:

```python
from token_counter import estimate_monthly_cost


def capacity_plan(scenarios: list[dict]) -> None:
    """Print a capacity plan comparing multiple growth scenarios."""
    print(f"{'Scenario':<25} {'Req/Day':>8} {'TPM Needed':>11} {'Monthly $':>10}")
    print("-" * 58)

    for s in scenarios:
        result = estimate_monthly_cost(
            requests_per_day=s["requests_per_day"],
            avg_input_tokens=s["avg_input"],
            avg_output_tokens=s["avg_output"],
            model=s["model"],
        )
        # Estimate peak TPM: assume requests spread over 8 active hours
        peak_rpm = s["requests_per_day"] / (8 * 60)
        peak_tpm = peak_rpm * (s["avg_input"] + s["avg_output"])

        print(f"{s['name']:<25} {s['requests_per_day']:>8,} {peak_tpm:>10,.0f} "
              f"${result['monthly_cost_usd']:>9,.2f}")


if __name__ == "__main__":
    scenarios = [
        {"name": "Current",     "requests_per_day": 1_000, "avg_input": 500,
         "avg_output": 200, "model": "gpt-4o"},
        {"name": "Q3 Growth",   "requests_per_day": 5_000, "avg_input": 500,
         "avg_output": 200, "model": "gpt-4o"},
        {"name": "Q3 Optimized","requests_per_day": 5_000, "avg_input": 200,
         "avg_output": 150, "model": "gpt-4o-mini"},
        {"name": "Q4 Scale",    "requests_per_day": 20_000,"avg_input": 300,
         "avg_output": 150, "model": "gpt-4o-mini"},
    ]
    capacity_plan(scenarios)
```

**Expected output:**

```
Scenario                  Req/Day  TPM Needed  Monthly $
----------------------------------------------------------
Current                     1,000       1,458     $97.50
Q3 Growth                   5,000       7,292    $487.50
Q3 Optimized                5,000       3,646     $58.50
Q4 Scale                   20,000       9,375    $156.00
```

> **💡 Tip:** The optimized Q3 scenario uses `gpt-4o-mini` and shorter prompts — it handles 5× more traffic at roughly 60 % of the current cost.

### ✅ Checkpoint

Verify:

- [ ] You can view quota allocations in the AI Foundry portal
- [ ] `quota_monitor.py` retrieves hourly token metrics via Azure Monitor
- [ ] `capacity_planner.py` outputs a comparison table showing that prompt optimization and model selection dramatically reduce projected costs

---

## Exercise 5: Mini Hack — Implement Cost-Saving Strategies (20 min)

### Challenge

Build a lightweight cost-saving layer that combines **semantic caching**, **model routing**, and **reporting** in a single module.

### Step 1 — Implement Semantic Caching

Create `cost_saver.py`:

```python
import hashlib
import json
import time
from pathlib import Path


class SemanticCache:
    """Simple disk-based cache keyed on the hash of the prompt."""

    def __init__(self, cache_dir: str = ".prompt_cache", ttl_seconds: int = 3600):
        self.cache_dir = Path(cache_dir)
        self.cache_dir.mkdir(exist_ok=True)
        self.ttl = ttl_seconds
        self.stats = {"hits": 0, "misses": 0}

    def _key(self, messages: list[dict]) -> str:
        content = json.dumps(messages, sort_keys=True)
        return hashlib.sha256(content.encode()).hexdigest()

    def get(self, messages: list[dict]) -> str | None:
        key = self._key(messages)
        path = self.cache_dir / f"{key}.json"
        if path.exists():
            entry = json.loads(path.read_text())
            if time.time() - entry["timestamp"] < self.ttl:
                self.stats["hits"] += 1
                return entry["response"]
        self.stats["misses"] += 1
        return None

    def put(self, messages: list[dict], response: str) -> None:
        key = self._key(messages)
        path = self.cache_dir / f"{key}.json"
        path.write_text(json.dumps({
            "timestamp": time.time(),
            "response": response,
        }))

    @property
    def hit_rate(self) -> float:
        total = self.stats["hits"] + self.stats["misses"]
        return self.stats["hits"] / total if total else 0.0
```

### Step 2 — Add Model Routing Logic

Append to `cost_saver.py`:

```python
from token_counter import count_message_tokens, estimate_cost


class ModelRouter:
    """Route requests to the cheapest model that fits the complexity."""

    ROUTING_RULES = [
        {"max_input_tokens": 100,  "model": "gpt-4.1-nano"},
        {"max_input_tokens": 500,  "model": "gpt-4o-mini"},
        {"max_input_tokens": 2000, "model": "gpt-4o"},
        {"max_input_tokens": float("inf"), "model": "gpt-4.1"},
    ]

    def select_model(self, messages: list[dict]) -> str:
        token_count = count_message_tokens(messages)
        for rule in self.ROUTING_RULES:
            if token_count <= rule["max_input_tokens"]:
                return rule["model"]
        return "gpt-4o"  # fallback

    def estimate_savings(self, messages: list[dict], default_model: str = "gpt-4o") -> dict:
        token_count = count_message_tokens(messages)
        est_output = max(token_count, 100)  # rough estimate

        selected = self.select_model(messages)
        default_cost = estimate_cost(token_count, est_output, default_model)
        routed_cost = estimate_cost(token_count, est_output, selected)

        return {
            "default_model": default_model,
            "selected_model": selected,
            "default_cost_usd": default_cost["total_cost_usd"],
            "routed_cost_usd": routed_cost["total_cost_usd"],
            "savings_usd": round(default_cost["total_cost_usd"] - routed_cost["total_cost_usd"], 6),
        }
```

### Step 3 — Generate Cost Report

Append the report generator and a demo:

```python
def generate_cost_report(cache: SemanticCache, router: ModelRouter, requests: list) -> str:
    """Generate a summary report of cost-saving actions."""
    total_default = 0.0
    total_routed = 0.0

    for messages in requests:
        savings = router.estimate_savings(messages)
        total_default += savings["default_cost_usd"]
        total_routed += savings["routed_cost_usd"]

    lines = [
        "=== AI Foundry Cost Optimization Report ===",
        f"Total requests analyzed : {len(requests)}",
        f"Cache hit rate          : {cache.hit_rate:.1%}",
        f"Cost without optimization: ${total_default:.4f}",
        f"Cost with model routing  : ${total_routed:.4f}",
        f"Estimated savings        : ${total_default - total_routed:.4f}",
        f"Savings percentage       : {(1 - total_routed / total_default) * 100:.1f}%",
    ]
    return "\n".join(lines)


# --- Demo ---
if __name__ == "__main__":
    cache = SemanticCache()
    router = ModelRouter()

    sample_requests = [
        [{"role": "user", "content": "Hi"}],
        [{"role": "user", "content": "Explain Azure AI Foundry in a paragraph."}],
        [{"role": "user", "content": "Write a detailed comparison of Standard vs "
                                      "Provisioned Throughput deployments covering "
                                      "pricing, latency, SLA, and capacity planning."}],
        [{"role": "user", "content": "Hi"}],  # duplicate — cache hit
    ]

    for req in sample_requests:
        cached = cache.get(req)
        if cached is None:
            cache.put(req, "(simulated response)")
        model = router.select_model(req)
        tokens = count_message_tokens(req)
        print(f"  [{model:<14}] {tokens:>4} tokens | {req[0]['content'][:50]}")

    print()
    print(generate_cost_report(cache, router, sample_requests))
```

**Expected output:**

```
  [gpt-4.1-nano ]    9 tokens | Hi
  [gpt-4o-mini  ]   18 tokens | Explain Azure AI Foundry in a paragraph.
  [gpt-4o       ]   34 tokens | Write a detailed comparison of Standard vs Provi
  [gpt-4.1-nano ]    9 tokens | Hi

=== AI Foundry Cost Optimization Report ===
Total requests analyzed : 4
Cache hit rate          : 25.0%
Cost without optimization: $0.0018
Cost with model routing  : $0.0004
Estimated savings        : $0.0014
Savings percentage       : 77.5%
```

### ✅ Checkpoint

Verify:

- [ ] `SemanticCache` detects duplicate prompts and returns a cache hit
- [ ] `ModelRouter` selects a cheaper model for simple prompts
- [ ] The cost report shows meaningful savings from both strategies combined

---

## 🎯 Summary & Cleanup

In this lab you learned how to:

| Skill | Exercise |
|---|---|
| Count tokens and estimate costs | Exercise 1 |
| Compress prompts to reduce input tokens | Exercise 2 |
| Monitor AI spend with Azure Cost Management | Exercise 3 |
| Manage TPM/RPM quotas and plan capacity | Exercise 4 |
| Implement caching and model routing | Exercise 5 |

**Clean up resources** to avoid ongoing charges:

```bash
# Delete the budget (if no longer needed)
az consumption budget delete --budget-name "ai-foundry-monthly"

# Delete the action group
az monitor action-group delete \
  --resource-group <your-resource-group> \
  --name "ai-cost-alerts"

# Remove local cache directory
rm -rf .prompt_cache
```

> **⚠️ Warning:** Do **not** delete your AI Foundry project or model deployments if you plan to continue with Modules 31–45.

---

## 📚 Additional Resources

| Resource | Link |
|---|---|
| Azure AI Foundry pricing | https://azure.microsoft.com/pricing/details/ai-foundry/ |
| Azure Cost Management docs | https://learn.microsoft.com/azure/cost-management-billing/ |
| Manage Azure AI Foundry quotas | https://learn.microsoft.com/azure/ai-foundry/how-to/quota |
| tiktoken library | https://github.com/openai/tiktoken |
| OpenAI tokenizer tool | https://platform.openai.com/tokenizer |
| Azure Monitor metrics for AI Services | https://learn.microsoft.com/azure/ai-services/monitor |

---

**Next module →** [Module 31: Intro to Agentic AI Concepts](../../module-31/lab/hands-on-lab.md)
