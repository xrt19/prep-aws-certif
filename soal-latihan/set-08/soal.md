# Soal Latihan — Set 08
## Domain: Resilient Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah perusahaan menjalankan aplikasi kritis di EC2 dengan RDS MySQL. Saat ini semua resource hanya ada di satu Availability Zone. Mereka khawatir jika AZ tersebut mengalami outage, aplikasi akan down.

Perubahan arsitektur mana yang paling efektif untuk mengatasi ini dengan **downtime minimal saat failover**?

- A. Buat manual snapshot RDS setiap hari dan restore jika terjadi outage
- B. Aktifkan **RDS Multi-AZ** — secondary standby instance otomatis di AZ berbeda, failover otomatis dalam hitungan menit
- C. Buat Read Replica RDS di AZ lain dan promote manual jika primary down
- D. Pindahkan database ke EC2 instance di AZ lain dan replikasi manual

---

**Soal 2**

Sebuah aplikasi e-commerce mengalami traffic spike setiap hari Senin pagi. Tim ingin infrastruktur EC2 **otomatis scale out** sebelum spike terjadi (bukan setelah), karena warm-up time EC2 sekitar 5 menit.

Fitur Auto Scaling mana yang paling tepat?

- A. Dynamic scaling dengan target tracking policy — scale saat CPU sudah tinggi
- B. **Scheduled scaling** — tambah kapasitas otomatis sesuai jadwal sebelum spike terjadi
- C. Simple scaling — tambah 1 instance setiap kali CloudWatch alarm trigger
- D. Scale manual setiap Senin pagi oleh tim ops

---

**Soal 3**

Sebuah perusahaan memiliki aplikasi yang memproses order via **SQS queue**. Consumer (EC2) kadang butuh waktu lama untuk proses satu message (hingga 30 menit). Selama diproses, message tidak boleh diambil oleh consumer lain.

Konfigurasi SQS mana yang harus diatur?

- A. Gunakan SQS FIFO queue — otomatis lock message yang sedang diproses
- B. Set **Visibility Timeout** ke nilai yang cukup lama (misalnya 35 menit) agar message tidak re-appear selama diproses
- C. Delete message segera setelah diterima, sebelum selesai diproses
- D. Gunakan SQS Dead Letter Queue untuk hold message selama processing

---

**Soal 4**

Sebuah perusahaan global ingin routing traffic ke region terdekat dari user untuk **mengurangi latency**. Jika region terdekat tidak tersedia, traffic harus otomatis di-route ke region lain.

Routing policy Route 53 mana yang paling tepat?

- A. Simple routing — satu record untuk semua traffic
- B. Weighted routing — bagi traffic berdasarkan persentase ke beberapa regions
- C. **Latency-based routing** — Route 53 route ke region dengan latency terendah, bisa dikombinasikan dengan health checks untuk failover
- D. Geolocation routing — route berdasarkan lokasi geografis user

---

**Soal 5**

Sebuah perusahaan menjalankan aplikasi stateless di belakang ALB dengan Auto Scaling Group. Mereka ingin memastikan bahwa ketika ada instance yang **unhealthy**, instance tersebut otomatis diganti tanpa intervensi manual.

Mekanisme mana yang bekerja untuk ini?

- A. CloudWatch alarm yang notify tim ops untuk terminate instance
- B. **Auto Scaling health checks** — ASG otomatis terminate instance unhealthy dan launch pengganti baru
- C. ALB health check yang restart instance yang gagal
- D. Route 53 health check yang remove instance dari DNS

---

**Soal 6**

Sebuah startup memiliki aplikasi dengan database RDS. Mereka ingin kemampuan untuk **restore database ke kondisi kapanpun dalam 5 hari terakhir** (misalnya restore ke kondisi database jam 14:32 kemarin) karena khawatir dengan human error.

Fitur RDS mana yang memungkinkan ini?

- A. RDS automated backups dengan daily snapshots — restore ke snapshot terdekat
- B. **RDS Point-in-Time Recovery (PITR)** — restore ke detik manapun dalam retention period menggunakan transaction logs
- C. RDS Manual snapshots yang dibuat setiap jam
- D. RDS Read Replica — promote replica ke kondisi yang diinginkan

---

**Soal 7**

Sebuah perusahaan memiliki data penting di S3 bucket di `us-east-1`. Mereka ingin data tersebut **otomatis tersedia di `eu-west-1`** juga untuk DR (Disaster Recovery), dan setiap object baru yang di-upload harus **otomatis ter-replicate**.

Solusi yang paling tepat adalah?

- A. Buat Lambda function yang copy object ke bucket lain setiap kali ada upload baru
- B. Gunakan **S3 Cross-Region Replication (CRR)** — otomatis replicate objects baru ke bucket di region lain
- C. Gunakan AWS DataSync untuk sync kedua bucket setiap jam
- D. Gunakan S3 Transfer Acceleration untuk upload ke kedua regions sekaligus

---

**Soal 8**

Sebuah perusahaan menjalankan aplikasi web dengan backend di banyak EC2 instances. Mereka butuh load balancer yang bisa:
- Route berdasarkan **path URL** (`/api/*` ke satu target group, `/images/*` ke target group lain)
- Support **WebSocket connections**
- **Health check** per target group

Load balancer mana yang paling tepat?

- A. Classic Load Balancer (CLB)
- B. Network Load Balancer (NLB)
- C. **Application Load Balancer (ALB)**
- D. Gateway Load Balancer (GWLB)

---

**Soal 9**

Sebuah perusahaan ingin mendesain sistem yang bisa bertahan dari **kegagalan total satu region AWS**. Aplikasi harus tetap berjalan dengan RTO (Recovery Time Objective) kurang dari 1 menit.

Strategi disaster recovery mana yang memenuhi requirement ini?

- A. Backup and Restore — backup data ke region lain, restore jika region utama down
- B. Pilot Light — jalankan versi minimal di region lain, scale up saat DR
- C. Warm Standby — jalankan versi scaled-down di region lain, scale up saat DR
- D. **Multi-Site Active-Active** — jalankan aplikasi penuh di dua region sekaligus, traffic langsung di-route ke region yang masih hidup

---

**Soal 10**

Sebuah aplikasi menggunakan SQS untuk decouple producers dan consumers. Producers kadang mengirim **burst of messages** dalam waktu singkat — jauh melebihi kapasitas consumer untuk memproses. Tim ingin memastikan tidak ada message yang hilang dan consumer tidak kelebihan beban.

Karakteristik SQS mana yang secara native menangani skenario ini?

- A. SQS FIFO — memastikan message diproses berurutan sehingga tidak ada yang hilang
- B. **SQS sebagai buffer** — message tersimpan di queue hingga 14 hari, consumer bisa proses sesuai kemampuan tanpa message loss
- C. SQS Dead Letter Queue — message yang tidak bisa diproses disimpan di DLQ
- D. SQS Long Polling — consumer bisa poll lebih efisien untuk handle burst

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
