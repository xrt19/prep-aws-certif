# Soal Latihan — Set 15
## Domain: High-Performing Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah aplikasi membutuhkan caching layer dengan kemampuan berikut: **data structures kompleks** (sorted sets, lists, hashes), **pub/sub messaging**, dan **persistence** agar cache tidak kosong saat restart.

Antara ElastiCache Redis dan Memcached, mana yang tepat dan kenapa?

- A. Memcached — lebih simpel dan performa lebih tinggi untuk simple key-value
- B. Memcached — support multi-threading sehingga lebih efisien untuk high concurrency
- C. **Redis** — support complex data structures, pub/sub, persistence (RDB/AOF), scripting dengan Lua, dan bisa digunakan sebagai message broker
- D. Keduanya equivalen untuk semua use cases

---

**Soal 2**

Sebuah aplikasi DynamoDB mengalami **throttling** pada beberapa items yang sangat sering diakses ("hot keys"), sementara sebagian besar items jarang diakses. Partition lain masih memiliki banyak capacity yang tersisa.

Apa penyebab utama dan solusi yang paling tepat?

- A. Tambah total RCU/WCU pada tabel — naik capacity secara keseluruhan
- B. Aktifkan DynamoDB Auto Scaling — capacity akan menyesuaikan traffic
- C. **Redesain partition key** untuk distribusi yang lebih merata — tambahkan random suffix atau timestamp ke partition key agar akses tersebar ke banyak partitions, bukan menumpuk di satu partition
- D. Gunakan DynamoDB Global Tables untuk distribusi load ke region lain

---

**Soal 3**

Sebuah workload **machine learning inference** membutuhkan storage sementara yang sangat cepat untuk **scratch space** — data ditulis dan dibaca secara intensif selama job berjalan, tapi tidak perlu disimpan setelah instance mati.

Storage mana yang paling tepat?

- A. EBS gp3 — persistent SSD storage
- B. EBS io2 — highest IOPS persistent storage
- C. Amazon S3 — object storage untuk data ML
- D. **EC2 Instance Store** — storage yang attached langsung ke host fisik, throughput dan latency terbaik, tidak ada network overhead, tapi **tidak persistent** (data hilang saat instance stop/terminate)

---

**Soal 4**

Sebuah perusahaan menggunakan **Kinesis Data Streams** untuk ingesti log dari ribuan aplikasi. Throughput yang dibutuhkan terus meningkat dan mereka mulai mengalami `ProvisionedThroughputExceededException`.

Cara paling tepat untuk meningkatkan throughput Kinesis adalah?

- A. Buat stream baru dan distribute producers ke beberapa streams
- B. Aktifkan Kinesis Enhanced Fan-Out untuk consumer
- C. **Tambah jumlah shards** via shard splitting — setiap shard mendukung 1 MB/s ingest dan 2 MB/s read, menambah shard meningkatkan total throughput secara proporsional
- D. Kompres data sebelum kirim ke Kinesis untuk kurangi ukuran

---

**Soal 5**

Sebuah perusahaan menjalankan query **analytics berat** (full table scan, complex aggregations) pada Aurora MySQL cluster mereka. Queries ini berjalan lambat dan mengganggu performance workload OLTP yang juga berjalan di cluster yang sama.

Fitur Aurora mana yang memungkinkan queries analytics berjalan lebih cepat **tanpa memindahkan data**?

- A. Aurora Read Replica — jalankan analytics queries di replica
- B. **Aurora Parallel Query** — pushes query processing down ke storage layer dan eksekusi secara paralel di banyak storage nodes, drastis mengurangi waktu analytics queries
- C. Aurora Global Database — jalankan analytics di secondary region
- D. Aurora Serverless v2 — scale kapasitas saat query berat

---

**Soal 6**

Sebuah Lambda function digunakan untuk **personalisasi konten** yang di-serve via CloudFront — setiap user mendapat response yang di-customize berdasarkan geolocation, device type, dan cookies mereka. Logic personalisasi perlu berjalan **sedekat mungkin ke user**.

Fitur CloudFront mana yang tepat?

- A. CloudFront cache behaviors — buat rule per user segment
- B. CloudFront Origin Shield — tambah caching layer di depan origin
- C. **CloudFront Functions atau Lambda@Edge** — jalankan code JavaScript/Node.js di edge locations, bisa inspect dan modify request/response berdasarkan headers, cookies, geolocation
- D. API Gateway regional endpoint di setiap region

---

**Soal 7**

Sebuah aplikasi perlu **mendownload bagian tertentu** dari file besar (500 MB) yang tersimpan di S3 — misalnya hanya 10 MB di tengah file — tanpa harus download seluruh file.

Fitur S3 mana yang memungkinkan ini?

- A. S3 Select — query data langsung dari S3 dengan SQL
- B. S3 Multipart Download via TransferManager
- C. **S3 Byte-Range Fetches** — tambahkan `Range` HTTP header pada GET request untuk retrieve hanya byte range yang dibutuhkan, mengurangi data transfer secara drastis
- D. S3 Glacier Expedited Retrieval untuk akses cepat ke portion file

---

**Soal 8**

Sebuah perusahaan memiliki arsitektur microservices di VPC. Banyak services saling memanggil satu sama lain via private IP. Tim menemukan bahwa koneksi keluar melewati NAT Gateway sebelum masuk ke **VPC endpoint** untuk DynamoDB, menyebabkan **extra cost dan latency** yang tidak perlu.

Apa yang harus diperbaiki?

- A. Gunakan Direct Connect untuk akses ke DynamoDB tanpa NAT
- B. Pindahkan instances ke public subnet agar tidak perlu NAT
- C. **Pastikan route table subnet mengarah ke VPC Gateway Endpoint untuk DynamoDB** — Gateway Endpoint tidak butuh NAT Gateway, traffic langsung ke DynamoDB via AWS network, gratis
- D. Gunakan Interface Endpoint untuk DynamoDB sebagai pengganti Gateway Endpoint

---

**Soal 9**

Sebuah perusahaan menjalankan ASG dengan target tracking policy. CPU utilization target adalah 50%. Saat ini ada 10 instances dengan CPU rata-rata 80%. Tim ingin ASG scale out **sesegera mungkin** tapi instances baru butuh 3 menit untuk warm-up.

Konfigurasi mana yang harus diperhatikan agar scaling tepat?

- A. Turunkan target CPU ke 30% agar ASG lebih agresif scale out
- B. Tambah maximum capacity ASG agar bisa scale lebih banyak
- C. **Set instance warm-up period sesuai actual warm-up time (3 menit)** — selama warm-up, instance baru tidak dihitung dalam average metric, mencegah ASG over-provision karena mengira capacity sudah cukup padahal instances belum fully operational
- D. Ganti ke Scheduled Scaling yang lebih predictable

---

**Soal 10**

Sebuah perusahaan memiliki RDS PostgreSQL dengan query yang lambat. Analisis menunjukkan query **`SELECT * FROM orders WHERE customer_id = 123`** melakukan full table scan meskipun tabel `orders` memiliki jutaan rows.

Solusi paling tepat untuk meningkatkan performance query ini?

- A. Tambahkan RDS Read Replica untuk handle query ini
- B. Upgrade RDS ke instance yang lebih besar
- C. Pindahkan ke Aurora PostgreSQL yang lebih cepat
- D. **Buat index pada kolom `customer_id`** — query dengan equality filter pada indexed column menggunakan index scan (O(log n)) alih-alih full table scan (O(n)), drastis mengurangi waktu query

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
