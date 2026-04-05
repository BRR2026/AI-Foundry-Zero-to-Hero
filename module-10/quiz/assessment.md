# Module 10 — Knowledge Assessment

## Evaluation, Content Safety & Testing

> **ARC 2 · Prompt Engineering & Models**  
> **Total Questions:** 10 | **Passing Score:** 80% (8/10)

---

### Question 1

**Which evaluation metric measures whether a model's response is faithful to the provided source context?**

- A) Relevance
- B) Fluency
- C) Groundedness
- D) Coherence

<details>
<summary>Answer</summary>

**C) Groundedness**

Groundedness evaluates whether the claims in a generated response can be traced back to the input context. This is the primary metric for detecting hallucination in RAG scenarios.

</details>

---

### Question 2

**A model response correctly answers the user's question with perfect grammar but includes a statistic that is not found in the provided context. Which metrics would be affected?**

- A) Fluency would be low
- B) Groundedness would be low, while relevance and fluency may still be high
- C) All metrics would be equally low
- D) Coherence would be low

<details>
<summary>Answer</summary>

**B) Groundedness would be low, while relevance and fluency may still be high**

The response is relevant (it answers the question), fluent (good grammar), and likely coherent, but it is not grounded because the statistic cannot be verified against the context.

</details>

---

### Question 3

**What are the four primary content safety categories in Azure AI Content Safety?**

- A) Spam, Phishing, Malware, Hate
- B) Hate & Fairness, Sexual, Violence, Self-Harm
- C) Profanity, Misinformation, Bias, Toxicity
- D) Copyright, Privacy, Security, Accuracy

<details>
<summary>Answer</summary>

**B) Hate & Fairness, Sexual, Violence, Self-Harm**

Azure AI Content Safety uses these four categories, each with severity levels from Safe (0) to High (6).

</details>

---

### Question 4

**In Azure AI Content Safety, what does a severity level of 4 (Medium) indicate?**

- A) Content is completely safe
- B) Content contains mild or indirect references
- C) Content is moderately harmful and may be inappropriate in some contexts
- D) Content is severely explicit and always blocked

<details>
<summary>Answer</summary>

**C) Content is moderately harmful and may be inappropriate in some contexts**

Severity levels are: Safe (0), Low (2), Medium (4), and High (6). Medium indicates moderate content that may be inappropriate depending on the application context.

</details>

---

### Question 5

**What is the purpose of Prompt Shield in Azure AI Foundry?**

- A) To encrypt user prompts before sending them to the model
- B) To detect and block jailbreak attempts and indirect prompt injection attacks
- C) To limit the length of user prompts
- D) To translate prompts into multiple languages

<details>
<summary>Answer</summary>

**B) To detect and block jailbreak attempts and indirect prompt injection attacks**

Prompt Shield protects against two attack vectors: direct jailbreaks (user trying to override the system message) and indirect prompt injection (malicious instructions embedded in external data sources processed during RAG).

</details>

---

### Question 6

**What is the primary purpose of custom blocklists in Azure AI Content Safety?**

- A) To improve model response quality
- B) To filter domain-specific terms that the built-in categories don't cover
- C) To speed up content safety processing
- D) To replace the four built-in safety categories

<details>
<summary>Answer</summary>

**B) To filter domain-specific terms that the built-in categories don't cover**

Custom blocklists extend the built-in safety system with organization-specific terms such as competitor product names, internal codenames, or regulated terminology.

</details>

---

### Question 7

**Which of the following is the correct order for a red-teaming exercise?**

- A) Probe → Scope → Remediate → Analyze → Retest
- B) Analyze → Probe → Scope → Retest → Remediate
- C) Scope → Probe → Analyze → Remediate → Retest
- D) Remediate → Scope → Probe → Retest → Analyze

<details>
<summary>Answer</summary>

**C) Scope → Probe → Analyze → Remediate → Retest**

The framework follows a logical sequence: define what to test (Scope), execute attacks (Probe), categorize findings (Analyze), fix issues (Remediate), and verify fixes (Retest).

</details>

---

### Question 8

**In an automated evaluation pipeline, what is the purpose of "threshold gating"?**

- A) Limiting the number of test cases that can run
- B) Defining minimum acceptable metric scores and failing the pipeline if any metric drops below them
- C) Restricting who can access evaluation results
- D) Setting a time limit for how long evaluations can run

<details>
<summary>Answer</summary>

**B) Defining minimum acceptable metric scores and failing the pipeline if any metric drops below them**

Threshold gating ensures that deployments only proceed when quality and safety metrics meet predefined standards — for example, requiring groundedness ≥ 4.0 before allowing deployment.

</details>

---

### Question 9

**Which Python package provides the primary SDK for running AI evaluations in Azure AI Foundry?**

- A) `azure-ai-ml`
- B) `azure-ai-evaluation`
- C) `openai`
- D) `azure-cognitiveservices-vision`

<details>
<summary>Answer</summary>

**B) `azure-ai-evaluation`**

The `azure-ai-evaluation` package provides evaluators for both quality metrics (groundedness, relevance, coherence, fluency, similarity) and safety metrics (violence, sexual, self-harm, hate), along with the `evaluate()` function for batch evaluation.

</details>

---

### Question 10

**Why is it recommended to run evaluations on a regular schedule against production traffic samples, even after deployment?**

- A) To generate more billing revenue
- B) To detect quality drift as real-world data and usage patterns evolve over time
- C) Because models automatically degrade after 30 days
- D) To satisfy a regulatory checkbox requirement only

<details>
<summary>Answer</summary>

**B) To detect quality drift as real-world data and usage patterns evolve over time**

Quality drift is the gradual degradation that can occur as the distribution of real-world queries shifts from what the model was tested on. Regular evaluation against production samples catches this early, before users are impacted.

</details>

---

## Scoring

| Score | Result |
|-------|--------|
| 10/10 | ⭐ Excellent — You've mastered evaluation and safety! |
| 8–9/10 | ✅ Pass — Strong understanding, review missed areas |
| 6–7/10 | ⚠️ Review Needed — Revisit Content Safety and SDK sections |
| < 6/10 | ❌ Retry — Re-read the module blog and attempt again |
