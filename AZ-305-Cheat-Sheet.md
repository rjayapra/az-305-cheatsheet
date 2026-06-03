# AZ-305 Exam Cheat Sheet
> Exam: May 21, 2026 | Score to pass: 700/1000 | Duration: 120 min

---

## 🧭 EXAM STRATEGY
- **Design, not implement** — "Which service should you use?" not "How do you configure it?"
- **Eliminate 2 wrong answers first**, then choose between the remaining two
- **PaaS > IaaS** when requirements permit
- **Cheapest that meets SLA** — cost always matters
- **Least privilege** — minimum required permissions
- **Combined SLA = SLA₁ × SLA₂ × ...** (e.g., 99.9% × 99.9% = 99.8%)
- Flag hard questions; return at the end

---

## 🏆 DOMAIN WEIGHTS (Know Your Priorities)

| Domain | Weight | Key Focus |
|--------|--------|-----------|
| Infrastructure Solutions | **30–35%** | Compute, networking, migration |
| Identity, Governance & Monitoring | **25–30%** | Entra ID, RBAC, Policy, Monitor |
| Data Storage | **20–25%** | SQL vs Cosmos vs Blob, redundancy |
| Business Continuity | **15–20%** | HA/DR, Backup, ASR, RTO/RPO |

---

## 🔐 DOMAIN 1 — IDENTITY, GOVERNANCE & MONITORING

### Entra ID (Azure AD) Tiers
| Tier | Key Features |
|------|-------------|
| Free | Basic SSO, MFA, B2B |
| P1 | Conditional Access, Hybrid Identity, SSPR, Groups |
| P2 | **PIM**, Identity Protection, Access Reviews |

### Hybrid Identity: Choose the Right Tool
| Scenario | Solution |
|----------|----------|
| Sync on-prem AD to Entra ID | **Azure AD Connect (sync)** |
| Lightweight sync, no on-prem agent server | **Azure AD Connect Cloud Sync** |
| Federated authentication (ADFS required) | **AD FS with Azure AD Connect** |
| App needs on-prem AD Kerberos | **Azure AD Application Proxy** |
| Legacy apps needing domain services in cloud | **Azure AD Domain Services (AADDS)** |

> **AD Connect vs Cloud Sync:** Cloud Sync = lighter, agent-based, good for distributed forests; AD Connect = more features, more complex topologies

### PIM (Privileged Identity Management) - Entra P2
- Provides **just-in-time** (JIT) privileged access
- Requires **justification + approval** for activation
- **Access reviews** — periodic validation of privileged access
- Use for: Global Admin, Subscription Owner, Key Vault Admin

### Conditional Access (Entra P1+)
- **Policy = Assignments + Access Controls**
- Assignments: Users/Groups, Cloud Apps, Conditions (location, device, risk)
- Controls: Grant (require MFA, compliant device) OR Block
- **Named Locations** — trusted IP ranges
- **Continuous Access Evaluation (CAE)** — real-time policy enforcement

### RBAC
| Scope (broad → narrow) | Container |
|------------------------|-----------|
| Management Group | Multiple subscriptions |
| Subscription | Resources in that sub |
| Resource Group | Resources in that RG |
| Resource | Single resource |

Key RBAC roles:
- **Owner** — full access + assign permissions
- **Contributor** — full access, CANNOT assign permissions
- **Reader** — read-only
- **User Access Administrator** — manage access only, no resource access

### Management Group Hierarchy
```
Tenant Root Group
  └── Management Groups (up to 6 levels)
        └── Subscriptions
              └── Resource Groups
                    └── Resources
```
- Policy applied at MG scope **inherits down** to all children
- Max **6 levels** deep (excluding root and subscription)

### Azure Policy vs RBAC
| | Azure Policy | RBAC |
|--|-------------|------|
| Purpose | **What** can be deployed (compliance) | **Who** can do what (access) |
| Enforces | Resource properties/configurations | Actions/operations |
| Example | "All VMs must use managed disks" | "User can create VMs" |

### Azure Policy Key Concepts
- **Initiative (Policy Set)** — group of policies (e.g., ISO 27001)
- **Effect types:** Deny, Audit, Append, Modify, DeployIfNotExists, AuditIfNotExists
- **Remediation tasks** — fix existing non-compliant resources
- **Exemption** — exclude scope from policy

