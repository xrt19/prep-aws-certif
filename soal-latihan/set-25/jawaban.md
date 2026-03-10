# Jawaban — Set 25
## Domain: Mixed (Final Set)

---

## Soal 1 — Jawaban: C

**SCP (Service Control Policies) di AWS Organizations**

**Kenapa C:**
SCP adalah **preventive guardrail** paling kuat di AWS Organizations:

- Berlaku untuk **semua principals** di member accounts — users, roles, bahkan **account root**
- Tidak bisa di-override oleh IAM policy di level account
- Bersifat **deny-based** — explicit deny tidak bisa di-override

Contoh SCP untuk protect CloudTrail:
```json
{
  "Effect": "Deny",
  "Action": [
    "cloudtrail:StopLogging",
    "cloudtrail:DeleteTrail",
    "cloudtrail:UpdateTrail"
  ],
  "Resource": "*"
}
```

SCP ini di-attach ke Root OU → berlaku ke semua 50+ accounts sekaligus.

**Kenapa bukan yang lain:**
- **A** — AWS Config rules adalah **detective controls** — detect setelah pelanggaran terjadi. CloudTrail sudah bisa di-stop sebelum Config mendeteksi dan alert. Tidak preventive.
- **B** — Permission Boundaries membatasi **maximum permissions** untuk IAM entities, tapi hanya berlaku untuk principals yang di-manage (tidak berlaku untuk account root yang tidak punya IAM user).
- **D** — Control Tower Guardrails menggunakan SCP dan Config rules di baliknya. SCP adalah mekanisme yang tepat. Control Tower adalah orchestration layer di atas SCP — untuk kontrol langsung, SCP lebih tepat. Tapi secara konsep Control Tower bisa valid — kunci jawaban adalah SCP sebagai mekanisme enforcement.

**Konsep yang diuji:** SCP preventive control, Organizations hierarchy, CloudTrail protection, account root restriction.

---

## Soal 2 — Jawaban: D

**Warm Standby**

**Kenapa D:**
Perbandingan DR strategies:

| Strategy | RTO | RPO | Cost |
|----------|-----|-----|------|
| Backup & Restore | Hours | Hours | Lowest |
| Pilot Light | 10–30 min | Minutes | Low |
| **Warm Standby** | **< 15 min** | **Seconds–minutes** | Medium |
| Multi-Site Active-Active | Near zero | Near zero | Highest |

**Warm Standby** untuk requirement RTO < 15 menit, RPO < 1 menit:
- Scaled-down version of production selalu running di DR region (misalnya 2 small instances vs 10 large di production)
- **Aurora Global Database** untuk replikasi cross-region dengan **lag < 1 detik** → RPO < 1 menit ✓
- Saat failover: promote Aurora secondary ke primary, scale up EC2 fleet (10–15 menit) → RTO < 15 menit ✓
- Lebih murah dari Multi-Site karena DR environment berjalan di kapasitas kecil

**Kenapa bukan yang lain:**
- **A** — Backup and Restore: RTO berjam-jam (restore snapshot + launch instance + warm up). Tidak memenuhi RTO < 15 menit.
- **B** — Pilot Light: hanya database core yang running, application server tidak. Saat failover, perlu launch application servers dari scratch → biasanya RTO 20–30 menit. Borderline untuk RTO < 15 menit, dan RPO bisa lebih dari 1 menit.
- **C** — Multi-Site Active-Active: RTO/RPO terbaik tapi soal menyatakan "biaya harus reasonable, tidak bisa afford Multi-Site Active-Active". Jawaban yang benar harus balance requirement.

**Konsep yang diuji:** DR strategies trade-offs, Warm Standby RTO/RPO, Aurora Global Database replication lag, cost-RTO balance.

---

## Soal 3 — Jawaban: D

**Lambda@Edge atau CloudFront Functions**

