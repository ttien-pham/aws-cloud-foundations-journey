# Module 4 — AWS Cloud Security 🔐

> AWS Academy Cloud Foundations | Module 4

---

## 📌 Overview

Security is the **highest priority** at AWS. This module introduces the AWS
approach to security: who is responsible for what, how to manage identities
and access, how to secure a new account from day one, how to protect data,
and how to demonstrate compliance.

**Topics covered:**
- AWS Shared Responsibility Model
- AWS Identity and Access Management (IAM)
- Securing a new AWS account (4 steps)
- Securing accounts (AWS Organizations, KMS, Cognito, Shield)
- Securing data on AWS (encryption at rest and in transit, S3 access control)
- Working to ensure compliance (AWS Config, AWS Artifact)

---

## 🧠 Part 1 — AWS Shared Responsibility Model

### The Core Idea

Security is a **shared responsibility** between AWS and the customer. Neither
party handles everything alone.

```
AWS is responsible for security OF the cloud
Customer is responsible for security IN the cloud
```

This model is designed to reduce the customer's operational burden while
giving customers the control they need to deploy securely.

---

### What AWS Is Responsible For

AWS operates, manages, and controls components from the **bare metal host OS
and hypervisor layer down to the physical security** of the facilities.

| AWS Responsibility | Details |
|-------------------|---------|
| **Physical security of data centers** | Controlled, need-based access; nondescript facilities; 24/7 security guards; two-factor authentication; access logging; video surveillance; disk degaussing and destruction |
| **Hardware infrastructure** | Servers, storage devices, and other appliances |
| **Software infrastructure** | Host operating systems, service applications, virtualization software |
| **Network infrastructure** | Routers, switches, load balancers, firewalls, cabling; continuous monitoring at external boundaries; redundant infrastructure with intrusion detection |

> 💡 You cannot visit AWS data centers to verify this protection, but
> third-party auditors regularly verify AWS compliance with computer security
> standards and regulations. These reports are available via AWS Artifact.

---

### What the Customer Is Responsible For

| Customer Responsibility | Examples |
|------------------------|---------|
| **EC2 instance OS** | Patching, updates, maintenance |
| **Applications** | Passwords, role-based access |
| **Security group configuration** | Firewall rules for EC2 instances |
| **OS or host-based firewalls** | Intrusion detection/prevention systems |
| **Network configurations** | VPC settings, subnet configs |
| **Account management** | Login and permission settings for each user |
| **Data encryption** | Encrypting data at rest and in transit |
| **Content decisions** | What data to store, which services to use, in which country, who has access |

Customers maintain **complete control over their content** and are responsible
for managing it — including what is stored, how it is formatted, whether it
is encrypted, and who can access it.

---

### Responsibility Varies by Service Model

| Service Model | Customer Manages | AWS Manages |
|---------------|-----------------|-------------|
| **IaaS** (e.g. Amazon EC2) | OS, apps, security groups, network config, access controls | Hardware, virtualization, global infrastructure |
| **PaaS** (e.g. AWS Lambda, Amazon RDS) | Data, asset classification, permissions | OS, runtime, database patching, firewall config, disaster recovery |
| **SaaS** (e.g. AWS Trusted Advisor, AWS Shield, Amazon Chime) | Just use the service | Everything else |

> 💡 **The Oracle database example:** If Oracle runs on an EC2 instance →
> customer patches it. If Oracle runs as an Amazon RDS instance → AWS patches
> it. The same database engine, different responsibility based on how it's
> deployed.

---

### Activity: Scenario Answers

**Scenario 1** — S3 + EC2 + Oracle in a VPC:

| Question | Responsible |
|----------|------------|
| OS upgrades and patches on EC2 | **Customer** |
| Physical security of the data center | **AWS** |
| Virtualization infrastructure | **AWS** |
| EC2 security group settings | **Customer** |
| Configuration of applications on EC2 | **Customer** |
| Oracle patches if running as RDS | **AWS** |
| Oracle patches if running on EC2 | **Customer** |
| S3 bucket access configuration | **Customer** |

