# Module 41: Choose Your Capstone Project

## Training Guide

> **Series:** Azure AI Foundry — Zero to Hero (Module 41 of 45)
> **Arc:** ARC 9 · CAPSTONE & MASTERY
> **Date:** April 2026
> **Duration:** 90 minutes

---

## 🎯 Module Overview

Module 41 marks the beginning of the capstone arc — the culminating experience of the Zero to Hero curriculum. In this module, learners choose their capstone project, define scope, gather requirements, plan architecture, establish success criteria, and produce a formal project proposal. This foundational planning phase ensures a smooth, focused build in Modules 42–44.

### Learning Objectives

By the end of this module, learners will be able to:

1. **Select** a capstone project that demonstrates breadth and depth across AI Foundry skills
2. **Define** project scope using the Scope Fitness Test framework
3. **Gather** functional and non-functional requirements using structured user stories
4. **Design** an end-to-end architecture using AI Foundry services and integration patterns
5. **Establish** SMART success criteria with measurable evaluation metrics
6. **Produce** a complete project proposal and architecture diagram

### Prerequisites

- Completion of Modules 1–40 (or equivalent AI Foundry experience)
- Active Azure subscription with AI Foundry project access
- Familiarity with AI Foundry agents, evaluation, Azure AI Search, and responsible AI
- Python 3.10+ development environment

---

## 📋 Agenda

| Time | Topic | Activity |
|------|-------|----------|
| 0:00–0:10 | Welcome & Capstone Overview | Lecture |
| 0:10–0:25 | Project Ideas & Scope Definition | Guided Discussion |
| 0:25–0:40 | Requirements Gathering Framework | Workshop |
| 0:40–0:55 | Architecture Planning with AI Foundry | Lecture + Demo |
| 0:55–1:05 | Success Criteria & Evaluation Plan | Interactive Exercise |
| 1:05–1:15 | Project Templates & Starter Code | Walkthrough |
| 1:15–1:30 | Mini Hack: Write Your Proposal | Hands-On Activity |

---

## 📚 Core Content

### 1. Project Ideas & Scope Definition

#### Curated Project Ideas

| Project | Domain | Complexity | Key AI Foundry Services |
|---------|--------|------------|------------------------|
| Healthcare Triage Agent | Healthcare | Advanced | Agents, AI Search, Content Safety, Evaluation |
| Enterprise BI Copilot | Business Intelligence | Advanced | Agents, Code Interpreter, Multi-model Deployments |
| Intelligent Document Pipeline | Document Processing | Intermediate | Agents, AI Search, Azure Functions, Evaluation |
| E-Commerce Shopping Agent | Retail | Intermediate | Agents, AI Search, Function Tools, Monitoring |
| SOC Security Agent | Cybersecurity | Expert | Multi-agent, Event-driven, Azure Monitor, Content Safety |
| Multilingual Support Hub | Customer Service | Advanced | Agents, AI Services (Translation), AI Search, Evaluation |

#### Scope Fitness Test

Every capstone project must meet these minimum criteria:

- **≥ 3 AI Foundry services** used in the solution
- **Agent with tools** — at least one agent with functional tool integrations
- **Data integration** — at least one external data source connected
- **Responsible AI** — content safety enabled and evaluation pipeline configured
- **Monitoring** — basic logging and performance tracking implemented
- **Completable** in 3 module build sprints (Modules 42–44)

#### Minimum Viable Capstone (MVC)

Define the smallest end-to-end slice that demonstrates value:

1. One functional agent with system prompt and at least one tool
2. One data source indexed in Azure AI Search
3. One evaluation run with baseline metrics captured
4. Content safety filter enabled
5. Basic conversation flow demonstrated end-to-end

### 2. Requirements Gathering

#### Functional Requirements Framework

Use this template to document functional requirements:

```markdown
### User Stories

1. As a [role], I want to [action], so that [benefit].
2. As a [role], I want to [action], so that [benefit].
...

### Core Interactions
- What questions/requests will users make?
- What tools does the agent need to fulfill requests?
- What output formats are expected?

### Multi-Turn Requirements
- Does the agent need session memory?
- How should context carry across turns?
- When should the agent escalate to a human?
```

#### Non-Functional Requirements

| Requirement | Target | How to Measure |
|-------------|--------|----------------|
| Latency | < 3 seconds per response | Azure Monitor metrics |
| Throughput | 10 concurrent users | Load testing |
| Availability | 99.5% uptime | Azure Health checks |
| Security | Managed identity, no secrets in code | Code review |
| Cost | < $50/month Azure spend | Azure Cost Management |

### 3. Architecture Planning

#### Reference Architecture Components

```
┌──────────────┐     ┌───────────────────┐     ┌──────────────┐
│  User        │────▶│  AI Foundry Agent  │────▶│  Tools &     │
│  Interface   │     │  (gpt-4o)         │     │  Functions   │
└──────────────┘     └─────────┬─────────┘     └──────────────┘
                               │
                    ┌──────────┼──────────┐
                    ▼          ▼          ▼
              ┌──────────┐ ┌────────┐ ┌───────────┐
              │ Azure AI │ │Content │ │ Model     │
              │ Search   │ │Safety  │ │Deployments│
              └──────────┘ └────────┘ └───────────┘
                               │
                    ┌──────────┼──────────┐
                    ▼          ▼          ▼
              ┌──────────┐ ┌────────┐ ┌───────────┐
              │Monitoring│ │ Eval   │ │  CI/CD    │
              │& Tracing │ │Pipeline│ │           │
              └──────────┘ └────────┘ └───────────┘
```

#### Architectural Decision Records (ADRs)

Document each key decision using this format:

```markdown
### ADR-001: Agent Topology

**Status:** Proposed
**Context:** The project requires [context]
**Decision:** We will use [decision]
**Consequences:** [positive and negative impacts]
```

