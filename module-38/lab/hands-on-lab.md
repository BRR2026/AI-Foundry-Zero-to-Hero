# Module 38 — Hands-On Lab

## Mini Hack: Create a Solution Template for a Customer FAQ Bot

**Azure AI Foundry: Zero to Hero** | ARC 8: Advanced Patterns & Enterprise | April 2026

---

## Lab Overview

| Field | Details |
|-------|---------|
| **Duration** | 60 minutes |
| **Level** | Advanced |
| **Objective** | Build a repeatable, parameterized solution template that deploys a knowledge-grounded FAQ bot for any customer using Azure AI Foundry |
| **Prerequisites** | Azure subscription with Contributor access; Azure CLI installed; Bicep CLI installed; Python 3.10+; access to Azure AI Foundry |

### What You Will Build

```
┌───────────────────────────────────────────────────┐
│           FAQ Bot Solution Template               │
├───────────────────────────────────────────────────┤
│                                                   │
│  customer-config.yaml ──→ deploy.py               │
│                              │                    │
│               ┌──────────────┼──────────┐         │
│               ▼              ▼          ▼         │
│      ┌──────────────┐ ┌──────────┐ ┌─────────┐   │
│      │ AI Foundry   │ │ AI Search│ │ Storage │   │
│      │ Project +    │ │ Index    │ │ Account │   │
│      │ Agent        │ │          │ │         │   │
│      └──────────────┘ └──────────┘ └─────────┘   │
│               │              ▲                    │
│               └──────────────┘                    │
│          (agent queries index)                    │
└───────────────────────────────────────────────────┘
```

### Learning Outcomes

By completing this lab, you will:
1. Design a parameterized solution template for AI offerings
2. Implement customer-specific configuration files
3. Create deployment automation that provisions infrastructure and AI assets
4. Build smoke tests that validate tenant isolation
5. Deploy the same template for two fictional customers

---

## Part 1: Project Scaffolding (10 minutes)

### Step 1.1 — Create the Template Directory Structure

Open a terminal and create the project scaffold:

```bash
mkdir -p faq-bot-template/{infra,src/prompts,data/contoso,data/fabrikam,tests,docs}
cd faq-bot-template
```

### Step 1.2 — Create the Customer Configuration Schema

Create `customer-config.schema.yaml` to define what each customer must provide:

```yaml
# customer-config.schema.yaml
# Schema for customer-specific configuration
required_fields:
  tenant_id:
    type: string
    description: "Unique identifier for the customer (lowercase, no spaces)"
    example: "contoso"
  display_name:
    type: string
    description: "Customer-facing name of the FAQ bot"
    example: "Contoso Support Assistant"
  branding:
    primary_color:
      type: string
      description: "Hex color code for UI theming"
      example: "#0078D4"
    greeting_message:
      type: string
      description: "Initial message shown to end users"
      example: "Hi! I'm the Contoso assistant. How can I help?"
    logo_url:
      type: string
      description: "URL to customer's logo"
      optional: true
  knowledge:
    source_container:
      type: string
      description: "Blob container with customer's FAQ documents"
    file_formats:
      type: list
      description: "Supported document formats"
      default: ["pdf", "md", "txt", "docx"]
  model:
    deployment_name:
      type: string
      default: "gpt-4o"
    temperature:
      type: float
      default: 0.3
      min: 0.0
      max: 1.0
  guardrails:
    blocked_topics:
      type: list
      description: "Topics the bot should decline to answer"
      default: []
    max_response_length:
      type: int
      default: 500
```

### Step 1.3 — Create Contoso's Configuration

Create `configs/contoso.yaml`:

```yaml
# configs/contoso.yaml
tenant_id: "contoso"
display_name: "Contoso Support Assistant"
branding:
  primary_color: "#0078D4"
  greeting_message: "Hello! I'm the Contoso virtual assistant. Ask me anything about our products and services."
  logo_url: "https://contoso.com/assets/logo.png"
knowledge:
  source_container: "contoso-faq-docs"
  file_formats: ["pdf", "md", "txt"]
model:
  deployment_name: "gpt-4o"
  temperature: 0.3
guardrails:
  blocked_topics: ["competitor-pricing", "internal-roadmap"]
  max_response_length: 500
```

### Step 1.4 — Create Fabrikam's Configuration

