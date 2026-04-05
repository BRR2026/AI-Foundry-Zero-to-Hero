# Module 21: Prompt Flow — Visual Orchestration

## **ARC 5: ORCHESTRATION & INTEGRATION** | Module 21 of 45 — Zero to Hero: Azure AI Foundry

---

| Detail              | Info                                                        |
|---------------------|-------------------------------------------------------------|
| **Arc**             | ARC 5 — Orchestration & Integration (Modules 19–24)        |
| **Module**          | 21                                                          |
| **Title**           | Prompt Flow — Visual Orchestration                          |
| **Estimated Time**  | 3–4 hours                                                   |
| **Difficulty**      | ⭐⭐ Intermediate                                           |
| **Last Updated**    | April 2026                                                  |

---

## 🎯 Learning Objectives

By the end of this module, you will be able to:

1. **Explain** what Prompt Flow is and how it fits into the Azure AI Foundry ecosystem
2. **Build** multi-step flows using inputs, LLM nodes, Python nodes, and outputs
3. **Test** flows interactively and with batch datasets to validate quality
4. **Debug** node-level issues using traces, variant comparison, and execution logs
5. **Deploy** completed flows as managed online endpoints with auto-scaling
6. **Design** routing logic in flows that classifies and directs requests to specialized handlers

---

## 📋 Prerequisites

- Completed **Module 20: SDK & API Integration Patterns** (or equivalent experience)
- Active **Azure AI Foundry project** with an Azure OpenAI connection
- A deployed **GPT-4o or GPT-4o-mini** model in your project
- Basic familiarity with **Python** and **Jinja2 templates**
- Understanding of **REST APIs** and **JSON data formats**

> 💡 **Tip:** If you haven't set up an AI Foundry project yet, refer to Module 2 for step-by-step setup instructions. Prompt Flow requires a compute session — make sure your subscription has sufficient quota.

---

## 📖 Module Overview

This module covers the complete lifecycle of building AI workflows with Prompt Flow in Azure AI Foundry. We move from understanding what Prompt Flow is and when to use it, through building and testing flows, to deploying production-ready endpoints.

### Module Flow

```
Understanding Prompt Flow → Building Flows → Testing & Debugging → Deployment → Mini Hack
```

### Key Concepts Map

```
Prompt Flow
├── Flow Types
│   ├── Standard Flow (general LLM apps)
│   ├── Chat Flow (conversational AI)
│   └── Evaluation Flow (quality assessment)
├── Node Types
│   ├── LLM Node (model calls with Jinja2 prompts)
│   ├── Python Node (custom code with @tool decorator)
│   ├── Prompt Node (template rendering)
│   ├── Embedding Node (vector embeddings)
│   ├── Content Safety Node (harm filtering)
│   └── Index Lookup Node (retrieval)
├── Testing
│   ├── Interactive single-input testing
│   ├── Batch testing with datasets
│   └── Variant comparison
└── Deployment
    ├── Managed online endpoints
    ├── Auto-scaling configuration
    └── Monitoring & tracing
```

---

## 1. Introduction — The Orchestration Challenge

### Why Orchestration Matters

Real-world AI applications are never just a single LLM call. Consider a customer support chatbot:

1. The user sends a message
2. The system classifies the intent
3. It retrieves relevant knowledge base articles
4. It generates a response using the appropriate context
5. It validates the response for safety and accuracy
6. It formats the output and logs the interaction

Each of these steps involves different logic — some need LLM calls, some need Python code, some need API calls. **Prompt Flow** provides a visual, graph-based way to compose these steps into a reliable, testable, deployable pipeline.

### What is Prompt Flow?

Prompt Flow is a **development and runtime tool** within Azure AI Foundry that enables you to:

- **Design** AI workflows as directed acyclic graphs (DAGs)
- **Prototype** rapidly with a visual drag-and-drop editor
- **Test** with both interactive and batch modes
- **Debug** with node-level inspection and execution traces
- **Deploy** as managed endpoints with a single click

### Key Teaching Points

- **Prompt Flow is NOT Azure ML pipelines** — it is purpose-built for LLM orchestration within Azure AI Foundry
- **Prompt Flow is NOT a replacement for LangChain/Semantic Kernel** — it complements these frameworks; you can use them *inside* Python nodes
- **Prompt Flow IS a visual IDE** for composing LLM-based application logic
- **Prompt Flow IS an open-source SDK** (`promptflow`) that can run locally or in the cloud

