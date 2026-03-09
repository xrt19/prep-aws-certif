# Jawaban — Set 04
## Domain: Secure Architectures

---

## Soal 1 — Jawaban: B

**AWS Shield Standard — sudah aktif otomatis di semua akun AWS**

**Kenapa B:**
AWS Shield Standard aktif **secara otomatis untuk semua akun AWS tanpa biaya tambahan** dan memberikan proteksi terhadap DDoS volumetric dan infrastructure layer attacks (Layer 3/4). Terintegrasi langsung dengan CloudFront, Route 53, ELB, dan AWS Global Accelerator. Tidak perlu konfigurasi apapun.

**Kenapa bukan yang lain:**
- **A** — WAF rate-based rules untuk mitigasi application-layer attacks (Layer 7), bukan volumetric DDoS. WAF tidak menangani flood traffic di Layer 3/4.
- **C** — Shield Advanced memberikan proteksi lebih canggih + DRT support + cost protection, tapi berbayar (~$3,000/bulan). Untuk pertanyaan "otomatis tanpa konfigurasi", Shield Standard sudah menjawab kebutuhan dasar.
- **D** — Auto Scaling menambah capacity tapi tidak memblokir DDoS traffic — tetap akan ada biaya dan kemungkinan downtime selama scaling.

**Konsep yang diuji:** AWS Shield Standard vs Advanced, DDoS protection layers, auto-enabled services.

---

## Soal 2 — Jawaban: B

**Konfigurasikan Session Manager preferences untuk stream session logs ke S3 dan/atau CloudWatch Logs**

**Kenapa B:**
Session Manager memiliki fitur built-in untuk **log session activity** — bisa stream ke S3 (untuk long-term storage) dan CloudWatch Logs (untuk real-time monitoring dan alerting). Konfigurasi di SSM > Session Manager > Preferences. Ini mencatat semua input/output sesi, bukan hanya API calls.

**Kenapa bukan yang lain:**
- **A** — CloudTrail merekam API calls SSM (seperti `StartSession`, `TerminateSession`), tapi **tidak merekam konten/isi dari sesi** itu sendiri. Tidak cukup untuk audit lengkap.
- **C** — VPC Flow Logs mencatat metadata network traffic (IP, port, bytes), tidak mencatat konten sesi atau commands yang dijalankan.
- **D** — Config mencatat perubahan konfigurasi resource, tidak bisa record interactive sessions.

**Konsep yang diuji:** Systems Manager Session Manager, session logging, audit trail untuk remote access.

---

## Soal 3 — Jawaban: B

**RAM memungkinkan share resources AWS ke accounts lain atau Organizations tanpa duplikasi**

**Kenapa B:**
AWS Resource Access Manager (RAM) dirancang persis untuk use case ini — share resources seperti Transit Gateway, VPC Subnets, Route 53 Resolver rules, License Manager configurations, dan lainnya ke multiple accounts atau seluruh Organizations. Resource tetap ada di satu account (owner), accounts lain mendapat akses tanpa resource di-copy atau di-duplikasi.

**Kenapa bukan yang lain:**
- **A** — RAM tidak terbatas pada VPC yang sama. Resource bisa di-share lintas accounts dan lintas regions (tergantung resource type-nya).
- **C** — VPC Peering bukan prasyarat RAM. Keduanya adalah mekanisme berbeda — RAM untuk share resource, VPC Peering untuk koneksi network antar VPC.
- **D** — RAM tidak copy resource. Account penerima hanya mendapat akses ke resource yang ada di account owner — ini keunggulan utama RAM (single source of truth).

**Konsep yang diuji:** AWS RAM, resource sharing, Transit Gateway sharing antar accounts.

---

## Soal 4 — Jawaban: B

**Generate S3 pre-signed URL dengan expiry 15 menit dari server-side**

**Kenapa B:**
S3 Pre-signed URL adalah mekanisme untuk memberikan **akses sementara dan terbatas** ke object S3 private. URL di-generate server-side menggunakan credentials IAM role/user yang punya akses S3, dengan expiry time yang ditentukan. Setelah 15 menit, URL tidak valid lagi. User tidak perlu punya AWS credentials sendiri.

