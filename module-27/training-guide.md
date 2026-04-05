# Module 27: CI/CD Pipelines for AI

> **Azure AI Foundry: Zero to Hero** · Arc 6: Deployment & MLOps · Module 27 of 45  
> **Duration:** ~90 minutes · **Level:** Intermediate–Advanced · **Date:** April 2026

---

## 🎯 Learning Objectives

By the end of this module, learners will be able to:

1. **Design** end-to-end CI/CD pipelines for Azure AI Foundry deployments
2. **Implement** GitHub Actions workflows with OIDC authentication to deploy AI resources
3. **Build** Azure DevOps multi-stage pipelines with approval gates
4. **Author** Infrastructure as Code (Bicep & Terraform) for AI Foundry hubs, projects, and deployments
5. **Integrate** automated AI evaluations as quality gates in CI/CD pipelines
6. **Apply** MLOps best practices for production AI systems

---

## 📋 Prerequisites

| Requirement | Details |
|---|---|
| Azure subscription | With Contributor access and AI Foundry enabled |
| GitHub account | With repository and Actions access |
| Azure CLI | v2.60+ installed locally |
| Python | 3.10+ with pip |
| Prior modules | Module 25 (Evaluations) and Module 26 (Model Packaging) recommended |

---

## 📑 Module Outline

### Unit 1: Why CI/CD for AI Matters (15 min)

**Key Concepts:**
- Traditional CI/CD vs. AI-specific CI/CD challenges
- The MLOps maturity model (Levels 0–4)
- Azure AI Foundry's automation-first design

**Discussion Points:**
- What makes AI deployments different from traditional software deployments?
- Why are evaluation gates critical for AI systems?
- How does Infrastructure as Code prevent configuration drift?

**MLOps Maturity Levels:**

| Level | Description | Characteristics |
|---|---|---|
| 0 | No MLOps | Manual everything, no versioning |
| 1 | DevOps, no MLOps | CI/CD for app code, manual model deployment |
| 2 | Automated Training | Automated training pipelines, manual deployment |
| 3 | Automated Deployment | Full CI/CD for models and infrastructure |
| 4 | Full MLOps | Automated retraining, monitoring, and redeployment |

---

### Unit 2: GitHub Actions for Azure AI Foundry (25 min)

**Topics Covered:**
- Workload Identity Federation (OIDC) — no stored secrets
- Workflow structure: triggers, permissions, jobs, steps
- Azure Login action (`azure/login@v2`)
- Deploying AI Foundry resources via Azure CLI
- Python SDK-driven model deployments
- Reusable workflows and composite actions

**Key Workflow Components:**

```yaml
# Authentication — OIDC (recommended)
permissions:
  id-token: write
  contents: read

steps:
  - uses: azure/login@v2
    with:
      client-id: ${{ secrets.AZURE_CLIENT_ID }}
      tenant-id: ${{ secrets.AZURE_TENANT_ID }}
      subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

**Hands-On Exercise:**
- Create a basic GitHub Actions workflow that authenticates to Azure and lists AI Foundry projects

---

### Unit 3: Azure DevOps Pipeline Integration (15 min)

**Topics Covered:**
- Service connections with workload identity federation
- Multi-stage YAML pipelines (Validate → Deploy Staging → Evaluate → Deploy Prod)
- Environment-based approval gates
- Variable groups for cross-environment configuration
- Pipeline artifacts for evaluation results

**Key Pipeline Structure:**

```yaml
stages:
  - stage: Validate        # Lint + IaC validation
  - stage: DeployStaging   # Deploy to staging environment
  - stage: Evaluate        # Run AI evaluations
  - stage: DeployProd      # Deploy to production (manual approval)
```

**Discussion:**
- GitHub Actions vs. Azure DevOps — when to use which?
- How do environment protection rules work?

---

### Unit 4: Infrastructure as Code — Bicep & Terraform (20 min)

**Topics Covered:**
- Why IaC for AI resources is essential
- Bicep templates for AI Foundry Hub, Project, and AI Services
- Terraform configurations with the AzureRM provider
- Parameterized deployments for multi-environment support
- Remote state management
- What-if / plan previews in PRs

**Bicep Resources Covered:**
- `Microsoft.CognitiveServices/accounts` — AI Services
- `Microsoft.MachineLearningServices/workspaces` (kind: `Hub`) — AI Foundry Hub
- `Microsoft.MachineLearningServices/workspaces` (kind: `Project`) — AI Foundry Project

**Terraform Resources Covered:**
- `azurerm_ai_services`
- `azurerm_ai_foundry`
- `azurerm_ai_foundry_project`

**Best Practices Table:**

| Practice | Tool | Command |
|---|---|---|
| Preview changes | Bicep | `az deployment group what-if` |
| Preview changes | Terraform | `terraform plan` |
| Validate syntax | Bicep | `az bicep build` |
| Validate syntax | Terraform | `terraform validate` |
| Format code | Bicep | `az bicep format` |
| Format code | Terraform | `terraform fmt` |

---

### Unit 5: Automated Testing in CI/CD (15 min)

**Testing Layers:**

1. **Unit Tests** — Prompt template validation, schema checks, preprocessing logic
2. **Integration Tests** — API connectivity, deployment health checks, end-to-end flows
3. **AI Evaluations** — Groundedness, relevance, coherence, safety scoring

**Evaluation as a Quality Gate:**

```python
from azure.ai.evaluation import evaluate, GroundednessEvaluator

