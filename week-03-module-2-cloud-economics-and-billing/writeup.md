# Module 2 — Cloud Economics and Billing 💰

> AWS Academy Cloud Foundations | Module 2

---

## 📌 Overview

This module covers the economic principles and billing mechanisms that govern
how organizations are charged for cloud resources and how they can manage and
optimize cloud spend. Understanding pricing before building anything on AWS
prevents expensive surprises.

**Topics covered:**
- AWS pricing fundamentals (three factors, pricing models)
- AWS Free Tier
- Total Cost of Ownership (TCO)
- AWS Organizations (consolidated billing, OUs, SCPs)
- AWS billing tools (Cost Explorer, Budgets, Cost and Usage Report)
- AWS Pricing Calculator
- Technical support tiers

---

## 🧠 Core Concepts

### The Key Mental Model Shift

```
Traditional IT: CapEx (buy hardware up front, depreciate over years)
Cloud:          OpEx (pay for usage over time, scale with demand)
```

This shift has real consequences:
- Finance teams need new approval models — cloud spend is ongoing, not a
  one-time purchase
- Engineering teams become cost owners — architectural decisions directly
  affect the monthly bill
- Governance requires tagging, budgets, and alerts — without these, spend
  is invisible until the invoice arrives

---

### AWS Pricing Fundamentals

#### Three Core Factors

AWS pricing is built on three dimensions:

| Factor | What you pay for |
|--------|-----------------|
| **Compute** | Time resources are running (per second or per hour depending on service) |
| **Storage** | Amount of data stored (per GB/month) |
| **Data transfer** | Data transferred OUT of AWS to the internet (inbound is generally free) |

> 💡 **Data transfer out** is the one that catches people off guard. Moving
> data between AWS services in the same Region is often free or very cheap,
> but egress to the internet adds up quickly at scale.

#### Four Pricing Models

| Model | Description | Best for |
|-------|-------------|---------|
| **On-demand** | Pay per use, no commitment | Unpredictable or short-lived workloads |
| **Reserved Instances / Committed use** | 1 or 3 year commitment → significant discount (up to ~75%) | Stable, long-running workloads |
| **Spot Instances** | Bid on unused capacity → deep discount (up to ~90%) | Fault-tolerant, interruptible workloads (batch jobs, data processing) |
| **Savings Plans** | Commit to a $ amount/hour of usage → flexible discount across instance families | Mixed or evolving workload types |

> 💡 **Spot vs Reserved:** Spot is cheaper but can be interrupted with 2
> minutes notice. Reserved is a billing commitment, not a capacity reservation
> (though you can also purchase Capacity Reservations separately). Know the
> difference before architecting around either.

#### Key Pricing Principle: Pay Less as You Use More

AWS uses tiered pricing for many services — the more you use, the lower the
per-unit cost. This is how economies of scale are passed to customers.

---

### AWS Free Tier

Three types of Free Tier offers:

| Type | Description | Example |
|------|-------------|---------|
| **Always Free** | Never expires, available to all customers | AWS Lambda: 1M requests/month free forever |
| **12 Months Free** | Available for 12 months from account creation date | EC2: 750 hrs/month of t2.micro or t3.micro |
| **Trials** | Short-term trial for specific services | Amazon SageMaker: 2-month free trial |

> ⚠️ Free Tier limits are per account, not per Region. If you run resources
> in multiple Regions simultaneously, usage aggregates toward the limit.
> Always set a billing alert on new accounts — accidentally exceeding Free
> Tier is a common beginner mistake.

---

### Total Cost of Ownership (TCO)

**Purpose:** Compare the *full lifecycle cost* of on-premises vs cloud —
not just hardware, but everything it takes to run that hardware.

#### What TCO includes

| Category | On-premises costs | Cloud equivalent |
|----------|------------------|-----------------|
| **Compute** | Server purchase, maintenance, refresh cycle | EC2 instance hours |
| **Storage** | SAN/NAS hardware, drives, maintenance | S3, EBS monthly fees |
| **Networking** | Switches, routers, cables, internet circuits | VPC, data transfer fees |
| **Facilities** | Data center space, power, cooling | Included in AWS pricing |
| **Staff** | IT operations, sysadmin, procurement, security | Reduced — AWS manages undifferentiated work |
| **Software licenses** | OS, middleware, database licenses | Included in managed service pricing (RDS, etc.) |
| **Opportunity cost** | Weeks to procure hardware → delayed launches | Resources in minutes |

