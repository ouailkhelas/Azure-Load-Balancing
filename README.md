# Azure Enterprise Infrastructure with Load Balancing & High Availability


> **Enterprise-grade Azure infrastructure featuring load-balanced VMs, VNet peering, private endpoints, backup solutions, and comprehensive monitoring with Azure Log Analytics.**

## 📋 Overview

This project deploys a complete enterprise Azure infrastructure with high availability, disaster recovery, security hardening, and monitoring capabilities. The architecture includes load-balanced virtual machines across multiple virtual networks with private endpoints for secure service access, automated backups, and real-time alerting.

## 🏗️ Architecture

![Architecture Diagram](RG-1_1_.png)

### Core Components

| Component | Count | Purpose |
|-----------|-------|---------|
| **Virtual Machines** | 2 | Load-balanced compute instances (vm1-LB, vm2-LB) |
| **Load Balancer** | 1 | Traffic distribution with public IP |
| **Virtual Networks** | 2 | Network isolation with peering (vnet-1, vnet-2) |
| **Private Endpoints** | 2 | Secure access to storage and backup services |
| **Storage Accounts** | 2 | Data storage with replication |
| **Recovery Services Vault** | 1 | Backup and disaster recovery |
| **Key Vault** | 1 | Secrets and certificate management |
| **Log Analytics Workspace** | 1 | Centralized monitoring and logging |
| **Network Security Groups** | 1 | Network traffic filtering |

## 🔄 Network Architecture

### Virtual Network Configuration

**VNet-1** (10.0.0.0/16)
- **subnet-1.1**: Application tier
- **subnet-1.2**: Data tier
- **Peering**: Connected to vnet-2

**VNet-2** (10.2.0.0/16)
- **Peering**: Connected to vnet-1
- **Purpose**: Isolated workload environment

### VNet Peering

- **Type**: Bidirectional peering
- **Traffic**: Forwarded traffic allowed
- **Status**: Connected and fully synchronized
- **Gateway Transit**: Disabled

## ⚖️ Load Balancer Configuration

### Frontend Configuration

- **Public IP**: Standard SKU, Static allocation
- **DNS Name**: puplic-ip-LB
- **Zone**: Regional (East US)

### Backend Pool

- **vm1-LB**: Primary compute instance
- **vm2-LB**: Secondary compute instance
- **Distribution**: Round-robin algorithm

### Health Probes

- Monitors VM health status
- Automatic failover on failure
- Configurable probe intervals

## 🔐 Security Features

### Private Endpoints

1. **Storage Private Endpoint**
   - Target: Storage account (stoaccounttotest)
   - Service: Blob storage
   - Network: vnet-1

2. **Backup Private Endpoint**
   - Target: Recovery Services Vault
   - Service: Azure Backup
   - Network: vnet-1

### Key Vault

- **Purpose**: Secure storage for:
  - SSH keys
  - Storage account keys
  - Application secrets
  - Certificates

### Network Security Groups (NSG)

- **Applied to**: vnet-1 subnets
- **Rules**: Customizable inbound/outbound rules
- **Default Action**: Deny all (whitelist approach)

### Authentication

- **SSH Keys**: 
  - vm1-LB_key
  - vm2-LB_key
  - vmmmmm_key
- **Key Type**: RSA 3072-bit
- **Management**: Azure-managed

## 💾 Storage & Backup

### Storage Accounts

**Primary Storage (stoaccounttotest)**
- **Access**: Private endpoint only
- **Replication**: Locally redundant (LRS)
- **Container**: `cont`
- **Public Access**: Disabled

**Replication Storage (testobjectreplication)**
- **SKU**: Standard_RAGRS (Read-Access Geo-Redundant)
- **Tier**: Hot
- **Access**: VNet service endpoints
- **Allowed Subnets**: subnet-1.1, subnet-1.2
- **Versioning**: Immutable storage enabled

### Recovery Services Vault

- **Name**: backup
- **Location**: East US
- **Features**:
  - VM backup policies
  - Point-in-time recovery
  - Long-term retention
  - Private endpoint connectivity

