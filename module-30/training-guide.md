# Module 30: Cost Optimization & Quota Management

## **ARC 6: DEPLOYMENT & MLOps** | Module 30 of 45 — Zero to Hero: Azure AI Foundry

---

| Detail              | Info                                                |
|---------------------|-----------------------------------------------------|
| **Arc**             | ARC 6 — Deployment & MLOps (Modules 25–30)         |
| **Module**          | 30                                                  |
| **Title**           | Cost Optimization & Quota Management                |
| **Estimated Time**  | 3–4 hours                                           |
| **Difficulty**      | 🟡 Intermediate                                     |
| **Last Updated**    | April 2026                                          |

---

## 🎯 Learning Objectives

By the end of this module, you will be able to:

1. **Compare pricing models** — Evaluate PTU (Provisioned Throughput Units) vs. pay-as-you-go and choose the right model for your workload profile.
2. **Optimize token consumption** — Apply prompt engineering, caching, and model right-sizing techniques to reduce costs by 30–60%.
3. **Manage quotas effectively** — Monitor and request TPM/RPM quota increases, plan regional capacity, and avoid throttling.
4. **Build cost dashboards** — Configure Azure Cost Management views, budget alerts, and custom tagging strategies for AI spend visibility.
5. **Implement FinOps governance** — Establish chargeback/showback models, resource organization, and continuous cost optimization practices.
6. **Analyze real usage** — Write Python scripts that measure token usage, calculate costs, and surface savings opportunities.

---

## 📋 Prerequisites

- **Modules 25–29 completed** — You should have deployed models and built inference pipelines in Azure AI Foundry.
- **Azure subscription** with at least one Azure AI Foundry hub and project provisioned.
- **Contributor or Owner** role on your resource group (required for Cost Management and quota operations).
- **Python 3.10+** with `tiktoken`, `openai`, and `azure-identity` packages installed.
- **Azure CLI 2.60+** installed and authenticated (`az login`).
- Familiarity with the Azure AI Foundry portal at `ai.azure.com`.

---

## 1. Introduction — Why Cost Optimization Matters

AI inference costs can escalate rapidly. A single GPT-4o deployment handling 1 million requests per day at an average of 2,000 tokens per request generates **2 billion tokens daily**. Without deliberate cost management, organizations routinely see monthly AI bills exceeding initial projections by 5–10×.

> 🏢 **Real Example:** A mid-size SaaS company deployed a customer support agent using GPT-4o in Azure AI Foundry. Within the first month, token consumption was 3× their forecast because internal testing had not accounted for multi-turn conversation depth. By applying the techniques in this module — prompt compression, response capping, and a model tiering strategy — they reduced monthly costs from $42,000 to $17,500 without degrading user experience.

Cost optimization in Azure AI Foundry spans three pillars:

```
┌─────────────────────────────────────────────────────────┐
│                 AI COST OPTIMIZATION                    │
├───────────────────┬──────────────────┬──────────────────┤
│   TOKEN USAGE     │  INFRASTRUCTURE  │   GOVERNANCE     │
│                   │                  │                  │
│ • Prompt design   │ • PTU vs PAYG    │ • Budgets        │
│ • Caching         │ • Quota mgmt     │ • Tagging        │
│ • Model selection │ • Regional       │ • Chargeback     │
│ • Response limits │   capacity       │ • FinOps process │
└───────────────────┴──────────────────┴──────────────────┘
```

> 💡 **Tip:** Cost optimization is not a one-time activity. Establish a monthly review cadence — model pricing, available models, and your own usage patterns all shift over time.

---

## 2. Azure AI Foundry Pricing Models

Azure AI Foundry offers two primary pricing models for model inference. Choosing the right one depends on your traffic volume, latency requirements, and budget predictability needs.

### PTU (Provisioned Throughput Units)

PTUs provide **dedicated capacity** reserved for your workload. You pay a fixed hourly rate per PTU regardless of actual usage, guaranteeing consistent throughput and latency.

**When to use PTU:**
- Steady, predictable traffic (e.g., production applications with consistent load).
- Latency-sensitive workloads requiring guaranteed response times.
- Workloads exceeding 300K+ tokens per minute sustained.

**Key characteristics:**
- Billed **per PTU per hour** (monthly commitment or hourly reservation).
- Each PTU delivers a specific tokens-per-minute (TPM) throughput depending on the model.
- No per-token charges — total cost is fixed regardless of token volume.
- Minimum deployment: varies by model (typically 50–100 PTUs).

### Pay-as-You-Go

Pay-as-you-go charges based on **actual token consumption** — you pay for input tokens and output tokens separately.

