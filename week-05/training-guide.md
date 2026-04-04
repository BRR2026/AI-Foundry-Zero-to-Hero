# Week 5: Setting Up Your Environment + First "Hello World" Model

## **ARC 1: Foundations** | Week 5 of 45 — Zero to Hero: Azure AI Foundry

---

| Detail              | Info                                                    |
|---------------------|---------------------------------------------------------|
| **Arc**             | ARC 1 — Foundations (Weeks 1–5)                         |
| **Week**            | 5                                                       |
| **Title**           | Setting Up Your Environment + First "Hello World" Model |
| **Estimated Time**  | 3–4 hours                                               |
| **Difficulty**      | ⭐⭐ Beginner–Intermediate                              |
| **Last Updated**    | July 2025                                               |

---

## 🎯 Learning Objectives

By the end of this week, you will be able to:

1. **Install and configure** a complete Azure AI development environment (Python, VS Code, Azure CLI, SDKs).
2. **Create** an AI Foundry Hub and Project using the portal, CLI, and Python SDK.
3. **Deploy** your first model (GPT-4o-mini) and understand deployment options.
4. **Send** your first AI prompt via four different methods: Playground, Python SDK, REST API, and Azure CLI.
5. **Interpret** API responses including tokens, usage statistics, and finish reasons.
6. **Experiment** with system prompts, temperature, and other parameters.
7. **Build** a simple interactive chatbot application in Python.
8. **Manage** costs by setting quotas and monitoring spend.

---

## 📋 Prerequisites

