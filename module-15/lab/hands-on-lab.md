# Module 15 — Hands-On Lab
## Create an Evaluation Dataset & Run a Quality Benchmark
**ARC 3: DATA & RAG** · 🟢 Beginner Tier (Final Module) · Duration: 90 minutes

---

## Lab Overview

In this lab, you will build a golden evaluation dataset from scratch, run automated quality evaluations using the `azure-ai-evaluation` SDK, and analyze the results to identify and fix quality issues. This is the capstone lab of the Beginner tier.

### What You'll Build
- A 20-example golden evaluation dataset in JSONL format
- An automated evaluation pipeline using four built-in evaluators
- A regression detection script for CI/CD integration
- (Stretch) A custom LLM-as-a-Judge evaluator

### Prerequisites
- Azure AI Foundry project with a deployed model (GPT-4o or GPT-4o-mini)
- Python 3.10+ installed
- Completed Modules 1–14 (familiarity with AI Foundry portal and RAG concepts)

---

## Part 1: Environment Setup (10 minutes)

### Step 1.1 — Install Dependencies

```bash
pip install azure-ai-evaluation azure-identity azure-ai-projects openai pandas
```

### Step 1.2 — Configure Your Project

Create a file called `config.py`:

```python
# config.py — AI Foundry project configuration
AZURE_AI_PROJECT = {
    "subscription_id": "<your-subscription-id>",
    "resource_group_name": "<your-resource-group>",
    "project_name": "<your-foundry-project>"
}

# Model deployment name for generating responses
MODEL_DEPLOYMENT = "gpt-4o"

# Model endpoint (from AI Foundry portal → Deployments)
MODEL_ENDPOINT = "https://<your-endpoint>.openai.azure.com/"
```

### Step 1.3 — Verify Authentication

```python
from azure.identity import DefaultAzureCredential

credential = DefaultAzureCredential()
token = credential.get_token("https://cognitiveservices.azure.com/.default")
print("✅ Authentication successful!")
```

> **Note:** Run `az login` first if you haven't authenticated via Azure CLI.

---

## Part 2: Build the Golden Dataset (25 minutes)

### Step 2.1 — Define Your Domain

Choose a domain for your evaluation dataset. For this lab, we'll use **Azure AI Foundry documentation** as our domain.

### Step 2.2 — Create the Dataset File

Create `eval_dataset.jsonl` with at least 20 examples. Include a mix of four categories:

| Category | Count | Description |
|----------|-------|-------------|
| `factual` | 5+ | Specific facts with clear correct answers |
| `how-to` | 5+ | Step-by-step procedural questions |
| `comparison` | 5+ | Compare features, services, or approaches |
| `out-of-scope` | 5+ | Questions the AI should decline or redirect |

### Step 2.3 — Write Your Examples

