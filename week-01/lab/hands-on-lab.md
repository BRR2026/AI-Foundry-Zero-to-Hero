# Lab 1: Exploring Azure AI Foundry

| | |
|---|---|
| **Estimated Time** | 45–60 minutes |
| **Difficulty** | ⭐ Beginner |
| **Week** | 1 of 45 |

## Prerequisites

| Requirement | Details |
|---|---|
| Azure subscription | Free tier is perfectly fine — no paid resources required |
| Web browser | Microsoft Edge or Google Chrome (latest version recommended) |
| Azure account | Sign up free at [azure.microsoft.com/free](https://azure.microsoft.com/free) if you don't have one |

> **💡 Note:** This is your very first lab. There is no coding required — today is all about exploring and getting comfortable with the platform.

---

## Learning Objectives

By the end of this lab you will be able to:

- ✅ Navigate the Azure AI Foundry portal confidently
- ✅ Explore the model catalog and understand what's available
- ✅ Create your first AI Foundry project from scratch
- ✅ Send your first prompt in the Playground and experiment with settings

---

## Exercise 1: Access the Azure AI Foundry Portal (10 min)

### Step 1 — Open the Portal

1. Open your web browser.
2. Navigate to **[https://ai.azure.com](https://ai.azure.com)**.
3. Click **Sign in** and enter your Azure credentials (the same email and password you use for the Azure portal).

> **🔑 First-time users:** If prompted, accept the terms of service and grant any requested permissions. This is normal for a first visit.

### Step 2 — Tour the Home Page

Once you're signed in, take a moment to look around. The home page is your launchpad for everything in AI Foundry. Here's what you'll see:

| Section | What It Does |
|---|---|
| **Home** | Your personalized dashboard showing recent projects and quick-start actions |
| **Model catalog** | A searchable library of 1,800+ AI models from Microsoft, OpenAI, Meta, Mistral, and more |
| **My projects** | A list of all the AI projects you've created — think of these as workspaces |
| **Management** | Where you handle resources, connections, deployments, and billing |
| **Learn** | Tutorials, documentation, and sample code to help you get started |

### Step 3 — Explore the Navigation

- Click on a few sections in the left sidebar to see how they're organized.
- Notice the **search bar** at the top — you can search for models, projects, and documentation.
- Look for the **"What's new"** section on the home page to see recent platform updates.

### ✅ Checkpoint

> Take a screenshot of the AI Foundry home page with your account signed in.
>
> **What to verify:** You can see the navigation sidebar, the home dashboard, and your account name in the top-right corner.

---

## Exercise 2: Explore the Model Catalog (10 min)

The model catalog is one of the most powerful features of AI Foundry — it gives you access to hundreds of pre-built AI models without writing a single line of code.

### Step 1 — Open the Model Catalog

1. From the home page, click **"Model catalog"** in the left sidebar (or click the Model catalog card on the home page).
2. You should see a searchable grid of AI models.

### Step 2 — Browse by Provider

The catalog includes models from many providers. Look for these:

| Provider | Example Models | Known For |
|---|---|---|
| **OpenAI** | GPT-4o, GPT-4o-mini, DALL·E | Industry-leading language and image generation |
| **Meta** | Llama 3, Llama 3.1 | High-performance open-source models |
| **Microsoft** | Phi-3, Phi-3.5 | Small but powerful models optimized for efficiency |
| **Mistral** | Mistral Large, Mistral Nemo | Strong European open-weight models |

- Use the **provider filter** on the left side to show only models from a specific company.
- Try selecting **"OpenAI"** to see their full model lineup.

### Step 3 — Filter by Task Type

Models are designed for different jobs. Use the task-type filter to explore:

| Task Type | What It Does | Example Use Case |
|---|---|---|
| **Chat completion** | Have a conversation with the model | Customer support chatbot |
| **Text completion** | Generate or continue text | Writing blog posts or emails |
| **Embeddings** | Convert text into numerical vectors | Search engines, recommendation systems |
| **Image generation** | Create images from text descriptions | Marketing visuals, concept art |

- Try filtering for **"Chat completion"** models — notice how the list narrows down.

### Step 4 — Read a Model Card

1. Search for **"GPT-4o-mini"** using the search bar.
2. Click on the model card to open its details page.
3. Read through the following sections:
   - **Description** — What the model does and what it's good at
   - **Capabilities** — Specific tasks this model supports
   - **Limitations** — What the model cannot do or struggles with
   - **Pricing** — How much it costs to use (pay-per-token)

> **💡 Why this matters:** Understanding model cards helps you choose the right model for your project. A model that's great for chat may not be ideal for embeddings.

### Step 5 — Compare Two Models

1. Go back to the model catalog.
2. Find **GPT-4o** and **GPT-4o-mini**.
3. Note the differences:
   - **GPT-4o** — More capable but costs more per token
   - **GPT-4o-mini** — Lighter, faster, and cheaper — great for learning and prototyping

### ✅ Checkpoint

> In your notes, write down **3 models** you found interesting and one capability for each:
>
> | Model | Capability I Found Interesting |
> |---|---|
> | 1. | |
> | 2. | |
> | 3. | |

---

## Exercise 3: Create Your First Project (15 min)

An AI Foundry **project** is your workspace. It's where you deploy models, run experiments, build AI applications, and manage everything in one place.

### Step 1 — Start the Creation Wizard

1. Click **"Create project"** from the home page or from the **"My projects"** section.

### Step 2 — Configure Your Project

Fill in the following fields:

| Field | What to Enter | Tips |
|---|---|---|
| **Project name** | `my-first-ai-project` | Use lowercase letters and hyphens. Pick something memorable. |
| **Hub** | Create new or select existing | A hub is a shared container for resources. Create a new one if this is your first time. |
| **Subscription** | Your Azure subscription | Select your free tier subscription if you have one. |
| **Resource group** | Create new → `rg-ai-foundry-training` | Resource groups are folders for organizing Azure resources. |
| **Region** | Choose a region close to you | **East US**, **West Europe**, or **Australia East** are common choices. Not all models are available in every region. |

### Step 3 — Review and Create

1. Click **"Review + Create"**.
2. Double-check your settings on the review page.
3. Click **"Create"**.

### Step 4 — Wait for Provisioning

- Azure is now setting up your project behind the scenes. This creates several resources:
  - An **AI Hub** (shared infrastructure)
  - A **project workspace** (your personal workspace)
  - A **storage account** (for your files and data)
  - A **Key Vault** (for securely storing secrets)

- This process typically takes **1–3 minutes**. You'll see a progress indicator.

> **⚠️ If provisioning fails:** The most common reasons are:
> - The region doesn't support AI Foundry yet — try **East US** or **Sweden Central**
> - Your subscription doesn't have enough quota — check the error message for details
> - You need to register resource providers — the portal will guide you through this

### Step 5 — Explore the Project Dashboard

Once provisioning completes, you'll be taken to your **project dashboard**. Take a moment to explore:

| Section | Purpose |
|---|---|
| **Overview** | Project summary, recent activity, and quick actions |
| **Playgrounds** | Interactive spaces to test models with prompts |
| **Deployments** | Models you've deployed and can use via API |
| **Data** | Upload and manage datasets for fine-tuning |
| **Evaluation** | Test and measure your AI's performance |
| **Tracing** | Monitor and debug your AI application's behavior |

### ✅ Checkpoint

> Your project dashboard should be visible and look similar to this layout:
>
> ```
> ┌─────────────────────────────────────────────┐
> │  my-first-ai-project                        │
> │                                             │
> │  Overview | Playgrounds | Deployments | ...  │
> │                                             │
> │  [Recent activity]     [Quick actions]       │
> └─────────────────────────────────────────────┘
> ```
>
> **What to verify:** You can see your project name, the navigation tabs, and no error messages.

---

## Exercise 4: Your First Prompt (15 min)

This is the fun part — you're going to have a conversation with an AI model!

### Step 1 — Open the Playground

1. Inside your project, click **"Playgrounds"** in the left sidebar.
2. Select **"Chat"** playground.

### Step 2 — Deploy a Model (If Needed)

If you don't have a model deployed yet:

1. You'll see a prompt to **deploy a model** — click it.
2. Select **GPT-4o-mini** (it's cost-effective and perfect for learning).
3. Keep the default settings and click **Deploy**.
4. Wait for the deployment to complete (usually under a minute).

> **💡 Why GPT-4o-mini?** It's one of the most affordable models, responds quickly, and is more than capable for learning. You can always upgrade later.

### Step 3 — Set Your System Message

The **system message** tells the model how to behave. Think of it as giving the AI its job description.

1. Find the **"System message"** box (usually on the left side or top of the playground).
2. Replace the default text with:

```
You are a helpful assistant that explains AI concepts in simple terms.
```

3. Click **"Apply changes"** if prompted.

### Step 4 — Send Your First Message

1. In the **user message** box at the bottom, type:

```
What is a large language model? Explain it like I'm 10 years old.
```

2. Press **Enter** or click **Send**.
3. Watch the response stream in — the model is generating text in real time!

> **🎉 Congratulations!** You just sent your first prompt to an AI model through Azure AI Foundry!

### Step 5 — Experiment with Temperature

Temperature controls how creative (or predictable) the model's responses are.

1. Find the **"Temperature"** slider in the configuration panel (usually on the right side).
2. Set it to **0.1** (very low) and send the same prompt again.
3. Now set it to **1.0** (high) and send it one more time.

| Temperature | Behavior | Good For |
|---|---|---|
| **0.0 – 0.3** | Very focused, consistent, predictable | Factual answers, code generation, data extraction |
| **0.4 – 0.7** | Balanced creativity and accuracy | General conversation, explanations |
| **0.8 – 1.0** | More creative, varied, surprising | Brainstorming, creative writing, storytelling |

> **💡 What to notice:** At low temperature, the response will be almost the same each time. At high temperature, you'll get different wording, examples, and analogies.

### Step 6 — Try Different System Messages

Clear the chat and try these system messages one at a time:

**Pirate Assistant:**
```
You are a pirate captain who explains technology using nautical metaphors. Always start with "Ahoy!"
```

**Socratic Teacher:**
```
You are a Socratic teacher. Never give direct answers. Instead, guide the student to discover the answer by asking thoughtful questions.
```

**Code Buddy:**
```
You are a friendly coding mentor. When asked about concepts, always include a simple code example in Python.
```

For each one, ask: *"What is artificial intelligence?"* and observe how the responses differ.

### ✅ Checkpoint

> Take a screenshot of your playground showing:
> - Your system message
> - At least one user message and the model's response
> - The temperature setting you used

---

## Exercise 5: Reflection (5 min)

Take a few minutes to write down your answers to these questions in your own notes. There are no wrong answers — this is about capturing your first impressions.

### Reflection Questions

1. **What surprised you most about the model catalog?**
   - Was it the number of models? The variety of providers? Something else?

2. **How is the Playground different from ChatGPT?**
   - Think about: system messages, temperature control, model selection, the fact that it's a development tool vs. a consumer product.

3. **What would you want to build with AI Foundry?**
   - Don't worry about feasibility right now — dream big! A chatbot? An image generator? An AI tutor? A code reviewer?

> **📝 Save your reflections!** We'll revisit them in Week 12 and Week 24 to see how your understanding has grown.

---

## Cleanup

### If You're Using the Free Tier

You can **leave everything running** — free tier resources don't incur charges. Your project will be there next week when we continue.

### If You're Using a Paid Subscription

To avoid unexpected charges, you can delete the resources:

1. Go to the [Azure Portal](https://portal.azure.com).
2. Navigate to **Resource groups**.
3. Find `rg-ai-foundry-training`.
4. Click **Delete resource group**.
5. Type the resource group name to confirm.
6. Click **Delete**.

> **⚠️ Warning:** Deleting the resource group removes everything — the project, deployments, and all data. You'll need to recreate them in the next lab.

> **💡 Recommendation:** If your subscription has free credits (e.g., Visual Studio subscriber benefits or Azure free account credits), keep the resources for continuity across weekly labs.

---

## Bonus Challenge 🏆

Already finished? Try these extra activities:

### Challenge 1: Compare Models Side by Side

1. In the playground, look for a **"Compare"** or **"Add model"** option.
2. Deploy a second model (e.g., **Phi-3-mini** or another available model).
3. Send the same prompt to both models simultaneously.
4. Compare: Which response did you prefer? Why?

### Challenge 2: Explore Content Filters

1. Go to your model deployment settings.
2. Find the **"Content filters"** section.
3. Review the default filter settings — these are part of Azure's **Responsible AI** framework.
4. Notice the categories: hate, sexual, violence, self-harm.
5. Understand that these filters are on by default to ensure safe AI usage.

> **💡 Why content filters matter:** In enterprise environments, content filters protect your users and your brand. We'll cover Responsible AI in depth in Week 5.

---

## Summary

Today you accomplished a lot! Here's what you did:

| ✅ | Accomplishment |
|---|---|
| 1 | Signed into Azure AI Foundry and explored the portal |
| 2 | Browsed the model catalog with 1,800+ models |
| 3 | Created your first AI Foundry project |
| 4 | Sent your first prompt and experimented with settings |
| 5 | Reflected on what you learned and what you want to build |

### What's Next

In **Week 2**, we'll dive deeper into:
- Understanding AI models in more detail
- Exploring prompt engineering basics
- Working with the Playground's advanced features

---

*Lab 1 of 45 — Azure AI Foundry Training Course*
