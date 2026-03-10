# Soal Latihan — Set 23
## Domain: Cost-Optimized Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah perusahaan menjalankan **Amazon EKS cluster** dengan worker nodes EC2. Banyak pods kecil yang berjalan secara sporadis dan tidak memerlukan dedicated EC2 nodes penuh. Tim ingin mengurangi biaya worker nodes tanpa mengorbankan availability.

Strategi mana yang paling cost-optimal?

- A. Gunakan EC2 Reserved Instances untuk semua worker nodes
- B. Upgrade ke instance type yang lebih besar agar lebih banyak pods per node
- C. Pindahkan semua pods ke EKS Fargate mode
- D. **Gunakan Karpenter atau Cluster Autoscaler dengan Spot Instances untuk worker nodes** — scale down saat tidak ada pods; gunakan Spot untuk worker nodes yang toleran interruption; mix On-Demand (untuk system pods kritis) dan Spot (untuk aplikasi workloads); bisa hemat 60–80%

---

**Soal 2**

Sebuah perusahaan memiliki **Amazon CloudWatch Logs** yang menyimpan log dari ratusan Lambda functions, EC2 instances, dan ECS tasks. Biaya CloudWatch Logs sangat tinggi karena semua logs disimpan dengan retention "Never Expire".

Cara paling efektif untuk mengurangi biaya CloudWatch Logs?

- A. Kurangi log level ke ERROR saja untuk semua applications
- B. Disable logging untuk non-production environments
- C. **Atur retention policy per Log Group** — set retention sesuai kebutuhan: misalnya production logs 30 hari, dev/test logs 7 hari, security/audit logs 1 tahun; CloudWatch Logs di-charge per GB stored; setelah retention period, logs otomatis dihapus tanpa biaya tambahan
- D. Ekspor semua logs ke S3 kemudian hapus dari CloudWatch

---

**Soal 3**

Sebuah perusahaan menggunakan **Amazon SQS** untuk message queuing antara microservices. Mereka menggunakan **short polling** dan banyak consumers yang terus-menerus poll queue bahkan saat queue kosong. Biaya SQS sangat tinggi karena jumlah API requests yang besar.

Cara mengurangi biaya SQS secara efektif?

- A. Ganti ke SNS untuk push-based messaging
- B. Kurangi jumlah consumer instances
- C. **Aktifkan SQS Long Polling (WaitTimeSeconds=20)** — setiap Long Polling request menunggu hingga 20 detik untuk ada message sebelum return empty response; mengurangi jumlah API calls secara drastis saat queue sepi; SQS di-charge per API request ($0.40 per million requests)
- D. Pindahkan ke Kinesis Data Streams untuk throughput lebih tinggi

---

**Soal 4**

Sebuah perusahaan memiliki **aplikasi multi-tier** dengan web tier (ALB + EC2) dan database tier (RDS). Mereka ingin mengurangi biaya RDS tanpa mengubah application code. Dari analisis, sebagian besar query adalah read queries untuk data yang **tidak sering berubah** (katalog produk, referensi data).

Cara paling tepat?

- A. Upgrade ke RDS instance yang lebih besar
- B. Aktifkan RDS Multi-AZ untuk reduce read load
- C. Gunakan RDS Read Replica untuk semua read traffic
- D. **Tambahkan ElastiCache di depan RDS** — cache hasil query yang sering diakses di memori; mengurangi RCU (database load) sehingga bisa downsize RDS instance; data yang tidak sering berubah sangat cocok untuk cache dengan TTL panjang

---

**Soal 5**

Sebuah perusahaan memiliki **S3 bucket** yang menyimpan jutaan objek kecil (rata-rata 1 KB per objek). Biaya S3 sangat tinggi padahal total storage hanya beberapa GB. Setelah diperiksa, biaya terbesar bukan dari storage tapi dari **S3 API requests** (GET, PUT, LIST).

Apa penyebab utama biaya tinggi ini dan bagaimana menguranginya?

- A. Pindahkan objek ke S3 Glacier untuk kurangi request cost
- B. Aktifkan S3 Intelligent-Tiering untuk kurangi storage cost
- C. Gunakan S3 Batch Operations untuk proses semua objek sekaligus
- D. **Gabungkan objek kecil menjadi objek yang lebih besar (aggregation/batching)** — S3 di-charge per request bukan per byte untuk operasi seperti PUT dan GET; jutaan objek 1 KB = jutaan requests; gabungkan menjadi ribuan file ZIP/TAR/Parquet yang lebih besar = ribuan requests, jauh lebih murah

