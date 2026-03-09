# Jawaban — Set 18
## Domain: High-Performing Architectures

---

## Soal 1 — Jawaban: C

**Dedikasikan satu Read Replica khusus untuk analytics**

**Kenapa C:**
RDS Read Replica adalah **asynchronous replica** yang bisa melayani read-only queries. Strategi dedicated replica untuk analytics:
- Satu atau lebih replicas khusus untuk OLTP reads (aplikasi production)
- Satu replica terpisah khusus untuk analytics/reporting queries
- Replica analytics bisa di-scale **instance type yang berbeda** — misal pakai r6g.2xlarge (memory-optimized) untuk analytics yang butuh banyak RAM, sementara OLTP pakai instance lebih kecil
- Analytics queries yang berjalan lama tidak mempengaruhi performance OLTP sama sekali
- Replicas bisa berada di region berbeda (cross-region) untuk analytics di region yang lebih dekat ke tim data

**Kenapa bukan yang lain:**
- **A** — "Distribusikan semua queries" tanpa strategi yang jelas tidak optimal. OLTP queries perlu low-latency, analytics queries butuh resource besar — mencampurnya di replica yang sama tetap bisa saling ganggu.
- **B** — **Exam trap klasik**: RDS Multi-AZ standby **tidak melayani read queries**. Standby hanya untuk automatic failover. Ini adalah salah satu trap paling sering di SAA-C03.
- **D** — RDS Proxy untuk **connection pooling** — mengurangi jumlah koneksi ke database, bukan untuk routing queries ke primary vs replica berdasarkan tipe workload.

**Konsep yang diuji:** RDS Read Replica isolation, dedicated replica per workload, instance type flexibility, Multi-AZ tidak serve reads (exam trap).

---

## Soal 2 — Jawaban: C

**CloudFront Functions**

**Kenapa C:**
CloudFront Functions adalah JavaScript runtime yang berjalan di **setiap edge location CloudFront**, di **viewer request/response stage**:

| | CloudFront Functions | Lambda@Edge |
|-|---------------------|-------------|
| Stage | Viewer req/resp only | Viewer + Origin req/resp |
| Runtime | JavaScript (ES5.1) | Node.js, Python |
| Max exec time | 1ms | 5s (viewer), 30s (origin) |
| Memory | 2 MB | 128 MB – 10 GB |
| Scale | Millions req/s | Thousands req/s |
| Price | $0.10/million | $0.60/million + duration |
| Use case | URL rewrite, redirect, header manip | Complex logic, auth, A/B testing |

Untuk redirect sederhana berdasarkan URL path lookup, CloudFront Functions lebih dari cukup dan **6x lebih murah** dari Lambda@Edge.

**Kenapa bukan yang lain:**
- **A** — Lambda + API Gateway untuk logic ini adalah over-engineering yang mahal. Setiap redirect butuh round-trip ke Lambda di specific region (tidak di edge), latency lebih tinggi.
- **B** — CloudFront Origin Groups untuk **origin failover** — jika primary origin gagal, traffic di-route ke secondary origin. Tidak bisa melakukan URL redirect logic.
- **D** — Lambda@Edge bisa melakukan ini, tapi lebih mahal dan lebih complex dari CloudFront Functions untuk use case sesederhana ini. Lambda@Edge diperlukan jika butuh akses ke origin request/response atau logic lebih complex.

**Konsep yang diuji:** CloudFront Functions vs Lambda@Edge, use case selection, edge computing, cost comparison.

---

## Soal 3 — Jawaban: C

**AWS Database Migration Service (DMS)**

**Kenapa C:**
AWS DMS adalah service untuk migrasi database dengan **minimal downtime**:

**Full Load + CDC workflow:**
1. **Full Load phase**: DMS copy semua existing data dari Oracle ke target (Aurora PostgreSQL)
2. **CDC (Change Data Capture) phase**: DMS capture dan replikasi semua changes yang terjadi di Oracle selama dan setelah full load (INSERT, UPDATE, DELETE)
3. Kedua database berjalan **parallel** — Oracle masih production, Aurora sudah terisi dan terus di-sync
4. Saat lag CDC sangat kecil (detik atau kurang): **cutover** — switch aplikasi ke Aurora, matikan Oracle
5. Total downtime: hanya saat cutover, bisa < 1 menit