> 💡 TCO comparisons often favor cloud more than expected because on-premises
> cost calculations frequently omit power, cooling, real estate, and the labor
> cost of procurement and maintenance. Full TCO surfaces these hidden costs.

#### AWS TCO Calculator / Pricing Calculator

```
AWS Pricing Calculator: https://calculator.aws/
```

Use it to:
- Estimate monthly costs for specific service configurations
- Model on-premises vs cloud cost comparisons
- Generate shareable cost estimates for stakeholders

---

## 🏦 AWS Organizations

### What It Is

AWS Organizations is a service for managing multiple AWS accounts centrally.
Essential for any organization running more than one AWS account (which is
almost everyone at production scale).

### Key Constructs

```
Root
└── Management Account (payer account — receives consolidated bill)
    ├── Organizational Unit (OU): Production
    │   ├── AWS Account: prod-app
    │   └── AWS Account: prod-data
    └── Organizational Unit (OU): Development
        ├── AWS Account: dev-team-a
        └── AWS Account: dev-team-b
```

| Construct | Description |
|-----------|-------------|
| **Root** | Top-level container for all accounts in the organization |
| **Management account** | The payer account — owns the organization and receives the consolidated bill |
| **Member accounts** | Individual AWS accounts belonging to the organization |
| **Organizational Unit (OU)** | Logical grouping of accounts (by team, environment, business unit) |
| **Service Control Policy (SCP)** | Policy applied at OU or account level that limits what actions can be taken — even by admins |

### Service Control Policies (SCPs)

SCPs are guardrails, not permissions. They define the **maximum permissions**
available in an account — even if an IAM admin grants full access, an SCP
can prevent specific actions.

Example use cases:
- Prevent member accounts from leaving the organization
- Restrict which AWS Regions accounts can use (data residency)
- Block deletion of CloudTrail logs
- Deny creation of expensive instance types in dev accounts

> ⚠️ SCPs do NOT apply to the management account itself. Apply your most
> restrictive controls to member accounts and use the management account only
> for billing and organization management.

### Consolidated Billing Benefits

- Single invoice for all accounts
- **Volume discount aggregation** — usage across all accounts is pooled, so
  combined usage reaches discount tiers faster than individual accounts would
- **Reserved Instance and Savings Plans sharing** — unused reserved capacity
  in one account can be applied to another account's on-demand usage

---

## 🛠️ Billing and Cost Management Tools

### AWS Billing Dashboard

The central hub for billing information. Shows:
- Current month estimated charges
- Month-to-date spend by service
- Free Tier usage and remaining limits
- Links to invoices, payment methods, and cost management tools

### AWS Cost Explorer

- **Purpose:** Visualize, understand, and analyze your AWS costs and usage
  over time
- Filter and group by service, Region, account, tag, instance type
- View historical trends and forecast future spend
- Identify the biggest cost drivers at a glance

### AWS Budgets

- **Purpose:** Set custom cost or usage thresholds and receive alerts when
  they are exceeded or forecasted to be exceeded
- Types: Cost budget, Usage budget, Reservation budget, Savings Plans budget
- Alerts via email or SNS (can trigger automated actions)

> 💡 **Cost Explorer vs Budgets:**
> Cost Explorer is for *analysis* — looking back at what happened.
> Budgets is for *control* — alerting you before or when limits are hit.
> Use both together.

### AWS Cost and Usage Report (CUR)

- The most detailed billing data AWS provides
- Line-item level — every individual resource charge
- Delivered to S3, queryable with Athena or loaded into a data warehouse
- Used by teams that need precise chargeback/showback or custom dashboards

### Cost Allocation Tags

Tags applied to AWS resources that appear in billing data, enabling you to
filter costs by team, project, environment, or any custom dimension.

```
Example tags:
  Environment: production
  Team: backend
  Project: payments-service
  CostCenter: 1042
```

> ⚠️ Tags must be **activated for cost allocation** in the Billing console
> before they appear in billing reports. Creating a tag on a resource is not
> enough — you must also activate it.

---

## 📞 Technical Support Tiers

| Tier | Who it's for | Response time (critical) | Key features |
|------|-------------|--------------------------|--------------|
| **Basic** | All accounts (free) | None — community forums only | Documentation, whitepapers, AWS Health Dashboard |
| **Developer** | Testing and development | < 12 business hours | Email support during business hours, general guidance |
| **Business** | Production workloads | < 1 hour | 24/7 phone/chat, full Trusted Advisor checks, third-party software support |
| **Enterprise On-Ramp** | Growing production workloads | < 30 minutes | Pool of Technical Account Managers, concierge support |
| **Enterprise** | Mission-critical workloads | < 15 minutes | Dedicated Technical Account Manager (TAM), Infrastructure Event Management |

