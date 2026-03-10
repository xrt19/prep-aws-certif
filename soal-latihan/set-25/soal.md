# Soal Latihan — Set 25
## Domain: Mixed (Final Set)

> 10 soal — campuran dari 4 domain. Ujian final sebelum exam.
> Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1** *(Secure Architectures)*

Sebuah perusahaan memiliki **multi-account AWS environment** dengan 50+ accounts. Security team ingin memastikan bahwa **tidak ada account yang bisa menonaktifkan CloudTrail** — bahkan oleh account root sekalipun. Mereka juga ingin enforce bahwa semua S3 buckets di seluruh organization harus memiliki enkripsi enabled.

Kombinasi kontrol mana yang paling tepat?

- A. AWS Config rules di setiap account untuk detect dan alert pelanggaran
- B. IAM Permission Boundaries di setiap account untuk restrict admin users
- C. **SCP (Service Control Policies) di AWS Organizations** — SCP adalah preventive control yang berlaku untuk semua principals di member accounts termasuk account root; bisa deny `cloudtrail:StopLogging` dan `cloudtrail:DeleteTrail` di seluruh organization; juga bisa deny `s3:PutBucketEncryption` dengan condition yang enforce enkripsi — tidak bisa di-bypass oleh siapapun di member account
- D. AWS Control Tower Guardrails untuk enforce compliance

---

**Soal 2** *(Resilient Architectures)*

Sebuah perusahaan menjalankan **aplikasi e-commerce** yang harus memiliki **RTO < 15 menit** dan **RPO < 1 menit** untuk database mereka. Biaya harus tetap reasonable — mereka tidak bisa afford Multi-Site Active-Active.

Strategi DR mana yang tepat?

- A. Backup and Restore — snapshot harian ke S3, restore saat disaster
- B. Pilot Light — hanya database minimal yang running di DR region
- C. Multi-Site Active-Active — traffic di-serve dari dua region sekaligus
- D. **Warm Standby** — versi scaled-down dari production selalu running di DR region; database di-replikasi secara continuous (Aurora Global Database atau RDS read replica cross-region); saat failover, scale up DR environment; RTO 10–15 menit, RPO detik

---

**Soal 3** *(High-Performing Architectures)*

Sebuah perusahaan menjalankan **aplikasi mobile** yang serve API ke jutaan pengguna global. Mereka menggunakan API Gateway + Lambda di `us-east-1`. Users di Asia Pasifik mengeluhkan latency tinggi (800–1000ms) untuk API calls sederhana yang tidak butuh data dari database.

Solusi terbaik untuk mengurangi latency bagi users Asia Pasifik?

- A. Deploy API Gateway + Lambda di region `ap-southeast-1` (Singapore)
- B. Gunakan AWS Global Accelerator untuk route API traffic ke us-east-1 lebih cepat
- C. Aktifkan API Gateway Edge-optimized endpoint
- D. **Gunakan Lambda@Edge atau CloudFront Functions di depan API Gateway** — untuk API calls sederhana yang tidak butuh database (misalnya validasi token, simple transformation, static config), jalankan logic di CloudFront edge location terdekat user; latency turun dari 800ms menjadi < 50ms karena diproses di edge Asia

---

**Soal 4** *(Cost-Optimized Architectures)*

Sebuah perusahaan menerima **file CSV besar** dari partner setiap hari yang perlu di-process dan hasilnya disimpan ke database. Processing membutuhkan 2 jam menggunakan Python scripts. Mereka saat ini menjalankan EC2 t3.large yang running 24/7 hanya untuk menunggu file ini.

Arsitektur paling cost-optimal?

- A. Ganti ke EC2 t3.micro yang lebih kecil untuk hemat biaya
- B. Jadwalkan EC2 start/stop sesuai waktu file diterima
- C. Gunakan Reserved Instance t3.large untuk diskon
- D. **S3 Event Notification → SQS → AWS Batch dengan Spot Instances** — saat file tiba di S3, trigger Batch job; Batch provision Spot Instance hanya saat diperlukan (2 jam processing); instance terminate setelah selesai; tidak ada idle EC2 running 22 jam/hari menunggu file

---

**Soal 5** *(Secure Architectures)*

Sebuah perusahaan memiliki **Lambda functions** yang mengakses berbagai AWS services: DynamoDB, S3, SSM Parameter Store, dan memanggil third-party API. Setelah security audit, tim menemukan Lambda function menggunakan satu IAM role dengan `AdministratorAccess` policy.

Apa yang harus dilakukan?

- A. Ganti ke Cognito Identity Pool untuk Lambda authentication
- B. Aktifkan Lambda Reserved Concurrency untuk membatasi blast radius
- C. Enkripsi environment variables Lambda dengan KMS
- D. **Terapkan least privilege — buat IAM role terpisah per Lambda dengan only permissions yang diperlukan**: DynamoDB function hanya dapat `dynamodb:GetItem`, `dynamodb:PutItem` pada table ARN spesifik; S3 function hanya `s3:GetObject` pada bucket spesifik; ikuti prinsip minimum necessary permissions

---

**Soal 6** *(Resilient Architectures)*

Sebuah perusahaan memiliki **SQS queue** yang menerima orders dari aplikasi. Lambda function memproses setiap order — query inventory database, charge payment, kirim konfirmasi email. Kadang Lambda gagal karena inventory database timeout. Orders yang gagal di-retry terus-menerus dan menyebabkan legitimate orders di belakangnya tidak diproses.

