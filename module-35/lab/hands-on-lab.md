# Module 35 — Hands-On Lab

## Deploy Azure AI Foundry with Private Endpoints in a VNet

| Field | Details |
|---|---|
| **Module** | 35 — Securing AI Workloads End-to-End |
| **Arc** | ARC 7 — Security, Governance & Compliance |
| **Duration** | 60–90 minutes |
| **Level** | Intermediate |
| **Objective** | Deploy a fully network-isolated AI Foundry environment with Private Endpoints for all dependent services |

---

## Prerequisites

- [x] Azure subscription with **Contributor** and **User Access Administrator** roles
- [x] Azure CLI v2.60+ installed (`az --version`)
- [x] Completed Module 34 (Responsible AI Governance)
- [x] Basic understanding of networking (IP addressing, subnets, DNS)

---

## Lab Architecture

```
┌────────────────────────────────────────────────────────────────┐
│  Resource Group: rg-ai-foundry-secure-lab                      │
│                                                                │
│  ┌──────────────── VNet: vnet-ai-secure (10.0.0.0/16) ──────┐ │
│  │                                                            │ │
│  │  ┌─────────────┐ ┌──────────────┐ ┌─────────────────────┐ │ │
│  │  │ snet-hub    │ │ snet-compute │ │ snet-pe             │ │ │
│  │  │ 10.0.1.0/24 │ │ 10.0.2.0/23  │ │ 10.0.4.0/24        │ │ │
│  │  │             │ │              │ │                     │ │ │
│  │  │ • Test VM   │ │ (Reserved)   │ │ • PE Key Vault      │ │ │
│  │  │             │ │              │ │ • PE Storage (Blob)  │ │ │
│  │  │             │ │              │ │ • PE ACR             │ │ │
│  │  └─────────────┘ └──────────────┘ └─────────────────────┘ │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                │
│  ┌────────────┐  ┌───────────────┐  ┌──────────┐              │
│  │ Key Vault  │  │ Storage Acct  │  │ ACR      │              │
│  │ (no pub)   │  │ (no pub)      │  │ (no pub) │              │
│  └────────────┘  └───────────────┘  └──────────┘              │
│                                                                │
│  Private DNS Zones:                                            │
│  • privatelink.vaultcore.azure.net                             │
│  • privatelink.blob.core.windows.net                           │
│  • privatelink.azurecr.io                                      │
└────────────────────────────────────────────────────────────────┘
```

---

## Exercise 1: Create the Network Foundation

**Objective:** Create a VNet with properly segmented subnets for an AI Foundry deployment.

### Step 1.1 — Set Up Variables

```bash
# Define all variables for the lab
export RG="rg-ai-foundry-secure-lab"
export LOCATION="eastus2"
export VNET_NAME="vnet-ai-secure"
export KV_NAME="kv-aiseclab-$RANDOM"
export STORAGE_NAME="staiseclab$RANDOM"
export ACR_NAME="acraiseclab$RANDOM"

# Display variable values for reference
echo "Resource Group:  $RG"
echo "Location:        $LOCATION"
echo "VNet:            $VNET_NAME"
echo "Key Vault:       $KV_NAME"
echo "Storage:         $STORAGE_NAME"
echo "ACR:             $ACR_NAME"
```

### Step 1.2 — Create Resource Group

```bash
az group create \
  --name $RG \
  --location $LOCATION \
  --output table
```

**Expected output:** Resource group created with `Succeeded` provisioning state.

### Step 1.3 — Create Virtual Network

```bash
az network vnet create \
  --resource-group $RG \
  --name $VNET_NAME \
  --address-prefix "10.0.0.0/16" \
  --location $LOCATION \
  --output table
```

### Step 1.4 — Create Subnets

```bash
# Hub subnet (for test VM and AI Foundry Hub)
az network vnet subnet create \
  --resource-group $RG \
  --vnet-name $VNET_NAME \
  --name "snet-hub" \
  --address-prefixes "10.0.1.0/24" \
  --output table

# Compute subnet (reserved for AI Foundry compute)
az network vnet subnet create \
  --resource-group $RG \
  --vnet-name $VNET_NAME \
  --name "snet-compute" \
  --address-prefixes "10.0.2.0/23" \
  --output table

# Private Endpoint subnet
az network vnet subnet create \
  --resource-group $RG \
  --vnet-name $VNET_NAME \
  --name "snet-pe" \
  --address-prefixes "10.0.4.0/24" \
  --private-endpoint-network-policies Disabled \
  --output table
```

