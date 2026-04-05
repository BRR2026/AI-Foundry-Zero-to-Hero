# Hands-On Lab: Create Your First AI Agent

## Module 16 — Intro to AI Agents & Agent Framework

**Duration:** 45–60 minutes  
**Level:** 🟡 Intermediate  
**Platform:** Azure AI Foundry (Microsoft Foundry)

---

## Lab Overview

In this lab, you'll create your first AI agent using the Azure AI Agent Service. You'll start with a **prompt agent** (no code required) in the Azure AI Foundry portal, configure it with built-in tools, test its autonomous behavior, and then extend it with the Python SDK.

### What You'll Build

A **Travel Planning Agent** that can:
- Suggest destinations based on user preferences
- Create day-by-day itineraries
- Calculate budget estimates using Code Interpreter
- Look up current travel information via Bing Grounding
- Maintain conversation context across multiple exchanges

### Learning Objectives

1. Create a prompt agent in the Azure AI Foundry portal
2. Write effective agent instructions (system prompt)
3. Enable and configure built-in tools (Code Interpreter, Bing Grounding)
4. Test agent behavior and observe tool usage patterns
5. Interact with the agent programmatically via the Python SDK

---

## Prerequisites

- **Azure subscription** with an active AI Foundry project
- **Deployed model**: GPT-4o (or GPT-4o-mini) in your project
- **Bing Grounding connection** configured in your project (optional but recommended)
- **Python 3.10+** with `azure-ai-projects` and `azure-identity` packages installed (for Exercise 3)
- Completed Modules 1–15 (Beginner tier)

> **Note:** If you don't have a Bing Grounding connection, you can skip the Bing-related steps. The agent will still work with Code Interpreter alone.

---

## Exercise 1: Create a Prompt Agent in the Portal (15 minutes)

### Step 1: Navigate to the Agents Section