> 💡 **Teaching Note:** Students often confuse Prompt Flow with other pipeline tools. Emphasize that Prompt Flow is specifically designed for LLM-based application orchestration, not for data processing or model training pipelines.

---

## 2. Understanding Flow Types

### Standard Flow

Standard flows are the most flexible type. They accept defined inputs, process them through a DAG of nodes, and return defined outputs. Use for:

- Q&A systems
- Content generation pipelines
- Document processing workflows
- Classification and extraction tasks

### Chat Flow

Chat flows extend standard flows with built-in **chat history** management. They automatically track conversation turns and pass history to nodes. Use for:

- Multi-turn chatbots
- Customer support agents
- Interactive assistants

### Evaluation Flow

Evaluation flows are specialized flows designed to measure the quality of other flows' outputs. They receive `groundtruth` and `prediction` inputs and compute metrics. Use for:

- Measuring groundedness, relevance, coherence
- Comparing prompt variants
- Regression testing across flow versions

### Decision Matrix

| Scenario | Recommended Flow Type |
|---|---|
| Single-turn Q&A | Standard Flow |
| Multi-turn conversation | Chat Flow |
| Quality measurement | Evaluation Flow |
| RAG pipeline | Standard Flow |
| Intent classification | Standard Flow |
| Chatbot with memory | Chat Flow |

---

## 3. Building Flows — Node by Node

### 3.1 Flow Structure

Every flow consists of:

1. **Inputs** — Typed parameters that the flow receives
2. **Nodes** — Processing steps (LLM, Python, Prompt, etc.)
3. **Connections** — Data flow between nodes via `${node_name.output}` references
4. **Outputs** — Named results that the flow returns

The flow definition is stored as a YAML file (`flow.dag.yaml`) alongside node source files.

### 3.2 Input Configuration

Inputs define the flow's contract with callers. Each input has:

- **Name** — Identifier used in node references
- **Type** — `string`, `int`, `double`, `bool`, `list`, `object`
- **Default** — Optional default value for testing

```yaml
inputs:
  question:
    type: string
    default: "What is Azure AI Foundry?"
  context:
    type: string
    default: ""
  chat_history:
    type: list
    default: []
```

### 3.3 LLM Nodes

LLM nodes are the core of most flows. They:

1. **Connect** to an Azure OpenAI deployment (or other LLM endpoint)
2. **Render** a Jinja2 prompt template with flow variables
3. **Call** the model with configurable parameters
4. **Return** the model's response as the node output

**Jinja2 Template Example:**

```jinja2
system:
You are a helpful assistant. Answer based on the provided context.

user:
Context: {{context}}

Question: {{question}}

Provide a clear, accurate answer.
```

**Configuration:**

| Parameter | Description | Typical Value |
|---|---|---|
| Connection | Azure OpenAI resource | *your-aoai-connection* |
| API | Chat or Completion | Chat |
| deployment_name | Model deployment | gpt-4o |
| temperature | Randomness (0–2) | 0.7 |
| max_tokens | Maximum response length | 1000 |
| top_p | Nucleus sampling | 0.95 |

### 3.4 Python Nodes

Python nodes execute custom code. They must:

1. Import `from promptflow import tool`
2. Define a function decorated with `@tool`
3. Use type hints for all parameters
4. Return a single value (string, dict, list, etc.)

```python
from promptflow import tool
from typing import List

@tool
def process_data(raw_input: str, history: List[dict]) -> str:
    """Process and clean input data."""
    cleaned = raw_input.strip().lower()
    context_from_history = "\n".join(
        [h.get("content", "") for h in history[-3:]]
    )
    return f"{cleaned}\n\nRecent context:\n{context_from_history}"
```

**Best Practices for Python Nodes:**

- ✅ Keep functions focused — one responsibility per node
- ✅ Use type hints for all parameters and return values
- ✅ Handle exceptions gracefully with try/except
- ✅ Return structured data (dicts, JSON strings) for downstream consumption
- ❌ Don't embed API keys — use connections
- ❌ Don't make nodes too large — if code exceeds ~50 lines, consider splitting

### 3.5 Output Configuration

Outputs define what the flow returns to callers. Each output references a specific node's output:

```yaml
outputs:
  answer:
    type: string
    reference: ${generate_answer.output}
  metadata:
    type: string
    reference: ${format_metadata.output}
```

---

## 4. Testing and Debugging Flows

### 4.1 Interactive Testing

The flow editor includes a built-in test panel:

1. Click **Run** at the top of the editor
2. Enter test values in the input fields
3. Observe real-time execution — each node lights up as it runs
4. Click any node to inspect its input/output
5. View execution time and token usage in the **Trace** panel

### 4.2 Variant Testing

Variants allow you to test different versions of a node side-by-side:

- Create a variant by clicking the variant icon on an LLM node
- Modify the prompt template, model, or parameters
- Run both variants and compare outputs in a table view
- Use batch runs to statistically compare variant performance

### 4.3 Batch Testing

For systematic evaluation:

1. Prepare a dataset (JSONL or CSV) with test cases
2. Include expected outputs for automated comparison
3. Click **Batch Run** and select your dataset
4. Review results in the run dashboard
5. Export metrics for further analysis

### 4.4 Debugging Strategies

| Issue | Debugging Approach |
|---|---|
| Wrong output from LLM | Inspect prompt template rendering in trace view |
| Node execution error | Check node logs for Python exceptions or API errors |
| Slow execution | Review per-node timing in execution trace |
| Type mismatch | Verify output types match downstream input expectations |
| Missing variables | Check that `${node.output}` references are correct |

> ⚠️ **Common Pitfall:** Students often forget to start a compute session before testing. Remind them to check the compute status indicator in the top-right corner of the flow editor.

---

## 5. Deploying Flows as Endpoints

### 5.1 Deployment Steps

1. **Validate** — Run a final batch test to confirm flow quality
2. **Click Deploy** — In the flow editor toolbar
3. **Configure** — Set endpoint name, compute size, instance count
4. **Authentication** — Choose key-based or Azure AD authentication
5. **Deploy** — Wait for provisioning (typically 5–10 minutes)
6. **Test** — Validate the deployed endpoint with the test tab

### 5.2 Compute Configuration

| Setting | Development | Production |
|---|---|---|
| Instance type | Standard_DS2_v2 | Standard_DS3_v2+ |
| Instance count | 1 | 2–5 (with auto-scale) |
| Min instances | 0 | 1 |
| Max instances | 1 | 10 |
| Scale-up threshold | N/A | 70% CPU |

### 5.3 Monitoring

Deployed endpoints provide:

- **Latency metrics** — P50, P95, P99 response times
- **Error rates** — Failed requests and error codes
- **Token usage** — Model token consumption per request
- **Request logs** — Full request/response payloads (configurable)

### 5.4 Versioning and Traffic Splitting

For production deployments:

- Deploy new flow versions as separate deployments under the same endpoint
- Use **traffic splitting** to gradually shift traffic (e.g., 90/10 → 50/50 → 0/100)
- Monitor metrics during the rollout
- Roll back instantly by redirecting traffic to the previous deployment

---

## 6. Mini Hack — Customer Support Flow with Routing

### Objective

Build a Prompt Flow that:

1. Accepts a customer message and customer ID
2. Classifies the message intent (billing, technical, general)
3. Routes to a specialized response handler based on intent
4. Generates a contextual response with metadata
5. Validates the response before returning

### Architecture

```
Input (message, customer_id)
    │
    ▼
[Classify Intent] ─── LLM Node (GPT-4o-mini)
    │
    ▼
[Route by Intent] ─── Python Node (routing logic)
    │
    ▼
[Generate Response] ─── LLM Node (GPT-4o, dynamic system prompt)
    │
    ▼
[Format Output] ─── Python Node (add metadata, timestamp)
    │
    ▼
Output (response, intent, metadata)
```

### Implementation Steps

#### Step 1 — Create a Standard Flow

1. Navigate to Prompt Flow in your AI Foundry project
2. Create a new Standard Flow from blank
3. Define inputs: `message` (string), `customer_id` (string)

#### Step 2 — Add Intent Classification Node

- Add an LLM node named `classify_intent`
- Use GPT-4o-mini for fast, cost-effective classification
- Set temperature to 0 for deterministic classification
- Prompt should classify into: billing, technical, general

#### Step 3 — Add Routing Node

