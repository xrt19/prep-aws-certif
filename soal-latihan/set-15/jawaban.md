# Jawaban — Set 15
## Domain: High-Performing Architectures

---

## Soal 1 — Jawaban: C

**Redis**

**Kenapa C:**
Redis jauh lebih feature-rich dibanding Memcached:

| Feature | Redis | Memcached |
|---------|-------|-----------|
| Data structures | Strings, Lists, Sets, Sorted Sets, Hashes, Bitmaps, HyperLogLog | Strings only |
| Persistence | RDB snapshots + AOF logging | Tidak ada |
| Pub/Sub | Ya | Tidak |
| Replication | Ya (Multi-AZ) | Tidak |
| Cluster Mode | Ya (sharding) | Ya (tapi tanpa replication) |
| Lua scripting | Ya | Tidak |

Untuk requirement ini (complex data structures + pub/sub + persistence), hanya Redis yang memenuhi.

**Kenapa bukan yang lain:**
- **A** — Benar bahwa Memcached lebih simpel untuk pure key-value, tapi tidak bisa memenuhi requirement complex data structures dan persistence.
- **B** — Memcached memang multi-threaded (vs Redis yang single-threaded per command), tapi Redis sudah cukup fast untuk sebagian besar workloads dan Redis 6+ mendukung I/O threading.
- **D** — Keduanya tidak equivalen — mereka punya trade-offs yang berbeda dan use cases yang berbeda.

**Konsep yang diuji:** Redis vs Memcached feature comparison, when to choose each, data structures, persistence.

---

## Soal 2 — Jawaban: C

**Redesain partition key untuk distribusi merata**

**Kenapa C:**
Ini adalah masalah **hot partition** — DynamoDB membagi data ke partitions berdasarkan partition key, dan setiap partition mendapat alokasi throughput yang equal. Jika semua traffic menumpuk di satu atau sedikit partition keys (hot keys), partition tersebut cepat habis capacity-nya meskipun total capacity tabel masih banyak.

Solusi: **partition key yang memiliki high cardinality** dan distribusi yang merata:
- Tambahkan **random suffix** (misal `user_id#1`, `user_id#2`... `user_id#10`) lalu scatter-gather saat read
- Tambahkan **timestamp** sebagai bagian partition key untuk time-series data
- Gunakan **composite key** yang lebih granular

**Kenapa bukan yang lain:**
- **A** — Menambah total capacity tidak menyelesaikan hot partition — capacity baru juga akan di-distribusikan merata ke semua partitions, tapi hot partition tetap kelebihan beban relatif terhadap alokasi per-partition-nya.
- **B** — Auto Scaling reaktif terhadap consumed capacity di level tabel, tapi tidak menyelesaikan fundamental masalah bahwa satu partition overloaded sementara yang lain idle.
- **D** — Global Tables untuk multi-region availability, bukan untuk menyelesaikan hot partition dalam satu region.

**Konsep yang diuji:** DynamoDB partition key design, hot partition problem, cardinality, write sharding.

---

## Soal 3 — Jawaban: D

**EC2 Instance Store**

**Kenapa D:**
Instance Store adalah **ephemeral storage** yang secara fisik attached ke host server EC2:
- **Tidak ada network I/O** — langsung ke storage device → latency sangat rendah
- **Throughput tertinggi** — bisa mencapai jutaan IOPS dan puluhan GB/s
- **Tidak persistent** — data hilang saat instance stop, terminate, atau hardware failure
- Tersedia di instance types tertentu (i3, i4i, d2, h1, dll)

Untuk scratch space ML yang hanya butuh performa tinggi selama job berlangsung dan tidak perlu disimpan, ini adalah pilihan optimal.

**Kenapa bukan yang lain:**
- **A & B** — EBS adalah network-attached storage — ada latency jaringan meskipun sangat rendah. Tidak secepat instance store untuk truly local I/O.
- **C** — S3 adalah object storage dengan latency lebih tinggi dari local disk — tidak cocok untuk scratch space yang membutuhkan frequent random I/O.

**Konsep yang diuji:** EC2 Instance Store, ephemeral storage, throughput vs persistence trade-off, ML scratch space.

---

## Soal 4 — Jawaban: C

**Tambah jumlah shards via shard splitting**

**Kenapa C:**
Kinesis Data Streams throughput capacity **berbanding lurus dengan jumlah shards**:
- 1 shard = 1 MB/s ingest (1000 records/s), 2 MB/s read
- 10 shards = 10 MB/s ingest, 20 MB/s read

Untuk meningkatkan throughput, **shard splitting** membagi satu shard menjadi dua — total capacity naik. Ini adalah cara scaling Kinesis yang native.

