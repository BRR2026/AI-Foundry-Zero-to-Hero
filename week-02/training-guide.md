# Week 2: Understanding the AI Lifecycle

## **ARC 1: Foundations** | Week 2 of 45 — Zero to Hero: Azure AI Foundry

---

| Detail              | Info                                                    |
|---------------------|---------------------------------------------------------|
| **Arc**             | ARC 1 — Foundations (Weeks 1–5)                         |
| **Week**            | 2                                                       |
| **Title**           | Understanding the AI Lifecycle                          |
| **Estimated Time**  | 3–4 hours                                               |
| **Difficulty**      | ⭐ Beginner                                              |
| **Prerequisites**   | Week 1: What is AI Foundry?                             |
| **Last Updated**    | July 2025                                               |

---

## 🎯 Learning Objectives

By the end of this week, you will be able to:

1. **Describe** the 6 stages of the AI development lifecycle from data collection through monitoring.
2. **Map** each lifecycle stage to specific Azure AI Foundry tools and features.
3. **Differentiate** between training, fine-tuning, and prompt engineering — and know when to use each.
4. **Compare** model selection strategies: pre-built, fine-tuned, and custom models.
5. **Identify** key evaluation metrics: accuracy, groundedness, relevance, coherence, and safety.
6. **Explain** deployment patterns available in Azure AI Foundry (managed endpoints, serverless APIs, containers).
7. **Describe** MLOps concepts and why they matter for production AI.
8. **Set up** an Azure account and AI Foundry project (hands-on lab).

---

## 📋 Prerequisites

- **Completed Week 1:** You should understand what Azure AI Foundry is and its key capabilities.
- **A Microsoft account** (free). If you don't have one, see [account.microsoft.com](https://account.microsoft.com).
- **Willingness to create an Azure account** — we'll walk through this step-by-step in the hands-on lab.
- **No AI/ML experience required** — we're building from the ground up.

> 💡 **Tip:** This week bridges *understanding* and *doing*. We'll explain the full AI lifecycle conceptually, then you'll set up your Azure environment and take your first hands-on steps in AI Foundry.

---

## 1. Introduction — From Idea to Production

### The Journey Every AI Application Takes

In Week 1, we learned that Azure AI Foundry is a unified platform — a "factory" for building AI applications. But every factory needs a *process*. Cars don't appear on the showroom floor by magic — they go through design, engineering, assembly, quality control, and delivery.

AI applications are exactly the same.

Whether you're building a customer support chatbot, a document summarizer, a code assistant, or an image classifier, **every AI application follows the same fundamental journey**:

```
  DATA  →  PREPARATION  →  MODEL  →  EVALUATION  →  DEPLOYMENT  →  MONITORING
   📊         🔧           🧠          ✅              🚀             👁️
```

This journey is called the **AI Development Lifecycle**, and understanding it is the single most important foundation for everything you'll learn in this 45-week course.

### Why This Matters

Here's a truth that surprises many beginners: **the model is often the easiest part.** Thanks to platforms like AI Foundry, you can deploy a world-class language model in minutes. The hard parts are:

- Getting the **right data** in the right format
- **Evaluating** whether your AI actually works well
- **Deploying** it reliably at scale
- **Monitoring** it so it doesn't drift, break, or produce harmful outputs

> 📝 **Note:** Studies consistently show that data scientists spend **60-80% of their time** on data preparation — not model building. Understanding the full lifecycle helps you plan your time and resources wisely.

### The Manufacturing Analogy

Let's extend Week 1's factory analogy:

| Manufacturing Step       | AI Lifecycle Step         | What Happens                                |
|--------------------------|---------------------------|---------------------------------------------|
| Source raw materials      | **Data Collection**       | Gather the data your AI needs               |
| Refine materials          | **Data Preparation**      | Clean, label, and format data               |
| Design the product        | **Model Selection/Training** | Choose and configure the AI model        |
| Quality inspection        | **Evaluation**            | Test if the AI meets standards              |
| Ship to customers         | **Deployment**            | Make the AI available to users              |
| Customer feedback loop    | **Monitoring**            | Track performance, fix issues, improve      |

A car manufacturer doesn't skip quality inspection. An AI developer shouldn't skip evaluation. A car company tracks recalls. An AI team tracks model drift. The parallels are exact.

---

## 2. The AI Development Lifecycle — The Complete Picture

Let's look at the full lifecycle as a visual diagram:

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                        THE AI DEVELOPMENT LIFECYCLE                         │
│                                                                              │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│   │   DATA    │    │   DATA   │    │  MODEL   │    │          │              │
│   │COLLECTION│───▸│PREPARATION│───▸│SELECTION │───▸│EVALUATION│              │
│   │          │    │          │    │/TRAINING │    │          │              │
│   │  📊      │    │  🔧      │    │  🧠      │    │  ✅      │              │
│   └──────────┘    └──────────┘    └──────────┘    └──────────┘              │
│        │                                                │                    │
│        │                                                ▼                    │
│        │                                         ┌──────────┐               │
│        │         ┌───────────────────────────────▸│DEPLOYMENT│               │
│        │         │                                │          │               │
│        │         │                                │  🚀      │               │
│        │         │                                └──────────┘               │
│        │         │                                      │                    │
│        │         │                                      ▼                    │
│        │    ┌──────────┐                          ┌──────────┐               │
│        │    │ FEEDBACK │◂─────────────────────────│MONITORING│               │
│        │    │   LOOP   │                          │          │               │
│        │    │  🔄      │                          │  👁️      │               │
│        │    └──────────┘                          └──────────┘               │
│        │         │                                                           │
│        ◂─────────┘                                                           │
│   (New data, updated requirements, retraining triggers)                      │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

