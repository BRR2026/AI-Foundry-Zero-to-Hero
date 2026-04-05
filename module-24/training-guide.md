# Module 24: Integrating with Microsoft 365 Copilot

## Training Guide

> **Series:** Azure AI Foundry — Zero to Hero (Module 24 of 45)
> **Arc:** 5 — Orchestration & Integration
> **Duration:** 4 hours (lecture + lab)
> **Date:** April 2026

---

## 📋 Module Overview

This module teaches how to extend Microsoft 365 Copilot with custom intelligence powered by Azure AI Foundry. Participants will learn the M365 Copilot extensibility model, build message extensions and plugins, connect AI Foundry endpoints to Office applications, and address enterprise compliance and data residency requirements.

### Target Audience

- AI engineers extending enterprise AI into Microsoft 365
- Microsoft 365 developers building Copilot extensions
- Solution architects designing integrated AI systems
- IT administrators managing Copilot deployments

### Prerequisites

| Requirement | Details |
|---|---|
| **Modules 1–23** | Completion of prior modules (especially Modules 18–23 on orchestration) |
| **M365 License** | Microsoft 365 E3/E5 with Copilot license |
| **Azure Subscription** | Active subscription with AI Foundry project deployed |
| **Development Tools** | VS Code, Teams Toolkit, Node.js 20+, Azure Functions Core Tools |
| **Knowledge** | Familiarity with REST APIs, TypeScript/JavaScript, Azure AD |

---

## 🎯 Learning Objectives

By the end of this module, participants will be able to:

1. **Describe** the M365 Copilot extensibility model and differentiate between declarative agents, API plugins, and message extensions
2. **Build** a search-based message extension that queries an AI Foundry knowledge base
3. **Create** a declarative agent with an API plugin for M365 Copilot
4. **Implement** the On-Behalf-Of (OBO) authentication flow for user-context queries
5. **Configure** compliance controls including data residency, sensitivity labels, and audit logging
6. **Connect** AI Foundry prompt flows and agents to Copilot Studio
7. **Deploy** and test a Copilot plugin using Teams Toolkit

---

## 📚 Module Outline

### Part 1: Understanding M365 Copilot Extensibility (60 min)

#### 1.1 The Copilot Extensibility Model (20 min)
- How M365 Copilot processes user requests
- The role of the orchestrator in routing to plugins
- Three extension types: Declarative Agents, API Plugins, Message Extensions
- Graph Connectors as a complementary pattern
- When to use each extension type

#### 1.2 Declarative Agents (20 min)
- What is a declarative agent?
- Manifest structure and schema
- Instructions, conversation starters, and capabilities
- Connecting to SharePoint, OneDrive, and external APIs
- Limitations and best practices

#### 1.3 API Plugins (20 min)
- OpenAPI-based plugin definition
- Authentication options (API key, OAuth, managed identity)
- Request/response schema design for Copilot
- Parameter descriptions and semantic hints
- Testing with the Copilot developer tools

### Part 2: Building Message Extensions and Plugins (60 min)

#### 2.1 Search-Based Message Extensions (30 min)
- Bot Framework architecture for message extensions
- Handling `composeExtension/query` activities
- Transforming AI Foundry responses to Adaptive Cards
- Thumbnail cards, hero cards, and list layouts
- **Demo:** Live search against an AI Foundry RAG endpoint

#### 2.2 Action-Based Extensions (15 min)
- Task modules and form-based interactions
- Submitting data back to AI Foundry agents
- Confirmation cards and follow-up actions

#### 2.3 Plugin Manifest Deep Dive (15 min)
- `manifest.json` structure for Teams apps with Copilot extensions
- `copilotExtensions` configuration block
- `ai-plugin.json` for API plugin registration
- Icon, description, and developer info requirements

### Part 3: Connecting AI Foundry to Office Apps (45 min)

#### 3.1 Integration Architecture (15 min)
- End-to-end data flow: User → Copilot → Plugin → AI Foundry → Response
- Hosting options: Azure Functions, App Service, Container Apps
- Prompt Flow as the orchestration middleware
- Agent-based architecture with AI Foundry Agents SDK

