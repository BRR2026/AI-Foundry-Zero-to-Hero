# Hands-On Lab: Deploy a Model with Blue/Green Traffic Splitting

> **Module 26** · Azure AI Foundry: Zero to Hero · Arc 6: Deployment & MLOps

---

## Lab Overview

| Field | Details |
|---|---|
| **Objective** | Deploy two model versions behind a single Azure AI Foundry endpoint and implement phased blue/green traffic splitting |
| **Duration** | 30–40 minutes |
| **Level** | Intermediate |
| **Prerequisites** | Azure subscription, Azure AI Foundry project, Python 3.10+, `azure-ai-projects` SDK |

## Learning Outcomes

By completing this lab, you will:

- Deploy multiple model versions to a single Azure AI Foundry endpoint
- Configure traffic splitting between blue (stable) and green (new) deployments
- Validate endpoint responses from both deployments
- Perform a phased traffic cutover (90/10 → 50/50 → 100% green)
- Execute an instant rollback to the blue deployment

---

## Prerequisites Setup

### 1. Install Required Packages

```bash
pip install azure-ai-projects azure-identity openai requests
```

### 2. Verify Azure Authentication

```bash
az login
az account set --subscription "<your-subscription-id>"
```

### 3. Confirm Azure AI Foundry Project

Ensure you have an existing Azure AI Foundry project. Note your:
- **Subscription ID**
- **Resource Group Name**
- **Project Name**

---

## Exercise 1: Initialize the Project Client (5 min)

### Step 1.1: Create the Configuration

Create a file called `deploy_blue_green.py`:

```python
"""
Module 26 Lab: Blue/Green Deployment with Traffic Splitting
Azure AI Foundry: Zero to Hero
"""

from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient
import time
import json

# ──── Configuration ────
SUBSCRIPTION_ID = "<your-subscription-id>"
RESOURCE_GROUP = "<your-resource-group>"
PROJECT_NAME = "<your-project-name>"

MODEL_NAME = "gpt-4o"
BLUE_VERSION = "2024-08-06"
GREEN_VERSION = "2024-11-20"

BLUE_DEPLOYMENT = "gpt4o-blue"
GREEN_DEPLOYMENT = "gpt4o-green"
ENDPOINT_NAME = "gpt4o-endpoint"

SKU_NAME = "GlobalStandard"
SKU_CAPACITY = 30  # 30K tokens per minute
```

### Step 1.2: Initialize the Client

```python
# ──── Initialize Client ────
credential = DefaultAzureCredential()
project_client = AIProjectClient(
    credential=credential,
    subscription_id=SUBSCRIPTION_ID,
    resource_group_name=RESOURCE_GROUP,
    project_name=PROJECT_NAME,
)

print(f"✅ Connected to project: {PROJECT_NAME}")
print(f"   Resource Group: {RESOURCE_GROUP}")
```

### ✅ Checkpoint

Run the script. You should see a successful connection message.

---

## Exercise 2: Deploy the Blue (Stable) Model (10 min)

### Step 2.1: Create the Blue Deployment

```python
# ──── Deploy Blue (Current Stable Version) ────
print(f"\n🔵 Deploying BLUE: {MODEL_NAME} v{BLUE_VERSION}...")

blue_deployment = project_client.deployments.begin_create_or_update(
    name=BLUE_DEPLOYMENT,
    model=MODEL_NAME,
    model_version=BLUE_VERSION,
    sku_name=SKU_NAME,
    sku_capacity=SKU_CAPACITY,
).result()

print(f"✅ Blue deployment created!")
print(f"   Name:     {blue_deployment.name}")
print(f"   Model:    {MODEL_NAME} v{BLUE_VERSION}")
print(f"   Status:   {blue_deployment.properties.provisioning_state}")
```

### Step 2.2: Validate the Blue Deployment

```python
# ──── Validate Blue ────
from openai import AzureOpenAI

# Get the endpoint URL and key
endpoint_url = blue_deployment.properties.endpoint_url
keys = project_client.deployments.list_keys(name=BLUE_DEPLOYMENT)

client = AzureOpenAI(
    azure_endpoint=endpoint_url,
    api_key=keys.primary_key,
    api_version="2024-12-01-preview",
)

response = client.chat.completions.create(
    model=BLUE_DEPLOYMENT,
    messages=[
        {"role": "system", "content": "You are a helpful assistant. Reply briefly."},
        {"role": "user", "content": "What deployment am I using? Say: Blue deployment active."}
    ],
    max_tokens=50,
)

print(f"\n🔵 Blue Response: {response.choices[0].message.content}")
```

