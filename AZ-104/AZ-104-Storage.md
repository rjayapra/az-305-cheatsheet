# AZ-104 — Implement and Manage Storage
## Domain 2 Deep-Dive Study Guide (15–20% of Exam)

---

## Table of Contents
1. [Storage Accounts — Overview & Types](#1-storage-accounts--overview--types)
2. [Storage Account Configuration](#2-storage-account-configuration)
3. [Storage Redundancy — Full Comparison](#3-storage-redundancy--full-comparison)
4. [Blob Storage — Containers & Types](#4-blob-storage--containers--types)
5. [Access Tiers & Lifecycle Management](#5-access-tiers--lifecycle-management)
6. [Azure Files](#6-azure-files)
7. [Azure File Sync](#7-azure-file-sync)
8. [Storage Security — Access Keys & SAS](#8-storage-security--access-keys--sas)
9. [Storage Security — Entra ID & Network](#9-storage-security--entra-id--network)
10. [AzCopy & Storage Explorer](#10-azcopy--storage-explorer)
11. [Import/Export & Data Box](#11-importexport--data-box)
12. [Soft Delete & Versioning](#12-soft-delete--versioning)
13. [Exam Tips — Domain 2 Master List](#13-exam-tips--domain-2-master-list)

---

## 1. Storage Accounts — Overview & Types

### What It Is
A container that groups Azure Storage services (Blob, Files, Queue, Table) under a single management endpoint and namespace.

### Storage Account Types

| Account Type | Supported Services | Performance Tiers | Redundancy Options | Use Case |
|-------------|-------------------|-------------------|-------------------|----------|
| **General-purpose v2 (GPv2)** | Blob, Files, Queue, Table, Data Lake | Standard, Premium | LRS, ZRS, GRS, GZRS, RA-GRS, RA-GZRS | **Default choice — supports all features** |
| **Premium Block Blob** | Block blobs, Append blobs | Premium only | LRS, ZRS | Low-latency, high transaction workloads |
| **Premium File Shares** | Azure Files only | Premium only | LRS, ZRS | High-performance file shares (NFS + SMB) |
| **Premium Page Blobs** | Page blobs only | Premium only | LRS | Unmanaged VM disks (legacy) |

> **Always choose GPv2** unless you specifically need Premium performance for a single service type

### Performance Tiers

| Tier | Backed By | Latency | Use Case |
|------|-----------|---------|----------|
| **Standard** | HDD | Milliseconds | General storage, backup, archive |
| **Premium** | SSD | Sub-millisecond | Databases, analytics, low-latency |

> Premium accounts do NOT support access tiers (Hot/Cool/Cold/Archive). Premium is always "hot."

---

## 2. Storage Account Configuration

### Naming Rules
| Rule | Detail |
|------|--------|
| Length | 3-24 characters |
| Characters | Lowercase letters and numbers only (no hyphens, no uppercase) |
| Uniqueness | **Globally unique** across all of Azure |

### Key Settings at Creation

| Setting | Options | Can Change Later? |
|---------|---------|-------------------|
| **Name** | Globally unique | ❌ No |
| **Region** | Azure region | ❌ No |
| **Performance** | Standard / Premium | ❌ No |
| **Redundancy** | LRS, ZRS, GRS, GZRS, RA-GRS, RA-GZRS | ✅ Partially (see limits) |
| **Access tier (default)** | Hot / Cool / Cold | ✅ Yes |
| **Enable hierarchical namespace** | Yes / No (enables Data Lake Gen2) | ❌ No |

### Redundancy Change Limitations
| From | Can Change To |
|------|---------------|
| LRS | GRS, RA-GRS (same region) |
| GRS/RA-GRS | LRS (same region) |
| ZRS | GZRS, RA-GZRS |
| GZRS/RA-GZRS | ZRS |
| LRS ↔ ZRS | ❌ Requires live migration or manual copy |

### Storage Account Endpoints
```
Blob:   https://<account>.blob.core.windows.net
Files:  https://<account>.file.core.windows.net
Queue:  https://<account>.queue.core.windows.net
Table:  https://<account>.table.core.windows.net
Data Lake: https://<account>.dfs.core.windows.net
```

### Creating Storage Accounts

```bash
# Azure CLI
az storage account create \
  --name mystorageacct2026 \
  --resource-group MyRG \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Hot

# PowerShell
New-AzStorageAccount -ResourceGroupName MyRG `
  -Name "mystorageacct2026" `
  -Location "eastus" `
  -SkuName "Standard_LRS" `
  -Kind "StorageV2" `
  -AccessTier "Hot"
```

### SKU Names for CLI/PowerShell
| Redundancy | Standard SKU | Premium SKU |
|-----------|-------------|-------------|
| LRS | Standard_LRS | Premium_LRS |
| ZRS | Standard_ZRS | Premium_ZRS |
| GRS | Standard_GRS | — |
| RA-GRS | Standard_RAGRS | — |
| GZRS | Standard_GZRS | — |
| RA-GZRS | Standard_RAGZRS | — |

---

## 3. Storage Redundancy — Full Comparison

### Redundancy Options

| Option | Copies | Zones | Regions | Read from Secondary | Durability |
|--------|--------|-------|---------|--------------------:|-----------|
| **LRS** | 3 | 1 | 1 | ❌ | 11 nines |
| **ZRS** | 3 | 3 | 1 | ❌ | 12 nines |
| **GRS** | 6 | 1+1 | 2 | ❌ (failover only) | 16 nines |
| **GZRS** | 6 | 3+1 | 2 | ❌ (failover only) | 16 nines |
| **RA-GRS** | 6 | 1+1 | 2 | ✅ | 16 nines |
| **RA-GZRS** | 6 | 3+1 | 2 | ✅ | 16 nines |

### Visual Breakdown

```
LRS:     [Copy1][Copy2][Copy3]  — single datacenter
ZRS:     [Zone1][Zone2][Zone3]  — 3 availability zones
GRS:     [DC-Primary: 3 copies] → [DC-Secondary: 3 copies]  — async replication
GZRS:    [Zone1][Zone2][Zone3] → [DC-Secondary: 3 copies]
RA-GRS:  Same as GRS + read endpoint for secondary
RA-GZRS: Same as GZRS + read endpoint for secondary
```

### Secondary Region Read Access

| Scenario | RA-GRS / RA-GZRS | GRS / GZRS |
|----------|-------------------|------------|
| Normal operation | Read from `-secondary` endpoint | ❌ Cannot read |
| After failover | Read from primary (now secondary region) | Read from primary |

**Secondary endpoint:**
```
https://<account>-secondary.blob.core.windows.net
```

### When to Use Each

| Requirement | Choose |
|-------------|--------|
| Cheapest, single DC OK | LRS |
| Zone protection, single region | ZRS |
| Cross-region protection, no read needed | GRS |
| Cross-region + zone, no read needed | GZRS |
| Cross-region + read from secondary | RA-GRS |
| Maximum protection + read secondary | RA-GZRS |

### Storage Account Failover
- **Customer-managed failover:** Manually initiate failover to secondary region
- After failover: secondary becomes new primary (LRS); original primary lost
- **RPO:** Typically minutes (async replication lag)
- Only available for GRS/GZRS/RA-GRS/RA-GZRS accounts

---

## 4. Blob Storage — Containers & Types

### Container Access Levels

| Level | Public Access | Use Case |
|-------|-------------|----------|
| **Private** (default) | ❌ No anonymous access | Most workloads (auth required) |
| **Blob** | Read access to blobs only (not list) | Public individual files |
| **Container** | Read + list all blobs | Public browsable content |

> **Best practice:** Keep containers Private; use SAS tokens for controlled access
> As of 2024, new storage accounts have **anonymous access disabled by default**

### Blob Types

| Type | Description | Scenarios | Max Size |
|------|-------------|-----------|----------|
| **Block Blob** | Composed of blocks; optimized for upload/download | Files, images, videos, backups | 190.7 TiB |
| **Append Blob** | Optimized for append operations | Log files, streaming data | 195 GiB |
| **Page Blob** | Random read/write access; 512-byte pages | VHD files (unmanaged disks) | 8 TiB |

### Block Blob Upload Methods
| Method | Best For | Notes |
|--------|----------|-------|
| **Put Blob** | Files ≤ 5,000 MiB | Single upload |
| **Put Block + Put Block List** | Files > 5,000 MiB | Upload in parallel blocks, then commit |
| **AzCopy** | Bulk uploads | Auto-handles chunking |
| **Portal drag-and-drop** | Quick ad-hoc uploads | Limited to file size browser supports |

### Blob Metadata & Properties
| Type | Description | Example |
|------|-------------|---------|
| **System properties** | Set by Azure (read-only or read-write) | Content-Type, Content-Length, Last-Modified |
| **User-defined metadata** | Custom key-value pairs | x-ms-meta-project: Alpha |

---

## 5. Access Tiers & Lifecycle Management

### Access Tiers

| Tier | Latency | Storage Cost | Access Cost | Min Retention | Availability SLA |
|------|---------|-------------|-------------|---------------|-----------------|
| **Hot** | ms | Highest | Lowest | None | 99.9% (RA: 99.99%) |
| **Cool** | ms | Lower | Higher | 30 days | 99% (RA: 99.9%) |
| **Cold** | ms | Even lower | Even higher | 90 days | 99% (RA: 99.9%) |
| **Archive** | Hours | Lowest | Highest | 180 days | Offline |

### Tier Scope
| Level | Hot | Cool | Cold | Archive |
|-------|-----|------|------|---------|
| **Account default** | ✅ | ✅ | ✅ | ❌ (blob-level only) |
| **Per-blob** | ✅ | ✅ | ✅ | ✅ |

> Archive can only be set at the **blob level**, not as account default

### Archive Tier — Rehydration

| Priority | Time | Cost |
|----------|------|------|
| **Standard** | Up to 15 hours | Lower |
| **High** | < 1 hour (not guaranteed) | Higher |

**Rehydration methods:**
1. **Change tier** — Move blob from Archive to Hot/Cool (in-place rehydration)
2. **Copy blob** — Copy to a new blob in Hot/Cool tier (keeps original in Archive)

### Early Deletion Penalty
- Delete or move a blob before minimum retention → charged for remaining days
- Cool: 30-day minimum, Cold: 90-day minimum, Archive: 180-day minimum
- Example: Delete a Cool blob after 10 days = charged for remaining 20 days

### Lifecycle Management Rules

| Component | Description |
|-----------|-------------|
| **Rule name** | Unique identifier |
| **Filters** | Blob type (block/append), prefix match, index tags |
| **Actions** | tierToCool, tierToCold, tierToArchive, delete |
| **Conditions** | daysAfterModificationGreaterThan, daysAfterCreationGreaterThan, daysAfterLastAccessTimeGreaterThan |

### Example Lifecycle Policy (JSON)

```json
{
  "rules": [
    {
      "name": "move-to-cool",
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["logs/"]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": { "daysAfterModificationGreaterThan": 30 },
            "tierToArchive": { "daysAfterModificationGreaterThan": 90 },
            "delete": { "daysAfterModificationGreaterThan": 365 }
          }
        }
      }
    }
  ]
}
```

### Setting Lifecycle via CLI
```bash
az storage account management-policy create \
  --account-name mystorageacct \
  --resource-group MyRG \
  --policy @lifecycle-policy.json
```

---

## 6. Azure Files

### What It Is
Fully managed file shares in the cloud, accessible via SMB (port 445) or NFS (port 2049).

### Azure Files vs Blob Storage

| | Azure Files | Blob Storage |
|-|-------------|-------------|
| Protocol | SMB / NFS / REST | REST / SDK |
| Use case | Shared file system (lift & shift) | Object storage (unstructured data) |
| Mount as drive | ✅ (Windows, Linux, macOS) | ❌ |
| Directory structure | ✅ Real directories | Virtual (prefix-based) |
| Concurrency | File-locking | Optimistic concurrency |

### File Share Tiers

| Tier | Performance | Storage | Use Case |
|------|-------------|---------|----------|
| **Premium** | SSD, low-latency | $/GiB provisioned | Databases, high IOPS |
| **Transaction Optimized** | HDD | Pay per transaction | High-transaction workloads |
| **Hot** | HDD | Balanced | General file sharing |
| **Cool** | HDD | Low storage, higher access | Archival, infrequent access |

### Key Limits
| Property | Standard | Premium |
|----------|----------|---------|
| Max share size | 100 TiB (large file shares enabled) | 100 TiB |
| Max file size | 4 TiB | 4 TiB |
| Max IOPS per share | 20,000 | 100,000 |
| Protocol | SMB 2.1, SMB 3.x, NFS 4.1 | SMB 3.x, NFS 4.1 |

### Mounting Azure File Shares

**Windows:**
```powershell
# Map network drive
net use Z: \\<account>.file.core.windows.net\<share> /u:AZURE\<account> <access-key>

# Or via PowerShell
New-PSDrive -Name Z -PSProvider FileSystem -Root "\\<account>.file.core.windows.net\<share>" -Credential $cred -Persist
```

**Linux:**
```bash
sudo mount -t cifs //<account>.file.core.windows.net/<share> /mnt/share \
  -o vers=3.0,username=<account>,password=<key>,dir_mode=0777,file_mode=0777
```

> **Port 445 requirement:** Many ISPs and corporate firewalls block port 445. Test with `Test-NetConnection -ComputerName <account>.file.core.windows.net -Port 445`

### Identity-Based Authentication for Azure Files
| Method | Description | On-Prem AD Required |
|--------|-------------|-------------------|
| **On-prem AD DS** | Kerberos auth against on-prem AD | ✅ |
| **Entra Domain Services** | Kerberos auth against cloud-managed AD | ❌ (cloud managed) |
| **Entra Kerberos (Hybrid)** | Entra ID users, no domain controller needed | Hybrid identities |

### Snapshots
- Point-in-time read-only copy of a file share
- Incremental (only changes since last snapshot stored)
- Max 200 snapshots per share
- Use for: backup, accidental deletion recovery, point-in-time data

---

## 7. Azure File Sync

### What It Is
Centralizes file shares in Azure Files while caching frequently accessed files on Windows Server(s).

### Architecture

```
Azure Files (Cloud Endpoint)
     ↕ (sync)
Storage Sync Service
     ↕
Sync Group
     ├── Cloud Endpoint (Azure File Share — one per group)
     └── Server Endpoint(s) (on-prem Windows Server paths)
```

### Components

| Component | Description | Limit |
|-----------|-------------|-------|
| **Storage Sync Service** | Top-level resource for File Sync | One per deployment |
| **Sync Group** | Defines sync topology | Contains endpoints |
| **Cloud Endpoint** | Azure File Share in the sync group | 1 per sync group |
| **Server Endpoint** | Path on a registered Windows Server | Multiple per sync group |
| **Registered Server** | Windows Server with File Sync agent installed | Trust relationship |

### Cloud Tiering
| Feature | Description |
|---------|-------------|
| **Purpose** | Keep only frequently accessed files locally; cold files in Azure (stub/placeholder) |
| **Policies** | Volume free space policy (%) + Date policy (days since last access) |
| **Recall** | Transparent — opening a tiered file downloads it seamlessly |
| **Benefit** | Reduces on-prem storage footprint dramatically |

### File Sync Requirements
- Windows Server 2016 or later
- NTFS volumes (ReFS not supported)
- Server endpoint cannot be the volume root (e.g., use D:\FileSync, not D:\)
- No DFS Replication on the same folder
- Azure File Sync agent installed on server

### Setup Steps
1. Deploy **Storage Sync Service** resource in Azure
2. Install **Azure File Sync agent** on Windows Server
3. **Register server** with Storage Sync Service
4. Create **Sync Group** with cloud endpoint (Azure File Share)
5. Add **Server Endpoint** (local path on registered server)
6. Configure **Cloud Tiering** policies (optional)

---

## 8. Storage Security — Access Keys & SAS

### Access Keys
| Property | Detail |
|----------|--------|
| What | Two 512-bit keys providing full access to storage account |
| Scope | **Entire account** — all services, all data |
| Best practice | Rotate regularly; use Key Vault to store |
| Risk | Anyone with the key has full access |

**Regenerating Keys:**
```bash
az storage account keys renew --account-name mystorageacct --resource-group MyRG --key primary
```

> Always have TWO keys so you can rotate one while the other remains active

### Shared Access Signatures (SAS)

| SAS Type | Scope | Signed By | Security |
|----------|-------|-----------|----------|
| **Account SAS** | Entire account (services/resources/APIs) | Account key | Medium |
| **Service SAS** | Single service (blob, file, queue, table) | Account key | Medium |
| **User Delegation SAS** | Blob/container only | Entra ID credential | **Highest** |

### SAS Token Parameters

| Parameter | Controls |
|-----------|----------|
| **sv** (version) | API version |
| **ss** (services) | Blob (b), File (f), Queue (q), Table (t) |
| **srt** (resource types) | Service (s), Container (c), Object (o) |
| **sp** (permissions) | r=read, w=write, d=delete, l=list, a=add, c=create |
| **se** (expiry)  | When the SAS expires (UTC) |
| **st** (start)  | When the SAS becomes valid (UTC) |
| **sip** (IP range) | Allowed client IP range |
| **spr** (protocol) | HTTPS only, or HTTP+HTTPS |

### Generating SAS

```bash
# Account SAS via CLI
az storage account generate-sas \
  --account-name mystorageacct \
  --services b \
  --resource-types sco \
  --permissions rwdlac \
  --expiry 2026-06-01T00:00:00Z \
  --https-only

# User Delegation SAS (recommended)
az storage blob generate-sas \
  --account-name mystorageacct \
  --container-name mycontainer \
  --name myblob.txt \
  --permissions r \
  --expiry 2026-06-01T00:00:00Z \
  --auth-mode login \
  --as-user
```

### Stored Access Policies
| Property | Detail |
|----------|--------|
| What | Named policy on a container that defines SAS parameters |
| Benefit | **Revocable** — delete the policy to invalidate all SAS tokens referencing it |
| Limit | Max **5** stored access policies per container/queue/table/share |
| Scope | Container, queue, table, or file share level |

> **Key insight:** SAS without stored access policy can only be invalidated by rotating the account key (affects ALL SAS tokens). Stored access policy lets you revoke specific tokens.

---

## 9. Storage Security — Entra ID & Network

### Entra ID (RBAC) for Storage

| Role | Scope | Permissions |
|------|-------|-------------|
| **Storage Blob Data Owner** | Account/Container | Full access (read/write/delete + manage permissions) |
| **Storage Blob Data Contributor** | Account/Container | Read/write/delete blobs |
| **Storage Blob Data Reader** | Account/Container | Read blobs only |
| **Storage File Data SMB Share Contributor** | File share | Read/write/delete files via SMB |
| **Storage File Data SMB Share Reader** | File share | Read files via SMB |
| **Storage File Data SMB Share Elevated Contributor** | File share | Read/write/delete + modify NTFS permissions |
| **Storage Queue Data Contributor** | Account/Queue | Read/write/delete queue messages |
| **Storage Table Data Contributor** | Account/Table | Read/write/delete table entities |

> **Best practice:** Use Entra ID RBAC (data plane roles) instead of access keys for production

### Network Security for Storage

| Method | Description |
|--------|-------------|
| **Firewall (IP rules)** | Allow specific public IPs or ranges |
| **Virtual Network rules** | Allow traffic from specific VNet subnets (via service endpoints) |
| **Private Endpoint** | Access storage via private IP in your VNet |
| **Trusted Azure services** | Bypass firewall for Azure Backup, Monitor, etc. |

### Configuring Storage Firewall

```bash
# Default deny all traffic
az storage account update --name mystorageacct --resource-group MyRG --default-action Deny

# Allow specific IP
az storage account network-rule add --account-name mystorageacct --ip-address 203.0.113.0/24

# Allow specific subnet
az storage account network-rule add --account-name mystorageacct \
  --vnet-name MyVNet --subnet MySubnet

# Allow trusted Azure services
az storage account update --name mystorageacct --bypass AzureServices
```

### Encryption
| Feature | Description |
|---------|-------------|
| **Encryption at rest** | Always enabled (256-bit AES); cannot be disabled |
| **Microsoft-managed keys (MMK)** | Default — Azure manages keys |
| **Customer-managed keys (CMK)** | Your keys in Key Vault or Managed HSM |
| **Infrastructure encryption** | Double encryption (service + infrastructure layer) |
| **Encryption in transit** | HTTPS enforced (Secure Transfer Required = enabled by default) |

---

## 10. AzCopy & Storage Explorer

### AzCopy

| Feature | Detail |
|---------|--------|
| What | Command-line tool for high-performance data transfer |
| Auth methods | Entra ID (login), SAS token, or account key |
| Operations | Copy, Sync, Remove, Make (create container), List |
| Platform | Windows, Linux, macOS |
| Performance | Parallel transfers, auto-retry, journal files for resume |

### Key AzCopy Commands

```bash
# Login with Entra ID
azcopy login

# Copy local file to blob
azcopy copy "C:\data\file.txt" "https://account.blob.core.windows.net/container/file.txt"

# Copy entire directory to container
azcopy copy "C:\data\*" "https://account.blob.core.windows.net/container/" --recursive

# Copy between storage accounts (server-side, no local download)
azcopy copy "https://source.blob.core.windows.net/container/*?<SAS>" \
  "https://dest.blob.core.windows.net/container/?<SAS>" --recursive

# Sync (only new/changed files)
azcopy sync "C:\data" "https://account.blob.core.windows.net/container" --recursive

# Download from blob
azcopy copy "https://account.blob.core.windows.net/container/file.txt" "C:\downloads\"
```

### AzCopy vs Other Tools

| Tool | Best For | Supports |
|------|----------|---------|
| **AzCopy** | Large data transfers, scripting, automation | Blob, Files, Table (limited) |
| **Azure Storage Explorer** | GUI-based management, ad-hoc operations | Blob, Files, Queue, Table |
| **Portal** | Quick uploads, small files | All services |
| **PowerShell/CLI** | Scripted operations, integration with automation | All services |

### Azure Storage Explorer
- Cross-platform desktop application (Windows, macOS, Linux)
- Visual interface for managing all storage services
- Supports: Entra ID, SAS, account key, and anonymous access
- Can manage multiple storage accounts/subscriptions
- Uses AzCopy under the hood for transfers

---

## 11. Import/Export & Data Box

### When to Use Offline Transfer

| Data Amount | Bandwidth | Time to Upload Online | Recommendation |
|-------------|-----------|----------------------|----------------|
| < 10 TB | Good internet | Hours-days | Use AzCopy/online |
| 10-100 TB | Limited bandwidth | Weeks-months | **Import/Export Service** |
| > 100 TB | Any | Months+ | **Azure Data Box** |

### Import/Export Service
| Property | Detail |
|----------|--------|
| What | Ship physical hard drives to Azure datacenter |
| Direction | Import (to Azure) OR Export (from Azure) |
| Drive types | 2.5"/3.5" SATA HDD/SSD |
| Encryption | BitLocker required |
| Supported targets | Blob Storage, Azure Files (import only) |
| Tool | WAImportExport tool (prepares drives) |

### Azure Data Box Family

| Product | Capacity | Use Case |
|---------|----------|----------|
| **Data Box Disk** | Up to 35 TB (5 × 8 TB SSD) | Small-medium offline transfer |
| **Data Box** | 80 TB usable | Medium-large offline transfer |
| **Data Box Heavy** | 770 TB usable | Massive data migration |

### Process Flow (Data Box)
1. Order Data Box from Azure Portal
2. Microsoft ships device to you
3. Connect to network, copy data via SMB/NFS
4. Ship back to Microsoft
5. Data uploaded to your storage account
6. Device securely wiped

---

## 12. Soft Delete & Versioning

### Blob Soft Delete
| Property | Detail |
|----------|--------|
| What | Deleted blobs retained for specified period |
| Retention | 1-365 days (configurable) |
| Recovery | Undelete via Portal, CLI, or REST API |
| Default | **Enabled** (7 days) on new storage accounts |
| Applies to | Block blobs, append blobs, page blobs |

### Container Soft Delete
| Property | Detail |
|----------|--------|
| What | Deleted containers retained for specified period |
| Retention | 1-365 days (configurable) |
| Default | **Enabled** (7 days) on new storage accounts |

### Blob Versioning
| Property | Detail |
|----------|--------|
| What | Automatically creates a version on every write/overwrite |
| Access | Access previous versions by version ID |
| Cost | Each version consumes storage (incremental) |
| Use with | Works with soft delete for comprehensive data protection |

### Azure File Share Soft Delete
| Property | Detail |
|----------|--------|
| What | Deleted file shares retained for recovery |
| Retention | 1-365 days |
| Default | **Enabled** (7 days) |
| Recovery | Portal → Deleted shares → Restore |

---

## 13. Exam Tips — Domain 2 Master List

### Storage Accounts
- **Name rules:** 3-24 chars, lowercase + numbers only, globally unique
- **Cannot change:** name, region, performance tier, hierarchical namespace after creation
- **GPv2 is the answer** unless Premium performance is explicitly required
- **Premium accounts** do NOT support access tiers (Hot/Cool/Cold/Archive)

### Redundancy
- **GRS secondary is NOT readable** — need **RA-GRS** for read access
- **LRS ↔ ZRS** requires live migration request or manual data copy
- **After failover:** storage account becomes LRS in the new primary region
- **ZRS** is recommended minimum for production workloads
- **Durability:** All options have at least 11 nines; GRS/GZRS have 16 nines

### Blob Storage
- **Archive requires rehydration** — hours, not instant access
- **Archive is blob-level only** — cannot set as account default tier
- **Early deletion fee** — deleting Cool blob before 30 days, Cold before 90 days, Archive before 180 days
- **Lifecycle management** — can only tier DOWN (Hot→Cool→Cold→Archive), not up; delete is possible at any point
- **Public access disabled by default** on new accounts (since 2024)

### Security
- **User Delegation SAS is most secure** (no account key, uses Entra ID, blob only)
- **Stored Access Policy** — only way to revoke SAS without rotating account key
- **Max 5 stored access policies** per container/queue/table/share
- **Secure transfer required** = HTTPS only (default: enabled)
- **Storage firewall default action Deny** — don't forget to add "Trusted Azure services" exception
- **Two access keys** — rotate one at a time (Key1 active → rotate Key2 → switch to Key2 → rotate Key1)

### Azure Files
- **Port 445 (SMB)** — often blocked by ISPs; verify connectivity first
- **Server endpoint ≠ volume root** — must be a subfolder (e.g., D:\Sync not D:\)
- **Cloud tiering** — files appear as stubs locally; seamless recall on access
- **NFS requires Premium tier** — standard tier only supports SMB
- **Identity auth for Files:** needs Entra Domain Services, on-prem AD DS, or Entra Kerberos

### Data Transfer
- **AzCopy** = best for large transfers and automation
- **AzCopy sync** = one-way sync (source to destination); not bidirectional
- **Server-to-server copy** with AzCopy: no local download needed (uses Azure backbone)
- **Import/Export** = BitLocker-encrypted drives shipped to Microsoft
- **Data Box** = Microsoft ships device to you; you copy data and ship back

---

*Next: [Domain 3 — Deploy and Manage Compute Resources](AZ-104-Compute.md)*