Key decisions to document:
1. **Agent topology** — single vs. multi-agent
2. **Knowledge strategy** — RAG vs. fine-tuning vs. hybrid
3. **Orchestration pattern** — sequential, parallel, or event-driven
4. **Deployment model** — AI Foundry hosted vs. containerized
5. **Evaluation approach** — batch, online, human-in-the-loop, or combined

### 4. Success Criteria & Evaluation Plan

#### SMART Criteria Template

```markdown
1. **Specific:** The agent answers [X]% of test queries correctly
2. **Measurable:** Groundedness score ≥ 4.0/5.0 on AI Foundry evaluation
3. **Achievable:** Completable within Modules 42–44 timeframe
4. **Relevant:** Solves [specific problem] for [target user]
5. **Time-bound:** MVP by Module 43, final by Module 44
```

#### Evaluation Plan Phases

| Phase | Module | Activities | Deliverables |
|-------|--------|------------|-------------|
| Baseline | 42 | Create test dataset, run initial evaluation | Baseline metrics report |
| Iteration | 43 | Optimize prompts, tune retrieval, refine tools | Iteration comparison report |
| Final | 44 | Comprehensive evaluation with red-teaming | Final evaluation report |
| Demo | 44 | Live presentation with Q&A | Presentation + demo recording |

### 5. Project Templates & Starter Code

#### Recommended Project Structure

```
capstone-project/
├── README.md                   # Project overview & setup
├── PROPOSAL.md                 # Capstone proposal
├── agent.yaml                  # AI Foundry agent definition
├── requirements.txt            # Python dependencies
├── src/
│   ├── agent.py                # Agent creation & configuration
│   ├── tools.py                # Custom tool functions
│   ├── prompts.py              # Prompt templates
│   └── utils.py                # Helpers
├── data/
│   ├── knowledge-base/         # RAG documents
│   ├── eval-datasets/          # Test cases
│   └── sample-inputs/          # Demo data
├── eval/
│   ├── evaluators.py           # Custom evaluators
│   ├── run_eval.py             # Batch evaluation
│   └── results/                # Output
├── infra/
│   ├── main.bicep              # Azure IaC
│   └── parameters.json         # Config
├── tests/
│   ├── test_agent.py           # Integration tests
│   └── test_tools.py           # Unit tests
└── docs/
    ├── architecture.md         # Architecture diagram
    └── evaluation-report.md    # Final results
```

#### Starter Dependencies

```txt
# requirements.txt
azure-ai-projects>=1.0.0
azure-ai-evaluation>=1.0.0
azure-ai-agents>=1.0.0
azure-identity>=1.17.0
azure-search-documents>=11.6.0
python-dotenv>=1.0.0
```

---

## 🧪 Mini Hack: Write Your Project Proposal

### Objective
Produce a complete capstone proposal document and architecture diagram.

### Time: 30 minutes

### Steps

1. **Choose your project** (5 min) — Select from curated ideas or define your own
2. **Write user stories** (5 min) — Document 5–8 user stories
3. **Design architecture** (10 min) — Draw a diagram with all AI Foundry components
4. **Define success criteria** (5 min) — Write 3–5 SMART criteria
5. **Complete PROPOSAL.md** (5 min) — Fill in all sections of the template

### Deliverables

- [ ] Completed `PROPOSAL.md` with all sections filled
- [ ] Architecture diagram showing AI Foundry components and data flows
- [ ] 5+ user stories documented
- [ ] 3+ SMART success criteria with target evaluation metrics
- [ ] Risk mitigation table with 3+ identified risks

---

## 📊 Evaluation Rubric

| Category | Weight | Excellent (5) | Good (4) | Adequate (3) | Needs Work (2) |
|----------|--------|--------------|----------|--------------|----------------|
| Technical Depth | 30% | 5+ services, custom integrations | 4 services, some customization | 3 services, default configs | < 3 services |
| Agent Design | 20% | Multi-agent, robust error handling | Single agent, good tool usage | Basic agent, minimal tools | Incomplete agent |
| Evaluation Quality | 20% | Custom metrics, red-teaming | Built-in metrics, good dataset | Basic evaluation run | No evaluation |
| Responsible AI | 15% | Full RAI analysis documented | Content safety + basic testing | Content filter only | No RAI measures |
| Presentation | 15% | Clear demo, strong Q&A | Good demo, adequate Q&A | Basic walkthrough | Incomplete demo |

---

## 📌 Key Takeaways

1. Your capstone demonstrates **end-to-end mastery** of Azure AI Foundry
2. **Scope ruthlessly** — define your MVC before building
3. **Requirements first** — user stories and NFRs prevent rework
4. **Architecture decisions** documented early save time later
5. **SMART criteria** with evaluation metrics ensure measurable outcomes
6. **Templates accelerate** — use the provided starter code and structure
7. The **evaluation rubric** is your guide throughout Modules 42–44

---

## 📖 Additional Resources

- [Azure AI Foundry Documentation](https://learn.microsoft.com/azure/ai-studio/)
- [AI Foundry Agent Service Overview](https://learn.microsoft.com/azure/ai-services/agents/)
- [Azure AI Evaluation SDK](https://learn.microsoft.com/azure/ai-studio/how-to/evaluate-sdk)
- [Responsible AI Practices](https://learn.microsoft.com/azure/ai-services/responsible-use-of-ai-overview)
- [Featured Video: Microsoft Foundry – AI App & Agent Factory](https://www.youtube.com/watch?v=C6rxEGJay70)

---

## ➡️ Next Module

**Module 42: Build Your Capstone — Part 1** — Set up infrastructure, create your agent, build the first end-to-end flow, and establish baseline evaluation metrics.