## 📊 Monitoring & Alerting

### Log Analytics Workspace

- **Name**: log-analytics
- **Purpose**: Centralized log aggregation
- **Data Sources**:
  - VM performance metrics
  - Storage account logs
  - Network flow logs
  - Security events

### Data Collection Rules

- **Name**: collection-rule
- **Scope**: All VMs and storage accounts
- **Metrics**:
  - CPU utilization
  - Memory usage
  - Disk I/O
  - Network throughput

### Action Groups & Alerts

**Action Group**: group-action
- **Email Notifications**: mokhtarouailkhelas@gmail.com
- **ARM Role Receivers**: Owner role notifications
- **Short Name**: notification

**Activity Log Alert**: alert-vm1
- **Scope**: VM1 monitoring
- **Triggers**:
  - VM stopped/deallocated
  - VM state changes
  - Resource health changes

## 🚀 Deployment Guide

### Prerequisites

- Azure subscription with Owner or Contributor role
- Azure CLI version 2.30.0 or higher
- PowerShell 7.0+ (for PowerShell deployment)
- SSH client for VM access

### Option 1: Azure Portal Deployment

1. Navigate to **Azure Portal** → **Create a resource**
2. Search for **Template deployment (deploy using custom templates)**
3. Click **Build your own template in the editor**
4. Copy and paste the contents of `template.json`
5. Click **Save**
6. Fill in the required parameters:
   - Resource group name
   - Location (default: East US)
   - Storage account names
   - VM names
   - Key vault name
7. Review and create

### Option 2: Azure CLI Deployment

```bash
# Login to Azure
az login

# Create resource group
az group create \
  --name rg-enterprise-infrastructure \
  --location eastus

# Deploy template
az deployment group create \
  --resource-group rg-enterprise-infrastructure \
  --template-file template.json \
  --parameters \
    loadBalancers_LB_name="LB" \
    virtualNetworks_vnet_1_name="vnet-1" \
    virtualNetworks_vnet_2_name="vnet-2" \
    virtualMachines_vm1_LB_name="vm1-LB" \
    virtualMachines_vm2_LB_name="vm2-LB" \
    storageAccounts_stoaccounttotest_name="stoaccountprod" \
    vaults_backup_name="backup-vault"

# Monitor deployment
az deployment group show \
  --resource-group rg-enterprise-infrastructure \
  --name template
```

### Option 3: PowerShell Deployment

```powershell
# Connect to Azure
Connect-AzAccount

# Set variables
$resourceGroupName = "rg-enterprise-infrastructure"
$location = "eastus"
$templateFile = "template.json"

# Create resource group
New-AzResourceGroup `
  -Name $resourceGroupName `
  -Location $location

# Deploy template
New-AzResourceGroupDeployment `
  -ResourceGroupName $resourceGroupName `
  -TemplateFile $templateFile `
  -Mode Incremental `
  -Verbose

# Verify deployment
Get-AzResourceGroupDeployment `
  -ResourceGroupName $resourceGroupName
```

## ⚙️ Post-Deployment Configuration

### 1. Configure Load Balancer Rules

```bash
# Create load balancing rule for HTTP
az network lb rule create \
  --resource-group rg-enterprise-infrastructure \
  --lb-name LB \
  --name http-rule \
  --protocol tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name LoadBalancerFrontEnd \
  --backend-pool-name BackendPool
```

### 2. Enable VM Backup

```bash
# Enable backup for VM1
az backup protection enable-for-vm \
  --resource-group rg-enterprise-infrastructure \
  --vault-name backup \
  --vm vm1-LB \
  --policy-name DefaultPolicy

# Enable backup for VM2
az backup protection enable-for-vm \
  --resource-group rg-enterprise-infrastructure \
  --vault-name backup \
  --vm vm2-LB \
  --policy-name DefaultPolicy
```

### 3. Configure NSG Rules

```bash
# Allow HTTP traffic
az network nsg rule create \
  --resource-group rg-enterprise-infrastructure \
  --nsg-name NSG \
  --name allow-http \
  --priority 100 \
  --source-address-prefixes '*' \
  --destination-port-ranges 80 \
  --access Allow \
  --protocol Tcp

