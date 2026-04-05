# Module 23: Connecting AI Foundry to Copilot Studio

> **Azure AI Foundry: Zero to Hero** — 45-Module Training Series  
> **Arc 5: Orchestration & Integration** | April 2026  
> **Duration:** ~90 minutes | **Level:** Intermediate–Advanced

---

## Instructor Guide Overview

This module bridges Azure AI Foundry's powerful model hosting and RAG capabilities with Microsoft Copilot Studio's low-code conversational platform. Students will learn to publish AI Foundry endpoints as Copilot plugins, build conversation flows, and configure secure authentication between the two services.

---

## Learning Objectives

By the end of this module, learners will be able to:

1. **Explain** Copilot Studio's role in the Microsoft AI ecosystem and describe its core capabilities
2. **Publish** AI Foundry models and endpoints as Copilot Studio custom plugins using OpenAPI specifications
3. **Design** conversational experiences in Copilot Studio that leverage AI Foundry intelligence
4. **Configure** authentication and secure data flow between AI Foundry and Copilot Studio using Entra ID
5. **Integrate** a RAG-powered agent into a Copilot Studio bot for end-to-end deployment

---

## Prerequisites

| Requirement | Details |
|---|---|
| Prior Modules | Module 22 (Event-Driven Architectures with AI Foundry) |
| Azure Resources | AI Foundry hub + project with a deployed model endpoint |
| Copilot Studio | Trial or licensed Copilot Studio environment |
| Knowledge | REST APIs, OAuth 2.0 basics, conversation design principles |
| Deployed Agent | A working RAG agent (from Module 14+) on an active endpoint |

---

## Module Outline

### Section 1: Copilot Studio Overview and Capabilities (20 min)

**Key Topics:**
- Evolution from Power Virtual Agents to Copilot Studio
- Core capabilities: visual authoring, generative answers, plugin ecosystem, omnichannel deployment
- Copilot Studio architecture: topic routing, generative orchestration, and AI classification
- Where AI Foundry fits: custom models as intelligence layer behind Copilot Studio

**Instructor Notes:**
- Start with a live demo of Copilot Studio's authoring canvas — show how easy it is to create a basic bot
- Emphasize the paradigm shift: Copilot Studio is the **front-end** (conversational UI), AI Foundry is the **back-end** (model intelligence)
- Show the built-in generative answers feature, then explain why enterprises need AI Foundry for domain-specific scenarios

**Discussion Prompt:**
> "Think about your organization's current chatbot or virtual assistant. What limitations does it have that AI Foundry could address? How would Copilot Studio improve the user experience?"

---

### Section 2: Publishing AI Foundry Models as Copilot Plugins (25 min)

**Key Topics:**
- Plugin architecture and OpenAPI specification structure
- Creating an OpenAPI spec for an AI Foundry inference endpoint
- Step-by-step plugin registration in Copilot Studio
- Plugin descriptions and their role in generative orchestration

**Instructor Notes:**
- Walk through the OpenAPI spec creation live — emphasize the `description` fields
- Demonstrate uploading the spec in Copilot Studio's plugin configuration
- Show both API Key and Entra ID authentication options
- **Common mistake:** Students often write vague plugin descriptions. Show examples of good vs. bad descriptions and how they affect routing accuracy

**Hands-On Activity (10 min):**
Have students write an OpenAPI specification for their AI Foundry endpoint:
1. Define the `/score` endpoint with POST method
2. Specify input schema (question, chat_history)
3. Specify output schema (answer, citations)
4. Write a descriptive `description` field

**Key OpenAPI Fields for Plugin Success:**

```yaml
info:
  description: |
    Be specific! Instead of "Answers questions" write:
    "Answers questions about Contoso Corp's HR policies,
    including benefits, leave, compensation, and onboarding
    procedures, using the official HR document library."
paths:
  /score:
    post:
      operationId: queryHRAgent  # Unique, descriptive
      summary: "Query the HR knowledge base"
      description: "Detailed description of what this does..."
```

