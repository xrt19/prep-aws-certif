# Soal Latihan — Set 06
## Domain: Secure Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah perusahaan besar ingin membangun **multi-account AWS environment** dari nol dengan best practices: account vending (pembuatan account otomatis), guardrails terpusat, dan landing zone yang sudah pre-configured sesuai AWS Well-Architected Framework.

Service AWS mana yang paling tepat untuk menyiapkan ini?

- A. AWS Organizations — buat OU dan SCP secara manual
- B. AWS Control Tower — setup landing zone otomatis dengan guardrails dan account factory
- C. AWS Config — buat rules untuk enforce standards di semua accounts
- D. AWS Service Catalog — buat product catalog untuk provisioning accounts

---

**Soal 2**

Sebuah perusahaan menyimpan **arsip backup jangka panjang** di S3 Glacier. Mereka ingin memastikan data tersebut **tidak bisa dihapus selama 10 tahun** sesuai regulasi, bahkan oleh root account sekalipun.

Fitur mana yang paling tepat?

- A. S3 Object Lock Compliance mode pada bucket Glacier
- B. S3 Glacier Vault Lock — apply Vault Lock policy dengan `deny DeleteArchive` untuk vault tersebut
- C. S3 MFA Delete pada Glacier vault
- D. IAM policy deny `glacier:DeleteArchive` untuk semua users

---

**Soal 3**

Sebuah tim security sedang **menginvestigasi security incident** di mana ada akses tidak sah ke beberapa EC2 instances dan S3 buckets. Mereka butuh tool untuk **visualisasi dan analisis hubungan** antar resources, IAM entities, dan aktivitas yang terlibat dalam incident tersebut.

Service AWS mana yang paling tepat?

- A. AWS Security Hub — lihat semua findings yang terkait
- B. Amazon GuardDuty — cek threat findings di periode tersebut
- C. Amazon Detective — analisis dan visualisasi root cause dari security incident
- D. AWS CloudTrail — query logs untuk aktivitas di periode tersebut

---

**Soal 4**

Sebuah perusahaan memiliki IAM policy yang di-attach ke banyak IAM users dan roles. Ketika perlu update policy (misalnya tambah permission baru), mereka harus update satu per satu. Mereka ingin solusi di mana **satu update policy langsung berlaku ke semua entities** yang menggunakannya.

Pendekatan IAM yang paling tepat adalah?

- A. Gunakan inline policies — bisa di-edit di satu tempat
- B. Gunakan AWS Managed Policies — di-manage oleh AWS
- C. Gunakan Customer Managed Policies — buat satu policy, attach ke banyak entities, update sekali langsung berlaku ke semua
- D. Gunakan IAM Groups dan attach policy ke group

---

**Soal 5**

Sebuah developer menggunakan `aws sts assume-role` untuk assume sebuah IAM role yang memiliki `s3:*` permissions. Namun saat menjalankan operasi, developer hanya boleh melakukan `s3:GetObject` — tidak boleh lebih dari itu, meskipun role memiliki permissions yang lebih luas.

Mekanisme mana yang bisa membatasi permissions **saat session tanpa mengubah role policy**?

- A. Tambahkan Permission Boundary ke role tersebut
- B. Gunakan SCP di Organizations untuk restrict s3 actions
- C. Pass **session policy** saat memanggil `AssumeRole` — session policy membatasi permissions menjadi intersection antara role policy dan session policy
- D. Buat IAM user terpisah dengan hanya `s3:GetObject` permission

---

**Soal 6**

Sebuah perusahaan memiliki dua VPC: **VPC-A** (production) dan **VPC-B** (development). EC2 di VPC-B perlu akses ke database di VPC-A. Tim security ingin Security Group di VPC-A **hanya mengizinkan traffic dari EC2 instances di VPC-B** — bukan dari seluruh CIDR range VPC-B.

Cara paling tepat untuk mengimplementasikan ini adalah?

- A. Allow CIDR range VPC-B (misalnya 10.1.0.0/16) di inbound rule Security Group VPC-A
- B. Gunakan Security Group referencing — allow inbound dari **Security Group ID** spesifik yang dipakai EC2 di VPC-B
- C. Gunakan NACL untuk restrict traffic dari VPC-B
- D. Deploy WAF di VPC-A untuk filter traffic dari VPC-B

---

**Soal 7**

Sebuah aplikasi web mendapatkan serangan **credential stuffing** — attacker mencoba ribuan kombinasi username/password secara otomatis dari berbagai IP address. Tim security ingin mendeteksi dan **memblokir request yang melebihi threshold** secara otomatis.

Solusi paling tepat adalah?

- A. Aktifkan AWS Shield Advanced untuk block credential stuffing
- B. Tambahkan AWS WAF rate-based rule yang block IP jika request ke `/login` melebihi threshold tertentu per 5 menit
- C. Gunakan GuardDuty untuk detect dan block login attempts
- D. Scale up EC2 instances agar bisa handle traffic tambahan

---

**Soal 8**

Sebuah perusahaan menggunakan S3 untuk menyimpan data yang di-upload oleh third-party vendors via Cross-Origin requests. Setelah audit, ditemukan bahwa **object ownership** dari file yang di-upload vendor masih di-assign ke AWS account vendor tersebut, bukan ke account perusahaan — menyebabkan perusahaan tidak bisa manage object tersebut sepenuhnya.

Solusi yang paling tepat adalah?

- A. Minta vendor untuk transfer ownership setiap kali upload
- B. Aktifkan **S3 Object Ownership** dengan mode `Bucket owner enforced` — semua objects otomatis dimiliki bucket owner terlepas dari siapa yang upload
- C. Gunakan S3 bucket policy yang require vendor menggunakan `bucket-owner-full-control` ACL saat upload
- D. Buat IAM role untuk vendor dan assign ownership via role policy

---

**Soal 9**

Sebuah perusahaan ingin mendapatkan **rekomendasi keamanan otomatis** untuk akun AWS mereka, seperti: apakah root account MFA aktif, apakah ada security groups yang terlalu open (0.0.0.0/0), dan apakah IAM access keys sudah lama tidak di-rotate.

Service mana yang menyediakan rekomendasi ini?

- A. Amazon GuardDuty
- B. AWS Config
- C. AWS Trusted Advisor
- D. IAM Access Analyzer

---

**Soal 10**

Sebuah perusahaan menjalankan aplikasi multi-tier: web tier (public subnet), app tier (private subnet), dan database tier (private subnet). Mereka ingin memastikan bahwa **database hanya bisa diakses dari app tier**, dan **app tier hanya bisa diakses dari web tier** — tidak ada akses langsung dari luar.

Konfigurasi Security Group yang paling tepat adalah?

- A. Assign semua tier ke Security Group yang sama dan gunakan NACL untuk isolasi
- B. Buat Security Group terpisah per tier: DB-SG allow inbound hanya dari App-SG, App-SG allow inbound hanya dari Web-SG
- C. Gunakan CIDR ranges private subnet di setiap Security Group rule
- D. Buat satu Security Group dengan rules berdasarkan port saja (3306 untuk DB, 8080 untuk app)

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
