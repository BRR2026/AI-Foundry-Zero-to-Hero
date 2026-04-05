# Module 9 — Assessment Quiz

## Fine-Tuning Models with Your Data

**Total Questions:** 10
**Passing Score:** 70% (7/10)
**Time Limit:** 15 minutes

---

### Question 1

**When is fine-tuning the BEST approach compared to prompt engineering alone?**

- A) When you need to access frequently changing data sources
- B) When you need consistent domain-specific output style that prompts can't reliably achieve
- C) When you want to avoid any additional costs beyond API usage
- D) When you're prototyping a new application quickly

**Correct Answer:** B

**Explanation:** Fine-tuning is ideal when you need the model to internalize a specific output style, tone, or domain expertise that can't be achieved reliably through prompting alone. RAG (A) handles dynamic data. Prompt engineering (C, D) is free and fastest for prototyping.

---

### Question 2

**What file format does Azure AI Foundry require for fine-tuning training data?**

- A) CSV with columns for input and output
- B) JSONL (JSON Lines) with a messages array
- C) Parquet with structured schema
- D) Plain text with tab-separated fields

**Correct Answer:** B

**Explanation:** AI Foundry uses the JSONL format where each line is a JSON object containing a `messages` array with `system`, `user`, and `assistant` roles — matching the chat completions API format.

---

### Question 3

**A proper JSONL training example for fine-tuning a chat model must include which of the following?**

- A) A `messages` array with at least one `user` and one `assistant` message
- B) A `prompt` field and a `completion` field
- C) An `input` object and `output` object
- D) A `context` field and a `response` field

**Correct Answer:** A

**Explanation:** Fine-tuning chat models in AI Foundry uses the chat format with a `messages` array. Each example must include at least a `user` message and an `assistant` message. A `system` message is optional but recommended for consistency.

---

### Question 4

**What is the MINIMUM number of training examples AI Foundry requires to start a fine-tuning job?**

- A) 5
- B) 10
- C) 50
- D) 100

**Correct Answer:** B

**Explanation:** AI Foundry requires a minimum of 10 training examples to start a fine-tuning job. However, 50–100+ examples are recommended for meaningful improvement in model performance.

---

### Question 5

**You notice that training loss is decreasing but validation loss is increasing during a fine-tuning job. What does this indicate?**

- A) The model is underfitting and needs more epochs
- B) The model is overfitting — memorizing training data instead of generalizing
- C) The model has converged and training is complete
- D) The training data is corrupted

**Correct Answer:** B

**Explanation:** When training loss decreases while validation loss increases, the model is overfitting — it's memorizing the training examples rather than learning generalizable patterns. Solutions include reducing epochs, adding more diverse training data, or lowering the learning rate.

---

### Question 6

**Which hyperparameter controls how many complete passes the model makes through the training dataset?**

- A) `batch_size`
- B) `learning_rate_multiplier`
- C) `n_epochs`
- D) `seed`

**Correct Answer:** C

**Explanation:** `n_epochs` determines how many times the model iterates through the entire training dataset. More epochs mean more exposure to the data but increase the risk of overfitting, especially with small datasets.

---

### Question 7

**You have a dataset of 200 domain-specific Q&A examples. What is the recommended number of epochs?**

- A) 1
- B) 3–4
- C) 10–15
- D) 20+

**Correct Answer:** B

**Explanation:** For a dataset of 100–500 examples, 3–4 epochs is generally recommended. This provides enough passes for the model to learn the patterns without overfitting. Larger datasets need fewer epochs; smaller datasets may benefit from slightly more, but with careful monitoring.

---

### Question 8

**What is a key benefit of combining fine-tuning with RAG in a production system?**

- A) It eliminates the need for any training data
- B) Fine-tuning provides consistent style/tone while RAG provides up-to-date factual grounding
- C) It makes the model 100% accurate on all queries
- D) It removes the need for model deployment

**Correct Answer:** B

**Explanation:** Combining fine-tuning and RAG leverages the strengths of both: fine-tuning teaches the model your domain's style, terminology, and output patterns, while RAG ensures responses are grounded in current, factual data. This combination produces production-grade AI systems.

---

### Question 9

**After fine-tuning is complete, what is the correct way to deploy the model in AI Foundry?**

- A) Download the model weights and host them on your own infrastructure
- B) Create a new deployment in AI Foundry's Deployments section, selecting the fine-tuned model
- C) Fine-tuned models are automatically deployed — no additional steps needed
- D) Export the model to a Docker container and deploy to Azure Kubernetes Service

**Correct Answer:** B

**Explanation:** After training completes, you create a new deployment in the AI Foundry portal by navigating to Deployments, clicking "+ Create deployment," selecting your fine-tuned model, setting a deployment name and TPM rate limit, and clicking Create. The model is then accessible via the standard Azure OpenAI API.

---

### Question 10

**Why does a fine-tuned model often require shorter prompts than the base model for the same task?**

- A) Fine-tuned models have larger context windows
- B) The model's weights have been adjusted to internalize the desired behavior, reducing the need for lengthy instructions and few-shot examples
- C) Fine-tuned models automatically ignore system messages
- D) Azure AI Foundry adds hidden instructions to fine-tuned models

**Correct Answer:** B

**Explanation:** Fine-tuning adjusts the model's internal weights based on your training data, so the model "knows" the expected behavior, format, and domain knowledge without being told each time. This eliminates the need for long system prompts and few-shot examples, reducing token usage by 60–80% and improving latency.

---

## Answer Key

| Q  | Answer | Topic |
|----|--------|-------|
| 1  | B | Decision framework |
| 2  | B | Data format |
| 3  | A | JSONL structure |
| 4  | B | Data requirements |
| 5  | B | Loss curve interpretation |
| 6  | C | Hyperparameters |
| 7  | B | Epoch selection |
| 8  | B | Fine-tuning + RAG |
| 9  | B | Deployment workflow |
| 10 | B | Fine-tuning benefits |