### ✅ Checkpoint 1

Verify the VNet and subnets were created:

```bash
az network vnet subnet list \
  --resource-group $RG \
  --vnet-name $VNET_NAME \
  --output table
```

**Expected:** Three subnets listed (`snet-hub`, `snet-compute`, `snet-pe`).

> **🤔 Knowledge Check:** Why do we set `--private-endpoint-network-policies Disabled` on the `snet-pe` subnet?
>
> **Answer:** Private Endpoints require this setting to allow the PE network interface to be created in the subnet. Without it, Azure's network policies block PE creation.

---

## Exercise 2: Deploy Services with Public Access Disabled

**Objective:** Deploy Key Vault, Storage, and Container Registry with public network access disabled from the start.

### Step 2.1 — Create Key Vault

```bash
az keyvault create \
  --resource-group $RG \
  --name $KV_NAME \
  --location $LOCATION \
  --enable-rbac-authorization true \
  --enable-purge-protection true \
  --enable-soft-delete true \
  --soft-delete-retention-days 90 \
  --public-network-access Disabled \
  --output table
```

### Step 2.2 — Create Storage Account

```bash
az storage account create \
  --resource-group $RG \
  --name $STORAGE_NAME \
  --location $LOCATION \
  --sku Standard_LRS \
  --kind StorageV2 \
  --default-action Deny \
  --public-network-access Disabled \
  --min-tls-version TLS1_2 \
  --output table
```

### Step 2.3 — Create Container Registry

```bash
az acr create \
  --resource-group $RG \
  --name $ACR_NAME \
  --location $LOCATION \
  --sku Premium \
  --public-network-access Disabled \
  --output table
```

> **Note:** ACR requires the **Premium** SKU for Private Endpoint support.

### ✅ Checkpoint 2

Verify all services are deployed with public access disabled:

```bash
# Check Key Vault
echo "Key Vault public access:"
az keyvault show --name $KV_NAME --query "properties.publicNetworkAccess" -o tsv

# Check Storage
echo "Storage public access:"
az storage account show -g $RG -n $STORAGE_NAME --query "publicNetworkAccess" -o tsv

# Check ACR
echo "ACR public access:"
az acr show -g $RG -n $ACR_NAME --query "publicNetworkAccess" -o tsv
```

**Expected:** All three should return `Disabled`.

---

## Exercise 3: Create Private Endpoints

**Objective:** Create Private Endpoints for Key Vault, Storage (Blob), and Container Registry to enable private connectivity.

### Step 3.1 — Get Resource IDs

```bash
export KV_ID=$(az keyvault show -g $RG -n $KV_NAME --query id -o tsv)
export ST_ID=$(az storage account show -g $RG -n $STORAGE_NAME --query id -o tsv)
export ACR_ID=$(az acr show -g $RG -n $ACR_NAME --query id -o tsv)

echo "Key Vault ID:  $KV_ID"
echo "Storage ID:    $ST_ID"
echo "ACR ID:        $ACR_ID"
```

### Step 3.2 — Create Key Vault Private Endpoint

```bash
az network private-endpoint create \
  --resource-group $RG \
  --name "pe-keyvault" \
  --vnet-name $VNET_NAME \
  --subnet "snet-pe" \
  --private-connection-resource-id $KV_ID \
  --group-id "vault" \
  --connection-name "conn-kv" \
  --output table
```

### Step 3.3 — Create Storage (Blob) Private Endpoint

```bash
az network private-endpoint create \
  --resource-group $RG \
  --name "pe-storage-blob" \
  --vnet-name $VNET_NAME \
  --subnet "snet-pe" \
  --private-connection-resource-id $ST_ID \
  --group-id "blob" \
  --connection-name "conn-blob" \
  --output table
```

### Step 3.4 — Create ACR Private Endpoint

```bash
az network private-endpoint create \
  --resource-group $RG \
  --name "pe-acr" \
  --vnet-name $VNET_NAME \
  --subnet "snet-pe" \
  --private-connection-resource-id $ACR_ID \
  --group-id "registry" \
  --connection-name "conn-acr" \
  --output table
```

### ✅ Checkpoint 3

List all Private Endpoints and verify they are in `Succeeded` state:

```bash
az network private-endpoint list \
  --resource-group $RG \
  --query "[].{Name:name, Subnet:subnet.id, Status:provisioningState}" \
  --output table
```

**Expected:** Three PEs listed, all with `Succeeded` status.

Inspect the private IP assigned to each PE:

```bash
az network private-endpoint list \
  --resource-group $RG \
  --query "[].{Name:name, PrivateIP:customDnsConfigs[0].ipAddresses[0]}" \
  --output table
```

**Expected:** Each PE has a private IP in the `10.0.4.x` range.

---

## Exercise 4: Configure Private DNS Zones

**Objective:** Create Private DNS Zones to ensure DNS queries resolve to private IPs instead of public IPs.

### Step 4.1 — Create DNS Zones

```bash
# Key Vault DNS Zone
az network private-dns zone create \
  --resource-group $RG \
  --name "privatelink.vaultcore.azure.net" \
  --output table

# Storage Blob DNS Zone
az network private-dns zone create \
  --resource-group $RG \
  --name "privatelink.blob.core.windows.net" \
  --output table

# ACR DNS Zone
az network private-dns zone create \
  --resource-group $RG \
  --name "privatelink.azurecr.io" \
  --output table
```

### Step 4.2 — Link DNS Zones to VNet

```bash
# Link Key Vault DNS Zone
az network private-dns link vnet create \
  --resource-group $RG \
  --zone-name "privatelink.vaultcore.azure.net" \
  --name "link-keyvault" \
  --virtual-network $VNET_NAME \
  --registration-enabled false \
  --output table

# Link Storage DNS Zone
az network private-dns link vnet create \
  --resource-group $RG \
  --zone-name "privatelink.blob.core.windows.net" \
  --name "link-storage" \
  --virtual-network $VNET_NAME \
  --registration-enabled false \
  --output table

# Link ACR DNS Zone
az network private-dns link vnet create \
  --resource-group $RG \
  --zone-name "privatelink.azurecr.io" \
  --name "link-acr" \
  --virtual-network $VNET_NAME \
  --registration-enabled false \
  --output table
```

### Step 4.3 — Create DNS Zone Groups

```bash
# Key Vault DNS Zone Group
az network private-endpoint dns-zone-group create \
  --resource-group $RG \
  --endpoint-name "pe-keyvault" \
  --name "dzg-keyvault" \
  --private-dns-zone "privatelink.vaultcore.azure.net" \
  --zone-name "keyvault" \
  --output table

# Storage DNS Zone Group
az network private-endpoint dns-zone-group create \
  --resource-group $RG \
  --endpoint-name "pe-storage-blob" \
  --name "dzg-storage" \
  --private-dns-zone "privatelink.blob.core.windows.net" \
  --zone-name "blob" \
  --output table

# ACR DNS Zone Group
az network private-endpoint dns-zone-group create \
  --resource-group $RG \
  --endpoint-name "pe-acr" \
  --name "dzg-acr" \
  --private-dns-zone "privatelink.azurecr.io" \
  --zone-name "acr" \
  --output table
```

### ✅ Checkpoint 4

List DNS records in each zone:

```bash
echo "=== Key Vault DNS Records ==="
az network private-dns record-set list \
  --resource-group $RG \
  --zone-name "privatelink.vaultcore.azure.net" \
  --output table

echo ""
echo "=== Storage DNS Records ==="
az network private-dns record-set list \
  --resource-group $RG \
  --zone-name "privatelink.blob.core.windows.net" \
  --output table

echo ""
echo "=== ACR DNS Records ==="
az network private-dns record-set list \
  --resource-group $RG \
  --zone-name "privatelink.azurecr.io" \
  --output table
```

**Expected:** A-records pointing to private IPs (10.0.4.x) for each service.

---

## Exercise 5: Validate Network Isolation

**Objective:** Deploy a test VM inside the VNet and verify DNS resolution returns private IPs.

### Step 5.1 — Create Network Security Group

```bash
# Create NSG for the hub subnet
az network nsg create \
  --resource-group $RG \
  --name "nsg-hub" \
  --output table

# Allow outbound HTTPS to Azure services
az network nsg rule create \
  --resource-group $RG \
  --nsg-name "nsg-hub" \
  --name "AllowAzureServicesOutbound" \
  --priority 100 \
  --direction Outbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes "VirtualNetwork" \
  --destination-address-prefixes "AzureCloud" \
  --destination-port-ranges 443 \
  --output table

# Associate NSG with hub subnet
az network vnet subnet update \
  --resource-group $RG \
  --vnet-name $VNET_NAME \
  --name "snet-hub" \
  --network-security-group "nsg-hub" \
  --output table
```

### Step 5.2 — Deploy Test VM