**Kenapa D:**
Untuk API calls sederhana yang tidak butuh backend database, memproses di **edge** eliminasi round-trip ke us-east-1:

```
Tanpa edge: User Asia → CloudFront Edge (Asia) → API Gateway us-east-1 → Lambda us-east-1
Latency: ~800ms (300ms network RTT us-east-1 + processing)

Dengan Lambda@Edge: User Asia → CloudFront Edge (Asia) → Lambda@Edge (Asia edge)
Latency: ~20–50ms (diproses lokal di edge)
```

Use cases yang cocok di edge:
- Token validation (verify JWT signature tanpa database call)
- A/B testing header injection
- URL rewriting dan routing
- Static config responses
- Simple data transformation

**Kenapa bukan yang lain:**
- **A** — Deploy di ap-southeast-1: valid untuk mengurangi latency, tapi ini artinya maintain dua deployments (us-east-1 + ap-southeast-1), manage database replication, API Gateway + Lambda di dua regions. Lebih complex dan lebih mahal daripada Lambda@Edge untuk use case sederhana.
- **B** — Global Accelerator menggunakan AWS backbone untuk routing ke endpoint — mengurangi latency dari 800ms menjadi ~500ms (masih ada round-trip ke us-east-1, hanya network path lebih optimal). Tidak sama dramatis dengan edge processing.
- **C** — API Gateway Edge-optimized endpoint menggunakan CloudFront untuk route requests ke API Gateway region. Mengurangi network path sedikit tapi Lambda masih berjalan di us-east-1. Bukan solusi yang optimal.

**Konsep yang diuji:** Lambda@Edge, edge processing for low-latency APIs, CloudFront Functions, Global Accelerator vs edge computing.

---

## Soal 4 — Jawaban: D

**S3 Event Notification → SQS → AWS Batch + Spot Instances**

**Kenapa D:**
Masalah: EC2 t3.large running 24/7 = **$0.0832/jam × 24 jam × 30 hari = ~$60/bulan** untuk menunggu file yang diproses hanya 2 jam per hari.

Dengan AWS Batch + Spot:
- Tidak ada instance idle — Batch provision hanya saat diperlukan
- Spot Instance: hemat 70–90% vs On-Demand
- 2 jam processing × Spot price (~$0.025/jam untuk c5.large Spot) = **~$0.05/hari = $1.5/bulan**
- **Penghematan: ~97%**

**Arsitektur:**
```
File tiba di S3 → S3 Event Notification → SQS →
AWS Batch Job Queue → Batch launch Spot Instance →
Process CSV → Store hasil ke DB → Instance terminate
```

**Kenapa bukan yang lain:**
- **A** — t3.micro lebih kecil: masih running 24/7 (~$8/bulan), masih ada 22 jam idle. Juga mungkin tidak cukup untuk 2 jam processing CSV besar.
- **B** — Schedule start/stop: lebih baik dari 24/7, tapi masih perlu running setiap hari meskipun mungkin tidak ada file (partner skip kirim). Dan tetap ada waktu instance running tanpa processing.
- **C** — RI t3.large: diskon ~40% tapi still $36/bulan untuk 24/7. Masih 22 jam idle per hari di-charge.

**Konsep yang diuji:** AWS Batch, event-driven batch processing, Spot for batch, S3 event trigger, cost elimination vs reduction.

---

## Soal 5 — Jawaban: D

**Least privilege per Lambda function**

**Kenapa D:**
`AdministratorAccess` pada Lambda adalah **critical security vulnerability**:
- Jika Lambda compromised (injection attack, dependency vulnerability), attacker dapat akses penuh ke seluruh AWS account
- Lambda tidak perlu create EC2, manage IAM users, atau delete S3 buckets — tapi `AdministratorAccess` mengizinkan semua itu

