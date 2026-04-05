# Module 24 — Assessment Quiz

## Integrating with Microsoft 365 Copilot

> **Module:** 24 of 45 · Azure AI Foundry: Zero to Hero
> **Arc:** 5 — Orchestration & Integration
> **Passing Score:** 80% (8 out of 10 correct)
> **Time Limit:** 15 minutes

---

### Question 1

**Which of the following is NOT one of the three primary M365 Copilot extensibility patterns?**

- A) Declarative Agents
- B) API Plugins
- C) Message Extensions
- D) Power Automate Flows

<details>
<summary>View Answer</summary>

**Correct Answer: D**

The three primary M365 Copilot extensibility patterns are **Declarative Agents**, **API Plugins**, and **Message Extensions**. Power Automate Flows integrate with Copilot through Copilot Studio connectors, but they are not a direct Copilot extensibility pattern. Graph Connectors are a complementary pattern for data ingestion.

</details>

---

### Question 2

**A declarative agent in M365 Copilot is best described as:**

- A) A fully custom backend service that processes all user interactions
- B) A manifest-defined agent with custom instructions, knowledge sources, and actions — without requiring backend code
- C) A Teams bot that uses the Bot Framework SDK to handle all message routing
- D) A Graph Connector that ingests external data into Microsoft 365

<details>
<summary>View Answer</summary>

**Correct Answer: B**

Declarative agents are defined via a manifest file that specifies custom instructions, knowledge sources (like SharePoint), and actions (API plugin references). They do not require backend code for the agent logic itself — Copilot's orchestrator handles the conversation flow based on the manifest configuration. Backend code is only needed if you add API plugin actions.

</details>

---

### Question 3

**When building an API plugin for M365 Copilot, what format is used to describe the endpoint?**

- A) GraphQL schema
- B) WSDL (Web Services Description Language)
- C) OpenAPI specification (Swagger)
- D) Protocol Buffers (.proto)

<details>
<summary>View Answer</summary>

**Correct Answer: C**

M365 Copilot API plugins use the **OpenAPI specification** (formerly Swagger) to describe REST endpoints. Copilot reads the OpenAPI spec to understand available operations, parameters, and response schemas. This enables Copilot's orchestrator to match user intent to the correct API operation and extract parameters from the conversation.

</details>

---

### Question 4

**Why are descriptive `operationId` names and detailed `description` fields critical in an OpenAPI spec for a Copilot plugin?**

- A) They determine the HTTP method used for the API call
- B) They are displayed as-is to the end user in the Copilot UI
- C) Copilot's orchestrator uses them for intent matching and parameter extraction
- D) They are required by the Azure API Management gateway

<details>
<summary>View Answer</summary>

**Correct Answer: C**

Copilot's orchestrator uses the `operationId`, `summary`, `description`, and parameter `description` fields to determine **when** to invoke a plugin and **how** to extract parameters from the user's natural language query. Vague or generic descriptions result in poor intent matching and incorrect parameter extraction.

</details>

---

### Question 5

**You need to ensure that when your M365 Copilot plugin queries Azure AI Foundry, the results respect the calling user's SharePoint permissions. Which authentication pattern should you implement?**

- A) API key authentication with a shared service key
- B) Client credentials flow with an application identity
- C) On-Behalf-Of (OBO) token flow
- D) Anonymous access with IP-based filtering

<details>
<summary>View Answer</summary>

**Correct Answer: C**

The **On-Behalf-Of (OBO) flow** enables your backend service to exchange the user's M365 token for an AI Foundry-scoped token, making the API call on behalf of the authenticated user. This ensures that search results respect the user's actual permissions (e.g., SharePoint ACLs), rather than using a service account that may have broader access.

</details>

---

### Question 6

**Your organization's M365 tenant is in the EU Data Boundary. Where must your Azure AI Foundry project be deployed?**

- A) Any Azure region — data residency only applies to Microsoft 365 data
- B) In an EU Azure region (e.g., West Europe, North Europe, France Central)
- C) In the US East region, since Azure OpenAI models are only available there
- D) In any region, as long as Private Link is enabled

<details>
<summary>View Answer</summary>

**Correct Answer: B**

When your M365 tenant is in the EU Data Boundary, your Azure AI Foundry project and associated resources (Azure AI Search, Azure OpenAI) must be deployed in an **EU Azure region** to comply with GDPR and the EU Data Boundary commitment. Cross-region API calls that transfer personal data outside the EU boundary violate these requirements, regardless of network isolation (Private Link).

