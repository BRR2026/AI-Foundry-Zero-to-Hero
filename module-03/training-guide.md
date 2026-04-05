# Module 3: AI Foundry Architecture & Components

## Zero to Hero: Azure AI Foundry Training Course

**Arc 1: Foundations | Module 3 of 45**

---

## 📋 Table of Contents

1. [Introduction & Learning Objectives](#1-introduction--learning-objectives)
2. [Resource Hierarchy](#2-resource-hierarchy)
3. [Hub vs Project: Deep Dive](#3-hub-vs-project-deep-dive)
4. [Underlying Azure Resources](#4-underlying-azure-resources)
5. [Networking Architecture](#5-networking-architecture)
6. [Authentication & Identity](#6-authentication--identity)
7. [Model Hosting Options](#7-model-hosting-options)
8. [Connections & Data Integration](#8-connections--data-integration)
9. [Compute Resources](#9-compute-resources)
10. [SDK & API Architecture](#10-sdk--api-architecture)
11. [Region Availability & Data Residency](#11-region-availability--data-residency)
12. [Cost Structure](#12-cost-structure)
13. [Architecture Diagrams](#13-architecture-diagrams)
14. [Real-World Architecture Examples](#14-real-world-architecture-examples)
15. [Summary & Key Takeaways](#15-summary--key-takeaways)
16. [Glossary](#16-glossary)

---

## 1. Introduction & Learning Objectives

Welcome to Module 3! In the first two modules, you got oriented with Azure AI and set up your first environment. Now we pull back the curtain and examine **how AI Foundry is actually built** — the resource hierarchy, the networking, the identity model, and the deployment options that determine how your AI solutions behave in production.

### 🎯 Learning Objectives

By the end of this module, you will be able to:

- Diagram the complete AI Foundry resource hierarchy from Subscription down to individual model deployments
- Explain the difference between a Hub and a Project and articulate when to use multiple of each
- Identify all underlying Azure resources that AI Foundry provisions and manages
- Describe networking options including public access, private endpoints, and VNet integration
- Configure RBAC roles and Managed Identity for secure access
- Compare Managed Compute, Serverless API (MaaS), and Provisioned deployment types
- Connect AI Foundry projects to external data sources
- Estimate costs for different AI Foundry configurations

### 🧠 Why Architecture Matters

> **Analogy:** Think of AI Foundry like a modern office building. The **Hub** is the building itself — it has the lobby, security desk, shared conference rooms, and central utilities. **Projects** are the individual office suites inside the building — each team has their own space, but they share the building's infrastructure. Understanding the building's blueprint (architecture) helps you know where the plumbing runs, where the electrical panels are, and how to get from floor to floor.

---

## 2. Resource Hierarchy

### The Four-Level Hierarchy

AI Foundry follows Azure's standard resource management model and adds its own organizational layers on top.

```
┌─────────────────────────────────────────────────────────┐
│                   AZURE SUBSCRIPTION                     │
│                   (Billing boundary)                     │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │              RESOURCE GROUP                       │   │
│  │              (Lifecycle container)                │   │
│  │                                                   │   │
│  │  ┌───────────────────────────────────────────┐   │   │
│  │  │      AI FOUNDRY ACCOUNT (HUB)             │   │   │
│  │  │      (Shared infra & governance)          │   │   │
│  │  │                                            │   │   │
│  │  │  ┌─────────────┐  ┌─────────────┐        │   │   │
│  │  │  │  PROJECT A   │  │  PROJECT B   │        │   │   │
│  │  │  │  (Team/App)  │  │  (Team/App)  │        │   │   │
│  │  │  │             │  │             │        │   │   │
│  │  │  │ • Deployments│  │ • Deployments│        │   │   │
│  │  │  │ • Connections│  │ • Connections│        │   │   │
│  │  │  │ • Data       │  │ • Data       │        │   │   │
│  │  │  │ • Flows      │  │ • Flows      │        │   │   │
│  │  │  └─────────────┘  └─────────────┘        │   │   │
│  │  └───────────────────────────────────────────┘   │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### Level-by-Level Breakdown

| Level | Resource | Purpose | Key Properties |
|-------|----------|---------|----------------|
| **1** | **Subscription** | Billing boundary, top-level container | Cost management, quotas, service limits |
| **2** | **Resource Group** | Logical container for related resources | Lifecycle management, tagging, RBAC scope |
| **3** | **AI Foundry Account (Hub)** | Shared infrastructure and governance | Networking, storage, key vault, identity |
| **4** | **Project** | Workspace for a team or application | Model deployments, connections, data, experiments |

💡 **Key Insight:** The Hub is the ARM resource type `Microsoft.MachineLearningServices/workspaces` with kind `Hub`. Projects are the same resource type but with kind `Project`. Under the hood, AI Foundry builds on the Azure Machine Learning workspace architecture.

### Resource Naming Conventions

📝 **Best Practice:** Adopt a consistent naming convention early:

```
Subscription:    sub-contoso-ai-prod
Resource Group:  rg-ai-foundry-eastus-prod
Hub:             hub-contoso-ai-eastus
Project:         proj-chatbot-prod
                 proj-doc-search-dev
                 proj-content-gen-staging
```

---

## 3. Hub vs Project: Deep Dive

### What is a Hub?

The **Hub** (also called **AI Foundry Account**) is the central governance and infrastructure layer. Think of it as the "parent" that owns shared resources.

**A Hub contains and manages:**

| Component | Description |
|-----------|-------------|
| **Storage Account** | Shared blob storage for data, models, logs |
| **Key Vault** | Centralized secrets, keys, and certificates |
| **Application Insights** | Monitoring and telemetry for all projects |
| **Container Registry** | Shared registry for custom container images |
| **Networking Config** | Public/private endpoint settings, VNet rules |
| **Shared Connections** | Connections available to all child projects |
| **Managed Identity** | System-assigned identity for resource access |
| **Compute Instances** | Shared compute resources (optional) |

### What is a Project?

A **Project** is the working space where development happens. Each project inherits the Hub's shared infrastructure but has its own isolated workspace.

**A Project contains:**

| Component | Description |
|-----------|-------------|
| **Model Deployments** | OpenAI, OSS, or custom model endpoints |
| **Connections** | Project-specific data source connections |
| **Prompt Flows** | Orchestrated AI workflows |
| **Evaluations** | Model evaluation runs and results |
| **Data Assets** | Registered datasets and data references |
| **Indexes** | Vector search indexes for RAG patterns |
| **Fine-tuning Jobs** | Custom model training jobs |
| **Playground** | Interactive model testing interface |

### When to Use Multiple Hubs vs Multiple Projects

```
SINGLE HUB, MULTIPLE PROJECTS (Most Common)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ┌─── Hub: hub-contoso-ai ───────────────┐
 │                                        │
 │  proj-chatbot      (Customer Support) │
 │  proj-doc-search   (Internal Search)  │
 │  proj-content-gen  (Marketing)        │
 │                                        │
 └────────────────────────────────────────┘
 ✅ Use when: Same team, same region, shared governance

MULTIPLE HUBS (Advanced Scenarios)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ┌─── Hub: hub-us-east ──┐  ┌─── Hub: hub-eu-west ──┐
 │  proj-us-chatbot      │  │  proj-eu-chatbot       │
 │  proj-us-analytics    │  │  proj-eu-analytics     │
 └───────────────────────┘  └────────────────────────┘
 ✅ Use when: Data residency requirements, different regions,
              different networking policies, separate billing
```

### Decision Matrix: Hub vs Project Isolation

| Scenario | Solution | Rationale |
|----------|----------|-----------|
| Different teams, same org, same region | Multiple **Projects** under one Hub | Shared infra, separate workspaces |
| Different compliance domains (HIPAA, GDPR) | Multiple **Hubs** | Separate networking, separate Key Vault |
| Dev/Staging/Prod environments | Multiple **Projects** OR multiple **Hubs** | Projects for light isolation; Hubs for strict isolation |
| Multi-region deployment | Multiple **Hubs** (one per region) | Hubs are region-specific |
| Different billing departments | Multiple **Subscriptions** with separate Hubs | Subscription is the billing boundary |

⚠️ **Important:** A Project can only belong to one Hub. You cannot move a Project between Hubs after creation.

---

## 4. Underlying Azure Resources

When you create an AI Foundry Hub, Azure provisions several underlying resources automatically. Understanding these resources is critical for troubleshooting, cost management, and security.

### Resource Provisioning Diagram

```
   Creating a Hub automatically provisions:
   ─────────────────────────────────────────

   ┌────────────────────────────┐
   │    AI Foundry Hub          │
   │    (hub-contoso-ai)        │
   └──────────┬─────────────────┘
              │ automatically creates
              ├──────────────────────────────────────┐
              │                                       │
   ┌──────────▼──────────┐  ┌────────────────────┐  │
   │   Storage Account    │  │   Key Vault         │  │
   │   stcontosoai...     │  │   kv-contoso-ai     │  │
   │                      │  │                     │  │
   │ • Blob containers    │  │ • API keys          │  │
   │ • File shares        │  │ • Connection strings│  │
   │ • Model artifacts    │  │ • Certificates      │  │
   │ • Logs & metrics     │  │ • Managed secrets    │  │
   └──────────────────────┘  └─────────────────────┘  │
              │                                        │
   ┌──────────▼──────────┐  ┌─────────────────────┐  │
   │ Application Insights │  │  Container Registry  │  │
   │ appi-contoso-ai      │  │  crcontosoai         │  │
   │                      │  │                      │  │
   │ • Telemetry          │  │ • Custom images      │  │
   │ • Performance data   │  │ • Flow containers    │  │
   │ • Error tracking     │  │ • Fine-tuned models  │  │
   │ • Usage analytics    │  │                      │  │
   └──────────────────────┘  └──────────────────────┘  │
              │                                        │
   ┌──────────▼─────────────────┐                      │
   │  Azure OpenAI Service       │◄─────────────────────┘
   │  oai-contoso-ai             │  (connection, not always
   │                             │   auto-provisioned)
   │ • GPT-4o, GPT-4, GPT-3.5   │
   │ • Embeddings (text-embedding)│
   │ • DALL·E, Whisper           │
   └─────────────────────────────┘
```

### Detailed Resource Breakdown

| Azure Resource | Type | Purpose | Created By | Shared Across Projects? |
|---------------|------|---------|------------|------------------------|
| **Storage Account** | `Microsoft.Storage/storageAccounts` | Data, model artifacts, logs, prompt flow files | Hub (auto) | ✅ Yes |
| **Key Vault** | `Microsoft.KeyVault/vaults` | Secrets, API keys, connection strings | Hub (auto) | ✅ Yes |
| **Application Insights** | `Microsoft.Insights/components` | Telemetry, monitoring, diagnostics | Hub (auto) | ✅ Yes |
| **Container Registry** | `Microsoft.ContainerRegistry/registries` | Custom container images, environments | Hub (auto or manual) | ✅ Yes |
| **Azure OpenAI** | `Microsoft.CognitiveServices/accounts` | LLM model hosting and inference | Connection (manual or auto) | Configurable |
| **AI Services** | `Microsoft.CognitiveServices/accounts` | Multi-service cognitive resource | Connection (manual) | Configurable |
| **Log Analytics Workspace** | `Microsoft.OperationalInsights/workspaces` | Log aggregation for App Insights | Hub (auto) | ✅ Yes |

💡 **Key Insight:** The Storage Account is the most critical underlying resource. It stores everything from your uploaded data to prompt flow definitions, evaluation results, and fine-tuning datasets. If you accidentally delete the storage account, you lose all project data.

### Storage Account Internal Structure

```
Storage Account (stcontosoai...)
├── azureml-blobstore-{guid}/          # Default datastore
│   ├── data/                           # Uploaded datasets
│   ├── runs/                           # Experiment runs
│   └── snapshots/                      # Code snapshots
├── azureml-filestore-{guid}/          # File share datastore
├── code-{hash}/                        # Prompt flow code
├── prompt-flow/                        # Flow definitions
└── logs/                               # Execution logs
```

---

## 5. Networking Architecture

### Overview of Networking Options

AI Foundry supports three main networking configurations, from simplest to most secure:

```
   ┌────────────────────────────────────────────────────────┐
   │            NETWORKING OPTIONS SPECTRUM                  │
   │                                                         │
   │  ◄── Simplest                          Most Secure ──► │
   │                                                         │
   │  ┌──────────┐   ┌──────────────┐   ┌───────────────┐ │
   │  │  PUBLIC   │   │   PRIVATE    │   │  FULLY PRIVATE │ │
   │  │  ACCESS   │   │  ENDPOINTS   │   │  (VNet Integ.) │ │
   │  │          │   │  + Public    │   │               │ │
   │  │ Internet  │   │  Internet +  │   │  VNet only    │ │
   │  │ accessible│   │  Private EP  │   │  No public    │ │
   │  └──────────┘   └──────────────┘   └───────────────┘ │
   │                                                         │
   │  Dev/Test         Hybrid/Staging      Production        │
   └────────────────────────────────────────────────────────┘
```

### Option 1: Public Access (Default)

```
   ┌─────────┐         ┌──────────────────┐
   │ Internet │────────►│  AI Foundry Hub   │
   │ Client   │  HTTPS  │  (Public IP)      │
   └─────────┘         │                    │
                        │  ┌──────────────┐ │
                        │  │ Storage Acct  │ │
                        │  │ Key Vault     │ │
                        │  │ OpenAI Service│ │
                        │  └──────────────┘ │
                        └──────────────────┘
```

- ✅ Easiest to set up
- ✅ No VNet configuration required
- ⚠️ Accessible from any network
- 📝 Use IP restrictions and conditional access for security

### Option 2: Private Endpoints

```
   ┌──────────────────────────────────────────────────┐
   │                    Azure VNet                     │
   │                                                   │
   │  ┌──────────┐    Private     ┌────────────────┐  │
   │  │ VM/App   │───Endpoint────►│ AI Foundry Hub  │  │
   │  │ (Client) │   10.0.1.5     │ (Private IP)    │  │
   │  └──────────┘                │                  │  │
   │                              │ ┌──────────────┐ │  │
   │                              │ │PE: Storage   │ │  │
   │                              │ │PE: Key Vault │ │  │
   │                              │ │PE: OpenAI    │ │  │
   │                              │ └──────────────┘ │  │
   │                              └──────────────────┘  │
   └──────────────────────────────────────────────────┘
```

- ✅ Traffic stays on Microsoft backbone
- ✅ No public IP exposure for data plane
- ⚠️ Requires VNet, DNS configuration
- 📝 Can still allow public access to control plane

### Option 3: Fully Private (Managed VNet)

```
   ┌─────────────────────────────────────────────────────┐
   │                 Customer VNet                        │
   │                                                      │
   │  ┌────────┐  ┌────────────────────────────────────┐ │
   │  │ Client │  │   AI Foundry Managed VNet           │ │
   │  │ Subnet │──│                                     │ │
   │  │        │  │  ┌─────────┐  ┌──────────────────┐ │ │
   │  └────────┘  │  │Hub/Proj │  │ Private Endpoints │ │ │
   │              │  │Compute  │  │ to all dependent  │ │ │
   │              │  │         │  │ resources          │ │ │
   │              │  └─────────┘  └──────────────────┘ │ │
   │              └────────────────────────────────────┘ │
   └─────────────────────────────────────────────────────┘
```

- ✅ Maximum security posture
- ✅ All traffic on private network
- ⚠️ Complex setup, requires network planning
- ⚠️ Some features may have limitations

### Networking Comparison Table

| Feature | Public Access | Private Endpoint | Managed VNet |
|---------|-------------|-----------------|--------------|
| Setup complexity | Low | Medium | High |
| Internet accessible | Yes | Optional | No |
| Data exfiltration risk | Higher | Lower | Lowest |
| VNet required | No | Yes | Yes |
| DNS configuration | No | Yes (Private DNS) | Managed |
| Cost | Lowest | Medium | Highest |
| Best for | Dev/test | Staging/Hybrid | Production |

---

## 6. Authentication & Identity

### Identity Model Overview

AI Foundry uses Azure's identity platform (Microsoft Entra ID) and supports multiple authentication methods:

```
   ┌──────────────────────────────────────────────────────┐
   │              AUTHENTICATION METHODS                   │
   │                                                       │
   │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
   │  │  Managed     │  │   Service    │  │   User       │ │
   │  │  Identity    │  │  Principal   │  │  Identity    │ │
   │  │             │  │  (App Reg)   │  │  (Entra ID)  │ │
   │  │ ✅ Preferred │  │ ✅ Automation │  │ ✅ Interactive│ │
   │  │ No secrets!  │  │ Client secret│  │ Browser SSO  │ │
   │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘ │
   │         │                │                 │         │
   │         ▼                ▼                 ▼         │
   │  ┌──────────────────────────────────────────────┐   │
   │  │              RBAC Role Assignment             │   │
   │  │                                               │   │
   │  │  Scope: Subscription / RG / Hub / Project     │   │
   │  │  Role: Owner / Contributor / Reader / Custom   │   │
   │  └──────────────────────────────────────────────┘   │
   └──────────────────────────────────────────────────────┘
```

### Managed Identity Types

| Type | Description | Use Case |
|------|-------------|----------|
| **System-assigned** | Created with the Hub, tied to its lifecycle | Hub accessing Storage, Key Vault |
| **User-assigned** | Created independently, can be shared | Multiple resources needing same access |

### RBAC Roles for AI Foundry

| Role | Scope | Permissions |
|------|-------|-------------|
| **Azure AI Developer** | Hub/Project | Deploy models, create connections, run experiments. Cannot manage access or create resources. |
| **Azure AI Inference Deployment Operator** | Hub/Project | Deploy and manage inference endpoints only |
| **Contributor** | Resource Group | Full resource management (no access management) |
| **Owner** | Resource Group | Full resource management + access management |
| **Reader** | Hub/Project | View-only access to all resources |
| **Cognitive Services OpenAI User** | OpenAI Resource | Call OpenAI model endpoints |
| **Cognitive Services OpenAI Contributor** | OpenAI Resource | Manage OpenAI deployments + call endpoints |
| **Storage Blob Data Contributor** | Storage Account | Read/write blob data |
| **Key Vault Secrets User** | Key Vault | Read secrets from Key Vault |

💡 **Best Practice:** Use **Azure AI Developer** for team members who need to build and experiment. Use **Contributor** sparingly — only for those who need to create or modify infrastructure.

### Authentication Flow Example

```
   Developer                    AI Foundry               Azure OpenAI
   ────────                    ──────────               ─────────────
       │                           │                         │
       │  1. az login (Entra ID)   │                         │
       │──────────────────────────►│                         │
       │                           │                         │
       │  2. Token (Bearer)        │                         │
       │◄──────────────────────────│                         │
       │                           │                         │
       │  3. API call + Bearer     │                         │
       │──────────────────────────►│                         │
       │                           │  4. Managed Identity    │
       │                           │─────────────────────────►
       │                           │                         │
       │                           │  5. Model response      │
       │                           │◄─────────────────────────
       │  6. Response              │                         │
       │◄──────────────────────────│                         │
```

### Service Principal Setup (for CI/CD)

```bash
# Create a service principal for automated deployments
az ad sp create-for-rbac \
  --name "sp-ai-foundry-cicd" \
  --role "Azure AI Developer" \
  --scopes "/subscriptions/{sub-id}/resourceGroups/{rg-name}"

# Assign additional roles as needed
az role assignment create \
  --assignee {sp-object-id} \
  --role "Cognitive Services OpenAI User" \
  --scope "/subscriptions/{sub-id}/resourceGroups/{rg-name}"
```

⚠️ **Security Warning:** Never hardcode service principal secrets in source code. Use Azure Key Vault or GitHub/Azure DevOps secret stores.

---

## 7. Model Hosting Options

### Deployment Types Overview

AI Foundry offers three primary ways to host and serve AI models:

```
   ┌──────────────────────────────────────────────────────────┐
   │               MODEL DEPLOYMENT OPTIONS                     │
   │                                                            │
   │  ┌──────────────┐  ┌──────────────┐  ┌────────────────┐ │
   │  │   MANAGED     │  │  SERVERLESS   │  │  PROVISIONED   │ │
   │  │   COMPUTE     │  │  API (MaaS)   │  │  THROUGHPUT    │ │
   │  │              │  │              │  │                │ │
   │  │ Your own VMs  │  │ Pay-per-token │  │ Reserved PTUs  │ │
   │  │ Full control  │  │ No infra mgmt │  │ Guaranteed     │ │
   │  │ OSS models    │  │ Auto-scales   │  │ capacity       │ │
   │  └──────────────┘  └──────────────┘  └────────────────┘ │
   └──────────────────────────────────────────────────────────┘
```

### Detailed Comparison Table

| Feature | Managed Compute | Serverless API (MaaS) | Provisioned Throughput |
|---------|----------------|----------------------|----------------------|
| **What it is** | Dedicated VMs you manage | Microsoft-hosted, pay-per-use | Reserved capacity units (PTUs) |
| **Billing model** | VM hours (always-on) | Per 1K tokens (input/output) | Monthly PTU reservation |
| **Auto-scaling** | Manual or auto-scale rules | Automatic (managed by Azure) | Fixed capacity |
| **Model types** | OSS models (Llama, Mistral, Phi) | OpenAI + select partner models | OpenAI models |
| **Infrastructure** | Customer-managed VMs | Fully managed by Microsoft | Fully managed by Microsoft |
| **Minimum cost** | ~$0.50/hr (small VM) | $0 (pay only for usage) | ~$2,000/month (min PTUs) |
| **Latency** | Depends on VM size | Variable (shared infra) | Consistent (dedicated) |
| **SLA** | VM SLA (99.9%+) | API SLA | Higher SLA with guaranteed throughput |
| **Best for** | Custom/OSS models, fine-tuned | Dev/test, variable workloads | Production, predictable workloads |
| **GPU options** | Full range (A100, H100, etc.) | N/A (managed) | N/A (managed) |
| **VNet support** | Full | Limited | Full |
| **Data processing** | In your subscription | Microsoft infrastructure | Microsoft infrastructure |

### Azure OpenAI Deployment Types

Within Azure OpenAI specifically, there are sub-categories:

```
Azure OpenAI Deployment Types
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌──────────────┐     ┌──────────────┐     ┌──────────────────┐
│   GLOBAL     │     │  STANDARD    │     │   PROVISIONED    │
│  STANDARD    │     │              │     │   (PTU)          │
│              │     │              │     │                  │
│ • Best effort│     │ • Regional   │     │ • Reserved       │
│ • Lowest cost│     │ • Pay-per-use│     │ • Guaranteed     │
│ • Global     │     │ • Per 1K     │     │   throughput     │
│   routing    │     │   tokens     │     │ • Monthly commit │
│ • No region  │     │ • Region-    │     │ • Consistent     │
│   guarantee  │     │   specific   │     │   latency        │
│              │     │              │     │                  │
│ Dev/Test     │     │ Standard     │     │ Production       │
│              │     │ Workloads    │     │ Critical Apps    │
└──────────────┘     └──────────────┘     └──────────────────┘
```

| Deployment Type | Data Residency | Rate Limits | Cost Model | SLA |
|----------------|---------------|-------------|------------|-----|
| **Global Standard** | Data may leave region | Highest default limits | Pay-per-token (lowest) | Standard |
| **Standard** | Data stays in region | Moderate limits | Pay-per-token | Standard |
| **Provisioned (PTU)** | Data stays in region | Based on PTUs purchased | Monthly reservation | Enhanced |
| **Global Provisioned** | Data may leave region | Based on PTUs purchased | Monthly reservation | Enhanced |

💡 **When to Choose What:**
- **Prototyping?** → Global Standard (cheapest, highest availability)
- **Production with compliance?** → Standard (regional data residency)
- **High-volume production?** → Provisioned (guaranteed throughput, predictable cost)

---

## 8. Connections & Data Integration

### What Are Connections?

Connections are the mechanism by which AI Foundry projects access external data and services. They securely store credentials and configuration needed to reach these resources.

```
   ┌─────────────────────────────────────────────────────┐
   │                  AI FOUNDRY PROJECT                  │
   │                                                      │
   │  ┌─────────────────────────────────────────────┐    │
   │  │              CONNECTIONS                     │    │
   │  │                                              │    │
   │  │  ┌─────────┐ ┌─────────┐ ┌──────────────┐  │    │
   │  │  │Azure AI  │ │Azure    │ │Azure Blob    │  │    │
   │  │  │Search    │ │OpenAI   │ │Storage       │  │    │
   │  │  └────┬────┘ └────┬────┘ └──────┬───────┘  │    │
   │  │       │           │             │           │    │
   │  │  ┌────┴────┐ ┌────┴────┐ ┌──────┴───────┐  │    │
   │  │  │CosmosDB │ │Custom   │ │Azure SQL     │  │    │
   │  │  │         │ │API      │ │Database      │  │    │
   │  │  └─────────┘ └─────────┘ └──────────────┘  │    │
   │  └─────────────────────────────────────────────┘    │
   └─────────────────────────────────────────────────────┘
```

### Supported Connection Types

| Connection Type | Service | Auth Method | Common Use Case |
|----------------|---------|-------------|-----------------|
| **Azure OpenAI** | Azure OpenAI Service | API Key / Managed Identity | LLM inference, embeddings |
| **Azure AI Search** | Azure AI Search | API Key / Managed Identity | RAG knowledge retrieval |
| **Azure Blob Storage** | Azure Storage | Account Key / SAS / MI | Document storage, data files |
| **Azure Cosmos DB** | Azure Cosmos DB | Connection String / MI | Structured data, chat history |
| **Azure SQL** | Azure SQL Database | Connection String | Relational data queries |
| **Azure Content Safety** | Azure AI Content Safety | API Key / MI | Content moderation |
| **Custom** | Any REST API | API Key / Bearer Token | Third-party services |
| **Serverless Model** | Model Catalog | API Key | MaaS model endpoints |

### Hub-Level vs Project-Level Connections

```
   ┌──── Hub ──────────────────────────────────────┐
   │                                                │
   │  Hub Connections (shared):                     │
   │  ├── Azure OpenAI (shared across projects)     │
   │  └── Azure AI Search (shared index)            │
   │                                                │
   │  ┌──── Project A ────────┐                     │
   │  │  Project Connections:  │                     │
   │  │  └── CosmosDB (team A) │                     │
   │  └───────────────────────┘                     │
   │                                                │
   │  ┌──── Project B ────────┐                     │
   │  │  Project Connections:  │                     │
   │  │  └── Azure SQL (team B)│                     │
   │  └───────────────────────┘                     │
   └────────────────────────────────────────────────┘
```

📝 **Note:** Hub-level connections are inherited by all projects. Project-level connections are only accessible within that project. Use Hub-level connections for shared services (like a central OpenAI resource) and project-level for team-specific data.

### Creating a Connection (Python SDK)

```python
from azure.ai.ml import MLClient
from azure.ai.ml.entities import AzureOpenAIConnection
from azure.identity import DefaultAzureCredential

ml_client = MLClient(
    credential=DefaultAzureCredential(),
    subscription_id="your-sub-id",
    resource_group_name="your-rg",
    workspace_name="your-project"
)

# Create Azure OpenAI connection
connection = AzureOpenAIConnection(
    name="my-openai-connection",
    azure_endpoint="https://my-oai.openai.azure.com/",
    api_key="your-api-key",  # Or use managed identity
    api_version="2024-02-01"
)

ml_client.connections.create_or_update(connection)
```

---

## 9. Compute Resources

### What Runs Where?

Understanding compute is essential for cost and performance planning:

```
   ┌──────────────────────────────────────────────────────────┐
   │                    COMPUTE LANDSCAPE                      │
   │                                                           │
   │  ┌─────────────────┐  ┌───────────────┐  ┌────────────┐ │
   │  │   INFERENCE      │  │  FINE-TUNING   │  │ EVALUATION  │ │
   │  │                  │  │               │  │             │ │
   │  │ • Managed Online │  │ • Managed     │  │ • Serverless│ │
   │  │   Endpoints      │  │   Compute     │  │   Compute   │ │
   │  │ • Serverless API │  │   Clusters    │  │ • Managed   │ │
   │  │ • Azure OpenAI   │  │ • GPU VMs     │  │   Compute   │ │
   │  │   Endpoints      │  │   (NC, ND     │  │             │ │
   │  │                  │  │    series)    │  │             │ │
   │  └─────────────────┘  └───────────────┘  └────────────┘ │
   │                                                           │
   │  ┌─────────────────┐  ┌───────────────┐                  │
   │  │ PROMPT FLOW      │  │  DEVELOPMENT   │                  │
   │  │                  │  │               │                  │
   │  │ • Managed Online │  │ • Compute     │                  │
   │  │   Endpoints      │  │   Instances   │                  │
   │  │ • Serverless     │  │   (Notebooks) │                  │
   │  │   Compute        │  │ • Local       │                  │
   │  │                  │  │   VS Code     │                  │
   │  └─────────────────┘  └───────────────┘                  │
   └──────────────────────────────────────────────────────────┘
```

### Compute Types Reference

| Compute Type | GPU Available | Use Case | Billing | Auto-Shutdown |
|-------------|-------------|----------|---------|---------------|
| **Compute Instance** | Yes | Development, notebooks | Per-hour (when running) | Configurable |
| **Compute Cluster** | Yes | Training, fine-tuning, batch inference | Per-hour (auto-scales to 0) | Auto-scale down |
| **Managed Online Endpoint** | Yes | Real-time inference (OSS models) | Per-hour + per-request | Always on |
| **Serverless Compute** | Managed | Evaluations, prompt flow testing | Per-job | N/A |
| **Azure OpenAI Endpoint** | Managed | OpenAI model inference | Per-token or PTU | Always on |

⚠️ **Cost Warning:** Compute instances and managed online endpoints run 24/7 unless you stop them. Set up auto-shutdown schedules for development compute instances!

---

## 10. SDK & API Architecture

### Access Methods Overview

```
   ┌──────────────────────────────────────────────────────┐
   │              HOW TO INTERACT WITH AI FOUNDRY           │
   │                                                       │
   │  ┌──────────┐  ┌──────────┐  ┌──────────┐           │
   │  │  REST    │  │  Python  │  │ Azure    │           │
   │  │  APIs    │  │  SDK     │  │ CLI      │           │
   │  └─────┬────┘  └─────┬────┘  └─────┬────┘           │
   │        │              │             │                 │
   │  ┌─────┴──────────────┴─────────────┴──────────────┐ │
   │  │          Azure Resource Manager (ARM)            │ │
   │  │          + AI Foundry Data Plane APIs            │ │
   │  └─────────────────────────────────────────────────┘ │
   │                                                       │
   │  ┌──────────┐  ┌──────────┐  ┌──────────┐           │
   │  │ VS Code  │  │ AI       │  │ Bicep /  │           │
   │  │Extension │  │ Foundry  │  │Terraform │           │
   │  │          │  │ Portal   │  │          │           │
   │  └──────────┘  └──────────┘  └──────────┘           │
   └──────────────────────────────────────────────────────┘
```

### SDK Landscape

| SDK / Tool | Language | Install | Primary Use |
|-----------|---------|---------|-------------|
| `azure-ai-projects` | Python | `pip install azure-ai-projects` | Project-level operations, connections, evaluations |
| `azure-ai-inference` | Python | `pip install azure-ai-inference` | Model inference (chat, embeddings) |
| `azure-ai-ml` | Python | `pip install azure-ai-ml` | ML workspace, compute, data management |
| `openai` | Python | `pip install openai` | OpenAI-compatible API calls |
| `azure-identity` | Python | `pip install azure-identity` | Authentication (DefaultAzureCredential) |
| `az ml` | CLI | `az extension add -n ml` | ML workspace operations via CLI |
| `az cognitiveservices` | CLI | Built-in | OpenAI and Cognitive Services management |
| Bicep/ARM | IaC | Built-in | Infrastructure as Code deployment |
| VS Code Extension | Extension | Marketplace | Interactive development in VS Code |

### REST API Endpoints

```
Control Plane (ARM):
  https://management.azure.com/subscriptions/{sub}/resourceGroups/{rg}/
    providers/Microsoft.MachineLearningServices/workspaces/{hub-or-project}
    ?api-version=2024-04-01

Data Plane (AI Foundry):
  https://{project-name}.{region}.api.azureml.ms/

Azure OpenAI Data Plane:
  https://{resource-name}.openai.azure.com/openai/deployments/{deployment}/
    chat/completions?api-version=2024-02-01

AI Foundry Inference:
  https://{endpoint-name}.{region}.inference.ml.azure.com/score
```

### Quick Code Examples

**Python SDK — Chat Completion:**
```python
from azure.ai.inference import ChatCompletionsClient
from azure.identity import DefaultAzureCredential

client = ChatCompletionsClient(
    endpoint="https://my-project.eastus.api.azureml.ms/",
    credential=DefaultAzureCredential()
)

response = client.complete(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is Azure AI Foundry?"}
    ]
)
print(response.choices[0].message.content)
```

**Azure CLI — List Projects:**
```bash
# List all projects in a hub
az ml workspace list \
  --resource-group rg-ai-foundry \
  --query "[?kind=='Project']" \
  --output table
```

---

## 11. Region Availability & Data Residency

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Region** | Physical location of Azure data centers (e.g., East US, West Europe) |
| **Data Residency** | Where your data is stored and processed |
| **Geo-Redundancy** | Data replicated to a paired region for disaster recovery |
| **Sovereign Clouds** | Government-specific Azure environments (Azure Gov, Azure China) |

### Region Selection Considerations

```
   ┌──────────────────────────────────────────────────────┐
   │           REGION SELECTION DECISION TREE               │
   │                                                       │
   │  1. Compliance Requirements?                          │
   │     └──► GDPR → West Europe / North Europe            │
   │     └──► HIPAA → US regions with BAA                  │
   │     └──► Government → Azure Government Cloud          │
   │                                                       │
   │  2. Model Availability?                               │
   │     └──► GPT-4o → Check model availability matrix     │
   │     └──► Llama/Mistral → Check MaaS availability      │
   │                                                       │
   │  3. Latency?                                          │
   │     └──► Choose closest region to your users           │
   │                                                       │
   │  4. Cost?                                             │
   │     └──► Some regions are more expensive               │
   │     └──► Global deployments can use any region         │
   └──────────────────────────────────────────────────────┘
```

💡 **Tip:** Not all models are available in all regions. Always check the [Azure OpenAI model availability matrix](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models) before choosing a region.

⚠️ **Important:** When you use **Global Standard** deployments, your data may be processed in any region globally. If you have data residency requirements, use **Standard** or **Provisioned** deployments.

---

## 12. Cost Structure

### What Costs Money vs What's Free

| Component | Free? | Cost Model | Typical Range |
|-----------|-------|------------|---------------|
| **AI Foundry Hub** | ✅ Free | No charge for the Hub resource itself | $0 |
| **AI Foundry Project** | ✅ Free | No charge for the Project resource itself | $0 |
| **AI Foundry Portal** | ✅ Free | Web portal access | $0 |
| **Storage Account** | ❌ Paid | Per GB stored + transactions | $5-50/month |
| **Key Vault** | ❌ Paid | Per operation + secrets stored | $1-5/month |
| **Application Insights** | ❌ Paid | Per GB ingested (5GB/month free) | $0-50/month |
| **Container Registry** | ❌ Paid | Tier-based (Basic/Standard/Premium) | $5-170/month |
| **Azure OpenAI (Standard)** | ❌ Paid | Per 1K tokens (input/output) | Varies by model |
| **Azure OpenAI (Provisioned)** | ❌ Paid | Monthly PTU reservation | $2,000+/month |
| **Compute Instance** | ❌ Paid | Per-hour (VM pricing) | $0.10-10/hour |
| **Managed Online Endpoint** | ❌ Paid | Per-hour (VM pricing) | $0.50-20/hour |
| **Fine-tuning** | ❌ Paid | Training hours + hosting | Model-specific |
| **Serverless Models (MaaS)** | ❌ Paid | Per 1K tokens | Model-specific |

### Cost Estimation Example

**Scenario: Small team, prototyping phase**

| Resource | Configuration | Monthly Estimate |
|----------|-------------|-----------------|
| Hub + 2 Projects | — | $0 |
| Storage Account | LRS, 10 GB | $2 |
| Key Vault | Standard, ~1000 ops | $1 |
| App Insights | 2 GB/month | $0 (free tier) |
| Container Registry | Basic tier | $5 |
| Azure OpenAI (GPT-4o) | ~1M tokens/month | $7.50 |
| Compute Instance | Standard_DS3_v2, 40 hrs/month | $15 |
| **Total** | | **~$30/month** |

**Scenario: Production deployment**

| Resource | Configuration | Monthly Estimate |
|----------|-------------|-----------------|
| Hub + 5 Projects | — | $0 |
| Storage Account | GRS, 500 GB | $50 |
| Key Vault | Standard, ~100K ops | $3 |
| App Insights | 50 GB/month | $115 |
| Container Registry | Standard tier | $20 |
| Azure OpenAI (PTU) | 50 PTUs GPT-4o | $3,300 |
| Managed Endpoints | 2x Standard_NC6s_v3 | $2,200 |
| Private Endpoints | 5 endpoints | $35 |
| **Total** | | **~$5,700/month** |

📝 **Cost Optimization Tips:**
1. Use auto-shutdown on compute instances (saves 60%+ on dev compute)
2. Start with Global Standard deployments (cheapest per-token)
3. Use Provisioned Throughput only when you have predictable, high-volume workloads
4. Monitor Application Insights data ingestion — it can be surprisingly expensive
5. Delete unused model deployments — some still incur hosting costs

---

## 13. Architecture Diagrams

### Complete AI Foundry Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                        AZURE SUBSCRIPTION                             │
│                                                                       │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │                      RESOURCE GROUP                            │  │
│  │                                                                │  │
│  │  ┌──────────────────── AI FOUNDRY HUB ──────────────────────┐ │  │
│  │  │                                                           │ │  │
│  │  │  ┌─────────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐│ │  │
│  │  │  │ Storage     │ │Key Vault │ │App       │ │Container ││ │  │
│  │  │  │ Account     │ │          │ │Insights  │ │Registry  ││ │  │
│  │  │  └──────┬──────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘│ │  │
│  │  │         │             │            │             │       │ │  │
│  │  │  ┌──────┴─────────────┴────────────┴─────────────┴────┐  │ │  │
│  │  │  │              SHARED INFRASTRUCTURE                  │  │ │  │
│  │  │  │  • Networking (Public/Private/VNet)                 │  │ │  │
│  │  │  │  • Managed Identity (System + User-assigned)        │  │ │  │
│  │  │  │  • Hub-level Connections (Azure OpenAI, AI Search)  │  │ │  │
│  │  │  └────────────────────┬───────────────────────────────┘  │ │  │
│  │  │                       │                                   │ │  │
│  │  │         ┌─────────────┴──────────────┐                   │ │  │
│  │  │         │                            │                   │ │  │
│  │  │  ┌──────▼──────┐          ┌──────────▼────────┐         │ │  │
│  │  │  │  PROJECT A   │          │   PROJECT B        │         │ │  │
│  │  │  │             │          │                   │         │ │  │
│  │  │  │ Deployments: │          │ Deployments:      │         │ │  │
│  │  │  │ • GPT-4o     │          │ • Llama-3          │         │ │  │
│  │  │  │ • Ada-002    │          │ • Phi-3            │         │ │  │
│  │  │  │             │          │                   │         │ │  │
│  │  │  │ Connections: │          │ Connections:      │         │ │  │
│  │  │  │ • CosmosDB   │          │ • Azure SQL        │         │ │  │
│  │  │  │ • Blob Store │          │ • Custom API       │         │ │  │
│  │  │  │             │          │                   │         │ │  │
│  │  │  │ Prompt Flows │          │ Evaluations       │         │ │  │
│  │  │  │ Indexes      │          │ Fine-tuning       │         │ │  │
│  │  │  └─────────────┘          └───────────────────┘         │ │  │
│  │  └───────────────────────────────────────────────────────────┘ │  │
│  │                                                                │  │
│  │  ┌─────────────────────────────────────────────────────────┐  │  │
│  │  │               CONNECTED SERVICES                         │  │  │
│  │  │                                                          │  │  │
│  │  │  ┌──────────┐ ┌────────────┐ ┌──────────┐ ┌──────────┐│  │  │
│  │  │  │Azure     │ │Azure AI    │ │Azure     │ │Custom    ││  │  │
│  │  │  │OpenAI    │ │Search      │ │CosmosDB  │ │APIs      ││  │  │
│  │  │  │Service   │ │            │ │          │ │          ││  │  │
│  │  │  └──────────┘ └────────────┘ └──────────┘ └──────────┘│  │  │
│  │  └─────────────────────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────┘
```

### Data Flow Architecture

```
   USER REQUEST FLOW
   ━━━━━━━━━━━━━━━━

   User ──► App/API ──► AI Foundry Project ──┐
                                              │
                    ┌─────────────────────────┤
                    │                         │
                    ▼                         ▼
            ┌──────────────┐         ┌──────────────┐
            │ Azure OpenAI  │         │ Azure AI     │
            │ (LLM Call)    │         │ Search       │
            │               │         │ (RAG Lookup) │
            └──────┬───────┘         └──────┬───────┘
                   │                        │
                   │    ┌───────────┐       │
                   └───►│ Prompt    │◄──────┘
                        │ Flow      │
                        │ (Orchest.)│
                        └─────┬─────┘
                              │
                              ▼
                        ┌───────────┐
                        │ Content   │
                        │ Safety    │
                        │ Filter    │
                        └─────┬─────┘
                              │
                              ▼
                        Response ──► User
```

---

## 14. Real-World Architecture Examples

### Example 1: Enterprise RAG Chatbot

```
Scenario: Customer support chatbot using company knowledge base.

Architecture:
• 1 Hub (hub-contoso-support)
  └── 1 Project (proj-support-chatbot)
      ├── Azure OpenAI: GPT-4o (Standard deployment)
      ├── Azure OpenAI: text-embedding-3-large
      ├── Azure AI Search: Company knowledge base index
      ├── Azure Blob Storage: Document repository
      ├── CosmosDB: Chat history & user sessions
      └── Prompt Flow: RAG orchestration pipeline

Networking: Private endpoints (production)
Identity: Managed Identity + RBAC
Cost: ~$2,000/month (Standard deployments)
```

### Example 2: Multi-Team AI Platform

```
Scenario: Central AI platform serving multiple business units.

Architecture:
• 1 Hub (hub-enterprise-ai)
  ├── Project: proj-marketing-content
  │   ├── GPT-4o (content generation)
  │   └── DALL·E 3 (image generation)
  ├── Project: proj-legal-review
  │   ├── GPT-4o (document analysis)
  │   └── Azure AI Search (legal document index)
  ├── Project: proj-data-analytics
  │   ├── GPT-4o (data interpretation)
  │   └── Azure SQL connection
  └── Project: proj-hr-assistant
      ├── GPT-4o (employee Q&A)
      └── SharePoint connection

Networking: Managed VNet (fully private)
Identity: Service principals per team + Managed Identity
Cost: ~$8,000/month (mix of Standard + Provisioned)
```

### Example 3: Multi-Region Deployment

```
Scenario: Global application with data residency requirements.

Architecture:
• Hub: hub-us-east (East US)
  └── Project: proj-us-app
      └── GPT-4o Standard (US data stays in US)

• Hub: hub-eu-west (West Europe)
  └── Project: proj-eu-app
      └── GPT-4o Standard (EU data stays in EU)

• Azure Front Door: Global load balancing
  └── Routes US users → US Hub, EU users → EU Hub

Networking: Private endpoints per region
Identity: Centralized Entra ID, region-specific RBAC
Cost: ~$6,000/month (two regions, Standard deployments)
```

---

## 15. Summary & Key Takeaways

### The Big Picture

```
   KEY CONCEPTS TO REMEMBER
   ━━━━━━━━━━━━━━━━━━━━━━━

   1. HIERARCHY:   Subscription → Resource Group → Hub → Project
   2. HUB:         Shared infra (Storage, Key Vault, Networking)
   3. PROJECT:     Team workspace (Deployments, Connections, Flows)
   4. NETWORKING:  Public → Private Endpoints → Managed VNet
   5. IDENTITY:    Managed Identity > Service Principal > API Keys
   6. DEPLOYMENTS: Global Standard → Standard → Provisioned
   7. CONNECTIONS:  How projects reach external data
   8. COST:        Hub/Project free; pay for compute + tokens + storage
```

### ✅ Checklist: Can You...

- [ ] Draw the 4-level resource hierarchy from memory?
- [ ] Explain why you'd use multiple Hubs vs multiple Projects?
- [ ] Name all 4 underlying resources a Hub creates automatically?
- [ ] Describe the three networking options and when to use each?
- [ ] List at least 4 RBAC roles relevant to AI Foundry?
- [ ] Compare Managed Compute, Serverless API, and Provisioned deployments?
- [ ] Estimate monthly costs for a basic AI Foundry setup?
- [ ] Create a connection using the Python SDK?

---

## 16. Glossary

| Term | Definition |
|------|-----------|
| **AI Foundry Account** | The Azure resource (type: Hub) that provides shared infrastructure for AI projects |
| **ARM** | Azure Resource Manager — the deployment and management layer for Azure resources |
| **Connection** | A secure reference to an external service (stores credentials in Key Vault) |
| **Data Plane** | APIs for working with data and models (inference, data upload, etc.) |
| **Control Plane** | APIs for managing resources (create, update, delete hubs/projects) |
| **Global Standard** | OpenAI deployment type with global routing for highest availability |
| **Hub** | Short name for AI Foundry Account — the shared infrastructure parent |
| **MaaS** | Model as a Service — serverless model hosting (pay-per-token, no infra management) |
| **Managed Identity** | Azure-managed credentials that eliminate the need for secrets in code |
| **Managed Online Endpoint** | A deployment target for real-time model inference on dedicated compute |
| **Private Endpoint** | A network interface that connects you privately to an Azure service |
| **Project** | A workspace within a Hub for a specific team or application |
| **Prompt Flow** | Visual workflow tool for building AI orchestration pipelines |
| **PTU** | Provisioned Throughput Unit — a unit of reserved model capacity |
| **RBAC** | Role-Based Access Control — Azure's authorization system |
| **RAG** | Retrieval-Augmented Generation — pattern combining search with LLM generation |
| **Serverless API** | Model deployment where Microsoft manages all infrastructure |
| **Standard Deployment** | Regional OpenAI deployment with pay-per-token billing |
| **VNet** | Virtual Network — a private network in Azure |

---

**next module:** [Module 4 — Working with Models in AI Foundry →](../module-04/training-guide.md)

**Previous module:** [← Module 2 — Setting Up Your Environment](../module-02/training-guide.md)

---

*© 2025 Zero to Hero Training Team. Azure AI Foundry Training Course.*
