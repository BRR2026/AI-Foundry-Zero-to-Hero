# Module 9 — Hands-On Lab

## Fine-Tune GPT-4o-mini on a Custom Q&A Dataset

**Duration:** 45–60 minutes
**Difficulty:** Intermediate
**Prerequisites:** Azure AI Foundry project with Azure OpenAI resource (fine-tuning-supported region)

---

## Objective

Fine-tune `gpt-4o-mini` to act as a product support assistant for **CloudFlow**, a fictional SaaS platform. By the end of this lab, you'll have a deployed custom model that answers CloudFlow support questions accurately — without needing a detailed system prompt.

---

## Lab Setup

### Requirements

- Azure subscription with an AI Foundry project
- Azure OpenAI resource in **East US2**, **North Central US**, **Sweden Central**, or **Switzerland West**
- Python 3.10+ with the `openai` package (`pip install openai`)
- A text editor for creating JSONL files

---

## Exercise 1: Create Training Data (15 min)

### Step 1.1 — Understand the JSONL Format

Each training example is a single JSON object on one line with a `messages` array:

```json
{"messages": [{"role": "system", "content": "You are CloudFlow's AI support assistant."}, {"role": "user", "content": "..."}, {"role": "assistant", "content": "..."}]}
```

### Step 1.2 — Create `cloudflow_train.jsonl`

Create a file called `cloudflow_train.jsonl` with **at least 50 Q&A pairs**. Cover these categories:

| Category | Example Topics |
|----------|---------------|
| Account Management | Password reset, profile updates, billing |
| Product Features | Workflows, automations, integrations |
| Troubleshooting | Login issues, sync errors, performance |
| Pricing & Plans | Free vs. Pro vs. Enterprise, upgrades |
| Getting Started | Onboarding, first project, tutorials |

Here are starter examples to follow:

```jsonl
{"messages": [{"role": "system", "content": "You are CloudFlow's AI support assistant. Answer customer questions accurately and helpfully."}, {"role": "user", "content": "How do I reset my CloudFlow password?"}, {"role": "assistant", "content": "To reset your CloudFlow password:\n1. Go to app.cloudflow.io/login\n2. Click 'Forgot Password?' below the sign-in button\n3. Enter your registered email address\n4. Check your inbox for a reset link (valid for 24 hours)\n5. Click the link and set your new password\n\nIf you don't receive the email, check your spam folder or contact support@cloudflow.io."}]}
{"messages": [{"role": "system", "content": "You are CloudFlow's AI support assistant. Answer customer questions accurately and helpfully."}, {"role": "user", "content": "What's the difference between Free and Pro plans?"}, {"role": "assistant", "content": "Here's a comparison of CloudFlow plans:\n\n**Free Plan:**\n• Up to 5 workflows\n• 3 team members\n• Community support\n• 1 GB storage\n\n**Pro Plan ($49/month per user):**\n• Unlimited workflows\n• Up to 50 team members\n• Priority support\n• 100 GB storage\n• Advanced analytics\n• REST API access\n• SSO with SAML 2.0\n\nStart a 14-day free Pro trial at app.cloudflow.io/pricing."}]}
{"messages": [{"role": "system", "content": "You are CloudFlow's AI support assistant. Answer customer questions accurately and helpfully."}, {"role": "user", "content": "My workflow automation stopped running. What should I check?"}, {"role": "assistant", "content": "If your CloudFlow automation stopped, check these things:\n\n1. **Trigger status** — Go to Workflows > your workflow > Trigger tab. Ensure the trigger is set to 'Active'\n2. **Execution logs** — Check Workflows > History for error messages\n3. **Connected services** — Verify your integrations haven't expired (Settings > Integrations)\n4. **Plan limits** — Free plans are limited to 100 automation runs/month\n5. **Schedule** — If it's a scheduled workflow, confirm the cron expression is correct\n\nIf none of these resolve it, contact support@cloudflow.io with your workflow ID."}]}
```

### Step 1.3 — Create `cloudflow_val.jsonl`

Create a separate validation file with **10–15 different examples** (do not reuse training data). These should cover similar categories but use different questions.

### Step 1.4 — Validate Your Files

Run this Python script to check your JSONL files:

```python
import json

def validate_jsonl(filepath):
    errors = []
    with open(filepath, 'r', encoding='utf-8') as f:
        for i, line in enumerate(f, 1):
            try:
                data = json.loads(line)
                if 'messages' not in data:
                    errors.append(f"Line {i}: Missing 'messages' key")
                    continue
                roles = [m['role'] for m in data['messages']]
                if roles[-1] != 'assistant':
                    errors.append(f"Line {i}: Last message must be 'assistant'")
                if 'user' not in roles:
                    errors.append(f"Line {i}: Must include a 'user' message")
            except json.JSONDecodeError as e:
                errors.append(f"Line {i}: Invalid JSON — {e}")
    
    if errors:
        print(f"❌ {filepath} has {len(errors)} error(s):")
        for err in errors:
            print(f"  {err}")
    else:
        print(f"✅ {filepath} is valid ({i} examples)")

validate_jsonl('cloudflow_train.jsonl')
validate_jsonl('cloudflow_val.jsonl')
```

