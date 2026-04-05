# Module 18 — Hands-on Lab: Build an Agent That Remembers User Preferences

## Azure AI Foundry: Zero to Hero Training Course

**ARC 4: AI AGENTS** | Mini Hack | April 2026

---

## Lab Overview

In this hands-on lab, you will build an AI agent that maintains memory of user preferences across multiple conversation sessions. Using Azure Cosmos DB as a persistent memory store and the Azure AI Foundry Agent Service, you will create a complete memory-augmented agent that can recall a user's name, favorite programming language, timezone, and other preferences — even after the original conversation thread has ended. By the end of this lab, you will understand how to design and implement long-term memory for AI agents.

**Estimated Duration:** 60 minutes

## Learning Objectives

- Provision Azure Cosmos DB as a persistent memory store for AI agents
- Build a reusable Python memory layer with CRUD operations
- Create an Azure AI Foundry agent that reads and writes user preferences
- Implement cross-session memory by injecting stored context into new threads
- Test and validate multi-turn, multi-session memory recall

## Prerequisites

- Active Azure subscription
- Azure AI Foundry project with a deployed `gpt-4o` model (created in earlier modules)
- Python 3.10 or later installed locally
- Azure CLI authenticated (`az login` completed)
- Basic familiarity with Azure Cosmos DB concepts
- Web browser with access to https://ai.azure.com

---

## Part 1: Set Up Azure Cosmos DB for Memory Storage (15 minutes)

### Step 1.1 — Create a Cosmos DB Account

If you already have an Azure Cosmos DB for NoSQL account, skip to **Step 1.2**.