```python
# create_dataset.py
import json

examples = [
    # FACTUAL examples
    {
        "query": "What programming languages does the azure-ai-evaluation SDK support?",
        "context": "The azure-ai-evaluation package is a Python SDK that provides evaluators for assessing the quality of generative AI applications built with Azure AI Foundry.",
        "ground_truth": "The azure-ai-evaluation SDK supports Python.",
        "category": "factual",
        "difficulty": "easy"
    },
    {
        "query": "What scale does the GroundednessEvaluator use?",
        "context": "The GroundednessEvaluator assesses whether the model's response is grounded in the provided context. It returns a score from 1 to 5, where 5 indicates the response is fully grounded.",
        "ground_truth": "The GroundednessEvaluator uses a 1 to 5 scale, where 5 means fully grounded in the context.",
        "category": "factual",
        "difficulty": "easy"
    },
    {
        "query": "What is the default authentication method for Azure AI Foundry?",
        "context": "Azure AI Foundry supports Microsoft Entra ID (formerly Azure Active Directory) authentication. The recommended approach is using DefaultAzureCredential which supports managed identity, Azure CLI, and other credential types.",
        "ground_truth": "Azure AI Foundry uses Microsoft Entra ID authentication, with DefaultAzureCredential as the recommended approach.",
        "category": "factual",
        "difficulty": "medium"
    },
    {
        "query": "How many built-in quality evaluators does the azure-ai-evaluation SDK provide?",
        "context": "The SDK includes several built-in evaluators: GroundednessEvaluator, RelevanceEvaluator, CoherenceEvaluator, FluencyEvaluator, SimilarityEvaluator, and F1ScoreEvaluator for quality assessment. It also includes ContentSafetyEvaluator for safety metrics.",
        "ground_truth": "The SDK provides 6 built-in quality evaluators (Groundedness, Relevance, Coherence, Fluency, Similarity, F1 Score) plus a ContentSafetyEvaluator for safety.",
        "category": "factual",
        "difficulty": "medium"
    },
    {
        "query": "What format does the evaluate() function accept for input data?",
        "context": "The evaluate() function accepts data in JSONL file format or as a pandas DataFrame. Each row should contain the required fields for the configured evaluators, such as query, context, response, and optionally ground_truth.",
        "ground_truth": "The evaluate() function accepts JSONL files or pandas DataFrames with fields like query, context, response, and ground_truth.",
        "category": "factual",
        "difficulty": "easy"
    },

    # HOW-TO examples
    {
        "query": "How do I run an evaluation in Azure AI Foundry?",
        "context": "To run an evaluation, install the azure-ai-evaluation package, configure your project credentials, prepare a JSONL dataset, instantiate evaluators, and call the evaluate() function with your data and evaluator configuration.",
        "ground_truth": "Install azure-ai-evaluation, configure credentials with DefaultAzureCredential, prepare a JSONL dataset, instantiate evaluators (e.g., GroundednessEvaluator), and call evaluate() with your data and evaluators.",
        "category": "how-to",
        "difficulty": "medium"
    },
    {
        "query": "How do I view evaluation results in the AI Foundry portal?",
        "context": "After running an evaluation with the azure_ai_project parameter, results are automatically logged to the portal. Navigate to your project in the AI Foundry portal at ai.azure.com, then click on the Evaluation section to see run summaries, per-row details, and run comparisons.",
        "ground_truth": "Go to ai.azure.com, navigate to your project, and click the Evaluation section. Results appear automatically when you pass azure_ai_project to the evaluate() function.",
        "category": "how-to",
        "difficulty": "easy"
    },
    {
        "query": "How do I create a custom evaluator in Azure AI Foundry?",
        "context": "Custom evaluators can be created using prompty files that define the judge prompt and scoring rubric. You can also create code-based evaluators by subclassing the base evaluator class and implementing the evaluate method.",
        "ground_truth": "Create a .prompty file defining your judge prompt and rubric, or create a code-based evaluator by implementing a class with an evaluate method. Use prompty files for LLM-based evaluation and code for deterministic checks.",
        "category": "how-to",
        "difficulty": "hard"
    },
    {
        "query": "How do I set up a CI/CD evaluation gate?",
        "context": "An evaluation gate checks quality metrics after each evaluation run and blocks deployment if metrics fall below thresholds. Implement a script that runs evaluate(), checks results against thresholds, and returns exit code 0 (pass) or 1 (fail).",
        "ground_truth": "Create a script that runs evaluate(), compares metric scores against defined thresholds (e.g., groundedness >= 3.5), and exits with code 0 for pass or 1 for fail. Integrate this script into your CI/CD pipeline.",
        "category": "how-to",
        "difficulty": "hard"
    },
    {
        "query": "How do I install the azure-ai-evaluation package?",
        "context": "The azure-ai-evaluation package is available on PyPI and can be installed using pip. You also need azure-identity for authentication.",
        "ground_truth": "Run 'pip install azure-ai-evaluation azure-identity' to install the evaluation SDK and authentication package.",
        "category": "how-to",
        "difficulty": "easy"
    },

    # COMPARISON examples
    {
        "query": "What is the difference between GroundednessEvaluator and SimilarityEvaluator?",
        "context": "GroundednessEvaluator checks if claims in the response are supported by the provided context. SimilarityEvaluator measures semantic closeness between the response and a ground truth reference answer. Groundedness does not require ground truth, while Similarity does.",
        "ground_truth": "GroundednessEvaluator checks if the response is supported by context (no ground truth needed). SimilarityEvaluator measures how close the response is to a reference ground truth answer (requires ground truth).",
        "category": "comparison",
        "difficulty": "medium"
    },
    {
        "query": "When should I use F1ScoreEvaluator vs SimilarityEvaluator?",
        "context": "F1ScoreEvaluator measures token-level overlap between response and ground truth, returning a 0-1 score. SimilarityEvaluator uses an LLM to assess semantic similarity on a 1-5 scale. F1 is deterministic and fast but misses paraphrases. Similarity captures meaning but costs more.",
        "ground_truth": "Use F1ScoreEvaluator for fast, deterministic token overlap checks (0-1 scale). Use SimilarityEvaluator for semantic meaning comparison using an LLM (1-5 scale). F1 misses valid paraphrases; Similarity is more nuanced but slower and more expensive.",
        "category": "comparison",
        "difficulty": "hard"
    },
    {
        "query": "Compare manual evaluation vs automated evaluation pipelines.",
        "context": "Manual evaluation involves humans reviewing AI outputs one by one. Automated evaluation uses programmatic evaluators to score outputs against datasets. Manual is higher quality but doesn't scale. Automated is consistent, repeatable, and can run in CI/CD.",
        "ground_truth": "Manual evaluation provides higher quality judgments but doesn't scale. Automated pipelines are consistent, repeatable, and can integrate into CI/CD for continuous quality checks. Best practice is to use automated for routine checks and manual for periodic calibration.",
        "category": "comparison",
        "difficulty": "medium"
    },
    {
        "query": "What are the tradeoffs between using GPT-4o vs GPT-4o-mini as an LLM judge?",
        "context": "GPT-4o provides more accurate and nuanced evaluation judgments due to its superior reasoning capabilities. GPT-4o-mini is faster and cheaper but may miss subtle quality issues. For production evaluation pipelines, the cost of using GPT-4o as a judge is typically justified.",
        "ground_truth": "GPT-4o is more accurate as a judge but costs more. GPT-4o-mini is faster and cheaper but may miss subtle issues. For production evaluations, GPT-4o is recommended since evaluation accuracy is critical and the volume is typically manageable.",
        "category": "comparison",
        "difficulty": "hard"
    },
    {
        "query": "Compare absolute thresholds vs relative delta for regression detection.",
        "context": "Absolute thresholds set fixed minimum scores (e.g., groundedness >= 3.5). Relative delta flags drops from the previous run (e.g., any metric drops > 0.3). Absolute prevents deploying below a quality floor. Relative catches sudden degradation even at high quality levels.",
        "ground_truth": "Absolute thresholds block deployment below a quality floor (e.g., groundedness < 3.5). Relative delta catches sudden drops from the previous run (e.g., > 0.3 decrease). Use both together: absolute for minimum quality and relative for regression detection.",
        "category": "comparison",
        "difficulty": "medium"
    },

    # OUT-OF-SCOPE examples
    {
        "query": "Can you write my college essay about Shakespeare?",
        "context": "This AI assistant is designed to help with Azure AI Foundry topics including evaluation, deployment, and development of AI applications.",
        "ground_truth": "I'm an Azure AI Foundry assistant and can help with AI evaluation, deployment, and development topics. For essay writing, please consult an appropriate writing resource.",
        "category": "out-of-scope",
        "difficulty": "easy"
    },
    {
        "query": "What is the stock price of Microsoft today?",
        "context": "This AI assistant provides information about Azure AI Foundry evaluation datasets and quality measurement.",
        "ground_truth": "I don't have access to real-time stock prices. I can help with Azure AI Foundry evaluation and quality measurement topics. For stock information, please check a financial data provider.",
        "category": "out-of-scope",
        "difficulty": "easy"
    },
    {
        "query": "How do I train a custom model from scratch using Azure Machine Learning?",
        "context": "Azure AI Foundry focuses on building AI applications using pre-trained models, evaluation, and deployment. Azure Machine Learning is a separate service for custom model training.",
        "ground_truth": "Custom model training from scratch is handled by Azure Machine Learning, not Azure AI Foundry. AI Foundry focuses on building applications with pre-trained models, evaluation, and deployment. Check the Azure ML documentation for custom training workflows.",
        "category": "out-of-scope",
        "difficulty": "medium"
    },
    {
        "query": "What medications should I take for my headache?",
        "context": "This assistant is designed for Azure AI Foundry technical topics only.",
        "ground_truth": "I'm not qualified to provide medical advice. Please consult a healthcare professional for medication questions. I can help with Azure AI Foundry evaluation and quality measurement topics.",
        "category": "out-of-scope",
        "difficulty": "easy"
    },
    {
        "query": "Explain quantum computing in simple terms.",
        "context": "This AI assistant specializes in Azure AI Foundry topics including evaluation datasets, quality metrics, and deployment best practices.",
        "ground_truth": "Quantum computing is outside my specialized domain. I focus on Azure AI Foundry topics like evaluation datasets, quality metrics, and deployment. For quantum computing, check educational resources like Microsoft Quantum documentation.",
        "category": "out-of-scope",
        "difficulty": "easy"
    }
]

# Write to JSONL
with open("eval_dataset.jsonl", "w") as f:
    for example in examples:
        f.write(json.dumps(example) + "\n")

print(f"✅ Created eval_dataset.jsonl with {len(examples)} examples")

# Print category distribution
from collections import Counter
categories = Counter(e["category"] for e in examples)
for cat, count in sorted(categories.items()):
    print(f"   {cat}: {count} examples")
```

