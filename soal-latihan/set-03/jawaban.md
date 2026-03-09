# Jawaban — Set 03
## Domain: Secure Architectures

---

## Soal 1 — Jawaban: B

**Amazon Macie**

**Kenapa B:**
Macie adalah managed data security service yang menggunakan machine learning untuk **secara otomatis discover dan classify data sensitif di S3** — termasuk PII (nama, alamat, nomor telepon), financial data (nomor kartu kredit), dan credentials. Macie scan seluruh S3 inventory dan report bucket mana yang mengandung data sensitif.

**Kenapa bukan yang lain:**
- **A** — Inspector untuk vulnerability assessment di EC2, ECR images, dan Lambda functions. Tidak scan konten S3.
- **C** — Config tracking konfigurasi resource (apakah bucket public, versioning aktif, dll), tidak menganalisis konten file.
- **D** — GuardDuty untuk threat detection (siapa yang mengakses data secara anomalous), bukan untuk classify konten data.

**Konsep yang diuji:** Amazon Macie, PII/sensitive data discovery, S3 data classification.

---

## Soal 2 — Jawaban: B

**Network Access Control List (NACL) dengan deny rule**

**Kenapa B:**
NACL adalah **stateless firewall di level subnet** yang mendukung explicit **deny rules** — inilah perbedaan utama dari Security Groups. NACL rules diproses berdasarkan nomor urut, dan explicit deny bisa ditambahkan tanpa mengubah Security Groups yang sudah ada. NACL bisa memblokir specific IP addresses.

**Kenapa bukan yang lain:**
- **A** — Security Groups **tidak support explicit deny rules** — hanya bisa Allow atau tidak Allow. Tidak bisa menambahkan "block IP X".
- **C** — AWS Network Firewall untuk inspeksi mendalam dan rule yang lebih kompleks (domain filtering, intrusion detection). Untuk kebutuhan simple IP blocking di level subnet, NACL lebih tepat dan lebih murah.
- **D** — VPC Flow Logs hanya pencatatan traffic, tidak bisa memblokir apapun.

**Konsep yang diuji:** NACL vs Security Groups, stateless vs stateful, explicit deny rules.

---

## Soal 3 — Jawaban: B

**AWS Certificate Manager (ACM) untuk provision dan deploy certificate ke ALB**

**Kenapa B:**
ACM menyediakan SSL/TLS certificates **gratis** untuk layanan AWS seperti ALB, CloudFront, dan API Gateway. ACM **otomatis merenew** certificate sebelum expired (biasanya 60 hari sebelum expiry) tanpa intervensi manual. Integrasi langsung dengan ALB — tinggal pilih certificate dari ACM saat konfigurasi HTTPS listener.

**Kenapa bukan yang lain:**
- **A** — Upload third-party certificate ke IAM valid secara teknis tapi tidak ada auto-renewal. Tetap butuh proses manual untuk renew.
- **C** — Self-signed certificate tidak dipercaya oleh browser dan bukan praktik yang aman untuk production.
- **D** — Let's Encrypt di EC2 langsung butuh automation sendiri (cron job, certbot) dan tidak terintegrasi native dengan ALB.

**Konsep yang diuji:** AWS Certificate Manager, auto-renewal, ALB HTTPS integration.

---

## Soal 4 — Jawaban: B

**IAM Role di account A dengan trust policy allow account B, developer assume role**

**Kenapa B:**
Cross-account access di AWS menggunakan **IAM Role assumption** — ini adalah pattern yang benar dan aman. Account A membuat role dengan trust policy yang mempercayai account B (atau specific IAM users/roles di account B). Developer di account B kemudian `sts:AssumeRole` untuk mendapat temporary credentials yang berlaku di account A.

**Kenapa bukan yang lain:**
- **A** — Membuat IAM user di account A dan share credentials adalah anti-pattern security. Long-term credentials yang di-share ke orang lain sulit dikontrol dan diaudit.
- **C** — VPC Peering untuk koneksi network antar VPC, tidak ada hubungannya dengan IAM access control.
- **D** — Bucket policy yang allow semua access dari account B terlalu broad — memberikan akses ke seluruh account B tanpa kontrol siapa spesifik yang boleh akses.

**Konsep yang diuji:** Cross-account IAM Role, STS AssumeRole, trust policy.

---

## Soal 5 — Jawaban: B

**IAM Permission Boundaries**

**Kenapa B:**
Permission Boundary adalah fitur IAM yang menetapkan **batas maksimum permissions** yang bisa dimiliki oleh IAM entity (user/role) — bahkan jika ada Allow policy yang lebih luas. Ketika developer membuat role baru, role tersebut tidak bisa memiliki permissions melebihi Permission Boundary yang di-set. Ini mencegah privilege escalation karena developer tidak bisa grant permissions yang mereka sendiri tidak miliki via boundary.

**Kenapa bukan yang lain:**
- **A** — SCP restrict permissions di level Organizations account, bukan per individu user/developer. Tidak target privilege escalation secara spesifik.
- **C** — MFA untuk authentication security, tidak mencegah privilege escalation setelah authenticated.
- **D** — Access Analyzer mendeteksi setelah policy dibuat, tidak preventif saat pembuatan.

**Konsep yang diuji:** IAM Permission Boundaries, privilege escalation prevention, effective permissions.

---

## Soal 6 — Jawaban: B

**S3 Block Public Access di level akun**

**Kenapa B:**
S3 Block Public Access di level akun adalah **satu konfigurasi yang mengoverride semua bucket policies dan ACLs** di akun tersebut — bahkan jika ada bucket policy yang explicit Allow public access, Block Public Access akan tetap mencegahnya. Berlaku untuk semua bucket yang ada maupun yang baru dibuat.

