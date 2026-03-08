# Jawaban — Set 01
## Domain: Secure Architectures

---

## Soal 1 — Jawaban: B

**AWS IAM Identity Center (SSO) dengan Active Directory sebagai identity source**

**Kenapa B:**
IAM Identity Center (dulu AWS SSO) dirancang khusus untuk federated access — karyawan login dengan corporate credentials (Active Directory) dan mendapat akses ke AWS tanpa perlu IAM user terpisah. Tidak perlu manage credentials di dua tempat.

**Kenapa bukan yang lain:**
- **A** — Membuat IAM user per karyawan adalah anti-pattern untuk enterprise. Tidak scalable dan tetap butuh manage credentials AWS terpisah.
- **C** — IAM Group hanya mengorganisir IAM users, tidak menyelesaikan masalah integrasi Active Directory.
- **D** — Cognito untuk authenticate end-users aplikasi (customer-facing), bukan karyawan internal yang butuh akses ke AWS console/services.

**Konsep yang diuji:** Identity federation, IAM Identity Center, Active Directory integration.

---

## Soal 2 — Jawaban: C

**Deactivate atau delete access key yang bocor segera**

**Kenapa C:**
Prioritas pertama adalah **stop the bleeding** — matikan key yang bocor secepat mungkin untuk menghentikan unauthorized access. Rotate (B) itu langkah berikutnya setelah key lama dimatikan.

**Kenapa bukan yang lain:**
- **A** — Menghapus repo tidak mencabut akses yang sudah terjadi, dan key mungkin sudah di-scrape sebelum dihapus.
- **B** — Rotate membuat key baru, tapi key lama masih aktif sampai benar-benar di-deactivate/delete. Urutannya: disable dulu, baru rotate.
- **D** — AWS Support bukan langkah pertama. Tindakan langsung di console jauh lebih cepat.

**Konsep yang diuji:** Incident response, credential compromise, IAM key management.

---

## Soal 3 — Jawaban: C

**Attach IAM Role dengan S3 permissions ke EC2 instance**

**Kenapa C:**
IAM Role adalah cara yang benar untuk memberikan permissions ke AWS services. EC2 akan mendapat temporary credentials secara otomatis via instance metadata — tidak ada long-term credentials yang perlu di-manage atau bisa bocor.

**Kenapa bukan yang lain:**
- **A** — Environment variable masih merupakan long-term credentials, melanggar requirement.
- **B** — Secrets Manager untuk secrets yang benar-benar secret (passwords, API keys third-party). Untuk akses antar AWS services, IAM Role lebih tepat dan tidak butuh credentials sama sekali.
- **D** — Hardcode credentials adalah security vulnerability yang serius, jelas salah.

**Konsep yang diuji:** IAM Role untuk EC2, least privilege, no long-term credentials.

---

## Soal 4 — Jawaban: B

**S3 Object Lock dengan Compliance mode**

**Kenapa B:**
Compliance mode adalah satu-satunya mode yang **tidak bisa di-override oleh siapapun**, termasuk root account. Object tidak bisa dihapus atau retention period-nya tidak bisa diperpendek sampai periode retensi habis. Dirancang untuk regulatory compliance seperti FINRA, SEC, dll.

**Kenapa bukan yang lain:**
- **A** — Versioning melindungi dari accidental delete (object jadi "delete marker"), tapi masih bisa dihapus permanen.
- **C** — Governance mode bisa di-override oleh user dengan permission `s3:BypassGovernanceRetention`. Tidak memenuhi requirement "tidak bisa dihapus oleh siapapun".
- **D** — MFA Delete membutuhkan MFA untuk delete, tapi root account dengan MFA tetap bisa menghapus.

**Konsep yang diuji:** S3 Object Lock, Compliance vs Governance mode, regulatory retention.

---

## Soal 5 — Jawaban: B

**AWS Config rule dengan SNS notification**

**Kenapa B:**
AWS Config dirancang khusus untuk track perubahan konfigurasi resource AWS. Config rule `restricted-ssh` atau custom rule untuk Security Group bisa trigger SNS notification secara otomatis ketika ada perubahan — tanpa perlu polling atau manual check.

**Kenapa bukan yang lain:**
- **A** — Lambda polling setiap 5 menit adalah solusi custom yang butuh maintenance, tidak scalable, dan ada delay hingga 5 menit. Overhead tinggi.
- **C** — VPC Flow Logs mencatat network traffic, bukan perubahan konfigurasi Security Group.
- **D** — CloudWatch dashboard untuk visualisasi metrics, tidak detect perubahan konfigurasi.

**Konsep yang diuji:** AWS Config, configuration change detection, CloudTrail vs Config perbedaan use case.

---

## Soal 6 — Jawaban: B

**AWS WAF dengan managed rule groups di depan API Gateway**

**Kenapa B:**
WAF adalah layer 7 firewall yang dirancang untuk melindungi dari web exploits termasuk SQL injection dan XSS. AWS menyediakan managed rule groups (AWS Managed Rules) yang sudah mencakup OWASP Top 10 — tinggal enable tanpa perlu tulis rules dari scratch.

