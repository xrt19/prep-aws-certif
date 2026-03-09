# Jawaban — Set 06
## Domain: Secure Architectures

---

## Soal 1 — Jawaban: B

**AWS Control Tower**

**Kenapa B:**
Control Tower adalah service yang dirancang khusus untuk **setup dan govern multi-account AWS environment** secara otomatis. Fitur utamanya:
- **Landing Zone**: multi-account environment yang sudah di-configured sesuai best practices (Log Archive account, Audit account, dll)
- **Account Factory**: vending machine untuk buat account baru secara otomatis dengan template yang konsisten
- **Guardrails**: preventive (SCP) dan detective (Config rules) controls yang langsung aktif
- Terintegrasi dengan Organizations, SSO, dan Config

**Kenapa bukan yang lain:**
- **A** — Organizations + manual SCP bisa dicapai sendiri tapi butuh effort besar dan tidak ada account vending atau pre-built landing zone.
- **C** — Config untuk compliance monitoring, bukan untuk setup multi-account structure.
- **D** — Service Catalog untuk catalog produk/templates yang bisa di-deploy, bukan untuk governance multi-account environment secara menyeluruh.

**Konsep yang diuji:** AWS Control Tower, landing zone, account factory, multi-account governance.

---

## Soal 2 — Jawaban: B

**S3 Glacier Vault Lock**

**Kenapa B:**
Glacier Vault Lock memungkinkan deploy **immutable compliance controls** pada Glacier vault via Vault Lock policy. Setelah policy di-lock, **tidak bisa diubah atau dihapus oleh siapapun** — termasuk root account. Policy bisa berisi `Deny DeleteArchive` dengan kondisi retensi berbasis tanggal. Ini adalah mekanisme yang spesifik untuk Glacier dan dirancang untuk WORM (Write Once Read Many) compliance.

**Kenapa bukan yang lain:**
- **A** — S3 Object Lock Compliance mode berlaku untuk S3 Standard/IA/Glacier **Instant Retrieval** (object di S3 bucket dengan storage class). Glacier Vault adalah produk berbeda (sekarang disebut S3 Glacier Flexible Retrieval) yang punya mekanisme locknya sendiri yaitu Vault Lock.
- **C** — MFA Delete untuk mencegah hapus object di S3 versioned bucket, bukan untuk Glacier vaults. Dan root account dengan MFA tetap bisa hapus.
- **D** — IAM policy bisa diubah oleh admin/root kapan saja — tidak immutable.

**Konsep yang diuji:** S3 Glacier Vault Lock, WORM compliance, immutable policy.

---

## Soal 3 — Jawaban: C

**Amazon Detective**

**Kenapa C:**
Detective adalah service khusus untuk **investigasi dan analisis security incidents**. Ia mengumpulkan data dari GuardDuty, CloudTrail, dan VPC Flow Logs lalu membangun **graph model** yang menvisualisasikan hubungan antar:
- IAM entities (siapa yang akses apa)
- Resources (EC2, S3, dll)
- IP addresses dan API calls
- Timeline aktivitas

Ini mempercepat root cause analysis yang biasanya butuh waktu lama jika dilakukan manual.

**Kenapa bukan yang lain:**
- **A** — Security Hub menampilkan findings tapi tidak melakukan analisis hubungan atau visualisasi.
- **B** — GuardDuty generate findings (alerting), bukan tool investigasi. Ia input untuk Detective.
- **D** — CloudTrail query bisa dilakukan tapi hasilnya raw logs yang butuh analisis manual. Detective sudah memproses dan memvisualisasikan data tersebut secara otomatis.

**Konsep yang diuji:** Amazon Detective, security incident investigation, graph-based analysis, root cause analysis.

---

## Soal 4 — Jawaban: C

**Customer Managed Policies**

**Kenapa C:**
Customer Managed Policy adalah IAM policy yang dibuat dan dikelola sendiri, bisa di-**attach ke banyak IAM users, groups, dan roles sekaligus**. Ketika policy di-update, semua entities yang menggunakan policy tersebut **otomatis mendapat versi terbaru** — tidak perlu update satu per satu. Policy versioning tersedia hingga 5 versi.