# Allow HTTPS traffic
az network nsg rule create \
  --resource-group rg-enterprise-infrastructure \
  --nsg-name NSG \
  --name allow-https \
  --priority 110 \
  --destination-port-ranges 443 \
  --access Allow \
  --protocol Tcp

# Allow SSH (restricted to your IP)
az network nsg rule create \
  --resource-group rg-enterprise-infrastructure \
  --nsg-name NSG \
  --name allow-ssh \
  --priority 120 \
  --source-address-prefixes "YOUR_IP_ADDRESS" \
  --destination-port-ranges 22 \
  --access Allow \
  --protocol Tcp
```

### 4. Add Secrets to Key Vault

```bash
# Store database connection string
az keyvault secret set \
  --vault-name key-vault-to-store-data \
  --name db-connection-string \
  --value "Server=myserver;Database=mydb;..."

# Store API key
az keyvault secret set \
  --vault-name key-vault-to-store-data \
  --name api-key \
  --value "your-api-key-here"
```

## 🧪 Testing & Validation

### Step 1: Verify Load Balancer

```bash
# Get load balancer public IP
LB_IP=$(az network public-ip show \
  --resource-group rg-enterprise-infrastructure \
  --name puplic-ip-LB \
  --query ipAddress \
  --output tsv)

echo "Load Balancer IP: $LB_IP"

# Test connectivity
curl http://$LB_IP
```

### Step 2: Verify VNet Peering

```bash
# Check peering status for vnet-1
az network vnet peering show \
  --resource-group rg-enterprise-infrastructure \
  --vnet-name vnet-1 \
  --name peering-link \
  --query peeringState

# Expected output: "Connected"
```

### Step 3: Test Private Endpoint Connectivity

```bash
# SSH into VM1
ssh -i ~/.ssh/vm1_key azureuser@<VM1_IP>

# From inside VM1, test storage access via private endpoint
nslookup stoaccounttotest.blob.core.windows.net
# Should resolve to a private IP (10.0.x.x)

# Test blob access
az storage blob list \
  --account-name stoaccounttotest \
  --container-name cont \
  --auth-mode login
```

### Step 4: Verify Monitoring

```bash
# Query Log Analytics
az monitor log-analytics query \
  --workspace log-analytics \
  --analytics-query "Perf | where TimeGenerated > ago(1h) | summarize avg(CounterValue) by Computer, CounterName"

# Check alerts
az monitor activity-log alert list \
  --resource-group rg-enterprise-infrastructure
```

## 📈 Monitoring Queries

### VM Performance

```kusto
Perf
| where TimeGenerated > ago(24h)
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| summarize avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
| render timechart
```

### Storage Operations

```kusto
StorageBlobLogs
| where TimeGenerated > ago(1h)
| summarize count() by OperationName, StatusCode
| order by count_ desc
```

### Network Flow

```kusto
AzureNetworkAnalytics_CL
| where TimeGenerated > ago(1h)
| summarize TotalBytes = sum(BytesSent_d + BytesReceived_d) by FlowDirection_s
```

## 🎯 Use Cases

### High Availability Web Applications
- Load-balanced VM tier for web servers
- Auto-scaling capabilities
- Health monitoring and automatic failover

### Secure Data Storage
- Private endpoint access to storage
- Geo-redundant replication
- Immutable versioning for compliance

### Disaster Recovery
- Automated VM backups
- Cross-region replication
- Point-in-time recovery

### Enterprise Compliance
- Network isolation with VNets
- Private connectivity to all services
- Audit logging and monitoring
- Secrets management with Key Vault

### Multi-Tier Applications
- Separate subnets for app and data tiers
- VNet peering for service communication
- NSG-based micro-segmentation

## 🔧 Customization Options

### Scale VMs

Modify the VM size in the template:

```json
"hardwareProfile": {
  "vmSize": "Standard_D4s_v3"  // Change to desired size
}
```

### Add More VMs to Load Balancer

1. Deploy additional VMs
2. Add to backend pool:

```bash
az network nic ip-config address-pool add \
  --resource-group rg-enterprise-infrastructure \
  --nic-name vm3-nic \
  --ip-config-name ipconfig1 \
  --lb-name LB \
  --address-pool BackendPool
