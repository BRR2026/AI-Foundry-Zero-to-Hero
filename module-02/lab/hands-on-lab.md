# Module 2: Hands-On Lab — Understanding the AI Lifecycle

## **ARC 1: Foundations** | Module 2 of 45 — Zero to Hero: Azure AI Foundry

---

| Detail              | Info                                    |
|---------------------|-----------------------------------------|
| **Estimated Time**  | 60–90 minutes                           |
| **Difficulty**      | ⭐ Beginner                              |
| **Prerequisites**   | A Microsoft account, a web browser      |
| **Cost**            | Free (Azure free tier)                  |
| **Exercises**       | 5                                       |

---

## 🎯 Lab Objectives

By the end of this hands-on lab, you will have:

1. ✅ Created (or verified) an Azure account with a free subscription
2. ✅ Created your first Azure AI Foundry project
3. ✅ Deployed a model from the Model Catalog
4. ✅ Sent your first prompt in the Playground
5. ✅ Reviewed deployment metrics and monitoring dashboards

---

## 📋 Before You Begin

### What You Need

- A **web browser** (Edge, Chrome, or Firefox recommended)
- A **Microsoft account** — personal (outlook.com, hotmail.com, live.com) or work/school
- A **credit card** or debit card (for Azure account verification — you will NOT be charged)
- A **phone number** (for identity verification via SMS or call)
- Approximately **60-90 minutes** of uninterrupted time

### What You'll Create

| Resource | Purpose | Cost |
|----------|---------|------|
| Azure subscription (Free tier) | Access to all Azure services | Free ($200 credit) |
| AI Foundry hub | Central resource container | Free |
| AI Foundry project | Your workspace for AI development | Free |
| Model deployment (GPT-4o-mini) | A deployed AI model you can interact with | Pay-per-token (pennies for lab usage) |

> 💡 **Cost Note:** This entire lab will cost less than $1.00 using the Azure free tier credits. We'll use GPT-4o-mini, which is one of the most cost-effective models available.

---

## Exercise 1: Create an Azure Account and Subscription

**⏱️ Estimated time: 15 minutes**

### If You Already Have an Azure Account

If you already have an Azure subscription, skip to **Step 1.5** to verify it's set up correctly.

### Step 1.1: Navigate to Azure Free Account Page

