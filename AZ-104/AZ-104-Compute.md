# AZ-104 — Deploy and Manage Azure Compute Resources
## Domain 3 Deep-Dive Study Guide (20–25% of Exam)

---

## Table of Contents
1. [Virtual Machines — Create & Configure](#1-virtual-machines--create--configure)
2. [VM Sizes & Series](#2-vm-sizes--series)
3. [VM Disks — Types & Management](#3-vm-disks--types--management)
4. [VM Availability — Sets, Zones & VMSS](#4-vm-availability--sets-zones--vmss)
5. [VM Images — Marketplace, Custom & Shared](#5-vm-images--marketplace-custom--shared)
6. [VM Extensions & Cloud-Init](#6-vm-extensions--cloud-init)
7. [VM Backup & Restore](#7-vm-backup--restore)
8. [Azure App Service](#8-azure-app-service)
9. [App Service Deployment & Slots](#9-app-service-deployment--slots)
10. [App Service Scaling & Configuration](#10-app-service-scaling--configuration)
11. [Azure Container Instances (ACI)](#11-azure-container-instances-aci)
12. [Azure Container Registry (ACR)](#12-azure-container-registry-acr)
13. [Azure Kubernetes Service (AKS) — Basics](#13-azure-kubernetes-service-aks--basics)
14. [ARM Templates & Bicep](#14-arm-templates--bicep)
15. [Azure Automation & Cloud Shell](#15-azure-automation--cloud-shell)
16. [Exam Tips — Domain 3 Master List](#16-exam-tips--domain-3-master-list)

---

## 1. Virtual Machines — Create & Configure

### VM Creation — What You Need to Decide

| Decision | Options | Notes |
|----------|---------|-------|
| **Subscription** | Which subscription to bill | — |
| **Resource Group** | Logical container | Can move later |
| **VM Name** | Up to 64 chars (Windows: 15 max for hostname) | Can't rename after creation |
| **Region** | Where VM is hosted | Impacts available sizes & pricing |
| **Availability** | None, Availability Set, Availability Zone | Choose at creation only |
| **Image** | Windows / Linux from Marketplace or custom | OS disk created from this |
| **Size (SKU)** | vCPUs, RAM, temp storage, max disks | Can resize later (may require deallocation) |
| **Authentication** | Password or SSH key (Linux) | SSH recommended for Linux |
| **Inbound ports** | Allow HTTP, HTTPS, SSH, RDP | Via NSG rules |
| **Disks** | OS + Data disks | Premium SSD for production |
| **Networking** | VNet, Subnet, Public IP, NSG | Created automatically or use existing |

### Creating VMs via CLI

```bash
# Create Linux VM with SSH
az vm create \
  --resource-group MyRG \
  --name MyLinuxVM \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-sku Standard

# Create Windows VM
az vm create \
  --resource-group MyRG \
  --name MyWinVM \
  --image Win2022Datacenter \
  --size Standard_D2s_v3 \
  --admin-username azureadmin \
  --admin-password "P@ssw0rd123!"
```

### Creating VMs via PowerShell

```powershell
# Create VM
New-AzVM -ResourceGroupName "MyRG" `
  -Name "MyVM" `
  -Location "eastus" `
  -Image "Ubuntu2204" `
  -Size "Standard_B2s" `
  -Credential (Get-Credential)
```

### VM States & Billing

| State | Compute Billing | Storage Billing | How |
|-------|----------------|----------------|-----|
| **Running** | ✅ Yes | ✅ Yes | Started via Azure |
| **Stopped (from OS)** | ✅ **Still billed!** | ✅ Yes | Shutdown from inside OS |
| **Stopped (Deallocated)** | ❌ No | ✅ Yes (disks) | `az vm deallocate` or Portal Stop |

> **Critical distinction:** "Stop" from inside the OS does NOT deallocate. You must use Azure Portal/CLI/PowerShell to deallocate and stop billing.

### VM Management Commands

```bash
# Start / Stop / Restart / Deallocate
az vm start --resource-group MyRG --name MyVM
az vm stop --resource-group MyRG --name MyVM          # OS stop (still billed!)
az vm deallocate --resource-group MyRG --name MyVM    # Full stop (no compute billing)
az vm restart --resource-group MyRG --name MyVM

# Resize
az vm resize --resource-group MyRG --name MyVM --size Standard_D4s_v3

# List available sizes for resize
az vm list-vm-resize-options --resource-group MyRG --name MyVM --output table

# Delete VM
az vm delete --resource-group MyRG --name MyVM --yes
```

---

## 2. VM Sizes & Series

### VM Families

| Series | Type | Optimized For | Use Cases |
|--------|------|--------------|-----------|
| **A** | Entry-level | Basic compute | Dev/test, build servers |
| **B** | Burstable | Variable workloads | Dev/test, light web servers |
| **D/Ds/Dv5** | General purpose | Balanced CPU/memory | Most production workloads |
| **E/Es/Ev5** | Memory-optimized | High memory-to-CPU ratio | Databases, caching, analytics |
| **F/Fs/Fv2** | Compute-optimized | High CPU-to-memory ratio | Batch, gaming, modeling |
| **L/Ls** | Storage-optimized | High disk throughput & IOPS | Big data, SQL, data warehousing |
| **M** | Memory-intensive | Very high memory | SAP HANA, very large DBs |
| **N/NC/ND/NV** | GPU | Graphics/AI/ML | Machine learning, rendering, VDI |
| **H** | HPC | High-performance compute | Fluid dynamics, finite element |

### Size Naming Convention
```
Standard_D4s_v5
  │       │ │ │
  │       │ │ └── Version (generation)
  │       │ └──── s = Premium SSD support
  │       └────── 4 = vCPUs
  └────────────── D = Family (General Purpose)
```

### Common Suffixes
| Suffix | Meaning |
|--------|---------|
| s | Supports Premium SSD |
| d | Has local temp disk |
| i | Isolated (dedicated hardware) |
| l | Low memory (relative to series) |
| a | AMD processor |
| p | ARM processor |

### Resizing VMs
- **Online resize:** If new size is available in current hardware cluster → no deallocation needed
- **Offline resize:** If new size requires different hardware → must deallocate first
- Check available sizes: `az vm list-vm-resize-options`
- Dynamic Public IP will change on deallocation (use Static IP if needed)

---

## 3. VM Disks — Types & Management

### Disk Types Comparison

| Disk Type | IOPS (max) | Throughput | Latency | Use Case | SLA |
|-----------|-----------|-----------|---------|----------|-----|
| **Standard HDD** | 2,000 | 500 MB/s | High | Dev/test, backups | ❌ No VM SLA |
| **Standard SSD** | 6,000 | 750 MB/s | Medium | Light production | ❌ No VM SLA |
| **Premium SSD (P-series)** | 20,000 | 900 MB/s | Low | Production workloads | ✅ 99.9% |
| **Premium SSD v2** | 80,000 | 1,200 MB/s | Sub-ms | High-perf workloads | ✅ 99.9% |
| **Ultra Disk** | 160,000 | 4,000 MB/s | Sub-ms | Top-tier databases | ✅ 99.9% |

> **Single VM SLA 99.9%** requires ALL disks to be Premium SSD or Ultra

### Managed vs Unmanaged Disks

| | Managed Disks | Unmanaged Disks |
|-|--------------|-----------------|
| Storage management | Azure handles | You manage storage account |
| Availability Sets | Auto-aligned to fault domains | Manual placement |
| Snapshots/Images | Built-in support | Manual VHD management |
| RBAC | Per-disk RBAC | Storage account level |
| Scale | 50,000 per subscription per region | Limited by storage account |
| **Recommendation** | ✅ **Always use managed** | ❌ Legacy only |

### Disk Operations

```bash
# Add data disk to existing VM
az vm disk attach \
  --resource-group MyRG \
  --vm-name MyVM \
  --name MyDataDisk \
  --size-gb 128 \
  --sku Premium_LRS \
  --new

# Detach disk
az vm disk detach --resource-group MyRG --vm-name MyVM --name MyDataDisk

# Resize disk (VM must be deallocated or disk unattached)
az disk update --resource-group MyRG --name MyDataDisk --size-gb 256

# Create snapshot
az snapshot create --resource-group MyRG --name MySnapshot --source MyDataDisk
```

### Disk Encryption Options

| Method | Description | Key Storage |
|--------|-------------|-------------|
| **Azure Disk Encryption (ADE)** | BitLocker (Windows) / DM-Crypt (Linux) | Key Vault |
| **Server-Side Encryption (SSE)** | Platform encryption at rest (default) | Platform keys or CMK |
| **Encryption at host** | Data encrypted before leaving VM host | Platform-managed |
| **Confidential disk encryption** | Integrity-protected + encrypted | Platform (ties to VM TPM) |

> **SSE is always enabled** (cannot disable). ADE adds an additional layer.

### Temporary (Ephemeral) Disk
- **Included with most VM sizes** (local SSD on host)
- **Data lost on deallocation/redeployment** — not persisted
- Used for: page file, swap, temp data
- Drive letter: D: (Windows), /dev/sdb (Linux)
- **Not backed up** by Azure Backup

---

## 4. VM Availability — Sets, Zones & VMSS

### Availability Sets

| Concept | Description | Limit |
|---------|-------------|-------|
| **Fault Domain (FD)** | Separate physical rack (power + switch) | 2-3 per set |
| **Update Domain (UD)** | Separate maintenance group (reboot sequencing) | 5 (default), up to 20 |
| **Purpose** | Protect against rack failure + planned maintenance | — |
| **SLA** | 99.95% (with 2+ VMs) | — |
| **Requirement** | VMs must be added at creation time | Same region |

```bash
# Create availability set
az vm availability-set create \
  --resource-group MyRG \
  --name MyAvSet \
  --platform-fault-domain-count 2 \
  --platform-update-domain-count 5

# Create VM in availability set
az vm create --resource-group MyRG --name VM1 --availability-set MyAvSet ...
```

### Availability Zones

| Concept | Description |
|---------|-------------|
| **What** | Physically separate datacenters within a region |
| **Protection** | Datacenter failure (power, cooling, networking) |
| **SLA** | 99.99% (with 2+ VMs across zones) |
| **Zones** | Typically 3 zones per region (zone 1, 2, 3) |
| **Networking** | VMs in different zones can communicate via VNet |
| **Not all regions** | Check availability per region |

```bash
# Create VM in specific zone
az vm create --resource-group MyRG --name VM1 --zone 1 ...
az vm create --resource-group MyRG --name VM2 --zone 2 ...
```

### Availability Sets vs Zones — Comparison

| | Availability Set | Availability Zone |
|-|-----------------|-------------------|
| Protection | Rack/power failure | Datacenter failure |
| SLA | 99.95% | 99.99% |
| Placement | Same datacenter, different racks | Different datacenters |
| Use together | ❌ Either one or the other | ❌ |
| Requirement | 2+ VMs | 2+ VMs in 2+ zones |
| Cost | Same as regular VMs | Same (cross-zone bandwidth charged) |

### Virtual Machine Scale Sets (VMSS)

| Feature | Description |
|---------|-------------|
| **What** | Group of identical, auto-managed VMs |
| **Scaling** | Scale out/in based on rules (metric-based or scheduled) |
| **Load Balancing** | Works with Azure LB or Application Gateway |
| **Max VMs** | 1,000 (Marketplace image) or 600 (custom image) |
| **Orchestration** | Uniform (identical) or Flexible (mixed configs) |
| **Upgrade Policy** | Automatic, Rolling, or Manual |

### VMSS Autoscale

```
Scale Rule:
  Metric source → Metric name → Time aggregation → Operator → Threshold
  → Action (increase/decrease) → Cool-down period
```

| Parameter | Description |
|-----------|-------------|
| **Metric** | CPU %, Memory %, Network In/Out, Disk Queue |
| **Time grain** | How often metric is collected (1 min, 5 min) |
| **Time aggregation** | Average, Min, Max, Total, Count |
| **Operator** | >, <, >=, <=, == |
| **Threshold** | Value that triggers action |
| **Action** | Increase/decrease count by X, or set to X |
| **Cool-down** | Wait time before another scale action (default 5 min) |

### VMSS Scaling Configuration

```bash
# Create VMSS with autoscale
az vmss create \
  --resource-group MyRG \
  --name MyScaleSet \
  --image Ubuntu2204 \
  --instance-count 2 \
  --vm-sku Standard_B2s \
  --upgrade-policy-mode automatic

# Configure autoscale rule
az monitor autoscale create \
  --resource-group MyRG \
  --resource MyScaleSet \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --min-count 2 --max-count 10 --count 2

az monitor autoscale rule create \
  --resource-group MyRG \
  --autoscale-name <autoscale-name> \
  --condition "Percentage CPU > 75 avg 5m" \
  --scale out 2
```

### VMSS Upgrade Policies

| Policy | Behavior | Use Case |
|--------|----------|----------|
| **Automatic** | All instances updated immediately | Dev/test |
| **Rolling** | Updated in batches with pause between | Production (controlled) |
| **Manual** | You trigger updates per instance | Full control |

---

## 5. VM Images — Marketplace, Custom & Shared

### Image Types

| Type | Description | Use Case |
|------|-------------|----------|
| **Marketplace** | Pre-built by Microsoft/partners | Quick deployment, standard OS |
| **Custom Image** | Created from your generalized/specialized VM | Standardized configurations |
| **Shared Image Gallery** | Versioned, replicated image repository | Multi-region, team sharing |

### Generalized vs Specialized Images

| | Generalized | Specialized |
|-|-------------|-------------|
| Preparation | sysprep (Windows) / waagent -deprovision (Linux) | None (as-is) |
| First boot | OOBE (new user/hostname) | Exact clone (same hostname, users) |
| Use case | Template for many VMs | Clone of specific VM |
| Create multiple VMs | ✅ Yes | ⚠️ Not recommended (conflicts) |
| Admin credentials | Required at deploy time | Inherited from source |

### Creating Custom Image

```bash
# 1. Generalize the VM (Linux)
sudo waagent -deprovision+user -force

# 2. Deallocate and mark as generalized
az vm deallocate --resource-group MyRG --name MyVM
az vm generalize --resource-group MyRG --name MyVM

# 3. Create image
az image create --resource-group MyRG --name MyImage --source MyVM
```

### Azure Compute Gallery (Shared Image Gallery)

| Feature | Description |
|---------|-------------|
| **Versioning** | Store multiple versions (1.0.0, 1.1.0, etc.) |
| **Replication** | Replicate to multiple regions |
| **Sharing** | Share across subscriptions/tenants via RBAC |
| **Image definition** | Metadata (OS type, generation, description) |
| **Image version** | Actual image data (VHD) |

---

## 6. VM Extensions & Cloud-Init

### Common VM Extensions

| Extension | Purpose | OS |
|-----------|---------|-----|
| **Custom Script Extension** | Run scripts post-deployment | Windows + Linux |
| **DSC (Desired State Configuration)** | Apply configuration state | Windows |
| **Azure Monitor Agent** | Collect metrics & logs | Both |
| **Network Watcher Agent** | Enable packet capture | Both |
| **Azure Disk Encryption** | Enable ADE (BitLocker/DM-Crypt) | Both |
| **BGInfo** | Display system info on desktop | Windows |
| **Chef/Puppet/Ansible** | Configuration management | Both |

### Custom Script Extension

```bash
# Apply Custom Script Extension (Linux)
az vm extension set \
  --resource-group MyRG \
  --vm-name MyVM \
  --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --settings '{"commandToExecute":"apt-get update && apt-get install -y nginx"}'

# Windows Custom Script Extension
az vm extension set \
  --resource-group MyRG \
  --vm-name MyWinVM \
  --name CustomScriptExtension \
  --publisher Microsoft.Compute \
  --settings '{"commandToExecute":"powershell Install-WindowsFeature -name Web-Server"}'
```

### Cloud-Init (Linux)
- Runs at first boot to configure a Linux VM
- Passed via `--custom-data` parameter at VM creation
- Supports: package install, file creation, user creation, script execution

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
  - docker.io
runcmd:
  - systemctl start nginx
  - systemctl enable nginx
```

```bash
az vm create --resource-group MyRG --name MyVM --image Ubuntu2204 \
  --custom-data cloud-init.yaml ...
```

---

## 7. VM Backup & Restore

### Azure Backup for VMs

| Component | Role |
|-----------|------|
| **Recovery Services Vault** | Container for backup data (must be in same region as VM) |
| **Backup Policy** | Defines frequency and retention |
| **Backup extension** | Installed on VM automatically during first backup |
| **Snapshot** | First stored locally (instant restore), then transferred to vault |

### Backup Process
1. Azure installs backup extension on VM (if not already present)
2. Takes application-consistent (Windows) or file-consistent (Linux) snapshot
3. Snapshot stored locally for instant restore (1-5 days configurable)
4. Data transferred to Recovery Services Vault

### Backup Policy Options

| Setting | Options |
|---------|---------|
| **Frequency** | Daily, Hourly (Enhanced policy) |
| **Retention — Daily** | 7-9999 days |
| **Retention — Weekly** | Up to 5163 weeks |
| **Retention — Monthly** | Up to 1188 months |
| **Retention — Yearly** | Up to 99 years |
| **Instant restore** | 1-5 days (snapshot retained locally) |

### Restore Options

| Option | Description | Use Case |
|--------|-------------|----------|
| **Create new VM** | Full VM from recovery point | Replace failed VM |
| **Restore disk** | Restore managed disks only | Custom recovery, attach to existing VM |
| **Replace existing** | Replace disks of running VM | Update in-place |
| **Cross-region restore** | Restore to paired region (GRS vaults) | DR scenario |
| **File recovery** | Mount recovery point, browse files | Recover specific files only |

### Backup CLI Commands

```bash
# Enable backup for a VM
az backup protection enable-for-vm \
  --resource-group MyRG \
  --vault-name MyVault \
  --vm MyVM \
  --policy-name DefaultPolicy

# Trigger on-demand backup
az backup protection backup-now \
  --resource-group MyRG \
  --vault-name MyVault \
  --container-name MyVM \
  --item-name MyVM

# List recovery points
az backup recoverypoint list \
  --resource-group MyRG \
  --vault-name MyVault \
  --container-name MyVM \
  --item-name MyVM \
  --output table

# Restore VM
az backup restore restore-disks \
  --resource-group MyRG \
  --vault-name MyVault \
  --container-name MyVM \
  --item-name MyVM \
  --rp-name <recovery-point-name> \
  --storage-account mystorageacct
```

---

## 8. Azure App Service

### What It Is
Fully managed PaaS platform for hosting web apps, REST APIs, and mobile backends.

### App Service Plan

| Component | Description |
|-----------|-------------|
| **App Service Plan** | Defines the compute resources (VM size, region, scaling) |
| **Apps in a plan** | Multiple apps can share one plan (share resources) |
| **Pricing tier** | Determines features, scale limits, and cost |

### Pricing Tiers Comparison

| Tier | Compute | Custom Domain | SSL | Slots | Auto-Scale | VNet | Monthly Cost |
|------|---------|---------------|-----|-------|-----------|------|-------------|
| **Free (F1)** | Shared | ❌ | ❌ | 0 | ❌ | ❌ | $0 |
| **Shared (D1)** | Shared | ✅ | ❌ | 0 | ❌ | ❌ | ~$10 |
| **Basic (B1-B3)** | Dedicated | ✅ | ✅ | 0 | ❌ (manual) | ❌ | ~$55+ |
| **Standard (S1-S3)** | Dedicated | ✅ | ✅ | 5 | ✅ | ❌ | ~$73+ |
| **Premium v3 (P1v3-P3v3)** | Dedicated | ✅ | ✅ | 20 | ✅ | ✅ | ~$138+ |
| **Isolated v2 (I1v2-I3v2)** | Dedicated (ASE) | ✅ | ✅ | 20 | ✅ | ✅ (full) | ~$370+ |

### App Service Plan Key Facts
- **Free/Shared:** No SLA, shared infrastructure, limited CPU time per day
- **Basic+:** Dedicated VMs, always running
- **Standard+:** Deployment slots, auto-scale
- **Premium+:** VNet integration, more slots, more power
- **Isolated:** App Service Environment (ASE), full network isolation

### Creating App Service

```bash
# Create App Service Plan
az appservice plan create \
  --name MyPlan \
  --resource-group MyRG \
  --sku S1 \
  --location eastus

# Create Web App
az webapp create \
  --name mywebapp2026 \
  --resource-group MyRG \
  --plan MyPlan \
  --runtime "DOTNET|8.0"

# List supported runtimes
az webapp list-runtimes --output table
```

### Supported Runtimes
| Runtime | Versions |
|---------|----------|
| .NET | 6.0, 7.0, 8.0 |
| Java | 8, 11, 17, 21 |
| Node.js | 16, 18, 20 |
| Python | 3.9, 3.10, 3.11, 3.12 |
| PHP | 8.0, 8.1, 8.2 |
| Ruby | 2.7, 3.0 |

---

## 9. App Service Deployment & Slots

### Deployment Methods

| Method | Description | Best For |
|--------|-------------|----------|
| **ZIP Deploy** | Upload ZIP package | Simple deployments |
| **Git (Local)** | Push from local Git to App Service | Developers |
| **GitHub Actions** | CI/CD from GitHub | Team workflows |
| **Azure DevOps Pipelines** | CI/CD from Azure DevOps | Enterprise |
| **FTP/FTPS** | File transfer | Legacy, quick fixes |
| **Azure CLI** | `az webapp deploy` | Scripted deployments |
| **Visual Studio / VS Code** | IDE-integrated publish | Developer convenience |

### Deployment Slots

| Feature | Description |
|---------|-------------|
| **What** | Separate app instances with own URL (e.g., myapp-staging.azurewebsites.net) |
| **Purpose** | Test in production-like environment before going live |
| **Swap** | Exchange traffic between slots (zero-downtime deployment) |
| **Tier requirement** | Standard+ (5 slots) or Premium+ (20 slots) |
| **Independent config** | Each slot can have different settings |

### Slot Settings Behavior

| Setting Type | Swaps with App | Stays with Slot |
|-------------|---------------|-----------------|
| App settings (default) | ✅ | — |
| App settings (slot setting ✓) | — | ✅ |
| Connection strings (default) | ✅ | — |
| Connection strings (slot setting ✓) | — | ✅ |
| Custom domains | — | ✅ |
| SSL certificates | — | ✅ |
| Scale settings | — | ✅ |
| General settings (Always On, etc.) | ✅ | — |

> **"Deployment slot setting"** checkbox = the setting is "sticky" to that slot (doesn't move during swap)

### Slot Swap Process
1. Apply **target slot's** settings to source slot (warm-up)
2. Wait for all instances to restart with new settings
3. If warm-up succeeds, **swap the VIP routing** (traffic switches instantly)
4. Source slot now has the old production content

### Traffic Routing (A/B Testing)
```bash
# Route 20% of traffic to staging slot
az webapp traffic-routing set \
  --resource-group MyRG \
  --name mywebapp \
  --distribution staging=20
```

### Auto-Swap
- Automatically swap staging → production after deployment
- Enable on the source slot (staging)
- Good for continuous deployment pipelines

---

## 10. App Service Scaling & Configuration

### Scaling Options

| Type | Description | Tier Required |
|------|-------------|--------------|
| **Scale Up (Vertical)** | Change to larger/smaller plan tier | Any (change plan) |
| **Scale Out (Horizontal)** | Add more instances of same size | Standard+ |
| **Autoscale** | Automatic scale out/in based on rules | Standard+ |

### Autoscale Rules
- Based on: CPU %, Memory %, HTTP Queue Length, custom metric
- Conditions: Greater than, Less than threshold
- Actions: Increase/decrease count by N, or set to specific count
- Cool-down: Minimum time between scale actions

### App Settings & Connection Strings

| Feature | Description |
|---------|-------------|
| **App settings** | Environment variables available to app (override appsettings.json) |
| **Connection strings** | Database/service connection strings (typed: SQL, MySQL, Custom) |
| **General settings** | Stack, platform (32/64-bit), Always On, ARR affinity |
| **Path mappings** | Virtual directories, handler mappings |

```bash
# Set app setting
az webapp config appsettings set \
  --resource-group MyRG \
  --name mywebapp \
  --settings WEBSITE_TIME_ZONE="Eastern Standard Time"

# Set connection string
az webapp config connection-string set \
  --resource-group MyRG \
  --name mywebapp \
  --connection-string-type SQLAzure \
  --settings MyDB="Server=tcp:myserver.database.windows.net;Database=mydb;..."
```

### Key Configuration Settings

| Setting | Description | Default |
|---------|-------------|---------|
| **Always On** | Keeps app loaded (no cold start) | Off (Free/Shared: unavailable) |
| **ARR Affinity** | Sticky sessions (route same client to same instance) | On |
| **HTTP version** | HTTP/1.1 or HTTP/2 | 1.1 |
| **Web sockets** | Enable WebSocket connections | Off |
| **Platform** | 32-bit or 64-bit | 32-bit |
| **FTP state** | FTPS Only, FTP (insecure), or Disabled | FTPS Only |
| **Minimum TLS version** | 1.0, 1.1, or 1.2 | 1.2 |

### Custom Domains & SSL

```bash
# Add custom domain
az webapp config hostname add \
  --resource-group MyRG \
  --webapp-name mywebapp \
  --hostname www.contoso.com

# Create managed SSL certificate (free)
az webapp config ssl create \
  --resource-group MyRG \
  --name mywebapp \
  --hostname www.contoso.com

# Bind certificate
az webapp config ssl bind \
  --resource-group MyRG \
  --name mywebapp \
  --certificate-thumbprint <thumbprint> \
  --ssl-type SNI
```

---

## 11. Azure Container Instances (ACI)

### What It Is
Serverless container platform — run containers without managing VMs or orchestrators.

### Key Features

| Feature | Detail |
|---------|--------|
| **Start time** | Seconds (vs minutes for VMs) |
| **Billing** | Per-second (vCPU + memory used) |
| **Orchestration** | None — single container or container group |
| **Networking** | Public IP or VNet (private) |
| **Storage** | Azure Files mount, emptyDir, secret volume |
| **OS** | Linux and Windows containers |
| **GPU** | Supported (Linux only) |
| **Max resources** | 4 vCPU, 16 GB per container (varies by region) |

### Container Groups
- Multiple containers sharing same host, lifecycle, network, and storage
- Similar to Kubernetes "pod"
- Containers in a group share localhost network
- Deployed via YAML or ARM template (not CLI for multi-container)

### Creating ACI

```bash
# Simple container from Docker Hub
az container create \
  --resource-group MyRG \
  --name mycontainer \
  --image nginx:latest \
  --ports 80 \
  --dns-name-label mycontainer2026 \
  --cpu 1 \
  --memory 1.5

# From Azure Container Registry
az container create \
  --resource-group MyRG \
  --name mycontainer \
  --image myacr.azurecr.io/myapp:v1 \
  --registry-login-server myacr.azurecr.io \
  --registry-username <username> \
  --registry-password <password> \
  --ports 80

# View logs
az container logs --resource-group MyRG --name mycontainer

# View container details
az container show --resource-group MyRG --name mycontainer --output table
```

### Restart Policies

| Policy | Behavior | Use Case |
|--------|----------|----------|
| **Always** (default) | Restart on exit (any code) | Long-running services |
| **OnFailure** | Restart only on non-zero exit | Batch jobs with retry |
| **Never** | Don't restart | One-time tasks |

### Environment Variables & Secrets

```bash
# Set environment variables
az container create ... \
  --environment-variables KEY1=value1 KEY2=value2

# Secure environment variables (not shown in logs/portal)
az container create ... \
  --secure-environment-variables SECRET_KEY=supersecret
```

---

## 12. Azure Container Registry (ACR)

### What It Is
Private Docker container registry in Azure for storing and managing container images.

### ACR Tiers

| Tier | Storage | Webhooks | Geo-Replication | Private Endpoint | Content Trust |
|------|---------|----------|----------------|-----------------|---------------|
| **Basic** | 10 GiB | 2 | ❌ | ❌ | ❌ |
| **Standard** | 100 GiB | 10 | ❌ | ❌ | ❌ |
| **Premium** | 500 GiB | 500 | ✅ | ✅ | ✅ |

### Common ACR Operations

```bash
# Create ACR
az acr create --resource-group MyRG --name myacr2026 --sku Standard

# Login to ACR
az acr login --name myacr2026

# Build image in ACR (no local Docker needed)
az acr build --registry myacr2026 --image myapp:v1 .

# Push local image
docker tag myapp:v1 myacr2026.azurecr.io/myapp:v1
docker push myacr2026.azurecr.io/myapp:v1

# List images
az acr repository list --name myacr2026 --output table

# List tags
az acr repository show-tags --name myacr2026 --repository myapp --output table
```

### ACR Authentication Methods
| Method | Description | Use Case |
|--------|-------------|----------|
| **Admin account** | Username/password (disabled by default) | Dev/test, quick demos |
| **Service principal** | App registration with password | CI/CD pipelines |
| **Managed identity** | No credentials to manage | ACI, AKS, App Service |
| **az acr login** | Entra ID token-based | Developer machines |
| **Repository-scoped token** | Limited to specific repos | External partners |

### ACR Tasks
- **Quick task:** `az acr build` — build in cloud without local Docker
- **Triggered task:** Auto-build on source code commit or base image update
- **Multi-step task:** Complex workflows defined in YAML

---

## 13. Azure Kubernetes Service (AKS) — Basics

### What It Is
Managed Kubernetes cluster — Azure manages the control plane; you manage worker nodes.

### AKS Architecture

```
Control Plane (Azure-managed, free)
  ├── API Server
  ├── etcd
  ├── Scheduler
  └── Controller Manager

Worker Nodes (you pay for these)
  ├── Node Pool 1 (System) — runs system pods
  └── Node Pool 2 (User) — runs your workloads
```

### Key Concepts for AZ-104

| Concept | Description |
|---------|-------------|
| **Node pool** | Group of VMs with same configuration |
| **System node pool** | Runs core K8s services (required, min 1) |
| **User node pool** | Runs your application workloads |
| **kubectl** | CLI tool to interact with Kubernetes |
| **Pods** | Smallest deployable unit (1+ containers) |
| **Services** | Expose pods to network (ClusterIP, LoadBalancer, NodePort) |
| **Deployments** | Manage pod replicas and rolling updates |

### AKS CLI Commands

```bash
# Create AKS cluster
az aks create \
  --resource-group MyRG \
  --name MyAKS \
  --node-count 2 \
  --node-vm-size Standard_B2s \
  --generate-ssh-keys

# Get credentials (configure kubectl)
az aks get-credentials --resource-group MyRG --name MyAKS

# Scale node pool
az aks scale --resource-group MyRG --name MyAKS --node-count 5

# Upgrade cluster
az aks upgrade --resource-group MyRG --name MyAKS --kubernetes-version 1.28.0
```

### AKS Scaling Options

| Method | Description |
|--------|-------------|
| **Manual** | `az aks scale --node-count N` |
| **Cluster Autoscaler** | Auto add/remove nodes based on pod resource requests |
| **Horizontal Pod Autoscaler (HPA)** | Scale pods based on CPU/memory |
| **KEDA** | Event-driven autoscaling (queue length, etc.) |

---

## 14. ARM Templates & Bicep

### ARM Template Structure

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "metadata": { "description": "Name of the virtual machine" }
    }
  },
  "variables": {
    "nicName": "[concat(parameters('vmName'), '-nic')]"
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2023-03-01",
      "name": "[parameters('vmName')]",
      "location": "[resourceGroup().location]",
      "properties": { }
    }
  ],
  "outputs": {
    "vmId": {
      "type": "string",
      "value": "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]"
    }
  }
}
```

### Template Sections

| Section | Purpose | Required |
|---------|---------|----------|
| **$schema** | ARM template schema URL | ✅ |
| **contentVersion** | Your version tracking | ✅ |
| **parameters** | Input values at deployment | ❌ |
| **variables** | Computed values used in template | ❌ |
| **resources** | Resources to deploy | ✅ |
| **outputs** | Return values after deployment | ❌ |
| **functions** | Custom template functions | ❌ |

### Deployment Modes

| Mode | Behavior | Risk |
|------|----------|------|
| **Incremental** (default) | Add/update resources; leave others alone | Low |
| **Complete** | Add/update resources; **DELETE** resources not in template | ⚠️ High |

> Always use **what-if** before Complete mode: `az deployment group what-if`

### Deploying Templates

```bash
# Deploy ARM template
az deployment group create \
  --resource-group MyRG \
  --template-file template.json \
  --parameters @parameters.json

# What-if (preview changes)
az deployment group what-if \
  --resource-group MyRG \
  --template-file template.json

# Deploy Bicep
az deployment group create \
  --resource-group MyRG \
  --template-file main.bicep
```

### Bicep Overview
- Domain-specific language (DSL) for ARM templates
- Compiles to ARM JSON (1:1 mapping)
- Cleaner syntax, better IntelliSense in VS Code
- Modules for reusability

**Bicep Example:**
```bicep
param location string = resourceGroup().location
param vmName string

resource vm 'Microsoft.Compute/virtualMachines@2023-03-01' = {
  name: vmName
  location: location
  properties: {
    // ...
  }
}

output vmId string = vm.id
```

---

## 15. Azure Automation & Cloud Shell

### Azure Cloud Shell
| Feature | Detail |
|---------|--------|
| **What** | Browser-based shell (Bash or PowerShell) |
| **Access** | Azure Portal, shell.azure.com, VS Code, mobile app |
| **Storage** | Requires Azure Files share (persists home directory) |
| **Pre-installed** | Azure CLI, PowerShell, Terraform, kubectl, AzCopy, etc. |
| **Timeout** | Idle timeout: 20 minutes |
| **Cost** | Storage share cost only (compute is free) |

### Azure Automation Account
| Feature | Description |
|---------|-------------|
| **Runbooks** | PowerShell or Python scripts that automate tasks |
| **Schedules** | Run runbooks at specified times/intervals |
| **Webhooks** | Trigger runbooks via HTTP POST |
| **Variables** | Store shared values across runbooks |
| **Credentials** | Securely store username/password |
| **Update Management** | Manage OS updates for VMs |
| **State Configuration (DSC)** | Ensure VMs maintain desired state |

### Auto-Shutdown for VMs
```bash
# Configure auto-shutdown
az vm auto-shutdown \
  --resource-group MyRG \
  --name MyVM \
  --time 1900 \
  --timezone "Eastern Standard Time"
```

---

## 16. Exam Tips — Domain 3 Master List

### Virtual Machines
- **Stop ≠ Deallocate** — OS shutdown still bills for compute; deallocate does not
- **Premium SSD on ALL disks** required for 99.9% single-VM SLA
- **Resize may require deallocation** if target size is in a different hardware cluster
- **Temp disk is ephemeral** — data lost on deallocation; don't store important data
- **Availability Set OR Zone** — cannot use both simultaneously
- **VM added to Availability Set at creation only** — cannot move existing VM into one
- **Custom image: Generalized** = runs sysprep, creates new identity; **Specialized** = exact clone

### VMSS
- **Uniform mode** = all VMs identical (traditional); **Flexible** = mixed configs
- **Max 1,000 instances** (Marketplace), 600 (custom image)
- **Upgrade policy** — Automatic (all at once), Rolling (batched), Manual
- **Cool-down period** in autoscale prevents thrashing (default 5 min)

### App Service
- **Free/Shared** = no SLA, no deployment slots, no Always On
- **Deployment slot settings marked "slot setting"** are sticky (don't move with swap)
- **Auto-swap** works from source (staging) only
- **Scale out** requires Standard tier or above
- **Always On** must be enabled for background jobs (WebJobs) to run continuously
- **VNet integration** requires Premium v3 or Isolated tier

### Containers
- **ACI restart policy "Never"** = one-time batch job (no restart on completion)
- **ACI container groups** share network (localhost) and lifecycle
- **ACR Premium** required for geo-replication and private endpoints
- **ACR admin account disabled by default** — use service principal or managed identity
- **AKS system node pool** cannot be deleted (at least one required)

### ARM Templates
- **Complete mode DELETES unlisted resources** — always use what-if first
- **Parameters vs Variables:** Parameters = user input; Variables = computed at deploy time
- **Template deployment is idempotent** — running same template twice produces same result
- **Bicep compiles to ARM JSON** — same deployment engine, cleaner syntax

---

*Next: [Domain 4 — Implement and Manage Virtual Networking](AZ-104-Networking.md)*
