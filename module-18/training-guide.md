# Module 18: Multi-Turn Conversations & Memory — Training Guide

## Azure AI Foundry: Zero to Hero Training Course

**ARC 4: AI AGENTS** | Module 18 of 45 | April 2026

---

## Module Overview

This module provides an in-depth exploration of multi-turn conversation management and memory patterns within Azure AI Foundry Agent Service. Participants will learn how to build agents that maintain coherent, context-aware conversations across multiple user interactions by leveraging the Threads API, implementing short-term and long-term memory architectures, managing context windows effectively, and persisting agent state using Azure storage services. By the end of this module, learners will understand how to design agents that remember user preferences, summarize past interactions, gracefully handle long conversations through truncation strategies, and store session data durably in services like Azure Cosmos DB, Redis, and Azure Table Storage. The Mini Hack puts all of these skills into practice by building a preference-aware agent from scratch using the Python SDK and Cosmos DB.

## Learning Objectives

By the end of this module, participants will be able to:

1. Create and manage conversation threads using the Azure AI Foundry Agent Service Threads API
2. Differentiate between short-term (in-context) memory and long-term (external store) memory patterns
3. Calculate and budget token usage across prompt and completion to stay within model context limits
4. Implement truncation strategies including auto truncation, `last_messages`, and custom summarization
5. Design persistent storage architectures for agent state using Cosmos DB, Table Storage, and Blob Storage
6. Apply semantic memory and episodic memory retrieval patterns to enhance agent recall
7. Configure `max_prompt_tokens` and `max_completion_tokens` parameters for deterministic token control
8. Build a fully functional multi-session, preference-remembering agent using the Python SDK

## Prerequisites

- Completion of Modules 1–17 (ARC 1, ARC 2, ARC 3, and ARC 4 through Module 17)
- Active Azure subscription with an Azure AI Foundry project provisioned
- Python 3.10+ installed with `azure-ai-projects` and `azure-identity` packages
- Azure Cosmos DB account provisioned (or ability to create one during the lab)
- Familiarity with Azure AI Agent Service concepts (agents, threads, runs, messages)
- Basic understanding of token-based language model mechanics

## Module Duration

| Section | Duration |
|---------|----------|
| Lecture & Concept Deep-Dive | 50 minutes |
| Featured Video | 20 minutes |
| Live Coding Demos | 25 minutes |
| Hands-on Lab (Mini Hack) | 45 minutes |
| Discussion & Quiz | 10 minutes |
| **Total** | **~2.5 hours** |

## Agenda

### 1. Introduction — Why Memory Matters for Agents (10 min)
- The illusion of memory in stateless LLMs
- Why multi-turn conversations require explicit memory management
- Real-world failures when agents "forget" mid-conversation
- Module roadmap and learning path within ARC 4

### 2. Threads and Conversation Management (15 min)
- The Azure AI Agent Service Threads API
- Creating, listing, retrieving, and deleting threads
- Adding user and assistant messages to threads
- Thread metadata and custom properties
- Conversation lifecycle management patterns
- Live demo: Creating a thread and running a multi-turn conversation

### 3. Short-Term vs. Long-Term Memory Architectures (10 min)
- In-context memory: messages stored within the thread
- Conversation summarization for compressing history
- External memory stores: Cosmos DB, Redis, Azure Table Storage
- Semantic memory (facts and knowledge) vs. episodic memory (events and experiences)
- Memory retrieval patterns: recency, relevance, importance scoring

### 4. Context Window Management and Token Budgeting (10 min)
- Understanding model context limits (GPT-4o: 128K tokens, GPT-4o-mini: 128K tokens)
- How token counts are calculated across system prompt, messages, tool definitions, and completions
- The `max_prompt_tokens` and `max_completion_tokens` parameters
- Prompt token budgeting strategies for production agents
- Monitoring token usage in run results

### 5. Truncation Strategies for Long Conversations (10 min)
- Auto truncation with `truncation_strategy` type `auto`
- The `last_messages` truncation parameter
- Custom summarization of earlier conversation segments
- Sliding window patterns for rolling context
- Graceful degradation when context limits are reached

### 6. Persistent Storage for Agent State (10 min)
- Azure Cosmos DB for user profiles and preference storage
- Azure Table Storage for session metadata
- Azure Blob Storage for conversation archives and audit logs
- Key-value patterns for fast state lookups
- TTL (Time-to-Live) configuration for automatic session expiry
- Choosing the right storage service for each use case