```

### Configure Auto-Scaling

```bash
# Create VM scale set (alternative to individual VMs)
az vmss create \
  --resource-group rg-enterprise-infrastructure \
  --name vmss-web \
  --image Ubuntu2204 \
  --lb LB \
  --backend-pool-name BackendPool \
  --instance-count 2 \
  --autoscale
```

## 🔒 Security Best Practices

- ✅ Enable Azure Defender for all resources
- ✅ Implement Just-In-Time (JIT) VM access
- ✅ Use managed identities instead of credentials
- ✅ Enable disk encryption on all VMs
- ✅ Regular security assessments with Azure Security Center
- ✅ Implement Azure Policy for compliance
- ✅ Enable Microsoft Defender for Cloud
- ✅ Regular backup testing and disaster recovery drills

## 💰 Cost Optimization

### Estimated Monthly Cost (East US Region)

| Resource | Quantity | Estimated Cost |
|----------|----------|----------------|
| VMs (Standard_B2s) | 2 | ~$60 |
| Load Balancer (Standard) | 1 | ~$25 |
| Storage (RAGRS, 100GB) | 1 | ~$15 |
| Log Analytics (5GB/day) | 1 | ~$12 |
| Recovery Services | 1 | ~$10 |
| Key Vault | 1 | ~$3 |
| **Total** | | **~$125/month** |

### Cost Reduction Tips

1. **Use Reserved Instances**: Save up to 72% on VMs
2. **Auto-shutdown VMs**: Configure shutdown schedules for dev/test
3. **Right-size VMs**: Monitor and adjust VM sizes
4. **Use Azure Spot VMs**: For non-critical workloads
5. **Enable storage lifecycle policies**: Move old data to cool/archive tiers

## 📚 Additional Resources

- [Azure Load Balancer Documentation](https://docs.microsoft.com/azure/load-balancer/)
- [VNet Peering Guide](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview)
- [Private Endpoint Documentation](https://docs.microsoft.com/azure/private-link/private-endpoint-overview)
- [Azure Backup Documentation](https://docs.microsoft.com/azure/backup/)
- [Log Analytics Documentation](https://docs.microsoft.com/azure/azure-monitor/logs/log-analytics-overview)
- [Azure Key Vault Best Practices](https://docs.microsoft.com/azure/key-vault/general/best-practices)

## 🐛 Troubleshooting

### Issue: VMs not accessible behind Load Balancer

**Solution:**
```bash
# Check backend pool health
az network lb show \
  --resource-group rg-enterprise-infrastructure \
  --name LB \
  --query "backendAddressPools[0].backendIPConfigurations"

# Verify NSG rules allow traffic
az network nsg rule list \
  --resource-group rg-enterprise-infrastructure \
  --nsg-name NSG \
  --output table
```

### Issue: Private Endpoint DNS not resolving

**Solution:**
```bash
# Verify private DNS zone link
az network private-dns link vnet list \
  --resource-group rg-enterprise-infrastructure \
  --zone-name privatelink.blob.core.windows.net

# Check DNS resolution from VM
nslookup stoaccounttotest.blob.core.windows.net
```

### Issue: VNet peering not working

**Solution:**
```bash
# Check peering state
az network vnet peering show \
  --resource-group rg-enterprise-infrastructure \
  --vnet-name vnet-1 \
  --name peering-link

# Verify address space doesn't overlap
az network vnet list \
  --resource-group rg-enterprise-infrastructure \
  --query "[].{Name:name, AddressSpace:addressSpace.addressPrefixes}"
```

## 👤 Author

**Ouail Mokhtar Khelas**


## 🌟 Acknowledgments

- Azure Architecture Center for design patterns
- Microsoft Learn for best practices documentation
- Azure Community for support and feedback

---

**Built with 💙 for Cloud Excellence**
