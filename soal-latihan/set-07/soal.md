# Soal Latihan — Set 07
## Domain: Secure Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah perusahaan ingin menerbitkan **TLS certificate untuk domain internal** mereka (bukan domain publik) agar digunakan oleh microservices yang berkomunikasi satu sama lain di dalam AWS. Mereka ingin kontrol penuh atas certificate authority-nya sendiri.

Solusi yang paling tepat adalah?

- A. Gunakan AWS Certificate Manager (ACM) public certificate
- B. Buat Private Certificate Authority menggunakan **AWS Private CA (ACM PCA)**
- C. Gunakan self-signed certificate di setiap service
- D. Beli certificate dari third-party CA seperti DigiCert

---

**Soal 2**

Sebuah perusahaan ingin memberikan akses **zero-trust** ke internal aplikasi web yang berjalan di AWS, tanpa harus setup VPN. Karyawan harus bisa akses dari device manapun setelah melewati identity verification, dan akses dikontrol per-aplikasi berdasarkan identity dan device posture.

Solusi AWS mana yang paling tepat?

- A. Setup AWS Client VPN untuk semua karyawan
- B. Gunakan AWS Verified Access — zero-trust network access tanpa VPN, akses dikontrol berdasarkan identity dan device signals
- C. Buat ALB dengan Cognito authentication
- D. Gunakan AWS Direct Connect untuk akses yang aman

---

**Soal 3**

Sebuah aplikasi web single-page (SPA) yang di-host di `app.example.com` perlu melakukan **AJAX request ke API** di `api.example.com`. Browser memblokir request tersebut karena cross-origin policy.

Konfigurasi AWS mana yang perlu dilakukan agar request berhasil?

- A. Aktifkan HTTPS di API Gateway
- B. Tambahkan **CORS (Cross-Origin Resource Sharing)** configuration di API Gateway atau S3 — allow origin `app.example.com`
- C. Gunakan CloudFront di depan API untuk menyamakan domain
- D. Pindahkan SPA ke domain yang sama dengan API

---

**Soal 4**

Sebuah perusahaan menjalankan Microsoft Active Directory on-premise. Mereka ingin aplikasi AWS bisa **autentikasi menggunakan Active Directory yang sudah ada** tanpa harus migrate atau replicate seluruh AD ke cloud.

Solusi yang paling tepat adalah?

- A. Buat AWS Managed Microsoft AD baru di AWS dan sync dengan on-premise AD
- B. Gunakan **AD Connector** — proxy yang meneruskan authentication request ke on-premise AD
- C. Buat IAM users untuk setiap AD user dan sync password secara manual
- D. Gunakan Cognito User Pool dengan LDAP connector

---

**Soal 5**

Sebuah tim security melakukan review dan menemukan bahwa sebuah **IAM role di-assume oleh role lain, yang kemudian di-assume oleh role lain lagi** (role chaining). Mereka khawatir dengan implikasi security dari pola ini.

Pernyataan mana yang benar tentang IAM role chaining?

- A. Role chaining tidak dibatasi — bisa chain role sebanyak apapun dengan duration session normal
- B. Saat role chaining, **maximum session duration dibatasi menjadi 1 jam**, tidak bisa di-extend seperti single role assumption
- C. Role chaining otomatis di-block oleh AWS sebagai security measure
- D. Role chaining hanya bisa dilakukan oleh root account

---

**Soal 6**

Sebuah perusahaan menyimpan **konten premium** (video, dokumen) di S3 dan di-serve via CloudFront. Mereka ingin memastikan bahwa konten tersebut **hanya bisa diakses oleh user yang sudah login dan berlangganan**, bukan oleh siapapun yang punya URL.

Solusi yang paling tepat adalah?

- A. Set bucket S3 menjadi public agar CloudFront bisa serve konten
- B. Gunakan **CloudFront Signed URLs atau Signed Cookies** — hanya user dengan valid signed token yang bisa akses konten
- C. Gunakan S3 pre-signed URL langsung tanpa CloudFront
- D. Tambahkan IAM authentication ke setiap CloudFront request

---

**Soal 7**

Sebuah perusahaan memiliki arsitektur: EC2 di private subnet → NAT Gateway → internet. Mereka ingin **semua service di AWS** yang mereka gunakan (S3, DynamoDB, KMS, SSM) diakses **tanpa melalui NAT Gateway**, untuk mengurangi cost dan meningkatkan security.

Pendekatan mana yang paling tepat?

- A. Pindahkan semua EC2 ke public subnet
- B. Gunakan kombinasi **Gateway Endpoints** (untuk S3 dan DynamoDB) dan **Interface Endpoints** (untuk KMS, SSM, dan services lainnya)
- C. Gunakan satu Interface Endpoint untuk semua AWS services
- D. Gunakan AWS Direct Connect sebagai pengganti NAT Gateway

---

**Soal 8**

Sebuah service AWS (misalnya ECS) perlu **permission untuk manage resources AWS** atas nama customer — misalnya ECS butuh akses ke ECR untuk pull images, dan ke CloudWatch untuk push logs.

Mekanisme IAM mana yang AWS gunakan untuk memberikan permissions ini secara otomatis?

- A. AWS Managed Policy yang di-attach ke account secara default
- B. **Service-Linked Role** — IAM role yang di-create otomatis oleh AWS service dengan predefined permissions yang diperlukan service tersebut
- C. Instance profile yang di-attach ke setiap container
- D. Root account credentials yang di-share ke AWS services

---

**Soal 9**

