# Module 7 — Hands-On Lab: Compare 3 Models in the AI Foundry Playground

> **Duration:** 45 minutes  
> **Difficulty:** Beginner  
> **Prerequisites:** Azure AI Foundry project with Contributor access

---

## Objective

Deploy three models from different families in Azure AI Foundry and compare their performance on the same task using the Playground. By the end of this lab, you will be able to articulate *why* one model outperforms another for a specific use case.

---

## Part 1 — Deploy Three Models (15 min)

### Step 1: Open the Model Catalog

1. Navigate to [ai.azure.com](https://ai.azure.com)
2. Select your project from the dashboard
3. Click **Model catalog** in the left sidebar

### Step 2: Deploy Model A — GPT-4o mini (OpenAI)

1. Search for **GPT-4o mini** in the catalog
2. Click the model card to review its details
3. Click **Deploy** → Select **Serverless API (pay-as-you-go)**
4. Name the deployment: `gpt-4o-mini-lab7`
5. Accept defaults and click **Deploy**
6. Wait for the deployment status to show **Succeeded**

### Step 3: Deploy Model B — Phi-4 (Microsoft)

1. Return to the Model Catalog
2. Search for **Phi-4**
3. Click **Deploy** → Select **Serverless API** (if available) or **Managed Compute**
4. Name the deployment: `phi-4-lab7`
5. Deploy and wait for it to succeed

### Step 4: Deploy Model C — Mistral Small (Mistral)

1. Return to the Model Catalog
2. Search for **Mistral Small**
3. Click **Deploy** → Select **Serverless API**
4. Name the deployment: `mistral-small-lab7`
5. Deploy and wait for it to succeed

> **💡 Tip:** If a model is not available in your region, choose an alternative from the same family (e.g., Mistral Nemo instead of Mistral Small).

---

## Part 2 — Test in the Playground (20 min)

### Step 5: Configure the System Prompt

1. Navigate to **Playground** in the left sidebar
2. Select your first deployment (`gpt-4o-mini-lab7`)
3. In the **System message** field, enter:

```
You are a helpful assistant that explains technical concepts in simple terms.
When explaining, use an analogy that a 10-year-old would understand.
Keep responses under 150 words.
```

### Step 6: Run Test Prompts

Send each of the following prompts **one at a time** and record the response:

| # | Prompt |
|---|--------|
| 1 | "What is an API?" |
| 2 | "Explain how encryption works" |
| 3 | "What is the difference between AI and machine learning?" |

### Step 7: Repeat for All Three Models

1. Switch the deployment selector to `phi-4-lab7`
2. Enter the **same system prompt** (copy/paste exactly)
3. Send the **same 3 test prompts** and record responses
4. Repeat for `mistral-small-lab7`

> **⚠️ Important:** Use the exact same system prompt and user messages for all three models. Consistency is essential for a fair comparison.

---

## Part 3 — Evaluate & Compare (10 min)

### Step 8: Score Each Response

For each model × prompt combination (9 total responses), score on a 1–5 scale:

| Dimension | What to Look For | Scale |
|-----------|-----------------|-------|
| **Accuracy** | Is the explanation factually correct? | 1 = wrong, 5 = perfect |
| **Simplicity** | Would a 10-year-old understand it? | 1 = too complex, 5 = very simple |
| **Analogy** | Is the analogy creative and appropriate? | 1 = no analogy, 5 = excellent |
| **Conciseness** | Is it under 150 words as instructed? | 1 = way over, 5 = perfectly concise |

### Step 9: Fill In Your Comparison Matrix

Copy and fill in this table:

```
| Model           | Prompt | Accuracy | Simplicity | Analogy | Conciseness | Total |
|-----------------|--------|----------|------------|---------|-------------|-------|
| GPT-4o mini     | 1      |          |            |         |             |       |
| GPT-4o mini     | 2      |          |            |         |             |       |
| GPT-4o mini     | 3      |          |            |         |             |       |
| Phi-4           | 1      |          |            |         |             |       |
| Phi-4           | 2      |          |            |         |             |       |
| Phi-4           | 3      |          |            |         |             |       |
| Mistral Small   | 1      |          |            |         |             |       |
| Mistral Small   | 2      |          |            |         |             |       |
| Mistral Small   | 3      |          |            |         |             |       |
```

### Step 10: Declare a Winner

Based on your total scores, answer these questions:

1. **Which model performed best overall?** Why?
2. **Which model was best at following the conciseness instruction?**
3. **Which model created the most creative analogies?**
4. **If you had to pick one model for a children's educational app, which would it be?**
5. **What trade-offs (cost, speed, quality) would influence your final choice?**

---

## Cleanup

To avoid unnecessary charges, delete the test deployments after completing the lab:

1. Go to **Deployments** in the left sidebar
2. Select each lab deployment (`gpt-4o-mini-lab7`, `phi-4-lab7`, `mistral-small-lab7`)
3. Click **Delete** and confirm

---

## Bonus Challenge

If you finish early, try these extensions:

- **Add a 4th model** (e.g., Cohere Command R or Llama 3.1 8B) and see how it compares
- **Change the task** — Try a classification task instead of explanation and see if the rankings change
- **Adjust temperature** — Test each model at temperature 0.0 vs 1.0 and compare output consistency

---

## Success Criteria

✅ Three models deployed from different families  
✅ Same system prompt and test prompts used for all models  
✅ Comparison matrix filled in with scores  
✅ Clear winner identified with reasoning  
✅ Trade-offs (cost, speed, quality) documented  

---

*Next Lab → Module 8: Deploy Your First Model Endpoint*
