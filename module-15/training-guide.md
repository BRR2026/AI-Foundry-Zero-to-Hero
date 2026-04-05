# Module 15 — Training Guide
## Evaluation Datasets & Quality Measurement
**ARC 3: DATA & RAG** · 🟢 Beginner Tier (Final Module) · April 2026

---

## Overview

Module 15 is the capstone of the Beginner tier. Learners will understand how to systematically measure AI quality using evaluation datasets, automated pipelines, and the `azure-ai-evaluation` SDK in Azure AI Foundry. By the end of this module, participants can build golden datasets, run automated evaluations, and implement regression detection gates.

**Duration:** 3 hours (1.5h lecture + 1.5h hands-on lab)
**Prerequisites:** Modules 1–14 (ARC 1–3 foundations, prompt engineering, RAG basics)

---

## Learning Objectives

By completing this module, learners will be able to:

1. **Design** a golden evaluation dataset with expert-curated ground truth
2. **Implement** automated evaluation pipelines using the `azure-ai-evaluation` SDK
3. **Apply** LLM-as-a-Judge patterns for nuanced quality assessment
4. **Track** quality metrics over time and detect regressions
5. **Integrate** evaluation gates into CI/CD pipelines to prevent quality regressions

---

## Agenda

| Time | Topic | Format |
|------|-------|--------|
| 0:00–0:10 | Welcome & module overview | Lecture |
| 0:10–0:30 | Why evaluation datasets matter | Lecture + discussion |
| 0:30–0:50 | Building golden datasets & ground truth | Lecture + demo |
| 0:50–1:05 | Automated evaluation pipelines | Live demo |
| 1:05–1:20 | LLM-as-a-Judge patterns | Lecture + code walkthrough |
| 1:20–1:30 | Quality tracking & regression detection | Lecture |
| 1:30–3:00 | **Mini Hack:** Create eval dataset & run benchmark | Hands-on lab |

---

## Key Concepts

### Golden Datasets
- Curated input/output pairs that define expected AI behavior
- Sources: real user conversations, SME-crafted questions, support tickets, adversarial tests
- Structure: query, context, ground_truth, category, difficulty

### Ground Truth Types
- Exact match, semantic equivalence, key-point coverage, rubric-based, constraint-based

### Built-in Evaluators (azure-ai-evaluation)
- **GroundednessEvaluator** — Are claims supported by context?
- **RelevanceEvaluator** — Does the answer address the query?
- **CoherenceEvaluator** — Is the response well-organized?
- **FluencyEvaluator** — Is the language natural?
- **SimilarityEvaluator** — How close to ground truth? (requires ground truth)
- **F1ScoreEvaluator** — Token-level overlap (requires ground truth)

### LLM-as-a-Judge
- Uses a stronger model to evaluate another model's output
- Requires calibration against human labels
- Watch for biases: self-evaluation, verbosity, position bias

---

## Instructor Notes

1. **Start with a failing example:** Show a RAG answer that looks fine but contains a factual error. Emphasize why automated evaluation catches what humans miss.
2. **Live demo:** Run the `evaluate()` function against a prepared dataset in real-time. Show results appearing in the AI Foundry portal.
3. **Discussion prompt:** "How would you know if your AI got worse after a model update?" — lead into regression detection.
4. **Celebrate the milestone:** This is the last Beginner module. Acknowledge progress and preview the Intermediate tier.

---

## Featured Video

**"Which LLM??? LLM Evaluation in Azure AI Foundry"**
- YouTube: https://www.youtube.com/watch?v=BFY6czQykMU
- Use as pre-work or play during the evaluation pipelines section
- Key takeaways: model comparison, evaluation runs, built-in evaluators

---

## Assessment

- **Quiz:** 10 multiple-choice questions (see `quiz/assessment.md`)
- **Mini Hack:** Create a 20-example eval dataset, run 4 evaluators, identify regressions
- **Completion criteria:** Quiz score ≥ 80%, Mini Hack success criteria met

---

## Materials Checklist

- [ ] Slides: `slides/presentation.html`
- [ ] Blog post: `blog.html`
- [ ] Hands-on lab: `lab/hands-on-lab.md`
- [ ] Quiz: `quiz/assessment.md`
- [ ] Sample dataset: prepared JSONL with 20 examples
- [ ] Azure AI Foundry project with deployed model for evaluation
- [ ] Pre-installed: `pip install azure-ai-evaluation azure-identity`

---

## Transition to Module 16

Emphasize that evaluation is the bridge between the Beginner and Intermediate tiers. Students now have the tools to *measure* the impact of everything they'll learn next. Module 16 opens ARC 4: AI Agents — where they'll build autonomous systems and use evaluation to ensure those agents behave correctly.

> **🟡 Intermediate Tier begins with Module 16!**