**When to use Pay-as-You-Go:**
- Variable or bursty traffic patterns.
- Development, testing, and experimentation environments.
- Low-volume production workloads (< 100K TPM sustained).
- Workloads where cost predictability is less critical than cost efficiency.

### Pricing Comparison

| Factor | PTU (Provisioned) | Pay-as-You-Go |
|---|---|---|
| **Billing unit** | Per PTU per hour | Per 1K tokens (input/output) |
| **Cost at low volume** | ❌ Higher (paying for reserved capacity) | ✅ Lower (pay only for what you use) |
| **Cost at high volume** | ✅ Lower (amortized fixed cost) | ❌ Higher (linear scaling) |
| **Latency guarantee** | ✅ Dedicated capacity, consistent latency | ❌ Shared capacity, variable latency |
| **Throttling risk** | ✅ Minimal (dedicated) | ⚠️ Subject to shared quota limits |
| **Minimum commitment** | 50–100 PTUs (model-dependent) | None |
| **Scaling** | Manual (adjust PTU count) | Automatic (within quota) |
| **Best for** | Production at scale | Dev/test, low-to-medium traffic |

> 📝 **Note:** Pricing figures change frequently. Always verify current rates at `ai.azure.com` → your deployment → **Pricing** tab, or via the Azure Pricing Calculator.

**Break-even analysis example (GPT-4o):**

```python
# Break-even: PTU vs Pay-as-You-Go for GPT-4o
# NOTE: Use current pricing from Azure — these are illustrative figures.

ptu_hourly_rate = 2.00          # $/PTU/hour (example)
ptu_count = 100                 # minimum deployment
ptu_monthly = ptu_hourly_rate * ptu_count * 730  # 730 hours/month
# PTU monthly cost: $146,000

payg_input_per_1k = 0.005       # $/1K input tokens (example)
payg_output_per_1k = 0.015      # $/1K output tokens (example)

avg_input_tokens = 1500
avg_output_tokens = 500
requests_per_day = 500_000

monthly_input_tokens = avg_input_tokens * requests_per_day * 30
monthly_output_tokens = avg_output_tokens * requests_per_day * 30

payg_monthly = (
    (monthly_input_tokens / 1000) * payg_input_per_1k +
    (monthly_output_tokens / 1000) * payg_output_per_1k
)

print(f"PTU monthly cost:  ${ptu_monthly:,.2f}")
print(f"PAYG monthly cost: ${payg_monthly:,.2f}")
print(f"Break-even favors: {'PTU' if ptu_monthly < payg_monthly else 'PAYG'}")
```

> ⚠️ **Important:** Always run a break-even analysis with your **actual** traffic patterns before committing to PTU reservations. A one-month pilot on pay-as-you-go provides the usage data you need.

---

## 3. Token Usage Optimization

Token consumption is the single largest cost driver in AI Foundry workloads. Optimizing how many tokens each request consumes delivers immediate, compounding savings.

### Understanding Token Consumption

Every API call to a model consumes tokens across three categories:

| Token Category | Description | You Control? |
|---|---|---|
| **System prompt** | Instructions sent with every request | ✅ Yes |
| **User input** | The user's message / context / documents | ✅ Partially |
| **Model output** | The generated response | ✅ Yes (via `max_tokens`) |

Use `tiktoken` to measure token counts before sending requests:

```python
import tiktoken

def count_tokens(text: str, model: str = "gpt-4o") -> int:
    """Count tokens for a given text and model."""
    encoding = tiktoken.encoding_for_model(model)
    return len(encoding.encode(text))

# Example
system_prompt = "You are a helpful customer support agent for Contoso Ltd."
user_message = "I need to return my order #12345. It arrived damaged."

system_tokens = count_tokens(system_prompt)
user_tokens = count_tokens(user_message)
print(f"System prompt: {system_tokens} tokens")
print(f"User message:  {user_tokens} tokens")
print(f"Total input:   {system_tokens + user_tokens} tokens")
```

### Prompt Engineering for Cost Efficiency

Reducing system prompt length is the highest-leverage optimization because it is sent with **every request**.

| Technique | Savings | Example |
|---|---|---|
| Remove filler language | 10–20% | "You are a helpful assistant that..." → "Assistant role: ..." |
| Use structured formats | 15–30% | Prose instructions → bullet points or YAML |
| Reference by ID not content | 40–60% | Embed full policy text → "Follow policy #CX-201" |
| Conditional injection | 20–50% | Always include all tools → include only relevant tools |