Create `configs/fabrikam.yaml`:

```yaml
# configs/fabrikam.yaml
tenant_id: "fabrikam"
display_name: "Fabrikam Help Desk"
branding:
  primary_color: "#107C10"
  greeting_message: "Welcome to Fabrikam Help Desk! What can I assist you with today?"
  logo_url: "https://fabrikam.com/brand/logo.png"
knowledge:
  source_container: "fabrikam-faq-docs"
  file_formats: ["pdf", "docx"]
model:
  deployment_name: "gpt-4o-mini"
  temperature: 0.2
guardrails:
  blocked_topics: ["legal-advice", "medical-advice"]
  max_response_length: 400
```

---

## Part 2: Infrastructure as Code (15 minutes)

### Step 2.1 — Create the Bicep Template

Create `infra/main.bicep`:

```bicep
// infra/main.bicep
// Parameterized infrastructure for the FAQ Bot Solution Template

@description('Customer tenant identifier')
param tenantId string

@description('Azure region for all resources')
param location string = resourceGroup().location

@description('AI model deployment name')
@allowed(['gpt-4o', 'gpt-4o-mini', 'gpt-4.1'])
param modelDeployment string = 'gpt-4o'

@description('Tokens-per-minute rate limit')
param tpmQuota int = 30000

@description('AI Search SKU')
@allowed(['basic', 'standard', 'standard2'])
param searchSku string = 'basic'

// --- Storage Account for Knowledge Base ---
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-05-01' = {
  name: '${tenantId}faqstorage'
  location: location
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
  properties: {
    supportsHttpsTrafficOnly: true
    minimumTlsVersion: 'TLS1_2'
  }
}

resource blobService 'Microsoft.Storage/storageAccounts/blobServices@2023-05-01' = {
  parent: storageAccount
  name: 'default'
}

resource knowledgeContainer 'Microsoft.Storage/storageAccounts/blobServices/containers@2023-05-01' = {
  parent: blobService
  name: '${tenantId}-faq-docs'
  properties: {
    publicAccess: 'None'
  }
}

// --- Azure AI Search ---
resource searchService 'Microsoft.Search/searchServices@2024-03-01-preview' = {
  name: '${tenantId}-faq-search'
  location: location
  sku: { name: searchSku }
  properties: {
    replicaCount: 1
    partitionCount: 1
    hostingMode: 'default'
  }
}

// --- Azure AI Foundry Hub ---
resource aiHub 'Microsoft.MachineLearningServices/workspaces@2024-10-01' = {
  name: '${tenantId}-ai-hub'
  location: location
  kind: 'Hub'
  identity: { type: 'SystemAssigned' }
  properties: {
    friendlyName: '${tenantId} FAQ Bot Hub'
    description: 'AI Foundry Hub for ${tenantId} FAQ Bot'
  }
}

// --- Azure AI Foundry Project ---
resource aiProject 'Microsoft.MachineLearningServices/workspaces@2024-10-01' = {
  name: '${tenantId}-faq-project'
  location: location
  kind: 'Project'
  identity: { type: 'SystemAssigned' }
  properties: {
    friendlyName: '${tenantId} FAQ Bot Project'
    hubResourceId: aiHub.id
  }
}

// --- Outputs ---
output storageAccountName string = storageAccount.name
output searchServiceName string = searchService.name
output aiHubName string = aiHub.name
output aiProjectName string = aiProject.name
output resourceGroupName string = resourceGroup().name
```

### Step 2.2 — Create the Parameters File

Create `infra/parameters.template.json`:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "tenantId": {
      "value": "${TENANT_ID}"
    },
    "modelDeployment": {
      "value": "${MODEL_DEPLOYMENT}"
    },
    "tpmQuota": {
      "value": ${TPM_QUOTA}
    },
    "searchSku": {
      "value": "basic"
    }
  }
}
```

---

## Part 3: Agent Configuration (10 minutes)

### Step 3.1 — Create the System Prompt Template

Create `src/prompts/system-prompt.template.md`:

```markdown
# {{display_name}} — System Prompt

You are **{{display_name}}**, a helpful FAQ assistant.

## Your Role
- Answer questions using ONLY the knowledge base provided to you.
- If the answer is not in the knowledge base, say: "I don't have information about that. Please contact our support team."
- Be concise, helpful, and professional.