**Scenario 2** — Management Console + CLI + VPC + EC2 + S3 + SSH keys:

| Question | Responsible |
|----------|------------|
| Ensuring Management Console is not hacked | **AWS** |
| Configuring the subnet | **Customer** |
| Configuring the VPC | **Customer** |
| Protecting against network outages in AWS Regions | **AWS** |
| Securing the SSH keys | **Customer** |
| Network isolation between AWS customers' data | **AWS** |
| Low-latency connection between web server and S3 | **AWS** |
| Enforcing MFA for all user logins | **Customer** |

---

### Section 1 Key Takeaways

- AWS = security **of** the cloud (infrastructure, hardware, software, network, facilities)
- Customer = security **in** the cloud (OS, apps, data, configurations, accounts)
- IaaS → customer responsible for all security configuration
- PaaS → AWS handles more; customer manages data and permissions

---

## 🔑 Part 2 — AWS Identity and Access Management (IAM)

### What IAM Does

IAM allows you to **control who can access what, and how** in your AWS account.

- Manages access to compute, storage, database, and application services
- Handles both **authentication** (who are you?) and **authorization** (what can you do?)
- Every action in AWS — whether via Console, CLI, or SDK — is an API call.
  IAM controls which API calls each identity can make.
- **No additional cost** — IAM is a free feature of every AWS account
- **Global scope** — IAM settings apply across all AWS Regions

---

### The Four IAM Components

| Component | What it is |
|-----------|-----------|
| **IAM User** | A person or application defined in an AWS account that must make API calls. Each user has a unique name and their own credentials — not shared. |
| **IAM Group** | A collection of IAM users. Attach policies to the group and all members inherit those permissions. |
| **IAM Policy** | A JSON document that defines what actions are allowed or denied on which resources. |
| **IAM Role** | An identity with permissions policies attached, designed to be assumed temporarily by users, applications, or services — no permanent credentials. |

---

### Authentication — Two Access Types

When defining an IAM user, you choose which type(s) of access to grant:

| Access Type | How to Authenticate | Used For |
|-------------|-------------------|---------|
| **Programmatic access** | Access key ID + Secret access key | AWS CLI, AWS SDKs, API calls |
| **Console access** | 12-digit Account ID (or alias) + IAM username + Password + (MFA if enabled) | AWS Management Console |

> 💡 You can grant one type, both types, or neither. Grant only what the user
> actually needs.

**MFA (Multi-Factor Authentication)** adds a second layer — a token in addition
to regular credentials. Options for generating MFA tokens:
- **Virtual MFA apps** — Google Authenticator, Authy
- **U2F security key devices** — YubiKey
- **Hardware MFA devices** — Key fob or display card (Gemalto)

---

### Authorization — IAM Policies

By default, **IAM users have zero permissions** — all actions are implicitly
denied unless explicitly allowed.

**Policy evaluation logic:**

```
Request comes in
      ↓
Is there an EXPLICIT DENY?  → YES → DENY (always wins)
      ↓ NO
Is there an EXPLICIT ALLOW? → YES → ALLOW
      ↓ NO
IMPLICIT DENY (default)
```

> ⚠️ An explicit deny **always** takes precedence over any allow, even if
> a separate policy grants access.

**Principle of Least Privilege:** Grant only the minimum permissions a user
needs to do their job. Start minimal and add more as needed — never start
broad and try to restrict later.

---

### Two Types of IAM Policies

| Policy Type | Attached To | Defined Where |
|-------------|------------|--------------|
| **Identity-based policy** | IAM user, group, or role | Separate IAM policy document |
| **Resource-based policy** | The resource itself (e.g. S3 bucket) | Inline on the resource |

**Identity-based policies** can be:
- **Managed policies** — standalone, reusable, attachable to multiple users/groups/roles
- **Inline policies** — embedded directly into a single user, group, or role