```python
# BEFORE: 847 tokens
system_prompt_verbose = """
You are a highly knowledgeable and helpful customer support agent working
for Contoso Ltd. Your primary responsibility is to assist customers with
their inquiries about orders, returns, shipping, and general product
information. You should always be polite, professional, and thorough in
your responses. When you don't know the answer, you should let the
customer know and offer to escalate their inquiry to a human agent...
"""

# AFTER: 193 tokens (77% reduction)
system_prompt_compact = """Role: Contoso support agent.
Rules:
- Handle: orders, returns, shipping, product info
- Tone: professional, concise
- Unknown answers: say so, offer escalation
- Format: short paragraphs, use bullet lists for steps"""

print(f"Verbose: {count_tokens(system_prompt_verbose)} tokens")
print(f"Compact: {count_tokens(system_prompt_compact)} tokens")
```

### Response Length Management

Set explicit `max_tokens` limits to prevent runaway output costs:

```python
from openai import AzureOpenAI

client = AzureOpenAI(
    azure_endpoint="https://<your-resource>.openai.azure.com/",
    api_version="2025-01-01-preview"
)

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": system_prompt_compact},
        {"role": "user", "content": "How do I return an item?"}
    ],
    max_tokens=300,         # Cap output cost
    temperature=0.3         # Lower temp = shorter, more focused responses
)

# Check actual usage
usage = response.usage
print(f"Input tokens:  {usage.prompt_tokens}")
print(f"Output tokens: {usage.completion_tokens}")
print(f"Total tokens:  {usage.total_tokens}")
```

### Caching Strategies

Prompt caching can dramatically reduce costs for repeated system prompts and common queries.

**Azure AI Foundry automatic prompt caching:**
- Enabled by default for eligible models.
- Caches the **prefix** of the prompt (system message + initial context).
- Cached input tokens are billed at a **50% discount**.
- Cache hits require the prompt prefix to be **identical** (byte-for-byte).

```
┌────────────────────────────────────────────────────────────┐
│ REQUEST STRUCTURE FOR OPTIMAL CACHING                      │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ┌──────────────────────────┐  ← STATIC (cacheable)       │
│  │  System prompt           │     Keep identical across    │
│  │  + Few-shot examples     │     all requests             │
│  │  + Tool definitions      │                              │
│  ├──────────────────────────┤                              │
│  │  Conversation history    │  ← SEMI-STATIC              │
│  │                          │     Grows per session        │
│  ├──────────────────────────┤                              │
│  │  Current user message    │  ← DYNAMIC (not cached)     │
│  └──────────────────────────┘                              │
│                                                            │
│  TIP: Place static content first to maximize cache hits.   │
└────────────────────────────────────────────────────────────┘
```

> 💡 **Tip:** Structure your prompts so the static portions (system message, tool definitions, few-shot examples) come first. This maximizes the cacheable prefix length and your discount.

### Model Selection & Right-Sizing

Not every request needs the most capable model. Implement a **model tiering strategy**:

| Tier | Model | Use Case | Relative Cost |
|---|---|---|---|
| **Tier 1 — Heavy** | GPT-4o | Complex reasoning, code generation, multi-step tasks | $$$$  |
| **Tier 2 — Standard** | GPT-4o mini | Summarization, classification, standard Q&A | $$ |
| **Tier 3 — Light** | GPT-4o mini (low max_tokens) | Intent detection, routing, simple extraction | $ |

```python
def select_model(task_complexity: str) -> str:
    """Route requests to the cheapest adequate model."""
    model_map = {
        "complex": "gpt-4o",          # Reasoning, code gen, analysis
        "standard": "gpt-4o-mini",    # General Q&A, summaries
        "simple": "gpt-4o-mini",      # Classification, extraction
    }
    return model_map.get(task_complexity, "gpt-4o-mini")

# Use a lightweight classifier to determine complexity
def classify_and_route(user_message: str) -> str:
    """Classify intent with a cheap model, then route to appropriate tier."""
    classification = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "Classify complexity: simple|standard|complex. Reply with one word."},
            {"role": "user", "content": user_message}
        ],
        max_tokens=5
    )
    complexity = classification.choices[0].message.content.strip().lower()
    return select_model(complexity)
```

> 🏢 **Real Example:** A legal-tech startup routing all queries through GPT-4o was spending $28,000/month. After implementing model tiering — sending 60% of queries (FAQ, simple lookup) to GPT-4o mini — monthly costs dropped to $11,200 with no measurable impact on user satisfaction scores.

---

## 4. Quota Management & Capacity Planning

### Understanding Quotas

Azure AI Foundry enforces quotas at multiple levels to ensure fair access and platform stability. Quotas limit how much throughput you can consume per model, per region, per subscription.