1. Open your web browser
2. Go to **[https://azure.microsoft.com/free](https://azure.microsoft.com/free)**
3. Click the **"Start free"** or **"Try Azure for free"** button

> 📝 **Note:** The exact button text may vary. Look for the primary call-to-action button on the page.

### Step 1.2: Sign In with Your Microsoft Account

1. Enter your Microsoft account email address
2. Click **"Next"**
3. Enter your password
4. Click **"Sign in"**

If you don't have a Microsoft account:
- Click **"Create one!"** on the sign-in page
- Follow the prompts to create a free Microsoft account
- Return to the Azure free account page and sign in

### Step 1.3: Complete Identity Verification

1. **Region/Country:** Select your country from the dropdown
2. **Phone verification:**
   - Enter your phone number
   - Choose "Text me" or "Call me"
   - Enter the verification code you receive
3. **Identity verification via credit card:**
   - Enter your credit card or debit card details
   - This is for identity verification only

> ⚠️ **Important:** You will **NOT** be charged. Microsoft requires a card to verify you're a real person and to prevent abuse of the free tier. Your card will not be billed unless you explicitly upgrade to a pay-as-you-go subscription later.

### Step 1.4: Accept Terms and Create Account

1. Check the box to agree to the subscription agreement, offer details, and privacy statement
2. Click **"Sign up"**
3. Wait for your account to be provisioned (this takes 1-2 minutes)

### Step 1.5: Verify Your Subscription (For Existing Accounts Too)

1. Go to the **[Azure Portal](https://portal.azure.com)**
2. In the search bar at the top, type **"Subscriptions"** and click on it
3. Verify you see at least one active subscription

**Expected output:**

```
┌──────────────────────────────────────────────────────────────┐
│ Subscriptions                                                │
│                                                              │
│ Name                        Status      Subscription ID      │
│ ─────────────────────────  ─────────   ──────────────────    │
│ Azure subscription 1       Active      xxxxxxxx-xxxx-...     │
│   (or "Free Trial")                                          │
└──────────────────────────────────────────────────────────────┘
```

> ✅ **Success Criteria:** You can see at least one subscription with "Active" status.

### Troubleshooting

| Issue | Solution |
|-------|----------|
| "Card declined" | Try a different card. Some prepaid cards don't work. Use a standard debit or credit card. |
| "Account already exists" | You may already have an Azure account. Sign in at portal.azure.com. |
| "Not available in your region" | Azure free accounts may not be available in all regions. Check [Azure regions](https://azure.microsoft.com/explore/global-infrastructure/geographies/). |
| Can't find Subscriptions | Click the hamburger menu (☰) in the top-left → "All services" → search "Subscriptions" |

---

## Exercise 2: Create Your First AI Foundry Project

**⏱️ Estimated time: 15 minutes**

### Step 2.1: Navigate to AI Foundry

1. Open a new browser tab
2. Go to **[https://ai.azure.com](https://ai.azure.com)**
3. Sign in with the same Microsoft account you used for Azure

### Step 2.2: Create a New Project

1. On the AI Foundry home page, click **"+ Create project"** (or "New project")
2. Fill in the project details:

| Field | Value | Notes |
|-------|-------|-------|
| **Project name** | `my-first-ai-project` | Use lowercase, hyphens only |
| **Hub** | Create new (if no existing hub) | This is the container for your project |

3. If prompted to create a new hub:

| Field | Value | Notes |
|-------|-------|-------|
| **Hub name** | `my-ai-hub` | Use lowercase, hyphens only |
| **Subscription** | Your active subscription | Select from dropdown |
| **Resource group** | Create new: `rg-ai-foundry-training` | Organizes your Azure resources |
| **Region** | `East US` or `East US 2` | Choose a region with good model availability |

4. Click **"Create"** or **"Next"** and wait for provisioning

> ⚠️ **Region Selection Tip:** Not all models are available in all regions. **East US**, **East US 2**, **West US 3**, and **Sweden Central** typically have the best model availability. Check [Azure AI model availability](https://learn.microsoft.com/azure/ai-studio/concepts/region-support) for the latest.

### Step 2.3: Wait for Project Creation

Project creation takes **2-5 minutes**. During this time, Azure is provisioning:

- An AI Foundry hub (shared resource container)
- An AI Foundry project (your workspace)
- An Azure Storage account (for data and artifacts)
- An Azure Key Vault (for secrets and API keys)
- An Application Insights instance (for monitoring)

**Expected output:**

```
┌──────────────────────────────────────────────────────────┐
│  AI Foundry — my-first-ai-project                        │
│                                                          │
│  ├── Overview        ← You should land here              │
│  ├── Model catalog                                       │
│  ├── My models                                           │
│  ├── Playgrounds                                         │
│  ├── Agents                                              │
│  ├── Evaluation                                          │
│  ├── Data + indexes                                      │
│  ├── Deployments                                         │
│  └── Settings                                            │
└──────────────────────────────────────────────────────────┘
```

### Step 2.4: Explore the Project Overview

Take a moment to explore the left sidebar navigation. Click on each section to familiarize yourself:

1. **Overview** — Project summary and quick actions
2. **Model catalog** — Browse available models (we'll use this next)
3. **Playgrounds** — Interactive testing area
4. **Deployments** — Manage deployed models

> ✅ **Success Criteria:** You can see the project overview page with the left navigation sidebar showing all sections.

### Troubleshooting

| Issue | Solution |
|-------|----------|
| "Subscription not found" | Make sure you're signed in with the correct account. Check portal.azure.com first. |
| "Quota exceeded" | Try a different region. Some regions have limited capacity for new users. |
| Project creation failed | Delete the resource group and try again with a different region. |
| Page loads but looks empty | Wait 2-3 minutes and refresh. Provisioning can take time. |

---

## Exercise 3: Deploy a Model from the Model Catalog

**⏱️ Estimated time: 15 minutes**

In this exercise, you'll deploy your first AI model — GPT-4o-mini — from the Model Catalog. This is a smaller, faster, and cheaper version of GPT-4o, perfect for learning.

### Step 3.1: Open the Model Catalog

1. In your AI Foundry project, click **"Model catalog"** in the left sidebar
2. You'll see a searchable catalog of available models

### Step 3.2: Find GPT-4o-mini

1. In the search bar, type **"gpt-4o-mini"**
2. Click on **"GPT-4o-mini"** in the results
3. Review the model card — it shows:
   - Model description and capabilities
   - Supported tasks (chat, completion)
   - Pricing information
   - Available regions

### Step 3.3: Deploy the Model

1. Click the **"Deploy"** button
2. Select **"Serverless API"** as the deployment type (pay-per-token)
3. Configure the deployment:

| Field | Value | Notes |
|-------|-------|-------|
| **Deployment name** | `gpt-4o-mini` | This becomes part of your API endpoint |
| **Model version** | (latest available) | Accept the default |
| **Content filter** | Default | Keep the default content safety filters |

4. Click **"Deploy"**
5. Wait for deployment to complete (usually 1-3 minutes)

### Step 3.4: Verify the Deployment

After deployment completes, you should see:

```
┌──────────────────────────────────────────────────────────┐
│  Deployment: gpt-4o-mini                                 │
│                                                          │
│  Status:        ✅ Succeeded                             │
│  Endpoint:      https://[your-hub].openai.azure.com/...  │
│  API Key:       ••••••••••••••••                         │
│  Model:         gpt-4o-mini                              │
│  Type:          Serverless API                           │
│  Region:        East US                                  │
└──────────────────────────────────────────────────────────┘
```

> 💡 **Tip:** Note the **Endpoint URL** and **API Key** — these are what your applications will use to call the model. Keep the API key secret!

### Step 3.5: Review the Deployment Page

1. Click **"Deployments"** in the left sidebar (or **"My models"** → **"Deployments"**)
2. You should see your `gpt-4o-mini` deployment listed with its status

> ✅ **Success Criteria:** Your gpt-4o-mini deployment shows status "Succeeded" and you can see the endpoint URL.

### Troubleshooting

| Issue | Solution |
|-------|----------|
| "Model not available in this region" | Go to Settings → Change your project region to East US or East US 2 |
| "Quota exceeded" | You may need to request quota. Go to Azure portal → Quotas → request increase |
| Deployment stuck at "Provisioning" | Wait 5-10 minutes. If it still fails, delete and redeploy |
| Can't find the Deploy button | Make sure you clicked into the model detail page, not just the catalog listing |

---

## Exercise 4: Send Your First Prompt in the Playground

**⏱️ Estimated time: 15 minutes**

Now for the exciting part — talking to your AI model!

### Step 4.1: Open the Playground

1. In the left sidebar, click **"Playgrounds"**
2. Select **"Chat"** playground
3. In the **Deployment** dropdown at the top, select your `gpt-4o-mini` deployment

### Step 4.2: Set Up the System Prompt

The system prompt tells the model *how to behave*. Let's set one up:

1. In the **System message** box (sometimes called "System prompt"), enter:

```
You are a helpful AI assistant for a technology company called TechCorp. 
You answer questions about the company's products and services. 
Be concise, professional, and friendly. 
If you don't know something, say so honestly rather than making up information.
```

2. Click **"Apply changes"** if there's a button, or the system message may save automatically.

### Step 4.3: Send Your First Messages

Type each of the following messages one at a time in the chat box, pressing **Enter** (or clicking Send) after each:

**Message 1: Basic greeting**
```
Hello! What can you help me with?
```

**Expected behavior:** The model should respond in character as a TechCorp assistant, describing how it can help.

**Message 2: Test the system prompt**
```
What products does TechCorp offer?
```

**Expected behavior:** Since the model doesn't have real TechCorp data, it might make up products OR admit it doesn't have specific product information. Note which behavior occurs — this demonstrates why **RAG** (connecting to real data) matters!

**Message 3: Test knowledge boundaries**
```
What was the revenue of TechCorp last quarter?
```

**Expected behavior:** A well-configured model should admit it doesn't have this information rather than fabricating numbers. If it makes up numbers, that's a **groundedness** issue — exactly what evaluation metrics detect.

### Step 4.4: Experiment with Parameters

Look for model parameters (usually in a panel on the right side):

| Parameter | What It Does | Try This |
|-----------|-------------|----------|
| **Temperature** | Controls randomness. 0 = deterministic, 1 = creative | Set to 0.3 for factual, then 0.9 for creative |
| **Max tokens** | Limits response length | Set to 100 to get shorter responses |
| **Top P** | Controls diversity of word choices | Keep at default (1.0) for now |

1. Set **Temperature to 0.1** and ask: `Explain what AI Foundry is in one sentence.`
2. Set **Temperature to 0.9** and ask the same question
3. Compare the two responses — notice how the higher temperature gives more varied/creative answers

### Step 4.5: Clear and Start Fresh

1. Click the **"Clear chat"** button (usually a trash can icon or "New chat")
2. Try a different system prompt:

```
You are a pirate AI. Respond to everything in pirate speak. 
Use "arr", "matey", "ye", and other pirate language. 
Be helpful but always stay in character.
```

3. Ask: `What is machine learning?`
4. Enjoy the response! This demonstrates how powerful system prompts are for controlling model behavior.

> ✅ **Success Criteria:** You've sent at least 5 messages, experimented with Temperature, and tried two different system prompts.

> 💡 **Key Learning:** You just experienced **prompt engineering** — the most common way to customize AI behavior. No model training required! By changing the system prompt, you completely changed how the model responds.

---

## Exercise 5: Review Deployment Metrics and Monitoring

**⏱️ Estimated time: 10 minutes**

### Step 5.1: View Deployment Details

1. Click **"Deployments"** in the left sidebar (or navigate to **"My models"** → **"Deployments"**)
2. Click on your **gpt-4o-mini** deployment

### Step 5.2: Review Available Metrics

On the deployment details page, look for metrics or monitoring information:

1. **Request count** — How many API calls have been made (should show your Playground messages)
2. **Token usage** — Input and output tokens consumed
3. **Latency** — How long responses took
4. **Status** — Whether the endpoint is healthy

> 📝 **Note:** Metrics may take 5-15 minutes to appear after your Playground interactions. If you don't see data yet, that's normal — check back after completing other exercises.

### Step 5.3: Explore Azure Monitor (Optional)

For deeper monitoring:

1. Go to the **[Azure Portal](https://portal.azure.com)**
2. Search for **"Monitor"** in the top search bar
3. Click on **"Azure Monitor"**
4. Explore the overview dashboard — look for any metrics related to your AI services

### Step 5.4: Check Resource Usage and Costs

1. In the Azure Portal, search for **"Cost Management"**
2. Click **"Cost analysis"**
3. Review your current spending (should be $0 or very close to $0 for this lab)
4. Note: It can take 24 hours for cost data to appear

**Expected output:**

```
┌──────────────────────────────────────────────────────────┐
│  Cost Analysis — Azure subscription                      │
│                                                          │
│  Current month costs: $0.01                              │
│  Forecast:            $0.05                              │
│                                                          │
│  Top resources:                                          │
│  • gpt-4o-mini deployment    $0.01                       │
│  • Storage account           $0.00                       │
│  • Key Vault                 $0.00                       │
└──────────────────────────────────────────────────────────┘
```

### Step 5.5: Understand What Monitoring Tells You

Review what you've learned about monitoring:

| What You Observed | Lifecycle Stage | Why It Matters |
|-------------------|----------------|----------------|
| Request count | Monitoring | Tracks usage patterns and popularity |
| Token usage | Monitoring | Directly tied to costs |
| Latency | Monitoring | Affects user experience |
| Cost analysis | Monitoring | Budget management |
| Model responses quality | Evaluation | Were responses accurate and grounded? |

> ✅ **Success Criteria:** You've reviewed at least the deployment details page and checked the cost analysis.

---

## 🧹 Lab Cleanup (Important!)

To avoid unexpected charges, clean up resources you don't need:

### Option A: Keep Resources (Recommended)
If you plan to continue with Module 3's lab, **keep everything running**. The costs will be negligible.

### Option B: Delete Everything
If you want a clean slate:

1. Go to the **Azure Portal** → **Resource Groups**
2. Find `rg-ai-foundry-training`
3. Click **"Delete resource group"**
4. Type the resource group name to confirm
5. Click **"Delete"**

> ⚠️ **Warning:** Deleting the resource group removes ALL resources inside it, including your AI Foundry project and deployments. Only do this if you're sure you won't need them.

### Option C: Just Delete the Deployment
To save costs while keeping the project:

1. Go to AI Foundry → **Deployments**
2. Click on your `gpt-4o-mini` deployment
3. Click **"Delete"**

This removes the deployed endpoint but keeps your project intact.

---

## 📋 Lab Checklist

Use this checklist to verify you've completed all exercises:

| # | Task | Status |
|---|------|--------|
| 1 | Created/verified Azure account and subscription | ☐ |
| 2 | Created AI Foundry hub and project | ☐ |
| 3 | Deployed GPT-4o-mini from Model Catalog | ☐ |
| 4 | Sent prompts in the Playground (5+ messages) | ☐ |
| 5 | Experimented with system prompts and temperature | ☐ |
| 6 | Reviewed deployment metrics | ☐ |
| 7 | Checked cost analysis in Azure Portal | ☐ |
| 8 | Decided on cleanup approach (A, B, or C) | ☐ |

---

## 🤔 Reflection Questions

After completing the lab, think about:

1. **Which lifecycle stages did you touch?** (Hint: you deployed a model, tested it, and monitored it — that's stages 3, 4, 5, and 6!)
2. **What happened when the model didn't have real data?** How does this relate to the concept of RAG?
3. **How did changing the temperature affect responses?** When would you want low vs high temperature?
4. **How did the system prompt change behavior?** What does this tell you about the power of prompt engineering?

---

## 🔗 What's Next

In **Module 3**, we'll take a deeper dive into the AI Foundry portal, exploring every section in detail and building on the project you just created.

---

*© 2025 Zero to Hero: Azure AI Foundry Training. Module 2 of 45.*