DMS support **heterogeneous migration** (Oracle → Aurora PostgreSQL) dengan AWS Schema Conversion Tool (SCT) untuk convert schema dan stored procedures.

**Kenapa bukan yang lain:**
- **A** — Snowball Edge untuk transfer data **dalam jumlah sangat besar** (petabytes) dari on-premise ke S3 via physical device. Tidak support ongoing CDC replication untuk live database migration.
- **B** — AWS DataSync untuk transfer file/object antara on-premise storage dan AWS storage (S3, EFS, FSx). Bukan untuk database migration dengan CDC.
- **D** — `pg_dump` adalah full export yang butuh downtime penuh (stop database, export, import, start). Tidak support Oracle format langsung dan tidak ada CDC.

**Konsep yang diuji:** AWS DMS, CDC (Change Data Capture), heterogeneous migration, minimal downtime migration pattern, full load + ongoing replication.

---

## Soal 4 — Jawaban: C

**Lambda Layers**

**Kenapa C:**
Lambda Layers adalah mekanisme untuk **share code dan dependencies** antar Lambda functions:
- Package FFmpeg sebagai ZIP file dan publish sebagai Layer
- Multiple functions bisa reference Layer yang sama — tidak perlu duplicate di setiap deployment package
- Layer di-extract ke `/opt/` di execution environment dan di-cache — tidak perlu download ulang setiap invocation
- **Size limits**: deployment package (code + dependencies) max 50 MB zipped, tapi dengan Layers total unzipped size bisa mencapai **250 MB**
- Bisa share Layers antar AWS accounts (public atau private sharing)
- Update Layer versi tanpa re-deploy functions yang menggunakannya

**Kenapa bukan yang lain:**
- **A** — Include langsung di deployment package: (1) Setiap function punya copy FFmpeg sendiri → storage waste; (2) Deployment package limit 50 MB zipped — FFmpeg 150 MB tidak akan muat.
- **B** — Download dari S3 setiap cold start: (1) Tambah latency cold start secara signifikan; (2) S3 request cost; (3) Network transfer dari S3 tiap init.
- **D** — Lambda mendukung binary dependencies natively — ini adalah use case yang valid untuk Lambda. ECS adalah over-engineering untuk use case ini.

**Konsep yang diuji:** Lambda Layers, shared dependencies, size limits, deployment package optimization.

---

## Soal 5 — Jawaban: C

**Amazon MSK (Managed Streaming for Apache Kafka)**

**Kenapa C:**
Amazon MSK adalah **fully managed Apache Kafka** service:
- **Fully compatible dengan Kafka APIs** — producer, consumer, Kafka Streams, Kafka Connect code tidak perlu diubah
- AWS handle: broker provisioning, patching, OS updates, broker failure replacement, ZooKeeper/KRaft management, storage scaling
- Monitoring via CloudWatch, broker metrics, consumer lag metrics
- Multi-AZ by default (3 AZs)
- Support MSK Serverless untuk variable workloads tanpa manage capacity
- Encryption in-transit (TLS) dan at-rest (KMS)

**Kenapa bukan yang lain:**
- **A** — SQS adalah message queue (point-to-point atau fan-out via SNS). Tidak compatible dengan Kafka APIs — perlu rewrite semua producer/consumer code. Juga tidak support Kafka concepts seperti topics, partitions, consumer groups, offset management, replay.
- **B** — Kinesis Data Streams juga managed streaming, tapi **tidak compatible dengan Kafka APIs**. Kinesis punya SDK/API sendiri — butuh rewrite code. Shard model vs Kafka partition model juga berbeda.
- **D** — EventBridge adalah serverless event bus untuk event-driven architecture. Tidak support Kafka APIs, tidak ada concept topics/partitions/consumer groups.

**Konsep yang diuji:** Amazon MSK, Kafka compatibility, managed Kafka, migration from self-managed Kafka, API compatibility.

---

## Soal 6 — Jawaban: C

**RDS Proxy**

**Kenapa C:**
Ini adalah use case **utama** RDS Proxy — Lambda yang frequently invoke ke RDS:

**Masalah:** Lambda bersifat stateless dan ephemeral. Setiap invocation bisa membuat koneksi database baru. Dengan ribuan concurrent invocations, bisa ada ribuan koneksi database simultan yang melebihi `max_connections` RDS.

