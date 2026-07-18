# Module 6 — Compute 💻

> AWS Academy Cloud Foundations | Module 6

---

## 📌 Overview

This module covers the core AWS compute services — from virtual machines
and containers to serverless and platform-as-a-service. Every workload you
run in AWS uses some form of compute, and choosing the right service directly
impacts cost, performance, and operational complexity.

**Topics covered:**
- Compute services overview and categories
- Amazon EC2 — launch configuration, AMIs, instance types, storage, lifecycle
- Amazon EC2 cost optimization — pricing models, the four pillars
- Container services — Docker, ECS, EKS, ECR
- AWS Lambda — serverless computing, event sources, limits
- AWS Elastic Beanstalk — PaaS deployment

---

## 🗂️ Part 1 — Compute Services Overview

### The AWS Compute Catalog

AWS offers a broad set of compute services, each optimized for specific use cases:

| Service | What it does |
|---------|-------------|
| **Amazon EC2** | Resizable virtual machines in the cloud |
| **Amazon EC2 Auto Scaling** | Automatically launches or terminates instances based on defined conditions |
| **Amazon ECR** | Store and retrieve Docker container images |
| **Amazon ECS** | Container orchestration service supporting Docker |
| **VMware Cloud on AWS** | Provision a hybrid cloud without custom hardware |
| **AWS Elastic Beanstalk** | Simple way to run and manage web applications (PaaS) |
| **AWS Lambda** | Serverless compute — pay only for compute time used |
| **Amazon EKS** | Run managed Kubernetes on AWS |
| **Amazon Lightsail** | Simple service for building applications or websites |
| **AWS Batch** | Run batch jobs at any scale |
| **AWS Fargate** | Run containers without managing servers or clusters |
| **AWS Outposts** | Run select AWS services in your on-premises data center |
| **AWS Serverless Application Repository** | Discover, deploy, and publish serverless applications |

---

### Four Broad Compute Categories

```
VIRTUAL MACHINES (IaaS)    → Amazon EC2
SERVERLESS                 → AWS Lambda
CONTAINER-BASED            → ECS, EKS, Fargate, ECR
PLATFORM AS A SERVICE      → AWS Elastic Beanstalk
```

| Category | Service | Key characteristic |
|----------|---------|-------------------|
| **IaaS / VM** | Amazon EC2 | You choose OS, size, resources — most flexibility, most responsibility |
| **Serverless** | AWS Lambda | Zero administration; run code without servers; pay per execution |
| **Container-based** | ECS, EKS, Fargate, ECR | Multiple workloads on one OS; spins up faster than VMs |
| **PaaS** | Elastic Beanstalk | AWS manages OS, app server, infrastructure — focus on code |

> 💡 The optimal compute service depends on your use case. **Legacy code may
> dictate your starting point**, but metrics should drive re-evaluation.
> Best practices: evaluate available options → understand configuration choices
> → collect metrics → use elasticity → re-evaluate based on metrics.

---

## 🖥️ Part 2 — Amazon EC2

### Why EC2 Exists

Running servers on-premises is expensive:
- Hardware must be procured based on projected needs, not actual usage
- Data centers require physical space, staff, power, and cooling
- Capacity must be over-provisioned for peak traffic
- Idle capacity wastes money continuously

**Amazon EC2 (Elastic Compute Cloud)** solves these problems by providing
resizable compute capacity in the cloud with secure, virtual machines.

**The EC2 acronym explained:**
- **Elastic** — easily increase/decrease the number or size of servers
- **Compute** — host applications and process data requiring CPU and RAM
- **Cloud** — EC2 instances are hosted in the AWS Cloud

**Common EC2 use cases:** Application servers, web servers, database servers,
game servers, mail servers, media servers, file servers, proxy servers.

**OS support:** Windows (2008, 2012, 2016, 2019), Red Hat, SuSE, Ubuntu,
Amazon Linux.

> 💡 The OS running on a VM is called the **guest operating system**; the
> OS directly on the physical hardware is the **host operating system**.

---

### Launching an EC2 Instance — Key Decisions

When using the Launch Instance Wizard, these are the core choices to make:

