# Module 10 — Evaluation, Content Safety & Testing

## Training Guide

> **ARC 2 · Prompt Engineering & Models** | Azure AI Foundry: Zero to Hero  
> **Date:** April 2026 | **Duration:** 90 minutes (lecture + hands-on)

---

## Module Overview

Module 10 is the capstone of ARC 2. Learners transition from building AI applications to systematically measuring quality and enforcing safety. This module covers evaluation metrics, Azure AI Content Safety, red-teaming strategies, and automated evaluation pipelines.

## Learning Objectives

By the end of this module, learners will be able to:

1. Explain the five core evaluation metrics (groundedness, relevance, coherence, fluency, similarity) and when to use each.
2. Configure Azure AI Content Safety filters with appropriate severity thresholds.
3. Create custom blocklists to enforce domain-specific content policies.
4. Describe how Prompt Shield protects against jailbreak and prompt injection attacks.
5. Design and execute a basic red-teaming exercise.
6. Build an automated evaluation pipeline using the `azure-ai-evaluation` Python SDK.
7. Integrate evaluation gates into a CI/CD workflow.

## Prerequisites

- Completed Modules 1–9 (especially Modules 7–9 on model deployment and fine-tuning)
- Azure AI Foundry project with a deployed model
- Python 3.10+ environment
- Basic familiarity with JSONL data format

## Session Agenda

| Time | Activity | Type |
|------|----------|------|
| 0:00–0:10 | Introduction & motivation — why evaluation matters | Lecture |
| 0:10–0:30 | Evaluation metrics deep dive — groundedness, relevance, coherence, fluency, similarity | Lecture + Demo |
| 0:30–0:45 | Built-in evaluators in AI Foundry portal & SDK | Demo |
| 0:45–1:00 | Content Safety deep dive — categories, severity, filters, blocklists, Prompt Shield | Lecture + Demo |
| 1:00–1:10 | Red-teaming framework and best practices | Lecture |
| 1:10–1:20 | Automated evaluation pipelines & CI/CD integration | Lecture |
| 1:20–1:30 | Mini Hack: Run evaluation with safety checks | Hands-on |

## Instructor Notes

### Key Teaching Points

- **Groundedness vs. Relevance:** Emphasize that these are independent axes. A response can be grounded but irrelevant (answers a different question with correct data) or relevant but ungrounded (answers the right question with fabricated data).
- **Severity Levels:** Walk through real examples at each severity level so learners develop intuition for the scale (0 = Safe, 2 = Low, 4 = Medium, 6 = High).
- **Prompt Shield:** Demo both direct jailbreak detection and indirect prompt injection scenarios. Show what happens when Prompt Shield blocks an input.
- **Red-Teaming:** Frame this as a positive, collaborative activity — not about blaming the model, but about hardening the system.

### Common Questions

1. **"Do I need to run all five quality evaluators?"** — No. Choose evaluators based on your use case. RAG apps should always include groundedness. Chatbots should focus on coherence and relevance.
2. **"How often should I run evaluations?"** — At minimum on every model or prompt change. Ideally daily against production traffic samples.
3. **"Can content filters be customized per deployment?"** — Yes, each deployment can have its own filter configuration.

### Demo Environment Setup

1. Ensure a GPT-4o deployment is available for AI-assisted evaluators.
2. Prepare a sample JSONL dataset with 10-20 rows covering normal and adversarial inputs.
3. Pre-configure a content filter with Prompt Shield enabled for the live demo.

## Assessment

Learners complete a 10-question quiz (see `quiz/assessment.md`) and the Mini Hack exercise. Success criteria for the Mini Hack:

- All quality metrics average ≥ 4.0/5.0
- Zero safety violations in test data
- Content filter properly configured
- At least 5 red-team prompts tested and documented

## Materials

| File | Description |
|------|-------------|
| `blog.html` | Comprehensive blog-style module content |
| `slides/presentation.html` | Slide deck for instructor-led delivery |
| `lab/hands-on-lab.md` | Step-by-step hands-on lab guide |
| `quiz/assessment.md` | 10-question knowledge assessment |

## Connection to Curriculum

- **Builds on:** Module 9 (Fine-tuning) — learners evaluate the model they fine-tuned
- **Leads into:** Module 11 (ARC 3: Data & RAG) — evaluation skills transfer directly to RAG groundedness testing
- **Cross-references:** Module 3 (Responsible AI principles), Module 7 (Model deployment & safety configuration)