**RDS Proxy solusi:**
- RDS Proxy **pool dan share** koneksi database
- 1000 Lambda invocations bisa share, misalnya, 50 koneksi ke RDS via proxy
- Proxy maintain koneksi ke RDS dalam pool — idle connections tidak langsung di-close
- Jika RDS failover, proxy handle reconnection secara transparan ke aplikasi
- Support IAM authentication untuk Lambda ke RDS Proxy

**Kenapa bukan yang lain:**
- **A** — Naikkan `max_connections` bisa sedikit membantu, tapi ada batas fisik (tergantung instance RAM). Dan banyak koneksi idle tetap konsumsi RDS memory. Bukan solusi untuk ribuan concurrent connections.
- **B** — PgBouncer di Lambda code: Lambda stateless, setiap execution environment punya connection pool sendiri. Tidak ada shared pool antar invocations di execution environments berbeda.
- **D** — Pindah ke public subnet adalah security anti-pattern — database tidak boleh di public subnet. Dan ini tidak menyelesaikan masalah too many connections sama sekali.

**Konsep yang diuji:** RDS Proxy, connection pooling, Lambda + RDS pattern, max connections problem.

---

## Soal 7 — Jawaban: C

**S3 Event Notification → SQS → Lambda**

**Kenapa C:**
Ini adalah pattern **event-driven processing** yang robust:

```
S3 (file upload) → S3 Event Notification → SQS Queue → Lambda (trigger)
```

**Kenapa SQS di tengah (bukan langsung S3 → Lambda):**
- **Buffering**: jika banyak file tiba bersamaan, SQS menyimpan events dalam queue — Lambda di-trigger sesuai capacity-nya, tidak overwhelmed
- **Retry**: jika Lambda gagal proses, message kembali ke queue dan di-retry (berdasarkan visibility timeout)
- **DLQ (Dead Letter Queue)**: setelah beberapa kali retry gagal, message di-move ke DLQ untuk investigasi
- **Decoupling**: S3 tidak perlu tahu Lambda; scaling Lambda tidak mempengaruhi S3

**Kenapa bukan yang lain:**
- **A** — Scheduled polling setiap 5 menit: (1) Latency hingga 5 menit; (2) Setiap poll harus list objects S3 untuk cari yang baru → tidak efisien; (3) Sulit track mana yang sudah diproses vs belum.
- **B** — S3 Inventory harian untuk batch processing berarti latency 24 jam. Tidak "event-driven" dan tidak suitable untuk pipeline yang harus respond cepat.
- **D** — EC2 continuously polling S3 adalah waste — instance harus running 24/7 meskipun tidak ada file baru. Biaya tinggi, tidak scalable.

**Konsep yang diuji:** S3 Event Notification, SQS as buffer, event-driven architecture, Lambda trigger patterns, decoupling.

---

## Soal 8 — Jawaban: C

**Tambahkan CloudFront di depan ALB**

**Kenapa C:**
CloudFront sebagai **CDN di depan dynamic application** (bukan hanya S3):
- CloudFront bisa cache responses dari **ALB** berdasarkan cache behavior rules
- Static content (images, CSS, JS) dengan Cache-Control headers → CloudFront cache di edge
- Dynamic requests (API calls, personalized content) → CloudFront forward ke ALB (no-cache)
- User di seluruh dunia di-serve static content dari edge terdekat — tidak ada round-trip ke ALB
- Reduce load pada EC2 instances secara signifikan untuk static content

**Konfigurasi:**
- Buat CloudFront distribution dengan ALB sebagai origin
- Buat **cache behaviors**: `/static/*` → cache 24 jam; `/*` → no-cache, forward ke ALB

**Kenapa bukan yang lain:**
- **A** — Tambah EC2 instances menambah compute capacity, tapi masalahnya adalah setiap request untuk static content tetap hit EC2. Scale out tidak mengurangi beban per-request.
- **B** — ALB tidak memiliki fitur caching. ALB hanya melakukan load balancing — forward requests ke target instances.
- **D** — EFS memang bisa jadi shared storage untuk static content antar EC2, tapi request tetap harus masuk ke EC2 untuk serve file tersebut. EFS tidak mengurangi jumlah requests yang masuk ke EC2.

**Konsep yang diuji:** CloudFront in front of ALB, cache behaviors, static vs dynamic content separation, CDN for application performance.

---

## Soal 9 — Jawaban: D