```
1. Choose an AMI
2. Choose an instance type
3. Configure network (VPC, subnet, public IP)
4. Configure IAM role
5. Add user data (optional startup script)
6. Configure storage
7. Add tags
8. Configure security groups
9. Choose/create a key pair
```

---

### 1. Amazon Machine Images (AMIs)

An **AMI** is a template for launching an EC2 instance. It contains:
- A **root volume template** — typically the OS and installed software
- **Launch permissions** — which AWS accounts can use the AMI
- **Block device mapping** — what volumes to attach at launch

**AMI sources:**

| Source | Description |
|--------|-------------|
| **Quick Start** | AWS pre-built AMIs (Linux and Windows options) |
| **My AMIs** | AMIs you created from your own instances |
| **AWS Marketplace** | Thousands of third-party software solutions (paid and free) |
| **Community AMIs** | Created by the community — not checked by AWS; use with caution |

> ⚠️ Never use Community AMIs in production or corporate environments —
> they are not vetted by AWS.

**AMI creation flow:**
```
Import VM / Start from existing AMI
          ↓
   Unmodified instance
          ↓
   Configure → "Golden instance" (OS + apps set exactly as you want)
          ↓
   Create AMI (EC2 stops instance, snapshots root volume, registers AMI)
          ↓
   Launch new instances from AMI in same Region
          ↓
   (Optional) Copy AMI to other Regions
```

---

### 2. EC2 Instance Types

Instance types provide varying combinations of CPU, memory, storage, and
networking. Choose based on your workload's requirements.

**Instance type name anatomy:**

```
t  3  .  2xlarge
│  │      │
│  │      └── Size (nano, micro, small, medium, large, xlarge, 2xlarge...)
│  └───────── Generation (higher = more powerful, better value)
└──────────── Family (T = burstable, C = compute, R = memory, etc.)
```

Sizes scale linearly: `t3.2xlarge` has 2× the vCPU and memory of `t3.xlarge`.

**Instance categories and use cases:**

| Category | Example family | Best for |
|----------|---------------|---------|
| **General Purpose** | T3, M5 | Websites, dev environments, microservices, code repos |
| **Compute Optimized** | C5 | Scientific modeling, batch processing, video encoding, gaming |
| **Memory Optimized** | R5 | In-memory databases, big data, Hadoop/Spark clusters, real-time processing |
| **Storage Optimized** | I3, D2 | High-frequency OLTP, data warehousing |
| **Accelerated Computing** | P3, G4 | Machine learning, GPU-intensive workloads |

**T3 specifically:** Provides a baseline CPU level with the ability to **burst**
above baseline — ideal for workloads with variable CPU needs.

**Networking considerations:**
- Network bandwidth varies by instance type (e.g. `a1.medium` = up to 10 Gbps;
  `p3dn.24xlarge` = up to 100 Gbps)
- **Placement groups** — specify that interdependent instances should be placed
  in the same AZ for lower latency and higher throughput
- **Enhanced networking** — significantly higher packets-per-second (PPS),
  lower network jitter, lower latency (supported on most instance types)

---

### 3. Network Configuration

- Choose **Region** before launching (must select the correct console Region)
- In a **default VPC**: instances get a public IP automatically
- In a **non-default VPC**: no public IP by default — configure subnet settings
  or override during launch

---

### 4. IAM Role for EC2 Instances

> ⚠️ **Never store AWS credentials on an EC2 instance.** This is highly insecure.

The correct approach: **attach an IAM role** to the EC2 instance. The role
grants the application running on the instance permission to make API calls
to other AWS services.

- An **instance profile** is the container for an IAM role; the console
  creates it automatically when you create a role for EC2
- You can attach or change IAM roles on running instances
- Changes to a role propagate to **all instances** with that role attached

**Example:** Application on EC2 needs to read from S3 → Attach an IAM role
with the appropriate S3 read policy. No credentials stored on the instance.

---

### 5. User Data (Startup Scripts)

User data lets you automate actions at instance launch — patch OS, install
software, configure settings.

```bash
#!/bin/bash
yum update -y
yum install -y wget
```

