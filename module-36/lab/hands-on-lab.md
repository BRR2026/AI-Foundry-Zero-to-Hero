# Hands-On Lab: Build a 3-Agent Content Pipeline

## Module 36 — Multi-Agent Systems & Orchestration

| Field | Details |
|---|---|
| **Duration** | 90 minutes |
| **Difficulty** | Advanced |
| **Arc** | 8 — Advanced Patterns & Enterprise |
| **Prerequisites** | Azure AI Foundry project, GPT-4o deployment, Python 3.10+, Azure CLI |

---

## Learning Objectives

By completing this lab, you will:

1. ✅ Create three specialized Azure AI Foundry agents with distinct roles
2. ✅ Build a supervisor orchestrator that coordinates agent execution
3. ✅ Implement agent-to-agent communication via message passing and shared state
4. ✅ Add error handling with timeouts, retries, and circuit breakers
5. ✅ Run a complete research → writing → review → revision pipeline
6. ✅ Monitor agent interactions with structured logging

---

## Prerequisites

| Requirement | Details |
|---|---|
| **Azure Subscription** | Active subscription with Azure AI Foundry access |
| **AI Foundry Project** | Existing project with endpoint URL |
| **Model Deployment** | GPT-4o deployed in your project |
| **Bing Grounding** | Bing Grounding connection configured (for researcher agent) |
| **Python** | 3.10 or later |
| **Packages** | `azure-ai-projects`, `azure-identity`, `tenacity` |
| **Azure CLI** | Authenticated via `az login` |

---

## Exercise 1: Environment Setup (15 minutes)

### Step 1.1: Create the Project Directory

```bash
mkdir multi-agent-lab
cd multi-agent-lab
```

### Step 1.2: Install Dependencies

```bash
pip install azure-ai-projects azure-identity tenacity
```

### Step 1.3: Set Environment Variables

```bash
# Windows (PowerShell)
$env:PROJECT_ENDPOINT = "https://<your-project>.services.ai.azure.com/api"

# Linux/macOS
export PROJECT_ENDPOINT="https://<your-project>.services.ai.azure.com/api"
```

### Step 1.4: Verify Authentication

Create a file named `verify_setup.py`:

```python
import os
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

client = AIProjectClient(
    credential=DefaultAzureCredential(),
    project_endpoint=os.environ["PROJECT_ENDPOINT"]
)

print("✅ Successfully connected to Azure AI Foundry!")
print(f"   Endpoint: {os.environ['PROJECT_ENDPOINT']}")
```

Run:

```bash
python verify_setup.py
```

**Expected output:** `✅ Successfully connected to Azure AI Foundry!`

---

## Exercise 2: Create Specialist Agents (20 minutes)

### Step 2.1: Create the Agent Definitions File

Create `agents.py`:

