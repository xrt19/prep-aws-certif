# Jawaban — Set 23
## Domain: Cost-Optimized Architectures

---

## Soal 1 — Jawaban: D

**Karpenter / Cluster Autoscaler + Spot Instances**

**Kenapa D:**
EKS worker nodes yang running 24/7 dengan banyak idle capacity adalah sumber pemborosan besar. Strategi optimal:

**1. Cluster Autoscaler atau Karpenter:**
- Scale down nodes yang idle (tidak ada pods)
- Scale up saat ada pods pending
- Karpenter (lebih modern): provision node tepat sesuai kebutuhan pod spec, tidak over-provision

**2. Spot Instances untuk worker nodes:**
- Pods aplikasi (stateless, dapat di-reschedule): jalankan di Spot nodes → hemat 60–80%
- System pods kritis (CoreDNS, kube-proxy, monitoring): jalankan di On-Demand nodes
- Tolerations dan node selectors untuk control di node mana pods berjalan

**3. Node diversification:**
- Request Spot dari berbagai instance types (m5.large, m5a.large, m4.large) → lebih sedikit interruption karena tidak semua pools habis bersamaan

**Kenapa bukan yang lain:**
- **A** — RI untuk semua worker nodes: mungkin cost-effective untuk baseline, tapi tidak menyelesaikan masalah nodes idle saat traffic rendah. Dan EKS worker nodes skalanya dinamis — sulit predict berapa RI yang tepat.
- **B** — Instance lebih besar memang meningkatkan pod density per node, tapi tetap ada idle capacity. Juga tidak menyelesaikan scale-down saat tidak diperlukan.
- **C** — Semua ke EKS Fargate: Fargate di-charge per pod (vCPU + memory per second) — lebih mahal dari Spot worker nodes untuk workload yang terus-menerus running. Fargate lebih cocok untuk burst/sporadic pods.

**Konsep yang diuji:** EKS cost optimization, Cluster Autoscaler/Karpenter, Spot worker nodes, On-Demand + Spot mix, node diversification.

---

## Soal 2 — Jawaban: C

**Atur retention policy per Log Group**

**Kenapa C:**
CloudWatch Logs pricing:
- **Ingestion**: $0.50 per GB ingested
- **Storage**: $0.03 per GB per bulan
- **"Never Expire"** berarti semua logs terakumulasi tanpa batas → storage terus bertambah

Retention policy per Log Group:
```
/aws/lambda/my-function  → 7 hari
/aws/ecs/production      → 30 hari
/aws/cloudtrail/audit    → 365 hari
/aws/rds/errors          → 14 hari
```

Setelah retention period tercapai, logs **otomatis dihapus** — tidak perlu action manual. Bisa set via console, CLI, atau Terraform/CDK untuk standardisasi.

Best practice: gunakan AWS Config rule `cloudwatch-log-group-retention-period-check` untuk detect log groups tanpa retention policy.

**Kenapa bukan yang lain:**
- **A** — Kurangi log level ke ERROR: mengurangi volume logs yang masuk (ingestion cost), tapi tidak mengurangi biaya storage untuk logs yang sudah ada. Juga mengurangi observability — DEBUG/INFO logs berguna untuk troubleshooting.
- **B** — Disable logging non-production: extreme measure, mengurangi observability untuk debugging. Lebih baik set retention pendek (3–7 hari) untuk non-prod daripada disable sama sekali.
- **D** — Ekspor ke S3 dan hapus dari CloudWatch: valid approach untuk long-term archival (S3 jauh lebih murah untuk storage), tapi lebih complex untuk setup dan maintain. Retention policy lebih simple dan cukup untuk sebagian besar use cases.

**Konsep yang diuji:** CloudWatch Logs pricing, retention policy, storage cost optimization, Never Expire anti-pattern.

---

## Soal 3 — Jawaban: C

**SQS Long Polling (WaitTimeSeconds=20)**

**Kenapa C:**
SQS pricing: **$0.40 per 1 juta requests** (Standard Queue).

**Short polling (default):**
- Consumer kirim ReceiveMessage request
- SQS check subset of servers, return immediately (0 messages jika queue kosong)
- Consumer langsung kirim request baru
- Misalnya 10 consumers × 1 request/detik × 3600 detik = 36.000 requests/jam = ~864.000 requests/hari

**Long polling (WaitTimeSeconds=1–20):**
- Consumer kirim ReceiveMessage dengan `WaitTimeSeconds=20`
- SQS **menunggu** hingga 20 detik untuk ada message sebelum return
- Jika queue kosong 90% waktu: dari 36.000 requests/jam menjadi ~1.800 requests/jam (10 consumers × 1 request/20 detik × 3600 detik)
- **Pengurangan requests: ~95%**

