# Week 2: Assessment — Understanding the AI Lifecycle

## **ARC 1: Foundations** | Week 2 of 45 — Zero to Hero: Azure AI Foundry

---

| Detail              | Info                                    |
|---------------------|-----------------------------------------|
| **Total Questions** | 20                                      |
| **Format**          | Multiple Choice, True/False, Short Answer, Matching |
| **Time Limit**      | 30 minutes (suggested)                  |
| **Passing Score**   | 80% (16/20)                             |
| **Difficulty**      | ⭐ Beginner                              |

---

## Instructions

- Answer all 20 questions.
- For multiple choice, select the **one best answer** unless stated otherwise.
- For short answer, write 1-3 sentences.
- The answer key with detailed explanations is at the bottom of this document.
- **No peeking!** Try your best before checking answers.

---

## Section 1: Multiple Choice (Questions 1–10)

### Question 1
What are the 6 stages of the AI development lifecycle **in the correct order**?

- A) Model Selection → Data Collection → Evaluation → Deployment → Monitoring → Data Preparation
- B) Data Collection → Data Preparation → Model Selection/Training → Evaluation → Deployment → Monitoring
- C) Data Preparation → Data Collection → Deployment → Model Selection → Evaluation → Monitoring
- D) Model Selection → Training → Data Preparation → Deployment → Evaluation → Monitoring

---

### Question 2
According to studies cited in the training material, what percentage of time do data scientists typically spend on data preparation?

- A) 10-20%
- B) 30-40%
- C) 60-80%
- D) 90-100%

---

### Question 3
Which of the following is an example of **semi-structured** data?

- A) A SQL database table
- B) A JSON file
- C) A handwritten letter (scanned image)
- D) A CSV spreadsheet

---

### Question 4
When should you consider **fine-tuning** a model instead of using prompt engineering?

- A) When you want the fastest possible setup
- B) When you need the model to consistently follow a specific format, style, or domain terminology
- C) When you have zero training data
- D) When you want to avoid all costs

---

### Question 5
What does the **Groundedness** evaluation metric measure?

- A) Whether the response is grammatically correct
- B) Whether the response is based on provided context and facts, rather than fabricated information
- C) Whether the response is relevant to the user's question
- D) Whether the response is free of harmful content

---

### Question 6
Which deployment pattern in Azure AI Foundry uses **pay-per-token** billing and requires no infrastructure management?

- A) Managed Compute (MaaP)
- B) Container Deployment
- C) Serverless API (MaaS)
- D) Azure Kubernetes Service

---

### Question 7
What is **RAG** (Retrieval-Augmented Generation)?

- A) A method of training models from scratch
- B) A technique that connects a pre-built model to external knowledge sources at query time
- C) A type of GPU used for AI training
- D) A monitoring dashboard in Azure AI Foundry

---

### Question 8
In the MLOps Maturity Model, what characterizes **Level 0**?

- A) Full automation with A/B testing and auto-scaling
- B) Automated CI/CD pipelines with monitoring dashboards
- C) Everything done by hand with no automation
- D) Auto-retraining with drift detection

---

### Question 9
Which of the following is **NOT** one of the key evaluation metrics built into Azure AI Foundry?

- A) Groundedness
- B) Profitability
- C) Coherence
- D) Relevance

---

### Question 10
In the customer support chatbot example, what was the primary model selection strategy used?

- A) Training a custom model from scratch
- B) Fine-tuning GPT-4o on support tickets
- C) Using a pre-built GPT-4o model with RAG (grounded in company data)
- D) Using an open-source model without any customization

---

## Section 2: True or False (Questions 11–14)

### Question 11
**True or False:** The AI development lifecycle is a straight line — once you deploy a model, the process is complete.

---

### Question 12
**True or False:** You need a credit card to create a free Azure account, but you won't be charged unless you explicitly upgrade to a paid plan.

---

### Question 13
**True or False:** Fine-tuning a model changes the model's internal weights, while prompt engineering only changes the input given to the model.

---

### Question 14
**True or False:** For 99% of business AI applications, training a model from scratch is the recommended approach.

---

## Section 3: Short Answer (Questions 15–17)

### Question 15
Explain the difference between **structured**, **semi-structured**, and **unstructured** data. Give one example of each.

---

### Question 16
A company deploys an AI chatbot and notices that after 3 months, customer satisfaction drops significantly. Using your knowledge of the AI lifecycle, explain what might be happening and what stages of the lifecycle they should revisit.

---

### Question 17
Why is evaluation considered a **continuous process** rather than a one-time event? Give at least two reasons.

---

## Section 4: Matching (Questions 18–20)

### Question 18
Match each **model strategy** to its best description:

| Strategy | Options |
|----------|---------|
| 1. Pre-built Model | A. Build and train a model architecture from scratch using your own data |
| 2. Fine-tuned Model | B. Use a model directly from the catalog with no modification |
| 3. Custom Model | C. Take a pre-built model and train it further on your specific data |

