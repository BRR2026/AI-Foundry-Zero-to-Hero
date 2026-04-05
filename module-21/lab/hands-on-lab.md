# Lab 21: Prompt Flow — Build a Customer Support Flow with Routing Logic

| | |
|---|---|
| **Estimated Time** | 60–75 minutes |
| **Difficulty** | ⭐⭐ Intermediate |
| **Module** | 21 of 45 |
| **Arc** | ARC 5 — Orchestration & Integration |

## Prerequisites

| Requirement | Details |
|---|---|
| Azure AI Foundry project | Active project with compute quota |
| Azure OpenAI connection | Configured in your AI Foundry project |
| GPT-4o deployment | Deployed in your Azure OpenAI resource |
| GPT-4o-mini deployment | Deployed for intent classification (cost-effective) |
| Web browser | Microsoft Edge or Chrome (latest version) |

> **💡 Note:** This lab is performed entirely in the Azure AI Foundry portal at [ai.azure.com](https://ai.azure.com). No local development setup is required. Ensure your compute session is started before beginning — it takes 1–2 minutes to initialize.

---

## Learning Objectives

By the end of this lab you will be able to:

- ✅ Create a new Standard Flow from scratch in Azure AI Foundry
- ✅ Configure inputs, outputs, and node connections
- ✅ Build LLM nodes with Jinja2 prompt templates
- ✅ Write Python tool nodes with the `@tool` decorator
- ✅ Implement intent classification and routing logic
- ✅ Test flows interactively and with batch datasets
- ✅ Debug node-level issues using execution traces
- ✅ Deploy a completed flow as a managed online endpoint

---

## Architecture Overview

In this lab, you'll build a customer support flow with the following architecture:

```
┌──────────────┐     ┌──────────────────┐     ┌────────────────────┐
│   INPUTS     │────▶│ CLASSIFY INTENT  │────▶│  ROUTE BY INTENT   │
│  - message   │     │  (LLM: GPT-4o-  │     │  (Python: routing  │
│  - customer_ │     │   mini)          │     │   config)          │
│    id        │     └──────────────────┘     └─────────┬──────────┘
└──────────────┘                                        │
                                               ┌────────▼──────────┐
                                               │ GENERATE RESPONSE │
                                               │ (LLM: GPT-4o,    │
                                               │  dynamic prompt)  │
                                               └────────┬──────────┘
                                               ┌────────▼──────────┐
                                               │  FORMAT OUTPUT    │
                                               │  (Python: meta-   │
                                               │   data + JSON)    │
                                               └────────┬──────────┘
                                               ┌────────▼──────────┐
                                               │    OUTPUTS        │
                                               │  - response       │
                                               │  - metadata       │
                                               └───────────────────┘
```

---

## Exercise 1: Create the Flow and Define Inputs (10 min)

### Step 1.1 — Navigate to Prompt Flow

1. Sign in to [Azure AI Foundry](https://ai.azure.com)
2. Select your project from the project list
3. In the left navigation, under **Build & Customize**, click **Prompt Flow**
4. Click **+ Create** at the top of the page

### Step 1.2 — Create a Standard Flow

1. Select **Standard Flow** from the flow type options
2. Choose **Create from blank** (not from template)
3. Enter the name: `customer-support-routing`
4. Click **Create**

### Step 1.3 — Configure Flow Inputs

1. In the flow editor, locate the **Inputs** section at the top
2. Add the following inputs:

| Input Name | Type | Default Value |
|---|---|---|
| `message` | `string` | `"I was charged twice for my subscription this month"` |
| `customer_id` | `string` | `"CUST-12345"` |

3. Verify the inputs appear in the flow graph as the starting node

### Step 1.4 — Configure Flow Outputs

1. Scroll to the **Outputs** section at the bottom
2. Add these outputs (you'll connect them to nodes later):

| Output Name | Type |
|---|---|
| `response` | `string` |
| `metadata` | `string` |

> **💡 Tip:** We'll set the output references after creating all nodes. Leave the reference fields empty for now.

### ✅ Checkpoint

- [ ] Flow is created and named `customer-support-routing`
- [ ] Two inputs are defined: `message` (string) and `customer_id` (string)
- [ ] Two outputs are defined: `response` and `metadata`
- [ ] The flow editor canvas is visible with the Inputs node

---

## Exercise 2: Build the Intent Classification Node (15 min)

### Step 2.1 — Add an LLM Node

1. Click **+ Add Node** in the flow toolbar
2. Select **LLM** as the node type
3. Name the node: `classify_intent`

### Step 2.2 — Configure the LLM Connection

1. In the node's settings panel on the right:
   - **Connection:** Select your Azure OpenAI connection
   - **API:** Chat
   - **deployment_name:** Select `gpt-4o-mini` (fast and cost-effective for classification)
2. Set parameters:
   - **temperature:** `0` (deterministic classification)
   - **max_tokens:** `20` (only need a single word)
   - **top_p:** `1.0`

### Step 2.3 — Write the Classification Prompt

In the prompt template editor, enter:

```jinja2
system:
You are an intent classifier for a customer support system.
Classify the customer message into exactly ONE of these categories:
- billing: Payment issues, invoices, charges, subscriptions, pricing, refunds
- technical: Bugs, errors, setup, configuration, integration, API issues
- general: Account information, feedback, general inquiries, feature requests

Rules:
- Respond with ONLY the category name in lowercase
- Do not include any explanation or punctuation
- If uncertain, classify as "general"

user:
Customer message: {{message}}
```

### Step 2.4 — Map Input Variables

In the node's input mappings:

| Template Variable | Source |
|---|---|
| `message` | `${inputs.message}` |

### Step 2.5 — Test the Classification Node

1. Click the **Run** button (▶) at the top of the editor
2. Use the default input: `"I was charged twice for my subscription this month"`
3. Verify the output is: `billing`
4. Test with other inputs:
   - `"My API calls are returning 500 errors"` → should output `technical`
   - `"How do I update my email address?"` → should output `general`

### ✅ Checkpoint

- [ ] LLM node `classify_intent` is created and connected to inputs
- [ ] GPT-4o-mini is selected with temperature 0
- [ ] Classification prompt correctly identifies billing/technical/general
- [ ] At least 3 test cases pass correctly

---

## Exercise 3: Build the Routing Logic Node (15 min)

### Step 3.1 — Add a Python Node

1. Click **+ Add Node** → Select **Python**
2. Name the node: `route_by_intent`

### Step 3.2 — Write the Routing Code

Replace the default code with:

```python
from promptflow import tool
import json

@tool
def route_by_intent(intent: str, message: str, customer_id: str) -> str:
    """Route customer message to specialized handler based on intent."""
    
    # Normalize the intent
    intent = intent.strip().lower()
    
    # Define handler configurations
    routing_config = {
        "billing": {
            "handler": "billing_specialist",
            "system_prompt": (
                "You are a billing specialist for a SaaS company. "
                "Help customers with payment issues, invoice questions, "
                "subscription management, and refund requests. "
                "Be empathetic about billing frustrations. "
                "Always offer to escalate to a supervisor if the "
                "customer is unhappy with the resolution."
            ),
            "priority": "high",
            "sla_minutes": 15
        },
        "technical": {
            "handler": "technical_support",
            "system_prompt": (
                "You are a technical support engineer. "
                "Help customers diagnose issues, provide step-by-step "
                "troubleshooting instructions, and suggest solutions. "
                "Include relevant error codes, documentation links, "
                "and escalation paths for complex issues."
            ),
            "priority": "medium",
            "sla_minutes": 30
        },
        "general": {
            "handler": "general_support",
            "system_prompt": (
                "You are a friendly customer support agent. "
                "Answer general questions about the service, "
                "provide account information, help with feature "
                "requests, and direct customers to appropriate "
                "resources or documentation."
            ),
            "priority": "low",
            "sla_minutes": 60
        }
    }
    
    # Get config (default to general if intent unknown)
    config = routing_config.get(intent, routing_config["general"])
    
    # Build routing result
    result = {
        "intent": intent,
        "system_prompt": config["system_prompt"],
        "handler": config["handler"],
        "priority": config["priority"],
        "sla_minutes": config["sla_minutes"],
        "message": message,
        "customer_id": customer_id
    }
    
    return json.dumps(result)
```

### Step 3.3 — Map Input Variables

| Input | Source |
|---|---|
| `intent` | `${classify_intent.output}` |
| `message` | `${inputs.message}` |
| `customer_id` | `${inputs.customer_id}` |

### Step 3.4 — Test the Routing Node

1. Run the flow end-to-end
2. Click the `route_by_intent` node to inspect its output
3. Verify the output JSON contains:
   - Correct intent classification
   - Appropriate system prompt for the handler
   - Priority level
   - SLA minutes

### ✅ Checkpoint

- [ ] Python node `route_by_intent` is created
- [ ] Routing logic handles billing, technical, and general intents
- [ ] Unknown intents fall back to "general"
- [ ] Output is a valid JSON string with all required fields

---

## Exercise 4: Build the Response Generation Node (10 min)

### Step 4.1 — Add a Second LLM Node

1. Click **+ Add Node** → Select **LLM**
2. Name the node: `generate_response`

### Step 4.2 — Configure the LLM

1. **Connection:** Same Azure OpenAI connection
2. **deployment_name:** `gpt-4o` (higher quality for customer-facing responses)
3. Set parameters:
   - **temperature:** `0.7`
   - **max_tokens:** `500`

### Step 4.3 — Write the Response Prompt

```jinja2
system:
{{system_prompt}}

Customer ID: {{customer_id}}
Priority: {{priority}}

Guidelines:
- Be empathetic and professional
- Provide clear, actionable next steps
- Include relevant reference numbers when applicable
- Offer to escalate if the issue requires it
- Keep response concise but thorough (2-3 paragraphs max)

user:
{{message}}
```

### Step 4.4 — Map Input Variables

You need to parse the JSON output from the routing node. Since the routing node returns a JSON string, map the variables as follows:

| Template Variable | Source |
|---|---|
| `system_prompt` | Parse from `${route_by_intent.output}` |
| `customer_id` | `${inputs.customer_id}` |
| `priority` | Parse from `${route_by_intent.output}` |
| `message` | `${inputs.message}` |

> **💡 Tip:** If direct JSON parsing is complex, add a small Python node between routing and response generation to extract individual fields from the JSON string.

### Step 4.5 — (Optional) Add a Parse Node

If you need to extract fields from the routing JSON, add a Python node named `parse_routing`:

```python
from promptflow import tool
import json

@tool
def parse_routing(routing_json: str) -> dict:
    """Parse routing JSON into individual fields."""
    data = json.loads(routing_json)
    return data
```

Map its input: `routing_json` → `${route_by_intent.output}`

Then update the `generate_response` LLM node mappings to use `${parse_routing.output.system_prompt}`, etc.

### ✅ Checkpoint

- [ ] LLM node `generate_response` is created with GPT-4o
- [ ] Dynamic system prompt is sourced from the routing node
- [ ] Response is contextual and appropriate for the classified intent
- [ ] Response tone matches the intent (empathetic for billing, precise for technical)

---

## Exercise 5: Build the Output Formatting Node (10 min)

### Step 5.1 — Add a Python Node

1. Click **+ Add Node** → Select **Python**
2. Name the node: `format_output`

### Step 5.2 — Write the Formatting Code

```python
from promptflow import tool
import json
from datetime import datetime, timezone

@tool
def format_output(llm_response: str, routing_json: str) -> dict:
    """Format the final response with metadata."""
    
    # Parse routing info
    try:
        routing = json.loads(routing_json)
    except json.JSONDecodeError:
        routing = {"intent": "unknown", "handler": "general", "priority": "low"}
    
    # Build structured response
    result = {
        "response": llm_response.strip(),
        "metadata": {
            "intent": routing.get("intent", "unknown"),
            "handler": routing.get("handler", "general_support"),
            "priority": routing.get("priority", "low"),
            "sla_minutes": routing.get("sla_minutes", 60),
            "customer_id": routing.get("customer_id", ""),
            "timestamp": datetime.now(timezone.utc).isoformat(),
            "model": "gpt-4o",
            "flow_version": "1.0.0"
        }
    }
    
    return result
```

### Step 5.3 — Map Input Variables

| Input | Source |
|---|---|
| `llm_response` | `${generate_response.output}` |
| `routing_json` | `${route_by_intent.output}` |

### Step 5.4 — Connect Outputs

Now go back to the flow **Outputs** section and set the references:

| Output | Reference |
|---|---|
| `response` | `${format_output.output.response}` |
| `metadata` | `${format_output.output.metadata}` |

### ✅ Checkpoint

- [ ] Python node `format_output` is created
- [ ] Output includes both response text and structured metadata
- [ ] Flow outputs are connected to the format node
- [ ] Metadata includes intent, handler, priority, timestamp, and customer_id

---

## Exercise 6: Test and Debug the Complete Flow (10 min)

### Step 6.1 — Run End-to-End Test

1. Click **Run** (▶) at the top of the editor
2. Use default inputs or enter:
   - **message:** `"I was charged twice for my subscription this month"`
   - **customer_id:** `"CUST-12345"`
3. Wait for all nodes to complete (observe the execution flow)

### Step 6.2 — Inspect Each Node

Click each node to verify:

| Node | Expected Output |
|---|---|
| `classify_intent` | `"billing"` |
| `route_by_intent` | JSON with billing handler config |
| `generate_response` | Empathetic billing support response |
| `format_output` | Complete response + metadata dict |

### Step 6.3 — Test with Multiple Scenarios

Run the flow with these test inputs:

| # | Message | Expected Intent |
|---|---|---|
| 1 | "I was charged twice for my subscription this month" | billing |
| 2 | "My API calls are returning 500 errors since yesterday" | technical |
| 3 | "How do I update my email address on file?" | general |
| 4 | "Can I get a refund for last month's overcharge?" | billing |
| 5 | "The dashboard is loading very slowly" | technical |

### Step 6.4 — Debug Any Issues

If a test fails, use this debugging workflow:

1. **Check the Trace view** — click the trace icon to see execution details
2. **Inspect node inputs** — verify `${node.output}` references resolve correctly
3. **Check API connectivity** — ensure Azure OpenAI connection is active
4. **Review token usage** — ensure prompts fit within model context limits
5. **Test nodes individually** — right-click a node and select "Run from here"

### ✅ Checkpoint

- [ ] All 5 test scenarios produce correct intent classification
- [ ] Routing assigns the correct handler for each intent
- [ ] Response tone matches the classified intent
- [ ] Output metadata is complete and correctly structured
- [ ] No errors in the execution trace

---

## Exercise 7: Deploy as a Managed Endpoint (10 min)

### Step 7.1 — Prepare for Deployment

1. Run a final batch test to confirm flow stability
2. Click **Deploy** in the flow editor toolbar

### Step 7.2 — Configure the Endpoint

1. **Endpoint name:** `customer-support-flow-endpoint`
2. **Deployment name:** `v1`
3. **Virtual machine:** `Standard_DS2_v2` (for development/testing)
4. **Instance count:** `1`
5. **Authentication:** Key-based (for testing; use Azure AD for production)

### Step 7.3 — Deploy

1. Click **Deploy** and wait for provisioning (typically 5–10 minutes)
2. Monitor the deployment status in the notification panel

### Step 7.4 — Test the Deployed Endpoint

Once deployed:

1. Navigate to the endpoint in **Deployments** section
2. Click the **Test** tab
3. Enter a test payload:

```json
{
    "message": "I need help understanding my latest invoice",
    "customer_id": "CUST-67890"
}
```

4. Verify the response matches expected behavior

### Step 7.5 — Get the Endpoint URL

1. Click the **Consume** tab
2. Copy the **REST endpoint URL** and **Primary key**
3. Test with cURL:

```bash
curl -X POST "<your-endpoint-url>" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <your-api-key>" \
  -d '{
    "message": "My integration is throwing authentication errors",
    "customer_id": "CUST-99999"
  }'
```

### ✅ Checkpoint

- [ ] Flow is deployed successfully as a managed endpoint
- [ ] Endpoint responds to test requests in the portal
- [ ] REST API call returns the expected JSON structure
- [ ] Response includes both the answer and metadata

---

## 🎉 Lab Complete!

### What You Built

You created a complete **customer support flow** with:

- **Intent classification** using GPT-4o-mini for fast, cost-effective categorization
- **Dynamic routing** that selects specialized handlers based on intent
- **Context-aware responses** generated by GPT-4o with intent-specific system prompts
- **Structured output** with metadata for downstream integration
- **Production deployment** as a managed REST endpoint

### Clean Up Resources

To avoid unnecessary charges:

1. Navigate to the endpoint in the **Deployments** section
2. Click **Delete** to remove the deployed endpoint
3. The flow definition is retained in your project for future use

> ⚠️ **Important:** Deployed endpoints incur compute costs even when idle. Always delete test endpoints when you're done.

---

## ⚠️ Troubleshooting

| Issue | Solution |
|---|---|
| Compute session not starting | Check quota in subscription settings; try a different region |
| LLM node shows "connection error" | Verify Azure OpenAI connection in project settings |
| Python node import fails | Add missing packages to the flow's `requirements.txt` |
| Deployment fails with quota error | Request quota increase or use a smaller VM size |
| Output format unexpected | Check `${node.output}` references match actual output structure |
| Classification returns wrong intent | Lower temperature to 0; add more examples to the prompt |
| Slow response times | Use GPT-4o-mini for classification; cache routing configs |

---

## 🚀 Bonus Challenges

1. **Add Content Safety:** Insert a Content Safety node before the final output to filter harmful content
2. **Add Sentiment Analysis:** Add a Python node that detects customer sentiment and adjusts response tone
3. **Add Escalation Logic:** If sentiment is "angry" and priority is "high", add an escalation flag to metadata
4. **Create an Evaluation Flow:** Build a separate evaluation flow that measures response relevance and empathy
5. **A/B Test Prompts:** Create variants of the response generation prompt and batch-test them

---

*Lab 21 of 45 — Zero to Hero: Azure AI Foundry Training · ARC 5: Orchestration & Integration · April 2026*
