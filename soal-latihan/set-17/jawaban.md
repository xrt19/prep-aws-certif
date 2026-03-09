# Jawaban — Set 17
## Domain: High-Performing Architectures

---

## Soal 1 — Jawaban: D

**Konversi ke Parquet/ORC + partitioning**

**Kenapa D:**
Athena di-charge berdasarkan **jumlah data yang di-scan** ($5 per TB). Dua optimasi terbesar:

**1. Columnar format (Parquet/ORC):**
- CSV adalah row-based: untuk baca kolom `revenue`, Athena harus scan seluruh row (semua kolom)
- Parquet/ORC adalah column-based: Athena hanya baca kolom `revenue` — skip kolom lain secara fisik
- Kompresi columnar juga lebih baik (Snappy, Zstd) → ukuran file lebih kecil
- Speedup query: 10–100x lebih cepat, cost reduction: 70–99% lebih murah

**2. Partitioning:**
- Organize data di S3 berdasarkan `s3://bucket/data/year=2024/month=03/day=10/`
- Query `WHERE date = '2024-03-10'` hanya scan folder yang relevan
- Partition pruning: Athena skip seluruh prefix S3 yang tidak match filter

Kombinasi keduanya adalah best practice standar untuk Athena optimization.

**Kenapa bukan yang lain:**
- **A** — S3 Intelligent-Tiering untuk cost optimization storage (auto-move ke cheaper tier saat tidak diakses). Tidak mempengaruhi query scan speed atau amount of data scanned.
- **B** — S3 Transfer Acceleration untuk upload ke S3 dari remote locations. Tidak ada hubungannya dengan Athena query performance.
- **C** — Partitioning saja (pilihan C) memang bagus, tapi kombinasi dengan columnar format (pilihan D) jauh lebih efektif. Pilihan D mencakup partitioning DAN columnar format.

**Konsep yang diuji:** Athena query optimization, columnar format (Parquet/ORC), partitioning, partition pruning, data scan cost.

---

## Soal 2 — Jawaban: D

**Tambah DPU atau aktifkan Glue Auto Scaling**

**Kenapa D:**
AWS Glue menggunakan **Apache Spark** di bawahnya. Spark bekerja dengan cara membagi data menjadi partitions dan memproses setiap partition secara parallel di multiple workers.

**DPU (Data Processing Unit):**
- 1 DPU = 4 vCPU + 16 GB RAM
- Default: 10 DPU untuk Glue ETL job
- Lebih banyak DPU = lebih banyak Spark executors = lebih banyak parallelism

**Glue Auto Scaling (Glue 3.0+):**
- Glue otomatis menambah/mengurangi DPU berdasarkan kebutuhan aktual workload
- Tidak perlu guess berapa DPU yang tepat
- Bayar hanya DPU yang digunakan

Untuk dataset besar, bottleneck biasanya di jumlah parallel workers — tambah DPU adalah solusi langsung.

**Kenapa bukan yang lain:**
- **A** — AWS Batch untuk containerized batch workloads (Docker). Pindah ke Batch membutuhkan rewrite ETL logic dari scratch, bukan "tanpa mengubah logika transformasi".
- **B** — Pindah ke DynamoDB tidak masuk akal untuk data warehouse/analytics yang berada di S3 — DynamoDB bukan analytics engine.
- **C** — Glue Job Bookmarks untuk **incremental processing** — hanya proses data yang belum pernah diproses sebelumnya. Ini mengurangi volume data yang diproses, bukan meningkatkan throughput per unit data. Untuk optimasi throughput dataset besar, DPU lebih tepat.

**Konsep yang diuji:** AWS Glue DPU, Apache Spark parallelism, Glue Auto Scaling, ETL performance tuning.

---

## Soal 3 — Jawaban: D

**DynamoDB TTL (Time to Live)**

**Kenapa D:**
DynamoDB TTL adalah fitur yang memungkinkan **expiry otomatis items**:
- Tambahkan attribute (misal `expiry_time`) pada setiap item dengan value Unix timestamp (epoch seconds) kapan item harus expired
- Enable TTL pada tabel dengan menentukan nama attribute tersebut
- DynamoDB background process secara otomatis scan dan hapus items yang `expiry_time < current_time`
- **Tidak konsumsi WCU** — penghapusan TTL tidak di-charge sebagai write operations
- Penghapusan bisa delay hingga 48 jam setelah expired (eventual, bukan immediate) — untuk session data ini acceptable
- Items yang sudah expired tapi belum dihapus tidak muncul di query/scan hasil

