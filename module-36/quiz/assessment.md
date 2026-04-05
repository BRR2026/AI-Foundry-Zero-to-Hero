# Module 36 Assessment: Multi-Agent Systems & Orchestration

| Field | Details |
|---|---|
| **Module** | 36 of 45 |
| **Arc** | 8 — Advanced Patterns & Enterprise |
| **Total Questions** | 10 Multiple Choice |
| **Passing Score** | 80% (8 out of 10) |
| **Time Limit** | 20 minutes |

---

## Section A: Multiple Choice Questions

Select the **best** answer for each question.

---

### Question 1 — Orchestration Pattern Selection
**Difficulty: Medium**

A customer support application receives requests about billing, technical issues, and general inquiries. Each request type should be handled by a dedicated specialist agent. No multi-step coordination is needed. Which orchestration pattern is most appropriate?

- A) Supervisor pattern — a central orchestrator delegates to specialists sequentially
- B) Router pattern — a classifier dispatches each request to the best-fit specialist
- C) Swarm pattern — agents communicate peer-to-peer and decide handoffs autonomously
- D) Pipeline pattern — requests flow through all agents in a fixed sequence

<details>
<summary>Answer</summary>

**B) Router pattern**

The Router pattern is designed for intent-based classification and single-hop dispatch. Since each request maps to exactly one specialist agent and no multi-step coordination is required, the Router is the simplest and most efficient choice. The Supervisor pattern would add unnecessary orchestration overhead, the Swarm pattern introduces complexity without benefit for single-hop routing, and the Pipeline pattern would process through all agents unnecessarily.
</details>

---

### Question 2 — Supervisor Pattern Characteristics
**Difficulty: Easy**

In the Supervisor orchestration pattern, which component is responsible for evaluating agent outputs and deciding whether to proceed, retry, or revise?

- A) The worker agents evaluate their own outputs
- B) The supervisor agent makes routing and evaluation decisions
- C) A separate quality assurance service external to the agent system
- D) The user manually reviews each output before proceeding

<details>
<summary>Answer</summary>

**B) The supervisor agent makes routing and evaluation decisions**

The defining characteristic of the Supervisor pattern is that a central orchestrator (the supervisor) delegates tasks, collects results, evaluates outputs, and decides next steps. Worker agents focus only on their specialized tasks; they don't self-evaluate for workflow routing purposes.
</details>

---

### Question 3 — Communication Pattern Tradeoffs
**Difficulty: Medium**

A multi-agent system uses a shared state store where all agents read and write to the same workspace. What is the primary risk of this communication pattern?

- A) Messages may be lost in transit between agents
- B) Agents cannot access context from previous agents' work
- C) Concurrency risks and difficulty tracing which agent caused state changes
- D) The system requires a message broker service to function

<details>
<summary>Answer</summary>

**C) Concurrency risks and difficulty tracing which agent caused state changes**

Shared state provides rich context but introduces concurrency risks when multiple agents read and write simultaneously. It also makes it harder to trace causality — understanding which agent made what change and when. This is why structured logging of state mutations (with agent IDs and timestamps) is essential. Message passing (A, D) is a different pattern; the shared state pattern explicitly provides access to previous work (B).
</details>

---

### Question 4 — Swarm Pattern Safety
**Difficulty: Medium**

In a Swarm orchestration pattern, agents make autonomous handoff decisions without a central controller. Which safety mechanism is MOST critical to prevent infinite loops?

- A) Using the largest available model for all agents
- B) Setting a maximum handoff count limit per workflow
- C) Logging all messages to Application Insights
- D) Deploying agents across different Azure regions

<details>
<summary>Answer</summary>

**B) Setting a maximum handoff count limit per workflow**

The most critical safety mechanism in a Swarm pattern is a maximum handoff count (recommended: 10-15 per workflow). Without this limit, agents can enter infinite handoff cycles, passing tasks back and forth indefinitely. While logging (C) is important for observability, it doesn't prevent the problem. Model size (A) and regional deployment (D) don't address handoff loops.
</details>

---

### Question 5 — Error Handling Strategy
**Difficulty: Hard**

An orchestrator dispatches a task to the Researcher agent, but the agent fails 5 consecutive times due to API rate limits. On the 6th request, the orchestrator should:

- A) Retry immediately with the same agent — transient errors resolve themselves
- B) Stop the entire workflow and return an error to the user
- C) Trip the circuit breaker and route to a fallback agent or gracefully degrade
- D) Increase the timeout to 10 minutes and try one more time

<details>
<summary>Answer</summary>

**C) Trip the circuit breaker and route to a fallback agent or gracefully degrade**

After 5 consecutive failures, the circuit breaker should open, preventing further requests to the failing agent. The orchestrator should then either route to a fallback agent or gracefully degrade (e.g., proceed with partial results). Immediately retrying (A) wastes resources and may worsen rate limiting. Stopping entirely (B) is unnecessarily harsh. Increasing the timeout (D) doesn't address rate limiting issues.
</details>

---

### Question 6 — Agent Design Principles
**Difficulty: Easy**

According to multi-agent design best practices, what model configuration strategy should you use for the supervisor agent vs. the worker agents?

