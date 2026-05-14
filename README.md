# Azure Enterprise Infrastructure with Load Balancing & High Availability

## 🚀 Overview

This project demonstrates deploying enterprise-grade Azure infrastructure with load balancing, high availability, and security hardening. The entire infrastructure was designed in Azure Portal.

## ✅ What I Built

- 1 Load Balancer (public IP for traffic distribution)
- 2 Load-Balanced Virtual Machines (vm1-LB, vm2-LB)
- 2 Virtual Networks with Peering (vnet-1, vnet-2)
- 2 Storage Accounts (data storage & replication)
- 1 Recovery Services Vault (backup & disaster recovery)
- 1 Key Vault (secrets management)
- 1 Log Analytics Workspace (monitoring & logging)
- Private Endpoints (secure access to storage & backup)
- Network Security Groups (traffic filtering)

## 📍 Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Azure Load Balancer                   │
│                    (Public IP)                           │
└────────────────────┬────────────────────────────────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
    ┌───▼────┐              ┌────▼────┐
    │ VM1-LB │              │ VM2-LB  │
    │(Linux) │              │ (Linux) │
    └────┬───┘              └────┬────┘
         │                       │
    ┌────▼──────────────────────▼────┐
    │       VNet-1 (10.0.0.0/16)     │
    │  ┌──────────┐  ┌──────────┐   │
    │  │ Subnet1  │  │ Subnet2  │   │
    │  └──────────┘  └──────────┘   │
    └────┬────────────────────────┬──┘
         │    VNet Peering        │
    ┌────▼──────────────────────▼──┐
    │      VNet-2 (10.2.0.0/16)    │
    └───────────────────────────────┘

    Storage: Private Endpoints
    Backup: Recovery Services Vault
    Secrets: Key Vault
    Monitoring: Log Analytics
```

## 📍 Technologies Used

- **Infrastructure**: Azure Portal + ARM Template JSON
- **Networking**: Virtual Networks, Load Balancer, VNet Peering
- **Compute**: Linux Virtual Machines (Ubuntu)
- **Storage**: Storage Accounts with replication
- **Backup**: Recovery Services Vault
- **Security**: Private Endpoints, Key Vault, NSGs
- **Monitoring**: Log Analytics Workspace
- **Tools**: Azure Portal, Azure CLI, ARM Template

## 🎯 How I Built This

### 📍 Phase 1: Design in Azure Portal

Created and configured all resources in Azure Portal:
```
1. Created Virtual Networks (vnet-1, vnet-2)
2. Created Subnets (subnet-1.1, subnet-1.2)
3. Deployed Load Balancer with public IP
4. Created Virtual Machines (vm1-LB, vm2-LB)
5. Configured VNet Peering (bidirectional)
6. Created Storage Accounts
7. Deployed Recovery Services Vault
8. Created Key Vault
9. Set up Log Analytics Workspace
10. Added Private Endpoints
11. Configured Network Security Groups
12. Set up monitoring & alerts
```

### 📍 Phase 2: Export & Customize

Exported the entire infrastructure from Azure Portal as ARM Template JSON:
```bash
1. Azure Portal → Resource Group
2. Click "Export template"
3. Downloaded template.json
4. Reviewed and customized parameters
```

## 📊 Infrastructure Components

| Component | Count | Details |
|-----------|-------|---------|
| **Load Balancer** | 1 | Distributes traffic to VMs |
| **Virtual Machines** | 2 | Ubuntu Linux (vm1-LB, vm2-LB) |
| **Virtual Networks** | 2 | vnet-1 (10.0.0.0/16), vnet-2 (10.2.0.0/16) |
| **Subnets** | 2 | subnet-1.1, subnet-1.2 |
| **Public IPs** | 1 | For Load Balancer |
| **Storage Accounts** | 2 | Primary + geo-replication |
| **Recovery Vault** | 1 | VM backup & disaster recovery |
| **Key Vault** | 1 | Secrets & SSH keys management |
| **Log Analytics** | 1 | Monitoring & alerting |
| **Private Endpoints** | 2 | Secure storage & backup access |
| **NSGs** | 1 | Network traffic filtering |
