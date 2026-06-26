# Module 5 — Networking and Content Delivery 🌐

> AWS Academy Cloud Foundations | Module 5

---

## 📌 Overview

This module covers the three core AWS networking and content delivery
services: Amazon VPC, Amazon Route 53, and Amazon CloudFront. Networking
knowledge is foundational — every resource you deploy in AWS lives inside
a network, and understanding how to configure, secure, and connect that
network is essential.

**Topics covered:**
- Networking basics (IP addressing, CIDR, OSI model)
- Amazon VPC — architecture, subnets, IP addressing, route tables
- VPC networking — gateways, peering, VPN, Direct Connect, Transit Gateway
- VPC security — security groups vs network ACLs
- Amazon Route 53 — DNS, routing policies, failover
- Amazon CloudFront — CDN, edge locations, benefits, pricing

---

## 🧠 Part 1 — Networking Basics

### What is a Computer Network?

A **computer network** is two or more client machines connected together to
share resources. Networks can be logically partitioned into **subnets**, and
require a networking device (router or switch) to connect clients and enable
communication.

Each machine in a network has a unique **IP address** — a numerical label
that identifies it on the network.

---

### IPv4 vs IPv6

| Feature | IPv4 | IPv6 |
|---------|------|------|
| **Bit length** | 32 bits | 128 bits |
| **Format** | 4 decimal groups separated by dots | 8 hexadecimal groups separated by colons |
| **Example** | `192.0.2.0` | `2600:1f18:22ba:8c00:ba86:a05e:a5ba:00FF` |
| **Range per group** | 0–255 (8 bits each) | 0–FFFF (16 bits each) |
| **Address capacity** | ~4.3 billion | 340 undecillion (accommodates far more devices) |

---

### CIDR — Classless Inter-Domain Routing

CIDR is the standard method for expressing a range of consecutive IP
addresses. Format: `IP_address/prefix_length`

```
192.0.2.0/24
│           │
│           └── 24 bits fixed → last 8 bits flexible → 2⁸ = 256 addresses
└── First address of the network range
```

**Practical examples:**

| CIDR | Fixed bits | Flexible bits | Available addresses | Range |
|------|-----------|--------------|--------------------|----|
| `192.0.2.0/24` | 24 | 8 | 2⁸ = 256 | 192.0.2.0 – 192.0.2.255 |
| `192.0.2.0/16` | 16 | 16 | 2¹⁶ = 65,536 | 192.0.0.0 – 192.0.255.255 |
| `192.0.2.0/32` | 32 | 0 | 1 (single host) | Specific host only |
| `0.0.0.0/0` | 0 | 32 | All (represents the entire internet) | Any IP address |

> 💡 `0.0.0.0/0` represents **all internet traffic** and is used in route
> tables to send non-local traffic to an internet gateway. `x.x.x.x/32` is
> used in firewall rules to allow/deny a single specific IP address.

---

### OSI Model (Brief Reference)

The **Open Systems Interconnection (OSI) model** explains how data travels
over a network across 7 layers. Relevant to VPC networking:

| Layer | Name | Example devices/protocols |
|-------|------|--------------------------|
| 2 | Data Link | Hubs, switches |
| 3 | Network | Routers, IP addresses |
| 7 | Application | HTTP, HTTPS, DNS |

Understanding these layers helps explain how VPC components (security groups,
route tables, internet gateways) interact with each other.

---

## 🏗️ Part 2 — Amazon VPC

### What is Amazon VPC?

**Amazon Virtual Private Cloud (Amazon VPC)** lets you provision a logically
isolated section of the AWS Cloud where you launch your AWS resources. You
have full control over:
- Your own IP address range
- Subnet creation
- Route table configuration
- Network gateway configuration

You can use both IPv4 and IPv6 in a VPC. You can create **public subnets**
(web servers with internet access) and **private subnets** (databases with
no public internet access), and protect them with security groups and
network ACLs.

---

### VPCs and Subnets

```
AWS Cloud
└── Region
    └── VPC  (logically isolated, belongs to one Region, spans multiple AZs)
        ├── Availability Zone 1
        │   └── Subnet  (public or private, belongs to one AZ)
        └── Availability Zone 2
            └── Subnet  (public or private, belongs to one AZ)
```

