# Module 15 — Assessment Quiz
## Evaluation Datasets & Quality Measurement
**ARC 3: DATA & RAG** · 🟢 Beginner Tier (Final Module)

**Instructions:** Choose the best answer for each question. Passing score: 80% (8/10).

---

### Question 1

What is a **golden dataset** in the context of AI evaluation?

- A) A dataset containing only gold-standard financial data
- B) A curated collection of input-output pairs that define expected AI behavior
- C) The largest possible dataset for training a model
- D) A dataset that has been encrypted for security

**✅ Answer: B**
> A golden dataset (also called a "gold set" or "benchmark set") is a curated collection of input-output pairs with expert-validated ground truth that defines the expected behavior of your AI application — essentially unit tests for your AI.

---

### Question 2

Which of the following is **NOT** a built-in evaluator in the `azure-ai-evaluation` SDK?

- A) GroundednessEvaluator
- B) RelevanceEvaluator
- C) CreativityEvaluator
- D) CoherenceEvaluator

**✅ Answer: C**
> The SDK provides GroundednessEvaluator, RelevanceEvaluator, CoherenceEvaluator, FluencyEvaluator, SimilarityEvaluator, and F1ScoreEvaluator. There is no built-in CreativityEvaluator.

---

### Question 3

What does the **GroundednessEvaluator** measure?

- A) Whether the response is grammatically correct
- B) Whether the claims in the response are supported by the provided context
- C) Whether the response matches the ground truth word-for-word
- D) Whether the response is shorter than 200 words

**✅ Answer: B**
> The GroundednessEvaluator assesses whether the claims and statements in the AI's response are supported by the provided context. It returns a score from 1 to 5 and does not require ground truth — it uses the context instead.

---

### Question 4

Which evaluator **requires** ground truth to function?

- A) GroundednessEvaluator
- B) CoherenceEvaluator
- C) SimilarityEvaluator
- D) FluencyEvaluator

**✅ Answer: C**
> The SimilarityEvaluator measures semantic closeness between the model's response and a reference ground truth answer. Without ground truth, it has nothing to compare against. Groundedness, Coherence, and Fluency evaluators do not require ground truth.

---

### Question 5

What is the primary advantage of **LLM-as-a-Judge** over traditional metrics like BLEU and ROUGE?

- A) It is always cheaper to run
- B) It provides deterministic, reproducible scores
- C) It captures nuanced quality aspects that token-overlap metrics miss
- D) It requires no compute resources

**✅ Answer: C**
> LLM-as-a-Judge uses a language model to evaluate output with human-like understanding, capturing nuances in meaning, completeness, and quality that simple token-overlap metrics like BLEU and ROUGE cannot assess. It is more expensive and non-deterministic, but far more nuanced.

---

### Question 6

Which of the following is a known **bias** in LLM-as-a-Judge evaluations?

- A) The judge prefers shorter answers
- B) The judge always gives the same score
- C) The judge tends to rate longer (more verbose) answers higher
- D) The judge cannot process code examples

**✅ Answer: C**
> Verbosity bias is a well-documented issue where LLM judges tend to rate longer answers more favorably, regardless of actual quality. Other known biases include self-evaluation bias and position bias. Mitigate verbosity bias by instructing the judge to focus on accuracy over length.

---

### Question 7

What is the recommended approach when your evaluation results need to appear in the AI Foundry portal?

- A) Manually upload a CSV file to the portal
- B) Pass the `azure_ai_project` parameter to the `evaluate()` function
- C) Use the Azure CLI to push results
- D) Configure a webhook in the portal settings

**✅ Answer: B**
> When you pass the `azure_ai_project` configuration to the `evaluate()` function, evaluation results are automatically logged to the AI Foundry portal. You can then view run summaries, per-row details, and compare runs in the Evaluation section of your project.

---

### Question 8

In a CI/CD evaluation gate, what should happen when a quality metric falls **below** the defined threshold?

- A) Log a warning but continue the deployment
- B) The pipeline should fail and block the deployment
- C) Automatically retrain the model
- D) Send an email to the CEO

**✅ Answer: B**
> An evaluation gate should exit with a non-zero exit code (e.g., `sys.exit(1)`) when metrics fall below thresholds, which causes the CI/CD pipeline to fail and block deployment. This prevents quality regressions from reaching production.

---

### Question 9

Why is **category-level analysis** important for regression detection?

- A) It reduces the cost of evaluation
- B) It reveals hidden regressions that aggregate metrics would mask
- C) It makes the evaluation run faster
- D) It is required by Azure compliance regulations

**✅ Answer: B**
> A model might improve overall but regress significantly on specific question categories (e.g., "how-to" questions). Aggregate metrics would hide this regression. Category-level analysis breaks down scores by question type, difficulty, or domain to reveal targeted quality issues.

---

### Question 10

How often should you run evaluations, even when no changes have been made to your AI application?

- A) Never — only run evaluations when code changes
- B) Once a year during annual reviews
- C) On a regular schedule (weekly or bi-weekly) to catch external drift
- D) Only when users report complaints

**✅ Answer: C**
> Regular scheduled evaluations (weekly or bi-weekly) catch issues caused by external factors: model API updates, data source changes, or environmental drift that occurs independently of your code changes. Proactive evaluation prevents surprises.

---

## Scoring

| Score | Result |
|-------|--------|
| 10/10 | ⭐ Perfect — Outstanding mastery of evaluation concepts! |
| 8-9/10 | ✅ Pass — Solid understanding, ready for the Intermediate tier |
| 6-7/10 | ⚠️ Review — Revisit the automated pipelines and LLM-as-a-Judge sections |
| < 6/10 | ❌ Retry — Re-read the blog post and complete the lab before retaking |

---

**🎓 Completing this quiz marks the end of the Beginner tier (🟢). Module 16 begins the Intermediate tier (🟡) with ARC 4: AI Agents!**