Masalah ini disebut apa dan solusinya apa?

- A. Race condition — gunakan SQS FIFO untuk ordering
- B. Backpressure — scale up Lambda concurrency
- C. **Poison pill message — tambahkan SQS Dead Letter Queue (DLQ)** — messages yang gagal setelah N kali retry (maxReceiveCount) otomatis dipindahkan ke DLQ; normal messages di main queue bisa terus diproses; tim bisa investigate dan reprocess messages di DLQ setelah root cause diperbaiki
- D. Thundering herd — tambahkan exponential backoff di Lambda code

---

**Soal 7** *(High-Performing Architectures)*

Sebuah perusahaan ingin **migrate on-premise Oracle database** ke AWS dengan tujuan mengurangi licensing cost Oracle yang sangat mahal. Mereka ingin pindah ke open-source compatible database. Aplikasi mereka cukup standard — tidak menggunakan Oracle-specific features yang exotic.

Jalur migrasi mana yang direkomendasikan AWS?

- A. Lift-and-shift Oracle ke EC2 — sama persis, hanya pindah ke cloud
- B. Migrasi ke Amazon RDS for Oracle — tetap Oracle tapi managed
- C. **Migrasi ke Aurora PostgreSQL menggunakan AWS SCT + DMS** — AWS Schema Conversion Tool (SCT) convert Oracle schema dan stored procedures ke PostgreSQL-compatible; DMS untuk data migration dengan CDC; Aurora PostgreSQL performa tinggi, biaya jauh lebih murah dari Oracle licensing; PostgreSQL open-source compatible
- D. Migrasi ke DynamoDB — no-SQL, tidak perlu licensing

---

**Soal 8** *(Cost-Optimized Architectures)*

Sebuah perusahaan menggunakan **Amazon CloudFront** untuk serve website mereka. Mereka baru sadar bahwa **CloudFront access logs** yang dikirim ke S3 menghasilkan jutaan file log kecil per hari, menyebabkan biaya S3 PUT requests dan storage yang tinggi.

Cara mengatasi ini?

- A. Matikan CloudFront access logs sepenuhnya
- B. Kurangi sampling rate logs ke 1%
- C. Pindahkan log bucket ke S3 One Zone-IA untuk storage lebih murah
- D. **Aktifkan CloudFront standard logging dengan agregasi, atau gunakan CloudFront real-time logs ke Kinesis** — untuk analisis, gunakan Athena query terhadap log S3 (tidak perlu download); atur S3 Lifecycle untuk hapus logs lama; atau switch ke CloudFront real-time logs yang hanya kirim sample yang diperlukan

---

**Soal 9** *(Secure + Resilient)*

Sebuah startup membangun **aplikasi SaaS multi-tenant**. Setiap customer data harus **terisolasi** — customer A tidak boleh bisa akses data customer B. Data tersimpan di DynamoDB dengan `tenant_id` sebagai partition key. API diakses via API Gateway + Lambda.

Arsitektur isolasi mana yang paling tepat dan scalable?

- A. Deploy DynamoDB table terpisah per tenant
- B. **Gunakan Cognito User Pools per tenant + IAM conditions dengan `dynamodb:LeadingKeys`** — Cognito authenticate user dan issue JWT dengan `tenant_id` claim; Lambda verify token dan extract `tenant_id`; IAM policy pada role Lambda menggunakan condition `dynamodb:LeadingKeys: "${cognito-identity.amazonaws.com:sub}"` atau custom authorizer yang enforce tenant isolation dalam query
- C. Enkripsi setiap tenant data dengan KMS key yang berbeda
- D. Gunakan VPC per tenant untuk network isolation

---

**Soal 10** *(All Domains)*

Sebuah perusahaan ingin **arsitektur serverless yang resilient, aman, dan cost-efficient** untuk aplikasi REST API mereka. Mereka mengevaluasi opsi-opsi berikut:

**Requirement:**
- Tidak ada server yang harus di-manage
- Scale otomatis dari 0 hingga ribuan requests/detik
- Data tersimpan secara durable
- Biaya hanya saat digunakan
- Proteksi dari SQL injection dan DDoS

Arsitektur mana yang memenuhi semua requirement?

- A. EC2 Auto Scaling + RDS Multi-AZ + WAF + Shield
- B. ECS Fargate + Aurora Serverless + WAF
- C. **API Gateway + Lambda + DynamoDB + WAF** — API Gateway (managed, scale otomatis); Lambda (serverless, bayar per invocation); DynamoDB (serverless, 99.999% durability, scale otomatis); WAF di depan API Gateway (proteksi SQLi, XSS, DDoS rate limiting); semua komponen serverless, tidak ada server yang di-manage, biaya hanya saat ada traffic
- D. Lambda + S3 + CloudFront + Cognito

---

> Set 25 selesai — ini adalah set terakhir dari 250 soal latihan SAA-C03.
> Buka `jawaban.md` untuk review semua jawaban.

---

**Ringkasan 250 Soal:**
| Domain | Set | Soal |
|--------|-----|------|
| Secure Architectures (30%) | 01–07 | 75 soal |
| Resilient Architectures (26%) | 08–13 | 60 soal (65 target, 5 sisa untuk Mixed) |
| High-Performing (24%) | 14–19 | 60 soal |
| Cost-Optimized (20%) | 20–24 | 50 soal |
| Mixed | 25 | 10 soal |
| **Total** | **25 set** | **255 soal** |