> 💡 **Key Insight:** Notice the arrow going from Monitoring back to Data Collection. The AI lifecycle is **not** a straight line — it's a continuous loop. Your AI gets smarter over time as you collect more data, refine your approach, and iterate.

### The 6 Stages at a Glance

| Stage | What You Do | AI Foundry Tool | Time Spent |
|-------|-------------|-----------------|------------|
| 1. Data Collection | Gather training data, documents, examples | Azure Storage, Data connections | 15-20% |
| 2. Data Preparation | Clean, label, format, split into train/test | Data assets, Index creation | 25-35% |
| 3. Model Selection/Training | Choose a model, optionally fine-tune | Model Catalog, Fine-tuning | 10-15% |
| 4. Evaluation | Test quality, safety, and accuracy | Evaluation tools, Playground | 15-20% |
| 5. Deployment | Make available via APIs or endpoints | Managed endpoints, Serverless | 5-10% |
| 6. Monitoring | Track usage, performance, and issues | Azure Monitor, Content Safety | 10-15% |

> ⚠️ **Common Mistake:** Many beginners jump straight to Stage 3 (picking a model) without thinking about data or evaluation. This almost always leads to problems later. Follow the lifecycle in order.

---

## 3. Stage 1 — Data Collection

### What is Data Collection?

Data collection is the process of gathering the raw information your AI application needs to function. This is the foundation — **your AI is only as good as your data.**

Think of it like cooking: even the best chef can't make a great meal with rotten ingredients. Even the most powerful AI model can't give great answers without good data.

### Types of Data

Data comes in three main formats, and AI Foundry can work with all of them:

```
┌─────────────────────────────────────────────────────────────┐
│                    TYPES OF DATA                            │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────────────┐    │
│  │ STRUCTURED  │  │SEMI-STRUCTURED│  │  UNSTRUCTURED  │    │
│  │             │  │             │  │                  │    │
│  │ • Databases │  │ • JSON      │  │ • Documents      │    │
│  │ • CSV files │  │ • XML       │  │ • Images         │    │
│  │ • Excel     │  │ • YAML      │  │ • Audio          │    │
│  │ • SQL tables│  │ • Log files │  │ • Video          │    │
│  │             │  │ • Email     │  │ • Free-form text │    │
│  │ Rows &      │  │ Tags &      │  │ No fixed         │    │
│  │ Columns     │  │ Hierarchies │  │ format           │    │
│  └─────────────┘  └─────────────┘  └──────────────────┘    │
│                                                             │
│    📊 Easiest      📋 Moderate      📝 Most common         │
│    to process      complexity       but hardest             │
└─────────────────────────────────────────────────────────────┘
```

| Data Type | Examples | Use in AI | AI Foundry Support |
|-----------|----------|-----------|-------------------|
| **Structured** | Database tables, CSV, spreadsheets | Classification, prediction, analytics | Azure SQL, Blob Storage, data connections |
| **Semi-structured** | JSON, XML, log files, emails | Information extraction, parsing | Blob Storage, data connections, indexing |
| **Unstructured** | PDFs, Word docs, images, audio, video | RAG, Q&A, summarization, vision | Blob Storage, Azure AI Search indexes |

### Data Sources for AI Foundry

Azure AI Foundry can connect to data from many sources:

- **Azure Blob Storage** — The most common. Upload files of any type.
- **Azure AI Search** — Create searchable indexes over your documents (critical for RAG patterns).
- **Azure SQL / Cosmos DB** — Connect to structured databases.
- **Azure Data Lake Storage** — For large-scale data sets.
- **OneLake (Microsoft Fabric)** — Enterprise data from Fabric lakehouses.
- **Upload directly** — Drag and drop files into the AI Foundry portal.

> 💡 **Tip:** For most beginner projects, simply uploading files to Azure Blob Storage and creating an Azure AI Search index is the easiest path. We'll practice this in later weeks.

### How Much Data Do You Need?

This is one of the most common questions, and the answer depends on your approach:

| Approach | Data Needed | Example |
|----------|------------|---------|
| **Prompt Engineering** | 0 training data (just examples in prompts) | Using GPT-4o with well-crafted prompts |
| **RAG (Retrieval-Augmented Generation)** | Your knowledge base documents | Company FAQ, product docs, policies |
| **Fine-tuning** | Hundreds to thousands of examples | 500+ question-answer pairs |
| **Training from scratch** | Millions of data points | Specialized domain model |

> 📝 **Note:** Most enterprise AI applications use **RAG** (Retrieval-Augmented Generation), which means connecting a pre-built model to your company's documents. You don't need to retrain the model — you just give it access to your data at query time. We'll cover RAG in depth in later weeks.

---

## 4. Stage 2 — Data Preparation

### What is Data Preparation?

Data preparation (also called "data preprocessing" or "data wrangling") is the process of transforming raw data into a clean, organized format that an AI model can use effectively.

### Why It Matters So Much

Remember: **60-80% of an AI project's time goes into data preparation.** Here's why:

