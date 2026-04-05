# Hands-On Lab: Building Your First Agent

## Module 17 — Azure AI Foundry: Zero to Hero

| Field            | Details                                         |
| ---------------- | ----------------------------------------------- |
| **Duration**     | 45 minutes                                      |
| **Level**        | Intermediate                                    |
| **Arc**          | 4 — AI Agents                                   |
| **Objective**    | Build a data analyst agent with Code Interpreter |

---

## Prerequisites

- [x] Azure subscription with Azure AI Foundry project
- [x] GPT-4o (or GPT-4.1) model deployed in the project
- [x] Python 3.9+ with `pip` available
- [x] Azure CLI installed and authenticated (`az login`)

### Install Required Packages

```bash
pip install azure-ai-projects azure-identity
```

---

## Lab Overview

In this lab, you will:

1. **Exercise 1:** Create an agent with Code Interpreter in the Playground
2. **Exercise 2:** Build the same agent programmatically with the Python SDK
3. **Exercise 3:** Enhance the agent with File Search for document Q&A
4. **Challenge:** Combine Code Interpreter and Bing Search in a single agent

---

## Exercise 1: Agent Playground (10 min)

### Step 1.1 — Open the Agent Playground

1. Navigate to [Azure AI Foundry](https://ai.azure.com)
2. Open your project
3. Select **Agents** from the left navigation menu
4. Click **+ New Agent**

### Step 1.2 — Configure the Agent

Set the following fields:

| Field            | Value                                      |
| ---------------- | ------------------------------------------ |
| **Name**         | `lab-data-analyst`                         |
| **Model**        | `gpt-4o` (select your deployment)         |
| **Instructions** | *(see below)*                              |

**Instructions:**

```text
You are an expert data analyst. When given data:
1. Inspect the data — show shape, columns, and first 5 rows
2. Compute summary statistics for all numeric columns
3. Create a clear bar chart of the top categories by value
4. Highlight the 3 most important insights

Use clean, labeled matplotlib visualizations.
Always explain your methodology.
```

### Step 1.3 — Enable Code Interpreter

1. In the **Tools** section, toggle **Code Interpreter** to **ON**
2. Click **Save**

### Step 1.4 — Test the Agent

1. In the chat panel (right side), type:

   ```
   Generate sample sales data with columns: Product, Region, Revenue, Units.
   Create 50 rows, then analyze the data and show a chart.
   ```

2. Observe:
   - The agent writes Python code to generate data
   - Summary statistics are computed
   - A bar chart is generated
   - Insights are highlighted

### Step 1.5 — Enable Tool Visibility

1. Click the **⚙️ Settings** icon in the chat panel
2. Enable **"Show tool calls"**
3. Send another message and observe the tool invocation details

> **✅ Checkpoint:** You should see the agent generating and executing Python code, producing both text output and a visualization.

---

## Exercise 2: Python SDK Agent (20 min)

### Step 2.1 — Set Up Environment

Create a new file called `agent_lab.py`:

```python
import os
from azure.ai.projects import AIProjectClient
from azure.ai.projects.models import CodeInterpreterTool, FilePurpose
from azure.identity import DefaultAzureCredential
```

### Step 2.2 — Configure the Client

```python
# Replace with your project endpoint
ENDPOINT = os.environ.get(
    "AZURE_AI_FOUNDRY_ENDPOINT",
    "https://<your-project>.services.ai.azure.com"
)

client = AIProjectClient(
    endpoint=ENDPOINT,
    credential=DefaultAzureCredential()
)
print("✅ Client initialized")
```

### Step 2.3 — Create a Sample Data File

Create a file called `sample_sales.csv`:

```csv
Product,Region,Revenue,Units,Quarter
Widget A,North,15000,120,Q1
Widget A,South,12000,95,Q1
Widget A,East,18000,140,Q1
Widget A,West,9000,70,Q1
Widget B,North,22000,180,Q1
Widget B,South,19000,155,Q1
Widget B,East,25000,200,Q1
Widget B,West,16000,130,Q1
Gadget C,North,8000,60,Q1
Gadget C,South,11000,85,Q1
Gadget C,East,7500,55,Q1
Gadget C,West,13000,100,Q1
Widget A,North,17000,135,Q2
Widget A,South,14000,110,Q2
Widget A,East,20000,160,Q2
Widget A,West,11000,85,Q2
Widget B,North,24000,195,Q2
Widget B,South,21000,170,Q2
Widget B,East,28000,225,Q2
Widget B,West,18000,145,Q2
Gadget C,North,10000,75,Q2
Gadget C,South,13000,100,Q2
Gadget C,East,9500,70,Q2
Gadget C,West,15000,115,Q2
```

### Step 2.4 — Create the Agent

```python
INSTRUCTIONS = """You are an expert data analyst. When given a dataset:

1. **Inspect** — Load the file, show shape, columns, dtypes, and first 5 rows
2. **Summarize** — Compute mean, median, std, min, max for numeric columns
3. **Visualize** — Create a bar chart of products by total revenue
   - Use matplotlib with a clean style
   - Add labeled axes and a title
   - Use distinct colors for each product
4. **Insights** — Identify the top 3 key trends or patterns

Always explain your methodology clearly."""

agent = client.agents.create_agent(
    model="gpt-4o",
    name="lab-sdk-analyst",
    instructions=INSTRUCTIONS,
    tools=[CodeInterpreterTool()]
)
print(f"✅ Agent created: {agent.id}")
```

### Step 2.5 — Upload File and Create Thread

```python
# Upload the data file
data_file = client.agents.upload_file_and_poll(
    file_path="sample_sales.csv",
    purpose=FilePurpose.AGENTS
)
print(f"📁 File uploaded: {data_file.id}")

# Create a thread
thread = client.agents.create_thread()
print(f"🧵 Thread created: {thread.id}")

# Add a message with the file attached
message = client.agents.create_message(
    thread_id=thread.id,
    role="user",
    content="Analyze this sales dataset. Show summary statistics, "
            "create a chart of total revenue by product, "
            "and share the 3 most important insights.",
    attachments=[{
        "file_id": data_file.id,
        "tools": [{"type": "code_interpreter"}]
    }]
)
print(f"💬 Message added: {message.id}")
```

### Step 2.6 — Run the Agent with Streaming

```python
print("\n🔄 Running agent...\n" + "=" * 50)

with client.agents.create_stream(
    thread_id=thread.id,
    agent_id=agent.id
) as stream:
    for event in stream:
        if hasattr(event, "data") and hasattr(event.data, "delta"):
            for content in event.data.delta.content:
                if hasattr(content, "text"):
                    print(content.text.value, end="")

print("\n" + "=" * 50)
```

### Step 2.7 — Download Generated Charts

```python
# Get all messages from the thread
messages = client.agents.list_messages(thread_id=thread.id)

image_count = 0
for msg in messages.data:
    for content in msg.content:
        if hasattr(content, "image_file"):
            image_count += 1
            file_id = content.image_file.file_id
            file_content = client.agents.get_file_content(file_id)
            filename = f"chart_{image_count}.png"
            with open(filename, "wb") as f:
                for chunk in file_content:
                    f.write(chunk)
            print(f"📊 Saved: {filename}")

if image_count == 0:
    print("ℹ️ No charts were generated in this run.")
```

### Step 2.8 — Clean Up

```python
client.agents.delete_agent(agent.id)
print("🧹 Agent deleted. Lab complete!")
```

> **✅ Checkpoint:** Run `python agent_lab.py` and verify that:
> - The agent analyzes the CSV data
> - Summary statistics are printed
> - A chart file is saved locally
> - The agent is cleaned up

---

## Exercise 3: Adding File Search (10 min)

### Step 3.1 — Create a Knowledge Document

Create a file called `product_info.md`:

```markdown
# Product Catalog

## Widget A
- Category: Electronics
- MSRP: $125
- Launch Date: January 2025
- Description: Compact wireless device for home automation

## Widget B
- Category: Electronics
- MSRP: $200
- Launch Date: March 2025
- Description: Premium smart display with voice control

## Gadget C
- Category: Accessories
- MSRP: $75
- Launch Date: June 2025
- Description: Portable charging hub with fast-charge support
```

### Step 3.2 — Create Agent with File Search

```python
from azure.ai.projects.models import FileSearchTool

# Upload document
doc_file = client.agents.upload_file_and_poll(
    file_path="product_info.md",
    purpose=FilePurpose.AGENTS
)

# Create vector store
vector_store = client.agents.create_vector_store_and_poll(
    name="product-knowledge",
    file_ids=[doc_file.id]
)
print(f"📚 Vector store created: {vector_store.id}")

# Create agent with both tools
agent = client.agents.create_agent(
    model="gpt-4o",
    name="lab-multi-tool-analyst",
    instructions="""You are a product analyst with access to:
- Code Interpreter for data analysis
- Product documentation via File Search

When analyzing sales data, cross-reference with product info
to provide richer insights (e.g., revenue per MSRP ratio).""",
    tools=[CodeInterpreterTool(), FileSearchTool()],
    tool_resources={
        "file_search": {"vector_store_ids": [vector_store.id]}
    }
)
```

### Step 3.3 — Test Multi-Tool Agent

```python
thread = client.agents.create_thread()
client.agents.create_message(
    thread_id=thread.id,
    role="user",
    content="Using the sales data and product documentation, "
            "which product has the best revenue-to-MSRP ratio? "
            "Create a comparison chart.",
    attachments=[{
        "file_id": data_file.id,
        "tools": [{"type": "code_interpreter"}]
    }]
)

# Run and print results
run = client.agents.create_and_process_run(
    thread_id=thread.id,
    agent_id=agent.id
)

messages = client.agents.list_messages(thread_id=thread.id)
for msg in messages.data:
    if msg.role == "assistant":
        for content in msg.content:
            if hasattr(content, "text"):
                print(content.text.value)
        break
```

> **✅ Checkpoint:** The agent should cross-reference sales revenue with MSRP data from the product docs to compute and visualize revenue-to-MSRP ratios.

---

## Challenge: Bing-Enhanced Agent (Optional, 5 min)

If you have a Bing Grounding resource configured in your project:

```python
from azure.ai.projects.models import BingGroundingTool

bing_conn = client.connections.get("bing-grounding")

agent = client.agents.create_agent(
    model="gpt-4o",
    name="lab-research-analyst",
    instructions="""You are a market research analyst.
- Use Code Interpreter for data analysis
- Use Bing Search for current market context
- Combine internal data with web research for comprehensive insights""",
    tools=[
        CodeInterpreterTool(),
        BingGroundingTool(connection_id=bing_conn.id)
    ]
)
```

Test with: *"Analyze our sales data and compare our Widget B pricing with current market competitors."*

---

## Clean-Up Checklist

After completing the lab, ensure you've cleaned up all resources:

- [ ] Deleted all agents created during the lab
- [ ] Removed uploaded files (optional — they expire automatically)
- [ ] Deleted vector stores if created

```python
# Clean up all resources
client.agents.delete_agent(agent.id)
# client.agents.delete_vector_store(vector_store.id)  # if created
print("🧹 All resources cleaned up.")
```

---

## Summary

In this lab, you:

1. ✅ Created and tested an agent in the AI Foundry Playground
2. ✅ Built a Code Interpreter agent with the Python SDK
3. ✅ Enhanced the agent with File Search for document grounding
4. ✅ (Optional) Added Bing Search for real-time web context

**Next:** [Module 18 — Custom Function Tools](../../module-18/lab/hands-on-lab.md)

---

*Azure AI Foundry: Zero to Hero — Module 17 Lab — April 2026*
