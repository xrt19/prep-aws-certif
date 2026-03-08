# Soal Latihan — Set 01
## Domain: Secure Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah perusahaan ingin memberikan akses ke AWS resources kepada karyawan yang sudah login ke corporate Active Directory, tanpa harus membuat IAM user baru untuk setiap karyawan.

Solusi yang paling tepat adalah?

- A. Buat IAM user untuk setiap karyawan dan sync password-nya dengan Active Directory
- B. Gunakan AWS IAM Identity Center (SSO) dengan Active Directory sebagai identity source
- C. Buat IAM Group dan assign policy ke seluruh karyawan
- D. Gunakan Amazon Cognito User Pool untuk authenticate karyawan

---

**Soal 2**

Seorang developer tidak sengaja meng-commit AWS access key ke public GitHub repository. Dalam hitungan menit, key tersebut sudah digunakan oleh pihak tidak dikenal.

Urutan tindakan yang paling tepat untuk dilakukan **pertama kali** adalah?

- A. Hapus repository GitHub-nya
- B. Rotate access key di IAM console
- C. Deactivate atau delete access key yang bocor segera
- D. Hubungi AWS Support

---

**Soal 3**

Sebuah aplikasi di EC2 butuh akses ke S3 bucket untuk read dan write objects. Security team melarang penggunaan long-term credentials (access key).

Cara paling tepat untuk memberikan akses tersebut adalah?

- A. Simpan access key di environment variable EC2
- B. Simpan access key di AWS Secrets Manager dan retrieve saat runtime
- C. Attach IAM Role dengan S3 permissions ke EC2 instance
- D. Hardcode access key di application code

---

**Soal 4**

Sebuah perusahaan memiliki S3 bucket yang menyimpan laporan keuangan sensitif. Mereka ingin memastikan bahwa data tersebut **tidak bisa dihapus** oleh siapapun, termasuk root account, selama periode retensi 7 tahun sesuai regulasi.

Fitur S3 mana yang paling tepat digunakan?

- A. S3 Versioning
- B. S3 Object Lock dengan Compliance mode
- C. S3 Object Lock dengan Governance mode
- D. S3 MFA Delete

---

**Soal 5**

Tim security ingin mendapatkan notifikasi setiap kali ada perubahan pada Security Group di seluruh akun AWS mereka.

Solusi paling tepat dengan operational overhead paling rendah adalah?

- A. Buat Lambda function yang polling EC2 API setiap 5 menit
- B. Gunakan AWS Config rule untuk detect perubahan Security Group dan trigger SNS notification
- C. Enable VPC Flow Logs dan analisis secara manual
- D. Buat CloudWatch dashboard untuk monitor Security Group

---

**Soal 6**

Sebuah aplikasi web menggunakan API Gateway dan Lambda. Tim security ingin melindungi API dari SQL injection dan cross-site scripting (XSS).

Solusi mana yang paling tepat?

- A. Aktifkan AWS Shield Advanced pada API Gateway
- B. Deploy AWS WAF di depan API Gateway dengan managed rule groups
- C. Gunakan Security Groups untuk block traffic berbahaya ke API Gateway
- D. Aktifkan API Gateway throttling

---

**Soal 7**

Sebuah perusahaan menggunakan AWS Organizations dengan banyak member accounts. Mereka ingin memastikan bahwa **tidak ada account manapun** yang bisa disable AWS CloudTrail, bahkan account dengan AdministratorAccess sekalipun.

Solusi yang paling tepat adalah?

- A. Attach IAM policy deny CloudTrail:DeleteTrail ke semua IAM users
- B. Gunakan AWS Config untuk detect dan auto-remediate jika CloudTrail dimatikan
- C. Buat SCP di Organizations yang deny CloudTrail disable actions dan attach ke root OU
- D. Enable CloudTrail di setiap account dan set alert di CloudWatch

---

**Soal 8**

Sebuah perusahaan menyimpan database credentials di beberapa EC2 instances dalam bentuk plaintext di config file. Mereka ingin migrasi ke solusi yang lebih aman dengan kemampuan **auto-rotate credentials** secara otomatis.

Solusi mana yang paling tepat?

- A. Enkripsi config file menggunakan KMS
- B. Pindahkan credentials ke SSM Parameter Store (SecureString)
- C. Pindahkan credentials ke AWS Secrets Manager
- D. Simpan credentials di S3 bucket dengan enkripsi SSE-KMS

---

**Soal 9**

Sebuah team security ingin tahu **siapa yang melakukan action apa dan kapan** terhadap resources AWS di akun mereka, termasuk siapa yang menghapus sebuah EC2 instance 3 hari lalu.

Service mana yang menyimpan informasi tersebut?

- A. Amazon CloudWatch Metrics
- B. AWS CloudTrail
- C. AWS Config
- D. Amazon GuardDuty

---

**Soal 10**

Sebuah perusahaan memiliki bucket S3 yang perlu diakses oleh EC2 instances di VPC mereka. Security team ingin memastikan traffic ke S3 **tidak pernah melewati public internet**.

Solusi yang paling tepat adalah?

- A. Gunakan NAT Gateway agar EC2 bisa akses S3
- B. Buat VPC Endpoint (Gateway type) untuk S3
- C. Buat VPC Endpoint (Interface type) untuk S3
- D. Gunakan Direct Connect untuk akses S3

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