**Amazon FSx for Lustre**

**Kenapa D:**
FSx for Lustre adalah **high-performance parallel file system** yang di-manage AWS:
- Throughput: hingga **ratusan GB/s** dan jutaan IOPS
- Latency: **sub-millisecond** (< 1ms) untuk first-byte latency
- **POSIX-compliant** — aplikasi HPC yang berjalan di Linux bisa mount dan akses seperti local filesystem
- **Parallel I/O** — ratusan clients bisa baca/tulis secara simultan karena data di-stripe ke banyak storage servers
- **S3 integration**: bisa import data dari S3 bucket dan export hasil kembali ke S3 secara transparent
- Cocok untuk: genomics sequencing, computational fluid dynamics, financial risk modeling, video rendering, ML training data

**Kenapa bukan yang lain:**
- **A** — EFS adalah NFS managed service yang bagus untuk general Linux workloads dan web applications. Throughput maksimum EFS (Elastic throughput) dalam GB/s, bukan ratusan GB/s seperti FSx for Lustre. Latency EFS juga lebih tinggi (low single-digit ms vs sub-ms).
- **B** — S3 adalah object storage — tidak menyediakan POSIX interface. Aplikasi HPC yang expect filesystem semantics (open, read, write, seek) tidak bisa langsung pakai S3 tanpa adapter.
- **C** — EBS Multi-Attach memungkinkan attach satu EBS volume ke beberapa instances (maksimum 16 instances), tapi hanya untuk **io1/io2 Block Express** dan butuh cluster-aware file system. Tidak scalable ke ratusan instances dan throughput jauh lebih rendah dari FSx for Lustre.

**Konsep yang diuji:** FSx for Lustre, HPC file system, parallel I/O, POSIX interface, S3 integration, FSx vs EFS vs EBS.

---

## Soal 10 — Jawaban: D

**RDS Performance Insights**

**Kenapa D:**
RDS Performance Insights adalah **database performance monitoring** yang built-in ke RDS:
- **DB Load** metric: visualisasi berapa banyak queries yang berjalan vs database capacity
- **Top SQL** — ranking query berdasarkan contribution ke DB load, average latency, execution count
- **Wait Events** — breakdown waktu query dihabiskan untuk apa: CPU, I/O wait, lock wait, dll
- Tersedia di RDS MySQL, PostgreSQL, Oracle, SQL Server, Aurora
- **Tidak perlu install agent** — terintegrasi langsung di RDS engine
- Free tier: 7 hari retention; berbayar: hingga 2 tahun retention

Dari Performance Insights, tim bisa langsung identifikasi: "Query X running 10.000x sehari dengan avg latency 5 detik" → prioritas untuk dioptimasi dengan index atau query rewrite.

**Kenapa bukan yang lain:**
- **A** — CloudWatch Metrics untuk RDS menyediakan **infrastructure metrics**: CPUUtilization, DatabaseConnections, ReadIOPS, WriteIOPS, FreeStorageSpace. Tidak ada breakdown per query SQL — tidak bisa tahu query mana yang lambat.
- **B** — RDS Enhanced Monitoring menyediakan **OS-level metrics** (per-process CPU, memory, I/O, swap) yang lebih granular dari standard CloudWatch, tapi masih di level OS, bukan per SQL query.
- **C** — CloudTrail merecord **API calls** ke RDS service (CreateDBInstance, ModifyDBInstance, dll). Bukan untuk monitoring SQL query performance sama sekali.

**Konsep yang diuji:** RDS Performance Insights, DB load, Top SQL, wait events, query performance monitoring vs infrastructure monitoring.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | RDS Read Replica dedicated per workload, Multi-AZ tidak serve reads |
| 2 | CloudFront Functions vs Lambda@Edge, simple redirect |
| 3 | AWS DMS, CDC, heterogeneous migration, minimal downtime |
| 4 | Lambda Layers, shared dependencies, size limits |
| 5 | Amazon MSK, managed Kafka, API compatibility |
| 6 | RDS Proxy, connection pooling, Lambda + RDS pattern |
| 7 | S3 Event Notification → SQS → Lambda, event-driven pipeline |
| 8 | CloudFront in front of ALB, static vs dynamic caching |
| 9 | FSx for Lustre, HPC parallel file system, sub-ms latency |
| 10 | RDS Performance Insights, Top SQL, wait events, query monitoring |
