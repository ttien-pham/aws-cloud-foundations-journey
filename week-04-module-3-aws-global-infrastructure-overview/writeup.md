# Module 3 — AWS Global Infrastructure Overview 🌐

> AWS Academy Cloud Foundations | Module 3

---

## 📌 Overview

This module covers the physical and logical structure of AWS's global
infrastructure — how AWS organizes its data centers into Regions,
Availability Zones, and Points of Presence, and why that structure matters
for building reliable, low-latency applications. The second half introduces
the full AWS service catalog organized by category.

**Topics covered:**
- AWS Regions — what they are and how to choose one
- Availability Zones — isolation, redundancy, and fault tolerance
- Data centers — how AWS designs and secures them
- Points of Presence — edge locations and regional edge caches
- AWS infrastructure features (elasticity, fault tolerance, high availability)
- AWS foundational service categories and key services within each

---

## 🧠 Part 1 — AWS Global Infrastructure

### The Big Picture

The AWS Global Infrastructure is designed and built to deliver a **flexible,
reliable, scalable, and secure** cloud computing environment with high-quality
global network performance.

The three levels of the physical infrastructure, from largest to smallest:

```
Region
└── Availability Zone (AZ)
    └── Data Center(s)
```

And for content delivery:

```
Points of Presence
├── Edge Locations        ← many, distributed globally
└── Regional Edge Caches  ← fewer, absorb infrequently accessed content
```

---

### AWS Regions

An **AWS Region** is a physical geographical location that contains one or
more Availability Zones.

**Key facts:**
- AWS has **22 Regions** worldwide (and continues to expand)
- Regions are **isolated from one another** — resources in one Region are
  not automatically replicated to other Regions
- Data stored in a Region **stays in that Region** unless you explicitly
  replicate it elsewhere — this is your responsibility, not AWS's
- Regions introduced **before March 20, 2019** are enabled by default
- Regions introduced **after March 20, 2019** (e.g. Asia Pacific Hong Kong,
  Middle East Bahrain) are **disabled by default** and must be manually enabled
  via the AWS Management Console

**Special Regions:**
- **AWS China** (Beijing, Ningxia) — accessible only via a separate Amazon
  AWS China account
- **AWS GovCloud (US)** — isolated Region designed for US government agencies
  and customers with specific regulatory and compliance requirements

---

### How to Choose a Region

Four factors to consider when selecting a Region:

| Factor | Explanation |
|--------|-------------|
| **Data governance & legal requirements** | Local laws may require data to stay within geographical boundaries (e.g. EU Data Protection Directive). This is the most constraining factor — check compliance first. |
| **Latency** | Run applications and store data as close as possible to your users. Use [CloudPing](http://www.cloudping.info/) to test latency from your location to all AWS Regions. |
| **Service availability** | Not all AWS services are available in all Regions. Check [regional service availability](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/) before designing your architecture. |
| **Cost** | Service pricing varies by Region. Example: an On-Demand t3.medium EC2 Linux instance costs $0.0416/hr in US East (Ohio) vs $0.0544/hr in Asia Pacific (Tokyo). |

> 💡 **Decision order:** Compliance first → latency second → service
> availability third → cost last. If your data legally cannot leave a country,
> the other three factors are irrelevant.

---

### Availability Zones (AZs)

Each AWS Region contains multiple **Availability Zones** — isolated locations
within the Region designed to enable fault-tolerant, highly available
architectures.

**Key facts:**
- Each AZ can include **multiple data centers** (typically three)
- At full scale, a single AZ can contain **hundreds of thousands of servers**
- AZs are **fully isolated partitions** — they have their own power
  infrastructure and are physically separated by many kilometers from other AZs
- All AZs within a Region are within **100 km of each other**
- AZs are interconnected with **high-bandwidth, low-latency networking** over
  fully redundant, dedicated fiber — this enables synchronous replication
  between AZs

**Why AZs matter for architecture:**
- Partitioning your application across multiple AZs protects against localized
  failures — lightning, tornadoes, earthquakes, power outages
- AWS **recommends replicating across AZs** for resiliency
- You are responsible for selecting which AZs your systems reside in
- **Design your systems to survive the temporary or prolonged failure of an
  entire AZ**

```
Region: eu-west-2 (London)
├── AZ: eu-west-2a  (data center A, B, C)
├── AZ: eu-west-2b  (data center D, E, F)
└── AZ: eu-west-2c  (data center G, H, I)
     ↑ physically separated, own power, connected by dedicated fiber
```

---

### Data Centers

Data centers are the foundation — the physical locations where your data
actually resides. Customers never specify a data center directly; the most
granular choice you make is the **Availability Zone**.

**How AWS designs data centers:**

| Design principle | What it means |
|-----------------|---------------|
| **Location evaluation** | Each site is carefully selected to mitigate environmental risk |
| **Redundant design** | Built to anticipate and tolerate failure while maintaining service levels |
| **Critical system backups** | Key components are backed up across multiple AZs |
| **Continuous capacity monitoring** | AWS monitors usage to deploy infrastructure ahead of demand |
| **Undisclosed locations** | Data center locations are not published; access is strictly restricted |
| **Automated failover** | In case of failure, automated processes move traffic away from the affected area |
| **Custom network equipment** | AWS uses custom hardware from multiple ODMs (Original Device Manufacturers) |

> 💡 **ODM** = a company that designs and manufactures a product based on
> another company's specifications, which then rebrands it. AWS does this
> to avoid single-vendor dependency in its hardware supply chain.

---

### Points of Presence — Edge Locations and Regional Edge Caches

**Points of Presence (PoPs)** are a global network of locations separate from
Regions and AZs — used specifically for content delivery and DNS resolution.

```
Points of Presence
├── Edge Locations          ← many locations, globally distributed
│                             used for frequently accessed content
└── Regional Edge Caches    ← fewer locations, larger cache
                              used for infrequently accessed content
                              sit between origin and edge locations
```

**Services that use Points of Presence:**
- **Amazon CloudFront** — CDN that delivers data, videos, APIs globally with
  low latency and high transfer speeds. Requests routed automatically to
  the nearest edge location.
- **Amazon Route 53** — Scalable DNS service. Translates domain names
  (www.example.com) to IP addresses (192.0.2.1). Routed to nearest edge
  location automatically.
- **AWS Shield** — Managed DDoS protection
- **AWS WAF** (Web Application Firewall) — filters malicious web traffic

**How Regional Edge Caches work:**

```
User request → Edge Location (cache hit? serve. miss? →)
             → Regional Edge Cache (cache hit? serve. miss? →)
             → Origin Server (fetch content, store in caches)
```

> 💡 Regional edge caches prevent content that is *not* accessed frequently
> enough to stay in an edge location from having to be fetched directly from
> the origin server every time — reducing both latency and origin load.

---

### AWS Infrastructure Features

| Feature | Description |
|---------|-------------|
| **Elasticity** | Resources dynamically adapt to increases or decreases in capacity requirements |
| **Scalability** | Infrastructure rapidly adjusts to accommodate growth |
| **Fault tolerance** | Built-in component redundancy; continues operating despite component failures |
| **High availability** | High operational performance with minimized downtime and no required human intervention |

---

### Key Takeaways — Part 1

- AWS Global Infrastructure = Regions → Availability Zones → Data Centers
- Region choice is driven by: compliance → latency → service availability → cost
- Each AZ is physically isolated with its own power, but connected by
  dedicated low-latency fiber within the Region
- Edge locations and Regional edge caches improve performance by caching
  content closer to users — used by CloudFront, Route 53, Shield, WAF
- Data stays in the Region you choose unless you explicitly replicate it

---

## 🏗️ Part 2 — AWS Service and Service Category Overview

### The Service Stack

AWS organizes its offerings into layers, from infrastructure up to applications:

```
Applications       Virtual desktops | Collaboration and sharing
                   ─────────────────────────────────────────────
Platform           Databases | Analytics | App Services |
Services           Deployment & Management | Mobile Services
                   ─────────────────────────────────────────────
Foundation    ▶    Compute | Networking | Storage          ◀ (highlighted)
Services           ─────────────────────────────────────────────
Infrastructure     Regions | Availability Zones | Edge Locations
```

There are **23 product/service categories** in total. This course focuses on
the most widely used and exam-relevant categories.

---

### Storage Services

| Service | Type | What it does |
|---------|------|-------------|
| **Amazon S3** | Object storage | Stores any amount of data — websites, backups, big data, IoT. Scalable, durable, secure. |
| **Amazon EBS** | Block storage | High-performance block storage for EC2 — databases, enterprise apps, media workflows |
| **Amazon EFS** | File storage | Scalable, fully managed NFS file system. Grows and shrinks automatically. Scales to petabytes. |
| **Amazon S3 Glacier** | Archive storage | Extremely low-cost archival storage. 11 nines of durability. For data rarely accessed but must be retained long-term. |

> 💡 **Storage type decision:**
> Object (S3) = files, backups, static websites — access via URL  
> Block (EBS) = OS drives, databases — attached to a single EC2 instance  
> File (EFS) = shared filesystem — multiple EC2 instances access simultaneously  
> Archive (Glacier) = long-term cold storage — retrieval takes minutes to hours

---

### Compute Services

| Service | What it does |
|---------|-------------|
| **Amazon EC2** | Resizable virtual machines in the cloud — full control over OS and instance |
| **Amazon EC2 Auto Scaling** | Automatically adds or removes EC2 instances based on conditions you define |
| **Amazon ECS** | Highly scalable container orchestration service — supports Docker |
| **Amazon ECR** | Fully managed Docker container registry — store, manage, and deploy container images |
| **AWS Elastic Beanstalk** | Deploy and scale web applications on familiar servers (Apache, IIS) — AWS manages the infrastructure |
| **AWS Lambda** | Run code without provisioning or managing servers — pay only for compute time consumed |
| **Amazon EKS** | Deploy, manage, and scale containerized applications using Kubernetes on AWS |
| **AWS Fargate** | Compute engine for ECS — run containers without managing servers or clusters |

> 💡 **Compute decision quick guide:**
> Full VM control → **EC2**  
> Auto-scale EC2 fleet → **EC2 Auto Scaling**  
> Run code, no servers → **Lambda**  
> Deploy web app, managed infra → **Elastic Beanstalk**  
> Docker containers → **ECS + ECR**  
> Kubernetes → **EKS**  
> Serverless containers → **Fargate**

---

### Database Services

| Service | Type | What it does |
|---------|------|-------------|
| **Amazon RDS** | Relational | Managed relational database — automates hardware provisioning, setup, patching, backups. Resizable capacity. |
| **Amazon Aurora** | Relational | MySQL and PostgreSQL-compatible. Up to 5× faster than MySQL, 3× faster than PostgreSQL. |
| **Amazon Redshift** | Data warehouse | Analytic queries against petabytes locally and exabytes in S3. Fast at any scale. |
| **Amazon DynamoDB** | Key-value / document | Single-digit millisecond performance at any scale. Built-in security, backup, in-memory caching. |

---

### Networking and Content Delivery Services

| Service | What it does |
|---------|-------------|
| **Amazon VPC** | Provision logically isolated sections of the AWS Cloud — your own private network |
| **Elastic Load Balancing** | Automatically distributes incoming traffic across EC2 instances, containers, IP addresses, Lambda functions |
| **Amazon CloudFront** | Fast CDN — delivers data, videos, APIs globally with low latency and high transfer speeds |
| **AWS Transit Gateway** | Connects multiple VPCs and on-premises networks to a single gateway |
| **Amazon Route 53** | Scalable DNS — translates domain names to IP addresses, routes users to applications reliably |
| **AWS Direct Connect** | Dedicated private network connection from your data center/office to AWS — reduces cost, increases bandwidth |
| **AWS VPN** | Secure private tunnel from your network or device to the AWS global network |

---

### Security, Identity, and Compliance Services

| Service | What it does |
|---------|-------------|
| **AWS IAM** | Manage access to AWS services and resources — create users, groups, permissions |
| **AWS Organizations** | Restrict what services and actions are allowed across accounts (SCPs) |
| **Amazon Cognito** | Add user sign-up, sign-in, and access control to web and mobile apps |
| **AWS Artifact** | On-demand access to AWS security and compliance reports and agreements |
| **AWS KMS** | Create and manage encryption keys — controls encryption across AWS services |
| **AWS Shield** | Managed DDoS protection for applications running on AWS |

---

### Cost Management Services

| Service | What it does |
|---------|-------------|
| **AWS Cost and Usage Report** | Most comprehensive set of cost and usage data — includes metadata about services, pricing, reservations |
| **AWS Budgets** | Set custom budgets that alert you when costs or usage exceed (or are forecast to exceed) your limit |
| **AWS Cost Explorer** | Visualize, understand, and manage costs and usage over time — easy-to-use interface |

---

### Management and Governance Services

| Service | What it does |
|---------|-------------|
| **AWS Management Console** | Web-based UI for accessing your AWS account |
| **AWS Config** | Track resource inventory and configuration changes over time |
| **Amazon CloudWatch** | Monitor resources and applications — metrics, logs, alarms |
| **AWS Auto Scaling** | Scale multiple resource types to meet demand |
| **AWS CLI** | Unified command-line tool to manage AWS services — scriptable and automatable |
| **AWS Trusted Advisor** | Scans your account for cost optimization, security, fault tolerance, and performance recommendations |
| **AWS Well-Architected Tool** | Review and improve workloads against AWS best practices |
| **AWS CloudTrail** | Tracks user activity and API usage — audit log for your AWS account |

---

## 💡 Insights

**Region isolation is a feature, not a limitation:**  
The fact that data doesn't automatically replicate between Regions is
intentional — it gives customers control over data sovereignty and compliance.
The responsibility to replicate cross-Region is yours, but so is the choice
of whether to do it at all. This is the cloud's version of explicit consent
for data movement.

**AZ design teaches you how to think about resilience:**  
The AZ model — physically separated, independently powered, but connected by
fast fiber — is the template for how to think about any resilient system.
Isolate failure domains, connect them with fast paths, design each to survive
independently. This mental model applies beyond AWS.

**Edge locations solve a different problem than Regions:**  
Regions are about where you *run* your workloads. Edge locations are about
where you *deliver your content* to end users. A startup in Singapore serving
global users deploys in one Region but distributes via 400+ edge locations.
Conflating these two concepts leads to architectural mistakes.

**The service catalog is overwhelming by design — don't memorize, categorize:**  
There are 23 service categories and hundreds of services. The exam and real
work both require you to know *which category* a problem belongs to, not
every service in it. Know the canonical service for each category (EC2 for
compute, S3 for object storage, RDS for relational DB, IAM for access
control) and the others follow logically.

---

## ⚖️ Reflection

**What clicked:**
- The Region → AZ → Data Center hierarchy finally makes the "redundancy"
  conversation concrete — you're not just told to be redundant, you're given
  an explicit geographic model to design against
- The edge location / Regional edge cache distinction makes CloudFront's
  performance claims make sense — it's a two-tier cache, not just a single
  cache layer
- AWS using custom ODM hardware is an interesting detail — it explains why
  AWS can achieve cost and performance levels that are hard to replicate
  on commodity hardware

**Still unclear:**
- What does "synchronous replication between AZs" mean for my data in
  practice? If I use RDS Multi-AZ, is my data guaranteed to be in both AZs
  before a write is acknowledged? (Yes — this is worth verifying in Module 8)
- AWS Direct Connect vs AWS VPN — when does the dedicated physical line
  justify the cost over an encrypted VPN tunnel? (Likely covered in
  Module 5: Networking)

**Next steps:**
- Module 4: AWS Cloud Security — IAM deep dive before touching any other
  service
- Explore the [AWS Global Infrastructure Map](https://aws.amazon.com/about-aws/global-infrastructure/)
  interactively to see current Region and AZ counts
- Check [CloudPing](http://www.cloudping.info/) to measure latency from
  your location to each Region

---

## 📚 Key Terms

| Term | Definition |
|------|-----------|
| **Region** | Physical geographical location containing one or more AZs |
| **Availability Zone (AZ)** | Isolated location within a Region — own power, physically separated, connected by dedicated fiber |
| **Data center** | Physical facility where servers and data reside; not directly selectable by customers |
| **Edge location** | PoP node used by CloudFront and Route 53 to serve content with low latency |
| **Regional edge cache** | Larger cache between origin and edge locations — for infrequently accessed content |
| **CloudFront** | AWS CDN — delivers content globally via edge locations |
| **Route 53** | AWS DNS service — routes users to applications by translating domain names to IPs |
| **Elasticity** | Dynamic adaptation to increases or decreases in capacity |
| **Scalability** | Ability to grow to accommodate increased demand |
| **Fault tolerance** | Continues operating correctly despite component failures |
| **High availability** | Minimized downtime, high operational performance, no manual intervention needed |
| **ODM** | Original Device Manufacturer — builds hardware to AWS specifications, which AWS rebrands |
| **AWS GovCloud** | Isolated Region for US government workloads with specific compliance requirements |

---

## 📚 References

- [AWS Global Infrastructure Map](https://aws.amazon.com/about-aws/global-infrastructure/)
- [AWS Regions and AZs](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/)
- [AWS Regional Service Availability](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/)
- [AWS Products Overview](https://aws.amazon.com/products/)
- [CloudPing — Latency Test](http://www.cloudping.info/)
- [AWS in China](https://www.amazonaws.cn/en/about-aws/china/)
