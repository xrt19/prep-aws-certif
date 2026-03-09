# Tracking — Soal Latihan SAA-C03

> Total target: **250 soal** | 10 soal per set | 25 set

---

## Cara Lanjut Setelah Token Reset

Jika sesi terhenti di tengah jalan, gunakan prompt ini di sesi baru:

```
Lanjutkan membuat soal latihan SAA-C03. 
Baca soal-latihan secara keseluruhan (untuk tahu mana yang sudah dan belum), 
kemudian baca juga soal-latihan/tracking.md
untuk tahu set mana yang harus dibuat berikutnya dan topik apa yang sudah
dipakai. Lanjutkan dari set yang statusnya "In Progress" dulu, atau set
berikutnya yang statusnya "Belum". Setelah selesai, update tracking.md.
```

---

## Ringkasan Progress

| Domain | Target | Dibuat | Sisa |
|--------|--------|--------|------|
| Secure Architectures (30%) | 75 | 75 | 0 |
| Resilient Architectures (26%) | 65 | 50 | 15 |
| High-Performing (24%) | 60 | 0 | 60 |
| Cost-Optimized (20%) | 50 | 0 | 50 |
| Mixed | 0 | 0 | 0 |
| **Total** | **250** | **125** | **125** |

---

## Detail Per Set

### Secure Architectures (Set 01–07)

| Set | Status | Topik Soal |
|-----|--------|-----------|
| 01 | Selesai | IAM Identity Center/SSO, credential compromise, IAM Role EC2, S3 Object Lock, AWS Config, WAF, SCP Organizations, Secrets Manager rotation, CloudTrail, VPC Gateway Endpoint |
| 02 | Selesai | KMS CMK + EBS encryption, SCP tag enforcement, Cognito Identity Pool S3 access, Database Activity Streams, GuardDuty, IAM SourceIp condition, Interface VPC Endpoints, Security Hub, Lambda least privilege, IAM Access Analyzer |
| 03 | Selesai | Amazon Macie PII discovery, NACL vs SG explicit deny, ACM auto-renewal, Cross-account IAM Role, Permission Boundaries, S3 Block Public Access, Network Firewall DPI, IAM policy evaluation order, Lambda resource-based policy, ALB HTTP→HTTPS redirect |
| 04 | Selesai | AWS Shield Standard/Advanced, SSM Session Manager logging, AWS RAM resource sharing, S3 pre-signed URL, Amazon Inspector CVE scan, AWS Firewall Manager, KMS key policy precedence, Cognito User Pool + Identity Pool, Config auto-remediation SSM, RDS encryption at rest immutable |
| 05 | Selesai | VPC Flow Logs, S3 Access Points, IRSA EKS, Envelope encryption + KMS DEK, GuardDuty delegated admin, SSM Parameter Store SecureString/tiers, CloudTrail multi-region trail, AWS Artifact compliance reports, S3 Block Public Access level bucket, EFS default encryption + Config rule |
| 06 | Selesai | AWS Control Tower landing zone, S3 Glacier Vault Lock, Amazon Detective incident investigation, Customer Managed vs Inline Policy, STS session policy, SG referencing cross-VPC, WAF rate-based rules, S3 Object Ownership bucket owner enforced, Trusted Advisor security checks, Multi-tier SG chaining |
| 07 | Selesai | AWS Private CA, AWS Verified Access, CORS, AD Connector vs Managed AD, IAM role chaining 1 jam, CloudFront Signed URLs/Cookies, Gateway+Interface Endpoints, Service-Linked Role, VPC Peering vs TGW, multi-service security mapping, CloudTrail org trail + S3 Lock, KMS audit via CloudTrail, Lambda least privilege per-ARN, Config SG-attached-to-ENI, Config access-keys-rotated |

### Resilient Architectures (Set 08–13)

