# Jawaban — Set 24
## Domain: Cost-Optimized Architectures

---

## Soal 1 — Jawaban: D

**S3 Partitioning**

**Kenapa D:**
S3 partitioning adalah teknik organisasi data yang memungkinkan **partition pruning** di Athena:

**Struktur folder S3 tanpa partitioning:**
```
s3://my-bucket/data/events.csv  ← scan seluruh file untuk setiap query
```

**Struktur dengan Hive-style partitioning:**
```
s3://my-bucket/data/year=2024/month=01/day=01/part-0001.parquet
s3://my-bucket/data/year=2024/month=01/day=02/part-0001.parquet
s3://my-bucket/data/year=2023/month=12/day=31/part-0001.parquet
```

Query `WHERE year = '2024' AND month = '01'`:
- Tanpa partitioning: Athena scan semua data (misal 5 TB)
- Dengan partitioning: Athena hanya scan folder `year=2024/month=01/` (misal 50 GB) → **100x lebih sedikit data di-scan**

Karena Athena charge per data yang di-scan ($5/TB), savings langsung proporsional dengan data reduction.

**Kenapa bukan yang lain:**
- **A** — S3 Object Lock untuk compliance WORM protection — mencegah deletion/modification. Tidak mempengaruhi Athena scan behavior.
- **B** — S3 Replication untuk copy data ke region/bucket lain. Tidak mempengaruhi scan efficiency.
- **C** — S3 Versioning untuk track versi objek. Athena query versi terbaru by default. Tidak mempengaruhi scan cost.

**Konsep yang diuji:** S3 partitioning, partition pruning, Athena scan cost reduction, Hive-style partitioning.

---

## Soal 2 — Jawaban: C

**Kinesis Data Streams On-Demand mode**

**Kenapa C:**
Kinesis Data Streams memiliki dua capacity modes:

| | Provisioned Mode | On-Demand Mode |
|--|-----------------|----------------|
| Billing | Per shard-hour ($0.015/shard/jam) | Per GB ingested + retrieved |
| Scaling | Manual shard split/merge | Otomatis |
| Cost saat idle | Bayar 100 shards meskipun pakai 10 | Bayar sesuai actual throughput |
| Best for | Predictable traffic | Variable traffic |

Dengan 100 shards: $0.015 × 100 × 24 × 30 = **$1.080/bulan** meskipun 70% waktu hanya pakai 10 shards.

On-Demand mode: hanya bayar untuk data yang benar-benar di-ingest. Untuk traffic variable yang rata-rata 15% utilization, On-Demand jauh lebih hemat.

**Kenapa bukan yang lain:**
- **A** — Enhanced Fan-Out untuk **consumer read throughput** — setiap consumer dapat dedicated 2 MB/s per shard. Tidak mempengaruhi biaya shard provisioning atau scale down saat traffic rendah.
- **B** — Buat stream baru untuk off-peak: operasional sangat complex — perlu routing logic di producers untuk switch stream, migrate consumers, dan manage dua streams. Tidak praktis.
- **D** — Kinesis Data Firehose untuk deliver streaming data ke S3, Redshift, OpenSearch — bukan pengganti Streams untuk use case yang butuh real-time processing dengan multiple consumers, replay, dan offset management.

**Konsep yang diuji:** Kinesis Data Streams On-Demand vs Provisioned mode, variable traffic cost optimization, shard pricing.

---

## Soal 3 — Jawaban: D

**AWS Compute Optimizer untuk right-sizing**

**Kenapa D:**
CPU utilization rendah bukan selalu berarti instance terlalu besar — bisa juga karena:
- Aplikasi **memory-bound**: CPU rendah tapi memory terpakai 90%
- Aplikasi **I/O-bound**: CPU menunggu disk atau network I/O
- **Burst workload**: rata-rata CPU 5% tapi peak 90% sesaat

Compute Optimizer menganalisis **multiple dimensions**:
- CPU utilization (average, max, percentile)
- Memory utilization (via CloudWatch agent)
- Network in/out
- Disk read/write

Dari analisis ini, Compute Optimizer memberikan rekomendasi spesifik:
- "Ganti m5.2xlarge → m5.large: projected savings $80/bulan, performance risk: low"
- Atau "Pertahankan: memory utilization tinggi, downsize akan cause performance issue"