### 7. Featured Video (20 min)
- **"Build An AI Agent From Scratch Using Microsoft Foundry"**
- YouTube: https://www.youtube.com/watch?v=wL8H0RySf6I
- Key takeaways discussion after viewing

### 8. Live Coding Demos (25 min)
- Demo 1: Multi-turn conversation with threads and truncation
- Demo 2: Persisting user preferences to Cosmos DB between sessions
- Walkthrough of the Mini Hack starter code

### 9. Mini Hack: Build a Preference-Remembering Agent (45 min)
- Hands-on lab where participants build an agent that remembers user preferences across sessions
- Uses Python SDK (`azure-ai-projects`) + Azure Cosmos DB
- See `lab/hands-on-lab.md` for detailed step-by-step instructions

### 10. Discussion, Quiz & Wrap-Up (10 min)
- Group discussion questions
- 10 multiple-choice quiz questions covering key concepts
- Preview of Module 19
- See `quiz/assessment.md`

---

## Detailed Content Outline

---

### Section 1: Introduction — Why Memory Matters for Agents

Large Language Models are inherently stateless. Each API call is independent — the model has no built-in mechanism to remember what was said in a previous request. The "memory" that users perceive in chat applications is an engineering construct: previous messages are re-sent with every new request to simulate continuity.

This creates three fundamental challenges for agent builders:

1. **Context limits** — Every model has a finite context window. As conversations grow, earlier messages must be dropped or summarized.
2. **Cross-session continuity** — When a user returns hours or days later, the thread may be gone. The agent needs external storage to recall past interactions.
3. **Cost and latency** — Sending the full conversation history with every request increases token usage (and therefore cost) and response latency.

Azure AI Foundry Agent Service addresses these challenges through the Threads API (for in-session memory), truncation strategies (for context management), and integration patterns with Azure storage services (for cross-session persistence).

**Instructor Talking Point:** Ask the class to share examples of chatbot experiences where the bot "forgot" something important. Use these anecdotes to motivate the engineering solutions covered in this module.

---

### Section 2: Threads and Conversation Management

#### 2.1 The Threads API

The Azure AI Agent Service Threads API provides a managed, server-side conversation store. A **thread** is an ordered sequence of messages that represents a single conversation. The service handles storage, ordering, and retrieval — you do not need to manage message arrays yourself.

Key characteristics of threads:
- Threads are stored server-side by the Agent Service
- Each thread has a unique ID and optional metadata
- Messages within a thread are ordered chronologically
- Threads persist until explicitly deleted
- A single agent can participate in many threads concurrently

#### 2.2 Creating and Managing Threads

```python
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

# Initialize the client
project_client = AIProjectClient(
    credential=DefaultAzureCredential(),
    endpoint="https://<your-foundry-project>.services.ai.azure.com/api",
)

# Create a new thread
thread = project_client.agents.create_thread()
print(f"Thread created: {thread.id}")

# Create a thread with metadata
thread_with_meta = project_client.agents.create_thread(
    metadata={
        "user_id": "user-12345",
        "session_type": "support",
        "created_by": "onboarding-flow"
    }
)
```

#### 2.3 Adding Messages to a Thread

Messages can have a role of `user` or `assistant`. In most workflows, you add `user` messages to represent the human's input, then create a run to generate the assistant's response.

```python
# Add a user message to the thread
message = project_client.agents.create_message(
    thread_id=thread.id,
    role="user",
    content="I'm interested in learning about Azure AI Foundry. What can it do?"
)
print(f"Message added: {message.id}")

# Add additional context as a second user message
project_client.agents.create_message(
    thread_id=thread.id,
    role="user",
    content="Specifically, I want to know about building agents."
)
```

#### 2.4 Running the Agent on a Thread

When you create a run, the agent processes all messages in the thread and generates a response:

```python
# Create an agent
agent = project_client.agents.create_agent(
    model="gpt-4o",
    name="ConversationAgent",
    instructions="You are a helpful assistant that explains Azure AI Foundry concepts clearly and concisely."
)

# Create a run — the agent reads the full thread and responds
run = project_client.agents.create_run(
    thread_id=thread.id,
    agent_id=agent.id
)

# Poll until the run completes
import time
while run.status in ["queued", "in_progress"]:
    time.sleep(1)
    run = project_client.agents.get_run(thread_id=thread.id, run_id=run.id)

print(f"Run completed with status: {run.status}")
```

