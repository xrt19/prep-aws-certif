# Jawaban — Set 21
## Domain: Cost-Optimized Architectures

---

## Soal 1 — Jawaban: C

**Redshift Serverless**

**Kenapa C:**
Redshift Serverless menghilangkan masalah over-provisioning untuk workload intermittent:
- **Billing**: per RPU-second (Redshift Processing Unit) — hanya saat ada query aktif berjalan
- **Idle cost = $0** — tidak ada cluster yang running saat tidak ada queries
- **Auto scaling**: scale up otomatis saat ada heavy query, scale down saat idle
- **Base RPU**: bisa set minimum (8 RPU = minimum) untuk balance cold start vs cost

Untuk analytics yang hanya intensif di jam kerja dan idle malam/weekend, Redshift Serverless menghilangkan cost untuk ~50–70% waktu dimana cluster sebelumnya idle.

**Kenapa bukan yang lain:**
- **A** — Snapshot + delete + restore setiap hari: (1) Restore Redshift cluster butuh 5–15 menit → downtime setiap pagi; (2) Snapshot storage cost; (3) Operational overhead tinggi; (4) Tidak practical untuk production analytics environment.
- **B** — RA3 nodes memiliki managed storage (pisah storage dari compute) yang bagus untuk scaling storage independently, tapi cluster tetap berjalan 24/7 dan di-charge per jam. Tidak menyelesaikan idle cost problem.
- **D** — Reserved Nodes Redshift memberikan diskon ~75% tapi di-charge per jam sepanjang commitment period. Tetap membayar saat idle malam dan weekend.

**Konsep yang diuji:** Redshift Serverless, RPU billing, idle cost elimination, intermittent analytics workload.

---

## Soal 2 — Jawaban: C

**Compute Savings Plans**

**Kenapa C:**
AWS Savings Plans adalah commitment berbasis **spend** ($/jam), bukan specific instances:

**Jenis Savings Plans:**
| | Compute Savings Plans | EC2 Instance Savings Plans |
|--|----------------------|--------------------------|
| Fleksibilitas | Instance family, size, region, OS, tenancy | Hanya instance family + region |
| Berlaku untuk | EC2, Lambda, Fargate | EC2 only |
| Diskon vs On-Demand | Hingga 66% | Hingga 72% |

**Keunggulan Compute Savings Plans:**
- Commit $X/jam, AWS apply diskon ke semua eligible compute usage
- Bisa pindah instance type dari m5.xlarge ke c5.2xlarge → diskon tetap berlaku
- Bisa pindah region (us-east-1 ke eu-west-1) → diskon tetap berlaku
- Berlaku juga untuk Lambda invocations dan Fargate tasks

**Kenapa bukan yang lain:**
- **A** — Standard RI: diskon tertinggi (hingga 75%) tapi terikat ke specific instance type, OS, region, tenancy. Tidak bisa diubah tanpa sell di RI Marketplace. Tidak fleksibel.
- **B** — Spot: tidak ada commitment, bisa di-interrupt. Bukan solusi untuk workload production yang butuh reliability.
- **D** — Convertible RI: bisa ubah instance attributes (family, size, OS) tapi: (1) Hanya berlaku untuk EC2 satu region; (2) Proses konversi manual; (3) Tidak berlaku untuk Lambda/Fargate. Compute Savings Plans lebih fleksibel.

**Konsep yang diuji:** Compute Savings Plans vs EC2 Instance Savings Plans vs RI, flexibility, coverage across services.

---

## Soal 3 — Jawaban: D

**CloudFront Price Class 100**

**Kenapa D:**
CloudFront di-charge berdasarkan **edge location yang melayani request** — beberapa regions lebih mahal dari yang lain:

| Price Class | Edge Locations | Relative Cost |
|------------|---------------|--------------|
| Price Class All | Semua regions | Tertinggi |
| Price Class 200 | US, Europe, Asia, Middle East, Africa | Menengah |
| Price Class 100 | US, Canada, Europe, Israel | Terendah |

Jika mayoritas traffic dari US dan Europe, Price Class 100 sudah cukup. User dari Asia/South America masih bisa dilayani — mereka akan di-route ke edge terdekat yang tersedia (US atau Europe) dengan sedikit latency tambahan.

**Trade-off:** Latency lebih tinggi untuk user di luar US/Europe, tapi biaya lebih rendah. Untuk konten yang tidak latency-sensitive (assets statis, downloads), ini acceptable.

