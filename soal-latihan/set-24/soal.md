# Soal Latihan — Set 24
## Domain: Cost-Optimized Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah perusahaan memiliki **S3 bucket** yang digunakan sebagai data lake. Dari analisis Cost Explorer, biaya S3 **data retrieval** sangat tinggi karena setiap query Athena men-scan seluruh dataset besar. Mereka tidak menggunakan Parquet — semua data tersimpan sebagai CSV flat.

Selain konversi ke Parquet, teknik S3 mana yang bisa mengurangi data yang di-scan Athena secara drastis dengan mengorganisir data dalam folder S3?

- A. S3 Object Lock — proteksi data dari deletion
- B. S3 Replication — copy data ke region lain untuk akses lebih cepat
- C. S3 Versioning — simpan versi lama file untuk recovery
- D. **S3 Partitioning** — organisir data dalam hierarki folder seperti `year=2024/month=03/day=10/`; Athena hanya scan folder yang relevan dengan query filter (partition pruning); query `WHERE year=2024 AND month=03` tidak perlu scan data tahun 2023 sama sekali

---

**Soal 2**

Sebuah perusahaan menggunakan **Amazon Kinesis Data Streams** untuk real-time data ingestion. Mereka membeli 100 shards untuk handle peak traffic, tapi di luar jam sibuk (70% waktu), utilisasi hanya 10–15 shards. Mereka membayar untuk 100 shards sepanjang waktu.

Cara paling cost-optimal untuk mengelola Kinesis capacity ini?

- A. Gunakan Enhanced Fan-Out untuk konsumsi lebih efisien
- B. Buat stream baru dengan kapasitas lebih kecil untuk off-peak
- C. **Aktifkan Kinesis Data Streams On-Demand mode** — otomatis scale capacity berdasarkan traffic aktual; tidak perlu provision shards manual; di-charge per GB data yang di-ingest dan di-retrieve; lebih hemat untuk traffic yang sangat variable
- D. Pindahkan ke Kinesis Data Firehose yang tidak butuh shard management

---

**Soal 3**

Sebuah perusahaan memiliki **fleet EC2 instances** untuk web application. Mereka sudah menggunakan Auto Scaling. Dari AWS Trusted Advisor, terlihat bahwa banyak instances memiliki **CPU utilization rata-rata di bawah 5%** selama 4 hari terakhir.

Langkah pertama yang harus dilakukan untuk mengurangi biaya?

- A. Langsung terminate semua instances yang CPU-nya rendah
- B. Pindahkan semua instances ke Spot
- C. Aktifkan Detailed Monitoring untuk data yang lebih akurat
- D. **Gunakan AWS Compute Optimizer untuk right-size instances** — analisis CPU, memory, network, dan disk metrics; rekomendasikan instance type yang lebih kecil sesuai actual utilization; hindari terminate terburu-buru karena CPU rendah bisa karena aplikasi memory-bound atau I/O-bound, bukan karena terlalu besar

---

**Soal 4**

Sebuah perusahaan menjalankan **Apache Spark jobs** nightly untuk data transformation. Jobs berjalan di Amazon EMR dan membutuhkan waktu 6 jam. Selama berjalan, ada fase tertentu di mana hanya beberapa tasks yang berjalan (akhir setiap stage Spark — "straggler" tasks), sementara sebagian besar cores idle menunggu.

Fitur EMR mana yang mengurangi waste dari idle cores selama fase ini?

- A. EMR Managed Scaling — scale nodes berdasarkan YARN metrics
- B. EMR Instance Fleet — diversifikasi instance types
- C. EMR Auto Termination — terminate cluster saat idle
- D. **EMR Managed Scaling dengan scale-down agresif** atau **Spark Dynamic Resource Allocation** — saat cores idle, Spark bisa return executors ke cluster dan reallocate ke jobs lain; EMR Managed Scaling bisa scale down jumlah nodes saat YARN utilization rendah secara otomatis

---

**Soal 5**

Sebuah perusahaan memiliki **Amazon API Gateway** dengan ribuan API endpoints. Mereka menemukan biaya API Gateway sangat tinggi padahal beberapa endpoints jarang digunakan. Mereka juga menggunakan REST API, bukan HTTP API.

Cara mengurangi biaya API Gateway?

- A. Aktifkan API Gateway caching untuk semua endpoints
- B. Kurangi jumlah stages (dev, staging, prod) menjadi satu
- C. **Migrasi dari REST API ke HTTP API** — HTTP API hingga 71% lebih murah dari REST API ($1.00 vs $3.50 per million requests); mendukung Lambda, HTTP backends, JWT authorization; jika fitur REST API yang advanced tidak dibutuhkan (WAF integration, X-Ray, private integration, usage plans), HTTP API lebih cost-optimal
- D. Gunakan API Gateway Edge-optimized untuk reduce latency dan cost