#### 2.5 Retrieving Messages

After a run completes, retrieve the updated message list to see the agent's response:

```python
# List all messages in the thread (newest first by default)
messages = project_client.agents.list_messages(thread_id=thread.id)

for msg in messages.data:
    role = msg.role
    for content_block in msg.content:
        if hasattr(content_block, "text"):
            print(f"[{role}]: {content_block.text.value}")
```

#### 2.6 Thread Lifecycle Management

```python
# Retrieve a specific thread
retrieved_thread = project_client.agents.get_thread(thread_id=thread.id)

# Update thread metadata
updated_thread = project_client.agents.update_thread(
    thread_id=thread.id,
    metadata={"status": "resolved", "satisfaction_score": "5"}
)

# Delete a thread when the conversation is complete
project_client.agents.delete_thread(thread_id=thread.id)
print("Thread deleted successfully.")
```

**Key Design Pattern:** In production applications, store the `thread.id` in your application's database (keyed by user ID or session ID) so that returning users can resume their conversation.

---

### Section 3: Short-Term vs. Long-Term Memory Architectures

#### 3.1 In-Context Memory (Short-Term)

The simplest form of agent memory is the messages already stored in the thread. When a run is created, the Agent Service sends the thread's message history to the model as part of the prompt. This is **in-context memory** — it works automatically but is limited by the model's context window.

Advantages:
- Zero additional infrastructure required
- Automatically managed by the Agent Service
- Perfect fidelity — the model sees the exact conversation

Limitations:
- Bounded by the model's context window (e.g., 128K tokens for GPT-4o)
- Older messages are lost if truncation is applied
- Does not persist across thread boundaries

#### 3.2 Conversation Summarization

When conversations grow long, a common pattern is to periodically summarize earlier messages and replace them with a condensed version. This compresses the conversation while preserving key information.

```python
def summarize_and_compact_thread(project_client, thread_id, agent_id, keep_recent=10):
    """
    Summarize older messages and keep only recent ones in the thread.
    Stores the summary as a system-level context message.
    """
    messages = project_client.agents.list_messages(thread_id=thread_id)
    all_messages = list(messages.data)

    if len(all_messages) <= keep_recent:
        return  # No summarization needed

    # Separate old messages from recent ones
    old_messages = all_messages[keep_recent:]
    
    # Build a summary prompt from the old messages
    history_text = "\n".join(
        f"{msg.role}: {msg.content[0].text.value}"
        for msg in reversed(old_messages)
        if msg.content and hasattr(msg.content[0], "text")
    )

    summary_prompt = f"""Summarize the following conversation history into a concise paragraph 
    that preserves all key facts, decisions, and user preferences:
    
    {history_text}"""

    # Use the agent to generate the summary (in a temporary thread)
    temp_thread = project_client.agents.create_thread()
    project_client.agents.create_message(
        thread_id=temp_thread.id, role="user", content=summary_prompt
    )
    summary_run = project_client.agents.create_run(
        thread_id=temp_thread.id, agent_id=agent_id
    )
    # ... poll for completion, retrieve summary, inject back into main thread ...
    project_client.agents.delete_thread(thread_id=temp_thread.id)
```

#### 3.3 External Memory Stores (Long-Term)

For cross-session memory, agents need external storage. The three most common patterns are:

| Memory Type | Storage Service | Use Case |
|-------------|----------------|----------|
| **User Profiles** | Azure Cosmos DB | Preferences, history, personalization |
| **Session State** | Azure Redis Cache | Fast ephemeral lookups, rate limiting |
| **Conversation Archives** | Azure Blob Storage | Audit trails, compliance, analytics |
| **Structured Facts** | Azure Table Storage | Key-value session metadata |

#### 3.4 Semantic Memory vs. Episodic Memory

These two memory paradigms, borrowed from cognitive science, apply directly to agent design:

- **Semantic Memory** stores facts and knowledge: "The user's preferred language is Spanish," "The user is a premium subscriber," "The user's timezone is PST." These are stable, factual attributes that rarely change.

- **Episodic Memory** stores events and experiences: "On March 5, the user asked about billing and was frustrated," "The user previously tried to deploy a model and encountered error X." These are temporal, narrative records of past interactions.

**Implementation Pattern:**