---

### Question 19
Match each **deployment pattern** to its primary use case:

| Deployment Pattern | Options |
|-------------------|---------|
| 1. Serverless API (MaaS) | A. Hybrid cloud, air-gapped environments, edge deployment |
| 2. Managed Compute (MaaP) | B. Prototyping, variable traffic, quick start |
| 3. Container Deployment | C. Production workloads, consistent traffic, compliance requirements |

---

### Question 20
Match each **lifecycle stage** to the correct Azure AI Foundry tool:

| Lifecycle Stage | Options |
|----------------|---------|
| 1. Data Preparation (for RAG) | A. Playground |
| 2. Model Selection | B. Azure Monitor integration |
| 3. Manual Testing | C. Azure AI Search index creation |
| 4. Production Monitoring | D. Model Catalog |

---

## ✅ Answer Key

### Question 1: **B**
**Data Collection → Data Preparation → Model Selection/Training → Evaluation → Deployment → Monitoring**

*Explanation:* The AI lifecycle always starts with data. You must collect data first, then prepare it before you can select or train a model. After selecting/training, you evaluate quality, then deploy, and finally monitor in production. The cycle then loops back as monitoring informs new data collection and improvements.

---

### Question 2: **C**
**60-80%**

*Explanation:* Data preparation is consistently the most time-consuming part of any AI project. Real-world data is messy — it contains errors, duplicates, inconsistent formats, and missing values. Cleaning and organizing this data takes significantly more time than model selection or deployment.

---

### Question 3: **B**
**A JSON file**

*Explanation:* Semi-structured data has some organizational properties (like tags, keys, or hierarchies) but doesn't fit neatly into rows and columns like a database table. JSON, XML, YAML, and log files are all semi-structured. A SQL table (A) and CSV (D) are structured. A scanned image (C) is unstructured.

---

### Question 4: **B**
**When you need the model to consistently follow a specific format, style, or domain terminology**

*Explanation:* Fine-tuning is most valuable when prompt engineering alone can't achieve the consistency or domain expertise you need. It's particularly useful for specialized terminology (medical, legal, financial), consistent output formats, or when you want to reduce prompt length by "baking in" instructions. It requires training data (rules out C), costs more than prompt engineering (rules out D), and takes longer to set up (rules out A).

---

### Question 5: **B**
**Whether the response is based on provided context and facts, rather than fabricated information**

*Explanation:* Groundedness specifically measures whether the AI's response is grounded in the source data/context provided, or if it's "hallucinating" (making up information). This is different from Fluency (grammar), Relevance (addressing the question), or Safety (harmful content).

---

### Question 6: **C**
**Serverless API (MaaS)**

*Explanation:* Serverless API, also known as Model as a Service (MaaS), uses pay-per-token billing. You don't manage any infrastructure — Azure handles everything. Managed Compute (MaaP) charges for VM uptime, not per-token. Container deployment uses your own infrastructure.

---

### Question 7: **B**
**A technique that connects a pre-built model to external knowledge sources at query time**

*Explanation:* RAG (Retrieval-Augmented Generation) is a pattern where a pre-built model is connected to external data sources (like company documents, FAQs, or databases). When a user asks a question, the system first retrieves relevant information from these sources, then includes it in the prompt so the model can generate grounded responses. This avoids the need to retrain the model.

---

### Question 8: **C**
**Everything done by hand with no automation**

*Explanation:* The MLOps Maturity Model ranges from Level 0 (fully manual) to Level 4 (expert, with auto-retraining and self-healing). Level 0 means no automation, version control, or systematic processes. Most beginners start here and should aim for Level 2 (intermediate) with automated CI/CD and monitoring.

---

### Question 9: **B**
**Profitability**

*Explanation:* Azure AI Foundry's built-in evaluation metrics include Groundedness, Relevance, Coherence, Fluency, Similarity, and Safety. "Profitability" is a business metric, not an AI quality metric. While cost and ROI are important business considerations, they're not evaluated by AI Foundry's evaluation tools.

---

### Question 10: **C**
**Using a pre-built GPT-4o model with RAG (grounded in company data)**

*Explanation:* The customer support chatbot example used GPT-4o (a pre-built model from the catalog) combined with RAG — connecting it to the company's support tickets, product manuals, and FAQ documents via Azure AI Search. This is the most common pattern for enterprise AI applications: pre-built model + RAG.

---

### Question 11: **False**

*Explanation:* The AI lifecycle is a continuous loop, not a straight line. After deployment, monitoring identifies issues that feed back into earlier stages. Low quality may require better data preparation, changing models, or updating prompts. The lifecycle repeats as you continuously improve your AI application.

---

### Question 12: **True**

*Explanation:* Azure's free tier requires a credit card for identity verification, but you receive $200 in credits and access to free services. You will not be charged until you explicitly upgrade to a pay-as-you-go subscription. This is a genuine free tier.

