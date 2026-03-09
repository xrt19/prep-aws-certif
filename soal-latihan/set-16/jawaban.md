# Jawaban — Set 16
## Domain: High-Performing Architectures

---

## Soal 1 — Jawaban: C

**EC2 Spot Instances**

**Kenapa C:**
Spot Instances menawarkan kapasitas EC2 yang tidak terpakai dengan harga hingga **90% lebih murah** dari On-Demand:
- AWS bisa **interrupt** Spot Instance dengan 2 menit notice jika butuh kapasitas kembali
- Ideal untuk workloads yang **fault-tolerant** dan bisa di-checkpoint: batch processing, ML training, rendering, genomics, CI/CD
- Strategi umum: checkpoint state ke S3 setiap beberapa menit, saat instance di-interrupt re-launch dan resume dari checkpoint
- Bisa kombinasikan dengan ASG **Spot Fleet** atau **EC2 Fleet** untuk diversifikasi ke berbagai instance types/AZs, meningkatkan availability

**Kenapa bukan yang lain:**
- **A** — On-Demand lebih mahal hingga 10x dibanding Spot untuk workload yang sama. Tidak optimal untuk budget-sensitive batch jobs.
- **B** — Reserved Instances untuk workloads yang berjalan secara konsisten 24/7 sepanjang tahun. Batch jobs yang hanya butuh burst compute tidak cocok — waste commitment jika tidak running.
- **D** — Dedicated Hosts untuk compliance licensing (SQL Server, Oracle per-socket/core licensing). Paling mahal di antara semua opsi.

**Konsep yang diuji:** EC2 Spot Instances, use cases, interruption handling, fault-tolerant batch workloads, pricing model comparison.

---

## Soal 2 — Jawaban: C

**Global Secondary Index (GSI)**

**Kenapa C:**
DynamoDB partition key menentukan di node mana data disimpan — query tanpa partition key memerlukan full table scan yang sangat mahal. **GSI** memungkinkan membuat "virtual table" dengan partition key dan sort key yang berbeda:

```
Table primary key: order_id (partition key)

GSI 1: status (partition key) + created_at (sort key)
→ Bisa query: WHERE status = 'PENDING' ORDER BY created_at DESC

GSI 2: user_id (partition key) + status (sort key)
→ Bisa query: WHERE user_id = 123 AND status = 'ACTIVE'
```

GSI menyimpan subset atau semua atribut (projection) dan di-charge terpisah untuk capacity dan storage.

**Kenapa bukan yang lain:**
- **A** — DAX adalah in-memory cache yang mempercepat read latency dari milidetik ke mikrodetik, tapi tidak membantu jika query pattern itu sendiri membutuhkan full table scan — caching query yang tidak efisien tetap inefficient.
- **B** — DynamoDB Streams untuk event-driven processing (trigger Lambda saat ada perubahan data), bukan untuk mengoptimalkan query patterns.
- **D** — On-Demand capacity untuk absorb unpredictable traffic spikes. Tidak membantu query efficiency — query inefficiency bukan masalah throughput, tapi masalah akses patterns.

**Konsep yang diuji:** DynamoDB GSI (Global Secondary Index), query patterns, projection, access pattern design.

---

## Soal 3 — Jawaban: B

**S3 Select**

**Kenapa B:**
S3 Select memungkinkan **push-down filter ke S3** — query dieksekusi langsung di S3 storage layer:
- Support CSV, JSON, dan Parquet (termasuk compressed: GZIP, BZIP2)
- Hanya bytes yang match filter yang dikirim via network — bukan seluruh file
- Contoh: file CSV 10 GB dengan 100 juta baris, query hanya butuh 50.000 baris → S3 Select hanya transfer ~5 MB bukan 10 GB
- Jauh lebih murah (data transfer dibayar per GB yang di-scan, bukan per GB yang di-return)
- Bisa digunakan dari AWS CLI, SDK, Lambda, atau Athena