**Kenapa bukan yang lain:**
- **A** — Membuat bucket public untuk data personal adalah security violation serius. Siapapun bisa akses semua file.
- **C** — Transfer Acceleration untuk mempercepat upload/download via CloudFront edge locations, tidak ada hubungannya dengan access control atau time-limited access.
- **D** — CloudFront signed cookies valid untuk use case dimana user perlu akses ke **banyak files sekaligus** (misalnya streaming video). Untuk single file download dengan expiry pendek, pre-signed URL lebih simpel dan tepat.

**Konsep yang diuji:** S3 pre-signed URL, time-limited access, private bucket access pattern.

---

## Soal 5 — Jawaban: C

**Amazon Inspector**

**Kenapa C:**
Inspector adalah managed vulnerability assessment service yang secara otomatis scan EC2 instances, ECR container images, dan Lambda functions untuk:
- **Network exposure** (port terbuka yang tidak perlu)
- **Known CVEs** di OS packages dan application dependencies
- **Software vulnerabilities**

Inspector v2 menggunakan SSM Agent yang sudah ada (tidak perlu install agent terpisah) dan berjalan secara continuous.

**Kenapa bukan yang lain:**
- **A** — GuardDuty untuk threat detection (malicious behavior, anomaly), bukan vulnerability assessment. GuardDuty tidak scan CVEs.
- **B** — Config untuk configuration compliance (apakah resource sesuai desired state), bukan vulnerability scanning.
- **D** — Security Hub untuk aggregasi findings dari berbagai services, tidak melakukan scanning sendiri.

**Konsep yang diuji:** Amazon Inspector, vulnerability assessment, CVE scanning, Inspector v2 agentless.

---

## Soal 6 — Jawaban: B

**AWS Firewall Manager untuk deploy WAF policies secara terpusat**

**Kenapa B:**
AWS Firewall Manager adalah management service khusus untuk mengelola **security policies secara terpusat** di semua accounts dalam Organizations — termasuk WAF WebACL, Shield Advanced, Security Groups, dan Network Firewall. Ketika account baru dibuat atau ALB baru di-deploy, Firewall Manager otomatis apply policy tanpa konfigurasi manual per account.

**Kenapa bukan yang lain:**
- **A** — Share WebACL via RAM tidak otomatis apply ke resources baru. Tetap butuh konfigurasi di setiap account/resource secara manual.
- **C** — SCP bisa require WAF attachment tapi tidak bisa deploy atau konfigurasi WAF rules-nya. SCP adalah guardrail, bukan deployment tool.
- **D** — Config rule bersifat detective — detect ketidakpatuhan setelah terjadi, tidak otomatis deploy WAF.

**Konsep yang diuji:** AWS Firewall Manager, centralized security policy management, Organizations integration.

---

## Soal 7 — Jawaban: B

**KMS Key Policy — harus ada explicit Allow di key policy untuk setiap action**

**Kenapa B:**
KMS Key Policy adalah **resource-based policy yang menjadi kontrol utama akses ke KMS key** — berbeda dengan hampir semua AWS resources lain yang default allow IAM policies. Aturan penting: **IAM policy saja tidak cukup tanpa Allow di key policy**. Jika key policy tidak explicitly allow `kms:DeleteKey` atau `kms:DisableKey`, action tersebut akan ditolak meskipun IAM policy punya `kms:*`. Admin bisa grant encrypt/decrypt di key policy tanpa grant delete/disable.

**Kenapa bukan yang lain:**
- **A** — Tidak ada "S3 bucket policy pada key material" — ini bukan konsep yang ada di KMS.
- **C** — SCP valid tapi terlalu broad untuk seluruh account. Key Policy lebih granular — bisa per-key, per-action, per-principal.
- **D** — Permission Boundary membatasi maximum IAM permissions, tapi untuk KMS, key policy adalah layer kontrol yang lebih fundamental dan harus ada Allow secara eksplisit.

**Konsep yang diuji:** KMS key policy, key policy vs IAM policy precedence, KMS access control.