**Kenapa bukan yang lain:**
- **A** — TTL lebih pendek berarti lebih banyak cache miss → lebih banyak origin requests → cost naik, bukan turun. Kebalikan dari yang diinginkan.
- **B** — Storage class S3 untuk origin tidak mempengaruhi CloudFront transfer cost. CloudFront di-charge untuk data transfer dari edge ke client, bukan storage class di origin.
- **C** — Compression memang mengurangi bytes yang di-transfer (dan cost data transfer CloudFront dihitung per GB), tapi ini optimasi incremental. Price Class change lebih berdampak karena mengurangi per-GB rate untuk semua traffic, bukan hanya data size.

**Konsep yang diuji:** CloudFront Price Class, edge location pricing, cost vs latency trade-off, Price Class 100/200/All.

---

## Soal 4 — Jawaban: C

**Fargate Spot**

**Kenapa C:**
Fargate Spot adalah Fargate capacity dengan harga diskon menggunakan spare AWS capacity:
- **Diskon hingga 70%** dari harga Fargate regular
- AWS bisa **interrupt tasks** dengan 2 menit notice (SIGTERM ke container)
- Cocok untuk: batch processing, background jobs, image/video processing, data transformation — workloads yang bisa di-retry atau di-checkpoint

**Cara implementasi:**
```json
{
  "capacityProviders": ["FARGATE", "FARGATE_SPOT"],
  "defaultCapacityProviderStrategy": [
    {"capacityProvider": "FARGATE_SPOT", "weight": 1, "base": 0}
  ]
}
```

Bisa mix FARGATE + FARGATE_SPOT: critical services di FARGATE, fault-tolerant services di FARGATE_SPOT.

**Kenapa bukan yang lain:**
- **A** — EC2 launch type memang bisa lebih murah untuk steady-state workloads (terutama dengan RI), tapi butuh manage EC2 instances (patching, scaling, availability). Untuk stateless containers, Fargate Spot lebih simple dan bisa lebih murah dengan Spot discount.
- **B** — Kurangi CPU/memory allocation memang mengurangi biaya (Fargate di-charge per vCPU dan per GB memory), tapi ada batas minimum — tidak bisa kurangi terlalu banyak tanpa mempengaruhi performance. Fargate Spot langsung 70% discount.
- **D** — ECS Capacity Providers dengan Spot EC2 juga valid dan bisa lebih murah, tapi butuh manage EC2 Auto Scaling Group. Fargate Spot lebih serverless dan less operational overhead.

**Konsep yang diuji:** Fargate Spot, interruption handling, fault-tolerant container workloads, Fargate pricing model.

---

## Soal 5 — Jawaban: B

**S3 Standard-IA**

**Kenapa B:**
Untuk data dengan **access pattern yang jelas** (sering di 30 hari pertama, kemudian jarang), S3 Standard-IA adalah pilihan optimal:

**Karakteristik S3 Standard-IA:**
- Storage cost: ~$0.0125/GB/bulan (vs $0.023/GB Standard — 46% lebih murah)
- Retrieval fee: $0.01/GB
- Minimum storage duration: **30 hari** — jika object dihapus sebelum 30 hari, tetap di-charge 30 hari
- Minimum object size: 128 KB (objek lebih kecil di-charge seolah 128 KB)
- Same durability dan availability sebagai Standard (99.99% availability vs 99.9% untuk One Zone-IA)
- Latency: same as Standard (milidetik)

Dengan lifecycle rule `transition after 30 days → Standard-IA`, semua objek pindah tepat setelah periode aktif selesai, langsung hemat ~46%.

**Kenapa bukan yang lain:**
- **A** — Intelligent-Tiering: ada **monitoring fee per object** ($0.0025 per 1000 objects). Untuk dataset besar dengan jutaan objek, monitoring fee bisa lebih mahal dari savings. Karena access pattern-nya sudah **jelas** (aktif 30 hari pertama), lifecycle rule lebih cost-optimal dari Intelligent-Tiering.
- **C** — Glacier Instant Retrieval: lebih murah tapi **minimum 90 hari** storage duration. Jika banyak objek dihapus sebelum 90 hari, biaya minimum duration mungkin lebih tinggi dari savings.
- **D** — One Zone-IA: hanya satu AZ — jika AZ down, data tidak tersedia. Untuk data bisnis yang penting, trade-off ini tidak acceptable hanya untuk ~20% tambahan penghematan.

**Konsep yang diuji:** S3 Standard-IA, minimum storage duration, retrieval fee, known access patterns, lifecycle policy design.

---

## Soal 6 — Jawaban: C

**Deploy CloudFront di depan EC2**

**Kenapa C:**
Data transfer pricing di AWS:

| Transfer Type | Cost |
|--------------|------|
| EC2 ke internet | $0.09/GB (us-east-1) |
| EC2 ke CloudFront (origin fetch) | **Gratis** (no charge) |
| CloudFront ke internet | $0.0085–$0.085/GB (tergantung region) |