**Behavior:**
- Runs with **root privileges** during the final phases of the boot process
- On Linux: run by the `cloud-init` service
- On Windows: run by `EC2Config` or `EC2Launch`
- By default: runs **only the first time** the instance starts
- Can be configured to run on every boot using MIME multipart scripts

---

### 6. Storage Options

| Storage type | Characteristics | Best for |
|-------------|----------------|---------|
| **Amazon EBS** | Durable block storage; high performance; persists beyond instance stop | Root volumes, databases, transactional workloads |
| **Instance Store** | Ephemeral (temporary); physically attached to host; **data lost when instance stops** | Buffers, caches, scratch data, replicated fleets |
| **Amazon EFS** | Scalable NFS; shared across multiple instances; auto-scales to petabytes | Shared file systems, containerized workloads |
| **Amazon S3** | Object storage; accessed via API; not a block device | Backups, static assets, large datasets |

**Storage configuration examples:**

*Instance 1 (EBS-backed):*
```
Root: 20 GB EBS (survives stop/start)
Additional: 500 GB EBS (survives stop/start)
Additional: Instance Store Ephemeral (LOST on stop)
```

*Instance 2 (Instance Store root):*
```
Root: Ephemeral Instance Store (LOST on termination)
→ Cannot be stopped via API; only terminated
→ Data on root LOST on termination (survives reboot)
```

> 💡 Only instances backed by Amazon EBS can be **stopped and restarted**.
> Instance Store-backed root instances can only be **terminated**.

---

### 7. Tags

A **tag** is a key-value pair label attached to an AWS resource.

- Tag keys and values are **case-sensitive**
- Common tag: `Name = My Web Server` (displayed in the console Instances list)
- Tags power cost allocation, search, filtering, and organizational governance
- Best practice: develop a consistent **tagging strategy** across your organization

---

### 8. Security Groups

A **security group** is a virtual firewall controlling inbound and outbound
traffic at the **instance level**.

**Default behavior:**
- No inbound rules → all inbound traffic blocked until you add rules
- One outbound rule → all outbound traffic allowed

**Key properties:**
- **Stateful** — return traffic is automatically allowed
- **All rules evaluated simultaneously** before allowing/denying
- Changes apply immediately to all associated instances

**Rule components:** Type, Protocol, Port range, Source/Destination

Example rule: Allow SSH (TCP port 22) from My IP only.

---

### 9. Key Pairs

EC2 uses **public-key cryptography** for secure login.

- You specify a key pair at launch (existing or new)
- **Download the private key immediately** — this is the only chance to do so
- **Windows instances:** Use the private key to decrypt the administrator
  password, then connect via RDP
- **Linux instances:** The public key is placed in `~/.ssh/authorized_keys`;
  use the private key with SSH to connect

---

### EC2 Instance Lifecycle

From the uploaded diagram:

```
AMI → pending → running ←→ rebooting
                   ↓
              shutting-down
                   ↓
              terminated

running → stopping → stopped (EBS-backed only)
stopped → pending (Start) → running
```

| State | Description |
|-------|-------------|
| **Pending** | Instance is booting and being deployed to a host |
| **Running** | Fully booted; accessible over the internet |
| **Rebooting** | Stays on the same host; retains public DNS/IP and instance store data |
| **Shutting down** | Intermediary state before termination |
| **Terminated** | Visible in console briefly; cannot connect or recover |
| **Stopping** | EBS-backed instances only; transitioning to stopped |
| **Stopped** | Not charged at running rate; starts into pending on next start (may move to new host) |

> ⚠️ A stopped instance receives a **new public IP** when restarted, unless
> an **Elastic IP** is associated with it. Elastic IPs persist across stops
> and restarts.

---

### Instance Metadata

Access information about your running instance from within the instance:

```bash
# From within the instance (browser or curl)
curl http://169.254.169.254/latest/meta-data/

# Access user data
curl http://169.254.169.254/latest/user-data
```

`169.254.169.254` is a **link-local address** — valid only from within the
instance. Metadata includes: public IP, private IP, public hostname, instance
ID, security groups, Region, AZ, and more.

---

### Monitoring with CloudWatch

