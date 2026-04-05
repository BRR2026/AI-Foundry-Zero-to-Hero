# Module 36: Multi-Agent Systems & Orchestration

| Field | Details |
|---|---|
| **Arc** | 8 — Advanced Patterns & Enterprise |
| **Module** | 36 of 45 |
| **Title** | Multi-Agent Systems & Orchestration |
| **Duration** | 4 hours (lecture + lab) |
| **Difficulty** | Advanced |
| **Date** | April 2026 |
| **Prerequisites** | Module 33 (Advanced Agent Architectures), Module 35 (Agent Memory & State) |

---

## Learning Objectives

By the end of this module, learners will be able to:

1. ✅ Design multi-agent system architectures with clearly defined agent roles and responsibilities
2. ✅ Implement three core orchestration patterns: Supervisor, Router, and Swarm
3. ✅ Build agent-to-agent communication using message passing, shared state, and handoff protocols
4. ✅ Apply error handling strategies including timeouts, circuit breakers, and graceful degradation
5. ✅ Construct a production-ready 3-agent pipeline (Researcher → Writer → Reviewer)
6. ✅ Evaluate when to use multi-agent vs. single-agent approaches
7. ✅ Monitor and debug multi-agent workflows using structured logging and tracing

---

## Module Overview

### Why Multi-Agent Systems?

Single-agent architectures excel for well-defined tasks but encounter limitations when workflows require diverse expertise, parallel execution, iterative quality assurance, or modular maintenance. Multi-agent systems address these challenges by decomposing complex tasks across specialized agents, each with its own system prompt, tools, and model configuration.

This module covers the theory, patterns, and hands-on implementation of multi-agent orchestration in Azure AI Foundry.

### Topics Covered

1. **Designing Multi-Agent Architectures** — Agent role definition, topology selection, and system design
2. **Agent-to-Agent Communication** — Message passing, shared state, and handoff protocols
3. **Supervisor Pattern** — Central orchestrator that delegates, evaluates, and routes
4. **Router Pattern** — Intent-based classification and single-step dispatch
5. **Swarm Pattern** — Decentralized peer-to-peer collaboration with autonomous handoffs
6. **Error Handling** — Timeouts, retries, circuit breakers, cascade failure prevention
7. **Mini Hack** — Build a complete 3-agent content creation pipeline

---

## Section 1: Designing Multi-Agent Architectures

### When to Go Multi-Agent

| Criterion | Single Agent | Multi-Agent |
|---|---|---|
| Task complexity | Low-medium, well-defined | High, multi-faceted |
| Domain breadth | Single domain | Cross-domain expertise needed |
| Quality requirements | Self-review acceptable | Independent review required |
| Latency tolerance | Sequential OK | Parallel execution beneficial |
| Maintenance model | Monolithic prompts acceptable | Modular agent updates needed |
| Cost sensitivity | One model for all | Right-size models per role |

### Agent Role Design Principles

1. **Single Responsibility**: Each agent should do ONE thing well. A researcher researches; it doesn't write or review.
2. **Clear Contracts**: Define the expected input format and output format for each agent explicitly in its system prompt.
3. **Minimal Coupling**: Agents should communicate through well-defined interfaces, not by sharing internal state.
4. **Fail-Safe Defaults**: Every agent should have a graceful failure mode that doesn't break the overall workflow.

### Topology Selection Guide

| Pattern | Best For | Complexity | Control |
|---|---|---|---|
| **Supervisor** | Structured multi-step workflows | Medium | Centralized |
| **Router** | Intent-based request classification | Low | Centralized |
| **Swarm** | Dynamic, emergent collaboration | High | Decentralized |
| **Pipeline** | Sequential transformation chains | Low | Linear |
| **Hierarchy** | Multi-level delegation trees | High | Hierarchical |

---

## Section 2: Agent-to-Agent Communication Patterns

### Pattern 1: Message Passing

Agents communicate by sending structured messages through direct invocation or a message bus. Each message includes:

- **Sender**: The agent ID of the message originator
- **Recipient**: The target agent ID (or `"broadcast"` for all agents)
- **Content**: The actual payload (task description, results, feedback)
- **Type**: Classification of the message (`"task"`, `"result"`, `"feedback"`, `"handoff"`)
- **Metadata**: Optional context (priority, constraints, timestamps)

**Advantages**: Loose coupling, easy to log and replay, supports async workflows  
**Disadvantages**: Requires serialization logic, no shared context without explicit passing

### Pattern 2: Shared State

Multiple agents read and write to a shared workspace (dictionary, database, or state store). This is ideal for iterative refinement cycles where each agent builds on the previous agent's work.

**Advantages**: Rich context available to all agents, natural for iterative workflows  
**Disadvantages**: Concurrency risks, harder to trace causality, tight coupling to state schema

### Pattern 3: Handoff Protocol

Formal transfer of control from one agent to another, including accumulated context, transfer reason, and constraints. The handoff model is particularly well-suited for swarm architectures.

**Advantages**: Explicit control flow, carries context naturally, supports autonomous decisions  
**Disadvantages**: Requires handoff parsing logic, agents must know their peers

---

## Section 3: The Supervisor Pattern

### Architecture

```
User → Supervisor → [Researcher] → Supervisor → [Writer] → Supervisor → [Reviewer] → Output
                         ↑                                        |
                         └──── Revision Loop (if not approved) ───┘
```

### Key Components

1. **Supervisor Agent**: Routes tasks, evaluates outputs, and decides next steps. Uses a fast model (e.g., GPT-4o-mini) since its job is routing, not generation.
2. **Worker Agents**: Domain specialists that execute specific tasks. Each has a focused system prompt and relevant tools.
3. **Workflow State**: Tracks progress, stores intermediate results, and logs agent activity.
4. **Dispatch Logic**: Sends tasks to workers and collects responses.
5. **Decision Logic**: Evaluates worker outputs and determines whether to proceed, retry, or revise.

### Best Practices

- Set a **maximum revision count** (e.g., 3) to prevent infinite revision loops
- Use the supervisor for **routing decisions only** — never give it domain tasks
- Log every dispatch and response for **debugging and auditing**
- Implement **timeouts per dispatch** to prevent any single agent from blocking the pipeline
- Consider **parallel dispatch** when worker tasks are independent (e.g., researching multiple subtopics)

---

## Section 4: The Router Pattern

### Architecture

```
User → Router (classifier) → Agent A (billing)
                            → Agent B (tech support)
                            → Agent C (general FAQ)
```

### When to Use

- Incoming requests span multiple distinct domains
- Each request maps to a single specialist
- No multi-step coordination is needed
- You want fast, single-hop routing

### Implementation Notes

- The router agent should use a **small, fast model** (GPT-4o-mini) for classification
- Keep the classification prompt **constrained** — list all categories explicitly
- Always include a **fallback route** for unclassified requests
- Consider a **confidence threshold** — if the router is uncertain, escalate to a human or general agent

---

## Section 5: The Swarm Pattern

### Architecture

```
Agent A ⇄ Agent B ⇄ Agent C
    ↕                  ↕
Agent D ⇄ Agent E
```

### Key Characteristics

- **No central controller**: Each agent decides independently when to hand off
- **Handoff signals**: Agents embed handoff instructions in their responses (e.g., `[HANDOFF:reviewer]`)
- **Peer awareness**: Each agent knows which other agents exist and what they do
- **Emergent workflows**: The actual execution path emerges from agent decisions, not from a predefined plan

### Safety Mechanisms

1. **Max handoffs limit**: Prevents infinite loops (recommended: 10-15 per workflow)
2. **Handoff validation**: Verify the target agent exists before executing a handoff
3. **Cycle detection**: Track the handoff chain and alert if an agent is revisited more than twice
4. **Timeout**: Global workflow timeout regardless of handoff count

