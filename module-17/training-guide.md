# Module 17: Building Your First Agent

## Azure AI Foundry: Zero to Hero | Arc 4 — AI Agents

| Field          | Details                                    |
| -------------- | ------------------------------------------ |
| **Module**     | 17 of 45                                   |
| **Arc**        | 4 — AI Agents                              |
| **Duration**   | 90 minutes                                 |
| **Level**      | Intermediate                               |
| **Date**       | April 2026                                 |

---

## Learning Objectives

By the end of this module, learners will be able to:

1. Explain the core agent primitives — Agent, Thread, Message, and Run
2. Configure effective agent instructions and persona definitions
3. Use built-in tools: Code Interpreter, File Search, and Bing Search
4. Test and iterate on agents using the Azure AI Foundry Playground
5. Create agents programmatically with the Python and .NET SDKs
6. Build a data analyst agent with Code Interpreter (Mini Hack)

---

## Prerequisites

- Completed Module 16: Introduction to AI Agents
- Active Azure subscription with Azure AI Foundry project
- Deployed GPT-4o or GPT-4.1 model in the project
- Python 3.9+ or .NET 8+ installed locally
- Azure CLI authenticated (`az login`)

---

## Module Outline

### Section 1: Agent Fundamentals (15 min)

#### What Is an AI Agent?

An AI Agent in Azure AI Foundry is an entity configured with:

- **Instructions** — System prompt defining behavior and persona
- **Model** — The LLM powering the agent (e.g., GPT-4o)
- **Tools** — Capabilities the agent can invoke (Code Interpreter, File Search, Bing Search, custom functions)

#### Core Primitives

| Primitive   | Description                                                    |
| ----------- | -------------------------------------------------------------- |
| **Agent**   | Entity with instructions, model, and tools                     |
| **Thread**  | Conversation session storing message history                   |
| **Message** | A unit of communication (text, image, or file) within a thread |
| **Run**     | An execution of the agent on a thread                          |

#### Execution Flow

```
User Message → Thread → Run → Agent reads Thread
  → Agent calls Tools → Tool outputs appended → Model generates response
  → Assistant Message added to Thread
```

### Section 2: Configuring Instructions & Persona (15 min)

#### Why Instructions Matter

Instructions are the single most important factor in agent quality. They define:

- **Role** — Who the agent is (e.g., "financial data analyst")
- **Capabilities** — What the agent can do
- **Behavior** — How the agent should respond
- **Constraints** — What the agent should NOT do

#### Instruction Template

```markdown
# Agent: [Role Name]

## Role
[1-2 sentence persona description]

## Capabilities
- [Capability 1]
- [Capability 2]
- [Capability 3]

## Behavior Guidelines
- [Guideline 1 — tone, style, methodology]
- [Guideline 2]

## Constraints
- [What the agent should NOT do]
- [Boundaries and limitations]
```

#### Best Practices for Instructions

1. **Be specific** — Vague instructions produce vague outputs
2. **Use Markdown** — Structured formatting helps the model parse sections
3. **Define error handling** — Tell the agent what to do when data is missing
4. **Set output format** — Specify how results should be presented
5. **Include examples** — Few-shot examples improve consistency

### Section 3: Built-in Tools (20 min)

#### Code Interpreter

**What it does:** Executes Python code in a sandboxed environment.

**Use cases:**
- Data analysis on CSV, Excel, JSON files
- Mathematical computations
- Chart and visualization generation
- File format conversion

**Key features:**
- Sandboxed execution (no network access from within code)
- File upload and download support
- Matplotlib, pandas, numpy pre-installed
- Persistent file system within a session

```python
from azure.ai.projects.models import CodeInterpreterTool

agent = client.agents.create_agent(
    model="gpt-4o",
    name="data-analyst",
    instructions="Analyze data and create visualizations.",
    tools=[CodeInterpreterTool()]
)
```

#### File Search (RAG)

**What it does:** Searches uploaded documents using vector embeddings.

**Use cases:**
- Document Q&A over PDFs, DOCX, Markdown
- Knowledge base retrieval
- Policy and compliance lookups

**Key features:**
- Automatic chunking and embedding
- Vector store persistence across sessions
- Supports PDF, DOCX, TXT, MD, HTML, and more

```python
from azure.ai.projects.models import FileSearchTool

# Create vector store with documents
vector_store = client.agents.create_vector_store_and_poll(
    name="knowledge-base",
    file_ids=[file1.id, file2.id]
)

agent = client.agents.create_agent(
    model="gpt-4o",
    tools=[FileSearchTool()],
    tool_resources={"file_search": {"vector_store_ids": [vector_store.id]}}
)
```

#### Bing Search (Grounding)

