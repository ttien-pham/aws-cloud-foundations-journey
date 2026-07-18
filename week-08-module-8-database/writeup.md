# Module 8 — Database 🗄️

> AWS Academy Cloud Foundations | Module 8

---

## 📌 Overview

Data is one of the most valuable assets a business has. Choosing the right
database service directly impacts application performance, availability,
cost, and operational complexity. This module covers four core AWS database
services and when to use each one.

**Topics covered:**
- Amazon RDS — managed relational database
- Amazon DynamoDB — fast, flexible NoSQL database
- Amazon Redshift — fully managed data warehouse
- Amazon Aurora — high-availability MySQL/PostgreSQL-compatible database
- Managed vs unmanaged database solutions
- SQL vs NoSQL — when to use which
- Choosing the right tool for the right job

---

## 🧠 Foundation: Managed vs Unmanaged Services

Before diving into specific database services, it's important to understand
the key distinction that shapes every database decision in AWS:

### Unmanaged Services (e.g. running a DB on EC2)

- You provision the resources
- You manage scaling, patching, backups, HA, and failover
- More control, more operational responsibility
- Example: Installing MySQL on an EC2 instance and managing it yourself

### Managed Services (e.g. Amazon RDS)

- AWS handles the infrastructure operations
- You configure the service and focus on your application and data
- Less control over low-level details, significantly less operational burden
- Example: Amazon RDS handles OS patching, backups, and failover automatically

**Self-managed database responsibilities (what you avoid with managed services):**

| Responsibility | Self-managed (EC2) | Amazon RDS |
|---------------|-------------------|-----------|
| Server maintenance & energy | You | AWS |
| OS installation & patching | You | AWS |
| Database software patching | You | AWS |
| Automatic backups | You | AWS |
| High availability setup | You | AWS |
| Scaling resources | You | AWS |
| Power and hardware | You | AWS |

> 💡 The key tradeoff: **more control = more operational work**. If you have
> specific requirements that AWS managed services cannot meet (custom DB
> configurations, unusual tuning), running on EC2 gives you full flexibility.
> For most use cases, the managed service is the better choice.

---

## 🏛️ Part 1 — Amazon Relational Database Service (Amazon RDS)

### What is Amazon RDS?

**Amazon RDS** is a managed service that sets up, operates, and scales a
relational database in the cloud — without ongoing administration burden.

- **Cost-efficient and resizable capacity**
- **Automates** time-consuming admin tasks: OS patching, DB patching, backups, HA
- Your focus: **data and application optimization**
- AWS's focus: everything else

---

### Core Concepts

**Database instance** — the basic building block of Amazon RDS:
- An isolated database environment that can contain multiple user-created databases
- Accessed with the same tools as a standalone database
- Resources determined by the **database instance class**
- Storage type determined by **disk type**

**Supported database engines:**

| Engine |
|--------|
| MySQL |
| IBM DB2 |
| Microsoft SQL Server |
| PostgreSQL |
| MariaDB |
| Oracle |

---

### Network and VPC Isolation

RDS instances run inside **Amazon VPC**, giving you:
- Control over your virtual networking environment
- Selection of IP address ranges
- Subnet configuration
- Access control lists (ACLs)

**Typical pattern:**
- Database instance placed in a **private subnet**
- Only accessible to specific application instances
- Subnet selection also determines the **Availability Zone** of the instance

---

### High Availability — Multi-AZ Deployment

One of the most powerful RDS features: **Multi-AZ deployment**.

```
Primary DB instance (AZ 1)
         ↓ synchronous replication
Standby DB instance (AZ 2)
```

**How it works:**
- Amazon RDS automatically generates a **standby copy** in another AZ in the same VPC
- Transactions are **synchronously replicated** — minimizes data loss potential
- If the primary fails, RDS **automatically promotes the standby** to become the new primary
- Applications connect via the **RDS DNS endpoint** — no application code changes needed
  during failover

**Benefits:** Protects against DB instance failure AND Availability Zone disruption.

---

### Read Replicas

For **MySQL, MariaDB, PostgreSQL, and Amazon Aurora**, RDS supports read replicas:

```
Primary DB instance
        ↓ asynchronous replication
Read Replica(s)
```

**Use cases:**
- **Reduce load** on the primary by routing read queries to replicas
- **Scale out** beyond single-instance capacity for read-heavy workloads
- **Disaster recovery** — replicas can be created in a different Region
- **Lower read latency** — direct reads to a replica closer to the user

