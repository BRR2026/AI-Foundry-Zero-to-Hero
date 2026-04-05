# Module 23 Assessment: Connecting AI Foundry to Copilot Studio

> **Azure AI Foundry: Zero to Hero** — 45-Module Training Series  
> **Arc 5: Orchestration & Integration** | April 2026  
> **Total Questions:** 10 | **Passing Score:** 80% (8/10)  
> **Time Limit:** 15 minutes

---

## Instructions

Select the **best** answer for each question. Each question has exactly one correct answer. Questions cover Copilot Studio fundamentals, AI Foundry plugin integration, authentication patterns, and conversational design principles.

---

### Question 1

**What specification format does Copilot Studio use to discover and register external API endpoints as custom plugins?**

- A) GraphQL Schema Definition Language (SDL)
- B) OpenAPI (Swagger) specification
- C) Azure Resource Manager (ARM) templates
- D) Protocol Buffers (protobuf) definitions

<details>
<summary>Show Answer</summary>

**✅ B) OpenAPI (Swagger) specification**

Copilot Studio uses OpenAPI 3.0 specifications to define the API contract for custom plugins. The spec describes endpoints, parameters, authentication requirements, and — critically — the descriptions that the AI orchestrator uses to determine when to invoke the plugin.

</details>

---

### Question 2

**Which field in the OpenAPI specification is most critical for Copilot Studio's generative orchestrator to determine when to invoke an AI Foundry plugin?**

- A) The `operationId` field
- B) The `servers` URL field
- C) The `description` field
- D) The `version` field

<details>
<summary>Show Answer</summary>

**✅ C) The `description` field**

The `description` field (at both the API and operation levels) is what Copilot Studio's generative orchestrator reads to understand what the plugin does and when it should be invoked. A vague description leads to poor routing; a specific, detailed description enables accurate automatic invocation.

</details>

---

### Question 3

**In the integration architecture between Copilot Studio and AI Foundry, what is the primary role of Copilot Studio?**

- A) Model training and fine-tuning platform
- B) Conversational front-end and user experience layer
- C) Vector database and document indexing service
- D) Model deployment and inference hosting platform

<details>
<summary>Show Answer</summary>

**✅ B) Conversational front-end and user experience layer**

Copilot Studio serves as the conversational front-end — handling user interactions, topic routing, channel deployment (Teams, web, etc.), and conversation management. Azure AI Foundry serves as the AI back-end, providing model inference, RAG pipelines, and content safety. Together, they deliver a complete AI conversational solution.

</details>

---

### Question 4

**Which authentication method is recommended for production deployments when connecting Copilot Studio to AI Foundry endpoints?**

- A) API Key stored in Copilot Studio configuration
- B) Basic authentication with username and password
- C) Microsoft Entra ID with OAuth 2.0 client credentials flow
- D) Shared Access Signatures (SAS tokens)

<details>
<summary>Show Answer</summary>

**✅ C) Microsoft Entra ID with OAuth 2.0 client credentials flow**

For production deployments, Entra ID (formerly Azure AD) with OAuth 2.0 is recommended because it eliminates the need to store static secrets (like API keys) in Copilot Studio. Tokens are acquired dynamically, and access is controlled through RBAC role assignments on the AI Foundry endpoint. API keys are acceptable only for development and testing.

</details>

---

### Question 5

**What Azure RBAC role should be assigned to the Copilot Studio service principal to allow it to call AI Foundry inference endpoints?**

- A) Contributor
- B) Owner
- C) Cognitive Services User
- D) Reader

<details>
<summary>Show Answer</summary>

**✅ C) Cognitive Services User**

The **Cognitive Services User** role grants permission to call inference endpoints on Azure AI Foundry (Cognitive Services) resources. This follows the principle of least privilege — it allows calling the model but not modifying the resource configuration. Contributor or Owner roles would be overly permissive.

</details>

---

### Question 6

**What is the key difference between Classic Topics and Generative Orchestration in Copilot Studio when routing queries to an AI Foundry plugin?**

- A) Classic Topics support multiple plugins; Generative Orchestration supports only one
- B) Classic Topics use keyword trigger phrases; Generative Orchestration uses AI-powered intent classification to automatically select plugins
- C) Generative Orchestration requires API Key auth; Classic Topics require Entra ID
- D) Classic Topics are faster; Generative Orchestration has higher latency due to model calls

<details>
<summary>Show Answer</summary>

**✅ B) Classic Topics use keyword trigger phrases; Generative Orchestration uses AI-powered intent classification to automatically select plugins**

Classic Topics route conversations based on manually configured trigger phrases and explicit action nodes. Generative Orchestration uses an AI model to classify user intent and automatically determine which plugin or topic to invoke, based on the plugin's description. This enables more natural, open-ended interactions without manual topic configuration for every query type.

</details>

---

### Question 7

**When building a multi-turn conversation with an AI Foundry RAG agent through Copilot Studio, what must be passed to the AI Foundry endpoint to enable contextual follow-up questions?**

- A) The user's email address for personalization
- B) The Copilot Studio session ID
- C) The conversation history (previous turns)
- D) The topic trigger phrase that initiated the conversation

<details>
<summary>Show Answer</summary>

**✅ C) The conversation history (previous turns)**

