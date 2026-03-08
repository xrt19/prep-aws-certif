# Domain 3 — Design High-Performing Architectures

> Bobot: **24%**

---

## EC2 Instance Types

| Family | Optimized For | Contoh Use Case |
|--------|--------------|-----------------|
| **M** (General) | Balanced | Web servers, app servers |
| **C** (Compute) | CPU | Batch processing, media encoding |
| **R** (Memory) | RAM | In-memory DB, real-time analytics |
| **I** (Storage) | IOPS | NoSQL DB, data warehousing |
| **G / P** (GPU) | GPU | ML training, video rendering |
| **T** (Burstable) | Baseline + burst | Dev/test, low-traffic apps |

---

## Lambda Performance

### Concurrency

- **Reserved Concurrency** — garantikan dan limit fungsi (jangan rebutan dengan fungsi lain)
- **Provisioned Concurrency** — pre-warm instances, eliminasi cold start

### Cold Start Mitigation

1. Provisioned Concurrency (paling efektif)
2. Kurangi deployment package size
3. Hindari VPC jika tidak perlu (VPC cold start lebih lama)
4. Gunakan Lambda Layers untuk dependencies besar

---

## Container Services

| Service | Deskripsi |
|---------|-----------|
| **ECS (EC2)** | Manage container di EC2 yang kamu manage |
| **ECS (Fargate)** | Serverless container, tidak manage EC2 |
| **EKS** | Managed Kubernetes |
| **EKS Fargate** | Serverless Kubernetes |

> **Exam tip:** Jika soal minta "least operational overhead" → pilih Fargate.

---

## S3 Performance

### Upload Optimization

- **Multipart Upload** — recommended untuk file > 100MB, wajib untuk > 5GB
- **S3 Transfer Acceleration** — pakai CloudFront edge locations untuk upload dari jauh

### Download Optimization

- **Byte-Range Fetches** — parallel download bagian-bagian file

### S3 Request Rate

- 3,500 PUT/COPY/POST/DELETE per prefix per detik
- 5,500 GET/HEAD per prefix per detik
- Gunakan multiple prefixes untuk scale lebih tinggi

---

## DynamoDB

### Partition Key Design

> Pilih partition key dengan **high cardinality** agar data terdistribusi merata dan hindari "hot partition".

### Index Types

| | LSI | GSI |
|-|-----|-----|
| **Dibuat** | Saat create table | Kapan saja |
| **Partition key** | Sama dengan table | Bisa berbeda |
| **Konsistensi** | Strongly/Eventually | Eventually only |
| **Kapasitas** | Sama dengan table | Terpisah |

### DAX (DynamoDB Accelerator)

- In-memory cache untuk DynamoDB
- Latency microseconds (dari milliseconds)
- Write-through cache
- Cocok untuk **read-heavy** workloads

---

## Caching Strategies

### CloudFront vs Global Accelerator

| | CloudFront | Global Accelerator |
|-|-----------|-------------------|
| **Tipe** | CDN (cache content) | Network routing |
| **Cocok untuk** | Static/dynamic content, video | Non-HTTP (TCP/UDP), gaming |
| **Caching** | ✅ | ❌ |
| **IP** | Dynamic | Static (2 Anycast IPs) |

### ElastiCache

| | Redis | Memcached |
|-|-------|-----------|
| **Persistence** | ✅ | ❌ |
| **Replication** | ✅ | ❌ |
| **Data structures** | Complex (lists, sets, etc) | Simple key-value |
| **Multi-threaded** | ❌ | ✅ |

> **Exam tip:** Jika soal menyebut "leaderboard", "session store", atau "pub/sub" → Redis.

---

## Networking

### VPC Connectivity

| Opsi | Use Case |
|------|---------|
| **VPC Peering** | 2 VPC, non-transitive, bisa cross-account/region |
| **Transit Gateway** | Hub-spoke untuk banyak VPC, transitive routing |
| **PrivateLink** | Expose service ke VPC lain secara private |

> **Exam tip:** VPC Peering **tidak transitive** — jika A↔B dan B↔C, A tidak otomatis bisa akses C.

### Direct Connect vs VPN

| | Direct Connect | Site-to-Site VPN |
|-|---------------|-----------------|
| **Koneksi** | Dedicated physical | Internet (encrypted) |
| **Bandwidth** | 1Gbps - 100Gbps | Max ~1.25Gbps |
| **Latency** | Konsisten, rendah | Variable |
| **Setup time** | Minggu-bulan | Menit-jam |
| **Cost** | Mahal | Murah |