Long polling juga mengurangi **empty responses** dan **false empty responses** karena SQS check semua servers (bukan subset).

**Kenapa bukan yang lain:**
- **A** — SNS adalah push-based: messages di-push ke subscribers (Lambda, SQS, HTTP endpoint). Tidak menyelesaikan masalah polling cost — dan SNS tidak bisa menggantikan SQS untuk queuing dengan acknowledgment dan retry.
- **B** — Kurangi consumers memang mengurangi total requests, tapi juga mengurangi throughput. Jika consumers memang butuh banyak untuk handle traffic, ini bukan solusi.
- **D** — Kinesis untuk streaming data dengan high throughput, bukan sebagai pengganti SQS yang butuh acknowledgment, visibility timeout, DLQ, dan FIFO ordering. Berbeda use case.

**Konsep yang diuji:** SQS Long Polling, short polling vs long polling, WaitTimeSeconds, API request cost reduction.

---

## Soal 4 — Jawaban: D

**Tambahkan ElastiCache di depan RDS**

**Kenapa D:**
**Caching layer** untuk read-heavy workload dengan data yang jarang berubah:

```
Application → ElastiCache (cache hit ~80-95%) → return cached result
Application → ElastiCache (cache miss) → RDS → cache result → return
```

**Benefit cost:**
- Dengan 80% cache hit rate: hanya 20% queries yang reach RDS
- RDS load turun 80% → bisa downsize instance (misalnya dari db.r6g.2xlarge ke db.r6g.large) → **hemat ~50% RDS cost**
- ElastiCache cache.r6g.large lebih murah dari RDS db.r6g.2xlarge

**Data yang tidak sering berubah** (katalog produk, referensi) cocok untuk TTL panjang (jam atau hari) → cache hit rate sangat tinggi.

**Kenapa bukan yang lain:**
- **A** — Upgrade RDS lebih mahal bukan lebih murah. Tidak mengurangi jumlah queries ke RDS.
- **B** — **Exam trap klasik**: RDS Multi-AZ standby tidak serve reads. Standby untuk failover saja. Tidak mengurangi load di primary.
- **C** — Read Replica memang bisa offload reads, tapi: (1) Read Replica di-charge sama dengan instance biasa; (2) Tidak ada caching — setiap query tetap hit database; (3) Lebih mahal dari ElastiCache untuk use case ini. ElastiCache lebih cost-effective karena cache layer mengeliminasi repeated database queries.

**Konsep yang diuji:** ElastiCache caching pattern, cache-aside, cost reduction via RDS downsizing, caching untuk infrequently-changing data.

---

## Soal 5 — Jawaban: D

**Gabungkan objek kecil menjadi objek lebih besar**

**Kenapa D:**
S3 API request pricing:
- **PUT, COPY, POST, LIST**: $0.005 per 1.000 requests
- **GET, SELECT**: $0.0004 per 1.000 requests

Jika ada 100 juta objek kecil (1 KB):
- Total storage: hanya 100 GB (~$2.30/bulan)
- Tapi 100 juta GET requests = $40, 100 juta PUT requests = $500

Dengan **aggregasi** (gabungkan per folder/waktu/kategori menjadi ZIP/Parquet):
- 100 juta objek → 10.000 file ZIP (masing-masing berisi 10.000 objek)
- GET requests untuk akses berkurang drastis
- S3 Intelligent-Tiering monitoring fee juga berkurang (per object)

**Best practice untuk data lake:** simpan data dalam format columnar Parquet dengan ukuran file 128 MB–1 GB untuk optimal cost dan performance Athena.

**Kenapa bukan yang lain:**
- **A** — Glacier tidak bisa di-query langsung dengan API request yang sama. Akses ke Glacier butuh restore request terlebih dahulu. Dan Glacier request pricing juga ada — tidak menyelesaikan fundamental masalah jumlah requests yang banyak.
- **B** — Intelligent-Tiering mengurangi **storage cost** per GB, tapi tidak mengurangi **jumlah API requests**. Masalah utamanya adalah request cost bukan storage cost.
- **C** — S3 Batch Operations untuk operasi bulk pada existing objects (copy, tag, restore). Tidak mengurangi ongoing request cost dari aplikasi yang terus access jutaan objek kecil.

**Konsep yang diuji:** S3 API request pricing, small objects aggregation, cost optimization for large object count.

---

## Soal 6 — Jawaban: D

**Kombinasi Reserved Instances (baseline) + Spot (burst)**

**Kenapa D:**
Ini adalah **best practice** untuk cost-optimized, predictable-variable traffic:

```
Traffic pattern:
Weekday peak:  15 instances needed
Weekday off:    5 instances needed
Weekend:        2 instances needed
```

**Strategi:**
- **2 Reserved Instances (All Upfront, 1 year)**: baseline minimum yang selalu running → diskon ~50%
- **Spot Instances untuk burst**: 3–13 instances tambahan saat traffic tinggi → 70–90% lebih murah dari On-Demand
- **On-Demand sebagai fallback**: jika Spot di-interrupt, ASG launch On-Demand untuk maintain minimum capacity

**Konfigurasi ASG Mixed Instances Policy:**
```
On-Demand base capacity: 2 (atau 0 jika pakai RI di luar ASG)
Spot percentage of scale: 80%
On-Demand percentage: 20% (fallback)
```

**Kenapa bukan yang lain:**
- **A** — Semua On-Demand dengan ASG: ASG scale down saat off-peak, tapi On-Demand lebih mahal dari RI untuk baseline dan lebih mahal dari Spot untuk burst.
- **B** — Semua RI: jumlah RI harus = peak capacity (15 instances) → idle RI di weekend wasted. RI di-charge per jam selama commitment period meskipun tidak running.
- **C** — Semua Spot: bisa di-interrupt bersamaan saat AWS reclaim capacity → seluruh aplikasi down. Tidak acceptable untuk production.

**Konsep yang diuji:** Mixed purchasing strategy, RI for baseline + Spot for burst, ASG Mixed Instances Policy, On-Demand fallback.

---

## Soal 7 — Jawaban: C

**Pindahkan S3 bucket ke region yang sama**

**Kenapa C:**
AWS data transfer pricing:
- **Antar region** (EC2 → S3 cross-region): **$0.02/GB** (untuk sebagian besar regions)
- **Dalam region yang sama** (EC2 → S3 same region via private/public endpoint): **Gratis** untuk transfer ke S3 dari EC2 di region yang sama

Jika ada 10 TB/bulan cross-region traffic:
- Biaya sekarang: 10.000 GB × $0.02 = **$200/bulan**
- Setelah S3 bucket dipindah ke us-east-1: **$0/bulan**

Opsi lain: gunakan **S3 CRR (Cross-Region Replication)** untuk replikasi data ke us-east-1, kemudian EC2 baca dari bucket di us-east-1. Ada biaya replikasi satu kali tapi ongoing read cost = $0.

**Kenapa bukan yang lain:**
- **A** — S3 Transfer Acceleration mempercepat **upload dari klien ke S3** menggunakan CloudFront edge. Tidak mengurangi data transfer cost antar region — justru ada biaya tambahan untuk Transfer Acceleration.
- **B** — Kompresi mengurangi bytes yang di-transfer, mengurangi biaya secara proporsional. Tapi masih ada $0.02/GB untuk bytes yang di-transfer. Memindahkan bucket menghilangkan biaya sepenuhnya.
- **D** — Direct Connect untuk koneksi dedicated dari **on-premise** ke AWS. Tidak membantu untuk EC2-to-S3 cross-region transfer di dalam AWS.

**Konsep yang diuji:** S3 cross-region vs same-region data transfer pricing, cost elimination vs reduction, S3 CRR as alternative.

---

## Soal 8 — Jawaban: C

**AWS Budgets + budget actions**

**Kenapa C:**
AWS Budgets dengan **Budget Actions** memberikan automated response terhadap cost overrun:

**Budget Actions yang tersedia:**
1. **Apply IAM policy**: attach IAM policy yang deny create resources ke specific users/groups/roles
2. **Apply SCP**: apply SCP ke OU atau account yang restrict resource creation
3. **Target EC2/RDS instances**: stop specific EC2 instances atau RDS instances

**Workflow untuk dev environment:**
- Set budget: $500/bulan per team
- Alert threshold 1 (80% = $400): kirim SNS notification ke team lead
- Alert threshold 2 (100% = $500): apply IAM policy deny `ec2:RunInstances`, `rds:CreateDBInstance`
- Team harus cleanup resources sebelum bisa create baru

Dikombinasikan dengan **mandatory cost allocation tags** agar accountability jelas per developer/project.

**Kenapa bukan yang lain:**
- **A** — Review manual mingguan: tidak scalable, reactive bukan preventive. Tim bisa sudah habiskan $1000 sebelum review.
- **B** — Separate account per developer: isolasi yang bagus untuk billing tapi overhead administrasi sangat besar (setup, access management, onboarding/offboarding).
- **D** — AWS Config track configuration dan detect compliance violations, tapi tidak otomatis restrict atau alert saat cost melebihi threshold. Perlu custom Lambda rule untuk cost enforcement yang jauh lebih complex.

