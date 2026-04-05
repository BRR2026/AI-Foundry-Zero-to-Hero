# Module 35: Securing AI Workloads End-to-End

## Training Guide

| Field | Details |
|---|---|
| **Module** | 35 of 45 |
| **Title** | Securing AI Workloads End-to-End |
| **Arc** | ARC 7 — Security, Governance & Compliance |
| **Duration** | 4 hours (lecture + hands-on) |
| **Level** | Intermediate to Advanced |
| **Date** | April 2026 |
| **Prerequisites** | Module 34 — Responsible AI Governance |

---

## Learning Objectives

By the end of this module, learners will be able to:

1. **Design** a Virtual Network (VNet) topology for Azure AI Foundry with properly segmented subnets
2. **Implement** Private Endpoints for AI Foundry Hub, Storage, Key Vault, Container Registry, and AI Services
3. **Configure** Network Security Groups (NSGs) with defense-in-depth rules and service tags
4. **Deploy** Azure Firewall for centralized egress filtering with FQDN-based application rules
5. **Manage** secrets, keys, and certificates using Azure Key Vault with RBAC authorization
6. **Validate** network isolation by verifying DNS resolution returns private IP addresses
7. **Troubleshoot** common networking and access issues in secured AI Foundry deployments

---

## Module Outline

### Section 1: Understanding the AI Security Landscape (30 min)

#### Topics
- Why AI workloads require specialized security controls
- Traditional network threats vs. AI-specific attack vectors
- Defense-in-depth strategy for AI platforms
- Microsoft Shared Responsibility Model for AI services

#### Key Concepts
- **Data exfiltration risk** — training data and model weights are high-value targets
- **Model extraction attacks** — repeated inference calls to reverse-engineer models
- **Credential theft** — API keys stored insecurely in code or environment variables
- **Lateral movement** — compromised VMs pivoting to AI service endpoints

#### Discussion Points
- What data does your organization's AI workload process?
- What would be the impact of a data breach involving your training data?
- How does your current security posture protect AI endpoints?

---

### Section 2: Virtual Network (VNet) Integration (45 min)

#### Topics
- VNet fundamentals for AI workloads
- Subnet planning and IP address allocation
- VNet integration modes: bring-your-own vs. managed VNet
- Hub-spoke topology for enterprise AI deployments

#### Subnet Planning Guide

| Subnet | Purpose | Recommended CIDR | Min IPs |
|---|---|---|---|
| `snet-ai-hub` | AI Foundry Hub & Projects | /24 (256 IPs) | 8 |
| `snet-compute` | Compute instances & clusters | /23 (512 IPs) | 16 |
| `snet-storage` | Storage Private Endpoints | /26 (64 IPs) | 4 |
| `snet-keyvault` | Key Vault Private Endpoints | /27 (32 IPs) | 2 |
| `snet-acr` | Container Registry endpoints | /27 (32 IPs) | 2 |

#### Hands-On: Create VNet with Azure CLI

```bash
# Define variables
RG="rg-ai-foundry-secure"
LOCATION="eastus2"
VNET_NAME="vnet-ai-foundry"

# Create resource group
az group create --name $RG --location $LOCATION

# Create VNet
az network vnet create \
  --resource-group $RG \
  --name $VNET_NAME \
  --address-prefix "10.1.0.0/16" \
  --location $LOCATION

# Create subnets
az network vnet subnet create \
  --resource-group $RG \
  --vnet-name $VNET_NAME \
  --name "snet-ai-hub" \
  --address-prefixes "10.1.1.0/24"

az network vnet subnet create \
  --resource-group $RG \
  --vnet-name $VNET_NAME \
  --name "snet-compute" \
  --address-prefixes "10.1.2.0/23"
```

#### Key Takeaways
- Always plan subnet sizes before deployment — VNets cannot be resized without downtime
- Use separate subnets for different AI Foundry components
- Enable `privateEndpointNetworkPolicies: Disabled` on subnets that will host Private Endpoints

---

### Section 3: Private Endpoints for AI Services (60 min)

#### Topics
- How Azure Private Link works
- Private Endpoints vs. Service Endpoints
- DNS configuration with Private DNS Zones
- Disabling public access after Private Endpoint creation

#### Services Requiring Private Endpoints

| Service | Group ID | Private DNS Zone |
|---|---|---|
| AI Foundry Hub | `amlworkspace` | `privatelink.api.azureml.ms` |
| Storage (Blob) | `blob` | `privatelink.blob.core.windows.net` |
| Storage (File) | `file` | `privatelink.file.core.windows.net` |
| Key Vault | `vault` | `privatelink.vaultcore.azure.net` |
| Container Registry | `registry` | `privatelink.azurecr.io` |
| Azure AI Services | `account` | `privatelink.cognitiveservices.azure.com` |

#### Hands-On: Create Private Endpoints

```bash
# Get resource ID
HUB_ID=$(az resource show \
  --resource-group $RG \
  --name "hub-ai-foundry" \
  --resource-type "Microsoft.MachineLearningServices/workspaces" \
  --query "id" -o tsv)

# Create private endpoint
az network private-endpoint create \
  --resource-group $RG \
  --name "pe-ai-hub" \
  --vnet-name $VNET_NAME \
  --subnet "snet-ai-hub" \
  --private-connection-resource-id $HUB_ID \
  --group-id "amlworkspace" \
  --connection-name "conn-ai-hub"

# Configure Private DNS Zone
az network private-dns zone create \
  --resource-group $RG \
  --name "privatelink.api.azureml.ms"

# Link DNS zone to VNet
az network private-dns link vnet create \
  --resource-group $RG \
  --zone-name "privatelink.api.azureml.ms" \
  --name "link-ai-foundry" \
  --virtual-network $VNET_NAME \
  --registration-enabled false
```

