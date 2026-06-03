# AZ-305 — Azure Migration Services
## Complete Study Guide: Purpose, Platforms, Requirements, Pros & Cons
> Exam: May 21, 2026 | Relevant Domain: Infrastructure Solutions (30–35%)

---

## Table of Contents
1. [Migration Strategy Framework — The 6 Rs](#1-migration-strategy-framework--the-6-rs)
2. [Azure Migrate — Server Migration](#2-azure-migrate--server-migration)
3. [Azure Database Migration Service (DMS)](#3-azure-database-migration-service-dms)
4. [App Service Migration Assistant](#4-app-service-migration-assistant)
5. [Azure Data Box Family — Offline Transfer](#5-azure-data-box-family--offline-transfer)
6. [Azure Storage Migration Service](#6-azure-storage-migration-service)
7. [Azure Site Recovery — DR / Workload Migration](#7-azure-site-recovery--workload-migration-use)
8. [Azure VMware Solution (AVS)](#8-azure-vmware-solution-avs)
9. [SQL Server to Azure — Migration Paths](#9-sql-server-to-azure--migration-paths)
10. [SAP on Azure Migration](#10-sap-on-azure-migration)
11. [Cross-Cloud Migration (AWS / GCP → Azure)](#11-cross-cloud-migration-aws--gcp--azure)
12. [Migration Tooling Comparison](#12-migration-tooling-comparison)
13. [Exam Tips — Migration](#13-exam-tips--migration)

---

## 1. Migration Strategy Framework — The 6 Rs

| Strategy | Also Called | Description | Effort | Cost Change | Risk |
|----------|------------|-------------|--------|------------|------|
| **Rehost** | Lift & Shift | Move to Azure IaaS as-is, no code changes | Low | Minimal | Low |
| **Replatform** | Lift, Tinker & Shift | Minor cloud optimizations (e.g., move SQL Server → Azure SQL DB) | Medium | Moderate savings | Medium |
| **Refactor** | Re-architect | Redesign app for cloud-native (containers, microservices, PaaS) | High | Significant savings long-term | High |
| **Repurchase** | Drop & Shop | Replace with SaaS (e.g., on-prem CRM → Dynamics 365 / Salesforce) | Medium | Opex shift | Medium |
| **Retire** | Decommission | Shut down apps that are no longer needed | Low | Cost savings | None |
| **Retain** | Revisit | Keep on-prem for now (compliance, low ROI, complex dependency) | None | No change | None |

### Decision Guidance

| Trigger | Recommended Strategy |
|---------|---------------------|
| Must meet deadline, minimal resources | **Rehost** |
| App has on-prem dependencies but DB can move | **Replatform** |
| Long-term investment; scalability required | **Refactor** |
| Legacy vendor software with cloud equivalent | **Repurchase** |
| Duplicate systems, unused apps | **Retire** |
| Compliance, data sovereignty, or not worth migrating | **Retain** |

> **Exam tip:** When two strategies seem valid, prefer the one with lowest risk that meets stated requirements. "Minimize migration effort" → Rehost. "Maximize long-term cloud benefit" → Refactor.

---

## 2. Azure Migrate — Server Migration

### Purpose
Central hub for **discovering, assessing, and migrating** on-premises and multi-cloud workloads to Azure. Provides a unified portal experience and integrates with Microsoft and third-party tools.

---

### Tools Within Azure Migrate

| Tool | Purpose | Who Provides |
|------|---------|-------------|
| **Azure Migrate: Discovery & Assessment** | Discover on-prem VMs + physical servers; assess readiness + cost estimate | Microsoft |
| **Azure Migrate: Server Migration** | Replicate and migrate VMs to Azure | Microsoft |
| **Azure Migrate: Database Assessment** | Assess SQL Server databases for Azure readiness | Microsoft (DMA integration) |
| **Azure Migrate: Web App Assessment** | Discover and assess .NET / Java web apps | Microsoft |
| **Third-party ISV tools** | Specialized migration tools (Carbonite, Corent, Zerto, etc.) | ISV partners |

---

### Supported Source Platforms

| Source | Discovery Method | Migration Method |
|--------|----------------|-----------------|
| **VMware vSphere** | Agentless appliance | Agentless replication (preferred) OR agent-based |
| **Microsoft Hyper-V** | Lightweight Hyper-V agent | Hyper-V replication provider |
| **Physical servers** | Agent-based (Mobility Service) | Agent-based only |
| **AWS EC2** | Appliance discovery | Agent-based replication |
| **GCP Compute Engine** | Appliance discovery | Agent-based replication |
| **Other cloud VMs** | Agent-based | Agent-based replication |

---

### Migration Modes

| Mode | How It Works | Agent on VM? | Downtime | Best For |
|------|-------------|-------------|---------|---------|
| **Agentless (VMware only)** | Uses vSphere APIs via appliance; no agent needed on VMs | ❌ None | Minimal (final delta sync) | VMware environments; less disruption |
| **Agent-based** | Mobility Service agent installed on each VM | ✅ Required | Minimal (final delta sync) | Physical servers, Hyper-V, non-VMware cloud |

---

### Discovery & Assessment — How It Works

```
Step 1: Deploy Azure Migrate Appliance (lightweight VM in your environment)
  └── Connects to vCenter / Hyper-V host / physical network
  └── Discovers: VMs, disks, network config, performance data, apps, dependencies

Step 2: Azure Migrate analyzes data and generates:
  ├── Azure Readiness Report (Ready / Conditionally Ready / Not Ready / Unknown)
  ├── Right-sized Azure VM recommendation (performance-based or as-is)
  ├── Monthly cost estimate (compute + storage)
  └── Dependency map (requires agents for agentless dependency analysis)

Step 3: Define migration groups → Create assessment
  └── Choose: Performance-based OR As on-premises sizing
  └── Choose: Target region, VM series, pricing model
```

### Assessment Output Details

| Output | Description |
|--------|-------------|
| **Azure Readiness** | Ready / Conditionally Ready (warnings) / Not Ready (blockers) / Unknown |
| **Recommended Azure VM size** | Based on CPU + memory utilization (percentile configurable: 50th, 95th, 99th) |
| **Storage recommendation** | Standard or Premium based on IOPS |
| **Monthly cost estimate** | Compute + Storage, with/without AHUB and Reserved Instances |
| **Confidence rating** | Based on data points collected (1–5 stars) |
| **Migration Issues** | Specific blockers: OS not supported, boot type (UEFI restrictions), disk size, etc.) |

---

### Server Migration Process

```
Phase 1: Replicate
  └── Enable replication → initial delta sync to Azure (recovery services vault)
  └── Ongoing delta sync (near-continuous)

Phase 2: Test Migration
  └── Launch test VM in isolated VNet → validate app works
  └── Clean up test migration

Phase 3: Migrate (Cutover)
  └── Final sync of deltas
  └── Shut down on-prem VM (optional but recommended)
  └── Cut over → VM running in Azure
  └── Cleanup: Remove on-prem replication
```

---

### Requirements

| Requirement | Detail |
|-------------|--------|
| **Azure Migrate project** | Create in Azure portal (resource in a Resource Group) |
| **Azure Migrate Appliance** | OVF/VHD deployed in on-prem env; needs outbound HTTPS (443) to Azure |
| **vCenter permissions (VMware agentless)** | Read-only on vCenter; additional permissions for snapshot-based replication |
| **OS support** | Windows Server 2003+, major Linux distros (RHEL, CentOS, Ubuntu, Debian, SUSE) |
| **Disk limitations** | Max 32 disks per VM (agentless); Max disk size 4 TB per disk |
| **Network** | VPN or ExpressRoute recommended for production; public internet possible |
| **Resource Group (Azure)** | Target resource group + VNet must exist before migration |

---

### Pros & Cons — Azure Migrate (Server Migration)

| ✅ Pros | ❌ Cons |
|---------|--------|
| Unified portal for discovery + assessment + migration | Agentless only for VMware; physical needs agent |
| Agentless VMware replication = no agent footprint | Initial assessment requires time for perf data collection (2–4 weeks for best quality) |
| Right-sizing recommendations reduce overprovisioning | Test migration strongly recommended (can't skip in production) |
| Integrated with AHUB, Reserved Instances cost estimates | Dependent on appliance connectivity; network interruptions can stall replication |
| Free to use (no tool cost for most migration tools) | Max 500 VMs per appliance (scale requires multiple appliances) |
| Supports multi-cloud (AWS, GCP discovery) | Complex networking (domain controllers, shared storage) needs manual planning |
| Generates dependency maps to identify migration groups | Performance-based sizing requires 2+ weeks of data for accuracy |
| Integrates with Azure Site Recovery for DR post-migration | Limited to OS-level migration; application reconfiguration is manual |

---

## 3. Azure Database Migration Service (DMS)

### Purpose
Managed service for **migrating databases** from on-premises or other cloud providers to Azure database platforms with **minimal downtime** (online mode) or with a maintenance window (offline mode).

---

### Tiers & Modes

| Tier | Pricing | Downtime Mode | Use For |
|------|---------|--------------|---------|
| **Free (Standard)** | Free | **Offline only** (requires downtime) | Small migrations, dev/test, scripts |
| **Premium** | Per SKU/hour | **Online (minimal downtime)** via CDC | Production, mission-critical |

| Mode | How It Works | Downtime | When to Use |
|------|------------|---------|------------|
| **Offline** | Full backup restore; cutover requires downtime | Hours | Acceptable maintenance window |
| **Online** | Full backup + continuous CDC (Change Data Capture) replication; cutover = minutes | Minutes | Mission-critical; < 30 min downtime target |

---

### Supported Migration Paths

| Source | Target | Offline | Online (CDC) | Notes |
|--------|--------|---------|-------------|-------|
| **SQL Server (on-prem / EC2 / IaaS VM)** | Azure SQL Database | ✅ | ✅ | Most common migration |
| **SQL Server** | Azure SQL Managed Instance | ✅ | ✅ | Lift & shift SQL Server |
| **SQL Server** | SQL Server on Azure VM | ✅ | ✅ Limited | Backup/restore preferred |
| **MySQL (on-prem / AWS RDS)** | Azure DB for MySQL Flexible Server | ✅ | ✅ | |
| **PostgreSQL (on-prem / AWS RDS)** | Azure DB for PostgreSQL Flexible Server | ✅ | ✅ | |
| **MongoDB (on-prem / Atlas)** | Azure Cosmos DB for MongoDB | ✅ | ✅ | |
| **Oracle** | Azure DB for PostgreSQL | ✅ | ❌ | Via ora2pg tool pre-conversion |
| **RDS SQL Server** | Azure SQL Database / MI | ✅ | ✅ | |

---

### Migration Prerequisites

| Prerequisite | Detail |
|-------------|--------|
| **Source firewall** | Allow DMS service IPs to connect to source DB |
| **SQL Server source requirements** | sysadmin role; TCP/IP enabled; SQL Agent for log shipping |
| **SQL MI target network** | DMS must be deployed in same VNet as SQL MI, or VNet peered to it |
| **SQL DB target** | Firewall rule to allow DMS service tag or IP |
| **DMS subnet** | Dedicated subnet in VNet; /28 minimum |
| **Database collation** | Source and target must be compatible |
| **CDC (online mode)** | SQL Server: CDC enabled on source DB; requires SQL Server 2008+ SP1+ |

---

### SQL Migration Process (Online)

```
Phase 1: Full Migration
  └── DMS takes full backup of source DB
  └── Restores to Azure target

Phase 2: Incremental Sync (CDC)
  └── Continuously replicates changes from source transaction log
  └── Target stays near-current with source

Phase 3: Cutover
  └── Stop all writes to source
  └── Wait for "Pending changes = 0"
  └── Initiate cutover in DMS portal
  └── Update connection strings to point to Azure target
  └── Total cutover window: minutes
```

---

### Pros & Cons — Azure DMS

| ✅ Pros | ❌ Cons |
|---------|--------|
| Online mode = near-zero downtime for production DBs | Online mode (CDC) requires Premium tier (paid) |
| Supports most major relational + NoSQL platforms | Oracle migration requires pre-conversion with ora2pg |
| Fully managed service (no infrastructure management) | Source DB must allow DMS connectivity (firewall changes) |
| Integrated with Azure Migrate project | Complex migrations (CLR, linked servers) may require manual remediation |
| Pre-migration assessment identifies compatibility issues | SQL MI migrations require DMS in same/peered VNet |
| Supports cross-cloud (AWS RDS, MySQL, PostgreSQL) | Limited support for exotic collations or non-standard SQL features |
| Free tier available for offline/simple migrations | Online CDC requires specific SQL Server versions (2008 SP1+) |
| Schema migration support via Database Migration Assessment | Large databases (TBs) may take days for initial full backup |

---

### Database Migration Assessment Tools

| Tool | Purpose | Use With |
|------|---------|---------|
| **Database Migration Assistant (DMA)** | Assess SQL Server compatibility for Azure SQL DB / MI | SQL Server → Azure SQL |
| **Azure Database Migration Guide** | Microsoft-guided migration learning + tooling | All database platforms |
| **SQL Server Migration Assistant (SSMA)** | Schema + data conversion for Oracle, MySQL, Sybase, Access | Heterogeneous migrations |
| **Data Migration Assistant in Azure Migrate** | Integrated assessment within Azure Migrate hub | SQL Server |
| **ora2pg** | Convert Oracle schema + data to PostgreSQL format | Oracle → PostgreSQL |

---

## 4. App Service Migration Assistant

### Purpose
Free tool that **discovers, assesses, and migrates** ASP.NET and Java web applications from on-premises IIS servers to **Azure App Service** with minimal effort.

---

### Supported Source & Target

| Source | Target |
|--------|-------|
| IIS on Windows Server (ASP.NET / Classic ASP) | Azure App Service (Windows) |
| Apache / Tomcat on Linux (Java) | Azure App Service (Linux — Java) |
| Standalone .NET Core / .NET 5+ | Azure App Service |

---

### Migration Assessment Checks

| Check Category | What's Validated |
|---------------|-----------------|
| **App compatibility** | .NET version support, 32/64-bit, Global.asax, HttpModules |
| **Authentication** | Windows Auth (not supported on App Service — suggest OIDC alternative) |
| **Port bindings** | Only HTTP/HTTPS; other ports not supported |
| **Virtual directories** | Multiple virtual dirs per site |
| **Dependencies** | COM/GAC dependencies, local file system access |
| **Database connectivity** | Connection string detection |

---

### Migration Steps

```
1. Install Migration Assistant on IIS server (free download — Microsoft)
2. Tool scans all IIS sites → readiness report per site
3. Select compatible site → login to Azure
4. Tool provisions App Service Plan + App Service
5. Packages + deploys site content and config
6. (Manual) Update DNS / connection strings
7. Test & validate
```

---

### Pros & Cons — App Service Migration Assistant

| ✅ Pros | ❌ Cons |
|---------|--------|
| Free tool; zero code changes for compatible apps | Only supports IIS / Java web apps (not .NET Windows Services, etc.) |
| Automated assessment reduces risk | Windows Authentication not supported on App Service without rework |
| One-click migration for compatible sites | COM/GAC dependencies = blockers; must resolve before migration |
| Deploys directly to Azure (no manual Zip deploy) | Limited to App Service target only (not AKS, Container Apps) |
| Outputs detailed reports with remediation guidance | Does NOT migrate databases — only the web tier |
| Integrates with Azure Migrate hub | Multi-site IIS deployments require multiple migration steps |
| Handles web.config transformation | Java migration requires additional configuration for Tomcat settings |

---

## 5. Azure Data Box Family — Offline Transfer

### Purpose
Physical devices shipped by Microsoft for **bulk data transfer** to/from Azure when network bandwidth is insufficient, too slow, or too expensive.

---

### Product Comparison

| Product | Usable Capacity | Interface | Transfer Speed | Use Case | Encryption | Cost |
|---------|----------------|-----------|---------------|---------|-----------|------|
| **Data Box Disk** | 35 TB usable (5 × 8 TB SSDs) | USB 3.1 / SATA | ~430 MB/s per disk | Small orgs, < 35 TB, quick copy | AES 128-bit | Low |
| **Data Box** | 80 TB usable | 1 GbE / 10 GbE / 25 GbE / 40 GbE | Up to 1 Gbps | Medium datasets 40–80 TB | AES 256-bit | Medium |
| **Data Box Heavy** | ~770 TB usable | 40 GbE (×2) | Up to 80 Gbps | Large enterprises, petabyte-scale | AES 256-bit | High |
| **Data Box Gateway** | Virtual (NAS gateway) | 1 GbE / 10 GbE (software virtual appliance) | Up to 2 Gbps | Ongoing cloud archival, tiering | AES 256-bit | Subscription per device |

---

### Use Cases: Import vs Export

| Direction | Description | Use Case |
|-----------|-------------|---------|
| **Import to Azure** | Ship device to Microsoft datacenter → data loaded to Azure Storage | Large migration, initial data load for analytics, compliance archive |
| **Export from Azure** | Microsoft loads your Azure Blob/ADLS data → ships device to you | Disaster recovery data retrieval, regulatory data export, branch office |

---

### When to Use Data Box vs Network Transfer

| Condition | Recommendation |
|-----------|--------------|
| Data size > 40 TB | **Data Box** (network would take too long) |
| Network bandwidth < 100 Mbps sustained | **Data Box** |
| Regulatory / compliance prohibits internet transfer | **Data Box** |
| One-time bulk initial migration | **Data Box** |
| Ongoing continuous replication (< 1 TB/day) | Network transfer (AzCopy, Azure File Sync, ExpressRoute) |

### Bandwidth vs Data Box Breakeven (Rough Guide)

| Internet Bandwidth | Time for 100 TB via Network | Use Data Box? |
|-------------------|----------------------------|--------------|
| 100 Mbps | ~90 days | ✅ Yes |
| 1 Gbps | ~9 days | ✅ Likely yes |
| 10 Gbps | ~22 hours | Consider cost vs. speed |
| 100 Gbps | ~2 hours | ❌ Network is faster |

---

### Data Box Process

```
Import to Azure:
1. Order Data Box in Azure portal
2. Microsoft ships device (within a week typically)
3. Customer copies data to device (network or direct attach)
4. Ship device back to Microsoft
5. Microsoft uploads data to Azure Storage Account
6. Receive completion email; data available in Azure
7. Microsoft wipes device (NIST 800-88 certified)

Export from Azure:
1. Order export Data Box in Azure portal → specify source Storage Account
2. Microsoft copies your Azure data to device
3. Ships device to you
4. Copy data locally
5. Ship device back
6. Microsoft wipes device
```

---

### Requirements

| Requirement | Detail |
|-------------|--------|
| **Storage Account type** | Azure Blob Storage (Block, Page, Append) or Azure Files |
| **Storage Account region** | Must match Data Box order region |
| **Network for setup** | 1 GbE management port for device configuration + local copy |
| **Operating System (Data Box Disk)** | Windows 10 / Server 2012+ (SMB copy); Linux for SATA |
| **Firewall** | Allow Data Box device to reach Azure Storage endpoints for validation |
| **Maximum file path** | 768 characters (Windows path limit) |
| **Blocked file types** | System files ($Recycle.Bin, pagefile.sys, etc.) |

---

### Pros & Cons — Data Box

| ✅ Pros | ❌ Cons |
|---------|--------|
| Fastest option for > 40 TB transfers | Not suitable for ongoing replication (one-time or periodic) |
| No internet bandwidth consumed for bulk data | Physical shipping adds 1–2 weeks to migration timeline |
| AES 256-bit encryption on device | Must plan carefully for file path length limits |
| Secure wipe certified (NIST 800-88) after use | Must have compatible Storage Account (GPv2 recommended) |
| Available in most Azure regions | Rental/usage fee + shipping cost (not free) |
| Import AND export directions supported | Cannot import directly to Azure SQL, Cosmos DB — must stage in Blob |
| Large-scale: Data Box Heavy handles ~770 TB per shipment | Data Box Disk limited to 35 TB usable only |
| Managed service — Microsoft handles logistics + tracking | Longer lead time for international shipping |

---

## 6. Azure Storage Migration Service

### Purpose
Migrates file data from **Windows file servers or NAS devices** to **Azure Files** or to **Azure VMs** running Windows file server roles — without requiring downtime.

---

### Key Features

| Feature | Description |
|---------|-------------|
| **Inventory** | Scan source to catalog files, permissions, timestamps |
| **Assessment** | Identify migration blockers (long paths, unsupported chars) |
| **Transfer** | Copy data + NTFS permissions + ACLs + timestamps to target |
| **Cutover** | Redirect client access from source to destination with minimal disruption |
| **Orchestrator** | Windows Server running SMS orchestrator role |
| **Proxy** | Installed on source server or separate proxy VM for remote shares |

### Supported Sources & Targets

| Source | Target |
|--------|--------|
| Windows Server (any version) | Azure Files (Premium or Standard) |
| Windows Server | Windows Server on Azure VM |
| NAS devices (SMB/NFS) | Azure Files |
| Linux file servers | Windows Server on Azure VM (SMB shares) |
| Amazon S3 | Azure Blob Storage |

---

### Pros & Cons — Storage Migration Service

| ✅ Pros | ❌ Cons |
|---------|--------|
| Preserves NTFS permissions and ACLs | Windows Server required for orchestrator role |
| Near-zero downtime cutover | Not suitable for very large (PB-scale) migrations without planning |
| GUI-based (Windows Admin Center) | Requires connectivity between orchestrator and source/target |
| Supports NAS devices | No direct support for cloud-to-cloud (S3 → Azure Blob basic only) |
| Pre-migration assessment identifies issues | Large file counts (millions) can be slow to inventory |
| Free (included in Windows Server) | |

---

## 7. Azure Site Recovery — Workload Migration Use

### Primary Purpose
Although primarily a **Disaster Recovery** service, ASR is also widely used as a **migration tool** — especially for moving on-premises VMs to Azure with minimal downtime.

> See full ASR detail in the [AZ-305-Azure-Services-Reference.md](AZ-305-Azure-Services-Reference.md). This section focuses on its migration use.

---

### Migration Use Cases

| Scenario | RPO | Notes |
|----------|-----|-------|
| VMware VMs → Azure | 30 seconds | Mobility Service required |
| Hyper-V VMs → Azure | 30 seconds | Hyper-V Recovery Manager |
| Physical servers → Azure | ~1 hour | Agent-based |
| AWS EC2 → Azure | ~1 hour | Agent-based (not typical — use Azure Migrate) |

### ASR vs Azure Migrate for Server Migration

| | Azure Migrate | Azure Site Recovery |
|-|--------------|-------------------|
| Primary purpose | Migration | Disaster Recovery |
| Assessment | ✅ Full assessment | ❌ No assessment |
| Cost estimation | ✅ | ❌ |
| Dependency mapping | ✅ | ❌ |
| Agentless (VMware) | ✅ | ❌ (always agent-based) |
| Post-migration DR | ❌ (requires ASR separately) | ✅ DR continues after migration |
| Recommended for migration | ✅ **Preferred** | For DR-first scenarios |

> **Exam tip:** For migration workloads, use **Azure Migrate** (preferred). Use **ASR** when you also need ongoing DR or when migrating unsupported source types.

---

## 8. Azure VMware Solution (AVS)

### Purpose
Run **VMware workloads natively in Azure** using familiar VMware tools (vSphere, vCenter, NSX, vSAN) without refactoring. Microsoft manages the physical infrastructure and VMware stack.

---

### Platform & Requirements

| Feature | Detail |
|---------|--------|
| **VMware stack** | vSphere, vCenter Server, NSX-T, vSAN, HCX |
| **Minimum cluster size** | 3 hosts (required for vSAN) |
| **Maximum hosts per cluster** | 16 |
| **Maximum clusters per private cloud** | 12 |
| **Host type** | Bare-metal dedicated Azure hardware |
| **Storage** | vSAN (all-flash NVMe SSD); optional Azure NetApp Files for additional NFS |
| **Networking** | ExpressRoute (mandatory) to Azure or on-prem |
| **Migration tool** | VMware HCX (included; hot/cold/bulk migration) |
| **IP addressing** | /22 CIDR block required for management network |
| **License** | VMware licenses included in AVS price |

### AVS Migration Methods (via HCX)

| HCX Method | Downtime | Description | Use Case |
|-----------|---------|-------------|---------|
| **Cold Migration** | Yes (VM must be off) | Moves offline VMs | Dev/test, low-priority VMs |
| **Bulk Migration** | Minimal (cutover only) | Scheduled replication then cutover | Large-scale migration waves |
| **Live Migration (vMotion)** | Zero downtime | Live vMotion across datacenter | Production VMs requiring zero downtime |
| **OS-assisted Migration (RAV)** | Minimal | For VMs not compatible with vMotion | Non-vMotion-compatible VMs |

---

### Pros & Cons — Azure VMware Solution

| ✅ Pros | ❌ Cons |
|---------|--------|
| Zero refactoring — same VMware tools, skills | Very expensive (dedicated bare-metal hosts) |
| Live migration (zero downtime) via HCX vMotion | Minimum 3 hosts = high baseline cost |
| Full VMware stack: vCenter, NSX, vSAN, vMotion | ExpressRoute required (additional cost + setup time) |
| Microsoft manages physical infra + VMware patching | /22 CIDR block required (may conflict with existing IP scheme) |
| VMware licenses included | Not truly PaaS — still managing VMs |
| Gradual migration path; extend on-prem to Azure | NOT a long-term target — purpose is eventual modernization |
| SLA: 99.9% | Scale out takes time (~hours to add host) |
| Connectivity to Azure native services (Blob, SQL, etc.) | Still requires VMware expertise to manage |

---

### When to Choose AVS

| Scenario | Use AVS? |
|----------|---------|
| Large VMware estate, must migrate fast with no refactoring | ✅ Yes |
| Hard contractual deadline; replatform/refactor not feasible | ✅ Yes |
| Organization wants to keep VMware tools + skills | ✅ Yes |
| Small number of VMs (< 10) | ❌ No — higher per-unit cost vs Azure Migrate |
| Apps already containerized or cloud-native | ❌ No — use AKS or Container Apps |
| Budget is primary constraint | ❌ No — AVS is expensive |

---

## 9. SQL Server to Azure — Migration Paths

### Decision Tree

```
SQL Server on-prem migration:

Do you need 100% SQL Server compatibility (CLR, SQL Agent, linked servers)?
  YES → SQL Managed Instance (Lift & Shift — PaaS)
          OR SQL Server on Azure VM (IaaS — full control)

  NO → What is the primary workload?
        Transactional (OLTP), cloud-native → Azure SQL Database
        Very large DB (> 4 TB), Hyperscale → Azure SQL Database Hyperscale
        Analytics / Data Warehouse → Azure Synapse Analytics
        Need serverless / variable → Azure SQL Database Serverless
```

### Migration Tool by Target

| Source | Target | Recommended Tool | Notes |
|--------|--------|-----------------|-------|
| SQL Server | Azure SQL Database | **DMS (online)** + DMA assessment | Schema changes may be needed |
| SQL Server | SQL Managed Instance | **DMS (online)** | Closest to 1:1 migration |
| SQL Server | SQL Server on Azure VM | Backup/restore, Log shipping, Distributed AG | Infrastructure migration |
| SQL Server | Azure SQL Database | **BACPAC export/import** | Simple offline; small DBs only |
| SQL Server | SQL Managed Instance | **Azure Migrate** (DMS integration) | Used within Migrate hub |
| Oracle | Azure DB for PostgreSQL | DMS + **ora2pg** (schema conversion) | Requires schema mapping |
| MySQL | Azure DB for MySQL | **DMS** (online) | Minimal changes usually needed |
| PostgreSQL | Azure DB for PostgreSQL | **DMS** (online) | Near-transparent |
| MongoDB | Cosmos DB for MongoDB | **DMS** (online) | Native API compatibility |

### Key SQL Migration Considerations

| Factor | SQL Database | SQL Managed Instance | SQL on VM |
|--------|------------|---------------------|----------|
| Schema compatibility check | Required | Minimal | None |
| SQL Agent needed | ❌ | ✅ | ✅ |
| Linked servers needed | ❌ | ✅ | ✅ |
| Cross-DB queries needed | ❌ | ✅ (same MI) | ✅ |
| Custom CLR assemblies | ❌ | ✅ | ✅ |
| Migration downtime (online) | Minutes | Minutes | Minimal |
| Recommended migration tool | DMS / BACPAC | **DMS** | Backup/restore |
| Post-migration management | Fully managed | Mostly managed | Full OS/SQL mgmt |

---

### BACPAC vs DMS vs Backup/Restore

| Method | What It Migrates | Downtime | Max Size | Use Case |
|--------|----------------|---------|---------|---------|
| **BACPAC** (export/import) | Schema + data snapshot | Yes (offline) | ~5 GB practical | Small databases, dev/test |
| **DMS (offline)** | Full backup restore | Yes | Any size | Scheduled maintenance window |
| **DMS (online / CDC)** | Full + ongoing changes | Minutes only | Any size | Production, minimal downtime |
| **Backup/Restore** | Native SQL backup files | Yes | Any size | SQL MI or SQL on VM |
| **Log Shipping** | Initial backup + log chain | Minutes (cutover) | Any size | SQL MI, gradual cut-over |
| **Distributed Availability Group** | Live sync | Near-zero | Any size | SQL MI, zero-downtime enterprise |

---

## 10. SAP on Azure Migration

### Purpose
Move SAP landscapes (ECC, S/4HANA, BW/4HANA, HANA DB) to Azure using Microsoft's SAP on Azure validated configurations.

---

### Key Components

| Component | Azure Service | Notes |
|-----------|-------------|-------|
| **SAP HANA DB** | M-series or Mv2 VMs (certified) | Specific HANA-certified VM sizes required |
| **SAP Application Servers** | E-series / D-series VMs | Right-size with SAP sizing guidelines |
| **Shared storage** | Azure NetApp Files (ANF) | NFS for SAP shared dir, transport mounts |
| **HA for HANA** | HANA System Replication (HSR) + Pacemaker | Zone-redundant |
| **HA for SAP Apps** | ASCS/SCS cluster + Windows/Linux cluster | |
| **Backup** | Azure Backup (Backint) + Azure NetApp Files snapshots | SAP-certified backup |
| **Monitoring** | Azure Monitor SAP Solutions | SAP-specific metrics in Azure Monitor |
| **Migration method** | SWPM (SAP Software Provisioning Manager), SAP HANA cockpit | Microsoft + SAP tooling |

---

### Pros & Cons — SAP on Azure

| ✅ Pros | ❌ Cons |
|---------|--------|
| Certified SAP-on-Azure configurations | Requires SAP expertise alongside Azure |
| M-series VMs provide up to 4 TB+ RAM for HANA | M-series VMs are among the most expensive in Azure |
| Azure NetApp Files = low-latency NFS for SAP | ANF adds cost and complexity |
| Full HA/DR support with Azure AZs + ASR | Complex architecture; multiple components |
| Microsoft + SAP partnership = joint support | Requires ExpressRoute for production (latency-sensitive) |
| SAP Business Technology Platform integration | SAP sizing + licensing expertise required |

---

## 11. Cross-Cloud Migration (AWS / GCP → Azure)

### Azure Migrate — Multi-Cloud Support

| Source | Discovery | Assessment | Migration |
|--------|----------|-----------|---------|
| **AWS EC2** | ✅ Via appliance | ✅ Cost + readiness | ✅ Agent-based replication |
| **GCP Compute Engine** | ✅ Via appliance | ✅ Cost + readiness | ✅ Agent-based replication |
| **AWS RDS (SQL)** | ❌ (use DMS) | ❌ | ✅ Via DMS |
| **AWS S3** | ❌ | ❌ | ✅ AzCopy or Azure Data Factory |

### Common AWS → Azure Service Mappings

| AWS Service | Azure Equivalent | Notes |
|-------------|----------------|-------|
| EC2 | Azure Virtual Machines | |
| ECS / EKS | AKS / Container Apps | ECS → Container Apps; EKS → AKS |
| Lambda | Azure Functions | |
| Elastic Beanstalk | Azure App Service | |
| RDS (SQL) | Azure SQL Database | |
| DynamoDB | Azure Cosmos DB | |
| S3 | Azure Blob Storage | |
| CloudFront | Azure Front Door / Azure CDN | |
| VPC | Azure Virtual Network | |
| Route 53 | Azure DNS + Traffic Manager | |
| IAM | Entra ID + Azure RBAC | |
| CloudWatch | Azure Monitor | |
| Elastic Load Balancer (ALB) | Application Gateway | |
| Elastic Load Balancer (NLB) | Azure Load Balancer | |

---

## 12. Migration Tooling Comparison

### By Workload Type

| Workload | Primary Tool | Secondary / Alternative |
|---------|-------------|------------------------|
| VMware VMs | **Azure Migrate** (agentless) | ASR (agent-based) |
| Hyper-V VMs | **Azure Migrate** | ASR |
| Physical Servers | **Azure Migrate** (agent-based) | ASR |
| SQL Server | **DMS** (online) | Backup/Restore, Log Shipping |
| MySQL / PostgreSQL | **DMS** (online) | Native dump/restore |
| MongoDB | **DMS** | mongoexport / mongorestore |
| Web Apps (.NET/Java on IIS) | **App Service Migration Assistant** | Manual repackage |
| Large file datasets (> 40 TB) | **Azure Data Box** | AzCopy (if bandwidth allows) |
| Windows File Servers | **Storage Migration Service** | Robocopy (manual) |
| NAS devices | **Storage Migration Service** | Azure File Sync |
| VMware at scale (no refactor) | **Azure VMware Solution** | Azure Migrate (phased) |
| AWS VMs | **Azure Migrate** (agent-based) | — |

### By Migration Speed vs Cost

```
Speed (Fast → Slow):
  Live vMotion (AVS) → Online DMS → Azure Migrate cutover → Offline DMS → Data Box

Cost (Low → High):
  Azure Migrate (free tool) → DMS Free (offline) → DMS Premium → ASR → AVS (most expensive)

Effort (Low → High):
  BACPAC → Azure Migrate agentless → App Service Migration Assistant → DMS Online → AVS → Refactor
```

---

### Migration Risk Levels

| Tool / Method | Risk Level | Key Risk Factor |
|--------------|-----------|----------------|
| Rehost via Azure Migrate | Low | App may need IP/DNS changes |
| BACPAC (SQL) | Low | Offline; must test restore |
| DMS Online | Low-Medium | CDC setup; source firewall rules |
| App Service Migration Assistant | Medium | Compatibility issues (Windows Auth, COM) |
| ASR Migration | Medium | Agent-based; manual cutover |
| Data Box | Low | Physical shipping delay |
| AVS | Low | Expensive but low technical risk |
| Refactor/Containerize | High | Code changes, testing, new dependencies |

---

## 13. Exam Tips — Migration

### Azure Migrate Tips

| # | Tip | Trap to Avoid |
|---|-----|--------------|
| 1 | **Agentless replication is VMware ONLY** — all other sources need the Mobility Service agent | Assuming agentless works for Hyper-V or physical |
| 2 | Azure Migrate is **free** for discovery and assessment; network/compute costs apply during replication | Don't confuse tool cost with Azure resource cost |
| 3 | **Performance-based sizing** requires 2–4 weeks of data collection for accurate recommendations | "As on-premises" sizing = copy current config; may overprovision |
| 4 | Azure Migrate can discover **AWS EC2 and GCP VMs** via the same appliance | Not just on-prem — multi-cloud discovery |
| 5 | Max **500 VMs per appliance** — scale needs multiple appliances | Large environments need planning |
| 6 | **Test Migration** is a critical step — run before actual cutover to validate in isolated VNet | Skipping test migration = risk blind spot |

### DMS Tips

| # | Tip | Trap to Avoid |
|---|-----|--------------|
| 7 | **Online mode (CDC) requires Premium tier (paid)** — Free/Standard = offline only | "Minimal downtime migration" needs Premium DMS |
| 8 | **SQL MI requires DMS in same or peered VNet** — DMS cannot reach MI over public internet | Forgetting VNet requirement for SQL MI |
| 9 | CDC requires **CDC enabled on source SQL Server** — not automatic | Check CDC prerequisite early |
| 10 | **BACPAC is for small databases** — large DBs should use DMS or backup/restore | BACPAC doesn't scale well beyond a few GB |
| 11 | Oracle → Azure DB for PostgreSQL requires **schema conversion first** (ora2pg) — DMS doesn't convert schema | Oracle migration is a two-step process |
| 12 | **DMS migrates data only** — application connection strings must be updated post-migration | Don't forget connection string updates |

### Data Box Tips

| # | Tip | Trap to Avoid |
|---|-----|--------------|
| 13 | Use Data Box when transferring **> 40 TB** or internet bandwidth would take > 1–2 weeks | Using network when Data Box is clearly faster/cheaper |
| 14 | Data Box **cannot import directly** into Azure SQL, Cosmos DB — must stage in Blob Storage first | Trying to bypass Blob staging |
| 15 | Data Box **Data in transit AND at rest is encrypted** (AES 256-bit); device wiped after use (NIST 800-88) | Security-focused questions about Data Box |
| 16 | **Export is also supported** — ship Azure Blob data back on a device | Assuming Data Box is import-only |

### AVS & VMware Tips

| # | Tip | Trap to Avoid |
|---|-----|--------------|
| 17 | AVS requires **ExpressRoute** for connectivity to on-prem or Azure — no VPN-only option for production | Assuming VPN is sufficient for AVS |
| 18 | **Minimum 3 hosts** for AVS (vSAN requirement) — price per host is high | Proposing AVS for small VM counts |
| 19 | AVS is a **transition strategy**, not the end state — purpose is eventual modernization | Treating AVS as permanent cloud architecture |
| 20 | **HCX is included** with AVS and handles live vMotion across datacenters | Forgetting HCX enables zero-downtime live migration |

### Strategy & Design Tips

| # | Tip | Trap to Avoid |
|---|-----|--------------|
| 21 | "**Minimize migration time and effort**" → **Rehost** (Lift & Shift) via Azure Migrate | Don't propose refactor when speed matters most |
| 22 | "**Minimize long-term cost**" → **Replatform or Refactor** | Don't leave everything on IaaS when PaaS is cheaper long-term |
| 23 | "**Application needs SQL Agent, CLR, linked servers**" → **SQL Managed Instance** | SQL Database doesn't support SQL Agent, CLR, linked servers |
| 24 | "**Migrate 10,000 VMs in 3 months**" → **Azure Migrate** with phased waves + multiple appliances | Single migrations don't scale without wave planning |
| 25 | "**Migrate 500 TB of archival data**" → **Data Box Heavy** (not network, not Data Box standard) | Standard Data Box = 80 TB; Heavy = ~770 TB |
| 26 | "**Zero downtime for production SQL Server workload**" → **DMS Premium (Online)** | Offline DMS requires a maintenance window |
| 27 | "**App uses Windows Authentication**" → **Cannot go to App Service as-is** — needs redesign (OIDC, AD) | App Service Migration Assistant will flag this |
| 28 | "**Migrate NAS or Windows file server**" → **Azure Storage Migration Service** or **Azure File Sync** | Azure Migrate handles VMs, not file shares |
| 29 | "**No internet transfer, data sovereignty**" → **Data Box** | Network replication is not always acceptable |
| 30 | "**Migrate VMware without any changes, keep VMware tools**" → **Azure VMware Solution** | Azure Migrate would require guest OS management vs AVS keeps vCenter |

---

## Quick Reference Summary

```
MIGRATION TOOL BY WORKLOAD:
  VMs (VMware, agentless)    → Azure Migrate
  VMs (Hyper-V, Physical)    → Azure Migrate (agent-based)
  SQL Server (minimal downtime) → DMS Premium (Online/CDC)
  Web Apps (.NET on IIS)     → App Service Migration Assistant
  Files / NAS                → Storage Migration Service / Azure File Sync
  Bulk data (> 40 TB)        → Azure Data Box
  VMware at scale (no refactor) → Azure VMware Solution

MIGRATION MODES:
  Online (CDC) = minutes downtime, needs Premium DMS
  Offline = hours downtime, Free DMS tier

6 Rs (by effort):
  Retire (none) → Retain (none) → Rehost (low) → Replatform (med) → Repurchase (med) → Refactor (high)

KEY CONSTRAINTS TO WATCH:
  "SQL Agent / Linked Servers / CLR" → MUST use SQL Managed Instance (not SQL DB)
  "< X hours downtime" → Online DMS or Azure Site Recovery migration mode
  "No internet transfer" → Data Box
  "Keep VMware tools" → AVS
  "Minimize effort / fastest migration" → Rehost via Azure Migrate
```

---

*Last updated: May 20, 2026 | Good luck on May 21! 🎯*
