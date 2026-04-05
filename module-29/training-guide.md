# Module 29: Scaling Deployments Across Environments

## Training Guide

**Course:** Azure AI Foundry — Zero to Hero (Module 29 of 45)
**Arc:** ARC 6 · DEPLOYMENT & MLOps
**Date:** April 2026
**Duration:** 3 hours (1.5 hours instruction + 1.5 hours hands-on lab)

---

## Module Overview

This module teaches participants how to scale AI agent deployments across multiple environments — from Development through Staging to Production — using Azure AI Foundry. You'll learn environment isolation patterns, multi-region deployment strategies, load balancing and failover configuration, environment-specific configuration management, and CI/CD promotion pipelines with quality gates.

## Prerequisites

- Completion of Modules 1–28 (especially Modules 27–28 on containerization and CI/CD)
- An active Azure subscription with Contributor access
- Azure AI Foundry hub and project provisioned (from Module 3)
- A containerized AI agent (from Module 28)
- Azure Container Registry with a pushed agent image
- GitHub repository with Actions enabled
- Azure CLI 2.60+ installed
- Python 3.10+ with `azure-ai-projects` and `azure-identity` SDKs

## Learning Objectives

By the end of this module, participants will be able to:

1. **Design** a multi-environment deployment topology using separate AI Foundry Hubs per environment
2. **Implement** promotion gates between Dev, Staging, and Production environments
3. **Deploy** AI agents across multiple Azure regions for high availability
4. **Configure** Azure Front Door for latency-based routing and automatic failover
5. **Manage** environment-specific configuration using Azure App Configuration and Key Vault
6. **Build** CI/CD pipelines with canary deployments and traffic splitting
7. **Monitor** deployments across environments with differentiated alert thresholds

## Agenda

| Time | Topic | Type |
|------|-------|------|
| 0:00–0:10 | Welcome & Module Overview | Lecture |
| 0:10–0:25 | Why Multi-Environment Matters for AI | Discussion |
| 0:25–0:50 | Dev → Staging → Prod Promotion Patterns | Lecture |
| 0:50–1:05 | Featured Video + Q&A | Video |
| 1:05–1:20 | Multi-Region Deployment Strategies | Lecture |
| 1:20–1:35 | Load Balancing & Failover with Azure Front Door | Demo |
| 1:35–1:45 | Break | — |
| 1:45–2:05 | Environment-Specific Configuration Management | Demo |
| 2:05–2:20 | CI/CD Promotion Pipelines with Quality Gates | Demo |
| 2:20–2:50 | Mini Hack: Deploy Agent to Dev & Prod | Lab |
| 2:50–3:00 | Key Takeaways & Module 30 Preview | Wrap-up |

## Key Concepts

### Dev → Staging → Production Promotion Patterns

The promotion model defines how AI agents move between environments. Each environment should map to a separate AI Foundry project (ideally a separate Hub) for complete isolation:

- **Development:** Rapid iteration with synthetic data, relaxed safety filters, minimal compute. Developers have direct access.
- **Staging:** Integration testing with a subset of production data, production-level safety filters, moderate compute. CI/CD and QA teams have access.
- **Production:** Live user traffic with full data, strict safety enforcement, production-grade compute with SLA. Only CI/CD and release managers have deployment access.

**Promotion Gates:**
- Dev → Staging: Unit tests pass, container builds, smoke tests pass
- Staging → Prod: Integration tests pass, evaluation benchmarks meet thresholds (groundedness ≥ 0.85, relevance ≥ 0.80), load tests pass, security scan clean, manual approval

### Environment Architecture

**Pattern 1 — Shared Hub, Separate Projects:** Simpler to manage but less isolation. Suitable for small teams.

**Pattern 2 — Separate Hubs per Environment (Recommended):** Complete isolation of networking, identity, data, and compute. Required for compliance-sensitive workloads.

### Infrastructure as Code