## Branding
- Always refer to yourself as "{{display_name}}"
- Use a friendly, professional tone

## Guardrails
- NEVER discuss: {{blocked_topics_list}}
- Keep responses under {{max_response_length}} words
- Do not make up information or speculate
- Do not share internal system details

## Response Format
- Use bullet points for lists
- Provide step-by-step instructions when applicable
- Include relevant links from the knowledge base when available
```

### Step 3.2 — Create the Agent Definition Template

Create `src/agent.template.yaml`:

```yaml
# agent.template.yaml — Parameterized agent definition
name: "{{tenant_id}}-faq-agent"
display_name: "{{display_name}}"
description: "FAQ assistant for {{display_name}}"

model:
  deployment: "{{model_deployment}}"
  parameters:
    temperature: {{temperature}}
    max_tokens: 1024
    top_p: 0.95

instructions_file: "./prompts/system-prompt.md"

tools:
  - type: "azure_ai_search"
    azure_ai_search:
      index_name: "{{tenant_id}}-faq-index"
      search_service: "{{tenant_id}}-faq-search"
      query_type: "vector_semantic_hybrid"
      top_k: 5

metadata:
  tenant_id: "{{tenant_id}}"
  template_version: "1.0.0"
  created_by: "faq-bot-template"
```

---

## Part 4: Deployment Automation (15 minutes)

### Step 4.1 — Create the Deployment Script

Create `deploy.py`:

```python
#!/usr/bin/env python3
"""
FAQ Bot Solution Template — Deployment Script
Reads a customer config file and provisions all resources.
"""

import yaml
import json
import subprocess
import sys
import os
import argparse
from pathlib import Path
from string import Template


def load_config(config_path: str) -> dict:
    """Load and validate customer configuration."""
    with open(config_path, 'r') as f:
        config = yaml.safe_load(f)

    required = ['tenant_id', 'display_name', 'branding', 'knowledge', 'model']
    for field in required:
        if field not in config:
            raise ValueError(f"Missing required field: {field}")

    print(f"✅ Loaded config for tenant: {config['tenant_id']}")
    return config


def render_template(template_path: str, config: dict) -> str:
    """Render a template file with customer config values."""
    with open(template_path, 'r') as f:
        content = f.read()

    replacements = {
        '{{tenant_id}}': config['tenant_id'],
        '{{display_name}}': config['display_name'],
        '{{model_deployment}}': config['model']['deployment_name'],
        '{{temperature}}': str(config['model']['temperature']),
        '{{max_response_length}}': str(config['guardrails']['max_response_length']),
        '{{blocked_topics_list}}': ', '.join(config['guardrails']['blocked_topics']),
    }

    for placeholder, value in replacements.items():
        content = content.replace(placeholder, value)

    return content


def generate_parameters(config: dict, output_path: str):
    """Generate Bicep parameters file from customer config."""
    params = {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "tenantId": {"value": config['tenant_id']},
            "modelDeployment": {"value": config['model']['deployment_name']},
            "tpmQuota": {"value": 30000},
            "searchSku": {"value": "basic"}
        }
    }

    with open(output_path, 'w') as f:
        json.dump(params, f, indent=2)

    print(f"✅ Generated parameters: {output_path}")


def render_agent_config(config: dict, output_dir: str):
    """Render agent.yaml and system prompt from templates."""
    # Render system prompt
    prompt = render_template('src/prompts/system-prompt.template.md', config)
    prompt_path = os.path.join(output_dir, 'prompts', 'system-prompt.md')
    os.makedirs(os.path.dirname(prompt_path), exist_ok=True)
    with open(prompt_path, 'w') as f:
        f.write(prompt)
    print(f"✅ Rendered system prompt: {prompt_path}")

    # Render agent.yaml
    agent = render_template('src/agent.template.yaml', config)
    agent_path = os.path.join(output_dir, 'agent.yaml')
    with open(agent_path, 'w') as f:
        f.write(agent)
    print(f"✅ Rendered agent config: {agent_path}")


