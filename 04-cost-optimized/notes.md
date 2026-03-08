# Domain 4 — Design Cost-Optimized Architectures

> Bobot: **20%**

---

## EC2 Pricing Models

| Model | Deskripsi | Savings | Use Case |
|-------|-----------|---------|---------|
| **On-Demand** | Pay per hour/second | - (baseline) | Short-term, unpredictable |
| **Reserved (1yr)** | Commit 1 tahun | ~40% | Steady-state workloads |
| **Reserved (3yr)** | Commit 3 tahun | ~60% | Long-term, stable |
| **Savings Plans** | Commit $ per hour | ~66% | Flexible (EC2 + Fargate + Lambda) |
| **Spot** | Bid untuk unused capacity | ~90% | Fault-tolerant, flexible timing |
| **Dedicated Host** | Physical server | - | Compliance, licensing |

### Spot Instance Tips

- Bisa di-interrupt dengan 2 menit notice
- Gunakan untuk: batch jobs, data processing, CI/CD, stateless web tier
- **Spot Fleet** — kombinasi instance types untuk maximize availability

---

## S3 Storage Classes

| Class | Availability | Retrieval | Use Case |
|-------|-------------|-----------|---------|
| **S3 Standard** | 99.99% | Instant | Frequently accessed |
| **S3 Standard-IA** | 99.9% | Instant | Infrequent access, min 30 hari |
| **S3 One Zone-IA** | 99.5% | Instant | Infrequent, non-critical |
| **S3 Glacier Instant** | 99.9% | Instant | Archive, accessed quarterly |
| **S3 Glacier Flexible** | 99.99% | Menit-jam | Archive, flexible retrieval |
| **S3 Glacier Deep Archive** | 99.99% | 12-48 jam | Long-term archive, min 180 hari |
| **S3 Intelligent-Tiering** | 99.9% | Instant | Unknown/changing access patterns |

### Lifecycle Policies

```
Standard → Standard-IA (setelah 30 hari)
         → Glacier Flexible (setelah 90 hari)
         → Deep Archive (setelah 180 hari)
         → Expire/Delete
```

> **Exam tip:** S3 Intelligent-Tiering cocok ketika pola akses **tidak diketahui** — AWS otomatis pindahkan object antar tier.

---

## EBS Volume Types

| Tipe | IOPS | Use Case |
|------|------|---------|
| **gp3** | Up to 16,000 | General purpose (default) |
| **gp2** | Up to 16,000 | General purpose (older) |
| **io1/io2** | Up to 64,000 | High IOPS (databases) |
| **st1** | Throughput-focused | Big data, log processing |
| **sc1** | Cheapest | Cold data, infrequent access |

> **Exam tip:** `gp3` lebih murah dari `gp2` dan IOPS-nya bisa di-scale independen.

---

## Serverless Cost Advantage

### Lambda vs EC2 Cost Pattern

- Lambda: bayar per invocation + duration (gratis jika tidak ada traffic)
- EC2: bayar per jam meskipun idle

> Gunakan serverless untuk workload dengan traffic yang **sporadic / unpredictable**.

---

## Cost Management Tools

| Tool | Fungsi |
|------|--------|
| **Cost Explorer** | Visualisasi & analisis pengeluaran historis |
| **AWS Budgets** | Set budget + alert jika mendekati/melebihi |
| **Trusted Advisor** | Rekomendasi cost, security, performance, reliability |
| **Compute Optimizer** | Rekomendasi right-sizing EC2, Lambda, EBS |
| **Savings Plans Recommendations** | Rekomendasi commitment berdasarkan usage |

---

## Architecture Patterns untuk Cost

### Serverless Pattern
```
User → API Gateway → Lambda → DynamoDB
                            → S3
```
- Bayar per request, tidak ada idle cost
- Cocok untuk: APIs, event-driven, low-to-medium traffic

### Auto Scaling Pattern
```
ALB → ASG (min: 1, desired: 2, max: 10)
```
- Scale down saat traffic rendah
- Gunakan Spot untuk non-critical layers

### Data Tiering Pattern
```
Hot data   → DynamoDB / ElastiCache
Warm data  → RDS / Aurora
Cold data  → S3 Standard-IA / Glacier
```