**Konsep yang diuji:** AWS Budgets budget actions, automated cost governance, IAM policy enforcement, SCP as cost control, dev environment cost sprawl.

---

## Soal 9 — Jawaban: D

**Spot Fleet dengan diversifikasi**

**Kenapa D:**
Untuk HPC yang butuh ribuan vCPU dengan deadline dan toleran interruption:

**Spot Fleet dengan diversifikasi:**
- Request dari **multiple instance pools**: c5.4xlarge di us-east-1a, c5.2xlarge di us-east-1b, c5a.4xlarge di us-east-1c, m5.4xlarge di us-east-1a, dst
- **`capacityOptimized` allocation strategy**: pilih pool dengan kapasitas terbanyak → lower interruption rate
- Diversifikasi mencegah "cliff event" — tidak semua Spot instances di-reclaim bersamaan karena berasal dari pools berbeda
- Hemat 70–90% vs On-Demand untuk compute workload HPC

**Checkpoint strategy:**
- Simpan intermediate results ke S3 secara periodik
- Saat ada interruption: hanya re-process dari checkpoint terakhir, tidak dari awal

**Kenapa bukan yang lain:**
- **A** — Quota increase untuk On-Demand dan launch semua On-Demand: (1) Prosesnya butuh waktu berhari-hari; (2) Sangat mahal untuk ribuan vCPU; (3) Tidak ada penghematan. Tidak tepat untuk HPC yang cost-sensitive dan toleran interruption.
- **B** — RI untuk HPC workload yang hanya dijalankan sekali atau beberapa kali: waste commitment. RI cocok untuk steady-state, bukan burst HPC.
- **C** — "Multiple Spot requests di berbagai instance types dan AZs" sama dengan Spot Fleet, tapi Spot Fleet adalah managed service yang handle ini secara otomatis. Lebih baik gunakan Spot Fleet daripada manage multiple individual requests.

**Konsep yang diuji:** Spot Fleet, capacityOptimized strategy, diversification across pools, HPC cost optimization, cliff event prevention.

---

## Soal 10 — Jawaban: C

**AWS Managed Rules**

**Kenapa C:**
AWS WAF pricing:
- **Web ACL**: $5/bulan
- **Rules**: $1/bulan per rule
- **Requests**: $0.60 per 1 juta requests

Jika ada 50 custom rules: $50/bulan hanya dari rules.

**AWS Managed Rules:**
- Rule groups pre-built: AWS Core Rule Set, Known Bad Inputs, SQL Database, Linux OS, PHP, WordPress, dll
- Harga flat per **rule group** (bukan per rule): ~$1–$10/bulan per rule group
- Satu rule group "AWS Core Rule Set" berisi 15+ rules → hanya di-charge satu rule group = $5, bukan $15 per rule
- AWS update rules secara otomatis berdasarkan threat intelligence terbaru
- Tersedia dari AWS dan AWS Marketplace (third-party: F5, Imperva, Fortinet)

Mengganti 30 custom rules dengan 2–3 managed rule groups: dari $30 menjadi $15 rule cost.

**Kenapa bukan yang lain:**
- **A** — Hapus WAF dan gunakan Security Groups: Security Groups hanya filter berdasarkan IP/port di Layer 4. WAF untuk Layer 7 (SQL injection, XSS, bot, rate limiting). Menghapus WAF sangat meningkatkan risk, bukan solusi yang acceptable.
- **B** — Pindah ke CloudFront dengan WAF: biaya WAF sama saja di mana di-attach (ALB atau CloudFront). Ada biaya CloudFront tambahan. Tidak mengurangi biaya WAF secara fundamental.
- **D** — Kurangi sampling rate WAF logging: mengurangi biaya CloudWatch Logs/S3 untuk WAF logs, tapi tidak mengurangi biaya utama WAF (rules dan requests). Minor savings.

**Konsep yang diuji:** AWS WAF pricing, Managed Rules vs custom rules, rule group flat pricing, security cost optimization.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | EKS Spot worker nodes, Karpenter/Cluster Autoscaler |
| 2 | CloudWatch Logs retention policy, storage cost |
| 3 | SQS Long Polling, API request cost reduction |
| 4 | ElastiCache caching untuk reduce RDS cost |
| 5 | S3 small objects aggregation, API request pricing |
| 6 | RI baseline + Spot burst, Mixed Instances Policy |
| 7 | S3 cross-region vs same-region transfer pricing |
| 8 | AWS Budgets actions, automated cost governance |
| 9 | Spot Fleet capacityOptimized, diversification, HPC |
| 10 | AWS WAF Managed Rules vs custom rules pricing |