**Kenapa bukan yang lain:**
- **A** — Membuat stream baru dan split producers lebih complex — perlu manage routing di sisi producers dan consumers. Shard splitting pada satu stream lebih simpel.
- **B** — Enhanced Fan-Out meningkatkan **read throughput per consumer** (dari shared 2 MB/s per shard menjadi dedicated 2 MB/s per shard per consumer) — tidak meningkatkan **write/ingest throughput** yang menyebabkan `ProvisionedThroughputExceededException`.
- **D** — Kompresi mengurangi ukuran data per record tapi tidak menambah throughput capacity (MB/s dan records/s limit tetap sama).

**Konsep yang diuji:** Kinesis shard scaling, shard capacity limits, shard splitting, Enhanced Fan-Out (read) vs shard count (write).

---

## Soal 5 — Jawaban: B

**Aurora Parallel Query**

**Kenapa B:**
Aurora Parallel Query adalah fitur yang memparalelkan eksekusi query analytics dengan cara **push processing ke storage layer**:
- Query execution di-push ke ribuan storage nodes yang menyimpan data
- Setiap storage node memproses subset data yang di-simpan di dalamnya secara paralel
- Hanya aggregate results yang dikirim ke query layer
- Drastis mengurangi data transfer dan waktu query untuk full-scan analytics
- Berjalan pada **same cluster** tanpa butuh pemindahan data

**Kenapa bukan yang lain:**
- **A** — Aurora Read Replica bisa digunakan untuk isolasi analytics queries dari OLTP, tapi Read Replica tetap melakukan serial query execution — tidak ada acceleration seperti Parallel Query.
- **C** — Global Database untuk cross-region DR — data di secondary region, butuh network transfer, dan latency read dari region lain.
- **D** — Aurora Serverless v2 untuk scaling capacity, tidak secara fundamental mengubah cara query dieksekusi.

**Konsep yang diuji:** Aurora Parallel Query, storage-layer processing, OLAP on Aurora, query acceleration.

---

## Soal 6 — Jawaban: C

**CloudFront Functions atau Lambda@Edge**

**Kenapa C:**
Keduanya memungkinkan eksekusi code di edge locations CloudFront:

| | CloudFront Functions | Lambda@Edge |
|-|---------------------|-------------|
| Trigger | Viewer request/response | Viewer & Origin request/response |
| Runtime | JavaScript (limited) | Node.js, Python |
| Latency | < 1ms | ~1ms |
| Use case | Simple rewrites, header manipulation | Complex logic, auth, personalization |
| Scale | Millions req/s | Thousands req/s |

Untuk personalisasi berdasarkan geolocation/device/cookies (logic lebih complex), Lambda@Edge lebih tepat. Untuk simple header manipulation, CloudFront Functions.

**Kenapa bukan yang lain:**
- **A** — Cache behaviors untuk routing berdasarkan path/header ke origin berbeda, bukan untuk server-side personalization logic.
- **B** — Origin Shield menambah caching layer regional di depan origin untuk meningkatkan cache hit ratio — tidak menjalankan custom code.
- **D** — API Gateway regional endpoint di setiap region adalah over-engineering yang mahal dan complex dibanding Lambda@Edge yang sudah ada di infrastruktur CloudFront.

**Konsep yang diuji:** Lambda@Edge vs CloudFront Functions, edge computing, personalization at edge.

---

## Soal 7 — Jawaban: C

**S3 Byte-Range Fetches**

**Kenapa C:**
S3 mendukung **HTTP Range requests** — standar HTTP yang memungkinkan retrieve sebagian konten file dengan menentukan byte range dalam request header:
```
GET /largefile.bin HTTP/1.1
Range: bytes=100000000-110000000
```
S3 hanya mengirimkan byte yang diminta (10 MB dari posisi 100 MB), bukan seluruh file 500 MB. Ini mengurangi data transfer, biaya, dan waktu download secara drastis.

Juga bisa digunakan untuk **parallel download** — download beberapa range sekaligus untuk maximize bandwidth.

**Kenapa bukan yang lain:**
- **A** — S3 Select untuk query **konten CSV/JSON/Parquet** dengan SQL filter — bukan untuk retrieve byte range arbitrary dari binary file.
- **B** — Multipart Download via TransferManager adalah S3 SDK feature yang menggunakan range requests di balik layar — tapi soal menanya fitur S3 native yang memungkinkan ini, yaitu Byte-Range Fetches.
- **D** — S3 Glacier Expedited Retrieval untuk mempercepat retrieval dari Glacier vault — tidak ada konsep "portion of file" di Glacier standard retrieval.

**Konsep yang diuji:** S3 Byte-Range Fetches, partial object retrieval, HTTP Range header, parallel download.