- **Basic monitoring** (free): metric data sent to CloudWatch every **5 minutes**
- **Detailed monitoring** (paid): metric data every **1 minute**
- CloudWatch stores metrics for **15 months**

> 💡 CloudWatch does **not** provide RAM metrics for EC2 by default — this
> requires a custom CloudWatch Agent configuration.

---

### EC2 via CLI

```bash
aws ec2 run-instances \
  --image-id ami-xxxxxxxx \
  --count 1 \
  --instance-type c3.large \
  --key-name MyKeyPair \
  --security-groups MySecurityGroup \
  --region us-east-1
```

---

### Section 2 Key Takeaways

- EC2 runs Windows and Linux virtual machines in the cloud
- Instances launch from AMI templates into a VPC
- Choose instance types based on CPU, RAM, storage, and network needs
- Security groups control access (allowed ports and sources)
- User data automates configuration at first launch
- Only EBS-backed instances can be stopped; instance store data is lost on stop
- CloudWatch captures and reviews EC2 metrics

---

## 💰 Part 3 — Amazon EC2 Cost Optimization

### Pricing Models

| Model | Commitment | Best for | Key property |
|-------|-----------|---------|-------------|
| **On-Demand** | None | Spiky, unpredictable, or short-term workloads | Most flexible; highest per-hour rate; eligible for Free Tier |
| **Reserved Instances** | 1 or 3 years | Steady, predictable long-term workloads | Significant discount vs On-Demand; fixed discounted price |
| **Scheduled Reserved** | 1 year (recurring schedule) | Predictable recurring workloads (daily/weekly/monthly) | Pay for scheduled time even if unused |
| **Spot Instances** | None (bid-based) | Fault-tolerant, interruptible workloads | Up to 90% discount; 2-minute warning before interruption |
| **Dedicated Hosts** | On-Demand or Reserved | Existing per-socket/per-core/per-VM software licenses; compliance requirements | Physical server dedicated to your use |
| **Dedicated Instances** | On-Demand or Reserved | Compliance requirements (hardware-level isolation) | Physically isolated at host hardware level from other accounts |

> ⚠️ **Spot Instances** can be interrupted with a **2-minute warning** — they
> can be configured to terminate, stop, or hibernate. Good for: web servers,
> API backends, big data processing, fault-tolerant batch jobs.

> 💡 Per-second billing is available for **On-Demand, Reserved, and Spot
> instances** running Amazon Linux or Ubuntu only.

---

### The Four Pillars of Cost Optimization

```
1. RIGHT-SIZE          → Choose the smallest instance that meets performance needs
2. INCREASE ELASTICITY → Turn off unused instances; use Auto Scaling for peaks
3. OPTIMAL PRICING     → Mix On-Demand, Reserved, and Spot for your patterns
4. OPTIMIZE STORAGE    → Resize EBS volumes; choose cheaper storage types; delete unused snapshots
```

#### Right-Sizing

- Select the cheapest instance that still meets performance requirements
- Review CPU, RAM, storage, and network utilization
- Use CloudWatch metrics and load testing to identify downsizing opportunities
- Target the cheapest instance that passes your performance tests

#### Elasticity

- Turn off non-production instances outside business hours
  → potential **65% runtime cost reduction** for development/test instances
- Use Auto Scaling for production — scale out for peaks, scale in when idle
- Rule of thumb: **20–30% of EC2 instances** should run as On-Demand or Spot,
  with the rest reserved

#### Pricing Model Mix

- Combine On-Demand + Reserved + Spot based on workload predictability
- Consider Lambda as an alternative — for some workloads, serverless is
  significantly cheaper than running EC2 24/7

#### Storage Optimization

- **Resize EBS volumes** — a 500-GB volume needed only 20 GB wastes 480 GB
- **Choose cheaper EBS types**: Throughput Optimized HDD (st1) ≈ half the cost
  of General Purpose SSD (gp2) — use if it meets your performance needs
- **Delete unneeded EBS snapshots** — they accumulate and cost money
- **Consider S3 over EBS** for data that doesn't need block storage access
- **Automate data lifecycle policies** — move cold data to S3 Glacier

---

