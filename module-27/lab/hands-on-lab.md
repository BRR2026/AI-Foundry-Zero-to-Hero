# Hands-On Lab: CI/CD Pipelines for AI with Azure AI Foundry

> **Module 27** · Azure AI Foundry: Zero to Hero  
> **Duration:** 60–90 minutes · **Level:** Intermediate–Advanced

---

## 🎯 Lab Objectives

In this lab, you will:

1. Configure OIDC (workload identity federation) for GitHub Actions to Azure authentication
2. Author a Bicep template that provisions an AI Foundry Hub, Project, and AI Services account
3. Write a Python script that deploys a model to Azure AI Foundry using the SDK
4. Create an evaluation script that acts as a quality gate in your CI/CD pipeline
5. Build and run a complete GitHub Actions workflow that deploys and evaluates end-to-end

---

## 📋 Prerequisites

| Requirement | Details |
|---|---|
| Azure subscription | Contributor role, AI Foundry resource provider registered |
| GitHub account | With a repository and Actions enabled |
| Azure CLI | v2.60+ (`az --version`) |
| Python | 3.10+ with pip |
| VS Code | With Azure and GitHub extensions (recommended) |

---

## 🔧 Lab Setup

### Step 1: Create the Repository Structure

Create a new GitHub repository (or use an existing one) and set up the following directory structure:

```bash
mkdir -p .github/workflows infra scripts data tests
```

Your repository should look like this:

```
ai-foundry-cicd/
├── .github/
│   └── workflows/
│       └── ai-foundry-deploy.yml
├── infra/
│   └── main.bicep
├── scripts/
│   ├── deploy_model.py
│   └── run_evaluation.py
├── data/
│   └── evaluation_dataset.jsonl
├── tests/
│   └── test_prompts.py
└── requirements.txt
```

### Step 2: Create requirements.txt

```text
azure-ai-projects>=1.0.0
azure-ai-evaluation>=1.0.0
azure-identity>=1.17.0
pytest>=8.0.0
ruff>=0.4.0
```

---

## 📝 Exercise 1: Configure OIDC Authentication (15 min)

### 1.1 Create an Azure AD App Registration

```bash
# Create the app registration
az ad app create --display-name "github-actions-ai-foundry"

# Note the appId (client ID) from the output
APP_ID=$(az ad app list --display-name "github-actions-ai-foundry" --query "[0].appId" -o tsv)
echo "Client ID: $APP_ID"

# Create a service principal
az ad sp create --id $APP_ID

# Get the object ID of the app
APP_OBJECT_ID=$(az ad app show --id $APP_ID --query "id" -o tsv)
```

### 1.2 Add Federated Credential for GitHub

```bash
# Create federated credential for the main branch
az ad app federated-credential create \
  --id $APP_OBJECT_ID \
  --parameters '{
    "name": "github-actions-main",
    "issuer": "https://token.actions.githubusercontent.com",
    "subject": "repo:YOUR_ORG/YOUR_REPO:ref:refs/heads/main",
    "description": "GitHub Actions deploy from main branch",
    "audiences": ["api://AzureADTokenExchange"]
  }'

# Create federated credential for pull requests
az ad app federated-credential create \
  --id $APP_OBJECT_ID \
  --parameters '{
    "name": "github-actions-pr",
    "issuer": "https://token.actions.githubusercontent.com",
    "subject": "repo:YOUR_ORG/YOUR_REPO:pull_request",
    "description": "GitHub Actions PR validation",
    "audiences": ["api://AzureADTokenExchange"]
  }'
```

> ⚠️ **Replace** `YOUR_ORG/YOUR_REPO` with your actual GitHub organization and repository name.

### 1.3 Assign Azure Permissions

```bash
SUBSCRIPTION_ID=$(az account show --query "id" -o tsv)
SP_OBJECT_ID=$(az ad sp show --id $APP_ID --query "id" -o tsv)

# Assign Contributor role on the resource group
az role assignment create \
  --assignee-object-id $SP_OBJECT_ID \
  --assignee-principal-type ServicePrincipal \
  --role "Contributor" \
  --scope "/subscriptions/$SUBSCRIPTION_ID/resourceGroups/rg-ai-foundry-lab"

# Assign Cognitive Services Contributor for AI Services
az role assignment create \
  --assignee-object-id $SP_OBJECT_ID \
  --assignee-principal-type ServicePrincipal \
  --role "Cognitive Services Contributor" \
  --scope "/subscriptions/$SUBSCRIPTION_ID/resourceGroups/rg-ai-foundry-lab"
```

