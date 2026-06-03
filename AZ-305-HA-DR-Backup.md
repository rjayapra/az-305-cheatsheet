# AZ-305 — High Availability, Disaster Recovery & Backup
## Complete Study Guide: HA Patterns, DR Strategies, Backup & Recovery
> Exam: May 21, 2026 | Relevant Domain: Business Continuity Solutions (15–20%)

---

## Table of Contents
1. [Core Concepts — RTO, RPO, SLA, MTTF, MTTR](#1-core-concepts--rto-rpo-sla-mttf-mttr)
2. [Availability Concepts — Fault Domains, Update Domains, Zones](#2-availability-concepts--fault-domains-update-domains-zones)
3. [VM Availability Options](#3-vm-availability-options)
4. [Application High Availability Patterns](#4-application-high-availability-patterns)
5. [Azure SQL Database — HA & DR](#5-azure-sql-database--ha--dr)
6. [Azure SQL Managed Instance — HA & DR](#6-azure-sql-managed-instance--ha--dr)
7. [Cosmos DB — HA & DR](#7-cosmos-db--ha--dr)
8. [Azure Storage — HA & DR](#8-azure-storage--ha--dr)
9. [App Service — HA & DR](#9-app-service--ha--dr)
10. [AKS — HA & DR](#10-aks--ha--dr)
11. [Azure Backup — Full Reference](#11-azure-backup--full-reference)
12. [Azure Site Recovery (ASR)](#12-azure-site-recovery-asr)
13. [Multi-Region Deployment Strategies](#13-multi-region-deployment-strategies)
14. [DR Strategy Tiers — Cold / Warm / Hot](#14-dr-strategy-tiers--cold--warm--hot)
15. [Well-Architected Framework — Reliability Pillar](#15-well-architected-framework--reliability-pillar)
16. [SLA Calculations](#16-sla-calculations)
17. [Backup & DR — Service Comparison Matrix](#17-backup--dr--service-comparison-matrix)
18. [Exam Tips — HA / DR / Backup](#18-exam-tips--ha--dr--backup)

---

## 1. Core Concepts — RTO, RPO, SLA, MTTF, MTTR

### Definitions

| Term | Full Name | Definition | Exam Focus |
|------|-----------|------------|-----------|
| **RTO** | Recovery Time Objective | Maximum acceptable downtime after a failure | How FAST must you recover? |
| **RPO** | Recovery Point Objective | Maximum acceptable data loss (measured in time) | How much DATA can you afford to lose? |
| **SLA** | Service Level Agreement | Committed uptime % from Microsoft | Used in SLA calculations |
| **MTTF** | Mean Time To Failure | Average time a system runs before failing | Reliability design metric |
| **MTTR** | Mean Time To Recover | Average time to restore after a failure | Recovery efficiency |
| **Availability** | Uptime % | `Availability = MTTF / (MTTF + MTTR)` | Design target |
| **Downtime** | Allowed outage | `Downtime = (1 - SLA%) × time period` | Business impact |

### RTO vs RPO — Design Impact

```
           Past ◀────────────────────────────────▶ Future
           ──────────────────────────────────────────────
                     |                |
               Last backup          Failure           Recovery
               point                point             point
                     |◀──── RPO ────▶|                |
                                     |◀──── RTO ──────▶|

RPO = How much data loss is acceptable (time from last backup to failure)
RTO = How long can the system be offline (time from failure to recovery)
```

### Annual Downtime by SLA

| SLA % | Annual Downtime | Monthly Downtime |
|-------|----------------|-----------------|
| 99% | 87.6 hours | 7.3 hours |
| 99.5% | 43.8 hours | 3.65 hours |
| 99.9% | 8.76 hours | 43.8 minutes |
| 99.95% | 4.38 hours | 21.9 minutes |
| 99.99% | 52.6 minutes | 4.38 minutes |
| 99.999% | 5.26 minutes | 26.3 seconds |

> **Exam tip:** Know that improving SLA from 99.9% to 99.99% reduces allowed annual downtime from 8+ hours to < 1 hour — big architectural investment for incremental improvement

---

## 2. Availability Concepts — Fault Domains, Update Domains, Zones

### Fault Domains

| Concept | Definition | Scope |
|---------|-----------|-------|
| **Fault Domain (FD)** | Group of hardware sharing a common power source and network switch | Protects against rack-level hardware failure |
| **Number of FDs** | 2–3 per Availability Set (depends on region) | — |
| **VM distribution** | Azure spreads VMs across FDs automatically in Availability Sets | Rack diversity |

### Update Domains

| Concept | Definition | Scope |
|---------|-----------|-------|
| **Update Domain (UD)** | Logical group of VMs that are updated together during planned maintenance | Protects against planned maintenance outage |
| **Default UDs** | 5 (configurable up to 20) | One UD reboots at a time |
| **Restart time between UDs** | 30 minutes default | Sequential patching |

### Availability Zones

| Concept | Definition | Scope |
|---------|-----------|-------|
| **Availability Zone** | Physically separate datacenter within a region | Independent power, cooling, networking |
| **Zones per region** | Minimum 3 in supported regions | Not all regions support AZs |
| **Distance** | Miles apart within same metro area | Low-latency inter-zone |
| **Latency** | < 2 ms round-trip between zones | Acceptable for synchronous replication |

### Availability Options Comparison

| Option | SLA | Protects Against | FDs | UDs | Cost Premium |
|--------|-----|-----------------|-----|-----|-------------|
| **No redundancy (single VM)** | 99.9% (Premium SSD) | Nothing | 1 | 1 | None |
| **Availability Set (2+ VMs)** | 99.95% | Rack failure, planned maintenance | 2–3 | 5 (default) | None (VM cost only) |
| **Availability Zones (2+ VMs)** | 99.99% | Datacenter failure | Independent | Independent | Zone-redundant IP/disk cost |
| **Multi-region (2+ regions)** | Custom (99.99%+) | Regional failure | — | — | Significant (duplicate infra) |

> **Exam tip:** Availability Set = same datacenter, different racks. Availability Zone = different datacenters. You cannot use both simultaneously for the same VMs.

---

## 3. VM Availability Options

### Availability Set Design Rules

| Rule | Detail |
|------|--------|
| **Max VMs per Availability Set** | 200 |
| **Max FDs** | 2 or 3 depending on region |
| **Max UDs** | 20 |
| **Must be in same region** | Yes |
| **Must be in same resource group** | Yes |
| **Managed disks** | Required for premium reliability |
| **Load balancer required** | For traffic distribution across VMs |
| **When to use** | Protects against hardware failures AND patching downtime |

### Availability Zone Design Rules

| Rule | Detail |
|------|--------|
| **Minimum zones** | 2 zones for 99.99% SLA |
| **VM placement** | Explicitly place VM in Zone 1, 2, or 3 |
| **Zone-pinned** | VM is fixed to the zone; cannot auto-move |
| **Regional capacity** | Different VM SKUs available per zone |
| **Zonal vs Zone-redundant services** | Zonal = pinned to one zone; Zone-redundant = spans all zones |
| **Use with LB** | Zone-redundant Standard LB for across-zone traffic |

### VMSS (Virtual Machine Scale Sets) HA Options

| Feature | Flexible Orchestration | Uniform Orchestration |
|---------|----------------------|----------------------|
| **SLA** | 99.99% (with AZs) | 99.95% / 99.99% with AZs |
| **Availability Zones** | ✅ Zone spread | ✅ Zone spread (fixed) |
| **Max instances** | 1,000 | 1,000 |
| **Mixed VM types** | ✅ | ❌ (same SKU only) |
| **Spot VMs** | ✅ | ✅ |
| **Recommended** | ✅ (default for new) | Legacy |

---

## 4. Application High Availability Patterns

### Health Endpoint Monitoring Pattern

```
Load Balancer → Health probe → /health endpoint on each VM
  ├── Returns HTTP 200 = Healthy → keeps in rotation
  └── Returns non-200 / timeout = Unhealthy → removed from rotation
```

### Circuit Breaker Pattern

| State | Behavior | Trigger |
|-------|---------|---------|
| **Closed** (normal) | Requests pass through | Success rate high |
| **Open** (failing) | Requests blocked; fast fail | Failure rate exceeds threshold |
| **Half-Open** (testing) | Limited requests allowed to test recovery | After timeout period |

> Used to prevent cascading failures when downstream service is degraded

### Retry Pattern

| Option | Description |
|--------|-------------|
| **Immediate retry** | Retry once immediately (transient faults) |
| **Fixed delay** | Wait fixed interval between retries |
| **Exponential backoff** | Increasing delays (1s, 2s, 4s, 8s...) |
| **Exponential backoff + jitter** | Add randomness to prevent thundering herd |

> Azure SDKs (SQL, Storage, Cosmos DB) implement retry logic by default

### Queue-Based Load Leveling

```
Producers → Queue (Azure Service Bus / Storage Queue) → Consumers

Benefits:
  - Producers not blocked if consumers slow
  - Consumers process at their own pace
  - Queue acts as buffer; absorbs traffic spikes
  - Dead-letter queue for failed messages
```

### Competing Consumers Pattern

```
Multiple consumer instances read from same queue
  → Horizontal scale of processing
  → If one consumer fails, others continue
  → Natural HA for async processing
```

---

## 5. Azure SQL Database — HA & DR

### Built-in HA (Per Tier)

| Tier | HA Mechanism | Read Replicas | Zone Redundancy | SLA |
|------|-------------|--------------|----------------|-----|
| **General Purpose** | Remote storage failover (99.99%+ storage HA); compute cold failover | Optional (read scale-out; extra cost) | Optional (add-on) | 99.99% |
| **Business Critical** | Always On AG internally; local SSD | ✅ 1 built-in, free (same region) | Built-in (spans AZs) | 99.99% |
| **Hyperscale** | Multiple replicas; distributed storage | Named replicas (up to 5; chargeable) | ✅ | 99.99% |
| **Serverless (GP)** | Same as General Purpose | ❌ | Optional | 99.99% |

### Geo-Replication Options

#### Active Geo-Replication

| Feature | Detail |
|---------|--------|
| **Purpose** | Readable secondary databases in up to 4 different regions |
| **Replication** | Asynchronous (RPO typically < 5 seconds) |
| **Availability** | SQL Database only (NOT SQL Managed Instance) |
| **Failover** | Manual failover required (no auto-failover) |
| **Read access** | ✅ Secondary is readable (offload read workloads) |
| **Secondaries per primary** | Up to 4 secondary databases |
| **Use case** | Read scale-out globally; manual DR failover |

#### Failover Groups (Auto-Failover)

| Feature | Detail |
|---------|--------|
| **Scope** | SQL Database AND SQL Managed Instance |
| **Replication** | Async geo-replication under the hood |
| **Failover** | ✅ Automatic (based on policy) OR manual |
| **Connection strings** | Single read-write listener + single read-only listener; no app update on failover |
| **RPO** | Typically < 5 seconds (async) |
| **RTO** | < 30 seconds (automatic failover) |
| **Failover policy** | Automatic (configurable grace period) or manual |
| **Multiple DBs** | One failover group can contain multiple databases |

> **Exam tip:** Failover Groups = single connection string; app doesn't need reconfiguring after failover. Active Geo-Replication = manual failover, connection strings change.

### SQL Database Backup

| Backup Type | Frequency | Stored In | Retention |
|-------------|---------|-----------|---------|
| **Full backup** | Weekly | Geo-redundant storage | Per retention policy |
| **Differential backup** | Every 12–24 hours | Geo-redundant storage | Per retention policy |
| **Transaction log backup** | Every 5–10 minutes | Geo-redundant storage | Per retention policy |
| **Standard retention** | 1–35 days (configurable) | Automated | Point-in-time restore |
| **Long-Term Retention (LTR)** | Weekly / Monthly / Yearly copies | Azure Blob (RA-GRS) | Up to 10 years |

### SQL Database Point-in-Time Restore (PITR)

| Feature | Detail |
|---------|--------|
| **Granularity** | Restore to any point within retention window (to the minute) |
| **Target** | New database (not in-place overwrite) |
| **Restore time** | Depends on DB size (minutes to hours) |
| **Geo-restore** | Restore from geo-redundant backup to ANY Azure region |
| **Use case** | Accidental delete, data corruption, ransomware recovery |

---

## 6. Azure SQL Managed Instance — HA & DR

### Built-in HA

| Tier | HA Mechanism | Built-in Read Replica | Zone Redundancy |
|------|-------------|----------------------|----------------|
| **General Purpose** | Remote storage (same as SQL DB GP) | ❌ | ✅ Optional |
| **Business Critical** | Always On AG (3 nodes); local SSD | ✅ 1 free replica | ✅ Optional |

### Geo-DR for SQL MI

| Feature | Detail |
|---------|--------|
| **Failover Groups** | ✅ Supported (same as SQL DB failover groups) |
| **Active Geo-Replication** | ❌ NOT supported for SQL MI (only for SQL DB) |
| **RPO** | < 5 seconds (async replication) |
| **RTO** | < 30 seconds (automatic failover) |
| **Listener endpoint** | Single connection string (read-write + read-only listeners) |

### SQL MI Backup

| Feature | Detail |
|---------|--------|
| **Automated backups** | Full (weekly), Differential (every 12–24 hrs), Log (every 5–10 min) |
| **Retention** | 1–35 days |
| **LTR** | ✅ Up to 10 years |
| **Geo-restore** | ✅ From geo-redundant backup storage |
| **PITR** | ✅ To the minute within retention period |

---

## 7. Cosmos DB — HA & DR

### Replication & Availability

| Configuration | SLA | RPO | RTO | Cost |
|--------------|-----|-----|-----|------|
| **Single region** | 99.99% | Near-zero (within region) | Seconds | Baseline |
| **Multi-region, single write** | 99.999% read | < 15 min (async) | Minutes (failover) | Medium |
| **Multi-region, multi-write** | 99.999% R+W | ~0 (sync to multiple) | Near-zero | High |
| **Zone redundancy (per region)** | Higher within region | — | — | +25% premium |

### Failover Types

| Type | Trigger | Data Loss Risk | Notes |
|------|---------|---------------|-------|
| **Automatic failover** | Region health (Azure-triggered) | RPO of async replication | Single-write mode only |
| **Manual failover** | Customer-initiated (testing/maintenance) | Potential data loss acknowledged | For planned failovers |
| **Multi-write** | No failover needed (always active-active) | Conflict resolution required | No single point of failure |

### Cosmos DB Backup

| Mode | RPO | Restore Type | Retention | Cost |
|------|-----|------------|---------|------|
| **Periodic (default)** | Up to 4 hours | Microsoft support ticket | 2 copies, 30 days | Included |
| **Continuous 7-day** | Up to 1 hour | Self-service PITR (portal/CLI) | 7 days | Additional charge |
| **Continuous 30-day** | Up to 1 hour | Self-service PITR | 30 days | Higher charge |

> **Self-service PITR** only available with Continuous backup mode
> Periodic backup restore = submit support ticket; takes longer

---

## 8. Azure Storage — HA & DR

### Redundancy & HA

| Redundancy | Primary Region Copies | Zone Protected | Geo Protected | Secondary Readable | SLA |
|-----------|----------------------|--------------|-------------|------------------|-----|
| **LRS** | 3 (same datacenter) | ❌ | ❌ | ❌ | 99.9% |
| **ZRS** | 3 (3 zones) | ✅ | ❌ | ❌ | 99.9% |
| **GRS** | 3 primary + 3 secondary | ❌ | ✅ | ❌ | 99.9% |
| **GZRS** | 3 zones primary + 3 secondary | ✅ | ✅ | ❌ | 99.9% |
| **RA-GRS** | 3 primary + 3 secondary | ❌ | ✅ | ✅ (read-only) | **99.99%** |
| **RA-GZRS** | 3 zones primary + 3 secondary | ✅ | ✅ | ✅ (read-only) | **99.99%** |

### Storage Failover

| Feature | Detail |
|---------|--------|
| **Manual failover** | Customer can trigger failover to secondary region (GRS/RA-GRS) |
| **Microsoft-initiated failover** | In case of full regional disaster |
| **After failover** | Secondary becomes new primary; single-region (LRS) until re-setup |
| **RPO** | Typically < 15 minutes (async replication lag) |
| **Data loss risk** | Any unsynced writes in replication lag window |
| **DNS update** | Storage endpoints automatically point to new primary post-failover |
| **Re-enable geo-redundancy** | Must manually re-enable GRS after failover |

### Blob Soft Delete & Versioning

| Feature | Purpose | Scope | Retention |
|---------|---------|-------|---------|
| **Blob Soft Delete** | Recover deleted blobs | Blob level | 1–365 days |
| **Container Soft Delete** | Recover deleted containers | Container level | 1–365 days |
| **Blob Versioning** | Maintain previous versions on overwrite | Per blob | Configurable |
| **Point-in-Time Restore** | Restore blocks to earlier state | Block blobs only | 1–365 days |
| **Change Feed** | Audit log of blob changes | Account level | Configurable |

> Enable all four — they work together for comprehensive blob data protection

---

## 9. App Service — HA & DR

### HA Patterns

| Feature | Tier Required | Description |
|---------|-------------|-------------|
| **Deployment Slots** | Standard+ | Blue/green deployments; swap with zero downtime |
| **Auto-scale** | Standard+ | Scale out instances based on metrics |
| **Availability Zones** | Premium v3+ | Spread instances across 3 AZs; minimum 3 instances |
| **Always On** | Basic+ | Keep app warm (no cold starts) |
| **Traffic Manager** | Any | Route across multiple App Services globally |
| **Azure Front Door** | Any | Global HTTP load balancing + WAF + CDN |

### App Service Zone Redundancy

| Requirement | Detail |
|-------------|--------|
| **Plan tier** | Premium v3 or Isolated v2 |
| **Minimum instances** | 3 (one per zone) |
| **Region** | Must be a Zone-enabled region |
| **SLA with zones** | 99.99% |
| **Cost** | Minimum 3 instances charged |

### Multi-Region App Service

```
Azure Front Door / Traffic Manager
  ├── Region A: App Service (Primary)
  │     └── Slots: Production + Staging
  └── Region B: App Service (Secondary / DR)
        └── Warm standby or cold standby
        
Database: Failover Group (SQL DB / SQL MI)
  → Same listener endpoint for apps in both regions
```

### Deployment Slots — Key Features

| Feature | Detail |
|---------|--------|
| **Swap** | Exchange production and staging slot; zero-downtime |
| **Auto-swap** | Auto-swap when code pushed to staging | 
| **Slot settings** | Sticky settings that don't swap (connection strings, app settings) |
| **Traffic splitting** | Route % of users to staging (A/B testing) |
| **Validation** | Pre-swap validation sequence to catch issues |

---

## 10. AKS — HA & DR

### Cluster-Level HA

| Feature | SLA Impact | Detail |
|---------|-----------|--------|
| **Standard/Premium tier** | 99.9%–99.95% | SLA on control plane API server |
| **Availability Zones** | 99.95% | Spread node pools across 3 AZs |
| **Multi-node pools** | Higher reliability | Spread workloads; isolate system from user pools |
| **Node auto-repair** | Reduces MTTR | Auto-detects and replaces unhealthy nodes |
| **Cluster autoscaler** | Prevents resource starvation | Adds nodes when pods pending |

### AKS Zone Redundancy

| Requirement | Detail |
|-------------|--------|
| **Region** | Zone-enabled region |
| **Node pool** | Configure with `--zones 1 2 3` |
| **Load Balancer** | Standard SKU (zone-redundant) |
| **Storage** | ZRS managed disks for zone-redundant PVCs |
| **Pod disruption budget** | Define min/maxUnavailable for graceful disruption |

### AKS Multi-Region DR

```
Primary Region:                    DR Region:
  AKS Cluster                        AKS Cluster (same config)
  ├── App deployments                ├── App deployments
  └── Persistent volumes             └── Persistent volumes

Shared:
  Azure Container Registry (geo-replicated)
  Database with Failover Group
  Azure Front Door / Traffic Manager (route to healthy cluster)
  GitOps repo (Flux/ArgoCD) for config consistency
```

### AKS Backup

| Tool | What It Backs Up | Method |
|------|----------------|--------|
| **Azure Backup for AKS** | Cluster config, workloads, persistent volumes | Backup extension in AKS |
| **Velero** | K8s resources + PVC data | Open-source; stores in Azure Blob |
| **ACR replication** | Container images | Geo-replicate registry to DR region |

---

## 11. Azure Backup — Full Reference

### Vault Types

| Vault | Supports | Redundancy Options | Notes |
|-------|---------|------------------|-------|
| **Recovery Services Vault (RSV)** | Azure VMs, SQL in VMs, SAP HANA in VMs, Azure Files, on-prem (MARS/DPM/MABS) | LRS, ZRS, GRS | Traditional; most workloads |
| **Backup Vault** | Azure Disks, Azure Blobs (operational), Azure DB for PostgreSQL, AKS | LRS, GRS, ZRS | Newer workloads |

### Backup Vault Redundancy

| Redundancy | Copies | Zone | Geo | Secondary Readable | Cost |
|-----------|--------|------|-----|-------------------|------|
| **LRS** | 3 | ❌ | ❌ | ❌ | Lowest |
| **ZRS** | 3 | ✅ | ❌ | ❌ | Medium |
| **GRS** (default) | 6 | ❌ | ✅ | ❌ | Higher |

> Change from GRS → LRS possible but NOT from LRS → GRS after backup items are registered
> **Cross-Region Restore (CRR):** Requires GRS vault; restores to secondary region from geo-replicated backup

### Workloads, RPO, RTO & Retention

| Workload | Backup Tool | Min RPO | Max Retention | Backup Type | Vault Type |
|----------|------------|---------|-------------|------------|-----------|
| **Azure VM** | Azure Backup (agentless) | 4 hours (daily) | **9,999 days** | App-consistent snapshot | RSV |
| **SQL Server in Azure VM** | Azure Backup (workload-aware) | **15 minutes** (log backup) | 99 years | Full + diff + log | RSV |
| **SAP HANA in Azure VM** | Azure Backup (Backint) | 15 minutes | 99 years | Full + diff + log | RSV |
| **Azure Files** | Azure Backup | 4 hours | 1 year (200 restore points) | Share snapshot | RSV |
| **On-prem machines (MARS agent)** | Microsoft Azure Recovery Services agent | 3 times/ day | 99 years | File + folder level | RSV |
| **On-prem workloads (DPM/MABS)** | DPM/MABS agent | 15 min (disk) | Long-term | App-consistent | RSV |
| **Azure Disks** | Azure Backup (disk) | 1 hour (up to 24 snapshots/day) | **810 days** | Crash-consistent snapshot | Backup Vault |
| **Azure Blobs (operational)** | Azure Backup (operational) | **Continuous** (near-zero) | **360 days** | Storage-level | Backup Vault |
| **Azure Blobs (vaulted)** | Azure Backup (vaulted) | Daily | **10 years** | Vaulted daily backup | Backup Vault |
| **Azure DB for PostgreSQL** | Azure Backup | Daily | 35 days (or up to 10 years via LTR) | Full backup | Backup Vault |
| **AKS** | Azure Backup (AKS extension) | 4 hours | 360 days | Cluster + PV snapshot | Backup Vault |

### Azure VM Backup — Key Details

| Feature | Detail |
|---------|--------|
| **Snapshot mechanism** | Application-consistent via VSS (Windows) / pre/post scripts (Linux) |
| **Initial backup** | Full copy; subsequent are incremental |
| **Backup frequency** | Daily (standard) or multiple times/day (Enhanced Policy) |
| **Enhanced Policy** | Up to 6 backups/day for VMs + Trusted Launch support |
| **Instant restore** | Restore from local snapshot (faster, ~minutes) before it transfers to vault |
| **Instant restore retention** | 1–5 days |
| **Restore options** | Full VM restore, Disk restore, File/Folder recovery, Replace existing disk |
| **Cross-region restore** | ✅ With GRS vault (restore VM from secondary region backup) |
| **Cross-subscription restore** | ✅ |

### MARS Agent (On-Premises)

| Feature | Detail |
|---------|--------|
| **Protects** | Files, folders, system state on Windows servers/laptops |
| **Frequency** | Up to 3 backups/day |
| **Retention** | Up to 99 years |
| **Encryption** | Passphrase-encrypted (customer holds key) |
| **Bandwidth throttling** | Configure to limit network impact |
| **Offline seeding** | Import initial backup via Data Box (large initial copies) |

### Backup Policies

| Policy Component | Options |
|-----------------|---------|
| **Backup frequency** | Daily, weekly, multiple times/day |
| **Retention — Daily** | 7–9,999 days |
| **Retention — Weekly** | Up to 99 weeks |
| **Retention — Monthly** | Up to 99 months |
| **Retention — Yearly** | Up to 99 years |
| **Instant restore tier** | 1–5 days (faster restore from local snapshot) |
| **Backup tier** | Standard or Archive (for long-term old recovery points) |

### Backup Archive Tier

| Feature | Detail |
|---------|--------|
| **Purpose** | Move older recovery points to cheaper archive storage |
| **Eligible workloads** | Azure VMs, SQL in VMs |
| **Minimum archive duration** | 180 days in archive tier |
| **Access for restore** | Rehydrate from archive (12–24 hours) |
| **Cost saving** | ~75% cheaper than vault standard tier |
| **Recovery time** | Longer (rehydration required before restore) |

---

## 12. Azure Site Recovery (ASR)

### Purpose
**Disaster Recovery as a Service (DRaaS)** — Replicates workloads to a secondary location (Azure region or on-premises) so they can be resumed there during a disaster.

### Supported Scenarios

| Source | Target | RPO | Notes |
|--------|--------|-----|-------|
| **Azure VM → Azure VM** (another region) | Azure | **30 seconds** | Most common; agentless |
| **VMware VMs → Azure** | Azure | 30 seconds | Mobility Service required |
| **Physical servers → Azure** | Azure | 1 hour | Agent-based |
| **Hyper-V VMs → Azure** | Azure | 30 seconds | Provider + agent |
| **Hyper-V VMs (VMM) → Azure** | Azure | 30 seconds | VMM integration |
| **Azure Stack Hub VMs → Azure** | Azure | 30 seconds | Supported |

### ASR Components

| Component | Role |
|-----------|------|
| **Recovery Services Vault** | Central orchestration and configuration store |
| **Replication Policy** | RPO threshold, recovery point retention, app-consistent snapshot frequency |
| **Mobility Service** | Agent installed on VMs (VMware/physical) |
| **Process Server** | Receives, caches, compresses, encrypts, sends replication data (VMware) |
| **Configuration Server** | Manages replication for VMware/physical (on-prem component) |
| **Recovery Plan** | Ordered failover of multiple VMs with custom scripts and manual actions |

### Recovery Points

| Type | Frequency | Data Captured | Use Case |
|------|---------|--------------|---------|
| **Crash-consistent** | Every 5 minutes | OS + disk state at moment of capture | Non-application-aware (like sudden power off) |
| **App-consistent** | Every 1–12 hours (configurable) | VSS snapshot; applications flushed | Database servers, exchange, etc. |

### ASR Operations

| Operation | Description | Impact | When Used |
|-----------|-------------|--------|----------|
| **Enable Replication** | Start replicating VM to target region | Initial sync + ongoing delta | Setup |
| **Test Failover** | Spin up VM in isolated VNet using recovery point | ✅ Non-disruptive; production unaffected | DR drills |
| **Planned Failover** | Graceful migration; shuts down source VM first | Zero data loss | Planned migrations |
| **Unplanned Failover** | Emergency; uses last recovery point | Potential data loss (within RPO) | Actual disaster |
| **Re-protect** | Reverse replication direction after failover | Enables failback | Post-failover |
| **Failback** | Return to original region | Requires re-protect first | Recovery after disaster |
| **Change Recovery Point** | Select which recovery point to use before failover | Affects RPO | Pre-failover validation |

### Recovery Plan

| Feature | Detail |
|---------|--------|
| **Purpose** | Orchestrate failover of a multi-VM, multi-tier application in the correct order |
| **Groups** | Group 1 → Group 2 → Group 3 (sequential); within a group = parallel |
| **Scripts** | Run Azure Automation Runbooks before/after each group |
| **Manual actions** | Pause failover for manual steps (update DNS, notify team) |
| **Test failover** | Test entire recovery plan in isolated VNet |
| **Max VMs per plan** | 100 |

#### Example Recovery Plan

```
Group 1 (Boot first):
  └── Domain Controllers, DNS servers

Group 2 (After Group 1 healthy):
  └── Database servers (SQL, Cosmos)
  [Script: Run DB startup validation]

Group 3 (After Group 2 healthy):
  └── Application servers, API layer
  [Script: Update DNS + load balancer]

Group 4 (After Group 3 healthy):
  └── Web front-end servers
```

### ASR Networking in Azure-to-Azure

| Component | DR Region Requirement |
|-----------|----------------------|
| **Target VNet** | Pre-created or auto-created by ASR |
| **Target subnet mapping** | Map source subnet → target subnet |
| **Target NSGs** | Pre-configure or use ASR defaults |
| **Public IP** | Assign new public IP in target region |
| **Load Balancer** | Create matching LB in target region |
| **Retained IPs** | Possible if target VNet has same address space |

### ASR Costs

| Component | Cost |
|-----------|------|
| **ASR license** | Per protected instance/month |
| **Storage (cache)** | Standard storage in source region |
| **Storage (recovery)** | Managed disks in target region (same disk type as source) |
| **Outbound data** | Data transfer from source to target region |
| **Compute** | Only charged when failover occurs (test or actual) |

> **Cost saving:** Source region VMs don't run in target during protection — only storage charged

---

## 13. Multi-Region Deployment Strategies

### Active-Active

```
Region A                           Region B
┌─────────────────────┐           ┌─────────────────────┐
│  App Tier (running) │           │  App Tier (running) │
│  DB (write capable) │◀─────────▶│  DB (write capable) │
└─────────────────────┘           └─────────────────────┘
              │                                 │
              └──────────────┬──────────────────┘
                             │
              Azure Front Door / Traffic Manager
              (distributes traffic to both regions)
```

| Attribute | Value |
|-----------|-------|
| **Availability** | Highest (both regions live) |
| **RPO** | Near-zero (synchronous or near-synchronous replication) |
| **RTO** | Near-zero (traffic re-routed automatically) |
| **Cost** | Highest (full infrastructure in both regions) |
| **Complexity** | Highest (distributed state, conflict resolution, global DNS) |
| **DB pattern** | Cosmos DB multi-write OR SQL Database Failover Group (async) |

### Active-Passive (Warm Standby)

```
Region A (Active)                  Region B (Passive - warm)
┌─────────────────────┐           ┌─────────────────────┐
│  App Tier (running) │           │  App Tier (running, │
│  DB (primary write) │──────────▶│  DB standby) no lvb│
└─────────────────────┘           └─────────────────────┘
```

| Attribute | Value |
|-----------|-------|
| **RPO** | Seconds to minutes (async replication) |
| **RTO** | Minutes (failover + DNS change) |
| **Cost** | Medium-High (DR region resources run but no traffic) |
| **Failover trigger** | Traffic Manager / Front Door health probe |

### Active-Passive (Cold Standby / Pilot Light)

```
Region A (Active)                  Region B (Cold)
┌─────────────────────┐           ┌─────────────────────┐
│  App Tier (running) │           │  App Tier (stopped) │
│  DB (primary write) │──────────▶│  DB replica         │
└─────────────────────┘           └─────────────────────┘
```

| Attribute | Value |
|-----------|-------|
| **RPO** | Minutes |
| **RTO** | 30 min – few hours (must start compute) |
| **Cost** | Lowest (compute stopped; only storage + DB replica charged) |
| **Start time** | VM startup + app warm-up required |

### Pilot Light Pattern

| Component | DR Region State |
|-----------|----------------|
| **Database** | Running (continuous replication; smallest SKU) |
| **Core services** | Minimal instances running |
| **Compute (App/Web)** | Stopped or scale = 0 |
| **DNS/LB** | Pre-configured; activate on failover |
| **Activation time** | Scale up compute + update DNS (~30–60 min) |

---

## 14. DR Strategy Tiers — Cold / Warm / Hot

| Tier | Strategy | Typical RTO | Typical RPO | Cost Factor | Azure Pattern |
|------|---------|-------------|-------------|------------|--------------|
| **Tier 0 (Hot)** | Active-Active | < 1 minute | Near-zero | Highest (2×) | Front Door + Cosmos multi-write |
| **Tier 1 (Warm)** | Active-Passive (Hot standby) | 1–30 minutes | Seconds–minutes | 1.5× | Traffic Manager + Failover Group |
| **Tier 2 (Warm)** | Pilot Light | 30–60 minutes | Minutes | 1.1–1.3× | ASR + DB replica + scale-out |
| **Tier 3 (Cold)** | Backup-Restore | Hours–days | Hours | Lowest | Azure Backup + Geo-restore |

### Choosing DR Tier

| Business Requirement | Recommended Tier |
|---------------------|-----------------|
| "Zero tolerance for downtime, zero data loss" | Tier 0 — Active-Active |
| "< 15 min RTO, < 5 min RPO" | Tier 1 — Warm standby |
| "< 1 hour RTO, < 15 min RPO, cost-sensitive" | Tier 2 — Pilot light |
| "< 4 hour RTO, < 1 hour RPO, dev/test-like" | Tier 3 — Backup-restore |

---

## 15. Well-Architected Framework — Reliability Pillar

### 5 Key Areas of Reliability (WAF)

| Area | Description | Azure Implementation |
|------|-------------|---------------------|
| **Design for failure** | Assume components WILL fail; design for graceful degradation | Circuit breaker, retry, bulkhead patterns |
| **Redundancy** | Eliminate single points of failure | Availability Zones, Availability Sets, multi-region |
| **Recovery** | Plan and practice recovery | ASR test failover, backup restore drills, runbooks |
| **Monitoring** | Detect failures quickly | Azure Monitor, Application Insights, Service Health alerts |
| **Testing** | Continuously test reliability | Chaos engineering, test failovers, load testing |

### Reliability Design Checklist

| Category | Checklist Item |
|----------|---------------|
| **Compute** | ☐ VMs in Availability Zones OR Availability Sets<br>☐ VMSS with auto-scale<br>☐ Health probes configured correctly |
| **Data** | ☐ Geo-redundant storage (GRS/RA-GRS)<br>☐ Database with Failover Group or Active Geo-Replication<br>☐ Automated backups with tested restore |
| **Networking** | ☐ Redundant network paths (ExpressRoute + VPN)<br>☐ Zone-redundant load balancers<br>☐ Traffic Manager / Front Door health probes |
| **Application** | ☐ Retry logic in all external calls<br>☐ Circuit breakers for downstream dependencies<br>☐ Health check endpoints |
| **Operations** | ☐ DR documentation and runbooks<br>☐ Regular DR drills (ASR test failover)<br>☐ Backup restore tested |
| **Monitoring** | ☐ Alerts on key metrics + log patterns<br>☐ Service Health alerts configured<br>☐ Resource Health monitoring |

---

## 16. SLA Calculations

### Composite SLA (Services in Series — ALL must work)

```
Formula: Combined SLA = SLA₁ × SLA₂ × SLA₃ ...

Example: 3-tier web app
  App Service (99.95%) × SQL Database (99.99%) × Redis Cache (99.9%)
  = 0.9995 × 0.9999 × 0.999
  = 0.9984 = 99.84%
```

> Adding more dependencies REDUCES composite SLA — every tier weakens overall availability

### Parallel SLA (Redundant Components — ANY can work)

```
Formula: Combined = 1 - (1 - SLA₁) × (1 - SLA₂)

Example: Two App Services behind Traffic Manager
  TM SLA = 99.99%
  App Service A = 99.95% | App Service B = 99.95%
  
  Parallel App Service SLA = 1 - (0.0005 × 0.0005) = 99.99975%
  Combined with TM = 99.99% × 99.99975% ≈ 99.9897%
  
  vs Single App Service = 99.95% → Significant improvement!
```

### Common SLA Calculation Scenarios

| Scenario | Calculation | Result |
|----------|-------------|--------|
| App Service + SQL DB (series) | 99.95% × 99.99% | 99.94% |
| App Service HA (2 regions, parallel) | 1-(0.0005)² | 99.99975% |
| 2 VMs in Availability Set | Via AS SLA guarantee | 99.95% |
| 2 VMs across AZs | Via AZ SLA guarantee | 99.99% |
| VM + Managed Disk (Premium SSD) | 99.9% (VM SLA itself) | 99.9% |

### SLA Improvement Strategies

| Problem | Solution | SLA Improvement |
|---------|----------|----------------|
| Single VM (99.9%) | Add to Availability Zone | → 99.99% |
| Two regional App Services | Add Traffic Manager (parallel) | Significantly better |
| SQL DB (single region, GP) | Add Failover Group to secondary | Composite improves |
| Dependent on on-prem via VPN | Add ExpressRoute | 99.9% → 99.95% |
| App dependent on 5 services | Decouple async (queues) | Break serial chain |

---

## 17. Backup & DR — Service Comparison Matrix

### By Recovery Scenario

| Scenario | Azure Feature | RPO | RTO | Service |
|----------|-------------|-----|-----|---------|
| Accidentally deleted a blob | Blob Soft Delete / Versioning | Near-zero | Minutes | Azure Blob Storage |
| Accidentally deleted a VM | Azure Backup (restore VM) | Per policy (daily = up to 24h) | Minutes–hours | Azure Backup |
| Corrupted SQL Database data | SQL PITR | 5–10 min (log backup freq) | Minutes | Azure SQL (auto) |
| Regional Azure outage — SQL | SQL Failover Group | < 5 seconds | < 30 seconds | Azure SQL / ASR |
| Regional Azure outage — VMs | Azure Site Recovery | 30 seconds | < 2 hours | ASR |
| Regional Azure outage — Storage | Storage failover (GRS) | < 15 min | Minutes (DNS update) | Azure Storage |
| Ransomware — encrypted files | Azure Backup (clean restore point) | Per backup frequency | Hours | Azure Backup |
| Ransomware — Cosmos DB | Cosmos DB Continuous backup PITR | 1 hour | Minutes (self-service) | Cosmos DB Backup |
| Compliance archive (10 years) | Azure Backup LTR / Archive tier | Per policy | Hours (rehydrate) | Azure Backup |
| DR drill (non-disruptive) | ASR Test Failover | — | — | ASR |
| Whole region migration | Azure Migrate | N/A | N/A | Azure Migrate |

### Backup Coverage by Service

| Azure Service | Built-in Backup | Azure Backup | Notes |
|--------------|----------------|-------------|-------|
| Azure VM | ❌ | ✅ (RSV) | Full-featured |
| Azure SQL Database | ✅ Auto (full/diff/log) | ✅ (LTR via Backup) | PITR built-in; LTR via Azure Backup |
| SQL Managed Instance | ✅ Auto | ✅ (LTR) | Same as SQL DB |
| Azure Cosmos DB | ✅ Auto (periodic or continuous) | ❌ | Cosmos manages its own |
| Azure Blob Storage | ✅ (Soft Delete, Versioning) | ✅ (Operational + Vaulted) | Multiple layers |
| Azure Files | ❌ | ✅ (RSV, share snapshots) | |
| Azure Disk | ❌ | ✅ (Backup Vault) | Snapshot-based |
| Azure DB for PostgreSQL | ✅ Auto (1–35 days) | ✅ (LTR up to 10 years via Backup Vault) | |
| AKS | ❌ | ✅ (Backup Vault + extension) | |
| App Service | Deployment slots (not backup) | ❌ | Use deployment slots + DB backup |

---

## 18. Exam Tips — HA / DR / Backup

### Core Concept Tips

| # | Tip | Trap |
|---|-----|------|
| 1 | **RTO = how FAST to recover; RPO = how much DATA can be lost** | Mixing up the two definitions |
| 2 | **Lower RTO + RPO = more expensive** — design to requirements, not exceed them | Over-engineering (active-active when pilot light suffices) |
| 3 | **Availability Set ≠ Availability Zone** — Set = same DC different racks; Zone = different DCs | Confusing Set and Zone |
| 4 | **99.95% SLA requires ≥ 2 VMs in Availability Set** with load balancer | Single VM only gets 99.9% |
| 5 | **99.99% SLA requires ≥ 2 VMs in Availability Zones** | Using Availability Set when AZ redundancy is required |
| 6 | **Cannot use Availability Set AND Availability Zones for same VMs** — mutually exclusive | Trying to combine both |

### SQL Database HA/DR Tips

| # | Tip | Trap |
|---|-----|------|
| 7 | **Business Critical has a built-in free read replica** in same region | Paying extra for read scale when BC already includes it |
| 8 | **Active Geo-Replication = SQL DB only** — SQL MI does NOT support it | Applying Active Geo-Replication to SQL MI |
| 9 | **Failover Groups = auto-failover + single connection string** — app doesn't change on failover | Active Geo-Replication requires connection string update |
| 10 | **SQL DB PITR is built-in** — no Azure Backup needed for PITR within 35 days | Thinking you need Azure Backup for Azure SQL Database PITR |
| 11 | **LTR (Long-Term Retention) for > 35 days** — configure separately | Assuming standard backup covers 7-year compliance requirement |

### Azure Backup Tips

| # | Tip | Trap |
|---|-----|------|
| 12 | **Azure VM backup minimum RPO = 4 hours (daily policy)** — not continuous | Assuming Azure Backup gives continuous VM protection |
| 13 | **SQL in VM backup = 15-minute RPO** (log backup) — much better than VM backup | Using VM-level backup for SQL Server (misses log backups) |
| 14 | **Blob operational backup = continuous (near-zero RPO)** — best for blob data protection | |
| 15 | **GRS vault required for Cross-Region Restore** — LRS/ZRS vaults cannot restore to secondary | Expecting CRR with LRS vault |
| 16 | **Recovery Services Vault vs Backup Vault** — RSV = VM/SQL/Files; Backup Vault = Disks/Blobs/PostgreSQL/AKS | Using wrong vault type for workload |
| 17 | **Archive tier minimum duration = 180 days** — moving out before that incurs early deletion fee | Archiving short-lived recovery points |

### ASR Tips

| # | Tip | Trap |
|---|-----|------|
| 18 | **ASR RPO = 30 seconds** (Azure-to-Azure VM replication) | Thinking ASR has same RPO as Azure Backup (it's much better) |
| 19 | **Test Failover is non-disruptive** — uses isolated VNet; production continues normally | Fearing test failover = fear of DR drills |
| 20 | **ASR charges compute only when failover occurs** — source VMs don't run in DR during protection | Thinking you pay for two sets of running VMs |
| 21 | **Recovery Plan** = ordered multi-VM failover; use for multi-tier apps | Failing individual VMs without considering dependency order |
| 22 | **Re-protect must run BEFORE failback** — after unplanned failover, re-protect reverses replication | Trying to failback without re-protecting |
| 23 | **App-consistent recovery points** require VSS/quiescing — more data-safe but less frequent | Using only crash-consistent for database VMs |

### Multi-Region & DR Strategy Tips

| # | Tip | Trap |
|---|-----|------|
| 24 | **Cosmos DB multi-write = 99.999% SLA + near-zero RPO** — highest available | Using single-write when question specifies zero data loss |
| 25 | **Failover Group RTO < 30 seconds, RPO < 5 seconds** — fastest for relational DB | Using Active Geo-Replication when auto-failover is needed |
| 26 | **Traffic Manager failover = DNS TTL delay** (~60s default) — not instant L7 failover | Expecting sub-second failover from Traffic Manager |
| 27 | **Front Door health probe failover** = faster (10 seconds) than Traffic Manager | Using TM when seconds matter |
| 28 | **Pilot Light = DB always running, compute stopped** — balance of cost vs RTO | Thinking Pilot Light = everything off |
| 29 | **Active-Active for databases needs conflict resolution** (Cosmos multi-write) — cannot always use | Proposing active-active SQL DB without considering write conflicts |
| 30 | **Storage failover after disaster = single-region LRS** — must re-enable GRS manually | Assuming storage is geo-redundant again right after failover |

### Cross-Domain Scenario Tips

| # | Scenario | Answer |
|---|----------|--------|
| 31 | "App must survive full Azure region outage, < 1 min RTO" | Active-Active + Cosmos multi-write + Front Door |
| 32 | "SQL Server workload, RTO < 30 sec, RPO < 5 sec" | SQL Database/MI with Auto-Failover Group |
| 33 | "Blob data must be recoverable from accidental deletion for 30 days" | Enable Blob Soft Delete (30-day retention) |
| 34 | "VM data must be recoverable even if entire Azure region fails" | Azure Backup with GRS vault + Cross-Region Restore |
| 35 | "Non-disruptive DR test for production VMs" | ASR Test Failover (isolated VNet) |
| 36 | "Compliance requires SQL backups retained for 7 years" | SQL Database Long-Term Retention (LTR) |
| 37 | "RTO 4 hours, RPO 1 hour, minimize cost" | Pilot Light / Backup-Restore (Azure Backup + ASR cold) |
| 38 | "On-prem files backed up to Azure, need 10 years retention" | MARS agent → Recovery Services Vault (99-year retention) |

---

## Quick Reference Summary

```
KEY FORMULAS:
  Composite SLA (series)  = SLA₁ × SLA₂ × SLA₃
  Parallel SLA (redundant) = 1 - (1-SLA₁) × (1-SLA₂)
  Availability = MTTF / (MTTF + MTTR)

AVAILABILITY OPTIONS (VMs):
  Single VM (Premium SSD)              = 99.9%
  Availability Set (2+ VMs)            = 99.95%
  Availability Zones (2+ VMs in 2+ AZ) = 99.99%

RPO BY SERVICE:
  Azure Backup (VM, daily policy)     = 4 hours
  SQL Server in VM (log backup)       = 15 minutes
  Azure Blob (operational backup)     = CONTINUOUS (near-zero)
  Azure Site Recovery (Azure→Azure)   = 30 SECONDS
  SQL Failover Group (async)          = < 5 seconds
  Cosmos DB multi-write               = NEAR ZERO

RTO BY STRATEGY:
  Active-Active (Front Door + Cosmos multi-write) = < 1 minute
  Warm Standby (Traffic Manager + Failover Group) = < 30 minutes
  Pilot Light (Scale-up compute)                  = 30–60 minutes
  Backup-Restore                                  = Hours

BACKUP VAULT TYPE:
  VMs / SQL in VMs / Azure Files / SAP HANA → Recovery Services Vault
  Azure Disks / Blobs / PostgreSQL / AKS    → Backup Vault

DR TIER SELECTION:
  "Zero downtime tolerance"      → Active-Active
  "< 30 sec RTO, < 5 sec RPO"   → Failover Groups
  "< 2 hr RTO, 30 sec RPO"      → ASR (Site Recovery)
  "Hours/days RTO acceptable"   → Azure Backup restore
```

---

*Last updated: May 20, 2026 | Domain 3 coverage: Business Continuity (15–20%) | Good luck tomorrow! 🎯*
