# Practice Questions — Common Patterns & Traps

> Kumpulan soal tipikal SAA-C03 beserta pola jawaban yang benar.

---

## Pattern 1: "Least Operational Overhead"

**Keyword ini selalu mengarah ke:**
- Serverless (Lambda, Fargate, Aurora Serverless)
- Managed services (RDS vs self-managed DB di EC2)
- Fargate over ECS on EC2

**Contoh soal:**
> A company wants to run containerized microservices with **minimum operational overhead**. Which solution should they use?

**Jawaban:** ECS dengan Fargate (bukan EC2 launch type).

---

## Pattern 2: "Cost-Effective"

**Keyword ini selalu mengarah ke:**
- Spot Instances untuk fault-tolerant workloads
- S3 Lifecycle policies + Glacier untuk archive
- Reserved Instances / Savings Plans untuk steady-state
- Serverless untuk sporadic workloads

---

## Pattern 3: "High Availability"

**Keyword ini selalu mengarah ke:**
- Multi-AZ deployment
- Auto Scaling Group
- ALB/NLB
- RDS Multi-AZ

---

## Pattern 4: "Decouple / Loose Coupling"

**Keyword ini selalu mengarah ke:**
- SQS (queue between producers & consumers)
- SNS (fan-out)
- EventBridge (event-driven)

---

## Pattern 5: Soal Klasik yang Sering Muncul

### S3 + Athena
> Analyze log files stored in S3 without loading into a database?
→ **Athena** (serverless SQL on S3)

### Cross-Region Replication
> DR solution untuk S3 di region lain?
→ **S3 CRR (Cross-Region Replication)**

### Static Website
> Host static website paling murah?
→ **S3 Static Website Hosting** + CloudFront (opsional untuk HTTPS/performance)

### Session State
> Store user session untuk horizontally-scaled web app?
→ **ElastiCache (Redis)** atau **DynamoDB**

### On-Premise to AWS Migration
> Migrate data 50TB ke AWS dengan koneksi internet terbatas?
→ **AWS Snowball Edge**

### Prevent Accidental S3 Delete
> Protect S3 objects dari accidental deletion?
→ **S3 Versioning** + **MFA Delete**

### API Throttling
> Protect backend dari traffic spike di API?
→ **API Gateway** dengan throttling + **SQS** untuk buffer

### Hybrid DNS
> Resolve DNS on-premise untuk resources di VPC?
→ **Route 53 Resolver** (Inbound/Outbound endpoints)

---

## Common Traps

| Trap | Yang Benar |
|------|-----------|
| "Read Replica untuk HA" | ❌ Read Replica untuk **read scaling**. Untuk HA → Multi-AZ |
| "NACLs stateful" | ❌ NACLs **stateless**. Security Groups yang stateful |
| "VPC Peering transitive" | ❌ VPC Peering **tidak transitive**. Gunakan Transit Gateway |
| "S3 strong consistency" | ✅ Sejak 2020, S3 sudah **strong read-after-write consistency** |
| "GuardDuty untuk vulnerability scan" | ❌ GuardDuty untuk **threat detection**. Inspector untuk vulnerability scan |
| "EBS bisa multi-attach ke banyak instance" | Hanya `io1/io2` yang support Multi-Attach, dan terbatas use case |