- **Completed Weeks 1–4** of this training course (foundational understanding of AI Foundry).
- An **Azure subscription** (free tier works — [azure.microsoft.com/free](https://azure.microsoft.com/free)).
- A computer with **admin rights** to install software.
- **Internet connection** for Azure services.

> 💡 **Tip:** This is the most hands-on week so far! Weeks 1–4 built your theoretical foundation — now it's time to get your hands dirty with real code and real AI models. Set aside 3–4 uninterrupted hours.

---

## 1. Introduction — Your Hands-On Journey Begins

### Welcome to Build Mode

Congratulations! You've made it through four weeks of foundational learning. You understand what AI Foundry is, how Azure's AI ecosystem works, the technology stack behind modern AI, and the architecture that ties it all together. Now it's time for the exciting part: **building something real**.

By the end of today, you'll have:

- ✅ A fully configured development environment
- ✅ Your first AI Foundry Hub and Project
- ✅ A deployed AI model ready to respond to prompts
- ✅ Working Python code that talks to GPT-4o-mini
- ✅ An interactive chatbot you built yourself

This is the "Hello World" moment — the first time you make something work. Every AI engineer remembers this day.

### What You'll Build

| Component | Description |
|-----------|-------------|
| **Development environment** | Python + VS Code + Azure CLI + SDKs |
| **AI Foundry project** | Hub and Project in Azure |
| **Model deployment** | GPT-4o-mini (fast, cheap, capable) |
| **Hello World script** | First API call via Python SDK |
| **REST API test** | Same call via curl |
| **Interactive chatbot** | Multi-turn chat application |

---

## 2. Prerequisites Check

Before we begin installing anything, let's verify your starting point.

### Hardware Requirements

| Requirement | Minimum | Recommended |
|-------------|---------|-------------|
| **RAM** | 4 GB | 8 GB+ |
| **Disk space** | 2 GB free | 5 GB+ free |
| **CPU** | Any modern processor | Multi-core |
| **Internet** | Required | Stable broadband |

### Software Requirements

| Software | Version | Purpose |
|----------|---------|---------|
| **Operating System** | Windows 10/11, macOS 12+, Ubuntu 20.04+ | Development platform |
| **Web browser** | Edge or Chrome (latest) | Azure portal access |
| **Python** | 3.11 or higher | Primary programming language |
| **VS Code** | Latest | Code editor |
| **Azure CLI** | 2.60+ | Command-line Azure management |
| **Git** | 2.40+ | Version control |

### Azure Requirements

| Requirement | Details |
|-------------|---------|
| **Azure account** | Free or Pay-As-You-Go |
| **Subscription** | Active with billing enabled |
| **Permissions** | Contributor role on subscription |
| **Region** | East US 2 recommended (best model availability) |

> ⚠️ **Warning:** Some Azure AI models require a Pay-As-You-Go subscription. The free tier may have limitations on model deployments. If you encounter deployment errors, consider upgrading to Pay-As-You-Go (you'll still only pay for what you use).

---

## 3. Installing Your Development Tools

### 3.1 Python 3.11+

Python is the primary language for AI development. Let's install it.

**Windows:**
```bash
# Option 1: Download from python.org
# Visit https://www.python.org/downloads/ and download Python 3.11+
# ⚠️ IMPORTANT: Check "Add Python to PATH" during installation!

# Option 2: Using winget
winget install -e --id Python.Python.3.12

# Verify installation
python --version
# Expected output: Python 3.12.x (or 3.11.x)

pip --version
# Expected output: pip 24.x.x from ...
```

**macOS:**
```bash
# Using Homebrew
brew install python@3.12

# Verify
python3 --version
pip3 --version
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt update
sudo apt install python3.12 python3.12-venv python3-pip

# Verify
python3 --version
pip3 --version
```

> 💡 **Tip:** On Windows, use `python` and `pip`. On macOS/Linux, you may need `python3` and `pip3`. If in doubt, try both and see which responds.

### 3.2 Visual Studio Code

VS Code is the recommended editor for AI Foundry development.

1. Download from [code.visualstudio.com](https://code.visualstudio.com/)
2. Install with default settings
3. Open VS Code and install these extensions:

| Extension | ID | Purpose |
|-----------|-----|---------|
| **Python** | `ms-python.python` | Python language support |
| **Azure Account** | `ms-vscode.azure-account` | Azure sign-in |
| **Azure Resources** | `ms-azuretools.vscode-azureresourcegroups` | Manage Azure resources |
| **Azure AI** | `ms-azure-devops.azure-ai` | AI Foundry integration |
| **Jupyter** | `ms-toolsai.jupyter` | Notebook support |
| **REST Client** | `humao.rest-client` | Test REST APIs in VS Code |

Install from the command line:
```bash
code --install-extension ms-python.python
code --install-extension ms-vscode.azure-account
code --install-extension ms-azuretools.vscode-azureresourcegroups
code --install-extension ms-toolsai.jupyter
code --install-extension humao.rest-client
```

### 3.3 Azure CLI

The Azure CLI lets you manage Azure resources from the command line.

**Windows:**
```bash
winget install -e --id Microsoft.AzureCLI
```

**macOS:**
```bash
brew install azure-cli
```

**Linux (Ubuntu/Debian):**
```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

**Verify installation:**
```bash
az --version
# Expected: azure-cli 2.60.0 or higher
```

**Install the ML extension** (required for AI Foundry commands):
```bash
az extension add --name ml --yes
az extension update --name ml
```

### 3.4 Git

**Windows:**
```bash
winget install -e --id Git.Git
```

**macOS:**
```bash
brew install git
```

**Linux:**
```bash
sudo apt install git
```

**Verify:**
```bash
git --version
# Expected: git version 2.40+ 
```

---

## 4. Azure CLI Authentication

### 4.1 Signing In

```bash
# Sign in to Azure (opens browser for authentication)
az login

# If you have multiple subscriptions, list them
az account list --output table

# Set the subscription you want to use
az account set --subscription "Your Subscription Name or ID"

# Verify you're using the right subscription
az account show --output table
```

**Expected output from `az account show`:**
```
EnvironmentName    Name                    State    IsDefault
-----------------  ----------------------  -------  ----------
AzureCloud         My AI Development Sub   Enabled  True
```

### 4.2 Verifying Permissions

```bash
# Check your role assignments
az role assignment list --assignee $(az account show --query user.name -o tsv) --output table
```

You need at least **Contributor** role on the subscription or resource group where you'll create resources.

### 4.3 Troubleshooting Authentication

| Issue | Solution |
|-------|----------|
| Browser doesn't open | Use `az login --use-device-code` for device code flow |
| Multiple tenants | Use `az login --tenant YOUR_TENANT_ID` |
| Token expired | Run `az login` again |
| Corporate proxy | Set `HTTP_PROXY` and `HTTPS_PROXY` environment variables |
| MFA required | Browser-based login handles MFA automatically |

> 🔑 **Key Concept:** Azure CLI stores your credentials locally in `~/.azure/`. These tokens auto-refresh but expire after extended inactivity. If you get authentication errors later, just run `az login` again.

---

## 5. Creating Your AI Foundry Hub and Project

AI Foundry uses a two-level hierarchy: **Hub** → **Project**. A Hub is the top-level container that holds shared resources (connections, compute, storage). Projects sit inside Hubs and represent individual workloads.

```
Azure Subscription
└── Resource Group
    └── AI Foundry Hub (shared resources)
        ├── Project A (your Hello World)
        ├── Project B (another workload)
        └── Project C (team project)
```

### Method 1: Portal (Recommended for First Time)

1. **Navigate** to [ai.azure.com](https://ai.azure.com) and sign in.
2. Click **"+ Create project"** on the home page.
3. **Project name:** `proj-hello-world`
4. Click **"Create a new hub"**:
   - **Hub name:** `hub-ai-foundry-dev`
   - **Subscription:** Select yours
   - **Resource group:** Create new → `rg-ai-foundry-dev`
   - **Location:** `East US 2`
5. Click **"Next"** and review the settings.
6. Click **"Create"** and wait 2–3 minutes for provisioning.

> 💡 **Tip:** East US 2 is recommended because it has the best availability of GPT-4o-mini and other models. Check [Azure products by region](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/) for current availability.

### Method 2: Azure CLI

```bash
# Step 1: Create a resource group
az group create \
  --name rg-ai-foundry-dev \
  --location eastus2

# Step 2: Create an AI Foundry Hub
az ml workspace create \
  --kind hub \
  --name hub-ai-foundry-dev \
  --resource-group rg-ai-foundry-dev \
  --location eastus2

# Step 3: Create a Project under the Hub
az ml workspace create \
  --kind project \
  --name proj-hello-world \
  --resource-group rg-ai-foundry-dev \
  --hub-name hub-ai-foundry-dev

# Step 4: Verify creation
az ml workspace show \
  --name proj-hello-world \
  --resource-group rg-ai-foundry-dev \
  --query "{name:name, kind:kind, location:location, provisioningState:provisioning_state}" \
  --output table
```

### Method 3: Python SDK

```python
"""Create AI Foundry Hub and Project using Python SDK"""
from azure.identity import DefaultAzureCredential
from azure.mgmt.resource import ResourceManagementClient
from azure.ai.ml import MLClient
from azure.ai.ml.entities import Hub, Project

credential = DefaultAzureCredential()
subscription_id = "your-subscription-id"

# Create resource group
resource_client = ResourceManagementClient(credential, subscription_id)
resource_client.resource_groups.create_or_update(
    "rg-ai-foundry-dev",
    {"location": "eastus2"}
)

# Create Hub
ml_client = MLClient(credential, subscription_id, "rg-ai-foundry-dev")
hub = Hub(name="hub-ai-foundry-dev", location="eastus2")
ml_client.workspaces.begin_create_or_update(hub).result()
print("✅ Hub created!")

# Create Project
project = Project(
    name="proj-hello-world",
    hub_id=hub.id
)
ml_client.workspaces.begin_create_or_update(project).result()
print("✅ Project created!")
```

> 🏭 **AI Foundry Note:** Hub creation also provisions supporting resources: a Storage Account, Key Vault, and Application Insights. This is automatic — you don't need to create them separately.

---

## 6. Python Environment Setup

### 6.1 Create Your Project Directory

```bash
# Create project folder
mkdir ai-foundry-hello-world
cd ai-foundry-hello-world

# Create virtual environment
python -m venv .venv

# Activate virtual environment
# Windows (Command Prompt):
.venv\Scripts\activate

# Windows (PowerShell):
.venv\Scripts\Activate.ps1

# macOS/Linux:
source .venv/bin/activate

# You should see (.venv) in your prompt:
# (.venv) C:\Users\you\ai-foundry-hello-world>
```

### 6.2 Install Required Packages

```bash
# Install Azure AI packages
pip install azure-ai-projects azure-ai-inference azure-identity python-dotenv

# Verify installation
pip list | findstr azure
# Expected output:
# azure-ai-inference     1.x.x
# azure-ai-projects      1.x.x
# azure-core             1.x.x
# azure-identity         1.x.x
```

### 6.3 Create Your .env File

Create a file named `.env` in your project root:

```env
# Azure AI Foundry Configuration
# ⚠️ NEVER commit this file to Git!

AZURE_SUBSCRIPTION_ID=your-subscription-id
AZURE_RESOURCE_GROUP=rg-ai-foundry-dev
AZURE_AI_PROJECT_NAME=proj-hello-world

# Model Endpoint (get from AI Foundry portal → Deployments)
AZURE_AI_ENDPOINT=https://your-endpoint.openai.azure.com/
AZURE_OPENAI_API_KEY=your-api-key-here
AZURE_OPENAI_DEPLOYMENT_NAME=gpt-4o-mini
AZURE_OPENAI_API_VERSION=2024-12-01-preview
```

### 6.4 Create .gitignore

```gitignore
# Environment
.env
.venv/

# Python
__pycache__/
*.pyc
*.pyo
*.egg-info/
dist/
build/

# IDE
.vscode/settings.json
.idea/

# OS
.DS_Store
Thumbs.db
```

### 6.5 Create requirements.txt

```text
azure-ai-projects>=1.0.0
azure-ai-inference>=1.0.0
azure-identity>=1.16.0
python-dotenv>=1.0.0
```

> ⚠️ **Warning:** The `.env` file contains sensitive credentials. **NEVER** commit it to Git. Always add `.env` to your `.gitignore` file. For production, use Azure Key Vault or managed identity instead of API keys.

---

## 7. Deploying Your First Model

### Why GPT-4o-mini?

For development and learning, GPT-4o-mini is the ideal choice:

| Reason | Details |
|--------|---------|
| **Cost** | $0.15/1M input tokens — 16x cheaper than GPT-4o |
| **Speed** | Faster inference than larger models |
| **Capability** | Handles most tasks surprisingly well |
| **Availability** | Available in most Azure regions |
| **Rate limits** | Higher default quotas for testing |

### Model Cost Comparison

| Model | Input (per 1M tokens) | Output (per 1M tokens) | Best For |
|-------|----------------------|------------------------|----------|
| **GPT-4o-mini** | $0.15 | $0.60 | Development, testing, simple tasks |
| **GPT-4.1-nano** | $0.10 | $0.40 | Cheapest option, basic tasks |
| **GPT-4.1-mini** | $0.40 | $1.60 | Balance of cost and capability |
| **GPT-4o** | $2.50 | $10.00 | Complex reasoning, production |
| **GPT-4.1** | $2.00 | $8.00 | Advanced tasks, long context |
| **o4-mini** | $1.10 | $4.40 | Reasoning tasks (thinking model) |

> 💡 **Tip:** For this course, always start with GPT-4o-mini. It'll cost you pennies during development. A typical "Hello World" session costs less than $0.01!

### Deploy via Portal

1. Go to [ai.azure.com](https://ai.azure.com) → Your project
2. Click **"Models + endpoints"** in the left sidebar
3. Click **"+ Deploy model"** → **"Deploy base model"**
4. Search for **"gpt-4o-mini"**
5. Click **"Confirm"**
6. Configure:
   - **Deployment name:** `gpt-4o-mini`
   - **Model version:** Latest
   - **Deployment type:** Global Standard (recommended)
   - **Tokens per minute rate limit:** 30K (default is fine)
7. Click **"Deploy"**
8. Wait for status to show **"Succeeded"**

### Get Your Endpoint and Key

After deployment:
1. Click on the deployment name
2. Copy the **Target URI** — this is your endpoint
3. Click **"Show Key"** and copy the **API key**
4. Update your `.env` file with these values

### Deploy via CLI

```bash
# Deploy GPT-4o-mini (after hub/project creation)
az ml online-deployment create \
  --name gpt-4o-mini \
  --endpoint-name my-ai-endpoint \
  --model azureai://modelId/gpt-4o-mini \
  --resource-group rg-ai-foundry-dev \
  --workspace-name proj-hello-world
```

> 🏭 **AI Foundry Note:** The simplest deployment method is through the portal. CLI and SDK deployment are more complex and require additional configuration. For learning purposes, use the portal.

---

## 8. The "Hello World" of AI — Four Ways

This is the moment you've been building toward. Let's send your first prompt to an AI model in four different ways.

### Method 1: Playground (No Code)

The Playground is AI Foundry's built-in testing tool — no coding required.

1. Navigate to **ai.azure.com** → Your project → **Playground**
2. Select the **Chat** playground
3. Choose your deployed model: `gpt-4o-mini`
4. In the **System message** box, enter:
   ```
   You are a helpful assistant.
   ```
5. In the **User message** box, type:
   ```
   Say 'Hello, World!' and explain what Azure AI Foundry is in one sentence.
   ```
6. Click **Send** (or press Enter)
7. Read the response!

**Expected response:**
```
Hello, World! Azure AI Foundry is Microsoft's unified platform for building,
deploying, and managing AI applications using a catalog of models, integrated
tools, and enterprise-grade security.
```

🎉 **Congratulations!** You just interacted with an AI model through Azure AI Foundry!

### Method 2: Python SDK

Create a file named `hello_world.py`:

```python
"""
Hello World — Azure AI Foundry + Python SDK
Week 5: Your first programmatic AI interaction
"""
import os
from dotenv import load_dotenv
from azure.ai.inference import ChatCompletionsClient
from azure.ai.inference.models import SystemMessage, UserMessage
from azure.core.credentials import AzureKeyCredential

# Load environment variables from .env file
load_dotenv()

# Get configuration
endpoint = os.getenv("AZURE_AI_ENDPOINT")
api_key = os.getenv("AZURE_OPENAI_API_KEY")
model = os.getenv("AZURE_OPENAI_DEPLOYMENT_NAME", "gpt-4o-mini")

# Create the client
client = ChatCompletionsClient(
    endpoint=endpoint,
    credential=AzureKeyCredential(api_key)
)

# Send your first prompt!
response = client.complete(
    model=model,
    messages=[
        SystemMessage(content="You are a helpful assistant."),
        UserMessage(content="Say 'Hello, World!' and explain what Azure AI Foundry is in one sentence.")
    ],
    temperature=0.7,
    max_tokens=200
)

# Display results
print("=" * 60)
print("🌍 HELLO WORLD — Azure AI Foundry + Python SDK")
print("=" * 60)
print()
print(f"Response: {response.choices[0].message.content}")
print()
print("--- Token Usage ---")
print(f"  Prompt tokens:     {response.usage.prompt_tokens}")
print(f"  Completion tokens: {response.usage.completion_tokens}")
print(f"  Total tokens:      {response.usage.total_tokens}")
print(f"  Finish reason:     {response.choices[0].finish_reason}")
print()
print("✅ Your first Azure AI Foundry API call worked!")
```

**Run it:**
```bash
python hello_world.py
```

**Expected output:**
```
============================================================
🌍 HELLO WORLD — Azure AI Foundry + Python SDK
============================================================

Response: Hello, World! Azure AI Foundry is Microsoft's unified
platform that enables developers to build, evaluate, and deploy
AI applications using a vast catalog of models with integrated
tools for prompt engineering, evaluation, and responsible AI.

--- Token Usage ---
  Prompt tokens:     30
  Completion tokens: 42
  Total tokens:      72
  Finish reason:     stop

✅ Your first Azure AI Foundry API call worked!
```

> 🔑 **Key Concept:** Notice the `finish_reason: stop` — this means the model completed its response naturally. We'll explore what other finish reasons mean in Section 9.

### Method 3: REST API (curl)

For those who want to understand the raw HTTP communication:

**Linux/macOS:**
```bash
curl -X POST "${AZURE_AI_ENDPOINT}openai/deployments/gpt-4o-mini/chat/completions?api-version=2024-12-01-preview" \
  -H "Content-Type: application/json" \
  -H "api-key: ${AZURE_OPENAI_API_KEY}" \
  -d '{
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "Say Hello World and tell me what Azure AI Foundry is."}
    ],
    "temperature": 0.7,
    "max_tokens": 200
  }'
```

**Windows PowerShell:**
```powershell
$headers = @{
    "Content-Type" = "application/json"
    "api-key" = $env:AZURE_OPENAI_API_KEY
}

$body = @{
    messages = @(
        @{ role = "system"; content = "You are a helpful assistant." }
        @{ role = "user"; content = "Say Hello World and tell me what Azure AI Foundry is." }
    )
    temperature = 0.7
    max_tokens = 200
} | ConvertTo-Json -Depth 3

$response = Invoke-RestMethod `
    -Uri "$($env:AZURE_AI_ENDPOINT)openai/deployments/gpt-4o-mini/chat/completions?api-version=2024-12-01-preview" `
    -Method POST `
    -Headers $headers `
    -Body $body

$response.choices[0].message.content
```

**The raw JSON response looks like this:**
```json
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "created": 1719500000,
  "model": "gpt-4o-mini",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Hello, World! Azure AI Foundry is..."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 28,
    "completion_tokens": 45,
    "total_tokens": 73
  }
}
```

### Method 4: Azure CLI

```bash
# Create a request file
cat > request.json << 'EOF'
{
  "messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Say Hello World!"}
  ],
  "max_tokens": 100
}
EOF