```python
# Semantic memory — stored as key-value pairs in Cosmos DB
semantic_memory = {
    "user_id": "user-12345",
    "preferred_language": "Spanish",
    "subscription_tier": "premium",
    "timezone": "America/Los_Angeles",
    "favorite_topics": ["AI agents", "Python", "cloud architecture"]
}

# Episodic memory — stored as timestamped event documents
episodic_memory = {
    "user_id": "user-12345",
    "event_type": "support_interaction",
    "timestamp": "2026-03-05T14:30:00Z",
    "summary": "User reported billing discrepancy. Issue escalated to finance team. Resolved within 24 hours.",
    "sentiment": "frustrated_then_satisfied",
    "thread_id": "thread_abc123"
}
```

#### 3.5 Memory Retrieval Patterns

When the agent starts a new session, it should retrieve relevant memory to inject into its system prompt or first message context. Common retrieval strategies:

1. **Recency** — Retrieve the N most recent episodic memories
2. **Relevance** — Use embedding similarity to find memories related to the current query
3. **Importance** — Weight memories by a computed importance score (e.g., unresolved issues rank higher)
4. **Hybrid** — Combine recency + relevance with a weighted scoring function

---

### Section 4: Context Window Management and Token Budgeting

#### 4.1 Understanding Model Context Limits

Every model has a maximum context window measured in tokens. The context window must accommodate everything sent to the model in a single run:

| Component | Description | Typical Token Count |
|-----------|-------------|-------------------|
| System Instructions | Agent's `instructions` parameter | 200–2,000 tokens |
| Tool Definitions | JSON schemas for all registered tools | 500–5,000 tokens |
| Thread Messages | All conversation messages included in the prompt | Variable |
| Retrieval Results | File search or knowledge base results | 0–20,000 tokens |
| **Completion** | The model's generated response | 500–4,000 tokens |

**Current model limits:**

| Model | Max Context Window | Recommended Prompt Budget |
|-------|-------------------|--------------------------|
| GPT-4o | 128,000 tokens | ~110,000 tokens (reserve 16K for completion) |
| GPT-4o-mini | 128,000 tokens | ~110,000 tokens (reserve 16K for completion) |

#### 4.2 Token Budgeting with `max_prompt_tokens` and `max_completion_tokens`

The Agent Service provides two parameters to enforce deterministic token limits on each run:

```python
run = project_client.agents.create_run(
    thread_id=thread.id,
    agent_id=agent.id,
    max_prompt_tokens=80000,      # Cap the prompt (system + messages + tools + retrieval)
    max_completion_tokens=16000   # Cap the model's response length
)
```

- **`max_prompt_tokens`** — Sets the upper limit on the total tokens consumed by the prompt. If the thread history plus system instructions plus tool definitions exceed this limit, the service applies truncation (see Section 5) to reduce the prompt size.
- **`max_completion_tokens`** — Sets the upper limit on the number of tokens the model may generate in its response. This prevents runaway responses and helps control cost.

**Best Practice:** Always set both parameters in production. This ensures predictable cost and prevents a single long conversation from consuming an unexpectedly large number of tokens.

#### 4.3 Monitoring Token Usage

After a run completes, inspect the `usage` property to understand actual token consumption:

```python
run = project_client.agents.get_run(thread_id=thread.id, run_id=run.id)

if run.usage:
    print(f"Prompt tokens used:     {run.usage.prompt_tokens}")
    print(f"Completion tokens used: {run.usage.completion_tokens}")
    print(f"Total tokens used:      {run.usage.total_tokens}")
```

**Instructor Tip:** Have participants run a 10-message conversation and observe how `prompt_tokens` grows with each turn. This viscerally demonstrates why truncation becomes necessary.

---

### Section 5: Truncation Strategies for Long Conversations

#### 5.1 The Truncation Problem

As conversations grow, the cumulative token count of all messages in a thread will eventually approach or exceed the model's context window. Without intervention, the run will fail with a context-length error. Truncation strategies solve this by intelligently reducing the number of messages sent to the model.

#### 5.2 Auto Truncation

The simplest approach is to let the Agent Service handle truncation automatically:

```python
from azure.ai.projects.models import TruncationObject

run = project_client.agents.create_run(
    thread_id=thread.id,
    agent_id=agent.id,
    truncation_strategy=TruncationObject(
        type="auto"
    ),
    max_prompt_tokens=80000
)
```

With `type="auto"`, the service will drop the oldest messages first to fit within the `max_prompt_tokens` budget. The system instructions and the most recent messages are always preserved.

#### 5.3 Last Messages Truncation

For finer control, use the `last_messages` parameter to specify exactly how many recent messages to include:

```python
run = project_client.agents.create_run(
    thread_id=thread.id,
    agent_id=agent.id,
    truncation_strategy=TruncationObject(
        type="last_messages",
        last_messages=20  # Only include the 20 most recent messages
    )
)
```

This is useful when you know the typical conversation length and want deterministic behavior. Messages beyond the `last_messages` count are excluded entirely from the prompt.

#### 5.4 Custom Summarization Pattern

For maximum control, implement a hybrid approach: summarize older messages and prepend the summary as context before the recent messages.

```python
def run_with_summarized_history(project_client, agent_id, thread_id, recent_count=10):
    """
    Retrieve all messages, summarize older ones, and create a run
    with the summary injected as context.
    """
    messages = project_client.agents.list_messages(thread_id=thread_id)
    all_msgs = list(messages.data)
    
    if len(all_msgs) > recent_count:
        # Summarize older messages (implementation as shown in Section 3.2)
        older_msgs = all_msgs[recent_count:]
        summary = generate_summary(project_client, agent_id, older_msgs)
        
        # Add summary as additional instructions for this run
        run = project_client.agents.create_run(
            thread_id=thread_id,
            agent_id=agent_id,
            additional_instructions=f"Summary of earlier conversation: {summary}",
            truncation_strategy=TruncationObject(
                type="last_messages",
                last_messages=recent_count
            )
        )
    else:
        run = project_client.agents.create_run(
            thread_id=thread_id,
            agent_id=agent_id
        )
    
    return run
```

#### 5.5 Sliding Window Pattern

A sliding window maintains a fixed-size window of the most recent messages while archiving older messages to external storage:

```python
WINDOW_SIZE = 20  # Maximum messages to keep in the active thread

def sliding_window_add_message(project_client, thread_id, role, content, archive_container):
    """
    Add a message to the thread. If the thread exceeds WINDOW_SIZE,
    archive the oldest messages to Blob Storage and remove them.
    """
    # Add the new message
    project_client.agents.create_message(
        thread_id=thread_id,
        role=role,
        content=content
    )
    
    # Check thread length
    messages = project_client.agents.list_messages(thread_id=thread_id)
    all_msgs = list(messages.data)
    
    if len(all_msgs) > WINDOW_SIZE:
        overflow = all_msgs[WINDOW_SIZE:]
        # Archive overflow messages to Azure Blob Storage
        archive_to_blob(archive_container, thread_id, overflow)
        # Note: The Agent Service does not support deleting individual messages.
        # Use truncation_strategy with last_messages=WINDOW_SIZE at run time instead.
```

#### 5.6 Choosing the Right Strategy

| Strategy | Best For | Trade-Off |
|----------|----------|-----------|
| `auto` | Simple applications, prototyping | Least control over what is dropped |
| `last_messages` | Predictable conversation patterns | Older context is completely lost |
| Custom summarization | High-value conversations | Adds latency and cost for summarization |
| Sliding window + archive | Compliance, audit requirements | Additional storage infrastructure needed |

---

### Section 6: Persistent Storage for Agent State

#### 6.1 Why Persistent Storage?

Threads in the Agent Service are designed for in-conversation state. For cross-session state — such as user preferences, accumulated knowledge, or historical interaction patterns — you need external persistent storage.

#### 6.2 Azure Cosmos DB for User Profiles

Cosmos DB is the recommended choice for user profile and preference storage due to its low latency, global distribution, and flexible schema.

```python
from azure.cosmos import CosmosClient, PartitionKey

# Initialize Cosmos DB client
cosmos_client = CosmosClient(
    url="https://<your-cosmos-account>.documents.azure.com:443/",
    credential=DefaultAzureCredential()
)

# Create or get database and container
database = cosmos_client.create_database_if_not_exists(id="agent-memory")
container = database.create_container_if_not_exists(
    id="user-profiles",
    partition_key=PartitionKey(path="/user_id"),
    default_ttl=-1  # No expiry for user profiles
)

# Store user preferences
def save_user_preferences(container, user_id, preferences):
    """Save or update a user's preference document."""
    document = {
        "id": user_id,
        "user_id": user_id,
        "preferences": preferences,
        "updated_at": datetime.utcnow().isoformat()
    }
    container.upsert_item(document)

# Retrieve user preferences
def get_user_preferences(container, user_id):
    """Retrieve a user's stored preferences."""
    try:
        item = container.read_item(item=user_id, partition_key=user_id)
        return item.get("preferences", {})
    except Exception:
        return {}  # New user, no preferences yet

# Usage
save_user_preferences(container, "user-12345", {
    "language": "Spanish",
    "response_style": "concise",
    "topics_of_interest": ["cloud architecture", "AI agents"],
    "timezone": "America/Los_Angeles"
})
```

