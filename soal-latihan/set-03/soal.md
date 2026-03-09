# Soal Latihan — Set 03
## Domain: Secure Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah perusahaan menyimpan jutaan file di S3 yang berisi data pelanggan, termasuk nomor kartu kredit dan informasi identitas pribadi (PII). Tim security ingin tahu **bucket mana saja yang mengandung data sensitif** tersebut secara otomatis.

Service AWS mana yang paling tepat untuk kebutuhan ini?

- A. Amazon Inspector
- B. Amazon Macie
- C. AWS Config
- D. Amazon GuardDuty

---

**Soal 2**

Sebuah perusahaan memiliki VPC dengan beberapa subnet. Mereka ingin menerapkan **stateless packet filtering** di level subnet untuk block traffic dari IP address tertentu, tanpa memengaruhi Security Groups yang sudah ada.

Solusi yang paling tepat adalah?

- A. Tambahkan inbound rule deny di Security Group
- B. Gunakan Network Access Control List (NACL) dengan deny rule untuk IP tersebut
- C. Deploy AWS Network Firewall di VPC
- D. Aktifkan VPC Flow Logs dan block secara manual

---

**Soal 3**

Sebuah aplikasi web menggunakan HTTPS. Certificate yang digunakan saat ini dikelola secara manual oleh tim ops dan sering expired karena lupa direnew. Mereka ingin solusi yang **auto-renew certificate** dan terintegrasi langsung dengan ALB.

Solusi yang paling tepat adalah?

- A. Beli SSL certificate dari third-party CA dan upload ke IAM
- B. Gunakan AWS Certificate Manager (ACM) untuk provision dan deploy certificate ke ALB
- C. Gunakan self-signed certificate yang di-upload ke S3
- D. Aktifkan HTTPS di EC2 instance langsung dengan Let's Encrypt

---

**Soal 4**

Sebuah perusahaan memiliki **akun AWS utama (account A)** dan ingin memberikan akses ke tim developer di **account B** untuk bisa deploy ke S3 bucket di account A, tanpa membuat IAM user baru di account A.

Solusi yang paling tepat adalah?

- A. Buat IAM user di account A dan bagikan credentials ke developer di account B
- B. Buat IAM Role di account A dengan trust policy yang allow account B, lalu developer di account B assume role tersebut
- C. Gunakan VPC Peering antara account A dan B
- D. Buat S3 bucket policy yang allow semua access dari account B

---

**Soal 5**

Sebuah perusahaan punya tim junior developer yang memiliki IAM permissions cukup luas. Security team khawatir developer bisa membuat IAM roles dengan permissions melebihi yang mereka miliki sendiri (**privilege escalation**).

Mekanisme IAM mana yang paling tepat untuk membatasi maksimum permissions yang bisa diberikan oleh developer tersebut?

- A. Attach SCP yang restrict IAM actions di level Organizations
- B. Gunakan IAM Permission Boundaries pada IAM roles yang dibuat oleh developer
- C. Aktifkan MFA untuk semua developer
- D. Gunakan IAM Access Analyzer untuk detect policy perubahan

---

**Soal 6**

Sebuah perusahaan baru saja memindahkan data ke S3. Security team menemukan bahwa beberapa bucket dapat diakses publik karena misconfiguration policy. Mereka ingin **mencegah siapapun membuat bucket atau object publik** di seluruh akun, sekarang dan di masa depan.

Solusi paling tepat dengan satu konfigurasi yang berlaku untuk seluruh akun adalah?

- A. Review dan perbaiki setiap bucket policy satu per satu
- B. Aktifkan S3 Block Public Access di level akun
- C. Gunakan SCP yang deny `s3:PutBucketPolicy`
- D. Aktifkan AWS Config rule `s3-bucket-public-read-prohibited`

---

**Soal 7**

Sebuah perusahaan ingin melakukan **inspeksi mendalam (deep packet inspection)** terhadap semua traffic yang masuk dan keluar VPC mereka, termasuk kemampuan untuk memblokir domain tertentu dan detect intrusion berdasarkan Suricata rules.

Solusi yang paling tepat adalah?

- A. Gunakan Security Groups dengan outbound rules yang ketat
- B. Deploy AWS Network Firewall dengan stateful rule groups
- C. Gunakan NACL dengan custom rules
- D. Aktifkan GuardDuty untuk semua traffic VPC

---

**Soal 8**

Seorang developer memiliki IAM policy berikut yang attached ke user-nya:

```json
{
  "Effect": "Allow",
  "Action": "s3:*",
  "Resource": "*"
}
```

Namun ada juga SCP di level Organizations yang attached ke account-nya:

```json
{
  "Effect": "Deny",
  "Action": "s3:DeleteObject",
  "Resource": "*"
}
```

Apa yang terjadi ketika developer tersebut mencoba menghapus object di S3?

- A. Berhasil, karena IAM policy Allow lebih spesifik dari SCP
- B. Berhasil, karena IAM policy Allow `s3:*` mencakup semua S3 actions
- C. Gagal, karena SCP Deny mengoverride IAM policy Allow
- D. Tergantung pada bucket policy yang ada di bucket tersebut

---

**Soal 9**

Sebuah perusahaan memiliki Lambda function yang perlu diakses oleh aplikasi di akun AWS yang berbeda. Mereka ingin mengizinkan invocation dari akun lain **tanpa perlu membuat IAM role** di akun pemanggil.

Solusi yang paling tepat adalah?

- A. Buat IAM user di akun Lambda dan bagikan credentials ke akun lain
- B. Tambahkan resource-based policy pada Lambda function yang allow invocation dari akun tertentu
- C. Gunakan VPC Peering dan akses Lambda via private IP
- D. Deploy Lambda di kedua akun dengan kode yang sama

---

**Soal 10**

Sebuah perusahaan ingin semua traffic HTTP ke aplikasi web mereka **otomatis di-redirect ke HTTPS**. Aplikasi berjalan di EC2 instances di belakang Application Load Balancer.

Cara yang paling tepat dan efisien untuk mengimplementasikan ini adalah?

- A. Tambahkan redirect logic di setiap EC2 application code
- B. Buat ALB listener rule di port 80 yang redirect ke HTTPS (port 443)
- C. Gunakan Route 53 untuk redirect HTTP ke HTTPS
- D. Aktifkan HSTS di Security Group

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
