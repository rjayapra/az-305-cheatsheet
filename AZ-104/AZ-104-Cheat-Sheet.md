# AZ-104 Exam Cheat Sheet
> Microsoft Azure Administrator | Score to pass: 700/1000 | Duration: 120 min

---

## 🧭 EXAM STRATEGY
- **Implement, not design** — "How do you configure X?" not "Which service should you use?"
- **Know CLI + PowerShell + Portal** — questions test all three approaches
- **Minimum administrative effort** — least steps wins when requirements are met
- **Least privilege** — assign minimum required RBAC role
- **Read the ENTIRE question** — subtle words like "only," "minimum," "without" change the answer
- Flag hard questions; return at the end

---

## 🏆 DOMAIN WEIGHTS

| Domain | Weight | Key Focus |
|--------|--------|-----------|
| Manage Azure Identities & Governance | **20–25%** | Entra ID, RBAC, Policy, Subscriptions |
| Implement & Manage Storage | **15–20%** | Accounts, Blob, Files, Security |
| Deploy & Manage Compute Resources | **20–25%** | VMs, VMSS, App Service, Containers |
| Implement & Manage Virtual Networking | **15–20%** | VNets, NSG, Peering, LB, DNS |
| Monitor & Maintain Azure Resources | **10–15%** | Monitor, Alerts, Backup, Network Watcher |

---

## 🔐 DOMAIN 1 — MANAGE AZURE IDENTITIES & GOVERNANCE

### Entra ID (Azure AD) Object Types
| Object | Description |
|--------|-------------|
| **User** | Cloud-only or synced from on-prem AD |
| **Group** | Security or Microsoft 365; Assigned or Dynamic membership |
| **Device** | Registered, Joined, or Hybrid Joined |
| **Service Principal** | Identity for apps/services |
| **Managed Identity** | System-assigned or User-assigned (no credential management) |

### Group Types
| | Security Group | Microsoft 365 Group |
|-|---------------|-------------------|
| Purpose | Access control (RBAC, NSG) | Collaboration (email, SharePoint) |
| Dynamic | ✅ Yes | ✅ Yes |
| Nested | ✅ Yes | ❌ No |
| Assign roles | ✅ Yes (if enabled) | ✅ Yes (if enabled) |

### Dynamic Group Membership Rules
```
user.department -eq "IT"
user.jobTitle -contains "Manager"
device.deviceOSType -eq "Windows"
```
> Dynamic groups require **Entra ID P1 or P2** license

### Administrative Units (AUs)
- Restrict scope of admin roles to specific users/groups/devices
- Delegate admin without giving tenant-wide access
- Example: Helpdesk admin for only the "Sales" AU

### RBAC Key Facts
| Concept | Detail |
|---------|--------|
| **Scope** | Management Group → Subscription → Resource Group → Resource |
| **Inheritance** | Permissions inherit downward (broader scope applies to children) |
| **Deny assignments** | Block access (from Blueprints); cannot create manually |
| **Custom roles** | Define Actions, NotActions, DataActions, AssignableScopes |

### Built-in RBAC Roles to Know
| Role | Can Do |
|------|--------|
| **Owner** | Everything + assign roles |
| **Contributor** | Everything EXCEPT assign roles |
| **Reader** | Read-only |
| **User Access Administrator** | Manage role assignments only |
| **Virtual Machine Contributor** | Manage VMs, not VNet or storage account they're attached to |
| **Storage Blob Data Reader/Contributor/Owner** | Data-plane access to blobs |
| **Network Contributor** | Manage networking resources |

### Custom Role Definition (JSON)
```json
{
  "Name": "VM Restart Operator",
  "Actions": [
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/read"
  ],
  "NotActions": [],
  "AssignableScopes": ["/subscriptions/<sub-id>"]
}
```