### ✅ Checkpoint

You should see a response from the blue deployment confirming it's active.

---

## Exercise 3: Deploy the Green (New) Model (5 min)

### Step 3.1: Create the Green Deployment

```python
# ──── Deploy Green (New Version) ────
print(f"\n🟢 Deploying GREEN: {MODEL_NAME} v{GREEN_VERSION}...")

green_deployment = project_client.deployments.begin_create_or_update(
    name=GREEN_DEPLOYMENT,
    model=MODEL_NAME,
    model_version=GREEN_VERSION,
    sku_name=SKU_NAME,
    sku_capacity=SKU_CAPACITY,
).result()

print(f"✅ Green deployment created!")
print(f"   Name:     {green_deployment.name}")
print(f"   Model:    {MODEL_NAME} v{GREEN_VERSION}")
print(f"   Status:   {green_deployment.properties.provisioning_state}")
```

### Step 3.2: Validate the Green Deployment

```python
# ──── Validate Green ────
green_keys = project_client.deployments.list_keys(name=GREEN_DEPLOYMENT)

green_client = AzureOpenAI(
    azure_endpoint=green_deployment.properties.endpoint_url,
    api_key=green_keys.primary_key,
    api_version="2024-12-01-preview",
)

response = green_client.chat.completions.create(
    model=GREEN_DEPLOYMENT,
    messages=[
        {"role": "system", "content": "You are a helpful assistant. Reply briefly."},
        {"role": "user", "content": "What deployment am I using? Say: Green deployment active."}
    ],
    max_tokens=50,
)

print(f"🟢 Green Response: {response.choices[0].message.content}")
```

### ✅ Checkpoint

Both blue and green deployments should be active and responding.

---

## Exercise 4: Configure Traffic Splitting (10 min)

### Step 4.1: Set Initial 90/10 Split (Canary)

```python
# ──── Phase 1: Canary — 90% Blue / 10% Green ────
print("\n📊 Phase 1: Setting traffic split to 90/10 (canary)...")

endpoint = project_client.endpoints.begin_create_or_update(
    name=ENDPOINT_NAME,
    traffic={
        BLUE_DEPLOYMENT: 90,
        GREEN_DEPLOYMENT: 10,
    },
).result()

print(f"✅ Traffic split configured:")
print(f"   🔵 Blue:  90%")
print(f"   🟢 Green: 10%")
print(f"   Endpoint: {endpoint.properties.scoring_uri}")
```

### Step 4.2: Test the Traffic Distribution

```python
# ──── Validate Traffic Distribution ────
import requests

endpoint_url = endpoint.properties.scoring_uri
api_key = keys.primary_key

blue_count = 0
green_count = 0
total_requests = 20

print(f"\n🧪 Sending {total_requests} requests to test traffic distribution...")

for i in range(total_requests):
    resp = requests.post(
        endpoint_url,
        headers={
            "Content-Type": "application/json",
            "api-key": api_key,
        },
        json={
            "messages": [
                {"role": "user", "content": "Which model version are you? Reply with just the version number."}
            ],
            "max_tokens": 20,
        },
    )
    
    result = resp.json()
    model_used = result.get("model", "unknown")
    
    if "blue" in model_used.lower() or BLUE_VERSION in str(result):
        blue_count += 1
    else:
        green_count += 1
    
    time.sleep(0.5)

print(f"\n📊 Traffic Distribution Results ({total_requests} requests):")
print(f"   🔵 Blue:  {blue_count} ({blue_count/total_requests*100:.0f}%)")
print(f"   🟢 Green: {green_count} ({green_count/total_requests*100:.0f}%)")
```

### Step 4.3: Shift to 50/50

```python
# ──── Phase 2: Validation — 50/50 Split ────
print("\n📊 Phase 2: Shifting traffic to 50/50...")

project_client.endpoints.begin_create_or_update(
    name=ENDPOINT_NAME,
    traffic={
        BLUE_DEPLOYMENT: 50,
        GREEN_DEPLOYMENT: 50,
    },
).result()

print("✅ Traffic split updated: 50% Blue / 50% Green")
print("⏳ Monitor metrics for 5-10 minutes before proceeding...")
```

### Step 4.4: Full Cutover to Green

