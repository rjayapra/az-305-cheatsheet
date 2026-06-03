# AZ-104 — Monitor and Maintain Azure Resources
## Domain 5 Deep-Dive Study Guide (10–15% of Exam)

---

## Table of Contents
1. [Azure Monitor — Overview & Architecture](#1-azure-monitor--overview--architecture)
2. [Metrics & Metrics Explorer](#2-metrics--metrics-explorer)
3. [Activity Log](#3-activity-log)
4. [Log Analytics & KQL](#4-log-analytics--kql)
5. [Diagnostic Settings](#5-diagnostic-settings)
6. [Azure Alerts & Action Groups](#6-azure-alerts--action-groups)
7. [Azure Backup — Recovery Services Vault](#7-azure-backup--recovery-services-vault)
8. [Azure Backup — VM Backup & Restore](#8-azure-backup--vm-backup--restore)
9. [Azure Backup — Files & SQL](#9-azure-backup--files--sql)
10. [Network Watcher — Diagnostics](#10-network-watcher--diagnostics)
11. [Azure Service Health](#11-azure-service-health)
12. [Azure Advisor](#12-azure-advisor)
13. [Exam Tips — Domain 5 Master List](#13-exam-tips--domain-5-master-list)

---

## 1. Azure Monitor — Overview & Architecture

### What It Is
Comprehensive monitoring platform that collects, analyzes, and acts on telemetry from Azure resources, applications, and infrastructure.

### Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    DATA SOURCES                                │
│  Azure Resources │ Applications │ OS (Guest) │ Custom Sources │
└────────────────────────────┬─────────────────────────────────┘
                             ▼
┌──────────────────────────────────────────────────────────────┐
│                    DATA PLATFORM                               │
│        Metrics            │        Logs                        │
│  (numeric, time-series)   │  (structured text, events)        │
│  → Metrics Explorer       │  → Log Analytics (KQL)            │
└────────────────────────────┬─────────────────────────────────┘
                             ▼
┌──────────────────────────────────────────────────────────────┐
│                    CONSUME & ACT                               │
│  Dashboards │ Alerts │ Workbooks │ Insights │ Power BI │ API │
└──────────────────────────────────────────────────────────────┘
```

### Data Types in Azure Monitor

| Data Type | Description | Storage | Retention |
|-----------|-------------|---------|-----------|
| **Metrics** | Numeric time-series data (CPU%, requests/sec) | Metrics database | 93 days (auto) |
| **Logs** | Structured/semi-structured events and traces | Log Analytics workspace | 30-730 days (configurable) |
| **Activity Log** | Control-plane operations (who did what) | Azure platform | 90 days (auto) |

### What Collects Data Automatically vs Requires Configuration

| Data | Collected Automatically | Requires Configuration |
|------|------------------------|----------------------|
| Platform metrics (CPU, memory, network) | ✅ | — |
| Activity Log | ✅ | — |
| Resource logs (diagnostics) | ❌ | Diagnostic Settings |
| Guest OS metrics | ❌ | Azure Monitor Agent |
| Application telemetry | ❌ | Application Insights SDK |
| Custom metrics | ❌ | API / Agent |

---

## 2. Metrics & Metrics Explorer

### What It Is
Lightweight numerical data collected at regular intervals. Ideal for real-time alerting and dashboards.

### Key Characteristics
| Property | Detail |
|----------|--------|
| **Format** | Time-stamped numerical values |
| **Collection** | Every 1 minute (most Azure resources) |
| **Retention** | 93 days automatically |
| **Near real-time** | Available within 1-3 minutes |
| **Dimensions** | Break down by category (e.g., CPU by instance) |

### Common Metrics

| Resource | Metric | Description |
|----------|--------|-------------|
| **VM** | Percentage CPU | CPU utilization |
| **VM** | Network In/Out Total | Bytes transferred |
| **VM** | Disk Read/Write Operations/Sec | IOPS |
| **Storage** | Transactions | Number of requests |
| **Storage** | UsedCapacity | Data stored |
| **App Service** | Http5xx | Server errors |
| **App Service** | ResponseTime | Average response time |
| **Load Balancer** | HealthProbeStatus | Backend health |
| **SQL Database** | DTU percentage | Database resource usage |

### Metrics Explorer
- Interactive charting tool in Azure Portal
- Pin charts to dashboards
- Split by dimension (e.g., per-instance CPU)
- Aggregation types: Average, Min, Max, Sum, Count

### Using Metrics via CLI

```bash
# List available metrics for a VM
az monitor metrics list-definitions \
  --resource "/subscriptions/<sub>/resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachines/MyVM" \
  --output table

# Query CPU metric
az monitor metrics list \
  --resource "/subscriptions/<sub>/resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachines/MyVM" \
  --metric "Percentage CPU" \
  --interval PT5M \
  --aggregation Average \
  --start-time 2026-05-24T00:00:00Z \
  --end-time 2026-05-24T12:00:00Z
```

---

## 3. Activity Log

### What It Is
Records all control-plane operations (management operations) against Azure resources.

### What's Recorded

| Information | Example |
|-------------|---------|
| **Who** initiated the operation | user@contoso.com |
| **What** operation was performed | Create/Update Virtual Machine |
| **When** it occurred | Timestamp |
| **Status** | Succeeded, Failed, Started |
| **Properties** | Resource ID, subscription, correlation ID |

### Event Categories

| Category | Description | Examples |
|----------|-------------|---------|
| **Administrative** | CRUD operations on resources | VM created, NSG rule added |
| **Service Health** | Azure service issues | Region outage, planned maintenance |
| **Resource Health** | Health state of your resources | VM unavailable |
| **Alert** | Alert activations | Alert fired, alert resolved |
| **Autoscale** | Scale events | VMSS scaled out |
| **Recommendation** | Azure Advisor recommendations | Right-size VM |
| **Security** | Microsoft Defender alerts | Suspicious login detected |
| **Policy** | Azure Policy operations | Resource denied by policy |

### Activity Log Retention & Export

| Destination | Retention | Use Case |
|-------------|-----------|----------|
| **Azure Portal** (built-in) | 90 days | Quick lookup |
| **Log Analytics Workspace** | Up to 730 days (or unlimited with archive) | Long-term queries, correlation |
| **Storage Account** | Custom (indefinite) | Compliance, archival |
| **Event Hub** | Real-time streaming | SIEM integration, third-party tools |

```bash
# Export Activity Log to Log Analytics
az monitor diagnostic-settings create \
  --name "ActivityLogToLA" \
  --resource "/subscriptions/<sub-id>" \
  --workspace <workspace-id> \
  --logs '[{"category": "Administrative", "enabled": true}]'
```

### Activity Log Queries (Portal)
- Filter by: Timespan, Subscription, Resource Group, Resource, Operation, Level (Info/Warning/Error)
- "Change history" tab shows exact property changes (before/after values)

---

## 4. Log Analytics & KQL

### What It Is
Central workspace for collecting, storing, and querying log data using Kusto Query Language (KQL).

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Workspace** | Container for log data (one or more per environment) |
| **Tables** | Log data organized in tables (e.g., Perf, Event, AzureDiagnostics) |
| **Retention** | Default 30 days; configurable up to 730 days (archive for longer) |
| **Ingestion** | Data from diagnostic settings, agents, APIs |
| **KQL** | Query language for exploring and analyzing logs |

### Common Tables

| Table | Contains |
|-------|----------|
| **AzureActivity** | Activity Log events (when exported to workspace) |
| **AzureDiagnostics** | Resource diagnostic logs (legacy, multi-resource) |
| **Perf** | Performance counters from agents |
| **Event** | Windows Event Log entries |
| **Syslog** | Linux syslog messages |
| **Heartbeat** | Agent health check (proves agent is alive) |
| **InsightsMetrics** | Metrics from Azure Monitor Agent |
| **SecurityEvent** | Windows security audit events |
| **VMConnection** | VM network connection data |

### KQL Essentials

```kusto
// Basic query — all records from a table
AzureActivity
| take 10

// Filter by time
AzureActivity
| where TimeGenerated > ago(24h)

// Filter by condition
AzureActivity
| where OperationNameValue == "MICROSOFT.COMPUTE/VIRTUALMACHINES/WRITE"
| where ActivityStatusValue == "Success"

// Count operations by caller
AzureActivity
| where TimeGenerated > ago(7d)
| summarize count() by Caller
| order by count_ desc

// VM CPU performance
Perf
| where ObjectName == "Processor"
| where CounterName == "% Processor Time"
| where InstanceName == "_Total"
| summarize AvgCPU = avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
| render timechart

// Find errors
AzureDiagnostics
| where Level == "Error"
| where TimeGenerated > ago(1h)
| project TimeGenerated, ResourceId, OperationName, Message
| order by TimeGenerated desc
```

### Key KQL Operators

| Operator | Purpose | Example |
|----------|---------|---------|
| `where` | Filter rows | `where CPU > 80` |
| `summarize` | Aggregate | `summarize count() by Computer` |
| `project` | Select columns | `project TimeGenerated, Computer` |
| `extend` | Add calculated column | `extend Duration = EndTime - StartTime` |
| `order by` | Sort results | `order by count_ desc` |
| `top` | Get top N rows | `top 10 by CPU` |
| `render` | Visualization | `render timechart` |
| `join` | Combine tables | `join kind=inner Table2 on Key` |
| `bin()` | Time bucketing | `bin(TimeGenerated, 1h)` |
| `ago()` | Relative time | `ago(24h)`, `ago(7d)` |
| `count()` | Count rows | `summarize count()` |
| `avg()` | Average value | `summarize avg(CPU)` |
| `max()`, `min()` | Max/min | `summarize max(ResponseTime)` |

### Creating Log Analytics Workspace

```bash
az monitor log-analytics workspace create \
  --resource-group MyRG \
  --workspace-name MyWorkspace \
  --location eastus \
  --retention-time 90
```

---

## 5. Diagnostic Settings

### What It Is
Configuration that routes resource-level logs and metrics to destinations for analysis and storage.

### Key Facts
| Property | Detail |
|----------|--------|
| **Per-resource** | Must configure on EACH resource individually |
| **Not enabled by default** | Resource logs are NOT collected until you create diagnostic settings |
| **Multiple destinations** | Can send to multiple destinations simultaneously |
| **Categories** | Each resource type has different log categories |

### Destinations

| Destination | Use Case | Retention |
|-------------|----------|-----------|
| **Log Analytics Workspace** | Query with KQL, correlate with other data | 30-730 days |
| **Storage Account** | Long-term archival, compliance | Configurable (indefinite) |
| **Event Hub** | Stream to SIEM, third-party tools | Real-time (no retention) |
| **Partner Solution** | Datadog, Elastic, etc. | Varies |

### Common Resource Diagnostic Categories

| Resource | Log Categories |
|----------|---------------|
| **Storage Account** | StorageRead, StorageWrite, StorageDelete |
| **Key Vault** | AuditEvent |
| **SQL Database** | SQLInsights, AutomaticTuning, QueryStoreRuntimeStatistics |
| **App Service** | AppServiceHTTPLogs, AppServiceConsoleLogs, AppServiceAppLogs |
| **Load Balancer** | LoadBalancerAlertEvent, LoadBalancerProbeHealthStatus |
| **NSG** | NetworkSecurityGroupEvent, NetworkSecurityGroupRuleCounter |
| **VPN Gateway** | GatewayDiagnosticLog, TunnelDiagnosticLog, IKEDiagnosticLog |

### Creating Diagnostic Settings

```bash
# Enable diagnostics on a storage account → send to Log Analytics
az monitor diagnostic-settings create \
  --name "StorageDiag" \
  --resource <storage-account-resource-id> \
  --workspace <workspace-id> \
  --logs '[{"category": "StorageRead", "enabled": true}, {"category": "StorageWrite", "enabled": true}]' \
  --metrics '[{"category": "Transaction", "enabled": true}]'

# Enable diagnostics on a VM (requires Azure Monitor Agent for guest OS)
# For platform metrics/logs:
az monitor diagnostic-settings create \
  --name "VMDiag" \
  --resource <vm-resource-id> \
  --workspace <workspace-id> \
  --metrics '[{"category": "AllMetrics", "enabled": true}]'
```

### Azure Monitor Agent (AMA) vs Legacy Agents

| | Azure Monitor Agent (AMA) | Log Analytics Agent (MMA/OMS) | Diagnostics Extension |
|-|--------------------------|------------------------------|----------------------|
| Status | **Current** (recommended) | Deprecated (Aug 2024) | Limited use |
| Configuration | Data Collection Rules (DCR) | Workspace config | Extension settings |
| Multi-homing | ✅ | ✅ | ❌ |
| Platform | Windows + Linux | Windows + Linux | Windows + Linux |
| Metrics to Azure Monitor | ✅ | ❌ | ✅ |
| Logs to workspace | ✅ | ✅ | ❌ (storage only) |

---

## 6. Azure Alerts & Action Groups

### Alert Rule Components

```
Alert Rule = Signal + Condition + Action Group + Severity
```

| Component | Description |
|-----------|-------------|
| **Target resource** | What to monitor (VM, Storage, App Service, etc.) |
| **Signal** | What data to evaluate (metric, log, activity log) |
| **Condition** | Logic (threshold, frequency, aggregation) |
| **Action Group** | What to do when alert fires |
| **Severity** | 0 = Critical, 1 = Error, 2 = Warning, 3 = Informational, 4 = Verbose |
| **Alert state** | New → Acknowledged → Closed |

### Alert Types

| Type | Signal Source | Example | Evaluation |
|------|-------------|---------|-----------|
| **Metric Alert** | Metrics data | CPU > 80% for 5 min | Real-time (every 1-5 min) |
| **Log Alert** | KQL query on logs | Error count > 10 | Periodic (5 min - 24 hours) |
| **Activity Log Alert** | Activity Log events | VM deleted | Event-based (immediate) |
| **Service Health Alert** | Azure service status | Region outage | Event-based |
| **Resource Health Alert** | Your resource status | VM became unavailable | Event-based |

### Metric Alert Conditions

| Setting | Options |
|---------|---------|
| **Aggregation type** | Average, Minimum, Maximum, Total, Count |
| **Operator** | Greater than, Less than, Equal to, etc. |
| **Threshold** | Static value OR Dynamic (ML-based baseline) |
| **Aggregation granularity** | 1 min, 5 min, 15 min, 30 min, 1 hour |
| **Frequency of evaluation** | Every 1 min, 5 min, 15 min, 30 min, 1 hour |

### Dynamic Thresholds
- Machine-learning-based (learns resource behavior patterns)
- Adjusts automatically based on historical patterns
- Sensitivity: High (tight), Medium, Low (wide)
- Use when you don't know the "right" threshold value

### Action Groups

| Action Type | Description | Rate Limits |
|-------------|-------------|-------------|
| **Email** | Send email notification | 100 emails/hour |
| **SMS** | Text message | 1 per 5 minutes |
| **Voice call** | Phone call notification | 1 per 5 minutes |
| **Push notification** | Azure mobile app | No limit |
| **Azure Function** | Trigger serverless function | — |
| **Logic App** | Trigger workflow | — |
| **Webhook** | HTTP POST to external URL | — |
| **ITSM** | Create ticket in ServiceNow, etc. | — |
| **Automation Runbook** | Run Azure Automation runbook | — |

### Creating Alerts

```bash
# Create Action Group
az monitor action-group create \
  --resource-group MyRG \
  --name MyActionGroup \
  --short-name MAG \
  --action email admin admin@contoso.com

# Create Metric Alert (CPU > 80%)
az monitor metrics alert create \
  --resource-group MyRG \
  --name "HighCPU" \
  --scopes <vm-resource-id> \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --severity 2 \
  --action MyActionGroup

# Create Activity Log Alert (VM deleted)
az monitor activity-log alert create \
  --resource-group MyRG \
  --name "VMDeleted" \
  --condition category=Administrative and operationName=Microsoft.Compute/virtualMachines/delete \
  --action-group MyActionGroup
```

### Alert Processing Rules (formerly Action Rules)
- Apply processing logic to alerts WITHOUT modifying alert rules
- Suppress alerts during maintenance windows
- Add action groups to alerts matching a filter
- Scope: subscription, resource group, or specific resource

---

## 7. Azure Backup — Recovery Services Vault

### What It Is
Azure resource that stores backup data and manages backup/recovery operations.

### Key Properties

| Property | Detail |
|----------|--------|
| **Region** | Must be in same region as resources being backed up |
| **Redundancy** | LRS, GRS, or ZRS for vault storage |
| **Soft delete** | Enabled by default — retains data 14 days after deletion |
| **Cross-region restore** | Available for GRS vaults (restore to paired region) |
| **Multi-user authorization** | Require additional approval for critical operations |

### What Can Be Backed Up

| Workload | Vault Type | Agent/Extension |
|----------|-----------|----------------|
| **Azure VMs** | Recovery Services Vault | VM backup extension (auto-installed) |
| **Azure Files** | Recovery Services Vault | Share snapshots |
| **SQL in Azure VMs** | Recovery Services Vault | SQL backup extension |
| **SAP HANA in VMs** | Recovery Services Vault | HANA backup extension |
| **Azure Blobs** | Backup Vault | Operational backup |
| **Azure Disks** | Backup Vault | Disk backup |
| **On-prem files** | Recovery Services Vault | MARS agent |
| **On-prem workloads** | Recovery Services Vault | MABS / DPM |

### Recovery Services Vault vs Backup Vault

| | Recovery Services Vault | Backup Vault |
|-|------------------------|-------------|
| Workloads | VMs, SQL, Files, MARS, DPM | Blobs, Disks, PostgreSQL |
| Technology | Older, comprehensive | Newer, specific workloads |
| Management | Azure Backup | Azure Backup |

### Creating Recovery Services Vault

```bash
# Create vault
az backup vault create \
  --resource-group MyRG \
  --name MyRecoveryVault \
  --location eastus

# Set storage redundancy
az backup vault backup-properties set \
  --resource-group MyRG \
  --name MyRecoveryVault \
  --backup-storage-redundancy GeoRedundant
```

### Vault Redundancy Options

| Option | Copies | Protection |
|--------|--------|-----------|
| **LRS** | 3 (same datacenter) | Hardware failure |
| **ZRS** | 3 (across zones) | Zone failure |
| **GRS** | 6 (2 regions) | Regional failure + cross-region restore |

> GRS is recommended for production; enables Cross Region Restore

---

## 8. Azure Backup — VM Backup & Restore

### Backup Process
1. Backup extension takes **application-consistent snapshot** (VSS on Windows, file-consistent on Linux)
2. Snapshot stored locally for **instant restore** (configurable 1-5 days)
3. Data transferred to Recovery Services Vault (according to policy)
4. Recovery points created per policy retention

### Backup Consistency Types

| Type | Description | When |
|------|-------------|------|
| **Application-consistent** | App-aware (flushes memory, quiesces I/O) | Windows VM + VSS writers |
| **File-system consistent** | File system consistent (no app awareness) | Linux default |
| **Crash-consistent** | Disk state at time of snapshot | Fallback (if app/FS consistency fails) |

### Backup Policy Configuration

| Setting | Standard Policy | Enhanced Policy |
|---------|----------------|-----------------|
| Frequency | Once daily | Multiple times/day (every 4/6/8/12 hours) |
| Instant restore | 1-5 days | 1-30 days |
| Retention | Daily: 7-9999 | Same |
| Weekly | ✅ | ✅ |
| Monthly | ✅ | ✅ |
| Yearly | ✅ | ✅ |

### Restore Options

| Option | Description | Use Case |
|--------|-------------|----------|
| **Create new VM** | Deploy new VM from recovery point | Replace failed VM |
| **Restore disk** | Restore managed disk(s) to a storage account | Custom assembly |
| **Replace existing** | Swap disks on running VM | In-place update |
| **Cross-region restore** | Restore to paired region | DR event |
| **File recovery** | Mount recovery point as drive, copy files | Recover specific files |

### File-Level Recovery Process
1. Select recovery point
2. Download script from portal (connects iSCSI to snapshot)
3. Run script on any machine (mounts volumes)
4. Browse and copy needed files
5. Unmount volumes when done

### Backup Management Commands

```bash
# Enable backup
az backup protection enable-for-vm \
  --resource-group MyRG \
  --vault-name MyRecoveryVault \
  --vm MyVM \
  --policy-name DefaultPolicy

# Trigger immediate backup
az backup protection backup-now \
  --resource-group MyRG \
  --vault-name MyRecoveryVault \
  --container-name "IaasVMContainer;iaasvmcontainerv2;MyRG;MyVM" \
  --item-name "VM;iaasvmcontainerv2;MyRG;MyVM" \
  --retain-until 2026-06-24

# List recovery points
az backup recoverypoint list \
  --resource-group MyRG \
  --vault-name MyRecoveryVault \
  --container-name "IaasVMContainer;iaasvmcontainerv2;MyRG;MyVM" \
  --item-name "VM;iaasvmcontainerv2;MyRG;MyVM" \
  --output table

# Stop backup (retain data)
az backup protection disable \
  --resource-group MyRG \
  --vault-name MyRecoveryVault \
  --container-name "..." \
  --item-name "..." \
  --backup-management-type AzureIaasVM
```

---

## 9. Azure Backup — Files & SQL

### Azure Files Backup
| Property | Detail |
|----------|--------|
| **Method** | Share snapshots (stored in same storage account) |
| **Frequency** | Multiple times per day (every 4/6/8/12 hours) or daily |
| **Restore** | Full share or individual files/folders |
| **Location** | Same storage account (snapshots are incremental) |
| **Max snapshots** | 200 per share |
| **Recovery Services Vault** | Required for management and policy |

### SQL Server in Azure VMs
| Feature | Detail |
|---------|--------|
| **Auto-discovery** | Discovers SQL instances in registered VMs |
| **Full backup** | Daily or weekly (configurable) |
| **Differential** | Every 12 hours (default) |
| **Log backup** | Every 15 minutes (can configure down to 5 min) |
| **RPO** | As low as 15 minutes (log backup frequency) |
| **Restore** | Point-in-time to any second within retention |
| **No infra needed** | Agent on VM handles everything |

### MARS Agent (On-Prem Files)

| Feature | Detail |
|---------|--------|
| **What** | Microsoft Azure Recovery Services agent |
| **Installs on** | Windows Server/Client (physical or VM) |
| **Backs up** | Files, folders, system state |
| **Does NOT back up** | Bare-metal, application-level (use DPM/MABS for that) |
| **Schedule** | Up to 3 times per day |
| **Encryption** | AES-256 with passphrase (you set at registration) |

---

## 10. Network Watcher — Diagnostics

### Available Tools (Exam-Relevant)

| Tool | Question It Answers | How to Use |
|------|-------------------|-----------|
| **IP Flow Verify** | "Is this traffic allowed or blocked?" | Specify source/dest IP, port, protocol |
| **Next Hop** | "Where will this packet be routed?" | Specify source VM and destination IP |
| **Connection Troubleshoot** | "Can VM-A reach VM-B on port X?" | End-to-end connectivity test |
| **Effective Security Rules** | "What NSG rules actually apply to this NIC?" | Shows combined rules from all NSGs |
| **NSG Flow Logs** | "What traffic is flowing through my NSGs?" | Detailed traffic logs (allow/deny) |
| **Packet Capture** | "What's on the wire?" | Capture raw packets on VM |
| **Topology** | "What does my network look like?" | Visual diagram |
| **Connection Monitor** | "Is connectivity stable over time?" | Ongoing monitoring |

### IP Flow Verify
```bash
# Check if SSH from internet to VM is allowed
az network watcher test-ip-flow \
  --resource-group MyRG \
  --vm MyVM \
  --direction Inbound \
  --protocol TCP \
  --local 10.0.1.4:22 \
  --remote 203.0.113.50:12345

# Output: Access=Allow/Deny, RuleName=<which NSG rule>
```

### Next Hop
```bash
# Where does traffic to 8.8.8.8 go?
az network watcher show-next-hop \
  --resource-group MyRG \
  --vm MyVM \
  --source-ip 10.0.1.4 \
  --dest-ip 8.8.8.8

# Output: NextHopType=Internet/VirtualAppliance/VnetLocal/etc.
```

### Connection Troubleshoot
```bash
# Test TCP connectivity from VM to destination
az network watcher test-connectivity \
  --resource-group MyRG \
  --source-resource MyVM \
  --dest-address 10.0.2.4 \
  --dest-port 443

# Output: ConnectionStatus=Reachable/Unreachable, latency, hops
```

### NSG Flow Logs

| Property | Detail |
|----------|--------|
| **Storage** | Written to Storage Account (JSON) |
| **Format** | Version 1 (basic) or Version 2 (+ state, bytes, packets) |
| **Analysis** | Use **Traffic Analytics** for visual dashboards |
| **Requirements** | Network Watcher + Storage Account + NSG |
| **Cost** | Storage costs + Traffic Analytics costs (Log Analytics) |

### Traffic Analytics
- Built on top of NSG Flow Logs
- Requires Log Analytics workspace
- Provides: top talkers, traffic distribution, security threats, geo visualization
- Processing interval: 10 minutes or 60 minutes

---

## 11. Azure Service Health

### What It Is
Personalized dashboard showing Azure service issues, planned maintenance, and health advisories that affect YOUR resources.

### Components

| Component | Shows | Example |
|-----------|-------|---------|
| **Azure Status** | Global Azure-wide outages | East US storage outage (affects everyone) |
| **Service Health** | Issues affecting YOUR subscriptions/regions | VM service degradation in your region |
| **Resource Health** | Individual resource health | "Your VM is unavailable" |

### Service Health Event Types

| Type | Description | Action |
|------|-------------|--------|
| **Service issues** | Active outages affecting Azure services | Check impact, plan workaround |
| **Planned maintenance** | Upcoming maintenance events | Schedule around it |
| **Health advisories** | Changes requiring action (deprecations, etc.) | Update before deadline |
| **Security advisories** | Security-related notifications | Patch/update |

### Service Health Alerts
- Create alerts to be notified of issues in specific regions/services
- Use Action Groups (email, SMS, webhook) for notification
- Filter by: Service, Region, Event Type

```bash
# Create service health alert
az monitor activity-log alert create \
  --resource-group MyRG \
  --name "ServiceHealthAlert" \
  --condition category=ServiceHealth and properties.incidentType=Incident \
  --action-group MyActionGroup
```

### Resource Health
- Per-resource health status:
  - **Available**: Resource is healthy
  - **Unavailable**: Azure detected a problem
  - **Unknown**: Stopped receiving health signals
  - **Degraded**: Reduced performance detected
- Root Cause Analysis (RCA) provided after incidents resolve

---

## 12. Azure Advisor

### What It Is
Personalized best-practice recommendation engine across 5 categories.

### Recommendation Categories

| Category | Example Recommendations |
|----------|----------------------|
| **Reliability** | Enable soft delete on Key Vault, use managed disks, add Availability Zones |
| **Security** | Enable MFA, restrict storage network access, apply security updates |
| **Performance** | Right-size VMs, enable accelerated networking, optimize queries |
| **Cost** | Shut down underutilized VMs, buy reserved instances, delete unused resources |
| **Operational Excellence** | Set up service health alerts, tag resources, configure diagnostics |

### Key Actions with Advisor

| Action | Description |
|--------|-------------|
| **View recommendations** | Portal → Advisor → per category |
| **Postpone** | Dismiss for a period (7 days, 30 days, etc.) |
| **Dismiss** | Permanently remove recommendation |
| **Quick Fix** | One-click remediation (available for some) |
| **Alerts** | Get notified of new recommendations |

### Advisor Alerts
```bash
# Create Advisor alert for new cost recommendations
az advisor configuration show  # View current settings
```

---

## 13. Exam Tips — Domain 5 Master List

### Azure Monitor
- **Platform metrics auto-collected** — no configuration needed for CPU, network, disk
- **Resource logs (diagnostics) NOT collected by default** — must create diagnostic settings per resource
- **Activity Log = 90 days** — send to Log Analytics for longer retention
- **Azure Monitor Agent (AMA)** is the current recommended agent (replaces MMA/OMS)
- **Metrics = 93 days retention** (automatic, cannot change)
- **Logs = configurable** (30-730 days in workspace, then archive)

### Alerts
- **Metric alerts** = near real-time (1-5 min evaluation)
- **Log alerts** = periodic (5 min to 24 hours, based on query)
- **Activity log alerts** = event-based (fires when event occurs)
- **Severity levels:** 0=Critical, 1=Error, 2=Warning, 3=Info, 4=Verbose
- **Rate limits:** SMS/Voice = 1 per 5 min; Email = 100/hour
- **Dynamic thresholds** use ML — good when you don't know the "right" threshold
- **Alert processing rules** suppress during maintenance windows

### Log Analytics & KQL
- **KQL is read-only** — cannot modify data with queries
- **`where`** = filter, **`summarize`** = aggregate, **`project`** = select columns
- **`ago(24h)`** = relative time, **`bin(TimeGenerated, 1h)`** = time bucketing
- **`render timechart`** = visualization
- **Tables you'll see:** AzureActivity, Perf, Event, Heartbeat, AzureDiagnostics

### Diagnostic Settings
- **Per-resource configuration** — not tenant-wide or subscription-wide
- **Multiple destinations** allowed simultaneously (workspace + storage + event hub)
- **Each resource has different log categories** — learn common ones
- **Diagnostic settings DON'T retroactively collect** — only from creation time forward

### Azure Backup
- **Vault must be in SAME REGION** as the resource being backed up
- **Soft delete = 14 additional days** after deletion (enabled by default)
- **Instant restore** — from local snapshot (fastest); configurable 1-5 days
- **Cross-region restore** — only available with GRS vaults
- **Cannot delete vault** with protected items (must stop protection first)
- **File recovery** — mount recovery point as drive, browse files
- **MARS agent** — files/folders only (not full VM backup)
- **VM backup is application-consistent** on Windows (VSS), file-consistent on Linux

### Network Watcher
- **IP Flow Verify** — tests NSG rules (tells you WHICH rule blocked/allowed)
- **Next Hop** — shows routing decision (Internet, VirtualAppliance, VNetLocal)
- **Connection Troubleshoot** — end-to-end connectivity test (reachable/unreachable + hops)
- **NSG Flow Logs** — stored in Storage Account; analyze with Traffic Analytics
- **Network Watcher agent extension** required for packet capture

### Service Health & Advisor
- **Service Health** = Azure issues affecting YOUR resources (personalized)
- **Azure Status** = global (everyone)
- **Resource Health** = individual resource state
- **Advisor** covers 5 pillars: Reliability, Security, Performance, Cost, Operational Excellence
- **Advisor recommendations** — can dismiss or postpone (but they come back)

---

*End of AZ-104 Domain Study Guides*