> 💡 **Trusted Advisor** (full version requires Business tier or above) scans
> your account for cost optimization, security, fault tolerance, performance,
> and service limits recommendations. It is one of the most useful tools for
> identifying low-hanging fruit improvements.

---

## 💡 Insights

**Billing governance is architectural, not just financial:**  
The decision to use a managed service (RDS vs self-managed MySQL on EC2) has
billing implications, operational implications, and security implications
simultaneously. You cannot separate cost optimization from architecture.

**Tagging strategy should be defined before resources are created:**  
Retrofitting tags onto hundreds of existing resources is painful. Establishing
a tagging standard (what tags are required, what values are allowed) at the
start of a project costs almost nothing and makes cost allocation possible
later.

**Free Tier is a trap if you're not watching:**  
Most Free Tier limits reset monthly, but some services charge immediately once
the limit is hit. Setting a billing alert at $1 and $10 on any new account is
a 2-minute task that prevents unpleasant surprises.

**Reserved Instances ≠ guaranteed capacity:**  
A Reserved Instance is a billing commitment. You get the discounted rate
whether or not you use the capacity. Unused reservations still cost money.
Right-sizing before committing to reservations is essential.

---

## ⚖️ Reflection

**What clicked:**
- The three pricing factors (compute, storage, data transfer) give a mental
  model for estimating any AWS bill before building — think in these three
  dimensions first
- AWS Organizations + SCPs is the correct answer to "how do enterprises
  maintain control across dozens of accounts" — this makes multi-account
  architecture feel tractable rather than chaotic
- The distinction between Cost Explorer (analysis) and Budgets (control)
  clarifies which tool to reach for in which situation

**Still unclear:**
- How do Savings Plans interact with Reserved Instances if you have both?
  Is there a priority order for how discounts are applied?
- What does a real tagging governance policy look like at a mid-size
  organization? (Something to research before working on a real AWS project)

**Next steps:**
- Explore AWS Pricing Calculator with a realistic workload estimate
- Set up a billing alert on any sandbox account before running experiments
- Module 3: AWS Global Infrastructure — understand Regions and AZs before
  going deeper into any service

---

## 📚 Key Terms

| Term | Definition |
|------|-----------|
| **CapEx** | Capital expenditure — large upfront investment (traditional IT hardware) |
| **OpEx** | Operational expenditure — ongoing usage-based costs (cloud model) |
| **On-demand** | Pay per use, no commitment |
| **Reserved Instance** | 1 or 3 year billing commitment in exchange for discounted rate |
| **Spot Instance** | Discounted, interruptible capacity using unused AWS capacity |
| **Savings Plan** | Flexible discount for committed hourly spend across instance families |
| **TCO** | Total Cost of Ownership — full lifecycle cost comparison including hidden costs |
| **AWS Organizations** | Service for centralized management of multiple AWS accounts |
| **Management account** | The payer account that owns the AWS Organization |
| **OU** | Organizational Unit — logical grouping of accounts within an organization |
| **SCP** | Service Control Policy — guardrail limiting maximum permissions in an account |
| **Cost Explorer** | Tool for visualizing and analyzing historical AWS costs and usage |
| **AWS Budgets** | Tool for setting cost/usage thresholds and receiving alerts |
| **CUR** | Cost and Usage Report — most granular billing data, delivered to S3 |
| **Cost allocation tag** | Resource tag activated for billing to attribute costs to teams/projects |
| **Trusted Advisor** | AWS tool that scans accounts for cost, security, and performance recommendations |
| **TAM** | Technical Account Manager — dedicated AWS support contact (Enterprise tier) |

---

## 📚 References

- [AWS Pricing Overview](https://aws.amazon.com/pricing/)
- [AWS Free Tier](https://aws.amazon.com/free/)
- [AWS Pricing Calculator](https://calculator.aws/)
- [AWS Organizations](https://aws.amazon.com/organizations/)
- [AWS Cost Explorer](https://aws.amazon.com/aws-cost-management/aws-cost-explorer/)
- [AWS Budgets](https://aws.amazon.com/aws-cost-management/aws-budgets/)
- [AWS Support Plans](https://aws.amazon.com/premiumsupport/plans/)
- [AWS Trusted Advisor](https://aws.amazon.com/premiumsupport/technology/trusted-advisor/)