---

## Soal 8 — Jawaban: C

**Cognito User Pool untuk autentikasi, kemudian Cognito Identity Pool untuk exchange token jadi AWS credentials**

**Kenapa C:**
Ini adalah pattern Cognito yang paling umum:
1. **User Pool** — handles user directory: register, login, MFA, password policy. Output: JWT tokens (ID token, access token).
2. **Identity Pool** — exchange JWT token dari User Pool (atau social identity provider) menjadi **temporary AWS credentials** (via STS AssumeRoleWithWebIdentity).

Keduanya bekerja bersama untuk melengkapi kedua kebutuhan.

**Kenapa bukan yang lain:**
- **A** — User Pool hanya untuk autentikasi dan user management. Tidak bisa generate AWS credentials langsung.
- **B** — Identity Pool bisa handle external identity providers tapi bukan dirancang sebagai user directory (register/login dengan username-password). Butuh identity provider di depannya.
- **D** — IAM user per user aplikasi tidak scalable untuk consumer apps, anti-pattern yang sudah dibahas sebelumnya.

**Konsep yang diuji:** Cognito User Pool vs Identity Pool, perbedaan fungsi, combined use case.

---

## Soal 9 — Jawaban: B

**AWS Config rule dengan auto-remediation menggunakan SSM Automation**

**Kenapa B:**
AWS Config mendukung **auto-remediation** via SSM Automation documents. Ketika Config rule `s3-bucket-server-side-encryption-enabled` menemukan bucket non-compliant, ia otomatis trigger SSM Automation runbook yang mengaktifkan enkripsi — tanpa intervensi manusia. Ini managed service yang low operational overhead.

**Kenapa bukan yang lain:**
- **A** — Lambda scheduled setiap malam adalah solusi custom dengan operational overhead tinggi (deploy, maintain, monitor Lambda), dan ada delay hingga satu malam sebelum remediation.
- **C** — GuardDuty untuk threat detection, tidak bisa detect konfigurasi compliance seperti missing encryption.
- **D** — S3 Block Public Access mengontrol public access settings, tidak ada hubungannya dengan server-side encryption. Keduanya adalah fitur yang berbeda.

**Konsep yang diuji:** AWS Config auto-remediation, SSM Automation, compliance as code.

---

## Soal 10 — Jawaban: B

**Enkripsi RDS hanya bisa diaktifkan saat pembuatan DB instance**

**Kenapa B:**
Ini adalah **exam trap yang sangat sering muncul**. Enkripsi RDS adalah immutable setting — **tidak bisa diaktifkan pada instance yang sudah ada**. Jika butuh enkripsi pada DB yang sudah berjalan tanpa enkripsi, caranya adalah:
1. Buat snapshot dari DB yang ada
2. Copy snapshot dengan enkripsi aktif (pilih KMS key saat copy)
3. Restore DB baru dari encrypted snapshot

**Kenapa bukan yang lain:**
- **A** — Salah. Tidak bisa enable encryption on existing unencrypted RDS instance secara langsung.
- **C** — Enkripsi RDS tidak otomatis. Harus dipilih saat launch dan KMS key harus di-specify jika ingin CMK.
- **D** — Enkripsi RDS dikonfigurasi di DB instance settings, tidak via S3 bucket policy. Snapshot memang disimpan di S3 internal tapi tidak dikonfigurasi via bucket policy.

**Konsep yang diuji:** RDS encryption at rest, immutable encryption setting, snapshot copy untuk enable encryption.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | AWS Shield Standard vs Advanced, DDoS protection |
| 2 | SSM Session Manager session logging |
| 3 | AWS RAM, resource sharing antar accounts |
| 4 | S3 pre-signed URL, time-limited access |
| 5 | Amazon Inspector, vulnerability assessment |
| 6 | AWS Firewall Manager, centralized WAF management |
| 7 | KMS key policy vs IAM policy precedence |
| 8 | Cognito User Pool vs Identity Pool, combined use case |
| 9 | AWS Config auto-remediation, SSM Automation |
| 10 | RDS encryption at rest, immutable setting saat launch |