### Ongoing Cost Optimization Tools

| Tool | Purpose |
|------|---------|
| **Tagging** | Attribute costs to teams, projects, or cost centers |
| **AWS Cost Explorer** | Visualize spending patterns; identify trends and anomalies |
| **AWS Trusted Advisor** | Real-time recommendations for cost, security, and performance |

> 💡 Cost optimization is not a one-time event — it requires continuous
> measurement, analysis, and adjustment. Assign cost optimization ownership
> to a specific individual or team for best results.

---

## 📦 Part 4 — Container Services

### What are Containers?

**Containers** are a method of OS virtualization that packages an application
with all its dependencies — code, libraries, system tools, runtime — into an
isolated, portable unit.

| Property | Detail |
|----------|--------|
| **Smaller than VMs** | Don't include an entire OS; share the virtualized OS kernel |
| **Faster startup** | Spin up in hundreds of milliseconds vs. minutes for VMs |
| **Consistent deployments** | Same container runs identically in dev, test, and production |
| **Resource isolation** | Each container gets its own process space |
| **Portable** | Runs on any host with Docker and kernel support |

---

### Docker

**Docker** is the software platform that packages applications into containers.
Installed on host servers, it provides simple commands to build, start, and stop
containers.

**Use Docker when you want to:**
- Standardize environments across dev/test/prod
- Reduce language stack and version conflicts
- Run microservices with standardized deployments
- Need portability for data processing
- Use containers as a service

**VMs vs Containers (architecture):**

```
VM-based deployment:               Container-based deployment:
Hypervisor (AWS)                   Hypervisor (AWS)
  EC2 (VM) → App 1                    EC2 (VM)
  EC2 (VM) → App 2                      Linux Guest OS
  EC2 (VM) → App 3                        Docker Engine
                                            Container → App 1
                                            Container → App 2
                                            Container → App 3
```

In the container model, **all three apps run on a single EC2 instance** — the
Docker engine manages container lifecycle and isolation. A large EC2 instance
can host hundreds of containers.

---

### Amazon ECS — Elastic Container Service

**Amazon ECS** is a highly scalable, high-performance container management
service for Docker containers — you don't need to install and manage Kubernetes
yourself.

**Key capabilities:**
- Launch up to **tens of thousands of Docker containers** in seconds
- Monitor container deployment
- Manage cluster state
- Schedule containers using the built-in scheduler or third-party schedulers

**ECS concepts:**

| Concept | Definition |
|---------|-----------|
| **Task definition** | Text file describing 1–10 containers; the application blueprint |
| **Task** | An instantiation of a task definition running in a cluster |
| **Cluster** | Group of EC2 instances (or Fargate capacity) running containers |
| **ECS container agent** | Runs on each EC2 instance in the cluster |

**ECS cluster launch types:**

| Launch type | Who manages the cluster? | Control level |
|-------------|------------------------|--------------|
| **EC2 Linux/Windows** | You manage EC2 instances | Full control over infrastructure |
| **Fargate (Networking Only)** | AWS manages everything | Focus on containers only; no server management |

---

### Amazon EKS — Elastic Kubernetes Service

**Kubernetes** is open-source container orchestration software — portable,
extensible, and supported by a large community.

**Kubernetes concepts:**
- **Cluster** — group of compute nodes
- **Pods** — logical groupings of containers (each pod gets an IP and DNS name)
- Key advantage: **run containerized apps anywhere** with the same toolset —
  on-premises or in the cloud

**Amazon EKS** is a managed Kubernetes service — AWS manages the control plane
availability and scalability; you focus on running workloads.

- Certified Kubernetes conformant — existing Kubernetes apps are compatible
- Integrates with: **ALB** (load distribution), **IAM** (access control), **VPC** (networking)
- Auto-detects and replaces unhealthy control plane nodes

> 💡 **ECS vs EKS:** Both orchestrate Docker containers. ECS is AWS-native
> and simpler to start with. EKS is better if you already use Kubernetes or
> need Kubernetes-specific features and portability across environments.

---

### Amazon ECR — Elastic Container Registry

**Amazon ECR** is a fully managed Docker container registry.