---

**Soal 6**

Sebuah perusahaan memiliki **data warehouse** di Amazon Redshift dan menjalankan query yang mengakses data **1–2 tahun terakhir** secara reguler, tapi data yang lebih lama (3–5 tahun) hanya perlu diakses beberapa kali per tahun untuk audit.

Cara paling cost-optimal untuk mengelola data ini di Redshift?

- A. Hapus data yang lebih dari 2 tahun dari Redshift
- B. Buat Redshift cluster kedua khusus untuk data historis
- C. Arsipkan data > 2 tahun ke S3 dan query via Athena saat diperlukan
- D. **Gunakan Redshift Spectrum** — data panas (1–2 tahun) tetap di Redshift untuk performance; data dingin (3–5 tahun) di-unload ke S3; query via Redshift Spectrum yang bisa join data Redshift dan S3 dalam satu query SQL; S3 storage jauh lebih murah dari Redshift storage

---

**Soal 7**

Sebuah perusahaan ingin **memonitor dan mengoptimalkan biaya AWS** secara berkelanjutan. Mereka ingin tool yang bisa memberikan rekomendasi actionable berdasarkan best practices untuk cost, performance, security, dan reliability — semuanya dalam satu dashboard.

Tool mana yang paling tepat?

- A. AWS Cost Explorer — analisis dan forecast biaya
- B. AWS Budgets — alert ketika biaya melebihi threshold
- C. Amazon CloudWatch — monitor infrastructure metrics
- D. **AWS Trusted Advisor** — automated best practice checks untuk 5 kategori: Cost Optimization, Performance, Security, Fault Tolerance, Service Limits; contoh cost checks: idle EC2, unused Elastic IPs, underutilized RDS, unassociated EIPs, low-utilization EBS volumes; tersedia di semua akun (sebagian checks), full checks dengan Business/Enterprise support

---

**Soal 8**

Sebuah startup menggunakan **Amazon SES (Simple Email Service)** untuk kirim marketing emails ke 500.000 subscribers setiap minggu. Biaya SES dirasa cukup tinggi. Mereka menggunakan SES di `us-east-1` dan application server mereka juga di `us-east-1`.

Cara paling mudah mengurangi biaya SES?

- A. Pindahkan ke third-party email provider yang lebih murah
- B. Kurangi frekuensi email menjadi bulanan
- C. Gunakan SES dedicated IP untuk mengurangi per-email cost
- D. **Kirim email dari EC2 instance menggunakan SES** — saat mengirim email dari EC2 di region yang sama dengan SES endpoint, tidak ada data transfer charge; SES juga memberikan **62.000 emails gratis per bulan** jika dikirim dari EC2; untuk volume yang lebih besar, SES adalah salah satu layanan email paling murah ($0.10 per 1000 emails)

---

**Soal 9**

Sebuah perusahaan memiliki **AWS account** yang sudah berjalan 2 tahun. Tim FinOps ingin melakukan **cost review** menyeluruh untuk identify semua resources yang tidak digunakan atau under-utilized: idle load balancers, unattached EBS volumes, unused Elastic IPs, low-utilization EC2.

Selain Trusted Advisor dan Compute Optimizer, tool mana yang memberikan **visibility terlengkap** tentang semua resources yang ada di account?

- A. AWS Config — track konfigurasi semua resources
- B. AWS CloudTrail — audit semua API calls dan resource creation
- C. **AWS Resource Explorer** — search dan discover semua resources di semua regions dalam satu tampilan; filter berdasarkan resource type, region, tag; identify resources tanpa tag (yang mungkin forgotten/abandoned); cocok untuk first-pass inventory before cost review
- D. Amazon Inspector — vulnerability assessment untuk EC2 dan containers

---

**Soal 10**

Sebuah perusahaan memiliki **workload machine learning training** yang berjalan di EC2 P3 instances (GPU). Training jobs berlangsung 12–24 jam dan dijalankan beberapa kali per minggu. Mereka sudah menggunakan Spot Instances tapi ingin mengoptimalkan lebih lanjut.

Selain Spot, strategi mana yang bisa mengurangi biaya ML training lebih jauh?

- A. Gunakan CloudFront untuk cache model artifacts
- B. Pindahkan ke SageMaker karena lebih murah dari EC2 langsung
- C. Simpan model checkpoints ke S3 Standard-IA untuk hemat storage
- D. **Gunakan EC2 Savings Plans untuk P3 instances yang digunakan secara reguler** — commit spend $/jam untuk EC2 compute; P3 instances yang digunakan beberapa kali per minggu secara reguler cocok untuk Savings Plans; dikombinasikan dengan Spot untuk jobs non-kritis, dan Savings Plans untuk production training jobs yang tidak bisa di-interrupt

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
