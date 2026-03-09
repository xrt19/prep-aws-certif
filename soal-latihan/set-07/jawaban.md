# Jawaban — Set 07
## Domain: Secure Architectures

---

## Soal 1 — Jawaban: B

**AWS Private CA (ACM Private Certificate Authority)**

**Kenapa B:**
AWS Private CA memungkinkan perusahaan membuat dan mengelola **private Certificate Authority hierarchy** sendiri di AWS. Certificate yang diterbitkan bisa digunakan untuk domain internal, microservices mTLS, VPN, dan lainnya — tidak perlu domain publik yang terverifikasi. Perusahaan punya full control: bisa define certificate templates, revoke certificates, dan manage CRL/OCSP.

**Kenapa bukan yang lain:**
- **A** — ACM public certificate untuk domain publik yang terverifikasi via DNS/email validation. Tidak bisa issue certificate untuk domain internal atau IP address private.
- **C** — Self-signed certificate tidak ada chain of trust terpusat — setiap service harus trust masing-masing cert secara manual. Tidak scalable untuk banyak microservices.
- **D** — Third-party CA untuk public certificate, tidak practical dan mahal untuk internal communication di AWS.

**Konsep yang diuji:** AWS Private CA, internal TLS, private certificate authority, mTLS.

---

## Soal 2 — Jawaban: B

**AWS Verified Access**

**Kenapa B:**
AWS Verified Access adalah implementasi **zero-trust network access (ZTNA)** dari AWS. Cara kerja:
- Karyawan akses aplikasi web internal tanpa VPN
- Verified Access evaluate identity (via IAM Identity Center, Okta, dll) dan device posture (via AWS Verified Access trust providers)
- Akses di-grant per-aplikasi berdasarkan policy — bukan broad network access seperti VPN
- Semua access di-log ke S3/CloudWatch

**Kenapa bukan yang lain:**
- **A** — Client VPN memberikan akses broad ke network (bukan per-aplikasi), masih memerlukan client software, dan tidak zero-trust by default.
- **C** — ALB dengan Cognito bisa authenticate users tapi tidak evaluate device posture atau bisa di-scale ke banyak aplikasi internal secara terpusat.
- **D** — Direct Connect untuk dedicated network connectivity dari on-premise ke AWS, bukan untuk employee access ke aplikasi.

**Konsep yang diuji:** AWS Verified Access, zero-trust network access, identity-based access tanpa VPN.

---

## Soal 3 — Jawaban: B

**CORS configuration di API Gateway atau S3**

**Kenapa B:**
CORS (Cross-Origin Resource Sharing) adalah mekanisme browser untuk mengizinkan/menolak cross-origin requests berdasarkan HTTP headers (`Access-Control-Allow-Origin`, dll). Untuk menyelesaikan masalah ini, **server (API Gateway atau S3) harus return CORS headers** yang mengizinkan origin `app.example.com`. Di API Gateway, ada menu CORS built-in. Di S3, ada CORS configuration JSON di bucket settings.

**Kenapa bukan yang lain:**
- **A** — HTTPS (enkripsi) tidak ada hubungannya dengan CORS (cross-origin policy). Ini dua hal berbeda.
- **C** — CloudFront bisa membantu jika semua content di-serve dari satu domain, tapi tidak secara otomatis solve CORS — tetap butuh configure CORS headers di origin.
- **D** — Memindahkan SPA ke domain yang sama bukan solusi arsitektur yang realistic jika API dan SPA memang di-design terpisah.

**Konsep yang diuji:** CORS, cross-origin requests, API Gateway CORS configuration, S3 CORS.

---

## Soal 4 — Jawaban: B

**AD Connector**

**Kenapa B:**
AD Connector adalah **directory proxy** yang meneruskan authentication dan directory lookup requests ke on-premise Active Directory yang sudah ada — tanpa menyimpan atau meng-cache data di AWS. Aplikasi AWS (seperti WorkSpaces, QuickSight, IAM Identity Center) bisa authenticate ke on-premise AD via AD Connector. Tidak perlu replicate atau migrate AD.

