# Module 7 — Training Guide: The Model Catalog Deep Dive

> **Series:** Azure AI Foundry: Zero to Hero (Module 7 of 45)  
> **Arc:** ARC 2 — Prompt Engineering & Models  
> **Duration:** 90 minutes (lecture 45 min · lab 45 min)  
> **Date:** April 2026

---

## Learning Objectives

By the end of this module, learners will be able to:

1. Navigate the Azure AI Foundry Model Catalog and use filters to discover models
2. Identify major model families (OpenAI, Meta Llama, Mistral, Phi, Cohere) and their strengths
3. Explain the three deployment types: Serverless API, Managed Compute, and Provisioned Throughput
4. Read and interpret Model Cards, including benchmarks, licensing, and safety information
5. Apply a structured decision framework to select the right model for a given use case
6. Use the AI Foundry Playground to compare multiple models on the same task

---

## Prerequisites

- Completed Module 6 (AI Foundry Portal Walkthrough)
- Active Azure AI Foundry project with Contributor access
- Web browser with access to [ai.azure.com](https://ai.azure.com)

---

## Instructor Agenda

| Time | Activity | Materials |
|------|----------|-----------|
| 0:00 – 0:10 | Welcome & Module 6 recap | Slides 1–3 |
| 0:10 – 0:25 | Model Catalog overview & live demo | Slides 4–6, Portal |
| 0:25 – 0:40 | Model families deep dive | Slides 7–9 |
| 0:40 – 0:55 | Deployment types & model cards | Slides 10–12 |
| 0:55 – 1:05 | Decision framework & cost discussion | Slides 13–14 |
| 1:05 – 1:10 | Mini Hack briefing | Slide 15 |
| 1:10 – 1:25 | Hands-on lab (guided) | Lab guide |
| 1:25 – 1:30 | Wrap-up, quiz, Module 8 preview | Assessment |

---

## Key Talking Points

### The Model Catalog
- 1,800+ models from 25+ providers — largest curated AI model directory
- Emphasize this is *not* a raw model zoo — every model is tested, documented, and deployable
- Show the filtering UI: task type, provider, license, fine-tuning support

### Model Families
- **OpenAI:** Frontier quality, proprietary, Serverless API deployment
- **Meta Llama:** Best open-weight models, Managed Compute deployment, fine-tunable
- **Mistral:** European AI lab, strong multilingual, Mixture of Experts efficiency
- **Phi (Microsoft):** Small models with outsized performance, ideal for edge/on-prem
- **Cohere:** Enterprise RAG specialist, leading embeddings and reranking models

### Deployment Types
- Serverless API → fastest start, pay-per-token, no infra management
- Managed Compute → full control, your VMs, required for open-weight models
- Provisioned Throughput → reserved capacity, predictable costs, production workloads

### Choosing the Right Model
- Walk through the 6-step framework: Task → Quality → Latency → Budget → Compliance → Test
- Emphasize: *benchmarks are useful but not sufficient* — always test with real data
- Introduce the cascade strategy (route by complexity) for cost optimization

---

## Common Questions (FAQ)

**Q: Can I fine-tune models in AI Foundry?**  
A: Yes — open-weight models (Llama, Phi, Mistral) deployed on Managed Compute support fine-tuning. Proprietary models have limited customization options.

**Q: Do my prompts/data get shared with model providers?**  
A: No. All API traffic stays within Azure's infrastructure. Your data is not used to train models.

**Q: How do I compare models systematically?**  
A: Deploy 2–3 candidates, use the Playground with identical prompts, and score responses across accuracy, latency, and cost dimensions.

**Q: What if a model I need isn't in the catalog?**  
A: You can bring your own model via custom containers on Managed Compute, or request new models through the AI Foundry feedback channel.

---

## Assessment Strategy

- **Quiz:** 10-question assessment covering catalog navigation, model families, deployment types, and decision-making (see `quiz/assessment.md`)
- **Mini Hack:** Compare 3 models in the Playground (see `lab/hands-on-lab.md`)
- **Discussion:** Which model would you choose for [given scenario] and why?

---

## Resources for Instructors

- [Model Catalog Documentation](https://learn.microsoft.com/en-us/azure/ai-studio/how-to/model-catalog-overview)
- [Deployment Options Overview](https://learn.microsoft.com/en-us/azure/ai-studio/concepts/deployments-overview)
- [Model Benchmarks](https://learn.microsoft.com/en-us/azure/ai-studio/how-to/model-benchmarks)
- [Azure Pricing Calculator](https://azure.microsoft.com/en-us/pricing/calculator/)
- [Video: Model Deployment](https://www.youtube.com/watch?v=LLO0lOufbwc)

---

*Next Module → Module 8: Your First Model Deployment*
