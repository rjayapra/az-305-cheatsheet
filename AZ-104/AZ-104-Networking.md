# AZ-104 — Implement and Manage Virtual Networking
## Domain 4 Deep-Dive Study Guide (15–20% of Exam)

---

## Table of Contents
1. [Virtual Networks (VNets)](#1-virtual-networks-vnets)
2. [Subnets & IP Addressing](#2-subnets--ip-addressing)
3. [Public & Private IP Addresses](#3-public--private-ip-addresses)
4. [Network Security Groups (NSGs)](#4-network-security-groups-nsgs)
5. [Application Security Groups (ASGs)](#5-application-security-groups-asgs)
6. [Azure DNS — Public Zones](#6-azure-dns--public-zones)
7. [Azure DNS — Private Zones](#7-azure-dns--private-zones)
8. [VNet Peering](#8-vnet-peering)
9. [VPN Gateway](#9-vpn-gateway)
10. [ExpressRoute — Overview](#10-expressroute--overview)
11. [Azure Load Balancer](#11-azure-load-balancer)
12. [Application Gateway](#12-application-gateway)
13. [Network Watcher](#13-network-watcher)
14. [Service Endpoints & Private Endpoints](#14-service-endpoints--private-endpoints)
15. [Azure Bastion](#15-azure-bastion)
16. [Exam Tips — Domain 4 Master List](#16-exam-tips--domain-4-master-list)

---

## 1. Virtual Networks (VNets)

### What It Is
Fundamental building block for private networking in Azure. Provides isolation, segmentation, and connectivity for Azure resources.

### Key Properties

| Property | Detail |
|----------|--------|
| **Region-bound** | VNet exists in one Azure region only |
| **Subscription-bound** | VNet belongs to one subscription |
| **Address space** | One or more CIDR blocks (private IP ranges) |
| **Subnets** | Subdivide address space for different workloads |
| **DNS** | Azure-provided (default) or custom DNS servers |
| **Cost** | VNet itself is free; charges for gateways, peering bandwidth, etc. |

### Valid Private Address Ranges (RFC 1918)
| Range | CIDR | Addresses |
|-------|------|-----------|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | 16,777,216 |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | 1,048,576 |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | 65,536 |

### Creating VNets

```bash
# Azure CLI
az network vnet create \
  --resource-group MyRG \
  --name MyVNet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name WebSubnet \
  --subnet-prefix 10.0.1.0/24 \
  --location eastus

# PowerShell
$vnet = New-AzVirtualNetwork -ResourceGroupName MyRG `
  -Name MyVNet `
  -Location eastus `
  -AddressPrefix 10.0.0.0/16

Add-AzVirtualNetworkSubnetConfig -Name WebSubnet `
  -VirtualNetwork $vnet `
  -AddressPrefix 10.0.1.0/24

$vnet | Set-AzVirtualNetwork
```

---

## 2. Subnets & IP Addressing

### Reserved IP Addresses (Per Subnet)
Azure reserves **5 IPs** in every subnet:

| Address | Purpose |
|---------|---------|
| x.x.x.**0** | Network address |
| x.x.x.**1** | Default gateway |
| x.x.x.**2** | Azure DNS mapping |
| x.x.x.**3** | Azure DNS mapping |
| x.x.x.**255** | Broadcast (for /24) |

### Usable IPs per Subnet Size
| CIDR | Total IPs | Usable IPs | Common Use |
|------|-----------|-----------|-----------|
| /24 | 256 | 251 | Standard workload subnet |
| /25 | 128 | 123 | Medium subnet |
| /26 | 64 | 59 | Small subnet (min for Firewall/Bastion) |
| /27 | 32 | 27 | Gateway subnet recommended |
| /28 | 16 | 11 | Minimum practical subnet |
| /29 | 8 | 3 | Smallest usable subnet |

### Special Subnets (Reserved Names)

| Subnet Name | Purpose | Requirements |
|------------|---------|-------------|
| **GatewaySubnet** | VPN / ExpressRoute Gateway | /27 recommended (/29 min) |
| **AzureBastionSubnet** | Azure Bastion | /26 minimum |
| **AzureFirewallSubnet** | Azure Firewall | /26 exactly |

> These names are EXACT — must use these specific names for the services to deploy

### Subnet Delegation
- Assigns a subnet to a specific Azure service (e.g., App Service, SQL MI, Container Instances)
- Only that service can deploy into the delegated subnet
- Prevents other resources from being placed in that subnet

### Adding Subnets

```bash
# Add subnet to existing VNet
az network vnet subnet create \
  --resource-group MyRG \
  --vnet-name MyVNet \
  --name DBSubnet \
  --address-prefix 10.0.2.0/24 \
  --network-security-group MyNSG

# List subnets
az network vnet subnet list --resource-group MyRG --vnet-name MyVNet --output table
```

---

## 3. Public & Private IP Addresses

### Public IP Addresses

| Property | Basic SKU | Standard SKU |
|----------|-----------|--------------|
| **Allocation** | Dynamic or Static | Static only |
| **Availability Zones** | ❌ | ✅ (zone-redundant) |
| **Default security** | Open (no NSG needed) | **Closed** (requires NSG) |
| **SLA** | — | 99.99% |
| **Load Balancer** | Basic LB only | Standard LB only |
| **Routing** | Microsoft network | Microsoft or Internet |

> **Standard SKU is recommended** for production. Basic SKU is being retired.

### Dynamic vs Static Public IP

| | Dynamic | Static |
|-|---------|--------|
| Assigned when | VM started | Created (never changes) |
| Released when | VM deallocated | Manually deleted |
| Use case | Dev/test, temp workloads | DNS records, firewall rules, VPN |
| Billing | Free when attached to running VM | Charged when not associated |

### Private IP Addresses
- Automatically assigned from subnet range (DHCP)
- **Dynamic** (default): May change on deallocation/restart
- **Static**: Fixed IP within subnet range (recommended for servers, DNS, DCs)
- Every NIC has at least one private IP (can have multiple)

### Managing IPs

```bash
# Create public IP
az network public-ip create \
  --resource-group MyRG \
  --name MyPublicIP \
  --sku Standard \
  --allocation-method Static \
  --zone 1 2 3

# Associate with NIC
az network nic ip-config update \
  --resource-group MyRG \
  --nic-name MyNIC \
  --name ipconfig1 \
  --public-ip-address MyPublicIP

# Set static private IP
az network nic ip-config update \
  --resource-group MyRG \
  --nic-name MyNIC \
  --name ipconfig1 \
  --private-ip-address 10.0.1.10
```

---

## 4. Network Security Groups (NSGs)

### What It Is
Stateful Layer-4 firewall that filters traffic based on 5-tuple (source/dest IP, source/dest port, protocol).

### NSG Placement

| Applied To | Filters |
|-----------|---------|
| **Subnet** | All traffic in/out of the subnet |
| **NIC** | Traffic specific to that VM's NIC |
| **Both** | Both evaluated — BOTH must allow traffic |

### Default Rules (Cannot Be Deleted)

| Rule | Priority | Direction | Action | Description |
|------|----------|-----------|--------|-------------|
| AllowVNetInBound | 65000 | Inbound | Allow | All intra-VNet traffic |
| AllowAzureLoadBalancerInBound | 65001 | Inbound | Allow | Health probes from LB |
| DenyAllInBound | 65500 | Inbound | **Deny** | Everything else blocked |
| AllowVNetOutBound | 65000 | Outbound | Allow | All intra-VNet traffic |
| AllowInternetOutBound | 65001 | Outbound | Allow | Outbound to internet |
| DenyAllOutBound | 65500 | Outbound | **Deny** | Everything else blocked |

### NSG Rule Properties

| Property | Description | Values |
|----------|-------------|--------|
| **Priority** | Order of evaluation | 100-4096 (lower = first) |
| **Source** | Traffic origin | IP, CIDR, Service Tag, ASG, Any |
| **Source port** | Origin port | *, number, range |
| **Destination** | Traffic target | IP, CIDR, Service Tag, ASG, Any |
| **Destination port** | Target port | *, number, range (e.g., 80, 443, 8080-8090) |
| **Protocol** | Network protocol | TCP, UDP, ICMP, Any |
| **Action** | Allow or Deny | Allow / Deny |

### Service Tags (Common)

| Tag | Represents |
|-----|-----------|
| **Internet** | All public IP addresses |
| **VirtualNetwork** | VNet + peered VNets + VPN-connected networks |
| **AzureLoadBalancer** | Azure Load Balancer health probes |
| **Storage** | Azure Storage service IPs |
| **Sql** | Azure SQL Database IPs |
| **AzureCloud** | All Azure datacenter IPs |
| **AzureCloud.EastUS** | Azure IPs in specific region |

### Processing Rules
1. Inbound: Subnet NSG evaluated first → NIC NSG evaluated second
2. Outbound: NIC NSG evaluated first → Subnet NSG evaluated second
3. Within each NSG: lowest priority number matched first
4. First matching rule wins (stop processing)
5. If no rule matches → DenyAll default applies

### Creating NSG Rules

```bash
# Create NSG
az network nsg create --resource-group MyRG --name WebNSG

# Allow HTTP inbound
az network nsg rule create \
  --resource-group MyRG \
  --nsg-name WebNSG \
  --name AllowHTTP \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-ranges 80

# Allow HTTPS inbound
az network nsg rule create \
  --resource-group MyRG \
  --nsg-name WebNSG \
  --name AllowHTTPS \
  --priority 110 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-ranges 443

# Deny all other inbound (explicit, for documentation)
az network nsg rule create \
  --resource-group MyRG \
  --nsg-name WebNSG \
  --name DenyAll \
  --priority 4000 \
  --direction Inbound \
  --access Deny \
  --protocol "*" \
  --destination-port-ranges "*"

# Associate NSG with subnet
az network vnet subnet update \
  --resource-group MyRG \
  --vnet-name MyVNet \
  --name WebSubnet \
  --network-security-group WebNSG
```

### Effective Security Rules
- Combined result of all NSGs applied to a NIC
- Check via: Portal → VM → Networking → Effective security rules
- CLI: `az network nic list-effective-nsg --resource-group MyRG --name MyNIC`
- Includes inherited rules from subnet NSG + NIC NSG

---

## 5. Application Security Groups (ASGs)

### What It Is
Logical grouping of VMs by function/role for use in NSG rules (instead of individual IPs).

### How It Works

```
Traditional NSG:
  Source: 10.0.1.4, 10.0.1.5, 10.0.1.6  → Port 443 → 10.0.2.4, 10.0.2.5

With ASGs:
  Source: WebServers-ASG → Port 1433 → DBServers-ASG
```

### Benefits
- Rules automatically apply when new VMs are added to the ASG
- No need to update rules when VM IPs change
- Cleaner, more readable NSG rules
- Enforce micro-segmentation

### Limitations
- ASG and NIC must be in the **same VNet** (or peered VNets)
- Source and destination ASGs must be in the same VNet
- Cannot mix ASGs from different VNets in one rule

### Using ASGs

```bash
# Create ASG
az network asg create --resource-group MyRG --name WebServers
az network asg create --resource-group MyRG --name DBServers

# Associate NIC with ASG
az network nic ip-config update \
  --resource-group MyRG \
  --nic-name WebVM-NIC \
  --name ipconfig1 \
  --application-security-groups WebServers

# NSG rule using ASGs
az network nsg rule create \
  --resource-group MyRG \
  --nsg-name MyNSG \
  --name AllowWebToDB \
  --priority 100 \
  --source-asgs WebServers \
  --destination-asgs DBServers \
  --destination-port-ranges 1433 \
  --protocol Tcp \
  --access Allow
```

---

## 6. Azure DNS — Public Zones

### What It Is
Host your public DNS domain in Azure for name resolution via Azure's global anycast DNS infrastructure.

### DNS Record Types

| Type | Purpose | Example |
|------|---------|---------|
| **A** | IPv4 address | www → 20.50.2.5 |
| **AAAA** | IPv6 address | www → 2001:db8::1 |
| **CNAME** | Alias to another domain name | blog → myblog.azurewebsites.net |
| **MX** | Mail exchange | @ → mail.contoso.com (priority 10) |
| **TXT** | Text (verification, SPF) | @ → "v=spf1 include:..." |
| **NS** | Name server delegation | — |
| **SOA** | Start of Authority | — |
| **SRV** | Service location | _sip._tcp → sipserver:5060 |
| **PTR** | Reverse lookup | — |
| **CAA** | Certificate authority authorization | — |

### Alias Record Sets
- Point directly to Azure resources (Load Balancer, Traffic Manager, CDN, Public IP)
- **Automatically update** when the resource's IP changes
- Supported on: A, AAAA, CNAME record types at zone apex (@)
- Solve the "CNAME at zone apex" problem (CNAME not allowed at @ normally)

### DNS Zone Management

```bash
# Create public DNS zone
az network dns zone create \
  --resource-group MyRG \
  --name contoso.com

# Add A record
az network dns record-set a add-record \
  --resource-group MyRG \
  --zone-name contoso.com \
  --record-set-name www \
  --ipv4-address 20.50.2.5

# Add CNAME record
az network dns record-set cname set-record \
  --resource-group MyRG \
  --zone-name contoso.com \
  --record-set-name blog \
  --cname myblog.azurewebsites.net

# Create alias record (pointing to Public IP)
az network dns record-set a update \
  --resource-group MyRG \
  --zone-name contoso.com \
  --name "@" \
  --target-resource <public-ip-resource-id>
```

### Domain Delegation
1. Create DNS zone in Azure (get NS records)
2. Go to domain registrar (GoDaddy, Namecheap, etc.)
3. Update NS records to point to Azure DNS name servers
4. Azure DNS name servers: `ns1-01.azure-dns.com`, `ns2-01.azure-dns.net`, etc.

---

## 7. Azure DNS — Private Zones

### What It Is
DNS name resolution within VNets without custom DNS servers. Not accessible from the internet.

### Key Features

| Feature | Detail |
|---------|--------|
| **Scope** | VNet-only (private, not internet-accessible) |
| **VNet Link** | Must link private zone to VNet for resolution to work |
| **Auto-registration** | Automatically create DNS records for VMs in linked VNet |
| **Registration VNet** | Only 1 VNet can have auto-registration enabled per zone |
| **Resolution VNet** | Multiple VNets can resolve (read) from the zone |
| **Cross-VNet** | Linked VNets can resolve records across peered VNets |

### Auto-Registration Behavior
- When enabled: VMs in the linked VNet automatically get A records created
- Record format: `<vm-name>.<zone-name>` (e.g., myvm.contoso.private)
- Records deleted when VM is deleted
- Only ONE VNet per zone can have auto-registration

### Private Zone Operations

```bash
# Create private DNS zone
az network private-dns zone create \
  --resource-group MyRG \
  --name contoso.private

# Link to VNet (with auto-registration)
az network private-dns link vnet create \
  --resource-group MyRG \
  --zone-name contoso.private \
  --name MyVNetLink \
  --virtual-network MyVNet \
  --registration-enabled true

# Add A record manually
az network private-dns record-set a add-record \
  --resource-group MyRG \
  --zone-name contoso.private \
  --record-set-name dbserver \
  --ipv4-address 10.0.2.4
```

### Public vs Private DNS Zones

| | Public Zone | Private Zone |
|-|-------------|-------------|
| Accessible from | Internet | VNet only |
| Use case | Public websites | Internal services |
| Auto-registration | ❌ | ✅ |
| VNet link required | ❌ | ✅ |
| Cost | Per zone + per query | Per zone + per query |
| Alias records | ✅ | Limited |

---

## 8. VNet Peering

### What It Is
Low-latency, high-bandwidth connection between two VNets using Microsoft's backbone network.

### Types

| Type | Scope | Latency |
|------|-------|---------|
| **VNet Peering** | Same region | Lowest (backbone) |
| **Global VNet Peering** | Cross-region | Low (still backbone) |

### Key Properties

| Property | Behavior |
|----------|---------|
| **Non-transitive** | A↔B + B↔C ≠ A↔C (must explicitly peer or route through hub) |
| **Bi-directional setup** | Must create peering on BOTH VNets (A→B and B→A) |
| **No address overlap** | Peered VNets cannot share IP ranges |
| **Cross-subscription** | ✅ Supported |
| **Cross-tenant** | ✅ Supported |
| **No downtime** | Creating peering doesn't interrupt traffic |
| **Bandwidth** | No artificial limit (limited by VM NIC speed) |
| **No encryption** | Traffic is private but not encrypted (stays on MS backbone) |

### Peering Settings

| Setting | Description | Set On |
|---------|-------------|--------|
| **Allow virtual network access** | Basic connectivity between VNets | Both sides |
| **Allow forwarded traffic** | Accept traffic that didn't originate from the peered VNet | Receiving side |
| **Allow gateway transit** | Share VPN/ER gateway with peered VNet | Gateway VNet (hub) |
| **Use remote gateways** | Use the peered VNet's gateway | Spoke VNet |

### Gateway Transit Pattern

```
On-Premises
     │
     │ VPN/ExpressRoute
     ▼
Hub VNet (has VPN Gateway)
  ├── "Allow Gateway Transit" = YES
  │
  ├── Peered to Spoke 1 ("Use Remote Gateways" = YES on Spoke 1)
  └── Peered to Spoke 2 ("Use Remote Gateways" = YES on Spoke 2)
```

> Spokes can reach on-prem through Hub's gateway without their own gateway

### Creating Peering

```bash
# Peer VNet-A to VNet-B (must do from BOTH sides)
az network vnet peering create \
  --resource-group MyRG \
  --name AtoB \
  --vnet-name VNetA \
  --remote-vnet VNetB \
  --allow-vnet-access true

az network vnet peering create \
  --resource-group MyRG \
  --name BtoA \
  --vnet-name VNetB \
  --remote-vnet VNetA \
  --allow-vnet-access true

# Verify peering status
az network vnet peering list --resource-group MyRG --vnet-name VNetA --output table
```

### Peering Status

| Status | Meaning |
|--------|---------|
| **Initiated** | Only one side configured (not yet connected) |
| **Connected** | Both sides configured (traffic flows) |
| **Disconnected** | One side deleted their peering link |

---

## 9. VPN Gateway

### What It Is
Encrypted tunnel connecting Azure VNet to on-premises network (S2S), individual clients (P2S), or another VNet (V2V) over the public internet.

### Connection Types

| Type | Connects | Use Case |
|------|----------|----------|
| **Site-to-Site (S2S)** | Azure VNet ↔ On-premises network | Branch office connectivity |
| **Point-to-Site (P2S)** | Individual client ↔ Azure VNet | Remote workers |
| **VNet-to-VNet (V2V)** | Azure VNet ↔ Azure VNet (encrypted) | Cross-region, cross-subscription |

### VPN Gateway SKUs

| SKU | Max S2S | Max P2S | Throughput | Use Case |
|-----|---------|---------|-----------|----------|
| **Basic** | 10 | 128 | 100 Mbps | Dev/test only |
| **VpnGw1** | 30 | 250 | 650 Mbps | Small production |
| **VpnGw2** | 30 | 500 | 1 Gbps | Medium production |
| **VpnGw3** | 30 | 1000 | 1.25 Gbps | Large production |
| **VpnGw4** | 100 | 5000 | 5 Gbps | High-performance |
| **VpnGw5** | 100 | 10000 | 10 Gbps | Maximum |

### VPN Gateway Requirements
| Component | Description |
|-----------|-------------|
| **GatewaySubnet** | Dedicated subnet in the VNet (/27 recommended) |
| **Public IP** | For the gateway's internet-facing endpoint |
| **Local Network Gateway** | Represents on-prem network (IP + address space) |
| **Connection** | Links VPN Gateway to Local Network Gateway |
| **Shared key (PSK)** | Pre-shared key for IPsec authentication |
| **On-prem VPN device** | Compatible device (Cisco, Fortinet, etc.) |

### S2S VPN Setup Steps
1. Create **GatewaySubnet** in your VNet
2. Create **Public IP** for the gateway
3. Create **VPN Gateway** (takes 30-45 minutes)
4. Create **Local Network Gateway** (on-prem public IP + address spaces)
5. Create **Connection** (link VPN GW ↔ Local Network GW with shared key)
6. Configure on-prem VPN device with matching settings

### P2S VPN Authentication Methods
| Method | Description |
|--------|-------------|
| **Certificate** | Azure + client certificates (self-signed or CA) |
| **RADIUS** | Integrate with existing RADIUS server |
| **Entra ID** | Azure AD authentication (OpenVPN only) |

### Active-Active VPN Gateway
- Deploy two gateway instances in two availability zones
- Each has its own public IP
- Provides redundancy within Azure
- On-prem device connects to both IPs

---

## 10. ExpressRoute — Overview

### What It Is
Private, dedicated connection between on-premises and Azure (NOT over public internet).

### Key Facts for AZ-104

| Property | Detail |
|----------|--------|
| **Connection** | Via connectivity provider (ISP/carrier) |
| **Bandwidth** | 50 Mbps to 100 Gbps |
| **Latency** | Predictable, low |
| **Encryption** | Not encrypted by default (private link, not public) |
| **SLA** | 99.95% |
| **Cost** | Higher than VPN (circuit fee + data transfer) |
| **Redundancy** | Built-in dual circuits |

### ExpressRoute vs VPN Gateway

| | VPN Gateway | ExpressRoute |
|-|-------------|-------------|
| Path | Public internet (encrypted) | Private link (provider) |
| Bandwidth | Up to 10 Gbps | Up to 100 Gbps |
| Latency | Variable | Predictable, low |
| Encryption | ✅ IPsec | ❌ (add your own if needed) |
| Cost | Lower | Higher |
| Setup time | Minutes (gateway ~45 min) | Days-weeks (provider circuit) |
| Failover | Active-active option | Pair with VPN for backup |
| Use case | Remote offices, dev/test | Production, data-intensive |

---

## 11. Azure Load Balancer

### What It Is
Layer-4 (TCP/UDP) load balancer for distributing traffic across backend VMs.

### Types

| | Public Load Balancer | Internal Load Balancer |
|-|---------------------|----------------------|
| Frontend | Public IP | Private IP in VNet |
| Traffic source | Internet | Internal VNet/VPN traffic |
| Use case | Internet-facing apps | Multi-tier internal apps |

### SKUs

| Feature | Basic | Standard |
|---------|-------|----------|
| Backend pool | VMs in single Availability Set | Any VMs in single VNet |
| Health probes | TCP, HTTP | TCP, HTTP, **HTTPS** |
| Availability Zones | ❌ | ✅ (zone-redundant) |
| SLA | ❌ No SLA | ✅ 99.99% |
| Security | Open by default | **Closed by default (requires NSG)** |
| Outbound rules | ❌ | ✅ |
| Multiple frontends | ❌ | ✅ |
| HA Ports | ❌ | ✅ |
| Diagnostic logs | ❌ | ✅ |

> **Standard SKU is recommended** for production. Basic is being retired.

### Load Balancer Components

| Component | Purpose |
|-----------|---------|
| **Frontend IP** | Entry point — public or private IP |
| **Backend Pool** | Group of VMs or VMSS that receive traffic |
| **Health Probe** | Detect unhealthy instances (remove from rotation) |
| **Load Balancing Rule** | Map frontend IP:port → backend pool:port |
| **Inbound NAT Rule** | Port forwarding to specific VM (e.g., 50001→VM1:22, 50002→VM2:22) |
| **Outbound Rule** | SNAT configuration for outbound connections (Standard SKU) |

### Health Probes

| Protocol | Port | Path | Interval | Threshold |
|----------|------|------|----------|-----------|
| TCP | Any | — | 5-15 sec | 2 failures = unhealthy |
| HTTP | 80/443/custom | /health | 5-15 sec | 2 failures = unhealthy |
| HTTPS | 443/custom | /health | 5-15 sec | 2 failures = unhealthy |

### Distribution Modes (Session Persistence)

| Mode | Hash | Behavior |
|------|------|----------|
| **None** (default) | 5-tuple (srcIP, srcPort, dstIP, dstPort, protocol) | Best distribution |
| **Client IP** | 2-tuple (srcIP, dstIP) | Same client → same VM |
| **Client IP + Protocol** | 3-tuple (srcIP, dstIP, protocol) | Same client+protocol → same VM |

### Creating Load Balancer

```bash
# Create public LB
az network lb create \
  --resource-group MyRG \
  --name MyLB \
  --sku Standard \
  --frontend-ip-name MyFrontend \
  --backend-pool-name MyBackend \
  --public-ip-address MyPublicIP

# Add health probe
az network lb probe create \
  --resource-group MyRG \
  --lb-name MyLB \
  --name HealthProbe \
  --protocol Tcp \
  --port 80 \
  --interval 5

# Add load balancing rule
az network lb rule create \
  --resource-group MyRG \
  --lb-name MyLB \
  --name HTTPRule \
  --protocol Tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name MyFrontend \
  --backend-pool-name MyBackend \
  --probe-name HealthProbe

# Add VM NIC to backend pool
az network nic ip-config address-pool add \
  --resource-group MyRG \
  --nic-name VM1-NIC \
  --ip-config-name ipconfig1 \
  --lb-name MyLB \
  --address-pool MyBackend
```

---

## 12. Application Gateway

### What It Is
Layer-7 (HTTP/HTTPS) load balancer with WAF, SSL termination, URL routing, and more.

### Key Features

| Feature | Description |
|---------|-------------|
| **URL path-based routing** | Route /images/* to pool A, /api/* to pool B |
| **Multi-site hosting** | Multiple domains on single App Gateway |
| **SSL/TLS termination** | Offload SSL at gateway (decrypt once, not at each VM) |
| **Web Application Firewall (WAF)** | OWASP CRS rules to protect against common attacks |
| **Autoscaling** | v2 SKU auto-scales based on traffic |
| **Session affinity** | Cookie-based (gateway-managed) |
| **Health probes** | HTTP/HTTPS custom path probing |
| **Redirect** | HTTP→HTTPS, URL rewrites |

### Application Gateway vs Load Balancer

| | Azure Load Balancer | Application Gateway |
|-|---------------------|-------------------|
| Layer | 4 (TCP/UDP) | 7 (HTTP/HTTPS) |
| Routing | IP:port hash | URL, host header, cookie |
| WAF | ❌ | ✅ |
| SSL termination | ❌ | ✅ |
| WebSocket | Pass-through | ✅ |
| URL rewrite | ❌ | ✅ |
| Cost | Lower | Higher |
| Use case | Non-HTTP workloads | Web applications |

### Application Gateway SKUs

| SKU | Features |
|-----|---------|
| **Standard v2** | Autoscaling, zone redundancy, URL routing, header rewrites |
| **WAF v2** | All Standard v2 + Web Application Firewall |

### Components

```
Frontend IP (public/private)
  → Listener (port, protocol, host, SSL cert)
    → Routing Rule (basic or path-based)
      → Backend Pool (VMs, VMSS, App Service, IP addresses)
        → HTTP Settings (port, protocol, cookie affinity, timeout)
          → Health Probe (custom path for backend health check)
```

### WAF Modes

| Mode | Behavior |
|------|----------|
| **Detection** | Log attacks but don't block |
| **Prevention** | Block attacks + log |

---

## 13. Network Watcher

### What It Is
Suite of network monitoring and diagnostic tools for Azure.

### Tools & Capabilities

| Tool | Purpose | Input |
|------|---------|-------|
| **IP Flow Verify** | Test if traffic is allowed/denied by NSG | Source/dest IP, port, protocol |
| **Next Hop** | Show routing decision for a packet | Source VM, destination IP |
| **Connection Troubleshoot** | End-to-end connectivity test | Source VM → destination |
| **Effective Security Rules** | View all NSG rules applied to a NIC | NIC resource |
| **NSG Flow Logs** | Log all traffic decisions (allow/deny) | NSG resource |
| **Packet Capture** | Capture network packets on a VM | VM with Network Watcher agent |
| **Topology** | Visual diagram of VNet resources | Resource group |
| **Connection Monitor** | Ongoing connectivity monitoring | Source → destination(s) |
| **VPN Troubleshoot** | Diagnose VPN gateway issues | VPN gateway resource |

### Using Network Watcher

```bash
# IP Flow Verify — is traffic allowed?
az network watcher test-ip-flow \
  --resource-group MyRG \
  --vm MyVM \
  --direction Inbound \
  --protocol TCP \
  --local 10.0.1.4:80 \
  --remote 203.0.113.50:12345

# Next Hop — where does traffic route?
az network watcher show-next-hop \
  --resource-group MyRG \
  --vm MyVM \
  --source-ip 10.0.1.4 \
  --dest-ip 10.0.2.4

# Connection Troubleshoot
az network watcher test-connectivity \
  --resource-group MyRG \
  --source-resource MyVM \
  --dest-address 10.0.2.4 \
  --dest-port 443
```

### NSG Flow Logs
- Log traffic that passes through NSG (source, dest, port, protocol, action)
- Stored in Storage Account (JSON format)
- Analyzed with **Traffic Analytics** (requires Log Analytics workspace)
- Version 1: Basic flow data
- Version 2: + flow state (new, continuing, terminating) + bytes/packets

---

## 14. Service Endpoints & Private Endpoints

### Service Endpoints

| Feature | Detail |
|---------|--------|
| **What** | Optimize route from VNet to Azure PaaS service |
| **How** | Traffic stays on Azure backbone (no public internet) |
| **IP** | Source IP becomes VNet private IP (from PaaS perspective) |
| **Public endpoint** | Service still has public IP (accessible from internet) |
| **Cost** | Free |
| **Configuration** | Enable on subnet → add VNet rule on PaaS service |

### Private Endpoints

| Feature | Detail |
|---------|--------|
| **What** | Private IP address in your VNet for a PaaS service |
| **How** | Maps PaaS service to a NIC with private IP in your subnet |
| **Public exposure** | Can disable public access entirely |
| **DNS** | Requires Private DNS zone for name resolution |
| **Cost** | Per-endpoint charge + data processing |
| **Configuration** | Create Private Endpoint → link Private DNS zone |

### Service Endpoint vs Private Endpoint

| | Service Endpoint | Private Endpoint |
|-|-----------------|-----------------|
| Traffic path | Azure backbone | Azure backbone |
| Service has public IP | ✅ Yes (still exposed) | ❌ Optional (can disable) |
| Source IP (from service view) | VNet private IP | Private IP of PE |
| DNS changes needed | ❌ | ✅ (private DNS zone) |
| Works across peering | ❌ (local subnet only) | ✅ (any connected network) |
| Cost | Free | Charged |
| Security | Medium (restrict to VNet) | **Highest** (fully private) |

> **Private Endpoint > Service Endpoint** for security-sensitive workloads

### Configuring Service Endpoint

```bash
# Enable service endpoint on subnet
az network vnet subnet update \
  --resource-group MyRG \
  --vnet-name MyVNet \
  --name MySubnet \
  --service-endpoints Microsoft.Storage

# Add VNet rule on storage account
az storage account network-rule add \
  --resource-group MyRG \
  --account-name mystorageacct \
  --vnet-name MyVNet \
  --subnet MySubnet
```

---

## 15. Azure Bastion

### What It Is
Managed PaaS service providing secure RDP/SSH access to VMs through the Azure Portal — no public IP needed on VMs.

### Key Features
| Feature | Detail |
|---------|--------|
| **Protocol** | RDP (3389) and SSH (22) over TLS (443) |
| **No public IP on VM** | Connect through Bastion's public IP |
| **No VPN/ExpressRoute needed** | Browser-based, through Azure Portal |
| **NSG on VM** | Only needs to allow Bastion subnet as source |
| **Subnet** | Requires dedicated **AzureBastionSubnet** (/26 minimum) |

### Bastion SKUs

| Feature | Basic | Standard |
|---------|-------|----------|
| Concurrent sessions | 25 | 50 |
| Native client (SSH/RDP) | ❌ | ✅ |
| File upload/download | ❌ | ✅ |
| Shareable link | ❌ | ✅ |
| IP-based connection | ❌ | ✅ |
| Kerberos | ❌ | ✅ |
| Scale units | 2 (fixed) | 2-50 (scalable) |

### Creating Bastion

```bash
# Create AzureBastionSubnet
az network vnet subnet create \
  --resource-group MyRG \
  --vnet-name MyVNet \
  --name AzureBastionSubnet \
  --address-prefix 10.0.255.0/26

# Create Bastion
az network bastion create \
  --resource-group MyRG \
  --name MyBastion \
  --vnet-name MyVNet \
  --public-ip-address BastionPublicIP \
  --sku Standard
```

---

## 16. Exam Tips — Domain 4 Master List

### VNets & Subnets
- **5 reserved IPs per subnet** — /29 is smallest usable (3 IPs)
- **VNets cannot span regions** — use peering for cross-region
- **Address spaces can be added** to existing VNet without downtime
- **Subnet names like GatewaySubnet, AzureBastionSubnet are EXACT** — typos fail

### NSGs
- **Lower priority number = higher priority** (100 evaluated before 200)
- **Default DenyAllInBound** means you MUST add Allow rules for any inbound traffic
- **Standard LB requires NSG** to allow traffic (Basic LB does not)
- **NSG on subnet + NIC** — both must allow (AND logic, NOT OR)
- **Stateful** — if inbound is allowed, return traffic is automatically allowed
- **Service Tags** simplify rules (use "Storage" instead of IP ranges)

### DNS
- **Private DNS Zone must be LINKED** to VNet for resolution to work
- **Auto-registration** — only 1 VNet per zone can have it enabled
- **Alias records** solve the zone-apex CNAME problem (point @ to Azure resource)
- **Zone delegation** — update NS records at registrar to Azure DNS name servers

### Peering
- **NON-TRANSITIVE** — A↔B + B↔C ≠ A↔C (most commonly tested concept)
- **Bi-directional setup** — must create peering from both VNets
- **No overlapping address spaces** — will fail if CIDRs overlap
- **Gateway transit** — spokes can use hub's VPN gateway (enable on both sides)
- **Cross-subscription and cross-tenant** — supported with proper permissions

### Connectivity
- **VPN Gateway takes ~45 minutes** to deploy (GatewaySubnet required)
- **ExpressRoute is NOT encrypted** by default (private but not encrypted)
- **P2S VPN** — clients must re-download config after certificate changes
- **Active-active VPN** — two public IPs, each connected to on-prem device

### Load Balancing
- **Standard LB: closed by default** (must add NSG Allow rule for traffic to flow)
- **Basic LB: no SLA** — always choose Standard for production
- **Health probes** — unhealthy VMs removed from rotation (not stopped)
- **Inbound NAT rules** — port forwarding (50001→VM1:22, 50002→VM2:22)
- **App Gateway** = Layer 7 (HTTP/HTTPS), WAF, URL routing
- **Load Balancer** = Layer 4 (TCP/UDP), faster, simpler

### Private Connectivity
- **Private Endpoint > Service Endpoint** for security
- **Service Endpoints** — free, but PaaS still has public IP
- **Private Endpoints** — fully private IP, requires Private DNS zone
- **Bastion** — connect to VMs without public IP (browser-based RDP/SSH)

---

*Next: [Domain 5 — Monitor and Maintain Azure Resources](AZ-104-Monitor-Maintain.md)*
