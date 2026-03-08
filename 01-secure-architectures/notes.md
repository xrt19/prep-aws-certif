# Domain 1 — Design Secure Architectures

> Bobot: **30%** — domain terbesar, prioritas utama.

---

## IAM Fundamentals

### Komponen Utama

| Komponen | Deskripsi |
|----------|-----------|
| **User** | Identitas untuk manusia / aplikasi (long-term credentials) |
| **Group** | Kumpulan users, policies attach ke group |
| **Role** | Identitas sementara, di-assume oleh services/users (short-term credentials via STS) |
| **Policy** | JSON document yang define permissions |

### Policy Evaluation Logic

```
1. Explicit DENY → selalu menang
2. Explicit ALLOW → diizinkan (jika tidak ada deny)
3. Default → DENY (implicit deny)
```

> **Exam tip:** Jika ada Explicit Deny di salah satu policy (identity-based maupun resource-based), action tersebut **pasti ditolak** meskipun ada Allow di tempat lain.

### Policy Types

- **Identity-based policy** — attach ke user/group/role
- **Resource-based policy** — attach ke resource (S3 bucket policy, SQS policy, dll)
- **SCP (Service Control Policy)** — attach ke AWS Organizations OU, set maximum permissions
- **Permission boundary** — set maximum permissions untuk IAM entity
- **Session policy** — dipass saat AssumeRole

### Cross-Account Access Pattern

```
Account A (user) → AssumeRole → Account B (role) → access resources
```

---

## KMS & Encryption

### Envelope Encryption

```
Data → encrypted dengan Data Key (DEK)
DEK  → encrypted dengan CMK (Customer Master Key) di KMS
```

Hanya ciphertext DEK yang disimpan, plaintext DEK tidak pernah persist.

### S3 Encryption Options

| Tipe | Key Management | Catatan |
|------|---------------|---------|
| SSE-S3 | AWS manages | Default, paling simple |
| SSE-KMS | KMS (bisa CMK) | Audit trail di CloudTrail, kontrol lebih |
| SSE-C | Customer provides key | AWS tidak simpan key |
| Client-side | Customer | Encrypt sebelum upload |

> **Exam tip:** SSE-KMS memberikan audit trail per-object access via CloudTrail.

---

## Secrets Manager vs SSM Parameter Store

| | Secrets Manager | SSM Parameter Store |
|-|----------------|---------------------|
| **Biaya** | Ada biaya per secret | Free (Standard tier) |
| **Auto rotation** | ✅ Native (Lambda) | ❌ Manual |
| **Use case** | DB credentials, API keys | Config, non-secret params |
| **Cross-account** | ✅ | ✅ (terbatas) |

---

## Network Security

### Security Groups vs NACLs

| | Security Group | NACL |
|-|---------------|------|
| Level | Instance (ENI) | Subnet |
| State | **Stateful** | **Stateless** |
| Rules | Allow only | Allow & Deny |
| Evaluation | All rules evaluated | Rules evaluated in order (number) |

> **Exam tip:** NACL stateless = harus define inbound AND outbound rules secara eksplisit.

### VPC Endpoints

- **Gateway Endpoint** — S3 dan DynamoDB saja, free, route table entry
- **Interface Endpoint (PrivateLink)** — service lainnya, ada biaya, pakai ENI

### Security Services

| Service | Fungsi |
|---------|--------|
| **GuardDuty** | Threat detection (ML-based, analisis CloudTrail/VPC Flow/DNS logs) |
| **Inspector** | Vulnerability assessment untuk EC2 & container images |
| **Macie** | Discover & protect sensitive data di S3 (PII detection) |
| **WAF** | Web Application Firewall (layer 7, attach ke ALB/CloudFront/API GW) |
| **Shield Standard** | DDoS protection, free, otomatis |
| **Shield Advanced** | DDoS protection enhanced, ada biaya, 24/7 DRT support |