**Double benefit:**
1. **Origin fetch gratis**: tidak ada charge untuk data transfer dari EC2 ke CloudFront
2. **CloudFront rate lebih murah**: CloudFront ke internet lebih murah dari EC2 ke internet langsung
3. **Caching**: setelah object di-cache, tidak ada lagi origin fetch — hanya CloudFront ke internet

Untuk file statis (jarang berubah), cache hit rate bisa sangat tinggi → hampir semua traffic served dari cache.

**Kenapa bukan yang lain:**
- **A** — Pindah region hanya membantu marginal karena data transfer pricing tidak berbeda drastis antar regions untuk us-east, eu-west, ap-southeast. Dan migrasi region adalah effort besar.
- **B** — Kompresi mengurangi bytes per response, tapi EC2 ke internet tetap mahal per GB. Savings terbatas pada persentase kompresi (20–80% tergantung content type). CloudFront menghilangkan sebagian besar origin traffic.
- **D** — S3 Transfer Acceleration untuk mempercepat **upload** ke S3 menggunakan CloudFront edge network. Tidak ada hubungannya dengan mengurangi data transfer cost dari EC2 ke users.

**Konsep yang diuji:** Data transfer pricing, EC2 to CloudFront free, CloudFront data transfer rates, caching to reduce origin fetch.

---

## Soal 7 — Jawaban: C

**Switch ke DynamoDB On-Demand capacity**

**Kenapa C:**
DynamoDB On-Demand vs Provisioned:

| | On-Demand | Provisioned |
|--|-----------|------------|
| Billing | Per request | Per provisioned WCU/RCU per hour |
| Unused capacity | Tidak ada | Dibayar meskipun tidak dipakai |
| Cost per request | Lebih mahal | Lebih murah jika utilisasi tinggi |
| Break-even | < ~20% utilization → On-Demand lebih murah | > ~20% utilization → Provisioned lebih murah |

Dengan utilization rata-rata **20% WCU dan 15% RCU** (jauh di bawah break-even), On-Demand lebih cost-optimal — hanya bayar untuk requests yang benar-benar terjadi.

**Kenapa bukan yang lain:**
- **A** — DAX (in-memory cache) mengurangi **read** requests ke DynamoDB → menurunkan RCU consumption. Tapi ada biaya DAX node per jam yang harus ditambahkan. Juga tidak menyelesaikan unused WCU yang terbuang.
- **B** — Global Table untuk multi-region replication — menambah biaya (replicated WRU lebih mahal dari standard WCU) bukan mengurangi. Tidak relevan untuk masalah ini.
- **D** — Kurangi manual ke 200 WCU/300 RCU bisa membantu, tapi provisioned capacity masih di-charge per hour meskipun ada momen utilization sangat rendah. On-Demand lebih tepat untuk utilisasi yang rendah dan tidak predictable.

**Konsep yang diuji:** DynamoDB On-Demand vs Provisioned, break-even utilization, unused capacity cost, cost optimization.

---

## Soal 8 — Jawaban: D

**AWS Cost Explorer + AWS Budgets + Cost Allocation Tags**

**Kenapa D:**
Tiga tools yang saling melengkapi untuk cost management:

**1. Cost Allocation Tags:**
- Tag semua resources: `Environment=prod`, `Team=backend`, `Project=checkout`
- AWS generate cost reports yang bisa di-breakdown per tag
- Perlu activate di Billing console

**2. AWS Cost Explorer:**
- Visualisasi spend per service, per account, per tag, per region
- Analisis trend (month-over-month)
- RI/Savings Plans utilization dan coverage reports
- Forecasting 12 bulan ke depan

**3. AWS Budgets:**
- Set threshold: misal "alert ketika EC2 cost > $1000/bulan"
- Alert via SNS/email saat mendekati atau melebihi budget
- Budget dapat berdasarkan cost, usage, RI utilization, atau Savings Plans coverage

**Kenapa bukan yang lain:**
- **A** — CloudTrail untuk audit API calls — bisa tahu siapa yang buat resource, tapi tidak memberikan cost analysis atau budget alerts.
- **B** — AWS Config untuk compliance dan configuration tracking. Tidak ada fungsi cost analysis.
- **C** — CloudWatch untuk infrastructure metrics dan alerts. Bisa monitor beberapa billing metrics dasar, tapi tidak sekomprehensif Cost Explorer untuk analysis.

**Konsep yang diuji:** AWS Cost Explorer, AWS Budgets, Cost Allocation Tags, cost visibility dan governance tools.

---

## Soal 9 — Jawaban: C