```
Raw Data (Messy)              Clean Data (Ready)
─────────────────             ──────────────────
"Price: $1,234.56"      →    1234.56
"price is 1234.56"      →    1234.56
"PRICE = $1,234.56"     →    1234.56
""                      →    (removed)
"N/A"                   →    (removed)
"price: twelve hundred" →    1200.00
```

Real-world data is messy, inconsistent, incomplete, and often contradictory. If you feed messy data into an AI model, you get messy results — the classic "garbage in, garbage out" problem.

### Key Data Preparation Steps

| Step | What It Means | Example |
|------|---------------|---------|
| **Cleaning** | Remove errors, duplicates, and invalid entries | Remove rows with missing customer IDs |
| **Normalization** | Standardize formats | Convert all dates to YYYY-MM-DD format |
| **Labeling** | Tag data with categories or answers | Mark emails as "spam" or "not spam" |
| **Splitting** | Divide into training, validation, and test sets | 80% train, 10% validation, 10% test |
| **Enrichment** | Add additional context or metadata | Add product categories to sales data |
| **Tokenization** | Break text into meaningful units | "AI is great" → ["AI", "is", "great"] |
| **Chunking** | Split large documents into smaller pieces | Split a 100-page PDF into 500-word chunks |

### Data Preparation in Azure AI Foundry

For different AI approaches, AI Foundry provides different preparation tools:

| AI Approach | Data Preparation in AI Foundry |
|-------------|-------------------------------|
| **Prompt Engineering** | Write example prompts and expected outputs directly in the Playground |
| **RAG** | Upload documents → Create an Azure AI Search index → AI Foundry automatically chunks and indexes |
| **Fine-tuning** | Prepare JSONL files with prompt-completion pairs → Upload as training data |
| **Evaluation** | Create evaluation datasets with questions, ground-truth answers, and context |

> 💡 **Tip:** When building a RAG application, Azure AI Foundry can automatically chunk your documents, create embeddings (mathematical representations of text), and build a searchable index. You don't need to do this manually!

---

## 5. Stage 3 — Model Selection and Training

### The Most Exciting Stage (But Not the Most Important)

This is the stage most people think of when they hear "AI" — choosing and configuring the model. While it's exciting, remember that data quality and evaluation matter just as much.

### Three Approaches to Getting a Model

There are three main strategies, and understanding when to use each is crucial:

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    MODEL SELECTION STRATEGIES                           │
│                                                                          │
│  EFFORT: Low ◀──────────────────────────────────────────────▸ High      │
│  COST:   Low ◀──────────────────────────────────────────────▸ High      │
│  DATA:   None ◀─────────────────────────────────────────────▸ Massive   │
│                                                                          │
│  ┌──────────────┐    ┌──────────────────┐    ┌──────────────────┐       │
│  │  PRE-BUILT   │    │   FINE-TUNED     │    │     CUSTOM       │       │
│  │   MODEL      │    │    MODEL         │    │     MODEL        │       │
│  │              │    │                  │    │                  │       │
│  │ Use as-is    │    │ Pre-built +      │    │ Train from       │       │
│  │ from catalog │    │ your data        │    │ scratch          │       │
│  │              │    │                  │    │                  │       │
│  │ ⏱️ Minutes   │    │ ⏱️ Hours-Days    │    │ ⏱️ Weeks-Months  │       │
│  │ 💰 Pay per   │    │ 💰 Training +    │    │ 💰 Significant   │       │
│  │    use       │    │    hosting       │    │    investment    │       │
│  └──────────────┘    └──────────────────┘    └──────────────────┘       │
│                                                                          │
│  MOST COMMON ▸▸▸▸▸▸▸▸▸▸ SOMETIMES NEEDED ▸▸▸▸▸▸▸▸ RARELY NEEDED      │
└──────────────────────────────────────────────────────────────────────────┘
```

### Strategy 1: Pre-Built Models (Most Common)

**What it means:** Use a model directly from the AI Foundry Model Catalog with no modification.

**When to use:**
- General-purpose tasks (summarization, Q&A, translation, code generation)
- You want to get started quickly
- Your use case doesn't require specialized domain knowledge
- You plan to use prompt engineering or RAG to customize behavior

**Examples in AI Foundry:**
- Deploy GPT-4o for a general chatbot
- Use Phi-4 for a lightweight, on-device assistant
- Deploy Llama 4 for an open-weight multilingual application

> 💡 **Tip:** Start here. 90% of AI projects can be solved with a pre-built model + good prompt engineering + RAG. Only move to fine-tuning if you've hit a wall.

### Strategy 2: Fine-Tuned Models (Sometimes Needed)

**What it means:** Take a pre-built model and train it further on your specific data so it learns your domain, style, or task.

**When to use:**
- The model needs to consistently follow a specific format or style
- You need specialized domain knowledge (medical, legal, financial terminology)
- Prompt engineering alone doesn't achieve the quality you need
- You want to reduce prompt length (and cost) by "baking in" instructions

**Fine-tuning in AI Foundry:**
1. Prepare a JSONL dataset with input-output pairs
2. Go to the Model Catalog → Select a model that supports fine-tuning
3. Upload your training data
4. Configure training parameters (epochs, learning rate, batch size)
5. Run the training job
6. Deploy your fine-tuned model

**Example fine-tuning data (JSONL format):**
```json
{"messages": [{"role": "system", "content": "You are a medical assistant."}, {"role": "user", "content": "What is hypertension?"}, {"role": "assistant", "content": "Hypertension, or high blood pressure, is a chronic condition where blood pressure in the arteries is persistently elevated above 130/80 mmHg."}]}
{"messages": [{"role": "system", "content": "You are a medical assistant."}, {"role": "user", "content": "What causes type 2 diabetes?"}, {"role": "assistant", "content": "Type 2 diabetes is primarily caused by insulin resistance, where cells don't respond effectively to insulin. Key risk factors include obesity, sedentary lifestyle, genetics, and age over 45."}]}
```

### Strategy 3: Custom Models (Rarely Needed)

**What it means:** Build and train a model architecture from scratch using your own data.

**When to use:**
- Highly specialized tasks where no existing model works
- Proprietary data that requires a completely new model architecture
- Edge computing where you need an extremely small, specialized model
- Research and academic use cases

**In AI Foundry:** Custom training typically uses the underlying Azure Machine Learning compute, which is accessible from within AI Foundry projects.

> ⚠️ **Warning:** Training a model from scratch requires significant ML expertise, large datasets, and substantial compute costs. For 99% of business applications, pre-built + fine-tuning is sufficient.

### Decision Flowchart: Which Strategy Should I Use?

```
                    Start Here
                        │
                        ▼
            ┌───────────────────────┐
            │ Can a pre-built model │
            │ handle your task?     │
            └───────────┬───────────┘
                   Yes  │  No
                   ▼    │  ▼
            ┌──────────┐│┌──────────────────────┐
            │Use it +  │││ Is the issue about    │
            │prompt    │││ style/format/domain?  │
            │engineering││└──────────┬───────────┘
            │+ RAG     ││      Yes  │  No
            └──────────┘│      ▼    │  ▼
                        │┌──────────┐│┌──────────────┐
                        ││Fine-tune │││Build custom  │
                        ││a model   │││model         │
                        │└──────────┘│└──────────────┘
                        │            │
                        ▼            ▼
                   90% of cases   <1% of cases