- Store, manage, and deploy Docker container images
- Integrated with ECS and EKS — specify the ECR repository in your task
  definition
- Supports Docker Registry HTTP API v2 — interact with standard Docker CLI
- Transfers via **HTTPS**
- Images automatically **encrypted at rest** using Amazon S3 server-side encryption

---

### Section 4 Key Takeaways

- Containers hold everything an app needs to run
- Docker packages software into containers
- A single application can span multiple containers
- **ECS** orchestrates Docker containers; **EKS** orchestrates Kubernetes
- **ECR** stores, manages, and deploys container images

---

## ⚡ Part 5 — AWS Lambda

### What is Serverless Computing?

Serverless does not mean "no servers" — it means you never provision or
manage them. AWS handles all infrastructure management.

**AWS Lambda** is an event-driven, serverless compute service.

- Write code as a **Lambda function**
- Set it to trigger on a schedule or in response to an event
- Code runs only when triggered
- **Pay only for compute time consumed** — no charge when code isn't running
- Billing in **100-millisecond increments**

---

### Lambda Key Features

| Feature | Detail |
|---------|--------|
| **Supported languages** | Java, Go, PowerShell, Node.js, C#, Python, Ruby |
| **Custom libraries** | Use any library — native or third-party |
| **Fully managed** | AWS handles infrastructure, patching, scaling, monitoring |
| **Fault tolerance** | Maintains capacity across multiple AZs; no maintenance windows |
| **Built-in monitoring** | Logs to Amazon CloudWatch automatically |
| **Automatic scaling** | Scales from a few requests/day to thousands/second |
| **Complex workflows** | Orchestrate multiple functions with **AWS Step Functions** |

---

### Event Sources

A **Lambda function** is triggered by an **event source** — an AWS service or
developer-created application that generates events.

| Trigger type | How it works | Examples |
|-------------|-------------|---------|
| **Asynchronous push** | Service publishes events to Lambda directly | Amazon S3, Amazon SNS, CloudWatch Events |
| **Poll-based** | Lambda polls the source for new records | Amazon SQS, Amazon DynamoDB Streams |
| **Direct invocation** | You or another service calls Lambda directly | ELB Application Load Balancer, Amazon API Gateway, Lambda console, CLI, SDK |

---

### Creating a Lambda Function

Configuration steps in the AWS Management Console:

1. **Function name** — give it a name
2. **Runtime** — choose language/version (e.g. Python 3.11, Node.js 18.x)
3. **Execution role** — IAM role granting permissions to interact with other services
4. **Add trigger** — choose the event source
5. **Function code** — write in the console editor or upload a deployment package
6. **Memory** — 128 MB to 10,240 MB
7. **Optional** — environment variables, timeout, VPC, tags

All settings are packaged into a **Lambda deployment package** (ZIP archive of
code + dependencies).

---

### Lambda Use Case Examples

**Schedule-based — EC2 stop/start automation:**

```
22:00 GMT → CloudWatch Event triggers Lambda → Lambda stops EC2 instances
05:00 AM  → CloudWatch Event triggers Lambda → Lambda starts EC2 instances
```

Cost savings: ~65% on instances that don't need to run overnight.

**Event-based — S3 image thumbnail generator:**

```
1. User uploads image to S3 source bucket
2. S3 detects object-created event
3. S3 invokes Lambda function with event data
4. Lambda reads the image using the execution role
5. Lambda generates thumbnail using graphics libraries
6. Lambda saves thumbnail to S3 target bucket
```

No servers, no management, no cost when idle — only pay for the execution time.

---

### Lambda Limits (Know These)

| Limit | Value |
|-------|-------|
| **Maximum memory** | 10,240 MB per function |
| **Maximum run time** | 15 minutes (900 seconds) per invocation |
| **Concurrent executions** | 1,000 per Region (soft limit — can request increase) |
| **Deployment package size** | 250 MB (unzipped) |
| **Container image size** | Up to 10 GB (for large ML/data workloads) |

> 💡 **Layers** — ZIP archives containing libraries or dependencies — let you
> share code across functions and avoid hitting the deployment package size
> limit. A layer is referenced by the function but not included in its package.

