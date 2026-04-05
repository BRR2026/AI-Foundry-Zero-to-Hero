# Module 43: Deploy & Integrate with Agents/Copilot

## Azure AI Foundry: Zero to Hero — Training Guide

**ARC 9: Capstone & Mastery** · Module 43 of 45 · April 2026

---

## 📋 Module Overview

| Field | Details |
|---|---|
| **Module** | 43 of 45 |
| **Title** | Deploy & Integrate with Agents/Copilot |
| **Arc** | 9 — Capstone & Mastery |
| **Duration** | 6–8 hours (lecture + lab) |
| **Level** | Advanced / Capstone |
| **Format** | Instructor-led or self-paced |
| **Prerequisites** | Modules 1–42 completed, active Azure subscription |

---

## 🎯 Learning Objectives

By the end of this module, learners will be able to:

1. **Build AI agents** on top of existing RAG pipelines using Azure AI Foundry Agent Service
2. **Register custom function tools** to extend agent capabilities beyond retrieval
3. **Integrate with Microsoft Copilot Studio** using custom connectors and topic authoring
4. **Publish agents to Microsoft Teams** and M365 channels
5. **Design RESTful API endpoints** for external consumer access with proper security
6. **Implement end-to-end testing** using pytest and the Azure AI Evaluation SDK
7. **Deploy a fully integrated solution** as a capstone project

---

## 📚 Curriculum Outline

### Session 1: Agent Foundations (90 min)

#### 1.1 From RAG to Agents (30 min)
- Review of RAG pipeline architecture (Modules 28–35)
- What AI agents add: state management, planning, tool orchestration
- Agent vs. chat completion: key differences
- Azure AI Foundry Agent Service overview

#### 1.2 Building Your First Agent (30 min)
- Setting up the `azure-ai-projects` SDK
- Creating an agent with Azure AI Search grounding
- Configuring semantic search and top-k retrieval
- Agent instructions and persona design

#### 1.3 Custom Function Tools (30 min)
- Defining function tools with proper schemas
- Implementing tool handlers (ticket creation, order lookup)
- ToolSet composition: combining RAG + functions
- Error handling in tool calls

**Hands-On Activity:** Create an agent with a RAG tool and one custom function tool. Test a multi-turn conversation.

---

### Session 2: Copilot Studio & Teams Integration (90 min)

#### 2.1 Copilot Studio Integration (45 min)
- Copilot Studio architecture and capabilities
- Exposing your agent as a FastAPI endpoint
- Creating a custom connector in Copilot Studio
- Topic authoring with agent handoff
- Authentication configuration (OAuth 2.0 / Entra ID)
- Testing and publishing

#### 2.2 Microsoft Teams Integration (45 min)
- Integration approaches comparison:
  - Copilot Studio → Teams (low-code)
  - Teams Bot Framework (medium complexity)
  - Teams AI Library (AI-native)
  - Message Extensions (high complexity)
- Publishing via Copilot Studio to Teams
- Teams AI Library integration pattern
- Adaptive Cards for rich responses
- SSO authentication in Teams

**Hands-On Activity:** Connect your agent to Copilot Studio and publish to Teams. Verify end-to-end chat flow.

---

### Session 3: API Endpoints & External Access (60 min)

#### 3.1 API Design for AI Agents (30 min)
- RESTful API design principles for conversational AI
- Request/response schema design
- Session management and conversation threading
- Streaming vs. synchronous responses
- Error handling and status codes

#### 3.2 Azure API Management (30 min)
- Wrapping agents in Azure Functions
- APIM gateway configuration
- Rate limiting and throttling policies
- JWT validation with Entra ID
- API versioning strategies
- Health check and readiness endpoints
- OpenAPI specification generation

**Hands-On Activity:** Deploy your agent API to Azure Functions and configure APIM with rate limiting and authentication.

---

### Session 4: End-to-End Testing & Quality Evaluation (90 min)

#### 4.1 Testing Strategy (30 min)
- Testing pyramid for AI agents
- Unit testing: functions, tools, prompts
- Integration testing: agent + RAG, agent + tools
- E2E testing: deployed endpoints, full conversations
- Test data management and fixtures

#### 4.2 Automated E2E Tests (30 min)
- pytest test suite structure
- Health check tests
- Single-turn query tests
- Multi-turn conversation tests
- Tool invocation verification tests
- Error handling and edge case tests
- CI/CD integration with GitHub Actions

#### 4.3 AI Quality Evaluation (30 min)
- Azure AI Evaluation SDK overview
- Quality metrics: groundedness, relevance, coherence, fluency
- Building evaluation datasets
- Running batch evaluations
- Setting quality thresholds and gates
- Continuous evaluation in production

**Hands-On Activity:** Write 5+ E2E tests and run an AI quality evaluation. Verify all quality scores exceed 4.0/5.0.

---

### Session 5: Capstone Mini Hack (120 min)

#### 5.1 Integration Challenge
Build and deploy a complete solution that includes:
1. AI agent with RAG + custom tools
2. FastAPI/Azure Functions wrapper
3. Copilot Studio integration
4. Teams channel deployment
5. APIM-protected external API
6. Comprehensive E2E test suite
7. AI quality evaluation report

#### 5.2 Demo & Peer Review
- 3-minute solution demonstration
- Peer review of architecture decisions
- Quality score comparison
- Discussion of production readiness

---

## 🎥 Featured Video

**Build An AI Agent From Scratch Using Microsoft Foundry**
- Channel: Azure Innovation Station
- URL: https://www.youtube.com/watch?v=wL8H0RySf6I
- Duration: ~30 minutes
- Topics: Agent creation, tool integration, testing, deployment