**Kenapa bukan yang lain:**
- **A** — Terminate langsung instances CPU-rendah: sangat berisiko. Bisa terminate instances yang sedang serve traffic (CPU rendah = request sedang sedikit). Atau instances yang memory/I/O-bound. Perlu analisis dulu.
- **B** — Pindah ke Spot langsung: Spot bisa di-interrupt. Untuk web application production, perlu careful planning, tidak bisa langsung semua ke Spot.
- **C** — Detailed Monitoring memberikan data 1-menit resolution (vs 5-menit basic). Berguna untuk diagnosis, tapi Compute Optimizer sudah menggunakan data yang ada. Bukan first step untuk right-sizing.

**Konsep yang diuji:** Compute Optimizer right-sizing, multi-dimensional analysis, CPU vs memory vs I/O bound, avoid premature termination.

---

## Soal 4 — Jawaban: A

**EMR Managed Scaling**

**Kenapa A:**
Spark jobs memiliki **wave pattern**: banyak tasks berjalan paralel di awal stage, kemudian berkurang saat mendekati akhir stage (straggler tasks). Selama periode straggler, mayoritas executor cores idle menunggu beberapa tasks selesai.

**EMR Managed Scaling:**
- Monitor **YARN metrics**: YARNMemoryAvailablePercentage, ContainerPendingRatio
- Scale down nodes yang YARN-nya idle
- Scale up saat ada containers pending
- Berlaku untuk **task nodes** (bukan core nodes yang menyimpan HDFS data)
- Gratis — tidak ada biaya tambahan untuk Managed Scaling

**Spark Dynamic Resource Allocation (tambahan):**
- Spark-level mechanism yang bisa **return executors** saat idle
- Executor yang tidak diperlukan dikembalikan ke YARN resource manager
- Bisa dikombinasikan dengan EMR Managed Scaling

Dengan Managed Scaling aktif, EMR scale down saat straggler phase → instances tidak perlu running saat idle → cost reduction.

**Kenapa bukan yang lain:**
- **B** — Instance Fleet untuk diversifikasi instance types di Spot Fleet context. Membantu availability, tidak langsung untuk scale down idle cores saat straggler phase.
- **C** — Auto Termination untuk terminate cluster setelah **semua jobs selesai** (cluster idle). Tidak membantu selama jobs masih berjalan tapi ada idle phase.
- **D** — Ini adalah deskripsi yang mirip dengan jawaban A — soal memang mengarah ke Managed Scaling sebagai fitur utama. Pilihan D adalah distractor dengan deskripsi yang overlap tapi kurang spesifik.

**Konsep yang diuji:** EMR Managed Scaling, YARN metrics, Spark straggler tasks, task node scale down, Dynamic Resource Allocation.

---

## Soal 5 — Jawaban: C

**Migrasi dari REST API ke HTTP API**

**Kenapa C:**
API Gateway menyediakan tiga jenis API dengan pricing berbeda:

| API Type | Price per million requests | Use case |
|---------|--------------------------|----------|
| REST API | $3.50 | Full features: WAF, X-Ray, private integrations, usage plans, API keys, request validation |
| HTTP API | $1.00 | Lambda, HTTP backends, JWT auth, OIDC — lebih simple |
| WebSocket API | $1.00 | Persistent bidirectional connections |

HTTP API **71% lebih murah** dari REST API. Untuk kebanyakan use cases modern (Lambda backend + JWT auth), HTTP API sudah cukup.

**Fitur yang HANYA ada di REST API** (tidak ada di HTTP API):
- AWS WAF integration
- AWS X-Ray tracing
- Private integration (VPC Link v1)
- Usage plans dan API keys dengan throttling
- Request/response validation
- Mock integration
- Custom gateway responses

Jika fitur-fitur di atas tidak digunakan, migrasi ke HTTP API adalah quick win yang signifikan.

**Kenapa bukan yang lain:**
- **A** — Caching mengurangi Lambda invocations (mengurangi Lambda cost), tapi biaya API Gateway per request tetap sama. Tidak mengurangi API Gateway cost itu sendiri.
- **B** — Mengurangi stages: stages di-charge berdasarkan requests, bukan jumlah stages. Menghapus stages dev/staging tidak mempengaruhi production request cost.
- **D** — Edge-optimized vs Regional: pricing sama ($3.50/million). Edge-optimized menggunakan CloudFront edge untuk routing — berguna untuk latency, bukan cost reduction.

**Konsep yang diuji:** API Gateway REST vs HTTP API pricing, HTTP API 71% cheaper, feature comparison, migration candidacy.

---

## Soal 6 — Jawaban: D

**Redshift Spectrum**

**Kenapa D:**
Redshift Spectrum memungkinkan **tiered storage architecture**:

```
Hot data (1–2 tahun): di Redshift cluster
    ↓ SQL JOIN
Cold data (3–5 tahun): di S3 (unloaded dari Redshift)
```

