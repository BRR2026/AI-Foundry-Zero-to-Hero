# Module 9 — Fine-Tuning Models with Your Data

## Training Guide

**Course:** Azure AI Foundry: Zero to Hero (Module 9 of 45)
**Arc:** ARC 2 — Prompt Engineering & Models
**Duration:** 90 minutes (lecture + hands-on lab)
**Date:** April 2026

---

## Learning Objectives

By the end of this module, learners will be able to:

1. **Differentiate** when to use prompt engineering, RAG, and fine-tuning based on use case requirements
2. **Prepare** training data in JSONL chat format with proper structure and quality
3. **Execute** a complete fine-tuning workflow in Azure AI Foundry
4. **Configure** hyperparameters (epochs, learning rate, batch size) for optimal results
5. **Interpret** training and validation loss curves to diagnose model health
6. **Validate** fine-tuned model performance against a held-out test set
7. **Deploy** and call a fine-tuned model via the Azure OpenAI API

---

## Prerequisites

- Completed Modules 1–8 (especially Module 6: Prompt Engineering Fundamentals)
- Access to an Azure AI Foundry project with a connected Azure OpenAI resource
- Azure OpenAI resource in a fine-tuning-supported region (East US2, North Central US, Sweden Central, or Switzerland West)
- Python 3.10+ with the `openai` SDK installed
- Basic understanding of JSON format

---

## Session Outline

### Part 1: Concepts (30 minutes)

| Time | Topic | Method |
|------|-------|--------|
| 0:00–0:05 | Welcome & module overview | Lecture |
| 0:05–0:15 | Decision framework: Prompts vs. RAG vs. Fine-tuning | Interactive discussion |
| 0:15–0:25 | Training data preparation: JSONL format, quality guidelines | Lecture + examples |
| 0:25–0:30 | Supported models and region availability | Reference walkthrough |

### Part 2: Workflow Deep Dive (25 minutes)

| Time | Topic | Method |
|------|-------|--------|
| 0:30–0:40 | Fine-tuning workflow walkthrough in AI Foundry | Live demo |
| 0:40–0:48 | Hyperparameters: epochs, learning rate, batch size | Lecture + visuals |
| 0:48–0:55 | Monitoring training: reading loss curves, diagnosing issues | Lecture + examples |

### Part 3: Hands-On Lab (30 minutes)

| Time | Topic | Method |
|------|-------|--------|
| 0:55–1:05 | Lab setup: create JSONL training & validation files | Guided exercise |
| 1:05–1:15 | Submit fine-tuning job in AI Foundry | Hands-on |
| 1:15–1:20 | Deploy and test in Playground | Hands-on |
| 1:20–1:25 | Evaluate: compare fine-tuned vs. base model | Guided exercise |

### Wrap-Up (5 minutes)

| Time | Topic | Method |
|------|-------|--------|
| 1:25–1:30 | Key takeaways, Q&A, Module 10 preview | Discussion |

---

## Key Concepts

### The Decision Framework

| Approach | Use When | Pros | Cons |
|----------|----------|------|------|
| **Prompt Engineering** | Quick adjustments, format changes | Free, fast, no training | Limited by context window |
| **RAG** | Dynamic/changing data, citations needed | Up-to-date, traceable | Retrieval latency, complexity |
| **Fine-Tuning** | Domain expertise, consistent style/tone | Reduced latency, better quality | Cost, requires training data |

### JSONL Format

Each line is a JSON object with a `messages` array containing `system`, `user`, and `assistant` roles:

```json
{"messages": [{"role": "system", "content": "..."}, {"role": "user", "content": "..."}, {"role": "assistant", "content": "..."}]}
```

### Hyperparameters

- **n_epochs** — Number of passes through training data (default: auto, typically 3–5)
- **learning_rate_multiplier** — Scales the learning rate (default: auto)
- **batch_size** — Examples per training step (default: auto)
- **seed** — For reproducibility

---

## Instructor Notes

1. **Pre-class setup:** Ensure all learners have an AI Foundry project connected to an Azure OpenAI resource in a supported region. Fine-tuning jobs take 15–30 minutes, so consider pre-submitting a job to show completed results.

2. **Demo strategy:** Walk through the portal UI step-by-step. Show the data validation screen, hyperparameter configuration, and the real-time loss curves.

3. **Common questions:**
   - *"Can I fine-tune with my own on-premises data?"* — Yes, as long as you upload it as JSONL. The data is used only for training and follows Azure's data processing policies.
   - *"How is this different from Azure Machine Learning?"* — AI Foundry provides a managed fine-tuning experience specifically for foundation models. It handles all GPU infrastructure automatically.
   - *"Can I combine fine-tuning and RAG?"* — Absolutely. Fine-tune for style/tone, RAG for grounding in current data.

4. **Assessment:** Use the Module 9 quiz (10 questions) to verify understanding of key concepts.

---

## Required Materials

- [ ] Slide deck: `slides/presentation.html`
- [ ] Hands-on lab guide: `lab/hands-on-lab.md`
- [ ] Assessment quiz: `quiz/assessment.md`
- [ ] Sample JSONL training data (provided in blog post and lab)
- [ ] Blog post: `blog.html`

---

## Featured Videos

1. **Fine-Tuning Models in Microsoft Foundry** — [YouTube](https://www.youtube.com/watch?v=NbResO8QcSM)
2. **AI Fine-Tuning for Unstoppable Agents (BRK188)** — [YouTube](https://www.youtube.com/watch?v=dXUy9evg1yo)

---

## Connection to Curriculum

- **Previous:** Module 8 covered advanced prompt engineering techniques and parameter tuning
- **This module:** Bridges the gap between prompt engineering and model customization
- **Next:** Module 10 introduces content safety and responsible AI guardrails
- **Arc context:** Completes the "customization spectrum" — from prompt engineering → fine-tuning → responsible deployment