#### 6.3 Azure Table Storage for Session Metadata

For lightweight, structured session data with fast lookups, Azure Table Storage is cost-effective and simple:

```python
from azure.data.tables import TableServiceClient

table_service = TableServiceClient(
    endpoint="https://<your-storage>.table.core.windows.net",
    credential=DefaultAzureCredential()
)
table_client = table_service.create_table_if_not_exists("agentsessions")

# Store session data
session_entity = {
    "PartitionKey": "user-12345",
    "RowKey": "session-2026-04-15-001",
    "ThreadId": thread.id,
    "AgentId": agent.id,
    "StartedAt": "2026-04-15T10:30:00Z",
    "Status": "active",
    "MessageCount": 0,
    "Topic": "Azure deployment troubleshooting"
}
table_client.upsert_entity(session_entity)

# Query active sessions for a user
sessions = table_client.query_entities(
    query_filter="PartitionKey eq 'user-12345' and Status eq 'active'"
)
for session in sessions:
    print(f"Active session: {session['RowKey']} — Topic: {session['Topic']}")
```

#### 6.4 Azure Blob Storage for Conversation Archives

For compliance, audit, or analytics purposes, archive complete conversations to Blob Storage:

```python
from azure.storage.blob import BlobServiceClient
import json

blob_service = BlobServiceClient(
    account_url="https://<your-storage>.blob.core.windows.net",
    credential=DefaultAzureCredential()
)
archive_container = blob_service.get_container_client("conversation-archives")

def archive_conversation(project_client, thread_id, user_id):
    """Archive a complete conversation thread to Blob Storage."""
    messages = project_client.agents.list_messages(thread_id=thread_id)
    
    archive_data = {
        "thread_id": thread_id,
        "user_id": user_id,
        "archived_at": datetime.utcnow().isoformat(),
        "messages": [
            {
                "id": msg.id,
                "role": msg.role,
                "content": [
                    block.text.value for block in msg.content 
                    if hasattr(block, "text")
                ],
                "created_at": msg.created_at
            }
            for msg in messages.data
        ]
    }
    
    blob_name = f"{user_id}/{thread_id}.json"
    archive_container.upload_blob(
        name=blob_name,
        data=json.dumps(archive_data, indent=2),
        overwrite=True
    )
    print(f"Conversation archived to: {blob_name}")
```

#### 6.5 TTL for Automatic Session Expiry

Configure Time-to-Live (TTL) on storage documents to automatically clean up stale sessions:

```python
# Cosmos DB — set TTL at document level (seconds)
session_document = {
    "id": "session-xyz",
    "user_id": "user-12345",
    "thread_id": thread.id,
    "created_at": datetime.utcnow().isoformat(),
    "ttl": 86400  # Auto-delete after 24 hours
}
container.upsert_item(session_document)

# For the container default TTL (set during creation):
container = database.create_container_if_not_exists(
    id="active-sessions",
    partition_key=PartitionKey(path="/user_id"),
    default_ttl=3600  # All documents expire after 1 hour unless overridden
)
```

#### 6.6 Putting It All Together — Injecting Memory into Agent Context

The final pattern ties persistent storage back to the agent's conversation by loading stored memory and injecting it into the system prompt or additional instructions:

```python
def create_memory_aware_run(project_client, agent_id, thread_id, user_id, cosmos_container):
    """
    Create a run that includes the user's stored preferences and
    recent interaction history as additional context.
    """
    # Load user preferences from Cosmos DB
    preferences = get_user_preferences(cosmos_container, user_id)
    
    # Build memory context string
    memory_context = "User Memory Context:\n"
    if preferences:
        memory_context += f"- Preferred language: {preferences.get('language', 'English')}\n"
        memory_context += f"- Response style: {preferences.get('response_style', 'balanced')}\n"
        memory_context += f"- Topics of interest: {', '.join(preferences.get('topics_of_interest', []))}\n"
        memory_context += f"- Timezone: {preferences.get('timezone', 'UTC')}\n"
    
    # Create the run with memory injected as additional instructions
    run = project_client.agents.create_run(
        thread_id=thread_id,
        agent_id=agent_id,
        additional_instructions=memory_context,
        max_prompt_tokens=80000,
        max_completion_tokens=4096
    )
    
    return run
```