Query bisa **join kedua dataset** dalam satu SQL:
```sql
SELECT r.customer_id, r.revenue, h.historical_revenue
FROM recent_sales r
JOIN spectrum.historical_sales h ON r.customer_id = h.customer_id
WHERE r.year >= 2023 OR h.year BETWEEN 2019 AND 2022;
```

**Perbandingan biaya storage:**
- Redshift storage (ra3.4xlarge): ~$0.024/GB/bulan (managed storage)
- S3 Standard: $0.023/GB/bulan
- S3 Standard-IA: $0.0125/GB/bulan → gunakan ini untuk historical data

Spectrum di-charge per TB data yang di-scan di S3 ($5/TB) — sama seperti Athena.

**Kenapa bukan yang lain:**
- **A** — Hapus data historis: data audit diperlukan beberapa kali per tahun. Menghapus melanggar requirement dan regulatory compliance.
- **B** — Redshift cluster kedua: biaya double untuk cluster + overhead operasional manage dua clusters. Data tidak bisa di-join lintas cluster secara native.
- **C** — Arsipkan ke S3 dan query via Athena: valid, tapi kehilangan kemampuan **join dengan data Redshift aktif** dalam satu query. Tim data harus manage dua tools (Redshift + Athena). Redshift Spectrum memberikan single pane of glass.

**Konsep yang diuji:** Redshift Spectrum, tiered storage hot/cold, S3 for cold data, cross-data-source query, storage cost comparison.

---

## Soal 7 — Jawaban: D

**AWS Trusted Advisor**

**Kenapa D:**
Trusted Advisor adalah **automated best practices advisor** yang cover 5 kategori:

**Cost Optimization checks (relevan):**
- Low Utilization Amazon EC2 Instances (CPU < 10%, network < 5 MB/day)
- Idle Load Balancers (< 100 requests/day untuk 14 hari)
- Underutilized Amazon EBS Volumes (< 1 IOPS/day untuk 7 hari)
- Unassociated Elastic IP Addresses
- Idle Amazon RDS DB Instances
- Amazon Route 53 Latency Record Sets
- Savings Plans / Reserved Instance coverage

**Access:**
- **Basic support (free)**: 6 core checks (Security + Service Limits)
- **Developer support**: semua Security checks
- **Business/Enterprise support**: semua checks termasuk Cost Optimization, Performance, Fault Tolerance
- Atau aktifkan via AWS Security Hub dan AWS Config

**Kenapa bukan yang lain:**
- **A** — Cost Explorer untuk **analisis spending trends, forecast**, dan RI/Savings Plans recommendations berdasarkan usage. Tidak memberikan best practice checks seperti "idle load balancer" secara otomatis.
- **B** — AWS Budgets untuk **alert** ketika spending approach atau exceed threshold. Bukan untuk identify unused resources.
- **C** — CloudWatch untuk **operational metrics monitoring**. Bisa monitor CPU, IOPS, dll tapi tidak memberikan consolidated best practice recommendations.

**Konsep yang diuji:** AWS Trusted Advisor, 5 categories, Cost Optimization checks, support tier access.

---

## Soal 8 — Jawaban: D

**SES dari EC2 di region yang sama**

**Kenapa D:**
Amazon SES pricing:

**Jika dikirim dari EC2 di region yang sama:**
- **Pertama 62.000 email/bulan**: **GRATIS**
- Di atas 62.000: $0.10 per 1.000 emails
- Data transfer antara EC2 dan SES di region yang sama: **gratis**
- Attachment: $0.12 per GB attachment

**Jika dikirim dari luar AWS (atau region berbeda):**
- Tidak ada free tier 62.000
- Harga sama $0.10 per 1.000 emails tapi ada data transfer cost

Untuk 500.000 emails/minggu = 2.000.000 emails/bulan:
- Dari EC2 same region: (2.000.000 - 62.000) ÷ 1.000 × $0.10 = **$193.8/bulan**
- Tanpa free tier: 2.000.000 ÷ 1.000 × $0.10 = $200/bulan
- Penghematan dari free tier: $6.2/bulan (kecil tapi ada)

SES secara keseluruhan sudah sangat murah dibanding provider lain. Mengirim dari EC2 same region memaksimalkan free tier dan eliminasi data transfer cost.

**Kenapa bukan yang lain:**
- **A** — Third-party email provider (Mailchimp, SendGrid): jauh lebih mahal untuk volume tinggi. SES $0.10/1000 adalah salah satu harga terendah di industri.
- **B** — Kurangi frekuensi: mengubah business requirement, bukan technical optimization.
- **C** — Dedicated IP untuk SES: biaya tambahan $24.95/bulan per dedicated IP untuk warming dan improved deliverability. Tidak mengurangi per-email cost.