**Kenapa bukan yang lain:**
- **A** — S3 Byte-Range Fetches untuk retrieve **sebagian file berdasarkan byte offset** (misalnya byte 1.000.000 sampai 2.000.000). Tidak bisa filter berdasarkan nilai kolom CSV — hanya bisa fetch range byte arbitrer dari binary file.
- **C** — S3 Batch Operations untuk menjalankan operasi (copy, tag, invoke Lambda, restore from Glacier) pada **banyak objek yang sudah ada** di S3. Bukan untuk query konten objek.
- **D** — S3 Inventory untuk generate laporan daftar objek di bucket (nama file, size, last modified, storage class, dll). Tidak ada fungsi query data.

**Konsep yang diuji:** S3 Select, SQL query on S3 objects, CSV/JSON/Parquet, push-down filter, data transfer optimization.

---

## Soal 4 — Jawaban: C

**Distribution key dan sort key yang tepat**

**Kenapa C:**
Redshift adalah massively parallel processing (MPP) database yang mendistribusikan data ke **compute nodes**. Optimasi fundamental:

**Distribution key (DISTKEY):**
- Menentukan bagaimana rows di-distribute ke nodes
- Pilih kolom yang sering di-JOIN — jika dua tabel di-JOIN pada kolom yang sama distribution key-nya, data yang perlu di-JOIN ada di node yang sama → tidak perlu network shuffle
- `DISTYLE EVEN` (round-robin), `DISTYLE ALL` (copy ke semua nodes, untuk dimension tables kecil), `DISTYLE KEY` (hash by column)

**Sort key (SORTKEY):**
- Data fisik di-sort berdasarkan sort key di dalam tiap node
- Redshift menyimpan **zone maps** (min/max value per block) → query dengan range filter bisa skip blocks yang tidak relevan tanpa membaca data
- Compound sort key vs Interleaved sort key

**Kenapa bukan yang lain:**
- **A** — Redshift tidak memiliki fitur Read Replica seperti RDS. Redshift punya **Redshift Concurrency Scaling** (tambah cluster capacity saat ada query burst) dan **RA3 node dengan managed storage**, tapi bukan "Read Replica" dalam konteks ini.
- **B** — Redshift Serverless untuk workloads yang intermittent/unpredictable, bukan khusus untuk query optimization.
- **D** — Pindah ke Athena adalah perubahan arsitektur besar. Soal minta optimasi "tanpa menambah node" — perbaikan distribution/sort key tidak butuh perubahan infrastruktur.

**Konsep yang diuji:** Redshift distribution key, sort key, zone maps, MPP architecture, query optimization.

---

## Soal 5 — Jawaban: C

**Amazon OpenSearch Service**

**Kenapa C:**
Amazon OpenSearch Service (sebelumnya Elasticsearch Service) adalah managed service untuk **search dan analytics**:
- **Full-text search** — inverted index memungkinkan search kata dalam dokumen dengan sangat cepat
- **Faceted search** — filter dan aggregate berdasarkan multiple dimensions (category, price range, brand) dalam satu query
- **Auto-complete dan suggestions** — completion suggester untuk real-time search suggestions
- **Near real-time indexing** — dokumen bisa dicari dalam detik setelah diindex
- **Complex aggregations** — histogram, terms aggregation untuk facets
- Common pattern: data di-index ke OpenSearch dari DynamoDB/RDS via Lambda + DynamoDB Streams atau Kinesis

**Kenapa bukan yang lain:**
- **A** — MySQL FULLTEXT index sangat terbatas untuk production search: tidak support scoring/relevance, tidak ada faceting native, performance degraded untuk large datasets. LIKE query bahkan lebih buruk — tidak bisa menggunakan index.
- **B** — DynamoDB GSI untuk access pattern berdasarkan specific attributes (equality filter), bukan untuk full-text search. Query `WHERE description CONTAINS 'laptop gaming'` tidak bisa efisien di DynamoDB.
- **D** — Redshift untuk OLAP analytics pada structured tabular data. Bukan dirancang untuk search dengan inverted index, faceting, atau real-time search suggestions.

**Konsep yang diuji:** Amazon OpenSearch, full-text search, faceted search, inverted index, search vs analytics use cases.

---

## Soal 6 — Jawaban: C

**API Gateway caching**