---

### Section 7: Featured Video

**"Build An AI Agent From Scratch Using Microsoft Foundry"**

- **URL:** https://www.youtube.com/watch?v=wL8H0RySf6I
- **Runtime:** ~20 minutes
- **Focus Areas:** Agent creation workflow, thread management, and tool integration using Microsoft Foundry

**Viewing Instructions:**
1. Watch the video as a group or have participants watch independently
2. Pay special attention to how the presenter manages conversation threads
3. Note the patterns used for maintaining context across interactions
4. After viewing, discuss how the demonstrated patterns relate to the memory architectures covered in Sections 2–6

---

### Section 8: Live Coding Demos

#### Demo 1: Multi-Turn Conversation with Truncation

Walk through a complete multi-turn conversation where the thread grows beyond a manageable size, then demonstrate the effect of different truncation strategies.

```python
from azure.ai.projects import AIProjectClient
from azure.ai.projects.models import TruncationObject
from azure.identity import DefaultAzureCredential
import time

# Setup
project_client = AIProjectClient(
    credential=DefaultAzureCredential(),
    endpoint="https://<your-project>.services.ai.azure.com/api",
)

agent = project_client.agents.create_agent(
    model="gpt-4o-mini",
    name="TruncationDemo",
    instructions="You are a helpful travel planning assistant. Keep track of all destinations the user mentions."
)

thread = project_client.agents.create_thread()

# Simulate a long conversation
user_messages = [
    "I want to plan a trip to Japan. What cities should I visit?",
    "Tell me more about Tokyo. What are the must-see spots?",
    "What about Kyoto? I love temples.",
    "How about Osaka for food?",
    "Can you compare Hiroshima and Nagasaki for a day trip?",
    "I also want to visit Mount Fuji. Best time of year?",
    "Let's add Nara to the itinerary for the deer park.",
    "What's the best rail pass for all these cities?",
    "Now summarize my complete itinerary with all cities we discussed.",
]

for user_msg in user_messages:
    project_client.agents.create_message(
        thread_id=thread.id, role="user", content=user_msg
    )
    
    # Run WITHOUT truncation
    run = project_client.agents.create_run(
        thread_id=thread.id, agent_id=agent.id
    )
    while run.status in ["queued", "in_progress"]:
        time.sleep(1)
        run = project_client.agents.get_run(thread_id=thread.id, run_id=run.id)
    
    print(f"Turn {user_messages.index(user_msg) + 1} — Tokens: {run.usage.prompt_tokens}")

# Now run the SAME final question with last_messages=4 truncation
print("\n--- With last_messages=4 truncation ---")
run_truncated = project_client.agents.create_run(
    thread_id=thread.id,
    agent_id=agent.id,
    truncation_strategy=TruncationObject(type="last_messages", last_messages=4),
)
# ... poll and show the result — the agent will NOT remember early cities
```

**Discussion Point:** Ask participants: "What cities do you think the agent will include in the truncated summary? Why?"

#### Demo 2: Persisting Preferences to Cosmos DB

Demonstrate the full end-to-end flow: user states a preference → agent extracts it → preference is stored in Cosmos DB → new session loads the preference → agent uses it.

This demo uses the code patterns from Section 6.2 and 6.6.

---

### Section 9: Mini Hack — Build a Preference-Remembering Agent

**Objective:** Build an AI agent that remembers user preferences (favorite color, preferred communication style, topics of interest) across multiple sessions using the Python SDK and Azure Cosmos DB.

**Architecture:**

```
User ──► Your App ──► Azure AI Agent Service (Thread + Agent)
                │                    │
                │                    ▼
                │            GPT-4o (generates responses)
                │
                └──► Azure Cosmos DB (stores/retrieves preferences)
```

**Steps:**

1. **Setup Azure Resources** — Ensure an AI Foundry project and Cosmos DB account are provisioned
2. **Create the Cosmos DB container** — `agent-memory` database, `user-profiles` container, partitioned by `/user_id`
3. **Create the agent** — With instructions that tell it to identify and report user preferences
4. **Implement Session 1** — User states preferences in conversation; app extracts and stores them
5. **Implement Session 2** — New thread is created; app loads stored preferences and injects them as `additional_instructions`; agent greets user by referencing their stored preferences
6. **Implement truncation** — Add `max_prompt_tokens` and `truncation_strategy` to handle long conversations
7. **Test and validate** — Verify that preferences persist across thread boundaries