### Azure Landing Zone
- Subscription-scale foundation for consistency
- Components: Management Groups + Policies + RBAC + Networking + Monitoring
- Platform subscriptions: Identity, Management, Connectivity
- Application Landing Zones: Corp (connected) / Online (internet-facing)

### Monitoring Stack
| Tool | Use For |
|------|---------|
| **Azure Monitor** | Central platform — collects metrics & logs |
| **Log Analytics Workspace** | Store & query logs via KQL |
| **Application Insights** | APM — app performance, traces, exceptions |
| **Azure Monitor Alerts** | Trigger on metric/log/activity conditions |
| **Diagnostic Settings** | Route resource logs → Log Analytics / Storage / Event Hub |
| **Azure Advisor** | Best practice recommendations (cost, security, HA) |
| **Microsoft Defender for Cloud** | Security posture + threat protection |

> **Application Insights** connects to a **Log Analytics Workspace** (workspace-based mode is current best practice)

---

## 💾 DOMAIN 2 — DATA STORAGE

### Relational Database Decision Tree
```
Need full SQL Server compatibility? 
  YES → SQL Managed Instance (lift and shift)
  NO → Need elastic scaling?
    YES → Azure SQL Database (Hyperscale or Elastic Pool)
    NO → SQL Database (General Purpose / Business Critical)
```

### Azure SQL Tiers
| Tier | Use Case | Key Feature |
|------|----------|-------------|
| **General Purpose** | Standard workloads | 5 9s in BC tier |
| **Business Critical** | High IOPS, read replicas | Built-in read replica |
| **Hyperscale** | Large DBs up to 100 TB | Fast backup/restore |
| **Serverless** | Intermittent workloads | Auto-pause, cost savings |

### Cosmos DB Consistency Levels (Strongest → Weakest)
| Level | Description | Use Case |
|-------|-------------|----------|
| **Strong** | Linearizable, always up-to-date | Financial transactions |
| **Bounded Staleness** | Max lag by time/operations | Near-real-time global |
| **Session** ⭐ (default) | Consistent within session | Most web apps |
| **Consistent Prefix** | No out-of-order reads | Social feeds |
| **Eventual** | Lowest latency, max throughput | Non-critical data |

> Stronger consistency = higher latency + lower availability

### Cosmos DB APIs
| API | Best For |
|-----|---------|
| NoSQL (Core) | JSON documents, default choice |
| MongoDB | Existing MongoDB apps |
| Cassandra | Wide-column, time-series |
| Gremlin | Graph databases |
| Table | Key-value (replaces Azure Table Storage) |

### Cosmos DB Key Facts
- **Partition key** — critical for performance; high cardinality, even distribution
- **RU/s** — Request Units per second; 1 RU = read 1 KB document
- **Multi-write regions** = higher availability, conflict resolution needed
- **Automatic indexing** by default on all properties
- **TTL** — Time-to-live for automatic document expiry

### Storage Account Types & Redundancy
| Redundancy | Copies | Zones | Regions | Read Access |
|-----------|--------|-------|---------|-------------|
| **LRS** | 3 | 1 zone | 1 region | Primary only |
| **ZRS** | 3 | 3 zones | 1 region | Primary only |
| **GRS** | 6 | 1 zone | 2 regions | Primary only |
| **GZRS** | 6 | 3 zones | 2 regions | Primary only |
| **RA-GRS** | 6 | 1 zone | 2 regions | Primary + Secondary |
| **RA-GZRS** | 6 | 3 zones | 2 regions | Primary + Secondary |

> **RA-GRS/RA-GZRS** = use when you need to READ from secondary region

### Blob Storage Access Tiers
| Tier | Access | Cost | Use Case |
|------|--------|------|---------|
| **Hot** | Milliseconds | High storage, low access | Frequently accessed |
| **Cool** | Milliseconds | Lower storage, higher access | Infrequently accessed (30-day min) |
| **Cold** | Milliseconds | Even lower storage | Rarely accessed (90-day min) |
| **Archive** | Hours (rehydrate) | Lowest storage, highest access | Compliance/backup (180-day min) |