> ⚠️ **Asynchronous replication** means there is a small lag — replicas may
> be slightly behind the primary. Promoting a read replica to primary requires
> **manual action** (unlike Multi-AZ standby which is automatic).

| Feature | Multi-AZ Standby | Read Replica |
|---------|-----------------|-------------|
| **Replication** | Synchronous | Asynchronous |
| **Purpose** | High availability / failover | Read scaling |
| **Failover** | Automatic | Manual promotion |
| **Cross-Region** | No | Yes |
| **Traffic** | No reads served | Serves read traffic |

---

### When to Use Amazon RDS

**Use RDS when your application requires:**
- Complex transactions or complex queries
- Medium to high query/write rate — up to **30,000 IOPS** (15,000 reads + 15,000 writes)
- No more than a single worker node or shard
- High durability

**Do NOT use RDS when your application requires:**
- Massive read/write rates (e.g. 150,000 writes per second)
- Sharding due to high data size or throughput demands
- Simple GET/PUT requests that a NoSQL database handles more efficiently
- RDBMS customization beyond what RDS allows

> 💡 **When RDS doesn't fit:** Consider **DynamoDB** (NoSQL) or run your
> relational database directly on **Amazon EC2** for full customization control.

---

### Amazon RDS Pricing Factors

| Factor | Details |
|--------|---------|
| **Clock hours of service** | Charges run from instance launch to termination |
| **Database characteristics** | Engine, size, and memory class affect cost |
| **Purchase type** | On-Demand (pay per hour) vs Reserved (1 or 3 year, upfront discount) |
| **Number of instances** | Multiple instances to handle peak loads |
| **Provisioned storage** | Backup storage up to 100% of provisioned DB is free while instance is active |
| **Additional backup storage** | Billed per GB/month after instance termination |
| **I/O requests** | Number of input/output requests to the database |
| **Deployment type** | Single-AZ vs Multi-AZ affects storage and I/O charges |
| **Data transfer** | Inbound is free; outbound is tiered |

> 💡 **Multi-AZ vs Single-AZ pricing:** Multi-AZ deployments cost more for
> storage and I/O but provide significantly better availability guarantees.
> Evaluate the cost of downtime against the incremental cost of Multi-AZ.

---

### Amazon RDS Key Features Summary

- Managed service — accessible via Console, AWS CLI, or API
- Scalable compute and storage
- Automated redundancy and backup
- Supports: Aurora, PostgreSQL, MySQL, MariaDB, Oracle, Microsoft SQL Server
- Two SSD storage options: **high-performance OLTP** and **cost-effective general purpose**
- Scale compute and storage with **no downtime**
- Runs within Amazon VPC for security and control

---

## ⚡ Part 2 — Amazon DynamoDB

### Relational vs Non-Relational — The Core Difference

| Feature | Relational (SQL) | Non-relational (NoSQL) |
|---------|-----------------|----------------------|
| **Data structure** | Structured — tables, records, columns | Flexible — key-value, document, column, graph |
| **Query language** | SQL (standardized) | Service-specific query methods |
| **Horizontal scaling** | Difficult | Designed for it |
| **Data consistency** | Strong consistency by default | Varies by service |
| **Schema** | Fixed schema | Dynamic — schema can evolve per item |
| **Use case** | Complex queries, transactions | Large scale, variable structure, low latency |

---

### What is Amazon DynamoDB?

**Amazon DynamoDB** is a fast and flexible NoSQL database service providing
**consistent, single-digit-millisecond latency at any scale**.

- Fully managed — AWS manages all underlying data infrastructure
- Data redundantly stored across **multiple facilities** in a native AWS Region
- **No practical limit** on items stored in a table (some customers store billions of items)
- All data stored on **SSDs** for consistent performance
- Automatic partitioning as data grows

---

### DynamoDB Core Components

| Component | Description |
|-----------|-------------|
| **Table** | A collection of data |
| **Item** | A group of attributes uniquely identifiable among all items in the table |
| **Attribute** | A fundamental data element — does not need to be broken down further |

**Key flexibility:** Items in the same table can have **different attributes** — no schema
migration needed when adding fields. Newer and older format items coexist in the same table.

---

### Primary Key Types

| Key type | Structure | Best for |
|----------|-----------|---------|
| **Partition key** (simple) | Single attribute | Unique, uniformly distributed identifiers (e.g. GUID, product ID) |
| **Partition key + Sort key** (composite) | Two attributes combined | Data you frequently query by one attribute and sort/filter by another (e.g. Author + Title for books) |

