# Domain 2 — Design Resilient Architectures

> Bobot: **26%**

---

## High Availability Concepts

### Multi-AZ vs Multi-Region

| | Multi-AZ | Multi-Region |
|-|----------|-------------|
| **Tujuan** | High Availability (HA) | Disaster Recovery (DR) |
| **Failover** | Otomatis (detik-menit) | Manual / semi-otomatis |
| **Latency** | Rendah (same region) | Lebih tinggi |
| **Cost** | Moderate | Tinggi |
| **Use case** | Production workloads | Compliance, DR requirements |

### RTO & RPO

- **RPO (Recovery Point Objective)** — berapa banyak data yang boleh hilang (time)
- **RTO (Recovery Time Objective)** — berapa lama sistem boleh down

---

## Elastic Load Balancing

| Tipe | Layer | Use Case |
|------|-------|----------|
| **ALB** | L7 (HTTP/HTTPS) | Web apps, microservices, path/host routing |
| **NLB** | L4 (TCP/UDP) | Ultra-low latency, static IP, gaming, IoT |
| **GWLB** | L3 | 3rd party virtual appliances (firewall, IDS) |

> **Exam tip:** NLB bisa assign **Elastic IP** → cocok jika client butuh whitelist IP tertentu.

### ALB Features

- Path-based routing (`/api/*` → service A)
- Host-based routing (`api.example.com` → service A)
- Weighted target groups (canary deployment)
- Sticky sessions (via cookies)

---

## Auto Scaling

### Scaling Policies

| Tipe | Cara Kerja |
|------|-----------|
| **Target Tracking** | Maintain metric di target value (simplest) |
| **Step Scaling** | Scale berdasarkan alarm threshold steps |
| **Scheduled** | Scale pada waktu tertentu (predictable traffic) |
| **Predictive** | ML-based, forecast traffic |

### Lifecycle Hooks

```
Pending → Pending:Wait → Pending:Proceed → InService
InService → Terminating:Wait → Terminating:Proceed → Terminated
```

Digunakan untuk: install software, drain connections, backup data sebelum terminate.

---

## Decoupling: SQS

### Standard vs FIFO

| | Standard | FIFO |
|-|----------|------|
| **Throughput** | Unlimited | 300 msg/s (3000 with batching) |
| **Ordering** | Best-effort | Guaranteed |
| **Delivery** | At-least-once | Exactly-once |
| **Use case** | High throughput | Order-sensitive processing |

### Penting untuk Exam

- **Visibility Timeout** — waktu consumer "memegang" message, default 30s
- **Dead Letter Queue (DLQ)** — message yang gagal diproses setelah N attempts
- **Long Polling** — kurangi empty responses, hemat cost (`WaitTimeSeconds` > 0)
- **Message retention** — 1 menit hingga 14 hari (default 4 hari)

---

## Route 53 Routing Policies

| Policy | Use Case |
|--------|----------|
| **Simple** | Single resource, no health check routing |
| **Weighted** | A/B testing, gradual migration |
| **Failover** | Active-passive DR |
| **Latency** | Route ke region dengan latency terendah |
| **Geolocation** | Route berdasarkan lokasi user (compliance) |
| **Geoproximity** | Route berdasarkan proximity + bias |
| **Multivalue Answer** | Simple load balancing dengan health checks |

---

## Database Resilience

### RDS

- **Multi-AZ** — synchronous replication, otomatis failover, untuk HA (bukan read scaling)
- **Read Replica** — asynchronous replication, untuk read scaling (bisa jadi cross-region)

> **Exam tip:** Read Replica bisa di-promote jadi standalone DB (untuk DR). Multi-AZ standby tidak bisa di-query langsung.

### Aurora

- Shared storage across 6 copies di 3 AZ (otomatis)
- **Aurora Global Database** — replikasi cross-region < 1 detik, untuk DR
- **Aurora Serverless v2** — auto-scale capacity, cocok untuk variable workload