### Data Platform Selection
| Need | Service |
|------|---------|
| SQL analytics + big data | **Azure Synapse Analytics** |
| Big data storage (HDFS-compatible) | **Azure Data Lake Storage Gen2** |
| Real-time streaming | **Azure Event Hubs** or **Azure Stream Analytics** |
| ETL/ELT pipelines | **Azure Data Factory** |
| ML/Analytics notebooks | **Azure Databricks** |
| Caching (sub-millisecond) | **Azure Cache for Redis** |
| Search | **Azure AI Search** (formerly Cognitive Search) |

### SQL Managed Instance vs SQL Database
| | SQL Managed Instance | SQL Database |
|-|---------------------|-------------|
| SQL Agent | ✅ Yes | ❌ No |
| CLR | ✅ Yes | ❌ Limited |
| Linked Servers | ✅ Yes | ❌ No |
| Cross-DB queries | ✅ Yes | ❌ No |
| VNet deployment | ✅ Always | ✅ Optional |
| Lift & Shift | ✅ Best option | ❌ Rework needed |
| Auto-scaling | ❌ Manual | ✅ Serverless/Elastic |

---

## 🔄 DOMAIN 3 — BUSINESS CONTINUITY

### RTO vs RPO
| Term | Definition | Measures |
|------|-----------|---------|
| **RTO** (Recovery Time Objective) | Max downtime tolerable | How FAST to recover |
| **RPO** (Recovery Point Objective) | Max data loss tolerable | How much DATA can be lost |

> Lower RTO/RPO = MORE expensive. Design to meet requirements, not exceed them.

### HA Availability Options
| Option | SLA | Protection |
|--------|-----|-----------|
| Single VM (Premium SSD) | 99.9% | None |
| Availability Set (2+ VMs) | 99.95% | Rack/power failure |
| Availability Zone (2+ VMs) | 99.99% | Datacenter failure |
| Multi-region | 99.99%+ | Regional failure |

> **Availability Set** = Fault Domains (physical racks) + Update Domains (planned maintenance)
> **Availability Zone** = physically separate datacenters within a region

### Azure Backup — What It Backs Up
| Service | Backup Vault | Key Setting |
|---------|-------------|-------------|
| Azure VMs | Recovery Services Vault | Policy: frequency + retention |
| Azure SQL Database | Auto-backup built-in | Point-in-time restore |
| Azure Files | Recovery Services Vault | Share snapshots |
| On-prem workloads | Recovery Services Vault via MARS/DPM agent | |
| Blobs | Backup Vault (operational) | Continuous backup |

### Azure Site Recovery (ASR)
- **Disaster Recovery** for VMs + physical servers
- Supported scenarios: Azure → Azure, On-prem → Azure, VMware → Azure, Hyper-V → Azure
- **RPO: as low as 30 seconds**
- **RTO: typically < 2 hours** (dependent on failover time)
- **Test failover** — non-disruptive DR drills using isolated network
- **Replication policy** — configures RPO threshold and recovery point retention

### DR Tier Comparison
| Tier | Strategy | Cost | RTO | RPO |
|------|---------|------|-----|-----|
| **Cold** | Pilot Light / Backup-Restore | Lowest | Hours-Days | Hours |
| **Warm** | ASR / Standby | Medium | Minutes-Hours | Minutes |
| **Hot** | Active-Active multi-region | Highest | Seconds | Near-zero |

### Multi-Region Deployment Patterns
| Pattern | Description | Use Case |
|---------|-------------|---------|
| **Active-Active** | Traffic in both regions simultaneously | Highest availability, zero RPO |
| **Active-Passive (Hot Standby)** | Secondary ready, no traffic | Fast failover |
| **Active-Passive (Cold Standby)** | Secondary not running | Lower cost |
| **Pilot Light** | Minimal core running in DR | Balance of cost/speed |

### SQL High Availability
| Feature | Scope | Provides |
|---------|-------|---------|
| **Active Geo-Replication** | SQL Database only | Readable secondary in another region |
| **Auto-Failover Group** | SQL DB + Managed Instance | Automatic failover, single connection string |
| **Readable Secondary** | Business Critical tier | Built-in, same region |

---

## 🏗️ DOMAIN 4 — INFRASTRUCTURE SOLUTIONS