# Run evaluation
result = evaluate(
    data="data/evaluation_dataset.jsonl",
    evaluators={"groundedness": GroundednessEvaluator(model_config)},
)

# Enforce threshold
if result["metrics"]["groundedness.groundedness"] < 4.0:
    sys.exit(1)  # Fail the pipeline
```

**Key Metrics for AI Quality Gates:**

| Metric | Threshold | Description |
|---|---|---|
| Groundedness | ≥ 4.0 | Response is supported by provided context |
| Relevance | ≥ 4.0 | Response addresses the query directly |
| Coherence | ≥ 4.0 | Response is well-structured and logical |
| Fluency | ≥ 4.0 | Response uses natural, correct language |

---

## 🎬 Video Resource

**How to create AI model deployment | Microsoft Foundry**

- **URL:** [https://www.youtube.com/watch?v=LLO0lOufbwc](https://www.youtube.com/watch?v=LLO0lOufbwc)
- **Duration:** ~10 minutes
- **Key Topics:** Model deployment creation, configuration options, SDK automation

---

## 🏆 Mini Hack: Deploy & Evaluate with GitHub Actions

**Objective:** Set up a complete GitHub Actions pipeline that deploys Azure AI Foundry resources and runs automated evaluations as a quality gate.

**Deliverables:**
1. GitHub Actions workflow file (`.github/workflows/ai-foundry-deploy.yml`)
2. Bicep template for AI Foundry infrastructure (`infra/main.bicep`)
3. Python deployment script (`scripts/deploy_model.py`)
4. Python evaluation script (`scripts/run_evaluation.py`)
5. Evaluation dataset (`data/evaluation_dataset.jsonl`)

**Success Criteria:**
- [ ] Pipeline triggers on push to `main`
- [ ] OIDC authentication — no stored secrets
- [ ] Infrastructure deployed via Bicep
- [ ] Model deployed via Python SDK
- [ ] Evaluation gate passes with scores ≥ 4.0
- [ ] Production deployment requires manual approval
- [ ] Failed evaluations block production

**Time Estimate:** 45 minutes

---

## 📊 Instructor Notes

### Delivery Tips

1. **Start with the "why"** — show a real-world failure caused by manual deployment to motivate CI/CD adoption
2. **Live demo** the GitHub Actions workflow — create a commit and show the pipeline running in real-time
3. **Compare approaches** — show both Bicep and Terraform side by side and discuss team preferences
4. **Break the pipeline intentionally** — modify evaluation thresholds to show how quality gates block bad deployments

### Common Student Questions

| Question | Answer |
|---|---|
| "Can I use GitHub Actions with Azure DevOps repos?" | Yes — GitHub Actions can be triggered by webhooks from any Git provider |
| "Is Bicep or Terraform better?" | Bicep for Azure-only shops (native, no state file). Terraform for multi-cloud or teams with existing HCL expertise |
| "How much does CI/CD pipeline compute cost?" | GitHub Actions is free for public repos; private repos get 2,000 min/month free. Azure DevOps provides 1,800 min/month |
| "Can I deploy models without the SDK?" | Yes — Azure CLI and REST API work too, but the SDK provides the richest type-safe experience |

### Assessment Strategy

- **Formative:** In-class poll after each unit
- **Summative:** Mini Hack completion + Quiz (10 MCQs in quiz/assessment.md)

---

## 🔗 Additional Resources

- [Azure AI Foundry Documentation](https://learn.microsoft.com/azure/ai-foundry/)
- [GitHub Actions for Azure](https://docs.github.com/actions/deployment/deploying-to-your-cloud-provider/deploying-to-azure)
- [Bicep Templates Reference](https://learn.microsoft.com/azure/templates/)
- [Terraform AzureRM Provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest)
- [Azure DevOps Pipelines](https://learn.microsoft.com/azure/devops/pipelines/)

---

## ⏭️ What's Next

**Module 28: Monitoring AI in Production** — Learn how to monitor deployed AI models, track performance metrics, detect drift, and set up alerting for production AI systems in Azure AI Foundry.

---

*Azure AI Foundry: Zero to Hero — Module 27 of 45*  
*© 2026 AI Training Series*