| Set | Status | Topik Soal |
|-----|--------|-----------|
| 08 | Selesai | RDS Multi-AZ failover, Scheduled scaling, SQS Visibility Timeout, Route 53 latency-based routing, ASG self-healing health checks, RDS PITR, S3 CRR, ALB vs NLB vs CLB, DR strategies RTO/RPO, SQS decoupling burst absorption |
| 09 | Selesai | RDS Proxy connection pooling, Aurora auto-scaling storage + failover, SNS fan-out pattern, SQS FIFO ordering + exactly-once, ASG Lifecycle Hooks, Aurora Global Database cross-region, Route 53 Failover routing + health check, Kinesis Data Streams replay, Step Functions orchestration, ElastiCache caching layer |
| 10 | Selesai | ECS Service self-healing Fargate, DynamoDB Global Tables multi-active, ALB weighted target groups blue/green, NLB Elastic IP static, DynamoDB Streams event-driven, Aurora Serverless v2, EventBridge content-based routing, API Gateway caching, S3 SRR vs CRR, Step scaling asymmetric scale |
| 11 | Selesai | SQS Long Polling reduce empty responses, Kinesis Firehose managed delivery S3, ASG cooldown period warm-up, DynamoDB DAX microsecond latency, Elastic Beanstalk Rolling deployment, EFS regional multi-AZ by default, CloudFront Origin Groups failover, Route 53 Geolocation routing, RDS Storage Auto Scaling, ECS Capacity Providers Cluster scaling |
| 12 | Selesai | ElastiCache Redis Multi-AZ failover, Lambda Reserved vs Provisioned Concurrency, FSx for Lustre HPC, AWS Elastic Disaster Recovery, AWS Backup centralized, DynamoDB On-Demand spiky workloads, FSx for Windows SMB AD, AWS Fault Injection Service chaos engineering, Route 53 weighted + health checks canary, Transit Gateway Peering inter-region |
| 13 | Belum | |

### High-Performing Architectures (Set 14–19)

| Set | Status | Topik Soal |
|-----|--------|-----------|
| 14 | Belum | |
| 15 | Belum | |
| 16 | Belum | |
| 17 | Belum | |
| 18 | Belum | |
| 19 | Belum | |

### Cost-Optimized Architectures (Set 20–24)

| Set | Status | Topik Soal |
|-----|--------|-----------|
| 20 | Belum | |
| 21 | Belum | |
| 22 | Belum | |
| 23 | Belum | |
| 24 | Belum | |

### Mixed (Set 25)

| Set | Status | Topik Soal |
|-----|--------|-----------|
| 25 | Belum | |

---

## Topik Yang Sudah Dipakai

> Diupdate setiap set selesai dibuat. Digunakan untuk menghindari duplikasi soal antar set.

### Secure Architectures
**Set 01:** IAM Identity Center/SSO, credential compromise response, IAM Role untuk EC2, S3 Object Lock Compliance mode, AWS Config change detection, AWS WAF SQLi/XSS, SCP Organizations, Secrets Manager auto-rotation, CloudTrail audit trail, VPC Gateway Endpoint S3

**Set 02:** KMS CMK + EBS encryption + key rotation, SCP tag enforcement (`aws:RequestTag`), Cognito Identity Pool + identity-based S3 access, Database Activity Streams RDS, GuardDuty threat detection, IAM `aws:SourceIp` condition, Interface VPC Endpoints PrivateLink, Security Hub findings aggregation, Lambda execution role least privilege, IAM Access Analyzer unused access

**Set 03:** Amazon Macie PII/sensitive data discovery, NACL explicit deny vs Security Groups, ACM auto-renewal + ALB, cross-account IAM Role + STS AssumeRole, IAM Permission Boundaries, S3 Block Public Access level akun, AWS Network Firewall DPI + Suricata, IAM policy evaluation order (Explicit Deny), Lambda resource-based policy cross-account, ALB HTTP→HTTPS redirect listener rule

**Set 04:** AWS Shield Standard vs Advanced, SSM Session Manager session logging, AWS RAM resource sharing, S3 pre-signed URL time-limited access, Amazon Inspector vulnerability assessment, AWS Firewall Manager centralized WAF, KMS key policy vs IAM policy precedence, Cognito User Pool + Identity Pool combined, AWS Config auto-remediation + SSM Automation, RDS encryption immutable at launch

