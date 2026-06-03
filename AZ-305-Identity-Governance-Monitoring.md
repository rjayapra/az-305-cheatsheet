# AZ-305 — Identity, Governance & Monitoring
## Domain 1 Deep-Dive Study Guide (25–30% of Exam)
> Exam: May 21, 2026

---

## Table of Contents
1. [Microsoft Entra ID — License Tiers & Core Features](#1-microsoft-entra-id--license-tiers--core-features)
2. [Authentication Methods](#2-authentication-methods)
3. [Multi-Factor Authentication (MFA)](#3-multi-factor-authentication-mfa)
4. [Conditional Access](#4-conditional-access)
5. [Privileged Identity Management (PIM)](#5-privileged-identity-management-pim)
6. [Identity Protection](#6-identity-protection)
7. [Access Reviews](#7-access-reviews)
8. [Entitlement Management](#8-entitlement-management)
9. [B2B vs B2C — External Identities](#9-b2b-vs-b2c--external-identities)
10. [Hybrid Identity](#10-hybrid-identity)
11. [Azure AD Domain Services (AADDS)](#11-azure-ad-domain-services-aadds)
12. [RBAC (Role-Based Access Control)](#12-rbac-role-based-access-control)
13. [Governance — Management Groups & Subscriptions](#13-governance--management-groups--subscriptions)
14. [Azure Policy](#14-azure-policy)
15. [Azure Blueprints](#15-azure-blueprints)
16. [Azure Landing Zones](#16-azure-landing-zones)
17. [Resource Locks & Tags](#17-resource-locks--tags)
18. [Cost Management & FinOps](#18-cost-management--finops)
19. [Azure Monitor](#19-azure-monitor)
20. [Log Analytics Workspace](#20-log-analytics-workspace)
21. [Application Insights](#21-application-insights)
22. [Azure Alerts & Action Groups](#22-azure-alerts--action-groups)
23. [Microsoft Defender for Cloud](#23-microsoft-defender-for-cloud)
24. [Azure Advisor](#24-azure-advisor)
25. [Azure Service Health](#25-azure-service-health)
26. [Exam Tips — Master List](#26-exam-tips--master-list)

---

## 1. Microsoft Entra ID — License Tiers & Core Features

### License Tier Comparison

| Feature | **Free** | **P1** | **P2** | License Source |
|---------|----------|--------|--------|----------------|
| **Directory objects** | 500,000 | Unlimited | Unlimited | — |
| **SSO** | Up to 10 apps | Unlimited | Unlimited | — |
| **MFA (basic per-user)** | ✅ | ✅ | ✅ | Free |
| **MFA (Conditional Access)** | ❌ | ✅ | ✅ | P1 |
| **B2B guest access** | ✅ | ✅ | ✅ | Free |
| **SSPR (cloud-only users)** | ✅ | ✅ | ✅ | Free |
| **SSPR writeback to on-prem AD** | ❌ | ✅ | ✅ | P1 |
| **Conditional Access policies** | ❌ | ✅ | ✅ | **P1** |
| **Dynamic Groups** | ❌ | ✅ | ✅ | P1 |
| **Group-based licensing** | ❌ | ✅ | ✅ | P1 |
| **Application Proxy** | ❌ | ✅ | ✅ | P1 |
| **Hybrid identity (AD Connect)** | ✅ (basic) | ✅ | ✅ | Free (sync), P1 (writeback) |
| **PIM** | ❌ | ❌ | ✅ | **P2** |
| **Identity Protection** | ❌ | ❌ | ✅ | **P2** |
| **Access Reviews** | ❌ | ❌ | ✅ | **P2** |
| **Entitlement Management** | ❌ | ❌ | ✅ | **P2** |
| **Entra ID Governance** | ❌ | ❌ | ✅ | **P2** |

> 💰 **Cost (approx.):** Free = included | P1 = ~$6/user/month | P2 = ~$9/user/month
> P1 and P2 are also included in **Microsoft 365 E3** (P1) and **Microsoft 365 E5** (P2)

---

### Key Entra ID Concepts

| Concept | Description | Exam Relevance |
|---------|-------------|----------------|
| **Tenant** | Dedicated Entra ID instance; represents an organization | High |
| **Directory** | Container for users, groups, apps, devices within a tenant | High |
| **Managed Identity** | Service principal without credential management; System-assigned or User-assigned | Very High |
| **Service Principal** | Identity for an application in a tenant | High |
| **App Registration** | Registers an app to obtain service principal | High |
| **Device Registration** | Registers device identity in Entra ID | Medium |
| **Entra ID Join** | Device fully managed by Entra ID (Azure AD Join) | Medium |
| **Hybrid Entra ID Join** | Device joined to on-prem AD AND registered in Entra ID | Medium |

### Managed Identity — System vs User Assigned

| | System-Assigned | User-Assigned |
|-|----------------|---------------|
| Lifecycle | Tied to the Azure resource | Independent resource |
| Sharing | 1:1 with resource | Can be shared across multiple resources |
| Delete on resource delete | ✅ Auto-deleted | ❌ Must be deleted separately |
| Use case | Simple, single-resource scenarios | Shared identity across multiple VMs/services |
| Creation | Auto on resource enable | Manually created as standalone resource |

> **Exam tip:** Questions about "avoid storing credentials in code" or "app accesses Key Vault without secrets" → **Managed Identity**

---

## 2. Authentication Methods

| Method | Description | Strength | Use Case |
|--------|-------------|---------|---------|
| **Password** | Traditional | Weakest | Legacy; combine with MFA |
| **Passwordless Phone Sign-in** | Microsoft Authenticator app approval | Strong | Mobile workers |
| **FIDO2 Security Key** | Hardware key (YubiKey, etc.) | Strongest | High-security, shared workstations |
| **Windows Hello for Business** | Biometric/PIN, device-bound | Very Strong | Corporate workstations |
| **SMS/Voice OTP** | One-time code via SMS or call | Weak (SIM swappable) | Fallback only |
| **TOTP (Authenticator app code)** | Time-based one-time password | Strong | General MFA |
| **Certificate-based Auth (CBA)** | X.509 certificate | Very Strong | Compliance, phishing-resistant |
| **Temporary Access Pass (TAP)** | Time-limited passcode | Temporary | Onboarding, recovery |

### Authentication Strength (phishing-resistant)

| Level | Methods | Use Case |
|-------|---------|---------|
| **MFA** | Authenticator push, TOTP, SMS | General MFA requirement |
| **Passwordless MFA** | Authenticator phone sign-in, FIDO2, WHfB | Higher assurance |
| **Phishing-resistant MFA** | FIDO2, WHfB, CBA | Highest assurance; government/compliance |

> **Exam tip:** "Phishing-resistant MFA" = FIDO2 or Windows Hello for Business

---

## 3. Multi-Factor Authentication (MFA)

### MFA Configuration Options

| Option | License | Scope | Management |
|--------|---------|-------|-----------|
| **Security Defaults** | Free | All users in tenant | Per-tenant toggle; opinionated baseline |
| **Per-User MFA** | Free | Individual users | Legacy; admin enables per user |
| **Conditional Access MFA** | **P1** | Granular (based on conditions) | Most flexible; recommended |
| **Identity Protection MFA risk policy** | **P2** | Risk-based (sign-in/user risk) | Automated, adaptive |

> **Exam tip:** Use Conditional Access (P1) for granular MFA control; Security Defaults is all-or-nothing

### MFA Registration Policy

| Policy | License | What it Does |
|--------|---------|-------------|
| **Combined registration** | Free | Users register MFA + SSPR in one flow |
| **MFA registration policy (ID Protection)** | P2 | Force all users to register; block old methods |

---

## 4. Conditional Access

### How Conditional Access Works

```
IF [Assignments — WHO and WHAT]
  + [Conditions — HOW and WHERE]
THEN [Access Controls — GRANT or BLOCK or SESSION]
```

### Assignments

| Assignment | Options |
|-----------|---------|
| **Users / Groups** | All users, specific groups, guests, roles |
| **Cloud Apps** | All apps, specific apps (e.g., Office 365) |
| **User Actions** | Register security info, register/join device |

### Conditions (Signals)

| Condition | Options | License |
|-----------|---------|---------|
| **Sign-in Risk** | Low, Medium, High (from Identity Protection) | **P2** |
| **User Risk** | Low, Medium, High | **P2** |
| **Device Platform** | iOS, Android, Windows, macOS | P1 |
| **Location** | Named locations, all locations, trusted IPs | P1 |
| **Client apps** | Browser, mobile apps, Exchange ActiveSync | P1 |
| **Device state** | Compliant, Hybrid Azure AD Joined | P1 |
| **Filter for devices** | Custom device attribute filters | P1 |

### Access Controls

| Control | Type | Description |
|---------|------|-------------|
| **Block access** | Grant | Deny all access |
| **Grant access** | Grant | Allow + optionally require: MFA, compliant device, domain-joined, approved app, app protection policy |
| **Require all selected controls** | Grant | AND logic (all must be satisfied) |
| **Require one of selected controls** | Grant | OR logic (any one sufficient) |
| **App enforced restrictions** | Session | Limit what user can do in-app (Office 365 only) |
| **Conditional Access App Control** | Session | Proxy via Microsoft Defender for Cloud Apps |
| **Sign-in frequency** | Session | Force re-auth after X hours |
| **Persistent browser session** | Session | Control "stay signed in" behavior |

### Key CA Policy Patterns (High Exam Relevance)

| Scenario | CA Policy Design |
|----------|----------------|
| Require MFA for all admins | Users = Admin roles; Grant = Require MFA |
| Require compliant device for corporate apps | Apps = All; Grant = Require compliant device OR HAADJ |
| Block access from untrusted locations | Locations = outside Named Locations; Grant = Block |
| Require phishing-resistant MFA for high-risk users | User risk = High; Grant = Require auth strength (phishing-resistant) |
| Limit external guest access to specific apps | Users = Guests; Apps = specific apps; Grant = Require MFA |
| Protect Azure management (portal, ARM, CLI) | Apps = Microsoft Azure Management; Grant = Require MFA |

> **License requirement:** Conditional Access = **P1** minimum. Risk-based conditions = **P2**
> **Exam tip:** "Sign-in Risk = High → require MFA or block" requires **P2** (Identity Protection detects risk)

---

## 5. Privileged Identity Management (PIM)

> **License: Entra ID P2** (or Microsoft 365 E5)

### What PIM Does

| Capability | Description |
|-----------|-------------|
| **Just-in-time (JIT) access** | Eligible users activate roles only when needed (time-limited) |
| **Approval workflows** | Role activation can require approver sign-off |
| **Justification** | Users must provide reason when activating |
| **Notification** | Alerts sent on activation, expiry, and anomalous use |
| **Access reviews** | Periodic review of who has standing access |
| **Audit history** | Full log of all role activations and changes |

### Role States in PIM

| State | Description | Activation Required |
|-------|-------------|-------------------|
| **Eligible** | User can activate the role | ✅ Yes (JIT) |
| **Active** | User permanently has the role | ❌ No (standing access) |
| **Expired** | Eligible or active assignment has lapsed | — |

> **Exam tip:** PIM makes privileged roles **eligible** (not active) — users activate when needed, role expires automatically

### PIM Scope

| Scope | What Can Be Managed |
|-------|-------------------|
| **Entra Id Roles** | Directory roles (Global Admin, User Admin, etc.) |
| **Azure Resource Roles** | Subscription Owner, Contributor, Key Vault Admin, etc. |
| **Groups** | Privileged access groups (nested role elevation) |

### PIM vs Direct Role Assignment

| | Direct Assignment | PIM Eligible |
|-|------------------|-------------|
| Access | Permanent (always active) | JIT only |
| Risk | High (standing privilege) | Low |
| Auditability | Basic | Full |
| Approval | ❌ | ✅ Optional |
| License req | None | **P2** |
| Best for | Break-glass accounts | All other privileged access |

> **Exam tip:** "Emergency break-glass accounts" should be **Permanent Active** (not eligible) with PIM — so they work even if PIM is down

---

## 6. Identity Protection

> **License: Entra ID P2**

### Risk Types

| Risk Type | Signals | Examples |
|-----------|---------|---------|
| **Sign-in Risk** | Suspicious sign-in properties | Atypical travel, anonymous IP, malware-linked IP, unfamiliar location |
| **User Risk** | Compromised account indicators | Leaked credentials, password spray attack |

### Risk Levels

| Level | Meaning | Typical Action |
|-------|---------|---------------|
| **Low** | Some suspicious signals | Monitor or require MFA |
| **Medium** | Likely compromise | Require MFA |
| **High** | Strong compromise indicator | Block or require password reset |

### Identity Protection Policies (Recommend Using CA instead)

| Policy | Trigger | Recommended Action |
|--------|---------|------------------|
| **User Risk Policy** | User risk ≥ Medium/High | Require secure password change |
| **Sign-in Risk Policy** | Sign-in risk ≥ Medium/High | Require MFA |
| **MFA Registration Policy** | New users | Force MFA registration within 14 days |

> **Exam tip:** Microsoft now recommends configuring risk-based access via **Conditional Access** (not the legacy Identity Protection policies), but both are valid answers. CA + IP gives more flexibility.

### Risky Users & Risky Sign-ins Reports

| Report | Shows | Action Available |
|--------|-------|-----------------|
| **Risky users** | Users flagged at risk | Confirm compromise, dismiss, reset password |
| **Risky sign-ins** | Sign-ins flagged as risky | Confirm safe, confirm compromise |
| **Risk detections** | Individual detection events | View details, dismiss |

---

## 7. Access Reviews

> **License: Entra ID P2**

### What Access Reviews Do

- **Periodically validate** that users/guests still need their access
- Reviewers can be: the user themselves (self-review), manager, resource owner, or designated reviewer
- **If no response:** access is automatically removed (or kept, based on policy)

### Review Scopes

| Scope | What Is Reviewed |
|-------|----------------|
| **Group membership** | Who is in a group (and gets access via that group) |
| **Application access** | Who has access to an enterprise app |
| **Entra ID role assignment** | Who holds a directory role |
| **Azure resource role (via PIM)** | Who holds an Azure resource role |

### Key Settings

| Setting | Options | Notes |
|---------|---------|-------|
| **Review frequency** | Once, weekly, monthly, quarterly, semi-annually, annually | — |
| **Duration** | 1–365 days | Window for completing reviews |
| **Auto-apply results** | On/Off | Auto-remove denied access or keep for manual action |
| **On no response** | No change / Remove access / Approve access / Recommendations | Typically set to "Remove access" |
| **Auto-review recommendations** | AI-based: approve if active sign-in, deny if inactive | P2 |

> **Exam tip:** Access Reviews = P2. "User hasn't signed in for 90 days" → AI recommendation = **Deny**

---

## 8. Entitlement Management

> **License: Entra ID P2** | Part of **Entra ID Governance**

### What It Does

| Capability | Description |
|-----------|-------------|
| **Access packages** | Bundle of resources (groups, apps, SharePoint sites, roles) |
| **Catalogs** | Containers for access packages; delegate management |
| **Policies** | Who can request, approval workflow, expiration |
| **Connected orgs** | Allow B2B users from specific tenants to request access |
| **Lifecycle management** | Auto-expire, auto-extend, reviews on expiry |

### Use Case

> "External partners need access to a specific SharePoint site, Teams group, and app for 90 days" → **Access Package** with connected org policy and 90-day expiration

---

## 9. B2B vs B2C — External Identities

| | **B2B (Entra External ID - B2B)** | **B2C (Azure AD B2C)** |
|-|----------------------------------|----------------------|
| **Purpose** | Partner/vendor/employee from another org | Customer-facing consumer identities |
| **Who are the users?** | Business partner users, vendors, guests | End consumers / customers |
| **Identity providers** | Any Entra ID tenant, Google, Facebook (via federation), email OTP | Google, Facebook, Twitter, Apple, local accounts, any OpenID Connect |
| **Tenant type** | Same Entra ID tenant (guests) | **Separate B2C tenant** |
| **Admin control** | Organization manages policies | Organization manages all customer identities |
| **License** | Free (up to 50,000 MAU) then metered | Per MAU — free up to 50,000/month, then ~$0.0016/MAU |
| **Use case** | Contractor accessing internal apps, partner portals | Customer login for retail app, public web app |
| **MFA** | Via Conditional Access | Built-in or custom policies |
| **Branding** | Limited | Full custom branding, custom UX (user flows) |

> **Exam tip:** "Customer-facing app" → B2C | "Partners/vendors in external org" → B2B

---

## 10. Hybrid Identity

### Authentication Options

| Method | How It Works | Passwords in Cloud? | Requires On-prem Component | Resilience |
|--------|-------------|--------------------|-----------------------------|-----------|
| **Password Hash Sync (PHS)** | Hash of hash synced to Entra ID | ✅ Yes (hash only) | AD Connect server | High (cloud auth if on-prem down) |
| **Pass-through Authentication (PTA)** | Auth request forwarded to on-prem AD | ❌ No | AD Connect + PTA Agent | Medium (needs agent up) |
| **Federated (ADFS)** | Entra ID redirects to ADFS for auth | ❌ No | ADFS farm + proxy | Low (ADFS single point of failure risk) |

### AD Connect vs Cloud Sync

| Feature | **Azure AD Connect (Sync)** | **Azure AD Connect Cloud Sync** |
|---------|---------------------------|--------------------------------|
| Architecture | On-prem sync service | Lightweight cloud provisioning agent |
| Agent weight | Heavy (SQL-based) | Lightweight (no on-prem DB) |
| Multiple forests | ✅ Full support | ✅ (limited topologies) |
| Device writeback | ✅ | ❌ |
| Exchange hybrid writeback | ✅ | ❌ |
| Password hash sync | ✅ | ✅ |
| Password writeback | ✅ | ✅ |
| Group writeback | ✅ v2 | ✅ (limited) |
| HA/redundancy | Single server + staging mode | Multiple agents (auto-HA) |
| Best for | Complex environments, full feature set | Simple, distributed forests, easier HA |
| License | Free | Free |

> **Exam tip:** "Distributed AD forest across 3 geographies, minimize on-prem infrastructure" → **Cloud Sync**
> "Need device writeback or Exchange hybrid management" → **AD Connect (full)**

### Sync Topologies

| Topology | Supported? | Notes |
|----------|-----------|-------|
| Single forest → Single tenant | ✅ | Standard |
| Multiple forests → Single tenant | ✅ | All forest objects merged |
| Single forest → Multiple tenants | ❌ | Not supported (object sync to one tenant only) |
| Multiple forests → Multiple tenants | ✅ (one forest per tenant) | No object overlap |
| Staging mode | ✅ | Secondary server does not export; used for DR/testing |

### Application Proxy

| Feature | Detail |
|---------|--------|
| **Purpose** | Publish on-prem web apps via Entra ID (remote access without VPN) |
| **How** | Connector installed on-prem server; no inbound firewall ports required |
| **Authentication** | Pre-authentication via Entra ID (SSO via Kerberos Constrained Delegation) |
| **Protocol** | HTTP/HTTPS (not for RDP/SSH — use Bastion for that) |
| **License** | **P1** required |
| **Use case** | Corporate intranet, legacy on-prem web apps for remote workers |

---

## 11. Azure AD Domain Services (AADDS)

| Feature | Detail |
|---------|--------|
| **What it is** | Managed, cloud-hosted Active Directory domain services (LDAP, Kerberos, NTLM, Group Policy) |
| **Purpose** | Apps in Azure VMs needing domain join, LDAP, Kerberos without on-prem AD dependency |
| **Management** | Microsoft manages DCs (you don't access DCs directly) |
| **Sync** | One-way sync FROM Entra ID → AADDS (not back to Entra ID or on-prem AD) |
| **SKUs** | Standard, Enterprise, Premium (differ by object count and backup frequency) |
| **HA** | Enterprise/Premium: replica sets in additional regions |
| **Backup** | Automatic; frequency depends on SKU |
| **License** | Paid (charged per hour based on SKU) |
| **vs AD on VM** | AADDS = managed PaaS; AD on VM = self-managed IaaS |

### AADDS SKUs

| SKU | Max Object Count | Backup Frequency | Replica Sets | Cost |
|-----|-----------------|-----------------|------------|------|
| **Standard** | 25,000 | Every 5 days | 1 (single region) | Low |
| **Enterprise** | 100,000 | Every 3 days | Up to 3 (multi-region) | Medium |
| **Premium** | 500,000 | Daily | Up to 5 (multi-region) | High |

> **Exam tip:** "Legacy app needs LDAP/Kerberos but no on-prem servers" → **AADDS**
> AADDS is NOT a replacement for on-prem AD — it's a one-way sync; on-prem can't rely on AADDS for auth

---

## 12. RBAC (Role-Based Access Control)

### Role Components

| Component | Description |
|-----------|-------------|
| **Security principal** | Who: User, Group, Service Principal, Managed Identity |
| **Role definition** | What: Permissions (Actions, NotActions, DataActions, NotDataActions) |
| **Scope** | Where: Management Group, Subscription, Resource Group, Resource |
| **Role assignment** | Binding of principal + role + scope |

### Scope Hierarchy (Inheritance flows downward)

```
Management Group (broadest)
  └── Subscription
        └── Resource Group
              └── Resource (narrowest)
```

> Permissions assigned at a higher scope **inherit** to all child scopes
> Deny assignments (via Blueprints) take precedence over allow

### Key Built-in Roles

| Role | Permissions | Can Assign Roles? |
|------|------------|-----------------|
| **Owner** | Full access to all resources | ✅ Yes |
| **Contributor** | Full access, no authorization | ❌ No |
| **Reader** | View only | ❌ No |
| **User Access Administrator** | Manage access, no resource operations | ✅ Yes |
| **Resource Policy Contributor** | Create/manage policies | ❌ No |

> **Owner vs Contributor:** Owner CAN assign/remove roles; Contributor CANNOT — high exam frequency

### Custom Roles

| Feature | Detail |
|---------|--------|
| **Created at** | Subscription or Management Group scope |
| **Built from** | JSON role definition (Actions, NotActions, DataActions, AssignableScopes) |
| **Limit** | 5,000 custom roles per tenant |
| **Use case** | When no built-in role matches least privilege requirement |

### RBAC Best Practices

| Practice | Why |
|----------|-----|
| Assign roles to **groups**, not individual users | Easier management |
| Use **least privilege** principle | Minimize blast radius |
| Prefer **built-in roles** over custom | Simpler, Microsoft-maintained |
| Assign at **resource group scope**, not subscription | More targeted |
| Use **PIM** for privileged roles (P2) | JIT access, audit trail |
| Review with **Access Reviews** (P2) | Remove stale assignments |

> **Exam tip:** "How to grant a team access to all resources in a specific environment without giving subscription access" → Assign Contributor at **Resource Group scope**

---

## 13. Governance — Management Groups & Subscriptions

### Management Group Hierarchy

```
Tenant Root Group (one per tenant, auto-created)
  └── Management Groups (custom; up to 6 levels deep below root)
        └── Subscriptions
              └── Resource Groups
                    └── Resources
```

| Fact | Value |
|------|-------|
| Max depth | **6 levels** below root (root + 6 = 7 total) |
| Max children per MG | 10,000 subscriptions or child MGs |
| Max MGs per tenant | 10,000 |
| Subscriptions per tenant | 10,000+ |
| Supported inheritance | Policy, RBAC, Blueprints |
| Moving subscriptions | ✅ between MGs (takes effect immediately) |

### Subscription Design Patterns

| Pattern | When to Use |
|---------|------------|
| **Single subscription** | Small orgs, dev/test, simple workloads |
| **Subscription per environment** | Dev / Staging / Production isolation |
| **Subscription per business unit** | Cost accountability, policy separation |
| **Subscription per region** | Regional compliance/governance |
| **Subscription per workload (Prod + Non-prod)** | Enterprise scale; Cloud Adoption Framework recommended |

> **Subscription limits are a design driver:** 800 resource groups, 980 deployments/RG/hour, etc.

### Management Group Common Structure (Enterprise)

```
Tenant Root
  ├── Platform MG
  │     ├── Identity Subscription (AADDS, DNS)
  │     ├── Management Subscription (Monitor, Backup)
  │     └── Connectivity Subscription (Hub VNet, Firewall, ExpressRoute)
  └── Landing Zones MG
        ├── Corp MG → Connected workloads (to Hub)
        └── Online MG → Internet-facing workloads
```

> This is the **Azure Landing Zone** structure (CAF-aligned) — know this hierarchy for the exam

---

## 14. Azure Policy

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Policy Definition** | Rule that evaluates resource properties (JSON format) |
| **Policy Assignment** | Apply a definition to a scope (MG, Sub, RG, Resource) |
| **Policy Initiative (Policy Set)** | Collection of policy definitions grouped together |
| **Compliance** | % of resources compliant with assigned policies |
| **Remediation Task** | Fix existing non-compliant resources using DeployIfNotExists/Modify policies |
| **Exemption** | Exclude specific scope/resource from policy evaluation |

### Policy Effect Types

| Effect | Behavior | Timing | Use Case |
|--------|----------|--------|---------|
| **Deny** | Block non-compliant resource creation/update | At request time | Enforce standards |
| **Audit** | Allow but flag as non-compliant | At evaluation time | Track without blocking |
| **Append** | Add fields to resource during create/update | At request time | Add required tags/IPs |
| **Modify** | Add/update/remove tags or resource properties | At request time or via remediation | Tag governance |
| **DeployIfNotExists (DINE)** | Deploy related resource if it doesn't exist | After resource creation | Install agents, enable diagnostics |
| **AuditIfNotExists** | Audit if a related resource doesn't exist | At evaluation time | Flag VMs without backup |
| **Disabled** | Policy turned off | — | Testing |

> **Exam tip — Effect precedence:** Deny > Append > Audit (if multiple policies apply to same resource)
> **DeployIfNotExists** is the most powerful — can auto-deploy related resources (e.g., enable monitoring agent on VM)

### Common Built-in Policy Examples

| Policy | Effect | Use Case |
|--------|--------|---------|
| Allowed locations | Deny | Enforce data residency |
| Require a tag and its value | Deny/Append | Tag governance |
| Allowed VM SKUs | Deny | Control VM sizes |
| Deploy Log Analytics agent | DINE | Auto-install monitoring |
| Storage accounts should use private endpoints | Audit/Deny | Security |
| SQL DB should have Azure Defender enabled | AuditIfNotExists | Security |
| Require HTTPS on App Service | Deny | Security |
| Not allowed resource types | Deny | Restrict services used |

### Policy Compliance Evaluation Triggers

| Trigger | When |
|---------|------|
| Resource create/update/delete | Real-time |
| Policy assigned to scope | Within 30 minutes |
| On-demand compliance scan | Manual trigger |
| Scheduled evaluation | Every 24 hours |

> Policy does NOT retroactively fix resources — use **Remediation Tasks** for that

---

## 15. Azure Blueprints

> ⚠️ **Note:** Azure Blueprints is in retirement — Microsoft is transitioning to **Azure Deployment Stacks** and **Template Specs**. It still appears on the exam.

### What Blueprints Include (Artifacts)

| Artifact Type | Description |
|---------------|-------------|
| **Role Assignments** | RBAC role applied to scope |
| **Policy Assignments** | Azure Policy applied to scope |
| **ARM Templates** | Deploy resources |
| **Resource Groups** | Create and configure resource groups |

### Blueprints vs ARM Templates vs Policy

| | Blueprints | ARM Templates | Policy |
|-|-----------|--------------|--------|
| Deploys resources | ✅ | ✅ | ❌ |
| Applies RBAC | ✅ | ❌ | ❌ |
| Applies Policy | ✅ | ❌ | ✅ |
| Tracks deployment | ✅ (versioned) | ❌ | ❌ |
| Locks resources | ✅ | ❌ | ❌ |
| Used for | Governance environments | IaC deployment | Compliance |

### Blueprint Lock Modes

| Lock Mode | Who Can Change | Use Case |
|-----------|---------------|---------|
| **Don't Lock** | Anyone with permissions | Default |
| **Do Not Delete** | Owner can delete manually | Protect key resources |
| **Read Only** | Cannot modify or delete | Strictest compliance |

> **Exam tip:** Blueprint **Read Only** lock CANNOT be removed by anyone except the Blueprint assignment itself — overrides Owner permissions

---

## 16. Azure Landing Zones

### Definition & Purpose

An **Azure Landing Zone** is a pre-configured, secure, scalable subscription environment based on prescriptive best practices (Azure Cloud Adoption Framework — CAF). It provides:

| Component | What It Provides |
|-----------|----------------|
| **Management Groups** | Organizational hierarchy for policy inheritance |
| **Azure Policy** | Guardrails (enforcement + audit) at each MG level |
| **RBAC** | Pre-configured role assignments per scope |
| **Networking** | Hub VNet, connectivity subscription (ExpressRoute/VPN) |
| **Monitoring** | Centralized Log Analytics workspace, Defender for Cloud |
| **Identity** | Identity subscription (AADDS or AD Connect) |
| **Cost management** | Budgets, alerts |

### Landing Zone Types

| Type | Description |
|------|-------------|
| **Platform Landing Zone** | Shared services (Identity, Management, Connectivity subscriptions) |
| **Application Landing Zone** | Workload subscriptions in Corp or Online MGs |

### Key Landing Zone Design Decisions

| Decision | Options |
|----------|---------|
| **Networking topology** | Hub-and-spoke OR Azure Virtual WAN |
| **DNS strategy** | Azure DNS Private Zones OR custom DNS servers in hub |
| **Identity** | AADDS OR AD Connect to cloud OR Entra ID-only |
| **Subscription vending** | Manual OR automated (Azure DevOps / GitHub + Terraform/Bicep) |
| **Policy enforcement** | Deny critical policies; Audit for informational |

> **Exam tip:** "Greenfield deployment, enterprise-scale, need consistent governance" → **Azure Landing Zone** (CAF-based)

---

## 17. Resource Locks & Tags

### Resource Locks

| Lock Type | Effect | Who Can Override |
|-----------|--------|-----------------|
| **CanNotDelete** | Read + Update allowed; Delete blocked | Owner of parent scope |
| **ReadOnly** | Read only; No update or delete | Owner of parent scope (must remove lock first) |

| Scope | Inheritance |
|-------|------------|
| Subscription | Applies to all RGs and resources within |
| Resource Group | Applies to all resources within |
| Resource | Applies to that resource only |

> **Exam tip:** **ReadOnly lock** on a Resource Group blocks creating new resources in it AND moving resources into it (write operation)
> Locks apply regardless of RBAC role — even **Owners** are blocked

### Resource Tags

| Feature | Detail |
|---------|--------|
| **Format** | Key:Value pairs (e.g., Environment:Production) |
| **Max tags per resource** | 50 |
| **Max tag name length** | 512 characters (128 for storage accounts) |
| **Inheritance** | Tags do NOT automatically inherit from parent scopes |
| **Enforce via Policy** | Use **Append** or **Modify** effect policies |
| **Cost allocation** | Tags enable cost reporting by team/project/environment |

> **Exam tip:** Tags don't inherit — if you want child resources tagged, use Azure Policy (Modify/Append effect)

---

## 18. Cost Management & FinOps

### Azure Cost Management Tools

| Tool | Purpose | Where |
|------|---------|-------|
| **Cost Analysis** | Visualize and analyze spend breakdown | Cost Management + Billing |
| **Budgets** | Set spending thresholds; trigger alerts or actions | Cost Management |
| **Cost Alerts** | Budget alerts, anomaly alerts, credit alerts | Cost Management |
| **Azure Advisor** | Recommendations to reduce cost | Advisor portal |
| **Pricing Calculator** | Estimate costs before deploying | azure.microsoft.com/pricing |
| **TCO Calculator** | Compare on-prem vs Azure costs | Microsoft website |
| **Reservations** | Commit to 1/3-year for savings | Reservations blade |
| **Savings Plans** | Flexible commitment (any compute) | Savings Plans blade |

### Cost Optimization Patterns

| Strategy | Discount | Requirement |
|----------|---------|-------------|
| **Reserved Instances (VM, SQL, etc.)** | 40–72% | 1 or 3-year commit to specific SKU/region |
| **Savings Plans (Compute)** | Up to 65% | 1 or 3-year spend commitment (flexible across VM types) |
| **Spot VMs** | Up to 90% | Interruptible workloads only |
| **Azure Hybrid Benefit (AHUB)** | ~40% | Own Windows Server or SQL Server license (SA required) |
| **Dev/Test subscription** | 15–55% | Non-production only |
| **Right-sizing** | Varies | Use Advisor + Monitor data |
| **Auto-shutdown (VMs)** | Significant for dev | Non-production VMs |
| **Lifecycle management (Storage)** | Varies | Move blobs to cooler tiers automatically |
| **Reserved Capacity (Cosmos DB)** | Up to 65% | Predictable RU workloads |
| **Reserved Capacity (SQL DB)** | Up to 80% | Predictable DTU/vCore workloads |

### Budget Actions

| Action Type | What Triggers | Example |
|-------------|--------------|---------|
| **Email alert** | % of budget reached | Notify team at 80% and 100% |
| **Action Group alert** | % of budget | Trigger runbook/function |
| **Azure Resource Action** | Forecasted or actual threshold | Auto-shutdown VMs when forecast hits 100% |

---

## 19. Azure Monitor

### Platform Overview

```
Data Sources                 Azure Monitor                  Destinations
──────────────               ───────────────                ──────────────
VMs, Apps                    ┌─────────────────┐            Log Analytics Workspace
Azure Resources    ────────▶ │  Metrics Store  │ ────────▶  Storage Account
Containers                   │  Logs Store      │            Event Hubs (SIEM)
Custom sources               └─────────────────┘            Partner solutions
```

### Key Components

| Component | Data Type | Retention (Default) | Query Language | Cost |
|-----------|----------|-------------------|----------------|------|
| **Metrics** | Numeric time-series | 93 days | Metrics Explorer (portal) | Free (platform metrics) |
| **Platform Logs (Activity Log)** | JSON events | 90 days (portal); archive to LA for longer | KQL | Free |
| **Resource Logs (Diagnostic Logs)** | Service-specific logs | Per destination | KQL (if sent to LA) | Per GB (LA ingestion) |
| **Log Analytics Workspace** | All log types | 30 days interactive (configurable) | **KQL** | Per GB ingested + retention |
| **Application Insights** | APM telemetry | 90 days | KQL | Per GB (8 GB/month free) |

### Diagnostic Settings — Routing Options

| Destination | Use Case | Cost |
|-------------|---------|------|
| **Log Analytics Workspace** | Centralized querying, alerts, dashboards | Per GB ingested |
| **Storage Account (Archive)** | Long-term compliance retention (cheap) | Per GB stored |
| **Event Hubs** | Stream to external SIEM (Splunk, Sentinel raw feed) | Per event |
| **Partner Solution** | Direct to certified partner tools | Per partner pricing |

> **Exam tip:** Most solutions combine: LA for query/alerts + Storage for cheap long-term retention + Event Hubs for SIEM integration

### Azure Monitor Workbooks

| Feature | Detail |
|---------|--------|
| **Purpose** | Create rich, interactive dashboards combining metrics + logs + custom visualizations |
| **Templated** | Many built-in templates (VM Insights, Network, Performance) |
| **Sharing** | Share via Azure portal; save to Log Analytics Workspace |
| **Use case** | Operational dashboards, executive reports, capacity planning |

---

## 20. Log Analytics Workspace

### Workspace Design Considerations

| Factor | Recommendation |
|--------|--------------|
| **Single vs multiple workspaces** | Single workspace per organization (simpler, cheaper, cross-resource queries) |
| **When to use multiple** | Data sovereignty (different regions), security boundary, chargeback requirements |
| **Workspace region** | Co-locate with primary resources (reduce egress) |
| **Access control** | Workspace-level RBAC OR resource-level RBAC (table-level) |

### Pricing Tiers

| Tier | Price Model | Savings | Min Daily Volume |
|------|------------|---------|-----------------|
| **Pay-as-you-go** | Per GB ingested | Baseline | Any |
| **Commitment: 100 GB/day** | Fixed rate | ~15% | 100 GB |
| **Commitment: 200 GB/day** | Fixed rate | ~20% | 200 GB |
| **Commitment: 300 GB/day** | Fixed rate | ~22% | 300 GB |
| **Commitment: 400 GB/day** | Fixed rate | ~25% | 400 GB |
| **Commitment: 500 GB/day** | Fixed rate | ~30% | 500 GB |
| **Commitment: 2000+ GB/day** | Fixed rate | Up to 52% | 2,000+ GB |

### Data Retention & Archiving

| Type | Duration | Cost |
|------|---------|------|
| **Interactive retention** | 30–730 days (configurable) | Full query cost |
| **Archive** | Up to 12 years total | ~90% cheaper; no interactive query |
| **Restore from Archive** | Bring data back to interactive | Query cost during restore |
| **Search Jobs** | Query archived data without full restore | Per GB searched |

### KQL — Key Query Patterns (Exam-relevant)

```kql
// Get failed logins in last hour
SigninLogs
| where TimeGenerated > ago(1h)
| where ResultType != "0"
| summarize count() by UserPrincipalName

// VM CPU over 90% in last 24h
Perf
| where TimeGenerated > ago(24h)
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| where CounterValue > 90

// Count of critical security alerts
SecurityAlert
| where AlertSeverity == "High"
| summarize count() by AlertName

// Resources without a specific tag
Resources
| where tags["Environment"] == ""
```

---

## 21. Application Insights

### What It Monitors

| Category | What It Captures |
|----------|----------------|
| **Availability** | Synthetic ping/multi-step tests from Azure PoPs globally |
| **Performance** | Response time, server response time, page load time |
| **Failures** | Exception rate, failed request rate, dependency failures |
| **Users** | User flows, session analytics, retention |
| **Custom Events** | Application-defined business events via SDK |
| **Dependency tracking** | SQL queries, HTTP calls to downstream services, queue calls |
| **Live Metrics** | Real-time stream of telemetry (1-second granularity) |

### Instrumentation Methods

| Method | Languages | Auto-collect | Code Changes |
|--------|----------|-------------|-------------|
| **Auto-instrumentation (codeless)** | .NET, Java, Node.js, Python | Most telemetry | ❌ None |
| **OpenTelemetry SDK** | All major languages | Full control | ✅ Code changes |
| **Classic SDK (legacy)** | .NET, Java, Node.js | Full | ✅ Code changes |

### Availability Tests

| Test Type | Description | Frequency | Cost |
|-----------|-------------|-----------|------|
| **URL ping test** | Simple HTTP GET check from 5 PoPs | 1–10 min | Free |
| **Multi-step web test** | Record browser flow | 5–10 min | Included |
| **Custom track availability** | Call `TrackAvailability()` from your code | Custom | Included |
| **Standard test** | Single URL with SSL validation + timeout | 1–15 min | Cost per test |

### Smart Detection (Proactive Alerts)

| Detection | What It Finds |
|-----------|-------------|
| **Failure Anomalies** | Unusual spike in failed requests (ML-based) |
| **Performance Anomalies** | Response time degradation compared to baseline |
| **Trace Degradation** | Increase in exception severity in traces |
| **Memory Leak** | Gradual increase in process memory |

> **Exam tip:** App Insights **workspace-based mode** = links to Log Analytics Workspace (required for new instances; enables combined KQL queries across app + infra)

---

## 22. Azure Alerts & Action Groups

### Alert Rule Types

| Type | Signal Source | Example | License |
|------|-------------|---------|---------|
| **Metric Alert** | Azure Monitor Metrics | CPU > 90% for 5 min | Free (metric) |
| **Log Alert (KQL)** | Log Analytics / App Insights | Error count > 50 in 10 min | Per query |
| **Activity Log Alert** | Azure Activity Log | VM deleted, policy assigned | Free |
| **Resource Health Alert** | Azure Resource Health | VM transitioning to Unavailable | Free |
| **Service Health Alert** | Azure Service Health | Azure outage / planned maintenance | Free |
| **Smart Detection Alert** | App Insights ML | Failure anomaly detected | Included with AI |

### Alert Severity Levels

| Severity | Label | Meaning |
|----------|-------|---------|
| Sev 0 | **Critical** | Service down or major impact |
| Sev 1 | **Error** | Significant issue |
| Sev 2 | **Warning** | Degraded or potential issue |
| Sev 3 | **Informational** | Noteworthy event |
| Sev 4 | **Verbose** | Diagnostic |

### Action Groups — Notification Types

| Action Type | Description |
|-------------|-------------|
| **Email / SMS / Push / Voice** | Human notification |
| **Webhook** | Call any HTTP endpoint |
| **Azure Function** | Trigger a function (automation) |
| **Logic App** | Trigger a workflow |
| **Automation Runbook** | Run Azure Automation script |
| **Azure DevOps Work Item** | Create work item in ADO |
| **ITSM connector** | ServiceNow, Remedy, etc. |
| **Event Hub** | Stream alert to Event Hub |

> Action Groups are **reusable** — one action group can be used by multiple alert rules

---

## 23. Microsoft Defender for Cloud

### Tiers

| Tier | Cost | What's Included |
|------|------|----------------|
| **CSPM Free (Foundational)** | Free | Security score, recommendations, basic compliance |
| **Defender CSPM (Paid)** | Per resource/month | Attack path analysis, data security posture, AI security, governance |
| **Workload Protection Plans** | Per resource/hr or /month | See table below |

### Workload Protection Plans

| Plan | Protects | Key Feature | Cost |
|------|---------|------------|------|
| **Defender for Servers P1** | Azure VMs + Arc-enabled servers | MDE integration, qualys VA | Per server/hr |
| **Defender for Servers P2** | Same + Multicloud | JIT VM access, FIM, 500 MB Log Analytics | Higher |
| **Defender for SQL** | Azure SQL DB, SQL MI, SQL on VM | Threat detection, VA scans | Per instance/hr |
| **Defender for Storage** | Storage accounts | Malware scanning, sensitive data discovery | Per storage/month |
| **Defender for Containers** | AKS, Arc K8s, container registries | Vuln scan on images, runtime protection | Per core/hr |
| **Defender for App Service** | App Service plans | Threat detection | Per ASP/hr |
| **Defender for Key Vault** | Key Vault operations | Anomalous access, suspicious operations | Per vault/hr |
| **Defender for DNS** | DNS queries from Azure resources | DNS exfiltration detection | Per subscription |
| **Defender for Resource Manager** | ARM operations | ARM-layer attack detection | Per subscription |

### Secure Score

| Concept | Description |
|---------|-------------|
| **Secure Score** | % of max points achieved across all recommendations |
| **Security Controls** | Groups of related recommendations (e.g., "Enable MFA") |
| **Max Score** | 100 (achieve all recommendations) |
| **Recommendations** | Specific actionable items with severity and effort |
| **Quick Fix** | One-click remediation for supported recommendations |
| **Exempt** | Suppress a recommendation for a scope (with justification) |

### JIT VM Access (Just-In-Time)

| Feature | Detail |
|---------|--------|
| **License** | Defender for Servers P2 |
| **Purpose** | Open RDP/SSH ports only when an authorized user requests access |
| **Duration** | Time-limited (default 3 hours, configurable) |
| **Network layer** | NSG rule auto-added for specific IP + port + duration, then removed |
| **Reduces attack surface** | Eliminates always-open management ports (top attack vector) |
| **Audit** | All access requests logged in Activity Log + Defender |

---

## 24. Azure Advisor

### Recommendation Categories

| Category | Focus | Examples |
|----------|-------|---------|
| **Cost** | Reduce spend | Rightsize/shutdown idle VMs, delete empty disks, purchase reservations |
| **Security** | Improve security posture | Enable MFA, rotate keys, enable Defender plans, close open ports |
| **Reliability** | Improve availability | Add Availability Zones, enable backup, add redundancy |
| **Performance** | Improve speed | Apply throttling policy (APIM), enable CDN, use Premium SSD |
| **Operational Excellence** | Improve operations | Enable diagnostics, use tags, fix subscription limits |

### Advisor Scores

| Feature | Detail |
|---------|--------|
| **Advisor Score** | Aggregate score (0–100) per category and overall |
| **Postpone/Dismiss** | Accept risk for specific recommendations |
| **Configure rules** | Set thresholds (e.g., CPU < X% = rightsize recommendation) |
| **Workbooks** | Advisor integrated with Monitor Workbooks for reporting |

---

## 25. Azure Service Health

### Components

| Component | What It Shows | Configured Via |
|-----------|-------------|---------------|
| **Azure Status** | Global Azure service outages (public page) | Public page: status.azure.com |
| **Service Health** | Personalized health for YOUR subscriptions + regions + services | Azure portal |
| **Resource Health** | Health of YOUR specific resources (VMs, SQL DBs, etc.) | Per resource |

### Service Health Event Types

| Type | Description | Action |
|------|-------------|--------|
| **Service Issues** | Active outage impacting Azure services | Monitor, plan failover |
| **Planned Maintenance** | Microsoft-scheduled updates | Plan maintenance windows |
| **Health Advisories** | Feature deprecations, breaking changes | Remediate before deadline |
| **Security Advisories** | Security issues affecting Azure platform | Immediate attention |

### Service Health Alerts (Best Practice)

```
Alert on:
  - Service Issues → Action Group → Email/SMS/PagerDuty/ServiceNow
  - Planned Maintenance → Email team leads
  - Health Advisories → Email architects
Scope: Your specific subscriptions/regions/services (not all Azure)
```

---

## 26. Exam Tips — Master List

### Identity Tips

| # | Tip | Common Trap |
|---|-----|------------|
| 1 | **Conditional Access requires P1** minimum | Using Security Defaults ≠ Conditional Access |
| 2 | **PIM requires P2** — JIT, approval, access reviews | P1 does NOT include PIM |
| 3 | **Identity Protection (risk detection) requires P2** | Sign-in risk conditions in CA need P2 |
| 4 | **Break-glass accounts** should be Permanent Active (not eligible) in PIM | Eligible accounts can't activate if PIM fails |
| 5 | **B2B = partners/vendors** in another org; **B2C = customers** of your product | Mixing B2B/B2C is a very common trap |
| 6 | **Managed Identity** = no credential management; always prefer over Service Principal with password | "App accesses Key Vault" = Managed Identity |
| 7 | **System-assigned MI** is deleted with the resource; **User-assigned MI** persists independently | Don't delete shared MI by deleting one VM |
| 8 | **Cloud Sync** cannot do device writeback or Exchange hybrid writeback | "Need device writeback" → AD Connect (full) |
| 9 | **AADDS = one-way sync** from Entra ID; on-prem AD cannot rely on it for auth | Common misconception |
| 10 | **Application Proxy** requires P1 and no inbound firewall ports | Not for RDP/SSH (that's Bastion) |

### Governance Tips

| # | Tip | Common Trap |
|---|-----|------------|
| 11 | **Tags do NOT inherit** from parent scopes | Must use Azure Policy (Append/Modify) to propagate tags |
| 12 | **Resource Locks override RBAC** — Owners can't delete ReadOnly locked resources | Lock must be removed before deletion |
| 13 | **Policy effects priority:** Deny > Append > Audit | If two policies apply, Deny wins |
| 14 | **DeployIfNotExists** requires a **Managed Identity** on the policy assignment (to deploy) | Missing MI = DINE won't work |
| 15 | **Policy compliance is NOT real-time retroactive** — existing resources evaluated on schedule | Use Remediation Tasks to fix existing |
| 16 | **Blueprint Read Only lock** cannot be overridden even by Owner — only Blueprint can remove | Stronger than resource lock |
| 17 | **Management Groups inherit** — policy at MG level applies to ALL descendant subscriptions | Good for broad governance guardrails |
| 18 | **Custom RBAC roles** — max 5,000 per tenant; created at subscription or MG scope | |
| 19 | **Owner** = full access + assign roles; **Contributor** = full access, NO role assignment | Most frequently tested RBAC distinction |
| 20 | **RBAC is additive** — effective permissions = union of all roles; Deny assignment is exception | Deny (from Blueprint) overrides RBAC allow |

### Monitoring Tips

| # | Tip | Common Trap |
|---|-----|------------|
| 21 | **Metrics retention** = 93 days; **Activity Log** = 90 days; **Log Analytics** = configurable | Don't assume logs are kept forever by default |
| 22 | **App Insights must use workspace-based mode** (new instances); links to Log Analytics | Classic mode is legacy |
| 23 | **Diagnostic Settings** route resource logs — not configured by default for most services | Logs don't go anywhere unless you configure destination |
| 24 | **Secure Score goes DOWN** if you add resources that have open recommendations | Score is % of best practices achieved |
| 25 | **JIT VM Access** = Defender for Servers P2; NOT in free tier | "Reduce attack surface for VMs" = JIT |
| 26 | **Azure Advisor recommendations** are subscription-scoped; Security is Defender for Cloud | Don't confuse Advisor with Defender for Cloud |
| 27 | **Alert rules and Action Groups** are separate — one Action Group reused by many alerts | Must create Action Group separately |
| 28 | **Service Health** is personalized to your subscriptions/regions; Azure Status = public global | "Notify when Azure service impacts MY resources" = Service Health alert |
| 29 | **Log Analytics Commitment tiers** = savings when > ~150–200 GB/day; calculate break-even | If < 100 GB/day, PAYG is often cheaper |
| 30 | **Resource Health** shows health of YOUR specific resource instance (not platform health) | Azure Status = global; Resource Health = your resource |

### Cross-Domain Design Tips

| # | Scenario | Answer |
|---|----------|--------|
| 31 | "Ensure all new VMs must use managed disks" | Azure Policy — **Deny** effect |
| 32 | "Automatically install Log Analytics agent on new VMs" | Azure Policy — **DeployIfNotExists** effect |
| 33 | "Notify when VM becomes unavailable" | **Resource Health Alert** via Azure Monitor |
| 34 | "Audit compliance without blocking deployments" | Azure Policy — **Audit** effect |
| 35 | "Require MFA only when sign-in risk is High" | **Conditional Access** with **Identity Protection** sign-in risk condition (P2) |
| 36 | "Quarterly review of who has Owner access to production" | **Access Reviews** in PIM (P2) |
| 37 | "Ensure team lead must approve before engineer activates Owner role" | **PIM** with **approval workflow** (P2) |
| 38 | "All workloads should have consistent tagging across environments" | Azure Policy + **Modify** effect (or **Append**) |
| 39 | "Control costs across 10 teams using shared Azure subscription" | **Management Groups** + **Budgets** per Resource Group + **Tags** for cost allocation |
| 40 | "External vendor needs access to 3 specific resources for 60 days" | **Entitlement Management** — Access Package with 60-day expiry (P2) |

---

## Quick Reference Summary Card

```
LICENSES:
  Free → Basic SSO, MFA, SSPR (cloud)
  P1   → Conditional Access, Dynamic Groups, App Proxy, SSPR writeback
  P2   → + PIM, Identity Protection, Access Reviews, Entitlement Management

HYBRID IDENTITY:
  PHS  → Simplest; passwords as hash in cloud; cloud auth when on-prem down
  PTA  → Auth forwarded to on-prem; no cloud passwords
  ADFS → Full federation; most complex; ADFS is single point of failure
  Cloud Sync → Lightweight agent; limited features; auto-HA

POLICY EFFECTS (priority if conflict):
  Deny > Append/Modify > AuditIfNotExists/DeployIfNotExists > Audit

RBAC KEY FACT:
  Owner = full + assign roles
  Contributor = full, NO role assignment
  Locks override RBAC (even Owner can't delete ReadOnly locked)

MONITORING RETENTION (defaults):
  Metrics = 93 days
  Activity Log = 90 days
  Log Analytics = 30 days interactive (configurable up to 12 years)
  App Insights = 90 days

WHEN TO USE:
  "Customer identity" → B2C
  "Partner/vendor" → B2B
  "App accessing Azure services without secrets" → Managed Identity
  "JIT admin access" → PIM (P2)
  "Sign-in risk auto-block" → CA + Identity Protection (P2)
  "Legacy app needs LDAP/Kerberos on Azure VMs" → AADDS
  "On-prem web app for remote workers" → Application Proxy (P1)
  "Auto-remediate non-compliant resources" → DINE policy + Remediation Task
  "Detect Azure outage affecting my subscription" → Service Health Alert
```

---

*Last updated: May 20, 2026 | Domain 1 coverage: 25–30% of AZ-305*