### Azure Policy
| Effect | Behavior |
|--------|----------|
| **Deny** | Block non-compliant resource creation/update |
| **Audit** | Log but don't block (for reporting) |
| **Append** | Add fields (e.g., force a tag) |
| **Modify** | Change properties on existing resources |
| **DeployIfNotExists** | Deploy a resource if missing (e.g., deploy diagnostics) |
| **AuditIfNotExists** | Audit if a related resource doesn't exist |
| **Disabled** | Turn off the policy |

### Policy vs Initiative
- **Policy** = single rule (e.g., "allowed VM sizes")
- **Initiative (Policy Set)** = collection of policies grouped together
- Assign initiatives to scope (MG, Sub, RG)

### Resource Locks
| Lock Type | Prevent |
|-----------|---------|
| **ReadOnly** | Any modification or deletion (can still read) |
| **CanNotDelete (Delete)** | Deletion only (can still modify) |

> Locks override RBAC — even Owner cannot delete a CanNotDelete-locked resource without removing the lock first

### Resource Tags
- **Not inherited** by default (use Policy to enforce tag inheritance)
- Max 50 tags per resource
- Used for cost management, automation, compliance
- Format: `Key: Value` (e.g., `Environment: Production`)

### Subscriptions & Management Groups
- Management Groups organize subscriptions (max 6 levels below root)
- Move subscription between MGs: requires write permissions on both
- Policy at MG level inherits to all child subscriptions
- 10,000 management groups per directory

---

## 💾 DOMAIN 2 — IMPLEMENT & MANAGE STORAGE

### Storage Account Types
| Type | Services | Performance |
|------|----------|-------------|
| **General-purpose v2 (GPv2)** | Blob, Files, Queue, Table | Standard or Premium |
| **Premium Block Blob** | Block blobs only | Premium (low latency) |
| **Premium File Shares** | Files only | Premium (NFS/SMB) |
| **Premium Page Blobs** | Page blobs only | Premium (VM disks) |

> **GPv2 is the default and recommended** — supports all features including access tiers

### Storage Redundancy
| Option | Copies | Protection | Min for HA |
|--------|--------|-----------|-----------|
| **LRS** | 3 in one datacenter | Hardware failure | ❌ |
| **ZRS** | 3 across 3 zones | Zone failure | ✅ |
| **GRS** | 6 (3 local + 3 in paired region) | Regional failure | ✅ |
| **GZRS** | 6 (3 zones + 3 in paired region) | Zone + regional | ✅ |
| **RA-GRS** | Same as GRS + read from secondary | Regional + read access | ✅ |
| **RA-GZRS** | Same as GZRS + read from secondary | Best protection | ✅ |

> **Key:** RA- prefix = Read Access to secondary. Without RA-, secondary is only available during failover.

### Blob Storage
| Blob Type | Use Case |
|-----------|---------|
| **Block Blob** | Files, images, videos, documents (default) |
| **Append Blob** | Log files, streaming data (append-only) |
| **Page Blob** | VHD files, random read/write (VM disks) |

### Access Tiers
| Tier | Access Latency | Min Retention | Use Case |
|------|---------------|---------------|----------|
| **Hot** | Milliseconds | None | Frequently accessed |
| **Cool** | Milliseconds | 30 days | Infrequently accessed |
| **Cold** | Milliseconds | 90 days | Rarely accessed |
| **Archive** | Hours (rehydrate) | 180 days | Compliance, backup |

> **Early deletion fee** applies if you delete/move before minimum retention
> Archive tier = offline — must rehydrate (Standard: up to 15 hours, High-priority: < 1 hour)

### Lifecycle Management Rules
- Automatically transition blobs between tiers or delete them
- Based on: last modified date, last accessed date, creation date
- Applies to block blobs and append blobs
- Example: Move to Cool after 30 days → Archive after 90 days → Delete after 365 days