### 1.4 Store Secrets in GitHub

Go to your GitHub repository → **Settings** → **Secrets and variables** → **Actions** and add:

| Secret Name | Value |
|---|---|
| `AZURE_CLIENT_ID` | The `APP_ID` from step 1.1 |
| `AZURE_TENANT_ID` | Your Azure AD tenant ID (`az account show --query tenantId -o tsv`) |
| `AZURE_SUBSCRIPTION_ID` | Your subscription ID (`az account show --query id -o tsv`) |

### ✅ Checkpoint

Verify your setup:
```bash
# Verify app registration exists
az ad app show --id $APP_ID --query "{name:displayName, appId:appId}" -o table

# Verify federated credentials
az ad app federated-credential list --id $APP_OBJECT_ID --query "[].{name:name, subject:subject}" -o table

# Verify role assignments
az role assignment list --assignee $SP_OBJECT_ID --query "[].{role:roleDefinitionName, scope:scope}" -o table
```

---

## 📝 Exercise 2: Author the Bicep Template (15 min)

### 2.1 Create the Infrastructure Template

Create `infra/main.bicep`:

```bicep
// infra/main.bicep
@description('Environment name')
param env string = 'dev'

@description('Azure region')
param location string = resourceGroup().location

@description('Base name for resources')
param baseName string = 'ailab'

var uniqueSuffix = uniqueString(resourceGroup().id)

// ── AI Services Account ──
resource aiServices 'Microsoft.CognitiveServices/accounts@2024-10-01' = {
  name: 'ais-${baseName}-${env}-${uniqueSuffix}'
  location: location
  kind: 'AIServices'
  sku: { name: 'S0' }
  properties: {
    customSubDomainName: 'ais-${baseName}-${env}-${uniqueSuffix}'
    publicNetworkAccess: 'Enabled'
  }
}

// ── AI Foundry Hub ──
resource hub 'Microsoft.MachineLearningServices/workspaces@2024-10-01' = {
  name: 'hub-${baseName}-${env}-${uniqueSuffix}'
  location: location
  kind: 'Hub'
  sku: { name: 'Basic'; tier: 'Basic' }
  identity: { type: 'SystemAssigned' }
  properties: {
    friendlyName: 'AI Foundry Hub (${env})'
    description: 'Deployed via CI/CD - Module 27 Lab'
  }
}

// ── AI Foundry Project ──
resource project 'Microsoft.MachineLearningServices/workspaces@2024-10-01' = {
  name: 'proj-${baseName}-${env}'
  location: location
  kind: 'Project'
  sku: { name: 'Basic'; tier: 'Basic' }
  identity: { type: 'SystemAssigned' }
  properties: {
    friendlyName: 'AI Project (${env})'
    hubResourceId: hub.id
  }
}

// ── Outputs ──
output hubId string = hub.id
output projectId string = project.id
output projectName string = project.name
output aiServicesEndpoint string = aiServices.properties.endpoint
```

### 2.2 Validate the Template Locally

```bash
# Create the resource group first
az group create --name rg-ai-foundry-lab --location eastus2

# Validate the Bicep template
az bicep build --file infra/main.bicep

# Preview what would be created
az deployment group what-if \
  --resource-group rg-ai-foundry-lab \
  --template-file infra/main.bicep \
  --parameters env=dev
```

### ✅ Checkpoint

- [ ] `az bicep build` succeeds without errors
- [ ] `what-if` output shows the expected resources (AI Services, Hub, Project)

---

## 📝 Exercise 3: Write the Deployment Script (10 min)

### 3.1 Create the Model Deployment Script

Create `scripts/deploy_model.py`:

```python
"""Deploy a model to Azure AI Foundry using the Python SDK."""
import argparse
import os
import sys
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient


def deploy_model(args):
    """Deploy or update a model deployment in AI Foundry."""
    credential = DefaultAzureCredential()

    client = AIProjectClient(
        credential=credential,
        subscription_id=args.subscription_id or os.environ.get("AZURE_SUBSCRIPTION_ID"),
        resource_group_name=args.resource_group,
        project_name=args.project_name,
    )

    print(f"🚀 Deploying model '{args.model_name}' as '{args.deployment_name}'...")

    deployment = client.deployments.begin_create_or_update(
        deployment_name=args.deployment_name,
        deployment={
            "model": {"name": args.model_name, "version": "latest"},
            "sku": {
                "name": "GlobalStandard",
                "capacity": args.sku_capacity,
            },
        },
    ).result()

    print(f"✅ Deployment '{deployment.name}' created successfully!")
    print(f"   Model: {args.model_name}")
    print(f"   SKU Capacity: {args.sku_capacity}")
    return deployment


def main():
    parser = argparse.ArgumentParser(description="Deploy model to AI Foundry")
    parser.add_argument("--project-name", required=True, help="AI Foundry project name")
    parser.add_argument("--resource-group", required=True, help="Azure resource group")
    parser.add_argument("--subscription-id", default=None, help="Azure subscription ID")
    parser.add_argument("--model-name", default="gpt-4o", help="Model to deploy")
    parser.add_argument("--deployment-name", required=True, help="Deployment name")
    parser.add_argument("--sku-capacity", type=int, default=30, help="SKU capacity (TPM in thousands)")

    args = parser.parse_args()

    try:
        deploy_model(args)
    except Exception as e:
        print(f"❌ Deployment failed: {e}", file=sys.stderr)
        sys.exit(1)


if __name__ == "__main__":
    main()
```

---

## 📝 Exercise 4: Create the Evaluation Script (15 min)

### 4.1 Create the Evaluation Dataset

Create `data/evaluation_dataset.jsonl`:

```jsonl
{"query": "What is Azure AI Foundry?", "context": "Azure AI Foundry is a unified platform for building, evaluating, and deploying AI solutions. It provides tools for model management, prompt engineering, and responsible AI.", "response": "Azure AI Foundry is Microsoft's unified platform for building enterprise AI applications, offering tools for model management, evaluation, and deployment."}
{"query": "How do I deploy a model in AI Foundry?", "context": "Models can be deployed via the AI Foundry portal, Azure CLI, Python SDK, or Infrastructure as Code templates like Bicep and Terraform.", "response": "You can deploy a model in AI Foundry using the portal UI, Azure CLI commands, the Python SDK (azure-ai-projects), or IaC templates."}
{"query": "What evaluation metrics does AI Foundry support?", "context": "AI Foundry's evaluation SDK supports groundedness, relevance, coherence, fluency, and safety metrics. Custom evaluators can also be created.", "response": "AI Foundry supports groundedness, relevance, coherence, fluency, and safety metrics through its evaluation SDK, plus custom evaluators."}
{"query": "What is workload identity federation?", "context": "Workload identity federation allows GitHub Actions and other external identity providers to authenticate to Azure without storing credentials, using short-lived OIDC tokens.", "response": "Workload identity federation enables passwordless authentication from GitHub Actions to Azure using short-lived OIDC tokens instead of stored secrets."}
{"query": "How do I create an AI Foundry project?", "context": "An AI Foundry project is created within a hub. It can be created via the portal, CLI, SDK, or Bicep/Terraform templates.", "response": "Create an AI Foundry project within a hub using the portal, Azure CLI, Python SDK, or IaC templates like Bicep."}
```

### 4.2 Create the Evaluation Script

Create `scripts/run_evaluation.py`:

```python
"""Run AI evaluations as a CI/CD quality gate."""
import argparse
import json
import sys
from azure.identity import DefaultAzureCredential
from azure.ai.evaluation import evaluate, GroundednessEvaluator, RelevanceEvaluator


def run_evaluation(args):
    """Execute evaluations and enforce quality thresholds."""
    credential = DefaultAzureCredential()

    # Configure the judge model
    model_config = {
        "azure_deployment": args.eval_deployment,
        "azure_endpoint": args.eval_endpoint,
    }

    # Initialize evaluators
    groundedness = GroundednessEvaluator(model_config=model_config)
    relevance = RelevanceEvaluator(model_config=model_config)

    print("🧪 Running AI evaluations...")
    print(f"   Dataset: {args.data_path}")
    print(f"   Judge model: {args.eval_deployment}")
    print()

    # Run the evaluation
    result = evaluate(
        data=args.data_path,
        evaluators={
            "groundedness": groundedness,
            "relevance": relevance,
        },
        evaluator_config={
            "groundedness": {
                "query": "${data.query}",
                "context": "${data.context}",
                "response": "${data.response}",
            },
            "relevance": {
                "query": "${data.query}",
                "response": "${data.response}",
            },
        },
    )

    # Check results against thresholds
    metrics = result["metrics"]
    thresholds = {
        "groundedness.groundedness": args.threshold,
        "relevance.relevance": args.threshold,
    }

    print("📊 Evaluation Results:")
    print("-" * 50)

    failures = []
    for metric_name, threshold in thresholds.items():
        score = metrics.get(metric_name, 0)
        status = "✅ PASS" if score >= threshold else "❌ FAIL"
        print(f"   {status} | {metric_name}: {score:.2f} (threshold: {threshold})")
        if score < threshold:
            failures.append(metric_name)

    print("-" * 50)

    # Save results as artifact
    with open("evaluation_results.json", "w") as f:
        json.dump({"metrics": metrics, "thresholds": thresholds, "passed": len(failures) == 0}, f, indent=2)
    print(f"\n💾 Results saved to evaluation_results.json")

    if failures:
        print(f"\n🚫 EVALUATION GATE FAILED — {len(failures)} metric(s) below threshold")
        sys.exit(1)
    else:
        print(f"\n🎉 ALL EVALUATION GATES PASSED")


def main():
    parser = argparse.ArgumentParser(description="Run AI evaluations")
    parser.add_argument("--data-path", default="data/evaluation_dataset.jsonl")
    parser.add_argument("--eval-deployment", default="gpt-4o-eval")
    parser.add_argument("--eval-endpoint", required=True)
    parser.add_argument("--threshold", type=float, default=4.0)
    parser.add_argument("--env", default="staging")
    args = parser.parse_args()

    try:
        run_evaluation(args)
    except Exception as e:
        print(f"❌ Evaluation failed: {e}", file=sys.stderr)
        sys.exit(1)


if __name__ == "__main__":
    main()
```

---

## 📝 Exercise 5: Build the GitHub Actions Workflow (15 min)

### 5.1 Create the Complete Workflow

Create `.github/workflows/ai-foundry-deploy.yml`:

```yaml
name: AI Foundry — Deploy & Evaluate

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  id-token: write
  contents: read

env:
  RESOURCE_GROUP: rg-ai-foundry-lab
  PROJECT_NAME: proj-ailab
  LOCATION: eastus2

jobs:
  # ── Job 1: Lint & Unit Tests ──
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: "pip"
      - run: |
          pip install -r requirements.txt
          ruff check scripts/
          pytest tests/ -v --tb=short

  # ── Job 2: Validate Infrastructure ──
  validate-iac:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      - name: Validate Bicep
        run: |
          az bicep build --file infra/main.bicep
          az deployment group what-if \
            --resource-group $RESOURCE_GROUP \
            --template-file infra/main.bicep \
            --parameters env=staging

  # ── Job 3: Deploy to Staging ──
  deploy-staging:
    needs: [lint-and-test, validate-iac]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: "pip"
      - name: Deploy AI Foundry Infrastructure
        run: |
          az deployment group create \
            --resource-group $RESOURCE_GROUP \
            --template-file infra/main.bicep \
            --parameters env=staging
      - name: Deploy Model
        run: |
          pip install azure-ai-projects azure-identity
          python scripts/deploy_model.py \
            --project-name $PROJECT_NAME-staging \
            --resource-group $RESOURCE_GROUP \
            --deployment-name gpt-4o-staging \
            --sku-capacity 10

  # ── Job 4: Evaluate ──
  evaluate:
    needs: deploy-staging
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: "pip"
      - name: Run Evaluations
        run: |
          pip install azure-ai-evaluation azure-ai-projects azure-identity
          python scripts/run_evaluation.py \
            --eval-endpoint "https://ais-ailab-staging.openai.azure.com/" \
            --threshold 4.0
      - name: Upload Evaluation Results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: evaluation-results
          path: evaluation_results.json

  # ── Job 5: Deploy to Production ──
  deploy-production:
    needs: evaluate
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      - name: Deploy to Production
        run: |
          az deployment group create \
            --resource-group $RESOURCE_GROUP \
            --template-file infra/main.bicep \
            --parameters env=production
```