- Add a Python node named `route_by_intent`
- Input: `${classify_intent.output}`, `${inputs.message}`, `${inputs.customer_id}`
- Return a dict with: `system_prompt`, `message`, `customer_id`, `priority`, `handler`

#### Step 4 — Add Response Generation Node

- Add an LLM node named `generate_response`
- Use GPT-4o for high-quality responses
- Map routing output fields to template variables
- Dynamic system prompt from the routing node

#### Step 5 — Add Output Formatting Node

- Add a Python node named `format_output`
- Combine LLM response with routing metadata
- Add timestamp and structured JSON output

#### Step 6 — Test with Sample Inputs

Test with these scenarios:

| Test Input | Expected Intent | Expected Behavior |
|---|---|---|
| "I was charged twice for my subscription" | billing | Billing-specific response with empathy |
| "My API calls are returning 500 errors" | technical | Technical troubleshooting steps |
| "How do I update my email address?" | general | Account management guidance |

### Success Criteria

- ✅ Flow correctly classifies at least 90% of test messages
- ✅ Routing logic assigns the correct handler for each intent
- ✅ Response tone matches the intent (empathetic for billing, precise for technical)
- ✅ Output includes structured metadata (intent, handler, timestamp)
- ✅ Flow deploys successfully as a managed endpoint

---

## 7. Instructor Notes

### Pacing Guide

| Section | Time | Activity |
|---|---|---|
| Introduction & Concepts | 30 min | Lecture + demo |
| Building Flows | 45 min | Guided walkthrough |
| Node Types Deep Dive | 30 min | Code-along |
| Testing & Debugging | 30 min | Interactive demo |
| Deployment | 20 min | Live deployment |
| Mini Hack | 60 min | Hands-on lab |
| Review & Q&A | 15 min | Discussion |

### Common Student Questions

1. **"How is this different from LangChain?"** — Prompt Flow is a visual orchestration tool with built-in testing and deployment. LangChain is a framework for building LLM chains in code. You can use LangChain *inside* Prompt Flow Python nodes.

2. **"Can I use Prompt Flow without the portal?"** — Yes, the `promptflow` SDK supports local development, CLI-based testing, and programmatic deployment. The portal provides the visual editor on top.

3. **"What about streaming responses?"** — Prompt Flow supports streaming for Chat flows when deployed as endpoints. Enable it in the deployment configuration.

4. **"Can multiple people work on the same flow?"** — Flows are stored as YAML + source files and can be version-controlled in Git. Use source control for team collaboration.

### Assessment Connections

- **Quiz:** 10 multiple-choice questions covering all sections → `quiz/assessment.md`
- **Lab:** Hands-on exercise building the Mini Hack flow → `lab/hands-on-lab.md`
- **Presentation:** 14-slide deck for instructor-led delivery → `slides/presentation.html`

---

## 8. Key Takeaways

1. **Prompt Flow is purpose-built** for LLM orchestration — not a general ML pipeline tool
2. **DAG-based design** makes complex AI workflows visual, testable, and maintainable
3. **Node types** (LLM, Python, Prompt, Embedding) cover the full spectrum of AI app needs
4. **Integrated testing** with variants and batch runs enables data-driven prompt optimization
5. **One-click deployment** to managed endpoints with auto-scaling simplifies production operations
6. **Routing patterns** enable sophisticated multi-intent applications within a single flow

---

## 📚 Additional Resources

### Documentation

- [Prompt Flow in Azure AI Foundry — Official Docs](https://learn.microsoft.com/azure/ai-studio/how-to/prompt-flow)
- [Deploy a Flow as an Endpoint](https://learn.microsoft.com/azure/ai-studio/how-to/flow-deploy)
- [Prompt Flow SDK — Open Source](https://microsoft.github.io/promptflow/)
- [Batch Testing and Evaluation](https://learn.microsoft.com/azure/ai-studio/how-to/flow-bulk-test-evaluation)

### Video Resources

- 📺 [Getting Started with Azure AI Foundry Prompt Flow — MS Developer Playlist](https://www.youtube.com/playlist?list=PLC8Ke1cGWBoskdzX_Z4QBA_YP3QtiAIue)

### Related Modules

- ← **Module 20:** SDK & API Integration Patterns
- → **Module 22:** Semantic Kernel Integration

---

*Module 21 of 45 — Zero to Hero: Azure AI Foundry Training · ARC 5: Orchestration & Integration · April 2026*
