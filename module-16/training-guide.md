# Module 16: Intro to AI Agents & Agent Framework

## Training Guide

**Course:** Azure AI Foundry — Zero to Hero (45-Module Series)  
**Arc:** 4 — AI Agents  
**Tier:** 🟡 Intermediate (First Intermediate Module!)  
**Duration:** 90 minutes (lecture + lab)  
**Date:** April 2026  
**Author:** Zero to Hero Training Team

---

## Learning Objectives

By the end of this module, learners will be able to:

1. Define what AI agents are and articulate how they differ from traditional chatbots
2. Describe the Azure AI Agent Service and its role within Microsoft Foundry
3. Explain the four pillars of agent architecture: model, tools, memory, and instructions
4. Differentiate between prompt agents and hosted agents and identify when to use each
5. Create a prompt agent using the Azure AI Foundry portal with built-in tools

---

## Prerequisites

- Completed Modules 1–15 (Beginner tier)
- Active Azure subscription with an AI Foundry project
- At least one deployed chat model (GPT-4o recommended)
- Familiarity with the Azure AI Foundry portal navigation

---

## Module Outline

### Part 1: Concepts (40 minutes)

| Time | Topic | Resources |
|------|-------|-----------|
| 0:00–0:10 | Welcome to Intermediate tier; what changes | Celebration callout, tier overview |
| 0:10–0:20 | What are AI agents? Evolution from prompts to agents | Blog Section 1, slides 1–4 |
| 0:20–0:30 | Agents vs. chatbots with concrete examples | Blog Section 2, comparison table |
| 0:30–0:40 | Azure AI Agent Service overview and ecosystem fit | Blog Section 3, slides 5–7 |

### Part 2: Architecture & Types (30 minutes)

| Time | Topic | Resources |
|------|-------|-----------|
| 0:40–0:55 | Agent architecture deep dive (4 pillars) | Blog Section 4, architecture diagram |
| 0:55–1:05 | Prompt agents vs. hosted agents | Blog Sections 5–8, decision table |
| 1:05–1:10 | Security considerations for autonomous agents | Blog Section 9 |

### Part 3: Hands-On (20 minutes)

| Time | Topic | Resources |
|------|-------|-----------|
| 1:10–1:25 | Mini Hack: Create your first prompt agent | Lab guide, portal walkthrough |
| 1:25–1:30 | Wrap-up, key takeaways, preview of Module 17 | Summary slide |

---

## Key Terminology

| Term | Definition |
|------|-----------|
| **AI Agent** | An autonomous system that uses an LLM to reason, plan, use tools, and take actions toward a goal |
| **Chatbot** | A reactive conversational system that responds to user prompts without autonomous action |
| **Azure AI Agent Service** | Managed platform in Microsoft Foundry for building, deploying, and scaling AI agents |
| **Prompt Agent** | A no-code agent configured via portal/API using built-in tools and instructions |
| **Hosted Agent** | A code-based agent using custom Python/containers deployed on managed infrastructure |
| **Thread** | A conversation session that stores messages, file attachments, and tool execution results |
| **Instructions** | The system prompt defining an agent's persona, goals, constraints, and tool guidance |
| **Tool Calling** | The mechanism by which an LLM decides to invoke external tools during its reasoning loop |

---

## Video Resources

1. **Microsoft Foundry — AI App & Agent Factory** (MS Mechanics)  
   https://www.youtube.com/watch?v=C6rxEGJay70

2. **Build An AI Agent From Scratch Using Microsoft Foundry** (Azure Innovation Station)  
   https://www.youtube.com/watch?v=wL8H0RySf6I

---

## Instructor Notes

- **Celebrate the transition!** This is the first Intermediate module — acknowledge learners' progress through the entire Beginner tier (Modules 1–15).
- **Emphasize the paradigm shift**: agents are not "better chatbots" — they represent a fundamentally different approach with autonomous decision-making.
- **Live demo**: Walk through creating a prompt agent in the portal during the lecture. Don't wait until the lab.
- **Security first**: Even in a demo, show how to set constraints in instructions. Agents without guardrails are a liability.
- **Common misconception**: Students may think they need hosted agents for everything. Emphasize that prompt agents are powerful and sufficient for many production use cases.
- **Connect to prior learning**: Reference RAG patterns from Modules 12–14 — File Search in agents uses the same Azure AI Search underneath.

---

## Assessment

- Complete the quiz (`quiz/assessment.md`) — 10 multiple-choice questions
- Complete the hands-on lab (`lab/hands-on-lab.md`) — create and test a prompt agent
- Bonus: Create a second agent with different instructions and compare behavior

---

## Additional Resources

- [Azure AI Agent Service Documentation](https://learn.microsoft.com/azure/ai-services/agents/)
- [Microsoft Foundry Documentation](https://learn.microsoft.com/azure/ai-foundry/)
- [Agent Architecture Patterns](https://learn.microsoft.com/azure/ai-services/agents/concepts/agents)
- [Prompt Agent Quickstart](https://learn.microsoft.com/azure/ai-services/agents/quickstart)

---

*Next Module: Module 17 — Building Agents with Tools (File Search, Code Interpreter, Bing Grounding)*
