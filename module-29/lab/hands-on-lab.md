# Module 29: Hands-On Lab

## Mini Hack — Deploy Your Agent to Dev and Prod Environments

**Course:** Azure AI Foundry — Zero to Hero (Module 29 of 45)
**Arc:** ARC 6 · DEPLOYMENT & MLOps
**Duration:** 90 minutes
**Difficulty:** Intermediate-Advanced

---

## Lab Overview

In this hands-on lab you will deploy an AI agent to two isolated environments — Development and Production — using Azure AI Foundry. You will provision infrastructure with parameterized Bicep templates, configure environment-specific settings via Azure App Configuration, implement a health check endpoint, and verify that the same container image behaves correctly in both environments with different configurations.

## Objectives

By the end of this lab you will have:

- ✅ Provisioned separate AI Foundry projects for Dev and Prod using Bicep
- ✅ Configured Azure App Configuration with environment-labeled settings
- ✅ Stored secrets in Azure Key Vault per environment
- ✅ Deployed the same container image to both environments
- ✅ Implemented and tested a `/health` endpoint
- ✅ Verified environment-specific behavior (model deployment, logging level, safety filters)

## Prerequisites

| Requirement | Details |
|-------------|---------|
| Azure Subscription | Active subscription with Contributor access |
| AI Foundry Hub | At least one hub provisioned (from Module 3) |
| Containerized Agent | Agent image from Module 28 pushed to ACR |
| Azure CLI | Version 2.60+ with `ml` extension |
| Python Environment | Python 3.10+ with `azure-ai-projects`, `azure-identity`, `azure-appconfiguration` |
| GitHub Repository | With Actions enabled (for optional CI/CD exercise) |

---

## Part 1: Prepare Infrastructure as Code (20 min)

### Step 1.1 — Create the Project Directory Structure

Create a directory structure for your multi-environment deployment:

```
scaling-lab/
├── infra/
│   ├── main.bicep
│   ├── params.dev.json
│   └── params.prod.json
├── src/
│   ├── app.py
│   ├── config.py
│   └── health.py
├── tests/
│   └── smoke_test.py
├── Dockerfile
└── .github/
    └── workflows/
        └── deploy-agent.yml
```

### Step 1.2 — Create the Bicep Template

Create `infra/main.bicep`:

```bicep
// Multi-environment AI Foundry deployment
param environmentName string   // 'dev' or 'prod'
param location string = resourceGroup().location
param aiHubName string
param aiProjectName string
param skuName string = 'Basic'
param modelDeploymentCapacity int = 10

// AI Foundry Hub
resource aiHub 'Microsoft.MachineLearningServices/workspaces@2024-10-01' = {
  name: aiHubName
  location: location
  kind: 'hub'
  sku: {
    name: skuName
  }
  identity: {
    type: 'SystemAssigned'
  }
  properties: {
    friendlyName: 'AI Hub - ${environmentName}'
    publicNetworkAccess: environmentName == 'prod' ? 'Disabled' : 'Enabled'
  }
  tags: {
    environment: environmentName
    managedBy: 'bicep'
    module: 'module-29-lab'
  }
}

// AI Foundry Project
resource aiProject 'Microsoft.MachineLearningServices/workspaces@2024-10-01' = {
  name: aiProjectName
  location: location
  kind: 'project'
  sku: {
    name: skuName
  }
  properties: {
    friendlyName: 'Agent Project - ${environmentName}'
    hubResourceId: aiHub.id
  }
  tags: {
    environment: environmentName
    managedBy: 'bicep'
  }
}

// Azure App Configuration
resource appConfig 'Microsoft.AppConfiguration/configurationStores@2023-03-01' = {
  name: 'appconfig-agent-${environmentName}'
  location: location
  sku: {
    name: 'Standard'
  }
  tags: {
    environment: environmentName
  }
}

// Outputs
output hubId string = aiHub.id
output projectId string = aiProject.id
output appConfigEndpoint string = appConfig.properties.endpoint
```

### Step 1.3 — Create Environment Parameter Files