Run the script:
```bash
python create_dataset.py
```

---

## Part 3: Generate Responses (15 minutes)

### Step 3.1 — Generate AI Responses for Each Query

```python
# generate_responses.py
import json
from openai import AzureOpenAI
from azure.identity import DefaultAzureCredential, get_bearer_token_provider
from config import MODEL_DEPLOYMENT, MODEL_ENDPOINT

credential = DefaultAzureCredential()
token_provider = get_bearer_token_provider(
    credential, "https://cognitiveservices.azure.com/.default"
)

client = AzureOpenAI(
    azure_endpoint=MODEL_ENDPOINT,
    azure_ad_token_provider=token_provider,
    api_version="2024-12-01-preview"
)

# Load dataset
with open("eval_dataset.jsonl") as f:
    examples = [json.loads(line) for line in f]

# Generate responses
results = []
for i, ex in enumerate(examples):
    print(f"Generating response {i+1}/{len(examples)}...")
    response = client.chat.completions.create(
        model=MODEL_DEPLOYMENT,
        messages=[
            {"role": "system", "content": f"Answer based on this context: {ex['context']}"},
            {"role": "user", "content": ex["query"]}
        ],
        max_tokens=300,
        temperature=0.3
    )
    ex["response"] = response.choices[0].message.content
    results.append(ex)

# Save with responses
with open("eval_dataset_with_responses.jsonl", "w") as f:
    for r in results:
        f.write(json.dumps(r) + "\n")

print(f"✅ Generated {len(results)} responses → eval_dataset_with_responses.jsonl")
```