| Component | Key properties |
|-----------|---------------|
| **VPC** | Logically isolated; dedicated to your account; belongs to one Region; can span multiple AZs; requires a CIDR block |
| **Subnet** | Range of IP addresses within a VPC; belongs to one AZ only; classified as public or private |

- **Public subnet** — has direct access to the internet
- **Private subnet** — no direct internet access

---

### VPC IP Addressing

**CIDR block size constraints:**
- Maximum: `/16` → 2¹⁶ = 65,536 addresses
- Minimum: `/28` → 2⁴ = 16 addresses

> ⚠️ **You cannot change the CIDR block after creating a VPC** — choose
> carefully. Plan for future growth.

**Rules for subnet CIDR blocks:**
- Subnet CIDR can be the same as the VPC CIDR (single subnet = entire VPC)
- Subnet CIDR can be a subset of VPC CIDR (multiple subnets)
- **Subnet CIDRs cannot overlap** — no duplicate IP addresses in the same VPC

---

### Reserved IP Addresses

For **every subnet CIDR block**, AWS reserves **5 IP addresses** that are
not available for use:

Example: Subnet `10.0.0.0/24` (256 total IPs → only **251 available**)

| Reserved IP | Purpose |
|-------------|---------|
| `10.0.0.0` | Network address |
| `10.0.0.1` | VPC local router (internal communications) |
| `10.0.0.2` | DNS resolution |
| `10.0.0.3` | Future use (reserved by AWS) |
| `10.0.0.255` | Network broadcast address |

**Larger example:** VPC `10.0.0.0/16` divided into four equal `/24` subnets:
- Each subnet has 256 total IPs but only **251 usable** (5 reserved per subnet)

---

### IP Address Types for EC2 Instances

| Type | Description |
|------|-------------|
| **Private IP** | Assigned automatically to every instance in a VPC |
| **Public IP** | Optional; assigned at launch by modifying subnet auto-assign settings; changes if instance is stopped/restarted |
| **Elastic IP** | Static public IPv4 address; persists even if instance is stopped; can be remapped to another instance to mask failures |

> 💡 **Elastic IP best practice:** Release Elastic IPs when no longer needed —
> AWS charges for allocated Elastic IPs that are not associated with a running
> instance.

**Elastic Network Interface (ENI):**
- A virtual network interface that can be attached to or detached from an
  EC2 instance
- When moved to a new instance, all its attributes (private IP, MAC address,
  security groups) follow it — network traffic redirects automatically
- Each instance has a **primary network interface** that cannot be detached
- You can attach additional ENIs; the number varies by instance type

---

### Route Tables and Routes

A **route table** contains a set of rules (routes) that direct network
traffic from your subnet.

| Concept | Description |
|---------|-------------|
| **Route** | A rule with a destination (CIDR) and a target (where to send traffic) |
| **Local route** | Built-in; enables communication within the VPC; **cannot be deleted** |
| **Main route table** | Default; automatically assigned to subnets not explicitly associated with another route table |

**Default route table example:**

| Destination | Target |
|-------------|--------|
| `10.0.0.0/16` | `local` ← VPC CIDR block, enables internal routing |

> 💡 Each subnet must be associated with exactly **one** route table at a
> time. Multiple subnets can share the same route table.

**Making a subnet public** requires adding a route to an internet gateway:

| Destination | Target |
|-------------|--------|
| `10.0.0.0/16` | `local` |
| `0.0.0.0/0` | `igw-id` ← internet gateway |

---

### Section 2 Key Takeaways

- VPC = logically isolated section of AWS Cloud; belongs to one Region; requires a CIDR block
- Subnets belong to one AZ; require a CIDR block; classified public or private
- Route tables control subnet traffic; have a built-in local route that cannot be deleted
- AWS reserves 5 IPs per subnet CIDR block

---

## 🔌 Part 3 — VPC Networking

### Internet Gateway

An **internet gateway** is a scalable, redundant, highly available VPC
component that enables communication between VPC instances and the internet.