**Kenapa bukan yang lain:**
- **A** — AWS Managed Microsoft AD membuat AD baru di AWS, dan bisa di-setup sebagai trust dengan on-premise AD — tapi ini melibatkan replikasi atau trust relationship, lebih complex dari yang dibutuhkan. Tepat jika butuh full AD features di AWS, bukan hanya proxy auth.
- **C** — Buat IAM users manual dan sync password bukan solusi enterprise, tidak scalable, dan tetap butuh manage credentials di dua tempat.
- **D** — Cognito User Pool bukan dirancang untuk integrate langsung dengan on-premise LDAP/AD secara native.

**Konsep yang diuji:** AD Connector vs AWS Managed Microsoft AD, AWS Directory Service, on-premise AD integration.

---

## Soal 5 — Jawaban: B

**Saat role chaining, maximum session duration dibatasi menjadi 1 jam**

**Kenapa B:**
Ini adalah **exam trap yang penting**. Ketika role di-assume secara langsung (bukan chaining), session duration bisa dikonfigurasi dari 1 jam hingga maksimum yang di-set di role (bisa sampai 12 jam). Namun ketika terjadi **role chaining** (role A assume role B yang assume role C), AWS membatasi maximum session duration menjadi **1 jam — tidak bisa di-extend**. Implikasi: jika workload butuh session panjang, jangan gunakan role chaining.

**Kenapa bukan yang lain:**
- **A** — Salah. Ada batasan 1 jam untuk role chaining, ini berbeda dari single role assumption.
- **C** — AWS tidak otomatis block role chaining. Ini fitur yang valid, hanya ada batasan durasi.
- **D** — Siapapun dengan permission `sts:AssumeRole` bisa chain roles, tidak eksklusif untuk root account.

**Konsep yang diuji:** IAM role chaining, STS session duration limit 1 jam, AssumeRole behavior.

---

## Soal 6 — Jawaban: B

**CloudFront Signed URLs atau Signed Cookies**

**Kenapa B:**
CloudFront Signed URLs/Cookies adalah mekanisme untuk **membatasi akses ke CloudFront content** hanya untuk user yang punya valid signature:
- **Signed URL**: untuk akses ke **satu file spesifik**, dengan expiry time dan optional IP restriction
- **Signed Cookies**: untuk akses ke **banyak files sekaligus** (misalnya semua video dalam satu subscription), lebih baik untuk streaming

Aplikasi back-end yang generate signature menggunakan CloudFront key pair. URL/cookie yang expired atau tidak valid akan di-reject oleh CloudFront edge.

**Kenapa bukan yang lain:**
- **A** — Bucket public berarti siapapun yang tahu URL S3 langsung bisa akses konten, bypassing CloudFront restrictions. Harus gunakan OAC (Origin Access Control) + private bucket.
- **C** — S3 pre-signed URL valid untuk akses langsung ke S3, tapi bypass CloudFront — tidak memanfaatkan edge caching dan tidak bisa kontrol access via CloudFront policies.
- **D** — CloudFront tidak support IAM authentication untuk end-users publik — IAM untuk AWS principals, bukan consumer users.

**Konsep yang diuji:** CloudFront Signed URLs vs Signed Cookies, private content distribution, OAC.

---

## Soal 7 — Jawaban: B

**Kombinasi Gateway Endpoints (S3, DynamoDB) dan Interface Endpoints (services lainnya)**

**Kenapa B:**
AWS menyediakan dua jenis VPC Endpoints:
- **Gateway Endpoints**: khusus untuk **S3 dan DynamoDB** — gratis, diimplementasikan via route table, tidak butuh ENI
- **Interface Endpoints (PrivateLink)**: untuk semua AWS services lainnya (KMS, SSM, SNS, SQS, dll) — menggunakan ENI di subnet, ada biaya per jam dan per GB

Kombinasi keduanya memungkinkan EC2 di private subnet mengakses semua AWS services yang dibutuhkan tanpa melalui NAT Gateway sama sekali.

**Kenapa bukan yang lain:**
- **A** — Pindah ke public subnet melanggar prinsip least privilege network access.
- **C** — Tidak ada satu Interface Endpoint yang cover semua AWS services. Setiap service butuh endpoint sendiri.
- **D** — Direct Connect untuk koneksi dedicated dari on-premise ke AWS, bukan pengganti NAT Gateway untuk akses ke AWS services dari dalam VPC.

**Konsep yang diuji:** Gateway Endpoints vs Interface Endpoints, S3/DynamoDB gateway, PrivateLink, NAT Gateway cost reduction.

---

## Soal 8 — Jawaban: B

**Service-Linked Role**