**Migrasi storage ke gp3**

**Kenapa C:**
Perbandingan gp2 vs gp3:

| | gp2 | gp3 |
|--|-----|-----|
| Baseline IOPS | 3 IOPS/GB (min 100) | 3000 IOPS (baseline, berapapun size) |
| Max IOPS | 16,000 (butuh 5334 GB) | 16,000 (bisa provision independent) |
| Max Throughput | 250 MB/s | 1000 MB/s |
| Price | $0.10/GB-month | $0.08/GB-month (~20% lebih murah) |
| Provision IOPS | Tidak bisa | Ya, terpisah dari size |

**Masalah dengan gp2:** Untuk dapat 3000 IOPS di gp2, butuh minimal 1000 GB volume. Jika hanya butuh 200 GB storage tapi 3000 IOPS, terpaksa over-provision storage ke 1000 GB → waste.

**Solusi gp3:** 200 GB storage + 3000 IOPS (baseline gratis) → bayar 200 GB saja, lebih murah dari 1000 GB gp2.

**Kenapa bukan yang lain:**
- **A** — Migrasi ke Aurora adalah perubahan besar — beda engine (Aurora MySQL vs RDS MySQL meskipun compatible, ada perbedaan), biaya migration, dan Aurora memiliki harga per ACU yang berbeda dari RDS. Over-engineered untuk masalah storage optimization.
- **B** — Snapshot dan restore ke instance lebih kecil tidak menyelesaikan masalah IOPS dependency pada storage size di gp2. Storage size harus tetap besar untuk dapat IOPS yang dibutuhkan.
- **D** — Magnetic (standard) storage jauh lebih lambat (100 IOPS average, burst ke few hundreds), tidak cocok untuk database yang butuh performance. Juga lebih mahal dari gp3 untuk equivalent performance.

**Konsep yang diuji:** EBS gp2 vs gp3, IOPS provisioning independent, cost optimization storage, gp2 IOPS per GB vs gp3 baseline.

---

## Soal 10 — Jawaban: D

**EMR Auto Termination + Spot Instances untuk task nodes**

**Kenapa D:**
Dua optimasi EMR yang komplementer:

**1. EMR Auto Termination:**
- Konfigurasi `idle timeout` — cluster otomatis terminate setelah idle X menit
- Menghilangkan 2 jam waste karena tidak ada yang terminate manual
- Bisa set `MaxIdleTime` saat create cluster

**2. Spot Instances untuk task nodes:**
- Task nodes di EMR adalah workers yang bisa di-remove kapanpun tanpa kehilangan data (data di HDFS atau S3)
- Spot Instances untuk task nodes: hemat 60–90%
- Core nodes harus On-Demand (menyimpan HDFS data — jika di-interrupt, data loss)
- Mix: 10 core nodes (On-Demand) + 20 task nodes (Spot)

Kombinasi keduanya menghilangkan idle waste DAN mengurangi compute cost selama job berjalan.

**Kenapa bukan yang lain:**
- **A** — Glue vs EMR adalah trade-off yang valid untuk ETL, tapi soal minta optimasi EMR yang sudah ada, bukan migrasi platform. Juga tidak semua EMR workloads bisa langsung dipindah ke Glue (custom Spark applications, Hive, HBase, dll).
- **B** — Reserved Instances untuk EMR: EMR instances bisa pakai RI karena tetap EC2 instances, memberikan diskon. Tapi idle time tetap di-charge. Dikombinasikan dengan auto-termination bisa saling melengkapi, tapi soal minta solusi yang paling tepat.
- **C** — Kurangi task nodes memang mengurangi biaya tapi juga mengurangi parallelism → jobs berjalan lebih lama → total biaya mungkin sama atau lebih mahal (lebih lama running). Trade-off tidak selalu menguntungkan.

**Konsep yang diuji:** EMR Auto Termination, EMR Spot Instances task nodes vs core nodes, HDFS data safety, EMR cost optimization.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | Redshift Serverless, RPU billing, idle cost elimination |
| 2 | Compute Savings Plans, flexibility vs Standard RI |
| 3 | CloudFront Price Class, edge location cost |
| 4 | Fargate Spot, interruptible container workloads |
| 5 | S3 Standard-IA, minimum duration, known access patterns |
| 6 | CloudFront data transfer cost, EC2 to CloudFront free |
| 7 | DynamoDB On-Demand vs Provisioned, break-even utilization |
| 8 | Cost Explorer + Budgets + Cost Allocation Tags |
| 9 | EBS gp3 vs gp2, independent IOPS provisioning |
| 10 | EMR Auto Termination, Spot task nodes vs On-Demand core nodes |