**Kenapa bukan yang lain:**
- **A** — Inline policy adalah policy yang **embedded langsung ke satu IAM entity** — tidak bisa di-share atau di-attach ke entity lain. Justru harus update satu per satu di setiap entity. Ini kebalikan dari yang dibutuhkan.
- **B** — AWS Managed Policy di-manage oleh AWS, tidak bisa di-edit atau dikustomisasi sesuai kebutuhan perusahaan.
- **D** — IAM Groups membantu organisasi tapi tetap memerlukan policy yang di-attach ke group. Poin utamanya adalah Customer Managed Policy sebagai policy yang di-share tersebut.

**Konsep yang diuji:** Inline policy vs Customer Managed Policy vs AWS Managed Policy, policy reusability.

---

## Soal 5 — Jawaban: C

**Session policy saat memanggil AssumeRole**

**Kenapa C:**
Session policy adalah IAM policy yang di-pass sebagai parameter saat memanggil `AssumeRole`, `AssumeRoleWithWebIdentity`, atau `GetFederationToken`. **Effective permissions = intersection (AND) antara role policy dan session policy** — tidak bisa lebih luas dari role policy maupun dari session policy sendiri. Ini memungkinkan pembatasan permissions secara dinamis per-session tanpa mengubah role.

Contoh: Role punya `s3:*`, session policy hanya `s3:GetObject` → effective permission hanya `s3:GetObject`.

**Kenapa bukan yang lain:**
- **A** — Permission Boundary adalah maximum permission untuk entity saat di-create, bukan untuk satu session tertentu. Butuh modify role itu sendiri.
- **B** — SCP berlaku di level account, tidak bisa di-scope per individual session atau per user.
- **D** — Membuat IAM user baru menyelesaikan masalah tapi bukan jawaban untuk "membatasi permissions saat session tanpa mengubah role policy" — soal menanya mekanisme yang spesifik.

**Konsep yang diuji:** STS session policy, AssumeRole, effective permissions = intersection, per-session scoping.

---

## Soal 6 — Jawaban: B

**Security Group referencing — allow inbound dari Security Group ID spesifik**

**Kenapa B:**
Security Groups mendukung **referencing Security Group lain sebagai source** di inbound rules — alih-alih menggunakan CIDR range. Ketika SG-VPC-B di-referensikan di inbound rule SG-VPC-A, hanya instances yang memiliki SG-VPC-B yang bisa terkoneksi — bukan seluruh subnet atau VPC. Ini memberikan kontrol yang lebih granular: jika ada instance baru di VPC-B yang tidak punya SG-VPC-B, ia tidak bisa akses database di VPC-A.

Catatan: Cross-VPC SG referencing memerlukan VPC Peering atau Transit Gateway antara dua VPC tersebut.

**Kenapa bukan yang lain:**
- **A** — Allow seluruh CIDR VPC-B berarti semua instances di VPC-B bisa akses DB, termasuk instances yang seharusnya tidak punya akses. Terlalu broad.
- **C** — NACL bekerja di level subnet dengan CIDR — tidak bisa filter berdasarkan Security Group atau instance spesifik.
- **D** — WAF untuk Layer 7 HTTP/HTTPS traffic, tidak cocok untuk database connections (MySQL port 3306, PostgreSQL port 5432, dll).

**Konsep yang diuji:** Security Group referencing, cross-VPC SG reference, granular access control vs CIDR.

---

## Soal 7 — Jawaban: B

**AWS WAF rate-based rule**

**Kenapa B:**
WAF rate-based rules dirancang persis untuk use case ini — **block IP address yang melebihi request threshold** dalam window waktu tertentu (5 menit). Bisa di-scope ke URI spesifik seperti `/login`, sehingga hanya endpoint login yang di-rate-limit. Ketika IP melebihi threshold, WAF otomatis block selama periode tertentu.

**Kenapa bukan yang lain:**
- **A** — Shield Advanced untuk volumetric DDoS (Layer 3/4). Credential stuffing adalah application-layer attack (Layer 7) yang butuh WAF, bukan Shield.
- **C** — GuardDuty mendeteksi threats tapi tidak melakukan blocking secara real-time. GuardDuty generate findings, bukan firewall.
- **D** — Scale up EC2 hanya menambah capacity untuk absorb traffic, tidak menghentikan credential stuffing. Attacker tetap bisa terus mencoba dengan lebih banyak request.

**Konsep yang diuji:** WAF rate-based rules, credential stuffing, Layer 7 protection, IP-based rate limiting.

---

## Soal 8 — Jawaban: B