### Compute Decision Tree
```
Web app / API?
  → Static? → Azure Static Web Apps
  → Dynamic, simple → App Service (PaaS)
  → Containerized, need orchestration → AKS
  → Containerized, serverless → Azure Container Apps
  → Event-triggered, short-running → Azure Functions
  → Need full OS control / custom OS → Virtual Machines
  → Batch/HPC jobs → Azure Batch
  → ML training → Azure Machine Learning Compute
```

### App Service Plans
| Tier | Features | Use Case |
|------|---------|---------|
| Free / Shared | No SLA, shared infra | Dev/Test |
| Basic | Dedicated, manual scale | Light production |
| Standard | Auto-scale, slots, TLS | Production |
| **Premium** | VNet integration, isolated | Secure production |
| **Isolated** | ASE (App Service Environment) | Full isolation, VNet |

### AKS vs Container Apps vs Functions
| | AKS | Container Apps | Azure Functions |
|-|-----|----------------|-----------------|
| Control | Full K8s | Managed K8s | None (serverless) |
| Scaling | Manual/HPA/KEDA | KEDA/HTTP | Trigger-based |
| Networking | Full VNet | VNet injection | VNet integration |
| Cost Model | Node pools (always on) | CPU/memory used | Consumption or Premium |
| Best For | Complex microservices | Event-driven containers | Lightweight event tasks |

### Virtual Machine Key Concepts
| Feature | Description |
|---------|-------------|
| **VMSS** | Scale set — auto-scale group of identical VMs |
| **Spot VMs** | Low cost, can be evicted — batch/fault-tolerant only |
| **Reserved Instances** | 1 or 3 year commitment — up to 72% savings |
| **Azure Dedicated Host** | Physical server for compliance/licensing |
| **Proximity Placement Group** | Low latency between VMs |

### Networking Decision Tree — Load Balancing
```
Global traffic?
  → HTTP/HTTPS? → Azure Front Door (with WAF)
  → Any protocol? → Azure Traffic Manager (DNS-based)
Regional traffic?
  → HTTP/HTTPS? → Application Gateway (with WAF)
  → Any protocol? → Azure Load Balancer (Layer 4)
```

### Load Balancer Comparison
| Service | Scope | Protocol | WAF | SSL Offload | Routing |
|---------|-------|----------|-----|-------------|---------|
| **Front Door** | Global | HTTP/S | ✅ | ✅ | URL, header, geo |
| **Traffic Manager** | Global | Any (DNS) | ❌ | ❌ | DNS-based |
| **App Gateway** | Regional | HTTP/S | ✅ | ✅ | URL path, host |
| **Load Balancer** | Regional | Any (L4) | ❌ | ❌ | IP/port hash |

> **Traffic Manager** does NOT inspect traffic — it's pure DNS routing
> **Front Door** replaces Traffic Manager + App Gateway for global HTTP

### VNet Connectivity Options
| Solution | Use Case | Notes |
|----------|---------|-------|
| **VNet Peering** | VNet-to-VNet, same/diff region | Low latency, no gateway needed |
| **VPN Gateway** | On-prem to Azure over internet | Up to 10 Gbps, encrypted |
| **ExpressRoute** | On-prem to Azure, private | High bandwidth, SLA-backed, expensive |
| **Virtual WAN** | Hub-and-spoke at scale | Managed hub, BGP routing |
| **Private Endpoint** | Private IP for PaaS | Service stays in VNet |
| **Service Endpoint** | Optimize route to PaaS | Not truly private, still public IP |

> **Private Endpoint > Service Endpoint** for security (traffic stays in VNet vs. optimized route to public endpoint)

### Hub-and-Spoke Architecture
```
Hub VNet (shared services)
  ├── Azure Firewall / NVA
  ├── VPN Gateway / ExpressRoute Gateway
  ├── Bastion Host
  └── DNS Resolver

Spoke VNets (workloads)
  ├── Spoke 1 — App A
  ├── Spoke 2 — App B
  └── Spoke 3 — App C

All spokes peer to Hub (not to each other)
Spoke-to-spoke traffic routes through Hub Firewall
```