---

## Exercise 2: Fine-Tune in AI Foundry (15 min)

### Step 2.1 — Navigate to Fine-Tuning

1. Open [ai.azure.com](https://ai.azure.com) and select your project
2. In the left sidebar, click **Fine-tuning** (under *Build & customize*)
3. Click **"+ Fine-tune a model"**

### Step 2.2 — Select Base Model

1. Choose **gpt-4o-mini** from the model list
2. Click **Next**

### Step 2.3 — Upload Training Data

1. Click **Upload file** and select `cloudflow_train.jsonl`
2. Wait for validation to complete (green checkmark)
3. Click **Upload file** for validation data and select `cloudflow_val.jsonl`
4. Review the data preview — verify your examples look correct

### Step 2.4 — Configure Hyperparameters

Use the default settings for your first run:

| Parameter | Value |
|-----------|-------|
| Epochs | Auto |
| Learning Rate Multiplier | Auto |
| Batch Size | Auto |
| Seed | 42 (for reproducibility) |

### Step 2.5 — Submit Training Job

1. Review your configuration
2. Click **Submit**
3. Note the job ID for tracking

> ⏱️ **Expected time:** 15–30 minutes for ~50 examples

---

## Exercise 3: Monitor & Deploy (10 min)

### Step 3.1 — Monitor Training Progress

1. Navigate to your fine-tuning job
2. Watch the **training loss** and **validation loss** curves
3. Verify both losses are decreasing and converging
4. Check for overfitting: validation loss should NOT increase while training loss decreases

### Step 3.2 — Deploy Your Model

Once training completes:

1. Go to **Deployments** (under *Manage*)
2. Click **"+ Create deployment"**
3. Select your fine-tuned model from the **Custom fine-tuned models** tab
4. Set deployment name: `cloudflow-support-v1`
5. Set TPM rate limit (start with 10K)
6. Click **Create**

### Step 3.3 — Test in Playground

1. Open the **Playground** in AI Foundry
2. Select your `cloudflow-support-v1` deployment
3. Test with questions the model was trained on:
   - *"How do I reset my password?"*
4. Test with **new questions** not in training data:
   - *"Can I export my workflow as a template?"*
   - *"What happens when my free trial expires?"*

---

## Exercise 4: Evaluate & Compare (10 min)

### Step 4.1 — Side-by-Side Comparison

Compare responses from your fine-tuned model vs. the base `gpt-4o-mini`:

```python
from openai import AzureOpenAI

client = AzureOpenAI(
    azure_endpoint="https://YOUR-RESOURCE.openai.azure.com/",
    api_key="YOUR-API-KEY",
    api_version="2024-12-01-preview"
)

test_questions = [
    "How do I upgrade to the Pro plan?",
    "My integrations aren't syncing. Help!",
    "What's included in the Enterprise plan?",
]

for q in test_questions:
    messages = [{"role": "user", "content": q}]
    
    # Fine-tuned model
    ft_response = client.chat.completions.create(
        model="cloudflow-support-v1",
        messages=messages, temperature=0
    )
    
    # Base model (with simple prompt)
    base_messages = [
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": q}
    ]
    base_response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=base_messages, temperature=0
    )
    
    print(f"\n{'='*60}")
    print(f"Q: {q}")
    print(f"\n--- Fine-tuned ---\n{ft_response.choices[0].message.content}")
    print(f"\n--- Base model ---\n{base_response.choices[0].message.content}")
```

### Step 4.2 — Document Your Results

Record your observations:

- [ ] Does the fine-tuned model give CloudFlow-specific answers?
- [ ] Does it use CloudFlow terminology without being prompted?
- [ ] Is the response format consistent?
- [ ] How do responses compare to the base model?
- [ ] Are there any areas where the model struggles?

---

## Cleanup

When you're done testing, consider deleting the deployment to avoid ongoing hosting costs:

1. Go to **Deployments** in AI Foundry
2. Select `cloudflow-support-v1`
3. Click **Delete**

> The fine-tuned model itself (not the deployment) is retained and can be redeployed later.

---

## Success Criteria

| Criterion | Check |
|-----------|-------|
| JSONL files pass validation | ✅ |
| Training completes with decreasing loss | ✅ |
| Validation loss converges (no overfitting) | ✅ |
| Model deployed and accessible via Playground | ✅ |
| Fine-tuned model gives CloudFlow-specific responses | ✅ |
| Documented comparison with base model | ✅ |

---

## Bonus Challenges

1. **Add 50 more examples** and retrain — does accuracy improve?
2. **Test with adversarial inputs** — what happens when you ask about a competitor's product?
3. **Try different hyperparameters** — train with 2 epochs vs. 5 epochs and compare
4. **Combine with RAG** — set up a simple retrieval layer to ground answers in a CloudFlow knowledge base