def deploy_infrastructure(config: dict, resource_group: str, params_path: str):
    """Deploy Azure resources using Bicep."""
    print(f"\n🚀 Deploying infrastructure for {config['tenant_id']}...")

    cmd = [
        'az', 'deployment', 'group', 'create',
        '--resource-group', resource_group,
        '--template-file', 'infra/main.bicep',
        '--parameters', f'@{params_path}',
        '--name', f"{config['tenant_id']}-faq-deployment",
        '--output', 'json'
    ]

    print(f"   Running: {' '.join(cmd)}")

    result = subprocess.run(cmd, capture_output=True, text=True)

    if result.returncode != 0:
        print(f"❌ Deployment failed:\n{result.stderr}")
        return False

    outputs = json.loads(result.stdout)
    print(f"✅ Infrastructure deployed successfully")
    return True


def upload_knowledge_base(config: dict, data_dir: str):
    """Upload FAQ documents to the tenant's blob container."""
    tenant_id = config['tenant_id']
    container = config['knowledge']['source_container']
    storage_account = f"{tenant_id}faqstorage"
    source = os.path.join(data_dir, tenant_id)

    if not os.path.exists(source):
        print(f"⚠️  No data directory found at {source}, skipping upload")
        return

    print(f"\n📤 Uploading knowledge base for {tenant_id}...")

    cmd = [
        'az', 'storage', 'blob', 'upload-batch',
        '--account-name', storage_account,
        '--destination', container,
        '--source', source,
        '--overwrite', 'true',
        '--auth-mode', 'login'
    ]

    result = subprocess.run(cmd, capture_output=True, text=True)

    if result.returncode == 0:
        print(f"✅ Knowledge base uploaded")
    else:
        print(f"⚠️  Upload warning: {result.stderr}")


def run_smoke_test(config: dict) -> bool:
    """Run post-deployment smoke test."""
    print(f"\n🧪 Running smoke tests for {config['tenant_id']}...")

    cmd = [
        'python', 'tests/smoke_test.py',
        '--tenant-id', config['tenant_id'],
        '--display-name', config['display_name']
    ]

    result = subprocess.run(cmd, capture_output=True, text=True)

    if result.returncode == 0:
        print(f"✅ All smoke tests passed")
        return True
    else:
        print(f"❌ Smoke tests failed:\n{result.stdout}\n{result.stderr}")
        return False


def main():
    parser = argparse.ArgumentParser(description='Deploy FAQ Bot for a customer')
    parser.add_argument('--config', required=True, help='Path to customer config YAML')
    parser.add_argument('--resource-group', required=True, help='Azure resource group name')
    parser.add_argument('--skip-infra', action='store_true', help='Skip infrastructure deployment')
    parser.add_argument('--skip-upload', action='store_true', help='Skip knowledge base upload')
    parser.add_argument('--skip-tests', action='store_true', help='Skip smoke tests')
    args = parser.parse_args()

    print("=" * 60)
    print("  FAQ Bot Solution Template — Deployment")
    print("=" * 60)

    # 1. Load config
    config = load_config(args.config)
    tenant_id = config['tenant_id']

    # 2. Create output directory
    output_dir = f"deployments/{tenant_id}"
    os.makedirs(output_dir, exist_ok=True)

    # 3. Generate parameters
    params_path = f"{output_dir}/parameters.json"
    generate_parameters(config, params_path)

    # 4. Render agent configuration
    render_agent_config(config, output_dir)

    # 5. Deploy infrastructure
    if not args.skip_infra:
        if not deploy_infrastructure(config, args.resource_group, params_path):
            sys.exit(1)

    # 6. Upload knowledge base
    if not args.skip_upload:
        upload_knowledge_base(config, 'data')

    # 7. Run smoke tests
    if not args.skip_tests:
        if not run_smoke_test(config):
            sys.exit(1)

    print("\n" + "=" * 60)
    print(f"  ✅ {config['display_name']} deployed successfully!")
    print("=" * 60)


if __name__ == '__main__':
    main()
```

### Step 4.2 — Create the Smoke Test

Create `tests/smoke_test.py`:

```python
#!/usr/bin/env python3
"""
Smoke test for FAQ Bot deployments.
Validates that the deployed agent responds correctly.
"""

import argparse
import sys