# Invoke the endpoint
az ml online-endpoint invoke \
  --name your-endpoint-name \
  --request-file request.json \
  --resource-group rg-ai-foundry-dev \
  --workspace-name proj-hello-world
```

> 💡 **Tip:** The Python SDK method is the most practical for development. The REST API method helps you understand what's happening under the hood. The Playground is great for quick testing. Use whichever feels most natural to you!

---

## 9. Understanding the Response

Every API response contains critical information beyond just the text. Let's decode it.

### Response Structure

```json
{
  "id": "chatcmpl-abc123def456",
  "object": "chat.completion",
  "created": 1719500000,
  "model": "gpt-4o-mini",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Hello, World! Azure AI Foundry is..."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 28,
    "completion_tokens": 45,
    "total_tokens": 73
  }
}
```

### Key Fields Explained

| Field | Description |
|-------|-------------|
| `id` | Unique identifier for this completion (useful for logging) |
| `model` | The model that generated the response |
| `choices` | Array of responses (usually just one) |
| `choices[0].message.content` | **The actual response text** |
| `choices[0].finish_reason` | Why the model stopped generating |
| `usage.prompt_tokens` | Tokens used by your input (system + user messages) |
| `usage.completion_tokens` | Tokens used by the model's response |
| `usage.total_tokens` | Sum of prompt + completion tokens |

### Understanding Finish Reasons

| finish_reason | Meaning | What to Do |
|---------------|---------|------------|
| `stop` | Model completed naturally | Response is complete ✅ |
| `length` | Hit `max_tokens` limit | Increase `max_tokens` if response is truncated |
| `content_filter` | Content safety triggered | Modify your prompt to avoid policy violations |

### What Are Tokens?

Tokens are the fundamental units of text processing. They're roughly:
- **~4 characters** in English text
- **~¾ of a word** on average

| Text | Approximate Tokens |
|------|-------------------|
| "Hello" | 1 token |
| "Hello, World!" | 3 tokens |
| "Azure AI Foundry" | 4 tokens |
| A typical sentence (15 words) | ~20 tokens |
| A full page of text (~500 words) | ~675 tokens |

> 🔑 **Key Concept:** You pay for tokens — both input (your prompt) and output (the response). Every word in your system prompt, user message, and conversation history counts toward input tokens. Keep this in mind when designing chatbots with long conversation histories!

### Token Counting in Python

```python
# After getting a response, check token usage:
print(f"Input cost:  ${response.usage.prompt_tokens * 0.15 / 1_000_000:.6f}")
print(f"Output cost: ${response.usage.completion_tokens * 0.60 / 1_000_000:.6f}")
print(f"Total cost:  ${(response.usage.prompt_tokens * 0.15 + response.usage.completion_tokens * 0.60) / 1_000_000:.6f}")
```

---

## 10. System Prompts and Their Effect

### What Are System Prompts?

The **system prompt** (or system message) sets the behavior, personality, and constraints for the AI model. It's like giving the model a role and set of instructions before the conversation begins.

### The Three Message Roles

| Role | Purpose | Who Writes It |
|------|---------|---------------|
| `system` | Sets behavior and context | Developer (you) |
| `user` | The actual question or input | End user |
| `assistant` | The model's response | AI model |

### System Prompt Examples

**Professional Assistant:**
```python
SystemMessage(content="You are a professional cloud architect. Provide clear, technical explanations. Use bullet points for lists. Always mention relevant Azure services.")
```

**For Kids:**
```python
SystemMessage(content="You are a friendly teacher explaining things to a 10-year-old. Use simple words, fun analogies, and occasional emojis. Keep responses under 3 sentences.")
```

**Pirate Translator:**
```python
SystemMessage(content="You are a pirate. Respond to everything in pirate speak. Use 'Arrr', 'matey', 'ye', and other pirate vocabulary. Always end with a pirate saying.")
```

**JSON Responder:**
```python
SystemMessage(content="You respond only in valid JSON format. Every response must be a JSON object with keys: 'answer' (string), 'confidence' (number 0-1), 'keywords' (array of strings). No other text.")
```

**Code Reviewer:**
```python
SystemMessage(content="You are an expert code reviewer. When shown code, identify bugs, security issues, and performance problems. Rate severity as HIGH/MEDIUM/LOW. Suggest fixes with code examples.")
```

### Best Practices for System Prompts

1. **Be specific** — "You are a helpful assistant" is too vague. Define the exact role and constraints.
2. **Set the format** — If you want bullet points, JSON, or markdown, say so explicitly.
3. **Define boundaries** — Tell the model what it should NOT do (e.g., "Do not make up information").
4. **Keep it concise** — Every token in the system prompt costs money and counts against context limits.
5. **Test and iterate** — Try different system prompts and compare the responses.

---

## 11. Temperature and Other Parameters

### The Key Parameters

| Parameter | Range | Default | Effect |
|-----------|-------|---------|--------|
| `temperature` | 0.0–2.0 | 1.0 | Controls randomness/creativity |
| `max_tokens` | 1–4096+ | Varies | Maximum response length |
| `top_p` | 0.0–1.0 | 1.0 | Nucleus sampling threshold |
| `frequency_penalty` | -2.0–2.0 | 0.0 | Reduces word repetition |
| `presence_penalty` | -2.0–2.0 | 0.0 | Encourages topic diversity |
| `stop` | string/array | None | Custom stop sequences |

### Temperature Deep Dive

Temperature controls the randomness of the model's output:

| Temperature | Behavior | Use Case |
|-------------|----------|----------|
| **0.0** | Nearly deterministic — same input gives same output | Factual Q&A, code generation, data extraction |
| **0.3** | Low creativity — mostly predictable | Customer support, documentation |
| **0.7** | Balanced — good default for most tasks | General conversation, creative writing |
| **1.0** | High creativity — more varied responses | Brainstorming, storytelling |
| **1.5–2.0** | Very random — often incoherent | Not recommended for most use cases |

**Experiment code:**
```python
"""Temperature Comparison"""
import os
from dotenv import load_dotenv
from azure.ai.inference import ChatCompletionsClient
from azure.ai.inference.models import SystemMessage, UserMessage
from azure.core.credentials import AzureKeyCredential

