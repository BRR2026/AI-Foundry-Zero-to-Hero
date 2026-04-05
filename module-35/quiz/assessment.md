# Module 35: Securing AI Workloads End-to-End

## Assessment — 10 Multiple Choice Questions

| Field | Details |
|---|---|
| **Module** | 35 of 45 |
| **Arc** | ARC 7 — Security, Governance & Compliance |
| **Passing Score** | 80% (8 out of 10 correct) |
| **Time Limit** | 20 minutes |
| **Topic Areas** | VNet Integration, Private Endpoints, NSGs, Firewall, Key Vault |

---

### Question 1

**What is the primary benefit of deploying Azure AI Foundry resources inside a Virtual Network (VNet)?**

- A) It reduces the cost of compute instances by 30%
- B) It isolates resources so that traffic stays within the Microsoft backbone network and never traverses the public internet
- C) It automatically enables GPU acceleration for all compute instances
- D) It eliminates the need for authentication between services

<details>
<summary>Show Answer</summary>

**✅ B) It isolates resources so that traffic stays within the Microsoft backbone network and never traverses the public internet**

VNet integration ensures that all communication between AI Foundry components uses private IP addresses within the VNet. Traffic flows over the Microsoft backbone network, eliminating exposure to the public internet and reducing the risk of data interception or exfiltration.

</details>

---

### Question 2

**When creating a subnet for Private Endpoints, which property must be configured?**

- A) `enableDelegation` must be set to `true`
- B) `privateEndpointNetworkPolicies` must be set to `Disabled`
- C) `serviceEndpoints` must include `Microsoft.Network`
- D) `addressPrefix` must be at least a /16 CIDR block

<details>
<summary>Show Answer</summary>

**✅ B) `privateEndpointNetworkPolicies` must be set to `Disabled`**

Azure network policies (such as NSG rules and UDRs) are not applied to Private Endpoints by default. The subnet must have `privateEndpointNetworkPolicies` set to `Disabled` to allow Private Endpoint network interfaces to be created. Without this setting, Private Endpoint creation will fail.

</details>

---

### Question 3

**Which Private DNS Zone is required for an AI Foundry Hub Private Endpoint?**

- A) `privatelink.azureml.net`
- B) `privatelink.api.azureml.ms`
- C) `privatelink.aifoundry.azure.com`
- D) `privatelink.machinelearning.azure.net`

<details>
<summary>Show Answer</summary>

**✅ B) `privatelink.api.azureml.ms`**

The AI Foundry Hub (which is built on the Machine Learning Services platform) uses the `privatelink.api.azureml.ms` Private DNS Zone. This zone ensures that the Hub's FQDN resolves to the private IP address of the Private Endpoint rather than the public IP.

</details>

---

### Question 4

**What is the difference between a Service Endpoint and a Private Endpoint?**

- A) Service Endpoints are free; Private Endpoints have a per-hour cost
- B) Service Endpoints route traffic via an optimized path but the service retains a public IP; Private Endpoints assign a private IP from your subnet to the service
- C) Service Endpoints work only in the same region; Private Endpoints work across regions
- D) Service Endpoints are for PaaS services; Private Endpoints are for IaaS services

<details>
<summary>Show Answer</summary>

**✅ B) Service Endpoints route traffic via an optimized path but the service retains a public IP; Private Endpoints assign a private IP from your subnet to the service**

Service Endpoints optimize the routing path from your VNet to the Azure service, but the service still has a public IP and is technically reachable from the internet. Private Endpoints create a network interface in your subnet with a private IP, making the service accessible only via that private address. This is why Private Endpoints provide stronger isolation.

</details>

---

### Question 5

**In an NSG with the following outbound rules, what happens when a compute instance tries to access a third-party API at `api.example.com` on port 443?**

| Priority | Name | Destination | Port | Action |
|---|---|---|---|---|
| 100 | AllowAzureAD | AzureActiveDirectory | 443 | Allow |
| 110 | AllowStorage | Storage | 443 | Allow |
| 4096 | DenyAllOutbound | * | * | Deny |

- A) The traffic is allowed because port 443 is permitted by the AllowAzureAD rule
- B) The traffic is allowed because HTTPS is always permitted by default
- C) The traffic is denied by the DenyAllOutbound rule because `api.example.com` doesn't match any allow rule's service tag
- D) The traffic is allowed because the default Azure NSG rules allow outbound internet access

<details>
<summary>Show Answer</summary>

**✅ C) The traffic is denied by the DenyAllOutbound rule because `api.example.com` doesn't match any allow rule's service tag**

NSG rules are evaluated in priority order (lowest number first). The traffic to `api.example.com` doesn't match `AzureActiveDirectory` (priority 100) or `Storage` (priority 110), so it falls through to the `DenyAllOutbound` rule at priority 4096, which denies all remaining outbound traffic. This implements the "default deny" security principle.

</details>

---

### Question 6

**Which Azure RBAC role should be assigned to an AI Foundry compute instance's managed identity for reading secrets from Key Vault at runtime?**