### Storage Security
| Method | Description | Use Case |
|--------|-------------|----------|
| **Access Keys** | Full access to entire account | Admin tools (rotate regularly) |
| **SAS (Shared Access Signature)** | Scoped, time-limited token | Delegated access to specific resources |
| **Stored Access Policy** | Named policy on container/queue | Revocable SAS (up to 5 per container) |
| **Entra ID + RBAC** | Identity-based access | Best practice for production |
| **Anonymous access** | Public read (container/blob level) | Static websites, public files |

### SAS Types
| Type | Scope | Signed By |
|------|-------|-----------|
| **Account SAS** | Storage account level | Account key |
| **Service SAS** | Single service (blob, file, queue, table) | Account key |
| **User Delegation SAS** | Blob/container only | Entra ID credentials |

> **User Delegation SAS is most secure** — no account key needed, uses Entra ID, shorter lifetime

### Azure Files
- SMB (445) and NFS (2049) file shares in the cloud
- Mount on Windows, Linux, macOS
- **Identity-based access:** Entra Domain Services, on-prem AD DS, Entra Kerberos
- **Snapshots:** Point-in-time read-only copies of file shares
- **Soft delete:** Recover deleted shares within retention period

### Azure File Sync
- Syncs Azure Files to on-prem Windows Servers
- Components: Storage Sync Service → Sync Group → Cloud Endpoint + Server Endpoint(s)
- **Cloud tiering:** Frequently used files cached locally; cold files stored in Azure
- Reduces on-prem storage footprint

### AzCopy & Import/Export
| Tool | Use Case |
|------|----------|
| **AzCopy** | CLI tool for fast copy to/from Azure Storage |
| **Azure Storage Explorer** | GUI for managing blobs, files, queues, tables |
| **Import/Export Service** | Ship physical disks to Azure (large data transfers) |
| **Azure Data Box** | Physical device for massive data transfer |

### AzCopy Key Commands
```bash
azcopy login                  # Authenticate with Entra ID
azcopy copy <source> <dest>   # Copy files/blobs
azcopy sync <source> <dest>   # Sync (incremental)
azcopy make <containerURL>    # Create container
```

---

## 🖥️ DOMAIN 3 — DEPLOY & MANAGE COMPUTE RESOURCES

### Virtual Machine Creation Checklist
| Setting | Key Decisions |
|---------|---------------|
| **Region** | Latency, compliance, service availability |
| **Size (SKU)** | vCPUs, RAM, disk IOPS (can resize later) |
| **Image** | OS (Windows/Linux), Marketplace or custom |
| **Authentication** | Password or SSH key (Linux) |
| **Disks** | OS disk + Data disks (Standard/Premium/Ultra) |
| **Networking** | VNet, subnet, NSG, public IP |
| **Extensions** | Custom Script, DSC, monitoring agents |

### VM Sizes (Families)
| Series | Optimized For | Example Use Case |
|--------|--------------|-----------------|
| **B** | Burstable | Dev/test, low-traffic web |
| **D/Ds** | General purpose | Most production workloads |
| **E/Es** | Memory-optimized | Databases, in-memory caching |
| **F/Fs** | Compute-optimized | Batch, gaming, analytics |
| **L** | Storage-optimized | Large databases, data warehousing |
| **N** | GPU | ML, rendering, HPC |
| **M** | Memory (very high) | SAP HANA, large in-memory DBs |

### VM Disks
| Disk Type | IOPS | Use Case |
|-----------|------|----------|
| **Standard HDD** | 500 | Dev/test, backups |
| **Standard SSD** | 6,000 | Light production |
| **Premium SSD** | 20,000 | Production databases |
| **Premium SSD v2** | 80,000 | High-performance workloads |
| **Ultra Disk** | 160,000 | Extreme IOPS (SAP, top-tier DBs) |

> **Premium SSD** required for single-instance 99.9% SLA

### Availability Options
| Option | SLA | Protects Against |
|--------|-----|------------------|
| **Single VM (Premium SSD)** | 99.9% | — |
| **Availability Set** | 99.95% | Rack/power failure |
| **Availability Zone** | 99.99% | Datacenter failure |

