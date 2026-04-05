# Module 29: Scaling Deployments Across Environments

## Quiz Assessment

**Course:** Azure AI Foundry — Zero to Hero (Module 29 of 45)
**Arc:** ARC 6 · DEPLOYMENT & MLOps
**Total Questions:** 10 | **Passing Score:** 70% (7/10)
**Time Limit:** 15 minutes

---

### Question 1

**What is the recommended environment architecture pattern for production AI workloads in Azure AI Foundry?**

- A) A single AI Foundry Hub with a single project for all environments
- B) A single AI Foundry Hub with separate projects per environment
- C) Separate AI Foundry Hubs per environment, each with its own project
- D) Separate Azure subscriptions but a shared AI Foundry Hub

**Correct Answer:** C

**Explanation:** Separate Hubs per environment provide complete isolation of networking, identity, data, and compute — which is critical for compliance, security, and independent scaling in production workloads. A shared Hub (option B) offers less isolation and is only suitable for smaller, non-compliance-sensitive teams.

---

### Question 2

**Which quality gate criterion is required for promoting an AI agent from Staging to Production?**

- A) The developer approves the deployment
- B) The container image builds successfully
- C) Evaluation benchmarks meet defined thresholds (e.g., groundedness ≥ 0.85)
- D) All unit tests pass in the development environment

**Correct Answer:** C

**Explanation:** The Staging → Production gate requires more rigorous validation than earlier gates, including evaluation benchmark thresholds (groundedness, relevance), load testing, security scans, and manual approval from a release manager. Building successfully and passing unit tests are Dev → Staging requirements.

---

### Question 3

**In a multi-region deployment, what does Azure Front Door provide for AI agent APIs?**

- A) Container image storage and version management
- B) Latency-based routing, health probes, and automatic failover
- C) Model fine-tuning and deployment across regions
- D) Data replication between Azure AI Foundry projects

**Correct Answer:** B

**Explanation:** Azure Front Door is a global load balancer that provides latency-based routing (sending users to the fastest region), health probes (detecting endpoint failures), automatic failover, WAF integration, and session affinity — all essential for multi-region AI deployments.

---

### Question 4

**What is the key principle for managing configuration across multiple environments?**

- A) Configuration should be baked into the container image at build time
- B) Each environment should have a separately built container image with different config
- C) Configuration should be externalized so the same image works across all environments
- D) Configuration should be stored in the application source code

**Correct Answer:** C

**Explanation:** The golden rule is that configuration must never be baked into container images. The same container image should be promotable through Dev → Staging → Prod. Environment-specific configuration is injected at runtime via Azure App Configuration, Key Vault, and environment variables.

---

### Question 5

**Which Azure service is used to store environment-labeled key-value configuration settings?**

- A) Azure Key Vault
- B) Azure App Configuration
- C) Azure Blob Storage
- D) Azure Cosmos DB

**Correct Answer:** B

**Explanation:** Azure App Configuration provides a centralized key-value store with support for labels. Labels allow the same key to have different values per environment (e.g., `Agent:ModelDeployment` with labels `dev`, `staging`, and `prod`). Azure Key Vault is used specifically for secrets, not general configuration.

---

### Question 6

**What is the difference between Active-Passive and Active-Active multi-region deployment strategies?**

- A) Active-Passive has lower latency; Active-Active has higher latency
- B) Active-Passive serves traffic from one region with a standby; Active-Active serves traffic from all regions simultaneously
- C) Active-Passive requires Azure Front Door; Active-Active does not
- D) Active-Passive is more expensive; Active-Active is cheaper

**Correct Answer:** B

**Explanation:** In Active-Passive, one primary region handles all traffic while a secondary remains on standby for failover. In Active-Active, all regions serve live traffic simultaneously via a global load balancer like Azure Front Door. Active-Active provides near-zero RTO but costs more because all regions run at full capacity.

---

### Question 7

**In a canary deployment on Azure Container Apps, what is the recommended traffic progression?**

- A) 0% → 100% (instant cutover)
- B) 50% → 100% (two-step)
- C) 10% → 50% → 100% (gradual shift with validation at each step)
- D) 100% → 0% → 100% (blue-green swap)

**Correct Answer:** C

**Explanation:** Canary deployments use a gradual traffic shift — starting with a small percentage (10%), validating behavior with automated evaluation, then increasing to 50% and finally 100%. This approach minimizes the blast radius if issues are discovered and allows rollback at any step.

---

### Question 8

**What is the purpose of a `/health` endpoint in a multi-region AI agent deployment?**

- A) To display the application version number to end users
- B) To enable load balancers to detect endpoint failures and reroute traffic
- C) To provide model fine-tuning status updates
- D) To expose configuration values for debugging

**Correct Answer:** B

**Explanation:** A `/health` endpoint is probed by load balancers (Azure Front Door, APIM) to verify that the agent is operational. It checks connectivity to AI Foundry, databases, and other dependencies. If it returns a non-200 status, the load balancer automatically reroutes traffic to a healthy region.

---

### Question 9

**When using Azure API Management as an AI gateway, which policy handles automatic failover between regions on throttling (429) errors?**

- A) `<cache-lookup>` policy
- B) `<rate-limit>` policy
- C) `<retry>` policy with `<set-backend-service>` to alternate backends
- D) `<validate-jwt>` policy

**Correct Answer:** C

**Explanation:** The `<retry>` policy in APIM allows you to define retry behavior on specific status codes (429, 503). Combined with `<set-backend-service>`, you can cycle through multiple backend URLs across regions, effectively implementing transparent failover when one region is throttled or unavailable.

---

### Question 10

**Why should monitoring thresholds differ between Dev, Staging, and Production environments?**

- A) Different environments have different billing rates
- B) Dev and Staging are expected to have more errors and higher latency, so thresholds should be more relaxed to avoid noise
- C) Production thresholds should be more relaxed because it handles more traffic
- D) Monitoring thresholds should be identical across all environments

**Correct Answer:** B

**Explanation:** Development environments have frequent changes, debugging, and experimentation — resulting in higher error rates and latency. Setting production-level thresholds in dev would create alert noise and desensitize teams. Production thresholds should be strict (e.g., P95 latency < 3s, error rate < 0.5%) to catch real issues affecting users.

---

## Scoring Guide

| Score | Rating | Recommendation |
|-------|--------|----------------|
| 10/10 | ⭐ Excellent | Ready for Module 30 |
| 8-9/10 | ✅ Good | Review missed topics, proceed to Module 30 |
| 7/10 | 📘 Passing | Review the blog content for missed areas |
| 5-6/10 | ⚠️ Needs Review | Re-read the module and retry the quiz |
| Below 5 | 🔄 Retake | Review Module 29 thoroughly before proceeding |