```
┌──────────────────────────────────────────────────────┐
│              QUOTA HIERARCHY                         │
├──────────────────────────────────────────────────────┤
│                                                      │
│  Subscription                                        │
│   └── Region (e.g., East US)                        │
│        └── Model (e.g., gpt-4o)                     │
│             ├── TPM Quota (Tokens Per Minute)        │
│             └── RPM Quota (Requests Per Minute)      │
│                  └── Deployment allocation           │
│                       (split across deployments)     │
│                                                      │
└──────────────────────────────────────────────────────┘
```

### TPM and RPM Limits

Default quota limits vary by model and region. Here are representative defaults (always verify current values in the portal):

| Model | Default TPM (Pay-as-You-Go) | Default RPM | Max Requestable TPM |
|---|---|---|---|
| `gpt-4o` | 150K | 900 | 10M+ |
| `gpt-4o-mini` | 200K | 1,200 | 10M+ |
| `gpt-4` | 80K | 480 | 1M |
| `text-embedding-ada-002` | 350K | 2,100 | 10M+ |
| `dall-e-3` | N/A | 6 | 30 |

> ⚠️ **Important:** TPM and RPM quotas are shared across **all deployments of the same model** within a region. Creating multiple deployments of `gpt-4o` in East US does not multiply your quota — it divides the same quota pool among them.

**Check your current quotas via Azure CLI:**

```bash
# List all quota allocations for your AI Foundry resource
az cognitiveservices usage list \
    --resource-group "rg-ai-foundry-prod" \
    --name "ai-foundry-hub-eastus" \
    --output table

# View model-specific quota
az cognitiveservices model list \
    --resource-group "rg-ai-foundry-prod" \
    --name "ai-foundry-hub-eastus" \
    --query "[?contains(model.name, 'gpt-4o')].{Model:model.name, Version:model.version, Format:model.format}" \
    --output table
```

### Regional Capacity Planning

Distribute deployments across regions to maximize aggregate throughput and provide redundancy:

```python
# Multi-region deployment strategy
regions = {
    "eastus":    {"gpt-4o": 300_000, "gpt-4o-mini": 500_000},   # TPM allocations
    "westus3":   {"gpt-4o": 200_000, "gpt-4o-mini": 300_000},
    "swedencentral": {"gpt-4o": 250_000, "gpt-4o-mini": 400_000},
}

total_gpt4o_tpm = sum(r["gpt-4o"] for r in regions.values())
print(f"Total gpt-4o capacity: {total_gpt4o_tpm:,} TPM across {len(regions)} regions")
# Output: Total gpt-4o capacity: 750,000 TPM across 3 regions
```

> 💡 **Tip:** Use Azure API Management or Azure AI Foundry's built-in load balancing to route traffic across regional deployments. This provides both higher aggregate throughput and regional failover.

### Requesting Quota Increases

When default quotas are insufficient:

1. Navigate to `ai.azure.com` → **Management** → **Quotas**.
2. Select the model and region where you need additional capacity.
3. Click **Request Quota Increase**.
4. Provide justification (workload description, expected TPM, timeline).
5. Submit — most increases for pay-as-you-go are approved within 1–2 business days.

**Via Azure CLI:**

```bash
# Request a quota increase (submit a support request)
az support tickets create \
    --ticket-name "Quota increase - gpt-4o East US" \
    --title "TPM quota increase for gpt-4o in East US" \
    --description "Production workload requires 1M TPM for gpt-4o. Current: 150K." \
    --problem-classification "/providers/Microsoft.Support/services/{service-id}/problemClassifications/{classification-id}" \
    --severity "minimal" \
    --contact-first-name "Your" \
    --contact-last-name "Name" \
    --contact-method "email" \
    --contact-email "you@contoso.com" \
    --contact-timezone "Pacific Standard Time" \
    --contact-language "en-us" \
    --contact-country "US"
```

### PTU Reservation Planning

For PTU deployments, capacity must be explicitly reserved:

| Planning Factor | Consideration |
|---|---|
| **Baseline load** | Measure average TPM over 2+ weeks before committing |
| **Peak multiplier** | Reserve 1.5–2× average for peak headroom |
| **Growth buffer** | Add 20–30% for anticipated growth over the commitment term |
| **Commitment term** | Monthly vs. annual — annual offers discounts up to 20% |

> 📝 **Note:** PTU reservations can take 24–48 hours to provision. Plan capacity changes ahead of expected traffic increases (marketing campaigns, product launches).

---

## 5. Cost Dashboards & Budget Alerts

### Azure Cost Management Integration

Azure AI Foundry costs flow into Azure Cost Management like any other Azure resource. Set up purpose-built views to track AI spend:

**Portal steps:**

