# Module 7 — Assessment: The Model Catalog Deep Dive

> **Format:** 10 multiple-choice questions  
> **Passing Score:** 80% (8 of 10 correct)  
> **Time Limit:** 15 minutes

---

## Questions

### Q1. How many models are available in the Azure AI Foundry Model Catalog (approximate)?

- A) 200+
- B) 500+
- C) 1,800+
- D) 10,000+

**Correct Answer:** C  
**Explanation:** The Azure AI Foundry Model Catalog provides access to over 1,800 models from 25+ providers, including OpenAI, Meta, Mistral, Microsoft, Cohere, and others.

---

### Q2. Which model family is known for open-weight models that can be fine-tuned on Managed Compute in AI Foundry?

- A) OpenAI GPT
- B) Meta Llama
- C) Cohere Command
- D) OpenAI o-series

**Correct Answer:** B  
**Explanation:** Meta's Llama models are open-weight and can be deployed on Managed Compute in AI Foundry, giving users full control including the ability to fine-tune the models.

---

### Q3. What are the three deployment types available in Azure AI Foundry?

- A) Serverless API, Managed Compute, Provisioned Throughput
- B) Serverless API, Container Instances, Virtual Machines
- C) Pay-as-you-go, Reserved Instances, Spot Instances
- D) Standard, Premium, Enterprise

**Correct Answer:** A  
**Explanation:** AI Foundry offers three deployment options: Serverless API (pay-per-token), Managed Compute (dedicated VMs), and Provisioned Throughput (reserved capacity with PTUs).

---

### Q4. Which deployment type is recommended for getting started quickly with minimal infrastructure management?

- A) Managed Compute
- B) Provisioned Throughput
- C) Serverless API
- D) Custom Containers

**Correct Answer:** C  
**Explanation:** Serverless API deployments require no infrastructure management, can be set up in minutes, and use pay-per-token pricing — making them ideal for prototyping and getting started.

---

### Q5. What is a Model Card in Azure AI Foundry?

- A) A credit card linked to your Azure subscription for model billing
- B) A standardized document with model details including benchmarks, capabilities, licensing, and safety info
- C) A physical card shipped to users with model access credentials
- D) A dashboard showing real-time model usage metrics

**Correct Answer:** B  
**Explanation:** A Model Card is a standardized document — like a "nutrition label" for AI models — that includes the model's overview, benchmarks, capabilities, licensing terms, and safety considerations.

---

### Q6. Which benchmark specifically measures a model's ability to generate correct Python code?

- A) MMLU
- B) HellaSwag
- C) HumanEval
- D) GSM8K

**Correct Answer:** C  
**Explanation:** HumanEval is a benchmark that measures code generation accuracy by testing whether models can produce correct Python functions. MMLU measures multitask knowledge, HellaSwag measures commonsense reasoning, and GSM8K measures math reasoning.

---

### Q7. Microsoft's Phi-4 model is best described as:

- A) A large language model with 400B+ parameters for enterprise use
- B) A small language model (14B params) with strong performance relative to its size, ideal for edge/on-prem
- C) An image generation model similar to DALL·E
- D) A proprietary model available only through Serverless API

**Correct Answer:** B  
**Explanation:** Phi-4 is a 14B parameter Small Language Model (SLM) from Microsoft that punches above its weight, excelling at math, coding, and structured reasoning. It is MIT-licensed and ideal for resource-constrained or edge deployments.

---

### Q8. What is the "model cascade" cost optimization strategy?

- A) Deploying all models in the catalog simultaneously
- B) Routing simple queries to cheap/fast models and only escalating complex queries to premium models
- C) Running models in a waterfall sequence where each model refines the previous output
- D) Gradually increasing model size over time as usage grows

**Correct Answer:** B  
**Explanation:** A model cascade routes incoming queries based on complexity — simple queries go to inexpensive models (e.g., GPT-4o mini), while complex queries are escalated to premium models (e.g., o3). This can reduce costs by 60–80%.

---

### Q9. Which of the following is true about licensing in the Model Catalog?

- A) All models use the same MIT open-source license
- B) Proprietary models like GPT-4o give you full access to model weights
- C) Models have different licenses (MIT, Apache 2.0, Llama Community, proprietary) with varying restrictions
- D) Licensing only matters for on-premise deployments

**Correct Answer:** C  
**Explanation:** The catalog includes models under various licenses. MIT/Apache 2.0 allow broad commercial use, the Llama Community License has a 700M MAU threshold, and proprietary models are accessed via API terms without weight access. Understanding these differences is critical before production deployment.

---

### Q10. When comparing models in the AI Foundry Playground, what is the MOST important practice?

- A) Always pick the model with the highest MMLU benchmark score
- B) Use the same system prompt and test prompts across all models for a fair comparison
- C) Only test one prompt per model to save time
- D) Choose the cheapest model regardless of output quality

**Correct Answer:** B  
**Explanation:** For a valid comparison, you must use identical system prompts and user messages across all models. This controls for variables and ensures you are comparing model performance, not prompt differences. Benchmarks are useful but don't replace testing with your actual use-case data.

---

## Scoring Guide

| Score | Result |
|-------|--------|
| 10/10 | ⭐ Perfect — Model Catalog Expert! |
| 8–9/10 | ✅ Pass — Strong understanding |
| 6–7/10 | ⚠️ Review — Revisit deployment types and decision framework |
| 0–5/10 | ❌ Retake — Re-read the blog and training guide before proceeding |

---

*Next Module → Module 8: Your First Model Deployment*