---

## Soal 8 — Jawaban: C

**Pastikan route table mengarah ke VPC Gateway Endpoint untuk DynamoDB**

**Kenapa C:**
Ini adalah masalah **routing yang salah**. Untuk DynamoDB dan S3, tersedia **Gateway VPC Endpoint** yang:
- **Gratis** — tidak ada biaya per jam atau per GB
- Diimplementasikan via **route table entry** di subnet
- Traffic ke DynamoDB/S3 langsung melalui AWS private network, tidak keluar ke internet

Jika subnet tidak punya route ke Gateway Endpoint, traffic akan menggunakan default route ke internet (melewati NAT Gateway) — mengakibatkan biaya NAT ($0.045/GB) dan latency tambahan.

**Kenapa bukan yang lain:**
- **A** — Direct Connect untuk koneksi dedicated dari on-premise ke AWS, tidak relevan untuk komunikasi dari VPC ke AWS services.
- **B** — Pindah ke public subnet melanggar prinsip security least privilege network access.
- **D** — Interface Endpoint untuk DynamoDB secara teknis ada tapi **berbayar** (per jam + per GB data processed) — Gateway Endpoint sudah cukup dan gratis.

**Konsep yang diuji:** VPC Gateway Endpoint, route table configuration, DynamoDB S3 private access, cost optimization via endpoint.

---

## Soal 9 — Jawaban: C

**Set instance warm-up period sesuai actual warm-up time**

**Kenapa C:**
Instance warm-up period di ASG target tracking policy mengontrol bagaimana **instances baru dihitung dalam metric calculation**:
- Selama warm-up period, instance baru **tidak dihitung** dalam average CPU calculation
- Tanpa warm-up period yang benar: misalnya 10 instances di 80% CPU, ASG scale out 4 instances baru → rata-rata CPU turun ke ~57% (karena 4 instance baru dihitung dengan CPU ~0%) → ASG berhenti scale out padahal instances belum fully serving traffic
- Dengan warm-up period 3 menit: instances baru tidak dihitung dulu, ASG terus scale sampai instances yang sudah warm mampu turunkan CPU ke target

**Kenapa bukan yang lain:**
- **A** — Turunkan target CPU memang membuat ASG lebih agresif, tapi ini mengubah steady-state behavior — saat load normal juga akan over-provision. Bukan solusi yang tepat.
- **B** — Maximum capacity hanya ceiling, tidak mempengaruhi seberapa cepat ASG memutuskan untuk scale.
- **D** — Scheduled scaling hanya cocok untuk traffic predictable berbasis waktu — tidak menyelesaikan masalah warm-up period.

**Konsep yang diuji:** ASG instance warm-up, target tracking metric calculation, scale-out accuracy.

---

## Soal 10 — Jawaban: D

**Buat index pada kolom `customer_id`**

**Kenapa D:**
Full table scan pada tabel dengan jutaan rows adalah performance killer klasik. Tanpa index, database harus **check setiap row** untuk menemukan rows dengan `customer_id = 123` — O(n) complexity.

Dengan **B-tree index** pada `customer_id`:
- Database langsung jump ke lokasi rows yang relevan — O(log n)
- Untuk query yang return sedikit rows dari tabel besar, speedup bisa **1000x atau lebih**
- Di RDS PostgreSQL: `CREATE INDEX idx_orders_customer_id ON orders(customer_id);`

**Kenapa bukan yang lain:**
- **A** — Read Replica mendistribusikan read load, tapi query tetap lambat di replica karena masalahnya adalah missing index, bukan insufficient database server capacity.
- **B** — Instance lebih besar menambah CPU dan memory, sedikit membantu untuk query yang complex, tapi full table scan pada jutaan rows tetap slow — ini adalah algorithmic problem, bukan resource problem.
- **C** — Aurora lebih performant dari RDS, tapi full table scan tanpa index di Aurora juga akan lambat. Root cause bukan database engine-nya.

**Konsep yang diuji:** Database indexing, full table scan vs index scan, query optimization, B-tree index.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | Redis vs Memcached feature comparison |
| 2 | DynamoDB hot partition, partition key design |
| 3 | EC2 Instance Store, ephemeral high-performance storage |
| 4 | Kinesis shard splitting, throughput scaling |
| 5 | Aurora Parallel Query, storage-layer processing |
| 6 | Lambda@Edge vs CloudFront Functions, edge personalization |
| 7 | S3 Byte-Range Fetches, partial object retrieval |
| 8 | VPC Gateway Endpoint routing, NAT Gateway bypass |
| 9 | ASG instance warm-up period, target tracking accuracy |
| 10 | Database indexing, full table scan vs index scan |