```python
"""Agent definitions for the multi-agent content pipeline."""

import os
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

client = AIProjectClient(
    credential=DefaultAzureCredential(),
    project_endpoint=os.environ["PROJECT_ENDPOINT"]
)

RESEARCHER_INSTRUCTIONS = """You are a meticulous research analyst.

Your job is to:
1. Break down the research topic into key questions
2. Gather comprehensive information from available sources
3. Synthesize findings into a structured research brief
4. Cite sources and flag uncertainty levels

Output format:
## Key Findings
- Finding 1 (confidence: high/medium/low)
- Finding 2 (confidence: high/medium/low)
...

## Sources
- Source 1: [description and URL]
- Source 2: [description and URL]

## Open Questions
- Questions that need further investigation

Keep findings factual and well-organized. Do NOT embellish or speculate."""

WRITER_INSTRUCTIONS = """You are a professional technical writer.

Your job is to:
1. Transform research findings into clear, engaging content
2. Structure the article with logical flow and headings
3. Use concrete examples and analogies for complex concepts
4. Maintain accuracy — never embellish beyond the research data

Style guidelines:
- Professional but accessible tone
- Short paragraphs (3-4 sentences max)
- Use bullet points for lists of 3+ items
- Include a TL;DR summary at the top
- Use markdown formatting for structure

Output a complete, polished article ready for review."""

REVIEWER_INSTRUCTIONS = """You are a senior content reviewer and editor.

Evaluate the submitted draft on these criteria:
1. **Accuracy**: Are all claims supported by the provided research?
2. **Clarity**: Is the writing clear, free of jargon, and accessible?
3. **Completeness**: Are the key topics adequately covered?
4. **Structure**: Is the flow logical, well-organized, with proper headings?

Scoring: Rate each criterion 1-5 (1=poor, 5=excellent).

If ALL scores are 4 or higher, respond with "APPROVED" at the end.
Otherwise, provide SPECIFIC and ACTIONABLE revision suggestions.

Required output format:
## Review Scores
- Accuracy: X/5
- Clarity: X/5
- Completeness: X/5
- Structure: X/5
- Overall: X/5

## Detailed Feedback
[Specific, actionable suggestions for improvement. Reference exact sections.]

## Verdict
APPROVED or REVISION_NEEDED"""


def create_agents():
    """Create all three specialist agents."""
    
    # Researcher Agent — with Bing Grounding for web search
    researcher = client.agents.create_agent(
        model="gpt-4o",
        name="ResearchAgent",
        instructions=RESEARCHER_INSTRUCTIONS,
        tools=[{"type": "bing_grounding"}]
    )
    print(f"✅ Created Researcher Agent: {researcher.id}")

    # Writer Agent — focused on content creation
    writer = client.agents.create_agent(
        model="gpt-4o",
        name="WriterAgent",
        instructions=WRITER_INSTRUCTIONS
    )
    print(f"✅ Created Writer Agent: {writer.id}")

    # Reviewer Agent — quality assurance
    reviewer = client.agents.create_agent(
        model="gpt-4o",
        name="ReviewerAgent",
        instructions=REVIEWER_INSTRUCTIONS
    )
    print(f"✅ Created Reviewer Agent: {reviewer.id}")

    return researcher, writer, reviewer


def cleanup_agents(*agents):
    """Delete all agents to clean up resources."""
    for agent in agents:
        client.agents.delete_agent(agent.id)
        print(f"🗑️  Deleted agent: {agent.id}")


if __name__ == "__main__":
    r, w, rv = create_agents()
    print("\nAgent IDs:")
    print(f"  Researcher: {r.id}")
    print(f"  Writer:     {w.id}")
    print(f"  Reviewer:   {rv.id}")
    
    input("\nPress Enter to cleanup agents...")
    cleanup_agents(r, w, rv)
    print("✅ All agents cleaned up!")
```

### Step 2.2: Test Agent Creation

```bash
python agents.py
```

**Expected output:**
```
✅ Created Researcher Agent: asst_abc123...
✅ Created Writer Agent: asst_def456...
✅ Created Reviewer Agent: asst_ghi789...
```

> **Note:** Press Enter when prompted to delete the agents. In Exercise 3, we'll keep them alive for the full pipeline.

---

## Exercise 3: Build the Supervisor Orchestrator (25 minutes)

### Step 3.1: Create the Workflow State Manager

Create `state.py`:

```python
"""Shared state management for multi-agent workflows."""

from datetime import datetime
from typing import Any


class WorkflowState:
    """Thread-safe shared state for multi-agent workflows."""

    def __init__(self):
        self.data = {
            "research_results": None,
            "draft_content": None,
            "review_feedback": [],
            "revision_count": 0,
            "status": "initialized",
            "agent_logs": [],
        }

    def update(self, key: str, value: Any, agent_id: str):
        """Update a state value and log the change."""
        self.data[key] = value
        self.data["agent_logs"].append({
            "agent": agent_id,
            "action": f"updated '{key}'",
            "timestamp": datetime.utcnow().isoformat(),
        })
        print(f"  📝 [{agent_id}] updated '{key}'")

    def get(self, key: str) -> Any:
        """Retrieve a state value."""
        return self.data.get(key)

    def get_logs(self) -> list:
        """Return all agent activity logs."""
        return self.data["agent_logs"]

    def summary(self) -> dict:
        """Return a summary of the current state."""
        return {
            "status": self.data["status"],
            "revision_count": self.data["revision_count"],
            "has_research": self.data["research_results"] is not None,
            "has_draft": self.data["draft_content"] is not None,
            "feedback_count": len(self.data["review_feedback"]),
            "log_entries": len(self.data["agent_logs"]),
        }
```

### Step 3.2: Create the Supervisor Orchestrator

Create `orchestrator.py`:

```python
"""Supervisor orchestrator for the multi-agent content pipeline."""

import os
import logging
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from state import WorkflowState

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger("orchestrator")

client = AIProjectClient(
    credential=DefaultAzureCredential(),
    project_endpoint=os.environ["PROJECT_ENDPOINT"]
)


class SupervisorOrchestrator:
    """Coordinates a team of specialized agents using the Supervisor pattern."""

    def __init__(self, max_revisions: int = 3):
        self.agents = {}
        self.state = WorkflowState()
        self.max_revisions = max_revisions

    def register_agent(self, role: str, agent):
        """Register an agent under a named role."""
        self.agents[role] = agent
        logger.info(f"Registered agent '{role}': {agent.id}")

    def dispatch(self, role: str, task: str) -> str:
        """Send a task to an agent and return the response."""
        agent = self.agents[role]
        logger.info(f"Dispatching to '{role}' ({agent.id})...")

        # Create a new thread for this interaction
        thread = client.agents.threads.create()

        # Send the task
        client.agents.messages.create(
            thread_id=thread.id,
            role="user",
            content=task
        )

        # Run the agent
        run = client.agents.runs.create_and_process(
            thread_id=thread.id,
            agent_id=agent.id
        )

        # Check run status
        if run.status != "completed":
            raise RuntimeError(
                f"Agent '{role}' run failed with status: {run.status}"
            )

        # Extract the response
        messages = client.agents.messages.list(thread_id=thread.id)
        response = messages.data[0].content[0].text.value

        logger.info(f"'{role}' responded ({len(response)} chars)")
        return response

    def run_pipeline(self, topic: str) -> str:
        """Execute the full Researcher → Writer → Reviewer pipeline."""
        print(f"\n{'='*60}")
        print(f"  🚀 Starting multi-agent pipeline")
        print(f"  📝 Topic: {topic}")
        print(f"{'='*60}\n")

        # --- Phase 1: Research ---
        print("📌 Phase 1: Research")
        self.state.update("status", "researching", "supervisor")
        
        research = self.dispatch(
            "researcher",
            f"Research the following topic thoroughly and provide a "
            f"structured research brief:\n\n{topic}"
        )
        self.state.update("research_results", research, "researcher")
        print(f"   ✅ Research complete ({len(research)} chars)\n")

        # --- Phase 2: Writing ---
        print("📌 Phase 2: Writing")
        self.state.update("status", "drafting", "supervisor")
        
        draft = self.dispatch(
            "writer",
            f"Write a comprehensive, well-structured article based on "
            f"the following research:\n\n{research}"
        )
        self.state.update("draft_content", draft, "writer")
        print(f"   ✅ Draft complete ({len(draft)} chars)\n")

        # --- Phase 3: Review & Revision Loop ---
        revision_count = 0
        current_draft = draft

        while revision_count < self.max_revisions:
            print(f"📌 Phase 3: Review (iteration {revision_count + 1})")
            self.state.update("status", "reviewing", "supervisor")

            review = self.dispatch(
                "reviewer",
                f"Review the following article for quality:\n\n{current_draft}"
            )
            self.state.update("review_feedback", review, "reviewer")
            print(f"   📝 Review received ({len(review)} chars)")

            # Check if approved
            if "APPROVED" in review.upper():
                print("   ✅ Article APPROVED by reviewer!\n")
                break

            # Revise
            revision_count += 1
            self.state.update("revision_count", revision_count, "supervisor")
            print(f"   🔄 Revision needed (attempt {revision_count}/{self.max_revisions})")

            self.state.update("status", "revising", "supervisor")
            current_draft = self.dispatch(
                "writer",
                f"Please revise the following article based on the reviewer's "
                f"feedback:\n\n"
                f"--- CURRENT DRAFT ---\n{current_draft}\n\n"
                f"--- REVIEWER FEEDBACK ---\n{review}\n\n"
                f"Address all feedback points and produce an improved version."
            )
            self.state.update("draft_content", current_draft, "writer")
            print(f"   ✅ Revision complete ({len(current_draft)} chars)\n")
        else:
            print(f"   ⚠️ Max revisions ({self.max_revisions}) reached.\n")

        # --- Done ---
        self.state.update("status", "complete", "supervisor")
        print(f"{'='*60}")
        print(f"  ✅ Pipeline complete!")
        print(f"  📊 Revisions: {revision_count}")
        print(f"  📏 Final article: {len(current_draft)} chars")
        print(f"{'='*60}\n")

        return current_draft
```

### Step 3.3: Create the Main Runner

Create `main.py`:

```python
"""Main entry point for the multi-agent content pipeline."""

from agents import create_agents, cleanup_agents
from orchestrator import SupervisorOrchestrator


def main():
    # Step 1: Create the agents
    print("🔧 Creating agents...")
    researcher, writer, reviewer = create_agents()

    # Step 2: Set up the orchestrator
    orchestrator = SupervisorOrchestrator(max_revisions=3)
    orchestrator.register_agent("researcher", researcher)
    orchestrator.register_agent("writer", writer)
    orchestrator.register_agent("reviewer", reviewer)

    try:
        # Step 3: Run the pipeline
        topic = (
            "The impact of multi-agent AI systems on enterprise "
            "software development: current capabilities, real-world "
            "use cases, and future trends."
        )
        
        result = orchestrator.run_pipeline(topic)

        # Step 4: Save the output
        with open("output.md", "w", encoding="utf-8") as f:
            f.write(result)
        print(f"📄 Output saved to output.md")

        # Step 5: Print state summary
        print("\n📊 Workflow State Summary:")
        summary = orchestrator.state.summary()
        for key, value in summary.items():
            print(f"   {key}: {value}")

        # Step 6: Print activity log
        print("\n📋 Agent Activity Log:")
        for log in orchestrator.state.get_logs():
            print(f"   [{log['agent']}] {log['action']} @ {log['timestamp']}")

    finally:
        # Step 7: Clean up agents
        print("\n🧹 Cleaning up agents...")
        cleanup_agents(researcher, writer, reviewer)


if __name__ == "__main__":
    main()
```

### Step 3.4: Run the Pipeline

```bash
python main.py
```

**Expected output flow:**
```
🔧 Creating agents...
✅ Created Researcher Agent: asst_...
✅ Created Writer Agent: asst_...
✅ Created Reviewer Agent: asst_...

============================================================
  🚀 Starting multi-agent pipeline
  📝 Topic: The impact of multi-agent AI systems...
============================================================

📌 Phase 1: Research
   ✅ Research complete (2345 chars)

📌 Phase 2: Writing
   ✅ Draft complete (3456 chars)

📌 Phase 3: Review (iteration 1)
   📝 Review received (890 chars)
   🔄 Revision needed (attempt 1/3)
   ✅ Revision complete (3678 chars)

📌 Phase 3: Review (iteration 2)
   📝 Review received (456 chars)
   ✅ Article APPROVED by reviewer!

============================================================
  ✅ Pipeline complete!
  📊 Revisions: 1
  📏 Final article: 3678 chars
============================================================
```

---

## Exercise 4: Add Error Handling (20 minutes)

### Step 4.1: Create a Resilient Dispatch Wrapper

Create `resilience.py`:

```python
"""Error handling and resilience utilities for multi-agent workflows."""

import time
import logging
from functools import wraps
from tenacity import retry, stop_after_attempt, wait_exponential

logger = logging.getLogger("resilience")


class CircuitBreaker:
    """Simple circuit breaker for agent dispatch."""

    def __init__(self, failure_threshold: int = 5, reset_timeout: int = 60):
        self.failure_threshold = failure_threshold
        self.reset_timeout = reset_timeout
        self.failures = {}
        self.last_failure_time = {}

    def record_success(self, agent_id: str):
        self.failures[agent_id] = 0

    def record_failure(self, agent_id: str):
        self.failures[agent_id] = self.failures.get(agent_id, 0) + 1
        self.last_failure_time[agent_id] = time.time()
        logger.warning(
            f"Agent '{agent_id}' failure count: "
            f"{self.failures[agent_id]}/{self.failure_threshold}"
        )

    def is_open(self, agent_id: str) -> bool:
        failures = self.failures.get(agent_id, 0)
        if failures < self.failure_threshold:
            return False
        
        # Check if reset timeout has passed
        last_failure = self.last_failure_time.get(agent_id, 0)
        if time.time() - last_failure > self.reset_timeout:
            logger.info(f"Circuit breaker reset for '{agent_id}'")
            self.failures[agent_id] = 0
            return False

        return True


def validate_response(response: str, min_length: int = 50) -> bool:
    """Validate that an agent response meets basic quality criteria."""
    if not response or not response.strip():
        logger.warning("Empty response received")
        return False

    if len(response.strip()) < min_length:
        logger.warning(f"Response too short: {len(response)} chars (min: {min_length})")
        return False

    return True


def with_retry(max_attempts: int = 3):
    """Decorator that adds retry logic with exponential backoff."""
    return retry(
        stop=stop_after_attempt(max_attempts),
        wait=wait_exponential(multiplier=1, min=2, max=30),
        reraise=True
    )
```

### Step 4.2: Update the Orchestrator with Resilience

Add the following method to `SupervisorOrchestrator` in `orchestrator.py`:

```python
from resilience import CircuitBreaker, validate_response

# In __init__, add:
self.circuit_breaker = CircuitBreaker(failure_threshold=5, reset_timeout=60)

# Replace dispatch() with:
def dispatch(self, role: str, task: str) -> str:
    """Send a task to an agent with error handling."""
    agent = self.agents[role]
    
    # Check circuit breaker
    if self.circuit_breaker.is_open(role):
        raise RuntimeError(f"Circuit breaker open for agent '{role}'")

    try:
        # Create thread and send task
        thread = client.agents.threads.create()
        client.agents.messages.create(
            thread_id=thread.id, role="user", content=task
        )

        # Run with monitoring
        run = client.agents.runs.create_and_process(
            thread_id=thread.id, agent_id=agent.id
        )

        if run.status != "completed":
            raise RuntimeError(f"Agent run status: {run.status}")

        # Extract and validate response
        messages = client.agents.messages.list(thread_id=thread.id)
        response = messages.data[0].content[0].text.value

        if not validate_response(response):
            raise ValueError(f"Agent '{role}' response failed validation")

        self.circuit_breaker.record_success(role)
        return response

    except Exception as e:
        self.circuit_breaker.record_failure(role)
        logger.error(f"Agent '{role}' failed: {e}")
        raise
```

### Step 4.3: Test Error Handling

Re-run the pipeline to verify error handling works:

```bash
python main.py
```

---

## Exercise 5: Observability & Analysis (10 minutes)

### Step 5.1: Review the Output

Open `output.md` and verify:

- [ ] The article has a TL;DR summary
- [ ] Content is well-structured with headings
- [ ] Claims are supported by research findings
- [ ] Writing is clear and accessible

### Step 5.2: Analyze the Workflow Logs

Review the agent activity log printed by the pipeline:

- [ ] Each phase (research, writing, review) is logged
- [ ] Revision count is tracked
- [ ] Agent IDs are recorded for each action
- [ ] Timestamps show reasonable processing times

### Step 5.3: Experiment with Different Topics

Try running the pipeline with different topics:

```python
topics = [
    "The future of quantum computing in cybersecurity",
    "How edge AI is transforming healthcare diagnostics",
    "Sustainable AI: reducing the carbon footprint of LLMs"
]
```

---

## Cleanup

After completing all exercises, ensure all agents are deleted:

```python
from agents import client

# List all agents and delete any leftover ones
agents = client.agents.list_agents()
for agent in agents.data:
    if agent.name in ["ResearchAgent", "WriterAgent", "ReviewerAgent"]:
        client.agents.delete_agent(agent.id)
        print(f"🗑️  Deleted: {agent.name} ({agent.id})")
```

---

## Success Criteria

| Criterion | Status |
|---|---|
| Three agents created with correct system prompts | ⬜ |
| Supervisor orchestrator dispatches tasks sequentially | ⬜ |
| Research agent produces structured findings | ⬜ |
| Writer agent creates well-formatted article | ⬜ |
| Reviewer agent provides scored feedback | ⬜ |
| At least one revision cycle executes | ⬜ |
| Pipeline completes within 3 revision cycles | ⬜ |
| Error handling (circuit breaker, validation) is implemented | ⬜ |
| All agents cleaned up after the run | ⬜ |
| Output saved to `output.md` | ⬜ |

---

## Bonus Challenges

### Challenge 1: Add a Parallel Research Phase
Create two researcher agents — one for breadth (overview) and one for depth (detailed analysis) — and run them concurrently using `asyncio.gather()`.

### Challenge 2: Implement the Router Pattern
Add a router agent at the entry point that classifies topics into categories (technology, science, business) and routes to specialized writer agents.

### Challenge 3: Add Application Insights Tracing
Integrate `azure-monitor-opentelemetry` to trace the full workflow with distributed tracing spans.

### Challenge 4: Convert to Swarm Pattern
Remove the supervisor and let agents hand off to each other by including `[HANDOFF:agent_name]` signals in their responses.

---

## Troubleshooting

| Issue | Solution |
|---|---|
| `AuthenticationError` | Run `az login` and ensure correct subscription is selected |
| `ResourceNotFoundError` | Verify `PROJECT_ENDPOINT` is correct and project exists |
| `ModelNotFoundError` | Ensure GPT-4o is deployed in your AI Foundry project |
| Agent run stuck in `queued` | Check model deployment quota and scaling settings |
| Reviewer never approves | Increase reviewer leniency in system prompt or adjust scoring criteria |
| `RateLimitError` | Add longer delays between dispatches or increase quota |