**Kenapa bukan yang lain:**
- **A** — Lambda scheduled setiap jam yang scan tabel dan hapus items: (1) Scan DynamoDB konsumsi RCU yang mahal untuk tabel besar; (2) Delete operations konsumsi WCU; (3) Lambda invocation cost; (4) kompleksitas operasional. TTL gratis dan zero-maintenance.
- **B** — DynamoDB Streams + Lambda timer sangat complex, tidak reliable, dan mahal. Tidak ada native "timer per item" di DynamoDB selain TTL.
- **C** — S3 Lifecycle Policy untuk mengelola S3 objects (transisi ke Glacier, expire setelah X hari). Tidak ada hubungannya dengan DynamoDB.

**Konsep yang diuji:** DynamoDB TTL, expiry attribute, automatic cleanup, no WCU consumption.

---

## Soal 4 — Jawaban: C

**Redis Cluster Mode**

**Kenapa C:**
Redis Cluster Mode (ElastiCache Cluster Mode Enabled) melakukan **sharding data ke multiple shard**:
- Setiap shard adalah primary node + optional replicas
- Keyspace (semua keys) di-distribute ke shards menggunakan hash slots (0–16383)
- Misalnya 3 shards: shard 1 handle hash slots 0–5460, shard 2: 5461–10922, shard 3: 10923–16383
- Total capacity = jumlah memory semua primary nodes
- Write throughput juga naik — writes ke keys di shard berbeda bisa paralel

Ini berbeda dengan **Cluster Mode Disabled** (single shard dengan replicas) yang hanya meningkatkan read throughput dan availability, bukan write throughput atau total capacity secara horizontal.

**Kenapa bukan yang lain:**
- **A** — Vertical scaling (node lebih besar) ada batasnya dan butuh downtime atau failover. Horizontal scaling via cluster mode lebih flexible.
- **B** — Tambah Read Replicas meningkatkan **read throughput** — lebih banyak nodes bisa serve reads. Tapi tidak meningkatkan write capacity atau total data capacity jika single shard sudah full.
- **D** — Memcached mendukung multi-threading dan horizontal scaling, tapi seperti yang dibahas di Set 15: Memcached tidak mendukung complex data structures dan tidak ada persistence. Pindah ke Memcached adalah downgrade fitur.

**Konsep yang diuji:** ElastiCache Redis Cluster Mode, sharding, hash slots, horizontal scaling, capacity expansion.

---

## Soal 5 — Jawaban: C

**S3 Multipart Upload + S3 Pre-signed URL**

**Kenapa C:**
Kombinasi ini adalah **best practice untuk large file upload dari untrusted clients**:

**S3 Pre-signed URL:**
- Server generate URL yang temporary (dengan expiry) yang mengizinkan upload langsung ke S3
- Client (creator) tidak perlu AWS credentials
- Upload langsung ke S3, tidak melalui application server → server tidak jadi bottleneck, tidak bayar EC2 bandwidth
- URL bisa di-scope ke specific bucket/key

**S3 Multipart Upload:**
- File dibagi menjadi parts (minimum 5 MB per part)
- Setiap part di-upload secara independent
- Jika part gagal karena koneksi putus → hanya re-upload part tersebut, tidak mulai dari awal
- Parts bisa di-upload secara paralel untuk maximize bandwidth
- Final `CompleteMultipartUpload` request menggabungkan semua parts

Kombinasi keduanya: server generate pre-signed URLs untuk setiap part, client upload parts langsung ke S3.

**Kenapa bukan yang lain:**
- **A** — Upload ke EC2 kemudian transfer ke S3 berarti server harus handle bandwidth yang sama dua kali (receive dari creator + send ke S3). Untuk file 2 GB, ini boros dan EC2 bisa jadi bottleneck. Juga tidak ada built-in resume.
- **B** — AWS DataSync untuk transfer data dari **on-premise data center** ke AWS menggunakan DataSync agent. Bukan untuk individual creators yang upload dari perangkat mereka.
- **D** — SQS memiliki message size limit 256 KB. Tidak bisa digunakan untuk file 2 GB.

**Konsep yang diuji:** S3 Multipart Upload, S3 pre-signed URL, direct-to-S3 upload pattern, resume on failure.

---

## Soal 6 — Jawaban: D

**SageMaker Managed Spot Training**