```bash
az vm create \
  --resource-group $RG \
  --name "vm-test-dns" \
  --image "Ubuntu2204" \
  --size "Standard_B1ms" \
  --vnet-name $VNET_NAME \
  --subnet "snet-hub" \
  --public-ip-address "" \
  --admin-username "azureuser" \
  --generate-ssh-keys \
  --output table
```

> **Note:** The VM is deployed without a public IP. You will use `az vm run-command` to execute commands.

### Step 5.3 — Test DNS Resolution

```bash
# Test Key Vault DNS resolution
echo "=== Key Vault DNS ==="
az vm run-command invoke \
  --resource-group $RG \
  --name "vm-test-dns" \
  --command-id RunShellScript \
  --scripts "nslookup $KV_NAME.vault.azure.net" \
  --query "value[0].message" -o tsv

# Test Storage DNS resolution
echo ""
echo "=== Storage DNS ==="
az vm run-command invoke \
  --resource-group $RG \
  --name "vm-test-dns" \
  --command-id RunShellScript \
  --scripts "nslookup $STORAGE_NAME.blob.core.windows.net" \
  --query "value[0].message" -o tsv

# Test ACR DNS resolution
echo ""
echo "=== ACR DNS ==="
az vm run-command invoke \
  --resource-group $RG \
  --name "vm-test-dns" \
  --command-id RunShellScript \
  --scripts "nslookup $ACR_NAME.azurecr.io" \
  --query "value[0].message" -o tsv
```

### ✅ Checkpoint 5

**Expected Results:**

Each `nslookup` command should resolve to a **private IP** address in the `10.0.4.x` range:

```
Server:    127.0.0.53
Address:   127.0.0.53#53

Non-authoritative answer:
kv-aiseclab-12345.vault.azure.net  canonical name = kv-aiseclab-12345.privatelink.vaultcore.azure.net
Name:      kv-aiseclab-12345.privatelink.vaultcore.azure.net
Address:   10.0.4.4
```

If the DNS resolves to a **public IP**, check:
1. Is the Private DNS Zone linked to the VNet?
2. Was the DNS Zone Group created for the Private Endpoint?
3. Is the A-record present in the Private DNS Zone?

---

## Exercise 6: Clean Up Resources

**Objective:** Remove all lab resources to avoid ongoing charges.

```bash
# Delete the entire resource group (this removes everything)
az group delete \
  --name $RG \
  --yes \
  --no-wait

echo "Resource group deletion initiated. Resources will be removed in ~5 minutes."
```

> **⚠️ Important:** Key Vaults with soft-delete enabled will be retained for 90 days. To permanently purge:
> ```bash
> az keyvault purge --name $KV_NAME --location $LOCATION
> ```

---

## Challenge Exercises (Optional)

### Challenge 1: Add NSG Flow Logs
Enable NSG Flow Logs on `nsg-hub` and send them to a Log Analytics workspace. Write a KQL query to find all denied traffic.

### Challenge 2: Azure Firewall
Deploy Azure Firewall in the hub subnet and create User-Defined Routes to force all egress through the firewall. Add FQDN-based application rules for AI Foundry dependencies.

### Challenge 3: Key Vault with Python SDK
From the test VM, install the Azure Python SDK and write a script that:
1. Authenticates using managed identity
2. Stores a secret in Key Vault
3. Retrieves and displays the secret name

---

## Troubleshooting Guide

| Problem | Cause | Solution |
|---|---|---|
| PE creation fails | Network policies enabled | Set `--private-endpoint-network-policies Disabled` on subnet |
| DNS resolves to public IP | DNS Zone not linked | Create DNS Zone Group and link zone to VNet |
| "Access denied" on Key Vault | Missing RBAC role | Assign `Key Vault Secrets User` role to the identity |
| VM cannot reach Key Vault | NSG blocking outbound | Add outbound rule for `AzureKeyVault` service tag |
| ACR pull fails | Wrong SKU | ACR must be **Premium** SKU for Private Endpoints |

---

## Summary

In this lab, you:

1. ✅ Created a VNet with three segmented subnets
2. ✅ Deployed Key Vault, Storage, and ACR with public access disabled
3. ✅ Created Private Endpoints for private connectivity
4. ✅ Configured Private DNS Zones for correct name resolution
5. ✅ Validated network isolation using a test VM

These skills form the foundation for securing any Azure AI Foundry production deployment.

---

*Module 35 — Hands-On Lab | Azure AI Foundry: Zero to Hero | ARC 7: Security, Governance & Compliance*
