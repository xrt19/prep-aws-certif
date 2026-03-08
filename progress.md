# Progress Tracker — SAA-C03

> Update checklist ini setiap selesai satu topik.

---

## Domain 1 — Design Secure Architectures (30%)

### 1.1 IAM & Identity
- [ ] IAM Users, Groups, Roles, Policies
- [ ] IAM Policy evaluation logic (deny > allow)
- [ ] STS & AssumeRole
- [ ] AWS Organizations & SCPs
- [ ] AWS SSO / IAM Identity Center
- [ ] Resource-based vs Identity-based policies

### 1.2 Data Protection
- [ ] KMS (CMK, data keys, envelope encryption)
- [ ] S3 encryption (SSE-S3, SSE-KMS, SSE-C, client-side)
- [ ] Secrets Manager vs SSM Parameter Store
- [ ] Certificate Manager (ACM)

### 1.3 Network Security
- [ ] Security Groups vs NACLs
- [ ] VPC Endpoints (Gateway vs Interface)
- [ ] AWS WAF, Shield, Firewall Manager
- [ ] GuardDuty, Inspector, Macie
- [ ] CloudTrail & Config

---

## Domain 2 — Design Resilient Architectures (26%)

### 2.1 High Availability & Fault Tolerance
- [ ] Multi-AZ vs Multi-Region
- [ ] EC2 Auto Scaling (policies, lifecycle hooks)
- [ ] Elastic Load Balancing (ALB, NLB, CLB, GWLB)
- [ ] Route 53 routing policies

### 2.2 Decoupling
- [ ] SQS (standard vs FIFO, DLQ, visibility timeout)
- [ ] SNS (topics, fan-out pattern)
- [ ] EventBridge
- [ ] Step Functions

### 2.3 Storage Resilience
- [ ] S3 versioning, replication (CRR, SRR)
- [ ] EBS vs EFS vs FSx
- [ ] RDS Multi-AZ vs Read Replicas
- [ ] Aurora (global database, serverless)

---

## Domain 3 — Design High-Performing Architectures (24%)

### 3.1 Compute
- [ ] EC2 instance types & use cases
- [ ] Lambda (concurrency, cold start, layers)
- [ ] ECS vs EKS vs Fargate
- [ ] Elastic Beanstalk

### 3.2 Storage & Database
- [ ] S3 performance (multipart upload, transfer acceleration, byte-range fetches)
- [ ] DynamoDB (partition key design, GSI, LSI, DAX)
- [ ] ElastiCache (Redis vs Memcached)
- [ ] Redshift

### 3.3 Networking & Caching
- [ ] CloudFront (origins, behaviors, cache policies)
- [ ] Global Accelerator vs CloudFront
- [ ] VPC peering, Transit Gateway
- [ ] Direct Connect vs VPN

---

## Domain 4 — Design Cost-Optimized Architectures (20%)

### 4.1 Compute Cost
- [ ] EC2 pricing (On-Demand, Reserved, Spot, Savings Plans)
- [ ] Spot Instances & Spot Fleet
- [ ] Lambda pricing model
- [ ] Right-sizing

### 4.2 Storage Cost
- [ ] S3 storage classes (Standard, IA, Glacier tiers)
- [ ] S3 Lifecycle policies
- [ ] EBS volume types & cost
- [ ] Data transfer costs

### 4.3 Architecture Patterns
- [ ] Serverless vs always-on
- [ ] Reserved capacity planning
- [ ] Cost Explorer & Budgets
- [ ] Trusted Advisor

---

## Practice Exams

| Set | Score | Tanggal | Catatan |
|-----|-------|---------|---------|
| Tutorials Dojo — Set 1 | | | |
| Tutorials Dojo — Set 2 | | | |
| Tutorials Dojo — Set 3 | | | |
| Tutorials Dojo — Set 4 | | | |
| Tutorials Dojo — Set 5 | | | |
| AWS Official Practice | | | |

> Target: konsisten di atas 80% sebelum jadwal ujian.
