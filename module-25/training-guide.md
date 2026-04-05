# Module 25 — Real-World Integration Patterns

## Training Guide

> **Arc 5: Orchestration & Integration** | Azure AI Foundry: Zero to Hero (Module 25 of 45)
>
> **Date:** April 2026 | **Duration:** 3 hours (lecture + lab) | **Level:** Intermediate–Advanced

---

## 🎯 Learning Objectives

By the end of this module, learners will be able to:

1. **Design** a Microsoft Teams bot that proxies user messages to an Azure AI Foundry agent and returns responses with Adaptive Cards.
2. **Build** Power Platform custom connectors (Power Automate, Power Apps) that invoke AI Foundry agents without writing application code.
3. **Implement** synchronous, asynchronous, and streaming REST API patterns for external application integration.
4. **Architect** event-driven workflows using Azure Event Grid, Service Bus, and webhooks to trigger AI agent invocations.
5. **Apply** production security best practices: managed identity, API Management, rate limiting, and observability.

---

## 📋 Prerequisites

| Requirement | Details |
|---|---|
| **Modules completed** | Modules 1–24 (especially Module 23: Agent Deployment, Module 24: Orchestration Frameworks) |
| **Azure subscription** | Active subscription with Contributor access |
| **AI Foundry project** | A deployed agent from Module 23 with at least one tool configured |
| **Development tools** | VS Code, Python 3.10+ or .NET 8+, Azure Functions Core Tools v4 |
| **Teams access** | Microsoft Teams with side-loading enabled (for Mini Hack) |
| **Power Platform** | Power Automate license (optional — for Power Platform exercises) |

---

## 📚 Module Outline

### Part 1: Microsoft Teams Bot Integration (45 min)

