# AWS Certified Solutions Architect – Associate (SAA-C03): Study Guide

This guide helps you prepare for the **AWS Certified Solutions Architect – Associate (SAA-C03)** exam. It follows the **4 official domains** and uses a concise, scenario-focused Q&A format with collapsible sections and practical examples.

> ℹ️ **Exam snapshot (SAA-C03)**
>
> * **Level:** Associate
> * **Exam length:** **130 minutes**
> * **Format:** **65 questions** total (multiple choice & multiple response). AWS states 65 questions; the official exam guide explains that **50 are scored** and **15 are unscored** pilot items. ([Amazon Web Services, Inc.][1])
> * **Passing score:** 720 (scaled 100–1,000) 
> * **Domains:**
>
>   * Domain 1 – Design Secure Architectures (**30%**)
>   * Domain 2 – Design Resilient Architectures (**26%**)
>   * Domain 3 – Design High-Performing Architectures (**24%**)
>   * Domain 4 – Design Cost-Optimized Architectures (**20%**) 
> * **Validity:** Certification is valid for **3 years**. ([Amazon Web Services, Inc.][1])
>
> 🔍 **Always check** the official AWS certification page for the latest info on exam version, policies, and languages. ([Amazon Web Services, Inc.][1])

---

## How to use this guide

* Treat each `<details>` block as a **practice question**.
* Before expanding, **answer it yourself** out loud or in notes.
* Focus on **service selection questions** and **tradeoffs** (security, availability, performance, cost) – that’s exactly how the real exam is written. 

---

## Domain 1: Design Secure Architectures (30%)

### 1.1 Design secure access to AWS resources

<details>
<summary>Explain the AWS shared responsibility model (what you secure vs what AWS secures)</summary>

**Concept:**

* AWS is responsible for **security *of* the cloud**

  * Physical facilities, hardware, networking, hypervisor, managed services infrastructure, etc.
* You are responsible for **security *in* the cloud**

  * IAM, OS hardening, network controls, data encryption, logging, patching for EC2/self-managed, etc. 

**Exam tips:**

* Anything about **patching EC2, configuring security groups, managing IAM, encrypting your data** → **customer responsibility**.
* Anything about **facility security, hardware, global infrastructure, managed service underlying infra** → **AWS responsibility**.

</details>

<details>
<summary>Design secure IAM: users vs groups vs roles vs policies</summary>

**Key pieces:**

* **IAM users** – long-lived identities for people or apps that **can’t** use roles (legacy; exam expects you to **minimize** these).
* **IAM groups** – collections of users with shared **identity-based policies**.
* **IAM roles** – **temporary credentials** assumed by users, apps, or services via **STS**.
* **Policies** – JSON documents that **allow/deny** actions on resources.

