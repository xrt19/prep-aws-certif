# Jawaban — Set 09
## Domain: Resilient Architectures

---

## Soal 1 — Jawaban: B

**Amazon RDS Proxy**

**Kenapa B:**
RDS Proxy adalah managed connection pooler yang duduk di antara aplikasi dan RDS/Aurora. Cara kerjanya: RDS Proxy menerima banyak koneksi dari aplikasi (ribuan), lalu **reuse sejumlah kecil koneksi** yang sudah establish ke database. Lambda yang serverless dan bisa spawn banyak instance sekaligus sangat cocok dengan RDS Proxy karena setiap Lambda invocation yang buat koneksi baru akan dialihkan ke pool yang sudah ada. Hasilnya: jumlah koneksi ke RDS tetap terkontrol meski ada ribuan Lambda concurrent.

**Kenapa bukan yang lain:**
- **A** — Upgrade instance menambah max connections, tapi tidak menyelesaikan masalah fundamental: Lambda yang serverless akan terus spawn koneksi baru. Bukan solusi yang scalable.
- **C** — Read Replica mendistribusikan beban baca, tapi tidak menyelesaikan masalah "too many connections" — write connections tetap semua ke primary.
- **D** — Pindah ke EC2 untuk kontrol connection pool sendiri adalah over-engineering dan mengubah arsitektur secara signifikan. RDS Proxy jauh lebih simpel.

**Konsep yang diuji:** RDS Proxy, connection pooling, Lambda + RDS pattern, serverless database connections.

---

## Soal 2 — Jawaban: B

**Aurora otomatis scale storage + failover < 30 detik**

**Kenapa B:**
Amazon Aurora memiliki arsitektur storage yang sangat berbeda dari RDS biasa:
- **Auto-scaling storage**: mulai dari 10 GB, otomatis bertambah dalam increment 10 GB hingga 128 TB — tidak perlu provision atau resize manual
- **Failover cepat**: Aurora menyimpan 6 copies data di 3 AZ. Saat failover ke Aurora Replica, prosesnya biasanya **< 30 detik** karena Replica sudah membaca dari shared storage yang sama — tidak perlu catch-up replication seperti RDS Multi-AZ

**Kenapa bukan yang lain:**
- **A** — Salah. Aurora storage berbeda fundamental — tidak perlu di-provision manual.
- **C** — Salah. RDS Multi-AZ butuh 60–120 detik karena standby harus di-promote dan DNS update. Aurora jauh lebih cepat karena shared storage architecture.
- **D** — Salah. Aurora Replica **memang bisa** digunakan sebagai failover target — itulah salah satu fungsi utamanya selain read scaling.

**Konsep yang diuji:** Aurora storage auto-scaling, Aurora Replica failover, Aurora vs RDS architecture differences.

---

## Soal 3 — Jawaban: B

**SNS topic dengan multiple subscriptions (fan-out pattern)**

**Kenapa B:**
**SNS Fan-out** adalah pattern klasik untuk satu-ke-banyak message delivery. Dengan satu `Publish` ke SNS topic, SNS otomatis deliver message ke **semua subscribers secara paralel dan bersamaan** — bisa berupa SQS queue, Lambda function, HTTP endpoint, email, SMS, mobile push. Ini memenuhi semua kebutuhan: email (SES via Lambda), database update (Lambda), mobile push (SNS Mobile Push endpoint).

**Kenapa bukan yang lain:**
- **A** — Sequential calls berarti jika satu gagal, yang berikutnya tidak jalan. Tidak resilient, tidak paralel.
- **C** — Satu SQS queue dengan tiga consumer: setiap message hanya akan di-consume oleh **satu** consumer (bukan ketiga-tiganya) — ini competing consumers pattern, bukan fan-out.
- **D** — DynamoDB sebagai event store dengan polling menambah latency, complexity, dan DynamoDB cost — anti-pattern untuk event distribution.

**Konsep yang diuji:** SNS fan-out pattern, one-to-many messaging, SNS subscription types.

---

## Soal 4 — Jawaban: B

**SQS FIFO**

**Kenapa B:**
SQS FIFO menyediakan dua guarantee yang persis dibutuhkan:
1. **Strict ordering**: dengan menggunakan `MessageGroupId` (misalnya account ID), semua transaksi untuk akun yang sama diproses **in-order, one at a time**
2. **Exactly-once processing**: menggunakan `MessageDeduplicationId` — message yang sama tidak akan diproses dua kali dalam window 5 menit