```

### Training vs Fine-Tuning vs Prompt Engineering

These three terms are often confused. Here's a clear comparison:

| Aspect | Prompt Engineering | Fine-Tuning | Training from Scratch |
|--------|-------------------|-------------|----------------------|
| **What changes** | The input to the model | The model's weights (slightly) | The entire model |
| **Data needed** | Zero (just write better prompts) | Hundreds to thousands of examples | Millions+ data points |
| **Time to implement** | Minutes to hours | Hours to days | Weeks to months |
| **Cost** | Free (just API calls) | Moderate (compute for training) | Very high (massive compute) |
| **Expertise needed** | Beginner | Intermediate | Expert (ML engineer) |
| **Model modified?** | No | Yes (adapted) | Yes (created) |
| **Best for** | Most tasks, getting started | Domain specialization, style | Novel architectures |
| **AI Foundry tool** | Playground, Prompt Flow | Fine-tuning in Model Catalog | Azure ML Compute |

> 💡 **Analogy:** Think of prompt engineering as *talking to a smart assistant*, fine-tuning as *sending them to a specialized training course*, and training from scratch as *raising a child from birth*.

---

## 6. Stage 4 — Evaluation

### Why Evaluation is Critical

Imagine shipping a car that hasn't been crash-tested. Or releasing medicine that hasn't been through clinical trials. That's what deploying an AI without evaluation is like.

Evaluation answers the question: **"Does my AI actually work well enough to put in front of real users?"**

### Key Evaluation Metrics

Azure AI Foundry provides built-in evaluation tools that measure these key metrics:

| Metric | What It Measures | Scale | Example |
|--------|-----------------|-------|---------|
| **Groundedness** | Are answers based on provided context/facts? | 1-5 | Does the AI make up information, or stick to source data? |
| **Relevance** | Does the answer address the user's question? | 1-5 | If asked about pricing, does it actually talk about pricing? |
| **Coherence** | Is the answer well-organized and readable? | 1-5 | Is the response logical, or jumbled and confusing? |
| **Fluency** | Is the language natural and grammatically correct? | 1-5 | Does it read like a human wrote it? |
| **Similarity** | Does the answer match the expected "ground truth" answer? | 1-5 | How close is it to the known correct answer? |
| **Safety** | Does the output avoid harmful content? | Pass/Fail | No hate speech, violence, self-harm, or sexual content |

```
┌──────────────────────────────────────────────────────────┐
│                  EVALUATION METRICS                      │
│                                                          │
│   Quality Metrics (1-5 scale)     Safety Metrics (P/F)   │
│   ┌─────────────────────────┐     ┌──────────────────┐   │
│   │ Groundedness    ████░ 4 │     │ Hate Speech   ✅ │   │
│   │ Relevance       █████ 5 │     │ Violence      ✅ │   │
│   │ Coherence       ████░ 4 │     │ Self-Harm     ✅ │   │
│   │ Fluency         █████ 5 │     │ Sexual        ✅ │   │
│   │ Similarity      ███░░ 3 │     │ Jailbreak     ✅ │   │
│   └─────────────────────────┘     └──────────────────┘   │
│                                                          │
│   Overall Quality Score: 4.2/5.0    Safety: PASS         │
└──────────────────────────────────────────────────────────┘
```

### Types of Evaluation

| Evaluation Type | How It Works | Best For |
|----------------|-------------|----------|
| **Manual** | Humans review AI outputs and rate quality | Small-scale testing, subjective quality |
| **Automated (AI-assisted)** | An AI model (like GPT-4) evaluates another model's outputs | Large-scale testing, consistent scoring |
| **Metric-based** | Compute statistical metrics (BLEU, ROUGE, F1) | NLP tasks with known correct answers |
| **Red-teaming** | Deliberately try to break the AI with adversarial inputs | Security and safety testing |
| **A/B testing** | Compare two model versions with real users | Production optimization |

### Evaluation in Azure AI Foundry

AI Foundry makes evaluation accessible with built-in tools:

1. **Playground Testing** — Manually test prompts and review outputs
2. **Evaluation Runs** — Upload a dataset of test questions and expected answers, then run automated evaluation
3. **Custom Evaluators** — Define your own evaluation criteria using Python or prompt-based evaluators
4. **Comparative Evaluation** — Run the same test set against multiple models to compare performance

> 📝 **Note:** Evaluation isn't a one-time event. You should evaluate continuously — before deployment, after deployment, and whenever you change your prompts, data, or model.

---

## 7. Stage 5 — Deployment

### What is Deployment?

Deployment is the process of making your AI model available for real-world use — turning a prototype into a product. In Azure AI Foundry, this means creating an **endpoint** that applications can call via API.

### Deployment Patterns in Azure AI Foundry

AI Foundry offers three main deployment approaches:

```
┌──────────────────────────────────────────────────────────────────────┐
│                    DEPLOYMENT PATTERNS                              │
│                                                                      │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐         │
│  │   SERVERLESS   │  │    MANAGED     │  │   CONTAINER    │         │
│  │   API (MaaS)   │  │   COMPUTE     │  │   DEPLOYMENT   │         │
│  │                │  │   (MaaP)      │  │                │         │
│  │ Pay-per-token  │  │ Dedicated VM  │  │ Your infra     │         │
│  │ No infra mgmt  │  │ Full control  │  │ Full portability│        │
│  │ Instant deploy │  │ Custom scaling│  │ Any cloud/edge │         │
│  │                │  │               │  │                │         │
│  │ Best for:      │  │ Best for:     │  │ Best for:      │         │
│  │ • Prototyping  │  │ • Production  │  │ • Hybrid cloud │         │
│  │ • Variable     │  │ • Consistent  │  │ • Air-gapped   │         │
│  │   traffic      │  │   workloads   │  │   environments │         │
│  │ • Quick start  │  │ • Compliance  │  │ • Edge deploy  │         │
│  └────────────────┘  └────────────────┘  └────────────────┘         │
│                                                                      │
│  Simplest ◀────────────────────────────────────────────▸ Most Flexible│
└──────────────────────────────────────────────────────────────────────┘
```

### Detailed Comparison

| Feature | Serverless API (MaaS) | Managed Compute (MaaP) | Container Deployment |
|---------|----------------------|----------------------|---------------------|
| **Billing model** | Pay per token/request | Pay for VM uptime | Your infrastructure costs |
| **Setup time** | Minutes | 10-30 minutes | Hours (build + deploy) |
| **Scaling** | Automatic | Configurable auto-scale | Manual or orchestrated |
| **GPU management** | Not needed | Azure-managed | Self-managed |
| **Model customization** | Limited | Full fine-tuning | Full control |
| **Data residency** | Azure regions | Your chosen region | Any location |
| **Offline support** | No | No | Yes |
| **Best for** | Development, variable loads | Production, steady loads | Hybrid, edge, regulated |

### How to Deploy in AI Foundry

The basic deployment flow:

1. **Go to Model Catalog** → Browse or search for a model
2. **Click "Deploy"** → Choose deployment type (Serverless or Managed)
3. **Configure settings** → Name, region, content filters
4. **Deploy** → AI Foundry provisions the endpoint
5. **Get endpoint URL and key** → Use in your application

> 💡 **Tip for Beginners:** Start with **Serverless API** deployment. It's the fastest way to get a model running, and you only pay for what you use. Move to Managed Compute when you need more control or predictable pricing.

### What You Get After Deployment

After deploying a model, AI Foundry gives you:

- **Endpoint URL** — The API address your application calls
- **API Key** — Authentication credential (keep this secret!)
- **SDK code samples** — Python, C#, JavaScript snippets ready to copy
- **Playground access** — Test the deployed model interactively
- **Monitoring dashboard** — See usage, latency, errors in real time

---

## 8. Stage 6 — Monitoring

### Why Monitoring Matters

Deploying your AI is not the finish line — it's the starting line. Production AI systems need continuous monitoring because:

1. **Models can drift** — Performance may degrade as real-world data patterns change
2. **Users are unpredictable** — Real users will ask things you never anticipated
3. **Safety risks emerge** — Adversarial users may try to exploit your AI
4. **Costs can spike** — Unexpected traffic can run up your bill
5. **Regulations evolve** — Compliance requirements may change

### What to Monitor

| Category | What to Track | Why It Matters |
|----------|--------------|----------------|
| **Performance** | Latency, throughput, error rates | Users expect fast, reliable responses |
| **Quality** | Groundedness, relevance, coherence | Ensure AI outputs stay accurate over time |
| **Safety** | Content filter triggers, jailbreak attempts | Protect users and your brand |
| **Usage** | Request volume, token consumption, costs | Budget management and capacity planning |
| **User Feedback** | Thumbs up/down, escalations, complaints | Direct signal about user satisfaction |

### Monitoring in Azure AI Foundry

```
┌──────────────────────────────────────────────────────────┐
│                  MONITORING DASHBOARD                    │
│                                                          │
│   Request Volume (24h)           Avg Latency (24h)       │
│   ┌──────────────────┐          ┌──────────────────┐     │
│   │    ▂▃▅▇█▇▅▃▂▂▃▅  │          │ 245ms (P50)      │     │
│   │                  │          │ 890ms (P95)      │     │
│   │  12,847 requests │          │ 1.2s  (P99)      │     │
│   └──────────────────┘          └──────────────────┘     │
│                                                          │
│   Content Safety                 Token Usage             │
│   ┌──────────────────┐          ┌──────────────────┐     │
│   │ ✅ 99.7% passed  │          │ Input:  2.4M     │     │
│   │ ⚠️  0.2% filtered│          │ Output: 1.1M     │     │
│   │ 🛑 0.1% blocked  │          │ Cost:   $34.20   │     │
│   └──────────────────┘          └──────────────────┘     │
└──────────────────────────────────────────────────────────┘
```

### The Feedback Loop

Monitoring feeds directly back into the lifecycle:

- **Low quality scores?** → Go back to Data Preparation (improve your data) or Model Selection (try a different model)
- **High latency?** → Optimize your deployment (scale up, use a faster model)
- **Safety issues?** → Tighten content filters, add guardrails
- **Users asking unexpected questions?** → Expand your knowledge base, update prompts
- **Cost too high?** → Use a smaller model, optimize token usage

> 📝 **Note:** The most successful AI applications have **tight feedback loops**. The faster you can detect issues and iterate, the better your AI will perform.

---

## 9. How the Lifecycle Maps to Azure AI Foundry

Let's bring it all together by mapping every lifecycle stage to specific AI Foundry tools:

| Lifecycle Stage | AI Foundry Feature | What It Does |
|----------------|-------------------|--------------|
| **Data Collection** | Data connections, Azure Storage | Connect to your data sources |
| **Data Collection** | Upload in portal | Drag-and-drop file upload |
| **Data Preparation** | Index creation | Automatically chunk and index documents |
| **Data Preparation** | Data assets | Manage datasets for fine-tuning and evaluation |
| **Model Selection** | Model Catalog | Browse 1,800+ models from OpenAI, Meta, Microsoft, Mistral, etc. |
| **Model Selection** | Model benchmarks | Compare model performance on standard benchmarks |
| **Training/Fine-tuning** | Fine-tuning | Train models on your data within the platform |
| **Training/Fine-tuning** | Prompt engineering (Playground) | Craft and iterate on prompts without training |
| **Evaluation** | Evaluation tools | Automated quality and safety assessment |
| **Evaluation** | Playground | Manual, interactive testing |
| **Deployment** | Serverless API | Deploy with pay-per-token pricing |
| **Deployment** | Managed compute | Deploy on dedicated infrastructure |
| **Monitoring** | Azure Monitor integration | Track performance, errors, costs |
| **Monitoring** | Content Safety | Real-time safety filtering |
| **Feedback Loop** | Tracing | Trace requests end-to-end for debugging |

```
┌──────────────────────────────────────────────────────────────────┐
│              AZURE AI FOUNDRY — LIFECYCLE MAPPING                │
│                                                                  │
│  ┌─ ai.azure.com ─────────────────────────────────────────────┐  │
│  │                                                             │  │
│  │  📊 DATA           🧠 MODEL          🚀 DEPLOY             │  │
│  │  ┌───────────┐     ┌───────────┐     ┌───────────┐         │  │
│  │  │ Storage   │     │ Catalog   │     │ Endpoints │         │  │
│  │  │ Indexes   │────▸│ Fine-tune │────▸│ Serverless│         │  │
│  │  │ Assets    │     │ Playground│     │ Managed   │         │  │
│  │  └───────────┘     └───────────┘     └───────────┘         │  │
│  │                                           │                 │  │
│  │  ✅ EVALUATE        👁️ MONITOR            │                 │  │
│  │  ┌───────────┐     ┌───────────┐          │                 │  │
│  │  │ Eval Runs │     │ Metrics   │◂─────────┘                 │  │
│  │  │ Benchmarks│     │ Safety    │                            │  │
│  │  │ Custom    │     │ Tracing   │                            │  │
│  │  └───────────┘     └───────────┘                            │  │
│  │                                                             │  │
│  └─────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 10. MLOps — Why It Matters

