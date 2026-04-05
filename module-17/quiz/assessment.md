# Module 17 Assessment: Building Your First Agent

## Azure AI Foundry: Zero to Hero | Arc 4 — AI Agents

| Field              | Details                       |
| ------------------ | ----------------------------- |
| **Questions**      | 10 multiple choice            |
| **Passing Score**  | 80% (8 out of 10)            |
| **Time Limit**     | 15 minutes                    |
| **Module**         | 17 — Building Your First Agent|

---

## Instructions

Select the **one best answer** for each question. Answers are provided at the end of the document.

---

### Question 1

**Which of the following are the four core primitives of the Azure AI Foundry Agent Service?**

- A) Model, Prompt, Completion, Token
- B) Agent, Thread, Message, Run
- C) Pipeline, Node, Connection, Flow
- D) Endpoint, Deployment, Session, Response

---

### Question 2

**What is the primary purpose of agent instructions (system prompt)?**

- A) To define the Azure subscription and resource group
- B) To configure network security and access policies
- C) To define the agent's role, capabilities, behavior, and constraints
- D) To specify the SDK version and authentication method

---

### Question 3

**Which built-in tool allows an agent to execute Python code in a sandboxed environment?**

- A) File Search
- B) Bing Grounding
- C) Code Interpreter
- D) Azure Functions

---

### Question 4

**When using File Search, documents are stored in which Azure AI Foundry construct?**

- A) Blob container
- B) Vector store
- C) SQL database
- D) Thread attachment

---

### Question 5

**What additional Azure resource is required to use the Bing Grounding tool with an agent?**

- A) Azure Cognitive Search
- B) Azure Data Factory
- C) Grounding with Bing Search resource
- D) Azure Bot Service

---

### Question 6

**In the Python SDK, which class is used as the main client for creating and managing agents?**

- A) `AzureOpenAI`
- B) `AIProjectClient`
- C) `AgentServiceClient`
- D) `FoundryClient`

---

### Question 7

**Which of the following is the correct order of the agent execution lifecycle?**

- A) Create Run → Create Thread → Create Agent → Add Message → Read Response
- B) Create Agent → Create Thread → Add Message → Create Run → Read Response
- C) Create Thread → Create Agent → Create Run → Add Message → Read Response
- D) Add Message → Create Agent → Create Thread → Read Response → Create Run

---

### Question 8

**In the Agent Playground, what feature should you enable to see which tools the agent invokes during a conversation?**

- A) Debug mode
- B) Verbose logging
- C) Show tool calls
- D) Trace diagnostics

---

### Question 9

**Which authentication method is recommended for production agent deployments?**

- A) API keys stored in environment variables
- B) Shared Access Signatures (SAS tokens)
- C) Managed Identity via `DefaultAzureCredential`
- D) Username and password authentication

---

### Question 10

**You are building a data analyst agent. You want it to analyze CSV files AND answer questions about product documentation. Which combination of tools should you enable?**

- A) Code Interpreter + Bing Grounding
- B) File Search + Bing Grounding
- C) Code Interpreter + File Search
- D) Code Interpreter only

---

## Answer Key

| Question | Answer | Explanation |
| -------- | ------ | ----------- |
| 1        | **B**  | The four core primitives are Agent (entity with instructions/tools), Thread (conversation session), Message (unit of communication), and Run (execution of the agent on a thread). |
| 2        | **C**  | Instructions define the agent's persona, what it can do, how it should behave, and what it should not do. They are the system prompt that governs all agent behavior. |
| 3        | **C**  | Code Interpreter provides a sandboxed Python environment where agents can write and execute code for data analysis, calculations, and visualization generation. |
| 4        | **B**  | Documents uploaded for File Search are chunked, embedded, and stored in a vector store, which enables semantic search and retrieval. |
| 5        | **C**  | Bing Grounding requires a "Grounding with Bing Search" resource to be created in the Azure portal and connected to the AI Foundry project. |
| 6        | **B**  | `AIProjectClient` from the `azure-ai-projects` package is the main client for interacting with the Azure AI Foundry Agent Service in Python. |
| 7        | **B**  | The correct lifecycle is: Create Agent → Create Thread → Add Message → Create Run (or Stream) → Read Response. The agent must exist before creating a thread. |
| 8        | **C**  | The "Show tool calls" toggle in the Playground reveals which tools the agent invokes, showing code execution, file searches, and web queries in real time. |
| 9        | **C**  | Managed Identity via `DefaultAzureCredential` is the recommended authentication for production, avoiding the need to manage and rotate API keys. |
| 10       | **C**  | Code Interpreter handles CSV data analysis, while File Search provides RAG over the product documentation. Together they allow the agent to analyze data and reference docs. |

---

## Scoring Guide

| Score   | Result       | Recommendation                                    |
| ------- | ------------ | -------------------------------------------------- |
| 10/10   | ⭐ Excellent | Ready for Module 18: Custom Function Tools         |
| 8-9/10  | ✅ Pass      | Review any missed topics, then proceed             |
| 6-7/10  | ⚠️ Review   | Re-read the training guide, retry the lab          |
| 0-5/10  | ❌ Retake    | Review Module 16-17 materials before retaking      |

---

*Azure AI Foundry: Zero to Hero — Module 17 Assessment — April 2026*