---

### Section 3: Building Conversational Experiences (20 min)

**Key Topics:**
- Designing conversation topics with AI Foundry plugin invocations
- Power Fx expressions for plugin calls and response formatting
- Multi-turn conversation patterns with context passing
- Classic topics vs. generative orchestration trade-offs

**Instructor Notes:**
- Demo building a complete topic from scratch:
  1. Create a new topic with trigger phrases
  2. Add a plugin action node
  3. Map input variables (user message, history)
  4. Format the response (including citations)
  5. Add error handling nodes
- Show generative orchestration mode — demonstrate how the AI automatically invokes the correct plugin
- Emphasize the Adaptive Card pattern for rich responses in Teams

**Live Demo Sequence:**
1. Open Copilot Studio → Create new copilot
2. Register AI Foundry plugin via OpenAPI
3. Create a "Knowledge Q&A" topic
4. Configure generative orchestration
5. Test in the built-in chat panel
6. Show the response with citations

---

### Section 4: Authentication and Data Flow (15 min)

**Key Topics:**
- Three authentication patterns: API Key, Entra ID (client credentials), User-delegated
- App registration and RBAC configuration steps
- End-to-end data flow with security checkpoints
- Data residency, DLP policies, and compliance considerations

**Instructor Notes:**
- Draw the authentication flow on the whiteboard — show token acquisition and validation steps
- Demo the Azure CLI commands for app registration and role assignment
- **Security emphasis:** Always use Entra ID in production; API keys are for dev/test only
- Cover DLP policies — show how to place the AI Foundry connector in the "Business" data group

**Compliance Checklist for Students:**
- [ ] Copilot Studio and AI Foundry in same Azure region
- [ ] Entra ID app registration created with minimal scopes
- [ ] Cognitive Services User role assigned (not Contributor)
- [ ] DLP policies configured in Power Platform Admin Center
- [ ] Audit logging enabled in both platforms
- [ ] Content safety filters active on AI Foundry endpoint

---

### Section 5: Deep Dive Video (10 min)

**Video:** "Multi-agent Orchestration in Microsoft Copilot Studio"  
**Speaker:** Shervin Shaffie, Microsoft Principal Tech Specialist  
**URL:** https://www.youtube.com/watch?v=WKKdBC2zM3k

**Pre-Video Context:**
- Set up the video by explaining multi-agent patterns
- Ask students to note: How does the orchestration layer decide which agent to call?

**Post-Video Discussion:**
- What patterns from the video apply to AI Foundry integration?
- How would you design a multi-agent copilot for your organization?

---

### Section 6: Advanced Integration Patterns (Optional, 10 min)

**Key Topics:**
- Multi-agent routing with multiple AI Foundry plugins
- Adaptive Cards for rich, interactive responses
- Error handling and fallback patterns
- Monitoring and analytics across both platforms

**Instructor Notes:**
- This section is optional for advanced classes
- Focus on the multi-agent pattern if time allows — it's the most impactful
- Show the Adaptive Card Designer (adaptivecards.io) for building response templates

---

## Mini Hack: Publish Your RAG Agent as a Copilot Studio Bot

**Duration:** 45–60 minutes  
**Difficulty:** Intermediate  
**Mode:** Individual or pairs

### Objectives
Students will connect their existing AI Foundry RAG agent to Copilot Studio and deploy a working conversational bot.

### Steps

| Step | Task | Time |
|------|------|------|
| 1 | Verify AI Foundry RAG agent endpoint is accessible | 5 min |
| 2 | Create OpenAPI specification for the endpoint | 10 min |
| 3 | Register plugin in Copilot Studio with API key auth | 10 min |
| 4 | Build a topic with plugin invocation and response formatting | 10 min |
| 5 | Enable generative orchestration and test routing | 5 min |
| 6 | Publish to Teams and test with a colleague | 10 min |