See `lab/hands-on-lab.md` for the complete step-by-step walkthrough with starter code and validation checks.

---

## Instructor Notes

- **Emphasize the distinction:** Azure AI Foundry Agent Service is NOT Azure Machine Learning. The Threads API, runs, and agents are specific to Foundry's Agent Service.
- **Common misconception:** Students often assume the model "remembers" previous conversations automatically. Reinforce that memory is an engineering responsibility — the model is stateless.
- **Lab preparation:** Pre-provision a shared Cosmos DB account if participants do not have their own Azure subscriptions. Provide connection strings via Azure Key Vault or environment variables.
- **Token budgeting exercise:** Before the lab, have participants calculate the token budget for a 50-message conversation with a 500-token system prompt. This builds intuition for when truncation kicks in.
- **Pacing tip:** The live coding demos and Mini Hack are the highest-value sections. If time is short, compress the lecture portions (Sections 3 and 4) and allocate more time to hands-on work.
- **Fallback plan:** If Cosmos DB provisioning fails during the lab, use a Python dictionary as a mock in-memory store. The patterns are identical — only the persistence layer changes.
- **Security reminder:** Never hard-code connection strings or keys in Python scripts. Always use `DefaultAzureCredential` or environment variables. Demonstrate this explicitly in every code example.
- **Extension activity:** Advanced participants can extend the Mini Hack to include episodic memory — storing a summary of each session and retrieving relevant past episodes using keyword matching.

## Discussion Questions

1. **Memory architecture trade-offs:** An agent for a healthcare triage system needs to remember patient history across visits but must comply with data retention regulations. How would you design the memory architecture? Which storage services would you choose, and what TTL policies would you apply?

2. **Truncation impact on quality:** When using `last_messages=10` truncation on a 50-message conversation, the agent loses access to 40 messages of context. How might this affect response quality? What strategies can mitigate information loss while keeping token usage manageable?

3. **Semantic vs. episodic memory in practice:** Consider a customer support agent that handles billing, technical, and account questions. Which user information should be stored as semantic memory (facts) vs. episodic memory (events)? Give specific examples of each.

4. **Cost implications of memory:** Sending a full 100-message conversation history with every run costs significantly more than sending a 10-message window with a summary. How would you design a system that balances response quality with token cost? At what conversation length would you trigger summarization?

5. **Cross-session consistency:** A user tells the agent "I prefer responses in French" in Session 1. In Session 2, the user's stored preference is loaded, but they send their first message in English. How should the agent handle this conflict? What design patterns ensure the agent respects the most recent user intent?

6. **Privacy and memory deletion:** A user requests that the agent "forget everything about me." What steps must your application take to fully honor this request across threads, Cosmos DB documents, Blob Storage archives, and any cached data? How does GDPR's "right to be forgotten" apply to agent memory systems?

7. **Scaling memory retrieval:** Your agent serves 1 million users, each with hundreds of episodic memories. How would you design the memory retrieval layer to keep latency under 200ms? What indexing, partitioning, and caching strategies would you use in Cosmos DB?

## Additional Resources

- [Azure AI Foundry Agent Service Documentation](https://learn.microsoft.com/azure/ai-services/agents/)
- [Azure AI Foundry Portal](https://ai.azure.com)
- [Threads API Reference — Azure AI Agent Service](https://learn.microsoft.com/azure/ai-services/agents/concepts/threads)
- [Azure AI Projects Python SDK (`azure-ai-projects`)](https://learn.microsoft.com/python/api/overview/azure/ai-projects-readme)
- [Azure Cosmos DB Python SDK](https://learn.microsoft.com/azure/cosmos-db/nosql/sdk-python)
- [Context Window Management Best Practices](https://learn.microsoft.com/azure/ai-services/agents/concepts/tokens)
- [Azure Table Storage Python SDK](https://learn.microsoft.com/azure/storage/tables/table-storage-quickstart-python)
- [Featured Video: Build An AI Agent From Scratch Using Microsoft Foundry](https://www.youtube.com/watch?v=wL8H0RySf6I)

## Navigation

| Previous | Next |
|----------|------|
| [Module 17 →](../module-17/training-guide.md) | [Module 19 →](../module-19/training-guide.md) |

---

*Azure AI Foundry: Zero to Hero Training Course — © 2026 Zero to Hero Training Team*
