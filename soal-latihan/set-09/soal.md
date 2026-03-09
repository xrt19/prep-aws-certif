# Soal Latihan — Set 09
## Domain: Resilient Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah aplikasi e-commerce menggunakan RDS MySQL sebagai database utama. Saat traffic tinggi, banyak Lambda functions melakukan koneksi database secara bersamaan, menyebabkan error "too many connections" karena RDS kehabisan connection pool.

Solusi paling tepat untuk mengatasi ini tanpa mengubah arsitektur aplikasi secara besar-besaran adalah?

- A. Upgrade RDS instance ke tipe yang lebih besar
- B. Gunakan **Amazon RDS Proxy** — connection pooling managed yang reuse koneksi database, mengurangi jumlah koneksi langsung ke RDS secara drastis
- C. Tambahkan Read Replicas untuk distribusi koneksi
- D. Pindahkan aplikasi ke EC2 agar bisa kontrol connection pool sendiri

---

**Soal 2**

Sebuah perusahaan menjalankan Aurora MySQL. Mereka ingin database bisa **otomatis scale storage** tanpa perlu provision storage di awal, dan ingin kemampuan **failover ke replica dalam hitungan detik** (bukan menit).

Pernyataan mana yang benar tentang Amazon Aurora?

- A. Aurora storage harus di-provision di awal seperti RDS biasa
- B. Aurora **otomatis scale storage** dari 10 GB hingga 128 TB sesuai kebutuhan, dan failover ke Aurora Replica biasanya selesai dalam **kurang dari 30 detik**
- C. Aurora failover sama dengan RDS Multi-AZ — butuh 60–120 detik
- D. Aurora Replica hanya untuk read traffic, tidak bisa digunakan untuk failover

---

**Soal 3**

Sebuah sistem notification perlu mengirim **satu event ke beberapa subscriber berbeda secara bersamaan**: kirim email (via SES), update database (via Lambda), dan push ke mobile (via SNS Mobile Push). Semua harus menerima event yang sama.

Pattern arsitektur mana yang paling tepat?

- A. Buat tiga Lambda functions yang dipanggil secara sequential
- B. Gunakan **SNS topic** dengan multiple subscriptions — satu publish ke SNS langsung di-deliver ke semua subscribers (fan-out pattern)
- C. Gunakan SQS queue dan tiga consumer yang poll queue yang sama
- D. Simpan event ke DynamoDB dan trigger polling dari setiap service

---

**Soal 4**

Sebuah aplikasi fintech memproses transaksi keuangan menggunakan SQS. Mereka butuh memastikan bahwa:
- Transaksi untuk **akun yang sama diproses secara berurutan**
- Tidak ada **duplikasi processing** untuk transaksi yang sama

Tipe SQS mana yang tepat?

- A. SQS Standard — throughput tinggi, best-effort ordering
- B. **SQS FIFO** — strict ordering per message group ID, dan exactly-once processing
- C. SQS Standard dengan Visibility Timeout panjang
- D. SQS Standard dengan Dead Letter Queue

---

**Soal 5**

Sebuah perusahaan memiliki aplikasi yang perlu melakukan **custom action** (misalnya warm-up cache, deregister dari service discovery) **sebelum** instance EC2 di-terminate oleh Auto Scaling Group saat scale-in.

Fitur Auto Scaling mana yang memungkinkan ini?

- A. Auto Scaling cooldown period
- B. Auto Scaling scheduled action
- C. **Auto Scaling Lifecycle Hooks** — hold instance dalam state `Terminating:Wait` sambil menjalankan custom action, baru terminate setelah action selesai
- D. Auto Scaling instance protection

---

**Soal 6**

Sebuah perusahaan memiliki aplikasi Aurora MySQL di `us-east-1`. Mereka ingin aplikasi di **region `ap-southeast-1`** bisa melakukan **read queries** dengan **latency rendah** tanpa kirim traffic ke US. Untuk DR, jika `us-east-1` down, `ap-southeast-1` bisa di-promote menjadi primary dalam hitungan menit.

Solusi yang paling tepat adalah?

- A. Buat Aurora Read Replica di `ap-southeast-1`
- B. Gunakan **Aurora Global Database** — primary cluster di `us-east-1`, secondary read-only cluster di `ap-southeast-1`, replication lag < 1 detik, bisa di-promote dalam < 1 menit
- C. Gunakan S3 Cross-Region Replication untuk sync data Aurora
- D. Buat Aurora cluster baru di `ap-southeast-1` dan sync secara manual

---

**Soal 7**

Sebuah perusahaan ingin menggunakan **Route 53 Failover routing** untuk arsitektur active-passive. Primary adalah aplikasi di `us-east-1`, secondary adalah static error page di S3. Jika primary unhealthy, traffic harus otomatis pindah ke secondary.

Komponen tambahan apa yang wajib dikonfigurasi agar failover ini bekerja?

- A. Tidak perlu konfigurasi tambahan — Failover routing otomatis bekerja
- B. Aktifkan Route 53 Resolver
- C. **Route 53 Health Check** pada primary record — failover hanya trigger jika health check menyatakan primary unhealthy
- D. Gunakan CloudFront di depan kedua endpoints

---

**Soal 8**

Sebuah aplikasi perlu memproses **data streaming** dari ribuan IoT devices secara real-time, dengan kemampuan untuk **replay data** yang sudah lewat (hingga 7 hari ke belakang) jika ada consumer yang gagal.

Service mana yang paling tepat?

- A. Amazon SQS Standard — simpan messages hingga 14 hari
- B. Amazon SNS — push notification ke subscribers
- C. **Amazon Kinesis Data Streams** — real-time streaming, data retention hingga 365 hari (default 24 jam), bisa di-replay oleh multiple consumers
- D. Amazon EventBridge — event bus untuk routing events

---

**Soal 9**

Sebuah workload Lambda memiliki proses yang terdiri dari banyak langkah berurutan: validasi input → proses data → simpan ke DB → kirim notifikasi. Jika satu langkah gagal, harus ada retry dan rollback yang terstruktur. Tim tidak ingin mengelola retry logic dan state secara manual di dalam kode Lambda.

Service AWS mana yang paling tepat untuk orchestrate workflow ini?

- A. Amazon SQS — chain beberapa queues
- B. Amazon EventBridge — trigger Lambda dari events
- C. **AWS Step Functions** — managed workflow orchestration dengan built-in error handling, retry, dan state management
- D. AWS Lambda Destinations — chain Lambda functions

---

**Soal 10**

Sebuah perusahaan ingin mengurangi beban baca pada RDS MySQL yang sudah mulai overloaded. Sebagian besar queries adalah **read queries untuk data yang jarang berubah** (product catalog, config settings). Mereka ingin solusi yang bisa serve read queries dalam **hitungan milidetik**.

Solusi mana yang paling tepat?

- A. Tambahkan RDS Read Replica
- B. Upgrade RDS ke instance yang lebih besar
- C. Gunakan **Amazon ElastiCache (Redis atau Memcached)** sebagai caching layer di depan RDS — cache hasil query yang sering diakses
- D. Aktifkan RDS Multi-AZ untuk distribusi read queries

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