1. Open your browser and go to [https://ai.azure.com](https://ai.azure.com)
2. Sign in with your Azure credentials
3. Select your **AI Foundry project** from the project list
4. In the left navigation menu, click **"Agents"**
5. Click the **"+ New agent"** button

### Step 2: Configure Agent Basics

1. **Agent name:** `Travel Planner`
2. **Model:** Select your deployed `gpt-4o` model from the dropdown
3. **Description:** `A helpful travel planning agent that creates itineraries, estimates budgets, and provides travel tips.`

### Step 3: Write Agent Instructions

In the **Instructions** text area, enter the following:

```text
## Role
You are a Travel Planning Agent. You help users plan trips by suggesting 
destinations, creating detailed itineraries, estimating budgets, and 
providing practical travel tips.

## Goals
- Help users discover ideal travel destinations based on their preferences
- Create detailed day-by-day itineraries with activities and timing
- Calculate and break down trip budgets including flights, hotels, food, and activities
- Provide practical tips about visas, weather, local customs, and safety

## Behavior
- Always ask clarifying questions before making recommendations:
  - Travel dates or preferred month/season
  - Budget range (total or per-person)
  - Travel style (adventure, relaxation, culture, food, etc.)
  - Number of travelers
- Format itineraries with clear day-by-day breakdowns
- Include at least one off-the-beaten-path recommendation
- Use Code Interpreter for budget calculations and comparisons
- Use Bing to look up current travel advisories, weather, and pricing

## Constraints
- Never make up prices — use Code Interpreter for calculations or Bing for current data
- Always mention if information might be outdated and suggest verifying
- If a destination has travel advisories, prominently mention them
- Be transparent about assumptions in budget estimates
```

### Step 4: Enable Tools

1. Toggle **ON** the **Code Interpreter** tool
2. Toggle **ON** the **Bing Grounding** tool (if available)
3. Leave **File Search** toggled OFF for now

### Step 5: Set Parameters

1. **Temperature:** `0.5` (balanced between creative and factual)
2. **Top P:** `0.9`
3. Leave other parameters at defaults

### Step 6: Save the Agent

Click **"Create"** to save your agent configuration.

> ✅ **Checkpoint:** Your agent should now appear in the Agents list with status "Ready."

---

## Exercise 2: Test Agent Behavior (15 minutes)

### Step 1: Open the Agent Playground

1. Click on your **Travel Planner** agent in the list
2. Click **"Try in Playground"** or open the **Test** pane

### Step 2: Test Conversation Flow

Send the following messages one at a time and observe the agent's behavior:

**Message 1:**
```
I want to plan a 5-day trip to Japan for two people in November with a $5,000 budget.
```

**What to observe:**
- Does the agent ask follow-up questions about travel style and preferences?
- Does it use Bing to look up current information about Japan travel?
- Does the response include practical details?

**Message 2:**
```
We love food and cultural experiences. We'd like to split time between Tokyo and Kyoto.
```

**What to observe:**
- Does the agent use the context from Message 1?
- Does it create a structured day-by-day itinerary?
- Does it include off-the-beaten-path recommendations?

**Message 3:**
```
Can you break down the budget? Show me estimated costs for flights, hotels, food, transport, and activities.
```

**What to observe:**
- Does the agent use **Code Interpreter** to calculate the budget?
- Look for the "Code Interpreter" tool usage indicator in the response
- Does it create a clear breakdown with totals?

### Step 3: Test Tool Awareness

Send a message that specifically requires tool use:

```
What's the current exchange rate for USD to Japanese Yen, and what's the weather typically like in Kyoto in November?
```

**What to observe:**
- Does the agent use **Bing Grounding** to look up current exchange rates?
- Does it acknowledge that exchange rates are real-time data?

### Step 4: Test Constraints

```
How much does a first-class flight from New York to Tokyo cost exactly?
```

**What to observe:**
- The agent should NOT fabricate a specific price
- It should either search Bing for approximate ranges or clearly state it's providing estimates
- This validates your constraint instructions are working

> ✅ **Checkpoint:** You should see the agent using tools autonomously, maintaining context across messages, and following your constraints.

---

## Exercise 3: Interact via Python SDK (15 minutes)

### Step 1: Set Up Your Environment

```bash
# Install required packages
pip install azure-ai-projects azure-identity

# Set your project endpoint (replace with your actual endpoint)
# You can find this in the AI Foundry portal under Project Settings > Overview
```

### Step 2: Create and Run Agent Programmatically

Create a file called `agent_demo.py`:

```python
"""Module 16 Lab — Interact with your Travel Planner agent via the Python SDK."""

from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

# Initialize the client
# Replace with your project's endpoint from the Foundry portal
PROJECT_ENDPOINT = "https://<your-project>.services.ai.azure.com"

client = AIProjectClient(
    credential=DefaultAzureCredential(),
    endpoint=PROJECT_ENDPOINT
)

# List existing agents to find your Travel Planner
print("=== Your Agents ===")
agents = client.agents.list_agents()
for agent in agents.data:
    print(f"  {agent.name} (ID: {agent.id}, Model: {agent.model})")

# Use your Travel Planner agent ID (copy from the portal or the list above)
AGENT_ID = "<your-travel-planner-agent-id>"

# Create a new conversation thread
thread = client.agents.create_thread()
print(f"\nThread created: {thread.id}")

# Send a message
client.agents.create_message(
    thread_id=thread.id,
    role="user",
    content="Plan a 3-day weekend trip to Barcelona for a solo traveler on a $1,500 budget. I love architecture and street food."
)

# Run the agent
print("\nRunning agent... (this may take 15-30 seconds)")
run = client.agents.create_and_process_run(
    thread_id=thread.id,
    agent_id=AGENT_ID
)

print(f"Run status: {run.status}")

# Get the response
messages = client.agents.list_messages(thread_id=thread.id)

print("\n=== Agent Response ===")
for msg in reversed(list(messages.data)):
    role = msg.role.upper()
    for content_block in msg.content:
        if hasattr(content_block, 'text'):
            print(f"\n[{role}]")
            print(content_block.text.value)
```

### Step 3: Run the Script

```bash
# Make sure you're logged in to Azure
az login

# Run the script
python agent_demo.py
```

### Step 4: Send a Follow-Up Message

Add this to the bottom of your script to test multi-turn conversation:

```python
# Send a follow-up in the same thread
client.agents.create_message(
    thread_id=thread.id,
    role="user",
    content="Can you calculate the daily budget breakdown? Use Code Interpreter to create a summary table."
)

run2 = client.agents.create_and_process_run(
    thread_id=thread.id,
    agent_id=AGENT_ID
)

messages2 = client.agents.list_messages(thread_id=thread.id)
latest = list(messages2.data)[0]  # Most recent message
for content_block in latest.content:
    if hasattr(content_block, 'text'):
        print("\n=== Follow-Up Response ===")
        print(content_block.text.value)
```

> ✅ **Checkpoint:** Your Python script should display the agent's complete response with itinerary and budget details.

---

## Exercise 4: Experiment and Iterate (10 minutes)

### Experiment 1: Create a Different Agent

Create a second agent with a completely different persona:

- **Name:** `Code Review Assistant`
- **Instructions:** *"You are a code review agent. When users share code, analyze it for bugs, security issues, performance problems, and style. Always provide specific line references and suggest fixes. Use Code Interpreter to test edge cases when possible."*
- **Tools:** Code Interpreter only

Test it by sharing a code snippet and comparing how it behaves differently from the Travel Planner.

### Experiment 2: Adjust Temperature

1. Go back to your Travel Planner agent
2. Change the **temperature** from `0.5` to `0.1`
3. Ask the same Japan trip question — notice more focused, deterministic responses
4. Change the **temperature** to `0.9`
5. Ask again — notice more creative, varied suggestions

### Experiment 3: Strengthen Constraints

Add this line to your instructions:
```
- CRITICAL: You must NEVER provide medical advice, even if travel-related (e.g., vaccines). Always direct users to consult a healthcare professional.
```

Then test: *"What vaccines do I need for traveling to Thailand?"*

---

## Cleanup

If you're done experimenting and want to clean up:

1. Go to **Agents** in the portal
2. Select agents you created during the lab
3. Click **Delete** to remove them

> **Note:** Threads and conversation history will be retained for 30 days after the agent is deleted.

---

## Summary

In this lab, you:

| Task | Status |
|------|--------|
| Created a prompt agent in the portal | ✅ |
| Wrote structured agent instructions | ✅ |
| Enabled Code Interpreter and Bing tools | ✅ |
| Tested autonomous tool usage behavior | ✅ |
| Interacted with the agent via Python SDK | ✅ |
| Experimented with temperature and constraints | ✅ |

### Key Observations

1. **Agents choose tools autonomously** — you didn't tell the agent when to use Code Interpreter or Bing. It decided based on the user's request and your instructions.
2. **Instructions are powerful** — small changes to the system prompt significantly change agent behavior.
3. **Threads maintain state** — the agent remembered context from earlier in the conversation.
4. **Temperature affects style** — lower temperature = more focused; higher = more creative.

---

## Next Lab

**Module 17 Lab:** Deep dive into agent tools — configure File Search with uploaded documents, build custom functions, and implement tool-calling patterns.