**S3 Object Ownership dengan mode `Bucket owner enforced`**

**Kenapa B:**
S3 Object Ownership adalah setting di level bucket yang mengontrol siapa pemilik objects yang di-upload ke bucket tersebut. Dengan mode **`Bucket owner enforced`**:
- Semua objects otomatis dimiliki bucket owner, terlepas dari siapa yang upload
- ACL di-disable secara otomatis (tidak bisa digunakan lagi)
- Bucket owner selalu punya full control atas semua objects

Ini adalah setting yang direkomendasikan AWS untuk semua bucket baru.

**Kenapa bukan yang lain:**
- **A** — Manual ownership transfer tidak scalable untuk upload volume tinggi dan vendor mungkin tidak selalu melakukannya.
- **C** — Require `bucket-owner-full-control` ACL via bucket policy adalah workaround yang bisa bekerja, tapi jika vendor tidak include header ACL tersebut, upload akan di-reject. `Bucket owner enforced` lebih simpel dan tidak bergantung pada kepatuhan vendor.
- **D** — IAM role untuk vendor tidak otomatis transfer ownership — ownership di S3 ditentukan oleh AWS account yang melakukan API call, bukan role.

**Konsep yang diuji:** S3 Object Ownership, Bucket owner enforced, ACL disable, cross-account upload.

---

## Soal 9 — Jawaban: C

**AWS Trusted Advisor**

**Kenapa C:**
Trusted Advisor adalah service yang memberikan **rekomendasi best practices** di beberapa kategori, termasuk kategori **Security**. Checks keamanan yang tersedia meliputi:
- Root account MFA
- IAM use (apakah sudah pakai IAM, bukan root)
- Security Groups dengan akses tidak terbatas (0.0.0.0/0 pada port sensitif)
- S3 bucket permissions
- IAM access key rotation
- CloudTrail logging

Beberapa checks gratis, beberapa butuh Business/Enterprise Support plan.

**Kenapa bukan yang lain:**
- **A** — GuardDuty untuk threat detection berdasarkan actual activity, bukan rekomendasi konfigurasi.
- **B** — Config untuk compliance terhadap rules yang di-define sendiri — butuh setup rules terlebih dahulu, bukan rekomendasi out-of-the-box.
- **D** — Access Analyzer untuk analisis resource access dan unused permissions, tidak memberikan rekomendasi security posture secara umum.

**Konsep yang diuji:** AWS Trusted Advisor, security best practice checks, MFA root account, security group recommendations.

---

## Soal 10 — Jawaban: B

**Security Group terpisah per tier dengan SG referencing**

**Kenapa B:**
Ini adalah **defense-in-depth** pattern untuk multi-tier architecture:
- **Web-SG**: allow inbound port 443 dari 0.0.0.0/0 (internet)
- **App-SG**: allow inbound hanya dari **Web-SG** (bukan CIDR)
- **DB-SG**: allow inbound port 3306 hanya dari **App-SG** (bukan CIDR)

Dengan SG referencing, jika instance web di-decommission dan diganti baru, akses tetap terkontrol berdasarkan Security Group membership — bukan IP address. Ini jauh lebih secure dan maintainable dari CIDR-based rules.

**Kenapa bukan yang lain:**
- **A** — Satu SG untuk semua tier berarti tidak ada isolasi — semua instance bisa komunikasi satu sama lain. NACL bisa membantu tapi lebih kompleks dan stateless.
- **C** — CIDR ranges private subnet kurang granular — semua instances di subnet app bisa akses DB, termasuk instance yang tidak seharusnya.
- **D** — Filter berdasarkan port saja (tanpa source restriction) masih memungkinkan semua instances di VPC akses database selama pakai port yang benar.

**Konsep yang diuji:** Multi-tier security groups, SG chaining/referencing, defense in depth architecture.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | AWS Control Tower, landing zone, account factory |
| 2 | S3 Glacier Vault Lock, WORM compliance |
| 3 | Amazon Detective, security incident investigation |
| 4 | Customer Managed Policy vs Inline Policy |
| 5 | STS session policy, AssumeRole, effective permissions |
| 6 | Security Group referencing cross-VPC |
| 7 | WAF rate-based rules, credential stuffing |
| 8 | S3 Object Ownership, Bucket owner enforced |
| 9 | AWS Trusted Advisor, security checks |
| 10 | Multi-tier SG chaining, defense in depth |
