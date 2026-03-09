# Soal Latihan — Set 02
## Domain: Secure Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah perusahaan ingin mengenkripsi data sensitif yang disimpan di EBS volumes. Mereka ingin menggunakan **kunci enkripsi mereka sendiri** (bukan AWS-managed key) dan bisa **rotate kunci tersebut setiap tahun** sesuai kebijakan internal.

Solusi mana yang paling tepat?

- A. Aktifkan EBS default encryption menggunakan AWS-managed key (aws/ebs)
- B. Buat Customer Managed Key (CMK) di AWS KMS, gunakan untuk EBS encryption, aktifkan automatic key rotation
- C. Enkripsi data di application layer sebelum ditulis ke EBS
- D. Gunakan AWS CloudHSM untuk enkripsi EBS

---

**Soal 2**

Sebuah perusahaan menjalankan workload di beberapa AWS accounts yang dikelola melalui AWS Organizations. Tim security ingin menerapkan kebijakan bahwa **semua EC2 instances harus diberi tag `Environment`** — jika tidak ada tag tersebut, instance tidak boleh dibuat.

Mekanisme mana yang paling tepat?

- A. Gunakan IAM policy `Condition` dengan `aws:RequestedRegion`
- B. Buat SCP yang deny `ec2:RunInstances` jika tag `Environment` tidak ada dalam request
- C. Gunakan AWS Config rule untuk detect instance tanpa tag dan terminate otomatis
- D. Buat Lambda function yang scan semua instances setiap jam dan hapus yang tidak ber-tag

---

**Soal 3**

Sebuah aplikasi mobile mengizinkan user untuk upload foto langsung ke S3. Developer ingin agar **setiap user hanya bisa upload dan read foto miliknya sendiri**, bukan milik user lain. Identitas user dikelola di Amazon Cognito.

Solusi yang paling tepat adalah?

- A. Buat IAM user untuk setiap user aplikasi
- B. Gunakan Cognito Identity Pool dengan IAM role yang menggunakan `${cognito-identity.amazonaws.com:sub}` di S3 bucket policy
- C. Buat Lambda authorizer di API Gateway untuk validasi setiap request
- D. Simpan semua foto di folder yang sama dan gunakan S3 ACL per object

---

**Soal 4**

Sebuah perusahaan menggunakan Amazon RDS MySQL. Security auditor menemukan bahwa beberapa query ke database mengandung pattern mencurigakan yang bisa jadi SQL injection. Mereka butuh solusi untuk **detect dan alert** aktivitas database yang mencurigakan tanpa mengubah aplikasi.

Solusi mana yang paling tepat?

- A. Aktifkan RDS Enhanced Monitoring
- B. Aktifkan AWS Database Activity Streams pada RDS instance
- C. Aktifkan RDS Performance Insights
- D. Gunakan AWS WAF di depan RDS

---

**Soal 5**

Sebuah perusahaan memiliki aplikasi yang berjalan di AWS. Tim security menerima alert bahwa ada EC2 instance yang sedang melakukan **port scanning terhadap instances lain** di VPC, dan berkomunikasi dengan **known malicious IP addresses**.

Service AWS mana yang mendeteksi dan men-generate alert tersebut?

- A. AWS Inspector
- B. AWS Config
- C. Amazon GuardDuty
- D. AWS Security Hub

---

**Soal 6**

Sebuah perusahaan ingin memastikan bahwa hanya request yang berasal dari **IP address kantor mereka (203.0.113.0/24)** yang bisa mengakses AWS Management Console untuk akun produksi.

Cara yang paling tepat untuk mengimplementasikan ini adalah?

- A. Tambahkan Security Group rule yang hanya izinkan IP kantor
- B. Buat IAM policy dengan `Condition: aws:SourceIp` dan attach ke semua IAM users
- C. Gunakan SCP di Organizations dengan kondisi `aws:SourceIp`
- D. Block IP di Network ACL VPC

---

**Soal 7**

Seorang Solutions Architect diminta untuk mendesain solusi dimana **EC2 instances di private subnet** bisa mengakses layanan AWS seperti SSM Parameter Store dan KMS tanpa harus melewati NAT Gateway (untuk alasan cost dan security).

Solusi yang paling tepat adalah?

- A. Pindahkan EC2 instances ke public subnet
- B. Buat Interface VPC Endpoints untuk SSM, KMS, dan services yang dibutuhkan
- C. Gunakan Transit Gateway untuk routing ke services AWS
- D. Deploy NAT Instance yang lebih murah sebagai pengganti NAT Gateway

---

**Soal 8**

Sebuah perusahaan baru saja onboard ke AWS dan ingin mendapatkan **visibility menyeluruh terhadap security posture** mereka — mulai dari findings di GuardDuty, Inspector, IAM Access Analyzer, hingga compliance checks dari Security Hub, semua dalam satu dashboard.

Service mana yang paling tepat untuk aggregasi ini?

- A. Amazon CloudWatch
- B. AWS Security Hub
- C. AWS Config
- D. Amazon Detective

---

**Soal 9**

Sebuah aplikasi Lambda butuh akses ke sebuah third-party API key yang disimpan di AWS Secrets Manager. Security team ingin memastikan bahwa Lambda **hanya bisa mengakses secret yang spesifik tersebut**, bukan secrets lainnya.

Konfigurasi IAM yang paling tepat adalah?

- A. Attach `SecretsManagerReadWrite` managed policy ke Lambda execution role
- B. Attach policy dengan `secretsmanager:GetSecretValue` hanya untuk ARN secret yang spesifik ke Lambda execution role
- C. Simpan API key di environment variable Lambda yang di-enkripsi dengan KMS
- D. Buat IAM user khusus untuk Lambda dan simpan access key-nya di Secrets Manager

---

**Soal 10**

Sebuah perusahaan ingin mengaudit **siapa saja yang punya akses ke resources AWS apa** di seluruh organisasi mereka, untuk memastikan tidak ada excessive permissions. Mereka ingin tool yang bisa **analyze access dan identify unused permissions** secara otomatis.

Service AWS mana yang paling tepat untuk kebutuhan ini?

- A. AWS CloudTrail
- B. AWS Config
- C. IAM Access Analyzer
- D. Amazon GuardDuty

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