**Policy document format (JSON):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["dynamodb:*", "s3:*"],
      "Resource": [
        "arn:aws:dynamodb:region:account-id:table/table-name",
        "arn:aws:s3:::bucket-name/*"
      ]
    },
    {
      "Effect": "Deny",
      "NotResource": [
        "arn:aws:dynamodb:region:account-id:table/table-name",
        "arn:aws:s3:::bucket-name/*"
      ],
      "Action": ["dynamodb:*", "s3:*"]
    }
  ]
}
```

> 💡 The `NotResource` + `Deny` combination is a powerful pattern — it
> ensures the user cannot use any other DynamoDB or S3 resource or action,
> even if another policy grants access.

---

### IAM Groups

A group is a collection of IAM users that share the same permissions.

**Key rules:**
- A user can belong to **multiple groups**
- Groups **cannot be nested** (no groups within groups)
- There is **no default group** — if you want all users in one group, create
  it manually and add each user

**Best practice flow:**
```
1. Create group (e.g. "Developers")
2. Attach policy to group
3. Add users to group → they automatically inherit permissions
4. New employee? Add to group. Role change? Remove from group.
```

---

### IAM Roles

An IAM role is like a user but:
- **Not permanently tied to one person** — designed to be assumed by anyone
  who needs it
- **No long-term credentials** — provides temporary security credentials
  when assumed
- Used to **delegate access** without sharing credentials

**Common use cases:**

| Use case | How roles help |
|----------|---------------|
| Application on EC2 needs S3 access | Attach a role to the EC2 instance — no hardcoded keys |
| User in Account A needs access to Account B | Create a cross-account role |
| Mobile app needs AWS resources | App assumes a role — no embedded keys |
| Corporate directory users need AWS access | Federated identity assumes a role |
| Third-party auditor needs read access | Create a role with specific permissions |

**Example IAM role flow (EC2 → S3):**
```
1. Admin creates IAM policy granting read-only access to S3 bucket "photos"
2. Admin creates IAM role and attaches the policy
3. Admin attaches the role to the EC2 instance
4. Application on EC2 assumes the role → gets temporary credentials
5. Application accesses S3 bucket "photos" using those credentials
6. No credentials are shared, stored in code, or managed by the developer
```

---

### Section 2 Key Takeaways

- **IAM policies** are JSON documents — attach to any IAM entity (user, group, role)
- **IAM user** = authenticates a person or application to AWS
- **IAM group** = simplifies managing permissions for multiple users
- **IAM role** = delegates temporary access without permanent credentials
- Entities are: IAM users, IAM groups, IAM roles

---

## 🛡️ Part 3 — Securing a New AWS Account

### The Root User Problem

When you create an AWS account, you get an **account root user** — it has
**full, unrestricted access to everything**. This is the most dangerous
credential in your account.

> ⚠️ AWS strongly recommends: **never use the root user for day-to-day tasks.**
> Some tasks require root (see the AWS documentation for the full list), but
> routine work should always use an IAM user.

### Four Steps to Secure a New Account

#### Step 1 — Stop using the root user immediately

1. While logged in as root, **create an IAM user** for yourself (save access keys if needed)
2. **Create an IAM group** with full administrator permissions, add your IAM user
3. **Disable and remove root user access keys** (if they exist)
4. **Enable a password policy** for all users
5. **Sign in with your new IAM user** from now on
6. **Store root credentials securely** — in a safe, not in your wallet

#### Step 2 — Enable MFA

Enable MFA for:
- The **account root user** (highest priority)
- **All IAM users**
- **AWS service APIs** (where applicable)

MFA token options:
- **Virtual apps** — Google Authenticator, Authy
- **U2F security keys** — YubiKey
- **Hardware devices** — Gemalto key fob or display card

#### Step 3 — Enable AWS CloudTrail

CloudTrail tracks **all API calls** (user activity) across all supported services.

```
Free default:    90 days of management event history (always on)
Extended logs:   Create a trail → logs stored in S3 → beyond 90 days
```

**How to access:**
1. Log in to AWS Management Console → CloudTrail service
2. Click **Event history** → view, filter, search last 90 days

**To enable extended logging:**
1. CloudTrail Console → Trails → **Create trail**
2. Name it, apply to all Regions, create a new S3 bucket for log storage
3. Restrict access to the S3 bucket (admins only)

#### Step 4 — Enable a Billing Report

Enable **AWS Cost and Usage Report** (CUR):
- Delivered to an S3 bucket you specify
- Updated at least once per day
- Tracks usage and provides estimated charges by hour or day

> 💡 Billing alerts are not security in the traditional sense, but unexpected
> charges often indicate unauthorized resource creation — a common sign of
> account compromise.

---

### Section 3 Best Practices Summary

| Practice | Why |
|----------|-----|
| Stop using root user | Root has unlimited access — too dangerous for daily use |
| Delete root access keys | Access keys can be used programmatically — remove the attack surface |
| Create individual IAM users | Principle of least privilege; audit trail per user |
| Use groups for permissions | Easier to manage; consistent permission assignment |
| Configure strong password policy | Reduces risk of credential compromise |
| Delegate using roles, not credential sharing | Roles provide temporary credentials; shared credentials are unauditable |
| Monitor with CloudTrail | Visibility into every API call in the account |

---

## 🏦 Part 4 — Securing Accounts

### AWS Organizations (Security Features)

Beyond billing consolidation, AWS Organizations provides critical security controls:

| Feature | How it works |
|---------|-------------|
| **Organizational Units (OUs)** | Group accounts by function, environment, or compliance requirement; attach different policies to each OU |
| **IAM integration** | Effective permissions = intersection of what Organizations allows AND what IAM grants |
| **Service Control Policies (SCPs)** | Define the **maximum permissions** member accounts can have — overrides even account administrators |

**SCPs are guardrails, not grants:**
- SCPs use similar JSON syntax to IAM policies
- **SCPs never grant permissions** — they only limit what is possible
- Even if an account admin explicitly grants a permission, an SCP can block it

```
Effective permission = IAM policy allows AND SCP allows
                       (both must say yes)