load_dotenv()

client = ChatCompletionsClient(
    endpoint=os.getenv("AZURE_AI_ENDPOINT"),
    credential=AzureKeyCredential(os.getenv("AZURE_OPENAI_API_KEY"))
)

prompt = "Describe the color blue in one sentence."

for temp in [0.0, 0.3, 0.7, 1.0, 1.5]:
    print(f"\n🌡️ Temperature: {temp}")
    print("-" * 40)
    for run in range(3):
        response = client.complete(
            model="gpt-4o-mini",
            messages=[
                SystemMessage(content="You are a creative writer."),
                UserMessage(content=prompt)
            ],
            temperature=temp,
            max_tokens=50
        )
        print(f"  Run {run+1}: {response.choices[0].message.content.strip()}")
```

### top_p vs. Temperature

Both control randomness, but differently:
- **Temperature** scales the probability distribution (makes unlikely tokens more/less likely)
- **top_p** cuts off the distribution (only considers the top P% of probable tokens)

> ⚠️ **Warning:** OpenAI recommends adjusting either `temperature` OR `top_p`, but not both simultaneously. For most use cases, just use `temperature`.

---

## 12. Building a Simple Chat Application

Now let's put it all together — a complete, interactive chatbot that maintains conversation history.

### The Complete Chatbot

Create a file named `chatbot.py`:

```python
"""
Simple Interactive Chatbot — Azure AI Foundry
Week 5: Your first chatbot application

Features:
- Multi-turn conversation with memory
- Token usage tracking
- Special commands (quit, clear, tokens, help)
- Error handling
"""
import os
import sys
from dotenv import load_dotenv
from azure.ai.inference import ChatCompletionsClient
from azure.ai.inference.models import SystemMessage, UserMessage, AssistantMessage
from azure.core.credentials import AzureKeyCredential

