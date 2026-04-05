# Module 43 Assessment: Deploy & Integrate with Agents/Copilot

## Azure AI Foundry: Zero to Hero — Knowledge Check

**ARC 9: Capstone & Mastery** · 10 Questions · Passing Score: 80% (8/10)

---

## Instructions

- Select the **single best answer** for each question.
- Each question is worth 1 point.
- Time limit: 20 minutes.
- Passing score: 8 out of 10 correct.

---

### Question 1

**What is the primary difference between an AI agent and a standard chat completion call in Azure AI Foundry?**

- A) Agents use a different LLM model than chat completions
- B) Agents maintain state across turns, plan multi-step reasoning, and autonomously invoke tools
- C) Agents can only answer questions from a knowledge base
- D) Chat completions are more accurate than agents for all use cases

<details>
<summary>✅ Answer</summary>

**B) Agents maintain state across turns, plan multi-step reasoning, and autonomously invoke tools**

Agents go beyond simple request/response by maintaining conversational context through threads, planning multi-step task execution, and autonomously selecting and invoking tools (RAG search, functions, code interpreter) based on user intent. Chat completions are stateless single-turn interactions.

</details>

---

### Question 2

**When creating an agent with Azure AI Foundry that uses your RAG pipeline, which SDK class configures the connection to your Azure AI Search index?**

- A) `SearchIndexClient`
- B) `AzureAISearchTool`
- C) `CognitiveSearchConnector`
- D) `RAGSearchProvider`

<details>
<summary>✅ Answer</summary>

**B) AzureAISearchTool**

The `AzureAISearchTool` class from the `azure-ai-projects` SDK is used to configure the connection to your Azure AI Search index, including the index name, connection ID, query type (keyword, semantic, hybrid), and top-k retrieval settings.

</details>

---

### Question 3

**What is the recommended first step when integrating your Azure AI Foundry agent with Microsoft Copilot Studio?**

- A) Publish the agent directly to Teams
- B) Create a Power Automate flow
- C) Expose your agent as a REST API endpoint (e.g., FastAPI or Azure Functions)
- D) Import the agent SDK into Copilot Studio

<details>
<summary>✅ Answer</summary>

**C) Expose your agent as a REST API endpoint (e.g., FastAPI or Azure Functions)**

Before connecting to Copilot Studio, you must expose your agent as a REST API that accepts conversation payloads and returns structured responses. Copilot Studio then connects to this API through a custom connector.

</details>

---

### Question 4

**Which Teams integration approach has the LOWEST complexity and is best suited for business users who need quick deployment?**

- A) Teams AI Library
- B) Bot Framework SDK
- C) Copilot Studio → Teams publish
- D) Message Extension with Teams Toolkit

<details>
<summary>✅ Answer</summary>

**C) Copilot Studio → Teams publish**

Publishing from Copilot Studio to Teams is the lowest-complexity approach. It requires no custom code for the Teams integration itself — you configure the channel in Copilot Studio and publish directly. Business users can manage the bot without developer involvement.

</details>

---

### Question 5

**When deploying your agent API for external consumers, which Azure service provides rate limiting, JWT validation, API versioning, and usage analytics as a gateway?**

- A) Azure Application Gateway
- B) Azure Front Door
- C) Azure API Management (APIM)
- D) Azure Load Balancer

<details>
<summary>✅ Answer</summary>

**C) Azure API Management (APIM)**

Azure API Management is a fully managed API gateway that provides rate limiting policies, JWT token validation with Entra ID, API versioning, subscription key management, usage analytics, and developer portal capabilities — all essential for exposing agent APIs to external consumers.

</details>

---

### Question 6

**In the testing pyramid for AI agents, which level of testing validates complete conversation flows through deployed endpoints with real data?**

- A) Unit tests
- B) Integration tests
- C) End-to-end (E2E) tests
- D) Static analysis

<details>
<summary>✅ Answer</summary>