Two purposes:
1. Provides a target in route tables for internet-routable traffic
2. Performs **NAT (Network Address Translation)** for instances with public IPv4 addresses

To make a subnet public:
1. Attach an internet gateway to the VPC
2. Add a route to the route table: destination `0.0.0.0/0` → target `igw-id`

---

### NAT Gateway

A **NAT gateway** allows instances in a **private subnet** to connect
outbound to the internet or other AWS services, while preventing the
internet from initiating inbound connections to those instances.

**Setup steps:**
1. Create the NAT gateway in a **public subnet**
2. Associate an **Elastic IP address** with it
3. Update the **private subnet's route table** to point internet-bound traffic
   (`0.0.0.0/0`) to the NAT gateway

> 💡 **NAT Gateway vs NAT Instance:** AWS recommends NAT Gateway for most
> use cases — it is a managed service with better availability, higher
> bandwidth, and less administrative overhead than a self-managed NAT instance.

---

### VPC Sharing

**VPC sharing** allows multiple AWS accounts within the **same AWS
Organization** to create resources into shared, centrally managed VPCs.

- **Owner account** shares subnets with **participant accounts**
- Participants can create, view, modify, and delete their own resources in
  shared subnets
- Participants **cannot** view or modify resources belonging to other
  participants or the VPC owner

**Benefits of VPC sharing:**

| Benefit | Detail |
|---------|--------|
| **Separation of duties** | Centrally controlled VPC structure, routing, IP allocation |
| **Ownership** | App owners retain ownership of their resources and security groups |
| **Security groups** | Participants can reference each other's security group IDs |
| **Efficiency** | Higher subnet density; efficient reuse of VPNs and Direct Connect |
| **No hard limits** | Avoids per-account limits (e.g. 50 virtual interfaces per Direct Connect) |
| **Cost optimization** | Reuse NAT gateways, VPC interface endpoints, and intra-AZ traffic |

---

### VPC Peering

A **VPC peering connection** is a direct networking connection between two
VPCs that enables private routing between them — as if they were on the
same network.

- Can peer between your own VPCs, VPCs in other AWS accounts, or VPCs in
  different AWS Regions
- Requires **route table updates** in both VPCs

**Setup example — Peering VPC A ↔ VPC B:**

| VPC A Route Table | | VPC B Route Table | |
|------------------|-|------------------|-|
| Destination | Target | Destination | Target |
| VPC B IP range | peering-id | VPC A IP range | peering-id |

**VPC Peering restrictions:**
- **IP ranges cannot overlap** between peered VPCs
- **No transitive peering** — if A↔B and A↔C, then B cannot reach C through A;
  a direct B↔C connection must be explicitly established
- **Only one peering connection** between the same two VPCs

---

### AWS Site-to-Site VPN

Connects your VPC to a remote network (on-premises data center) over the
internet using an encrypted tunnel.

**Setup steps:**
1. Create a **virtual private gateway (VGW)** and attach it to your VPC
2. Define the **customer gateway** (AWS resource providing info about your
   VPN device)
3. Create a **custom route table** pointing corporate traffic to the VGW;
   update security group rules
4. Establish the **Site-to-Site VPN connection**
5. Configure routing to pass traffic through the connection

---

### AWS Direct Connect (DX)

**AWS Direct Connect** provides a **dedicated, private network connection**
between your data center or office and AWS — not over the public internet.

| Feature | Detail |
|---------|--------|
| **Connection type** | Private, dedicated physical link |
| **Protocol** | Open standard 802.1q VLANs |
| **Benefits** | Lower network costs; higher bandwidth throughput; more consistent latency than internet-based connections |
| **Use case** | Data centers far from an AWS Region where internet performance is insufficient |

---

### VPC Endpoints

A **VPC endpoint** enables private connections from your VPC to supported
AWS services **without** requiring an internet gateway, NAT device, VPN, or
Direct Connect.

- Traffic **never leaves the Amazon network**
- Instances do not need public IP addresses

**Two types:**

| Type | What it connects to | Cost |
|------|-------------------|------|
| **Interface endpoint** | Services powered by AWS PrivateLink (some AWS services, APN partner services, Marketplace services) | Hourly usage + data processing fees |
| **Gateway endpoint** | Amazon S3 and DynamoDB only | No additional charge |