- A) Key Vault Administrator
- B) Key Vault Secrets Officer
- C) Key Vault Secrets User
- D) Key Vault Contributor

<details>
<summary>Show Answer</summary>

**✅ C) Key Vault Secrets User**

The **Key Vault Secrets User** role grants read-only access to secrets, which is the minimum permission needed for runtime access. Following the principle of least privilege, compute instances should not have write access (Secrets Officer) or full management access (Administrator). The Key Vault Contributor role manages the vault resource itself but does not grant access to the data plane (secrets, keys, certificates).

</details>

---

### Question 7

**Why is Azure Firewall recommended over NSGs alone for enterprise AI Foundry deployments?**

- A) Azure Firewall is free for AI Foundry customers
- B) Azure Firewall supports FQDN-based application rules, threat intelligence, and centralized logging, which NSGs cannot provide
- C) NSGs are deprecated and will be removed in future Azure releases
- D) Azure Firewall automatically creates Private Endpoints for all services

<details>
<summary>Show Answer</summary>

**✅ B) Azure Firewall supports FQDN-based application rules, threat intelligence, and centralized logging, which NSGs cannot provide**

NSGs operate at Layer 3/4 (IP addresses and ports) and use service tags for destination matching. Azure Firewall adds Layer 7 capabilities including FQDN-based filtering (e.g., allow `*.api.azureml.ms`), threat intelligence feeds, TLS inspection, and centralized logging. For enterprise deployments, this provides more granular control over egress traffic.

</details>

---

### Question 8

**What should you do AFTER creating Private Endpoints for all dependent services of an AI Foundry Hub?**

- A) Delete the VNet to reduce costs
- B) Enable public network access on all resources
- C) Disable public network access on all resources to ensure traffic flows exclusively through Private Endpoints
- D) Remove the managed identity from the AI Foundry Hub

<details>
<summary>Show Answer</summary>

**✅ C) Disable public network access on all resources to ensure traffic flows exclusively through Private Endpoints**

Creating a Private Endpoint does not automatically disable the public endpoint. Both public and private access can coexist. To ensure all traffic flows exclusively through Private Endpoints (and eliminate the public attack surface), you must explicitly disable public network access on each resource using `--public-network-access Disabled`.

</details>

---

### Question 9

**A developer reports that they cannot access the AI Foundry portal after Private Endpoints were configured and public access was disabled. What is the most likely cause?**

- A) The AI Foundry Hub needs to be redeployed with a new name
- B) The developer is accessing from outside the VNet and needs to connect via VPN, ExpressRoute, or Azure Bastion first
- C) Private Endpoints do not support web portal access
- D) The developer's Azure AD account has been locked

<details>
<summary>Show Answer</summary>

**✅ B) The developer is accessing from outside the VNet and needs to connect via VPN, ExpressRoute, or Azure Bastion first**

When public network access is disabled, the AI Foundry portal is only accessible from within the VNet (or from networks connected to it). Developers must connect via VPN, ExpressRoute private peering, or Azure Bastion before they can access the portal. Their DNS must also resolve the Hub's FQDN to the private IP via the Private DNS Zone.

</details>

---

### Question 10

**Which of the following is a best practice for managing API keys used by AI Foundry to connect to Azure AI Services?**

- A) Store them as environment variables on each compute instance
- B) Hardcode them in the application configuration file and encrypt the file
- C) Store them in Azure Key Vault and access them at runtime using managed identities
- D) Share them via a secure email to the development team

<details>
<summary>Show Answer</summary>

**✅ C) Store them in Azure Key Vault and access them at runtime using managed identities**

Azure Key Vault is the recommended store for all secrets, API keys, and connection strings. Combined with managed identities, this approach eliminates the need to embed credentials in code, configuration files, or environment variables. Key Vault provides RBAC-based access control, audit logging, soft-delete protection, and automatic secret rotation — none of which are available with the other approaches.

</details>

---

## Scoring Guide

| Score | Result | Recommendation |
|---|---|---|
| 10/10 | ⭐ Excellent | Ready for Module 36 |
| 8–9/10 | ✅ Pass | Review missed topics, then proceed |
| 6–7/10 | ⚠️ Needs Review | Re-read the blog post and retry |
| 0–5/10 | ❌ Retake | Complete the hands-on lab before retaking |

---

## Answer Key (Quick Reference)

| Q | Answer | Topic |
|---|---|---|
| 1 | B | VNet Integration |
| 2 | B | Private Endpoints — Subnet Config |
| 3 | B | Private DNS Zones |
| 4 | B | Service Endpoints vs. Private Endpoints |
| 5 | C | NSG Rule Evaluation |
| 6 | C | Key Vault RBAC Roles |
| 7 | B | Azure Firewall vs. NSGs |
| 8 | C | Post-PE Configuration |
| 9 | B | Troubleshooting Private Access |
| 10 | C | Secrets Management Best Practices |

---

*Module 35 — Assessment | Azure AI Foundry: Zero to Hero | ARC 7: Security, Governance & Compliance*