> 💡 Choosing the right primary key is critical for DynamoDB performance.
> A well-designed partition key distributes data evenly across partitions,
> preventing "hot partition" issues that degrade throughput.

---

### Data Retrieval Methods

| Method | How it works | Efficiency |
|--------|-------------|-----------|
| **Query** | Uses the partition key to locate items directly | **Fast** — takes advantage of partitioning |
| **Scan** | Checks all items in the table for matching non-key attributes | **Slow** — reads entire table; use sparingly |

---

### DynamoDB Key Features

| Feature | Detail |
|---------|--------|
| **Throughput scaling** | Provision read/write throughput manually, or enable **automatic scaling** |
| **Global Tables** | Automatically replicate across your choice of AWS Regions |
| **Encryption at rest** | Built-in data protection |
| **Item TTL** | Automatically delete items after a specified time |
| **Consistent performance** | Predictable low latency even in large tables |

---

### DynamoDB Use Cases

DynamoDB works well for: **mobile, web, gaming, ad tech, and IoT applications**

- **High-scale throughput** — large numbers of clients making many requests per second
- **Latency-sensitive applications** — consistent, predictable sub-millisecond response times
- **Ad tech** — where variable latency could cause business impact
- **Gaming** — where performance consistency is critical to user experience
- **Global Tables** — for applications that need global availability and business continuity

---

## 📊 Part 3 — Amazon Redshift

### What is Amazon Redshift?

**Amazon Redshift** is a fast, fully managed **data warehouse** service that
makes it simple and cost-effective to analyze large volumes of data using
standard SQL and existing BI tools.

> 💡 **Database vs Data Warehouse distinction:**
> - **Databases (RDS, DynamoDB)** are optimized for **transactional workloads** — reading and writing individual records frequently (OLTP)
> - **Data warehouses (Redshift)** are optimized for **analytical workloads** — running complex queries across massive historical datasets (OLAP)

---

### Why Redshift?

Building a traditional data warehouse is **complex and expensive** — months
of setup, significant financial investment, specialized expertise.

Amazon Redshift provides:
- Complex analytic queries against **petabytes of structured data**
- **Columnar storage** on high-performance local disks
- **Massively parallel processing (MPP)** across multiple nodes
- Most results return in **seconds**
- Starting price: as low as **$0.25/hour**
- At scale: approximately **$1,000/TB/year** (3-year Reserved pricing)
- **Redshift Spectrum** — run queries against **exabytes of data directly in Amazon S3**

---

### Redshift Architecture

```
Client applications / BI tools
         ↓
   Leader Node
   (manages client communication, parses queries,
    develops execution plans, compiles code,
    distributes to compute nodes, aggregates results)
         ↓
   Compute Nodes
   (execute compiled code, process data in parallel,
    return intermediate results to leader node)
```

---

### Redshift Key Features

| Feature | Detail |
|---------|--------|
| **Scalability** | Scale up or down with a few clicks; no downtime |
| **Automated management** | Monitoring, backups, restoration handled automatically |
| **Security** | Encryption at rest and in transit; built-in |
| **SQL compatibility** | Standard SQL; JDBC and ODBC connectors |
| **BI tool integration** | Works with existing tools you already use |
| **Redshift Spectrum** | Query exabytes of data directly in S3 |

---

### Redshift Use Cases

| Use case | Why Redshift |
|----------|-------------|
| **Enterprise data warehouse migration** | Start at any scale; experiment without complex IT procurement |
| **Big data analytics** | Handle massive datasets that stretch existing systems |
| **SaaS analytics** | Provide analytic capabilities within applications; deploy per-customer clusters |
| **Self-service analytics** | Managed service reduces DBA overhead; teams focus on querying data |

---

## 🌟 Part 4 — Amazon Aurora

### What is Amazon Aurora?

**Amazon Aurora** is a MySQL- and PostgreSQL-compatible relational database
built for the cloud. It combines the **performance and availability of
high-end commercial databases** with the **simplicity and cost-effectiveness
of open-source databases**.

- Fully managed by Amazon RDS (inherits all RDS management features)
- **Drop-in compatible** with MySQL and PostgreSQL — minimal or no application changes needed
- Pay-as-you-go pricing
- Integrates with **AWS DMS** and **AWS Schema Conversion Tool** for migrations

---

### Aurora vs Standard RDS — Why Choose Aurora?

The primary reason to choose Aurora over standard RDS MySQL/PostgreSQL is
**superior high availability and resilient design**.

