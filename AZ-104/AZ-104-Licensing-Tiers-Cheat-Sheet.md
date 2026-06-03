# Azure & Microsoft 365 — Licensing, Tiers & Subscriptions Cheat Sheet

---

## Table of Contents
1. [Azure Subscription Types](#1-azure-subscription-types)
2. [Microsoft Entra ID (Azure AD) Tiers](#2-microsoft-entra-id-azure-ad-tiers)
3. [Microsoft 365 Enterprise Plans (E1/E3/E5)](#3-microsoft-365-enterprise-plans-e1e3e5)
4. [Microsoft 365 Frontline Plans (F1/F3)](#4-microsoft-365-frontline-plans-f1f3)
5. [Microsoft 365 Business Plans](#5-microsoft-365-business-plans)
6. [Office 365 vs Microsoft 365](#6-office-365-vs-microsoft-365)
7. [Entra ID P1 vs P2 — Feature Comparison](#7-entra-id-p1-vs-p2--feature-comparison)
8. [Microsoft Intune Plans](#8-microsoft-intune-plans)
9. [Azure Support Plans](#9-azure-support-plans)
10. [Azure Reserved Instances & Savings Plans](#10-azure-reserved-instances--savings-plans)
11. [Key Licensing Facts for Exams](#11-key-licensing-facts-for-exams)

---

## 1. Azure Subscription Types

| Subscription | Cost | Credit | Best For | Key Limits |
|-------------|------|--------|----------|-----------|
| **Free** | $0 | $200 (30 days) | Learning, POC | 12 months of free-tier services; limited to select services |
| **Pay-As-You-Go (PAYG)** | Usage-based | None | Small businesses, dev/test | No commitment; standard retail pricing |
| **Pay-As-You-Go Dev/Test** | Discounted | None | Dev/test workloads | Discounted rates on select services; no production SLA |
| **Visual Studio (MPN/MSDN)** | Monthly credit | $50-$150/mo | VS Enterprise/Pro subscribers | Dev/test only; cannot run production |
| **Enterprise Agreement (EA)** | Commitment | Varies | Large orgs (500+ seats) | 1-3 year term; volume discounts; custom pricing |
| **Microsoft Customer Agreement (MCA)** | Usage-based | None | Replacing EA for many orgs | Flexible billing; Azure Plan pricing |
| **CSP (Cloud Solution Provider)** | Partner-managed | None | Orgs buying through a partner | Partner provides support; partner manages billing |
| **Sponsorship** | Free | Varies | Microsoft-sponsored events, startups | Time-limited; specific programs only |
| **Student** | $0 | $100 (12 months) | Verified students | No credit card required; limited services |

### Azure Free Account — What's Included

| Category | Free for 12 Months | Always Free |
|----------|-------------------|-------------|
| **Compute** | 750 hrs B1S Windows/Linux VM | — |
| **Storage** | 5 GB Blob (LRS), 5 GB File | — |
| **Database** | 250 GB SQL Database (S0) | 1000 RU/s Cosmos DB (25 GB) |
| **Networking** | 15 GB outbound data transfer | — |
| **AI** | — | 5000 transactions/mo (Computer Vision) |
| **Functions** | — | 1M requests/month |
| **App Service** | — | 10 web apps (F1 tier) |
| **DevOps** | — | 5 users, unlimited private repos |

---

## 2. Microsoft Entra ID (Azure AD) Tiers

| Feature | Free | P1 | P2 |
|---------|------|----|----|
| **Included with** | Azure subscription | M365 E3, EMS E3 | M365 E5, EMS E5 |
| **Standalone price** | — | ~$6/user/mo | ~$9/user/mo |
| **User/Group management** | ✅ | ✅ | ✅ |
| **SSO (unlimited apps)** | ✅ (10 apps limit) | ✅ Unlimited | ✅ Unlimited |
| **B2B collaboration** | ✅ | ✅ | ✅ |
| **Self-service password reset (cloud)** | ❌ | ✅ | ✅ |
| **Self-service password reset (on-prem writeback)** | ❌ | ✅ | ✅ |
| **Conditional Access** | ❌ | ✅ | ✅ |
| **MFA** | Security defaults only | ✅ Full CA-based | ✅ Full CA-based |
| **Device management (Entra Join)** | ✅ Basic | ✅ + MDM auto-enroll | ✅ + MDM auto-enroll |
| **Application Proxy** | ❌ | ✅ | ✅ |
| **Dynamic Groups** | ❌ | ✅ | ✅ |
| **Group-based licensing** | ❌ | ✅ | ✅ |
| **Hybrid identities (Connect sync)** | ✅ | ✅ | ✅ |
| **Advanced group features** | ❌ | ✅ | ✅ |
| **Identity Protection (risk-based CA)** | ❌ | ❌ | ✅ |
| **Privileged Identity Management (PIM)** | ❌ | ❌ | ✅ |
| **Access Reviews** | ❌ | ❌ | ✅ |
| **Entitlement Management** | ❌ | ❌ | ✅ |
| **Risk-based sign-in policies** | ❌ | ❌ | ✅ |
| **Risk-based user policies** | ❌ | ❌ | ✅ |
| **Vulnerable accounts detection** | ❌ | ❌ | ✅ |
| **SLA** | ❌ | 99.99% | 99.99% |

### Quick Decision Guide

```
Need Conditional Access?              → P1 minimum
Need Self-service password reset?     → P1 minimum
Need PIM (Just-in-time admin)?        → P2
Need Identity Protection?             → P2
Need Access Reviews?                  → P2
Need risk-based Conditional Access?   → P2
```

---

## 3. Microsoft 365 Enterprise Plans (E1/E3/E5)

| Feature | E1 | E3 | E5 |
|---------|----|----|-----|
| **Price (approx.)** | ~$10/user/mo | ~$36/user/mo | ~$57/user/mo |
| **Target** | Knowledge workers (web/mobile) | Full desktop + security | Premium security + analytics |
| **Office Desktop Apps** | ❌ Web/mobile only | ✅ Full desktop | ✅ Full desktop |
| **Exchange Online** | ✅ (50 GB mailbox) | ✅ (100 GB mailbox) | ✅ (100 GB mailbox) |
| **SharePoint Online** | ✅ | ✅ | ✅ |
| **OneDrive** | ✅ (1 TB) | ✅ (Unlimited*) | ✅ (Unlimited*) |
| **Teams** | ✅ | ✅ | ✅ + Phone System |
| **Yammer / Viva Engage** | ✅ | ✅ | ✅ |
| **Stream** | ✅ | ✅ | ✅ |
| **Power Apps / Automate** | ✅ Limited | ✅ Limited | ✅ Limited |
| **Entra ID Plan** | Entra ID P1 | Entra ID P1 | Entra ID P2 |
| **Intune** | ❌ | ✅ (Plan 1) | ✅ (Plan 1) |
| **Information Protection (AIP)** | ❌ | ✅ (P1) | ✅ (P2 + auto-label) |
| **Data Loss Prevention (DLP)** | ❌ | ✅ | ✅ + Endpoint DLP |
| **eDiscovery** | ❌ | ✅ Standard | ✅ Premium |
| **Advanced Threat Protection** | ❌ | ❌ | ✅ Defender for O365 P2 |
| **Cloud App Security (MCAS)** | ❌ | ❌ | ✅ |
| **Defender for Endpoint** | ❌ | ❌ | ✅ (Plan 2) |
| **Audio Conferencing** | ❌ | ❌ | ✅ |
| **Phone System** | ❌ | ❌ | ✅ |
| **Power BI Pro** | ❌ | ❌ | ✅ |
| **Compliance Manager** | Basic | Standard | Premium |
| **Insider Risk Management** | ❌ | ❌ | ✅ |
| **Communication Compliance** | ❌ | ❌ | ✅ |
| **Retention policies** | Basic | ✅ Advanced | ✅ Advanced + auto-labeling |
| **Windows upgrade rights** | ❌ | ✅ Windows E3 | ✅ Windows E5 |

> **Note:** There is no Microsoft 365 E7 plan. Microsoft's enterprise tier maxes at E5. If you've seen "E7" referenced, it may be confusion with Office 365 F-plans or a rumored future SKU.

### What's Bundled (E3 vs E5 Key Differentiator)

```
E3 = Office Apps + Core Security + Compliance Basics + Entra P1
E5 = E3 + Advanced Security (Defender) + Advanced Compliance + Analytics + Entra P2
```

---

## 4. Microsoft 365 Frontline Plans (F1/F3)

For shift workers, retail, manufacturing, field service staff.

| Feature | F1 | F3 |
|---------|----|----|
| **Price (approx.)** | ~$2.25/user/mo | ~$8/user/mo |
| **Target** | Frontline workers (limited needs) | Frontline workers (more features) |
| **Office Web/Mobile Apps** | ✅ (view only + mobile) | ✅ (web + mobile editing) |
| **Office Desktop Apps** | ❌ | ❌ |
| **Exchange Online** | ❌ (no mailbox) | ✅ (2 GB mailbox) |
| **Teams** | ✅ | ✅ |
| **SharePoint** | ✅ (read-only) | ✅ |
| **OneDrive** | ❌ | ✅ (2 GB) |
| **Shifts (in Teams)** | ✅ | ✅ |
| **Walkie Talkie** | ✅ | ✅ |
| **Intune** | ❌ | ✅ |
| **Entra ID** | Entra P1 | Entra P1 |
| **Windows** | ❌ | ✅ (Windows E3) |

---

## 5. Microsoft 365 Business Plans

For small-to-medium businesses (≤ 300 users).

| Feature | Business Basic | Business Standard | Business Premium |
|---------|---------------|-------------------|------------------|
| **Price (approx.)** | ~$6/user/mo | ~$12.50/user/mo | ~$22/user/mo |
| **Max users** | 300 | 300 | 300 |
| **Office Web/Mobile** | ✅ | ✅ | ✅ |
| **Office Desktop Apps** | ❌ | ✅ | ✅ |
| **Exchange Online** | ✅ (50 GB) | ✅ (50 GB) | ✅ (50 GB) |
| **Teams** | ✅ | ✅ | ✅ |
| **SharePoint** | ✅ | ✅ | ✅ |
| **OneDrive** | ✅ (1 TB) | ✅ (1 TB) | ✅ (1 TB) |
| **Bookings** | ✅ | ✅ | ✅ |
| **Intune** | ❌ | ❌ | ✅ |
| **Entra ID** | Entra Free | Entra Free | Entra ID P1 |
| **Conditional Access** | ❌ | ❌ | ✅ |
| **Defender for Business** | ❌ | ❌ | ✅ |
| **DLP** | ❌ | ❌ | ✅ |
| **Information Protection** | ❌ | ❌ | ✅ |
| **Autopilot** | ❌ | ❌ | ✅ |
| **Sensitivity labels** | ❌ | ❌ | ✅ |

> **Key takeaway:** Business Premium ≈ E3 features at SMB scale (≤300 users)

---

## 6. Office 365 vs Microsoft 365

| | Office 365 | Microsoft 365 |
|-|------------|---------------|
| **Includes** | Productivity apps + cloud services | Office 365 + Windows + EMS (Security) |
| **Windows license** | ❌ | ✅ (E3/E5 plans) |
| **Intune/EMS** | ❌ | ✅ |
| **Status** | Being phased into M365 branding | Current branding |

### Legacy Office 365 Plans → Microsoft 365 Mapping

| Old Name | New Name / Equivalent |
|----------|----------------------|
| Office 365 E1 | Microsoft 365 E3 (minus desktop apps, minus EMS) |
| Office 365 E3 | Part of Microsoft 365 E3 (productivity portion) |
| Office 365 E5 | Part of Microsoft 365 E5 (productivity portion) |
| Office 365 F1 | Microsoft 365 F1 |

---

## 7. Entra ID P1 vs P2 — Feature Comparison

This is one of the **most tested comparisons** on Microsoft exams.

| Feature | P1 | P2 |
|---------|----|----|
| **Conditional Access** | ✅ | ✅ |
| **MFA (via Conditional Access)** | ✅ | ✅ |
| **Self-Service Password Reset** | ✅ | ✅ |
| **SSPR with on-prem writeback** | ✅ | ✅ |
| **Application Proxy** | ✅ | ✅ |
| **Dynamic Groups** | ✅ | ✅ |
| **Group-based licensing** | ✅ | ✅ |
| **Microsoft Entra Connect Health** | ✅ | ✅ |
| **Cloud App Discovery** | ✅ | ✅ |
| **Conditional Access: location/device/risk** | Location + Device only | ✅ All including **sign-in risk** |
| **Identity Protection** | ❌ | ✅ |
| **Risky sign-ins report** | ❌ | ✅ |
| **Risky users report** | ❌ | ✅ |
| **Risk-based Conditional Access** | ❌ | ✅ |
| **Privileged Identity Management (PIM)** | ❌ | ✅ |
| **Just-in-time (JIT) access** | ❌ | ✅ (via PIM) |
| **Access Reviews** | ❌ | ✅ |
| **Entitlement Management** | ❌ | ✅ |
| **Identity Governance (full)** | ❌ | ✅ |

### Memory Aid

```
P1 = "Control access" (Conditional Access, SSPR, Dynamic Groups, App Proxy)
P2 = P1 + "Protect & Govern identities" (PIM, Identity Protection, Access Reviews)
```

---

## 8. Microsoft Intune Plans

| Feature | Intune Plan 1 | Intune Plan 2 | Intune Suite |
|---------|--------------|--------------|-------------|
| **Included with** | M365 E3/E5, Business Premium | Add-on | Add-on |
| **MDM (device management)** | ✅ | ✅ | ✅ |
| **MAM (app management)** | ✅ | ✅ | ✅ |
| **Conditional Access (device compliance)** | ✅ | ✅ | ✅ |
| **Autopilot** | ✅ | ✅ | ✅ |
| **Configuration profiles** | ✅ | ✅ | ✅ |
| **Advanced endpoint analytics** | ❌ | ✅ | ✅ |
| **Tunnel for MAM** | ❌ | ❌ | ✅ |
| **Firmware-over-the-air** | ❌ | ❌ | ✅ |
| **Endpoint Privilege Management** | ❌ | ❌ | ✅ |
| **Remote Help** | ❌ | ❌ | ✅ |

---

## 9. Azure Support Plans

| Plan | Price | Scope | Response Time (Critical) | Technical Support |
|------|-------|-------|------------------------|-------------------|
| **Basic** | Free | All Azure customers | — | No technical support (billing + subscription only) |
| **Developer** | ~$29/mo | Trial/dev environments | 8 business hours (Sev C) | Email during business hours |
| **Standard** | ~$100/mo | Production workloads | **1 hour** (Sev A) | 24/7 phone + email |
| **Professional Direct** | ~$1,000/mo | Business-critical | **1 hour** (Sev A) + ProDirect delivery managers | 24/7 + advisory |
| **Premier / Unified** | Custom | Org-wide (all Microsoft products) | **15 min** (Sev A) + designated TAM | Full enterprise support |

### Severity Levels

| Severity | Description | Standard Response |
|----------|-------------|-------------------|
| **Sev A (Critical)** | Production down, complete loss of service | 1 hour |
| **Sev B (High)** | Significant business impact, degraded service | 4 hours |
| **Sev C (Moderate)** | Minor impact, workaround available | 8 business hours |

---

## 10. Azure Reserved Instances & Savings Plans

| Option | Commitment | Discount | Flexibility | Best For |
|--------|-----------|----------|-------------|----------|
| **Pay-As-You-Go** | None | 0% | Full flexibility | Unpredictable workloads |
| **Spot VMs** | None (can be evicted) | Up to 90% | Evictable anytime | Batch, fault-tolerant |
| **Reserved Instances (RI)** | 1 or 3 years | 40-72% | Specific VM size/region | Steady-state VMs |
| **Savings Plan (Compute)** | 1 or 3 years ($/hr) | 20-65% | Any VM size/region/OS | Variable compute needs |
| **Hybrid Benefit (AHUB)** | Existing license | 40-80% with RI | Windows/SQL licensing | Existing SA licenses |
| **Dev/Test Pricing** | VS subscription | ~50% off VMs | Dev/test only | Non-production |

### Reserved Instance Scopes

| Scope | Applies To |
|-------|-----------|
| **Single resource group** | Specific RG only |
| **Single subscription** | That subscription only |
| **Shared** | All subscriptions in billing account |
| **Management group** | All subscriptions in the MG |

---

## 11. Key Licensing Facts for Exams

### Must-Know Mappings

| You Need This Feature... | You Need This License |
|--------------------------|---------------------|
| Conditional Access | Entra ID **P1** (or M365 E3+) |
| PIM (Privileged Identity Management) | Entra ID **P2** (or M365 E5) |
| Identity Protection (risk-based CA) | Entra ID **P2** (or M365 E5) |
| Access Reviews | Entra ID **P2** (or M365 E5) |
| Self-Service Password Reset | Entra ID **P1** (or M365 E3+) |
| Dynamic Groups | Entra ID **P1** |
| Application Proxy | Entra ID **P1** |
| Intune (MDM/MAM) | M365 E3+ or Intune standalone |
| Defender for Endpoint P2 | M365 **E5** |
| Defender for Office 365 P2 | M365 **E5** |
| Microsoft Sentinel | Separate pricing (per GB ingested) |
| Power BI Pro | M365 **E5** (or standalone) |
| Phone System (Teams calling) | M365 **E5** (or add-on) |
| Audio Conferencing | M365 **E5** (or add-on) |
| eDiscovery Premium | M365 **E5** |
| Windows Enterprise upgrade | M365 E3+ (not E1) |
| Cloud App Security (MCAS/MDA) | M365 **E5** (or EMS E5) |
| Information Protection auto-labeling | M365 **E5** |

### Enterprise Mobility + Security (EMS) Add-On

| | EMS E3 | EMS E5 |
|-|--------|--------|
| **Entra ID** | P1 | P2 |
| **Intune** | Plan 1 | Plan 1 |
| **Information Protection** | P1 | P2 |
| **Advanced Threat Analytics** | ✅ | ✅ |
| **Cloud App Security** | ❌ | ✅ |
| **Defender for Identity** | ❌ | ✅ |

### Common Exam Traps

| Trap | Correct Answer |
|------|---------------|
| "Use Conditional Access with Free tier" | ❌ Requires **P1** |
| "PIM works with P1" | ❌ Requires **P2** |
| "E1 includes desktop Office apps" | ❌ E1 = web/mobile only |
| "Business plans support 500 users" | ❌ Max **300 users** (then must go Enterprise) |
| "Identity Protection = P1" | ❌ **P2** only |
| "Access Reviews available in P1" | ❌ **P2** only |
| "Entra ID Free includes Conditional Access" | ❌ Free = Security Defaults only |
| "Basic support includes phone support" | ❌ Basic = billing only; Standard+ for technical |
| "M365 E3 includes Defender for Endpoint" | ❌ **E5** (or add-on P2) |
| "Savings Plans lock you to a VM size" | ❌ Savings Plans are flexible; **RIs** are size-specific |

---

## Quick Reference — "Which Plan Do I Need?"

```
┌────────────────────────────────────────────────────────┐
│           IDENTITY FEATURE DECISION TREE               │
├────────────────────────────────────────────────────────┤
│                                                        │
│  Basic SSO + MFA (Security Defaults)?  → FREE         │
│                                                        │
│  Conditional Access?                   → P1            │
│  SSPR with writeback?                  → P1            │
│  Dynamic Groups?                       → P1            │
│  App Proxy?                            → P1            │
│                                                        │
│  PIM / JIT access?                     → P2            │
│  Identity Protection?                  → P2            │
│  Risk-based policies?                  → P2            │
│  Access Reviews?                       → P2            │
│  Entitlement Management?               → P2            │
│                                                        │
└────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────┐
│           M365 PLAN DECISION TREE                      │
├────────────────────────────────────────────────────────┤
│                                                        │
│  Web-only productivity?                → E1            │
│  Desktop Office + basic security?      → E3            │
│  Advanced security + compliance?       → E5            │
│                                                        │
│  Frontline (no desktop apps)?          → F1/F3        │
│  Small business (≤300 users)?          → Business     │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

*Part of AZ-104 Study Materials — see also [AZ-104-Identity-Governance.md](AZ-104-Identity-Governance.md) for related identity concepts.*
