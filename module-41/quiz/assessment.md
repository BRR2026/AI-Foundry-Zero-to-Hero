# Module 41: Assessment Quiz

## Choose Your Capstone Project

> **Azure AI Foundry: Zero to Hero** — Module 41 of 45
> **Arc:** ARC 9 · CAPSTONE & MASTERY
> **Total Questions:** 10 (Multiple Choice)
> **Passing Score:** 80% (8/10)
> **Time Limit:** 15 minutes

---

### Question 1

What is the primary purpose of the **Scope Fitness Test** when choosing a capstone project?

- A) To rank project ideas by difficulty level
- B) To ensure the project meets minimum requirements for breadth, feasibility, and responsible AI coverage
- C) To calculate the estimated Azure cost of the project
- D) To determine which AI Foundry model deployment to use

**Correct Answer:** B

**Explanation:** The Scope Fitness Test validates that a capstone project uses ≥3 AI Foundry services, includes agent-tool integration, connects to at least one data source, implements responsible AI guardrails, includes monitoring, and is completable within 3 module sprints. It ensures breadth and feasibility, not ranking or cost estimation.

---

### Question 2

What does **MVC** stand for in the context of capstone planning, and why is it important?

- A) Model-View-Controller — it defines the software architecture pattern to use
- B) Minimum Viable Capstone — it defines the smallest end-to-end slice that demonstrates value
- C) Maximum Verification Checkpoint — it sets the highest quality bar for evaluation
- D) Managed Virtual Compute — it specifies the Azure compute tier for deployment

**Correct Answer:** B

**Explanation:** Minimum Viable Capstone (MVC) defines the smallest functional end-to-end slice that demonstrates real value. It prevents scope creep by locking the core scope before building begins, with additional features going to a "future work" backlog.

---

### Question 3

When writing user stories for requirements gathering, which format is recommended?

- A) "The system shall [requirement]"
- B) "As a [role], I want to [action], so that [benefit]"
- C) "Given [context], when [action], then [result]"
- D) "Feature: [name] — Description: [details]"

**Correct Answer:** B

**Explanation:** The "As a [role], I want to [action], so that [benefit]" format is the standard user story format recommended for capstone requirements gathering. It captures who the user is, what they need, and the measurable benefit they expect.

---

### Question 4

Which of the following is **NOT** a recommended architecture pattern for the capstone project?

- A) Single-Agent RAG Application
- B) Multi-Agent Orchestration
- C) Event-Driven AI Pipeline
- D) Azure Machine Learning AutoML Pipeline

**Correct Answer:** D

**Explanation:** The three recommended architecture patterns for the AI Foundry capstone are Single-Agent RAG, Multi-Agent Orchestration, and Event-Driven AI Pipeline. Azure Machine Learning AutoML Pipeline is a separate service and not part of the AI Foundry capstone architecture patterns covered in this curriculum.

---

### Question 5

In the SMART success criteria framework, what does the "M" in SMART stand for, and how should it be applied to capstone evaluation?

- A) Modular — criteria should cover each module independently
- B) Measurable — criteria should include specific numeric targets (e.g., "groundedness ≥ 4.0/5.0")
- C) Meaningful — criteria should align with company objectives
- D) Managed — criteria should use Azure managed services exclusively

**Correct Answer:** B

**Explanation:** The "M" in SMART stands for Measurable. Capstone success criteria should include specific, quantifiable targets such as evaluation scores (groundedness ≥ 4.0/5.0), accuracy percentages (≥ 85% on test dataset), or latency targets (< 3 seconds). This ensures objective assessment.

---

### Question 6

What is the recommended **minimum number of test cases** in the capstone evaluation dataset?

- A) 10 test cases
- B) 25 test cases
- C) 50 test cases
- D) 100 test cases

**Correct Answer:** C

**Explanation:** The evaluation plan recommends a minimum of 50 test cases spanning multiple categories: happy path queries (20), edge cases (10), out-of-scope queries (10), adversarial inputs (5), and multi-turn conversations (5). This provides sufficient coverage for meaningful evaluation.

---

### Question 7

Which AI Foundry evaluation metrics should capstone projects measure at minimum?

- A) Only groundedness and relevance
- B) Groundedness, relevance, coherence, and content safety
- C) Only latency and throughput
- D) User satisfaction score and net promoter score

**Correct Answer:** B

**Explanation:** Capstone projects should measure at minimum: groundedness (factual accuracy), relevance (answer appropriateness), coherence (logical consistency), and content safety (absence of harmful outputs). These are the core AI Foundry built-in evaluators that cover both quality and responsible AI.

---

### Question 8

What are **Architecture Decision Records (ADRs)**, and how many should be documented in the capstone proposal?

- A) Performance benchmarks for Azure services; at least 5
- B) Documented rationale for key technical choices (topology, knowledge strategy, deployment); at least 3
- C) Security compliance audit records; at least 10
- D) Azure Resource Manager deployment templates; at least 2

**Correct Answer:** B

**Explanation:** ADRs document the rationale behind key architectural decisions, including agent topology (single vs. multi-agent), knowledge strategy (RAG vs. fine-tuning), orchestration pattern, deployment model, and evaluation approach. At least 3 ADRs should be documented in the proposal.

---

### Question 9

During the four-phase evaluation plan, what is the target improvement from Phase 1 (Baseline) to Phase 2 (Iteration)?

- A) ≥ 5% improvement over baseline
- B) ≥ 10% improvement over baseline
- C) ≥ 15% improvement over baseline
- D) ≥ 25% improvement over baseline

**Correct Answer:** C

**Explanation:** The evaluation plan targets ≥ 15% improvement over baseline metrics during the Iteration phase (Module 43). This is achieved through prompt optimization, retrieval parameter tuning, and tool refinement, with metric improvements tracked per iteration.

---

### Question 10

Which of the following is the **highest-weighted category** in the capstone evaluation rubric?

- A) Agent Design (20%)
- B) Technical Depth (30%)
- C) Responsible AI (15%)
- D) Presentation (15%)

**Correct Answer:** B

**Explanation:** Technical Depth carries the highest weight at 30% of the capstone evaluation rubric. An excellent score (5) requires using 5+ AI Foundry services with custom integrations and optimized configurations. Agent Design (20%), Evaluation Quality (20%), Responsible AI (15%), and Presentation (15%) make up the remainder.

---

## 📊 Answer Key

| Question | Answer | Topic |
|----------|--------|-------|
| 1 | B | Scope Fitness Test |
| 2 | B | Minimum Viable Capstone |
| 3 | B | Requirements Gathering |
| 4 | D | Architecture Patterns |
| 5 | B | SMART Criteria |
| 6 | C | Evaluation Dataset |
| 7 | B | Evaluation Metrics |
| 8 | B | Architecture Decision Records |
| 9 | C | Evaluation Plan |
| 10 | B | Evaluation Rubric |

---

## 🎯 Post-Assessment

- **8-10 correct:** You're ready to begin your capstone build in Module 42!
- **6-7 correct:** Review the sections for missed questions, then proceed.
- **Below 6:** Revisit the training guide and blog post before starting your proposal.