def test_agent_responds(tenant_id: str, display_name: str) -> bool:
    """Test that the agent endpoint is reachable and responds."""
    print(f"  [TEST] Agent responds for {tenant_id}...")

    # In a real deployment, this would call the agent endpoint
    # For the lab, we simulate with a configuration check
    try:
        # Verify agent.yaml was rendered correctly
        import yaml
        with open(f"deployments/{tenant_id}/agent.yaml", 'r') as f:
            agent = yaml.safe_load(f)

        assert agent['name'] == f"{tenant_id}-faq-agent", \
            f"Agent name mismatch: {agent['name']}"
        assert agent['display_name'] == display_name, \
            f"Display name mismatch: {agent['display_name']}"

        print(f"    ✅ Agent config is valid")
        return True
    except Exception as e:
        print(f"    ❌ Failed: {e}")
        return False


def test_tenant_isolation(tenant_id: str) -> bool:
    """Test that the agent uses tenant-specific resources."""
    print(f"  [TEST] Tenant isolation for {tenant_id}...")

    try:
        import yaml
        with open(f"deployments/{tenant_id}/agent.yaml", 'r') as f:
            agent = yaml.safe_load(f)

        search_tool = agent['tools'][0]
        index_name = search_tool['azure_ai_search']['index_name']

        assert tenant_id in index_name, \
            f"Index name '{index_name}' does not contain tenant_id '{tenant_id}'"

        print(f"    ✅ Tenant isolation verified (index: {index_name})")
        return True
    except Exception as e:
        print(f"    ❌ Failed: {e}")
        return False


def test_guardrails(tenant_id: str) -> bool:
    """Test that guardrails are configured."""
    print(f"  [TEST] Guardrails for {tenant_id}...")

    try:
        with open(f"deployments/{tenant_id}/prompts/system-prompt.md", 'r') as f:
            prompt = f.read()

        assert "NEVER discuss" in prompt, "Guardrails section missing from prompt"
        assert len(prompt) > 100, "System prompt too short"

        print(f"    ✅ Guardrails configured in system prompt")
        return True
    except Exception as e:
        print(f"    ❌ Failed: {e}")
        return False


def main():
    parser = argparse.ArgumentParser(description='FAQ Bot Smoke Tests')
    parser.add_argument('--tenant-id', required=True)
    parser.add_argument('--display-name', required=True)
    args = parser.parse_args()

    print(f"\n🧪 Smoke Tests — {args.tenant_id}")
    print("-" * 40)

    results = [
        test_agent_responds(args.tenant_id, args.display_name),
        test_tenant_isolation(args.tenant_id),
        test_guardrails(args.tenant_id),
    ]

    passed = sum(results)
    total = len(results)

    print(f"\nResults: {passed}/{total} passed")

    if all(results):
        print("✅ All tests passed!")
        sys.exit(0)
    else:
        print("❌ Some tests failed!")
        sys.exit(1)


if __name__ == '__main__':
    main()
```

---

## Part 5: Deploy & Validate (10 minutes)

### Step 5.1 — Create Sample Knowledge Base Files

Create sample FAQ documents for each customer:

**`data/contoso/product-faq.md`**:
```markdown
# Contoso Product FAQ

## What is Contoso Cloud Suite?
Contoso Cloud Suite is our flagship productivity platform that helps
teams collaborate, manage projects, and automate workflows.

## How do I reset my password?
1. Go to https://account.contoso.com
2. Click "Forgot Password"
3. Enter your registered email
4. Follow the reset link sent to your inbox

## What are the pricing plans?
- **Starter**: $9/user/month — Up to 10 users
- **Business**: $29/user/month — Unlimited users, advanced analytics
- **Enterprise**: Custom pricing — SSO, compliance, SLA
```

**`data/fabrikam/support-faq.md`**:
```markdown
# Fabrikam Support FAQ

## What products does Fabrikam offer?
Fabrikam specializes in IoT devices for smart manufacturing,
including sensors, controllers, and monitoring dashboards.

## How do I contact support?
- Email: support@fabrikam.com
- Phone: 1-800-FAB-HELP (M-F 9AM-6PM PST)
- Live chat: Available on fabrikam.com

## What is the warranty policy?
All Fabrikam devices come with a 2-year standard warranty.
Extended warranty (5 years) is available for $99/device.
```

### Step 5.2 — Deploy for Contoso

```bash
# Deploy Contoso FAQ Bot (skip infra for lab environment)
python deploy.py \
  --config configs/contoso.yaml \
  --resource-group rg-faq-lab \
  --skip-infra \
  --skip-upload