### What is MLOps?

**MLOps** (Machine Learning Operations) is the practice of applying DevOps principles — automation, version control, CI/CD, monitoring — to the AI/ML lifecycle.

Think of it this way: traditional software development has DevOps. AI development has **MLOps**.

### Why Do You Need MLOps?

Without MLOps, AI projects face these problems:

| Problem | Without MLOps | With MLOps |
|---------|---------------|------------|
| **Reproducibility** | "It worked on my laptop" | Versioned models, data, and code |
| **Deployment speed** | Manual, error-prone deploys | Automated CI/CD pipelines |
| **Model quality** | Quality degrades silently | Automated testing and monitoring |
| **Collaboration** | Data scientists work in silos | Shared workflows and artifacts |
| **Compliance** | "Who changed what?" unknown | Full audit trail and governance |
| **Scale** | Managing 1-2 models manually | Managing hundreds of models at scale |

### MLOps Principles

| Principle | What It Means | AI Foundry Support |
|-----------|---------------|-------------------|
| **Version Control** | Track changes to models, data, and prompts | Model versioning, data assets |
| **Automation** | Automate testing, deployment, monitoring | GitHub Actions integration, pipelines |
| **Monitoring** | Continuously track production performance | Azure Monitor, content safety logs |
| **Reproducibility** | Any result can be recreated | Environment configs, compute specs |
| **Testing** | Validate before deploying changes | Evaluation runs, benchmarks |
| **Governance** | Control access, audit changes | RBAC, Azure Policy, audit logs |

