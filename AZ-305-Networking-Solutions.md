# AZ-305 — Azure Networking Solutions
## Complete Study Guide: Services, Tiers, HA, DR, Comparisons & Exam Tips
> Exam: May 21, 2026 | Relevant Domain: Infrastructure Solutions (30–35%)

---

## Table of Contents
1. [Virtual Networks (VNets)](#1-virtual-networks-vnets)
2. [VNet Peering](#2-vnet-peering)
3. [Network Security Groups (NSGs)](#3-network-security-groups-nsgs)
4. [User-Defined Routes (UDRs)](#4-user-defined-routes-udrs)
5. [Azure Firewall](#5-azure-firewall)
6. [Azure Load Balancer](#6-azure-load-balancer)
7. [Application Gateway](#7-application-gateway)
8. [Azure Front Door](#8-azure-front-door)
9. [Azure Traffic Manager](#9-azure-traffic-manager)
10. [Load Balancing — Full Comparison](#10-load-balancing--full-comparison)
11. [VPN Gateway](#11-vpn-gateway)
12. [Azure ExpressRoute](#12-azure-expressroute)
13. [VPN vs ExpressRoute — Full Comparison](#13-vpn-vs-expressroute--full-comparison)
14. [Azure Virtual WAN](#14-azure-virtual-wan)
15. [Private Endpoint & Azure Private Link](#15-private-endpoint--azure-private-link)
16. [Service Endpoints](#16-service-endpoints)
17. [Private Endpoint vs Service Endpoint](#17-private-endpoint-vs-service-endpoint)
18. [Azure Bastion](#18-azure-bastion)
19. [Azure DDoS Protection](#19-azure-ddos-protection)
20. [Azure DNS](#20-azure-dns)
21. [NAT Gateway](#21-nat-gateway)
22. [Azure CDN](#22-azure-cdn)
23. [Web Application Firewall (WAF)](#23-web-application-firewall-waf)
24. [Network Virtual Appliances (NVAs)](#24-network-virtual-appliances-nvas)
25. [Azure Route Server](#25-azure-route-server)
26. [Azure Network Watcher](#26-azure-network-watcher)
27. [Hub-and-Spoke Topology](#27-hub-and-spoke-topology)
28. [HA & DR — Networking Patterns](#28-ha--dr--networking-patterns)
29. [Exam Tips — Networking](#29-exam-tips--networking)

---

## 1. Virtual Networks (VNets)

### What It Is
Fundamental networking building block in Azure. Provides isolation, segmentation, and connectivity for Azure resources.

### Core Concepts

| Concept | Description | Notes |
|---------|-------------|-------|
| **Address Space** | One or more CIDR blocks (e.g., 10.0.0.0/16) | Must not overlap with peered VNets |
| **Subnets** | Subdivisions of the VNet address space | Resources deployed into subnets |
| **System Routes** | Auto-created routes for intra-VNet, internet, peering | Cannot be deleted; only override with UDR |
| **DNS** | Azure-provided DNS or custom DNS servers | Custom DNS = set at VNet level |
| **Region-scoped** | VNet exists within one region | Use peering or VPN for cross-region |
| **Subscription-scoped** | VNet belongs to one subscription | Use peering for cross-subscription |

### Reserved Addresses per Subnet

| Address | Purpose |
|---------|---------|
| x.x.x.0 | Network address |
| x.x.x.1 | Default gateway |
| x.x.x.2 | Azure DNS mapping |
| x.x.x.3 | Azure DNS mapping |
| x.x.x.255 | Broadcast |

> Azure reserves **5 IP addresses** per subnet — account for this in IP planning
> Minimum subnet size: **/29** (8 IPs − 5 reserved = 3 usable)

### Special Subnets (Reserved Names)

| Subnet Name | Purpose | Min Size |
|------------|---------|---------|
| **GatewaySubnet** | VPN Gateway or ExpressRoute Gateway | /27 recommended (/29 minimum) |
| **AzureFirewallSubnet** | Azure Firewall | /26 mandatory |
| **AzureFirewallManagementSubnet** | Azure Firewall forced tunneling | /26 mandatory |
| **AzureBastionSubnet** | Azure Bastion | /26 mandatory (/27 minimum for Basic) |
| **RouteServerSubnet** | Azure Route Server | /27 minimum |

> **Exam tip:** Special subnet names are exact — typos mean the feature won't deploy

---

## 2. VNet Peering

### What It Is
Direct, low-latency connectivity between two VNets using Microsoft backbone. No gateways required.

### Types

| Type | Scope | Latency | Use Case |
|------|-------|---------|---------|
| **VNet Peering** | Same region | Lowest | Intra-region VNet connectivity |
| **Global VNet Peering** | Cross-region | Low (backbone) | Cross-region without gateway |

### Key Properties

| Property | Behavior |
|----------|---------|
| **Non-transitive** | A↔B + B↔C does NOT mean A↔C — must peer explicitly or route through hub |
| **Bi-directional setup** | Peering must be created from both VNets (A→B and B→A) |
| **No downtime** | Creating peering doesn't interrupt existing traffic |
| **Address space must not overlap** | Peered VNets cannot have overlapping CIDRs |
| **Cross-subscription** | ✅ Supported (requires permissions in both subscriptions) |
| **Cross-tenant** | ✅ Supported (requires B2B guest or permissions in both tenants) |
| **Gateway transit** | Hub VNet can share its gateway with spoke VNets |
| **Bandwidth** | No artificial bandwidth limits; subject to VM NIC limits |
| **Cost** | Per GB transferred (both ingress and egress charged) |

### Peering Settings

| Setting | Description | When to Enable |
|---------|-------------|---------------|
| **Allow Gateway Transit** | Hub VNet allows spokes to use its VPN/ExpressRoute gateway | Enable on hub side |
| **Use Remote Gateways** | Spoke uses hub's gateway for on-prem connectivity | Enable on spoke side |
| **Allow Forwarded Traffic** | Allow traffic that didn't originate in the peered VNet | Required for hub-spoke routing through NVA/Firewall |
| **Allow Virtual Network Access** | Resources in peered VNet can communicate | Enabled by default (disable to restrict) |

> **Exam tip:** "A↔B + B↔C → A can reach C" requires: hub firewall/NVA + UDRs OR re-peering A↔C directly

---

## 3. Network Security Groups (NSGs)

### What It Is
Stateful Layer-4 packet filter applied to subnets or individual NICs.

### Rule Components

| Field | Description | Example |
|-------|-------------|---------|
| **Priority** | 100–4096; lower = higher priority | 100 |
| **Source** | IP, CIDR, service tag, ASG | 10.0.0.0/24 |
| **Destination** | IP, CIDR, service tag, ASG | 0.0.0.0/0 |
| **Protocol** | TCP, UDP, ICMP, Any | TCP |
| **Port range** | Single, range, or * | 443 or 8080-8090 |
| **Action** | Allow or Deny | Allow |

### Default Rules (Cannot be deleted, lowest priority)

| Rule Name | Priority | Direction | Effect |
|-----------|----------|-----------|--------|
| AllowVNetInBound | 65000 | Inbound | Allow all intra-VNet |
| AllowAzureLoadBalancerInBound | 65001 | Inbound | Allow LB health probes |
| DenyAllInBound | 65500 | Inbound | **Block all other inbound** |
| AllowVNetOutBound | 65000 | Outbound | Allow all intra-VNet |
| AllowInternetOutBound | 65001 | Outbound | Allow outbound internet |
| DenyAllOutBound | 65500 | Outbound | Block all other outbound |

### NSG Placement

| Where Applied | Traffic Controlled |
|--------------|-------------------|
| **Subnet** | All traffic entering/leaving the subnet |
| **NIC (VM)** | Traffic specific to that VM's network interface |
| Best practice | Apply at subnet; NIC-level for extra granularity |

> If NSG on both subnet AND NIC: **both** must allow traffic (AND logic)
> **Stateful** = if you allow inbound on port 443, return traffic is automatically allowed

### Application Security Groups (ASGs)

| Feature | Description |
|---------|-------------|
| **Purpose** | Group VMs logically (e.g., "WebTier", "DbTier") without specifying IPs |
| **Use in NSG rules** | Use ASG as source/destination instead of IP ranges |
| **Benefit** | Rules don't need updating when VM IPs change |
| **Limitation** | ASG and NIC must be in the same VNet |

---

## 4. User-Defined Routes (UDRs)

### What It Is
Custom routing rules that override Azure's default system routes. Applied via **Route Tables** associated with subnets.

### Next Hop Types

| Next Hop | Description | Use Case |
|----------|-------------|---------|
| **Virtual appliance** | Specify IP of NVA/Firewall as next hop | Force traffic through Azure Firewall or NVA |
| **Virtual network gateway** | Direct traffic to VPN/ExpressRoute gateway | Custom on-prem routing |
| **Virtual network** | Keep within VNet | Override incorrect routes |
| **Internet** | Route out to internet directly | Bypass NVA for certain prefixes |
| **None** | Drop the traffic (black hole) | Block specific destinations |

### Common UDR Scenarios

| Scenario | UDR Configuration |
|----------|-----------------|
| **Force all internet traffic through Azure Firewall** | 0.0.0.0/0 → Virtual Appliance (Firewall private IP) |
| **Force spoke-to-spoke traffic through hub firewall** | spoke prefix → Virtual Appliance (hub firewall IP) |
| **Forced tunneling to on-prem** | 0.0.0.0/0 → Virtual Network Gateway |
| **Bypass NVA for Azure Backup** | Storage service tags → Internet |
| **Route specific workloads through different path** | Specific CIDR → specific next hop |

> **Route Table** is a separate resource — must be **associated** with one or more subnets
> **BGP routes** from VPN/ExpressRoute can be overridden by UDR (UDR has higher priority)

---

## 5. Azure Firewall

### What It Is
Managed, cloud-native, stateful firewall as a service. Deployed into a dedicated subnet in a hub VNet.

### Tiers Comparison

| Feature | **Basic** | **Standard** | **Premium** |
|---------|----------|------------|-----------|
| **Stateful L3/L4 filtering** | ✅ | ✅ | ✅ |
| **FQDN Application Rules** | ✅ | ✅ | ✅ |
| **Network Rules** | ✅ | ✅ | ✅ |
| **NAT Rules (DNAT)** | ✅ | ✅ | ✅ |
| **DNS Proxy** | ❌ | ✅ | ✅ |
| **Custom DNS** | ❌ | ✅ | ✅ |
| **Threat Intelligence (alert only)** | ❌ | ✅ Alert mode | ✅ Alert + Deny |
| **Threat Intelligence (alert + deny)** | ❌ | ❌ | ✅ |
| **TLS Inspection** | ❌ | ❌ | ✅ |
| **IDPS (Intrusion Detection + Prevention)** | ❌ | ❌ | ✅ |
| **URL Filtering (beyond FQDN)** | ❌ | ❌ | ✅ |
| **Web Categories** | ❌ | Limited | ✅ Full |
| **Firewall Policy** | ✅ | ✅ | ✅ |
| **Forced tunneling** | ❌ | ✅ | ✅ |
| **Availability Zones** | ❌ | ✅ | ✅ |
| **SKU target** | Small/medium workloads | Production | High-security, enterprise |
| **SLA** | 99.95% | 99.95% | 99.95% |
| **Cost** | Lowest | Medium | Highest |

### Firewall Rule Types

| Rule Type | Evaluated At | Controls | Example |
|-----------|------------|---------|---------|
| **NAT Rules (DNAT)** | First | Inbound traffic translation | Translate public IP:port → private VM IP:port |
| **Network Rules** | Second | L3/L4 (IP, port, protocol) | Allow 10.0.0.0/24 → 10.1.0.0/24 on TCP:443 |
| **Application Rules** | Third | L7 HTTP/S + FQDN | Allow *.microsoft.com on HTTPS |

> Rules evaluated in order: NAT → Network → Application
> **Deny all** is implicit if no rule matches

### Firewall Policy

| Feature | Detail |
|---------|--------|
| **Firewall Policy** | Centralized rule management (replaces classic rules) |
| **Inheritance** | Child policies inherit from parent; cannot override parent rules |
| **RBAC** | Delegate rule management to teams without Firewall access |
| **Reuse** | One policy across multiple firewalls |
| **Rule Collection Groups** | Priority-ordered groups within a policy |

### HA & Scaling

| Feature | Detail |
|---------|--------|
| **Availability Zones** | Standard and Premium support zone deployment (1, 2, 3) |
| **Auto-scale** | Scales automatically based on traffic volume (no manual intervention) |
| **Active-active** | Azure manages redundancy internally; no separate active/passive config needed |
| **SLA** | 99.95% (with zones) |

---

## 6. Azure Load Balancer

### What It Is
Layer-4 (TCP/UDP) load balancer for distributing traffic across backend pool VMs within a region.

### SKUs

| Feature | **Basic** (Legacy) | **Standard** |
|---------|------------------|------------|
| Backend pool size | Up to 300 instances | Up to **1,000 instances** |
| Health probes | HTTP, TCP | HTTP, HTTPS, TCP |
| Availability Zones | ❌ | ✅ Zone-redundant or zonal |
| HA Ports (all ports forwarding) | ❌ | ✅ |
| Outbound rules (SNAT) | Auto (insecure) | Explicit outbound rules required |
| VNet integration | Same VNet only | Any VNet in region |
| Diagnostics | Limited | Azure Monitor metrics + diagnostics |
| SLA | **None** | **99.99%** |
| NSG required for inbound | ❌ | ✅ (secure by default) |
| Cross-zone LB | ❌ | ✅ |
| Global tier | ❌ | ✅ (cross-region LB) |
| Cost | Free | Charged per rule + data |

> ⚠️ **Basic SKU is retired** — only use Standard in any design

### Load Balancer Types

| Type | Direction | Scenarios |
|------|-----------|---------|
| **Public (External)** | Internet → Backend | Web-facing apps, public APIs |
| **Internal (Private)** | VNet → Backend | Internal tiers (app → DB), multi-tier apps |

### Load Balancing Algorithms

| Algorithm | Description | Configurable? |
|-----------|-------------|--------------|
| **5-tuple Hash** (default) | Source IP, dest IP, src port, dest port, protocol | Distribution mode setting |
| **Source IP affinity (2-tuple)** | Same source IP → same backend | Session persistence (source IP) |
| **Source IP + protocol (3-tuple)** | Source IP + protocol → same backend | Session persistence (source IP + proto) |

### HA Ports

| Feature | Description |
|---------|-------------|
| **HA Ports** (Standard only) | Single rule covering ALL ports (0–65535) for ALL protocols |
| **Use case** | NVAs (firewalls, routers) that must accept all traffic |
| **Only for Internal LB** | Not available on public LB |

### Cross-Region Load Balancer (Global Tier)

| Feature | Detail |
|---------|--------|
| **Purpose** | Distribute traffic across multiple regional Standard Load Balancers |
| **Layer** | Layer 4 (TCP/UDP) |
| **Failover** | Automatic region failover |
| **Use case** | Multi-region active-active for non-HTTP workloads (e.g., gaming, financial) |
| **Works with** | Standard LB in each region as backend |

---

## 7. Application Gateway

### What It Is
Regional Layer-7 HTTP/HTTPS load balancer with WAF, SSL termination, URL routing, and cookie-based session affinity.

### SKUs

| Feature | **Standard v2** | **WAF v2** |
|---------|----------------|----------|
| SSL offload / TLS termination | ✅ | ✅ |
| End-to-end SSL | ✅ | ✅ |
| URL path-based routing | ✅ | ✅ |
| Host-based routing (multi-site) | ✅ | ✅ |
| Redirect (HTTP→HTTPS) | ✅ | ✅ |
| Cookie-based session affinity | ✅ | ✅ |
| WebSocket support | ✅ | ✅ |
| Custom error pages | ✅ | ✅ |
| Autoscaling | ✅ | ✅ |
| Availability Zones | ✅ | ✅ |
| **Web Application Firewall (WAF)** | ❌ | ✅ |
| **WAF Policy** | ❌ | ✅ |
| **Bot protection** | ❌ | ✅ |
| SLA | 99.95% | 99.95% |
| Cost | Medium | Higher |

> Always prefer **v2 SKUs** — v1 is retired

### Application Gateway Routing

| Routing Type | Description | Example |
|-------------|-------------|---------|
| **Path-based** | Route by URL path | /api/* → API backend; /images/* → Storage backend |
| **Host-based (multi-site)** | Route by hostname | app1.contoso.com → Backend1; app2.contoso.com → Backend2 |
| **Redirect** | HTTP → HTTPS; one domain → another | http://old.com → https://new.com |
| **Rewrite** | Modify HTTP headers or URL | Strip /v1/ prefix; add security headers |

### Application Gateway Components

| Component | Purpose |
|-----------|---------|
| **Frontend IP** | Public or private IP that receives traffic |
| **Listener** | Combines Frontend IP + Port + Protocol + (optional) Host header |
| **Backend Pool** | VMs, VMSS, App Service, IP addresses, FQDNs |
| **Backend HTTP Settings** | Protocol, port, timeout, cookie affinity for backend communication |
| **Health Probe** | Custom HTTP probe to check backend health |
| **Routing Rule** | Maps Listener → Backend Pool (via HTTP Settings) |

### WAF on Application Gateway

| WAF Mode | Behavior | Use Case |
|----------|---------|---------|
| **Detection** | Log suspicious requests; do NOT block | Initial rollout; baseline tuning |
| **Prevention** | Block + log suspicious requests | Production enforcement |

| WAF Rule Set | Version | What It Covers |
|-------------|---------|---------------|
| **OWASP 3.2** | Latest | SQLi, XSS, Remote Code Execution, LFI, RFI, etc. |
| **OWASP 3.1** | Previous | Same categories, older signatures |
| **Microsoft Bot Manager** | Separate | Block bad bots; allow good bots (Google, Bing) |

---

## 8. Azure Front Door

### What It Is
Global Layer-7 HTTP/HTTPS anycast load balancer with WAF, CDN, URL routing, and SSL offload — operating at Azure's global edge PoPs (130+ locations).

### SKUs

| Feature | **Standard** | **Premium** |
|---------|------------|-----------|
| Custom domains + HTTPS | ✅ | ✅ |
| CDN / caching | ✅ | ✅ |
| URL path-based routing | ✅ | ✅ |
| SSL offload | ✅ | ✅ |
| Compression | ✅ | ✅ |
| **WAF** | Basic OWASP | Full OWASP + bot + custom rules |
| **Bot protection** | Limited | ✅ Full Microsoft Bot Manager |
| **DDoS protection** | Network layer only | ✅ Enhanced |
| **Private Link to origins** | ❌ | ✅ |
| **Security reports** | ❌ | ✅ |
| **Sensitive data masking** | ❌ | ✅ |
| **Azure Web Application Firewall** | Limited | Full |
| Health probes | ✅ | ✅ |
| Traffic acceleration (Anycast + split TCP) | ✅ | ✅ |
| SLA | **99.99%** | **99.99%** |
| Cost | Medium | Higher |

> Classic Front Door tier (v1) is retired — use Standard or Premium

### Front Door Routing Algorithms

| Method | Behavior | Use Case |
|--------|---------|---------|
| **Latency** | Route to lowest-latency origin | Performance optimization |
| **Priority** | Primary → failover to secondary | Active-passive DR |
| **Weighted** | Distribute % of traffic across origins | Canary deployments, blue-green |
| **Session affinity** | Same user → same origin | Stateful applications |

### Front Door Origin Groups

| Concept | Description |
|---------|-------------|
| **Origin** | Backend endpoint (App Service, Storage, VM public IP, APIM, etc.) |
| **Origin Group** | Pool of origins for a route |
| **Health probe** | Determines which origins are healthy |
| **Load balancing settings** | Latency sensitivity threshold (ms) + sample count |

### Front Door with Private Link (Premium)

| Feature | Detail |
|---------|--------|
| **Private Link origins** | Backend App Service / Storage / LB remains private (no public endpoint needed) |
| **How** | Front Door Premium routes through Private Link to private origin |
| **Use case** | Public-facing CDN/WAF with fully private backend |
| **Requires** | Private Link service on origin OR direct Private Link support (App Service, Storage) |

---

## 9. Azure Traffic Manager

### What It Is
**DNS-based** global traffic routing service. Does NOT proxy or inspect traffic — it returns DNS answers pointing clients to the best endpoint.

### Routing Methods

| Method | How It Routes | Use Case |
|--------|-------------|---------|
| **Performance** | Client → nearest endpoint (latency table from Azure) | Optimize user experience globally |
| **Priority** | Primary (1) → Secondary (2) → Tertiary (3)... | Active-passive DR |
| **Weighted** | Distribute by weight (1–1000) | Canary deployment, blue-green |
| **Geographic** | Map client's country/region → specific endpoint | Data residency, regional compliance |
| **Multivalue** | Return multiple healthy endpoints | Client-side load balancing (UDP-only endpoints) |
| **Subnet** | Map client IP range → specific endpoint | Testing, segmentation by ISP/corp network |

### Traffic Manager Properties

| Property | Detail |
|----------|--------|
| **Protocol** | DNS only — returns A/CNAME record; NOT a proxy |
| **Endpoint types** | Azure endpoints (App Service, Public IP, Cloud Service), External endpoints, Nested TM profiles |
| **Health probes** | HTTP/HTTPS/TCP checks to determine endpoint health |
| **TTL** | Default 60 seconds; can be 0–2147483647 |
| **Failover speed** | ~60–300 seconds (depends on TTL + probe interval) |
| **SLA** | 99.99% |
| **Cost** | Low (per million DNS queries + per health check) |

### Nested Traffic Manager Profiles

```
Parent TM (Geographic by region)
  ├── North America → Child TM (Performance within NA)
  │     ├── East US endpoint
  │     └── West US endpoint
  └── Europe → Child TM (Priority within Europe)
        ├── West Europe (primary)
        └── North Europe (secondary)
```

> **Use cases:** Combine routing methods — e.g., geographic + performance within each geo

---

## 10. Load Balancing — Full Comparison

### Decision Matrix

| Service | Scope | Layer | Protocol | WAF | SSL Offload | Routing | Proxy? | SLA |
|---------|-------|-------|----------|-----|------------|---------|--------|-----|
| **Azure LB (Standard)** | Regional | L4 | TCP/UDP/Any | ❌ | ❌ | IP+port hash | ❌ | 99.99% |
| **Application Gateway WAF v2** | Regional | L7 | HTTP/HTTPS | ✅ | ✅ | URL path, host, redirect | ✅ | 99.95% |
| **Azure Front Door Premium** | Global | L7 | HTTP/HTTPS | ✅ | ✅ | Latency/priority/weighted | ✅ | 99.99% |
| **Traffic Manager** | Global | DNS | Any | ❌ | ❌ | DNS-based (geo/perf/priority) | ❌ (DNS only) | 99.99% |
| **Cross-Region LB** | Global | L4 | TCP/UDP | ❌ | ❌ | Regional failover | ❌ | 99.99% |

### When to Use Which

| Requirement | Best Choice |
|-------------|------------|
| Web app with WAF, regional | **Application Gateway WAF v2** |
| Global HTTP app + WAF + CDN + bot protection | **Azure Front Door Premium** |
| Global non-HTTP (gaming, financial L4) | **Traffic Manager** + regional **Load Balancers** |
| Global HTTP failover, simple | **Traffic Manager** (Performance or Priority routing) |
| L4 internal load balancing (NVA, database tier) | **Azure Load Balancer (Internal)** |
| Multi-region L4 active-active | **Cross-Region Load Balancer** |
| Global HTTP + private backends | **Front Door Premium** with Private Link |

### Combining Services (Common Architectures)

```
Global Users
    │
    ▼
Azure Front Door (global HTTP, WAF, CDN)
    │
    ▼
Application Gateway (regional WAF, URL routing per region)
    │
    ▼
Azure Load Balancer (internal, VM pool distribution)
    │
    ▼
VM Backend Pool
```

```
Global Users
    │
    ▼
Traffic Manager (global DNS, geographic routing)
    │
    ▼
Regional Application Gateway (per-region WAF)
    │
    ▼
App Service / AKS
```

---

## 11. VPN Gateway

### What It Is
Encrypted cross-premises connectivity over the public internet between Azure VNets and on-premises networks.

### SKUs

| SKU | Max Throughput | S2S Tunnels | P2S Connections | BGP | Active-Active | Zone Redundant | Cost |
|-----|--------------|-------------|----------------|-----|--------------|---------------|------|
| **Basic** | 100 Mbps | 10 | 128 | ❌ | ❌ | ❌ | Lowest |
| **VpnGw1** | 650 Mbps | 30 | 250 | ✅ | ✅ | ❌ | Low |
| **VpnGw2** | 1 Gbps | 30 | 500 | ✅ | ✅ | ❌ | Medium |
| **VpnGw3** | 1.25 Gbps | 30 | 1,000 | ✅ | ✅ | ❌ | Medium |
| **VpnGw4** | 5 Gbps | 100 | 5,000 | ✅ | ✅ | ❌ | High |
| **VpnGw5** | 10 Gbps | 100 | 10,000 | ✅ | ✅ | ❌ | Highest |
| **VpnGw1AZ–5AZ** | Same | Same | Same | ✅ | ✅ | ✅ | +Zone premium |

> **Basic SKU:** No IKEv2, no BGP, no active-active, no ExpressRoute coexistence → **Never use for production**
> **Zone Redundant (AZ SKUs):** Deploy in multiple AZs for higher resilience

### VPN Types

| VPN Type | Description | Use Case |
|----------|-------------|---------|
| **Route-Based** | Uses routing table; supports dynamic routing (BGP) | Most deployments; required for active-active |
| **Policy-Based** | Static policy-based routing; IKEv1 only | Legacy; only for Basic SKU + specific on-prem devices |

### Connection Types

| Connection Type | Description | Requirements | Use Case |
|----------------|-------------|-------------|---------|
| **Site-to-Site (S2S)** | On-prem VPN device → Azure VPN Gateway | VPN device with public IP; preshared key or cert | Branch office, datacenter |
| **Point-to-Site (P2S)** | Individual client machines → Azure | VPN client software (Azure VPN Client) | Remote workers |
| **VNet-to-VNet** | Azure VNet ↔ Azure VNet via gateways | Two VPN Gateways | Cross-region where peering not sufficient |
| **ExpressRoute coexistence** | VPN as failover backup for ExpressRoute | Both gateways in same VNet | High-availability WAN |

### Active-Active vs Active-Passive

| Topology | Description | SLA |
|----------|-------------|-----|
| **Active-Passive** (default) | One gateway instance active; standby takes over on failure | 99.9% per tunnel |
| **Active-Active** | Both instances active; BGP required; two public IPs | **99.95%** |

> **Active-Active** is recommended for production — doubles throughput and provides faster failover

### P2S Authentication Methods

| Method | Description | Use Case |
|--------|-------------|---------|
| **Certificate** | Client + server certificates | Corp-managed devices |
| **RADIUS** | Integrate with on-prem RADIUS (NPS) | Existing AAA infrastructure |
| **Entra ID (Azure AD)** | Azure AD authentication + Conditional Access | User-based, MFA-capable |

### VPN High Availability Design

```
On-Premises:                    Azure:
  VPN Device A (active)  ──────▶ Gateway Instance A (active) ──── VNet
  VPN Device B (standby) ──────▶ Gateway Instance B (active) ──┘
                                  (Active-Active mode)

For highest HA: Dual-redundant S2S (two on-prem VPN devices, two Azure gateway IPs)
```

---

## 12. Azure ExpressRoute

### What It Is
Private, dedicated network circuit connecting on-premises networks to Azure via a connectivity provider. Traffic does NOT traverse the public internet.

### Circuit SKUs

| SKU | Region Coverage | Billing Option | Use Case | Cost |
|-----|---------------|---------------|---------|------|
| **Local** | Single Azure metro region (same as peering location) | **Unlimited included** | Local-only connectivity; built-in cost cap | Lowest |
| **Standard** | All Azure regions in same geopolitical boundary (e.g., all North America or all Europe) | Metered or Unlimited | Standard enterprise WAN | Medium |
| **Premium** | **All Azure global regions** | Metered or Unlimited | Global orgs needing cross-geo access | Highest |

> **Local circuit:** Available only when on-prem is near a specific peering location (e.g., Dallas → South Central US only)
> **Premium add-ons:** Global Reach, larger route table, more VNet links

### Bandwidth Options
`50 Mbps → 100 Mbps → 200 Mbps → 500 Mbps → 1 Gbps → 2 Gbps → 5 Gbps → 10 Gbps → 100 Gbps`

### Peering Types

| Peering | Connects To | Private? | Protocol | Use Case |
|---------|------------|---------|---------|---------|
| **Azure Private Peering** | VNet resources (via gateway) | ✅ Private | BGP | Enterprise connectivity to Azure IaaS/PaaS via Private Endpoints |
| **Microsoft Peering** | Microsoft 365, Azure PaaS public endpoints | ❌ (public IPs) | BGP | SaaS + Azure public services (Storage, SQL, etc.) |

> **Azure Private Peering** connects to Virtual Network Gateway; uses private IP space
> **Microsoft Peering** connects to Microsoft's public IP ranges (for O365, Azure public services)

### ExpressRoute Connectivity Models

| Model | Description | Provider Type |
|-------|-------------|-------------|
| **Co-located at cloud exchange** | Cross-connect in a colocation facility (Equinix, etc.) | Exchange provider |
| **Point-to-point Ethernet** | Dedicated line from office to Azure edge | Network service provider |
| **Any-to-any (IPVPN)** | Integrate Azure into existing MPLS WAN | WAN provider |
| **ExpressRoute Direct** | Direct 10/100 Gbps connection to Microsoft backbone | Largest enterprises |

### ExpressRoute Global Reach

| Feature | Description |
|---------|-------------|
| **Purpose** | Connect two on-premises sites through Azure backbone (not via internet) |
| **Requires** | Two ExpressRoute circuits in different peering locations |
| **Use case** | Connect branch office in US → branch office in UK via Azure backbone |
| **License** | Requires **Premium** add-on |

### ExpressRoute High Availability

| Configuration | SLA | Description |
|--------------|-----|-------------|
| Single circuit, single peering location | 99.9% | Basic HA |
| Dual circuits, different peering locations | 99.95% | Geo-redundant |
| Dual circuits + VPN failover | Highest | VPN activates if ExpressRoute fails |
| ExpressRoute + VPN coexistence | Best resilience | Automatic failover to VPN |

---

## 13. VPN vs ExpressRoute — Full Comparison

| Factor | **VPN Gateway** | **ExpressRoute** |
|--------|----------------|-----------------|
| **Connection type** | Encrypted over public internet | Private circuit via provider |
| **Security** | IPsec encryption | Private (not encrypted by default; add MACsec) |
| **Max bandwidth** | 10 Gbps (VpnGw5) | **100 Gbps** (ExpressRoute Direct) |
| **Latency** | Variable (internet dependent) | **Consistent, predictable, low** |
| **Reliability** | Subject to internet outages | SLA-backed circuit (99.95%) |
| **SLA** | 99.9% (active-passive), 99.95% (active-active) | 99.95% |
| **Setup time** | Hours | **Weeks** (provider provisioning) |
| **Cost** | Low–Medium | **High** |
| **Internet dependency** | ✅ Required | ❌ Bypasses internet |
| **Encryption** | ✅ IPsec built-in | ❌ Not encrypted by default (MACsec optional) |
| **Use case** | Branch offices, remote workers, DR connectivity | Enterprise WAN, regulated industries, DC-to-Azure |
| **P2S (remote users)** | ✅ | ❌ |
| **BGP support** | ✅ (not Basic) | ✅ Required |

### When to Choose What

| Scenario | Recommendation |
|----------|--------------|
| Small organization, < 1 Gbps, cost-sensitive | **VPN Gateway** |
| Compliance requires no internet transit | **ExpressRoute** |
| Need > 1 Gbps sustained bandwidth | **ExpressRoute** |
| Predictable low latency for latency-sensitive apps | **ExpressRoute** |
| Disaster recovery / failover secondary path | **VPN** (as ExpressRoute backup) |
| Remote user access | **VPN (P2S)** only |
| Maximum resilience | **ExpressRoute primary + VPN secondary** |

---

## 14. Azure Virtual WAN

### What It Is
Microsoft-managed **hub-and-spoke** networking service for connecting branch offices, remote users, and VNets at scale — with automated routing and global transit.

### Tiers

| Feature | **Basic** | **Standard** |
|---------|----------|------------|
| Site-to-site VPN | ✅ | ✅ |
| Point-to-site VPN | ❌ | ✅ |
| ExpressRoute | ❌ | ✅ |
| VNet connections | ✅ | ✅ |
| VNet-to-VNet transit (spoke-to-spoke) | ❌ | ✅ |
| Azure Firewall in hub | ❌ | ✅ (Secured Virtual Hub) |
| Custom routing | ❌ | ✅ |
| BGP peering with NVA in hub | ❌ | ✅ |
| Multi-hub (global transit) | ❌ | ✅ |
| Cost | Low | Higher |

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Virtual Hub** | Microsoft-managed hub VNet; you cannot deploy resources directly into it |
| **Hub-to-hub routing** | Any-to-any connectivity between hubs (regions) automatically |
| **Secured Virtual Hub** | Virtual Hub + Azure Firewall (managed by Firewall Manager) |
| **Branch connectivity** | S2S VPN from branch offices → VWAN hub |
| **Scale units** | Define throughput for VPN gateway, ExpressRoute GW, P2S GW in hub |

### vWAN vs Manual Hub-and-Spoke

| | vWAN | Manual Hub-and-Spoke |
|-|------|---------------------|
| Hub management | Managed by Microsoft | You manage hub VNet |
| Routing automation | ✅ Automatic | Manual UDRs required |
| Deployment complexity | Lower | Higher |
| NVA in hub | Limited (BGP-only NVA) | Full flexibility |
| ExpressRoute | ✅ (Standard) | ✅ Via ExpressRoute Gateway |
| Best for | Large-scale, many branches, minimal admin | Custom routing needs, NVA requirements |

---

## 15. Private Endpoint & Azure Private Link

### What It Is
**Private Endpoint** = a network interface with a private IP from your VNet assigned to a specific PaaS resource.
**Azure Private Link** = the service/technology that enables this connectivity.

### How It Works

```
Your VNet (10.0.0.0/16)
  └── Private Endpoint (10.0.1.5)
         └── Maps to: storage.blob.core.windows.net (Azure Storage account)

Client in VNet resolves storage.blob.core.windows.net
  → DNS returns 10.0.1.5 (private IP in your VNet)
  → Traffic never leaves VNet / Microsoft backbone
  → Public endpoint on storage account can be disabled
```

### Supported Services (Common ones for AZ-305)

| Service | Private Link Support |
|---------|---------------------|
| Azure Storage (Blob, File, Queue, Table) | ✅ |
| Azure SQL Database | ✅ |
| Azure SQL Managed Instance | ✅ (native VNet deployment) |
| Azure Cosmos DB | ✅ |
| Azure Key Vault | ✅ |
| Azure App Service | ✅ |
| Azure Kubernetes Service (API server) | ✅ |
| Azure Container Registry | ✅ |
| Azure Event Hubs | ✅ |
| Azure Service Bus | ✅ |
| Azure Monitor / Log Analytics | ✅ |
| Azure Cognitive Services / OpenAI | ✅ |

### Private Endpoint Requirements

| Requirement | Detail |
|-------------|--------|
| **DNS** | Must configure Private DNS Zone to resolve FQDN to private IP |
| **Private DNS Zone name** | Service-specific (e.g., `privatelink.blob.core.windows.net`) |
| **NSG on PE subnet** | Supported (can filter traffic to/from PE) |
| **UDR on PE subnet** | Supported (force PE traffic through firewall) |
| **Cross-region** | PE can be in different region from the service |
| **Cross-subscription** | ✅ Supported |
| **Cross-tenant** | ✅ Supported (requires approval workflow) |

### Private DNS Zone Integration

| Zone | Service |
|------|---------|
| `privatelink.blob.core.windows.net` | Azure Blob Storage |
| `privatelink.file.core.windows.net` | Azure Files |
| `privatelink.database.windows.net` | Azure SQL Database |
| `privatelink.documents.azure.com` | Azure Cosmos DB |
| `privatelink.vaultcore.azure.net` | Azure Key Vault |
| `privatelink.azurewebsites.net` | Azure App Service |
| `privatelink.servicebus.windows.net` | Azure Service Bus |

> DNS zones must be **linked to every VNet** that needs to resolve the private IP

---

## 16. Service Endpoints

### What It Is
A VNet route optimization that directs traffic from a subnet to Azure PaaS services via the **Azure backbone** — but the PaaS service **still uses its public IP**.

### How It Works

```
Without Service Endpoint:
  VM (10.0.0.4) → Internet → Storage public IP (52.x.x.x)

With Service Endpoint:
  VM (10.0.0.4) → Azure backbone → Storage public IP (52.x.x.x)
                  [Optimized route; still reaches public IP]
```

### Service Endpoint Features

| Feature | Detail |
|---------|--------|
| **Configuration** | Enable on subnet for specific service (e.g., Microsoft.Storage) |
| **Firewall rule** | PaaS firewall can allow "selected VNet" instead of public IPs |
| **Cost** | Free (no additional charge) |
| **Privacy** | NOT truly private — public endpoint still exists; only routing is optimized |
| **Supported services** | Storage, SQL, Cosmos DB, Key Vault, Service Bus, Event Hubs, App Service, etc. |

---

## 17. Private Endpoint vs Service Endpoint

| | **Private Endpoint** | **Service Endpoint** |
|-|---------------------|---------------------|
| **Traffic destination** | Private IP in your VNet | Public IP of PaaS service |
| **Public endpoint** | Can be **disabled** entirely | Still active (just restricted) |
| **Traffic stays in VNet** | ✅ Never leaves your VNet | ❌ Exits VNet to Azure backbone then public IP |
| **DNS** | Requires Private DNS Zone | No DNS change; uses public FQDN → public IP |
| **Cross-VNet access** | ✅ Any connected VNet | ❌ Only from enabled subnets |
| **On-prem access** | ✅ Via ExpressRoute/VPN | ❌ Not accessible on-prem without NVA |
| **NSG support** | ✅ | Limited (not inbound NSG filtering) |
| **Cost** | Per hour + data processed | Free |
| **Security** | **Higher** | Moderate |
| **Complexity** | Higher (DNS, PE resource) | Lower |
| **Preferred for** | **Production, security-sensitive** | Simple, same-VNet-only access |

> **Exam tip:** The exam almost always prefers **Private Endpoint** over Service Endpoint when security is mentioned. Service Endpoints are acceptable for simple, same-VNet scenarios where cost matters.

---

## 18. Azure Bastion

### What It Is
Managed PaaS service providing browser-based RDP and SSH access to Azure VMs **without exposing public IPs** on the VMs — over TLS (port 443).

### Tiers

| Feature | **Basic** | **Standard** | **Premium** |
|---------|----------|------------|-----------|
| Browser-based RDP/SSH | ✅ | ✅ | ✅ |
| Native client support (mstsc, SSH CLI) | ❌ | ✅ | ✅ |
| File transfer (upload/download) | ❌ | ✅ | ✅ |
| IP-based connection | ❌ | ✅ | ✅ |
| Shareable links | ❌ | ✅ | ✅ |
| VNet peering (access VMs in peered VNets) | ❌ | ✅ | ✅ |
| Session recording | ❌ | ❌ | ✅ |
| Kerberos authentication | ❌ | ❌ | ✅ |
| Private-only Bastion (no public IP) | ❌ | ❌ | ✅ |
| Required subnet size | /27 | /26 | /26 |
| SLA | 99.95% | 99.95% | 99.95% |
| Cost | Lowest | Higher | Highest |

### Deployment Requirements

| Requirement | Detail |
|-------------|--------|
| **Subnet name** | Must be exactly `AzureBastionSubnet` |
| **Subnet size** | /26 or larger (Standard/Premium); /27 minimum for Basic |
| **Public IP** | Required on the Bastion host itself (Standard SKU public IP) |
| **VMs** | No public IP required; no NSG on VMs for 3389/22 |
| **NSG on Bastion subnet** | Allow inbound 443 from internet; allow outbound 3389/22 to VNet; allow Azure platform tags |

---

## 19. Azure DDoS Protection

### Tiers

| | **DDoS Network Protection** | **DDoS IP Protection** | **Default (Free)** |
|-|--------------------------|----------------------|------------------|
| Scope | Per VNet | Per public IP | Azure platform |
| Volumetric attack mitigation | ✅ Auto | ✅ Auto | Basic (infrastructure only) |
| Protocol attacks | ✅ | ✅ | Basic |
| Application layer (L7) | ✅ (with WAF) | ✅ (with WAF) | ❌ |
| Adaptive tuning | ✅ | ✅ | ❌ |
| Attack analytics + metrics | ✅ | ✅ | ❌ |
| DRT (Rapid Response team) | ✅ 24/7 | ✅ 24/7 | ❌ |
| Cost guarantee (Azure credit) | ✅ | ✅ | ❌ |
| Cost | ~$2,944/mo per VNet | Per public IP | Free |

> Default protection is **free** but only protects Azure infrastructure, NOT your workloads
> Network Protection covers all public IPs in the VNet; IP Protection is per-IP (cheaper for few IPs)

### DDoS + WAF: Layered Defense

```
Attack types covered by layer:
  L3/L4 volumetric attacks     → DDoS Network Protection (auto-mitigated)
  L7 HTTP application attacks  → WAF (Application Gateway or Front Door)
  Protocol attacks (SYN flood) → DDoS Network Protection
  Slow/low attacks (SlowLoris) → WAF + App Gateway timeouts
```

---

## 20. Azure DNS

### Public DNS Zones

| Feature | Detail |
|---------|--------|
| **Purpose** | Host public DNS zones; resolve external domain names |
| **SLA** | **100%** (anycast via 4+ name servers globally) |
| **Record types** | A, AAAA, CNAME, MX, NS, PTR, SOA, SRV, TXT, CAA, DS, TLSA |
| **Alias records** | Point to Azure resources (Public IP, Traffic Manager, Front Door, CDN) — auto-updates on IP change |
| **Cost** | Per zone + per million queries (very low) |
| **Integration** | Works with Traffic Manager, Front Door, App Service custom domains |

### Private DNS Zones

| Feature | Detail |
|---------|--------|
| **Purpose** | DNS resolution within VNets; no internet exposure |
| **Auto-registration** | VMs automatically registered when joined to VNet link with auto-registration enabled |
| **VNet Links** | Zone must be linked to each VNet that needs to resolve it |
| **Forwarding** | Use Azure DNS Private Resolver for conditional forwarding |
| **Private Endpoint integration** | Private DNS Zones used to map PaaS FQDN → private IP |
| **Split-horizon DNS** | Same FQDN resolves differently from VNet vs internet |
| **Cost** | Per zone + per million queries + per auto-registered record |

### Azure DNS Private Resolver

| Feature | Detail |
|---------|--------|
| **Purpose** | Resolve on-prem DNS from Azure AND resolve Azure private DNS from on-prem (via ExpressRoute/VPN) |
| **Components** | Inbound endpoint (on-prem → Azure queries) + Outbound endpoint (Azure → on-prem queries) |
| **Replaces** | Custom DNS VMs (IaaS DNS forwarders) — previous method |
| **Use case** | Hybrid DNS: on-prem clients resolve `privatelink.blob.core.windows.net` → private IP over ExpressRoute |
| **SLA** | 99.9% |

---

## 21. NAT Gateway

### What It Is
Managed outbound-only SNAT (Source Network Address Translation) service for VNet subnets. Provides consistent, scalable outbound internet connectivity.

### Features

| Feature | Detail |
|---------|--------|
| **Direction** | Outbound only — no inbound traffic |
| **SNAT ports** | Up to 64,512 ports per public IP; up to 16 public IPs per NAT GW |
| **Scale** | Up to **50 Gbps** throughput per NAT Gateway |
| **Association** | Assigned to a subnet; all VMs in subnet use NAT GW for outbound |
| **Public IPs** | Attach static public IPs or prefix (predictable outbound IP) |
| **SLA** | 99.9% |
| **Zones** | Zone-redundant or zonal |
| **Cost** | Per hour + per GB processed |

### NAT Gateway vs Load Balancer Outbound Rules

| | NAT Gateway | LB Outbound Rules |
|-|------------|------------------|
| Purpose | Subnet-level SNAT | Backend pool SNAT |
| SNAT port exhaustion risk | Lower (more ports per IP) | Higher (shared per LB) |
| Configuration | Simple subnet association | Requires outbound rule configuration |
| Inbound traffic | ❌ | ✅ |
| Coexistence | Can work with internal LB | NAT GW takes precedence for outbound when present |
| Recommended | ✅ Preferred | Legacy pattern |

---

## 22. Azure CDN

### What It Is
Distributes static content to users from **edge PoPs** (Points of Presence) geographically close to users — reduces latency and origin server load.

### CDN Profiles / Providers

| Provider | Best For | Key Features | SLA |
|----------|---------|-------------|-----|
| **Microsoft (Standard)** | General purpose, integrated Azure | Rules engine, compression, custom domains | 99.9% |
| **Akamai (Standard)** | Media, large-scale delivery | Large PoP network | 99.9% |
| **Verizon (Standard)** | General, simple rules | — | 99.9% |
| **Verizon (Premium)** | Advanced rules, analytics | Advanced rules engine, real-time stats | 99.9% |
| **Azure Front Door** | HTTP/HTTPS + CDN + WAF + LB | Combined CDN + global LB + WAF | 99.99% |

> **Azure Front Door Standard/Premium** now unifies CDN + WAF + global LB — for new architectures, prefer Front Door over separate CDN

### CDN Caching Behavior

| Setting | Behavior |
|---------|---------|
| **Cache-Control: max-age** | CDN respects origin cache headers |
| **CDN caching rules** | Override origin headers with custom TTL |
| **Query string caching** | Bypass / cache per query string / ignore |
| **Purge** | Force-clear cached content |
| **Preload** | Pre-populate CDN cache for predictable content |

---

## 23. Web Application Firewall (WAF)

### WAF Integration Points

| Platform | WAF Type | Scope | Tier Required |
|----------|---------|-------|--------------|
| **Application Gateway** | Regional WAF | Single region | WAF v2 SKU |
| **Azure Front Door** | Global WAF | All edge PoPs globally | Standard (basic) or Premium (full) |
| **Azure CDN (Verizon Premium)** | CDN WAF | Edge | Verizon Premium |

### WAF Rule Sets

| Rule Set | What It Covers |
|----------|---------------|
| **OWASP Core Rule Set (CRS) 3.2** | SQL injection, XSS, RCE, Path Traversal, Scanner detection |
| **OWASP CRS 3.1** | Previous version (still valid) |
| **Microsoft Default Rule Set (DRS)** | Front Door default — similar to OWASP with Microsoft tuning |
| **Bot Manager** | Known good bots (Google, Bing) vs bad bots (scrapers, scanners) |

### WAF Policy Components

| Component | Description |
|-----------|-------------|
| **Managed rules** | Pre-built OWASP/DRS rulesets (enable/disable per rule) |
| **Custom rules** | Priority-ordered rules (IP match, geo match, string match, rate limit) |
| **Exclusions** | Exclude specific request fields from rule evaluation (reduce false positives) |
| **Rate limiting (Front Door)** | Block/count requests per IP per time window |
| **Policy association** | One WAF Policy can be attached to multiple App GW listeners or Front Door domains |

---

## 24. Network Virtual Appliances (NVAs)

### What It Is
Third-party virtual network appliances (firewalls, routers, WAN optimizers) from Azure Marketplace, deployed as VMs in a hub VNet.

### Common NVA Vendors

| Vendor | Product | Use Case |
|--------|---------|---------|
| Palo Alto | VM-Series | Next-gen firewall |
| Fortinet | FortiGate | Next-gen firewall + SD-WAN |
| Check Point | CloudGuard | Firewall + threat prevention |
| Cisco | CSR 1000V | Router, SD-WAN |
| F5 | BIG-IP | Application delivery (LB + WAF + security) |
| Barracuda | CloudGen Firewall | Firewall + VPN |

### NVA HA Patterns

| Pattern | Architecture | Notes |
|---------|-------------|-------|
| **Active-Passive with LB** | Internal LB health probes detect failure; reroute to passive | Manual failover |
| **Active-Active with LB** | Internal LB distributes across both NVAs | Azure LB HA Ports required |
| **Zone-redundant** | NVAs in different AZs + zone-redundant LB | Best HA |

### NVA vs Azure Firewall

| | NVA (3rd party) | Azure Firewall |
|-|----------------|---------------|
| Management | Customer managed | Fully managed PaaS |
| Feature depth | Deepest (vendor-specific) | Standard L4/L7 + IDPS (Premium) |
| Licensing | Bring your own (Marketplace) | Azure pay-per-hour |
| HA | Manual setup | Built-in auto-scale and zone redundancy |
| Compliance certifications | Vendor-specific | Microsoft |
| Integration with Azure | Manual UDRs, Firewall Manager partial | Native Firewall Manager, Policy |
| Best for | Specific features unavailable in Azure FW | Simplicity + manageability |

---

## 25. Azure Route Server

### What It Is
Azure-managed BGP endpoint in a VNet that automatically propagates NVA (third-party appliance) routes to all VNets and connected on-prem networks — eliminates manual UDR management.

### Key Features

| Feature | Detail |
|---------|--------|
| **BGP peering** | NVA or SD-WAN appliance establishes BGP session with Route Server |
| **Route propagation** | Routes learned from NVA automatically programmed in VNet |
| **Branch-to-branch** | Enable NVA to route traffic between on-prem branches in Azure hub |
| **Subnet** | Required: `RouteServerSubnet` (/27 minimum) |
| **Zone redundancy** | Deployed across AZs (built-in) |
| **SLA** | 99.99% |
| **Cost** | Per peering hour |

> Use Route Server when running **SD-WAN or third-party NVA** in hub — eliminates need for manual UDR updates on every spoke when routes change

---

## 26. Azure Network Watcher

### What It Is
Collection of network monitoring and diagnostic tools for Azure networks.

### Tools

| Tool | Purpose | Use Case |
|------|---------|---------|
| **Topology** | Visual map of network resources | Understand VNet layout |
| **Connection Monitor** | End-to-end connectivity monitoring | Ongoing latency + loss tracking |
| **IP Flow Verify** | Test if a packet would be allowed or denied by NSG | Debug NSG rules |
| **NSG Diagnostic** | Show which NSG rule allows/denies traffic for a flow | Detailed NSG troubleshooting |
| **Next Hop** | Show which next hop a packet takes from a VM NIC | Debug routing issues |
| **Effective Routes** | Show all active routes for a VM NIC | Check UDR + system route conflicts |
| **Effective Security Rules** | Show all effective NSG rules for a VM NIC (merged subnet + NIC NSG) | Debug combined NSG rules |
| **Packet Capture** | Capture packets on a VM | Deep troubleshooting |
| **VPN Diagnostics** | Diagnose VPN Gateway connections | VPN troubleshooting |
| **Flow Logs** | Log all network flows (allow + deny) for NSGs or VNets | Security auditing, compliance, cost analysis |
| **Traffic Analytics** | Analyze flow logs; visualize hot spots | Security + operational insights |

> **NSG Flow Logs:** Stored in Storage Account; processed by Traffic Analytics (Log Analytics)
> **VNet Flow Logs** (newer): VNet-level flow capture (not just NSG-level)

---

## 27. Hub-and-Spoke Topology

### Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    HUB VNet                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐ │
│  │ VPN/ER   │  │  Azure   │  │  Azure   │  │Bastion │ │
│  │ Gateway  │  │ Firewall │  │   DNS    │  │        │ │
│  └──────────┘  └──────────┘  └──────────┘  └────────┘ │
└────────────────────────┬────────────────────────────────┘
                         │ VNet Peering (both directions)
              ┌──────────┼──────────┐
              │          │          │
         ┌────┴────┐ ┌───┴───┐ ┌───┴────┐
         │ Spoke 1 │ │Spoke 2│ │Spoke 3 │
         │  (App)  │ │ (DB)  │ │ (Dev)  │
         └─────────┘ └───────┘ └────────┘
```

### Hub Services

| Service in Hub | Purpose |
|---------------|---------|
| **VPN/ExpressRoute Gateway** | On-prem connectivity; shared via gateway transit |
| **Azure Firewall** | Inspect + filter spoke-to-spoke and spoke-to-internet traffic |
| **Azure Bastion** | Centralized RDP/SSH access to all spoke VMs |
| **DNS Resolver / DNS Server** | Centralized DNS for all spokes |
| **Shared services** | AD Domain Controllers, monitoring VMs, jump boxes |

### Routing in Hub-and-Spoke

```
Spoke VMs use UDR:
  Default route (0.0.0.0/0) → Azure Firewall private IP (hub)
  Other spoke prefixes → Azure Firewall private IP (hub)

Hub Firewall:
  Allows/denies traffic between spokes
  Logs all inter-spoke and internet traffic
```

### Key Design Rules

| Rule | Why |
|------|-----|
| VNet Peering is NOT transitive | Must route spoke-to-spoke through hub firewall |
| Enable "Allow Gateway Transit" on hub | Spokes can use hub's VPN/ER gateway |
| Enable "Use Remote Gateways" on spokes | Spokes route on-prem traffic through hub gateway |
| Associate UDR with ALL spoke subnets | Force traffic through hub firewall |
| AzureFirewallSubnet must be /26 | Azure requirement |

---

## 28. HA & DR — Networking Patterns

### Availability Zones — Network Services

| Service | Zone-Redundant Option | Notes |
|---------|----------------------|-------|
| **Azure Load Balancer (Standard)** | ✅ Zone-redundant (frontend + backend) | Single frontend spans all zones |
| **Application Gateway v2** | ✅ Deploy instances across zones | Configure min instances in each zone |
| **Azure Front Door** | ✅ Built-in (anycast global) | Inherently resilient |
| **VPN Gateway (AZ SKUs)** | ✅ GatewaySubnet + AZ SKU | Requires zone-redundant public IP |
| **ExpressRoute Gateway (AZ SKUs)** | ✅ ErGw1AZ, 2AZ, 3AZ | Higher availability |
| **Azure Firewall Standard/Premium** | ✅ Configure in specific zones | Auto-scales across zones |
| **Azure Bastion Standard** | ✅ Zone-redundant | Built-in |
| **NAT Gateway** | ✅ Zone-redundant or zonal | |
| **Public IP (Standard)** | ✅ Zone-redundant (assign to any zone) | Standard SKU required |

> **Basic Public IP** = no zone support → **never use for HA workloads**

### Multi-Region DR Patterns for Networking

| Pattern | Description | RTO | Cost |
|---------|-------------|-----|------|
| **Active-Active** | Traffic Manager + resources running in both regions | Near-zero | Highest |
| **Active-Passive (Hot)** | Standby resources in DR region; failover via Traffic Manager | Minutes | High |
| **Active-Passive (Warm/Cold)** | DR region provisioned but scaled down or stopped | Hours | Medium |
| **Pilot Light** | Core resources (DB, config) running; compute scaled to zero | 30–60 min | Lower |

### Network Failover — Service-Specific

| Service | HA Option | DR Option |
|---------|-----------|-----------|
| VPN Gateway | Active-active config (99.95%) | Backup via second circuit + route failover |
| ExpressRoute | Dual circuits (different providers/locations) | VPN Gateway as fallback |
| Application Gateway | AZ deployment | Multi-region via Traffic Manager |
| Front Door | Built-in global resilience | Automatic failover to healthy origins |
| Traffic Manager | Multiple endpoints + health probes + TTL | Auto failover within TTL window |
| Azure Firewall | Auto-scaling + zone deployment | Deploy in DR region hub |
| Private DNS | Zone-redundant (Microsoft-managed) | Globally replicated |

---

## 29. Exam Tips — Networking

### VNet & Routing Tips

| # | Tip | Trap |
|---|-----|------|
| 1 | **VNet Peering is NOT transitive** — A↔B + B↔C does NOT give A↔C | Assuming peering is transitive in hub-spoke |
| 2 | Azure reserves **5 IPs per subnet** — factor into IP planning | Sizing a /29 gives only 3 usable IPs |
| 3 | **GatewaySubnet** must be named exactly "GatewaySubnet" (no NSG recommended) | Applying NSG to GatewaySubnet can break VPN/ER |
| 4 | **AzureFirewallSubnet** must be /26 minimum | Using /27 will fail deployment |
| 5 | **AzureBastionSubnet** must be /26 (Standard/Premium) | Using /27 on Standard tier blocks some features |
| 6 | UDRs **override system routes** and BGP routes — UDR has highest priority | BGP routes won't work if UDR overrides them |
| 7 | UDR route table must be **associated** with a subnet — creating it isn't enough | Creating table without associating = no effect |

### Load Balancing Tips

| # | Tip | Trap |
|---|-----|------|
| 8 | **Traffic Manager is DNS-based** — it does NOT inspect or proxy traffic | Expecting TM to do L7 routing or WAF |
| 9 | **Traffic Manager failover time = DNS TTL** (default 60s) — not instant | Assuming instant failover |
| 10 | **Application Gateway = regional** — for global HTTP use Front Door | Using App GW for global routing |
| 11 | **Front Door = global HTTP/S** with WAF and CDN — replaces TM + App GW for HTTP globally | Using TM + App GW when Front Door covers both |
| 12 | **Load Balancer Basic has NO SLA** and is being retired — never use | Designing with Basic LB for production |
| 13 | **HA Ports** rule on Internal LB = all ports, all protocols — only works on Internal LB | Trying to configure HA Ports on public LB |
| 14 | **Front Door Premium** needed for Private Link origins + full WAF | Using Standard Front Door and expecting private backends |

### VPN & ExpressRoute Tips

| # | Tip | Trap |
|---|-----|------|
| 15 | **VPN Basic SKU:** No IKEv2, no BGP, no ExpressRoute coexistence — **avoid for production** | Recommending Basic SKU for any serious workload |
| 16 | **ExpressRoute is NOT encrypted** by default — add IPsec over ER or use MACsec if needed | Assuming private = encrypted |
| 17 | **ExpressRoute Local** = unlimited data transfer BUT only connects to nearby Azure region | Using Local SKU and expecting global region access |
| 18 | **ExpressRoute Global Reach** = on-prem to on-prem via Azure backbone; requires Premium add-on | Not knowing this exists |
| 19 | **VPN Active-Active** requires Route-Based VPN + BGP | Trying to do active-active with policy-based |
| 20 | For **maximum resilience**: ExpressRoute primary + VPN Gateway secondary (coexistence) | Using either alone for highly critical workloads |

### Firewall, Security & DNS Tips

| # | Tip | Trap |
|---|-----|------|
| 21 | **Azure Firewall Premium** required for IDPS and TLS inspection | Using Standard when compliance requires IDPS |
| 22 | **NSG on subnet AND NIC** = both must allow traffic (AND logic) | Allowing on subnet but blocking on NIC = blocked |
| 23 | **Private Endpoint > Service Endpoint** for security-sensitive scenarios | Choosing Service Endpoint when question asks for "private connectivity" |
| 24 | **Service Endpoints don't block public access** to PaaS — only restrict from other networks | Thinking SE makes service private |
| 25 | **Private Endpoint DNS**: must configure Private DNS Zone AND link it to each VNet | Creating PE but forgetting DNS zone = public IP still resolves |
| 26 | **DDoS Default is free** but only protects Azure infrastructure, not your workload | Thinking Default DDoS protects your app |
| 27 | **NSG does NOT support FQDN rules** — use Azure Firewall for FQDN filtering | Trying to write NSG rule with domain name |
| 28 | **Azure Firewall rule evaluation order:** NAT → Network → Application | Getting confused by rule type priorities |

### Cross-Domain Design Tips

| # | Scenario | Answer |
|---|----------|--------|
| 29 | "Inspect all traffic between spoke VNets" | Hub-spoke + Azure Firewall + UDRs on all spoke subnets |
| 30 | "On-prem users access private Azure PaaS services" | ExpressRoute + Private Endpoint + Private DNS Resolver |
| 31 | "Block all RDP/SSH from internet; still allow admin access" | **Azure Bastion** (no public IP on VMs) |
| 32 | "App service must not be publicly accessible" | **Private Endpoint** on App Service + disable public endpoint |
| 33 | "Regulate which regions Azure resources can be deployed in" | **Azure Policy** (Allowed Locations) — not a networking service |
| 34 | "Global HTTPS app with WAF, CDN, private backends" | **Azure Front Door Premium** (WAF + CDN + Private Link to origins) |
| 35 | "Branch offices need reliable, private WAN to Azure at 10 Gbps" | **ExpressRoute Standard** with 10 Gbps circuit |
| 36 | "Remote workers need access to Azure VNet resources" | **VPN Gateway P2S** with Entra ID auth + Conditional Access |
| 37 | "NVA routing table changes propagate automatically to all spokes" | **Azure Route Server** (BGP propagation from NVA) |
| 38 | "All outbound internet traffic from VMs must use fixed IPs" | **NAT Gateway** with static public IP prefix |
| 39 | "Multi-region active-passive HTTP app; failover in minutes" | **Traffic Manager (Priority routing)** + App Service in both regions |
| 40 | "VM can't reach Azure Storage — diagnose the issue" | **Network Watcher: IP Flow Verify + Effective Routes + Effective Security Rules** |

---

## Quick Reference Summary

```
LOAD BALANCING CHOICE:
  Global HTTP + WAF + CDN         → Azure Front Door (Standard/Premium)
  Global non-HTTP / DNS routing   → Traffic Manager
  Regional HTTP + WAF             → Application Gateway WAF v2
  Regional L4 (TCP/UDP)           → Azure Load Balancer Standard
  Multi-region L4 active-active   → Cross-Region Load Balancer

PRIVATE CONNECTIVITY:
  Private IP for PaaS in VNet     → Private Endpoint (secure)
  Optimized route to PaaS (not private) → Service Endpoint (basic)
  On-prem users to Azure PaaS     → ExpressRoute + Private Endpoint + DNS Resolver

HYBRID CONNECTIVITY:
  < 1 Gbps, cost-sensitive        → VPN Gateway
  > 1 Gbps, private, SLA-critical → ExpressRoute
  Maximum resilience              → ExpressRoute + VPN (coexistence)
  Many branch offices at scale    → Azure Virtual WAN Standard

HUB-AND-SPOKE RULES:
  Traffic between spokes          → Must go through hub firewall (UDRs required)
  Spoke-to-on-prem                → Use hub gateway (Allow Gateway Transit + Use Remote Gateways)
  Peering is NOT transitive       → A↔B + B↔C ≠ A↔C

SECURITY LAYERS:
  L3/L4 rules (IP/port)          → NSG
  L4 stateful + FQDN + DNAT      → Azure Firewall (Standard)
  + IDPS + TLS inspect            → Azure Firewall (Premium)
  OWASP L7 HTTP protection        → WAF (App Gateway or Front Door)
  DDoS volumetric                 → DDoS Network/IP Protection
  VM admin without public IP      → Azure Bastion
  Outbound fixed IP               → NAT Gateway

SPECIAL SUBNET NAMES (exact):
  GatewaySubnet                   → VPN / ER Gateway (/27+)
  AzureFirewallSubnet             → Azure Firewall (/26 mandatory)
  AzureBastionSubnet              → Azure Bastion (/26 Standard, /27 Basic)
  RouteServerSubnet               → Azure Route Server (/27+)
```

---

*Last updated: May 20, 2026 | Good luck on May 21! 🎯*