#### Lecture Topics
- Architecture: Teams → Bot Service → Azure Functions → AI Foundry Agent
- Bot Framework SDK fundamentals (Python & C#)
- Activity handling: messages, card actions, proactive notifications
- Adaptive Cards for rich, interactive responses
- Thread management: mapping Teams conversation IDs to AI Foundry thread IDs

#### Instructor Notes
- **Demo:** Walk through the Bot Framework Emulator showing a live connection to an AI Foundry agent.
- **Key concept:** Emphasize the "proxy pattern" — the bot backend is a thin layer that authenticates with AI Foundry and forwards requests. Business logic lives in the agent, not the bot code.
- **Common mistake:** Students often try to embed the AI Foundry connection string in the Teams app manifest. Stress that all secrets must stay server-side.

#### Discussion Questions
1. When would you choose a Teams bot over a Teams Message Extension for AI integration?
2. How would you handle a scenario where the AI agent takes 45+ seconds to respond?
3. What strategies can you use to provide a "typing indicator" experience in Teams?

---

### Part 2: Power Platform Connectors (30 min)

#### Lecture Topics
- Custom connector architecture: OpenAPI definition → Power Platform action
- Azure Function as a connector backend
- Authentication options: API Key, OAuth 2.0, Microsoft Entra ID
- Power Automate flows: email-to-summary pipeline
- Power Apps canvas app: question-and-answer interface

#### Instructor Notes
- **Demo:** Build a Power Automate flow live that takes an email trigger, sends the body to the custom connector, and posts the AI summary to Teams.
- **Key concept:** Custom connectors democratize AI access. A citizen developer with zero coding skills can incorporate AI Foundry agent intelligence into their workflows.
- **Tip:** Show the OpenAPI definition import feature — students can generate the connector from the Azure Function's swagger endpoint.

#### Discussion Questions
1. What governance controls should you implement when exposing AI agents to citizen developers?
2. How does DLP (Data Loss Prevention) policy in Power Platform interact with custom connectors?
3. When should you use Power Virtual Agents vs. a custom Teams bot backed by AI Foundry?

---

### Part 3: REST API Patterns (30 min)

#### Lecture Topics
- Synchronous request-response pattern
- Asynchronous polling with 202 Accepted
- Server-Sent Events (SSE) for streaming responses
- API versioning and contract design
- Authentication matrix: API Key, OAuth 2.0, Managed Identity, APIM

#### Instructor Notes
- **Demo:** Show all three patterns side-by-side using Postman or httpie. Highlight the UX difference — especially how streaming delivers first tokens in under 1 second vs. 5–10 seconds for synchronous.
- **Key concept:** The choice of API pattern depends on the client's capabilities and the expected response time. Mobile apps benefit from streaming; batch pipelines prefer async polling.
- **Lab tie-in:** The Mini Hack uses the synchronous pattern; challenge advanced students to add streaming.

#### Code Samples
- Python: Flask streaming endpoint with SSE
- C#: Azure Function with synchronous invoke
- JavaScript: Fetch API with ReadableStream for client-side streaming consumption

---

### Part 4: Webhook & Event-Driven Architectures (30 min)

#### Lecture Topics
- Incoming webhooks: validating signatures, parsing payloads, invoking agents
- Event Grid integration: blob triggers, custom topics, partner events
- Service Bus patterns: queue-based load leveling for agent invocations
- Outgoing webhooks: agent function tools that call external APIs
- Dead-letter handling and retry strategies

#### Instructor Notes
- **Demo:** Configure an Event Grid subscription that triggers on Blob Storage uploads. Show a PDF being uploaded, the Azure Function receiving the event, the AI Foundry agent analyzing the document, and the result being posted to Teams.
- **Key concept:** Event-driven architectures decouple the "trigger" from the "processing." This enables reliable, scalable, and asynchronous AI workflows.
- **Production pattern:** Always implement dead-letter queues and exponential backoff for agent invocations in event-driven scenarios.

#### Discussion Questions
1. How do you prevent duplicate processing when Event Grid delivers the same event twice?
2. What is the difference between Event Grid and Service Bus, and when would you choose each?
3. How would you implement a "circuit breaker" pattern for AI agent invocations?

---

### Part 5: Mini Hack — Teams Bot + AI Foundry Agent (45 min)

#### Lab Summary
Students build a complete Teams bot that:
1. Receives messages from Microsoft Teams
2. Maintains conversation context using Azure Table Storage
3. Forwards messages to an AI Foundry agent
4. Returns formatted responses using Adaptive Cards
5. Handles errors gracefully with fallback messages

See **lab/hands-on-lab.md** for detailed step-by-step instructions.

#### Instructor Tips
- Pre-provision Bot Service resources if time is limited.
- Have students pair up — one drives the bot code, the other tests in Teams.
- Keep the Bot Framework Emulator available as a fallback if Teams side-loading is restricted.

---

## 🛠️ Resources & References

### Documentation
- [Azure AI Foundry Agent Service — Overview](https://learn.microsoft.com/azure/ai-services/agents/overview)
- [Bot Framework SDK — Python](https://learn.microsoft.com/azure/bot-service/bot-service-quickstart-create-bot)
- [Power Platform Custom Connectors](https://learn.microsoft.com/connectors/custom-connectors/)
- [Azure Event Grid — Overview](https://learn.microsoft.com/azure/event-grid/overview)
- [Azure API Management — Overview](https://learn.microsoft.com/azure/api-management/api-management-key-concepts)

### Sample Code
- [AI Foundry Agent Samples (GitHub)](https://github.com/Azure/azure-ai-projects)
- [Bot Framework Samples](https://github.com/microsoft/BotBuilder-Samples)
- [Teams AI Library](https://github.com/microsoft/teams-ai)

### Video
- **Microsoft Foundry — AI app & agent factory** (MS Mechanics)
  - YouTube: https://www.youtube.com/watch?v=C6rxEGJay70

---

## 📝 Assessment

- **Quiz:** 10 multiple-choice questions covering all integration patterns — see `quiz/assessment.md`
- **Lab Completion:** Functional Teams bot with verified conversation round-trip
- **Stretch Goal:** Add streaming support to the REST API endpoint

---

## ⏭️ What's Next

| Direction | Module |
|---|---|
| ← Previous | Module 24: Orchestration Frameworks |
| → Next | Module 26: CI/CD for AI Agents |

---

> **Note:** This training covers **Azure AI Foundry** — the unified platform for building and deploying AI agents. This is not Azure Machine Learning. AI Foundry provides agent hosting, prompt management, evaluation, and integration capabilities in a single experience.