**Kenapa C:**
API Gateway memiliki built-in **response caching** di stage level:
- Cache key: HTTP method + URL path + query string parameters + (optionally) headers/cookies
- Responses di-cache di dedicated cache (0.5 GB hingga 237 GB)
- **TTL** bisa dikonfigurasi (default 300 detik, max 3600 detik)
- Bila cache hit: API Gateway langsung return response tanpa invoke Lambda → latency turun dari ~10–50ms ke <1ms
- Efektif untuk: GET endpoints dengan stable data, reference data, product catalogs

Ini langsung mengurangi Lambda invocations (cost) dan database queries (load).

**Kenapa bukan yang lain:**
- **A** — Usage Plans untuk rate limiting dan throttling per API key/customer. Mengatur jumlah requests yang diizinkan, bukan mengoptimalkan performance repeated requests.
- **B** — Custom Domain untuk branded URL (api.yourdomain.com alih-alih <random>.execute-api.region.amazonaws.com). Tidak ada hubungannya dengan response caching.
- **D** — WebSocket API untuk persistent bidirectional connections (chat, real-time dashboard). Ganti dari REST ke WebSocket adalah perubahan arsitektur besar yang tidak relevan dengan caching repeated REST requests.

**Konsep yang diuji:** API Gateway caching, TTL, cache key, latency reduction, Lambda invocation reduction.

---

## Soal 7 — Jawaban: C

**Naikkan memory allocation**

**Kenapa C:**
Di Lambda, **CPU power yang diberikan proporsional dengan memory**:
- 128 MB memory → ~0.083 vCPU
- 1769 MB memory → 1 vCPU penuh
- 10240 MB memory → ~6 vCPU

Untuk CPU-intensive workload, menaikkan memory **langsung meningkatkan CPU** yang tersedia. Meskipun biaya per GB-second naik, total cost bisa **turun** jika execution time berkurang lebih dari proporsi kenaikan memory.

Contoh: 512 MB × 10 detik = 5120 MB-seconds. Jika naik ke 2048 MB (4x memory) dan execution time turun ke 2 detik = 4096 MB-seconds — lebih murah sekaligus lebih cepat.

**AWS Lambda Power Tuning** (open-source tool) bisa membantu mencari konfigurasi memory optimal.

**Kenapa bukan yang lain:**
- **A** — Lambda Layers untuk share code/libraries antar functions (deployment package reuse). Tidak mempengaruhi CPU/memory yang di-alokasikan ke function.
- **B** — Upgrade runtime memang ada micro-optimizations di runtime startup, tapi tidak secara signifikan mengubah CPU capacity untuk CPU-intensive workloads.
- **D** — Reserved Concurrency untuk mengontrol berapa banyak concurrent executions yang diizinkan — bisa mencegah throttling atau melindungi downstream systems. Tidak mempengaruhi performance per-invocation.

**Konsep yang diuji:** Lambda power tuning, memory-CPU relationship, execution time vs cost trade-off, performance optimization.

---

## Soal 8 — Jawaban: D

**Naikkan CPU unit allocation di task definition**

**Kenapa D:**
Di ECS Fargate, resources di-specify di **task level** (dan optionally di container level):
- CPU units: 256 (.25 vCPU) hingga 16384 (16 vCPU) — tergantung memory yang dipilih
- Jika CPU units terlalu rendah untuk workload yang CPU-intensive, container akan **CPU throttled** — OS-level CFS (Completely Fair Scheduler) throttling
- CPU throttling menyebabkan proses yang seharusnya selesai dalam 30 detik bisa menjadi 5 menit

Gejala yang benar teridentifikasi: CPU throttled meskipun memory masih banyak → root cause adalah CPU allocation, bukan memory.

**Kenapa bukan yang lain:**
- **A** — Menambah lebih banyak tasks mendistribusikan load, tapi setiap task tetap CPU throttled. Biaya naik tapi setiap individual task tetap lambat.
- **B** — Pindah ke EC2 launch type tidak secara langsung menyelesaikan masalah CPU allocation. Di EC2 launch type pun, task definition CPU limits berlaku.
- **C** — Naikkan memory limit tidak akan membantu jika bottleneck adalah CPU, bukan memory. Di Fargate, CPU dan memory di-alokasikan independent (dalam kombinasi yang valid).