**Prinsip least privilege untuk Lambda:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["dynamodb:GetItem", "dynamodb:PutItem", "dynamodb:UpdateItem"],
      "Resource": "arn:aws:dynamodb:us-east-1:123456789:table/Orders"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::my-bucket/uploads/*"
    }
  ]
}
```

Setiap Lambda function mendapat **separate IAM role** dengan hanya permissions yang benar-benar dibutuhkan pada resource ARN yang spesifik.

**Kenapa bukan yang lain:**
- **A** — Cognito Identity Pool untuk end-user authentication (mobile/web apps), bukan untuk Lambda-to-AWS service authentication. Lambda sudah punya IAM execution role.
- **B** — Reserved Concurrency membatasi jumlah concurrent executions — tidak ada hubungannya dengan IAM permissions atau blast radius dari compromised credentials.
- **C** — Enkripsi environment variables melindungi secrets di rest, tapi tidak mengurangi permissions yang dimiliki Lambda. Jika Lambda compromised, attacker tetap bisa memanggil AWS APIs dengan permissions yang dimiliki role.

**Konsep yang diuji:** Lambda IAM execution role, least privilege principle, separate roles per function, specific ARN permissions, blast radius reduction.

---

## Soal 6 — Jawaban: C

**Poison pill message + SQS Dead Letter Queue**

**Kenapa C:**
**Poison pill** adalah message yang menyebabkan consumer gagal setiap kali mencoba memprosesnya — dalam kasus ini, order yang membutuhkan inventory database saat database timeout.

Tanpa DLQ: message gagal → visibility timeout expire → message kembali ke queue → di-retry → gagal lagi → terus berulang → **queue head-of-line blocking**.

**Dengan DLQ:**
```
Main Queue → Lambda process → Gagal → retry 1,2,3...
Setelah maxReceiveCount (misal 3): → Pindah ke DLQ otomatis
Main Queue continues processing other messages ✓
DLQ: tim investigate poison pill, fix root cause, reprocess
```

Konfigurasi:
- `RedrivePolicy`: `{ "deadLetterTargetArn": "arn:aws:sqs:...:orders-dlq", "maxReceiveCount": 3 }`
- Alarm CloudWatch pada DLQ depth untuk notification

**Kenapa bukan yang lain:**
- **A** — SQS FIFO menjamin ordering dan exactly-once, tapi jika ada poison pill di FIFO queue, **semua messages di MessageGroup yang sama ter-blokir** — lebih buruk dari Standard queue.
- **B** — Scale up Lambda concurrency tidak membantu — masalahnya adalah message yang selalu gagal, bukan kurang concurrency.
- **D** — Exponential backoff memang best practice untuk transient failures, tapi untuk database yang terus timeout, backoff hanya memperlambat retry — tidak menyelesaikan blocking messages.

**Konsep yang diuji:** SQS Dead Letter Queue, poison pill message, maxReceiveCount, message blocking, queue head-of-line blocking.

---

## Soal 7 — Jawaban: C

**Aurora PostgreSQL menggunakan AWS SCT + DMS**

**Kenapa C:**
Oracle licensing sangat mahal — bisa 30–50% dari total infrastructure cost. Migrasi ke open-source adalah **major cost optimization** jangka panjang.

**AWS-recommended path: Oracle → Aurora PostgreSQL:**

**Step 1 — Schema Conversion (AWS SCT):**
- Analyze Oracle schema: tables, indexes, views, stored procedures, triggers, sequences
- Otomatis convert yang bisa di-convert (table definitions, basic queries)
- Flag yang perlu manual review (Oracle-specific syntax seperti CONNECT BY, MERGE)
- Generate conversion report dengan complexity score

**Step 2 — Data Migration (AWS DMS):**
- Full load: copy semua existing data
- CDC (Change Data Capture): replikasi ongoing changes selama cutover period
- Minimal downtime cutover

**Kenapa bukan yang lain:**
- **A** — Lift-and-shift ke EC2 Oracle: masih butuh Oracle license (BYOL), dan EC2 Oracle license per-core sangat mahal. Tidak menyelesaikan licensing cost problem.
- **B** — RDS for Oracle: managed service yang lebih mudah dari self-managed, tapi Oracle licensing masih ada (license included atau BYOL). Tidak mengurangi licensing cost secara signifikan.
- **D** — DynamoDB: perubahan data model yang sangat besar (relational → NoSQL). Bukan pilihan yang tepat untuk aplikasi standard yang menggunakan SQL joins, transactions, dan relational model.

**Konsep yang diuji:** Oracle to Aurora PostgreSQL migration, AWS SCT, AWS DMS, heterogeneous migration, Oracle licensing cost elimination.

---

## Soal 8 — Jawaban: D

**Agregasi logs + S3 Lifecycle + Athena query**

**Kenapa D:**
CloudFront access logs menghasilkan **jutaan file kecil** karena setiap edge location menulis log file sendiri secara periodik (setiap 10–60 detik). Ini menyebabkan:
- Jutaan S3 PUT requests ($0.005 per 1.000 PUTs)
- Jutaan kecil files → tinggi S3 LIST cost
- Storage untuk file kecil inefficient

**Solusi komprehensif:**
1. **Tetap aktifkan logging** (disable = kehilangan visibility) tapi atur dengan bijak
2. **S3 Lifecycle**: hapus logs setelah 30/90 hari (sesuai kebutuhan audit)
3. **Athena query** untuk analysis — tidak perlu download/process log files secara manual
4. **CloudFront real-time logs ke Kinesis Data Firehose** → Firehose buffer dan batch records → deliver ke S3 dalam file yang lebih besar → lebih sedikit PUT requests

Opsi modern: **CloudFront Standard Logging v2** (structured JSON, lebih efisien).

**Kenapa bukan yang lain:**
- **A** — Matikan logs sepenuhnya: kehilangan visibility untuk security investigation, debugging, analytics (popular pages, error rates, cache hit ratio). Tidak acceptable untuk production.
- **B** — 1% sampling: kehilangan 99% log data. Security incidents bisa tidak terdeteksi. Tidak tepat.
- **C** — One Zone-IA untuk log bucket: mengurangi storage cost sedikit (~20%) tapi tidak menyelesaikan masalah utama (jumlah PUT requests dan file kecil).

**Konsep yang diuji:** CloudFront logging options, S3 small files problem, Lifecycle for cost control, Athena for log analysis, Kinesis Firehose batching.

---

## Soal 9 — Jawaban: B

**Cognito + DynamoDB LeadingKeys condition**

**Kenapa B:**
Tenant isolation di aplikasi SaaS adalah fundamental security requirement. Pendekatan **row-level security** di DynamoDB:

**Flow lengkap:**
1. User login via Cognito User Pool → JWT token berisi `custom:tenant_id` claim
2. API Gateway menggunakan Cognito Authorizer atau Lambda Authorizer untuk verify JWT
3. Lambda extract `tenant_id` dari token yang sudah diverifikasi
4. Lambda execution role punya condition:

```json
{
  "Effect": "Allow",
  "Action": ["dynamodb:GetItem", "dynamodb:Query", "dynamodb:PutItem"],
  "Resource": "arn:aws:dynamodb:...:table/AppData",
  "Condition": {
    "ForAllValues:StringEquals": {
      "dynamodb:LeadingKeys": ["${jwt:custom:tenant_id}"]
    }
  }
}
```

Atau lebih praktis: Lambda enforce tenant isolation dalam code — query selalu menyertakan `tenant_id` filter dari verified JWT, tidak dari user input.

**Kenapa bukan yang lain:**
- **A** — DynamoDB table per tenant: scalable untuk < 100 tenant tapi untuk ratusan/ribuan tenant: overhead management sangat besar, biaya DynamoDB minimum per table, IAM policies per table. Tidak scalable.
- **C** — KMS key per tenant: enkripsi melindungi data at-rest tapi tidak mencegah query cross-tenant. Lambda yang compromised masih bisa query data tenant lain karena KMS key bukan access control untuk queries.
- **D** — VPC per tenant: sangat mahal (VPC + subnet + NAT Gateway per tenant), over-engineered untuk aplikasi SaaS, dan tidak scalable.

**Konsep yang diuji:** Multi-tenant isolation, Cognito JWT claims, DynamoDB LeadingKeys condition, row-level security, SaaS architecture.

---

## Soal 10 — Jawaban: C

**API Gateway + Lambda + DynamoDB + WAF**

**Kenapa C:**
Verifikasi setiap requirement:

| Requirement | API GW + Lambda + DynamoDB + WAF |
|-------------|----------------------------------|
| Tidak ada server yang di-manage | ✅ Semua serverless |
| Scale dari 0 hingga ribuan req/s | ✅ API GW scale otomatis, Lambda concurrent, DynamoDB on-demand |
| Data tersimpan durable | ✅ DynamoDB 99.999% durability, 3 AZ replication |
| Biaya hanya saat digunakan | ✅ API GW per request, Lambda per invocation, DynamoDB per RCU/WCU (on-demand) |
| Proteksi SQLi dan DDoS | ✅ AWS WAF (SQLi, XSS rules) + API GW throttling |

**Kenapa bukan yang lain:**
- **A** — EC2 Auto Scaling + RDS: server yang harus di-manage (EC2, RDS); tidak scale to zero; RDS di-charge per jam meskipun idle. Tidak memenuhi "tidak ada server yang di-manage" dan "biaya hanya saat digunakan".
- **B** — ECS Fargate + Aurora Serverless: Fargate serverless tapi masih container yang perlu di-manage (Docker image, task definition); Aurora Serverless v2 memang scale tapi minimum ACU tetap di-charge. Partially memenuhi requirement.
- **D** — Lambda + S3 + CloudFront + Cognito: S3 untuk static storage bukan database transactional; CloudFront untuk CDN; Cognito untuk auth. Tidak ada WAF untuk SQLi/DDoS protection. Tidak complete.

**Konsep yang diuji:** Serverless architecture, API Gateway + Lambda + DynamoDB pattern, WAF protection, full serverless requirement mapping.

---

## Rekap Skor — Set 25 (Final)

| Soal | Domain | Topik |
|------|--------|-------|
| 1 | Secure | SCP preventive control, CloudTrail protection, Organizations |
| 2 | Resilient | Warm Standby DR, Aurora Global Database, RTO/RPO |
| 3 | High-Performing | Lambda@Edge, edge processing, latency reduction |
| 4 | Cost-Optimized | AWS Batch + Spot, event-driven batch, idle EC2 elimination |
| 5 | Secure | Lambda least privilege, separate IAM roles per function |
| 6 | Resilient | SQS DLQ, poison pill, maxReceiveCount |
| 7 | High-Performing | Oracle → Aurora PostgreSQL, SCT + DMS, licensing |
| 8 | Cost-Optimized | CloudFront logs, S3 small files, Lifecycle + Athena |
| 9 | Secure + Resilient | Multi-tenant isolation, Cognito, DynamoDB LeadingKeys |
| 10 | All Domains | Serverless architecture, API GW + Lambda + DynamoDB + WAF |

---

## 🎯 Selamat! 250 Soal Latihan SAA-C03 Selesai

| Domain | Target | Dibuat |
|--------|--------|--------|
| Secure Architectures (30%) | 75 | 75 ✅ |
| Resilient Architectures (26%) | 65 | 60 + 5 di Mixed ✅ |
| High-Performing (24%) | 60 | 60 ✅ |
| Cost-Optimized (20%) | 50 | 50 ✅ |
| Mixed | — | 10 ✅ |
| **Total** | **250** | **255** ✅ |

**Good luck dengan ujian SAA-C03!**
