# Soal Latihan — Set 18
## Domain: High-Performing Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah perusahaan memiliki aplikasi yang menggunakan **Amazon RDS PostgreSQL**. Beberapa query analitik kompleks membutuhkan waktu menit-an dan memblokir workload OLTP. Tim tidak mau memindahkan data ke data warehouse lain, tapi ingin query analitik tidak mengganggu production traffic.

Solusi paling tepat?

- A. Tambah lebih banyak RDS Read Replicas dan distribusikan semua queries ke sana
- B. Aktifkan RDS Multi-AZ agar standby bisa handle analytics queries
- C. **Dedikasikan satu RDS Read Replica khusus untuk analytics** — arahkan aplikasi OLTP ke primary/replica lain, arahkan analytics queries ke replica khusus; replicas bisa di-promote atau di-scale instance type secara independent
- D. Gunakan RDS Proxy untuk distribusi queries otomatis ke primary vs replica

---

**Soal 2**

Sebuah perusahaan memiliki **static website** yang di-host di S3 dan di-serve via CloudFront. Mereka ingin menambahkan fungsionalitas sederhana: **redirect URL pendek** (misalnya `/go/promo` → `https://example.com/landing-page-promo`) yang di-konfigurasi via file JSON di S3. Logic-nya sederhana: lookup key dari JSON, redirect dengan HTTP 301.

Mana yang paling tepat dan cost-efficient untuk implementasi ini?

- A. Lambda function di belakang API Gateway untuk handle redirect logic
- B. CloudFront Origin Groups dengan dua origins berbeda
- C. **CloudFront Functions** — jalankan JavaScript ringan di viewer request stage langsung di edge; cocok untuk simple request manipulation seperti redirect, URL rewriting, header manipulation; latency < 1ms; biaya sangat murah (1/6 harga Lambda@Edge)
- D. Lambda@Edge untuk handle redirect di origin request stage

---

**Soal 3**

Sebuah perusahaan ingin memigrasikan **on-premise Oracle database** ke AWS dengan downtime seminimal mungkin. Database aktif digunakan 24/7 oleh aplikasi production. Setelah migrasi selesai, mereka ingin switch traffic ke database baru dengan downtime < 1 menit.

Service mana yang paling tepat?

- A. AWS Snowball Edge untuk bulk transfer data
- B. AWS DataSync untuk continuous data transfer
- C. **AWS Database Migration Service (DMS)** — replikasi data secara continuous dari source ke target database; mendukung heterogeneous migration (Oracle → Aurora PostgreSQL); gunakan Change Data Capture (CDC) untuk replikasi ongoing changes; cutover saat lag minimal
- D. Manual export/import dengan pg_dump dan downtime penuh

---

**Soal 4**

Sebuah startup menggunakan **AWS Lambda** untuk image resizing. Setiap kali user upload foto ke S3, Lambda dipanggil untuk generate thumbnail dalam 3 ukuran berbeda. Lambda function butuh **FFmpeg binary** (150 MB) yang selalu sama untuk semua invocations.

Cara terbaik untuk mengelola dependency besar ini di Lambda?

- A. Include FFmpeg binary langsung di deployment package setiap function
- B. Download FFmpeg dari S3 setiap kali Lambda cold start
- C. **Gunakan Lambda Layers** — package FFmpeg sebagai Layer yang bisa di-share ke banyak functions; Layer di-cache di execution environment; tidak perlu include di setiap deployment package; batas deployment package 50 MB tapi dengan Layer total bisa 250 MB
- D. Pindahkan ke ECS karena Lambda tidak support binary dependencies

---

**Soal 5**

Sebuah perusahaan menjalankan **Apache Kafka** self-managed di EC2 untuk message streaming. Tim kesulitan dengan operational overhead: patching, scaling, monitoring, dan broker management. Mereka ingin pindah ke managed service yang compatible dengan existing Kafka producer/consumer code.

AWS service mana yang paling tepat?

- A. Amazon SQS — managed queue service
- B. Amazon Kinesis Data Streams — managed streaming
- C. **Amazon MSK (Managed Streaming for Apache Kafka)** — fully managed Kafka yang compatible dengan Kafka APIs; existing producer/consumer code tidak perlu diubah; AWS handle broker provisioning, patching, monitoring, storage scaling
- D. Amazon EventBridge — managed event bus