---

### AWS Transit Gateway

**AWS Transit Gateway** solves the complexity of connecting many VPCs at
scale using a **hub-and-spoke model**.

**The problem it solves:**
- With VPC peering, connections are point-to-point — as VPCs grow, the
  number of connections grows exponentially
- On-premises VPN must be attached to each individual VPC separately

**How Transit Gateway works:**
```
On-premises DC ──────┐
                      │
VPC A ───────────── Transit Gateway (hub)
VPC B ───────────────┘
VPC C ───────────────┘
Remote office ───────┘
```

- Each network connects to the **Transit Gateway once**
- Any new VPC connected to Transit Gateway is automatically reachable by
  all other connected networks
- Centralized routing policies replace per-VPC management

---

### VPC Networking Options Summary

| Option | Connects to | Key use |
|--------|------------|---------|
| **Internet Gateway** | Public internet | Make subnet public |
| **NAT Gateway** | Internet (outbound only) | Private subnet internet access |
| **VPC Endpoint** | AWS services | Private connectivity without internet |
| **VPC Peering** | Other VPCs | Private routing between VPCs |
| **VPC Sharing** | Other AWS accounts | Shared centrally managed VPCs |
| **Site-to-Site VPN** | Remote network | Encrypted tunnel over internet |
| **Direct Connect** | Remote network | Dedicated private physical connection |
| **Transit Gateway** | Many VPCs + on-premises | Hub-and-spoke at scale |

---

### Activity: VPC Architecture Solution

From the slide diagram, a complete VPC architecture with:
- **Public subnet** (`10.0.1.0/24`) containing NAT Gateway with private IP
- **Private subnet** (`10.0.2.0/24`) containing EC2 instance with Elastic
  Network Interface and private IP address
- **Internet Gateway** attached to VPC (`10.0.0.0/16`)
- **Route table** with two entries:

| Destination | Target |
|-------------|--------|
| `10.0.0.0/16` | `local` |
| `0.0.0.0/0` | `igw-id` |

> 💡 The NAT Gateway lives in the **public subnet** and uses the internet
> gateway for outbound internet access. Private subnet instances route
> through the NAT Gateway (not the internet gateway directly).

---

## 🔒 Part 4 — VPC Security

### Security Groups

A **security group** acts as a **virtual firewall at the instance level**,
controlling inbound and outbound traffic per EC2 instance.

**Default behavior:**
- **Inbound:** No rules → all inbound traffic blocked by default
- **Outbound:** One rule → all outbound traffic allowed by default

**Key characteristic — Stateful:**
- If you send a request from your instance, the **response is automatically
  allowed** in regardless of inbound rules
- If inbound traffic is allowed, its **response is automatically allowed**
  out regardless of outbound rules

**Rule evaluation:** All rules are evaluated simultaneously before a
decision is made to allow or deny traffic.

---

### Network ACLs

A **network ACL (Access Control List)** is an **optional** additional layer
of security operating at the **subnet level**.

**Default behavior:**
- Default network ACL allows all inbound and outbound IPv4 and IPv6 traffic
- Custom network ACLs deny all traffic by default until rules are added

**Key characteristic — Stateless:**
- Return traffic must be **explicitly allowed** by rules
- Both directions (inbound and outbound) must be configured independently

**Rule evaluation:** Rules are evaluated in **number order** (lowest number
first) before a decision is made. The first matching rule is applied.

Each subnet must be associated with exactly one network ACL. If not
explicitly associated, the default network ACL is used.

---

### Security Groups vs Network ACLs — Side-by-Side

| Attribute | Security Groups | Network ACLs |
|-----------|----------------|-------------|
| **Scope** | Instance level | Subnet level |
| **Supported rules** | Allow rules only | Allow AND deny rules |
| **State** | **Stateful** — return traffic automatically allowed | **Stateless** — return traffic must be explicitly allowed |
| **Rule evaluation** | All rules evaluated before decision | Rules evaluated in number order; first match applies |
| **Default behavior** | No inbound; all outbound allowed | Default ACL allows all; custom ACL denies all |