### 5.2 Create a Simple Unit Test

Create `tests/test_prompts.py`:

```python
"""Unit tests for prompt templates and configuration."""
import json
import os
import pytest


def test_evaluation_dataset_is_valid_jsonl():
    """Verify the evaluation dataset is valid JSONL format."""
    data_path = os.path.join(os.path.dirname(__file__), "..", "data", "evaluation_dataset.jsonl")
    with open(data_path, "r") as f:
        lines = f.readlines()

    assert len(lines) >= 5, "Evaluation dataset should have at least 5 samples"

    for i, line in enumerate(lines):
        record = json.loads(line.strip())
        assert "query" in record, f"Line {i+1}: missing 'query' field"
        assert "context" in record, f"Line {i+1}: missing 'context' field"
        assert "response" in record, f"Line {i+1}: missing 'response' field"
        assert len(record["query"]) > 0, f"Line {i+1}: 'query' is empty"


def test_bicep_template_exists():
    """Verify the Bicep template file exists."""
    bicep_path = os.path.join(os.path.dirname(__file__), "..", "infra", "main.bicep")
    assert os.path.exists(bicep_path), "infra/main.bicep must exist"


def test_requirements_include_ai_packages():
    """Verify required packages are in requirements.txt."""
    req_path = os.path.join(os.path.dirname(__file__), "..", "requirements.txt")
    with open(req_path, "r") as f:
        content = f.read()

    assert "azure-ai-projects" in content
    assert "azure-ai-evaluation" in content
    assert "azure-identity" in content
```

---

## 📝 Exercise 6: Run and Verify (10 min)

### 6.1 Configure GitHub Environments

1. Go to your GitHub repository → **Settings** → **Environments**
2. Create two environments:
   - **staging** — no protection rules
   - **production** — add a required reviewer (yourself or a team member)

### 6.2 Push and Monitor

```bash
# Add all files
git add -A
git commit -m "feat: add CI/CD pipeline for AI Foundry deployment"
git push origin main
```

3. Navigate to the **Actions** tab in your GitHub repository
4. Watch the pipeline execute through each stage
5. Verify the evaluation results artifact is uploaded

### ✅ Final Checklist

- [ ] OIDC authentication works (Azure Login step succeeds)
- [ ] Bicep validation passes
- [ ] Unit tests pass
- [ ] Infrastructure deploys to staging
- [ ] Model deployment succeeds
- [ ] Evaluation gate runs and produces results
- [ ] Evaluation results artifact is uploaded
- [ ] Production deployment waits for manual approval
- [ ] Production deployment succeeds after approval

---

## 🧹 Cleanup

After completing the lab, clean up resources to avoid charges:

```bash
# Delete the resource group and all resources
az group delete --name rg-ai-foundry-lab --yes --no-wait

# Delete the app registration
az ad app delete --id $APP_ID
```

---

## 🎓 Summary

In this lab, you built a complete CI/CD pipeline for Azure AI Foundry that:

| Component | Technology | Purpose |
|---|---|---|
| Authentication | OIDC / Federated credentials | Passwordless, secure auth |
| Infrastructure | Bicep templates | Reproducible resource provisioning |
| Model Deployment | Python SDK | Automated model deployment |
| Quality Gates | AI Evaluation SDK | Automated quality enforcement |
| Pipeline | GitHub Actions | End-to-end automation |
| Approval | GitHub Environments | Human-in-the-loop for production |

---

## 🔗 Additional Resources

- [Azure AI Foundry Documentation](https://learn.microsoft.com/azure/ai-foundry/)
- [GitHub Actions OIDC for Azure](https://docs.github.com/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-azure)
- [Bicep Documentation](https://learn.microsoft.com/azure/azure-resource-manager/bicep/)
- [Azure AI Evaluation SDK](https://learn.microsoft.com/azure/ai-foundry/how-to/evaluate-sdk)

---

*Azure AI Foundry: Zero to Hero — Module 27, Hands-On Lab*  
*© 2026 AI Training Series*