---

## Part 4: Run Evaluators (15 minutes)

### Step 4.1 — Run Built-in Evaluators

```python
# run_evaluation.py
from azure.ai.evaluation import evaluate
from azure.ai.evaluation import (
    GroundednessEvaluator,
    RelevanceEvaluator,
    CoherenceEvaluator,
    SimilarityEvaluator
)
from azure.identity import DefaultAzureCredential
from config import AZURE_AI_PROJECT

credential = DefaultAzureCredential()

results = evaluate(
    data="eval_dataset_with_responses.jsonl",
    evaluators={
        "groundedness": GroundednessEvaluator(
            credential=credential, azure_ai_project=AZURE_AI_PROJECT
        ),
        "relevance": RelevanceEvaluator(
            credential=credential, azure_ai_project=AZURE_AI_PROJECT
        ),
        "coherence": CoherenceEvaluator(
            credential=credential, azure_ai_project=AZURE_AI_PROJECT
        ),
        "similarity": SimilarityEvaluator(
            credential=credential, azure_ai_project=AZURE_AI_PROJECT
        )
    },
    evaluator_config={
        "groundedness": {
            "query": "${data.query}",
            "context": "${data.context}",
            "response": "${data.response}"
        },
        "relevance": {
            "query": "${data.query}",
            "response": "${data.response}"
        },
        "coherence": {
            "response": "${data.response}"
        },
        "similarity": {
            "query": "${data.query}",
            "response": "${data.response}",
            "ground_truth": "${data.ground_truth}"
        }
    },
    azure_ai_project=AZURE_AI_PROJECT,
    output_path="./eval_output"
)

# Print aggregate metrics
print("\n=== Evaluation Results ===")
for metric, value in sorted(results["metrics"].items()):
    print(f"  {metric}: {value:.3f}")
```