Limits are either **soft** (adjustable via support ticket) or **hard** (cannot
be increased).

---

### Section 5 Key Takeaways

- Serverless computing = no server provisioning or management
- Lambda provides built-in fault tolerance and automatic scaling
- Event sources trigger Lambda functions — S3, SNS, SQS, API Gateway, etc.
- Maximum memory: **10,240 MB**
- Maximum run time: **15 minutes**

---

## 🌱 Part 6 — AWS Elastic Beanstalk

### What is Elastic Beanstalk?

**AWS Elastic Beanstalk** is a **Platform as a Service (PaaS)** that handles
the deployment, scaling, and management of web applications and services.

**Core value proposition:**
- Upload your code → Elastic Beanstalk handles everything else
- Capacity provisioning, load balancing, auto scaling, health monitoring
- You retain **full control** over the underlying AWS resources at any time

**Pricing:**
- **No additional charge** for Elastic Beanstalk itself
- Pay only for the underlying resources used (EC2 instances, S3 buckets, etc.)
- No minimum fees or upfront commitments

---

### Supported Platforms

| Platform | Application server |
|----------|-------------------|
| Java | Apache Tomcat |
| PHP | Apache HTTP Server |
| Python | Apache HTTP Server |
| Node.js | NGINX or Apache HTTP Server |
| Ruby | Passenger or Puma |
| .NET | Microsoft IIS |
| Go | Native Go server |
| Docker | Docker container |

---

### How to Deploy

Deployment interfaces:
- **AWS Management Console** — web-based
- **AWS CLI** — command line
- **Git repository** — direct integration
- **IDE plugins** — Eclipse, Visual Studio

---

### Key Benefits

| Benefit | Detail |
|---------|--------|
| **Developer productivity** | Focus on writing code, not managing infrastructure |
| **Simple deployment** | Upload once; Beanstalk handles the rest |
| **Reduced management complexity** | No need to manage load balancers, firewalls, networks manually |
| **Automatic scaling** | Scales up/down based on CPU utilization or custom triggers |
| **Platform maintenance** | AWS applies patches and updates to the underlying platform |
| **Full control retained** | Access and manage underlying resources whenever needed |

> 💡 Elastic Beanstalk is designed to be **difficult to outgrow** — it handles
> peaks in workload and traffic by automatically scaling, while minimizing costs
> during low-traffic periods.

---

### Section 6 Key Takeaways

- Elastic Beanstalk enhances developer productivity
- Simplifies the deployment process
- Reduces management complexity
- Supports: Java, .NET, PHP, Node.js, Python, Ruby, Go, Docker
- **No charge for Elastic Beanstalk** — pay only for AWS resources used

---

## 💡 Insights

**The compute decision is the most consequential architectural choice:**
EC2 gives maximum control at maximum operational cost (patching, scaling, HA
all on you). Lambda gives minimum operational overhead but requires designing
for stateless event-driven patterns. The right answer depends on your workload
characteristics — not what's newest or most talked about.

**EC2 instance type selection is worth optimizing before anything else:**
An R5 instance for a CPU-bound workload is wasted money. A T3 for a database
under constant load will hit its burst budget and throttle. Understanding the
three dimensions (compute/memory/storage optimized) before selecting an
instance type avoids a common and expensive mistake.

**Containers accelerate everything — but add a new operational layer:**
Containers solve the "it works on my machine" problem permanently. But
orchestrating containers at scale (ECS, EKS) adds operational complexity that
must be managed. AWS Fargate exists specifically to absorb that complexity —
if you don't need fine-grained cluster control, Fargate is often the right
default.

**Lambda's 15-minute limit is a design constraint, not a bug:**
Lambda is designed for short, event-driven executions. If your workload needs
to run for hours, Lambda is the wrong tool — EC2 or Fargate is. Understanding
these constraints before starting is what separates good architecture from
expensive redesigns.

**Elastic Beanstalk is underrated for teams that want speed without chaos:**
Teams that default to "raw EC2" often spend weeks configuring ALBs, Auto
Scaling Groups, launch templates, and health checks. Elastic Beanstalk
provides all of this out-of-the-box, and you can drop down to the underlying
resources whenever you need more control. The zero-cost model makes it worth
evaluating for every web application.