| Feature | Standard RDS | Amazon Aurora |
|---------|-------------|--------------|
| **Data copies** | Single instance + optional standby | **6 copies across 3 AZs** (always) |
| **Continuous backup** | Configurable | **Automatic to S3** |
| **Read replicas** | Up to 5 (MySQL) | **Up to 15** |
| **Crash recovery time** | Minutes | **Under 60 seconds** (most cases) |
| **Buffer cache** | In database process (lost on crash) | **Moved out of DB process** — available immediately at restart |
| **Storage** | Standard EBS | Distributed, fault-tolerant, self-healing |

---

### Aurora High Availability Architecture

```
Aurora Cluster (spans 3 Availability Zones)
│
├── AZ 1: Primary instance + 2 data copies
├── AZ 2: Read replica + 2 data copies
└── AZ 3: Read replica + 2 data copies
                ↓
    Continuous backup → Amazon S3
```

**Key properties:**
- Stores **6 copies of data across 3 AZs** — data survives losing 2 of 6 copies without write availability loss
- **Continuous backups to S3** — automatic and incremental
- Up to **15 read replicas** — reduce data loss risk and serve read traffic
- **Instant crash recovery** — no redo log replay from last checkpoint; performed on every read

---

### Aurora Crash Recovery — Why It's Faster

**Standard relational databases after a crash:**
```
Crash → Replay redo log from last checkpoint → Recovery → Available
(This can take many minutes)
```

**Aurora after a crash:**
```
Crash → Buffer cache already outside DB process → Available in < 60 seconds
(No need to throttle access while cache repopulates — avoids "brownouts")
```

---

### Aurora Security

Multiple layers of security:

| Layer | Mechanism |
|-------|-----------|
| **Network isolation** | Amazon VPC |
| **Encryption at rest** | Keys managed via AWS KMS |
| **Encryption in transit** | SSL/TLS |

---

### Aurora Key Features Summary

- **Highly available** — 6 copies across 3 AZs; continuous S3 backup
- **Fault-tolerant** — self-healing distributed storage
- **Fast crash recovery** — under 60 seconds in most cases
- **Up to 15 read replicas** — reduce data loss risk; scale reads
- **MySQL and PostgreSQL compatible** — minimal migration effort
- **Fully managed by RDS** — automated provisioning, patching, backups, recovery

---

## 🔧 Choosing the Right Database — The Decision Framework

From the module's "Right Tool for the Right Job" slide:

| Requirement | Best Service |
|-------------|-------------|
| **Enterprise-class relational database** | Amazon RDS |
| **Fast and flexible NoSQL database for any scale** | Amazon DynamoDB |
| **OS access or app features not supported by AWS database services** | Databases on Amazon EC2 |
| **Specific case-driven requirements** (machine learning, data warehouse, graphs) | AWS purpose-built database services (Redshift, Neptune, SageMaker, etc.) |

### Modern Database Requirements

The cloud continues to drive down storage and compute costs. A new generation
of applications has emerged that requires databases to:
- Store **terabytes to petabytes** of new types of data
- Provide access with **millisecond latency**
- Process **millions of requests per second**
- Scale to support **millions of users anywhere in the world**

These requirements demand **both relational and non-relational databases**
that are purpose-built for specific application needs. AWS offers a broad
range of databases built for specific application use cases.

---

### Quick Comparison of All Four Services

| Service | Type | Best for | Key differentiator |
|---------|------|---------|-------------------|
| **Amazon RDS** | Relational (SQL) | Web/mobile apps, e-commerce, enterprise workloads requiring ACID transactions | Managed SQL with 6 engine options; Multi-AZ HA |
| **Amazon DynamoDB** | NoSQL (key-value/document) | Mobile, gaming, IoT, ad tech; single-digit ms latency at any scale | Serverless, auto-scaling, Global Tables |
| **Amazon Redshift** | Data warehouse | Complex analytics on large datasets; BI/reporting | Columnar storage, MPP, Redshift Spectrum for S3 |
| **Amazon Aurora** | Relational (SQL) | Applications needing MySQL/PostgreSQL with higher availability and performance | 6 copies across 3 AZs; up to 15 replicas; < 60s crash recovery |

---

## 💡 Insights

**The managed vs unmanaged decision has a hidden cost dimension:**
When comparing Amazon RDS to running a database on EC2, the hourly rate for
RDS is higher. But the total cost equation must include DBA labor — configuring
backups, building HA, patching, monitoring. For most teams, RDS is cheaper
when total cost is calculated honestly.