---

**Soal 6**

Sebuah perusahaan memiliki **Lambda function** yang perlu mengakses RDS database di VPC private subnet. Lambda dipanggil ribuan kali per menit dan setiap invocation membuka koneksi database baru. Tim mengalami **"too many connections"** error di RDS.

Solusi paling tepat?

- A. Naikkan `max_connections` parameter di RDS parameter group
- B. Gunakan connection pooling di Lambda code dengan library seperti PgBouncer
- C. **Gunakan RDS Proxy** — proxy yang duduk di antara Lambda dan RDS; melakukan connection pooling; ribuan Lambda invocations share koneksi yang sama ke RDS; mengurangi connection overhead secara drastis
- D. Pindahkan RDS ke public subnet agar Lambda tidak perlu VPC

---

**Soal 7**

Sebuah perusahaan ingin membangun **data pipeline** yang secara otomatis memproses file CSV baru yang di-upload ke S3, melakukan transformasi, dan menyimpan hasilnya ke DynamoDB. File di-upload secara sporadis sepanjang hari.

Arsitektur event-driven mana yang paling tepat?

- A. Scheduled EventBridge rule setiap 5 menit untuk cek file baru di S3
- B. S3 Inventory report harian kemudian batch process
- C. **S3 Event Notification → SQS → Lambda** — saat file baru di-upload, S3 kirim event ke SQS; Lambda di-trigger dari SQS; SQS sebagai buffer mencegah Lambda overwhelmed jika banyak file tiba bersamaan; retry otomatis jika processing gagal
- D. EC2 instance yang continuously poll S3 bucket untuk file baru

---

**Soal 8**

Sebuah perusahaan memiliki aplikasi multi-tier: **ALB → EC2 (web tier) → RDS**. Mereka ingin meningkatkan performance web tier untuk static content (gambar, CSS, JS) yang jarang berubah, tanpa mengubah aplikasi code.

Optimasi mana yang paling tepat?

- A. Tambah lebih banyak EC2 instances di web tier
- B. Aktifkan ALB caching untuk static content
- C. **Tambahkan CloudFront di depan ALB** — CloudFront cache static content di edge locations; request untuk static content di-serve dari CloudFront cache tanpa hit ALB atau EC2; dynamic requests tetap di-forward ke ALB
- D. Pindahkan static content ke EFS agar EC2 akses lebih cepat

---

**Soal 9**

Sebuah perusahaan memiliki workload yang membutuhkan **shared file system** yang bisa di-akses oleh ratusan EC2 instances Linux secara bersamaan dengan **throughput sangat tinggi** (decibel GB/s) untuk workload HPC seperti genomics, financial modeling, dan rendering.

Storage solution mana yang tepat?

- A. Amazon EFS — managed NFS untuk Linux workloads
- B. Amazon S3 — object storage dengan throughput tinggi
- C. Amazon EBS Multi-Attach — attach satu EBS ke beberapa instances
- D. **Amazon FSx for Lustre** — high-performance parallel file system dirancang untuk HPC; throughput hingga ratusan GB/s; bisa di-integrasikan dengan S3 sebagai data repository; support POSIX interface; latency sub-millisecond

---

**Soal 10**

Sebuah perusahaan ingin memonitor **performance query** di RDS MySQL mereka. Mereka ingin tahu query mana yang paling lambat, berapa lama rata-rata execution time, dan query mana yang paling sering dieksekusi — tanpa perlu install monitoring agent.

Fitur RDS mana yang tepat?

- A. CloudWatch Metrics untuk RDS — monitor CPU, IOPS, connections
- B. RDS Enhanced Monitoring — OS-level metrics via CloudWatch agent
- C. CloudTrail untuk audit API calls ke RDS
- D. **RDS Performance Insights** — dashboard yang menunjukkan database load breakdown per SQL query, wait events, dan top SQL statements; identifikasi query bottleneck tanpa perlu install agent; free tier tersedia, retention 7 hari gratis

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