### Success Criteria
- [ ] OpenAPI spec accurately describes the AI Foundry endpoint
- [ ] Plugin successfully registered and returns responses in test panel
- [ ] Topic correctly formats responses with citations
- [ ] Generative orchestration correctly routes relevant queries to the plugin
- [ ] Bot is accessible in Microsoft Teams
- [ ] At least 3 different queries produce grounded, accurate responses

### Common Issues and Solutions

| Issue | Solution |
|---|---|
| Plugin returns 401 Unauthorized | Verify API key is correct; check RBAC roles on AI Foundry endpoint |
| Plugin never gets invoked | Improve the plugin description; make it more specific about query types |
| Slow response times (>10s) | Check AI Foundry endpoint scaling; consider enabling auto-scale |
| Citations not showing | Verify output schema maps `citations` array correctly |
| Generative orchestration ignores plugin | Ensure orchestration mode is enabled and plugin description is clear |

---

## Assessment Strategy

| Method | Weight | Details |
|---|---|---|
| Quiz (10 MCQs) | 25% | Covers concepts, architecture, and security patterns |
| Mini Hack Completion | 50% | Working Copilot Studio bot connected to AI Foundry |
| Peer Review | 25% | Students test and provide feedback on each other's bots |

---

## Supplementary Materials

### Recommended Reading
- [Microsoft Copilot Studio Documentation](https://learn.microsoft.com/microsoft-copilot-studio/)
- [Azure AI Foundry Documentation](https://learn.microsoft.com/azure/ai-studio/)
- [Copilot Studio Plugins Overview](https://learn.microsoft.com/microsoft-copilot-studio/copilot-plugins-overview)
- [Generative Orchestration in Copilot Studio](https://learn.microsoft.com/microsoft-copilot-studio/advanced-generative-actions)
- [Microsoft Entra ID Platform Docs](https://learn.microsoft.com/entra/identity-platform/)

### Environment Setup
```
Required Services:
├── Azure AI Foundry (hub + project + deployed RAG agent)
├── Microsoft Copilot Studio (trial or licensed)
├── Microsoft Entra ID (app registration)
├── Microsoft Teams (for deployment testing)
└── Azure CLI (for RBAC configuration)
```

### Key Terminology

| Term | Definition |
|---|---|
| **Copilot Studio** | Microsoft's low-code platform for building conversational AI agents (copilots) |
| **Plugin** | An OpenAPI-defined extension that exposes an external API to Copilot Studio |
| **Generative Orchestration** | AI-powered routing that automatically selects the right topic or plugin |
| **Topic** | A self-contained conversational unit in Copilot Studio with triggers and actions |
| **Adaptive Card** | A cross-platform UI card format for rendering rich, interactive content |
| **Custom Connector** | A Power Platform connector wrapping an external REST API |
| **On-behalf-of (OBO) flow** | OAuth pattern where the service acts with the user's delegated permissions |

---

## Facilitator Checklist

- [ ] AI Foundry RAG agent deployed and endpoint verified
- [ ] Copilot Studio environments provisioned for all students
- [ ] Test Entra ID app registration created and shared
- [ ] OpenAPI spec template distributed (starter file)
- [ ] Teams test channel created for bot deployment
- [ ] Slide deck reviewed and customized
- [ ] Video tested for playback
- [ ] Assessment quiz loaded into LMS

---

## Timing Guide

| Section | Duration | Running Total |
|---|---|---|
| Copilot Studio Overview | 20 min | 20 min |
| Publishing Models as Plugins | 25 min | 45 min |
| Building Conversational Experiences | 20 min | 65 min |
| Authentication & Data Flow | 15 min | 80 min |
| Video + Discussion | 10 min | 90 min |
| Mini Hack | 45–60 min | 135–150 min |

**Total Module Time:** ~2.5 hours (including Mini Hack)

---

*Azure AI Foundry: Zero to Hero — Module 23 of 45*  
*Arc 5: Orchestration & Integration | April 2026*