# Expected output:
# ✅ Loaded config for tenant: contoso
# ✅ Generated parameters: deployments/contoso/parameters.json
# ✅ Rendered system prompt: deployments/contoso/prompts/system-prompt.md
# ✅ Rendered agent config: deployments/contoso/agent.yaml
# ✅ All smoke tests passed
# ✅ Contoso Support Assistant deployed successfully!
```

### Step 5.3 — Deploy for Fabrikam

```bash
# Deploy Fabrikam FAQ Bot
python deploy.py \
  --config configs/fabrikam.yaml \
  --resource-group rg-faq-lab \
  --skip-infra \
  --skip-upload

# Expected output:
# ✅ Loaded config for tenant: fabrikam
# ✅ Fabrikam Help Desk deployed successfully!
```

### Step 5.4 — Verify Tenant Isolation

```bash
# Compare the two deployments
diff deployments/contoso/agent.yaml deployments/fabrikam/agent.yaml
```

Expected differences:
- Agent name: `contoso-faq-agent` vs `fabrikam-faq-agent`
- Display name: `Contoso Support Assistant` vs `Fabrikam Help Desk`
- Model deployment: `gpt-4o` vs `gpt-4o-mini`
- Search index: `contoso-faq-index` vs `fabrikam-faq-index`
- Temperature: `0.3` vs `0.2`

---

## Stretch Goals (Optional)

### Stretch Goal 1: Add a Web Frontend Template

Create a parameterized chat widget that reads branding from the customer config:

```html
<!-- src/frontend/chat-widget.template.html -->
<div id="faq-chat" style="--brand-color: {{primary_color}}">
  <div class="chat-header">
    <img src="{{logo_url}}" alt="Logo" class="chat-logo">
    <span>{{display_name}}</span>
  </div>
  <div class="chat-messages" id="messages">
    <div class="bot-message">{{greeting_message}}</div>
  </div>
  <input type="text" placeholder="Ask a question..." id="chat-input">
</div>
```

### Stretch Goal 2: Add a GitHub Actions Deployment Pipeline

Create `.github/workflows/deploy-customer.yml`:

```yaml
name: Deploy FAQ Bot for Customer
on:
  workflow_dispatch:
    inputs:
      config_file:
        description: 'Path to customer config YAML'
        required: true
      resource_group:
        description: 'Azure resource group'
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - run: pip install pyyaml
      - run: |
          python deploy.py \
            --config ${{ github.event.inputs.config_file }} \
            --resource-group ${{ github.event.inputs.resource_group }}
```

### Stretch Goal 3: Cost Estimation Report

Add a `--estimate-cost` flag to `deploy.py` that queries Azure pricing APIs and generates a monthly cost estimate per customer.

---

## Lab Checklist

Before submitting, verify:

- [ ] Template directory structure is complete
- [ ] Customer config schema is defined
- [ ] Two customer configs exist (Contoso + Fabrikam)
- [ ] Bicep template parameterizes all resources with tenant ID
- [ ] System prompt template includes configurable branding and guardrails
- [ ] Agent template uses tenant-specific search indexes
- [ ] Deployment script processes config and renders all templates
- [ ] Smoke tests validate agent config, isolation, and guardrails
- [ ] Both customers deploy successfully with different configurations
- [ ] `diff` between deployments shows only customer-specific values

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `yaml.safe_load` fails | Ensure YAML indentation uses spaces, not tabs |
| Bicep deployment error | Run `az bicep build -f infra/main.bicep` to validate syntax |
| Smoke test can't find `deployments/` | Run `deploy.py` before tests — it creates the directory |
| Tenant isolation test fails | Check that `agent.template.yaml` uses `{{tenant_id}}` in the index name |
| Template rendering misses values | Verify all `{{placeholders}}` match keys in the config YAML |

---

## Clean Up

```bash
# Remove generated deployment artifacts
rm -rf deployments/

# If you deployed real Azure resources:
az group delete --name rg-faq-lab --yes --no-wait
```

---

*Module 38 of 45 — Azure AI Foundry: Zero to Hero*
*ARC 8: Advanced Patterns & Enterprise*
*© 2026 Azure AI Foundry Training Team*
