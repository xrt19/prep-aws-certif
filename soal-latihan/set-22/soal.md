# Soal Latihan — Set 22
## Domain: Cost-Optimized Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah perusahaan memiliki **Lambda functions** yang berjalan lama (10–15 menit) memproses data transformation. Setiap invocation membutuhkan memory 3 GB. Mereka ingin mengurangi biaya Lambda tanpa mengubah logika transformasi.

Strategi mana yang paling tepat?

- A. Naikkan memory ke 10 GB untuk mempercepat eksekusi
- B. Pindahkan ke EC2 On-Demand karena Lambda mahal untuk long-running tasks
- C. **Pindahkan ke AWS Fargate atau EC2 Spot untuk batch processing** — Lambda di-charge per GB-second; 3 GB × 15 menit = 2700 GB-seconds per invocation; untuk long-running heavy tasks, Fargate atau Batch dengan Spot jauh lebih hemat dari Lambda
- D. Aktifkan Lambda Provisioned Concurrency untuk kurangi cold start overhead

---

**Soal 2**

Sebuah perusahaan memiliki **aplikasi yang jarang digunakan** — hanya beberapa kali per hari dengan traffic burst singkat. Mereka menggunakan EC2 t3.medium yang berjalan 24/7. Biaya terasa tinggi untuk utilization yang sangat rendah.

Arsitektur mana yang paling cost-optimal untuk pola traffic ini?

- A. Ganti ke EC2 t3.nano yang lebih kecil
- B. Gunakan Reserved Instance 1 tahun untuk diskon
- C. Pindahkan ke EC2 Spot Instance untuk lebih murah
- D. **Refactor ke serverless: API Gateway + Lambda** — charge hanya saat ada invocations; tidak ada idle EC2 cost; untuk aplikasi yang berjalan beberapa kali per hari, biaya Lambda bisa jauh lebih rendah dari EC2 yang terus berjalan

---

**Soal 3**

Sebuah perusahaan memiliki **EBS volumes** yang attached ke EC2 instances. Setelah EC2 instances di-terminate, banyak EBS volumes yang tertinggal dalam state "available" (tidak ter-attach ke instance manapun) dan terus di-charge.

Cara otomatis untuk mencegah EBS volumes tidak ter-attach terus di-charge?

- A. Buat CloudWatch alarm untuk setiap EBS volume
- B. Gunakan AWS Config rule untuk detect unattached volumes
- C. **Aktifkan "Delete on Termination" flag saat launch EC2** — ketika EC2 di-terminate, EBS root volume (dan optional volumes lain dengan flag ini) otomatis di-delete; tidak ada manual cleanup yang diperlukan
- D. Buat Lambda scheduled yang scan dan delete unattached EBS setiap hari

---

**Soal 4**

Sebuah perusahaan menjalankan **Amazon RDS** untuk aplikasi mereka. Dari Cost Explorer, mereka melihat biaya **snapshot backup** sangat tinggi karena setiap hari ada automated backup dan manual snapshots yang jarang dihapus.

Cara paling tepat untuk mengoptimalkan backup cost?

- A. Matikan automated backup untuk hemat biaya
- B. Kurangi retention period automated backup ke 1 hari
- C. **Atur retention period yang tepat dan hapus manual snapshots yang tidak diperlukan** — automated backup gratis sampai ukuran database (first 100% storage gratis di RDS); biaya muncul dari manual snapshots yang terakumulasi dan tidak dihapus; buat lifecycle policy untuk hapus manual snapshots setelah X hari
- D. Pindahkan semua snapshots ke S3 Glacier untuk storage lebih murah

---

**Soal 5**

Sebuah perusahaan memiliki **EC2 instances** yang menggunakan **Elastic IP addresses**. Beberapa Elastic IPs tidak ter-asosiasikan ke instance yang running (misalnya instance-nya sudah di-stop atau di-terminate).

Mengapa ini menjadi masalah biaya, dan bagaimana cara menghindarinya?

- A. Elastic IP gratis selama masih terdaftar di akun
- B. Elastic IP di-charge hanya saat ter-asosiasikan ke instance yang running
- C. **Elastic IP di-charge saat TIDAK ter-asosiasikan ke instance yang running** — AWS charge $0.005/jam per Elastic IP yang tidak digunakan; solusi: release Elastic IP yang tidak diperlukan, atau asosiasikan ke running instance
- D. Elastic IP di-charge berdasarkan jumlah requests yang melewatinya