**Set 05:** VPC Flow Logs network traffic metadata, S3 Access Points multi-tenant, IRSA IAM Roles for Service Accounts EKS, envelope encryption + KMS DEK 4KB limit, GuardDuty delegated administrator multi-account, SSM Parameter Store SecureString + Standard/Advanced tier, CloudTrail multi-region trail, AWS Artifact compliance reports ISO/SOC, S3 Block Public Access level bucket, EFS default encryption + Config rule defense-in-depth

**Set 06:** AWS Control Tower landing zone + account factory, S3 Glacier Vault Lock WORM compliance, Amazon Detective incident investigation graph analysis, Customer Managed Policy vs Inline Policy reusability, STS session policy AssumeRole effective permissions intersection, Security Group cross-VPC referencing, WAF rate-based rules credential stuffing, S3 Object Ownership bucket owner enforced, AWS Trusted Advisor security checks, multi-tier SG chaining defense in depth

**Set 07:** AWS Private CA internal TLS mTLS, AWS Verified Access zero-trust tanpa VPN, CORS cross-origin API Gateway/S3, AD Connector vs AWS Managed Microsoft AD, IAM role chaining 1 jam session limit, CloudFront Signed URLs vs Signed Cookies, Gateway Endpoints + Interface Endpoints kombinasi, Service-Linked Role AWS service permissions, VPC Peering non-transitive vs Transit Gateway, multi-service security architecture mapping, CloudTrail organization trail + S3 Object Lock Log Archive, KMS API calls audit via CloudTrail, Lambda execution role least privilege per-ARN, Config rule ec2-security-group-attached-to-eni, Config rule access-keys-rotated 90 hari

### Resilient Architectures
**Set 08:** RDS Multi-AZ automatic failover, scheduled scaling proactive, SQS Visibility Timeout long-running processing, Route 53 latency-based routing + health checks, ASG self-healing terminate & replace, RDS Point-in-Time Recovery, S3 Cross-Region Replication, ALB vs NLB vs CLB Layer 7 path routing, DR strategies RTO/RPO Multi-Site Active-Active, SQS decoupling burst absorption 14-day retention

**Set 09:** RDS Proxy connection pooling Lambda, Aurora auto-scaling storage + failover < 30 detik, SNS fan-out one-to-many pattern, SQS FIFO ordering + exactly-once MessageGroupId, ASG Lifecycle Hooks pre-terminate custom action, Aurora Global Database cross-region < 1 detik lag, Route 53 Failover routing + health check requirement, Kinesis Data Streams replay multiple consumers, Step Functions workflow orchestration state machine, ElastiCache caching layer sub-millisecond (Multi-AZ tidak serve reads)

**Set 10:** ECS Service desired count self-healing Fargate, DynamoDB Global Tables multi-region multi-active, ALB weighted target groups blue/green canary, NLB Elastic IP static TCP Layer 4, DynamoDB Streams event-driven Lambda trigger, Aurora Serverless v2 near-zero idle, EventBridge content-based routing event bus, API Gateway caching TTL reduce Lambda invocations, S3 SRR vs CRR same-region replication, Step scaling multi-step asymmetric scale out vs scale in

**Set 11:** SQS Long Polling WaitTimeSeconds reduce empty responses, Kinesis Data Firehose managed delivery transform S3, ASG cooldown period instance warm-up, DynamoDB DAX microsecond latency hot items, Elastic Beanstalk Rolling deployment no downtime, EFS regional redundancy multi-AZ by default, CloudFront Origin Groups origin failover, Route 53 Geolocation routing country-level default record, RDS Storage Auto Scaling no downtime, ECS Capacity Providers Cluster Auto Scaling EC2 infrastructure

**Set 12:** ElastiCache Redis Multi-AZ automatic failover, Lambda Reserved Concurrency vs Provisioned Concurrency, FSx for Lustre HPC parallel file system, AWS Elastic Disaster Recovery continuous replication on-premise, AWS Backup centralized multi-service, DynamoDB On-Demand vs Provisioned spiky workloads, FSx for Windows File Server SMB NTFS AD integration, AWS Fault Injection Service chaos engineering, Route 53 weighted routing + health checks canary failover, Transit Gateway Peering inter-region hub-and-spoke

### High-Performing Architectures
*(belum ada)*

### Cost-Optimized Architectures
*(belum ada)*

### Mixed
*(belum ada)*