**Kenapa B:**
Service-Linked Role adalah tipe khusus IAM role yang **dibuat, dikelola, dan digunakan langsung oleh AWS service**. Karakteristiknya:
- Di-create otomatis saat pertama kali service digunakan (atau bisa di-create manual)
- Trust policy sudah di-define oleh AWS — hanya service tersebut yang bisa assume role ini
- Permission policy sudah di-define oleh AWS sesuai kebutuhan service
- Tidak bisa edit atau delete selama masih ada resources yang menggunakannya

Contoh: `AWSServiceRoleForECS`, `AWSServiceRoleForElasticLoadBalancing`, dll.

**Kenapa bukan yang lain:**
- **A** — AWS Managed Policies di-attach ke IAM entities customer, bukan mekanisme untuk AWS service itu sendiri manage resources.
- **C** — Instance profile untuk EC2 applications, bukan untuk AWS service management layer.
- **D** — Root credentials tidak pernah di-share ke AWS services — ini security violation yang fundamental.

**Konsep yang diuji:** Service-Linked Role, AWS service permissions, automatic role creation.

---

## Soal 9 — Jawaban: D

**Semua jawaban relevan untuk dipertimbangkan bersama**

**Kenapa D:**
Soal ini menguji pemahaman tentang trade-off antara dua pendekatan:

**VPC Peering:**
- **By design tidak support transitive routing** — VPC-A peered dengan VPC-B, VPC-B peered dengan VPC-C, VPC-A **tidak otomatis** bisa reach VPC-C
- Pro: isolasi terjamin by default, simpel untuk few VPCs
- Con: tidak scalable — untuk N VPCs butuh N*(N-1)/2 peering connections

**Transit Gateway:**
- Support transitive routing dan hub-and-spoke topology
- Scalable untuk banyak VPCs
- Con: perlu **desain route tables TGW dengan hati-hati** untuk prevent unwanted routing antar VPC

Untuk puluhan VPCs: TGW lebih tepat secara operasional, dengan route table control sebagai security mechanism. Pemahaman tentang keduanya diperlukan untuk exam.

**Kenapa bukan A, B, atau C saja:**
Jawaban A dan B keduanya benar sebagai pernyataan. C adalah rekomendasi yang valid. Soal menanya mana yang "relevan untuk dipertimbangkan" — semua tiga pernyataan tersebut relevan dan benar.

**Konsep yang diuji:** VPC Peering non-transitive, Transit Gateway route tables, network topology security.

---

## Soal 10 — Jawaban: B

**Shield Standard + WAF + Route 53 Resolver Query Logs + GuardDuty**

**Kenapa B:**
Mapping requirement ke service:
| Requirement | Service |
|-------------|---------|
| Proteksi DDoS volumetric | **Shield Standard** (otomatis aktif, free) |
| Block SQL injection & XSS | **AWS WAF** (Layer 7 filtering) |
| Log semua DNS queries | **Route 53 Resolver Query Logs** |
| Detect anomalous behavior dalam VPC | **GuardDuty** (analisis VPC Flow Logs, DNS, CloudTrail) |

**Kenapa bukan yang lain:**
- **A** — VPC Flow Logs mencatat network metadata tapi tidak log DNS queries. Tidak memenuhi requirement ketiga.
- **C** — CloudTrail mencatat API calls, bukan DNS queries. Inspector untuk vulnerability scan, bukan anomaly detection dalam VPC.
- **D** — Network Firewall untuk DPI dan domain filtering tapi tidak khusus untuk DDoS protection. Macie untuk S3 data classification, tidak relevan untuk requirement yang ada.

**Konsep yang diuji:** Multi-service security architecture, Route 53 Resolver Query Logs, service selection matching requirements.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | AWS Private CA, internal TLS, mTLS microservices |
| 2 | AWS Verified Access, zero-trust, tanpa VPN |
| 3 | CORS configuration, API Gateway, cross-origin |
| 4 | AD Connector vs AWS Managed Microsoft AD |
| 5 | IAM role chaining, 1 jam session duration limit |
| 6 | CloudFront Signed URLs vs Signed Cookies |
| 7 | Gateway Endpoints + Interface Endpoints kombinasi |
| 8 | Service-Linked Role, AWS service permissions |
| 9 | VPC Peering non-transitive vs Transit Gateway |
| 10 | Multi-service security architecture mapping |