Create `infra/params.dev.json`:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "environmentName": { "value": "dev" },
    "aiHubName": { "value": "hub-agent-dev" },
    "aiProjectName": { "value": "proj-agent-dev" },
    "skuName": { "value": "Basic" },
    "modelDeploymentCapacity": { "value": 10 }
  }
}
```

Create `infra/params.prod.json`:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "environmentName": { "value": "prod" },
    "aiHubName": { "value": "hub-agent-prod" },
    "aiProjectName": { "value": "proj-agent-prod" },
    "skuName": { "value": "Standard" },
    "modelDeploymentCapacity": { "value": 100 }
  }
}
```

### ✅ Checkpoint 1

Verify your directory structure matches the layout above and both parameter files have valid JSON.

```bash
cat infra/params.dev.json | python -m json.tool
cat infra/params.prod.json | python -m json.tool
```

---

## Part 2: Deploy Infrastructure (20 min)

### Step 2.1 — Create Resource Groups

```bash
# Create Dev resource group
az group create \
  --name rg-ai-dev \
  --location eastus \
  --tags environment=dev module=module-29

# Create Prod resource group
az group create \
  --name rg-ai-prod \
  --location eastus \
  --tags environment=prod module=module-29
```

### Step 2.2 — Deploy Dev Environment

```bash
az deployment group create \
  --resource-group rg-ai-dev \
  --template-file infra/main.bicep \
  --parameters infra/params.dev.json \
  --name deploy-dev-$(date +%Y%m%d%H%M)
```

### Step 2.3 — Deploy Prod Environment

```bash
az deployment group create \
  --resource-group rg-ai-prod \
  --template-file infra/main.bicep \
  --parameters infra/params.prod.json \
  --name deploy-prod-$(date +%Y%m%d%H%M)
```

### Step 2.4 — Verify Deployments

```bash
# List Dev resources
az resource list --resource-group rg-ai-dev --output table

# List Prod resources
az resource list --resource-group rg-ai-prod --output table
```

### ✅ Checkpoint 2

You should see AI Foundry Hub, Project, and App Configuration resources in both resource groups. Verify that the Dev hub has `publicNetworkAccess: Enabled` and Prod has `publicNetworkAccess: Disabled`.

---

## Part 3: Configure Environment-Specific Settings (15 min)

### Step 3.1 — Set Configuration Values for Dev

```bash
APP_CONFIG_DEV="appconfig-agent-dev"

az appconfig kv set --name $APP_CONFIG_DEV \
  --key "Agent:ModelDeployment" --value "gpt-4o-mini-dev" \
  --label "dev" --yes

az appconfig kv set --name $APP_CONFIG_DEV \
  --key "Agent:MaxTokens" --value "2048" \
  --label "dev" --yes

az appconfig kv set --name $APP_CONFIG_DEV \
  --key "Agent:Temperature" --value "0.9" \
  --label "dev" --yes

az appconfig kv set --name $APP_CONFIG_DEV \
  --key "Logging:Level" --value "Debug" \
  --label "dev" --yes

az appconfig kv set --name $APP_CONFIG_DEV \
  --key "Safety:ContentFilter" --value "Low" \
  --label "dev" --yes
```

### Step 3.2 — Set Configuration Values for Prod

```bash
APP_CONFIG_PROD="appconfig-agent-prod"

az appconfig kv set --name $APP_CONFIG_PROD \
  --key "Agent:ModelDeployment" --value "gpt-4o-prod" \
  --label "prod" --yes

az appconfig kv set --name $APP_CONFIG_PROD \
  --key "Agent:MaxTokens" --value "4096" \
  --label "prod" --yes

az appconfig kv set --name $APP_CONFIG_PROD \
  --key "Agent:Temperature" --value "0.7" \
  --label "prod" --yes

az appconfig kv set --name $APP_CONFIG_PROD \
  --key "Logging:Level" --value "Warning" \
  --label "prod" --yes

az appconfig kv set --name $APP_CONFIG_PROD \
  --key "Safety:ContentFilter" --value "High" \
  --label "prod" --yes
```

### Step 3.3 — Verify Configuration

```bash
# List dev settings
az appconfig kv list --name $APP_CONFIG_DEV --label "dev" --output table

# List prod settings
az appconfig kv list --name $APP_CONFIG_PROD --label "prod" --output table
```