- A) Use the same large model for all agents to ensure consistent quality
- B) Use a smaller, faster model for the supervisor and larger models for workers that generate content
- C) Use the largest model for the supervisor since it makes the most important decisions
- D) Use no model for the supervisor — implement routing with deterministic code only

<details>
<summary>Answer</summary>

**B) Use a smaller, faster model for the supervisor and larger models for workers that generate content**

The supervisor primarily makes routing and evaluation decisions, which don't require the full power of a large model. Using a smaller, faster model (e.g., GPT-4o-mini) for the supervisor reduces cost and latency. Worker agents that perform substantive tasks like research, writing, and reviewing benefit from larger models. This "right-sizing" approach optimizes both cost and performance.
</details>

---

### Question 7 — Handoff Protocol
**Difficulty: Medium**

In a well-designed agent handoff protocol, which pieces of information should be included in the handoff message? (Select the most complete answer)

- A) Only the task description for the next agent
- B) The accumulated context, reason for handoff, and any constraints for the receiving agent
- C) The full conversation history from all previous agents
- D) Only the name of the next agent to handle the task

<details>
<summary>Answer</summary>

**B) The accumulated context, reason for handoff, and any constraints for the receiving agent**

A well-designed handoff includes: the accumulated context (relevant state and findings), the reason for the transfer (why this agent is being called), and any constraints the receiving agent should respect (output format, length limits, etc.). Sending only the task (A) or name (D) lacks necessary context. Sending the full conversation history (C) is wasteful and may exceed token limits — the handoff should carry curated, relevant context.
</details>

---

### Question 8 — Multi-Agent Anti-Patterns
**Difficulty: Hard**

Which of the following is an anti-pattern in multi-agent system design?

- A) Setting per-agent timeouts to prevent blocking the workflow
- B) Giving the supervisor agent domain-specific tasks in addition to routing responsibilities
- C) Implementing circuit breakers that trip after consecutive agent failures
- D) Logging every dispatch and response with correlation IDs for tracing

<details>
<summary>Answer</summary>

**B) Giving the supervisor agent domain-specific tasks in addition to routing responsibilities**

The supervisor should focus exclusively on orchestration: routing tasks, evaluating outputs, and deciding next steps. Adding domain tasks (like research or writing) to the supervisor violates the single-responsibility principle, makes the supervisor's prompt complex and fragile, and defeats the purpose of having specialized worker agents. Options A, C, and D are all recommended best practices.
</details>

---

### Question 9 — When NOT to Use Multi-Agent
**Difficulty: Medium**

In which scenario would a single-agent architecture be MORE appropriate than a multi-agent system?

- A) An enterprise content pipeline requiring research, drafting, and editorial review
- B) A simple FAQ chatbot that answers questions from a predefined knowledge base
- C) A customer support system handling billing, technical, and general inquiries
- D) A code review system that analyzes code for bugs, security issues, and style violations

<details>
<summary>Answer</summary>

**B) A simple FAQ chatbot that answers questions from a predefined knowledge base**

A FAQ chatbot with a predefined knowledge base is a well-defined, single-domain task that doesn't benefit from specialization, parallelism, or quality review loops. A single agent with RAG (Retrieval-Augmented Generation) is simpler, cheaper, and faster to maintain. The other scenarios involve multiple distinct subtasks or domains that benefit from multi-agent decomposition.
</details>

---

### Question 10 — Production Observability
**Difficulty: Hard**

You're running a multi-agent system in production with a Supervisor pattern. The system occasionally produces low-quality articles, but you can't reproduce the issue in testing. Which observability strategy would be MOST effective for diagnosing the root cause?

- A) Increase the model temperature for all agents to generate more diverse outputs
- B) Add structured logging with correlation IDs that trace the full workflow chain, including each agent's input, output, and timing
- C) Switch to a Swarm pattern so agents can self-correct
- D) Deploy the system in a different Azure region for better performance

<details>
<summary>Answer</summary>

**B) Add structured logging with correlation IDs that trace the full workflow chain, including each agent's input, output, and timing**

Structured logging with correlation IDs allows you to trace the complete path of any workflow — seeing exactly what input each agent received, what it produced, how long it took, and where quality degraded. This is essential for diagnosing intermittent issues. Increasing temperature (A) would likely make quality worse. Switching patterns (C) doesn't address the observability gap. Regional deployment (D) addresses latency, not quality issues.
</details>

---

## Scoring Guide

| Score | Grade | Recommendation |
|---|---|---|
| 10/10 | ⭐ Excellent | Ready for Module 37 |
| 8-9/10 | ✅ Pass | Proceed to Module 37, review missed topics |
| 6-7/10 | ⚠️ Needs Review | Re-read the training guide and redo the lab |
| 0-5/10 | ❌ Fail | Review Modules 33-36 before proceeding |

---

## Answer Key (Quick Reference)

| Question | Answer | Topic |
|---|---|---|
| 1 | B | Pattern selection (Router) |
| 2 | B | Supervisor characteristics |
| 3 | C | Shared state tradeoffs |
| 4 | B | Swarm safety (max handoffs) |
| 5 | C | Circuit breaker pattern |
| 6 | B | Model right-sizing |
| 7 | B | Handoff protocol design |
| 8 | B | Anti-patterns (supervisor overload) |
| 9 | B | When to use single-agent |
| 10 | B | Production observability |