**Kenapa D:**
SageMaker Managed Spot Training mengintegrasikan **EC2 Spot Instances** ke dalam training workflow dengan otomatisasi checkpoint:
- SageMaker otomatis handle Spot interruptions — jika instance di-interrupt, training dijeda
- SageMaker restart training di Spot Instance baru yang available
- Training resume dari **checkpoint terakhir** yang disimpan ke S3 (developer perlu implement checkpoint di training code, tapi SageMaker handle lifecycle-nya)
- **Hemat hingga 90%** biaya compute dibanding On-Demand training instances
- `MaxWaitTimeInSeconds` harus lebih besar dari `MaxRuntimeInSeconds` untuk allow waiting saat Spot tidak available

Untuk training jobs 8–12 jam yang toleran terhadap interruption (bisa checkpoint), ini adalah optimasi biaya terbesar.

**Kenapa bukan yang lain:**
- **A** — SageMaker Serverless Inference untuk **model inference** (serving predictions), bukan training. Serverless Inference tidak cocok untuk workload training yang CPU/GPU intensive berkelanjutan.
- **B** — SageMaker Ground Truth untuk **data labeling** — membantu tim membuat labeled dataset untuk training. Tidak mempengaruhi biaya training compute.
- **C** — SageMaker Experiments untuk tracking metrics, parameters, dan artifacts dari training runs. Berguna untuk reproducibility dan comparison, tapi tidak mengurangi biaya training.

**Konsep yang diuji:** SageMaker Managed Spot Training, checkpoint, cost optimization, Spot Instance interruption handling.

---

## Soal 7 — Jawaban: C

**SageMaker Real-Time Inference**

**Kenapa C:**
SageMaker Real-Time Inference adalah pilihan untuk **latency-sensitive, high-throughput inference**:
- **Persistent endpoint** — instances selalu running, tidak ada cold start
- Latency: **single-digit milliseconds** untuk model ringan, < 50ms untuk model besar
- Support **instance types yang tepat**: GPU instances (g4dn, p3) untuk deep learning models yang butuh GPU, CPU instances (c5) untuk model klasik ML
- **Auto Scaling** bisa dikonfigurasi berdasarkan `InvocationsPerInstance` metric
- SLA yang konsisten karena dedicated instances

**Kenapa bukan yang lain:**
- **A** — SageMaker Batch Transform untuk **offline batch inference** — proses large datasets sekaligus tanpa real-time requirement. Tidak ada persistent endpoint, tidak cocok untuk real-time requests.
- **B** — SageMaker Serverless Inference untuk workloads intermittent dengan traffic yang tidak predictable. Ada **cold start** jika endpoint tidak diakses — latency bisa mencapai detik. Tidak cocok untuk < 5ms requirement.
- **D** — Lambda dengan model di S3 sangat terbatas: Lambda maximum memory 10 GB, execution time 15 menit max, dan cold start untuk model besar bisa sangat lambat. Tidak cocok untuk model ML yang kompleks dengan latency requirement ketat.

**Konsep yang diuji:** SageMaker Real-Time Inference, Serverless Inference trade-offs, Batch Transform, inference latency requirements.

---

## Soal 8 — Jawaban: D

**Naikkan memory requests dan limits di pod spec**

**Kenapa D:**
**OOMKilled** (Out of Memory Killed) di Kubernetes terjadi karena Kubernetes mengimplementasikan **resource limits** yang ketat:

```yaml
resources:
  requests:
    memory: "256Mi"  # Minimum guarantee untuk scheduling
    cpu: "250m"
  limits:
    memory: "512Mi"  # HARD LIMIT — jika container pakai > ini → OOMKilled
    cpu: "500m"
```

Saat container menggunakan memory melebihi `limits.memory`, kernel Linux langsung **kill proses** tersebut (OOM killer). Ini terjadi meskipun node masih punya memory tersedia — Kubernetes tidak akan allow container melampaui limit-nya.

Solusi: naikkan `limits.memory` (dan `requests.memory` agar scheduling lebih akurat).

**Kenapa bukan yang lain:**
- **A** — Tambah nodes tidak membantu jika pods di-kill karena melebihi limit mereka sendiri. Node baru pun, container masih akan OOMKilled karena limits belum diubah.
- **B** — Cluster autoscaler menambah nodes saat ada pods yang `Pending` karena resource tidak cukup di nodes yang ada. Tapi OOMKilled bukan karena pod tidak bisa scheduled — pod sudah running, lalu di-kill karena melebihi limit.
- **C** — Fargate mode di EKS punya default resource limits — tidak secara otomatis menyelesaikan OOMKilled. Masalahnya tetap pada konfigurasi resource spec.

**Konsep yang diuji:** Kubernetes resource requests vs limits, OOMKilled, memory management, container spec tuning.