> 💡 **Use both together for defense in depth:** Security groups protect
> individual instances; network ACLs protect entire subnets. A misconfigured
> security group doesn't expose the subnet if the network ACL is restrictive.

---

### Section 4 Key Takeaways

- Build security into your VPC architecture
- Isolate subnets where possible
- Choose appropriate gateway device or VPN connection
- Use firewalls — both security groups (instance level) and network ACLs
  (subnet level) are firewall options for securing your VPC

---

## 🌍 Part 5 — Amazon Route 53

### What is Amazon Route 53?

**Amazon Route 53** is a highly available and scalable cloud **DNS (Domain
Name System)** web service. It translates human-readable domain names (like
`www.example.com`) into machine-readable IP addresses (like `192.0.2.1`).

**Key capabilities:**
- Routes users to AWS infrastructure (EC2, ELB, S3) and external resources
- Configures DNS health checks to route traffic only to healthy endpoints
- Supports domain name registration (purchase and manage domains)
- Fully IPv6 compliant

---

### Route 53 Routing Policies

| Policy | When to use | How it works |
|--------|------------|-------------|
| **Simple (round robin)** | Single resource serving your domain | Returns the single configured value |
| **Weighted round robin** | Route traffic to multiple resources in set proportions | Assign weights 0–255; traffic distributed proportionally (e.g. weight 3 = 75%, weight 1 = 25%) — useful for A/B testing |
| **Latency (LBR)** | Resources in multiple Regions; want best performance | Routes to the Region with the lowest measured latency for that user |
| **Geolocation** | Route users based on geographic location | Localizes content; restricts distribution by location; consistent endpoint per user location |
| **Geoproximity** | Route based on resource location with optional traffic shifting | Shifts traffic between locations optionally; useful for gradual migration |
| **Failover (DNS failover)** | Active-passive failover configuration | Health checks monitor endpoints; routes to secondary if primary fails |
| **Multivalue answer** | DNS-level load distribution with health checking | Returns up to 8 randomly selected healthy records — not a load balancer replacement, but improves availability |

---

### Multi-Region Deployment with Route 53

Route 53 automatically directs users to the **geographically closest**
Elastic Load Balancing load balancer:

```
User
  ↓
Amazon Route 53 (latency-based routing)
  ↓
Nearest ELB load balancer (Region A or Region B)
  ↓
EC2 instances (Auto Scaling group across AZs)
  ↓
Amazon RDS (in each Region)
```

**Benefits:**
- **Latency-based routing** → routes to the nearest Region
- **Load balancing routing** → distributes within the Region across AZs

---

### DNS Failover for a Multi-Tiered Web Application

Route 53 enables high availability through active-passive failover:

**Setup:**
1. Create two DNS records for CNAME `www` with **Failover routing policy**:
   - **Primary** record → points to the ELB load balancer (web app stack)
   - **Secondary** record → points to a static Amazon S3 website (backup)
2. Enable **Route 53 health checks** on the primary endpoint

**Behavior:**
- If primary is healthy → all traffic goes to the web application stack
- If primary fails (web server down OR database down) → Route 53 
  automatically fails over to the S3 static website

---

### Section 5 Key Takeaways

- Route 53 is a highly available and scalable cloud DNS service that
  translates domain names to numeric IP addresses
- Supports multiple routing policies for different traffic management needs
- Multi-Region deployment improves global application performance
- Route 53 failover improves application availability through health-checked
  DNS routing

---

## 🚀 Part 6 — Amazon CloudFront

### The Problem CloudFront Solves

When a user requests content from a website, the request travels through
multiple networks to reach the **origin server** (where the original content
is stored). The number of network hops and geographic distance significantly
affect **performance and latency** — and latency varies by location.

---

### What is a CDN?

A **Content Delivery Network (CDN)** is a globally distributed system of
caching servers that:
- Caches copies of **static content** (HTML, CSS, JavaScript, images) from
  the origin server
- Delivers content from the **closest cache edge** to the requester
- Also delivers **dynamic content** by establishing secure connections
  closer to the requester and proxying requests back to the origin faster