> 💡 **Tip:** You don't need to implement full MLOps from day one. Start with the basics: version your prompts, evaluate before deploying, and monitor in production. Add more automation as you grow.

### The MLOps Maturity Model

| Level | Name | Characteristics |
|-------|------|-----------------|
| **0** | Manual | Everything done by hand, no automation |
| **1** | Basic | Some automated testing, manual deployment |
| **2** | Intermediate | Automated CI/CD, monitoring dashboards |
| **3** | Advanced | Full automation, A/B testing, auto-scaling |
| **4** | Expert | Auto-retraining, drift detection, self-healing |

> 📝 **Note:** Most teams should aim for Level 2 initially. Levels 3-4 are for teams managing many models at scale.

---

## 11. Real-World Examples — The Full Lifecycle in Action

### Example 1: Customer Support Chatbot

Let's walk through the entire lifecycle for a real-world application:

**Business Problem:** A retail company wants to reduce customer support ticket volume by 40% using an AI chatbot.

| Stage | What They Did | AI Foundry Tool Used |
|-------|--------------|---------------------|
| **1. Data Collection** | Exported 50,000 historical support tickets, product manuals, FAQ documents, and return policies | Azure Blob Storage, data connections |
| **2. Data Preparation** | Cleaned tickets (removed PII), chunked product manuals into 500-word sections, created AI Search index | Azure AI Search, data assets |
| **3. Model Selection** | Chose GPT-4o for accuracy, used RAG pattern to ground answers in company data | Model Catalog, Playground |
| **4. Evaluation** | Created 200-question test set from real tickets, evaluated groundedness (4.3/5), relevance (4.6/5), safety (100% pass) | Evaluation tools |
| **5. Deployment** | Deployed as Serverless API, added content safety filters, integrated with company's chat widget | Serverless endpoint |
| **6. Monitoring** | Tracked resolution rate, customer satisfaction, escalation rate, cost per conversation | Azure Monitor, dashboards |