---

## Part 5: Analyze Results (15 minutes)

### Step 5.1 — Find Low-Scoring Examples

```python
# analyze_results.py
import pandas as pd

df = pd.DataFrame(results["rows"])

# Find lowest groundedness scores
print("=== Low Groundedness (< 3.0) ===")
low = df[df["outputs.groundedness.groundedness"] < 3.0]
for _, row in low.iterrows():
    print(f"  Query: {row['inputs.query'][:80]}")
    print(f"  Score: {row['outputs.groundedness.groundedness']}")
    print(f"  Category: {row['inputs.category']}")
    print()

# Category-level breakdown
print("=== Scores by Category ===")
for cat in df["inputs.category"].unique():
    subset = df[df["inputs.category"] == cat]
    avg_g = subset["outputs.groundedness.groundedness"].mean()
    avg_r = subset["outputs.relevance.relevance"].mean()
    print(f"  {cat}: groundedness={avg_g:.2f}, relevance={avg_r:.2f}")
```

### Step 5.2 — Document Findings

For each low-scoring example, answer:
1. **What went wrong?** (e.g., hallucinated facts, missed key points, wrong format)
2. **Root cause?** (e.g., insufficient context, ambiguous query, prompt issue)
3. **Fix:** What would you change? (prompt, context, or dataset)

---

## Part 6: Stretch Goal — Custom LLM-as-a-Judge (15 minutes)

### Step 6.1 — Create a Custom Evaluator

Create a file called `helpfulness_evaluator.prompty`:

```
---
name: Helpfulness Evaluator
description: Evaluates how helpful the response is for the user's task.
model:
  api: chat
  parameters:
    temperature: 0
inputs:
  query:
    type: string
  response:
    type: string
outputs:
  score:
    type: int
  reasoning:
    type: string
---
system:
You are an expert evaluator. Score the helpfulness of an AI response.

## Scoring Rubric
- 5: Exceptionally helpful — directly solves the user's problem with clear, actionable guidance
- 4: Helpful — addresses the question well with minor room for improvement
- 3: Somewhat helpful — partially addresses the question but misses key aspects
- 2: Minimally helpful — vaguely related but doesn't address the core question
- 1: Not helpful — irrelevant, confusing, or incorrect

## Input
Query: {{query}}
Response: {{response}}

## Output
Return ONLY a JSON object: {"score": <1-5>, "reasoning": "<brief explanation>"}
```

---

## Completion Checklist

- [ ] Environment set up with `azure-ai-evaluation` installed
- [ ] Created `eval_dataset.jsonl` with 20+ examples across 4 categories
- [ ] Generated responses using AI Foundry deployment
- [ ] Ran 4 evaluators (Groundedness, Relevance, Coherence, Similarity)
- [ ] Identified at least 3 low-scoring examples with documented root causes
- [ ] Verified results appear in AI Foundry portal Evaluation tab
- [ ] (Stretch) Created and ran a custom LLM-as-a-Judge evaluator

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `AuthenticationError` | Run `az login` to authenticate with Azure CLI |
| `ResourceNotFoundError` | Verify project name and resource group in `config.py` |
| Low scores across all examples | Check that context field contains relevant information |
| `ModuleNotFoundError` | Run `pip install azure-ai-evaluation azure-identity` |
| Results not showing in portal | Ensure `azure_ai_project` is passed to `evaluate()` |

---

**🎉 Congratulations!** You've completed the final lab of the Beginner tier. You now have the skills to systematically measure and improve AI quality in Azure AI Foundry.
