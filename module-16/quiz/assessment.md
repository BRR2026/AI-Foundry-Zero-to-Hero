# Module 16 Assessment: Intro to AI Agents & Agent Framework

**Course:** Azure AI Foundry — Zero to Hero  
**Module:** 16 — Intro to AI Agents & Agent Framework  
**Type:** Multiple Choice (10 Questions)  
**Passing Score:** 80% (8/10)  
**Time Limit:** 15 minutes

---

## Questions

### Question 1

**What is the PRIMARY characteristic that distinguishes an AI agent from a traditional chatbot?**

- A) AI agents use larger language models than chatbots
- B) AI agents can autonomously plan, use tools, and take actions toward a goal
- C) AI agents always require custom code to build
- D) AI agents can only respond in English

**✅ Correct Answer: B**

> **Explanation:** The fundamental difference is *agency* — the ability to reason, plan, select tools, and take autonomous actions toward a goal without requiring step-by-step user guidance. Chatbots are reactive and respond only to direct prompts.

---

### Question 2

**Which of the following are the four pillars of AI agent architecture in Azure AI Foundry?**

- A) Model, Dataset, Pipeline, Endpoint
- B) Instructions, Model, Tools, Memory (Threads)
- C) Prompt, Completion, Embedding, Index
- D) Input, Processing, Output, Feedback

**✅ Correct Answer: B**

> **Explanation:** Every agent in Azure AI Foundry is built on four pillars: *Instructions* (system prompt defining behavior), *Model* (the LLM reasoning engine), *Tools* (capabilities like Code Interpreter and File Search), and *Memory/Threads* (conversation state and history).

---

### Question 3

**What is a "prompt agent" in Azure AI Agent Service?**

- A) An agent that only responds to prompt-based queries without any tools
- B) A no-code agent configured via the portal or API using built-in tools and instructions
- C) An agent that automatically generates prompts for other agents
- D) An agent that requires a Dockerfile and custom Python code

**✅ Correct Answer: B**

> **Explanation:** A prompt agent is configured entirely through the portal or API — no custom code required. You define instructions, select a model, enable built-in tools (File Search, Code Interpreter, Bing), and the platform manages execution.

---

### Question 4

**Which built-in tool would an agent use to perform mathematical calculations and generate charts?**

- A) File Search
- B) Bing Grounding
- C) Code Interpreter
- D) Azure Logic Apps

**✅ Correct Answer: C**

> **Explanation:** Code Interpreter executes Python code in a sandboxed environment, enabling the agent to perform calculations, generate visualizations, process data files, and create charts — all autonomously.

---

### Question 5

**What is the role of "threads" in the Azure AI Agent Service?**

- A) They represent parallel execution paths for multi-agent systems
- B) They store the conversation history, file attachments, and tool execution results for a session
- C) They define the sequence of tools an agent must use
- D) They connect multiple agents in a pipeline

**✅ Correct Answer: B**

> **Explanation:** A thread is a conversation session that stores the complete interaction history between a user and an agent, including messages, uploaded files, and tool execution outputs. The Agent Service manages threads automatically, including token window management.

---

### Question 6

**When should you choose a hosted agent over a prompt agent?**

- A) When you want the fastest possible setup time
- B) When you only need File Search and Code Interpreter
- C) When you need custom business logic, external API integrations, or multi-agent orchestration
- D) When you want to avoid using containers

**✅ Correct Answer: C**

> **Explanation:** Hosted agents are the right choice when you need custom Python code, integration with proprietary databases or APIs, multi-agent coordination, or when you want to use frameworks like Semantic Kernel or LangChain. Prompt agents are better for rapid prototyping with built-in tools.

---

### Question 7

**Which security principle is MOST critical when deploying AI agents compared to traditional chatbots?**

- A) Using HTTPS for all connections
- B) Applying the principle of least privilege — only enabling tools and permissions the agent needs
- C) Encrypting the model weights at rest
- D) Using a larger token context window

**✅ Correct Answer: B**

> **Explanation:** Because agents can take *autonomous actions* (execute code, call APIs, search files), the principle of least privilege is critical. An agent with unnecessary tool access or permissions can cause real harm — deleting data, exposing sensitive information, or executing unintended operations.

---

### Question 8

**Which of the following is NOT a built-in tool available in the Azure AI Agent Service?**

- A) File Search
- B) Code Interpreter
- C) Bing Grounding
- D) GPU Training

**✅ Correct Answer: D**

> **Explanation:** GPU Training is not an agent tool — it's a separate concern for model fine-tuning. The built-in agent tools include File Search, Code Interpreter, Bing Grounding, Azure Functions, OpenAPI tools, and Azure Logic Apps.

---

### Question 9

**In the agent architecture, what do "instructions" define?**

- A) The API endpoints the agent can call
- B) The agent's persona, goals, behavioral rules, constraints, and tool usage guidance
- C) The training data used to fine-tune the model
- D) The deployment region and compute size

**✅ Correct Answer: B**

> **Explanation:** Instructions (the system prompt) define the agent's identity and behavior: who it is (persona), what it should accomplish (goals), how it should behave (rules), what it should avoid (constraints), and when to use which tools (tool guidance).

---

### Question 10

**What is the RECOMMENDED approach for getting started with AI agents in Azure AI Foundry?**

- A) Always start with hosted agents for maximum flexibility
- B) Start with prompt agents to validate your use case quickly, then graduate to hosted agents as complexity grows
- C) Build a multi-agent system from day one
- D) Skip agents and use Prompt Flow for all orchestration

**✅ Correct Answer: B**

> **Explanation:** The recommended approach is to start with prompt agents for rapid prototyping and validation, then evolve to hosted agents when you need custom business logic, external integrations, or multi-agent patterns. Many production systems begin as prompt agents.

---

## Scoring Guide

| Score | Rating | Action |
|-------|--------|--------|
| 10/10 | ⭐ Excellent | Ready for Module 17 |
| 8–9/10 | ✅ Pass | Review missed topics, proceed to Module 17 |
| 6–7/10 | ⚠️ Review | Re-read blog sections for missed questions |
| 0–5/10 | ❌ Retry | Review the full module before re-attempting |

---

*Module 16 — Intro to AI Agents & Agent Framework | Azure AI Foundry: Zero to Hero*
