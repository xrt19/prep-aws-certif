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
| Secure Architectures (30%) | 75 | 60 | 15 |
| Resilient Architectures (26%) | 65 | 0 | 65 |
| High-Performing (24%) | 60 | 0 | 60 |
| Cost-Optimized (20%) | 50 | 0 | 50 |
| Mixed | 0 | 0 | 0 |
| **Total** | **250** | **60** | **190** |

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
| 07 | Belum | |

### Resilient Architectures (Set 08–13)

| Set | Status | Topik Soal |
|-----|--------|-----------|
| 08 | Belum | |
| 09 | Belum | |
| 10 | Belum | |
| 11 | Belum | |
| 12 | Belum | |
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

### Resilient Architectures
*(belum ada)*

### High-Performing Architectures
*(belum ada)*

### Cost-Optimized Architectures
*(belum ada)*

### Mixed
*(belum ada)*