**Konsep yang diuji:** SES pricing, free tier 62.000 emails/bulan dari EC2, data transfer optimization, SES cost comparison.

---

## Soal 9 — Jawaban: C

**AWS Resource Explorer**

**Kenapa C:**
AWS Resource Explorer adalah **multi-region resource discovery service**:
- Index dan search semua resources di semua regions dalam satu akun
- Filter berdasarkan: resource type, region, tag key/value, account
- Identify resources **tanpa tag** → kemungkinan forgotten/unmanaged resources
- Resource types yang di-support: EC2, RDS, S3, ELB, VPC, Lambda, dll.

Untuk cost review audit:
1. Search `resourcetype:aws:ec2:instance` → lihat semua EC2 di semua regions
2. Filter `tag:!Environment` → temukan EC2 tanpa tag environment (potentially abandoned)
3. Cross-reference dengan Trusted Advisor recommendations

**Kenapa bukan yang lain:**
- **A** — AWS Config track **configuration history** dan compliance. Bisa lihat resource attributes, tapi interface untuk discover resources tidak se-user-friendly Resource Explorer. Config lebih untuk compliance auditing.
- **B** — CloudTrail untuk **API call audit** — siapa yang buat resource, kapan, dari IP mana. Berguna untuk forensic, bukan untuk inventory discovery.
- **D** — Amazon Inspector untuk **vulnerability scanning** (CVE, network reachability). Bukan untuk discover unused resources.

**Konsep yang diuji:** AWS Resource Explorer, multi-region resource discovery, resource inventory, untagged resources identification.

---

## Soal 10 — Jawaban: D

**EC2 Savings Plans untuk P3 instances reguler**

**Kenapa D:**
Untuk ML training yang dijalankan **beberapa kali per minggu secara reguler**, ada dua komponen biaya:

**1. Regular scheduled training (predictable):**
- Pola reguler (misalnya Senin/Rabu/Jumat) = predictable usage
- **Compute Savings Plans** atau **EC2 Instance Savings Plans** memberikan diskon 30–50% vs On-Demand
- Tidak butuh commitment ke specific instance — Compute Savings Plans cover semua instance families termasuk P3

**2. Ad-hoc / experimental training (unpredictable):**
- Tetap gunakan **Spot Instances** untuk maksimum savings
- Dengan checkpoint, Spot interruption tidak hilangkan progress

**Strategi gabungan:**
```
Regular training jobs → Savings Plans (On-Demand price dengan diskon)
Experimental jobs → Spot Instances (70-90% lebih murah, toleran interrupt)
```

**Kenapa bukan yang lain:**
- **A** — CloudFront untuk cache model artifacts: CloudFront untuk distribusi content, bukan untuk compute cost reduction. Training jobs tidak melibatkan model delivery ke users.
- **B** — SageMaker vs EC2 langsung: SageMaker memiliki overhead ~20–25% vs EC2 On-Demand (SageMaker charge = EC2 + managed service fee). Untuk GPU training, SageMaker bisa lebih mahal dari EC2 direct, meskipun operasional lebih mudah.
- **C** — S3 Standard-IA untuk checkpoints: checkpoint biasanya berukuran GB, diakses dalam hitungan jam (jika ada interruption). Standard-IA memiliki minimum 30-hari duration dan retrieval fee. Untuk checkpoints yang mungkin diakses dalam jam, S3 Standard lebih tepat. Penghematan storage minimal vs risiko retrieval cost.

**Konsep yang diuji:** Savings Plans untuk predictable GPU workloads, combined strategy Savings Plans + Spot, EC2 vs SageMaker pricing, ML training cost optimization.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | S3 partitioning, partition pruning, Athena scan cost |
| 2 | Kinesis On-Demand mode vs Provisioned shards |
| 3 | Compute Optimizer right-sizing, multi-dimensional analysis |
| 4 | EMR Managed Scaling, Spark straggler tasks |
| 5 | API Gateway HTTP API vs REST API, 71% cheaper |
| 6 | Redshift Spectrum, tiered hot/cold storage |
| 7 | AWS Trusted Advisor, 5 categories, Cost Optimization checks |
| 8 | SES pricing, 62.000 free emails dari EC2 |
| 9 | AWS Resource Explorer, multi-region resource discovery |
| 10 | Savings Plans + Spot kombinasi untuk ML training |