---

**Soal 6**

Sebuah perusahaan menjalankan **EC2 instances** untuk aplikasi yang memiliki traffic predictable — selalu tinggi di hari kerja dan hampir tidak ada di weekend. Mereka sudah menggunakan Auto Scaling. Tim ingin mengoptimalkan biaya lebih lanjut dengan kombinasi purchasing options.

Strategi mana yang paling cost-optimal?

- A. Semua On-Demand dengan Auto Scaling
- B. Semua Reserved Instances dengan jumlah minimum yang selalu running
- C. Semua Spot Instances untuk maksimum savings
- D. **Kombinasi: Reserved Instances untuk baseline capacity + Spot Instances untuk burst capacity** — Reserved untuk minimum instances yang selalu running (steady-state 5 hari/minggu); Spot untuk instances tambahan saat traffic tinggi; jika Spot di-interrupt, ASG launch On-Demand sebagai fallback

---

**Soal 7**

Sebuah perusahaan memiliki **EC2 instances** di `us-east-1` yang perlu akses ke S3 bucket di `us-west-2`. Traffic data transfer antar region ini sangat besar (terabytes per bulan) dan biayanya mahal.

Cara paling tepat untuk mengurangi cross-region data transfer cost?

- A. Gunakan S3 Transfer Acceleration untuk mempercepat transfer
- B. Kompres semua data sebelum transfer antar region
- C. **Pindahkan S3 bucket ke region yang sama dengan EC2 (`us-east-1`)** — data transfer antar region di-charge ($0.02/GB); data transfer dalam satu region antara EC2 dan S3 via private IP = gratis; pindahkan bucket atau replikasi data ke region yang sama
- D. Gunakan AWS Direct Connect untuk transfer antar region

---

**Soal 8**

Sebuah startup menggunakan **AWS untuk development environment**. Mereka memiliki banyak resources yang di-buat oleh developer untuk eksperimen, tapi tidak selalu di-cleanup setelah selesai. Biaya terus bertambah dari resources yang "lupa" di-terminate.

Solusi governance mana yang paling tepat untuk mencegah cost sprawl ini?

- A. Review manual biaya setiap minggu dan hapus resources yang tidak diperlukan
- B. Buat separate AWS account untuk setiap developer
- C. **Gunakan AWS Budgets + budget actions** — set budget threshold; ketika cost melebihi threshold, budget action otomatis apply SCP atau IAM policy untuk restrict kemampuan create resources baru, atau kirim SNS notification; dikombinasikan dengan tagging policy untuk accountability
- D. Aktifkan AWS Config untuk track semua resource creation

---

**Soal 9**

Sebuah perusahaan memiliki **workload HPC** yang sangat besar yang harus diselesaikan dalam waktu singkat (deadline ketat). Mereka butuh ribuan vCPU dalam waktu singkat — lebih dari On-Demand quota mereka. Mereka mau toleransi interruption asalkan ada fallback mechanism.

Strategi mana yang tepat?

- A. Request quota increase untuk On-Demand dan launch semua sebagai On-Demand
- B. Gunakan Reserved Instances untuk HPC workload
- C. Buat multiple Spot Instance requests di berbagai instance types dan AZs
- D. **Gunakan EC2 Spot dengan Spot Blocks (defined duration) atau Spot Fleet dengan diversifikasi** — Spot Fleet request dari multiple pools (instance types × AZs) untuk maximize capacity availability; diversifikasi mengurangi kemungkinan semua instances di-interrupt bersamaan; untuk workload dengan deadline, Spot Fleet dengan `capacityOptimized` strategy cocok

---

**Soal 10**

Sebuah perusahaan menggunakan **AWS WAF** di depan ALB mereka. Setelah melihat tagihan, mereka menemukan biaya WAF sangat tinggi karena **jumlah rules yang sangat banyak** dan traffic yang besar. Mereka ingin mengurangi biaya WAF tanpa mengurangi protection level secara signifikan.

Strategi mana yang paling tepat?

- A. Hapus WAF dan gunakan Security Groups saja untuk filtering
- B. Pindahkan WAF ke CloudFront agar lebih murah
- C. **Gunakan AWS Managed Rules sebagai pengganti custom rules** — AWS Managed Rules adalah rule groups pre-built yang di-charge flat per rule group (bukan per rule); satu Managed Rule Group bisa menggantikan puluhan custom rules; lebih murah, selalu up-to-date dengan threat intelligence terbaru
- D. Kurangi sampling rate WAF logging untuk kurangi log storage cost

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