**Konsep yang diuji:** ECS Fargate task definition, CPU units, CPU throttling, resource allocation, container performance tuning.

---

## Soal 9 — Jawaban: C

**Predictive Scaling**

**Kenapa C:**
**Predictive Scaling** menggunakan machine learning untuk menganalisis load patterns historis (minimum 2 minggu data) dan **forecast kapan traffic akan naik**, kemudian pre-scale sebelum event tersebut terjadi:
- Cocok untuk cyclical patterns: diurnal (daily), weekly, seasonal
- Untuk traffic yang naik setiap weekday jam 09:00: Predictive Scaling akan launch instances beberapa menit sebelum jam 09:00 agar instances sudah warm dan siap
- Dikombinasikan dengan target tracking untuk handle unexpected deviations dari forecast

Ini berbeda dengan reactive scaling (target tracking, step scaling) yang menunggu metric naik dulu baru scale — ada delay dari metric naik → alarm trigger → launch → warm-up → serving traffic.

**Kenapa bukan yang lain:**
- **A** — Step Scaling juga reactive — trigger berdasarkan CloudWatch alarm threshold yang sudah terlampaui. Lebih granular dari Simple Scaling tapi masih reaktif.
- **B** — Simple Scaling paling basic — scale dengan fixed capacity setelah alarm, lalu cooldown period. Tidak ada forecasting.
- **D** — Lifecycle Hooks untuk pause instance launch/terminate agar custom action bisa dijalankan (pre-warm, drain connections). Bukan strategi untuk proactive pre-scaling.

**Konsep yang diuji:** Predictive Scaling, forecast-based pre-scaling, cyclical traffic patterns, proactive vs reactive scaling.

---

## Soal 10 — Jawaban: D

**Elastic Fabric Adapter (EFA)**

**Kenapa D:**
EFA adalah network interface untuk EC2 yang menyediakan:
- **OS-bypass** — aplikasi MPI bisa berkomunikasi langsung dengan EFA hardware tanpa melalui OS kernel network stack → drastis mengurangi latency
- **RDMA (Remote Direct Memory Access)** — data bisa di-transfer langsung ke/dari memory node lain tanpa CPU overhead
- Latency: **single-digit microseconds** (vs tens of microseconds untuk standard Enhanced Networking)
- Throughput: hingga 400 Gbps untuk instance types tertentu (hf1, p4d.24xlarge)
- Tersedia di instance types khusus HPC
- Digunakan dengan software MPI seperti OpenMPI, Intel MPI, dan NVIDIA NCCL (untuk distributed ML training)

**Kenapa bukan yang lain:**
- **A** — Enhanced Networking (ENA) adalah dasar untuk high-bandwidth networking di EC2. EFA adalah layer di atas ENA yang menambahkan OS-bypass dan RDMA capabilities. ENA saja tidak cukup untuk MPI workloads yang butuh latency minimal.
- **B** — Jumbo Frames (MTU 9001) meningkatkan efficiency transfer data besar dengan mengurangi overhead per-packet. Membantu throughput tapi tidak secara fundamental mengurangi latency seperti OS-bypass EFA.
- **C** — Direct Connect untuk koneksi dedicated dari **on-premise** ke AWS. Tidak relevan untuk komunikasi antar EC2 instances dalam VPC/Placement Group.

**Konsep yang diuji:** EFA (Elastic Fabric Adapter), OS-bypass, RDMA, MPI workloads, HPC networking, EFA vs ENA.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | EC2 Spot Instances, fault-tolerant batch, pricing models |
| 2 | DynamoDB GSI, query patterns, access pattern design |
| 3 | S3 Select, SQL query on S3, push-down filter |
| 4 | Redshift distribution key, sort key, zone maps |
| 5 | Amazon OpenSearch, full-text search, faceted search |
| 6 | API Gateway caching, TTL, Lambda invocation reduction |
| 7 | Lambda power tuning, memory-CPU relationship |
| 8 | ECS Fargate CPU units, CPU throttling, task definition |
| 9 | Predictive Scaling, forecast, cyclical traffic patterns |
| 10 | EFA, OS-bypass, RDMA, MPI HPC networking |