To enable contextual follow-up questions, the full conversation history (previous user messages and assistant responses) must be sent to the AI Foundry RAG agent with each request. This is typically passed via a `chat_history` parameter in the OpenAPI spec. The RAG agent uses this context to understand follow-up references (e.g., "What about for digital products?" following a question about refund policies).

</details>

---

### Question 8

**An organization has deployed three AI Foundry agents — one for HR, one for IT Support, and one for Finance — each as separate Copilot Studio plugins. How does Copilot Studio determine which plugin to invoke for a user's query?**

- A) It calls all three plugins simultaneously and returns the best response
- B) The user must manually select which agent to query before asking their question
- C) The generative orchestrator reads each plugin's description and routes to the most relevant one based on the user's intent
- D) Plugins are invoked in the order they were registered, and the first successful response is used

<details>
<summary>Show Answer</summary>

**✅ C) The generative orchestrator reads each plugin's description and routes to the most relevant one based on the user's intent**

When generative orchestration is enabled, Copilot Studio's AI engine evaluates each registered plugin's description against the user's message to determine the best match. This is why plugin descriptions must be specific about their domain — the orchestrator uses them to differentiate between plugins. For example, an HR question would be routed to the HR plugin, while an IT issue would go to the IT Support plugin.

</details>

---

### Question 9

**Which compliance feature provides "defense-in-depth" when Copilot Studio calls an AI Foundry endpoint?**

- A) AI Foundry's content safety filters remain active even when called through Copilot Studio, providing dual-layer protection alongside Copilot Studio's DLP policies
- B) Copilot Studio disables AI Foundry's content safety to avoid duplicate filtering
- C) All traffic is encrypted with customer-managed keys stored in Azure Key Vault
- D) Copilot Studio applies its own content safety model to replace AI Foundry's filters

<details>
<summary>Show Answer</summary>

**✅ A) AI Foundry's content safety filters remain active even when called through Copilot Studio, providing dual-layer protection alongside Copilot Studio's DLP policies**

Defense-in-depth is achieved because both platforms enforce their own governance independently. Copilot Studio applies Power Platform DLP policies and environment-level access controls, while AI Foundry's content safety filters (for harmful content, jailbreak detection, etc.) remain active regardless of how the endpoint is called. Neither platform disables the other's protections — they operate as complementary layers.

</details>

---

### Question 10

**You register an AI Foundry RAG plugin in Copilot Studio, but the generative orchestrator never invokes it — even for clearly relevant queries. What is the most likely cause?**

- A) The AI Foundry endpoint is returning HTTP 200 responses too quickly
- B) The plugin's OpenAPI description is too vague or generic for the orchestrator to match user intent
- C) Copilot Studio only supports Microsoft-built plugins, not custom connectors
- D) The API key has expired and needs to be rotated in Azure Key Vault

<details>
<summary>Show Answer</summary>

**✅ B) The plugin's OpenAPI description is too vague or generic for the orchestrator to match user intent**

The most common reason for a plugin not being invoked is a poorly written description. If the description says something vague like "Answers questions using AI," the orchestrator cannot determine when to use it versus other plugins or built-in capabilities. The fix is to write specific, detailed descriptions that clearly explain the domain, types of questions, and data sources the plugin handles. Additionally, ensure generative orchestration mode is actually enabled in the Copilot Studio settings.

</details>

---

## Scoring Guide

| Score | Grade | Status |
|---|---|---|
| 10/10 | ⭐ Excellent | Mastery of AI Foundry + Copilot Studio integration |
| 8–9/10 | ✅ Pass | Solid understanding; review missed topics |
| 6–7/10 | ⚠️ Needs Review | Revisit Sections 2 (Plugins) and 4 (Authentication) |
| Below 6 | ❌ Retake | Complete the lab exercises and re-read the blog before retaking |

---

## Answer Key (Quick Reference)

| Question | Answer | Key Concept |
|---|---|---|
| 1 | B | OpenAPI specification for plugins |
| 2 | C | Description field drives orchestration routing |
| 3 | B | Copilot Studio = conversational front-end |
| 4 | C | Entra ID OAuth 2.0 for production auth |
| 5 | C | Cognitive Services User RBAC role |
| 6 | B | Classic vs. Generative orchestration routing |
| 7 | C | Conversation history for multi-turn context |
| 8 | C | Multi-agent routing via plugin descriptions |
| 9 | A | Defense-in-depth with dual governance layers |
| 10 | B | Vague descriptions prevent plugin invocation |

---

## Topics for Further Study

If you scored below 80%, focus on these areas:

- **Questions 1–2 missed:** Review Section 2 (Publishing Models as Plugins) in the blog
- **Questions 3–4 missed:** Review Section 1 (Copilot Studio Overview) and Section 4 (Authentication)
- **Questions 5–6 missed:** Review Section 4 (Authentication) and Section 3 (Building Conversations)
- **Questions 7–8 missed:** Review Section 3 (Multi-Turn Conversations) and Section 6 (Advanced Patterns)
- **Questions 9–10 missed:** Review Section 4 (Compliance) and Section 2 (Plugin Descriptions)

---

*Azure AI Foundry: Zero to Hero — Module 23 Assessment*  
*Arc 5: Orchestration & Integration | April 2026*
