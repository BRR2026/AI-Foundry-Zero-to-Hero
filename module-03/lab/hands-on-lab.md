# Module 3 Hands-On Lab: Exploring AI Foundry Architecture & Components

## Zero to Hero: Azure AI Foundry Training Course

**Arc 1: Foundations | Module 3 of 45**

**Estimated Duration: 90 minutes**

---

## 📋 Prerequisites

Before starting this lab, ensure you have:

- [x] An active Azure subscription with at least **Contributor** access
- [x] An AI Foundry Hub and at least one Project (created in Module 2)
- [x] Azure CLI installed (`az --version` should return 2.60+)
- [x] Python 3.10+ with pip
- [x] A web browser for Azure Portal access

## 🎯 Lab Objectives

By the end of this lab, you will have:

1. Explored the AI Foundry Hub and Project hierarchy in the Azure Portal
2. Identified and examined all underlying Azure resources
3. Reviewed RBAC roles and access configuration
4. Deployed the same model using two different deployment types
5. Connected an Azure Blob Storage data source to your project

---

## Exercise 1: Explore Your AI Foundry Hub and Project in the Azure Portal

**Duration: 15 minutes**

### Step 1.1: Navigate to AI Foundry in the Azure Portal

1. Open your browser and navigate to [https://portal.azure.com](https://portal.azure.com)
2. Sign in with your Azure credentials
3. In the search bar at the top, type **"AI Foundry"** and select **Azure AI Foundry** from the results

> 💡 **Tip:** You can also access AI Foundry directly at [https://ai.azure.com](https://ai.azure.com)

### Step 1.2: Examine the Hub

1. Click on your Hub resource (e.g., `hub-contoso-ai`)
2. On the **Overview** page, observe:
   - **Resource ID**: Note the full ARM resource path
   - **Location**: The Azure region where the Hub is deployed
   - **Kind**: Should show `Hub`
   - **Associated resources**: Storage Account, Key Vault, Application Insights, Container Registry

3. Record the following information:

| Property | Your Value |
|----------|-----------|
| Hub Name | |
| Resource Group | |
| Location | |
| Subscription ID | |
| Storage Account Name | |
| Key Vault Name | |

### Step 1.3: Examine the Project

1. In the Hub's left navigation, click **Projects** (or navigate directly to your Project)
2. Click on your project (e.g., `proj-chatbot-dev`)
3. On the **Overview** page, observe:
   - **Kind**: Should show `Project`
   - **Hub**: Shows the parent Hub name
   - **Project connection string**: Used for SDK authentication

4. Record:

| Property | Your Value |
|----------|-----------|
| Project Name | |
| Parent Hub | |
| Discovery URL | |

### Step 1.4: Verify the Hierarchy via Azure CLI

Open a terminal and run:

```bash
# Login to Azure (if not already logged in)
az login

# Set your subscription
az account set --subscription "YOUR_SUBSCRIPTION_ID"

# List all workspaces in your resource group
az ml workspace list \
  --resource-group YOUR_RESOURCE_GROUP \
  --output table

# Show Hub details
az ml workspace show \
  --name YOUR_HUB_NAME \
  --resource-group YOUR_RESOURCE_GROUP \
  --output json
```

**Expected Output:**
```
Name              ResourceGroup      Location    Kind
----------------  -----------------  ----------  -------
hub-contoso-ai    rg-ai-foundry      eastus      Hub
proj-chatbot      rg-ai-foundry      eastus      Project
```

### ✅ Exercise 1 Checkpoint

- [ ] I can see my Hub in the Azure Portal with its properties
- [ ] I can see my Project as a child of the Hub
- [ ] I've confirmed the hierarchy using Azure CLI
- [ ] I've recorded all resource names for later exercises

---

## Exercise 2: Examine the Underlying Azure Resources

**Duration: 20 minutes**

### Step 2.1: List All Resources in the Resource Group

1. In the Azure Portal, navigate to your **Resource Group**
2. Click **Resources** in the left menu (or view the main pane)
3. You should see resources similar to:

```
Expected Resources:
━━━━━━━━━━━━━━━━━━

Type                              Name                    
─────────────────────────────     ────────────────────────
Machine Learning workspace       hub-contoso-ai (Hub)     
Machine Learning workspace       proj-chatbot (Project)   
Storage account                  stcontosoai...           
Key vault                        kv-contoso-ai            
Application Insights             appi-contoso-ai          
Container registry               crcontosoai              
Log Analytics workspace          log-contoso-ai           
```

> ⚠️ **Note:** Your resource names will differ. The key is identifying each resource type.

### Step 2.2: Explore the Storage Account

1. Click on the **Storage Account** resource
2. Navigate to **Containers** (under Data storage)
3. Explore the blob containers:

```
Expected containers:
├── azureml-blobstore-{guid}     ← Default datastore
├── code-{hash}                   ← Code artifacts
└── Other containers...
```

4. Click into the `azureml-blobstore-{guid}` container to see the folder structure
5. Note: This is where your project data, models, and logs are stored

> 📝 **Document:** Write down 2-3 container names you find and their apparent purpose.

### Step 2.3: Explore the Key Vault

1. Navigate back to the Resource Group and click the **Key Vault** resource
2. Click **Secrets** in the left navigation
3. You should see secrets that were automatically created for connections

> ⚠️ **Security Note:** Do not share or screenshot secret values. Simply note that secrets exist.

4. Check **Access policies** or **Access configuration** to see how RBAC is configured

### Step 2.4: Explore Application Insights

1. Navigate to the **Application Insights** resource
2. Click **Overview** to see the dashboard
3. Note the **Instrumentation Key** and **Connection String** — these are used for telemetry
4. Click **Logs** in the left menu to open the query editor
5. Try this KQL query:

```kusto
// Check for any recent traces
traces
| where timestamp > ago(24h)
| take 10
```

### Step 2.5: Verify via Azure CLI

```bash
# List all resources in the resource group
az resource list \
  --resource-group YOUR_RESOURCE_GROUP \
  --output table \
  --query "[].{Name:name, Type:type, Location:location}"

# Show the Storage Account linked to the Hub
az ml workspace show \
  --name YOUR_HUB_NAME \
  --resource-group YOUR_RESOURCE_GROUP \
  --query "storageAccount" \
  --output tsv

# Show the Key Vault linked to the Hub
az ml workspace show \
  --name YOUR_HUB_NAME \
  --resource-group YOUR_RESOURCE_GROUP \
  --query "keyVault" \
  --output tsv
```

### ✅ Exercise 2 Checkpoint

- [ ] I can identify all underlying resources in my resource group
- [ ] I've explored the Storage Account containers and understand the structure
- [ ] I've verified secrets exist in Key Vault
- [ ] I've accessed Application Insights and queried logs
- [ ] I can retrieve linked resource IDs via Azure CLI

---

## Exercise 3: Review RBAC Roles and Access Settings

**Duration: 15 minutes**

### Step 3.1: View Current Role Assignments on the Hub

1. In the Azure Portal, navigate to your **AI Foundry Hub** resource
2. Click **Access control (IAM)** in the left menu
3. Click **Role assignments** tab
4. Observe the current assignments:

| Principal | Role | Scope |
|-----------|------|-------|
| (Your account) | ? | ? |
| (System-assigned MI) | ? | ? |

5. Record the roles assigned to your account

### Step 3.2: Explore Available Roles

1. Still in **Access control (IAM)**, click **Roles** tab
2. Search for **"AI"** in the search box
3. You should see roles like:
   - Azure AI Developer
   - Azure AI Inference Deployment Operator
   - Cognitive Services OpenAI User
   - Cognitive Services OpenAI Contributor

4. Click on **Azure AI Developer** to see its permissions
5. Click **Permissions** to see the detailed list

### Step 3.3: Check Managed Identity

1. Navigate to your Hub resource
2. Click **Identity** in the left menu
3. Observe:
   - **System-assigned:** Should be **On** — note the Object ID
   - **User-assigned:** Check if any user-assigned identities are attached

4. Note the System-assigned identity's Object ID:

```
System-assigned MI Object ID: ________________________________
```

### Step 3.4: Verify Role Assignments via CLI

```bash
# List role assignments at the Hub scope
az role assignment list \
  --scope "/subscriptions/YOUR_SUB/resourceGroups/YOUR_RG/providers/Microsoft.MachineLearningServices/workspaces/YOUR_HUB" \
  --output table

# List your own role assignments
az role assignment list \
  --assignee YOUR_EMAIL@domain.com \
  --output table

# Check the Hub's Managed Identity assignments
az role assignment list \
  --assignee YOUR_MANAGED_IDENTITY_OBJECT_ID \
  --output table
```

### Step 3.5: Simulate Access Restriction (Read-Only)

> ⚠️ **Warning:** Only do this if you have Owner access and can undo it. If in a shared environment, skip this step.

1. Understanding the principle: If you assigned **Reader** role to a colleague on the Hub, they would:
   - ✅ See the Hub and Projects in the portal
   - ✅ View model deployments and connections
   - ❌ Cannot deploy models or create connections
   - ❌ Cannot run prompt flows or evaluations

> 📝 **Document:** Write down which role you would assign to: (a) a data scientist who needs to experiment, (b) an auditor who needs to review configurations, (c) a DevOps engineer who manages infrastructure.

### ✅ Exercise 3 Checkpoint

- [ ] I can view role assignments on my Hub
- [ ] I can identify the difference between AI-specific roles
- [ ] I've confirmed the Managed Identity is enabled
- [ ] I understand which roles to assign for different personas

---

## Exercise 4: Compare Deployment Types

**Duration: 25 minutes**

### Step 4.1: Deploy a Model as Standard (Pay-per-token)

1. Navigate to [https://ai.azure.com](https://ai.azure.com)
2. Select your Project
3. Click **Deployments** in the left navigation
4. Click **+ Create deployment** → **Deploy base model**
5. Select **gpt-4o-mini** (cost-effective for this lab)
6. Configure:
   - **Deployment name:** `gpt4omini-standard`
   - **Deployment type:** **Standard**
   - **Tokens per minute rate limit:** 10K (minimum for testing)
7. Click **Deploy**

> ⏳ Wait for deployment to complete (usually 1-2 minutes)

8. Once deployed, click on the deployment and note:
   - **Endpoint URL**: The URL to call for inference
   - **API version**: The API version used
   - **Rate limits**: Tokens per minute

### Step 4.2: Deploy the Same Model as Global Standard

1. Click **+ Create deployment** → **Deploy base model** again
2. Select **gpt-4o-mini** again
3. Configure:
   - **Deployment name:** `gpt4omini-global`
   - **Deployment type:** **Global Standard**
   - **Tokens per minute rate limit:** 10K
4. Click **Deploy**

### Step 4.3: Compare the Two Deployments

Use the Playground or API to test both:

**Via the AI Foundry Playground:**

1. Go to **Playground** → **Chat**
2. Select deployment `gpt4omini-standard`
3. Send the message: `"Explain the difference between Standard and Global Standard deployments in one paragraph."`
4. Note the response time
5. Switch to deployment `gpt4omini-global`
6. Send the same message
7. Note the response time

**Via Azure CLI / curl:**

```bash
# Test Standard deployment
curl -X POST "https://YOUR_OPENAI_RESOURCE.openai.azure.com/openai/deployments/gpt4omini-standard/chat/completions?api-version=2024-02-01" \
  -H "Content-Type: application/json" \
  -H "api-key: YOUR_API_KEY" \
  -d '{
    "messages": [{"role": "user", "content": "Hello, which deployment type am I hitting?"}],
    "max_tokens": 100
  }'

# Test Global Standard deployment
curl -X POST "https://YOUR_OPENAI_RESOURCE.openai.azure.com/openai/deployments/gpt4omini-global/chat/completions?api-version=2024-02-01" \
  -H "Content-Type: application/json" \
  -H "api-key: YOUR_API_KEY" \
  -d '{
    "messages": [{"role": "user", "content": "Hello, which deployment type am I hitting?"}],
    "max_tokens": 100
  }'
```

### Step 4.4: Document Your Comparison

Fill in this table based on your observations:

| Property | Standard | Global Standard |
|----------|----------|----------------|
| Deployment Name | gpt4omini-standard | gpt4omini-global |
| Response Time (approx) | ___ms | ___ms |
| Data Residency | Regional | Global |
| Endpoint URL | | |
| Cost per 1K tokens (input) | | |

### Step 4.5: Clean Up (Optional)

If you want to minimize costs, delete the test deployments:

```bash
az cognitiveservices account deployment delete \
  --name YOUR_OPENAI_RESOURCE \
  --resource-group YOUR_RESOURCE_GROUP \
  --deployment-name gpt4omini-standard

az cognitiveservices account deployment delete \
  --name YOUR_OPENAI_RESOURCE \
  --resource-group YOUR_RESOURCE_GROUP \
  --deployment-name gpt4omini-global
```

### ✅ Exercise 4 Checkpoint

- [ ] I've deployed gpt-4o-mini as Standard
- [ ] I've deployed gpt-4o-mini as Global Standard
- [ ] I've tested both deployments and compared response times
- [ ] I understand the key differences between deployment types

---

## Exercise 5: Connect a Data Source (Azure Blob Storage)

**Duration: 15 minutes**

### Step 5.1: Create a Test Storage Container

If you don't already have test data, create a container and upload a sample file:

```bash
# Get your Hub's Storage Account name
STORAGE_NAME=$(az ml workspace show \
  --name YOUR_HUB_NAME \
  --resource-group YOUR_RESOURCE_GROUP \
  --query "storageAccount" \
  --output tsv | sed 's|.*/||')

echo "Storage Account: $STORAGE_NAME"

# Create a test container
az storage container create \
  --name "training-data" \
  --account-name $STORAGE_NAME \
  --auth-mode login

# Create a sample test file locally
echo "Azure AI Foundry is a comprehensive platform for building AI applications." > sample-data.txt
echo "It provides tools for model deployment, prompt engineering, and evaluation." >> sample-data.txt
echo "The platform supports both Azure OpenAI models and open-source models." >> sample-data.txt

# Upload the sample file
az storage blob upload \
  --container-name "training-data" \
  --file sample-data.txt \
  --name "module-03/sample-data.txt" \
  --account-name $STORAGE_NAME \
  --auth-mode login
```

### Step 5.2: Create a Connection in AI Foundry Portal

1. Navigate to [https://ai.azure.com](https://ai.azure.com)
2. Select your **Project**
3. Go to **Management** → **Connected resources** (or **Settings** → **Connections**)
4. Click **+ New connection**
5. Select **Azure Blob Storage**
6. Configure:
   - **Name:** `blob-training-data`
   - **Storage Account:** Select the storage account linked to your Hub
   - **Container:** `training-data`
   - **Authentication:** Managed Identity (recommended) or Account Key
7. Click **Create**

### Step 5.3: Verify the Connection via Python SDK

Install the required packages (if not already installed):

```bash
pip install azure-ai-ml azure-identity
```

Create and run this Python script:

```python
# verify_connection.py
from azure.ai.ml import MLClient
from azure.identity import DefaultAzureCredential

# Replace with your values
subscription_id = "YOUR_SUBSCRIPTION_ID"
resource_group = "YOUR_RESOURCE_GROUP"
project_name = "YOUR_PROJECT_NAME"

# Create the client
ml_client = MLClient(
    credential=DefaultAzureCredential(),
    subscription_id=subscription_id,
    resource_group_name=resource_group,
    workspace_name=project_name
)

# List all connections
print("=" * 50)
print("CONNECTIONS IN PROJECT")
print("=" * 50)

connections = ml_client.connections.list()
for conn in connections:
    print(f"\nName: {conn.name}")
    print(f"  Type: {conn.type}")
    print(f"  Target: {conn.target if hasattr(conn, 'target') else 'N/A'}")
    print(f"  Created: {conn.created_date if hasattr(conn, 'created_date') else 'N/A'}")

print("\n" + "=" * 50)
print("CONNECTION DETAILS: blob-training-data")
print("=" * 50)

try:
    conn = ml_client.connections.get("blob-training-data")
    print(f"Name: {conn.name}")
    print(f"Type: {conn.type}")
    print(f"Target: {conn.target}")
except Exception as e:
    print(f"Connection not found or error: {e}")
```

Run the script:
```bash
python verify_connection.py
```

**Expected Output:**
```
==================================================
CONNECTIONS IN PROJECT
==================================================

Name: blob-training-data
  Type: azure_blob
  Target: https://stcontosoai.blob.core.windows.net
  Created: 2026-01-15

Name: my-openai-connection
  Type: azure_open_ai
  Target: https://oai-contoso.openai.azure.com/

==================================================
CONNECTION DETAILS: blob-training-data
==================================================
Name: blob-training-data
Type: azure_blob
Target: https://stcontosoai.blob.core.windows.net
```

### Step 5.4: Access Data Through the Connection

```python
# access_data.py
from azure.ai.ml import MLClient
from azure.identity import DefaultAzureCredential
from azure.storage.blob import BlobServiceClient

# Create ML client
ml_client = MLClient(
    credential=DefaultAzureCredential(),
    subscription_id="YOUR_SUBSCRIPTION_ID",
    resource_group_name="YOUR_RESOURCE_GROUP",
    workspace_name="YOUR_PROJECT_NAME"
)

# Get the connection details
conn = ml_client.connections.get("blob-training-data")
print(f"Connected to: {conn.target}")

# Use DefaultAzureCredential to access the blob
blob_client = BlobServiceClient(
    account_url=conn.target,
    credential=DefaultAzureCredential()
)

# List blobs in the training-data container
container_client = blob_client.get_container_client("training-data")
print("\nBlobs in 'training-data' container:")
for blob in container_client.list_blobs():
    print(f"  📄 {blob.name} ({blob.size} bytes)")

# Download and read the sample file
blob = container_client.download_blob("module-03/sample-data.txt")
content = blob.readall().decode('utf-8')
print(f"\nContent of sample-data.txt:")
print(content)
```

### Step 5.5: Clean Up

```bash
# Remove the sample file (optional)
rm sample-data.txt
```

> 📝 **Note:** Keep the connection and storage container — you'll use them in future labs.

### ✅ Exercise 5 Checkpoint

- [ ] I've created a storage container with test data
- [ ] I've created a Blob Storage connection in AI Foundry
- [ ] I've verified the connection using the Python SDK
- [ ] I've accessed data through the connection programmatically

---

## 🎉 Lab Complete!

### What You Accomplished

| Exercise | Skill Practiced |
|----------|----------------|
| 1 | Navigating the AI Foundry resource hierarchy |
| 2 | Identifying and understanding underlying Azure resources |
| 3 | Understanding RBAC roles and Managed Identity |
| 4 | Comparing deployment types (Standard vs Global Standard) |
| 5 | Creating and using data source connections |

### Key Learnings

1. **The Hub is the foundation** — it owns the shared infrastructure that all projects depend on
2. **Underlying resources matter** — understanding Storage, Key Vault, and App Insights helps with troubleshooting and cost management
3. **RBAC is granular** — use the principle of least privilege with AI-specific roles
4. **Deployment types have tradeoffs** — choose based on data residency, cost, and performance needs
5. **Connections are the glue** — they securely bridge your AI projects to external data

### 🔗 Resources for Further Exploration

- [AI Foundry Architecture Documentation](https://learn.microsoft.com/en-us/azure/ai-studio/concepts/architecture)
- [RBAC Roles for AI Foundry](https://learn.microsoft.com/en-us/azure/ai-studio/concepts/rbac-ai-studio)
- [Azure OpenAI Deployment Types](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/deployment-types)
- [AI Foundry Networking](https://learn.microsoft.com/en-us/azure/ai-studio/how-to/configure-private-link)

---

**Next Lab:** [Module 4 — Working with Models in AI Foundry →](../../module-04/lab/hands-on-lab.md)

---

*© 2026 Zero to Hero Training Team. Azure AI Foundry Training Course.*