---

## Section 6: Error Handling in Multi-Agent Workflows

### Failure Taxonomy

| Category | Examples | Impact |
|---|---|---|
| **Infrastructure** | API rate limits, network timeouts, service outages | Agent cannot execute |
| **Quality** | Off-topic response, hallucinated data, incomplete output | Downstream agents receive bad input |
| **Coordination** | Invalid handoff target, state corruption, deadlock | Workflow breaks down |
| **Resource** | Token limit exceeded, budget exceeded | Partial execution |

### Defense Strategies

1. **Per-Agent Timeouts**: Set a maximum execution time for each agent dispatch (recommended: 60-120 seconds)
2. **Retry with Backoff**: Use exponential backoff with jitter for transient failures (recommended: 3 retries, 1s base delay)
3. **Circuit Breakers**: After N consecutive failures, stop dispatching to that agent and use a fallback
4. **Quality Validation**: Check response length, format compliance, and relevance before passing to the next agent
5. **Graceful Degradation**: If a specialist fails, fall back to a general-purpose agent
6. **Immutable State Snapshots**: Take checkpoints of the workflow state so you can recover from corruption
7. **Dead Letter Queue**: Store failed messages for later retry or manual inspection

### Observability

- **Structured logging**: Log every agent dispatch, response, and error with correlation IDs
- **Distributed tracing**: Use Application Insights or OpenTelemetry to trace the full workflow
- **Metrics**: Track per-agent latency, error rate, retry count, and revision count
- **Alerts**: Set up alerts for circuit breaker trips, timeout spikes, and budget overruns

---

## Section 7: Advanced Patterns

### Parallel Fan-Out / Fan-In

Dispatch the same task to multiple agents simultaneously and aggregate their results:

```python
import asyncio

async def parallel_research(topics: list[str]) -> list[str]:
    tasks = [dispatch_to_researcher(topic) for topic in topics]
    results = await asyncio.gather(*tasks, return_exceptions=True)
    return [r for r in results if not isinstance(r, Exception)]
```

### Hierarchical Supervision

For complex workflows, use a hierarchy of supervisors:

```
Chief Supervisor
├── Research Supervisor → [Researcher A, Researcher B]
├── Content Supervisor → [Writer, Editor]
└── QA Supervisor → [Reviewer, Fact-Checker]
```

### Dynamic Agent Creation

Create agents on-the-fly based on the task requirements:

```python
def create_specialist(client, topic: str) -> Agent:
    return client.agents.create_agent(
        model="gpt-4o",
        name=f"Specialist_{topic}",
        instructions=f"You are an expert in {topic}. Provide detailed analysis."
    )
```

---

## Recommended Video

**Multi-agent Orchestration in Microsoft Copilot Studio**  
🎥 [https://www.youtube.com/watch?v=WKKdBC2zM3k](https://www.youtube.com/watch?v=WKKdBC2zM3k)

This video covers real-world multi-agent orchestration patterns implemented within Microsoft Copilot Studio, demonstrating how to coordinate multiple AI agents for enterprise workflows.

---

## Summary

| Concept | Key Point |
|---|---|
| **Multi-agent systems** | Decompose complex tasks across specialized agents |
| **Supervisor pattern** | Central orchestrator delegates and evaluates |
| **Router pattern** | Intent classifier dispatches to specialists |
| **Swarm pattern** | Peer-to-peer handoffs without central control |
| **Communication** | Message passing, shared state, or handoff protocol |
| **Error handling** | Timeouts, retries, circuit breakers, validation |
| **Agent design** | Single responsibility, clear contracts, minimal coupling |

---

## Next Steps

- **Module 37**: Enterprise Guardrails & Safety — implementing safety layers for production AI agents
- **Module 38**: Performance Optimization & Scaling — optimizing multi-agent throughput and cost
- **Lab**: Complete the hands-on lab to build a full 3-agent system
- **Assessment**: Take the Module 36 quiz to validate your understanding