---

## Soal 9 — Jawaban: C

**MSCK REPAIR TABLE atau ALTER TABLE ADD PARTITION**

**Kenapa C:**
Athena menggunakan **AWS Glue Data Catalog** sebagai metastore untuk menyimpan metadata tabel termasuk **partisi** (lokasi S3 untuk setiap partition value):

Saat data baru di-upload ke S3 dengan struktur partisi baru (misal `year=2024/month=04/`), **Glue Data Catalog tidak otomatis update** — Athena masih hanya tahu partisi yang terdaftar sebelumnya.

Solusi:
1. **`MSCK REPAIR TABLE tablename`** — scan S3 path dan automatically add semua partitions yang ada di S3 tapi belum di catalog. Cocok untuk one-time sync atau via scheduled query.
2. **`ALTER TABLE tablename ADD PARTITION (year='2024', month='04') LOCATION 's3://...'`** — tambah partisi spesifik secara manual.
3. **Glue Crawler** — schedule Glue Crawler untuk otomatis detect dan register partitions baru.

**Kenapa bukan yang lain:**
- **A** — Delete dan re-create tabel setiap hari adalah destruktif, tidak scalable, dan butuh re-define semua properties tabel. Bukan standard practice.
- **B** — S3 Event Notification + Lambda bisa di-trigger untuk jalankan `ALTER TABLE ADD PARTITION` — tapi soal menanya cara menyelesaikan masalah, bukan arsitektur event-driven-nya. Inti solusinya tetap update Glue catalog.
- **D** — S3 Versioning untuk recovery dari accidental deletes/overwrites. Athena tidak menggunakan versioning untuk determine data freshness — ia menggunakan Glue catalog metadata.

**Konsep yang diuji:** Athena partition management, Glue Data Catalog, MSCK REPAIR TABLE, partition pruning, data lake operations.

---

## Soal 10 — Jawaban: D

**Amazon Kinesis Data Analytics (Apache Flink)**

**Kenapa D:**
Amazon Kinesis Data Analytics for Apache Flink adalah **managed streaming analytics service**:
- Jalankan Flink applications (Java/Python/SQL) atau Kinesis Data Analytics Studio (Zeppelin notebook dengan SQL)
- **Windowing** — tumbling window (non-overlapping), sliding window, session window untuk aggregasi per period
- Contoh: tumbling window 1 menit → hitung `COUNT, AVG, SUM` per device type setiap menit
- Latency: **detik** (near real-time)
- Output ke Kinesis Data Streams, S3, OpenSearch, RDS untuk downstream consumption
- Auto scales berdasarkan throughput input

```sql
-- Contoh query Kinesis Data Analytics
SELECT device_type,
       COUNT(*) as count,
       AVG(temperature) as avg_temp
FROM input_stream
WINDOW TUMBLING (OVER 1 MINUTE)
GROUP BY device_type
```

**Kenapa bukan yang lain:**
- **A** — AWS Glue Streaming ETL bisa memproses streaming data, tapi dirancang untuk ETL/transformation, bukan untuk real-time analytics dengan windowed aggregations. Latency lebih tinggi dan lebih cocok untuk near-real-time ETL.
- **B** — AWS Batch adalah batch processing — tidak ada streaming, tidak ada real-time. "Setiap menit" butuh setup dan teardown job, latency tinggi, overhead besar.
- **C** — Lambda triggered by Kinesis bisa melakukan aggregasi sederhana per batch, tapi **tidak support stateful windowing** natively. Untuk menghitung rata-rata per menit per device type across multiple batches, perlu manage state secara manual (misalnya di DynamoDB) — complex dan error-prone dibanding Flink yang punya native windowing.

**Konsep yang diuji:** Kinesis Data Analytics, Apache Flink, streaming analytics, windowing (tumbling window), real-time aggregation, IoT analytics.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | Athena optimization — Parquet/ORC + partitioning |
| 2 | AWS Glue DPU scaling, Apache Spark parallelism |
| 3 | DynamoDB TTL, automatic item expiry |
| 4 | ElastiCache Redis Cluster Mode, sharding |
| 5 | S3 Multipart Upload + pre-signed URL pattern |
| 6 | SageMaker Managed Spot Training, checkpoint |
| 7 | SageMaker Real-Time vs Serverless vs Batch inference |
| 8 | Kubernetes OOMKilled, resource limits, pod spec |
| 9 | Athena partition management, MSCK REPAIR TABLE, Glue catalog |
| 10 | Kinesis Data Analytics (Flink), tumbling window, streaming aggregation |