**C) End-to-end (E2E) tests**

E2E tests validate the entire system by sending HTTP requests to deployed endpoints and verifying that the full pipeline — from API routing through agent reasoning, tool invocation, RAG retrieval, and response generation — works correctly as an integrated system.

</details>

---

### Question 7

**Which Azure AI Evaluation SDK metric measures how well an agent's response is supported by the source data retrieved from the knowledge base?**

- A) Fluency
- B) Coherence
- C) Relevance
- D) Groundedness

<details>
<summary>✅ Answer</summary>

**D) Groundedness**

Groundedness evaluates whether the agent's response is factually supported by the context/source data retrieved from the knowledge base. A high groundedness score means the agent is not hallucinating and is basing its answers on actual retrieved information.

</details>

---

### Question 8

**When deploying an AI agent to Microsoft Teams, which authentication method should you use for secure access?**

- A) Basic authentication with username/password
- B) API key embedded in the client app
- C) Entra ID Single Sign-On (SSO)
- D) No authentication (public access)

<details>
<summary>✅ Answer</summary>

**C) Entra ID Single Sign-On (SSO)**

Microsoft Teams provides a seamless SSO experience through Entra ID (formerly Azure AD). The Teams platform handles token acquisition, and your agent API validates the token. This is the recommended approach — never expose unauthenticated agent endpoints.

</details>

---

### Question 9

**You want your agent to create support tickets and check order status in addition to answering questions from your knowledge base. Which component do you use to register these custom actions?**

- A) `AzureAISearchTool` with custom schemas
- B) `FunctionTool` with Python function definitions
- C) `CodeInterpreterTool` with inline scripts
- D) `WebhookTool` with HTTP callbacks

<details>
<summary>✅ Answer</summary>

**B) FunctionTool with Python function definitions**

The `FunctionTool` class allows you to register Python functions as callable tools for the agent. The agent uses the function name, docstring, and parameter type hints to decide when and how to invoke each function. Functions are composed into a `ToolSet` alongside other tools like the RAG search tool.

</details>

---

### Question 10

**Your E2E test for multi-turn conversation verifies that the agent maintains context across turns. Which of the following is the correct approach?**

- A) Send both messages in a single API call
- B) Send the first message, capture the `session_id` from the response, and include it in the second request
- C) Use a different API endpoint for follow-up messages
- D) Include the full conversation history in each request body

<details>
<summary>✅ Answer</summary>

**B) Send the first message, capture the `session_id` from the response, and include it in the second request**

The agent API returns a `session_id` (mapped to an underlying thread) with each response. To continue a conversation, the client includes this `session_id` in subsequent requests. The agent service uses it to retrieve the existing thread and maintain conversation context.

</details>

---

## 📊 Scoring Guide

| Score | Grade | Status |
|---|---|---|
| 10/10 | A+ | ⭐ Excellent — Mastery demonstrated |
| 9/10 | A | ✅ Excellent |
| 8/10 | B+ | ✅ Pass — Meets expectations |
| 7/10 | B | ⚠️ Below passing — Review weak areas |
| 6/10 | C | ❌ Needs improvement — Revisit module |
| ≤5/10 | F | ❌ Significant gaps — Re-study required |

---

## 📝 Answer Key (Quick Reference)

| Q | Answer | Topic |
|---|---|---|
| 1 | B | Agent vs. Chat Completions |
| 2 | B | AzureAISearchTool SDK |
| 3 | C | Copilot Studio integration |
| 4 | C | Teams deployment |
| 5 | C | API Management |
| 6 | C | E2E Testing |
| 7 | D | Groundedness evaluation |
| 8 | C | Entra ID SSO |
| 9 | B | FunctionTool |
| 10 | B | Session continuity |

---

*Module 43 of 45 — Azure AI Foundry: Zero to Hero*
*ARC 9: Capstone & Mastery*
*© 2026 Azure AI Foundry Training Series*