# Load configuration
load_dotenv()

# Configuration
MODEL = os.getenv("AZURE_OPENAI_DEPLOYMENT_NAME", "gpt-4o-mini")
MAX_TOKENS = 500
TEMPERATURE = 0.7
SYSTEM_PROMPT = """You are a friendly and helpful AI assistant created for 
the Azure AI Foundry training course. You help users learn about AI, Azure, 
and cloud computing. Keep responses concise but informative. If you don't know 
something, say so honestly."""

# Initialize client
client = ChatCompletionsClient(
    endpoint=os.getenv("AZURE_AI_ENDPOINT"),
    credential=AzureKeyCredential(os.getenv("AZURE_OPENAI_API_KEY"))
)

# Conversation history
conversation = [SystemMessage(content=SYSTEM_PROMPT)]
total_tokens_used = 0


def send_message(user_input):
    """Send a message and get a response from the model."""
    global total_tokens_used

    conversation.append(UserMessage(content=user_input))

    try:
        response = client.complete(
            model=MODEL,
            messages=conversation,
            temperature=TEMPERATURE,
            max_tokens=MAX_TOKENS
        )

        assistant_reply = response.choices[0].message.content
        conversation.append(AssistantMessage(content=assistant_reply))

        tokens_this_turn = response.usage.total_tokens
        total_tokens_used += tokens_this_turn

        return assistant_reply, tokens_this_turn

    except Exception as e:
        conversation.pop()  # Remove failed user message
        return f"❌ Error: {str(e)}", 0