**What it does:** Searches the web for real-time information.

**Use cases:**
- Current events and news
- Real-time data (stock prices, weather)
- Fact-checking and verification

**Prerequisites:**
- Grounding with Bing Search resource in Azure portal
- Connection configured in AI Foundry project

```python
from azure.ai.projects.models import BingGroundingTool

bing_connection = client.connections.get("bing-grounding")
agent = client.agents.create_agent(
    model="gpt-4o",
    tools=[BingGroundingTool(connection_id=bing_connection.id)]
)
```

#### Tool Comparison Matrix

| Feature           | Code Interpreter        | File Search               | Bing Search           |
| ----------------- | ----------------------- | ------------------------- | --------------------- |
| Primary Use       | Computation & analysis  | Document Q&A              | Real-time web info    |
| Input             | CSV, Excel, JSON, code  | PDF, DOCX, TXT, MD       | Natural language      |
| Output            | Text, charts, files     | Grounded text             | Text with citations   |
| State             | Per-session sandbox     | Persistent vector store   | Stateless             |
| Network Access    | No (sandboxed)          | N/A                       | Yes                   |
| Extra Cost        | Per session-minute      | Per GB + queries          | Per Bing API call     |

### Section 4: Testing in the Playground (10 min)

#### Walkthrough

1. Navigate to **AI Foundry portal** → Your project → **Agents**
2. Click **+ New Agent**
3. Configure:
   - Name: `test-analyst`
   - Model: `gpt-4o`
   - Instructions: *(paste from template above)*
4. Enable **Code Interpreter** toggle
5. Send test messages in the chat panel
6. Toggle **"Show tool calls"** to see tool invocations
7. Iterate on instructions based on results

#### What to Look For

- Does the agent use the correct tools?
- Is the output format as expected?
- Does the agent handle edge cases (missing data, ambiguous questions)?
- Are responses grounded and factual?

### Section 5: Agent SDKs (20 min)

#### Python SDK

**Package:** `azure-ai-projects`

```bash
pip install azure-ai-projects azure-identity
```

**Key classes:**
- `AIProjectClient` — Main client
- `CodeInterpreterTool` — Code Interpreter tool definition
- `FileSearchTool` — File Search tool definition
- `BingGroundingTool` — Bing Search tool definition

**Agent lifecycle:**
1. Create agent → 2. Create thread → 3. Add message → 4. Create run/stream → 5. Read response → 6. Delete agent

#### .NET SDK

**Package:** `Azure.AI.Projects`

```bash
dotnet add package Azure.AI.Projects --prerelease
dotnet add package Azure.Identity
```

**Key classes:**
- `AgentsClient` — Main client
- `CodeInterpreterToolDefinition` — Code Interpreter tool
- `FileSearchToolDefinition` — File Search tool

**Agent lifecycle** (same steps as Python, with C# idioms).

### Section 6: Mini Hack — Data Analyst Agent (10 min)

#### Objective

Build a data analyst agent that:
1. Loads a CSV sales dataset
2. Computes summary statistics
3. Creates a bar chart of top products by revenue
4. Identifies the top 3 insights

#### Success Criteria

- [ ] Agent created with Code Interpreter enabled
- [ ] CSV file uploaded and attached to thread
- [ ] Summary statistics displayed correctly
- [ ] Bar chart generated and saved
- [ ] 3 key insights identified
- [ ] Agent cleaned up after completion

---

## Key Takeaways

1. **Instructions are everything** — Invest time in crafting clear, structured agent instructions
2. **Right tool for the job** — Match tools to use cases (Code Interpreter for data, File Search for docs, Bing for web)
3. **Playground first** — Always prototype in the playground before writing SDK code
4. **Clean up resources** — Delete agents and files when done to avoid costs
5. **Security by default** — Use Managed Identity, not API keys, in production

---

## Additional Resources

- [Azure AI Agent Service Documentation](https://learn.microsoft.com/azure/ai-services/agents/)
- [Python SDK Reference — azure-ai-projects](https://learn.microsoft.com/python/api/azure-ai-projects/)
- [.NET SDK Reference — Azure.AI.Projects](https://learn.microsoft.com/dotnet/api/azure.ai.projects/)
- [Agent Quickstart Tutorial](https://learn.microsoft.com/azure/ai-services/agents/quickstart)
- [Video: Build An AI Agent From Scratch](https://www.youtube.com/watch?v=wL8H0RySf6I)

---

## Assessment

Complete the [Module 17 Quiz](quiz/assessment.md) (10 questions, 80% passing score) and the [Hands-on Lab](lab/hands-on-lab.md).

---

*Azure AI Foundry: Zero to Hero — Module 17 of 45 — April 2026*