Sebuah perusahaan memiliki **puluhan VPCs** di berbagai regions dan ingin menghubungkan semuanya. Dari perspektif security, mereka khawatir dengan **transitive routing** — mereka tidak ingin VPC-A bisa reach VPC-C hanya karena keduanya terhubung ke VPC-B.

Solusi mana yang relevan untuk dipertimbangkan dari perspektif ini?

- A. VPC Peering — by design tidak support transitive routing, setiap pair VPC harus punya peering connection sendiri
- B. Transit Gateway — support transitive routing, tapi bisa dikontrol dengan route tables di TGW
- C. VPC Peering tidak scalable untuk puluhan VPCs, Transit Gateway lebih tepat dengan route table control
- D. Semua jawaban di atas relevan untuk dipertimbangkan bersama

---

**Soal 10**

Sebuah Solutions Architect diminta mendesain security architecture untuk aplikasi web yang menerima traffic dari internet. Requirements:
- Proteksi dari DDoS volumetric attack
- Block SQL injection dan XSS
- Log semua DNS queries
- Detect anomalous behavior dari dalam VPC

Kombinasi services mana yang memenuhi **semua empat** requirements tersebut?

- A. Shield Standard + WAF + VPC Flow Logs + GuardDuty
- B. Shield Standard + WAF + Route 53 Resolver Query Logs + GuardDuty
- C. Shield Advanced + WAF + CloudTrail + Inspector
- D. WAF + Network Firewall + VPC Flow Logs + Macie

---

**Soal 11**

Sebuah perusahaan ingin memastikan bahwa **semua API calls ke AWS** dari akun mereka di-log, termasuk calls yang dilakukan oleh AWS services atas nama mereka (misalnya Auto Scaling yang launch EC2). Mereka juga ingin log ini **tidak bisa dihapus** oleh siapapun di akun tersebut.

Kombinasi konfigurasi CloudTrail mana yang paling tepat?

- A. Enable CloudTrail dan simpan log ke S3 bucket di akun yang sama
- B. Enable CloudTrail organization trail, kirim log ke S3 bucket di **akun terpisah (Log Archive account)**, aktifkan **S3 Object Lock Compliance mode** pada bucket tersebut
- C. Enable CloudTrail dan enkripsi log dengan KMS
- D. Enable CloudTrail dan aktifkan log file validation saja

---

**Soal 12**

Sebuah perusahaan menjalankan aplikasi yang memproses data keuangan. Mereka menggunakan **AWS KMS untuk enkripsi**, dan security auditor meminta bukti bahwa **semua penggunaan KMS key** (encrypt, decrypt, siapa yang pakai, kapan) bisa di-audit.

Cara paling tepat untuk memenuhi ini adalah?

- A. Aktifkan KMS key rotation — ini secara otomatis log semua key usage
- B. Gunakan AWS CloudTrail — semua KMS API calls (Encrypt, Decrypt, GenerateDataKey, dll) otomatis di-log di CloudTrail
- C. Aktifkan KMS key policy logging di CloudWatch
- D. Gunakan AWS Config untuk track KMS key usage

---

**Soal 13**

Sebuah aplikasi Lambda di-trigger oleh **S3 event** setiap kali ada file baru di-upload. Lambda memproses file tersebut dan menyimpan hasilnya ke DynamoDB. Security team ingin memastikan bahwa Lambda **hanya bisa akses bucket sumber dan tabel DynamoDB tertentu** — tidak boleh akses resource lain.

IAM configuration yang paling tepat untuk Lambda execution role adalah?

- A. Attach `AmazonS3FullAccess` dan `AmazonDynamoDBFullAccess` managed policies
- B. Buat custom policy dengan `s3:GetObject` untuk ARN bucket spesifik, dan `dynamodb:PutItem` untuk ARN tabel spesifik saja
- C. Attach `AdministratorAccess` agar Lambda bisa akses apapun yang diperlukan
- D. Gunakan Lambda environment variables untuk store credentials S3 dan DynamoDB

---

**Soal 14**

Sebuah perusahaan punya tim developer yang **sering lupa delete Security Groups** yang sudah tidak dipakai. Security Groups yang orphaned ini menumpuk dan menyulitkan audit. Mereka ingin sistem yang **otomatis detect** Security Groups tanpa attachment manapun.

Solusi paling tepat dengan operational overhead rendah adalah?

- A. Buat script manual yang dijalankan tim security setiap minggu
- B. Gunakan AWS Config managed rule **`ec2-security-group-attached-to-eni`** yang detect Security Groups yang tidak ter-attach ke ENI apapun
- C. Gunakan AWS Trusted Advisor untuk check unused Security Groups
- D. Gunakan GuardDuty untuk detect orphaned Security Groups

---

**Soal 15**

Sebuah perusahaan sedang diaudit dan auditor menemukan bahwa beberapa **IAM users masih menggunakan access keys yang sudah lebih dari 90 hari tidak di-rotate**. Security policy perusahaan mewajibkan rotasi setiap 90 hari.

Solusi mana yang paling tepat untuk **enforce dan monitor** kebijakan ini secara berkelanjutan?

- A. Kirim email reminder ke semua developer setiap 3 bulan
- B. Gunakan AWS Config rule **`access-keys-rotated`** dengan parameter `maxAccessKeyAge: 90` — flag semua access keys yang melebihi batas usia
- C. Gunakan IAM Access Analyzer untuk detect access keys yang lama
- D. Aktifkan MFA untuk semua IAM users sebagai pengganti access key rotation

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