def main():
    """Main chatbot loop."""
    global total_tokens_used

    print()
    print("╔══════════════════════════════════════════════════════╗")
    print("║  🤖 AI Foundry Chatbot — Week 5                      ║")
    print("║  Commands: quit | clear | tokens | help               ║")
    print("╚══════════════════════════════════════════════════════╝")
    print()

    while True:
        try:
            user_input = input("You: ").strip()
        except (KeyboardInterrupt, EOFError):
            print("\n\n👋 Goodbye!")
            break

        if not user_input:
            continue

        # Handle special commands
        cmd = user_input.lower()
        if cmd in ("quit", "exit", "q"):
            print(f"\n👋 Goodbye! Total tokens used: {total_tokens_used}")
            break
        elif cmd == "clear":
            conversation.clear()
            conversation.append(SystemMessage(content=SYSTEM_PROMPT))
            total_tokens_used = 0
            print("🧹 Conversation cleared!\n")
            continue
        elif cmd == "tokens":
            print(f"📊 Total tokens: {total_tokens_used}")
            print(f"📝 Messages in history: {len(conversation)}\n")
            continue
        elif cmd == "help":
            print("📋 Commands:")
            print("  quit   — End the conversation")
            print("  clear  — Reset conversation history")
            print("  tokens — Show token usage")
            print("  help   — Show this message\n")
            continue

        # Send message and display response
        reply, tokens = send_message(user_input)
        print(f"\n🤖 Assistant: {reply}")
        print(f"   [{tokens} tokens | {total_tokens_used} total]\n")


if __name__ == "__main__":
    main()