1. Navigate to the [Azure Portal](https://portal.azure.com)
2. Click **+ Create a resource** → search for **Azure Cosmos DB**
3. Select **Azure Cosmos DB** and click **Create**
4. Choose **Azure Cosmos DB for NoSQL** and click **Create**
5. Configure the account:
   - **Subscription:** Select your subscription
   - **Resource Group:** Use your existing AI Foundry resource group
   - **Account Name:** `agent-memory-<your-initials>` (must be globally unique)
   - **Location:** Same region as your AI Foundry project
   - **Capacity mode:** **Serverless** (recommended for labs)
6. Click **Review + Create** → **Create**
7. Wait for deployment to complete (typically 3–5 minutes)

> **💡 Tip:** Serverless mode is ideal for development and labs — you only pay for the request units and storage you actually consume, with no minimum charge.

### Step 1.2 — Create the Database and Container

1. Open your Cosmos DB account in the Azure Portal
2. In the left navigation, click **Data Explorer**
3. Click **New Container**
4. Configure the container:
   - **Database id:** Select **Create new** → enter `agent-memory`
   - **Container id:** `user-preferences`
   - **Partition key:** `/userId`
5. Click **OK** and wait for creation to complete

You should now see the following structure in Data Explorer:

```
📁 agent-memory (database)
└── 📁 user-preferences (container)
    └── Items
```

### Step 1.3 — Install Required Python Packages

Open your terminal and create a project directory:

```bash
mkdir module-18-memory-agent
cd module-18-memory-agent
```

Install the required packages:

```bash
pip install azure-ai-projects azure-cosmos azure-identity
```

> **⚠️ Warning:** Ensure you install `azure-ai-projects` (not `azure-ai-project` without the "s"). The package name must be exact.

### Step 1.4 — Set Up Environment Variables

Create a `.env` file in your project directory:

```env
# Azure AI Foundry
PROJECT_CONNECTION_STRING=<your-ai-foundry-project-connection-string>

# Azure Cosmos DB
COSMOS_ENDPOINT=https://<your-cosmos-account>.documents.azure.com:443/
COSMOS_DATABASE=agent-memory
COSMOS_CONTAINER=user-preferences
```

**To find your Project Connection String:**
1. Open [https://ai.azure.com](https://ai.azure.com)
2. Navigate to your project
3. Go to **Project settings**
4. Copy the **Project connection string**

**To find your Cosmos DB Endpoint:**
1. Open your Cosmos DB account in the Azure Portal
2. Go to **Settings** → **Keys**
3. Copy the **URI** value

> **💡 Tip:** Using `DefaultAzureCredential` means you do not need to copy Cosmos DB keys into your environment file. Your Azure CLI login is sufficient for authentication.

---

## Part 2: Build the Memory Layer (15 minutes)

### Step 2.1 — Create the Memory Store Class

Create a new file called `memory_store.py`:

```python
import os
from datetime import datetime, timezone
from azure.cosmos import CosmosClient, PartitionKey, exceptions
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv

load_dotenv()


class UserMemoryStore:
    """Persistent memory store for user preferences using Azure Cosmos DB."""

    def __init__(self):
        credential = DefaultAzureCredential()
        self.client = CosmosClient(
            url=os.environ["COSMOS_ENDPOINT"],
            credential=credential,
        )
        database = self.client.get_database_client(os.environ["COSMOS_DATABASE"])
        self.container = database.get_container_client(os.environ["COSMOS_CONTAINER"])

    def save_preference(self, user_id: str, key: str, value: str) -> None:
        """Save a single preference for a user. Creates the user document
        if it does not exist, or merges into the existing document."""
        self._upsert_preferences(user_id, {key: value})

    def get_preferences(self, user_id: str) -> dict:
        """Retrieve all stored preferences for a user.
        Returns an empty dict if no preferences exist."""
        try:
            item = self.container.read_item(
                item=user_id,
                partition_key=user_id,
            )
            # Return only the preferences sub-document
            return item.get("preferences", {})
        except exceptions.CosmosResourceNotFoundError:
            return {}

    def update_preference(self, user_id: str, key: str, value: str) -> None:
        """Update a single preference for a user.
        Uses upsert so it works whether the document exists or not."""
        self._upsert_preferences(user_id, {key: value})

    def _upsert_preferences(self, user_id: str, new_prefs: dict) -> None:
        """Internal helper — merges new preferences into the user document
        using the Cosmos DB upsert pattern for idempotency."""
        try:
            item = self.container.read_item(
                item=user_id,
                partition_key=user_id,
            )
            item["preferences"].update(new_prefs)
            item["updatedAt"] = datetime.now(timezone.utc).isoformat()
        except exceptions.CosmosResourceNotFoundError:
            item = {
                "id": user_id,
                "userId": user_id,
                "preferences": new_prefs,
                "createdAt": datetime.now(timezone.utc).isoformat(),
                "updatedAt": datetime.now(timezone.utc).isoformat(),
            }

        self.container.upsert_item(item)
```

### Step 2.2 — Validate the Memory Store

Create a quick test script called `test_memory.py` to verify Cosmos DB connectivity:

```python
from memory_store import UserMemoryStore


def main():
    store = UserMemoryStore()
    test_user = "test-user-001"

    # Save preferences
    store.save_preference(test_user, "name", "Alice")
    store.save_preference(test_user, "language", "Python")
    store.save_preference(test_user, "timezone", "US/Eastern")
    print("✅ Preferences saved successfully.")

    # Read preferences
    prefs = store.get_preferences(test_user)
    print(f"📋 Stored preferences: {prefs}")

    # Update a preference
    store.update_preference(test_user, "language", "Rust")
    updated = store.get_preferences(test_user)
    print(f"🔄 Updated preferences: {updated}")

    assert updated["name"] == "Alice"
    assert updated["language"] == "Rust"
    assert updated["timezone"] == "US/Eastern"
    print("✅ All memory store tests passed!")


if __name__ == "__main__":
    main()
```

Run the test:

```bash
python test_memory.py
```

**Expected output:**

```
✅ Preferences saved successfully.
📋 Stored preferences: {'name': 'Alice', 'language': 'Python', 'timezone': 'US/Eastern'}
🔄 Updated preferences: {'name': 'Alice', 'language': 'Rust', 'timezone': 'US/Eastern'}
✅ All memory store tests passed!
```

> **⚠️ Warning:** If you receive a `401 Unauthorized` error, ensure your Azure CLI session is active (`az login`) and that your identity has the **Cosmos DB Built-in Data Contributor** role on the Cosmos DB account.

---

## Part 3: Create the Agent with Memory (20 minutes)

### Step 3.1 — Create the Memory-Augmented Agent

Create a new file called `memory_agent.py`:

```python
import os
import json
import time
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv
from memory_store import UserMemoryStore

load_dotenv()

AGENT_INSTRUCTIONS = """You are a helpful personal assistant with persistent memory.

MEMORY CAPABILITIES:
- You can remember user preferences across conversations.
- At the start of each conversation, you will be provided with the user's
  stored preferences (if any).
- When a user tells you something about themselves (name, favorite
  programming language, timezone, preferred communication style, etc.),
  acknowledge it and remember it.

PREFERENCE EXTRACTION:
- When the user shares a preference, respond naturally AND include a JSON
  block at the very end of your response in this exact format:
  [PREFERENCES_UPDATE]{"key": "value", "key2": "value2"}[/PREFERENCES_UPDATE]
- Only include the JSON block when the user is explicitly sharing or
  updating a preference. Do not include it for general questions.

BEHAVIOR:
- Be friendly and conversational.
- Reference stored preferences naturally (e.g., "Since you prefer Python…").
- If no preferences are stored, greet the user and offer to learn about them.
"""


def create_agent(client: AIProjectClient) -> str:
    """Create an agent and return its ID."""
    agent = client.agents.create_agent(
        model="gpt-4o",
        name="memory-agent",
        instructions=AGENT_INSTRUCTIONS,
    )
    print(f"🤖 Agent created: {agent.id}")
    return agent.id


def build_context_message(preferences: dict) -> str:
    """Build a context message that injects stored preferences into
    the conversation."""
    if not preferences:
        return (
            "[SYSTEM CONTEXT] This is a new user with no stored preferences. "
            "Greet them and offer to learn about their preferences."
        )

    pref_lines = "\n".join(f"  - {k}: {v}" for k, v in preferences.items())
    return (
        f"[SYSTEM CONTEXT] Here are the stored preferences for this user:\n"
        f"{pref_lines}\n"
        f"Use these preferences to personalize your responses. "
        f"Do not list them back unless the user asks."
    )


def extract_preferences(response_text: str) -> dict:
    """Extract preference updates from the agent's response."""
    start_tag = "[PREFERENCES_UPDATE]"
    end_tag = "[/PREFERENCES_UPDATE]"

    start = response_text.find(start_tag)
    end = response_text.find(end_tag)

    if start == -1 or end == -1:
        return {}

    json_str = response_text[start + len(start_tag):end].strip()
    try:
        return json.loads(json_str)
    except json.JSONDecodeError:
        print("⚠️  Could not parse preference JSON from response.")
        return {}


def clean_response(response_text: str) -> str:
    """Remove the preference JSON block from the displayed response."""
    start_tag = "[PREFERENCES_UPDATE]"
    end_tag = "[/PREFERENCES_UPDATE]"

    start = response_text.find(start_tag)
    end = response_text.find(end_tag)

    if start == -1 or end == -1:
        return response_text.strip()

    return (response_text[:start] + response_text[end + len(end_tag):]).strip()


def run_conversation_turn(
    client: AIProjectClient,
    agent_id: str,
    thread_id: str,
    user_message: str,
    memory: UserMemoryStore,
    user_id: str,
) -> str:
    """Process a single conversation turn with memory integration."""

    # 1. If this is the first message in the thread, inject preferences
    existing = client.agents.list_messages(thread_id=thread_id)
    if not existing.data:
        preferences = memory.get_preferences(user_id)
        context_msg = build_context_message(preferences)
        client.agents.create_message(
            thread_id=thread_id,
            role="user",
            content=context_msg,
        )

    # 2. Send the user message
    client.agents.create_message(
        thread_id=thread_id,
        role="user",
        content=user_message,
    )

    # 3. Create a run and poll for completion
    run = client.agents.create_run(
        thread_id=thread_id,
        agent_id=agent_id,
    )

    while run.status in ("queued", "in_progress"):
        time.sleep(1)
        run = client.agents.get_run(
            thread_id=thread_id,
            run_id=run.id,
        )

    if run.status != "completed":
        raise RuntimeError(f"Run failed with status: {run.status}")

    # 4. Retrieve the assistant's response
    messages = client.agents.list_messages(thread_id=thread_id)
    assistant_msg = messages.data[0]  # Most recent message
    response_text = assistant_msg.content[0].text.value

    # 5. Extract and save any new preferences
    new_prefs = extract_preferences(response_text)
    if new_prefs:
        for key, value in new_prefs.items():
            memory.update_preference(user_id, key, value)
        print(f"💾 Saved preferences: {new_prefs}")

    # 6. Return the cleaned response
    return clean_response(response_text)
```

### Step 3.2 — Create the Interactive Chat Loop

Create the main entry point file called `main.py`:

```python
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv
import os

from memory_store import UserMemoryStore
from memory_agent import create_agent, run_conversation_turn

load_dotenv()


def run_session(
    client: AIProjectClient,
    agent_id: str,
    memory: UserMemoryStore,
    user_id: str,
    session_label: str,
) -> None:
    """Run a single conversation session (one thread)."""
    print(f"\n{'='*60}")
    print(f"  {session_label}")
    print(f"  User: {user_id}")
    print(f"{'='*60}\n")

    # Each session gets a new thread
    thread = client.agents.create_thread()
    print(f"📎 Thread created: {thread.id}\n")

    while True:
        user_input = input("You: ").strip()
        if user_input.lower() in ("exit", "quit", "bye"):
            print("\n👋 Session ended.\n")
            break

        response = run_conversation_turn(
            client=client,
            agent_id=agent_id,
            thread_id=thread.id,
            user_message=user_input,
            memory=memory,
            user_id=user_id,
        )
        print(f"\nAgent: {response}\n")


def main():
    # Initialize clients
    credential = DefaultAzureCredential()
    client = AIProjectClient.from_connection_string(
        conn_str=os.environ["PROJECT_CONNECTION_STRING"],
        credential=credential,
    )
    memory = UserMemoryStore()

    # Create the agent once
    agent_id = create_agent(client)

    user_id = "user-demo-001"

    # Session 1 — Share preferences
    run_session(client, agent_id, memory, user_id, "SESSION 1: Introduce yourself")

    # Session 2 — Test recall
    run_session(client, agent_id, memory, user_id, "SESSION 2: Test memory recall")

    # Clean up the agent
    client.agents.delete_agent(agent_id)
    print("🧹 Agent deleted. Lab complete!")


if __name__ == "__main__":
    main()
```

### Step 3.3 — Review the Project Structure

Your project directory should now look like this:

```
📁 module-18-memory-agent/
├── .env
├── memory_store.py
├── memory_agent.py
├── main.py
└── test_memory.py
```

> **💡 Tip:** The agent is created once and reused across sessions. Each session creates a **new thread**, simulating a fresh conversation. The memory layer bridges the gap between threads by persisting user preferences in Cosmos DB.

---

## Part 4: Test Multi-Session Memory (10 minutes)

### Step 4.1 — Run Session 1: Share Your Preferences

Start the application:

```bash
python main.py
```

In **Session 1**, introduce yourself and share preferences:

```
============================================================
  SESSION 1: Introduce yourself
  User: user-demo-001
============================================================

📎 Thread created: thread_abc123

You: Hi! My name is Jordan and I'm a software developer.
💾 Saved preferences: {'name': 'Jordan', 'role': 'software developer'}

Agent: Nice to meet you, Jordan! Great to have a fellow software developer here.
       I'd love to learn more about your preferences so I can be more helpful.
       What programming languages or tools do you work with most?

You: I mainly work with Python and my timezone is US/Pacific.
💾 Saved preferences: {'language': 'Python', 'timezone': 'US/Pacific'}

Agent: Great choices! Python is such a versatile language. And now I know
       you're on Pacific time, so I can keep that in mind. Anything else
       you'd like me to remember?

You: exit
👋 Session ended.
```

### Step 4.2 — Run Session 2: Verify Memory Recall

Session 2 starts automatically with a **new thread**:

```
============================================================
  SESSION 2: Test memory recall
  User: user-demo-001
============================================================

📎 Thread created: thread_def456

You: Hey, do you remember anything about me?

Agent: Of course! You're Jordan, a software developer who works mainly
       with Python. You're in the US/Pacific timezone. Is there anything
       you'd like to update or add?

You: Actually, I've been switching to TypeScript lately.
💾 Saved preferences: {'language': 'TypeScript'}

Agent: Noted! I've updated your preferred language to TypeScript. That's a
       great choice — the type system really shines in larger projects.

You: exit
👋 Session ended.

🧹 Agent deleted. Lab complete!
```

### Step 4.3 — Verify Data in Cosmos DB

1. Open the Azure Portal → your Cosmos DB account → **Data Explorer**
2. Navigate to `agent-memory` → `user-preferences` → **Items**
3. Click on the document for `user-demo-001`
4. Verify the stored preferences:

```json
{
    "id": "user-demo-001",
    "userId": "user-demo-001",
    "preferences": {
        "name": "Jordan",
        "role": "software developer",
        "language": "TypeScript",
        "timezone": "US/Pacific"
    },
    "createdAt": "2026-04-15T10:30:00+00:00",
    "updatedAt": "2026-04-15T10:35:22+00:00"
}
```

> **💡 Tip:** Notice that `language` was updated from `"Python"` to `"TypeScript"` in Session 2. The upsert pattern in the memory store ensures updates are applied cleanly without duplicating the document.

---

## Validation & Testing

Use the checklist below to confirm your lab is working correctly:

| # | Validation Check | Expected Result | ✅ |
|---|-----------------|-----------------|-----|
| 1 | Cosmos DB container created | `user-preferences` container exists in `agent-memory` database | ☐ |
| 2 | Memory store test passes | `test_memory.py` prints all three ✅ messages | ☐ |
| 3 | Agent responds in Session 1 | Agent acknowledges preferences and responds conversationally | ☐ |
| 4 | Preferences saved to Cosmos DB | Document appears in Data Explorer with correct key-value pairs | ☐ |
| 5 | Session 2 uses a new thread | A new `thread_` ID is printed at the start of Session 2 | ☐ |
| 6 | Agent recalls preferences in Session 2 | Agent mentions your name, language, and timezone without being told | ☐ |
| 7 | Preference update works | Changing a preference in Session 2 updates the Cosmos DB document | ☐ |
| 8 | Agent deleted on exit | "Agent deleted" confirmation message appears | ☐ |

---

## Stretch Goals (Optional)

### Stretch 1 — Add TTL for Automatic Preference Expiry

Add a time-to-live (TTL) field so stale preferences expire automatically:

```python
# In memory_store.py — update the _upsert_preferences method
item["ttl"] = 60 * 60 * 24 * 90  # 90 days in seconds
```

Then enable TTL on the Cosmos DB container:
1. Open Data Explorer → `user-preferences` → **Settings**
2. Set **Time to Live** to **On**
3. Leave the default TTL blank (per-item TTL will be used)

### Stretch 2 — Add Conversation Summarization

After each session ends, use the agent to generate a summary and store it:

```python
def summarize_and_store(client, agent_id, thread_id, memory, user_id):
    """Generate a conversation summary and persist it as memory."""
    summary_prompt = (
        "Summarize our conversation in 2-3 sentences, focusing on "
        "what you learned about the user."
    )
    response = run_conversation_turn(
        client, agent_id, thread_id, summary_prompt, memory, user_id
    )
    memory.save_preference(user_id, "last_conversation_summary", response)
```

### Stretch 3 — Store Full Conversation History as Long-Term Memory

Create a separate Cosmos DB container `conversation-history` to store message logs:

```python
def save_conversation(self, user_id: str, thread_id: str, messages: list) -> None:
    """Archive a full conversation for long-term recall."""
    item = {
        "id": thread_id,
        "userId": user_id,
        "messages": messages,
        "timestamp": datetime.now(timezone.utc).isoformat(),
    }
    self.history_container.upsert_item(item)
```

### Stretch 4 — Add Vector Search for Semantic Memory Recall

Use Azure Cosmos DB vector search to find semantically relevant past conversations:

1. Enable vector indexing on the `conversation-history` container
2. Generate embeddings for conversation summaries using your AI Foundry embedding model
3. At the start of each session, search for relevant past conversations based on the user's first message

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `401 Unauthorized` from Cosmos DB | Run `az login` to refresh credentials. Ensure your identity has the **Cosmos DB Built-in Data Contributor** role on the account. |
| `ModuleNotFoundError: azure.ai.projects` | Install the correct package: `pip install azure-ai-projects`. Ensure you are using the right Python environment. |
| Agent returns empty responses | Check that your `gpt-4o` model deployment is active in AI Foundry. Verify the model name matches your deployment. |
| `ResourceNotFound` for database or container | Verify that the database is named `agent-memory` and the container is named `user-preferences` — names are case-sensitive. |
| Preferences not being extracted | Check the agent's response for the `[PREFERENCES_UPDATE]` tags. If missing, refine the agent instructions to be more explicit. |
| `EnvironmentError` on startup | Confirm your `.env` file is in the project root and contains both `PROJECT_CONNECTION_STRING` and `COSMOS_ENDPOINT`. |
| Run status is `failed` | Call `client.agents.get_run(thread_id, run_id)` and inspect `run.last_error` for details. Common cause: model deployment quota exceeded. |
| Slow responses from agent | The first run after agent creation may take longer. Subsequent turns are typically faster. Serverless Cosmos DB may also have cold-start latency. |

---

## Clean-up (Optional)

If you want to conserve costs after the lab:

1. Delete the agent (handled automatically by `main.py`)
2. Delete the Cosmos DB database: **Data Explorer** → `agent-memory` → **Delete Database**
3. Optionally delete the Cosmos DB account if no longer needed

> **Note:** Keep these resources if you plan to use them in Module 19 and later agent modules.

---

## Lab Summary

In this lab, you successfully:

- ✅ Provisioned Azure Cosmos DB as a persistent memory store
- ✅ Built a `UserMemoryStore` class with save, get, and update operations
- ✅ Created an Azure AI Foundry agent with memory-aware instructions
- ✅ Implemented cross-session preference injection via context messages
- ✅ Tested multi-session memory recall with a new thread per session
- ✅ Verified preference persistence and updates in Cosmos DB

## What's Next

In **Module 19**, you will extend your agent with tool-calling capabilities, enabling it to take real-world actions — such as querying databases, calling APIs, and executing workflows — in response to user requests.

---

*Azure AI Foundry: Zero to Hero Training Course — © 2026 Zero to Hero Training Team*