---

**Soal 6**

Sebuah perusahaan menggunakan **AWS Organizations** dengan lebih dari 20 accounts. Mereka ingin **centralized billing** dan memanfaatkan **volume discounts** — semakin banyak usage, semakin murah per unit untuk beberapa services seperti S3 dan data transfer.

Fitur AWS Organizations mana yang memberikan benefit ini?

- A. AWS Control Tower — landing zone untuk multi-account governance
- B. AWS Service Catalog — standardized product portfolios
- C. **Consolidated Billing** — semua usage dari member accounts digabungkan di payer account; total usage gabungan memenuhi threshold yang lebih tinggi untuk volume discounts (tiered pricing S3, data transfer, dll); satu tagihan untuk semua accounts
- D. AWS Config Aggregator — aggregate compliance data dari semua accounts

---

**Soal 7**

Sebuah perusahaan ingin menganalisis **pola penggunaan EC2** mereka selama 3 bulan terakhir untuk menentukan apakah Reserved Instances atau Savings Plans akan menghemat biaya. Mereka butuh rekomendasi spesifik dari AWS.

Tools mana yang paling tepat?

- A. AWS Trusted Advisor — cek apakah ada resources yang under-utilized
- B. AWS Cost Explorer RI recommendations — analisis historical usage dan rekomendasikan RI yang optimal
- C. AWS Compute Optimizer — rekomendasikan right-sizing instance
- D. **AWS Cost Explorer dengan Savings Plans recommendations** — analisis 7, 30, atau 60 hari historical usage; rekomendasikan Compute Savings Plans atau EC2 Instance Savings Plans dengan projected savings; pilih commitment amount dan term yang sesuai

---

**Soal 8**

Sebuah perusahaan menyimpan **database backup** di S3 setiap hari. Backup harian disimpan selama 30 hari, backup mingguan selama 1 tahun, dan backup bulanan selama 5 tahun. Saat ini semuanya di S3 Standard.

Konfigurasi mana yang paling cost-optimal?

- A. Simpan semua backup di S3 Glacier Deep Archive untuk harga terendah
- B. Gunakan S3 Intelligent-Tiering untuk semua backup
- C. **Buat S3 Lifecycle rules**: backup 0–30 hari di S3 Standard → backup 30 hari–1 tahun di S3 Standard-IA → backup 1 tahun–5 tahun di S3 Glacier Deep Archive → delete setelah 5 tahun
- D. Simpan backup harian di EBS snapshots bukan S3 untuk lebih murah

---

**Soal 9**

Sebuah perusahaan memiliki **EC2 instances** yang sering over-provisioned — instance besar yang CPU dan memory-nya hanya terpakai 10–15% secara rata-rata. Tim ingin right-size instances tanpa harus manually analyze setiap instance.

AWS service mana yang memberikan rekomendasi right-sizing otomatis?

- A. AWS Cost Explorer — analisis biaya dan usage
- B. AWS Trusted Advisor — best practice checks
- C. Amazon CloudWatch — monitor metrics CPU dan memory
- D. **AWS Compute Optimizer** — analisis CloudWatch metrics historis (CPU, memory, network, disk) dan rekomendasikan instance type yang optimal; bisa hemat 25% atau lebih; mendukung EC2, Lambda, ECS on Fargate, EBS volumes, dan Auto Scaling groups

---

**Soal 10**

Sebuah perusahaan memiliki **microservices** yang saling berkomunikasi banyak via REST API calls melewati **NAT Gateway**. Traffic antar services sangat tinggi (terabytes per bulan). Setelah ditelusuri, semua services berada di **VPC yang sama**.

Apa yang salah dan bagaimana cara memperbaikinya untuk mengurangi biaya?

- A. Gunakan VPC Peering untuk koneksi antar services
- B. Aktifkan VPC Flow Logs untuk track traffic patterns
- C. Pindahkan services ke public subnet agar tidak perlu NAT
- D. **Gunakan private IP addresses untuk komunikasi antar services dalam VPC yang sama** — komunikasi antar EC2/containers dalam satu VPC via private IP tidak di-charge; NAT Gateway seharusnya hanya untuk traffic ke internet; jika services menggunakan public DNS names yang resolve ke public IP, traffic keluar ke internet via NAT walaupun destination ada di VPC yang sama

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