### ✅ Checkpoint 3

Confirm that both App Configuration stores have 5 keys each, with different values (e.g., Dev uses `gpt-4o-mini-dev`, Prod uses `gpt-4o-prod`).

---

## Part 4: Implement the Health Check & Configuration Loader (20 min)

### Step 4.1 — Create the Configuration Loader

Create `src/config.py`:

```python
"""Environment-specific configuration loader."""
from azure.appconfiguration import AzureAppConfigurationClient
from azure.identity import DefaultAzureCredential
import os
import logging

logger = logging.getLogger(__name__)


def load_config(environment: str) -> dict:
    """Load configuration from Azure App Configuration by environment label."""
    endpoint = os.environ["APP_CONFIG_ENDPOINT"]
    client = AzureAppConfigurationClient(
        base_url=endpoint,
        credential=DefaultAzureCredential()
    )

    config = {}
    settings = client.list_configuration_settings(label_filter=environment)
    for setting in settings:
        config[setting.key] = setting.value
        logger.info(f"Loaded config: {setting.key} = {setting.value}")

    return config


# Load at startup
ENVIRONMENT = os.environ.get("ENVIRONMENT", "dev")
CONFIG = load_config(ENVIRONMENT)
```

### Step 4.2 — Create the Health Check Endpoint

Create `src/health.py`:

```python
"""Health check endpoint for load balancer integration."""
from fastapi import APIRouter
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
import os
import time
import json

router = APIRouter()


@router.get("/health")
async def health_check():
    """Check connectivity to AI Foundry and dependencies."""
    checks = {}
    environment = os.environ.get("ENVIRONMENT", "unknown")

    # Check AI Foundry connectivity
    try:
        client = AIProjectClient(
            credential=DefaultAzureCredential(),
            project_connection_string=os.environ["PROJECT_CONNECTION"]
        )
        start = time.time()
        client.agents.list_agents(limit=1)
        latency_ms = round((time.time() - start) * 1000)
        checks["ai_foundry"] = {"status": "healthy", "latency_ms": latency_ms}
    except Exception as e:
        checks["ai_foundry"] = {"status": "unhealthy", "error": str(e)}

    # Check App Configuration connectivity
    try:
        from azure.appconfiguration import AzureAppConfigurationClient
        app_config_client = AzureAppConfigurationClient(
            base_url=os.environ["APP_CONFIG_ENDPOINT"],
            credential=DefaultAzureCredential()
        )
        start = time.time()
        list(app_config_client.list_configuration_settings(label_filter=environment))
        latency_ms = round((time.time() - start) * 1000)
        checks["app_config"] = {"status": "healthy", "latency_ms": latency_ms}
    except Exception as e:
        checks["app_config"] = {"status": "unhealthy", "error": str(e)}

    all_healthy = all(c["status"] == "healthy" for c in checks.values())
    status_code = 200 if all_healthy else 503

    return {
        "status": "healthy" if all_healthy else "unhealthy",
        "environment": environment,
        "checks": checks
    }
```

### Step 4.3 — Update the Main Application

Create `src/app.py`:

```python
"""Main application entry point."""
from fastapi import FastAPI
from src.health import router as health_router
from src.config import CONFIG, ENVIRONMENT
import logging

# Configure logging based on environment
log_level = CONFIG.get("Logging:Level", "Info").upper()
logging.basicConfig(level=getattr(logging, log_level, logging.INFO))
logger = logging.getLogger(__name__)

app = FastAPI(
    title="AI Agent",
    description=f"Running in {ENVIRONMENT} environment"
)

# Register health router
app.include_router(health_router)


@app.get("/")
async def root():
    return {
        "agent": "AI Agent",
        "environment": ENVIRONMENT,
        "model_deployment": CONFIG.get("Agent:ModelDeployment", "unknown"),
        "status": "running"
    }


@app.get("/config")
async def show_config():
    """Display non-secret configuration (dev/staging only)."""
    if ENVIRONMENT == "prod":
        return {"message": "Config endpoint disabled in production"}
    return {"environment": ENVIRONMENT, "config": CONFIG}
```

### ✅ Checkpoint 4

