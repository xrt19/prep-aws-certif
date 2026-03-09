# Soal Latihan — Set 05
## Domain: Secure Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah tim security ingin menganalisis **siapa yang melakukan koneksi ke EC2 instance mereka, dari IP mana, port berapa, dan apakah koneksi tersebut diterima atau ditolak** — tanpa harus login ke instance.

Service AWS mana yang menyediakan data tersebut?

- A. AWS CloudTrail
- B. VPC Flow Logs
- C. AWS Config
- D. Amazon Inspector

---

**Soal 2**

Sebuah perusahaan menyimpan data di Amazon S3 dan ingin memberikan akses ke **masing-masing departemen** (Finance, HR, Engineering) hanya ke prefix folder mereka masing-masing. Setiap departemen bisa punya access policy sendiri tanpa mengubah bucket policy utama.

Fitur S3 mana yang paling tepat untuk kebutuhan ini?

- A. S3 Bucket Policy dengan kondisi per-prefix
- B. S3 Access Points — buat satu access point per departemen dengan policy masing-masing
- C. S3 ACL per folder
- D. IAM policy per departemen dengan resource ARN prefix

---

**Soal 3**

Sebuah perusahaan menjalankan aplikasi di **Amazon EKS (Kubernetes)**. Pods di dalam cluster butuh akses ke DynamoDB dan S3. Tim security tidak ingin menggunakan long-term credentials atau menyimpan access key di dalam pods.

Solusi yang paling tepat adalah?

- A. Buat IAM user dan simpan credentials sebagai Kubernetes Secret
- B. Gunakan IAM Roles for Service Accounts (IRSA) — associate IAM role ke Kubernetes service account
- C. Attach IAM Role ke EC2 node group, semua pods di node akan inherit permissions
- D. Gunakan AWS Secrets Manager dan inject credentials ke pods via init container

---

**Soal 4**

Sebuah perusahaan butuh mengenkripsi data berukuran **beberapa GB** menggunakan KMS. Mereka tahu bahwa KMS API memiliki limit ukuran data yang bisa dienkripsi langsung.

Teknik enkripsi mana yang harus digunakan?

- A. Split data menjadi potongan kecil dan enkripsi satu per satu via KMS API
- B. Gunakan envelope encryption — KMS generate Data Encryption Key (DEK), enkripsi data dengan DEK secara lokal, simpan DEK yang sudah di-enkripsi bersama data
- C. Upload data ke S3 lalu minta KMS untuk enkripsi langsung dari S3
- D. Gunakan CloudHSM karena KMS tidak bisa handle data besar

---

**Soal 5**

Sebuah perusahaan menggunakan AWS Organizations dengan ratusan member accounts. Mereka ingin tim security terpusat bisa **mengelola GuardDuty di semua accounts** — termasuk melihat semua findings dari satu tempat — tanpa harus login ke setiap account.

Fitur mana yang memungkinkan ini?

- A. Buat IAM role di setiap account dan assume role dari management account
- B. Gunakan GuardDuty delegated administrator — designate satu account sebagai admin untuk manage GuardDuty di seluruh Organizations
- C. Aktifkan AWS Security Hub di management account untuk aggregate semua GuardDuty findings
- D. Gunakan AWS Config aggregator untuk collect GuardDuty findings dari semua accounts

---

**Soal 6**

Sebuah aplikasi di EC2 menggunakan SSM Parameter Store untuk menyimpan konfigurasi. Ada dua jenis parameter: konfigurasi umum (tidak sensitif) dan database password (sensitif). Tim ingin **mengenkripsi parameter sensitif** menggunakan KMS, tapi tetap bisa menyimpan banyak parameter tanpa biaya tambahan.

Konfigurasi Parameter Store yang tepat adalah?

- A. Gunakan Standard tier untuk semua parameter, enkripsi dengan SSE
- B. Gunakan SecureString type untuk parameter sensitif (database password) — otomatis dienkripsi dengan KMS, Standard tier tidak ada biaya per parameter
- C. Gunakan Advanced tier untuk semua parameter agar bisa dienkripsi
- D. Simpan semua di Secrets Manager karena Parameter Store tidak support enkripsi

---

**Soal 7**

Sebuah perusahaan multinasional menggunakan AWS di **banyak regions**. Mereka ingin CloudTrail merekam semua API activity di **semua regions sekaligus** dan menyimpan log ke satu S3 bucket terpusat, tanpa harus enable CloudTrail satu per satu di setiap region.

Solusi yang paling tepat adalah?

- A. Buat CloudTrail trail di setiap region secara manual
- B. Buat satu CloudTrail trail dengan opsi **"Apply trail to all regions"** diaktifkan
- C. Gunakan AWS Config multi-region aggregator sebagai pengganti CloudTrail
- D. Aktifkan CloudWatch Logs di setiap region dan aggregate ke pusat

---

**Soal 8**

Sebuah perusahaan perlu menunjukkan kepada auditor bahwa mereka comply dengan **standar ISO 27001 dan SOC 2**. Auditor meminta **laporan compliance resmi dari AWS** yang menjelaskan bagaimana AWS memenuhi standar-standar tersebut.

Di mana perusahaan bisa mendapatkan dokumen resmi tersebut?

- A. AWS CloudTrail — export semua activity logs sebagai bukti compliance
- B. AWS Artifact — download AWS compliance reports dan agreements
- C. AWS Config — generate compliance report dari semua rules
- D. AWS Security Hub — export compliance findings sebagai laporan

---

**Soal 9**

Sebuah perusahaan memiliki **S3 bucket yang berisi data sensitif**. Mereka khawatir bahwa IAM user atau role di dalam akun bisa secara tidak sengaja atau sengaja membuat bucket tersebut **publicly accessible**. Mereka ingin mencegah ini bahkan jika ada explicit `s3:PutBucketAcl` Allow di IAM policy.

Kontrol mana yang paling tepat untuk ini?

- A. Gunakan SCP yang deny `s3:PutBucketAcl` di level Organizations
- B. Aktifkan S3 Block Public Access di level **bucket** — mengoverride semua ACL dan policy yang mencoba membuat objek/bucket menjadi publik
- C. Hapus semua IAM policy yang mengandung `s3:PutBucketAcl`
- D. Gunakan S3 Object Lock untuk mencegah perubahan ACL

---

**Soal 10**

Sebuah perusahaan menjalankan workload di AWS dan ingin memastikan bahwa semua **EFS (Elastic File System)** yang dibuat di akun mereka selalu dienkripsi at rest. Jika ada EFS yang dibuat tanpa enkripsi, harus langsung terdeteksi.

Solusi dengan operational overhead paling rendah adalah?

- A. Buat Lambda function yang cek semua EFS baru setiap 5 menit
- B. Gunakan AWS Config managed rule `efs-encrypted-check` untuk detect EFS tanpa enkripsi
- C. Aktifkan enkripsi default di EFS console sehingga semua EFS baru otomatis terenkripsi, dan gunakan Config rule sebagai detective control
- D. Buat SCP yang deny `elasticfilesystem:CreateFileSystem` jika tidak ada enkripsi

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