**Kenapa bukan yang lain:**
- **A** — SQS Standard adalah **best-effort ordering** (tidak ada garansi urutan) dan **at-least-once delivery** (bisa ada duplikat). Tidak aman untuk transaksi keuangan.
- **C** — Visibility Timeout panjang mencegah re-processing sementara, tapi tidak menjamin ordering dan tidak menyelesaikan masalah duplikasi secara fundamental.
- **D** — DLQ untuk handle failed messages, tidak menyelesaikan masalah ordering atau deduplication.

**Konsep yang diuji:** SQS FIFO vs Standard, MessageGroupId ordering, exactly-once processing, financial transaction use case.

---

## Soal 5 — Jawaban: C

**Auto Scaling Lifecycle Hooks**

**Kenapa C:**
Lifecycle Hooks memungkinkan ASG **meng-pause** instance di state tertentu (`Pending:Wait` saat launch, `Terminating:Wait` saat terminate) sambil menunggu custom action selesai. Untuk use case ini:
1. ASG putuskan untuk terminate instance (scale-in)
2. Instance masuk state `Terminating:Wait`
3. Lifecycle hook notify EventBridge/SNS/SQS
4. Custom action dijalankan (warm-up, deregister dari service discovery)
5. Setelah action selesai, kirim `CompleteLifecycleAction` signal
6. Instance benar-benar di-terminate

**Kenapa bukan yang lain:**
- **A** — Cooldown period adalah jeda waktu setelah scaling activity selesai sebelum scaling berikutnya bisa terjadi — untuk mencegah scaling terlalu agresif. Bukan untuk menjalankan custom action.
- **B** — Scheduled action untuk mengatur kapasitas ASG berdasarkan jadwal, bukan untuk custom pre-terminate action.
- **D** — Instance protection **mencegah** instance ter-terminate oleh scale-in event — kebalikan dari yang dibutuhkan. Ini untuk instances yang sedang menyelesaikan pekerjaan penting.

**Konsep yang diuji:** ASG Lifecycle Hooks, Terminating:Wait state, custom pre-terminate actions.

---

## Soal 6 — Jawaban: B

**Aurora Global Database**

**Kenapa B:**
Aurora Global Database dirancang untuk multi-region use case:
- **Replication**: menggunakan dedicated replication infrastructure (bukan replikasi berbasis binlog), sehingga lag biasanya **< 1 detik**
- **Low-latency reads**: secondary cluster di `ap-southeast-1` bisa serve read queries secara lokal tanpa round-trip ke US
- **Fast DR**: secondary bisa di-promote menjadi primary baru dalam **< 1 menit** (biasanya sekitar 1 menit atau kurang)

**Kenapa bukan yang lain:**
- **A** — Aurora Read Replica biasa hanya dalam satu region. Cross-region Read Replica ada, tapi Global Database jauh lebih optimal: latency replikasi lebih rendah, failover lebih cepat, dan lebih managed.
- **C** — S3 CRR untuk object storage, tidak relevan untuk Aurora database.
- **D** — Manual sync bukan solusi — tidak real-time, tidak managed, error-prone.

**Konsep yang diuji:** Aurora Global Database, cross-region read, < 1 detik replication lag, fast failover DR.

---

## Soal 7 — Jawaban: C

**Route 53 Health Check pada primary record**

**Kenapa C:**
Route 53 Failover routing **tidak bisa mendeteksi kegagalan sendiri** — ia bergantung sepenuhnya pada hasil **health check** yang dikonfigurasi. Tanpa health check, Route 53 tidak tahu apakah primary endpoint masih hidup atau tidak, sehingga failover tidak akan pernah terjadi. Health check melakukan probe (HTTP, HTTPS, TCP) ke endpoint secara berkala dan menentukan apakah record tersebut "healthy" atau tidak.

**Kenapa bukan yang lain:**
- **A** — Salah. Failover routing **memerlukan health check** untuk bisa trigger. Tanpa health check, ia hanya seperti simple routing.
- **B** — Route 53 Resolver untuk DNS resolution di VPC (untuk private DNS), tidak berhubungan dengan failover routing antar endpoints.
- **D** — CloudFront opsional untuk performance, bukan requirement untuk failover routing bekerja.

**Konsep yang diuji:** Route 53 Failover routing, health checks requirement, active-passive architecture.

---

