# Module 1: What is AI Foundry?

## **ARC 1: Foundations** | Module 1 of 45 — Zero to Hero: Azure AI Foundry

---

| Detail              | Info                                                    |
|---------------------|---------------------------------------------------------|
| **Arc**             | ARC 1 — Foundations (Modules 1–5)                         |
| **Module**            | 1                                                       |
| **Title**           | What is AI Foundry?                                     |
| **Estimated Time**  | 2–3 hours                                               |
| **Difficulty**      | ⭐ Beginner                                              |
| **Last Updated**    | April 2026                                               |

---

## 🎯 Learning Objectives

By the end of this module, you will be able to:

1. **Explain** what Azure AI Foundry is and why it exists.
2. **Identify** the key capabilities of the platform (Model Catalog, Agent Framework, Playground, Evaluation, Deployment).
3. **Distinguish** AI Foundry from other Azure AI services (Azure OpenAI, Azure Machine Learning, Cognitive Services).
4. **Describe** at least three real-world use cases powered by AI Foundry.
5. **Navigate** the AI Foundry portal and recognize its major sections.

---

## 📋 Prerequisites

- **No prior AI or machine learning experience required** — this is truly day one.
- A basic understanding of what "the cloud" is (you know that Azure is Microsoft's cloud platform).
- A Microsoft account (free). If you don't have one, create one at [account.microsoft.com](https://account.microsoft.com).
- Curiosity and willingness to learn!

> 💡 **Tip:** You do *not* need an Azure subscription to follow along with this module's material. We'll set one up together in Module 2.

---

## 1. Introduction — Why AI Matters Now

### The AI Moment

We are living through one of the most significant technology shifts in history. In just the past two years:

- **ChatGPT** reached 100 million users faster than any product ever.
- Businesses are deploying AI to automate tasks, generate content, and make smarter decisions.
- AI is no longer locked in research labs — it's available to *every* developer through cloud platforms.

But here's the challenge: **building AI applications is still hard.** You need to pick the right model, prepare your data, write prompts, evaluate outputs, deploy to production, and monitor for safety — all while meeting enterprise requirements like security, compliance, and cost control.

That's a lot of moving parts. What if there was *one place* that brought it all together?

### The Problem AI Foundry Solves

Imagine you want to build a customer support chatbot for your company. Before AI Foundry, you'd need to:

1. Research which AI model to use (GPT-4? Llama? Something else?)
2. Figure out how to access it (API keys? SDKs? Containers?)
3. Write and test your prompts manually
4. Build custom evaluation pipelines
5. Set up deployment infrastructure
6. Add safety guardrails yourself
7. Monitor everything in production

That's **seven different problems** before you even ship version 1.

> 🏭 **Azure AI Foundry is Microsoft's answer: a single, unified platform that handles all of this in one place.**

Think of it like this:

| Without AI Foundry | With AI Foundry |
|---------------------|-----------------|
| Visit 5+ different portals | One portal: [ai.azure.com](https://ai.azure.com) |
| Manually compare models | Browse 1,800+ models in a catalog |
| Build your own eval pipeline | Built-in evaluation tools |
| DIY safety guardrails | Responsible AI built-in |
| Stitch together services | Integrated end-to-end workflow |

---

## 2. What is Azure AI Foundry?

### The Simple Definition

**Azure AI Foundry** is Microsoft's unified platform for building, evaluating, and deploying AI applications and agents at scale.

Think of it as an **AI app and agent factory** — a single workbench where you go from *idea* to *production-ready AI application*.

### A Brief History

You might see references to "Azure AI Studio" in older documentation or blog posts. Here's what happened:

| Timeline | Name | What Changed |
|----------|------|--------------|
| 2023 | Azure AI Studio (Preview) | Launched as a new AI development experience |
| November 2024 | **Azure AI Foundry** | Rebranded and expanded with agent capabilities, deeper integrations, and GA (General Availability) features |

> 📝 **Note:** If you see "Azure AI Studio" anywhere, it's the *same product* — just the older name. Microsoft renamed it to **AI Foundry** to better reflect its role as a comprehensive AI development platform.

### The "AI App and Agent Factory" Concept

Microsoft describes AI Foundry as a **factory** — and that analogy is worth unpacking:

```
┌─────────────────────────────────────────────────────────┐
│                   AI FOUNDRY = FACTORY                  │
│                                                         │
│   Raw Materials     →  Assembly Line  →  Finished       │
│   (Models, Data)       (Build, Test,     Product        │
│                         Evaluate)        (AI App or     │
│                                           Agent)        │
│                                                         │
│   ┌───────────┐     ┌─────────────┐   ┌─────────────┐  │
│   │ Model     │     │ Playground  │   │ Deployed    │  │
│   │ Catalog   │ ──▸ │ Prompt Flow │ ─▸│ Endpoint    │  │
│   │ Your Data │     │ Evaluation  │   │ Monitoring  │  │
│   └───────────┘     └─────────────┘   └─────────────┘  │
│                                                         │
│          🔒 Responsible AI & Security Throughout        │
└─────────────────────────────────────────────────────────┘
```

Just like a real factory:
- **Raw materials** = AI models and your data
- **Assembly line** = tools to build, test, and refine your AI application
- **Quality control** = evaluation and responsible AI checks
- **Finished product** = a deployed, monitored AI application or agent

---

## 3. Key Capabilities

Let's walk through the six major capabilities of Azure AI Foundry. Don't worry about mastering these now — we'll dive deep into each one in later modules.

### 3.1 🗂️ Model Catalog

**What it is:** A library of **1,800+ AI models** you can browse, compare, and deploy — all from one place.

**Available models include:**

| Provider | Example Models | Strengths |
|----------|---------------|-----------|
| **OpenAI** | GPT-4o, GPT-4.1, o3 | Best-in-class language understanding and generation |
| **Meta** | Llama 4 Scout, Llama 4 Maverick | Open-weight, customizable, strong multilingual |
| **Microsoft** | Phi-4, MAI | Small but powerful, efficient for edge/mobile |
| **Mistral** | Mistral Large, Mistral Small | Fast, efficient, great price-performance |
| **And more** | Cohere, AI21, JAIS, etc. | Specialized models for different tasks |

**Analogy:** Think of the Model Catalog like an **app store for AI brains**. Instead of downloading apps, you're browsing AI models — each with different strengths, sizes, and price points. You pick the one that fits your need, and AI Foundry handles the rest.

> 💡 **Tip:** You don't need to build your own AI model from scratch. The Model Catalog lets you start with a powerful pre-built model and customize it for your specific use case.

### 3.2 🤖 Agent Framework

**What it is:** A built-in system for creating **intelligent AI agents** — AI systems that can reason, plan, use tools, and take actions on behalf of users.

An AI agent isn't just a chatbot that responds to questions. It can:

- **Reason** through multi-step problems
- **Use tools** (search the web, query databases, call APIs)
- **Remember context** across a conversation
- **Take actions** (create tickets, send emails, update records)

**Example:** Imagine an IT support agent that can:
1. Understand an employee's problem ("My VPN won't connect")
2. Look up the employee's device configuration
3. Check if there's a known outage
4. Walk the employee through troubleshooting steps
5. Escalate to a human if unresolved

That's the kind of thing you can build with the Agent Framework.

### 3.3 ✍️ Prompt Engineering & Playground

**What it is:** An interactive environment where you can **write, test, and refine prompts** — the instructions you give to AI models.

The Playground lets you:
- Send messages to any deployed model and see responses in real-time
- Adjust parameters (temperature, max tokens, etc.)
- Add system messages to control the model's behavior
- Compare outputs from different models side-by-side
- Test with your own data

**Analogy:** The Playground is like a **test kitchen** for AI. Before you serve a dish to customers (deploy to production), you experiment with recipes (prompts) until you get the flavor (output) just right.

> 💡 **Tip:** Prompt engineering is one of the most valuable skills in AI today. We'll dedicate an entire module to it later in the course.

### 3.4 📊 Evaluation & Testing Tools

**What it is:** Built-in tools to **measure how well your AI application performs** before you deploy it.

You can evaluate:
- **Accuracy** — Is the model giving correct answers?
- **Groundedness** — Are the responses based on provided data (not hallucinated)?
- **Relevance** — Are responses on-topic and useful?
- **Coherence** — Do responses make logical sense?
- **Safety** — Are responses free from harmful content?

**Why it matters:** You wouldn't ship software without testing it. The same applies to AI. AI Foundry's evaluation tools give you confidence that your AI behaves correctly *before* real users see it.

### 3.5 🚀 Deployment & Monitoring

**What it is:** Tools to **deploy your AI application to production** and **monitor its performance** over time.

Deployment options include:
- **Managed endpoints** — Microsoft handles the infrastructure
- **Serverless APIs** — Pay-per-call, no infrastructure to manage
- **Container-based** — For custom hosting requirements

Once deployed, you can monitor:
- Latency (how fast responses are)
- Token usage and cost
- Error rates
- User feedback

### 3.6 🛡️ Responsible AI Built-In

**What it is:** Safety features and guardrails integrated throughout the platform to help you build AI that is **fair, reliable, safe, private, inclusive, transparent, and accountable**.

Key features include:
- **Content filters** — Block harmful or inappropriate outputs
- **Groundedness detection** — Flag responses that aren't supported by your data
- **Jailbreak protection** — Prevent users from manipulating the AI into unsafe behavior
- **Fairness analysis** — Check for bias in your model's outputs

> ⚠️ **Important:** Responsible AI isn't optional — it's a core part of building with AI Foundry. Microsoft provides these tools because deploying AI without safety measures can cause real harm. We'll cover this extensively in ARC 6 (Modules 26–30).

---

## 4. How AI Foundry Fits in the Azure Ecosystem

If you've explored Azure before, you may have heard of other AI-related services. Here's how they all fit together:

### The Big Picture

```
┌─────────────────────────────────────────────────────────────┐
│                      AZURE AI FOUNDRY                       │
│          (Unified platform — your "front door")             │
│                                                             │
│   ┌──────────────┐  ┌──────────────┐  ┌────────────────┐   │
│   │  Azure       │  │  Azure       │  │  Azure AI      │   │
│   │  OpenAI      │  │  Machine     │  │  Services      │   │
│   │  Service     │  │  Learning    │  │  (Vision,      │   │
│   │              │  │              │  │   Speech, etc.) │   │
│   │  GPT-4, o3,  │  │  Custom ML   │  │  Pre-built     │   │
│   │  DALL-E,     │  │  Training,   │  │  AI APIs       │   │
│   │  Whisper     │  │  MLOps       │  │                │   │
│   └──────────────┘  └──────────────┘  └────────────────┘   │
│                                                             │
│          All accessible through AI Foundry                  │
└─────────────────────────────────────────────────────────────┘
```

### Comparison Table

| Feature | Azure AI Foundry | Azure OpenAI Service | Azure Machine Learning | Azure AI Services |
|---------|-----------------|---------------------|----------------------|-------------------|
| **What is it?** | Unified AI development platform | Access to OpenAI models | Custom ML model training & deployment | Pre-built AI APIs (Vision, Speech, Language) |
| **Best for** | End-to-end AI app development | Using GPT, DALL-E, Whisper directly | Training your own ML models | Adding specific AI capabilities quickly |
| **Model selection** | 1,800+ models from many providers | OpenAI models only | Bring your own or use curated | Pre-built, task-specific |
| **Agents** | ✅ Built-in Agent Framework | ❌ Not directly | ❌ Not directly | ❌ Not directly |
| **Code required?** | Optional (portal + code) | Yes (API-based) | Yes (heavy code/notebooks) | Minimal (REST APIs) |
| **Skill level** | Beginner to Advanced | Intermediate | Advanced | Beginner to Intermediate |

### When to Use What

- **"I want to build an AI-powered application from scratch"** → **AI Foundry**
- **"I just need to call GPT-4 from my code"** → Azure OpenAI Service (also accessible through AI Foundry)
- **"I need to train a custom machine learning model on my data"** → Azure Machine Learning
- **"I just need to add speech-to-text or image analysis to my app"** → Azure AI Services

> 💡 **Tip:** AI Foundry acts as a **unifying layer** on top of many Azure AI services. You can access Azure OpenAI models, Azure AI Services, and more — all from within the AI Foundry portal. Think of AI Foundry as the "front door" to Azure's entire AI ecosystem.

---

## 5. Real-World Use Cases

AI Foundry isn't theoretical — companies are using it right now to solve real business problems. Here are some examples:

### 5.1 🗣️ Customer Support Chatbot

**The problem:** A telecom company receives 50,000 support tickets per month. Wait times are long, and customers are frustrated.

**The AI Foundry solution:**
- Deploy a GPT-4o model from the Model Catalog
- Connect it to the company's knowledge base using RAG (Retrieval-Augmented Generation)
- Build an agent that can look up account information and troubleshoot common issues
- Use evaluation tools to ensure accuracy before going live

**The result:** 60% of tickets resolved without human intervention. Average response time drops from 4 hours to 30 seconds.

### 5.2 📄 Document Q&A Over Enterprise Data

**The problem:** A law firm has 10 years of legal documents. Lawyers spend hours searching for relevant precedents and clauses.

**The AI Foundry solution:**
- Index the document library using Azure AI Search (integrated with AI Foundry)
- Deploy a model that can answer questions grounded in the actual documents
- Build a simple chat interface for lawyers to ask questions in natural language

**The result:** Research that used to take 3 hours now takes 5 minutes.

> 🏢 **Real Example — KPMG:** KPMG uses Azure AI Foundry to power AI solutions for audit, tax, and advisory services. They leverage the platform to build AI agents that help professionals analyze complex documents and provide insights faster than manual review.

### 5.3 💻 Code Generation Assistant

**The problem:** A software development team wants to accelerate coding and reduce bugs.

**The AI Foundry solution:**
- Deploy a code-capable model (like GPT-4o or a Phi model)
- Fine-tune it on the company's internal coding standards and libraries
- Integrate it into the development workflow via API

**The result:** Developers report 30–40% faster coding for routine tasks.

### 5.4 📈 Predictive Analytics

**The problem:** An e-commerce company wants to predict which products will trend next quarter.

**The AI Foundry solution:**
- Use Azure Machine Learning (accessible through AI Foundry) to train a forecasting model
- Combine it with a language model that can explain predictions in plain English
- Deploy as an internal dashboard with a chat interface

**The result:** Inventory planning accuracy improves by 25%.

> 🏢 **Real Example — Carvana:** Carvana, the online used car retailer, leverages Azure AI services to enhance their customer experience — from vehicle image processing to personalized recommendations, using AI to streamline the car-buying journey.

> 🏢 **Real Example — Fujitsu:** Fujitsu partners with Microsoft to deliver AI-powered transformation for enterprises. They use Azure AI Foundry to build industry-specific AI solutions, helping clients in manufacturing, healthcare, and finance automate complex workflows and derive insights from vast datasets.

---

## 6. The AI Foundry Portal Tour

Let's take a virtual tour of the AI Foundry portal. You can follow along at [ai.azure.com](https://ai.azure.com).

> 📝 **Note:** You can browse the portal and explore the Model Catalog without an Azure subscription. To create projects and deploy models, you'll need one — we'll set that up in Module 2.

### 6.1 🏠 Home Page

**What you'd see:** A clean landing page with:
- A welcome banner introducing AI Foundry
- Quick-start options: "Create a project," "Explore models," "Try the playground"
- Recent projects (once you have some)
- Learning resources and what's new

**Think of it as:** The lobby of the AI factory — a directory to help you find where to go.

### 6.2 📚 Model Catalog

**What you'd see:** A searchable, filterable gallery of 1,800+ models:
- Filter by **provider** (OpenAI, Meta, Microsoft, Mistral, etc.)
- Filter by **task** (text generation, image generation, embeddings, etc.)
- Filter by **license** (proprietary, open-source, etc.)
- Each model card shows: name, provider, description, capabilities, and a "Deploy" button

**Think of it as:** The warehouse floor — rows and rows of "AI brains" ready to be selected for your project.

### 6.3 🧪 Playground

**What you'd see:** A split-screen interface:
- **Left panel:** System message (instructions for the AI), parameter settings (temperature, max tokens)
- **Right panel:** A chat interface where you type messages and see the model's responses
- Options to switch between different deployed models
- A "View Code" button that generates API code for your conversation

**Think of it as:** The R&D lab — where you experiment before committing to production.

### 6.4 📁 Projects

**What you'd see:** A list of your AI Foundry projects, each containing:
- Deployed models and endpoints
- Data connections
- Evaluation runs
- Prompt flow configurations
- Team members and access settings

**Think of it as:** Individual production lines in the factory — each project is a separate product you're building.

### 6.5 🚀 Deployments

**What you'd see:** A dashboard showing all your deployed models:
- Model name and version
- Deployment type (managed, serverless, etc.)
- Endpoint URL
- Status (active, updating, failed)
- Usage metrics (requests, tokens, latency)

**Think of it as:** The shipping dock — where your finished AI products go out into the world.

### 6.6 🤖 Agents (Preview)

**What you'd see:** An interface for building and managing AI agents:
- Agent configuration (model, instructions, tools)
- Tool integration (code interpreter, file search, custom functions)
- Testing interface to interact with your agent
- Deployment options

**Think of it as:** The robotics assembly area — where you build AI workers that can think and act independently.

---

## 7. Key Terminology Glossary

Here are the essential terms you'll encounter throughout this course. Bookmark this section — you'll refer back to it often!

| Term | Definition | Analogy |
|------|-----------|---------|
| **Model** | A trained AI system that can perform specific tasks (generate text, classify images, etc.) | A skilled worker trained to do a specific job |
| **Large Language Model (LLM)** | A type of AI model trained on vast amounts of text data, capable of understanding and generating human language | A worker who has read every book in the library and can write on any topic |
| **Foundation Model** | A large, general-purpose model that can be adapted (fine-tuned) for many different tasks | A Swiss Army knife — versatile and useful for many situations |
| **Fine-Tuning** | The process of further training a pre-built model on your specific data to improve its performance for your use case | Teaching a general chef to specialize in Italian cuisine |
| **Prompt** | The text input (instruction or question) you send to an AI model | The question you ask a knowledgeable colleague |
| **System Prompt** | Special instructions that define how the AI model should behave across all interactions | The job description you give an employee before they start work |
| **Token** | The smallest unit of text that a model processes. Roughly 1 token ≈ ¾ of a word in English | Individual LEGO bricks that make up a sentence |
| **Embedding** | A numerical representation of text (or images) that captures meaning, used for search and comparison | A GPS coordinate for a concept — similar concepts have nearby coordinates |
| **RAG (Retrieval-Augmented Generation)** | A technique where the AI retrieves relevant information from your data before generating a response — reduces hallucination | Instead of answering from memory, the AI first looks up the answer in your reference books |
| **Agent** | An AI system that can reason, plan, use tools, and take autonomous actions to complete tasks | A virtual employee who can think through problems and use software tools on their own |
| **Deployment** | The process of making your AI model available for use (via an API endpoint or application) | Opening your store for business — going from "built" to "live" |
| **Endpoint** | A URL where your deployed model listens for requests | The front counter where customers place orders |
| **Inference** | The process of a model generating a response to an input (as opposed to *training*, which is how it learned) | A chef cooking a meal (inference) vs. a chef in cooking school (training) |
| **Grounding** | Connecting an AI model's responses to specific, verifiable data sources so it answers based on facts, not guesses | Citing your sources in a research paper |
| **Hallucination** | When an AI model generates information that sounds plausible but is factually incorrect | A student who confidently writes wrong answers on a test |
| **Responsible AI** | Principles and practices to ensure AI systems are fair, reliable, safe, private, inclusive, transparent, and accountable | Safety regulations in a factory — protecting workers and customers |
| **MLOps** | Practices for managing the full lifecycle of machine learning models in production (deploy, monitor, update, retrain) | DevOps, but for AI models instead of traditional software |
| **Temperature** | A parameter that controls how creative (or random) a model's responses are. Low = focused and deterministic. High = creative and varied. | The "wildness dial" on the AI — turn it down for facts, up for creative writing |
| **Context Window** | The maximum amount of text a model can process in a single interaction (measured in tokens) | The size of the model's working memory — how much it can "see" at once |

---

## 8. Hands-On Exercise (Optional)

While we won't be building anything yet, here's a simple exercise to familiarize yourself with the platform:

### Exercise: Explore the Model Catalog

1. **Open** [ai.azure.com](https://ai.azure.com) in your browser
2. **Navigate** to the **Model Catalog**
3. **Browse** the available models — try filtering by:
   - Provider: "OpenAI" → note the available GPT models
   - Provider: "Meta" → find the Llama models
   - Provider: "Microsoft" → find the Phi models
   - Task: "Chat completion" → see which models support conversations
4. **Click** on any model card to read its details
5. **Answer** these questions in your notes:
   - How many total models are available?
   - Which providers have the most models?
   - What's the difference between "Managed Compute" and "Serverless API" deployment options?

> 💡 **Tip:** Don't worry if you don't understand everything on the model cards yet. The goal is just to get comfortable navigating the portal. We'll explain all the technical details in coming modules.

---

## 9. Summary & Key Takeaways

Let's recap what we covered this module:

1. **Azure AI Foundry is Microsoft's unified platform** for building, evaluating, and deploying AI applications and agents — think of it as an "AI app and agent factory."

2. **You don't need to build AI from scratch.** The Model Catalog gives you access to 1,800+ pre-built models from OpenAI, Meta, Microsoft, and more — ready to use or customize.

3. **AI Foundry covers the entire AI development lifecycle** — from selecting a model, to testing prompts, to evaluating quality, to deploying and monitoring in production.

4. **Responsible AI is built in, not bolted on.** Content filters, groundedness detection, and fairness tools are integrated throughout the platform, helping you build AI that is safe and trustworthy.

5. **AI Foundry is the "front door" to Azure's AI ecosystem.** It brings together Azure OpenAI, Azure Machine Learning, and Azure AI Services into a single, cohesive experience.

---

## 10. Preview of Module 2

### 📅 Module 2: Understanding the AI Lifecycle

next module, we'll explore the **end-to-end journey** of building an AI application:

- The 6 stages of the AI development lifecycle
- How each stage maps to AI Foundry's tools
- Setting up your Azure account and creating your first AI Foundry project
- The difference between training, fine-tuning, and prompt engineering
- Your first hands-on lab: deploying a model and sending your first prompt!

> 🚀 **Get ready!** Module 2 is where we go from *understanding* to *doing*. Make sure you have a Microsoft account ready.

---

## 11. Additional Resources

### 📖 Microsoft Learn

| Resource | Link | Description |
|----------|------|-------------|
| What is Azure AI Foundry? | [learn.microsoft.com/azure/ai-foundry/what-is-azure-ai-foundry](https://learn.microsoft.com/azure/ai-foundry/what-is-azure-ai-foundry) | Official overview documentation |
| AI Foundry Quickstart | [learn.microsoft.com/azure/ai-foundry/quickstarts/get-started-playground](https://learn.microsoft.com/azure/ai-foundry/quickstarts/get-started-playground) | Get started with the Playground |
| Model Catalog Overview | [learn.microsoft.com/azure/ai-foundry/how-to/model-catalog-overview](https://learn.microsoft.com/azure/ai-foundry/how-to/model-catalog-overview) | Browse and deploy models |
| Responsible AI Overview | [learn.microsoft.com/azure/ai-foundry/concepts/responsible-ai](https://learn.microsoft.com/azure/ai-foundry/concepts/responsible-ai) | Responsible AI principles |

### 📺 Videos & Tutorials

- [Azure AI Foundry Overview — Microsoft Ignite](https://ignite.microsoft.com) — Search for "AI Foundry" sessions
- [Microsoft AI YouTube Channel](https://www.youtube.com/@MicrosoftAI) — Regular tutorials and announcements

### 🌐 Community

- [Microsoft Tech Community — AI](https://techcommunity.microsoft.com/category/ai) — Forums, articles, and discussions
- [Azure AI Foundry GitHub Samples](https://github.com/azure-samples) — Search for "ai-foundry" for sample code and templates
- [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-ai) — Tag: `azure-ai` for Q&A

### 📚 Recommended Reading

- [Microsoft Responsible AI Principles](https://www.microsoft.com/ai/responsible-ai) — Understand Microsoft's AI ethics framework
- [State of AI Report](https://www.stateof.ai/) — Annual report on global AI trends (great context for why this all matters)

---

## 📋 Course Roadmap — Where You Are

```
✅ Module 1  — What is AI Foundry? (YOU ARE HERE)
   Module 2  — Understanding the AI Lifecycle
   Module 3  — Azure Fundamentals for AI
   Module 4  — AI Foundry Architecture Deep Dive
   Module 5  — Your First AI Project (End-to-End)
   ─────────────────────────────────────────────
   ARC 2: Data & Prep (Modules 6–10)
   ARC 3: Model Building (Modules 11–15)
   ARC 4: Deployment & MLOps (Modules 16–20)
   ARC 5: Agents & Copilot Integration (Modules 21–25)
   ARC 6: Security, Governance & Compliance (Modules 26–30)
   ARC 7: Advanced AI (Modules 31–35)
   ARC 8: Partner & Business Scenarios (Modules 36–40)
   ARC 9: Capstone & Mastery (Modules 41–45)
```

---

> 🎉 **Congratulations!** You've completed Module 1 of your Zero to Hero journey. You now know what Azure AI Foundry is, why it matters, and what it can do. next module, we roll up our sleeves and start building. See you there!

---

*© 2026 Zero to Hero: Azure AI Foundry Training Program. All rights reserved.*