---

### What is Amazon CloudFront?

**Amazon CloudFront** is AWS's fast CDN service — it securely delivers data,
videos, applications, and APIs globally with low latency and high transfer
speeds. It uses a pay-as-you-go model with no minimum commitments.

**How content delivery works:**
```
User requests content
        ↓
Route 53 → nearest CloudFront edge location
        ↓
Cache hit? → serve immediately from edge
Cache miss? → check Regional edge cache
        ↓
Regional cache hit? → serve from regional cache
Cache miss? → fetch from origin server → cache it
```

**Two cache tiers:**
- **Edge locations** — globally distributed, serve popular content with
  lowest latency; less popular content eventually evicted
- **Regional edge caches** — between origin and edge locations; larger cache;
  content stays longer; reduces origin server load for moderately popular content

---

### CloudFront Benefits

| Benefit | Detail |
|---------|--------|
| **Fast and global** | Massively scaled; global network of edge locations and regional caches |
| **Security at the edge** | Network-level and application-level protection; AWS Shield Standard included at no cost; ACM for custom SSL at no extra cost |
| **Highly programmable** | Lambda@Edge runs custom code at AWS edge locations worldwide; CI/CD integration |
| **Deeply integrated with AWS** | Direct physical connections to AWS Global Infrastructure; configurable via API or AWS Management Console |
| **Cost-effective** | No minimum commitments; pay only for what you use; collapses simultaneous viewer requests into single origin requests; no data transfer charges between S3/ELB and CloudFront |

---

### CloudFront Pricing

Charges are based on four areas:

| Area | How charged |
|------|------------|
| **Data transfer out** | Volume of data (GB) transferred from CloudFront edge locations to internet or origin; tiered pricing by geographic region |
| **HTTP(S) requests** | Number of HTTP/HTTPS requests made to CloudFront for your content |
| **Invalidation requests** | Per path listed in invalidation request; first **1,000 paths/month free**, then charged per path beyond that |
| **Dedicated IP custom SSL** | $600/month per custom SSL certificate using Dedicated IP; prorated by hour (e.g. 1 day in June = $600 × 1/30 = $20) |

> 💡 **Data transfer between AWS origins (S3, ELB) and CloudFront is free.**
> You only pay for storage and compute at the origin, plus CloudFront's
> egress to end users.

---

### Section 6 Key Takeaways

- A CDN is a globally distributed system of caching servers that accelerates
  content delivery
- Amazon CloudFront securely delivers data, videos, applications, and APIs
  over a global infrastructure with low latency and high transfer speeds
- CloudFront benefits: fast and global, security at the edge, highly
  programmable, deeply integrated with AWS, cost-effective

---

## 💡 Insights

**CIDR mastery unlocks everything else in networking:**
Every VPC, subnet, route table entry, and security group rule involves CIDR
notation. Understanding that `/16` means 65,536 addresses and `/24` means 256
(minus 5 reserved = 251 usable) makes every networking design decision
concrete rather than abstract. Plan CIDR ranges before creating any VPC —
they cannot be changed after creation.

**Stateful vs stateless is the most important security group/ACL distinction:**
Stateful (security groups) means you only need to allow the initial
direction — responses are automatically permitted. Stateless (network ACLs)
means you must explicitly allow both directions. This is why you can add
network ACLs as a defense-in-depth layer without disrupting existing security
group rules, as long as you allow both directions.

**NAT Gateway is for outbound, Internet Gateway is for both:**
Private subnet instances can access the internet via NAT Gateway (outbound
only — internet cannot initiate connections to them). Public subnet instances
use the Internet Gateway for both inbound and outbound. This is the
fundamental public/private subnet architecture pattern.

**Route 53 routing policies are not just about performance:**
Weighted routing enables A/B testing, geolocation routing handles compliance
(e.g. GDPR data residency), failover routing provides high availability, and
multivalue answer improves DNS-level resilience. The choice depends on your
business requirement, not just technical performance goals.