**NoSQL is not "better than SQL" — it solves a different problem:**
DynamoDB's single-digit millisecond latency at any scale is genuinely
remarkable, but it requires schema design upfront (primary key choice is
irreversible without migration). SQL databases make schema changes easy but
horizontal scaling hard. Match the tool to the actual workload characteristics.

**Aurora's crash recovery is architecturally unique:**
The buffer cache living outside the database process is not a small detail —
it means an Aurora instance that crashes restarts in under 60 seconds without
the brownout period that plagues traditional databases during cache warm-up.
For customer-facing applications, this can be the difference between an
imperceptible blip and a visible outage.

**Redshift Spectrum changes the economics of analytics:**
Storing and querying exabytes of data in S3 at Redshift query speeds, without
loading it into Redshift cluster storage first, eliminates the traditional
"ETL into data warehouse" bottleneck. Query the data where it lives.

---

## ⚖️ Reflection

**What clicked:**
- The Multi-AZ standby (synchronous, automatic failover) vs read replica
  (asynchronous, manual promotion) distinction finally makes the HA vs
  performance scaling decision concrete and clear
- Aurora's design of storing 6 copies across 3 AZs always — not optionally —
  explains why it can survive losing 2 copies without impact. This is
  fundamentally different from "turn on Multi-AZ and get 2 copies"
- DynamoDB Global Tables makes multi-region active-active architectures
  accessible without custom replication logic — a significant engineering
  complexity reduction

**Still unclear:**
- When would you choose **DynamoDB with provisioned capacity** vs **DynamoDB
  On-Demand capacity**? What are the cost tradeoffs for predictable vs
  unpredictable workloads?
- How does **Aurora Serverless** differ from standard Aurora? This seems
  relevant for variable/intermittent workloads.
- What is the right strategy for **migrating an existing on-premises MySQL
  database to Aurora** using AWS DMS? What are the common pitfalls?

**Next steps:**
- Module 9: Cloud Architecture — understand how these database services fit
  into Well-Architected design patterns
- Explore the RDS hands-on lab — launch, configure, and interact with an
  RDS MySQL instance
- Review the DBA activity: compare EC2-hosted vs RDS-hosted database deployments

---

## 📚 Key Terms

| Term | Definition |
|------|-----------|
| **RDS** | Relational Database Service — managed relational database in the cloud |
| **Database instance** | Isolated database environment in RDS; can contain multiple databases |
| **Multi-AZ deployment** | Synchronous standby in another AZ; automatic failover |
| **Read replica** | Asynchronously replicated copy of primary DB for read scaling |
| **DynamoDB** | Fast, flexible NoSQL database; single-digit ms latency at any scale |
| **Table** | DynamoDB collection of data |
| **Item** | DynamoDB uniquely identifiable group of attributes (equivalent to a row) |
| **Attribute** | DynamoDB fundamental data element (equivalent to a column) |
| **Partition key** | Simple primary key in DynamoDB — one attribute |
| **Composite key** | DynamoDB partition key + sort key together |
| **Query** | DynamoDB operation using partition key — fast |
| **Scan** | DynamoDB operation checking all items — slow |
| **Global Tables** | DynamoDB multi-Region automatic replication |
| **TTL** | DynamoDB Time-to-Live — auto-delete items after a timestamp |
| **Redshift** | Fast, fully managed cloud data warehouse |
| **Leader node** | Redshift coordinator — manages queries, distributes work |
| **Compute node** | Redshift worker — executes queries in parallel |
| **Columnar storage** | Storage format optimized for analytical reads (Redshift) |
| **MPP** | Massively Parallel Processing — Redshift's distributed query execution |
| **Redshift Spectrum** | Query exabytes of data directly in S3 using Redshift |
| **Aurora** | High-availability MySQL/PostgreSQL-compatible managed database |
| **OLTP** | Online Transactional Processing — frequent reads/writes of individual records |
| **OLAP** | Online Analytical Processing — complex queries over large historical datasets |
| **AWS DMS** | AWS Database Migration Service — helps move data to Aurora or RDS |

---

## 📚 References

- [Amazon RDS Documentation](https://docs.aws.amazon.com/rds/)
- [Amazon DynamoDB Documentation](https://docs.aws.amazon.com/dynamodb/)
- [DynamoDB Core Components](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html)
- [Amazon Redshift Documentation](https://aws.amazon.com/redshift/)
- [Amazon Aurora Documentation](https://aws.amazon.com/rds/aurora/)
- [Choosing the Right AWS Database Service](https://aws.amazon.com/products/databases/)