**Kenapa bukan yang lain:**
- **A** — Review per bucket tidak scalable dan tidak mencegah misconfiguration di masa depan untuk bucket baru.
- **C** — SCP deny `s3:PutBucketPolicy` terlalu agresif — tidak bisa update policy sama sekali, termasuk untuk legitimate purposes.
- **D** — Config rule bersifat detective (detect dan alert/remediate setelah terjadi), bukan preventif. Ada window waktu bucket masih publik.

**Konsep yang diuji:** S3 Block Public Access, account-level vs bucket-level settings, preventive control.

---

## Soal 7 — Jawaban: B

**AWS Network Firewall dengan stateful rule groups**

**Kenapa B:**
AWS Network Firewall adalah managed network firewall yang mendukung **deep packet inspection, stateful traffic filtering, domain-based filtering, dan IDS/IPS dengan Suricata-compatible rules**. Ini adalah satu-satunya service AWS yang secara native mendukung Suricata rules untuk intrusion detection. Bisa inspect traffic di layer 7.

**Kenapa bukan yang lain:**
- **A** — Security Groups stateful tapi hanya bisa filter berdasarkan IP, port, dan protocol — tidak ada DPI atau domain filtering.
- **C** — NACL stateless dan hanya bisa filter berdasarkan IP/port. Tidak ada DPI sama sekali.
- **D** — GuardDuty menganalisis log (VPC Flow Logs, DNS, CloudTrail) untuk threat detection, tidak melakukan packet inspection real-time atau blocking.

**Konsep yang diuji:** AWS Network Firewall, deep packet inspection, Suricata rules, IDS/IPS.

---

## Soal 8 — Jawaban: C

**Gagal, karena SCP Deny mengoverride IAM policy Allow**

**Kenapa C:**
IAM policy evaluation logic: **Explicit Deny selalu menang**. Urutan evaluasinya adalah:
1. Explicit Deny (dari SCP, Resource policy, IAM policy) → **DENIED langsung**
2. Allow harus ada di semua layer yang relevan
3. Default Deny jika tidak ada Allow

SCP adalah guardrail di level Organizations — SCP Deny tidak bisa di-override oleh IAM policy Allow di level account, bahkan oleh Administrator.

**Kenapa bukan yang lain:**
- **A & B** — Salah. IAM Allow tidak bisa menang melawan Explicit Deny, terlepas dari seberapa broad Allow-nya.
- **D** — Bucket policy tidak relevan di sini. SCP Deny sudah cukup untuk block action, tidak perlu cek bucket policy.

**Konsep yang diuji:** IAM policy evaluation order, Explicit Deny, SCP vs IAM policy precedence.

---

## Soal 9 — Jawaban: B

**Resource-based policy pada Lambda yang allow invocation dari akun lain**

**Kenapa B:**
Lambda mendukung **resource-based policy** (Lambda function policy) yang bisa di-attach langsung ke function. Policy ini bisa grant permission ke principal dari akun lain untuk invoke Lambda. Ini tidak memerlukan IAM role di akun pemanggil — mereka cukup memiliki izin untuk melakukan `lambda:InvokeFunction` yang sudah di-grant oleh resource policy Lambda.

**Kenapa bukan yang lain:**
- **A** — Share credentials IAM user adalah anti-pattern, sudah dijelaskan di Soal 4.
- **C** — VPC Peering untuk network connectivity, Lambda invocation adalah API call — tidak butuh network peering.
- **D** — Deploy ulang di kedua akun tidak menyelesaikan masalah cross-account invocation, dan menambah duplikasi.

**Konsep yang diuji:** Resource-based policies, Lambda function policy, cross-account access tanpa role assumption.

---

## Soal 10 — Jawaban: B

**ALB listener rule di port 80 yang redirect ke HTTPS**

**Kenapa B:**
ALB mendukung **redirect action** natively di listener rules. Dengan membuat listener di port 80 dan menambahkan default rule berupa redirect ke `https://#{host}:443/#{path}?#{query}`, semua HTTP traffic otomatis di-redirect ke HTTPS di level load balancer — sebelum request mencapai EC2. Tidak butuh perubahan application code.

**Kenapa bukan yang lain:**
- **A** — Menambahkan redirect di application code di setiap instance adalah redundant, menambah coupling, dan sulit di-maintain. Best practice: handle di layer infrastructure (ALB).
- **C** — Route 53 tidak bisa redirect HTTP ke HTTPS pada level protocol. Route 53 hanya DNS — bisa redirect domain ke domain lain tapi tidak menangani HTTP → HTTPS redirect.
- **D** — HSTS adalah HTTP response header (security policy browser), bukan konfigurasi Security Group. Security Group tidak punya konsep HSTS.

**Konsep yang diuji:** ALB listener rules, HTTP to HTTPS redirect, load balancer best practices.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | Amazon Macie, PII/sensitive data discovery di S3 |
| 2 | NACL vs Security Groups, explicit deny, stateless filtering |
| 3 | AWS Certificate Manager, auto-renewal, ALB integration |
| 4 | Cross-account IAM Role, STS AssumeRole |
| 5 | IAM Permission Boundaries, privilege escalation prevention |
| 6 | S3 Block Public Access level akun |
| 7 | AWS Network Firewall, deep packet inspection, Suricata |
| 8 | IAM policy evaluation order, Explicit Deny vs Allow |
| 9 | Lambda resource-based policy, cross-account invocation |
| 10 | ALB HTTP to HTTPS redirect listener rule |