#### Validation Exercise
- Use `nslookup` from inside the VNet to verify DNS resolution returns a private IP
- Attempt to access the AI Foundry Hub from outside the VNet — it should fail
- Review the Private Endpoint's network interface to confirm the assigned private IP

#### Key Takeaways
- Private Endpoints provide private IP connectivity — no data leaves the Microsoft backbone
- Always configure Private DNS Zones for correct name resolution
- Disable public access after Private Endpoint creation to eliminate attack surface

---

### Section 4: Network Security Groups & Firewall Rules (45 min)

#### Topics
- NSG fundamentals: inbound vs. outbound rules, priority, evaluation order
- Azure service tags for simplified rule management
- Required outbound rules for AI Foundry compute
- Azure Firewall for centralized egress control
- NSG Flow Logs for auditing and compliance

#### Essential NSG Rules

| Priority | Name | Direction | Source | Destination | Port | Action |
|---|---|---|---|---|---|---|
| 100 | AllowAzureAD | Outbound | VNet | AzureActiveDirectory | 443 | Allow |
| 110 | AllowARM | Outbound | VNet | AzureResourceManager | 443 | Allow |
| 120 | AllowStorage | Outbound | VNet | Storage.EastUS2 | 443 | Allow |
| 130 | AllowKeyVault | Outbound | VNet | AzureKeyVault | 443 | Allow |
| 140 | AllowACR | Outbound | VNet | AzureContainerRegistry | 443 | Allow |
| 150 | AllowMonitor | Outbound | VNet | AzureMonitor | 443 | Allow |
| 4096 | DenyAllOutbound | Outbound | * | * | * | Deny |

#### Key Takeaways
- Start with default-deny and explicitly allow required services
- Use service tags instead of hardcoded IP ranges
- NSG Flow Logs are essential for security auditing
- Azure Firewall provides FQDN-based filtering and threat intelligence

---

### Section 5: Key Vault for Secrets Management (45 min)

#### Topics
- Key Vault integration with AI Foundry Hub
- RBAC vs. access policies (prefer RBAC)
- Storing and retrieving secrets programmatically
- Managed identity authentication (no keys in code)
- Key Vault monitoring and diagnostic logging

#### RBAC Roles for Key Vault

| Role | Permissions | Use Case |
|---|---|---|
| Key Vault Administrator | Full management | Vault configuration |
| Key Vault Secrets Officer | CRUD on secrets | Storing secrets |
| Key Vault Secrets User | Read secrets | Runtime access |
| Key Vault Crypto Officer | CRUD on keys | Encryption keys |
| Key Vault Certificates Officer | CRUD on certs | TLS certificates |

#### Hands-On: Python SDK Integration

```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

credential = DefaultAzureCredential()
vault_url = "https://kv-ai-foundry-secure.vault.azure.net"
client = SecretClient(vault_url=vault_url, credential=credential)

# Store a secret
client.set_secret("openai-api-key", "sk-your-key-here")

# Retrieve a secret
secret = client.get_secret("openai-api-key")
print(f"Secret retrieved: {secret.name}")
```

#### Key Takeaways
- Always use RBAC authorization for new Key Vault deployments
- Enable soft-delete and purge protection for compliance
- Use managed identities to eliminate the need for API keys in code
- Enable diagnostic logging for audit trails

---

### Section 6: Mini Hack — Secure Deployment (60 min)

#### Objective
Deploy a fully secured AI Foundry Hub inside a VNet with Private Endpoints for all dependent services.

#### Steps
1. Create resource group and VNet with subnets
2. Deploy Key Vault, Storage Account, and Container Registry with public access disabled
3. Create Private Endpoints for each service
4. Configure Private DNS Zones and link to VNet
5. Deploy a test VM and validate DNS resolution
6. Clean up all resources

> **Refer to the hands-on lab** (`lab/hands-on-lab.md`) for detailed step-by-step instructions.

---

## Assessment

Complete the **10-question assessment** in `quiz/assessment.md` to validate your understanding.

**Passing score:** 80% (8 out of 10 correct)

---

## Video Resource

📺 **AI Foundry Networking and Security: Deep Dive**

[![Video Thumbnail](https://img.youtube.com/vi/bL9n83uPV84/maxresdefault.jpg)](https://www.youtube.com/watch?v=bL9n83uPV84)

Watch the full session covering VNet integration, Private Endpoints, firewall configuration, and Key Vault secrets management.

---

## Additional Resources

- [Azure AI Foundry Network Isolation](https://learn.microsoft.com/azure/ai-studio/how-to/configure-private-link)
- [Azure Private Link Documentation](https://learn.microsoft.com/azure/private-link/private-link-overview)
- [Azure Key Vault Best Practices](https://learn.microsoft.com/azure/key-vault/general/best-practices)
- [NSG Security Rules Reference](https://learn.microsoft.com/azure/virtual-network/network-security-groups-overview)
- [Azure Firewall FQDN Tags](https://learn.microsoft.com/azure/firewall/fqdn-tags)

---

## Navigation

| Previous | Current | Next |
|---|---|---|
| [← Module 34: Responsible AI Governance](../module-34/blog.html) | **Module 35: Securing AI Workloads** | [Module 36: Cost Management & FinOps →](../module-36/blog.html) |

---

*Azure AI Foundry: Zero to Hero Training Program — Module 35 of 45*
*ARC 7: Security, Governance & Compliance — April 2026*
