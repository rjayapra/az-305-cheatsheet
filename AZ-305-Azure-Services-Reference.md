# AZ-305 Azure Services Reference — Complete Study Notes
> Exam: May 21, 2026 | Covers: All tiers, compute options, latency, redundancy, cost, backup

---

## Table of Contents
1. [Compute Services](#1-compute-services)
2. [Database Services](#2-database-services)
3. [Storage Services](#3-storage-services)
4. [Analytics & Big Data](#4-analytics--big-data)
5. [Messaging & Integration](#5-messaging--integration)
6. [Networking Services](#6-networking-services)
7. [Identity & Security](#7-identity--security)
8. [Monitoring & Management](#8-monitoring--management)
9. [Backup, DR & Migration](#9-backup-dr--migration)
10. [Cross-Service Decision Tables](#10-cross-service-decision-tables)
11. [SLA Quick Reference](#11-sla-quick-reference)

---

## 1. COMPUTE SERVICES

### 1.1 Virtual Machines — Size Families

| Family | Category | CPU:Memory Ratio | Key Workload | Notes |
|--------|----------|-----------------|-------------|-------|
| **Av2** | General Purpose | ~1:4 | Dev/test, entry-level | Cheapest option |
| **B** | Burstable | Variable | Variable CPU workloads | Accrues CPU credits; burst above baseline |
| **D/Dv5** | General Purpose | 1:4 | Most production workloads | Balanced, common choice |
| **E/Ev5** | Memory Optimized | 1:8 | SAP, in-memory analytics, Redis | High memory ratio |
| **F/Fsv2** | Compute Optimized | 1:2 | Gaming, batch, web serving | High CPU ratio |
| **L/Lsv3** | Storage Optimized | 1:8 + local NVMe | NoSQL DBs, Elasticsearch | Local NVMe SSDs |
| **M** | Memory Intensive | up to 1:30 | SAP HANA, largest in-memory | Up to 4 TB RAM |
| **N (NC/ND/NV)** | GPU | — | ML training (NC/ND), visualization (NV) | NVIDIA GPUs |
| **H** | HPC | 1:4 + InfiniBand | Scientific simulations, MPI | High-perf networking |

> **Exam tip:** Questions about "VM with burstable CPU" → B-series. "Need InfiniBand/MPI" → H-series. "SAP HANA" → M-series.

---

### 1.2 VM Availability Options & SLA

| Configuration | SLA | Protection Against | Fault Domains | Update Domains |
|-------------|-----|------------------|---------------|----------------|
| Single VM (Premium SSD) | **99.9%** | Nothing | 1 | 1 |
| Single VM (Standard SSD) | **99.5%** | Nothing | 1 | 1 |
| Availability Set (2+ VMs) | **99.95%** | Rack/PDU/planned maintenance | 2–3 | Up to 20 |
| Availability Zone (2+ VMs in different zones) | **99.99%** | Datacenter failure | — | — |
| Multi-region (2+ regions) | **99.99%+** | Regional failure | — | — |

> **Availability Set** = groups VMs across fault domains (hardware racks) + update domains (patching batches)
> **Availability Zone** = physically separate datacenters (independent power/cooling/networking) within a region

---

### 1.3 VM Pricing Models

| Model | Discount vs PAYG | Commitment | Best For |
|-------|-----------------|-----------|---------|
| **Pay-as-you-go** | Baseline | None | Unpredictable workloads |
| **Reserved Instance (1-year)** | ~40% | 1 year | Steady-state production |
| **Reserved Instance (3-year)** | ~72% | 3 years | Long-running, stable workloads |
| **Spot VM** | Up to 90% | None (evictable) | Batch, fault-tolerant, stateless only |
| **Azure Hybrid Benefit (AHUB)** | ~40% on Windows/SQL | Own license | Customers with Windows Server/SQL licenses |
| **Dev/Test Pricing** | ~55% | None | Non-production subscriptions only |

---

### 1.4 Azure App Service Plans

| Tier | SLA | Instances | Scale | VNet | Deployment Slots | Auto-Scale | Cost |
|------|-----|-----------|-------|------|-----------------|-----------|------|
| **Free (F1)** | None | Shared | 1 | ❌ | 0 | ❌ | Free |
| **Shared (D1)** | None | Shared | 1 | ❌ | 0 | ❌ | Lowest |
| **Basic (B1–B3)** | 99.9% | 1–3 dedicated | Manual | ❌ | 0 | ❌ | Low |
| **Standard (S1–S3)** | 99.95% | 1–10 | Manual/Auto | ❌ | 5 | ✅ | Medium |
| **Premium v3 (P1v3–P3v3)** | 99.95% | 1–30 | Auto | ✅ (VNet integration) | 20 | ✅ | High |
| **Isolated v2 (I1v2–I3v2)** | 99.95% | 1–100 | Auto | Dedicated VNet (ASEv3) | 20 | ✅ | Highest |

> **VNet Integration** (outbound to VNet) = Premium v3+
> **Private Endpoint** (inbound from VNet) = Basic+
> **ASE (App Service Environment)** = single-tenant, fully isolated within your own VNet; use for strictest compliance
> **Deployment Slots** = staging environments; swap with zero-downtime (Standard+)

---

### 1.5 Azure Functions — Hosting Plans

| Plan | Cold Start | VNet Integration | Max Timeout | Scale Out | Min Instances | Cost Model |
|------|-----------|-----------------|------------|----------|--------------|-----------|
| **Consumption** | Yes | ❌ | 10 min | 0→200 instances (auto) | 0 | Per execution + GB-s |
| **Flex Consumption** | Minimized | ✅ | Configurable | 0→1000 (auto) | 0 (configurable) | Per execution + concurrency |
| **Premium (EP1–EP3)** | No (pre-warmed) | ✅ | Unlimited | Auto (pre-warmed pool) | 1+ | Higher baseline + per execution |
| **Dedicated (ASP)** | No | ✅ | Unlimited | Manual/Auto (ASP rules) | 1+ | Full App Service Plan cost + always-on |
| **Container Apps** | No | ✅ | Unlimited | KEDA-based | 0 | Per vCPU-second + memory |

> **Consumption** = no VNet integration (critical exam trap!)
> **Premium** = pre-warmed instances eliminate cold start; needed for VNet + low-latency
> **Functions timeout (Consumption):** default 5 min, max 10 min

---

### 1.6 Azure Kubernetes Service (AKS)

#### Cluster Tiers

| Tier | Control Plane SLA | Long-Term Support | Cost |
|------|------------------|------------------|------|
| **Free** | 99.5% (best effort) | ❌ | Free control plane |
| **Standard** | 99.9% (99.95% with AZs) | ❌ | ~$0.10/hr |
| **Premium** | 99.95% | ✅ (extended K8s versions) | ~$0.60/hr |

#### Node Pool Types

| Pool Type | Description | Use Case |
|-----------|-------------|---------|
| **System** | Required; runs core K8s services (CoreDNS, metrics) | Every cluster needs one |
| **User** | Application workloads | Separate workloads from system |
| **Spot** | Low-cost, evictable nodes | Batch, fault-tolerant jobs |
| **Virtual Nodes** | Burst to ACI | Rapid burst scaling without VM provisioning |

#### Networking Modes

| Mode | Pod IPs | VNet IPs Used | Notes |
|------|---------|--------------|-------|
| **Kubenet** | Private overlay | Node IPs only | Simpler, fewer IPs consumed |
| **Azure CNI** | VNet IPs directly | Every pod gets VNet IP | Required for Windows nodes; high IP consumption |
| **Azure CNI Overlay** | Overlay (RFC 1918) | Node IPs only | Best of both; recommended new default |

---

### 1.7 Azure Container Apps vs AKS vs Functions vs ACI

| Feature | ACI | Azure Functions | Container Apps | AKS |
|---------|-----|-----------------|----------------|-----|
| Orchestration | None | Managed | Managed (K8s) | Full K8s |
| Scaling | Manual (no auto-scale) | Trigger-based | KEDA + HTTP | HPA/KEDA/manual |
| Scale to zero | ❌ | ✅ (Consumption) | ✅ | ❌ |
| Cold start | ❌ | ✅ (Consumption) | Configurable | ❌ |
| VNet | ✅ injection | Premium/Flex only | ✅ injection | ✅ |
| Dapr | ❌ | ❌ | ✅ built-in | ❌ (add-on) |
| Cost model | Per second | Per execution | Per vCPU-s + mem | Node pools (always on) |
| SLA | 99.9% | 99.95% | 99.95% | 99.9%–99.95% |
| Best for | Simple containers, burst | Event-driven short tasks | Event-driven microservices | Complex microservices, full control |

---

### 1.8 Azure Dedicated Compute Options

| Option | Description | Use Case |
|--------|-------------|---------|
| **Azure Dedicated Host** | Physical server exclusively for your VMs | Compliance, hardware isolation, BYOL at host level |
| **Isolated VM sizes** | VMs isolated at hardware level (e.g., Standard_E96ids_v5) | Sensitive workloads requiring single-tenant hardware |
| **Proximity Placement Group (PPG)** | Co-locate VMs in same datacenter | Ultra-low latency between VMs (e.g., HPC, SAP) |

---

## 2. DATABASE SERVICES

### 2.1 Azure SQL Database

#### Purchase Models

| Model | Unit | Description | When to Use |
|-------|------|-------------|------------|
| **DTU** | Database Transaction Units | Bundled CPU + IO + Memory; simpler | Simple, predictable workloads |
| **vCore** | Virtual Cores | Separate CPU, memory, storage; transparent | Production, need control, AHUB eligible |

> **AHUB (Azure Hybrid Benefit)** only available with **vCore** model (save ~30% with SQL Server license)

---

#### DTU Tiers

| Tier | DTU Range | Max Storage | Max DB Size | In-Memory OLTP | Zone Redundancy | Backup Retention | Cost |
|------|----------|------------|------------|---------------|----------------|-----------------|------|
| **Basic** | 5 | 2 GB | 2 GB | ❌ | ❌ | 7 days | Lowest |
| **Standard (S0–S3)** | 10–100 | 250 GB | 1 TB | ❌ | ❌ | 7–35 days | Low |
| **Standard (S4–S12)** | 200–3,000 | 250 GB | 1 TB | ❌ | ❌ | 7–35 days | Medium |
| **Premium (P1–P4)** | 125–500 | 500 GB | 1 TB | ✅ | ✅ (optional) | 7–35 days | High |
| **Premium (P6–P15)** | 1,000–4,000 | 1 TB | 1 TB | ✅ | ✅ (optional) | 7–35 days | Highest |

---

#### vCore Service Tiers

| Tier | vCores | Storage Type | Max Storage | Max IOPS | Read Replica | Zone Redundancy | Latency | Cost |
|------|--------|-------------|------------|---------|------------|----------------|---------|------|
| **General Purpose** | 2–80 | Remote Premium SSD | 4 TB | 20,000 | ❌ (optional read scale) | Optional | 5–10 ms | Medium |
| **Business Critical** | 2–80 | Local SSD | 4 TB | 200,000 | ✅ (built-in, free) | Built-in (3 zones) | 1–2 ms | High |
| **Hyperscale** | 1–80 | Distributed storage | **100 TB** | Very high | Named replicas (up to 5) | ✅ | 1–2 ms | High |

> **Serverless** = compute tier under General Purpose; auto-pauses after inactivity; billed per second when active
> **Elastic Pool** = shared DTUs or vCores across multiple DBs; reduces cost when DBs have staggered peak usage

---

#### SQL Database: Latency, Throughput & HA Summary

| Feature | General Purpose | Business Critical | Hyperscale |
|---------|----------------|-----------------|-----------|
| Storage | Remote (Azure Premium) | Local SSD | Distributed (page server) |
| Read Latency | 5–10 ms | **1–2 ms** | 1–2 ms |
| Write Latency | 5–10 ms | **1–2 ms** | 1–3 ms |
| Max IOPS | 20,000 | **200,000** | Very high (scales with storage) |
| Max storage | 4 TB | 4 TB | **100 TB** |
| Built-in read replica | ❌ | ✅ | ✅ (named replicas) |
| Geo-replication | Up to 4 secondaries | Up to 4 secondaries | ✅ |
| Auto-failover groups | ✅ | ✅ | ✅ |
| Serverless option | ✅ | ❌ | ❌ |
| Backup | Full/diff/log | Full/diff/log | Log-based (instant) |
| PITR retention | 1–35 days | 1–35 days | 1–35 days |
| Long-term retention (LTR) | Up to 10 years | Up to 10 years | Up to 10 years |
| SLA | 99.99% | 99.99% | 99.99% |

---

### 2.2 Azure SQL Managed Instance

#### Tiers

| Tier | vCores | Storage Type | Max Storage | Max IOPS | Read Replica | Zone Redundancy | Latency | Cost |
|------|--------|-------------|------------|---------|------------|----------------|---------|------|
| **General Purpose** | 4–80 | Remote Premium SSD | 8 TB | 16,000 | ❌ | ✅ | 5–10 ms | Medium |
| **Business Critical** | 4–80 | Local SSD | 4 TB | 120,000 | ✅ (1 built-in) | ✅ | 1–2 ms | High |

#### SQL MI vs SQL Database — Feature Matrix

| Capability | SQL MI | SQL Database | Exam Relevance |
|-----------|--------|--------------|---------------|
| SQL Server Agent | ✅ | ❌ | High |
| CLR / SQLCLR | ✅ | Limited | Medium |
| Linked Servers | ✅ | ❌ | High |
| Cross-database queries | ✅ (same MI) | ❌ | High |
| Database Mail | ✅ | ❌ | Medium |
| Always On AG (native) | ✅ | Via failover groups | Medium |
| VNet deployment | ✅ (always) | Optional (via PE) | High |
| Lift & shift from SQL Server | ✅ **Best option** | ❌ (rework needed) | High |
| Auto-scale (serverless) | ❌ | ✅ (GP only) | Medium |
| Elastic Pools | ❌ | ✅ | Medium |
| Near-zero downtime migration | ✅ (DMS online) | ✅ | High |
| SLA | 99.99% | 99.99% | — |

---

### 2.3 SQL Server on Azure VMs

| Feature | Detail |
|---------|--------|
| **Compatibility** | 100% SQL Server (full feature parity) |
| **Management** | Full control — patching, config, OS |
| **HA options** | Always On Availability Groups, Failover Cluster Instances (Azure shared disks) |
| **DR** | Azure Site Recovery + SQL HA |
| **Backup** | Azure Backup for SQL in VMs; or native SQL backup to Blob |
| **Licensing** | PAYG (bundled) or AHUB (bring own SQL Server license) |
| **Cost** | Highest (VM + OS + SQL license) |
| **SLA** | Up to 99.99% with AZs |

#### SQL Option Comparison

| Factor | SQL Database | SQL Managed Instance | SQL on VM |
|--------|------------|---------------------|-----------|
| SQL Server compatibility | ~95% | ~99.9% | **100%** |
| Management overhead | **Lowest** | Medium | Highest |
| Cost | **Lowest** | Medium | Highest |
| Scale | Auto/Serverless | Manual | Manual |
| Lift & shift ease | ❌ | ✅ | ✅ |
| Best for | New cloud-native apps | Lift & shift existing SQL Server | Full control/legacy |

---

### 2.4 Azure Cosmos DB

#### Capacity Modes

| Mode | RU/s Provisioning | Auto-scale | Cold Start | Cost |
|------|------------------|-----------|-----------|------|
| **Provisioned (Manual)** | Fixed RU/s (min 400) | ❌ | None | Steady baseline |
| **Provisioned (Autoscale)** | 10% to max RU/s | ✅ | None | 1.5× manual at max |
| **Serverless** | No provisioning | ✅ per operation | None | Pay per RU consumed |

> **1 RU** = read 1 KB document | Write ≈ 5× read cost | **Serverless**: ideal for dev/test and intermittent workloads

---

#### Consistency Levels (Strongest → Weakest)

| Level | Latency | Throughput | Staleness | Multi-write Support | RPO | Use Case |
|-------|---------|-----------|---------|-------------------|-----|---------|
| **Strong** | High (waits for quorum) | Lowest | None | ❌ Not supported | 0 ms | Financial transactions |
| **Bounded Staleness** | Medium-High | Medium | Max K ops or T seconds lag | ✅ (same region only) | Bounded | Near-real-time global |
| **Session** ⭐ | Low-Medium | High | Consistent within session | ✅ | Unknown | Most web/mobile apps (default) |
| **Consistent Prefix** | Low | High | No out-of-order reads | ✅ | Unknown | Social feeds, telemetry |
| **Eventual** | Lowest | Highest | May be stale | ✅ | Unknown | Non-critical counters, likes |

> **Strong consistency is NOT available with multi-write regions** — exam trap!

---

#### Cosmos DB APIs

| API | Data Model | Best For | Migration From |
|-----|-----------|---------|---------------|
| **NoSQL (Core)** | JSON documents | Default; all new apps | — |
| **MongoDB** | BSON documents | Existing MongoDB apps | MongoDB |
| **Cassandra** | Wide-column | Time-series, IoT telemetry | Cassandra |
| **Gremlin** | Graph (nodes + edges) | Graph traversals, social networks | Gremlin/TinkerPop |
| **Table** | Key-value | Simple k/v, replacing Azure Table Storage | Azure Table Storage |

---

#### Cosmos DB: Regional Support & HA

| Feature | Detail |
|---------|--------|
| **Single-write region** | 1 write + up to 25 read replicas globally; 99.99% availability SLA |
| **Multi-write region** | Write to any region; conflict resolution required; 99.999% SLA |
| **Conflict resolution** | Last Write Wins (LWW by timestamp), Custom (stored procedure), Manual (conflict feed) |
| **Zone redundancy** | Optional per region (adds zone-level HA) |
| **SLA (single region)** | 99.99% read + write |
| **SLA (multi-region writes)** | 99.999% read + write |

#### Cosmos DB Backup

| Mode | RPO | Retention | Restore Process | Cost |
|------|-----|-----------|----------------|------|
| **Periodic** | Up to 4 hours | 30 days (2 backup copies) | Request via Microsoft support | Included |
| **Continuous (7-day)** | Up to 1 hour | 7 days | Self-service PITR in portal | Additional |
| **Continuous (30-day)** | Up to 1 hour | 30 days | Self-service PITR in portal | Higher |

---

### 2.5 Azure Database for PostgreSQL — Flexible Server

#### Compute Tiers

| Tier | vCores | RAM | Max IOPS | HA Support | Zone Redundancy | Geo-redundant Backup | Cost |
|------|--------|-----|---------|-----------|----------------|---------------------|------|
| **Burstable (B)** | 1–2 | 2–8 GB | Up to 2,400 | ❌ | ❌ | ❌ | Lowest |
| **General Purpose (D)** | 2–96 | 8–384 GB | Up to 20,000 | ✅ | ✅ | ✅ | Medium |
| **Memory Optimized (E)** | 2–96 | 16–672 GB | Up to 20,000 | ✅ | ✅ | ✅ | High |

#### HA, Latency & Backup (Flexible Server)

| Feature | Burstable | General Purpose | Memory Optimized |
|---------|---------|----------------|-----------------|
| Zone-redundant HA | ❌ | ✅ (99.99% SLA) | ✅ (99.99% SLA) |
| Same-zone HA | ❌ | ✅ (99.9% SLA) | ✅ (99.9% SLA) |
| Read replicas | Up to 5 | Up to 5 | Up to 5 |
| Backup retention | 1–35 days | 1–35 days | 1–35 days |
| Geo-redundant backup | ❌ | ✅ | ✅ |
| Point-in-time restore | ✅ | ✅ | ✅ |
| Max storage | 64 TB | 64 TB | 64 TB |
| Read latency | Low | Low | Low |
| Throughput | Moderate | High | Very High |

---

### 2.6 Azure Database for MySQL — Flexible Server

| Tier | vCores | RAM | Max IOPS | HA/Zone Redundancy | Geo-redundant Backup | Cost |
|------|--------|-----|---------|-------------------|---------------------|------|
| **Burstable** | 1–2 | 1–8 GB | Up to 1,600 | ❌ | ❌ | Lowest |
| **General Purpose** | 2–64 | 4–256 GB | Up to 20,000 | ✅ | ✅ | Medium |
| **Memory Optimized** | 2–64 | 8–512 GB | Up to 20,000 | ✅ | ✅ | High |

> Backup retention: 1–35 days. Same HA model as PostgreSQL Flexible Server (zone-redundant HA = 99.99%).

---

### 2.7 Azure Cache for Redis

#### Tiers

| Tier | Architecture | Persistence | Clustering | VNet / PE | Geo-Replication | Max Memory | SLA | Cost |
|------|------------|------------|-----------|-----------|----------------|-----------|-----|------|
| **Basic** | Single node | ❌ | ❌ | ❌ | ❌ | 53 GB | None (no SLA) | Lowest |
| **Standard** | Primary + replica | ❌ | ❌ | ❌ | ❌ | 53 GB | 99.9% | Low |
| **Premium** | Primary + replica + optional shards | ✅ (RDB + AOF) | ✅ (up to 10 shards) | ✅ | Active-passive | 1.2 TB | 99.9% | Medium |
| **Enterprise** | Redis Enterprise active-active | ✅ | ✅ | ✅ | Active-active | 2 TB | 99.9% | High |
| **Enterprise Flash** | Redis on SSD + RAM | ✅ | ✅ | ✅ | Active-active | 13 TB | 99.9% | Highest |

#### Redis Latency & Throughput

| Tier | Latency | Throughput | Key Extra Feature |
|------|---------|-----------|-----------------|
| Basic | Sub-millisecond | Low | Dev/test only, no HA |
| Standard | Sub-millisecond | Medium | HA via replica |
| Premium | Sub-millisecond | High | Persistence + VNet + clustering |
| Enterprise | Sub-millisecond | Very high | Redis modules (Search, JSON, TimeSeries, etc.) |
| Enterprise Flash | Sub-millisecond | High | Cost-effective for large datasets (SSD-backed) |

> **Basic has no SLA** — never use for production
> **Persistence:** RDB = periodic snapshot; AOF = append-only log (more durable, more write overhead)

---

## 3. STORAGE SERVICES

### 3.1 Azure Blob Storage — Access Tiers

| Tier | Access Latency | Storage Cost | Access/Transfer Cost | Min Storage Duration | Early Deletion Fee | Use Case |
|------|--------------|------------|---------------------|---------------------|-------------------|---------|
| **Hot** | Milliseconds | Highest | Lowest | None | None | Frequently accessed (active) |
| **Cool** | Milliseconds | Lower | Higher | 30 days | Pro-rated if deleted early | Infrequently accessed |
| **Cold** | Milliseconds | Even lower | Higher still | 90 days | Pro-rated | Rarely accessed, must stay accessible |
| **Archive** | **1–15 hours** (rehydrate first) | Lowest | Highest | 180 days | Pro-rated | Compliance, long-term backup |

> **Archive rehydration:** Standard (up to 15 hrs) or High Priority (< 1 hr, more expensive)
> **Lifecycle management** = auto-move blobs between tiers based on age/last-accessed rules

---

### 3.2 Storage Account Redundancy

| Redundancy | Total Copies | Zone Protected | Geo Protected | Secondary Readable | Durability | SLA | Cost |
|-----------|------------|--------------|-------------|------------------|----------|-----|------|
| **LRS** | 3 | ❌ | ❌ | ❌ | 11 nines | 99.9% | Lowest |
| **ZRS** | 3 | ✅ (3 zones) | ❌ | ❌ | 12 nines | 99.9% | Low |
| **GRS** | 6 | ❌ | ✅ (2 regions) | ❌ | 16 nines | 99.9% (primary) | Medium |
| **GZRS** | 6 | ✅ (3 zones) | ✅ (2 regions) | ❌ | 16 nines | 99.9% | Medium-High |
| **RA-GRS** | 6 | ❌ | ✅ (2 regions) | ✅ secondary | 16 nines | **99.99%** read | High |
| **RA-GZRS** | 6 | ✅ (3 zones) | ✅ (2 regions) | ✅ secondary | 16 nines | **99.99%** read | Highest |

> **GRS secondary is NOT readable** unless you choose RA-GRS — common exam trap!
> Secondary region is async replica; **RPO typically < 15 minutes** (no guarantee)

---

### 3.3 Storage Account Types

| Account Type | Services Supported | Performance Tiers | Blob Tiers (Hot/Cool/Archive) | Notes |
|-------------|------------------|------------------|-------------------------------|-------|
| **Standard GPv2** | Blob, Files, Queue, Table, Data Lake | Standard (HDD) | ✅ All tiers | **Recommended; most flexible** |
| **Standard GPv1** | Blob, Files, Queue, Table | Standard | ❌ (no Archive) | Legacy — migrate to GPv2 |
| **Premium Block Blob** | Block Blobs only | Premium (SSD) | ❌ (Hot only) | High-transaction, low-latency workloads |
| **Premium File Share** | Azure Files only | Premium (SSD) | ❌ | High-IOPS file shares (NFS/SMB) |
| **Premium Page Blob** | Page Blobs only | Premium (SSD) | ❌ | VM unmanaged disks (legacy) |

---

### 3.4 Azure Files

| Tier | Storage Medium | Max IOPS | Max Throughput | Protocol | Redundancy | Use Case | Cost |
|------|--------------|---------|--------------|---------|-----------|---------|------|
| **Standard (Transaction Optimized)** | HDD | 10,000 | 300 MB/s | SMB/NFS | LRS/ZRS/GRS | General file shares | Low |
| **Standard (Hot)** | HDD | 10,000 | 300 MB/s | SMB | LRS/ZRS/GRS | Frequently accessed shares | Low |
| **Standard (Cool)** | HDD | 10,000 | 300 MB/s | SMB | LRS/ZRS | Online archival | Lowest |
| **Premium** | SSD | **100,000** | 10 GB/s | SMB/NFS | LRS/ZRS | Latency-sensitive, high IOPS | High |

> **NFS protocol** requires **Premium** tier + VNet (no public endpoint for NFS)
> **Azure File Sync** = sync on-prem Windows file servers to Azure Files; **cloud tiering** moves cold files to Azure (hot files remain local)

---

### 3.5 Azure Disk Storage

| Disk Type | Max IOPS | Max Throughput | Latency | Max Size | Redundancy | Best For | Relative Cost |
|-----------|---------|--------------|---------|---------|-----------|---------|--------------|
| **Standard HDD** | 2,000 | 500 MB/s | 10s of ms | 32 TB | LRS/ZRS | Backup, archive, non-critical | Lowest |
| **Standard SSD** | 6,000 | 750 MB/s | Single-digit ms | 32 TB | LRS/ZRS | Web servers, dev/test | Low |
| **Premium SSD** | 20,000 | 900 MB/s | < 1 ms | 32 TB | LRS/ZRS | Production databases, apps | Medium |
| **Premium SSD v2** | 80,000 | 1,200 MB/s | < 1 ms | 64 TB | LRS | High-performance apps | High |
| **Ultra Disk** | **400,000** | **10,000 MB/s** | Sub-ms | 64 TB | LRS (zone-pinned) | SAP HANA, top-tier databases | Highest |

> **Ultra Disk** must be in the same Availability Zone as the VM; not available in all regions/VM sizes
> **VM SLA of 99.9%** requires **Premium SSD** (or better) for managed OS disk
> **ZRS disks** (Premium SSD + Standard SSD) = zone-redundant; higher cost but no data loss on zone failure

---

### 3.6 Azure Data Lake Storage Gen2

| Feature | Detail |
|---------|--------|
| **Built on** | Azure Blob Storage (GPv2) with hierarchical namespace (HNS) enabled |
| **Protocol** | HDFS-compatible + Azure Blob REST APIs |
| **Access control** | POSIX ACLs + Azure RBAC (fine-grained) |
| **Performance** | Optimized for analytics; parallel reads/writes |
| **Redundancy** | LRS / ZRS / GRS / RA-GRS |
| **Integration** | Azure Synapse, Databricks, HDInsight, Azure Data Factory |
| **Tiers** | Hot / Cool / Archive (same as Blob Storage) |
| **Cost** | Standard (HDD) or Premium (SSD); pay per GB + transactions |

---

## 4. ANALYTICS & BIG DATA

### 4.1 Azure Synapse Analytics

#### Pool Types

| Pool | Engine | Compute Unit | Scaling | Use Case | Cost |
|------|--------|-------------|---------|---------|------|
| **Dedicated SQL Pool** | MPP (Massively Parallel) | DWU (Data Warehouse Units) | Manual scale up/down; pause when idle | Planned large-scale data warehouse | DWU-based |
| **Serverless SQL Pool** | On-demand | TB scanned | Auto | Ad-hoc queries over Data Lake | Per TB scanned |
| **Apache Spark Pool** | Spark | Nodes (small/medium/large) | Auto-scale + auto-pause | Data engineering, ML, transformation | Per node-hour |

#### Dedicated SQL Pool — DWU Tiers

| DWU | Compute Nodes | Max Concurrent Queries | Use Case | Relative Cost |
|-----|-------------|----------------------|---------|--------------|
| DW100c | 1 | 4 | Dev/Test | Lowest |
| DW500c | 5 | 10 | Small production | Low |
| DW1000c | 10 | 20 | Medium analytics | Medium |
| DW5000c | 50 | 40 | Large analytics | High |
| DW30000c | 300 | 128 | Enterprise-scale | Highest |

> **Pause Dedicated SQL Pool** when not in use (stops compute charges; storage charges continue)
> **Serverless pool** is always available at no base cost — pay only for queries run

---

### 4.2 Azure Data Factory

| Feature | Detail |
|---------|--------|
| **Purpose** | ETL/ELT pipelines, data movement, orchestration |
| **SLA** | 99.9% |
| **Pricing** | Per pipeline run, DIU-hours (data movement), activity runs |

#### Integration Runtime Types

| IR Type | Connects To | Deployment | Use Case |
|---------|------------|-----------|---------|
| **Azure IR** | Cloud data stores | Managed by Azure | Cloud-to-cloud data movement |
| **Self-hosted IR** | On-prem or private networks | Customer-managed VM | On-prem sources behind firewall |
| **Azure-SSIS IR** | SSIS packages | Managed cluster | Lift & shift SSIS to cloud |

---

### 4.3 Azure Databricks

| Feature | Detail |
|---------|--------|
| **Based on** | Apache Spark (managed) |
| **Use case** | Data engineering, ML, collaborative analytics |
| **Tiers** | Standard, Premium (role-based access, audit, AAD integration), Trial |
| **Compute** | Auto-scaling clusters (CPUs or GPUs) |
| **Cost** | DBU (Databricks Unit) + underlying VM cost |
| **Integration** | ADLS Gen2, Synapse, ADF, MLflow, Azure ML |

---

## 5. MESSAGING & INTEGRATION

### 5.1 Azure Event Hubs

#### Tiers

| Tier | Throughput Unit | Consumer Groups | Retention | Kafka | Private Endpoint | Partitions | Cost |
|------|----------------|----------------|---------|-------|-----------------|-----------|------|
| **Basic** | 1–20 TU | **1** | **1 day** | ❌ | ❌ | 2 | Lowest |
| **Standard** | 1–20 TU (scale up) | **20** | Up to 7 days | ✅ | ❌ | 2–32 | Medium |
| **Premium** | PU (Processing Unit) | Unlimited | Up to 90 days | ✅ | ✅ | 1–100 | High |
| **Dedicated** | CU (Capacity Unit) | Unlimited | Up to 90 days | ✅ | ✅ | Custom | Highest |

> **1 Throughput Unit (TU)** = 1 MB/s ingress, 2 MB/s egress
> **Premium** uses isolated compute; **Dedicated** = single-tenant cluster
> **Basic** has only 1 consumer group — NOT suitable for multiple independent consumers

#### Event Hubs: Latency & Throughput

| Tier | Latency | Max Ingress | Notes |
|------|---------|------------|-------|
| Basic / Standard | Milliseconds | 1 MB/s per TU | Multi-tenant |
| Premium | Milliseconds | Up to 1 GB/s per PU | Isolated, consistent |
| Dedicated | Milliseconds | Up to 256 Mbps per CU | Single-tenant, predictable |

---

### 5.2 Azure Service Bus

#### Tiers

| Feature | Basic | Standard | Premium |
|---------|-------|---------|---------|
| Queues | ✅ | ✅ | ✅ |
| Topics & Subscriptions | ❌ | ✅ | ✅ |
| Sessions (FIFO ordering) | ❌ | ✅ | ✅ |
| Transactions | ❌ | ✅ | ✅ |
| Message size | 256 KB | 256 KB | **100 MB** |
| Geo-DR | ❌ | ❌ | ✅ |
| VNet / Private Endpoint | ❌ | ❌ | ✅ |
| Dedicated capacity | ❌ | ❌ | ✅ |
| Message retention | 14 days | 14 days | 14 days |
| SLA | 99.9% | 99.9% | 99.9% |
| Cost | Lowest | Medium | Highest |

> **Topics/Subscriptions** = publish-subscribe; needs **Standard or Premium**
> **VNet integration** = **Premium** only
> **Message Lock** ≠ delete — receiver must explicitly complete message after processing

---

### 5.3 Azure Event Grid

| Feature | Detail |
|---------|--------|
| **Model** | Push-based, reactive event routing |
| **Latency** | Near real-time (~seconds) |
| **Sources (Publishers)** | Azure services (Blob, Resource Groups, Custom Topics, Event Grid Namespaces) |
| **Handlers (Subscribers)** | Functions, Logic Apps, Webhooks, Event Hubs, Service Bus |
| **Retry** | Up to 24 hours with exponential backoff |
| **SLA** | 99.99% |
| **Cost** | Per 1M operations (first 100K/month free) |
| **Filtering** | Subject prefix/suffix + advanced attribute filters |

#### Event Grid vs Event Hubs vs Service Bus

| | Event Grid | Event Hubs | Service Bus |
|-|-----------|-----------|------------|
| Model | Push (reactive) | Pull/Push (streaming) | Pull (queue/topic) |
| Order guarantee | ❌ | Per partition | ✅ (sessions) |
| Consumer groups | ❌ | ✅ | ✅ (subscriptions) |
| Replay events | ❌ | ✅ (retention) | ❌ |
| Max message size | 1 MB | 1 MB | 100 MB (Premium) |
| Best for | React to state changes | Telemetry, log streaming | Reliable message queuing |

---

### 5.4 Azure Logic Apps

| Feature | **Consumption** | **Standard** |
|---------|----------------|-------------|
| Hosting | Multi-tenant | Single-tenant (or ASE) |
| Pricing | Per action/trigger execution | Fixed plan + per execution |
| VNet Integration | ❌ | ✅ |
| Connectors | 400+ managed | 400+ + local connections |
| Stateful workflows | ✅ | ✅ |
| Stateless workflows | ❌ | ✅ (faster, lower state overhead) |
| SLA | 99.9% | 99.95% |
| Throughput | Limited | Higher; parallelism |
| On-prem connectivity | Via on-prem data gateway | ✅ native |

---

### 5.5 Azure API Management (APIM)

| Tier | SLA | Units | VNet Integration | Multi-Region | Dev Portal | Cache | Cost |
|------|-----|-------|-----------------|-------------|-----------|-------|------|
| **Consumption** | 99.9% | Serverless | ❌ | ❌ | ❌ | External only | Per call (lowest) |
| **Developer** | None | 1 | External VNet | ❌ | ✅ | Internal | Low (no SLA) |
| **Basic** | 99.9% | 1–2 | ❌ | ❌ | ✅ | Internal | Medium |
| **Standard** | 99.9% | 1–4 | ❌ | ❌ | ✅ | Internal | Medium-High |
| **Premium** | **99.95%** | Up to 31 | Internal+External VNet | ✅ | ✅ | Internal | Highest |

> **Multi-region** requires **Premium**
> **Internal VNet** (private API gateway) requires **Premium**
> **Consumption** has NO VNet and NO dev portal

---

## 6. NETWORKING SERVICES

### 6.1 Azure VPN Gateway

#### SKUs

| SKU | Max Throughput | S2S VPN Tunnels | P2S SSTP | IKEv2/OpenVPN | BGP | Zone Redundant | Cost |
|-----|--------------|----------------|---------|--------------|-----|---------------|------|
| **Basic** | 100 Mbps | 10 | ✅ | ❌ IKEv2 | ❌ | ❌ | Lowest |
| **VpnGw1** | 650 Mbps | 30 | ✅ | ✅ | ✅ | ❌ | Low |
| **VpnGw2** | 1 Gbps | 30 | ✅ | ✅ | ✅ | ❌ | Medium |
| **VpnGw3** | 1.25 Gbps | 30 | ✅ | ✅ | ✅ | ❌ | Medium-High |
| **VpnGw4** | 5 Gbps | 100 | ✅ | ✅ | ✅ | ❌ | High |
| **VpnGw5** | 10 Gbps | 100 | ✅ | ✅ | ✅ | ❌ | Highest |
| **VpnGw1AZ–5AZ** | Same as non-AZ | Same | ✅ | ✅ | ✅ | ✅ | +zone premium |

> **Basic SKU:** No IKEv2, no BGP, not supported for ExpressRoute coexistence — avoid for production
> **SLA:** 99.9% (single tunnel), 99.95% (active-active)

#### VPN Connection Types

| Type | Use Case |
|------|---------|
| **Site-to-Site (S2S)** | On-prem datacenter → Azure (VPN device required) |
| **Point-to-Site (P2S)** | Individual remote clients → Azure |
| **VNet-to-VNet** | Azure VNet → Azure VNet (cross-region; use peering for same-region) |
| **ExpressRoute coexistence** | VPN as failover for ExpressRoute |

---

### 6.2 Azure ExpressRoute

#### Circuit SKUs

| SKU | Region Coverage | Metering | Cost |
|-----|---------------|---------|------|
| **Local** | Single Azure region (same metro as peering location) | **Unlimited data included** | Lowest |
| **Standard** | All Azure regions in same geopolitical boundary (e.g., all of North America) | Metered or Unlimited | Medium |
| **Premium** | **Global** — all Azure regions worldwide | Metered or Unlimited | Highest |

> **ExpressRoute Global Reach** (Premium add-on) = connect two on-prem sites via Azure backbone

#### Bandwidth Options
`100 Mbps | 200 Mbps | 500 Mbps | 1 Gbps | 2 Gbps | 5 Gbps | 10 Gbps | 100 Gbps`

#### Peering Types

| Peering | Connects To | Use Case |
|---------|------------|---------|
| **Azure Private Peering** | VNet resources (VMs, LBs, PaaS via PE) | Primary enterprise connectivity |
| **Microsoft Peering** | Microsoft 365, Azure PaaS public endpoints | SaaS + Azure public services |

#### ExpressRoute vs VPN Gateway

| | ExpressRoute | VPN Gateway |
|-|-------------|------------|
| Connection type | Private (telco circuit) | Encrypted over public internet |
| Max bandwidth | **100 Gbps** | 10 Gbps |
| Latency | Consistent, low | Variable |
| SLA | **99.95%** | 99.9% |
| Setup time | Weeks | Hours |
| Cost | High | Low–Medium |
| Internet dependency | ❌ | ✅ |
| Encryption | Not by default (MACsec optional) | ✅ IPsec |

---

### 6.3 Azure Application Gateway

#### Tiers

| Tier | WAF | WAF Policy | Autoscale | AZ Support | URL Routing | TLS Offload | Cost |
|------|-----|-----------|---------|----------|------------|------------|------|
| **Standard v2** | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ | Medium |
| **WAF v2** | ✅ (OWASP 3.x) | ✅ | ✅ | ✅ | ✅ | ✅ | Higher |

> Always use **v2** SKUs — v1 is legacy and being retired
> **WAF Modes:** Detection (log only, no blocking) vs Prevention (block malicious requests)
> **WAF Rule Sets:** OWASP 3.2 (default), Microsoft Bot Manager, Custom Rules

---

### 6.4 Azure Front Door

#### Tiers

| Feature | Standard | Premium |
|---------|---------|---------|
| Custom domains + HTTPS | ✅ | ✅ |
| CDN / Caching | ✅ | ✅ |
| WAF (OWASP basic) | Basic | Full OWASP + bot + custom |
| Bot protection | Limited | ✅ Full |
| DDoS (network layer) | ❌ | ✅ |
| Private Link to origins | ❌ | ✅ |
| Security reports | ❌ | ✅ |
| Cost | Medium | Higher |
| SLA | 99.99% | 99.99% |

> **Front Door Premium with Private Link** = keep backend PaaS services (App Service, Storage, etc.) fully private
> Front Door operates at Azure's **anycast edge PoPs** (100+ locations globally) — fastest response

#### Front Door Routing Algorithms

| Method | Behavior |
|--------|---------|
| **Latency** | Route to lowest-latency backend |
| **Priority** | Primary → failover to secondary |
| **Weighted** | Distribute % of traffic across backends |
| **Session affinity** | Same user session → same origin |

---

### 6.5 Load Balancing: Full Comparison

| Service | Scope | Layer | Protocol | WAF | SSL Offload | Routing | Cost |
|---------|-------|-------|----------|-----|------------|---------|------|
| **Azure Load Balancer (Standard)** | Regional | L4 | Any (TCP/UDP) | ❌ | ❌ | IP/port hash, HA ports | Low |
| **Application Gateway (WAF v2)** | Regional | L7 | HTTP/HTTPS | ✅ | ✅ | URL path, host header | Medium |
| **Traffic Manager** | Global | DNS | Any | ❌ | ❌ | DNS-based | Low |
| **Azure Front Door (Premium)** | Global | L7 | HTTP/HTTPS | ✅ | ✅ | Latency, priority, weighted, affinity | High |

> **Azure Load Balancer Basic** is free but no SLA — use **Standard** (99.99% SLA) for production
> **Traffic Manager** is DNS routing — NOT a proxy; does NOT inspect HTTP traffic; failover takes ~60–300s (DNS TTL)

---

### 6.6 Azure Firewall

#### Tiers

| Feature | Basic | Standard | Premium |
|---------|-------|---------|---------|
| Stateful L4 filtering | ✅ | ✅ | ✅ |
| FQDN application rules | ✅ | ✅ | ✅ |
| Network rules | ✅ | ✅ | ✅ |
| Threat intelligence | ❌ | ✅ Alert only | ✅ Alert + Deny |
| TLS inspection | ❌ | ❌ | ✅ |
| IDPS (Signature-based) | ❌ | ❌ | ✅ |
| URL filtering | ❌ | ❌ | ✅ |
| Web categories | ❌ | ❌ | ✅ |
| Cost | Lowest | Medium | Highest |
| SLA | 99.95% | 99.95% | 99.95% |

> **Premium** = required for IDPS (Intrusion Detection & Prevention System) and TLS inspection
> **Firewall Policy** = centralized rule management for multiple firewalls (Structured Rules + RBAC + Inheritance)

---

### 6.7 Azure DDoS Protection

| Plan | Scope | Auto-Mitigation | Cost Guarantee | DRT Access | Cost |
|------|-------|----------------|---------------|-----------|------|
| **Default (free)** | Azure infrastructure | Basic | ❌ | ❌ | Free |
| **Network Protection** | Per VNet | ✅ Volumetric | ✅ (Azure credit) | ✅ 24/7 | ~$2,944/mo per VNet |
| **IP Protection** | Per public IP | ✅ | ✅ | ✅ 24/7 | Per-IP pricing |

> Default protection is **free** but only protects Azure infrastructure — NOT your workloads
> Network Protection includes attack analytics, rapid response, and SLA guarantee

---

### 6.8 Azure Bastion

#### Tiers

| Feature | Basic | Standard | Premium |
|---------|-------|---------|---------|
| RDP/SSH via browser | ✅ | ✅ | ✅ |
| Native client (RDP/SSH tools) | ❌ | ✅ | ✅ |
| File transfer | ❌ | ✅ | ✅ |
| IP-based connection | ❌ | ✅ | ✅ |
| Shareable links | ❌ | ✅ | ✅ |
| Session recording | ❌ | ❌ | ✅ |
| VNet peering support | ❌ | ✅ | ✅ |
| Cost | Very Low | Higher | Highest |
| SLA | 99.95% | 99.95% | 99.95% |

> No public IP required on VMs — Bastion connects over TLS/443
> Bastion host deploys into a dedicated subnet named **`AzureBastionSubnet`** (/26 or larger required)

---

### 6.9 Azure Virtual WAN

| Tier | VPN (S2S) | P2S VPN | ExpressRoute | VNet-to-VNet | Routing | Cost |
|------|----------|--------|-------------|------------|---------|------|
| **Basic** | ✅ S2S only | ❌ | ❌ | ❌ | Basic | Low |
| **Standard** | ✅ S2S + P2S | ✅ | ✅ | ✅ | Full (custom routes, BGP) | Higher |

> Microsoft-managed hub; automated hub-and-spoke at scale
> **Hub VNet** = managed by Azure (no access to hub compute); spoke VNets connect via VNet connections

---

### 6.10 Additional Networking Services

| Service | Purpose | Key Notes |
|---------|---------|-----------|
| **Private Endpoint** | Private IP for a specific PaaS resource in your VNet | Traffic stays in VNet; blocks public access |
| **Service Endpoint** | Optimized route to PaaS; resource still has public IP | NOT truly private — public IP still active |
| **Azure DNS Private Zones** | Private DNS resolution within VNet | Autoregistration for VMs; link to multiple VNets |
| **NAT Gateway** | Outbound-only SNAT for VNet subnets | Single/multiple public IPs; no inbound; scales to 50 Gbps |
| **Azure CDN** | Cache static content at edge PoPs | Providers: Microsoft, Verizon, Akamai |
| **VNet Peering** | Low-latency VNet-to-VNet connectivity | **NOT transitive** — A↔B + B↔C ≠ A↔C |
| **Azure DNS** (Public) | Host public DNS zones | 100% SLA; anycast global |

> **Private Endpoint** > **Service Endpoint** for security (exam almost always prefers Private Endpoint)

---

## 7. IDENTITY & SECURITY

### 7.1 Microsoft Entra ID (Azure AD) — License Tiers

| Feature | Free | P1 | P2 |
|---------|------|-----|-----|
| SSO — limited apps | Up to 10 | Unlimited | Unlimited |
| MFA (basic) | ✅ | ✅ | ✅ |
| **Conditional Access** | ❌ | ✅ | ✅ |
| SSPR (cloud users) | ✅ | ✅ | ✅ |
| SSPR writeback to on-prem | ❌ | ✅ | ✅ |
| Dynamic Groups | ❌ | ✅ | ✅ |
| **PIM (Privileged Identity Management)** | ❌ | ❌ | **✅** |
| **Identity Protection** | ❌ | ❌ | **✅** |
| **Access Reviews** | ❌ | ❌ | **✅** |
| Entitlement Management | ❌ | ❌ | **✅** |

> **Conditional Access = P1 minimum** | **PIM = P2** — most-tested exam fact in this domain

---

### 7.2 Hybrid Identity Solutions

| Scenario | Recommended Solution | Notes |
|----------|---------------------|-------|
| Sync on-prem AD to Entra ID + Password Hash Sync | **Azure AD Connect (PHS)** | Simplest; passwords synced as hash |
| Sync without cloud-stored passwords | **Azure AD Connect + PTA (Pass-through Auth)** | Auth happens on-prem |
| Federated auth with ADFS | **Azure AD Connect + AD FS** | Complex; legacy; requires ADFS servers |
| Lightweight sync across multiple forests | **Azure AD Connect Cloud Sync** | Agent-based; simpler infra |
| On-prem apps reachable from Entra ID | **Application Proxy** | No inbound firewall ports needed |
| IaaS apps needing domain join (LDAP/Kerberos) | **Azure AD Domain Services (AADDS)** | Managed AD DS; no on-prem required |

> **Cloud Sync vs AD Connect:** Cloud Sync = lighter (no on-prem sync service), agent-based, limited features (no device writeback, no Exchange hybrid management)

---

### 7.3 Azure Key Vault

#### Tiers

| Tier | Key Protection | Max Key Size | FIPS | Objects | Use Case | Cost |
|------|--------------|-------------|------|---------|---------|------|
| **Standard** | Software | RSA 2048/4096 | 140-2 L1 | Keys, Secrets, Certificates | Most apps | Low |
| **Premium** | HSM-protected | RSA 2048/4096/8192 | **140-2 L2** | Keys, Secrets, Certificates | Compliance (PCI-DSS, HIPAA) | Medium |
| **Managed HSM** | Dedicated HSM | RSA + EC | **140-2 L3** | Keys only | Strictest compliance (banking, gov) | Highest |

> **Soft-delete** = enabled by default; deleted items recoverable for 7–90 days
> **Purge protection** = prevents permanent deletion during retention; required for CMK encryption
> **Access models:** Vault Access Policy (legacy) vs **Azure RBAC** (recommended; use for new deployments)

---

### 7.4 Microsoft Defender for Cloud

| Plan | What It Protects | Key Capability | Cost |
|------|-----------------|---------------|------|
| **CSPM Free** | All Azure resources | Security score, basic recommendations | Free |
| **Defender for Servers P1** | Servers (Azure/on-prem/multi-cloud) | Microsoft Defender for Endpoint integration | Per server/hr |
| **Defender for Servers P2** | Servers | P1 + file integrity monitoring, JIT VM access, vulnerability scan | Higher |
| **Defender for SQL** | Azure SQL + SQL on VMs | Threat detection, VA scans | Per instance |
| **Defender for Storage** | Storage accounts | Malware scanning, sensitive data discovery | Per storage |
| **Defender for Containers** | AKS, container registries | Vulnerability scanning, runtime protection | Per core |
| **Defender for App Service** | App Service | Threat detection for web apps | Per ASP |
| **Defender for Key Vault** | Key Vault | Anomalous access detection | Per vault |
| **Defender CSPM (Paid)** | All resources | Attack path analysis, data security posture | Per resource |

> **JIT VM Access** = Defender for Servers P2; reduces attack surface by opening RDP/SSH ports only on request

---

## 8. MONITORING & MANAGEMENT

### 8.1 Azure Monitor — Component Overview

| Component | Purpose | Data Type | Default Retention | Cost |
|-----------|---------|----------|-----------------|------|
| **Metrics** | Numerical time-series (performance) | Metrics (numeric) | 93 days | Free (standard metrics) |
| **Activity Log** | Subscription-level audit trail of operations | Logs | 90 days (extend via LA) | Free |
| **Log Analytics Workspace** | Central store + KQL query engine | Logs | 30 days (up to 12 years) | Per GB ingested |
| **Application Insights** | APM — traces, dependencies, exceptions | Logs + Metrics | 90 days | Per GB (8 GB/workspace free) |
| **Diagnostic Settings** | Route resource logs to targets | Logs | Per destination | Per destination |

#### Log Analytics Pricing Tiers

| Tier | Model | Savings | Notes |
|------|-------|---------|-------|
| **Pay-as-you-go** | Per GB ingested | Baseline | Flexible, no commitment |
| **Commitment: 100 GB/day** | Fixed daily cap | ~15% | Commit to minimum daily ingestion |
| **Commitment: 500 GB/day…** | Fixed | Up to 30% | Higher volume = higher savings |
| **Sentinel Benefit** | Discounted | ~48% | Use when also running Microsoft Sentinel |

---

### 8.2 Alert Rule Types

| Type | Trigger | Example |
|------|---------|---------|
| **Metric Alert** | Metric crosses threshold | CPU > 80% for 5 min |
| **Log (KQL) Alert** | KQL query returns results | Count of 500 errors > 10 in 5 min |
| **Activity Log Alert** | Azure operation occurs | VM deleted, Policy assigned |
| **Smart Detection** | ML-based anomaly (App Insights) | Unusual failure rate spike |

> **Action Groups** = reusable targets for alerts (email, SMS, webhook, Azure Function, ITSM, Automation Runbook)

---

### 8.3 Azure Advisor — Recommendation Categories

| Category | Examples |
|----------|---------|
| **Cost** | Idle VMs, underutilized disks, reserved instance recommendations |
| **Security** | Enable MFA, rotate keys, enable Defender plans |
| **Reliability** | Add Availability Zones, enable backups, geo-replication |
| **Performance** | SSL offload, CDN, VM right-sizing |
| **Operational Excellence** | Enable diagnostics, use ARM templates |

---

## 9. BACKUP, DR & MIGRATION

### 9.1 Azure Backup

#### Vault Types

| Vault Type | Protects | Redundancy Options | Use |
|-----------|---------|------------------|-----|
| **Recovery Services Vault** | Azure VMs, SQL in VMs, Azure Files, SAP HANA, on-prem (MARS agent) | LRS, GRS, ZRS | Most workloads |
| **Backup Vault** | Azure Disks, Azure Blobs, Azure Database for PostgreSQL | LRS, GRS, ZRS | Newer workloads |

#### Workload RPO & Retention

| Workload | Min RPO | Max Retention | Backup Type |
|----------|---------|-------------|------------|
| Azure VM | 4 hours (daily policy) | 9,999 days | Agentless snapshot |
| SQL Server on VM | **15 minutes** | 99 years | Workload-aware log backup |
| Azure Files | 4 hours | 1 year | Share snapshot |
| Azure Blobs | **Continuous** (near-zero) | 360 days | Operational (storage-level) |
| PostgreSQL (Flexible Server) | Daily | 35 days | Service-managed |
| Azure Disks | 1 hour (up to 4/day) | 810 days | Crash-consistent snapshot |

#### Backup Redundancy Options (Recovery Services Vault)

| Option | Copies | Zone | Geo | Readable Secondary | Cost |
|--------|--------|------|-----|-------------------|------|
| **LRS** | 3 | ❌ | ❌ | ❌ | Lowest |
| **ZRS** | 3 | ✅ | ❌ | ❌ | Medium |
| **GRS** (default) | 6 | ❌ | ✅ | ❌ | Higher |

> **Cross-Region Restore (CRR)** = restore backup to secondary region; requires GRS vault

---

### 9.2 Azure Site Recovery (ASR)

#### Supported Scenarios

| Source → Target | RPO | RTO | Notes |
|----------------|-----|-----|-------|
| **Azure VM → Azure VM** (another region) | **30 seconds** | < 2 hours | Most common scenario |
| **VMware VMs → Azure** | 30 seconds | < 2 hours | Requires Mobility Service agent |
| **Hyper-V VMs → Azure** | 30 seconds | < 2 hours | Via Hyper-V Recovery Manager |
| **Physical servers → Azure** | 1 hour | < 2 hours | Via Mobility Service |

#### ASR Key Operations

| Operation | Description | Impact |
|-----------|-------------|--------|
| **Test Failover** | DR drill — uses isolated network | Non-disruptive; no production impact |
| **Planned Failover** | Graceful migration (e.g., moving workloads) | Zero data loss |
| **Unplanned Failover** | Emergency failover | Uses last recovery point; potential data loss |
| **Failback** | Return to original region after recovery | Requires reprotect first |
| **Reprotect** | Reverse replication direction after failover | Sets up return DR |

#### ASR Replication Points

| Type | Frequency | Use Case |
|------|-----------|---------|
| **Crash-consistent** | Every 5 minutes | OS + disk state (like a power cut) |
| **App-consistent** | Every 1–12 hours (configurable) | VSS snapshot; application-consistent data |

---

### 9.3 Azure Migrate

| Tool | Purpose | Supports |
|------|---------|---------|
| **Discovery & Assessment** | Scan on-prem inventory + size/cost estimate | VMware, Hyper-V, physical, AWS, GCP |
| **Server Migration** | Rehost VMs to Azure | VMware (agentless), Hyper-V, physical |
| **Database Migration** | Assess + migrate databases | SQL Server, MySQL, PostgreSQL, MongoDB |
| **Web App Migration** | Assess + migrate .NET/Java web apps | IIS on Windows Server |
| **Azure VMware Solution** | Migrate VMware workloads to AVS | VMware vSphere |

---

### 9.4 Azure Database Migration Service (DMS)

| Mode | Downtime | Use Case | Cost |
|------|---------|---------|------|
| **Offline (Standard tier)** | Business downtime required | Large one-time migrations | Free |
| **Online (Premium tier)** | Minimal downtime (CDC-based) | Mission-critical migrations | Paid |

#### Supported Migrations

| Source | Target |
|--------|-------|
| SQL Server | Azure SQL Database, SQL Managed Instance |
| MySQL | Azure Database for MySQL |
| PostgreSQL | Azure Database for PostgreSQL |
| MongoDB | Azure Cosmos DB for MongoDB |
| Oracle | Azure Database for PostgreSQL (via ora2pg) |

---

### 9.5 Azure Data Box Family (Offline Data Transfer)

| Product | Usable Capacity | Transfer Rate | Use Case |
|---------|----------------|--------------|---------|
| **Data Box Disk** | Up to 35 TB (5 × 8 TB SSDs) | USB 3.1 | Small datasets, single shipment |
| **Data Box** | 80 TB usable | 1 Gbps | Medium datasets |
| **Data Box Heavy** | ~770 TB usable | 40 GbE | Very large (petabyte-scale) datasets |
| **Data Box Gateway** | Unlimited (virtual) | Up to 2 Gbps | On-going cloud upload, edge analytics |

> Use Data Box when: **> 40 TB to transfer**, limited internet bandwidth, or compliance requires offline transfer
> Import: ship box → Microsoft loads to Azure Storage. Export: Microsoft ships data to you.

---

## 10. CROSS-SERVICE DECISION TABLES

### Compute: When to Use What

| Requirement | Best Service | Reason |
|-------------|-------------|--------|
| Full OS control, custom apps | Virtual Machine | Maximum flexibility |
| Burstable, variable CPU | B-series VM | Cost-efficient for variable load |
| Simple web app/API (no containers) | App Service | Zero infrastructure management |
| Simple single container (no orchestration) | Azure Container Instances | Easiest container deployment |
| Event-driven, short tasks (<10 min) | Azure Functions (Consumption) | Serverless, pay per execution |
| Event-driven + VNet + no cold start | Azure Functions (Premium) | Pre-warmed + VNet |
| Microservices, event-driven scale | Azure Container Apps | KEDA + Dapr; simpler than AKS |
| Complex K8s, full control | AKS | Full Kubernetes API |
| Large-scale parallel/HPC jobs | Azure Batch | Purpose-built job scheduler |
| Global burst scaling for AKS | AKS + Virtual Nodes (ACI burst) | Instant pod scaling via ACI |

---

### Database: Decision Matrix

| Requirement | Best Service |
|-------------|-------------|
| Relational, most PaaS benefits, new app | Azure SQL Database (vCore GP) |
| Relational + high IOPS + read replica | Azure SQL Database Business Critical |
| Very large relational DB (> 4 TB) | Azure SQL Database Hyperscale |
| Relational, lift & shift SQL Server | **Azure SQL Managed Instance** |
| 100% SQL Server control, specific version | SQL Server on Azure VM |
| NoSQL, global distribution, multi-model | **Azure Cosmos DB** |
| PostgreSQL open-source, production | Azure DB for PostgreSQL Flexible Server |
| MySQL open-source, production | Azure DB for MySQL Flexible Server |
| Sub-millisecond caching, session state | Azure Cache for Redis |
| Data warehouse, large-scale analytics | Azure Synapse (Dedicated SQL Pool) |
| Ad-hoc analytics over Data Lake | Azure Synapse (Serverless SQL Pool) |
| Semi-structured analytics + ML | Azure Databricks |

---

### Storage: Redundancy Decision

| Requirement | Choose |
|-------------|--------|
| Lowest cost, single-region | LRS |
| Single-region protection against zone failure | ZRS |
| Geo-redundancy, no secondary reads needed | GRS |
| Geo + zone redundancy, no secondary reads | GZRS |
| Geo-redundancy + read from secondary region | RA-GRS |
| Best protection (geo + zone + secondary reads) | RA-GZRS |

---

### Load Balancing: Decision Tree

| Need | Service |
|------|---------|
| Global HTTP/HTTPS + WAF + CDN + bot protection | Azure Front Door Premium |
| Global HTTP/HTTPS + WAF + CDN | Azure Front Door Standard |
| Global non-HTTP / any protocol (DNS routing) | Azure Traffic Manager |
| Regional HTTP/HTTPS + WAF + SSL offload | Application Gateway WAF v2 |
| Regional HTTP/HTTPS, no WAF | Application Gateway Standard v2 |
| Regional TCP/UDP (Layer 4) | Azure Load Balancer Standard |

---

### Networking Connectivity Decision

| Scenario | Solution |
|----------|---------|
| Azure VNet ↔ Azure VNet, same/different region | VNet Peering |
| On-prem ↔ Azure, small/medium, internet-based | VPN Gateway |
| On-prem ↔ Azure, high bandwidth, private, SLA-backed | ExpressRoute |
| On-prem ↔ Azure, high bandwidth + failover | ExpressRoute + VPN (coexistence) |
| Hub-and-spoke at scale, many branch sites | Azure Virtual WAN (Standard) |
| PaaS service with private IP in your VNet | Private Endpoint |
| PaaS service with optimized route (still public IP) | Service Endpoint |
| Stop RDP/SSH public IP exposure on VMs | Azure Bastion |

### Migration Strategy (6 Rs)

| Strategy | Effort | Description | Azure Tool |
|----------|--------|-------------|-----------|
| **Rehost** (Lift & Shift) | Low | Move to Azure IaaS as-is | Azure Migrate |
| **Replatform** | Medium | Minor cloud optimizations (e.g., SQL DB instead of SQL on VM) | DMS, App Service Migration |
| **Refactor** | High | Re-architect to cloud-native (microservices, containers) | Custom |
| **Repurchase** | Medium | Switch to SaaS product (e.g., Dynamics 365) | — |
| **Retire** | Low | Decommission (no longer needed) | — |
| **Retain** | None | Keep on-prem (not ready or not worth migrating) | — |

---

## 11. SLA QUICK REFERENCE

### SLA by Service

| Service / Configuration | SLA |
|------------------------|-----|
| Single VM (Premium SSD) | 99.9% |
| Single VM (Standard SSD) | 99.5% |
| 2+ VMs in Availability Set | 99.95% |
| 2+ VMs across Availability Zones | 99.99% |
| Azure SQL Database (all tiers) | 99.99% |
| Azure SQL Managed Instance | 99.99% |
| Azure Cosmos DB (single region) | 99.99% |
| Azure Cosmos DB (multi-write, multi-region) | 99.999% |
| Azure Blob/Table/Queue Storage (RA-GRS) | 99.99% read |
| Azure Blob Storage (LRS/ZRS/GRS) | 99.9% |
| Azure Load Balancer Standard | 99.99% |
| Azure Application Gateway v2 | 99.95% |
| Azure Front Door | 99.99% |
| Azure Traffic Manager | 99.99% |
| Azure Firewall | 99.95% |
| Azure VPN Gateway (active-active) | 99.95% |
| Azure ExpressRoute circuit | 99.95% |
| Entra ID (Entra P1/P2) | 99.9% |
| Azure Key Vault | 99.99% |
| Azure Backup | 99.9% |
| Azure App Service (Standard+) | 99.95% |
| Azure Functions (Premium) | 99.95% |
| Azure Kubernetes Service (Standard) | 99.9% (99.95% with AZs) |
| Azure Container Apps | 99.95% |
| Azure Event Hubs (Standard+) | 99.95% |
| Azure Service Bus | 99.9% |
| Azure API Management (Premium) | 99.95% |

---

### SLA Calculation Formulas

```
Composite SLA (serial dependency — both must work):
  Combined = SLA_A × SLA_B × SLA_C ...
  Example: App Service (99.95%) × SQL DB (99.99%) = 99.94%

Redundant SLA (parallel — either can work):
  Combined = 1 - (1 - SLA_A) × (1 - SLA_B)
  Example: Two Load Balancers each at 99.9%:
    = 1 - (0.001 × 0.001) = 99.9999%

Combined availability zones SLA:
  = 1 - (1 - single_zone_SLA)^N  (N = number of zones)
```

---

### Cost Ranking by Service Category (Low → High)

| Category | Low Cost → High Cost |
|----------|---------------------|
| **Compute** | Functions Consumption → ACI → App Service (Basic) → VMs → AKS → App Service (Isolated) |
| **SQL Database** | Basic DTU → Standard DTU → GP vCore → BC vCore → Hyperscale |
| **Cosmos DB** | Serverless → Provisioned Manual → Provisioned Autoscale |
| **Storage** | Archive → Cold → Cool → Hot |
| **Redis** | Basic → Standard → Premium → Enterprise → Enterprise Flash |
| **Networking** | Load Balancer → Traffic Manager → VPN Gateway → App Gateway → Front Door → ExpressRoute |
| **VPN Gateway** | Basic → VpnGw1 → VpnGw2 → VpnGw3 → VpnGw4 → VpnGw5 |
| **App Service** | Free → Shared → Basic → Standard → Premium → Isolated |

---

*Last updated: May 20, 2026 | Good luck tomorrow! 🎯*