**Availability Set:**
- Fault Domains (FDs): Separate physical racks (2-3 per set)
- Update Domains (UDs): Separate maintenance groups (up to 20)
- VMs must be in same region

**Availability Zone:**
- Physically separate datacenters within a region
- Independent power, cooling, networking
- Not all regions support zones

### Virtual Machine Scale Sets (VMSS)
- Group of identical, load-balanced VMs
- **Autoscale:** Scale out/in based on metrics (CPU, memory, custom)
- **Orchestration modes:** Uniform (identical) or Flexible (mixed)
- Supports up to 1,000 VMs (100 with custom images)
- Use with Load Balancer or Application Gateway

### VM Extensions
| Extension | Purpose |
|-----------|---------|
| **Custom Script Extension** | Run scripts on VM at deploy/post-deploy |
| **DSC Extension** | Desired State Configuration |
| **Azure Monitor Agent** | Collect metrics and logs |
| **Network Watcher Agent** | Network monitoring |
| **Disk Encryption (ADE)** | BitLocker (Windows) / DM-Crypt (Linux) |

### Azure App Service
| Concept | Detail |
|---------|--------|
| **App Service Plan** | Defines compute resources (size, scaling, OS) |
| **Deployment Slots** | Separate environments (staging, production); swap with no downtime |
| **Scaling** | Scale up (change plan) or scale out (add instances, autoscale) |
| **Custom domains** | Map custom DNS to App Service |
| **TLS/SSL** | Free managed certificate or bring your own |
| **Always On** | Keeps app loaded (not available in Free/Shared) |

### App Service Plan Tiers
| Tier | Key Features |
|------|-------------|
| **Free / Shared** | No SLA, shared infra, no slots |
| **Basic** | Dedicated, manual scale, no slots |
| **Standard** | Auto-scale, 5 slots, TLS |
| **Premium** | More slots (20), VNet integration, higher scale |
| **Isolated** | ASE, full network isolation |

### Deployment Slots
- Each slot is a live app with its own URL
- **Swap:** Routes traffic from one slot to another (zero-downtime deployment)
- **Slot settings:** Some settings "stick" to slot (connection strings marked as slot setting)
- **Traffic routing:** Split traffic between slots (A/B testing)
- Auto-swap: Auto-swap staging to production after deployment

### Containers
| Service | Use Case | Key Fact |
|---------|----------|----------|
| **Azure Container Registry (ACR)** | Store container images | Private Docker registry; tiers: Basic, Standard, Premium |
| **Azure Container Instances (ACI)** | Run single containers quickly | Serverless, no orchestration; seconds to start |
| **Azure Kubernetes Service (AKS)** | Full container orchestration | Managed K8s; node pools, scaling, networking |

### ARM Templates & Bicep
| Feature | ARM Template | Bicep |
|---------|-------------|-------|
| Format | JSON | DSL (cleaner syntax) |
| Modularity | Linked/nested templates | Modules |
| Parameters | Separate .parameters.json | Inline or .bicepparam |
| Deployment | `az deployment group create` | Same (compiles to ARM) |

### Deployment Modes
| Mode | Behavior |
|------|----------|
| **Incremental** (default) | Adds/updates resources; leaves existing resources unchanged |
| **Complete** | Adds/updates resources; **DELETES** resources not in template |

> ⚠️ **Complete mode can delete resources!** Always use `what-if` before deploying in Complete mode.

---

## 🌐 DOMAIN 4 — IMPLEMENT & MANAGE VIRTUAL NETWORKING

### VNet Key Facts
| Property | Detail |
|----------|--------|
| **Address space** | Private IP ranges (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) |
| **Subnets** | Segment VNet; each subnet uses a contiguous portion of address space |
| **Reserved IPs** | Azure reserves 5 IPs per subnet (.0, .1, .2, .3, .255) |
| **Region-bound** | VNet exists in one region only |
| **Subscription-bound** | VNet belongs to one subscription |