> Watch this video before or during Session 1 to see a complete agent build walkthrough.

---

## 🛠️ Technical Requirements

### Software
| Tool | Version | Purpose |
|---|---|---|
| Python | 3.10+ | Agent development |
| Azure CLI | Latest | Azure resource management |
| VS Code | Latest | Development IDE |
| Node.js | 18+ | Teams AI Library (optional) |
| Docker | Latest | Container deployment (optional) |

### Azure Resources
| Resource | SKU | Purpose |
|---|---|---|
| Azure AI Foundry Project | Standard | Agent hosting and management |
| Azure AI Search | Standard (S1) | RAG index |
| Azure OpenAI Service | Standard | GPT-4o model deployment |
| Azure Functions | Consumption/Premium | Agent API hosting |
| API Management | Developer/Basic | API gateway |
| Azure App Service | B1+ | FastAPI hosting (alternative) |

### Python Packages
```
azure-ai-projects>=1.0.0
azure-ai-evaluation>=1.0.0
azure-identity>=1.17.0
fastapi>=0.110.0
uvicorn>=0.29.0
httpx>=0.27.0
pytest>=8.0.0
pytest-asyncio>=0.23.0
```

### Accounts & Access
- Azure subscription with Contributor role
- Microsoft 365 developer tenant (for Teams testing)
- Copilot Studio license (included in M365)
- GitHub account (for CI/CD)

---

## 📊 Assessment Criteria

### Knowledge Check (Quiz)
- 10 multiple-choice questions covering all module topics
- Passing score: 80% (8/10)
- See `quiz/assessment.md`

### Lab Completion
- All 8 mini hack steps completed
- See `lab/hands-on-lab.md`

### Capstone Rubric

| Criterion | Points | Requirements |
|---|---|---|
| Agent Architecture | 20 | Agent uses RAG + 2+ custom tools |
| Copilot Studio Integration | 15 | Custom connector + topic configured |
| Teams Deployment | 15 | Bot accessible in Teams |
| API Design | 15 | APIM with auth + rate limiting |
| E2E Testing | 20 | 5+ passing E2E tests |
| Quality Scores | 15 | All metrics ≥ 4.0/5.0 |
| **Total** | **100** | **Pass: 70+** |

---

## 🔑 Key Terminology

| Term | Definition |
|---|---|
| **AI Agent** | An autonomous AI system that plans actions, uses tools, and maintains conversational state |
| **Agent Service** | Azure AI Foundry service for creating and managing AI agents |
| **Custom Connector** | A bridge in Copilot Studio that connects to external APIs |
| **Function Tool** | A callable function registered with an agent for performing actions |
| **ToolSet** | A collection of tools (RAG, functions, code interpreter) available to an agent |
| **Thread** | A conversation session that maintains context across multiple turns |
| **APIM** | Azure API Management — gateway service for API security and management |
| **E2E Testing** | End-to-end testing that validates complete system flows |
| **Groundedness** | Quality metric measuring how well responses are supported by source data |

---

## 🔗 Module Dependencies

```
Module 28-35 (RAG Pipeline)
        ↓
Module 36-38 (Security & Governance)
        ↓
Module 39-42 (Advanced Patterns)
        ↓
  ► Module 43 (Deploy & Integrate) ◄ You are here
        ↓
Module 44 (Performance & Cost)
        ↓
Module 45 (Final Capstone)
```

---

## 📖 Additional Resources

### Documentation
- [Azure AI Agent Service](https://learn.microsoft.com/azure/ai-services/agents/)
- [Microsoft Copilot Studio](https://learn.microsoft.com/microsoft-copilot-studio/)
- [Teams Platform](https://learn.microsoft.com/microsoftteams/platform/)
- [Azure API Management](https://learn.microsoft.com/azure/api-management/)
- [Azure AI Evaluation SDK](https://learn.microsoft.com/azure/ai-studio/how-to/develop/evaluate-sdk)

### Sample Code
- [Azure AI Foundry Samples](https://github.com/Azure-Samples/azure-ai-foundry-samples)
- [Teams AI Library Samples](https://github.com/microsoft/teams-ai)
- [Copilot Studio Samples](https://github.com/microsoft/CopilotStudioSamples)

---

## 🎓 Instructor Notes

### Pacing Guide
| Session | Duration | Notes |
|---|---|---|
| Session 1 | 90 min | Live coding recommended; have fallback code ready |
| Session 2 | 90 min | Need Teams dev tenant pre-configured |
| Session 3 | 60 min | APIM deployment takes ~20 min — start early |
| Session 4 | 90 min | Evaluation may need pre-deployed model endpoint |
| Session 5 | 120 min | Allow flexible time for demos |

### Common Issues
1. **APIM provisioning time** — Create APIM instance before the session starts
2. **Teams app approval** — Pre-approve the bot in the admin center for testing
3. **Copilot Studio licensing** — Verify all learners have access
4. **Model quota** — Ensure sufficient GPT-4o quota for concurrent agent usage
5. **Network connectivity** — Some corporate networks block WebSocket connections needed for streaming

### Differentiation
- **Accelerated learners:** Challenge them to add streaming responses and adaptive cards
- **Learners needing support:** Provide pre-built agent code; focus on integration steps
- **Group work:** Pair learners for the mini hack — one focuses on backend, one on integration

---

*Module 43 of 45 — Azure AI Foundry: Zero to Hero*
*ARC 9: Capstone & Mastery*
*© 2026 Azure AI Foundry Training Series*