**Least privilege example:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::my-bucket/app/*"
    }
  ]
}
```

**What the exam loves:**

* Prefer **roles over access keys** for apps.
* Enable **MFA on root** and avoid using it for daily tasks. 
* Attach policies to **groups/roles**, not directly to users (in most scenarios).

</details>

<details>
<summary>When to use identity-based policies vs resource-based policies</summary>

**Identity-based policies** (IAM user/group/role):

* “Who can do *what* on *which* resources?”
* Example: Allow a role to read/write a specific S3 bucket.

**Resource-based policies** (S3 bucket policies, KMS key policies, SQS queue policies, etc.):

* “This resource **trusts** these principals.”
* Common use cases:

  * **Cross-account access** to S3, KMS, SQS, SNS.
  * Allow a specific AWS service to access the resource. 

Exam rules of thumb:

* **Single account?** – Usually identity-based policies are enough.
* **Cross-account or public access?** – Expect to see **resource policies** in correct answers.

</details>

<details>
<summary>Design multi-account security and centralized access</summary>

Tools and patterns:

* **AWS Organizations + Service Control Policies (SCPs):**

  * Apply **guardrails** across multiple accounts (e.g., “No public S3 buckets”, “Deny disable CloudTrail”).
* **AWS Control Tower:**

  * Automates **landing zone** setup with org units, baselines, guardrails. 
* **AWS IAM Identity Center (AWS SSO):**

  * Central SSO to AWS accounts and apps.
  * Assign **permission sets** that map to IAM roles in each account.

Exam patterns:

* “Company has many AWS accounts, wants centralized governance” → **Organizations + Control Tower + SCPs**.
* “Central login with existing corporate IdP (AD/Okta/etc) to multiple AWS accounts” → **Identity Center + federation**.

</details>

<details>
<summary>Explain federation & STS: how external identities get AWS access</summary>

**High-level flow:**

1. User authenticates to **IdP** (SAML/OIDC; e.g., Active Directory Federation Services, Okta).
2. IdP issues an assertion.
3. AWS **STS** issues **temporary credentials** for an IAM role.
4. User uses AWS console/CLI with these short-lived credentials.

**When to use:**

* Large enterprise with **existing directory**.
* Need **centralized identity lifecycle** (joining/leaving company).
* Want **no long-lived IAM users**.

Look for terms like **“federation”, “SAML 2.0”, “STS:AssumeRole”, “single sign-on across accounts”**.

</details>

---

### 1.2 Design secure workloads and applications

<details>
<summary>Secure VPC architecture: security groups vs network ACLs</summary>

**Security groups (SGs):**

* **Stateful**, instance-/ENI-level.
* Allow rules only (deny is implicit).
* Return traffic is automatically allowed.

**Network ACLs (NACLs):**

* **Stateless**, subnet-level.
* Numbered **allow/deny** rules, evaluated in order.
* Return traffic must be explicitly allowed.

Exam patterns:

* “Block a specific IP” → often **NACL**.
* “Simplify instance-level rules” or “app-level security” → **SGs**.

Also know:

* **Public subnet:** route to **IGW**.
* **Private subnet with outbound internet:** route to **NAT Gateway** (or NAT instance).
* **VPC endpoints:** private access to S3, DynamoDB, etc., without traversing the public internet. 

</details>

<details>
<summary>Protect applications from common web threats (DDoS, SQLi, XSS)</summary>

Important services:

* **AWS Shield** – DDoS protection:

  * Shield Standard – automatic for all customers on CloudFront/Route 53.
  * Shield Advanced – additional protection, cost protection, 24/7 DDoS response.
* **AWS WAF** – Web application firewall:

  * Blocks **SQL injection, XSS, bad bots** using rules.
  * Integrates with **ALB, API Gateway, CloudFront**.
* **Amazon CloudFront** – edge caching + **DDoS protection** front-door.
* **Amazon GuardDuty** – **threat detection** (VPC Flow Logs, CloudTrail, DNS logs). 

Exam patterns:

* “Protect public web app from SQL injection” → **WAF**.
* “Mitigate large DDoS on public website” → **Shield + CloudFront + Route 53**.
* “Detect suspicious activity in logs” → **GuardDuty**.

</details>

<details>
<summary>Secure external connectivity: VPN, Direct Connect, and hybrid patterns</summary>

* **Site-to-Site VPN:**

  * Encrypted IPsec tunnel over the internet.
  * Faster to set up; variable latency.
* **AWS Direct Connect:**

  * Dedicated private line; predictable performance; **not inherently encrypted** (encrypt with VPN over DX if needed).
* **Client VPN:**

  * Remote users connect securely into VPC.

Exam patterns:

* “Stable, consistent high-throughput connection from data center” → **Direct Connect**.
* “Quick, low-cost hybrid connectivity” → **Site-to-Site VPN**.
* “Need both reliability and encryption for hybrid” → **Direct Connect + VPN** (aka DX with VPN as backup/overlay).

</details>

---

### 1.3 Determine appropriate data security controls

<details>
<summary>Encrypt data at rest with KMS, and manage keys securely</summary>

**Services that integrate with KMS:**

* S3, EBS, RDS, DynamoDB, EFS, FSx, many more. 

Key types:

* **AWS-managed keys (aws/*)** – minimal overhead.
* **Customer managed keys (CMKs)** – you control:

  * Key policy and grants.
  * Rotation (automatic for some key types).
  * Access via IAM policies.

Exam patterns:

* “Compliance requires full control over key policies and rotation” → **customer managed CMK**.
* “Encrypt S3 bucket with per-team access control” → KMS CMK + **key policy + IAM**.

</details>

<details>
<summary>Encrypt data in transit using ACM and TLS</summary>

* Use **AWS Certificate Manager (ACM)** to issue and manage TLS certificates:

  * For **ALB, NLB (TLS), CloudFront, API Gateway, ELB**, etc.
* Always think **HTTPS/TLS** for public endpoints.
* For internal microservices, ALB/API Gateway + ACM certs is common. 

Exam patterns:

* “Rotate certificates automatically” → use **ACM-issued** certificates (auto rotation).
* “Customer brings own certificate” → **import into ACM**.

</details>

<details>
<summary>Backup, replication, and data lifecycle for security & durability</summary>

Common tools:

* **S3:**

  * Versioning, Cross-Region Replication (CRR), lifecycle rules (transition to **Glacier/Glacier Deep Archive**).
* **RDS / Aurora:**

  * Automated backups, manual snapshots, **Multi-AZ**, read replicas, cross-Region replicas.
* **AWS Backup:**

  * Centralized backup policies for EBS, RDS, DynamoDB, EFS, FSx, etc.
* **DynamoDB:**

  * Point-in-time recovery (PITR), on-demand backups.

Exam patterns:

* “Meet compliance for 7-year retention at low cost” → S3 lifecycle to **Glacier / Deep Archive**.
* “Fast failover within Region” → **Multi-AZ** database.
* “Isolated copy for DR in another Region” → **cross-Region replication / snapshots**.

</details>

---

## Domain 2: Design Resilient Architectures (26%)

Domain 2 focuses on **scalability, loose coupling, high availability, fault tolerance, and DR strategies** across Regions and AZs. 

### 2.1 Design scalable and loosely coupled architectures

<details>
<summary>Design event-driven & microservices architectures (SQS, SNS, EventBridge, Lambda, containers)</summary>

Key services:

* **Amazon SQS** – queue for **decoupling** producers/consumers.
* **Amazon SNS** – **pub/sub** fan-out.
* **Amazon EventBridge** – event bus + routing based on **event patterns**.
* **AWS Lambda / Fargate / ECS / EKS** – stateless compute. 

Patterns:

* **Producer → SQS → Consumers**:

  * Smooths traffic spikes; supports buffering and retries.
* **SNS → SQS/Lambda/HTTP fan-out**:

  * E.g., “When an object is uploaded to S3, notify multiple systems.”
* **API Gateway → Lambda / ECS**:

  * Serverless/microservice front-door.

Exam patterns:

* “Components must scale independently and avoid tight coupling” → **SQS/SNS/EventBridge** in the design.
* “Handle bursts without overloading downstream system” → put **SQS** in front.

</details>

<details>
<summary>When to use serverless vs containers vs EC2</summary>

* **Lambda / serverless:**

  * Event-driven, spiky or unpredictable workloads, short-running tasks.
  * You don’t manage servers or scaling.
* **Fargate / ECS / EKS (containers):**

  * Microservices, porting existing containerized apps, better control over runtime vs Lambda.
* **EC2:**

  * Full control over OS and runtime, long-running processes, specialized software.

Exam shortcuts:

* “Minimal ops, spiky traffic, pay per request” → **Lambda**.
* “Already containerized app, want to orchestrate” → **ECS/EKS**.
* “Legacy app requiring custom OS-level packages” → **EC2**. 

</details>

<details>
<summary>Select storage patterns for loosely coupled designs</summary>

* **Object storage:** S3 – static assets, backups, logs, data lake.
* **Block:** EBS – attached to EC2; high-performance block storage.
* **File:** EFS / FSx – shared POSIX file systems for multiple instances or containers.

Loose coupling examples:

* Store files in **S3**, not on EC2 instance disk.
* Use **S3 events → SQS / EventBridge → Lambda** for processing.

</details>

---

### 2.2 Design highly available and fault-tolerant architectures

<details>
<summary>Multi-AZ vs Multi-Region: when do you use each?</summary>

* **Multi-AZ (within Region):**

  * Protects against **AZ failure**.
  * Used by **RDS Multi-AZ, Aurora, EFS, ALB across subnets**, etc.
* **Multi-Region:**

  * Protects against **Regional failures**, meets ultra-low RPO/RTO or regulatory needs.

Exam patterns:

* “Require high availability within one Region” → **Multi-AZ** + load balancing.
* “Mission-critical global app with very low RPO/RTO” → **multi-Region active/active** or active/passive with Route 53 failover. 

</details>

<details>
<summary>DR strategies: backup & restore, pilot light, warm standby, multi-site</summary>

Quick comparison (RPO/RTO from worst to best):

| Strategy         | Description (simplified)                                     |
| ---------------- | ------------------------------------------------------------ |
| Backup & restore | Periodic backups; restore infra and data after disaster.     |
| Pilot light      | Minimal core services running; scale out when needed.        |
| Warm standby     | Reduced-size full environment running, scaled down.          |
| Multi-site / a-a | Full-size environments in multiple Regions, serving traffic. |

Exam mapping:

* **Cheapest, longest RTO/RPO** → **backup & restore**.
* **Fastest recovery, highest cost** → **multi-site active/active**.
* “Small core infra running, can scale on disaster” → **pilot light**.
* “Scaled-down full stack always on” → **warm standby**. 

</details>

<details>
<summary>Eliminate single points of failure: load balancers, proxies, and queues</summary>

Design patterns:

* Use **ALB/NLB** across **multiple subnets in multiple AZs**.
* Use **RDS Multi-AZ** or **Aurora** instead of single-AZ DB.
* Use **SQS** between tiers so that if a consumer fails, messages are retried.
* Use **RDS Proxy** to pool connections and improve DB resilience. 

Exam patterns:

* If you see “single EC2 instance behind Route 53 A record” and requirement is high availability → fix by adding **ASG + ALB in multiple AZs**.
* “Too many DB connections from Lambda” → answer includes **RDS Proxy**.

</details>

---

## Domain 3: Design High-Performing Architectures (24%)

Domain 3 wants you to pick **the right compute, storage, database, network, and ingestion pattern** for performance and scalability needs. 

### 3.1 Determine high-performing and scalable storage solutions

<details>
<summary>Choose between S3, EBS, EFS, and FSx for performance</summary>

* **Amazon S3 (object):**

  * Massive scalability, virtually unlimited objects.
  * Great for static content, data lakes, backups, logs.
* **EBS (block):**

  * Attached to EC2; choose volume types (gp3, io1/io2, st1, sc1).
* **EFS (file):**

  * Scalable, shared file system; NFS; multi-AZ.
* **FSx (file):**

  * High-performance or specialized filesystems (Windows File Server, Lustre, NetApp ONTAP, OpenZFS). 

Exam mental shortcuts:

* “High IOPS, low latency storage for DB on EC2” → **EBS io1/io2**.
* “Shared POSIX file system for many Linux instances/containers” → **EFS**.
* “Windows file shares with SMB” → **FSx for Windows**.
* “HPC with massive throughput” → **FSx for Lustre**.

</details>

<details>
<summary>Scale storage over time (capacity and throughput)</summary>

Examples:

* **EBS gp3:** Independent tuning of IOPS and throughput from capacity.
* **EFS:** Scales automatically with usage; throughput modes (bursting vs provisioned).
* **S3:** Effectively infinite; scale via **multi-part upload**, request parallelism, and proper prefixing (modern S3 scales without aggressive prefix design, but parallelism still matters under heavy load).

Exam patterns:

* “Need shared storage that automatically grows with data” → **EFS**.
* “Need high throughput for big data workloads” → **FSx for Lustre** or **S3 + EMR/Athena**.

</details>

---

### 3.2 Design high-performing and elastic compute solutions

<details>
<summary>Select instance families, Auto Scaling, and elasticity options</summary>

**Instance families (high-level):**

* **General purpose (T/M):** balanced CPU, memory, networking.
* **Compute optimized (C):** CPU-bound workloads.
* **Memory optimized (R/X):** in-memory DBs, analytics.
* **Storage optimized (I/D):** high sequential read/write, IOPS.

**Elasticity:**

* **EC2 Auto Scaling** (per ASG).
* **AWS Auto Scaling** (across multiple resource types). 

Exam patterns:

* “Variable workload, need scale out and in” → **Auto Scaling group**.
* “Batch workload to run on spot instances” → **EC2 Spot + ASG** or **AWS Batch**.
* “Event-driven, per-request scaling” → **Lambda**.

</details>

<details>
<summary>Decouple and scale components independently</summary>

Patterns:

* Web tier behind **ALB**.
* App tier in **ASG or Lambda**.
* Async processing with **SQS** or **Kinesis**.
* DB tier using **read replicas** for read scaling; **ElastiCache** for caching.

Exam clue: if performance issue is **DB read heavy**, correct answer often adds **read replicas and/or caching** instead of simply making instance bigger.

</details>

---

### 3.3 Determine high-performing database solutions

<details>
<summary>Choose the right database service: RDS, Aurora, DynamoDB, ElastiCache, Redshift</summary>

Quick mapping:

* **RDS (MySQL/Postgres/SQL Server/etc.):** managed relational DB for standard OLTP.
* **Aurora:** cloud-optimized relational DB, better performance/scaling.
* **DynamoDB:** NoSQL key-value, millisecond latency at any scale.
* **ElastiCache (Redis/Memcached):** in-memory cache for ultra-low latency.
* **Redshift:** columnar data warehouse for analytics. 

Exam patterns:

* “Unpredictable throughput, NoSQL, need global scale” → **DynamoDB on-demand, Global Tables**.
* “MySQL app that needs near drop-in with better availability/performance” → **Aurora**.
* “Read-heavy relational workload” → **read replicas** + **ElastiCache**.
* “Complex analytical queries over TB+ of data” → **Redshift** or **Athena over S3**.

</details>

<details>
<summary>Use replicas, caching, and proxies to scale DB performance</summary>

* **Read replicas (RDS/Aurora/DynamoDB DAX/Global Tables):**

  * Offload read traffic.
* **ElastiCache:**

  * Cache hot keys or full pages/query results.
* **RDS Proxy:**

  * Connection pooling for spiky workloads (e.g., Lambda). 

On the exam:

* “DB CPU high due to many connections from Lambda” → add **RDS Proxy**.
* “Slow reads but low writes” → add **read replicas** and/or **ElastiCache**.

</details>

---

### 3.4 Determine high-performing network architectures

<details>
<summary>Use CloudFront, Global Accelerator, and proper network design</summary>

* **Amazon CloudFront:**

  * CDN; caches content at edge; reduces latency, offloads origin.
* **AWS Global Accelerator:**

  * Anycast IPs; improves performance for non-HTTP protocols and multi-Region apps.
* **VPC design:**

  * Separate subnets by tier (public vs private).
  * Plan CIDR ranges and routing.

Exam patterns:

* “Improve performance of global web app delivering static/dynamic content” → **CloudFront**.
* “Non-HTTP, multi-Region TCP app, need consistent latency” → **Global Accelerator**.
* “Need private connectivity to AWS services” → **VPC endpoints (Interface/Gateway)**. 

</details>

---

### 3.5 Determine high-performing data ingestion and transformation solutions

<details>
<summary>Choose streaming vs batch data services (Kinesis, DataSync, Glue, Athena, EMR)</summary>

* **Kinesis Data Streams / Firehose:**

  * Real-time or near-real-time streaming ingestion. 
* **AWS DataSync / Transfer Family / Storage Gateway:**

  * Moving large datasets or hybrid transfer.
* **AWS Glue:**

  * ETL – catalog and transform data (e.g., CSV → Parquet).
* **Amazon Athena:**

  * Serverless SQL on S3 data.
* **EMR:**

  * Managed big data frameworks (Spark, Hadoop).

Exam patterns:

* “Continuous streaming sensor data, need near-real-time analytics” → **Kinesis** + **Lambda/Firehose**.
* “One-time or scheduled bulk transfer from on-prem to S3” → **DataSync**.
* “Query data in S3 with SQL, no infra to manage” → **Athena**.

</details>

---

## Domain 4: Design Cost-Optimized Architectures (20%)

Domain 4 tests your ability to **control costs** while still meeting business requirements for performance and availability. 

### 4.1 Design cost-optimized storage solutions

<details>
<summary>Pick the right S3 storage class and lifecycle policy</summary>

Common S3 classes:

* **Standard:** frequent access, low latency.
* **Standard-IA / One Zone-IA:** infrequent access; lower cost with retrieval fee.
* **Intelligent-Tiering:** moves objects between tiers automatically.
* **Glacier / Glacier Deep Archive:** archival; hours-to-retrieve.

Exam patterns:

* “Long-term archive, rarely accessed, lowest cost” → **Glacier Deep Archive**.
* “Unknown/variable access patterns” → **Intelligent-Tiering**.
* Use **lifecycle policies** to automatically move older data to cheaper tiers. 

</details>

<details>
<summary>Optimize storage transfer and hybrid options</summary>

* Use **multipart uploads** and batching to reduce overhead.
* For hybrid:

  * **Storage Gateway** (file/volume/tape) for on-prem integration.
  * **DataSync** for accelerated, scheduled transfers.
* Consider **Requester Pays** S3 buckets where consumers pay data transfer costs.

Exam pattern: “Migrate 100 TB from on-prem to S3 as cheaply as possible but not time-critical” → **Snowball** is likely if network is a bottleneck; for online data movement, **DataSync** can optimize.

</details>

---

### 4.2 Design cost-optimized compute solutions

<details>
<summary>Use pricing models: On-Demand, Reserved Instances, Savings Plans, Spot</summary>

* **On-Demand:** pay by the hour/second; flexible but most expensive.
* **Reserved Instances (RIs):** commit to instance family/Region for 1–3 years.
* **Savings Plans:** commit to dollar spend per hour, more flexible than RIs.
* **Spot Instances:** use spare capacity at steep discount; can be interrupted. 

Exam patterns:

* “Steady, predictable workload for 1–3 years” → **RIs or Savings Plans**.
* “Fault-tolerant batch processing; OK with interruptions” → **Spot**.
* “Unpredictable short-lived workloads” → **On-Demand or serverless**.

</details>

<details>
<summary>Right-size and scale compute for cost</summary>

Techniques:

* Use **Auto Scaling** to avoid over-provisioning.
* **Right-size** instances (monitor CPU, memory, network).
* Move to **containers (ECS/Fargate)** or **Lambda** to increase utilization.
* Use **Compute Optimizer / Trusted Advisor** for recommendations. 

Exam clues:

* “CPU utilization is low but costs are high” → choose answer with **smaller instance type or Auto Scaling**.
* “Development/staging environments idle overnight” → shut down or reduce size; maybe **use smaller purchasing commitments**.

</details>

---

### 4.3 Design cost-optimized database solutions

<details>
<summary>Optimize database cost by choosing the right engine and capacity model</summary>

* **Aurora Serverless / RDS Serverless** options for variable workloads.
* **DynamoDB on-demand vs provisioned**:

  * On-demand: pay per request, great for spiky/unpredictable workloads.
  * Provisioned: cheaper for steady load; plus auto scaling.
* Use **caching (ElastiCache)** to reduce DB load.

Exam patterns:

* “Occasional spikes, unpredictable traffic, want simplified capacity management” → **DynamoDB on-demand** or **Aurora Serverless**. 
* “Read-heavy analytics over rarely updated data” → move to **Redshift / Athena** instead of hitting OLTP DB.

</details>

<details>
<summary>Backup, retention, and data tiering to reduce DB cost</summary>

* Set **appropriate snapshot frequency** and retention for RDS/Aurora.
* Use **shorter retention in expensive storage**, archive older backups to cheaper storage if possible.
* For logs/old data, move from DB to **S3 + Glacier** with lifecycle policies.

Exam clue: If a scenario mentions “too many expensive DB snapshots kept forever”, fix with **tuned backup retention policies**.

</details>

---

### 4.4 Design cost-optimized network architectures

<details>
<summary>Reduce data transfer and NAT gateway costs</summary>

Key ideas:

* **VPC endpoints** (Gateway/Interface) reduce data transfer to public internet and NAT.
* Place **NAT gateways per AZ** for resilience, but consider whether you truly need many (cost vs HA).
* Use **private subnets** for most resources; minimize public IP usage.
* Choose **Direct Connect vs VPN vs internet** based on cost and throughput needs. 

Exam patterns:

* “High NAT Gateway costs from private subnets to S3” → add a **Gateway VPC endpoint for S3**.
* “Expensive cross-Region data transfer between services” → consider **co-locating resources in same Region/AZ** or using **CloudFront/CDN**.

</details>

<details>
<summary>Use AWS cost tools: Cost Explorer, Budgets, Cost & Usage Report</summary>

* **Cost Explorer:** visualize spending over time; identify trends.
* **AWS Budgets:** alert when costs/usage exceed thresholds.
* **Cost & Usage Report (CUR):** detailed line-item billing data (for deep analysis in Athena/Redshift). 

Exam clues:

* “Notify finance team when monthly spend exceeds $X” → **AWS Budgets** alert.
* “Analyze detailed cost drivers across tags, services, accounts” → **CUR + Athena/QuickSight**.

</details>

---

## Final exam prep tips

Use this guide together with:

* The **official exam guide PDF** (which you already have) – treat each task statement as a checklist item. 
* The **AWS certification page** for SAA-C03 (for latest details, FAQs, and prep resources). ([Amazon Web Services, Inc.][1])
* Hands-on labs:

  * Build at least one **3-tier app** on AWS: ALB → EC2/ECS → RDS/Aurora, with S3, CloudFront, and IAM.
  * Practice using **VPCs, security groups, NACLs, endpoints, and peering**.
* Lots of **scenario questions**:

  * For each question ask: “What is the **primary constraint**: security, availability, performance, or cost?” Then eliminate answers that don’t optimize for that.

[1]: https://aws.amazon.com/certification/certified-solutions-architect-associate/ "AWS Certified Solutions Architect – Associate"