### IP Addressing
| Type | Description | Billing |
|------|-------------|---------|
| **Dynamic Public IP** | Changes on deallocation | Free when attached to running VM |
| **Static Public IP** | Fixed IP, persists through stop/start | Charged when not in use |
| **Private IP** | Internal VNet communication | Always free |

> **Static Public IP** is required for: VPN Gateways, Application Gateways, and scenarios needing fixed IPs

### Network Security Groups (NSGs)
- Layer 4 firewall (IP, port, protocol)
- Applied to **subnet** or **NIC** (both can apply simultaneously)
- Rules evaluated by **priority** (100-4096; lower number = higher priority)
- Default rules: Allow VNet inbound/outbound, Allow LB inbound, Deny all inbound

### NSG Rule Properties
| Property | Description |
|----------|-------------|
| **Priority** | 100-4096 (lower = evaluated first) |
| **Source/Destination** | IP, CIDR, Service Tag, or ASG |
| **Protocol** | TCP, UDP, ICMP, Any |
| **Port range** | Single, range (80-443), or * (all) |
| **Action** | Allow or Deny |

### Service Tags (Common)
| Tag | Represents |
|-----|-----------|
| **Internet** | All public IPs |
| **VirtualNetwork** | VNet + peered VNets + on-prem (VPN/ER) |
| **AzureLoadBalancer** | Azure LB health probes |
| **Storage** | Azure Storage IPs |
| **Sql** | Azure SQL IPs |
| **AzureCloud** | All Azure datacenter IPs |

### Application Security Groups (ASGs)
- Group VMs by application role (e.g., "WebServers," "DBServers")
- Use in NSG rules instead of individual IPs
- Simplifies rule management for multi-tier apps

### Azure DNS
| Type | Description |
|------|-------------|
| **Public DNS Zone** | Host public domain records in Azure |
| **Private DNS Zone** | Name resolution within VNets (no public exposure) |
| **Alias Record Set** | Point to Azure resource (LB, Front Door, Traffic Manager) directly |

> **Private DNS Zone** must be **linked** to a VNet for resolution to work. Auto-registration creates records automatically.

### VNet Peering
| Property | Detail |
|----------|--------|
| **Non-transitive** | A↔B + B↔C ≠ A↔C |
| **Cross-region** | Global VNet Peering (higher latency, cost) |
| **Gateway transit** | Share VPN/ER gateway through peered VNet |
| **No IP overlap** | Peered VNets cannot have overlapping address spaces |
| **Bi-directional setup** | Must configure on BOTH VNets |

### VPN Gateway
| Type | Description |
|------|-------------|
| **Site-to-Site (S2S)** | On-prem to Azure over IPsec/IKE tunnel |
| **Point-to-Site (P2S)** | Individual client to Azure VPN (remote workers) |
| **VNet-to-VNet** | Azure VNet to Azure VNet over VPN |

**SKU matters:** Basic (dev/test), VpnGw1-5 (production), generation 1 vs 2

### ExpressRoute
- Private, dedicated connection to Azure (not over public internet)
- Provider-based (ISP peering) or Direct (physical port to Microsoft)
- Up to 100 Gbps; predictable latency
- Requires partner/provider for connectivity

### Azure Load Balancer
| Property | Public LB | Internal LB |
|----------|-----------|-------------|
| Frontend IP | Public IP | Private IP in VNet |
| Use case | Internet-facing apps | Internal multi-tier apps |
| SKU | Basic or **Standard** | Basic or **Standard** |

**Standard vs Basic LB:**
| Feature | Standard | Basic |
|---------|----------|-------|
| Backend pool | VMs in any single VNet | Availability Set only |
| Health probes | TCP, HTTP, HTTPS | TCP, HTTP |
| SLA | 99.99% | No SLA |
| Availability Zones | ✅ Zone-redundant | ❌ |
| Secure by default | ✅ (requires NSG to allow) | ❌ (open by default) |