---

## ⚖️ Reflection

**What clicked:**
- The AMI "golden instance" pattern — configure once, snapshot, deploy
  everywhere — is the foundation of immutable infrastructure thinking
- The four pillars of cost optimization (right-size, elasticity, pricing mix,
  storage) apply to every AWS account and should be revisited quarterly
- Lambda's event-source model (S3 upload → Lambda → thumbnail) makes the
  serverless architecture pattern concrete — the code only runs when there's
  actual work to do, and you pay nothing when there isn't

**Still unclear:**
- How does **EC2 Auto Scaling** specifically interact with Reserved Instances?
  If I have Reserved capacity but Auto Scaling spins up more, do those new
  instances use the reservation or incur On-Demand charges?
- What are the tradeoffs between **ECS with EC2 launch type** vs **ECS with
  Fargate** for a production containerized microservice? Cost profile?
  Debugging experience?

**Next steps:**
- Module 7: Storage — EBS deep dive complements what we learned about EC2
  storage options
- Practice launching an EC2 instance with the Launch Instance Wizard in the
  AWS console
- Try the Lambda activity — deploy a schedule-based function for EC2 stop/start

---

## 📚 Key Terms

| Term | Definition |
|------|-----------|
| **EC2** | Elastic Compute Cloud — resizable VMs in the cloud |
| **AMI** | Amazon Machine Image — template for launching EC2 instances |
| **Instance type** | Hardware spec (CPU, memory, storage, network) of an EC2 instance |
| **Instance Store** | Ephemeral block storage physically attached to the host; lost on stop |
| **EBS** | Elastic Block Store — durable block storage for EC2 |
| **User data** | Script that runs at first boot of an EC2 instance |
| **Key pair** | Public/private key used for secure login to EC2 instances |
| **Security group** | Virtual firewall at the instance level; stateful; allow rules only |
| **Placement group** | Spec for placing interdependent instances close together for low latency |
| **Enhanced networking** | Higher PPS, lower jitter and latency on supported instance types |
| **Elastic IP** | Static public IPv4 address; persists across instance stop/start |
| **On-Demand** | Pay per second/hour; no commitment; most flexible |
| **Reserved Instance** | 1 or 3 year commitment; significant discount |
| **Spot Instance** | Bid on unused capacity; up to 90% discount; 2-min interruption warning |
| **Dedicated Host** | Physical server dedicated to your use; supports per-socket licensing |
| **Container** | Packaged app + dependencies running as an isolated process on a shared OS |
| **Docker** | Software platform for building and running containers |
| **ECS** | Elastic Container Service — AWS-native Docker orchestration |
| **EKS** | Elastic Kubernetes Service — managed Kubernetes on AWS |
| **ECR** | Elastic Container Registry — managed Docker image registry |
| **Fargate** | Serverless container compute — no cluster management required |
| **Task definition** | ECS blueprint describing 1–10 containers |
| **Lambda function** | Custom code that Lambda runs in response to events |
| **Event source** | AWS service or app that triggers a Lambda function |
| **Lambda layer** | ZIP archive of shared libraries/dependencies for Lambda functions |
| **Elastic Beanstalk** | PaaS — upload code, AWS handles everything else |

---

## 📚 References

- [Amazon EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)
- [EC2 Pricing](https://aws.amazon.com/ec2/pricing/)
- [Amazon ECS Documentation](https://docs.aws.amazon.com/ecs/)
- [Amazon EKS Documentation](https://docs.aws.amazon.com/eks/)
- [Amazon ECR Documentation](https://docs.aws.amazon.com/ecr/)
- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- [AWS Lambda Quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)
- [AWS Elastic Beanstalk Documentation](https://docs.aws.amazon.com/elasticbeanstalk/)
- [EC2 Placement Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html)
- [EC2 Enhanced Networking (ENA)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enhanced-networking-ena.html)
- [AWS Tagging Best Practices](https://d1.awsstatic.com/whitepapers/aws-tagging-best-practices.pdf)