#### 3.2 Per-Application Integration Patterns (15 min)
- **Teams:** Message extensions and bot interactions
- **Outlook:** Email processing and draft generation
- **Word:** Document generation from knowledge bases
- **Excel:** Data analysis with AI-powered insights
- **PowerPoint:** Content summarization and slide generation

#### 3.3 Copilot Studio Integration (15 min)
- Creating custom copilots with Copilot Studio
- Multi-agent orchestration patterns
- Connecting AI Foundry endpoints as plugin actions
- Publishing custom copilots to M365 Copilot
- **Video:** Multi-agent Orchestration in Microsoft Copilot Studio

### Part 4: Compliance and Data Residency (45 min)

#### 4.1 Data Residency Requirements (15 min)
- Understanding the EU Data Boundary
- Matching Azure regions to M365 tenant geography
- Cross-region data transfer implications
- Azure Private Link for network isolation

#### 4.2 Security and Authentication (15 min)
- Azure AD SSO for Copilot extensions
- On-Behalf-Of (OBO) token flow
- Managed identity for service-to-service calls
- Key Vault integration for secrets management

#### 4.3 Governance and Compliance (15 min)
- Microsoft Purview integration for sensitivity labels
- Data Loss Prevention (DLP) policies
- Unified Audit Log for plugin invocations
- Responsible AI content filters
- Admin consent and app approval workflows

### Part 5: Hands-on Lab (60 min)

#### Mini Hack: Create a M365 Copilot Plugin
- Scaffold a declarative agent project with Teams Toolkit
- Build an Azure Functions backend calling AI Foundry
- Define the OpenAPI specification
- Configure the declarative agent manifest
- Test in M365 Copilot with sideloading
- See `lab/hands-on-lab.md` for detailed instructions

---

## 🎥 Video Resource

**Multi-agent Orchestration in Microsoft Copilot Studio**
- **URL:** https://www.youtube.com/watch?v=WKKdBC2zM3k
- **Duration:** ~45 minutes
- **Key Topics:** Agent routing, context handoff, plugin actions, AI Foundry integration
- **When to Watch:** During Part 3.3 or as pre-reading

---

## 🧪 Assessment

Complete the 10-question assessment in `quiz/assessment.md` covering:
- Extensibility model concepts
- Plugin architecture and manifest structure
- Authentication and security patterns
- Compliance and data residency
- Integration implementation details

**Passing Score:** 80% (8 out of 10 correct)

---

## 📖 Key Terminology

| Term | Definition |
|---|---|
| **Declarative Agent** | A Copilot extension defined via manifest with instructions, knowledge, and actions |
| **API Plugin** | An OpenAPI-described REST endpoint that Copilot invokes during conversations |
| **Message Extension** | A Teams-based search/action extension surfaced in the compose experience |
| **Graph Connector** | A pipeline that ingests external data into Microsoft Graph for Copilot grounding |
| **OBO Flow** | On-Behalf-Of authentication where a service acts on behalf of the calling user |
| **Copilot Studio** | Low-code platform for building custom copilots and extending M365 Copilot |
| **Orchestrator** | The M365 Copilot component that routes user queries to appropriate plugins |
| **Adaptive Card** | A platform-agnostic card format for rich content in Teams and Copilot |

---

## 🔗 Resources

- [M365 Copilot Extensibility Documentation](https://learn.microsoft.com/microsoft-365-copilot/extensibility/)
- [Building Declarative Agents](https://learn.microsoft.com/microsoft-365-copilot/extensibility/build-declarative-agents)
- [Teams Toolkit Overview](https://learn.microsoft.com/microsoftteams/platform/toolkit/teams-toolkit-fundamentals)
- [Microsoft Copilot Studio](https://learn.microsoft.com/microsoft-copilot-studio/overview)
- [Azure AI Foundry Documentation](https://learn.microsoft.com/azure/ai-studio/)
- [EU Data Boundary for Microsoft Cloud](https://learn.microsoft.com/privacy/eudb/eu-data-boundary-learn)

---

## ⏭️ What's Next

**Module 25: Power Platform Integration** — Extend AI Foundry intelligence into Power Automate flows, Power Apps, and custom business process automation.

---

*Module 24 of 45 · Azure AI Foundry: Zero to Hero · Arc 5: Orchestration & Integration · April 2026*
