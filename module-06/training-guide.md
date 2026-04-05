# Module 6 — Prompt Engineering Fundamentals

## Training Guide

**Course:** Azure AI Foundry — Zero to Hero (Module 6 of 45)  
**Arc:** ARC 2 · Prompt Engineering & Models  
**Duration:** 90 minutes (lecture + hands-on)  
**Date:** April 2026  

---

## Learning Objectives

By the end of this module, learners will be able to:

1. **Explain** the role of system, user, and assistant messages in the chat-completion API.
2. **Differentiate** between zero-shot, few-shot, and chain-of-thought prompting techniques.
3. **Design** reusable prompt templates with variable placeholders.
4. **Tune** generation parameters — temperature, top_p, max_tokens, frequency_penalty, and presence_penalty — for specific use cases.
5. **Apply** best practices and avoid common anti-patterns when crafting prompts.
6. **Build** a working prompt in the Azure AI Foundry Playground that classifies customer feedback into predefined categories.

---

## Key Concepts Summary

### 1. Message Roles

| Role | Purpose | Example |
|------|---------|---------|
| **system** | Sets the model's persona, constraints, and output format | "You are a senior support agent. Reply in JSON." |
| **user** | Contains the end-user's request or data | "Classify this review: 'Battery dies too fast.'" |
| **assistant** | Prior model responses (used for context continuity or few-shot examples) | `{"category": "hardware", "sentiment": "negative"}` |

### 2. Prompting Techniques

- **Zero-shot** — No examples provided; the model infers the task from instructions alone. Best for straightforward classification or Q&A.
- **Few-shot** — 2–5 input/output examples prepended to the prompt. Dramatically improves consistency for formatting or domain-specific tasks.
- **Chain-of-thought (CoT)** — Instruct the model to "think step by step." Improves accuracy on multi-step reasoning, math, and logic tasks.

### 3. Prompt Templates

Prompt templates separate *logic* from *data*. Use placeholders like `{{feedback_text}}` and `{{categories}}` so the same prompt works across different inputs. In AI Foundry, you can save and version templates inside Prompt Flow.

### 4. Generation Parameters

| Parameter | Range | Low Value Effect | High Value Effect |
|-----------|-------|------------------|-------------------|
| **temperature** | 0 – 2 | Deterministic, focused | Creative, varied |
| **top_p** | 0 – 1 | Narrow vocabulary | Broad vocabulary |
| **max_tokens** | 1 – model limit | Short response | Long response |
| **frequency_penalty** | -2 – 2 | Allows repetition | Reduces repetition |
| **presence_penalty** | -2 – 2 | Stays on topic | Encourages new topics |

> **Rule of thumb:** Adjust *either* temperature *or* top_p, not both at once.

---

## Step-by-Step Instructions

### Part A — Explore Message Roles (15 min)

1. Open the [Azure AI Foundry Portal](https://ai.azure.com).
2. Navigate to a project, then open the **Playground → Chat**.
3. In the **System message** panel, enter:
   ```
   You are a concise technical support agent. Always respond with a JSON object containing "issue_type" and "suggested_action".
   ```
4. Send a user message: `"My laptop screen is flickering after the latest update."`
5. Observe how the system prompt shapes the response format and tone.
6. Edit the system prompt to change the persona (e.g., friendly retail assistant) and resend the same user message. Compare results.

### Part B — Zero-Shot vs Few-Shot (20 min)

1. Clear the conversation. Set a neutral system prompt.
2. **Zero-shot test:** Send `"Classify the sentiment of: 'Absolutely love the new design!'"` — note the response.
3. **Few-shot test:** Add 3 assistant-turn examples before your question:
   - User: "The packaging was damaged." → Assistant: "negative"
   - User: "Arrived on time, works great." → Assistant: "positive"
   - User: "It's okay, nothing special." → Assistant: "neutral"
   - User: "Absolutely love the new design!" → ?
4. Compare consistency and formatting between both approaches.

### Part C — Chain-of-Thought (15 min)

1. Ask a multi-step math or logic question *without* CoT instruction.
2. Append `"Let's think step by step."` to the same prompt.
3. Compare accuracy and observe how reasoning traces improve output.

### Part D — Parameter Exploration (15 min)

1. In the Playground, locate the **Parameters** panel (right side).
2. Set temperature to **0.0** and send a creative-writing prompt 3 times. Note identical outputs.
3. Increase temperature to **1.2** and repeat. Observe variation.
4. Reset temperature to **0.7** and experiment with top_p values of **0.1** vs **0.95**.

### Part E — Mini Hack: Customer Feedback Classifier (25 min)

Follow the full instructions in `lab/hands-on-lab.md`.

---

## Practice Exercises

1. **Template Design:** Create a reusable prompt template that summarises a news article in 3 bullet points. Use `{{article_text}}` and `{{language}}` placeholders.
2. **Parameter Tuning:** For a code-generation task, recommend and justify appropriate values for temperature, top_p, and max_tokens.
3. **Anti-Pattern Fix:** Given the prompt `"Do stuff with this data: [blob of CSV]"`, rewrite it following best practices (clear instruction, output format, constraints).
4. **CoT Application:** Write a chain-of-thought prompt that helps the model calculate compound interest for a savings account.

---

## Preparation for Module 7

Module 7 covers **Advanced Prompt Engineering Patterns** — including meta-prompting, self-consistency, prompt chaining, and retrieval-augmented generation (RAG) prompt design. Review the outputs you created in this module; you will refine them in the next session.

---

*Azure AI Foundry — Zero to Hero Training Team · April 2026*