---

### Question 13: **True**

*Explanation:* This is a critical distinction. Fine-tuning modifies the model's internal weights (parameters) by training it on new data, permanently changing how it responds. Prompt engineering only changes the text input you send to the model — the model itself remains unchanged. Fine-tuning requires training data and compute; prompt engineering is free.

---

### Question 14: **False**

*Explanation:* Training a model from scratch is rarely needed for business applications. It requires massive datasets, significant ML expertise, and substantial compute costs. For 99% of business use cases, pre-built models (with prompt engineering and/or RAG) or fine-tuned models are sufficient and far more practical.

---

### Question 15: **Sample Answer**

**Structured data** has a fixed, predefined schema with rows and columns. *Example: A SQL database table of customer orders with columns for order_id, customer_name, date, and amount.*

**Semi-structured data** has some organizational properties (tags, keys, hierarchies) but doesn't fit into rigid rows and columns. *Example: A JSON file containing product information with nested attributes like reviews and specifications.*

**Unstructured data** has no predefined format or organization. *Example: A collection of PDF documents containing company policies, with free-form text, images, and tables.*

*Grading: Award full credit for correctly defining all three types and providing a valid example for each. Award partial credit for correct definitions with missing or incorrect examples.*

---

### Question 16: **Sample Answer**

The company is likely experiencing **model drift** — the AI's performance has degraded because real-world data patterns have changed since deployment. Possible causes include:

1. **New products or policies** were introduced that the AI's knowledge base doesn't cover
2. **Customer behavior changed** (seasonal trends, new issues arising)
3. **The AI's training data became stale** — it's answering based on outdated information

They should revisit:
- **Monitoring** (Stage 6): Analyze logs to identify where the AI is failing
- **Data Collection** (Stage 1): Gather recent support tickets and updated documents
- **Data Preparation** (Stage 2): Update the knowledge base with new information
- **Evaluation** (Stage 4): Re-run evaluation with recent test cases to measure current performance

This demonstrates the **feedback loop** in the AI lifecycle — monitoring findings trigger updates to earlier stages.

*Grading: Award full credit for identifying model drift or data staleness, mentioning at least 2 lifecycle stages to revisit, and explaining the feedback loop concept.*

---

### Question 17: **Sample Answer**

Evaluation should be continuous because:

1. **Models can drift over time** — as real-world data patterns change, a model that was accurate at deployment may become less accurate. Continuous evaluation catches this degradation early.

2. **Prompts and data change** — whenever you update prompts, add new documents to your knowledge base, or modify system instructions, you need to re-evaluate to ensure quality hasn't regressed.

3. **User needs evolve** — users may start asking different types of questions over time, and what was "relevant" at launch may not cover new use cases.

4. **Safety risks emerge** — new adversarial techniques and jailbreak methods are constantly being discovered, so safety evaluation must be ongoing.

*Grading: Award full credit for providing at least two valid reasons with explanations. Model drift and changing data/prompts are the most important reasons.*

---

### Question 18: **Matching**

| Strategy | Answer |
|----------|--------|
| 1. Pre-built Model | **B** — Use a model directly from the catalog with no modification |
| 2. Fine-tuned Model | **C** — Take a pre-built model and train it further on your specific data |
| 3. Custom Model | **A** — Build and train a model architecture from scratch using your own data |

---

### Question 19: **Matching**

| Deployment Pattern | Answer |
|-------------------|--------|
| 1. Serverless API (MaaS) | **B** — Prototyping, variable traffic, quick start |
| 2. Managed Compute (MaaP) | **C** — Production workloads, consistent traffic, compliance requirements |
| 3. Container Deployment | **A** — Hybrid cloud, air-gapped environments, edge deployment |

---

### Question 20: **Matching**

| Lifecycle Stage | Answer |
|----------------|--------|
| 1. Data Preparation (for RAG) | **C** — Azure AI Search index creation |
| 2. Model Selection | **D** — Model Catalog |
| 3. Manual Testing | **A** — Playground |
| 4. Production Monitoring | **B** — Azure Monitor integration |

---

## 📊 Score Tracker

| Section | Questions | Your Score |
|---------|-----------|-----------|
| Multiple Choice (1-10) | 10 points | ___ / 10 |
| True/False (11-14) | 4 points | ___ / 4 |
| Short Answer (15-17) | 3 points | ___ / 3 |
| Matching (18-20) | 3 points | ___ / 3 |
| **Total** | **20 points** | **___ / 20** |

| Score | Rating |
|-------|--------|
| 18-20 | ⭐⭐⭐ Excellent — You've mastered the AI lifecycle! |
| 16-17 | ⭐⭐ Good — Solid understanding, review missed topics |
| 13-15 | ⭐ Fair — Re-read the training guide and try again |
| 0-12 | Needs work — Review the material carefully and retake |

---

*© 2025 Zero to Hero: Azure AI Foundry Training. Week 2 of 45.*