### ExpressRoute vs VPN Gateway
| | ExpressRoute | VPN Gateway |
|-|-------------|-------------|
| Connection | Private (via provider) | Public internet (encrypted) |
| Bandwidth | Up to 100 Gbps | Up to 10 Gbps |
| Latency | Predictable, low | Variable |
| SLA | 99.95% | 99.9% |
| Cost | High | Lower |
| Failover | Pair with VPN for backup | ✅ Active-active option |

### Network Security
| Service | Function |
|---------|---------|
| **NSG** (Network Security Group) | L4 filter on subnet/NIC (allow/deny by port/IP) |
| **Azure Firewall** | Stateful L4-L7 firewall, FQDN filtering |
| **Web Application Firewall (WAF)** | OWASP-based HTTP protection (App GW / Front Door) |
| **DDoS Protection Standard** | Auto-mitigation of volumetric DDoS attacks |
| **Azure Bastion** | Browser-based SSH/RDP, no public IP on VMs |

### Migration Strategies (6 Rs)
| Strategy | Description | Effort | Azure Tool |
|----------|-------------|--------|-----------|
| **Rehost** (Lift & Shift) | Move as-is to IaaS | Low | Azure Migrate |
| **Replatform** | Minor cloud optimizations | Medium | App Service Migration |
| **Refactor** | Re-architect for cloud-native | High | — |
| **Repurchase** | Switch to SaaS | Medium | — |
| **Retire** | Decommission | Low | — |
| **Retain** | Keep on-prem for now | None | — |

### Azure Migration Tools
| Tool | Purpose |
|------|---------|
| **Azure Migrate** | Discovery + assessment + rehost VMs |
| **Azure Database Migration Service** | Database migrations to Azure SQL / Managed Instance |
| **Azure App Service Migration Assistant** | Assess + migrate .NET web apps |
| **Azure Data Box** | Offline bulk data transfer (physical device) |
| **Azure Import/Export** | Ship physical drives to Azure |

---

## ⚖️ WELL-ARCHITECTED FRAMEWORK (WAF) — 5 Pillars

| Pillar | Description | Key Azure Services |
|--------|-------------|-------------------|
| **Reliability** | HA, DR, resilience | Availability Zones, ASR, Azure Backup |
| **Security** | Protect data and systems | Defender for Cloud, Key Vault, PIM |
| **Cost Optimization** | Avoid waste, right-size | Advisor, Reserved Instances, Spot VMs |
| **Operational Excellence** | DevOps, monitoring, automation | Monitor, Pipelines, Automation |
| **Performance Efficiency** | Scale to meet demand | VMSS, Front Door, CDN, Redis |

---

## 🧮 SLA CALCULATIONS

```
Single VM (Premium SSD) = 99.9%
Two VMs in Availability Set = 99.95%
Two VMs in Availability Zones = 99.99%

Composite SLA = SLA_A × SLA_B
Example: App Service (99.95%) × SQL DB (99.99%) = 99.94%

To INCREASE composite SLA → use redundancy (parallel paths):
Combined = 1 - (1 - SLA_A) × (1 - SLA_B)
Example: Two LBs each at 99.9% → 1-(0.001×0.001) = 99.9999%
```

---

## 💲 COST OPTIMIZATION PATTERNS

| Strategy | Savings | Notes |
|----------|---------|-------|
| **Reserved Instances (1yr)** | ~40% | Commit to usage |
| **Reserved Instances (3yr)** | ~72% | More commitment |
| **Spot VMs** | up to 90% | Interruptible workloads only |
| **Hybrid Benefit (AHUB)** | ~40% for Windows/SQL | Own-license transfer |
| **Right-sizing** | Varies | Use Advisor recommendations |
| **Auto-shutdown** | High for dev/test | Dev VMs off-hours |
| **Storage tiering** | Significant | Move cold data to Cool/Archive |
| **Reserved Capacity for Cosmos** | Up to 65% | Predictable throughput |

---

## 🔑 KEY GOTCHAS & EXAM TRAPS

### Identity
- **Conditional Access needs P1** — not available on Free tier
- **PIM needs P2** — commonly forgotten
- **Azure AD B2C** is for external/customer identities (not employees)
- **Azure AD B2B** is for partner/guest user access
- **Cloud Sync** cannot sync all features (no device writeback, no password hash sync fine-tuning)