1. Open the **Azure portal** → `portal.azure.com`.
2. Navigate to **Cost Management + Billing** → **Cost Management**.
3. Select **Cost analysis** in the left menu.
4. Set scope to your AI Foundry resource group.
5. Group by **Resource** to see per-hub costs, or by **Meter** to see token-level detail.
6. Set date range to the current billing period.
7. Click **Save** to create a reusable view, name it `AI Foundry — Monthly Spend`.

### AI Foundry Usage Analytics

Within the AI Foundry portal itself, usage metrics are available per deployment:

1. Open `ai.azure.com` → select your **Project**.
2. Navigate to **Deployments** → select a model deployment.
3. Click the **Metrics** tab.
4. Review: **Total Tokens**, **Token Rate (TPM)**, **Request Count**, **Latency (P50/P99)**.
5. Set the time range and granularity (1 hour, 1 day, 1 week).

> 💡 **Tip:** Export metrics to **Azure Monitor** for long-term retention and to build cross-deployment dashboards using **Azure Workbooks** or **Grafana**.

### Setting Up Budget Alerts

Prevent bill shock by creating proactive budget alerts:

```bash
# Create a budget for your AI Foundry resource group
az consumption budget create \
    --budget-name "ai-foundry-monthly" \
    --resource-group "rg-ai-foundry-prod" \
    --amount 10000 \
    --time-grain Monthly \
    --start-date "2026-04-01" \
    --end-date "2027-04-01" \
    --category Cost

# Add alert thresholds (50%, 80%, 100%, 120%)
# Configure via portal: Cost Management → Budgets → ai-foundry-monthly → Alert conditions
```

**Recommended alert thresholds:**

| Threshold | Action | Notification Target |
|---|---|---|
| **50%** of budget | Informational — on track | Team email |
| **80%** of budget | Warning — review usage | Team email + Slack/Teams |
| **100%** of budget | Critical — at budget limit | Team + management + action group |
| **120%** of budget | Overage — escalate immediately | All stakeholders + auto-scale down (optional) |

### Custom Cost Allocation with Tags

Tags enable precise cost attribution across teams, projects, and environments:

```bash
# Tag your AI Foundry resources for cost allocation
az resource tag \
    --resource-group "rg-ai-foundry-prod" \
    --name "ai-foundry-hub-eastus" \
    --resource-type "Microsoft.CognitiveServices/accounts" \
    --tags \
        CostCenter="CC-4200" \
        Team="customer-experience" \
        Environment="production" \
        Project="support-agent-v2"
```

Then in **Cost Management → Cost analysis**, group by tag to see spend per cost center, team, or project.

> 📝 **Note:** Apply tags consistently using **Azure Policy** with a `require tag` policy to enforce tagging at resource creation time.

---

## 6. Cost Governance Best Practices

### Resource Organization

Structure your Azure AI Foundry resources for clear cost visibility:

```
Management Group: Contoso-AI
├── Subscription: AI-Production
│   ├── rg-ai-foundry-prod-eastus
│   │   └── AI Foundry Hub (production, East US)
│   └── rg-ai-foundry-prod-westus3
│       └── AI Foundry Hub (production, West US 3)
├── Subscription: AI-Development
│   └── rg-ai-foundry-dev
│       └── AI Foundry Hub (dev/test, shared)
└── Subscription: AI-Sandbox
    └── rg-ai-foundry-sandbox
        └── AI Foundry Hub (experimentation)
```

> ⚠️ **Important:** Separate production and development into different subscriptions. This provides hard isolation for quotas, budgets, and access control — and prevents a runaway experiment from consuming production capacity.

### Chargeback & Showback Models

| Model | Description | When to Use |
|---|---|---|
| **Showback** | Report AI costs by team/project for awareness | Early stages, building cost culture |
| **Chargeback** | Allocate actual costs to team budgets | Mature organizations, enforced accountability |
| **Hybrid** | Shared platform cost + per-team usage cost | Most common for centralized AI platforms |

Implement chargeback using a combination of **tags** (per-request attribution) and **Cost Management exports**:

```python
import json
from datetime import datetime, timedelta

def generate_chargeback_report(usage_data: list[dict]) -> dict:
    """Aggregate token costs by team from usage logs."""
    team_costs = {}
    
    for record in usage_data:
        team = record.get("team", "unattributed")
        input_cost = (record["prompt_tokens"] / 1000) * record["input_rate"]
        output_cost = (record["completion_tokens"] / 1000) * record["output_rate"]
        total = input_cost + output_cost
        
        if team not in team_costs:
            team_costs[team] = {"total_cost": 0, "total_requests": 0, "total_tokens": 0}
        
        team_costs[team]["total_cost"] += total
        team_costs[team]["total_requests"] += 1
        team_costs[team]["total_tokens"] += record["prompt_tokens"] + record["completion_tokens"]
    
    return team_costs

# Example output:
# {"customer-experience": {"total_cost": 4230.50, "total_requests": 892140, "total_tokens": 1284920000},
#  "engineering":         {"total_cost": 1870.25, "total_requests": 234500, "total_tokens": 487320000}}
```