Review your code and ensure:
- `config.py` loads settings filtered by the `ENVIRONMENT` env var
- `health.py` checks AI Foundry and App Configuration connectivity
- `app.py` exposes `/`, `/health`, and `/config` (config disabled in prod)

---

## Part 5: Deploy and Validate (15 min)

### Step 5.1 — Build the Container Image

```bash
# Build and push to ACR
ACR_NAME="acragent"

az acr build \
  --registry $ACR_NAME \
  --image ai-agent:module29 \
  .
```

### Step 5.2 — Deploy to Dev Environment

```bash
az containerapp update \
  --name agent-dev \
  --resource-group rg-ai-dev \
  --image $ACR_NAME.azurecr.io/ai-agent:module29 \
  --set-env-vars \
    ENVIRONMENT=dev \
    APP_CONFIG_ENDPOINT=https://appconfig-agent-dev.azconfig.io \
    PROJECT_CONNECTION="<dev-project-connection-string>"
```

### Step 5.3 — Deploy to Prod Environment

```bash
az containerapp update \
  --name agent-prod \
  --resource-group rg-ai-prod \
  --image $ACR_NAME.azurecr.io/ai-agent:module29 \
  --set-env-vars \
    ENVIRONMENT=prod \
    APP_CONFIG_ENDPOINT=https://appconfig-agent-prod.azconfig.io \
    PROJECT_CONNECTION="<prod-project-connection-string>"
```

### Step 5.4 — Validate Dev Environment

```bash
# Check health
curl -s https://agent-dev.azurecontainerapps.io/health | python -m json.tool

# Check root endpoint
curl -s https://agent-dev.azurecontainerapps.io/ | python -m json.tool

# Check config (only works in dev)
curl -s https://agent-dev.azurecontainerapps.io/config | python -m json.tool
```

**Expected output for `/` endpoint:**
```json
{
  "agent": "AI Agent",
  "environment": "dev",
  "model_deployment": "gpt-4o-mini-dev",
  "status": "running"
}
```

### Step 5.5 — Validate Prod Environment

```bash
# Check health
curl -s https://agent-prod.azurecontainerapps.io/health | python -m json.tool

# Check root endpoint
curl -s https://agent-prod.azurecontainerapps.io/ | python -m json.tool

# Config should be disabled
curl -s https://agent-prod.azurecontainerapps.io/config | python -m json.tool
```

**Expected output for `/` endpoint:**
```json
{
  "agent": "AI Agent",
  "environment": "prod",
  "model_deployment": "gpt-4o-prod",
  "status": "running"
}
```

### ✅ Checkpoint 5 — Final Validation

| Validation | Dev | Prod |
|------------|-----|------|
| `/health` returns 200 | ☐ | ☐ |
| `/` shows correct environment | ☐ | ☐ |
| Model deployment name is correct | ☐ | ☐ |
| `/config` returns settings | ☐ | N/A (disabled) |
| Same container image tag used | ☐ | ☐ |
| Health check reports all dependencies healthy | ☐ | ☐ |

---

## Bonus Challenge: Add CI/CD Promotion Pipeline (Optional, 20 min)

Create `.github/workflows/deploy-agent.yml` with the promotion pipeline from the blog content. Configure GitHub Environments for `development`, `staging`, and `production` with the appropriate approval rules.

---

## Cleanup

When you're finished, remove the lab resources to avoid ongoing charges:

```bash
# Delete Dev resources
az group delete --name rg-ai-dev --yes --no-wait

# Delete Prod resources
az group delete --name rg-ai-prod --yes --no-wait
```

> **Note:** If you plan to continue to Module 30 (Monitoring & Observability), keep these environments running — we'll add telemetry and dashboards in the next lab.

---

## Summary

In this lab, you:

1. **Created parameterized Bicep templates** for multi-environment infrastructure
2. **Deployed separate AI Foundry projects** for Dev and Prod
3. **Configured Azure App Configuration** with environment-labeled settings
4. **Implemented a health check endpoint** for load balancer integration
5. **Deployed the same container image** to both environments with different configurations
6. **Validated environment isolation** — different model deployments, logging levels, and safety configurations

These patterns form the foundation for production-grade AI deployments on Azure AI Foundry.