```python
# ──── Phase 3: Full Cutover — 100% Green ────
print("\n📊 Phase 3: Full cutover to Green...")

project_client.endpoints.begin_create_or_update(
    name=ENDPOINT_NAME,
    traffic={
        BLUE_DEPLOYMENT: 0,
        GREEN_DEPLOYMENT: 100,
    },
).result()

print("✅ Full cutover complete: 100% Green")
print("🟢 All traffic is now routed to the new model version")
```

### ✅ Checkpoint

You've successfully performed a phased traffic cutover from blue to green.

---

## Exercise 5: Rollback (5 min)

### Step 5.1: Instant Rollback to Blue

```python
# ──── ROLLBACK: Shift 100% back to Blue ────
print("\n⚠️ Performing rollback...")

project_client.endpoints.begin_create_or_update(
    name=ENDPOINT_NAME,
    traffic={
        BLUE_DEPLOYMENT: 100,
        GREEN_DEPLOYMENT: 0,
    },
).result()

print("✅ ROLLBACK COMPLETE")
print("🔵 All traffic restored to Blue (stable) deployment")
print("🟢 Green deployment is still available but receiving 0% traffic")
```

### Step 5.2: Verify Rollback

```python
# ──── Verify rollback ────
response = requests.post(
    endpoint_url,
    headers={
        "Content-Type": "application/json",
        "api-key": api_key,
    },
    json={
        "messages": [
            {"role": "user", "content": "Confirm you are responding. Say 'Rollback verified.'"}
        ],
        "max_tokens": 20,
    },
)

print(f"\n🔵 Post-Rollback Response: {response.json()['choices'][0]['message']['content']}")
```

### ✅ Checkpoint

All traffic should be routed back to the blue deployment.

---

## Exercise 6: Cleanup (5 min)

### Step 6.1: Delete Deployments

```python
# ──── Cleanup ────
print("\n🧹 Cleaning up deployments...")

project_client.deployments.begin_delete(name=GREEN_DEPLOYMENT).result()
print(f"   ✅ Deleted: {GREEN_DEPLOYMENT}")

project_client.deployments.begin_delete(name=BLUE_DEPLOYMENT).result()
print(f"   ✅ Deleted: {BLUE_DEPLOYMENT}")

print("\n🎉 Lab complete! All resources cleaned up.")
```

> **⚠️ Cost Note:** Always delete deployments when you're done to avoid ongoing charges. Serverless deployments incur costs only when tokens are consumed, but managed compute and PTU deployments are billed continuously.

---

## Success Criteria

| # | Criterion | Status |
|---|-----------|--------|
| 1 | Blue deployment created and serving requests | ☐ |
| 2 | Green deployment created with a different model version | ☐ |
| 3 | 90/10 traffic split configured and validated | ☐ |
| 4 | Traffic shifted to 50/50, then 100% green | ☐ |
| 5 | Rollback performed — 100% traffic restored to blue | ☐ |
| 6 | Cleanup completed — all deployments deleted | ☐ |

---

## Bonus Challenges

### Challenge 1: Automated Rollback Script
Write a monitoring script that checks error rates every 60 seconds and automatically rolls back to blue if the error rate exceeds 5%.

### Challenge 2: Entra ID Authentication
Modify the lab to use Microsoft Entra ID (managed identity) instead of API keys for all endpoint calls.

### Challenge 3: PTU Deployment
Repeat the lab using `ProvisionedManaged` SKU instead of `GlobalStandard`. Compare latency metrics between the two deployment types.

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `AuthenticationError` | Run `az login` and verify your subscription is set correctly |
| Deployment stuck in `Creating` | Wait 5–10 minutes; large models take time to provision |
| `QuotaExceeded` | Reduce `sku_capacity` or request a quota increase in the portal |
| Traffic split not taking effect | Verify percentages sum to 100; wait 30 seconds for propagation |
| `ModelNotFound` | Verify the model name and version are available in your region |

---

## Summary

In this lab, you performed a complete blue/green deployment lifecycle:

1. **Created** two model deployments (blue = stable, green = new version)
2. **Configured** phased traffic splitting (90/10 → 50/50 → 100%)
3. **Validated** endpoint responses at each phase
4. **Executed** an instant rollback to the stable deployment
5. **Cleaned up** all resources to avoid unnecessary costs

This pattern is the foundation for safe, zero-downtime model upgrades in production Azure AI Foundry environments.

---

*Module 26 Lab · Azure AI Foundry: Zero to Hero · Arc 6: Deployment & MLOps*