```

**Run the chatbot:**
```bash
python chatbot.py
```

**Example conversation:**
```
╔══════════════════════════════════════════════════════╗
║  🤖 AI Foundry Chatbot — Week 5                      ║
║  Commands: quit | clear | tokens | help               ║
╚══════════════════════════════════════════════════════╝

You: What is Azure AI Foundry?

🤖 Assistant: Azure AI Foundry is Microsoft's unified platform for building,
deploying, and managing AI applications. It provides access to 1,800+ models,
tools for prompt engineering, built-in evaluation, and enterprise security.
   [85 tokens | 85 total]

You: How is it different from Azure OpenAI?

🤖 Assistant: Azure OpenAI is one service within the Azure AI ecosystem that
specifically provides access to OpenAI models (GPT-4, DALL-E, etc.). Azure AI
Foundry is broader — it includes Azure OpenAI plus models from Meta, Mistral,
and others, along with orchestration, evaluation, and deployment tools.
   [195 tokens | 280 total]

You: tokens
📊 Total tokens: 280
📝 Messages in history: 5

You: quit
👋 Goodbye! Total tokens used: 280
```

> 🔑 **Key Concept:** Notice how the token count increases with each turn. That's because the entire conversation history is sent with every request. In production chatbots, you'll need strategies to manage this (summarization, sliding windows, etc.).

---

## 13. Cost Management

### Understanding the Cost Model

Azure AI Foundry charges based on **token consumption**:
- **Input tokens:** Your prompts (system + user + conversation history)
- **Output tokens:** The model's responses
- Output tokens are typically 3–4x more expensive than input tokens

### What Will This Course Cost?

| Activity | Estimated Tokens | Estimated Cost (GPT-4o-mini) |
|----------|-----------------|------------------------------|
| This week's labs | ~10,000 | ~$0.005 |
| A full week of development | ~100,000 | ~$0.05 |
| Building a chatbot prototype | ~1,000,000 | ~$0.50 |
| All 45 weeks (estimate) | ~5,000,000 | ~$2.50 |

### Cost Optimization Strategies

1. **Use GPT-4o-mini for development** — Only switch to GPT-4o for production tasks.
2. **Set appropriate max_tokens** — Don't default to 4096 if you only need 200.
3. **Use temperature 0 for testing** — Deterministic responses let you cache results.
4. **Monitor token usage** — Check `response.usage` after every call.
5. **Trim conversation history** — Don't send 100 turns of history when 10 will do.
6. **Use the Azure Cost Management dashboard** — Set up budget alerts.

### Setting Budget Alerts

```bash
# View your current spending
az consumption usage list \
  --start-date 2025-07-01 \
  --end-date 2025-07-31 \
  --query "[?contains(instanceName, 'openai')]" \
  --output table
```

In the Azure portal:
1. Go to **Cost Management + Billing**
2. Click **Budgets** → **+ Add**
3. Set a monthly budget (e.g., $10 for learning)
4. Configure email alerts at 50%, 80%, and 100%

---

## 14. Troubleshooting Common Issues

### Error Reference Table

| Error | HTTP Code | Cause | Solution |
|-------|-----------|-------|----------|
| `Unauthorized` | 401 | Invalid API key | Check `.env` file, regenerate key in portal |
| `Not Found` | 404 | Wrong endpoint URL | Verify endpoint in deployment details |
| `Too Many Requests` | 429 | Rate limit exceeded | Wait and retry, or request quota increase |
| `Connection Timeout` | — | Network/firewall | Check proxy settings, try `curl` test |
| `ModuleNotFoundError` | — | Package not installed | `pip install azure-ai-inference` |
| `DeploymentNotFound` | 404 | Model not deployed | Deploy model in AI Foundry portal |
| `ContentFilterError` | 400 | Content policy violation | Modify prompt to avoid policy triggers |
| `InvalidAPIVersion` | 400 | Wrong API version | Use `2024-12-01-preview` |
| `ResourceGroupNotFound` | 404 | Wrong resource group | Verify with `az group list --output table` |
| `AuthenticationError` | 401 | Azure CLI not logged in | Run `az login` |

### Quick Diagnostic Script

```python
"""Quick diagnostic — run this if things aren't working"""
import os
from dotenv import load_dotenv

load_dotenv()

print("=== Environment Check ===")

checks = {
    "AZURE_AI_ENDPOINT": os.getenv("AZURE_AI_ENDPOINT"),
    "AZURE_OPENAI_API_KEY": os.getenv("AZURE_OPENAI_API_KEY"),
    "AZURE_OPENAI_DEPLOYMENT_NAME": os.getenv("AZURE_OPENAI_DEPLOYMENT_NAME"),
}

all_good = True
for key, value in checks.items():
    if value:
        masked = value[:8] + "..." if len(value) > 12 else value
        print(f"  ✅ {key}: {masked}")
    else:
        print(f"  ❌ {key}: NOT SET")
        all_good = False

if all_good:
    print("\n🔍 Testing connection...")
    try:
        from azure.ai.inference import ChatCompletionsClient
        from azure.ai.inference.models import UserMessage
        from azure.core.credentials import AzureKeyCredential

        client = ChatCompletionsClient(
            endpoint=os.getenv("AZURE_AI_ENDPOINT"),
            credential=AzureKeyCredential(os.getenv("AZURE_OPENAI_API_KEY"))
        )
        response = client.complete(
            model=os.getenv("AZURE_OPENAI_DEPLOYMENT_NAME"),
            messages=[UserMessage(content="Say OK")],
            max_tokens=5
        )
        print(f"  ✅ Connection successful! Response: {response.choices[0].message.content}")
    except Exception as e:
        print(f"  ❌ Connection failed: {e}")