### Azure Advisor for AI Workloads

Azure Advisor provides automated cost recommendations for your AI Foundry deployments:

1. Open **Azure portal** → **Advisor** → **Cost** tab.
2. Look for recommendations such as:
   - **Right-size PTU deployments** — reduce PTUs if utilization is consistently below 40%.
   - **Switch pricing model** — move from PTU to pay-as-you-go (or vice versa) based on usage patterns.
   - **Delete unused deployments** — deployments with zero traffic for 30+ days.
3. Review each recommendation's projected savings and apply as appropriate.

### FinOps for AI

FinOps (Financial Operations) extends cloud cost management practices to AI workloads. Key principles:

| FinOps Principle | AI Application |
|---|---|
| **Visibility** | Tag every deployment, log every request with cost metadata |
| **Optimization** | Weekly review of model tiering, prompt efficiency, cache hit rates |
| **Accountability** | Teams own their AI costs, review in sprint retrospectives |
| **Benchmarking** | Track cost-per-conversation, cost-per-task, cost-per-user over time |

```python
# Key FinOps metrics to track
finops_metrics = {
    "cost_per_conversation": "Total monthly cost / Total conversations",
    "cost_per_task_completion": "Total cost / Successfully completed tasks",
    "cost_per_active_user": "Total cost / Monthly active users",
    "token_efficiency": "Useful output tokens / Total tokens consumed",
    "cache_hit_rate": "Cached token requests / Total requests",
    "model_tier_distribution": "% requests per model tier (heavy/standard/light)",
}
```

> 💡 **Tip:** The most impactful FinOps metric for AI is **cost per successful outcome** — not raw cost. A $0.05 interaction that resolves a support ticket is far more valuable than a $0.01 interaction that requires human escalation.

---

## 7. Mini Hack — Analyze Token Usage & Implement Cost Savings

**Scenario:** You manage an Azure AI Foundry project running a customer support agent. Your monthly costs have grown to $15,000 and leadership wants a 40% reduction without degrading quality. You will analyze usage, identify savings, and implement optimizations.

**Duration:** 60–90 minutes

---

### Exercise 1: Set Up Token Usage Tracking (15 min)

Create a wrapper that logs token usage for every API call:

```python
import tiktoken
import json
from datetime import datetime
from openai import AzureOpenAI

client = AzureOpenAI(
    azure_endpoint="https://<your-resource>.openai.azure.com/",
    api_version="2025-01-01-preview"
)

usage_log = []

def tracked_completion(model: str, messages: list, max_tokens: int = 500, team: str = "default") -> dict:
    """Make a completion call with full usage tracking."""
    
    # Pre-call: count input tokens
    enc = tiktoken.encoding_for_model(model)
    input_text = " ".join(m["content"] for m in messages)
    estimated_input = len(enc.encode(input_text))
    
    # Make the API call
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        max_tokens=max_tokens,
        temperature=0.3
    )
    
    # Log usage
    usage_record = {
        "timestamp": datetime.utcnow().isoformat(),
        "model": model,
        "team": team,
        "prompt_tokens": response.usage.prompt_tokens,
        "completion_tokens": response.usage.completion_tokens,
        "total_tokens": response.usage.total_tokens,
        "cached_tokens": getattr(response.usage, "prompt_tokens_details", {}).get("cached_tokens", 0),
        "max_tokens_set": max_tokens,
        "estimated_input": estimated_input,
    }
    usage_log.append(usage_record)
    
    return {
        "content": response.choices[0].message.content,
        "usage": usage_record
    }

# Test it
result = tracked_completion(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful support agent for Contoso."},
        {"role": "user", "content": "What is your return policy?"}
    ],
    team="customer-experience"
)
print(json.dumps(result["usage"], indent=2))
```

### Exercise 2: Analyze Token Waste (15 min)

Analyze your usage log to find optimization opportunities:

```python
def analyze_usage(log: list[dict]) -> dict:
    """Analyze usage log for cost optimization opportunities."""
    total_input = sum(r["prompt_tokens"] for r in log)
    total_output = sum(r["completion_tokens"] for r in log)
    total_cached = sum(r["cached_tokens"] for r in log)
    
    # Calculate output utilization (how much of max_tokens was used)
    output_utilization = []
    for r in log:
        if r["max_tokens_set"] > 0:
            utilization = r["completion_tokens"] / r["max_tokens_set"]
            output_utilization.append(utilization)
    
    avg_utilization = sum(output_utilization) / len(output_utilization) if output_utilization else 0
    cache_hit_rate = total_cached / total_input if total_input > 0 else 0
    
    analysis = {
        "total_requests": len(log),
        "total_input_tokens": total_input,
        "total_output_tokens": total_output,
        "input_output_ratio": round(total_input / max(total_output, 1), 2),
        "avg_output_utilization": f"{avg_utilization:.1%}",
        "cache_hit_rate": f"{cache_hit_rate:.1%}",
        "recommendations": []
    }
    
    # Generate recommendations
    if avg_utilization < 0.3:
        analysis["recommendations"].append(
            f"⬇️ REDUCE max_tokens — average utilization is only {avg_utilization:.0%}. "
            f"Set max_tokens to 2x the average completion length."
        )
    
    if cache_hit_rate < 0.5:
        analysis["recommendations"].append(
            "📦 IMPROVE CACHING — cache hit rate is below 50%. "
            "Ensure system prompts are identical across requests."
        )
    
    if total_input / max(total_output, 1) > 5:
        analysis["recommendations"].append(
            "✂️ COMPRESS PROMPTS — input-to-output ratio exceeds 5:1. "
            "Review system prompt length and conversation history management."
        )
    
    return analysis

# Run analysis
report = analyze_usage(usage_log)
for key, value in report.items():
    if key == "recommendations":
        print("\n📋 Recommendations:")
        for rec in value:
            print(f"  • {rec}")
    else:
        print(f"{key}: {value}")
```

### Exercise 3: Implement Model Tiering (15 min)

Build a request router that sends simple queries to a cheaper model:

```python
def smart_router(user_message: str, system_prompt: str, team: str = "default") -> dict:
    """Route requests to the most cost-effective model."""
    
    # Step 1: Classify complexity with the cheapest model
    classification = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "Classify this support query. Reply ONLY with: simple, standard, or complex."},
            {"role": "user", "content": user_message}
        ],
        max_tokens=5,
        temperature=0
    )
    complexity = classification.choices[0].message.content.strip().lower()
    
    # Step 2: Route to appropriate model with appropriate limits
    routing_config = {
        "simple":   {"model": "gpt-4o-mini", "max_tokens": 150},
        "standard": {"model": "gpt-4o-mini", "max_tokens": 300},
        "complex":  {"model": "gpt-4o",      "max_tokens": 600},
    }
    config = routing_config.get(complexity, routing_config["standard"])
    
    # Step 3: Make the routed call with tracking
    result = tracked_completion(
        model=config["model"],
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_message}
        ],
        max_tokens=config["max_tokens"],
        team=team
    )
    result["routed_complexity"] = complexity
    result["routed_model"] = config["model"]
    
    return result

# Test with different queries
queries = [
    "What are your business hours?",                    # simple
    "I want to return an item I bought last week.",     # standard
    "I was charged twice and need a refund. I also have a warranty claim for a defective product.",  # complex
]

for q in queries:
    r = smart_router(q, "You are a Contoso support agent. Be concise.")
    print(f"Query: {q[:50]}...")
    print(f"  → Complexity: {r['routed_complexity']}, Model: {r['routed_model']}")
    print(f"  → Tokens: {r['usage']['total_tokens']}\n")
```

### Exercise 4: Build a Cost Dashboard Query (15 min)

Create a script that calculates actual costs from your usage data:

```python
# Current pricing rates (replace with actual rates)
PRICING = {
    "gpt-4o":      {"input": 0.005, "output": 0.015},   # per 1K tokens
    "gpt-4o-mini": {"input": 0.00015, "output": 0.0006}, # per 1K tokens
}

def calculate_costs(log: list[dict]) -> dict:
    """Calculate actual costs from usage log."""
    costs_by_model = {}
    costs_by_team = {}
    
    for record in log:
        model = record["model"]
        team = record["team"]
        rates = PRICING.get(model, PRICING["gpt-4o-mini"])
        
        input_cost = (record["prompt_tokens"] / 1000) * rates["input"]
        output_cost = (record["completion_tokens"] / 1000) * rates["output"]
        total_cost = input_cost + output_cost
        
        # Aggregate by model
        if model not in costs_by_model:
            costs_by_model[model] = {"cost": 0, "requests": 0, "tokens": 0}
        costs_by_model[model]["cost"] += total_cost
        costs_by_model[model]["requests"] += 1
        costs_by_model[model]["tokens"] += record["total_tokens"]
        
        # Aggregate by team
        if team not in costs_by_team:
            costs_by_team[team] = {"cost": 0, "requests": 0}
        costs_by_team[team]["cost"] += total_cost
        costs_by_team[team]["requests"] += 1
    
    return {
        "by_model": costs_by_model,
        "by_team": costs_by_team,
        "total_cost": sum(m["cost"] for m in costs_by_model.values()),
        "total_requests": sum(m["requests"] for m in costs_by_model.values()),
    }

cost_report = calculate_costs(usage_log)
print(f"\n💰 Total cost: ${cost_report['total_cost']:.4f}")
print(f"📊 Total requests: {cost_report['total_requests']}")
print("\nCost by model:")
for model, data in cost_report["by_model"].items():
    print(f"  {model}: ${data['cost']:.4f} ({data['requests']} requests, {data['tokens']:,} tokens)")
```