**Result:** 45% reduction in support tickets, 92% customer satisfaction, $2.1M annual savings.

### Example 2: Medical Document Analyzer

**Business Problem:** A healthcare organization needs to extract key information from clinical trial documents.

| Stage | What They Did | AI Foundry Tool Used |
|-------|--------------|---------------------|
| **1. Data Collection** | Gathered 10,000 clinical trial PDFs, regulatory documents, and medical ontologies | Azure Blob Storage |
| **2. Data Preparation** | OCR'd scanned documents, extracted text, tagged with medical entity labels, created specialized index | Azure AI Search, AI Document Intelligence |
| **3. Model Selection** | Chose GPT-4o as base, fine-tuned on 2,000 medical Q&A pairs for domain accuracy | Model Catalog, Fine-tuning |
| **4. Evaluation** | Evaluated against board-certified physicians' answers, achieved 94% agreement rate | Custom evaluation, expert review |
| **5. Deployment** | Deployed on Managed Compute (data residency requirements), HIPAA-compliant configuration | Managed endpoint |
| **6. Monitoring** | Tracked accuracy weekly, flagged any outputs that contradicted established medical literature | Azure Monitor, custom alerts |

**Result:** 80% reduction in document review time, maintained 94%+ accuracy, full compliance with healthcare regulations.

### Example 3: E-Commerce Product Recommendations

**Business Problem:** An online retailer wants to personalize product recommendations using AI.

| Stage | What They Did | AI Foundry Tool Used |
|-------|--------------|---------------------|
| **1. Data Collection** | Collected browsing history, purchase data, product descriptions, customer reviews | Azure SQL, Blob Storage |
| **2. Data Preparation** | Normalized product categories, created user behavior profiles, generated embedding vectors | Data assets, Azure AI Search |
| **3. Model Selection** | Used embedding model for product similarity + GPT-4o for natural language recommendations | Model Catalog |
| **4. Evaluation** | A/B tested against existing recommendation engine, measured click-through rate and conversion | Evaluation tools, custom metrics |
| **5. Deployment** | Serverless API with auto-scaling for traffic spikes (Black Friday, etc.) | Serverless endpoint |
| **6. Monitoring** | Real-time tracking of recommendation quality, conversion rates, API latency | Azure Monitor |

**Result:** 23% increase in click-through rate, 15% increase in average order value, handled 10x traffic spikes.

---

## 12. Setting Up Your Azure Account

### Why You Need an Azure Account

Starting this week, we'll be doing hands-on work in Azure AI Foundry. To follow along, you need:

1. **A Microsoft account** (personal or work/school)
2. **An Azure subscription** (free tier available)
3. **An AI Foundry project** (created within the subscription)

### Azure Free Account Details

Microsoft offers a generous free tier for new accounts:

| What You Get | Details |
|-------------|---------|
| **$200 credit** | Use on any Azure service for 30 days |
| **12 months of free services** | Selected services free for a year |
| **Always-free services** | 25+ services are permanently free |
| **No charge until you upgrade** | Free tier won't charge your card |

> 💡 **Tip:** The $200 credit is more than enough to follow along with this entire training course's hands-on labs. Just remember to clean up resources you're not using!

### Step-by-Step Account Setup

Detailed instructions are in the **Hands-On Lab** for this week, but here's the overview:

1. Go to [azure.microsoft.com/free](https://azure.microsoft.com/free)
2. Click "Start free" or "Try Azure for free"
3. Sign in with your Microsoft account
4. Complete identity verification (phone number, credit card)
5. Accept terms and create the account
6. Navigate to [ai.azure.com](https://ai.azure.com) to access AI Foundry

> ⚠️ **Important:** You will need a credit card to create an Azure account, but you will **not** be charged unless you explicitly upgrade to a paid subscription. The free tier is genuinely free.

---

## 13. Connecting the Dots — From Week 1 to Week 3

### What We've Built So Far

```
Week 1: WHAT is AI Foundry?        Week 2: HOW does AI work?
┌──────────────────────────┐       ┌──────────────────────────┐
│ • Platform overview      │       │ • The AI lifecycle       │
│ • Key capabilities       │  ──▸  │ • Data → Deploy → Monitor│
│ • Why it exists           │       │ • Model selection        │
│ • First look at portal   │       │ • Evaluation metrics     │
└──────────────────────────┘       │ • MLOps concepts         │
                                   │ • Azure account setup    │
                                   └──────────────────────────┘
                                              │
                                              ▼
                                   ┌──────────────────────────┐
                                   │ Week 3: BUILDING your    │
                                   │ first AI application     │
                                   │ (Coming next!)           │
                                   └──────────────────────────┘
```

### Key Takeaways from Week 2

1. **The AI lifecycle is a loop, not a line** — Data → Prepare → Model → Evaluate → Deploy → Monitor → (repeat)
2. **Start simple** — Pre-built models + prompt engineering + RAG before fine-tuning
3. **Evaluation is not optional** — Always test before deploying
4. **Monitoring is ongoing** — Your AI's job isn't done when you deploy it
5. **MLOps brings order to chaos** — Version, automate, and monitor everything
6. **Azure AI Foundry covers the entire lifecycle** — One platform for all six stages

### Preview: Week 3

In Week 3, we'll move from theory to practice with **"Navigating the AI Foundry Portal."** You'll:

- Take a guided tour of every section of the AI Foundry portal
- Create your first project and configure settings
- Explore the Model Catalog in depth
- Test models in the Playground
- Understand projects, hubs, and resource organization

---

## 📖 Glossary

| Term | Definition |
|------|-----------|
| **AI Lifecycle** | The end-to-end process of building, deploying, and maintaining an AI application |
| **Chunking** | Splitting large documents into smaller pieces for AI processing |
| **Content Safety** | AI-powered filtering to prevent harmful outputs (hate speech, violence, etc.) |
| **Data Preparation** | Cleaning, formatting, and organizing data for AI use |
| **Deployment** | Making an AI model available for production use via APIs or endpoints |
| **Embeddings** | Mathematical representations of text used for similarity search |
| **Endpoint** | A URL where your deployed AI model can be accessed via API calls |
| **Evaluation** | Testing AI outputs against quality and safety metrics |
| **Fine-tuning** | Training a pre-built model further on your specific data |
| **Groundedness** | Whether AI answers are based on provided facts vs made-up information |
| **JSONL** | JSON Lines format — one JSON object per line, used for training data |
| **MaaS** | Model as a Service — serverless, pay-per-token deployment |
| **MaaP** | Model as a Platform — dedicated compute, full control deployment |
| **MLOps** | Machine Learning Operations — applying DevOps practices to AI/ML |
| **Model Catalog** | AI Foundry's library of 1,800+ deployable AI models |
| **Model Drift** | When a model's performance degrades over time due to changing data patterns |
| **Monitoring** | Continuously tracking AI performance, safety, and costs in production |
| **Prompt Engineering** | Crafting effective inputs to get better outputs from AI models |
| **RAG** | Retrieval-Augmented Generation — connecting a model to external knowledge |
| **Tokenization** | Breaking text into smaller units (tokens) for AI processing |
| **Training** | Teaching a model from scratch using large datasets |

---

## 📚 Additional Resources

- [Azure AI Foundry documentation](https://learn.microsoft.com/azure/ai-studio/)
- [Azure AI Foundry Model Catalog](https://ai.azure.com/explore/models)
- [Responsible AI principles](https://www.microsoft.com/ai/responsible-ai)
- [MLOps with Azure Machine Learning](https://learn.microsoft.com/azure/machine-learning/concept-model-management-and-deployment)
- [Azure free account](https://azure.microsoft.com/free)
- [AI lifecycle best practices (Microsoft Learn)](https://learn.microsoft.com/azure/architecture/ai-ml/)

---

> **Next Week:** [Week 3 — Navigating the AI Foundry Portal →](../week-03/training-guide.md)
>
> **Previous Week:** [← Week 1 — What is AI Foundry?](../week-01/training-guide.md)

---

*© 2025 Zero to Hero: Azure AI Foundry Training. Week 2 of 45.*