## Soal 8 — Jawaban: C

**Amazon Kinesis Data Streams**

**Kenapa C:**
Kinesis Data Streams dirancang untuk **real-time data streaming** dengan beberapa keunggulan dibanding SQS untuk use case ini:
- **Multiple consumers** bisa membaca data yang sama secara independen (SQS tidak bisa)
- **Data replay**: data tersimpan di shard dan bisa di-replay oleh consumer yang tertinggal — default 24 jam, bisa dikonfigurasi hingga 365 hari
- **Real-time**: latency sangat rendah (milidetik)
- Cocok untuk IoT data ingestion dengan volume tinggi

**Kenapa bukan yang lain:**
- **A** — SQS message setelah di-consume oleh satu consumer akan dihapus (atau setelah visibility timeout) — tidak bisa di-replay oleh consumer lain. Juga tidak dirancang untuk streaming.
- **B** — SNS push notification ke subscribers — tidak support replay dan tidak cocok untuk IoT data streaming volume tinggi.
- **D** — EventBridge untuk routing events berdasarkan rules, tidak dirancang untuk high-volume streaming atau replay capability.

**Konsep yang diuji:** Kinesis Data Streams, real-time streaming, data replay, multiple consumers, IoT use case.

---

## Soal 9 — Jawaban: C

**AWS Step Functions**

**Kenapa C:**
Step Functions adalah **managed workflow orchestration service** yang dirancang persis untuk use case ini:
- Definisikan setiap langkah sebagai state dalam **State Machine**
- Built-in **retry dengan exponential backoff** per step
- Built-in **error handling dan catch** — bisa routing ke compensating transactions jika ada step yang gagal
- **Visual workflow** untuk monitoring eksekusi
- **State persisted** oleh Step Functions — tidak perlu simpan state di Lambda

**Kenapa bukan yang lain:**
- **A** — SQS chain queue tidak punya konsep state machine, error handling terstruktur, atau rollback.
- **B** — EventBridge untuk event-driven routing, bukan untuk multi-step workflow orchestration dengan state.
- **D** — Lambda Destinations untuk routing hasil Lambda (success/failure) ke service lain — hanya bisa forward result, tidak bisa definisikan complex workflow, retry logic, atau rollback.

**Konsep yang diuji:** AWS Step Functions, workflow orchestration, state machine, built-in retry & error handling.

---

## Soal 10 — Jawaban: C

**Amazon ElastiCache sebagai caching layer**

**Kenapa C:**
ElastiCache (Redis atau Memcached) adalah **in-memory cache** yang menyimpan hasil query di RAM — latency baca **sub-millisecond** dibanding RDS yang bisa puluhan milidetik. Pattern yang digunakan: **cache-aside** (lazy loading) — aplikasi cek cache dulu, jika tidak ada (cache miss) baru query ke RDS dan simpan hasilnya ke cache. Untuk data yang jarang berubah (product catalog, config), TTL bisa di-set panjang sehingga cache hit rate tinggi dan beban RDS turun drastis.

**Kenapa bukan yang lain:**
- **A** — Read Replica mendistribusikan read load ke replica, tapi latency tetap sama dengan RDS (disk-based). Tidak bisa mencapai sub-millisecond. Juga tetap butuh query ke database.
- **B** — Upgrade instance menambah resource tapi tidak mengurangi jumlah queries ke RDS — hanya sementara sebelum load naik lagi.
- **D** — **Exam trap klasik**: RDS Multi-AZ standby **tidak melayani read traffic** — ia hanya hot standby untuk failover. Semua read dan write tetap ke primary.

**Konsep yang diuji:** ElastiCache caching pattern, cache-aside, sub-millisecond latency, RDS Multi-AZ tidak serve reads.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | RDS Proxy, connection pooling, Lambda + RDS |
| 2 | Aurora auto-scaling storage, failover < 30 detik |
| 3 | SNS fan-out pattern, one-to-many messaging |
| 4 | SQS FIFO, ordering + exactly-once, MessageGroupId |
| 5 | ASG Lifecycle Hooks, pre-terminate custom action |
| 6 | Aurora Global Database, cross-region < 1 detik lag |
| 7 | Route 53 Failover routing + health check requirement |
| 8 | Kinesis Data Streams, replay, multiple consumers |
| 9 | AWS Step Functions, workflow orchestration |
| 10 | ElastiCache caching layer, Multi-AZ tidak serve reads |