### ✅ Mini Hack Checklist

- [ ] Token usage tracker wrapper is functional and logging all API calls.
- [ ] Usage analysis identifies at least 2 optimization recommendations.
- [ ] Model tiering router correctly classifies and routes queries to different models.
- [ ] Cost calculation script produces accurate per-model and per-team breakdowns.
- [ ] You can articulate a plan to achieve the 40% cost reduction target.

---

## 8. Key Takeaways

1. **Pricing model selection is foundational.** PTU for high-volume production, pay-as-you-go for everything else. Run a break-even analysis before committing.
2. **Token optimization delivers the fastest ROI.** Compact system prompts, enforce `max_tokens`, and leverage prompt caching for an immediate 30–60% reduction.
3. **Model tiering is essential.** Not every request needs the most capable model. Route simple tasks to cheaper models — most workloads are 50–70% simple queries.
4. **Quotas are per-model, per-region, per-subscription.** Monitor utilization, plan multi-region deployments, and request increases proactively.
5. **Budget alerts prevent surprises.** Set alerts at 50%, 80%, 100%, and 120% thresholds with escalating notification severity.
6. **Tags power cost governance.** Without consistent tagging, you cannot implement chargeback, showback, or meaningful cost analysis.
7. **FinOps is continuous.** Track cost-per-outcome weekly, review model tiering monthly, and renegotiate PTU commitments quarterly.

---

## 9. Looking Ahead — Module 31 Preview

🎉 **Congratulations!** You have completed **ARC 6: Deployment & MLOps** and the entire **Intermediate (🟡) tier** of the Zero to Hero training.

**Module 31: Advanced Prompt Engineering Patterns** begins the **Advanced (🔴) tier**, where we tackle sophisticated techniques used in production-grade AI systems:

- **🔴 Advanced tier begins!** — Modules 31–45 assume solid fluency with all concepts from the Beginner and Intermediate tiers.
- Multi-agent orchestration prompt design.
- Chain-of-thought, tree-of-thought, and graph-of-thought patterns.
- Dynamic prompt assembly and meta-prompting.
- Adversarial robustness and prompt hardening.

> 📝 **Note:** Before starting Module 31, review your notes from Modules 1–30. The Advanced tier builds directly on every concept covered so far — there are no refresher sections.

---

## 📚 Additional Resources

| Resource | Link |
|---|---|
| **Azure AI Foundry Pricing** | [azure.microsoft.com/pricing/details/ai-foundry](https://azure.microsoft.com/pricing/details/cognitive-services/openai-service/) |
| **Azure Cost Management Docs** | [learn.microsoft.com/azure/cost-management-billing](https://learn.microsoft.com/azure/cost-management-billing/) |
| **PTU Onboarding Guide** | [learn.microsoft.com/azure/ai-services/openai/concepts/provisioned-throughput](https://learn.microsoft.com/azure/ai-services/openai/concepts/provisioned-throughput) |
| **Quota Management** | [learn.microsoft.com/azure/ai-services/openai/quotas-limits](https://learn.microsoft.com/azure/ai-services/openai/quotas-limits) |
| **tiktoken Library** | [github.com/openai/tiktoken](https://github.com/openai/tiktoken) |
| **Azure Advisor** | [learn.microsoft.com/azure/advisor](https://learn.microsoft.com/azure/advisor/) |
| **FinOps Foundation** | [finops.org](https://www.finops.org/) |
| **Azure Pricing Calculator** | [azure.microsoft.com/pricing/calculator](https://azure.microsoft.com/pricing/calculator/) |

---

**End of Module 30 — Cost Optimization & Quota Management**

*ARC 6: Deployment & MLOps — Complete ✅*
*Next: ARC 7: Advanced Patterns (Modules 31–36) — 🔴 Advanced Tier*
