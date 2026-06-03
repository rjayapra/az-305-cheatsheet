# AZ-104 — Manage Azure Identities and Governance
## Domain 1 Deep-Dive Study Guide (20–25% of Exam)

---

## Table of Contents
1. [Microsoft Entra ID — Core Concepts](#1-microsoft-entra-id--core-concepts)
2. [Users — Create & Manage](#2-users--create--manage)
3. [Groups — Types & Membership](#3-groups--types--membership)
4. [Administrative Units](#4-administrative-units)
5. [Devices — Registration & Join](#5-devices--registration--join)
6. [RBAC — Role-Based Access Control](#6-rbac--role-based-access-control)
7. [Custom RBAC Roles](#7-custom-rbac-roles)
8. [Management Groups](#8-management-groups)
9. [Subscriptions](#9-subscriptions)
10. [Azure Policy](#10-azure-policy)
11. [Resource Locks](#11-resource-locks)
12. [Resource Tags](#12-resource-tags)
13. [Cost Management](#13-cost-management)
14. [Exam Tips — Domain 1 Master List](#14-exam-tips--domain-1-master-list)

---

## 1. Microsoft Entra ID — Core Concepts

### What It Is
Cloud-based identity and access management service. Every Azure subscription is associated with one Entra ID tenant.

### Key Terminology

| Term | Definition | Admin Action |
|------|-----------|-------------|
| **Tenant** | Dedicated Entra ID instance for an organization | Created automatically with Azure signup |
| **Directory** | Container for users, groups, apps, devices | Synonymous with tenant in practice |
| **Domain** | DNS name associated with tenant (e.g., contoso.onmicrosoft.com) | Can add custom domains |
| **Custom Domain** | Your own domain (contoso.com) verified via DNS TXT record | Add → verify → make primary |
| **Subscription** | Billing & access boundary tied to one Entra ID tenant | Can transfer between tenants |

### Entra ID License Tiers (AZ-104 Relevance)

| Feature | Free | P1 | P2 |
|---------|------|----|----|
| Users, Groups, Basic Auth | ✅ | ✅ | ✅ |
| Self-Service Password Reset (SSPR) | Cloud-only | + writeback to on-prem | ✅ |
| Dynamic Groups | ❌ | ✅ | ✅ |
| Conditional Access | ❌ | ✅ | ✅ |
| Group-based Licensing | ❌ | ✅ | ✅ |
| Identity Protection | ❌ | ❌ | ✅ |
| PIM | ❌ | ❌ | ✅ |
| Access Reviews | ❌ | ❌ | ✅ |

> **AZ-104 focus:** You need to KNOW license requirements but don't need to deeply configure P2 features (that's AZ-305 territory)

---

## 2. Users — Create & Manage

### User Types

| Type | Description | Source |
|------|-------------|--------|
| **Cloud identity** | Created directly in Entra ID | Azure Portal / CLI / PowerShell |
| **Directory-synced** | Synced from on-prem AD via Azure AD Connect | On-premises AD |
| **Guest user** | Invited from external tenant (B2B) | Email invitation |

### Creating Users

**Azure Portal:**
Entra ID → Users → + New user → Create / Invite

**Azure CLI:**
```bash
# Create cloud user
az ad user create \
  --display-name "John Smith" \
  --user-principal-name john@contoso.onmicrosoft.com \
  --password "P@ssword123!" \
  --force-change-password-next-sign-in true

# List users
az ad user list --output table

# Delete user
az ad user delete --id john@contoso.onmicrosoft.com
```

**PowerShell:**
```powershell
# Create user
$PasswordProfile = @{ Password = "P@ssword123!" }
New-MgUser -DisplayName "John Smith" `
  -UserPrincipalName "john@contoso.onmicrosoft.com" `
  -PasswordProfile $PasswordProfile `
  -AccountEnabled $true `
  -MailNickname "john"

# Bulk create users (CSV)
$users = Import-Csv -Path "users.csv"
foreach ($user in $users) {
    New-MgUser -DisplayName $user.DisplayName -UserPrincipalName $user.UPN ...
}
```

### User Properties to Know

| Property | Required | Notes |
|----------|----------|-------|
| Display Name | ✅ | What appears in directory |
| User Principal Name | ✅ | Sign-in name (email format) |
| Password | ✅ (create) | Temporary; force change on first login |
| Usage Location | For licensing | 2-letter country code (e.g., US, CA) |
| Job Title | ❌ | Can be used in dynamic group rules |
| Department | ❌ | Can be used in dynamic group rules |
| Manager | ❌ | For reporting structure |

### Bulk Operations

| Operation | Method | File Format |
|-----------|--------|-------------|
| Bulk create users | Portal → Users → Bulk operations → Create | CSV template |
| Bulk invite users | Portal → Users → Bulk operations → Invite | CSV template |
| Bulk delete users | Portal → Users → Bulk operations → Delete | CSV template |
| Bulk download | Portal → Users → Download users | CSV export |

### Deleted Users (Soft Delete)
- Deleted users are recoverable for **30 days**
- During soft-delete period: user appears in "Deleted users" list
- After 30 days: permanently deleted (unrecoverable)
- Restore: `az ad user restore --id <user-id>` or Portal → Deleted users → Restore

### Self-Service Password Reset (SSPR)

| Setting | Options |
|---------|---------|
| **Enabled for** | None, Selected (group), All |
| **Authentication methods** | Mobile phone, email, security questions, Microsoft Authenticator, office phone |
| **Number of methods required** | 1 or 2 |
| **Registration** | Require users to register; re-confirm after X days |
| **Notifications** | Notify user on reset; notify admins when admin resets |
| **On-prem writeback** | Requires P1 + Azure AD Connect (password writeback enabled) |

> **Exam tip:** SSPR for cloud users = Free. SSPR writeback to on-prem = P1.

---

## 3. Groups — Types & Membership

### Group Types

| | Security Group | Microsoft 365 Group |
|-|---------------|-------------------|
| **Purpose** | Assign permissions (RBAC, NSG, resources) | Collaboration (shared mailbox, SharePoint, Teams) |
| **Membership types** | Assigned, Dynamic User, Dynamic Device | Assigned, Dynamic User |
| **Can assign Azure roles** | ✅ (if role-assignable) | ✅ (if role-assignable) |
| **Dynamic device membership** | ✅ | ❌ |
| **Nested groups** | ✅ | ❌ |
| **License assignment** | ✅ | ✅ |

### Membership Types

| Type | How Members Are Added | License Required |
|------|----------------------|-----------------|
| **Assigned** | Manually added by admin | Free |
| **Dynamic User** | Automatically based on user properties | **P1** |
| **Dynamic Device** | Automatically based on device properties (Security only) | **P1** |

### Dynamic Membership Rules

```
# Users in IT department
user.department -eq "IT"

# Users with specific job title
user.jobTitle -contains "Engineer"

# Users in Canada
user.usageLocation -eq "CA"

# Devices running Windows
device.deviceOSType -eq "Windows"

# Compound rule
(user.department -eq "Sales") -and (user.country -eq "US")
```

**Operators:** `-eq`, `-ne`, `-contains`, `-notContains`, `-startsWith`, `-in`, `-match`

### Group Management

```bash
# Azure CLI
az ad group create --display-name "DevTeam" --mail-nickname "devteam"
az ad group member add --group "DevTeam" --member-id <user-object-id>
az ad group member list --group "DevTeam" --output table

# PowerShell
New-MgGroup -DisplayName "DevTeam" -MailEnabled:$false -SecurityEnabled:$true -MailNickname "devteam"
```

### Group-Based Licensing
- Assign licenses to a group → all members automatically get the license
- Requires **P1** license
- If license pool exhausted: members get error state, not auto-removed
- Use **Usage Location** must be set on users before assigning licenses

---

## 4. Administrative Units

### What It Is
A container in Entra ID that restricts the scope of an admin role to a specific set of users, groups, or devices.

### Why Use AUs
- **Delegated administration** without tenant-wide access
- Example: Regional helpdesk admin can only reset passwords for users in their AU

### AU Management

| Action | Method |
|--------|--------|
| Create AU | Portal → Entra ID → Administrative Units → + Add |
| Add members | Users, Groups, or Devices → Add to AU |
| Assign scoped role | Assign "Helpdesk Administrator" at AU scope |

### Supported Scoped Roles
- Authentication Administrator
- Groups Administrator
- Helpdesk Administrator
- License Administrator
- Password Administrator
- User Administrator

### AU Membership Types
| Type | Description |
|------|-------------|
| **Assigned** | Manually add users/groups/devices |
| **Dynamic** | Rule-based membership (requires P1) |

> **Exam tip:** AUs scope ADMIN roles. They do NOT affect resource access (RBAC) or Conditional Access.

---

## 5. Devices — Registration & Join

### Device Identity Options

| Option | Description | Management | MDM Required |
|--------|-------------|-----------|-------------|
| **Entra ID Registered** | Personal device; user signs in with personal + work account | Light | ❌ (optional) |
| **Entra ID Joined** | Organization-owned device; sign in with Entra ID only | Full | ✅ |
| **Hybrid Entra ID Joined** | Joined to both on-prem AD and Entra ID | Full | Optional |

### Comparison

| Feature | Registered | Joined | Hybrid Joined |
|---------|-----------|--------|--------------|
| Device ownership | Personal (BYOD) | Corporate | Corporate |
| Sign-in with | Local/personal + work account | Entra ID only | Domain + Entra ID |
| SSO to cloud resources | ✅ | ✅ | ✅ |
| SSO to on-prem resources | ❌ | ❌ (unless config'd) | ✅ |
| Conditional Access (device compliance) | ✅ | ✅ | ✅ |
| OS support | Windows, iOS, Android, macOS | Windows 10/11 only | Windows 10/11 |
| Requires on-prem AD | ❌ | ❌ | ✅ |

### Device Settings in Entra ID

| Setting | Description |
|---------|-------------|
| Users may join devices | All / Selected / None |
| Users may register devices | All / Selected / None |
| Require MFA to join | Yes / No |
| Max devices per user | Default: 50 |
| Enterprise State Roaming | Sync settings across Windows devices |

---

## 6. RBAC — Role-Based Access Control

### How RBAC Works

```
Security Principal + Role Definition + Scope = Role Assignment
(Who)             (What can they do) (Where)  (The permission binding)
```

### Security Principals
| Type | Description |
|------|-------------|
| **User** | Individual person |
| **Group** | Set of users |
| **Service Principal** | App/service identity |
| **Managed Identity** | Auto-managed identity for Azure services |

### Scope Hierarchy (Broadest → Narrowest)

```
Management Group
  └── Subscription
        └── Resource Group
              └── Resource
```

> Permissions inherit **downward**. Role assigned at subscription scope applies to ALL resource groups and resources in that subscription.

### Key Built-in Roles

| Role | Description | Can Assign Roles? |
|------|-------------|-------------------|
| **Owner** | Full access to all resources + manage access | ✅ |
| **Contributor** | Full access to all resources | ❌ |
| **Reader** | View all resources | ❌ |
| **User Access Administrator** | Manage role assignments only | ✅ |
| **Virtual Machine Contributor** | Manage VMs (not network/storage they use) | ❌ |
| **Storage Blob Data Contributor** | Read/write/delete blob data | ❌ |
| **Storage Blob Data Reader** | Read blob data only | ❌ |
| **Network Contributor** | Manage all networking resources | ❌ |
| **Monitoring Reader** | Read monitoring data | ❌ |
| **Monitoring Contributor** | Read + manage monitoring | ❌ |
| **Backup Contributor** | Manage backup (not vault access) | ❌ |
| **Key Vault Secrets User** | Read secrets from Key Vault | ❌ |

### RBAC Assignment via CLI

```bash
# Assign role
az role assignment create \
  --assignee "user@contoso.com" \
  --role "Contributor" \
  --scope "/subscriptions/<sub-id>/resourceGroups/MyRG"

# List role assignments
az role assignment list --resource-group MyRG --output table

# Remove role assignment
az role assignment delete --assignee "user@contoso.com" --role "Contributor" --scope <scope>
```

### RBAC Assignment via PowerShell

```powershell
# Assign role
New-AzRoleAssignment -SignInName "user@contoso.com" `
  -RoleDefinitionName "Contributor" `
  -ResourceGroupName "MyRG"

# List assignments
Get-AzRoleAssignment -ResourceGroupName "MyRG"

# Remove assignment
Remove-AzRoleAssignment -SignInName "user@contoso.com" `
  -RoleDefinitionName "Contributor" `
  -ResourceGroupName "MyRG"
```

### RBAC vs Azure Policy

| | RBAC | Azure Policy |
|-|------|-------------|
| Controls | **WHO** can do what | **WHAT** can be deployed/configured |
| Focus | Actions (create, delete, write) | Resource properties & compliance |
| Example | "User X can create VMs" | "All VMs must use managed disks" |
| Deny mechanism | ❌ (no RBAC deny assignments for users) | ✅ (Deny effect blocks deployments) |

---

## 7. Custom RBAC Roles

### When to Use
- Built-in roles are too broad or too narrow
- Need to allow specific actions (e.g., restart VMs only)

### Custom Role Definition Structure

```json
{
  "Name": "VM Restart Operator",
  "Description": "Can restart virtual machines only",
  "Actions": [
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/<subscription-id>"
  ]
}
```

### Role Definition Properties

| Property | Description |
|----------|-------------|
| **Actions** | Control-plane operations allowed (e.g., Microsoft.Compute/*/read) |
| **NotActions** | Operations excluded from Actions (subtract from Actions) |
| **DataActions** | Data-plane operations allowed (e.g., Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read) |
| **NotDataActions** | Data-plane operations excluded |
| **AssignableScopes** | Where the role can be assigned (subscription, resource group, management group) |

### Actions vs DataActions

| | Actions (Control Plane) | DataActions (Data Plane) |
|-|------------------------|--------------------------|
| What | Manage the resource itself | Access data within the resource |
| Example | Create/delete storage account | Read/write blobs in the container |
| Tools | Portal, CLI, ARM API | SDK, REST API, Storage Explorer |
| Example role | "Storage Account Contributor" | "Storage Blob Data Reader" |

### Creating Custom Roles

```bash
# CLI - from JSON file
az role definition create --role-definition @custom-role.json

# CLI - list custom roles
az role definition list --custom-role-only true --output table

# PowerShell
New-AzRoleDefinition -InputFile "custom-role.json"
```

### Custom Role Limits
- Max **5,000** custom roles per tenant
- AssignableScopes: can include subscriptions or resource groups (not individual resources)
- Cannot use wildcard (*) in AssignableScopes for custom roles

---

## 8. Management Groups

### What It Is
Containers above subscriptions for organizing and applying governance at scale.

### Hierarchy

```
Tenant Root Management Group (auto-created)
  └── IT Management Group
        ├── Production MG
        │     ├── Prod Subscription 1
        │     └── Prod Subscription 2
        └── Dev/Test MG
              └── Dev Subscription 1
```

### Key Facts

| Property | Limit/Detail |
|----------|-------------|
| Max depth | 6 levels (below root) |
| Max MGs per directory | 10,000 |
| Subscriptions per MG | Unlimited (technically 10,000) |
| Subscription can belong to | Exactly 1 MG |
| Policy inheritance | Applies to all children (MGs + subscriptions below) |
| RBAC inheritance | Applies to all children |
| Move subscription | Requires permissions on source MG, target MG, and subscription |

### Management Group Operations

```bash
# CLI
az account management-group create --name "ProductionMG" --display-name "Production"
az account management-group move --name <subscription-id> --new-parent-id "ProductionMG"
az account management-group list --output table

# PowerShell
New-AzManagementGroup -GroupId "ProductionMG" -DisplayName "Production"
```

### Root Management Group
- Every tenant has one **Tenant Root Management Group**
- All subscriptions and MGs are under it
- Cannot be moved or deleted
- By default, only Global Admin can access it (must elevate)
- Elevation: Entra ID → Properties → "Access management for Azure resources" = Yes

---

## 9. Subscriptions

### What It Is
A logical billing and access boundary for Azure resources.

### Subscription vs Resource Group vs Resource

| Level | Purpose | Example |
|-------|---------|---------|
| **Subscription** | Billing boundary, trust boundary | "Production Sub," "Dev Sub" |
| **Resource Group** | Logical grouping, lifecycle management | "WebApp-RG," "Database-RG" |
| **Resource** | Individual service instance | A specific VM, storage account, VNet |

### Key Facts
- A subscription trusts exactly ONE Entra ID tenant
- Resources in a subscription can only use identities from that tenant
- Can transfer subscription between tenants (breaks RBAC assignments)
- Each subscription has spending limits (free/sponsorship) or is pay-as-you-go

### Moving Resources Between Resource Groups / Subscriptions

| What Moves | Between RGs | Between Subscriptions |
|------------|-------------|----------------------|
| Most resources | ✅ | ✅ (same tenant) |
| Resource locks | ❌ Must remove first | ❌ Must remove first |
| RBAC assignments | ❌ Don't move (re-create) | ❌ Don't move |
| Resource Group itself | ❌ (it's a container) | ❌ |

### Resources That CANNOT Move
- Azure AD Domain Services
- Azure Backup Recovery Services Vault (with items)
- Classic deployment model resources
- Some networking resources (when dependencies exist)

> **Exam tip:** Check `az resource move` documentation — not all resources support cross-subscription move

---

## 10. Azure Policy

### What It Is
Service that enforces organizational standards and assesses compliance at scale.

### How Policy Works

```
Policy Definition → Policy Assignment → Compliance Evaluation
  (What to enforce)   (Where to enforce)   (Are resources compliant?)
```

### Policy Effects (Evaluated in this order)

| Effect | When Evaluated | Behavior |
|--------|---------------|----------|
| **Disabled** | — | Policy is off |
| **Append** | Create/Update | Add fields to resource (e.g., add a tag) |
| **Modify** | Create/Update | Change properties using patch operations |
| **Deny** | Create/Update | Block the operation |
| **Audit** | Create/Update + existing | Log non-compliance (no block) |
| **AuditIfNotExists** | After create/update | Audit if related resource doesn't exist |
| **DeployIfNotExists** | After create/update | Deploy a resource if it doesn't exist |

> **Evaluation order:** Disabled → Deny → Append/Modify → Audit/AuditIfNotExists/DeployIfNotExists

### Policy vs Initiative

| | Policy | Initiative (Policy Set) |
|-|--------|------------------------|
| Scope | Single rule | Collection of policies |
| Use case | "Allowed VM sizes" | "ISO 27001 compliance" |
| Assignment | Assign individually | Assign as a group |
| Parameters | Per-policy | Can share across policies |

### Common Built-in Policies

| Policy | Effect | What It Does |
|--------|--------|-------------|
| Allowed locations | Deny | Restrict resource deployment to specific regions |
| Allowed virtual machine size SKUs | Deny | Restrict VM sizes |
| Require a tag on resources | Deny | Block creation without required tag |
| Inherit a tag from the resource group | Modify | Auto-apply RG tag to child resources |
| Deploy Diagnostic Settings | DeployIfNotExists | Auto-deploy diagnostics to Log Analytics |
| Allowed storage account SKUs | Deny | Restrict storage redundancy options |

### Policy Assignment

```bash
# CLI - Assign policy
az policy assignment create \
  --name "require-tag-env" \
  --policy "/providers/Microsoft.Authorization/policyDefinitions/<policy-id>" \
  --scope "/subscriptions/<sub-id>/resourceGroups/MyRG" \
  --params '{"tagName": {"value": "Environment"}}'

# CLI - Check compliance
az policy state list --resource-group MyRG --output table
```

### Policy Remediation
- For existing non-compliant resources
- Create a **remediation task** to apply Modify/DeployIfNotExists retroactively
- Requires a **managed identity** for the remediation (auto-created with appropriate permissions)
- Only works with Modify and DeployIfNotExists effects

### Policy Exemptions
- Temporarily exempt a resource/scope from policy
- Types: **Waiver** (intentional non-compliance) or **Mitigated** (compliance met through other means)
- Has expiration date

---

## 11. Resource Locks

### What It Is
Protection mechanism that prevents accidental modification or deletion of Azure resources.

### Lock Types

| Lock | Can Read | Can Modify | Can Delete |
|------|----------|-----------|-----------|
| **ReadOnly** | ✅ | ❌ | ❌ |
| **CanNotDelete (Delete)** | ✅ | ✅ | ❌ |
| **No Lock** | ✅ | ✅ | ✅ |

### Key Behaviors
- **Locks override RBAC** — even Owner cannot delete a CanNotDelete-locked resource
- Must **remove the lock first**, then delete
- Locks can be applied at: Subscription, Resource Group, or Resource level
- Lock on RG **applies to all resources** in that RG
- Locks are **inherited** downward (parent → child)

### Lock Gotchas

| Scenario | ReadOnly Lock Effect |
|----------|---------------------|
| Storage account | Cannot list keys (write operation) |
| App Service | Cannot scale up/down |
| Virtual Machine | Cannot start/stop/restart |
| Resource Group | Cannot add new resources (write operation) |

> **Exam tip:** ReadOnly is MORE restrictive than it sounds. Many "read" operations in the portal actually require a write operation (e.g., listing storage keys).

### Managing Locks

```bash
# CLI
az lock create --name "NoDelete" --resource-group MyRG --lock-type CanNotDelete
az lock create --name "ReadOnly" --resource-group MyRG --lock-type ReadOnly
az lock list --resource-group MyRG --output table
az lock delete --name "NoDelete" --resource-group MyRG
```

---

## 12. Resource Tags

### What It Is
Name-value pairs applied to resources for organization, cost management, and automation.

### Key Facts

| Property | Detail |
|----------|--------|
| Max tags per resource | 50 |
| Tag name max length | 512 characters |
| Tag value max length | 256 characters |
| Case sensitive | Tag NAMES are case-insensitive; VALUES are case-sensitive |
| Inheritance | ❌ Tags do NOT inherit (use Policy to enforce) |
| Supported on | Most resources (not all) |

### Common Tag Strategies

| Tag Name | Purpose | Example Values |
|----------|---------|---------------|
| Environment | Identify environment | Production, Staging, Dev, Test |
| CostCenter | Chargeback | CC-1234, Finance, Engineering |
| Owner | Accountability | john@contoso.com |
| Project | Project tracking | ProjectAlpha, Q4-Migration |
| CreatedBy | Audit trail | ARM-Template, Manual |
| ExpirationDate | Cleanup automation | 2026-12-31 |

### Tagging Commands

```bash
# CLI - Add/update tag
az resource tag --tags Environment=Production CostCenter=CC-1234 --ids <resource-id>

# CLI - Add tag without removing existing
az tag update --resource-id <resource-id> --operation merge --tags NewTag=Value

# PowerShell
Set-AzResource -ResourceId <id> -Tag @{Environment="Production"; CostCenter="CC-1234"} -Force

# Tag a resource group
az group update --name MyRG --tags Environment=Production
```

### Enforce Tags with Policy
- **"Require a tag and its value on resources"** — Deny effect
- **"Inherit a tag from the resource group"** — Modify effect (copies RG tag to resources)
- **"Require a tag on resource groups"** — Deny effect

---

## 13. Cost Management

### Key Tools

| Tool | Purpose |
|------|---------|
| **Cost Analysis** | View current and forecasted costs; break down by service, resource group, tag |
| **Budgets** | Set spending thresholds with alerts |
| **Azure Advisor** | Cost recommendations (reserved instances, right-sizing) |
| **Pricing Calculator** | Estimate costs before deploying |
| **TCO Calculator** | Compare on-prem vs Azure costs |
| **Cost alerts** | Notify when budget threshold reached |

### Budget Alerts

| Alert Type | Trigger |
|-----------|---------|
| **Budget alert** | Actual spend or forecast reaches % of budget |
| **Credit alert** | Enterprise Agreement credit usage |
| **Department quota alert** | EA department spending |

### Cost Saving Strategies (AZ-104 Level)

| Strategy | Savings | How |
|----------|---------|-----|
| **Right-sizing** | 10-60% | Use Advisor recommendations to downsize under-used VMs |
| **Reserved Instances** | Up to 72% | 1 or 3 year commitment for VMs/SQL/Cosmos |
| **Spot VMs** | Up to 90% | Interruptible workloads (batch, dev/test) |
| **Auto-shutdown** | Varies | Schedule dev/test VMs to stop after hours |
| **Azure Hybrid Benefit** | ~40% | Use existing Windows/SQL Server licenses |
| **Storage tiering** | Varies | Move cold data to Cool/Archive tiers |
| **Delete unused resources** | Direct | Orphaned disks, old snapshots, unused public IPs |

---

## 14. Exam Tips — Domain 1 Master List

### Identity
- **Deleted users are recoverable for 30 days** (soft delete)
- **Guest users** (B2B) have #EXT# in their UPN
- **Dynamic groups require P1** — test this with user property rules
- **SSPR writeback requires P1** + Azure AD Connect configured
- **Administrative Units** scope admin roles — NOT resource access
- **Usage Location** must be set before assigning licenses

### RBAC
- **Inheritance is downward only** — never upward
- **Deny assignments** cannot be created by users (only Azure Blueprints/managed apps)
- **Owner vs Contributor:** Owner can assign roles; Contributor cannot
- **User Access Administrator:** can assign roles but NOT manage resources
- **Check effective access:** Portal → Resource → Access Control (IAM) → Check access
- **Custom roles:** max 5,000 per tenant; AssignableScopes required

### Governance
- **Policy Deny evaluates BEFORE resource creation** — blocks immediately
- **Policy effects order:** Disabled → Deny → Append/Modify → Audit → AuditIfNotExists → DeployIfNotExists
- **Locks override RBAC** — Owner can't delete a locked resource without removing lock
- **ReadOnly lock on storage** = cannot list keys
- **Tags are NOT inherited** — use Policy with Modify effect to enforce
- **Management Groups max 6 levels deep** (below root)
- **Moving subscription to new MG** requires write on both MGs + subscription
- **Policy at MG scope** inherits to all child subscriptions (cannot be overridden at lower scope)

---

*Next: [Domain 2 — Implement and Manage Storage](AZ-104-Storage.md)*
