# Soal Latihan — Set 04
## Domain: Secure Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah perusahaan menjalankan aplikasi e-commerce di EC2 dan CloudFront. Mereka sering menjadi target serangan **DDoS volumetric** yang menghabiskan bandwidth dan membuat aplikasi tidak bisa diakses. Tim security ingin proteksi DDoS yang **otomatis tanpa konfigurasi manual** dan terintegrasi langsung dengan layanan AWS.

Solusi yang paling tepat adalah?

- A. Deploy AWS WAF di depan CloudFront dengan rate-based rules
- B. AWS Shield Standard — sudah aktif otomatis di semua akun AWS tanpa biaya tambahan
- C. AWS Shield Advanced dengan DDoS Response Team (DRT) support
- D. Gunakan Auto Scaling untuk absorb traffic DDoS

---

**Soal 2**

Sebuah tim DevOps menggunakan AWS Systems Manager Session Manager untuk remote access ke EC2 instances, menggantikan SSH. Security team ingin memastikan bahwa **semua sesi Session Manager di-log** ke S3 untuk keperluan audit.

Fitur Systems Manager mana yang digunakan untuk ini?

- A. AWS CloudTrail — akan otomatis log semua SSM API calls
- B. Konfigurasikan Session Manager preferences untuk stream session logs ke S3 dan/atau CloudWatch Logs
- C. Aktifkan VPC Flow Logs untuk capture traffic Session Manager
- D. Gunakan AWS Config untuk record semua sesi yang terjadi

---

**Soal 3**

Sebuah perusahaan ingin berbagi sebuah **AWS RAM (Resource Access Manager)** untuk share Transit Gateway ke beberapa AWS accounts dalam Organizations mereka, tanpa harus membuat Transit Gateway terpisah di setiap account.

Pernyataan mana yang benar tentang AWS RAM?

- A. RAM hanya bisa share resources antar accounts dalam VPC yang sama
- B. RAM memungkinkan share resources AWS (seperti Transit Gateway, Subnets, dll) ke accounts lain atau Organizations tanpa duplikasi resource
- C. RAM memerlukan VPC Peering sebagai prasyarat sebelum bisa share resources
- D. RAM otomatis copy resource ke setiap account yang menerima share

---

**Soal 4**

Sebuah aplikasi web memungkinkan user untuk **download file laporan** yang tersimpan di S3 private bucket. File ini bersifat personal — hanya boleh diakses oleh user yang bersangkutan, dan link download hanya boleh valid selama **15 menit**.

Solusi yang paling tepat adalah?

- A. Buat bucket public dan gunakan IAM policy untuk restrict access
- B. Generate S3 pre-signed URL dengan expiry 15 menit dari server-side
- C. Gunakan S3 Transfer Acceleration untuk akses yang lebih cepat
- D. Buat CloudFront distribution dengan signed cookies untuk semua user

---

**Soal 5**

Sebuah perusahaan memiliki EC2 instances yang menjalankan aplikasi web. Tim security ingin melakukan **vulnerability assessment** secara otomatis — mencari known CVEs, network exposure, dan software vulnerabilities pada instances tersebut, tanpa install agent.

Service AWS mana yang paling tepat?

- A. Amazon GuardDuty
- B. AWS Config
- C. Amazon Inspector
- D. AWS Security Hub

---

**Soal 6**

Sebuah perusahaan menggunakan AWS Organizations dengan puluhan member accounts. Tim security ingin menerapkan **AWS WAF rules yang sama** ke semua ALB di seluruh accounts secara terpusat, tanpa harus konfigurasi WAF satu per satu di setiap account.

Solusi yang paling tepat adalah?

- A. Buat AWS WAF WebACL di management account dan share via RAM
- B. Gunakan AWS Firewall Manager untuk deploy WAF policies secara terpusat ke seluruh accounts dalam Organizations
- C. Buat SCP yang require WAF di setiap ALB deployment
- D. Gunakan AWS Config rule untuk detect ALB tanpa WAF

---

**Soal 7**

Sebuah developer memiliki akses ke AWS KMS key dan ingin menggunakan key tersebut untuk encrypt data. Namun security admin ingin memastikan bahwa **developer tidak bisa delete atau disable key tersebut**, meskipun developer memiliki IAM policy dengan `kms:*`.

Mekanisme KMS mana yang mengontrol hal ini?

- A. S3 bucket policy pada key material
- B. KMS Key Policy — harus ada explicit Allow di key policy untuk setiap action
- C. SCP yang deny kms:DeleteKey untuk semua users
- D. IAM Permission Boundary yang restrict KMS actions

---

**Soal 8**

Sebuah aplikasi menggunakan Amazon Cognito untuk autentikasi user. Ada dua kebutuhan: (1) user bisa **register dan login** ke aplikasi, dan (2) setelah login, user mendapat **temporary AWS credentials** untuk akses langsung ke S3.

Kombinasi fitur Cognito yang tepat adalah?

- A. Cognito User Pool saja — sudah cukup untuk kedua kebutuhan
- B. Cognito Identity Pool saja — bisa handle login dan AWS credentials
- C. Cognito User Pool untuk autentikasi user, kemudian Cognito Identity Pool untuk exchange token jadi AWS credentials
- D. Cognito User Pool untuk autentikasi, IAM user per user untuk AWS credentials

---

**Soal 9**

Sebuah perusahaan ingin memastikan bahwa setiap kali ada **S3 bucket yang tidak memiliki server-side encryption**, sistem secara otomatis **mengaktifkan enkripsi** tanpa intervensi manual.

Solusi paling tepat dengan operational overhead terendah adalah?

- A. Jadwalkan Lambda function setiap malam untuk scan dan enable enkripsi
- B. Buat AWS Config rule `s3-bucket-server-side-encryption-enabled` dengan auto-remediation menggunakan SSM Automation
- C. Gunakan GuardDuty untuk detect bucket tanpa enkripsi
- D. Aktifkan S3 Block Public Access — ini juga mengaktifkan enkripsi otomatis

---

**Soal 10**

Sebuah perusahaan sedang mendesain arsitektur untuk aplikasi yang memproses data medis pasien (PHI — Protected Health Information). Mereka perlu memastikan bahwa **semua data at rest di RDS dienkripsi**, dan enkripsi harus menggunakan **KMS key yang dikontrol perusahaan** (bukan AWS-managed key).

Kapan enkripsi RDS harus dikonfigurasi?

- A. Enkripsi bisa diaktifkan kapan saja di RDS settings setelah database dibuat
- B. Enkripsi RDS hanya bisa diaktifkan **saat pembuatan DB instance** — tidak bisa diaktifkan pada instance yang sudah berjalan
- C. Enkripsi RDS otomatis aktif untuk semua instance tanpa konfigurasi
- D. Enkripsi RDS diaktifkan via S3 bucket policy pada snapshot

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