### Storage
- **Archive tier requires rehydration** (hours, not instant)
- **GRS secondary is NOT readable** — need RA-GRS for read access to secondary
- **Cosmos DB strong consistency not available** with multi-write regions (conflict)
- **LRS has 11 9s durability** but no zone/geo protection
- **Storage Account kind matters:** General Purpose v2 supports all tiers; Blob Storage supports only Blob

### Networking
- **VNet Peering is NOT transitive** — A↔B + B↔C does NOT mean A↔C (need hub or mesh)
- **Private Link vs Private Endpoint:** Private Endpoint = specific resource; Private Link = the service
- **ExpressRoute Global Reach** — connect on-prem sites through Azure backbone
- **UDR (User Defined Route)** — force traffic through NVA/Firewall

### Compute
- **App Service cannot use Spot VMs** — use VMSS or AKS for spot
- **Functions Consumption plan** — no VNet integration (need Premium or Dedicated)
- **Container Apps** — KEDA for event-driven scaling, Dapr for microservices patterns
- **Azure Batch** — for large-scale parallel/HPC jobs (not for web apps)

### Business Continuity
- **Azure Backup doesn't do zero RPO** — minimum 4-hour backup freq for Azure VMs
- **ASR RPO can be 30 seconds** (replication is near-continuous)
- **Geo-replication for SQL DB** = async (RPO ~ seconds); Active Geo-Replication = up to 4 secondaries
- **Auto-failover group** gives single read-write AND read-only connection strings (don't update connection strings on failover)

### Governance
- **Policy assignment scope** — can exclude child scopes via exclusions
- **Blueprint** creates a lock — "DoNotDelete" resources part of blueprint
- **Management Group policy** cannot be overridden by subscription-level policy within that scope

---

## 📋 QUICK REFERENCE — SERVICE LIMITS & KEY NUMBERS

| Item | Limit |
|------|-------|
| Management group levels | 6 levels (below root) |
| Subscriptions per management group | 10,000 |
| VMs per Availability Set | 200 |
| Fault domains per Availability Set | 2–3 |
| Update domains per Availability Set | 5 (default), up to 20 |
| VNet peerings per VNet | 500 |
| Cosmos DB max partition key size | 2048 bytes |
| SQL DB max size (Hyperscale) | 100 TB |
| Azure Backup VM retention max | 9999 days |
| Front Door backend pool regions | Global |
| Azure Function max timeout (Consumption) | 5 min default, 10 min max |
| Azure Function max timeout (Premium/Dedicated) | Unlimited |
| App Service slots (Standard) | 5 |
| App Service slots (Premium) | 20 |

---

## 🗺️ CASE STUDY — QUICK CHECKLIST

When reading a case study, identify:
1. **Scale requirements** → Compute tier, auto-scale
2. **Compliance/regulatory** → Dedicated Host, policies, encryption
3. **Data residency** → Region, data sovereignty
4. **Connectivity** → Public internet vs ExpressRoute vs VPN
5. **Existing on-prem footprint** → Hybrid, migration path
6. **Budget constraints** → Reserved, Spot, cheaper redundancy
7. **RTO/RPO requirements** → Backup, ASR, active-active
8. **Identity requirements** → Entra P1/P2, PIM, B2B/B2C
9. **Multi-team/subscription** → Management groups, RBAC
10. **Traffic patterns** → Load balancer choice, CDN, caching

---

## ✅ LAST-MINUTE MEMORIZATION LIST

- [ ] Load balancer selection (4 services, when to use each)
- [ ] Storage redundancy (LRS/ZRS/GRS/RA-GRS differences)
- [ ] Cosmos DB consistency levels (5 levels, weakest = eventual)
- [ ] SLA formula for composite + parallel redundancy
- [ ] RTO vs RPO definitions
- [ ] When to use SQL Managed Instance (lift & shift, full SQL Server compat)
- [ ] Conditional Access needs P1, PIM needs P2
- [ ] VNet Peering is NOT transitive
- [ ] Private Endpoint keeps traffic in VNet; Service Endpoint does NOT
- [ ] Auto-Failover Groups work for SQL DB AND SQL MI
- [ ] WAF 5 pillars

---

*Good luck on May 21! You've got this. 🎯*