**Kenapa bukan yang lain:**
- **A** — Shield Advanced untuk DDoS protection, bukan application-layer attacks seperti SQLi/XSS.
- **C** — Security Groups bekerja di layer 4 (IP/port), tidak bisa inspect HTTP payload untuk detect SQLi/XSS.
- **D** — Throttling membatasi jumlah request, tidak melindungi dari content-based attacks.

**Konsep yang diuji:** AWS WAF, layer 7 protection, SQLi/XSS mitigation.

---

## Soal 7 — Jawaban: C

**SCP di Organizations yang deny CloudTrail actions, attach ke root OU**

**Kenapa C:**
SCP (Service Control Policy) adalah satu-satunya mekanisme yang bisa membatasi permissions di level Organizations — berlaku untuk semua accounts di bawahnya, **termasuk account dengan AdministratorAccess**. SCP yang attach ke root OU berlaku ke semua member accounts.

**Kenapa bukan yang lain:**
- **A** — IAM policy hanya berlaku untuk IAM entity di account tersebut. Admin bisa saja hapus atau modify IAM policy itu sendiri.
- **B** — Config dengan auto-remediate bisa mendeteksi dan memperbaiki, tapi ada window waktu di mana CloudTrail sudah mati. Tidak preventif.
- **D** — Alert di CloudWatch hanya notifikasi setelah kejadian, tidak mencegah.

**Konsep yang diuji:** SCP, AWS Organizations, preventive vs detective controls.

---

## Soal 8 — Jawaban: C

**AWS Secrets Manager**

**Kenapa C:**
Secrets Manager adalah satu-satunya opsi yang punya fitur **auto-rotation native** untuk database credentials (RDS, Redshift, dll) — ini sesuai requirement soal. Rotation dilakukan via Lambda function yang sudah di-manage AWS.

**Kenapa bukan yang lain:**
- **A** — Enkripsi config file dengan KMS lebih aman dari plaintext, tapi tidak menyelesaikan masalah management dan tidak ada auto-rotation.
- **B** — SSM Parameter Store SecureString bisa simpan credentials dengan enkripsi, tapi **tidak ada auto-rotation native**. Harus buat rotasi sendiri.
- **D** — S3 dengan SSE-KMS untuk storing credentials adalah anti-pattern — tidak ada mekanisme rotation dan access control-nya lebih sulit di-manage.

**Konsep yang diuji:** Secrets Manager vs SSM Parameter Store, auto-rotation.

---

## Soal 9 — Jawaban: B

**AWS CloudTrail**

**Kenapa B:**
CloudTrail merekam semua **API calls** di akun AWS — siapa (IAM user/role), action apa, kapan, dari IP mana. Termasuk `TerminateInstances` API call yang menghapus EC2. Default retention 90 hari di Event History, bisa lebih lama jika disimpan ke S3.

**Kenapa bukan yang lain:**
- **A** — CloudWatch Metrics untuk performance metrics (CPU, memory, network), bukan audit trail API calls.
- **C** — AWS Config mencatat **perubahan konfigurasi resource** (resource configuration history), bukan siapa yang melakukan action. Bisa tau bahwa EC2 instance berubah state, tapi tidak se-detail CloudTrail untuk audit.
- **D** — GuardDuty untuk threat detection (analisis log untuk cari anomali), bukan sebagai audit trail.

**Konsep yang diuji:** CloudTrail use case, audit trail, API logging.

---

## Soal 10 — Jawaban: B

**VPC Endpoint (Gateway type) untuk S3**

**Kenapa B:**
Gateway Endpoint untuk S3 (dan DynamoDB) memungkinkan traffic dari VPC ke S3 melalui AWS network — tidak pernah melewati public internet. Free, dan diimplementasikan via route table entry.

**Kenapa bukan yang lain:**
- **A** — NAT Gateway memungkinkan EC2 di private subnet akses internet, tapi traffic ke S3 tetap melewati public internet.
- **C** — Interface Endpoint untuk S3 juga valid secara teknis, tapi **Gateway Endpoint lebih tepat** karena khusus untuk S3/DynamoDB, gratis, dan lebih simpel. Interface Endpoint untuk S3 ada biaya per jam dan per GB.
- **D** — Direct Connect untuk koneksi dedicated on-premise ke AWS, overkill untuk use case ini.

**Konsep yang diuji:** VPC Endpoints, Gateway vs Interface Endpoint, private connectivity ke S3.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | IAM Identity Center, Active Directory federation |
| 2 | Credential compromise incident response |
| 3 | IAM Role untuk EC2, no long-term credentials |
| 4 | S3 Object Lock Compliance mode |
| 5 | AWS Config untuk change detection |
| 6 | AWS WAF, SQLi/XSS protection |
| 7 | SCP di AWS Organizations |
| 8 | Secrets Manager auto-rotation |
| 9 | CloudTrail audit trail |
| 10 | VPC Gateway Endpoint untuk S3 |
