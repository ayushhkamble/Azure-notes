
# Day 10 — Azure Administration
### Topics: VNet Peering (Intra & Inter-Region) | VNet-to-VNet Connections | ExpressRoute | VPN Gateway

---

## 1. VNet Peering: Intra-Region and Inter-Region

### Recap: What is VNet Peering? (from Day 8)
VNet Peering connects two VNets so resources can communicate privately over Azure's backbone network, without traffic crossing the public internet.

### Two Flavors of Peering

| Type | Also Called | Scope |
|---|---|---|
| **Intra-region (Regional) Peering** | Same-region peering | Both VNets live in the **same** Azure region |
| **Inter-region Peering** | **Global VNet Peering** | VNets live in **different** Azure regions |

### How It Works
<cite index="117-1">Azure VNET Peering is a way to securely connect two Virtual Networks within Azure. When connected in the same region, latency between the two networks will be the same as within a single network. When peering VNets in different regions, traffic flows across the Azure backbone network to the new region.</cite>

<cite index="117-1">VNET Peering tends to be faster than a VPN because traffic does not need to be encrypted and passed across VPN Gateways.</cite>

### Cost Difference — An Important Teaching Point
Both peering types are billed for data transfer, but at very different rates:

| Peering Type | Approx. Data Transfer Cost |
|---|---|
| **Regional (intra-region) peering** | <cite index="111-1">$0.01/GB for both inbound and outbound traffic</cite> |
| **Global (inter-region) peering** | <cite index="109-1">Starting from $0.035/GB, depending on the zones involved</cite> |

<cite index="110-1">Inbound and outbound traffic is charged at both ends of the peered networks</cite> — so a Zone 1-to-Zone 1 global peering transfer effectively costs **$0.07/GB total** (both directions combined), while regional peering is far cheaper at roughly **$0.02/GB total**.

> 🎯 **Teaching point:** This cost difference is a real architectural consideration — if a workload doesn't *need* to span regions, keeping VNets in the same region and using regional peering is significantly cheaper at scale.

### Cross-Subscription and Cross-Tenant Peering
<cite index="117-1">VNets can be peered even when they exist in different subscriptions or different Microsoft Entra tenants</cite> — useful for organizations managing multiple business units or environments (dev/test/prod) under separate subscriptions.

### Important Limitation — Non-Transitive (Recap from Day 8)
Whether regional or global, peering is still **non-transitive**: VNet A peered with VNet B, and VNet B peered with VNet C, does **not** let A reach C automatically.