> **Standard LB requires NSG** — traffic is blocked by default until you add an Allow rule

### Load Balancer Components
| Component | Purpose |
|-----------|---------|
| **Frontend IP** | Entry point (public or private IP) |
| **Backend Pool** | Target VMs/VMSS |
| **Health Probe** | Detect unhealthy instances |
| **Load Balancing Rule** | Map frontend port → backend port |
| **Inbound NAT Rule** | Port forwarding to specific VM |
| **Outbound Rule** | SNAT for outbound connections |

### Application Gateway
- Layer 7 (HTTP/HTTPS) load balancer
- **WAF** (Web Application Firewall) — OWASP protection
- **URL path-based routing** — route /images/* to one pool, /api/* to another
- **Multi-site hosting** — multiple websites on single App GW
- **SSL termination** — offload SSL at the gateway
- **Autoscaling** (v2 SKU)
- **Health probes** — HTTP-based, custom path

### Load Balancing Decision
```
Layer 4 (TCP/UDP) + Regional → Azure Load Balancer
Layer 7 (HTTP/HTTPS) + Regional + WAF → Application Gateway
Global DNS-based (any protocol) → Traffic Manager
Global HTTP/HTTPS + WAF + CDN → Azure Front Door
```

---

## 📊 DOMAIN 5 — MONITOR & MAINTAIN AZURE RESOURCES

### Azure Monitor Architecture
```
Data Sources → Data Platform → Visualization/Actions
  Metrics (numeric, time-series)    → Metrics Explorer, Dashboards
  Logs (text, structured)           → Log Analytics (KQL queries)
  Activity Log (control plane)      → Alerts, Action Groups
```

### Data Types
| Type | Description | Retention |
|------|-------------|-----------|
| **Metrics** | Numeric time-series (CPU%, disk IO) | 93 days (auto) |
| **Logs** | Text/structured (events, traces) | Configurable (30-730 days) |
| **Activity Log** | Control-plane operations (who did what) | 90 days (auto) |

### Diagnostic Settings
- Route platform logs and metrics to: Log Analytics, Storage Account, Event Hub
- Must be configured per resource
- Enable to see resource-level logs (not collected by default)

### Alert Rule Components
| Component | Description |
|-----------|-------------|
| **Target resource** | What to monitor |
| **Condition** | Signal + threshold (e.g., CPU > 80%) |
| **Action Group** | What to do (email, SMS, webhook, Logic App, Azure Function) |
| **Severity** | 0 (Critical) to 4 (Verbose) |

### Alert Types
| Type | Based On | Example |
|------|----------|---------|
| **Metric Alert** | Metrics data | CPU > 80% for 5 minutes |
| **Log Alert** | KQL query results | Error count > 10 in last hour |
| **Activity Log Alert** | Activity Log events | VM deleted, policy assigned |
| **Service Health Alert** | Azure service issues | Region outage notification |

### Log Analytics & KQL Basics
```kusto
// Find errors in last 24 hours
AzureDiagnostics
| where TimeGenerated > ago(24h)
| where Level == "Error"
| summarize count() by ResourceId

// VM CPU usage
Perf
| where ObjectName == "Processor"
| where CounterName == "% Processor Time"
| summarize avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
```

### Azure Backup
| Component | Purpose |
|-----------|---------|
| **Recovery Services Vault** | Container for backup data (region-specific) |
| **Backup Policy** | Frequency (daily/weekly) + retention (days/weeks/months/years) |
| **MARS Agent** | Backup on-prem files/folders to Azure |
| **Azure Backup Server (MABS)** | Backup on-prem workloads (SQL, SharePoint, Exchange) |
| **VM Backup** | Application-consistent snapshots of Azure VMs |

### Backup Key Facts
- Vault must be in **same region** as resources being backed up
- **Soft delete:** 14 additional days of retention after deletion (enabled by default)
- **Instant restore:** Restore from local snapshot (before data reaches vault)
- **Cross-region restore:** Must enable on vault (supported with GRS vaults)

### VM Backup Restore Options
| Option | Description |
|--------|-------------|
| **Create a new VM** | Full VM from recovery point |
| **Restore disk** | Restore managed disks, then attach manually |
| **Replace existing** | Replace disks of existing VM |
| **File recovery** | Mount recovery point as drive, copy specific files |

### Network Watcher Tools
| Tool | Purpose |
|------|---------|
| **IP Flow Verify** | Test if traffic is allowed/denied by NSG |
| **Next Hop** | Show next hop for a specific destination |
| **Connection Troubleshoot** | End-to-end connectivity test |
| **NSG Flow Logs** | Log all NSG traffic decisions |
| **Topology** | Visual diagram of network resources |
| **Packet Capture** | Capture network packets on a VM |
| **Connection Monitor** | Ongoing connectivity monitoring |

---

## 🔑 KEY GOTCHAS & EXAM TRAPS

### Identity & Governance
- **Dynamic groups need P1/P2** — not available with Free Entra ID
- **RBAC role assigned at RG scope applies to ALL resources in that RG**
- **Resource locks override RBAC** — Owner can't delete a locked resource without removing lock
- **Tags are NOT inherited** — use Policy to enforce tag inheritance
- **Custom roles:** `AssignableScopes` cannot be Management Group in some scenarios
- **Deleted users recoverable for 30 days** (soft delete)

### Storage
- **Storage account name:** 3-24 chars, lowercase + numbers only, globally unique
- **Cannot change redundancy** from ZRS/GZRS to LRS/GRS on existing account (must create new)
- **Archive rehydration** takes hours — plan ahead
- **SAS with stored access policy** — can be revoked; SAS without policy can only expire
- **Firewall on storage** — must add exceptions for trusted Azure services
- **Azure Files SMB requires port 445** — often blocked by ISPs (test connectivity first)
- **File Sync:** Server endpoint cannot be root of a volume (need subfolder)

### Compute
- **Stop (from OS) ≠ Deallocate** — OS stop keeps VM allocated (still billed for compute)
- **Resize requires deallocation** if new size isn't available in current cluster
- **Custom images:** Generalized (sysprep) = new VMs; Specialized = clone of existing
- **VMSS Uniform mode:** All VMs identical; Flexible mode: mixed configurations
- **App Service deployment slot swap** — settings marked "slot setting" DON'T move with app
- **ACI restart policy:** Always (default), OnFailure, Never
- **ACR Premium required** for geo-replication and private endpoints

### Networking
- **VNet Peering is NOT transitive** — A↔B + B↔C ≠ A↔C
- **NSG on subnet AND NIC** — both evaluated; most restrictive wins
- **Standard LB requires NSG** to allow traffic (Basic LB does not)
- **5 IPs reserved per subnet** — /29 is smallest usable subnet (3 usable IPs)
- **P2S VPN clients** must download client config after changes
- **ExpressRoute circuits** — not encrypted by default (add IPsec if needed)
- **Private DNS auto-registration** — only one VNet can have auto-registration enabled per zone

### Monitoring
- **Activity Log** retains 90 days — send to Log Analytics for longer retention
- **Diagnostic settings** per-resource — not enabled by default
- **Alert action groups** — max 1 SMS per 5 min, 1 voice per 5 min (rate limiting)
- **Backup vault region** must match resource region
- **Soft delete on Recovery Services Vault** — can't delete vault with protected items

---

## 📋 QUICK REFERENCE — SERVICE LIMITS & KEY NUMBERS

| Item | Limit |
|------|-------|
| VNets per subscription per region | 1,000 |
| Subnets per VNet | 3,000 |
| NSGs per subscription per region | 5,000 |
| NSG rules per NSG | 1,000 |
| Public IPs per subscription per region | 1,000 |
| VNet peerings per VNet | 500 |
| NIC per VM | Depends on size (2-8 typical) |
| VMs per Availability Set | 200 |
| Fault Domains per Availability Set | 2-3 |
| Update Domains per Availability Set | 5 (default), up to 20 |
| Tags per resource | 50 |
| Storage accounts per subscription per region | 250 |
| Max blob size (block blob) | 190.7 TiB |
| Max file share size | 100 TiB |
| Deployment slots (Standard plan) | 5 |
| Deployment slots (Premium plan) | 20 |
| Management group depth | 6 levels (below root) |

---

## ⌨️ ESSENTIAL CLI / POWERSHELL COMMANDS

### Resource Management
```bash
# CLI
az group create --name MyRG --location eastus
az group delete --name MyRG --yes --no-wait
az resource list --resource-group MyRG --output table

# PowerShell
New-AzResourceGroup -Name MyRG -Location eastus
Remove-AzResourceGroup -Name MyRG -Force
Get-AzResource -ResourceGroupName MyRG
```

### Virtual Machines
```bash
# CLI
az vm create --resource-group MyRG --name MyVM --image Ubuntu2204 --size Standard_B2s --admin-username azureuser --generate-ssh-keys
az vm stop --resource-group MyRG --name MyVM
az vm deallocate --resource-group MyRG --name MyVM
az vm resize --resource-group MyRG --name MyVM --size Standard_D2s_v3

# PowerShell
New-AzVM -ResourceGroupName MyRG -Name MyVM -Image Ubuntu2204
Stop-AzVM -ResourceGroupName MyRG -Name MyVM -Force
Start-AzVM -ResourceGroupName MyRG -Name MyVM
```

### Storage
```bash
# CLI
az storage account create --name mystorageacct --resource-group MyRG --sku Standard_LRS
az storage container create --name mycontainer --account-name mystorageacct
az storage blob upload --container-name mycontainer --file ./data.txt --name data.txt --account-name mystorageacct

# PowerShell
New-AzStorageAccount -ResourceGroupName MyRG -Name mystorageacct -SkuName Standard_LRS -Location eastus
```

### Networking
```bash
# CLI
az network vnet create --resource-group MyRG --name MyVNet --address-prefix 10.0.0.0/16 --subnet-name default --subnet-prefix 10.0.0.0/24
az network nsg create --resource-group MyRG --name MyNSG
az network nsg rule create --resource-group MyRG --nsg-name MyNSG --name AllowHTTP --priority 100 --destination-port-ranges 80 --access Allow --protocol Tcp

# PowerShell
New-AzVirtualNetwork -ResourceGroupName MyRG -Name MyVNet -AddressPrefix 10.0.0.0/16 -Location eastus
```

---

## 🎯 DECISION TREES SUMMARY

### "Which storage redundancy?"
```
Need read access to secondary? → RA-GRS or RA-GZRS
Need zone protection + geo? → GZRS
Need zone protection only? → ZRS
Need geo protection only? → GRS
Single datacenter OK? → LRS (cheapest)
```

### "Which VM availability option?"
```
Protect against datacenter failure? → Availability Zone
Protect against rack failure only? → Availability Set
Single VM, need 99.9% SLA? → Premium SSD (no AS/AZ needed)
Dev/test, no SLA needed? → Single VM, Standard disk
```

### "Which load balancer?"
```
HTTP/HTTPS + Regional + WAF? → Application Gateway
TCP/UDP + Regional? → Azure Load Balancer (Standard)
HTTP/HTTPS + Global + CDN? → Azure Front Door
Any protocol + Global (DNS only)? → Traffic Manager
```

### "Which connectivity option?"
```
VNet to VNet (same/cross-region)? → VNet Peering (first choice)
On-prem to Azure (encrypted, internet)? → VPN Gateway (S2S)
On-prem to Azure (private, high bandwidth)? → ExpressRoute
Single user to Azure? → VPN Gateway (P2S)
```

---

*Last updated: May 24, 2026*
