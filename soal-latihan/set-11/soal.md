# Soal Latihan — Set 11
## Domain: Resilient Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah aplikasi consumer yang berjalan di EC2 polling SQS queue secara terus-menerus. Tim menemukan bahwa sebagian besar polling requests mengembalikan **empty responses** karena queue sering kosong, mengakibatkan biaya API calls yang tidak perlu dan CPU usage yang tinggi.

Konfigurasi SQS mana yang harus diaktifkan?

- A. Naikkan Visibility Timeout agar message lebih lama di queue
- B. Aktifkan SQS Dead Letter Queue untuk handle empty responses
- C. **Aktifkan SQS Long Polling** (`WaitTimeSeconds` hingga 20 detik) — consumer menunggu hingga ada message atau timeout, mengurangi empty responses secara drastis
- D. Gunakan SQS FIFO untuk mengurangi jumlah polling requests

---

**Soal 2**

Sebuah perusahaan ingin mem-**backup data log** dari EC2 instances secara real-time ke S3 untuk long-term storage, dengan kemampuan transformasi data (konversi ke Parquet, kompresi) sebelum disimpan. Mereka tidak ingin manage infrastruktur streaming.

Service mana yang paling tepat?

- A. Amazon Kinesis Data Streams + Lambda untuk transform dan upload ke S3
- B. **Amazon Kinesis Data Firehose** — managed delivery stream yang otomatis batch, transform (via Lambda), compress, dan deliver ke S3 tanpa manage infrastruktur
- C. AWS DataSync — sync data dari EC2 ke S3 secara periodik
- D. Amazon SQS + Lambda untuk buffer dan upload ke S3

---

**Soal 3**

Sebuah aplikasi web memiliki Auto Scaling Group dengan target tracking policy yang di-set ke **CPU utilization 60%**. Saat ini CPU rata-rata 80%, tapi ASG tidak segera scale out karena masih dalam cooldown period dari scaling action sebelumnya.

Apa yang seharusnya terjadi, dan mana yang paling tepat untuk mempercepat respons?

- A. Ganti ke Simple Scaling — tidak punya cooldown period
- B. Naikkan target CPU ke 80% agar tidak perlu scale out
- C. **Kurangi default cooldown period** atau gunakan instance warm-up time yang lebih pendek sesuai actual warm-up time EC2 instances
- D. Tambahkan Scheduled Scaling sebagai override

---

**Soal 4**

Sebuah perusahaan memiliki aplikasi DynamoDB dengan read traffic yang sangat tinggi — hingga **jutaan read requests per detik** untuk data yang sama berulang-ulang (misalnya item catalog yang populer). Latency read DynamoDB yang biasanya single-digit milidetik masih terlalu tinggi untuk use case ini.

Solusi mana yang tepat untuk mencapai **microsecond latency**?

- A. Aktifkan DynamoDB Auto Scaling untuk handle read traffic tinggi
- B. Gunakan DynamoDB Global Tables untuk distribusi read traffic
- C. Upgrade ke DynamoDB On-Demand capacity mode
- D. **Tambahkan DynamoDB Accelerator (DAX)** — in-memory cache khusus DynamoDB, latency microsecond, fully compatible dengan DynamoDB API

---

**Soal 5**

Sebuah perusahaan ingin men-deploy aplikasi web mereka ke **multiple Availability Zones** menggunakan **Elastic Beanstalk**. Mereka sedang dalam proses deployment versi baru dan ingin memastikan bahwa **tidak ada downtime** selama deployment, meskipun prosesnya lebih lambat.

Deployment policy Elastic Beanstalk mana yang tepat?

- A. All at once — deploy ke semua instances sekaligus, cepat tapi ada downtime
- B. **Rolling** — update instances dalam batch kecil, kapasitas berkurang sementara tapi tidak ada downtime total
- C. Immutable — launch batch baru instances, zero downtime tapi butuh kapasitas double sementara
- D. Blue/Green — deploy ke environment terpisah, zero downtime tapi butuh full environment baru

---

**Soal 6**

Sebuah perusahaan menggunakan **Amazon EFS** (Elastic File System) sebagai shared storage untuk EC2 instances mereka. Mereka khawatir jika satu AZ mengalami outage, aplikasi akan kehilangan akses ke file system.

Bagaimana EFS menangani AZ failure secara default?

- A. EFS hanya tersedia di satu AZ — harus manual backup ke AZ lain
- B. EFS butuh konfigurasi Multi-AZ secara manual seperti RDS
- C. **EFS secara default menyimpan data redundant di multiple AZs** dalam satu region — mount targets tersedia di setiap AZ, jika satu AZ down, EC2 di AZ lain masih bisa akses via mount target mereka
- D. EFS otomatis failover ke backup S3 jika AZ down

---

**Soal 7**

Sebuah perusahaan memiliki aplikasi yang di-serve via **CloudFront**. Origin utama adalah ALB di `us-east-1`. Untuk DR, mereka ingin jika origin utama tidak tersedia, CloudFront otomatis route ke **origin backup** (ALB di `us-west-2`).

Fitur CloudFront mana yang tepat?

- A. CloudFront geo-restriction — block traffic ke region yang down
- B. Route 53 Failover routing di depan CloudFront
- C. **CloudFront Origin Groups dengan origin failover** — definisikan primary dan secondary origin, CloudFront otomatis switch ke secondary jika primary return 5xx errors
- D. CloudFront Lambda@Edge untuk detect dan redirect ke backup

---

**Soal 8**

Sebuah perusahaan ingin routing traffic ke **aplikasi yang berbeda** berdasarkan **negara asal user**: user dari Indonesia dan Malaysia ke server di `ap-southeast-1`, user dari Jerman dan Prancis ke server di `eu-central-1`, dan semua lainnya ke `us-east-1` sebagai default.

Routing policy Route 53 mana yang tepat?

- A. Latency-based routing — route ke server dengan latency terendah
- B. **Geolocation routing** — route berdasarkan lokasi geografis user (negara/benua), bisa set default untuk yang tidak cocok
- C. Weighted routing — bagi traffic berdasarkan persentase ke setiap region
- D. Geoproximity routing — route berdasarkan jarak ke resource

---

**Soal 9**

Sebuah startup menggunakan RDS MySQL. Database mereka tumbuh sangat cepat dan tim sering harus **manually resize storage** sebelum penuh, yang berisiko downtime jika terlambat.

Fitur RDS mana yang menghilangkan kebutuhan resize manual ini?

- A. RDS Multi-AZ — standby instance punya storage lebih besar
- B. RDS Read Replica — distribusi data ke replica mengurangi storage primary
- C. **RDS Storage Auto Scaling** — otomatis expand storage saat mendekati batas, tanpa downtime, hingga maximum yang di-set
- D. Migrate ke Aurora yang auto-scale storage

---

**Soal 10**

Sebuah perusahaan menjalankan ECS cluster dengan EC2 launch type. Mereka ingin cluster otomatis **menambah EC2 instances** ke cluster saat task tidak bisa di-schedule karena insufficient capacity, dan **mengurangi instances** saat cluster underutilized.

Fitur ECS mana yang mengelola scaling infrastruktur EC2 di bawah cluster secara otomatis?

- A. ECS Service Auto Scaling — scale jumlah tasks berdasarkan metrik
- B. CloudWatch alarm yang trigger Auto Scaling Group
- C. **ECS Cluster Auto Scaling dengan Capacity Providers** — link ASG ke ECS cluster, otomatis scale EC2 instances berdasarkan kebutuhan scheduling tasks
- D. AWS Auto Scaling Plans untuk ECS clusters

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