Use parameterized Bicep or Terraform templates with environment-specific parameter files:
- `main.bicep` — Single template defining all resources
- `params.dev.json` — Dev-specific values (Basic SKU, low capacity)
- `params.staging.json` — Staging values (Standard SKU, moderate capacity)
- `params.prod.json` — Production values (Standard SKU, high capacity, VNet integration)

### Multi-Region Deployment Strategies

- **Active-Passive:** One region handles traffic; secondary on standby. Lower cost, higher RTO.
- **Active-Active:** Both regions serve live traffic via Azure Front Door. Higher cost, near-zero RTO.

**Region Selection Factors:** Model availability, data residency requirements, latency to users, TPM quota distribution, paired region availability.

### Load Balancing & Failover

- **Azure Front Door:** Global load balancer with latency-based routing, health probes, WAF, and session affinity.
- **Azure API Management:** AI gateway with token rate limiting, retry/failover policies, request logging, and response caching.
- **Failover Patterns:** Circuit breaker, weighted distribution (gradual traffic shift), health endpoint monitoring.

### Environment-Specific Configuration

**Principle:** Configuration must never be baked into container images. One image, many environments.

**Configuration Hierarchy:**
1. Azure App Configuration — Centralized key-value store with labels per environment
2. Azure Key Vault — Secrets, connection strings, API keys
3. Environment Variables — Injected at container runtime
4. Feature Flags — Toggle capabilities without redeployment

### CI/CD Promotion Pipelines

GitHub Actions workflow with environment-specific jobs:
- `build` → `deploy-dev` → `deploy-staging` → `deploy-prod`
- Each environment stage uses GitHub Environments with required approvals
- Production uses canary deployment (10% → 50% → 100% traffic shift)

### Monitoring & Observability

- **Application Insights** with environment-tagged telemetry
- **Azure Monitor Workbooks** for cross-environment comparison dashboards
- **Differentiated alert thresholds** per environment (e.g., Prod P95 latency < 3s vs Dev < 10s)
- **Azure AI Foundry Tracing** for agent-specific observability

## Instructor Notes

### Discussion Prompts
1. "What are the risks of deploying directly from Dev to Production?"
2. "When would you choose Active-Active over Active-Passive multi-region?"
3. "What configuration values are most dangerous if they leak across environments?"

### Common Misconceptions
- **"We only need two environments"** — Staging catches integration issues that Dev cannot.
- **"Multi-region is only for large enterprises"** — Any production AI system with SLA requirements benefits from multi-region deployment.
- **"We can just change env vars and rebuild"** — Rebuilding creates different images. The same image must be promoted through all environments.

### Demo Tips
- Show the Azure AI Foundry portal with two separate projects (dev and prod) side by side
- Demonstrate Azure App Configuration label filtering in real time
- Walk through a GitHub Actions run showing the promotion flow with approval gates
- Show Azure Front Door health probe detecting a failed backend

## Assessment

Direct participants to complete the quiz in `quiz/assessment.md` (10 questions, 70% passing score).

## Materials Checklist

- [ ] Slide deck: `slides/presentation.html`
- [ ] Hands-on lab: `lab/hands-on-lab.md`
- [ ] Assessment quiz: `quiz/assessment.md`
- [ ] Sample Bicep templates (main.bicep, params.dev.json, params.prod.json)
- [ ] Sample GitHub Actions workflow (deploy-agent.yml)
- [ ] Sample health check endpoint code (Python/FastAPI)

## References

- [Azure AI Foundry Documentation](https://learn.microsoft.com/azure/ai-studio/)
- [Azure Front Door Documentation](https://learn.microsoft.com/azure/frontdoor/)
- [Azure App Configuration](https://learn.microsoft.com/azure/azure-app-configuration/)
- [Azure Container Apps — Revisions & Traffic Splitting](https://learn.microsoft.com/azure/container-apps/revisions)
- [Azure API Management — AI Gateway](https://learn.microsoft.com/azure/api-management/)
- [Azure Well-Architected Framework](https://learn.microsoft.com/azure/well-architected/)