</details>

---

### Question 7

**In Copilot Studio's multi-agent orchestration pattern, what is the role of the orchestrator agent?**

- A) It directly processes all user queries using a single AI model
- B) It routes user queries to specialized sub-agents based on intent and manages context handoff
- C) It stores conversation history in Azure Cosmos DB
- D) It generates Adaptive Cards for all responses

<details>
<summary>View Answer</summary>

**Correct Answer: B**

The orchestrator agent in Copilot Studio's multi-agent pattern acts as a **central router**. It receives user queries, determines the user's intent, and routes the request to the appropriate specialized sub-agent (e.g., a knowledge agent, analytics agent, or action agent). It also manages context handoff between agents and assembles the final response.

</details>

---

### Question 8

**You're building a message extension that returns search results from AI Foundry as rich cards in Teams. Which component handles the `composeExtension/query` activity?**

- A) Microsoft Graph API
- B) A class extending `TeamsActivityHandler` in the Bot Framework SDK
- C) An Azure Logic App with a Teams connector
- D) The M365 Copilot orchestrator directly

<details>
<summary>View Answer</summary>

**Correct Answer: B**

Search-based message extensions use the **Bot Framework SDK**. You create a class that extends `TeamsActivityHandler` and implement the `handleTeamsMessagingExtensionQuery` method. This method receives the user's search query, calls your AI Foundry endpoint, and returns the results formatted as Adaptive Cards or Hero Cards.

</details>

---

### Question 9

**Which of the following is the recommended approach for authenticating your Azure Functions backend to Azure AI Foundry services?**

- A) Store the API key in the function's application settings
- B) Use a system-assigned managed identity with RBAC role assignments
- C) Embed the API key directly in the OpenAPI specification
- D) Use a shared access signature (SAS) token

<details>
<summary>View Answer</summary>

**Correct Answer: B**

The recommended approach is to use a **system-assigned managed identity** for your Azure Functions app, with appropriate RBAC roles (e.g., "Cognitive Services User" for Azure OpenAI, "Search Index Data Reader" for Azure AI Search). This eliminates the need to manage secrets — no API keys in code, config, or Key Vault. The managed identity authenticates via `DefaultAzureCredential` in your code.

</details>

---

### Question 10

**You want to build a Copilot extension that generates Word documents from structured data stored in Azure AI Foundry. Which extension pattern is most appropriate?**

- A) Message Extension — because it supports rich card responses
- B) Graph Connector — because it ingests data into Microsoft Graph
- C) Declarative Agent — because it can be configured with document generation instructions and scoped knowledge
- D) Azure Logic App — because it can create files in SharePoint

<details>
<summary>View Answer</summary>

**Correct Answer: C**

A **Declarative Agent** is the most appropriate pattern for Word document generation scenarios. You can configure the agent with specific instructions for document formatting, connect it to your AI Foundry data sources via API plugin actions, and it will operate within the Word Copilot experience. While a Graph Connector could ingest data, it doesn't provide the generative capability. Message extensions are Teams-focused and don't integrate with Word's Copilot surface.

</details>

---

## 📊 Scoring Guide

| Score | Rating | Recommendation |
|---|---|---|
| **10/10** | ⭐ Expert | Proceed to Module 25 |
| **8–9/10** | ✅ Proficient | Proceed to Module 25 |
| **6–7/10** | 📖 Review Needed | Review sections on extensibility model and compliance |
| **4–5/10** | 🔄 Retake | Re-read the blog post and training guide, then retake |
| **0–3/10** | ⚠️ Foundation Gaps | Review Modules 18–23 before retaking |

---

## 📝 Answer Key

| Q | Answer | Topic |
|---|---|---|
| 1 | D | Extensibility model |
| 2 | B | Declarative agents |
| 3 | C | API plugin format |
| 4 | C | Plugin design |
| 5 | C | Authentication (OBO) |
| 6 | B | Data residency |
| 7 | B | Multi-agent orchestration |
| 8 | B | Message extensions |
| 9 | B | Managed identity |
| 10 | C | Per-app integration |

---

*Module 24 Assessment · Azure AI Foundry: Zero to Hero · Arc 5: Orchestration & Integration*