```

---

### AWS Key Management Service (AWS KMS)

| Feature | Detail |
|---------|--------|
| **Creates and manages encryption keys** | Customer Master Keys (CMKs) control access to data encryption keys |
| **Controls encryption across AWS** | Integrates with S3, EBS, EFS, RDS, and more |
| **CloudTrail integration** | Logs every key usage — meets regulatory requirements |
| **Hardware security modules (HSMs)** | FIPS 140-2 validated — keys protected in tamper-resistant hardware |

**CMK (Customer Master Key):**
- You control who can access and use each key
- You can create new keys at any time
- You can import keys from your own key management infrastructure

---

### Amazon Cognito

Cognito provides **user authentication and access control** for web and mobile apps.

- Define roles and map users to roles → app accesses only authorized resources
- Supports **SAML 2.0** (Security Assertion Markup Language) — open standard for
  identity federation
- Enables **Single Sign-On (SSO)** — sign in once with corporate credentials
  (e.g. Microsoft Active Directory) to access all SAML-enabled applications

---

### AWS Shield — DDoS Protection

| Tier | Cost | Protection |
|------|------|-----------|
| **AWS Shield Standard** | Free — automatically enabled for all customers | Infrastructure layer attacks (UDP floods), state exhaustion attacks (TCP SYN floods), application layer attacks (HTTP floods) |
| **AWS Shield Advanced** | Optional paid service | Enhanced protection for EC2, ELB, CloudFront, Global Accelerator, Route 53; access to DDoS Response Team (requires Business or Enterprise Support) |

---

## 🔒 Part 5 — Securing Data on AWS

### Encryption at Rest

Data at rest = data stored physically (on disk or tape). Encryption makes
data unreadable without the secret key.

**AWS KMS manages secret keys** for encryption across:
- Amazon S3
- Amazon EBS
- Amazon EFS
- Amazon RDS managed databases

---

### Encryption in Transit

Data in transit = data moving across a network.

| Tool | What it does |
|------|-------------|
| **TLS (Transport Layer Security)** | Open standard protocol (formerly SSL) — encrypts data in transit using AES-256 |
| **HTTPS** | HTTP over TLS/SSL — protects against eavesdropping and man-in-the-middle attacks |
| **AWS Certificate Manager** | Provision, manage, deploy, and renew SSL/TLS certificates for AWS resources (load balancers, CloudFront) |
| **AWS Storage Gateway** | Encrypts data in transit when connecting on-premises storage to Amazon S3 |

> 💡 HTTP traffic is unencrypted — anyone on the network path can read it.
> HTTPS encrypts the communication bidirectionally. Always use HTTPS for
> web traffic involving sensitive data.

---

### Controlling Access to Amazon S3

By default, **all S3 buckets are private** — accessible only to users explicitly
granted access.

| Tool | When to use |
|------|------------|
| **Amazon S3 Block Public Access** | Overrides all other policies — use this for any bucket that should never be public. Simplest protection against accidental exposure. |
| **IAM policies** | Control which users or roles can access specific buckets and objects |
| **Bucket policies** | Grant access across AWS accounts or public/anonymous access — use carefully and test thoroughly |
| **Access Control Lists (ACLs)** | Legacy mechanism (predates IAM) — use sparingly; avoid overly permissive settings |
| **AWS Trusted Advisor** | Bucket permission check — identifies buckets with global access permissions |

> ⚠️ If you define an explicit deny in a bucket policy, it restricts access
> even if the user has an IAM policy that grants access. Explicit deny always
> wins.

---

## ✅ Part 6 — Working to Ensure Compliance

### AWS Compliance Programs

Customers are subject to security and compliance regulations. AWS engages
with certifying bodies and independent auditors to provide detailed evidence
of its controls.

Compliance programs fall into three categories:

| Category | Description | Examples |
|----------|-------------|---------|
| **Certifications and attestations** | Assessed by third-party independent auditors | ISO 27001, ISO 27017, ISO 27018, ISO/IEC 9001 |
| **Laws, regulations, and privacy** | AWS provides features and legal agreements to support compliance | EU GDPR, HIPAA |
| **Alignments and frameworks** | Industry or function-specific requirements | Center for Internet Security (CIS), EU-US Privacy Shield |

---

### AWS Config

AWS Config continuously **monitors, records, and evaluates** your AWS resource
configurations.

| Capability | What it does |
|-----------|-------------|
| **Resource inventory** | Tracks all resources that exist in your account |
| **Configuration recording** | Records every configuration change over time |
| **Compliance evaluation** | Checks recorded configurations against your desired configuration rules |
| **Non-compliance flagging** | Flags resources that violate your rules — alerts you to issues |
| **Change history** | Review detailed resource configuration histories for audit and troubleshooting |

> ⚠️ AWS Config is a **Regional service**. To track resources across Regions,
> enable it in every Region you use. The aggregator feature provides a
> combined view across Regions and accounts.

---

### AWS Artifact

AWS Artifact provides **on-demand access to AWS security and compliance reports**
and select online agreements — directly from the AWS Management Console.

Available downloads include:
- AWS ISO certifications
- Payment Card Industry (PCI) reports
- Service Organization Control (SOC) reports

**How to access:**
```
AWS Management Console → Security, Identity & Compliance → Artifact
```

---

### Section 6 Key Takeaways

- **AWS compliance programs** provide information about policies, processes,
  and controls established and operated by AWS
- **AWS Config** assesses, audits, and evaluates the configurations of AWS
  resources continuously
- **AWS Artifact** provides access to security and compliance reports on demand

---

## 💡 Insights

**The shared responsibility model clarifies ownership — use it actively:**
When designing any architecture, map each security concern to "is this AWS's
job or mine?" before assuming a service is secure by default. The Oracle
database example shows how the same technology (Oracle DB) can shift
responsibility entirely based on deployment choice (EC2 vs RDS).

**IAM roles are the right answer for application credentials:**
Hardcoding AWS credentials in application code or configuration files is a
persistent anti-pattern. Roles provide temporary credentials automatically
rotated by AWS — no secret to leak, no rotation to manage.

**Explicit deny is the most powerful IAM tool:**
Most people focus on what they allow. But the explicit deny wins over
everything — even permissions granted in another policy, another account,
or at the organization level. Using explicit deny for critical resources
is a safety net that works even when other policies are misconfigured.

**The four account setup steps are a checklist, not a suggestion:**
Root user → MFA → CloudTrail → billing alerts. These four steps take about
30 minutes on a new account and prevent the most common account compromise
scenarios. Missing any one of them is a meaningful risk.

---

## ⚖️ Reflection

**What clicked:**
- The IaaS/PaaS/SaaS breakdown of responsibility is the most concrete way
  to understand the shared model — not abstract "of vs in" language, but
  "EC2 = you patch the OS; RDS = AWS patches the DB"
- IAM policy evaluation (explicit deny wins → explicit allow → implicit deny)
  is a simple algorithm that explains every surprising permission behavior
- SCPs as guardrails rather than grants is a mental model worth locking in —
  SCPs cap what's possible, IAM policies determine what's actually granted
  within that cap

**Still unclear:**
- How does SAML federation with Amazon Cognito compare to AWS IAM Identity
  Center (formerly SSO)? When would you choose one over the other?
- What is the difference between AWS Config rules and AWS Security Hub?
  They both seem to evaluate compliance — do they overlap?

**Next steps:**
- Module 5: Networking — understand VPCs, subnets, and security groups
  before trying to configure them correctly
- Explore the IAM Policy Simulator to test policies before applying them
  to real users

---

## 📚 Key Terms

| Term | Definition |
|------|-----------|
| **Shared responsibility model** | AWS secures the cloud infrastructure; customer secures what they put in the cloud |
| **IAM** | AWS Identity and Access Management — controls who can access what in AWS |
| **IAM user** | A person or application with its own credentials to authenticate to AWS |
| **IAM group** | Collection of IAM users sharing the same permissions |
| **IAM policy** | JSON document defining allowed and denied actions on resources |
| **IAM role** | Temporary identity with permissions — assumable by users, apps, or services |
| **Least privilege** | Grant only the minimum permissions required to perform a task |
| **MFA** | Multi-Factor Authentication — second factor beyond password |
| **CloudTrail** | AWS service that logs all API calls across your account |
| **SCP** | Service Control Policy — caps the maximum permissions in an AWS Organizations account |
| **AWS KMS** | Key Management Service — creates and manages encryption keys |
| **CMK** | Customer Master Key — controls access to data encryption keys |
| **TLS** | Transport Layer Security — protocol for encrypting data in transit |
| **AWS Config** | Continuously monitors and records AWS resource configurations for compliance |
| **AWS Artifact** | On-demand access to AWS security and compliance reports |
| **SAML 2.0** | Open standard for identity federation and single sign-on |
| **Implicit deny** | Default state in IAM — all actions denied unless explicitly allowed |
| **Explicit deny** | A policy statement that denies access — always takes precedence over any allow |

---

## 📚 References

- [AWS Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/)
- [AWS IAM Documentation](https://docs.aws.amazon.com/iam/)
- [IAM Policy Simulator](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_testing-policies.html)
- [Tasks Requiring Root User Credentials](https://docs.aws.amazon.com/general/latest/gr/root-vs-iam.html#aws_tasks-that-require-root)
- [AWS CloudTrail](https://aws.amazon.com/cloudtrail/)
- [AWS KMS](https://aws.amazon.com/kms/)
- [AWS Shield](https://aws.amazon.com/shield/)
- [AWS Config](https://aws.amazon.com/config/)
- [AWS Artifact](https://aws.amazon.com/artifact/)
- [Oracle Database on AWS Best Practices](https://docs.aws.amazon.com/whitepapers/latest/oracle-database-aws-best-practices/oracle-database-aws-best-practices.html)