else:
    print("\n⚠️ Fix missing environment variables in your .env file first.")
```

---

## 15. Best Practices for Project Organization

### Recommended Folder Structure

```
my-ai-project/
├── .env                    # Environment variables (NEVER commit!)
├── .gitignore              # Git ignore rules
├── requirements.txt        # Python dependencies
├── README.md               # Project documentation
├── src/
│   ├── __init__.py
│   ├── hello_world.py      # First test script
│   ├── chatbot.py          # Interactive chatbot
│   ├── config.py           # Centralized configuration
│   └── utils.py            # Helper functions
├── tests/
│   ├── __init__.py
│   └── test_connection.py  # Connectivity tests
├── notebooks/
│   └── experiments.ipynb   # Jupyter experiments
└── docs/
    └── setup.md            # Setup instructions
```

### Configuration Helper (config.py)

```python
"""Centralized configuration management"""
import os
from dotenv import load_dotenv

load_dotenv()

class Config:
    ENDPOINT = os.getenv("AZURE_AI_ENDPOINT")
    API_KEY = os.getenv("AZURE_OPENAI_API_KEY")
    MODEL = os.getenv("AZURE_OPENAI_DEPLOYMENT_NAME", "gpt-4o-mini")
    API_VERSION = os.getenv("AZURE_OPENAI_API_VERSION", "2024-12-01-preview")
    MAX_TOKENS = int(os.getenv("MAX_TOKENS", "500"))
    TEMPERATURE = float(os.getenv("TEMPERATURE", "0.7"))

    @classmethod
    def validate(cls):
        required = ["ENDPOINT", "API_KEY", "MODEL"]
        missing = [f for f in required if not getattr(cls, f)]
        if missing:
            raise ValueError(f"Missing config: {', '.join(missing)}")
```

### Best Practices Summary

1. **Keep secrets out of code** — Always use `.env` files and Key Vault.
2. **Use virtual environments** — Isolate your project dependencies.
3. **Version control everything** — Except secrets and virtual environments.
4. **Centralize configuration** — One place to manage settings.
5. **Test your connection** — Run diagnostics before building features.
6. **Track token usage** — Monitor costs from day one.
7. **Document your setup** — Future-you will thank present-you.

---

## 16. Summary and What's Next

### What You Accomplished This Week 🎉

1. ✅ Installed Python, VS Code, Azure CLI, and Git
2. ✅ Configured Azure CLI authentication
3. ✅ Created an AI Foundry Hub and Project
4. ✅ Deployed your first model (GPT-4o-mini)
5. ✅ Sent your first prompt four ways (Playground, SDK, REST, CLI)
6. ✅ Understood API responses: tokens, usage, finish_reason
7. ✅ Experimented with system prompts and temperature
8. ✅ Built an interactive chatbot with conversation history
9. ✅ Learned cost management strategies
10. ✅ Set up a proper project structure

### Key Takeaways

- **AI Foundry provides multiple interaction methods** — Playground for testing, SDK for applications, REST for integration, CLI for automation.
- **Tokens are currency** — Every interaction costs tokens. Monitor and optimize.
- **System prompts are powerful** — They dramatically change model behavior without changing the model.
- **Temperature controls creativity** — Low for facts, medium for conversation, high for brainstorming.
- **Conversation history matters** — It provides context but grows token costs.

### Looking Ahead: Week 6 — Prompt Engineering Deep Dive

Next week we'll master the art and science of writing effective prompts:
- Prompt anatomy and structure
- Zero-shot, few-shot, and chain-of-thought techniques
- Prompt templates and variables
- Advanced system prompts
- Output formatting and structured responses
- Common prompt patterns and anti-patterns

> 💡 **Tip:** Practice using your chatbot this week! The more you interact with the model, the better you'll understand how prompts affect responses — which is exactly what Week 6 is all about.

---

## 17. Additional Resources

### Official Documentation
- [Azure AI Foundry Documentation](https://learn.microsoft.com/azure/ai-studio/)
- [Azure AI Inference SDK Reference](https://learn.microsoft.com/python/api/azure-ai-inference/)
- [Azure OpenAI REST API Reference](https://learn.microsoft.com/azure/ai-services/openai/reference)
- [Azure CLI Reference](https://learn.microsoft.com/cli/azure/)

### SDKs and Tools
- [azure-ai-inference on PyPI](https://pypi.org/project/azure-ai-inference/)
- [azure-ai-projects on PyPI](https://pypi.org/project/azure-ai-projects/)
- [VS Code Azure Extensions](https://marketplace.visualstudio.com/search?term=azure&target=VSCode)

### Learning Resources
- [Azure AI Foundry Quickstart](https://learn.microsoft.com/azure/ai-studio/quickstarts/get-started-playground)
- [Azure Free Account](https://azure.microsoft.com/free/)
- [Azure Pricing Calculator](https://azure.microsoft.com/pricing/calculator/)

### Community
- [Microsoft Q&A — Azure AI](https://learn.microsoft.com/answers/tags/azure-ai-services)
- [Stack Overflow — azure-ai tag](https://stackoverflow.com/questions/tagged/azure-ai)
- [Azure AI GitHub Samples](https://github.com/Azure-Samples?q=ai&type=all)

---

*© 2025 Azure AI Foundry Training Course. Week 5 of 45 — Zero to Hero.*
*Next week: Week 6 — Prompt Engineering Deep Dive →*