**CloudFront's two-tier cache (edge + regional) is a key design decision:**
Content that is very popular lands at edge locations (fastest). Content that
is moderately popular lives longer in regional edge caches (still fast,
better than going to origin). Content that is rarely requested goes to origin.
Designing your cache behavior (TTLs, invalidation strategy) determines how
well CloudFront performs for your specific content profile.

---

## ⚖️ Reflection

**What clicked:**
- The five reserved IP addresses per subnet is easy to overlook when planning
  capacity — always plan with `/24 = 251 usable`, not 256
- VPC peering's no-transitive-peering rule is the exact reason AWS Transit
  Gateway exists — it solves the O(n²) connection problem with an O(n) hub
- The public subnet vs private subnet architecture (EC2 in private +
  NAT Gateway in public + Internet Gateway on VPC) is the canonical pattern
  for almost every production deployment

**Still unclear:**
- What is the difference between a VPC endpoint **interface** and a VPC
  endpoint **gateway** in terms of how they work internally? (PrivateLink
  creates a private ENI vs gateway modifies route tables)
- How do **Lambda@Edge** functions interact with CloudFront? What can they
  do at the edge vs what must go to origin?

**Next steps:**
- Module 6: Compute — EC2 is deployed inside VPCs configured here
- Explore the VPC Wizard in the AWS console to build a VPC with public and
  private subnets
- Review [AWS VPC Connectivity Options whitepaper](https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/introduction.html) for advanced connectivity patterns

---

## 📚 Key Terms

| Term | Definition |
|------|-----------|
| **VPC** | Virtual Private Cloud — logically isolated section of AWS Cloud |
| **Subnet** | Range of IP addresses within a VPC; belongs to one AZ |
| **CIDR** | Classless Inter-Domain Routing — notation for expressing IP address ranges |
| **Route table** | Set of rules (routes) that direct subnet network traffic |
| **Internet Gateway** | Enables communication between VPC instances and the internet |
| **NAT Gateway** | Allows private subnet instances to reach the internet outbound; blocks inbound initiation |
| **Elastic IP** | Static public IPv4 address; can be remapped between instances |
| **ENI** | Elastic Network Interface — virtual network interface attachable to EC2 instances |
| **VPC Peering** | Private networking connection between two VPCs |
| **VPC Sharing** | Allows multiple AWS accounts to share centrally managed VPCs |
| **VPC Endpoint** | Private connection from VPC to AWS services without internet |
| **Transit Gateway** | Hub-and-spoke networking device for connecting many VPCs at scale |
| **Security Group** | Instance-level virtual firewall; stateful; allow rules only |
| **Network ACL** | Subnet-level optional firewall; stateless; allow and deny rules |
| **Route 53** | AWS cloud DNS service — translates domain names to IP addresses |
| **Routing policy** | Rules determining how Route 53 responds to DNS queries |
| **DNS failover** | Active-passive failover via Route 53 health checks |
| **CDN** | Content Delivery Network — globally distributed caching servers |
| **CloudFront** | AWS CDN service — delivers content via edge locations |
| **Edge location** | CloudFront node that caches and serves content close to users |
| **Regional edge cache** | Larger CloudFront cache between origin and edge locations |
| **Origin server** | Server storing the original, authoritative version of content |
| **Lambda@Edge** | Run custom code at CloudFront edge locations globally |

---

## 📚 References

- [Amazon VPC Overview](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [VPC Route Tables](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)
- [Internet Gateways](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)
- [NAT Gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
- [NAT Gateway vs NAT Instance comparison](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-comparison.html)
- [VPC Peering](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-peering.html)
- [VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/concepts.html)
- [AWS VPC Connectivity Options whitepaper](https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/introduction.html)
- [One to Many: Evolving VPC Design](https://aws.amazon.com/blogs/architecture/one-to-many-evolving-vpc-design/)
- [Amazon Route 53 DNS](https://aws.amazon.com/route53/what-is-dns/)
- [Amazon CloudFront Overview](https://aws.amazon.com/cloudfront/)
- [How CloudFront Delivers Content](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/HowCloudFrontWorks.html)
- [Amazon CloudFront Pricing](https://aws.amazon.com/cloudfront/pricing/)
- [Elastic IP Addresses](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-eips.html)
- [Elastic Network Interfaces](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html)
