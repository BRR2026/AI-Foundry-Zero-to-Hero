# Lab 5: Setting Up Your Environment + First "Hello World" Model

| | |
|---|---|
| **Estimated Time** | 90–120 minutes |
| **Difficulty** | ⭐⭐ Beginner–Intermediate |
| **Module** | 5 of 45 |
| **Lab Type** | 🔧 Hands-On Setup + Coding |

## Prerequisites

| Requirement | Details |
|---|---|
| Azure subscription | Free tier or Pay-As-You-Go — [azure.microsoft.com/free](https://azure.microsoft.com/free) |
| Operating system | Windows 10/11, macOS 12+, or Ubuntu 20.04+ |
| Internet connection | Required for Azure services and package downloads |
| Admin/sudo rights | Needed for installing software (Python, Azure CLI, etc.) |
| Browser | Microsoft Edge or Google Chrome (latest version recommended) |
| Modules 1–4 completed | Familiarity with AI Foundry portal concepts from prior labs |

> **💡 Note:** This is the most hands-on lab so far! You will be installing real development tools, creating live Azure resources, writing Python code, and making API calls. Set aside dedicated, uninterrupted time — and keep your Azure credentials handy. By the end of this lab, you will have a fully working local development environment and your first AI-powered application.

---

## Learning Objectives

By the end of this lab you will be able to:

- ✅ Install and configure Python, VS Code, Azure CLI, and the required Azure AI SDKs
- ✅ Create an AI Foundry Hub and Project using both the Azure portal and the CLI
- ✅ Deploy a GPT-4o-mini model in your AI Foundry project
- ✅ Send your first AI prompt via the Playground, Python SDK, REST API, and CLI
- ✅ Experiment with system prompts and temperature settings to understand model behavior
- ✅ Build a simple interactive chatbot in Python with conversation history and token tracking

---

## Exercise 1: Set Up Your Development Environment (25 min)

> **🎯 Goal:** Install all the tools you need for the rest of this 45-Module course. This is a one-time setup that pays dividends for every future lab.

### Step 1 — Install Python 3.11+

Python is the primary language for Azure AI development. We need version 3.11 or higher.

**Windows:**

1. Open your browser and navigate to [python.org/downloads](https://www.python.org/downloads/)
2. Download the latest Python 3.11+ installer for Windows
3. Run the installer — **IMPORTANT:** Check the box that says **"Add python.exe to PATH"** before clicking Install
4. Choose **"Install Now"** (default settings are fine)
5. When installation completes, click **"Disable path length limit"** if prompted

**macOS:**

```bash
# Option 1: Using Homebrew (recommended)
brew install python@3.11

# Option 2: Download from python.org/downloads
# Run the .pkg installer and follow the prompts
```

**Linux (Ubuntu/Debian):**

```bash
sudo apt update
sudo apt install python3.11 python3.11-venv python3-pip -y
```

**Verify your installation** (all platforms):

```bash
python --version
# Expected output: Python 3.11.x (or higher, e.g., 3.12.x)

pip --version
# Expected output: pip 23.x.x from ... (python 3.11)
```

> **⚠️ Windows users:** If `python` is not recognized, try `python3` or `py` instead. If none work, the PATH was not set during installation — reinstall and check the PATH checkbox.

> **⚠️ macOS/Linux users:** You may need to use `python3` and `pip3` instead of `python` and `pip`. To simplify, you can create an alias: `alias python=python3`

### Step 2 — Install Visual Studio Code

VS Code is our recommended code editor for the entire course.

1. Navigate to [code.visualstudio.com](https://code.visualstudio.com/)
2. Download the installer for your operating system
3. Run the installer with default settings
4. Launch VS Code after installation

**Install recommended extensions** — open the Extensions panel (`Ctrl+Shift+X` / `Cmd+Shift+X`) and install each of these:

| Extension | ID | Purpose |
|---|---|---|
| Python | `ms-python.python` | Python language support, debugging, linting |
| Azure Account | `ms-vscode.azure-account` | Sign in to Azure from VS Code |
| Azure Resources | `ms-azuretools.vscode-azureresourcegroups` | Browse and manage Azure resources |
| Jupyter | `ms-toolsai.jupyter` | Run Jupyter notebooks in VS Code |
| Azure AI Inference | `ms-dotnettools.azure-ai-inference` | AI Foundry integration tools |

> **💡 Tip:** You can install extensions from the terminal too:
> ```bash
> code --install-extension ms-python.python
> code --install-extension ms-vscode.azure-account
> code --install-extension ms-azuretools.vscode-azureresourcegroups
> code --install-extension ms-toolsai.jupyter
> ```

### Step 3 — Install Azure CLI

The Azure CLI lets you manage Azure resources from your terminal.

**Windows (using winget — recommended):**

```bash
winget install -e --id Microsoft.AzureCLI
```

**Windows (alternative — MSI installer):**

Download from [aka.ms/installazurecliwindows](https://aka.ms/installazurecliwindows) and run the installer.

**macOS:**

```bash
brew install azure-cli
```

**Linux (Ubuntu/Debian):**

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

**Verify your installation:**

```bash
az --version
# Expected output (first line): azure-cli 2.x.x
```

> **⚠️ Important:** After installing Azure CLI on Windows, you may need to close and reopen your terminal for the `az` command to be recognized.

### Step 4 — Install Git

Git is essential for version control and collaboration.

**Windows:**

Download from [git-scm.com](https://git-scm.com/download/win) and run the installer. Use default settings.

**macOS:**

```bash
# Git comes pre-installed on macOS. If not:
xcode-select --install
```

**Linux:**

```bash
sudo apt install git -y
```

**Verify:**

```bash
git --version
# Expected output: git version 2.x.x
```

### Step 5 — Create Project Directory and Python Virtual Environment

A virtual environment keeps your project's Python packages isolated from other projects.

```bash
# Create and navigate to your project directory
mkdir ai-foundry-hello-world
cd ai-foundry-hello-world

# Create a virtual environment
python -m venv .venv
```

**Activate the virtual environment:**

```bash
# Windows (Command Prompt)
.venv\Scripts\activate

# Windows (PowerShell)
.venv\Scripts\Activate.ps1

# macOS / Linux
source .venv/bin/activate
```

You should see `(.venv)` appear at the beginning of your terminal prompt. This confirms the virtual environment is active.

**Install the required Azure AI packages:**

```bash
pip install azure-ai-projects azure-ai-inference azure-identity python-dotenv
```

**Verify the packages installed correctly:**

```bash
pip list | grep azure
```

Expected output (version numbers may vary):

```
azure-ai-inference        1.0.0b9
azure-ai-projects         1.0.0b7
azure-core                1.32.0
azure-identity            1.19.0
```

> **💡 Tip:** If `grep` is not available on Windows, use: `pip list | findstr azure`

### Step 6 — Create Project Files

Create the following three files in your `ai-foundry-hello-world` directory:

**File 1: `.env`** — Environment variables (you will fill in the real values in Exercise 3)

```
# Azure AI Foundry Configuration
# Fill in these values after deploying your model in Exercise 3

AZURE_AI_ENDPOINT=https://your-endpoint.openai.azure.com/
AZURE_OPENAI_API_KEY=your-key-here
AZURE_OPENAI_DEPLOYMENT_NAME=gpt-4o-mini
AZURE_OPENAI_API_VERSION=2024-12-01-preview
```

**File 2: `.gitignore`** — Prevent secrets and unnecessary files from being committed

```
# Environment variables (NEVER commit this!)
.env

# Python virtual environment
.venv/

# Python cache
__pycache__/
*.pyc
*.pyo

# IDE settings
.vscode/
.idea/

# OS files
.DS_Store
Thumbs.db
```

**File 3: `requirements.txt`** — Pin your dependencies for reproducibility

```
azure-ai-projects>=1.0.0b7
azure-ai-inference>=1.0.0b9
azure-identity>=1.19.0
python-dotenv>=1.0.0
```

### ✅ Checkpoint

Before moving on, verify everything is in place:

- [ ] Python 3.11+ installed and `python --version` returns 3.11 or higher
- [ ] VS Code installed with Python, Azure Account, Azure Resources, and Jupyter extensions
- [ ] Azure CLI installed and `az --version` returns a valid version
- [ ] Git installed and `git --version` returns a valid version
- [ ] Virtual environment created and activated (you see `(.venv)` in your prompt)
- [ ] Azure AI packages installed (`pip list` shows `azure-ai-inference` and `azure-ai-projects`)
- [ ] `.env`, `.gitignore`, and `requirements.txt` files created

> **🎉 Congratulations!** Your development environment is ready. This setup will serve you for the remaining 40 modules of the course.

---

## Exercise 2: Create an AI Foundry Hub and Project (20 min)

> **🎯 Goal:** Create the Azure resources you need to deploy and use AI models. You will learn both the portal (GUI) and CLI methods — pick whichever you prefer, or try both!

### Part A — Portal Method (Recommended for First-Timers)

#### Step 1 — Sign in to AI Foundry

1. Open your browser and navigate to [ai.azure.com](https://ai.azure.com)
2. Click **Sign in** and enter your Azure credentials
3. If prompted, select your Azure subscription
4. You should land on the AI Foundry home page

#### Step 2 — Create a Hub

A Hub is the top-level container for your AI resources, including compute, storage, and security settings.

1. In the left navigation, click **Management** → **All hubs**
2. Click **+ New hub**
3. Fill in the following fields:

| Field | Value |
|---|---|
| Hub name | `hub-ai-foundry-dev` |
| Subscription | *(select your Azure subscription)* |
| Resource group | Click **Create new** → enter `rg-ai-foundry-dev` → click **OK** |
| Location | `East US 2` *(good balance of model availability and cost)* |

4. Leave all other settings at their defaults
5. Click **Next** through any remaining tabs, reviewing the defaults
6. Click **Create**
7. Wait for the deployment to complete (this typically takes 2–5 minutes)

> **💡 Why East US 2?** This region has broad availability for GPT-4o-mini and other models. If you are in Europe or Asia-Pacific, `West Europe` or `Japan East` are good alternatives — just make sure GPT-4o-mini is available in your chosen region.

#### Step 3 — Create a Project

A Project lives inside a Hub and is where you deploy models, run experiments, and build applications.

1. After your Hub is created, click **+ New project** (or navigate to **Management** → **All projects** → **+ New project**)
2. Fill in the following fields:

| Field | Value |
|---|---|
| Project name | `proj-hello-world` |
| Hub | Select `hub-ai-foundry-dev` (the Hub you just created) |

3. Click **Create**
4. Wait for the project to be ready (usually under 1 minute)
5. You should be redirected to your new project's overview page

> **📝 Take note:** You now have a Resource Group (`rg-ai-foundry-dev`) containing a Hub (`hub-ai-foundry-dev`) and a Project (`proj-hello-world`). This hierarchy — Resource Group → Hub → Project — is the standard organizational pattern in AI Foundry.

### Part B — CLI Method (For Power Users)

If you already created resources via the portal, you can skip this section — or create a second project to practice!

#### Step 1 — Authenticate with Azure

```bash
# Log in to Azure (a browser window will open)
az login

# Set your active subscription (replace with your subscription name or ID)
az account set --subscription "Your Subscription Name"

# Verify you're connected to the right subscription
az account show --output table
```

Expected output:

```
EnvironmentName    IsDefault    Name                    State    TenantId
-----------------  -----------  ----------------------  -------  ------------------------------------
AzureCloud         True         Your Subscription Name  Enabled  xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

#### Step 2 — Install the ML Extension (if needed)

```bash
# Check if the ML extension is installed
az extension list --output table | findstr ml

# If not listed, install it
az extension add --name ml --yes

# If already installed, update it
az extension update --name ml
```

#### Step 3 — Create a Resource Group

```bash
az group create \
  --name rg-ai-foundry-dev \
  --location eastus2
```

Expected output (truncated):

```json
{
  "id": "/subscriptions/.../resourceGroups/rg-ai-foundry-dev",
  "location": "eastus2",
  "name": "rg-ai-foundry-dev",
  "properties": {
    "provisioningState": "Succeeded"
  }
}
```

> **⚠️ Windows users:** If the `\` line continuation character does not work in PowerShell, put the entire command on one line or use the backtick (`` ` ``) character instead.

#### Step 4 — Create an AI Foundry Hub

```bash
az ml workspace create \
  --kind hub \
  --name hub-ai-foundry-dev \
  --resource-group rg-ai-foundry-dev \
  --location eastus2
```

This command takes 2–5 minutes to complete. You will see a JSON response when it finishes.

#### Step 5 — Create a Project Under the Hub

```bash
az ml workspace create \
  --kind project \
  --name proj-hello-world \
  --resource-group rg-ai-foundry-dev \
  --hub-name hub-ai-foundry-dev
```

#### Step 6 — Verify Everything Was Created

```bash
# List all workspaces in your resource group
az ml workspace list \
  --resource-group rg-ai-foundry-dev \
  --output table
```

Expected output:

```
Name                 Location    Kind     Resource Group
-------------------  ----------  -------  -------------------
hub-ai-foundry-dev   eastus2     Hub      rg-ai-foundry-dev
proj-hello-world     eastus2     Project  rg-ai-foundry-dev
```

You can also verify by visiting [ai.azure.com](https://ai.azure.com) — your Hub and Project should appear.

### ✅ Checkpoint

- [ ] AI Foundry Hub created (via portal OR CLI) — named `hub-ai-foundry-dev`
- [ ] Project created under the Hub — named `proj-hello-world`
- [ ] Can see both resources in the AI Foundry portal at [ai.azure.com](https://ai.azure.com)
- [ ] Resource group `rg-ai-foundry-dev` exists in your Azure subscription

> **⚠️ Troubleshooting:**
>
> | Issue | Solution |
> |---|---|
> | `az ml` command not found | Run `az extension add --name ml --yes` |
> | "Location not available" error | Try `eastus`, `westus2`, or `westeurope` instead |
> | "Subscription not registered" | Run `az provider register --namespace Microsoft.MachineLearningServices` |
> | Hub creation fails silently | Check the Azure portal → Activity Log for detailed error messages |
> | "Quota exceeded" error | Try a different region or request a quota increase in the portal |

---

## Exercise 3: Deploy GPT-4o-mini and Send "Hello World" (25 min)

> **🎯 Goal:** Deploy your first AI model and communicate with it four different ways — Playground, Python SDK, REST API, and PowerShell. By the end, you'll truly understand how the pieces fit together.

### Step 1 — Deploy the GPT-4o-mini Model

1. Navigate to [ai.azure.com](https://ai.azure.com)
2. Select your project **proj-hello-world**
3. In the left navigation, click **Models + endpoints** → **+ Deploy model** → **Deploy base model**
4. In the model catalog, search for **gpt-4o-mini**
5. Select **gpt-4o-mini** and click **Confirm**
6. Configure the deployment:

| Setting | Value |
|---|---|
| Deployment name | `gpt-4o-mini` |
| Deployment type | Standard |
| Model version | *(use latest available)* |
| Tokens per Minute Rate Limit | `10K` *(sufficient for this lab)* |

7. Click **Deploy**
8. Wait for the deployment status to show **Succeeded** (usually 1–3 minutes)

> **💡 Why GPT-4o-mini?** It is cost-effective, fast, and highly capable — perfect for learning and development. The "mini" model costs roughly 10x less than GPT-4o while delivering excellent results for most tasks.

### Step 2 — Get Your Endpoint and Key

1. After deployment succeeds, click on the **gpt-4o-mini** deployment name
2. In the deployment details page, locate:
   - **Target URI** (your endpoint URL) — it looks like `https://your-resource.openai.azure.com/`
   - **Key** — click **Show** to reveal the API key
3. Open your `.env` file and replace the placeholder values:

```
AZURE_AI_ENDPOINT=https://your-actual-endpoint.openai.azure.com/
AZURE_OPENAI_API_KEY=abc123your-actual-key-here
AZURE_OPENAI_DEPLOYMENT_NAME=gpt-4o-mini
AZURE_OPENAI_API_VERSION=2024-12-01-preview
```

> **🔒 Security Warning:** Your API key is a secret! Never commit it to Git, share it in screenshots, or post it online. The `.gitignore` file you created in Exercise 1 protects your `.env` file from being committed.

### Step 3 — Hello World via Playground

The Playground lets you test your model without writing any code.

1. In the AI Foundry portal, go to **Playgrounds** → **Chat**
2. In the **Deployment** dropdown, select your **gpt-4o-mini** deployment
3. In the **System message** box, enter:
   ```
   You are a helpful assistant.
   ```
4. In the **User message** box at the bottom, type:
   ```
   Say 'Hello, World!' and explain what Azure AI Foundry is in one sentence.
   ```
5. Click **Send** (or press `Enter`)

You should see a response like:

```
Hello, World! Azure AI Foundry is Microsoft's unified platform for building,
deploying, and managing AI applications using a variety of models and tools
in a secure, enterprise-ready environment.
```

> **📝 Record your response** — copy it to a notepad. You will compare it with the SDK and API responses to confirm they produce similar outputs.

### Step 4 — Hello World via Python SDK

Create a file called `hello_world.py` in your `ai-foundry-hello-world` directory:

```python
"""
Exercise 3, Step 4: Hello World via Python SDK
Azure AI Foundry — Module 5 Lab

This script sends a single prompt to your deployed GPT-4o-mini model
using the Azure AI Inference SDK and prints the response.
"""

import os
from dotenv import load_dotenv
from azure.ai.inference import ChatCompletionsClient
from azure.ai.inference.models import SystemMessage, UserMessage
from azure.core.credentials import AzureKeyCredential

# Load environment variables from .env
load_dotenv()

# Read configuration
endpoint = os.getenv("AZURE_AI_ENDPOINT")
api_key = os.getenv("AZURE_OPENAI_API_KEY")
deployment = os.getenv("AZURE_OPENAI_DEPLOYMENT_NAME", "gpt-4o-mini")

# Validate configuration
if not endpoint or not api_key:
    print("❌ Error: Missing AZURE_AI_ENDPOINT or AZURE_OPENAI_API_KEY in .env file")
    print("   Please complete Step 2 of Exercise 3 first.")
    exit(1)

# Initialize the client
client = ChatCompletionsClient(
    endpoint=endpoint,
    credential=AzureKeyCredential(api_key)
)

# Send Hello World prompt
print("📡 Sending request to Azure AI Foundry...")
print()

response = client.complete(
    model=deployment,
    messages=[
        SystemMessage(content="You are a helpful assistant."),
        UserMessage(
            content="Say 'Hello, World!' and explain what Azure AI Foundry is in one sentence."
        ),
    ],
    temperature=0.7,
    max_tokens=200,
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
print("--- Model Info ---")
print(f"  Model:             {response.model}")
print(f"  Deployment:        {deployment}")
print()
print("✅ Success! Your first AI Foundry API call worked!")
```

**Run it:**

```bash
python hello_world.py
```

**Expected output:**

```
📡 Sending request to Azure AI Foundry...

============================================================
🌍 HELLO WORLD — Azure AI Foundry + Python SDK
============================================================

Response: Hello, World! Azure AI Foundry is Microsoft's unified
platform for building, deploying, and managing AI applications
using a variety of models and tools in a secure, enterprise-ready
environment.

--- Token Usage ---
  Prompt tokens:     29
  Completion tokens: 38
  Total tokens:      67
  Finish reason:     stop

--- Model Info ---
  Model:             gpt-4o-mini
  Deployment:        gpt-4o-mini

✅ Success! Your first AI Foundry API call worked!
```

> **🎉 This is a milestone moment!** You just made your first programmatic call to an AI model deployed on Azure. Every AI application you build in this course — chatbots, RAG systems, agents, multimodal apps — will build on this same pattern: create a client, send messages, process the response.

### Step 5 — Hello World via REST API (curl)

Understanding the raw REST API helps you debug issues and work in any language.

**macOS / Linux / Git Bash on Windows:**

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
# First, load your environment variables
$envFile = Get-Content .env | Where-Object { $_ -match "=" -and $_ -notmatch "^#" }
foreach ($line in $envFile) {
    $key, $value = $line -split "=", 2
    Set-Item -Path "Env:$key" -Value $value
}

# Build the request
$headers = @{
    "Content-Type" = "application/json"
    "api-key"      = $env:AZURE_OPENAI_API_KEY
}

$body = @{
    messages = @(
        @{ role = "system"; content = "You are a helpful assistant." }
        @{ role = "user"; content = "Say Hello World and tell me what Azure AI Foundry is." }
    )
    temperature = 0.7
    max_tokens  = 200
} | ConvertTo-Json -Depth 3

$uri = "$($env:AZURE_AI_ENDPOINT)openai/deployments/gpt-4o-mini/chat/completions?api-version=2024-12-01-preview"

# Send the request
$response = Invoke-RestMethod -Uri $uri -Method POST -Headers $headers -Body $body

# Display the response
Write-Host "Response: $($response.choices[0].message.content)"
Write-Host "Tokens used: $($response.usage.total_tokens)"
```

**Expected output** (JSON, truncated):

```json
{
  "choices": [
    {
      "message": {
        "role": "assistant",
        "content": "Hello, World! Azure AI Foundry is ..."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 27,
    "completion_tokens": 42,
    "total_tokens": 69
  }
}
```

> **📝 Compare your three responses** — Playground, Python SDK, and REST API. The wording will differ (due to the `temperature` setting adding randomness), but the meaning should be consistent. All three methods hit the same model behind the scenes.

### ✅ Checkpoint

- [ ] GPT-4o-mini model deployed successfully in AI Foundry portal
- [ ] `.env` file updated with your real endpoint and key
- [ ] Playground response received and recorded
- [ ] `hello_world.py` ran successfully and printed a response with token counts
- [ ] REST API call (curl or PowerShell) returned a valid JSON response
- [ ] You understand that all three methods use the same underlying model and API

---

## Exercise 4: Experiment with System Prompts and Temperature (15 min)

> **🎯 Goal:** Understand how system prompts shape the model's personality and how temperature controls randomness. These are two of the most important "knobs" you will use throughout this course.

### Step 1 — Create `experiment_prompts.py`

This script sends the **same user question** to the model with **five different system prompts** to show how dramatically the system prompt changes the response:

```python
"""
Exercise 4, Step 1: Experimenting with System Prompts
Azure AI Foundry — Module 5 Lab

Demonstrates how different system prompts dramatically change
the model's response to the same user question.
"""

import os
from dotenv import load_dotenv
from azure.ai.inference import ChatCompletionsClient
from azure.ai.inference.models import SystemMessage, UserMessage
from azure.core.credentials import AzureKeyCredential

load_dotenv()

# Initialize client
client = ChatCompletionsClient(
    endpoint=os.getenv("AZURE_AI_ENDPOINT"),
    credential=AzureKeyCredential(os.getenv("AZURE_OPENAI_API_KEY")),
)

# The question stays the same every time
user_question = "What is cloud computing?"

# Five very different system prompts
system_prompts = [
    (
        "Professional",
        "You are a professional cloud architect with 20 years of experience. "
        "Provide clear, technical explanations suitable for an IT audience.",
    ),
    (
        "For Kids",
        "You are a friendly teacher explaining things to a 10-year-old. "
        "Use simple words, fun analogies, and maybe an emoji or two.",
    ),
    (
        "Pirate",
        "You are a pirate who knows about technology. Answer everything in "
        "pirate speak. Start every answer with 'Ahoy!' and use nautical metaphors. Arrr!",
    ),
    (
        "Haiku",
        "You respond ONLY in haiku format (three lines: 5 syllables, 7 syllables, "
        "5 syllables). No other text besides the haiku.",
    ),
    (
        "JSON",
        "You respond ONLY in valid JSON format. Your response must be a JSON object "
        'with exactly these keys: "answer" (string, 1-2 sentences), "confidence" '
        '(number 0-1), "keywords" (array of 3-5 strings).',
    ),
]

print("=" * 70)
print("🧪 EXPERIMENT: Same Question, Different System Prompts")
print(f'📝 Question: "{user_question}"')
print("=" * 70)

for name, system_prompt in system_prompts:
    response = client.complete(
        model=os.getenv("AZURE_OPENAI_DEPLOYMENT_NAME", "gpt-4o-mini"),
        messages=[
            SystemMessage(content=system_prompt),
            UserMessage(content=user_question),
        ],
        temperature=0.7,
        max_tokens=150,
    )

    answer = response.choices[0].message.content
    tokens = response.usage.total_tokens

    print(f"\n{'─' * 60}")
    print(f"🎭 System Prompt Style: {name}")
    print(f"{'─' * 60}")
    print(answer)
    print(f"  [Tokens used: {tokens}]")

print(f"\n{'=' * 70}")
print("🏁 Experiment complete! Notice how the same question")
print("   produced wildly different responses based on the system prompt.")
print("=" * 70)
```

**Run it:**

```bash
python experiment_prompts.py
```

**Expected behavior:** You should see five distinctly different responses to "What is cloud computing?" — a technical explanation, a kid-friendly analogy, pirate-themed fun, a three-line haiku, and a structured JSON object.

### Step 2 — Create `experiment_temperature.py`

This script demonstrates the effect of temperature by running the **same prompt multiple times** at different temperature values:

```python
"""
Exercise 4, Step 2: Temperature Experiment
Azure AI Foundry — Module 5 Lab

Demonstrates how the 'temperature' parameter controls randomness.
Lower temperature = more deterministic. Higher temperature = more creative.
"""

import os
from dotenv import load_dotenv
from azure.ai.inference import ChatCompletionsClient
from azure.ai.inference.models import SystemMessage, UserMessage
from azure.core.credentials import AzureKeyCredential

load_dotenv()

# Initialize client
client = ChatCompletionsClient(
    endpoint=os.getenv("AZURE_AI_ENDPOINT"),
    credential=AzureKeyCredential(os.getenv("AZURE_OPENAI_API_KEY")),
)

# Temperature values to test (from most deterministic to most random)
temperatures = [0.0, 0.3, 0.7, 1.0, 1.5]

# A creative prompt where temperature differences are easy to spot
prompt = "Write a one-sentence description of the color blue."

print("=" * 70)
print("🌡️  EXPERIMENT: Temperature Effects on Response Variability")
print(f'📝 Prompt: "{prompt}"')
print()
print("Each temperature is tested 3 times so you can observe variation.")
print("=" * 70)

for temp in temperatures:
    print(f"\n{'─' * 60}")
    print(f"🌡️  Temperature: {temp}")
    if temp == 0.0:
        print("   (Most deterministic — responses should be nearly identical)")
    elif temp <= 0.3:
        print("   (Low randomness — minor variations expected)")
    elif temp <= 0.7:
        print("   (Balanced — good for general use)")
    elif temp <= 1.0:
        print("   (High creativity — noticeable variation)")
    else:
        print("   (Maximum creativity — responses may be surprising)")
    print(f"{'─' * 60}")

    for run in range(1, 4):
        response = client.complete(
            model=os.getenv("AZURE_OPENAI_DEPLOYMENT_NAME", "gpt-4o-mini"),
            messages=[
                SystemMessage(content="You are a creative writer."),
                UserMessage(content=prompt),
            ],
            temperature=temp,
            max_tokens=60,
        )
        answer = response.choices[0].message.content.strip()
        print(f"  Run {run}: {answer}")

print(f"\n{'=' * 70}")
print("🏁 Experiment complete!")
print()
print("Key observations:")
print("  • Temperature 0.0 — All 3 runs should be identical or nearly so")
print("  • Temperature 0.7 — Some variation, but similar themes")
print("  • Temperature 1.5 — High variation, possibly unusual wording")
print()
print("Rule of thumb:")
print("  • Use 0.0–0.3 for factual/analytical tasks (coding, math, data)")
print("  • Use 0.5–0.8 for balanced tasks (chatbots, summaries)")
print("  • Use 0.8–1.2 for creative tasks (writing, brainstorming)")
print("=" * 70)
```

**Run it:**

```bash
python experiment_temperature.py
```

**Expected behavior:** At temperature 0.0, all three runs should produce identical (or nearly identical) responses. As temperature increases, you will see progressively more variation between runs.

### Step 3 — Run Both Scripts and Record Observations

Run both scripts and note what you observe. Pay attention to:

1. **System prompts experiment:** How dramatically did the persona change the style of the answer? Was the factual content still accurate across all personas?
2. **Temperature experiment:** At what temperature did you first notice differences between runs? Was temperature 1.5 still coherent?

### ✅ Checkpoint

- [ ] Ran `experiment_prompts.py` — observed five different response styles
- [ ] Ran `experiment_temperature.py` — noted variation patterns across temperatures
- [ ] Can explain in your own words how system prompts shape model behavior
- [ ] Can explain in your own words how temperature controls randomness vs. determinism

> **📝 Reflection Questions:**
>
> 1. Which system prompt produced the most **useful** response for a real-world application?
> 2. At what temperature value did you start seeing **significant** variation between runs?
> 3. When would you set temperature to exactly **0**? *(Think: code generation, classification, data extraction)*
> 4. When would you set temperature to **1.0 or higher**? *(Think: creative writing, brainstorming, game dialogue)*
> 5. What happened when you asked for JSON output — did the model always produce valid JSON?

---

## Exercise 5: Build a Simple Interactive Chatbot (20 min)

> **🎯 Goal:** Build a working command-line chatbot that maintains conversation history, tracks token usage, and supports special commands. This is your first real AI application!

### Step 1 — Create `chatbot.py`

Create a file called `chatbot.py` in your project directory with the following complete code:

```python
"""
Exercise 5: Interactive Chatbot
Azure AI Foundry — Module 5 Lab

A command-line chatbot that maintains conversation history across turns,
tracks token usage, and supports special commands.

Usage:
    python chatbot.py

Commands:
    quit / exit / q  — End the conversation
    clear            — Reset conversation history
    tokens           — Show token usage statistics
    help             — Show available commands
"""

import os
import sys
from dotenv import load_dotenv
from azure.ai.inference import ChatCompletionsClient
from azure.ai.inference.models import SystemMessage, UserMessage, AssistantMessage
from azure.core.credentials import AzureKeyCredential

load_dotenv()

# ── Configuration ──────────────────────────────────────────
MODEL = os.getenv("AZURE_OPENAI_DEPLOYMENT_NAME", "gpt-4o-mini")
MAX_TOKENS = 500
TEMPERATURE = 0.7

DEFAULT_SYSTEM_PROMPT = (
    "You are a friendly and helpful AI assistant created for the Azure AI Foundry "
    "training course. You help users learn about AI, Azure, and cloud computing. "
    "Keep responses concise but informative. If you don't know something, say so "
    "honestly. When explaining technical concepts, use analogies when possible."
)

# ── Initialize Client ─────────────────────────────────────
endpoint = os.getenv("AZURE_AI_ENDPOINT")
api_key = os.getenv("AZURE_OPENAI_API_KEY")

if not endpoint or not api_key:
    print("❌ Error: Missing AZURE_AI_ENDPOINT or AZURE_OPENAI_API_KEY in .env file")
    print("   Please complete Exercise 3, Step 2 first.")
    sys.exit(1)

client = ChatCompletionsClient(
    endpoint=endpoint,
    credential=AzureKeyCredential(api_key),
)

# ── Conversation State ─────────────────────────────────────
conversation = [SystemMessage(content=DEFAULT_SYSTEM_PROMPT)]
total_tokens_used = 0
turn_count = 0


def send_message(user_input):
    """Send a message to the model and return the response."""
    global total_tokens_used, turn_count

    # Add the user's message to conversation history
    conversation.append(UserMessage(content=user_input))

    try:
        response = client.complete(
            model=MODEL,
            messages=conversation,
            temperature=TEMPERATURE,
            max_tokens=MAX_TOKENS,
        )

        # Extract the assistant's reply
        assistant_reply = response.choices[0].message.content

        # Add the assistant's reply to conversation history
        conversation.append(AssistantMessage(content=assistant_reply))

        # Track token usage
        tokens_this_turn = response.usage.total_tokens
        total_tokens_used += tokens_this_turn
        turn_count += 1

        return assistant_reply, tokens_this_turn

    except Exception as e:
        # Remove the failed user message so history stays clean
        conversation.pop()
        return f"❌ Error: {str(e)}", 0


def handle_command(command):
    """Handle special chatbot commands. Returns True if a command was handled."""
    global total_tokens_used, turn_count, conversation

    cmd = command.lower().strip()

    if cmd in ("quit", "exit", "q"):
        print(f"\n👋 Goodbye! Session stats:")
        print(f"   Turns: {turn_count}")
        print(f"   Total tokens: {total_tokens_used}")
        print(f"   Messages in history: {len(conversation)}")
        sys.exit(0)

    if cmd == "clear":
        conversation.clear()
        conversation.append(SystemMessage(content=DEFAULT_SYSTEM_PROMPT))
        total_tokens_used = 0
        turn_count = 0
        print("🧹 Conversation cleared! Starting fresh.\n")
        return True

    if cmd == "tokens":
        print(f"📊 Token Usage Statistics:")
        print(f"   Total tokens used this session: {total_tokens_used}")
        print(f"   Conversation turns: {turn_count}")
        print(f"   Messages in history: {len(conversation)}")
        print(f"   (System: 1, User: {turn_count}, Assistant: {turn_count})")
        print()
        return True

    if cmd == "history":
        print(f"📜 Conversation History ({len(conversation)} messages):")
        for i, msg in enumerate(conversation):
            if isinstance(msg, SystemMessage):
                print(f"   [{i}] SYSTEM: {msg.content[:80]}...")
            elif isinstance(msg, UserMessage):
                print(f"   [{i}] USER: {msg.content[:80]}")
            elif isinstance(msg, AssistantMessage):
                print(f"   [{i}] ASSISTANT: {msg.content[:80]}...")
        print()
        return True

    if cmd == "help":
        print("📋 Available Commands:")
        print("  quit / exit / q  — End the conversation")
        print("  clear            — Reset conversation history and token counter")
        print("  tokens           — Show token usage statistics")
        print("  history          — Show conversation message history")
        print("  help             — Show this help message")
        print()
        return True

    return False


def main():
    """Main chatbot loop."""
    print()
    print("╔══════════════════════════════════════════════════════╗")
    print("║  🤖 AI Foundry Chatbot — Module 5 Lab                 ║")
    print("║                                                      ║")
    print("║  Type your message and press Enter to chat.          ║")
    print("║  Type 'help' for available commands.                 ║")
    print("╚══════════════════════════════════════════════════════╝")
    print()

    while True:
        try:
            user_input = input("You: ").strip()
        except (KeyboardInterrupt, EOFError):
            print("\n\n👋 Goodbye!")
            break

        # Skip empty input
        if not user_input:
            continue

        # Check for special commands
        if handle_command(user_input):
            continue

        # Send the message and display the response
        print("🤖 Thinking...", end="\r")
        reply, tokens = send_message(user_input)
        print(f"                ", end="\r")  # Clear "Thinking..." line
        print(f"\n🤖 Assistant: {reply}")
        print(f"   [{tokens} tokens this turn | {total_tokens_used} total]\n")


if __name__ == "__main__":
    main()
```

### Step 2 — Run the Chatbot

```bash
python chatbot.py
```

You should see the welcome banner. Now you can type messages and get responses!

### Step 3 — Test These Conversations

Work through this script to exercise all the chatbot features:

**Test 1: Basic conversation**
```
You: What is Azure AI Foundry?
```
Read the response. Note the token count.

**Test 2: Follow-up question (tests conversation history)**
```
You: How is it different from Azure OpenAI Service?
```
The model should reference the previous exchange, showing that conversation history is working.

**Test 3: Ask about tokens**
```
You: Explain what tokens are in the context of large language models.
```
Notice this response might use more tokens due to the technical detail.

**Test 4: Check your usage**
```
You: tokens
```
This should display the cumulative token count across all turns.

**Test 5: View message history**
```
You: history
```
You should see the system message, your three questions, and the three responses.

**Test 6: Clear and verify**
```
You: clear
You: What did we talk about earlier?
```
The model should NOT remember the previous conversation — it was cleared.

**Test 7: Exit gracefully**
```
You: quit
```
The chatbot should display final session stats and exit.

### Step 4 — Bonus Challenges (For Students Who Finish Early)

If you completed the chatbot ahead of schedule, try adding one or more of these features:

**Challenge A: Save Conversation to JSON**
Add a `save` command that writes the conversation history to a `conversation.json` file with timestamps.

**Challenge B: Change System Prompt**
Add a `/system <new prompt>` command that lets the user change the system prompt mid-conversation without clearing history.

**Challenge C: Streaming Output**
Modify `send_message` to use streaming so the response appears word-by-word instead of all at once. Look into `client.complete(..., stream=True)`.

**Challenge D: Error Retry**
Add automatic retry with exponential backoff when API calls fail due to rate limiting or transient errors.

### ✅ Checkpoint

- [ ] Chatbot starts without errors and displays the welcome banner
- [ ] Can send messages and receive AI responses
- [ ] Multi-turn conversations work (the model remembers context from earlier turns)
- [ ] `tokens` command displays cumulative usage
- [ ] `history` command shows all messages
- [ ] `clear` command resets the conversation (model forgets previous context)
- [ ] `quit` command exits gracefully with session statistics
- [ ] You understand why conversation history grows the token count with each turn

---

## 🏆 Lab Summary

### What You Accomplished Today

| Task | Status |
|---|---|
| Installed Python 3.11+, VS Code, Azure CLI, and Git | ✅ |
| Set up a virtual environment with Azure AI packages | ✅ |
| Created an AI Foundry Hub and Project (portal and/or CLI) | ✅ |
| Deployed the GPT-4o-mini model | ✅ |
| Sent "Hello World" via the Playground | ✅ |
| Sent "Hello World" via the Python SDK | ✅ |
| Sent "Hello World" via the REST API (curl/PowerShell) | ✅ |
| Experimented with 5 different system prompts | ✅ |
| Experimented with temperature from 0.0 to 1.5 | ✅ |
| Built a fully working interactive chatbot | ✅ |

### Files You Created

```
ai-foundry-hello-world/
├── .env                         # API endpoint, key, and deployment config
├── .gitignore                   # Keeps secrets and temp files out of Git
├── requirements.txt             # Python package dependencies
├── hello_world.py               # Exercise 3 — First API call
├── experiment_prompts.py        # Exercise 4 — System prompt experiment
├── experiment_temperature.py    # Exercise 4 — Temperature experiment
└── chatbot.py                   # Exercise 5 — Interactive chatbot
```

### Key Takeaways

1. **The development stack is straightforward:** Python + Azure AI SDK + a `.env` file is all you need to start building AI applications on Azure.
2. **Hub → Project → Deployment** is the organizational hierarchy in AI Foundry. A Hub contains shared resources; a Project is where you work; a Deployment is a specific model ready to accept requests.
3. **Three ways to call the same model:** Playground (no code), Python SDK (production code), and REST API (language-agnostic). They all hit the same endpoint under the hood.
4. **System prompts are powerful:** The same model can behave like a professional consultant, a children's teacher, or a pirate — just by changing the system message.
5. **Temperature controls randomness:** Low temperature (0.0–0.3) for deterministic tasks like coding and classification; higher temperature (0.7–1.0) for creative tasks like writing and brainstorming.
6. **Conversation history accumulates tokens:** Each turn sends the entire history to the model. This is why longer conversations cost more and why you may eventually need strategies to manage history length.
7. **API keys are secrets:** Always use `.env` files, never hardcode keys, and always add `.env` to `.gitignore`.

### Common Issues and Solutions

| Issue | Possible Cause | Solution |
|---|---|---|
| `ModuleNotFoundError: No module named 'azure'` | Virtual environment not activated or packages not installed | Run `.venv\Scripts\activate` then `pip install -r requirements.txt` |
| `AuthenticationError: 401 Unauthorized` | Wrong API key in `.env` | Double-check the key in the AI Foundry portal under deployment details |
| `ResourceNotFoundError: 404` | Wrong endpoint URL or deployment name | Verify the endpoint ends with `/` and the deployment name matches exactly |
| `RateLimitError: 429 Too Many Requests` | Exceeded your tokens-per-minute quota | Wait 60 seconds and retry, or increase the quota in deployment settings |
| `python` command not found | Python not added to PATH | Reinstall Python and check "Add to PATH" or use `python3` / `py` |
| `az ml` command not found | ML extension not installed | Run `az extension add --name ml --yes` |
| Chatbot forgets context after clearing | This is expected behavior | `clear` resets the conversation history — the model has no memory of prior turns |
| Very high token counts after many turns | Conversation history grows with each turn | Use `clear` to reset, or in production, implement a sliding window strategy |

---

## 🎯 Stretch Goals (Optional)

For students who want to go further before next module:

### Stretch 1: Azure Identity Authentication
Replace the API key authentication with Azure Identity (DefaultAzureCredential). This is the production-recommended approach:
```python
from azure.identity import DefaultAzureCredential

credential = DefaultAzureCredential()
client = ChatCompletionsClient(endpoint=endpoint, credential=credential)
```
You will need to assign the **Cognitive Services User** role to your Azure identity.

### Stretch 2: Deploy a Second Model
Deploy **GPT-4o** (the full model) alongside GPT-4o-mini. Modify `hello_world.py` to send the same prompt to both models and compare response quality, speed, and token cost.

### Stretch 3: Build a Simple Web Interface
Install Flask (`pip install flask`) and create a minimal web chat interface. Serve an HTML page with a text input and display the chatbot responses in the browser instead of the terminal.

### Stretch 4: Explore the Model Catalog
Browse the AI Foundry model catalog at [ai.azure.com](https://ai.azure.com). Find three non-OpenAI models (e.g., Mistral, Llama, Phi) and note their capabilities, pricing, and deployment options. You will use these in future labs.

### Stretch 5: Prompt Engineering Warm-Up
Create a `prompt_engineering.py` script that tests three different prompting techniques for the same task (summarizing a paragraph):
- **Zero-shot:** Just ask the model to summarize
- **One-shot:** Provide one example summary, then ask
- **Chain-of-thought:** Ask the model to think step by step before summarizing

Compare the quality and token usage of each approach.

---

## 📚 Resources

### Official Documentation
- [Azure AI Foundry Documentation](https://learn.microsoft.com/en-us/azure/ai-studio/) — Comprehensive platform documentation
- [Azure AI Inference SDK for Python](https://learn.microsoft.com/en-us/python/api/overview/azure/ai-inference-readme) — SDK reference
- [Azure OpenAI REST API Reference](https://learn.microsoft.com/en-us/azure/ai-services/openai/reference) — Complete API specification
- [Azure CLI Documentation](https://learn.microsoft.com/en-us/cli/azure/) — CLI command reference

### Quickstarts and Tutorials
- [Quickstart: Chat with Azure AI models using Python](https://learn.microsoft.com/en-us/azure/ai-studio/quickstarts/get-started-playground) — Official quickstart guide
- [Deploy models in Azure AI Foundry](https://learn.microsoft.com/en-us/azure/ai-studio/how-to/deploy-models) — Deployment walkthrough
- [Azure AI Foundry SDK Overview](https://learn.microsoft.com/en-us/azure/ai-studio/how-to/develop/sdk-overview) — SDK capabilities summary

### Concepts (For Deeper Understanding)
- [What are tokens?](https://learn.microsoft.com/en-us/azure/ai-services/openai/overview#tokens) — How tokenization works
- [Azure OpenAI models](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models) — Model capabilities and pricing
- [Prompt engineering techniques](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/prompt-engineering) — Best practices for prompts

### Tools
- [Python Downloads](https://www.python.org/downloads/) — Get the latest Python
- [Visual Studio Code](https://code.visualstudio.com/) — Download VS Code
- [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli) — Install the Azure CLI

---

## 🔜 next module Preview

**Module 6: Understanding Models, Tokens, and Pricing** — Now that you have a working environment, we will dive deep into how large language models work under the hood, how tokenization affects your costs, and how to choose the right model for your use case. Bring your `experiment_temperature.py` and `experiment_prompts.py` scripts — we will build on them!

---

> **💬 Need help?** If you get stuck at any point during this lab, try the troubleshooting table above first. If that does not resolve your issue, share the exact error message (screenshot or copy/paste) in the course discussion forum with the tag `#week5-lab`.