📘 **Official Docs:**
- [Virtual network peering overview – Microsoft Learn](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview)
- [Virtual Network pricing – Microsoft Azure](https://azure.microsoft.com/en-us/pricing/details/virtual-network/)

### 🧪 Practice Lab
1. Create two VNets in the **same region** (e.g., both East US) and peer them (reuse the Day 8 lab steps if needed).
2. Create a third VNet in a **different region** (e.g., West Europe):
   ```bash
   az network vnet create \
     --resource-group rg-network-day8 \
     --name vnet-demo-3 \
     --address-prefix 10.2.0.0/16 \
     --location westeurope \
     --subnet-name subnet-3a \
     --subnet-prefix 10.2.1.0/24
   ```
3. Set up **global peering** between `vnet-demo` (East US) and `vnet-demo-3` (West Europe) — same commands as regional peering, Azure detects the region difference automatically:
   ```bash
   az network vnet peering create \
     --resource-group rg-network-day8 \
     --name peer-demo-to-demo3 \
     --vnet-name vnet-demo \
     --remote-vnet vnet-demo-3 \
     --allow-vnet-access
   ```
4. Discuss: a company has a database VNet in East US and an analytics VNet in Southeast Asia that syncs 500GB/month. Estimate the rough monthly global peering cost, and discuss whether a regional consolidation might make sense.

---

## 2. VNet-to-VNet Connections

### What is VNet-to-VNet?
<cite index="104-1">VNet-to-VNet is a VPN connection over IPsec (IKEv2) between two VPN Gateways — it doesn't require a separate on-premises VPN device, and functions the same way as a site-to-site configuration, just between two Azure VNets instead of an on-premises site and Azure.</cite>

### VNet-to-VNet vs. VNet Peering — When to Use Which
This is one of the most commonly confused topics — a clear comparison matters:

| Aspect | VNet Peering | VNet-to-VNet (VPN Gateway) |
|---|---|---|
| **Requires a Gateway?** | ❌ No | ✅ Yes — a VPN Gateway on each VNet |
| **Traffic path** | Azure's private backbone network | Encrypted IPsec tunnel (still Azure backbone, but with encryption overhead) |
| **Speed/Latency** | <cite index="117-1">Faster — no encryption overhead</cite> | Slightly slower due to IPsec encryption/decryption |
| **Cost** | Data transfer charges only | Data transfer charges **+** VPN Gateway hourly cost |
| **Overlapping IPs?** | ❌ Not allowed | ❌ Not allowed |
| **Best for** | Most VNet-to-VNet connectivity needs — the default choice | Scenarios needing encryption in transit, or connecting to an environment that already uses VPN elsewhere |

<cite index="102-1">In some cases, you might want to use virtual network peering instead of VNet-to-VNet to connect your virtual networks — peering doesn't use a virtual network gateway.</cite> In modern Azure architecture, **VNet Peering is the default/preferred choice** for VNet-to-VNet connectivity; VPN-based VNet-to-VNet is now mainly used in specific hybrid or legacy scenarios.

### Gateway Transit — A Related Peering Feature
<cite index="102-1">You can configure a site-to-site VPN and ExpressRoute connections for the same virtual network, and configure peered VNets to use "Allow gateway transit" so peered VNets can share a single VNet's on-premises connectivity, instead of each needing its own gateway.</cite>

<cite index="117-1">Gateway Transit with VNet Peering allows peered connections to access a remote VNet's gateway — but only in one direction: if two VNets each have their own VPN Gateway connecting to separate on-premises sites, only one of the VNets can be configured to communicate across the other VNet's gateway, not both simultaneously.</cite>

📘 **Official Docs:**
- [About VPN Gateway – Microsoft Learn](https://learn.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpngateways)
- [VPN Gateway FAQ – Microsoft Learn](https://learn.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-vpn-faq)

### 🧪 Practice Lab (Conceptual + Portal Exploration)
1. In the Portal, search **"Virtual network gateways"** → **+ Create** → walk through the setup screen (don't need to complete, since Gateway deployment takes 30-45 minutes and incurs cost).
2. Note the **Connection type** dropdown — observe **VNet-to-VNet** as one of the available options alongside Site-to-site and ExpressRoute.
3. Discuss: your company has two VNets that need to talk to each other constantly, with no special encryption requirement beyond what Azure already provides. Which option (Peering or VNet-to-VNet) would you pick, and why?

---

## 3. ExpressRoute Overview and Use Cases

### What is ExpressRoute?
<cite index="93-1">ExpressRoute lets you extend your on-premises networks into the Microsoft cloud over a private connection with the help of a connectivity provider.</cite> <cite index="101-1">ExpressRoute connections don't go over the public internet — they offer higher security, reliability, and speeds, with lower and consistent latencies than typical internet connections.</cite>

### How You Connect
<cite index="98-1">You can connect from an any-to-any (IP VPN) network, a point-to-point Ethernet connection, or through a virtual cross-connection via an Ethernet exchange (colocation facility).</cite> <cite index="100-1">There are four connectivity models: CloudExchange Colocation, Point-to-point Ethernet Connection, Any-to-any (IPVPN) Connection, and ExpressRoute Direct.</cite>

### Key Characteristics
- <cite index="98-1">Provides Layer 3 connectivity between your on-premises network and the Microsoft cloud through a connectivity provider</cite>
- <cite index="98-1">Uses dynamic routing between your network and Azure via BGP (Border Gateway Protocol)</cite>
- <cite index="93-1">You can connect to Microsoft services across all regions within the same geopolitical region — and with the Premium add-on, globally across all regions</cite>
- <cite index="94-1">ExpressRoute Direct provides massive data ingestion capability for services like Cosmos DB, physical isolation for regulated industries, and control of circuit distribution by business unit</cite>

### When to Use ExpressRoute — Real Use Cases
<cite index="95-1">Organizations typically consider ExpressRoute when they need their network to reliably handle rich business application traffic, and when the predictability of traffic supporting mission-critical applications matters more than for other, less-critical internet traffic.</cite>

| Use Case | Why ExpressRoute Fits |
|---|---|
| **Regulatory/compliance-sensitive workloads** | <cite index="97-1">Private, compliant access supports standards like HIPAA, PCI-DSS, or FedRAMP</cite> |
| **Large-scale data migration** | <cite index="97-1">Consistent low-latency connectivity minimizes disruption during migration</cite> |
| **Disaster recovery / replication** | <cite index="97-1">Data replicates faster and more securely, with minimal downtime on failover, helping meet RTO/RPO objectives</cite> |
| **Real-time / latency-sensitive analytics** | <cite index="97-1">Secure, high-bandwidth pipelines for scenarios like genomics or banking transaction processing</cite> |
| **Hybrid infrastructure extension** | <cite index="97-1">Seamless integration of on-premises networks with Azure VNets, supporting hybrid workloads across environments</cite> |

### Resiliency Options
<cite index="96-1">When creating an ExpressRoute circuit, you choose a resiliency level: Standard resiliency provides diverse connections to distinct Microsoft Enterprise Edge (MSEE) devices in a single location, while High resiliency (ExpressRoute Metro) provides diverse connections to MSEE routers in multiple locations within one metro area, appearing as a single connection.</cite>

<cite index="101-1">Multi-site deployments — using multiple ExpressRoute circuits in different peering locations, or ExpressRoute Metro — protect connectivity in case one location becomes unavailable. Microsoft recommends implementing multiple circuits in different peering locations to avoid single points of failure when accessing Azure public services like Storage or SQL, or Microsoft 365.</cite>

### ExpressRoute vs. VPN Gateway — Quick Distinction
| Aspect | ExpressRoute | VPN Gateway (Site-to-Site) |
|---|---|---|
| **Path** | Private connection via connectivity provider | <cite index="106-1">Encrypted tunnel over the public internet</cite> |
| **Latency/Reliability** | Consistent, predictable | Variable (depends on internet conditions) |
| **Setup complexity** | Higher — requires a connectivity provider | Lower — can be set up entirely within Azure |
| **Cost** | Higher (circuit + provider fees) | Lower |
| **Best for** | Enterprise, compliance-heavy, high-bandwidth needs | Smaller-scale, quick-to-deploy hybrid connectivity |

> 💡 They aren't mutually exclusive — <cite index="102-1">a site-to-site VPN can be configured as a secure failover path for ExpressRoute</cite>, combining ExpressRoute's performance with VPN's resilience as a backup.

📘 **Official Docs:**
- [Azure ExpressRoute Overview – Microsoft Learn](https://learn.microsoft.com/en-us/azure/expressroute/expressroute-introduction)
- [ExpressRoute FAQ – Microsoft Learn](https://learn.microsoft.com/en-us/azure/expressroute/expressroute-faqs)

### 🧪 Practice Lab (Conceptual — ExpressRoute requires a connectivity provider and isn't practical to fully deploy in a training lab)
1. In the Portal, search **"ExpressRoute circuits"** → **+ Create** → walk through the setup form to see the required inputs (Provider, Peering Location, Bandwidth, SKU).
2. Note the **SKU tiers** available (Local, Standard, Premium) and how they affect regional vs. global reach.
3. Discuss: a hospital network needs to transmit large medical imaging files to Azure daily, with strict HIPAA compliance and predictable performance. Would you recommend ExpressRoute, VPN Gateway, or both together? Justify your answer.

---

## 4. VPN Gateway Overview

### What is Azure VPN Gateway?
<cite index="106-1">Azure VPN Gateway sends encrypted traffic between an Azure virtual network and on-premises locations over the public internet</cite>, and can also connect Azure VNets to each other.

### Connection Types

| Type | Description | Requires VPN Device? |
|---|---|---|
| **Site-to-Site (S2S)** | <cite index="106-1">A cross-premises IPsec/IKE VPN tunnel connection between the VPN Gateway and an on-premises VPN device</cite> | ✅ Yes (on-premises) |
| **Point-to-Site (P2S)** | <cite index="102-1">A secure connection to your virtual network from an individual client computer — useful for telecommuters connecting from home or a conference, or when only a few clients need access</cite> | ❌ No |
| **VNet-to-VNet** | <cite index="106-1">An IPsec/IKE VPN tunnel connection between the VPN Gateway and another Azure VPN gateway, designed specifically for VNet-to-VNet connections</cite> | ❌ No |

### Point-to-Site Details
- <cite index="102-1">Unlike site-to-site connections, point-to-site connections don't require an on-premises public-facing IP address or a VPN device</cite>
- <cite index="103-1">Supports OpenVPN, IKEv2, or SSTP protocols</cite>
- <cite index="102-1">Point-to-site connections can be used alongside site-to-site connections through the same VPN gateway</cite>, letting you support both branch offices and remote individual users simultaneously

### P2S Routing Behavior with Peered VNets — Important Nuance
<cite index="105-1">P2S VPN routing behavior depends on the client OS, the protocol used, and how the VNets are connected to each other.</cite> <cite index="105-1">Access to peered VNets through a P2S connection isn't transitive and is limited to only directly peered VNets — and if changes are made to VNet peering or network topology, Windows VPN clients must download and reinstall the updated VPN client package for the changes to take effect.</cite>

### High Availability
<cite index="106-1">VPN gateways can be deployed in Azure Availability Zones — availability zone deployments bring resiliency, scalability, and higher availability to virtual network gateways.</cite>

### Deploying a Site-to-Site VPN Gateway

**CLI:**
```bash
az network public-ip create --resource-group rg-network-day8 --name pip-vpngw --sku Standard

az network vnet subnet create \
  --resource-group rg-network-day8 \
  --vnet-name vnet-demo \
  --name GatewaySubnet \
  --address-prefix 10.0.255.0/27

az network vnet-gateway create \
  --resource-group rg-network-day8 \
  --name vpngw-demo \
  --public-ip-address pip-vpngw \
  --vnet vnet-demo \
  --gateway-type Vpn \
  --vpn-type RouteBased \
  --sku VpnGw1
```

> ⚠️ Note: the subnet **must** be named exactly `GatewaySubnet` (recall the naming pattern from Day 8's `AzureBastionSubnet`) and VPN Gateway deployment typically takes **30-45 minutes**.

### VPN Gateway vs ExpressRoute vs VNet Peering — Full Decision Guide
```
Need to connect 2 Azure VNets, no encryption requirement, lowest cost/latency?
        → VNet Peering (default choice)

Need to connect 2 Azure VNets over an encrypted VPN tunnel specifically?
        → VNet-to-VNet (VPN Gateway)

Need to connect on-premises to Azure, moderate scale, cost-sensitive?
        → Site-to-Site VPN Gateway

Need individual remote users to connect to Azure directly?
        → Point-to-Site VPN Gateway

Need enterprise-grade, private, high-bandwidth, compliance-critical
on-premises-to-Azure connectivity?
        → ExpressRoute (optionally with VPN as failover)
```

📘 **Official Docs:**
- [About Azure VPN Gateway – Microsoft Learn](https://learn.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpngateways)
- [About Point-to-Site VPN – Microsoft Learn](https://learn.microsoft.com/en-us/azure/vpn-gateway/point-to-site-about)
- [VPN Gateway topologies and design – Microsoft Learn](https://learn.microsoft.com/en-us/azure/vpn-gateway/design)

### 🧪 Practice Lab
1. In the Portal, search **"Virtual network gateways"** → **+ Create**, and walk through configuring a Site-to-Site gateway (SKU, VPN type, Gateway subnet) without necessarily completing deployment (to avoid the 30-45 min wait / cost, unless your training environment has time budgeted).
2. Compare the setup screens for **Point-to-Site** configuration (Portal → your VPN Gateway → Point-to-site configuration) — note the certificate/Entra ID authentication options required.
3. Fill out the full decision guide above with a real scenario your students propose (e.g., "a 50-person branch office," "a single remote consultant," "a bank's core transaction system") and have them justify the correct connectivity choice for each.

---

## Quick Recap Table

| Concept | One-Line Summary |
|---|---|
| **Intra vs Inter-Region Peering** | Same-region peering is cheaper (~$0.01/GB); Global (cross-region) peering costs more (~$0.035/GB per direction) |
| **VNet-to-VNet** | VPN Gateway-based connection between VNets — encrypted but slower/costlier than plain Peering, which is the modern default |
| **ExpressRoute** | Private, non-internet connection from on-premises to Azure — best for compliance-heavy, high-bandwidth, mission-critical scenarios |
| **VPN Gateway** | Encrypted tunnels over the public internet — Site-to-Site (branch offices), Point-to-Site (individual remote users), VNet-to-VNet (Azure-to-Azure) |

> 🎯 **Key takeaway:** Azure gives you a full spectrum of connectivity options — from free/fast **VNet Peering** for Azure-to-Azure traffic, to **VPN Gateway** for encrypted internet-based hybrid connectivity, up to **ExpressRoute** for enterprise-grade private connections. The right choice always comes down to balancing **cost, latency, encryption needs, and compliance requirements** for the specific workload.


---
